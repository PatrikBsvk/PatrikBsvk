# Vibell — Credit System Spec

## Goal
Predictable pricing for users + protection against runaway API costs for Vibell.

## Plans

| Plan | Price (CZK / USD) | Credits/month | Notes |
|---|---|---|---|
| **Free** | 0 / $0 | 50 | Watermark, shared subdomain, sleeps after 24h idle |
| **Basic** | 349 / $15 | 200 | No watermark, custom subdomain on `vibell.app` |
| **Standard** | 899 / $39 | 1 000 | Priority queue, more storage |
| **Pro** | 2 290 / $99 | 3 000 | Custom domain, marketplace 80/20 split |
| **Team** | 5 990 / $259 | 9 000 | 5 seats, collaboration, shared workspace |

## Action → credit cost

| Action | Credits | Internal cost (USD, est.) |
|---|---|---|
| New project from template | 5 | $0.07 |
| AI text/color edit (small) | 1 | $0.015 |
| Click-to-edit component | 2 | $0.03 |
| 3 variants side-by-side | 3 | $0.045 |
| Generate full new section | 5 | $0.075 |
| DB schema change | 3 | $0.045 |
| Voice → wizard | 2 | $0.03 |
| Auto-save snapshot | 0 | (free) |
| Deploy / publish | 0 | (free) |

## Rules

- **Reset monthly**, do NOT roll over. Forces engagement, simplifies accounting.
- **Overage pack:** 100 credits for 199 CZK / $8.50, available to all paid plans.
- **Daily hard limit:** 200 actions/day (anti-abuse, even on Pro).
- **Transparency:** UI shows estimated cost BEFORE every action.
- **Soft warning:** Notify at 80% used, 95% used.
- **No surprise charges:** Free users hit a wall; paid users can opt in to overage.

## Internal economics

- **Target:** ~$0.015 USD actual API cost per credit (with prompt caching).
- **Gross margin target:** ≥ 60%, ideally ~80% on Basic.
- Higher-tier plans have lower per-credit margin (more credits given) but higher absolute revenue per user. Pro+ unlocks white-label/team features that justify the price.

## Database schema (sketch)

```sql
-- Plans (static, seeded)
CREATE TABLE plans (
  id              text PRIMARY KEY,         -- 'free', 'basic', 'standard', 'pro', 'team'
  display_name    text NOT NULL,
  price_usd_cents int  NOT NULL,
  monthly_credits int  NOT NULL,
  features        jsonb NOT NULL,           -- { watermark, custom_domain, marketplace_split, ... }
  stripe_price_id text,
  created_at      timestamptz DEFAULT now()
);

-- One row per user, current plan + period
CREATE TABLE subscriptions (
  user_id              uuid PRIMARY KEY REFERENCES auth.users,
  plan_id              text NOT NULL REFERENCES plans,
  stripe_subscription  text,
  status               text NOT NULL,       -- 'active', 'past_due', 'canceled'
  current_period_end   timestamptz,
  cancel_at_period_end boolean DEFAULT false,
  created_at           timestamptz DEFAULT now(),
  updated_at           timestamptz DEFAULT now()
);

-- Current credit balance, refreshed monthly
CREATE TABLE credit_balances (
  user_id           uuid PRIMARY KEY REFERENCES auth.users,
  credits_remaining int  NOT NULL,
  period_start      timestamptz NOT NULL,
  period_end        timestamptz NOT NULL,
  updated_at        timestamptz DEFAULT now()
);

-- Append-only log
CREATE TABLE credit_transactions (
  id            bigserial PRIMARY KEY,
  user_id       uuid NOT NULL REFERENCES auth.users,
  project_id    uuid,
  action        text NOT NULL,             -- 'new_project', 'ui_edit', 'variant', ...
  credits_delta int  NOT NULL,             -- negative = consumption, positive = grant/overage
  agent_call_id uuid,                      -- link to agent_calls
  created_at    timestamptz DEFAULT now()
);

-- Overage purchases
CREATE TABLE credit_packs (
  id              bigserial PRIMARY KEY,
  user_id         uuid NOT NULL REFERENCES auth.users,
  credits         int  NOT NULL,
  amount_usd_cents int NOT NULL,
  stripe_payment  text,
  created_at      timestamptz DEFAULT now()
);
```

Full schema (incl. projects, snapshots, marketplace) lives in `docs/architecture/database.md` (TBD).

## Stripe integration

- **Subscriptions:** one Stripe Price per plan (Basic, Standard, Pro, Team).
- **Overage:** Stripe Checkout one-off for 100-credit packs.
- **Marketplace payouts:** Stripe Connect (Express accounts for creators).
- **Webhooks:**
  - `customer.subscription.created/updated/deleted` → update `subscriptions`
  - `invoice.paid` → reset `credit_balances` for new period
  - `checkout.session.completed` → grant credit pack

## Edge cases

- **Plan downgrade mid-period:** keep current credits until period end, downgrade applies next period.
- **Plan upgrade mid-period:** prorate via Stripe; credits topped up immediately.
- **Failed payment:** mark `subscriptions.status='past_due'`, downgrade to free after 7 days.
- **Refunds:** full refund within 14 days if < 10 credits used (manual review).
