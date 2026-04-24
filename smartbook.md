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

---

## 1. Project Overview

**SmartBooks** is a cloud-native invoicing, accounting, and inventory management platform built for Indian small-to-medium businesses. It handles the full spectrum of day-to-day accounting:

- **Invoicing** — Create sale, purchase, sale-return (credit note), and purchase-return (debit note) invoices with automatic GST calculations (CGST/SGST/IGST)
- **Payments** — Record payment-in (from customers) and payment-out (to suppliers) with automatic outstanding balance tracking
- **Inventory** — Track products and services with dual-unit support, stock management, and low-stock alerts
- **Party Management** — Customer and supplier directory with GSTIN validation, opening balances, and credit limits
- **GST Reports** — GSTR-1, GSTR-2, GSTR-3B report generation and export
- **Multi-Company** — Users can create and manage multiple businesses under a single account
- **Team Collaboration** — Invite staff with granular page-level permissions (view/create/edit/delete)
- **Party Portal** — External customers/suppliers can log in to view their own invoices, payments, and shared reports
- **Licensing** — Trial/paid/complimentary licenses with Razorpay payment integration and coupon system
- **Backup & Restore** — Full data export/import with FK-aware ordering

### Business Context

- **Target Market**: Indian SMBs requiring GST-compliant invoicing
- **Currency**: INR (₹)
- **GST Logic**: Intra-state → CGST + SGST (50/50 split), Inter-state → IGST
- **Supported GST Rates**: 0%, 5%, 12%, 18%, 28%
- **All 36 Indian States/UTs** supported for GST Place of Supply

---

## 2. Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 18, TypeScript, Vite 5 |
| **Styling** | Tailwind CSS 3, shadcn/ui (Radix UI primitives) |
| **State Management** | React Context (AppContext, AuthContext, CompanyContext, LicenseContext), TanStack React Query |
| **Routing** | React Router DOM v6 |
| **Forms** | React Hook Form + Zod validation |
| **Animations** | Framer Motion |
| **Charts** | Recharts |
| **Backend** | Supabase (PostgreSQL, Auth, Edge Functions, Storage) |
| **Payments** | Razorpay Checkout SDK + Server-side Orders API |
| **PDF Generation** | HTML-to-print (iframe/window), QR codes via `qrcode` library |
| **Excel/CSV** | `xlsx` (SheetJS), `papaparse` |
| **Voice Input** | Web Speech API (SpeechRecognition) |
| **Testing** | Vitest, Testing Library, Playwright |
| **Linting** | ESLint with TypeScript + React Hooks plugins |
| **Package Manager** | Bun (bun.lockb) |

---

## 3. Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                      Browser (SPA)                       │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  React + Vite + TypeScript + Tailwind + shadcn/ui   │ │
│  │                                                     │ │
│  │  ┌───────────┐ ┌──────────────┐ ┌───────────────┐   │ │
│  │  │ AuthCtx   │→│ CompanyCtx   │→│ LicenseCtx    │   │ │
│  │  └───────────┘ └──────────────┘ └───────────────┘   │ │
│  │        │               │               │            │ │
│  │        └───────────────┼───────────────┘            │ │
│  │                        ▼                            │ │
│  │              ┌──────────────────┐                   │ │
│  │              │    AppContext    │                   │ │
│  │              │  (All CRUD ops) │                    │ │
│  │              └────────┬─────────┘                   │ │
│  │                       │                             │ │
│  │  ┌────────┬──────────┼──────────┬────────────┐      │ │
│  │  │Pages   │Components│ Hooks    │ Utils       │     │ │
│  │  │22 pages│20 comps  │5 hooks   │10 modules   │     │ │
│  │  └────────┴──────────┴──────────┴────────────┘      │ │
│  └─────────────────────────────────────────────────────┘ │
│                          │                               │
│              ┌───────────┼───────────┐                   │
│              ▼           ▼           ▼                   │
│    Supabase Client   Razorpay SDK  Web Speech API        │
└──────────────┬───────────────────────────────────────────┘
               │ HTTPS / WebSocket
               ▼
┌──────────────────────────────────────────────────────────┐
│                    Supabase Platform                     │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐  │
│  │  PostgreSQL  │ │ Auth (GoTrue)│ │  Edge Functions  │  │
│  │  19 tables   │ │  JWT + OAuth │ │  6 functions     │  │
│  │  22+ RLS     │ │              │ │  - create-user   │  │
│  │  policies    │ │              │ │  - manage-user   │  │
│  │  16 functions│ │              │ │  - lookup-email  │  │
│  │  12 triggers │ │              │ │  - razorpay-*    │  │
│  └──────────────┘ └──────────────┘ └──────────────────┘  │
│  ┌──────────────┐                                        │
│  │   Storage    │                                        │
│  │  shared-pdfs │                                        │
│  └──────────────┘                                        │
└──────────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────┐
│              Razorpay Payment Gateway                    │
│  Orders API → Checkout Modal → Webhook Verification      │
└──────────────────────────────────────────────────────────┘
```

---

## 4. Project Structure

```
remix-of-lighting-fast-main/
├── public/
│   └── robots.txt
├── src/
│   ├── main.tsx                        # App entry point (React root)
│   ├── App.tsx                         # Router, providers, route definitions
│   ├── App.css                         # Global app styles
│   ├── index.css                       # Tailwind base + custom CSS variables
│   ├── vite-env.d.ts                   # Vite type declarations
│   ├── assets/                         # Static assets
│   ├── components/
│   │   ├── AppLayout.tsx               # Main shell (sidebar + header + content)
│   │   ├── AppSidebar.tsx              # Navigation sidebar with permission filtering
│   │   ├── BulkActionBar.tsx           # Sticky bar for bulk operations
│   │   ├── CompanySwitcher.tsx         # Multi-company dropdown
│   │   ├── CouponsAdmin.tsx            # Coupon CRUD management
│   │   ├── ExtraChargesEditor.tsx      # Invoice extra charges/discounts
│   │   ├── InvoicePreviewDialog.tsx    # Invoice PDF preview & print
│   │   ├── ItemPickerDialog.tsx        # Multi-item selection for invoices
│   │   ├── NavLink.tsx                 # Active-aware navigation link
│   │   ├── PartyCombobox.tsx           # Searchable party selector
│   │   ├── PaymentPreviewDialog.tsx    # Payment receipt preview & print
│   │   ├── QuickAddItemDialog.tsx      # Inline item creation from invoices
│   │   ├── QuickAddPartyDialog.tsx     # Inline party creation from invoices
│   │   ├── SentReportsTab.tsx          # Reports sent by current user
│   │   ├── SharedReportsTab.tsx        # Reports received by current user
│   │   ├── ShareReportDialog.tsx       # Share report with portal users
│   │   ├── SortHeader.tsx              # Sortable table column header
│   │   ├── StatCard.tsx                # Animated metric card
│   │   ├── UserMenu.tsx                # User avatar/menu dropdown
│   │   ├── VoiceItemInput.tsx          # Voice-based item entry
│   │   └── ui/                         # shadcn/ui component library
│   ├── contexts/
│   │   ├── AppContext.tsx              # Central data store (CRUD for all entities)
│   │   ├── AuthContext.tsx             # Authentication state
│   │   ├── CompanyContext.tsx          # Multi-company & permissions
│   │   └── LicenseContext.tsx          # License/subscription state
│   ├── hooks/
│   │   ├── use-mobile.tsx             # Responsive breakpoint (< 768px)
│   │   ├── use-persisted-columns.ts   # Column visibility persistence
│   │   ├── use-row-selection.ts       # Table row selection state
│   │   ├── use-show-inactive-items.ts # Toggle inactive items visibility
│   │   └── use-toast.ts              # Toast notification system
│   ├── integrations/
│   │   └── supabase/
│   │       ├── client.ts              # Supabase client singleton
│   │       └── types.ts              # Auto-generated DB types
│   ├── lib/
│   │   └── utils.ts                   # cn() Tailwind merge utility
│   ├── pages/
│   │   ├── AdminPanel.tsx             # Company member & settings management
│   │   ├── Auth.tsx                   # Login/signup page
│   │   ├── BackupRestore.tsx          # Full data backup & restore
│   │   ├── Billing.tsx                # License purchase & redemption
│   │   ├── CreateInvoice.tsx          # Invoice creation (sale/purchase/returns)
│   │   ├── Dashboard.tsx              # Business overview with stats
│   │   ├── EditInvoice.tsx            # Edit existing invoice
│   │   ├── Index.tsx                  # Root redirect
│   │   ├── ItemHistory.tsx            # Per-item transaction history
│   │   ├── Items.tsx                  # Product/service inventory management
│   │   ├── Landing.tsx                # Public marketing page
│   │   ├── LicensesAdmin.tsx          # Super admin license management
│   │   ├── NotFound.tsx               # 404 page
│   │   ├── Parties.tsx                # Customer/supplier management
│   │   ├── PartyLedger.tsx            # Per-party debit/credit ledger
│   │   ├── PartyPortal.tsx            # External party self-service portal
│   │   ├── PaymentIn.tsx              # Payment received from customers
│   │   ├── PaymentOut.tsx             # Payment made to suppliers
│   │   ├── PurchaseReturn.tsx         # Purchase return (debit notes)
│   │   ├── Purchases.tsx              # Purchase invoice list
│   │   ├── Reports.tsx                # 14 report types with export
│   │   ├── SaleReturn.tsx             # Sale return (credit notes)
│   │   ├── Sales.tsx                  # Sales invoice list
│   │   └── SettingsPage.tsx           # Business profile configuration
│   ├── test/
│   │   ├── example.test.ts            # Example test
│   │   └── setup.ts                   # Vitest setup
│   ├── types/
│   │   └── index.ts                   # TypeScript type definitions
│   └── utils/
│       ├── activityLog.ts             # Audit trail logging
│       ├── gstExport.ts              # GSTR-1/2/3B Excel export
│       ├── importExport.ts           # Bulk import/export (Excel/CSV)
│       ├── invoiceCalc.ts            # Invoice calculation engine
│       ├── invoicePdf.ts             # Invoice HTML/PDF generation
│       ├── paymentPdf.ts             # Payment receipt generation
│       ├── razorpay.ts               # Razorpay SDK loader & launcher
│       ├── reportArtifacts.ts        # Shareable report artifacts
│       ├── reportExport.ts           # Report PDF/Excel export
│       └── reportGenerators.ts       # Pure data computation for reports
├── supabase/
│   ├── config.toml                    # Supabase project configuration
│   ├── functions/
│   │   ├── create-user/index.ts       # Create/invite company member
│   │   ├── lookup-email/index.ts      # Phone-to-email lookup (login)
│   │   ├── manage-user/index.ts       # Admin user update/delete
│   │   ├── razorpay-create-order/     # Create Razorpay payment order
│   │   ├── razorpay-verify-payment/   # Verify payment & issue license
│   │   └── razorpay-webhook/          # Razorpay async event handler
│   ├── migrations/                    # 19 sequential SQL migrations
│   └── manual-migrations/             # 10 manual migration scripts
├── package.json
├── vite.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
├── eslint.config.js
├── postcss.config.js
├── components.json                    # shadcn/ui configuration
└── index.html                         # SPA entry HTML
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

# Edge function environment variables (set in Supabase dashboard):
# SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIs...
# RAZORPAY_KEY_ID=rzp_live_...
# RAZORPAY_KEY_SECRET=...
# RAZORPAY_WEBHOOK_SECRET=...
```

---

## 7. Application Workflow Diagrams

### 7.1 Authentication Flow

```
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  Landing    │     │  Auth Page   │     │   Supabase     │
│  Page (/)   │───▶│  (/auth)     │     │   Auth         │
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
│  │ [Share] → ShareReportDialog                        │  │
│  │           ├─ Select party-portal recipients        │  │
│  │           ├─ buildReportArtifact(HTML + XLSX64)    │  │
│  │           └─ Insert into shared_reports table      │  │
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
│  │     payments, business_profiles, custom_units,  │   │
│  │     categories, activity_log, shared_reports,   │   │
│  │     party_access                                │   │
│  │  2. Serialize to JSON                           │   │
│  │  3. Download as .bkp file                       │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  ┌─── Import (.bkp) ───────────────────────────────┐   │
│  │  1. Parse .bkp JSON file                        │   │
│  │  2. Purge existing data (FK-safe order):        │   │
│  │     a. NULL linked_invoice_id on invoices       │   │
│  │     b. Delete: payments → invoice_items →       │   │
│  │        invoices → party_access → items →        │   │
│  │        parties → activity_log →                 │   │
│  │        shared_reports → custom_units →          │   │
│  │        categories → business_profiles           │   │
│  │  3. Upsert in batches of 500:                   │   │
│  │     Reverse order (business_profiles first,     │   │
│  │     then parties, items, invoices, etc.)        │   │
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
│  │  │ Dashboard│  ✓   │   -    │  -   │   -     │   │    │
│  │  │ Sales    │  ✓   │   ✓    │  ✓  │   ✗    │    │    │
│  │  │ Items    │  ✓   │   ✓    │  ✗  │   ✗    │    │    │
│  │  │ ...      │      │        │      │         │    │    │
│  │  └──────────┴──────┴────────┴──────┴─────────┘    │    │
│  └───────────────────────────────────────────────────┘    │
│                                                          │
│  ┌─── Activity Log Tab ─────────────────────────────┐    │
│  │ [2026-04-24 10:30] Alice created Invoice INV-042 │    │
│  │ [2026-04-24 09:15] Bob edited Party "XYZ Corp"   │    │
│  │ [2026-04-23 18:00] Carol deleted Payment PAY-007 │    │
│  └──────────────────────────────────────────────────┘    │
│                                                          │
│  ┌─── Settings Tab ─────────────────────────────────┐    │
│  │ Allow New Signups: [Toggle]                      │    │
│  │ (Stored in app_settings.allow_signups)           │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

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
       ┌──────────────┐ ┌──────────────┐      │
       │ party_access │ │  invoices    │◄─────┘
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
Reports shared between users via the portal.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | `uuid` | PK |
| `shared_by` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
| `recipient_user_id` | `uuid` | NOT NULL, FK → `auth.users(id)` CASCADE |
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
| `created_at` | `timestamptz` | NOT NULL, DEFAULT `now()` |
| `viewed_at` | `timestamptz` | |
| `downloaded_at` | `timestamptz` | |

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

### 8.3 Enum Types

| Enum | Values | Usage |
|------|--------|-------|
| `app_role` | `admin`, `staff`, `user`, `party` | Global user roles |
| `company_role` | `owner`, `admin`, `staff`, `member`, `party` | Per-company roles |
| `license_type` | `trial`, `paid`, `complimentary` | License classification |
| `license_status` | `active`, `expired`, `revoked` | License state |
| `plan_code` | `monthly`, `lifetime` | Subscription plan |
| `coupon_type` | `percent`, `flat` | Discount calculation mode |
| `coupon_applies_to` | `monthly`, `lifetime`, `both` | Plan-specific coupons |
| `payment_status` | `created`, `paid`, `failed`, `refunded` | Razorpay payment state |

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
| `validate_coupon` | `(text, plan_code, numeric) → jsonb` | Validates coupon and returns discount |

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
| `update_companies_updated_at` | `companies` | BEFORE UPDATE | `update_updated_at_column()` |
| `update_company_members_updated_at` | `company_members` | BEFORE UPDATE | `update_updated_at_column()` |
| `trg_create_trial_license` | `companies` | AFTER INSERT | `create_trial_license()` |

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

### 8.8 Row-Level Security (RLS) Policies

All tables have RLS enabled. Policies use `SECURITY DEFINER` helper functions for role checks.

#### Core Business Tables (parties, items, invoices, invoice_items, payments, custom_units, categories)

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
                <AppLayout>    ← Sidebar, header, content area
                  {page}
                </AppLayout>
              </AppProvider>
          </LicenseProvider>
        </CompanyProvider>
      </AuthProvider>
    </BrowserRouter>
  </TooltipProvider>
</QueryClientProvider>
```

### 10.2 Routing & Page Guards

**Unauthenticated routes:**
| Path | Page | Access |
|------|------|--------|
| `/` | Landing | Public marketing page |
| `/auth` | Auth | Login/signup |
| `*` | → `/auth` | Redirect with return URL |

**Party user routes (role = party):**
| Path | Page | Access |
|------|------|--------|
| `/portal` | PartyPortal | Party users only |
| `*` | → `/portal` | Redirect |

**Authenticated user routes (license-gated):**
| Path | Page | Guard |
|------|------|-------|
| `/` | Dashboard | `dashboard` |
| `/parties` | Parties | `parties` |
| `/items` | Items | `items` |
| `/sales` | Sales | `sales` |
| `/purchases` | Purchases | `purchases` |
| `/sales/new` | CreateInvoice | `sales` |
| `/sales/edit/:id` | EditInvoice | `sales` |
| `/party-ledger/:id` | PartyLedger | `parties` |
| `/item-history/:id` | ItemHistory | `items` |
| `/payment-in` | PaymentIn | `payment_in` |
| `/payment-out` | PaymentOut | `payment_out` |
| `/sale-return` | SaleReturn | `sale_return` |
| `/purchase-return` | PurchaseReturn | `purchase_return` |
| `/reports` | Reports | `reports` |
| `/settings` | SettingsPage | `settings` |
| `/backup` | BackupRestore | `settings` |
| `/admin` | AdminPanel | (unrestricted for logged-in) |
| `/admin/licenses` | LicensesAdmin | (super admin redirect) |
| `/billing` | Billing | (always accessible, even expired) |
| `/portal` | PartyPortal | (any authenticated user) |

**LicenseGate:** When license is expired/missing, all routes except `/billing` redirect to the billing page.

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
| **Reports** | 14 report types | PDF/Excel export, share to portal, GSTR-1/2/3B |
| **PartyLedger** | Per-party ledger | Running balance, debit/credit entries, export |
| **ItemHistory** | Per-item transactions | Stock movements, sale/purchase totals, export |
| **SettingsPage** | Business configuration | Logo, GSTIN, state, invoice template, UPI ID |
| **AdminPanel** | Team management | Member CRUD, roles, page permissions, activity log, app settings |
| **Billing** | License management | Plans, Razorpay payment, coupon apply, key redemption |
| **LicensesAdmin** | Super admin licenses | Generate keys, manage licenses, coupons admin, payment toggle |
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
| **VoiceItemInput** | Web Speech API voice-to-item matching with NLP-lite parsing |

### 10.5 Custom Hooks

| Hook | Purpose |
|------|---------|
| `useIsMobile()` | Returns `true` when viewport < 768px |
| `usePersistedColumns(key, allKeys, defaults)` | Persist table column visibility to localStorage |
| `useRowSelection(rows)` | Generic row selection state for bulk operations |
| `useShowInactiveItems()` | Toggle inactive items visibility (localStorage persisted) |
| `useToast()` | Toast notification system (max 1 visible, auto-dismiss) |

### 10.6 Utility Modules

| Module | Purpose | Key Exports |
|--------|---------|-------------|
| **activityLog.ts** | Audit logging | `logActivity()` — inserts to `activity_log` |
| **gstExport.ts** | GST return Excel files | `exportGSTR1()`, `exportGSTR2()`, `exportGSTR3B()` — B2B, B2CL, B2CS, CDNR, HSN sheets |
| **importExport.ts** | Bulk data import/export | Party/item/invoice/payment Excel/CSV import & export with templates |
| **invoiceCalc.ts** | Invoice math engine | `calcLineAmount()`, `solveRateFromAmount()`, `computeInvoiceTotals()` — supports tax-inclusive/exclusive, extra charges, GST split |
| **invoicePdf.ts** | Invoice PDF generation | Thermal 58mm/80mm, A5 (single/2-page), A4 formats; UPI QR codes; amount in words (Lakh/Crore) |
| **paymentPdf.ts** | Payment receipt generation | Thermal/A5/A4 receipt and voucher formats |
| **razorpay.ts** | Payment SDK | `loadRazorpay()`, `openRazorpay()` — lazy-loads Razorpay script and opens checkout modal |
| **reportArtifacts.ts** | Shareable report artifacts | `buildReportArtifact()` — generates HTML + XLSX base64 for all 14 report types |
| **reportExport.ts** | Direct report export | 28 export functions (PDF + Excel for each report type) |
| **reportGenerators.ts** | Report data computation | Pure functions: `getSaleReportData()`, `getProfitLossData()`, `getStockSummaryData()`, etc. |

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
- [x] 5 invoice formats: A4, A5, A5 2-page, Thermal 58mm, Thermal 80mm
- [x] Extra charges (TCS, Freight, Packing) and extra discounts (Trade, Loyalty)
- [x] Pre-tax and post-tax charge/discount application
- [x] Auto round-off with manual override
- [x] UPI QR code on invoices
- [x] Amount in words (Indian numbering: Lakh, Crore)
- [x] Reverse calculations (edit amount → solve rate, edit GST ₹ → solve GST rate)
- [x] Multi-tab simultaneous invoice creation

### Voice Input
- [x] Web Speech API integration (Chrome/Edge)
- [x] Fuzzy item name matching (exact → endsWith → contains → word match)
- [x] Multi-item voice commands ("5 kg rice and 2 cement")
- [x] Quantity + unit recognition ("5 kg", "dozen", etc.)

### Inventory
- [x] Products and services with categories
- [x] Dual unit support (e.g., 1 box = 12 pcs)
- [x] Opening stock with as-of date
- [x] Min stock quantity alerts
- [x] Stock auto-adjustment on invoice creation
- [x] Active/inactive item toggle
- [x] Item images

### Import/Export
- [x] Excel and CSV import/export for parties, items, invoices, payments
- [x] Template downloads for imports
- [x] Import validation with error reporting
- [x] Full company data backup (.bkp JSON format)
- [x] FK-aware restore with batch upserts (500 per batch)

### Multi-Company
- [x] Create and manage multiple companies
- [x] Switch between companies via CompanySwitcher
- [x] Per-company data isolation (RLS enforced)
- [x] Company-specific business profiles

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

### Reports (14 Types)
- [x] Sale Report, Purchase Report
- [x] Day Book, All Transactions
- [x] Profit & Loss, Party-wise P&L, Item-wise P&L
- [x] Party Statement, All Parties Report
- [x] Stock Summary, Stock Detail
- [x] GSTR-1, GSTR-2, GSTR-3B
- [x] PDF and Excel export for all reports
- [x] Share reports to party portal users

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

## License

This project is proprietary software. All rights reserved.
