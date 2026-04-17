---
name: moonpay-mpp
description: Make paid API requests to MPP-protected endpoints using Tempo stablecoins. Use when the user wants to call an API that requires MPP (Machine Payments Protocol) payment — detects HTTP 402 responses and pays automatically with Tempo stablecoins.
tags: [payments, api, mpp, http402, tempo]
---

# MPP — Machine Payments Protocol

## Overview

MPP (Machine Payments Protocol) is the open standard for machine-to-machine payments via HTTP 402, co-developed by Tempo and Stripe. Backwards-compatible with x402, but additionally supports Stripe cards, Tempo stablecoins, and Lightning.

The MoonPay CLI automatically detects 402 Payment Required responses, pays via Tempo stablecoins using your local wallet, and retries the request with the payment credential.

## Prerequisites

- MoonPay CLI installed: `npm i -g @moonpay/cli`
- Authenticated: `mp login`
- Local wallet with Tempo stablecoin balance (see `moonpay-auth`, `moonpay-check-wallet`)

## Command

```bash
mp mpp request \
  --method GET \
  --url <mpp-endpoint-url> \
  --wallet <wallet-name-or-address>
```

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `--method` | Yes | HTTP method: GET, POST, PUT, PATCH, DELETE |
| `--url` | Yes | HTTPS URL of the MPP-protected endpoint |
| `--body` | No | JSON request body (for POST, PUT, PATCH) |
| `--params` | No | Query parameters as JSON object |
| `--wallet` | Yes | Wallet name or address to pay with |

## Workflow

1. Verify wallet has funds: `mp token balance list --wallet <name>`
2. Make the MPP request — payment is handled automatically on 402 response:
   ```bash
   mp mpp request --method GET --url <endpoint> --wallet <wallet>
   ```
3. The CLI: detects 402 → pays via Tempo → retries with payment credential → returns response

## Examples

```bash
# GET request to an MPP-protected data endpoint
mp mpp request \
  --method GET \
  --url https://api.example.com/data \
  --wallet my-wallet

# POST with JSON body
mp mpp request \
  --method POST \
  --url https://api.example.com/query \
  --body '{"query": "token_price", "symbol": "BTC"}' \
  --wallet my-wallet
```

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `Insufficient balance` | Wallet has no Tempo | Fund via `moonpay-buy-crypto` or `moonpay-swap-tokens` |
| `Wallet not found` | Wrong wallet name | Run `mp wallet list` to see available wallets |
| `Non-MPP endpoint` | Server doesn't return 402 | Request proceeds without payment — check endpoint docs |
| `Payment rejected` | Endpoint rejected Tempo payment | Contact endpoint provider for supported payment rails |

## Related Skills

- **moonpay-auth** — Create or import a local wallet
- **moonpay-check-wallet** — Check wallet balance before making paid requests
- **moonpay-x402** — For x402-protected endpoints (crypto payments on Solana/Base)
- **moonpay-upgrade** — Upgrade your CLI rate limit via x402 or MPP
