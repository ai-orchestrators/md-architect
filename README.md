# md-architect

A Claude Code / Claude Cowork skill that architects, audits, and refactors **CLAUDE.md**, **GEMINI.md**, **AGENTS.md**, **MEMORY.md**, and `.claude/rules/*.md` files for any Claude Code or multi-harness project.

## Why this exists

CLAUDE.md is loaded into the model's context on every turn. Most of them are bloated, stale, or written like documentation when they should be a control panel. Common failure modes:

- 300+ lines that quietly burn context budget
- Generic templates that don't match the actual project
- Agent and skill indexes that drift from reality
- "When X happens, do Y" rules in prose that never fire (those need hooks in `settings.json`)
- MEMORY.md indexes that hit the silent 200-line / 25KB hard truncation
- Missing GEMINI.md / AGENTS.md when the project deploys to multiple harnesses

This skill applies the platform's documented behaviour (loading hierarchy, `paths:` frontmatter, `@include` directives, AUTO-MANAGED block patterns, 6-layer memory architecture) and produces a CLAUDE.md that is short, accurate, and stays in sync.

## What you get

The skill walks through the file as a **control panel** and applies these rules:

- **Required sections.** Project Overview, Client Context, Who Claude Is, Key Files, Agents, Skills, Core Principles, Voice & Comms Style, Navigation.
- **Hard caps.** 200-line target / 40K char warning for CLAUDE.md. 200-line / 25KB hard truncation for MEMORY.md.
- **Conditional loading.** `paths:` frontmatter scopes rules to globs so frontend rules don't pollute backend sessions.
- **Multi-harness deployment.** Generates and keeps CLAUDE.md / GEMINI.md / AGENTS.md / `.codex/config.toml` in sync.
- **AUTO-MANAGED blocks.** Wraps agent index / skill index / MCP list in `<!-- BEGIN/END AUTO-MANAGED -->` markers so deploy scripts can regenerate without trampling hand-edits.
- **`.claude/rules/` pattern.** Splits rules by domain (frontend, backend, tests, deploy) for clean conditional loading.
- **Memory split.** CLAUDE.md vs MEMORY.md vs `settings.json` vs memdir. Puts each rule type where it actually fires.

## Use cases

| Trigger | What the skill does |
|---|---|
| "build a CLAUDE.md for this project" | Generates a fresh CLAUDE.md using the right sections and the project's actual structure |
| "review my CLAUDE.md" | Runs the 12-item audit checklist and reports issues |
| "compress my CLAUDE.md" | Cuts to <200 lines without dropping load-bearing context |
| "rules not firing" | Diagnoses whether content belongs in CLAUDE.md, `.claude/rules/`, hooks, or memdir |
| "set up multi-harness deployment" | Generates GEMINI.md and AGENTS.md alongside CLAUDE.md |
| "scope rules to specific paths" | Adds `paths:` frontmatter for conditional loading |

## Installation

### Claude Code (CLI)

User-level (available across all projects):

```bash
git clone https://github.com/ai-orchestrators/md-architect.git ~/.claude/skills/md-architect
```

Project-level (this project only):

```bash
git clone https://github.com/ai-orchestrators/md-architect.git .claude/skills/md-architect
```

### Claude Cowork (Desktop)

Drop the `md-architect/` directory into `~/.claude/skills/`.

### Verify

In any Claude Code session, type `/md-architect` or ask "review my CLAUDE.md". The skill should auto-trigger.

## Anti-patterns this skill catches

- **The Novel.** 300+ lines covering every scenario. Move detail to agents and skills.
- **The Stale Index.** Lists agents/skills that don't exist. Wrap in AUTO-MANAGED markers.
- **The Hook Mistake.** "Always do X when Y" in prose. Move to `settings.json` hooks.
- **The Cache Killer.** Edited mid-session for "small fixes." CLAUDE.md edits permanently bust the prompt cache.
- **The MCP Overload.** 15 MCPs always-on, CLAUDE.md unaware that each is ~17.6K tokens.
- **The Memory Confusion.** "Last quarter we decided X" baked into CLAUDE.md. That's a memdir feedback entry.
- **The Eager Universalist.** Frontend rules loaded for backend sessions. Use `paths:` frontmatter.

## License

MIT
