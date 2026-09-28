---
title: Fastly for Brands
excerpt: >-
  Set up TollBit analytics, Agent Site, and visitor routing from cited content
  for a brand site served through Fastly.
deprecated: false
hidden: true
metadata:
  robots: noindex
---
This guide covers setting up TollBit for a site served through Fastly: streaming logs to our platform for analytics, setting up Agent Site, and routing visitors from cited content. Everything is created in your own Fastly service: two hosts, one logging endpoint, and three dynamic VCL snippets. It replaces the publisher-oriented Fastly instructions for your integration.

| What you create | Name | Used for |
| :--- | :--- | :--- |
| Host | `tollbit_origin` | Agent Site |
| Host | `tollbit_fallback_origin` | Routing visitors from cited content |
| HTTPS logging endpoint | Any, e.g. `tollbit-prod` | Analytics |
| Dynamic VCL snippet, type `recv` | `tollbit_recv_dynamic_snippet` | Agent Site |
| Dynamic VCL snippet, type `recv` | `tollbit_fallback_recv_snippet` | Routing visitors from cited content |
| Dynamic VCL snippet, type `deliver` | `tollbit_fallback_deliver_snippet` | Routing visitors from cited content |

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

- **Fastly**: ACLs, rate limiting, the Next-Gen WAF, and any custom VCL that blocks or challenges requests.
- **Any firewall or WAF in front of your origin servers**, outside of Fastly.

If you are unsure whether your configuration blocks us, contact [team@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

#### Clone Your Active Version

Go to the **Deliver** tab and select the service for your site. Click **Edit configuration** and choose to clone your current active version. This saves a new version as a draft, and lets you roll back if necessary. Make all of the changes in this guide on that draft, then activate it once at the end.

![Fastly Edit Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-edit-configuration.png)

<Callout icon="📘" theme="info">
  ### Test Before Production

  The snippets below change how requests are routed at the edge. Where possible, apply them to a staging service first and confirm the results with the test requests in Verifying the Setup before activating them on the service that serves your live traffic.
</Callout>

<Callout icon="📘" theme="info">
  ### Existing VCL

  The snippets here are written for a service with no other request routing logic. They use Fastly's `restart`, so if your service already uses restarts or shielding, or has VCL that changes the backend or the `Host` header, contact [team@tollbit.com](mailto:team@tollbit.com) before activating and we will work through it with you.
</Callout>

# Steps for Analytics

Analytics is powered by a log streaming endpoint on your own Fastly service.

#### Create the Logging Endpoint

In your draft version, scroll down the sidebar to **Logging** and click **Create Endpoint**.

![Fastly Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-sidebar.png)

Find the **HTTPS** logging endpoint and click **Create endpoint**.

![Fastly Http Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-http-config.png)

Set the name to anything descriptive, such as `tollbit-prod`, and keep the placement option as the default. Make sure your log format is exactly as follows, without extra trailing spaces or newlines:

```json
{ "timestamp": "%{strftime(\{"%Y-%m-%dT%H:%M:%S%z"\}, time.start)}V", "ip_address": "%{req.http.Fastly-Client-IP}V", "geo_country": "%{client.geo.country_name}V", "geo_city": "%{client.geo.city}V", "geo_postal_code":"%{client.geo.postal_code}V", "geo_latitude":"%{client.geo.latitude}V", "geo_longitude":"%{client.geo.longitude}V", "host": "%{if(req.http.Fastly-Orig-Host, req.http.Fastly-Orig-Host, req.http.Host)}V", "url": "%{json.escape(req.url)}V", "request_method": "%{json.escape(req.method)}V", "request_protocol": "%{json.escape(req.proto)}V", "request_referer": "%{json.escape(req.http.referer)}V", "request_user_agent": "%{json.escape(req.http.User-Agent)}V", "request_latency":"%{time.elapsed.usec}V", "response_state": "%{json.escape(fastly_info.state)}V", "response_status": %{std.itoa(resp.status)}V, "response_reason": %{if(resp.response, "%22"+json.escape(resp.response)+"%22", "null")}V, "response_body_size": %{resp.body_bytes_written}V, "fastly_server": "%{json.escape(server.identity)}V", "fastly_is_edge": %{if(fastly.ff.visits_this_service == 0, "true", "false")}V, "signature": "%{json.escape(req.http.signature)}V", "signature_agent": "%{json.escape(req.http.signature-agent)}V", "signature_input": "%{json.escape(req.http.signature-input)}V" }
```

Set the URL to `https://log.tollbit.com/log`.

![Fastly Log Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-log-config.png)

#### Authenticate Your Logs

Open **Advanced options** and set **Custom header name** to `TollbitKey`. Set **Custom header value** to the secret key from the API key tab of your <Anchor target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor>, with no trailing spaces. Keep all the other settings as default, scroll to the bottom, and save.

<Callout icon="🚧" theme="warn">
  ### Note

  If the `TollbitKey` header is missing or wrong, your logs are rejected and no analytics data appears. If your analytics charts stay empty, check this header first.
</Callout>

# Steps for Agent Site

AI agents that request your site are served from your Agent Site on your `tollbit` subdomain, while visitors continue to reach your own origin. A dynamic VCL snippet matches the `User-Agent` header and sends those requests through the `tollbit_origin` host.

#### Add the Agent Site Host

Your `tollbit` subdomain is set up as part of onboarding your site to the TollBit platform, so complete that first. In your draft version, go to **Origins** in the sidebar and create a new host, using your `tollbit` subdomain (e.g. `tollbit.example.com`) as the address.

You may see a warning that this host is unused. That is expected. The snippet below is what sends requests to it.

![](https://files.readme.io/1590d22a507a3fbd4af45aee2e88356de590545284a199a607ed83849baf61f9-image6.png)

Once it has been added, click the pencil icon next to the host to edit it:

- **Name**: exactly `tollbit_origin`. The snippets refer to it by this name.
- **TLS**: enabled, on port `443`.
- **Auto load balance**: `No`, so that only the snippet sends requests to this host.

Scroll down and click **Update** to save.

![](https://files.readme.io/0cfa040e63a3b941f453e5810ed05c842579e955a1d77af966e089893a14cbb1-image1.png)

#### Add the Agent Site Snippet

Go to **VCL snippets** in the sidebar and create a snippet:

- **Name**: `tollbit_recv_dynamic_snippet`
- **Type**: **Dynamic**
- **Placement**: within subroutine, `recv`

![](https://files.readme.io/39bc1086ee50031cf99b86bb1ba70e4cde335bf3c219f9a3c6b635a154ecb39c-Screenshot_2026-07-23_at_5.30.14_PM.png)

Paste in the following VCL:

```vcl
if (req.http.user-agent ~ "(?i)amazonbot|amzn-searchbot|anthropic-ai|bytespider|ccbot|chatgpt-user|claude-code|claude-searchbot|claude-user|claude-web|claudebot|cohere-ai|diffbot|exabot|gptbot|meta-externalagent|meta-webindexer|oai-adsbot|oai-searchbot|perplexity-user|perplexitybot|shapbot|shap-user|timpibot|youbot") {
  set req.backend = F_tollbit_origin;
  set req.http.Fastly-Orig-Host = req.http.host;
  if (std.prefixof(req.http.host, "www.")) {
    set req.http.host = std.replace_prefix(req.http.host, "www.", "tollbit.");
  } else {
    set req.http.host = "tollbit." + req.http.host;
  }
  return(pass);
}
```

Edit the user agent list to control which AI agents are routed to your Agent Site.

# Routing Visitors from Cited Content

Content published through the Agent Site CMS lives on your `tollbit` subdomain and is served to AI agents. When an agent cites one of these pages in an answer, the reader can click that citation to visit your site. Since the page was published for agents, the URL may not exist on your main site. This setup lets TollBit route these visitors to a destination that you configure, such as your home page or the published page itself.

This only applies to URLs that your site has no page for, so existing pages and normal traffic are not affected. If TollBit has nothing configured for a URL, or TollBit cannot be reached, your site serves its own error page, exactly as it does today.

#### How It Works

1. A visitor requests a URL and your origin responds with a `404`.
2. Fastly sends the same request, with the same path and query string, to `fallback.tollbit.com`. Your site's hostname is sent in the `X-Tollbit-Host` header, which tells TollBit which site the URL belongs to. Requests without it receive a `400`.
3. If TollBit has a destination configured, it responds with a redirect, either to that destination or to the published page on your `tollbit` subdomain, and the visitor follows it.
4. If TollBit responds with a `404`, times out, or fails in any way, Fastly requests the URL from your origin again and your origin's own error page is served.

TollBit is only consulted for `404` responses to `GET` and `HEAD` requests from visitors, so no other request does extra work. Requests from AI agents that were routed to your Agent Site are excluded.

#### Add the Lookup Host

In your draft version, go to **Origins** and create a second host with `fallback.tollbit.com` as the address. Then edit it:

- **Name**: exactly `tollbit_fallback_origin`. The snippets refer to it by this name.
- **TLS**: enabled, on port `443`, with `fallback.tollbit.com` as the certificate hostname and SNI hostname.
- **Auto load balance**: `No`, so that only the snippets send requests to this host.
- **First byte timeout**: `2000` milliseconds, so that a slow lookup does not hold up your own error page.

#### Add the Lookup Snippets

Go to **VCL snippets** and create two more snippets. Both are **Dynamic**.

The first is named `tollbit_fallback_recv_snippet`, placed within subroutine `recv`:

```vcl
if (req.restarts == 0) {
  unset req.http.X-Tollbit-Host;
  unset req.http.X-Tollbit-Fallback-Host;
  unset req.http.X-Tollbit-Skip-Fallback;
}

if (req.http.X-Tollbit-Fallback-Host && req.http.X-Tollbit-Skip-Fallback) {
  set req.http.host = req.http.X-Tollbit-Fallback-Host;
  unset req.http.X-Tollbit-Host;
} elsif (req.http.X-Tollbit-Fallback-Host) {
  set req.backend = F_tollbit_fallback_origin;
  set req.http.X-Tollbit-Host = req.http.X-Tollbit-Fallback-Host;
  set req.http.Fastly-Orig-Host = req.http.X-Tollbit-Fallback-Host;
  set req.http.host = "fallback.tollbit.com";
  return(pass);
}
```

The second is named `tollbit_fallback_deliver_snippet`, placed within subroutine `deliver`:

```vcl
# Origin 404 on a GET/HEAD: restart and let the recv snippet retry it against
# the TollBit fallback. Bot traffic already forwarded to the tollbit origin
# is excluded.
if (resp.status == 404 && req.restarts == 0 && (req.method == "GET" || req.method == "HEAD") && req.backend != F_tollbit_origin) {
  set req.http.X-Tollbit-Fallback-Host = req.http.host;
  restart;
}

# The fallback had nothing configured (404) or errored (5xx): restart once
# more back to the default origin so its own error page is served.
if (resp.status >= 400 && req.restarts == 1 && req.http.X-Tollbit-Fallback-Host && !req.http.X-Tollbit-Skip-Fallback) {
  set req.http.X-Tollbit-Skip-Fallback = "1";
  restart;
}
```

The `X-Tollbit-Fallback-Host` and `X-Tollbit-Skip-Fallback` headers are only used inside Fastly to keep track of the lookup. They are removed from incoming requests, so a visitor cannot set them, and they are not sent to your origin's visitors.

#### Caching

Lookups are sent with `pass`, so Fastly does not cache TollBit's responses. Redirect responses from TollBit include `Cache-Control: public, max-age=300`, so a visitor's browser may reuse a redirect for up to five minutes. Keep this in mind when testing so a cached response is not mistaken for a misconfiguration.

# Activate

Once the hosts, the logging endpoint, and the three snippets are in your draft version, click **Activate**. Keep in mind that if the version has other unpublished changes, this publishes those as well.

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)

<Callout icon="📘" theme="info">
  ### Note

  Once the snippets exist on your active version, later edits to a dynamic snippet's content take effect immediately. There is no draft and activate step for content changes, so you can update the user agent list without publishing a new version.
</Callout>

# Verifying the Setup

#### Agent Site

Every response served by TollBit includes the response header `X-Edge-State: active`. Request a page on your site with one of the user agents in the list and confirm that header is present:

```shell
curl -sI -A "GPTBot" https://www.example.com/some-article | grep -i x-edge-state
```

#### Routing Visitors from Cited Content

Request an agent-only URL that has a configured destination and confirm you receive the redirect. Then request a URL your site has no page for, such as a made-up path, and confirm your site's normal error page is served with a `404` status.

```shell
# Expect a redirect status and a Location header
curl -sI https://www.example.com/agent-only-page

# Expect 404, with your site's own error page as the body
curl -s -o /dev/null -w "%{http_code}\n" https://www.example.com/made-up-path
```

Contact [team@tollbit.com](mailto:team@tollbit.com) to have us configure an agent-only test URL and destination for your property.

#### Analytics

Charts in your TollBit dashboard are updated once a day, so allow up to 24 hours after activating.
