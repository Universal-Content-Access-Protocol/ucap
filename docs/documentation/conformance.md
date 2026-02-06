# Conformance

This section defines the requirements for conformant UCAP servers and agents.

## Server Conformance

### MUST (Required)

A conformant UCAP server **MUST**:

1. Implement the Content Access capability (`dev.ucap.content.access`)
2. Verify OpenBotAuth signatures on all requests
3. Return proper `402 Payment Required` responses with offers for paywalled content
4. Support the well-known discovery endpoint (`/.well-known/ucap.json`)

### SHOULD (Recommended)

A conformant UCAP server **SHOULD**:

1. Implement Catalog Search (`dev.ucap.content.catalog.search`)
2. Implement Catalog Lookup (`dev.ucap.content.catalog.lookup`)
3. Implement Subscription Management (`dev.ucap.content.subscription`)
4. Provide MCP tools for AI agent integration
5. Follow UCP error handling conventions (200 with messages, not 404) for catalog operations

### MAY (Optional)

A conformant UCAP server **MAY**:

1. Implement Identity Linking (`dev.ucap.content.identity`)
2. Support USDC cryptocurrency payments
3. Maintain a global publisher registry
4. Support content access by original URL

## Agent Conformance

### MUST (Required)

A conformant UCAP agent **MUST**:

1. Sign all requests with OpenBotAuth
2. Handle `402 Payment Required` responses gracefully
3. Present checkout URLs to humans (not auto-purchase) for Stripe payments
4. Respect `UCAP-Purpose` semantics

### SHOULD (Recommended)

A conformant UCAP agent **SHOULD**:

1. Include the `UCAP-Purpose` header on content access requests
2. Provide `context` with search requests for improved relevance
3. Check `messages` arrays for errors on 200 responses
4. Respect attribution requirements when displaying content

### MAY (Optional)

A conformant UCAP agent **MAY**:

1. Execute USDC payments autonomously when authorized
2. Use the `intent` field in context to improve search relevance
3. Cache catalog responses for improved performance
4. Support multiple transport bindings (REST, MCP)

## Conformance Levels

| Level | Requirements |
|-------|-------------|
| **Basic** | Content Access + OpenBotAuth verification |
| **Standard** | Basic + Catalog Search + Catalog Lookup |
| **Full** | Standard + Subscription + Identity Linking + MCP |
