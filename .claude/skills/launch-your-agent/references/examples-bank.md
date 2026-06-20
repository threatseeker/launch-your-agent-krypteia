<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Inspiration & example bank — Claude Managed Agents

> Reference points for the skill: official cookbooks to lift config patterns from, real production deployments to anchor "what good looks like," and officially-grounded archetypes to offer as opening examples in the interview.
> **Sourcing rule:** everything in §1–§3 traces to an official Anthropic source (cookbook, docs, or Claude/Anthropic blog). Our own unvalidated ideas live in §5 (backlog).
> Collected 2026-06-14. Live sources win over this file.

## How the skill uses this bank

- **In the opening (and any time during Q1):** offer the 2–3 closest archetypes (§3) as concrete examples of what an agent can be; if the founder wants a menu, AskUserQuestion with the closest few.
- **When emitting the build kit:** if the founder's problem rhymes with a cookbook (§1), lift its config shape (tools, resources, event flow) rather than inventing one — and put the cookbook link in NEXT-DIRECTIONS.md as their "go deeper" reading.
- **When a founder asks "is anyone actually doing this?":** the production stories (§2) are the answer.

---

## 1. Official Anthropic cookbooks (the canonical examples)

Repo: [anthropics/claude-cookbooks → managed_agents/](https://github.com/anthropics/claude-cookbooks/tree/main/managed_agents) · rendered on [platform.claude.com/cookbook](https://platform.claude.com/cookbook/managed-agents-cma-iterate-fix-failing-tests). Python notebooks; include `utilities.py` (e.g. `stream_until_end_turn`), example data, and integration subfolders (slack/, sentry/, linear/, cma-mcp/, self_hosted_sandboxes/).

### Applied cookbooks (closest to "a founder's real agent")

| Cookbook | What it builds | Primitives it demonstrates | Good reference for |
|---|---|---|---|
| [data_analyst_agent](https://platform.claude.com/cookbook/managed-agents-data-analyst-agent) | CSV in → narrative HTML report with charts (pandas/plotly) | Environment `packages`, file mounting, streaming, artifact retrieval | Any "analyze my data and report" agent |
| slack_data_bot | The analyst wrapped in a Slack bot; @mention with a CSV, replies continue the session | Sessions as conversations, their-app-as-interface, resuming sessions | Founders productizing an agent inside a chat surface |
| [sre_incident_responder](https://platform.claude.com/cookbook/managed-agents-sre-incident-responder) | Pager alert → investigate → open PR → pause for human approval → merge | Event-triggered sessions (webhook), custom tools, human-in-the-loop gating, Skills | Ops/on-call agents; any "act, then wait for approval" flow |

### Guided tutorials (one primitive each, end-to-end)

| Tutorial | Teaches |
|---|---|
| [CMA_iterate_fix_failing_tests](https://platform.claude.com/cookbook/managed-agents-cma-iterate-fix-failing-tests) | **The entry point.** Agent/environment/session, file mounts, streaming event loop — via a do→observe→fix loop on a failing test suite |
| CMA_orchestrate_issue_to_pr | Multi-turn steering: issue → fix → PR → CI → review → merge, with mid-chain recovery |
| CMA_explore_unfamiliar_codebase | Grounding in a codebase; adding resources to a *running* session (`sessions.resources.add`) |
| CMA_gate_human_in_the_loop | Human-in-the-loop **expense approval** via custom tools (`decide()` / `escalate()`), the `requires_action` idle bounce |
| [CMA_prompt_versioning_and_rollback](https://platform.claude.com/cookbook/managed-agents-cma-prompt-versioning-and-rollback) | Agent versioning, regression detection, version pinning + rollback |
| [CMA_operate_in_production](https://platform.claude.com/cookbook/managed-agents-cma-operate-in-production) | MCP toolsets, vaults for per-end-user credentials, webhooks, resource lifecycle |
| [CMA_remember_user_preferences](https://platform.claude.com/cookbook/managed-agents-cma-remember-user-preferences) | Memory stores across sessions — customer preference learning and recall |
| [CMA_coordinate_specialist_team](https://platform.claude.com/cookbook/managed-agents-cma-coordinate-specialist-team) | Multiagent coordinator + three scoped specialists |
| [CMA_verify_with_outcome_grader](https://platform.claude.com/cookbook/managed-agents-cma-verify-with-outcome-grader) | Outcomes: the grade-and-revise loop (our default kickoff pattern) |

**Mapping to our interview:** Q2/Q2b ↔ verify_with_outcome_grader · Q3 credentials ↔ operate_in_production · Q5 event-driven ↔ sre_incident_responder · Q7 memory ↔ remember_user_preferences · Q8 multiagent ↔ coordinate_specialist_team · iteration/eval regression ↔ prompt_versioning_and_rollback.

### Worked examples inside the docs themselves

- **Financial analyst building a DCF model in .xlsx** — the running example of the [Outcomes page](https://platform.claude.com/docs/en/managed-agents/define-outcomes) (rubric-graded spreadsheet deliverable) and the [Skills page](https://platform.claude.com/docs/en/managed-agents/skills) (`xlsx` skill).
- **Weekly compliance scan** — the running example of the [Scheduled deployments page](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments) (`0 20 * * 5`, America/New_York); the launch post also names **nightly data sync** and **daily digest** as the intended uses.
- **Coding assistant** — the quickstart's default agent (write code, run it, verify the output file).
- **Per-end-user agents with vaults** — the [Vaults page](https://platform.claude.com/docs/en/managed-agents/vaults)'s "Alice's Slack digest" pattern: one vault per end user, `external_user_id` in metadata.

---

## 2. Real production deployments (proof points)

For "is anyone actually doing this?" — point at the customer stories in [the evolution of agentic surfaces](https://claude.com/blog/building-with-claude-managed-agents) and [Claude Managed Agents: get to production 10x faster](https://claude.com/blog/claude-managed-agents). The common thread (and the pitch to founders): the agent operates **inside the systems they already use** — codebase, wiki, ticketing, chat — and the managed harness removes the infra build entirely.

Also worth reading: [Scaling Managed Agents: decoupling the brain from the hands](https://www.anthropic.com/engineering/managed-agents) (Anthropic engineering) — the architecture story (brain = model+harness, hands = sandbox, session = the persistent event log), useful framing language for explaining CMA to a founder in one breath.

---

## 3. Archetypes to offer (every one traces to an official source)

| # | Archetype | v0 (today) | Source | Natural next directions |
|---|---|---|---|---|
| 1 | **Data analyst** | Hand it a CSV/export → narrative HTML report with charts: what's interesting, what changed. | data_analyst_agent cookbook | Weekly scheduled deployment against a fresh export; lives in Slack via the slack_data_bot pattern; their app embeds the report |
| 2 | **Ops responder with approval** | An alert/event → investigate → draft the fix or PR → stop and wait for human approval. | sre_incident_responder cookbook | Wire to their alerting webhook; custom tools into their backend; always_ask gates |
| 3 | **Document-producing analyst** | A research/finance task whose deliverable is a graded artifact — e.g. a DCF model in .xlsx judged against a rubric. | Outcomes + Skills docs (DCF example) | File-based rubrics; pptx/docx variants; recurring refresh on a deployment |
| 4 | **Engineering agent: issue → PR** | Take an issue, fix it, open the PR, recover when CI or review pushes back. | CMA_orchestrate_issue_to_pr + fix_failing_tests cookbooks | GitHub MCP + vault; triggered from their tracker; grounding pass on an unfamiliar repo first (explore_unfamiliar_codebase) |
| 5 | **Assistant that remembers your users** | A customer-facing helper whose memory store accumulates each user's preferences across sessions. | CMA_remember_user_preferences cookbook | One memory store + one vault per end user; Dreams to consolidate |
| 6 | **Recurring compliance / digest scan** | A scheduled sweep — weekly compliance scan, nightly data sync, daily digest — that runs and files its report without anyone present. | Scheduled-deployments docs + launch blog | Limited-networking allowlist; webhook notifications; outcome grading per run |
| 7 | **Specialist team** | A coordinator that delegates to a small roster of scoped specialists (research / review / write) working in parallel. | CMA_coordinate_specialist_team cookbook | Per-specialist models and tools; escalation tier on Opus |

Personalization rule: change only the name/company/product slots, one sentence of domain context, and the output format — the underlying config shape comes from the source cookbook/doc.

---

## 3b. Which archetypes fit which build target

The fork (🤖 Build target) is asked early — but the archetype menu above is a useful tell for *which* track the founder probably wants. The §1–§3 material is all CMA-sourced; that doesn't mean every archetype is CMA-native. Most run beautifully on **Claude Code (Max)** too — and some are a *better* fit there, because the job is the founder's own recurring chore, touches local files, or reuses logins they already have. Tag, then recommend; the founder still chooses.

| Archetype (from §3) | Natural target | Why |
|---|---|---|
| 1 · Data analyst | **Either** | Their own export on their disk → 💻 Max (no key, reads the file directly). Same report inside *their product*, multi-tenant → ☁️ CMA. |
| 2 · Ops responder with approval | **Either** | A solo dev triaging their own alerts → 💻 Max (launchd or `/schedule`). On-call for a team/product, must run laptop-closed → ☁️ CMA. |
| 3 · Document-producing analyst | **Either** | A doc the founder needs on a cadence → 💻 Max. A graded artifact that's part of a customer deliverable → ☁️ CMA. |
| 4 · Engineering agent: issue → PR | **Max-leaning** | Runs against *their* repo with *their* gh/GitHub MCP already authenticated → 💻 Max, no vault. Goes ☁️ CMA only when it's a hosted service acting on many tenants' repos. |
| 5 · Assistant that remembers your users | **CMA** | "Your users," per-end-user memory + vault, customer-facing → ☁️ CMA by definition (multi-tenant). |
| 6 · Recurring compliance / digest scan | **Max-leaning** | The founder's own scheduled sweep → 💻 Max via launchd (or `/schedule` for laptop-closed). ☁️ CMA when the scan is a product feature run server-side for others. |
| 7 · Specialist team | **Either** | In-session coordinator over local work → 💻 Max (spawn specialists via the Agent/Task tool). A productized multiagent service → ☁️ CMA. |

**The cut, in one line:** *for-you / your-own-chore / dev task / uses your logins / local files / no per-run billing* → **💻 Claude Code (Max)**. *for-your-product / customer-facing / multi-tenant / must run without your laptop / needs a hosted isolated sandbox* → **☁️ CMA**.

## 3c. Max-native archetypes to offer

These have **no official CMA cookbook** — they're ideal on **Claude Code (Max)** precisely because the value *is* the local machine: no API key (OAuth on the Max plan), direct local-file access, and reuse of MCP servers the founder already authenticated. Offer them when the founder's job rhymes with "my own recurring thing" and the §3 menu feels too productized. Mark them clearly as Max-track examples, not CMA cookbooks.

> Mechanics reminder (state when offering): in-session, the just-built 🤖 agent is fired via the **Agent/Task tool** and graded by a **⚖️ judge subagent** — never a nested `claude -p`. Unattended, the generated `run.sh` uses `claude -p` under **launchd** (or a `/schedule` ☁️ routine for laptop-closed); ANTHROPIC_API_KEY stays unset so the Max OAuth keychain login is used.

| # | Archetype | v0 (today) | Why Max | Natural next directions |
|---|---|---|---|---|
| M1 | **Nightly repo health/triage** | Each night, sweep a local repo (build/test/lint, stale branches, TODO/FIXME drift, dependency flags) → write `report-<date>.md` to the project. | Reads the working tree directly, runs the founder's real toolchain, reuses their git auth — no key, no vault, no upload. | launchd at a fixed hour; add a ⚖️ judge so empty/garbled reports re-run; widen to a multi-repo loop; post the digest to Slack only via the outbox/gate pattern. |
| M2 | **"My morning brief"** | Read local files (today's notes, a tasks file, a project log) **and** the founder's already-authenticated MCP servers (calendar, mail-read) → write `brief-<date>.md` digest. | Zero credential handoff — it just *has* the logins the founder already set up; touches private local files that never leave the machine. | Schedule via launchd for a morning hour (or `/schedule` ☁️ for laptop-closed); 🧠 memory file to carry "what changed since yesterday"; voice/notify hand-off; promote any send action through the outbox gate. |
| M3 | **Dev-loop fix agent** (on-demand) | Run the test suite, read failures, draft a fix on a scratch branch, re-run, stop with a diff for review — invoked when the founder asks, not on a clock. | Runs in the founder's repo with their tools live; in-session it's literally a subagent spawned via the Agent/Task tool — no key, no schedule, no infra. | Bound the run→judge→re-run loop in `run.sh` for headless/CI use; ⚖️ judge against an outcome rubric; cap iterations; never auto-merge — leave the diff for a human. |
| M4 | **Inbox / notes triage over local data** | Cluster a local maildir / notes folder / exports by topic or root cause, draft labels or replies into `./outputs/` — **never sends**. | Operates on local data and reuses any already-wired mail-read MCP; the no-send posture is enforced by simply leaving the send tool out of the runner's `--allowedTools`. | Add a real mail MCP for richer reads; outbox the drafts for human review; 🧠 memory to learn the founder's categories over time; gate any "actually send" behind explicit human go. |

All four assume the Max defaults: a scoped `--tools`/`--allowedTools` allowlist plus `--permission-mode acceptEdits` for the unattended runner, write-actions gated out of the runner (outbox under `./outputs/outbox/`) until the founder explicitly wants them firing on their own.

---

## 4. Other references

- [Quickstart](https://platform.claude.com/docs/en/managed-agents/quickstart) — the minimal four-call launch; also points at `/claude-api managed-agents-onboard` inside Claude Code.
- [What's new in Claude Managed Agents](https://claude.com/blog/whats-new-in-claude-managed-agents) — scheduled deployments + env-var vault credentials announcement (June 2026).
- [anthropics/skills → claude-api/shared/managed-agents-overview.md](https://github.com/anthropics/skills/blob/main/skills/claude-api/shared/managed-agents-overview.md) — how Anthropic's own skills summarize CMA (tone/altitude reference for our SKILL.md).
- Community walk-throughs exist (Substack/blog tutorials, a third-party onboarding repo) — fine as colour, but always defer to the official docs/cookbooks for API shapes.

---

## 5. Other ideas backlog

Not yet validated against an official example — don't offer in the interview menu.

- Inbox triage (bucket emails, draft replies, never send)
- Competitive / market watch (weekly cited digest of competitor changes)
- Support-ticket / review summarizer (cluster by root cause, top-3 fixes)
- Research-to-brief (one question → decision-ready brief with sources)
- Investor update drafter (monthly metrics + notes → update draft)
