---
title: Microsoft (Azure, Front Door)
excerpt: Learn how to integrate TollBit with Microsoft.
---
# Allow TollBit's IP Addresses

<Callout icon="🚧" theme="warn">
  ### Required

  TollBit must be able to reach your sites. We request your pages to set up your Agent Site. If our requests are blocked, challenged, or rate-limited, your Agent Site will not work correctly.
</Callout>

TollBit sends these requests from a fixed set of static IP addresses, published at [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt). At the time of writing, the list is:

```text
52.22.183.94
3.220.109.109
```

Always use the published list as the source of truth, and check it again if TollBit tells you it has changed.

Our requests use the user agent `tollbot/1.0`, so you may see it show up in your logs.

Make sure these addresses are allowed through every security layer that could block or challenge our requests. This applies to every hostname you onboard with us:

- **Your CDN's WAF and firewall rules**: IP allow and block lists, managed rules, and rate limiting.
- **Bot protection**, if enabled: make sure requests from these IPs are not blocked, challenged, or served alternate content.
- **Any firewall or WAF in front of your origin servers**, outside of your CDN.

If you are unsure whether your configuration blocks us, contact [team@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

# Steps for Analytics

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
[team@tollbit.com](mailto:team@tollbit.com) to coordinate with our engineering team on the rest of the TollBit<br />Analytics setup. To finalize your setup, we will need access to the directory in your Storage bucket where your logs are stored, along with the pattern for how the logs are stored for instance `/service/logs/2024/12/04/log-file`.

# Steps for Agent Site

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
agentnativesearchbot, ai2bot, aionbot, aiwebindex, amazonbot, amzn-searchbot, amzn-user, anthropic-ai, atevesearchsourceurldiscovery, azureai-searchbot, bravebot, bytespider, ccbot, chatgpt-user, claude-code, claude-searchbot, claude-user, claude-web, claude/, claudebot, cloudflarebrowserrenderingcrawler, cohere-ai, comet/, cursor/, desearchbot, devin/, diffbot, duckassistbot, exabot, exasearchbot, exauser, fellou, gensparkbot, githubcopilotruntime, goose/, gptbot, grok-agent, grokbot, hermes-agent, iaskspider, ibou-search, iboubot, isonsearchbot, keenable, kimi-searchbot, kimibot, lightpanda, linkupbot, manus-user, meta-externalagent, meta-externalfetcher, meta-webindexer, mistralai-user, mojeekbot, mysearchbot, oai-adsbot, oai-searchbot, openclaw, perplexity-user, perplexity/, perplexitybot, rednote-websearch-bot, reflectionbot, shap-user, shapbot, sofyabot, talarionsearchbot, timpibot, youbot
```

You can add or remove from this as befits your bot strategy. We can also keep these lowercased, since in the rules we are comparing the lowercased values.
Click save to save these changes.

![Azure Front Door Bot Rules](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-bot-rules.png)

Once you've created this ruleset, you need to associate your Front Door route to it.
Click the 3 horizontal dots to the right of the rule set, and click "Associate a route".

![Azure Front Door Associate Route](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/azure-front-door-associate-route.png)

Choose the route to the relevant Front Door instance and go through the flow of associating this route.

### Premium

If you have a Premium tier Front Door instance, contact our team at [team@tollbit.com](mailto:team@tollbit.com) and we can<br />connect with you and evaluate the best path forward.
