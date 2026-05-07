# Leading Vibell — PM Playbook for Patrik

> Personalized founder playbook. Not generic startup advice — written for **a solo OSVČ founder, premium-aesthetic builder, working with a local Claude Code agent, cost-conscious, single-language non-native English market**. Update this doc as we learn what works.

---

## 1. Operating cadence — your weekly rhythm

A solo founder has one resource: **focused hours per week**. The cadence below assumes ~15–20 deep hours/week (typical alongside other obligations). Adjust to your reality.

### Weekly ritual (60 min total)
| When | What | Why |
|---|---|---|
| **Monday morning, 30 min** | PM check-in with me (this session). Review `asistenta.md`, `decisions-log.md`. Pick 1–3 outcomes for the week. | One clear focus beats a long todo list. |
| **Mid-week, 5 min** | Mid-week pulse — open the active task, glance at progress, check for blockers. | Catch blockers early, not on Friday. |
| **Friday afternoon, 25 min** | Weekly review: what shipped, what learned, what slipped. Update `decisions-log.md` if direction shifted. Plan Monday. | Compounding learnings; protected weekend. |

### Daily ritual (5 min)
- Open the **active task `.md`** before any work session. Read its acceptance criteria. Match your day's effort to that scope, not adjacent tempting work.

### Monthly ritual (60 min, last Friday)
- "Founder review" against `metrics.md` (we'll build this together). Are you closer to shipping MVP than 4 weeks ago?
- Re-read `asistenta.md` Section 4 (scope). If a feature has crept in that wasn't there last month, kill it.

### Ritual-buster — one rule
If you skip Monday's PM check-in, the next session you reopen Vibell **starts with that 30 min**, not with coding. Discipline beats velocity.

---

## 2. Founder time — what only YOU can do

You're using a local Claude Code Opus 4.7 to write actual code. That frees your time for things only the founder can do. Spend your hours here:

### Highest leverage (60 % of your time)
- **Talk to potential users** — minimum 2 conversations / week. Non-programmers with ideas. What do they struggle with? What would they pay for?
- **Build relationships with first 10 customers** — not 100, not 1 000. Ten. Know them by name.
- **Make strategic decisions** — features in/out, pricing, brand, partnerships. Don't delegate these to me; I propose, you decide.
- **Watch real users use Vibell live** — once you have a demo, sit next to a non-programmer using it. You'll learn more in 1 hour than from 100 ideas in your head.

### Medium leverage (25 % of your time)
- **Write the public story** — Twitter/X, LinkedIn, indie communities. Build in public; documenting the journey IS marketing for premium products.
- **Recruit beta users** — DM 5 people / week from communities of non-programmer creators (IndieHackers, design Twitter, Czech maker communities).
- **Review what the local Claude Code agent built** — at least skim the diff; don't accept code blindly.

### Lower leverage (15 % of your time, automate or delegate)
- Routine docs / writing → delegate to me (this PM session)
- Code → delegate to local Opus
- Repetitive setup (Stripe, domains, accounts) → batch once a month, don't snack

### What to NEVER spend time on at this stage
- Custom logo design (Geist wordmark + violet works — defer to designer in Phase 2)
- Pixel-perfect marketing site polish before MVP works
- Optimizing CI / DevOps (you're solo, GitHub Actions free tier is enough)
- Reading every "AI app builder" competitor's product update
- Generic startup books (read 1 if you must, then stop)

---

## 3. Validation checkpoints — before each milestone

Don't build a milestone without passing its checkpoint. If you can't pass, the milestone scope is wrong, not the validation.

| Before shipping | Question to answer | How |
|---|---|---|
| **M2 (Wizard + Builder)** | Will 5 non-programmers complete the wizard without a confused face? | Watch 5 people try it on a video call. Iterate copy + flow. |
| **M3 (Click-to-edit)** | Can a non-programmer change their hero text + color in <2 min, first try? | Same — watch 5 attempts. p50 ≤ 2 min, p95 ≤ 5 min, otherwise reshape UX. |
| **M4 (Deploy + SMB enablers)** | Will at least 2 of those 5 publish + share their app voluntarily? | Don't ask them to share — see if they do it themselves. Voluntary share = product-market fit signal. |
| **M5 (Billing + soft launch)** | Will at least 3 of 50 beta users pay you on day 1 of billing? | Stripe Checkout, real money. Friends-buying-out-of-pity does NOT count. |

If any checkpoint fails, **pause shipping the next milestone** and fix the gap. Common temptation: "we'll fix it later". Later never comes.

---

## 4. Risk register — top 10 things that can kill Vibell

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | **Generated apps look "AI-generic"** (every Vibell app feels the same) | High | Critical | Lock layouts, vary content via Builder; invest in great template variants; manual QA every template before public ship |
| 2 | **Token cost runaway** (one user burns $50/month, we charge $15) | Medium | Critical | Hard daily limits per plan, prompt caching, monitoring + alerts in `agent_calls` |
| 3 | **WebContainers reliability** (preview broken → user can't edit) | Medium | High | Add server-side preview fallback (Vercel preview deploys) for paid tiers |
| 4 | **Builder Agent fails on uncommon wizard combos** | High | High | Per-template zod schema with retry; smart-error UX; manual QA matrix of 20 wizard combinations |
| 5 | **Solo-founder burnout** | Medium | Critical | 15–20 hours/week max, one full day off/week, ship 1 thing per week not 5 |
| 6 | **Premium pricing → market resistance** ("I'll just use Lovable for $20") | Medium | High | Clear differentiation in landing copy; show, don't tell, with live demo on landing |
| 7 | **Anthropic API outage / rate limits** | Low | High | Multi-model fallback; queue persistent jobs; status page from day 1 |
| 8 | **GDPR / data subjects requests in EU** | Medium | Medium | Privacy policy ship before public launch; "delete my data" endpoint; per-user data export |
| 9 | **A bigger player (Vercel, OpenAI) ships a Vibell-like product** | Medium | Critical | **Moat = guide system + memory + creator network**. Ship Phase 2 fast once Phase 1 validates. |
| 10 | **Patrik over-builds Phase 2 features in Phase 1** | Already happened once | Medium | This playbook + monthly scope review; PM (Claude) flags every "while we're at it" temptation |

Update this list quarterly. Risk #5 (burnout) and #10 (scope creep) are the ones that come from inside — most likely killers.

---

## 5. Metrics that matter

### North-star metric (the one number)
**Apps published per active user per month.** Captures both engagement and core value delivery. Target: ≥ 1.0 by end of M5.

### Leading indicators (predict north star)
| Metric | Target by M5 |
|---|---|
| Wizard completion rate | ≥ 75 % |
| Time from sign-up to first published app | p50 ≤ 30 min |
| Click-to-edit p50 latency | ≤ 8 s |
| Builder Agent success rate (no smart-error fallback) | ≥ 90 % |
| Free → Basic conversion at month 1 | ≥ 5 % |
| NPS (post-publish survey) | ≥ 30 |

### Lagging indicators (financial)
| Metric | Target |
|---|---|
| Monthly Recurring Revenue (MRR) at end of M5 | ≥ 5 000 CZK |
| Gross margin | ≥ 70 % (token cost target) |
| Cash burn (your time × hourly rate equivalent) | track for sanity |

### Counter-metrics (watch for trouble)
- Average tokens per published app (should drop over time as we cache better)
- Support tickets per user / week
- "Smart error" rate (Builder failures, Studio rollbacks)

We'll wire these up in M5 (Built-in analytics for Vibell itself, not just user apps).

---

## 6. Distribution strategy — build in public

Premium products **need a story**, not paid ads. Solo founders + premium aesthetic = build in public is your distribution.

### Channels (in priority)
1. **Twitter/X** — daily 1 thing learned, weekly progress thread. Tag @vercel @anthropicai when relevant. Aim for 1 viral thread / month.
2. **LinkedIn** — Czech network, business audience, longer-form posts. 2× / week.
3. **IndieHackers / Reddit r/SideProject** — milestone announcements only, not constant promotion.
4. **Czech maker communities** (Maker Tour, kdyžto…) — your home market is also your test market. Respect it.

### Cadence
- 1 post / day on the active build channel (you pick X or LinkedIn — don't try both daily)
- 1 long-form / week on the other
- Don't post for engagement; post for **build journal that doubles as marketing**

### Content angles (rotate)
- Decision being made (e.g., "should Studio Mode unlock cost 8 or 15 credits? Here's the trade-off…")
- Before/after of a UX choice
- Real generated app screenshot (with creator's permission)
- A risk we're watching (this playbook §4 has 10 to choose from)
- A wins/losses for the week

### What NOT to do
- Don't build a "launch" thread before Phase 1 validation works. Visible launch flop is worse than no launch.
- Don't write generic "AI-app builders are the future" think pieces. They're noise.
- Don't follow back random AI bros for engagement.

---

## 7. Decision discipline

### When to ask the PM (this session)
- Before adding a new feature to scope (literally always — see Risk #10)
- When a technical decision has product implications (e.g., "should we use WebContainers or Vercel preview?")
- When you don't know how to break a problem into tasks
- Pricing questions
- Brand questions

### When to act without asking
- Code-level decisions inside an agreed task (delegate to local Opus)
- Wording polish on docs you've already approved
- Personal preferences (e.g., your daily tools)

### When to defer ("not now")
- Anything that doesn't pass: *"will this directly help a non-programmer ship their app this week?"*
- Hires (you're solo until M5+ revenue)
- Funding (pre-revenue solo founders should not raise unless they want to; Vibell can be bootstrapped to profit at ~50 paying users)
- Localization beyond CZ + EN

### Default: write it down before deciding
For any decision worth >2 hours of work, write a 3-line note in `decisions-log.md`: **what · why · alternatives considered**. Saves your future self and the agents.

---

## 8. Burnout prevention (the most important section)

Solo founders fail more often from giving up than from technical reasons. Energy management beats time management.

### Hard rules
- **One full day off / week.** No Vibell. No Twitter. No "just one task". Brain rest = better work for the other 6 days.
- **Max 5 hours of deep work / day.** After that you're shipping bugs.
- **No work after 22:00.** Sleep is a Vibell investment.
- **One walk / day, alone, no phone.** That's where your best ideas come from, not in front of the screen.

### Warning signs (act when you notice these)
- Skipping Monday PM ritual 2 weeks in a row → take 3 days off, then re-plan
- Adding features to scope when you're tired → that's the worst-quality thinking. Stop, sleep, decide tomorrow.
- "I'll just push to main without testing" — never. Don't break the build because you're rushing.
- Comparing yourself to base44 / Lovable's revenue daily — toxic. Once a month, max.

### Find one accountability partner
A friend (technical or not) who you can text on Friday with "this week I shipped X". External commitment beats internal motivation when energy dips.

---

## 9. First 100 days plan

Assuming you start the M1 task queue this week.

| Week | Focus | Outcome |
|---|---|---|
| 1–2 | M1 foundation tasks (T001–T006) on local Opus | Repo, auth, schema, brand, marketing landing live at vibell.app |
| 3 | **First user research** — talk to 5 non-programmers about their idea, what they'd want | 5 interviews documented |
| 4–5 | M2 wizard + Builder (T007–T010) | Working wizard → first generated landing page |
| 6 | **Watch 5 users complete wizard.** Fix top 3 friction points. | M2 checkpoint passed |
| 7–9 | M3 click-to-edit (T011–T016) | Live preview + click-to-edit demo |
| 10 | **M3 checkpoint** — 5 users edit successfully | Iteration round |
| 11–13 | M4 deploy + SMB enablers (Brand Designer, Autofill, Debugger) | Apps publishable to vibell.app subdomains |
| 14–15 | M5 billing + 50 beta launch | First paying customers |
| **15+** | Iterate based on paid users; consider Phase 2 only if Phase 1 metrics hit |

**Key insight:** by week 6 you should already be watching users, not just shipping features. The shift from "I'm building" to "I'm watching them use it" is what separates products from projects.

---

## 10. When to think about funding (not yet)

You don't need money. You need **time** and **users**. With Claude Code Opus 4.7 as your engineer and Anthropic API as your runtime, your costs are tiny until you have users.

Re-evaluate funding when ALL of these are true:
- You have ≥ 50 paying users
- MRR ≥ 50 000 CZK
- You can articulate one specific reason raising helps (hire someone specific, ship a specific project, expand to specific market)
- Investors are reaching out to *you*, not the other way around

Until then: **profitable solo > funded but distracted**. Vibell at 100 users × $39/mo = $3 900/mo. That's life-changing income on a solo budget. You don't need a Series A.

---

## 11. What I (PM) commit to

To make this work as a duo:
- Update `asistenta.md` every time I learn something new about you
- Capture every strategic decision in `decisions-log.md` with founder-stated context
- Flag every scope creep risk before writing the task
- Keep tasks self-contained for the local Opus
- Recommend, don't dictate — you decide
- Push back gently when an idea is misaligned with scope, but accept your override
- Update this playbook quarterly

---

## 12. Three things to do **this week**

If you do nothing else, do these:

1. **Spin up a one-page waitlist at vibell.app TODAY.** Just hero + email capture + one paragraph. Use Tally or a tiny page locally. Start collecting emails before anything else ships. Free distribution insurance.

2. **List 10 potential first users from your network.** Names, contact methods, why they fit. Print or write — physical, not digital. Pick 2 to message this week.

3. **Pick build-in-public channel (X or LinkedIn) and post one update.** Anything. "Started Vibell, a guided AI app builder. First task: a wizard that walks non-programmers through the build." Tag @anthropicai. Just start.

These take 2 hours combined. They compound for the next 6 months.
