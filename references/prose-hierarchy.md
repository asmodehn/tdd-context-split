# The prose hierarchy: principles and rules, above the spec

A spec says what one change must do. Above it, a project has reasons that hold across every change:
what it protects, what it promises, what it refuses. Written down, they form a hierarchy:

```
principle   one sentence, in the owner's words          owner states it
  rule      a consequence, with its "because"            agent derives it, logs it
    example Given / When / Then (spec-format.md)         one change's spec
      test  the test author's set                        fresh context
        code                                             implementer, step 7
```

**The principle of this mechanism:** every rule an agent applies traces to a principle the owner
stated; a rule that traces is applied and logged, a rule that does not goes to the owner. Everything
below derives from that sentence.

## Why it sits above the spec

Without a rules level, a spec grows one clause per input found: each gate pass names an input the
spec never listed, the clause is settled on its own, and the next pass names another. Each clause is
then a case the code handles by itself, and the cases disagree. With the rules written down, a new
input is first checked against them: either a rule already decides it (one more Given, nothing
new to settle) or no rule does, and that gap goes up the hierarchy, not into another clause.

The rules must live where the test author may read them. A rule kept only in a plan document never
crosses the boundary, since plan documents are exactly what is withheld.

## Who writes which level

| Level     | Proposed by       | Decided by                                     |
| --------- | ----------------- | ---------------------------------------------- |
| principle | owner, or cited   | owner, always: stated by them, or adopted      |
| rule      | agent             | the derivation itself; the owner only on a gap |
| example   | implementer       | the spec checklist, then the gate              |
| test      | fresh test author | the gate                                       |

The split follows what each side was observed to do well: agents derive and check concrete cases,
and drift when they choose the abstraction, drafting sentences that cover the evidence but are not
the owner's reasons; owners hold the reasons and tire of confirming cases. So the owner states the
top, the agent derives downward, and the owner is asked only where a derivation fails.

## Building it while learning a project

Run it the first time you work in a project with no hierarchy, and again whenever a trigger below
fires. Each pass is short; the hierarchy grows with the work, never as a survey up front.

1. **Harvest, without asking.** Read the instruction files, the contract documentation, recent
   review comments and the code you are about to change. List the regularities you observe as
   CANDIDATE rules, each with where you saw it. A correction the owner gives in the session is a
   candidate too, quoted.
   Keep apart what the project IS from what it WANTS: a fact (the daemon reads only `*.json` in
   one directory, the API answers 400 to an invalid write) is checked against the code or the system
   and recorded with where; a rule is what the project wants given those facts. A fact is never a
   question for the owner; it is measured.
2. **Climb: ask why, and let the owner answer.** Show the owner the candidates, grouped by what
   they act on, with where each was seen, and ask which principle they come from. Do not draft the
   principle: a sentence written to cover the evidence is an abstraction the owner then has to
   correct. The owner's answer, in their words, is the principle; record it verbatim. An owner may
   also state a principle no candidate pointed to: descend from it all the same.

   Not every rule needs a principle, but several similar rules left over mean something is
   unstated: a principle, or a feature whose contract they are. Rules that all serve one capability
   the project offers are that feature's, homed in its documentation, and need no principle. For
   the rest, look in the established references of the field before asking - a design principle, a
   usability heuristic, the language's own idioms - and offer the ones that cover the group, each
   quoted from its source, for the owner to adopt or refuse. Quoting a published principle is not
   drafting one: the wording and its meaning are on record, and the owner judges whether it fits.
   An adopted published principle is cited, by name and source, never restated: what the project
   records under it is only its own rules.

3. **Descend: derive, and sort.** From each principle and the facts, derive its rules, and sort
   every candidate:
   - **It derives.** Record it with its "because" clause (the principle and the facts it rests on)
     and apply it. Do not ask for confirmation: a derivation the owner must confirm one by one is a
     list of rulings, not a hierarchy.
   - **It contradicts the principle.** It is an exception already in the project. Show the owner
     the uniform alternative and its cost; the owner keeps it as a recorded exception or the
     project changes.
   - **The principle gives no answer.** Ask. The answer amends a principle, adds one, or becomes a
     recorded exception - never an unrecorded rule.

   Then, for each rule, ask what would make it false: an input, an order of events, a stale view, a
   fact that stops holding. Each obstacle found either is decided by a rule already written or is
   the next gap. This is the pass that finds, once per rule, the inputs a gate would otherwise find
   one per pass.

4. **Record**, in the structure below, in the project's home for it. A derived rule the code does
   not follow is a finding against the code, not a reason to drop the rule.

**Asking the owner.** The owner's attention is what this mechanism spends, so: one question at a
time, each carrying the evidence, and only at the two places above - a principle to state, or a
case the principle does not decide, which comes with a recommended answer. Word it so each possible
answer says exactly what will be recorded: an option the owner can read two ways is answered one
way and recorded the other. A question nobody present can answer is parked, not argued: it goes
under its principle as an open question, with who can answer it, and the rules that depend on it
wait.

## The structure

One heading per principle, identified by name, never by number: a number is a coordinate that dies
with the list it indexes.

```
## <name of the principle>

<the principle: one sentence, the owner's words> (owner, <date>)

Facts (each checked, with where):
- <what the project or system does> (<where it was read or measured>)

Rules:
- <rule> - because <the principle and the facts it rests on, one clause>

Out of reach:
- <what the principle cannot cover> - because <the fact that puts it out of reach>

Costs accepted (each the owner's, <date>):
- <what following the rules uniformly costs>

Exceptions (each the owner's decision, <date>):
- <the exception> - instead of <the uniform rule>, at the cost of <what it costs>

Open (each with who can answer it):
- <the question> - <who>; waiting on it: <the rules>
```

A published principle the owner adopted takes the same block with its citation in place of the
sentence - `## <name> (<source>)`, then `adopted (owner, <date>)` - and only the project's own
rules beneath it.

A fact that stops holding reopens every rule whose "because" names it.

A cost is the price of a rule applied without exception, an exception the price of not applying
it: both are the owner's to accept, and only the exception is a smell. Rejected alternatives, and
what each rejection rests on, go in the project's decision log, beside the principle's name. A
principle with more exceptions than rules is not the owner's principle yet: climb again.

## Where it lives

The hierarchy is permanent content, not a plan: it outlives every change it shaped. The skill names
the slot; the project's own rules about where things live decide the file.

- **A project's principles** go in its always-on instruction file, kept short: they are rules, and
  that file crossing into a fresh context is correct and wanted.
- **A feature's principle and rules** go in that feature's contract documentation, the reader's
  home, and derive from the project's principles.
- **Promote, never copy.** A rule found true across a family of repositories moves to a skill the
  family shares; one true of every project moves to a generic skill. It leaves the lower home in
  the same change.

## Using it in the sequence

- **Step 2.** Each Given/When/Then is an example of a rule, and the spec names the rules it
  instantiates. A clause that instantiates no rule is a gap: descend from the principle first, and
  ask the owner only if the principle gives no answer. A rule settled for this change enters the
  durable home before step 3 only if it passes the spec discriminator - it names nothing that does
  not exist yet. A rule that names the design waits until after step 7.
- **Step 3.** The principle and the rules the spec cites go inline in the test author's prompt,
  expanded, never as a path.
- **A review finding no rule explains**, and **an owner's correction**, re-enter at step 3 of the
  build: derive it, flag it, or ask.
