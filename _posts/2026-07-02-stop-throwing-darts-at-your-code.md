---
layout: post
title: Stop Throwing Darts at Your Code
date: 2026-07-02
---

One of the most expensive habits in software is changing random things until the error goes away.

I call that dart throwing.

You see it when someone changes three props, adds `await` in two places, comments out a hook, clears a cache, reruns the test, and then says "okay, that fixed it" without really knowing what fixed it.

I have done this too. Everyone who builds software long enough does it at some point, especially when the thing is blocking real work and the pressure is to get green fast.

The problem is not that it sometimes works.

The problem is that it does not compound.

If you fix something without understanding why it was broken, you usually bought one short-term win and one future repeat problem. The next time the same class of issue shows up, you are back at zero.

That is why I try to separate two different jobs:

- getting the thing working again
- understanding the mechanism well enough that I can do it on purpose next time

Those two jobs do not always happen at the same speed.

Sometimes the right move is to stop the bleeding first. Fine. Ship the fix. Unblock the test. Get the page rendering again.

But if you never come back for the second job, you stay stuck in amateur mode longer than you need to.

The real issue is usually not that the code is "mysterious." It is that I do not yet have a clear model of what part is in control.

Is the bug coming from state shape?
Is it the mocked hook?
Is the provider missing?
Is the request never resolving?
Is React doing exactly what I told it to do, and I just told it the wrong thing?

Once I can answer that kind of question, the search space gets much smaller.

That is the part I like about testing work, even when it is annoying. A lot of test pain is really model pain.

I wrote down some [Vitest mocking patterns I keep forgetting](/2026/05/28/vitest-mocking-patterns-i-keep-forgetting.html) for exactly this reason. The value was not memorizing syntax. The value was forcing myself to name what I was actually trying to control: the hook return value, one named export, the provider wrapper, or the updater function passed into state.

That tiny shift matters.

When I know what I am controlling, I stop poking at the whole system.

I also think AI has made dart throwing easier.

Not better. Easier.

Now you can get five plausible fixes in thirty seconds. Add this dependency. Wrap that in `act`. Change the mock shape. Move the effect. Use a ref. Remove the ref. One of them might work.

But a fast pile of plausible guesses is still a pile of guesses.

That is why AI can speed up the wrong habit just as easily as the right one. It is great at generating options. It does not automatically give you understanding.

If anything, the better rule is this:

use AI to generate possibilities, then slow down enough to explain why the winning one should work.

If I cannot explain that in plain language, I probably do not own the fix yet.

This is also why I write so many small notes when I am debugging. I am not trying to sound smart. I am trying to make the mechanism visible. What was the bad assumption? Which piece actually owned the behavior? What did I think the library was doing that it was not doing?

That part compounds.

A person who understands one mocking pattern, one async edge, one render loop, or one API contract deeply is usually faster a month later than the person who got past all five by trial and error.

I am not arguing for turning every bug into a research paper.

I am arguing against pretending that "it works now" is the same as "I learned something."

Those are different outcomes.

What actually works better for me is a simple loop:

- make one concrete guess about what owns the behavior
- change one thing that tests that guess
- look at the result
- write down the pattern if it is likely to come back

That last step matters more than people think. If I do not write it down, there is a good chance I only rented the understanding.

That is true for tests, API work, build bugs, and even prompt work.

Especially prompt work, honestly.

There is a version of AI usage that is basically industrialized dart throwing. Keep asking for another version until one sounds right, then call that understanding. That works just well enough to be dangerous.

I would rather be slower and actually know what happened.

Not because perfect understanding is always possible.

Because real understanding is the part that pays me back.
