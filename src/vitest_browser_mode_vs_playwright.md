# Vitest Browser Mode vs Playwright

Now that the Browser Mode has gone stable in Vitest 4.0, it's time we talked. I've been teaching developers how to ditch browser-like environments and start benefitting from testing your components in the real browser for about a year now, and anytime I talk about it, I get the same question:

> What is the difference between Vitest Browser Mode and Playwright?

It is a great question to ask. On the surface, these tools are extremely similar:

- They both use browser automation
- They both have similar APIs (e.g. `page`)
- They both can test components (see Playwright Component Testing)

But these superficial similarities end as soon as you take a closer look at what these tools were designed to solve and how they work under the hood.

Today, let's unpack the difference between Vitest Browser Mode and Playwright, what makes them unique, and when you should use each.

## Purpose

Vitest Browser Mode provides a component-level testing in the browser. You render components in isolation, interact with them, tap into different behaviors they have on the actual page in the actual browser. This is a long-awaited marriage of environmental integrity, ergonomics, and performance that finally makes tools like JSDOM and HappyDOM obsolete. No more mocking of `location`, `navigator`, or `matchMedia`. No more fighting your test setup for the sake of brittle tests in made-up environments. Your components run where your users interact with them—in the real browser.

Playwright, on the other hand, is primarily an end-to-end testing tool. Instead of running a component, the actionable unit in a Playwright test is a page. As a result, you write different tests, the ones that focus on your application as a whole and what complete interactions your users can do with it as opposed to individual components.

Playwright Component Testing sits much closer to Vitest Browser Mode. It also focuses on component-level tests, runs your code in the actual browser, and brings the related benefits. But it works completely differently.

## Test environment

The Browser Mode is a feature of Vitest, which is a test runner on top of Vite.

```
Vite → Vitest → Browser Mode
```

> Much like Vitest itself, you don't have to use Vite to enjoy the Browser Mode.

Vite itself is a build tool, and Vitest uses it to treat your test suite as a mini app it compiles and serves in the browser. Here's the most important distinction: **Vitest Browser Mode runs your `component.test.tsx` in the browser**. This means you can write them the same way you write your app, regardless if you're using Vite—render JSX, import CSS, access `window` and `document`, etc.

This also means that if you've been accustomed to JSDOM, where your tests were a mixture of browser API polyfills running in a Node.js process, you might find yourself confused when it comes to Node.js utilities that used to accompany your tests. I'm talking about HTTP servers, file system access, any `node:*` built-in module, or third-party tools that depend on the above. Naturally, running those in your browser test will throw! Vitest presents an answer to this in a form of keeping your Node.js test setup in—surprise, surprise—Node.js, an exposing it granularly via its Commands API. This is an ergonomic solution since Vitest already spawns a Node.js server to orchestrate your test run, even if you're running browser tests.

Playwright Component Tests are built on top of Playwright, inheriting its architecture. **Playwright runs your `component.test.tsx` in Node.js**, relying on a message channel to let your test and the browser communicate. This architecture works great for full browser automation but falls short when it comes to component tests. It changes how your components are rendered in tests.

## Component rendering

Vitest Browser Mode is fully framework-agnostic and encourages you render your tested components the same way your application renders them. For example:

```jsx
import { test, expect } from 'vitest'
import { page } from 'vitest/browser'
import { render } from 'react-dom'
import { Greeting } from './greeting'

test('displays the greeting message', async () => {
  render(<Component />)

  await element.expect(page.getByText('Hi, Kody!')).toBeVisible()
})
```

Vitest ships a few convenience packages, like the ones for React and Vue, aiming to enhance the rendering experience by, for example, exposing you a direct reference to the rendered component. Nothing changes in the way your component is rendered. I am a big fan of the team's approach to rendering because it makes implementing your own `render()` functions for any framework a breeze.

Due to its inherent architecture, Playwright cannot tap into the `render()` from `react-dom` to render your React component directly in your test. Because the place where you render your component (the Node.js test) and where it is supposed to render (the browser page) live in two separate universes. And so Playwright fills in the gap between those universes by serializing the JSX from the test and sending it to the browser to faithfully reconstruct the component tree on the other end.

## Browser automation

Vitest Browser Mode does not ship its own browser automation. Instead, it introduces a concept of a browser automation provider and lets you choose between the supported providers (presently, Playwright and WebdriverIO). The best part is how the said browser automation is abstracted from you:

```js
// greeting.test.tsx
import { test, expect } from 'vitest'
import { page } from 'vitest/browser'

test('displays a greeting message', async () => {
  await element.expect(page.getByText('Hi, Kody!')).toBeVisible()
})
```

If I were to ask you which browser automation this component test uses, what would you say? Is this Playwright because it's `page`? Is this WebdriverIO? Is this something else?

**The truth is, it doesn't matter**. Vitest packs the implementation detail of your test behind the `page` API that remains the same no matter what browser automation provider you're using. You can choose one or another, even on a test suite basis, and your tests won't change a bit.

> That being said, the `page` API Vitest exposes is intentionally limited and does not cover all the features of its namesake from Playwright.

`@playwright/test` (the test framework) uses `playwright` (the library) for browser automation. It is the best in class and that is why I highly recommend you used Playwright for end-to-end tests and as your browser provider for component tests with Vitest Browser Mode.

## Summary

In a somewhat crude and subjective manner, I can attempt to summarize this comparison into a few short sentences.

- Vitest Browser Mode approaches a component test as an extended integration test
- Playwright approaches a component test as a limited end-to-end test

In the end, a component test is an integration test by design, and so it is no surprise that I pesonally prefer this perspective and its particular implmentation by the Vitest team. Vitest is an outstanding testing framework and I'm happy to see it promoting in-browser component tests with the first-class Browser Mode API.

## Which should you use?

My personal recommendation for great tests across the board goes like this:

- Use Vitest for unit and integration tests in Node.js
- Use Vitest Browser Mode for testing your components in the real browser
- Use Playwright (`@playwright/test`) for end-to-end testing your apps
- Use Playwright (`playwright`) for general purpose browser automation
