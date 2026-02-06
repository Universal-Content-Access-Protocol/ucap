# Core Concepts

The Universal Content Access Protocol (UCAP) is an open protocol that enables AI agents and automated systems to discover, access, and subscribe to long-form content from publishers. Built on HTTP with OpenBotAuth for agent authentication, UCAP provides a standardized way for agents to interact with paywalled content, handle subscription purchases, and access entitled content — all while keeping humans in control of payment decisions.

## Problem Statement

The rise of AI agents creates new challenges for content publishers:

- **No standardized access** — Agents lack a uniform way to access paywalled content
- **Authentication gaps** — Traditional cookie/session auth doesn't work for autonomous agents
- **Payment friction** — Agents cannot execute payments autonomously; humans must authorize
- **Discovery limitations** — Agents cannot easily discover what content is available

## Design Goals

UCAP is designed with the following principles:

| Principle | Description |
|-----------|-------------|
| **HTTP-Native** | Built on standard HTTP, accessible to any client |
| **Agent-First** | Designed for automated access, not human browsers |
| **Human-in-Loop Payments** | Agents facilitate, humans authorize payments |
| **Transport Agnostic** | Works via REST, MCP, or other protocols |
| **Platform Agnostic** | Supports any content platform (Patreon, Ghost, WordPress, etc.) |

## Roles and Participants

### Agent

An automated system — AI assistant, bot, or script — that accesses content on behalf of a user.

**Responsibilities:**

- Authenticate via OpenBotAuth HTTP signatures
- Discover and search for content via the Catalog capability
- Request content and handle entitlement responses
- Present checkout URLs to humans when payment is required

**Examples:** Claude, ChatGPT, custom AI assistants, automated research tools

### Publisher

A content creator or organization offering content via UCAP.

**Responsibilities:**

- Register with a UCAP server and configure content tiers
- Map content to access tiers (public, free, members, premium)
- Set pricing and subscription options
- Provide content in structured format

**Examples:** Newsletter authors, media organizations, independent bloggers

### Subscriber

A human user who authorizes an agent to access content on their behalf.

**Responsibilities:**

- Authorize agent access via OAuth identity linking
- Complete payment when subscription is required
- Manage entitlements and subscriptions

### UCAP Server

A service implementing the UCAP specification that mediates between agents and publishers.

**Responsibilities:**

- Verify agent authentication (OpenBotAuth signatures)
- Check entitlements and return content or offers
- Manage subscriptions and payment sessions
- Maintain the content catalog

## Key Goals

- **Interoperability** — Any agent can access any publisher's content through one protocol
- **Discovery** — Agents can search and browse content catalogs programmatically
- **Security** — Cryptographic agent identity verification on every request
- **Agentic Commerce** — Structured payment flows that keep humans in control

## Protocol Stack

UCAP is designed to work alongside [UCP (Universal Commerce Protocol)](https://github.com/Universal-Commerce-Protocol/ucp){ target="_blank" } for payment handling and [OpenBotAuth](https://github.com/OpenBotAuth/openbotauth){ target="_blank" } for agent identity verification.

```text
┌─────────────────────────────────────────────────────────────┐
│                      Protocol Stack                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────┐    ┌──────────┐    ┌─────────────────┐      │
│   │   UCAP   │    │   UCP    │    │   OpenBotAuth   │      │
│   │(Content) │    │(Payments)│    │ (Agent Identity) │      │
│   └──────────┘    └──────────┘    └─────────────────┘      │
│        │               │                  │                 │
│        └───────────────┴──────────────────┘                 │
│                         │                                   │
│                  ┌──────┴──────┐                            │
│                  │    HTTP     │                            │
│                  └─────────────┘                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Terminology

| Term | Definition |
|------|------------|
| **Agent** | An automated system (AI assistant, bot, script) accessing content |
| **Publisher** | A content creator or organization offering content via UCAP |
| **Subscriber** | A human user who authorizes an agent to access content on their behalf |
| **Entitlement** | A record granting an agent/user access to specific content tiers |
| **UCAP Server** | A service implementing the UCAP specification |
| **Content Store** | Storage system holding publisher content |
| **Capability** | A discrete piece of protocol functionality (e.g., content access, catalog search) |

## Content Access Model

UCAP defines four content access tiers, forming a progressive hierarchy:

```text
┌──────────────────────────────────────────────────────────────┐
│                    Content Access Tiers                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   PUBLIC          Anyone can access, no authentication       │
│      │                                                       │
│      ▼                                                       │
│   FREE            Requires agent authentication only         │
│      │                                                       │
│      ▼                                                       │
│   MEMBERS         Requires subscription (any paid tier)      │
│      │                                                       │
│      ▼                                                       │
│   PREMIUM         Requires specific tier subscription        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Response Model

All UCAP content requests result in one of three responses:

| Response | HTTP Status | Meaning |
|----------|-------------|---------|
| **ALLOW** | `200 OK` | Content returned, agent is entitled |
| **DENY** | `402 Payment Required` | Subscription needed, offers returned |
| **ERROR** | `4xx/5xx` | Authentication failure, not found, etc. |

## Core Concepts Summary

UCAP is organized around three concept categories:

- **Capabilities** — Discrete protocol functions that servers implement (content access, catalog search, catalog lookup, subscription management, identity linking)
- **Extensions** — Optional augmentations to capabilities via the `metadata` object (topics, freshness, ratings, recommendations)
- **Transports** — Protocol bindings for different integration patterns (REST/HTTP, MCP)
