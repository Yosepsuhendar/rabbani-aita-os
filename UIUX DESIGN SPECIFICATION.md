# RABBANI AITA OS — UI/UX DESIGN SPECIFICATION (v1.0)

| Field | Value |
|---|---|
| Builds on | Product/Module Architecture (locked) + Database (locked) |
| Scope | 14 page areas · Desktop / Tablet / Mobile · Design System · Component Library · Navigation · User Journeys · Wireframes |
| Output | Design specification — **no code** |
| Status | PROPOSED — awaiting design approval |

---

## 0. DESIGN THESIS & PRINCIPLES

**Thesis — "the ledger as a calm instrument."** A dignified workspace for accountants, tax consultants, auditors, partners, and their clients. Compliance work is full of deadlines, money, documents, and sign-offs; the interface makes that feel *controlled and trustworthy*, never noisy.

**Principles**
1. **Numbers are the hero.** Money, NPWP, NTPN, invoice numbers use tabular figures and a monospace treatment so columns align and never shift.
2. **Sign-off is sacred.** The acts that define this firm — filing a return, issuing a management letter, approving an invoice — culminate in a deliberate **Sign-off Seal**. Approvals are never an afterthought button.
3. **One calm accent.** Brass gold appears only at moments of value (seals, the active step, premium states). Everything else is quiet ink-green and paper.
4. **The AI co-pilot assists, it doesn't chat.** A right-rail assistant that drafts, computes, and validates *in context*, always handing consequential actions back to a human gate.
5. **Density is a setting, not a default.** Compact mode for power users (data grids); comfortable mode for clients and reading.
6. **Plain, end-user language.** "File return," not "submit SPT payload." Same verb across the whole flow.

---

## 1. DESIGN SYSTEM

### 1.1 Brand & Voice
Trustworthy, precise, calm, quietly premium. Sentence case. Active verbs. Errors explain *what happened and the fix*; empty states invite the next action.

### 1.2 Color Tokens

**Core (light)**
| Token | Hex | Use |
|---|---|---|
| `--ink` | `#0F3D2E` | Primary — ledger ink-green; headers, primary buttons, active nav |
| `--ink-700` | `#16513D` | Hover/pressed |
| `--brass` | `#B0852F` | Accent — seals, active step, premium; **used sparingly** |
| `--brass-soft` | `#E9DCC0` | Brass tint fills |
| `--paper` | `#F5F7F6` | App background (cool, faint green-gray) |
| `--surface` | `#FFFFFF` | Cards, tables, sheets |
| `--text` | `#15201C` | Primary text |
| `--text-muted` | `#5B6661` | Secondary text |
| `--border` | `#E2E6E3` | Hairlines, dividers, table rules |

**Semantic (status)**
| Token | Hex | Meaning |
|---|---|---|
| `--success` | `#1E7A52` | filed / paid / approved / completed / accepted |
| `--warning` | `#B5821E` | review / pending / closing / partially_paid |
| `--danger` | `#B23A2E` | overdue / rejected / invalid / cancelled / critical |
| `--info` | `#2C6E8F` | in_progress / submitted / draft-in-motion |
| `--neutral` | `#6B7672` | draft / inactive / archived |

**Dark mode:** `--paper #0F1714`, `--surface #15201C`, `--text #E7ECE9`, `--border #243029`; `--ink` lightens to `#3E9A78` for contrast; brass stays `#C39A4E`.

### 1.3 Status → DB Enum Mapping (single source of truth for pills)
| Domain (enum) | Value → color |
|---|---|
| EngagementStatus | draft·neutral / active·info / on_hold·warning / completed·success / cancelled·danger |
| TaxObligationStatus | pending·neutral / in_progress·info / filed·success / paid·success / overdue·danger / waived·neutral |
| TaxReturnStatus | draft·neutral / review·warning / approved·info / filed·success / rejected·danger |
| InvoiceStatus | draft·neutral / pending_approval·warning / approved·info / sent·info / partially_paid·warning / paid·success / overdue·danger / void·neutral / cancelled·danger |
| MlStatus | draft·neutral / manager_review·warning / partner_review·warning / approved·info / issued·success |
| AuditEngStatus | planning·neutral / fieldwork·info / review·warning / reporting·info / completed·success |
| FindingSeverity | low·neutral / medium·info / high·warning / critical·danger |
| CoretaxStatus | prepared·neutral / validating·info / valid·success / invalid·danger / submitted·info / accepted·success / rejected·danger |
| RunStatus (approvals) | queued·neutral / running·info / waiting_approval·warning / completed·success / failed·danger / cancelled·neutral |

### 1.4 Typography — IBM Plex superfamily
| Role | Face | Use |
|---|---|---|
| Display | **IBM Plex Serif** | Page hero titles, letterhead, issued documents (management letters, formal reports) |
| UI / body | **IBM Plex Sans** | All interface text; tabular numerals on for tables |
| Data / mono | **IBM Plex Mono** | Money, NPWP, NTPN, invoice numbers, references, code-like IDs |

**Type scale (px / line):** Display 32/40 · H1 24/32 · H2 20/28 · H3 16/24 · Body 14/22 · Small 13/20 · Caption 12/16 · Data 14 mono tabular. Weights: 600 headings, 500 labels/buttons, 400 body.

### 1.5 Spacing, Grid, Breakpoints, Density
- **Spacing scale:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64.
- **Grid:** 12-col desktop (max content 1320, gutter 24), 8-col tablet, 4-col mobile.
- **Breakpoints:** Mobile `<640` · Tablet `640–1023` · Desktop `1024–1439` · Wide `≥1440`.
- **Density modes:** Comfortable (row 48px) / Compact (row 36px) — toggle in top bar; Compact default for staff data grids, Comfortable default for Client Portal.

### 1.6 Radius / Elevation / Borders
- **Radius:** 6 (inputs/buttons) · 10 (cards) · 14 (sheets/modals) · full (pills/avatars).
- **Elevation:** flat-first. `e0` hairline border only · `e1` card `0 1px 2px rgba(15,61,46,.06)` · `e2` popover `0 8px 24px rgba(15,61,46,.10)` · `e3` modal. Brass never used for shadow.

### 1.7 Iconography & Imagery
- Line icons, 1.5px stroke, 20/24px (Lucide-style), squared-soft to match radius.
- Minimal imagery in-app. Landing uses one restrained hero motif (a precise ledger/seal motif), not stock photos.

### 1.8 Motion
- Durations 120–240ms, ease-out. Purposeful only: sheet/drawer slide, row expand, toast, **seal stamp** (the one signature flourish on sign-off). Respect `prefers-reduced-motion`.

### 1.9 Accessibility (quality floor)
WCAG 2.2 AA contrast · visible keyboard focus ring (`--ink` 2px offset) · full keyboard nav incl. data grids · 44px min touch target on mobile · semantic headings/landmarks · status never color-only (pill = dot + label) · forms with inline, specific errors · screen-reader labels on icon-only controls.

---

## 2. COMPONENT LIBRARY

**Atoms:** Button (primary/secondary/ghost/danger), Icon button, Input, Textarea, Select, Combobox, Checkbox, Radio, Switch, Date/period picker, Money input (mono, IDR), Status pill (dot+label, mapped §1.3), Severity tag, Avatar, Badge/counter, Tooltip, Tab, Breadcrumb, Link, Skeleton.

**Molecules:** KPI stat card (label + big tabular number + delta), Filter bar (search + facet chips + saved views), Pagination (cursor), Field group, File chip, Upload dropzone, Toast, Inline alert, Segmented control, Density toggle, Command-palette result row, Approval chip, **Sign-off Seal** (brass stamp with signer + timestamp).

**Organisms:**
- **Data Grid** — sortable, sticky header, column pin, row select, inline status pills, row actions, density-aware, empty/loading/error states, mobile→card fallback.
- **Record Detail (two-pane)** — left: record fields/timeline; right: related items tabs.
- **Approval Bar** — appears on records in `waiting_approval`; shows chain (maker → checker → partner), current gate, Approve / Request changes / Reject, reason field; writes to audit trail.
- **Stepper** — multi-stage flows (tax cycle, audit phases, coretax validate→submit).
- **Document Viewer** — PDF/preview with versions (ML letters, invoices, statements).
- **Timeline / Activity** — engagement & audit history, comments.
- **AI Co-pilot Rail** — context actions, drafts, citations, "send to approval" handoff (see §7).
- **Command Palette (⌘K)** — jump to client/engagement/return/invoice; run actions.
- **Charts** — line (revenue/AR aging), bar, donut (mix), heat (deadline calendar); ink+brass palette, tabular tooltips.
- **Notification Center** — grouped, channel icons, mark read, preferences.
- **Calendar / Deadline board** — tax & engagement due dates.

Every interactive component ships states: default, hover, focus, active, disabled, loading, error, empty, read-only (for `Client` role & locked periods).

---

## 3. NAVIGATION & INFORMATION ARCHITECTURE

### 3.1 Sitemap
```
PUBLIC                         STAFF APP (/app)                       CLIENT PORTAL (/portal)
├ Landing                      ├ Dashboard (role home)               ├ Portal Home
└ Login / SSO / Invite accept  ├ CRM ─ Clients · Leads · Activities  ├ Documents (requests/upload)
                               ├ Engagements                         ├ Deliverables
                               ├ Accounting                          ├ Invoices & Payments
                               ├ Tax Compliance ─ Returns · Calendar ├ Messages
                               ├ Internal Audit                      └ Profile / Users
                               ├ Management Letter
                               ├ Business Advisory
                               ├ Coretax Center
                               ├ Billing (AR)
                               ├ Notifications
                               ├ CEO Dashboard            (CEO/Super Admin)
                               └ Settings ─ Users · Roles · Integrations · Templates
```

### 3.2 Staff Shell
Left **sidebar** (collapsible icon rail) · **top bar** (⌘K search, density toggle, theme, notifications, profile/tenant switch, AI toggle) · optional right **AI rail** · content with breadcrumb + title + primary actions.

### 3.3 Client Portal Shell
Lighter, comfortable density, warmer paper. Top bar + simple left nav (Documents, Deliverables, Invoices, Messages). No staff modules ever visible. Read-mostly with clear upload/pay actions.

### 3.4 Role-Based Navigation (locked 6 roles)
| Nav item | Super Admin | CEO | Accounting Admin | Tax Admin | Audit Admin | Client |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| Dashboard | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| CRM / Engagements | ✓ | ✓ | ✓ | view | view | — |
| Accounting | ✓ | ✓ | ✓ | view | — | — |
| Tax Compliance | ✓ | ✓ | view | ✓ | — | — |
| Internal Audit | ✓ | ✓ | — | — | ✓ | — |
| Management Letter | ✓ | ✓ | — | — | ✓ | — |
| Business Advisory | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Coretax Center | ✓ | ✓ | view | ✓ | — | — |
| Billing (AR) | ✓ | ✓ | ✓ | — | — | — |
| Notifications | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CEO Dashboard | ✓ | ✓ | — | — | — | — |
| Settings | ✓ | ✓(tenant) | — | — | — | — |
| Client Portal | — | — | — | — | — | ✓ (scoped) |

"view" = read-only (no create/approve). Nav renders only items the role's permissions allow.

### 3.5 Responsive Navigation
- **Desktop:** full sidebar + labels; AI rail dockable.
- **Tablet:** sidebar collapses to 64px icon rail (labels on hover/expand); AI rail becomes an overlay sheet.
- **Mobile:** sidebar → bottom tab bar (Home · Work · Add · Alerts · More); full menu in "More" sheet; ⌘K becomes a search icon; AI becomes a floating button → bottom sheet.

---

## 4. USER JOURNEYS

**J1 — Sign in (all staff).** Login → (optional 2FA) → role-aware Dashboard. Pending approvals & due-soon items surface first.

**J2 — Onboard a client (Accounting Admin / CEO).**
CRM ▸ New client (legal name, NPWP, industry) → create **Engagement** (service line, period, partner, manager) → **Invite to Portal** (email/WhatsApp) → issue first **Document request**. Gate: new client/engagement → *Partner approves (KYC)*.

**J3 — Monthly tax cycle (Tax Admin).**
Tax ▸ Obligations (due-soon) → open obligation → **Prepare return** (AI compute-assist, cited) → **Review** (Supervisor) → **Approval Bar: Partner sign-off** (Seal) → **Coretax Center**: validate → submit → reconcile **NTPN/BPE** receipt. Statuses move pending → in_progress → review → approved → filed; Coretax prepared → valid → submitted → accepted.

**J4 — Audit → Management Letter (Audit Admin).**
Audit ▸ engagement → fieldwork (workpapers, evidence) → record **findings** (severity) → **Management Letter**: AI drafts findings → editor → **Approval Bar: Manager → Partner → Issue** (Seal) → published to Client Portal. ML status draft → manager_review → partner_review → approved → issued.

**J5 — Bill & get paid (Accounting Admin + Client).**
Billing ▸ New invoice (from engagement/time, PPN line) → **Approval Bar** (Manager; Partner if over threshold) → **Send** → Client Portal shows invoice → **Client pays** (gateway/transfer) → payment reconciled. InvoiceStatus draft → pending_approval → approved → sent → partially_paid/paid.

**J6 — Client in the Portal (Client).**
Portal Home → **Documents**: see requested items, upload → status to firm → **Deliverables**: view/download issued letters/statements → **Invoices**: view & pay → **Messages** with the team. Read-only elsewhere; scoped to own client only.

**J7 — CEO review (CEO).** CEO Dashboard → revenue & AR aging, engagement pipeline, deadline risk, productivity, AI usage → drill into any client/engagement.

---

## 5. WIREFRAMES — DESKTOP

**Shell (applies to all staff pages)**
```
┌────────────────────────────────────────────────────────────────────────────┐
│ ≡  RABBANI AITA OS      ⌘K Search clients, returns, invoices…   ◧ ◐ 🔔3 ⬡AI │
│                                                          Tax Admin · Firm ▾  │
├──────────┬──────────────────────────────────────────────────────┬──────────┤
│ Home     │  Home / Tax / Returns                          [+ New]│  AI      │
│ CRM      │  ─────────────────────────────────────────────────── │  CO-PILOT│
│ Account. │  page content                                         │  (dock)  │
│ Tax    ◀ │                                                       │          │
│ Audit    │                                                       │          │
│ Mgmt Ltr │                                                       │          │
│ Advisory │                                                       │          │
│ Coretax  │                                                       │          │
│ Billing  │                                                       │          │
│ Notif    │                                                       │          │
│ CEO Dash │                                                       │          │
│ Settings │                                                       │          │
└──────────┴──────────────────────────────────────────────────────┴──────────┘
```

**5.1 Landing**
```
┌──────────────────────────────────────────────────────────────┐
│ RABBANI AITA OS                         Features  Pricing  [Sign in]│
├──────────────────────────────────────────────────────────────┤
│  HERO (serif):  "Run the whole firm.                          │
│                  Books, tax, audit, advisory — with AI."      │
│  [Request access]  [Sign in]      ◇ seal/ledger motif         │
├──────────────────────────────────────────────────────────────┤
│  trust row: Coretax-ready · multi-client · audit trail        │
│  3 value cards: Compliance on time · One client portal · CEO view│
│  Footer: company · legal · ID/EN                              │
└──────────────────────────────────────────────────────────────┘
```

**5.2 Login**
```
            ┌───────────────────────────────┐
            │  RABBANI AITA OS              │
            │  Sign in                      │
            │  Email     [______________]   │
            │  Password  [______________]   │
            │  [ ] Remember   Forgot?       │
            │  [   Sign in   ]              │
            │  ── or ──   [ SSO ]           │
            │  Invited? Accept invitation → │
            └───────────────────────────────┘
```

**5.3 Dashboard (role home)**
```
Title: Good morning, Andi · Tax Admin            Density ◧  Period: Jun 2026 ▾
┌ KPIs ───────────────────────────────────────────────────────────────────┐
│ [Due this week 12] [Awaiting my approval 5] [Filed MTD 38] [Overdue 2]   │
└──────────────────────────────────────────────────────────────────────────┘
┌ My approvals (waiting_approval) ──────────┐ ┌ Deadline board (calendar) ──┐
│ ▸ PPN — PT Surya · review · due Jun 30    │ │ Jun  ░░██░░░██░  PPh/PPN dots│
│ ▸ Return PPh23 — CV Mitra · partner sign  │ │ click day → list            │
└───────────────────────────────────────────┘ └─────────────────────────────┘
┌ Recent activity / assigned work (data grid) ────────────────────────────┐
│ Client      Service   Item            Status        Due        Owner     │
│ PT Surya    Tax       PPN Jun         ●review       Jun 30     me        │
│ CV Mitra    Tax       PPh23 Jun       ●in_progress  Jul 10     me        │
└──────────────────────────────────────────────────────────────────────────┘
```

**5.4 Client Portal (external)**
```
RABBANI · Portal — PT Surya Abadi                         🔔  Profile ▾
┌ Home ┬ Documents ┬ Deliverables ┬ Invoices ┬ Messages ──────────────────┐
│ Welcome. 3 documents requested · 1 invoice due · 2 new deliverables      │
│ ┌ Action needed ──────────────┐  ┌ Recent deliverables ───────────────┐ │
│ │ Upload: Bank statement May  │  │ Mgmt Letter FY2025  [view]         │ │
│ │ Upload: Sales invoices May  │  │ Financial Stmt Q1   [view]         │ │
│ │ [ Upload files ]            │  └─────────────────────────────────────┘ │
│ └─────────────────────────────┘  Invoice INV-2026-014  Rp 12.500.000 [Pay]│
└──────────────────────────────────────────────────────────────────────────┘
```

**5.5 CRM (list + detail)**
```
CRM / Clients                              Filter: status ▾  industry ▾  [+ New client]
┌ Data grid ──────────────────────────────────────────────────────────────┐
│ Client          NPWP            Industry   Status     Owner    Engs       │
│ PT Surya Abadi  01.234.567.8…   Retail     ●active    Andi     3          │
│ CV Mitra Jaya   02.345.678.9…   Services   ●prospect  Sari     1          │
└──────────────────────────────────────────────────────────────────────────┘
  Detail (two-pane): [Profile | Contacts | Engagements | Documents | Activity | Billing]
```

**5.6 Accounting**
```
Accounting / PT Surya Abadi · FY2026                 Period Jun ▾  [Close period]
[ Journals | Trial balance | Statements | Reconciliations ]
┌ Journals (grid) ──────────────────────────────────────────┐ ┌ Period close ─┐
│ Date    Ref     Description       Debit       Credit  ●st  │ │ status ●open  │
│ 06/03   JV-014  Sales accrual    12.000.000        —  post │ │ checklist… [▸]│
│ 06/03   JV-014  AR               —    12.000.000      post │ │ Close → approve│
└────────────────────────────────────────────────────────────┘ └───────────────┘
  (debit/credit columns mono, right-aligned, balanced indicator)
```

**5.7 Tax Compliance**
```
Tax / Returns                       Tax type ▾  Period Jun 2026 ▾  [+ New return]
┌ Obligations & returns (grid) ─────────────────────────────────────────────┐
│ Client     Tax     Period   Due      Return        Status      Action     │
│ PT Surya   PPN     Jun-26   Jun 30   SPT Masa PPN   ●review     [Open]     │
│ CV Mitra   PPh23   Jun-26   Jul 10   —             ●pending     [Prepare]  │
└────────────────────────────────────────────────────────────────────────────┘
  Return detail: Stepper [Prepare ▸ Review ▸ Sign-off ▸ File] + Approval Bar + AI compute
```

**5.8 Internal Audit**
```
Audit / PT Surya Abadi · FY2025         Phase: ●fieldwork    [Add workpaper]
[ Planning | Risk | Programs | Workpapers | Findings | Evidence ]
┌ Findings (grid) ──────────────────────────────────────────────────────────┐
│ Ref   Title                     Severity   Status               Owner      │
│ F-01  Revenue cutoff weakness   ●high      ●management_response Dewi       │
│ F-02  Missing approvals on PO   ●medium    ●open                Dewi       │
└────────────────────────────────────────────────────────────────────────────┘
  [Generate Management Letter from findings →]
```

**5.9 Management Letter**
```
Management Letter / PT Surya FY2025      Status ●partner_review   v3 ▾
┌ Editor ───────────────────────────────────────┐ ┌ Findings → letter ──────┐
│ (serif letterhead preview)                     │ │ ☑ F-01 high              │
│ Observation · Risk · Recommendation · Response │ │ ☑ F-02 medium            │
│ [AI: draft from findings] [Insert]             │ │ Approval Bar:            │
│                                                │ │  Manager ✓ → Partner ⌛   │
│                                                │ │ [Approve & Issue ⮕ Seal] │
└────────────────────────────────────────────────┘ └──────────────────────────┘
```

**5.10 Business Advisory**
```
Advisory / Projects                                   [+ New project]
┌ Projects (grid) ─────────────────────────────┐ ┌ Project detail ───────────┐
│ Client     Type            Status   Due       │ │ [Analyses|Deliverables|   │
│ PT Surya   valuation       ●in_prog Jul 20    │ │  Recommendations]         │
│ CV Mitra   business_plan   ●review  Aug 02    │ │ AI: scenario analysis ▸   │
└───────────────────────────────────────────────┘ └───────────────────────────┘
```

**5.11 Coretax Center**
```
Coretax Center                        Status ▾  Period Jun 2026 ▾
┌ Submissions (grid) ───────────────────────────────────────────────────────┐
│ Client    Doc          Period  Validation   Status        Receipt          │
│ PT Surya  SPT Masa PPN Jun-26  ●valid (0✕)  ●submitted    —                │
│ CV Mitra  Bupot        Jun-26  ●invalid(3✕) ●prepared     —                │
└────────────────────────────────────────────────────────────────────────────┘
  Detail: Stepper [Map ▸ Validate ▸ Submit ▸ Receipt] · validation list (error/warning/info)
```

**5.12 Notifications**
```
Notifications                                   [Mark all read]  ⚙ Preferences
┌ Today ─────────────────────────────────────────────────────────────────────┐
│ 🔔 Return PPh23 (CV Mitra) needs your sign-off            · 2h   in_app·email │
│ 🔔 Client PT Surya uploaded 3 documents                  · 4h   in_app       │
├ Earlier ───────────────────────────────────────────────────────────────────┤
│ 🔔 Invoice INV-2026-014 paid                             · 1d                │
└────────────────────────────────────────────────────────────────────────────┘
  Preferences: channels (in_app/email/WhatsApp/SMS) × categories · digest daily/weekly
```

**5.13 Billing (AR)**
```
Billing / Invoices                     Status ▾  Client ▾   [+ New invoice]
┌ KPIs [Outstanding Rp 84.2jt] [Overdue Rp 12.0jt] [Paid MTD Rp 210jt] ─────┐
┌ Invoices (grid) ──────────────────────────────────────────────────────────┐
│ Number          Client     Issue    Due      Total         Status         │
│ INV-2026-014    PT Surya   Jun 10   Jun 25   12.500.000    ●overdue        │
│ INV-2026-015    CV Mitra   Jun 12   Jun 27   8.000.000     ●sent           │
└────────────────────────────────────────────────────────────────────────────┘
  Invoice detail: lines + PPN, Approval Bar (threshold), Send, PDF, payments timeline
```

**5.14 CEO Dashboard**
```
CEO Dashboard · Firm overview                       Period Q2 2026 ▾  Export ▾
┌ [Revenue Rp 1.42bn ▲8%] [AR aging Rp 84jt] [Active engagements 46] [On-time 92%] ┐
┌ Revenue trend (line) ───────────────┐ ┌ Engagement pipeline (donut) ─────┐
│   ╱╲    ╱╲                           │ │ accounting/tax/audit/advisory mix │
└─────────────────────────────────────┘ └───────────────────────────────────┘
┌ Deadline risk (heat) ───────────────┐ ┌ Client health / top clients ─────┐
│ overdue · due-7d · due-30d           │ │ revenue · status · risk          │
└─────────────────────────────────────┘ └───────────────────────────────────┘
  [AI: summarize this quarter →]
```

---

## 6. RESPONSIVE — TABLET & MOBILE

### 6.1 Strategy
- **Layout collapse:** 12-col → 8-col (tablet) → 4-col, single-column stacks (mobile).
- **Sidebar:** full → 64px icon rail (tablet) → bottom tab bar + "More" sheet (mobile).
- **Data grids → cards:** each row becomes a card (primary field + status pill + 2 meta + chevron); horizontal scroll avoided. Bulk select via long-press.
- **Two-pane detail → stacked:** list → tap → full-screen detail with back; tabs become a scrollable segmented control.
- **Approval Bar → sticky bottom bar** on mobile (Approve / Changes / Reject).
- **AI rail → floating button → bottom sheet.**
- **Tables of money:** key figure shown; secondary columns behind "Details".
- **Forms:** single column, large 44px targets, native pickers, sticky save bar.

### 6.2 Mobile wireframes (representative)
```
DASHBOARD (mobile)            CRM LIST (mobile)           APPROVAL (mobile)
┌───────────────┐            ┌───────────────┐           ┌───────────────┐
│ ≡  Home    🔔 │            │ ‹ Clients   +│           │ ‹ PPN · PT Surya│
│ Due 12 │Appr 5│            │ [search…]     │           │ Period Jun-26  │
│ Filed 38│Ovd 2│            │┌─────────────┐│           │ Amount         │
│───────────────│            ││PT Surya     ││           │  Rp 4.200.000  │
│ Approvals     │            ││●active · 3  ││           │ Status ●review │
│ ▸ PPN PT Surya│            │└─────────────┘│           │ Chain: Mgr✓→You │
│ ▸ PPh23 Mitra │            │┌─────────────┐│           │─ AI summary ──  │
│───────────────│            ││CV Mitra     ││           │ Checks passed 6 │
│ Deadlines ▸   │            ││●prospect ·1 ││           │                 │
│               │            │└─────────────┘│           │ [Sign off ⮕Seal]│
│ ⌂  ▦  ＋  🔔 ⋯│            │ ⌂  ▦  ＋ 🔔 ⋯ │           │ [Changes][Reject]│
└───────────────┘            └───────────────┘           └───────────────┘
```
Client Portal (mobile): bottom tabs Home · Docs · Bills · Messages; big "Upload files" and "Pay invoice" buttons; comfortable density.

### 6.3 Per-page adaptation
| Page | Tablet | Mobile |
|---|---|---|
| Landing | 1–2 col, sticky CTA | single col, hero first |
| Login | centered card | full-width card |
| Dashboard | 2-col KPIs, stacked panels | KPI 2×2, list panels |
| Client Portal | tabs + cards | bottom tabs, action cards |
| CRM | icon rail + grid | list cards → full detail |
| Accounting | tabs scroll, grid scroll-x for ledgers | summary cards + "view ledger" |
| Tax | grid → cards | obligation cards + stepper sheet |
| Audit | tabbed cards | phase chips + findings cards |
| Mgmt Letter | editor over findings | editor full-screen; findings sheet; sticky approval |
| Advisory | grid + detail | project cards + tabs sheet |
| Coretax | grid + stepper | submission cards + validation sheet |
| Notifications | list | list + filter sheet |
| Billing | KPIs + grid | KPI cards + invoice cards |
| CEO Dashboard | 2-col charts | stacked charts, swipe |

---

## 7. AI ASSISTANT UX (cross-cutting)

- **Surface:** right **co-pilot rail** (desktop), bottom sheet (mobile). Toggle in top bar; opens with the current record's context.
- **In-context actions per module:** Tax → "Compute PPN / draft return"; Audit → "Analyze for anomalies / draft finding"; ML → "Draft letter from findings"; Advisory → "Build scenario"; Coretax → "Explain validation errors"; Docs → "Classify upload"; CEO → "Summarize quarter".
- **Always cited & gated:** AI output shows sources (regulation/standard/firm template) and a **"Send to approval"** handoff — never auto-files, auto-sends, or auto-issues. Consequential output routes into the **Approval Bar**.
- **Tone:** drafts in the firm's voice; user reviews/edits before anything leaves the system.
- **Trust cues:** "AI-drafted · review before use" label; token/cost not shown to users but metered server-side.

---

## 8. STATES & MICROCOPY PATTERNS

- **Empty:** action-first. e.g. Tax returns empty → "No returns this period. Prepare your first return." [Prepare return]
- **Loading:** skeleton rows/cards; never spinner-only on data grids.
- **Error:** specific + recoverable. "Coretax validation failed: 3 errors. Open the list to fix." (not "Something went wrong.")
- **Permission/read-only (`Client`, locked period, "view" roles):** controls hidden or disabled with reason tooltip "Read-only — period closed."
- **Approval pending:** record shows amber `waiting_approval` banner + Approval Bar.
- **Sign-off success:** brass **Seal stamp** animation + toast "Return filed. NTPN recorded."

---

## 9. WHAT'S LOCKED VS FLEXIBLE

**Locked (this spec):** IA/sitemap, role-based nav matrix, status→enum color mapping, the 6-role surfaces, shell structure (sidebar/top bar/AI rail/portal shell), component inventory, responsive collapse rules, approval & sign-off pattern.

**Flexible (tunable without rework):** exact palette hex within the ink/brass/paper family, final typeface licensing (Plex is the proposal), illustration style, chart specifics, copy.

**STATUS: AWAITING DESIGN APPROVAL.** Reply `APPROVE DESIGN` to lock the UX foundation, or flag any page/flow to adjust. No code generated.
