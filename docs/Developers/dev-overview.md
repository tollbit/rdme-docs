---
title: Developer Dashboard Overview
excerpt: Detailed dive into the entire Developer Dashboard and what pages mean.
---

# Developer Dashboard Overview

A full overview of the pages and main ideas presented in the [Developer Dashboard](https://hack.tollbit.com).

## The Home page

The home page is where you can get up to date summaries of your usage. You can also register your applications' AgentIDs.

## User Agents

User Agents are unique "usernames" for your AI applications. They are meant to self-identify your AI agents. The idea behind an AgentID is directly built on top of the HTTP header [User Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent), where the AgentID is the "name" field of the UA String.

**Example User Agent:**

```
Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; tollbot/1.0; +https://tollbit.com/bot
```

This is an example full User Agent string with AgentID of `tollbot`.

## Access

This page allows you to copy and regenerate your secret key. Secret keys should remain secret! We use an encrypted key store so that your keys will always be secure. They are generated using native 256-bit crypto functions and encrypted using [New GCM symmetric](https://en.wikipedia.org/wiki/Galois/Counter_Mode) key algorithms.

## Transactions

This page allows you to see your recent data access transactions and their detailed descriptions.

## Billing

This page allows you to add your payment method of choice through Stripe.

