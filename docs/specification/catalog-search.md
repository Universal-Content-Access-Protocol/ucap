# Catalog Search

* **Capability Name:** `dev.ucap.content.catalog.search`

Free-text search across publishers and posts with filters, context, and pagination.

## REST Binding

### Endpoint

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/catalog/search` | Search publications and posts |

### Example

=== "Request"

    ```http
    POST /v1/catalog/search HTTP/1.1
    Host: api.example.com
    Content-Type: application/json
    Signature-Input: sig=("@method" "@path" "host" "content-digest");keyid="agent_123"
    Signature: sig=:BASE64_SIGNATURE:

    {
      "query": "AI newsletters",
      "filters": {
        "categories": ["technology", "artificial-intelligence"],
        "price": {
          "min": 0,
          "max": 1500,
          "currency": "USD"
        },
        "language": "en",
        "recency": "30d",
        "tier_access": "free"
      },
      "context": {
        "language": "en",
        "currency": "USD",
        "intent": "weekly digest of AI news for research"
      },
      "pagination": {
        "limit": 10,
        "cursor": null
      }
    }
    ```

=== "Response"

    ```json
    {
      "results": [
        {
          "type": "publication",
          "publication": {
            "id": "pub_ai-digest",
            "slug": "ai-digest",
            "title": "AI Digest Weekly",
            "description": "Curated AI news and analysis every Monday",
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
                "alt": "AI Digest logo"
              }
            ],
            "tiers": [
              {
                "id": "tier_free",
                "name": "Free",
                "price": { "amount": 0 },
                "post_count": 52
              },
              {
                "id": "tier_pro",
                "name": "Pro",
                "price": {
                  "amount": 1000,
                  "currency": "USD",
                  "interval": "month"
                },
                "post_count": 156
              }
            ],
            "post_count": 208,
            "language": "en",
            "metadata": {
              "source_platform": "patreon",
              "subscriber_count": 5000
            }
          },
          "relevance_score": 0.95
        }
      ],
      "pagination": {
        "total": 45,
        "limit": 10,
        "cursor": "eyJvZmZzZXQiOjEwfQ=="
      },
      "messages": []
    }
    ```

## Request Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | No | Free-text search query |
| `filters` | object | No | Search filters (see below) |
| `filters.categories` | string[] | No | Category/tag filter |
| `filters.price` | object | No | Price range filter |
| `filters.price.min` | integer | No | Minimum price in cents |
| `filters.price.max` | integer | No | Maximum price in cents |
| `filters.price.currency` | string | No | Currency code (default: `USD`) |
| `filters.language` | string | No | Content language (BCP 47) |
| `filters.recency` | string | No | Content freshness: `"24h"`, `"7d"`, `"30d"`, `"90d"` |
| `filters.tier_access` | string | No | `"free"`, `"paid"`, `"any"` |
| `context` | object | No | Request context (see [Catalog Overview](catalog.md#context-object)) |
| `pagination.limit` | integer | No | Results per page (default: 10, max: 25) |
| `pagination.cursor` | string | No | Cursor for next page |

## Response Schema

| Field | Type | Description |
|-------|------|-------------|
| `results[]` | array | Array of search results |
| `results[].type` | string | Result type: `"publication"` or `"post"` |
| `results[].publication` | object | Publication data (see [Catalog Overview](catalog.md#publication)) |
| `results[].relevance_score` | number | Relevance score (0.0–1.0) |
| `pagination.total` | integer | Total number of results |
| `pagination.limit` | integer | Results per page |
| `pagination.cursor` | string | Cursor for next page (null if last page) |
| `messages[]` | array | Informational messages |

## MCP Binding

```json
{
  "name": "search_catalog",
  "description": "Search for publications and posts",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": { "type": "string" },
      "filters": { "$ref": "#/definitions/SearchFilters" },
      "context": { "$ref": "#/definitions/Context" },
      "pagination": { "$ref": "#/definitions/Pagination" }
    }
  }
}
```

See [MCP Integration](mcp-integration.md) for the full MCP tool definition.

## Guidelines

### For UCAP Servers

- Servers **SHOULD** support free-text search across publication titles, descriptions, and post content
- Servers **SHOULD** support all filter fields defined in the request schema
- Servers **MUST** return results sorted by relevance when a query is provided
- Servers **MAY** use the `context.intent` field to improve search relevance
- Servers **MUST** respect `pagination.limit` (max 25 per page)

### For Agents

- Agents **SHOULD** provide the `context` object for better search results
- Agents **SHOULD** use the `intent` field to describe the semantic goal of the search
- Agents **MAY** cache search results for improved performance
