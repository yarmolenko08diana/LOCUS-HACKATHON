# Qadam — personal admissions route.

A project for **LOCUS Startup Hackathon 2026, case 02 "Personal Admissions Route"**.
Submission code: `LOCUSCASE2`.

**Qadam** (қадам — "step") turns a short admissions questionnaire into a route: where to apply, why that option fits, and what to do next.

The interface is available in **Kazakh and Russian**, with light and dark themes. It covers 336 programs in 23 countries, scholarship matching, a deadline calendar, feedback on motivation essays and study plans, and recommendations for extracurricular activities.

- **Demo:** https://locus-hackathon-psi.vercel.app/
- **Code:** https://github.com/yarmolenko08diana/LOCUS-HACKATHON
- **Languages:** Kazakh and Russian. **Themes:** light, dark, and system default.

---

## 1. The problem

Applying to university involves dozens of decisions: field of study, country, university, entrance exams, documents, deadlines, and activities. The information is scattered, the advice is too generic, and applicants can't answer two questions: **what fits them specifically** and **what to do right now**.

Existing services offer either a catalog of universities or a chat that replies in general terms. Neither takes a person all the way to a submitted application.

## 2. The solution

A single connected five-step flow rather than a set of separate screens. Users can go through it from scratch or open **one of three ready-made examples** — with high, medium, and low chances — to see how the profile affects the results.

| Step | What the user gets |
| --- | --- |
| 1. Profile | An 8-screen questionnaire: education stage, interests, school system and scores, languages, exams, achievements, countries, budget, and priorities. Each achievement records not only its scale and result but also its **type**: a research paper, a conference talk, a patent, or a lab internship each carry different weights. Hackathons are split by format (classic 24–48 hours, online, corporate, themed, CTF), and competitions by type (competitive programming, research papers, case championships, pitch sessions). The scale labels match the type of competition: not "school-level," but "local: within a school or university" |
| 2. Diagnosis | A profile summary: educational goal, what works in the user's favor, what limits their choices, and how complete the questionnaire is |
| 3. Recommendations | Programs with a match percentage and an explanation of "why it fits," written in plain language |
| 4. Comparison | Up to three options side by side across 13 parameters, marking where each one wins, with a short conclusion |
| 5. Plan | A personal roadmap in phases "Now → Preparation → Application → Final," with one highlighted next step and progress tracking |

There is also a detailed breakdown for each program: the match percentage split across seven criteria, a chance estimate **itemized by factor** ("ENT: 112 against a benchmark of 90"), a list of gaps, costs, requirements, and application periods with a link to the official site.

Four sections strengthen the route and are available from any screen:

| Section | What it does |
| --- | --- |
| 💰 Scholarships | Matches 15 scholarships and grants to the profile, with explanations and a list of unmet requirements |
| 🗓 Calendar | Application periods for programs, scholarships, and exams, by month; a star marks reminders |
| 🚀 Activities | What to add to an application, in three groups: application documents, academic work, and experience and people. The list is built from gaps in the profile, not from a generic catalog |
| ✍️ Application texts | A review of a motivation essay and study plan against admissions committee requirements: what they look for, what is already covered, and what to do about each missing item |
| 👤 Account | Name, surname, birth year, and photo. Sign-in with email and password, and password change. Works without a server: the password is never stored, only a salt and the SHA-256 of the salt combined with the password |
| ★ Favorites | Saved programs in one place |

### Audience

A 10th–11th grade student or graduate from Kazakhstan applying to a bachelor's program — domestically or abroad, including **NIS and IB** students. The demo database is built for this audience: 336 programs in 23 countries, roughly 15 per country:

- **Kazakhstan** (24 programs) and **Russia** (15), **Georgia** (12);
- **Turkey** (15) and **UAE** (12);
- **Europe:** Czechia, Poland, Hungary, Germany, Austria, the Netherlands, Italy, Spain, France, the United Kingdom (147);
- **Asia:** China, Hong Kong, Japan, Korea, Singapore, Malaysia (80);
- **Americas:** the USA (15) and Canada (15).

Each country has a briefing on why it's worth considering for an applicant from Kazakhstan — free tuition in Germany, Diritto allo Studio in Italy, GKS in Korea, Stipendium Hungaricum in Hungary, PGWP in Canada — with a link to the official portal. It appears as a line on the program card and as a block on the program's page.

## 3. What makes the product personal

Nothing is hard-coded. **Everything the user sees is derived from the profile.** Change the budget, country, interest, or exam, and the recommendations, explanations, plan, and next step are all recalculated. The product makes this explicit: after an edit, a notification appears, such as "Matches updated: 2 new options in the top 3," with a specific list of what changed.

A quick-edit block on the recommendations screen — "Change conditions and see the difference" — lets users adjust settings without going back to the questionnaire.

**Achievements change the results mechanically, not cosmetically.** Each program has a `holistic` parameter (1–5), which measures how thoroughly the university reads an application as a whole rather than relying only on scores. Adding an olympiad, hackathon, research project, or volunteer experience:

- raises programs with holistic admissions in the ranking and lowers those where only exams count (this does not happen for programs with `holistic = 1`);
- changes the contribution of the "Achievements" criterion to the match percentage;
- adds an item to the chance breakdown;
- rebuilds the plan: the "add achievements" step disappears, and the next most important gap is suggested instead.

This is covered by tests (`src/engine/__tests__/personalization.test.ts`).

**Communication style** is chosen in the questionnaire: friendly, mentor, coach, or business. It changes only the wording (`src/engine/tone.ts`) — never the program matching or the chance estimate.

## 4. How matching works

The core is a **transparent, explainable engine**, not a black box. The overall match percentage is the sum of six criteria with fixed weights:

| Criterion | Weight | What it evaluates |
| --- | --- | --- |
| Field of study | 26 | How well the program matches the interests; the main field counts for more |
| Admissibility | 20 | The gap between the profile and the requirements: ENT, IELTS/TOEFL, SAT, GPA or IB diploma, portfolio, internal entrance tests |
| Budget | 17 | Tuition cost against the budget; presence of a grant and room within the budget |
| Geography | 13 | Whether the country is on the user's list, and how close it is if relocation is allowed |
| Language | 9 | Language of instruction against the user's languages and English level |
| Achievements | 9 | The strength and relevance of the portfolio, multiplied by how thoroughly the program reviews the application as a whole |
| Priorities | 6 | Cost, reputation, employment, proximity to home, community |

Several details noticeably affect the quality of results:

1. **Field of study works as a gate, not as one more criterion.** A cheap program near home doesn't become a match if it isn't about what the person wants to study. All other criteria are multiplied by a relevance coefficient, `0.25 + 0.75 × field_match`, so irrelevant programs can't rise to the top on budget and geography alone.
2. **Willingness to relocate is a hard constraint, not a preference.** If the person can't move, a program in another country doesn't exist for them, however high its score on other criteria. It is removed by a filter, not by a lowering coefficient.
3. **Barely passing is different from passing with room to spare.** If a score just reaches the threshold, the criterion gives 0.85 rather than 1.0. This separates options where the user passes comfortably from borderline ones.
4. **NIS and IB scores are converted to a common scale.** Admission requirements in the demo database are written on the familiar Kazakhstani scale, so the NIS final score (0–100) and the IB diploma (24–45) are converted to a five-point scale (`src/engine/academics.ts`) rather than compared directly. Where a university accepts IB directly, the comparison is made on the diploma score. A passed TOEFL counts toward a university's IELTS threshold where one is specified.

Explanations are generated from the same breakdown: each sentence on a card is tied to a specific criterion that applied and includes real numbers from the profile ("Your ENT score of 112 is above the benchmark of 95").

**The chance estimate** is calculated separately, using only the admissibility criterion. It is always labeled as approximate and lists what must be addressed before applying. On the program card it expands **itemized by factor**: each factor (ENT, English, academic performance, portfolio, achievements) shows its own verdict and the actual numbers from the profile. The wording deliberately contains no probabilities and no promises of admission.

## 5. Data honesty

The case prohibits showing fabricated precision and unconfirmed deadlines, so:

- the database of **336 programs** is marked as demonstration data in the code and the interface (`src/data/programs/`, constant `DATA_DISCLAIMER`);
- **every program and every scholarship has a link to its official page** — it appears on the card, in the comparison, in the plan, in the calendar, and in the breakdown;
- passing scores are **benchmarks**, not official thresholds. They are rounded and given for the prototype, so each figure comes with a link to its primary source. A test enforces this: every program must have an https source;
- deadlines are given as **periods** ("December – February"), not specific dates, marked "approximate," with a "verify" button linking to the university's site;
- the first task in the roadmap is "Check the requirements against the official website";
- if exam scores are missing, the chance estimate becomes more cautious rather than filling in invented values.

## 6. Stack and architecture

- **React 19 + TypeScript + Vite** — a static build, deployable without a server.
- **Bilingual interface (Kazakh / Russian)** without an i18n library. Engine texts are paired strings `L('in Russian', 'in Kazakh')` (`src/i18n/lang.ts`); component texts use the same `L` through a React context (`src/i18n/LangContext.tsx`). The language is read at render time, not at module load, so switching rebuilds the recommendations, the plan, and the explanations. Dictionaries return labels through a `Proxy`, so call sites like `FIELD_LABEL[id]` did not need to change.
- **Dark theme on CSS variables:** Tailwind colors are declared as `rgb(var(--c-*) / <alpha-value>)`, and the `.dark` class overrides the variables. No component contains a single `dark:` variant, so the theme cannot drift out of sync with the light one.
- **Tailwind CSS 3** — a custom design system in `tailwind.config.js`: palette, typography, radii, shadows, animations. No ready-made UI kits were used; all components are written in the project (`src/components/ui.tsx`).
- **react-router-dom (HashRouter)** — real URLs for every screen that work on GitHub Pages without server rewrite rules.
- **localStorage** — saves the profile, progress, favorites, and comparison. All access is wrapped in try/catch: in a private browsing window the product keeps working, just without saving.

```
src/
  types.ts              domain model
  data/
    taxonomy.ts         reference data: fields, countries, languages, budgets, exams, achievements
    programs/           demo database of 105 programs with sources, by region
    scholarships.ts     15 scholarships and grants with sources
    activities.ts       24 activity ideas: essays, CV, volunteering, hackathons, research
  i18n/
    lang.ts             engine language and the L(ru, kk) function
    LangContext.tsx     interface language for components
  engine/
    match.ts            weights, scoring, explanation generation, chance estimate
    academics.ts        NIS, IB, and TOEFL on a common scale
    achievements.ts     achievement weight: level, award, recency, relevance
    diagnosis.ts        profile summary: strengths, constraints, goal
    roadmap.ts          building the personal plan and choosing the next step
    scholarships.ts     scholarship matching and the list of unmet requirements
    activities.ts       activity matching from profile gaps
    calendar.ts         calendar of application and exam periods
    tone.ts             communication style: changes only the wording
    ai.ts               optional LLM layer (disabled by default)
  store/
    app.tsx             application state, recalculation, change detection
    storage.ts          safe localStorage access
  components/           design system, shell, navigation, cards
  screens/              route screens
```

The separation is intentional: `engine/` contains pure functions without React, so they can be tested and reused; `screens/` only display the result. No screen contains matching logic.

## 7. AI and external services

**What is used as "AI":** an explainable weighted-scoring engine with natural-language explanations. The case explicitly rules out limiting the solution to a standard AI chat, and for a product where the user makes decisions about their future, deterministic and verifiable logic is a deliberate choice: every number can be opened and explained, the result is repeatable, and it does not depend on an external API being available.

**The optional LLM layer** (`src/engine/ai.ts`) rewrites the diagnosis text in more natural language. It:

- is **disabled** by default (`VITE_LLM_ENABLED=false`);
- makes no decisions and adds no facts — it receives the finished diagnosis;
- silently falls back to the engine's text on unavailability, a timeout (6 s), or any error, so the flow never breaks;
- calls a **server-side proxy** via `VITE_LLM_ENDPOINT`: the key is never stored on the client and never enters the repository.

The deployed demo has this layer turned off: a public static site is not the place for keys.

**External APIs and data:** none. The demo database is static, and the product works offline after the first load. The only external resource is the Inter font from Google Fonts, with a system font fallback in the font stack.

**Ready-made components and libraries:** React, React DOM, react-router-dom, Vite, Tailwind CSS, autoprefixer, PostCSS, TypeScript, oxlint. There are no UI kits (MUI, shadcn, Ant, or similar) — all components are custom.

## 8. Running the project

```bash
npm install
npm run dev        # http://localhost:5173
```

Other commands:

```bash
npm run build      # type check + production build in dist/
npm run preview    # local preview of the production build
npm run test       # 66 engine tests (vitest)
npm run lint       # oxlint
```

The tests cover what matters most not to break: the range and ordering of scores, filtering of irrelevant programs, how results respond to a change in budget and field, the requirement that every recommendation has an explanation, the absence of admission promises in the chance text, an https source for every program and scholarship, rebuilding the roadmap when the profile changes, conversion of NIS, IB, and TOEFL scores, the order and absence of duplicates in the calendar, and the fact that **an added achievement really does change the top 3 and the plan**, but does not affect programs where only exams matter. A separate test checks that switching to Kazakh changes the texts but not the matching or the percentages. Newer tests cover the review of essays and study plans (readiness grows only together with closed requirements, a partial answer counts as exactly half, and a document's importance depends on the selection), the weight of achievement types (a scientific paper outweighs a school project of the same scale), the difference in results between the three ready-made examples, and the presence of a briefing with an https source for each country in the database. Another group covers scholarship matching (a creative-portfolio scholarship does not fall into an IT profile; scholarships with small quotas carry an honest warning) and the CSCA exam: professional Chinese appears only on the Chinese-language track, physics only for engineers, chemistry only for medical applicants, and the step enters the plan when China is selected in the questionnaire.

The optional LLM layer is configured through `.env` (template: `.env.example`). The `.env` file is in `.gitignore`: keys and secrets are not committed to the repository.

### Deployment

GitHub Pages via Actions (`.github/workflows/deploy.yml`). The base path is set automatically from the repository name through the `VITE_BASE` variable. Enable once: **Settings → Pages → Source → GitHub Actions**.

## 9. Test scenario for judges

1. Open the demo and click **"View the ready-made example."** This loads the profile of an 11th-grade NIS student: IT and science, ENT 112, NIS final score 88, English B2, three achievements, a budget up to $6,000, and priorities "low cost" and "employment after graduation."
2. On the **Diagnosis** screen, review the goal, strengths, and limitations.
3. Go to **Recommendations**: each card states what matched, what to take into account, the cost of a year with living expenses, and the approximate chances.
4. Open **"Breakdown"** for any program: the match percentage across six criteria, the gaps to the requirements, and a link to the official site.
5. **Check the reaction to data changes.** On the recommendations screen, expand "Change conditions and see the difference" and switch the budget to **"Grant only"** or the main field to **"Medicine."** A notification appears at the bottom with what changed in the top 3; the list and explanations are rebuilt.
6. Mark two options with the **"Compare"** button and open **Comparison**: a table of 13 parameters, a green dot marks the winner in each row, and a short conclusion sits at the top.
7. Open **Plan**: one next action is highlighted at the top, with an explanation of "why it matters." Click "Mark as done": the next step and the progress update.
8. **Check the reaction to achievements.** Click "Edit questionnaire" → the **"Achievements"** step → add, for example, a republic-level olympiad with gold. After saving, a notification appears, universities with holistic admissions rise in the top 3, an "Achievements" item appears in the chance breakdown, and the "add achievements" step disappears from the plan.
9. Open the sections above the main path: **Scholarships** (matching with gaps), **Calendar** (periods by month; star = reminder), **Activities** (what to add to the application), **Favorites**.
10. **Kazakh language and dark theme:** the header has the `ҚАЗ / РУС` switch and the theme button. Switching the language changes the interface, the explanations, and the plan — but not the composition of the matching or the percentages.
11. Check the full path from scratch: **"Edit questionnaire" → go through all 8 steps.**
12. **Mobile version:** open on a phone or narrow the window to 390 px — the full path works, and a five-step navigation appears at the bottom.

Interface states worth checking: an empty matching result (in the questionnaire, leave one country and "Grant only" — a clear explanation appears instead of a blank screen), opening the comparison screen with nothing selected, and a plan with all steps marked as done.

## 10. Prototype limitations

- The program database is demonstration data: 336 programs, passing scores are rounded and given as benchmarks, deadlines are periods. Production use requires parsing official sources and regular verification.
- The chance estimate does not account for a given year's competition or the quality of a motivation letter. The review of essays and study plans compares the person's own marks against requirements; it does not read the text itself.
- The CSCA subject list is compiled from open requirements and varies by university. The interface marks it as a guideline with a link to campuschina.org.
- **The account is local.** Registration, sign-in, and password change work, but everything is stored in this browser: there is nowhere to send a password reset email, and the account will not exist on another device. The password is never stored — only a salt and the SHA-256 of the salt combined with the password. This protects against reading the storage, but it is not full authorization. Real authorization requires a server, and the interface says so directly.
- The sign-in session lives in memory: after a page reload the password will be requested again. "Remember me" without a server would mean storing a sign-in marker next to the hash, and then the password would protect nothing.
- **Reminders work without a server:** starring an event highlights it in the calendar and is saved in the browser. The prototype has no emails or push notifications, and the interface states this directly.
- **A mobile app is planned.** The header and home screen show an "App — coming soon" banner: more honest than a button that leads nowhere.
- The interface and all engine texts are translated into Kazakh. **The data** — university names, descriptions of programs, scholarships, and activities — remains in Russian. That is the next step, not a delivered feature.
- The LLM layer is disabled in the demo; the project has no server-side proxy.
- The engine is covered by tests; the interface was checked by running the full scenario in a headless browser at 1200 px and 390 px, in light and dark themes, in both languages.

## 11. How the project was built

The project was written with Claude Code: the architecture, the matching engine, the interface, the demo database, and the tests were created in a dialogue with the model, while decisions were made and verified by a team member. This is visible in the commit history — some commits list Claude as a co-author. The case allows the use of LLMs, and we consider it right to state this explicitly rather than hide it.

## 12. Team roles

Team: **luiqi**

| Member | Role | Area of responsibility |
| --- | --- | --- |
| Yarmolenko Diana Dmitrievna | Lead Fullstack & AI Engineer | Backend, logic, and AI |
| Moldagali Zhasmina Meirmankyzy | Frontend & UI/UX Developer | Frontend, interface, and pitch |
