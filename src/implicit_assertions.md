# Implicit Assertions

Assertions are the heart and soul of any test. You write assertions to describe your expectations for particular behaviors of your code, and thus also describe the intention behind the tested system overall.

The way you declare assertions differs depending on the testing framework and assertion libraries you may use. It may be `assert.strictEqual()`, `expect().toEqual()`, or even a plain if/throw statement—that’s an assertion too!

Let’s refer to the assertions you write in your test as explicit assertions. You put them there for a reason, to serve a purpose, which is an explicit choice. There are, however, assertions that are still there but which you haven’t written yourself. Those are implicit assertions, and they can be tremendously useful when testing. Let’s talk about those today.


## Quiz time

We will start with a short quiz. Let me show you a few tests and you will tell me how many assertions you see in them (don’t scroll past the code snippets if you want a true quiz experience).

Ready? Go!

```
import { sum } from './sum.js'

it('adds two given numbers', () => {
  expect(sum(2, 5)).toBe(7)
})
```

Okay, it’s one assertion, you’ve earned yourself a point.
What about this one?

```
import { fetchUsername } from 'fetch-username.js'

it('fetches the user', async () => {
  const result = await fetchUsername('abc-123')
  expect(result.data).toBe('john.maverick')
  expect(result.errors).toBeNull()
})
```

Now it’s two assertions, you’re right. One for the `result.data` object, and the other to make sure there has been no errors (`result.errors` is null).

Finally, how many assertions do you spot in this test?

```
import { Greeting } from './greeting.js'

it('displays the greeting message', () => {
  render(<Greeting name="John" />)
  expect(screen.getByText('Hello, John!')).toBeVisible()
})
```

One, that the element by the given text is visible in the DOM. What was the challenge, again?

The thing is, **none of those answers are right**. There are much more to assertions than meets the eye, even in the most basic test.

Take a look at the first example I showcased:
```
import { sum } from './sum.js'

it('adds two given numbers', () => {
  expect(sum(2, 5)).toBe(7)
})
```

Visually, there’s only one `expect()` statement that describes one assertion. But in reality, there are more things that this tests validates outside of the reach of our eye:

- The `sum.js` module exists and has a sum export;
- The `sum()` is a function;
- The `sum()` can be called without throwing an error.

These assertions are not directly related to the intention that we are testing (the addition of numbers) but they still provide us with useful information about the testing setup and the tested code. We just happened not to write them ourselves.

It’s time we looked at what implicit assertions are and what value they bring to the tests.

## Implicit assertions

As the name suggests, *implicit* assertions are those implied in the test. You don’t author those yourself but rather have them derived from other test steps of even other assertions.

I guarantee you have already written implicit assertions in the past without realizing it. Here’s an example:

```
expect(data).toEqual({ id: 'abc-123' })
```

This `expect()` call makes sure the `data` variable equals to the `{ id: 'abc-123' }` object. In reality, it also checks that `data` is an object. That check, however, is implied by the value comparison since other primitives, like a string or an array, won’t equal to the expected object.

In other words, this single expect() statement can unfold into two:
```
expect(data).toBeTypeOf('object')
expect(data).toEqual({ id: 'abc-123' })
```

> Note that having both of these assertions is redundant. We will talk about the benefits of implicit assertions in a moment.

Implicit assertions can also manifest in less apparent ways. For example, rendering your React component before you even perform any actions on it already implies that the component can be rendered.

```
render(<Component />)
```

This is why you don’t follow each `render()` call with an `expect()` to check that the component’s HTML has indeed been written to the DOM. That’s *implied*. You proceed straight to the actions that interact with that HTML, and if the component failed to render for some reason, the `render()` function would probably let you know with an error, and none of the following actions would work anyway since they rely on the fact of the successful render.

## Benefits of implicit assertions

Understanding which assertions are implied and why is crucial to writing a good test.

### Skip redundant assertions
First of all, you can skip checking things that are implied by other assertions.

Coming back to the object equality example I mentioned earlier, having both `.toBeTypeOf()` and `.toEqual()` assertions is redundant, as the type check is implied by the equality check:
```
-expect(data).toBeTypeOf('object')
expect(data).toEqual({ id: 'abc-123' })
```

I’ve seen repetition like this in more test suites than I can count. Those may not be as obvious as you’d think either. Consider this:


```
-expect(someArray).toHaveLength(3)
expect(someArray).toEqual(['a', 'b', 'c'])
```

Once again, the length of the array is implied by the equality check. The length assertion here is not only redundant but also negatively affects the overall testing experience. If `someArray` becomes `[‘a’, 'b']` due to a bug, the first assertion to fail would be the length assertion, but on its own it doesn’t give you enough useful information to spot the issue.

> We write tests to fail, and good assertions help us understand the nature of those failures in an efficient, detailed way.

You’d want to see the diff between the expected and the actual arrays rather than a generic “expected to have length 3 but got 2” message, would you not?

Utilizing implicit assertions can help with keeping your test cases concise and focused on the thing that matters—the intention behind the tested system.

## Avoid insecure testing

One common mistake I see is the lack of confidence behind testing actions caused by the lack of understanding of implied assertions. I’m talking about cases like this:

```
render(<Component />)
-expect(screen.getByTestId('container')).toBeInTheDocument()

-expect(screen.getByRole('input', { name: 'email' })).toBeInTheDocument()
fireEvent.change(screen.getByRole('input', { name: 'email' }), {
  target: { value: 'kody@epicweb.dev' }
})
-expect(screen.getByRole('input', { name: 'email' })).toHaveValue('kody@epicweb.dev')
```

None of these `expect()` statements bring value to the test because all of them are already implied by the actions that follow before or after them:

- `render()` guarantees that the component has rendered;
- `fireEvent.change()` guarantees that the value of the given input is changed and that the input exists.

Overcautious assertions like these take away from the usefulness (and the main goal) of the `expect()` statements, which is to alert you when the intention behind the system is not met. Remember that the assertions you write must directly relate to the intention you are testing.

> This is one of the many situations when following the Golden Rule of Assertions comes in handy. I highly recommend you read about that rule!

It’s worth saying that custom actions and test utilities may require occasional checks to provide a nice developer experience around them. If that’s the case, consider using plain if/throw statements to clearly communicate when something goes wrong with the testing setup and not the tested system itself.


## Conclusion

I hope you have a better understanding about the explicit and implicit assertions now, and perhaps even have a test or two in mind to improve.

Before you go, I will leave you with an open question: is this a meaningful test?

```
render(<Greeting name="John" />)
```



