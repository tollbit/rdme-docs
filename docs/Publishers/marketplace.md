---
title: Marketplace
excerpt: Learn how to price and make your content available in the TollBit marketplace.
---

# Marketplace

## Requirements

To participate in the TollBit marketplace, you will need to ensure your TollBit subdomain is provisioned and that AI bot traffic is being redirected to hit your bot paywall.

For details on how to implement the bot paywall, please review the Bot Paywall steps under the appropriate integration within your tech stack.

## Setting Rates

This is the core of the TollBit product, and where you can set prices on your content. At the moment there are a few ways to set rates. You can set global rates which apply to all your content across all subdirectories and pages. There are two types of licenses, rates should be established for both:

1. **Summarization License** - Allows AI customers to access your content to create a summary, grounding, or citation with a single use license. Simply, set your rate per 1000 pages accessed and click activate and allow AI customers interested to view rates and pay for your content, you can consider your current RPM as a benchmark when determining what value to set as your Summarization Rate
2. **Full Display License** - Allows AI customers to display the complete text of an article once. Set your rate per 1000 pages accessed and click activate to begin generating revenue, you can consider your syndication rates as a benchmark when determining this rate.

Within both types of licenses you can also set custom rates according to the following hierarchy: `bot` -> `page` -> `keyword` -> `time` -> `subdirectories`. This means that when determining the price of a page for a particular request, we first check if that request is from a bot that matches any of your bot rates. If so, we return that rate. If there are no bot matches, we then check if the requested page matches any of your page rates. We keep going down the chain, trying to find a match, and if we find no matches at the end, the content cannot be accessed by AI customers.

TollBit doesn't take a percentage of your rates or revenue share. We simply charge AI customers a small transaction fee on top of the rates you set. Your payments reflect your rates completely; if you set a rate of $0.001 per page, AI customers will pay that and you will receive that amount completely.

## Customizations for Rates

### Bot Rates

These rates allow you to set special rates for any specific bots that access your platform, and will override all other rates. You should set this type of rate if you have struck a licensing deal with a company that employs a particular user agent, and want to give them special rates to access your content (usually 0).

### Page Rates

These rates allow you to set a rate for a specific page on your website. If you have any page that you know gets high bot traffic (i.e. sports or election results), or if you have a very high quality piece of original reporting, you can set a special rate for that page. This will override all other rates except bot rates.

### Keyword Rates

These rates allow you to set a price for pages that may contain a particular keyword. If you know that there are some high profile sporting events coming up, you may want to set a higher price for pages that mention `football` or `basketball`. This rate is still in beta.

### Directory Rates

These rates let you set a flat fee for all the content within a page directory of your site. For a quick way to instantly price your content, you can set a price for your top level directory, and this will automatically apply to all pages. You can drill down into further subdirectories and set pricing there, and it will override any price in a higher directory. For example, you can set a base price of $0.001 at the root level, and then set a price of $0.005 for the `/sports` directory. Everything under `/sports` will now be $0.005 while something under `/cooking` will still be $0.001.

## Standard Licenses

The TollBit platform has two standard licenses.

1. **"Summarization" license** - At a high level, this allows AI customers to use a piece of your content once as part of a summary, citation, or grounding.
2. **"Full Display" license** - At a high level, this allows AI customers to display the content in its original format within their application once. Neither license allows for AI customers to utilize your content for training or creating generative AI models.

When activating rates, you are agreeing to the standard license terms. These license terms will apply each time an AI company uses your content. You can review the full licenses within your portal on the Rates page by clicking the question mark next to the license name, and rates can be independently set and activated per standard license.

<Callout icon="ℹ️" theme="info">
  When you first onboard onto the platform, your rates will *not* be active and demand side users will not be able to fetch your content through TollBit. In order to activate your rates and price your content, click the "Activate" button on the top of the rates page. Once this is active, all your rates will be live.
</Callout>

## Custom Licenses

For any partners that you have struck deal with, you can upload a custom license to the user agents of that specific partner. Any requests made to TollBit with that partner's user agents will include the license that you uploaded in the transactions.

## Transactions

This page provides an audit trail where you can see all the requests that have been made to your website through TollBit. For each request, you are able to see the user agent that made the request, the page they hit, and the price they paid for that page.

## Content Controls

Control what data to include or exclude when developers request content from your website through TollBit.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/rdme-docs/public/content-controls-page.png" alt="Content Controls Page" />

### Element Filters

Element filters help exclude certain types of content from the HTML of your website, such as removing all images, links or embedded content from being returned to agentic consumers. Note that in order for this feature to work properly, the content to be excluded needs to be written in well formatted HTML. For example, we won't be able to exclude a hyperlink if it's not within an `<a>` tag.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/rdme-docs/public/content-controls-element-filter-example.png" alt="Content Controls Element Filter Example" />

For more advanced use cases you can exclude all elements that are styled with a specific CSS class.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/rdme-docs/public/content-controls-element-filter-example-class.png" alt="Content Controls Element Filter Example Class" />

### Article Filters

Article filters allow you to programmatically exclude entire HTML pages from AI access. An article filter can be configured to match a specific HTML tag with specific attributes and attribute values. Any page that contains the tag that matches will be excluded from agentic access. This feature is intended to be used to exclude content like syndicated articles.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/rdme-docs/public/content-controls-article-filter-create.png" alt="Content Controls Article Filter Create" />

If you already have a `<meta name='robots' content='noindex,nofollow'>` tag (or similar) to exclude pages from traditional web crawling, configuring an article filter to match that tag will also exclude pages containing that tag from AI access.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/rdme-docs/public/content-controls-article-filter-example-meta.png" alt="Content Controls Article Filter Example Meta" />

## Content Formatting

We format content to most effectively integrate within AI applications and into LLM contexts. This feature comes out of the box when using TollBit. All sites onboarded with TollBit will work with this functionality.

### What does the formatted content look like?

This formatting process makes no changes to the original content. We simply clean the content for you to be perfectly ready for your data pipeline. Specifically, the data comes back as a markdown representation of the original web page. The `main` field of the `content` response will likely contain the actual content of the article without any clutter of navigational components or social media links. Should you want to use those fields, you may get them from the `header` and `footer` fields if we were able to parse them out.

Finally, the `metadata` field may contain additional information that isn't part of the original content, but can provide additional context around the content. This could include raw data, follow up link, or additional topics for the end user to explore.

The following is what a user who hits our FAQs page might see.

```json
{
  "content": {
    "header": "",
    "body": "# TollBit - FAQs\n\n[Get started](https://signup.tollbit.com)\n\n# FAQs\n\n[Request a demo](https://signup.tollbit.com)\n\n-\n\n## What is TollBit?\n\nTollBit is a first-of-its-kind platform to help websites ensure fair compensation for their content and data. The platform allows AI bots and data scrapers to pay websites directly, rewarding quality content creation and mitigating the legal uncertainty of scraping.\n\n-\n\n## How does the platform work?\n\nOn the supply side, TollBit's clients are companies with openly accessible websites, whose data are vulnerable to scraping. They includes publishers, sites with user-generated content, and sites that allow end users to take action - such as e-commerce sites.\n\nUsing TollBit, websites can sign up rates and rules can be set for autonomous (non-human) access to any specific URL. TollBit also provides powerful analytics and visibility to companies about autonomous traffic.\n\nOn the other side, companies doing the scraping today can use TollBit to access content and data on websites for a fee in exchange for licensing and a cleaner more digestible version of the URL page.\n\nTollBit enables websites to realize the true value of their data, which would otherwise be prone to payment-free scraping.\n\n-\n\n## Are you onboarding publishers?\n\nYes, we are onboarding publishers and partners.\n\n-\n\n## A number of publishers are cutting their own content licensing deals with tech companies - how would this platform impact the platform?\n\nTollBit is an \"and\" product. We encourage our websites to pursue 1:1 licensing deals when they make sense. TollBit can also help provide critical missing infrastructure for licensing deals, including reporting, rate limits, and authentication.\n\n-\n\n## How do you expect to set pricing/value of content?\n\nOn-demand access rates are something that will depend on the unique needs and business model of our individual clients. Publishers set their own rates on TollBit. However, private licensing deal terms are never public.\n\n-\n\n## Was this developed in concert with any publishers?\n\nThe development of the TollBit platform was informed by conversations with dozens of publishers and the product will continue to be improved with their feedback.\n\n-\n\n## Do publishers need to have a paywall in order to use TollBit to generate revenue? If information is already in the public domain then how can AI companies be expected to pay?\n\nPublishers do not have to have an existing paywall in place to generate revenue via TollBit.\n\n-\n\n## Are there restrictions on how content can be used once licensed through TollBit?\n\nYes, there are specific scopes and restrictions of the on-demand license.\n\nIf you use the content/data in a form that is not covered by the license, then the license is not valid and you do not have protection or permission for use.\n\nMade with care in Nashua, Boston, and\n\nNew York City\n\n© Novoscribe, Inc. 2024",
    "footer": ""
  },
  "metadata": "",
  "rate": {
    "priceMicros": 20000,
    "currency": "USD",
    "licenseType": "ON_DEMAND_LICENSE",
    "licensePath": "...",
    "error": ""
  }
}
```

As you can see, none of the original content is affected in any way. Just some trimming down of the extra HTML!

### Why is this format good for AI?

This formatting maintains crucial context for the LLM like titles, paragraphs, links, etc. At the same time, this format strips away the excessive HTML tags, scripts and other clutter that comes back from scraping typical websites. This format should optimize the value in the content, while being efficient to how many tokens you pass to the LLM.

