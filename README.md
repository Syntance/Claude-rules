# Claude-rules

Rules and skills for **Claude Code** — a port of the Syntance philosophy (Awwwards-caliber craft + production-grade reliability) from [`cursor-rules`](https://github.com/Syntance/cursor-rules). Claude builds sites and stores at the level of top-tier studios: backend, frontend, UX/UI, security, performance, and PL/EU legal compliance.

Designed for **low context cost** (progressive disclosure): a small `CLAUDE.md` loaded always + skills loaded only when needed.

## How it works (3 layers)

| Tier | What | Loaded | Cost |
| --- | --- | --- | --- |
| **0 — `CLAUDE.md`** | role, invariants, skill registry, commands | EVERY model call | fixed (keep it short) |
| **1 — `skills/*/SKILL.md`** | domain knowledge (design, security, checkout…) | only when Claude decides it's needed, or via `/name` | only when used |
| **2 — `Syntance/moduly` code** | proven code to copy | only when actually implementing | only when read |

Rule of thumb: `CLAUDE.md` stays under ~180 lines, each skill is concise. This way the full body of knowledge never taxes every single request.

## Structure

```
Claude-rules/
├── CLAUDE.md                        # Tier 0 — orchestrator
├── skills/
│   ├── syntance-frontend-stack/
│   ├── syntance-design/
│   ├── syntance-motion-3d/
│   ├── syntance-conversion-cro/
│   ├── syntance-strategia-negacz/   # B2B strategy (SellWise-inspired)
│   ├── syntance-perf-a11y-seo-geo/  # + GEO + PageSpeed audit loop
│   ├── syntance-security/
│   ├── syntance-legal-pl-eu/
│   ├── syntance-backend-api/        # integration/API reliability
│   ├── syntance-commerce-medusa/
│   ├── syntance-checkout-payments/
│   ├── syntance-magazyn-panel/
│   ├── syntance-quality-release/
│   ├── syntance-moduly-deploy/
│   └── syntance-graphify-workflow/  # graph-first debug/refactor/plan (cost control)
└── README.md
```

## Installing the Syntance skills

### Option A — global (all projects on this machine, recommended for solo/agency use)
Copy `skills/*` into `~/.claude/skills/` and `CLAUDE.md` into `~/.claude/CLAUDE.md`. Claude Code auto-discovers both locations at startup — no per-project setup needed afterward.

### Option B — per project (committable, shared with your team)
```bash
# inside the project directory
pnpm dlx degit Syntance/Claude-rules/skills .claude/skills
# copy or symlink CLAUDE.md into the project root as well
```
Claude Code auto-detects `.claude/skills/` and a root `CLAUDE.md`. Accept the workspace trust prompt.

Verify with `/context` (what loaded), `/doctor` (are skill descriptions being truncated?), or `/skill-name` (manual invocation).

## External skills and tools to install

These skills/MCP servers fulfill the design/quality bar and are **cheap** (loaded on-demand).

### Design / UI
```bash
# UI/UX Pro Max — design intelligence (palettes, typography, UI patterns)
npm install -g uipro-cli
uipro init --ai claude            # installs into .claude/skills/

# 21st.dev Magic MCP — component generation (/ui). Get an API key at 21st.dev/mcp
claude mcp add magic --scope user --env API_KEY="your-key"
```

### Documentation and performance (MCP)
```bash
# context7 — up-to-date library APIs (instead of guessing versions)
claude mcp add context7 --scope user -- npx -y @upstash/context7-mcp

# Chrome DevTools MCP — PageSpeed / Core Web Vitals audits
claude mcp add chrome-devtools --scope user -- npx -y @chromedevtools/chrome-devtools-mcp

# Playwright MCP — live E2E and a11y testing
claude mcp add playwright --scope user -- npx -y @playwright/mcp
```

### Marketplace skills (plugins)
```bash
# Official Anthropic
/plugin marketplace add anthropics/claude-plugins-official
/plugin install skill-creator@claude-plugins-official
/plugin marketplace add anthropics/skills
/plugin install webapp-testing@anthropic-agent-skills

# Vercel (Next.js/deploy stack): nextjs, react-best-practices, shadcn,
# deployments-cicd, vercel-cli, env-vars, verification, (ai-sdk if using AI)
# Cloudflare: web-perf (PageSpeed audits via Chrome DevTools MCP)
```
> Confirm exact marketplace/plugin names with `/plugin marketplace` — the ecosystem evolves. After installing: `/reload-plugins`.

## Which skill for which task (quick reference)

Full registry lives in `CLAUDE.md`. In short: project setup → `frontend-stack`; visuals → `design` (+ UI/UX Pro Max, `/ui`); motion/3D → `motion-3d`; conversion → `conversion-cro`; B2B site → `strategia-negacz`; speed/SEO/GEO → `perf-a11y-seo-geo` (+ web-perf); security → `security`; compliance → `legal-pl-eu`; API/integrations → `backend-api`; store → `commerce-medusa`; checkout → `checkout-payments`; admin panel → `magazyn-panel`; testing/deploy → `quality-release`; ready-made modules → `moduly-deploy`; debugging/refactor/redesign with a knowledge graph → `graphify-workflow`.

## Relation to cursor-rules
This repo is the Claude Code counterpart to [`cursor-rules`](https://github.com/Syntance/cursor-rules). Source code to copy from: [`Syntance/moduly`](https://github.com/Syntance/moduly).
