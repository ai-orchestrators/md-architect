---
name: md-architect
description: Architect, build, audit, review, refactor, or compress CLAUDE.md, GEMINI.md, AGENTS.md, MEMORY.md, and .claude/rules/*.md files for any Claude Code or multi-harness project. Use when creating a new CLAUDE.md, splitting CLAUDE.md from MEMORY.md, scoping rules with `paths:` frontmatter for conditional loading, deploying multi-harness (CLAUDE.md + GEMINI.md + AGENTS.md), wiring `@include` directives, adding auto-managed agent/skill index blocks, or auditing against the 200-line / 25KB / 40K-char thresholds. TRIGGERS on "architect CLAUDE.md", "design CLAUDE.md", "build CLAUDE.md", "review my CLAUDE.md", "audit CLAUDE.md", "compress CLAUDE.md", "refactor CLAUDE.md", "fix CLAUDE.md", "CLAUDE.md too long", "CLAUDE.md not loading", "rules not firing", "/init wasn't enough", "improve CLAUDE.md", "set up .claude/rules", "scope rules to specific paths", "md-architect".
---

# md-architect

Authoring, auditing and refactoring CLAUDE.md files. CLAUDE.md is the framework's **control panel**. It loads on every turn. Every line is a tax.

## Mental Model

1. **Loaded every turn.** Every word costs context on every conversation.
2. **It's a map, not a manual.** Detail lives in agents, skills, and `.claude/rules/`.
3. **First 20 lines set the model's mental frame.** Get the framing right. Everything flows.
4. **Humans read it too.** Operators, clients, future-you. Must work for both audiences.
5. **Less is more.** Target 200 lines. Hard cap 25KB for memdir entrypoint. 40K char warning for regular CLAUDE.md.

## The 6-Layer Memory Architecture

CLAUDE.md is **Layer 1**. Don't put memory-system content in CLAUDE.md. Use the right layer:

| Layer | What | Who writes it | Where |
|---|---|---|---|
| **1. CLAUDE.md** | Standing instructions, rules, structure | You | Project root, `.claude/CLAUDE.md`, `.claude/rules/*.md` |
| **2. Memdir** | Auto-managed memory the agent builds over time | Claude | `~/.claude/projects/<slug>/memory/` (user/feedback/project/reference) |
| **3. Team Memory** | Server-synced shared knowledge | Team (gitleaks scanned) | `~/.claude/projects/<slug>/memory/team/` |
| **4. Session Memory** | Per-session distillation at token thresholds | Forked sub-agent | `~/.claude/projects/<slug>/sessions/<id>/memory.md` |
| **5. Extracted Memories** | Post-turn distillation into memdir | Background agent | Memdir |
| **6. Memory Prefetch** | Sonnet picks ≤5 relevant memories per turn | Sonnet selector | Injected at runtime |

**Key implication:** memory prefetch reads memdir file titles + first lines, then picks ≤5 to load fully. So write memory files with a clear first line. Don't bury the lede.

## CLAUDE.md vs MEMORY.md vs settings.json vs Memdir

The biggest mistake newcomers make is putting the wrong content in CLAUDE.md. Use this split:

| File | What it holds | Who writes it |
|---|---|---|
| **CLAUDE.md** | Rules, structure, navigation, voice, persona | You |
| **MEMORY.md** (memdir entrypoint) | Index of cross-session memory files | Auto + you |
| **settings.json** | Hooks, permissions, env vars, model selection | You |
| **Memdir files** | Cross-session learnings (user/feedback/project/reference) | Claude (auto) |

**Rules of thumb:**
- "From now on, when X happens, do Y" goes in **hooks** in `settings.json`. Not CLAUDE.md.
- "I prefer en-GB spelling" goes in **CLAUDE.md** (project-wide rule).
- "Last week the user said don't mock the database" goes in a **memdir feedback file**.
- "Allow npm commands without prompt" goes in **permissions** in `settings.json`.

If you put automation rules in CLAUDE.md, they will not fire. The harness executes hooks. Claude reads CLAUDE.md.

## Loading Hierarchy

Claude Code merges memory from multiple sources (later wins, all loaded in parallel):

1. **Managed** (`/etc/claude-code/CLAUDE.md` and `/etc/claude-code/.claude/rules/*.md`). Enterprise.
2. **User** (`~/.claude/CLAUDE.md` and `~/.claude/rules/*.md`). Global personal.
3. **Project** (multiple paths merged, walks from CWD up to root):
   - `CLAUDE.md` at each directory in the walk
   - `.claude/CLAUDE.md` at each directory
   - `.claude/rules/*.md` at each directory
4. **Local** (`CLAUDE.local.md`). Gitignored personal override. Highest priority.
5. **Memdir** entrypoint (auto-memory).
6. **Team memory** (if enabled).

**Disable knobs:**
- Env: `CLAUDE_CODE_DISABLE_CLAUDE_MDS=1` turns off all CLAUDE.md loading.
- Setting: `claudeMdExcludes` (glob array) excludes specific paths.
- Flag: `--bare` skips auto-discovery but honours explicit `--add-dir`.

## Hard Rules (non-negotiable)

| Rule | Why |
|---|---|
| **MEMORY.md cap: 200 lines / 25KB.** Past this is **hard truncated** mid-line. | Truncation is real. Keep MEMORY.md as a one-line-per-entry index. |
| **CLAUDE.md soft cap: 40,000 chars.** Past this triggers a UI warning. | No auto-truncation but performance and clarity degrade. |
| **Never modify CLAUDE.md mid-session.** | Permanently breaks prompt cache. Set before starting. |
| **No automation rules in CLAUDE.md prose.** | Hooks live in `settings.json`. |
| **No duplicating project state files, agent files, or skill bodies.** | Point. Don't copy. |
| **`@include` recursion limit: 5 deep.** | Past depth, files are silently skipped. |

## Frontmatter Tricks

Frontmatter is **stripped from content** before sending to the model. So metadata is free.

### `paths:` for conditional loading

The killer feature most teams miss. Scope a CLAUDE.md or rules file to specific globs:

```markdown
---
paths:
  - src/frontend/**
  - src/components/**
---

Use React 18 patterns. Functional components with hooks. Tailwind for styling.
```

This file only loads when the conversation touches matching files. Activation persists for the session.

**Where it works:**
- Child-directory CLAUDE.md files (loaded conditionally based on `paths:`)
- `.claude/rules/*.md` files (lazy-loaded; without `paths:`, loaded eagerly)

**Use it for:**
- Frontend-only React rules that don't pollute backend sessions
- Test-only conventions (`paths: ["**/*.test.ts"]`)
- Migration guides that only fire when touching legacy code
- Per-feature voice guides

### HTML comments are stripped

Block-level `<!-- ... -->` comments are stripped before the model sees CLAUDE.md. Use them for author notes:

```markdown
## Core Principles

<!-- Reviewed 2026-04 by the framework owner. Update when adding new onboarding steps. -->

1. Fix before build.
2. Revenue first.
```

The model sees the principles. The note is free.

### `@include` directives

Inline another file's content:

```markdown
@./agents/how-to-use-agents.md
@./skills/how-to-use-skills.md
@~/.claude/shared/coding-standards.md
@/absolute/path/to/file.md
```

**Syntax:**
- `@./relative/path` (or just `@path`, treated as relative)
- `@~/home/path`
- `@/absolute/path`
- Works in leaf text nodes only (NOT inside code blocks)
- Strips fragment identifiers (`@file.md#section` becomes `@file.md`)
- Whitelisted text extensions only (`.md`, `.txt`, `.yaml`, `.json`, etc.). Binary files skipped.
- Circular references blocked.
- Max depth: 5.

## Multi-Harness Deployment

As of Apr 2026, frameworks deploy to multiple harnesses. CLAUDE.md is no longer the only target.

| Harness | File | Notes |
|---|---|---|
| Claude Code | `CLAUDE.md` | Primary. Supports `paths:` frontmatter and `@include`. |
| Gemini CLI | `GEMINI.md` | Uses `@imports/` for modular sections. |
| Codex (GPT-5.x) | `AGENTS.md` + `.codex/config.toml` | Codex reads AGENTS.md. config.toml sets behaviour. |

**Deploy rule:** all three files must be regenerated in lock-step. Don't hand-edit one without the others. Use `BEGIN/END AUTO-MANAGED` markers (below) so regeneration is safe.

## AUTO-MANAGED Blocks

Sections that the deploy script regenerates (agent index, skill index, MCP list) MUST be wrapped in markers:

```markdown
## Agents

<!-- BEGIN AUTO-MANAGED: agent-index -->
| Agent | Codename | Model | Phase/Purpose |
|-------|----------|-------|---------------|
| ...auto-generated... |
<!-- END AUTO-MANAGED: agent-index -->
```

**Rules:**
- Anything between markers is overwritten on every deploy.
- Hand-edit OUTSIDE the markers, never inside.
- Each auto-managed section gets a unique ID (`agent-index`, `skill-index`, `mcp-list`).
- Markers are HTML comments, so they're stripped from model context. Free.

## .claude/rules/ Directory Pattern

Splitting rules into a directory beats one giant CLAUDE.md.

```
.claude/
├── CLAUDE.md              # Project map. Always loaded.
└── rules/
    ├── voice.md           # Always loaded (no paths:)
    ├── frontend.md        # Conditional (paths: src/frontend/**)
    ├── backend.md         # Conditional (paths: src/backend/**)
    ├── tests.md           # Conditional (paths: **/*.test.ts)
    └── deploy.md          # Conditional (paths: ops/**, .github/**)
```

**Why this pattern wins:**
- Eager rules stay short. Conditional rules don't pollute when irrelevant.
- Per-domain context, no master CLAUDE.md bloat.
- Clear ownership. Frontend team owns `frontend.md`, backend owns `backend.md`.

## Two CLAUDE.md Contexts in Frameworks

**Framework-level (master source).** Lives in the framework repo. Provides context when developing the framework itself. NOT deployed.

Must include at the top:

```markdown
> **YOU ARE IN THE MASTER FRAMEWORK SOURCE.** This is NOT a deployed client project.
> Work done here improves the framework for all future deployments.

| Context | Purpose | State file | CLAUDE.md |
|---------|---------|-----------|-----------|
| **Here (master source)** | Improve the framework | None | THIS file |
| **Deployed client project** | Deliver outcomes | project state YAML | Generated by init agent |
```

**Project-level (deployed)** is generated fresh by the init agent per deployment. Unique per client. Contains client context.

The deploy script must EXCLUDE the master CLAUDE.md from copy. The init agent always generates the project-level one fresh.

## CLAUDE.md Anatomy

### Required sections

```markdown
# [Framework Name]

## Project Overview
[2-3 sentences: what it is, who for, methodology]

## Client Context
[Who, what they do, where they are, why here, current focus]

## Who Claude Is
[Persona, voice, accountability rules. Short. See template below.]

## Key Files
[3-5 must-read files. The project state file goes here.]

## Agents
<!-- BEGIN AUTO-MANAGED: agent-index -->
[Auto-generated table]
<!-- END AUTO-MANAGED: agent-index -->

## Skills
<!-- BEGIN AUTO-MANAGED: skill-index -->
[Auto-generated grouped list]
<!-- END AUTO-MANAGED: skill-index -->

## Core Principles
[3-10 rules every agent must follow]

## Voice & Comms Style
[Language, formatting, banned words. See template below.]

## Navigation
[Links to how-to-use guides and `.claude/rules/`]
```

### Optional sections (when relevant)

```markdown
## Blocking Gates
## CLI / Commands
## Claude Code Performance Notes
## External References (MEMORY.md, portfolio manifest pointers)
```

## Section Templates

### Project Overview

```markdown
**Program:** [Framework name]
**Purpose:** [One sentence. What it does for whom.]
**Methodology:** [Named methodology + phase summary]
```

Under 10 lines.

### Client Context

Critical section. Without it, agents must read the project state file before doing anything.

```markdown
**Client:** [Name] | **Company:** [Company]
**Industry:** [Industry] | **Stage:** [Stage]

### The Business
[2-3 sentences: what they do, who they serve, how they make money]

### Current Situation
- **Revenue model:** [subscription / service / etc.]
- **Team:** [size, key roles]
- **Key metrics:** [customers, revenue, growth, churn]
- **Primary constraint:** [main growth blocker]

### Engagement Focus
- **90-day priority:** [what we're solving first]
- **Current phase:** [methodology phase]
- **Active workstreams:** [what's running now]

Read the project state file for full state.
```

Target 15-25 lines.

### Who Claude Is (persona)

```markdown
## Who Claude Is

Claude = [role for this project]. Not a mirror. Not passive.

### Identity
- **Vibe:** [Direct, blunt, results-obsessed, etc.]
- **Role:** [What Claude owns]
- **Voice:** [Regional English variant, sentence length, formatting preference]

### Operating Principles
1. [Principle 1]
2. [Principle 2]

### Anti-Patterns (Never)
- [Specific don'ts]
```

Load-bearing. Don't skip it.

### Voice & Comms Style

```markdown
## Voice & Comms Style

- **Language:** [AUS / US / UK English]
- **Currency:** [AUD / USD / etc.]
- **Reading level:** [e.g. third-grade. Genius 10-year-old energy.]
- **Em dashes:** [BANNED. Use full stops, commas, colons.]
- **Defaults:** Bullets and tables over prose. Bold the key phrase per bullet.
- **Banned words:** leverage, synergy, holistic, robust, navigate, tapestry.
- **Lead with the answer.** No "Great question." Just say it.
```

Skip if generic. Use when the project has a strong voice.

### Key Files

```markdown
- `<project-state>.yaml`. Program state. Read before every decision.
- `docs/ip-vault/`. Extracted client IP.
- `MEMORY.md`. Cross-session learnings index.
- `.claude/rules/`. Conditional and unconditional rules.
```

Rule: if an agent reads it less than once per session, it doesn't belong here.

### Core Principles

3-10 rules. Actionable, universal, testable.

**Good:** "Fix before build." "One bottleneck at a time." "Revenue first."
**Bad:** "Strive for excellence." "Write good code."

### Navigation

```markdown
- `agents/how-to-use-agents.md`
- `skills/how-to-use-skills.md`
- `.claude/rules/` (frontend.md, backend.md, voice.md...)
- `MEMORY.md` (cross-session memory index)
- `<portfolio-manifest>.md` (portfolio-level manifest, if applicable)
```

## Anti-Patterns

| Pattern | What it looks like | Fix |
|---|---|---|
| **The Novel** | 300+ lines covering every scenario | Move detail to agents/skills/`.claude/rules/`. CLAUDE.md is a map. |
| **The Empty Shell** | Project name + nothing else | Add Client Context, Key Files, Core Principles. |
| **The Stale Index** | Lists agents/skills that don't exist | Wrap in AUTO-MANAGED markers. Regenerate on deploy. |
| **The Duplicate** | Repeats agent file content | Point to the agent file. |
| **The Generic Template** | Could belong to any project | Use THIS client's methodology, names, principles. |
| **The Hook Mistake** | "Always do X when Y" rules in prose | Move to `settings.json` hooks. They won't fire from CLAUDE.md. |
| **The Cache Killer** | Edited mid-session for "small fixes" | Set before session start. Restart Claude after edits. |
| **The MCP Overload** | 15 MCPs always-on, CLAUDE.md unaware | Each MCP ~17.6K tokens. Document which to disconnect when not needed. |
| **The Memory Confusion** | "Last quarter we decided X" baked into CLAUDE.md | That's a memdir feedback entry. CLAUDE.md is for rules, not events. |
| **The Eager Universalist** | Frontend rules loaded for backend sessions | Use `paths:` frontmatter to scope rules to globs. |
| **The Bloated Index** | MEMORY.md > 200 lines | Past 200 lines is hard-truncated. Keep entries to one line. |
| **The Frontmatter Mismatch** | `whenToUse`, `allowedTools` (camelCase) | Spec is kebab-case: `when_to_use`, `allowed-tools`, `paths`. |

## Sizing Guide

| File | Target | Hard Limit |
|---|---|---|
| Root `CLAUDE.md` | 80-200 lines | 40K char (warning) |
| Per-rule file (`.claude/rules/*.md`) | Whatever the rule needs | 40K char (warning) |
| `MEMORY.md` (memdir entrypoint) | <150 lines | **200 lines / 25KB hard truncation** |
| Memdir individual files | Short, scannable | No hard cap |

If CLAUDE.md hits 200 lines, audit before cutting Client Context or Who Claude Is. Cut agent tables (move to AUTO-MANAGED), navigation links, optional sections first. Then split rules into `.claude/rules/`.

## Performance Considerations

- **Each MCP server ~17.6K tokens.** A 100-line CLAUDE.md with 15 MCPs = 264K of MCP overhead before any work starts. Document which MCPs to disconnect.
- **Prompt cache TTL: 5 min idle.** Long pauses bust the cache.
- **Cache invalidation:** ANY edit to CLAUDE.md mid-session permanently busts cache for that session.
- **Subagents inherit CLAUDE.md.** Bloat propagates.
- **Memory prefetch reads first lines.** Title and lead-in matter for memdir files (Sonnet selector reads only the head).

## /init vs md-architect

| Use `/init` | Use this skill |
|---|---|
| Bootstrapping a brand-new project | Authoring a framework-source CLAUDE.md |
| Standard codebase docs | Per-client deployed CLAUDE.md with custom voice + persona |
| Quick scaffold | Auditing, compressing, refactoring an existing one |
| One-off | Multi-harness deployment (CLAUDE.md + GEMINI.md + AGENTS.md) |
| Single root CLAUDE.md | Splitting rules into `.claude/rules/` with `paths:` |

`/init` writes a generic codebase summary. This skill writes the operational map for a specific framework with persona, voice, conditional rules, and AUTO-MANAGED indexes.

## Audit Checklist (use when reviewing existing CLAUDE.md)

Run through this list when asked to "review" or "audit" a CLAUDE.md:

1. **Size.** Lines. Bytes. Within 200 / 40K?
2. **First 20 lines.** Do they set the right mental frame?
3. **Client Context present?** Or is it just framework boilerplate?
4. **Who Claude Is present?** Persona, voice, principles?
5. **Auto-managed blocks present?** Will deploys be safe?
6. **Conditional rules used?** Or is everything loaded eagerly?
7. **MCP awareness?** Does it call out which MCPs to disconnect?
8. **Hooks NOT in prose?** Anything that should be in `settings.json`?
9. **Memory NOT in prose?** Anything that should be in memdir?
10. **`@include` paths valid?** Files exist? Under depth 5?
11. **No duplication of agent/skill bodies?**
12. **Regional language consistent, em dash policy enforced** (or whatever the project's Voice section dictates)?

## Sync After Edit

When you edit a framework-source CLAUDE.md, it drifts from deployed copies. Two paths:

1. **Manual.** Run the framework's deploy script for each affected client.
2. **Scheduled.** Add to your sync routine (e.g. a weekly cron) so deploys happen on a schedule.

If you maintain a portfolio-level manifest of CLAUDE.md/skills/MCPs/plugins, log the change there same turn.
