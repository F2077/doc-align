---
name: doc-whisperer
description: Use when the user runs /doc-align to check if documentation needs updating after code changes. AI-analyzes the change impact on project docs, generates a detailed report, and updates docs after user confirmation.
---

# DocWhisperer

> The document whisperer: listens to the pulse of code, gently awakens sleeping docs.

## Overview

This skill is invoked via the `/doc-align` command. It analyzes whether recent code changes require documentation updates, presents a detailed report, and applies updates after user confirmation.

## Workflow

### Step 1: Identify Documentation Files

Do NOT assume documentation is `.md` only. Instead, scan the project and use AI judgment to identify which files are documentation:

1. List the project structure:
```bash
find . -maxdepth 3 -not -path "./.git/*" -not -path "./node_modules/*" -not -path "./.cargo/*" -not -path "./.claude/*" | sort
```

2. Identify documentation files based on:
   - **Location**: files in `docs/`, `doc/`, `documentation/`, or root-level docs (README, CHANGELOG, etc.)
   - **Content**: files that describe the project rather than implement it
   - **Format**: any text-based format — `.md`, `.rst`, `.adoc`, `.txt`, `.html`, `.tex`, `.org`, `.pod`, etc.
   - **Convention**: files named README, CHANGELOG, CONTRIBUTING, LICENSE (text), API, etc.

3. **Exclude** files that ARE the project's source code, even if they use doc-like formats. For example:
   - A blog engine's `.md` posts are source code, not docs about the project
   - A static site's content files are source code
   - `.md` files in `src/` that are part of the build pipeline

4. Present the identified doc list briefly so the user can verify if anything was missed or incorrectly included.

### Step 2: Gather Context

Gather commit information:

For `/doc-align` with no args (analyze last commit):
```bash
git log -1 --format="%h %s"
git diff HEAD~1 --stat
git diff HEAD~1 --name-only
```

For `/doc-align HEAD~3..HEAD` (analyze a range):
```bash
git log --oneline HEAD~3..HEAD
git diff HEAD~3..HEAD --stat
git diff HEAD~3..HEAD --name-only
```

### Step 3: Classify Changes

Analyze the diff and classify the commit into one or more categories:

| Category | Examples | Doc Impact |
|----------|----------|------------|
| **Structural** | Files added/removed, dirs reorganized, modules split/merged | High |
| **API** | Function signatures changed, endpoints added/removed, params modified | High |
| **Logic** | Algorithm changes, business rule updates, flow modifications | Medium |
| **Config** | Dependencies added/removed, build config, env vars | Medium |
| **Style/Format** | Whitespace, formatting, comments, no behavior change | None |

**If ALL changes are Style/Format → STOP. No doc update needed. Inform the user briefly and exit.**

**If the total diff is >5000 lines → Skip per-line analysis. Classify based on file names and `--stat` only. Note this limitation in the report.**

### Step 4: Assess Document Impact

For each identified documentation file, determine:

1. **Is it affected?** — Based on what changed vs. what the doc covers
2. **Impact level:**
   - **HIGH**: Doc is now factually incorrect or missing critical info
   - **MED**: Doc could be more accurate but isn't misleading
   - **LOW**: Minor improvement possible
   - **OK**: No update needed
3. **Specific sections** — Which paragraphs/headings need changes
4. **What to change** — Brief description of the required update

To assess accurately, READ the relevant documentation files. Do not guess based on filenames alone.

### Step 5: Generate Report

Present the report in this exact format:

```
── DocWhisperer Report ──────────────────────────────

Commit: <hash> <subject>
Scope: <comma-separated categories>
Docs identified: <count> (<formats found>)

Documents to update:
  [HIGH] path/to/doc — <what's wrong and what to change>
  [MED]  path/to/other — <what could be improved>
  [LOW]  path/to/extra — <minor suggestion>

No update needed:
  [OK]   path/to/fine
  [OK]   path/to/also-fine

─────────────────────────────────────────────────────
```

### Step 6: User Confirmation

**MANDATORY: You MUST call AskUserQuestion to get user approval.**

Present options:
- "Update all suggested docs" — apply all HIGH/MED/LOW changes
- "Let me choose which to update" — multi-select from the list
- "Skip this time" — do nothing

Do NOT modify any documentation file without explicit user confirmation.

### Step 7: Execute Updates

Based on user selection:

1. For each approved doc:
   - Read the current file content
   - Make targeted edits to the affected sections only
   - Do not rewrite entire files — preserve existing structure and style
2. After all edits, briefly summarize what was changed in each file

### Important Rules

- **Never skip Step 6 (user confirmation).** This is always interactive.
- **Never create new documentation files.** This skill only aligns existing docs.
- **Preserve existing writing style.** Match the tone and format of the existing doc.
- **Be specific in reports.** Don't say "README needs updating" — say "README line 23: the install command is missing the `--save-dev` flag for the new dependency."
- **If unsure about impact, mark as LOW** and let the user decide.
- **Do not assume `.md` = documentation.** Let the project context guide identification.
