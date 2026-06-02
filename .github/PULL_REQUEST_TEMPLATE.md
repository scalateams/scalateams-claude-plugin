<!-- Thanks for contributing! Keep PRs scoped — ideally one specialist or one concern. -->

## What and why

<!-- Briefly describe the change and the motivation. -->

## Type of change

- [ ] New specialist agent
- [ ] Change to an existing agent's body
- [ ] Change to an agent's `tools:` or other frontmatter
- [ ] Docs / build / CI
- [ ] Other:

## Checklist

- [ ] Scope holds: one agent per library / narrow concern (no generalist merges)
- [ ] Agent body is focused (aiming for < ~150 lines)
- [ ] Frontmatter is valid — ran `claude plugin validate ./scalateams` locally with **0 errors**
- [ ] Tool grants are least-privilege (read-only agents do **not** request `Write`/`Edit`)
- [ ] Inventory updated in `scalateams/CLAUDE.md` and catalog in `docs/usage.md` (if agents changed)
- [ ] No new bundled executable dependencies

## Security note

If this PR changes an agent body or its `tools:` line, flag it here — those changes
get a security-focused review. See [SECURITY.md](../SECURITY.md).
