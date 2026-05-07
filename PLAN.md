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

- [ ] **Milestone 2:** Pierwszy realny test na serwisie GFT
  - Opis: wybrać 1 serwis, uruchomić v2 (świeży) lub upgrade (jeśli już ma v1), zmierzyć trafność, zebrać korekty. Specjalna uwaga: czy advisor handoffs (Coder→Reviewer) działa płynnie i redukuje turn-y w sesji.
  - Deadline: do końca maja 2026

- [ ] **Milestone 3:** v2.1 prompta po lekcjach z 2-3 serwisów
  - Opis: zaktualizować v2 (i upgrade prompt) na podstawie tego co wygenerował bzdury / co pominął. Najpewniej: dopracowanie handoff promptów, ewentualne dodatkowe agents (np. migration-author dla Flyway-heavy serwisów).
  - Deadline: ~lipiec 2026

- [ ] **Milestone 4:** v3 — MCP servers per-agent + hooks
  - Opis: dodać `mcp-servers` w frontmatter agentów (np. Architect z Atlassian MCP do czytania ticketów), hooks preToolUse/postToolUse gdy wyjdą z preview. Tylko gdy v2 sprawdzony w 2-3 serwisach.
  - Deadline: TBD

- [ ] **Milestone 5 (opcjonalny):** MCP server zamiast wklejania prompta
  - Opis: rozważyć przepisanie prompta na MCP server żeby był single source of truth, aktualizowany centralnie. Prawdopodobnie przeskakuje milestone 4 i wchodzi razem z MCP per-agent.
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

- [x] 2026-05-07: v2 bootstrap prompt — Custom Agents + handoffs + nested AGENTS.md guidance
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
