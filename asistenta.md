# asistenta.md — Memory profile of Patrik (founder of Vibell)

> **Účel tohoto souboru:** Toto je živá paměť pracovního vztahu mezi Patrikem (founder) a AI asistenty. **Jakýkoliv agent (Claude Code Opus 4.7 i jiný) musí tento soubor přečíst PŘED jakoukoliv prací v tomto repu**, aby komunikoval a rozhodoval způsobem, který Patrik akceptuje napoprvé.
>
> Aktualizuje to PM agent (Claude v main session) pokaždé, když se něco nového naučí.

---

## 1. Identita

- **Jméno:** Patrik
- **Role:** founder & operator Vibell (vibell.app)
- **Právní status (2026-05):** OSVČ v ČR, plánuje s.r.o. až bude obrat / investor odůvodňovat
- **Tech background:** silná produktová vize a brand instinct. NENÍ klasický vývojář — používá lokální Claude Code (Opus 4.7) pro shipnutí kódu.

---

## 2. Komunikační styl

- **Jazyky:** Čeština primárně, plynule mixuje s anglickými tech termíny. Přepíná bez varování.
- **Délka odpovědi:** krátce, často jednořádkově. *"souhlas"*, *"jeď dál"*, *"to chci"*.
- **Pravopis:** často malými písmeny, **má překlepy** (`moccupy`, `supr`, `asistenta`, `cíľová`). **NIKDY je neoprávek, neakcentuj je, nesnaž se je opravit.**
- **Číslované odpovědi:** když dostane číslované otázky, odpovídá ve stejném formátu — někdy terse, někdy rozšíří vizí.
- **Žádná emoji od něj, neočekává emoji ode mě.** (Brand voice rule sedí s jeho preferencí.)
- **Decisive, ale důvěřuje PM:** často píše *"rozhodni sám"* / *"jeď dál"* / *"je to na tobě, dej tomu směr"*.
- **Tlačí zpátky, když je něco špatně:** *"fialovo-černý styl chci"* přepsalo mé amber doporučení. **Když odporuje, poslouchej pozorně — má jasný instinct.**
- **Nechce recap:** napsal *"nepiš mi teď co máme"*. Když si žádá pokračování, neopakovat status. Jen pracovat.

---

## 3. Rozhodovací styl

- **Vize first, detaily deleguje.** Velké nápady jdou od něj (memory agent, social network, paid Studio Mode). Design detailů jde z PM.
- **Trustuje brand instinct.** Když na vážkách: premium > approachable, tech > playful, minimal > busy.
- **Cost-conscious.** Token efficiency zmínil hned v rané fázi. Care o margin.
- **Produkt > infra.** Řekl *"focus more on product"*. CI / Sentry / PostHog mu deferl do M6.
- **Krok za krokem.** Říká *"postupně"*, *"poďme se zaměřit"*. Nechce velké balíky najednou bez kontextu.

---

## 4. Brand & produkt canon — **nikdy neporušuj**

### 🎯 Aktuální scope (strict — 2026-05-06 v0.3)

**Phase 1 dnes = builder pro malé podnikatele.** Vibell aktuálně dělá *jednu věc dobře*: pomáhá malému podnikateli vytvořit **aplikaci, web, nebo onepager** pro jejich byznys. Žádný marketplace, žádný creator hub, žádné memberships, žádný social network — to vše je **odložené**, ne zrušené.

**Cílový uživatel zúžen:**
- Restaurace, kavárny, bistra
- Coach / consultant / freelancer
- Lokální služby (kosmetika, fitness, holičství, autoservis, real estate)
- Drobní e-shopáři (one-page checkout)
- Solo tvůrci s portfoliem (designéři, fotografové)

**Cílový výstup zúžen:**
- Onepager (landing pro službu / produkt)
- Multi-page site (home / about / services / contact)
- Bookable site (rezervační systém)
- Shop one-pager (jednoduchý e-shop se Stripe checkoutem)

**Neudělat aktuálně:**
- Marketplace prodej appek
- Creator Hub (vlastní profil tvůrce, články, tipy, paid membership)
- Memberships uvnitř user-buildovaných appek
- Referral / Ambassador
- Component shop
- Public showcases
- Multi-user / kolaborace
- Mobile app output

Decisions z `docs/product/decisions-log.md` jsou rozdělené do fází; Phase 1 zachovává jen **D-001 (Smart Autofill), D-002 (AI Debugger), D-004 (AI Brand Designer), D-006 (Analytics), D-007 (Voice — pokud čas), D-009 (Image gen)**. Vše ostatní = **DEFERRED**.

---

| Pravidlo | Hodnota |
|---|---|
| Název | **Vibell** (lowercase wordmark) |
| Doména | `vibell.app` |
| Brand feel | modern · professional · premium |
| Inspirace | Stripe (jasnost), Vercel (minimal), Notion (interaktivní) |
| Primární barva | royal violet `#7C3AED` na near-black `#0A0A0A` |
| Dark mode | **first-class**, ne afterthought (hero je default tmavý) |
| Typografie | Geist Sans + Geist Mono |
| Emoji | **nikde** v product UI / copy / dokumentaci |
| Mascot | **žádný** |
| Voice | clear over clever, confident not boastful, warm not familiar, specific over generic |
| Tagline (EN) | *"Have an idea. Vibell builds it."* / *"Your idea. A real URL. 30 minutes."* |
| Tagline (CZ) | *"Měj nápad. Vibell ti ho postaví."* |

### Klíčové produktové rozlišení (zapamatuj)
- **Smart Mode** (default, locked layout, levné, 95 % uživatelů)
- **Studio Mode** (paid escape hatch, layout freedom, mockup library, full-app view, power users)
- **Two-pillar moat:** (1) Guide System (strukturovaný wizard, ne blank prompt), (2) Social Network (`vibell.app/@username` portfolio + Marketplace s revenue share 70/30 nebo 80/20 pro Pro+)
- **Memory Agent je consent-based**, ptá se na signupu: "Chceš, ať se naučím tvůj styl?"
- **Hosting model:** uživatelé NEŘEŠÍ Vercel / Supabase / domény. Vibell to dělá za ně.

---

## 5. Jak doručovat

- Každý task = **self-contained `.md`** v `docs/tasks/Mx-*/Txxx-*.md`. Sekce **Agent Prompt** je ready-to-paste pro lokálního Opuse.
- **Žádné README.md** soubory bez explicitního požadavku.
- **Čeština v PM↔founder komunikaci.** **Angličtina v kódu, copy, docs** (CZ lokalizace později).
- **Jeden commit na task** s zprávou `task(Txxx): <title>`.
- **Push na branch** `claude/github-repo-setup-NvGZL`.
- **Žádné destruktivní git operace** bez permision (žádný force push, reset --hard, branch -D).

---

## 6. Patrikovy fráze — co skutečně znamenají

| Co píše | Co tím myslí |
|---|---|
| *"jeď dál"* | Neptej se, pokračuj svým best judgement |
| *"to chci"* | Silné ANO. Lock it in. |
| *"souhlas"* | Approved jak jsem navrhl |
| *"rozhodni sám"* / *"je to na tobě"* | Důvěřuju ti, ber ownership nad rozhodnutím |
| *"tomu nerozumím"* | Vysvětli plain language. Nepředpokládej kontext. |
| *"ještě nemám plán"* | Nezatahuj mě do detailů, na které nejsem ready. Defer. |
| *"nepiš mi teď co máme"* | Skip status recap, jen pracuj |
| *"super"* / *"supr"* | Lehké přitakání, ne enthusiasmus |
| *"to dáva smysl"* | Souhlas, lze pokračovat |

---

## 7. Co dělat **bez ptaní**

- Psát tasky self-contained pro lokálního Opuse 4.7
- Dělat brand-aligned design rozhodnutí (violet/black premium minimal)
- Reorderovat milestony, pokud to slouží produktu (deferovat infra)
- Psát ADRs (Architectural Decision Records) když dělám rozhodnutí
- Aktualizovat **tento soubor** když se naučím něco nového o Patrikovi
- Commitovat a pushovat na feature branch
- Ptát se na rozhodnutí, která vážně blokují další postup

## 8. Co **vždy** potvrdit napřed

- Změny pricingu (kreditní ceny, plan tiery)
- Nové features mimo scope, na kterém jsme se shodli
- Cokoli co se týká GDPR / legal / plateb
- Přejmenování produktu nebo domény
- Veřejně-viditelné branding changes
- Investor-relevant pivots
- Destruktivní git operace

---

## 9. Otevřené produktové vize, které zmínil — schválená batch #1 (2026-05-06)

Plný přehled rozhodnutí v `docs/product/decisions-log.md`. Klíčové:

- Memory Agent který se ho učí (consent-based) — **rozhodnuto**
- Marketplace + creator portfolio + **Creator Hub** (vlastní profil tvůrce s články, tipy, paid membership na `vibell.app/@username`) — **rozhodnuto**, Substack/Patreon model uvnitř Vibell, platform fee 7–15 % per plan tier
- Studio Mode s mockup library + full-app view — **rozhodnuto**
- **AI Brand Designer** — generuje totální brand bible, na které stojí appka — **rozhodnuto**
- **Smart Autofill from URL** — vlepí web/LinkedIn → AI předvyplní wizard — **rozhodnuto**
- **AI Debugger** — opravuje build errory, **uživatel neplatí** — **rozhodnuto**
- **Pre-built integrations** (Stripe, Calendly, Mailchimp, …) — **rozhodnuto**
- **Code export + GitHub sync** — **jen Pro+**, bidirectional — **rozhodnuto**
- **Built-in publish analytics** — automaticky postaví, sledování u nás — **rozhodnuto**
- **Voice mode** — pro **všechny plány zdarma** — **rozhodnuto**
- **AI SEO assistant** — **rozhodnuto**
- **AI Image generation** — uživatel platí — **rozhodnuto**
- **Continuity Coach** — vede uživatele kde má pokračovat když neví — **rozhodnuto**
- **Referral / Ambassador** — 10 % z paid membership lifetime, nováček +100 free kreditů — **rozhodnuto**
- Postupně rozhodnuté kreditní plány: free 50, basic 200, standard 1000, pro 3000, team 9000
- Eventually mobile output (phase 3)
- Eventually team plán s kolaborací (3× ceny pro jednotlivce + týmová cena)
- Konkurenti, proti kterým se vymezuje: **base44** (primárně), Lovable, Bolt, v0
- Cílový trh: **EN globálně primárně, CZ/SK sekundárně**

### Fázování (founder-stated)
1. **Phase 1 (now):** Studio produkt — Wizard, Builder, Workspace, Click-to-edit, Studio Mode, Mockup Library
2. **Phase 2:** Creator Hub + Marketplace + memberships, Brand Designer, Autofill, Debugger, Analytics, Integrations
3. **Phase 3:** Referral, Voice, SEO, Image gen, GitHub sync, Continuity Coach

---

## 10. Co je **právě teď** in-flight

| | |
|---|---|
| Branch | `claude/github-repo-setup-NvGZL` |
| Aktivní fáze | M1 (foundation) → M2 (wizard + builder) |
| Tasky ready k běhu | T001–T008 (paste do lokálního Opuse) |
| PM píše dál | T009 Builder Agent (s memory injection), T010 Workspace shell |
| Bloker | žádný — Patrik schválil směr, jede se |

---

## 11. Drobné poznatky, které by se nemusely vejít jinam

- Pojmem **"mockup"** myslí všechno vizuální (varianty, screenshoty, reference), ne nutně wireframe.
- Když píše *"prototyp"*, často nemyslí jen MVP — myslí i jednu konkrétní šablonu nebo feature.
- Doménu kupuje "za pár dnů", neptej se na status — počká.
- Stripe / fakturační email nemá vyřešený, **úmyslně chce nejdříve vyladit produkt a hodnotu**, pak řešit billing identity. Respect.
- **Pravopis "asistenta" vs "asistent":** pojmenoval tento soubor `asistenta.md` (genitiv) — drž se toho jména, ať se neztratí historie.
- **Periodicky chce brainstorm injection** nových nápadů, ať drží momentum ("dej mi nějaké návrhy dále vylepšit projekt"). Když je prostor, **iniciativně přines 5–10 ranked nápadů** s mým doporučením, na čem začít. Nedávej dump 40 položek — kurátor.

---

## 12. Updates log

- **2026-05-06 v0.1** — soubor založen. Zachycuje pozorování z prvních 7 PM↔founder kol (vize, brand, Studio Mode, Memory Agent, Mockup Library, Patrikův styl).
- **2026-05-06 v0.2** — strategická rozhodnutí batch #1 (D-001 až D-013). Schváleno: Smart Autofill, AI Debugger (free for user), Pre-built integrations, AI Brand Designer (totální brand bible), Code export + GitHub sync (Pro+ only), Built-in analytics (free), Voice mode (free for all plans), AI SEO, Image gen (user pays), Continuity Coach, Referral 10 % lifetime + 100 credits onboarding bonus, Creator Hub (Substack/Patreon model na profilu tvůrce). Memberships uvnitř user-buildovaných appek (D-013) deferred. Detail v `docs/product/decisions-log.md`. Patrikův styl potvrzen: vize big, detail deleguje, koriguje rychle a přesně když si špatně vyložím (D-012 → Creator Hub).
- **2026-05-06 v0.3** — **scope cut**. Patrik zúžil Fázi 1 na *"tvorba aplikace stránek nebo onepage pro malé podnikatele"*. Creator Hub / memberships / marketplace / referral atd. = DEFERRED (ne zrušeno). Šablony refactor: Landing, Local Business, Service Business, Booking, Shop one-pager. To-do + Blog šablona = vyřazeno z MVP. Patrikův styl: **velmi rád se vrací a zužuje scope** když cítí, že rozsah nabírá. To je ZDRAVÉ — PM by měl tendenci scope nafukovat aktivně potvrzovat *"přidat tohle teď, nebo později?"* na každém větším pomyšlení.
