---
title: Free & Paid Access
excerpt: >-
  Choose whether or not you'd like to provide free or paywalled access to your
  content.
deprecated: false
hidden: false
metadata:
  robots: index
---
Content Cache is a routing mode that serves cached HTML to whitelisted bots without charging them. It creates a sanctioned, cost-neutral access path for AI agents while protecting publishers from overwhelming bot traffic.

To use this feature to its fullest extent, start by redirecting bots from your main site to your agent site (`tollbit` subdomain); you can view how to do this in your CDN via our <Anchor label="integrations" target="_blank" href="/docs/integrations">integrations</Anchor> page. AI bots would continue to be able to fetch your content, but it would be doing so through your agent site instead of your human site. TollBit intelligently routes the request based on specified preferences. Simply whitelist agents and their crawling automatically becomes sanctioned through the cache.

Select a TollBit property, and under Web Content select Content cache on the top of the page. You may either select all AI bots to use content cache or select specific bots by selecting them in the table or searching/adding a string.

<Image align="center" src="https://files.readme.io/202bc477ae99d046e17ad8c6b86804eca0bdf100d9a225142df93f0c4d0870e3-Screenshot_2026-02-27_at_2.54.05_AM.png" />

In summary, a redrected bot can fall in 2 categories:

* Whitelisted → Check cache → Serve from cache or fetch once and cache
* Not whitelisted → Require payment token or block

Note that Content Cache is not a permanent archive. Pages are stored depending on whether they are requested by bots. The TTL is set to 5 minutes for non article pages (home page, hub pages, etc), and longer for articles, after which the cached content is automatically evicted.