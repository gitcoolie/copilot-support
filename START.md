# START - Quick Reference

**Nazwa projektu:** copilot-support

**Cel:** Maksymalne wykorzystanie GitHub Copilot w środowisku JetBrains klienta (GFT) — stworzyć i utrzymywać jeden bootstrap prompt który po jednorazowym uruchomieniu w danym serwisie konfiguruje Copilota tak, żeby był maksymalnie inteligentny, kontekstowy, i nie wkurwiał (nie overengineerował, dopasowywał się do konwencji repo, uczył się z korekt).

**Status:** AKTYWNY — v3 (stable) i v4 (EXPERIMENTAL single-CTO) działają równolegle. Do testów na realnych serwisach. v2 zachowany dla VS Code, v1 dla pluginu 1.5.65.

**Data utworzenia:** 2026-05-06
**Ostatnia istotna zmiana:** 2026-05-27 — dodane v4 + upgrade v3→v4. v4 to alternatywa architektoniczna: jeden uniwersalny agent CTO (Werner Vogels-style, polski, symulator perspektyw, Protokół Zero, Zero Halucynacji, Weryfikacja Sędziego) zamiast 4 wyspecjalizowanych helperów z v3. Inspirowane `TEAM/CTO.md`. Do testowania równolegle z v3 — wybór architektury per Twoja preferencja.

**Kluczowe pliki:**
- `copilot_vscode_multirepo_techlead.md` — **MULTI-REPO** TechLead dla VS Code workspace nadrzędnego (cross-service planning, audit, mapowanie). Komplementarne do v4.
- `copilot_jetbrains_bootstrap_v4.md` — **EXPERIMENTAL** single-CTO per-project (IntelliJ).
- `copilot_jetbrains_bootstrap_v3.md` — **STABLE** 4-agent advisor per-project (IntelliJ).
- `copilot_jetbrains_upgrade_v3_to_v4.md` — przejście v3 → v4 (kasuje 4 helpery, dodaje 1 CTO). Dual-mode.
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

1. [ ] Wybrać pierwszy serwis GFT do testu v4 (najprostszy, niska stawka)
2. [ ] Uruchomić `upgrade_v3_to_v4.md` na serwisie który jest na v3 (najlepiej ten sam co już testowany — porównanie)
3. [ ] Specjalnie zmierzyć vs v3:
   - Czy CTO pojawia się w agents dropdown jako jedyny
   - Czy Protokół Zero faktycznie się odpala (widać że czyta pliki)
   - Czy AUTO-BRIEF jest po polsku, z 4 sekcjami
   - Czy CTO sam decyduje o fazach (plan/impl/debug/review) czy gubi się
   - Symulator perspektyw — czy faktycznie pokazuje >1 perspektywę czy pomija
   - Weryfikacja Sędziego przed claim "done" — czy ją robi
4. [ ] Porównanie subiektywne v3 vs v4 po 3 użyciach: który workflow szybszy / przyjemniejszy / dokładniejszy
5. [ ] Po wyborze faworyta: zostawić oba, oznaczyć faworyt jako "REKOMENDOWANE" w README
