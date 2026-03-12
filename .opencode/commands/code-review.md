---
description: Comprehensive security and quality review of uncommitted changes. Blocks on CRITICAL or HIGH issues.
agent: code-reviewer
---

# Code Review

Comprehensive security and quality review of uncommitted changes:

1. Get changed files: git diff --name-only HEAD

2. For each changed file, check for:

**Security Issues (CRITICAL):**
- Hardcoded credentials, API keys, tokens
- SQL injection vulnerabilities
- XSS vulnerabilities  
- Missing input validation
- Insecure dependencies
- Path traversal risks

**Code Quality (HIGH):**
- Functions > 50 lines
- Files > 800 lines
- Nesting depth > 4 levels
- Missing error handling
- console.log statements
- TODO/FIXME comments
- Missing JSDoc for public APIs

**Best Practices (MEDIUM):**
- Mutation patterns (use immutable instead)
- Emoji usage in code/comments
- Accessibility issues (a11y)
- Missing SPEC.md for new features (Spec-First principle)

**AI Slop (MEDIUM):**
- Placeholder comments: `// TODO: implement`, `// Add code here`, `// ... rest of`
- Hallucinated imports (modules not in project deps)
- Empty function bodies or unreachable code
- Overly verbose/obvious comments that just repeat the code

3. Generate report with:
   - Severity: CRITICAL, HIGH, MEDIUM, LOW
   - File location and line numbers
   - Issue description
   - Suggested fix

4. Block commit if CRITICAL or HIGH issues found

Never approve code with security vulnerabilities!
