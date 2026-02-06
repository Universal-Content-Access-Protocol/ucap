# Identity Linking Capability

* **Capability Name:** `dev.ucap.content.identity`

Link agent identity to user identity via OAuth 2.0. This enables agents to act on behalf of a specific user, accessing their entitlements and managing their subscriptions.

## Overview

Identity linking bridges the gap between agent authentication (OpenBotAuth) and user authorization (OAuth 2.0). Once linked, an agent can make requests that reference a user's entitlements without the user being present.

```text
┌─────────────────────────────────────────────────────────────┐
│                  Identity Linking Flow                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. Agent discovers OAuth server metadata                  │
│      GET /.well-known/oauth-authorization-server            │
│                                                             │
│   2. Agent redirects user to authorization endpoint         │
│      GET /oauth/authorize?client_id=...&scope=read          │
│                                                             │
│   3. User authorizes the agent                              │
│      (Human approves in browser)                            │
│                                                             │
│   4. Agent exchanges code for token                         │
│      POST /oauth/token                                      │
│                                                             │
│   5. Agent uses token in subsequent requests                │
│      Authorization: Bearer {user_token}                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/.well-known/oauth-authorization-server` | OAuth server metadata discovery |
| `GET` | `/oauth/authorize` | Authorization endpoint |
| `POST` | `/oauth/token` | Token endpoint |

## OAuth Server Discovery

=== "Request"

    ```http
    GET /.well-known/oauth-authorization-server HTTP/1.1
    Host: api.example.com
    ```

=== "Response"

    ```json
    {
      "issuer": "https://api.example.com",
      "authorization_endpoint": "https://api.example.com/oauth/authorize",
      "token_endpoint": "https://api.example.com/oauth/token",
      "scopes_supported": ["read", "subscribe", "manage"],
      "response_types_supported": ["code"],
      "grant_types_supported": ["authorization_code", "refresh_token"]
    }
    ```

## Scopes

| Scope | Description |
|-------|-------------|
| `read` | Read content on behalf of the user |
| `subscribe` | Initiate subscriptions on behalf of the user |
| `manage` | Manage entitlements (cancel, modify) |

## Guidelines

### For UCAP Servers

- Servers **MUST** implement the OAuth 2.0 Authorization Code flow per [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749){ target="_blank" }
- Servers **MUST** support the `/.well-known/oauth-authorization-server` discovery endpoint
- Servers **SHOULD** support refresh tokens for long-lived agent access
- Servers **MUST** validate both the OpenBotAuth signature and the OAuth bearer token on identity-linked requests

### For Agents

- Agents **MUST** discover OAuth endpoints via the well-known URL before initiating the flow
- Agents **MUST** request only the scopes they need
- Agents **SHOULD** store and refresh tokens securely
- Agents **MUST** handle token expiration gracefully
