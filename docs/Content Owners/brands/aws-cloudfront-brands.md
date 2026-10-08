---
title: AWS CloudFront for Brands
excerpt: >-
  Set up TollBit analytics, Agent Site, and visitor routing from cited content
  for a brand site served through Amazon CloudFront.
deprecated: false
hidden: true
metadata:
  robots: noindex
---
This guide covers setting up TollBit for a site served through Amazon CloudFront: streaming logs to our platform for analytics, setting up Agent Site, and routing visitors from cited content.

# Before You Start

#### Allow TollBit's IP Addresses

<Callout icon="🚧" theme="warn">
  ### Required

  TollBit must be able to reach your sites. We request your pages to set up your Agent Site, to show you accurate analytics, and to power the Agent CMS view of your content. If our requests are blocked, challenged, or rate-limited, none of these will work correctly.
</Callout>

TollBit sends these requests from a fixed set of static IP addresses, published at [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt). At the time of writing, the list is:

```text
52.22.183.94
3.220.109.109
```

Always use the published list as the source of truth, and check it again if TollBit tells you it has changed.

Make sure these addresses are allowed through every security layer that could block or challenge our requests. This applies to every hostname you onboard with us:

- **AWS WAF**: the rules in your Web ACL, including IP set rules, rate-based rules, and managed rule groups such as Bot Control.
- **Any firewall or WAF in front of your origin servers**, including security groups and any protection outside of AWS.

If you are unsure whether your configuration blocks us, contact [team@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

# Steps for Analytics

#### Enable CloudFront Standard Logging

Enable standard logging for your distribution by following the <Anchor target="_blank" href="https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging.html#set-up-standard-logging">AWS documentation</Anchor>, and point the logs at an S3 bucket. We currently support the default W3C, tab-delimited format with the default 33 fields. If you would like to use JSON or change the logged fields, contact [team@tollbit.com](mailto:team@tollbit.com) and we will set that up with you.

#### Grant TollBit Access to the Bucket

Add the following policy to the bucket so TollBit can read the logs:

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

Then send [team@tollbit.com](mailto:team@tollbit.com) the bucket name, the directory your logs are written to, and the naming pattern of the log files (for example `/service/logs/2026/09/21/log-file`) so we can finish the analytics setup.

# Steps for Agent Site

AI agents that request your site are routed to your Agent Site on your `tollbit` subdomain, while visitors continue to reach your own origin. AWS WAF identifies the agents, and a CloudFront Function on the Viewer request event switches the origin for those requests. There is no Lambda to deploy.

<Callout icon="📘" theme="info">
  ### Test Before Production

  The steps below change how requests are routed at the edge. Where possible, apply them to a staging distribution first, or to a single low-traffic behavior, and confirm the results with the test requests in Verifying the Setup before applying them to the behavior that serves your live traffic.
</Callout>

#### Set Up Your WAF

In **WAF & Shield**, create a Web ACL for CloudFront distributions and associate it with your distribution under the "Associated AWS resources" section of the page.

![Aws Acl Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/aws-acl-configuration.png)

Then add our agent detection rule. Select the option for using your own rules and rule groups, and paste the following into the JSON editor.

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
                        "SearchString": "shapbot",
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
                        "SearchString": "shap-user",
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

The rule matches the `User-Agent` header against known AI agents such as `GPTBot`, `ClaudeBot` and `PerplexityBot`. Its action is **Allow** with a custom request header `Bot: true`. AWS prefixes custom WAF headers with `x-amzn-waf-`, so it reaches CloudFront as `x-amzn-waf-bot`, which is the header the rest of this guide relies on. The rule is set to priority 0 so it evaluates first; if you have existing rules, place it where it makes sense in your stack.

![Waf Action](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/waf-action.png)

#### Add the TollBit Origins

Your `tollbit` subdomain is set up as part of onboarding your site to the TollBit platform, so complete that first. Then go to **Distribution → Origins** and create two origins. CloudFront names an origin after its domain by default, so make sure to set the names shown here; the function refers to them.

The fallback origin, `fallback.tollbit.com`, requires the two custom headers shown in the table, added under **Add custom header** in the origin's settings. CloudFront attaches them only to requests it sends to that origin. TollBit uses `X-Tollbit-Host` to tell which site a URL belongs to, and rejects requests without it.

| Setting | Agent Site origin | Fallback origin |
| :--- | :--- | :--- |
| Origin domain | Your TollBit subdomain, e.g. `tollbit.example.com` | `fallback.tollbit.com` |
| Name | `tollbit-origin` | `tollbit-fallback` |
| Protocol | HTTPS only, TLSv1.2 | HTTPS only, TLSv1.2 |
| Custom headers | None | `X-Tollbit-Host`: your site's hostname, e.g. `www.example.com`<br />`X-Tollbit-Fallback-Mode`: `passthrough` |
| Connection attempts | Default | `1` |
| Connection timeout | Default | `2` seconds |
| Response timeout | Default | `3` seconds |

Also note the **Name** of your existing site origin. The function below calls it `site-origin`; change the constant to match yours. Your behaviors keep pointing at your existing origin. The fallback origin is used for routing visitors from cited content, described later in this guide.

#### Update Your Cache Policy

This step keeps a visitor from being served an agent's response and an agent from being served a visitor's. The integration will not work correctly without it. Go to **CloudFront → Policies → Cache** and create a cache policy, or edit your existing one if it is not AWS managed.

- Under **Headers**, choose **Include the following headers** and add `x-amzn-waf-bot` and `x-tollbit-fallback-probe`, along with any headers your caching already relies on.
- Set **Minimum TTL** to `0`. With a non-zero minimum, CloudFront caches Agent Site responses even though they are returned with `no-store`.
- Leave query strings and cookies matching what your distribution uses today.

If your behavior currently uses an AWS managed policy such as `CachingOptimized`, a new policy does not inherit its settings. Copy them over first, in particular Gzip and Brotli compression and the Default and Maximum TTL values.

#### Create the CloudFront Function

Go to **CloudFront → Functions**, create a function (for example `tollbit_agent_site`) with the **cloudfront-js-2.0** runtime, and paste the code below. The older 1.0 runtime cannot change the origin.

```javascript
import cf from 'cloudfront';

const BOT_HEADER = 'x-amzn-waf-bot';
const PROBE_HEADER = 'x-tollbit-fallback-probe';

// The Names you gave the origins on your distribution.
const SITE_ORIGIN_ID = 'site-origin';
const BOT_ORIGIN_ID = 'tollbit-origin';
const FALLBACK_ORIGIN_ID = 'tollbit-fallback';

function handler(event) {
  var request = event.request;
  var botRequestHeader = request.headers[BOT_HEADER];

  // The WAF rule inserts this header only on a bot match. Anything other than
  // an exact "true", including the header being absent, is treated as a human.
  if (botRequestHeader && botRequestHeader.value === 'true') {
    cf.selectRequestOriginById(BOT_ORIGIN_ID);
    return request;
  }

  // TollBit fetching your site's own error page. Serve it straight from your
  // origin so the fallback can never trigger another fallback. The header is part
  // of the cache key, so its value is fixed to keep it to one cache entry.
  if (request.headers[PROBE_HEADER]) {
    request.headers[PROBE_HEADER] = { value: '1' };
    return request;
  }

  // Visitors: if your origin has no page for this URL, CloudFront asks TollBit
  // whether a destination is configured for it.
  if (request.method === 'GET' || request.method === 'HEAD') {
    cf.createRequestOriginGroup({
      originIds: [{ originId: SITE_ORIGIN_ID }, { originId: FALLBACK_ORIGIN_ID }],
      failoverCriteria: { statusCodes: [404] }
    });
  }

  return request;
}
```

The function does not proxy anything. It tells CloudFront which origin to fetch from, so response headers, cookies and large pages pass through untouched, and the path, query string, method and `User-Agent` are forwarded as they are. The **Test** tab only confirms the code runs without error; it does not show the origin change. Click **Publish** when you are done. A function must be published before it can be attached.

#### Update Your Behavior

Go to **Distribution → Behaviors** and edit the behavior your regular traffic routes through. Make all three changes on the one edit page and save once, so the distribution is never half configured.

- **Cache policy**: select the cache policy from the step above.
- **Origin request policy**: the AWS managed `AllViewerExceptHostHeader`. It forwards `User-Agent` and lets CloudFront set `Host` to each origin's own domain. TollBit routes requests by the `tollbit` subdomain in the `Host` header, so requests that arrive with your public hostname instead will fail. A custom policy also works as long as it forwards `User-Agent` and does not forward `Host`.
- **Function associations**: on the **Viewer request** row, choose **CloudFront Function** and select the function you published.

<Callout icon="📘" theme="info">
  ### Note

  The origin request policy applies to all traffic through the behavior, so your own origin also receives its own domain as `Host` rather than your public hostname. Most origins accept this.
</Callout>

# Routing Visitors from Cited Content

Content published through the Agent Site CMS lives on your `tollbit` subdomain and is served to AI agents. When an agent cites one of these pages in an answer, the reader can click that citation to visit your site. Since the page was published for agents, the URL may not exist on your main site. This setup lets TollBit route these visitors to a destination that you configure, such as your home page or the published page itself.

It only applies to URLs that your site has no page for, so existing pages and normal traffic are not affected. The function and the `tollbit-fallback` origin above are all it needs; there is nothing further to deploy.

#### How It Works

1. A visitor requests a URL and your origin responds with a `404`.
2. CloudFront sends the same request, with the same path and query string, to the TollBit fallback origin, `fallback.tollbit.com`. The `X-Tollbit-Host` header on that origin tells TollBit which site the URL belongs to. Requests without it receive a `400`.
3. If TollBit has a destination configured, it responds with a redirect, either to that destination or to the published page on your `tollbit` subdomain, and the visitor follows it.
4. If nothing is configured, TollBit requests the same URL from your site, marked with the `X-Tollbit-Fallback-Probe` header, and returns your site's own `404` page to the visitor. The function serves that marked request directly from your origin, so it cannot loop.

TollBit is only consulted for `404` responses to `GET` and `HEAD` requests from visitors, so no other request does extra work.

#### Caching

Redirect responses from TollBit include `Cache-Control: public, max-age=300`, and `404` responses include `max-age=30`. CloudFront caches them under your site's URL for those durations, subject to your cache policy and your distribution's error caching minimum TTL. Keep this in mind when testing so a cached response is not mistaken for a misconfiguration.

#### Verifying the Setup

Make these requests as a regular visitor, not with the user agent of an AI agent. Requests from AI agents are served from your Agent Site instead, so they do not receive the redirect. The commands below use curl's own user agent, which counts as a visitor.

Request an agent-only URL that has a configured destination and confirm you receive the redirect. Then request a URL your site has no page for, such as a made-up path, and confirm your site's normal error page is served with a `404` status.

```shell
# Expect a redirect status and a Location header
curl -sI https://www.example.com/agent-only-page

# Expect 404, with your site's own error page as the body
curl -s -o /dev/null -w "%{http_code}\n" https://www.example.com/made-up-path
```

Contact [team@tollbit.com](mailto:team@tollbit.com) to have us configure an agent-only test URL and destination for your property.
