---
name: validate-completion
description: Verify that claimed task completion actually achieves the underlying goal
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

Verify that claimed task completion actually works end-to-end against the user's request. Run real tests, builds, or checks where available — do not rely on code inspection alone.

**Validation Checklist:**
1. **Core Functionality**: Implemented end-to-end, not stubbed, mocked, or commented out.
2. **Error Handling**: Critical error paths handled, not silently swallowed.
3. **Integration Points**: DB calls, API calls, and external wiring actually match real contracts.
4. **Tests & Verification**: Run the test suite or verification commands and confirm they test real behavior, not trivial mocks.
5. **Missing Pieces**: Config, migrations, and dependencies present.
6. **Shortcuts**: No hardcoded secrets/values, skipped validation, or bypassed checks.

Look for red flags:
- TODO, FIXME, or "Not implemented yet" comments
- Empty catch blocks or swallowed errors
- Mock objects replacing real integrations in non-test code
- Tests that pass without exercising the changed behavior

**Output Format:**
```
## Task Completion Validation

**VALIDATION STATUS**: APPROVED | REJECTED

**CRITICAL ISSUES**: [Only concrete, verified blockers with file:line]
**MISSING COMPONENTS**: [Only missing pieces required for completion]
**QUALITY CONCERNS**: [Only real shortcuts or bugs]
**RECOMMENDATION**: [Clear next steps]
```

Omit empty sections (or write `None`). Never invent issues just to fill a section.
