# Forward-Deployed Operations Platform

> The operating system for a Forward Deployed Engineering / Professional Services organization.

## The Problem

When I looked at how PS teams actually operate—managing customer engagements, tracking
delivery health, coordinating resources, surfacing risk early—I found a gap that no
off-the-shelf tool fills cleanly.

- **CRMs** track pipeline. They don't model delivery.
- **Project tools** track tasks. They don't understand engagement health, utilization,
  or what a "blocked" engagement costs in renewal risk.
- **BI tools** show data. They don't let a team act on it.

The result is that most PS organizations manage their book of business across a
combination of spreadsheets, Slack threads, and CRM notes. Critical information is
invisible until it becomes a crisis.

This is the system I built to close that gap.

## What It Is

A full-stack React application in production at v9+, used by a real PS team, managing
real customer engagements.

### Views

| View | Persona | Purpose |
|------|---------|---------|
| Portfolio | Leader / EM | Health index across all active accounts |
| Engagement Workspace | FDE / EM | Delivery tracking, milestones, risk, EVM |
| Meeting Prep | FDE | AI-synthesized briefing before every customer call |
| Time & Resources | FDE / EM | Logged hours, capacity planning, utilization |
| Command Center | Leader | Practice-wide capacity queue, needs-attention priority strip |
| Client Portal | Client | Tailored engagement visibility for external users |
| Admin | Admin | Configurable per-org: toggles, persona maps, master data |

### AI Features

- **Meeting Prep synthesis** — Gong call data + pending tasks → structured briefing document (objectives, key context, action items, landmines, stakeholders)
- **Account intelligence** — "Why this account needs attention" grounded in real signals (ARR, health grade, support cases, adoption data)
- **Post-call synthesis** — All attendee notes → deduplicated action items → tasks automatically created
- **DomoGPT integration** throughout with rule-based fallbacks (AI should degrade gracefully, never fail loudly)

## Architecture

```
┌──────────────────────────────────────────┐
│  React + Vite + TypeScript               │
│  Tailwind CSS · Framer Motion            │
│  7 persona-aware views                   │
├──────────────────────────────────────────┤
│  Service Layer                           │
│  Code Engine functions (JS, single       │
│  dispatcher pattern):                    │
│  · computeCapacity                       │
│  · recordEngagementChange                │
│  · logTime                               │
│  · extractSOW (PDF parsing)              │
│  · sppCommentSync                        │
│  · sendEmailNotification                 │
├──────────────────────────────────────────┤
│  Data Layer (22 datasets)                │
│  · Salesforce (accounts, pipeline,       │
│    contacts, opportunities)              │
│  · SPP time tracking                     │
│  · Gong call history                     │
│  · Health grades + signals               │
│  · Support cases                         │
│  · Employee timecards                    │
├──────────────────────────────────────────┤
│  State (14 AppDB collections)            │
│  · Audit trail                           │
│  · Config                                │
│  · Historical engagement data            │
│  · Metrics snapshots                     │
│  · Meeting prep docs + notes             │
│  · Portal meeting recaps                 │
└──────────────────────────────────────────┘
```

## Persona System

The application enforces different views of the same underlying data based on
who is using it. This isn't just CSS visibility — it's enforced at the data
query layer.

```
technical_ic     → own engagements only
engagement_owner → engagements they lead
scoping_team     → pipeline + pre-sales view
leader           → full portfolio + command center
admin            → configuration surface
csm              → customer health + relationship
client           → curated engagement view (external)
```

The `client` persona activates automatically for non-internal users. External
contacts get a portal view of their own engagement — milestones, team, key
decisions, meeting recaps — without access to internal commentary or other
accounts.

## Engineering Decisions (and Tradeoffs)

### AppDB for mutable state, datasets for analytical reads

AppDB collections handle all user-generated content (notes, engagement changes,
audit events, config). Domo datasets handle analytical reads (hours, pipeline,
health data). This separation keeps the write layer fast and the read layer
scalable.

**Tradeoff:** The AppDB has no row-level security. Client portal isolation is
enforced at the application layer. For truly sensitive multi-tenant scenarios,
this would need a proxy layer.

### Single Code Engine dispatcher

All CE functions route through a single dispatcher with `function_name` routing.
This avoids the manifest `functions[]` block that causes blank apps on publish
when package permissions aren't granted before the app loads.

**Tradeoff:** One CE package means one deployment unit. Adding a function
requires redeploying the whole package.

### AI degrades to rule-based

Every AI feature has a rule-based fallback. If the AI call fails (rate limit,
model unavailability, malformed response), the app surfaces the rule-based
output instead of an error state.

**Tradeoff:** Users may not notice AI degradation. This is intentional — a
broken AI feature that surfaces an error is worse UX than a functional
rule-based output.

## Capacity Planning Model

```
utilization = delivery_hours / work_capacity

where:
  delivery_hours = hours on ACTIVE engagements by delivery-role staff
  work_capacity  = target weekly hours × weeks in period

Managers (EM/PM) are excluded from utilization calculations.
"Planned scope hours" drives the denominator (not remaining hours —
  over-budget engagements were showing 0% utilization with remaining hours).
```

## Key Technical Challenges Solved

### Resource identity mismatch
Salesforce stores `owner_id` as numeric user IDs. SPP stores the same
person as email addresses. The People model resolves both and builds a
unified identity layer across all 22 datasets.

### SPP volume (580 → 123 active engagements)
Raw SPP data contained 580 "In Progress" engagements. Most were stale.
The fix: cross-reference against actual time entries in the last 90 days.
580 → 123 truly active.

### >10k join key trap
One ETL join was producing duplicate rows silently. Root cause: Magic ETL
has undocumented behavior when join keys exceed 10,000 distinct values.
Fix: moved to SQL Action tile with explicit deduplication.

## What I'd Build Next

**1. Native mobile client portal**
The current web-responsive approach works for mobile, but external clients
deserve a purpose-built mobile experience. The portal persona is already
isolated — the next step is a React Native shell targeting the same AppDB
collections.

**2. ML-based engagement risk scoring**
The current risk surface is rule-based (ARR + health grade + support case
volume + time since last touch). The training data from 18+ months of
engagement outcomes is now large enough to train a real risk model.

**3. Real-time collaborative editing**
Meeting prep notes currently use a last-write-wins pattern across AppDB.
Multi-user note editing deserves operational transformation or at minimum
a CRDT-style merge strategy.

**4. Automated SOW extraction → engagement setup**
`extractSOW` CE function already parses PDF SOWs for deliverables and hours.
The next step is wiring the extraction output directly to engagement creation,
eliminating the manual setup step entirely.

## Technologies

`React` · `TypeScript` · `Vite` · `Tailwind CSS` · `Framer Motion` ·
`Python` · `SQL` · `Salesforce` · `Domo` · `AppDB` · `Code Engine` ·
`Domo AI / DomoGPT` · `Magic ETL` · `Gong`
