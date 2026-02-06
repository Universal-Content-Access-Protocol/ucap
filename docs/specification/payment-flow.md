# Payment Flow

UCAP uses a human-in-the-loop payment model. Agents cannot execute payments autonomously — except when using cryptocurrency (USDC on Base) where agents can pay directly.

## Flow Overview

```text
┌─────────────────────────────────────────────────────────────────┐
│                    UCAP Payment Flow                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Agent requests content                                      │
│     GET /v1/content/publisher/post                              │
│                                                                 │
│  2. Server returns 402 with offers                              │
│     { offers: [{ checkout_url: "..." }] }                       │
│                                                                 │
│  3. Agent presents checkout URL to human                        │
│     "Click here to subscribe: [link]"                           │
│                                                                 │
│  4. Human completes payment in browser                          │
│     - Visits checkout URL                                       │
│     - Pays via Stripe/Google Pay/etc.                           │
│                                                                 │
│  5. Server mints entitlement                                    │
│     - Webhook from payment processor                            │
│     - Entitlement stored for user+agent                         │
│                                                                 │
│  6. Agent requests content again                                │
│     GET /v1/content/publisher/post                              │
│                                                                 │
│  7. Server returns 200 with content                             │
│     { content: "..." }                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Step-by-Step

### 1. Content Request

The agent requests content as normal via the [Content Access](content-access.md) capability.

### 2. Payment Required Response

The server returns `402 Payment Required` with one or more subscription offers. Each offer includes a `checkout_url`.

### 3. Human Handoff

The agent presents the checkout URL to the human user. The agent **MUST NOT** auto-purchase via Stripe or similar traditional payment methods.

!!! warning "Human-in-the-loop"
    The checkout URL is the critical handoff point. Agents present the URL; humans click and pay. This keeps payment authorization under human control.

### 4. Payment Completion

The human visits the checkout URL in a browser and completes payment through the publisher's payment processor (Stripe, Google Pay, etc.).

### 5. Entitlement Minting

After successful payment, the payment processor notifies the UCAP server via webhook. The server creates an entitlement record linking the user (and optionally the agent) to the publisher's content tier.

### 6–7. Entitled Access

The agent requests the same content again. This time the server finds a valid entitlement and returns the full content with a `200 OK`.

## Checkout URL

The `checkout_url` returned in 402 responses leads to a payment page. The URL:

- Contains a session ID for tracking
- Expires after a reasonable time (e.g., 1 hour)
- Redirects back to agent context after completion (if supported)

## UCP Integration

UCAP servers **MAY** use [UCP](https://github.com/Universal-Commerce-Protocol/ucp){ target="_blank" } for payment processing. When UCP is used, offers include a `ucp_plan_id`:

```json
{
  "offers": [
    {
      "tier_id": "premium",
      "checkout_url": "https://ucp.example.com/checkout/session_abc",
      "ucp_plan_id": "plan_premium_monthly"
    }
  ]
}
```

## USDC Payments

For cryptocurrency payments, agents **MAY** execute USDC payments autonomously when authorized. The subscription endpoint returns payment details instead of a checkout URL when `payment_method` is `"usdc"`.

## Guidelines

### For UCAP Servers

- Servers **MUST** include at least one offer with a `checkout_url` in 402 responses
- Servers **MUST** expire checkout sessions after a reasonable period
- Servers **SHOULD** support webhook-based entitlement minting
- Servers **MAY** support UCP for payment processing

### For Agents

- Agents **MUST** present checkout URLs to humans — not auto-purchase for Stripe payments
- Agents **MUST** handle the async nature of payment (poll or wait for entitlement)
- Agents **MAY** execute USDC payments autonomously when authorized
- Agents **SHOULD** inform the user about the subscription price before presenting the checkout URL
