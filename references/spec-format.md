# The spec: the one artifact that crosses the boundary

The spec is written by the implementer, before any implementation reasoning, and is the only thing
the test author receives about your intent. Everything else it must discover from the code itself.

Because the implementer writes it, the spec is where the plan leaks. The format exists to make that
leak visible.

## Format: Given / When / Then

Borrowed from BDD for the discipline, not for the toolchain. **No `.feature` files, no Cucumber, no
Gherkin parser.** The tests are written in whatever the project's suite already uses. What is
adopted is the three-part shape, because it forces each clause to name something observable.

```
Given  <the state of the world before, in terms a user or a caller can observe>
When   <the event or call, named the way a caller names it>
Then   <the observable consequence>
And    <the consequence that must still hold - the RETAINED behaviour>
```

Every spec needs at least one `And` clause pinning retained behaviour. A spec with only new
behaviour produces a test set that proves the feature appeared and nothing about what it broke.

## The discriminator

> **If it names a class, field, method, or file that does not exist yet, it is a plan, not a spec.**

Names that already exist are fine - they are part of the world the test author can read. Names that
do not yet exist are the implementation you have not told them about, and writing one into the spec
hands over the answer.

The test that catches this in practice: read each noun in the spec and ask "can the test author find
this by reading the repository?" If no, it came from your head.

## Worked examples

### Bad - a plan wearing a spec's clothes

```
Given an Account with a Settings record
When the new `Settings.retry_mode` field is set to "aggressive"
Then `SettingsPresenter` includes retry_mode in its output
```

Three tells. `retry_mode` does not exist yet. `SettingsPresenter` names the layer you decided to
change. And there is no retained-behaviour clause at all. A test author given this writes exactly
the test you would have written - the split bought nothing.

### Good - the same change, stated observably

```
Given  an account that belongs to a workspace and has stored settings
When   a member of that workspace reads those settings
Then   the result states how aggressively failed operations are retried
And    a member of a different workspace still cannot read them
And    an account whose retry behaviour was never chosen behaves as it does today
```

The test author must now find the layer that produces the result, the access check and the current
default by reading. It can choose a different field name, a different layer, or notice that the
existing default is not what you assumed. All three are outcomes you want, and all three are
impossible if you named the field.

### Bad - a bugfix spec that describes the fix

```
Given the client raises on a timeout
When we catch that error and return an empty result
Then no error propagates
```

Catching and returning empty IS the fix. The spec has become a diff.

### Good - the same bugfix

```
Given  a remote service that does not answer within the configured timeout
When   a job is dispatched to it
Then   the job is recorded as failed, with the timeout named in its output
And    the failure does not prevent the next job to that service from being dispatched
And    a service that answers normally is unaffected
```

Note the second `And`. It is the behaviour a naive fix (swallowing the error and leaving the
connection half-open) would break, and it is the clause most likely to turn the test set red for
the right reason.

## Empirical experiments

The same shape works when the subject is an external system rather than your own code, and the same
discriminator applies, shifted one place: naming the PROBE you would run is the plan. The prediction
is part of the spec and must cross, because a falsification test cannot be designed against a
hypothesis that predicts nothing.

```
Given  <the system in a state you can establish and verify>
When   <the single thing you vary>
Then   <what you will observe>
And    <the observation that would FALSIFY the hypothesis>
```

The falsification clause is not optional. Without it the experiment can only confirm.

## Checklist before dispatching

- Every noun is findable in the repository, or is plainly a user-facing concept.
- At least one clause pins behaviour that must be RETAINED.
- No clause names a layer, a pattern, or a mechanism.
- Written before any implementation reasoning appeared in your context, not after.
- For a bugfix: the spec describes the observable defect, not the repair.
