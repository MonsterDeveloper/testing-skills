# Prefer the .resolves Chaining

Testing asynchronous code can get tricky. By its very nature, those tests deal with eventual state of your system and it becomes your responsibility to await that state before asserting on it. Luckily, both your test runner and the language itself come with utilities to help you tackle asynchronicity.

Today, you will learn a small practical tip that will make your assertions on async logic more useful and you will do so through one of my favorite aspects of testing: designing a great failure experience.

## The async/await syntax

The primary way of declaring and consuming asynchronous logic in JavaScript is the `async`/`await` syntax. You're all familiar with it already: declare your logic with the `async` keyword, wait for that logic to complete with `await`:

```js
// Declare.
async function fetchUser(id: string): User {}

// Consume.
fetchUser('abc-123') // Promise<pending>
await fetchUser('abc-123') // User
```

Your `fetchUser()` function returns a `Promise` and that promise can either resolve with the data or reject with an error. The `await` keyword allows you to wait for when it resolves and retrieve the data.

But what about rejections?

For those, you'd use the `.catch()` method on the promise to catch rejection errors:

```js
await fetchUser('abc-123').catch((error) => {
  const fetchUserError = new Error('Fetching the user failed', { cause: error })
  console.error(fetchUserError)
})
```

Just like the quality of error handling tells you about the quality of the software, the quality of error experience tells you about the quality of the test.

> After all, we write tests for them to fail and it matters a lot when and how they fail.

And yes, as the author of tests, you become the designer of that failure experience.

## Asserting on asynchronous

Just like in the rest of your code, you should use `async`/`await` to handle asynchronicity in your tests. But how you use those keywords might lead to a completely different experience when your tests fail. Let me show you.

Let's start with a simple assertion around our `fetchUser()` function:

```js
test('fetches a user by id', async () => {
  expect(await fetchUser('abc-123')).toEqual({ id: 'abc-123' })
})
```

We expect that fetching the user with this ID will resolve to the given user object. Pretty straightforward. But this is what we see. What does the test runner see when it runs this test?

It sees this:

```js
expect(user).toEqual({ id: 'abc-123' })
//     ^^^^
```

JavaScript will `await` the promise returned from `fetchUser()` and provide the resolved value to the `expect()` function. In other words, it is the same as passing a `user` object directly to `expect()`.

But what will the `user` argument be if the `fetchUser()` promise rejects?

**An unhandled exception!**

Modern test runners are capable of catching such exceptions during the test run and presenting them to you. For example, here's what you would see in Vitest if the `fetchUser()` function rejected in your test:

```
FAIL  fetch-user.test.ts > fetches the user by id

Error: Failed to fetch user by id "abc-123"

❯ fetchUser fetch-user.ts:2:9
      1| export async function fetchUser(id: string) {
      2|   throw new Error(`Failed to fetch user by id "${id}"`)
       |         ^
      3| }
      4|

❯ fetch-user.test.ts:5:16
```

Notice how the error is pointing to the `fetchUser()` function where the exception originated from and not your test. Not at first, at least. The test file is, too, included in the stack trace at the very bottom: `fetch-user.test.ts:5:16`. Why is that so?

**Because it's not your test runner's job to assume where unhandled exceptions originate from**. Location `5:16` in your test file can point to an assertion, or to a utility function you call in the `test()` block, or to a third-party library misbehaving in `beforeAll()`. All of these scenarios will be reported to you in the same way and it will take you extra steps to untangle the stack trace to trim down what exactly is going on.

Or you can do better. This is precisely the moment where you become the designer of failure (trust me, it's much cooler than it sounds). Because as it turns out, we did a pretty big mistake of not telling our test runner what we actually expected from the `fetchUser()` function.

## The .resolves/.rejects chaining

We've already established that awaiting the user via `await fetchUser('abc-123')` in the test is equivalent to passing the `user` object to `expect()` directly. This completely skips a crucial part of our expectation toward the tested function:

> The `fetchUser()` function must resolve to a given user.

Instead of clearly telling that to the test runner through our assertion, we assumed the tested function always resolved and only verified the value. The resolution/rejection state of our logic is a part of that logic. A part that also has to be tested.

And to tell that to our test runner, it has to be aware of that promise's state. In other words, it can't assert on a resolved/rejected value, it has to assert on the promise itself.

```js
expect(fetchUser('abc-123')).toEqual({ id: 'abc-123' }) // ❌
//     ^^^^^^^^^^^^^^^^^^^^ // Promise<pending>
```

Only this won't work at all! We don't want to check that the promise equals to a user. It obviously doesn't! What we want is to express the expected promise state: whether it should resolve or reject, and then with which data or error.

Modern test runners come with built-in utilities to help you express that expectation. I'm talking about the `.resolves` and `.rejects` chaining:

```js
await expect(fetchUser('abc-123')).resolves.toEqual({ id: 'abc-123' })
```

Let's unwrap what changed here:

- We passed the promise returned from `fetchUser()` directly to the `expect()` function
- Then, instead of tapping into a matcher (`.toEqual()`) directly, we added the `.resolves.` chaining. By doing so, we made any subsequent matchers we access asynchronous! Yes, calling `.resolves.toEqual()` now returns a promise, which means...
- We had to await the entire `expect().resolves[matcher]` expression. The last thing we want are rogue promises yielding false-positive results in our tests.

Take a look at how this transforms what you see if this assertion fails:

```
FAIL  fetch-user.test.ts > fetches the user by id

AssertionError: promise rejected "Error: Failed to fetch user by id \"abc-12…\" instead of resolving

❯ fetch-user.test.ts:5:36
      3|
      4| test('fetches the user by id', async () => {
      5|   await expect(fetchUser('abc-123')).resolves.toBe('hello world')
       |                                    ^
      6| })
      7|

Caused by: Error: Failed to fetch user by id "abc-123"
❯ fetchUser fetch-user.ts:2:9
❯ fetch-user.test.ts:5:16
```

**Notice how the failed assertion takes the spotlight**. You immediately see which of your expectations didn't pass and then you see why. The original exception is preserved both as the assertion rejection reason listed in `Caused by:` and as a frame in the stack trace below: `fetch-user.ts:2:9`.

The error itself hasn't changed. It originates from `fetchUser()` and surfaces in `fetch-user-test.ts`. What changes is that the error is now handled. It fails the assertion because we do not expect our promise to reject. It has to resolve.

Similarly, you can employ the `.rejects` chaining to test asynchronous error scenarios!

## Conclusion

Designing the error experience in tests sits in a perfect spot between complexity and practical usefulness. You don't have to push it too far, you just have to think about it. When will this test fail? How can I make those failures help me debug and fix the issue faster?

There are a lot of things that influence how your test fails. It can be custom fixtures, like the `createMockDatabase()`, that allows you to explore the state of your database after failures; or soft assertions with `expect.soft()` that let you run multiple assertions even if one of them failed. And, of course, it is the `.resolves`/`.rejects` chaining when it comes to testing anything asynchronous.
