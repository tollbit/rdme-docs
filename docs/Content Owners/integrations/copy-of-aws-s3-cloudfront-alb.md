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

#### Set up Your WAF

<br />

**Create a new Lambda Function**

You can call your function name something straightforward like "**tollbit\_agent\_site**", and select the latest Node.js Runtime version.

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

**Deploy the Lambda first**

On the left tab, click “Deploy”.


<Image src="https://files.readme.io/1e5ef4f7f15aa886221bde643334124c23d603354b87b6d22092f25e5af50c8f-Screenshot_2026-07-09_at_12.00.50_PM.png" align="left" width="-15px" wrap={false} />


<br />

**Deploy to Lambda\@Edge**

<Callout icon="📘" theme="info">
  ### Pro Tip

  Before deploying, ensure that your site’s `tollbit` subdomain is set up and running correctly. Otherwise this will cause errors in the lambda.
</Callout>

Scroll to the top, click the “Actions” dropdown, and Deploy to Lambda\@Edge under Capabilities.

![](https://files.readme.io/1f8009ef5a14409cf054e99521dc2d0ea4e780f55e157441549c14f1e99cb44e-Screenshot_2026-07-09_at_12.10.40_PM.png)

<br />

Ensure you pick your Cloudfront distribution and that the event is “Viewer request”. Once done, click deploy at the bottom.

![](https://files.readme.io/918c5df2cfeeeb8eb8a1972a6bc9d7e3f3ae1490fdec1e7a2d44a78638e6f448-Screenshot_2026-07-09_at_12.11.53_PM.png)

<br />

<Callout icon="🚧" theme="warn">
  ### Note

  Deploying this has the lambda intercept all traffic to your site. Please ensure you have tested this change.

  If you run into a deploy error and permissions issue, you will need to update your lambda’s execution role.&#x20;

  Click on the “Configure” tab, and then on the left, click on “Permissions” and click on the blue highlighted role name. In this case it would be “**tollbit\_agent\_site-role-z6218mit**”

  ![](https://files.readme.io/d3cb4c2e775534ed0daedb7601f7402d8b8d0aa7fac294f59396226b65153f56-Screenshot_2026-07-09_at_12.18.15_PM.png)

  Go to the Trust Relationship tab and edit the Trust Policy and make sure it contains the following block. Make sure to click “Update Policy” at the bottom after updating.

  ```text
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Principal": {
                  "Service": [
                      "edgelambda.amazonaws.com",
                      "lambda.amazonaws.com"
                  ]
              },
              "Action": "sts:AssumeRole"
          }
      ]
  }
  ```

  ![](https://files.readme.io/52e80994e22a349bf89bfea36f29d79076dc8adbc4e45b8d3fabef6cf49b164d-Screenshot_2026-07-09_at_12.22.35_PM.png)

  Once you update this, you can try deploying the Lambda\@Edge to your cloudfront distribution again. You may need to wait a few minutes and refresh the lambda page before the permissions propagate.
</Callout>

<br />

**Updating your Lambda**

If you need to update your lambda, you can follow these steps.

<br />**Step 1** - Go back to the Lambda code editor and edit the code, and click Deploy once you are done<br />**Step 2&#x20;**- Under the actions drop down at the top, click Publish New Version

**Step 3** - Take note of the latest version number, these will be incrementing numbers

![](https://files.readme.io/0d70ea2541d88d2e7b81dec733d7c9a5d2d9988c7a9d4c87c5ea4011e924b9bd-Screenshot_2026-07-09_at_12.25.03_PM.png)

<br />

**Step 4** - Go to your cloudfront distribution and go to Behaviors, and check the default behavior. If you have multiple, check the one that regular traffic routes through, and then click edit.

![](https://files.readme.io/6a53f5c5772df0fbd7072d04b5734bf5562732b966691734ab55603ef5d8eb64-Screenshot_2026-07-09_at_12.25.30_PM.png)

**Step 5** - Scroll to the bottom of this next page and update the version number of your lambda. You will just edit the number all the way at the end to match the newly updated version number.

![](https://files.readme.io/4b5fc736be1880fee7ddb4ac3562f737987da79dedfdfe68e814b1c2c8019b2c-Screenshot_2026-07-09_at_12.25.58_PM.png)

<br />

<br />

<br />

<br />
