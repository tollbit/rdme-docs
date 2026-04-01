---
title: Arc XP
excerpt: Learn how to integrate TollBit with Arc XP.
deprecated: false
hidden: false
metadata:
  robots: index
---
# Arc XP

This integration supports TollBit's AI bot monitoring, management and monetization capabilities as well as seamless connections with MCP and NLWeb.

Follow these steps to set up an integration into our platform if you use Arc XP.

## Setup Analytics with Arc XP

Submit an ACS ticket to the ARC XP team to request the log ingestion for each site where you would like to enable for TollBit analytics.

**ACS Ticket Example:**

* **Org**: Your Org Name
* **Sites**: example1.com, example2.com
* **Environment**: Prod
* **Change type**: Add
* **Sample %**: 100
* **Push Frequency**: 30 seconds
* **Integration type**: Direct feed from CDN to TollBit

Once the ticket is submitted, the Arc XP team will configure then confirm the analytics integration is complete. Please wait up to 24 hours for the data to populate within the TollBit dashboard.

## Setup Agent Site with Arc XP

You can configure the edge integration within the Arc Delivery UI. Please follow these steps for each site you are looking to set up the bot redirect for.

1. Open Delivery UI, and go to Edge Integrations for the chose site

<Image align="center" src="https://files.readme.io/6f3b959d61fe5e08c5ba6211b31c0fa568f256645e7ad44e95517547749299ab-Screenshot_2026-04-01_at_12.48.12_PM.png" />

1. Select **View** for the site you wish to configure, and then select **Edge Integrations**

<Image align="center" src="https://files.readme.io/b7686e9fe841ecc27558366572c8d22b90c0773b65755e9e3922ad270ef72b11-Screenshot_2026-04-01_at_12.48.54_PM.png" />

1. Scroll down to the TollBit integration and enable the toggle on the righthand side. Enabling the toggle (without adding any user agent string in the text bar) will allow all your AI bots to be sent to the TollBit Agent Site. This will allow you to manage the bot paywall or whitelists directly within TollBit UI.

<Image align="center" src="https://files.readme.io/4a73b6b37de2b649234b20bdb388b270cd3bb32fe8b971cc113cd994fa5dee54-Screenshot_2026-04-01_at_12.50.13_PM.png" />

**Note**: the Override list text entry is there if you **don't** want to allow any of the following AI agents to be sent to the TollBit subdomain. Please use commas to separate multiple user agents. The default list includes the following bots:

_chatgpt-user, perplexitybot, gptbot, anthropic-ai, ccbot, claude-web, claudebot, cohere-ai, youbot, diffbot, oai-searchbot, meta-externalagent, timpibot, amazonbot, bytespider, perplexity-user_
