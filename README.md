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

Symlink the repository in under the skill's own name, at either discovery path:

```bash
ln -s ~/Projects/tdd-context-split .agents/skills/tdd-context-split
```

Measured 2026-09-25, against a consumer whose `.agents/skills` is itself a git submodule and whose
`.claude/skills` is a symlink to it: both Claude Code and OpenCode discover a SYMLINKED skill
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

The skill's whole claim is that it travels. Two rules follow, and both are checkable:

- **No repo-relative path, no project-specific tool, no instruction-file name.** A reader in
  another repository must not be sent to a file that does not exist there.
- **Nothing that duplicates a consumer's always-on instructions.** Where a project's `AGENTS.md`
  or equivalent already states a rule, the skill states the part that is general and lets the
  project keep the part that is local.
