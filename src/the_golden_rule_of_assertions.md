# The Golden Rule of Assertions

What if I told you there was a single rule that can reliably tell apart a good test from a bad one?

This is not a trick, not a sales pitch. It’s a rule I’ve been using for years to help me refine my tests, make them less flaky and more valuable countless times. In fact, it’s such an indispensable rule in practice that I hereby coin it **The Golden Rule of Assertions**:

> A test must fail if, and only if, the intention behind the system is not met.

We talked about the intentions and the role they play in the purpose of testing before. Now, it’s time to get some practical value out of adopting that mental model.

## Assertions

The heart of every test lives in its assertions. Those are the expressions that compare the state of the system we expect and the state we get. Expressions like this one:
```
expect(this).toEqual(that)
```

If that comparison fails, the assertion throws an error, letting us know that something is wrong. But it’s when the assertion throws that matters the most.

Let’s go through a few example of how The Golden Rule of Assertions can point out some critical mistakes you make in your tests.

## Implementation details

Consider the following code:

```
import { cookieUtils } from './utils'

export function parseCookieName(cookie: string) {
  const { name } = cookieUtils.parse(cookie, { pick: ['name'] })
  return name
}
```

The intention behind the `parseCookieName` function is to accept a cookie string and return the name of the cookie present in it. To do it, it relies on some `cookieUtils` object for the actual parsing.

Now, a test for such a function may look something like this:

```
import { parseCookieName } from './parseCookieName'
import { cookieUtils } from './utils'

it('returns the cookie name from the given cookie', () => {
  const cookieName = parseCookieName('sessionId=abc-123')

  expect(cookieUtils.parse).toHaveBeenCalledWith(
		'sessionId=abc-123',
		{ pick: ['name'] }
	)
  expect(cookieName).toBe('sessionId')
})
```

Since we only expect the cookie name, we are also asserting that the `cookieUtils.parse` has been called with the right `pick` argument. Seems reasonable enough.

But what will happen if the `parseCookieName` function doesn’t need `cookieUtils` anymore? Perhaps we adopt a well-established third-party package to do the parsing for us, or even implement a basic version of it ourselves.

Well, the test will fail:
```
× returns the cookie name from the given cookie
  Error: expected the "cookieUtils.parse" to be called but it never was.
```

It’s good when tests fail. We write them so they do fail when something is wrong. But here’s the thing: is something wrong with the `parseCookieName` function? We don’t even know. The assertion that checks the returned cookie name isn’t even run because the `expect()` call that comes before it throws an error.

The test fails while the intention behind `parseCookieName` isn’t necessarily broken. In fact, if we remove the `cookieUtils` assertion, the test passes! So the code was fulfilling the intention all along but the test still failed.

In this particular case, it happened because we introduced an assertion over the implementation detail—the usage of the `cookieUtils` object. It doesn’t really matter how we get the cookie name. As long as we do that, the function does what it was intended to do.

This is why testing implementation details is discouraged. Because if those details change or get removed, your test will fail even if the code you are testing remains perfectly functional. The Golden Rule of Assertions helps you decide what to test and what to omit.

> The implementation may change but the intention stays the same.

## Test boundaries

Okay, we learned our lesson and don’t include implementation details in our tests anymore. Is this all this fancy new rule is about? Do we even need it now if that’s all it does?

I have another example for you:

```
export async function fetchUser(id) {
  const response = await fetch(`https://example.com/user?id=${id}`)
  const user = await response.json()
  return user
}
```
The `fetchUser` function should fetch the user by `id`. How it does so—using the fetch function—doesn’t matter in the context of the test, and so we write it accordingly:

```
import { fetchUser } from './fetchUser'

it('fetches the user by id', async () => {
  const user = await fetchUser('abc-123')
  expect(user).toEqual({ name: 'Cody' })
})
```

Now, if we replace `fetch` with React Query, for example, the test will still pass as long as the returned user has the right value. No implementation details involved, we should be fine (you see this isn’t the end of the article, you know we aren’t fine).

What will happen to this test if the server returns an error? Or if the connection to https://example.com times out?

You guessed it—the test will fail.

```
× fetches the user by id
  TypeError: Failed to fetch
```

Although we don’t mention any implementation details in our test, it still implicitly relies on one external factor—a GET request to the https://example.com/user endpoint. Because of this, apart from validating the `fetchUser` function, the test now also concerns itself with the validity of the server. This blows the scope of this test from a simple function to the network connection, server runtime, response timings… There are way too many moving parts—parts we may not own or control—for this test to ever be reliable.

> Despite the `fetchUser` function depending on the network call, it cannot guarantee the validity of the server, neither is it responsible for it. All it cares about is to make the right request and handle the response to return the expected user object.

None of those factors determine the validity of the fetchUser function. And to correctly test that validity, we must take the HTTP interaction out of the testing equation. In this case, we can reach out for API mocking tools like Mock Service Worker to make that network request a fixed, predictable given in our test.

> Keep in mind that mocking is just a tool to establish boundaries. In the context of this integration test, we are taking the network out because we gain no value from it at this level. We’d still have to test the `fetchUser` function end-to-end where keeping that request un-mocked would be essential. Let me know if you would like me to cover mocking in more detail in future articles!

## Conclusion

I would like to stress that it’s not about mocking network calls or testing implementation details. Those are just symptoms of a larger problem when the test fails for any other reason than the broken intention. **When it doesn’t pass The Golden Rule of Assertions.**

In practice, this rule can encourage many best practices and help you spot problematic tests before they become such. It directs the spotlight on the importance of the testing setup and test scope, handling side effects and external dependencies, correctly written assertions and even incorrect implementations also.

So the next time you write a test, I want you to ask yourself: **When will this test fail?**

Luckily, you know exactly the rule to follow. The kind that will help you get the most out of your tests so they would shine like gold.
