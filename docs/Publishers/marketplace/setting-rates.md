---
title: Setting Rates
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

This is the core of the TollBit product, and where you can set prices on your content. At the moment there are a few ways to set rates. You can set global rates which apply to all your content across all subdirectories and pages. There are two types of licenses, rates should be established for both:

1. **Summarization License** - Allows AI customers to access your content to create a summary, grounding, or citation with a single use license. Simply, set your rate per 1000 pages accessed and click activate and allow AI customers interested to view rates and pay for your content, you can consider your current RPM as a benchmark when determining what value to set as your Summarization Rate
2. **Full Display License** - Allows AI customers to display the complete text of an article once. Set your rate per 1000 pages accessed and click activate to begin generating revenue, you can consider your syndication rates as a benchmark when determining this rate.

Within both types of licenses you can also set custom rates according to the following hierarchy: `bot` -> `page` -> `keyword` -> `time` -> `subdirectories`. This means that when determining the price of a page for a particular request, we first check if that request is from a bot that matches any of your bot rates. If so, we return that rate. If there are no bot matches, we then check if the requested page matches any of your page rates. We keep going down the chain, trying to find a match, and if we find no matches at the end, the content cannot be accessed by AI customers.

TollBit doesn't take a percentage of your rates or revenue share. We simply charge AI customers a small transaction fee on top of the rates you set. Your payments reflect your rates completely; if you set a rate of $0.001 per page, AI customers will pay that and you will receive that amount completely.
