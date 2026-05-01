# DocWhisperer

> The document whisperer: listens to the pulse of code, gently awakens sleeping docs.

AI-powered documentation alignment plugin for Claude Code.

## What It Does

When code changes, documentation drifts. DocWhisperer analyzes your commits, identifies which documentation files are affected, and helps you keep them in sync.

## Installation

```bash
# Add this repo as a marketplace
claude plugin marketplace add https://github.com/F2077/doc-whisperer

# Install the plugin
claude plugin install doc-whisperer
```

## Usage

Run `/doc-align` in your Claude Code session:

```
> /doc-align              # Analyze last commit
> /doc-align HEAD~3..HEAD # Analyze a range of commits
```

DocWhisperer will:

1. **Identify docs** — intelligently find all documentation files (any format: .md, .rst, .txt, etc.)
2. **Classify changes** — categorize your code changes by impact type
3. **Generate report** — show which docs need updating and why
4. **Confirm** — ask for your approval before making any changes
5. **Execute** — apply targeted updates to affected sections only

## Design Principles

- **On-demand only** — no automatic hooks or background processes. You run it when you want it.
- **Format agnostic** — detects documentation in any text format, not just Markdown.
- **Surgical updates** — edits affected sections only, preserves existing writing style.
- **Never creates files** — only aligns existing documentation.
- **Always confirms** — you see the report and approve before any changes.

## Plugin Structure

```
plugins/doc-whisperer/
├── .claude-plugin/plugin.json     Plugin identity
├── commands/doc-align.md          /doc-align command
└── skills/doc-whisperer/SKILL.md  Core analysis skill
```

## License

MIT
