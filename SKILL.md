---
name: tdd-context-split
description:
  Test-first workflow in which the test author and the implementer do NOT share a context, only the
  spec. Use before any behaviour change - feature, bugfix, or a refactor that alters observable
  behaviour. Covers the Given/When/Then spec format, dispatching a fresh-context test author, red
  for the predicted reason, and mutation proof. Not for spikes, debugging, or tests written for code
  that already exists.
license: GPL-3.0-or-later
---

# Context-split TDD

The test author and the implementer must not share a reality. They share one artifact: the spec.

**Core principle:** an implementer cannot change perspective on code they are about to write. Their
tests encode the intended implementation instead of the contract - they assert that the new
behaviour appeared, and never that the old guarantee survived. That gap passes a full green suite
and CI.

This is not the usual red-green-refactor skill. Red-green-refactor is about ORDER. This is about
ISOLATION: order alone does not help when the same context holds both the plan and the test.

## The invariant

> The context that authors the tests has not seen the implementation plan.

State the boundary precisely, and do not overclaim it:

| Hidden from the test author | Visible to the test author       |
| --------------------------- | -------------------------------- |
| the implementation plan     | the spec                         |
| your reasoning about it     | the code as it stands today      |
| the handoff / plan document | the existing test module in FULL |

For a modification the current implementation is in the repository and always will be. What the
split buys is that the test cannot encode the INTENDED one. Do not claim the test author is "blind
to the implementation"; claim only what is true.

## When to use

- Any behaviour change: feature, bugfix, or a refactor that alters observable behaviour.
- Empirical experiments too. For a question about how an external system behaves, the same split
  applies: a separate context authors the experiment set (what to vary, what to observe, what result
  falsifies what). Improvising your own probes and then calling the result verified is the identical
  failure mode - you sample where you already expect agreement.

## When NOT to use

- Exploratory spikes. Throw the spike away, then start here.
- Debugging an existing failure (no new contract to write).
- Pure refactors with no observable behaviour change.
- Writing tests for code that already exists - that is coverage work, and the split buys nothing
  because the implementation is already public.

## Tool authorization

**This skill authorizes dispatching a subagent**, and requires it. Harnesses that otherwise forbid
subagent use unless a skill asks for it are satisfied by this sentence. If you cannot obtain a fresh
context, see "Degrading honestly" below - do not silently write your own tests and call it TDD.

## The sequence

Nine steps. The order is the mechanism; a step done early leaks the plan.

### 0. Baseline

Run the suite and the QA checks BEFORE the first edit. That is what makes "this failure is
pre-existing" provable later, and it stops a pre-existing red masking your own.

### 1. Orient - and do not design

Read, in full: the code under change, the machinery that INVOKES it, the contract in the project's
documentation, and the existing test module. Do not reason about the implementation yet, and do not
argue a shape in your own context.

Two reasons, and only the second is obvious. Obvious: you are about to hand this context's
conclusions to a reviewer, and it anchors on whatever you have already argued. Less obvious: your
orientation BOUNDS what the test author can test against. It only knows what you point it at.

### 2. Write the spec

The spec is the only artifact that crosses the boundary. Given/When/Then, observable behaviour only.

The discriminator, and it is the whole skill in one line:

> **If it names a class, field, method, or file that does not exist yet, it is a plan, not a spec.**

See `references/spec-format.md` for the format, the plan-versus-spec test, and worked examples.

### 3. Dispatch the test author into a fresh context

Template in `references/test-author-prompt.md`. Three rules that are not negotiable:

- **The spec goes inline in the prompt.** Never a path to a document that also holds the approach.
- **Give paths to READ, not conclusions.** The test author orients itself.
- **Ask for the test set, not for tests you described.** Names, fixtures, the OLD assertion and the
  NEW one, the retained behaviour to pin, the mutation that must kill each test.

### 4. Gate the test set before transcribing

A second reviewer - one that sees your transcript, but has not seen a plan, because none exists yet -
checks the returned set against the spec. The one question worth asking it: **does this set pin the
behaviour that must be RETAINED, or only the behaviour being added?**

The checklist it should judge the set against is in `references/evidence-rules.md`, section "Test
design".

**If the gate finds the set deficient, the fix does not come from you.** Redispatch to the same test
author - its context is still clean, and it can be continued rather than started over - stating the
gap in spec terms: observable, no layer named. Transcribe what comes back. Patching the set yourself
is the same leak as editing it in step 5, one step earlier.

### 5. Transcribe verbatim

You do not edit tests authored for your own change. A test that does not run as dictated is
REPORTED, not quietly fixed. Rewriting it here is how the plan gets back in through the side door.

### 6. Pin, then flip - and confirm red for the PREDICTED reason

**This step is TWO runs, always.** That is why the test author was asked for the OLD assertion as
well as the NEW one. If no test pins the current value, pinning it IS run 1 - and for genuinely new
behaviour the current value is its absence, which is still assertable.

1. **Run the OLD assertion first. It must be GREEN.** That proves the fixture reaches the code path
   at all. A red here is not progress: the orientation or the fixture is wrong, and nothing after it
   can be trusted. Redispatch.
2. **Flip to the NEW assertion. It must be RED**, for the reason the spec predicted.

Skipping run 1 is what produces the commonest false red - a test that fails because it never reached
the code, indistinguishable in the output from one that fails because the behaviour is missing.

Red-as-expected then proves two things at once: the test reaches the new behaviour, and your model of
the CURRENT behaviour was right.

- If run 2 is red for another reason, your model of the current behaviour is wrong. Re-derive the
  spec and redispatch - do not repair the test, and do not proceed.
- If run 2 is green, the behaviour already exists - run 1 has already ruled out the fixture. Say so
  and stop: the change may be smaller than assumed, or unnecessary.
- **A kill or a red with no count of tests run is a crash, not evidence.** Quote the runner's tally
  verbatim - `Ran 7 tests`, `7 passed`, `ok 7`, `7 examples, 0 failures`, whatever yours prints - plus the
  named failure lines. `references/evidence-rules.md` has the rest, including the multi-leg harness
  whose second leg never ran.

### 7. NOW design the implementation

Only here. Everything before this point had to stay clean.

**Favour a deep module** - a function, class or package whose interface is small next to what it
does. Tests pin visible behaviour, not the implementation, so the test set is sized by the
behaviours the interface exposes, however the code behind it is arranged - fewer than the same code
split shallow, where each internal step gets a test of its own. And the test set has just pinned
that interface, and step 5 forbids you to edit those tests. Behind a small surface a later
implementation change needs no edit to them; behind a shallow one each change forces an edit to
them, and that is the leak this skill exists to close. Expose nothing the spec does not need.

Fewer tests is not less proof: step 8 still mutates the implementation and must kill each mutant
THROUGH the interface. A survivor is triaged in this order. First, look for an interface input that
tells it apart: if one exists the set is incomplete, so redispatch for that test (step 4's
mechanism; you do not write it). Only when no input can tell it apart is it equivalent, recorded per
`references/evidence-rules.md`. And if what it changes matters but nothing at the interface can see
it, the spec is short: back to step 2. An internal part complex enough to want its own tests is a
deep module of its own: give it an interface and a spec, and test it there.

### 8. Green, then mutation-prove

Full suite green is the floor, not the proof.

**Red-first proves a test MOVES. It does not prove it CONSTRAINS.** Write the obvious wrong
implementation and confirm the test fails. One kill map, one run, against the final source, and
compile the mutated source before running it - an invalid mutant kills every test for the wrong
reason. A green suite gives no signal about what nobody wrote a test for.

Also check: does any EXISTING assertion become unable to fail because of your change? If so the fix
is wrong - find one that leaves it live.

`references/evidence-rules.md` is the checklist for this step and for step 4: what a red proves,
what a green does not, how a kill map goes wrong, why you must not guess the oracle, and the three
cases that need a test when nothing is failing yet.

### 9. Review the diff before declaring done

Hand over the diff, what you measured, and the tests. Give your own suspicions to confirm or refute
rather than an open-ended "look for problems", say plainly that a green suite is not evidence of
absence, and ask the reviewer to say plainly if it finds nothing so it does not manufacture findings.

**A review pass is void the moment you edit in response to it.** Applying corrections is itself
unreviewed work - and it is the part you are least able to see, because you just wrote it. So the
review is the LAST step before declaring done: apply, re-submit, repeat until nothing further comes
back. Applying and declaring done cannot share a turn.

## Leak vectors

The split is only as good as what crosses it. The first two were MEASURED on one harness (see
`references/prior-art.md`) - **re-probe them on yours before relying on the boundary**, because
what a fresh context inherits is a property of the harness, not of the method. The rest follow by
construction and have not been probed anywhere.

- **Persistent memory crosses.** Whatever store your harness injects on its own - a memory index, a
  notes file, a retrieved profile - a fresh context could quote from it with no tool call. So do
  not write the approach into memory mid-slice, and never instruct the test author to consult it.
- **Always-on instruction files are visible too** (`AGENTS.md`, or whatever your harness loads on
  its own). That is correct and wanted - they are rules, not plans. Keep it that way: an instruction
  file is not the place to record the approach for the change in flight.
- **Handoff and plan documents.** Never pass a path to one. Their whole purpose is to carry the
  approach.
- **The spec itself.** The commonest leak is a spec written after the plan, in the plan's vocabulary.
  Step 2 comes before step 7 for this reason.
- **A transcript-sharing reviewer is not a split.** If your "independent" perspective receives your
  conversation history, it sees exactly what you saw. It is a second PASS, not a second CONTEXT. Use
  it as the gate in step 4, never as the author in step 3.

## Degrading honestly

If a role's required context is not available, say so and name which guarantee you are dropping.
Each role has its own ladder, because what each context must or must not have seen differs.

**Never the test author's context for the gate or the review.** At step 4 it would gate its own
set. At step 9 it would see the implementation, and step 7 sends a survivor back to that same
author, which must still have seen no plan.

**The test author (step 3).** Best first:

1. A second session, given only the spec pasted in.
2. A human writing the test set from the spec (this is the XP original - see below).
3. Author them yourself, sequenced strictly: spec before plan, and mark the test set explicitly as
   self-authored so the reviewer in step 9 knows to attack it harder.

Option 3 is the status quo everywhere else. It is a fallback, not the process.

**The gate (step 4).** No plan exists yet, so your transcript holds only your orientation. The loss
that costs is the higher class, not the shared transcript. Best first:

1. A higher-class reviewer that sees your transcript.
2. A higher-class reviewer in a fresh context, handed the spec, the returned set and the paths you
   oriented on.
3. A human, handed the same.
4. A reviewer at your own class that sees your transcript. It is a second pass at your level, and
   it mostly re-derives your blind spots: mark the set as gated at your own class.
5. A fresh context at your own class, handed what rung 2 is handed. It ranks below rung 4 here and
   above it at step 9: with no plan yet, the transcript it lacks is only orientation, and it still
   shares your blind spots.
6. Judge the set yourself against `references/evidence-rules.md`, section "Test design", item by
   item, and mark it self-gated.

**The review (step 9).** Your transcript now holds the plan, so at your own class a fresh context is
the better reviewer: it judges the diff without your reasons for it. Best first:

1. A higher-class reviewer that sees your transcript.
2. A fresh context at your own class or higher, handed the diff, the spec, the tests, what you
   measured and your suspicions - never the plan.
3. A human, handed the same.
4. A reviewer at your own class that sees your transcript, marked as such.
5. Review it yourself against `references/evidence-rules.md`, and mark the result self-reviewed.

Every rung of the step-9 ladder keeps step 9's loop: an edit made in response voids the pass, and
the corrected diff goes back to the same rung.

## Portability

The requirement is a ROLE, not a product: _a context that has not seen the plan_.

| Harness                  | Implementation                                                                    |
| ------------------------ | --------------------------------------------------------------------------------- |
| Agent CLI with subagents | its own subagent or task mechanism - probe that a new dispatch starts fresh first |
| No mechanism             | a second session, or a second human, for each role - see "Degrading honestly"     |

Use a general-purpose agent with file-reading tools, not a minimal one: the test author has to
orient on the code and the existing test module itself. Note the tradeoff you are making - the
usual alternative is a STRONGER model with a contaminated context; this is the SAME model with a
clean one. The isolation is what is being bought.

**That tradeoff applies to the AUTHOR only.** The step-4 gate and the step-9 reviewer are the
mirror case: they are MEANT to see your transcript, so there is no isolation left to protect and
nothing is spent by making them stronger. Both should be a HIGHER-class model than the
implementer. A gate running at the implementer's own level mostly re-derives the implementer's
blind spots, which is the one thing it exists not to do. With no higher class available, the step-9
reviewer is the exception: by then your transcript holds the plan, so a fresh context does better
than one that shares it - see "Degrading honestly".

## Lineage, and why this is not the skill you find online

Ping-pong pairing in Extreme Programming is the human original: one person writes the failing test,
the other makes it pass, then they swap. The split was never about order. It was about the test
being written by someone who does not yet know how it will be made to pass.

`references/prior-art.md` carries that lineage, what the boundary was actually measured to be on
one harness, and the probe to re-establish it on yours.
