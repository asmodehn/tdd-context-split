# Dispatching the test author

The prompt below is the whole interface. Anything not in it, the test author must discover by
reading; anything in it that should not be, is a leak.

## Template

```
You are authoring a test set for a change you will NOT implement. Someone else implements it,
from your tests. You have not been told how they intend to do it, and you must not ask.

SPEC (the only statement of intent you get):

  Given  ...
  When   ...
  Then   ...
  And    ...

ORIENT FIRST, by reading - in full, not skimmed:

  - <path to the code under change>
  - <path to the machinery that invokes it>
  - <path to the existing test module, IN FULL>
  - <path to the contract or documentation, if one exists>

HOW TO RUN TESTS HERE: <the project's test command, and any constraint on it>

RETURN a test set, not prose. For each test:

  - the test name
  - the fixtures it needs, preferring fixtures and mixins that already exist in this codebase
    over anything new
  - the assertion as it stands TODAY (so we can confirm the test reaches the code path), and
    the assertion it must become
  - which clause of the spec it covers - including the retained-behaviour clauses, which are
    not optional
  - the MUTATION that must kill it: the obvious wrong implementation which, if written, this
    test must fail against
  - where its expected value comes from: a literal from the spec, or a measurement of the real
    system. Never anything a mutation of the code under test would also move - a call into it,
    a constant it returns unchanged, a mock of it. A constant it takes as INPUT is fine

Also return, plainly:

  - anything in the spec you could not pin to code you read
  - anything the spec asserts that the code already does (we need to know before implementing)
  - any retained behaviour you found that the spec failed to mention

Do not write implementation code. Do not propose a design. If you find yourself needing to know
how it will be implemented in order to write a test, that test is testing the implementation -
say so instead of writing it.
```

## When the subject is an experiment, not code

The same dispatch, with the return shape swapped. An experiment set is: what to VARY, what to
OBSERVE, what result would FALSIFY the hypothesis, and what the instrument must be able to show for
an empty result to count as evidence. Ask for those four instead of names/fixtures/assertions/
mutations, and keep every other rule - the hypothesis goes inline WITH its prediction, since nothing
can be falsified against a hypothesis that predicts nothing, and the system is pointed at rather
than characterised. What is withheld here is the PROBE: the conditions you would have sampled and
the instrument you had in mind. That is where agreeable samples come from, and it is the analog of
naming the field.

## Filling it in

- **The spec goes inline, expanded.** Not a path. A path to a document is a path to whatever else
  that document contains, and plan documents are exactly the thing being kept out.
- **Paths are to read, not conclusions.** "Read the scheduling module" is a pointer. "The bug is in
  the dispatch routine, around the timeout handling" is your diagnosis, and handing it over
  collapses the split for the one part that mattered.
- **Name the test command and its constraints.** The test author should be able to say "this test
  cannot run in that harness" before you discover it at transcription time.
- **Do not name the layer.** If the spec is observable and the paths cover the area, the test author
  picking a different layer than you expected is a finding, not a mistake.

## What to do with what comes back

- **Transcribe verbatim.** Names, fixtures, assertions. If something does not run as dictated,
  report it - to the reviewer and to the user - rather than repairing it yourself. The repair is
  where your plan re-enters.
- **"The code already does this" is the most valuable line in the reply.** It means the spec was
  wrong, or the change is smaller than assumed. Stop and re-derive before implementing.
- **"Retained behaviour the spec failed to mention" is the second most valuable.** Add it to the
  spec. That is the clause the split exists to produce.
- **A returned test that cannot fail** - an assertion that matches incidentally, a fixture that
  cannot reach the branch it is named for, an expected value the code under test itself produces -
  is a defect in the set, and it is the reviewer's job in step 4 to catch it, not yours to silently
  rewrite.

## Anti-patterns in the dispatch

| Smell in the prompt                           | What it leaks                                  |
| --------------------------------------------- | ---------------------------------------------- |
| "add a test for the new `retry_count` field"  | the field name, so the design                  |
| "see the plan in `docs/<something>.md`"       | the entire approach                            |
| "the fix is to catch the timeout - test that" | the implementation, stated as a requirement    |
| "write a test like the one at `<file>:88`"    | the shape, which forecloses a better one       |
| "don't worry about the existing tests"        | removes the retained-behaviour half of the job |
