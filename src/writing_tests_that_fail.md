# Writing Tests That Fail

You write tests for them to fail.

As much as I love the sight of a passing test, I've grown to appreciate those that fail more. A "green" test may still be faulty (those false positive cases), giving you a false feeling of assurance, but a failing test is always good news.

The main value of an automated test is for it to fail at the right time and with the right level of detail. When that happens, tests are a bliss. Something has been broken, and now you know precisely what and, sometimes, even precisely why. A test will fail at the right time if you've modeled it around the intention behind the system, and with the right detail if you've crafted your assertions with thought. Those are the two main cheat codes in the testing game.

"But what of the false positives? Flakes? Aren't those equally bad?" Every failure indicates a problem, and every problem surfaced is a victory. A failing test is usually your indicator that your application doesn't behave as expected. I admit, sometimes tests may fail for other reasons, but they do fail, letting those reasons known. Perhaps the test was written incorrectly, doesn't establish clear test boundaries, has inefficient setup, or simply didn't do what you thought it did. As long as you are aware of that, you can trace it down and fix the problem. There's hardly anything worse in software engineering that the problems that don't surface in any way.

What I love to do is make sure that the tests I write actually fail when the behaviors they are testing go awry. Alternatively, I can approach the upcoming implementation test-first, and then all my tests will be gradually shifting from red to green as I work my way through the code. It is in knowing that a test can fail but doesn't where one finds beauty in testing.

This mindset goes way beyond cherishing test failures. In fact, one of the most useful questions you can ask yourself when testing (and even developing) is:

> But when will this fail?

I've had a number of testing-related questions answered with this single question (and yes, answering questions with questions can be rather effective!). It made people think and evaluate the factors that influence the outcome of the test. Because it's your job as an engineer to ensure those factors are strictly related to the intention you are testing, and nothing else.

For example, performing actual HTTP requests in tests means that those tests will fail if the real server is down or responds with an unexpected data. Those behaviors likely lie outside of your test's responsibilities, and as such, must be excluded from your tests via tools like mocking.

In other cases, a test may fail when ran alongside other tests but never in isolation, which often suggests a shared state issue that you have to resolve. And there are also situations when a test won't fail at all, making itself both useless and harmful simultaneously.

Failing tests can be painful. But just as pain in our bodies, it's indicative of a larger problem, one that shouldn't be ignored. So the next time you see that dreaded failed test run, take a breath, and be happy in knowing your tests serve their purpose. They indicate a problem. And whether it's a problem with your application or the tests themselves, one thing is certain—the thing you are building will become better once you resolve this failure.

Stay healthy and write good tests!
