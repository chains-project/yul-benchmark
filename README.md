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

### Open Code

TODO
