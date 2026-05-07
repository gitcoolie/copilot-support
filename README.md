# copilot-support

Bootstrap + upgrade prompty do konfiguracji GitHub Copilot w mikroserwisach (plugin JetBrains). Jeden prompt → pełna analiza repo + materializacja konfiguracji Copilota dopasowanej do konkretnego serwisu, z architekturą advisor (Coder→Reviewer handoff).

## Co jest w tym folderze

- **START.md** — quick reference (cel, status, jak użyć)
- **PLAN.md** — kamienie milowe i zadania
- **DECYZJE.md** — log decyzji projektowych
- **copilot_jetbrains_bootstrap.md** — **v1** (plugin 1.5.65+). Świeży serwis, bez Custom Agents. Zostaje dla referencji i serwisów na starszym pluginie.
- **copilot_jetbrains_bootstrap_v2.md** — **v2** (plugin 1.6.1+). Świeży serwis, Z Custom Agents (architect/coder/reviewer/debugger) + handoffs + nested AGENTS.md guidance. **REKOMENDOWANE** dla nowych konfiguracji.
- **copilot_jetbrains_upgrade_v1_to_v2.md** — **upgrade prompt** dla serwisów które już mają v1 i chcą dostać Custom Agents bez utraty `.github/copilot-lessons.md` ani lokalnych dostosowań. Dual-mode (interactive / autonomous).

## Który prompt wybrać

| Sytuacja | Prompt |
|---|---|
| Świeży serwis, plugin 1.6.1+ | `copilot_jetbrains_bootstrap_v2.md` |
| Świeży serwis, plugin 1.5.65 (1.6 niedostępny) | `copilot_jetbrains_bootstrap.md` (v1) |
| Serwis MA już config z v1, plugin podbity do 1.6.1+ | `copilot_jetbrains_upgrade_v1_to_v2.md` |
| Serwis MA już config z v1, plugin ZOSTAŁ na 1.5.65 | nie ruszaj — v1 dalej działa |

## Jak użyć (v2 / świeży serwis)

1. Otwórz docelowy serwis w JetBrains IDE.
2. Settings → Tools → GitHub Copilot → Chat → włącz: Agent mode, Custom Agents, Plan agent.
3. Otwórz Copilot Chat → tryb Agent → model Claude Opus 4.6.
4. Wklej całą zawartość `copilot_jetbrains_bootstrap_v2.md`.
5. 6 faz: Discovery → Synthesis → Materialize (z Custom Agents) → Verify → Tailored Extensions → Registration Guide. Zatwierdzaj checkpointy.
6. Po zakończeniu: Settings → Tools → GitHub Copilot → Customizations → Reload. Smoke test: agents dropdown → Coder → mała zmiana → handoff "Request review".

## Jak użyć (upgrade v1→v2)

1. Otwórz serwis który ma już `.github/copilot-instructions.md` z v1.
2. Settings → Tools → GitHub Copilot → Chat → włącz Custom Agents + Plan agent.
3. Wklej `copilot_jetbrains_upgrade_v1_to_v2.md` w Copilot Chat (Agent mode, Opus 4.6).
4. Wybierz tryb: `1` interactive (krok-po-kroku) lub `2` autonomous (jeden checkpoint).
5. Po zakończeniu: reload plugin, smoke test jak wyżej.

## Jak aktualizować

- Każda lekcja z używania prompta w realnych serwisach → DECYZJE.md.
- Ulepszenia v2 → edytuj `copilot_jetbrains_bootstrap_v2.md`, zaznacz changelog na górze pliku.
- Ulepszenia upgrade prompt → edytuj `copilot_jetbrains_upgrade_v1_to_v2.md`.
- Status projektu → DATA/projekty-status.md.

## Powiązani asystenci

- **@cto** — utrzymanie i ewolucja prompta, dopasowanie do nowych wersji pluginu.
- **@coo** — gdy bootstrap prompt rozwija się w pełnoprawny system (np. CLI, MCP server).
