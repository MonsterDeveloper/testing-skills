# Vitest Patterns

## Table of Contents

1. [Configuration Defaults](#configuration-defaults)
2. [Test Isolation](#test-isolation)
3. [Async Patterns](#async-patterns)
4. [Browser Mode](#browser-mode)
5. [Mock Patterns](#mock-patterns)

## Configuration Defaults

Vitest's defaults are designed for reliability:

```js
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // Auto-reset mocks between tests (recommended)
    mockReset: true,

    // Browser mode for component tests
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },

    // Polling interval for expect.poll()
    expect: {
      poll: { interval: 250 }
    }
  }
})
```

### Key Defaults

- **ESM-first**: Native ESM, TypeScript, JSX support without configuration
- **Vite config reuse**: Same transformation pipeline as production
- **Opt-in globals**: Import `test`, `expect` explicitly (reduces magic)
- **Isolated test files**: Each file runs in its own worker
- **Parallel files**: Files run simultaneously for performance
- **Sequential test cases**: Tests within a file run in order

## Test Isolation

### File-Level Isolation

Each test file runs in its own worker, preventing state leakage:

```js
// test-a.test.ts runs isolated from test-b.test.ts
```

### Test Case Isolation with Disposables

Use `using` for automatic cleanup:

```js
function createTestServer() {
  const server = new Server()
  return {
    instance: server,
    async [Symbol.asyncDispose]() {
      await server.close()
    }
  }
}

test('handles request', async () => {
  await using server = createTestServer()
  // Server automatically closes when test ends, even if assertions fail
})
```

## Async Patterns

### Promise State Assertions

```js
// Assert resolution
await expect(asyncFn()).resolves.toEqual(expectedValue)

// Assert rejection
await expect(asyncFn()).rejects.toThrow('error message')

// Assert rejection with specific error
await expect(asyncFn()).rejects.toThrowError(CustomError)
```

### Retryable Assertions (Browser Mode)

```js
import { page } from '@vitest/browser/context'

// Auto-retries until passing or timeout
await expect.element(page.getByText('Loaded')).toBeVisible()

// With custom timeout
await expect.element(page.getByRole('button')).toBeEnabled({ timeout: 5000 })
```

### Polling Assertions

```js
// Poll until condition is met
await expect.poll(() => counter.value).toBe(5)

// With options
await expect.poll(() => api.status(), {
  interval: 100,
  timeout: 5000,
}).toBe('ready')
```

### Avoid `sleep()` - Use `waitFor()`

```js
// BAD: Arbitrary delay
await sleep(1000)
expect(element).toBeVisible()

// GOOD: Wait for state
await waitFor(() => {
  expect(element).toBeVisible()
})
```

## Browser Mode

### Setup

```bash
npm i -D vitest @vitest/browser playwright
```

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

### Component Testing

```js
import { page } from '@vitest/browser/context'
import { render } from 'vitest-browser-react'
import { Greeting } from './greeting'

test('displays greeting', async () => {
  render(<Greeting name="Kody" />)
  await expect.element(page.getByText('Hello, Kody!')).toBeVisible()
})
```

### Page API

```js
import { page } from '@vitest/browser/context'

// Locators (same across providers)
page.getByRole('button', { name: 'Submit' })
page.getByText('Welcome')
page.getByTestId('container')
page.getByLabel('Email')
page.getByPlaceholder('Enter email')

// Actions
await element.click()
await element.fill('text')
await element.selectOption('value')
```

### Node.js Code in Browser Tests

Use Vitest Commands for server-side operations:

```js
// vitest.config.ts
export default defineConfig({
  test: {
    browser: {
      commands: {
        readFile: (ctx, path) => fs.readFile(path, 'utf-8'),
      },
    },
  },
})

// test.ts
import { commands } from '@vitest/browser/context'
const content = await commands.readFile('/path/to/file')
```

## Mock Patterns

### Function Mocks

```js
const fn = vi.fn()

// With implementation
const fn = vi.fn(() => 'mocked')

// With return value
const fn = vi.fn().mockReturnValue('value')
const fn = vi.fn().mockResolvedValue('async value')

// Conditional returns
const fn = vi.fn()
  .mockReturnValueOnce('first')
  .mockReturnValueOnce('second')
  .mockReturnValue('default')
```

### Spies

```js
// Spy on object method
const spy = vi.spyOn(object, 'method')

// Spy with mock implementation
const spy = vi.spyOn(console, 'log').mockImplementation(() => {})

// Restore original
spy.mockRestore()
```

### Module Mocks

```js
// Mock entire module
vi.mock('./utils', () => ({
  formatDate: vi.fn(() => '2024-01-01'),
}))

// Partial mock (keep other exports)
vi.mock('./utils', async (importOriginal) => {
  const actual = await importOriginal()
  return {
    ...actual,
    formatDate: vi.fn(() => '2024-01-01'),
  }
})
```

### Mock Lifecycle

```js
// Clear call history
fn.mockClear()
vi.clearAllMocks()

// Clear calls + remove implementation
fn.mockReset()
vi.resetAllMocks()

// Completely remove mock
fn.mockRestore()
vi.restoreAllMocks()
```

### Global Config

```js
// vitest.config.ts
export default defineConfig({
  test: {
    // Recommended: auto-reset between tests
    mockReset: true,

    // Alternative: auto-restore after each test file
    restoreMocks: true,
  }
})
```
