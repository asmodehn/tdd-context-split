# tdd-context-split

One agent skill, in the [Agent Skills](https://agentskills.io/specification) format. **The
repository root IS the skill directory** - `SKILL.md` and `references/` are here, not one level
down - so a consumer symlinks the repository in under the skill's own name and nothing else is
needed. `.git/` beside them is ignored by every loader.

Test-first where the test author and the implementer do not share a context, only the spec. It is
generic working practice: it names no repository, no path and no tool of any project, and it must
stay that way.

## Why it has its own repository

It belongs to no project, so it does not belong in any project's skill set; and it is under active
development against real work, so user scope (`~/.agents/skills/`) would make every edit
machine-wide before it has settled. A repository of its own is the third answer: versioned and
recoverable, mounted only where it is wanted.

Before this existed it lived as an untracked directory inside another project's submodule
checkout - one copy, in no git anywhere. It was one `rm -rf` from gone, which is how a sibling
skill was in fact lost earlier the same week.

## Using it in a repository

Either way the consumer ends up with a `tdd-context-split/` directory holding `SKILL.md` in its
`.agents/skills/`, which is all a loader looks for. What differs is who else gets it.

### As a submodule - shared and pinned

The consumer tracks the skill at a fixed commit, so every clone gets it and an update is a reviewed
pointer bump:

```bash
git submodule add https://github.com/asmodehn/tdd-context-split.git .agents/skills/tdd-context-split
```

If the consumer's `.agents/skills` is itself a submodule, add it from inside that one instead, and
clone with `--recurse-submodules` (or run `git submodule update --init --recursive`) so the nested
checkout is not left empty:

```bash
cd .agents/skills
git submodule add https://github.com/asmodehn/tdd-context-split.git tdd-context-split
```

Landing a change to the skill is then one commit per level: here, push; in `.agents/skills`, commit
the pointer bump; in the consumer, commit the `.agents/skills` bump.

### As a symlink - one developer's checkout

Nothing is tracked, so only the machine that made the link has the skill, and edits in the checkout
are live at once:

```bash
ln -s ~/Projects/tdd-context-split .agents/skills/tdd-context-split
```

Measured 2026-09-25, against a consumer whose `.agents/skills` is itself a git submodule, with a
harness-specific skills directory symlinked to it: two agent CLIs discovered a SYMLINKED skill
directory through that double indirection, and the consumer's own lint and format checks stayed
green with the link in place.

If the consumer's `.agents/skills` is a submodule, the link is untracked inside it. Exclude it
there, and note the pattern must NOT end in a slash - a trailing slash matches directories only,
and git sees a symlink as a blob:

```bash
cd .agents/skills
echo 'tdd-context-split' >> "$(git rev-parse --git-dir)/info/exclude"
```

## Keeping it portable

The maintenance rules for this repository are in `AGENTS.md`.
