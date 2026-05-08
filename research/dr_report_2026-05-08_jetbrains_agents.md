# Deep Research Report — GitHub Copilot Custom Agents w JetBrains 1.6.1-243

**Data:** 2026-05-08
**Trigger:** Subagent invocation w `coder.agent.md` zwracał "failed" mimo poprawnego (jak myśleliśmy) frontmatter v2 z `handoffs:` i `model:`. Pierwsza diagnoza @cto (brak `'agent'` w `tools:`) była niepełna. Zażądano niezależnej weryfikacji przez Deep Research.
**Wykonawca DR:** Perplexity Pro / ChatGPT Pro Deep Research (prompt z chatu, oparty na pytaniach 1-8)
**Outcome:** Rewrite architektury custom agents na v3 (official-safe), publikacja `copilot_jetbrains_bootstrap_v3.md` + `copilot_jetbrains_upgrade_v2_to_v3.md`.

---

## TL;DR

Architektura advisor v2 (4 agenty z `handoffs:` i per-agent `model:`) jest **kompozycją VS Code'owych ficzerów, nieudokumentowanych dla JetBrains 1.6.x**. Część działa nieoficjalnie, część jest ignorowana, część rzuca runtime errory. Konkretnie:

1. **`handoffs:`** — nieudokumentowane w GitHub Docs dla JetBrains. Częściowo działa od 1.7.0-rc.4 (community evidence), w 1.6.1-243 niestabilne.
2. **`model:` per agent** — **ignorowany gdy agent działa jako subagent**. Subagent dziedziczy model z głównej sesji. Plus znany bug w 1.6.1 że bywa ignorowany też poza subagent context.
3. **Chain of subagents** — JetBrains official docs: "subagents cannot create other subagents". Max 1 poziom delegacji per sesja. Czyli pętla "architect → coder → reviewer" w jednej sesji to fantazja.
4. **Run-time bugs** w 1.6.1.x — nawet poprawny format pliku może zwracać "failed" z powodu issues #1476, #1467, #1374, plus utrata kontekstu projektu i broken auto-approve recovery.

Konsekwencja: rewrite na v3 z tylko oficjalnie udokumentowanymi polami, jednolitym modelem per sesja, i wieloetapowymi workflow przez `.github/prompts/full-feature-loop.prompt.md` zamiast subagent chains.

---

## Co weryfikowano (8 pytań do DR)

1. Oficjalna specyfikacja pełnego frontmatter pliku `.github/agents/<name>.agent.md` dla JetBrains 1.6.x — wszystkie pola, typy, required/optional, źródła z URL i datą.
2. Lista akceptowanych wartości w polu `tools:` dla IDE custom agents. Czy `'agent'` / `'runSubagent'` / `'Task'` to aliasy.
3. Czy `handoffs:` jest obsługiwane w JetBrains 1.6.x. Mechanizm zastępczy.
4. Minimalny działający przykład pliku `agent.md` wywoływalnego przez subagent.
5. Co dokładnie wymaga "Editor preview features" policy w enterprise.
6. Czy ktoś rozwiązał błąd "not invokable via run_subagent".
7. Różnice między JetBrains / VS Code / CLI / cloud agent.
8. Roadmap/workaround dla broken `run_subagent` w JetBrains.

## Hipotezy startowe @cto i werdykt DR

| Hipoteza | Werdykt |
|---|---|
| **A.** Brak `'agent'` w `tools:` Codera = root cause failure | **Częściowo błędne.** `'agent'` jest potrzebne tylko delegującemu (Architect). Brak go w child-agencie nie tłumaczy "not invokable". Faktyczny problem: `handoffs:` nieobsługiwane + bugi runtime w 1.6.1. |
| **B.** `handoffs:` może nie działać w JetBrains | **Potwierdzone — PARTIALLY.** GitHub Docs nie dokumentują dla JetBrains. Community evidence: częściowe działanie od 1.7.0-rc.4. W 1.6.1-243 niejednoznacznie. |
| **C.** Wymagane dodatkowe pole frontmatter | **Częściowo.** Dodatkowe użyteczne pola: `user-invocable` i `disable-model-invocation`. Ale brak ich nie jest blokerem dla discovery — to fine-tuning visibility/invocability. |
| **D.** Bug pluginu w 1.6.1.x | **Najbardziej prawdziwa hipoteza.** Cztery klasy bugów blokujące funkcjonalność niezależnie od formatu pliku (#1476, #1467, #1374, #1548). |

## Kluczowe ustalenia z DR

### 1. Oficjalny schema frontmatter (z `docs.github.com/en/copilot/reference/custom-agents-configuration`, ostatnia aktualizacja 2026-04-07)

| Pole | Typ | Wymagane | Uwagi |
|---|---|---|---|
| `name` | string | nie | default: filename bez `.agent.md` |
| `description` | string | **TAK** | używane przez orchestrator do auto-selekcji subagenta |
| `target` | string | nie | `vscode` \| `github-copilot`; default = oba |
| `tools` | string \| list | nie | omit = inherit wszystko z sesji |
| `model` | string | nie | **IGNOROWANY w subagencie** (subagent dziedziczy model sesji) + bug w 1.6.1 |
| `disable-model-invocation` | bool | nie | default `false` |
| `user-invocable` | bool | nie | default `true`; `false` ukrywa z dropdown ale callable jako subagent |
| `mcp-servers` | object | nie | nieużywane w JetBrains custom agents |
| `metadata` | object | nie | nieużywane w JetBrains |
| `infer` | bool | deprecated | nie używać |

**NIE ma w oficjalnej referencji JetBrains:** `handoffs`, `agents`, `argument-hint`, `hooks`. Te pola SĄ udokumentowane dla VS Code, ale nie dla JetBrains. GitHub Docs explicite: *"some properties may function differently, or be ignored, between the GitHub.com and IDE environments."*

### 2. Aliasy `tools:` (oficjalne, case-insensitive)

| Primary | Aliasy kompatybilne |
|---|---|
| `execute` | `shell`, `Bash`, `powershell` |
| `read` | `Read`, `NotebookRead` |
| `edit` | `Edit`, `MultiEdit`, `Write`, `NotebookEdit` |
| `search` | `Grep`, `Glob` |
| `agent` | `custom-agent`, `Task` |
| `web` | `WebSearch`, `WebFetch` |
| `todo` | `TodoWrite` |

MCP tools przez namespacing: `serverName/toolName` lub `serverName/*`.

**Ważne:** `runSubagent` z dokumentacji VS Code NIE jest oficjalnym aliasem w plik agent.md. To jest nazwa runtime'u w VS Code UX. W frontmatter: `agent`.

### 3. Subagent semantics w JetBrains (krytyczne ograniczenia)

Cytaty z GitHub Docs dla JetBrains:
- *"Subagents use the same tools and AI model as the main session."* → **per-agent `model:` jest zignorowany w subagent flow**.
- *"Subagents cannot create other subagents."* → **brak chain delegacji**.
- *"Copilot analyzes your prompt, the description field of configured custom agents, the current context and available tools, then automatically selects a subagent"* → selekcja subagenta odbywa się przez `description:`, NIE przez explicit handoffs.

To podważa sens advisor architecture v2 jako "Sonnet jako Coder + Opus jako Reviewer w jednej sesji subagentowej": w JetBrains te dwa agenty będą używać tego samego modelu (modelu sesji). Cost optimization wymaga osobnych sesji.

### 4. Status `handoffs:` w JetBrains 1.6.x = PARTIALLY

- GitHub Docs JetBrains: ani potwierdza, ani wyklucza. Dla Xcode mówią o handoffs UI explicite, dla JetBrains nie.
- GitHub Docs cloud agent: *"The argument-hint and handoffs properties from VS Code and other IDE custom agents are currently not supported for Copilot cloud agent on GitHub.com."* → potwierdza że to jest IDE-side pojęcie, nie cloud.
- Community evidence (issue feedback): w 1.7.0-rc.4 (WebStorm) handoffs są częściowo funkcjonalne — buttons pojawiają się przy WYBORZE agenta (z dropdown), nie po zakończeniu odpowiedzi (jak w VS Code). Bug fix był obiecany, status w 1.6.1 niejednoznaczny.
- W 1.7.0-rc.4 parser loguje: `unknown fields ignored: argument-hint, agents, handoffs` — czyli walidacja YAML mówi że pola są nieznane, a runtime częściowo je respektuje. To rozjazd.

**Wniosek dla v3:** nie używamy `handoffs:`. Delegation realizujemy przez explicit instrukcje w body Architecta + tool `agent`.

### 5. Known bugs w 1.6.1-243 blokujące custom agents niezależnie od formatu

Cztery klasy z DR:

| Klasa | Issue | Objaw |
|---|---|---|
| Visibility w enterprise | #1476 | GA features (Sub-agents, Custom Agents, Plan Agent) niewidoczne gdy "Editor preview features" wyłączone w org policy |
| Discovery w UI | #1467 | Custom agents nie wykryte w chat dropdown mimo poprawnych plików |
| Persistence | #1374 | Custom Agents i Plan mode znikają po update pluginu mimo poprawnych plików w `.github/agents/` |
| Visibility cross-org | #1548 | Agents enabled org-wide nie pojawiają się w agent select mimo Custom Agent toggle ON |

Plus runtime bugs:
- Utrata kontekstu projektu w sesji agentowej, child agents szczególnie podatni ("Cannot find project for tool invocation")
- Stan auto-approve dla `run_in_terminal` może się zepsuć po odmowie

To znaczy: **nawet poprawny v3 format może zwracać "failed" z powodu plugin bugs**. Trzeba mieć Plan B (manual sessions z `#file:` reference).

### 6. Co GitHub realnie zaleca dla wieloetapowych workflow w JetBrains

Z DR:
1. **Custom agent** jako trwała persona (jeden główny — u nas Architect)
2. **Prompt files** dla powtarzalnych jednorazowych kroków (u nas `analyze-feature`, `add-endpoint`, `full-feature-loop`)
3. **Agent skills** (preview) dla wieloetapowych workflow — wymaga "Editor preview features"
4. **Plan mode** z przyciskiem "Start Implementation"

W macierzy GitHub customization support: custom agents i subagents w JetBrains mają status preview (P), prompt files i agent skills też preview.

**Wniosek:** "Pełna pętla 4-agentowa w jednej sesji" to nie jest oficjalna ścieżka GitHub dla JetBrains. Zamiast tego: prompt files + selektywne subagent calls (max 1 poziom) + osobne sesje gdy potrzebny inny model.

### 7. Co działa, co nie — tabela podsumowująca

| Feature | VS Code | JetBrains 1.6.1 |
|---|---|---|
| Custom agent as persona | ✅ | ✅ (z bugami discovery) |
| Subagent invocation (1 level) | ✅ | ⚠️ (działa gdy plugin nie ma bug runtime) |
| Multi-level subagent chain | ✅ | ❌ (oficjalne ograniczenie) |
| Per-agent `model:` różny od sesji | ✅ | ❌ (ignorowane w subagent + bug w 1.6.1) |
| `handoffs:` UI buttons | ✅ | ⚠️ (od 1.7.0-rc.4, w 1.6.1 niestabilne) |
| `agents:`, `argument-hint:`, `hooks:` | ✅ | ❌ (parser loguje "unknown fields ignored") |
| `user-invocable`, `disable-model-invocation` | ✅ | ✅ (oficjalna common spec) |
| Prompt files (`#prompt:`) | ✅ | ✅ |
| Inline agent | ✅ | ⚠️ (preview od 1.6.x) |

### 8. Roadmap / oficjalny workaround

Brak oficjalnego dokumentu GitHub z konkretnym fixem dla "not invokable via run_subagent" w 1.6.1-243. Najbliższy stan wiedzy: czekać na nowsze buildy (1.6.2+, 1.7.0) i używać manualnego workflow przez `.github/prompts/` + `#file:` references.

---

## Implikacje dla naszej architektury copilot-support

### Co zmieniliśmy (v2 → v3)

1. **Wycięte z agent files:** `handoffs:`, `model:` (oba nieportable / ignored / buggy w JetBrains).
2. **Dodane:** `user-invocable: true` (Architect — entry point z dropdown), `user-invocable: false` (Coder/Reviewer/Debugger — helpers), `disable-model-invocation: false` jawnie (callable jako subagent).
3. **Tools:** Architect ma `'agent'` (deleguje). Coder/Reviewer/Debugger NIE mają `'agent'` (no chain). Coder ma `execute` żeby uruchamiać testy. Reviewer pozostaje read-only. Debugger ma `execute` dla diagnostic commands.
4. **Body agent files:** zamiast handoffs section — explicit "use the `agent` tool to invoke `coder`" w Architect + jawne ograniczenia ("DO NOT invoke other agents") w helperach.
5. **Nowy plik prompts/full-feature-loop.prompt.md** — multi-stage workflow z explicit user checkpoints między stage'ami. Zastępuje "advisor architecture przez subagent chain" z v2.
6. **Narrative w copilot-instructions.md i AGENTS.md** — wprost wymienia ograniczenia JetBrains 1.6.x żeby przyszły dev nie myślał że ma do dyspozycji VS Code semantics.

### Co NIE zmieniliśmy (intentionally)

- v1 (`copilot_jetbrains_bootstrap.md`) zostaje dla pluginu 1.5.65.
- v2 (`copilot_jetbrains_bootstrap_v2.md`) zostaje jako "VS Code-style experimental" — w czystym VS Code 1.6+ format z `handoffs:` powinien działać, więc nie kasujemy.
- v1→v2 upgrade prompt zostaje (potrzebny dla użytkowników którzy chcą wskoczyć z 1.5 do v2 najpierw, potem z v2 do v3).

### Action items

Zrobione (2026-05-08):
- [x] `copilot_jetbrains_bootstrap_v3.md` (świeży serwis na 1.6.1+, official-safe)
- [x] `copilot_jetbrains_upgrade_v2_to_v3.md` (przerabia istniejący v2 setup)
- [x] Ten DR report jako trwała lekcja
- [x] Update README z tabelą "który prompt wybrać"

Do zrobienia (po pierwszym realnym teście v3):
- [ ] Test v3 na realnym serwisie GFT — zmierzyć czy subagent invocation Architect → Coder działa, czy zwraca "failed"
- [ ] Test `#prompt:full-feature-loop` na realnym feature — czy stage'e są jasne, czy user wie kiedy switchować model/sesję
- [ ] Update DECYZJE.md po teście o realne lekcje
- [ ] Rozważyć v3.1 z `.github/skills/` (Agent Skills, preview) gdy GFT enable preview features policy

---

## Cytaty kluczowe (do referencji w przyszłości)

> "Subagents use the same tools and AI model as the main session."
> *— GitHub Docs, JetBrains custom agents, 2026-04*

> "Subagents cannot create other subagents."
> *— GitHub Docs, JetBrains custom agents, 2026-04*

> "Copilot analyzes your prompt, the description field of configured custom agents, the current context and available tools, then automatically selects a subagent."
> *— GitHub Docs, custom-agents-configuration, 2026-04-07*

> "The argument-hint and handoffs properties from VS Code and other IDE custom agents are currently not supported for Copilot cloud agent on GitHub.com."
> *— GitHub Docs, custom-agents-configuration, 2026-04-07*

> "Some properties may function differently, or be ignored, between the GitHub.com and IDE environments."
> *— GitHub Docs, custom-agents-configuration, 2026-04-07*

> "For consistent subagent behavior, define when to use subagents in your custom agent's instructions rather than prompting for them manually each time."
> *— VS Code Docs, Subagents, 2026-04-29 (concept transferable to JetBrains)*

---

## Następna weryfikacja

Plan: po **2 realnych użyciach v3 na serwisach GFT** wrócić do tych ustaleń. Jeśli okaże się że:
- subagent invocation działa stabilnie → potwierdzenie kierunku v3
- nadal zwraca "failed" → eskalacja: albo policy "Editor preview features", albo czekanie na 1.6.2+ buildy, albo jeszcze prostsza architektura "1 main agent + tylko prompt files, bez helperów"
- wieloetapowy workflow przez `full-feature-loop.prompt.md` jest niewygodny w praktyce → uproszczenie do "Architect tylko planuje + manualne sesje per faza"

Następna potencjalna iteracja: v3.1 (drobne korekty) lub v4 (jeśli rdzeń architektury wymaga rewritu po realnym tescie).
