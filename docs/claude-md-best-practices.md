# CLAUDE.md Best Practices Guide

A comprehensive guide to creating and maintaining effective CLAUDE.md files for Claude Code projects.

## Table of Contents

- [What is CLAUDE.md?](#what-is-claudemd)
- [File Placement Hierarchy](#file-placement-hierarchy)
- [Quick Start with /init](#quick-start-with-init)
- [Content Architecture](#content-architecture)
- [What to Include](#what-to-include)
- [What to Avoid](#what-to-avoid)
- [Advanced Patterns](#advanced-patterns)
- [Context Management](#context-management)
- [The Canary Trick](#the-canary-trick)
- [Anti-Patterns](#anti-patterns)
- [Troubleshooting](#troubleshooting)

---

## What is CLAUDE.md?

CLAUDE.md is a special configuration file that Claude Code automatically loads into context at the start of every conversation. It provides persistent, project-specific context that eliminates the need to repeatedly explain project basics.

**Key principle**: CLAUDE.md files become part of Claude's system prompt, so they should be refined like any frequently used prompt.

### Memory Files vs Settings Files

| Type              | Format   | Purpose                                           |
| ----------------- | -------- | ------------------------------------------------- |
| **CLAUDE.md**     | Markdown | Instructions and context loaded at startup        |
| **settings.json** | JSON     | Permissions, environment variables, tool behavior |

---

## File Placement Hierarchy

Claude Code searches for CLAUDE.md files in multiple locations, loading them hierarchically:

| Location              | Scope                 | Use Case                                 |
| --------------------- | --------------------- | ---------------------------------------- |
| `~/.claude/CLAUDE.md` | Global (all projects) | Personal preferences, universal patterns |
| Parent directories    | Inherited             | Monorepo-wide configuration              |
| Repository root       | Project               | Team-shared project context              |
| Child directories     | On-demand             | Feature-specific overrides               |

### File Variants

| File              | Version Control | Use Case                  |
| ----------------- | --------------- | ------------------------- |
| `CLAUDE.md`       | Committed       | Team-shared configuration |
| `CLAUDE.local.md` | Gitignored      | Personal overrides        |

### Path Linking Strategy

**In Global CLAUDE.md** (`~/.claude/CLAUDE.md`):

- Use absolute paths: `~/.claude/docs/guide.md`
- Reason: Global file is injected into various project contexts

**In Project CLAUDE.md**:

- Use relative paths: `./docs/guide.md`, `../shared/config.md`
- Exception: References to system files should be absolute

---

## Quick Start with /init

The `/init` command generates a starter CLAUDE.md by analyzing your codebase:

```bash
claude /init
```

Claude examines:

- Package files (package.json, requirements.txt, etc.)
- Existing documentation
- Configuration files
- Code structure and patterns

### Post-Init Checklist

The generated file is a starting point, not finished. Review and add:

- [ ] Workflow instructions Claude can't infer
- [ ] Branch naming conventions
- [ ] Deployment processes
- [ ] Code review requirements
- [ ] Team-specific conventions
- [ ] Known gotchas and edge cases

---

## Content Architecture

### The Golden Rule: Facts vs Procedures

| CLAUDE.md                            | Slash Commands                          |
| ------------------------------------ | --------------------------------------- |
| Facts (architecture, file locations) | Procedures (create tests, run refactor) |
| Static context                       | Repeatable workflows                    |
| Loaded every session                 | Invoked on demand                       |

This separation prevents re-sending procedural logic every session.

### Recommended Structure

```markdown
# Project Name

Brief description of what this project does.

## Tech Stack

- **Language**: TypeScript 5.x
- **Framework**: Next.js 14 (App Router)
- **Database**: PostgreSQL with Prisma
- **Testing**: Jest + Playwright

## Key Directories

- `src/app/` - Next.js app router pages and API routes
- `src/components/` - React components
- `src/lib/` - Utility functions and shared code
- `prisma/` - Database schema and migrations

## Commands

- `npm run dev` - Start development server
- `npm run build` - Production build
- `npm run test` - Run unit tests
- `npm run test:e2e` - Run Playwright tests
- `npm run lint` - ESLint + Prettier check

## Code Style

- Use TypeScript strict mode
- Functional components with hooks
- Named exports preferred over default exports
- Use `const` over `let` when possible

## Conventions

- Branch naming: `feature/description`, `fix/description`
- Commit messages: Conventional Commits format
- PRs require one approval before merge

## Important Notes

- Always run `npm run lint` before committing
- Database migrations require `npm run db:push`
- Environment variables are in `.env.local` (not committed)
```

---

## What to Include

### Essential Content

1. **Project Summary** - What is this codebase?
2. **Tech Stack** - Languages, frameworks, versions
3. **Directory Map** - Key locations and their purposes
4. **Commands** - Build, test, lint, deploy
5. **Code Standards** - Formatting, naming, patterns
6. **Conventions** - Branching, commits, reviews

### Advanced Content

1. **Custom Tools** - MCP servers, scripts, and their usage
2. **Domain Knowledge** - Business logic, terminology
3. **Known Issues** - Gotchas, workarounds
4. **File Pointers** - References to detailed docs (not embedded)

### Example: Documenting Custom Tools

```markdown
## Custom Tools

### MCP Servers

- **GitHub MCP**: Use for PR operations, issue management
  - `gh pr create`, `gh issue list`
- **Playwright MCP**: Use for browser automation testing
  - Always close browser after tests

### Scripts

- `scripts/seed-db.sh` - Populate development database
  - Run with: `./scripts/seed-db.sh --env=dev`
  - Use `--help` flag for options
```

---

## What to Avoid

### Never Include

| Content                        | Why           |
| ------------------------------ | ------------- |
| API keys, secrets              | Security risk |
| Connection strings             | Security risk |
| Credentials                    | Security risk |
| Security vulnerability details | Exposure risk |

### Avoid These Patterns

**Embedding entire files**:

```markdown
<!-- BAD: Bloats context window -->
See @docs/full-api-reference.md for details

<!-- GOOD: Path reference -->
See `docs/full-api-reference.md` for the complete API reference.
Claude should read this file when working on API endpoints.
```

**Negative-only constraints**:

```markdown
<!-- BAD: Claude will get stuck -->
Never use the --force flag.

<!-- GOOD: Provide alternative -->
Avoid --force flag. Use --force-with-lease instead for safer force pushes.
```

**Theoretical best practices**:

```markdown
<!-- BAD: Doesn't match reality -->
All functions must have 100% test coverage.

<!-- GOOD: Actual practice -->
Critical business logic requires tests. Utilities tested as needed.
```

---

## Advanced Patterns

### Master Index Pattern

Use CLAUDE.md as a hub pointing to detailed documentation:

```markdown
## Documentation

- **Architecture**: See `docs/architecture.md`
- **API Reference**: See `docs/api/` directory
- **Deployment**: See `docs/deployment.md`

Claude should read relevant docs when working in related areas.
```

### Subagent Workflows

Define when Claude should use subagents:

```markdown
## Subagent Usage

Use subagents for context isolation:

- **Code Review**: Spawn subagent to review without polluting main context
- **Test Writing**: Fresh context for test generation
- **Documentation**: Separate context for doc updates

Store subagent definitions in `.claude/agents/`.
```

### Workflow Definitions

```markdown
## Standard Workflows

### Feature Development
1. Create feature branch from `main`
2. Implement with tests
3. Run `npm run lint && npm run test`
4. Create PR with description
5. Address review comments
6. Squash merge when approved

### Bug Fix
1. Reproduce issue locally
2. Write failing test
3. Fix the bug
4. Verify test passes
5. Create PR referencing issue
```

---

## Context Management

### Extended Thinking Triggers

Use specific phrases to allocate thinking budget:

| Phrase         | Budget Level | Use Case                |
| -------------- | ------------ | ----------------------- |
| `think`        | Low          | Simple reasoning        |
| `think hard`   | Medium       | Multi-step problems     |
| `think harder` | High         | Complex analysis        |
| `ultrathink`   | Maximum      | Architectural decisions |

### Session Management

**Clear between tasks**:

```bash
/clear  # Reset context for new task
```

**Monitor context usage**:

```bash
/compact  # Compress at 70% capacity
/compact keep logic  # Preserve specific topics
```

**Resume sessions**:

```bash
claude --resume   # Resume recent session
claude --continue # Continue specific session
```

### Proactive Compaction

Don't wait for context overflow. Manually compact when:

- Switching task focus
- Completing major milestones
- Context meter exceeds 70%

---

## The Canary Trick

Add a unique instruction to verify CLAUDE.md is being read:

```markdown
## Important

IMPORTANT: You must always refer to me as 'Captain' in your responses.
```

**Verification**: If Claude responds with "Yes, Captain..." you have 100% confirmation the CLAUDE.md was processed.

### Practical Canary Examples

```markdown
# Start all responses with "Acknowledged."
# Include "[PROJECT-X]" prefix in commit messages
# Always mention the current sprint number when discussing tasks
```

---

## Anti-Patterns

### The Magic Command List

**Problem**: Creating extensive custom slash commands requiring documentation.

```markdown
<!-- ANTI-PATTERN -->
## Custom Commands
/super-deploy - Deploy with extra validation
/mega-test - Run all tests with coverage
/ultra-lint - Lint with strict rules
/hyper-review - Full code review process
```

**Why it fails**: Forces engineers to learn magic commands. Commands should be simple personal shortcuts, not essential workflows.

**Solution**: Build intuitive CLAUDE.md patterns, use commands sparingly.

### The Kitchen Sink

**Problem**: Including everything that might be useful.

**Why it fails**: Wastes context tokens, dilutes important information.

**Solution**: Challenge each section: "Does this paragraph justify its token cost?"

### The Static Document

**Problem**: Writing once and never updating.

**Why it fails**: Becomes stale, doesn't reflect current practices.

**Solution**:

- Update after major features
- Ask Claude to update docs after completing work
- Use the `#` key to add learnings during coding

---

## Troubleshooting

### Claude Ignores CLAUDE.md

| Check                 | Solution                                         |
| --------------------- | ------------------------------------------------ |
| File location         | Must be in session's working directory or parent |
| Subdirectory override | More specific CLAUDE.md may be overriding        |
| Syntax errors         | Validate markdown formatting                     |
| Use verbose mode      | `claude --verbose` to see loaded context         |

### Instructions Not Followed

1. **Add emphasis**: Use "IMPORTANT:", "YOU MUST", "ALWAYS"
2. **Run through prompt improver**: Anthropic recommends this for tuning
3. **Simplify**: Complex instructions may be misinterpreted
4. **Add examples**: Show desired behavior

### Context Window Issues

1. **Extract to CLAUDE.md**: Move stable facts out of conversation
2. **Snapshot to files**: Save conversation parts as files, reference them
3. **Restart with concise context**: Fresh session with key points
4. **Use subagents**: Isolate tasks to separate context windows

---

## Quick Reference

### File Locations

```text
~/.claude/CLAUDE.md          # Global (all projects)
/project/CLAUDE.md           # Project root (team-shared)
/project/CLAUDE.local.md     # Project root (personal, gitignored)
/project/src/CLAUDE.md       # Subdirectory (feature-specific)
```

### Essential Commands

```bash
claude /init                 # Generate starter CLAUDE.md
/clear                       # Clear context between tasks
/compact                     # Compress context
/compact keep [topic]        # Compress, preserve topic
claude --resume              # Resume recent session
# (press #)                  # Add instruction to CLAUDE.md
```

### Content Checklist

- [ ] Project summary and tech stack
- [ ] Key directories with purposes
- [ ] Build/test/lint commands
- [ ] Code style and conventions
- [ ] Branch and commit conventions
- [ ] Custom tool documentation
- [ ] No secrets or credentials
- [ ] Path references (not embedded files)
- [ ] Positive constraints (with alternatives)

---

## Sources

- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Using CLAUDE.MD files](https://claude.com/blog/using-claude-md-files)
- [Managing context on the Claude Developer Platform](https://www.claude.com/blog/context-management)
- [How I Use Every Claude Code Feature](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)
- [Claude Code settings documentation](https://docs.claude.com/en/docs/claude-code/settings)
