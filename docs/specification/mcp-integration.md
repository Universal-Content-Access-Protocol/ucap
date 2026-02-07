---
title: MCP Integration
description: UCAP MCP (Model Context Protocol) integration — tool definitions for search, lookup, read, subscribe, and entitlements for AI agents.
---

# MCP Integration

UCAP servers **MAY** expose functionality via MCP (Model Context Protocol) for AI agents. MCP provides a natural integration point for AI assistants like Claude.

## MCP Tools

UCAP defines the following MCP tools, each mapping to a protocol capability:

### search_catalog

Maps to: `dev.ucap.content.catalog.search`

```yaml
name: search_catalog
description: Search for publications and posts by query, category, price, and recency
inputSchema:
  type: object
  properties:
    query:
      type: string
      description: Free-text search query
    filters:
      type: object
      properties:
        categories:
          type: array
          items: { type: string }
          description: Filter by category/tag
        price:
          type: object
          properties:
            max:
              type: integer
              description: Maximum price in cents per month
        language:
          type: string
          description: Content language (BCP 47)
        recency:
          type: string
          enum: ["24h", "7d", "30d", "90d"]
          description: Content freshness filter
    context:
      type: object
      properties:
        language:
          type: string
          description: Preferred language
        currency:
          type: string
          description: Preferred currency (ISO 4217)
        intent:
          type: string
          description: Semantic description of search goal
    pagination:
      type: object
      properties:
        limit:
          type: integer
          description: Results per page (default 10, max 25)
        cursor:
          type: string
          description: Pagination cursor
```

### get_catalog_item

Maps to: `dev.ucap.content.catalog.lookup`

```yaml
name: get_catalog_item
description: Get publication or post details by ID (supports multiple ID formats)
inputSchema:
  type: object
  properties:
    id:
      type: string
      description: >
        ID in format: pub_{id}, post_{id}, slug:{slug},
        patreon:{id}, url:{encoded_url}
    context:
      type: object
      properties:
        language:
          type: string
        currency:
          type: string
  required: [id]
```

### read_content

Maps to: `dev.ucap.content.access`

```yaml
name: read_content
description: Read full content from a post (may return 402 if subscription required)
inputSchema:
  type: object
  properties:
    publisher_id:
      type: string
      description: Publication ID
    post_id:
      type: string
      description: Post ID
  required: [publisher_id, post_id]
```

### subscribe

Maps to: `dev.ucap.content.subscription`

```yaml
name: subscribe
description: >
  Subscribe to a publisher (returns checkout URL for human payment)
inputSchema:
  type: object
  properties:
    publisher_id:
      type: string
    tier_id:
      type: string
    payment_method:
      type: string
      enum: ["stripe"]
      description: Payment method preference
  required: [publisher_id, tier_id]
```

### list_entitlements

Maps to: `dev.ucap.content.subscription`

```yaml
name: list_entitlements
description: List active subscriptions for the authenticated agent/user
inputSchema:
  type: object
  properties: {}
```

## MCP as Wrapper

MCP tools **SHOULD** wrap the HTTP API, not replace it. This ensures that:

- The HTTP API remains the source of truth
- MCP tools inherit authentication and authorization from the HTTP layer
- Server implementations don't need to maintain two separate codebases

### Example Implementation

```python
async def ucap_read(publisher_id: str, post_id: str) -> dict:
    """MCP tool wraps HTTP API"""
    response = await http_client.get(
        f"https://api.ucap.example/v1/content/{publisher_id}/{post_id}",
        headers=sign_request_with_http_signatures()
    )

    if response.status == 402:
        return {
            "status": "payment_required",
            "offers": response.json()["offers"],
            "message": "Please click the checkout URL to subscribe"
        }

    return {
        "status": "ok",
        "content": response.json()["content"]
    }
```

## Guidelines

### For UCAP Servers

- Servers **MAY** expose capabilities as MCP tools
- MCP tools **SHOULD** be thin wrappers around the HTTP API
- Servers **MUST** still verify RFC 9421 HTTP signatures for MCP-originated requests
- Servers **SHOULD** document which MCP tools are available in their discovery endpoint

### For Agents

- Agents **SHOULD** prefer MCP tools when available for tighter integration
- Agents **MUST** handle `payment_required` responses from MCP tools the same as HTTP 402 responses
- Agents **SHOULD** use the `intent` field in the `context` parameter for better results
