---
title: Content Cache
excerpt: >-
  Control your CDN costs through Content Cache. You allow all bots to use the
  cache, or explicitly allow or block bots individually.
deprecated: false
hidden: false
metadata:
  robots: index
---
Content Cache is a routing mode that serves cached HTML to whitelisted bots without charging them. It creates a sanctioned, cost-neutral access path for AI agents while protecting publishers from overwhelming bot traffic.

This feature requires no changes to how bots currently access content. AI bots would continue hitting your site, but TollBit intelligently routes the request based on specified preferences. Simply whitelist agents and their crawling automatically becomes sanctioned through the cache.

Select a TollBit property, and under Web Content select Content cache on the top of the page. You may either select all AI bots to use content cache or select specific bots by selecting them in the table or searching/adding a string.

<Image align="center" src="https://files.readme.io/202bc477ae99d046e17ad8c6b86804eca0bdf100d9a225142df93f0c4d0870e3-Screenshot_2026-02-27_at_2.54.05_AM.png" />

In summary, a redrected bot can fall in 2 categories:

* Whitelisted → Check cache → Serve from cache or fetch once and cache
* Not whitelisted → Require payment token or block

Note that Content Cache is not a permanent archive. Pages are stored depending on whether they are requested by bots. The TTL is set to 5 minutes by default, after which the cached content is automatically evicted.
