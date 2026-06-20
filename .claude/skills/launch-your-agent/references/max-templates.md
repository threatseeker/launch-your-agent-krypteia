<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Claude Code (Max) — generated artifacts (copy-ready)

> The concrete files the skill writes into the founder's `./my-agent/` when **`build_target: "claude-code-max"`**.
> Every flag here is a VERIFIED Claude Code flag — never invent one. The Max track runs on the founder's
> machine under their Claude Code Max plan via keychain OAuth: **no `ANTHROPIC_API_KEY`, no per-run billing.**
> Primitive mapping and the per-call mechanics both live in `max-build.md`. This file is
> just the emitted text. Truth order if anything drifts: **the live `claude --help` / `claude -p --help` win.**

Two load-bearing rules that shape every file below:

- **Never run `claude -p` inline during the build session.** Inside Claude Code, `CLAUDECODE=1` is set and a
  nested `claude` subprocess hangs. So in-session work (build, first run, grading) goes through the **Agent/Task
  tool** (a native subagent spawn — runs on Max, no nested CLI). The generated `run.sh` / `judge.sh` use
  `claude -p` because **launchd runs them OUTSIDE any Claude Code session** (`CLAUDECODE` unset) — that's the
  only place `claude -p` is allowed to fire.
- **`ANTHROPIC_API_KEY` is never written here.** Every runner `unset`s it so keychain OAuth (the Max plan) is
  used. There is **no `🔐 vault` and no Claude key file** on this track — the agent runs as the founder, reusing
  their existing logins and MCP servers. `.env` exists only if some *third-party* MCP server needs its own key.

Emoji on this track: 🤖 agent · 💻 environment (their machine / working dir) · 🎯 outcome · ▶️ run · 🧪 evals ·
⚖️ judge (the grader — the Max stand-in for CMA's server-side outcome grader) · 🧠 memory · 🔌 connector · 🗓️ deployment (the launchd job).

---

## `agent.md` — the subagent definition

**What this is:** the agent itself — a Claude Code subagent file. YAML frontmatter (`name`, `description`,
`model`, `tools`) followed by a body that *is* the system prompt. Drop it into `~/.claude/agents/` (global) or a
project's `.claude/agents/` (project wins on name clash). In-session it's invoked by the Agent/Task tool
(`subagent_type` = the `name`); unattended, `run.sh` references it. **What to fill in:** `{{AGENT_NAME}}` (kebab,
must match the file's slug and the launchd label), `{{ONE_LINE_DESCRIPTION}}`, the `{{MODEL_SLUG}}` (full slug
e.g. `claude-opus-4-8`, or an alias `opus`/`sonnet`/`haiku`), the `tools:` allowlist (omit the line entirely to
inherit all tools), `{{JOB}}`, and the `{{NEVER_DOS}}` bullets. Keep the two standing instructions
(write to `./outputs/`, relative dates only) verbatim — `run.sh` and `judge.sh` depend on them.

```markdown
---
name: {{AGENT_NAME}}
description: {{ONE_LINE_DESCRIPTION}}
model: {{MODEL_SLUG}}
tools: Bash, Read, Write, Edit, WebSearch, WebFetch
---

You are {{AGENT_NAME}}, a focused worker that does one job well and stops.

## Your job
{{JOB}}

Do exactly this job and nothing beyond it. If the task as given is ambiguous, make the most
reasonable assumption, state it in your output, and continue — do not stall waiting for input
(you may be running unattended on a schedule with no human present).

## Hard rules — never do these
{{NEVER_DOS}}
- Never present a guess as a fact. Mark anything you could not verify as "unverified".
- Never contact anyone, post anywhere, send, or place an order. Research and drafting only,
  unless your job above explicitly authorizes a specific write-action.

## Output
- Write every deliverable to the `./outputs/` directory (relative to where you are run),
  one self-contained file per run, named with the run's date, e.g. `outputs/digest-<YYYY-MM-DD>.md`.
- The file must stand on its own: a reader should understand it without seeing this prompt.
- End with a short "Notes / unverified" section if anything was assumed or could not be confirmed.

## Dates
- Always reason in RELATIVE dates: "today", "the last 7 days as of this run", "since yesterday".
- Never hard-code a literal calendar date in your logic — this agent re-runs on a schedule and
  a literal date would silently rot. Derive every date from the current run.
```

---

## `outcome.md` — the rubric (🎯 outcome)

**What this is:** the founder's definition of done — 3–6 binary, independently-checkable criteria. It is NOT in
the agent's system prompt (so it can be sharpened without touching the agent); `judge.sh` reads it to grade every
run. **What to fill in:** replace the criteria with the founder's real checks (mirror the `outcome.rubric_criteria`
in `build-sheet.max.example.json`). Keep each line binary — "has X", "every Y has a source", not "is good".

```markdown
# Outcome — {{AGENT_NAME}}

This run passes only if EVERY criterion below is true of the file in `outputs/`:

- [ ] The deliverable exists in `outputs/`, dated for this run, and is self-contained.
- [ ] It covers {{THE SCOPE — e.g. all 5 named competitors}} with nothing omitted.
- [ ] Every factual claim has a source link, or is explicitly tagged "unverified".
- [ ] {{A criterion specific to the job — e.g. each section ends with one "so what for us" line}}.
- [ ] Nothing unchanged-since-last-run is padded out; "no movement" is one line.
```

---

## `run.sh` — the headless unattended runner

**What this is:** the script launchd (or you, by hand) executes to do one real run, with no human present. It
forces Max OAuth by un-setting the API key, fires the subagent headless with a scoped tool allowlist, writes the
deliverable under `outputs/`, and logs the JSON result envelope (parsed with python3, not jq — the `.result`
text carries control characters). **What to fill in:** `{{AGENT_NAME}}`, `{{MODEL_SLUG}}`, the `TASK` text (use
relative dates), and the `ALLOWED_TOOLS` allowlist — exactly the tools the job needs, nothing more. The default
posture is a scoped allowlist + `--permission-mode acceptEdits`; the comment shows the `dontAsk`/bypass upgrade
and names its risk.

```bash
#!/usr/bin/env bash
# Copyright 2026 Anthropic PBC — SPDX-License-Identifier: Apache-2.0
#
# {{AGENT_NAME}} — unattended run on the Claude Code Max plan.
# Runs OUTSIDE any Claude Code session (launchd/cron), so `claude -p` is safe here.
# NEVER run this from inside an interactive Claude Code session (CLAUDECODE=1 → nested claude hangs).
set -euo pipefail

# --- Max OAuth, not the API ------------------------------------------------
# Leave ANTHROPIC_API_KEY UNSET so `claude -p` uses the keychain OAuth login (the Max plan: no per-run cost).
# DO NOT pass --bare to the runner: --bare disables keychain/OAuth and forces ANTHROPIC_API_KEY. Forbidden here.
unset ANTHROPIC_API_KEY

# --- Locate ourselves & set up dirs ----------------------------------------
AGENT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$AGENT_DIR"
mkdir -p outputs logs
TS="$(date +%Y-%m-%dT%H-%M-%S)"
RUN_JSON="logs/run-${TS}.json"
LOG="logs/run.log"

# --- The task (RELATIVE dates only — this re-runs on a schedule) ------------
TASK="{{JOB, phrased as an imperative task. e.g.:}}
Produce today's competitor digest covering the 5 named competitors, looking back over the
last 7 days as of this run. Write it to outputs/digest-$(date +%F).md per your output rules."

# --- Tool posture (be deliberate — no human is here to approve a prompt) ----
# Default: a SCOPED allowlist of exactly what the job needs + acceptEdits (file writes auto-approved,
# everything else still gated → safe to run unattended).
ALLOWED_TOOLS="Bash,Read,Write,Edit,WebSearch,WebFetch"
PERMISSION_MODE="acceptEdits"
# UPGRADE (only if the founder explicitly accepts it): PERMISSION_MODE="dontAsk" (or "bypassPermissions",
# or add --dangerously-skip-permissions) lets the unattended agent run ANY tool with no gate. RISK: a
# prompt-injected or buggy run could execute arbitrary Bash/Write with no human in the loop. Keep the scoped
# allowlist above unless the job genuinely needs more and the founder has said yes.

# --- Run it (headless; subagent body is the system prompt) -----------------
# --add-dir makes ./agent.md + ./outputs reachable; --agents loads the subagent definition.
echo "[$TS] starting {{AGENT_NAME}}" >>"$LOG"
set +e
claude -p "$TASK" \
  --model {{MODEL_SLUG}} \
  --append-system-prompt-file "$AGENT_DIR/agent.md" \
  --allowedTools "$ALLOWED_TOOLS" \
  --permission-mode "$PERMISSION_MODE" \
  --add-dir "$AGENT_DIR" \
  --output-format json \
  >"$RUN_JSON" 2>>"$LOG"
CLAUDE_EXIT=$?
set -e

# --- Parse the JSON result envelope with python3 (NOT jq: .result has control chars) ----
# Envelope: { type:"result", subtype:"success", is_error:bool, result:"<final text>",
#             session_id, total_cost_usd, usage:{...}, stop_reason }
read -r IS_ERROR RESULT_PREVIEW < <(python3 - "$RUN_JSON" <<'PY'
import json, sys
with open(sys.argv[1]) as f:
    d = json.JSONDecoder(strict=False).decode(f.read())
is_error = bool(d.get("is_error", True))
result = (d.get("result") or "").replace("\n", " ")[:120]
print("true" if is_error else "false", result)
PY
)

echo "[$TS] is_error=${IS_ERROR} exit=${CLAUDE_EXIT} :: ${RESULT_PREVIEW}" >>"$LOG"

if [ "$IS_ERROR" = "true" ] || [ "$CLAUDE_EXIT" -ne 0 ]; then
  echo "[$TS] RUN FAILED — see $RUN_JSON" >>"$LOG"
  # 401 here = the Max OAuth token expired → run `claude auth login` (or `claude setup-token`) to refresh.
  exit 1
fi

echo "[$TS] run ok — output in outputs/" >>"$LOG"
```

---

## `judge.sh` — the grader (⚖️ judge)

**What this is:** the Max stand-in for CMA's server-side outcome grader. A *second* `claude -p` call — given a
strict `--json-schema` so it must emit a structured verdict — that reads `outcome.md` (the rubric) plus the
latest file in `outputs/`, decides pass/fail per criterion, and prints `{verdict, criteria:[…], explanation}`.
Exits non-zero on fail so `run.sh`/launchd/CI can branch on it. **What to fill in:** nothing required — it reads
`outcome.md` and the newest output automatically; optionally pin `{{MODEL_SLUG}}` to the same model as the agent.
The JSON schema string is included verbatim below.

```bash
#!/usr/bin/env bash
# Copyright 2026 Anthropic PBC — SPDX-License-Identifier: Apache-2.0
#
# ⚖️ judge for {{AGENT_NAME}} — grades the latest run against outcome.md.
# Like run.sh, this runs OUTSIDE a Claude Code session and uses Max OAuth (no API key).
set -euo pipefail
unset ANTHROPIC_API_KEY   # force keychain OAuth; never --bare

AGENT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$AGENT_DIR"

RUBRIC_FILE="$AGENT_DIR/outcome.md"
LATEST_OUTPUT="$(ls -t outputs/* 2>/dev/null | head -n1 || true)"
if [ -z "$LATEST_OUTPUT" ]; then
  echo '{"verdict":"fail","criteria":[],"explanation":"No file in outputs/ to grade."}'
  exit 1
fi

RUBRIC="$(cat "$RUBRIC_FILE")"
OUTPUT="$(cat "$LATEST_OUTPUT")"

# Structured-output schema the judge MUST satisfy (forced via --json-schema).
SCHEMA='{
  "type": "object",
  "properties": {
    "verdict": { "type": "string", "enum": ["pass", "fail"] },
    "criteria": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name":     { "type": "string" },
          "pass":     { "type": "boolean" },
          "evidence": { "type": "string" }
        },
        "required": ["name", "pass", "evidence"],
        "additionalProperties": false
      }
    },
    "explanation": { "type": "string" }
  },
  "required": ["verdict", "criteria", "explanation"],
  "additionalProperties": false
}'

JUDGE_TASK="You are a strict grader. Below is a RUBRIC (binary criteria) and the AGENT OUTPUT it produced.
Evaluate the output against EVERY rubric criterion. For each, decide pass=true/false and quote the specific
evidence (or note what is missing). Set verdict to \"pass\" only if every criterion passes. Be literal; do
not give benefit of the doubt. Return ONLY the JSON object the schema requires.

=== RUBRIC (outcome.md) ===
${RUBRIC}

=== AGENT OUTPUT (${LATEST_OUTPUT}) ===
${OUTPUT}"

VERDICT_JSON="$(claude -p "$JUDGE_TASK" \
  --model {{MODEL_SLUG}} \
  --json-schema "$SCHEMA" \
  --tools "" \
  --output-format json)"

# Unwrap the run envelope (.result holds the judge's JSON string), then read the verdict.
# Pass the captured JSON via a FILE (never interpolated into the python source) so a stray ''' or
# trailing backslash in the text can't break the literal — python3, not jq (.result has control chars).
ENVELOPE="$(mktemp)"; trap 'rm -f "$ENVELOPE"' EXIT
printf '%s' "$VERDICT_JSON" >"$ENVELOPE"
python3 - "$ENVELOPE" <<'PY'
import json, sys
with open(sys.argv[1]) as f:
    env = json.JSONDecoder(strict=False).decode(f.read())
inner = json.loads(env["result"]) if isinstance(env.get("result"), str) else env.get("result", {})
print(json.dumps(inner, indent=2))
sys.exit(0 if inner.get("verdict") == "pass" else 2)
PY
```

---

## `evals/run-evals.sh` — loop the held-back cases through run + judge

**What this is:** the regression check. Each `evals/case-*/` folder holds an `input` (the task/prompt for that
case) and an `expected` (the known-good answer). This loops every case through the *same pinned* agent + model,
runs the agent on the case input, judges the result against `outcome.md`, and collects every verdict plus token
usage into `evals/results.json`. **What to fill in:** `{{AGENT_NAME}}`, `{{MODEL_SLUG}}` (pin the exact same slug
the agent uses — a different model is a different agent). Add case folders as you accumulate known-good runs;
case-01 is today's first verified output.

```bash
#!/usr/bin/env bash
# Copyright 2026 Anthropic PBC — SPDX-License-Identifier: Apache-2.0
#
# Run every evals/case-*/ through {{AGENT_NAME}} (pinned model) + ⚖️ judge → evals/results.json.
# Outside a Claude Code session; Max OAuth.
set -euo pipefail
unset ANTHROPIC_API_KEY

AGENT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"   # this script lives in evals/
cd "$AGENT_DIR"
MODEL="{{MODEL_SLUG}}"                                          # pin: same slug as the agent
ALLOWED_TOOLS="Bash,Read,Write,Edit,WebSearch,WebFetch"
RESULTS="evals/results.json"
mkdir -p evals outputs

echo "[" >"$RESULTS"
FIRST=1
for CASE in evals/case-*/; do
  [ -d "$CASE" ] || continue
  NAME="$(basename "$CASE")"
  [ -f "${CASE}input" ] || { echo "skip ${NAME}: no input file" >&2; continue; }
  TASK="$(cat "${CASE}input")"
  echo ">>> ${NAME}" >&2

  # Run the agent on this case (pinned model + scoped tools).
  RUN_JSON="$(claude -p "$TASK" \
    --model "$MODEL" \
    --append-system-prompt-file "$AGENT_DIR/agent.md" \
    --allowedTools "$ALLOWED_TOOLS" \
    --permission-mode acceptEdits \
    --add-dir "$AGENT_DIR" \
    --output-format json || true)"

  # Judge the freshest output against the rubric (reuse judge.sh — exits 0 pass / 2 fail).
  set +e
  VERDICT_JSON="$(./judge.sh 2>/dev/null)"
  JUDGE_EXIT=$?
  set -e
  [ "$JUDGE_EXIT" -eq 0 ] && PASS="true" || PASS="false"

  # Pull usage/cost from the agent run envelope (python3, not jq; via a file so the text can't break the literal).
  RUN_ENV="$(mktemp)"; printf '%s' "$RUN_JSON" >"$RUN_ENV"
  USAGE="$(python3 - "$RUN_ENV" <<'PY'
import json, sys
try:
    with open(sys.argv[1]) as f:
        d = json.JSONDecoder(strict=False).decode(f.read())
    print(json.dumps({"total_cost_usd": d.get("total_cost_usd"),
                      "usage": d.get("usage", {}),
                      "session_id": d.get("session_id")}))
except Exception as e:
    print(json.dumps({"error": str(e)}))
PY
)"
  rm -f "$RUN_ENV"

  [ $FIRST -eq 1 ] && FIRST=0 || echo "," >>"$RESULTS"
  python3 - "$NAME" "$PASS" "$USAGE" >>"$RESULTS" <<'PY'
import json, sys
name, passed, usage = sys.argv[1], sys.argv[2] == "true", sys.argv[3]
print(json.dumps({"case": name, "pass": passed, "run": json.loads(usage)}), end="")
PY
done
echo "]" >>"$RESULTS"

echo "Done. Verdicts + usage in $RESULTS" >&2
python3 -c "import json;d=json.load(open('$RESULTS'));p=sum(1 for c in d if c['pass']);print(f'{p}/{len(d)} cases passed')"
```

---

## `<label>.plist` — the launchd schedule (🗓️ deployment)

**What this is:** the macOS launchd job that fires `run.sh` on a calendar interval, with the Mac awake. House
label convention `com.launch-your-agent.<slug>`. Note the two things launchd gets wrong by default and this fixes:
launchd does **not** inherit the shell `PATH`, so an explicit full PATH (including `~/.bun/bin` and Homebrew) is
set; and `ANTHROPIC_API_KEY` is **absent** from the env so `run.sh` uses Max OAuth. **What to fill in:** `<slug>`
(matches `{{AGENT_NAME}}` and the filename), the absolute path to `run.sh`, the `<username>` in every path, and
`StartCalendarInterval` (the example below = 07:00 every Monday). For laptop-closed / always-on, use the cloud
`/schedule` routine instead — see `INSTALL.md`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.launch-your-agent.<slug></string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/<username>/my-agent/run.sh</string>
    </array>

    <key>EnvironmentVariables</key>
    <dict>
        <key>HOME</key>
        <string>/Users/<username></string>
        <!-- launchd does NOT inherit the shell PATH — set it explicitly so claude/bun/python3 resolve. -->
        <key>PATH</key>
        <string>/Users/<username>/.bun/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
        <!-- NO ANTHROPIC_API_KEY here on purpose: absence forces keychain OAuth (the Max plan). -->
    </dict>

    <!-- 07:00 every Monday. Edit / repeat the dict for other times; the Mac must be awake to fire. -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>1</integer>
        <key>Hour</key>
        <integer>7</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <key>StandardOutPath</key>
    <string>/Users/<username>/Library/Logs/launch-your-agent/<slug>.out.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/<username>/Library/Logs/launch-your-agent/<slug>.err.log</string>

    <key>RunAtLoad</key>
    <false/>
    <key>Nice</key>
    <integer>10</integer>
</dict>
</plist>
```

---

## `INSTALL.md` — install, test, schedule

**What this is:** the one-page setup card the founder follows to put the agent on disk, prove it runs once by
hand, and load the schedule. Fill in `{{AGENT_NAME}}`/`<slug>`/`<username>` to match the other files. The 401
note at the bottom is the single most common unattended failure (expired Max token).

````markdown
# Install — {{AGENT_NAME}} (Claude Code Max)

Runs on your machine under your Claude Code Max plan via keychain OAuth. **No API key, no per-run cost.**

## 1. Install the subagent
Copy `agent.md` to where Claude Code looks for subagents — project scope wins over global:

```bash
# Global (available everywhere):
cp agent.md ~/.claude/agents/{{AGENT_NAME}}.md
# — or — Project-scoped (this repo only):
mkdir -p .claude/agents && cp agent.md .claude/agents/{{AGENT_NAME}}.md
```

## 2. Manual test — run it once, by hand
Do this in a PLAIN terminal, **not inside a Claude Code session** (a nested `claude -p` hangs):

```bash
cd ~/my-agent
./run.sh        # writes outputs/<dated-file>, logs to logs/run.log
./judge.sh      # ⚖️ grades the latest output; prints the verdict JSON, exits 0 pass / 2 fail
```

Read the deliverable in `outputs/` and the verdict before trusting the schedule.

## 3. Load the schedule (launchd — Mac must be awake)
```bash
mkdir -p ~/Library/Logs/launch-your-agent
cp com.launch-your-agent.<slug>.plist ~/Library/LaunchAgents/
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.launch-your-agent.<slug>.plist
```

Fire it once immediately to confirm launchd can run it end-to-end:
```bash
launchctl kickstart -k gui/$(id -u)/com.launch-your-agent.<slug>
# then check the logs:
tail -n 40 ~/Library/Logs/launch-your-agent/<slug>.out.log ~/Library/Logs/launch-your-agent/<slug>.err.log
```

Unload later with: `launchctl bootout gui/$(id -u)/com.launch-your-agent.<slug>`.

**Laptop-closed / always-on instead?** Don't use launchd — use the cloud `/schedule` routine (the `schedule`
skill): it runs even with the lid shut and counts against your Max subscription usage (≈1h minimum interval).

## Troubleshooting
- **A scheduled run 401s** → your Max OAuth token expired. Refresh it: `claude auth login` (or, for a
  long-lived unattended token, `claude setup-token`). Check the method anytime with `claude auth status`.
- **launchd "fires" but nothing happens** → `PATH`/`HOME` not set in the plist env (claude/bun/python3 don't
  resolve), or `ANTHROPIC_API_KEY` leaked into the env (forces the API instead of OAuth). Both are handled in
  the provided plist — confirm you didn't add a key.
- **`claude -p` hangs** → you ran it inside a Claude Code session (`CLAUDECODE=1`). Use a plain terminal.
````

---

## `LAUNCH.md` — the on-demand interface

**What this is:** the tiny card for running the agent *by hand*, on demand — the whole interface for the
on-demand cadence. Same name as the CMA track's `LAUNCH.md`. **What to fill in:** `{{AGENT_NAME}}` and the
agent dir path.

````markdown
# Run {{AGENT_NAME}} on demand

Runs on your Claude Code Max plan — no API key, no per-run cost. Use a PLAIN terminal (not inside a
Claude Code session — a nested `claude -p` hangs).

```bash
cd ~/my-agent
./run.sh      # one run → outputs/<dated file>, logs/run.log
./judge.sh    # ⚖️ grade it → verdict JSON, exit 0 pass / 2 fail
```

To put it on a schedule instead, see `INSTALL.md` (launchd, or the cloud `/schedule` routine).
````

---

## Cohesion notes (for the skill emitting these)

- Filenames are FROZEN: `agent.md`, `outcome.md`, `run.sh`, `judge.sh`, `evals/run-evals.sh`, `<label>.plist`,
  `INSTALL.md`, `LAUNCH.md` (all templated above), plus `build-sheet.json`, `agent-overview.html`, and
  `NEXT-DIRECTIONS.md`. `outcome.md`, `LAUNCH.md`, `agent-overview.html`, `NEXT-DIRECTIONS.md` keep the SAME
  names as the CMA track so the two tracks cohere.
- The label `<slug>`, the `agent.md` `name:`, and the launchd filename are the same string — keep them in lockstep.
- `outputs/`, `logs/`, and `evals/` are created by the scripts; no need to scaffold empty dirs.
- On Max there is **no `🔐 vault` and no Claude API key file** — say so when introducing the folder, and only
  write `.env` if a *third-party* MCP server needs its own key.
