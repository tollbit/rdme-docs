---
title: AWS (S3, CloudFront, ALB)
excerpt: Learn how to implementate TollBit with Amazon S3, ALB, and/or CloudFront.
deprecated: false
hidden: false
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

First, go to the WAF & Shield and create a new Web ACL. Ensure that the ACL being created is for CloudFront distributions. Add your existing CloudFront distribution to this ACL under the "Associated AWS resources" section of the page.

![Aws Acl Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/aws-acl-configuration.png)

Once you've created the ACL, you can choose any rules you'd like to enable bot detection. AWS Marketplace has managed bot detection rules that you can add to your ACL. We will provide our own WAF rule as well. To use our WAF rule, select the option for using your own rules and rule groups, and use the JSON editor. Copy and paste the following rule:

```json
{
    "Name": "cloudfront-agent-rule",
    "Priority": 0,
    "Statement": {
        "OrStatement": {
            "Statements": [
                {
                    "ByteMatchStatement": {
                        "SearchString": "amazonbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "amzn-searchbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "anthropic-ai",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "bytespider",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "ccbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "chatgpt-user",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "claude-code",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "claude-searchbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "claude-user",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "claude-web",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "claudebot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "cohere-ai",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "diffbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "exabot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "gptbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "meta-externalagent",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "meta-webindexer",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "oai-adsbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "oai-searchbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "omgili",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "perplexity-user",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "perplexitybot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "timpibot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                },
                {
                    "ByteMatchStatement": {
                        "SearchString": "youbot",
                        "FieldToMatch": {
                            "SingleHeader": {
                                "Name": "user-agent"
                            }
                        },
                        "TextTransformations": [
                            {
                                "Priority": 0,
                                "Type": "LOWERCASE"
                            }
                        ],
                        "PositionalConstraint": "CONTAINS"
                    }
                }
            ]
        }
    },
    "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "cloudfront-agent-rule"
    },
    "Action": {
        "Allow": {
            "CustomRequestHandling": {
                "InsertHeaders": [
                    {
                        "Name": "Bot",
                        "Value": "true"
                    }
                ]
            }
        }
    }
}
```

This will detect the top known AI bots. Next, for the action, be sure to choose "Allow" and to add a custom header. Ours is called `bot`, but feel free to make this anything unique.

![Waf Action](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/waf-action.png)

<br />

**Create a new Lambda Function**

You can call your function name something straightforward like "**tollbit\_agent\_site**", and select the latest Node.js Runtime version.

![](https://files.readme.io/8fba56a790bd54142918760c4bd651ff6bc490d08f0fbde067adcda6436a16b8-image10.png)

<br />

**Add the Lambda\@Edge Code**

Paste the snippet below. This will use the header that the WAF added when it detected a bot to understand if a specific request is a bot request, and route it appropriately.

```javascript
import https from 'https';

  const BOT_HEADER = 'x-amzn-waf-bot';

  // Only these origin response headers are passed back to the viewer.
  // Everything else (hop-by-hop, x-amz-*, keep-alive, etc.) is dropped,
  // since CloudFront rejects the entire response if a disallowed header is set.
  const ALLOWED_RESPONSE_HEADERS = new Set([
    'content-type', 'content-encoding', 'content-language',
    'set-cookie', 'location', 'vary', 'etag', 'last-modified',
    'expires', 'retry-after', 'www-authenticate', 'x-robots-tag'
  ]);

  // Viewer-request lambdas are killed at 5s total; time out the origin
  // fetch earlier so we can still fail open.
  const PROXY_TIMEOUT_MS = 4000;

  const proxy = (url, reqHeaders) => new Promise((resolve, reject) => {
    const req = https.get(url, { headers: reqHeaders }, (res) => {
      const chunks = [];
      res.on('data', chunk => chunks.push(chunk));
      res.on('error', reject);
      res.on('end', () => resolve({
        status: res.statusCode,
        headers: res.headers,
        body: Buffer.concat(chunks)
      }));
    });
    req.setTimeout(PROXY_TIMEOUT_MS, () => req.destroy(new Error('tollbit origin timeout')));
    req.on('error', reject);
  });

  export const handler = async (event) => {
    const request = event.Records[0].cf.request;
    const headers = request.headers;

    const isBot = headers[BOT_HEADER]?.[0]?.value === 'true';

    if (!isBot) {
      return request;
    }

    try {
      const host = headers['host'][0].value;
      const tollbitDomain = `tollbit.${host.replace(/^www\./, '')}`;

      const reqHeaders = Object.fromEntries(
        Object.entries(headers).map(([k, v]) => [k, v[0].value])
      );
      reqHeaders['host'] = tollbitDomain;
      delete reqHeaders[BOT_HEADER]; // internal signal, don't forward it

      const qs = request.querystring ? `?${request.querystring}` : '';
      const { status, headers: resHeaders, body } = await proxy(
        `https://${tollbitDomain}${request.uri}${qs}`,
        reqHeaders
      );

      const cfHeaders = {};
      for (const [key, value] of Object.entries(resHeaders)) {
        if (!ALLOWED_RESPONSE_HEADERS.has(key.toLowerCase())) continue;
        const values = Array.isArray(value) ? value : [value];
        cfHeaders[key.toLowerCase()] = values.map(v => ({ key, value: v }));
      }
      cfHeaders['cache-control'] = [{ key: 'Cache-Control', value: 'no-store' }];

      return {
        status: String(status),
        headers: cfHeaders,
        bodyEncoding: 'base64',
        body: body.toString('base64')
      };
    } catch (err) {
      console.error('tollbit proxy failed, falling back to origin:', err.message);
      return request; // fail open: serve normal origin content
    }
  };
```

**Deploy the Lambda first**

On the left tab, click “Deploy”.

![](https://files.readme.io/e08e6662396ae9f7985c67d76fdb3ecf79925eafe1528e5180517a447eb1abc2-image16.png)

<br />

**Deploy to Lambda\@Edge**

<Callout icon="📘" theme="info">
  ### Pro Tip

  Before deploying, ensure that your site’s `tollbit` subdomain is set up and running correctly. Otherwise this will cause errors in the lambda.
</Callout>

Scroll to the top, click the “Actions” dropdown, and Deploy to Lambda\@Edge under Capabilities.

![](https://files.readme.io/eeca2bf7c322da5a65a990ef2989f89162ec4fac4935a37fd52fd0ce9ac0c667-image2.png)

<br />

Ensure you pick your Cloudfront distribution and that the event is “Viewer request”. Once done, click deploy at the bottom.

![](https://files.readme.io/ab2e9ced4bb8379db96c75e79e930a9249441cfe4c7361a1fc0ca977feaa7ec7-image3.png)

<br />

<Callout icon="🚧" theme="warn">
  ### Note

  Deploying this has the lambda intercept all traffic to your site. Please ensure you have tested this change.

  If you run into a deploy error and permissions issue, you will need to update your lambda’s execution role.&#x20;

  Click on the “Configure” tab, and then on the left, click on “Permissions” and click on the blue highlighted role name. In this case it would be “**tollbit\_agent\_site-role-z6218mit**”

  ![](https://files.readme.io/d5a221325cd0a12b7636091242f340883cc26ef747fdfae362006cbc46f4e38a-image5.png)



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

  ![](https://files.readme.io/82ec2e686885ea72b2264167ca0207d6eddb2f53a2d7bffdf9fb3a4941185d13-image13.png)

  Once you update this, you can try deploying the Lambda\@Edge to your cloudfront distribution again. You may need to wait a few minutes and refresh the lambda page before the permissions propagate.
</Callout>

<br />

**Updating your Lambda**

If you need to update your lambda, you can follow these steps.

<br />**Step 1** - Go back to the Lambda code editor and edit the code, and click Deploy once you are done<br />**Step 2&#x20;**- Under the actions drop down at the top, click Publish New Version

**Step 3** - Take note of the latest version number, these will be incrementing numbers

![](https://files.readme.io/b114470303c6af72161aeb977e0f851895d6eeb9e783747129b8648e9f4397e0-image11.png)

<br />

**Step 4** - Go to your cloudfront distribution and go to Behaviors, and check the default behavior. If you have multiple, check the one that regular traffic routes through, and then click edit.

![](https://files.readme.io/81d8981e7c297ec98e69eead2d31560acdddf73edc6a3c91c0f3c3a572f6ca4b-image15.png)

<br />

**Step 5** - Scroll to the bottom of this next page and update the version number of your lambda. You will just edit the number all the way at the end to match the newly updated version number.

![](https://files.readme.io/7eb0ed83938e85b29bc74294e44f677f8a8e84e05271f2c967a25e43b7e7225f-image7.png)

<br />

### AWS WAF + CloudFront Route To Agent Site (Origin Request)

This is an alternate to the setup above. It does the same thing — routes detected bots to your Agent Site — but attaches the Lambda to the **Origin request** event instead of **Viewer request**, and uses CloudFront's cache key to keep bot and human responses separate.

<Callout icon="📘" theme="info">
  ### When To Use This

  Use this setup instead of the Viewer request one if your cache policy relies on CloudFront viewer headers such as `CloudFront-Viewer-Country`, `CloudFront-Viewer-City`, or the device type headers.

  CloudFront adds those headers *after* the viewer request event, so they are not available to a Viewer request Lambda\@Edge function. If a Viewer request function tries to add `CloudFront-Viewer-Country`, CloudFront rejects the response and returns a `502` to the visitor.
</Callout>

<br />

#### Set up Your WAF

Identical to the Viewer request setup above. Create the Web ACL, associate your CloudFront distribution, and add the `cloudfront-agent-rule` bot detection rule with the Allow action and the `bot` custom request header.

AWS prefixes WAF custom request headers with `x-amzn-waf-`, so a header named `bot` arrives as `x-amzn-waf-bot`.

<br />

#### Update Your Cache Policy

This step is what keeps a human from ever being served a bot response and vice versa, and the integration will not work correctly without it.

Go to **CloudFront → Policies → Cache** and create a cache policy (or edit your existing one, if it is not an AWS managed policy — managed policies cannot be edited).

* Under **Headers**, choose *Include the following headers* and add **`x-amzn-waf-bot`**, along with any viewer headers your caching already relies on, such as `CloudFront-Viewer-Country`.
* Set **Minimum TTL** to **`0`**.
* Leave query strings and cookies matching whatever your distribution uses today.

Then go to **Distribution → Behaviors**, edit the behavior that regular traffic routes through, and attach the policy under **Cache key and origin requests**.

<!-- screenshot: cache policy header allowlist showing x-amzn-waf-bot -->

<Callout icon="🚧" theme="warn">
  ### Note

  Minimum TTL must be `0`. With a non-zero Minimum TTL, CloudFront caches Agent Site responses even though they are returned with `no-store`.
</Callout>

<br />

#### Update Your Origin Request Policy

On the same **Behaviors** edit screen, set **Origin request policy** to **`AllViewer`**. Your Agent Site needs the visitor's `User-Agent` and other request headers, and this forwards all of them.

If `AllViewer` is greyed out and cannot be selected, your behavior's origin is an S3 bucket, an API Gateway, or a Lambda function URL. Those origins require the `Host` header to be their own domain, so CloudFront blocks the policy. Select **`AllViewerExceptHostHeader`** instead — it forwards everything except `Host`, which is all this setup needs.

<!-- screenshot: behavior edit screen with cache policy + origin request policy selected -->

<br />

#### Create a new Lambda Function

Same as the Viewer request setup — call it something like "**tollbit\_agent\_site**" and select the latest Node.js runtime.

<br />

**Add the Lambda\@Edge Code**

Paste the snippet below and change `TOLLBIT_DOMAIN` on the second line to your site's TollBit subdomain. Lambda\@Edge does not support environment variables, so this value has to live in the code.

Rather than proxying the request, this function points CloudFront at your Agent Site by rewriting the origin. CloudFront then fetches from it directly, so response headers, cookies, and large pages all pass through untouched.

```javascript
const BOT_HEADER = 'x-amzn-waf-bot';

// Change this to your site's TollBit subdomain.
const TOLLBIT_DOMAIN = 'tollbit.example.com';

export const handler = async (event) => {
  const request = event.Records[0].cf.request;

  // The WAF rule inserts this header only on a bot match, so anything other
  // than an exact "true" — including the header being absent — is a human.
  if (request.headers[BOT_HEADER]?.[0]?.value !== 'true') {
    return request;
  }

  request.origin = {
    custom: {
      domainName: TOLLBIT_DOMAIN,
      port: 443,
      protocol: 'https',
      path: '',
      sslProtocols: ['TLSv1.2'],
      readTimeout: 30,
      keepaliveTimeout: 5,
      customHeaders: {}
    }
  };
  request.headers.host = [{ key: 'Host', value: TOLLBIT_DOMAIN }];

  // The cache key is computed before this event fires, so removing the header
  // here does not affect where the response is stored.
  delete request.headers[BOT_HEADER];

  return request;
};
```

<br />

**Deploy the Lambda first**

On the left tab, click "Deploy".

<br />

**Deploy to Lambda\@Edge**

<Callout icon="📘" theme="info">
  ### Pro Tip

  Before deploying, ensure that your site's `tollbit` subdomain is set up and running correctly. Unlike the Viewer request setup, this function cannot fall back to your origin if the Agent Site is unreachable — CloudFront will return an error to the visitor instead.
</Callout>

Scroll to the top, click the "Actions" dropdown, and choose Deploy to Lambda\@Edge under Capabilities.

Pick your CloudFront distribution and set the event to **"Origin request"**. Once done, click deploy at the bottom.

<!-- screenshot: deploy dialog with Origin request selected -->

If you are migrating from the Viewer request setup, remove the old **Viewer request** function association from the behavior at the same time. Leaving both attached will proxy the request twice.

The execution role requirements and the "Updating your Lambda" steps are identical to the Viewer request setup above.

<br />

#### Order of Operations

Apply the changes in this order:

1. Update the cache policy and origin request policy, and wait for the distribution to show **Deployed**.
2. Attach the Lambda to the **Origin request** event.

Doing it in this order matters. If the Lambda goes on first, there is a window where `x-amzn-waf-bot` is not yet part of the cache key, and Agent Site responses can be cached and then served to human visitors — the exact thing this setup prevents. In the recommended order the worst case is the reverse and far less harmful: a bot receives a cached human page until the entry expires.

No invalidation is needed. Adding `x-amzn-waf-bot` to your cache policy changes the cache key for every object, so entries cached before the change are no longer matched and age out on their own. Keep the gap between the two steps short — anything cached in between expires on your normal TTL.

<br />

### Just Lambda + CloudFront Route To Agent Site (No WAF)

If you'd like to set this up without WAF managing the bot detection and want to only use Lambda and CloudFront, use the following Lambda instead. This will do the bot user agent check directly within the Lambda.

```javascript
import https from 'https';

  const TOLLBIT_USER_AGENTS = [
    'Amazonbot', 'Amzn-SearchBot', 'anthropic-ai', 'Bytespider', 'CCBot',
    'ChatGPT-User', 'claude-code', 'Claude-SearchBot', 'Claude-User',
    'Claude-Web', 'ClaudeBot', 'cohere-ai', 'Diffbot', 'ExaBot', 'GPTBot',
    'meta-externalagent', 'Meta-Webindexer', 'OAI-AdsBot', 'OAI-SearchBot',
    'omgili', 'Perplexity-User', 'PerplexityBot', 'Timpibot', 'YouBot'
  ];

  const TOLLBIT_UA_REGEX = new RegExp(TOLLBIT_USER_AGENTS.join('|'), 'i');

  const matchesUserAgent = (ua) => ua && TOLLBIT_UA_REGEX.test(ua);

  // Only these origin response headers are passed back to the viewer.
  // Everything else (hop-by-hop, x-amz-*, keep-alive, etc.) is dropped,
  // since CloudFront rejects the entire response if a disallowed header is set.
  const ALLOWED_RESPONSE_HEADERS = new Set([
    'content-type', 'content-encoding', 'content-language',
    'set-cookie', 'location', 'vary', 'etag', 'last-modified',
    'expires', 'retry-after', 'www-authenticate', 'x-robots-tag'
  ]);

  // Viewer-request lambdas are killed at 5s total; time out the origin
  // fetch earlier so we can still fail open.
  const PROXY_TIMEOUT_MS = 4000;

  const proxy = (url, reqHeaders) => new Promise((resolve, reject) => {
    const req = https.get(url, { headers: reqHeaders }, (res) => {
      const chunks = [];
      res.on('data', chunk => chunks.push(chunk));
      res.on('error', reject);
      res.on('end', () => resolve({
        status: res.statusCode,
        headers: res.headers,
        body: Buffer.concat(chunks)
      }));
    });
    req.setTimeout(PROXY_TIMEOUT_MS, () => req.destroy(new Error('tollbit origin timeout')));
    req.on('error', reject);
  });

  export const handler = async (event) => {
    const request = event.Records[0].cf.request;
    const headers = request.headers;

    const userAgent = headers['user-agent']?.[0]?.value ?? '';

    if (!matchesUserAgent(userAgent)) {
      return request;
    }

    try {
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
        if (!ALLOWED_RESPONSE_HEADERS.has(key.toLowerCase())) continue;
        const values = Array.isArray(value) ? value : [value];
        cfHeaders[key.toLowerCase()] = values.map(v => ({ key, value: v }));
      }
      cfHeaders['cache-control'] = [{ key: 'Cache-Control', value: 'no-store' }];

      return {
        status: String(status),
        headers: cfHeaders,
        bodyEncoding: 'base64',
        body: body.toString('base64')
      };
    } catch (err) {
      console.error('tollbit proxy failed, falling back to origin:', err.message);
      return request; // fail open: serve normal origin content
    }
  };
```

<br />

<br />
