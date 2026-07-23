---
name: Transfer assets from a wallet
description: Move digital assets out of an Anchorage Digital wallet safely, using idempotent retries and Ed25519 request signing.
api: openapi/anchorage-digital-openapi-original.json
operations: [getVaults, getWallets, createTransfer, getTransfer]
---

# Transfer assets from an Anchorage Digital wallet

Base URL: `https://api.anchorage.com/v2` (sandbox: `https://api.anchorage-staging.com/v2`).

## Auth
- Every request sends `Api-Access-Key: <key>`.
- `createTransfer` is a sensitive endpoint: also sign with Ed25519 and send `Api-Signature` + `Api-Timestamp`.
- The key's permission group must grant transfer permission on the source vault, and (for external transfers) the trusted destination must be connected and endorsed.

## Steps
1. `getVaults` — find the vault the source wallet lives in (permission: Read vault activity).
2. `getWallets` — locate the source `walletId` and confirm it holds the asset and sufficient balance (amount + network fee).
3. `createTransfer` — POST the transfer. Include a unique `idempotentId` in the body so a retried request never double-sends. Sign the request (Ed25519). For same-asset fee handling, `deductFeeFromAmountIfSameType: true` nets the fee out of the amount.
4. `getTransfer` — poll by `transferId` for terminal status. A `201` is not final: the transfer can still fail after quorum approval or at broadcast (insufficient fee / chain error).

## Rules
- Retries: on `429`/`5xx`, back off (1s, 2s, 4s, 8s) and **reuse the same `idempotentId`**. Do not retry other `4xx`.
- `409` on a withdrawal means one is already in flight from that wallet — wait, then retry.
- Rate limit: 20 req/s per organization (burst 100).
- Errors return `{ errorType, message }` — see errors/anchorage-digital-problem-types.yml.
