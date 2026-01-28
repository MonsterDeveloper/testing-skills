# .toBeVisible() or .toBeInTheDocument()?

If you've been testing your components with Testing Library, you are no stranger to these two matchers:

- `.toBeVisible()`
- `.toBeInTheDocument()`

While they may seem similar, if not identical, there's actually an important difference between the two. Let's explore them in more detail and also learn when you would want to use one over another.

## Visibility

```js
expect(greetingText).toBeVisible()
```

The `.toBeVisible()` matcher, as the name implies, focuses on the element's visibility. In practice, this single matcher introduces a bunch of implicit assertions that make sure that your tested element is:

- Present in the DOM
- Doesn't have the `display` CSS property set to `none`
- Doesn't have the `opacity` CSS property set to `0`
- Doesn't have the `visibility` CSS property set to `hidden`/`collapse`
- Doesn't have the `aria-hidden` attribute set to `true`
- All of its parent elements also satisfy the criteria above

This matcher checks visibility from the user's perspective, which means the element is not only present in the DOM, but is also visible and interactive. **Due to being accessibility-first, this matcher is your go-to choice when checking element's presence and visibility**. Respectively, there's little reason to use it to test that the element is absent from the page (absence implies that all of the accessibility checks above are false).

**Do NOT use `.toBeVisible()` if you need to assert element's absence.**

**Use `.toBeVisible()` if you need to check that the element is present and accessible.**

```js
await expect.element(greetingText).toBeVisible()
```

This assertion uses Vitest's auto-retry mechanism built into `expect.element`! This is just one of many quality-of-life improvements that Vitest Browser Mode provides.

## Presence

```js
expect(greetingText).toBeInTheDocument()
```

The `.toBeInTheDocument()` matcher, on the other hand, only checks if the tested element is present in the document, and nothing else. **This matcher is not accessibility-friendly** and as such must never be used to test element's presence. It is a great choice for testing things that should be missing on the page though as it yields faster results.

**Do NOT use `.toBeInTheDocument()` if you need to assert element's presence.**

**Use `.toBeInTheDocument()` if you need to assert element's absence.**

```js
await expect.element(errorNotification).not.toBeInTheDocument()
```
