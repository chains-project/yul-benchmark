# yul-benchmark

Benchmark for [yul](https://github.com/chains-project/yul), a `PreToolUse` hook that blocks coding agents from writing stale dependency versions.

Each task asks an agent to set up a project or add a dependency.
Every task runs twice: once without yul (`nohook`) and once with yul installed as a hook (`hook`).
The main set has 60 tasks built around popular packages, 10 per ecosystem: PyPI, Maven, npm, Go modules, Cargo and GitHub Actions.
All runs with yul use [yul v0.0.21](https://github.com/chains-project/yul/releases/tag/v0.0.21).

## Results

[`CASES.md`](CASES.md) describes what each run did, task by task, for two agents:

- Claude Code with Claude Opus 5.5, in `benchmark/claude-code-opus/`
- OpenCode with DeepSeek V4.1 Flash, in `benchmark/runs-opencode-deepseek-docker-v3/`

Each agent has three runs per task and condition.

## Layout

| Path | Contents |
| --- | --- |
| `benchmark/cases_top.json` | The 60 top-package tasks |
| `benchmark/cases.json` | Outdated tasks |
| `benchmark/latest_versions_snapshot.json` | Latest release of each target package, used as ground truth |
| `benchmark/run_case.sh` | Runs one task under one condition with Claude Code |
| `benchmark/run_case_opencode.sh` | Same, with OpenCode |
| `benchmark/run_all.sh` | Runs every task under both conditions in parallel |
| `benchmark/run_case_opencode_deepseek.sh` | Runs one task under one condition with OpenCode + DeepSeek, isolated in a container |
| `benchmark/run_all_opencode_deepseek.sh` | Runs every task under both conditions with OpenCode + DeepSeek; resumable |
| `benchmark/run_deepseek_pilot.sh` | Orchestrator used for the earlier DeepSeek pilot |
| `benchmark/Dockerfile.opencode-sandbox`, `benchmark/opencode-sandbox.def` | Container images for the OpenCode runs (Docker / Apptainer) |
| `benchmark/analyze_top_opencode.py` | Ground-truth analysis of OpenCode runs |
| `benchmark/latest_version_oracle.py` | Looks up the latest release of a package on its registry |
| `benchmark/analyze_runs.py` | Classifies each run as stale, already latest or mitigated |
| `benchmark/top_packages.sh` | Fetches the most-used packages per ecosystem from ecosyste.ms |
| `benchmark/Dockerfile` | Image the agents run in, with a toolchain for every ecosystem |
| `benchmark/claude-code-opus/` | Claude Code with Claude Opus 5.5 runs of `cases_top.json` |
| `benchmark/runs-opencode-deepseek-docker-v3/` | OpenCode with DeepSeek V4.1 Flash runs of `cases_top.json` |
| `benchmark/top/` | Outdated runs |
| `benchmark/runs/` | Outdated runs |

Each run directory holds the agent's work directory, its `transcript.jsonl`, and a copy of the final manifest in `final_manifest`.

## Running

### Claude Code

See [`benchmark/README.md`](benchmark/README.md) for details.
In short:

```
docker build -t yul-bench benchmark/
export CLAUDE_CODE_OAUTH_TOKEN=...   # from `claude setup-token`
benchmark/run_all.sh benchmark/cases_top.json <output_dir> [parallelism] [repeats]
python3 benchmark/analyze_runs.py <output_dir>
```

`run_case.sh` mounts the yul binary from a fixed host path (`YUL_BIN`).
Point it at your own yul build before running.

### OpenCode + DeepSeek

Each run executes in its own container with no cache shared between runs, so `nohook` runs never see yul.
With `hook`, yul is loaded through the published [`opencode-yul`](https://www.npmjs.com/package/opencode-yul) plugin (pinned to 0.0.21; override with `OPENCODE_YUL_VERSION`).
Only the `deepseek` entry of your OpenCode auth store (`~/.local/share/opencode/auth.json`, from `opencode auth login`) is copied into the container.

```
docker build -t yul-opencode-sandbox -f benchmark/Dockerfile.opencode-sandbox benchmark/
benchmark/run_all_opencode_deepseek.sh benchmark/cases_top.json <output_dir> [jobs] [repeats] [timeout_seconds]
python3 benchmark/analyze_runs.py <output_dir>
```

The launcher skips runs that already have a `transcript.jsonl`, so rerunning the same command resumes an interrupted batch.
`benchmark/runs-opencode-deepseek-docker-v3/` was produced with `jobs=6 repeats=3 timeout=900`.
