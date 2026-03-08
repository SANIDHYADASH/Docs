# SplitMate v2 — Release Notes

> Features and enhancements delivered in the v2 development cycle.

---

## 🗓️ Date Selection for Expenses, Budgets & Income

### Problem
Users could not backdate or future-date any financial entries. All expenses, budgets, and income entries were locked to the current timestamp.

### Solution
- Added **calendar date picker** to the **Add Expense**, **Create Budget**, and **Add Income** dialogs
- Defaults to today's date but allows selecting any past or future date
- Edit mode pre-populates the calendar with the existing date
- Uses `react-day-picker` with a Popover-based UI consistent with the app's design system

### Affected Components
- `AddExpenseDialog.tsx` — New date field with calendar picker
- `CreateBudgetDialog.tsx` — New date field with calendar picker
- `AddIncomeDialog.tsx` — New date field with calendar picker

---

## 🧭 Dashboard Navigation Restructure

### Problem
Navigation icons were placed below the summary cards, requiring users to scroll. When viewing budgets, the summary cards (Total Spent / Members) consumed valuable screen space without adding context.

### Solution
- **Moved icon navigation bar** from below cards to **above cards** (directly below the navbar)
- **Summary cards auto-hide** when the Budget section is active, giving budgets full top-of-screen real estate
- Smoother section switching with reduced scrolling

### Affected Components
- `GroupDashboard.tsx` — Reordered layout: navbar → icon tabs → conditional cards → content

---

## 🔍 Consolidated Budget Filters with Date Range

### Problem
Budget view lacked date-based filtering. Filter/sort/search controls were scattered, cluttering the UI especially on mobile.

### Solution
- **Single filter icon** (sliders) placed next to the "New Budget" button opens a responsive popover
- **All filters consolidated** into one panel:
  - Text search
  - Budget type filter (All / Personal / Group)
  - Member filter
  - Sort options (Newest / Oldest / Amount High→Low / Amount Low→High)
  - View mode toggle (Combined / Separate)
  - **Date range filter** (From / To with calendar pickers)
- **Same-day filtering**: Selecting identical From and To dates correctly shows all transactions for that specific day
- **Active filter badge**: Shows count of currently active filters on the icon
- **Clear all filters** button for quick reset
- Fully responsive on mobile and desktop

### Affected Components
- `BudgetView.tsx` — Complete filter UI overhaul with date range support

---

## Summary of All Changes

| Feature | Type | Description |
|---------|------|-------------|
| Expense date picker | Enhancement | Calendar-based date selection for adding/editing expenses |
| Budget date picker | Enhancement | Calendar-based date selection for creating/editing budgets |
| Income date picker | Enhancement | Calendar-based date selection for adding/editing income entries |
| Navigation restructure | UX Improvement | Icon tabs moved above cards for faster access |
| Budget-mode card hiding | UX Improvement | Summary cards hidden in budget view for more space |
| Consolidated filter popover | New Feature | All budget filters in a single responsive popover |
| Date range filtering | New Feature | Filter budget transactions by date range |
| Same-day date filtering | New Feature | From = To date shows all entries for that day |
| Active filter badge | Enhancement | Visual count of applied filters |
| Clear all filters | Enhancement | One-click filter reset |