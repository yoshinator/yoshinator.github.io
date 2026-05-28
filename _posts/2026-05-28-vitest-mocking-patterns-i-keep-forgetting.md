---
layout: post
title: Vitest Mocking Patterns I Keep Forgetting
date: 2026-05-28
---

I keep relearning the same few Vitest mocking patterns, so I wanted one place to put the versions that actually stick.

This is not a full testing guide. The [Vitest mocking docs](https://vitest.dev/guide/mocking.html) and [`vi` API docs](https://vitest.dev/api/vi) are where I would go for the exact rules. This is the smaller set of patterns I reach for when a React test gets stuck because the component is pulling from a hook, a provider, a named export, or a state updater that is hard to see from the outside.

The real issue is usually not Vitest itself. It is figuring out what I am trying to control.

It also fits the reason I started writing here in the first place: get the thing I keep learning out of my head and onto the page instead of making future me rediscover it again. I wrote about that a bit when I first stood up [this blog with Jekyll and GitHub Pages](/2026/02/22/starting-a-blog-in-5-minutes.html).

## Mocking a custom hook

If a hook just needs to return one fixed value for the whole file, the simplest version is a module mock:

```ts
vi.mock("hooks/someFile", () => ({
  useMyHook: () => true,
}))
```

That is enough when the test does not need much flexibility.

If I need to change the return value per test, I usually import the module as a namespace and spy on the hook:

```ts
import * as hooks from "hooks/someFile"

vi.spyOn(hooks, "useMyHook").mockImplementation(() => true)
```

That keeps the mock local to the test setup instead of making the whole file depend on one fixed factory.

## Mocking one named export but keeping the rest

When I want to mock one component or one named export and leave everything else alone, I use `importOriginal`:

```tsx
vi.mock("feature/someModule", async (importOriginal) => {
  const actualModule = await importOriginal()

  return {
    ...(actualModule as object),
    SomeComponent: ({ children }: { children: React.ReactNode }) => (
      <div>{children}</div>
    ),
  }
})
```

This is useful when a module exports a mix of helpers and UI pieces. I do not want to rebuild the whole module in the test. I only want to replace the part that makes the test noisy.

## Asserting that a mocked callback was called

If the function I want to assert lives inside the mock factory, I need to keep a handle to it.

With Vitest, the safer version is to use `vi.hoisted` because `vi.mock` factories are hoisted:

```ts
const mocks = vi.hoisted(() => ({
  onOpen: vi.fn(),
}))

vi.mock("path/to/module", () => ({
  useSomething: () => ({
    onOpen: mocks.onOpen,
  }),
}))

expect(mocks.onOpen).toHaveBeenCalledTimes(1)
```

The important part is that the test can inspect the same function the component received. If I create the function anonymously inside the factory and never expose it, I make the assertion harder than it needs to be.

## Testing a state updater function

Sometimes the thing passed into `setState` is more important than whether `setState` was called.

In those cases, I pull the updater function out of the mock call and run it manually:

```ts
expect(mockSetSomeState).toHaveBeenCalled()

const stateUpdateFunction = mockSetSomeState.mock.calls[0][0]

const prevState = ["Some Other Report"]
const newState = stateUpdateFunction(prevState)

expect(newState).toContain("something")
expect(newState).toHaveLength(2)
```

That makes the state transition testable. Instead of only checking that a setter fired, I can check what the updater would actually do with a previous state.

This is especially helpful when the component uses the callback form of `setState`:

```ts
setItems((prevItems) => [...prevItems, nextItem])
```

The behavior lives inside that function. Pulling it out makes the behavior visible.

## Wrapping a component with providers

If a component depends on context, React Testing Library's `render` accepts a `wrapper` option:

```tsx
render(ui, {
  wrapper: ({ children }) => <DataProvider>{children}</DataProvider>,
})
```

If the codebase has a shared helper like `renderWithMockProvider`, that wrapper pattern is probably most of what it is doing under the hood.

The useful thing to remember is that provider setup does not need to be mysterious. The test renders the component, and the wrapper gives it the context it normally expects from the app.

## The pattern underneath all of this

Most mocking problems get easier once I ask one question:

what am I actually trying to control here?

Usually it is one of these:

- the return value of a hook
- one named export inside a larger module
- the function passed to a callback or state setter
- the context wrapped around a component

Once I know which one I am dealing with, the mocking setup gets much simpler.

I still forget the exact syntax, which is why this note exists. But the shape is usually the same: keep the real module when possible, mock the smallest thing that creates friction, and make sure the test can still inspect the behavior that matters.
