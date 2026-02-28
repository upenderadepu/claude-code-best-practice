---
description: Track Claude Code subagents report changes and find what needs updating
argument-hint: [number of versions to check, default 10]
---

# Workflow Changelog — Subagents Report

You are a coordinator for the claude-code-best-practice project. Your job is to launch two research agents in parallel, wait for their results, merge findings, and present a unified report about drift in the **Subagents Reference** report (`reports/claude-subagents.md`).

**Versions to check:** `$ARGUMENTS` (default: 10 if empty or not a number)

This is a **read-then-report** workflow. Launch agents, merge results, and produce a report. Only take action if the user approves.

---

## Phase 0: Launch Both Agents in Parallel

**Immediately** spawn both agents using the Task tool **in the same message** (parallel launch):

### Agent 1: workflow-changelog-claude-subagents-agent

Spawn using `subagent_type: "workflow-changelog-claude-subagents-agent"`. Give it this prompt:

> Research the claude-code-best-practice project for subagents report drift. Check the last $ARGUMENTS versions (default: 10).
>
> Fetch these 3 external sources:
> 1. Sub-agents Reference: https://code.claude.com/docs/en/sub-agents
> 2. CLI Reference: https://code.claude.com/docs/en/cli-reference
> 3. Changelog: https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
>
> Then read the local report file (`reports/claude-subagents.md`) and the CLAUDE.md file. Analyze differences between what the official docs say about agent frontmatter fields and what our report documents. Return a structured findings report covering missing fields, changed field types, new field additions, deprecated fields, memory scope changes, invocation pattern changes, scope/priority changes, and example accuracy.

### Agent 2: claude-code-guide

Spawn using `subagent_type: "claude-code-guide"`. Give it this prompt:

> Research the latest Claude Code agents/subagents system. I need you to find:
> 1. The complete list of all currently supported agent frontmatter fields (name, description, tools, model, etc.) with their types and descriptions
> 2. Any new agent frontmatter fields introduced in recent Claude Code versions
> 3. Changes to existing field behavior (e.g. new model aliases, new permission modes, new tool syntax)
> 4. Changes to agent memory scopes or memory behavior
> 5. Changes to agent invocation patterns (Task tool syntax, --agents CLI flag)
> 6. New agent features (isolation, background, hooks, mcpServers, skills preloading)
> 7. Changes to agent scope/priority resolution order
> 8. Any deprecations or removals of agent frontmatter fields
>
> Be thorough — search the web, fetch docs, and provide concrete version numbers and details for everything you find.

Both agents run independently and will return their findings.

---

## Phase 0.5: Read Verification Checklist

**While agents are running**, read `changelog/reports/claude-subagents/verification-checklist.md`. This file contains accumulated verification rules — each rule specifies what to check, at what depth, and against which source. Every rule MUST be executed during Phase 2. The checklist is the project's regression test suite for drift detection.

---

## Phase 1: Read Previous Changelog Entries

**Before merging findings**, read the file `changelog/reports/claude-subagents/changelog.md` to get the last 25 changelog entries. Each entry is separated by `---`. Parse the priority actions from those previous entries so you can compare them against the current findings. This lets you identify:
- **Recurring items** — issues that appeared before and are still unresolved
- **Newly resolved items** — issues from previous runs that are now fixed
- **New items** — issues that appear for the first time in this run

---

## Phase 2: Merge Findings & Generate Report

**Wait for both agents to complete.** Once you have:
- **workflow-changelog-claude-subagents-agent findings** — detailed report analysis with local file reads, external doc fetches, and drift detection
- **claude-code-guide findings** — independent research on latest Claude Code agents features and frontmatter changes

Cross-reference the two. The dedicated agent provides report-specific drift analysis, while the claude-code-guide agent may surface things it missed (e.g. very recent changes, undocumented features, or context from web searches). Flag any contradictions between the two for the user to resolve.

**Execute the verification checklist:** For every rule in `changelog/reports/claude-subagents/verification-checklist.md`, perform the check at the specified depth using the agent findings as source data. Include a **Verification Log** section in the report showing each rule's result:

```
Verification Log:
Rule # | Category              | Depth         | Result | Notes
1      | Frontmatter Fields    | field-level   | PASS   | All fields match
2      | Memory Scopes         | content-match | FAIL   | New scope added
...
```

**Update the checklist if needed:** If a finding reveals a new type of drift that no existing checklist rule covers (or covers at insufficient depth), append a new rule to `changelog/reports/claude-subagents/verification-checklist.md`. The rule must include: category, what to check, depth level, what source to compare against, date added, and the origin (what error prompted this rule). Do NOT add rules for one-off issues that won't recur.

Also compare the current findings against the previous changelog entries (from Phase 1). For each priority action, mark it as:
- `NEW` — first time this issue appears
- `RECURRING` — appeared in a previous run and is still unresolved (include which run date it first appeared)
- `RESOLVED` — appeared in a previous run but is now fixed (include resolution date)

Produce a structured report with these sections:

1. **New Frontmatter Fields** — Fields in official docs but missing from report, with version introduced
2. **Changed Field Behavior** — Fields whose type, description, or behavior has changed
3. **Deprecated/Removed Fields** — Fields in report but no longer in official docs
4. **Memory Scope Changes** — Updates to memory scope storage, behavior, or new scopes
5. **Invocation Pattern Changes** — Updates to Task tool syntax, CLI flags, or invocation methods
6. **Scope & Priority Changes** — Updates to resolution order or new scope locations
7. **Example Accuracy** — Whether minimal and full-featured examples reflect current field set
8. **Field Type/Description Accuracy** — Per-field verification against official docs
9. **claude-code-guide Agent Findings** — Unique insights from the agent that weren't captured by the dedicated agent. Only include findings that add new information. If there are contradictions between the two agents, flag them for the user to resolve. Do NOT list "confirmed agreements".

End with a prioritized **Action Items** summary table. Each item must include a `Status` column showing `NEW`, `RECURRING (first seen: <date>)`, or `RESOLVED`:

```
Priority Actions:
#  | Type                  | Action                                    | Status
1  | New Field             | Add <field> to frontmatter table           | NEW
2  | Changed Behavior      | Update <field> description                 | NEW
3  | Deprecated Field      | Remove <field> from table                  | RECURRING (first seen: 2026-02-20)
4  | Memory Scope          | Update memory scope table                  | NEW
5  | Example Update        | Update full-featured example               | NEW
6  | Invocation Change     | Update Task tool syntax                    | NEW
7  | Scope/Priority        | Update resolution order table              | NEW
```

Also include a **Resolved Since Last Run** section listing any items from the previous run that are no longer issues.

---

## Phase 2.5: Append Summary to Changelog

**This phase is MANDATORY — always execute it before presenting the report to the user.**

Read the existing `changelog/reports/claude-subagents/changelog.md` file, then **append** (do NOT overwrite) a new entry at the end. The entry format must be exactly:

```markdown
---

## [<YYYY-MM-DD HH:MM AM/PM PKT>] Claude Code v<VERSION>

| # | Priority | Type | Action | Status |
|---|----------|------|--------|--------|
| 1 | HIGH/MED/LOW | <type> | <action description> | <status> |
| ... | ... | ... | ... | ... |
```

**Status format — MUST use one of these three formats:**
- `COMPLETE (reason)` — action was taken and resolved successfully
- `INVALID (reason)` — finding was incorrect, not applicable, or intentional
- `ON HOLD (reason)` — action deferred, waiting on external dependency or user decision

The `(reason)` is mandatory and must briefly explain what was done or why.

**Rules for appending:**
- Always append — never overwrite or replace previous entries
- The date and time is when the command is executed in Pakistan Standard Time (PKT, UTC+5); get it by running `TZ=Asia/Karachi date "+%Y-%m-%d %I:%M %p PKT"`. The version comes from agent findings
- If `changelog/reports/claude-subagents/changelog.md` doesn't exist or is empty, create it with the Status Legend table (see top of file) then the first entry
- Each entry is separated by `---`
- **Only include items with HIGH, MEDIUM, or LOW priority** — omit NONE priority items (things that need no action)

---

## Phase 2.6: Update Last Updated Badge

**This phase is MANDATORY — always execute it immediately after Phase 2.5, before presenting the report.**

Update the "Last Updated" badge at the top of `reports/claude-subagents.md`. Run `TZ=Asia/Karachi date "+%b %d, %Y %-I:%M %p PKT"` to get the time, URL-encode it (spaces to `%20`, commas to `%2C`), and replace the date portion in the badge. Also update the Claude Code version in the badge if it has changed.

**Do NOT log badge updates as action items in the changelog or report.** Badge syncing is a routine part of every run, not a finding.

---

## Phase 3: Offer to Take Action

After presenting the report (and confirming both changelog and badge were updated), ask the user:

1. **Execute all actions** — Handle everything (add missing fields, update descriptions, fix examples)
2. **Execute specific actions** — User picks which numbers to execute
3. **Just save the report** — No changes

When executing:
- **New fields**: Add to the Frontmatter Fields table with correct type, required status, and description
- **Changed behavior**: Update the field description in the table
- **Deprecated fields**: Confirm with user before removing
- **Memory scope changes**: Update the Memory Scopes table
- **Example updates**: Update both minimal and full-featured examples to reflect current field set
- **Invocation changes**: Update the Invocation section
- **Scope/priority changes**: Update the Scope and Priority table
- After all actions, re-run verification to confirm consistency

---

## Critical Rules

1. **Launch BOTH agents in parallel** in a single message — never sequentially
2. **Wait for both agents** before generating the report
3. **Never guess** versions or dates — use data from the agents
4. **New frontmatter fields are HIGH PRIORITY** — they require table, example, and CLAUDE.md updates
5. **Cross-reference field counts** — the number of fields in the table must match official docs
6. **Don't auto-execute** — always present the report first
7. **ALWAYS append to changelog** — Phase 2.5 is mandatory. Never skip it. Never overwrite previous entries.
8. **Compare with previous runs** — read the last 25 entries from the changelog and mark each action item as NEW, RECURRING, or RESOLVED.
9. **ALWAYS execute the verification checklist** — read the verification-checklist.md and execute every rule. Include a Verification Log in the report. Append new rules when a new type of drift is discovered.
10. **Checklist rules are append-only** — never remove or weaken existing rules. Only add new rules or upgrade depth levels.
11. **ALWAYS update the Last Updated badge** — Phase 2.6 is mandatory. Never skip it.
