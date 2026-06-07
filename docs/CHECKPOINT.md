# CHECKPOINT — Tenfold

**Last updated:** 2026-06-06

## Current state

**Phase:** Planning complete. No code yet.
**Current milestone:** PRD §10 **Milestone 1 — Skeleton** (not started): Next.js + TS + Tailwind app, DB connected, deploys to Vercel. Plain web app only — no mini-app layer yet.

## Done so far

- PRD and Roadmap written (`Tenfold PRD.md`, `Tenfold Roadmap.md`)
- Name decided: **Tenfold** (was working title "10ideas"); docs renamed and cross-references fixed
- `CLAUDE.md` created: stack, hard constraints, milestone order, reference docs
- Git initialized on `main`, remote `https://github.com/0xSardius/tenfold.git`, initial commit pushed
- Neynar MCP server registered and healthy (`claude mcp list` → ✓ Connected)
- **Architecture decisions locked (2026-06-06):** DB = **Neon + Drizzle ORM**; **no MiniKit/OnchainKit** — use `@farcaster/miniapp-sdk` + Neynar from day 1; scaffold = `npx @neynar/create-farcaster-mini-app@latest` then strip to scope. PRD §3/§9/§10 and CLAUDE.md updated to match.
- Environment verified: Node v22.16.0 ✓, npm 11.4.2, Vercel CLI 50.1.6

## Next steps

1. **Verify Neynar MCP tools load** in the new session (server was added mid-session; needed a restart to appear). If absent, tell the founder.
2. **Milestone 1 — Skeleton:** run the Neynar scaffold, strip to scope (remove wallet/token surfaces), connect Neon Postgres, deploy to Vercel. Verify output directory structure with `ls` before writing code (global rule).
3. Founder prerequisites for milestone 1: Neon account/project, Vercel project link. (Anthropic API key not needed until milestone 3.)
4. Confirm milestone 1 runs end-to-end before starting milestone 2 (practice loop).

## Open decisions (founder)

- Reflection timing (immediate vs next morning) — build behind a flag, PRD §5.3
- Bad-day rigidity (fewer than target allowed?) — Roadmap open decision

## Notes / gotchas

- Node.js ≥ 22.11.0 required by Farcaster mini-app tooling
- Watch for nested-directory output when scaffolding (global rule: verify with `ls` before writing code)
- Hard product constraints in `CLAUDE.md` / PRD §4 — review the PRD §11 guardrail checklist before any release
