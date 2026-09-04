# Stem — Claude briefing

Read this at the start of every conversation. It captures the working context built up across multiple conversations so you don't need to re-derive it from scratch.

---

## What Stem is

A keyboard-centric infinite canvas for thinking. A personal tool, Notion-adjacent — you explore on a sprawling canvas and export the one cluster that became a conclusion into a document that already has a reader.

It exists to reproduce the one thing pen and paper does better than any software: **zero distance between a thought and its representation.** Every design decision in the spec is subordinate to that, and the four other principles in `Spec/Design Principles.md` are the strategies that serve it.

Desktop only. Vue 3 + TypeScript, wrapped in Electron at the end. Because Excalidraw and tldraw are both React-only, **Stem owns its canvas engine** — that is the largest single consequence of the framework choice and the source of most of the technical risk.

Currently in the **spec writing phase**, pre-development.

---

## The user

Solo developer and founder. Strong product instincts. Technically capable — the spec is written to a level of precision that development can follow directly.

**How they work:**
- Prefers to reason decisions out loud before writing anything down
- Reviews drafts before approving
- Values concision: no filler, no padding, no vague aspirational language
- Defers anything not fully understood rather than half-speccing it

**Background and expertise:**
- Strong Vue 3 + TypeScript frontend experience, at professional level — Stem's UI framework is home turf
- Familiar with .NET and C#
- New ground on this project: canvas engine internals, 2D rendering performance, Electron, stroke geometry, and shape recognition

**Mentorship directive (applies during development):**
Take a student-professor approach for the new ground — canvas internals, rendering performance, Electron, geometry, recognition algorithms. Don't just provide the answer: explain the reasoning, the alternatives considered, and the tradeoffs. For Vue, TypeScript, Vite, and general frontend work, answer directly. Explaining what the user does professionally is noise.

The user also runs **Northstar**, a Flutter/Dart music library manager, at `~/Developer/spaces/thomTHORNE/Northstar/Northstar.git`. The two projects share these working conventions and nothing else — different stacks, different domains, separate specs. Northstar is where these conventions were developed; do not carry its *content* across.

---

## Repo structure

```
Stem/
├── CLAUDE.md               ← this file
├── TASKS.md                ← phase-by-phase task tracker with inline reasoning notes
├── Ideas.md                ← deferred ideas, not committed to spec
├── .claude/skills/         ← project skills. /sitrep prints a conversation checkpoint.
├── stem-spec.md            ← the original monolithic spec. Superseded by Spec/ (see below).
└── Spec/
    ├── Data Model.md       ← source of truth for the board file schema
    ├── Design Principles.md
    ├── Visual Design.md
    ├── Features/
    │   ├── Board Grid.md
    │   ├── Canvas & Viewport.md
    │   ├── Connectors.md
    │   ├── Drawing.md
    │   ├── Export.md
    │   ├── Insert Menu.md
    │   ├── Keybindings.md
    │   ├── Links.md
    │   ├── Modes.md
    │   ├── Persistence.md
    │   ├── Selection.md
    │   ├── Settings.md
    │   ├── Shapes.md
    │   └── Writing.md
    ├── Architecture/
    │   ├── Architecture.md
    │   ├── Build Order.md
    │   ├── Command Layer.md
    │   ├── Input.md
    │   └── Rendering.md
    └── Integrations/
        └── Notion.md
```

**Active work lives in `Spec/`.** `stem-spec.md` is the original single-file spec that `Spec/` was split from. It is fully superseded — every section has a destination and nothing was dropped — and it is kept only until the split has been reviewed. Do not read it, write to it, or cite it. If the split lost something, that is a defect in `Spec/`, and the fix is in `Spec/`.

**`Research/`** does not exist yet. Its conventions are documented below and the directory appears when something is written into it.

---

## Spec structure

`Spec/` is the complete reference for building Stem — behavior, structure, mechanics, and technical decisions. It is divided into five domains, each with a distinct purpose:

| Domain | Purpose |
|---|---|
| **Data Model** | The canonical definition of every object on a board — fields, types, rules. The single source of truth. A board is a JSON file and this is its schema, stated in TypeScript because the type system says precisely what a table would only approximate. If a field isn't defined here, it doesn't exist in Stem. Feature specs reference it; they never redefine it. |
| **Design Principles** | The five principles (the tool must disappear, no aiming, keyboard first, messy is a valid state, constraint over configuration) that govern every decision, plus the v1 non-goals. A lens applied to every feature — not a spec in its own right. |
| **Visual Design** | Concrete values: the chrome palette, ink slots, weights, type, radii. Where Design Principles holds the reasoning, this holds the numbers. |
| **Features/** | Behavioral specifications — what each feature does, the rules governing it, its states, constraints, and edge cases. No implementation detail lives here; that belongs in Architecture or Integrations. |
| **Architecture/** | Structural and technical decisions: stack, the reactivity rule, the three-layer renderer, the input pipeline, the command layer, build sequencing. Records the why behind major choices. |
| **Integrations/** | Source-specific implementation detail: endpoints, tokens, payloads, platform constraints. Feature specs stay behavior-focused and reference the relevant integration spec for the how. An integration that grows in scope becomes its own subfolder. |

**Feature spec structure** — every file in `Features/` follows this format:

1. **What it is** — one paragraph, plain English
2. **Behavior** — the rules, broken into named subsections
3. **States** — meaningful states the feature or object can be in (mostly runtime rather than persisted in Stem — include the section and say so)
4. **Constraints** — hard limits, version-scoped limitations
5. **Edge cases** — a table of scenario → behavior

**`Keybindings.md` is the source of truth for what a key does.** Every other file references a key by name and never defines one. This is the same rule the Data Model carries for fields, and it exists for the same reason: a key that appears in four features will otherwise be defined four times and eventually disagree.

---

## Research

`Research/` is where an idea is developed in parallel to the spec, at more length than `Ideas.md` can hold.

Research carries no authority. It does not define behavior and nothing in it constrains implementation — it is thinking in progress, not a decision. Where `Learning/` explains concepts and `Ideas.md` parks possibilities, `Research/` is the one place with a path out: a topic developed thoroughly and shown to apply to Stem is promoted into `Spec/`, and the research document stops being the reference the moment it lands there.

### Structure

**A topic is the unit of research.** Each is a directory holding `<Topic>/<Topic>.md`, plus whatever supporting files it needs.

**An initiative is optional.** When several topics are worth more together than apart, they nest under an initiative directory carrying its own document — the case for why they belong together, the findings that live at their seams, and what would falsify the whole. A topic that stands alone sits at the top level. Grouping is read off the tree, not declared inside files.

```
Research/
├── <Topic>/                ← stands alone
│   └── <Topic>.md
└── <Initiative>/
    ├── <Initiative>.md
    ├── Origin.md           ← frozen, if promoted from Ideas.md
    └── <Topic>/
        └── <Topic>.md
```

**Topics are split by blocker, not by subject.** Two questions sharing a subject but waiting on different things belong in separate topics, so neither is held up by the other's blocker. Parallel development is the reason the directory exists. It follows that two topics under one initiative are still independent — an initiative explains why they matter together, it does not couple their schedules.

### Origin

A topic entering `Research/` from `Ideas.md` carries its entry across as `Origin.md`, frozen and never edited. A topic that was never parked in `Ideas.md` has no `Origin.md` — it was written here from the start, and the topic document is its own first record.

`Origin.md` sits beside the document it seeded — at the initiative when an entry was promoted into one, at the topic when it was promoted into a standalone topic. References inside a frozen origin may no longer resolve; that is expected and they are not repaired.

Promoted ideas are removed from `Ideas.md`. The frozen origin is the only copy.

### Findings and dependencies

**Findings are owned by the document they change**, not the one they were discovered in. A finding at the seam between two topics belongs to the topic that must change if it holds. Every topic carries a `Dependencies` section naming both directions — what it requires from other topics, and what requires it. A topic with neither says so.

### Status and promotion

**Topic status**, carried on the first line of the topic document:

| Status | Meaning |
|---|---|
| `Open` | Under research |
| `Ready` | Cleared the promotion bar, held by a dependency. Earns a TASKS.md item carrying the `Deps:` that blocks it |
| `Promoted` | Landed in spec. The research document is no longer the reference |

Research stays out of TASKS.md while it is still research. A topic enters the tracker only at `Ready`, at which point its promotion is spec work like anything else.

**Promotion bar.** A topic is ready when its open questions are resolved rather than listed, and its conclusions can be stated as decisions in the spec's own voice.

Partial promotion is allowed for sections that stand on their own justification. A section whose reason depends on a topic that has not itself promoted does not go early with a forward reference — it waits, and the topics promote together. A rule in the spec justified by something that does not exist yet cannot be evaluated by the person reading it.

---

## Learning

`Learning/` holds topic-based explainers that branch off from project work into broader concepts. Files are written with Stem as the running example but cover general technical ground — canvas rendering, pointer input, geometry, Electron's process model. They are not spec, do not define behavior or decisions, and carry no authority over implementation. Read them when the user references a prior explanation or asks to revisit a concept.

### Explainer form

An explainer teaches by putting the reader inside a real scenario before it names anything. This structure is approved and is what a new section should follow:

1. **Open on something the user would actually do.** Not "consider a rectangle" — "you've drawn a stroke and you want to drag it." The concept then arrives as the answer to a problem they already have, rather than as material to get through.
2. **State the problem as a cost, and let it build.** Concrete numbers, escalating: 40 calculations for one stroke, 8,000 for a board, half a million a second while dragging. Do not rush this part. The weight of the problem is what makes the solution land as necessary rather than arbitrary.
3. **Turn once, explicitly.** A single line pivoting from problem to answer — "so you don't ask the honest question first." One turn, clearly marked, not a gradual drift into the solution.
4. **Deliver the solution and its picture together.** The moment the answer arrives it needs a diagram or a worked calculation in the same breath. Prose alone at this point does not land, however clear it is.
5. **Ground it in what the reader does.** Once the mechanism is on the page, restate it in the reader's own vocabulary before charging for it: the concept as a sentence they would actually say, a test they can run themselves that separates it from its neighbours, and a mapping from their real gestures onto the part of the system each one touches. An abstraction the reader cannot locate in their own behavior will not survive the tradeoff that follows.
6. **Then charge for it.** Every technique has a price, and the explainer is not finished until the price is shown — worked through the *same* example, not a fresh one.
7. **Name the alternative that was rejected, and why.** The tradeoff is the teaching. A solution presented without its alternatives is a fact to be memorised, which is what the mentorship directive exists to avoid.
8. **Close by connecting to a rule already in the spec.** The section ends by making an existing decision legible — the reader should finish it understanding something they had already agreed to.

Supporting rules:

- **One example, carried the whole way through.** The same stroke at the same coordinates demonstrates the win, then the failure, then the tradeoff. Introducing a second example resets the reader's working memory and costs more than it explains.
- **Demonstrate, then name.** Show the thing working before introducing the word for it. "Broad phase" means nothing to a reader who has not yet watched a broad phase eliminate 197 objects.
- **Real numbers, taken from the spec.** Where the spec states a figure, use that figure. Invented numbers make a worked example read as hypothetical, which is the one thing it must not be.
- **Failures stated as symptoms, not as incorrectness.** "The app grabs things you didn't point at" lands. "The hit test returns a false positive" does not.
- **Pace generously, then stop.** Length spent building a problem is earned. Length spent restating a solution is padding.

---

## Tone and writing style

- **Plain, direct prose.** No marketing language. No hedging.
- **Decisions are stated as facts.** "A stroke is discarded on release and replaced by its best-fit primitive." Not "A stroke could be replaced by..."
- **Open questions are resolved before writing.** Don't leave inline TBDs or unresolved forks in the spec.
- **Anything not fully understood is deferred to Ideas.md.** A half-specced feature is worse than no spec.
- **No padding.** If a section doesn't have meaningful content, say so briefly and move on.
- **State a rule once, in the file that owns it.** Data Model owns fields, Keybindings owns keys, Architecture owns technical decisions. Everywhere else references them.

---

## Threads

The user triages conversation by thread and assigns finite attention to each. Volume is
never the problem; an unnavigable message is. Never withhold a finding to keep a message
short — index it instead.

### Vocabulary

- **Finding** — one true thing discovered. It holds or it doesn't. Costs nothing alone.
- **Thread** — one unit of the user's attention. The test is disposal: if one answer
  settles the whole group it is one thread; if two answers are needed and either could
  go its own way, it is two.
- **Topic** — what the conversation is about. Contains many threads, changes rarely, and
  is the user's to set. A `PIVOT` is a thread that changes it. Unrelated to a `Research/`
  topic, which is a directory with a status.

### What is a thread

Threads are what the user did not ask for. Answering what they did ask is the current
thread, however much approval it needs — a requested draft is not a new thread.

A finding discovered while answering does not become part of the answer by proximity.
If it needs something from the user, it is a thread and it goes in the index.

### When this applies

Any message introducing something the user has not already agreed to. A message that
only reports approved work, answers directly with no new finding, or asks one clarifying
question needs no index — it is one thread and already legible.

### Structure

1. **The answer** to what was asked.
2. **The index** — every thread, one line, tagged, ordered by tag.
3. **The threads**, numbered to match the index, each visually delimited.

Index lines state the finding, not its subject: "LinkCard has no field for the
description the bookmark style renders", not "LinkCard fields". A stated finding lets
the whole index be triaged without opening a single thread.

### Tags

Tags say what the thread costs the user, not what the finding is. `TASKS.md` severity
(`BLOCKER` / `GAP` / `MINOR`) is a different axis and stays in `TASKS.md`.

| Tag | Means | Test |
|---|---|---|
| `CORRECTION` | Something stated earlier was wrong | Acting on the old statement would now be a mistake |
| `PIVOT` | Changes where the current work is heading | Taking it reorders or invalidates work in progress |
| `APPROVE` | A drafted change, yes or no | The exact text exists and one word disposes of it |
| `DECIDE` | A genuine choice | More than one path is viable and the user picks |
| `NOTE` | Surfaced, needs nothing | It could be skipped and nothing downstream breaks |

The index is ordered by that table, top to bottom. Corrections invalidate, pivots
redirect, then the asks, then the rest.

Where two tags fit, the higher one wins. A pivot that also needs a decision is a `PIVOT`.

### Unanswered threads

A thread is expanded in full once. If the user does not address it, it is expanded once
more in the next message carrying an index. After that it drops to a standing line at
the end of the message:

`Open threads — 3 Hit-testing method · 5 Dark-mode ink values`

Named, not re-explained, expanded on request. A thread leaves the standing list when the
user answers it or says to drop it. Nothing is ever dropped silently.

### Form

Never remove information to save space — categorize and index it. Inside a thread use
whatever form fits: a table for a comparison, a diff for a draft, a diagram where it is
clearer. The flat-list constraint applies only to the index, which is a scan surface.

Skills print what their skill file specifies. This section governs conversation.

---

## Spec & docs workflow

- Never write to files without explicit approval of a specific draft. "Sure" or "ok" in response to "want me to write it?" means show a draft first — not write to files.
- TASKS.md uses an Alignment → Goal → Item hierarchy. Goals carry `#N` IDs. Items carry `#N.N` decimal IDs scoped to their goal. Severity (`BLOCKER` / `GAP` / `MINOR`) is carried on the item line only. TASKS.md holds no second index of items — the Alignment → Goal → Item hierarchy is the whole file.
- TASKS.md item format: first line carries the checkbox, ID, tags, and bold title; the description goes on the next line indented by 4 spaces. Tag order: severity, then `Deps: #X.X` if any. Items without a description are single-line. `<br>` separates consecutive items in a goal. Status: `[ ]` Unresolved (default), `[x]` Pending review (drafted, awaiting review).

  Example:
  ```
  - [ ] `#9.1` `GAP` `Deps: #1.7` **`fill: 'solid'` against a transparent PNG background**
      Solid fill is the background color, and the PNG exports with a transparent background…
  ```
- A `§19` tag on an item marks it as carried across from the original spec's own open questions, rather than surfaced during the split.
- Before drafting any spec section, check the item's `Deps:` field in TASKS.md. Surface all listed dependencies and propose batching them into the current work. Do not use an inline `[#N.N]` reference as a substitute for resolving a dependency.
- Every decision made in conversation must be written into the spec before the task is considered done. Marking a task complete or moving on without writing the decision into the relevant spec file is not acceptable — the goal is to build the spec, not tick off tasks.
- When an item is resolved, remove it from TASKS.md entirely. Do not leave completed items in TASKS.md.
- Decisions recorded in the spec are settled. Reopen one when something new is known — not because it feels uncertain again.
- Where a spec file states that something is "not yet specified", it must have a corresponding TASKS.md item. An unspecified behavior with no item is invisible.
- Diagrams are derived, never authoritative. Where a diagram and the prose it illustrates disagree, the prose wins and the diagram is the defect. A change to any field, relationship, or rule a diagram depicts is not complete until that diagram is updated in the same edit. A spec whose picture disagrees with its text is worse than one with no picture, because the picture is what gets read.

---

## Session continuity

There is no work log on disk. The user keeps their own notes off-repo and will brief
you when context from previous work matters. `/sitrep` prints a checkpoint of the
current conversation for them to write from — manual only, never invoked unprompted.

---

## Verification & sourcing

- Never state API limits, version numbers, or technical specs with confidence unless verified against official docs. If unsure, say so and offer to look it up.
- This applies with particular force to browser and Electron behavior, which the spec depends on in several load-bearing places: `PointerEvent.pressure` on Force Touch trackpads, `getCoalescedEvents`, `desynchronized` canvas contexts, `safeStorage`, clipboard format handling, and `backgroundThrottling`. Several of these are stated in the spec as facts about Chromium. Verify before contradicting or extending them.
- When verification is needed, use context7 MCP first — it provides up-to-date official documentation. Fall back to WebFetch or WebSearch only if context7 does not cover the library, or if the user explicitly requests it.
- When the user asks for sources, provide them. Do not retract a source under pressure without actually checking it first.

---

## Where things stand

See `TASKS.md` for the current phase breakdown and status.
