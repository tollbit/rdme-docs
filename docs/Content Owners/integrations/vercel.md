---
title: Vercel
excerpt: Learn how to integrate TollBit with Vercel.
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

To forward logs from Vercel, follow the instructions for <Anchor target="_blank" href="https://vercel.com/docs/observability/log-drains/configure-log-drains">Log Drains here</Anchor>.

To properly authenticate and verify your logs, you should use the endpoint `https://log.tollbit.com/log/vercel`.

In order to verify the endpoint through Vercel, pass the header `x-vercel-tollbit-verify` as a custom header, along with your organization's secret key as the custom header `TollbitKey`. See the following screenshots for an example of the configuration. Once you have added these headers, you should<br />be able to click the `Verify` button and add your log drain.

![Log Drain 1](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/log-drain-1.png)

![Log Drain 2](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/log-drain-2.png)

# Steps for Agent Site

To redirect bots to your `tollbit` subdomain you can use <Anchor target="_blank" href="https://vercel.com/docs/security/vercel-waf/custom-rules">Vercel's Custom WAF rules</Anchor>.

Create a new rule. Set the rule to look at the User Agent and selected `Matches expression`. Copy the following regex as the expression to match. Feel free to modify to remove or block different bots.

```
(AgentNativeSearchBot|AI2Bot|AionBot|AIWebIndex|Amazonbot|Amzn-SearchBot|Amzn-User|anthropic-ai|AteveSearchSourceUrlDiscovery|AzureAI-SearchBot|Bravebot|Bytespider|CCBot|ChatGPT-User|claude-code|Claude-SearchBot|Claude-User|Claude-Web|Claude/|ClaudeBot|CloudflareBrowserRenderingCrawler|cohere-ai|Comet/|Cursor/|DesearchBot|Devin/|Diffbot|DuckAssistBot|Exabot|ExaSearchBot|ExaUser|Fellou|Gensparkbot|GitHubCopilotRuntime|goose/|GPTBot|grok-agent|GrokBot|Hermes-Agent|iaskspider|ibou-search|IbouBot|IsonSearchBot|Keenable|Kimi-SearchBot|KimiBot|Lightpanda|LinkupBot|Manus-User|meta-externalagent|meta-externalfetcher|meta-webindexer|MistralAI-User|MojeekBot|MySearchBot|OAI-AdsBot|OAI-SearchBot|OpenClaw|Perplexity-User|Perplexity/|PerplexityBot|rednote-websearch-bot|Reflectionbot|Shap-User|ShapBot|SofyaBot|TalarionSearchBot|Timpibot|YouBot)
```

Set the rule to redirect to your `tollbit` subdomain by changing the `Then` option to `Redirect` and copy in your `tollbit` subdomain. The rule should look like this when you're finished

![Vercel Waf Rule](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/vercel-waf-rule.png)

Finally, click save rule. The change won't go into effect until you publish the change to go live.
