---
title: AWS (S3, CloudFront, ALB) v2
excerpt: Learn how to implementate TollBit with Amazon S3, ALB, and/or CloudFront.
deprecated: false
hidden: true
metadata:
  robots: index
---
# Steps for Analytics

### Forwarding Logs with ALB

To forward logs from an ALB, follow these steps outlined in the <Anchor target="_blank" href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/enable-access-logging.html">AWS docs</Anchor>.

Once you have started forwarding your logs to an S3 bucket, create an IAM policy to allow TollBit to access your logs: If your logs are already being sent to an S3 bucket, add the following IAM policy to your bucket to enable TollBit to process your logs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowTollbitAccountsAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::339712821696:root",
          "arn:aws:iam::654654318267:root"
        ]
      },
      "Action": ["s3:GetObject*", "s3:ListBucket*"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    }
  ]
}
```

Once you have created the policy, reach out to [team@tollbit.com](mailto:team@tollbit.com) to coordinate with our engineering team on the rest of the TollBit Analytics setup.

To finalize your setup, we will need access to the directory in your S3 bucket where your logs are stored, along with the pattern for how the logs are stored for instance `/service/logs/2024/12/04/log-file`

<Callout icon="📘" theme="info">
  ### Pro Tip

  If your bucket has ACLs, follow the instructions [here](/docs/other#buckets-with-acls-access-control-lists).
</Callout>

### Forwarding Logs with Cloudfront

To forward logs from Cloudfront follow these steps:

Enable standard logging for your Cloudfront distribution following the <Anchor target="_blank" href="https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging.html#set-up-standard-logging">AWS docs</Anchor>.

Point your logs at an S3 Bucket. Note, we only currently support the default w3c, tab delimited format with the default 33 fields that are included in the logs. If you wish to use JSON and/or modify the fields that Cloudfront logs, please reach out to [team@tollbit.com](mailto:team@tollbit.com) and we can get that set up for you.

Create the following IAM policy for your bucket to allow TollBit to process your logs: If your logs are already being sent to an S3 bucket, add the following IAM policy to your bucket to enable TollBit to process your logs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowTollbitAccountsAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::339712821696:root",
          "arn:aws:iam::654654318267:root"
        ]
      },
      "Action": ["s3:GetObject*", "s3:ListBucket*"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    }
  ]
}
```

Once you have started forwarding your logs to an S3 bucket, and granted TollBit access, reach out to [team@tollbit.com](mailto:team@tollbit.com) to coordinate with our engineering team on the rest of the TollBit Analytics setup. To finalize your setup, we will need access to the directory in your S3 bucket where your logs are stored, along with the pattern for how the logs are stored for instance `/service/logs/2024/12/04/log-file`

<Callout icon="📘" theme="info">
  ### Pro Tip

  If your bucket has ACLs, follow the instructions [here](/docs/other#buckets-with-acls-access-control-lists).
</Callout>

# Steps for Agent Site

### AWS WAF + CloudFront Route To Agent Site

Agent Site can be set up with AWS Lambda\@Edge.&#x20;

<Callout icon="🚧" theme="warn">
  ### Note

  If you have set up Agent Site via redirection previously, you will need to disable the WAF and Cloudfront Function after this is created to ensure correct rewrite/proxy flow.
</Callout>

**Create a new Lambda Function**

You can call your function name something straightforward like "tollbit\_agent\_site", and select the latest Node.js Runtime version.

![](https://files.readme.io/3f513c43f3acaad687a91af800a5a613d94fd65ed60ac19f56d021685c6df2c8-Screenshot_2026-07-09_at_11.56.20_AM.png)

<br />

**Add the Worker Code**

Paste the snippet below. You may need to add/remove user agents as necessary based on who you want to target. Ensure that these user agents are allowed in your agent site configurations.

```json
import https from 'https';


  const TOLLBIT_USER_AGENTS = [
  'Amazonbot', 'Amzn-SearchBot', 'anthropic-ai', 'Bytespider', 'CCBot', 'ChatGPT-User', 'claude-code', 'Claude-SearchBot', 'Claude-User', 'Claude-Web', 'ClaudeBot', 'cohere-ai', 'Diffbot', 'ExaBot', 'Exabot', 'GPTBot', 'meta-externalagent', 'Meta-Webindexer', 'OAI-AdsBot', 'OAI-SearchBot', 'Perplexity-User', 'PerplexityBot', 'Timpibot', 'YouBot'
];


const TOLLBIT_UA_REGEX = new RegExp(
  TOLLBIT_USER_AGENTS.map(p => p instanceof RegExp ? p.source : p).join('|'),
  'i'
);


const matchesUserAgent = (ua) => ua && TOLLBIT_UA_REGEX.test(ua);


  const RESTRICTED_HEADERS = new Set([
    'connection', 'content-length', 'transfer-encoding', 'via',
    'x-amz-cf-id', 'x-amzn-trace-id', 'x-cache', 'x-forwarded-for'
  ]);


  const proxy = (url, reqHeaders) => new Promise((resolve, reject) => {
    const req = https.get(url, { headers: reqHeaders }, (res) => {
      const chunks = [];
      res.on('data', chunk => chunks.push(chunk));
      res.on('end', () => resolve({
        status: res.statusCode,
        headers: res.headers,
        body: Buffer.concat(chunks)
      }));
    });
    req.on('error', reject);
  });


  export const handler = async (event) => {
    const request = event.Records[0].cf.request;
    const headers = request.headers;


    const userAgent = headers['user-agent']?.[0]?.value ?? '';


    if (matchesUserAgent(userAgent)) {
      const host = headers['host'][0].value;
      const tollbitDomain = `tollbit.${host.replace(/^www\./, '')}`;


      const reqHeaders = Object.fromEntries(
        Object.entries(headers).map(([k, v]) => [k, v[0].value])
      );
      reqHeaders['host'] = tollbitDomain;


      const qs = request.querystring ? `?${request.querystring}` : '';
      const { status, headers: resHeaders, body } = await proxy(
        `https://${tollbitDomain}${request.uri}${qs}`,
        reqHeaders
      );


      const cfHeaders = {};
      for (const [key, value] of Object.entries(resHeaders)) {
        if (!RESTRICTED_HEADERS.has(key.toLowerCase()) && typeof value === 'string') {
          cfHeaders[key.toLowerCase()] = [{ key, value }];
        }
      }
      cfHeaders['cache-control'] = [{ key: 'Cache-Control', value: 'no-store' }];


      return {
        status: String(status),
        headers: cfHeaders,
        bodyEncoding: 'base64',
        body: body.toString('base64')
      };
    }


    return request;
  };

```

<br />

**Deploy the Lambda first**

On the left tab, click “deploy”


<Image src="https://files.readme.io/1e5ef4f7f15aa886221bde643334124c23d603354b87b6d22092f25e5af50c8f-Screenshot_2026-07-09_at_12.00.50_PM.png" align="left" width="-15px" wrap={false} />


<br />

**Deploy to Lambda\@Edge**

<Callout icon="📘" theme="info">
  ### Pro Tip

  Before deploying, ensure that your site’s tollbit subdomain is set up and running correctly. Otherwise this will cause errors in the lambda.
</Callout>

Scroll to the top, click the “Actions” dropdown, and Deploy to Lambda\@Edge.

<br />

```javascript
function handler(event) {
  if (event.request.headers['x-amzn-waf-bot'] !== undefined) {
    const host = event.request.headers.host.value
    const uri = event.request.uri
    const newurl = `https://tollbit.${host}${uri}`
    const response = {
      statusCode: 302,
      statusDescription: 'Found',
      headers: { location: { value: newurl } },
    }
    return response
  }
  return event.request
}
```

Earlier, our WAF rule had set a header called `bot` onto the request if it matched the rule. Amazon automatically appends `x-amzn-waf-` to the header, so the actual header to look for is now called `x-amzn-waf-bot`. If this header exists, it means that our WAF rule detected that this request
is a bot request, so we now want to forward it to our `tollbit` subdomain. Once you are ready, save the changes and publish this code. On the publish tab, you will then need to associate this function to your existing CloudFront distribution.

<Callout icon="🚧" theme="warn">
  ### Note

  This code snippet will run for every request to your distribution. Please<br />ensure you've tested this function before completing this step.
</Callout>

<br />
