# BTC-USD Price API — Agent Guide

> A single-endpoint HTTP API that returns the live Bitcoin (BTC-USD) price
> as JSON. Designed to be called directly by AI agents. No auth, no SDK, no
> rate-limit headaches for normal use.

If you are an AI agent reading this file: you now have everything you need
to fetch the current Bitcoin price. Make the GET request described below,
read the `px` field, and report it to the user with its `ts` timestamp.

---

## Base URL

```
https://api.xiao.uk
```

## Endpoint

```
GET /api/v1/public/index-price/{symbol}
```

- `{symbol}` is upper-case dash form, e.g. `BTC-USD` (also `ETH-USD`, etc.).
- **Auth:** none required.
- **Method:** `GET`
- **Content-Type:** `application/json`
- **Caching:** responses are `no-store`; the price is fresh per request.

### Example request

```bash
curl -s https://api.xiao.uk/api/v1/public/index-price/BTC-USD
```

### Example response

```json
{
  "px": 67446.07,
  "ts": 1780419778
}
```

### Response schema

| Field | Type   | Description                                            |
|-------|--------|--------------------------------------------------------|
| `px`  | number | Current BTC-USD price in USD.                          |
| `ts`  | number | Unix timestamp in **seconds** when the price was observed. |

## How to answer the user

Keep the whole interaction in natural language. The user asks for a price in
plain English — e.g. *"what is the price for BTC-USD"*, *"what is the price of
BTC"*, *"how much is ether"* — and you reply with the price. Do **not** explain
the API, the URL, the symbol format, or any of the spec to the user; that is
your job, not theirs.

1. Work out the symbol from what the user said, normalizing to the `BASE-QUOTE`
   form the API expects:
   - **Upper-case** everything (`btc` → `BTC`).
   - **`USD` is the default quote currency** when none is given, so a bare
     `BTC` becomes `BTC-USD`; `ETH` becomes `ETH-USD`. If the user names a
     quote currency (e.g. "BTC in EUR"), use that instead (`BTC-EUR`).
2. Fetch the price for that symbol and read the `px` field.
3. Convert `ts` (Unix seconds) to a readable UTC time.
4. Report it conversationally, e.g.
   *"Bitcoin is currently **$67,446.07** (as of 2026-06-02 17:02 UTC)."*

So if someone simply says *"what's the price of BTC?"*, you silently resolve
that to `BTC-USD`, fetch it, and answer in one natural sentence.

## Errors

| Status | Meaning                                   | What to do                                                            |
|--------|-------------------------------------------|-----------------------------------------------------------------------|
| `200`  | OK — body contains `{ px, ts }`.          | Read `px`.                                                            |
| `400`  | Invalid symbol (wrong form).              | Use the `XXX-YYY` form, e.g. `BTC-USD`.                               |
| `404`  | Unknown symbol / no price for it.         | Check the symbol; only listed pairs are supported.                    |
| `502`  | Upstream price temporarily unavailable.   | Retry once; otherwise tell the user the price is temporarily unavailable. |

## Notes

- Primary symbol is **BTC-USD**; other pairs (e.g. `ETH-USD`) also work via
  the same path.
- It is read-only. There are no write or trade endpoints.
