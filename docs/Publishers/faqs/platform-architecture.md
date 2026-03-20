---
title: Platform Architecture
excerpt: Questions about subdomains, CDNs, analytics, and how TollBit works.
---

# Platform Architecture

## Why do I need to verify my property / set up a subdomain? What is the purpose of TXT and NS records?

Setting up a TollBit subdomain (e.g., `tollbit.yourcompany.com`) is a critical step in establishing a trusted, monitored gateway for AI agents to access your content and services.

The Tollbit subdomain serves as the unified gateway where AI agents can access website content, headless browsing capabilities, protocol interactions like MCP and NLWeb, and all future services and integrations. In addition, it provides enhanced security and control, unified access point for AI agent requests, and trust and authenticity.

## Do I need a CDN to get started with TollBit?

We integrate with HTTP traffic logs primarily from CDNs because client-side tools like Google Analytics rely on JavaScript, which most sophisticated bots don't execute. That means tools like GA will miss the majority of bot traffic.

By using server logs, we get full visibility into every request including bots because the logs come straight from your edge or origin infrastructure, not the browser. This lets us give publishers a complete picture of how and where bots are accessing their content.

That said, we do offer a few additional options to get started with TollBit in case your team is currently not using a CDN. Please reach out to team@tollbit.com to visit other options.

### Supporting TollBit Analytics

Access to HTTP traffic logs that can either be either dropped in a S3, GCP, or Azure storage bucket or called via our Log Sink API endpoint.

### Supporting TollBit Bot Paywall

Access to a method to redirect bot-specific user agents from source URL to a target URL. Apart from CDNs, this can be done via cyber security tools like DataDome and Human Security, or hosting platforms like WordPress VIP as well.

## How exactly do you gather analytics data from publishers?

We ingest server-side logs directly from publishers, either via a streaming endpoint or by pulling from S3. This is critical because client-side tools like Google Analytics rely on JavaScript, which most sophisticated bots don't execute. That means tools like GA will miss the majority of bot traffic.

By using server logs, we get full visibility into every request including bots because the logs come straight from your edge or origin infrastructure, not the browser. This lets us give publishers a complete picture of how and where bots are accessing their content. In addition, this approach also doesn't affect page load or performance as well.

For verification, real-time streaming allows for instantaneous verification of log setup. In contrast, S3-based log ingestion currently involves a manual process and takes up to 24 hours for verification, as we perform nightly jobs to process the logs. We plan to support self-service onboarding for S3 in the future to enable more immediate verification.

## Is the TollBit subdomain required for the paywall?

Yes, the subdomain is a technical requirement for the paywall to function properly. This is because:

- The redirect from your CDN (via edge logic or firewall rule) must point to a location you control, which we enforce as a custom subdomain like ai.example.com.
- That subdomain is where TollBit can serve a real paywall page to unauthenticated bots (e.g. Claude, ChatGPT).
- It's also how we isolate bot traffic from your human users.

Setting up this subdomain is usually a quick DNS config.

## How does the bot redirect mechanism work?

It's straightforward. At your CDN edge (for example, Cloudflare or Akamai), you set up a small worker script or edge logic. That logic checks each request against a simple list of known bot user agents. If the incoming request matches one of those user agents, the CDN immediately redirects it to a TollBit-managed subdomain.

TollBit then checks if the bot has a valid authentication token. If not, we show the bot a paywall page, instructing them to get a valid token. If the bot does have a token, we fetch the requested content securely from the publisher and deliver it, ensuring publishers get paid accordingly.

Blocking at the CDN level would only block the bot traffic without giving the bot a method to authenticate and pay for governed access.

## What kind of testing environment does TollBit provide for integrations?

We provide a flexible testing approach. Publishers or integration partners (like yourselves) can create test organizations within TollBit, set up sandbox websites, and configure edge rules pointing to these test environments.

You can also create custom test user-agents and API tokens to simulate the full request-response flow end-to-end. This makes automated integration testing straightforward and comprehensive, so you can confidently deploy integrations to production.

## Can I use another AI marketplace like TollBit in parallel?

Yes, this is dependent on how you set up your log exports and bot redirect policies within your edge servers. In addition, if you do have existing deals with other AI companies, we can work with them to manage the content access through TollBit.
