---
title: Catalog Lookup
description: UCAP Catalog Lookup specification — retrieve publications or posts by ID with support for multiple ID formats and error handling.
---

# Catalog Lookup

* **Capability Name:** `dev.ucap.content.catalog.lookup`

Direct lookup of a publication or post by ID. Supports multiple ID types for flexibility across platforms.

## ID Types

| ID Format | Description | Example |
|-----------|-------------|---------|
| `pub_{id}` | Publication by internal ID | `pub_ai-digest` |
| `post_{id}` | Post by internal ID | `post_abc123` |
| `slug:{slug}` | Publication by slug | `slug:ai-digest` |
| `patreon:{id}` | Patreon post/campaign ID | `patreon:12345678` |
| `ghost:{id}` | Ghost post ID | `ghost:abc123` |
| `url:{encoded_url}` | By original URL (URL-encoded) | `url:https%3A%2F%2F...` |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/v1/catalog/item/{id}` | Look up by ID with query params |
| `POST` | `/v1/catalog/lookup` | Look up by ID with full context |

## REST Binding (GET)

For simple lookups with optional query parameters.

=== "Request"

    ```http
    GET /v1/catalog/item/pub_ai-digest?language=en&currency=USD HTTP/1.1
    Host: api.example.com
    ```

    Supported query parameters: `language`, `currency`, `country`

=== "Response (Publication)"

    ```json
    {
      "type": "publication",
      "publication": {
        "id": "pub_ai-digest",
        "slug": "ai-digest",
        "title": "AI Digest Weekly",
        "description": "Curated AI news and analysis every Monday. Deep dives into the latest AI research, industry news, and practical applications.",
        "url": "https://example.com/ai-digest",
        "category": "technology",
        "price": {
          "min": 0,
          "max": 1000,
          "currency": "USD",
          "interval": "month"
        },
        "media": [
          {
            "type": "image",
            "url": "https://example.com/media/ai-digest-logo.png",
            "alt": "AI Digest logo",
            "width": 400,
            "height": 400
          }
        ],
        "tiers": [
          {
            "id": "tier_free",
            "name": "Free",
            "description": "Weekly AI news roundup",
            "price": { "amount": 0 },
            "features": ["Weekly newsletter", "Public archive access"],
            "post_count": 52
          },
          {
            "id": "tier_pro",
            "name": "Pro",
            "description": "Full access to all content",
            "price": {
              "amount": 1000,
              "currency": "USD",
              "interval": "month"
            },
            "features": [
              "Everything in Free",
              "Deep dive analyses",
              "Early access",
              "Discord community"
            ],
            "post_count": 156
          }
        ],
        "recent_posts": [
          {
            "id": "post_xyz789",
            "title": "GPT-5: What We Know So Far",
            "excerpt": "An analysis of the leaked benchmarks and what they mean...",
            "published_at": "2026-02-05T10:00:00Z",
            "min_tier": "tier_pro",
            "reading_time_minutes": 12
          },
          {
            "id": "post_xyz788",
            "title": "Weekly AI News Roundup #208",
            "excerpt": "This week: Claude 4, Gemini updates, and more...",
            "published_at": "2026-02-03T10:00:00Z",
            "min_tier": "tier_free",
            "reading_time_minutes": 5
          }
        ],
        "post_count": 208,
        "language": "en",
        "created_at": "2024-01-15T00:00:00Z",
        "metadata": {
          "source_platform": "patreon",
          "source_id": "campaign_12345",
          "subscriber_count": 5000,
          "publishing_frequency": "weekly"
        }
      },
      "messages": []
    }
    ```

## REST Binding (POST)

For lookups with full context support.

=== "Request"

    ```http
    POST /v1/catalog/lookup HTTP/1.1
    Host: api.example.com
    Content-Type: application/json

    {
      "id": "pub_ai-digest",
      "context": {
        "language": "en",
        "currency": "USD",
        "intent": "checking subscription options"
      }
    }
    ```

=== "Response (Post)"

    When looking up a post ID, returns the parent publication summary alongside the requested post:

    ```json
    {
      "type": "post",
      "publication": {
        "id": "pub_ai-digest",
        "slug": "ai-digest",
        "title": "AI Digest Weekly"
      },
      "post": {
        "id": "post_xyz789",
        "external_id": "patreon:post_98765",
        "title": "GPT-5: What We Know So Far",
        "excerpt": "An analysis of the leaked benchmarks and what they mean for the industry...",
        "published_at": "2026-02-05T10:00:00Z",
        "updated_at": "2026-02-05T14:30:00Z",
        "min_tier": {
          "id": "tier_pro",
          "name": "Pro",
          "price": {
            "amount": 1000,
            "currency": "USD",
            "interval": "month"
          }
        },
        "category": ["analysis", "gpt", "openai"],
        "media": [
          {
            "type": "image",
            "url": "https://example.com/media/gpt5-analysis-hero.png",
            "alt": "GPT-5 analysis header image"
          }
        ],
        "word_count": 2500,
        "reading_time_minutes": 12,
        "content_url": "/v1/content/ai-digest/post_xyz789",
        "metadata": {
          "source_url": "https://patreon.com/posts/98765"
        }
      },
      "messages": []
    }
    ```

## MCP Binding

```json
{
  "name": "get_catalog_item",
  "description": "Get publication or post details by ID (supports multiple ID formats)",
  "inputSchema": {
    "type": "object",
    "properties": {
      "id": {
        "type": "string",
        "description": "ID in format: pub_{id}, post_{id}, slug:{slug}, patreon:{id}, url:{encoded_url}"
      },
      "context": {
        "type": "object",
        "properties": {
          "language": { "type": "string" },
          "currency": { "type": "string" }
        }
      }
    },
    "required": ["id"]
  }
}
```

## Error Handling

Following UCP convention, `NOT_FOUND` returns HTTP 200 with an error message (not 404):

```json
{
  "type": "error",
  "publication": null,
  "post": null,
  "messages": [
    {
      "type": "error",
      "code": "NOT_FOUND",
      "message": "Publication not found: pub_invalid-id",
      "severity": "recoverable"
    }
  ]
}
```

## Guidelines

### For UCAP Servers

- Servers **MUST** support lookup by internal IDs (`pub_*`, `post_*`)
- Servers **SHOULD** support lookup by slug (`slug:*`)
- Servers **MAY** support lookup by external platform IDs (`patreon:*`, `ghost:*`)
- Servers **SHOULD** return `recent_posts` for publication lookups
- Servers **MUST** follow UCP error handling convention (200 with messages, not 404)

### For Agents

- Agents **SHOULD** prefer internal IDs when available
- Agents **SHOULD** provide `context` for localized responses
- Agents **MUST** check the `messages` array for errors even on 200 responses
