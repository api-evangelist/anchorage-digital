---
name: Register a webhook endpoint and subscribe to events
description: Register an HTTPS webhook endpoint with Anchorage Digital, subscribe to event types, and verify delivery signatures.
api: openapi/anchorage-digital-openapi-original.json
operations: [listWebhookEventTypes, createWebhookEndpoints, createWebhookEndpointSubscriptions, getWebhookValidationKey]
---

# Register an Anchorage Digital webhook and subscribe to events

Base URL: `https://api.anchorage.com/v2`. Requires `Api-Access-Key`.

## Steps
1. `listWebhookEventTypes` — discover the event types (and their `eventTypeId`s) you can subscribe to.
2. `createWebhookEndpoints` — register your HTTPS endpoint URL; capture the returned `endpointId`.
3. `createWebhookEndpointSubscriptions` — subscribe the `endpointId` to the event types you want.
4. `getWebhookValidationKey` — fetch the public validation key and verify the signature on every inbound payload before trusting it.

## Rules
- Serve a stable HTTPS endpoint that returns 2xx quickly; process asynchronously.
- Verify signatures with the validation key on receipt — do not act on unverified payloads.
- Manage subscriptions with `listWebhookEndpointSubscriptions` and `cancelWebhookEndpointSubscription`.
- Standard error envelope `{ errorType, message }`; back off on `429`/`5xx`.
