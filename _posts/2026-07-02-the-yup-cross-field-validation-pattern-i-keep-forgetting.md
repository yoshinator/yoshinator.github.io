---
layout: post
title: The Yup Cross-Field Validation Pattern I Keep Forgetting
date: 2026-07-02
---

I keep running into the same small Yup problem:

one field needs to validate against another field, and I can never remember the cleanest way to do it.

This usually shows up in forms where one value depends on a sibling value. Password confirmation is the obvious example, but it also comes up with date ranges, paired numeric inputs, and forms where one field becomes required only when another field has a certain value.

The pattern I come back to is using `test` and reading the sibling values from `context.parent`.

## Use `context.parent` inside `test`

If I need the current field value and I also need to inspect the rest of the form, `test` is usually the simplest place to do it.

```ts
import * as yup from "yup"

const schema = yup.object({
  password: yup.string().required(),
  confirmPassword: yup
    .string()
    .required("Please confirm your password")
    .test({
      name: "matches-password",
      message: "Passwords must match",
      test: (value, context) => value === context.parent.password,
    }),
})
```

The important part is `context.parent`. That gives me the other values in the same object, so I do not have to push a bunch of state around just to compare two fields.

If I use a regular function instead of an arrow function, I can also reach the same data with `this.parent`. I usually stick with `context.parent` because I think it reads more clearly.

## This is useful beyond password confirmation

The same pattern works anywhere the rule depends on more than one field.

For example, if I want an `endDate` to stay on or after `startDate`, I can compare them directly inside the test:

```ts
const schema = yup.object({
  startDate: yup.date().required(),
  endDate: yup
    .date()
    .required()
    .test({
      name: "end-after-start",
      message: "End date must be on or after start date",
      test: (value, context) => {
        const { startDate } = context.parent
        if (!value || !startDate) return true
        return value >= startDate
      },
    }),
})
```

That pattern is usually easier for me to reason about than bouncing between multiple `when` branches.

## The other thing I forget: cyclic dependency errors

Sometimes two fields depend on each other and Yup complains about a cyclic dependency.

When that happens, I use the second argument to `shape` and declare the dependency pair explicitly:

```ts
const schema = yup.object().shape(
  {
    field1: foo,
    field2: bar,
  },
  [["field1", "field2"]]
)
```

I do not need this often, but when I do, it saves me from wasting time debugging the wrong thing.

## The main takeaway

When a Yup rule needs access to sibling fields, I reach for `test` first and inspect `context.parent`.

When Yup starts complaining that two fields depend on each other, I check whether I need to declare the dependency pair in `shape`.

This is exactly the kind of small technical pattern I like to write down, for the same reason I wrote [Vitest Mocking Patterns I Keep Forgetting](/2026/05/28/vitest-mocking-patterns-i-keep-forgetting.html). A lot of useful notes are not deep theories. They are just small pieces of friction that are worth solving once and keeping.

That is also why I care so much about [information management](/2026/06/04/information-management-matters-more-than-time-management.html). The goal is not to collect more notes. The goal is to stop paying the same mental tax every time an old problem comes back.
