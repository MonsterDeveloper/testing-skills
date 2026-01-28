# Anatomy of a Test

As humans, we are bound to chase understanding, and so we are also bound to seek structure and logic in things. Both give us a sense of order that, in turn, inspires and encourages us that everything can be learned, explained, and understood.

In my first year as a medical student, I had to memorize about 30 pages of human anatomy for every class only to be asked about a thing on page 31 the day after. Human anatomy has to be one of the most challenging things I’ve ever learned in my life (and I’ve learned JavaScript so that’s something). But it was crucial to understanding the human body. The bones, the muscles that attach to them, the ligaments to make them move, the nerves to control the movement, and the organs to help us function. All pieces fit in together because, in the end…

> The complex is inevitably built upon the simple.

Testing your code is no different. It can be challenging and it can certainly be complex, and yet it always builds on top of much simpler basics.

Today, we will learn about the anatomy of a test, and for that, we will conduct an experiment!


## Dissecting tests

Putting our lab coats on 🧑‍⚕️, let’s examine different tests from various programming languages and see if we can spot something in common. To make things a bit easier on our eyes, we will write the same unit test in JavaScript, Go, Java, and Rust.

```
// sum.test.js
it('adds two given numbers', () => {
  const result = sum(2, 5)
  expect(result).toBe(7)
})
```

```
// sum_test.go
func Test_sum(t *testing.T) {
	t.Run("adds two given numbers", func() {
    var result = sum(2, 5)
		assert.Equal(t, 7, result)
	})
}
```

```
// SumTest.java
public class SumTest {
	public void addsTwoGivenNumbers() {
    int result = sum(2, 5)
		assertEquals(result, 7);
  }
}
```

```
// sum_test.rs
mod tests {
	#[test]
	fn adds_two_given_numbers() {
		let result = sum(2, 5)
		assert_eq!(result, 7);
	}
}
```

Why, aren’t those quite alike!

First, naturally, we will have the actual tested code invoked one way or another (i.e `sum(2, 5)`). Then, there will always be an assertion that compares the actual and the expected result of the sum operation (i.e. `expect`, `assert.Equal`, `assert_eq`).

But looking even higher than that, each test has a name, and the test function, and a particular *arrangement* of classes, functions, or modules that make it a test. It’s almost as if there’s an ephemeral structure to them that repeats and stays true no matter the language…

## Test structure

Any automated test, at any level, in any programming language consists of three main steps:

1. Setup.
2. Action.
3. Assertion.

Each step is important because each step has its own distinct purpose. We will look more closely at each of those steps below.

## Setup

The **setup** step is responsible for preparing everything around your tested code so the code would actually run. This makes it one of the most important steps of your test as it concerns itself with everything from the test environment and test definition to putting your code into a desired initial state, handling any unwanted side effects, and establishing the testing boundaries.

> I often like to explain the setup step as a “box”. You test your code by putting it in that box so you better make sure it has everything the code needs to work properly inside of it.

That being said, you don’t need the setup phase for every test you write.

In fact, if you are using a testing framework, it will do a lot of the setup for you. For example, you don’t need to prepare anything to call a simple `sum()` function in a test. Much of that is determined by the function itself since it’s pure, has no dependencies, and introduces no side effects.

Once you move to testing more complex code, the setup step becomes indispensable. It also becomes quite complex itself. I firmly believe that it is the setup step that must mitigate the most complexity carried into the test from your code.

A few examples of the things you may see during the setup:

- Run you application or render an isolated component;
- Mock network requests;
- Use dependency injection;
- Initiate test databases;
- Wrap the tested code in providers.

## Action

The **action** step is about performing actions on the tested code.

```
sum(2, 5)
```
The actions you perform come from the expectations you have, and concern themselves with bringing your code to the expected state. In the case of the `sum()` function, the action is to call it with two numbers, and get the sum of them in return.

> In stateful systems, actions always make your system transition between states.

Not all the code you test will be as simple as `sum()`. Often, you would have to perform multiple different actions before making the assertion. It is important you don’t stray from the intended behavior you are testing and stay true to the purpose of tests.

Modeling your actions from the right perspective is equally as crucial. I believe Kent C. Dodds said it best:

> The more your tests resemble the way your software is used, the more confidence they can give you.

It’s important to remember that the test actions you write aren’t yours, they are the user’s. They belong to the person (or another machine!) who will be using your code with a particular intention in mind. So the more you model your actions to resemble those of the user, the better your test will be overall.

Here are some of the actions you may make in a test:

- Call functions and methods;
- Navigate to a particular page;
- Interact with DOM (e.g. click on things);
- Dispatch network requests.

## Assertion

The **assertion** step is where you compare the actual state of your system with the expected state.

```
expect(actual).toBe(expected)
```

Once again, it’s quite straightforward with the `sum()` function: when given 2 and 5, you expect the result to be 7. If it’s anything but 7, the test must throw because the intention behind the `sum()` function hasn’t been met.

> **Assertions are the most important part of any test.**

I honestly don’t know how to stress this enough. Assertions are the heart and soul of what the test is about because they are written around the expectations toward the code’s behavior (this is why you should generally have at least one assertion in a test).

Much like the actions, assertions also derive from the intention behind the tested code. But it may not always be clear what and to which extent to assert. When in doubt, remember the Golden Rule of Assertions, and the fact that **assertions are only useful when they throw**, so you have to make them throw reliably and for a good reason.

Assertions can be literally anything, but here’s a few examples:

- The return data of a function is what you expect;
- A certain UI element is visible (or no longer visible) on the page;
- The browser has navigated to the next page.

## Conclusion

The three-step test structure is not something new. I’ve read about it a lot but I always found it strangely missing when talking about anything but unit testing. In practice, tests of any level follow the same structure since all tests need to prepare the environment and the system, perform some actions, and validate the result. Without much simplification, that is the essence of any test, and with a solid test structure, it makes it a tad easier to grasp.
