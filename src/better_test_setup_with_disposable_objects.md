# Better Test Setup with Disposable Objects

Almost six years ago, Kent C. Dodds wrote a fantastic piece called "Avoid Nesting When You're Testing". I do not exaggerate when I say following the advice from that article has been one of the most impactful testing decisions I've ever made.

## Prefer flat tests

Think of your test suite. You've got your software and you've got a bunch of different behaviors to put in automated tests. You want those behaviors to be clear on the test case level so you work hard to give those cases meaningful names. Some of those behaviors require similar setup, so you follow the DRY principle and use hooks like `before` and `after` to achieve more by writing less.

And, eventually, your test file ends up looking like this:

```js
describe('authentication', () => {
  let testServer

  beforeAll(async () => {
    testServer = createTestServer()
    testServer.get('/user', userHandler)
    await testServer.listen()
  })

  afterAll(async () => {
    await testServer.close()
  })

  describe('sign in with Google', () => {
    beforeEach(() => {
      testServer.post('/auth', authHandler)
    })

    describe('with a returning customer', () => {
      test('displays previously used auth suggestion', () => {
        // ...
      })
    })
  })
})
```

Despite the visual clutter, everything above is just the setup for a single test case. It is the most important phase of the test spread thin across multiple hooks and closures, and as a result, it requires not a small degree of mental gymnastics to know what affects that test case.

But the thing is, the test scenario itself is also quite complex. It involves a specific authentication provider and a particular state of the user and then exhibits the tested behavior (displays the auth suggestion). **The real mistake here is letting that complexity leak into the test setup**.

When faced with a problem, most developers will try to solve it in a clever way. For some, it would be the habit that guides them, for other their hubris. And, sometimes, they would be right to think clever.

But not in tests.

> Your tests is the absolute worst place to get smart.

If you abandon the notion to architect and be clever in tests, the same example I used above can be presented like this:

```js
// tests/auth/google.test.ts
test('displays previously used auth suggestion for a returning customer', async () => {
  const testServer = createTestServer()
  testServer.get('/user', userHandler)
  testServer.post('/auth', authHandler)
  await testServer.listen()

  // ...

  await testServer.close()
})
```

Huh, and what do you know, it's even shorter then before (not that it's indicative of anything). The important part here is that the test case is **explicit in its prerequisites**. You immediately see the setup it needs to run. Not only that, but you can change that setup, knowing you won't be affecting anything outside of its scope!

> Notice how I'm utilizing the folder structure and test file names to replace the need for `describe()` blocks. Your file system is your best friend when it comes to splitting complex test scenarios.

Keeping your tests flat is a great way to foster isolation and predictability. As much as I love this approach, it has one big problem.

## The problem

**It's the cleanup.**

What happens if something throws before the `await testServer.close()` line is executed? Something like, I don't know, a failing assertion! Those are quite common in tests. Well, the function's closure will short-circuit and the test server will remain running, eating resources, making the test run slow, and, maybe, even introducing a shared state across the tests.

Yikes.

To be fair, the flat test approach isn't exactly to blame here. That's how functions work in JavaScript, after all. Unfortunately, the testing frameworks don't provide any better way to arrange the cleanup on the test case level (or do they?).

Here's what you can do to solve this problem for good.

## Disposable objects

Disposable objects is a part of the Explicit Resource Management proposal to TC39 that allows you to describe how an object will be garbage-collected. In simplified form, you can use a disposable object to run a callback when that object is no longer needed.

Here's a quick example:

```js
using client = {
  id: '5fe3dae6-be69-4ff6-afe6-cdf6a729f730',
  [Symbol.dispose]() {
    console.log('Disposed!')
  }
}

console.log(client.id) // "5fe3dae6-be69-4ff6-afe6-cdf6a729f730"
// "Disposed!"
```

`using` is a special keyword that lets JavaScript know you are consuming a disposable object. The object itself describes how to dispose of it in the root-level `[Symbol.dispose]()` method.

That's why in the above example we will see the client ID printed first and then, when the `client` object gets removed from memory, the "Disposed!" message will be printed in the console.

> In addition to `Symbol.dispose`, you can also specify the `Symbol.asyncDispose`, which allows you to execute asynchronous logic in your dispose callback.

Disposable objects are **incredible** for test utilities because they allow you to collocate utility logic with its cleanup.

Using disposable objects, we can refactor the `createTestServer()` test utility to ensure proper server closures:

```js
export function createTestServer() {
  const testServer = new Server()

  return {
    instance: testServer,
    async [Symbol.asyncDispose]() {
      // Close the server, abort pending requests, etc.
      await testServer.close()
    }
  }
}
```

And make sure we are consuming the test server with the `using` keyword in tests:

```js
test('displays previously used auth suggestion for a returning customer', async () => {
  await using testServer = createTestServer()

  testServer.instance.get(...)
  await testServer.instance.listen()

  // ...
})
```

> Asynchronous disposable objects must be consumed via `await using`, which means "await the disposable callback of this object". And if your test utility itself returns a promise, do not forget to await it, too! E.g. `await using value = await utility()`.

Not only did we achieve a reliable cleanup with this refactor, but we've also made our test setup leaner since we no longer need to close the test server manually at the end of each test. Once the test's closure is garbage-collected, the test server is guaranteed to close.

## Disposable objects availability

At the moment of writing this, the Explicit Resource Management proposal is at stage 3, which means that disposable objects are coming to JavaScript but aren't here just yet.

**But you can still benefit from them today.**

### Using TypeScript (Recommended)

You can use disposable object API natively in TypeScript since version 5.2.

### Using polyfill

If you cannot use TypeScript, consider the `disposablestack` polyfill on NPM, which implements the disposable object as well as other adjacent APIs for explicit memory management.

## References

If you liked the disposable HTTP test server example, I highly recommend you take a closer look at `@epic-web/test-server`, which is a full-fledged version of that utility (it also supports WebSockets!). Use in your tests like I do or use it as a reference when implementing your own disposable test utilities.

Good luck!
