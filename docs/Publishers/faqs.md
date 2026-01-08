---
title: FAQs
excerpt: Commonly asked questions about the TollBit platform.
---

# Publisher FAQs

This section covers commonly asked questions about the TollBit platform. Questions are broken down into 5 categories - platform architecture, bot management, monetization, security, and pricing.

## Platform Architecture

### Why do I need to verify my property / set up a subdomain? What is the purpose of TXT and NS records?

Setting up a Tollbit subdomain (e.g., tollbit.yourcompany.com) is a critical step in establishing a trusted, monitored gateway for AI agents to access your content and services.

The Tollbit subdomain serves as the unified gateway where AI agents can access website content, headless browsing capabilities, protocol interactions like MCP and NLWeb, and all future services and integrations. In addition, it provides enhanced security and control, unified access point for AI agent requests, and trust and authenticity.

### Do I need a CDN to get started with TollBit?

We integrate with HTTP traffic logs primarily from CDNs because client-side tools like Google Analytics rely on JavaScript, which most sophisticated bots don't execute. That means tools like GA will miss the majority of bot traffic.

By using server logs, we get full visibility into every request including bots because the logs come straight from your edge or origin infrastructure, not the browser. This lets us give publishers a complete picture of how and where bots are accessing their content.

That said, we do offer a few additional options to get started with TollBit in case your team is currently not using a CDN. Please reach out to team@tollbit.com to visit other options.

#### Supporting TollBit Analytics

Access to HTTP traffic logs that can either be either dropped in a S3, GCP, or Azure storage bucket or called via our Log Sink API endpoint.

#### Supporting TollBit Bot Paywall

Access to a method to redirect bot-specific user agents from source URL to a target URL. Apart from CDNs, this can be done via cyber security tools like DataDome and Human Security, or hosting platforms like WordPress VIP as well.

### How exactly do you gather analytics data from publishers?

We ingest server-side logs directly from publishers, either via a streaming endpoint or by pulling from S3. This is critical because client-side tools like Google Analytics rely on JavaScript, which most sophisticated bots don't execute. That means tools like GA will miss the majority of bot traffic.

By using server logs, we get full visibility into every request including bots because the logs come straight from your edge or origin infrastructure, not the browser. This lets us give publishers a complete picture of how and where bots are accessing their content. In addition, this approach also doesn't affect page load or performance as well.

For verification, real-time streaming allows for instantaneous verification of log setup. In contrast, S3-based log ingestion currently involves a manual process and takes up to 24 hours for verification, as we perform nightly jobs to process the logs. We plan to support self-service onboarding for S3 in the future to enable more immediate verification.

### Is the TollBit subdomain required for the paywall?

Yes, the subdomain is a technical requirement for the paywall to function properly. This is because:

- The redirect from your CDN (via edge logic or firewall rule) must point to a location you control, which we enforce as a custom subdomain like ai.example.com.
- That subdomain is where TollBit can serve a real paywall page to unauthenticated bots (e.g. Claude, ChatGPT).
- It's also how we isolate bot traffic from your human users.

Setting up this subdomain is usually a quick DNS config.

### How does the bot redirect mechanism work?

It's straightforward. At your CDN edge (for example, Cloudflare or Akamai), you set up a small worker script or edge logic. That logic checks each request against a simple list of known bot user agents. If the incoming request matches one of those user agents, the CDN immediately redirects it to a TollBit-managed subdomain.

TollBit then checks if the bot has a valid authentication token. If not, we show the bot a paywall page, instructing them to get a valid token. If the bot does have a token, we fetch the requested content securely from the publisher and deliver it, ensuring publishers get paid accordingly.

Blocking at the CDN level would only block the bot traffic without giving the bot a method to authenticate and pay for governed access.

### What kind of testing environment does TollBit provide for integrations?

We provide a flexible testing approach. Publishers or integration partners (like yourselves) can create test organizations within TollBit, set up sandbox websites, and configure edge rules pointing to these test environments.

You can also create custom test user-agents and API tokens to simulate the full request-response flow end-to-end. This makes automated integration testing straightforward and comprehensive, so you can confidently deploy integrations to production.

### Can I use another AI marketplace like TollBit in parallel?

Yes, this is dependent on how you set up your log exports and bot redirect policies within your edge servers. In addition, if you do have existing deals with other AI companies, we can work with them to manage the content access through TollBit.

## Bot management

### Do you only detect bots that declare themselves, or do you also identify those trying to remain hidden or undeclared?

We primarily focus on the big, self-identified bots, think ChatGPT, Perplexity, Claude, the well-known players. These large players generally self-identify clearly in their user agents, although that doesn't necessarily mean they're always behaving correctly.

For bots that actively hide themselves or pretend to be human browsers – like the long tail of undeclared scrapers -- we partner with specialized cybersecurity tools (such as Datadome and Human Security). Those tools have advanced fingerprinting and machine-learning algorithms that detect even the most elusive bots.

### How frequently is the bot list updated? What happens when new bots appear?

Today, we generally update our bot lists every quarter, and we communicate new crawlers through email or our quarterly reports. Ideally, publishers would then manually update their edge configurations or robots.txt files.

### Is TollBit a bot protection solution, or is it more about monetization?

We're primarily a monetization and enforcement solution rather than purely a cybersecurity play. We're not competing with products like DataDome or Human Security. Our recommendation to publishers is typically to use those types of advanced bot detection tools alongside TollBit. So, TollBit handles monetization, billing, and stronger enforcement of content usage terms, while a cybersecurity tool would handle protection from malicious, anonymous bots.

We have partnerships with both Datadome and Human Security, and integrate with their tooling seamlessly. For those interested in advanced bot detection, please reach out to your TollBit account manager.

## Monetization

### How do publishers set content pricing – manual or dynamic?

Publishers have a lot of flexibility. Today, they can set simple manual pricing rules across their content, typically mirroring their CPM or RPM ad rates. But we also support more sophisticated, dynamic pricing methods.

For instance, publishers can set pricing by categories (e.g., sports, politics), content freshness (new articles priced higher) through time-based rates, bots, or even individual pages. Over time, we envision publishers using dynamic, automated pricing based on content exclusivity or real-time demand, similar to an AdWords auction-style system.

### How does TollBit retrieve content for bots that pass the paywall?

Once the bot presents a valid TollBit token, we authorize the request and fetch the content from your site securely. Today, this happens via our internal reverse proxy:

- The request is sent to api.tollbit.com/GetContent
- We scrape the page (on your behalf) and return the content in clean Markdown, not raw HTML
- This allows agents to consume structured, legible content without ads, scripts, or layout junk

### Do publishers typically provide customized or stripped-down versions of their content specifically for AI bots?

Currently, most publishers don't provide specialized or stripped-down versions of their content. However, we're beginning to see interest from publishers in potentially serving simpler or cleaner versions specifically to bots.

### How can I bring all my licenses under the TollBit "hood"? I already have direct deals with AI players (1:1 licenses), but I also want to offer some content under a general license. How does this all fit together?

This is one of the core reasons publishers use TollBit in the first place. You can think of us as the rules engine sitting between your content and the AI ecosystem. We make it easy to enforce, meter, and report on any mix of licensing terms, whether it's bespoke or standard.

## Security

### How do you handle PII info in the logs and is there a delay?

We take PII handling and log privacy seriously. We ingest raw server-side logs to detect bot activity, and securely store all of our data.

There is a 24-hour delay between when logs are received and when analytics appear in the dashboard. Customers either stream logs to our log sink endpoint continuously or let us pull from their S3 buckets once per day (nightly). A few hours after ingestion, we run a batch process that aggregates the previous day's logs and inserts the results into our BigQuery table. Once that's done, updated analytics appear in the dashboard. This means dashboards reflect the previous full day's data, not real-time traffic.

### How do publishers ensure TollBit doesn't trigger their security alarms, given it's acting like a proxy?

We work directly with publishers to ensure our proxy servers are clearly whitelisted and identified within their security and firewall settings. Publishers explicitly approve TollBit's servers or specific user agents. This clear identification prevents accidental triggering of security alarms or rate-limiting measures.

### What is a TollBit token? How does authentication work and is it secure against token reuse or interception?

Yes, our authentication system is designed with security in mind. Every single request a bot makes has a unique, single-use token generated specifically for that content page. Once the token is used, it's immediately invalidated. Even if someone intercepts the token, they won't be able to reuse it. Additionally, this token validation happens entirely on our TollBit infrastructure, which is securely managed.

### What happens after bots pass through TollBit's authentication – do they return directly to publishers' main websites?

Not directly. When bots pass our authentication, TollBit acts as a proxy as we fetch the content directly from the publisher's infrastructure (either from their main website or a dedicated API endpoint they provide to us). The bots themselves never directly access the publisher's main website after redirection. This approach ensures a controlled environment, reduces load on publishers, and centralizes access management.

## Pricing

### What does it cost me to implement TollBit?

For publishers, access to TollBit analytics and the marketplace is free of cost. We charge developers a small percentage on top of every transaction in the marketplace.

