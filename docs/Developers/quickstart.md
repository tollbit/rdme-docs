---
title: Quickstart
excerpt: This guide will get you all set up and ready to use the TollBit API. We'll cover how to get started with an API client and how to make your first API request.
---

# Quickstart

This guide will get you all set up and ready to use the TollBit API. We'll cover how to get your account ready and get started using one of our API clients and how to make your first API request.

<Callout icon="ℹ️" theme="info">
  Before getting started, please create a Developer Account through the [Developer Dashboard](https://hack.tollb.it)
</Callout>

## Register your AgentID

Once you've created an account on the [Developer Dashboard](https://hack.tollb.it), head to the [Dashboard > Home](https://hack.tollb.it/home) page. There, you can register your unique [AgentID](/docs/developers/dev-overview#agentid). Think of this as your username as you make requests to content providers on TollBit.

## Copy your secret key

Next, head to the [Dashboard > Access](https://hack.tollb.it/access) page and copy your secret key. DO NOT share this with anyone. Keep it safe!

## Get a signed token for your request

Anyone can set up a `tollbit.<example>.com` domain, so it is important to **not include your api key** in requests to these domains. Instead, you should call a TollBit API to request a signed token for your requests.

This endpoint will validate that this subdomain is registered with us, and will generate a cryptographically signed token for requests to that domain. You set the parameters that you want for your request at token generation time. This token has limited scope of where it can be used and accepted to limit abuse and man-in-the-middle attacks.

**POST** `https://gateway.tollbit.com/tollbit/dev/v2/tokens/content`

```bash
curl https://gateway.tollbit.com/tollbit/dev/v2/tokens/content \
  --header 'TollbitKey: <your tollbit key>' \
  --header 'Content-Type: application/json' \
  -d '{
    "url": "pioneervalleygazette.com/daydream",
    "userAgent": "<user agent you will use>",
    "maxPriceMicros": 1000000,
    "currency": "USD",
    "licenseType": "ON_DEMAND_LICENSE"
  }'
```

## Making your first API request

After picking your preferred client, you are ready to make your first call to the TollBit API. Below, you can see how to send a GET request to fetch the content from a page of Josh's blog.

**GET** `https://tollbit.pioneervalleygazette.com/daydream`

```bash
curl -G https://tollbit.pioneervalleygazette.com/daydream \
  -H "Tollbit-Token: <token>" \
  -H "User-Agent: <user_agent>" \
  -H "Tollbit-Accept-Content: text/markdown"
```

> **Note**: For detailed API documentation on the Get Content endpoint, see the API Reference section (generated from OpenAPI specs).

## What's next?

Great, you're now set up with an API client and have made your first request to the API. Here are a few links that might be handy as you venture further into the TollBit API:

- [Checkout the TollBit Developer Dashboard](https://hack.tollb.it)
- [Go over the dashboard overview](/docs/developers/dev-overview)
- [Look at our Python SDK](https://github.com/tollbit/tollbit-python-sdk)

