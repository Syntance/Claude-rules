---
name: syntance-graphify-workflow
description: Graph-first, cost-optimized codebase workflow using graphify. Activate for debugging, refactoring, impact analysis, and planning/redesign of whole systems in any repo with graphify-out/. Replaces grep/read exploration with free local graph queries; reserves expensive models (Opus / Fable 5) for synthesis and edits only.
---

# Graph-first workflow (cost-optimized)

Core rule: **assemble context for free, spend expensive tokens only on decisions and edits.**
`graphify query / path / explain / affected / update` are local JSON traversals — 0 LLM calls, 0 API cost, 1–5 s each.

## Cost ladder (always go top-down)
1. **FREE** — `graphify query/path/explain/affected`, `graphify-out/GRAPH_REPORT.md`, `graphify-out/wiki/index.md`
2. **CHEAP** — read ONLY the 1–3 specific files the graph pointed to (with line ranges)
3. **EXPENSIVE** — Opus / Fable 5 reasoning on assembled context, final edits
Never invert the ladder: no directory listing, no multi-file reads, no grep until the graph has oriented you.

## Query discipline
- One scoped query > several broad ones: `graphify query "why does X fail when Y" --budget 1500`
- Narrow question → lower budget (`--budget 800`). Default cap: always pass `--budget`.
- Trace a single flow → `--dfs`; broad orientation → default BFS.
- Symbol details → `graphify explain "Symbol"` (not query).
- Relationship between two things → `graphify path "A" "B"` (not query).

## DEBUG loop
1. `graphify query "<symptom>: <error/behavior>" --budget 1500`
2. `graphify path "<entry point>" "<failing symbol>"` — the call chain
3. `graphify explain "<suspect>"` — neighbors + work-memory lessons
4. Read ONLY the files surfaced (usually 1–3), targeted line ranges
5. Fix → `graphify update .` (AST-only, free, <5 s)
6. `graphify save-result --outcome useful|dead_end|corrected` — feeds work memory so future queries rank better
Never: grep the repo for a string when a graph exists; read a directory "for context".

## REFACTOR / rebuild loop
1. BEFORE touching anything: `graphify affected "<symbol>" --depth 2` — the impact radius (what breaks)
2. `graphify explain` each high-degree node in the radius (god nodes = danger zones)
3. Plan edits per-file from the radius — not by discovery-while-editing
4. Edit → `graphify update .` → re-run `affected` to confirm the radius is closed
Skipping step 1 is the #1 source of expensive fix-the-fix loops.

## PLAN / system redesign loop
Planning burns the most Opus/Fable tokens — front-load every free artifact first:
1. `graphify-out/GRAPH_REPORT.md` — architecture, communities, god nodes (already generated, free)
2. `graphify query "current architecture of <area>" --budget 2000`
3. God nodes = coupling hotspots — the top candidates to break up in a redesign
4. Communities = natural module boundaries — align the new design with them, or explicitly justify why not
5. `graphify affected "<module>"` for each part you intend to replace — the migration blast radius
6. Only THEN escalate to Opus / Fable 5 for the actual design decision, pasting graph output as context
7. The written plan lists affected files per phase **from the graph**, not from guesses
Never load raw source into a planning session before graph artifacts are exhausted.

## Model routing
- Orientation, running graph queries, reading surfaced files → cheap model (Sonnet/Haiku) or a subagent
- Exploration subagents: cheap model, include this graph-first rule in the prompt, return a summary only
- Opus / Fable 5 → final architecture decisions, hard multi-constraint debugging, critical edits
- Escalate only WITH assembled context. Never let an expensive model list directories or read files "to get oriented".

## Graph hygiene (keeps everything free)
- After each edit session: `graphify update .` (git hook also runs it on commit)
- Full rebuild only after massive refactors: `graphify extract . --force`
- Repo without a graph → build once (`/graphify .`), then this skill applies

## Anti-patterns (forbidden)
- Grep/read exploration in a repo that has `graphify-out/graph.json`
- `graphify query` without `--budget`
- Reading GRAPH_REPORT.md for a narrow question (query output is ~10× smaller)
- Rebuilding the graph when `update` suffices
- Opus / Fable 5 doing raw-file exploration of any kind
