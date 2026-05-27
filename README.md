# copilot-support

Bootstrap + upgrade prompty do konfiguracji GitHub Copilot w mikroserwisach (plugin JetBrains). Jeden prompt → pełna analiza repo + materializacja konfiguracji Copilota dopasowanej do konkretnego serwisu, z architekturą advisor (Coder→Reviewer handoff).

## Co jest w tym folderze

- **START.md** — quick reference (cel, status, jak użyć)
- **PLAN.md** — kamienie milowe i zadania
- **DECYZJE.md** — log decyzji projektowych
- **copilot_jetbrains_bootstrap.md** — **v1** (plugin 1.5.65). Bez Custom Agents. Zostaje dla serwisów na starym pluginie.
- **copilot_jetbrains_bootstrap_v2.md** — **v2** (plugin 1.6.1+, VS Code-style). Custom Agents z `handoffs:` i per-agent `model:`. **DEPRECATED w JetBrains** (DR z 2026-05-08 wykazał że `handoffs:` i `model:` są nieportable / ignored / buggy w JetBrains 1.6.x — patrz `research/dr_report_2026-05-08_jetbrains_agents.md`). Zostaje dla VS Code i jako historia decyzji.
- **copilot_jetbrains_bootstrap_v3.md** — **v3** (plugin 1.6.1+, JetBrains official-safe). 4 agenty (Architect/Coder/Reviewer/Debugger), 1 main + 3 helpers, max 1 poziom delegacji. Wieloetapowe workflows przez `.github/prompts/full-feature-loop.prompt.md`. Stabilne podejście.
- **copilot_jetbrains_bootstrap_v4.md** — **v4.1 EXPERIMENTAL** (plugin 1.6.1+). Jeden uniwersalny agent **CTO** (Werner Vogels-style) który robi wszystko sam: plan/impl/debug/review w jednej sesji. Polski język, symulator perspektyw, Protokół Zero, Zero Halucynacji, Weryfikacja Sędziego. Inspirowane `TEAM/CTO.md` Michała. **v4.1 nowe:** generuje `data/structure-map.md` + `data/api-inventory.md` + `data/conventions.md` + `data/known-issues.md` jako trwały kontekst (~3k tokens, czytane przy Protokole Zero zamiast re-discovery). Plus `refresh-context.prompt.md` do utrzymania spójności + Consistency Spot-Check w CTO.
- **copilot_jetbrains_upgrade_v1_to_v2.md** — upgrade v1 → v2. Dual-mode.
- **copilot_jetbrains_upgrade_v2_to_v3.md** — upgrade v2 → v3. Przebudowuje agent files (usuwa handoffs/model, dodaje user-invocable/disable-model-invocation), aktualizuje narrative, dokłada `full-feature-loop.prompt.md`. Zachowuje `copilot-lessons.md`, `instructions/`, `prompts/`. Dual-mode.
- **copilot_jetbrains_upgrade_v3_to_v4.md** — upgrade v3 → v4. Kasuje 4 helpery v3, tworzy 1 CTO agent, aktualizuje narrative. Zachowuje lessons / instructions / prompts (w tym full-feature-loop jako opcję alternatywną). Dual-mode.
- **copilot_jetbrains_upgrade_v4_to_v4_1.md** — upgrade v4 → v4.1 dla serwisów które już mają v4 bez `data/` files. Targeted discovery + generuje 4 pliki data/ + refresh-context prompt + updates CTO body o Protokół Zero rozszerzony i Consistency Spot-Check. Preserves wszystko inne. Dual-mode.
- **copilot_workspace_init.md** — **meta-meta-prompt**: jednorazowy setup workspace nadrzędnego ZANIM uruchomisz TechLead. Skanuje folder, interactive whitelist sub-repo, generuje `.code-workspace` + `.copilot-workspace-config.yml` + `data/tech-radar.md` + `data/architecture-decisions.md`, audyt per sub-repo (które mają CTO v3/v4/brak), raport z listą następnych komend.
- **copilot_vscode_multirepo_techlead.md** — **dla VS Code workspace nadrzędnego** z multi-root (kilka repo GFT jednocześnie). Wymaga uprzedniej inicjalizacji przez `copilot_workspace_init.md`. Generuje workspace-level `TechLead` (Solution Architect): mapuje serwisy z whitelist, audytuje cross-cutting, planuje cross-service refactor. Honoruje `data/tech-radar.md` (nie proponuje HOLD/RETIRED tech) i `data/architecture-decisions.md` (nie łamie istniejących ADR). Komunikacja po polsku, wszystkie artifakty pisane po angielsku (shareable z międzynarodowym zespołem). Chain: TechLead → CTO per-repo (handoff). Komplementarne do v4 — TechLead big picture, CTO per-project deep work.
- **autonomous_subagent_workflow.md** — gotowe prompty do uruchamiania pętli subagentów. **UWAGA:** v2-era doc, część założeń (subagent chain, model per agent w jednej sesji) NIE działa w JetBrains 1.6.x. W praktyce stosuj wariant Plan B (manualne sesje) lub `#prompt:full-feature-loop` z v3.
- **research/dr_report_2026-05-08_jetbrains_agents.md** — Deep Research raport: co realnie działa w JetBrains 1.6.x, co NIE, dlaczego v3 odeszło od v2 architektury. Cytaty z oficjalnych GitHub Docs.

## Który prompt wybrać

**Per-project (IntelliJ, jedno repo):**

| Sytuacja | Prompt |
|---|---|
| Świeży serwis, JetBrains 1.6.1+, single-CTO architecture | `copilot_jetbrains_bootstrap_v4.md` 🧪 |
| Świeży serwis, JetBrains 1.6.1+, stabilny 4-agent advisor | `copilot_jetbrains_bootstrap_v3.md` |
| Świeży serwis, VS Code (per-repo) | `copilot_jetbrains_bootstrap_v2.md` (handoffs działają w VS Code) |
| Świeży serwis, plugin 1.5.65 | `copilot_jetbrains_bootstrap.md` (v1) |
| Serwis MA config z v3, chcę przejść na single-CTO | `copilot_jetbrains_upgrade_v3_to_v4.md` 🧪 |
| Serwis MA config z v4 (bez data/), chcę dorobić trwały kontekst | `copilot_jetbrains_upgrade_v4_to_v4_1.md` ⭐ |
| Serwis MA config z v2, chcę v3 stable | `copilot_jetbrains_upgrade_v2_to_v3.md` |
| Serwis MA config z v2, chcę od razu v4 | najpierw v2→v3, potem v3→v4 |
| Serwis MA config z v1, plugin 1.6.1+ | v1→v2→v3 (opcjonalnie →v4) |
| Serwis MA config z v1, plugin 1.5.65 | nie ruszaj — v1 dalej działa |

**Multi-repo (VS Code workspace nadrzędny, kilka serwisów jednocześnie):**

| Sytuacja | Prompt |
|---|---|
| Świeży folder nadrzędny — najpierw setup workspace (whitelist sub-repo + skeleton data/) | `copilot_workspace_init.md` ⭐ (uruchom PIERWSZE) |
| Po inicjalizacji — generowanie agenta TechLead | `copilot_vscode_multirepo_techlead.md` ⭐ (uruchom DRUGIE) |

Multi-repo i per-project są KOMPLEMENTARNE: w IntelliJ pracujesz deep w jednym repo z CTO, w VS Code planujesz cross-service zmiany z TechLead. TechLead może handoff do CTO konkretnego sub-repo.

**Kolejność dla świeżego workspace nadrzędnego:**
1. `copilot_workspace_init.md` → wybierasz whitelist sub-repo, generuje config + data templates
2. (opcjonalnie) wypełnij `data/tech-radar.md` i `data/architecture-decisions.md`
3. Dla sub-repo bez CTO: `copilot_jetbrains_bootstrap_v4.md` w IntelliJ (per repo, ~10 min każde)
4. `copilot_vscode_multirepo_techlead.md` → generuje TechLead w workspace
5. Reload VS Code → gotowe

## Jak użyć (v4 EXPERIMENTAL / świeży serwis JetBrains 1.6.1+, single CTO)

1. Otwórz serwis w JetBrains IDE.
2. Settings → Tools → GitHub Copilot → Chat → włącz: Agent mode, Custom Agent. Ustaw Agent Max Requests ≥ 50.
3. Copilot Chat → Agent mode → Claude Opus 4.6.
4. Wklej `copilot_jetbrains_bootstrap_v4.md`.
5. 6 faz. Zatwierdzaj checkpointy.
6. Pełny restart IDE → smoke test: agents dropdown → **CTO** → "Daj mi AUTO-BRIEF tego serwisu" → powinien odpowiedzieć po polsku z 4 sekcjami.

## Jak użyć (v3 / świeży serwis JetBrains 1.6.1+, 4-agent stable)

1. Otwórz serwis w JetBrains IDE.
2. Settings → Tools → GitHub Copilot → Chat → włącz: Agent mode, Custom Agent, Subagent. Ustaw Agent Max Requests ≥ 50.
3. Copilot Chat → Agent mode → Claude Opus 4.6.
4. Wklej `copilot_jetbrains_bootstrap_v3.md`.
5. 6 faz. Zatwierdzaj checkpointy.
6. Pełny restart IDE → smoke test: agents dropdown → Architect → mała zmiana → deleguje raz do Coder.

## Jak użyć (upgrade v3 → v4, gdy chcesz przetestować single CTO)

1. Otwórz serwis na v3.
2. Wklej `copilot_jetbrains_upgrade_v3_to_v4.md` w Copilot Chat (Agent mode, Opus 4.6).
3. Wybierz tryb: `1` interactive lub `2` autonomous.
4. Po zakończeniu: pełny restart IDE.

## Jak użyć (multi-repo workspace — kolejność)

**Krok 1: workspace init (jednorazowo)**
1. Otwórz folder nadrzędny w VS Code (np. `~/work/gft/`).
2. Copilot Chat → Agent mode → Opus 4.6.
3. Wklej `copilot_workspace_init.md`.
4. Interactive: wybierz które sub-foldery to GFT scope (whitelist).
5. Generuje: `.code-workspace`, `.copilot-workspace-config.yml`, `data/tech-radar.md`, `data/architecture-decisions.md`.
6. Czytasz raport — które sub-repo wymagają CTO bootstrap.

**Krok 2: bootstrap per-repo (dla sub-repo bez CTO)**
- IntelliJ → otwórz sub-repo → wklej `copilot_jetbrains_bootstrap_v4.md`.
- Powtórz dla każdego brakującego.

**Krok 3: generowanie TechLead**
1. Wróć do VS Code workspace.
2. Wklej `copilot_vscode_multirepo_techlead.md` (wymaga że krok 1 jest zrobiony — sprawdza `.copilot-workspace-config.yml`).
3. 6 faz: Discovery (z whitelist) → Synthesis → Materialize → Verify → Tailored Extensions → Registration Guide.
4. Reload VS Code window → smoke test: agents dropdown → **TechLead** → "Daj mi mapę serwisów".
5. Test handoff: TechLead → "drill into service-X" → CTO konkretnego sub-repo.

## Jak użyć (upgrade v2 → v3, gdy serwis ma już v2)

1. Otwórz serwis który ma `.github/agents/*.agent.md` z `handoffs:` i `model:` (sygnatura v2).
2. Settings → Tools → GitHub Copilot → Chat → upewnij się że Custom Agent + Subagent są ON.
3. Wklej `copilot_jetbrains_upgrade_v2_to_v3.md` w Copilot Chat (Agent mode, Opus 4.6).
4. Wybierz tryb: `1` interactive (krok-po-kroku) lub `2` autonomous (jeden checkpoint).
5. Po zakończeniu: pełny restart IDE, smoke test jak wyżej.

## Jak użyć (upgrade v1 → v2 → v3)

Gdy serwis stoi na v1 (1.5.65) a plugin podbiłeś do 1.6.1+:
1. Najpierw `copilot_jetbrains_upgrade_v1_to_v2.md` (dodaje custom agents w v2 formacie).
2. Potem `copilot_jetbrains_upgrade_v2_to_v3.md` (przerabia na v3 official-safe).

## Jak aktualizować

- Każda lekcja z używania prompta w realnych serwisach → DECYZJE.md.
- Ulepszenia v3 → edytuj `copilot_jetbrains_bootstrap_v3.md`, zaznacz changelog na górze pliku.
- Ulepszenia upgrade promptów → edytuj odpowiedni plik upgrade.
- Większe rewrite (v3 → v4) → osobny DR + osobny raport w `research/`.
- Status projektu → DATA/projekty-status.md.

## Powiązani asystenci

- **@cto** — utrzymanie i ewolucja prompta, dopasowanie do nowych wersji pluginu.
- **@coo** — gdy bootstrap prompt rozwija się w pełnoprawny system (np. CLI, MCP server).
