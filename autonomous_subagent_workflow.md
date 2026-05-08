# Autonomous Subagent Workflow

**Wersja:** 1.0 (2026-05-08)
**Target:** GitHub Copilot plugin for JetBrains 1.6.1+ z **`Enable Subagent: ON`**
**Wymaga:** wcześniejszego bootstrapa serwisu przez `copilot_jetbrains_bootstrap_v2.md` (tj. obecności `.github/agents/architect.agent.md`, `coder.agent.md`, `reviewer.agent.md`, `debugger.agent.md`)

---

## Cel

Uruchomić **jedną sesję** w Copilot Chat (Agent mode), w której orkiestrator wywołuje custom agentów jako subagenty w pętli: **plan → implement → debug → fix → review → fix → done**, sam decydując kiedy potrzebuje którego specjalisty, z explicit warunkami stopu i pytaniami do użytkownika tylko gdy naprawdę musi.

Cel finalny: **dotykasz Copilota raz na początku sesji, kolejny raz przy ewentualnym pytaniu lub na końcu**.

## Kiedy używać (vs ręczne klikanie)

| Scenariusz | Workflow |
|---|---|
| Feature ticket z jasnym specem, 1-3 pliki | Sesja 2 bezpośrednio (skip sesji 1) |
| Feature wymagający planowania, >3 pliki, kontrakt | Sesja 1 (Architect) → Sesja 2 (autonomiczna pętla) |
| Bug fix z stacktrace | Wariant "Bug fix" niżej |
| Refactor wewnętrzny, bez nowych funkcjonalności | Wariant "Refactor only" niżej |
| Mikrozmiana (typo, 1-line) | NIE używaj tego — inline agent (Shift+Cmd+I) lub zwykły Coder |

## Wymagania techniczne (sprawdź raz, przed pierwszym uruchomieniem)

Settings → Tools → GitHub Copilot → Chat:
- [x] **Enable Agent mode** ON
- [x] **Enable Custom Agent** ON
- [x] **Enable Subagent** ON ← **kluczowe; po włączeniu wymaga restartu IDE**
- [x] **Agent Max Requests** ≥ 50 (default 25 może być za mało dla pełnej pętli)

Settings → Tools → GitHub Copilot → Chat → Auto Approve:
- Edits Auto-approve patterns dodaj/upewnij się że są:
  - `**/.github/copilot-lessons.md` ✅
  - `**/.github/agents/*` ✅
  - `**/WIP/*` ✅ (jeśli używasz folderu na specyfikacje)

Po pełnym restarcie IDE upewnij się że pliki w `.github/agents/` są widoczne w Settings → Tools → GitHub Copilot → Customizations jako "Chat Agents".

---

## Sesja 1 — Architect produkuje specyfikację

Cel: wyciągnąć z głowy / ticketu / wymagań plan, który da się potem autonomicznie wykonać. Spec idzie do pliku, więc sesja 2 zaczyna z pełnym kontekstem nawet jeśli context window się resetuje.

### Setup
- **Model:** Claude Opus 4.6 (planowanie wymaga rozumowania)
- **Tryb:** Agent mode
- **Folder na specy:** utwórz `WIP/` w repo (gitignored albo w gałęzi feature)

### Prompt sesji 1 (kopiuj, edytuj `<...>`)

```
[Agent mode, model: Opus 4.6]

Zadanie: <wklej ticket / opis featury w 1-5 zdaniach>

Działaj jako Architect zgodnie z .github/agents/architect.agent.md.
Wynik zapisz do pliku WIP/<nazwa-feature>-plan.md w formacie Architect:
Goal / Affected files / Contract impact / Steps / Test plan / Risks / Out of scope.

Po zapisaniu pliku — STOP. Nie implementuj. Nie wywołuj innych agentów.
Wypisz w czacie ścieżkę zapisanego pliku.
```

### Co dostaniesz
Plik `WIP/<nazwa>-plan.md` ze szczegółowym planem. **Przeczytaj go**. To jest jedyny moment w całym workflow gdzie naprawdę musisz pomyśleć — czy plan ma sens. Jeśli tak → sesja 2. Jeśli nie → wróć do Architecta z feedbackiem (handoff "Refine plan" lub po prostu nowy prompt z tym samym planem + uwagi).

---

## Sesja 2 — Autonomiczna pętla

Cel: wziąć plan, wykonać go, znaleźć błędy, naprawić, zreview'ować, naprawić ponownie, dostarczyć czysty wynik. Bez Twojego udziału, dopóki nie napotka realnej blokady.

### Setup
- **Nowa sesja w Copilot Chat** (czyste context window — to ważne; plan jest w pliku, nie w pamięci)
- **Model:** Opus 4.6 jako orkiestrator (musi podejmować decyzje którego subagenta wywołać)
- Subagenty same w sobie używają modeli zdefiniowanych w swoich `.agent.md` (Sonnet dla Coder, Opus dla pozostałych)

### Prompt sesji 2 — pełna pętla z planu

```
[Agent mode, model: Opus 4.6]

Plan: WIP/<nazwa-feature>-plan.md ← przeczytaj.

Wykonaj następującą pętlę używając subagentów z .github/agents/:

1. CODER: implementuj plan krok po kroku, smallest viable diff, match conventions.
2. DEBUGGER: przeanalizuj implementację pod kątem błędów logicznych, edge cases, 
   contract violations. Zwróć ranked hipotezy z evidence (file:line).
3. Jeśli DEBUGGER znalazł błędy o LIKELIHOOD: high/medium → CODER fixuje → wróć do 2.
   Jeśli wszystkie low lub brak → przejdź do 4.
4. CODER: uruchom build i testy lokalnie. Jeśli failują → fixuj → wróć do 4.
5. REVIEWER: pełen review diffu wg checklisty z reviewer.agent.md.
6. Jeśli REVIEWER zwrócił FIXES_NEEDED z severity must → CODER aplikuje → wróć do 5.
   Jeśli tylko should/nit → zostaw, raportuj na końcu.
   Jeśli OK → przejdź do 7.
7. Final report: lista zmienionych plików (file:line), nowych testów, 
   ryzyk/open questions, wyniku build+testów, pominiętych nits.

WARUNKI STOPU PĘTLI (oprócz sukcesu):
- Pętla 2-3 wykonała 3 obroty bez progresu (te same hipotezy wracają) → STOP, zapytaj.
- Pętla 5-6 wykonała 3 obroty bez OK → STOP, zapytaj.
- Brakuje danych których nie da się zdobyć z repo (np. kontrakt zewnętrznego API) → STOP, zapytaj jednym konkretnym pytaniem.
- Krok wymaga NEW DEPENDENCY / NEW PATTERN / CONTRACT CHANGE → STOP, zapytaj zanim wprowadzisz.
- Agent Max Requests się wyczerpuje (zostało <5) → STOP, raportuj progres.

Każde wywołanie subagenta ogłaszaj w chacie nagłówkiem typu:
"→ Subagent: coder (implementacja kroku 2 z planu)"
"→ Subagent: reviewer (review diffu)"

Po każdym subagencie pokaż w chacie kluczowy output (diff, lista hipotez, 
review outcome) zanim przejdziesz do następnego kroku.

Zacznij teraz.
```

### Co powinno się stać

W czacie zobaczysz coś w stylu:
```
→ Subagent: coder (implementacja kroku 1: dodanie endpoint /api/v1/foo)
[diff]
→ Subagent: debugger (analiza implementacji)
[hipotezy]
→ Subagent: coder (fix hipoteza #1)
[diff fix]
→ Subagent: reviewer (pełen review)
REVIEW OUTCOME: FIXES_NEEDED
1. [must] FooController.java:42 — brak walidacji X
→ Subagent: coder (fix must items)
[diff fix]
→ Subagent: reviewer (re-review)
REVIEW OUTCOME: OK
[final report]
```

---

## Warianty workflow (różne use case'y)

### Wariant A — Bug fix (zaczynasz od Debuggera, nie Architect)

```
[Agent mode, model: Opus 4.6]

Bug: <opis symptomu>
Stacktrace / log:
<wklej>

Pętla z subagentów .github/agents/:
1. DEBUGGER: ranked hipotezy z evidence.
2. STOP po pierwszej iteracji DEBUGGERA i pokaż mi hipotezy. 
   Zaczekaj na mój pick (#1, #2, ...) lub "investigate more".
3. Po moim picku: CODER implementuje fix + regression test.
4. REVIEWER: review.
5. Jeśli FIXES_NEEDED must → CODER fixuje → wróć do 4.
6. Jeśli OK → final report.

Warunki stopu jak wyżej.
```

Tu jeden checkpoint dla Ciebie (pick hipotezy) jest celowy — Debugger nigdy nie powinien zgadywać sam.

### Wariant B — Refactor (bez Debuggera, bez Architecta jeśli zmiana mała)

```
[Agent mode, model: Sonnet 4.6]

Refactor: <opis>
Pliki: <lista>
NIE zmieniaj zachowania zewnętrznego (kontrakt, API, schema).

Pętla z subagentów .github/agents/:
1. CODER: refaktor zgodny z istniejącymi konwencjami, smallest viable diff.
2. CODER: uruchom istniejące testy. Wszystkie zielone? Tak → 3. Nie → fix → 2.
3. REVIEWER: review pod kątem (a) brak zmiany behaviour, (b) overengineering, (c) konwencje.
4. Jeśli FIXES_NEEDED must → CODER → wróć do 3.
5. Jeśli OK → final report.

Warunki stopu jak wyżej. Jeśli refactor wymaga zmiany kontraktu → STOP, zapytaj.
```

Orkiestrator może być Sonnet (taniej) bo refactor nie wymaga ciężkiego rozumowania.

### Wariant C — Code review only (przed PR-em, bez implementacji)

```
[Agent mode, model: Opus 4.6]

Branch: <feature-branch> (diff vs main / develop).
Cel: pełen code review, wygenerowanie listy poprawek do akceptacji.

Pętla z subagentów .github/agents/:
1. REVIEWER: review wszystkich plików w diffie wg pełnej checklisty.
2. STOP po review. Pokaż mi outcome:
   - Lista must items (do naprawienia przed PR)
   - Lista should items (warto naprawić)
   - Lista nits (kosmetyka)
3. Czekaj na moje "fix all musts" / "fix all" / "skip" / instrukcję.
4. Jeśli zlecę fix → CODER aplikuje wskazane → REVIEWER re-review → 4 jeśli must zostały.
5. Jeśli OK lub "skip" → final report.
```

Tu user driver decyzji — bo to jego PR, nie chce żeby AI sama zdecydowała co "trzeba" zmienić.

### Wariant D — Pełna feature od zera w jednej sesji (bez sesji 1)

Jeśli czas naciska i nie chcesz robić Architect → spec → Coder workflow:

```
[Agent mode, model: Opus 4.6]

Feature: <opis ticketu, 5-15 zdań>

Pętla z subagentów .github/agents/:
1. ARCHITECT: krótki plan w chacie (NIE zapisuj do pliku, sesja jest jedna).
   Format Architect: Goal / Affected files / Contract impact / Steps / Test plan / Risks.
2. STOP po planie. Pokaż mi go. Czekaj na moje "go" lub uwagi.
3. Po "go": CODER implementuje krok po kroku.
4. DEBUGGER: analiza, hipotezy.
5. CODER: fix high/medium hipotez → wróć do 4 (max 3 obroty).
6. REVIEWER: review.
7. CODER: fix must items → wróć do 6 (max 3 obroty).
8. Final report.

Warunki stopu standardowe. Jeden checkpoint dla mnie po Architect (krok 2).
```

Mniej autonomiczne (1 checkpoint), ale szybsze niż dwie sesje. Plan nie idzie do pliku — jeśli context się skończy, zaczynasz od zera.

---

## Diagnostyka: czy subagent invocation rzeczywiście działa

W trakcie pierwszego uruchomienia popatrz w czat:

| Co widzisz | Co to znaczy |
|---|---|
| Nagłówki "→ Subagent: coder" + zmiana stylu / długości output | ✅ Działa, plugin wywołuje custom agentów |
| Brak nagłówków, Copilot pisze całość jednym ciągiem | ❌ Plugin udaje że ma subagentów ale ich nie wywołuje |
| "I cannot call subagents" / "subagent feature unavailable" | ❌ Subagent toggle wyłączony, restart nie zadziałał, lub policy blokuje |
| Wywoła 1 subagenta i koniec | ⚠️ Częściowo działa — orkiestrator nie kontynuuje pętli |

W przypadku ❌ lub ⚠️ → patrz **Plan B** niżej.

### Sprawdzenie liczby premium requestów

Po pierwszym uruchomieniu pętli wejdź na: **github.com → Settings → Copilot → Premium request usage** (lub equivalentny dashboard zależnie od planu). Porównaj liczbę przed/po. To pokazuje realnie ile cię kosztowała pętla.

Heurystyka spodziewanego kosztu pełnej pętli z planu:
- 1 sesja Architect (Opus): ~3-5 requestów
- 1 sesja autonomiczna (Opus orkiestrator + Sonnet Coder + Opus Reviewer/Debugger): ~10-25 requestów
- Razem: ~15-30 requestów na średni feature

Jeśli widzisz wielokrotnie więcej (>50), Copilot wpadł w pętlę bez wykrycia. Skup się na lepszych warunkach stopu w prompcie.

---

## Plan B — gdy subagent invocation NIE działa (manualnie ale działa zawsze)

Jeśli plugin nie wywołuje subagentów automatycznie (znany bug w niektórych buildach 1.6.x), workflow rozkłada się na ręczne kroki — wciąż 4-5x szybciej niż pisanie wszystkiego od zera bez agent files.

### Manualna pętla (ten sam plan, ale klikasz między sub-stepami)

```
KROK 1 — Coder implementuje:
[Agent mode, model: Sonnet 4.6]
> #file:.github/agents/coder.agent.md
> #file:WIP/<nazwa>-plan.md
> Implementuj plan. Po każdym kroku pokaż diff i czekaj na "next" lub "fix X".

[user klika "next" / poprawia]

KROK 2 — Debugger sprawdza:
[ten sam czat lub nowy, model: Opus 4.6]
> #file:.github/agents/debugger.agent.md
> Powyżej diff Codera. Ranked hipotezy z evidence.

[user wybiera hipotezę albo "no issues"]

KROK 3 — Coder fix (jeśli były błędy):
> #file:.github/agents/coder.agent.md
> Fix hipoteza #N. Smallest possible change. Add regression test.

KROK 4 — Reviewer review:
[model: Opus 4.6]
> #file:.github/agents/reviewer.agent.md
> Pełen review diffu powyżej.

KROK 5 — Coder fix must items (jeśli były):
> #file:.github/agents/coder.agent.md
> Fix must items z review powyżej.

KROK 6 — Reviewer re-review:
> #file:.github/agents/reviewer.agent.md
> Re-review po fixach.
```

To jest 6 wklejeń zamiast jednego, ale wciąż używasz tych samych prompt files i agentów.

### Różnica vs sub-agent workflow

| Aspekt | Subagent (auto) | Manualnie (Plan B) |
|---|---|---|
| Liczba twoich klików | 1-2 | 4-6 |
| Liczba premium requestów | ~15-30 | ~10-20 (mniej bo orkiestrator nie żyje) |
| Twój czas zaangażowany | 5 min | 20-30 min |
| Działa w buggy buildach pluginu | Może nie | Zawsze |

---

## Pułapki i tipy

### 1. Context window indicator
Patrz na pasek w panelu Copilot. Po przekroczeniu ~70% jakość spada — skończ sesję, zapisz progres do `WIP/<nazwa>-progress.md`, zacznij nową z plikiem progress.

### 2. Agent Max Requests w jednej turze
Default 25 wystarcza na 1-2 obroty pętli. Dla pełnego workflow ustaw 50 lub 100. Settings → Chat → Agent Max Requests.

### 3. Edits Auto-approve
Bez tego pętla zatrzyma się przy każdym `write_file` z prośbą o akceptację. Dodaj wzorce dla folderów które agent legalnie modyfikuje (`src/**`, `**/.github/copilot-lessons.md`, `WIP/**`). NIE dodawaj jeśli boisz się że Coder coś rozjedzie — wtedy lepiej manualnie.

### 4. Tani orkiestrator zamiast Opus
Jeśli pętla jest prosta (np. wariant Refactor), spróbuj Sonnet 4.6 jako orkiestrator. Działa, oszczędza requesty. Dla skomplikowanych decyzji (Architect first, contract changes, debug ambiguity) zostaw Opus.

### 5. Jedno zadanie = jedna pętla
Nie próbuj "zrób mi feature X i przy okazji refactor Y i bug Z". Pętla zgubi się. Jedno zadanie, jedna pętla. Następne zadanie = nowa sesja.

### 6. Self-improvement działa też w pętli
Gdy w trakcie pętli zauważysz że agent robi coś nie tak — przerwij, popraw, dopisz do `.github/copilot-lessons.md`. Następna pętla już to honoruje (lekcje są w kontekście wszystkich agentów).

### 7. Plan w pliku ≠ plan w pamięci
Kiedy używasz sesji 1 → sesji 2 z plikiem planu, **plik jest jedynym source of truth**. Jeśli edytujesz plan w pliku, sesja 2 zobaczy edycję. Jeśli edytujesz w pamięci (chat) i nie zapiszesz do pliku — przepada.

### 8. Brak progresu = pytanie do Ciebie, nie pętla bez końca
Warunki stopu w prompcie ("3 obroty bez progresu → STOP, zapytaj") są krytyczne. Bez nich orkiestrator może zalogować 100 requestów próbując te same fixy. Zawsze daj limit obrotów.

---

## Mini-przykład end-to-end (referencja)

**Ticket:** "Dodaj endpoint POST /api/v1/users do tworzenia usera, walidacja email, persistence przez UserRepository, integracyjny test."

### Sesja 1 (~3 min, ~5 requestów)
Wklejasz prompt sesji 1. Architect tworzy `WIP/create-user-plan.md` z:
- Goal: nowy endpoint z walidacją
- Affected: `UserController.java`, `UserService.java`, nowy `CreateUserRequest.java`, `UserControllerIT.java`
- Contract: REST `POST /api/v1/users` body `{email, name}` → 201 + `{id}` lub 400.
- Steps: 5 kroków
- Tests: 1 IT, 2 unit, 1 walidacji
- Risks: email uniqueness vs DB constraint

Czytasz, OK, przechodzisz dalej.

### Sesja 2 (~10 min, ~20 requestów, 0-1 twoich kliknięć)
Wklejasz prompt sesji 2. Czat:
```
→ Subagent: coder (kroki 1-5 z planu)
[diff: 4 pliki zmodyfikowane / utworzone]
→ Subagent: debugger
LIKELIHOOD: medium — UserRepository.findByEmail może zwrócić null, kod zakłada Optional
→ Subagent: coder (fix #1)
[diff fix]
→ Subagent: debugger (re-check)
[brak nowych hipotez high/medium]
→ Subagent: coder (build + tests)
[zielone]
→ Subagent: reviewer
REVIEW OUTCOME: FIXES_NEEDED
1. [must] UserController.java:34 — brak HTTP 409 dla duplicate email
2. [should] CreateUserRequest.java:18 — @Valid na poziomie controllera, nie service
→ Subagent: coder (fix must #1)
[diff]
→ Subagent: reviewer (re-review)
REVIEW OUTCOME: OK
[final report]
```

Total: ~13 minut, ~25 requestów, 0 twoich kliknięć po wklejeniu prompta sesji 2 (chyba że chcesz fixnąć "should" item — wtedy 1 follow-up).

---

## Po użyciu workflow w 2-3 realnych zadaniach

Zaktualizuj ten plik o lekcje:
- Co warunki stopu rzeczywiście wyłapały, a co przegapiły
- Czy orkiestrator wywołuje subagentów w przewidywanej kolejności
- Czy realny koszt requestów odpowiada heurystyce
- Które warianty (A/B/C/D) są realnie używane, a które nie

Nieużywane warianty wytnij. Nowe pojawiające się patterny dopisz.
