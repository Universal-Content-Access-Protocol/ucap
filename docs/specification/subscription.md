---
title: Subscription Capability
description: UCAP Subscription capability specification — a UCP Checkout extension for recurring content subscriptions with entitlement management.
---

# Subscription Capability

* **Capability Name:** `dev.ucap.content.subscription`
* **Extends:** `dev.ucp.shopping.checkout`

Handle subscription creation and entitlement management. This capability enables agents to initiate recurring subscriptions on behalf of users and query active entitlements.

## Relationship to UCP

The Subscription capability is implemented as a [UCP extension](https://ucp.dev/latest/specification/overview/){ target="_blank" } that extends the Checkout capability (`dev.ucp.shopping.checkout`). It follows UCP's extension model — using the `extends` field to declare its parent and augmenting the Checkout schema with subscription-specific fields.

### Extension Declaration

A UCAP server advertising subscription support declares it in its profile:

```json
{
  "ucap": {
    "capabilities": {
      "dev.ucap.content.access": [
        { "version": "2026-02-06" }
      ],
      "dev.ucap.content.subscription": [
        {
          "version": "2026-02-06",
          "spec": "https://ucap.dev/specification/subscription",
          "extends": "dev.ucp.shopping.checkout"
        }
      ]
    }
  }
}
```

### Schema Extension

The subscription capability extends UCP's Checkout schema by adding a `subscription` object to the checkout entity:

```json
{
  "$id": "https://ucap.dev/schemas/content/subscription.json",
  "name": "dev.ucap.content.subscription",
  "title": "Content Subscription Extension",
  "description": "Extends UCP Checkout with recurring content subscription support.",
  "$defs": {
    "subscription_plan": {
      "type": "object",
      "required": ["interval", "tier_id"],
      "properties": {
        "interval": {
          "type": "string",
          "enum": ["month", "year"],
          "description": "Billing interval."
        },
        "tier_id": {
          "type": "string",
          "description": "Publisher content tier."
        },
        "trial_period_days": {
          "type": "integer",
          "minimum": 0,
          "description": "Trial days before first billing."
        }
      }
    },
    "dev.ucp.shopping.checkout": {
      "title": "Checkout with Content Subscription",
      "description": "UCP Checkout extended with recurring content subscription.",
      "allOf": [
        { "$ref": "https://ucp.dev/schemas/shopping/checkout.json" },
        {
          "type": "object",
          "properties": {
            "subscription": {
              "$ref": "#/$defs/subscription_plan",
              "description": "Subscription plan configuration.",
              "ucp_request": {
                "create": "optional",
                "update": "optional",
                "complete": "omit"
              }
            },
            "publisher_id": {
              "type": "string",
              "description": "Publisher to subscribe to."
            }
          }
        }
      ]
    }
  }
}
```

This means any UCP-compliant Checkout flow can be extended with UCAP subscription fields — payment processing, buyer consent, and fulfillment all work as defined by UCP.

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/subscribe` | Create a new subscription |
| `GET` | `/v1/entitlements` | List active entitlements |
| `DELETE` | `/v1/entitlements/{id}` | Cancel an entitlement |

## Operations

### Create Subscription

Initiate a subscription to a publisher's tier. Under the hood, this creates a UCP Checkout session extended with subscription fields. Returns a checkout URL for the human to complete payment.

=== "Request"

    ```http
    POST /v1/subscribe HTTP/1.1
    Host: api.example.com
    Content-Type: application/json
    Signature-Agent: {agent_id}
    Signature: sig=:{base64_signature}:
    Authorization: Bearer {user_token}

    {
      "publisher_id": "ai-digest",
      "tier_id": "pro",
      "payment_method": "stripe"
    }
    ```

=== "Response"

    ```json
    {
      "status": "pending_payment",
      "checkout": {
        "session_id": "session_abc123",
        "url": "https://example.com/checkout/session_abc123",
        "expires_at": "2026-02-04T13:00:00Z"
      },
      "subscription": {
        "interval": "month",
        "tier_id": "pro",
        "tier_name": "Pro"
      },
      "message": "Complete payment to activate subscription"
    }
    ```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `publisher_id` | string | Yes | Publisher to subscribe to |
| `tier_id` | string | Yes | Subscription tier |
| `payment_method` | string | No | Payment method preference (e.g., `"stripe"`) |

### List Entitlements

Retrieve all active subscriptions for the authenticated agent/user.

=== "Request"

    ```http
    GET /v1/entitlements HTTP/1.1
    Host: api.example.com
    Authorization: Bearer {user_token}
    ```

=== "Response"

    ```json
    {
      "entitlements": [
        {
          "id": "ent-123",
          "publisher_id": "ai-digest",
          "publisher_name": "AI Digest Weekly",
          "tier_id": "pro",
          "tier_name": "Pro",
          "status": "active",
          "created_at": "2026-01-15T10:00:00Z",
          "renews_at": "2026-02-15T10:00:00Z",
          "price": {
            "amount": 800,
            "currency": "USD",
            "interval": "month"
          }
        }
      ]
    }
    ```

### Entitlement Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entitlement identifier |
| `publisher_id` | string | Publisher identifier |
| `publisher_name` | string | Publisher display name |
| `tier_id` | string | Subscription tier ID |
| `tier_name` | string | Subscription tier display name |
| `status` | string | Status: `"active"`, `"expired"`, `"cancelled"` |
| `created_at` | string | Subscription start timestamp (RFC 3339) |
| `renews_at` | string | Next renewal timestamp (RFC 3339) |
| `price` | object | Subscription price with `amount`, `currency`, `interval` |

## Guidelines

### For UCAP Servers

- Servers **MUST** return a checkout URL in the subscription response
- Servers **MUST NOT** process payments directly — the checkout URL leads to a payment page for the human
- Servers **SHOULD** set a reasonable expiration on checkout sessions (e.g., 1 hour)
- Servers **MUST** mint entitlements after successful payment via webhook
- Servers **SHOULD** support entitlement listing for identity-linked agents
- Servers **SHOULD** declare `dev.ucap.content.subscription` with `extends: "dev.ucp.shopping.checkout"` in their capability profile

### For Agents

- Agents **MUST** present the checkout URL to the human — not auto-purchase
- Agents **SHOULD** check entitlements before requesting content to avoid unnecessary 402 responses
