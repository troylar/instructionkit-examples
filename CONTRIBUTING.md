# Contributing to InstructionKit Official Examples

Thank you for your interest in improving these example instructions! This repository provides curated, tested instructions for AI coding assistants.

## How to Contribute

### Reporting Issues

If you notice an instruction that doesn't work well with your AI coding tool:

1. Open a GitHub issue with:
   - Which instruction file (e.g., `python-best-practices.md`)
   - Which AI tool you tested (Cursor, Claude Code, Windsurf, GitHub Copilot, etc.)
   - Specific examples where the AI didn't follow guidelines
   - Expected vs actual behavior

### Improving Existing Instructions

1. **Fork** this repository
2. **Create a branch** for your changes (e.g., `improve-python-async`)
3. **Make your changes** to the instruction file(s)
4. **Test thoroughly** with at least 2 AI coding tools (see TESTING.md)
5. **Submit a PR** with:
   - Clear description of what you improved and why
   - Test results showing improved AI adherence
   - Reference to any related issues

### Proposing New Instructions

Before creating a new instruction:

1. **Check existing instructions** to avoid duplication
2. **Open an issue** proposing the new instruction with:
   - Topic/technology it will cover
   - Why it's valuable for AI-assisted coding
   - Which category it belongs to
3. **Wait for maintainer feedback** before starting work

Once approved:

1. Use the template in `.templates/instruction-template.md`
2. Follow the quality checklist in `.templates/quality-checklist.md`
3. Test with all 4 primary AI tools (Cursor, Claude Code, Windsurf, Copilot)
4. Document test results in TESTING.md
5. Update `instructionkit.yaml` with the new instruction entry

## Quality Standards

All instructions must:

- Be **400-600 words** (acceptable range: 300-800)
- Include **2-5 code examples** with "good" and "avoid" patterns
- Have **3-7 core guidelines** (cognitive limit)
- Include a **Quick Reference** checklist
- Be **tool-agnostic** (no references to specific AI assistants)
- Achieve **80%+ guideline adherence** when tested with AI tools

## Testing Instructions

See TESTING.md for the validation process. Every instruction must be tested with:

1. **Cursor** (VS Code extension)
2. **Claude Code** (CLI)
3. **Windsurf** (IDE)
4. **GitHub Copilot** (VS Code/JetBrains)

Create 5-10 test prompts that exercise different guidelines in the instruction. Record which guidelines the AI followed vs. didn't follow.

**80% threshold**: If an AI tool follows at least 8 out of 10 tested guidelines, it passes.

## Pull Request Process

1. Ensure all tests pass (documented in TESTING.md)
2. Update TESTING.md with your validation results
3. If adding new instruction, update `instructionkit.yaml`
4. Maintainers will review within 1 week
5. Address any feedback
6. Once approved, changes will be merged and included in next release

## Code of Conduct

- Be respectful and constructive in all interactions
- Focus on improving instruction quality, not debating AI tools
- Provide specific examples and test results
- Help others learn and improve

## Questions?

Open an issue with the `question` label or reach out to the maintainer.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
