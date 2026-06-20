---
name: wrap-up
description: Close out (or revisit) a build from /launch-your-agent — either a Claude Code (Max) agent that runs on your own machine with no API key, or a Claude Managed Agent (CMA) hosted in Anthropic's cloud. Regenerate the overview page, recap every primitive the founder now owns, show the run log and live status, suggest 1–2 tailored next upgrades, and sweep hygiene (Max: no key committed, relative-date kickoffs, subagent installed; CMA: sessions archived, key only in .env, no literal dates in deployment kickoffs). Use when the founder says "/wrap-up", "wrap up", "close it out", "where do things stand with my agent", or at the end of a /launch-your-agent build.
version: 0.3.0
dependencies: A `./my-agent/` folder from /launch-your-agent. Track is read from `build-sheet.json`'s `build_target`. CMA only — ANTHROPIC_API_KEY in `my-agent/.env` is optional; without it the wrap-up is describe-only. Max needs no key at all (the agent runs on the Max plan via OAuth).
---

<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Wrap-up — celebrate it, show what they built, point at what's next

You are closing out (or checking in on) a founder's agent from `/launch-your-agent`. It is built on **one of two tracks** — read `build-sheet.json`'s `build_target` first and let it steer everything below:

- `claude-code-max` → **Claude Code (Max)**: the agent runs on the founder's own machine under their Claude Code Max plan via OAuth — no API key, no per-run cost, scheduled with launchd (or a cloud `/schedule` routine).
- `cma` → **Claude Managed Agents (CMA)**: the agent runs in Anthropic's cloud sandbox under the founder's own `ANTHROPIC_API_KEY`, scheduled as a cloud deployment.

The tone is **celebratory** either way: they just shipped an agent that runs without them — say so, warmly, in one or two sentences ("🎉 you've shipped a Claude Code (Max) agent — here's what you built", or "🎉 you've shipped a managed agent — here's what you built"), then show them what they own and the one or two upgrades worth doing next. End on the overview page, not a wall of text. This is a moment, not a checklist read-out.

Follow the same voice rules as `/launch-your-agent`: warm, compact, tables for anything enumerable, primitives called by their real names with a plain gloss on first use, no unverified timings.

Emoji shorthand — shared both tracks: 🤖 agent · 🎯 outcome · ▶️ run · 🧪 evals · 🧠 memory · 🔌 connector · 📄 skill · 🛠️ tools. Then per track:

- **Claude Code (Max):** 💻 your machine (working dir) · 🗓️ launchd job (or a cloud routine) · ⚖️ judge (the grader). There is **no 🔐 vault** — the agent runs as the founder, reusing their own logins.
- **CMA (unchanged):** 📦 environment (cloud) · 🗓️ deployment (cloud) · 🔐 vault.

**Cost notes — don't volunteer spend figures.** On **Max** it's simpler still: there is **no per-run cost at all** — the agent runs on the Max plan via OAuth with no API billing. On **CMA**, don't volunteer per-run or per-month figures; answer cost questions only if the founder asks.

## 1. Read the state

- Read `./my-agent/build-sheet.json` first and note **`build_target`** — it decides which branch you take. (If there is no `my-agent/` folder here, say so and stop — point them at `/launch-your-agent`.)
- Read the shared artifacts: `NEXT-DIRECTIONS.md`, `outcome.md`, `evals/`, the existing `agent-overview.html`.
- Then refresh live state **per track**:

### If `build_target == "claude-code-max"` — refresh state LOCALLY, no API key

Everything is on the founder's own machine; **never source a key or hit the API**.

- **Subagent installed?** Confirm the subagent file exists at `.claude/agents/<Name>.md` (project) or `~/.claude/agents/<Name>.md` (global) — it's the 🤖 agent. Note its `name`, `model`, and `tools:` allowlist.
- **Last run output.** Read the most recent file in `./outputs/` — that's the latest ▶️ run result.
- **Last judge verdict.** Read the latest ⚖️ judge output (the `judge.sh` JSON: `{verdict, criteria:[{name,pass,evidence}], explanation}`, or whatever the run loop wrote alongside the output). Parse with python (`strict=False`), never jq.
- **Schedule status.** If a `<label>.plist` exists, the agent is on launchd. Check it's loaded and read its state with `launchctl print gui/$(id -u)/com.launch-your-agent.<slug>` (use the founder's own namespace if they chose one). The `state`, `runs`, and `last exit code` lines tell you if it's healthy and whether it has fired. Tail the logs under `~/Library/Logs/<ns>/` (the plist's `StandardOutPath`/`StandardErrorPath`) for the last run. If there's a cloud `/schedule` routine instead, say so and point at the `/schedule` skill rather than launchctl.
- Nothing run yet → wrap what's built; status is "○ Planned", no outputs/verdict to show.

### If `build_target == "cma"` — refresh state via the API (today's behavior, unchanged)

- Read `./my-agent/`: `IDS.env` and the CMA artifacts.
- If `my-agent/.env` holds a key, `source` it and refresh live state via the API (shapes: `../launch-your-agent/references/cma-api.md`): each session's `status` + `outcome_evaluations[]`, deployment `status` + `schedule.upcoming_runs_at`, latest `usage`. Parse with python (`strict=False`), never jq.
- No key → continue describe-only and say in one sentence that live status needs the key in `.env`.
- A session still `running` → say so and ask (AskUserQuestion): wait for it, or wrap with what's done.
- Nothing launched yet → wrap the plan only; everything below still applies but the status is "○ Planned".

## 2. Produce the wrap-up (in this order)

1. **Congratulate them.** One or two warm sentences: name the agent, say what it now does on its own. 🎉 On **Max**: "runs on your machine on the Max plan — no API cost." On **CMA**: "runs in the cloud without you."
2. **Overview page.** Regenerate `agent-overview.html` (template: `../launch-your-agent/references/overview-template.html`), rendering for the agent's track:
   - **Max:** the subagent (name, model, tools), 💻 your machine / working dir, the run log from `./outputs/`, ⚖️ judge verdicts, the 🗓️ launchd schedule (label + next fire, or the cloud routine), and the v1/v2 next directions. No vault card, no Console links — there's nothing hosted; the artifacts are the founder's local files.
   - **CMA:** live IDs, the run log, eval verdicts, the schedule with next run times, Console links for the key's workspace, and the v1/v2 next directions.
   **Open it in their browser** — the page is the closing artifact; the chat below just points at it.
3. **"Here's what you built"** — the primitives recap table, one row per primitive that now exists, mapped to the agent's track per the emoji legend: emoji · what it is (one plain sentence) · what it's set to · where it lives · which card on the page shows it. Then the run log table (run · rubric version · verdict · one-line note).
   - **Max rows:** 🤖 subagent (`.claude/agents/<Name>.md`) · 💻 your machine (working dir) · 🎯 outcome (`outcome.md` rubric) · ⚖️ judge (`judge.sh`) · 🗓️ launchd job (`<label>.plist`, or cloud routine) · 🧠 local memory (`memory/`, if learning) · 🧪 evals. **No 🔐 vault row** — the agent runs as the founder. The "where it lives" column is a path, not a live ID.
   - **CMA rows (unchanged):** one row per CMA primitive with its live ID. Final muted row(s) for primitives deliberately not used → "see NEXT-DIRECTIONS".
4. **"Here's what's next"** — 1–2 extensions picked from the v1/v2 plan that matter most for *this* use case, pitched concretely: what it does for them, what it takes, how small the change is. The rest of the plan stays written down. Standard candidates to weigh alongside whatever the plan already holds: delivery via a connector (Slack/email — Max: gated in the runner / an outbox you approve; CMA: `always_ask`-gated), a 🧠 memory store if runs repeat themselves, wiring a connector for real, and — when it suits how they'll actually use the agent — a **generated interface**: a page or small app Claude Code builds for them in a follow-up session, shaped to the need (a results viewer over the run outputs + judge verdicts, or a simple way to kick off a run, steer it, answer its confirmations).
5. **Hygiene sweep — quiet by default.** Do the sweep for the agent's track and **fix silently what you can**. Only mention something if it's materially relevant — the founder must act, or it changes how they use the agent. No "✅ all good" lists.
   - **Max:** confirm **no `ANTHROPIC_API_KEY` is committed anywhere** (not in `.env`, the plist `EnvironmentVariables`, `run.sh`, or git) — Max is OAuth-only and a stray key means it's silently billing per run; the only `.env` keys allowed are for an MCP server that needs one. Confirm the launchd kickoff prompt (and `run.sh`/`first_prompt.txt`) uses **relative dates** ("today", "the last 7 days as of this run"), never a literal date. Confirm the 🤖 subagent is actually installed in `.claude/agents/`. Sanity-check the `outputs/` and `~/Library/Logs/<ns>/` paths are real and writable. Save a passed run as `evals/case-01/expected.md` if there's no golden case yet.
   - **CMA (unchanged):** archive finished sessions; check the deployment's `initial_events` for literal dates; check the key only ever lived in `.env`; save a passed run as `evals/case-01/expected.md` if there's no golden case yet. A key that touched chat → tell them to rotate it.
6. **Last words** — one short line:
   - **Max:** it all lives on their machine — this folder recreates it (`INSTALL.md` to reinstall the subagent + load the launchd job, `LAUNCH.md` to run it on demand), it runs on the Max plan with no API cost, and they can rerun `/wrap-up` anytime for a fresh picture.
   - **CMA:** everything lives in their Console (platform.claude.com → Claude Managed Agents, in the key's workspace), this folder recreates it anywhere (`LAUNCH.md`), and they can rerun `/wrap-up` whenever they want a fresh picture.

## Notes

- Idempotent: running it again just refreshes the page and tables.
- Don't re-litigate design decisions here — this skill reports and suggests; changes go through `/launch-your-agent` (or a plain conversation) afterwards.
- **Max — never run `claude -p` inline.** `/wrap-up` runs inside a Claude Code session (`CLAUDECODE=1` is set), so spawning a nested `claude` subprocess hangs. Read the existing `./outputs/` and judge JSON that the unattended `run.sh`/`judge.sh` already produced — don't fire a fresh `claude -p` to "get a status." If the founder wants a live demo run during wrap-up, spawn the subagent via the **Agent/Task tool** (native, runs on Max, no nested CLI). `launchctl print` and reading the log files are fine — those aren't nested Claude.
- **Max — a scheduled 401 isn't broken, it's an expired token.** If `launchctl print` shows the last run failed and the log has a 401, the OAuth token lapsed → tell them `claude auth login` to refresh (or `claude setup-token` for a long-lived subscription token). If launchd itself is flaky, the cloud `/schedule` routine is the fallback.
