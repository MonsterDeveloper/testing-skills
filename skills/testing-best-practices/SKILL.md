---
name: testing-best-practices
description: >-
  Write high-quality automated tests for JavaScript/TypeScript applications.
  Use when: (1) writing unit, integration, or component tests, (2) testing
  React/Vue components, (3) working with Vitest or similar test runners,
  (4) mocking APIs or dependencies, (5) debugging flaky tests, (6) improving
  test reliability. Provides patterns for assertions, async testing, mocking,
  and test structure.
---

# Testing Best Practices

## Core Principle: Intention Over Implementation

Tests validate **intentions**, not implementations. The intention is what your code does for users; the implementation is how it achieves that.

```js
// BAD: Testing implementation details
expect(cookieUtils.parse).toHaveBeenCalledWith('sessionId=abc-123', { pick: ['name'] })

// GOOD: Testing the intention
expect(parseCookieName('sessionId=abc-123')).toBe('sessionId')
```

## The Golden Rule of Assertions

> A test must fail if, and only if, the intention behind the system is not met.

Ask: "When will this test fail?" If it can fail for reasons unrelated to the intention, fix the test boundaries.

## Test Structure (Setup → Action → Assertion)

```js
test('adds product to cart', async () => {
  // SETUP: Prepare environment and mocks
  server.use(http.post('/cart', () => new Response(null, { status: 201 })))

  // ACTION: Perform user actions
  await user.click(page.getByRole('button', { name: 'Add to cart' }))

  // ASSERTION: Verify user-facing outcome
  await expect.element(page.getByRole('alert')).toHaveTextContent('Added to cart!')
})
```

## Flat Tests Over Nested Hooks

Avoid complexity leaking into test setup through nested `describe` and hooks.

```js
// BAD: Nested setup spread across hooks
describe('auth', () => {
  beforeAll(() => { /* server setup */ })
  describe('Google sign-in', () => {
    beforeEach(() => { /* more setup */ })
    test('displays suggestion', () => { /* finally the test */ })
  })
})

// GOOD: Explicit, self-contained test
test('displays auth suggestion for returning Google user', async () => {
  await using server = createTestServer()
  server.get('/user', userHandler)
  server.post('/auth', authHandler)
  // Test is explicit about its setup
})
```

Use `using` (disposable objects) for automatic cleanup.

## Test Boundaries

Mock external factors that don't determine your code's validity:
- HTTP requests (use MSW or similar)
- Non-deterministic values (dates, RNG)
- File system, databases

Don't mock: Internal logic you're testing.

```js
// The network is a boundary - mock it
server.use(http.get('/user/:id', () => HttpResponse.json({ name: 'John' })))

// Then test user-facing behavior
await expect(fetchUser('abc-123')).resolves.toEqual({ name: 'John' })
```

## Assertions

### Use `.resolves`/`.rejects` for Async

```js
// BAD: Unhandled rejections give poor error messages
expect(await fetchUser('abc-123')).toEqual({ id: 'abc-123' })

// GOOD: Clear assertion on promise state
await expect(fetchUser('abc-123')).resolves.toEqual({ id: 'abc-123' })
await expect(fetchUser('bad-id')).rejects.toThrow('User not found')
```

### Use `.toBeVisible()` for Presence, `.not.toBeInTheDocument()` for Absence

```js
// Check element is visible and accessible
await expect.element(page.getByText('Welcome')).toBeVisible()

// Check element is absent
await expect.element(page.getByRole('alert')).not.toBeInTheDocument()
```

### Inverse Assertions for "Doesn't Happen"

```js
// Assert that notification does NOT appear
const notificationPromise = waitFor(() => {
  expect(notification).toBeVisible()
})
await expect(notificationPromise).rejects.toThrow()
```

### Skip Implicit Assertions

```js
// BAD: Redundant - length is implied by equality
expect(arr).toHaveLength(3)
expect(arr).toEqual(['a', 'b', 'c'])

// GOOD: Single meaningful assertion
expect(arr).toEqual(['a', 'b', 'c'])
```

## Don't Assert on Requests

Move request validation into mocks, not assertions:

```js
// BAD: Implementation details in assertions
expect(fetch).toHaveBeenCalledWith('/cart', { body: { product: 'mug' } })

// GOOD: Validation in mock affects UI outcome
server.use(
  http.post('/cart', async ({ request }) => {
    const data = await request.json()
    if (data.productName !== 'Porcelain mug') {
      return new Response(null, { status: 400 })
    }
    return new Response(null, { status: 201 })
  })
)
// Then assert on UI state that depends on the response
```

## Mock Management

- `.mockClear()`: Clears call history only
- `.mockReset()`: Clears calls AND removes mock implementation
- `.mockRestore()`: Completely removes the mock

```js
// Auto-reset between tests
// vitest.config.ts
export default defineConfig({ test: { mockReset: true } })
```

## Flaky Tests: S.M.A.R.T. Framework

1. **Skip** immediately - one flaky test undermines entire suite
2. **Mitigate** - gather context, document findings
3. **Assess** - plan fix, estimate effort
4. **Rewrite** - try isolated reproduction
5. **Throw away** - delete if unfixable, rewrite from scratch

## Environment: Use Real Browsers

Prefer Vitest Browser Mode over JSDOM/HappyDOM:

```js
// vitest.config.ts
export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },
  },
})
```

JSDOM polyfills create false confidence. Test components where users use them.

## Code Coverage

Use sparingly as a discovery tool, never as a target. 100% coverage can still miss bugs.

## Further Reading

For detailed patterns, see:
- [references/vitest-patterns.md](references/vitest-patterns.md) - Vitest configuration and defaults
- [references/component-testing.md](references/component-testing.md) - Component testing patterns
