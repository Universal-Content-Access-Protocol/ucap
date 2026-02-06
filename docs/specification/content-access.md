# Content Access Capability

* **Capability Name:** `dev.ucap.content.access`

The primary capability — fetching content with automatic entitlement verification. When an agent requests content, the server checks the agent's entitlements and returns either the full content (200) or subscription offers (402).

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/v1/content/{publisher_id}/{post_id}` | Fetch content by publisher and post ID |
| `GET` | `/v1/content?url={original_url}` | Fetch content by original URL |

## Operations

### Fetch Content

Retrieve a single piece of content. The server verifies the agent's identity and entitlements before returning the content.

=== "Request"

    ```http
    GET /v1/content/{publisher_id}/{post_id} HTTP/1.1
    Host: api.example.com
    Signature-Agent: {agent_id}
    Signature: sig=:{base64_signature}:
    Signature-Input: sig=("@method" "@path" "host" "date");keyid="{agent_id}"
    Date: Wed, 04 Feb 2026 12:00:00 GMT
    UCAP-Purpose: read
    Authorization: Bearer {user_token}
    ```

    The `Authorization` header is **OPTIONAL** — it is only required for identity-linked requests where an agent acts on behalf of a specific user.

=== "Response (200 — Entitled)"

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
        "updated_at": "2026-02-02T15:30:00Z",
        "tier": "premium",
        "word_count": 2500,
        "reading_time_minutes": 10
      },
      "publisher": {
        "id": "my-newsletter",
        "name": "My Newsletter",
        "url": "https://example.com/my-newsletter"
      },
      "attribution": {
        "required": true,
        "format": "Source: [My Newsletter](https://example.com/my-newsletter/post-123)"
      }
    }
    ```

=== "Response (402 — Payment Required)"

    ```json
    {
      "error": {
        "code": "payment_required",
        "message": "This content requires a subscription"
      },
      "teaser": {
        "title": "Premium Article Title",
        "excerpt": "First 200 characters of the article...",
        "author": "Creator Name",
        "published_at": "2026-02-01T10:00:00Z"
      },
      "offers": [
        {
          "tier_id": "premium",
          "tier_name": "Premium",
          "price": {
            "amount": 1500,
            "currency": "USD",
            "interval": "month"
          },
          "features": [
            "Full access to all premium posts",
            "Early access to new content"
          ],
          "checkout_url": "https://example.com/checkout/session_abc123"
        },
        {
          "tier_id": "supporter",
          "tier_name": "Supporter",
          "price": {
            "amount": 500,
            "currency": "USD",
            "interval": "month"
          },
          "features": [
            "Access to members-only posts"
          ],
          "checkout_url": "https://example.com/checkout/session_def456"
        }
      ],
      "publisher": {
        "id": "my-newsletter",
        "name": "My Newsletter"
      }
    }
    ```

## Response Fields

### Content Object

| Field | Type | Description |
|-------|------|-------------|
| `html` | string | Full content in HTML format |
| `text` | string | Plain text version of the content |
| `format` | string | Content format (`html`, `markdown`, `text`) |

### Metadata Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Post identifier |
| `title` | string | Post title |
| `author` | string | Author name |
| `published_at` | string | Publication timestamp (RFC 3339) |
| `updated_at` | string | Last update timestamp (RFC 3339) |
| `tier` | string | Content tier ID |
| `word_count` | integer | Content word count |
| `reading_time_minutes` | integer | Estimated reading time in minutes |

### Attribution Object

| Field | Type | Description |
|-------|------|-------------|
| `required` | boolean | Whether attribution is required |
| `license` | string | Content license (e.g., `CC-BY-4.0`) |
| `format` | string | Suggested attribution format string |

### Offer Object (402 Response)

| Field | Type | Description |
|-------|------|-------------|
| `tier_id` | string | Subscription tier identifier |
| `tier_name` | string | Human-readable tier name |
| `price` | object | Price with `amount` (cents), `currency`, `interval` |
| `features` | string[] | List of tier features |
| `checkout_url` | string | URL for the human to complete payment |

## Status Codes

| Status | Meaning |
|--------|---------|
| `200 OK` | Content returned — agent is entitled |
| `402 Payment Required` | Subscription needed — offers returned |
| `403 Forbidden` | Agent not authenticated or invalid signature |
| `404 Not Found` | Post does not exist |
| `429 Too Many Requests` | Rate limit exceeded |

## Guidelines

### For UCAP Servers

- Servers **MUST** verify OpenBotAuth signatures before returning content
- Servers **MUST** return 402 with at least one offer for paywalled content
- Servers **SHOULD** include a `teaser` with title and excerpt in 402 responses
- Servers **SHOULD** include `attribution` requirements in 200 responses
- Servers **MAY** support content access by original URL via the `?url=` query parameter

### For Agents

- Agents **MUST** sign all requests with OpenBotAuth
- Agents **MUST** handle 402 responses gracefully
- Agents **MUST** present checkout URLs to humans — not auto-purchase
- Agents **SHOULD** include the `UCAP-Purpose` header
- Agents **SHOULD** respect attribution requirements when displaying content
