# CHECKPOINT — Tenfold

**Last updated:** 2026-06-03

## Current state

**Phase:** Planning complete. No code yet.
**Current milestone:** PRD §10 **Milestone 1 — Skeleton** (not started): Next.js + TS + Tailwind app, DB connected, deploys to Vercel. Plain web app only — no mini-app layer yet.

## Done so far

- PRD and Roadmap written (`Tenfold PRD.md`, `Tenfold Roadmap.md`)
- Name decided: **Tenfold** (was working title "10ideas"); docs renamed and cross-references fixed
- `CLAUDE.md` created: stack, hard constraints, milestone order, reference docs
- Git initialized on `main`, remote `https://github.com/0xSardius/tenfold.git`, initial commit pushed
- Neynar MCP server registered and healthy (`claude mcp list` → ✓ Connected)

## Next steps

1. **Verify Neynar MCP tools load** in the new session (server was added mid-session; needed a restart to appear). If absent, tell the founder.
2. **Milestone 1 — Skeleton:** scaffold Next.js (App Router) + TypeScript + Tailwind; connect Postgres (Supabase or Neon — founder hasn't picked yet); deploy to Vercel. Consider `npx @neynar/create-farcaster-mini-app@latest` then strip to scope, or scaffold plain and add MiniKit at milestone 6.
3. Confirm milestone 1 runs end-to-end before starting milestone 2 (practice loop).

## Open decisions (founder)

- DB provider: Supabase vs Neon
- Scaffold path: Neynar scaffold-then-strip vs plain create-next-app
- Reflection timing (immediate vs next morning) — build behind a flag, PRD §5.3
- Bad-day rigidity (fewer than target allowed?) — Roadmap open decision

## Notes / gotchas

- Node.js ≥ 22.11.0 required by Farcaster mini-app tooling
- Watch for nested-directory output when scaffolding (global rule: verify with `ls` before writing code)
- Hard product constraints in `CLAUDE.md` / PRD §4 — review the PRD §11 guardrail checklist before any release
