# Copilot Workspace Init — Meta-Meta Prompt

**Wersja:** v1.0 (2026-05-27)
**Target:** GitHub Copilot for VS Code (latest, 2026)
**Cel:** Zainicjalizować workspace nadrzędny do pracy multi-repo z TechLead. JEDNORAZOWY setup PRZED uruchomieniem `copilot_vscode_multirepo_techlead.md`.

## Co to robi

Otwierasz folder nadrzędny (`~/work/` albo dedykowany `~/gft-platform/`) w VS Code i wklejasz ten prompt. Meta-init:

1. Skanuje folder i wykrywa kandydatów na sub-repo (heurystyka: `.git/` + build file).
2. Pyta Cię interactive które chcesz wciągnąć w scope GFT workspace (whitelist).
3. Generuje:
   - `<workspace-name>.code-workspace` (VS Code multi-root config)
   - `.copilot-workspace-config.yml` (whitelist sub-repo — TechLead to czyta)
   - `data/tech-radar.md` (template: allowed / avoided / planned technologies)
   - `data/architecture-decisions.md` (template ADR-ów ekosystemu)
4. Audytuje per sub-repo: czy ma `.github/agents/cto.agent.md` (v3 czy v4)? Pokazuje raport.
5. Generuje listę następnych kroków: które sub-repo bootstrap'ować/upgradować, kiedy uruchomić TechLead.

NIE robi: nie uruchamia bootstrap_v4 autonomicznie w sub-repo, nie modyfikuje sub-repo. Tylko skeleton workspace + raport.

## Workflow (kolejność użycia)

```
KROK 1: VS Code → otwórz folder nadrzędny
KROK 2: Wklej `copilot_workspace_init.md` → wybierz whitelist → generuje setup
KROK 3: Wypełnij data/tech-radar.md i data/architecture-decisions.md (opcjonalnie, można później)
KROK 4: Dla każdego sub-repo BEZ CTO (z raportu):
        → IntelliJ → otwórz repo → wklej copilot_jetbrains_bootstrap_v4.md
KROK 5: VS Code → wklej copilot_vscode_multirepo_techlead.md → wygeneruje TechLead
KROK 6: Reload window → smoke test TechLead → gotowe
```

## Jak użyć

1. Otwórz folder nadrzędny w VS Code (lub utwórz nowy dla GFT i tam wrzuć interesujące repo).
2. Settings → GitHub Copilot → Chat → włącz Agent mode, Custom Agent.
3. Copilot Chat → Agent mode → model **Claude Opus 4.6**.
4. Wklej całość prompta poniżej.
5. Interactive — pyta o whitelist sub-repo, potwierdzasz, generuje.
6. Po zakończeniu czytaj raport i wykonuj kroki z "Następne akcje".

---

# PROMPT — kopiuj wszystko poniżej

````markdown
# ROLE
You are a Senior Solution Architect doing one-time workspace initialization for a multi-repo Copilot setup in VS Code. Read access everywhere; write access only at workspace root for these specific files: `.code-workspace`, `.copilot-workspace-config.yml`, `data/tech-radar.md`, `data/architecture-decisions.md`, optional `AGENTS.md` and `CLAUDE.md`. DO NOT touch any sub-repo internals — they have their own configs (each managed by `copilot_jetbrains_bootstrap_v4.md` or v3 equivalent).

# OBJECTIVE
1. Scan the parent folder for candidate sub-repos.
2. Interactively confirm which sub-repos are in scope (user whitelist).
3. Generate workspace-level scaffold:
   - `<name>.code-workspace` (VS Code multi-root config with whitelisted folders)
   - `.copilot-workspace-config.yml` (explicit whitelist — TechLead reads this)
   - `data/tech-radar.md` (template for technology allow-list)
   - `data/architecture-decisions.md` (template for ecosystem-level ADRs)
4. Audit per sub-repo: presence of `.github/agents/cto.agent.md`, version (v3 with 4 helpers / v4 single CTO / none).
5. Produce a report with next actions:
   - Which sub-repos need bootstrap (no CTO at all)
   - Which sub-repos could upgrade (v3 → v4)
   - When to run TechLead bootstrap (after sub-repos are ready)
6. Do NOT execute the per-repo bootstrap or the TechLead bootstrap. Report only.

Operate in 5 phases. Interactive (the user picks the whitelist).

---

# PHASE 1 — SCAN PARENT FOLDER

## 1.1 Inventory of top-level folders
List all directories directly under the workspace root. Skip hidden ones (starting with `.`) unless they're known config (e.g., `.github`).

## 1.2 Classify each folder
For each top-level folder, check:
- Has `.git/`? (real git repo vs just a directory)
- Has a build file? (`pom.xml`, `build.gradle`, `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`)
- Has `README.md` at root? (extract 1-line description if so)
- Has `.github/agents/` already? Which agent files exist?

Classify each folder as:
- **CANDIDATE-REPO** — has `.git/` AND has a build file. Likely a real sub-repo to include.
- **POSSIBLE-REPO** — has `.git/` but no recognized build file. Could be docs, scripts, infra repo.
- **NOT-REPO** — no `.git/`. Likely personal notes, archives, scratch, temp folders.
- **WORKSPACE-CONFIG** — `.github/`, `data/`, `WIP/` already present at root (meta itself).

## 1.3 Output (chat, no files yet)
Print a table:
```
WORKSPACE SCAN — <absolute path>

| Folder                | Type            | .git | Build file        | Has CTO agent? |
|-----------------------|-----------------|------|-------------------|----------------|
| service-orders        | CANDIDATE-REPO  | ✓    | pom.xml           | v4 (cto.agent.md) |
| service-payments      | CANDIDATE-REPO  | ✓    | build.gradle      | v3 (4 helpers) |
| service-notifications | CANDIDATE-REPO  | ✓    | pom.xml           | none |
| shared-lib            | CANDIDATE-REPO  | ✓    | pom.xml           | none |
| docs-repo             | POSSIBLE-REPO   | ✓    | (none)            | n/a |
| personal-notes        | NOT-REPO        | ✗    | (none)            | n/a |
| .archive              | NOT-REPO        | ✗    | (none)            | n/a |
| tmp                   | NOT-REPO        | ✗    | (none)            | n/a |
```

---

# PHASE 2 — INTERACTIVE WHITELIST

Print:
```
Pick sub-repos to include in GFT workspace scope.

Default recommendation (CANDIDATE-REPO type, all selected):
- [x] service-orders
- [x] service-payments
- [x] service-notifications
- [x] shared-lib

Optional (POSSIBLE-REPO — usually skip unless you actively work on them):
- [ ] docs-repo

NOT-REPO folders are automatically excluded.

Reply with:
- `default` → use recommendation above
- `+<folder>` → add a POSSIBLE-REPO to scope (e.g. `+docs-repo`)
- `-<folder>` → remove a CANDIDATE-REPO from scope (e.g. `-shared-lib`)
- `none` → empty whitelist (you'll add manually later)
- Or comma-separated list: `service-orders, service-payments, shared-lib`

STOP, wait for reply.
```

After user reply: parse, confirm final whitelist with user once more:
```
Confirmed scope:
- <folder 1>
- <folder 2>
- ...

Reply `go` to generate files, or revise.
```

STOP. Wait for `go`.

---

# PHASE 3 — GENERATE WORKSPACE SCAFFOLD

## 3.1 `<workspace-name>.code-workspace` (VS Code multi-root config)

Workspace name: ask user once, or default to parent folder name (e.g. `gft-platform.code-workspace`).

```json
{
  "folders": [
    { "path": "." },
    { "path": "service-orders" },
    { "path": "service-payments" },
    { "path": "service-notifications" },
    { "path": "shared-lib" }
  ],
  "settings": {
    "files.exclude": {
      "personal-notes": true,
      ".archive": true,
      "tmp": true
    }
  }
}
```

The `.` entry exposes workspace root itself (so TechLead, `.github/`, `data/` are visible). Each whitelisted sub-repo gets its own root entry (so VS Code treats each as a separate workspace folder — better for per-folder Copilot context).

The `files.exclude` hides NOT-REPO folders from explorer (cleaner UI; they still exist on disk).

## 3.2 `.copilot-workspace-config.yml` (explicit whitelist for TechLead)

```yaml
# Copilot Workspace Configuration
# TechLead reads this file at Protocol Zero to know which sub-repos are in scope.
# Folders NOT in `repos` list are ignored — TechLead will not scan them or include them in maps.

workspace_name: <workspace-name>
workspace_root: <absolute path of workspace root>

# Repos in scope — TechLead operates on these
repos:
  - name: service-orders
    path: ./service-orders
    role: <auto-detected from README or "TBD">
    has_cto_agent: true  # v4 detected
  - name: service-payments
    path: ./service-payments
    role: TBD
    has_cto_agent: true  # v3 detected — recommend upgrade to v4
  - name: service-notifications
    path: ./service-notifications
    role: TBD
    has_cto_agent: false  # bootstrap recommended
  - name: shared-lib
    path: ./shared-lib
    role: shared library
    has_cto_agent: false  # bootstrap recommended (lower priority — library)

# Folders explicitly excluded (for reference; TechLead ignores them)
excluded:
  - personal-notes
  - .archive
  - tmp
  - docs-repo  # POSSIBLE-REPO that user chose to exclude

# Updated by user when context changes
last_updated: 2026-05-27
```

The `role:` field — fill from `README.md` first line if there's a clear one-liner; else mark "TBD" and prompt user to fill in.

The `has_cto_agent:` field — set based on Phase 1.2 inventory.

## 3.3 `data/tech-radar.md` (template)

```markdown
# Tech Radar — <workspace-name>

Allowed / avoided / planned technologies across this ecosystem. TechLead consults this when proposing changes — won't suggest avoided technologies, will prefer allowed ones.

Format: each entry is one line `- <technology> — <one-line reason>`. Keep tight.

## ADOPT (proven, default choice)
- Java 21 + Spring Boot 3.x — primary application stack
- Apache Kafka — async messaging
- Kinetica — analytic database for high-throughput queries
- OpenShift — container platform (client-managed)
- Jenkins — CI (client standard)
- JUnit 5 + Testcontainers — test stack
<!-- Add more -->

## TRIAL (in use, evaluating wider adoption)
- gRPC — for low-latency inter-service calls (currently in 2 services)
- OpenTelemetry — observability standard (rolling out)
<!-- Add more -->

## ASSESS (interesting, not used yet)
- Quarkus — alternative to Spring Boot for low-memory services
<!-- Add more -->

## HOLD (avoid in new code; existing usages OK until replaced)
- JPA-Hibernate — replaced by jOOQ for new persistence code (legacy services still use it)
- RabbitMQ — replaced by Kafka (one legacy service still on RabbitMQ, migrating)
<!-- Add more -->

## RETIRED (do not use)
- Spring MVC sync controllers for high-traffic endpoints — use WebFlux
<!-- Add more -->
```

User fills this in manually after init. It's a living document — update when ecosystem-level tech decisions change.

## 3.4 `data/architecture-decisions.md` (template)

```markdown
# Architecture Decisions — <workspace-name>

Ecosystem-level Architecture Decision Records (ADRs). Per-service ADRs stay in each sub-repo (e.g., `<repo>/docs/adr/`).

Format: each ADR is short — Title, Context, Decision, Consequences, Status. Reverse-chronological (newest on top).

---

## ADR-NNN — <title>

**Status:** ACCEPTED | DEPRECATED | SUPERSEDED-BY-ADR-XXX
**Date:** YYYY-MM-DD
**Decided by:** <person / team>

### Context
<what problem we faced, what alternatives we considered>

### Decision
<what we chose>

### Consequences
- Positive: <gains>
- Negative: <costs / constraints>
- Neutral: <observations>

---

<!-- Add new ADRs above this line -->

<!-- Example skeleton — delete after first real ADR -->
## ADR-001 — Use Kafka over RabbitMQ for new async flows
**Status:** ACCEPTED
**Date:** 2025-06-15
**Decided by:** platform team

### Context
RabbitMQ existing infra has limits in throughput for analytics events. Considered: keep RabbitMQ, switch to Kafka, switch to AWS SNS+SQS.

### Decision
Use Kafka for all new async flows. Existing RabbitMQ services remain until they have unrelated reasons to refactor.

### Consequences
- Positive: better throughput, replay capability, alignment with industry standard
- Negative: ops overhead (Kafka cluster), longer learning curve for new joiners
- Neutral: dual-stack period (Kafka + RabbitMQ) for ~12 months
```

User can delete the example ADR or replace with real ones over time.

In **interactive** mode: write each file, show preview, ask `OK? (y/n/edit)`. In **autonomous**: write all four.

---

# PHASE 4 — AUDIT REPORT

After writing scaffold, print:

```
WORKSPACE INIT REPORT

Workspace name: <name>
Workspace root: <absolute path>
Sub-repos in scope: <count>
Files generated:
  ✓ <name>.code-workspace
  ✓ .copilot-workspace-config.yml
  ✓ data/tech-radar.md (template — fill in manually)
  ✓ data/architecture-decisions.md (template — fill when first ADR happens)

PER SUB-REPO STATUS

| Sub-repo              | CTO setup     | Action recommended |
|-----------------------|---------------|--------------------|
| service-orders        | v4 (current)  | none — ready |
| service-payments      | v3 (4 helpers)| upgrade to v4 (optional, see commands below) |
| service-notifications | none          | bootstrap v4 (recommended) |
| shared-lib            | none          | bootstrap v4 (low priority — library, less Copilot use) |

NEXT ACTIONS (copy-paste into terminal / IntelliJ as you go)

1) BOOTSTRAP missing CTOs:
   For service-notifications:
     - Open service-notifications/ in IntelliJ
     - Copilot Chat → Agent mode → Opus 4.6
     - Paste copilot_jetbrains_bootstrap_v4.md
     - Follow phases 1-6, restart IDE, smoke test
   
   For shared-lib (optional — only if you actively edit it):
     - Same procedure

2) UPGRADE older CTOs (optional, do when convenient):
   For service-payments (currently v3):
     - Open service-payments/ in IntelliJ
     - Copilot Chat → Agent mode → Opus 4.6
     - Paste copilot_jetbrains_upgrade_v3_to_v4.md
     - Pick mode `2` autonomous if you trust the conversion

3) FILL workspace context (when you have time):
   - Edit data/tech-radar.md → list real Allowed/Trial/Assess/Hold/Retired technologies
   - Edit data/architecture-decisions.md → add first real ADR (or delete example)
   TechLead will work without these filled, but quality of cross-service planning improves with them.

4) GENERATE TechLead (last step):
   - In THIS VS Code workspace
   - Copilot Chat → Agent mode → Opus 4.6
   - Paste copilot_vscode_multirepo_techlead.md
   - Follow phases — TechLead will read .copilot-workspace-config.yml + data/*.md + sub-repos in whitelist
   - Reload VS Code window
   - Smoke test: agents dropdown → TechLead → "Daj mi mapę serwisów" (Polish)

5) DAILY WORKFLOW:
   - VS Code (this workspace) + TechLead → big picture, audits, cross-service planning
   - IntelliJ per repo + CTO → deep coding work in a single service
   - TechLead can "Drill into specific service" → handoff to that repo's CTO (in VS Code multi-root)
   - For longer Java sessions: open repo separately in IntelliJ (better Java tooling)

ESTIMATED TIME TO FULL SETUP
- Bootstrap missing CTOs: ~10 min per sub-repo (Phases 1-6 with restarts)
- Fill data/ context: 30-60 min one-time (or grow organically)
- Generate TechLead: ~10 min
Total: depends on number of sub-repos, but ~1-2h for typical 4-repo ecosystem.
```

---

# PHASE 5 — CONSTRAINTS REMINDER + STOP

Print:

```
WORKSPACE INIT DONE.

What this prompt did NOT do:
- Did not modify any sub-repo (intentional — each sub-repo owns its config)
- Did not run bootstrap_v4 for missing CTOs (you do this in IntelliJ per repo)
- Did not generate TechLead agent (separate step — paste copilot_vscode_multirepo_techlead.md)
- Did not fill data/ files (those are templates for you to edit)

What this prompt set up:
- VS Code multi-root workspace configured (.code-workspace)
- Explicit whitelist of sub-repos in scope (.copilot-workspace-config.yml)
- Skeleton data/ context for TechLead to consume
- Audit report with next actions

When ecosystem changes (new sub-repo, removed sub-repo, ownership change):
- Re-run THIS prompt (it's idempotent — handles existing config)
- OR manually edit .copilot-workspace-config.yml + .code-workspace folders list

Final note: when committing to git, decide per-file:
- .code-workspace — usually commit (team uses same workspace setup)
- .copilot-workspace-config.yml — usually commit (team needs same whitelist)
- data/tech-radar.md, data/architecture-decisions.md — commit if shared team artifact, gitignore if personal notes
```

---

# CONSTRAINTS (all phases)
- Read scan only — no installs, no network calls.
- Write only the 4 specified files at workspace root. NO edits to sub-repos.
- If a target file already exists (re-running on initialized workspace) — read current content, propose merged version, ask user "overwrite or merge?". Don't silently overwrite.
- Idempotent: running this twice on the same workspace should NOT break anything.
- Never print secrets. Redact if any sub-repo path / file content contains apparent secrets.
- All generated files in English.
- Be terse in chat. Tables > prose.
- If sub-repo has unusual structure (no clear build file but obviously a real repo) — flag it in Phase 1.3 as POSSIBLE-REPO and let user decide in Phase 2.
````
