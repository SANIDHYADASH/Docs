# Smart Books REST API — v1

A single Supabase edge function (`supabase/functions/api`) exposes every business
resource of Smart Books over plain REST/JSON, so that:

- mobile apps (Android / iOS / React Native / Flutter) can be built on the same data,
- integrations (Tally, marketplaces, WhatsApp bots, ERPs) can push/pull records,
- performance / load testing can be scripted with k6, JMeter, Postman or `curl`,
- external systems can receive **signed webhooks** in real time.

Every request runs with the caller's own access token, so **Row-Level Security is
enforced end-to-end** — a token can only ever see the companies the user belongs to.

Live, always-current references generated from the deployed code:

| Link | Description |
|------|-------------|
| `GET /v1/openapi.json` | OpenAPI 3.1 contract (machine-readable, no auth) |
| `GET /v1/docs` | Rendered API reference page (no auth) |
| `GET /v1/health` | Liveness probe (no auth) |

---

## 1. Base URL

```
https://<project-ref>.supabase.co/functions/v1/api/v1
```

For this project:

```
https://qmbekbrvefqekhkapgrp.supabase.co/functions/v1/api/v1
```

Both `/api/v1/...` and `/v1/...` suffixes are accepted by the router.

## 2. Authentication

1. Sign in through Supabase Auth to get an access token:

```bash
curl -X POST "https://qmbekbrvefqekhkapgrp.supabase.co/auth/v1/token?grant_type=password" \
  -H "apikey: <SUPABASE_PUBLISHABLE_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"email":"owner@example.com","password":"••••••••"}'
```

2. Send the token on every API call:

```
Authorization: Bearer <access_token>
apikey: <SUPABASE_PUBLISHABLE_KEY>
X-Company-Id: <company uuid>        # optional, defaults to the user's first company
Content-Type: application/json
```

Tokens expire in one hour; refresh with `grant_type=refresh_token`.

### Errors

```json
{ "error": { "message": "Invalid or expired access token.", "status": 401 } }
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request / database constraint or validation error |
| 401 | Missing or invalid bearer token |
| 403 | Token valid but no company membership / RLS denied |
| 404 | Unknown resource, unknown report, or row not found |
| 405 | Method not allowed (e.g. writing to a read-only resource) |
| 500 | Unexpected server error |

### Response envelope

```json
{ "data": [ ... ], "meta": { "count": 128, "limit": 50, "offset": 0, "page": 1 } }
```

Single-row responses return `data` as an object.

---

## 3. Resources

`GET|POST /v1/<resource>` and `GET|PATCH|DELETE /v1/<resource>/<id>`.

| Resource path | Table | Notes |
|---------------|-------|-------|
| `parties` | parties | Customers + suppliers, opening balance, credit limit, loyalty config |
| `items` | items | Products & services, HSN, dual units, stock, images, `is_active` |
| `categories` | categories | Item categories |
| `units` | custom_units | User-defined units |
| `invoices` | invoices | `type`: `sale`, `purchase`, `sale_return`, `purchase_return` |
| `invoice-items` | invoice_items | Line items (filter with `?invoice_id=`) |
| `payments` | payments | Payment-in / payment-out |
| `expenses` | expenses | Business expenses with GST + ITC flag |
| `expense-payments` | expense_payments | Settlements against an expense |
| `documents` | documents | Estimates, sale orders, delivery challans, purchase orders |
| `document-items` | document_items | Document line items (filter with `?document_id=`) |
| `recurring-invoices` | recurring_invoices | Schedules that auto-generate invoices |
| `stock-adjustments` | stock_adjustments | Manual stock corrections |
| `loyalty-transactions` | loyalty_transactions | Points earned / redeemed ledger |
| `invoice-templates` | invoice_templates | Saved designer / layout templates |
| `business-profile` | business_profiles | Company profile, branding, module settings |
| `restaurant-areas` | restaurant_areas | Floors / sections |
| `restaurant-tables` | restaurant_tables | Table status & seating |
| `kots` | kots | Kitchen order tickets |
| `reservations` | reservations | Table bookings |
| `store-settings` | store_settings | Online store configuration |
| `store-items` | store_items | Published catalogue |
| `store-orders` | store_orders | Online orders |
| `store-order-items` | store_order_items | Online order lines |
| `store-customers` | store_customers | Storefront customers |
| `shared-reports` | shared_reports | Reports shared with users / parties |
| `webhooks` | webhook_endpoints | Register / update / delete webhook subscriptions |
| `webhook-deliveries` | webhook_deliveries | **read-only** delivery log with status codes |
| `activity-log` | activity_log | **read-only** audit trail |
| `members` | company_members | **read-only** team + permissions |
| `licenses` | licenses | **read-only** licence history |

`company_id` is injected automatically on create — never send it yourself.

### 3.1 Standard query parameters (every list endpoint)

| Param | Example | Effect |
|-------|---------|--------|
| `limit` / `per_page` | `?limit=100` | Page size, max 500 (default 50) |
| `offset` | `?offset=100` | Pagination offset |
| `page` | `?page=3&per_page=50` | 1-based paging (alternative to `offset`) |
| `sort` | `?sort=-date,invoice_number` | Multi-column sort, `-` prefix = descending |
| `order` + `dir` | `?order=date&dir=desc` | Legacy single-column sort |
| `search` | `?search=ravi` | Case-insensitive partial match on the resource search columns |
| `from` / `to` | `?from=2026-04-01&to=2026-06-30` | Inclusive date range |
| `date_field` | `?date_field=created_at` | Which column `from`/`to` apply to (default `date`) |
| `select` | `?select=id,name,stock` | Column projection |
| `count` | `?count=none` | Skip the total row count (faster on big tables) |
| any column | `?type=sale&status=unpaid` | Exact-match filter |

#### Filter operators

Append a double-underscore suffix to any column:

| Suffix | Meaning | Example |
|--------|---------|---------|
| `__neq` | not equal | `?status__neq=paid` |
| `__gt`, `__gte`, `__lt`, `__lte` | comparisons | `?total__gte=10000` |
| `__like` | case-insensitive contains | `?party_name__like=trad` |
| `__starts`, `__ends` | prefix / suffix match | `?invoice_number__starts=INV-2026` |
| `__in` | comma list | `?status__in=unpaid,partial` |
| `__null` | `true` / `false` | `?party_id__null=false` |

### Examples

```bash
BASE="https://qmbekbrvefqekhkapgrp.supabase.co/functions/v1/api/v1"
AUTH=(-H "Authorization: Bearer $TOKEN" -H "apikey: $ANON")

# Unpaid sales over ₹10,000 for Q1, newest first, page 2
curl -s "$BASE/invoices?type=sale&status__in=unpaid,partial&total__gte=10000\
&from=2026-04-01&to=2026-06-30&sort=-date&page=2&per_page=25" "${AUTH[@]}"

# Delivery challans only
curl -s "$BASE/documents?doc_type=delivery_challan&sort=-date" "${AUTH[@]}"

# Item catalogue delta sync (active items changed since last sync)
curl -s "$BASE/items?is_active=true&date_field=updated_at&from=2026-08-01\
&select=id,name,sale_price,stock,updated_at&sort=-updated_at" "${AUTH[@]}"

# Create a party
curl -s -X POST "$BASE/parties" "${AUTH[@]}" -H "Content-Type: application/json" \
  -d '{"name":"Ravi Traders","phone":"9876543210","type":"customer","state":"Maharashtra"}'

# Update stock on an item
curl -s -X PATCH "$BASE/items/<item-id>" "${AUTH[@]}" \
  -H "Content-Type: application/json" -d '{"stock":42}'

# Delete a payment
curl -s -X DELETE "$BASE/payments/<payment-id>" "${AUTH[@]}"
```

Bulk create: POST an **array** of objects to any resource and an array is returned.

---

## 4. Special endpoints

| Method & path | Purpose |
|---------------|---------|
| `GET /v1/health` | Liveness probe (no auth) |
| `GET /v1/openapi.json` | OpenAPI 3.1 contract (no auth) |
| `GET /v1/docs` | Human-readable reference page (no auth) |
| `GET /v1/me` | Caller identity, profile, active company, memberships & permissions |
| `GET /v1/companies` | Companies the caller can access (for a company switcher) |
| `GET /v1/invoices/:id/items` | Line items of one invoice |
| `POST /v1/invoices/with-items` | Create an invoice **and** its lines in one call |
| `GET /v1/reports/summary?from&to` | Sales, purchases, returns, payments, expenses, gross profit |
| `GET /v1/reports/gst-summary?from&to` | Output tax, input tax, credit/debit notes, net payable |
| `GET /v1/reports/stock?search&low_stock` | Item-wise stock, valuation and low-stock flags |
| `GET /v1/reports/outstanding?type` | Party-wise receivables and payables |
| `GET /v1/webhooks/events` | Catalogue of every subscribable event |
| `POST /v1/webhooks/:id/ping` | Send a signed test delivery to one endpoint |

### `POST /v1/invoices/with-items`

```json
{
  "invoice_number": "INV-1042",
  "date": "2026-08-19",
  "party_id": "…",
  "party_name": "Ravi Traders",
  "party_state": "Maharashtra",
  "type": "sale",
  "subtotal": 10000, "cgst": 900, "sgst": 900, "igst": 0,
  "total": 11800, "amount_paid": 0, "status": "unpaid",
  "items": [
    { "item_name": "Cement 50kg", "hsn_code": "2523", "unit": "bag",
      "qty": 10, "rate": 1000, "gst_rate": 18, "gst_amount": 1800, "amount": 11800 }
  ]
}
```

Returns `201` with the created invoice plus its `items`.

### Report response shapes

```bash
curl -s "$BASE/reports/summary?from=2026-04-01&to=2026-06-30" "${AUTH[@]}"
```
```json
{ "data": { "period": {"from":"2026-04-01","to":"2026-06-30"},
  "sales": { "count": 214, "total": 1842300, "received": 1600000 },
  "purchases": { "count": 88, "total": 980400, "paid": 910000 },
  "saleReturns": { "count": 4, "total": 12800 },
  "purchaseReturns": { "count": 1, "total": 2400 },
  "paymentsIn": 1620000, "paymentsOut": 905000, "expenses": 74300,
  "grossProfit": 612400 } }
```

```bash
curl -s "$BASE/reports/gst-summary?from=2026-04-01&to=2026-06-30" "${AUTH[@]}"
```
```json
{ "data": { "period": {...},
  "outputTax": { "taxable": 1560000, "cgst": 140400, "sgst": 140400, "igst": 0, "total": 1840800, "invoices": 214 },
  "inputTax":  { "taxable": 830000, "cgst": 74700, "sgst": 74700, "igst": 0, "total": 979400, "invoices": 88 },
  "creditNotes": {...}, "debitNotes": {...}, "netPayable": 131400 } }
```

```bash
curl -s "$BASE/reports/stock?low_stock=true" "${AUTH[@]}"
curl -s "$BASE/reports/outstanding?type=customer" "${AUTH[@]}"
```

Statutory returns (GSTR-1 / 2 / 3B / 9) are rendered in-app; use
`/reports/gst-summary` plus `invoices` + `invoice-items` to reproduce them
externally.

---

## 5. Webhooks

Register an endpoint (any resource CRUD rules apply):

```bash
curl -s -X POST "$BASE/webhooks" "${AUTH[@]}" -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/hooks/smartbooks",
       "events":["invoice.created","payment.created","stock.adjusted"],
       "description":"Mobile sync","is_active":true,"secret":"whsec_your_random_string"}'
```

`GET /v1/webhooks/events` returns the full catalogue:

```
invoice.created  invoice.updated  invoice.deleted
document.created document.updated document.deleted
delivery_challan.issued
payment.created  payment.updated  expense.created
stock.adjusted   item.created     item.updated
party.created    party.updated    kot.created
store_order.created  report.generated  ping
```

Set `events: ["*"]` to receive everything.

### Delivery format

```
POST <your url>
X-SmartBooks-Event      invoice.created
X-SmartBooks-Delivery   <uuid>
X-SmartBooks-Timestamp  1755859200
X-SmartBooks-Signature  sha256=<hex>
```
```json
{ "event": "invoice.created", "company_id": "…",
  "created_at": "2026-08-22T16:00:00.000Z", "data": { "id": "…", "invoice_number": "INV-1042" } }
```

Verify in Node:

```js
import crypto from 'node:crypto';
const expected = 'sha256=' + crypto.createHmac('sha256', SECRET)
  .update(`${req.headers['x-smartbooks-timestamp']}.${rawBody}`).digest('hex');
const ok = crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(req.headers['x-smartbooks-signature']));
```

Delivery is fire-and-forget (API latency is unaffected). Every attempt — status
code, response snippet, error — is logged in `webhook-deliveries`:

```bash
curl -s "$BASE/webhook-deliveries?sort=-created_at&limit=20" "${AUTH[@]}"
```

Send a test delivery: `POST /v1/webhooks/<endpoint-id>/ping` → `202`.

---

## 6. Mobile integration examples

Copy-paste starters using the exact URLs above.

### 6.1 Constants

```ts
export const SUPABASE_URL = 'https://qmbekbrvefqekhkapgrp.supabase.co';
export const ANON_KEY = '<SUPABASE_PUBLISHABLE_KEY>';
export const API = `${SUPABASE_URL}/functions/v1/api/v1`;
```

### 6.2 Auth flow (React Native / Flutter-agnostic fetch)

```ts
export async function signIn(email: string, password: string) {
  const r = await fetch(`${SUPABASE_URL}/auth/v1/token?grant_type=password`, {
    method: 'POST',
    headers: { apikey: ANON_KEY, 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });
  if (!r.ok) throw new Error((await r.json()).error_description ?? 'Login failed');
  const s = await r.json();               // { access_token, refresh_token, expires_in }
  await secureStore.set('refresh_token', s.refresh_token);   // Keychain / Keystore
  return s;
}

export async function refresh(refreshToken: string) {
  const r = await fetch(`${SUPABASE_URL}/auth/v1/token?grant_type=refresh_token`, {
    method: 'POST',
    headers: { apikey: ANON_KEY, 'Content-Type': 'application/json' },
    body: JSON.stringify({ refresh_token: refreshToken }),
  });
  return r.json();
}

let companyId: string | null = null;
export async function api(path: string, init: RequestInit = {}, token: string) {
  const r = await fetch(`${API}${path}`, {
    ...init,
    headers: {
      Authorization: `Bearer ${token}`,
      apikey: ANON_KEY,
      'Content-Type': 'application/json',
      ...(companyId ? { 'X-Company-Id': companyId } : {}),
      ...(init.headers || {}),
    },
  });
  const body = await r.json();
  if (!r.ok) throw new Error(body?.error?.message ?? `HTTP ${r.status}`);
  return body;
}

export async function bootstrap(token: string) {
  const me = await api('/me', {}, token);
  companyId = me.data.companyId;          // cache it
  return me.data;
}
```

### 6.3 Create a bill from the mobile billing screen

```ts
const { data: invoice } = await api('/invoices/with-items', {
  method: 'POST',
  body: JSON.stringify({
    invoice_number: 'INV-1042',
    date: new Date().toISOString().slice(0, 10),
    party_id: party.id, party_name: party.name, party_state: party.state,
    type: 'sale',
    subtotal: 10000, cgst: 900, sgst: 900, igst: 0,
    total: 11800, amount_paid: 11800, status: 'paid',
    items: lines.map(l => ({
      item_id: l.id, item_name: l.name, hsn_code: l.hsn, unit: l.unit,
      qty: l.qty, rate: l.rate, gst_rate: l.gst,
      gst_amount: l.qty * l.rate * l.gst / 100,
      amount: l.qty * l.rate * (1 + l.gst / 100),
    })),
  }),
}, token);
```

### 6.4 Dashboard tiles

```ts
const [summary, outstanding, lowStock] = await Promise.all([
  api('/reports/summary?from=2026-04-01&to=2026-06-30', {}, token),
  api('/reports/outstanding?type=customer', {}, token),
  api('/reports/stock?low_stock=true', {}, token),
]);
```

### 6.5 Offline-first delta sync

```ts
const since = await db.get('lastSync');   // ISO timestamp
const { data } = await api(
  `/items?date_field=updated_at&from=${since}&sort=-updated_at&per_page=500&count=none`,
  {}, token,
);
await db.upsertItems(data);
await db.set('lastSync', new Date().toISOString());
```

### 6.6 Kotlin (Android) one-liner example

```kotlin
val req = Request.Builder()
  .url("$API/invoices?type=sale&status=unpaid&sort=-date&per_page=50")
  .addHeader("Authorization", "Bearer $token")
  .addHeader("apikey", ANON_KEY)
  .addHeader("X-Company-Id", companyId)
  .build()
client.newCall(req).execute().use { println(it.body?.string()) }
```

### 6.7 Swift (iOS)

```swift
var req = URLRequest(url: URL(string: "\(api)/reports/summary?from=2026-04-01&to=2026-06-30")!)
req.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
req.setValue(anonKey, forHTTPHeaderField: "apikey")
req.setValue(companyId, forHTTPHeaderField: "X-Company-Id")
let (data, _) = try await URLSession.shared.data(for: req)
```

---

## 7. Postman / Insomnia collection

`docs/smartbooks-api.postman_collection.json` imports into both tools. It ships:

- variables `baseUrl`, `anonKey`, `email`, `password`, `companyId`, `token`
- a **Sign in** request that stores `token` automatically
- folders for System, Identity, Resources, Invoices, Reports and Webhooks
- tests asserting `200/201` and the `data` envelope on every request

Import → set `baseUrl` + `anonKey` + credentials → *Run collection*.

## 8. Automated contract tests

`src/test/api-contract.test.ts` (vitest) covers the happy path plus
auth/RLS negatives: no token → 401, bad token → 401, unknown resource → 404,
write to a read-only resource → 405, foreign `X-Company-Id` → 403.

```bash
SB_API_BASE="https://qmbekbrvefqekhkapgrp.supabase.co/functions/v1/api/v1" \
SB_ANON_KEY="<publishable key>" \
SB_TEST_EMAIL="owner@example.com" SB_TEST_PASSWORD="••••" \
npm test -- api-contract
```

All live cases are opt-in: without `SB_API_BASE` + `SB_ANON_KEY` the whole suite
is skipped, and without `SB_TEST_EMAIL` / `SB_TEST_PASSWORD` only the public and
negative cases run — so CI never fails on missing secrets.

## 9. Performance testing

```bash
# k6 smoke test
k6 run - <<'JS'
import http from 'k6/http';
import { check } from 'k6';
export const options = { vus: 20, duration: '30s' };
const BASE = `${__ENV.BASE}`;
const headers = { Authorization: `Bearer ${__ENV.TOKEN}`, apikey: `${__ENV.ANON}` };
export default function () {
  const r = http.get(`${BASE}/invoices?limit=50&count=none`, { headers });
  check(r, { 'status 200': (res) => res.status === 200 });
}
JS
```

Guidelines:
- Warm up with `GET /v1/health` (edge functions cold-start on first hit).
- Keep `limit <= 500`; deep paginate with `offset`/`page`.
- Add `count=none` in load tests — skipping the exact count is markedly faster.
- Reuse one access token for the whole run; refresh before the 1-hour expiry.
- Test writes against a throwaway company so RLS-scoped data stays clean.

## 10. Deploying / versioning

The function is declared in `supabase/config.toml` as `[functions.api] verify_jwt = false`
(the JWT is validated inside the function so that `/v1/health`, `/v1/docs` and
`/v1/openapi.json` stay public). Deploy with `supabase functions deploy api`.
Webhooks require the `038_api_webhooks.sql` migration. Breaking changes get a new
`/v2` prefix; `/v1` stays additive.
