---
title: Akamai
excerpt: Learn how to integrate TollBit with Akamai.
deprecated: false
hidden: false
metadata:
  robots: index
---
We provide a way for all Akamai customers to stream logs to our platform, as well as configure agent site.

# Steps for Analytics

###

### Create a Stream with DataStream 2

You will need to first create a stream by going to your <Anchor target="_blank" href="https://control.akamai.com/apps/data-stream-ui/#/streams/group/all">Akamai Control Center</Anchor>. Follow <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/create-stream">these instructions</Anchor> on how to create your stream.

**Choose Data Parameters**

When choosing <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/choose-data-parameters">data parameters</Anchor>, make sure to parameters that cover at least everything in following sample log JSON. Also, please ensure that your log format is JSON.

```json
{
  "version": 1,
  "streamId": "12345",
  "cp": "123456",
  "reqId": "1239f220",
  "reqTimeSec": "1573840000",
  "bytes": "4995",
  "cliIP": "128.147.28.68",
  "statusCode": "206",
  "proto": "HTTPS",
  "reqHost": "test.hostname.net",
  "reqMethod": "GET",
  "reqPath": "/path1/path2/file.ext",
  "reqPort": "443",
  "rspContentLen": "5000",
  "rspContentType": "text/html",
  "UA": "Mozilla%2F5.0+%28Macintosh%3B+Intel+Mac+OS+X+10_14_3%29",
  "tlsOverheadTimeMSec": "0",
  "tlsVersion": "TLSv1",
  "objSize": "484",
  "uncompressedSize": "484",
  "overheadBytes": "232",
  "totalBytes": "0",
  "queryStr": "param=value",
  "breadcrumbs": "//BC/%5Ba=23.33.41.20,c=g,k=0,l=1%5D",
  "accLang": "en-US",
  "cookie": "cookie-content",
  "range": "37334-42356",
  "referer": "https%3A%2F%2Ftest.referrer.net%2Fen-US%2Fdocs%2FWeb%2Ftest",
  "xForwardedFor": "8.47.28.38",
  "maxAgeSec": "3600",
  "reqEndTimeMSec": "3",
  "errorCode": "ERR_ACCESS_DENIED|fwd_acl",
  "turnAroundTimeMSec": "11",
  "transferTimeMSec": "125",
  "dnsLookupTimeMSec": "50",
  "lastByte": "1",
  "country": "IN",
  "state": "Virginia",
  "city": "HERNDON"
}
```

**Stream to Endpoint**

To forward your logs to us, follow the steps outlined <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/stream-custom-https">here</Anchor>. The endpoint url that you should be streaming to is `https://log.tollbit.com/log/akamai`.

Select none for authentication for now, as we will be setting up custom authentication. To do so, go to "Custom header". For the content type, you can select `application/json`. Add a new header value with the key `TollbitKey` and the value as your secret key from your <Anchor target="_blank" href="https://app.tollbit.com">dashboard</Anchor>.

Finally, you can <Anchor target="_blank" href="https://techdocs.akamai.com/datastream2/docs/review-activate-stream">review and activate</Anchor> your stream!

# Steps for Agent Site

Akamai allows you to set up redirection rules at the edge using either Cloudlets or Content Protector.

- For Cloudlets, please see the steps outlined below.
- For Content Protector, please reach out to your Akamai account team and loop in [team@tollbit.com](mailto:team@tollbit.com) as well.

**Cloudlets Set Up**

To set up agent sites for Akamai with Cloudlets, you would use their Forward Rewrite Cloudlet. Follow the documentation <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/what-forward-rewrite">here</Anchor> for reference.&#x20;

<Callout icon="📘" theme="info">
  ### Pro Tip

  If you are previously following our docs and have set up Agent Site via **Edge Redirector** to forward bot traffic to your `tollbit` subdomain, you would replace the Edge Redirector with the Forward Rewrite Cloudlet.
</Callout>

**Setup New Origin<br />**&#x59;ou’ll need to set up a new conditional origin that is your `tollbit` subdomain to have the forward rewrite cloudlet correctly route the request. Follow the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/about-conditional-origins#set-up-a-conditional-origin-definition-rule">here</Anchor> on doing this.&#x20;

At a high level, you would go to Property Manager and create a new Conditional Origin with the server domain name url as your `tollbit` subdomain (i.e. tollbit.example.com). Ensure that this origin is activated and deployed.

**Forward Rewrite Cloudlet**<br />Create a new forward rewrite policy by following the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/create-forward-rewrite-policy">here</Anchor>. Then create a forward rewrite rule following the docs <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/add-forward-rewrite-rule">here</Anchor>. For the match type, you want to match on if the request header’s User-Agent header contains one of the following user agents (case insensitive):

```
'Amazonbot', 'Amzn-SearchBot', 'anthropic-ai', 'Bytespider', 'CCBot',  'ChatGPT-User', 'claude-code', 'Claude-SearchBot', 'Claude-User',  'Claude-Web', 'ClaudeBot', 'cohere-ai', 'Diffbot', 'ExaBot', 'Exabot',  'GPTBot', 'meta-externalagent', 'Meta-Webindexer', 'OAI-AdsBot',  'OAI-SearchBot', 'Perplexity-User', 'PerplexityBot', 'Timpibot', 'YouBot'
```

Ensure that the rewrite points to the new `tollbit` subdomain origin you added, while preserving the original URL path and query strings.

Once you’ve tested this appropriately, you can activate and deploy.

<Callout icon="📘" theme="info">
  ### Pro Tip

  Cloudlets Policy Manager evaluates rules from top to bottom, and picks the first rule that matches. If you have other Cloudlets with rules that also intercept requests, they may match before the rule you just added.
</Callout>

<br />

<br />