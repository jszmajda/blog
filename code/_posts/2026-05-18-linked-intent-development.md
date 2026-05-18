---
layout: post
title: AI Is Rotting Your Codebase
---

A few months ago I wrote about [the arrow of intent](/code/2026/01/25/the-arrow-of-intent.html)—the idea that code should flow unidirectionally from what you want, through design and specs and tests, down into implementation. That post ended on an experiment. This one is about what the experiment became: a methodology I've been calling **Linked-Intent Development (LID)**, and the problem it solves that I don't see other spec-driven-development systems solving.

## Spec-driven development has a semantic diffusion problem

"Spec-driven development" doesn't mean much anymore. [Semantic diffusion](https://martinfowler.com/bliki/SemanticDiffusion.html) is Martin Fowler's name for what happens when a useful term spreads faster than its definition—it erodes into something everyone nods at and nobody acts on. SDD has diffused into "write something down before you code," which everyone already agrees with and which doesn't help. The systems that carry the name mostly share a single assumption, and it's the assumption I want to pick at.

The promise of SDD is that you can specify your intent better before you make a change. That's real, and it's useful. But almost every SDD system helps you define intent *for the change you're about to make right now*—and then hands the result to the codebase to remember. The code becomes the system of record for what you meant. The spec was scaffolding; once the code exists, the code is the truth.

That works right up until it doesn't. Code is detailed, low-level, and—when an agent is producing it by the thousands of lines per session—largely unread. It's easy for a human to miss the place where the change that got made quietly diverged from the change they meant. Do that a few hundred times and you don't have a bug. You have a codebase whose intent core has rotted, one reasonable-looking decision at a time.

## What intent actually is

So what is intent? Intent is whatever you actually want the code to do—usually to solve a problem for a person or a business, sometimes to deliver value in some plainer way.

Take `ls`. It just lists the files in a directory. Simple. Except `ls` has an enormous amount of *latent* intent packed into it the moment you look closely. What are all the flags? What happens in a hidden-file directory versus a normal one? How does sorting behave when the locale changes? The answers feel obvious—but they only feel obvious because we've all used `ls` for decades and absorbed its conventions. If an alien on another planet sat down to write "a thing that lists files in a directory," they could hand you something completely different and be completely correct by their own lights.

When you describe what you want to an agent, you are leaning—constantly, invisibly—on the agent sharing your cultural context, your history, the way humans think, what you had for breakfast. All of that quietly shapes the decisions you'd make. When you don't say those things, the agent fills the blanks. Often it fills them well; with frontier models, probably most of the choices are good ones. Sometimes it makes a boneheaded choice because it assumed something, or believes something, that you simply don't agree with.

That's the real shape of intent. There's **direct intent**—the things you say to the agent—and there's **latent intent**—the things you didn't know you needed to say in order to get what you actually wanted out the other end.

## The two things most SDD systems miss

Here's the gap. Most SDD systems miss the need to *extract latent intent* from the user in the first place. And they miss the need to *maintain all of that intent over time*, so that every subsequent change is made in the context of every other thing the system is already supposed to be.

Without those two things, the agent does what agents do: it edits the codebase, and each edit encodes a decision that may or may not have matched your latent intent. Rely on the code to carry your complete intent forward and you've chosen a substrate that records every decision and remembers the reason for none of them. That's the rotting intent core. It doesn't announce itself. It accumulates.

## How LID closes the gap

LID treats the agent as what it actually is: an English compiler. Specifications are the source; code is the compiled output. The job is to make the English unambiguous enough that the compiler doesn't have to guess—because the guesses are where intent gaps come from, and the gaps compound across sessions.

It does that in a specific order.

**First it aligns on true intent through the HLD.** The high-level design surfaces context, background, goals, *non-goals*, and the decisions you're making whether you announced them or not. The point of the HLD phase isn't documentation—it's interrogation. LID guides the agent to walk you through your own latent intent and pull it into the open, step by step.

**Then it pushes the assumptions down into the LLDs.** System design, data models, messaging, the seams between components—these are where "the agent would obviously know that" goes to die. LID makes you and the agent write those assumptions down as an agreed document for how to proceed, instead of leaving them to be inferred later under context pressure.

**Then it links the clarified intent to the code.** LID uses [EARS](https://alistairmavin.com/ears/) specs—single-line requirements with globally unique IDs—and `@spec` annotations in tests and code. The arrow becomes walkable in both directions: from a line of code to the spec ID to the LLD section to the HLD principle, one identifier per hop, without loading the whole world into a context window.

The payoff shows up when intent *changes*. You don't start in the code. You start by clarifying the changed intent in the documents—and because it's expressed there, the change lands in the context of all the rest of the system's intent. It has to make sense against everything else you've already said you wanted. Honestly, most of the difficulty of using LID well is exactly this: getting your complete intent out of your head and into a form where you can trust the agent to make the same kinds of choices you would have made.

## This is just tenets, scaled down

None of this is new thinking for me—it predates agents. At AWS the problem was encoding my own decision-making so the team could make the same calls when I wasn't in the room. Amazon has a tool for exactly that: **tenets**. A tenet isn't a value and it isn't a rule—it's a tie-breaker. In one line, it says which way to lean when a decision has two defensible answers and no policy covers it.

The test of a real tenet is that its opposite is also reasonable. "We optimize for speed" is a tenet, because "we optimize for correctness" is a coherent choice a different team would make. "Be good to customers" is not a tenet—it has no defensible opposite, so it resolves nothing. And the whole apparatus carries Amazon's own escape hatch—*unless you know better ones*—because a tenet is meant to be durable, not sacred.

LID builds tenets into the HLD for the same reason Amazon does: so the call gets made the same way when the decider isn't in the room. With an agent, the decider is *never* in the room—it's a fresh teammate every session, holding none of yesterday's context. Tenets are how you hand it your priorities for the decisions you didn't know to anticipate, so it leans the way you would instead of guessing.

## A note on personas

One more thing about other SDD systems. A lot of them try to reflect current human engineering practice by pre-encoding it into your environment—the agent role-plays a product manager, then a program manager, then a QA engineer, then a developer, reviewing the intent as it passes between them. This presupposes that the way human teams are structured is the right structure for an agent to follow.

But that's not how agents work. It's one model wearing different hats—the same underlying thing, thinking in the same ways, just started from a different position in the output space each time. Those hats don't reliably reproduce the difference a real cross-functional team makes. The research bears this out: assigning a persona to a model is a [double-edged sword](https://arxiv.org/abs/2408.08631)—it doesn't reliably improve reasoning, and on plenty of tasks it does nothing or makes it worse. So LID doesn't take a position on team structure at all. Let the agent use whatever management scaffolding helps it. LID is *only* about expressing and maintaining intent over time. That's the whole job; the rest is the agent's to figure out.

## A real one, caught

I don't hit the December-import kind of surprise much anymore—not because the agent stopped making assumptions, but because the arrows are now in place to catch them. Here's a recent one, from a trading system I've been building.

I asked the agent to audit the codebase for anything it had built that no arrow covered—code with no traceable line back up to intent. It found six. Five were the ordinary kind: real behavior that worked fine but had never been written down, now specified as-built.

The sixth was the interesting one. Price ingest rounded half-away-from-zero—plain `round()`—while the design doc said the system used banker's rounding (round-half-to-even), the same mode the pricing engine uses internally for margin. The code was correct. The tests passed. Nothing was broken. It was a pure intent gap: a decision the agent had made at a boundary, entirely plausible in isolation, silently disagreeing with stated intent across about ten call sites and every reconciliation figure downstream.

Without the arrow, that's invisible—it looks like working code, because it *is* working code. With the arrow, it surfaced as exactly what it was: not a bug to fix, but a decision only I could make. Flip ingest to banker's rounding to match the engine, or consciously ratify half-away in the design and pin the tie direction with a test? The agent couldn't answer that. It's my intent to set. The arrow's only job was to make sure the question got asked instead of buried.

That's the whole argument in one example. The code was never wrong. The intent was unsettled—and nothing in a normal workflow would ever have told me.

## Correctness comes back from correct intent

That story generalizes. The frontier models we use for coding write good code. It compiles, it works, it usually doesn't have errors in the way we used to mean "errors." What it has instead are places where the agent assumed the wrong thing. Those aren't bugs. They're *intent gaps*—and I'm not the only one landing here. Microsoft Research's [work on intent formalization](https://arxiv.org/abs/2603.17150) puts it sharply: AI-generated code is *plausible by construction, not correct by construction*. The plausibility is the trap. It looks right, so the gap hides.

And if you write tests for code that does the wrong thing for your users, the tests pass and the software is still wrong. The green checkmark is lying to you, faithfully. In a world where the agent writes correct code on demand, correctness stops being a property of the code. It comes back from correct intent, and from the correct expression of correct intent. That's the thing LID exists to protect.

## Try it

LID ships as two Claude Code plugins. If you want to try it:

```
/plugin marketplace add jszmajda/lid
/plugin install linked-intent-dev@jszmajda-lid
/plugin install arrow-maintenance@jszmajda-lid
```

`linked-intent-dev` is the core workflow—consult it for every code change. `arrow-maintenance` overlays navigation and tracking for projects too big to hold in one context window, and includes a `/map-codebase` command for bootstrapping LID onto an existing codebase. The repo also has rule-file adapters for Cursor, Windsurf, Copilot, Aider, and anything else that reads `AGENTS.md`—the methodology is identical across tools; only the invocation changes.

It's still evolving—the [arrow of intent post](/code/2026/01/25/the-arrow-of-intent.html) was pre-LID, and a lot has changed since. But the core has held: intent flows one direction, you maintain it deliberately, and you stop pretending the code remembers what you meant. We're not writing code anymore. We're producing software—and the thing worth maintaining is the intent behind it.
