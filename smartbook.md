# SmartBooks — Accounting, Inventory & Invoicing Platform

> A multi-tenant, full-featured invoicing and accounting SaaS designed for Indian businesses with GST compliance, multi-company support, role-based access, Razorpay payments, and a rich set of business reports.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Architecture Overview](#3-architecture-overview)
4. [Project Structure](#4-project-structure)
5. [Getting Started](#5-getting-started)
6. [Environment Variables](#6-environment-variables)
7. [Application Workflow Diagrams](#7-application-workflow-diagrams)
   - 7.1 [Authentication Flow](#71-authentication-flow)
   - 7.2 [Multi-Company & Tenant Flow](#72-multi-company--tenant-flow)
   - 7.3 [Invoice Creation Flow](#73-invoice-creation-flow)
   - 7.4 [Payment Processing Flow](#74-payment-processing-flow)
   - 7.5 [License & Billing Flow](#75-license--billing-flow)
   - 7.6 [Report Generation & Sharing Flow](#76-report-generation--sharing-flow)
   - 7.7 [Party Portal Flow](#77-party-portal-flow)
   - 7.8 [Backup & Restore Flow](#78-backup--restore-flow)
   - 7.9 [Admin User Management Flow](#79-admin-user-management-flow)
   - 7.10 [Offline-First Sync Architecture](#710-offline-first-sync-architecture)

8. [Database Schema](#8-database-schema)
   - 8.1 [Entity Relationship Diagram](#81-entity-relationship-diagram)
   - 8.2 [Tables Reference](#82-tables-reference)
   - 8.3 [Enum Types](#83-enum-types)
   - 8.4 [Views](#84-views)
   - 8.5 [Database Functions](#85-database-functions)
   - 8.6 [Triggers](#86-triggers)
   - 8.7 [Indexes](#87-indexes)
   - 8.8 [Row-Level Security (RLS) Policies](#88-row-level-security-rls-policies)
   - 8.9 [Storage Buckets](#89-storage-buckets)
9. [Database Migration History](#9-database-migration-history)
10. [Frontend Architecture](#10-frontend-architecture)
    - 10.1 [Context Provider Hierarchy](#101-context-provider-hierarchy)
    - 10.2 [Routing & Page Guards](#102-routing--page-guards)
    - 10.3 [Pages](#103-pages)
    - 10.4 [Components](#104-components)
    - 10.5 [Custom Hooks](#105-custom-hooks)
    - 10.6 [Utility Modules](#106-utility-modules)
11. [Supabase Edge Functions](#11-supabase-edge-functions)
12. [GST Compliance](#12-gst-compliance)
13. [Licensing & Billing System](#13-licensing--billing-system)
14. [Role-Based Access Control (RBAC)](#14-role-based-access-control-rbac)
15. [Key Features Summary](#15-key-features-summary)
16. [Testing](#16-testing)
17. [Scripts Reference](#17-scripts-reference)
18. [Module Reference (Documents, Restaurant, POS, Loyalty, Store)](#18-module-reference)
19. [End-to-End Application Flow](#19-end-to-end-application-flow)
20. [REST API (v1)](#20-rest-api-v1)
21. [Related Documentation](#21-related-documentation)

---

## 1. Project Overview

**SmartBooks** is a cloud-native, offline-capable invoicing, accounting, inventory and retail platform built for Indian small-to-medium businesses. It runs as a single-page React app on Supabase (Postgres + Auth + Edge Functions + Storage), is installable as a PWA, and is also shipped as a thin Capacitor (Android/iOS) and Electron (desktop) shell around the hosted site.

**Core accounting**

- **Invoicing** — sale, purchase, sale-return (credit note) and purchase-return (debit note) invoices with automatic GST split (CGST/SGST/IGST), reverse calculations, extra charges/discounts, round-off and 7 print formats
- **Sales documents** — estimates/quotations, sale orders, purchase orders and delivery challans, each convertible into an invoice
- **Recurring invoices** — daily/weekly/monthly/quarterly/yearly schedules with next-run tracking and auto-generation
- **Payments** — payment-in and payment-out, invoice linking, outstanding tracking, receipts and vouchers
- **Expenses** — categories, GST/ITC eligibility, vendor linking, part payments and recurrence
- **Inventory** — items and services with dual units, HSN, barcodes, images, opening stock, min-stock alerts and manual stock adjustments
- **Parties** — customer/supplier directory with GSTIN, opening balances, credit limits and per-party ledger
- **Reports** — 18 report types plus an expense report, each exportable to PDF/Excel, printable, shareable by email or public link
- **GST** — GSTR-1, GSTR-2, GSTR-3B and GSTR-9 layouts, in-app view and Excel export

**Selling channels & industry modules**

- **Point of Sale** — touch billing screen with tiles, keypad, barcode scanning and thermal 58/80 mm receipts
- **Restaurant** — areas, tables and live table status, KOT + Kitchen Display, reservations, table-to-bill
- **Online store** — publishable catalogue, public storefront, online orders/customers and order-to-invoice conversion
- **Loyalty & rewards** — points rules, tiers, redemption as invoice discount, printable loyalty cards and a public card portal
- **Industry packs** — pharma, manufacturing, garment, jewellery and restaurant field sets, plus user-defined custom fields

**Platform**

- **Multi-company** — one login, many businesses, strict per-company isolation enforced by RLS
- **Team collaboration** — owner/admin/staff/member/party roles with per-page view/create/edit/delete permissions
- **Party portal** — external customers/suppliers sign in to see only their own invoices, payments and shared reports
- **Public sharing** — password-protectable invoice/payment/report links with expiry, revocation, view counts and email delivery
- **AI invoice reading** — upload a bill as PDF/JPG/PNG and have the sale/purchase panel prefilled; multi-provider fallback chain, file never stored
- **Offline-first** — writes queue in IndexedDB and replay automatically when connectivity returns, with a live network/sync indicator
- **Licensing & billing** — trial/paid/complimentary licences, Razorpay checkout, coupons and redeemable licence keys
- **Backup & restore** — full per-company `.bkp` export/import with FK-aware ordering
- **REST API v1** — a single edge function exposing every business resource for mobile apps and integrations
- **Super Admin console** — a separate `super.html` build for cross-tenant businesses, users, licences and AI provider keys

### Business Context

- **Target Market**: Indian SMBs requiring GST-compliant invoicing
- **Primary currency**: INR (₹). An optional multi-currency mode exists (`src/lib/currency.ts` + `business_profiles.multi_currency_enabled`) that adds a per-invoice currency and exchange rate with live FX rates; it is **off by default and has no toggle on the Settings page** — it is switched from the Backup & Restore screen's data/settings section
- **GST Logic**: Intra-state → CGST + SGST (50/50 split), Inter-state → IGST
- **Supported GST Rates**: 0%, 5%, 12%, 18%, 28%
- **All 36 Indian States/UTs** supported for GST Place of Supply
- **Invoice languages**: 11 Indian languages with per-company font selection

---

## 2. Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 18, TypeScript 5, Vite 5 (two entries: `index.html` app, `super.html` console) |
| **Styling** | Tailwind CSS 3 + typography plugin, shadcn/ui (Radix UI primitives), `next-themes` |
| **State** | React Context (Auth, Company, License, App, Expense, Documents, Fields, Restaurant, Theme) + TanStack React Query |
| **Routing** | React Router DOM v6 |
| **Forms** | React Hook Form + Zod (`@hookform/resolvers`) |
| **Animations / charts** | Framer Motion, Recharts |
| **Backend** | Supabase — PostgreSQL, Auth (GoTrue), Edge Functions (Deno), Storage |
| **Offline** | IndexedDB outbox + snapshot cache (`src/lib/offline/*`), no external dependency |
| **Payments** | Razorpay Checkout (script loaded on demand) + Orders API + webhook |
| **AI** | Gemini / OpenAI / Anthropic / OpenRouter / any OpenAI-compatible endpoint, with the Lovable AI gateway as a final fallback |
| **Email** | Resend (share links, password reset) |
| **PDF / print** | HTML-to-print via iframe (`src/utils/printHtml.ts`), `jspdf` + `html2canvas` (`src/lib/htmlToPdf.ts`) |
| **Barcodes / QR** | `jsbarcode`, `qrcode`, `@zxing/browser` (camera & USB scanning) |
| **Excel/CSV** | `xlsx` (SheetJS), `papaparse` |
| **Voice input** | Web Speech API (SpeechRecognition) |
| **Packaging** | PWA (`public/manifest.webmanifest`), Capacitor 8 (Android/iOS shell), Electron (`electron/main.cjs`) |
| **Hosting** | Vercel (`vercel.json` SPA rewrites) |
| **Testing** | Vitest + Testing Library, Playwright |
| **Linting** | ESLint 9 with TypeScript + React Hooks plugins |
| **Package Manager** | Bun (bun.lockb) |

---

## 3. Architecture Overview

```text
┌───────────────────────────────────────────────────────────────────────┐
│  Clients                                                              │
│  Browser SPA / installed PWA  •  Capacitor Android+iOS shell          │
│  Electron desktop shell       •  super.html (Super Admin console)     │
└───────────────────────────────────────────────────────────────────────┘
                                  │
┌───────────────────────────────────────────────────────────────────────┐
│  React application (src/)                                             │
│                                                                       │
│  AuthContext ──► CompanyContext ──► LicenseContext ──► LicenseGate     │
│                        │                                              │
│                        ▼                                              │
│   AppContext (parties, items, invoices, payments, profile, loyalty,    │
│               stock adjustments)                                      │
│     ├── ExpenseContext      ├── DocumentsContext (docs + recurring)    │
│     ├── FieldsContext       └── RestaurantContext                      │
│                        │                                              │
│   Pages (48) ── Components (60+) ── Hooks (10) ── utils/ + lib/        │
│                        │                                              │
│                        ▼                                              │
│   offline.from()  ─────────────► IndexedDB  (kv snapshot + outbox)     │
│   (Proxy over supabase.from)          ▲                               │
│                        │              │ replay when reachable         │
│                        ▼              │                               │
│              sync engine (src/lib/offline/sync.ts)                    │
└───────────────────────────────────────────────────────────────────────┘
                                  │ HTTPS / WebSocket
                                  ▼
┌───────────────────────────────────────────────────────────────────────┐
│  Supabase                                                             │
│  ┌────────────────┐ ┌───────────────┐ ┌────────────────────────────┐  │
│  │  PostgreSQL    │ │ Auth (GoTrue) │ │  Edge Functions (Deno)     │  │
│  │  50+ tables    │ │ email + phone │ │  api (REST v1)             │  │
│  │  RLS on all    │ │ JWT sessions  │ │  create-user, manage-user  │  │
│  │  company-      │ │               │ │  lookup-email              │  │
│  │  scoped tables │ │               │ │  parse-invoice-file (AI)   │  │
│  │  RPC helpers   │ │               │ │  public-doc, public-report │  │
│  │  (has_role,    │ │               │ │  send-doc-link,            │  │
│  │  is_company_*, │ │               │ │  send-report-link,         │  │
│  │  redeem_       │ │               │ │  send-password-reset       │  │
│  │  license, sa_*)│ │               │ │  razorpay-* (3)            │  │
│  └────────────────┘ └───────────────┘ │  loyalty-card-lookup       │  │
│  ┌────────────────┐                   │  super-admin               │  │
│  │ Storage        │                   └────────────────────────────┘  │
│  │ shared-pdfs,   │                                                   │
│  │ item/store img │                                                   │
│  └────────────────┘                                                   │
└───────────────────────────────────────────────────────────────────────┘
        │                    │                       │
        ▼                    ▼                       ▼
  Razorpay (orders,   Resend (share &        AI providers (Gemini,
  checkout, webhook)  reset emails)          OpenAI, Anthropic,
                                             OpenRouter, Lovable AI)
```

---

## 4. Project Structure

```text
smartbooks/
├── public/
│   ├── manifest.webmanifest            # PWA manifest (installable app)
│   ├── favicon.svg / favicon.ico / apple-touch-icon.png / pwa-192.png / pwa-512.png
│   ├── icons.svg                       # sprite used by print templates
│   └── robots.txt
├── index.html                          # main SPA entry
├── super.html                          # Super Admin console entry (separate bundle)
├── electron/main.cjs                   # Electron desktop shell (loads hosted site)
├── capacitor.config.ts                 # Android / iOS shell config
├── vercel.json                         # SPA rewrites for hosting
├── docs/
│   ├── FEATURES.md                     # plain-English feature list
│   ├── API.md                          # REST API v1 reference
│   └── smartbooks-api.postman_collection.json
├── src/
│   ├── main.tsx                        # app entry — starts the offline sync engine, mounts App
│   ├── App.tsx                         # providers, LicenseGate, all route definitions
│   ├── index.css / App.css             # Tailwind layers + design tokens
│   ├── assets/                         # hero and auth imagery
│   ├── components/
│   │   ├── AppLayout.tsx               # shell: sidebar + header + content + HeaderPortal
│   │   ├── AppSidebar.tsx              # permission-filtered navigation, module toggles
│   │   ├── NetworkStatus.tsx           # online/offline dot, pending count, manual sync
│   │   ├── AlertsBell.tsx              # low stock, overdue parties, licence expiry
│   │   ├── CompanySwitcher.tsx         # switch/create companies
│   │   ├── UserMenu.tsx / ThemeToggle.tsx / NavLink.tsx / HeaderPortal.tsx
│   │   ├── InvoiceUploadDialog.tsx     # AI bill upload → prefilled invoice panel
│   │   ├── DocumentItemsTable.tsx      # shared spreadsheet line grid (invoices + documents)
│   │   ├── InlineItemSearch.tsx / LineItemNameInput.tsx / ItemPickerDialog.tsx
│   │   ├── QuickAddItemDialog.tsx / QuickAddPartyDialog.tsx / PartyCombobox.tsx
│   │   ├── ExtraChargesEditor.tsx / SignaturePad.tsx / SpreadsheetEditor.tsx
│   │   ├── InvoicePreviewDialog.tsx / InvoiceFullPreview.tsx / PaymentPreviewDialog.tsx
│   │   ├── InvoiceDesigner.tsx / InvoiceCanvasDesigner.tsx / BackgroundEditor.tsx
│   │   ├── CustomFieldsManager.tsx / CustomFieldsSection.tsx / IndustrySettingsPanel.tsx
│   │   ├── EwayBillSection.tsx / EwaySettingsPanel.tsx
│   │   ├── PosSettingsPanel.tsx / BillingZoomControl.tsx / BarcodeScannerDialog.tsx
│   │   ├── BulkBarcodePrintDialog.tsx / BulkActionBar.tsx / SortHeader.tsx / StatCard.tsx
│   │   ├── KotOrderDialog.tsx / RestaurantSettingsPanel.tsx
│   │   ├── LoyaltyCardPrintDialog.tsx / LoyaltyRedeemCard.tsx / LoyaltyRedeemPanel.tsx
│   │   │   / PartyLoyaltyFields.tsx / RewardsPosSettings.tsx
│   │   ├── ShareLinkPanel.tsx / SendDocLinkDialog.tsx / ShareReportDialog.tsx
│   │   │   / ReportViewDialog.tsx / SentReportsTab.tsx / SharedReportsTab.tsx
│   │   ├── WhatsAppShareDialog.tsx / WhatsAppSettingsPanel.tsx
│   │   ├── CouponsAdmin.tsx / LicenseHistory.tsx / SavingOverlay.tsx / SmartBooksLogo.tsx
│   │   └── ui/                         # shadcn/ui library (48 primitives)
│   ├── contexts/
│   │   ├── AuthContext.tsx             # Supabase session, user, profile
│   │   ├── CompanyContext.tsx          # memberships, active company, usePermissions()
│   │   ├── LicenseContext.tsx          # licence state, isExpired, isSuperAdmin
│   │   ├── AppContext.tsx              # core data store (offline-aware CRUD)
│   │   ├── ExpenseContext.tsx          # expenses + expense payments
│   │   ├── DocumentsContext.tsx        # estimates/orders/challans + recurring invoices
│   │   ├── FieldsContext.tsx           # custom fields + active industry pack
│   │   ├── RestaurantContext.tsx       # areas, tables, KOTs, reservations
│   │   └── ThemeContext.tsx            # light/dark/system
│   ├── hooks/                          # use-mobile, use-toast, use-sync-status,
│   │                                   # use-billing-zoom, use-custom-templates,
│   │                                   # use-hidden-columns, use-persisted-columns,
│   │                                   # use-row-activate, use-row-selection,
│   │                                   # use-show-inactive-items
│   ├── industries/                     # pharma, manufacturing, garment, jewellery,
│   │                                   # restaurant packs + shared types
│   ├── integrations/supabase/          # client.ts singleton, generated types.ts
│   ├── lib/
│   │   ├── offline/idb.ts              # IndexedDB kv + outbox stores
│   │   ├── offline/sync.ts             # outbox queue, reachability probe, replay loop
│   │   ├── offline/offlineClient.ts    # offline.from() Proxy over supabase.from()
│   │   ├── currency.ts                 # optional multi-currency + live FX rates
│   │   ├── i18n.ts                     # 11-language invoice dictionary
│   │   ├── barcode.ts / loyalty.ts / loyaltyCard.ts
│   │   ├── shareLinks.ts / reportShare.ts
│   │   ├── whatsapp.ts / whatsappDocs.ts
│   │   ├── htmlToPdf.ts / backgroundSamples.ts / utils.ts
│   ├── pages/                          # see §10.3 for the full list (48 pages)
│   ├── super/                          # Super Admin console (own entry point)
│   │   ├── main.tsx / SuperConsole.tsx / lib/api.ts
│   │   └── pages/SuperOverview|SuperBusinesses|SuperUsers|SuperLicenses|SuperAiProviders
│   ├── test/                           # setup.ts, example.test.ts, api-contract.test.ts
│   ├── types/                          # index.ts, documents.ts, expense.ts, restaurant.ts
│   └── utils/                          # calculation, print, export and report modules (§10.6)
├── supabase/
│   ├── config.toml                     # edge function JWT settings
│   ├── functions/                      # 15 edge functions (§11)
│   └── manual-migrations/              # 001 → 042 SQL scripts, run in order (§9)
├── package.json / bun.lockb
├── vite.config.ts                      # port 8080, two build entries
├── tailwind.config.ts / postcss.config.js / components.json
├── tsconfig*.json / eslint.config.js
└── playwright.config.ts / playwright-fixture.ts
```

---

## 5. Getting Started

### Prerequisites

- **Node.js** 18+ or **Bun** runtime
- **Supabase** account and project
- **Razorpay** account (for billing features)

### Installation

```bash
# Clone the repository
git clone <repository-url>
cd remix-of-lighting-fast-main

# Install dependencies (using Bun)
bun install

# Or using npm
npm install
```

### Development

```bash
# Start dev server (runs on port 8080)
bun dev
# or
npm run dev
```

### Build

```bash
# Production build
bun run build

# Development build (with source maps)
bun run build:dev
```

### Preview Production Build

```bash
bun run preview
```

---

## 6. Environment Variables

Create a `.env` file in the project root:

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=eyJhbGciOiJIUzI1NiIs...
```

Edge function secrets (set in the Supabase dashboard → Edge Functions → Secrets):

| Secret | Used by | Required for |
|--------|---------|--------------|
| `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` | all functions | injected by Supabase; anon key is used to build RLS-scoped clients |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` | `razorpay-create-order`, `razorpay-verify-payment` | online licence purchase |
| `RAZORPAY_WEBHOOK_SECRET` | `razorpay-webhook` | async payment/refund events |
| `RESEND_API_KEY` | `send-doc-link`, `send-report-link`, `send-password-reset` | all outbound email |
| `RESET_FROM_EMAIL` | `send-password-reset` (+ fallback sender elsewhere) | password reset mails |
| `SHARE_FROM_EMAIL` | `send-doc-link`, `send-report-link` | optional sender override for share mails |
| `GEMINI_API_KEY` | `parse-invoice-file` | AI invoice reading (fallback after `ai_providers` rows) |
| `LOVABLE_API_KEY` | `parse-invoice-file` | last-resort AI gateway |

Additional AI provider keys (OpenAI, Anthropic, OpenRouter, any OpenAI-compatible endpoint)
are **not** env vars — they live in the service-role-only `ai_providers` table and are
managed from Super Admin → AI providers.

---

## 7. Application Workflow Diagrams

### 7.1 Authentication Flow

```
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  Landing    │     │  Auth Page   │     │   Supabase     │
│  Page (/)   │───▶ │  (/auth)     │     │   Auth         │
└─────────────┘     └──────┬───────┘     └───────┬────────┘
                           │                     │
              ┌────────────┼────────────┐        │
              ▼            ▼            ▼        │
        ┌──────────┐ ┌──────────┐ ┌──────────┐   │
        │ Email    │ │ Phone    │ │ Sign Up  │   │
        │ Login    │ │ Login    │ │ (if      │   │
        └────┬─────┘ └────┬─────┘ │ enabled) │   │
             │            │       └────┬─────┘   │
             │            │            │         │
             │     ┌──────▼──────┐     │         │
             │     │ lookup-email│     │         │
             │     │ Edge Fn     │     │         │
             │     │ (phone→email)     │         │
             │     └──────┬──────┘     │         │
             │            │            │         │
             ▼            ▼            ▼         │
        ┌─────────────────────────────────────┐  │
        │   supabase.auth.signInWithPassword  │──┤
        │   supabase.auth.signUp              │  │
        └─────────────────────────────────────┘  │
                                                 ▼
                                          ┌──────────────┐
                                          │  Triggers:   │
                                          │  1. Profile  │
                                          │  2. Role     │
                                          │  3. Company  │
                                          │  4. Trial    │
                                          │     License  │
                                          └──────┬───────┘
                                                 │
                                                 ▼
                                          ┌──────────────┐
                                          │  Dashboard / │
                                          │  redirect    │
                                          └──────────────┘

On Sign Up (auth.users AFTER INSERT triggers):
  1. handle_new_user()        → Creates profile row
  2. handle_new_user_role()   → Assigns 'admin' (first user) or 'user' role
  3. handle_new_user_company()→ Creates "Default Company" + owner membership
                                + business_profiles row
  4. create_trial_license()   → Creates trial license (N days from app_settings)
```

### 7.2 Multi-Company & Tenant Flow

```
┌────────────────────────────────────────────────────-───┐
│                    User Logs In                        │
│                        │                               │
│                        ▼                               │
│              ┌─────────────────┐                       │
│              │ CompanyContext  │                       │
│              │ fetchMemberships│                       │
│              └────────┬────────┘                       │
│                       │                                │
│            ┌──────────▼────────────┐                   │
│            │  company_members      │                   │
│            │  JOIN companies       │                   │
│            │  WHERE user_id = me   │                   │
│            └──────────┬────────────┘                   │
│                       │                                │
│         ┌─────────────▼──────────────┐                 │
│         │  Returns: Company[] with   │                 │
│         │  role, page_permissions,   │                 │
│         │  can_edit_payments         │                 │
│         └─────────────┬──────────────┘                 │
│                       │                                │
│          ┌────────────▼─────────────┐                  │
│          │ Active company picked    │                  │
│          │ (localStorage persisted) │                  │
│          └────────────┬─────────────┘                  │
│                       │                                │
│          ┌────────────▼─────────────┐                  │
│          │ AppContext.fetchAll()    │                  │
│          │ Loads ALL data scoped    │                  │
│          │ to active company_id     │                  │
│          └──────────────────────────┘                  │
│                                                        │
│  ┌──────────────────────────────────────────────┐      │
│  │           CompanySwitcher UI                 │      │
│  │  ┌──────────────┐  ┌──────────────────────┐  │      │
│  │  │ Company A ✓  │  │ + Create New Company │  │      │
│  │  │ Company B    │  │   (name → insert →   │  │      │
│  │  │ Company C    │  │    owner membership  │  │      │
│  │  └──────────────┘  │    → business profile│  │      │
│  │                    │    → trial license)  │  │      │
│  │                    └──────────────────────┘  │      │
│  └──────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────-─┘

Data Isolation (via RLS):
  - All business tables have company_id column
  - RLS policies enforce: is_company_member(company_id, auth.uid())
  - Even if SQL is injected, PostgreSQL enforces isolation
```

### 7.3 Invoice Creation Flow

```
┌──────────────────────────────────────────────────────────────-──┐
│                    CreateInvoice Page                           │
│                                                                 │
│  URL: /sales/new?type=sale|purchase|sale_return|purchase_return │
│                                                                 │
│  ┌──────────────── Multi-Tab System ────────────────────┐       │
│  │ [Tab 1: INV-001] [Tab 2: INV-002] [+]                │       │
│  │                                                      │       │
│  │  ┌─── Party Selection ───────────────────────────┐   │       │
│  │  │ PartyCombobox (search by name/phone)          │   │       │
│  │  │ [+ Quick Add Party] → QuickAddPartyDialog     │   │       │
│  │  │ Auto-fills: state, GSTIN for GST calculation  │   │       │
│  │  └───────────────────────────────────────────────┘   │       │
│  │                                                      │       │
│  │  ┌─── Line Items ───────────────────────────────────┐│       │
│  │  │ [Add Item] → ItemPickerDialog (search, multi)    ││       │
│  │  │ [Voice]    → VoiceItemInput (speech-to-item)     ││       │
│  │  │ [+ Quick]  → QuickAddItemDialog (inline create)  ││       │
│  │  │                                                  ││       │
│  │  │ ┌───┬──────┬────┬─────┬──────┬────┬─────┬─────┐  ││       │
│  │  │ │Itm│ Qty  │Unit│ MRP │ Rate │Disc│ GST │ Amt │  ││       │
│  │  │ ├───┼──────┼────┼─────┼──────┼────┼─────┼─────┤  ││       │
│  │  │ │...│ edit │ ...│ edit│ edit │edit│edit │edit │  ││       │
│  │  │ └───┴──────┴────┴─────┴──────┴────┴─────┴─────┘  ││       │
│  │  │                                                  ││       │
│  │  │ Reverse Calculations:                            ││       │
│  │  │  Edit Amount → solveRateFromAmount()             ││       │
│  │  │  Edit GST ₹  → solveGstRateFromGstAmount()       ││       │
│  │  └──────────────────────────────────────────────────┘│       │
│  │                                                      │       │
│  │  ┌─── Extra Charges/Discounts ──────────────────────┐│       │
│  │  │ ExtraChargesEditor                               ││       │
│  │  │ Charges: TCS, Freight, Packing (flat/%)          ││       │
│  │  │ Discounts: Trade, Loyalty, Cash (flat/%)         ││       │
│  │  └──────────────────────────────────────────────────┘│       │
│  │                                                      │       │
│  │  ┌─── Totals ───────────────────────────────────────┐│       │
│  │  │ Subtotal: Σ line amounts                         ││       │
│  │  │ + Extra Charges / - Extra Discounts              ││       │
│  │  │ GST Split:                                       ││       │
│  │  │   Same State  → CGST (50%) + SGST (50%)          ││       │
│  │  │   Diff State  → IGST (100%)                      ││       │
│  │  │ Round Off: auto (manual override)                ││       │
│  │  │ Grand Total: computed                            ││       │
│  │  │ Payment Received: toggle + amount                ││       │
│  │  └──────────────────────────────────────────────────┘│       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  On Save:                                                       │
│  ┌────────────────────────────────────────────────────┐         │
│  │ 1. Insert invoice row (invoices table)             │         │
│  │ 2. Insert invoice_items rows (bulk)                │         │
│  │ 3. Adjust stock per line item:                     │         │
│  │    • sale/sale_return(credit): stock -= qty        │         │
│  │    • purchase/purchase_return(debit): stock += qty │         │
│  │ 4. If payment received > 0:                        │         │
│  │    • Insert payment row                            │         │
│  │    • Update invoice status (paid/partial)          │         │
│  │ 5. Log activity                                    │         │
│  └────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### 7.4 Payment Processing Flow

```
┌───────────────────────────────────────────────────┐
│            PaymentIn / PaymentOut Page            │
│                                                   │
│  ┌─ New Payment Dialog ──────────────────────┐    │
│  │                                           │    │
│  │  Party: [PartyCombobox]                   │    │
│  │    ├─ Auto-fills outstanding balance      │    │
│  │    └─ Filters: customers (in) /           │    │
│  │            suppliers (out)                │    │
│  │                                           │    │
│  │  Amount: [₹ _____] (prefilled from        │    │
│  │          outstanding)                     │    │
│  │                                           │    │
│  │  Mode: [Cash] [Bank] [UPI] [Cheque]       │    │
│  │                                           │    │
│  │  Link Invoice: [Dropdown of unpaid        │    │
│  │                 invoices for party]       │    │
│  │                                           │    │
│  │  Date: [DatePicker]                       │    │
│  │  Notes: [TextField]                       │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  On Save:                                         │
│  ┌──────────────────────────────────────────┐     │
│  │ 1. Insert payment row                    │     │
│  │ 2. If linked to invoice:                 │     │
│  │    • invoice.amountPaid += payment.amount│     │
│  │    • invoice.status =                    │     │
│  │        amountPaid >= total ? 'paid'      │     │
│  │        : amountPaid > 0 ? 'partial'      │     │
│  │        : 'unpaid'                        │     │
│  │ 3. Log activity                          │     │
│  └──────────────────────────────────────────┘     │
│                                                   │
│  Outstanding Calculation (getPartyOutstanding):   │
│  ┌───────────────────────────────────────────┐    │
│  │ openingBalance                            │    │
│  │ + Σ(sale invoices total)                  │    │
│  │ - Σ(purchase invoices total)              │    │
│  │ - Σ(payments_in amount)                   │    │
│  │ + Σ(payments_out amount)                  │    │
│  │ + Σ(sale_return total)                    │    │
│  │ - Σ(purchase_return total)                │    │
│  │ = Outstanding Balance                     │    │
│  └───────────────────────────────────────────┘    │
└───────────────────────────────────────────────────┘
```

### 7.5 License & Billing Flow

```
┌────────────────────────────────────────────────────────────┐
│                     Billing Page                           │
│                                                            │
│  ┌─── Current License Status ───────────────────────────┐  │
│  │ Type: Trial/Paid/Complimentary  Status: Active       │  │
│  │ Seats: 3   Expires: 2026-05-01  Days Left: 7         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌─── Plans ────────────────────────────────────────────┐  │
│  │                                                      │  │
│  │  ┌─ Monthly ─┐  ┌─ Lifetime ─┐                       │  │
│  │  │ ₹199/mo   │  │ ₹9,999     │                       │  │
│  │  │ 3 seats   │  │ one-time   │                       │  │
│  │  └─────┬─────┘  └─────┬──────┘                       │  │
│  │        └───────┬───────┘                             │  │
│  │                ▼                                     │  │
│  │  ┌─── Coupon (Optional) ──┐                          │  │
│  │  │ Code: [______] [Apply] │                          │  │
│  │  │ → validate_coupon RPC  │                          │  │
│  │  │ → Shows discount       │                          │  │
│  │  └────────────────────────┘                          │  │
│  │                │                                     │  │
│  │                ▼                                     │  │
│  │  ┌─── Razorpay Checkout ─────────────────────────┐   │  │
│  │  │                                               │   │  │
│  │  │  1. POST razorpay-create-order                │   │  │
│  │  │     → Creates Razorpay order (amount in paise)│   │  │
│  │  │     → Inserts subscription_payments (created) │   │  │
│  │  │                                               │   │  │
│  │  │  2. openRazorpay(options)                     │   │  │
│  │  │     → Razorpay modal (Card/UPI/Net Banking)   │   │  │
│  │  │                                               │   │  │
│  │  │  3. On Success:                               │   │  │
│  │  │     POST razorpay-verify-payment              │   │  │
│  │  │     → HMAC-SHA256 signature verify            │   │  │
│  │  │     → extend_or_create_paid_license RPC       │   │  │
│  │  │     → Update payment status → 'paid'          │   │  │
│  │  │     → Record coupon redemption                │   │  │
│  │  │                                               │   │  │
│  │  │  4. Webhook (backup):                         │   │  │
│  │  │     razorpay-webhook (async)                  │   │  │
│  │  │     → Handles payment.captured/failed/refunded│   │  │
│  │  └───────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌─── License Key Redemption ───────────────────────────┐  │
│  │ Key: [XXXX-XXXX-XXXX-XXXX] [Redeem]                  │  │
│  │ → redeem_license(key, company_id) RPC                │  │
│  │ → Extends existing license or creates new one        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘

License Lifecycle:
  ┌──────────┐   signup   ┌───────┐  expires  ┌─────────┐
  │ No       │ ─────────▶ │ Trial │ ────────▶  Expired │
  │ License  │            │ (N d) │           │ (locked)│
  └──────────┘            └───┬───┘           └────┬────┘
                              │ pay                │ pay/redeem
                              ▼                    ▼
                         ┌─────────┐          ┌─────────┐
                         │  Paid   │◀──────── │ Renewed │
                         │ (30d or │  extend  │         │
                         │ lifetime)│         └─────────┘
                         └─────────┘
  
  When license expires → all routes redirect to /billing (LicenseGate)
```

### 7.6 Report Generation & Sharing Flow

```
┌──────────────────────────────────────────────────────────┐
│                    Reports Page                          │
│                                                          │
│  ┌─── Report Selection ───────────────────────────────┐  │
│  │ 14 Report Types:                                   │  │
│  │                                                    │  │
│  │  Sales & Purchases:    Profitability:              │  │
│  │  • Sale Report          • Profit & Loss            │  │
│  │  • Purchase Report      • Party-wise P&L           │  │
│  │  • Day Book             • Item-wise P&L            │  │
│  │  • All Transactions                                │  │
│  │                        Inventory:                  │  │
│  │  GST Returns:          • Stock Summary             │  │
│  │  • GSTR-1              • Stock Detail              │  │
│  │  • GSTR-2                                          │  │
│  │  • GSTR-3B            Party:                       │  │
│  │                        • Party Statement           │  │
│  │                        • All Parties Report        │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌─── Filters ────────────────────────────────────────┐  │
│  │ From: [Date]  To: [Date]  Party: [PartyCombobox]   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌─── Actions ────────────────────────────────────────┐  │
│  │ [Export PDF] → Opens HTML in new tab → window.print│  │
│  │ [Export Excel] → XLSX.writeFile download           │  │
│  │ [Share] → ShareReportDialog (two tabs)             │  │
│  │   Email & link:                                    │  │
│  │           ├─ buildReportArtifact(HTML + XLSX64)    │  │
│  │           ├─ createReportShare() → shared_reports  │  │
│  │           │    row with public_token               │  │
│  │           ├─ [Create view link] → /r/:token, copied│  │
│  │           └─ [Send email] → send-report-link fn    │  │
│  │                (Resend, logs report_share_emails)  │  │
│  │   Portal users:                                    │  │
│  │           ├─ Select party-portal recipients        │  │
│  │           └─ Insert into shared_reports table      │  │
│  │                                                    │  │
│  │ Public link: /r/:token → PublicReport page →        │  │
│  │   public-report edge fn (no sign-in) → view,       │  │
│  │   print/PDF or Excel; view_count incremented       │  │

│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌─── Tabs ───────────────────────────────────────────┐  │
│  │ [Generate] [Sent Reports] [Shared With Me]         │  │
│  │                                                    │  │
│  │ SentReportsTab:                                    │  │
│  │  Shows reports sent by me, view/download status    │  │
│  │                                                    │  │
│  │ SharedReportsTab:                                  │  │
│  │  Shows reports received, auto-marks viewed         │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘

GSTR Export Details:
  GSTR-1 (Sales):
    Sheets: B2B, B2CL (>₹2.5L inter-state), B2CS, CDNR, HSN
  GSTR-2 (Purchases):
    Sheets: B2B, CDNR
  GSTR-3B (Summary):
    Sheets: 3.1 Outward Supplies, 4. ITC, 6.1 Tax Payment
```

### 7.7 Party Portal Flow

```
┌───────────────────────────────────────────────────────────┐
│                   Party Portal                            │
│                                                           │
│  ┌─── Access Model ───────────────────────────────────┐   │
│  │                                                    │   │
│  │  Admin creates user with role='party'              │   │
│  │       ↓                                            │   │
│  │  Admin links user to a party record                │   │
│  │  (party_access table):                             │   │
│  │    • party_user_id → auth user                     │   │
│  │    • party_id → parties.id                         │   │
│  │    • can_view_sales: true/false                    │   │
│  │    • can_view_purchases: true/false                │   │
│  │    • can_view_payments: true/false                 │   │
│  │       ↓                                            │   │
│  │  Party user logs in → isParty flag set             │   │
│  │       ↓                                            │   │
│  │  All routes redirect to /portal                    │   │
│  │  (No license check — they're not paying users)     │   │
│  └────────────────────────────────────────────────────┘   │
│                                                           │
│  ┌─── Portal Tabs ────────────────────────────────────┐   │
│  │ [Ledger] [Sales] [Purchases] [Payments] [Reports]  │   │
│  │                                                    │   │
│  │ Each tab:                                          │   │
│  │  • Filters to linked party only                    │   │
│  │  • Respects can_view_* permissions                 │   │
│  │  • RLS enforces at DB level via party_access       │   │
│  │  • Search, sort, export PDF/Excel                  │   │
│  │  • Shared Reports tab shows received reports       │   │
│  └────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────┘
```

### 7.8 Backup & Restore Flow

```
┌────────────────────────────────────────────────────────┐
│              Backup & Restore Page                     │
│                                                        │
│  ┌─── Export (.bkp) ───────────────────────────────┐   │
│  │  1. Fetch all company data from Supabase:       │   │
│  │     parties, items, invoices, invoice_items,    │   │
│  │     payments, expenses, expense_payments,       │   │
│  │     business_profiles, custom_units, categories,│   │
│  │     activity_log, shared_reports, party_access  │   │
│  │  2. Serialize to JSON                           │   │
│  │  3. Download as .bkp file                       │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  ┌─── Import (.bkp) ───────────────────────────────┐   │
│  │  1. Parse .bkp JSON file                        │   │
│  │  2. Purge existing data (FK-safe order):        │   │
│  │     a. NULL linked_invoice_id on invoices       │   │
│  │     b. Delete: expense_payments → payments →    │   │
│  │        invoice_items → invoices → party_access  │   │
│  │        → expenses → items → parties →           │   │
│  │        activity_log → shared_reports →          │   │
│  │        custom_units → categories →              │   │
│  │        business_profiles                        │   │
│  │  3. Upsert in batches of 500:                   │   │
│  │     Reverse order (business_profiles first,     │   │
│  │     parties, items, expenses, invoices, etc.).  │   │
│  │  4. Verify record counts                        │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  ┌─── Delete All ──────────────────────────────────┐   │
│  │  Purges all company data with confirmation      │   │
│  └─────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

### 7.9 Admin User Management Flow

```
┌───────────────────────────────────────────────────────────┐
│                    Admin Panel                            │
│                                                           │
│  ┌─── Members Tab ───────────────────────────────────┐    │
│  │                                                   │    │
│  │  [+ Add Member]                                   │    │
│  │  ┌──────────────────────────────────────────┐     │    │
│  │  │ Email: user@example.com                  │     │    │
│  │  │ Password: (for new users)                │     │    │
│  │  │ Display Name: John                       │     │    │
│  │  │ Role: [owner|admin|staff|member|party]   │     │    │
│  │  │                                          │     │    │
│  │  │ → Calls create-user Edge Function        │     │    │
│  │  │   • Checks seat limit from license       │     │    │
│  │  │   • Creates auth user (if new)           │     │    │
│  │  │   • Inserts company_members              │     │    │
│  │  └──────────────────────────────────────────┘     │    │
│  │                                                   │    │
│  │  Members List:                                    │    │
│  │  ┌────────────┬──────┬────────────────────────┐   │    │
│  │  │ Name/Email │ Role │ Actions                │   │    │
│  │  ├────────────┼──────┼────────────────────────┤   │    │
│  │  │ Alice      │owner │ (no actions on self)   │   │    │
│  │  │ Bob        │admin │ [Edit] [Permissions]   │   │    │
│  │  │ Carol      │staff │ [Edit] [Permissions]   │   │    │
│  │  │ Dave       │party │ [Link Party] [Remove]  │   │    │
│  │  └────────────┴──────┴────────────────────────┘   │    │
│  │                                                   │    │
│  │  Page Permissions Matrix (per member):            │    │
│  │  ┌──────────┬──────┬────────┬──────┬─────────┐    │    │
│  │  │ Page     │ View │ Create │ Edit │ Delete  │    │    │
│  │  ├──────────┼──────┼────────┼──────┼─────────┤    │    │
│  │  │ Dashboard│  ✓   │   -    │  -   │   -     │    │    │
│  │  │ Sales    │  ✓   │   ✓    │  ✓   │   ✗     │    │    │
│  │  │ Items    │  ✓   │   ✓    │  ✗   │   ✗     │    │    │
│  │  │ ...      │      │        │      │         │    │    │
│  │  └──────────┴──────┴────────┴──────┴─────────┘    │    │
│  └───────────────────────────────────────────────────┘    │
│                                                           │
│  ┌─── Activity Log Tab ─────────────────────────────┐     │
│  │ [2026-04-24 10:30] Alice created Invoice INV-042 │     │
│  │ [2026-04-24 09:15] Bob edited Party "XYZ Corp"   │     │
│  │ [2026-04-23 18:00] Carol deleted Payment PAY-007 │     │
│  └──────────────────────────────────────────────────┘     │
│                                                           │
│  ┌─── Settings Tab ─────────────────────────────────┐     │
│  │ Allow New Signups: [Toggle]                      │     │
│  │ (Stored in app_settings.allow_signups)           │     │
│  └──────────────────────────────────────────────────┘     │
└───────────────────────────────────────────────────────────┘
```

---

### 7.10 Offline-First Sync Architecture

SmartBooks keeps working when the network drops. Every write is described
declaratively, queued in IndexedDB and replayed in order once connectivity
returns. Reads fall back to the last cached snapshot, so the app still renders
invoices, parties, items and settings with no connection at all.

**Modules**

| File | Responsibility |
|------|----------------|
| `src/lib/offline/idb.ts` | Minimal IndexedDB helper. Two stores: `kv` (cached server snapshots) and `outbox` (queued writes). |
| `src/lib/offline/sync.ts` | Sync engine: connectivity state, `offlineWrite()`, durable outbox, `syncNow()`, auto-sync timer and reachability probe. |
| `src/lib/offline/offlineClient.ts` | `offline.from(table)` — a drop-in proxy over `supabase.from()` that queues insert/update/delete when offline. |
| `src/hooks/use-sync-status.ts` | `useSyncStatus()` — live `{ online, pending, syncing, autoSync, lastSyncedAt, lastError }`. |
| `src/components/NetworkStatus.tsx` | Header indicator: green dot online, grey dot offline, pending badge, auto-sync toggle, **Sync now**. |
| `src/contexts/AppContext.tsx` | Hydrates from the IndexedDB snapshot, persists after every change, refreshes when the outbox flushes. |

**Write path (online vs offline)**

```
        ┌──────────────────────────┐
        │  User saves invoice /    │
        │  payment / party / item  │
        └────────────┬─────────────┘
                     │
                     ▼
            ┌────────────────┐
            │ offlineWrite() │
            └───┬────────┬───┘
        online  │        │  offline OR network error
                ▼        ▼
   ┌──────────────────┐  ┌───────────────────────────┐
   │ Supabase / RLS   │  │ IndexedDB `outbox`        │
   │ real insert      │  │ { id, table, op, payload, │
   │                  │  │   match, createdAt }      │
   └────────┬─────────┘  └─────────────┬─────────────┘
            │                          │
            │  success                 │ optimistic success
            ▼                          ▼
   ┌──────────────────────────────────────────────────┐
   │ UI updates immediately + snapshot saved to `kv`  │
   └──────────────────────────────────────────────────┘
```

**Sync loop**

```
 ┌─────────┐  browser 'online' event ┌──────────┐
 │ OFFLINE │ ──────────────────────▶ │  ONLINE  │
 │ grey dot│ ◀────────────────────── │ green dot│
 └─────────┘  'offline' / probe fail └────┬─────┘
      ▲                                   │
      │                    every 15s probe│(SELECT 1 on companies)
      │                                   ▼
      │                        ┌────────────────────┐
      │                        │ autoSync && pending│
      │                        └─────────┬──────────┘
      │                                  │ yes
      │                                  ▼
      │                        ┌────────────────────┐
      │                        │      syncNow()     │
      │                        │ replay outbox in   │
      │                        │ createdAt order    │
      │                        └───┬────────┬───────┘
      │             network error  │        │ permanent error (RLS/validation)
      └────────────────────────────┘        ▼
                                   ┌──────────────────────────┐
                                   │ drop entry + surface     │
                                   │ lastError (queue can't   │
                                   │ wedge)                   │
                                   └───────────┬──────────────┘
                                               │ all done
                                               ▼
                                   ┌──────────────────────────┐
                                   │ lastSyncedAt stamped     │
                                   │ onOutboxFlushed() →      │
                                   │ AppContext full refresh  │
                                   └──────────────────────────┘
```

**Behaviour notes**

- Auto-sync is on by default; it can be turned off from the header panel
  (persisted in `localStorage` as `sb_auto_sync`).
- `navigator.onLine` is trusted only as a hint — a lightweight Supabase query
  confirms real reachability every 15 seconds (captive-portal safe).
- Offline inserts use a client-generated `crypto.randomUUID()` primary key, so
  replayed rows keep the same id the UI already showed.
- Server-dependent features still need a connection: AI invoice parsing,
  outbound email, Razorpay checkout and bulk imports.

---

## 8. Database Schema


### 8.1 Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         auth.users (Supabase)                       │
│                              │ PK: id                               │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
   ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐
   │   profiles   │   │  user_roles  │   │  super_admins    │
   │   (1:1)      │   │  (1:N)       │   │  (email PK)      │
   └──────────────┘   └──────────────┘   └──────────────────┘
          │
          │ user_id
          ▼
   ┌──────────────┐        ┌────────────────────┐
   │  companies   │◄───────│  company_members   │
   │  PK: id      │ 1:N    │  (user_id,         │
   │  owner_id ───┘        │   company_id,      │
   └──────┬───────┘        │   role,            │
          │                │   page_permissions)│
          │                └────────────────────┘
          │
     ┌────┼────┬───────────────┬───────────────┐
     │    │    │               │               │
     ▼    ▼    ▼               ▼               ▼
┌────────┐┌────────┐  ┌──────────────┐ ┌─────────────┐
│business││parties │  │    items     │ │  payments   │
│profiles││        │  │              │ │             │
└────────┘└───┬────┘  └──────────────┘ └─────────────┘
              │                │               │
              │                │               │
              ▼                ▼               │
       ┌──────────────┐ ┌──────────────┐       │
       │ party_access │ │  invoices    │◄──────┘
       │(portal perms)│ │  (sale/      │  linked
       └──────────────┘ │   purchase/  │
                        │   returns)   │
                        └──────┬───────┘
                               │
                               ▼
                        ┌──────────────┐
                        │invoice_items │
                        │(line items)  │
                        └──────────────┘

      ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
      │ custom_units │  │  categories  │  │ activity_log │
      │ (per company)│  │(per company) │  │ (audit trail)│
      └──────────────┘  └──────────────┘  └──────────────┘

      ┌──────────────┐  ┌────────────────┐
      │   expenses   │  │expense_payments│
      │(per company) │  │(per company)   │
      └──────┬───────┘  └────────────────┘
             │
             ▼
      ┌──────────────┐
      │shared_reports│
      │(sender→recip)│
      └──────────────┘

     ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐
     │   licenses   │  │subscription_     │  │coupon_redemptions│
     │ (per company)│  │  payments        │  │                  │
     └──────────────┘  └──────────────────┘  └──────────────────┘
                              │
                              ▼
                       ┌──────────────┐
                       │   coupons    │
                       └──────────────┘

     ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
     │ app_settings │  │staff_perms   │  │page_perms    │
     │ (global KV)  │  │(legacy)      │  │(legacy)      │
     └──────────────┘  └──────────────┘  └──────────────┘
```

### 8.2 Tables Reference

#### `profiles`
User profile data, auto-created on signup.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK, `gen_random_uuid()` |
| `user_id` | `uuid` | NOT NULL, UNIQUE, FK → `auth.users(id)` CASCADE |
| `display_name` | `text` | |
| `avatar_url` | `text` | |
| `phone` | `text` | |
| `email` | `text` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `user_roles`
Global application roles.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `role` | `app_role` | NOT NULL, DEFAULT `'user'` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| | | UNIQUE(`user_id`, `role`) |

#### `companies`
Multi-tenant company records.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `name` | `text` | NOT NULL |
| `owner_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `company_members`
Company membership with roles and permissions.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `role` | `company_role` | NOT NULL, DEFAULT `'member'` |
| `can_edit_payments` | `boolean` | NOT NULL, DEFAULT `false` |
| `page_permissions` | `jsonb` | NOT NULL, DEFAULT `'{}'` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| | | UNIQUE(`company_id`, `user_id`) |

**`page_permissions` JSON structure:**
```json
{
  "dashboard": { "view": true },
  "parties": { "view": true, "create": true, "edit": true, "delete": false },
  "items": { "view": true, "create": true, "edit": false, "delete": false },
  "sales": { "view": true, "create": true, "edit": true, "delete": false },
  "purchases": { "view": true, "create": true, "edit": true, "delete": false },
  "payment_in": { "view": true, "create": true, "edit": false, "delete": false },
  "payment_out": { "view": true, "create": true, "edit": false, "delete": false },
  "sale_return": { "view": true, "create": true },
  "purchase_return": { "view": true, "create": true },
  "reports": { "view": true },
  "settings": { "view": true, "edit": true }
}
```

#### `business_profiles`
Business configuration per company.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, UNIQUE, FK → `companies(id)` CASCADE |
| `name` | `text` | NOT NULL, DEFAULT `'My Business'` |
| `gstin` | `text` | DEFAULT `''` |
| `phone` | `text` | DEFAULT `''` |
| `email` | `text` | DEFAULT `''` |
| `address` | `text` | DEFAULT `''` |
| `state` | `text` | DEFAULT `'Maharashtra'` |
| `upi_id` | `text` | DEFAULT `''` |
| `logo_url` | `text` | |
| `default_invoice_template` | `text` | DEFAULT `'a5'` |
| `invoice_panel_style` | `text` | NOT NULL, DEFAULT `'classic'` |
| `whatsapp_phone` | `text` | |
| `whatsapp_auto_share_sales` | `boolean` | NOT NULL, DEFAULT `false` |
| `whatsapp_auto_share_purchases` | `boolean` | NOT NULL, DEFAULT `false` |
| `whatsapp_auto_share_payments` | `boolean` | NOT NULL, DEFAULT `false` |
| `whatsapp_auto_share_outstanding` | `boolean` | NOT NULL, DEFAULT `false` |
| `invoice_font_family` | `text` | NOT NULL, DEFAULT `''` |
| `invoice_language` | `text` | NOT NULL, DEFAULT `'en'` |
| `multi_currency_enabled` | `boolean` | NOT NULL, DEFAULT `false` |
| `base_currency` | `text` | NOT NULL, DEFAULT `'INR'` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `parties`
Customer and supplier records.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `name` | `text` | NOT NULL |
| `phone` | `text` | DEFAULT `''` |
| `email` | `text` | DEFAULT `''` |
| `gstin` | `text` | DEFAULT `''` |
| `gst_type` | `text` | DEFAULT `'unregistered'` |
| `state` | `text` | DEFAULT `''` |
| `billing_address` | `text` | DEFAULT `''` |
| `shipping_address` | `text` | DEFAULT `''` |
| `type` | `text` | NOT NULL, DEFAULT `'customer'` |
| `opening_balance` | `numeric` | DEFAULT `0` |
| `opening_balance_date` | `text` | DEFAULT `''` |
| `opening_balance_type` | `text` | DEFAULT `'receive'` |
| `credit_limit` | `numeric` | DEFAULT `0` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `items`
Product and service catalog.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `name` | `text` | NOT NULL |
| `item_type` | `text` | DEFAULT `'product'` |
| `item_code` | `text` | DEFAULT `''` |
| `hsn_code` | `text` | DEFAULT `''` |
| `category` | `text` | DEFAULT `''` |
| `primary_unit` | `text` | DEFAULT `'pcs'` |
| `secondary_unit` | `text` | DEFAULT `''` |
| `conversion_rate` | `numeric` | DEFAULT `1` |
| `sale_price` | `numeric` | DEFAULT `0` |
| `sale_price_secondary` | `numeric` | DEFAULT `0` |
| `sale_price_with_tax` | `boolean` | DEFAULT `false` |
| `purchase_price` | `numeric` | DEFAULT `0` |
| `purchase_price_secondary` | `numeric` | DEFAULT `0` |
| `purchase_price_with_tax` | `boolean` | DEFAULT `false` |
| `discount_value` | `numeric` | DEFAULT `0` |
| `discount_type` | `text` | DEFAULT `'percentage'` |
| `gst_rate` | `numeric` | DEFAULT `18` |
| `opening_stock` | `numeric` | DEFAULT `0` |
| `stock_as_of_date` | `text` | DEFAULT `''` |
| `stock_price_per_unit` | `numeric` | DEFAULT `0` |
| `min_stock_qty` | `numeric` | DEFAULT `0` |
| `item_location` | `text` | DEFAULT `''` |
| `stock` | `numeric` | DEFAULT `0` |
| `mrp` | `numeric` | DEFAULT `0` |
| `is_active` | `boolean` | NOT NULL, DEFAULT `true` |
| `image_url` | `text` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `invoices`
All invoice types (sale, purchase, sale_return, purchase_return).

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `invoice_number` | `text` | NOT NULL |
| `date` | `text` | NOT NULL |
| `party_id` | `text` | NOT NULL |
| `party_name` | `text` | NOT NULL |
| `party_state` | `text` | DEFAULT `''` |
| `party_gstin` | `text` | DEFAULT `''` |
| `subtotal` | `numeric` | DEFAULT `0` |
| `cgst` | `numeric` | DEFAULT `0` |
| `sgst` | `numeric` | DEFAULT `0` |
| `igst` | `numeric` | DEFAULT `0` |
| `total` | `numeric` | DEFAULT `0` |
| `notes` | `text` | DEFAULT `''` |
| `type` | `text` | NOT NULL (`sale`, `purchase`, `sale_return`, `purchase_return`) |
| `status` | `text` | DEFAULT `'unpaid'` (`paid`, `unpaid`, `partial`) |
| `amount_paid` | `numeric` | DEFAULT `0` |
| `received_amount` | `numeric` | DEFAULT `0` |
| `linked_invoice_id` | `text` | (for returns → original invoice) |
| `extra_charges` | `jsonb` | NOT NULL, DEFAULT `'[]'` |
| `extra_discounts` | `jsonb` | NOT NULL, DEFAULT `'[]'` |
| `round_off` | `numeric` | NOT NULL, DEFAULT `0` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

**`extra_charges` / `extra_discounts` JSON structure (array of ExtraLine):**
```json
[
  {
    "id": "uuid",
    "label": "Freight",
    "amount": 500,
    "amountType": "flat",
    "taxable": false,
    "gstRate": 0,
    "applyStage": "post"
  }
]
```

#### `invoice_items`
Line items within an invoice.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `invoice_id` | `uuid` | NOT NULL, FK → `invoices(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `item_id` | `text` | NOT NULL |
| `item_name` | `text` | NOT NULL |
| `hsn_code` | `text` | DEFAULT `''` |
| `unit` | `text` | DEFAULT `'pcs'` |
| `qty` | `numeric` | DEFAULT `1` |
| `mrp` | `numeric` | DEFAULT `0` |
| `rate` | `numeric` | DEFAULT `0` |
| `price_with_tax` | `boolean` | DEFAULT `false` |
| `discount` | `numeric` | DEFAULT `0` |
| `discount_type` | `text` | DEFAULT `'percentage'` |
| `gst_rate` | `numeric` | DEFAULT `0` |
| `gst_amount` | `numeric` | DEFAULT `0` |
| `amount` | `numeric` | DEFAULT `0` |

#### `payments`
Payment-in (from customers) and payment-out (to suppliers).

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `date` | `text` | NOT NULL |
| `party_id` | `text` | NOT NULL |
| `party_name` | `text` | NOT NULL |
| `invoice_id` | `text` | (linked invoice) |
| `invoice_number` | `text` | |
| `amount` | `numeric` | DEFAULT `0` |
| `mode` | `text` | DEFAULT `'cash'` (`cash`, `bank`, `upi`, `cheque`) |
| `type` | `text` | NOT NULL (`in`, `out`) |
| `notes` | `text` | DEFAULT `''` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `party_access`
Controls which party users can view what data in the Party Portal.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `party_user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `party_id` | `text` | NOT NULL |
| `can_view_sales` | `boolean` | DEFAULT `false` |
| `can_view_purchases` | `boolean` | DEFAULT `false` |
| `can_view_payments` | `boolean` | DEFAULT `false` |
| `granted_by` | `uuid` | FK → `auth.users(id)` |
| `company_id` | `uuid` | FK → `companies(id)` CASCADE |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| | | UNIQUE(`party_user_id`, `party_id`) |

#### `custom_units`
User-defined measurement units.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `unit_name` | `text` | NOT NULL |
| | | UNIQUE(`company_id`, `unit_name`) |

#### `categories`
Item categories.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `name` | `text` | NOT NULL |
| | | UNIQUE(`company_id`, `name`) |

#### `activity_log`
Audit trail for all create/edit/delete operations.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | FK → `auth.users(id)` SET NULL |
| `user_name` | `text` | |
| `action` | `text` | NOT NULL, CHECK IN (`create`, `edit`, `delete`) |
| `entity_type` | `text` | NOT NULL |
| `entity_id` | `text` | |
| `entity_description` | `text` | |
| `details` | `jsonb` | DEFAULT `'{}'` |
| `company_id` | `uuid` | FK → `companies(id)` CASCADE |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `shared_reports`
Report snapshots shared with portal users and/or via a public view link
(migration `042_report_share_links.sql`).

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `shared_by` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `recipient_user_id` | `uuid` | FK → `auth.users(id)` CASCADE (null for email-only shares) |
| `recipient_email` | `text` | Email recipient of the view link |
| `report_key` | `text` | NOT NULL |
| `report_label` | `text` | NOT NULL |
| `title` | `text` | NOT NULL |
| `from_date` | `text` | NOT NULL |
| `to_date` | `text` | NOT NULL |
| `party_id` | `uuid` | |
| `file_name` | `text` | NOT NULL |
| `html_content` | `text` | NOT NULL |
| `xlsx_base64` | `text` | NOT NULL |
| `message` | `text` | |
| `company_id` | `uuid` | FK → `companies(id)` CASCADE |
| `public_token` | `uuid` | NOT NULL, UNIQUE, DEFAULT `gen_random_uuid()` — slug of `/r/:token` |
| `public_enabled` | `boolean` | NOT NULL, DEFAULT `true` (revoke the link) |
| `expires_at` | `timestamptz` | Optional link expiry |
| `view_count` | `integer` | NOT NULL, DEFAULT 0 |
| `last_viewed_at` | `timestamptz` | |
| `last_sent_at` | `timestamptz` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `viewed_at` | `timestamptz` | |
| `downloaded_at` | `timestamptz` | |

#### `report_share_emails`
Delivery log for report view links emailed through the `send-report-link` function.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `report_id` | `uuid` | FK → `shared_reports(id)` CASCADE |
| `user_id` | `uuid` | FK → `auth.users(id)` SET NULL |
| `to_email` | `text` | NOT NULL |
| `message` | `text` | |
| `status` | `text` | NOT NULL, DEFAULT `'sent'` (`sent` \| `failed`) |
| `error` | `text` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |



#### `expenses`
Business expenses with GST, vendor linking, payment tracking, and recurrence.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK, `gen_random_uuid()` |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `date` | `date` | NOT NULL, DEFAULT `CURRENT_DATE` |
| `category` | `text` | NOT NULL, DEFAULT `'Miscellaneous'` |
| `vendor_id` | `uuid` | FK → `parties(id)` SET NULL |
| `vendor_name` | `text` | |
| `description` | `text` | NOT NULL, DEFAULT `''` |
| `amount` | `numeric(14,2)` | NOT NULL, DEFAULT `0` |
| `gst_rate` | `numeric(5,2)` | NOT NULL, DEFAULT `0` |
| `gst_amount` | `numeric(14,2)` | NOT NULL, DEFAULT `0` |
| `is_inter_state` | `boolean` | NOT NULL, DEFAULT `false` |
| `total` | `numeric(14,2)` | NOT NULL, DEFAULT `0` |
| `itc_eligible` | `boolean` | NOT NULL, DEFAULT `true` |
| `payment_status` | `text` | NOT NULL, DEFAULT `'unpaid'` (`paid`\|`unpaid`\|`partial`) |
| `attachment_data_url` | `text` | |
| `notes` | `text` | |
| `recurrence` | `text` | NOT NULL, DEFAULT `'none'` (`none`\|`daily`\|`weekly`\|`monthly`\|`quarterly`\|`yearly`) |
| `next_due_date` | `date` | |
| `recurring_parent_id` | `uuid` | FK → `expenses(id)` SET NULL |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `expense_payments`
Payments made against an expense.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK, `gen_random_uuid()` |
| `expense_id` | `uuid` | NOT NULL, FK → `expenses(id)` CASCADE |
| `company_id` | `uuid` | NOT NULL, FK → `companies(id)` CASCADE |
| `date` | `date` | NOT NULL, DEFAULT `CURRENT_DATE` |
| `amount` | `numeric(14,2)` | NOT NULL, DEFAULT `0` |
| `mode` | `text` | NOT NULL, DEFAULT `'cash'` (`cash`\|`bank`\|`upi`\|`cheque`\|`card`) |
| `notes` | `text` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `page_permissions` (Legacy)
Per-user page-level access control. Superseded by `company_members.page_permissions` JSON.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `page_key` | `text` | NOT NULL |
| `can_view` | `boolean` | NOT NULL, DEFAULT `false` |
| `can_create` | `boolean` | NOT NULL, DEFAULT `false` |
| `can_edit` | `boolean` | NOT NULL, DEFAULT `false` |
| `can_delete` | `boolean` | NOT NULL, DEFAULT `false` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| | | UNIQUE(`user_id`, `page_key`) |

#### `staff_permissions` (Legacy)
Staff payment edit permissions. Superseded by `company_members.can_edit_payments`.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, UNIQUE, FK → `auth.users(id)` CASCADE |
| `can_edit_payments` | `boolean` | NOT NULL, DEFAULT `false` |
| `granted_by` | `uuid` | FK → `auth.users(id)` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `app_settings`
Global key-value application settings.

| Column | Type | Constraints |
|--------|------|-------------|
| `setting_key` | `text` | PK |
| `setting_value` | `text` | NOT NULL |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

**Seeded values:**
| Key | Default Value | Description |
|-----|--------------|-------------|
| `allow_signups` | `'true'` | Whether new user registration is enabled |
| `trial_days` | `'7'` | Default trial license duration in days |

#### `super_admins`
Platform super-administrators.

| Column | Type | Constraints |
|--------|------|-------------|
| `email` | `text` | PK |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

**Seeded:** `admin@admin.com`

#### `licenses`
Per-company license records.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `company_id` | `uuid` | FK → `companies(id)` CASCADE (nullable) |
| `license_key` | `text` | UNIQUE |
| `type` | `license_type` | NOT NULL, DEFAULT `'trial'` |
| `status` | `license_status` | NOT NULL, DEFAULT `'active'` |
| `seats` | `integer` | NOT NULL, DEFAULT `3` |
| `starts_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `expires_at` | `timestamptz` | NOT NULL |
| `notes` | `text` | |
| `issued_by` | `uuid` | FK → `auth.users(id)` SET NULL |
| `issued_to_email` | `text` | |
| `redeemed_at` | `timestamptz` | |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `coupons`
Discount coupons for subscription payments.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `code` | `text` | UNIQUE, NOT NULL |
| `type` | `coupon_type` | NOT NULL (`percent`, `flat`) |
| `value` | `numeric(10,2)` | NOT NULL, CHECK `> 0` |
| `applies_to` | `coupon_applies_to` | NOT NULL, DEFAULT `'both'` |
| `max_uses` | `integer` | (NULL = unlimited) |
| `used_count` | `integer` | NOT NULL, DEFAULT `0` |
| `expires_at` | `timestamptz` | |
| `active` | `boolean` | NOT NULL, DEFAULT `true` |
| `notes` | `text` | |
| `created_by` | `uuid` | FK → `auth.users(id)` SET NULL |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |

#### `subscription_payments`
Razorpay payment records for license purchases.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `company_id` | `uuid` | FK → `companies(id)` SET NULL |
| `plan` | `plan_code` | NOT NULL (`monthly`, `lifetime`) |
| `amount` | `numeric(10,2)` | NOT NULL |
| `base_amount` | `numeric(10,2)` | NOT NULL |
| `discount_amount` | `numeric(10,2)` | NOT NULL, DEFAULT `0` |
| `coupon_id` | `uuid` | FK → `coupons(id)` SET NULL |
| `coupon_code` | `text` | |
| `razorpay_order_id` | `text` | UNIQUE |
| `razorpay_payment_id` | `text` | |
| `razorpay_signature` | `text` | |
| `status` | `payment_status` | NOT NULL, DEFAULT `'created'` |
| `license_id` | `uuid` | FK → `licenses(id)` SET NULL |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `paid_at` | `timestamptz` | |

#### `coupon_redemptions`
Tracks which users have used which coupons.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `coupon_id` | `uuid` | NOT NULL, FK → `coupons(id)` CASCADE |
| `user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `payment_id` | `uuid` | FK → `subscription_payments(id)` SET NULL |
| `redeemed_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| | | UNIQUE(`coupon_id`, `user_id`) |

#### Module tables (024 → 042)

These tables are all company-scoped (`company_id` + RLS via `is_company_member` /
`can_write_company`) unless noted.

| Table | Migration | Purpose | Notable columns |
|-------|-----------|---------|-----------------|
| `invoice_templates` | 024 | Saved custom invoice-designer layouts | `template_id`, `design` jsonb |
| `loyalty_transactions` | 028 | Signed points ledger | `party_id`, `invoice_id`, `points`, `kind` (earn/redeem/adjust/bonus/expire) |
| `stock_adjustments` | 029 | Manual stock corrections | `item_id`, `qty` (signed), `reason`, `note` |
| `documents` | 036 | Estimates, sale orders, delivery challans, purchase orders | `type` (`document_type`), `items` jsonb, `converted_invoice_id` |
| `recurring_invoices` | 036 | Recurring invoice schedules | `frequency`, `next_run_date`, `history` jsonb |
| `restaurant_areas` | 037 | Seating zones | `name`, `sort_order` |
| `restaurant_tables` | 037 | Tables and live status | `area_id`, `capacity`, `status`, `current_invoice_id` |
| `kots` | 037 | Kitchen order tickets | `table_id`, `items` jsonb, `status`, `order_type` |
| `reservations` | 037 | Table bookings | `guest_name`, `phone`, `reserved_at`, `covers`, `status` |
| `store_settings` | 026, 027, 030 | Storefront config per company | unique `slug`, `enabled`, logo/banner/theme, `allow_orders`, `min_order_value` |
| `store_items` | 026, 027 | Catalogue visibility & overrides | `item_id`, `store_price`, `image_url`, `featured`, `sort_order` |
| `store_customers` | 026 | Public shopper profiles (1:1 with `auth.users`) | created by signup trigger |
| `store_orders` | 026 | Storefront orders | `status` (`store_order_status`), `invoice_id`, totals |
| `store_order_items` | 026 | Order lines | `order_id`, `item_id`, `qty`, `rate` |
| `webhook_endpoints` | 038 | Outbound webhook registrations (REST API) | `url`, `secret`, `events` |
| `webhook_deliveries` | 038 | Delivery attempt log | `endpoint_id`, `event`, `status_code`, `error` |
| `doc_shares` | 039, 040 | Public invoice/payment link control (row UUID is the slug) | `kind`, `doc_id`, `revoked`, `expires_at`, `password_hash`, `view_count`, `last_viewed_at`, `last_sent_at` |
| `doc_share_emails` | 040 | Invoice/payment share email log | `to_email`, `sent_at`, `status` |
| `report_share_emails` | 042 | Report share email log | `report_id`, `to_email`, `sent_at` |
| `ai_providers` | 041 | AI provider credentials for the invoice reader | `provider`, `model`, `api_key`, `base_url`, `priority`, `last_used_at`, `last_error` — **no `anon`/`authenticated` grants**; reachable only via service role and the `sa_ai_provider_*` RPCs |

### 8.3 Enum Types

| Enum | Values | Usage |
|------|--------|-------|
| `app_role` | `admin`, `staff`, `user`, `party` | Global (legacy) user roles, still read by `has_role()` |
| `company_role` | `owner`, `admin`, `staff`, `member`, `party` | Per-company roles — the live RBAC model |
| `license_type` | `trial`, `paid`, `complimentary` | License classification |
| `license_status` | `active`, `expired`, `revoked` | License state |
| `plan_code` | `monthly`, `lifetime`, `year1`, `year3` | Subscription plan (`year1`/`year3` added in 031) |
| `coupon_type` | `percent`, `flat` | Discount calculation mode |
| `coupon_applies_to` | `monthly`, `lifetime`, `both`, `year1`, `year3` | Plan-specific coupons |
| `payment_status` | `created`, `paid`, `failed`, `refunded` | Razorpay payment state |
| `store_order_status` | `pending`, `accepted`, `rejected`, `fulfilled`, `cancelled` | Online store orders |
| `document_type` | `estimate`, `sale_order`, `delivery_challan`, `purchase_order` | Non-invoice documents |

### 8.4 Views

#### `company_active_license`
Returns the most recently expiring active license per company.

```sql
SELECT company_id, id, type, status, seats, starts_at, expires_at, license_key, notes
FROM licenses
WHERE status = 'active'
ORDER BY expires_at DESC
LIMIT 1 per company (DISTINCT ON company_id)
```

Granted `SELECT` to `authenticated` role.

### 8.5 Database Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `has_role` | `(uuid, app_role) → boolean` | SECURITY DEFINER. Checks global `user_roles` table |
| `get_user_role` | `(uuid) → app_role` | Returns highest-priority app role |
| `is_company_member` | `(company_id uuid, user_id uuid) → boolean` | SECURITY DEFINER. Checks `company_members` |
| `has_company_role` | `(company_id uuid, user_id uuid, roles company_role[]) → boolean` | SECURITY DEFINER. Checks for specific roles |
| `is_company_admin` | `(company_id uuid, user_id uuid) → boolean` | SECURITY DEFINER. Checks for owner/admin |
| `can_write_company` | `(company_id uuid, user_id uuid) → boolean` | SECURITY DEFINER. Owner/admin/staff/member (not party) |
| `is_company_party` | `(company_id uuid, user_id uuid) → boolean` | SECURITY DEFINER. Checks party role |
| `is_super_admin` | `() → boolean` | SECURITY DEFINER. JWT email in `super_admins` |
| `handle_new_user` | `() → trigger` | Creates profile row on signup |
| `handle_new_user_role` | `() → trigger` | First user → admin, rest → user role |
| `handle_new_user_company` | `() → trigger` | Creates "Default Company" + membership + profile |
| `update_updated_at_column` | `() → trigger` | Sets `updated_at = now()` on UPDATE |
| `create_trial_license` | `() → trigger` | Creates trial license when company is created |
| `redeem_license` | `(text, uuid) → licenses` | Redeems a license key for a company |
| `extend_or_create_paid_license` | `(uuid, int, text, text, text, boolean) → licenses` | Extends or creates paid license |
| `validate_coupon` | `(text, plan_code, numeric) → jsonb` | Validates coupon and returns discount. Executable by `anon` |
| `record_coupon_use` | `(uuid) → void` | Idempotent coupon usage counter (service role only) — migration 033 |
| `company_license_active` | `(uuid) → boolean` | Gates the public storefront on a live licence — migration 026 |
| `store_public_catalog` | `(text slug) → jsonb` | Anon RPC returning storefront branding + catalogue — 026, replaced in 030 |
| `store_place_order` | `(...) → jsonb` | Anon/authenticated RPC: validates the cart, prices server-side, writes `store_orders` |
| `store_my_orders` | `() → jsonb` | Current shopper's orders with invoice/payment status |
| `store_slug_available` | `(text, uuid) → boolean` | Storefront slug uniqueness check |
| `handle_new_store_customer` | `() → trigger` | Creates a `store_customers` row when signup metadata marks a shopper |
| `lookup_loyalty_card` | `(phone, card_number) → jsonb` | Anon RPC behind the public `/cards` portal — 032, replaced in 034 (adds card design) |
| `sa_business_overview` / `sa_business_breakdown` / `sa_usage_by_table` / `sa_storage_usage` / `sa_db_stats` | `(…) → jsonb` | Super Admin console metrics — migration 035 |
| `sa_users` / `sa_grant_access` / `sa_revoke_access` / `sa_delete_business` | `(…)` | Super Admin cross-tenant user and business management (cascading delete across every `company_id` table + storage objects) |
| `sa_ai_providers` / `sa_ai_provider_save` / `sa_ai_provider_delete` | `(…)` | Masked-key CRUD over `ai_providers`, super admin only — migration 041 |
| `sa_assert_super_admin` | `() → void` | Guard used by every `sa_*` function |

### 8.6 Triggers

| Trigger | Table | Event | Function |
|---------|-------|-------|----------|
| `on_auth_user_created` | `auth.users` | AFTER INSERT | `handle_new_user()` |
| `on_auth_user_created_role` | `auth.users` | AFTER INSERT | `handle_new_user_role()` |
| `on_auth_user_created_company` | `auth.users` | AFTER INSERT | `handle_new_user_company()` |
| `update_profiles_updated_at` | `profiles` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_party_access_updated_at` | `party_access` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_staff_permissions_updated_at` | `staff_permissions` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_business_profiles_updated_at` | `business_profiles` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_parties_updated_at` | `parties` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_items_updated_at` | `items` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_invoices_updated_at` | `invoices` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_payments_updated_at` | `payments` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_expenses_updated_at` | `expenses` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_companies_updated_at` | `companies` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_company_members_updated_at` | `company_members` | BEFORE UPDATE | `update_updated_at_column()` |
| `trg_create_trial_license` | `companies` | AFTER INSERT | `create_trial_license()` |
| `on_auth_user_created_store_customer` | `auth.users` | AFTER INSERT | `handle_new_store_customer()` |

### 8.7 Indexes

| Index | Table | Column(s) |
|-------|-------|-----------|
| `idx_parties_user_id` | parties | `user_id` |
| `idx_parties_type` | parties | `type` |
| `idx_items_user_id` | items | `user_id` |
| `idx_invoices_user_id` | invoices | `user_id` |
| `idx_invoices_party_id` | invoices | `party_id` |
| `idx_invoices_type` | invoices | `type` |
| `idx_invoice_items_invoice_id` | invoice_items | `invoice_id` |
| `idx_payments_user_id` | payments | `user_id` |
| `idx_payments_party_id` | payments | `party_id` |
| `items_is_active_idx` | items | `is_active` |
| `shared_reports_recipient_idx` | shared_reports | `(recipient_user_id, created_at DESC)` |
| `shared_reports_shared_by_idx` | shared_reports | `(shared_by, created_at DESC)` |
| `companies_owner_id_idx` | companies | `owner_id` |
| `company_members_user_id_idx` | company_members | `user_id` |
| `company_members_company_id_idx` | company_members | `company_id` |
| `business_profiles_company_id_idx` | business_profiles | `company_id` |
| `parties_company_id_idx` | parties | `company_id` |
| `items_company_id_idx` | items | `company_id` |
| `invoices_company_id_idx` | invoices | `company_id` |
| `invoice_items_company_id_idx` | invoice_items | `company_id` |
| `payments_company_id_idx` | payments | `company_id` |
| `custom_units_company_id_idx` | custom_units | `company_id` |
| `categories_company_id_idx` | categories | `company_id` |
| `activity_log_company_id_idx` | activity_log | `company_id` |
| `shared_reports_company_id_idx` | shared_reports | `company_id` |
| `party_access_company_id_idx` | party_access | `company_id` |
| `idx_licenses_company` | licenses | `company_id` |
| `idx_licenses_status` | licenses | `status` |
| `idx_licenses_key` | licenses | `license_key` |
| `idx_coupons_code` | coupons | `lower(code)` |
| `idx_subscription_payments_user` | subscription_payments | `user_id` |
| `idx_subscription_payments_order` | subscription_payments | `razorpay_order_id` |
| `expenses_company_id_idx` | expenses | `company_id` |
| `expenses_date_idx` | expenses | `date` |
| `expenses_vendor_id_idx` | expenses | `vendor_id` |
| `expense_payments_expense_id_idx` | expense_payments | `expense_id` |
| `expense_payments_company_id_idx` | expense_payments | `company_id` |

### 8.8 Row-Level Security (RLS) Policies

All tables have RLS enabled. Policies use `SECURITY DEFINER` helper functions for role checks.

#### Core Business Tables (parties, items, invoices, invoice_items, payments, expenses, expense_payments, custom_units, categories)

| Operation | Who | Logic |
|-----------|-----|-------|
| **SELECT** | Company members | `is_company_member(company_id, auth.uid())` |
| **INSERT** | Writers | `can_write_company(company_id, auth.uid())` — owner/admin/staff/member |
| **UPDATE** | Writers | `can_write_company(company_id, auth.uid())` |
| **DELETE** | Admins only | `is_company_admin(company_id, auth.uid())` |

#### Party Portal Access (via `party_access` JOIN)

| Table | Operation | Logic |
|-------|-----------|-------|
| `invoices` | SELECT | Party user can view invoices linked to their party (sales if `can_view_sales`, purchases if `can_view_purchases`) |
| `invoice_items` | SELECT | Via invoice JOIN |
| `payments` | SELECT | Party user can view payments if `can_view_payments` |

#### Licensing & Billing Tables

| Table | Read | Write |
|-------|------|-------|
| `licenses` | Company members + super admins | Super admins only |
| `coupons` | All authenticated users | Super admins only |
| `subscription_payments` | Own user + super admins | (Created via edge functions) |
| `coupon_redemptions` | Own user + super admins | (Created via edge functions) |

#### Admin Tables

| Table | Policy |
|-------|--------|
| `super_admins` | SELECT: `lower(email) = jwt_email` (self-check only) |
| `app_settings` | SELECT: public read; WRITE: `has_role('admin')` |
| `companies` | SELECT: members + super admins; INSERT: own; UPDATE: admin/owner; DELETE: owner |

### 8.9 Storage Buckets

| Bucket | Public | Purpose |
|--------|--------|---------|
| `shared-pdfs` | Yes | WhatsApp-shareable invoice/report PDFs |

**Policies:**
- Public read: `bucket_id = 'shared-pdfs'`
- Owner upload: `bucket_id = 'shared-pdfs' AND auth.uid() = folder owner`

---

## 9. Database Migration History

| # | Date | Migration | Description |
|---|------|-----------|-------------|
| 001 | 2026-04-04 | Initial schema | `profiles`, `user_roles`, `party_access`, `staff_permissions`, `activity_log` + `app_role` enum + role functions + RLS + triggers |
| 002 | 2026-04-04 | Auto-admin role | `handle_new_user_role()` — first user → admin, rest → user |
| 003 | 2026-04-06 | Business schema | `business_profiles`, `parties`, `items`, `invoices`, `invoice_items`, `payments`, `custom_units`, `categories`, `page_permissions`, `app_settings` + party portal RLS |
| 004 | 2026-04-07 | Schema refinement | Re-creates business tables with `updated_at`, granular RLS, indexes |
| 005 | 2026-04-10 | MRP column | Adds `mrp` to `items` |
| 006 | 2026-04-12 | (Empty) | Intentionally blank migration |
| 007 | 2026-04-16 | Logo URL | Adds `logo_url` to `business_profiles` |
| 008 | 2026-04-17 | Admin policies | Admin DELETE on `invoices` and `payments` |
| 009 | — | Fix migration | `business_profiles` unique, `default_invoice_template`, idempotent DDL fixes |
| 010 | — | Extra charges | `extra_charges`, `extra_discounts`, `round_off` on invoices; WhatsApp settings; `shared-pdfs` bucket |
| 011 | — | Active flag | `is_active` boolean on `items` with index |
| 012 | — | Shared reports | `shared_reports` table with sender/recipient RLS |
| 013 | — | Multi-company | `companies`, `company_members`, `company_role` enum, security helpers, `company_id` on all tables, backfill |
| 014 | — | RLS rewrite | All old RLS replaced with company-membership-based policies |
| 015 | — | Backup columns | `invoice_panel_style`, `image_url` |
| 016 | — | Licensing | `super_admins`, `licenses`, enums, `company_active_license` view, `redeem_license()`, trial trigger |
| 017 | — | Razorpay + Coupons | `coupons`, `subscription_payments`, `coupon_redemptions`, `validate_coupon()` |
| 018 | — | Nullable company | `licenses.company_id` nullable for pre-assigned keys |
| 019 | — | Super admin view | SELECT policy on `companies` for super admins |
| 020 | — | License extension | `extend_or_create_paid_license()`, rewritten `redeem_license()` |
| 021 | — | Unique key fix | Clears `license_key` before extending to avoid constraint violation |
| 022 | — | Key/email propagate | Updated extension function to prefer newest key/email |
| 023 | — | Expenses + profile settings | `expenses` + `expense_payments` tables; `invoice_font_family`, `invoice_language`, `multi_currency_enabled`, `base_currency` on `business_profiles` |
| 024 | — | Invoice templates | `invoice_templates` (saved designer layouts, `design` jsonb) + RLS |
| 025 | — | Custom fields + industry packs | `custom_fields` jsonb on `items`/`parties`/`invoices`/`invoice_items`; `industry_settings`, `custom_field_defs`, `signature_url` on `business_profiles`; GIN indexes |
| 026 | — | Online store | `store_settings`, `store_items`, `store_customers` (+ signup trigger), `store_orders`, `store_order_items`, `store_order_status` enum, `company_license_active()`, `store_public_catalog/place_order/my_orders/slug_available()` |
| 027 | — | Store branding + alerts | Storefront branding/customisation columns; business `tagline`/`website`/brand colours; `items.low_stock_alert`; `parties.payment_alert_enabled`/`payment_due_days`; explicit Data-API grants |
| 028 | — | POS, e-way, loyalty | `pos_settings`/`eway_settings`/`loyalty_settings` on `business_profiles`; loyalty + e-way + `pos_sale`/`payment_mode` columns on `parties`/`invoices`; `loyalty_transactions` table |
| 029 | — | Stock adjustments | `stock_adjustments` table (signed qty, reason, note) + RLS |
| 030 | — | Storefront catalogue rewrite | Replaces `store_public_catalog()` to expose 027 branding fields and hide out-of-stock items |
| 031 | — | 1-year / 3-year plans | Adds `year1`, `year3` to `plan_code` and `coupon_applies_to` |
| 032 | — | Loyalty card portal | `lookup_loyalty_card()` anon RPC for the public `/cards` portal |
| 033 | — | Coupon usage tracking | `record_coupon_use()` (idempotent) + backfill of historical redemptions |
| 034 | — | Card portal design | Replaces `lookup_loyalty_card()` to return card design + member phone |
| 035 | — | Super Admin console | `sa_*` RPCs: usage/storage/DB stats, business overview & breakdown, user listing, access grant/revoke, cascading `sa_delete_business()` |
| 036 | — | Documents + recurring | `document_type` enum, `documents` (estimate/sale order/delivery challan/purchase order), `recurring_invoices` |
| 037 | — | Restaurant module | `restaurant_settings` on `business_profiles`; `restaurant_areas`, `restaurant_tables`, `kots`, `reservations` with generated RLS |
| 038 | — | API webhooks | `webhook_endpoints`, `webhook_deliveries` for the REST API's outbound events |
| 039 | — | Doc share links | `doc_shares` — revoke/expiry control for public invoice & payment links |
| 040 | — | Share centre | `doc_shares` password hash, view counts and timestamps; `doc_share_emails` log |
| 041 | — | AI providers | `ai_providers` (service-role only) + masked-key `sa_ai_provider_*` RPCs |
| 042 | — | Report share links | `shared_reports` gains `public_token`, `public_enabled`, `recipient_email`, view stats, `expires_at`; `recipient_user_id` nullable; `report_share_emails` log |

> Run order matters. The scripts are idempotent where practical, but 013 must complete before 014,
> and 040/042 assume 039/012 have run. See `supabase/manual-migrations/README.md` for the
> multi-tenancy run book.

---

## 10. Frontend Architecture

### 10.1 Context Provider Hierarchy

```
<QueryClientProvider>
  <TooltipProvider>
    <BrowserRouter>
      <AuthProvider>           ← Supabase Auth, user state, profile
        <CompanyProvider>      ← Multi-company memberships, roles, permissions
          <LicenseProvider>    ← License state, super admin check
            <AppRoutes />      ← Route definitions + guards
              <AppProvider>    ← All business data CRUD (scoped to active company)
                <ExpenseProvider> ← Expense CRUD (DB-backed, per-company)
                  <AppLayout>    ← Sidebar, header, content area
                    {page}
                  </AppLayout>
                </ExpenseProvider>
              </AppProvider>
          </LicenseProvider>
        </CompanyProvider>
      </AuthProvider>
    </BrowserRouter>
  </TooltipProvider>
</QueryClientProvider>
```

### 10.2 Routing & Page Guards

All routing lives in `AppRoutes()` in `src/App.tsx`, which branches by user type before mounting `<Routes>`.

**Always public (no session needed, mounted outside the auth branch):**
| Path | Page | Purpose |
|------|------|---------|
| `/d/i/:id` | PublicDoc | Shared invoice view (expiry, revocation, optional password) |
| `/d/p/:id` | PublicDoc | Shared payment receipt view |
| `/r/:token` | PublicReport | Shared report snapshot (print/PDF + Excel) |

**Signed out:**
| Path | Page |
|------|------|
| `/` | Landing |
| `/auth` | Auth (single email-or-phone field, signup toggle) |
| `/reset-password` | ResetPassword |
| `/store/:slug` | StoreFront (public storefront) |
| `/cards` | CardsPortal (public loyalty card lookup) |
| `*` | → `/auth?redirect=…` |

**Store customers (signed in, no company membership):** `/store/:slug`, `/cards`, `/my-orders` (MyOnlineOrders), `/reset-password`, `*` → `/my-orders`.

**Party portal users (`company_role = party`, no licence check):** `/portal` (PartyPortal), `/store/:slug`, `/cards`, `/my-orders`, `*` → `/portal`.

**Full app (signed in, licence-gated by `LicenseGate`):**
| Path | Page | Permission key |
|------|------|----------------|
| `/` | Dashboard | `dashboard` |
| `/parties`, `/party-ledger/:id` | Parties, PartyLedger | `parties` |
| `/items`, `/item-history/:id` | Items, ItemHistory | `items` |
| `/sales`, `/sales/new`, `/sales/edit/:id` | Sales, CreateInvoice, EditInvoice | `sales` |
| `/purchases` | Purchases | `purchases` |
| `/pos` | Pos | `sales` |
| `/loyalty` | Loyalty | `parties` |
| `/payment-in`, `/payment-out` | PaymentIn, PaymentOut | `payment_in`, `payment_out` |
| `/sale-return`, `/purchase-return` | SaleReturn, PurchaseReturn | `sale_return`, `purchase_return` |
| `/estimates`, `/sale-orders`, `/delivery-challans` | DocumentsPage | `sales` |
| `/purchase-orders` | DocumentsPage | `purchases` |
| `/documents/:docType/new`, `/documents/:docType/edit/:id` | DocumentEditor | `sales` |
| `/recurring-invoices`, `/recurring-invoices/new`, `/recurring-invoices/edit/:id` | RecurringInvoices, RecurringEditor | `sales` |
| `/tables`, `/kot`, `/reservations` | RestaurantTables, KitchenDisplay, Reservations | `sales` |
| `/expenses` | Expenses | `payment_out` |
| `/reports` | Reports | `reports` |
| `/shared-links` | SharedLinks | `sales` |
| `/settings`, `/settings/eway`, `/settings/pos` | SettingsPage, EwaySettings, PosSettings | `settings` |
| `/backup` | BackupRestore | `settings` |
| `/store` | OnlineStore | — |
| `/admin` | AdminPanel | self-gated by role |
| `/billing` | Billing | exempt from LicenseGate |
| `/portal` | PartyPortal | any signed-in user with party access |
| `*` | NotFound | — |

**LicenseGate:** when a company's licence is missing or expired, every route except `/billing` redirects to the billing page.

**Super Admin console:** not part of this router. `super.html` is a second Vite entry that mounts `src/super/SuperConsole.tsx`, which checks membership of the `super_admins` table before rendering Overview, Businesses, Users, Licences and AI providers.

### 10.3 Pages

| Page | Purpose | Key Features |
|------|---------|-------------|
| **Dashboard** | Business overview | Sales/purchase/payment totals, outstanding, global search, recent transactions, quick actions |
| **Parties** | Customer/supplier CRUD | Search, filter by type, bulk operations, import/export Excel/CSV, GSTIN validation |
| **Items** | Product/service inventory | Category management, dual units, stock tracking, inactive toggle, bulk ops, import/export |
| **Sales** | Sale invoice list | Status filter, date range, sortable columns, PDF preview, bulk ops, import/export |
| **Purchases** | Purchase invoice list | Same as Sales for purchase type |
| **CreateInvoice** | Multi-tab invoice creator | Party search + quick-add, item picker + voice input + quick-add, reverse calculations, extra charges, GST split, payment recording |
| **EditInvoice** | Edit existing invoice | Same as CreateInvoice, loads from existing data |
| **PaymentIn** | Customer payment recording | Auto-fill from outstanding, link to invoice, receipt preview |
| **PaymentOut** | Supplier payment recording | Same as PaymentIn for outgoing payments |
| **SaleReturn** | Credit note list | Create/list/preview sale returns |
| **PurchaseReturn** | Debit note list | Create/list/preview purchase returns |
| **Reports** | 15 report types | PDF/Excel export, share to portal, GSTR-1/2/3B, Expense Report |
| **Expenses** | Expense tracking | Category-wise expenses, GST/ITC, vendor linking, payment tracking, recurrence |
| **PartyLedger** | Per-party ledger | Running balance, debit/credit entries, export |
| **ItemHistory** | Per-item transactions | Stock movements, sale/purchase totals, export |
| **SettingsPage** | Business configuration | Logo, GSTIN, state, invoice template, UPI ID, invoice language, invoice font, multi-currency |
| **AdminPanel** | Team management | Member CRUD, roles, page permissions, activity log, app settings |
| **Billing** | License management | Plans, Razorpay payment, coupon apply, key redemption |
| **Super console** (`src/super/*`) | Cross-tenant admin (separate `super.html` build) | Businesses, users, licences/keys, coupons, DB & storage usage, AI provider keys |
| **Pos / PosSettings** | Touch billing | Item tiles, keypad, barcode scan, tender modes, thermal receipts |
| **DocumentsPage / DocumentEditor** | Estimates, sale & purchase orders, delivery challans | Same line grid as invoices, convert to invoice |
| **RecurringInvoices / RecurringEditor** | Recurring schedules | Frequency, next run, auto-generation history |
| **RestaurantTables / KitchenDisplay / Reservations** | Restaurant floor | Table status, KOTs, bookings, table-to-bill |
| **Loyalty / CardsPortal** | Rewards | Points rules, tiers, redemption, printable cards, public lookup |
| **OnlineStore / StoreFront / MyOnlineOrders** | Online selling | Catalogue, storefront branding, orders, order-to-invoice |
| **SharedLinks** | Share centre | All public invoice/payment links with status, views, expiry, password, email |
| **PublicDoc / PublicReport** | Sign-in-free views | Shared invoice, receipt and report pages |
| **EwaySettings** | E-way bill defaults | Transporter defaults and auto-open threshold |
| **ResetPassword** | Password reset | Resend-delivered reset flow |
| **BackupRestore** | Data backup/restore | Export/import .bkp files, delete all |
| **PartyPortal** | External party portal | Filtered view of invoices, payments, shared reports |
| **Auth** | Login/signup | Email/phone login, signup with toggle |
| **Landing** | Marketing page | Hero, features, pricing (₹0/₹199/₹9,999), CTA |

### 10.4 Components

| Component | Purpose |
|-----------|---------|
| **AppLayout** | Shell: sidebar + header (help, quick create, user menu) + content |
| **AppSidebar** | Navigation: permission-filtered menu, company switcher, trial badge |
| **BulkActionBar** | Sticky bar for bulk edit/delete when rows selected |
| **CompanySwitcher** | Dropdown to switch/create companies |
| **CouponsAdmin** | Coupon CRUD (embedded in LicensesAdmin) |
| **ExtraChargesEditor** | Invoice extra charges/discounts tabs |
| **InvoicePreviewDialog** | Multi-format invoice PDF preview + print |
| **ItemPickerDialog** | Multi-item selection with qty for invoices |
| **NavLink** | Active-aware navigation link wrapper |
| **PartyCombobox** | Searchable party dropdown (name + phone) |
| **PaymentPreviewDialog** | Payment receipt preview + print |
| **QuickAddItemDialog** | Full item creation dialog from within invoice |
| **QuickAddPartyDialog** | Quick party creation from within invoice/payment |
| **SentReportsTab** | Reports sent by current user |
| **SharedReportsTab** | Reports received by current user |
| **ShareReportDialog** | Multi-recipient report sharing dialog |
| **SortHeader** | Sortable column header with `useSort` hook |
| **StatCard** | Animated metric card with icon and variant colors |
| **UserMenu** | User avatar dropdown with role display and sign out |
| **LineItemNameInput / InlineItemSearch** | In-grid item search with voice (Web Speech API) and fuzzy matching |
| **NetworkStatus** | Online/offline dot, pending-change count, auto-sync toggle, manual sync |
| **AlertsBell** | Low stock, overdue party payments, licence expiry alerts |
| **InvoiceUploadDialog** | AI bill upload (PDF/JPG/PNG) with progress, error detail and prefill hand-off |
| **DocumentItemsTable** | Shared spreadsheet line grid used by invoices and documents |
| **ShareLinkPanel / SendDocLinkDialog** | Public link creation, password, expiry, revoke and email delivery |
| **ReportViewDialog / ShareReportDialog** | In-app report view; email + public link + portal sharing |
| **InvoiceDesigner / InvoiceCanvasDesigner / BackgroundEditor** | Custom invoice layout designer |
| **CustomFieldsManager / IndustrySettingsPanel** | Custom field defs and industry packs |
| **BarcodeScannerDialog / BulkBarcodePrintDialog** | Camera/USB scanning and label sheet printing |
| **KotOrderDialog / RestaurantSettingsPanel** | Restaurant order tickets and module settings |
| **LoyaltyRedeemPanel / LoyaltyCardPrintDialog** | Points redemption and card printing |
| **WhatsAppShareDialog / WhatsAppSettingsPanel** | WhatsApp message templates and sharing |
| **LicenseHistory / SavingOverlay / ThemeToggle / HeaderPortal** | Licence history, save feedback, theming, header slots |

### 10.5 Custom Hooks

| Hook | Purpose |
|------|---------|
| `useIsMobile()` | Returns `true` when viewport < 768px |
| `useToast()` | Toast notification system (max 1 visible, auto-dismiss) |
| `useSyncStatus()` | Subscribes to the offline sync engine (online, pending, syncing, autoSync, lastSyncedAt) |
| `usePersistedColumns(key, allKeys, defaults)` | Persist table column visibility to localStorage |
| `useHiddenColumns(...)` | Hide/show columns on the invoice & document line grids |
| `useRowSelection(rows)` | Row selection state for bulk operations |
| `useRowActivate(...)` | Keyboard/click row activation for list pages |
| `useShowInactiveItems()` | Toggle inactive items visibility (localStorage persisted) |
| `useBillingZoom()` | Zoom level for the billing/POS panel |
| `useCustomTemplates()` | Load/save custom invoice designer templates (`invoice_templates`) |

### 10.6 Utility Modules

| Module | Purpose | Key Exports |
|--------|---------|-------------|
| **activityLog.ts** | Audit logging | `logActivity()` — inserts to `activity_log` |
| **invoiceCalc.ts** | Invoice math engine | `calcLineAmount()`, `solveRateFromAmount()`, `computeInvoiceTotals()` — tax-inclusive/exclusive, extra charges, GST split |
| **invoicePdf.ts** | Invoice print/PDF | Thermal 58/80 mm, A5 (single/2-page), A4; UPI QR; amount in words (Lakh/Crore) |
| **invoiceGstTemplates.ts** | Statutory GST invoice layouts | A4/A5 GST templates with HSN summary, language + font support |
| **customInvoiceTemplate.ts** | Designer output | Renders saved `invoice_templates` designs to print HTML |
| **invoiceFieldPrint.ts** | Custom fields on print | Maps custom field defs to invoice columns/blocks |
| **printHtml.ts** | Print pipeline | Opens generated HTML in a hidden iframe for print/PDF |
| **documentPrint.ts** | Documents print | Estimate / order / challan print layouts |
| **kotPrint.ts** | Kitchen tickets | Thermal KOT layout for the restaurant module |
| **paymentPdf.ts** | Payment receipts | Thermal/A5/A4 receipt and voucher formats |
| **statementPdf.ts** | Party statements | Ledger/statement PDF with running balance |
| **gstExport.ts** | GST return Excel | `exportGSTR1/2/3B()` — B2B, B2CL, B2CS, CDNR, HSN sheets |
| **gstReturns.ts** | GST return HTML | `gstr1Html`, `gstr2Html`, `gstr3bHtml`, `gstr9Html` shared by view + export |
| **reportGenerators.ts** | Report data computation | Pure data functions for all 18 reports + expenses |
| **reportArtifacts.ts** | Shareable artifacts | `buildReportArtifact()` — HTML + XLSX base64 used by view, export and share |
| **reportExport.ts** | Direct export | PDF + Excel export per report type |
| **expenseReports.ts** | Expense reporting | Category breakdown + ITC summary export |
| **importExport.ts** | Bulk import/export | Party/item/invoice/payment/expense Excel & CSV with templates and validation |
| **bulkExport.ts** | Multi-record export | Bulk PDF/Excel export from list pages |
| **razorpay.ts** | Payment SDK | `loadRazorpay()`, `openRazorpay()` — lazy-loads checkout |

---

## 11. Supabase Edge Functions

| Function | Endpoint | Auth | Purpose |
|----------|----------|------|---------|
| **create-user** | POST | JWT (admin/owner) | Create or invite user into a company. Checks seat limit. Creates auth user if new, inserts `company_members` |
| **lookup-email** | POST | None (public) | Phone-to-email lookup for phone login. Normalizes +91 prefix, matches last 10 digits |
| **manage-user** | POST | JWT (admin) | Update (email, password, name) or delete a user. Cannot delete self |
| **razorpay-create-order** | POST | JWT | Create Razorpay order for monthly (₹199) or lifetime (₹9,999) plan. Applies coupon discount. Returns order_id + key_id |
| **razorpay-verify-payment** | POST | JWT | Verify HMAC-SHA256 signature. Issue license via `extend_or_create_paid_license`. Record coupon redemption |
| **razorpay-webhook** | POST | Razorpay signature | Async payment event handler. Handles `payment.captured`, `payment.failed`, `refund.processed`. Uses `timingSafeEqual` |
| **send-password-reset** | POST | None (public) | Sends a password-reset link for an email/phone account |
| **loyalty-card-lookup** | POST | None (public) | Customer-facing card portal lookup by phone + card number |
| **public-doc** | GET/POST | None (public) | Serves a shared invoice/payment by document id; enforces `doc_shares` revocation, expiry and SHA-256 password gate; tracks view counts |
| **public-report** | GET/POST | None (public) | Serves a shared report snapshot by `public_token`; enforces `public_enabled`/`expires_at`; tracks view counts |
| **send-doc-link** | POST | JWT (RLS-verified) | Emails an invoice/payment share link via Resend; logs to `doc_share_emails` |
| **send-report-link** | POST | JWT (RLS-verified) | Emails a report share link via Resend; logs to `report_share_emails` |
| **parse-invoice-file** | POST | JWT | AI invoice/bill OCR. Accepts base64 PDF or image, returns structured invoice JSON (party, dates, line items, discounts, GST, totals). Tries providers from `ai_providers` by priority, then `GEMINI_API_KEY`, then the Lovable AI gateway. The file is never persisted |

| **super-admin** | POST | JWT (super admin) | Cross-tenant console actions: businesses, users, licences, storage/DB stats |
| **api** | GET/POST/PATCH/DELETE | JWT (manual) | Public REST API v1 over every business resource + report endpoints. See [docs/API.md](docs/API.md) |

### Edge Function Security Notes

- Most functions have `verify_jwt = false` in `config.toml` because they extract and validate JWT manually
- `razorpay-webhook` uses Razorpay webhook signature verification (HMAC-SHA256)
- `lookup-email` is intentionally public but only returns email (no other PII)
- All admin operations validate caller role before executing

---

## 12. GST Compliance

### GST Calculation Logic

```
If business_state === party_state:
  CGST = total_gst × 0.5
  SGST = total_gst × 0.5
  IGST = 0
Else:
  CGST = 0
  SGST = 0
  IGST = total_gst
```

### Supported GST Rates
`0%`, `5%`, `12%`, `18%`, `28%`

### Tax-Inclusive Pricing
When `priceWithTax = true`, the rate is treated as inclusive of GST:
```
baseRate = rate / (1 + gstRate/100)
taxableAmount = qty × baseRate × (1 - discount%)
gstAmount = taxableAmount × gstRate / 100
```

### Invoice Extra Charges/Discounts
| Stage | Behavior |
|-------|----------|
| `pre` | Applied before GST — modifies the taxable base |
| `post` | Applied after GST — modifies the grand total only |

### GSTR Report Formats

| Report | Purpose | Sheets Generated |
|--------|---------|-----------------|
| **GSTR-1** | Sales return | B2B (registered dealers), B2CL (>₹2.5L inter-state unregistered), B2CS (small unregistered), CDNR (credit/debit notes), HSN (HSN-wise summary) |
| **GSTR-2** | Purchase return | B2B, CDNR |
| **GSTR-3B** | Summary return | 3.1 Outward Supplies, 4. Input Tax Credit, 6.1 Tax Payment |

### Indian State Codes
All 36 states and union territories mapped with proper GST state codes for Place of Supply.

---

## 13. Licensing & Billing System

### License Types

| Type | Description | Duration |
|------|-------------|----------|
| `trial` | Auto-created on signup | Configurable via `app_settings.trial_days` (default 7) |
| `paid` | Purchased via Razorpay | Monthly (30 days) or Lifetime (~100 years) |
| `complimentary` | Issued by super admin | Custom duration |

### Pricing Plans

| Plan | Price | Duration | Seats |
|------|-------|----------|-------|
| Monthly | ₹199/month | 30 days | 3 |
| Lifetime | ₹9,999 one-time | ~100 years | 3 |

### License Lifecycle

1. **User signs up** → Trigger creates trial license (N days)
2. **Trial expires** → All routes redirect to `/billing` (LicenseGate)
3. **User pays** → Razorpay checkout → `extend_or_create_paid_license()`
4. **License extended** → If active license exists, days are added; otherwise new license created
5. **Key redemption** → Pre-generated keys can be redeemed via `redeem_license()` RPC
6. **Super admin** → Can generate, revoke, and manage all licenses

### Coupon System

- Coupon types: `percent` (e.g., 20% off) or `flat` (e.g., ₹50 off)
- Plan-specific: `monthly`, `lifetime`, or `both`
- Usage limits: `max_uses` (NULL = unlimited)
- Expiry: optional `expires_at` timestamp
- Validation via `validate_coupon()` RPC → returns `{ valid, discount, final_amount }`
- One redemption per user per coupon (enforced by `UNIQUE(coupon_id, user_id)`)

---

## 14. Role-Based Access Control (RBAC)

### Role Hierarchy

```
super_admin (platform level — super_admins table)
  └── owner (company level)
       └── admin
            └── staff
                 └── member
                      └── party (external, portal-only)
```

### Role Capabilities

| Capability | Owner | Admin | Staff | Member | Party |
|------------|-------|-------|-------|--------|-------|
| View company data | ✓ | ✓ | ✓ | ✓ | Limited |
| Create/edit records | ✓ | ✓ | ✓ | ✓ | ✗ |
| Delete records | ✓ | ✓ | ✗ | ✗ | ✗ |
| Manage members | ✓ | ✓ | ✗ | ✗ | ✗ |
| Edit payments | ✓ | ✓ | ✓* | ✗ | ✗ |
| Delete company | ✓ | ✗ | ✗ | ✗ | ✗ |
| View all companies (super admin) | ✗ | ✗ | ✗ | ✗ | ✗ |

*Staff can edit payments only if `can_edit_payments = true` on their membership.

### Page-Level Permissions

Each non-owner/admin member has granular `page_permissions` stored as JSON in `company_members`:

| Permission | Description |
|-----------|-------------|
| `view` | Can see the page and its data |
| `create` | Can create new records |
| `edit` | Can modify existing records |
| `delete` | Can remove records |

Pages: `dashboard`, `parties`, `items`, `sales`, `purchases`, `payment_in`, `payment_out`, `sale_return`, `purchase_return`, `reports`, `settings`

### Data Isolation

- All business tables have `company_id` column
- RLS policies enforce `is_company_member(company_id, auth.uid())`
- Party users only see data linked via `party_access` table
- Even raw SQL queries are isolated by PostgreSQL RLS

---

## 15. Key Features Summary

### Core Accounting
- [x] Sale/Purchase invoices with auto-numbering
- [x] Sale returns (credit notes) and purchase returns (debit notes)
- [x] Payment-in and payment-out with invoice linking
- [x] Outstanding balance tracking per party
- [x] Party ledger with running balance
- [x] Item transaction history with stock movements

### GST Compliance
- [x] CGST/SGST/IGST auto-split based on state
- [x] Tax-inclusive and tax-exclusive pricing
- [x] HSN code tracking
- [x] GSTR-1, GSTR-2, GSTR-3B exports
- [x] B2B, B2CL, B2CS, CDNR, HSN summary sheets

### Invoice Features
- [x] 7 invoice formats: A4, A5, A5 2-page, Thermal 58mm, Thermal 80mm, A4 GST, A5 GST
- [x] GST invoice templates with HSN summary and GST rate aggregation
- [x] Invoice language selection (11 Indian languages) — applies to GST templates
- [x] Invoice font family selection (per-company, stored in DB)
- [x] Extra charges (TCS, Freight, Packing) and extra discounts (Trade, Loyalty)
- [x] Pre-tax and post-tax charge/discount application
- [x] Auto round-off with manual override
- [x] UPI QR code on invoices
- [x] Amount in words (Indian numbering: Lakh, Crore)
- [x] Reverse calculations (edit amount → solve rate, edit GST ₹ → solve GST rate)
- [x] Multi-tab simultaneous invoice creation

### AI Invoice Upload & Parsing
- [x] Upload a sale invoice or purchase bill as PDF / JPG / PNG (max 8MB) via `InvoiceUploadDialog` (drag-and-drop or file picker), from Sales → Import and Purchases → Import
- [x] File is read client-side as base64 and streamed to the `parse-invoice-file` edge function — **never uploaded to Storage or stored in the DB**
- [x] Structured extraction via tool/function calling: invoice number, date, party (name, GSTIN, state, phone, email, address), notes, line items (name, HSN, unit, qty, MRP, rate, discount + type, GST %/₹, line total), subtotal, CGST/SGST/IGST, round-off, total, amount paid
- [x] No review popup — extracted JSON is stashed in `sessionStorage` (`sb:invoice-prefill`) and `CreateInvoice` opens prefilled at `/sales/new?type=sale|purchase&prefill=1`
- [x] Party and item names are fuzzy-matched to existing records; GST/IGST recalculated by the normal invoice calc so totals stay consistent
- [x] Everything remains editable before save; nothing is written until the user saves the invoice
- [x] Multi-provider fallback chain, tried in priority order: `ai_providers` rows (Gemini, OpenAI, Anthropic, OpenRouter, any OpenAI-compatible endpoint) → `GEMINI_API_KEY` secret → Lovable AI gateway (`LOVABLE_API_KEY`)
- [x] Falls through on 429 / 402 / 403 / 5xx and on Gemini model 404s (retries current model ids); per-provider `last_used_at` / `last_error` recorded in `ai_providers`
- [x] Keys managed from Super Admin → AI providers (`SuperAiProviders`, backed by migration `041_ai_providers.sql`, service-role-only table + super-admin RPCs)



### Voice Input
- [x] Web Speech API integration (Chrome/Edge)
- [x] Fuzzy item name matching (exact → endsWith → contains → word match)
- [x] Multi-item voice commands ("5 kg rice and 2 cement")
- [x] Quantity + unit recognition ("5 kg", "dozen", etc.)

### Expenses
- [x] Category-wise expense tracking with GST
- [x] Vendor linking and payment status tracking
- [x] Recurring expenses (daily/weekly/monthly/quarterly/yearly)
- [x] ITC eligibility flag for GST credit claims
- [x] Expense report with category breakdown and ITC summary

### Inventory
- [x] Products and services with categories
- [x] Dual unit support (e.g., 1 box = 12 pcs)
- [x] Opening stock with as-of date
- [x] Min stock quantity alerts
- [x] Stock auto-adjustment on invoice creation
- [x] Active/inactive item toggle
- [x] Item images

### Import/Export
- [x] Excel and CSV import/export for parties, items, invoices, payments, expenses
- [x] Template downloads for imports
- [x] Import validation with error reporting
- [x] Full company data backup (.bkp JSON format)
- [x] FK-aware restore with batch upserts (500 per batch) — includes expenses + expense_payments

### Multi-Company
- [x] Create and manage multiple companies
- [x] Switch between companies via CompanySwitcher
- [x] Per-company data isolation (RLS enforced)
- [x] Company-specific business profiles

### Multi-Currency (optional, off by default — not exposed on the Settings page)
- [x] `src/lib/currency.ts`: 17 currencies, live FX rates from `open.er-api.com` with a 1-hour cache, `convert()` / `formatMoney()`
- [x] When enabled, `CreateInvoice` shows a currency + exchange-rate selector and stores `currencyCode`/`exchangeRate` on the invoice
- [x] Flag and base currency persist on `business_profiles.multi_currency_enabled` / `base_currency` (mirrored to localStorage)
- [ ] No toggle on the Settings page — it is switched from the Backup & Restore screen; most reports and print templates still assume the base currency

### Team & Portal
- [x] Invite team members with roles (owner/admin/staff/member/party)
- [x] Granular page-level permissions (view/create/edit/delete)
- [x] Party portal for external customers/suppliers
- [x] Report sharing between users
- [x] Activity audit log

### Billing & Licensing
- [x] Trial license auto-provisioned on signup
- [x] Razorpay payment integration (Monthly ₹199, Lifetime ₹9,999)
- [x] Coupon/discount system (percent/flat, plan-specific)
- [x] License key generation and redemption
- [x] Super admin dashboard for license management
- [x] Payment mode toggle (enable/disable online payments)

### Reports (18 Types)
- [x] 1. Sale Report, 2. Purchase Report
- [x] 3. Day Book, 5. All Transactions
- [x] 4. Profit & Loss, 7. Party-wise P&L, 10. Item-wise P&L
- [x] 6. Party Statement, 8. All Parties Report
- [x] 9. Stock Summary, 11. Stock Detail
- [x] 12. Item Sale & Purchase Detail, 13. Party Sale & Purchase Detail
- [x] 14. Party Payment Report
- [x] 15. GSTR-1, 16. GSTR-2, 17. GSTR-3B, 18. GSTR-9 (annual)
- [x] Expense Report (with category breakdown & ITC summary)
- [x] In-app **View** dialog and PDF/Excel export share one HTML builder — the statutory GST
      layouts (`gstr1Html` / `gstr2Html` / `gstr3bHtml` / `gstr9Html` in `src/utils/gstReturns.ts`)
      are used by both paths via `standaloneHtml` in `src/utils/reportArtifacts.ts`
- [x] PDF, Excel, print and WhatsApp share for all reports
- [x] Share reports to team members and party portal users

### Documents & Recurring (module)
- [x] Estimates / quotations, sale orders, purchase orders, delivery challans (`documents` table)
- [x] Shared spreadsheet-style line grid (`src/components/DocumentItemsTable.tsx`) identical to
      the invoice panel: inline item search, item picker, quick-add item, drag reorder,
      HSN/qty/unit/MRP/rate/disc %/disc ₹/GST %/GST ₹/amount columns with reverse-calc
- [x] Convert any document into an invoice (carrying lines, charges and custom fields)
- [x] Recurring invoice schedules (daily/weekly/monthly/quarterly/yearly) with next-run tracking

### Point of Sale
- [x] Touch POS screen with item tiles/images, category quick filters and keypad
- [x] Default walk-in party, default tender mode, mark-paid and auto-print settings
- [x] Thermal 58/80 mm receipt printing; POS sales flagged with `pos_sale` + `payment_mode`
- [x] Billing zoom control and barcode scanner dialog

### Restaurant Module
- [x] Areas and tables with live status (free / running / billed) — `restaurant_areas`, `restaurant_tables`
- [x] KOT creation and Kitchen Display Screen — `kots`
- [x] Table reservations with guest, time and cover count — `reservations`
- [x] Table-to-invoice conversion and restaurant settings panel

### Loyalty & Rewards
- [x] Points earn/redeem rules, tiers and per-party overrides (`loyalty_transactions`)
- [x] Redemption applied as an invoice discount with balance printed on the bill
- [x] Printable loyalty cards with barcodes and a public `/cards` portal (edge function lookup)

### Online Store
- [x] Publishable catalogue and storefront (`store_settings`, `store_items`)
- [x] Online orders and customers (`store_orders`, `store_order_items`, `store_customers`)
- [x] Order → invoice conversion, storefront branding/customisation

### E-Way Bill & Compliance Extras
- [x] Per-invoice transport block (transporter, vehicle, distance, doc details) with print toggle
- [x] Business-level e-way defaults and an auto-open amount threshold

### Barcodes & Labels
- [x] Item barcodes, camera/USB scanning into invoices and POS
- [x] Bulk barcode/price label printing dialog with sheet layout options

### Custom Fields & Industry Packs
- [x] User-defined custom fields scoped to business / party / item / invoice / line
- [x] Industry packs: pharma, manufacturing, garment, jewellery
- [x] Custom fields printable as invoice columns

### WhatsApp & Sharing
- [x] Editable WhatsApp message templates for invoices, receipts and statements
- [x] Share dialogs for invoices, payments and reports; sender number in settings

### Alerts & Stock Corrections
- [x] Alerts bell: low stock, overdue party payments, licence expiry
- [x] Manual stock adjustments with reason and note (`stock_adjustments`)

### Super Admin Console
- [x] Separate `super.html` entry with businesses, users, licences and DB/storage usage views
- [x] Backed by `sa_*` RPCs and the `super-admin` edge function

---
## 16. Testing

### Unit Tests (Vitest)

```bash
# Run tests
bun run test

# Run tests in watch mode
bun run test:watch
```

**Configuration:** `vitest.config.ts` with `@testing-library/jest-dom` matchers. Test setup in `src/test/setup.ts`.

### E2E Tests (Playwright)

```bash
# Run Playwright tests
npx playwright test
```

**Configuration:** `playwright.config.ts` with custom fixture in `playwright-fixture.ts`.

---

## 17. Scripts Reference

| Script | Command | Description |
|--------|---------|-------------|
| `dev` | `vite` | Start dev server on port 8080 |
| `build` | `vite build` | Production build |
| `build:dev` | `vite build --mode development` | Development build with source maps |
| `preview` | `vite preview` | Preview production build locally |
| `lint` | `eslint .` | Run ESLint |
| `test` | `vitest run` | Run unit tests once |
| `test:watch` | `vitest` | Run unit tests in watch mode |

---


## 18. Module Reference

| Module | Pages | Contexts | Key tables |
|--------|-------|----------|------------|
| Core billing | `Sales`, `Purchases`, `CreateInvoice`, `EditInvoice`, `SaleReturn`, `PurchaseReturn` | `AppContext` | `invoices`, `invoice_items` |
| Parties & items | `Parties`, `Items`, `PartyLedger`, `ItemHistory` | `AppContext`, `FieldsContext` | `parties`, `items`, `categories`, `custom_units`, `stock_adjustments` |
| Money | `PaymentIn`, `PaymentOut`, `Expenses` | `AppContext`, `ExpenseContext` | `payments`, `expenses`, `expense_payments` |
| Documents | `DocumentsPage`, `DocumentEditor`, `RecurringInvoices`, `RecurringEditor` | `DocumentsContext` | `documents`, `recurring_invoices` |
| POS | `Pos`, `PosSettings` | `AppContext` | `invoices` (`pos_sale`) |
| Restaurant | `RestaurantTables`, `KitchenDisplay`, `Reservations` | `RestaurantContext` | `restaurant_areas`, `restaurant_tables`, `kots`, `reservations` |
| Loyalty | `Loyalty`, `CardsPortal` | `AppContext` | `loyalty_transactions`, `parties.loyalty_config` |
| Online store | `OnlineStore`, `StoreFront`, `MyOnlineOrders` | — | `store_settings`, `store_items`, `store_orders`, `store_order_items`, `store_customers` |
| Reports | `Reports` | `AppContext`, `ExpenseContext` | reads all business tables, writes `shared_reports` |
| Admin & billing | `AdminPanel`, `Billing`, `BackupRestore`, `SettingsPage`, `EwaySettings` | `CompanyContext`, `LicenseContext` | `company_members`, `licenses`, `coupons`, `subscription_payments`, `business_profiles`, `invoice_templates`, `activity_log` |
| Super console | `src/super/*` (`super.html`) | — | `super_admins` + `sa_*` RPCs |

---

## 19. End-to-End Application Flow

### 19.1 First run to first bill

```text
Landing (/)  ->  Sign up (/auth?signup=1)
   |                 |- auth.users row created
   |                 |- trigger: profiles + personal company + company_members(owner)
   |                 |- trigger: trial license issued
   v
Dashboard (/)  ->  Settings: business profile, GSTIN, logo, brand colours,
   |               invoice template + font, industry packs, custom fields,
   |               POS / e-way / loyalty / WhatsApp / restaurant settings
   v
Masters      ->  Parties (customers & suppliers, opening balances, credit limits)
   |            Items (HSN, units, prices, opening stock, barcodes, images)
   |            (or Excel/CSV import from the Items/Parties pages)
   v
Transaction  ->  Sales -> New Invoice  (or POS, or Restaurant table, or Store order)
   |               |- pick party, add lines (search / picker / barcode / voice)
   |               |- GST auto-split by state, charges & discounts, round-off,
   |                  loyalty redemption, e-way block, custom fields
   |               |- Save -> invoices + invoice_items, stock decremented,
   |                  loyalty points posted, activity_log entry
   v
Share        ->  Print (A4/A5/thermal/designer) | PDF | WhatsApp | Party portal
   v
Collect      ->  Payment In (links invoice, updates amount_paid + status)
   v
Analyse      ->  Dashboard tiles, Alerts bell, Reports (18) -> View / PDF / Excel / Share
   v
Comply       ->  GSTR-1 / 2 / 3B / 9 exports for the CA
   v
Protect      ->  Backup & Restore (.bkp) and continuous cloud sync
```

### 19.2 Quote-to-cash (documents path)

```text
Estimate (documents.type = estimate)
   -> customer approves
   -> Sale Order            (stock reserved conceptually, not deducted)
   -> Delivery Challan      (goods dispatched)
   -> Convert to Invoice    (invoices + invoice_items, stock deducted)
   -> Payment In            (partial or full; status paid / partial / unpaid)
   -> Party Statement / Outstanding report
```

### 19.3 Procure-to-pay

```text
Purchase Order (documents.type = purchase_order)
   -> Purchase Invoice (stock incremented, ITC captured)
   -> Payment Out
   -> Purchase Return (debit note) if goods are rejected
   -> GSTR-2 / GSTR-3B input tax
```

### 19.4 Restaurant service cycle

```text
Reservation (optional)  ->  Table occupied on the floor plan
   -> Order taken on the table  ->  KOT sent to Kitchen Display
   -> KOT marked ready / served
   -> Table bill  ->  invoice (pos_sale = true) + thermal receipt
   -> Payment captured, table released back to free
```

### 19.5 Retail POS cycle

```text
POS screen -> scan barcode / tap tile -> qty & discount
   -> loyalty points redeemed (optional) -> tender mode (cash/UPI/bank/cheque)
   -> Save = invoice + stock update + auto-print receipt
   -> Day Book / Sale Report reflect it instantly
```

### 19.6 Online store cycle

```text
Publish items -> share storefront link -> customer places order (store_orders)
   -> owner reviews in Online Store / My Online Orders
   -> convert to invoice -> fulfil -> payment -> reports
```

### 19.7 Team, licence and data lifecycle

```text
Owner -> Admin Panel -> invite user (create-user edge fn, seat check)
      -> role + per-page permissions stored on company_members
Licence -> trial -> Billing page -> Razorpay order -> signature verified
      -> extend_or_create_paid_license -> licences history + alerts before expiry
Data  -> Backup & Restore (.bkp) | Excel export | REST API (section 20)
```

---

## 20. REST API (v1)

`supabase/functions/api` is a single-function REST gateway that exposes every business
resource and a set of aggregate report endpoints. It exists for mobile apps,
third-party integrations and load/performance testing.

- Base URL: `https://<project-ref>.supabase.co/functions/v1/api/v1`
- Auth: `Authorization: Bearer <supabase access token>` (validated inside the function)
- Tenancy: optional `X-Company-Id` header; defaults to the caller's first company
- Security: all queries run with the caller's JWT, so **RLS is enforced** and
  `company_id` is injected automatically on writes
- Envelope: `{ "data": ..., "meta": { count, limit, offset } }`, errors as
  `{ "error": { message, status } }`

Resource paths include `parties`, `items`, `invoices`, `invoice-items`, `payments`,
`expenses`, `documents`, `recurring-invoices`, `stock-adjustments`,
`loyalty-transactions`, `invoice-templates`, `business-profile`, `restaurant-areas`,
`restaurant-tables`, `kots`, `reservations`, `store-*`, `shared-reports`, plus
read-only `activity-log`, `members` and `licenses`.

Aggregates: `/reports/summary`, `/reports/gst-summary`, `/reports/stock`,
`/reports/outstanding`. Composite write: `POST /invoices/with-items`.

Deploy with `supabase functions deploy api`. Full reference, query parameters,
curl examples, mobile-app guidance and a k6 load-test script:
**[docs/API.md](docs/API.md)**.

---

## 21. Related Documentation

| Document | Audience | Contents |
|----------|----------|----------|
| `README.md` (this file) | Developers | Architecture, schema, flows, features, API overview |
| [`docs/FEATURES.md`](docs/FEATURES.md) | Marketing, sales, end users | Plain-English, point-wise feature list with no technical terms |
| [`docs/API.md`](docs/API.md) | Integrators, mobile & QA teams | REST API v1 reference, auth, examples, performance testing |
| `supabase/manual-migrations/README.md` | Operators | Multi-tenancy migration run book |

---

## License

This project is proprietary software. All rights reserved.
