# LOG DECYZJI

**Data utworzenia:** 2026-05-06

---

## Decyzje

| Data | Decyzja | Dlaczego | Kto |
|------|---------|----------|-----|
| 2026-05-07 | Plugin update do 1.6.1-243 → tworzymy v2 prompt + osobny upgrade prompt dla istniejących serwisów (zamiast wymuszać re-bootstrap) | v1 nie wykorzystuje Custom Agents (GA w 1.6.x). Re-bootstrap niszczyłby `.github/copilot-lessons.md` i lokalne dostosowania. Upgrade prompt jest in-place i additive. | Michał + @cto |
| 2026-05-07 | Architektura advisor: 4 custom agents (architect/coder/reviewer/debugger) + handoffs | Tani Sonnet jako default (Coder) automatycznie deleguje do drogiego Opus (Reviewer) PRZED claim "done" → maksymalna wartość/sesja przy minimalnym koszcie. Architect dla multi-step, Debugger dla incydentów. `handoffs` field w frontmatter agent files jest natywnie wspierany w 1.6.x. | Michał + @cto |
| 2026-05-07 | Upgrade prompt jest dual-mode (interactive vs autonomous, user wybiera w Phase 0) | Różne serwisy mają różną tolerancję na ryzyko. Krytyczne repo → interactive. Greenfield/playground → autonomous (jedna sesja, max wartości). | Michał |
| 2026-05-07 | Hooks (preview) i MCP per-agent — skip w v2, zostaje na v3 | Hooks są preview, mogą się zepsuć przy update plugina. MCP per-agent (`mcp-servers` w frontmatter) dodaje znaczną złożoność, lepiej wprowadzać dopiero gdy v2 będzie sprawdzony w terenie. | Michał + @cto |
| 2026-05-07 | Nested AGENTS.md generujemy tylko gdy Phase 1.7/2.2 znajdzie konkretne singularities (legacy subtree, vendor integration, FE w BE repo) | Bez evidence → bloat. AGENTS.md działa hierarchicznie (narrower wins) tylko jeśli rzeczywiście jest co wpisać. | @cto |
| 2026-05-07 | `target` field w agent frontmatter pomijamy (cross-IDE) | `target: 'vscode'` zawężałby agenta do VS Code. Nasze agents mają działać w JetBrains (główny use case GFT) i nie ograniczać dostępności. | @cto |
| 2026-05-07 | `model:` w custom agentach: dokładny string z JetBrains dropdown ("Claude Sonnet 4.6", "Claude Opus 4.6") | Docs używają "Claude Sonnet 4.5" w przykładzie — string jest case-sensitive i match-sensitive. Dla pewności trzymamy się stringa z UI; w prompcie zaznaczamy żeby user zweryfikował i poprawił jeśli inny. | @cto |
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
- v3: dodać `mcp-servers` per-agent (np. Atlassian/Jira MCP dla architect.agent.md w GFT) gdy v2 będzie sprawdzony w 2-3 serwisach.
- v3: dodać hooks (preview) — preToolUse confirm-destructive + postToolUse format-on-write, gdy preview wyjdzie do GA.
- Czy warto dodać dedykowaną sekcję "compliance/security" do prompta dla projektów regulowanych (bankowość, GDPR)?
- Wzbogacić o automatyczne wykrywanie języka komentarzy / docstringów (PL/EN) i dopasować styl?
- Po pierwszym realnym teście v2: zmierzyć czy 4-agent advisor faktycznie redukuje liczbę turn na zadanie (handoff Coder→Reviewer powinien zastąpić ręczne odświeżanie konwersacji).

---

## Zmiany w planie

- 2026-05-07: plugin GFT zaktualizowany do 1.6.1-243 (native AGENTS.md/CLAUDE.md/Custom Agents). Dodajemy v2 prompt + upgrade prompt v1→v2 ZANIM zrobimy pierwszy realny test z Milestone 2. Test będzie na v2 (świeży serwis) lub upgrade (jeśli któryś serwis dostanie v1 wcześniej).
- 2026-05-06: pierwsza wersja v1 — żadnych zmian kierunku jeszcze.
