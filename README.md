# InstructionKit Official Examples

Curated instruction examples for common development scenarios across Python, JavaScript, testing, API design, security, documentation, and git workflows.

## Quick Start

### Install InstructionKit

```bash
pip install instructionkit
```

### Download These Examples

```bash
inskit download --from https://github.com/troylar/instructionkit-examples
```

### Browse and Install

```bash
inskit install
```

Use the interactive TUI to browse examples, search by tag (e.g., `/python`, `/react`, `/security`), and install to your project.

## What's Included

This repository contains **12 high-quality instruction examples** across essential categories:

### Python Development (2 instructions)
- **python-best-practices**: Type hints, naming conventions, docstrings, error handling, PEP 8
- **python-async-patterns**: Async/await, asyncio.gather, error handling in async contexts

### JavaScript/TypeScript (3 instructions)
- **javascript-modern-patterns**: ES6+, async/await, destructuring, arrow functions, modules
- **react-component-guide**: Functional components, hooks (useState, useEffect, custom), composition
- **typescript-best-practices**: Type definitions, generics, utility types, strict mode

### Testing (1 instruction)
- **pytest-testing-guide**: Fixtures, parametrization, mocking, test organization

### API Design (1 instruction)
- **api-design-principles**: RESTful resource naming, HTTP methods, status codes, pagination

### Security (2 instructions)
- **security-guidelines**: Input validation, parameterized queries, secrets management, XSS prevention
- **security-owasp-checklist**: OWASP Top 10 mapped to code patterns with specific examples

### Documentation (1 instruction)
- **documentation-standards**: Docstrings, comments, README structure, API documentation

### Git (1 instruction)
- **git-commit-conventions**: Conventional Commits format with types, scopes, and descriptions

### DevOps (1 instruction)
- **docker-best-practices**: Dockerfile optimization, multi-stage builds, security hardening

---

Each instruction is:
- ✅ Tested with real AI coding tools (Cursor, Claude Code, Windsurf, GitHub Copilot)
- ✅ Validated for 80%+ guideline adherence
- ✅ 400-600 words with concrete code examples
- ✅ Tool-agnostic (works with any AI coding assistant)

## How to Use

### 1. Download to Your Library

```bash
inskit download --from https://github.com/troylar/instructionkit-examples
```

This clones the repository to your local InstructionKit library (`~/.instructionkit/library/`).

### 2. Browse Available Instructions

```bash
inskit install
```

Opens an interactive TUI where you can:
- Browse all downloaded instructions
- Search by tag: `/python`, `/javascript`, `/security`, `/testing`
- Preview instruction content
- Select instructions to install

### 3. Install to Your Project

Select instructions in the TUI and press Enter to install. Instructions are copied to:
- `.cursor/rules/` (for Cursor)
- `.claude/rules/` (for Claude Code)
- `.windsurf/rules/` (for Windsurf)
- `.github/copilot-instructions.md` (for GitHub Copilot)

### 4. Start Coding

Your AI assistant now follows the installed guidelines automatically. Try prompts like:
- "Create a function to fetch user data from API" → AI uses type hints, async/await, error handling
- "Write tests for this function" → AI uses pytest fixtures and parametrization
- "Add a REST endpoint for users" → AI follows proper resource naming and status codes

## For Teams

Commit installed instructions to Git:

```bash
git add .cursor/rules/ .claude/rules/
git commit -m "chore: add AI coding guidelines"
git push
```

Team members who pull the changes automatically get the same AI coding standards.

## Search by Tag

Use the TUI search feature to find relevant instructions:

| Search | Shows |
|--------|-------|
| `/python` | python-best-practices, python-async-patterns, pytest-testing-guide |
| `/javascript` | javascript-modern-patterns, react-component-guide, typescript-best-practices |
| `/react` | react-component-guide |
| `/typescript` | typescript-best-practices |
| `/testing` | pytest-testing-guide |
| `/security` | security-guidelines, security-owasp-checklist |
| `/owasp` | security-owasp-checklist |
| `/api-design` | api-design-principles |
| `/documentation` | documentation-standards |
| `/git` | git-commit-conventions |
| `/docker` | docker-best-practices |
| `/devops` | docker-best-practices |
| `/backend` | python-best-practices, python-async-patterns, api-design-principles, typescript-best-practices |
| `/frontend` | javascript-modern-patterns, react-component-guide, typescript-best-practices |

## Contributing

We welcome improvements! See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to report issues with instructions
- How to improve existing instructions
- How to propose new instructions
- Testing requirements (80% AI tool adherence)

## Quality Standards

Every instruction in this repository meets these criteria:

- **Length**: 400-600 words (acceptable range: 300-800)
- **Structure**: 3-7 core guidelines with code examples
- **Examples**: 2-5 code snippets showing good/avoid patterns
- **Testing**: Validated with 4 AI tools (Cursor, Claude Code, Windsurf, Copilot)
- **Adherence**: 80%+ of guidelines followed by AI tools
- **Tool-agnostic**: No references to specific AI assistants

See [TESTING.md](TESTING.md) for validation results.

## Creating Your Own Instructions

Want to create similar instructions for your team?

1. Use our template: `.templates/instruction-template.md`
2. Follow the quality checklist: `.templates/quality-checklist.md`
3. Test with multiple AI tools
4. Share with your team or contribute back to this repository

## License

MIT License - see [LICENSE](LICENSE) for details.

## Author

Troy Larson / InstructionKit

## Links

- [InstructionKit CLI](https://github.com/troylar/instructionkit)
- [Report Issues](https://github.com/troylar/instructionkit-examples/issues)
- [Contribute](CONTRIBUTING.md)
