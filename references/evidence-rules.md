# Evidence rules: what a run proves, and what must be tested at all

The split gets you a test set written from the contract. It does nothing about the second failure
mode: a run that is quoted as evidence for something it never measured. Every rule below was earned
by a run that was scored as a success and was not one.

**The parent rule, which most of the others are one instance of: every result you quote must
post-date every file it is a claim about.** A claim is verified as of the tree it was measured on;
the tree moves and the claim does not follow it. A suite result older than your last edit is not a
result, a gate that ran before you wrote the file did not check it, and a review pass is void the
moment you edit in response to it.

## Red

- **A red is evidence only when the failure happened INSIDE the tests.** A non-zero exit also means
  the runner never started - so the very result you were hoping for is the one an infrastructure
  crash counterfeits.
- **Report a red by its TALLY, never its exit code.** The count of tests run, plus the named failure
  lines. A verdict with no tally is BROKEN, not RED.
- **Confirm it failed for the PREDICTED reason, quoted from the output.** Not merely that it failed.
- **A harness with more than one leg is verified per leg.** A runner that exits on the first failing
  leg never reaches the second, and a suite-wide claim then covers a leg that did not execute. After
  any run whose first leg was red, find the second leg's own summary line before saying the suite
  ran.
- **One test run at a time, per machine.** Concurrent suites sharing a test database, a cache
  database or the CPU corrupt each other's results silently, and the corruption looks like
  flakiness. Before launching a run - especially one launched indirectly, from a script - check that
  no other suite is running.
- **A RETRYING harness under-reports.** Where failures are retried and only the last attempt is
  recorded, the tally can say one failure while several tests failed - worse, if the harness assigns
  into a shared result object, each failing test overwrites the previous one's traceback. Count the
  retry markers in the log before trusting any tally, and run a new or suspect test ALONE, where no
  retry can hide it.

## Green

- **State which command produced the green and what it SELECTED.** "The suite passes" is not a
  result; "these labels passed, and here is the count" is.
- **A module's ALONE run is the reference for its red or green.** Selections interfere; a count that
  looks plausible can be hundreds short of the real one, and plausible is not a check.
- **Removing something does not make its tests fail.** In a dynamic language the attribute still
  assigns and the assertion still passes, asserting nothing. After a deletion, grep for the name and
  convert those tests to read the real storage - or mutate once, on purpose, to prove they still
  test anything.
- **Five ways a green suite constrains nothing:** an assertion that matches incidentally; a patch
  that pins the USE of a value and not its DERIVATION; a kill read off an exit code with no tally; a
  fixture that cannot reach the branch the test is named for; an expected value the subject itself
  produces.
- **That last one is the tautological test, and the criterion is a mutation, not a source.** The
  expected side is tautological when a mutation of the subject would move it too, so the test
  passes against every wrong implementation. The common shapes: the expected value is computed by
  calling the subject; it is a constant the subject returns or passes through unchanged; the
  subject itself is mocked and the test asserts the mock's configured return. A constant that is an
  INPUT to the derivation is not one: `build_url() == BASE + "/foo"` pins the derivation, since
  mutating it does not move `BASE`. Mutation proof is what exposes a tautology, because the obvious
  wrong implementation SURVIVES it; a set never mutation-proved has not been checked for one.

## Mutation proof

Red-first proves a test MOVES. Mutation proof is what shows it CONSTRAINS.

- **Coverage first.** Before running the map, measure which changed lines the tests execute. A
  changed line no test runs cannot kill anything placed on it: triage it as a survivor - redispatch
  for a test (step 4's mechanism - you do not write it), or, when no interface input reaches it, the
  spec is short. Coverage says a line ran, never that anything checked it, so it bounds mutation
  proof and never replaces it. A tool can misattribute lines, so check a gap against a line known to
  run before believing it; where no tool measures the language, say so - the gap is reported, never
  assumed covered.
- **One source file per kill map, and only the tests that exercise it.** A mutant runs the test
  module for that file, or the tests a coverage pass taken with the baseline run credits with the
  mutated line - never a stored map, which goes stale silently, and never the whole suite once per
  mutant. A change touching several files gets one map per file, in turn. That selection runs green
  against the unmutated source first, since run alone it can fail for reasons the full suite hides,
  and every such failure would read as a kill. A narrowed run can also miss a test and report a
  false survivor: re-run a survivor once against the full suite before triaging it. The full suite
  runs once, as step 8's floor.
- **Compile the mutated source before running it.** A mutation that is syntactically invalid kills
  every test for the wrong reason, and every label "dies" identically. Report it as broken rather
  than as a verdict, and move on. This makes the whole class of false kills impossible rather than
  merely noticeable.
- **One kill map = ONE run, against the FINAL source.** Touch the probe, re-run that file's whole
  map; a file edited after its map is mapped again. Never splice verdicts from two runs with an
  edit in between - stitched verdicts read exactly like real ones.
- **Read the full output, not a grep for the mutation you just added.** That grep is how the other
  verdicts go unlooked-at.
- **An equivalent mutant is RECORDED, never absorbed by a passing run.** Some mutations cannot be
  killed because they do not change behaviour. That is a finding about the code, and it has to be
  written down and argued; letting the map go green around it turns "no test covers this" into "this
  is covered".
- **Run the lint and QA gate AFTER the last artifact exists**, including untracked probe files. A
  gate that ran before you wrote the file did not check it.
- **Do not commit while a measurement of uncommitted work is running.** Commit hooks of the
  stash-checkout-run-restore kind put HEAD on disk for the duration, so a mutation run or a red-first
  run of an uncommitted fix silently measures HEAD instead of your tree. The probe's own end-of-run
  checksum cannot see it - the patch is back before the check runs. If a commit landed mid-label,
  abort the label and rerun it; an unscored mutation is recoverable, a wrongly-scored one is not.

## The oracle

- **When the expected value is system- or device-dependent, MEASURE it** - probe or record against
  the real system - before asserting it. A green test against a guessed oracle certifies nothing,
  and it certifies nothing loudly.
- **An observation that cannot discriminate is not verification.** If every sample is taken under
  conditions where the competing hypotheses predict the same result, nothing has been measured
  however many times it is repeated. Name the observation that would falsify the claim, and check
  the instrument can actually produce it, before calling anything verified.
- **Audit each measurement as it is taken, on three questions, not one.** COVERAGE: does what I
  measured cover what I am about to assert, or only one candidate out of several I never enumerated?
  POSITIVE CONTROL: if the thing I am looking for WERE present, would this command show it? An empty
  result is evidence only once that is established. SPECIFICITY: could anything OTHER than the thing
  under test produce this same reading? A control with two possible causes for one outcome is not a
  control.
- **Assert the PRECONDITIONS of the action.** An action against a control that earlier state
  disabled is a silent no-op, and the test passes vacuously while reporting on whatever the previous
  run left behind. Assert that the thing is in the state that makes the action meaningful, loudly,
  before performing it.

## Test design, when the returned set has to be judged

These decide whether a test set is worth transcribing - they are the reviewer's checklist at step 4
and yours at step 8.

- **Test code is code.** Same review as production for structure, duplication and dead paths. The
  tradeoffs resolve differently in one place: explicitness beats DRY, so keep the assertion visible
  rather than generated.
- **A change that leaves an existing assertion unable to FAIL is a wrong fix.** Find the fix that
  leaves it live. "It is dead now, but I will keep it as documentation" is the signal to change
  approach, not to add a comment.
- **Every expected value in the set has a source the reviewer can name.** One that a mutation of
  the subject would move is a tautology, the fifth of the "Five ways" above, and step 4 is where it
  is caught by reading: there is no implementation to mutate yet.
- **A test that reaches past the surface the spec describes pins shape, not behaviour** - a private
  helper called directly, an internal mocked as if it were a collaborator, one test per internal
  step. It forces the next design to be shallow. Return it to the author with the gap stated in spec
  terms.
- **Configure a double so the real path COMPLETES, rather than injecting an exception into the code
  under test.** A raise aborts mid-flight: the test stops exercising what it claims, it shortcuts
  the normal failure-reporting path, and it kills sibling assertions that can now never be what
  fails. A plain `200 != 403` from a real round trip beats a bespoke message from a detonated one.
- **Check whether the module already has the helper.** If the sibling tests are "already safe", the
  thing that makes them safe is the answer to what your fix is. Read your own measurement.
- **A pin asserting an unreachable state pins a fiction.** If the design admits a state the world
  cannot produce, delete the state rather than assert on it - and drop its test with the branch.
- **A test authored AFTER the change was not red-first, and saying so is the point.** Mutation proof
  is the weaker substitute available to it, and a set proved that way is reported as
  mutation-proved, never as test-first. Silently counting it as TDD is the one dishonesty this whole
  process exists to prevent.

## What must get a test at all

Three cases where the need for a test is easy to miss, because nothing is failing yet.

- **Extending something you do not own** - its output, its DOM, its data shape, its behaviour - is
  depending on it. Pin the assumption in a test, so their next version fails in your CI rather than
  in front of a user.
- **Code the framework may replay, reverse or interleave** must be reversible AND idempotent. Test
  both laws, not one round trip: a reverse that merely exists can still destroy data, and idempotence
  says nothing about the reverse.
- **A document naming a CONDITION the code could check is a missing assertion.** Write the
  assertion and shrink the document to the WHY. The boundary, or this rule deletes every useful
  docstring: a document explaining why a shape was chosen has no assertion to become and stays. Compare
  sizes out loud - a thirteen-line note explaining the absence of a two-line assertion is the tell.
