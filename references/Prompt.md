# Prompt.md

## Project name

`research-os` (working title)

## Mission

Build a **publishable, ready-to-clone multi-agent research operating system** for long-horizon AI-assisted research and course-project execution.

The system must be designed so that a user can:

1. clone the repo on a workstation or school cluster,
2. connect one or more model providers using either subscription-backed local tools or API-backed lab mode,
3. import a bounded project spec or benchmark pack,
4. let the system run for hours to days with resumable state,
5. inspect progress through files, CLI, Slack, or Discord,
6. and obtain reproducible artifacts: code changes, run logs, metrics, plots, a report/paper draft, and reviewer notes.

This project is **quality-first**, not volume-first.

## Product thesis

The product is not “another chat wrapper.”
It is a **research OS** with:

- durable project memory,
- milestone-based execution,
- bounded benchmark packs,
- agent orchestration,
- experiment search with verification,
- resumable local/cluster execution,
- and GitHub-ready documentation.

The design is inspired by:

- Codex long-horizon file-driven execution,
- AI Scientist v1/v2 idea → experiment → writeup patterns,
- FARS’s stage decomposition,
- autoresearch’s minimal program-and-score loop,
- OpenClaw / Sleepless / Ralph patterns for persistence, resumability, and operator messaging.

These external projects are **references**, not the codebase we are building. Treat them as design inputs.

## Core principles

1. **File-first, not chat-first**
   - Every run has durable markdown files and artifacts.
   - Agents communicate through state, tasks, logs, and artifacts, not a single giant transcript.

2. **Quality-first, not paper-count-first**
   - The system should optimize for reproducibility, evaluator quality, and clear improvement over baselines.

3. **Benchmark-pack-first**
   - The system must be able to ingest bounded project specs (course projects, controlled research tasks, benchmark packs).
   - It must not assume every task is open-world.

4. **Dual access mode**
   - Personal / subscription mode for local use.
   - Lab / API mode for funded or higher-scale use.

5. **Safe by construction, flexible by configuration**
   - Default behavior should be conservative.
   - Operators may expand sandbox paths, providers, and quotas explicitly.

6. **Resumable by default**
   - Interruptions must not destroy progress.
   - The system should be able to resume after termination or cluster job preemption.

7. **Publishable repo quality**
   - Someone who lands on the GitHub repo should be able to understand setup, auth, channels, benchmark format, and how to run the system.

## User stories

### User story A — personal/subscription user
A student clones the repo on a school server, signs into Codex or Gemini locally, imports a project spec, and launches a long-horizon run without needing API billing on day one.

### User story B — lab/API user
A research lab configures provider API keys or cloud credentials, runs longer experiments on a scheduler, and collects full run traces, metrics, and artifacts.

### User story C — operator
An operator monitors status via CLI/Slack/Discord, can inspect current milestone, pause/resume work, redirect the system, and approve certain actions.

## Required operating modes

### 1) Personal / subscription mode
Support user-managed local model tools when installed and authenticated by the user:

- OpenAI Codex CLI / Codex app / Codex-capable local flow
- Gemini CLI
- Optional local Claude Code passthrough adapter

Important:
- For Claude, this project must **not** implement or proxy Claude.ai consumer login itself.
- If optional Claude Code support is added, it must be **BYO local CLI passthrough** only: detect a locally installed, already authenticated CLI and shell out to it as a user-managed tool.
- Make this adapter opt-in, clearly labeled experimental, and disabled by default.

### 2) Lab / API mode
Support provider adapters for production-style use:

- OpenAI API
- Gemini API / Vertex-compatible mode
- Anthropic API / cloud-provider-compatible mode

API providers must be configured via environment variables or config files.

## High-level architecture

Implement the project as a Python monorepo with the following layers.

### A. Control plane
Responsible for the durable run files and top-level orchestration.

Per run create:

- `Prompt.md`
- `Plan.md`
- `Implement.md`
- `Documentation.md`
- `artifacts/`
- `logs/`
- `metrics/`
- `summaries/`

### B. Agent plane
Implement agents as role-specific workers that read/write shared state and artifacts.

Required roles:

- **Director** — owns the objective, current phase, and run-level decisions.
- **PM/Triage** — translates benchmark specs or user requests into scoped stories and tasks.
- **Librarian** — literature/notes/citation helper.
- **Planner** — transforms the objective into experiments and acceptance criteria.
- **Builder** — edits code and configs in allowed paths only.
- **Runner** — executes commands, tests, experiments, and scheduler jobs.
- **Analyst** — computes metrics, plots, comparisons, and sanity checks.
- **Writer** — generates README/report/paper/update drafts.
- **Reviewer** — red-teams claims, missing baselines, and weak evidence.
- **Concierge** — handles Slack/Discord/CLI operator interactions.

Agent communication must happen through explicit task records, run state, logs, and artifacts.
Do **not** make agent-to-agent communication depend on a monolithic chat transcript.

### C. State plane
Default persistence:

- file system for artifacts and durable memory,
- SQLite for task queue, run metadata, decisions, and event logs.

Optional later:

- Redis pub/sub or Redis-backed task coordination behind a feature flag.

SQLite + files is the default and should be production-usable for a single-host or small-cluster setup.

### D. Provider plane
Abstract model/tool providers behind a common interface.

Providers/adapters to support:

1. `openai_codex_cli`
2. `openai_api`
3. `gemini_cli`
4. `gemini_api`
5. `claude_code_cli_passthrough` (experimental, opt-in)
6. `anthropic_api` (not required in the first minimal slice, but the abstraction should allow it)

Provider routing should be configurable by role.
Example:
- Director/Planner/Reviewer → strongest reasoning provider
- Builder → code-specialized provider or local coding CLI
- Analyst/Writer → cheaper or task-appropriate provider

### E. Runner plane
Support both local and scheduler-backed execution.

Required runners:

- local subprocess runner,
- Slurm `sbatch` runner,
- resumable job watcher.

### F. Channel plane
Support:

- CLI operator interface (mandatory)
- Slack integration (v1)
- Discord integration (v1)

The channel-facing agent is the Concierge. It does not directly do heavy research work. It reads state, posts updates, accepts operator instructions, and can escalate to the Director.

### G. Benchmark plane
Create a benchmark-pack format that can represent:

- current course project folders,
- archived course project folders,
- bounded research tasks,
- toy smoke-test tasks,
- future benchmark packs.

Each benchmark pack must support:

- raw spec input(s),
- normalized machine-readable manifest,
- editable path allowlist,
- validation commands,
- scoring rules,
- artifact expectations,
- report rubric.

## Reference repo policy

External repos will exist under `references/` as read-only inputs.

Expected references:

- AI-Scientist v1
- AI-Scientist v2
- FARS docs/notes
- autoresearch
- OpenClaw
- Sleepless Agent
- Ralph
- course project input folders

Rules:

- Do not modify external repos in place.
- Do not entangle runtime code with reference repos.
- If borrowing ideas from a repo with a non-permissive or custom license, reimplement from scratch.
- If copying code from a permissively licensed project, preserve attribution and update `THIRD_PARTY.md`.
- Maintain a clear legal boundary between references and our runtime.

## Technology constraints

Use:

- Python 3.11+
- `uv` for environment and lockfile management
- typed Python throughout
- Typer or equivalent for CLI
- Pydantic settings/config
- SQLite for default state
- pytest for tests
- Ruff and/or equivalent linting
- simple JSON/YAML/TOML config files

Avoid:

- heavy distributed systems dependency in v1,
- mandatory Redis in v1,
- mandatory Kubernetes in v1,
- a browser dashboard in v1,
- a complex microservice decomposition in v1.

## Sandbox and filesystem model

Default sandbox:
- current repo workspace only.

Operator-expandable sandbox:
- explicit allowlist of additional roots.

Personal Duke-cluster-oriented configuration should support:
- a working directory under the operator’s personal folder,
- a scratch/temporary directory under xtmp or equivalent,
- no access to other user folders.

Implement:

- path allowlists,
- path deny rules,
- read/write policies by role,
- per-run isolated workspace directories.

The system must make sandbox roots explicit in config and in the `doctor` output.

## Cluster/scheduler policy

When configured for the target shared cluster:

- submit jobs with `sbatch` only,
- support per-project resource caps,
- enforce policy before submission,
- log scheduler scripts and job IDs,
- persist job metadata in SQLite.

Initial policy knobs to support:

- maximum 2 nodes of the 4×RTX PRO 6000 class,
- maximum total GPUs per project: 25,
- maximum CPUs per project: 300,
- maximum RAM per project: 5T,
- provider-specific or experiment-specific wall-clock limits.

These values must be configurable.
Do not hardcode school-specific values in a way that blocks general users.

## Required repo structure

Implement this structure or a very close equivalent:

```text
research-os/
  README.md
  LICENSE
  SECURITY.md
  THIRD_PARTY.md
  .env.example
  pyproject.toml
  uv.lock

  docs/
    architecture.md
    quickstart.md
    auth_openai.md
    auth_gemini.md
    auth_anthropic.md
    auth_claude_code_passthrough.md
    channels_slack.md
    channels_discord.md
    benchmark_pack.md
    operator_guide.md

  references/
    README.md
    ai-scientist-v1/
    ai-scientist-v2/
    fars/
    autoresearch/
    openclaw/
    sleepless-agent/
    ralph/
    course-inputs/

  src/research_os/
    __init__.py
    cli.py
    config.py
    paths.py
    logging.py

    orchestrator/
    agents/
    providers/
    channels/
    runner/
    evaluator/
    writer/
    reviewer/
    memory/
    state/
    benchmark_pack/
    templates/

  tests/
  benchmarks/
    sample_toy/
    course_projects/

  runs/
  .research-os/
    app.db
```

## CLI requirements

At minimum implement commands like:

- `research-os doctor`
- `research-os init`
- `research-os benchmark import <path>`
- `research-os run create --benchmark <id>`
- `research-os run start <run-id>`
- `research-os run resume <run-id>`
- `research-os run status <run-id>`
- `research-os run tail <run-id>`
- `research-os task list <run-id>`
- `research-os channel slack test`
- `research-os channel discord test`
- `research-os provider list`

Command names can differ slightly, but the repo must ship a coherent CLI.

## Benchmark-pack requirements

The benchmark importer must handle at least:

1. raw PDF/Markdown/txt project specs,
2. optional dataset notes/URLs,
3. optional local scaffold code,
4. report templates/rubrics,
5. validation/scoring manifests.

Create a normalized benchmark manifest format such as:

```yaml
id: ece685d_2025_project2
name: Hallucination detection and steering with SAE
category: course_project
inputs:
  - type: pdf
    path: raw/spec.pdf
allowed_edit_paths:
  - scaffold/
  - solution/
validation:
  - name: unit_tests
    command: uv run pytest -q
  - name: smoke_run
    command: uv run python scaffold/run_smoke.py
scoring:
  primary_metric: detection_f1
  secondary_metrics:
    - hallucination_rate_after_steering
artifacts:
  - report/report.md
  - report/figures/
```

The exact schema is up to implementation, but it must be simple and documented.

## Verifier-first execution

The system must embody **verifiable search**.
That means it should search over candidate code/experiment/report variants and check them with explicit evaluators.

Required evaluator categories:

1. **Build/health**
   - lint
   - type checks (if configured)
   - unit tests
   - smoke tests

2. **Experiment**
   - baseline runs
   - metric extraction
   - rerun/reproducibility checks
   - regression checks against prior best

3. **Artifact**
   - required files exist
   - figures regenerate
   - reports compile or render

4. **Reviewer**
   - claim/evidence alignment
   - missing baselines checklist
   - novelty/citation TODOs

5. **Policy**
   - sandbox path compliance
   - scheduler quota compliance
   - no unsupported auth flows

## Communication design

Slack and Discord are status/control channels, not the primary memory store.

Requirements:

- inbound messages become queued operator tasks,
- outbound updates summarize current milestone, best metric, failures, and next step,
- operator can ask for status, pause, resume, or redirect,
- all important decisions must still be written to files and SQLite.

## Documentation quality bar

The finished repo must look publishable.
At minimum ship:

- top-level README with architecture diagram and quickstart,
- security model and sandbox explanation,
- per-provider auth docs,
- Slack and Discord setup docs,
- benchmark pack format docs,
- operator guide,
- one example run walkthrough.

## Non-goals for v1

Do **not** build these in the first implementation pass:

- automatic paper submission,
- automatic email/phone/credit-card identity,
- unconstrained browser automation,
- payment execution,
- arbitrary outbound SaaS integrations,
- distributed multi-host cluster orchestration beyond the single-orchestrator + Slurm runner model,
- a rich GUI dashboard.

## Security and auth rules

1. Never implement hidden credential proxying.
2. Never route consumer plan credentials through our own backend.
3. Local CLI passthrough adapters are allowed only when the user installed/authenticated them separately.
4. Prefer API/cloud auth for shared or production deployments.
5. Secrets must come from environment variables or local config files excluded from git.
6. The `doctor` command must identify missing secrets and unsafe config.

## MVP deliverables

The MVP is complete when all of the following exist:

### Core repo
- Python repo scaffold with `uv`, tests, linting, docs skeleton.

### State and control plane
- Run directories and the four markdown control files are generated.
- SQLite stores runs, tasks, events, and decisions.
- Resuming a run works.

### Providers
- OpenAI Codex local adapter works.
- Gemini CLI local adapter works.
- Optional Claude Code local passthrough adapter exists behind an explicit flag and warning.
- OpenAI API adapter exists.
- Provider routing by role exists.

### Benchmark system
- Benchmark pack schema + importer exists.
- Example benchmark pack exists.
- Course-project-style raw input folder can be imported.

### Agent orchestration
- Director, Planner, Builder, Runner, Analyst, Writer, Reviewer, Concierge exist as concrete roles.
- Single-run loop can progress through milestones and update state.

### Runner
- Local runner works.
- Slurm `sbatch` runner exists with quota checks.

### Channels
- Slack adapter works.
- Discord adapter works.

### Docs
- README and setup docs are GitHub-ready.
- Operator guide exists.
- Example `.env.example` exists.

## Done when

The project is done when a fresh user can:

1. clone the repo,
2. install dependencies,
3. run `research-os doctor`,
4. configure either Codex or Gemini sign-in locally,
5. optionally configure Slack/Discord,
6. import a benchmark pack,
7. create a run,
8. start the run,
9. watch the system create/update the four control files,
10. see code changes and validations happen inside an allowed workspace,
11. pause/resume the run,
12. and end with reproducible artifacts plus a status/report summary.

## What success looks like for Codex

Implement a coherent, maintainable v1 that is small enough to understand and strong enough to publish.

Do not chase every feature from every reference repo.
Build the smallest system that credibly demonstrates:

- long-horizon durable execution,
- multi-agent role separation,
- benchmark/task ingestion,
- provider flexibility,
- cluster-aware experimentation,
- operator messaging,
- and publishable documentation.
