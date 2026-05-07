# START - Quick Reference

**Nazwa projektu:** copilot-support

**Cel:** Maksymalne wykorzystanie GitHub Copilot w środowisku JetBrains klienta (GFT) — stworzyć i utrzymywać jeden bootstrap prompt który po jednorazowym uruchomieniu w danym serwisie konfiguruje Copilota tak, żeby był maksymalnie inteligentny, kontekstowy, i nie wkurwiał (nie overengineerował, dopasowywał się do konwencji repo, uczył się z korekt).

**Status:** AKTYWNY — v2 prompt + upgrade v1→v2 gotowe (2026-05-07), do testów na realnych serwisach. v1 zachowany dla pluginu 1.5.65.

**Data utworzenia:** 2026-05-06
**Ostatnia istotna zmiana:** 2026-05-07 — plugin GFT podbity do 1.6.1-243, dodane v2 + upgrade prompt z architekturą advisor (4-agent: architect/coder/reviewer/debugger).

**Kluczowe pliki:**
- `copilot_jetbrains_bootstrap_v2.md` — **GŁÓWNY ARTEFAKT** dla świeżych serwisów na pluginie 1.6.1+. Custom Agents + handoffs + nested AGENTS.md.
- `copilot_jetbrains_upgrade_v1_to_v2.md` — upgrade prompt dla serwisów które już mają v1. Dual-mode (interactive / autonomous), additive (zachowuje `.github/copilot-lessons.md`).
- `copilot_jetbrains_bootstrap.md` — v1, dla pluginu 1.5.65. Zostaje dla referencji.
- `README.md` — instrukcja użycia + tabela "który prompt wybrać"
- `DECYZJE.md` — log iteracji prompta i lekcji

---

## Szybki kontekst

Pracuję w GFT na środowisku klienta. Mam Copilota z Claude Sonnet 4.6, Opus 4.6, GPT-5.3-Codex (plugin JetBrains **1.6.1-243** od 2026-05-07). Architektura: dużo serwisów gadających po REST i Kafka. Bez konfiguracji Copilot jest generic i overengineeruje. Z tym promptem dla każdego serwisu generuję `.github/copilot-instructions.md` + `instructions/` overlays + `prompts/` workflows + `AGENTS.md` + `copilot-lessons.md` (self-improvement) + **`.github/agents/` (architect/coder/reviewer/debugger z handoffs — advisor architecture)** + Tailored Extensions dopasowane do realnego kodu.

**Architektura advisor (v2):** Coder (Sonnet, default, tani) implementuje → handoff "Request review" → Reviewer (Opus, drogi) zwraca OK lub fix-list → handoff back do Coder. Cel: jedna sesja = max wartość, minimum dotykania, kontrola jakości przed claim "done".

---

## Zespół

- **Owner:** Michał (CTO @ GFT)
- **Wspiera:** @cto (utrzymanie prompta), @coo (gdy projekt rośnie)

---

## Next Steps

1. [ ] Wybrać pierwszy serwis GFT do testu v2 (najprostszy, niska stawka)
2. [ ] Uruchomić odpowiedni prompt: świeży serwis → `bootstrap_v2.md`; serwis już z v1 → `upgrade_v1_to_v2.md`
3. [ ] Specjalnie zmierzyć: czy `handoffs` button w UI (Coder → Reviewer) faktycznie się pojawia i jak płynnie przechodzi sesja
4. [ ] Po 2-3 użyciach: zebrać lekcje → iterować do v2.1 (a potem v3 z MCP per-agent + hooks)
5. [ ] Push v2 + upgrade prompt do public repo na GitHubie
