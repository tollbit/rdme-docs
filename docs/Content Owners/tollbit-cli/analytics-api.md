---
title: Analytics Data
excerpt: Getting started and querying analytics data via the TollBit CLI
deprecated: false
hidden: true
metadata:
  robots: index
---
The Analytics API gives you access to the data behind your TollBit dashboard: which AI bots read which pages, where AI referrals land, and how traffic changes day to day. Ask your AI agent questions in plain language, or run read-only SQL yourself.


Access: request it at [team@tollbit.com](mailto:team@tollbit.com). Once enabled, you'll see it under CLI → Analytics API in your dashboard. For installation and sign-in, see Getting Started.

# Getting Started

This guide covers requesting access, installing the TollBit CLI and connecting it to your organization. If it's already done on your local environemnt, please proceed to the next section.&#x20;

### **Request Access**

Each API is enabled separately for each organization.

1. Email [team@tollbit.com](mailto:team@tollbit.com) and say which APIs you'd like enabled: Analytics API, Search API, or both.
2. Make sure you can sign in to the TollBit dashboard. You'll approve the CLI's access in your browser while signed in. The CLI runs on macOS, Linux and Windows.

The TollBit team will notify you once the access is enabled, and you can proceed to setting up the CLI with your preferred method below.

### **Choose Set up Method**

**Option A: With your AI agent (recommended)**

This is the fastest path, and it requires no code. Your agent installs the CLI, learns how to use it, and asks you to approve access.

1. In your TollBit dashboard, go to CLI → Quickstart and select your agent (Claude Code, Codex, Cursor or Other).
2. Select Copy setup prompt. For Claude Code, the prompt is:

<Callout icon="🖥️" theme="info">
  ### Copy the Prompt Below for Your Agent

  Install TollBit from [https://github.com/tollbit/cli](https://github.com/tollbit/cli) using the installation method appropriate for this environment, then run tollbit guide --install \~/.claude/skills to register the bundled skill. Then run tollbit auth login and tell me when to approve access in my browser.
</Callout>

3. Paste the prompt into your agent. It installs the CLI, adds the TollBit skill so it knows how to use the CLI, and starts sign-in.
4. Approve in your browser. TollBit opens a page where you link the agent to your organization. Make sure you're signed in to the correct TollBit organization before approving.
5. Start asking questions.

**Option B: In a terminal**

1. Install the CLI (available here: [https://github.com/tollbit/cli](https://github.com/tollbit/cli "https://github.com/tollbit/cli"))

```shell
npm install -g @tollbit/tollbit-cli
```
```powershell
irm "https://raw.githubusercontent.com/tollbit/cli/main/scripts/install.ps1" | iex
```

2. Sign in

```shell
tollbit auth login
```

<br />A browser window opens. Approve access for your TollBit organization. Note to log out, you can enter **tollbit auth logout**.&#x20;

3. Confirm you're connected

```shell
tollbit auth status
```

<br />This shows the email address and organization you're signed in as.

# Querying Data

The CLI provides 2 ways to query your data. 1) Plain English using your chat agent or 2) directly via the CLI using a tool like Terminal.

## Via Plain English

Once your agent is connected, ask questions the way you'd ask a colleague. The agent checks the available data, writes the SQL, runs it, and summarizes the answer. You can also try sample questions on the Analytics API page in your dashboard and select Copy prompt to reuse them.

<br />

| Goal                 | Example Prompt                                                                            |
| -------------------- | ----------------------------------------------------------------------------------------- |
| Top AI bots          | Which AI bots crawled my site most this month?                                            |
| Trends               | How has GPTBot traffic changed week over week since September?                            |
| Top Pages            | Which 20 pages did ChatGPT-User fetch most in the last 7 days?                            |
| Blocking and paywall | Which bots received the most 400-class responses last week?                               |
| Referrals            | Which AI platforms sent us referral traffic last month, and which pages did they land on? |
| Sections             | Which sections of the site are AI search bots most interested in?                         |
| Comparisons          | Compare AI training bots vs. AI search bots by request volume this quarter.               |
| Verification         | How much Googlebot traffic came from verified Google IP addresses?                        |

<Callout icon="far fa-circle-info" theme="info">
  ### Pro Tip

  The more specific the question (time range, site, bot, section), the more detail you're receive back. If an answer looks off, ask the agent to show the SQL it ran.
</Callout>

#### **Note on TollBit Intelligence:**&#x20;

If you have Tollbit Intelligence enabled, you can query about entity and topic information via Plain English as well.&#x20;

<Table>
  <thead>
    <tr>
      <th style={{ width: "23.9%" }}>
        Goal
      </th>

      <th style={{ width: "76%" }}>
        Example Prompt
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        Brand Insights
      </td>

      <td>
        Which brands mentioned in our content got the most AI bot impressions over the last 7 days?
      </td>
    </tr>

    <tr>
      <td>
        Content Categories
      </td>

      <td>
        What content categories do our articles from the past week fall into?
      </td>
    </tr>
  </tbody>
</Table>

## Via SQL&#x20;

If you would like to query your data directly through your CLI, you can choose to query your data directly via SQL.&#x20;

Below are some example queries based on the schema the CLI provides. Note, if you would like to see the full schema, run **tollbit analytics schema** or check out the next section.&#x20;

<Table>
  <thead>
    <tr>
      <th style={{ width: "26.5%" }}>
        Goal
      </th>

      <th style={{ width: "73.4%" }}>
        Example Prompt
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        Top AI bots in the last 7 days
      </td>

      <td>
        tollbit analytics query "SELECT user_agent, SUM(count) AS requests FROM user_agent_aggregate WHERE timestamp >= TIMESTAMP(DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)) AND type = 'REQUEST' GROUP BY user_agent ORDER BY requests DESC LIMIT 20"
      </td>
    </tr>

    <tr>
      <td>
        Pages a bot fetched most
      </td>

      <td>
        tollbit analytics query "SELECT path, SUM(count) AS requests FROM agent_logs_by_page WHERE host = 'example.com' AND user_agent = 'ChatGPT-User' AND timestamp >= TIMESTAMP(DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)) GROUP BY path ORDER BY requests DESC LIMIT 20"
      </td>
    </tr>

    <tr>
      <td>
        Status codes by bot (allowed vs. blocked)
      </td>

      <td>
        tollbit analytics query "SELECT user_agent, status_code, SUM(count) AS requests FROM user_agent_aggregate WHERE timestamp >= TIMESTAMP(DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)) AND type = 'REQUEST' GROUP BY user_agent, status_code ORDER BY requests DESC LIMIT 50"
      </td>
    </tr>

    <tr>
      <td>
        AI referrals by platform and landing page
      </td>

      <td>
        tollbit analytics query "SELECT referrer_domain, landing_path, SUM(count) AS visits FROM referrer_logs WHERE timestamp >= TIMESTAMP(DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)) AND referrer_domain IN ('chatgpt.com', 'perplexity.ai', 'claude.ai', 'gemini.google.com', 'copilot.microsoft.com') GROUP BY referrer_domain, landing_path ORDER BY visits DESC LIMIT 50"
      </td>
    </tr>
  </tbody>
</Table>

#### **Note on TollBit Intelligence:**&#x20;

If you have Tollbit Intelligence enabled, you can query about entity and topic information via SQL as well.&#x20;

<Table>
  <thead>
    <tr>
      <th style={{ width: "23.9%" }}>
        Goal
      </th>

      <th style={{ width: "76%" }}>
        Example Prompt
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        Brand Insights
      </td>

      <td>
        tollbit analytics query "SELECT entity_name, SUM(impressions) AS impressions FROM entity_brand_traffic_daily WHERE day >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY) AND day \< CURRENT_DATE() AND level = 'all' AND entity_name != '' GROUP BY entity_name ORDER BY impressions DESC LIMIT 20"
      </td>
    </tr>

    <tr>
      <td>
        Content Categories
      </td>

      <td>
        tollbit analytics query "SELECT hostname, category.name FROM entity_pages, UNNEST(categories) AS category WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY) AND timestamp \< CURRENT_TIMESTAMP() LIMIT 20"
      </td>
    </tr>
  </tbody>
</Table>

# Technical API Details

In order to make the most of the Analytics API, it's best to get familiar with the types of commands and the full table schema available within the CLI.

## Commands

There are two primary command types:

| Command                           | What It Does                                                            |
| --------------------------------- | ----------------------------------------------------------------------- |
| `tollbit analytics schema`        | Lists the tables and columns you can query, as JSON. Run this first.    |
| `tollbit analytics query "<SQL>"` | Runs a single read-only `SELECT` query and returns the results as JSON. |

<br />

**_Note_**_: Always wrap the SQL in double quotes, and use single quotes for values inside it. Without the outer quotes, your shell will try to interpret the query itself and fail._

**Output format.** Results come back as JSON with three parts:

- columns: column names and types
- rows: the results, in column order
- meta: row count, whether the results were truncated, bytes scanned and query duration

**Saving Results**

Redirect the output to a file:

```shell
tollbit analytics query "SELECT ..." > results.json
```

If you're using an AI agent, ask it to save the results as a CSV file for you.

## Table Schema

All traffic tables contain daily counts: each row is one day (00:00 UTC) for a given combination of site, bot, page and so on.

There are 5 tables available for query

| Table                     | Key columns                                                        | Best for                                                                                                     |
| ------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `user_agent_aggregate`    | timestamp, host, user_agent, status_code, type, ip_provider, count | Overall bot volume, trends, status codes and IP verification. The smallest table, good for long date ranges. |
| `agent_logs_by_page`      | timestamp, host, user_agent, path, status_code, count              | Starting from a bot: which pages did GPTBot fetch?                                                           |
| `page_logs_by_agent`      | timestamp, host, path, user_agent, status_code, count              | Starting from a page: which bots fetched this article?                                                       |
| `referrer_logs`           | timestamp, host, referrer_domain, landing_path, count              | Starting from a referrer: where did chatgpt.com send visitors?                                               |
| `page_logs_for_referrers` | timestamp, host, landing_path, referrer_domain, count              | Starting from a page: which referrers sent traffic to it?                                                    |

<br />

A few notes based on the columns shared above:

- **host:&#x20;**&#x79;our site's hostname, without [www](http://www).
- **user_agent:&#x20;**&#x61; normalized bot or browser name, such as GPTBot, ChatGPT-User or Chrome.
- **status_code:&#x20;**&#x74;he response class (200, 300, 400 or 500), not the exact code. For example, a 402 Payment Required appears as 400.
- **type (in user_agent_aggregate only):&#x20;**&#x52;EQUEST for pages, ROBOT for robots.txt, SITEMAP for sitemaps, WELL-KNOWN for /.well-known paths.
- **ip_provider:** the company that owns the requesting IP range, when verified. It's empty otherwise, which is useful for spotting bots that impersonate real crawlers.

<Callout icon="far fa-circle-info" theme="info">
  ### Note on TollBit Inteligence

  Organizations with TollBit Intelligence access also see content and brand tables (entity_data, entity_pages, entity_brand_traffic_daily). Please reach out to support\@tollbit.com if you would like to retreive additional details.
</Callout>

# Security & Access

1. You approve every connection. No agent can use the CLI on your organization's behalf until someone on your team approves it in the browser.
2. Scoped to your organization. Access is limited to the organization you approved at sign-in.
3. Revoking access. Run tollbit auth logout or tollbit auth logout --all. Running tollbit auth login again also revokes all previous tokens.
4. Credentials stay local. The access token is stored on the machine where the CLI is installed.
5. Verified releases. The CLI is open source under the MIT license and published on GitHub, with checksums and build attestations for each release.