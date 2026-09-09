# ProductNow

**Answers you can act on. Actions you can trust.**

The context engine for people and AI agents.

This Cursor plugin connects the agent to ProductNow’s hosted MCP server so it can search shared organizational context, persist decisions as documents, and take governed actions as the signed-in user.

## Why ProductNow

Organizational context has become a big data problem of its own: volume, velocity, variety, and veracity. Decisions, chats, commits, tickets, and meetings pile up faster than anyone can reconcile them, in incompatible systems, and most of it goes stale the moment it is written.

ProductNow continuously captures, reconciles, and organizes that context from every system, human, and agent into one shared source of truth, represented as approachable documents anyone (or any agent) can read, write, and edit. From that shared context engine, answers are grounded in evidence, agents can reason across the company, and governed actions can write back into systems of record.

One shared source of truth from all your systems, people, and agents, so you can act with confidence.

## What this plugin includes

- **MCP server** — remote Streamable HTTP at `https://api.productnow-prod.com/mcp`
- **Skill** — when and how to search, persist, and coordinate through ProductNow tools
- **Rule** — treat ProductNow as the team’s context engine; chat is ephemeral

The ProductNow application itself is hosted. This repository packages Cursor plugin metadata and agent guidance only.

## Install

Install from the Cursor Marketplace, or test locally:

```bash
ln -s /path/to/this-repo ~/.cursor/plugins/local/productnow
```

Then reload the window and enable the plugin in **Customize**. On first connection, complete the browser OAuth sign-in. There is no API key to paste. Required scope: `mcp:use`.

## Authentication

| | |
| --- | --- |
| Endpoint | `https://api.productnow-prod.com/mcp` |
| Transport | Streamable HTTP |
| Auth | OAuth 2.0 (Auth0) via protected-resource discovery |
| App | [app.productnow.ai](https://app.productnow.ai) |
| Website | [productnow.ai](https://productnow.ai) |

Tool calls run as the authenticated ProductNow user and go through normal workspace, document, and comment permissions.

## After you connect

Ask the agent to search ProductNow first (`search_knowledge_warehouse`) for org questions, specs, and decisions. Persist anything that should outlive this chat with `create_document` or `post_document_chat_message`. Confirm write actions before they run.

## Support

`support@productnow.ai`
