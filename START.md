# START - Quick Reference

**Nazwa projektu:** copilot-support

**Cel:** Maksymalne wykorzystanie GitHub Copilot w środowisku JetBrains klienta (GFT) — stworzyć i utrzymywać jeden bootstrap prompt który po jednorazowym uruchomieniu w danym serwisie konfiguruje Copilota tak, żeby był maksymalnie inteligentny, kontekstowy, i nie wkurwiał (nie overengineerował, dopasowywał się do konwencji repo, uczył się z korekt).

**Status:** AKTYWNY — v3 prompt + upgrade v2→v3 gotowe (2026-05-08), do testów na realnych serwisach. v2 zachowany dla VS Code, v1 dla pluginu 1.5.65.

**Data utworzenia:** 2026-05-06
**Ostatnia istotna zmiana:** 2026-05-08 — Deep Research wykazał że v2 architektura (handoffs + per-agent model) jest VS Code-style, nieportable do JetBrains 1.6.x. Dodane v3 (official-safe) + upgrade v2→v3 + raport DR w `research/`.

**Kluczowe pliki:**
- `copilot_jetbrains_bootstrap_v3.md` — **GŁÓWNY ARTEFAKT** dla świeżych serwisów na pluginie JetBrains 1.6.1+. Tylko oficjalnie udokumentowane pola, 1 main + 3 helpers, max 1 poziom delegacji. Wieloetapowe workflows przez `full-feature-loop.prompt.md`.
- `copilot_jetbrains_upgrade_v2_to_v3.md` — upgrade dla serwisów na v2. Dual-mode, zachowuje lessons/instructions/prompts.
- `copilot_jetbrains_bootstrap_v2.md` — v2 (VS Code-style), z `handoffs:` i per-agent `model:`. **Deprecated w JetBrains**, zostaje dla VS Code i historii decyzji.
- `copilot_jetbrains_upgrade_v1_to_v2.md` — upgrade v1 → v2 (potrzebny przed v2→v3 jeśli serwis stoi na v1).
- `copilot_jetbrains_bootstrap.md` — v1, dla pluginu 1.5.65.
- `autonomous_subagent_workflow.md` — gotowe prompty workflowowe (część założeń v2-era, w v3 zastąpione przez `full-feature-loop.prompt.md`).
- `research/dr_report_2026-05-08_jetbrains_agents.md` — Deep Research raport: dlaczego v2 architektura nie działa w JetBrains, co realnie udokumentowane, cytaty z GitHub Docs.
- `README.md` — instrukcja użycia + tabela "który prompt wybrać"
- `DECYZJE.md` — log iteracji prompta i lekcji

---

## Szybki kontekst

Pracuję w GFT na środowisku klienta. Mam Copilota z Claude Sonnet 4.6, Opus 4.6, GPT-5.3-Codex (plugin JetBrains **1.6.1-243** od 2026-05-07). Architektura: dużo serwisów gadających po REST i Kafka. Bez konfiguracji Copilot jest generic i overengineeruje. Z tym promptem dla każdego serwisu generuję `.github/copilot-instructions.md` + `instructions/` overlays + `prompts/` workflows (w tym `full-feature-loop.prompt.md`) + `AGENTS.md` + `copilot-lessons.md` (self-improvement) + **`.github/agents/` (Architect entry-point + Coder/Reviewer/Debugger helpers, official-safe v3 format)**.

**Architektura v3 (od 2026-05-08, po DR):** Architect (z dropdown) deleguje JEDNĄ rzecz w dół do helpera (Coder/Reviewer/Debugger) przez tool `agent`. Max 1 poziom delegacji. Model jednolity per sesja (subagenty dziedziczą model sesji w JetBrains). Wieloetapowe workflows (plan → implement → debug → review) idą przez `#prompt:full-feature-loop` z explicit user checkpoints zamiast subagent chains.

**Co odpadło z v2 (po DR):** `handoffs:` (nieudokumentowane dla JetBrains, niestabilne), per-agent `model:` (ignorowany w subagencie + bug w 1.6.1), advisor pattern przez chain Coder→Reviewer w jednej sesji (max 1 level delegacji). Wszystko w `research/dr_report_2026-05-08_jetbrains_agents.md`.

---

## Zespół

- **Owner:** Michał (CTO @ GFT)
- **Wspiera:** @cto (utrzymanie prompta), @coo (gdy projekt rośnie)

---

## Next Steps

1. [ ] Wybrać pierwszy serwis GFT do testu v3 (najprostszy, niska stawka)
2. [ ] Uruchomić odpowiedni prompt: świeży serwis → `bootstrap_v3.md`; serwis z v2 → `upgrade_v2_to_v3.md`; serwis z v1 → najpierw `upgrade_v1_to_v2.md`, potem `upgrade_v2_to_v3.md`
3. [ ] Specjalnie zmierzyć:
   - Czy Architect pojawia się w agents dropdown (visible: user-invocable=true)
   - Czy Architect → Coder delegation przez `agent` tool zwraca "finished" zamiast "failed"
   - Jak działa `#prompt:full-feature-loop` w realnym wieloetapowym ticktecie
4. [ ] Po 2-3 użyciach: zebrać lekcje → iterować v3.1 lub potencjalnie v4 jeśli rdzeń wymaga rewritu
5. [ ] Push v3 + upgrade v2→v3 + DR raport do public repo na GitHubie
