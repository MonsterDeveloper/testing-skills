# Be S.M.A.R.T. About Flaky Tests

A test you cannot trust is a useless test. If I had to pick a single reason that undermines developers' trust in their tests that would be _flakiness_.

A flaky test is the one that returns unpredictable results without any apparent changes to the tested code. It's a test that can be green nine times out of ten only to randomly fail on the tenth attempt.

Because of that unpredictable nature, flaky tests are difficult to debug, which makes them difficult to fix. While they may indicate underlying problems with the tested software, they may also be caused by a number of other factors, like environment difference, asynchronicity, or be a result of a poorly written test, to begin with.

There is a number of approaches you can take in fixing a flaky test, and some will be more useful than others depending on your particular situation. But what do you do _while_ you are fixing it? Do you leave the test on? Do you skip it? What if finding the root cause takes a week? What if you can't figure out what's wrong?

Those question are as important as the fix itself.

Today, I would like to give you answers to those questions. I will do so by sharing with you a framework I use to approach and manage flaky tests.

## The S.M.A.R.T. framework

Given what risk flakiness posses to your trust in tests, it's only due that you approached any flaky test in a thoughtful, smart way. Literally, in a S.M.A.R.T. way.

"SMART" is an abbreviation that describes the sequence of steps you take to resolve a flaky test:

- (S)kip.
- (M)itigate.
- (A)ssess.
- (R)ewrite.
- (T)hrow away.

At its core, this is a debugging runbook for you, a plan of action whenever a perfectly fine test decides to fail on CI.

Let's go through each of those steps in order at which you will apply them.

### Skip

**The very first thing you do when encountered a flaky test is you skip it**. That same minute of the same day. Commit and push to `main`. No exceptions. Don't delete it, skip it.

Even a single flaky test brings the reliability of your entire test suite to zero. And yes, that includes the tests that are working fine. Just like you thought the flaky test was before it revealed its true colors.

Whenever you see a passed test run, you must have a rock-solid confidence that it indicates your software is functioning as intended. Flaky tests rob you of that confidence and, thus, must be taken out from your test suite entirely until they are either fixed or deleted.

> One of the common pushbacks to skipping a flaky test is "But it's still useful when it does pass!". If any failed test run makes you wonder even for a second whether that's an issue with your code or just flakiness, you lose any usefulness from that entire test suite.
> Nobody wants to see their work thrown away, I get it. Perhaps you or your colleagues have spent days putting that test together. And that's why I'm advising you to _skip_ the test, not delete it. We are barricading the crime scene with a tape, not wiping it squeaky-clean!

### Mitigate

Now that the flaky test is skipped, it's time you put on your investigator hat and found out as much information about the test at question as possible.

The goal of this step is not to fix the test (although, by all means do so if you already know the culprit) but to understand the context around it.

Here are some of the questions you should answer, to the best of your ability, during this step:

- What contributes to flakiness?
- Is this test flaky everywhere on only in certain environments?
- Was there any similar precedent in the past? How was it resolved?
- Has there been any changes that might have affected this test or the code it's testing?
- What is the last step reached in test? What causes the failure? Do you see any hints as to what might be the root cause?

It is crucial that you **write down** all the discoveries you make and the related information you find. At the end of this step, you should have a document (or a ticket) accumulating your findings.

> Never underestimate the nature and extent of factors that can contribute to flakiness. I once had a flaky test because the machine running it ran out of memory! Don't be afraid to look around and ask questions.

The point of this step is to perform early issue analysis without dedicating significant time to any particular assumption or solution. You are able to complete the mitigation step within the same day of discovering the flaky test. But you may not be the same person tasked with fixing it. This is where research and knowledge sharing is crucial, especially if working in larger teams.

### Assess

The first two steps have already restored trust in your test suite and provided you with the additional information surrounding the test. Anybody on the team can take it from here.

The goal of the next step is to **assess what it would take to fix this test**. This involves putting up proposals as to what may be the root cause, drafting possible solutions, and making time estimates for them. This is also a great time to bring this topic up with the team since everyone would be able to contribute to the discussion having objective results of the mitigation step!

**Make fixing the flaky test your official task**. This is not overtime charity work. There is an issue that puts the quality of your software in question, and it must be solved. Plan that work into your next sprint and see it through.

Chances are, you will discover the problem, fix it, and reinstate the test back into the test suite. There's also a chance that the issue would require more time than you thought, or lead to new discoveries or even more complex problems. That is fine also. Document your findings, refine your approach, plan your next work. Rinse and repeat.

If you couldn't identify the issue behind the flakiness, proceed to the next step.

### Rewrite

At this point, it's time to employ one of the most useful techniques in debugging software—isolated reproduction.

Try rewriting your flaky test in an empty repo. Cherry-pick only the needed parts from your application and substitute or remove the irrelevant ones (be careful in this judgement!). This is not going to be easy. You will soon discover that you are pulling on a flower and the whole root system comes out, wide and tangled and nasty. Take it on step-by-step. From configurations to components to dependencies.

> Keep in mind to run the isolated test in the same environment where the flaky test manifests (locally or on CI).

There is a solid chance you will spot the culprit by assembling that same test from ground-up. It may be a bug in an outdated dependency, a wrong testing setup, incorrect configuration for your testing framework... Many different things can contribute to flakiness, and not seldom at the same time.

But if the test just refuses to yield, there's one last step you can try.

### Throw it away

The flaky test stops bringing you value the moment you identify it as flaky. If you've given it your best but nothing helped with resolving it, don't feel bad to delete the test altogether. At this point, remaining a part of your code base only makes it more confusing and harmful.

That does not mean forgetting about it though. That test was written for a reason. Try writing the same test from scratch, verifying the same intention. Perhaps _that_ will be the "aha" moment for you.

## Conclusion

Flakiness is not an easy problem to solve, and will require a good deal of patience, skill, and perseverance. I hope the framework I shared with you today will come in handy the next time you have to deal with a flaky test.
