---
title: Monetization
excerpt: Questions about pricing, content retrieval, and licensing.
---

# Monetization

## How do publishers set content pricing – manual or dynamic?

Publishers have a lot of flexibility. Today, they can set simple manual pricing rules across their content, typically mirroring their CPM or RPM ad rates. But we also support more sophisticated, dynamic pricing methods.

For instance, publishers can set pricing by categories (e.g., sports, politics), content freshness (new articles priced higher) through time-based rates, bots, or even individual pages. Over time, we envision publishers using dynamic, automated pricing based on content exclusivity or real-time demand, similar to an AdWords auction-style system.

## How does TollBit retrieve content for bots that pass the paywall?

Once the bot presents a valid TollBit token, we authorize the request and fetch the content from your site securely. Today, this happens via our internal reverse proxy:

- The request is sent to `api.tollbit.com/GetContent`
- We scrape the page (on your behalf) and return the content in clean Markdown, not raw HTML
- This allows agents to consume structured, legible content without ads, scripts, or layout junk

## Do publishers typically provide customized or stripped-down versions of their content specifically for AI bots?

Currently, most publishers don't provide specialized or stripped-down versions of their content. However, we're beginning to see interest from publishers in potentially serving simpler or cleaner versions specifically to bots.

## How can I bring all my licenses under the TollBit "hood"? I already have direct deals with AI players (1:1 licenses), but I also want to offer some content under a general license. How does this all fit together?

This is one of the core reasons publishers use TollBit in the first place. You can think of us as the rules engine sitting between your content and the AI ecosystem. We make it easy to enforce, meter, and report on any mix of licensing terms, whether it's bespoke or standard.
