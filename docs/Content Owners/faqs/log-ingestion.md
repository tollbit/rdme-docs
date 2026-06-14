---
title: Analytics
excerpt: Frequently Asked Questions about Log Ingestion & Analytics
deprecated: false
hidden: false
metadata:
  robots: index
---
## How do I send IP addresses in my request logs? IP address logging is supported across all major CDN integrations. 

Follow the instructions for your setup below. 

**Fastly**

* Standard method — IP address fields are included automatically. No changes needed. 
* Custom method — Replace your current log format in the HTTPS endpoint with the updated format documented [here](https://docs.tollbit.com/docs/fastly#enable-analytics-1).

**Cloudflare**

* Enterprise (LogPush) — In your LogPush settings, navigate to the Fields step and enable ClientIP. 
* Free, Pro, or Business (Workers) — Update your Worker script to the latest version documented [here](https://docs.tollbit.com/docs/cloudflare#free-pro-or-business-plan-customers). Note: you will need to re-add your TollBit API key to the script after updating.

**Akamai** 

In your DataStream settings, add cliIP to your Data Parameters, then click Activate. 

**AWS (CloudFront) **

Ensure Standard Logging is enabled on your CloudFront distribution. See AWS documentation [here](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging.html#set-up-standard-logging) for setup instructions.

<br />

## I just enabled log ingestion. Why are my charts not loading?

It may take up to 24 hours for the data to populate. All charts are updated at 5am ET daily.