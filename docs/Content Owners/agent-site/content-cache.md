---
title: Enabling Open Access
excerpt: >-
  Choose whether or not you'd like to provide free or paywalled access to your
  content.
deprecated: false
hidden: false
metadata:
  robots: index
---
Open Access is a routing mode that serves cached HTML to whitelisted bots or page categories without charging them. It creates a sanctioned, cost-neutral access path for AI agents while protecting publishers and site content from overwhelming bot traffic.

To use this feature to its fullest extent, start by redirecting bots from your main site to your agent site (`tollbit` subdomain); you can view how to do this in your CDN via our <Anchor target="_blank" href="/docs/integrations">integrations</Anchor> page. AI bots would continue to be able to fetch your content, but it would be doing so through your agent site instead of your human site. TollBit intelligently routes the request based on specified preferences. Simply whitelist agents and their crawling automatically becomes sanctioned through the cache.

Select a TollBit property, and under Agent Site select Open Access on the top of the page. You may either select all AI bots to use content cache or select specific bots by selecting them in the table or searching/adding a string.

![](https://files.readme.io/e709be393955ab0876312d1450fd109ad3a028d721569b10105c27038a2ec676-Screenshot_2026-06-25_at_12.15.20_PM.png)

Once you add open access, you can configure open access by either selecting a content path (via dropdown or text entry) or user agent. In the example below, all articles under the Opinion / Editorials folder path have been enabled for ChatGPT-User, as an example.

![](https://files.readme.io/7242a582633eae7bfdd260036703290c92b273e7541706f87a5fb3c81fb2884d-Screenshot_2026-06-25_at_12.21.22_PM.png)

<br />

<br />

**Caching**

Note that Open Access is not a permanent archive. Pages are stored depending on whether they are requested by bots. The TTL is set to 5 minutes for non article pages (home page, hub pages, etc), and longer for articles, after which the cached content is automatically evicted.
