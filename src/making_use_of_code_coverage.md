# Making Use of Code Coverage

Code coverage is a peculiar metric. It’s spawned a lot of debate and the views on it differ quite significantly in the JavaScript community. On its own, it’s a tool. Like any tool, it can be useful but it can also be misused.

Let’s talk about code coverage and how you should (or shouldn’t) be using it when testing modern web applications.

## What is code coverage?

*Code coverage* is a metric that measures how much of the tested system is run during the test.

Imagine you have a simple `sum()` function that adds any two given numbers:

```
// sum.js
export function sum(a: number, b: number): number {
	return a + b
}
```

And then a test case for this function covering its intention:

```
// sum.test.js
import { sum } from './sum.js'

it('adds two given numbers', () => {
	expect(sum(2, 5)).toBe(7)
})
```

The code coverage of this test is 100% because the entirety of the `sum()` function has been executed during the test run (it quite literally has only one line of code). In other words, this test covers everything that the `sum()` function does.

Let’s take a look at another example.

The `getParity()` function accepts any number and returns either an "odd" or an "even" string depending on whether the given number if odd or even, respectively.

```
// get-parity.js
export function getParity(input: number): 'odd' | 'even' {
  return input % 2 === 0 ? 'even' : 'odd'
}
```

```
// get-parity.test.ts
import { getParity } from './get-parity.js'

it('returns "even" given an even number', () => {
	expect(getParity(2)).toBe('even')
})
```

Despite the `getParity()` function only having one line of code, it has two logical branches:

- The "even" string is returned if the `input` is even;
- The "odd" string is returned if the `input` is odd.

Since our test only covers the first logical branch, the code coverage for the `getParity()` function is 50% (we are missing tests for the "odd" behavior).

## Code coverage criteria

There are a number of different criteria that determine code coverage. Most used criteria in practice are:

1. **Statement coverage**, measures if every statement of the program has been executed;
2. **Branch coverage**, measures if every branch of the behavior has been executed;
3. **Modified condition/decision coverage**, measures if every possible input and output of the program has been achieved.

You can often find these and additional code coverage criteria when using tools like Istanbul:

```
----------------|---------|----------|---------|---------|-------------------
File            | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
----------------|---------|----------|---------|---------|-------------------
All files       |     100 |       50 |     100 |     100 |                   
 get-parity.ts  |     100 |       50 |     100 |     100 | 2                 
----------------|---------|----------|---------|---------|-------------------
```

## Code coverage in practice

Taking its roots from the safety-critical systems, like avionics and spacecraft engineering, code coverage provides a ton of value in that context. Building and testing applications where the cost of a bug is measured in human lives doesn’t leave much room for debate. The software of that caliber has to be analyzed and verified to the last bit of information, to the farthermost logical branch in the code.

The practical application of code coverage for the rest of us varies. It’s essential to understand that code coverage is a tool. It can be useful and it can also be harmful. It’s not a panacea and it mustn’t drive your entire testing effort.

Code coverage can be a valuable instrument in detecting the logic you missed when testing, as well as identifying dead code and giving you more context on what exactly gets tested in your app.

For library authors, code coverage is a great way to ensure all the public APIs their library is shipping is covered with tests. For those working on product development, code coverage can still be useful but has to be approached with a much higher degree of caution. In my practice, adopting code coverage in companies degrades the development and testing experience more often than it improves it. That is in no way a fault of the tool but rather of the hands wielding it.

At the end of the day, any tool is only as good as your understanding of it. That understanding, inevitably, involves knowing the shortcomings and caveats of the tool. Let’s talk about those.

## Caveats

### Implementation-driven

Code coverage is by design derived from the implementation of the system. It can be an easy way for the implementation detail testing to sneak its way into your test suite, harming its reliability and making you go after numbers instead of focusing your tests on what really matters—the intention behind the system.

The requirements toward modern software have grown rather complex. It’s not seldom you will find yourself writing multiple lines of code and performing a number of logical operations to achieve a single intention.

And it’s crucial to put that intention first when testing your code. Following code coverage will never encourage you to do that. In fact, it will encourage the opposite. It’s a metric on machines by machines. But you aren’t writing tests for machines, you are writing them for yourself! Ultimately, you are the only one making calls in your testing strategy, and you are the judge of the test’s usefulness, whether you want that or not.

Because here’s the truth:

> The only thing code coverage encourages you is **to write tests that run 100% of your code**.

This is an extremely slippery direction to follow. Not only will this waste your time but it will result in absolutely useless tests that bring no value (but do bring higher coverage!).

> Kent C. Dodds makes an interesting argument about the 100% code coverage in his Common Testing Mistakes article. I highly recommend you read it!

Good testing practices always derive from the intention-driven testing. By design, code coverage cannot give you any details or metrics related to that intention. It’s incapable of understanding it.

## Limitations

Code coverage is based on static code analysis. Machines are great at analyzing code but not that great at understanding the intention behind it. Our intention lives in the tests, not the code, so code coverage cannot predict what we are trying to achieve (neither should it, really).

> Imagine writing software as writing a novel. Your intention is the message you put into the story. You then use words (the implementation, the code) to present that message to the reader. But the words alone aren’t the message. It may be hidden between the lines, only for the reader to uncover.

In practice, this simply means that code coverage isn’t always a reliable metric. It can produce both incomplete and straight incorrect results. Let’s look at the example.

Here’s a function called `isEven()` that accepts a number and returns a boolean indicating whether that number is an even number. In other words, given 1 it will return false , and given 2 it returns true, and so forth. That’s the intention, and here’s how we achieve it:

```
// is-even.ts
export function isEven(input: number): boolean {
	return input % 2 === 0
}
```

```
// is-even.test.ts
import { isEven } from './is-even.js'

it('returns true given an even number', () => {
	expect(isEven(2)).toBe(true)
})
```

Since the function has just 1 statement (`input % 2 === 0`), and that statement gets executed in the test, we will see 100% code coverage (try it yourself it you don’t believe me). But, wait, aren’t we missing something? What guarantees do we have that isEven() would return false given an odd number?

We don’t. But we do have a high number, the highest number, in fact. This is where code coverage can get misleading and rather dangerous. If the same metric can result both in testing the things you shouldn’t and forgetting to test what you should, one must treat it with extreme caution.

Which brings me to the next point.

## Misusage
It’s not unknown in business to base decisions around numbers. Code coverage is one of those numbers that I’ve seen managers and developers use to reason and asses various areas of product development. In fact, it’s an easy metric to get false assurance from, and I suspect that being a major factor why some teams feel tempted by it. If the percentage rises, then the team must be shipping good code, and if it falls, then the things aren’t going well.

Unfortunately, it’s not that simple. I understand the appeal of using a single metric—a magic number—to guide you through the dark forest which is automated testing. But ask yourself this: Where do I want to get?

Because if your goal is chasing higher numbers, then, by all means, you will get those. The problem is that testing isn’t about numbers. We write tests to help us ship more reliable software faster. We get there by writing useful tests. The lines of code executed during a test don’t define the test’s usefulness. Code coverage may be helpful but it mustn’t be a single thing that decides your testing direction.

Let me tell you a story. I worked in a team that relied heavily on code coverage. One day, I got a bug report that was causing customers trouble. I knew where the issue was, and after looking into it, I opened a pull request with a fix. Since this unexpected behavior managed to slip into production, I added an automated test for it to prevent regressions. With the changes pushed, I patiently awaited the code review.

While that was happening, an automated code coverage report came through. Turned out, my pull request decreased the code coverage of the product, which prevented me from merging it. “Nice!” I thought, since automations like this can be handy to spot mistakes. So I followed the report to find myself in the part of the product that neither have I changed nor that had anything to do with the bug. What’s even more surprising, the coverage drop was 0.0001%!

That failure did help! It helped us spot a misconfigured tool, which we then fixed. We owned the code end-to-end, that wasn’t a big deal.

But how many teams are out there who don’t own these automations? They can be set up by senior engineers, SREs, and CTOs who didn't give a proper thought into how blocking software delivery behind a code coverage threshold may backfire. How many developers are there who weren’t allowed to merge a legitimate bug fix, and were forced to chase contrived numbers?

I know there are far too many. That is why you shouldn’t follow any single metric blindly (and should also regularly review your automations to make sure they actually help you—that’s their only purpose).

> Code coverage is a metric much like the line of code added and deleted in a pull request. Those numbers are useful, they give a higher picture. But you would never decline a meaningful change because it adds more LOC than it’s “supposed to”, would you?

And the opposite is true also. Code coverage reporting can be fooled and inflated, giving a high number without high returns. In my experience, I’ve seen it misused far more times than it ever provided actual value.

## When I would use code coverage

With all this in mind, when and how would I use code coverage today?

Truth be told, I wouldn’t. Unless I’m working on (human) safety-critical systems, I find the cost of introducing code coverage to significantly outweigh the gain.

> I don’t want to become dependent on the coverage increase giving me a dopamine boost. I want to get that boost after writing mindful tests and getting the confidence from those tests, not the metrics.

I may use code coverage as a tool to help me audit the software I’m building and help find problematic areas in it on a run-and-see basis. I would then use that report and devise a testing strategy that makes sense in that particular case. I may revisit that analysis over time but I would not integrate code coverage automation into any of my projects, let alone allow it to drive my decisions when testing.

## Conclusion
Code coverage is a metric. It can be useful but its applications are niche enough to promote misusage, turning it into a harmful tool in the hands of those who chose to put too much trust in it.

On its own, code coverage must never drive your testing effort. If it happened to do so now, I highly encourage you to move away from it and spend the time you’d invest in pleasing the coverage percentage on learning what automated testing actually is and how to make the most out of it.



