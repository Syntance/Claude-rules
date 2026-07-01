# Claude-rules

Reguły i skille dla **Claude Code** — port filozofii Syntance (klasa Awwwards + produkcyjna niezawodność) z [`cursor-rules`](https://github.com/Syntance/cursor-rules). Claude buduje strony i sklepy na poziomie najlepszych studiów: backend, frontend, UX/UI, bezpieczeństwo, wydajność oraz zgodność prawna PL/EU.

Zaprojektowane pod **niski koszt kontekstu** (progressive disclosure): mały `CLAUDE.md` ładowany zawsze + skille ładowane dopiero gdy potrzebne.

## Jak to działa (3 warstwy)

| Tier | Co | Kiedy ładowane | Koszt |
| --- | --- | --- | --- |
| **0 — `CLAUDE.md`** | rola, inwarianty, rejestr skilli, komendy | KAŻDE wywołanie modelu | stały (dlatego krótki) |
| **1 — `skills/*/SKILL.md`** | wiedza domenowa (design, security, checkout…) | dopiero gdy Claude uzna za potrzebne lub `/nazwa` | tylko gdy użyte |
| **2 — kod `Syntance/moduly`** | sprawdzony kod do skopiowania | gdy realnie implementujesz | tylko przy czytaniu |

Zasada: `CLAUDE.md` < ~180 linii, każdy skill zwięzły. Dzięki temu komplet wiedzy nie obciąża każdego zapytania.

## Struktura

```
Claude-rules/
├── CLAUDE.md                        # Tier 0 — orchestrator
├── skills/
│   ├── syntance-frontend-stack/
│   ├── syntance-design/
│   ├── syntance-motion-3d/
│   ├── syntance-conversion-cro/
│   ├── syntance-strategia-negacz/   # NOWE: strategia B2B (SellWise)
│   ├── syntance-perf-a11y-seo-geo/  # + GEO + pętla PageSpeed
│   ├── syntance-security/
│   ├── syntance-legal-pl-eu/
│   ├── syntance-backend-api/        # NOWE: niezawodność integracji/API
│   ├── syntance-commerce-medusa/
│   ├── syntance-checkout-payments/
│   ├── syntance-magazyn-panel/
│   ├── syntance-quality-release/
│   └── syntance-moduly-deploy/
└── README.md
```

## Instalacja skilli Syntance

### Wariant A — per projekt (commitowalne, zalecane)
```bash
# w katalogu projektu
pnpm dlx degit Syntance/Claude-rules/skills .claude/skills
pnpm dlx degit Syntance/Claude-rules/CLAUDE.md ./CLAUDE.md   # lub skopiuj ręcznie
```
Claude Code automatycznie wykrywa `.claude/skills/` i `CLAUDE.md` w rootcie. Zaakceptuj workspace trust.

### Wariant B — globalnie (wszystkie projekty na tym komputerze)
Skopiuj `skills/*` do `~/.claude/skills/` oraz `CLAUDE.md` do `~/.claude/CLAUDE.md`.

Weryfikacja: `/context` (co się załadowało), `/doctor` (czy opisy skilli nie są obcinane), `/skill-name` (ręczne wywołanie).

## Skille i narzędzia zewnętrzne (zainstaluj)

Te skille/MCP realizują wymogi jakości i są **tanie** (ładują się on-demand).

### Design / UI
```bash
# UI/UX Pro Max — design intelligence (palety, typografia, wzorce UI)
npm install -g uipro-cli
uipro init --ai claude            # instaluje do .claude/skills/

# 21st.dev Magic MCP — generowanie komponentów (/ui). Klucz z 21st.dev/mcp
claude mcp add magic --scope user --env API_KEY="twoj-klucz"
```

### Dokumentacja i wydajność (MCP)
```bash
# context7 — aktualne API bibliotek (zamiast zgadywać wersje)
claude mcp add context7 --scope user -- npx -y @upstash/context7-mcp

# Chrome DevTools MCP — audyt PageSpeed / Core Web Vitals
claude mcp add chrome-devtools --scope user -- npx -y @chromedevtools/chrome-devtools-mcp

# Playwright MCP — E2E i a11y na żywo
claude mcp add playwright --scope user -- npx -y @playwright/mcp
```

### Skille z marketplace (pluginy)
```bash
# Oficjalne Anthropic
/plugin marketplace add anthropics/claude-plugins-official
/plugin install skill-creator@claude-plugins-official
/plugin marketplace add anthropics/skills
/plugin install webapp-testing@anthropic-agent-skills

# Vercel (stack Next.js/deploy): nextjs, react-best-practices, shadcn,
# deployments-cicd, vercel-cli, env-vars, verification, (ai-sdk gdy AI)
# Cloudflare: web-perf (audyt PageSpeed przez Chrome DevTools MCP)
```
> Nazwy marketplace/pluginów potwierdź aktualnym `/plugin marketplace` — ekosystem się zmienia. Po instalacji: `/reload-plugins`.

## Kiedy który skill (skrót)

Pełny rejestr w `CLAUDE.md`. W skrócie: setup → `frontend-stack`; wygląd → `design` (+ UI/UX Pro Max, `/ui`); ruch/3D → `motion-3d`; konwersja → `conversion-cro`; strona B2B → `strategia-negacz`; szybkość/SEO/GEO → `perf-a11y-seo-geo` (+ web-perf); bezpieczeństwo → `security`; prawo → `legal-pl-eu`; API/integracje → `backend-api`; sklep → `commerce-medusa`; checkout → `checkout-payments`; panel → `magazyn-panel`; testy/deploy → `quality-release`; gotowe moduły → `moduly-deploy`.

## Relacja do cursor-rules
To repo jest odpowiednikiem [`cursor-rules`](https://github.com/Syntance/cursor-rules) dla Claude Code. Kod źródłowy do kopiowania: [`Syntance/moduly`](https://github.com/Syntance/moduly).
