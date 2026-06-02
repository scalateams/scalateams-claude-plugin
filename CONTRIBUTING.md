# Contributing

Thanks for helping improve the Scala Teams plugin. This project is a collection of
narrow, single-purpose Claude Code agents for the Scala ecosystem. Contributions
that keep agents sharp and current are very welcome.

## Ground rules

**One agent per library or narrow concern.** No generalist agents. If a task spans
multiple libraries, the expectation is that the main session invokes several
specialists in sequence — not that one agent half-knows everything. PRs that merge
specialties back together will be declined.

**Agents are executable.** An agent body is a prompt that runs on a user's machine
with the tools listed in its frontmatter — including `Bash`, `Write`, and `Edit`.
Treat every change to an agent body or its `tools:` line as security-relevant. See
[SECURITY.md](./SECURITY.md).

## Adding or changing a specialist

1. Create or edit `scalateams/agents/<name>-specialist.md` with frontmatter:

   ```yaml
   ---
   name: <name>-specialist
   description: <what this agent specializes in — used for context-based routing>
   tools: Read, Write, Edit, Grep, Glob, Bash
   ---
   ```

   - `description` drives routing — make it specific. State the library and, where it
     matters, the version range the agent targets.
   - Read-only review/navigation agents should grant `Read, Grep, Glob, Bash` only —
     no `Write`/`Edit`.
   - Avoid YAML-hazardous characters in `description`: an unquoted value containing
     `: ` (colon-space) or `[` will fail to parse and the agent silently loads with
     **no** metadata. Quote the value if in doubt.

2. Keep the body focused and tight — aim for **under ~150 lines**. A few existing
   agents exceed this; that's a known debt, not a license to add more.

3. Update the inventory in [`scalateams/CLAUDE.md`](./scalateams/CLAUDE.md) and the
   agent catalog in [`docs/usage.md`](./docs/usage.md).

## Validate before opening a PR

Validation runs in CI on every PR and is a required check. Run it locally first:

```bash
claude plugin validate .            # marketplace manifest
claude plugin validate ./scalateams # plugin + every agent's frontmatter
```

Warnings are fine; errors must be zero.

## Pull request flow

- `main` is protected: changes land through a PR with one approving review and a
  passing `validate` check.
- Branch from `main`, push your branch, open a PR, fill in the template.
- Keep PRs scoped — one specialist or one concern per PR is ideal for review.

## Releases

We follow [semantic versioning](https://semver.org/) via the `version` field in
`scalateams/.claude-plugin/plugin.json`, git tags, and GitHub Releases. Notable
changes are recorded in [CHANGELOG.md](./CHANGELOG.md).
