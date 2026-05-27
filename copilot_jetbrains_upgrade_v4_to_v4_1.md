# Copilot JetBrains Upgrade Prompt — v4 → v4.1

**Wersja:** 1.0 (2026-05-27)
**Cel:** Dla serwisów które już mają v4 (`.github/agents/cto.agent.md`, brak `data/`) — dorobić warstwę trwałego kontekstu (`data/structure-map.md`, `data/api-inventory.md`, `data/conventions.md`, `data/known-issues.md`) + `refresh-context.prompt.md` + update CTO body o Protokół Zero data/ + Consistency Spot-Check.
**Target plugin:** GitHub Copilot for JetBrains 1.6.1+
**Token-economical:** ~3000 tokens trwałego context per session zamiast re-discovery struktury za każdym razem.

---

## Kiedy użyć

Użyj **gdy:**
- Serwis ma `.github/agents/cto.agent.md` (v4 bootstrap zrobiony).
- NIE ma folderu `data/` (lub jest ale nie wygląda na format v4.1).
- CTO przy każdej sesji "re-discoveruje" strukturę — chcesz to wyciąć.

NIE używaj **gdy:**
- Świeży serwis bez v4 → użyj nowy `copilot_jetbrains_bootstrap_v4.md` (zawiera już data/).
- Serwis ma v3 lub starszy → najpierw upgrade do v4, potem ten.

## Jak użyć

1. Otwórz serwis w IntelliJ.
2. Copilot Chat → Agent mode → Opus 4.6.
3. Wklej całość poniżej.
4. Wybierz tryb: `1` interactive lub `2` autonomous.
5. Po zakończeniu pełen restart IDE → smoke test CTO.

---

# PROMPT — kopiuj wszystko poniżej

````markdown
# ROLE
You are a Principal/Staff Engineer adding persistent context layer to an already-bootstrapped v4 setup. The service has `.github/agents/cto.agent.md` (single CTO agent, v4) but lacks `data/*.md` files. Your job: re-run TARGETED discovery to materialize `data/structure-map.md`, `data/api-inventory.md`, `data/conventions.md`, plus empty scaffold for `data/known-issues.md`, plus `.github/prompts/refresh-context.prompt.md`, plus update `cto.agent.md` body's Protocol Zero to read these new files + add Consistency Spot-Check.

You have read access; write only under `data/`, `.github/prompts/refresh-context.prompt.md`, and `.github/agents/cto.agent.md` (body update). DO NOT touch `.github/copilot-instructions.md`, `.github/copilot-lessons.md`, existing prompts, instructions/, or AGENTS.md.

# OBJECTIVE
1. Detect v4 setup (signatures below).
2. Re-run targeted Phase 1 discovery (minimal — just what's needed for data/ files).
3. Generate `data/structure-map.md` (≤80 lines), `data/api-inventory.md` (≤150 lines), `data/conventions.md` (≤80 lines), `data/known-issues.md` (empty scaffold).
4. Generate `.github/prompts/refresh-context.prompt.md`.
5. Update `.github/agents/cto.agent.md` body: extend Protokół Zero list with data/* files, insert Consistency Spot-Check section, insert WHEN DATA/ GETS STALE section.
6. Preserve all other files byte-for-byte.

Dual-mode: interactive vs autonomous.

---

# PHASE 0 — MODE SELECTION

Print:
```
Upgrade modes:
  [1] interactive   — confirm each file write
  [2] autonomous    — one checkpoint after Plan, then full materialize

Reply with `1` or `2`.
```

Stop. Wait for reply.

---

# PHASE 1 — DETECT v4 (read-only)

Check:
- `.github/agents/cto.agent.md` exists
- `.github/copilot-instructions.md` exists (any version)
- `.github/copilot-lessons.md` exists (preserve)
- `data/` folder DOES NOT exist (or exists but empty / missing v4.1 files)

If `cto.agent.md` MISSING → STOP:
```
This service does not have v4 setup. Run `copilot_jetbrains_bootstrap_v4.md` first.
```

If `data/structure-map.md` EXISTS → STOP:
```
data/ already exists. This service may already be on v4.1. If you want to refresh data/, use `#prompt:refresh-context` instead.
```

If signatures OK → record state:
- Sub-repo path
- Whether `.github/instructions/*` exists (to know what's already covered)
- Whether `AGENTS.md` exists at root

---

# PHASE 2 — TARGETED DISCOVERY (read-only, minimum needed)

Don't re-do full v4 Phase 1 — that's already in `copilot-instructions.md`. Just gather what `data/` files need:

## 2.1 Structure (for structure-map.md)
- Top-level directory listing
- Per top-level dir: 1-line role (read README or infer from package names)
- Module structure if multi-module

## 2.2 Endpoints (for api-inventory.md)
- All REST controllers → method/path/file:line, auth annotation, has OpenAPI?
- All Kafka producers → topic, file:line, partition key
- All Kafka consumers → topic, consumer group, listener file:line, idempotent?, DLQ?
- Outbound HTTP clients → target service, called from file:line, failure mode
- DB schemas + main tables (from Flyway/Liquibase migrations or @Entity classes)

## 2.3 Conventions (for conventions.md)
- Naming patterns (controller / service / repo / DTO / test) — pick 1 real example each with file:line
- Error handling style — custom exception class file:line + global handler file:line
- Logging conventions — sample with file:line
- Test framework + 1 example file:line
- DI style — 1 example
- Async/reactive vs blocking — note which packages use which

This is FAST — most of this is already in copilot-instructions.md, just need to extract concrete file:line examples for data/.

---

# PHASE 3 — PLAN

Print:
```
UPGRADE PLAN — v4 → v4.1

ADD (new files)
  data/structure-map.md              (~50-80 lines, structure from Phase 2.1)
  data/api-inventory.md              (~80-150 lines, endpoints from Phase 2.2)
  data/conventions.md                (~50-80 lines, conventions from Phase 2.3)
  data/known-issues.md               (empty scaffold ~15 lines)
  .github/prompts/refresh-context.prompt.md  (~50 lines)

UPDATE (in place, only body section)
  .github/agents/cto.agent.md        Extend Protokół Zero with data/* files, insert Consistency Spot-Check and WHEN DATA/ GETS STALE sections

PRESERVE byte-for-byte
  .github/copilot-instructions.md
  .github/copilot-lessons.md
  .github/instructions/*.instructions.md
  .github/prompts/*.prompt.md (existing — only ADD refresh-context.prompt.md)
  AGENTS.md
  CLAUDE.md (if exists)
```

Mode branch:
- **interactive**: "Phase 4 confirms each file. Reply `go`."
- **autonomous**: "Phase 4 runs in one pass. Reply `go`."

STOP. Wait.

---

# PHASE 4 — APPLY

## 4.1 Generate `data/structure-map.md`
Use template from v4.1 bootstrap Phase 3.9.1, fill with Phase 2.1 findings. Cap ≤80 lines.

## 4.2 Generate `data/api-inventory.md`
Use template from v4.1 bootstrap Phase 3.9.2, fill with Phase 2.2 findings. Tables format. ALL entries MUST have file:line. Cap ≤150 lines.

If service has many endpoints (>50) — group sensibly OR truncate to top ones + note `(+ N more, see <package> for full list)`. Don't blow past cap.

## 4.3 Generate `data/conventions.md`
Use template from v4.1 bootstrap Phase 3.9.3, fill with Phase 2.3 findings. Each rule MUST have ONE concrete file:line example. Cap ≤80 lines.

## 4.4 Generate `data/known-issues.md`
Empty scaffold per v4.1 bootstrap Phase 3.9.4. Header + format guidance + empty Issues section.

## 4.5 Generate `.github/prompts/refresh-context.prompt.md`
Exact content from v4.1 bootstrap Phase 3.4 NEW addition (refresh-context.prompt.md template).

## 4.6 Update `.github/agents/cto.agent.md` body

Read file. Locate `## PROTOKÓŁ ZERO (PRZED KAŻDĄ ODPOWIEDZIĄ)` section.

REPLACE its content (preserving frontmatter and surrounding sections) with v4.1 version that includes:
- Extended file list (data/structure-map.md, data/api-inventory.md, data/conventions.md, data/known-issues.md in Protocol Zero)
- New `## CONSISTENCY SPOT-CHECK (NEW v4.1)` section AFTER Protokół Zero
- New `## WHEN DATA/ GETS STALE — REFRESH PROTOCOL` section AFTER Consistency Spot-Check

Use exact wording from v4.1 bootstrap Phase 3.5 (cto.agent.md template).

In **interactive**: show diff before write. In **autonomous**: write directly.

## 4.7 DO NOT TOUCH — sanity check
Verify untouched:
- `.github/copilot-instructions.md`
- `.github/copilot-lessons.md`
- `.github/instructions/*`
- `.github/prompts/*` (except new refresh-context)
- `AGENTS.md`
- `CLAUDE.md`

---

# PHASE 5 — REPORT

```
UPGRADE REPORT — v4 → v4.1

Added (data/ persistent context)
  data/structure-map.md              <N lines>
  data/api-inventory.md              <N lines>  (M endpoints + M topics + M tables documented)
  data/conventions.md                <N lines>
  data/known-issues.md               <N lines>  (empty scaffold)

Added (new prompt)
  .github/prompts/refresh-context.prompt.md   <N lines>

Updated (cto.agent.md body — additive)
  .github/agents/cto.agent.md        +<N lines>  (Protokół Zero extended, Consistency Spot-Check added, WHEN DATA/ GETS STALE added)

Preserved (untouched)
  .github/copilot-instructions.md
  .github/copilot-lessons.md         <N entries — your accumulated lessons>
  .github/instructions/              <M files>
  .github/prompts/                   <M files, including new refresh-context>
  AGENTS.md
  CLAUDE.md (if exists)
```

---

# PHASE 6 — TEST

Print:
```
NEXT STEPS:

1. Pełny restart IntelliJ (Cmd+Q / File → Exit).

2. Settings → Tools → GitHub Copilot → Customizations → verify:
   - data/* files NIE pokazują się jako "instructions" (są czytane przez CTO body, nie przez Customizations native)
   - prompts/refresh-context.prompt.md widoczny w prompt files
   - cto.agent.md zaktualizowany

3. Smoke test (60 sec):
   - Agents dropdown → CTO → "Daj mi krótki AUTO-BRIEF tego serwisu na podstawie data/"
   - CTO powinien:
     a) Wykonać Protokół Zero (zobaczysz że czyta data/structure-map.md, data/api-inventory.md, data/conventions.md)
     b) Zrobić Consistency Spot-Check (sprawdzi 3 random entries z api-inventory)
     c) Dać AUTO-BRIEF bazując na trwałym kontekście — nie re-skanując kodu od zera
   - Jeśli widzisz "[STALE CONTEXT WARNING]" → ktoś dodał endpoint po bootstrap. Uruchom `#prompt:refresh-context`.

4. Test refresh:
   - Wpisz: `#prompt:refresh-context`
   - CTO porówna data/ z aktualnym kodem, zaproponuje diff
   - Zatwierdź → updates data/ files

5. Daily workflow zmienia się minimalnie — CTO szybciej startuje (mniej re-discovery), reszta jak v4.
   Kluczowe: gdy zrobisz duży refactor / nowy feature → uruchom `#prompt:refresh-context` żeby data/ nie zostało za kodem.
```

---

# CONSTRAINTS (all phases)
- No external network calls. No installs.
- No edits outside `data/`, `.github/prompts/refresh-context.prompt.md`, `.github/agents/cto.agent.md`.
- Never modify `.github/copilot-instructions.md`, `.github/copilot-lessons.md`, existing prompts, instructions/, AGENTS.md, CLAUDE.md.
- Never print secret values. Redact and warn if any file contains apparent secrets.
- All generated files in English.
- Be terse. Tables > prose.
- Honor line caps. If service is too big to fit api-inventory in 150 lines, truncate to most important + note "(+ N more, see <package>)" rather than blowing past cap.
- If uncertain, write `OPEN:` and continue.
````
