# Inverse Assertions
Recently, I’ve shared a [short tip](https://x.com/kettanaito/status/1838957793883095518) (There's no reason to ever use `sleep` in your tests. Ever. In fact, that's a red flag that you've got a flaky test. ✅ Don't wait for time, wait for state. Use utilities like `waitFor` instead.) on how you should never use `sleep` in tests, and what you should use instead. During that discussion, I got asked a great question:

> _But what if I want to make sure a certain side effect DOESN’T happen?_

That is a case for a negative assertion, and those can be quite tricky to get right. Take a look for yourself:

```js
import { render } from '@testing-library/react'

import userEvent from '@testing-library/user-event'

import { Dashboard } from '~/components/dashboard.jsx'

test('does not display notification when saving a post', async () => {

render(<Dashboard />)

await userEvent.click(saveButton)

expect(notification).not.toBeInTheDocument()

})

```

[The intention](https://www.epicweb.dev/the-true-purpose-of-testing) of this test is that the `notification` DOM element is not present in the document once we click the `saveButton`.

**But what if the notification appears some time later, after we clicked that button?**

Well, in that case, we will get a passing test where it should’ve failed—in other words, a false positive test. That won’t do.

If displaying the notification is a side effect of pressing the button, we need to somehow wait for that side effect to “complete” before running our assertion.

What should determine that side effect’s completion then? We’ve got just the `saveButton` and the `notification`, and we certainly don’t want to leak any implementation details or rely on `sleep` here…

The answer to this becomes clear if we consider a positive assertion first.

If you want to assert that the notification **is** displayed after a button click, you will be dealing with the same problem. In the case of a positive assertion, you can rely on `waitFor` to express that intention:

```js
import { render, waitFor } from '@testing-library/react'

import userEvent from '@testing-library/user-event'

import { Dashboard } from '~/components/dashboard.jsx'

const user = userEvent.setup()

test('displays notification when saving a post', async () => {

render(<Dashboard />)

await user.click(saveButton)

await waitFor(() => {

    expect(notification).toBeVisible()

})

})

```

Here, we are using the `waitFor` utility from React Testing Library to retry our assertion over and over until it resolves, and the notification element is visible on the page. If the element appears straight away, the test passes. If it takes some time for the element to appear, it passes as well!

**The trick here is that we are waiting for a particular state**. This is why I highly encourage you to prefer `waitFor` over `sleep` absolutely every time you are dealing with time-dependent side effects.

Our negative assertion is no different. We still want to await a particular state—the notification not appearing on the page.

But if we just wrap that assertion in `waitFor`, we will still be getting the same problem:

```js
import { render, waitFor } from '@testing-library/react'

import userEvent from '@testing-library/user-event'

import { Dashboard } from '~/components/dashboard.jsx'

const user = userEvent.setup()

test('does not display notification when saving a post', async () => {

render(<Dashboard />)

await user.click(saveButton)

await waitFor(() => {

    expect(notification).not.toBeInTheDocument()

})

})

```

Since `waitFor` will resolve as soon as the given callback resolves, this test will pass on the first attempt, even if the notification gets displayed a bit later. It’s the same false positive issue all over again!

**The solution to this is an inverse assertion.**

In other words, instead of asserting that the element is not in the document, assert that the element has appeared in the document, and then expect that assertion to fail.

```js
import { render, waitFor } from '@testing-library/react'

import userEvent from '@testing-library/user-event'

import { Dashboard } from '~/components/dashboard'

const user = userEvent.setup()

test('does not display notification when saving a post', async () => {

render(<Dashboard />)

await user.click(saveButton)

// Using `waitFor`, create a pending promise

// that resolves if the notification gets displayed,

// and rejects if it doesn't.

const notificationVisiblePromise = waitFor(() => {

    expect(notification).toBeVisible()

})

// Assert that the notification promise has, eventually, rejected.

await expect(notificationVisiblePromise).rejects.toThrow()

})

```

> Note that you can fine-tune any `waitFor` call by setting a custom `interval`, `timeout`, and other [options](https://testing-library.com/docs/dom-testing-library/api-async/#waitfor). This gives you more control over awaiting your side effects.

Similar to the positive test case, we are creating an eventual assertion on the notification getting displayed using the `waitFor` utility. But notice that we aren’t awaiting the promise returned from `waitFor` because we don’t want its rejection to fail our test (we do expect the notification to be missing!). Then, we add an assertion that the pending notification visibility promise rejects.

> **Instead of asserting that the expected state happens, you assert that the unexpected state doesn’t happen (thus, the _inverse_ assertion).**

By flipping the assertion around, we are able to wait for the potential notification render and then fail the test if that ever happens, covering the intention behind it.
