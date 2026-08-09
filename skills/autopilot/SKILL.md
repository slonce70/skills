---
name: autopilot
description: Orchestrate an app, site, bot, or feature from brief to tested code through the mattpocock/skills pipeline. Use for end-to-end "build it for me", "собери под ключ", vibecoding, or explicit /autopilot requests. Supports full, semi, and manual approval modes.
---

# Autopilot

Run the pipeline in order: setup → brief → spec → tickets → implement → review → finish. The invoked skills own their phases; this file defines only orchestration and overrides. Never write implementation code before the spec and tickets exist.

If the brief has no single project outcome and is too broad for one spec, stop and recommend `wayfinder` when available.

## Dependencies

Use:

- `setup-matt-pocock-skills`
- `grilling`
- `domain-modeling`
- `grill-with-docs`
- `to-spec`
- `to-tickets`
- `implement`
- `code-review`

Before Phase 0, inspect the current agent's global skills and install only missing dependencies:

```bash
npx --yes skills@latest add mattpocock/skills -g -s <missing skills...> -y --full-depth
```

During skill bootstrap, install from no other repository and install no other skills. Verify every installed `SKILL.md`; stop with the failed command if installation fails. If only `grill-with-docs` is missing, use `grilling` with `domain-modeling`.

A skill installed during the current session may not appear in discovery until restart; read its `SKILL.md` directly from disk and continue.

## Modes

| Mode | Trigger | Approval gates |
|---|---|---|
| `full` | `/autopilot full`, “полный автомат”, “don't ask” | none |
| `semi` | `/autopilot semi` or no mode | brief only |
| `manual` | `/autopilot manual`, “ручной режим” | brief, spec, tickets |

Announce the resolved mode before Phase 1. Default ambiguity to `semi`; if two modes are explicitly requested, ask which one. A mode change applies to the next phase. Put every stack, budget, language, deadline, and scope constraint from the user's brief into the spec.

No mode bypasses approval for deploy, publish, payment, third-party messages, data deletion, or git-history rewriting.

Never promise a countdown or artificial pause: `full` and `semi` continue immediately after their gates, while `manual` waits for explicit approval.

## Pipeline

### 0. Setup

Skip when `docs/agents/issue-tracker.md` already exists. Otherwise run `setup-matt-pocock-skills` with these Autopilot defaults:

- local Markdown tracker under `.scratch/<feature-slug>/`;
- default triage labels when triage is installed;
- single-context domain docs;
- edit the existing `CLAUDE.md` or `AGENTS.md`; if neither exists, create the file native to the current agent.

Derive one short kebab-case `feature-slug` for all artifacts.

### 1. Brief

- `semi`: run `grill-with-docs` for one batch of 5–8 highest-priority blocking product questions; record anything unresolved as `PLACEHOLDER`.
- `manual`: run it until the decision frontier is empty.
- `full`: ask no product questions. Create a self-brief and use `domain-modeling` while doing so.

In `full`, choose reversible options that run locally without paid services or third-party accounts. Mark choices as `ASSUMPTION — принято за пользователя`. Never invent user facts such as prices, copy, accounts, or business rules; mark them `PLACEHOLDER`. Keep unrequested external integrations out of scope.

Ask for decisions, never credentials. Settle testing seams during this phase so `to-spec` does not reopen the interview.

### 2. Spec

Run `to-spec` from the brief and save `.scratch/<feature-slug>/spec.md`. Do not ask new questions; unresolved facts remain `PLACEHOLDER`.

- `manual`: show the spec and wait for explicit approval.
- `semi` and `full`: continue automatically.

### 3. Tickets

Run `to-tickets` and write one dependency-ordered file per ticket under `.scratch/<feature-slug>/issues/`.

- `manual`: keep its quiz and wait for explicit approval.
- `semi` and `full`: skip the quiz, show a plain-language summary, and continue.

Each ticket must be a verifiable vertical slice, name its blockers, and start as `ready-for-agent`.

### 4. Implement

If needed, initialize Git and ignore `.env`. If `HEAD` does not exist, create a baseline commit before the first ticket. Capture that pre-implementation commit for review, then work the unblocked ticket frontier.

For each ticket:

1. Mark it `in progress`.
2. Give one fresh subagent the ticket path and body, relevant spec sections, and relevant code paths.
3. Run `implement`, but use targeted checks only; Phase 6 owns the full suite, and Phase 5 owns `code-review`.
4. On success, mark the ticket `done` and create one commit containing both implementation and status; then report one plain-language progress line.

Parallelize only tickets that touch disjoint files. On failure, mark the ticket `failed` and retry once in a fresh context with the error. Stop after the second failure.

Resume an interrupted run from Git history and ticket statuses; do not maintain a duplicate progress ledger.

### 5. Review

Run `code-review` once from the captured baseline, using the saved spec.

- Missing requirements and hard documented-standard violations become fix tickets and use Phase 4.
- Do one fix round and no second review.
- Do not auto-fix judgement calls or scope creep; report them.

### 6. Finish

Run the full test suite. If it fails, create one fix ticket, use Phase 4, and rerun the suite once; stop if it still fails. Report in the user's language:

- assumptions made in `full`;
- what works and the exact run command;
- out-of-scope items;
- review findings fixed and left open;
- placeholders, manual steps, and empty environment-variable names;
- paths to the spec and ticket directory.

## Safety

Never request, store, repeat, commit, or pass secret values to subagents; use variable names only and keep `.env` ignored. If a secret reaches a file or commit, stop and advise rotation.

Stop on a failed dependency install, conflicting explicit modes, a ticket failing twice, a leaked secret, a final suite failing after its fix attempt, or an action awaiting an approval gate above.
