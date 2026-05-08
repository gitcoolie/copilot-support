# copilot-support

Bootstrap + upgrade prompty do konfiguracji GitHub Copilot w mikroserwisach (plugin JetBrains). Jeden prompt → pełna analiza repo + materializacja konfiguracji Copilota dopasowanej do konkretnego serwisu, z architekturą advisor (Coder→Reviewer handoff).

## Co jest w tym folderze

- **START.md** — quick reference (cel, status, jak użyć)
- **PLAN.md** — kamienie milowe i zadania
- **DECYZJE.md** — log decyzji projektowych
- **copilot_jetbrains_bootstrap.md** — **v1** (plugin 1.5.65). Bez Custom Agents. Zostaje dla serwisów na starym pluginie.
- **copilot_jetbrains_bootstrap_v2.md** — **v2** (plugin 1.6.1+, VS Code-style). Custom Agents z `handoffs:` i per-agent `model:`. **DEPRECATED w JetBrains** (DR z 2026-05-08 wykazał że `handoffs:` i `model:` są nieportable / ignored / buggy w JetBrains 1.6.x — patrz `research/dr_report_2026-05-08_jetbrains_agents.md`). Zostaje dla VS Code i jako historia decyzji.
- **copilot_jetbrains_bootstrap_v3.md** — **v3** (plugin 1.6.1+, JetBrains official-safe). **REKOMENDOWANE** dla nowych konfiguracji w JetBrains. Tylko oficjalnie udokumentowane pola (`user-invocable`, `disable-model-invocation`, bez `handoffs:`/`model:`). Architektura: 1 main agent (Architect) + 3 helpers, max 1 poziom delegacji, model jednolity per sesja. Wieloetapowe workflows przez nowy `.github/prompts/full-feature-loop.prompt.md`.
- **copilot_jetbrains_upgrade_v1_to_v2.md** — upgrade v1 → v2. Dual-mode.
- **copilot_jetbrains_upgrade_v2_to_v3.md** — upgrade v2 → v3. Przebudowuje agent files (usuwa handoffs/model, dodaje user-invocable/disable-model-invocation), aktualizuje narrative, dokłada `full-feature-loop.prompt.md`. Zachowuje `copilot-lessons.md`, `instructions/`, `prompts/`. Dual-mode.
- **autonomous_subagent_workflow.md** — gotowe prompty do uruchamiania pętli subagentów. **UWAGA:** v2-era doc, część założeń (subagent chain, model per agent w jednej sesji) NIE działa w JetBrains 1.6.x. W praktyce stosuj wariant Plan B (manualne sesje) lub `#prompt:full-feature-loop` z v3.
- **research/dr_report_2026-05-08_jetbrains_agents.md** — Deep Research raport: co realnie działa w JetBrains 1.6.x, co NIE, dlaczego v3 odeszło od v2 architektury. Cytaty z oficjalnych GitHub Docs.

## Który prompt wybrać

| Sytuacja | Prompt |
|---|---|
| Świeży serwis, plugin JetBrains 1.6.1+ | `copilot_jetbrains_bootstrap_v3.md` ⭐ |
| Świeży serwis, VS Code (nie JetBrains) | `copilot_jetbrains_bootstrap_v2.md` (handoffs działają w VS Code) |
| Świeży serwis, plugin 1.5.65 | `copilot_jetbrains_bootstrap.md` (v1) |
| Serwis MA config z v2, plugin JetBrains 1.6.1+ | `copilot_jetbrains_upgrade_v2_to_v3.md` ⭐ |
| Serwis MA config z v1, plugin podbity do 1.6.1+ | najpierw `copilot_jetbrains_upgrade_v1_to_v2.md`, potem `copilot_jetbrains_upgrade_v2_to_v3.md` |
| Serwis MA config z v1, plugin ZOSTAŁ na 1.5.65 | nie ruszaj — v1 dalej działa |

## Jak użyć (v3 / świeży serwis JetBrains 1.6.1+)

1. Otwórz docelowy serwis w JetBrains IDE.
2. Settings → Tools → GitHub Copilot → Chat → włącz: Agent mode, Custom Agent, Subagent. Ustaw Agent Max Requests ≥ 50.
3. Otwórz Copilot Chat → tryb Agent → model Claude Opus 4.6.
4. Wklej całą zawartość `copilot_jetbrains_bootstrap_v3.md`.
5. 6 faz: Discovery → Synthesis → Materialize → Verify → Tailored Extensions → Registration Guide. Zatwierdzaj checkpointy.
6. Po zakończeniu: pełny restart IDE → Customizations weryfikacja. Smoke test: agents dropdown → Architect → mała zmiana → on deleguje raz do Coder.

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
