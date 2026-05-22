---
layout: post
title: How Math Builders Works
date: 2026-05-22
---

If you've looked at Math Builders and wondered what it's actually doing under the hood, the short answer is this:

it's a scheduler, not a worksheet generator.

Every fact a student sees has its own little memory record attached to it. The app tracks which review box that fact is in, when it should appear again, how many times the student has seen it, how fast they answered, whether they missed it recently, and whether it's becoming automatic or still shaky.

That matters because the goal is not just "got it right once." The goal is to get a fact from effortful recall to instant recall.

The core system is a 16-box spaced repetition model. The early boxes are very short on purpose: 1 minute, 2 minutes, 4 minutes, 9 minutes, 15 minutes, and 30 minutes. After that, the intervals widen to 1 day, 3 days, 7 days, 21 days, 60 days, 180 days, 1 year, 3 years, 10 years, and 20 years. Easy facts disappear for longer stretches. Weak facts come back quickly.

Math Builders also grades on more than accuracy. It cares about speed.

- wrong answers reset the fact to box 1
- correct answers under 3 seconds move the fact forward
- instant answers under 1 second can jump even further
- correct answers around 3 to 5 seconds can hold a fact in place
- correct answers around 5 to 9 seconds can push a fact backward
- very slow answers reset hard, because "eventually right" is not the same thing as automatic

That speed check is one of the most important parts of the system, but the student doesn't experience it as pressure. The timer is used by the scheduler, not weaponized in the interface. The app quietly records response time and uses it to decide what happens next. That's how Math Builders can aim for automaticity without turning every session into a stressful race. I wrote more about that target here: [How to Build Math Automaticity](https://mathbuilders.com/how-to-build-math-automaticity).

The session queue is also more deliberate than it probably looks from the outside.

When a student starts a session, the app builds the queue in three passes:

1. due facts come first
2. if there's still room, it pulls in learning facts from the lowest boxes
3. if there's still room after that, it introduces new facts within the student's daily cap

That last part matters more than it sounds. New cards are not just dumped in a flat sequence. The system mostly pulls from the front of the pack, but it also mixes in a small percentage of slightly deeper cards. That keeps an early multiplication session from feeling like an endless wall of the 1-times table while still keeping the overall difficulty manageable.

Another nice detail is what happens to fragile facts. If a fact is still sitting in one of the first few boxes, it gets re-queued back into the same session after an answer. That means a student can see it again today instead of waiting for tomorrow. Once a fact moves beyond those early learning boxes, it usually leaves the current session and comes back later when it's actually due.

The system also stays scoped to the active pack. If a student is working on multiplication up to 12, the queue is built from that pack. If they're in addition within 20, the queue is built from that pack instead. That sounds obvious, but it prevents practice from turning into a random blur of unrelated facts.

Behind the scenes, Math Builders keeps updating the same fact record over time: box, next due time, streak, correctness, average response time, recent misses, and more. Those updates are saved in small batches, which makes the product more resilient if a session gets interrupted halfway through.

There are a few other interesting mechanics buried in the implementation too. New users can start with a smaller first session. Packs can provision more cards when the local queue gets too thin. And a pack is basically treated as mastered once about 80 percent of its facts have reached box 9 or higher, which means those facts are surviving on week-scale review intervals instead of minute-scale ones.

So if I had to describe Math Builders in one sentence, I'd say this:

it's a memory queue for math facts.

The app is always making a decision about what to show now, what to delay, what to repeat today, and what is finally strong enough to leave alone for a while. That's the real product. The cards and scenes are just the interface sitting on top of that system.
