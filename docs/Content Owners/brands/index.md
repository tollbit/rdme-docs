---
title: Brands
excerpt: Onboarding guides written specifically for brands setting up TollBit.
deprecated: false
hidden: true
metadata:
  robots: noindex
---
This section covers onboarding for brands. The rest of our documentation is written for publishers, who use TollBit to license and monetize their content. Brands use TollBit differently: to see how AI agents are accessing their site, and to serve those agents content that is written for them through an Agent Site on a `tollbit` subdomain. The guides here follow the same integration points as the publisher docs, but leave out the steps that do not apply to brands.

Each guide is complete on its own, so everything you need for a setup is on one page. A guide covers streaming logs to TollBit for analytics, routing AI agents to your Agent Site, and routing visitors who click through from cited agent content back to a destination on your site.

## Guides

- [AWS CloudFront for Brands](aws-cloudfront-brands). Analytics, Agent Site, and visitor routing from cited content for sites served through Amazon CloudFront.
- [Akamai for Brands](akamai-brands). Analytics, Agent Site, and visitor routing from cited content for sites served through Akamai.
- [Cloudflare for Brands](cloudflare-brands). Analytics, Agent Site, and visitor routing from cited content for sites served through Cloudflare, using a single Worker, with Logpush as an analytics option on the Enterprise plan.
- [Fastly for Brands](fastly-brands). Analytics, Agent Site, and visitor routing from cited content for sites served through Fastly, using dynamic VCL snippets.
