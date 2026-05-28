---
layout: post
title: Vitest Mocking Patterns I Keep Forgetting
date: 2026-05-28
---

I keep relearning the same few Vitest mocking patterns, so I wanted one place to put the versions that actually stick.

This is not a full guide. It is the small set of patterns I end up needing over and over in React tests, especially when a component depends on a hook, provider, callback, or one annoying named export.

## Mocking a custom hook

If I just need a hook to return a fixed value, the simplest version is:

```ts
vi.mock("hooks/someFile", () => ({
  useMyHook: () => true,
}))
```

If I need to change the return value per test, importing the whole module and using `spyOn` is usually easier to reason about:

```ts
import * as hooks from "hooks/someFile"

vi.spyOn(hooks, "useMyHook").mockImplementation(() => true)
```

That is cleaner than trying to redefine the whole mock for each test.

## Mocking one named export but keeping the rest of the module

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

This is especially useful when a module exports a mix of helpers and UI pieces and I only want to replace one of them.

## Asserting that a mocked callback was called

If the thing I want to assert lives inside the mock factory, I declare it outside first:

```ts
const onOpen = vi.fn()

vi.mock("path/to/module", () => ({
  useSomething: () => ({
    onOpen,
  }),
}))

expect(onOpen).toHaveBeenCalledTimes(1)
```

If I create the function inside the factory, it is much harder to inspect cleanly later.

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

That makes the state transition testable instead of only checking that a setter fired.

## Wrapping a component with providers

If a component depends on context, remember that React Testing Library's `render` accepts a `wrapper`:

```tsx
render(ui, {
  wrapper: ({ children }) => <DataProvider>{children}</DataProvider>,
})
```

If the codebase has a shared `renderWithMockProvider`, that wrapper pattern is usually all it is doing underneath.

## The pattern underneath all of this

Most mocking problems get easier once I ask one question:

what am I actually trying to control here?

Usually it is one of three things:

- the return value of a hook
- one named export inside a larger module
- the function passed to a callback or state setter

Once I know which of those I am dealing with, the mocking setup gets much simpler.
