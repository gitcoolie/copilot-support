# PLAN PROJEKTU

## Cel główny

Wycisnąć z GitHub Copilota w JetBrains (środowisko GFT) maksimum: dopasowanie do konwencji każdego serwisu, agentowa praca z self-improvement, brak overengineering, low friction codzienna współpraca.

---

## Kamienie milowe

- [x] **Milestone 1:** v1 bootstrap prompta
  - Opis: 6 faz + self-improvement + no-overengineering + zaktualizowane do JetBrains plugin 1.5.65 (instructions overlays, prompts, lessons file)
  - Deadline: 2026-05-06 ✅

- [x] **Milestone 1.5:** v2 prompt + upgrade prompt v1→v2 (po update plugina do 1.6.1-243)
  - Opis: Native AGENTS.md/CLAUDE.md (z nested), 4-agent advisor architecture (architect/coder/reviewer/debugger) z `handoffs`, dual-mode upgrade prompt (interactive/autonomous), Phase 6 zaktualizowany o `/memory`, inline agent, auto-approve, MCP auto-approve
  - Deadline: 2026-05-07 ✅
  - **POST-MORTEM (2026-05-08):** v2 oparte na założeniu że `handoffs:` i per-agent `model:` działają w JetBrains. DR (raport: research/dr_report_2026-05-08_jetbrains_agents.md) wykazał że to jest VS Code-style i nieudokumentowane / unstable / ignorowane w JetBrains 1.6.x. v2 zostaje dla VS Code, JetBrains przeszedł na v3.

- [x] **Milestone 1.6:** v3 prompt + upgrade v2→v3 + DR raport (po DR z 2026-05-08)
  - Opis: official-safe format (tylko pola udokumentowane przez GitHub dla JetBrains: name, description, tools, user-invocable, disable-model-invocation; bez handoffs, bez model). Architektura: 1 main (Architect) + 3 helpers, max 1 poziom delegacji, model jednolity per sesja. Wieloetapowe workflows przez `.github/prompts/full-feature-loop.prompt.md`. DR raport jako trwała lekcja w `research/`.
  - Deadline: 2026-05-08 ✅

- [x] **Milestone 1.7:** v4 prompt + upgrade v3→v4 — single CTO agent (EXPERIMENTAL)
  - Opis: alternatywa architektoniczna do v3. Jeden uniwersalny agent CTO inspirowany TEAM/CTO.md (Werner Vogels-style: pragmatyzm, primitives, build it run it). Polski język, symulator perspektyw, Protokół Zero, Zero Halucynacji, Weryfikacja Sędziego. Robi sam: plan/impl/debug/review w jednej sesji. v3 zostaje stable, v4 do testowania równolegle.
  - Deadline: 2026-05-27 ✅

- [x] **Milestone 1.8:** Multi-repo TechLead dla VS Code workspace
  - Opis: nowy wymiar — VS Code workspace nadrzędny z multi-root daje widok kilku repo GFT jednocześnie. Generuje workspace-level `TechLead` (Solution Architect): mapuje serwisy, audytuje cross-cutting, planuje cross-service refactor. Komunikacja po polsku, artifakty pisane po angielsku (shareable). Korzysta z VS Code'owych ficzerów: handoffs (działa!), model fallback array. Chain: TechLead → CTO per-repo (handoff do konkretnego serwisu, mokra na execution).
  - Deadline: 2026-05-27 ✅

- [x] **Milestone 1.9:** Workspace init meta-prompt + integracja data/ z TechLead
  - Opis: nowy meta-meta-prompt `copilot_workspace_init.md` jako KROK 1 dla świeżego workspace nadrzędnego. Skanuje folder, interactive whitelist sub-repo, generuje `.code-workspace` + `.copilot-workspace-config.yml` + `data/tech-radar.md` (allowed/avoided tech) + `data/architecture-decisions.md` (ekosystem ADR). Audytuje per sub-repo (CTO v3/v4/brak), raport z listą komend. TechLead zaktualizowany: czyta whitelist (nie skanuje folderów spoza scope), czyta tech-radar (nie proponuje HOLD/RETIRED tech), czyta ADR (flaguje konflikty propozycji z istniejącymi decyzjami).
  - Deadline: 2026-05-27 ✅

- [ ] **Milestone 2:** Pierwszy realny test v3 vs v4 na serwisie GFT (porównanie)
  - Opis: wybrać 1 serwis, najpierw uruchomić v3 (lub upgrade do v3), zrobić feature lub fix. Potem upgrade do v4 (`upgrade_v3_to_v4.md`), zrobić podobny feature/fix. Porównać subiektywnie:
    - Liczba kliknięć / interakcji per task
    - Jakość final reportu
    - Czy CTO faktycznie wykonuje wszystkie fazy które v3 deleguje
    - Polski język i symulator perspektyw — czy realnie pomagają, czy są bloat
    - Czy subagent invocation v3 zwracał "failed" — czy v4 omija ten problem
  - Deadline: do końca czerwca 2026

- [ ] **Milestone 3:** v3.1 po lekcjach z 2-3 serwisów
  - Opis: zaktualizować v3 + upgrade v2→v3 na podstawie realnych użyć. Możliwe kierunki: dopracowanie body Architecta (kiedy delegować vs nie), uproszczenie `full-feature-loop`, usunięcie helperów które nie są używane (np. Debugger jeśli rzadko triggerowany).
  - Deadline: ~lipiec 2026

- [ ] **Milestone 4:** Agent Skills (preview) gdy GFT zezwoli
  - Opis: dodać `.github/skills/` (Agent Skills) jako alternatywę dla wieloetapowych workflows — wymaga policy "Editor preview features" włączonej. Tylko gdy v3 sprawdzony i policy dostępna.
  - Deadline: TBD

- [ ] **Milestone 5 (opcjonalny):** MCP server zamiast wklejania prompta
  - Opis: rozważyć przepisanie prompta na MCP server żeby był single source of truth, aktualizowany centralnie.
  - Deadline: TBD

---

## Zadania

### Do zrobienia

- [ ] Wybrać pierwszy serwis GFT do testu v2 (najprostszy, niska stawka). Jeśli ma już v1 → użyć upgrade prompt; jeśli świeży → bootstrap v2.
- [ ] Uruchomić odpowiedni prompt, zapisać czas + wynik
- [ ] Specjalnie zmierzyć: czy `handoffs` button (Coder → Reviewer) faktycznie się pojawia w UI i czy działa
- [ ] Po użyciu: zebrać 5 najmocniejszych obserwacji (co dobre, co do poprawy)
- [ ] Zaktualizować DECYZJE.md o lekcje
- [ ] Push v2 + upgrade prompt do public repo na GitHubie
- [ ] Zaproponować zmiany w v2 → v2.1

### W trakcie

(puste)

### Zakończone

- [x] 2026-05-27: Workspace init meta-prompt — interactive whitelist sub-repo, generuje .code-workspace + .copilot-workspace-config.yml + data/ skeleton + raport per sub-repo. TechLead updated o czytanie whitelist + tech-radar + ADR.
- [x] 2026-05-27: Multi-repo TechLead — workspace-level Solution Architect dla VS Code z multi-root. Mapowanie, audit, cross-service refactor planning. Chain do per-repo CTO. Polski runtime, angielskie artifakty.
- [x] 2026-05-27: v4 bootstrap prompt — single CTO agent (Werner Vogels-style, polski, symulator perspektyw, Protokół Zero, Zero Halucynacji, Weryfikacja Sędziego)
- [x] 2026-05-27: upgrade prompt v3→v4 — kasuje 4 helpery, dodaje 1 CTO, aktualizuje narrative, zachowuje lessons/instructions/prompts (w tym full-feature-loop jako opcję)
- [x] 2026-05-08: v3 bootstrap prompt — JetBrains official-safe (1 main + 3 helpers, max 1 poziom delegacji)
- [x] 2026-05-08: upgrade prompt v2→v3 — dual-mode, przebudowuje agent files, dodaje full-feature-loop.prompt.md
- [x] 2026-05-08: Deep Research raport — `research/dr_report_2026-05-08_jetbrains_agents.md` (cytaty GitHub Docs, mapa "co działa / co nie")
- [x] 2026-05-08: autonomous_subagent_workflow.md (gotowe prompty subagent loop, wariant Plan B działa nawet gdy plugin ma bug)
- [x] 2026-05-07: v2 bootstrap prompt — Custom Agents + handoffs + nested AGENTS.md guidance (deprecated dla JetBrains po DR z 05-08, zostaje dla VS Code)
- [x] 2026-05-07: upgrade prompt v1→v2 — dual-mode, additive migration, preserve copilot-lessons.md
- [x] 2026-05-07: research zmian w plugin 1.6.x (changelogi 11/2025, 03/2026, 04/2026, docs custom-agents)
- [x] Iteracja v1 prompta (4 rundy refinementu z @cto: support matrix, no-overengineering, JetBrains-specific corrections, self-improvement loop)
- [x] Utworzenie projektu copilot-support
- [x] Materializacja v1 do pliku
- [x] Wypchnięcie v1 do public repo na GitHubie

---

## Notatki

### Plugin 1.5.65 (v1 prompt — `copilot_jetbrains_bootstrap.md`)
- Wspiera: `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` (z `applyTo`), `.github/git-commit-instructions.md`, `.github/prompts/*.prompt.md`, `AGENTS.md`, `CLAUDE.md`, hooks (preview), MCP servers.
- `/` w czacie = TYLKO wbudowane komendy. Custom prompty: dropdown lub `#prompt:nazwa`.
- Frontmatter w prompt files: tylko `description:`. NIE `model:`/`mode:`/`tools:` (to VS Code).
- Self-improvement: `.github/copilot-lessons.md` referencjonowany z głównych instrukcji.

### Plugin 1.6.x (v2 prompt — `copilot_jetbrains_bootstrap_v2.md`)
Wszystko z 1.5.65 NADAL działa. Plus:
- **Custom Agents (GA)**: `.github/agents/<name>.agent.md`. Frontmatter `name`, `description` (req), `tools`, `model`, `mcp-servers`, `target`, `handoffs`. **`handoffs` field** = lista przejść do innych agentów (`label`/`agent`/`prompt`/`send`) — to jest mechanizm advisor architecture.
- W agent files (vs prompt files): `model:` i `tools:` ARE supported.
- AGENTS.md / CLAUDE.md: native UI w Customizations, z generatorem "Generate Agent Instructions". **Nested supported** (np. `src/integrations/AGENTS.md`).
- Plan agent + sub-agents (GA).
- `/memory` slash → otwiera panel agent instructions.
- Inline agent (preview): Shift+Cmd+I.
- MCP auto-approve per server/tool: Settings → Chat → MCP Server and Tool Auto-approve.
- Global auto-approve + granular ("commands not covered by rules", "edits not covered by rules").
- Edit mode w chat → deprecated.

### Architektura advisor (4-agent, w v2)
- **Coder** (Sonnet 4.6, default) — pisze kod, handoff do Reviewer przed "done".
- **Reviewer** (Opus 4.6) — review diffu, handoff back do Coder z fixami albo OK.
- **Architect** (Opus 4.6) — multi-step planning, NIGDY nie pisze kodu, handoff do Coder z planem.
- **Debugger** (Opus 4.6) — hipotezy z evidence, handoff do Coder po user pick.
