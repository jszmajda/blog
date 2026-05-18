---
layout: post
title: Your Code Doesn't Remember What You Meant
---

A few months ago I wrote about [the arrow of intent](/code/2026/01/25/the-arrow-of-intent.html)—code should flow one way: from what you want, through design and specs and tests, into implementation. That post ended on an experiment. This one is about what the experiment became: **[Linked-Intent Development (LID)](https://linked-intent.dev)**.

In one paragraph: with LID you keep what you want the system to do in design docs and one-line specs, and every spec carries a short ID. The code and tests that satisfy a spec cite its ID in a comment. The agent maintains those links as it works, so at any point you can walk from a line of code back to the decision that asked for it—or the other way around. That's the whole mechanism.

## A bug I caught last week

I've been building a paper trading system using LID (like everyone else right now, lol). I asked the agent a maintenance question: find anything in the codebase that doesn't trace back to a written-down intent—code that exists but that no design doc or spec ever asked for. Because every behavior is supposed to have a spec and every implementation cites one.

It found six. Five were boring: working behavior nobody had written down. We wrote it down and moved on.

The sixth mattered. In price handling, the code rounded halfway cases up—plain `round()`. But the design document said prices use *banker's rounding* (round half to even), the same rule the pricing engine used everywhere else. Both are legitimate. The code worked. The tests passed. Nothing was "broken." The code was just quietly doing something other than what the design said, in about ten places, and it moved every reconciliation number downstream.

The tests passed because the code was written against the tests with the wrong behavior. They didn't match the design—so they certified the rounding the code already did. Nothing in a normal setup would have flagged this. It surfaced only because the intended behavior was written down somewhere the code couldn't quietly overwrite, and linked tightly enough to the code that the two could be seen to disagree.

And the disagreement isn't a bug to go fix. It's a question only I can answer: should price handling match the engine, or is rounding up genuinely fine here and the design document is the thing that's wrong? The agent can't know—both are defensible. LID's contribution here is small and specific: it turned a silent disagreement buried in the code into a question someone actually got asked.

That's the idea in one bug. The rest of this post is why it works.

## Spec-driven development has a semantic diffusion problem

With agentic coding, everyone is talking about spec-driven development—it's the box LID would get filed under—so let me share why I think the box is broken.

"Spec-driven development" doesn't mean much anymore. [Semantic diffusion](https://martinfowler.com/bliki/SemanticDiffusion.html) is Martin Fowler's name for what happens when a useful term spreads faster than its definition—it erodes into something everyone nods at and nobody acts on. SDD has diffused into "write something down before you code," which everyone already agrees with and which doesn't help. The systems that carry the name mostly share a single assumption, and it's the assumption I want to pick at.

The promise of SDD is that you can specify your intent better before you make a change. That's real, and it's useful. But almost every SDD system helps you define intent *for the change you're about to make right now*—and then hands the result to the codebase to remember. The code becomes the system of record for what you meant. The spec was scaffolding; once the code exists, the code is the truth.

Code is detailed, low-level, and—when an agent is producing thousands of lines of it per session—largely unread. Each unreviewed decision is a small bet that the agent guessed what you meant. String enough of those together and you don't have a bug; you have a codebase that runs and doesn't do what you need, an intent core that has rotted, one reasonable-looking decision at a time.

## What intent actually is

Intent is whatever you actually want the code to do—usually to solve a problem for a person or a business.

Take `ls`. It just lists files in a directory—simple, until you look closely. What are all the flags? What happens with hidden files, or when the locale changes the sort order? The answers feel obvious only because we've absorbed `ls`'s conventions over decades; an alien asked to "list the files in a directory" could hand you something completely different and be completely right by their own lights.

When you describe what you want to an agent, you are leaning—constantly, invisibly—on the agent sharing your cultural context, your history, the way humans think, what you had for breakfast. All of that quietly shapes the decisions you'd make. When you don't say those things, the agent fills the blanks. Often it fills them well; with today's frontier models, probably most of the choices are good ones. Sometimes it makes a boneheaded choice because it assumed something you simply don't agree with.

So intent has two parts. There's **direct intent**—the things you say to the agent—and there's **latent intent**—the things you didn't know you needed to say in order to get what you actually wanted out the other end.

## The two things most SDD systems miss

Most SDD systems miss two things. First, they don't *extract* your latent intent in the first place—they take the prompt you gave and run with it. Addy Osmani makes the same point in his [guide to writing specs for agents](https://addyosmani.com/blog/good-spec/): a foggy prompt doesn't fail loudly, it produces polished, confident, wrong output. Second, they don't *maintain* that intent over time, so that every later change is made in the context of everything the system is already supposed to be.

Without those two things, the agent does what agents do: it edits the codebase, and each edit encodes a decision that may or may not have matched your latent intent. Rely on the code to carry your complete intent forward and you've chosen a substrate that records every decision and remembers the reason for none of them. That's the rotting intent core. It accumulates quietly, and nothing flags it.

## How LID closes the gap

LID treats the agent like a compiler for English: specifications are the source, code is the output. It's a lossy, non-deterministic compiler—it guesses whenever the English is ambiguous, and the guesses are where intent gaps come from. The whole job is to leave it less to guess. (That cuts the other way too: vague input doesn't just produce one bad guess, it compounds into drift across sessions.)

It does that in a specific order.

**First it aligns on true intent through the HLD**—the high-level design: one document covering context, background, goals, *non-goals*, and the decisions you're making whether you announced them or not. The point of the HLD phase isn't documentation—it's interrogation: the agent questions you until the unspoken decisions are on the table. If you've used the [`/grill-me` skill](https://github.com/Jekudy/grillme-skill), this is the same instinct—Socratic pressure to surface what you already know but haven't said. The difference is where the answers go. `/grill-me` pulls them into the conversation, where they evaporate when the session ends; LID writes them into the HLD, where they outlive the session and constrain every change after it.

**Then it pins down the design decisions, component by component, in the LLDs**—the low-level designs, one per component. The HLD settled *what* and *why*. The LLDs are where you and the agent commit to *how*: the data models, the seams between components, what happens when something fails. These are exactly the decisions an agent makes silently the moment it starts writing code—and makes differently the next session. Writing them down first is what turns them from the agent's assumptions into your agreements.

**Then it links that intent to the code.** Each requirement becomes a single line with a unique ID, written in [EARS](https://alistairmavin.com/ears/) (a boring, regular phrasing for requirements), and the code and tests that satisfy it carry a comment naming that ID:

```
# in the spec file:
PRICE-ROUND-014: Price ingest SHALL round half to even (banker's rounding).

# in the code:
// @spec PRICE-ROUND-014
fn to_cents(price: f64) -> i64 { ... }
```

Now a grep for `PRICE-ROUND-014` turns up the rule, the code that implements it, and the test that checks it. That ID is the entire linkage mechanism—deliberately dull, so the agent can walk from a line of code up to the decision that asked for it (and back) without re-reading the whole project to orient itself. That walkability is the part the rounding bug turned on.

The payoff shows up when intent *changes*. You don't start in the code. You start by changing the intent in the documents—and because it's expressed there, the change lands in the context of all the rest of the system's intent. It has to make sense against everything else you've already said you wanted. Most of the difficulty of using LID well is exactly this: getting your complete intent out of your head and into a form where you can trust the agent to make the same kinds of choices you would have made. It is real work, every change, not a one-time setup.

## Tenets: deciding what you'd decide

None of this is new thinking for me—it predates agents by years. Before this I was a GM at AWS, running a product organization: teams of people shipping real things, most of the decisions getting made when I wasn't in the room. That's the actual job—not making the calls yourself, but making sure your teams make the calls you would have. Amazon has a tool built for exactly that: **tenets** (if "principles" lands better for you, read it that way—I'll say tenet). A tenet isn't a value and it isn't a rule—it's a tie-breaker. In one line, it says which way to lean when a decision has two defensible answers and no policy covers it.

The test of a real tenet is that its opposite is also reasonable. "We optimize for speed over correctness" is a tenet, because "correctness over speed" is a choice a different team would defensibly make. "Be good to customers" is not a tenet—it has no defensible opposite, so it decides nothing. And the whole apparatus carries Amazon's own escape hatch—*unless you know better ones*—because a tenet is meant to be durable, not sacred.

LID builds tenets into the HLD for the same reason Amazon does: so the call gets made the same way when the decider isn't in the room. With an agent the decider is *rarely* in the room—it's effectively a new teammate every session, holding none of yesterday's context. Tenets are how you hand it your priorities for the decisions you didn't know to anticipate, so it leans the way you would instead of guessing. The rounding question—half-to-even or half-up—is exactly the shape of decision a tenet is supposed to have already answered.

## A note on personas

One more thing about other SDD systems. A lot of them pre-encode current human team structure into your environment—the agent role-plays a product manager, then a program manager, then a QA engineer, then a developer, passing the intent between them. That presupposes the way human teams are organized is the right structure for an agent to follow.

But that's not how agents work. It's one model wearing different hats—the same underlying thing, thinking the same way, just started from a different position each time. The research bears this out: assigning a persona to a model is a [double-edged sword](https://arxiv.org/abs/2408.08631)—it doesn't reliably improve reasoning, and on plenty of tasks it does nothing or makes it worse. So LID takes no position on team structure. Let the agent use whatever scaffolding helps it; LID is only about expressing and maintaining intent over time.

## But won't the docs rot too?

This is the obvious objection, and it's the right one. Everything I said about code—records the decision, forgets the reason, drifts silently, goes unread—is exactly what killed every prior version of this idea. Requirements traceability matrices rotted. The executable specifications of [Specification by Example](https://en.wikipedia.org/wiki/Specification_by_example) drifted from the systems they specified. Design-by-contract assertions went stale. It's a large graveyard and I'm not the first to walk into it.

So why would an HLD/LLD/spec tree be any different? Three reasons it might be, and the limit of them.

First, the docs are small and in English. An HLD is an orientation document, not a thousand-line file nobody reads, and it's written to be read cold by the next session—because with an agent the next session is always a stranger. Rot in a 200-line design doc is visible in a way rot across 275,000 lines of code is not.

Second, the links are cheap and the agent walks them as part of doing the work. The thing that actually killed traceability matrices was that maintaining the links was manual and nobody did it. Here the link is a grep-able ID the agent follows every time it touches the area—not a parallel artifact a human updates out of duty.

Third—and this is what the rounding bug was really about—LID runs an audit that hunts the drift. The reason that gap surfaced is that I can point the agent at the whole tree and ask "what here disagrees with what here." Coherence between intent and code is checkable on demand, and the check is the same one that found the rounding bug.

I won't oversell it: none of this makes drift impossible. It makes drift *detectable and repairable* instead of silent and compounding. That's a weaker claim than "the docs are always right," and it's the true one. LID's bet isn't that intent never rots—it's that rotting intent you can find beats rotting intent you can't.

This started on one project—[Threadkeeper](https://threadkeeper.app), my side project for voice journaling. It's spreading though: I know of a couple dozen codebases running LID, and the reports coming back are that it does what it claims. That's encouraging, but not a benchmark—I'm excited to understand where it breaks for other people.

## Correctness comes back from correct intent

The models we use for coding generally write good code. It compiles, it works, it usually doesn't have errors in the way we used to mean "errors." What it has instead are places where the agent assumed the wrong thing. Those aren't bugs—they're *intent gaps*, and I'm not the only one who's found this. Microsoft Research's [work on intent formalization](https://arxiv.org/abs/2603.17150) frames the core question the same way: not whether the generated code runs, but whether it does what the user actually intended. Plausible output is not the same as correct output, and the plausibility is what makes the gap easy to miss.

If you write tests for code that does the wrong thing for your users, the tests pass and the software is still wrong: green suite, broken product. In a world where the agent writes correct code on demand, correctness stops being a property of the code. It comes back from correct intent, and from the correct expression of it. That's the thing LID exists to protect.

## Try it

[LID](https://linked-intent.dev) ships as two Claude Code plugins. If you want to try it:

```
/plugin marketplace add jszmajda/lid
/plugin install linked-intent-dev@jszmajda-lid
/plugin install arrow-maintenance@jszmajda-lid
```

`linked-intent-dev` is the core workflow—consult it for every code change. `arrow-maintenance` overlays navigation and tracking for projects too big to hold in one context window, and includes a `/map-codebase` command for bootstrapping LID onto an existing codebase. The repo also has rule-file adapters for Cursor, Windsurf, Copilot, Aider, and anything else that reads `AGENTS.md`—the methodology is identical across tools; only the invocation changes.

Background and docs are at [linked-intent.dev](https://linked-intent.dev). Note that LID is built with LID itself - so feel free to check out how that all works too. But it's early, and the friction reports are the useful ones—issues and PRs welcome at [github.com/jszmajda/lid](https://github.com/jszmajda/lid).

It's still evolving—the [arrow of intent post](/code/2026/01/25/the-arrow-of-intent.html) was pre-LID, and a lot has changed since. But the core has held: intent flows one direction, you maintain it deliberately, and you stop pretending the code remembers what you meant.
