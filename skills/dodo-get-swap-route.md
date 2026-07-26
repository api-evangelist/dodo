---
name: Get a DODO swap route quote
description: Fetch a real-time token swap quote and executable ABI calldata from the DODO Trade / Route API, then execute it on-chain.
api: openapi/dodo-trade-openapi.yml
operations: [getDodoRoute]
---

# Get a DODO swap route quote

Use the DODO Trade / Route API to price a swap between two tokens on an EVM chain and
obtain the ABI calldata needed to execute it.

## Prerequisites
- An API key from the DODO Developer Portal (https://docs.dodoex.io/en/developer),
  passed as the `apikey` query parameter.
- The user's wallet address and an RPC node URL for the target chain.
- Token contract addresses and decimals for both sides. For the native asset
  (ETH/BNB/MATIC) use the sentinel address `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.

## Steps
1. Call `getDodoRoute` (`GET https://route-api.dodoex.io/dodoapi/getdodoroute`) with the
   required query params: `fromTokenAddress`, `fromTokenDecimals`, `toTokenAddress`,
   `toTokenDecimals`, `fromAmount` (in base units, e.g. 1 ETH = 10**18), `slippage`
   (0-100), `userAddr`, `chainId`, and `rpc`. Optionally set `deadLine` and `source`
   (set `source=dodo` to quote DODO v1/v2 liquidity only).
2. Check the response envelope: `status` must be `200`. A non-200 `status` means the
   request failed (unsupported chain, no route/illiquid pair, or invalid params) —
   see `errors/dodo-error-codes.yml`.
3. From `data`, read `resAmount` (expected receive amount) and `priceImpact`
   (multiply by 100 for a percentage) to confirm the quote is acceptable.
4. If selling an ERC-20 (not the native token), ensure the wallet has granted an
   allowance to `data.targetApproveAddr` (the DODOApprove contract) for the sell token.
5. Submit the transaction to `data.to` (DODORouteProxy / DODOV2Proxy) with `data.data`
   as the calldata, respecting `slippage` and `deadLine`.

## Conventions & error handling
- The API is read-only (GET) and has no idempotency key; on-chain execution
  idempotency is governed by the blockchain transaction. See `conventions/dodo-conventions.yml`.
- Supported chains (`chainId`): 1 Ethereum, 56 BSC, 66 OEC, 128 HECO, 137 Polygon,
  42161 Arbitrum One, 1285 MoonRiver, 1313161554 Aurora, 288 Boba, 43114 Avalanche.
