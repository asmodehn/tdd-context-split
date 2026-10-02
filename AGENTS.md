# AGENTS.md

Rules for working on this repository. The repository root is the skill: `SKILL.md` and `references/`
are what a loader reads; `README.md`, `LICENSE` and this file are for maintainers.

## Keeping it portable

The skill's whole claim is that it travels. Three rules follow, and all are checkable:

- **Standard files only.** The skill follows the [Agent Skills](https://agentskills.io/specification)
  format and the `AGENTS.md` convention. No harness-specific instruction file or directory belongs
  here, and no harness or vendor tool is named in the skill content.
- **No repo-relative path, no project-specific tool.** A reader in another repository must not be
  sent to a file that does not exist there. Every path in `SKILL.md` and `references/` resolves
  inside this directory.
- **Nothing that duplicates a consumer's always-on instructions.** Where a project's `AGENTS.md`
  already states a rule, the skill states the part that is general and lets the project keep the
  part that is local.

Check before committing, over the files a loader reads (this file names what it rejects, so it is
out of scope):

```bash
grep -rniE 'claude|anthropic|\.\./|~/|/home' SKILL.md references/
```

It must print nothing.
