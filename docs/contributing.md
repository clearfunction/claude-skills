# Contributing to CF DevTools

Thank you for your interest in contributing! This guide will help you create and submit new skills.

## Skill Structure

Each skill is a self-contained directory under `skills/`:

```text
skills/
└── your-skill-name/
    ├── SKILL.md          # Required: Skill definition
    ├── references/       # Optional: Reference documentation
    ├── scripts/          # Optional: Helper scripts
    └── assets/           # Optional: Images, templates
```

## Creating a New Skill

### 1. Create the Skill Directory

```bash
mkdir -p skills/your-skill-name/{references,scripts,assets}
```

### 2. Write SKILL.md

Every skill requires a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: your-skill-name
description: Brief description of what this skill does and when to use it (max 1024 chars)
---

# Your Skill Name

Detailed instructions for Claude to follow when this skill is activated.

## When to Use This Skill

Describe the triggers that should activate this skill.

## Instructions

Step-by-step guidance for Claude.

## Examples

Concrete examples of inputs and outputs.

## Best Practices

Guidelines for optimal results.
```

### 3. Frontmatter Requirements

| Field         | Required | Max Length | Description                              |
| ------------- | -------- | ---------- | ---------------------------------------- |
| `name`        | Yes      | 64 chars   | Lowercase, hyphens for spaces            |
| `description` | Yes      | 1024 chars | Claude uses this to determine activation |

### 4. Content Guidelines

- **Be specific**: Vague instructions produce vague results
- **Include examples**: Show expected inputs and outputs
- **Document edge cases**: What should Claude avoid?
- **Test thoroughly**: Run the skill with various prompts

## Testing Your Skill

### Local Testing

1. Add the plugin locally:

   ```bash
   # In Claude Code
   /plugin add /path/to/cf-devtools
   ```

2. Test with prompts that should trigger your skill

3. Verify Claude activates the skill appropriately

4. Test edge cases and failure modes

### Validation Checklist

- [ ] SKILL.md has valid YAML frontmatter
- [ ] `name` is lowercase with hyphens
- [ ] `description` clearly explains when to use
- [ ] Instructions are clear and actionable
- [ ] Examples demonstrate expected behavior
- [ ] Skill activates on appropriate prompts
- [ ] Skill does NOT activate on unrelated prompts

## Submitting Changes

### For New Skills

1. Fork the repository
2. Create a feature branch: `git checkout -b skill/your-skill-name`
3. Add your skill directory and files
4. Update `README.md` to list the new skill
5. Submit a pull request

### For Skill Improvements

1. Create a branch: `git checkout -b improve/skill-name`
2. Make your changes
3. Test thoroughly
4. Submit a pull request with description of improvements

### Pull Request Guidelines

- One skill per PR (for new skills)
- Include testing notes
- Describe activation triggers
- Document any limitations

## Code Style

- Use standard Markdown formatting
- Bullet points use `-` not `*` or `+`
- Code blocks specify language
- Tables are properly aligned
- No trailing whitespace

## Questions?

Open an issue for discussion before starting work on major changes.
