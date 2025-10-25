# Instruction Quality Checklist

Use this checklist before considering an instruction "done" and ready for submission.

## Content Structure

- [ ] **Length**: Between 300-800 words (target: 400-600)
- [ ] **Core guidelines**: Contains 3-7 main guidelines (not more, not less)
- [ ] **Code examples**: Includes 2-5 code examples
- [ ] **Good/Avoid pairs**: Each code example shows both correct and incorrect patterns
- [ ] **Quick Reference**: Includes actionable checklist with 3-7 items
- [ ] **Section headers**: Uses clear, descriptive headings

## Code Quality

- [ ] **Language tags**: All code blocks have language specified (```python, ```javascript)
- [ ] **Syntax correctness**: All code examples are syntactically valid
- [ ] **Realistic examples**: Code examples reflect real-world scenarios
- [ ] **Consistency**: Examples follow the same style/patterns throughout
- [ ] **Comments**: Code includes helpful comments where needed

## Clarity & Specificity

- [ ] **Clear purpose**: Opening paragraph clearly states what instruction covers
- [ ] **Specific rules**: Guidelines are concrete and measurable
- [ ] **Imperative tone**: Uses directive language ("Use", "Avoid", "Always")
- [ ] **No vague advice**: Avoids generic statements like "write clean code"
- [ ] **Actionable**: Every guideline can be directly applied by AI

## Tool Agnostic

- [ ] **No tool references**: Doesn't mention Cursor, Copilot, Claude, or other specific AI tools
- [ ] **No IDE references**: Doesn't reference VS Code, JetBrains, etc.
- [ ] **Generic prompts**: Examples work with any AI coding assistant
- [ ] **Platform neutral**: Works across different development environments

## Validation & Testing

- [ ] **Test prompts created**: Prepared 5-10 test prompts that exercise guidelines
- [ ] **Cursor tested**: Validated with Cursor (80%+ adherence)
- [ ] **Claude Code tested**: Validated with Claude Code (80%+ adherence)
- [ ] **Windsurf tested**: Validated with Windsurf (80%+ adherence)
- [ ] **GitHub Copilot tested**: Validated with GitHub Copilot (80%+ adherence)
- [ ] **Results documented**: Test results recorded in TESTING.md

## Metadata & Integration

- [ ] **instructionkit.yaml updated**: Entry added with correct name, description, file path, tags
- [ ] **Tags appropriate**: Uses 2-4 tags mixing categories for discoverability
- [ ] **File naming**: Filename matches name field in instructionkit.yaml (kebab-case)
- [ ] **File location**: File placed in `instructions/` directory
- [ ] **Unique name**: Name doesn't conflict with existing instructions

## Polish

- [ ] **Grammar check**: No spelling or grammar errors
- [ ] **Formatting consistent**: Uses consistent markdown formatting
- [ ] **Links work**: Any external links are valid and relevant
- [ ] **No placeholder text**: All [bracketed] template text replaced
- [ ] **Professional tone**: Language is professional and inclusive

## Final Review

- [ ] **Re-read entire instruction**: Fresh eyes review for flow and clarity
- [ ] **Colleague review**: Had someone else review (if available)
- [ ] **Cross-reference**: Checked against similar instructions for consistency
- [ ] **Edge cases**: Considered and tested edge cases for the guidelines

---

## Scoring

**Required for approval**: All items in "Content Structure", "Tool Agnostic", and "Validation & Testing" must be checked.

**Recommended**: All other sections should have 90%+ items checked.

**If any required items are unchecked**: Instruction needs more work before submission.

---

## How to Use This Checklist

1. **During creation**: Reference this checklist while writing your instruction
2. **Before testing**: Complete first pass through checklist before AI tool testing
3. **Before PR**: Final pass through entire checklist before submitting pull request
4. **Document in PR**: Mention checklist completion in PR description

---

## Notes

This checklist ensures all instructions meet the quality standard expected for the official InstructionKit example repository. These instructions represent best practices and will be used by developers worldwide - quality matters!
