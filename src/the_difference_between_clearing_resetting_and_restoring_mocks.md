# The Difference Between Clearing, Resetting, and Restoring Mocks

One source of confusion around mocking is that mocks can be stateful.

Like this mock function:

```js
const fn = vi.fn()

fn('one')
fn('two')

fn.mock.calls
// [ ["one"], ["two"] ]
```

In the example above, the `fn` mock function has a state that keeps track of all the calls made to it. The state itself isn't the issue. In fact, you need that state! You need it to assert on the right number of calls and their arguments during the test.

**A state starts causing problems once it's mismanaged.**

When it changes or persists when you aren't expecting it to. An extremely common example of a mismanaged state is this:

```js
const listener = vi.fn()

test('calls the listener when the event is emitted', () => {
  const emitter = new Emitter()
  emitter.on('hello', listener)
  emitter.emit('hello')

  expect(listener).toHaveBeenCalledTimes(1)
})

test('does not call the listener after `.removeAllListeners()` has been called', () => {
  const emitter = new Emitter()
  emitter.on('hello', listener)
  emitter.removeAllListeners()
  emitter.emit('hello')

  expect(listener).not.toHaveBeenCalled()
})
```

Here, we have to test cases for the `Emitter` class. The first one expects the listener to be called when the event is emitted, and the other expects no calls to the listener because all the listeners have been removed.

In order to test that, we're introducing a mocked `listener` function to assert on the calls.

But if we run this test suite, we'll get an error:

```
FAIL  src/example.test.ts > does not call the listener after `.removeAllListeners()` has been called

AssertionError: expected "spy" to not be called at all, but actually been called 1 times

Received:

1st spy call:
    Array []

Number of calls: 1
```

Although our test cases are scoped, declaring and manipulating their own resources, the `listener` function becomes a shared state between them. As a result, the call to `listener` from the first test leaks to the second one, where no calls are expected.

There are a few ways we can solve this problem. We can declare the `listener` mock function in individual tests to stop it from being shared altogether, or we can manage that mocked function's state.

Let's talk about the second approach.

## Ways to manage mock state

Both Vitest and Jest provide you with great APIs to manage the state of your mocks:

- `.mockClear()` / `vi.clearAllMocks()`
- `.mockReset()` / `vi.resetAllMocks()`
- `.mockRestore()` / `vi.restoreAllMocks()`

You can apply these methods on individual mock functions, as well as on the entire test suite.

There's just one thing.

**The names of these methods are really alike.**

It's time to clear the confusion once and for all.

## Before we begin

First, it's important to acknowledge that any mock function tracks two things:

- The list of calls (i.e. how that function is being called)
- The function implementation (i.e. what that function does when called)

All the state management methods we are going to talk below revolve around helping you reset one or both of those things.

## Clearing mocks

Imagine your mock function as a box. Every call to that function puts a nice blue ball into the box. You can check how many balls are in the box at any point in time, but there may come the time when you need to empty that box altogether.

That's what the `.mockClear()` method does. It clears all the recorded calls of the mock function, and that's it. No other changes to the mock.

```js
const fn = vi.fn()

fn('one')
expect(fn).toHaveBeenCalledTimes(1) // ✅

// Makes the mock function forget about
// any calls that happened before this point.
fn.mockClear()

fn('two')
expect(fn).toHaveBeenCalledTimes(1) // ✅
```

The `.mockClear()` method is a good choice to clear the recorded call information within the same test. This comes in handy when testing complex, often repetitive scenarios that involve the same mock function.

```js
fn('one')
// ...assert

fn.mockClear()

fn('two')
// ...assert
```

## Resetting mocks

Building on top of our box analogy, let's imagine that in addition to putting balls in our box we can also put a smaller box inside of it (the mock implementation). If we do, that smaller box will get all the balls now, and do with them however we please.

Resetting a mock means both clearing its calls and removing any mock implementation it may have. It empties the box and throws away the smaller box inside of it, if any.

The `.mockReset()` method is extremely handy when you're dealing with functions that have a mocked implementation. And that may involve both mock functions and spies!

```js
// Let's spy on console!
// Calling `vi.spyOn` wraps the global `console.log` function
// in a mock function. Let's also provide a mock implementation
// so console calls in test don't actually print anything.
const spy = vi.spyOn(console, 'log').mockImplementation(() => {})

console.log('you will not see this') // nothing is printed!
expect(console.log).toHaveBeenCalledTimes(1) // ✅

// Makes the mock forget about any calls to it
// and also removes any mock implementation it may have.
spy.mockReset()

console.log('you will see this!') // prints "you will see this"
expect(console.log).toHaveBeenCalledTimes(1) // ✅
```

The fact that the mock itself remains active make `.mockReset()` / `vi.resetAllMocks()` a perfect choice for clearing the mock's state between tests:

```js
afterEach(() => {
  // Make sure that no state leaks
  // between mocks in the tests.
  vi.resetAllMocks()
})
```

## Restoring mocks

In our box analogy, restoring a mock simply means throwing the box away.

The `.mockRestore()` method effectively undoes the mock, removing both its recorded calls and any mocked implementation. It is only relevant for spies. The function we spy over always has either our mock implementation or the original implementation in place, and restoring that spy means restoring that function to its original, "clean" state.

Here's how it looks like in code:

```js
const spy = vi.spyOn(console, 'log')

console.log(1)
expect(console.log).toHaveBeenCalledTimes(1) // ✅

// Revert this mock, restoring the original implementation
// of the spied function (console.log).
spy.mockRestore()

expect(console.log).toHaveBeenCalledTimes(0)
// ❌ TypeError: [Function log] is not a spy or a call to a spy!
```

Because of this, the `.mockRestore()` / `vi.restoreAllMocks()` is the go-to choice for cleaning up after yourself in hooks like `afterAll`:

```js
afterAll(() => {
  // Make sure to restore all mocks
  // once this test suite is done.
  vi.restoreAllMocks()
})
```

## Summary

Let's summarize the difference between those methods:

- `.mockClear()` clears the recorded calls to the mock functions
- `.mockReset()` clears the recorded calls and removes any mock implementation
- `.mockRestore()` removes the mock, undoing it altogether

Since you never want the state of a mock to be shared between tests, it's a good idea to set up automatic reset of mocks in your testing framework. In Vitest, you do that by providing the `mockReset` option in your `vite.config.ts` / `vitest.config.ts`:

```js
export default defineConfig({
  test: {
    mockReset: true
  }
})
```

That's it! I hope you will remember this explanation the next time you are clearing, resetting, or restoring your mocks.
