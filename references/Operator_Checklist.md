# Operator_Checklist.md

## Your job as operator

You are the human owner of the environment, secrets, reference inputs, and final judgment.

The system can build and run the repo, but you control:

- external repo checkout,
- benchmark input staging,
- provider sign-in / API secrets,
- Slack/Discord credentials,
- cluster paths and quotas,
- and the final go/no-go choices.

## 1) Create the repo and add these files

Create a new repo directory (for example `research-os/`) and place:

- `Prompt.md`
- `Plan.md`
- `Implement.md`
- `Documentation.md`

at the repo root.

## 2) Clone references into `references/`

Create a `references/` directory and clone or copy in read-only references such as:

- AI-Scientist v1
- AI-Scientist v2
- FARS docs/notes snapshot
- autoresearch
- OpenClaw
- Sleepless Agent
- Ralph
- course project input folders

Do not wire these as runtime dependencies.
Keep them as reference material for Codex to inspect.

## 3) Stage your course-project inputs

Copy your current and archived course-project folders into something like:

```text
references/course-inputs/
```

or

```text
benchmarks/course_projects/raw/
```

Include:

- the current graduate course project specs,
- old project folders,
- templates/rubrics if available,
- any non-secret notes about what “full score” looks like.

Do not commit private student solutions unless you intentionally want them in the repo.

## 4) Prepare your local/sandbox directories

Decide your real sandbox roots.
For your personal cluster use, make something like:

```text
/home/<you>/research-os-work/
/xtmp/<you>/research-os/
```

The system should only access the repo plus these explicit roots.

## 5) Install the model tools you want to support

At minimum, install whichever you want to use first.

Recommended order:

1. Codex / OpenAI local flow
2. Gemini CLI
3. optional Claude Code local install

Important for Claude:
- sign into Claude Code yourself if you choose to use it,
- do not expect the project to manage Claude.ai consumer login for you.

## 6) Prepare lab-mode secrets if needed

If you want API mode, prepare environment variables for:

- OpenAI API
- Gemini API / Vertex
- Anthropic API (later if you enable that adapter)
- Slack tokens
- Discord bot token / app credentials

Keep them out of git.

## 7) Decide the first benchmark/demo target

My practical recommendation:

- first public-looking demo: your current Project 2 or the clearest bounded benchmark you know well,
- first smoke test: a tiny toy benchmark or one of the simpler archived projects,
- first “research engine” demo: a benchmark with explicit metrics and repeated experiment search.

## 8) Start Codex with the repo root as working directory

Then give it an instruction like:

```text
Read Prompt.md, Plan.md, Implement.md, and Documentation.md.
Implement the project milestone by milestone.
Start with Milestone 0.
Do not skip validations.
Update Documentation.md after every milestone.
```

## 9) Review the repo after each milestone

After Codex finishes a milestone, you should check:

- did it actually satisfy the milestone?
- do the validation commands pass on your machine?
- did it keep the scope reasonable?
- did it update docs?
- did it avoid muddy coupling with reference repos?

## 10) Only then enable more power

After the core works locally:

- enable Slack/Discord,
- enable cluster/sbatch mode,
- expand sandbox roots,
- allow larger experiments,
- optionally enable the Claude passthrough adapter,
- and only then move toward API-funded or cluster-heavy runs.

## 11) For the cluster specifically

Before letting it submit jobs:

- confirm the Slurm partition/account settings,
- encode your quota preferences,
- test with tiny jobs first,
- confirm workspace and scratch cleanup behavior,
- confirm run resumption after interruption,
- and verify it does not touch paths outside your allowlist.

## 12) Things you should decide early

- repo name (`research-os` is just a working title)
- whether references live as git submodules or normal clones
- whether to ship a toy benchmark in the public repo
- whether course-project raw inputs stay private or get normalized into a shareable benchmark format
- whether to keep Slack/Discord in the initial public release or behind optional extras

## 13) What to do if Codex goes off track

Tell it to:

- re-read the four markdown files,
- narrow to the current milestone,
- stop widening scope,
- and repair failing validations before continuing.

## 14) First clean launch sequence

Suggested sequence:

1. create repo + root markdown files,
2. stage references,
3. install `uv`,
4. install Codex and/or Gemini CLI,
5. run Codex on Milestone 0,
6. verify tests and CLI,
7. continue milestone by milestone,
8. only later enable channels and Slurm.
