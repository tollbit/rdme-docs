---
title: 'Step 3: Configure Agent Site'
excerpt: >-
  Steps to configure agent site within TollBit dashboard and upstream
  configurations
deprecated: false
hidden: false
metadata:
  robots: index
---
With analytics running, the final step is setting up your Agent Site — a dedicated subdomain optimized for AI agents, separate from your human-facing website.

# What is an Agent Site?

<br />Your Agent Site lives at **tollbit.yourdomain.com**. Rather than letting AI crawlers scrape your main site, this subdomain serves as a purpose-built endpoint for agent traffic — returning content in formats like Markdown that are optimized for how AI systems actually consume it.

This gives you a clean separation between your human experience and your agent experience, while putting you in control of what bots see, how they access it, and what they pay for it.

# What you can configure

- Access controls — define which agents are allowed in, which are blocked, and which are subject to your paywall
- Content tailoring — serve different content to different agents based on their user agent string
- Paywall enforcement — optionally gate access so bots must have a valid TollBit token to retrieve content

# How it works

<br />TollBit provisions the tollbit.yourdomain.com subdomain and handles authentication, licensing, and content delivery from there. When an agent requests a page, Agent Site returns content formatted for that agent — no changes needed to your main site.

<br />TollBit also supports additional agentic protocols including MCP, NLWeb, and Agent2Agent (A2A), which can be enabled from the same setup.

<br />

# Next Steps

For full setup instructions, see the technical integration guide →

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

<br />
