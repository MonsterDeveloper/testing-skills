# Why I Won't Use JSDOM

If you've been testing anything front-end related in the past 10 years, you've likely used JSDOM. In itself, JSDOM is not an environment or runtime. It's a polyfill library, containing implementations for a bunch of WHATWG standards, like HTML and DOM, for Node.js. Why Node.js? Well, because the library's goal is to execute browser-oriented code without paying the cost of spawning the actual browser, which means running that code in Node.js and relying on polyfills to breach the API gap.

Today, I would like to talk about the origins of browser-like environments, what hidden dangers they pose, and also what I am using instead in 2025. But first, a bit of history.

## The origins of JSDOM

JSDOM was created in 2010, when both JavaScript and its testing practices were in a rather different place. Back then, your main way to automate an actual browser was Selenium, which often yielded tests that were slow to run and too expensive to maintain. It is no surprise that something better had to come forth to make integration-level testing more accessible and efficient. JSDOM took that responsibility on its shoulders, carrying the weight over the next decade. Practicality and robustness of JSDOM resulted in it getting wide adoption to eventually become synonymous with integration testing on the web.

At its core, the library creates an enclosed execution context that promises sufficient standard compliance to run most of the code you mean for the browser.

Fun fact: the very first commit in the repository lists JSDOM as a "ServerJS JavaScript DOM".

I want to make it clear that JSDOM had place to be both conceptually and practically. It was an answer to the missing developer experience back in the day. It was a trade-off, and now that we have better tools available, its price is no longer worth the pay.

Let's talk about why.

## Chasing rabbits

The person who chases two rabbits catches neither. That much is true for JSDOM also, as it attempts to give you the evaluation context of a real browser while running in a Node.js process.

I find the following sentence to be a good summary of my issue with the library:

> JSDOM runs in Node.js, pretends to be a browser but ends up being neither.

Take a look at this example:

```js
import { JSDOM } from 'jsdom'

const dom = new JSDOM()
const clickEvent = new dom.window.Event('click')
```

Here, I'm manually constructing a click `Event` using the `dom.window.Event` class from JSDOM. When my test interacts with an element in the DOM, this is the kind of event I get.

It's rather obvious that this event isn't a browser event since there's no browser involved, to begin with. The odd part is that the `clickEvent` isn't a Node.js event either:

```js
console.log(clickEvent instanceof globalThis.Event)
// false
```

This preludes to a much bigger problem. While this click event will be handled just fine within JSDOM, things will fall apart rather quickly and loudly once this event is consumed by anything else but the library that created it.

```js
const target = new EventTarget()
target.dispatchEvent(clickEvent)
```

In the code above, I am creating an `EventTarget` instance in Node.js and providing it with my click event to be dispatched. This is a valid JavaScript code in the browser and in Node.js, but not in JSDOM:

```
node:internal/event_target:746
      throw new ERR_INVALID_ARG_TYPE('event', 'Event', event);
            ^

TypeError [ERR_INVALID_ARG_TYPE]: The "event" argument must be an instance of Event. Received an instance of Event
```

Running this code results in the error above. The error itself comes from Node.js, and it's put there to prevent invalid event instances from being dispatched on event targets. You shouldn't dispatch things that are not real events—Node.js is in the right here. But what I am doing this forbidden thing, in the first place?

Well, it's not that forbidden. In fact, it's rather common. I've spent an absurd amount of time last year debugging developer's issues with JSDOM once Mock Service Worker has introduced its standard-based API. Turned out, the historic addiction to polyfills was holding the ecosystem back more than I could've ever imagined. It was a combination of many things, but browser-like environments and their polyfill nature lied at its very core.

## Test environments

You are likely using JSDOM as a test environment. But you might remember me saying that it's not a test environment, it is a library. What you are actually using in your tests is an environment created on top of this library, e.g. `jest-environment-jsdom`.

Such environments are necessary to integrate the executing capabilities of JSDOM into a particular test runner. Unfortunately, this integration has took a bad turn.

### Patching globals

You shouldn't modify things you don't own. And by "you" I really mean yourself and, by extension, the tooling you introduce.

Unfortunately, browser-like environments are riddled with patched globals. As a matter of fact, those environments used to intentionally remove all Node.js global APIs, re-introducing their polyfilled counterparts from JSDOM on a cherry-pick basis. This led to a barrage of issues around missing APIs that are normally there, as well as code compatibility and standard-compliance.

Those environments have improved a bit since then, but, sadly, the issue sits too deep to be eradicated in any refactor. One of my favorite examples of this is the support of the Fetch API in JSDOM, which took years of constant polyfilling even after the API has landed in Node.js in 2017. You couldn't use global `fetch` in your tests until recently without getting the `fetch is not defined` error in Jest for a standard API that's been around for almost two decades.

This aspect is inevitable for the JSDOM architecture, imposing polyfills even for the standard APIs that are shared and available between the browser and Node.js, like `fetch`, `Event`, `MessageChannel`, etc. It makes things worse that most developers don't realize this. So when a global API like `structuredClone` starts throwing errors for the code that runs normally in the browser, they are rightfully at their wits' end.

### Dangerous assumptions

Browser-like environments try their best to behave as an actual browser but in so doing, they make a few rather dangerous assumptions. One of such assumptions is using the `browser` export condition for all the dependencies you import in your tests.

This is one of those details that seem insignificant until they become a show-stopper.

**Resolving dependencies as if they were imported in the actual browser while running in Node.js is a terrible idea**. For starters, JSDOM has never promised a full browser compatibility. I don't suppose anyone expects, say, the Navigator API to be fully functional in their integration tests. But applying the browser export condition means that it is completely allowed to import packages that depend on that browser API. More to that, that condition will force the browser bundle even if the dependency has a designated Node.js alternative. It punishes the ecosystem for trying to run Node.js code in Node.js.

These two issues alone should be enough for you to stop and reconsider your testing choices. But I saved the deal-breaker for last.

## The rule of test environments

> The closer your test environment resembles the actual environment where you code runs, the more value your tests will bring you.

Would you test Node.js applications in the browser? That sounds ridiculous. Well, it's much the same the other way around.

If you're creating UI components to be rendered in the browser, you should be testing them in the real browser. This is true today as much as it was fifteen years ago, but the tooling simply wasn't there. That's not the case anymore. We've got fantastic tools like Playwright and Vitest that make in-browser tests more performant and accessible than ever. It's time for you to use them.

## In-browser component testing

When I talk about testing your code in the browser, I don't mean an end-to-end test, mind you. Those are valid in their own right as they pursue different goals than integration, component-level tests.

My tool of choice for those component-level tests is Vitest Browser Mode. It's a new API in Vitest built to solve the same problem JSDOM set off to solve almost two decades ago. This time, using the actual browser and the flexible, performant, and familiar API of Vitest.

I know what you might be thinking. It's a big investment. Oh, great, another refactor. I get you. I mean, if you take a look at your existing component tests in JSDOM, this is just so clear and nice:

```jsx
test('renders the dashboard heading', async () => {
  render(<Dashboard />)

  await expect.element(
    page.getByRole('heading', { name: 'Dashboard' })
  ).toBeVisible()
})
```

Only, this isn't a test in JSDOM. This is a real test in Vitest Browser Mode that uses the same familiar, battle-proven patterns of React Testing Library but in the browser:

```jsx
import { page } from '@vitest/browser/context'
import { render } from 'vitest-browser-react'
import { Dashboard } from '#app/components/dashboard.js'

test('renders the dashboard heading', async () => {
  render(<Dashboard />)

  await expect.element(
    page.getByRole('heading', { name: 'Dashboard' })
  ).toBeVisible()
})
```

**This is huge!** The reliability of the actual browser and the comfort of the familiar testing API is the mixture we've been craving for years. It is finally here, and you can get your hands on it:

```bash
npm i -D vitest @vitest/browser
```

### Honorable mentions

I believe that the team at Playwright are doing a phenomenal job with their Component testing model, and you should certainly check it out as well. I am choosing Vitest because I find it more natural to use an integration-level test runner for integration tests, no matter their environment. That being said, I am sticking with Playwright for end-to-end tests as it's simply the most powerful browser automation option out there. It's not an either/or choice though as you can (and should) use Playwright as the browser provider in Vitest Browser Mode!

## What's next?

Next comes a paradigm shift that will finally help you get more value out of your integration tests. You can test the code you write for the browser in the actual browser, leaving the age of polyfills behind. As with any good tooling, once you do, you won't ever come back.

I will try to do my best to help you with that transition. Either it's through articles like this one, tutorials, or workshops—I am here to see the dawn of a new era of component testing, and I am excited for what the future may hold.
