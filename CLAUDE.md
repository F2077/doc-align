# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DocWhisperer is a Claude Code plugin that analyzes code changes and updates documentation to keep it aligned. It is invoked manually via `/doc-align` — there are no automatic hooks or background processes.

## Architecture

This repository is a **marketplace + plugin** combined:

```
doc-whisperer/                    # GitHub repo = Claude Code marketplace
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (publishes plugins/)
├── plugins/
│   └── doc-whisperer/            # The actual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json      # Plugin identity and metadata
│       ├── commands/
│       │   └── doc-align.md     # /doc-align command definition
│       └── skills/
│           └── doc-whisperer/
│               └── SKILL.md     # Core 7-step analysis workflow
└── docs/                         # Design specs (not part of plugin distribution)
```

### How /doc-align works

1. User runs `/doc-align` (with optional commit range like `HEAD~3..HEAD`)
2. Command delegates to the `doc-whisperer` skill
3. Skill executes 7-step workflow: scan docs → gather diff → classify changes → assess impact → generate report → **user confirmation** → execute updates
4. User must explicitly approve via `AskUserQuestion` before any file is modified

### Key Design Principles

- **On-demand only** — no hooks, no auto-triggers
- **Format agnostic** — detects any text-based doc format (.md, .rst, .txt, .adoc, etc.)
- **Surgical edits** — modifies only affected sections, preserves existing writing style
- **Never creates files** — only aligns existing documentation
- **Always confirms** — report shown before any changes

## Working with This Repository

### Installing the plugin for local development

```bash
# Add this repo as a marketplace
claude plugin marketplace add /mnt/g/VSCodeProjects/doc-whisperer

# Install the plugin
claude plugin install doc-whisperer
```

### Testing /doc-align

```bash
# Analyze last commit
/doc-align

# Analyze a commit range
/doc-align HEAD~3..HEAD
```

### Adding new commands

Commands are defined in `plugins/doc-whisperer/commands/*.md` with YAML frontmatter specifying `description` and `allowed-tools`.

### Adding new skills

Skills are defined in `plugins/doc-whisperer/skills/*/SKILL.md`. The skill name in frontmatter (`name`) must match the directory name.

## Development Notes

- Edits to `plugins/doc-whisperer/` require running `claude plugin install doc-whisperer` to pick up changes
- Design decisions and rationale are in `docs/specs/2026-04-30-doc-whisperer-design.md`
- Plans and work tracking are in `docs/plans/`
