# Plan.md

## Purpose

This file turns the project specification into milestone-sized implementation steps with explicit validations.

Rules:

- Complete milestones in order unless a small dependency inversion is clearly better.
- Keep each milestone small enough to complete and validate in one focused loop.
- If validation fails, stop and fix before moving on.
- Update `Documentation.md` after each milestone.
- Do not widen scope without recording a decision.

## Global validation commands

Use these commands when they become available:

```bash
uv run ruff check .
uv run pytest -q
uv run research-os doctor
```

If a command does not exist yet, create it as part of the relevant milestone.

---

## Milestone 0 — Repo scaffold and engineering baseline

### Goal
Create the base Python project, package structure, config skeleton, test harness, linting, and top-level docs placeholders.

### Required outputs
- `pyproject.toml`
- installable package under `src/research_os/`
- CLI entry point
- `tests/` skeleton
- `README.md`, `SECURITY.md`, `THIRD_PARTY.md`, `.env.example`
- `docs/` skeleton

### Acceptance criteria
- `uv sync` succeeds
- `uv run pytest -q` succeeds with at least one smoke test
- `uv run research-os --help` succeeds
- `uv run ruff check .` succeeds

### Validation
```bash
uv sync
uv run research-os --help
uv run pytest -q
uv run ruff check .
```

---

## Milestone 1 — Reference catalog and legal boundary

### Goal
Create a clean `references/` area and document how external repos are used.

### Required outputs
- `references/README.md`
- clear read-only policy
- `THIRD_PARTY.md` describing references vs vendored code
- helper to inspect/reference external repos without coupling runtime logic to them

### Acceptance criteria
- references are segregated from runtime code
- docs clearly state that restricted-license repos are reference-only unless explicitly accepted
- runtime tests do not import from `references/`

### Validation
```bash
uv run pytest -q tests/test_references_boundary.py
```

---

## Milestone 2 — Config, paths, sandbox, and SQLite state

### Goal
Implement the config system, path policy, app directories, SQLite schema, and event/task models.

### Required outputs
- config loader
- sandbox root allowlist support
- SQLite schema for runs/tasks/events/decisions/jobs
- state repository layer
- `research-os doctor` reports paths and config health

### Acceptance criteria
- app creates `.research-os/app.db`
- app can create a run record and list it
- sandbox validation rejects forbidden paths

### Validation
```bash
uv run research-os doctor
uv run pytest -q tests/state tests/config tests/paths
```

---

## Milestone 3 — Run control plane and four-file stack

### Goal
Implement run creation and durable file generation.

### Required outputs
- `research-os run create`
- per-run directory generation under `runs/<run-id>/`
- automatic creation of `Prompt.md`, `Plan.md`, `Implement.md`, `Documentation.md`
- event log for run creation

### Acceptance criteria
- creating a run materializes the expected directory structure
- run metadata is persisted in SQLite
- generated files are templated and editable

### Validation
```bash
uv run research-os run create --name smoke
uv run pytest -q tests/runs
```

---

## Milestone 4 — Provider abstraction and provider registry

### Goal
Build a common provider interface and registry without implementing all providers yet.

### Required outputs
- provider base classes/protocols
- role-based routing config
- provider registry and diagnostics
- `research-os provider list`

### Acceptance criteria
- providers can be registered and resolved by role
- config can declare different providers per agent role
- doctor/provider listing reflects installed/configured providers

### Validation
```bash
uv run research-os provider list
uv run pytest -q tests/providers/test_registry.py
```

---

## Milestone 5 — OpenAI Codex local adapter + OpenAI API adapter

### Goal
Support OpenAI-backed personal and lab paths.

### Required outputs
- local Codex adapter that shells out to a user-managed local Codex CLI/tool flow
- OpenAI API adapter
- provider docs in `docs/auth_openai.md`

### Acceptance criteria
- dry-run or mock tests for both adapters pass
- doctor identifies available auth mode and missing requirements
- adapter interface supports prompt, tool instructions, and artifact-aware execution metadata

### Validation
```bash
uv run pytest -q tests/providers/test_openai_codex_cli.py tests/providers/test_openai_api.py
uv run research-os doctor
```

---

## Milestone 6 — Gemini local adapter + Gemini API adapter

### Goal
Support Gemini-backed personal and lab paths.

### Required outputs
- Gemini CLI adapter
- Gemini API adapter
- `docs/auth_gemini.md`

### Acceptance criteria
- doctor can detect Gemini CLI availability
- mock tests cover CLI and API modes
- role-based routing can choose Gemini for selected agents

### Validation
```bash
uv run pytest -q tests/providers/test_gemini_cli.py tests/providers/test_gemini_api.py
uv run research-os provider list
```

---

## Milestone 7 — Optional Claude Code local passthrough adapter

### Goal
Implement a clearly labeled experimental adapter for a locally installed, user-authenticated Claude Code CLI.

### Required outputs
- `claude_code_cli_passthrough` adapter
- explicit config flag to enable it
- warning text in CLI/docs
- `docs/auth_claude_code_passthrough.md`

### Acceptance criteria
- disabled by default
- runtime refuses to proxy credentials
- doctor clearly explains the local-only passthrough model

### Validation
```bash
uv run pytest -q tests/providers/test_claude_passthrough.py
uv run research-os doctor
```

---

## Milestone 8 — Benchmark-pack schema and importer

### Goal
Create the benchmark-pack format and import pipeline.

### Required outputs
- manifest schema
- importer for raw folder inputs
- benchmark registry
- docs for benchmark packs
- at least one toy/sample benchmark

### Acceptance criteria
- importer can normalize a sample raw benchmark folder
- benchmark registry can list imported benchmarks
- validation and scoring manifest parsing works

### Validation
```bash
uv run research-os benchmark import benchmarks/sample_toy
uv run pytest -q tests/benchmark_pack
```

---

## Milestone 9 — Core agent roles and task orchestration

### Goal
Implement the core orchestration loop and role-specific task handlers.

### Required outputs
- Director, PM/Triage, Planner, Builder, Runner, Analyst, Writer, Reviewer, Concierge classes
- task queue in SQLite
- event log and decision log
- simple scheduler for sequential/parallel task dispatch on one host

### Acceptance criteria
- a run can progress through at least one full cycle:
  create tasks → execute builder/runner/analyst → update docs/state
- task retries and failure states are persisted

### Validation
```bash
uv run pytest -q tests/agents tests/orchestrator tests/state
uv run research-os run start <sample-run-id> --dry-run
```

---

## Milestone 10 — Builder/Runner/Analyst verified loop

### Goal
Make the system do real milestone execution with validation gates.

### Required outputs
- scoped file editing support
- validation command execution
- metric extraction and comparison
- best-so-far tracking
- stop-and-fix behavior on validation failure

### Acceptance criteria
- the system refuses to proceed when validation fails
- best metrics and failed validations are written to run state and `Documentation.md`
- allowed-edit-path enforcement works

### Validation
```bash
uv run pytest -q tests/execution tests/evaluator tests/runner
uv run research-os run start <sample-run-id>
```

---

## Milestone 11 — Writer/Reviewer and artifact packaging

### Goal
Generate usable reports/summaries and a reviewer pass.

### Required outputs
- report template(s)
- reviewer checklist/rubric
- artifact manifest
- summary generation for operators

### Acceptance criteria
- each finished run produces a structured summary
- reviewer output references concrete artifacts/metrics
- required artifacts are checked before marking milestone complete

### Validation
```bash
uv run pytest -q tests/writer tests/reviewer
uv run research-os run status <sample-run-id>
```

---

## Milestone 12 — Slack and Discord channels

### Goal
Add operator-facing messaging adapters.

### Required outputs
- Slack channel adapter
- Discord channel adapter
- channel docs
- test command(s)

### Acceptance criteria
- inbound operator message can be converted into a queued task
- outbound status summary can be posted
- secrets are loaded from env/config only

### Validation
```bash
uv run pytest -q tests/channels
uv run research-os channel slack test
uv run research-os channel discord test
```

---

## Milestone 13 — Slurm runner and quota policy

### Goal
Support scheduler-backed execution with explicit resource policy.

### Required outputs
- Slurm job spec renderer
- `sbatch` submitter
- quota policy module
- job watcher/update path
- cluster docs

### Acceptance criteria
- job submission is blocked if policy caps are exceeded
- job IDs and resource requests are persisted
- the system can resume after a job transitions through queue/running/completed/failed

### Validation
```bash
uv run pytest -q tests/runner/test_slurm.py tests/policy
uv run research-os doctor
```

---

## Milestone 14 — Example benchmark and end-to-end demo run

### Goal
Show the whole stack working on at least one bounded benchmark.

### Required outputs
- an example benchmark included in repo
- an example end-to-end run walkthrough in docs
- screenshot/log snippets for README

### Acceptance criteria
- a fresh user can reproduce the example with documented steps
- the example exercises run creation, provider selection, validation, state updates, and artifact generation

### Validation
```bash
uv run research-os benchmark list
uv run research-os run create --benchmark sample_toy
uv run research-os run start <sample-run-id>
```

---

## Milestone 15 — Publishability pass

### Goal
Polish the repo for public release.

### Required outputs
- README rewritten for external users
- installation and quickstart validated from scratch
- `.env.example` completed
- `docs/operator_guide.md` completed
- `SECURITY.md` and `THIRD_PARTY.md` finalized

### Acceptance criteria
- clean onboarding path exists for personal mode and lab mode
- docs mention Slack, Discord, provider setup, sandboxing, and benchmark import
- project can be pushed to GitHub with minimal embarrassment

### Validation
```bash
uv run pytest -q
uv run ruff check .
uv run research-os doctor
```

---

## Stretch milestones (only after MVP)

- Redis-backed optional coordination
- Anthropic API adapter
- richer reviewer rubrics
- multiple concurrent benchmark runs
- advanced compaction/summarization layer
- optional web UI
- benchmark result dashboards

## Stop-and-fix rule

If any milestone validation fails:

1. do not move on,
2. fix the failure,
3. record what was wrong and what changed in `Documentation.md`,
4. re-run the milestone validation,
5. only continue when the milestone passes.

## Decision logging rule

When design ambiguity appears, prefer:

1. simpler runtime,
2. stronger documentation,
3. safer default behavior,
4. explicit opt-in for risky/flexible features,
5. pluggable abstractions over one-off hacks.
