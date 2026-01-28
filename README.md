# Testing Skills

[Skills](https://agentskills.io/) for AI coding agents with best testing practices based on amazing articles by [Artem Zakharchenko](https://github.com/kettanaito) on [EpicWeb](https://www.epicweb.dev/articles). Source articles available in the `articles/` folder.

## Installation

Install with `npm`:
```sh
npx skills add https://github.com/MonsterDeveloper/testing-skills --skill testing-best-practices
```

or `bun`:
```sh
bunx skills add https://github.com/MonsterDeveloper/testing-skills --skill testing-best-practices
```

## Skill Structure

```
testing-best-practices/
├── SKILL.md                            # Core principles and patterns
└── references/
    ├── vitest-patterns.md              # Vitest config, async, mocks
    └── component-testing.md            # Component/UI test patterns
```

The skill is designed for context window efficiency:
- **SKILL.md** (~200 lines) contains essential principles loaded when the skill triggers
- **Reference files** are loaded only when needed for detailed patterns

## Key Principles

1. **Test intentions, not implementations** — Tests should fail only when user-facing behavior breaks
2. **The Golden Rule of Assertions** — A test must fail if, and only if, the intention behind the system is not met
3. **Don't assert on requests** — Validate in mocks, assert on UI state
4. **Use real browsers** — JSDOM creates false confidence; prefer Vitest Browser Mode
5. **Flat, explicit tests** — Avoid nested `describe` and scattered hooks
6. **Code coverage is a tool, not a target** — Never let it drive testing decisions

## Topics Covered

- Test structure (Setup → Action → Assertion)
- Test boundaries and mocking principles
- Assertion patterns (`.resolves`, `.toBeVisible`, inverse assertions)
- Mock management (`.mockClear()` vs `.mockReset()` vs `.mockRestore()`)
- S.M.A.R.T. framework for flaky tests
- Vitest Browser Mode setup and patterns
- Component testing with MSW
- Disposable objects for test cleanup

## Source Articles

Based on 18 articles from EpicWeb covering:
- The True Purpose of Testing
- The Golden Rule of Assertions
- Anatomy of a Test
- Test Boundaries
- Implicit Assertions
- Inverse Assertions
- Vitest Defaults and Browser Mode
- Why Not JSDOM
- And more...
