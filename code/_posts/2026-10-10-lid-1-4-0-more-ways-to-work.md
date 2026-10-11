---
layout: post
title: "LID 1.4.0: More Ways to Work"
---

A lot more people have started using [LID](https://linked-intent.dev)! It's been super exciting to see more people use it, and I've been hearing how much it's been helping them with their agentic development.

Then I started watching how they used it, and was surprised how different their styles could be!

That surprise turned into [LID 1.4.0](https://github.com/jszmajda/lid/discussions/85).

### If this is your first time here

LID is a way of working with a coding agent where what you want lives in design docs and one-line specs, not in the agent's head or in the code. Each spec has a short ID, and the code and tests that satisfy it cite that ID, so you can walk from the code back to the decision that generated it. Work flows one way - a high-level design (the HLD), low-level designs per component (the LLDs), then specs, tests, code - and the agent stops between phases so you can be sure you're on the same page. That chain is called the "arrow." 

Most of the time, people who work with agents to write code find out what the agent assumed once the code is done (if at all); LID catches it while it's still a sentence in a doc. The longer version is in my post from May: [Your Code Doesn't Remember What You Meant](/code/2026/05/18/your-code-doesnt-remember-what-you-meant.html).

### My way, and others'

When I wrote LID, I based it on how I like to work (which I wrote up in [How I Claude Code](/code/2025/11/15/how-i-claude-code.html)): the agent and I draft a design together, then I send back one long reply - numbered and specific, often a dozen items. I know which phases I want to watch, and I say so up front: "I don't need to see the tests on this one, let it rip."

Others didn't always do the same. They'd answer one question at a time, and agents using LID would, over time, sometimes lose track of where they were. Or users would sit through every inspection stop because they didn't know they could skip some. One person said "I love LID, but all the reviews get really frustrating! Do I have to do all of them?" Others accepted the reviews as just what they had to get through to get to what they wanted. LID benefits from review, but not all projects need the same level of consideration.

So LID had one working style baked in: mine. As I started to work on the fix, Claude named it *plural instruments*: more than one way to move forward, and more than one way to check coherence between what you wanted (your intent) and what you got.

### Plural instruments

If you prefer to work one concept at a time, the agent using LID can now ask you one question at a time and build the design from your answers. If you think in documents, it drafts and you can mark it up. You can switch mid-flow.

For checking, the default is still that you review every phase's output. LID also works well when you hand a review phase to a "context-free sub-agent" - a fresh sub-agent that sees only the work to be inspected, not the conversation, and brings its findings back for you to rule on. And if you don't want to read below a certain depth, the `review-depth` experiment in `lid-experimental` lets you say so in your instruction file:

> Review depth: I review through LLD; below that, consolidate to one review. I don't need to see the tests before code.

Everything below still runs (specs, tests, code); the agent will consolidate the decisions and changes into one review instead of stopping at each phase.

Whichever you pick, a sub-agent's findings are shown to you, not applied without your buy-in, and anything that can be read two ways always comes back to you so you can resolve any ambiguity in what you meant.

### What still reaches you

Probably the most important improvement in LID 1.4.0 is that I've built in checks so that the agent will catch subtle ambiguity or misinterpretation more effectively.

I went back through my projects and my conversations with the community for the moments where a human reviewer changed the outcome. It's a short list, and 1.4.0 has it baked in: a spec that forks into two readings; a choice still open; a design working against one of the project's own tenets (the tie-breakers in the HLD); a decision recorded too thinly to reconstruct; a change that's hard to undo. LID makes sure you catch and clarify those yourself.

LID has helped agents get good at this largely thanks to what I found from the experimental `/bidirectional-differential` skill. BiDiff has one blind agent write code from a spec alone (nothing else), and another write the spec from the code (also blind), then compares the outcomes. It works well but it's expensive, so over the summer I tested which part was most able to find intent drift. It was the ability to find alternate readings of atomic specifications of intent. Five of six audited specs had one. So 1.4.0 bakes the cheap version in as a divergence probe: before tests, the agent reads each new spec line a few different ways and brings you only the lines where the readings fork (where there are alternate plausible interpretations).

`/lid-coach` also gained two review dimensions: an existing design in your system that works against one of your tenets, and a decision a cold reader couldn't reconstruct. Same idea, different area - it turns out language tends to be imprecise without clarification!

### Can I use this with model X?

I also found that not every model will catch all of that. I'd been assuming Claude (and the latest, strongest models) with Claude Code, even while LID's setup page lists a dozen other tools. And the world keeps exploding with other, exciting, and often more efficient models that can be used to write code.

Before 1.4.0, LID had no way to find out how those other environments and possible models would handle the judgment LID expects a model to have. The core workflow had no evals, and nothing could run them on a non-Claude model or outside of `claude -p`. So I added both in this release: scenario tests for expected judgment, and a runner that points them at other models through OpenRouter. Thirteen model configurations, ten runs per test project, graded blind - a bit over eight hundred runs.

The result is [a new lookup table inside LID](https://github.com/jszmajda/lid/blob/main/plugins/linked-intent-dev/skills/linked-intent-dev/references/capability-flags.md). Now, your agent finds its model and reasoning effort in one row and reads which kind of judgment LID needs that it's known to miss. A flagged model isn't then prompted harder; it brings that judgment to you: "models like me are known to miss this - please check it yourself." LID now also has a tenet that reads *design for the pair, not the model*.

These are of course about how agents handle LID 1.4.0, nothing more, but today: Claude Sonnet 4.5 and Haiku 4.5 tend to miss tenet conflicts and under-captured decisions; GLM 5.3 Flash tends to miss tenet conflicts; GPT 5.6 Terra misses under-captured decisions. Impressively, Haiku 5.5 caught everything in these tests - same as Opus 5.5 and Sonnet 5.5. Opus 4.8 did too, but offers decision docs too readily, so LID has it say so when it offers one.

So far, the test projects are small and state their conditions plainly, so a pass means "makes this judgment when the docs make it visible." As much as I wanted to, I haven't yet put together a bigger test for a big codebase with tougher conflicts. I needed to get this out the door!

### When to write things down

I think the trickiest call in 1.4.0 was clarifying when the agent should write a *decision doc*: a standalone record of a choice, the options, what we gave up, and why. The simplest rule is "every time you decide something," but it's wrong. If every choice gets a document, you can't tell the hard-won decisions from the thirty-second ones, the intent tree fills up with pages nobody needs, every session burns tokens working through them, and the agent starts to believe the point of LID is to document, when the point is to *build*.

So in 1.4.0 the agent offers a doc when one is warranted: when the choice could reasonably be relitigated by a later reader, with real options traded off against more than one thing and other parts of the design about to be built around it. The rest become a row in the design's decisions table, and we keep moving.

This works because LID enables you to change your mind later. I relied on this myself: right before release I reversed how LID ships its workflow to tools that don't use a plugin system - Codex, Zed, Aider and friends. Instead of copying it into every project at setup, the agent now offers to add it the first time a tool that needs it shows up, so more projects can stay on the upgrade path by default. That was an HLD change. It cascaded through a decision doc, an LLD, a handful of spec lines, the skill, a template, and three evals in an hour. LID's linked IDs make that possible: every spec and test cites the line it came from, so the agent finds everything the change touches without me pointing.

### Everything I didn't ship

Since June, I also opened about forty issues on the repo as I found other places LID needs to improve. The ones closest to this release are the most frustrating not to fix immediately:

- The divergence probe falls back to reading in-context when the harness won't let the agent spawn a blind reader ([#65](https://github.com/jszmajda/lid/issues/65)).
- The same agent writes the tests and the code, so a test can share the code's wrong reading of a spec and go green ([#58](https://github.com/jszmajda/lid/issues/58)).
- Nothing checks that the HLD is faithful to what's in your head ([#41](https://github.com/jszmajda/lid/issues/41)).
- You can let go of review where the risk is low, but you can't mark part of a system high-risk and raise the bar there ([#64](https://github.com/jszmajda/lid/issues/64)).
- The capability table says which model misses what, not which phases deserve which model ([#76](https://github.com/jszmajda/lid/issues/76)).
- The core promise, that the intent is complete enough to regenerate the system, has no test ([#70](https://github.com/jszmajda/lid/issues/70)).
- I say LID's token spend is proportional, and I can't substantiate that ([#75](https://github.com/jszmajda/lid/issues/75)).

I wanted these in 1.4.0, but I pushed them out to get this out the door. It's hard to know when a release is ready to ship, especially when you keep finding more things to fix, but I think this one is ready now. Top of my todo list next though is an onboarding guide ([#84](https://github.com/jszmajda/lid/issues/84)): right now you learn LID by reading the HLD of LID, or having someone teach you. That doesn't scale.

### Meanwhile

All of this took longer than I wanted. In the middle of it we moved from DC to New Jersey - I watched the house I'd lived in for 10 years turn into boxes over photos, from a work offsite - and work got busy in July and hasn't let up. LID went quiet for weeks at a time.

What pulled me back was people: the community growing, people at work using it, and an email in late September from someone I respect deeply in the industry saying he runs LID on all his personal projects now. That was extremely motivating. If you use LID, thank you - I'm deeply moved that it's helping you. I'm excited to hear how!

### Try it

If you're new to LID, in Claude Code you can get started simply:

```
/plugin marketplace add jszmajda/lid
/plugin install linked-intent-dev@jszmajda-lid
/plugin install arrow-maintenance@jszmajda-lid
```

For other harnesses, point your agent at [https://github.com/jszmajda/lid/blob/main/docs/setup.md](https://github.com/jszmajda/lid/blob/main/docs/setup.md) and it'll figure out the best path from there.

Already on LID 1.3? Update the plugin and run `/update-lid` in each project; it walks you through what changed. Full notes are in the [release](https://github.com/jszmajda/lid/releases/tag/v1.4.0), and the eval tooling is under `tools/alt-model-evals/` if you want to run a model I haven't.

I'm excited for whatever comes back: a field report, an issue, a pull request, a "this stop annoyed me." If you use LID from Cursor, Windsurf, Copilot, Aider or Codex, there's an [open issue asking how it went](https://github.com/jszmajda/lid/issues?q=is%3Aissue+label%3Apending-feedback) for each, mostly unanswered. Contributions of every size are welcome, and so is telling me where LID slowed you down.
