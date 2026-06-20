<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# launch-your-agent

A [Claude Code](https://code.claude.com) skill that helps a technical founder build whatever they want — an internal worker, a dev-loop agent, a piece of their product, a customer-facing agent — and run it on **EITHER** of two targets:

- **Claude Code (Max subscription)** — runs on your own machine under your Claude Code Max plan via OAuth. **No API key, no per-run cost.** Built as a Claude Code subagent, run+graded in-session via the Agent tool, scheduled with macOS `launchd` (or a cloud `/schedule` routine). Best for your own chores, dev loops, anything that uses your existing logins/MCP servers or local files.
- **Claude Managed Agents (CMA)** — hosted in Anthropic's cloud, isolated sandbox, metered API key, per-user vaults, cloud scheduled deployments. Best for a piece of your product, customer-facing/multi-tenant agents, anything that must run server-side without your laptop.

The skill **asks which target fits**, then interviews you, scopes a v0, builds it, grades it against your own definition of done, iterates, and (if it should run on a clock) schedules it — with everything bigger laid out as an explicit v1/v2 plan.

> **Krypteia fork.** Upgraded from the upstream [anthropics/launch-your-agent](https://github.com/anthropics/launch-your-agent) reference implementation (CMA-only) to add the Claude Code (Max) build target and the build-target fork. Licensed under [Apache 2.0](./LICENSE).

## Quickstart

```bash
git clone <this-repo>
cd <this-repo>
claude
```

Then type:

```
/launch-your-agent
```

The skills in `.claude/skills/` are picked up automatically when you run Claude Code inside this folder — nothing to install.

When you're done (or any time later), `/wrap-up` regenerates the overview page, recaps every primitive you now own, and suggests the next 1–2 upgrades.

## What you need

- Claude Code installed and signed in.
- **Claude Code (Max) track:** nothing else — just a Max (or Pro/Team) login (`claude auth status` should show `claude.ai`). No API key, no per-run cost; the agent runs on your subscription via keychain OAuth.
- **CMA track:** an Anthropic API key for **your own** account (you'll create it during the flow at platform.claude.com → API keys; it goes into a local `.env` file, never into the chat). Runs cost cents.

## What you walk away with

- **Claude Code (Max):** a subagent in `~/.claude/agents/`, a `my-agent/` folder (the build sheet, `agent.md`, a headless `run.sh`, a `judge.sh` grader, an eval scaffold, a `launchd` plist, an overview page, NEXT-DIRECTIONS.md) — a graded agent you can run on demand or on a schedule, with no API key.
- **CMA:** a live managed agent in your Console (agent + environment + graded run, plus a scheduled deployment if your task recurs), and a `my-agent/` folder with the build sheet, the exact API payloads, a resumable launch script, an eval scaffold, an overview page, and NEXT-DIRECTIONS.md.

## Repo layout

| Path | What it is |
|---|---|
| `.claude/skills/launch-your-agent/` | The main skill: pick a build target, then interview → build → grade & iterate → run without you. References: shared interview mapping, verified CMA API shapes (`cma-api.md`), the Claude Code (Max) mechanics (`max-build.md`), copy-ready Max templates (`max-templates.md`, incl. `agent.md`/`run.sh`/`judge.sh`/launchd plist), examples bank, dual-track overview template, and a build sheet per track |
| `.claude/skills/wrap-up/` | Companion skill: explicit close-out / status check for a built agent (dual-track — refreshes Max state locally with no API key, or CMA state via the API) |
| `cma-primitives.md` | Inventory of CMA primitives and limits, from the public docs |
| `interview-to-config.md` | Background: how interview answers map to CMA primitives |
| `examples-bank.md` | Sourced example agents and production proof points |
| `ui/` | Example overview page + build sheet |

The CMA documentation is the source of truth for the API: https://platform.claude.com/docs/en/managed-agents/overview
