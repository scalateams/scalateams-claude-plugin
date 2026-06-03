# Changelog

All notable changes to this plugin are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
The plugin version lives in `scalateams/.claude-plugin/plugin.json`.

## [Unreleased]

### Added
- Privacy policy (`PRIVACY.md`) — the plugin collects no data.
- README: usage-examples section showing how prompts route to specialists,
  and a terminal demo illustration (`docs/assets/demo.svg`).

## [0.1.0] - 2026-06-02

### Added
- Initial release: 31 specialist agents covering build tools, effect systems
  (Pekko, ZIO, Cats Effect), testing, API design, codecs, protocols, and databases.
- Contribution, security, and code-of-conduct docs.
- Issue and pull-request templates.
- CI workflow that validates the marketplace manifest and every agent's frontmatter
  on each pull request.
- Dependabot for GitHub Actions.

### Fixed
- `ce-core-specialist` frontmatter no longer fails YAML parsing (the description's
  `F[_]: Sync/...` is now quoted), so the agent loads with its metadata intact.

[Unreleased]: https://github.com/scalateams/scalateams-claude-plugin/compare/scalateams--v0.1.0...HEAD
[0.1.0]: https://github.com/scalateams/scalateams-claude-plugin/releases/tag/scalateams--v0.1.0
