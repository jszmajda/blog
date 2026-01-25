---
layout: post
title: The Arrow of Intent
---

This post is about improving agent intent alignment using what I call "the arrow of intent"—a mental model I've developed over a year of building [Threadkeeper](https://threadkeeper.app) with Claude Code.

Modern AI coding agents aren't really writing bugs anymore. What they do write are *intent gaps*—places where they assumed I meant something different than I did. With Claude Opus 4.5 (and I imagine the other frontier models), the biggest problem I've found to solve when building complex systems is how to create, clarify, and maintain intent as a system grows and changes. I've landed on a mental model that's helped.

## Scale Creates Complexity

To ground this in something concrete: Threadkeeper is a complex system now. It's ~275,000 lines of code across 1,500 files—roughly a $10M, 27-person project by traditional estimates. I also have something on the order of ~2500 tests across unit and integration tests throughout the system.

Last December I added an import feature. It worked—technically. But after selecting a date, the app took me back to the entries list. No spinner, no confirmation. I had to check the database to know if it had even started. The code did what I'd specified. The problem was that my specification was incomplete—I'd forgotten to say "and tell the user what's happening." This kind of gap is what I keep hitting.

## How I've Tried to Maintain Intent

I long believed that tests were the best way to create and maintain what I think of as "the arrow of intent"—a unidirectional flow from what you *want* the system to do, through specifications, to tests, to code. Tests define what the system should do, and directly validate that behavior. But I've struggled with how to evolve intent. Before AI, I tried several approaches:

**Tribal knowledge** is the default—have one or two people who truly grok the system, know what it should do, and know where it's deviating from that intent. Longevity and memory capacity in these individuals was highly prized. Usually this was me, but as I moved between projects more frequently, it started to become others. This splitting of me from this function led me to understand that it was insufficient on its own. In big tech companies, where turnover is a continuous geological force, tribal knowledge doesn't scale.

**Documentation** promises to onboard anyone—what is the system overall, what are its major components, how do they work together. But it rots faster than you can maintain it. Most often, we'd change things way faster than we could keep up with documentation. At best this was useful in 0-1 phases (the initial build), where it tended to be _invaluable_, but it quickly went out of date.

**End-to-end tests** validate real flows and break loudly when things change. I usually write one or two of these when I build a new thing—they exercise the entire happy-path flow of the system, from user login through to a completion task. But they're slow, which creates pressure to add mocks. Any mock you introduce gives you speed but also guarantees that your tests are lying to you. Mocks in E2E tests are particularly toxic.

**BDD tests** ("Behavior-Driven Development," like [Gherkin](https://en.wikipedia.org/wiki/Cucumber_(software))) make specs readable by non-engineers, enabling conversations about how the system varies from expectations. But you still need someone who understands overlapping expectations across multiple tests to spot when intent drifts (e.g. Tribal Knowledge). BDD tests also attract mocks and over time deviate from the truth.

**Canaries** (or _Synthetic Test Monitoring_—see [this post from Dynatrace](https://www.dynatrace.com/news/blog/what-is-synthetic-testing/) or [Cloudwatch's tooling](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)) continuously exercise production with synthetic users, running through all aspects of the system every few minutes. We used them heavily at AWS—they're the best I've found. Most operational failures at AWS included expanding the system's Canary test suite to cover that failure and ensure it never happens again. Because they run in production, they tend _not_ to attract mocks—the mental model is different. The tradeoff: they're expensive. Continuous validation costs compute and IO.

**Modularization**—breaking a system into subcomponents or microservices—helps reduce complexity for any given part. But it still has the problem of maintaining intent across the whole. Paradoxically, it often makes overall understanding *worse*: now you have multiple systems to grok, with implicit contracts between them. And tests that cross subsystem boundaries—the ones that validate the combined system's intent—become harder to write and maintain.

So—no clear answers, continuous tradeoffs, and battling tech debt over time.

## Agentic Coding Magnifies the Intent Problem

With agentic coding, the intent maintenance problem is magnified. Agentic coding is best thought of as throwing a new very capable programmer at your software every time you boot up a new session. Things like CLAUDE.md / AGENTS.md provide context and can help reduce context loss from session to session, but they're never quite that "one engineer who knows the system" yet. There's a huge gap between what's in my brain and what's in the agent's context window.

I imagine as agent memory systems improve this will become less of an issue, but even with that you have context loss as you change agent systems, or as they evolve. This is one of the long-term problems of creating and maintaining complex systems.

## What I've Tried with AI

Over the year I've been working on Threadkeeper, I've been trying to work out what I could do to reduce that gap from what's in my head to what's in the agent context window. I've come to think of this as intent maintenance. I've tried a few things:

1. I started working on Threadkeeper in the same way as I built other 0-1 projects at AWS—with a High Level Design (HLD) first, then Low Level Designs (LLDs) for the major components of the system. This worked pretty well to help me and Claude come to an agreement on what we should build.
2. We'd then move to TDD (Test-Driven Development), which worked pretty well for getting things off the ground. TDD guaranteed that the intent was manifested in code, and could be maintained as code was written to fulfill it.
3. I tried adding BDD systems, but I found that they didn't really add much value as Claude and I could both read the tests themselves a bit more quickly, and separating test code from test spec (e.g. the Gherkin from step definitions) only led to the agent struggling more as it needed to load in more files (and know to do that..)

Mostly this worked for quite a bit, but over time I started to struggle more with making change. As I would change my intent for the system, I'd generally write _new_ LLDs, archive the old ones, and write new tests. This would cause old tests to break, so I'd have to go through and adjust or remove them where they were no longer correct. Not optimal.

## EARS and Spec-Driven Development

Eventually I heard of what the [AWS Kiro](https://kiro.dev/) team was doing with Spec-Driven development. Most interestingly I heard that Kiro used [EARS](https://alistairmavin.com/ears/) ("Easy Approach to Requirements Syntax") specs. EARS are a simple way to transform intent into one-line requirement statements (easily grep-able!), and most interestingly I could add meaningful unique serial ids to each requirement. This meant that I could annotate code and tests with that requirement ID, providing the agent hints as to why something was there. E.g:

```
// in docs/specs/import-specs.md:
IMPORT-002: When a user initiates an import, the system shall navigate to the entry detail screen and display upload progress.
```

```typescript
// in tests:
// @spec IMPORT-002
it('navigates to entry detail and shows progress during import') { ... }

// later, in code:
// @spec IMPORT-002
async function handleImportSelected(date: Date) { ... }
```

I've been storing my EARS specs in a spec 'registry' of sorts, in `docs/specs/<subsystem>-specs.md`. Each spec file in there has a header that shows what LLD it's related to, so I or the agent can go through and read the expectations of that LLD and how they formed the specs.

This helped a fair bit, as I could tie things together, but it still didn't quite get to what I was trying to solve. The agent would still make mistakes. I had the components; I just hadn't figured out how they should connect.

## The Arrow of Intent

I left it like this for several months, trying to work out what to do instead. As I was preparing a training on agentic coding though, I was thinking about it again and realized that there's a fundamental _arrow of intent_, where implementation should flow unidirectionally from intent to code, and that I wasn't thinking about that as a first-class problem.

Here's what I'm doing now. Note that this is still experimental, but so far it seems to be helping:

The arrow of intent is a mental model for how systems are formed. Code is a representation of intent, and is only valid when it maintains the total intent of the system. Intent is an arrow:

`HLD -> LLDs -> Tests -> Code`

All change to the system _must_ originate from change upstream. Usually the HLD doesn't change as it's often quite broad, but often the LLDs change. EARS specs change frequently, and then tests and code change to fit the new specs.

I've built a Skill to help me build and maintain the arrow for my project—feel free to check [my "design-driven-dev" skill](https://github.com/jszmajda/how_i_claude/tree/main/skills/design-driven-dev) out for yourself—and that seems to help, but Claude doesn't always trigger the Skill. I've added notes to my project's CLAUDE.md as well:

````markdown
## Development Best Practices

### Design-Driven Development (MANDATORY)

**Consult the `design-driven-dev` skill for ALL code changes.** The skill lives at `.claude/skills/design-driven-dev/`.

**All changes to the system must start with intent.** The arrow of intent flows in one direction:

```
HLD → LLDs → EARS → Tests → Code
```

Changes emanate from intent—docs first, then tests, then code. Never the reverse.

- **New features**: Generate intent first. Create or update docs (HLD → LLD → EARS), then write tests that capture that intent, then implement code
- **Bug fixes**: Clarify, adjust, or add intent so the bug is clearly a bug based on the arrow of intent. If existing specs don't make the bug obvious, update docs first to capture the correct behavior, then fix tests if needed, then fix code

When requirements change, cascade updates downward through the chain. Mutation, not accumulation—docs should always reflect current intent.
````

This is helping, but I still often find that I have to remind Claude in Plan mode to "maintain the arrow of intent."

### Example: Fixing a Gap with the Arrow

To make this practical, let's revisit that import feature. Say I'd had the arrow back in December. Instead of diving into code after noticing the problem, I'd trace back up:

1. **Check the HLD:** Does it mention import feedback? Probably not—HLDs are broad.
2. **Check the LLD:** Does the import LLD specify what happens after the user selects a date? It says "import the file"—but nothing about navigation or progress.
3. **Check the specs:** Is there an EARS spec covering user feedback during import? No.

So the "bug" isn't in the code—it's upstream. I fix it by flowing down the arrow:

1. **Update the LLD:** Add a section clarifying that import should navigate to the entry detail screen and show progress.
2. **Add the spec:** `IMPORT-002: When a user initiates an import, the system shall navigate to the entry detail screen and display upload progress.`
3. **Write the test:** A test tagged `@spec IMPORT-002` that validates the navigation and progress indicator. It fails.
4. **Fix the code:** Red-green-refactor until the test passes.

I don't ask Claude to "fix the bug." I ask Claude to "update the arrow of intent so that import feedback is clearly specified, tested, and implemented."

More often, the problem is that the intent wasn't sufficiently clear. It's most likely I forgot to say that the system should do this, or _how_ it should do it, rather than there being a clear software bug.

## Where I Am Now

Overall, this gives me much more of what I want—a reflection of my intent through documentation that's always accurate, tests that add value, and code that meets my needs. Where I have problems in my build now, they're either where I haven't specified clearly enough all the behavior I expect (e.g. I've made a mental leap and not broken it down properly), or where I made an assumption about the system where Claude's assumption differed. Sometimes it's through context loss—e.g. across compaction windows nuance gets missed. This is usually resolvable by adjusting and clarifying the Arrow though.

Here's what I've realized: I don't care about the code itself like I used to. I used to spend energy on well-factored, well-written code—not just because it worked better, but because code was how I communicated intent to my future self and to other engineers. Clean code was a form of documentation.

That's changed. Now the intent documentation is the documentation. Code has become output—something the agent produces—not the artifact I craft and maintain.

This is probably what we should have been doing all along. But translating intent into code took so much energy that code became the primary object of attention. That's flipping. We're moving into an SDLC where humans specify intent and agents produce code. The arrow of intent is about recognizing that shift.

My wife put it well: we're not writing code anymore, we're producing software.

I'm sure I'll refine this as I keep going. But the core idea has held up: intent flows in one direction—from design to specs to tests to code. When something breaks, I don't just fix code—I specify the behavior I want, then ask Claude to adjust the arrow of intent: fix the docs, fix the specs, fix the tests, fix the code. The arrow always flows downstream. The agent isn't the one who "knows" my system; the arrow is. And maintaining it has made the difference between fighting the AI and collaborating with it.
