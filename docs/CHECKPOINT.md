# CHECKPOINT — Tenfold

**Last updated:** 2026-06-10

## Resume in one line

Planning + decisions are done; nothing is built yet. Next action = **scaffold Milestone 1** (Neynar mini-app + Neon/Drizzle + Vercel). Start a **fresh session first** so the Neynar MCP tools load, then read this file + `git log --oneline -10`.

## Current state

**Phase:** Planning complete; architecture decisions locked. **No application code yet** — repo holds docs + config only.
**Current milestone:** PRD §10 **Milestone 1 — Skeleton** (not started).

Repo contents so far: `Tenfold PRD.md`, `Tenfold Roadmap.md`, `CLAUDE.md`, `.gitignore`, `.env.example`, `.env.local` (gitignored), `docs/CHECKPOINT.md`, `docs/farcaster-miniapps-docs.md`, `docs/neynar-docs-full.md`. All pushed to `origin/main` (https://github.com/0xSardius/tenfold.git).

## Stack — LOCKED decisions (don't relitigate without reason)

- **Framework:** Next.js (App Router) + TypeScript + Tailwind
- **Mini-app:** `@farcaster/miniapp-sdk` directly + Neynar infra — **NO MiniKit/OnchainKit** (founder decision 2026-06-06)
- **DB:** Neon Postgres + **Drizzle ORM** (Drizzle Kit for migrations)
- **Auth:** SIWF via Neynar (SIWN) / AuthKit, + email/passkey fallback
- **AI coach:** Anthropic Claude API, server-side only (milestone 3)
- **Scaffold:** `npx @neynar/create-farcaster-mini-app@latest`, then strip to V1 scope
- **Hosting:** Vercel. **Runtime:** Node v22.16.0 ✓ (≥22.11 required). Vercel CLI 50.1.6 ✓.

## Env / account setup status

Real values live in `.env.local` (gitignored — never commit). Template + notes in `.env.example`.

- ✅ `DATABASE_URL` (Neon pooled) — **filled in** by founder
- ⬜ `DATABASE_URL_UNPOOLED` (Neon direct, for Drizzle migrations) — **pending** (get the non-pooler/direct string from the Neon dashboard)
- ⬜ `NEYNAR_API_KEY` — pending (dev.neynar.com)
- ⬜ `NEXT_PUBLIC_NEYNAR_CLIENT_ID` — pending (Neynar app; also add `localhost` + prod domain to Authorized Origins)
- ✅ `NEXT_PUBLIC_URL` = http://localhost:3000 (set)
- ⬜ `ANTHROPIC_API_KEY` — not needed until milestone 3
- ⬜ `FARCASTER_HEADER/PAYLOAD/SIGNATURE` — generated during milestone 6 manifest signing

**Minimum to start the milestone-1 build:** `DATABASE_URL_UNPOOLED`, `NEYNAR_API_KEY`, `NEXT_PUBLIC_NEYNAR_CLIENT_ID`. Also confirm `vercel login` works (`! vercel whoami`).

## Next steps — Milestone 1 (Skeleton)

1. **Verify Neynar MCP tools load** (server is registered + healthy but needs a fresh session to appear in the tool registry). If absent, tell the founder.
2. **Scaffold carefully — avoid the nested-directory trap.** The Neynar generator creates a NEW subdirectory and writes its own `.gitignore`/`.env`. Plan: scaffold into a temp dir, then lift the app files to the repo root — **merge** into the existing `.gitignore` (don't overwrite) and **do not clobber `.env.local`**. Run `ls` to verify structure before writing any further code (global rule).
3. **Strip to V1 scope:** remove wallet/token/swap surfaces the generator adds; keep mini-app manifest + `sdk.actions.ready()`.
4. **Wire Neon + Drizzle:** install `drizzle-orm` + `drizzle-kit`; point migrations at `DATABASE_URL_UNPOOLED`, runtime at `DATABASE_URL`; implement the PRD §8 schema (User, Prompt, Session, Idea, StreakState). Keep `Idea`/`Session` clean for a future public layer (no migration pain later).
5. **Boot + deploy:** `npm run dev` runs locally; deploy to Vercel; set the same env vars in the Vercel project.
6. **Confirm milestone 1 runs end-to-end before starting milestone 2** (practice loop). Then update this checkpoint, commit, push.

## Hard product constraints (carry into every milestone — PRD §4, §11)

Never grade ideas good/bad; no judgment during list entry; ramp 5→10 (never force 10 day one); practice stays private + token-free; raw lists never shared (only one opt-in curated idea); humane streaks only (no pay-to-protect, no guilt); API keys server-side only; practice works without a wallet (fallback auth). Review PRD §11 checklist before any release.

## Open decisions (founder)

- Reflection timing: immediate vs next morning — build behind a flag (PRD §5.3)
- Bad-day rigidity: allow fewer than target on rough days? (Roadmap open decision)

## How to resume (commands)

```
# in the project dir, in a FRESH claude session:
git log --oneline -10        # see recent history
git status                   # confirm clean tree
claude mcp list              # confirm Neynar shows ✓ Connected
! vercel whoami              # confirm Vercel auth
```
Then say: "resume Tenfold milestone 1" and follow the Next steps above.
