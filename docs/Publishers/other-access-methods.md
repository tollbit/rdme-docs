---
title: Other Access Methods
excerpt: Learn how to enable NLWeb, MCP, and Agent2Agent protocols for your website.
---
# Other Access Methods

TollBit offers ways to monetize your content via various agentic protocols - including Microsoft's NLWeb, Google's Agent2Agent, and Anthropic's MCP protocols. Instructions for getting started with each protocol are in their respective sections.

## NLWeb

<Anchor label="NLWeb" target="_blank" href="https://news.microsoft.com/source/features/company-news/introducing-nlweb-bringing-conversational-interfaces-directly-to-the-web/">NLWeb</Anchor> is a protocol that aims to simplify the creation of natural language interfaces for websites. There are two main endpoints in the NLWeb protocol, an HTTP `/ask` endpoint and an MCP compatible `/mcp` endpoint. This allows developers and agents to seamlessly interact with your content via natural language.

Your NLWeb ask endpoint will be hosted at `https://tollbit.<yoursite>/ask` and your mcp endpoint will be hosted at `https://tollbit.<yoursite>/mcp`

Here's what an example request to an `/ask` endpoint might look like:

```
https://tollbit.time.com/ask?query=what%27s%20happening%20with%20tarrifs&mode=generate&streaming=false
```

To test your MCP endpoint, download the official <Anchor label="MCP inspector" target="_blank" href="https://modelcontextprotocol.io/docs/tools/inspector">MCP inspector</Anchor> from Anthropic and follow along with our demo here.

### Important query parameters:

**`query`**: This is required, and is the natural language query that you want to ask the website to get content back and your query answered based on that content.

**`streaming`**: This is optional. NLWeb streams results back via SSE by default. If you want to disable streaming, set the "streaming" query param equal to "false".

**`mode`**: The values here are "list", "summarize", and "generate".

* `list` will simply return a list of documents that match the query.
* `summarize` will return a list plus a summary of the contents of the docs.
* `generate` will return the list plus an answer to the query based on the documents.

## MCP

MCP is an open protocol created by Anthropic that enables AI models to securely connect to external data sources and tools. TollBit implements MCP servers that allow AI agents to discover and access publisher content through a standardized interface. This means Claude, ChatGPT, and other MCP-compatible agents can seamlessly query your content using natural language.

Your MCP endpoint will be hosted at `https://tollbit.yoursite/mcp`

Please reach out to [team@tollbit.com](mailto:team@tollbit.com) if you're interested in making your data available via MCP.

## Agent2Agent

Agent2Agent (A2A) is a protocol that enables AI agents to discover, negotiate access to, and consume content from publishers in a standardized way.

The protocol is an open standard designed to let AI agents collaborate securely without sharing their internal logic or data. It enables agents to discover each other's capabilities, negotiate how to interact, and work together on tasks over http(s). For more A2A details, please review the README.

Your A2A server will be hosted at `https://tollbit.capital.de/a2a`

To validate and test your A2A server, you can use the a2a-inspector. Follow the instructions for starting the inspector, and for the Agent Card URL use `https://tollbit.capital.de/a2a/.well-known/agent.json`. Clicking Connect should produce an Agent card valid message and allow you to communicate with the server using the A2A protocol using the chat feature of the inspector.
