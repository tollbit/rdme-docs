---
title: How Monetization Works
deprecated: false
hidden: false
metadata:
  robots: index
---
We want to provide a high level walkthrough of how a developer accesses your content, and how TollBit ensures that this access is secure and tracked to ensure you are fairly compensated.

### Agent Registration

When a developer first signs up, they are required to register their user agent to the TollBit Platform. TollBit maintains this global registry of all user agents on the platform, and verifies their identity and connects them to the developer accounts.

### Requesting Access

Developers are able to see who is on the TollBit marketplace and which of them are ready to transaction. However, do not initially have any visibility into their rates or licenses. They must request access to publishers through the developer portal, and TollBit will vet each developer before granting them access.

### Tokens

Tokens are one-time access grants tied to specific pages and user agents. The token system allows authenticated access for retrieving content, with a one-time use model to ensure security.

Tollbit tokens are [JWTs](https://datatracker.ietf.org/doc/html/rfc7519) used to authorize a specific content request. They include authentication details, request intent, and, when applicable, pricing and license terms. Tollbit generates and validates tokens automatically; this section explains the token claims for reference. Tokens expire 5 minutes after issuance.

### Content Intent

TollBit allows developers to specify their intent when retrieving content, and will be able to serve different types of data with different monetization flows, for each of these intents. These can range from indexing content for building an internal knowledge graph, fetching static content in Markdown for RAG to respond to a user query, or getting a simplified version of the site that preserves key structure to enable faster agentic browsing.

<br />
