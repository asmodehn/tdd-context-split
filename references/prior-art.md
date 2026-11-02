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

## The prose hierarchy: what was borrowed

Surveyed 2026-11-02, from the sources named; `prose-hierarchy.md` carries the mechanism. Two
published spec-driven agent toolkits were read as well: one keeps a constitution of principles, the
other files of conventions, and neither derives its rules from a principle - both list them. What
the mechanism takes, and from where:

| Source                                                                 | Taken                                                                                       |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Example Mapping (Wynne, 2015)                                          | rules between story and examples; a question nobody present answers is parked with an owner |
| KAOS (van Lamsweerde, RE'01)                                           | WHY questions climb, HOW questions descend; obstacle analysis: what makes a goal false      |
| GSN Community Standard v3 (SCSC-141C, 2021)                            | the inference between levels stated explicitly, with its justification                      |
| Zave and Jackson, "Four Dark Corners" (TOSEM 1997)                     | domain knowledge kept apart from requirements, and checked as true of the environment       |
| Leveson, intent specifications                                         | the WHY recorded at each level; an assumption that stops holding triggers re-analysis       |
| DO-178C derived requirements (via a checklist citing 5.1.1.b, 5.1.2.h) | what traces to no parent is named, states its reason, and goes back up                      |
| a spec-driven toolkit's clarification step                             | one question at a time, each with a recommended answer                                      |

Not taken: formal refinement with proof obligations (Event-B), which needs a formal model; ADR
numbering, since a number is a coordinate - the project's own decision log keeps the rejected
alternatives.
