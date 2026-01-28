# The True Purpose of Testing

Let me ask you a question: What are the automated tests for?

Whether you’ve written some tests in the past or not, you’ve likely heard that tests can be useful. And I know you’ve also heard stories about tests that are more of a chore and nuisance than any help. Frankly, I believe both parties are right! But to understand why, I will first let you in on a little secret.

**No test is inherently useful just because it exists. A test becomes useful when it fulfills its purpose.**

So, what is that purpose exactly?

As the name implies, automated tests are here to help us automate something. We take a particular state of our application, perform some actions, and check the resulting system to be what we expect. That’s what any test does, and yet some tests are more useful than others. There is precisely one thing that defines and controls that difference.

## The intention

Just as every piece of code exists for a reason, every test exists to verify the intention behind that code.

A paragraph is rendered when the user lands on a page, a request is made when they click a button, the right data is returned when a function is called—there is no count to the things we want computers to do. We then go and use a programming language the computer understands to implement that intention.

Sometimes, the intention and the implementation are the same. Take a look at this simple function that adds two numbers:
```
function sum(a, b) {
  return a + b
}
```

The intention is for the `add` function to add any two given numbers `a` and `b`. Its implementation quite literally does just that: `a + b`.

In the real world, however, our intentions are seldom as simple as that. We often write hundreds of lines of code to achieve one particular thing. We end up in the situation where the system’s implementation doesn’t directly correspond to the intention behind it. This is when the distinction between the intention and the implementation becomes crucial in the context of testing.

## The purpose of tests

A test’s true purpose is tightly connected to the intention behind each test case, and can be put together as such:

> You write tests to validate the intention behind the system.

It is only when a test validates the intention that it becomes useful.

I’ve seen projects with hundreds, sometimes thousands of automated tests that contributed to nothing. They often broke across pull requests. They were hard to read and painful to navigate. They could’ve been removed tomorrow, and nobody would notice. With time, I came to realize what made them that way.

They had nothing to do with the intention of the code they were attempting to test. Instead of validating what the code did (the intention), they went in length to focus on how it did it (the implementation).

“Don’t test implementation details,” is a common advice you hear in automated testing. Hearing doesn’t mean understanding, and understanding doesn’t mean applying. We spend so much time writing code that we often become blind to the line between the intention and the implementation. In the end, we also intend every programmatic choice we make so why not test that?

You can only unlock the value of automated testing once you start treating implementation details as the means to an end, and focus on the larger intention at hand. Your users don’t know how the features of your app work, and neither should your tests. What the users do care about, is that whichever promises your app makes, whichever intentions it allows them to achieve, it does without fault.

Switching to this mental model also does wonders whenever you are unsure what to test. Suddenly, you can approach any system at any scale. When in doubt, test the intention. Always test the intention.

That’s what the automated tests are for.