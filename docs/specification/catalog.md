# Catalog Capability

The Catalog capability enables agents to discover publishers and search content before accessing it. This fills the "discovery gap" — while Content Access assumes you already have a post ID, the Catalog enables scenarios like *"find me AI newsletters under $10/month"* that lead to subscription and access.

Following [UCP's catalog pattern](https://github.com/Universal-Commerce-Protocol/ucp/pull/55){ target="_blank" }, UCAP splits the catalog into two independent capabilities that implementers can adopt separately:

| Capability | ID | Description |
|---|---|---|
| [Catalog Search](catalog-search.md) | `dev.ucap.content.catalog.search` | Free-text search with filters and pagination |
| [Catalog Lookup](catalog-lookup.md) | `dev.ucap.content.catalog.lookup` | Direct lookup by publication or post ID |

## Motivation

Without a standardized catalog interface:

- Agents cannot discover what content is available
- Every integration requires custom discovery mechanisms
- There's no way to search across multiple publishers

## Data Structures

### Publication

A publication represents a publisher's content collection (analogous to UCP's `Product`).

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique publication identifier |
| `slug` | string | URL-friendly identifier |
| `title` | string | Publication name |
| `description` | string | Publisher description |
| `url` | string | Canonical URL |
| `category` | string | Primary category |
| `price` | PriceRange | Price range (min/max across tiers) |
| `media[]` | array | Images, logos (first = featured) |
| `tiers[]` | array | Subscription options (first = featured) |
| `rating` | object | Aggregate agent ratings (optional) |
| `post_count` | integer | Total posts available |
| `language` | string | Primary content language (BCP 47) |
| `metadata` | object | Publisher-defined extensions |

### Post

A post represents a single piece of content (analogous to UCP's `Variant`).

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Post identifier (used as `post_id` in content access) |
| `external_id` | string | Source platform ID (e.g., Patreon post ID) |
| `title` | string | Post title |
| `excerpt` | string | Preview text |
| `published_at` | string | Publication timestamp (RFC 3339) |
| `min_tier` | string | Minimum tier required for access |
| `category` | string/array | Post category or tags |
| `media[]` | array | Featured images |
| `word_count` | integer | Content length |
| `reading_time_minutes` | integer | Estimated read time |
| `metadata` | object | Post-specific extensions |

### Context Object

The context object allows agents to specify preferences that affect search and display.

```json
{
  "context": {
    "language": "en",
    "currency": "USD",
    "country": "US",
    "intent": "research on AI agent architectures"
  }
}
```

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `language` | string | Preferred content language (BCP 47) | `"en"`, `"es"`, `"fr-CA"` |
| `currency` | string | Preferred price display currency (ISO 4217) | `"USD"`, `"EUR"` |
| `country` | string | Agent's operating region (ISO 3166-1 alpha-2) | `"US"`, `"GB"` |
| `intent` | string | Semantic description of search goal | `"find weekly AI newsletters"` |

## Extensions

The catalog capability is designed for extension. Publishers and platforms can add domain-specific fields via the `metadata` object without protocol changes.

### Planned Extensions

| Extension | Capability ID | Description |
|-----------|---------------|-------------|
| Topics | `dev.ucap.content.catalog.topics` | Detailed topic/tag taxonomy |
| Freshness | `dev.ucap.content.catalog.freshness` | Content recency signals |
| Ratings | `dev.ucap.content.catalog.ratings` | Agent ratings and reviews |
| Recommendations | `dev.ucap.content.catalog.recommendations` | Personalized suggestions |
