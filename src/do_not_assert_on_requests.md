# Do NOT Assert on Requests (Do This Instead)

Throughout the years of building Mock Service Worker, there's been one group of questions that seems to always confuse developers:

- How do I make sure that my app performed this request with that payload?
- How can I be sure my app dispatched the request at all?
- How do I know that the server sent the correct response back?

All of these are valid questions to ask yourself when testing your UIs but asserting directly on API requests is never the answer to any of them.

Let's talk about what is.

## Wants vs Needs

There's a useful rule in writing that character growth lies in the conflict between what they want and what they actually need. I cannot hope for a better metaphor to help explain request assertions and make you grow as an engineer in the process.

What you want is to make sure the requests you've added into your app behave as you intended. After all, they are there for a reason, shaped the way they are, so something has to verify that, right?

What you need, however, is to test the end, not the means. Making a request is rarely the end goal of your software. It is a technical translation of your actual intent into code, and being the one to write it often blinds you to see the difference between the two.

"My app is making a `POST /cart` request." Great, but **why?** "So the server would update the cart's state." Why? "So that when the user adds a product to the cart, it actually gets added."

**Bingo.**

You've escaped the forest of implementation details and now stand on the path to meaningful testing, which is always user-driven. It doesn't matter how your app works, only what it does for the user. Consequently, your expectations in tests must be your user's expectations upon interacting with your software. And the thing is, your users don't expect a `POST /cart` request. They expect clicking on the "Add to cart" button and seeing the product added. That is the only thing that matters to them. That should be the only thing that matters to you, too.

> Instead of asserting on the requests and responses, assert on the state of the user-facing UI that depends on the network.

## Naive approach

Equipped with that knowledge and realizing that the actual request handling isn't your app's concern, you reach out to API mocking tools to draw a boundary in your system under test. It might look something like this:

```js
test('adds the product to the cart', async () => {
  // Draw the boundary at the network level so this test is self-contained.
  server.use(
    http.post('/cart', () => {
      return new Response(null, { status: 201 })
    })
  )

  // Interct with the UI.
  await page.getByRole('button', { name: 'Add "Porcelain mug" to cart' })

  // Assert on the user-facing outcome.
  await expect.element(page.getByRole('alert'))
    .toHaveTextContent(`Successfully added to cart!`)
})
```

Despite you creating a mock for the `POST /cart` request, that mock is a part of the test setup, not your expectations. Implementation details might belong in the setup but never next to your `expect()` functions.

By adding that request handler, you are achieving multiple things for this test:

1. You make it self-contained. The state and behavior of the production server cannot influence the outcome of this test anymore. This way, it focuses solely on whether your UI allows the user to add something to cart.
2. You implicitly declare any other request except for `POST /cart` as unhandled. If your app suddenly performs `PUT /cart`, your test will error because you haven't handled that request. This is one of the ways mocking can facilitate implicit assertions and help you test more by writing less.
3. You fix the interaction in the test to always receive a `201 Created` response. That mocked response, as well as the state of the network, becomes a given for this test (that is what the setup is for!).

But this test has one big flaw. While it clearly expects the product with the name "Porcelain mug" being added to the cart, the `POST /cart` request will succeed even if a different, wrong product ends up being added. That won't do!

## Better approach

This is where it becomes extremely temping to get that intercepted `request` reference and assert on its headers or body. But that would bring implementation details to the expectations realm, which we've long established we don't want to do.

What you can do instead is lean on the implicit characteristics of your mock.

The only time the user will actually see the successful notification is when the server (mock) responds with a successful status code. So make that code unsuccessful if the sent payload wasn't the one you expect for this interaction.

```js
test('adds the product to the cart', async () => {
  server.use(
    http.post('/cart', async ({ request }) => {
      const data = await request.clone().json()

      // Adding the request payload validation to the mock
      // provides an extra layer of assurance around the
      // request's correctness.
      if (data.productName !== 'Procelain mug') {
        return new Response(null, { status: 400 })
      }

      return new Response(null, { status: 201 })
    })
  )

  // ...
})
```

Now, if a wrong product ends up being sent in that request, that will result in the `400 Bad Request` response and no notification will be shown in the UI (or, perhaps, the user will be presented with an error instead). That's better!

To make it even better, you can bring the validation layer your application already has, such as payload schemas, and apply them to the intercepted request.

```js
import { addProductRequestSchema } from '../schemas/cart.js'

test('adds the product to the cart', async () => {
  server.use(
    http.post('/cart', async ({ request }) => {
      const data = await request.clone().json()

      // Parse the outgoing request against a schema
      // so malformed payloads throw, producing a 500 response.
      addProductRequestSchema.parse(data)

      if (data.productName !== 'Procelain mug') {
        return new Response(null, { status: 400 })
      }

      return new Response(null, { status: 201 })
    })
  )

  // ...
})
```

> Keep in mind that I'm not advocating for listing your entire business logic in the mock. That would prove detrimental to what you're trying to achieve here—reliable test coverage. What I'm illustrating with these examples is how you can use your test setup, not assertions, to reflect your expectations towards the outgoing requests.

This approach can be pushed even further, using tools like Source to generate a request handler from the actual source of truth—the API specification, which will not only ensure its validity but also synchronize your mocks with the state of the API you're using.

> The same applies to the responses, as they often result in the UI change that you should test instead. If the expected response wasn't received, the UI will not transition into the expected state, and your UI-based assertions will fail. That is the correct chain of network → UI expectations.

If you happened to use a standard-based API mocking library, you can also reuse your existing request validation logic straight in your request handlers, minimizing the amount of the setup you have to write.

If you find this approach verbose, you can tackle it as any other verbosity in the test setup—by introducing a utility function to keep your test cases clean. No matter where it is defined, the important part is that you keep your expectations focus on your users and leave everything else to the setup.

Keep this in mind whenever the thought of asserting on a request crosses your mind.
