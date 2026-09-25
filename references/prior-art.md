# Lineage, and what the boundary was measured to be

## Lineage

Ping-pong pairing, from Extreme Programming: one person writes the failing test, the other makes it
pass, then they swap. **The split was never about ORDER.** It was about the test being written by
someone who does not yet know how it will be made to pass.

That is the property this skill restores. Red-green-refactor kept the order and dropped the split,
because with one person - or one agent - doing both, there was nothing to split.

A survey of the published TDD, BDD and subagent skills was run before this one was written
(2026-09-25). None of them carried the invariant: in every case the same context that plans the
change also authors the tests for it, and the isolation on offer is either fresh-per-TASK or tool
scoping, which is a different property. What BDD contributed is the Given/When/Then SHAPE, adopted
in `spec-format.md` as the spec's required form and nothing more - no parser, no `.feature` files,
no runner.

## What the boundary was measured to be

**Measured on ONE harness, 2026-09-25, with a probe sent into a fresh context and asked what it
could see. Re-run it on yours before relying on any of this** - what a fresh context inherits is a
property of the harness, not of the method, and it is cheap to check.

- **A fresh dispatch carried no parent context.** First user message: `NONE VISIBLE`. Prior tool
  calls or results: `NO`. This is what makes the split real rather than aspirational, and it is the
  one fact worth re-establishing yourself.
- **It DID see the always-on instruction files and the injected memory.** It quoted a heading from
  the project's instruction file and a line from its memory index, with no tool call. Rules crossing
  the boundary is correct and wanted; an APPROACH recorded in either would cross too, which is why
  the leak-vector section names them.

Unmeasured, and treated conservatively: whether a handoff or plan document is visible the same way.
The mitigation is identical either way - the spec goes inline, and no path to a plan or handoff is
ever passed - so it was not worth a second probe.

## How to run that probe

Dispatch a fresh context with no task, asking only what it can see: the first user message of the
conversation, any prior tool call or result, any instruction file, any memory or profile injected
without a tool call. The answers tell you exactly which leak vectors are live in your setup, and
which the harness closes for you.
