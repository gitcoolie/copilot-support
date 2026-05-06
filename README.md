# copilot-support

Bootstrap prompt + materiały do konfiguracji GitHub Copilot (plugin JetBrains 1.5.65+) w mikroserwisach. Jeden prompt → pełna analiza repo + materializacja konfiguracji Copilota dopasowanej do konkretnego serwisu.

## Co jest w tym folderze

- **START.md** — quick reference (cel, status, jak użyć)
- **PLAN.md** — kamienie milowe i zadania
- **DECYZJE.md** — log decyzji projektowych
- **copilot_jetbrains_bootstrap.md** — **GŁÓWNY ARTEFAKT**: kompletny prompt do wklejenia w Copilot Chat (Agent mode, Opus 4.6) w danym serwisie. Generuje całą konfigurację `.github/` + `AGENTS.md` + `CLAUDE.md` + `.github/copilot-lessons.md`.

## Jak użyć

1. Otwórz docelowy serwis w JetBrains IDE.
2. Otwórz Copilot Chat → tryb Agent → model Claude Opus 4.6.
3. Wklej całą zawartość `copilot_jetbrains_bootstrap.md`.
4. Copilot przejdzie przez 6 faz: Discovery → Synthesis → Materialize → Verify → Tailored Extensions → Registration Guide.
5. Zatwierdzaj checkpointy (po Phase 2 i przy proposed extensions w Phase 5).
6. Po zakończeniu zarejestruj pliki w Settings → Tools → GitHub Copilot → Customizations.

## Jak aktualizować

- Każda lekcja z używania prompta w realnych serwisach → DECYZJE.md.
- Ulepszenia samego prompta → edytuj `copilot_jetbrains_bootstrap.md`, zaznacz changelog na górze pliku.
- Status projektu → DATA/projekty-status.md.

## Powiązani asystenci

- **@cto** — utrzymanie i ewolucja prompta, dopasowanie do nowych wersji pluginu.
- **@coo** — gdy bootstrap prompt rozwija się w pełnoprawny system (np. CLI, MCP server).
