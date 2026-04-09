---
title: Vercel
excerpt: Learn how to integrate TollBit with Vercel.
---
# Vercel

To forward logs from Vercel, follow the instructions for
<Anchor label="Log Drains here" target="_blank" href="https://vercel.com/docs/observability/log-drains/configure-log-drains">Log Drains here</Anchor>.

To properly authenticate and verify your logs, you should use the endpoint
`https://log.tollbit.com/log/vercel`.

In order to verify the endpoint through Vercel, pass the header
`x-vercel-tollbit-verify` as a custom header, along with your organization's
secret key as the custom header `TollbitKey`. See the following screenshots for
an example of the configuration. Once you have added these headers, you should
be able to click the `Verify` button and add your log drain.

![Log Drain 1](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/log-drain-1.png)

![Log Drain 2](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/log-drain-2.png)

### Route to Agent Site

To redirect bots to your `tollbit` subdomain you can use <Anchor label="Vercel's Custom WAF rules" target="_blank" href="https://vercel.com/docs/security/vercel-waf/custom-rules">Vercel's Custom WAF rules</Anchor>.

Create a new rule. Set the rule to look at the User Agent and selected `Matches expression`. Copy the following regex as the expression to match. Feel free to modify to remove or block different bots.

```
(ChatGPT-User|PerplexityBot|GPTBot|anthropic-ai|CCBot|Claude-Web|ClaudeBot|cohere-ai|YouBot|Diffbot|OAI-SearchBot|meta-externalagent|Timpibot|Amazonbot|Bytespider|Perplexity-User)
```

Set the rule to redirect to your `tollbit` subdomain by changing the `Then` option to `Redirect` and copy in your `tollbit` subdomain. The rule should look like this when you're finished

![Vercel Waf Rule](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/vercel-waf-rule.png)

Finally, click save rule. The change won't go into effect until you publish the change to go live.
