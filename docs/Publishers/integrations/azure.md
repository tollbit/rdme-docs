---
title: Microsoft (Azure)
excerpt: Learn how to integrate TollBit with Microsoft.
---
### Setting up Logging with Azure Front Door

To set up logging for your Azure front door, first navigate to your specific Front Door instance.
On the left sidebar, open the dropdown for "Monitoring" and select the "Diagnostics settings" tab.

![Azure Front Door Monitoring](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-monitoring.png)

This will take you to a screen where you can create a new Diagnostics Setting.

![Azure Front Door Diagnostic](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-diagnostic.png)

Go ahead and create a new setting. Within, the settings page, selectthe options that will
send all access logs to a Storage account. If you don't already have a storage bucket in place
for these logs, please create one.

![Azure Front Door Log Settings](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-log-settings.png)

Once you've confirmed that logs are setting stored in your chosen bucket, please reach out to
[team@tollbit.com](mailto:team@tollbit.com) to coordinate with our engineering team on the rest of the TollBit
Analytics setup. To finalize your setup, we will need
access to the directory in your Storage bucket where your logs are stored, along with
the pattern for how the logs are stored for instance
`/service/logs/2024/12/04/log-file`.

### Route to Agent Site

Azure's CDN lets you easily set up redirection rules for different bots. We'll explore how to do this
in the standard tier Front Door as well as the premium tier.

### Standard

Navigate to your Front Door instance and click the dropdown for "Settings" on the left navbar.

![Azure Front Door Rule Set](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-rule-set.png)

You can create a new rule set using the button at the top.

![Azure Front Door Add Rule](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-add-rule.png)

Then you can add rules for the bots you want to forward off to the TollBit Bot Paywall. We
recommend the following list:

```
chatgpt-user, perplexitybot, gptbot, anthropic-ai, ccbot, claude-web, claudebot, cohere-ai, youbot, diffbot, oai-searchbot, meta-externalagent, timpibot, amazonbot, bytespider, perplexity-user
```

You can add or remove from this as befits your bot strategy. We can also keep these lowercased, since in the rules we are comparing the lowercased values.
Click save to save these changes.

![Azure Front Door Bot Rules](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-bot-rules.png)

Once you've created this ruleset, you need to associate your Front Door route to it.
Click the 3 horizontal dots to the right of the rule set, and click "Associate a route".

![Azure Front Door Associate Route](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-associate-route.png)

Choose the route to the relevant Front Door instance and go through the flow of associating this route.

### Premium

If you have a Premium tier Front Door instance, contact our team at [team@tollbit.com](mailto:team@tollbit.com) and we can
connect with you and evalulate the best path forward.
