---
title: 'Step 2: Enable Analytics'
excerpt: Implementation steps to enable logging to TollBit
deprecated: false
hidden: false
metadata:
  robots: index
---
Once your property is verified, the next step is connecting TollBit to your server-side logs so TollBit can start surfacing AI bot traffic data for your site.

# Introduction

After analytics is enabled TollBit gives you visibility into:

- Scrapes vs. visits — how often AI platforms like ChatGPT and Perplexity are crawling your content versus how often users actually click through to your site
- Traffic by agent — bot activity broken down by user agent (e.g. GPTBot, ClaudeBot, PerplexityBot)
- Referral metrics — where your bot traffic is coming from and how it's trending over time
- And more

# How It Works

TollBit analytics runs on server-side logs from your CDN or edge layer — not client-side JavaScript. This is important: most bots don't execute JavaScript, so client-side tracking misses the majority of AI crawler activity. Server-side logs give you a clean, complete view of everything hitting your site.

<br />TollBit supports integrations with the most common CDN and cloud platforms, including Cloudflare, AWS (ALB and S3), and others.

# Next Steps

<br />For setup instructions specific to your infrastructure, see the full integration guide

## Available Integrations

Choose your platform below to view detailed integration instructions. Each integration guide includes step-by-step instructions for setting up both analytics and bot paywall features.

### CDN & Edge Platforms

- **[Fastly](fastly)** - CDN integration with UI-based setup and VCL script options
- **[CloudFlare](cloudflare)** - Worker-based integration available for all plans (Free, Pro, Business, Enterprise)
- **[Akamai](akamai)** - DataStream 2 for analytics and Cloudlets for bot paywall
- **[Vercel](vercel)** - Log drains for analytics and WAF rules for bot paywall

### Cloud Providers

- **[Amazon (AWS)](aws)** - ALB, CloudFront, and WAF integration options
- **[Google (GCP)](google)** - Cloud Load Balancer for analytics and Cloud Armor for bot paywall
- **[Microsoft (Azure)](azure)** - Front Door integration for both analytics and bot paywall

### Security & Platform Integrations

- **[Datadome](datadome)** - Bot paywall integration (analytics via other methods)
- **[WordPress VIP](wordpress-vip)** - Platform integration for bot paywall
- **[Imperva](imperva)** - Log forwarding and bot paywall configuration

### Other Methods

- **[Other Methods](other)** - Log sink API endpoint and file storage (S3, R2, GCS) options

<br />If you run into any issues during setup, reach out to [team@tollbit.com](mailto:team@tollbit.com) and the TollBit team can help get your logs connected.
