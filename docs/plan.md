# Vibell — Produktový plán

> Stav: Brainstorming hotov, plán schválen → další krok = MVP scaffold
> Datum: 2026-05-06

---

## 1. Vize

**Vibell** je vizuální průvodce, který provede neprogramátora od myšlenky k publikované webové aplikaci za 30 minut. Místo prázdného promptu nabízí strukturované kroky, šablony, mockupy a klikací editaci. AI agenti dělají těžkou práci v pozadí, uživatel vidí pouze výsledek.

**One-liner:** *"Měj nápad. Vibell ti ho postaví."*

---

## 2. Cílový uživatel

- **Primárně:** Nepogramátor s nápadem (drobní podnikatelé, kreativci, studenti, freelanceři, hobby tvůrci).
- **Sekundárně:** Junioři / "vibe coders" co chtějí rychlý start.
- **Trh:** EN globálně (primárně), CZ/SK (sekundárně) — produkt EN-first, lokalizace CZ od začátku.

---

## 3. Konkurence a pozicování

| Produkt | Silné stránky | Slabiny (naše příležitost) |
|---|---|---|
| **base44** | Jednoduché UI, hotový hosting | Méně struktury při onboardingu, omezená klikací editace |
| **Lovable** | Široký záběr, populární | "Prompt-and-pray" UX, velké tokeny, frustrace u neprogramátorů |
| **Bolt.new** | Rychlost, IDE feel | Pro programátory, ne pro laiky |
| **v0 (Vercel)** | Kvalita UI komponent | Jen UI, ne celá appka |
| **Replit Agent** | Plná aplikace včetně backendu | Komplexní, "vývojářský" UX |

**Pozicování Vibell:**
> *"Strukturovaný průvodce, ne prázdné okno. Klikáš, neprompuješ. Hostujeme my, ty se učíš tvořit."*

Diferenciátory:
1. **Wizard místo promptu** — strukturovaný vstup šetří tokeny i nervy.
2. **Šablony + mockup před kódem** — uživatel vidí výsledek dřív než cokoli generujeme.
3. **Klikací editace s ikonkami** — žádný kód, žádný terminál.
4. **3 varianty vedle sebe** — ne odhadovat, vybrat.
5. **Hosting u nás zdarma** — 0 friction při publikování.
6. **Marketplace + portfolio** — komunita prodává a kupuje appky, Vibell má provizi.

---

## 4. Hlavní funkce (schváleno)

### Vstupní fáze
1. **Galerie šablon** (20–30 typů) jako default start.
2. **Hlasový vstup** ("popiš nápad mluvou" → strukturovaný formulář).
3. **AI vyplní wizard za uživatele** z 1 věty.
4. **Žádný krok není povinný** — AI doplní default.
5. **Klikací příklady vedle inputů** ("Klikni pro inspiraci").

### Editace
6. **Klikni → změň** — vizuální klikací editor.
7. **3 varianty vedle sebe** — uživatel vybírá, ne diktuje.
8. **Undo všeho jednou klávesou** (⌘Z všude).
9. **Mobile + desktop preview vedle sebe** přes ikonku.
10. **Chytré chyby** — lidský popis + 2–3 řešení k odkliknutí.

### Komunita & růst
11. **Marketplace + digitální portfolio uživatele**:
    - Každý uživatel má profil ("portfolio") s jeho appkama.
    - Může nabídnout aplikaci k prodeji jiným uživatelům.
    - Revenue split: **70 % tvůrce / 30 % Vibell** (Apple-like).
    - Možnost zdarma šablona (sdílení) → tvůrce dostane "shoutout" + analytics použití.
12. **Vibell Coach** — animovaný průvodce prvním projektem (5 min).
13. **Transparentní cena akce** — "Tato úprava: 2 kredity (~3 Kč)".
14. **Auto-save + verzování** — každá změna = snapshot, vrátit kamkoli.

### Pro budoucnost
15. **Mobile-first samotná Vibell** (tvořit appky z telefonu).

---

## 5. Architektura agentů

Místo jednoho velkého modelu sada specializovaných agentů. Každý dostane jen relevantní kontext → menší prompty, levnější běh, lepší výstupy.

```
                  ┌─────────────────────┐
                  │   Orchestrator      │
                  │  (rozhoduje o toku) │
                  └──────────┬──────────┘
                             │
       ┌──────────┬──────────┼──────────┬──────────┐
       ▼          ▼          ▼          ▼          ▼
   ┌───────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐
   │Builder│ │  UI    │ │  Data  │ │ Deploy │ │  Coach  │
   │ Agent │ │ Editor │ │ Schema │ │ Agent  │ │  Agent  │
   └───────┘ └────────┘ └────────┘ └────────┘ └─────────┘
```

| Agent | Odpovědnost | Model |
|---|---|---|
| **Orchestrator** | Routing intentu, plánování kroků | Sonnet 4.6 |
| **Builder** | Postaví scaffold projektu z wizardu (Next.js + obsah) | Sonnet 4.6 |
| **UI Editor** | Mikro-úpravy komponent (text, barva, layout) | Haiku 4.5 |
| **Data Schema** | Návrh DB tabulek + Supabase migrace | Sonnet 4.6 |
| **Deploy** | Push na náš Vercel/Supabase, doménění | Haiku 4.5 |
| **Coach** | Konverzační průvodce, vysvětlování | Haiku 4.5 |
| **Variant** | 3 paralelní varianty pro stejný úkol | Haiku 4.5 ×3 |

**Token-saving techniky:**
- **Prompt caching** (Claude API) na všechny system prompty a šablony.
- **Komponenta-level scope** — agent edituje jen 1 soubor, ne celý projekt.
- **Streamované partial editace** — pošle pouze diff, ne celý soubor.
- **Cache šablon** — generický wizard output pro běžné typy je memoizovaný.

---

## 6. Tech stack

### Frontend (Vibell sám)
- **Next.js 15** (App Router) + **React 19**
- **Tailwind CSS 4** + **shadcn/ui** (vlastní design system)
- **Framer Motion** pro animace průvodce
- **Zustand** pro stav editoru
- **TanStack Query** pro data
- **WebContainers (StackBlitz)** pro live preview generovaných appek

### Backend
- **Vercel Functions** (serverless API)
- **Supabase** — auth, databáze (Postgres), storage
- **Anthropic SDK** — Claude API (Sonnet 4.6, Haiku 4.5, prompt caching)
- **Stripe** — billing, kreditní systém
- **Vercel API** — programatické deploye uživatelských appek
- **Resend** — transakční e-maily

### Generované aplikace (output pro uživatele)
- **Next.js + Tailwind + shadcn/ui** (jednotný stack = jednodušší údržba šablon)
- **Supabase** jako backend (sdílená infra Vibell, izolace přes RLS / project-per-app)
- Každá appka dostane subdoménu `[name].vibell.app` (free/basic) nebo vlastní doménu (pro/team)

### DevOps
- **GitHub** repo `patrikbsvk/patrikbsvk` (monorepo)
- **Vercel** pro Vibell hosting
- **GitHub Actions** pro CI (typecheck, test, lint)
- **Sentry** pro error tracking

---

## 7. Kreditní model a cenotvorba

### Kolik stojí 1 kredit (náš náklad)
- 1 kredit ≈ **0,015 USD** (po prompt cachingu) → marže 80 %+

### Mapování akcí na kredity
| Akce | Kredity | Cca náš náklad |
|---|---|---|
| Vytvoření nového projektu (z šablony) | 5 | $0.07 |
| AI úprava textu / barvy (malá) | 1 | $0.015 |
| Klikací úprava komponenty | 2 | $0.03 |
| 3 varianty vedle sebe | 3 | $0.045 |
| Generování celé sekce | 5 | $0.075 |
| Změna DB schématu | 3 | $0.045 |
| Hlasový vstup → wizard | 2 | $0.03 |
| Auto-save snapshot | 0 | (zdarma) |
| Deploy / publish | 0 | (zdarma) |

### Plány

| Plán | Cena (CZK) | Cena (USD) | Kredity / měs | Náš náklad | Marže |
|---|---|---|---|---|---|
| **Free** | 0 | $0 | 50 | $0.75 | -$0.75 (akvizice) |
| **Basic** | 349 | $15 | 200 | $3 | $12 / 80 % |
| **Standard** | 899 | $39 | 1 000 | $15 | $24 / 61 % |
| **Pro** | 2 290 | $99 | 3 000 | $45 | $54 / 54 % |
| **Team** | 5 990 | $259 | 9 000 (3× Pro) + collab | $135 | $124 / 48 % |

### Pravidla
- Kredity se obnovují měsíčně, **nepřevádějí se**.
- **Overage:** dokoupit balíček 100 kreditů za 199 Kč ($8.50) kdykoli.
- **Free uživatelé:** appka má watermark "Made with Vibell", běží na shared subdoméně, sleep po 24 h nečinnosti.
- **Hard limity** na heavy users (max 200 úprav/den i u Pro) → ochrana proti runaway nákladům.

### Marketplace ekonomika
- Tvůrce nastaví cenu (např. $9, $29, $99 jednorázově).
- **Split:** 70 % tvůrce / 30 % Vibell.
- Stripe Connect pro výplaty.
- Free šablony bez peněz → tvůrce dostane analytics + visibility.
- **Bonus pro plán Pro+:** 80 % tvůrci / 20 % Vibell (motivace upgradu).

---

## 8. UX flow (high-level)

```
1. Landing                  →  "Začni s nápadem"
2. Welcome Coach            →  5min interaktivní tour
3. Šablona vs. blank        →  20+ šablon nebo "od nuly"
4. Wizard (4 kroky)         →  název / funkce / mockup / styl
5. Builder Agent generuje   →  "Tvořím… (15s)"
6. Workspace                →  preview + edit panel + chat
7. Klikací editace          →  hover → click → "Co změnit?"
8. Auto-save → verze        →  každá změna = snapshot
9. Publish                  →  1 klik → naše subdoména
10. Marketplace publish     →  volitelně, nastaví cenu/free
```

---

## 9. MVP — milníky (8 týdnů)

### Týden 1–2: Základní kostra
- Next.js projekt, Tailwind, shadcn/ui setup
- Supabase auth + basic schéma (users, projects, snapshots)
- Landing page + signup
- Základní layout workspace

### Týden 3–4: Wizard + Builder Agent
- Wizard UI (4 kroky)
- 5 startovních šablon (To-do, Blog, Portfolio, Booking, Landing)
- Builder Agent (Claude API integrace + prompt caching)
- Generování projektu → uložení do DB

### Týden 5: Live preview + UI Editor Agent
- WebContainers integrace
- Klikací overlay (data-vibe-id na komponenty)
- UI Editor Agent (mikro-úpravy)
- Auto-save + verzování

### Týden 6: Deploy
- Deploy Agent → programatické publishování na náš Vercel
- Subdomény `[slug].vibell.app`
- Watermark pro free uživatele

### Týden 7: Kreditní systém + Stripe
- Mapování akcí → kredity
- Stripe Checkout pro Basic plán
- Kreditní counter v UI + transparentní ceny

### Týden 8: Polish + soft launch
- Vibell Coach (basic)
- Chytré chyby
- Mobile preview
- Beta launch pro 50 uživatelů

**Cíl MVP:** 50 beta uživatelů, 5 % konverze (3 platící), validace UX.

---

## 10. Roadmap fáze 2 (měsíc 3–4)

- Hlasový vstup
- AI vyplnění wizardu
- 3 varianty vedle sebe
- Marketplace + portfolio (revenue share)
- Standard a Pro plány
- 20+ šablon
- Public templates pro free uživatele

## 11. Roadmap fáze 3 (měsíc 5–6)

- Team plán + collaborative editing
- Custom domény pro Pro+
- API přístup
- White-label
- Vibell mobile app (tvorba z telefonu)
- Mobile output (React Native / Expo)
- Pokročilé integrace (Stripe, Resend, OAuth providers)

---

## 12. Rizika a otevřené otázky

### Rizika
| Riziko | Mitigace |
|---|---|
| Runaway API náklady | Hard limity per plán, monitoring + alerts |
| Kvalita generovaných appek | Šablony jako "rails", AI mění jen detail |
| Bezpečnost user-generated kódu | Sandbox preview, žádný eval na našem serveru |
| GDPR / data uživatelů appek | Per-app Supabase schema + RLS izolace |
| Spam / abuse v marketplace | Manuální review prvních N appek, později ML moderace |

### Otevřené otázky (k dořešení před MVP)
1. **Doména** — `vibell.app`, `vibell.com`, `vibell.io`? Zkontrolovat dostupnost.
2. **Subdomény generovaných appek** — `[slug].vibell.app` nebo `[slug].vibell.site`?
3. **Stripe entity** — kdo bude účet vlastnit (CZ s.r.o., zahraniční)?
4. **GDPR / DPA** s uživateli appek — kdo je zpracovatel? Vibell je sub-processor?
5. **Free tier abuse** — limit 1 appka, nebo timer?
6. **Branding** — logo, barvy, font (může počkat na MVP).

---

## 13. Definition of Done pro MVP

- [ ] Uživatel se registruje a projde Coach tour < 5 min
- [ ] Vybere šablonu, projde wizard, vidí preview < 2 min
- [ ] Klikne na text/barvu, AI změní < 10 s
- [ ] Publish na subdoménu < 30 s
- [ ] Stripe checkout pro Basic plán funguje
- [ ] Kreditní counter funguje a odráží reálné náklady
- [ ] 50 beta uživatelů, NPS ≥ 30, ≥ 3 platící
