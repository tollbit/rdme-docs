---
title: Security
excerpt: Questions about PII handling, authentication, and security measures.
---

# Security

## How do you handle PII info in the logs and is there a delay?

We take PII handling and log privacy seriously. We ingest raw server-side logs to detect bot activity, and securely store all of our data.

There is a 24-hour delay between when logs are received and when analytics appear in the dashboard. Customers either stream logs to our log sink endpoint continuously or let us pull from their S3 buckets once per day (nightly). A few hours after ingestion, we run a batch process that aggregates the previous day's logs and inserts the results into our BigQuery table. Once that's done, updated analytics appear in the dashboard. This means dashboards reflect the previous full day's data, not real-time traffic.

## How do publishers ensure TollBit doesn't trigger their security alarms, given it's acting like a proxy?

We work directly with publishers to ensure our proxy servers are clearly whitelisted and identified within their security and firewall settings. Publishers explicitly approve TollBit's servers or specific user agents. This clear identification prevents accidental triggering of security alarms or rate-limiting measures.

## What is a TollBit token? How does authentication work and is it secure against token reuse or interception?

Yes, our authentication system is designed with security in mind. Every single request a bot makes has a unique, single-use token generated specifically for that content page. Once the token is used, it's immediately invalidated. Even if someone intercepts the token, they won't be able to reuse it. Additionally, this token validation happens entirely on our TollBit infrastructure, which is securely managed.

## What happens after bots pass through TollBit's authentication – do they return directly to publishers' main websites?

Not directly. When bots pass our authentication, TollBit acts as a proxy as we fetch the content directly from the publisher's infrastructure (either from their main website or a dedicated API endpoint they provide to us). The bots themselves never directly access the publisher's main website after redirection. This approach ensures a controlled environment, reduces load on publishers, and centralizes access management.

