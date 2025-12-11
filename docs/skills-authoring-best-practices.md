# Skills Authoring Best Practices

A comprehensive guide to creating effective Claude Code Skills based on official Anthropic documentation and real-world learnings.

## Table of Contents

- [Overview](#overview)
- [Size Limits and Constraints](#size-limits-and-constraints)
- [Progressive Disclosure](#progressive-disclosure)
- [Writing Effective Descriptions](#writing-effective-descriptions)
- [Naming Conventions](#naming-conventions)
- [SKILL.md Structure](#skillmd-structure)
- [Reference File Organization](#reference-file-organization)
- [Conciseness Guidelines](#conciseness-guidelines)
- [Workflows and Feedback Loops](#workflows-and-feedback-loops)
- [Testing Requirements](#testing-requirements)
- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
- [Troubleshooting](#troubleshooting)
- [Checklist](#checklist)

---

## Overview

Agent Skills are organized directories containing instructions, scripts, and resources that Claude can discover and load dynamically. The core design principle is **progressive disclosure** - showing just enough information for Claude to decide what to do next, then revealing details as needed.

### How Skills Are Loaded

1. **Startup**: Only `name` and `description` from all Skills are pre-loaded (~100 tokens per skill)
2. **Activation**: When relevant, Claude reads the full `SKILL.md`
3. **Discovery**: Additional referenced files are loaded only as needed

This means **the amount of context bundled into a skill is effectively unbounded** because agents needn't load everything into their context window simultaneously.

---

## Size Limits and Constraints

**IMPORTANT**: These are the official limits. Previous documentation errors (like "200 characters" or "500 characters") were incorrect.

### Official Limits (Anthropic Specification)

| Field             | Limit               | Notes                            |
| ----------------- | ------------------- | -------------------------------- |
| `name`            | **64 characters**   | Lowercase, numbers, hyphens only |
| `description`     | **1024 characters** | Critical for discovery           |
| **SKILL.md body** | **500 lines**       | NOT characters - split if longer |

### Name Field Rules

- Maximum 64 characters
- Must contain only lowercase letters, numbers, and hyphens
- Cannot contain XML tags
- Cannot contain reserved words: "anthropic", "claude"

### Description Field Rules

- Must be non-empty
- Maximum 1024 characters
- Cannot contain XML tags
- Should describe what the Skill does AND when to use it

---

## Progressive Disclosure

### The Core Principle

**SKILL.md is a table of contents, not the encyclopedia.**

Skills use progressive disclosure - like a well-organized manual that starts with a table of contents, then specific chapters, and finally a detailed appendix.

### Three Levels of Information

| Level                    | What                          | When Loaded              |
| ------------------------ | ----------------------------- | ------------------------ |
| **1. Metadata**          | `name` and `description`      | At startup (all skills)  |
| **2. Main Instructions** | SKILL.md body                 | When skill is triggered  |
| **3. Reference Files**   | Additional .md files, scripts | When specifically needed |

### Visual Structure

```text
skill-name/
├── SKILL.md              # Level 2: Overview + navigation (keep under 500 lines)
├── references/
│   ├── detailed-guide.md # Level 3: Comprehensive documentation
│   ├── api-reference.md  # Level 3: API details
│   └── examples.md       # Level 3: Input/output examples
└── scripts/
    └── validate.py       # Level 3: Executed, not loaded into context
```

---

## Writing Effective Descriptions

The `description` field is the **single most critical component** for skill discovery. Claude uses it to choose the right skill from potentially 100+ available skills.

### The Golden Rule: Third Person

**ALWAYS write in third person.** The description is injected into the system prompt, and inconsistent point-of-view causes discovery problems.

| Status   | Example                                       |
| -------- | --------------------------------------------- |
| **GOOD** | "Processes Excel files and generates reports" |
| **BAD**  | "I can help you process Excel files"          |
| **BAD**  | "Use when you need to process Excel files"    |
| **BAD**  | "You can use this to process Excel files"     |

### Include Triggers and Capabilities

Each description should include:

1. **What the skill does** (capabilities)
2. **When to use it** (triggers)
3. **Specific terms** that might appear in user requests

### Effective Description Examples

**PDF Processing**:

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel Analysis**:

```yaml
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when working with Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git Commit Helper**:

```yaml
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

**1Password + direnv** (from this repo):

```yaml
description: Configures 1Password CLI with direnv for fast secret loading using op-run pattern. Activates for: 1Password + direnv setup, slow secrets (>2 sec), environment variables from 1Password, .env.op files, op:// references, or migrating from multiple op-read calls to single op-run.
```

### Descriptions to Avoid

```yaml
# TOO VAGUE
description: Helps with documents

# TOO GENERIC
description: Processes data

# NO TRIGGERS
description: Does stuff with files
```

---

## Naming Conventions

### Use Gerund Form (verb + -ing)

This clearly describes the activity or capability the skill provides.

| Good (Gerund)            | Also Acceptable        | Avoid       |
| ------------------------ | ---------------------- | ----------- |
| `processing-pdfs`        | `pdf-processing`       | `helper`    |
| `analyzing-spreadsheets` | `spreadsheet-analysis` | `utils`     |
| `managing-databases`     | `database-management`  | `tools`     |
| `testing-code`           | `code-testing`         | `documents` |
| `writing-documentation`  | `doc-writer`           | `data`      |

### What to Avoid

- **Vague names**: `helper`, `utils`, `tools`
- **Overly generic**: `documents`, `data`, `files`
- **Reserved words**: `anthropic-helper`, `claude-tools`
- **Inconsistent patterns**: Mix of styles in your collection

---

## SKILL.md Structure

### Required YAML Frontmatter

```yaml
---
name: skill-name-here
description: Third-person description of what the skill does. Activates for: trigger1, trigger2, trigger3. Not for: anti-trigger1.
---
```

### Recommended Body Structure

````markdown
---
name: processing-pdfs
description: Extract text and tables from PDF files, fill forms, merge documents. Activates for: PDF files, forms, document extraction, merging PDFs.
---

# PDF Processing

Brief overview (1-2 sentences).

## Quick Start

Minimal example to get started immediately.

```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
````

## When to Use

- Extracting text from PDF documents
- Filling PDF forms
- Merging multiple PDFs
- Analyzing PDF structure

## Detailed Documentation

- **Form Filling**: See [references/forms.md](references/forms.md)
- **API Reference**: See [references/api.md](references/api.md)
- **Examples**: See [references/examples.md](references/examples.md)

## Common Patterns

[Key patterns and workflows]

## Troubleshooting

[Common issues and solutions]

````text

---

## Reference File Organization

### Keep References One Level Deep

Claude may partially read files when they're referenced from other referenced files. **All reference files should link directly from SKILL.md.**

**BAD (Too Deep)**:
```markdown
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
````

**GOOD (One Level)**:

```markdown
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### Add Table of Contents to Long Files

For reference files over 100 lines, include a TOC at the top:

```markdown
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...
```

### Domain-Specific Organization

For skills with multiple domains, organize by domain:

```text
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

When a user asks about sales metrics, Claude only loads `sales.md`, not the others.

---

## Conciseness Guidelines

### The Default Assumption

**Claude is already very smart.** Only add context Claude doesn't already have.

Challenge each piece of information:

- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"
- "Does this paragraph justify its token cost?"

### Good vs Bad Examples

**GOOD (Concise, ~50 tokens)**:

````markdown
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**BAD (Verbose, ~150 tokens)**:

```markdown
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

The concise version assumes Claude knows what PDFs are and how libraries work.

---

## Workflows and Feedback Loops

### Use Checklists for Complex Tasks

Provide a checklist that Claude can track:

````markdown
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**
Review each document in the `sources/` directory...
````

### Implement Feedback Loops

**Common pattern**: Run validator -> fix errors -> repeat

```markdown
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python scripts/pack.py unpacked_dir/ output.docx`
```

### Conditional Workflows

```markdown
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** -> Follow "Creation workflow" below
   **Editing existing content?** -> Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

---

## Testing Requirements

### Test with All Target Models

Skills effectiveness depends on the underlying model:

| Model                                | Consideration                           |
| ------------------------------------ | --------------------------------------- |
| **Claude Haiku** (fast, economical)  | Does the Skill provide enough guidance? |
| **Claude Sonnet** (balanced)         | Is the Skill clear and efficient?       |
| **Claude Opus** (powerful reasoning) | Does the Skill avoid over-explaining?   |

What works perfectly for Opus might need more detail for Haiku.

### Test Both Activation and Non-Activation

1. **Activation tests**: Prompts that SHOULD trigger the skill
2. **Non-activation tests**: Prompts that should NOT trigger the skill
3. **Edge cases**: Ambiguous prompts near the skill's boundaries

### Evaluation-Driven Development

**Create evaluations BEFORE writing extensive documentation.**

1. **Identify gaps**: Run Claude on tasks without a Skill. Document failures
2. **Create evaluations**: Build 3+ scenarios that test these gaps
3. **Establish baseline**: Measure performance without the Skill
4. **Write minimal instructions**: Just enough to pass evaluations
5. **Iterate**: Execute evaluations, compare against baseline, refine

---

## Anti-Patterns to Avoid

### Deeply Nested References

**Problem**: Claude may use `head -100` to preview nested files, missing content.

**Solution**: Keep all references one level deep from SKILL.md.

### Time-Sensitive Information

**BAD**:

```markdown
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**GOOD**:

```markdown
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>
The v1 API used: `api.example.com/v1/messages`
</details>
```

### Multiple Library Options

**BAD**:

```markdown
You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or...
```

**GOOD**:

```markdown
Use pdfplumber for text extraction.

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead.
```

### Windows-Style Paths

**BAD**: `scripts\helper.py`
**GOOD**: `scripts/helper.py`

Unix-style paths work across all platforms.

### Vague Descriptions

**BAD**: "Helps with documents"
**GOOD**: "Extracts text from PDFs, fills forms, merges documents. Activates for: PDF files, document extraction, form filling."

### Second-Person Descriptions

**BAD**: "Use when you need to..."
**GOOD**: "Processes files when user requests..."

---

## Troubleshooting

### Skill Not Activating

| Issue                  | Solution                            |
| ---------------------- | ----------------------------------- |
| Description too vague  | Add specific trigger terms          |
| Description too narrow | Broaden with more use cases         |
| Wrong perspective      | Use third person, not "Use when..." |
| Missing triggers       | Add "Activates for: X, Y, Z"        |

### Skill Activating When It Shouldn't

| Issue                 | Solution                          |
| --------------------- | --------------------------------- |
| Description too broad | Add "Not for: X, Y"               |
| Overlapping triggers  | Use distinct terms between skills |
| Generic terms         | Be more specific about the domain |

### Claude Missing Information

| Issue                | Solution                        |
| -------------------- | ------------------------------- |
| Nested references    | Keep references one level deep  |
| No TOC in long files | Add table of contents           |
| Critical info buried | Move to top of file or SKILL.md |

### Inconsistent Output Quality

| Issue               | Solution                          |
| ------------------- | --------------------------------- |
| No examples         | Add input/output examples         |
| No validation steps | Add feedback loops                |
| Vague instructions  | Provide specific steps or scripts |

---

## Checklist

### Before Publishing

**Core Quality**:

- [ ] `name` is lowercase with hyphens (max 64 chars)
- [ ] `description` is third person (max 1024 chars)
- [ ] `description` includes what it does AND when to use it
- [ ] SKILL.md body is under 500 lines
- [ ] Additional details are in separate reference files
- [ ] References are one level deep (not nested)
- [ ] Long reference files (100+ lines) have TOC
- [ ] No time-sensitive information
- [ ] Consistent terminology throughout
- [ ] Examples are concrete input/output pairs

**Discovery**:

- [ ] Description includes specific trigger terms
- [ ] Description mentions file types/extensions if relevant
- [ ] "Activates for:" section lists triggers
- [ ] "Not for:" section clarifies boundaries (if needed)

**Testing**:

- [ ] Tested activation with expected prompts
- [ ] Tested non-activation with unrelated prompts
- [ ] Tested with Haiku, Sonnet, and Opus
- [ ] Team feedback incorporated (if applicable)

**For Skills with Scripts**:

- [ ] Scripts handle errors explicitly
- [ ] No "voodoo constants" (all values documented)
- [ ] Required packages listed
- [ ] Scripts are executable (`chmod +x`)
- [ ] Unix-style paths throughout

---

## Sources

- [Skill authoring best practices - Claude Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Agent Skills Overview - Claude Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Introducing Agent Skills](https://www.anthropic.com/news/skills)
- [GitHub - anthropics/skills](https://github.com/anthropics/skills)
