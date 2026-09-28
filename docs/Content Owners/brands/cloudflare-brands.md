---
title: Cloudflare for Brands
excerpt: >-
  Set up TollBit analytics, Agent Site, and visitor routing from cited content
  for a brand site served through Cloudflare.
deprecated: false
hidden: true
metadata:
  robots: noindex
---
This guide covers setting up TollBit for a site served through Cloudflare: sending logs to our platform for analytics, setting up Agent Site, and routing visitors from cited content. Agent Site and visitor routing are handled by a single Cloudflare Worker, which works the same way on every Cloudflare plan. Analytics comes either from that same Worker or, on the Enterprise plan, from Logpush.

# Before You Start

#### Proxy Your Site Through Cloudflare

The Worker only runs on traffic that is proxied through Cloudflare. On your site's DNS page, make sure the records for your site have the proxy status `Proxied`.

![Cloudflare Proxied](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-proxied.png)

#### Set SSL / TLS to Full (Strict)

Go to the **SSL/TLS** tab, open the overview page, and click **Configure**. Choose **Full (strict)**, so that Cloudflare fetches from your origin over HTTPS. Before doing this, make sure your origin server accepts HTTPS requests. Most do, unless you are running custom or legacy servers.

![](https://files.readme.io/992ff8dd9ad7ea3acc89cfe0f0013d4e0b6d2be296b09b74bdeeb4edd6d7c0eb-image12.png)

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

Allow these addresses in Cloudflare, and make sure they are allowed through every security layer that could block or challenge our requests. This applies to every hostname you onboard with us:

- **WAF**: custom rules, managed rules, rate limiting rules, and your Security Level.
- **Bot protection**, if enabled: make sure requests from these IPs are not blocked or challenged by Bot Fight Mode, Super Bot Fight Mode, or Bot Management.
- **Any firewall or WAF in front of your origin servers**, outside of Cloudflare.

If you are unsure whether your configuration blocks us, contact [team@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

# Steps for Analytics

There are two ways to send your logs to TollBit. Choose one.

|                            | Logpush                                                              | Worker                                           |
| :------------------------- | :------------------------------------------------------------------- | :----------------------------------------------- |
| Cloudflare plan            | Enterprise only                                                      | Any plan                                         |
| How logs reach TollBit     | Cloudflare delivers them to a storage bucket that TollBit reads from | The Worker forwards them to TollBit directly     |
| What is logged             | Every request to your site                                           | Requests on the routes the Worker is attached to |
| Setting in the Worker code | `FORWARD_LOGS = false`                                               | `FORWARD_LOGS = true`                            |

<br />

<Callout icon="🚧" theme="warn">
  ### Use only one analytics setup method

  If Logpush is set up and the Worker also forwards logs, your traffic is counted twice.
</Callout>

## Option 1: Logpush (Enterprise)

On the Enterprise plan you have access to Cloudflare's <Anchor target="_blank" href="https://developers.cloudflare.com/logs/about/">Logpush</Anchor> feature, which delivers HTTP request logs for your site to an S3, R2 or GCS bucket. If you already push logs to one of these, we can ingest them from where they are stored.

Make sure the following are included in the logs:

- The `ClientIP` field, enabled in the **Fields** step of your Logpush settings. It is optional, but including it gives you better analytics.
- The `location` response header, and the `signature-agent`, `signature-input` and `signature` request headers. Follow <Anchor target="_blank" href="https://developers.cloudflare.com/logs/reference/custom-fields/#enable-custom-fields-via-dashboard">these steps</Anchor> in Cloudflare's documentation to add them as custom fields. Select **Response Header** as the field type for `location`, and **Request Header** for the other three.

If the logs are delivered to an S3 bucket, add the following policy to the bucket so TollBit can read them:

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

Then send [team@tollbit.com](mailto:team@tollbit.com) the bucket name and the path your logs are written to, so we can finish the analytics setup. If your logs are delivered to R2 or GCS, contact us and we will set up access with you.

When you add the Worker code below, set `FORWARD_LOGS` to `false`.

## Option 2: Worker

The Worker below forwards logs to TollBit by default, so there is nothing to set up here. When you add the Worker code, replace `YOUR_SECRET_KEY_HERE` with the secret key from your <Anchor target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor>.

# Set Up the Worker

<Callout icon="📘" theme="info">
  ### Test Before Production

  The Worker changes how requests are routed at the edge. Where possible, attach it to a staging hostname first, or to a route that covers a small part of your site, and confirm the results with the test requests in Verifying the Setup before attaching it to the route that serves your live traffic.
</Callout>

<Callout icon="📘" theme="info">
  ### One Worker Per Route

  Cloudflare runs only one Worker per route. If a Worker already handles requests for your site, add this code to that Worker instead of creating a second one.
</Callout>

#### Create the Worker

Log into your <Anchor target="_blank" href="https://dash.cloudflare.com/">Cloudflare Dashboard</Anchor>, open the **Compute (Workers)** dropdown, and click **Workers & Pages**.

![Cloudflare Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-sidebar.png)

Click the blue **Create** button near the top.

![Cloudflare Workers And Pages](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-workers-and-pages.png)

Choose the option to create a hello world Worker. Its code is replaced in the next step.

![Cloudflare Worker Get Started](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-worker-get-started.png)

Give the Worker a name such as `tollbit-worker`, and click **Deploy**.

![Worker Creation](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/worker-creation.png)

#### Add the Worker Code

Your `tollbit` subdomain is set up as part of onboarding your site to the TollBit platform, so complete that first. Once the Worker has finished deploying, click **Edit code**.

![Cloudflare Edit Code](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-edit-code.png)

In the `worker.js` file, delete everything and paste the code below. Then update the two settings near the top of the code to match how you send logs:

- **Worker analytics**: leave `FORWARD_LOGS` as `true`, and replace `YOUR_SECRET_KEY_HERE` with the secret key from your <Anchor target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor>.
- **Logpush analytics**: set `FORWARD_LOGS` to `false`. The secret key is not used, so you can leave it as it is.

```js
// The AI agents routed to your Agent Site. Add any others you would like to route.
const botList = [
  'Amazonbot',
  'Amzn-SearchBot',
  'anthropic-ai',
  'Bytespider',
  'CCBot',
  'ChatGPT-User',
  'claude-code',
  'Claude-SearchBot',
  'Claude-User',
  'Claude-Web',
  'ClaudeBot',
  'cohere-ai',
  'Diffbot',
  'Exabot',
  'GPTBot',
  'meta-externalagent',
  'Meta-Webindexer',
  'OAI-AdsBot',
  'OAI-SearchBot',
  'Perplexity-User',
  'PerplexityBot',
  'Shap-User',
  'ShapBot',
  'Timpibot',
  'YouBot'
]

// Analytics. Set FORWARD_LOGS to false if you send logs to TollBit with Logpush.
const FORWARD_LOGS = true
const tollbitLogEndpoint = 'https://log.tollbit.com/log'
const tollbitToken = 'YOUR_SECRET_KEY_HERE'

// Routing visitors from cited content
const tollbitFallbackOrigin = 'https://fallback.tollbit.com'
const FALLBACK_TIMEOUT_MS = 2000

const sleep = (ms) => {
  return new Promise((resolve) => {
    setTimeout(resolve, ms)
  })
}

const buildLogMessage = (request, response) => {
  const logObject = {
    timestamp: new Date().toISOString(),
    ip_address: request.headers.get('cf-connecting-ip'),
    geo_country: request.cf['country'],
    geo_city: request.cf['city'],
    geo_postal_code: request.cf['postalCode'],
    geo_latitude: request.cf['latitude'],
    geo_longitude: request.cf['longitude'],
    host: request.headers.get('host'),
    url: request.url.replace('https://' + request.headers.get('host'), ''),
    request_method: request.method,
    request_protocol: request.cf['httpProtocol'],
    request_user_agent: request.headers.get('user-agent'),
    request_latency: null, // cloudflare does not have latency information
    request_referer: request.headers.get('referer'),
    response_state: null,
    response_status: response.status,
    response_reason: response.statusText,
    response_body_size: response.contentLength,
    signature: request.headers.get('signature'),
    signature_agent: request.headers.get('signature-agent'),
    signature_input: request.headers.get('signature-input'),
  }
  return logObject
}

// Batching
const BATCH_INTERVAL_MS = 20000 // 20 seconds
const MAX_REQUESTS_PER_BATCH = 500 // 500 logs

let batchTimeoutReached = true
let logEventsBatch = []

// Backoff
const BACKOFF_INTERVAL = 10000
let backoff = 0

async function addToBatch(body, event) {
  logEventsBatch.push(body)

  if (logEventsBatch.length >= MAX_REQUESTS_PER_BATCH) {
    event.waitUntil(postBatch(event))
  }

  return true
}

const checkIfBotRequest = (request) => {
  const userAgent = (request.headers.get('User-Agent') || '').toLowerCase()

  for (var i = 0; i < botList.length; i++) {
    if (userAgent.includes(botList[i].toLowerCase())) {
      return true
    }
  }
  return false
}

// AI agents: fetch the same path from your tollbit subdomain.
async function fetchFromAgentSite(request) {
  const path = request.url.replace('https://' + request.headers.get('host'), '')
  let host = request.headers.get('host') || ''
  if (host.startsWith('www.')) {
    host = host.slice(4)
  }
  const tollbitUrl = 'https://tollbit.' + host + path

  const proxiedResponse = await fetch(new Request(tollbitUrl, {
    method: request.method,
    headers: request.headers,
    body: request.body,
    redirect: 'manual'
  }))

  const responseHeaders = new Headers(proxiedResponse.headers)
  responseHeaders.set('Cache-Control', 'no-store')

  return new Response(proxiedResponse.body, {
    status: proxiedResponse.status,
    statusText: proxiedResponse.statusText,
    headers: responseHeaders
  })
}

// Visitors: your site has no page for this URL, so ask TollBit whether a
// destination is configured for it. Returns a redirect, or null to leave your
// site's own response untouched. Any error or timeout returns null.
async function notFoundFallback(request) {
  const controller = new AbortController()
  const timer = setTimeout(() => controller.abort(), FALLBACK_TIMEOUT_MS)

  try {
    const url = new URL(request.url)
    const fallback = await fetch(tollbitFallbackOrigin + url.pathname + url.search, {
      method: request.method,
      headers: { 'X-Tollbit-Host': url.hostname },
      redirect: 'manual',
      signal: controller.signal
    })

    const location = fallback.headers.get('Location')
    if (fallback.status >= 300 && fallback.status < 400 && location) {
      const headers = new Headers({ Location: location })
      const cacheControl = fallback.headers.get('Cache-Control')
      if (cacheControl) {
        headers.set('Cache-Control', cacheControl)
      }
      return new Response(null, { status: fallback.status, headers })
    }
    return null
  } catch (err) {
    return null
  } finally {
    clearTimeout(timer)
  }
}

async function handleRequest(event) {
  const { request } = event
  let response

  if (checkIfBotRequest(request)) {
    response = await fetchFromAgentSite(request)
  } else {
    response = await fetch(request)

    if (response.status === 404 && (request.method === 'GET' || request.method === 'HEAD')) {
      const redirect = await notFoundFallback(request)
      if (redirect) {
        response = redirect
      }
    }
  }

  if (FORWARD_LOGS) {
    event.waitUntil(addToBatch(buildLogMessage(request, response), event))
  }
  return response
}

const fetchAndSetBackOff = async (lfRequest, event) => {
  if (backoff <= Date.now()) {
    const resp = await fetch(tollbitLogEndpoint, lfRequest)
    if (resp.status === 403 || resp.status === 429) {
      backoff = Date.now() + BACKOFF_INTERVAL
    }
  }

  event.waitUntil(scheduleBatch(event))

  return true
}

const postBatch = async (event) => {
  const batchInFlight = [...logEventsBatch.map((e) => JSON.stringify(e))]
  logEventsBatch = []
  const body = batchInFlight.join('\n')
  const request = {
    method: 'POST',
    headers: {
      TollbitKey: `${tollbitToken}`,
      'Content-Type': 'application/json',
    },
    body,
  }
  event.waitUntil(fetchAndSetBackOff(request, event))
}

const scheduleBatch = async (event) => {
  if (batchTimeoutReached) {
    batchTimeoutReached = false
    await sleep(BATCH_INTERVAL_MS)
    if (logEventsBatch.length > 0) {
      event.waitUntil(postBatch(event))
    }
    batchTimeoutReached = true
  }
  return true
}

addEventListener('fetch', (event) => {
  // If the Worker itself fails, the request is served by your site as usual.
  event.passThroughOnException()

  if (FORWARD_LOGS) {
    event.waitUntil(scheduleBatch(event))
  }
  event.respondWith(handleRequest(event))
})
```

Click **Deploy** in the upper right corner once you are finished, then leave the editor with the back arrow in the upper left.

#### Attach the Worker to Your Site

Click **Account Home**, select your site, and open **Worker Routes**. Click **Add route**, set the route to `*.<your_site.com>/*`, and choose the Worker you just created.

![Add Route](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/add-route.png)

Click **Save**. The Worker starts handling requests as soon as the route is saved.

<Callout icon="📘" theme="info">
  ### Note

  If your site does not use the `www` subdomain and all traffic to `www` is redirected to your main site (`www.example.com` is redirected to `example.com`), set the route to `<your_site.com>/*` instead.
</Callout>

#### Minimizing Worker Usage

With the route above, every request to your site runs through the Worker. You can leave out requests for static assets and scripts, which do not need it.

If your site serves its static assets from a common path, such as `example.com/assets`, add routes for that path, in this example `*.example.com/assets*` and `example.com/assets*`, and set the Worker for those routes to **Empty**.

![Cloudflare Worker Route Disable](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-worker-route-disable.png)

Requests on a route without the Worker are not routed to your Agent Site and do not get visitor routing, so only exclude paths that serve static files. If the Worker forwards your logs, those requests are also left out of your analytics, which works best with requests that correspond to page views. Logpush records them either way.

# What the Worker Does

#### Analytics

With `FORWARD_LOGS` set to `true`, the Worker records each request it handles, with the status that was actually served, and forwards the logs to TollBit in batches. Forwarding happens after the response has been returned, so it does not slow down your site. Logs are authenticated with your secret key.

With `FORWARD_LOGS` set to `false`, the Worker sends nothing to TollBit. Logpush records every request to your site, including the ones the Worker serves from your Agent Site or redirects.

#### Agent Site

AI agents that request your site are served from your Agent Site on your `tollbit` subdomain, while visitors continue to reach your own origin. The Worker matches the `User-Agent` header against the list at the top of the code, without regard to case, and fetches the same path and query string from your `tollbit` subdomain. For `www.example.com` and `example.com`, that is `tollbit.example.com`.

#### Routing Visitors from Cited Content

Content published through the Agent Site CMS lives on your `tollbit` subdomain and is served to AI agents. When an agent cites one of these pages in an answer, the reader can click that citation to visit your site. Since the page was published for agents, the URL may not exist on your main site. The Worker lets TollBit route these visitors to a destination that you configure, such as your home page or the published page itself.

This only applies to URLs that your site has no page for, so existing pages and normal traffic are not affected. If TollBit has nothing configured for a URL, or TollBit cannot be reached, your site serves its own error page, exactly as it does today.

1. A visitor requests a URL and your origin responds with a `404`.
2. The Worker asks TollBit whether anything is configured for that URL. The request is sent to the TollBit fallback origin, `fallback.tollbit.com`, with the same path and query string and your site's hostname in the `X-Tollbit-Host` header.
3. If TollBit has a destination configured, it responds with a redirect, either to that destination or to the published page on your `tollbit` subdomain. The Worker returns that redirect and the visitor follows it.
4. If TollBit responds with a `404`, takes longer than two seconds, or fails in any way, the response is left untouched and your origin's own error page is served.

TollBit is only consulted for `404` responses to `GET` and `HEAD` requests from visitors, so no other request does extra work.

#### Caching

Agent Site responses are returned with `Cache-Control: no-store`. Redirect responses from TollBit include `Cache-Control: public, max-age=300`, which the Worker passes on, so a visitor's browser may reuse a redirect for up to five minutes. Keep this in mind when testing so a cached response is not mistaken for a misconfiguration.

# Verifying the Setup

#### Routing Visitors from Cited Content

Make these requests as a regular visitor, not with the user agent of an AI agent. Requests from AI agents are served from your Agent Site instead, so they do not receive the redirect. The commands below use curl's own user agent, which counts as a visitor.

Request an agent-only URL that has a configured destination and confirm you receive the redirect. Then request a URL your site has no page for, such as a made-up path, and confirm your site's normal error page is served with a `404` status.

```shell
# Expect a redirect status and a Location header
curl -sI https://www.example.com/agent-only-page

# Expect 404, with your site's own error page as the body
curl -s -o /dev/null -w "%{http_code}\n" https://www.example.com/made-up-path
```

Contact [team@tollbit.com](mailto:team@tollbit.com) to have us configure an agent-only test URL and destination for your property.

#### Analytics

Charts in your TollBit dashboard are updated once a day, so allow up to 24 hours after the Worker is attached. With Logpush, analytics starts once we confirm your logs are being ingested.
