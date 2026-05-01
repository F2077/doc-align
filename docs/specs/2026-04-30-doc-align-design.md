# DocAlign - Design Spec

## Problem

Code evolves continuously; documentation often lags behind. After significant modifications to a project, docs (README, API docs, architecture guides, CHANGELOG, etc.) can diverge from the actual codebase — causing onboarding friction, API misuse, and architectural misunderstandings.

## Solution

A Claude Code plugin that analyzes recent code changes against project documentation, generates an impact report, and updates docs after user confirmation. Invoked on-demand via `/doc-align` — no background hooks, no automatic triggers.

## Architecture

```
doc-align/                          # GitHub repository (also a marketplace)
├── .claude-plugin/
│   └── marketplace.json              # Marketplace manifest
├── plugins/
│   └── doc-align/
│       ├── .claude-plugin/
│       │   └── plugin.json          # Plugin identity
│       ├── commands/
│       │   └── doc-align.md         # /doc-align command
│       └── skills/
│           └── doc-align/
│               └── SKILL.md         # Core analysis skill
├── docs/                             # Design docs (not part of plugin)
└── README.md
```

## Trigger Mechanism

### Manual: /doc-align Command

The sole entry point. Users run `/doc-align` when they want to check documentation alignment.

- **No args**: analyze the latest commit
- **With args**: e.g. `/doc-align HEAD~3..HEAD` to analyze a commit range

### Why No Automatic Trigger

Earlier versions used a PostToolUse hook to auto-detect commits. This was removed because:

1. Hook output is JSON context injected into AI context — not visible to users
2. Frequent commits would waste tokens on AI analysis each time
3. Context pollution from repeated reports degrades conversation quality
4. Alert fatigue: no human can sustain attention over hundreds of auto-generated reports

Users who want automation can compose with other plugins or CI tools.

## AI Analysis Flow

### Step 1: Identify Documentation Files

AI scans the project structure and identifies documentation files using judgment, not hardcoded patterns:

- **Location**: `docs/`, `doc/`, root-level files (README, CHANGELOG, etc.)
- **Format**: any text format — `.md`, `.rst`, `.adoc`, `.txt`, `.html`, `.tex`, `.org`, etc.
- **Content**: files describing the project, not implementing it
- **Exclusion**: source code in doc-like formats (e.g., blog engine `.md` posts)

### Step 2: Change Classification

AI categorizes the commit diff into:

| Category | Description | Doc Impact |
|----------|-------------|------------|
| Structural | File add/delete, directory reorg, module split | High |
| API | Interface signatures, params, new/removed endpoints | High |
| Logic | Core algorithms, business logic, flow changes | Medium |
| Config | Dependency updates, build config, env vars | Medium |
| Style/Format | Non-behavioral changes | None (skip) |

### Step 3: Document Impact Assessment

For each identified documentation file, AI determines:

- Whether the doc is affected by this change
- Impact level: HIGH / MED / LOW / OK
- Which specific sections need modification

### Step 4: Report Generation

```
── DocAlign Report ──────────────────────────────

Commit: <hash> <subject>
Scope: <categories>
Docs identified: <count> (<formats>)

Documents to update:
  [HIGH] path/to/doc — description
  [MED]  path/to/other — description
  [LOW]  path/to/extra — description

No update needed:
  [OK]   path/to/fine

─────────────────────────────────────────────────────
```

### Step 5: User Confirmation (Mandatory)

`AskUserQuestion` presents three options:

- Update all suggested docs
- Let me choose which to update (multi-select)
- Skip this time

### Step 6: Execute Updates

Surgical edits to affected sections only. Preserves existing writing style and file structure.

## Edge Cases

| Scenario | Handling |
|----------|----------|
| Non-git repo | Command shows error |
| First commit (no HEAD~1) | Cannot diff, shows message |
| No documentation files found | Report shows "no docs identified" |
| All changes are Style/Format | Skip silently |
| Huge diff (>5000 lines) | Stat-only analysis, skip per-line diff |
| Doc is in source format (blog `.md`) | AI excludes based on content analysis |

## Design Principles

- **On-demand only** — no hooks, no background processes
- **Format agnostic** — AI identifies docs regardless of file extension
- **Surgical updates** — edit affected sections, preserve style
- **Never creates files** — alignment only
- **Always confirms** — user sees report before any changes

## Distribution

Published as a GitHub repository that doubles as a Claude Code marketplace:

```bash
claude plugin marketplace add https://github.com/<user>/doc-align
claude plugin install doc-align
```

## Scope

- Git repositories only
- Any text-based documentation format
- No binary/non-text file docs
- No greenfield doc generation (alignment only)
