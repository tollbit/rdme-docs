---
title: Akamai
excerpt: Learn how to integrate TollBit with Akamai.
---
<br />

We provide a way for all Akamai customers to stream logs to our platform.

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

### Steps for Agent Site

Akamai allows you to set up redirection rules at the edge using either Cloudlets or Content Protector.

- For Cloudlets, please see the steps outlined below.
- For Content Protector, please reach out to your Akamai account team and loop in [team@tollbit.com](mailto:team@tollbit.com) as well.

**Cloudlets set up**

Akamai provides <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/what-edge-redirector">Edge Redirector</Anchor> Cloudlets that help you manage redirection using certain matching rules.

We want to first start by creating an Edge Redirector policy. Follow the documentation <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/create-edge-redirector-policy">here</Anchor> to do so in accordance with how your Akamai instance is set up.

Once you have set up your policy, follow the documentation <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/add-edge-redirector-rules">here</Anchor> to set up rules for your Edge Redirector. Because we want to be redirecting based on the `User-Agent` header, we will need to create a redirector with advance matching rules. You will want to create a match type based on the <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/req-header">request header</Anchor>. The name of the header should be `User-Agent`, and the value should be a tab separated list of bad user agents. You can use the following list:

```
ChatGPT-User PerplexityBot GPTBot anthropic-ai CCBot Claude-Web ClaudeBot cohere-ai YouBot Diffbot OAI-SearchBot meta-externalagent Timpibot Amazonbot Bytespider Perplexity-User
```

For the operator value, use `is one of` without case sensitivity. These settings should let you match our known bad users agents. In the redirection rule, you can set the redirect url to your `tollbit` subdomain.
Ensure that the path is preserved in the redirect. Akamai has an example of this in their <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/review-available-match-types-edge-redirector#edge-redirector-regular-expression-example">docs</Anchor>
and we should be able to follow it by setting the redirect url to `https://tollbit.<your_site>/\2`. The `\2` should preserve the path.

> 📘 Pro Tip
>
> Cloudlets Policy Manager evaluates rules from top to bottom, and picks the
> first rule that matches. If you have other Cloudlets with rules that also
> intercept requests, they may match before the rule you just added.

Click save rule to save your changes, and you should be ready to activate! Follow the steps <Anchor target="_blank" href="https://techdocs.akamai.com/cloudlets/docs/activate-cloudlets-beh-prop-manager">here</Anchor> to do so.

> 🚧 Note
>
> The rule you just added **will intercept** and potentially **redirect**
> traffic to your main site. Please ensure that you have tested this in a test
> environment or for a small subset of pages before activating this across your
> entire site.

<br />
