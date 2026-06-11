# FlowSentry — Product & Build Specification

**One-liner:** Client-facing monitoring, alerting, and reporting for automation agencies running n8n workflows.

**Working name:** "FlowSentry" is a placeholder. Run a trademark/domain check before launch.

---

## 1. The Problem (and why people pay)

AI automation agencies and freelancers build n8n workflows for clients and charge monthly retainers ($500–$5,000/client). Their two recurring pains:

1. **Silent failures.** A client's lead-routing workflow breaks on a Friday, nobody notices until Monday, the client is furious, the retainer is at risk. n8n's built-in error handling requires building an error workflow per instance and gives no cross-client view.
2. **Proving value.** Retainers churn when clients ask "what am I paying for?" Agencies have no clean way to show "your automations ran 4,212 times this month and saved you ~38 hours."

FlowSentry is the agency's control tower: every client, every workflow, one screen — plus instant failure alerts and white-labeled weekly reports the agency sends to clients under its own brand.

**Why this buyer:** agencies have revenue, a recurring cost they can pass through, and a tool budget. A $79/mo tool that protects a $3,000/mo retainer is an easy sale. This is not a consumer product hoping for volume.

**Why n8n first:** the n8n agency ecosystem is large and growing, self-hosted instances have no centralized monitoring, and n8n exposes both a REST API and a global error-workflow hook — so integration is genuinely buildable in days, not months. Make and Zapier connectors come later (both support outbound webhooks, so the ingestion model extends).

**Competitive landscape (verified June 2026) and the wedge:** this niche is no longer empty. Administrate (administrate.dev) monitors multi-instance n8n grouped by client, with LLM cost tracking per client as its hook. FlowMonix offers 5-minute sync, incident grouping, AI root-cause analysis, status pages, and one-click retry. FlowEngine sells white-label n8n *hosting* with branded client panels. n8nhackers ships a self-hosted monitoring agent, and indie builders are actively launching beta tools in the n8n community forum. n8n itself exposes health endpoints and is adding dashboards on enterprise tiers. Conclusion: pure failure-alerting for n8n is becoming table stakes — it validates demand but is not a moat.

FlowSentry's wedge is therefore positioning, not feature count: (1) **report-first** — the white-label weekly client report is the hero product and monitoring is its data source; every competitor leads with monitoring or cost tracking, none leads with the artifact that renews the agency's retainer; and (2) **multi-platform** — agencies run mixed n8n/Make/Zapier stacks, all current rivals are n8n-only, so the Make connector moves up to a v1.1 fast-follow (month 2–3, not "someday"). The ingestion model (token webhook + per-platform poller) was designed for this from day one. Marketing, landing copy, and onboarding should all reflect report-first framing.

---

## 2. Target Customer & $10K MRR Math (no fantasy)

**ICP (ideal customer profile):**
- Automation agencies (1–10 people) and solo freelancers
- Running 3–30 client accounts on self-hosted or cloud n8n
- Found in: n8n community forum, r/n8n, AI-automation Discord/Skool communities, Upwork "n8n expert" sellers, X/LinkedIn automation builders

**Pricing (see §5):** Solo $29 / Agency $79 / Scale $149 per month.

**Example mix at $10K MRR:**

| Plan | Customers | MRR |
|---|---|---|
| Solo ($29) | 45 | $1,305 |
| Agency ($79) | 85 | $6,715 |
| Scale ($149) | 14 | $2,086 |
| **Total** | **144** | **$10,106** |

**Funnel assumptions (conservative for a high-intent niche):**
- Visitor → trial: 3%
- Trial → paid: 25%
- Monthly churn: 5% (agencies churn when they lose clients)

To reach and hold 144 customers you need roughly 575 cumulative trials → ~19,000 targeted visitors over the growth period. At 5% churn, steady state requires ~7 new paying customers/month just to stand still. **Realistic timeline to $10K MRR: 12–18 months** with consistent distribution (content in n8n communities, template marketplace presence, partnerships with automation course creators). If that distribution work doesn't happen, the product alone will not get there. That's the honest constraint.

---

## 3. MVP Feature Set (build exactly this, nothing more)

### In scope (v1)
1. **Connect n8n instance** — base URL + API key; verify via `GET /api/v1/workflows`; one-click install of a FlowSentry "error workflow" template for instant failure pushes.
2. **Workflow registry** — auto-import workflows from connected instances; toggle monitoring per workflow; set `expected frequency` (e.g., "should run at least every 6h") and `minutes saved per run` (powers reports).
3. **Client workspaces** — group workflows under named clients; per-client logo and brand color.
4. **Execution ingestion** — push (error workflow → webhook) for failures, poll (n8n executions API, every 5 min) for everything else. Daily rollups for fast dashboards.
5. **Alerting** — email + Slack incoming-webhook channels. Triggers: single failure, N consecutive failures, "no runs in expected window." Cooldowns to prevent alert storms.
6. **Incidents** — failure groups with open/acknowledged/resolved states and a notes field.
7. **White-label client reports** — auto-generated weekly/monthly HTML email per client workspace: runs, success rate, hours saved, incidents resolved. Sent from "Reports — {Agency Name}".
8. **Client status page** — public tokenized URL per client workspace showing last-30-day health per workflow (no login required; share with the client).
9. **Billing** — Stripe Checkout + customer portal, 14-day trial (card required), plan limits enforced.

### Explicitly NOT in v1 (cut list — resist these)
- Make/Zapier/Pipedream connectors in v1 itself — but the **Make connector is now a committed v1.1 fast-follow (month 2–3)**, since multi-platform is half the competitive wedge; design the connections table and ingestion path with `type='make'` in mind from week 2
- Custom domains for status pages
- PDF report export
- Team roles beyond owner/member
- Public API & webhooks out
- AI failure diagnosis ("why did this break")
- SSO / SAML
- Mobile app

Each cut item is a future upsell, not an MVP requirement.

---

## 4. User Flows

### 4.1 Onboarding (target: first alert configured in <10 minutes)

1. **Sign up** — email + password or Google OAuth. Collect agency name → creates `organization`.
2. **Connect n8n** — form: instance base URL, API key (n8n Settings → API). On submit, server calls `GET {base_url}/api/v1/workflows?limit=1` with `X-N8N-API-KEY` header. Success → connection saved (key encrypted), workflows imported in background. Failure → inline error with the three common causes (wrong URL, API disabled, key invalid).
3. **Install error hook** — show a copy-paste n8n workflow JSON (an Error Trigger node + HTTP Request node POSTing to the org's unique ingest URL) with a 30-second GIF: *Import → Save → Set as default error workflow in instance settings.* A "Test it" button fires a sample failure and confirms receipt live on screen.
4. **Create first client** — name + optional logo. Multi-select imported workflows → assign to client.
5. **Set alert channel** — email pre-filled with signup address; optional Slack webhook URL with "send test" button.
6. **Done screen** — checklist recap + "Your first weekly report goes out Monday 8am. Preview it now →"

Empty states everywhere deep-link back to the missing step (e.g., dashboard with no connection shows step 2 inline).

**Onboarding email sequence (transactional, via Resend):**
- Day 0: welcome + setup checklist link
- Day 2 (if no connection): "2-minute n8n connection guide" 
- Day 5: "Send your first client report" walkthrough
- Day 11: trial ending reminder + what they'll lose (active monitors count)
- Day 14: trial ended → grace banner, data retained 30 days

### 4.2 Failure → Alert → Resolve

1. Client workflow fails in n8n → error workflow fires → POST to `/api/ingest/{ingest_token}` (<1s after failure).
2. Ingest endpoint validates token, writes execution row, evaluates alert rules.
3. Rule matches and cooldown clear → create/update `incident`, dispatch to channels: *"🔴 [Acme Corp] 'Lead Router' failed — Error: Invalid credentials at HubSpot node. 2nd failure in a row. View incident →"*
4. Agency clicks through → incident page: error message, recent execution history, link out to the execution in their n8n instance, Acknowledge / Resolve buttons, notes field.
5. Resolution timestamps feed the client report ("3 issues caught and fixed, median response 22 min").

### 4.3 Weekly report

1. Cron (Mon 08:00 org-local) per client workspace with reports enabled.
2. Aggregate from `workflow_daily_stats`: total runs, success rate, hours saved (Σ runs × minutes_saved_per_run ÷ 60), incidents opened/resolved, top workflows by volume.
3. Render HTML email with agency logo/brand color; send to configured recipients (the client's email), reply-to = agency owner.
4. Store rendered report; list in dashboard with resend/preview.

### 4.4 No-run detection ("dead workflow")

Every 10 min, a cron scans monitored workflows where `expected_frequency` is set and `last_execution_at < now() - expected_frequency` → opens an incident of type `stale`, alerts: *"⚠️ [Acme Corp] 'Daily Invoice Sync' hasn't run in 26h (expected: every 24h)."* This catches the silent killer: workflows that stop triggering entirely — which produce no error event at all.

---

## 5. Pricing

| | **Solo — $29/mo** | **Agency — $79/mo** | **Scale — $149/mo** |
|---|---|---|---|
| Team members | 1 | 5 | Unlimited |
| Client workspaces | 3 | 15 | Unlimited |
| Monitored workflows | 25 | 150 | 500 |
| n8n connections | 1 | 5 | 15 |
| Alert channels | Email | Email + Slack | Email + Slack |
| Client reports | FlowSentry-branded | **White-label** | White-label |
| Status pages | 1 | Per client | Per client |
| Execution history | 14 days | 60 days | 180 days |

- Annual: 2 months free (Solo $290/yr, Agency $790/yr, Scale $1,490/yr).
- 14-day trial on Agency plan by default (trial the plan you want them to keep), card required — filters tire-kickers, and this ICP has cards.
- No free plan: support load on a solo founder is the scarcest resource. The status page (client-visible, "Powered by FlowSentry" on Solo) is the organic growth loop instead.
- White-label gated to Agency+ deliberately — it's the feature agencies actually need, and the main upgrade driver from Solo.

---

## 6. Database Schema (PostgreSQL)

Conventions: UUID v7 primary keys, `created_at timestamptz default now()` on every table, soft deletes only where noted. Multi-tenancy enforced by `org_id` on every tenant table + Postgres RLS if using Supabase.

```sql
-- ============ Identity & tenancy ============
create table users (
  id            uuid primary key,
  email         citext unique not null,
  name          text,
  password_hash text,                -- null if OAuth-only
  created_at    timestamptz not null default now()
);

create table organizations (
  id                     uuid primary key,
  name                   text not null,
  slug                   text unique not null,
  logo_url               text,
  brand_color            text default '#4F46E5',
  plan                   text not null default 'trial',   -- trial|solo|agency|scale|canceled
  trial_ends_at          timestamptz,
  stripe_customer_id     text unique,
  stripe_subscription_id text,
  ingest_token           text unique not null,            -- random 32-byte, for webhook URL
  timezone               text not null default 'America/Los_Angeles',
  created_at             timestamptz not null default now()
);

create table org_members (
  org_id     uuid references organizations(id) on delete cascade,
  user_id    uuid references users(id) on delete cascade,
  role       text not null default 'member',              -- owner|member
  primary key (org_id, user_id)
);

-- ============ Clients & connections ============
create table client_workspaces (
  id          uuid primary key,
  org_id      uuid not null references organizations(id) on delete cascade,
  name        text not null,
  logo_url    text,
  brand_color text,
  contact_email text,
  archived_at timestamptz,
  created_at  timestamptz not null default now()
);

create table connections (
  id                uuid primary key,
  org_id            uuid not null references organizations(id) on delete cascade,
  type              text not null default 'n8n',           -- future: make|zapier
  label             text not null,
  base_url          text not null,
  api_key_encrypted text not null,                         -- AES-256-GCM, key in env/KMS
  status            text not null default 'active',        -- active|unreachable|revoked
  last_synced_at    timestamptz,
  poll_cursor       text,                                  -- last seen n8n execution id
  created_at        timestamptz not null default now()
);

-- ============ Workflows & executions ============
create table workflows (
  id                    uuid primary key,
  org_id                uuid not null references organizations(id) on delete cascade,
  connection_id         uuid not null references connections(id) on delete cascade,
  client_workspace_id   uuid references client_workspaces(id) on delete set null,
  external_id           text not null,                     -- n8n workflow id
  name                  text not null,
  is_monitored          boolean not null default false,
  active_in_n8n         boolean not null default true,
  expected_run_interval interval,                          -- null = no stale detection
  minutes_saved_per_run numeric(6,1) default 0,
  last_execution_at     timestamptz,
  last_status           text,                              -- success|error|stale
  created_at            timestamptz not null default now(),
  unique (connection_id, external_id)
);

create table executions (
  id                    uuid primary key,
  org_id                uuid not null,                     -- denormalized for RLS/retention
  workflow_id           uuid not null references workflows(id) on delete cascade,
  external_execution_id text,
  status                text not null,                     -- success|error
  source                text not null,                     -- push|poll
  started_at            timestamptz not null,
  finished_at           timestamptz,
  duration_ms           integer,
  error_message         text,
  error_node            text,
  created_at            timestamptz not null default now(),
  unique (workflow_id, external_execution_id)
);
create index executions_wf_time on executions (workflow_id, started_at desc);
create index executions_org_time on executions (org_id, started_at desc); -- retention sweeps

-- Pre-aggregated for dashboards & reports (updated on ingest + nightly reconcile)
create table workflow_daily_stats (
  workflow_id   uuid references workflows(id) on delete cascade,
  day           date not null,
  runs          integer not null default 0,
  successes     integer not null default 0,
  failures      integer not null default 0,
  total_duration_ms bigint not null default 0,
  primary key (workflow_id, day)
);

-- ============ Alerting ============
create table alert_channels (
  id         uuid primary key,
  org_id     uuid not null references organizations(id) on delete cascade,
  type       text not null,                                -- email|slack_webhook
  config     jsonb not null,                               -- {email} or {webhook_url}
  is_verified boolean not null default false,
  created_at timestamptz not null default now()
);

create table alert_rules (
  id               uuid primary key,
  org_id           uuid not null references organizations(id) on delete cascade,
  scope            text not null default 'org',            -- org|client|workflow
  scope_id         uuid,                                   -- null when scope='org'
  trigger          text not null,                          -- on_failure|consecutive_failures|stale
  threshold        integer not null default 1,             -- e.g. 3 consecutive
  cooldown_minutes integer not null default 30,
  channel_ids      uuid[] not null,
  is_enabled       boolean not null default true,
  created_at       timestamptz not null default now()
);

create table incidents (
  id              uuid primary key,
  org_id          uuid not null references organizations(id) on delete cascade,
  workflow_id     uuid not null references workflows(id) on delete cascade,
  kind            text not null,                           -- failure|stale
  status          text not null default 'open',            -- open|acknowledged|resolved
  failure_count   integer not null default 1,
  first_seen_at   timestamptz not null,
  last_seen_at    timestamptz not null,
  acknowledged_at timestamptz,
  resolved_at     timestamptz,
  resolved_by     uuid references users(id),
  last_error      text,
  notes           text,
  created_at      timestamptz not null default now()
);
create index incidents_org_open on incidents (org_id) where status != 'resolved';

create table alert_events (
  id          uuid primary key,
  incident_id uuid not null references incidents(id) on delete cascade,
  rule_id     uuid references alert_rules(id) on delete set null,
  channel_id  uuid references alert_channels(id) on delete set null,
  status      text not null,                               -- sent|failed
  sent_at     timestamptz not null default now()
);

-- ============ Reports & status pages ============
create table report_settings (
  id                  uuid primary key,
  client_workspace_id uuid unique not null references client_workspaces(id) on delete cascade,
  cadence             text not null default 'weekly',      -- weekly|monthly|off
  send_hour_local     integer not null default 8,
  recipients          text[] not null default '{}',
  is_white_label      boolean not null default false,      -- enforced by plan at send time
  reply_to            text
);

create table reports (
  id                  uuid primary key,
  org_id              uuid not null references organizations(id) on delete cascade,
  client_workspace_id uuid not null references client_workspaces(id) on delete cascade,
  period_start        date not null,
  period_end          date not null,
  html                text not null,                       -- rendered snapshot
  stats               jsonb not null,                      -- {runs, success_rate, hours_saved,...}
  sent_at             timestamptz,
  created_at          timestamptz not null default now()
);

create table status_pages (
  id                  uuid primary key,
  client_workspace_id uuid unique not null references client_workspaces(id) on delete cascade,
  public_token        text unique not null,                -- random 24-byte url token
  is_enabled          boolean not null default false,
  show_history_days   integer not null default 30
);
```

**Retention job (nightly):** delete `executions` older than plan limit (14/60/180 days) per org; `workflow_daily_stats` is kept forever (tiny). This keeps the hot table bounded — the one genuine scaling concern in this product.

---

## 7. API Routes

Next.js App Router (`app/api/...`). All `/api/v1/*` routes require session auth + org scoping; mutations validated with Zod. The two public endpoints are token-authenticated.

### Public (token auth)
| Method | Route | Purpose |
|---|---|---|
| POST | `/api/ingest/:ingestToken` | Receive execution events from n8n error workflow (and future platforms). Validates token → org, upserts execution, updates rollup, evaluates alert rules. Must respond <300ms; alert dispatch goes to a queue. |
| GET | `/api/status/:publicToken` | JSON powering the public client status page (SSR page consumes it server-side). |

### Auth & org
| Method | Route | Purpose |
|---|---|---|
| POST | `/api/auth/signup` `/login` `/logout` | Or delegate entirely to Auth.js/Supabase Auth. |
| GET/PATCH | `/api/v1/org` | Org profile, branding, timezone. |
| GET/POST/DELETE | `/api/v1/org/members` | Invite by email, remove. Plan-limit enforced. |

### Connections & workflows
| Method | Route | Purpose |
|---|---|---|
| POST | `/api/v1/connections` | Create: server-side test call to n8n before saving; encrypt key. |
| POST | `/api/v1/connections/:id/sync` | Manual re-import of workflows. |
| GET/DELETE | `/api/v1/connections[/:id]` | List / revoke. |
| GET | `/api/v1/workflows?client_id=&monitored=` | List with filters + health summary. |
| PATCH | `/api/v1/workflows/:id` | Assign client, toggle `is_monitored`, set `expected_run_interval`, `minutes_saved_per_run`. |
| GET | `/api/v1/workflows/:id/executions?limit=50&cursor=` | Execution feed (keyset pagination). |

### Clients, incidents, alerts
| Method | Route | Purpose |
|---|---|---|
| GET/POST/PATCH/DELETE | `/api/v1/clients[/:id]` | Client workspace CRUD (delete = archive). |
| GET | `/api/v1/clients/:id/overview` | Aggregates for client detail page. |
| GET | `/api/v1/incidents?status=open` | Incident list. |
| PATCH | `/api/v1/incidents/:id` | Acknowledge / resolve / notes. |
| GET/POST/DELETE | `/api/v1/alert-channels[/:id]` | Channels; POST sends verification test. |
| GET/POST/PATCH/DELETE | `/api/v1/alert-rules[/:id]` | Rules CRUD. |

### Reports, status pages, billing
| Method | Route | Purpose |
|---|---|---|
| GET/PATCH | `/api/v1/clients/:id/report-settings` | Cadence, recipients, white-label toggle. |
| POST | `/api/v1/clients/:id/reports/preview` | Render current period without sending. |
| GET / POST | `/api/v1/reports` · `/api/v1/reports/:id/resend` | History / resend. |
| PATCH | `/api/v1/clients/:id/status-page` | Enable, regenerate token. |
| POST | `/api/v1/billing/checkout` | Stripe Checkout session for plan. |
| POST | `/api/v1/billing/portal` | Stripe customer portal session. |
| POST | `/api/webhooks/stripe` | Signature-verified; handles subscription created/updated/deleted → sets `organizations.plan`. |

### Background jobs (Inngest or cron + queue)
| Job | Schedule | Work |
|---|---|---|
| `poll-executions` | every 5 min per active connection | `GET /api/v1/executions?limit=100&lastId={cursor}` against n8n; upsert (dedup on `external_execution_id` makes push+poll overlap safe); update rollups + `last_execution_at`. |
| `stale-scan` | every 10 min | Open `stale` incidents for overdue monitored workflows; auto-resolve when a new run arrives. |
| `send-reports` | hourly | Send reports where org-local time matches settings. |
| `retention-sweep` | nightly | Delete old executions per plan. |
| `trial-lifecycle` | daily | Trial emails, downgrade expired trials. |

**n8n integration facts the developer needs:** auth is the `X-N8N-API-KEY` header; key endpoints are `GET /api/v1/workflows` and `GET /api/v1/executions` (supports `limit` and `lastId` cursor); failures are pushed via a workflow that starts with an **Error Trigger** node (set as the instance's default error workflow), which receives workflow name/id, execution id/url, and error message — wire those into the HTTP Request node body. Verify exact field names against the n8n API docs for the version you target during week 2; pin the minimum supported n8n version in the connection UI.

---

## 8. Dashboard Spec (pages & components)

Stack assumption: Next.js App Router + Tailwind + shadcn/ui. Sidebar layout: Overview · Clients · Incidents · Workflows · Reports · Status Pages · Settings (Connections, Alerts, Team, Billing).

### 8.1 Overview (home)
- **Top stat row:** Workflows monitored · Runs (7d) · Success rate (7d) · Open incidents (red if >0) · Hours saved (7d).
- **Health grid:** one card per client workspace — name, logo, sparkline of daily runs (14d), success %, worst workflow. Click → client detail. This grid IS the product's screenshot; make it the best screen.
- **Open incidents strip** pinned above the grid when non-empty.
- Empty state: 3-step setup checklist (connect → assign → alert channel) with progress.

### 8.2 Client detail
- Header: client name/logo, status page link (copy), "Preview report" button.
- Stats for selectable range (7/30/90d): runs, success rate, hours saved, incidents.
- Workflow table: name · monitored toggle · last run (relative) · 24h/7d success bars · expected interval · minutes-saved (inline edit) · status dot (green/red/amber=stale).
- Incident history tab.

### 8.3 Workflow detail
- Run-history chart (daily stacked success/failure, 30d) from rollups.
- Execution feed (50, paginated): time, status, duration, error preview; row expands to full error + deep link to the execution in the user's n8n instance (`{base_url}/workflow/{external_id}/executions/{external_execution_id}`).
- Settings panel: client assignment, expected interval, minutes saved, per-workflow alert override.

### 8.4 Incidents
- Filterable table (status, client): severity dot, workflow, client, kind (failure/stale), count, first/last seen, age. Row → drawer with error detail, timeline (opened → alerts sent → acknowledged → resolved), notes, action buttons.

### 8.5 Reports
- Per-client cadence settings + recipients (chips input) + white-label toggle (locked with upgrade prompt on Solo).
- Sent-report history with HTML preview modal and resend.

### 8.6 Status page (public, no auth)
- Agency logo + client name, overall status banner ("All systems operational" / "1 workflow degraded"), per-workflow row with 30-day bar (one segment per day, green/red/grey), last-updated stamp. Solo plan footer: "Powered by FlowSentry" (the growth loop).

### 8.7 Settings
- **Connections:** list with status dot + last sync; add-connection modal (URL, key, test button); per-connection "reinstall error workflow" helper.
- **Alert channels:** email/Slack list, add + test.
- **Alert rules:** sane default created at signup (org-wide, on_failure, 30-min cooldown); advanced rules behind "Add rule".
- **Billing:** current plan, usage vs limits (workflows, clients, members), Stripe portal link.

---

## 9. Landing Page (structure + actual copy)

Single page + /pricing. Ship with real product screenshots only — no illustrations of fake UI.

**Hero**
> **Know your client workflows broke — before your clients do.**
> FlowSentry watches every n8n workflow across all your clients, alerts you the second one fails, and sends each client a white-labeled report proving the value of your retainer.
> [Start 14-day trial] · [See a live status page →]
> *Connect your n8n instance in under 10 minutes. Card required, cancel in two clicks.*

**Social proof bar** (post-beta): "Monitoring 4,000+ client workflows for automation agencies." Use real numbers only; omit the bar until they exist.

**Problem section**
> **The Friday-night failure.**
> A credential expires. A webhook dies. Your client's lead pipeline silently stops — and you find out Monday, from an angry email. n8n won't tell you. Your client shouldn't have to.

**Three-pillar features**
1. **One screen, every client.** Stop tab-cycling through n8n instances. See the health of every workflow you've ever shipped, grouped by client.
2. **Alerts that wake you up (politely).** Instant Slack or email when a workflow fails — or when it *stops running* and fails silently. Cooldowns included; alert storms aren't.
3. **Reports that renew retainers.** Every Monday, your clients get a branded email: runs completed, hours saved, issues you caught and fixed. Your logo, not ours.

**How it works (3 steps):** Connect n8n (URL + API key) → Import the error hook (copy-paste, 30 seconds) → Assign workflows to clients. Done.

**Pricing section:** the §5 table, annual toggle.

**FAQ**
- *Does this work with self-hosted n8n?* Yes — self-hosted and n8n cloud. We need API access to your instance; your workflow data stays in your n8n.
- *Do you store my client data?* We store execution metadata (status, timing, error messages) — never the data flowing through your workflows.
- *Make/Zapier?* n8n today; Make is next on the roadmap. Tell us what you run.
- *What happens after the trial?* You pick a plan or your monitors pause. Data kept 30 days.

**Footer CTA:** "Your retainer is worth protecting. **Start monitoring in 10 minutes →**"

---

## 10. MVP Implementation Plan (6 weeks, one full-stack developer)

**Stack:** Next.js (App Router, TS) · Postgres via Supabase (auth + RLS + storage) · Inngest (jobs/queues) · Stripe · Resend (email) · Vercel. Total infra cost at launch: ~$45–90/mo. Everything here has a generous free tier to start.

**Week 1 — Foundation**
Repo, CI, Supabase project; schema migrations (§6); auth + org creation; app shell with sidebar; Stripe products/prices, checkout, webhook → plan column; plan-limit middleware. *Exit: sign up, subscribe in test mode, see empty dashboard.*

**Week 2 — Ingestion (the core)**
Connections CRUD with live n8n test + key encryption; workflow import; ingest endpoint (token auth, upsert, rollup update); error-workflow JSON template + "Test it" flow; `poll-executions` job with cursor + dedup. *Exit: a real n8n failure appears in the DB within seconds; successes within 5 minutes. This week proves the product; do it against a real n8n instance from day one.*

**Week 3 — Dashboard read paths**
Client workspaces CRUD + workflow assignment; Overview with stat row + health grid; client detail; workflow detail with execution feed; rollup-backed charts. *Exit: demo-able product.*

**Week 4 — Alerting & incidents**
Alert channels (email/Slack) + verification; rule engine on ingest path (queued dispatch); incident lifecycle + UI; `stale-scan` job; cooldowns. *Exit: kill a credential in n8n → Slack ping <60s; pause a scheduled workflow → stale alert.*

**Week 5 — Reports & status pages**
Report renderer (React Email) + hourly sender; preview/resend; settings UI; public status page (SSR, cache 60s); retention sweep; white-label plan gate. *Exit: receive a real branded Monday report; status page is shareable.*

**Week 6 — Polish & launch**
Onboarding checklist + empty states; trial-lifecycle emails; landing page + pricing; error tracking (Sentry) + uptime monitor on the ingest endpoint (eat your own dog food: monitor it with Cronitor or similar); seed 10 design-partner agencies from n8n community/Discord at 50% off for 6 months in exchange for feedback + testimonials. *Exit: first 10 trials in.*

**Definition of MVP-done:** an agency can go signup → connected → assigned → alerted → first client report received, with zero founder intervention.

---

## 11. Risks & Honest Caveats

1. **Distribution is the actual hard part.** This document de-risks the build, not the sales. Budget 50% of post-launch time for n8n forum/Discord presence, template giveaways (free "error workflow + monitoring starter kit"), YouTube tutorials with automation-course creators, and an affiliate cut for course owners. No distribution → no 144 customers, regardless of product quality.
2. **Competition is real, not hypothetical:** Administrate and FlowMonix already sell n8n monitoring to this exact buyer, and n8n could ship native multi-instance dashboards. Feature-matching them is a losing game for a solo founder. Win on positioning (report-first — the retainer-renewal artifact nobody else leads with), multi-platform coverage by v1.1, and speed in community distribution. Re-check both competitors' pricing and features before launch and position explicitly against them on the landing page's FAQ.
3. **Polling load:** 144 orgs × ~3 connections × 12 polls/hr is trivial (~5k req/hr), but respect per-instance failures — back off unreachable instances and surface "connection unreachable" as its own alert (it's information the agency wants anyway).
4. **Support load:** self-hosted n8n setups vary (reverse proxies, auth in front of the API). The connection-failure UX in §4.1 and a single great troubleshooting doc will absorb most tickets.
5. **Churn floor:** agencies that lose clients churn through no fault of the product. Annual plans and the status-page lock-in (clients bookmark them) are the levers.
6. **Verify before building:** n8n API field names/limits against current docs (week 2, day 1); Stripe tax handling for your situation; the name/domain.

*Spec version 1.0 — June 2026. Everything in §6–§7 is implementation-ready but should be treated as the contract to refine during week 1–2, not gospel.*
