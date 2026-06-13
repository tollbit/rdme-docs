---
title: Agent Sites
excerpt: Learn the features and controls behind Agent Site
deprecated: false
hidden: false
metadata:
  robots: index
---
## Requirements

You will need to ensure your `tollbit` subdomain is provisioned and that AI bot traffic is being redirected to hit your bot paywall. For details on how to implement the bot paywall, please review the Bot Paywall steps under the appropriate integration within your tech stack.

# Introduction


Agent Site gives you a dedicated subdomain — **tollbit.yourdomain.com** — built specifically for AI visitors. Rather than letting bots scrape your human-facing site, Agent Site serves a machine-optimized version of your content: semantically clean, efficiently structured, and designed for how agents actually read and act on the web.


This separation matters for a few reasons. Human websites are full of layout code, scripts, and UI elements that are meaningless to agents — they waste tokens parsing noise instead of retrieving content. As agents move beyond simple retrieval into completing workflows, that same bloat causes brittle, unpredictable behavior. Agent Site removes that friction.


At the same time, you stay in control. You decide which agents can access your site, what they can see, and what they pay for it. Agent traffic is routed away from your main site, keeping your CDN costs predictable and your human experience unaffected.


The sections below cover everything you need to configure Agent Site for your use case:

- Default configs — what's set up out of the box and how content is served to agents by default
- Free vs. paid access — how to open access to some agents while paywalling others
- Content controls — tailoring what different agents can see and how requests are handled
- Monetization — setting rates, license types, and connecting to the TollBit Marketplace
- Access methods — enabling MCP, NLWeb, and Agent2Agent protocols on your Agent Site

<br />
