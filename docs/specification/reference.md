---
title: Reference
description: UCAP reference — authentication headers, security considerations, error codes, capability IDs, ID formats, and context field definitions.
---

# Reference

This page consolidates authentication details, security considerations, error codes, and format references for the UCAP specification.

---

## Authentication & Authorization

### Agent Authentication (OpenBotAuth)

All UCAP requests **MUST** include OpenBotAuth HTTP signatures per [RFC 9421](https://datatracker.ietf.org/doc/html/rfc9421){ target="_blank" }.

#### Required Headers

| Header | Description |
|--------|-------------|
| `Signature-Agent` | Agent's unique identifier |
| `Signature` | Base64-encoded Ed25519 signature |
| `Signature-Input` | Signature parameters including covered components |
| `Date` | Request timestamp (for replay protection) |

#### Signature Components

The signature **MUST** cover at minimum:

```text
("@method" "@path" "host" "date")
```

For requests with bodies:

```text
("@method" "@path" "host" "date" "content-type" "content-digest")
```

#### JWKS Resolution

UCAP servers resolve agent public keys via the OpenBotAuth JWKS registry:

```text
https://openbotauth.org/jwks/{agent_id}
```

### UCAP-Purpose Header

Agents **SHOULD** declare their intent using the `UCAP-Purpose` header:

| Purpose | Description |
|---------|-------------|
| `read` | Reading content for human consumption |
| `summarize` | Generating summaries (may require attribution) |
| `index` | Indexing for search (metadata only) |
| `archive` | Archival purposes |

```http
UCAP-Purpose: summarize
```

---

## Security Considerations

### Agent Verification

- All agent requests **MUST** include valid OpenBotAuth signatures
- Signatures **MUST** be verified against the agent's JWKS
- Replay attacks **MUST** be prevented via nonce/timestamp checking

### Rate Limiting

UCAP servers **SHOULD** implement rate limiting:

| Tier | Limit |
|------|-------|
| Unauthenticated | 10 requests/hour |
| Authenticated (free) | 100 requests/hour |
| Subscribed | 1000 requests/hour |

Rate-limited responses **MUST** use HTTP status `429 Too Many Requests` and **SHOULD** include a `Retry-After` header.

### Content Attribution

Publishers **MAY** require attribution for AI-processed content:

```json
{
  "attribution": {
    "required": true,
    "license": "CC-BY-4.0",
    "format": "Source: [Title](URL) by Author"
  }
}
```

---

## Capability Identifiers

| Capability ID | Description |
|---------------|-------------|
| `dev.ucap.content.access` | Fetch content with entitlement checking |
| `dev.ucap.content.catalog.search` | Free-text search with filters |
| `dev.ucap.content.catalog.lookup` | Direct ID lookup |
| `dev.ucap.content.subscription` | Subscription management |
| `dev.ucap.content.identity` | OAuth identity linking |

---

## Media Types

| Media Type | Description |
|------------|-------------|
| `application/json` | Default for all responses |
| `application/ucap+json` | UCAP-specific responses (optional) |

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `payment_required` | 402 | Subscription needed |
| `agent_unauthorized` | 403 | Invalid or missing signature |
| `content_not_found` | 404 | Post does not exist |
| `NOT_FOUND` | 200 | Catalog item not found (UCP convention) |
| `rate_limited` | 429 | Too many requests |
| `server_error` | 500 | Internal error |

---

## ID Format Reference

| Format | Pattern | Example | Description |
|--------|---------|---------|-------------|
| Publication ID | `pub_{id}` | `pub_ai-digest` | Internal publication identifier |
| Post ID | `post_{id}` | `post_abc123` | Internal post identifier |
| Slug | `slug:{slug}` | `slug:ai-digest` | URL-friendly identifier |
| Patreon | `patreon:{id}` | `patreon:12345678` | Patreon campaign or post ID |
| Ghost | `ghost:{id}` | `ghost:abc123` | Ghost post ID |
| URL | `url:{encoded}` | `url:https%3A%2F%2F...` | Original content URL (URL-encoded) |

---

## Context Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `language` | string | BCP 47 language tag | `"en"`, `"es"`, `"fr-CA"` |
| `currency` | string | ISO 4217 currency code | `"USD"`, `"EUR"`, `"GBP"` |
| `country` | string | ISO 3166-1 alpha-2 | `"US"`, `"GB"`, `"DE"` |
| `intent` | string | Semantic search description | `"weekly AI news digest"` |
