# Git Commit Conventions

Conventional Commits format for writing clear, structured commit messages. These conventions help AI assistants generate consistent commit messages that support automated changelog generation and semantic versioning.

## Core Guidelines

### 1. Use Conventional Commits Format

Structure commits as `type(scope): description`. Type and description are required. Scope is optional. Keep description under 72 characters.

**Example**:
```bash
# Good
feat(auth): add password reset functionality
fix(api): handle null user in profile endpoint
docs(readme): update installation instructions
refactor(database): simplify query builder logic

# Avoid
Added password reset
fixed bug
Update README.md
```

### 2. Use Correct Commit Types

Use standard types: `feat` (feature), `fix` (bug fix), `docs` (documentation), `style` (formatting), `refactor` (code restructure), `test` (tests), `chore` (maintenance), `perf` (performance).

**Example**:
```bash
# Good
feat(users): add user profile export to CSV
fix(auth): resolve token expiration race condition
docs(api): add authentication flow diagram
test(payments): add unit tests for refund logic
refactor(core): extract validation to separate module
perf(search): optimize database query with indexes
chore(deps): upgrade FastAPI to 0.100.0

# Avoid
update(users): add export  # "update" is not standard
change(auth): fix thing    # "change" is too vague
stuff(api): various updates # "stuff" is not a type
```

### 3. Write Descriptive Subjects in Imperative Mood

Use imperative mood ("add" not "added", "fix" not "fixed"). Describe what the commit does, not what you did. No period at end.

**Example**:
```bash
# Good
feat(api): add pagination to users endpoint
fix(auth): prevent duplicate login sessions
refactor(email): extract template rendering logic

# Avoid
feat(api): added pagination to users endpoint  # Past tense
fix(auth): fixed the login bug                 # Past tense
refactor(email): I extracted the templates.    # First person + period
```

### 4. Use Scope to Indicate Component

Include optional scope in parentheses to show which part of codebase is affected. Use consistent scope names across commits.

**Example**:
```bash
# Good (clear scopes)
feat(users): add email verification
feat(posts): add draft functionality
fix(comments): resolve threading issue
test(payments): add refund flow tests

# Good (no scope when change is global)
chore: upgrade Python to 3.11
docs: update contributing guidelines

# Avoid (inconsistent scopes)
feat(user-management): add feature
feat(users-module): add other feature
feat(user): add third feature  # Pick one convention!
```

### 5. Add Body for Complex Changes

Include optional body after blank line to explain "why" and "what changed". Wrap at 72 characters. Use bullet points for multiple changes.

**Example**:
```bash
# Good
feat(api): add rate limiting to authentication endpoints

Implement token bucket algorithm to prevent brute force attacks.
Rate limit set to 5 attempts per minute per IP address.

- Add RateLimiter middleware
- Configure limits in environment variables
- Return 429 status when limit exceeded

# Simple commits don't need body
fix(auth): correct token expiration check
```

### 6. Mark Breaking Changes Explicitly

Add `BREAKING CHANGE:` in footer or `!` after type/scope for breaking changes. Describe migration path in footer.

**Example**:
```bash
# Good (with !)
feat(api)!: change user endpoint response format

BREAKING CHANGE: User endpoint now returns nested objects
instead of flat structure. Update API clients to use
response.data.user instead of response.user.

# Good (footer only)
refactor(database): rename users table to accounts

BREAKING CHANGE: Database migration required. Run
`migrate.py` before deploying this version.

# Avoid (breaking change not marked)
feat(api): change user endpoint response
```

### 7. Reference Issues in Footer

Include issue references in footer with keywords `Closes`, `Fixes`, or `Refs`. Place after breaking change notices if present.

**Example**:
```bash
# Good
fix(payments): resolve duplicate charge issue

Payment gateway was called twice when user double-clicked
submit button. Added client-side debouncing and server-side
idempotency key validation.

Fixes #123
Refs #124

# Good (multiple issues)
feat(reports): add export to PDF and Excel

Closes #234, #235

# Avoid (issue in subject line)
fix(payments): resolve duplicate charge (#123)
```

## Quick Reference

- [ ] Format: `type(scope): description`
- [ ] Type is one of: feat, fix, docs, style, refactor, test, chore, perf
- [ ] Description in imperative mood (add, fix, update)
- [ ] Subject line under 72 characters, no period
- [ ] Scope indicates component/module affected
- [ ] Body explains "why" for complex changes
- [ ] Breaking changes marked with `!` or `BREAKING CHANGE:`
- [ ] Issues referenced with Fixes/Closes in footer
