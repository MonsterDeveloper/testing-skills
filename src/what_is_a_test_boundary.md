# What Is A Test Boundary?

If automated testing was a math formula, it would look something like this:

```
Tested Code + Action = Expectation
```

That is an automated test in a nutshell:

1. You take the code you wish to test;
2. You perform actions to get it to the right state;
3. You check that the state (or value) is as expected.

Let's refer to the formula above as _the testing equation_. Here's how that equation manifests in an actual test:

```js
import { fetchUser } from './fetch-user.js'

test('fetches the user by id', async () => {
  await expect(fetchUser('abc-123')).resolves.toEqual({ firstName: 'John' })
  // fetchUser() function + called with "abc-123" = this user object
})
```

> These three members of the equation are also the three phases of any test.

In practice, you are often testing code that exists as a part of a larger system. That code may depend on other code or execute side effects that may or may not be relevant to the intention you are trying to test.

Let's take the `fetchUser()` function as an example and see how it's implemented:

```js
import { toCamelCase } from './to-cammel-case.js'

export function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/user/${id}`)
  const user = await response.json()
  const normalizedUser = toCamelCase(user)
  return normalizedUser
}
```

There are a couple of things happening in this function:

1. First, it makes an HTTP call to the `/user/:id` endpoint to fetch the user;
2. Then, it reads the response body as JSON in the `user` object.
3. Finally, it normalizes the `user` object to ensure that all of its keys are in camel case.

All these steps add up to what becomes the `fetchUser()` function. Using the substitution method, we can replace the `fetchUser()` function in the equation with the sum of its counterparts:

```
Request to /user/:id
Read response as JSON    + Action = Expectation
Normalize object keys
```

This gives us a complete view on every piece involved in this particular test. While each piece brings its own logic, it is the `fetchUser()` function that puts it together in a correct order.

Why is this useful? Because . . .

> _Every member of the testing equation affects the test's result._

The biggest value you get from tests is knowing when they are supposed to fail. For our `fetchUser()` function, the test will fail in these cases:

1. Network error. If the request to `/user/:id` failed for any reason, none of the dependent code will execute correctly;
2. Server response. If the server responds with a non-JSON response, or that response is not the object we expect (e.g. an error), the returned user will not match the expected one in test;
3. Bugs in `toCamelCase()`. If the utility function fails to normalize the `user` object keys, the actual user object still won't match the expected one.

Knowing this, you are in charge of deciding _when_ this test should fail. Your decision will affect both the reliability of your test and also the behavior(s) that you are testing.

One of the primary things you'd want to do in this case is prevent the network from affecting the test results. The operability and correctness of the server is not the tested function's concern. The function doesn't affect the server and cannot guarantee its behavior. As such, the test must not fail due to the things that lie outside of the tested code's concern.

A common way to achieve that is by using API mocking. By mocking the response, you _take the network out of the testing equation_, transforming it from variable to given. The only things that actually affect the test now are the logic in `fetchUser()` and its dependency on `toCamelCase()`.

Whenever you take something out of the testing equation, you establish a _test boundary_.

## Test boundaries

> _The test boundary is the extent of the tested system executed in test._

You can think of the test boundary as an imaginary line you draw through your code telling the test that nothing past that line matters.

**Knowing when and how to establish the test boundary is essential to any test**.

When done poorly, it leads to brittle and flaky tests that you'd wish to delete (and you probably should). When done correctly, it gives you full control over _what_ it is you are testing and what value the test failures bring you.

The test boundary is inevitably connected to mocking. In fact, mocking at its core is a tool designed to establish that boundary in your tests.

> _A chisel to a block of marble is what mocking is to your tested code._

By removing the network from the `fetchUser()` test, you focus that test on the behavior of the function and also its dependency on `toCamelCase()` (the remaining members of the testing equation). If you also decide to mock the `toCamelCase()` function, you dial the test's lens even further, now asserting only the right steps and their sequence in `fetchUser()`.

And here's the thing:

> _If you chisel the marble enough times, there will be nothing left._

Since the test boundaries help you decide what doesn't matter in test, overdoing them leads to a test where nothing matters because nothing is actually being tested. This is often referred to as _overmocking_, which is the state you never want to reach.

That begs a question: _Where do I draw the line?_

## Where to put test boundaries?

The things you "take out" from your tested code vastly depend on the code itself and what you want to test. But I hate "it depends" answers so I will try to give you some actionable pointers that apply to any situation.

You can never miss by starting with The Golden Rule of Assertions, which reads like this:

> _A test must fail if, and only if, the intention behind the system is not met._

Eliminate the members of the testing equation that are not related to the intention behind the code. If a function is supposed to fetch a user by their ID, its intention is:

1. To make a request to the correct endpoint with the correct payload;
2. To read, normalize, and return that user response.

Naturally, the part where you draw the boundary is the request and its response.

HTTP requests, side effects (like writing to the file system or interacting with the DOM), and non-deterministic values (like dates or RNG) are some of the common things you always want mocked.

Once you have those taken care of, the test boundary becomes your magnifying glass, and the way you dial it directly affects what kind of test you get. Here's what I mean.

If you are testing a Component A that depends on a Component B, you have a choice. You can either leave that dependency as-is or mock it out. **Neither of those choices is wrong**. You simply get a different test as a result. Mocking the dependency allows you to focus on Component A in what would likely be a _unit test_. Leaving the dependency be, you create an _integration test_, where the way A and B communicate matters and affects the test results.

## Conclusion

I hope you realize and come to love the power the testing boundaries give you when testing. They help you eliminate unwanted code while focusing on the behaviors or integrations that matter.
