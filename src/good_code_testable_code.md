# Good Code, Testable Code

## What is testability?

You can find many definitions of software "testability" in the wild. Unfortunately, all of them come down to terms like "how easy it is to test a system" or "to which degree or extent your code can be tested". I am saying it's unfortunate because terms like "easy" and "difficult" are inherently relative, and, as a result, subjective.

Your ease of writing automated tests will grow as your skills grow. Something that took you a day to test will take you an hour. Does that mean the testability of that code improved? No, not really. Only you have.

I would like to propose a more tangible definition of software testability. The one that would apply to someone who's just starting their testing journey just as well as it would to a battered software testing professional.

> _**Testability** is a characteristic of software that describes the complexity of the testing setup in relation to the complexity of the tested code._

It is important to stress that testability cannot be observed through the code alone. It is always the relationship the code and its complexity have with the complexity of the testing setup.

Consider this function below:

```ts
import { toAbsolutePath, cleanUrl } from './utils.ts'

export function normalizePath(path: string | RegExp) {
  if (path instanceof RegExp) {
    return path
  }

  const absolutePath = toAbsolutePath(path)

  return cleanUrl(absolutePath)
}
```

Can you tell how testable is the `normalizePath` function based on its implementation alone?

I don't think you reasonably can.

The only thing you can judge based on the code is its complexity. You can reason whether the `path` argument should be a union or, perhaps, should instead accept a plain string and check the regular expression input beforehand to make the function simpler. You can try to see how well the function implements its intention and how true it stays to the best practices of engineering while doing so.

You won't be wrong doing any of those things. But they are not directly related to this function's testability, neither should they be influenced by it.

## Relationship Between Complexity

The relationship between the system complexity and the test setup complexity is a tricky one to define. Complex code does not mean less testable code, and vice versa.

Take these two functions as an example:

```ts
export function add(a: number, b: number) {
  return a + b
}
```

```ts
export async function updatePost(postId: string, payload: Post) {
  const existingPost = await db.posts.findFirst({ id: postId })

  if (!existingPost) {
    throw new InputError(`Cannot find post with id "${postId}"`)
  }

  const nextPost = await existingPost.update(payload)

  return nextPost
}
```

Objectively, the `add` function is less complex than the `updatePost` function. Does it mean it's more testable? Should you try for all your function to be pure and simple like `add`?

No, I don't think you should.

I think you should make software design decisions based on what that software, or its part, is supposed to do. Software complexity is not bad as long as it's justified. The same stands true for the complexity of tests.

Following the examples from above, you would need no setup to reliably test the `add` function. In the case of `updatePost`, you'd likely want to mock the connection to the database in one way or another to prevent it from affecting the test results. Both of them are perfectly testable despite being different in complexity.

But what about the cases of worse testability?

I'm glad you asked. Here's another example of a simple function:

```ts
export async function isLegacyUser(userId: string) {
  const user = await db.users.findFirst({ id: userId })

  return user?.type === 'legacy'
}
```

On the surface, the `isLegacyUser` function is not that complex because the intention behind it is simple: check if the user's `type` is `"legacy"` or not.

If we try to test this function, however, we will quickly discover that the complexity of its testing setup outweighs the complexity of the function itself:

```ts
// is-legacy-user.test.ts
import { db } from './db.js'
import { isLegacyUser } from './is-legacy-user.js'

afterEach(() => {
  vi.resetAllMocks()
})

afterAll(() => {
  vi.restoreAllMocks()
})

test('returns true for a legacy user', async () => {
  vi.spyOn(db.users, 'findFirst').mockResolvedValue({
    id: 'abc-123',
    type: 'legacy',
  })

  await expect(isLegacyUser('abc-123')).resolves.toBe(true)
})

test('returns false for a regular user', async () => {
  vi.spyOn(db.users, 'findFirst').mockResolvedValue({
    id: 'abc-123',
  })

  await expect(isLegacyUser('abc-123')).resolves.toBe(true)
})
```

Suddenly, due to the function's dependency on `db`, we need to mock the database query in the test. The intention behind `isLegacyUser` was never to _fetch_ the user, just to check some if its properties. The implementation is unreasonably complex, and we can see that complexity leaking into the testing setup as well.

Which brings me to another point . . .

## Testability As an Implicit Test

> _A testability of the code is an implicit test in itself._

This is such a powerful mindset to embrace. Well-written code will always be easier to test, just as poorly written code will inevitably remind you about the poor decisions behind it through the complexity of its tests.

You can leverage code's testability to spot mistakes in your code. Perhaps it does too much, and you should reconsider its scope. Or, maybe, it violates the Law of Demeter, referencing things it normally shouldn't. Things like those often manifest in test and in the testing setup in particular, since the relationship between the code's and the test's complexity is seldom linear.

That being said, one shouldn't allow the tests to guide their design decisions.

The testability is a good extra check to perform to analyze your system, but it must never be the driving factor behind its implementation. Both the test and the implementation are derived from your intention to achieve something, where the former describes that intention in code while the latter implements it. As I've mentioned in The True Purpose of Testing, the two may be the same for simple units of logic, but it's not uncommon to have dozens, if not hundreds of lines of code that achieve a single thing.

Moreover, letting the testability guide your implementation can decrease the quality of your code.

A prominent example of that is one of the common pieces of advice on how to make your code more testable through dependency injection:

```ts
export function query(address: string) {
  sql.open(address)
}
```

The example advocates that in order to improve the testability of the `query` function we can accept the `sql.open` function as an argument, which would allow us to replace it with a mock in test.

```ts
// Now, any consumer (including tests) can pass any
// matching function as the `open` argument.
export function query(open: (address: string) => void) {
  open()
}
```

In fact, this is a common practice in other languages, where, I believe, the tools for dependency mocking are rather lacking.

Because what's happening in this example is you changing the call signature of a function _just so it's easier for **you** to test it_. But you don't write code for _you_, you write it for whichever consumer it has (a user of your website or another developer). You should craft great API experiences for the consumer, not your tests.

Remember that you write tests to describe the intention behind the system. And just as it is discouraged to let implementation details leak into your tests, it should be discouraged to let your tests affect the code they are testing.

In this particular example, the original `query(address: string)` call signature could instead be preserved, and the dependency on the `sql` object could be handled in tests (by mocking the `sql` object, as one of the solutions). This decouples the function design from the tests, and allows both the implementation and tests fulfill their purpose within their own constraints.

> Note that dependency injection is a viable technique in testing, and you should definitely take advantage of it if your system is designed to accommodate for it. Just make sure you are designing it for its consumers, not your tests.

## Improving Testability

Those have been a lot of words, but how do you make your code more testable?

Start from following the best practices for writing code. Those still matter and can help you in more ways than one. Good code is always easier to test, so strive toward the "good" code, whatever definition your language, CTO, or you yourself give it.

But if writing good code was enough to get testable code, you wouldn't be reading this right now. There is obviously more to testability than just adhering to the best practices.

Testability is a constant observation as your code changes over time. You have to analyze the relationship between the code's complexity and the test's complexity. Use every mismatch as an opportunity to revisit the design decisions behind your code, and see if changing them would improve the complexity ratio.

Invest into the testing setup by providing utility functions to run and interact with your code in tests. The testing setup phase is where you will be paying for the complexity of your code the most. Make mocking a database a matter of a single function call. Encapsulate repetitive actions in tiny functions. Allow the same test suite to run against local and staging through environment variables. Your system will guide you toward the setup you miss as long as you pay it due attention to it. All of that will be crucial in removing the perceived complexity from your testing setup that's created due to the lack of the latter.
