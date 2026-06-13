---
title: Integrations
excerpt: >-
  Technical documents on how to configure Analytics and Agent Site by
  integration type.
---
We provide integrations with a variety of different platforms to enable analytics and bot forwarding to your `tollbit` subdomain.

## Integrating Analytics with TollBit

We provide a way for you to forward logs to our platform so that we can provide analytics on bot traffic and more. This provides a great way to consolidate your logs and gain fast insights about what your bot traffic looks like. It is essential that the logs forwarded to us are server side logs from your CDN/edge, as client side javascript plugins are only triggered if the javascript is run, which most bots do not do. Server side logs will ensure that we have the cleanest view of the traffic hitting your site.

## Setting up Bot Paywall with TollBit

Once you have TollBit set up for your website, you are now able to set up bot deterrence settings on your existing cloud cybersecurity platform to forward known bot traffic to your new `tollbit` subdomain.

At a high level, you are simply modifying your existing bot blocking solution to, instead of returning an error response if it detects a bad bot, to instead forward that traffic over to us through your `tollbit` subdomain.

The example solutions we provide here assume that you currently do not have bot detection and blocking in place. It should be straightforward to use these examples to understand how you can update your current blocking solutions to instead forward detected bots to your `tollbit` subdomain. Forwarded bots will see a message like the following:

```json
{
  "message": "You are not authorized to access this content without a valid TollBit Token. Please follow this URL to find out more.",
  "url": "https://tollbit.com"
}
```

<br />
