---
name: interview
description: Explore a codebase and interview the user to deeply understand scope, intent, and constraints — then execute whatever followup action was requested
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

Understand the user's goal through codebase exploration and targeted questioning, then act on the instruction given.

## Process

**1. Explore Codebase**
Understand existing patterns, tech stack, similar features, and integration points relevant to the topic.

**2. Interview User**
Use your interactive question tool (`AskUserQuestion` / `ask_question`) to probe these topics:
- Core functionality: problem, edge cases, composition
- Technical: data flow, state, performance, errors, testing
- UX: user journey, error/loading states, accessibility
- Trade-offs: performance vs simplicity, new vs existing patterns
- Integration: dependencies, shared state, reusability

"Probe" means cover these topics — it does not mean ask vague questions. Each question stays specific, well-contextualized, and chunked (2-4 at a time). Follow up on vague answers. Present options with trade-offs. Stop interviewing when you have enough clarity to execute the requested action.

**3. Execute the Requested Action**
The user's invocation tells you what to do after the interview. Parse the argument and act accordingly:

- `/interview and write a spec` → write `specs/[feature-name]-spec.md` (see spec format below)
- `/interview to identify scope` → summarize scope, boundaries, and open questions in a clear response
- `/interview and make the changes` → implement directly based on gathered context
- `/interview` (no arg) → present a synthesis of what you learned and ask what to do next

## Spec Format (when writing a spec)

Omit any section below that does not apply to the feature — never write `N/A` placeholders.

```markdown
# [Feature Name] Specification

## Overview
[What and why in 2-3 sentences]

## Requirements
- [Testable functional requirements]
- Performance: [targets]
- Accessibility: [requirements]

## Architecture
### Data Model
```
[Types, schemas]
```

### Components & State
- [Component hierarchy and responsibilities]
- [State shape, location, update triggers]

### API (if needed)
```
[Endpoints, request/response shapes]
```

## Implementation Phases
1. [Foundation tasks with file:line references]
2. [Core logic in testable increments]
3. [Integration and verification]

## Testing
- Unit: [isolation tests, edge cases]
- Integration: [interactions, error paths]
- E2E: [user journeys, critical paths]

## Decisions
**[Choice]**: Chose B over A/C because [rationale]. Trade-off: [gains/losses]

## Edge Cases
**[Scenario]**: Detect via [method]. Handle by [action]. User sees [experience].

## Success Metrics
[Measurable acceptance criteria]
```

**Summary**: Interview first, act second. The argument drives the outcome.
