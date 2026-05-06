# PLAN PROJEKTU

## Cel główny

Wycisnąć z GitHub Copilota w JetBrains (środowisko GFT) maksimum: dopasowanie do konwencji każdego serwisu, agentowa praca z self-improvement, brak overengineering, low friction codzienna współpraca.

---

## Kamienie milowe

- [x] **Milestone 1:** v1 bootstrap prompta
  - Opis: 6 faz + self-improvement + no-overengineering + zaktualizowane do JetBrains plugin 1.5.65 (instructions overlays, prompts, lessons file)
  - Deadline: 2026-05-06 ✅

- [ ] **Milestone 2:** Pierwszy realny test na serwisie GFT
  - Opis: uruchomić prompt na 1 serwisie, zmierzyć trafność, zebrać korekty
  - Deadline: do końca maja 2026

- [ ] **Milestone 3:** v2 prompta po lekcjach z 2-3 serwisów
  - Opis: zaktualizować prompt na podstawie tego co wygenerował bzdury / co pominął
  - Deadline: ~lipiec 2026

- [ ] **Milestone 4 (opcjonalny):** MCP server zamiast wklejania prompta
  - Opis: rozważyć przepisanie prompta na MCP server żeby był single source of truth, aktualizowany centralnie
  - Deadline: TBD

---

## Zadania

### Do zrobienia

- [ ] Wybrać pierwszy serwis GFT do testu (najprostszy, niska stawka)
- [ ] Uruchomić bootstrap prompt na nim, zapisać czas + wynik
- [ ] Po użyciu: zebrać 5 najmocniejszych obserwacji (co dobre, co do poprawy)
- [ ] Zaktualizować DECYZJE.md o lekcje
- [ ] Zaproponować zmiany w prompcie → v1.1

### W trakcie

(puste)

### Zakończone

- [x] Iteracja v1 prompta (4 rundy refinementu z @cto: support matrix, no-overengineering, JetBrains-specific corrections, self-improvement loop)
- [x] Utworzenie projektu copilot-support
- [x] Materializacja v1 do pliku
- [x] Wypchnięcie do public repo na GitHubie

---

## Notatki

- Plugin JetBrains 1.5.65+ wspiera: `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` (z `applyTo`), `.github/git-commit-instructions.md`, `.github/prompts/*.prompt.md`, `AGENTS.md`, `CLAUDE.md`, hooks (preview), MCP servers.
- W JetBrains `/` w czacie pokazuje TYLKO wbudowane komendy. Custom prompty wybiera się z dropdownu lub przez `#prompt:nazwa` (to feature VS Code którego JetBrains nie ma).
- Frontmatter w prompt files w JetBrains: tylko `description:`. NIE: `model:`, `mode:`, `tools:` (to VS Code).
- Self-improvement działa przez `.github/copilot-lessons.md` referencjonowany z głównych instrukcji.
