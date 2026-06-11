---
title: Log Ingestion
excerpt: Frequently Asked Questions about Log Ingestion
deprecated: false
hidden: false
metadata:
  robots: index
---
### I would like to send IP addresses in the request calls I'm sending. How do I enable this? 

You can log IP addresses depending on the CDN you are using. Please see the notes for our key integrations below:

1. **Fastly** - The Standard method would automatically allow you to send IP address fields to TollBit. However, if you are using the Custom method, please remove your current log format within your HTTPS endpoint and paste the one linked [here](https://docs.tollbit.com/docs/fastly#enable-analytics-1) instead.
2. **CloudFlare** - If you are on the Enterprise plan and using LogPush to send TollBit logs, go under your LogPush settings and under the Fields step, enable 'ClientIP'. If you are not on Enterprise and using Workers to send logs to TollBit, please update your Worker code to the one documented [here](https://docs.tollbit.com/docs/cloudflare#free-pro-or-business-plan-customers). Note you'll need to add back your TollBit API key in the script.
3. **Akamai** - Go under your Akamai DataStream settings, and update your Data Parameters to add in "cliIP". Once added, please click on Activate.
4. **AWS (and CloudFront**) - Ensure you have selected Standard Logging within your CloudFront distribution. Docs [here](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging.html#set-up-standard-logging) for details.

<br />

### I just enabled log ingestion. Why are my charts not loading?

It may take up to 24 hours for the data to populate. All charts are updated at 5am ET daily.
