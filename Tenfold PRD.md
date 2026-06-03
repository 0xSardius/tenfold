# PRD — Daily Idea-Generation App (V1)

**Name:** _Tenfold_
**Audience for this doc:** Claude Code (build agent) + the founder.
**Scope of this PRD:** Phases 0–1 from `Tenfold Roadmap.md` — the solo practice app and its public, web-first / mini-app launch. **No token, staking, lottery, or public commons in V1.** Those are Phases 2–4 and are explicitly out of scope here.
**Last updated:** 2026-06-01

---

## 0. How to use this doc (note to the build agent)

You are building a real, deployable web application — not an artifact/sandbox. Use a real database and real persistence. Browser storage limitations from sandbox environments do **not** apply.

**Before writing integration code, fetch and read the reference docs in §9.** The Farcaster and Base docs are explicitly LLM-friendly (single-markdown / `llms.txt` formats) and should be pulled into context. Prefer the official docs over your training data for any SDK/manifest/API detail, because this ecosystem changes fast and shifted again in early 2026 (see §3).

Work in the milestone order in §10. After each milestone, stop and confirm it runs before proceeding.

---

## 1. Product summary

A daily practice app that makes the user measurably better at generating ideas, based on James Altucher's "10 ideas a day" method. The user picks or is given a prompt, writes a list of ideas (starting at 5, ramping to 10), and an AI coach reflects back *how the user's idea muscle is performing* — never whether the ideas are "good." Ideas accumulate in a private, searchable archive that grows more valuable over time through resurfacing and theme-clustering.

### Design principles (these override convenience)
1. **Tool, not game.** Return-value comes from visible skill growth, the compounding archive, and real-world useful ideas — not points. Any gamification is a thin scaffold for the first weeks only.
2. **The practice is sacred and private.** Writing the daily list is private by default, judgment-free, and contains zero token/commerce mechanics. Sharing is always opt-in and limited.
3. **Measure the muscle, not the idea.** Progress = divergent-thinking metrics (§5). Never grade idea quality.
4. **Altucher fidelity** (§4) is a hard requirement, not a theme.

---

## 2. Target user & platform

- **Primary user (V1):** the founder + early Farcaster/Base users (builders, founders, crypto-native). Sophisticated, mobile-first, comfortable in a social feed.
- **Platform:** **web-first app that also renders as a Mini App** across Base App and other Farcaster clients from a single codebase. Mobile-first responsive UI; mini-app viewport target **424×695px** for web hosts.

---

## 3. Architecture & stack

**Decision:** Build a standard web app first, then make it mini-app-capable via MiniKit. Rationale: as of the April 9, 2026 Base App change, hosts increasingly treat apps as **standard web app + wallet** rather than a Farcaster-specific spec. MiniKit gives one codebase that runs in **both** Base App and Farcaster clients, so it's the safe primitive. Do **not** couple core logic to any single client's spec.

Recommended stack (adjust if you have strong reasons):
- **Framework:** Next.js (App Router) + TypeScript.
- **Styling:** Tailwind CSS. Clean, calm, fast. This is a contemplative tool, not a casino — restraint over confetti.
- **Mini-app integration:** `@coinbase/onchainkit/minikit` (MiniKit) as the primary cross-client layer; `@farcaster/miniapp-sdk` where Farcaster-specific calls are needed. Call the readiness hook (`useMiniKit` → `setFrameReady()` / `sdk.actions.ready()`) or users get an infinite splash.
- **Auth:** Sign In With Farcaster (SIWF) via MiniKit / AuthKit (Privy is an acceptable wrapper). Allow an email/passkey fallback so the practice works for non-Farcaster users too (Phase 1 widens the door).
- **Database:** Postgres (Supabase or Neon are fine). Persistent server-side storage for the archive is essential — the compounding archive is the core retention asset.
- **AI coach:** Anthropic Claude API (server-side; never expose keys client-side). See §6.
- **Hosting:** Vercel.
- **Runtime:** Node.js ≥ 22.11.0 (required by the Farcaster mini-app tooling).

**Fast scaffold options** (pick one, then strip to our scope):
- `npx @neynar/create-farcaster-mini-app@latest` — scaffolds auth, wallet, notifications, manifest signing, analytics.
- MiniKit CLI / `builders-garden/base-minikit-starter` template.

---

## 4. Altucher fidelity requirements (hard constraints)

- **Ramp, don't dump:** new users start at **5 ideas/day**, increasing toward 10 over their first ~2 weeks. Don't force 10 on day one.
- **Suspend judgment in the moment:** during list entry there is **no scoring, no AI commentary, no "good/bad" signal**. Just capture. All reflection happens *after* the user marks the list complete.
- **Honor "the burn":** ideas ~5–10 are where the muscle works. The UI should gently encourage pushing past idea 3–4 (e.g., "this is where it starts working") and the coach should *celebrate* reaching the harder ideas — never imply the easy first ones were the point.
- **Quantity over quality** is the instruction surfaced to the user. The app never tells a user an idea is weak.
- **The 180-day arc:** the practice pays off over ~6 months, with a documented motivation dip between **~30 and 90 days**. Concentrate retention support (see §7) in that window.
- **Prompts solve the real friction:** the hard part is "ten ideas about *what?*", not the writing. The prompt engine (§5) is core infrastructure, not a nicety.

---

## 5. V1 feature scope

### 5.1 Daily practice loop (the heart)
- Open app → see today's prompt (or choose own / tap an alternative) → enter ideas one per line → mark complete.
- Optional, non-coercive timer (off by default).
- Target count adapts to the user's ramp (5 → 10).
- Zero judgment during entry. On completion, transition to the reflection screen.

### 5.2 Prompt engine
Three blended sources:
1. **Served daily prompt** (removes friction; default).
2. **Self-chosen topic** (highest ownership; one tap to switch).
3. **Personalized prompts** — AI notices recurring themes in the user's archive and offers sharper angles ("3 lists on education lately — here's a sharper one").
- Difficulty ratchets up as fluency rises (like adding weight in a fitness app).
- Maintain a seed library of high-quality starter prompts for cold-start.

### 5.3 AI coach reflection (post-completion)
- Reflects **muscle-metrics only** (see §6): e.g. "ten in 4 min, your fastest," "all ten in one category — want a flexibility challenge tomorrow?", "#9 was your most original this week."
- Optionally surfaces **one** connection to a past idea from the archive.
- Never grades idea quality. Never uses discouraging language.
- **Open decision:** show reflection immediately vs. next morning. Build it behind a flag so both can be tested.

### 5.4 The archive (core retention asset)
- Every idea stored, searchable, browsable by date / prompt / theme.
- **Resurfacing:** periodically resurface relevant past ideas ("an idea you had 90 days ago is suddenly relevant").
- **Theme clustering:** AI tags and clusters ideas so users can see what they keep circling (seeds the future "knowledge database").
- The archive visibly *appreciates* with use. This is the thing that makes leaving feel costly.

### 5.5 Progress view
- Charts of fluency / flexibility / originality / elaboration over time.
- Calm, honest visualization — progress, not leaderboards. (No competitive or social ranking in V1.)

### 5.6 Light streak scaffold (cold-start only, must be humane)
- A "don't break the chain" streak **with forgiveness**: freezes / earn-backs. **Never** pay-to-protect. **Never** shame/guilt notifications (explicit anti-pattern — see the Duolingo-streak-anxiety cautionary case).
- Designed to **fade**: as a user's habit stabilizes (e.g. consistent 60+ day practice), dial down external nudges so intrinsic value carries retention.

### 5.7 Opt-in sharing (the only social surface in V1)
- After completing a list, the user may share **one curated idea** (their choice) as a cast via the compose-cast action (`useComposeCast` / SDK). 
- Raw 10-idea lists are **never** shared or made public in V1.
- This is the acquisition loop AND faithful to Altucher's "be a transmitter" principle.

### 5.8 Notifications (re-engagement, humane)
- A single, gentle daily reminder at a user-chosen time via the mini-app notification API (`useNotification`) + push where available.
- Frequency-capped, no guilt language, easily tunable/disable-able.

### Explicitly OUT of scope for V1
Token, wallet-based rewards, staking, curation markets, lottery, quadratic funding, public idea pool/commons, builder bounties, leaderboards, competition. See `Tenfold Roadmap.md` Phases 2–4. **Do not build these now**, but keep the data model (§8) clean enough that a public layer can be added later without migration pain.

---

## 6. AI coach specification

**Provider:** Anthropic Claude API, called server-side only.

**Job:** measure and reflect the *idea muscle*, never judge idea quality.

**Metrics to compute per session (divergent-thinking dimensions):**
- **Fluency** — count of ideas + time to produce (did they beat the blank page?).
- **Flexibility** — number of distinct *categories/angles* the list spans (10 variations of one thing = low; 10 different domains = high). Use the model to cluster the list into categories and count distinct ones.
- **Originality** — distance from common/obvious responses for that prompt. Approximate via the model's judgment of commonness; store a 0–1 score per idea.
- **Elaboration** — how developed each idea is (length/specificity is a weak proxy; have the model rate lightly).

**Hard guardrails (put these in the system prompt):**
- Never tell the user an idea is bad, weak, or wrong. No quality grading, ever.
- Frame everything as muscle performance and growth.
- Encouraging, concise, specific. One observation + at most one optional forward-looking suggestion.
- Output structured JSON (metrics + one short reflection string + optional archive-connection id) so the client renders it deterministically.

**Other AI tasks:** prompt personalization from archive themes; theme tagging/clustering for the archive; resurfacing relevance matching.

**Reference:** Claude API docs in §9. Use structured outputs; keep prompts versioned in the repo.

---

## 7. The 30–90 day danger window (special handling)

This is where most users quit. Design explicit support here:
- Around day ~21: acknowledge the shift positively ("three weeks in — this is where it gets interesting").
- Days ~30–90: increase gentle encouragement, surface *progress evidence* (their own metric curves trending up), resurface their best past ideas to remind them the archive is accruing value. Lighter targets are OK on rough days (see open decision on rigidity).
- Post-~90: begin fading external scaffolding; lean on intrinsic value.

---

## 8. Data model (minimum)

- **User:** id, fid (nullable), auth identity, display info, settings (reminder time, timer on/off, reflection-timing flag), ramp state (current target count), created_at.
- **Prompt:** id, text, source (served/self/personalized), difficulty, theme_tags.
- **Session:** id, user_id, date, prompt_id (nullable for self-chosen), duration_seconds, completed_at, computed metrics (fluency, flexibility, originality_avg, elaboration_avg).
- **Idea:** id, session_id, user_id, text, order_index, category_tag, originality_score, theme_tags, created_at. (One row per idea — needed for clustering, resurfacing, and future per-idea curation.)
- **StreakState:** user_id, current_streak, longest_streak, freezes_available, last_active_date.
- **(Reserve for later, do not build):** keep `Idea` and `Session` clean so a future `public_visibility`, `stake`, or `curation` table can attach without migration pain.

Privacy: ideas are private by default. Only an explicitly shared single idea is ever exposed externally.

---

## 9. Reference documentation (fetch these before building the relevant parts)

**Farcaster Mini Apps (LLM-friendly docs):**
- Getting started: https://miniapps.farcaster.xyz/docs/getting-started
- Specification (manifest, `fc:miniapp` meta tag, embeds): https://miniapps.farcaster.xyz/docs/specification
- Publishing & verification: https://miniapps.farcaster.xyz/docs/guides/publishing
- SDK package: `@farcaster/miniapp-sdk` (remember `sdk.actions.ready()`)
- Sign In With Farcaster / AuthKit: https://docs.farcaster.xyz/

**Base / MiniKit (primary cross-client layer; important post-April-2026 shift):**
- MiniKit overview: https://docs.base.org/builderkits/minikit/overview
- MiniKit quickstart: https://docs.base.org/builderkits/minikit/quickstart
- MiniKit debugging: https://docs.base.org/builderkits/minikit/debugging
- Base wallet mini apps: https://docs.base.org/wallet-app/mini-apps
- "Thinking social" design guide: https://docs.base.org/builderkits/minikit/thinking-social
- OnchainKit AI prompting guide (useful for this build): https://docs.base.org/builderkits/onchainkit/guides/ai-prompting-guide
- Base Builder MCP (optional — gives an agent live access to Base docs/codegen): https://github.com/base/base-builder-mcp
- Coinbase Developer Portal (API keys): https://portal.cdp.coinbase.com/

**Neynar (Farcaster infra / fastest scaffold):**
- **Neynar MCP server** — use it throughout the build for live Farcaster/Neynar docs and API reference instead of relying on training data.
- Docs: https://docs.neynar.com
- Scaffold: `npx @neynar/create-farcaster-mini-app@latest`

**Anthropic (AI coach + this build agent):**
- Claude API overview: https://docs.claude.com/en/api/overview
- Claude API docs map: https://docs.claude.com/en/docs_site_map.md
- Claude Code overview: https://docs.claude.com/en/docs/claude-code/overview
- Claude Code docs map: https://docs.anthropic.com/en/docs/claude-code/claude_code_docs_map.md

**Bankr (Phase 3 only — token launcher; DO NOT integrate in V1):**
- https://bankr.bot — note: creator earns ~0.4%+ of each swap on tokens launched via its rails. Out of scope until the Phase 3 token-readiness checklist in `Tenfold Roadmap.md` is fully satisfied.

---

## 10. Build milestones (do in order; confirm each runs before moving on)

1. **Skeleton:** Next.js + TS + Tailwind app; DB connected; deploys to Vercel. No mini-app yet — just a working web app you can open in a browser.
2. **Practice loop:** prompt display, idea entry (one per line), completion, persistence to DB. Ramp logic (5→10). Zero judgment during entry.
3. **AI coach:** server-side Claude call computing the four muscle-metrics + one reflection string as structured JSON; reflection screen renders it. Guardrails enforced.
4. **Archive + progress:** searchable archive, theme tagging/clustering, resurfacing; progress charts over time.
5. **Streak scaffold:** humane streak with freezes/earn-backs; fade logic stubbed; humane reminder notification.
6. **Mini-app layer:** integrate MiniKit; signed `farcaster.json` manifest; readiness hook; SIWF auth (+ email/passkey fallback); verify it loads inside Base App and a Farcaster client. Target 424×695 web viewport.
7. **Opt-in sharing:** compose-cast of a single curated idea; dynamic share/embed preview. Confirm raw lists are never exposed.
8. **Polish + instrument:** calm UI pass; analytics for the Phase-1 gate metric (**Day-60 practice retention**) and the 30–90 day funnel.

**Definition of done for V1:** a deployable app where a user can do the daily practice, get muscle-metric reflection, watch an archive compound, receive humane reminders, optionally share one idea to a feed — with **no** token/commerce anywhere, and instrumentation in place to measure Day-60 retention.

---

## 11. Guardrail checklist (review before every release)

- [ ] No idea is ever graded "good/bad" anywhere in the product.
- [ ] No judgment or AI commentary appears *during* list entry.
- [ ] Raw 10-idea lists are never shared or public.
- [ ] No pay-to-protect-streak, no guilt/shame notifications.
- [ ] No token, wallet-reward, staking, or competition mechanics in V1.
- [ ] New users ramp from 5; not forced to 10 on day one.
- [ ] API keys are server-side only.
- [ ] The practice works without a crypto wallet (fallback auth present).
