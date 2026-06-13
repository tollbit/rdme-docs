---
title: Content Paywall
excerpt: Introduction to rate types and how to activate them on TollBit
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

## Introduction

This is the core of the TollBit product, and where you can set prices on your content. At the moment there are a few ways to set rates. You can set global rates which apply to all your content across all subdirectories and pages. There are two types of licenses, rates should be established for both:

1. **Summarization License** - Allows AI customers to access your content to create a summary, grounding, or citation with a single use license. Simply, set your rate per 1000 pages accessed and click activate and allow AI customers interested to view rates and pay for your content, you can consider your current RPM as a benchmark when determining what value to set as your Summarization Rate
2. **Full Display License** - Allows AI customers to display the complete text of an article once. Set your rate per 1000 pages accessed and click activate to begin generating revenue, you can consider your syndication rates as a benchmark when determining this rate.

Within both types of licenses you can also set custom rates according to the following hierarchy: `bot` -> `page` -> `keyword` -> `time` -> `subdirectories`. This means that when determining the price of a page for a particular request, we first check if that request is from a bot that matches any of your bot rates. If so, we return that rate. If there are no bot matches, we then check if the requested page matches any of your page rates. We keep going down the chain, trying to find a match, and if we find no matches at the end, the content cannot be accessed by AI customers.

TollBit doesn't take a percentage of your rates or revenue share. We simply charge AI customers a small transaction fee on top of the rates you set. Your payments reflect your rates completely; if you set a rate of $0.001 per page, AI customers will pay that and you will receive that amount completely.

<br />

## Other Types of Rates

### Bot Rates

These rates allow you to set special rates for any specific bots that access your platform, and will override all other rates. You should set this type of rate if you have struck a licensing deal with a company that employs a particular user agent, and want to give them special rates to access your content (usually 0).

### Page Rates

These rates allow you to set a rate for a specific page on your website. If you have any page that you know gets high bot traffic (i.e. sports or election results), or if you have a very high quality piece of original reporting, you can set a special rate for that page. This will override all other rates except bot rates.

### Keyword Rates

These rates allow you to set a price for pages that may contain a particular keyword. If you know that there are some high profile sporting events coming up, you may want to set a higher price for pages that mention `football` or `basketball`. This rate is still in beta.

### Directory Rates

These rates let you set a flat fee for all the content within a page directory of your site. For a quick way to instantly price your content, you can set a price for your top level directory, and this will automatically apply to all pages. You can drill down into further subdirectories and set pricing there, and it will override any price in a higher directory. For example, you can set a base price of $0.001 at the root level, and then set a price of $0.005 for the `/sports` directory. Everything under `/sports` will now be $0.005 while something under `/cooking` will still be $0.001.