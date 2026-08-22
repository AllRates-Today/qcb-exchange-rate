# Qatar Central Bank Exchange Rate API — qcb-exchange-rate

[![npm version](https://img.shields.io/npm/v/qcb-exchange-rate.svg)](https://www.npmjs.com/package/qcb-exchange-rate)
[![license](https://img.shields.io/npm/l/qcb-exchange-rate.svg)](https://github.com/AllRates-Today/qcb-exchange-rate/blob/main/LICENSE)
[![zero dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](https://www.npmjs.com/package/qcb-exchange-rate)
[![TypeScript](https://img.shields.io/badge/TypeScript-types%20included-3178C6.svg)](https://www.typescriptlang.org/)

**Official Qatar Central Bank (Qatar) daily exchange rates for Node.js and TypeScript. The published central bank rates behind tax filings, customs valuations, audits, and compliant invoicing — not market estimates, but the numbers Qatar Central Bank itself prints, every business day.**

## 🚀 Why this client?

- 🏛️ **Official published rates** — Qatar Central Bank's own table, with the publisher's own `rate_date` on every response
- 📅 **History back to 2024** — point-in-time tables and daily series for any past date
- 🔀 **Published vs derived, always flagged** — computed inverse/cross pairs carry `derived: true`, never mixed with official prints
- ⚡ **Zero dependencies** — pure ESM + CJS over global `fetch`; Node 18+, Bun, Deno, and edge runtimes
- 🔷 **Type-safe** — full TypeScript definitions shipped with the package
- 🧾 **Compliance-grade metadata** — `rate_type`, publication date, and source disclaimer on every response

> **Official rate, not mid-market:** every value here is a number Qatar Central Bank itself published, fixed once printed and carrying the central bank's own `rate_date` — what filings and audits require. Need the live interbank midpoint for pricing or display instead? Use the [mid-market API](https://allratestoday.com/docs/) or [`@allratestoday/sdk`](https://www.npmjs.com/package/@allratestoday/sdk). The two can diverge by several percent.

## 🔑 Get your API key

Get a free API key at [allratestoday.com/register](https://allratestoday.com/register) — no credit card required. Latest rates are on every plan, including free.

## 📦 Installation

```bash
npm install qcb-exchange-rate
```

```bash
yarn add qcb-exchange-rate
```

```bash
pnpm add qcb-exchange-rate
```

Also published under the org scope as [`@allratestoday/qcb-exchange-rate`](https://www.npmjs.com/package/@allratestoday/qcb-exchange-rate) — same code, same versions.

## 🏁 Quick start

```js
import { getRate } from 'qcb-exchange-rate';

const pair = await getRate('USD', 'QAR', { apiKey: 'art_live_...' });
console.log(pair.rate, pair.rate_date); // the official Qatar Central Bank rate, on the central bank's own date
```

## 📚 API reference

- [Latest pair rate](#latest-pair-rate) — one pair from the latest published table
- [Full published table](#full-published-table) — everything the central bank printed, in one call
- [Table for a date](#table-for-a-date) — the official table for an invoice or filing date
- [Daily time series](#daily-time-series) — one pair across a date range

---

### Latest pair rate

Free plan and up. Pairs the central bank does not print directly are resolved from its table and flagged (see *Published vs derived rates* below).

```js
const pair = await getRate('USD', 'QAR', { apiKey: 'art_live_...' });
```

**Response:**

```javascript
{
  bank: 'qcb',
  name: 'Qatar Central Bank',
  rate_date: '2026-08-21',   // Qatar Central Bank's own publication date
  source: 'USD',
  target: 'QAR',
  rate: 3.64,
  rate_type: 'reference',
  derived: false,
  method: 'published',
  disclaimer: '…'
}
```

### Full published table

Free plan and up. The complete table for the latest publication date.

```js
import { getLatestRates } from 'qcb-exchange-rate';

const table = await getLatestRates({ apiKey: 'art_live_...' });
console.log(table.rate_date, table.rates.length);
```

**Response:**

```javascript
{
  bank: 'qcb',
  name: 'Qatar Central Bank',
  rate_date: '2026-08-21',
  rates: [
    { "base": "USD", "quote": "QAR", "type": "reference", "value": 3.64 },
    // … the rest of the published table (7 currencies vs QAR)
  ],
  disclaimer: '…'
}
```

### Table for a date

Paid plans. The official table for any date since 2024 — weekends and holidays return the most recent published date, flagged via `published_on_requested_date`, which is exactly the in-force rate a filing needs.

```js
import { getRatesForDate } from 'qcb-exchange-rate';

const day = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...' });
// Optionally narrow to one pair:
const one = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...', source: 'USD', target: 'QAR' });
```

**Response:**

```javascript
{
  bank: 'qcb',
  requested_date: '2026-06-30',
  rate_date: '2026-06-30',                // the date actually published
  published_on_requested_date: true,      // false when a weekend/holiday fell back
  rates: [ /* the full table for that date */ ],
  disclaimer: '…'
}
```

### Daily time series

Paid plans. One resolved rate per publication date — ready for charting, revaluation runs, or audit workpapers.

```js
import { getHistory } from 'qcb-exchange-rate';

const series = await getHistory(
  { source: 'USD', target: 'QAR', from: '2026-01-01', to: '2026-08-21' },
  { apiKey: 'art_live_...' }
);
```

**Response:**

```javascript
{
  bank: 'qcb',
  source: 'USD',
  target: 'QAR',
  from: '2026-01-01',
  to: '2026-08-21',
  count: 152,
  rates: [
    // one entry per publication date
    { date: '2026-08-21', rate: 3.64, rate_type: 'reference', derived: false, method: 'published' },
    // …
  ],
  disclaimer: '…'
}
```

Pass `{ symbol: 'USD' }` instead of `source`/`target` to get the raw published rows for one currency (all rate types, no pair resolution).

---

## 🗺️ Currencies covered

Qatar Central Bank currently publishes rates covering **7 currencies** against the QAR (as of the latest table):

🇦🇺 `AUD` · 🇨🇦 `CAD` · 🇨🇭 `CHF` · 🇪🇺 `EUR` · 🇬🇧 `GBP` · 🇭🇰 `HKD` · 🇺🇸 `USD`

## ⚖️ Published vs derived rates

If Qatar Central Bank does not print a pair directly, the API resolves it from the central bank's own table and says so — official and computed values are never confused:

| `method` | `derived` | Meaning |
| --- | --- | --- |
| `published` | `false` | The central bank printed this pair directly |
| `inverse` | `true` | Computed as 1 ÷ the published opposite direction |
| `cross` | `true` | Computed via QAR from two published rates |

## 🛡️ Error handling

Errors are thrown as `Error` with `status` (HTTP code) and `body` (the API's JSON error) attached:

```js
try {
  const pair = await getRate('USD', 'XXX', { apiKey: 'art_live_...' });
} catch (err) {
  console.log(err.message); // human-readable reason
  console.log(err.status);  // e.g. 404
}
```

| Status | Meaning |
| ------ | ------- |
| — | Missing `apiKey` (thrown before any request) |
| `400` | Malformed date or parameters |
| `401` | Invalid API key |
| `403` | Endpoint needs a [paid plan](https://allratestoday.com/pricing/) (historical dates & series) |
| `404` | Pair or date range not covered by Qatar Central Bank |
| `429` | Monthly quota exceeded |

## 🔷 TypeScript

Full definitions ship with the package — no `@types` install:

```ts
import type { LatestRates, PairRate, DatedRates, RateEntry, HistoryQuery, RequestOptions } from 'qcb-exchange-rate';
```

## 📦 CommonJS

```javascript
const { getRate } = require('qcb-exchange-rate');

getRate('USD', 'QAR', { apiKey: 'art_live_...' }).then((pair) => console.log(pair.rate));
```

## 💡 Quota tips

- Rates change once per business day — cache the published table locally and a small monthly quota goes a long way.
- Every request counts toward your AllRatesToday quota, shared across all AllRatesToday endpoints on your key.

## 📖 Methods reference

| Method | Plan | Description |
| ------ | ---- | ----------- |
| `getRate(source, target, { apiKey })` | Free | Latest rate for one pair, resolved from the published table |
| `getLatestRates({ apiKey })` | Free | The central bank's full latest published table |
| `getRatesForDate(date, { apiKey, source?, target? })` | Paid | The official table (or one pair) for a YYYY-MM-DD date |
| `getHistory({ symbol \| source+target, from?, to? }, { apiKey })` | Paid | Daily series since 2024 |

## 🔗 Links

- [Qatar Central Bank rates page](https://allratestoday.com/central-bank-rates-api/qcb/) — live table, publication cadence, FAQ
- [All central bank sources](https://allratestoday.com/central-bank-rates-api/)
- [API documentation](https://allratestoday.com/docs/#central-bank) · [Interactive reference](https://allratestoday.com/api-reference/)
- [Register (free)](https://allratestoday.com/register) · [Pricing](https://allratestoday.com/pricing/)
- [GitHub](https://github.com/AllRates-Today/qcb-exchange-rate)

## 📜 License

MIT
