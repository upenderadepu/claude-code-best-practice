# Verification Checklist — Subagents Report

Rules accumulate over time. Each workflow-changelog run MUST execute ALL rules at the specified depth. When a new type of drift is caught that an existing rule should have caught (but didn't exist or was too shallow), append a new rule here.

## Depth Levels

| Depth | Meaning | Example |
|-------|---------|---------|
| `exists` | Check if a section/table/file exists | "Does the report have a Memory Scopes table?" |
| `presence-check` | Check if a specific item is present or absent | "Is the `color` field in the Frontmatter Fields table?" |
| `content-match` | Compare actual values word-by-word against source | "Does the `model` field description match official docs?" |
| `field-level` | Verify every individual field is accounted for | "Does each frontmatter field from official docs appear in the table?" |
| `cross-file` | Same value must match across multiple files | "Does CLAUDE.md agent section match the report's field list?" |

---

## 1. Frontmatter Fields Table

Rules that verify the Frontmatter Fields table against official docs.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 1A | Field Completeness | For each agent frontmatter field in official docs, verify it appears in the report's Frontmatter Fields table | field-level | sub-agents reference page | 2026-02-28 | Initial checklist — ensures no new fields are missed |
| 1B | Field Types | For each field in the table, verify the Type column matches official docs | content-match | sub-agents reference page | 2026-02-28 | Initial checklist — type mismatches cause user confusion |
| 1C | Required Status | For each field, verify the Required column matches official docs | content-match | sub-agents reference page | 2026-02-28 | Initial checklist — wrong required status causes broken agents |
| 1D | Field Descriptions | For each field, verify the Description column accurately reflects official docs behavior | content-match | sub-agents reference page | 2026-02-28 | Initial checklist — stale descriptions mislead users |

---

## 2. Memory Scopes

Rules that verify the Memory Scopes table.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 2A | Scope Completeness | Verify all memory scopes from official docs appear in the Memory Scopes table | field-level | sub-agents reference page | 2026-02-28 | Initial checklist — new scopes could be added |
| 2B | Storage Locations | For each scope, verify the Storage Location column matches official docs | content-match | sub-agents reference page | 2026-02-28 | Initial checklist — wrong paths cause data loss |

---

## 3. Examples

Rules that verify example accuracy.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 3A | Minimal Example | Verify the minimal example uses only required fields with valid syntax | content-match | sub-agents reference page | 2026-02-28 | Initial checklist — minimal example should stay minimal |
| 3B | Full-Featured Example | Verify the full-featured example demonstrates ALL available frontmatter fields | field-level | sub-agents reference page | 2026-02-28 | Initial checklist — full example must show every field |

---

## 4. Scope & Priority

Rules that verify scope and priority information.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 4A | Priority Order | Verify the Scope and Priority table lists all agent locations in correct priority order | content-match | sub-agents reference page + CLI reference page | 2026-02-28 | Initial checklist — wrong priority order causes resolution bugs |

---

## 5. Cross-File Consistency

Rules that verify consistency between the report and other repo files.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 5A | CLAUDE.md Sync | Verify CLAUDE.md's Subagent Definition Structure section lists the same fields as the report's Frontmatter Fields table | cross-file | CLAUDE.md vs report | 2026-02-28 | Initial checklist — CLAUDE.md could drift from report |

---

## 6. Process

Meta-rules about the workflow verification process itself.

| # | Category | Check | Depth | Compare Against | Added | Origin |
|---|----------|-------|-------|-----------------|-------|--------|
| 6A | Source Credibility Guard | Only flag items as drift if confirmed by official sources (sub-agents reference page, CLI reference page, GitHub changelog). Third-party blog sources may be outdated or wrong — use them for leads only, verify against official docs before flagging | content-match | official docs only | 2026-02-28 | Adopted from voice-hooks workflow — prevents false positives from blog sources |
