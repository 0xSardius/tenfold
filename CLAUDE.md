# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current state

This repo is **planning-stage** — it contains only `Tenfold PRD.md` and `Tenfold Roadmap.md`. There is no code, build system, or dependencies yet. The first task (PRD §10 milestone 1) is to scaffold the app. `Tenfold PRD.md` is the authoritative build spec; read it in full before writing code. `Tenfold Roadmap.md` is the longer-term phase plan and is mostly out of V1 scope.

Remote: https://github.com/0xSardius/tenfold.git

## Session workflow

- **At session start:** read `docs/CHECKPOINT.md` and recent git history (`git log --oneline -10`) to resume context.
- **At session end / natural checkpoints:** update `docs/CHECKPOINT.md` (current state, what's done, what's next, open decisions), commit, and push. Commit after each completed milestone — milestones are the natural checkpoints for this project.
- The checkpoint should always name the **current PRD §10 milestone** and its verification status (each milestone must be confirmed running before the next begins).

## What is being built

**Tenfold**: a daily practice app based on James Altucher's "10 ideas a day" method. The user writes a list of ideas to a prompt; an AI coach reflects back *how their idea-generation muscle is performing* (divergent-thinking metrics) and **never grades idea quality**. Ideas accumulate in a private, compounding archive. It's a web-first app that also renders as a Mini App across Base App and Farcaster clients from a single codebase.

## Intended stack (PRD §3 — adjust only with strong reason)

- **Framework:** Next.js (App Router) + TypeScript
- **Styling:** Tailwind CSS — calm/contemplative, "restraint over confetti"
- **Mini-app layer:** `@farcaster/miniapp-sdk` directly + Neynar infra (`@neynar/nodejs-sdk` + APIs) — **no MiniKit/OnchainKit** (founder decision 2026-06-06; the Farcaster SDK is the common primitive and runs in both Farcaster clients and Base App). You **must** call `sdk.actions.ready()` once loaded or users get an infinite splash.
- **Auth:** Sign In With Farcaster (SIWF) via Neynar (SIWN) or AuthKit, **with an email/passkey fallback** so the practice works without a crypto wallet.
- **Database:** Postgres on **Neon** with **Drizzle ORM** (decided 2026-06-06). Real server-side persistence is required — the compounding archive is the core retention asset, not optional.
- **AI coach:** Anthropic Claude API, **server-side only** (never expose keys client-side). Use structured JSON outputs; keep prompts versioned in the repo.
- **Hosting:** Vercel. **Runtime:** Node.js ≥ 22.11.0 (required by Farcaster mini-app tooling).

Scaffold (decided): `npx @neynar/create-farcaster-mini-app@latest`, then strip to scope — remove wallet/token surfaces not needed in V1.

## Hard product constraints (these override convenience — see PRD §4, §11)

These are not stylistic preferences; violating them breaks the method. Treat the PRD §11 checklist as a release gate.

- **Never grade an idea "good/bad" anywhere** — measure the muscle (fluency, flexibility, originality, elaboration), not the idea.
- **No judgment, scoring, or AI commentary *during* list entry** — all reflection happens only after the user marks the list complete.
- **Ramp, don't dump:** new users start at **5 ideas/day**, increasing toward 10 over ~2 weeks. Never force 10 on day one.
- **The practice is private and token-free, forever.** Raw 10-idea lists are never shared or made public. Only an explicitly chosen *single* idea may be shared (as a cast). No token, wallet-reward, staking, lottery, leaderboard, or competition mechanics in V1.
- **Humane streaks only:** forgiveness/freezes, no pay-to-protect, no guilt/shame notifications. The streak scaffold is designed to *fade* as the habit stabilizes.
- Concentrate retention support in the **30–90 day motivation dip** (PRD §7).

## Build order (PRD §10 — do in order, confirm each runs before moving on)

1. Skeleton: Neynar scaffold stripped to scope (mini-app-capable from day 1), Neon DB connected, deploys to Vercel.
2. Practice loop: prompt display → idea entry (one per line) → completion → DB persistence; ramp logic (5→10); zero judgment during entry.
3. AI coach: server-side Claude call computing the four muscle-metrics + one reflection string as structured JSON; reflection screen.
4. Archive + progress: searchable archive, theme tagging/clustering, resurfacing; progress charts.
5. Streak scaffold: humane streak with freezes/earn-backs; humane reminder notification.
6. Mini-app completion: signed `farcaster.json` manifest verified, SIWF/SIWN auth (+ fallback), notifications via Neynar; verify it loads in Base App and a Farcaster client. Target 424×695 web viewport.
7. Opt-in sharing: compose-cast of a single curated idea; dynamic share/embed preview.
8. Polish + instrument: calm UI pass; analytics for the Phase-1 gate metric (**Day-60 retention**).

## Data model & future-proofing (PRD §8)

Core entities: **User** (incl. nullable `fid`, ramp state, settings), **Prompt** (text, source, difficulty, theme_tags), **Session** (per-day, with computed metrics), **Idea** (one row per idea — needed for clustering/resurfacing/future curation), **StreakState**. Ideas are private by default. **Do not build** token/curation/public tables now, but keep `Idea` and `Session` clean so a `public_visibility` / `stake` / `curation` layer can attach later without migration pain.

## Reference docs — fetch before integration work (PRD §9)

This ecosystem shifted in early 2026; prefer official docs over training data. The Farcaster and Base docs are LLM-friendly — pull them into context before writing SDK/manifest/API code.

- **Neynar MCP server:** the project uses the Neynar MCP for live Farcaster/Neynar docs and API reference. **Use it throughout** — prefer it over training data for anything Farcaster-related (SIWF, casts, manifest, notifications, user data). If its tools aren't available in the session, tell the user so they can reconnect it, and fall back to https://docs.neynar.com.
- **Local doc dumps (searchable offline):** `docs/farcaster-miniapps-docs.md` (full Farcaster Mini Apps docs) and `docs/neynar-docs-full.md` (full Neynar docs, ~58k lines). Grep these for SDK/API details before reaching for the web; they're too large to read whole.
- Farcaster Mini Apps: https://miniapps.farcaster.xyz/docs/getting-started · spec: https://miniapps.farcaster.xyz/docs/specification
- Base App host compatibility: https://docs.base.org/wallet-app/mini-apps
- Claude API: https://docs.claude.com/en/api/overview
- Neynar: https://docs.neynar.com

Note the April 9 2026 Base App change: hosts increasingly treat apps as **standard web app + wallet** rather than a Farcaster-specific spec. Do not couple core logic to any single client's spec — `@farcaster/miniapp-sdk` is the common primitive (Base App hosts standard Farcaster mini apps); MiniKit/OnchainKit is deliberately not used.
