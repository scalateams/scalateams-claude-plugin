# Scala Teams Claude Code plugin

[![Validate plugin](https://github.com/scalateams/scalateams-claude-plugin/actions/workflows/validate.yml/badge.svg)](https://github.com/scalateams/scalateams-claude-plugin/actions/workflows/validate.yml)

Specialist agents for the Scala ecosystem, designed for use with [Claude Code](https://claude.com/claude-code).

One agent per library or concern. No bloat.

## Install

```bash
claude plugin marketplace add https://github.com/scalateams/scalateams-claude-plugin
claude plugin install scalateams@scalateams
```

Or from a local clone:

```bash
claude plugin marketplace add /path/to/scalateams-claude-plugin
claude plugin install scalateams@scalateams
```

## What you get

Specialist agents covering:

- **Build:** sbt, Mill
- **Effect systems:** Pekko (actor, streams, persistence, HTTP, Kafka), ZIO (core, streams, HTTP, Kafka), Cats Effect (core, fs2, http4s, fs2-kafka)
- **Testing:** ScalaTest, MUnit, Weaver, zio-test
- **API design:** Tapir
- **Codecs:** Circe, jsoniter-scala, Play JSON
- **Protocols:** ScalaPB (gRPC + protobuf), avro4s
- **Databases:** Doobie, Quill, Slick
- **FP review:** separate Scala 2 and Scala 3 reviewers (the idiom catalogs differ enough to warrant the split)

See [`docs/usage.md`](./docs/usage.md) for the full agent catalog, triggers, and composition patterns. See [`CLAUDE.md`](./CLAUDE.md) for design rationale.

## Philosophy

Specialists are **narrow on purpose**. Reviewing a Doobie query and a Slick query draw on different libraries with different ergonomics — bundling them into one "DB reviewer" forces every invocation to carry context for libraries it isn't using. We picked sharper over broader.

When a task crosses libraries, the main agent invokes multiple specialists in sequence. That's the intended pattern, not a workaround.

## Contributing

PRs welcome. Add a specialist by:

1. Creating `agents/<name>-specialist.md`
2. Keeping the agent body under ~150 lines
3. Listing it in `CLAUDE.md`

## License

MIT
