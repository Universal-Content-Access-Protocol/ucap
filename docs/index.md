---
title: Universal Content Access Protocol (UCAP)
description: An open protocol enabling AI agents to discover, access, and subscribe to long-form content from publishers.
hide:
  - toc
tags:
  - UCAP
  - AI agents
  - content access
  - open protocol
---

<div class="ucap-hero" markdown>
<div class="ucap-hero__content" markdown>

# Universal Content Access Protocol

The open standard for AI agents to access publisher content. Built on [UCP](https://ucp.dev){ target="_blank" }.

UCAP extends the [Universal Commerce Protocol (UCP)](https://ucp.dev){ target="_blank" } with capabilities purpose-built for content access — discovery, catalog search, entitlement-based reading, and subscription management. Where UCP handles commerce, UCAP handles content.

</div>
<div class="ucap-hero__logo">
  <img src="assets/logo.svg" alt="UCAP logo" width="200">
</div>
</div>

<div class="ucap-promo" markdown>

<div class="ucap-promo__card" markdown>

### Learn

Protocol overview, core concepts, and design principles.

[Read the docs](documentation/core-concepts.md){ .md-button }

</div>

<div class="ucap-promo__card" markdown>

### Implement

Technical spec, HTTP API, MCP integration, and reference implementations.

[View the spec](specification/overview.md){ .md-button }

</div>

</div>

---

<div class="ucap-features" markdown>

<div class="ucap-feature" markdown>

### :material-api: HTTP-Native

Built on standard HTTP. Any client that speaks HTTP can access content — no proprietary SDKs required.

</div>

<div class="ucap-feature" markdown>

### :material-robot: Agent-First

Designed for automated access by AI agents, not human browsers. Structured data in, structured data out.

</div>

<div class="ucap-feature" markdown>

### :material-account-check: Human-in-Loop Payments

Agents facilitate content access. Humans authorize payments. Trust stays where it belongs.

</div>

<div class="ucap-feature" markdown>

### :material-swap-horizontal: Transport Agnostic

Works via REST, MCP, or other protocols. Choose the transport that fits your stack.

</div>

<div class="ucap-feature" markdown>

### :material-puzzle: Platform Agnostic

Supports any content platform — Patreon, Ghost, WordPress, Substack, and more.

</div>

<div class="ucap-feature" markdown>

### :material-shield-check: Secure by Default

RFC 9421 HTTP Message Signatures on every request. OAuth 2.0 for user delegation. Rate limiting built in.

</div>

</div>

---

<div class="ucap-action-carousel" markdown>

### See it in action

=== "Content Access"

    An agent requests paywalled content. UCAP returns the full article if entitled, or subscription offers if not.

    ```json
    {
      "content": {
        "html": "<article>Full post content...</article>",
        "text": "Plain text version...",
        "format": "html"
      },
      "metadata": {
        "id": "post-123",
        "title": "Premium Article Title",
        "author": "Creator Name",
        "published_at": "2026-02-01T10:00:00Z",
        "tier": "premium",
        "word_count": 2500
      },
      "attribution": {
        "required": true,
        "format": "Source: [My Newsletter](https://example.com/post-123)"
      }
    }
    ```

=== "Catalog Search"

    Agents discover content across publishers with structured search — filter by category, price, language, and freshness.

    ```json
    {
      "query": "AI newsletters",
      "filters": {
        "categories": ["technology", "artificial-intelligence"],
        "price": { "max": 1500, "currency": "USD" },
        "recency": "30d"
      },
      "context": {
        "intent": "weekly digest of AI news for research"
      }
    }
    ```

=== "Subscription"

    When content requires payment, UCAP returns a checkout URL for the human to complete — keeping agents out of the payment loop.

    ```json
    {
      "status": "pending_payment",
      "checkout": {
        "session_id": "session_abc123",
        "url": "https://example.com/checkout/session_abc123",
        "expires_at": "2026-02-04T13:00:00Z"
      },
      "message": "Complete payment to activate subscription"
    }
    ```

</div>

---

<div class="ucap-protocol-stack" markdown>

### Protocol Stack

UCAP works alongside UCP for payment handling and RFC 9421 HTTP Message Signatures for agent identity verification.

```text
┌─────────────────────────────────────────────────────────────┐
│                      Protocol Stack                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────┐    ┌──────────┐    ┌─────────────────┐      │
│   │   UCAP   │    │   UCP    │    │   RFC 9421      │      │
│   │(Content) │    │(Payments)│    │ (HTTP Signatures)│      │
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

</div>

---

<div class="ucap-get-started" markdown>

### Get Started

<div class="ucap-get-started__steps" markdown>

<div class="ucap-get-started__step" markdown>

#### :material-book-open-variant: Read

Understand the [core concepts](documentation/core-concepts.md) and protocol design.

</div>

<div class="ucap-get-started__step" markdown>

#### :material-code-braces: Implement

Follow the [specification](specification/overview.md) to build a conformant server or agent.

</div>

<div class="ucap-get-started__step" markdown>

#### :material-account-group: Contribute

Join the community and help shape the future of agentic content access.

</div>

</div>

</div>
