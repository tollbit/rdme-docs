---
title: Response Formatting
excerpt: Dive into the formatting of content as it comes from TollBit.
---

# Content Formatting: LLM Ready out of the box

We format content to most effectively integrate within AI applications and into LLM contexts. This feature comes out of the box when using TollBit. All sites onboarded with TollBit will work with this functionality.

## What does the formatted content look like?

This formatting process makes no changes to the original content. We simply clean the content for you to be perfectly ready for your data pipeline. Specifically, the data comes back as a markdown representation of the original web page. The `body` field of the `content` response will likely contain the actual content of the article without any clutter of navigational components or social media links. Should you want to use those fields, you may get them from the `header` and `footer` fields if we were able to parse them out.

Finally, the `metadata` field may contain additional information that isn't part of the original content, but can provide additional context around the content. This could include raw data, follow up link, or additional topics for the end user to explore.

The following is what a user who hits our FAQs page might see.

**Example Content Formatting** - `https://tollbit.com/faqs`

```json
{
  "content": {
    "header": "",
    "body": "# TollBit - FAQs\n\n[Get started](https://signup.tollbit.com)\n\n# FAQs\n\n[Request a demo](https://signup.tollbit.com)\n\n-\n\n## What is TollBit?\n\nTollBit is a first-of-its-kind platform to help websites ensure fair compensation for their content and data. The platform allows AI bots and data scrapers to pay websites directly, rewarding quality content creation and mitigating the legal uncertainty of scraping.\n\n-\n\n## How does the platform work?\n\nOn the supply side, TollBit's clients are companies with openly accessible websites, whose data are vulnerable to scraping. They includes publishers, sites with user-generated content, and sites that allow end users to take action - such as e-commerce sites.\n\nUsing TollBit, websites can sign up rates and rules can be set for autonomous (non-human) access to any specific URL. TollBit also provides powerful analytics and visibility to companies about autonomous traffic.\n\nOn the other side, companies doing the scraping today can use TollBit to access content and data on websites for a fee in exchange for licensing and a cleaner more digestible version of the URL page.\n\nTollBit enables websites to realize the true value of their data, which would otherwise be prone to payment-free scraping.\n\n-\n\n## Are you onboarding publishers?\n\nYes, we are onboarding publishers and partners.\n\n-\n\n## A number of publishers are cutting their own content licensing deals with tech companies - how would this platform impact the platform?\n\nTollBit is an \"and\" product. We encourage our websites to pursue 1:1 licensing deals when they make sense. TollBit can also help provide critical missing infrastructure for licensing deals, including reporting, rate limits, and authentication.\n\n-\n\n## How do you expect to set pricing/value of content?\n\nOn-demand access rates are something that will depend on the unique needs and business model of our individual clients. Publishers set their own rates on TollBit. However, private licensing deal terms are never public.\n\n-\n\n## Was this developed in concert with any publishers?\n\nThe development of the TollBit platform was informed by conversations with dozens of publishers and the product will continue to be improved with their feedback.\n\n-\n\n## Do publishers need to have a paywall in order to use TollBit to generate revenue? If information is already in the public domain then how can AI companies be expected to pay?\n\nPublishers do not have to have an existing paywall in place to generate revenue via TollBit.\n\n-\n\n## Are there restrictions on how content can be used once licensed through TollBit?\n\nYes, there are specific scopes and restrictions of the on-demand license.\n\nIf you use the content/data in a form that is not covered by the license, then the license is not valid and you do not have protection or permission for use.\n\nMade with care in Nashua, Boston, and\n\nNew York City\n\n© Novoscribe, Inc. 2024",
    "footer": ""
  },
  "metadata": {
    ...
  },
  "rate": {
    "license": {
      "id": "ji73lwbqjnhfgpu1bk3sjzj1",
      "licensePath": "<license_url>",
      "licenseType": "ON_DEMAND_LICENSE",
      "permissions": [
        {
          "name": "PARTIAL_USE"
        }
      ]
    },
    "price": {
      "currency": "USD",
      "priceMicros": 5000
    }
  }
}
```

As you can see, none of the original content is affected in any way. Just some trimming down of the extra HTML!

## Why is this format good for AI?

This formatting maintains crucial context for the LLM like titles, paragraphs, links, etc. At the same time, this format strips away the excessive HTML tags, scripts and other clutter that comes back from scraping typical websites. This format should optimize the value in the content, while being efficient to how many tokens you pass to the LLM.

