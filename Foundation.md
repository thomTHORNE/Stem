# Foundation

Stem keeps the whole path of a thought, not just where it ended up.

Stem is a personal tool for thinking. You put a thought down the moment you have it, in whatever shape it arrives — a sketch, a sentence, a half-formed question — and Stem keeps it in that shape. When a thought grows, Stem gives it a structure to grow into. When you need it again, you ask for it. And because the whole path is kept, you can look back at how you got somewhere, not only at where you arrived.

This document is the reason Stem is shaped the way it is. The design principles, the features, and the technical decisions all answer to it. Where another document disagrees with this one, this one holds and the other is the defect.

---

## The problem

Thinking is not tidy. It moves by hunches, doubts, false starts, and returns. Most of what it produces is unfinished, and most of what is unfinished is still useful.

Tools for knowledge work are built for the opposite. They ask where a note belongs before it is written, reward the finished document, and treat everything before it as a draft to clean up or throw away. The price is paid twice. Once in the moment: a thought you stop to file is a thought you are no longer holding. And again later: the path to a conclusion — the doubts, the dead ends, the backtracks — is the part that was tidied away, and it is the part that would have shown you how you think.

Filing also costs something that grows. Every note carries a small obligation: name it, place it, keep the structure around it current. That administration grows with the archive until it is what you are maintaining instead of thinking.

AI adds a new pressure. A machine can now produce fluent, confident text about anything, on demand. Set beside your own rough thinking, it reads as more finished, and finished reads as more correct. The risk is not that the machine is wrong. It is that you gradually lose track of which thoughts were yours — and a thought you did not have is one you cannot build on, defend, or learn from.

---

## Ownership

Your thinking is the most important thing in Stem. Every other part of the system — search, AI, cognition blocks, integrations — exists to serve it and never competes with it for significance.

**Original means written in Stem.** A thought is yours when it was written or drawn in Stem, and moving it anywhere within Stem keeps it yours. Everything that arrives from elsewhere is a reference: an AI response, a pasted paragraph, content shown through an integration. That includes your own writing pasted from another tool. Stem cannot verify where outside text came from, so it does not try. Original is a fact Stem knows, never a claim anyone makes.

**References are quoted, not edited.** A reference is read-only. To make its content yours, you restate it in your own words, and the restatement is original. There is no other way for a reference to become yours.

**AI responds; it does not author.** AI helps you recall what you have written and lets you talk a topic through. Its words always read as responses. The parts of Stem that carry significance, priority, and action belong to you, and nothing in how AI output looks or behaves borrows from them. AI content cannot be mistaken for yours, by design or by accident.

**Nothing leaves without consent.** Stem sends none of your thinking anywhere on its own. Anything sent to a cloud model or an integration goes only with your informed consent, and you know exactly what is sent.

---

## Messiness is honest

A thought rarely arrives finished. It arrives as a hunch you cannot justify yet, a sketch that is half wrong, a question you do not know how to answer. Stopping to tidy it — to phrase it properly, to decide where it goes — costs the thought itself. Self-editing at the moment of capture is the friction that loses most of what a person thinks.

The tool can add the same friction. Every choice the software asks for, every small target it makes you aim at, is attention taken from the thought you were holding.

The untidy parts are not waste either. A doubt, a dead end, a pivot and a backtrack are how understanding is actually reached. Feeling lost is part of the work, and the record of being lost is information: it shows where a problem was hard and how you got through it.

So Stem accepts a thought in whatever state it arrives and keeps it in that state. Follow the intuition first; whether it was right can be decided later. Cleanup exists, and it is always something you ask for, never something done to you.

Stem never tidies, files, or corrects on its own, and never requires a thought to be finished, named, or placed before it can exist. Nothing is discarded unless you discard it.

---

## Thinking about thinking

Most tools store what you found. How you found it — which questions you return to, where you stall, what you abandon — is lost, and with it the chance to understand how your own mind works.

Stem gives that understanding a place. Over the base where you normally work sits a second layer, like tracing paper laid over a drawing: the work stays visible underneath, and the layer is where you look at it from one step back. It is a space for self-awareness to develop on its own terms and at its own pace, not a report delivered to you.

The ownership rules hold here as everywhere else. Anything Stem or an AI contributes to this layer reads as a response.

The meta layer never changes the work beneath it, and nothing on it is presented as a conclusion about you. Conclusions are yours to draw.

---

## Recollecting instead of filing

People do not remember where they put a thought. They remember what it was about, what it was near, what they were stuck on at the time. A system that can only find what was filed correctly asks you to predict, at the moment of writing, how you will look for it months later — and the prediction is usually wrong.

Stem does not ask you to predict. Everything you put down is indexed — freeform drawing and writing, diagrams, markdown, cognition blocks, integrations — and you find it by asking for it in your own words. Results are ranked by what they mean, not by whether they contain the word you happened to type.

This is what makes the rest affordable. Thoughts can stay messy and unfiled because finding them never depended on filing.

Stem never requires a thought to be named, tagged, or placed in order to be found. A result is always your thought itself, shown where it lives. An AI may help you get to it; it never stands in for it.

---

## Growing with the thought

Thoughts grow. A tool that suits the first minute of an idea — a blank surface, nothing to set up — is wrong for its third week, and a tool built for the third week is too heavy for the first minute. Growth also repeats: people approach the same kinds of problems in the same way, and redrawing the same structure every time is administration, not thinking.

Stem starts with the board. Freeform drawing and writing are immediate and need nothing decided first. When a thought starts developing along a pattern you recognise — an idea taking shape, a subject being learned, an action being carried out — a cognition block gives it the structure that pattern needs: sections with distinct purposes, interactive pieces where they help, and a visual design built for that kind of thinking. Integrations extend the same reach to outside systems, brought into Stem as blocks.

The board is where you spend most of your time, so it stays constrained: few choices, nothing to configure. Configuration lives off the board, where cognition types are defined and refined, and a defined type is ready to place on any board at once.

Stem never asks you to choose a structure before you start, never moves a thought into a block on its own, and never brings configuration onto the board.

---

## Terms

**Thought** — anything you put into Stem, at any size: a stroke, a sentence, a question. The unit everything else is built from.

**Cognition type** — Stem's named shape for a recurring way of working on something: its sections and its visual design. A type is defined off the board and has one definition. Changing it changes the type; a variation is a new type with its own name.

**Idea · Learning · Action** — the first cognition types: developing something new, understanding something that exists, carrying something out.

**Cognition block** — an instance of a cognition type on a board, holding real thoughts.

**Integration** — a block whose content lives in an outside system.

**Board** — the infinite canvas a thought is put down on, as the spec defines it.

**Freeform** — drawing and writing directly on the board, outside any block.
