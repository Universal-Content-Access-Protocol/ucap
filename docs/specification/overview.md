# Specification Overview

**Version:** 1.1.0-draft
**Status:** Draft Specification
**Last Updated:** 2026-02-06

---

## Conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this specification are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119){ target="_blank" } and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174){ target="_blank" }.

## Protocol Capabilities

UCAP defines five core capabilities, following the composable pattern from [UCP](https://github.com/Universal-Commerce-Protocol/ucp){ target="_blank" }:

| Capability | ID | Description |
|---|---|---|
| [Content Access](content-access.md) | `dev.ucap.content.access` | Fetch published content with automatic entitlement checking |
| [Catalog Search](catalog-search.md) | `dev.ucap.content.catalog.search` | Free-text search across publishers and posts |
| [Catalog Lookup](catalog-lookup.md) | `dev.ucap.content.catalog.lookup` | Direct lookup by publication or post ID |
| [Subscription](subscription.md) | `dev.ucap.content.subscription` | Subscribe to publishers and manage entitlements |
| [Identity Linking](identity-linking.md) | `dev.ucap.content.identity` | Link agent identity to user identity via OAuth |

### Capability Design

Each capability is independently implementable. A conformant UCAP server **MUST** implement Content Access and **SHOULD** implement Catalog Search and Catalog Lookup. Other capabilities are **OPTIONAL**.

Capabilities follow a reverse-domain naming convention (`dev.ucap.*`) consistent with UCP's namespace governance model.

### Capability Composition

```text
┌──────────────────────────────────────────────────────────┐
│                   UCAP Capabilities                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────────────────────────────────────────────┐    │
│   │            Content Access (REQUIRED)            │    │
│   │          dev.ucap.content.access                │    │
│   └─────────────────────────────────────────────────┘    │
│                                                          │
│   ┌──────────────────────┐ ┌────────────────────────┐    │
│   │   Catalog Search     │ │   Catalog Lookup       │    │
│   │  (RECOMMENDED)       │ │  (RECOMMENDED)         │    │
│   └──────────────────────┘ └────────────────────────┘    │
│                                                          │
│   ┌──────────────────────┐ ┌────────────────────────┐    │
│   │   Subscription       │ │   Identity Linking     │    │
│   │  (OPTIONAL)          │ │  (OPTIONAL)            │    │
│   └──────────────────────┘ └────────────────────────┘    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Transport Bindings

UCAP capabilities can be accessed through multiple transport bindings:

| Transport | Description | Use Case |
|-----------|-------------|----------|
| **REST / HTTP** | Standard HTTP API with JSON request/response | Direct integration, server-to-server |
| **MCP** | Model Context Protocol tools | AI agent integration (Claude, etc.) |

### REST / HTTP

The primary transport. All capabilities define HTTP endpoints under the `/v1/` path prefix. Requests and responses use `application/json`.

### MCP

UCAP servers **MAY** expose capabilities as MCP tools. MCP tools **SHOULD** wrap the HTTP API, not replace it. See [MCP Integration](mcp-integration.md) for details.

## Versioning

The UCAP specification follows [Semantic Versioning 2.0](https://semver.org/){ target="_blank" }:

- **Major** version for breaking changes to capability interfaces
- **Minor** version for new capabilities or backward-compatible additions
- **Patch** version for clarifications and editorial fixes

All HTTP endpoints are prefixed with `/v1/` for the current major version.

## References

- [UCP — Universal Commerce Protocol](https://github.com/Universal-Commerce-Protocol/ucp){ target="_blank" }
- [OpenBotAuth](https://github.com/OpenBotAuth/openbotauth){ target="_blank" }
- [RFC 9421 — HTTP Message Signatures](https://datatracker.ietf.org/doc/html/rfc9421){ target="_blank" }
- [RFC 6749 — OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749){ target="_blank" }
- [MCP — Model Context Protocol](https://modelcontextprotocol.io){ target="_blank" }
- [BCP 47 — Language Tags](https://www.rfc-editor.org/info/bcp47){ target="_blank" }
- [ISO 4217 — Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){ target="_blank" }
