# Component Testing Patterns

## Table of Contents

1. [Test Environment Choice](#test-environment-choice)
2. [Rendering Components](#rendering-components)
3. [Querying Elements](#querying-elements)
4. [Assertions](#assertions)
5. [User Interactions](#user-interactions)
6. [API Mocking with MSW](#api-mocking-with-msw)
7. [Common Patterns](#common-patterns)

## Test Environment Choice

### Vitest Browser Mode (Recommended)

Real browser execution. No polyfill issues.

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

### Why Not JSDOM?

JSDOM runs in Node.js, pretending to be a browser:
- Polyfilled APIs diverge from real browser behavior
- `browser` export condition forces wrong bundles
- Events aren't real browser events
- CSS, layout, visibility don't work correctly

Test components where users use them: the real browser.

## Rendering Components

### React (Vitest Browser Mode)

```js
import { render } from 'vitest-browser-react'
import { page } from '@vitest/browser/context'

test('renders component', async () => {
  render(<MyComponent prop="value" />)
  await expect.element(page.getByText('Expected text')).toBeVisible()
})
```

### Vue (Vitest Browser Mode)

```js
import { render } from 'vitest-browser-vue'
import { page } from '@vitest/browser/context'

test('renders component', async () => {
  render(MyComponent, { props: { value: 'test' } })
  await expect.element(page.getByText('Expected text')).toBeVisible()
})
```

### With Providers/Wrappers

```js
function renderWithProviders(ui, options = {}) {
  return render(
    <ThemeProvider>
      <RouterProvider>
        {ui}
      </RouterProvider>
    </ThemeProvider>,
    options
  )
}

test('renders with context', async () => {
  renderWithProviders(<Dashboard />)
  // ...
})
```

## Querying Elements

### Priority Order (Accessibility-First)

1. **`getByRole`** - Most accessible, mirrors how users interact
2. **`getByLabelText`** - Form elements
3. **`getByPlaceholderText`** - Fallback for inputs
4. **`getByText`** - Non-interactive content
5. **`getByTestId`** - Last resort

```js
// BEST: Role queries
page.getByRole('button', { name: 'Submit' })
page.getByRole('heading', { name: 'Dashboard', level: 1 })
page.getByRole('link', { name: 'Home' })
page.getByRole('textbox', { name: 'Email' })

// GOOD: Label queries for forms
page.getByLabel('Email address')
page.getByPlaceholder('Enter your email')

// OK: Text queries
page.getByText('Welcome back!')
page.getByText(/welcome/i) // regex for flexibility

// AVOID: Test IDs (not accessible)
page.getByTestId('submit-button') // only when nothing else works
```

### Vitest Browser Mode: No findBy/queryBy Needed

`page.getBy*` auto-retries, eliminating the need for `findBy*`:

```js
// Just use getBy* - it waits automatically
await expect.element(page.getByRole('button')).toBeVisible()
```

## Assertions

### Presence: `.toBeVisible()`

Checks accessibility criteria (not just DOM presence):
- Present in DOM
- Not `display: none`
- Not `opacity: 0`
- Not `visibility: hidden`
- Not `aria-hidden="true"`
- Parents also visible

```js
// Element is visible and accessible
await expect.element(page.getByRole('alert')).toBeVisible()
```

### Absence: `.not.toBeInTheDocument()`

Only checks DOM presence (faster for absence):

```js
// Element is not in DOM
await expect.element(page.getByRole('alert')).not.toBeInTheDocument()
```

### Element State

```js
await expect.element(button).toBeEnabled()
await expect.element(button).toBeDisabled()
await expect.element(input).toHaveValue('expected')
await expect.element(checkbox).toBeChecked()
await expect.element(link).toHaveAttribute('href', '/path')
await expect.element(element).toHaveClass('active')
```

### Text Content

```js
await expect.element(heading).toHaveTextContent('Welcome')
await expect.element(heading).toHaveTextContent(/welcome/i)
```

## User Interactions

### Vitest Browser Mode

```js
import { userEvent } from '@vitest/browser/context'

const user = userEvent.setup()

// Click
await user.click(page.getByRole('button'))

// Type text
await user.fill(page.getByRole('textbox'), 'Hello')

// Clear and type
await user.clear(page.getByRole('textbox'))
await user.fill(page.getByRole('textbox'), 'New value')

// Keyboard
await user.keyboard('{Enter}')
await user.keyboard('{Control>}a{/Control}') // Select all

// Select
await user.selectOptions(page.getByRole('combobox'), 'option-value')

// Hover
await user.hover(page.getByText('Menu'))
```

## API Mocking with MSW

### Setup

```js
import { http, HttpResponse } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer()

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### Per-Test Handlers

```js
test('displays user data', async () => {
  server.use(
    http.get('/api/user', () => {
      return HttpResponse.json({ name: 'John' })
    })
  )

  render(<UserProfile />)
  await expect.element(page.getByText('John')).toBeVisible()
})
```

### Request Validation in Mocks (Not Assertions)

```js
test('adds product to cart', async () => {
  server.use(
    http.post('/api/cart', async ({ request }) => {
      const body = await request.json()

      // Validation in mock - wrong payload = error response
      if (body.productId !== 'expected-id') {
        return new HttpResponse(null, { status: 400 })
      }

      return new HttpResponse(null, { status: 201 })
    })
  )

  render(<ProductPage productId="expected-id" />)
  await user.click(page.getByRole('button', { name: 'Add to cart' }))

  // Assert on UI outcome, not request details
  await expect.element(page.getByText('Added to cart')).toBeVisible()
})
```

### Error States

```js
test('displays error on network failure', async () => {
  server.use(
    http.get('/api/user', () => {
      return HttpResponse.error()
    })
  )

  render(<UserProfile />)
  await expect.element(page.getByRole('alert')).toHaveTextContent('Failed to load')
})

test('displays error on 404', async () => {
  server.use(
    http.get('/api/user', () => {
      return new HttpResponse(null, { status: 404 })
    })
  )

  render(<UserProfile />)
  await expect.element(page.getByText('User not found')).toBeVisible()
})
```

## Common Patterns

### Testing Forms

```js
test('submits form with valid data', async () => {
  server.use(
    http.post('/api/register', async ({ request }) => {
      const data = await request.json()
      return HttpResponse.json({ id: '123', email: data.email })
    })
  )

  render(<RegistrationForm />)

  await user.fill(page.getByLabel('Email'), 'test@example.com')
  await user.fill(page.getByLabel('Password'), 'password123')
  await user.click(page.getByRole('button', { name: 'Register' }))

  await expect.element(page.getByText('Registration successful')).toBeVisible()
})

test('shows validation errors', async () => {
  render(<RegistrationForm />)

  // Submit without filling
  await user.click(page.getByRole('button', { name: 'Register' }))

  await expect.element(page.getByText('Email is required')).toBeVisible()
})
```

### Testing Loading States

```js
test('shows loading then content', async () => {
  let resolveRequest
  server.use(
    http.get('/api/data', async () => {
      await new Promise(r => resolveRequest = r)
      return HttpResponse.json({ items: ['a', 'b'] })
    })
  )

  render(<DataList />)

  // Loading state
  await expect.element(page.getByText('Loading...')).toBeVisible()

  // Resolve and check content
  resolveRequest()
  await expect.element(page.getByText('Loading...')).not.toBeInTheDocument()
  await expect.element(page.getByText('a')).toBeVisible()
})
```

### Testing Navigation/Routing

```js
test('navigates to details page', async () => {
  render(
    <MemoryRouter initialEntries={['/products']}>
      <Routes>
        <Route path="/products" element={<ProductList />} />
        <Route path="/products/:id" element={<ProductDetails />} />
      </Routes>
    </MemoryRouter>
  )

  await user.click(page.getByRole('link', { name: 'View Product A' }))

  await expect.element(page.getByRole('heading', { name: 'Product A' })).toBeVisible()
})
```

### Inverse Assertions (Something Doesn't Happen)

```js
test('does not show notification on cancel', async () => {
  render(<Form />)

  await user.click(page.getByRole('button', { name: 'Cancel' }))

  // Wait to ensure notification would have appeared if buggy
  const notificationPromise = waitFor(() => {
    expect(page.getByRole('alert')).toBeVisible()
  })

  await expect(notificationPromise).rejects.toThrow()
})
```
