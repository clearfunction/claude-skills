# Clear Function Claude Skills

Production-ready Claude Code skills for presentations, documentation, and developer workflows.

## Installation

### Claude Code (CLI)

```bash
# Add from marketplace
/plugin marketplace add clearfunction/clearfunction-claude-skills

# Install the plugin
/plugin install clearfunction-skills@clearfunction-claude-skills
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/clearfunction/clearfunction-claude-skills.git

# Add as local plugin
/plugin add /path/to/clearfunction-claude-skills
```

## Available Skills

### slidev-presentations

Generate browser-based presentations using [Slidev](https://sli.dev/) - the presentation framework for developers.

**Features:**

- Markdown-based slide generation
- Vue component support
- Code syntax highlighting with live execution
- Presenter mode with notes
- Export to PDF/PNG

**Usage:** Claude automatically activates this skill when you request presentation creation.

```text
Create a technical presentation about Kubernetes architecture for a DevOps team
```

## Project Structure

```text
clearfunction-claude-skills/
├── .claude-plugin/
│   ├── plugin.json        # Plugin manifest
│   └── marketplace.json   # Marketplace metadata
├── skills/
│   └── slidev-presentations/
│       ├── SKILL.md       # Skill definition
│       └── references/    # Slidev documentation
├── agents/                # Future: Custom agents
├── commands/              # Future: Slash commands
└── docs/                  # Documentation
```

## Skill Development

Each skill follows the [Anthropic Skills specification](https://github.com/anthropics/skills):

1. Create a directory under `skills/`
2. Add a `SKILL.md` with YAML frontmatter
3. Include any reference materials in `references/`
4. Test thoroughly before committing

See [docs/contributing.md](docs/contributing.md) for detailed guidelines.

## Roadmap

- [x] slidev-presentations - Browser-based presentations
- [ ] mermaid-diagrams - Architecture and flow diagrams
- [ ] adr-generator - Architecture Decision Records
- [ ] changelog-writer - Conventional changelog generation
- [ ] api-docs - OpenAPI documentation generation

## License

Apache 2.0 - See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please read [docs/contributing.md](docs/contributing.md) first.

---

Built by [Clear Function](https://clearfunction.com)
