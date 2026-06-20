<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# launch-your-agent

A [Claude Code](https://code.claude.com) skill that helps a technical founder build whatever they want — an internal worker, a dev-loop agent, a piece of their product, a customer-facing agent — and run it on **EITHER** of two targets:

- **Claude Code (Max subscription)** — runs on your own machine under your Claude Code Max plan via OAuth. **No API key, no per-run cost.** Built as a Claude Code subagent, run+graded in-session via the Agent tool, scheduled with macOS `launchd` (or a cloud `/schedule` routine). Best for your own chores, dev loops, anything that uses your existing logins/MCP servers or local files.
- **Claude Managed Agents (CMA)** — hosted in Anthropic's cloud, isolated sandbox, metered API key, per-user vaults, cloud scheduled deployments. Best for a piece of your product, customer-facing/multi-tenant agents, anything that must run server-side without your laptop.

The skill **asks which target fits**, then interviews you, scopes a v0, builds it, grades it against your own definition of done, iterates, and (if it should run on a clock) schedules it — with everything bigger laid out as an explicit v1/v2 plan.

> **Krypteia fork — enhanced by [threatseeker](https://github.com/threatseeker) of [krypteiasec.com](https://krypteiasec.com).** Upgraded from the upstream [anthropics/launch-your-agent](https://github.com/anthropics/launch-your-agent) reference implementation (CMA-only) to add the **Claude Code (Max subscription)** build target — build and run agents on your own Claude subscription with **no API key and no per-run cost** — plus the build-target fork that asks which target to use. Verified end-to-end on a live Max plan. Licensed under [Apache 2.0](./LICENSE).

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

## How to use — with or without a Max subscription

The skill works two ways. Pick by what you have and where the agent should live — the skill **asks you at the start**, you decide.

| | **With a Claude subscription** (Max / Pro / Team) | **Without a subscription** (API key only) |
|---|---|---|
| Build target | **Claude Code (Max)** | **Claude Managed Agents (CMA)** |
| Auth | your Claude Code login (OAuth) — `claude auth status` shows `claude.ai` | your own Anthropic API key in a local `.env` |
| Cost | none beyond your subscription — **no per-run charge** | metered, cents per run |
| Where it runs | **your machine**, as you (your files, your already-logged-in MCP servers) | **Anthropic's cloud**, isolated sandbox |
| Scheduling | macOS `launchd` (or a cloud `/schedule` routine) | cloud scheduled deployment |
| Best for | your own chores, dev loops, anything local or that reuses your logins | a piece of your product, customer-facing / multi-tenant, server-side without your laptop |

You can pick the CMA track **even if you do have a subscription** — choose it when the agent must run in the cloud without your machine on, or serve many users.

### WITH a Max subscription — no API key, no per-run cost

1. Confirm you're signed in: `claude auth status` should show `"authMethod": "claude.ai"`.
2. Run `/launch-your-agent` and choose **Claude Code (Max)** when it asks where the agent should run.
3. The skill interviews you, then writes a `my-agent/` folder: the subagent (`agent.md`), the rubric (`outcome.md`), a headless runner (`run.sh`), a grader (`judge.sh`), an eval scaffold, and — if the task recurs — a `launchd` plist.
4. It runs and grades a first version **right in your session** (via the Agent tool — no API key, nothing to pay), then iterates with you.
5. Install + schedule it (the skill writes an `INSTALL.md`): copy the subagent into `~/.claude/agents/`, run `./run.sh && ./judge.sh` once by hand, then load the schedule:
   ```bash
   launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.launch-your-agent.<slug>.plist
   ```
   Laptop-closed / always-on instead? Use the cloud `/schedule` routine.

> **How it stays on your subscription:** the generated `run.sh` leaves `ANTHROPIC_API_KEY` unset, so `claude -p` uses your keychain OAuth login (never `--bare`, which would force an API key). If a scheduled run ever 401s, your token expired — run `claude auth login` (or `claude setup-token` for a long-lived unattended token).

### WITHOUT a subscription — CMA, bring your own API key

1. Create an API key for your own account at platform.claude.com → API keys (note the workspace).
2. Run `/launch-your-agent` and choose **Claude Managed Agents (CMA)**.
3. The skill stages the whole build with no key needed, then asks once for the key — it lands in a local `my-agent/.env` (chmod 600), never in the chat.
4. It launches the agent in **your Console**, watches the graded run, iterates, and — if recurring — creates a cloud scheduled deployment. Runs cost cents.

Either track: when you're done, run `/wrap-up` to regenerate the overview page, recap what you built, and get the next 1–2 upgrades.

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

## Credits

Originally [anthropics/launch-your-agent](https://github.com/anthropics/launch-your-agent) (Apache 2.0). The **Claude Code (Max subscription)** build target, the build-target fork, and the dual-track docs were added by **[threatseeker](https://github.com/threatseeker)** of **[krypteiasec.com](https://krypteiasec.com)**. The Max track was verified end-to-end on a live Max plan: a generated `run.sh` produced a real deliverable on subscription OAuth (no API key) and the `judge.sh` grader returned a structured `pass` verdict.
