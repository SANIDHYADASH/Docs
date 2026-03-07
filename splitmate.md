# SplitMate — Group Expense Splitting Application

> A full-stack web application for splitting expenses among friends and groups. Built with React, TypeScript, Tailwind CSS, and Supabase.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Tech Stack](#tech-stack)
4. [Database Schema](#database-schema)
5. [Application Flow](#application-flow)
6. [Frontend Structure](#frontend-structure)
7. [Backend (Edge Functions / API)](#backend-edge-functions--api)
8. [Authentication](#authentication)
9. [Core Features](#core-features)
10. [Split Calculation Algorithm](#split-calculation-algorithm)
11. [API Reference](#api-reference)
12. [Security & RLS Policies](#security--rls-policies)
13. [Deployment](#deployment)

---

## Overview

**SplitMate** allows users to:

- Create expense groups and invite friends via a 4-digit code
- Add, edit, and delete expenses with optional categories
- Automatically calculate who owes whom with a minimal-transaction settlement algorithm
- View activity history for full audit trail
- Share read-only expense summaries via a public link (code + password protected)
- Export expense reports as PDF

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENTS                             │
│                                                         │
│  ┌──────────┐   ┌──────────────┐   ┌────────────────┐   │
│  │ React    │   │ Android/iOS  │   │ Shared View    │   │
│  │ Web App  │   │ Native Apps  │   │ (Public Page)  │   │
│  └────┬─────┘   └──────┬───────┘   └───────┬────────┘   │
│       │                │                    │           │
└───────┼────────────────┼────────────────────┼───────────┘
        │                │                    │
        ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────┐
│                   SUPABASE BACKEND                      │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Edge Functions (REST API)           │   │
│  │                                                  │   │
│  │  api-groups     api-expenses    api-members      │   │
│  │  api-settlements api-activity  api-share-settings│   │
│  │  api-shared-view (PUBLIC)                        │   │
│  └────────────────────┬─────────────────────────────┘   │
│                       │                                 │
│  ┌────────────────────▼─────────────────────────────┐   │
│  │              PostgreSQL Database                 │   │
│  │                                                  │   │
│  │  expense_groups │ group_members │ expenses       │   │
│  │  activity_log   │ profiles                       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Supabase Auth                       │   │
│  │  Email/Password signup & login                   │   │
│  │  JWT token management                            │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 18, TypeScript, Vite |
| **Styling** | Tailwind CSS, shadcn/ui component library |
| **State Management** | React hooks, TanStack React Query |
| **Routing** | React Router v6 |
| **Backend** | Supabase (PostgreSQL + Auth + Edge Functions) |
| **PDF Export** | jsPDF + jspdf-autotable |
| **Edge Functions** | Deno (TypeScript), deployed on Supabase |

---

## Database Schema

### Entity Relationship Diagram

```
┌──────────────────┐       ┌──────────────────────────┐
│    profiles      │       │   expense_groups         │
├──────────────────┤       ├──────────────────────────┤
│ id (PK, uuid)    │       │ id (PK, uuid)            │
│ display_name     │       │ name                     │
│ created_at       │       │ code (4-digit unique)    │
└──────────────────┘       │ created_by (user uuid)   │
       │                   │ share_password (nullable)│
       │                   │ created_at               │
       │                   └──────────┬───────────────┘
       │                              │
       │                              │ 1:N
       │                              ▼
       │                   ┌──────────────────────┐
       │                   │   group_members      │
       │                   ├──────────────────────┤
       │                   │ id (PK, uuid)        │
       └──────────────────►│ user_id (nullable)   │
                           │ group_id (FK)        │
                           │ name                 │
                           │ created_at           │
                           └──────────┬───────────┘
                                      │
                          ┌───────────┴───────────┐
                          │                       │
                          ▼                       ▼
               ┌──────────────────┐    ┌──────────────────┐
               │    expenses      │    │   activity_log   │
               ├──────────────────┤    ├──────────────────┤
               │ id (PK, uuid)    │    │ id (PK, uuid)    │
               │ group_id (FK)    │    │ group_id (FK)    │
               │ paid_by (FK →    │    │ member_id (FK)   │
               │   group_members) │    │ action           │
               │ amount (numeric) │    │ details          │
               │ description      │    │ created_at       │
               │ split_among[]    │    └──────────────────┘
               │ category         │
               │ created_at       │
               │ updated_at       │
               └──────────────────┘
```

### Table Details

#### `profiles`
Auto-created on user signup via a database trigger (`handle_new_user`). Stores the user's display name.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid (PK) | References `auth.users.id` |
| `display_name` | text | User's chosen display name |
| `created_at` | timestamptz | Auto-set |

#### `expense_groups`
Each group has a unique 4-digit code for easy sharing.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid (PK) | Auto-generated |
| `name` | text | Group name (e.g., "Weekend Trip") |
| `code` | text | 4-digit invite code |
| `created_by` | uuid | The user who created the group |
| `share_password` | text (nullable) | If set, enables public read-only sharing |
| `created_at` | timestamptz | Auto-set |

#### `group_members`
Links users to groups. A user can be in multiple groups.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid (PK) | Auto-generated |
| `group_id` | uuid (FK) | References `expense_groups.id` |
| `user_id` | uuid (nullable) | References `auth.users.id` |
| `name` | text | Display name in this group |
| `created_at` | timestamptz | Auto-set |

#### `expenses`
Each expense records who paid, how much, and who it's split among.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid (PK) | Auto-generated |
| `group_id` | uuid (FK) | References `expense_groups.id` |
| `paid_by` | uuid (FK) | References `group_members.id` |
| `amount` | numeric | Expense amount |
| `description` | text | What the expense was for |
| `split_among` | uuid[] | Array of member IDs to split among (empty = all) |
| `category` | text (nullable) | Category tag (food, transport, etc.) |
| `created_at` | timestamptz | Auto-set |
| `updated_at` | timestamptz | Auto-updated via trigger |

#### `activity_log`
Immutable audit trail for all group actions.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid (PK) | Auto-generated |
| `group_id` | uuid (FK) | References `expense_groups.id` |
| `member_id` | uuid (FK) | References `group_members.id` |
| `action` | text | Action type enum (see below) |
| `details` | text (nullable) | Human-readable description |
| `created_at` | timestamptz | Auto-set |

**Action Types:**
- `GROUP_CREATED` — Group was created
- `MEMBER_JOINED` — A new member joined
- `EXPENSE_ADDED` / `EXPENSE_CREATED` — Expense was added
- `EXPENSE_UPDATED` — Expense was edited
- `EXPENSE_DELETED` — Expense was removed

---

## Application Flow

### Complete User Journey

```
┌─────────────┐
│   App Load  │
└──────┬──────┘
       │
       ▼
┌──────────────┐     No      ┌──────────────┐
│ Auth Session ├────────────►│  Auth Page   │
│   exists?    │             │ Login/Signup │
└──────┬───────┘             └──────┬───────┘
       │ Yes                        │ Success
       ▼                            ▼
┌──────────────────────────────────────────┐
│           JoinGroup Screen               │
│                                          │
│  ┌────────────┐  ┌────────────────────┐  │
│  │ My Groups  │  │ Create / Join Tab  │  │
│  │ (list)     │  │                    │  │
│  └─────┬──────┘  └────────┬───────────┘  │
│        │                  │              │
│    Tap group      Create group (→ code)  │
│        │          Join via 4-digit code  │
│        │                  │              │
└────────┼──────────────────┼──────────────┘
         │                  │
         ▼                  ▼
┌──────────────────────────────────────────┐
│          GroupDashboard                  │
│                                          │
│  Header: Group name, code, user info     │
│  Stats:  Total | Expenses | Members      │
│                                          │
│  ┌──────────┬─────────┬────────────┐     │
│  │ Expenses │  Split  │  History   │     │
│  │  Tab     │  Tab    │   Tab      │     │
│  ├──────────┼─────────┼────────────┤     │
│  │ List of  │ Balance │ Activity   │     │
│  │ expenses │ per     │ log with   │     │
│  │ with     │ member  │ timestamps │     │
│  │ edit/    │         │            │     │
│  │ delete   │ Who     │            │     │
│  │          │ pays    │            │     │
│  │ Add new  │ whom    │            │     │
│  │ expense  │ (min    │            │     │
│  │ button   │ txns)   │            │     │
│  └──────────┴─────────┴────────────┘     │
│                                          │
│  Actions: Add Expense | Export PDF       │
│           Share Settings | Manage Members│
│           Close Group (creator only)     │
└──────────────────────────────────────────┘
```

### Shared View Flow (Public, No Auth)

```
┌───────────────┐
│  /view page   │
└───────┬───────┘
        │
        ▼
┌───────────────────┐
│ Enter 4-digit     │
│ group code +      │
│ share password    │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐     ┌────────────────────┐
│ Verify against    │ No  │ Error:             │
│ expense_groups    ├────►│ "Group not found"  │
│ table             │     │ or "Wrong password │
└───────┬───────────┘     └────────────────────┘
        │ Yes
        ▼
┌───────────────────────────────────┐
│ Read-only dashboard               │
│ - Expense list                    │
│ - Split/settlement view           │
│ - PDF export                      │
│ (No edit/delete/add capabilities) │
└───────────────────────────────────┘
```

### Expense Creation Flow

```
┌──────────────────┐
│ Click "Add       │
│ Expense" button  │
└───────┬──────────┘
        │
        ▼
┌──────────────────────────────────────┐
│ AddExpenseDialog                     │
│                                      │
│  Description:  [optional text]       │
│  Amount (₹):   [required number]     │
│  Category:     [dropdown, optional]  │
│  Paid by:      [dropdown - members]  │
│  Split among:  [checkbox list]       │
│                                      │
│  Preview: "Each person pays: ₹X.XX"  │
│                                      │
│  [Add Expense] button                │
└───────┬──────────────────────────────┘
        │
        ▼
┌──────────────────┐
│ Insert into      │
│ expenses table   │
│ + activity_log   │
└───────┬──────────┘
        │
        ▼
┌──────────────────┐
│ Refresh data &   │
│ recalculate      │
│ balances         │
└──────────────────┘
```

---

## Frontend Structure

### Routes

| Route | Component | Auth Required | Description |
|-------|-----------|---------------|-------------|
| `/` | `Index` → `Auth` or `JoinGroup` or `GroupDashboard` | Yes | Main app flow |
| `/view` | `SharedView` | No | Public read-only expense viewer |
| `*` | `NotFound` | No | 404 page |

### Component Tree

```
App
├── Index (/)
│   ├── Auth (when not logged in)
│   │   ├── Login Tab
│   │   └── Signup Tab
│   │
│   ├── JoinGroup (when logged in, no active group)
│   │   ├── My Groups List
│   │   ├── Create Group Tab
│   │   └── Join Group Tab
│   │
│   └── GroupDashboard (when group is active)
│       ├── Header (group name, code, actions)
│       ├── Stats Cards (total, count, members)
│       ├── Tabs
│       │   ├── ExpenseList
│       │   ├── SettlementView
│       │   └── ActivityHistory
│       ├── AddExpenseDialog
│       ├── ShareSettings (dialog)
│       └── MemberManagement (dialog, creator only)
│
└── SharedView (/view)
    ├── Code + Password Input
    └── Read-only Dashboard
        ├── ExpenseList (readOnly mode)
        └── SettlementView
```

### Key Hooks

| Hook | File | Purpose |
|------|------|---------|
| `useAuth` | `src/hooks/useAuth.ts` | Manages auth state, session restoration, display name |
| `useGroupSession` | `src/hooks/useGroupSession.ts` | Persists active group to localStorage |

### Utility Libraries

| File | Purpose |
|------|---------|
| `src/lib/splitCalculator.ts` | Balance calculation & greedy settlement algorithm |
| `src/lib/pdfExport.ts` | Generates PDF reports with expenses, balances, settlements |
| `src/lib/categories.ts` | Expense category definitions with emoji icons |

---

## Backend (Edge Functions / API)

All business logic is exposed via **7 Supabase Edge Functions** deployed as REST APIs. These are designed to be consumed by both the web app and native mobile apps (Android/iOS).

### Edge Function Overview

| Function | Auth | Method(s) | Purpose |
|----------|------|-----------|---------|
| `api-groups` | JWT Required | GET, POST, DELETE | Group CRUD + join |
| `api-expenses` | JWT Required | GET, POST, PUT, DELETE | Expense CRUD |
| `api-members` | JWT Required | GET, PUT, DELETE | Member management |
| `api-settlements` | JWT Required | GET | Balance & settlement calculation |
| `api-activity` | JWT Required | GET | Activity log retrieval |
| `api-share-settings` | JWT Required | GET, PUT | Sharing toggle & password |
| `api-shared-view` | **Public** | POST | Read-only view with code+password |

### Base URL

```
https://tzvgmjszxxmtewmwacpv.supabase.co/functions/v1
```

---

## Authentication

SplitMate uses **email/password authentication** via Supabase Auth.

### Signup Flow

```
User fills: name, email, password
        │
        ▼
supabase.auth.signUp({
  email, password,
  options: { data: { display_name } }
})
        │
        ▼
Database trigger: handle_new_user()
  → Inserts row into profiles table
  → display_name from user metadata
        │
        ▼
User is logged in with JWT token
```

### Session Management

1. On app load, `useAuth` calls `supabase.auth.getSession()` to restore session from localStorage
2. `onAuthStateChange` listener handles subsequent login/logout events
3. JWT tokens are automatically refreshed by the Supabase client
4. The `loading` state prevents flash of auth screen during session restoration

### API Authentication Headers

```
Authorization: Bearer <access_token>
apikey: <SUPABASE_ANON_KEY>
Content-Type: application/json
```

---

## Core Features

### 1. Group Management
- **Create Group**: Generates a random 4-digit code, adds creator as first member
- **Join Group**: Enter 4-digit code to join; idempotent (rejoining returns existing membership)
- **My Groups**: Lists all groups the user belongs to with member counts
- **Close Group** (creator only): Cascading delete of all expenses, activity logs, members, then group

### 2. Expense Management
- **Add Expense**: Description (optional), amount (required), category (optional), paid by (select member), split among (checkbox list)
- **Edit Expense**: Full in-place editing with pre-populated form
- **Delete Expense**: With confirmation, logs action
- **Category Support**: 8 predefined categories — Food, Transport, Accommodation, Entertainment, Shopping, Utilities, Health, Other

### 3. Split & Settlement
- **Real-time Balances**: Each member's total paid vs total owed
- **Greedy Settlement Algorithm**: Minimizes the number of transactions needed to settle all debts
- **Visual Indicators**: Green for creditors (owed money), red for debtors (owes money)

### 4. Activity History
- Immutable audit trail of all group actions
- Shows who did what and when
- Limited to 50 most recent entries
- Action types: GROUP_CREATED, MEMBER_JOINED, EXPENSE_ADDED/UPDATED/DELETED

### 5. Sharing (Public View)
- Group creator can enable sharing by setting a password
- Anyone with the 4-digit code + password can view expenses at `/view`
- Read-only access: no editing, no deletion
- Accessible without authentication

### 6. PDF Export
- Generates a multi-section PDF report:
  - **Section 1**: Expense table (description, amount, paid by, split among, date)
  - **Section 2**: Member balances (paid, owed, net balance)
  - **Section 3**: Settlement instructions (who pays whom, how much)
- Available in both authenticated dashboard and shared view

### 7. Member Management (Creator Only)
- **Rename Members**: Edit any member's display name
- **Remove Members**: Removes member and all their expenses (with confirmation)
- Creator cannot remove themselves

---

## Split Calculation Algorithm

Located in `src/lib/splitCalculator.ts`.

### Step 1: Calculate Balances

```
For each expense:
  1. Full amount credited to the payer (totalPaid += amount)
  2. Equal share (amount / split_count) debited from each member in split_among
  3. If split_among is empty, all group members are included

Balance = totalPaid - totalOwed
  Positive → others owe you money (creditor)
  Negative → you owe money (debtor)
  Zero     → settled
```

### Step 2: Calculate Settlements (Greedy Algorithm)

```
1. Separate members into debtors (balance < 0) and creditors (balance > 0)
2. Sort debtors ascending (largest debt first)
3. Sort creditors descending (largest credit first)
4. Use two-pointer approach:
   - Match largest debtor with largest creditor
   - Transfer amount = min(|debtor.balance|, creditor.balance)
   - Adjust both balances
   - Move pointer when balance reaches zero
5. Continue until all settled

This produces minimal number of transactions.
```

**Example:**
```
Members: Alice (+$30), Bob (-$20), Charlie (-$10)

Settlement:
  Bob    → Alice:  $20
  Charlie → Alice: $10
```

---

## API Reference

### Authentication Endpoints

#### Sign Up
```http
POST /auth/v1/signup
Host: tzvgmjszxxmtewmwacpv.supabase.co
apikey: <SUPABASE_ANON_KEY>
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword",
  "data": { "display_name": "John Doe" }
}

→ 200: { "access_token": "...", "refresh_token": "...", "user": {...} }
```

#### Sign In
```http
POST /auth/v1/token?grant_type=password
Host: tzvgmjszxxmtewmwacpv.supabase.co
apikey: <SUPABASE_ANON_KEY>
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword"
}

→ 200: { "access_token": "...", "refresh_token": "...", "user": {...} }
```

#### Refresh Token
```http
POST /auth/v1/token?grant_type=refresh_token
Host: tzvgmjszxxmtewmwacpv.supabase.co
apikey: <SUPABASE_ANON_KEY>
Content-Type: application/json

{ "refresh_token": "xxx" }
```

---

### Groups API — `POST|GET|DELETE /api-groups`

All endpoints require `Authorization: Bearer <token>` header.

#### List My Groups
```http
GET /functions/v1/api-groups

→ 200:
{
  "groups": [{
    "group_id": "uuid",
    "member_id": "uuid",
    "group_name": "Weekend Trip",
    "group_code": "1234",
    "is_creator": true,
    "member_count": 4,
    "created_at": "2026-03-07T10:00:00Z"
  }]
}
```

#### Create Group
```http
POST /functions/v1/api-groups
{ "name": "Weekend Trip" }

→ 201:
{
  "group_id": "uuid",
  "member_id": "uuid",
  "group_name": "Weekend Trip",
  "group_code": "1234",
  "member_name": "John Doe"
}
```

#### Join Group by Code
```http
POST /functions/v1/api-groups/join
{ "code": "1234" }

→ 201:
{
  "group_id": "uuid",
  "member_id": "uuid",
  "group_name": "Weekend Trip",
  "group_code": "1234",
  "member_name": "John Doe",
  "already_member": false
}
```

#### Delete Group (Creator Only)
```http
DELETE /functions/v1/api-groups/<group_id>

→ 200: { "message": "Group and all associated data deleted" }
```

---

### Expenses API — `GET|POST|PUT|DELETE /api-expenses`

#### List Expenses
```http
GET /functions/v1/api-expenses?group_id=<uuid>

→ 200:
{
  "expenses": [{
    "id": "uuid",
    "group_id": "uuid",
    "paid_by": "uuid",
    "description": "Dinner",
    "amount": 1500.00,
    "split_among": ["member_id_1", "member_id_2"],
    "category": "food",
    "created_at": "2026-03-07T10:00:00Z",
    "updated_at": "2026-03-07T10:00:00Z"
  }]
}
```

#### Create Expense
```http
POST /functions/v1/api-expenses
{
  "group_id": "uuid",
  "paid_by": "uuid (member_id)",
  "amount": 1500.00,
  "description": "Dinner",
  "split_among": ["member_id_1", "member_id_2"],
  "category": "food"
}

→ 201: { "expense": {...} }
```

#### Update Expense
```http
PUT /functions/v1/api-expenses/<expense_id>
{
  "amount": 2000.00,
  "description": "Updated dinner",
  "paid_by": "uuid",
  "split_among": ["member_id_1", "member_id_2"],
  "category": "food",
  "member_id": "uuid"  // for activity logging
}

→ 200: { "expense": {...} }
```

#### Delete Expense
```http
DELETE /functions/v1/api-expenses/<expense_id>?member_id=<uuid>

→ 200: { "message": "Expense deleted" }
```

---

### Members API — `GET|PUT|DELETE /api-members`

#### List Members
```http
GET /functions/v1/api-members?group_id=<uuid>

→ 200:
{
  "members": [{
    "id": "uuid",
    "name": "John Doe",
    "user_id": "uuid or null",
    "created_at": "2026-03-07T10:00:00Z"
  }]
}
```

#### Rename Member (Creator Only)
```http
PUT /functions/v1/api-members/<member_id>
{ "name": "New Name" }

→ 200: { "member": {...} }
```

#### Remove Member (Creator Only)
```http
DELETE /functions/v1/api-members/<member_id>

→ 200: { "message": "Member \"John\" removed" }
```

---

### Settlements API — `GET /api-settlements`

#### Get Balances & Settlements
```http
GET /functions/v1/api-settlements?group_id=<uuid>

→ 200:
{
  "balances": [{
    "memberId": "uuid",
    "memberName": "John",
    "totalPaid": 3000.00,
    "totalOwed": 1500.00,
    "balance": 1500.00
  }],
  "settlements": [{
    "from": "uuid",
    "from_name": "Alice",
    "to": "uuid",
    "to_name": "John",
    "amount": 750.00
  }]
}
```

---

### Activity API — `GET /api-activity`

#### Get Activity History
```http
GET /functions/v1/api-activity?group_id=<uuid>&limit=50

→ 200:
{
  "activities": [{
    "id": "uuid",
    "action": "EXPENSE_ADDED",
    "details": "John added \"Dinner\" (₹1500)",
    "member_id": "uuid",
    "created_at": "2026-03-07T10:00:00Z"
  }]
}
```

---

### Share Settings API — `GET|PUT /api-share-settings`

#### Get Share Settings
```http
GET /functions/v1/api-share-settings?group_id=<uuid>

→ 200: { "sharing_enabled": true, "is_creator": true }
```

#### Update Share Settings (Creator Only)
```http
PUT /functions/v1/api-share-settings?group_id=<uuid>
{ "share_password": "mypassword" }   // null or empty to disable

→ 200: { "message": "Sharing enabled", "sharing_enabled": true }
```

---

### Shared View API — `POST /api-shared-view` (PUBLIC)

**No authentication required.** Only requires `apikey` header.

```http
POST /functions/v1/api-shared-view
apikey: <SUPABASE_ANON_KEY>
Content-Type: application/json

{
  "code": "1234",
  "password": "sharepassword"
}

→ 200:
{
  "group_name": "Weekend Trip",
  "group_code": "1234",
  "members": [{ "id": "uuid", "name": "John" }],
  "expenses": [{ "id": "uuid", "paid_by": "uuid", "description": "Dinner", "amount": 1500, ... }],
  "balances": [{ "memberId": "uuid", "memberName": "John", "totalPaid": 3000, "totalOwed": 1500, "balance": 1500 }],
  "total_expenses": 5000.00
}
```

---

### Error Response Format

All errors follow a consistent structure:

```json
{ "error": "Description of the error" }
```

| Status Code | Meaning |
|-------------|---------|
| `400` | Bad request / validation error |
| `401` | Unauthorized / incorrect password |
| `403` | Forbidden (not creator / sharing not enabled) |
| `404` | Resource not found |
| `405` | Method not allowed |
| `500` | Internal server error |

---

### Expense Categories

| Value | Label | Icon |
|-------|-------|------|
| `food` | Food | 🍕 |
| `transport` | Transport | 🚗 |
| `accommodation` | Accommodation | 🏨 |
| `entertainment` | Entertainment | 🎬 |
| `shopping` | Shopping | 🛍️ |
| `utilities` | Utilities | 💡 |
| `health` | Health | 🏥 |
| `other` | Other | 📦 |

---

## Security & RLS Policies

All tables have **Row-Level Security (RLS)** enabled with the following policies:

| Table | Action | Policy |
|-------|--------|--------|
| `profiles` | SELECT | Own profile only (`auth.uid() = id`) |
| `profiles` | INSERT | Own profile only |
| `profiles` | UPDATE | Own profile only |
| `expense_groups` | SELECT | All authenticated users |
| `expense_groups` | INSERT | All authenticated users |
| `expense_groups` | UPDATE | All authenticated users |
| `expense_groups` | DELETE | Creator only (`created_by = auth.uid()`) |
| `group_members` | SELECT | All authenticated users |
| `group_members` | INSERT | All authenticated users |
| `group_members` | UPDATE/DELETE | Group creator only (via FK check) |
| `expenses` | SELECT/INSERT/UPDATE/DELETE | All authenticated users |
| `activity_log` | SELECT/INSERT | All authenticated users |
| `activity_log` | UPDATE/DELETE | Not allowed |

### Database Triggers

| Trigger | Table | Function | Purpose |
|---------|-------|----------|---------|
| `handle_new_user` | `auth.users` (on INSERT) | `handle_new_user()` | Auto-creates profile row |
| `update_updated_at` | `expenses` (on UPDATE) | `update_updated_at_column()` | Auto-updates `updated_at` |

---

## Deployment


### Edge Functions
Edge functions are **automatically deployed** when code is pushed. No manual deployment needed.

### Native App Development

For building Android/iOS apps against these APIs, use the native Supabase SDKs:

| Platform | SDK | Auth | API Calls |
|----------|-----|------|-----------|
| **Android (Kotlin)** | `io.github.jan-tennert.supabase:gotrue-kt` | Built-in | `supabase.functions.invoke("api-groups")` |
| **iOS (Swift)** | `supabase-swift` (SPM) | Built-in | `supabase.functions.invoke("api-groups")` |
| **Flutter** | `supabase_flutter` | Built-in | `Supabase.instance.client.functions.invoke("api-groups")` |
| **React Native** | `@supabase/supabase-js` | Built-in | Same as web |

### Realtime Subscriptions

For real-time updates, connect to:
```
wss://tzvgmjszxxmtewmwacpv.supabase.co/realtime/v1/websocket?apikey=<SUPABASE_ANON_KEY>
```

Subscribe to `expenses` and `group_members` tables filtered by `group_id`.

---

## Project Structure

```
├── src/
│   ├── App.tsx                    # Root component with routing
│   ├── main.tsx                   # Entry point
│   ├── index.css                  # Global styles & Tailwind config
│   ├── pages/
│   │   ├── Index.tsx              # Main page (auth → groups → dashboard)
│   │   ├── Auth.tsx               # Login/signup page
│   │   ├── SharedView.tsx         # Public read-only expense viewer
│   │   └── NotFound.tsx           # 404 page
│   ├── components/
│   │   ├── JoinGroup.tsx          # Group listing, creation, joining
│   │   ├── GroupDashboard.tsx      # Main dashboard with tabs
│   │   ├── ExpenseList.tsx        # Expense cards with edit/delete
│   │   ├── AddExpenseDialog.tsx   # Create/edit expense modal
│   │   ├── SettlementView.tsx     # Balances & settlement display
│   │   ├── ActivityHistory.tsx    # Activity log timeline
│   │   ├── ShareSettings.tsx      # Sharing toggle & password
│   │   ├── MemberManagement.tsx   # Rename/remove members
│   │   └── ui/                    # shadcn/ui components
│   ├── hooks/
│   │   ├── useAuth.ts             # Authentication state management
│   │   └── useGroupSession.ts     # Group session persistence
│   ├── lib/
│   │   ├── splitCalculator.ts     # Balance & settlement algorithms
│   │   ├── pdfExport.ts           # PDF report generation
│   │   ├── categories.ts          # Expense category definitions
│   │   └── utils.ts               # Tailwind utility helpers
│   └── integrations/supabase/
│       ├── client.ts              # Supabase client (auto-generated)
│       └── types.ts               # Database types (auto-generated)
├── supabase/
│   ├── config.toml                # Supabase project configuration
│   └── functions/
│       ├── api-groups/index.ts
│       ├── api-expenses/index.ts
│       ├── api-members/index.ts
│       ├── api-settlements/index.ts
│       ├── api-activity/index.ts
│       ├── api-share-settings/index.ts
│       └── api-shared-view/index.ts
└── API_DOCUMENTATION.md           # Standalone API reference
```
