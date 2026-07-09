---
title: CloudFlare (old)
excerpt: Learn how to integrate TollBit with CloudFlare.
hidden: true
---
We provide a way for all CloudFlare customers, regardless of plan, to forward HTTP logs to our platform for analytics. We recommend this method over others like LogPush as CloudFlare Enterprise is not required to create workers, and you have much more control over how logs are sent.

### Enable Analytics with CloudFlare

#### Enterprise Plan Customers

If you are on the Enterprise plan, you should have access to CloudFlare's <Anchor label="Logpush" target="_blank" href="https://developers.cloudflare.com/logs/about/">Logpush</Anchor> feature. You may already be pushing logs to an S3, R2 or GCP bucket. If this is the case, we are able to ingest your logs from where they are already being stored.

One small update you may need to make is adding the `location` response header and the `signature-agent`, `signature-input` and `signature` request headers to the logs. Follow <Anchor label="these steps" target="_blank" href="https://developers.cloudflare.com/logs/reference/custom-fields/#enable-custom-fields-via-dashboard">these steps</Anchor> in Cloudflare's documentation to add this header. You will want to select "Response Header" as the field type and type in `location`, and select "Request Header" and type in `signature-agent`, `signature-input` and `signature`.

If your logs are already being sent to an S3 bucket, add the following IAM policy to your bucket to enable TollBit to process your logs:

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

Once you have done that, reach out to [team@tollbit.com](mailto:team@tollbit.com) and provide the path to your logs in your bucket and we will be able to quickly enable TollBit analytics for your site.

If you are not on Enterprise, read on to set up a worker to forward logs.

#### Free, Pro or Business Plan Customers

** Create new Worker **

<Callout icon="📘" theme="info">
  You must be proxying traffic through CloudFlare in order have the worker seconds your logs over to us. Most websites are already doing this, but if you are not certain, you can check by going into your site's DNS page and ensuring that your main site's DNS settings have proxy status as `Proxied`.

  ![Cloudflare Proxied](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-proxied.png)
</Callout>

If you already have an existing worker that is intercepting requests for your site, or you already set up a worker in the Agent Site section below, you will need to integrate this logging code with that worker. If you just have a bot deterrence worker set up, see that section to get a code snippet that also pushes logs.

Log into your <Anchor label="CloudFlare Dashboard" target="_blank" href="https://dash.cloudflare.com/">CloudFlare Dashboard</Anchor> and click on the "Compute (Workers)" tab to have it open as a dropdown, and click on "Workers & Pages".

![Cloudflare Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-sidebar.png)

Click on the blue "Create" button near the top.

![Cloudflare Workers And Pages](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-workers-and-pages.png)

This will take you to a get started screen. Choose the option to create a hello world worker, as we were be overwriting all the worker code in the next few steps anyways.

![Cloudflare Worker Get Started](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-worker-get-started.png)

Next will be a screen where you can name your worker and see the initial code that it will be running. Set the name to something TollBit related such as `tollbit-worker`, and click deploy. We will be modifying the worker code shortly.

![Worker Creation](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/worker-creation.png)

** Updating the Worker Code **

Once your worker has finished deploying, click "Edit code".

![Cloudflare Edit Code](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-edit-code.png)

In the `worker.js` file, delete everything and copy the following code over exactly, making sure to replace `YOUR_SECRET_KEY_HERE` with the secret key you can find in your <Anchor label="portal" target="_blank" href="https://app.tollbit.com">portal</Anchor>.

```js
const CF_APP_VERSION = '1.0.0'

const tollbitLogEndpoint = 'https://log.tollbit.com/log'
const tollbitToken = 'YOUR_SECRET_KEY_HERE'

const sleep = (ms) => {
  return new Promise((resolve) => {
    setTimeout(resolve, ms)
  })
}

const makeid = (length) => {
  let text = ''
  const possible = 'ABCDEFGHIJKLMNPQRSTUVWXYZ0123456789'
  for (let i = 0; i < length; i += 1) {
    text += possible.charAt(Math.floor(Math.random() * possible.length))
  }
  return text
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
const WORKER_ID = makeid(6)

let workerTimestamp

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

async function handleRequest(event) {
  const { request } = event

  const response = await fetch(request)
  const rCf = request.cf
  delete rCf.tlsClientAuth
  delete rCf.tlsExportedAuthenticator

  const eventBody = buildLogMessage(request, response)
  event.waitUntil(addToBatch(eventBody, event))

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
  event.passThroughOnException()

  if (!workerTimestamp) {
    workerTimestamp = new Date().toISOString()
  }

  event.waitUntil(scheduleBatch(event))
  event.respondWith(handleRequest(event))
})
```

Hit "Deploy" on the upper righthand corner once you are finished, and then navigate out of the editor with the little back arrow on the upper left side of the page, next to the name of the worker.

** Link worker to CloudFlare HTTP Logs **

Click on "Account Home" on the left pane and select the website that you would like to forward logs for, and click into it. On the left panel, click into "Worker Routes", and then click "Add route". Set the route to `*.<your_site.com>/*`, or a custom path if you only want to forward logs for certain URL patterns. Under workers, choose the worker that you just created.

![Add Route](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/add-route.png)

Once you are ready, click "Save", and you are all set!

<Callout icon="📘" theme="info">
  If your main site does not use the `www` subdomain and all traffic to `www` gets redirected to your main site (`www.example.com` gets redirected to `example.com`), you will need to set your worker route to just `<your_site.com>/*`.
</Callout>

### Minimizing Worker Usage

By default, the above configuration will have every request to your site run through your worker. To reduce the number of requests for workers, we must keep in mind that our analytics platform works best if we try to send logs that correspond to page views, and avoid sending logs that are for requests for static assets or javascript files.

To successfully minimize worker usage, investigate your directory structure and see if you have a common paths for static assets. For example, some CMS frameworks will have a directory similar to `example.com/assets` for assets. To avoid running the worker on these request paths, create a new route for your worker for that path, in this example `*.example.com/assets*` and `example.com/assets*`, and set the worker for that route to be "Empty".

Your route page will then look something like the following.

![Cloudflare Worker Route Disable](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/cloudflare-worker-route-disable.png)

If you aren't sure which route to disable, consider running the worker on your full site and then using the top pages chart in our analytics platform to understand any routes you wish to filter out.

### Route to Agent Site

There are several levels of bot detection and forwarding that you can configure for CloudFlare, depending on whether or not you are on their Enterprise plan.

<Callout icon="📘" theme="info">
  The code snippets here are for a clean CloudFlare environment. If you have existing workers that are processing requests from your domain, you will need to integrate these scripts into your existing worker.
</Callout>

### CloudFlare Snippets

For customers on the Pro, Business or Enterprise plans that want to set up a simple redirect without the overhead of setting up workers, Snippets are a quick and cost effective way to set up a redirect following simple rules.

First, in the left nav when you're within the view for one of your websites, find the Rules tab and click the dropdown, and then click into the Snippets section.

<Image align="center" src="https://files.readme.io/77f26a783e280037b14b03c5bd4ce0477048afb8d8df98f7844c518d1a6ad7c3-Screenshot_2026-03-04_at_10.20.06_AM.png" />

Once you're on the snippets page, click the Create Snippet button. This will take you to an editor similar to the Workers editor. Name this file something along the lines of `redirect_to_tollbit`. You can paste in the following code, making modifications to the bot list as you find appropriate for your goals.

```javascript
const botList = [
  'ChatGPT-User',
  'PerplexityBot',
  'GPTBot',
  'anthropic-ai',
  'CCBot',
  'Claude-Web',
  'ClaudeBot',
  'Claude-User',
  'cohere-ai',
  'YouBot',
  'Diffbot',
  'OAI-SearchBot',
  'meta-externalagent',
  'Timpibot',
  'Amazonbot',
  'Bytespider',
  'Perplexity-User',
  'Claude-SearchBot',
'Meta-Webindexer',
'Amzn-SearchBot'
]

export default {
  async fetch(request) {
    const isBotRequest = checkIfBotRequest(request)

    // if bot request, immediately forward to subdomain
    if (isBotRequest) {
      const path = request.url.replace(
        'https://' + request.headers.get('host'),
        '',
      )
      let host = request.headers.get('host') || ''
      if (host.startsWith('www.')) {
        // remove www
        host = host.slice(4)
      }
      return Response.redirect('https://tollbit.' + host + path, 302)
    } else {
      // otherwise return the regular content
      const response = await fetch(request)
      return response
    }
  },
};

const checkIfBotRequest = (request) => {
  const userAgent = request.headers.get('User-Agent') || ''

  for (var i = 0; i < botList.length; i++) {
    if (userAgent.toLowerCase().includes(botList[i].toLowerCase())) {
      return true
    }
  }
  return false
}
```

Before saving and deploying this Snippet, click on the "Snippet rule" button on the upper right and select "All incoming requests".

<Image align="center" src="https://files.readme.io/3aa20f5a697cdf913c2b93e0b6af029e89c7d1a7b61b2d2e8ce9912f436a0454-Screenshot_2026-03-04_at_10.39.05_AM.png" />

Now, you should be able to click Deploy, and this Snippet will immediately begin forwarding requests with these user agents to your `tollbit` subdomain.

<Callout icon="🚧" theme="warn">
  This Snippet **will intercept** and **forward** traffic from your site to your `tollbit` subdomain. It is crucial to make sure that you are certain of this change and QA it thoroughly to ensure that it is not blocking human traffic or good bot traffic (Google, etc) before elevating it across your entire website.
</Callout>

### CloudFlare Workers

This section is for customers who have advanced functionality with their current request interception flow, or for customers who do not have access to Snippets. Follow the steps described above (within the Cloudflare Analytics section) up until you have created a new worker. Name this working something to help you keep track of it's function (such as `bot-forwarding-worker`). Once you've created this worker, click into edit code and do the following to set up your forwarding worker.

<Callout icon="📘" theme="info">
  If you have already created a CloudFlare worker for log forwarding, **DO NOT** create a new worker. Use your existing worker when following these instructions. This is because you cannot have two CloudFlare workers on the same route, and if you do, only one will be receive requests.
</Callout>

**If you have set up log forwarding**, copy and replace your `worker.js` file with this code instead. Make sure that you keep your TollBit token copied over into the code.

```js
// this is a non-exhaustive list of agents that we recommend you get started with first
// Add any other agents you would like to forward into this list.
const botList = [
  'ChatGPT-User',
  'PerplexityBot',
  'GPTBot',
  'anthropic-ai',
  'CCBot',
  'Claude-Web',
  'ClaudeBot',
  'Claude-User',
  'cohere-ai',
  'YouBot',
  'Diffbot',
  'OAI-SearchBot',
  'meta-externalagent',
  'Timpibot',
  'Amazonbot',
  'Bytespider',
  'Perplexity-User',
  'Claude-SearchBot',
'Meta-Webindexer',
'Amzn-SearchBot'
]

const CF_APP_VERSION = '1.0.0'

const tollbitLogEndpoint = 'https://log.tollbit.com/log'
const tollbitToken = 'YOUR_SECRET_KEY_HERE'

const sleep = (ms) => {
  return new Promise((resolve) => {
    setTimeout(resolve, ms)
  })
}

const makeid = (length) => {
  let text = ''
  const possible = 'ABCDEFGHIJKLMNPQRSTUVWXYZ0123456789'
  for (let i = 0; i < length; i += 1) {
    text += possible.charAt(Math.floor(Math.random() * possible.length))
  }
  return text
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
const BATCH_INTERVAL_MS = 20000 // 30 seconds
const MAX_REQUESTS_PER_BATCH = 500 // 500 logs
const WORKER_ID = makeid(6)

let workerTimestamp

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

async function handleRequest(event) {
  const { request } = event
  const isBotRequest = checkIfBotRequest(request)

  // if bot request, immediately forward to subdomain
  if (isBotRequest) {
    const path = request.url.replace(
      'https://' + request.headers.get('host'),
      '',
    )
    let host = request.headers.get('host') || ''
    if (host.startsWith('www.')) {
      // remove www
      host = host.slice(4)
    }
    const redirectUrl = 'https://tollbit.' + host + path;
    const response = new Response(null, {
      status: 302,
      headers: {
        'Location': redirectUrl,
        'Cache-Control': 'no-store',
        'Pragma': 'no-cache',
        'Expires': '0'
      }
    });
    const logObject = buildLogMessage(request, response);
    event.waitUntil(addToBatch(logObject, event));

    return response;
  } else {
    const response = await fetch(request)
    // otherwise add to log batch and return response
    const eventBody = buildLogMessage(request, response)
    event.waitUntil(addToBatch(eventBody, event))
    return response
  }
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

const checkIfBotRequest = (request) => {
  const userAgent = request.headers.get('User-Agent') || ''

  for (var i = 0; i < botList.length; i++) {
    if (userAgent.toLowerCase().includes(botList[i].toLowerCase())) {
      return true
    }
  }
  return false
}

addEventListener('fetch', (event) => {
  event.passThroughOnException()

  if (!workerTimestamp) {
    workerTimestamp = new Date().toISOString()
  }

  event.waitUntil(scheduleBatch(event))
  event.respondWith(handleRequest(event))
})
```

This code will immediately let through anyone with a known browser, and check all other requests against a list that we will periodically update with known bad user agents.

**If you have not set up log forwarding and just want to forward bot traffic**, put this code in your `worker.js` file.

```js
// this is a non-exhaustive list of agents that we recommend you get started with first
// Add any other agents you would like to forward into this list.
const botList = [
  'ChatGPT-User',
  'PerplexityBot',
  'GPTBot',
  'anthropic-ai',
  'CCBot',
  'Claude-Web',
  'ClaudeBot',
  'Claude-User',
  'cohere-ai',
  'YouBot',
  'Diffbot',
  'OAI-SearchBot',
  'meta-externalagent',
  'Timpibot',
  'Amazonbot',
  'Bytespider',
  'Perplexity-User',
  'Claude-SearchBot',
]

export default {
  fetch(request) {
    const userAgent = request.headers.get('User-Agent') || ''
    const path = request.url.replace(
      'https://' + request.headers.get('host'),
      '',
    )
    let host = request.headers.get('host') || ''
    if (host.startsWith('www.')) {
      // remove www
      host = host.slice(4)
    }
    for (var i = 0; i < botList.length; i++) {
      if (userAgent.toLowerCase().includes(botList[i].toLowerCase())) {
        return Response.redirect('https://tollbit.' + host + path, 302)
      }
    }

    // Default behaviour
    return fetch(request)
  },
}
```

### CloudFlare Enterprise and Bot Management

If you are on Enterprise and are using Bot Management, you should have access to the bot score in the header of the request. You can replace the `checkIfBotRequest` function in the previous worker scripts to use something similar to the following, and you can set the `BOT_SCORE_THRESHOLD` to determine how strict your forwarding is. CloudFlare <Anchor label="lists what each score range means" target="_blank" href="https://developers.cloudflare.com/bots/concepts/bot-score/#bot-groupings">lists what each score range means</Anchor>.

```js
const checkIfBotRequest = (request) => {
  const userAgent = request.headers.get('User-Agent') || '';

  // Check for known AI agents
  for (let i = 0; i < botList.length; i++) {
    if (userAgent.toLowerCase().includes(botList[i].toLowerCase())) {
      return true;
    }
  }

  // Check bot score
  const botScore = request.cf?.botManagement?.score;
  if (botScore !== undefined && botScore < BOT_SCORE_THRESHOLD) {
    return true;
  }

  return false;
};
```

<Callout icon="🚧" theme="warn">
  This Worker **will intercept** and potentially **forward** traffic from your site to your `tollbit` subdomain. It is crucial to make sure that you are certain of this change and QA it thoroughly to ensure that it is not blocking human traffic or good bot traffic (Google, etc) before elevating it across your entire website.
</Callout>

### Enterprise

If you have CloudFlare enterprise, you should be able to use the <Anchor label="Bot Management" target="_blank" href="https://developers.cloudflare.com/bots/get-started/bm-subscription/">Bot Management</Anchor> product to get a bot score for each request. You can add logic in the above code's `checkIfBotRequest` function to also return `true` if the bot score is lower than a certain threshold.