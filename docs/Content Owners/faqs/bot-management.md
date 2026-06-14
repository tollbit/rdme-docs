---
title: Agent Site
excerpt: >-
  Questions about bot detection, identification, and management with TollBit
  Agent Site
---
# Bot Management

## Do you only detect bots that declare themselves, or do you also identify those trying to remain hidden or undeclared?

We primarily focus on the big, self-identified bots, think ChatGPT, Perplexity, Claude, the well-known players. These large players generally self-identify clearly in their user agents, although that doesn't necessarily mean they're always behaving correctly.

For bots that actively hide themselves or pretend to be human browsers – like the long tail of undeclared scrapers -- we partner with specialized cybersecurity tools (such as Datadome and Human Security). Those tools have advanced fingerprinting and machine-learning algorithms that detect even the most elusive bots.

## How frequently is the bot list updated? What happens when new bots appear?

Today, we generally update our bot lists every quarter, and we communicate new crawlers through email or our quarterly reports. Ideally, publishers would then manually update their edge configurations or robots.txt files.

## Is TollBit a bot protection solution, or is it more about monetization?

We're primarily a monetization and enforcement solution rather than purely a cybersecurity play. We're not competing with products like DataDome or Human Security. Our recommendation to publishers is typically to use those types of advanced bot detection tools alongside TollBit. So, TollBit handles monetization, billing, and stronger enforcement of content usage terms, while a cybersecurity tool would handle protection from malicious, anonymous bots.

We have partnerships with both Datadome and Human Security, and integrate with their tooling seamlessly. For those interested in advanced bot detection, please reach out to your TollBit account manager.

<br />

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
