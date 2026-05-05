# PrintOps 360 — Project Context

A single-file React prototype of a 3D-print business operations suite. Built as a high-fidelity demo for a small print studio (default workspace: "Layla's Print Studio"). Hosted on GitHub Pages from `main`.

- **Repo:** `noureldin0007/printops-360`
- **Live site:** `https://noureldin0007.github.io/printops-360/`
- **Default branch deployed:** `main`
- **Active feature branch:** `claude/financial-analytics-time-filter-QzOwV`

---

## Tech stack

Everything lives in a **single file**: `index.html`.

- **React 18.3** (UMD build) loaded via `unpkg`
- **ReactDOM 18.3** (UMD)
- **@babel/standalone 7.29** for in-browser JSX transpilation
- **Tailwind CDN** (`cdn.tailwindcss.com`) — used for layout primitives (`flex`, `grid`, `gap-*`, `mb-*`)
- **Inter** font family + **Anton** display font (intro screen) from Google Fonts
- **No build step. No package.json. No backend.** Open `index.html` in a browser and it runs.
- All icons are inline SVG components defined in `window.LIcons` via two factory helpers (`mk`, `mkRaw`). Lucide-style outline icons.

> **Important constraint:** because everything is in one file and Babel is in-browser, any JS syntax error in the file will render the **entire page blank**. Past breakage: a duplicate `const products = [...]` declaration. Always grep for duplicate identifiers when refactoring.

---

## File layout

```
printops-360/
├── index.html         # ENTIRE app (~2400+ lines)
└── PROJECT_CONTEXT.md # this file
```

There is no README in the repo. There are no other source files.

### Top-of-file structure (`index.html`)

| Lines (approx.) | Section |
|---|---|
| 1–14    | `<head>`: meta, fonts, React/Babel CDNs |
| 14–108  | `<script>` defining `window.LIcons` (icon factory + all icons) |
| 110–181 | `<style>` block with custom keyframes (`twinkle`, `drift`, `introTitleIn`, `glowPulse`, `pulseDot`, etc.) and helper classes |
| 183–185 | `<div id="root">` mount point |
| 188+    | `<script type="text/babel">` containing all React |

Inside the React script, in order:
1. Hooks alias: `const { useState, useMemo, useEffect, useRef } = React;`
2. Helpers: `bootWhenReady`, `fmtUSD`, `fmtUSDInt`
3. `Sidebar`, `PageHeader`, `Card`, `KPI`, `StatusPill`, `Modal`, `Select`
4. **SCREEN 1 — Dashboard** (`function Dashboard`)
5. **SCREEN 2 — Pricing Calculator** (`function PricingCalculator`)
6. **SCREEN 3 — Inventory** (`function Inventory`, with top-level `INVENTORY` seed)
7. **SCREEN 4 — Financial Analytics** (`function FinancialAnalytics`)
8. **SCREEN 5 — Reports** (`function Reports`)
9. **SCREEN 6 — Settings** (`function Settings`, with `SETTINGS_KEY` and `DEFAULT_SETTINGS`)
10. **Intro Screen** (`function IntroScreen`) — animated splash on first load
11. **`function App`** — owns `screen` state and renders sidebar + active screen

---

## Brand & design system

### Color palette

| Token | Hex | Use |
|---|---|---|
| Brand orange | `#D97757` | Primary accent, CTAs, active KPIs |
| Brand orange dark | `#C25E3F` | Hover state for orange |
| Brand orange light | `#F0A584` | Gradient lighter stop |
| Brand orange darkest | `#A8442C` | Donut/expense category |
| Sidebar / hero dark | `#262624` | Sidebar bg, dashboard hero card |
| Sidebar dark variant | `#1A1A18` / `#1F1F1D` | Hero gradient stop |
| Page background | `#FAF9F5` | App background, table headers, input fills |
| Card background | `#FFFFFF` | Cards |
| Border | `#E5E7EB` | Card borders, input borders |
| Subtle border | `#F3F4F6` | Row dividers |
| Text primary | `#111827` | Headlines |
| Text secondary | `#374151` | Body |
| Text muted | `#6B7280` | Subtitles, table sub-text |
| Text faint | `#9CA3AF` | Axis labels, placeholder helpers |
| Success green | `#16A34A` (text), `#DCFCE7` (bg) | Profit, "In Stock" |
| Warn amber | `#D97706` / `#F59E0B` (text), `#FEF3C7` (bg) | "Low" |
| Critical red | `#DC2626` (text), `#FEE2E2` (bg) | "Critical", "Out of Stock" |
| Info teal | `#0D9488` | Donut category |
| Info indigo | `#4338CA` (text), `#E0E7FF` (bg) | "Design" stage |
| Pink | `#EC4899` | Donut category |

### Typography

- **Body:** Inter 400/500/600/700
- **Display (intro):** Anton
- Tabular numerals via `.num-tabular { font-variant-numeric: tabular-nums; }` for any monetary or quantity figure
- KPI display sizes: 28–64px, font-weight 700–800, letter-spacing tightened (`-0.5` to `-2.2`)

### Layout primitives

- **Border radius:** `8` (inputs/buttons), `10–14` (small cards), `16–18` (bento hero cards)
- **Shadow ladder:**
  - Subtle card: `0 1px 2px rgba(16,24,40,0.04), 0 1px 3px rgba(16,24,40,0.06)`
  - Lifted bento: `0 12–18px 32–40px rgba(38,38,36,0.15)`
  - Brand orange glow: `0 14px 32px rgba(217,119,87,0.30)`
- **Page padding:** `32px` around `<main>`
- **Card padding:** default `24px` (override with `<Card padding={N}>`)
- **Bento hover:** `.bento-card { transition; } .bento-card:hover { translateY(-2px); }`

### Status pill mapping (`StatusPill`)

| Kind | bg | fg |
|---|---|---|
| In Stock / Shipped | `#DCFCE7` | `#16A34A` |
| Low | `#FEF3C7` | `#D97706` |
| Critical | `#FEE2E2` | `#DC2626` |
| Out of Stock / Completed | `#F3F4F6` | `#374151` |
| In Progress | `#F5DDCD` | `#C25E3F` |

---

## Screens

### Sidebar (`function Sidebar`)

Dark `#262624` rail, 240px wide. Six entries:

1. Dashboard (`LayoutDashboard`)
2. Pricing Calculator (`Calculator`)
3. Inventory (`Package`)
4. Financial Analytics (`LineChart`)
5. Reports (`FileText`)
6. **Settings (`Settings` — cog icon)**

Active item: white text, `rgba(255,255,255,0.06)` background, 4px brand-orange left bar.

Bottom: avatar pill ("LM" gradient circle) with name "Layla Mitchell" and role.

### 1. Dashboard

Bento layout (recently redesigned for "bold & modern"):

- **Hero row** (2fr / 1fr grid):
  - **Net Profit hero (dark card):** `$2,345` at 64px/800-weight on a `#262624 → #1A1A18` gradient. Decorations: orange radial glow blob, faint grid pattern (radial-mask), green delta pill ("↑ 18% vs last month"), "54.8% margin" label, inline 6-month net-profit SVG sparkline (`#D97757 → #F0A584` gradient area + line). Click → `analytics`.
  - **Active Orders (orange gradient card):** `12` at 56px/800-weight on `#D97757 → #C25E3F`. Decorative wireframe hexagon, stage chips derived from `activeOrders`. Click → opens `active` modal.
  - **Alerts tile:** white card with 44px red icon-square containing `AlertTriangle` and a `pulseDot`-animated red dot. Click → opens `alerts` modal.
- **Secondary KPI strip** (3-col grid): Monthly Revenue / Monthly Expenses / Profit Margin. Each tile: label + colored delta pill (green or red), 32px value, gradient mini progress bar.
- **Chart row** (60/40 grid):
  - **Revenue vs Expenses** bar chart with gridlines, Y-axis ticks (`$0.0k → $5.0k`), gradient bars (`#F0A584 → #D97757` for revenue, `#CBD5E1 → #94A3B8` for expenses), soft glow on the latest month's revenue bar.
  - **Low Stock** card: per-item depletion bars. Critical = red gradient, Low = amber gradient. Shows `qty unit of reorder` and percent.
- **Recent Orders table:** customer initials in gradient avatar circles, hover row tint, `View All →` link to all-orders modal.

**State:** local `modal` (`'alerts' | 'active' | 'all' | null`).

**Modals:**
- `alerts` — list with reorder/dismiss buttons, navigates to inventory.
- `active` — table of 12 active orders with stage pills (`Design`, `Queued`, `Printing`, `Curing`).
- `all` — extended order history table.

### 2. Pricing Calculator

Form for computing the cost of a custom 3D print. Inputs:
- Material name + amount used + cost per kg/L
- Print time, electricity rate, printer wattage, printer cost, lifespan
- Labor time + hourly rate
- Packaging cost
- Platform fees (%)
- Margin (%)

Returns suggested price with breakdown. Empty inputs fall back to `placeholders`. Results render into a `result` panel anchored at `resultRef`.

### 3. Inventory

- **Summary cards (3):** Total Materials / Low Stock / Out of Stock — derived from live row state (not hardcoded).
- **Action bar:** search input, category `Select` (All / Resin / Filament / Accessories), `+ Add Material` button (orange).
- **Table:** Material Name, Category, Quantity, Reorder Point, Status pill, Last Updated, kebab.
- **Add Material modal** (recently added):
  - Fields: name (autoFocus), category dropdown, unit dropdown (`ml` / `g` / `pcs` / `sheets` / `sets`), quantity, reorder point.
  - Validates required + non-negative numbers; surfaces a red error banner.
  - Status auto-derived on submit:
    - `qty <= 0` → **Out of Stock**
    - `qty <= reorder * 0.5` → **Critical**
    - `qty < reorder` → **Low**
    - else → **In Stock**
  - Date set to today via `new Date().toLocaleDateString("en-US", { month: "short", day: "numeric" })`.
  - New rows are prepended to the list.

**State:** `q`, `cat`, `rows` (initialized from `INVENTORY`), `addOpen`, `form`, `err`. Filtering uses a `useMemo`.

**Persistence:** none — additions live for the session only.

### 4. Financial Analytics

Daily / Weekly / Monthly tabs that **actually drive the data** (recently wired up). Each period has its own dataset:

- **KPIs** (4): Total Revenue, Total Expenses, Net Profit, Profit Margin — each with delta + comparison label ("vs yesterday" / "vs last week" / "vs last month").
- **Revenue Trend** SVG line chart with adaptive Y-axis (uses `fmtMoney` to switch between `$143` and `$4.3k`).
- **Expense Breakdown** donut + legend.
- **Top Products** horizontal bar list.

Period datasets:

| Period | Trend window | Snapshot label | Sample revenue |
|---|---|---|---|
| Daily | Last 7 days (Wed–Tue, Apr 22–28) | Apr 28, 2026 | $143 |
| Weekly | Last 6 weeks (W12–W17) | Apr 22 – Apr 28, 2026 | $995 |
| Monthly | Last 6 months (Nov–Apr) | April 2026 | $4,280 |

**State:** `period` ("Monthly" default).

### 5. Reports

Form to generate financial reports. Inputs:
- Report type (`Monthly Revenue Summary`, `Profit & Loss`, etc.)
- Date range (`from`, `to` — default Apr 1–30, 2026)
- Section toggles (`revenue`, `expenses`, `profit`, `products`, `inventory`)
- Format (`PDF` / `Print`)

Generates an inline preview (`generated` flag). `Print` format triggers `window.print()`. Right column lists previous reports.

### 6. Settings (recently added)

Four cards plus a save/reset footer:

1. **Business Profile:** Business Name, Owner Name, Contact Email, Phone.
2. **Localization & Pricing:** Currency dropdown (USD / EUR / GBP / AED / EGP / SAR), Tax Rate %.
3. **Inventory Defaults:** Default Reorder Point.
4. **Notifications:** Three toggle switches (Low stock alerts, Order updates, Weekly digest).

**Persistence:** writes to `localStorage` under key `printops360.settings` (the **only** persisted state in the app). `loadSettings()` runs on mount. Reset clears the storage key and reverts to `DEFAULT_SETTINGS`.

**Note:** values are not yet *consumed* by other screens — the Pricing Calculator, dashboard alerts, and inventory defaults still use hardcoded values. Wiring them through is open work.

### Intro screen

Black-and-white 3D-printer-themed splash on first load. Animated wireframe shapes (`WireShape`), Anton-typeface title "Noureldin Submission" with `glowPulse` / `whiteGlow` keyframes, scanning beam (`scanDown`/`scanUp`), pulsing "Tap to enter" hint. Loops indefinitely until tap or any key.

State lives on `App`: `showIntro` defaults `true`, set `false` on completion. Intro overlays the app (`React.Fragment` wrapper).

---

## Shared components reference

```jsx
<PageHeader title="..." subtitle="..." right={<ReactNode/>} />
<Card padding={24} className="..." style={{}}>...</Card>
<KPI label value valueColor trend trendKind sub onClick />
<StatusPill kind="In Stock" />          // see status pill mapping above
<Modal open onClose title subtitle width={560}>...</Modal>
<Select value onChange options={["A","B"]} />
```

Icons used via `const I = window.LIcons; <I.Plus size={16} />`.
Available icons: `LayoutDashboard, Calculator, Package, LineChart, FileText, Search, Plus, MoreHorizontal, AlertTriangle, ArrowRight, ChevronDown, TrendingUp, TrendingDown, Download, Printer, Save, Bell, Filter, Settings`.

To add a new icon: extend the `window.LIcons` object in the head `<script>` using `mk([...paths])` for path-only icons or `mkRaw([{tag, attrs}])` for icons that need `<circle>`, `<rect>`, etc.

---

## Data shapes

### Inventory row
```js
{ name: "UV Resin (Clear)", cat: "Resin", qty: 120, unit: "ml", reorder: 200, status: "Critical", date: "Apr 28" }
```

### Order row (recent / all)
```js
{ id: "#1047", c: "Sarah K.", items: "Resin Earrings (x3)", total: 45.00, status: "Shipped", date: "Apr 27" }
```

### Active order
```js
{ id: "#1046", c: "Mike R.", items: "Wedding Favor Set", total: 120.00, due: "May 1", stage: "Printing" }
```
Stages: `Design`, `Queued`, `Printing`, `Curing`.

### Alert
```js
{ kind: "stock", severity: "Critical" | "Low", title: "...", detail: "..." }
```

### Settings (persisted to `localStorage`)
```js
{
  businessName: "Layla's Print Studio",
  ownerName: "Layla Mitchell",
  email: "layla@printops360.com",
  phone: "+1 (555) 123-4567",
  currency: "USD ($)",        // or "EUR (€)" / "GBP (£)" / "AED (د.إ)" / "EGP (£)" / "SAR (﷼)"
  taxRate: "8.5",             // string
  defaultReorderPoint: "100", // string
  lowStockAlerts: true,
  orderUpdates: true,
  weeklyDigest: false,
}
```

---

## Routing / state model

`App` owns:
- `showIntro` — boolean
- `screen` — one of `dashboard | pricing | inventory | analytics | reports | settings`

```jsx
const screens = {
  dashboard: <Dashboard onNavigate={setScreen} />,
  pricing:   <PricingCalculator />,
  inventory: <Inventory />,
  analytics: <FinancialAnalytics />,
  reports:   <Reports />,
  settings:  <Settings />,
};
```

The `Dashboard` is the only screen that receives `onNavigate` (used for click-throughs from KPI cards, "View All Inventory", and the alerts modal).

There is **no router**. Screen state is purely in-memory and resets on full reload. URL never changes.

---

## Recent feature work (changelog)

| Commit | Branch | Summary |
|---|---|---|
| `Financial Analytics: Daily/Weekly/Monthly tabs now drive all values` | `claude/financial-analytics-time-filter-QzOwV` → `main` | Wired the previously-decorative period tabs to per-period datasets (KPIs, trend, donut, products). Added adaptive y-axis and money formatter. |
| `Fix duplicate products declaration breaking Financial Analytics` | same | Removed a stale `const products = [...]` left over from refactor that caused a SyntaxError and rendered the whole app blank. |
| `Inventory: wire up Add Material button to a working modal form` | same | Added Modal-based add-material flow with validation, auto-derived status, prepend to live row list. Summary cards now derive from rows. |
| `Add Settings page with persistent business profile, currency, tax, inventory defaults, and notification toggles` | same | New 6th sidebar entry. Settings persist to `localStorage`. |
| `Dashboard: bold modern bento redesign` | same | Replaced the equal-column KPI row with a hero bento layout, added gradient bars/gridlines, depletion bars on Low Stock, customer avatars on Recent Orders. |

---

## Known limitations / open work

These are areas the app handwaves and would need work before "real" use:

- **No persistence** for inventory rows, orders, financial analytics, or any user-entered material — only `Settings` survives a refresh.
- **No backend / no auth.** All data is hardcoded module-level constants.
- **No customer/CRM module.** Orders carry a customer name string but there's no notion of a customer entity.
- **No order detail view.** Clicking an order does nothing — modals only show lists.
- **Inventory ↔ Orders not linked.** Completing an order doesn't deduct material.
- **Settings values are not consumed elsewhere.** Currency, tax rate, and default reorder point are stored but not yet read by other screens.
- **No pricing-calculator persistence.** Quotes can't be saved.
- **No reorder/PO flow.** "Reorder Now" in the alerts modal currently just navigates to inventory.
- **No printer queue / production scheduler** — would be a high-value differentiator for a 3D-print SaaS.
- **Mobile responsiveness:** designed desktop-first; not explicitly tested on small screens.

A recommended development priority order:
1. Persist app state to `localStorage` (extend the `Settings` pattern). Single biggest "wow" win.
2. Wire `Settings` values into Pricing Calculator, Inventory defaults, and dashboard.
3. Customer (CRM) module + tie orders to customers.
4. Order detail view with stage workflow (Design → Print → Finishing → Shipped).
5. Inventory deduction on order completion.
6. Reorder/PO flow from low-stock alerts.

---

## Operational notes

### Running locally
Open `index.html` directly in a browser. There's nothing to install or build. Babel transpiles JSX in-browser on every load.

### Deploying
Pushing to `main` triggers a GitHub Pages build. URL: `https://noureldin0007.github.io/printops-360/`. Allow ~1–2 minutes after a push, then **hard-refresh** (`Cmd/Ctrl+Shift+R`) to bypass the in-browser cache.

### Editing safely
Because everything is one file with in-browser Babel:
- A single syntax error blanks the entire site. Always `grep` the file before/after refactors for duplicate identifiers (`grep -n "const products\|const trend\|const expenses"`).
- Test by loading the local file in a browser before pushing.
- The current branch convention is `claude/<feature-name>-<random>`; merge fast-forward to `main` to deploy.

### Style conventions
- Inline `style={{}}` is the dominant pattern. Tailwind classes are used only for layout (`flex`, `grid`, `gap-*`, `mb-*`, `items-center`, etc.).
- Numeric values that should align in tables/KPIs use `className="num-tabular"`.
- Hover effects: prefer `onMouseEnter/Leave` mutating `e.currentTarget.style` (matches existing pattern), or use the `.bento-card` class for the standard lift.
- Color literals are used directly (no theme object). When introducing new colors, reuse the palette table above.
