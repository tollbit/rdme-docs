---
title: TollBit Agent Site
deprecated: false
hidden: false
metadata:
  robots: index
---
Your Agent Site, which is automatically set up after you onboard to TollBit, is how you control how the agentic internet sees your content. We want to provide a high level walkthrough of how agents access your content and the different layers of controls you can provide.

## Whitelisting Bots

By default, your agent site blocks all bot access. However, there are many circumstances in which you wish to allow certain bots to be able to access your content. The TollBit Agent Site allows you to whitelist specific bots to be able to retrieve your content in Markdown.

## Monetization

Monetization is an additional layer of access controls you can provision over your Agent Site. This enforces that all agents that wish you access your content go through the follow workflow.

### Agent Registration

When a developer first signs up, they are required to register their user agent to the TollBit Platform. TollBit maintains this global registry of all user agents on the platform, and verifies their identity and connects them to the developer accounts.

### Requesting Access

Developers are able to see who is on the TollBit marketplace and which of them are ready to transaction. However, do not initially have any visibility into their rates or licenses. They must request access to publishers through the developer portal, and TollBit will vet each developer before granting them access.

### Token Generation

In order to access content, developers must generate TollBit Tokens to include with each of their requests to your agent site.

Tokens are one-time access grants tied to specific pages and user agents. The token system allows authenticated access for retrieving content, with a one-time use model to ensure security.

Tollbit tokens are [JWTs](https://datatracker.ietf.org/doc/html/rfc7519) used to authorize a specific content request. They include authentication details, request intent, and, when applicable, pricing and license terms. Tollbit generates and validates tokens automatically; this section explains the token claims for reference. Tokens expire 5 minutes after issuance.

### Content Intent

TollBit allows developers to specify their intent when retrieving content, and will be able to serve different types of data with different monetization flows, for each of these intents. These can range from indexing content for building an internal knowledge graph, fetching static content in Markdown for RAG to respond to a user query, or getting a simplified version of the site that preserves key structure to enable faster agentic browsing.

### Transactions

Each request is ledgered as a transaction within our system, and all transactions include data around who, what and when it was accessed, the value of the transaction, the license it was retrieved under, and additional metadata that developers pass back, including information about the queries that led to retrieving that content. You are paid out at the end of each month for your total usage across all developers during the previous month.
