# LOG DECYZJI

**Data utworzenia:** 2026-05-06

---

## Decyzje

| Data | Decyzja | Dlaczego | Kto |
|------|---------|----------|-----|
| 2026-05-06 | Prompt operuje w 6 fazach (Discovery → Synthesis → Materialize → Verify → Tailored Extensions → Registration Guide) z checkpointem po Phase 2 | Bez Synthesis-checkpointa Copilot generuje config oparty na halucynacjach. Checkpoint łapie błędy zanim powstaną pliki. | Michał + @cto |
| 2026-05-06 | Materializujemy `.github/instructions/*.instructions.md` z `applyTo` — wbrew oficjalnemu cheat-sheetowi GitHuba | Plugin JetBrains 1.5.65 to wspiera (sekcja "Instruction Files" w Settings → Customizations). Docs są niedoaktualizowane. | Michał + @cto |
| 2026-05-06 | NIE generujemy `.github/chatmodes/*.chatmode.md` ani `.vscode/settings.json` Copilot keys | To feature VS Code, plugin JetBrains tego nie czyta. | @cto |
| 2026-05-06 | Frontmatter prompt files: tylko `description:`. NIE `model:`/`mode:`/`tools:` | Plugin JetBrains ignoruje pola VS Code. Sterowanie modelem zostaje w treści prompta jako rekomendacja dla użytkownika. | @cto |
| 2026-05-06 | Filozofia "no overengineering" wpisana do KAŻDEJ warstwy (instructions, instruction overlays, każdy prompt, AGENTS.md, CLAUDE.md) | Z którejkolwiek warstwy Copilot ciągnie kontekst, ta zasada ma być widoczna. Inaczej tendencja do overengineerowania wraca. | Michał |
| 2026-05-06 | Self-improvement realizowany przez `.github/copilot-lessons.md` + protokół w `copilot-instructions.md` | Copilot nie ma natywnej pamięci. Plik referencjonowany z głównych instrukcji jest auto-loadowany jak każdy inny kontekst → simulacja memory. | Michał + @cto |
| 2026-05-06 | Tailored Extensions (Phase 5) wymagają evidence (`Evidence: file:line`) zanim cokolwiek zaproponują | Bez tego model proponuje 15 generic items, połowa nieadekwatna. Wymóg dowodu zmusza do uzasadnienia. | @cto |
| 2026-05-06 | Repo `copilot-support` jest publiczne na GitHubie | Materiał użyteczny dla innych developerów w podobnej sytuacji. Brak danych firmy/klienta — tylko prompt + opis. | Michał |

---

## Do przemyślenia

- Czy MCP server zamiast prompta do wklejania byłby lepszy? Plus: single source of truth, aktualizowany centralnie. Minus: kolejny komponent do utrzymania, autoryzacja w środowisku klienta.
- Co zrobić jak GFT zaktualizuje plugin do nowszej wersji która ma już slash-invocation custom promptów? → Zaktualizować Phase 6 registration guide.
- Czy warto dodać dedykowaną sekcję "compliance/security" do prompta dla projektów regulowanych (bankowość, GDPR)?
- Wzbogacić o automatyczne wykrywanie języka komentarzy / docstringów (PL/EN) i dopasować styl?

---

## Zmiany w planie

- 2026-05-06: pierwsza wersja v1 — żadnych zmian kierunku jeszcze.
