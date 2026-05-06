# START - Quick Reference

**Nazwa projektu:** copilot-support

**Cel:** Maksymalne wykorzystanie GitHub Copilot w środowisku JetBrains klienta (GFT) — stworzyć i utrzymywać jeden bootstrap prompt który po jednorazowym uruchomieniu w danym serwisie konfiguruje Copilota tak, żeby był maksymalnie inteligentny, kontekstowy, i nie wkurwiał (nie overengineerował, dopasowywał się do konwencji repo, uczył się z korekt).

**Status:** AKTYWNY — v1 prompta gotowa, do testów na realnych serwisach

**Data utworzenia:** 2026-05-06

**Kluczowe pliki:**
- `copilot_jetbrains_bootstrap.md` — finalny prompt v1 (6 faz + self-improvement + no-overengineering)
- `README.md` — instrukcja użycia
- `DECYZJE.md` — log iteracji prompta i lekcji

---

## Szybki kontekst

Pracuję w GFT na środowisku klienta. Mam Copilota z Claude Sonnet 4.6, Opus 4.6, GPT-5.3-Codex (plugin JetBrains 1.5.65+). Architektura: dużo serwisów gadających po REST i Kafka. Bez konfiguracji Copilot jest generic i overengineeruje. Z tym promptem dla każdego serwisu generuję `.github/copilot-instructions.md` + `instructions/` overlays + `prompts/` workflows + `AGENTS.md` + `copilot-lessons.md` (self-improvement) + Tailored Extensions dopasowane do realnego kodu.

---

## Zespół

- **Owner:** Michał (CTO @ GFT)
- **Wspiera:** @cto (utrzymanie prompta), @coo (gdy projekt rośnie)

---

## Next Steps

1. [ ] Przetestować prompt na 1 realnym serwisie w GFT — zmierzyć ile linii konfiguracji generuje, czy są celne
2. [ ] Po 2-3 użyciach: zebrać lekcje (co dało wartość, co było bloatem) → iterować prompt do v2
3. [ ] Rozważyć: czy MCP server byłby lepszy niż prompt do wklejania (single source of truth zamiast kopii w każdym repo)
