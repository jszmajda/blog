---
layout: post
title: Your Code Doesn't Remember What You Meant
---

A few months ago I wrote about [the arrow of intent](/code/2026/01/25/the-arrow-of-intent.html)—the idea that code should flow unidirectionally from what you want, through design and specs and tests, down into implementation. That post ended on an experiment. This one is about what the experiment became: a methodology I've been calling **Linked-Intent Development (LID)**, and the problem it solves that I don't see other spec-driven-development systems solving.

## Spec-driven development has a semantic diffusion problem

"Spec-driven development" doesn't mean much anymore. [Semantic diffusion](https://martinfowler.com/bliki/SemanticDiffusion.html) is Martin Fowler's name for what happens when a useful term spreads faster than its definition—it erodes into something everyone nods at and nobody acts on. SDD has diffused into "write something down before you code," which everyone already agrees with and which doesn't help. The systems that carry the name mostly share a single assumption, and it's the assumption I want to pick at.

The promise of SDD is that you can specify your intent better before you make a change. That's real, and it's useful. But almost every SDD system helps you define intent *for the change you're about to make right now*—and then hands the result to the codebase to remember. The code becomes the system of record for what you meant. The spec was scaffolding; once the code exists, the code is the truth.

Code is detailed, low-level, and—when an agent is producing thousands of lines of it per session—largely unread. It's easy to miss the spot where the change that got made quietly diverged from the change you meant. The Gnar Company [described the mechanism well](https://www.thegnar.com/blog/why-your-ai-coding-agent-keeps-making-bad-decisions-and-how-to-fix-it): each assumption is a coin flip, and "string enough of them together and you've got a codebase that technically runs but doesn't actually do what you need." Do that a few hundred times and you don't have a bug—you have a codebase whose intent core has rotted, one reasonable-looking decision at a time.

## What intent actually is

So what is intent? Intent is whatever you actually want the code to do—usually to solve a problem for a person or a business, sometimes to deliver value in some plainer way.

Take `ls`. It just lists the files in a directory. Simple. Except `ls` has an enormous amount of *latent* intent packed into it the moment you look closely. What are all the flags? What happens in a hidden-file directory versus a normal one? How does sorting behave when the locale changes? The answers feel obvious—but they only feel obvious because we've all used `ls` for decades and absorbed its conventions. If an alien on another planet sat down to write "a thing that lists files in a directory," they could hand you something completely different and be completely correct by their own lights.

When you describe what you want to an agent, you are leaning—constantly, invisibly—on the agent sharing your cultural context, your history, the way humans think, what you had for breakfast. All of that quietly shapes the decisions you'd make. When you don't say those things, the agent fills the blanks. Often it fills them well; with frontier models, probably most of the choices are good ones. Sometimes it makes a boneheaded choice because it assumed something, or believes something, that you simply don't agree with.

So intent has two parts. There's **direct intent**—the things you say to the agent—and there's **latent intent**—the things you didn't know you needed to say in order to get what you actually wanted out the other end.

## The two things most SDD systems miss

Most SDD systems miss two things. First, they don't *extract* your latent intent in the first place—they take the prompt you gave and run with it. FreeCodeCamp's framing of the failure mode is exact: it isn't hallucination, it's ["the agent being exactly as helpful as the prompt allowed, which wasn't helpful enough."](https://www.freecodecamp.org/news/how-to-stop-letting-ai-agents-guess-your-requirements/) Second, they don't *maintain* that intent over time, so that every later change is made in the context of everything the system is already supposed to be.

Without those two things, the agent does what agents do: it edits the codebase, and each edit encodes a decision that may or may not have matched your latent intent. Rely on the code to carry your complete intent forward and you've chosen a substrate that records every decision and remembers the reason for none of them. That's the rotting intent core. It accumulates quietly, and nothing flags it.

## How LID closes the gap

LID treats the agent as what it actually is: an English compiler. Specifications are the source; code is the compiled output. The job is to make the English unambiguous enough that the compiler doesn't have to guess—because the guesses are where intent gaps come from, and the gaps compound across sessions.

It does that in a specific order.

**First it aligns on true intent through the HLD.** The high-level design surfaces context, background, goals, *non-goals*, and the decisions you're making whether you announced them or not. The point of the HLD phase isn't documentation—it's interrogation: the agent questions you until the unspoken decisions are on the table. If you've used the [`/grill-me` skill](https://github.com/Jekudy/grillme-skill), this is the same instinct—Socratic pressure to surface what you already know but haven't said. The difference is where the answers go. `/grill-me` pulls them into the conversation, where they evaporate when the session ends; LID writes them into the HLD, where they outlive the session and constrain every change after it.

**Then it pins down the design decisions, component by component, in the LLDs.** The HLD settled *what* and *why*. The low-level designs are where you and the agent commit to *how*: the data models, the seams between components, what happens when something fails. These are exactly the decisions an agent makes silently the moment it starts writing code—and makes differently the next session. Writing them down first is what turns them from the agent's assumptions into your agreements.

**Then it links the clarified intent to the code.** LID uses [EARS](https://alistairmavin.com/ears/) specs—single-line requirements with globally unique IDs—and `@spec` annotations in tests and code. The arrow becomes walkable in both directions: from a line of code to the spec ID to the LLD section to the HLD principle, one identifier per hop, without loading the whole world into a context window.

The payoff shows up when intent *changes*. You don't start in the code. You start by clarifying the changed intent in the documents—and because it's expressed there, the change lands in the context of all the rest of the system's intent. It has to make sense against everything else you've already said you wanted. Most of the difficulty of using LID well is exactly this: getting your complete intent out of your head and into a form where you can trust the agent to make the same kinds of choices you would have made.

## This is just tenets, scaled down

None of this is new thinking for me—it predates agents by years. Before this I was a GM at AWS, running a product organization: teams of people shipping real things, most of the decisions getting made when I wasn't in the room. That's the actual job—not making the calls yourself, but making sure your teams make the calls you would have. Amazon has a tool built for exactly that: **tenets** (if "principles" lands better for you, read it that way—I'll say tenet). A tenet isn't a value and it isn't a rule—it's a tie-breaker. In one line, it says which way to lean when a decision has two defensible answers and no policy covers it.

The test of a real tenet is that its opposite is also reasonable. "We optimize for speed over correctness" is a tenet, because "correctness over speed" is a choice a different team would defensibly make. "Be good to customers" is not a tenet—it has no defensible opposite, so it decides nothing. And the whole apparatus carries Amazon's own escape hatch—*unless you know better ones*—because a tenet is meant to be durable, not sacred.

LID builds tenets into the HLD for the same reason Amazon does: so the call gets made the same way when the decider isn't in the room. With an agent the decider is *rarely* in the room—it's effectively a new teammate every session, holding none of yesterday's context. Tenets are how you hand it your priorities for the decisions you didn't know to anticipate, so it leans the way you would instead of guessing.

## A note on personas

One more thing about other SDD systems. A lot of them try to reflect current human engineering practice by pre-encoding it into your environment—the agent role-plays a product manager, then a program manager, then a QA engineer, then a developer, reviewing the intent as it passes between them. This presupposes that the way human teams are structured is the right structure for an agent to follow.

But that's not how agents work. It's one model wearing different hats—the same underlying thing, thinking in the same ways, just started from a different position in the output space each time. Those hats don't reliably reproduce the difference a real cross-functional team makes. The research bears this out: assigning a persona to a model is a [double-edged sword](https://arxiv.org/abs/2408.08631)—it doesn't reliably improve reasoning, and on plenty of tasks it does nothing or makes it worse. So LID doesn't take a position on team structure at all. Let the agent use whatever management scaffolding helps it. LID is *only* about expressing and maintaining intent over time. That's the whole job; the rest is the agent's to figure out.

## A real one, caught

Here is what that looks like in practice. I've been building a trading system with LID. Recently I asked the agent a maintenance question: go through the codebase and find anything you've built that doesn't trace back to a written-down intent—code that exists but that no design doc or spec ever asked for.

That question is only answerable because LID leaves a trail. Under LID, every behavior is supposed to have a one-line written spec, and the code that implements it carries a short comment pointing back to that spec. So "find code that traces back to nothing" isn't a vibe—it's a concrete search the agent can actually run.

It found six. Five were boring: real, working behavior nobody had bothered to write down. We wrote the intent down after the fact and moved on.

The sixth was the one that mattered. Somewhere in price handling, the code rounded halfway cases up—plain `round()`. But the design document for the system said prices use *banker's rounding* (round half to even), the same rule the pricing engine already used everywhere else. Both are legitimate ways to round a number. The code worked. The tests passed. Nothing was "broken." But the code was quietly doing something different from what the design said it should, in about ten places, and it shifted every reconciliation number downstream.

It's worth being precise about why this stays invisible in a normal workflow. If the intended behavior was never written down anywhere but the code, there is nothing to compare the code against—the code *is* the only statement of what it should do, so it trivially agrees with itself. Tests don't save you either: you write tests against what the code does, so they encode the rounding the code already does, and they pass. The only way a gap like this surfaces is if the intended behavior exists *separately from the code*, written down specifically enough that it can disagree with the code. That is the entire job of "the arrow": the design said half-to-even, the code said half-up, the two were linked, and so the disagreement could be seen at all.

And once you can see it, look at what it actually is. It is not a bug to go fix. It's a question only I can answer: should price handling match the engine and round half-to-even, or is rounding half-up genuinely fine here and the design document is the thing that's wrong? The agent cannot know—both are defensible, which is exactly the kind of decision a tenet exists to settle in advance. LID's whole contribution in this story is small and specific: it turned a silent disagreement buried in the code into a question someone actually got asked.

## Correctness comes back from correct intent

The frontier models we use for coding generally write good code. It compiles, it works, it usually doesn't have errors in the way we used to mean "errors." What it has instead are places where the agent assumed the wrong thing. Those aren't bugs. They're *intent gaps*—and I'm not the only one landing here. Microsoft Research's [work on intent formalization](https://arxiv.org/abs/2603.17150) puts it sharply: AI-generated code is *plausible by construction, not correct by construction*. That plausibility is the whole problem—it looks right, which is exactly why the gap is easy to miss.

And if you write tests for code that does the wrong thing for your users, the tests pass and the software is still wrong: green suite, broken product. In a world where the agent writes correct code on demand, correctness stops being a property of the code. It comes back from correct intent, and from the correct expression of it. That's the thing LID exists to protect.

## Try it

LID ships as two Claude Code plugins. If you want to try it:

```
/plugin marketplace add jszmajda/lid
/plugin install linked-intent-dev@jszmajda-lid
/plugin install arrow-maintenance@jszmajda-lid
```

`linked-intent-dev` is the core workflow—consult it for every code change. `arrow-maintenance` overlays navigation and tracking for projects too big to hold in one context window, and includes a `/map-codebase` command for bootstrapping LID onto an existing codebase. The repo also has rule-file adapters for Cursor, Windsurf, Copilot, Aider, and anything else that reads `AGENTS.md`—the methodology is identical across tools; only the invocation changes.

Background and docs are at [linked-intent.dev](https://linked-intent.dev). It's early, and the friction reports are the useful ones—issues and PRs welcome at [github.com/jszmajda/lid](https://github.com/jszmajda/lid).

It's still evolving—the [arrow of intent post](/code/2026/01/25/the-arrow-of-intent.html) was pre-LID, and a lot has changed since. But the core has held: intent flows one direction, you maintain it deliberately, and you stop pretending the code remembers what you meant. And in truth, we were never really writing code. We were producing durable, automatic solutions to human problems—code was only ever the medium. The intent behind it was always the real artifact. LID just treats it that way.
