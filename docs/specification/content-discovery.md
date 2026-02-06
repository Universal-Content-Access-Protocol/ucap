# Content Discovery

Content discovery enables agents to find UCAP-enabled publishers and content sources. UCAP provides two discovery mechanisms: a well-known endpoint for individual publishers and a global registry for cross-publisher discovery.

## Well-Known Endpoint

Publishers **MAY** expose a discovery document at a well-known URL on their domain. This allows agents to determine whether a publisher supports UCAP and find the UCAP server endpoint.

=== "Request"

    ```http
    GET /.well-known/ucap.json HTTP/1.1
    Host: newsletter.example.com
    ```

=== "Response"

    ```json
    {
      "ucap_version": "1.0",
      "publisher_id": "my-newsletter",
      "publisher_name": "My Newsletter",
      "ucap_server": "https://api.example.com",
      "catalog_url": "https://api.example.com/v1/publishers/my-newsletter/catalog",
      "subscribe_url": "https://example.com/subscribe/my-newsletter"
    }
    ```

### Discovery Document Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ucap_version` | string | Yes | UCAP specification version |
| `publisher_id` | string | Yes | Publisher's unique identifier |
| `publisher_name` | string | Yes | Publisher's display name |
| `ucap_server` | string | Yes | Base URL of the UCAP server |
| `catalog_url` | string | No | Direct URL to publisher's catalog |
| `subscribe_url` | string | No | URL for subscription page |

## Global Registry

UCAP servers **MAY** maintain a global registry of publishers for agent discovery. The registry allows agents to search for publishers without knowing their individual domains.

!!! note
    The global registry specification is under development. Current implementations should rely on the well-known endpoint for publisher discovery and the [Catalog Search](catalog-search.md) capability for content discovery.

## Publisher Registration

Publishers register with a UCAP server to make their content available to agents.

### Registration Requirements

| Requirement | Description |
|-------------|-------------|
| Content Source | Connection to content platform (API, export, etc.) |
| Tier Configuration | Mapping of content tiers to prices |
| Payment Setup | Payment processor connection (Stripe, etc.) |

### Supported Content Sources

UCAP is source-agnostic. Common integrations include:

| Source | Integration Method |
|--------|-------------------|
| Patreon | OAuth 2.0 + API |
| Ghost | Admin API Key |
| WordPress | REST API + Application Password |
| Substack | Manual export (ZIP upload) |
| Custom | Webhook or direct upload |

## Guidelines

### For Publishers

- Publishers **SHOULD** expose the `/.well-known/ucap.json` discovery document
- Publishers **MUST** configure at least one content tier
- Publishers **SHOULD** provide complete metadata (description, category, media) for catalog listing
- Publishers **MAY** support multiple content sources

### For Agents

- Agents **SHOULD** check `/.well-known/ucap.json` when encountering a publisher URL
- Agents **SHOULD** use the [Catalog Search](catalog-search.md) capability for broad content discovery
- Agents **SHOULD** cache discovery documents for reasonable periods
