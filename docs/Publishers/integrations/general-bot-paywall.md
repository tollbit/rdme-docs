---
title: General Bot Paywall
excerpt: Learn how to integrate TollBit Bot Paywall with other bot redirection methods.
deprecated: false
hidden: false
metadata:
  robots: index
---
** Setting Up the Bot Paywall with an Unlisted CDN or Cybersecurity Tool

If your CDN or cybersecurity tool is not listed on our Integrations page, you can still set up the TollBit bot paywall as long as your platform supports bot redirect capabilities — the ability to detect bot user agents and redirect them to a custom URL.

** How It Works

TollBit's bot paywall works by intercepting requests from AI crawlers and redirecting them to a TollBit-hosted URL, where access and monetization logic is handled. Any tool that can inspect the User-Agent header and conditionally redirect traffic can support this setup.

<br />

### Prerequisites

Before configuring your tool, make sure you have:

* Your TollBit subdomain (e.g., tollbit.yoursite.com or ai.yoursite.com) — found in your TollBit dashboard
* The list of bot user agents you want to monetize — available in your TollBit dashboard under Bot Management
* Access to your CDN or security tool's bot rules, firewall rules, or redirect configuration

<br />

### General Setup Steps

Please see the following 4 steps below to integrate bot redirect at your edge layer to pass traffic to TollBit for monetization.

If you're unsure whether your platform supports bot redirect capabilities, or need help translating these steps to your specific tool, contact [team@tollbit.com](mailto:team@tollbit.com) or book a technical office hours session with our team.

** Step 1 - Identify your redirect configuration **
Log into your CDN or cybersecurity platform and locate where you can define custom rules based on request headers. This is typically found under:

* Firewall Rules or WAF Rules
* Bot Management
* Edge Rules or Request Manipulation
* Traffic Policies

** Step 2 - Create a rule to match TollBit bot user agents **

Create a new rule that triggers when the incoming User-Agent header matches any of the bots in your TollBit bot list. For example:

**Recommend AI user agent list: **

chatgpt-user, perplexitybot, gptbot, anthropic-ai, ccbot, claude-web, claudebot, cohere-ai, youbot, diffbot, oai-searchbot, meta-externalagent, timpibot, amazonbot, bytespider, perplexity-user

Sample rule format:

User-Agent contains "ChatGPT-User" | User-Agent contains "GPTBot" | User-Agent contains "PerplexityBot" | User-Agent contains "anthropic-ai" | User-Agent contains "CCBot" | User-Agent contains "Claude-Web" | User-Agent contains "ClaudeBot" | User-Agent contains "cohere-ai" | User-Agent contains "YouBot" | User-Agent contains "Diffbot" | User-Agent contains "OAI-SearchBot" | User-Agent contains "meta-externalagent" | User-Agent contains "Timpibot" | User-Agent contains "Amazonbot" | User-Agent contains "Bytespider" | User-Agent contains "Perplexity-Use"

Preserve the original request path when redirecting, so TollBit can serve the correct content:

**https://your-tollbit-subdomain$request_uri**

** Step 3 - Set the redirect type **

Use a 302 (temporary) redirect so that changes to your bot list or TollBit configuration take effect without caching issues.

** Step 4 - Test your configuration **

Use a tool like curl to simulate a bot request and confirm the redirect is working:

**curl -A "GPTBot" -I [https://yoursite.com/sample-article](https://yoursite.com/sample-article)**

You should see a 302 response redirecting to your TollBit subdomain.

<br />

### Tips

* Order of rules matters. Make sure the TollBit redirect rule runs before any other rules that might block or alter bot traffic.
* Allowlist TollBit's infrastructure. If your platform has aggressive bot blocking, ensure TollBit's own crawler (used for content indexing) is whitelisted. Check your TollBot section under Marketplace for more details.
* Keep your bot list in sync. As new AI crawlers emerge, update the user agents in both your TollBit dashboard and your CDN rule to ensure consistent coverage.

<br />
