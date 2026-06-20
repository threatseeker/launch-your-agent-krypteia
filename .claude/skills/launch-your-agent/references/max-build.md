<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Claude Code (Max) track — primitive map + verified mechanics

> The Max-track analog of `interview.md`'s primitive map and `cma-api.md`'s command shapes, in one
> file. When `build_target: "claude-code-max"` is chosen at THE FORK, this is the mechanics reference.
> Companion to `interview.md` (the SHARED interview) and `cma-api.md` (the peer CMA track).

## The shape of the thing

The interview is identical to CMA's — same eight clusters, same Outcome rubric, same eval thinking
(`interview.md` owns it). What changes is **where the agent runs** and **what each primitive maps
onto**. On Max the agent runs on the founder's own machine, under their Claude Code Max plan via
OAuth — 💻 **no API key, no per-run billing, runs AS them** (reusing their existing logins, MCP
servers, and local files). The build kit lands in `./my-agent/` exactly as on CMA; only the
filenames and mechanics differ where noted.

The headline simplifications versus CMA:
- 💻 **No vault.** The agent runs as the user, so it inherits their already-authenticated logins and
  MCP servers. There is no per-run credential handoff. (On CMA every run needs a 🔒 Vault credential.)
- 💻 **No Claude API key.** Max = OAuth from the keychain. The only `.env` that ever exists holds a
  *non-Claude* MCP server key, and only if some server needs one.
- ⚖️ **You build the grader.** CMA has a server-side Outcome grader; on Max there is none, so the
  skill builds **the judge** (a subagent in-session, a second `claude -p` call unattended).

---

## 1. The CMA → Claude Code (Max) primitive map

For every CMA primitive, the Max equivalent. The **decided-by** column is unchanged from
`interview.md` (the interview is shared); only the right two columns are Max-specific.

| CMA primitive | Decided by (shared interview) | Claude Code (Max) equivalent | Lands in `./my-agent/` as |
|---|---|---|---|
| 🤖 **Agent** (name, model, system) | Q1 job, Q6 never-dos, Q8 model | **Subagent file** `.claude/agents/<Name>.md` — frontmatter (`name`, `description`, `model`, `tools`) + body = system prompt | `agent.md` |
| **Model** | Q8 posture | `model:` frontmatter field — alias (`opus`/`sonnet`/`haiku`/`fable`) or full slug (e.g. `claude-opus-4-8`); omit → inherits the session's model | `agent.md` frontmatter |
| 💻 **Environment** (packages, networking) | Q3 inputs, Q6 hardening | **The local machine** — the user's own shell, installed tools, working dir. Scope reachable dirs with `--add-dir <dirs...>`; packages are whatever's already installed (no cloud env to provision) | `--add-dir` lines in `run.sh` / `INSTALL.md` |
| 🔨 **Tools + permission policies** | Q3 inputs, Q4 outputs, Q6 approvals | `tools:` allowlist in the subagent frontmatter (comma-separated, e.g. `Bash, Read, Write, WebSearch`; omit → inherits all). Unattended posture set by `--tools` / `--allowedTools` / `--disallowedTools` + `--permission-mode` on the runner | `agent.md` `tools:` + `run.sh` flags |
| 🎯 **Outcome** (rubric) → grader | Q2 done, Q2b evidence | The **rubric stays** (`outcome.md`, same name). The grader becomes ⚖️ **the judge** — a judge subagent in-session, a `claude -p --json-schema` call unattended | `outcome.md` + `judge.sh` |
| **Session** (one run) | every run | **A single `claude -p` run** (unattended) OR **an Agent/Task-tool spawn** (in-session). No session resource to create — the run IS the session; `--session-id`/`-c`/`-r` resume if needed | each `run.sh` invocation |
| 📅 **Scheduled deployment** (cron) | Q5 = recurring | **launchd plist** (`StartCalendarInterval`) by default; ☁️ **/schedule routine** for laptop-closed | `<label>.plist` (if scheduled) |
| 🔒 **Vault + credentials** | Q3 logins | **N/A — the agent runs as you.** It reuses the user's already-authenticated MCP servers and logins. No per-run credential handoff. (`.env` exists only for a *non-Claude* MCP key) | nothing (or `.env` for one MCP key) |
| 🔌 **MCP servers / connectors** | Q3 SaaS behind login | The user's **existing** `.mcp.json` (project) / `~/.claude.json` (user) servers — already authenticated. Headless inherits them, or `--mcp-config <file>` to add/scope, `--strict-mcp-config` to use only those | reuse existing; optional `--mcp-config` in `run.sh` |
| 🧠 **Memory store** | Q7 learning | **Local memory files** the agent reads/writes (a `memory/` dir the run is pointed at). No server-side store; the agent reads it at start, appends at end | `memory/` (only if learning) |
| 📄 **Skills** | Q4 house formats | The user's **installed skills** (`~/.claude/skills/`, project `.claude/skills/`) — inherited by the spawned agent/run, same as in any Claude Code session | reuse installed; author one if small |
| 🔨 **Custom tools** | Q4 trigger their product | A **local script** the agent runs via `Bash` (it's their machine — no client-tool round-trip needed). Gate it with the outbox pattern if it writes | a script under `my-agent/`, called from `agent.md` |
| 📄 **Files API** | Q3 inputs, Q4 outputs | 💻 **Local files.** Inputs live on disk (point the run at them with `--add-dir`); outputs are written to a path the user names. No upload/download API | local paths |
| 🤖 **Multiagent** (coordinator + roster) | Q8 shape | **The agent spawns sub-subagents via the Agent/Task tool** (in-session) or `claude -p --agents '<json>'` (headless). Same default-NO rule as CMA — earn it | extra `.claude/agents/*.md` (only when earned) |

Reading the table: the **CMA primitive** and **decided-by** columns are the bridge — a founder who
saw the CMA overview recognizes every row; the right two columns are what actually gets built on
their machine. Anything that's "deferred" still goes in `NEXT-DIRECTIONS.md` (same name) with its
Max mechanism.

### The starting point (what every Max build begins from)

- 🤖 **One subagent** (`.claude/agents/<Name>.md`), newest Opus-class model, an inherited or scoped
  `tools:` allowlist
- 💻 **The user's own machine** as the environment, working dir + any `--add-dir` paths
- 🎯 **Outcome rubric** in `outcome.md`, graded by ⚖️ **the judge**
- Outputs written to a local path the user names; inputs read from local paths

From there, build what the job needs into numbered versions (v1, v2, …) in `NEXT-DIRECTIONS.md` —
identical philosophy to CMA. The standing recommendation is the same: v0 is read/analyze/draft only;
write-actions ship gated in a later version once the user trusts the output.

---

## 2. ⭐ The keystone — in-session vs unattended (the nested-Claude rule)

This is the load-bearing design rule of the whole Max track. Read it before touching any mechanics.

> **Inside an interactive Claude Code session, `CLAUDECODE=1` is set, and spawning a nested
> `claude` subprocess hangs or fails.** Therefore the skill must **NEVER run `claude -p` inline
> during the build session.**

The two contexts split cleanly:

| Context | When | How the agent runs | How grading runs |
|---|---|---|---|
| **IN-SESSION** (live build + first run + grade — Phase 3) | While the skill is driving the build, inside Claude Code | Fire the just-built agent via the **Agent/Task tool** (native subagent spawn — runs on Max, no nested CLI, no key) | Spawn a **judge subagent** via the Agent/Task tool against the `outcome.md` rubric |
| **UNATTENDED** (scheduled / CI — Phase 4) | launchd / cron, OUTSIDE any Claude Code session | The generated `run.sh` calls **`claude -p`** — fine here, because `CLAUDECODE` is unset | `judge.sh` (a second `claude -p --json-schema` call) |

**Why:** the nested-session block means `claude -p` only works when `CLAUDECODE` is *absent* — which
is exactly the launchd/cron case and never the in-session case. So: build, run, and grade in-session
go through the Agent/Task tool; the generated `run.sh`/`judge.sh` use `claude -p` and only ever
execute later, fired by launchd. **Never demo `run.sh` by running `claude -p` from inside the build
session** — it will hang. To verify the agent live during the build, spawn it with the Agent/Task
tool instead.

---

## 3. Auth — the verified Max / OAuth / no-key story

The user's machine reports `authMethod: "claude.ai"`, `subscriptionType: "max"`,
`apiProvider: "firstParty"`. On Max:

| Fact | What to do |
|---|---|
| **No API key, no per-run cost** | `claude -p` runs on the Claude Code OAuth login in the keychain. Leave `ANTHROPIC_API_KEY` **UNSET** so keychain OAuth is used (proven: "`ANTHROPIC_API_KEY` empty → `claude -p` uses Max OAuth"). |
| `--bare` disables OAuth | **NEVER run the runner with `--bare`** — it disables keychain/OAuth and forces `ANTHROPIC_API_KEY`. |
| Scheduled run **401s** | The token expired → `claude auth login` to refresh. |
| Robust unattended auth | `claude setup-token` issues a long-lived subscription token (real subcommand) — use it for hands-off scheduled runs. |
| Check the method | `claude auth status` shows the auth method. |

Corollary for the whole track: there is **no Claude key in `.env`, ever**. The only secret that can
appear is a *non-Claude* MCP server key (and only if a server needs one). State this plainly to the
user — it's the biggest difference from CMA.

---

## 4. The headless runner — verified `claude -p` mechanics

`run.sh` is the unattended runner. It runs OUTSIDE a Claude Code session (launchd/cron), so
`claude -p` is safe here. Use ONLY these verified flags — never invent one.

### Flags this skill uses (spell them exactly)

| Flag | Use |
|---|---|
| `-p` / `--print` | Headless (non-interactive) mode |
| `--output-format text\|json\|stream-json` | `json` to read the result envelope |
| `--model <alias\|slug>` | Pin the model (`opus` / `claude-opus-4-8`) |
| `--append-system-prompt <s>` / `--system-prompt <s>` | Inject the agent's system prompt (or `--append-system-prompt-file` / `--system-prompt-file` — the `-file` variants are CLI-accepted but **not** surfaced in the top-level `--help` list; confirm with a real invocation, not by grepping `--help`) |
| `--agents '<json>'` | Define subagents inline: `{"name":{"description":"...","prompt":"..."}}` |
| `--tools "Bash,Read,Write"` | Built-in tool scoping (`""` = none, default = all) |
| `--allowedTools` / `--allowed-tools <list>` · `--disallowedTools <list>` | Fine-grained allow/deny (e.g. `"mcp__github__*"`) |
| `--permission-mode acceptEdits\|auto\|bypassPermissions\|default\|dontAsk\|plan` | Unattended permission posture (see §8) |
| `--dangerously-skip-permissions` | Full bypass — only with explicit user consent (see §8) |
| `--mcp-config <files...>` · `--strict-mcp-config` | Add/scope MCP servers; `--strict` = only those (see §7) |
| `--add-dir <dirs...>` | Grant the run access to input/output dirs (the "environment") |
| `--json-schema '<schema>'` | **Force structured output** — used for ⚖️ the judge (see §5) |
| `--session-id <uuid>` · `-c`/`--continue` · `-r`/`--resume` · `--fork-session` | Resume/continue a run if needed |
| `--max-budget-usd <n>` · `--fallback-model <list>` | Print-only safety rails |
| `--settings <file\|json>` · `--setting-sources user,project,local` | Load settings explicitly |

### The JSON result envelope (`--output-format json`)

```json
{ "type": "result", "subtype": "success", "is_error": false,
  "result": "<final text>", "session_id": "<uuid>",
  "total_cost_usd": 0, "usage": { }, "stop_reason": "end_turn" }
```

Read `.result` (the final text) and `.is_error` (the failure flag).

**Parse with python, NOT jq** — the `result` field embeds the agent's output with literal
newlines/control characters that jq rejects (same gotcha as CMA's session payloads):

```bash
python3 -c "import json; d=json.JSONDecoder(strict=False).decode(open('/tmp/out.json').read()); print(d['result'])"
```

A minimal `run.sh` skeleton (illustrative — only verified flags):

```bash
#!/bin/bash
# Copyright 2026 Anthropic PBC — Apache-2.0
# Unattended runner. Runs OUTSIDE any Claude Code session (launchd/cron) — claude -p is safe here.
# ANTHROPIC_API_KEY must be ABSENT so Max OAuth (keychain) is used. Never --bare.
set -euo pipefail
cd "$(dirname "$0")"
claude -p "$(cat first-prompt.txt)" \
  --append-system-prompt-file agent.md \
  --model claude-opus-4-8 \
  --add-dir "$HOME/work/inputs" "$HOME/work/outputs" \
  --tools "Bash,Read,Write,WebSearch,WebFetch" \
  --permission-mode acceptEdits \
  --output-format json > /tmp/run-out.json
python3 -c "import json;d=json.JSONDecoder(strict=False).decode(open('/tmp/run-out.json').read());open('outputs/result.md','w').write(d['result'])"
```

---

## 5. ⚖️ Grading / evals — the judge (you build it)

There is **no built-in grader on Max** — the skill builds **the judge**. The 🎯 Outcome rubric
concept and `outcome.md` file stay (same name as CMA); only the grader implementation differs.

| Context | The judge is | The `max_iterations` analog |
|---|---|---|
| **In-session** (Phase 3) | A **judge subagent** spawned via the Agent/Task tool — reads the agent's output + `outcome.md`, returns a verdict | n/a (you spawn run + judge once, iterate by hand) |
| **Unattended** (`run.sh`/`judge.sh`) | A second **`claude -p --json-schema '<schema>'`** call that emits a structured verdict | A **bash loop bound** in `run.sh`: run → judge → if fail and tries < N, re-run with the judge feedback appended |

### The judge JSON schema shape

`judge.sh` runs `claude -p --json-schema '<schema>'` so the verdict is structured and parseable:

```json
{ "verdict": "pass" | "fail",
  "criteria": [ { "name": "<rubric criterion>", "pass": true, "evidence": "<why>" } ],
  "explanation": "<overall>" }
```

### The iteration loop (the `max_iterations` analog)

```bash
# in run.sh — run → judge → retry-with-feedback, bounded by N
MAX_ITERS=3; tries=0; feedback=""
while [ "$tries" -lt "$MAX_ITERS" ]; do
  claude -p "$(cat first-prompt.txt)${feedback:+

Reviewer feedback to fix: $feedback}" \
    --append-system-prompt-file agent.md --add-dir ./outputs \
    --tools "Bash,Read,Write" --permission-mode acceptEdits \
    --output-format json > /tmp/run.json
  verdict=$(bash judge.sh /tmp/run.json)   # judge.sh: claude -p --json-schema, prints pass|fail+feedback
  case "$verdict" in pass*) break;; esac
  feedback="${verdict#fail }"; tries=$((tries+1))
done
```

`evals/run-evals.sh` is the golden-set loop: for each case in `evals/`, run `run.sh` against the
case input and pass the output to `judge.sh`, collecting verdicts. Same golden-set logic as CMA
(`interview.md` Q2b) — only the run+grade primitives change. The fresh-data-recurring rule still
applies: today's first verified output becomes `evals/case-01/expected.md`, the regression baseline.

---

## 6. 📅 Scheduling

Two options, in order of default preference.

### (a) macOS launchd — default for a personal Max agent

A plist runs `run.sh` on a calendar interval. **The Mac must be awake** for the job to fire.

Plist anatomy (verified requirements):

| Key | Value |
|---|---|
| `Label` | `com.launch-your-agent.<slug>` (house convention) or the user's own namespace |
| `ProgramArguments` | `["/bin/bash", "<abs path>/run.sh"]` |
| `StartCalendarInterval` | The schedule (`{Hour, Minute, Weekday}` dicts) |
| `EnvironmentVariables` | `HOME` **and an EXPLICIT full PATH** — launchd does NOT inherit the shell PATH. Include `/Users/<user>/.bun/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin`. **`ANTHROPIC_API_KEY` must be ABSENT** so OAuth keychain is used. |
| `StandardOutPath` / `StandardErrorPath` | `~/Library/Logs/<namespace>/...` |
| `RunAtLoad` | `false` |
| `Nice` | `10` |

```bash
# Load the job:
launchctl bootstrap gui/$(id -u) <plist>
# Manual test fire (don't wait for the clock):
launchctl kickstart -k gui/$(id -u)/<label>
# (or just run run.sh directly to test the runner itself)
```

Two gotchas that bite every time:
- **PATH:** launchd starts with a minimal PATH — `claude`, `bun`, `python3` won't resolve unless you
  set the full PATH explicitly in `EnvironmentVariables`.
- **No Claude key in the plist env** — its absence is what makes OAuth keychain kick in. A key here
  silently flips the run onto metered API billing.
- **Awake-machine caveat:** a sleeping/closed Mac won't run the job. If the user needs
  laptop-closed, that's option (b).

### (b) ☁️ /schedule routine — laptop-closed / always-on

The `schedule` skill (real) creates an Anthropic-hosted routine that **runs even with the laptop
closed**, counts against Max subscription usage, has ~1h minimum interval, and supports cron/preset +
GitHub/API triggers. Offer it as the "always-on / laptop-closed" alternative and **point at the
`/schedule` skill** rather than over-specifying its management CLI.

---

## 7. 🔌 MCP / connectors on Max

The big simplification: 💻 **the agent reuses the user's ALREADY-authenticated MCP servers** —
`.mcp.json` (project) and `~/.claude.json` (user). There is **no per-run vault / credential handoff**
(CMA needs a Vault credential every run; Max already has the login).

| Need | Mechanism |
|---|---|
| Use the servers the user already has | Headless inherits the configured servers automatically — nothing to wire |
| Add or scope servers for this run | `--mcp-config <file>` to add; `--strict-mcp-config` to use ONLY those |
| Limit which MCP tools the run may call | `--allowedTools "mcp__github__*"` (scope by server/tool) |

Because the login already exists, **connector "mocking" is rarely needed on Max** — you usually just
have the server. The outbox pattern still applies, but only to **gate a write-action** you don't want
firing unattended (see §8), not to substitute for a missing credential.

---

## 8. 🔨 Unattended permission posture

A scheduled agent runs `Bash`/`Write` with no human present, so its permission posture is a
deliberate security decision — name the tradeoff, never hand it full bypass silently.

| Posture | When | Flags |
|---|---|---|
| **Scoped allowlist (DEFAULT)** | Almost always — limit to exactly what the job needs | `--tools` / `--allowedTools` limited to the needed tools **+** `--permission-mode acceptEdits` |
| **Broad bypass** | ONLY when the user explicitly accepts that the unattended agent can run any tool | `--permission-mode dontAsk` / `bypassPermissions`, or `--dangerously-skip-permissions` |

Default the generated `run.sh` to the **scoped allowlist** posture. Reach for `dontAsk`/`bypass`/
`--dangerously-skip-permissions` only on explicit consent — and say what it means: the unattended
agent can then run any tool with no gate. For a write-action the user wants but doesn't fully trust
yet, keep the agent draft-only and gate the send with the outbox pattern (write the schema-true
payload to a file; a human or a later gated step sends it) rather than granting broad bypass.

---

## 9. Troubleshooting quick hits (Max)

- **Nested-Claude hang** → you ran `claude -p` inside the build session (`CLAUDECODE=1`). Don't.
  Use the Agent/Task tool in-session; `claude -p` only runs unattended (launchd/cron). (§2)
- **401 on a scheduled run** → the OAuth token expired → `claude auth login` to refresh; or
  `claude setup-token` for a long-lived subscription token. (§3)
- **Run silently billed to API** → `ANTHROPIC_API_KEY` was set (in shell, `.env`, or the plist env),
  or `--bare` was used. Unset the key everywhere; never `--bare`. (§3)
- **launchd job never fires** → check the explicit PATH in the plist env (launchd doesn't inherit
  the shell PATH), confirm the Mac was awake, and read `StandardErrorPath`. (§6a)
- **`claude`/`bun`/`python3` not found in the launchd run** → missing full PATH in
  `EnvironmentVariables`. (§6a)
- **Need laptop-closed scheduling** → launchd can't (machine must be awake) → use the `/schedule`
  routine instead. (§6b)
- **jq fails on the result** → the `result` field embeds control characters. Parse with
  `python3` + `json.JSONDecoder(strict=False)`, not jq. (§4)
- **MCP tool not available unattended** → it wasn't inherited or wasn't scoped in. Pass
  `--mcp-config <file>` and allow it with `--allowedTools "mcp__<server>__*"`. (§7)
- **Unattended agent did something risky** → it had too broad a posture. Default to scoped
  `--tools`/`--allowedTools` + `--permission-mode acceptEdits`; reserve bypass for explicit consent. (§8)

---

## FROZEN `./my-agent/` artifacts (Max track)

| File | Contents |
|---|---|
| `build-sheet.json` | The full structured mapping (includes `build_target: "claude-code-max"`) |
| `agent.md` | 🤖 The subagent: frontmatter (`name`, `description`, `model`, `tools`) + body = system prompt |
| `run.sh` | 🎬 Headless unattended runner (`claude -p`, scoped posture, run→judge loop) |
| `judge.sh` | ⚖️ The grader (`claude -p --json-schema`) |
| `outcome.md` | 🎯 The rubric (SAME name as CMA) |
| `evals/` + `evals/run-evals.sh` | 🧪 Loop cases through run + judge |
| `<label>.plist` | 📅 launchd job (only if scheduled) |
| `INSTALL.md` | Install the subagent + load the launchd job + manual test fire |
| `LAUNCH.md` | ▶️ On-demand run instructions (SAME name as CMA) |
| `agent-overview.html` | The schema page (SAME name; dual-track render) |
| `NEXT-DIRECTIONS.md` | The version plan (SAME name) |
| `memory/` | 🧠 Local memory files (only if learning) |
| `.env` | ONLY if some MCP server needs a key — **the Claude key is NEVER here; Max = OAuth** |

On Max there is **NO vault and NO Claude API key file.** Say this.
