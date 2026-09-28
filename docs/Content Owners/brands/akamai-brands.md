---
title: Akamai for Brands
excerpt: >-
  Set up TollBit analytics, Agent Site, and visitor routing from cited content
  for a brand site served through Akamai.
deprecated: false
hidden: true
metadata:
  robots: noindex
---
This guide covers setting up TollBit for a site served through Akamai: sending logs to our platform for analytics, setting up Agent Site, and routing visitors from cited content.

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

Add these addresses to an IP network list in Akamai. Then make sure that list is allowed through every security layer that could block or challenge our requests. This applies to every hostname you onboard with us:

- **App & API Protector / Kona Site Defender**: IP/Geo Firewall, WAF rule exceptions, and rate controls.
- **Bot Manager**, if enabled: make sure requests from these IPs are not denied, challenged, tarpitted, or served alternate content.
- **Any firewall or WAF in front of your origin servers**, outside of Akamai.

If you are unsure whether your configuration blocks us, contact [team@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

# Steps for Analytics

Send us your Akamai logs with DataStream 2. There are two ways to set it up, and which one you use depends on how your sites are arranged in Akamai properties.

| Your Akamai setup                  | How to send logs                                                |
| :--------------------------------- | :-------------------------------------------------------------- |
| Each site has its own property     | Option 1: stream directly to TollBit                            |
| One property serves multiple sites | Option 2: stream to an Amazon S3 bucket that TollBit reads from |

#### Data Parameters

Both options use the same data parameters. When <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/choose-data-parameters">choosing data parameters</Anchor> for your stream, include at least the fields shown in the following sample log JSON. The client IP (`cliIP`) is optional, but including it gives you better analytics. Also, please make sure your log format is JSON.

```json
{
  "reqTimeSec": "1573840000",
  "cliIP": "128.147.28.68",
  "statusCode": "206",
  "proto": "HTTPS",
  "reqHost": "test.hostname.net",
  "reqMethod": "GET",
  "reqPath": "/path1/path2/file.ext",
  "queryStr": "param=value",
  "UA": "Mozilla%2F5.0+%28Macintosh%3B+Intel+Mac+OS+X+10_14_3%29",
  "referer": "https%3A%2F%2Ftest.referrer.net%2Fen-US%2Fdocs%2FWeb%2Ftest"
}
```

## Option 1: Stream Directly to TollBit

Use this option when each site has its own property.

#### 1. Create the stream

In <Anchor target="_blank" href="https://control.akamai.com/">Akamai Control Center</Anchor>, <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/create-stream">create a stream</Anchor> and add the properties you are onboarding with TollBit. Include the fields listed under Data Parameters above, and set the log format to JSON.

#### 2. Stream to the TollBit endpoint

For the destination, follow Akamai's <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/stream-custom-https">custom HTTPS endpoint instructions</Anchor> with these settings:

- **Endpoint URL**: `https://log.tollbit.com/log/akamai`
- **Authentication**: `None`. Authentication is handled by the custom header below.
- **Content type**: `application/json`
- **Custom header**: name `TollbitKey`, with the value set to your organization's secret key from your <Anchor target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor>.

The secret key belongs to your TollBit organization, so one stream covers the properties of one organization. If you have more than one organization, create a stream for each, with that organization's properties and secret key.

#### 3. Activate the stream

<Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/review-activate-stream">Review and activate the stream</Anchor>, and <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/enable-datastream-behavior">enable the DataStream behavior</Anchor> in each property included in the stream.

## Option 2: Stream to Amazon S3

Use this option when one property serves multiple sites. One stream can deliver logs for all of your properties to a single bucket, and onboarding another property later does not need a new stream.

If you already stream DataStream 2 logs to S3, you may be able to reuse that stream. It must use the JSON log format and include the fields listed under Data Parameters above. If it doesn't, create a new stream as described here.

#### 1. Create or choose an S3 bucket

Create a bucket for the logs, or choose an existing one. We recommend a dedicated bucket, or at least a dedicated folder, so that TollBit only has access to the logs meant for us.

#### 2. Create the stream

In <Anchor target="_blank" href="https://control.akamai.com/">Akamai Control Center</Anchor>, <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/create-stream">create a stream</Anchor> and add every property you are onboarding with TollBit. Then:

- **Data parameters and format**: include the fields listed under Data Parameters above, and set the log format to JSON.
- **Destination**: follow Akamai's <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/stream-amazon-s3">Amazon S3 destination instructions</Anchor>. Akamai needs an access key for an IAM user or role that has `s3:PutObject`, `s3:GetObject`, and `s3:ListBucket` on the bucket.
- **Folder path**: use dynamic variables so logs land in dated subfolders, as described below.

**Folder path: dated subfolders**

Set the stream's Folder path using Akamai's <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/dynamic-time-variables">dynamic time variables</Anchor>, so that each day's logs go into their own subfolder named by year, month, and day, in that order. For example, with a bucket named `example-bucket`, set the folder path to:

```text
logs/{%Y/%m/%d}
```

Logs for September 17, 2026 are then written to:

```text
s3://example-bucket/logs/2026/09/17/<logfile>
```

Keep the slashes inside the braces (`{%Y/%m/%d}`) so that year, month, and day become separate nested folders. Writing `{%Y}{%m}{%d}` instead produces a single folder such as `20260917`. You can use any bucket name and base folder (`logs` above), but keep the `/YYYY/MM/DD/` structure after it, and leave the file name prefix and suffix at their defaults.

Then <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/review-activate-stream">review and activate the stream</Anchor>, and <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/enable-datastream-behavior">enable the DataStream behavior</Anchor> in each property included in the stream.

#### 3. Grant TollBit read access

Add the following statement to your bucket policy, replacing `YOUR-BUCKET-NAME`:

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

Set the bucket's **Object Ownership** to **Bucket owner enforced**, which disables ACLs. Access must be granted only through the bucket policy above, so don't use ACLs on this bucket.

If the bucket is encrypted with a customer-managed KMS key, the key policy must also allow the TollBit accounts above to use `kms:Decrypt`.

#### 4. Send us the details

Email [team@tollbit.com](mailto:team@tollbit.com) with:

- The bucket name, its region, and the folder path (e.g. `logs/{%Y/%m/%d}`)
- The hostnames in the stream. If you have more than one TollBit organization, include which organization each hostname belongs to.

We will confirm once logs are being ingested.

# Steps for Agent Site

Akamai lets you set up rewrite rules at the edge using Cloudlets. Please see the steps outlined below.

<Callout icon="📘" theme="info">
  ### Test on Staging First

  Both the Forward Rewrite Cloudlet and the visitor routing setup below change how requests are routed at the edge. Where possible, activate and test each change on Akamai's Staging network before activating to Production, so you can confirm the behavior with test requests before it affects live visitors.
</Callout>

#### Cloudlets Setup

To set up Agent Site on Akamai with Cloudlets, use the Forward Rewrite Cloudlet. Follow the documentation <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/what-is-forward-rewrite">here</Anchor> for reference.

#### Set Up a New Origin

Your `tollbit` subdomain is set up as part of onboarding your site to the TollBit platform, so complete that first. You'll need to set up a new conditional origin for your `tollbit` subdomain so the Forward Rewrite Cloudlet can route requests to it. Follow the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/about-conditional-origins">here</Anchor>.

At a high level, you would go to Property Manager and create a new Conditional Origin whose origin server hostname is your `tollbit` subdomain (e.g. `tollbit.example.com`). In the origin's settings, set **Forward Host Header** to **Origin Hostname**. TollBit routes agent requests by the `tollbit` subdomain in the Host header, so rewritten requests must arrive with the Host header set to `tollbit.example.com` rather than your site's incoming hostname. If the incoming hostname is forwarded instead, TollBit cannot tell which site the request is for and the request will fail. Ensure that this origin is activated and deployed.

#### Forward Rewrite Cloudlet

Create a new forward rewrite policy by following the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/create-forward-rewrite-policy">here</Anchor>. Then create a forward rewrite rule following the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/add-forward-rewrite-rule">here</Anchor>. For the match type, match on the `User-Agent` request header containing any of the following user agents (case-insensitive):

```text
Amazonbot
Amzn-SearchBot
anthropic-ai
Bytespider
CCBot
ChatGPT-User
claude-code
Claude-SearchBot
Claude-User
Claude-Web
ClaudeBot
cohere-ai
Diffbot
Exabot
GPTBot
meta-externalagent
Meta-Webindexer
OAI-AdsBot
OAI-SearchBot
Perplexity-User
PerplexityBot
Shap-User
ShapBot
Timpibot
YouBot
```

Ensure that the rewrite points to the `tollbit` subdomain origin you created above, and that the original URL path and query string are preserved. Each request must be forwarded to the same path on your `tollbit` subdomain, not to a single fixed URL.

<Callout icon="📘" theme="info">
  ### Pro Tip

  Cloudlets Policy Manager evaluates rules from top to bottom, and picks the first rule that matches. If you have other Cloudlets with rules that also intercept requests, they may match before the rule you just added.
</Callout>

Once the rule is in place, activate the policy, on Akamai's Staging network first where possible.

# Routing Visitors from Cited Content

Content published through the Agent Site CMS lives on your `tollbit` subdomain and is served to AI agents. When an agent cites one of these pages in an answer, the reader can click that citation to visit your site. Since the page was published for agents, the URL may not exist on your main site. This setup lets TollBit route these visitors to a destination that you configure, such as your home page or the published page itself.

This only applies to URLs that your site has no page for, so existing pages and normal traffic are not affected. If TollBit has nothing configured for a URL, or TollBit cannot be reached, your site serves its own error page, exactly as it does today. The setup uses Akamai EdgeWorkers.

#### How It Works

1. A visitor requests a URL and your origin responds with a `404`.
2. An EdgeWorker asks TollBit whether anything is configured for that URL. The request is sent, with the same path and query string and your site's hostname in the `X-Tollbit-Host` header, to a fallback hostname you own that forwards to the TollBit fallback origin, `fallback.tollbit.com`.
3. If TollBit has a destination configured, it responds with a redirect, either to that destination or to the published page on your `tollbit` subdomain. The EdgeWorker turns the `404` into that redirect and the visitor follows it.
4. If TollBit responds with a `404`, times out, or fails in any way, the response is left untouched and your origin's own error page is served.

TollBit is only consulted for `404` responses to `GET` and `HEAD` requests that reach your origin, so no other request does extra work.

#### Fallback Hostname

Akamai only allows an EdgeWorker to make requests to hostnames served by Akamai, so the request goes to a hostname you own that is set up on Akamai for this purpose. Any hostname works, for example `tollbit-fallback.example.com`. Give it its own property, with an edge hostname and certificate, whose <Anchor target="_blank" href="https://techdocs.akamai.com/property-mgr/docs/origin-server">Origin Server</Anchor> is `fallback.tollbit.com` over HTTPS with **Forward Host Header** set to **Origin Hostname**. No other behaviors are needed: the `X-Tollbit-Host` header set by the EdgeWorker is passed through to TollBit, which uses it to tell which site the URL belongs to. Requests without it receive a `400`.

Because these requests are handled by a separate property, they never touch your main site's property, so they do not appear in its DataStream logs or interact with its other behaviors.

#### EdgeWorker Setup

You will need to write and deploy an EdgeWorker on your main property that runs on the origin response and does the following:

- Acts only when the origin response is a `404` to a `GET` or `HEAD` request.
- Makes one sub-request to the same path and query string on your fallback hostname, adding the `X-Tollbit-Host` header with your site's hostname (without a port), with a short timeout (one to two seconds).
- If the sub-request returns a redirect, sets the same redirect status and `Location` header on the response.
- On any other result, including a timeout or error, leaves the response untouched.

See Akamai's <Anchor target="_blank" href="https://techdocs.akamai.com/edgeworkers/docs">EdgeWorkers documentation</Anchor> for creating and activating an EdgeWorker. In your main property, add a rule matching Request Method `GET` or `HEAD` with the <Anchor target="_blank" href="https://techdocs.akamai.com/edgeworkers/docs/add-the-edgeworkers-behavior">EdgeWorkers behavior</Anchor> for it.

Activate the fallback property, the EdgeWorker, and the main property version on Staging, verify as described below, then activate them on Production.

#### Caching

Redirect responses from TollBit include `Cache-Control: public, max-age=300`, and `404` responses for URLs with nothing configured include `max-age=30`. How long Akamai holds these depends on the fallback property's caching rules. Separately, if <Anchor target="_blank" href="https://techdocs.akamai.com/property-mgr/docs/cache-http-err-responses">Cache HTTP Error Responses</Anchor> is enabled on your main property, a `404` and the redirect derived from it may be cached under your site's URL for that setting's TTL. Keep this in mind when testing so a cached response is not mistaken for a misconfiguration.

#### Verifying the Setup

Make these requests as a regular visitor, not with the user agent of an AI agent. Requests from AI agents are served from your Agent Site instead, so they do not receive the redirect.

Request an agent-only URL that has a configured destination and confirm you receive the redirect. Then request a URL your site has no page for (e.g. a made-up path) and confirm your site's normal error page is served. Test on Staging first, then repeat on Production once activated.

Contact [team@tollbit.com](mailto:team@tollbit.com) to have us configure an agent-only test URL and destination for your property.
