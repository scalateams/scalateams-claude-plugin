# CLAUDE.md — scalateams plugin

This is a Claude Code plugin providing specialist agents for the Scala ecosystem.

## Design principle

**One agent per library or narrow concern.** No generalist agents that try to cover multiple libraries. No skill preloading — context cost is paid per invocation, so we keep agents lean by giving each one a single sharp specialty.

When a task spans multiple libraries (e.g., a Tapir endpoint using Circe codecs reviewed for ZIO error handling), the main session invokes 2–3 specialists in sequence rather than one bloated agent that half-knows everything.

## Agent inventory

Single `agents/` directory, flat. Specialists are organized below for human readability — they all live in the same folder.

### Cross-cutting (3)
- `scala2-fp-reviewer` — language-level FP review for Scala 2.13 idioms
- `scala3-fp-reviewer` — language-level FP review for Scala 3 idioms (`enum`, `extension`, `given`, `derives`, opaque types)
- `codebase-explorer` — read-only navigation, framework detection from build.sbt

### Build (2)
- `sbt-specialist`
- `mill-specialist`

### Effect systems

**Pekko (5)**
- `pekko-actor-specialist`
- `pekko-streams-specialist`
- `pekko-persistence-specialist`
- `pekko-http-specialist`
- `pekko-kafka-specialist`

**ZIO (4)**
- `zio-core-specialist`
- `zio-streams-specialist`
- `zio-http-specialist`
- `zio-kafka-specialist`

**Cats Effect (4)**
- `ce-core-specialist`
- `fs2-specialist`
- `http4s-specialist`
- `fs2-kafka-specialist`

### Testing (4)
- `scalatest-specialist`
- `munit-specialist`
- `weaver-specialist`
- `zio-test-specialist`

### API design (1)
- `tapir-specialist` — binding-agnostic endpoint design; binding details delegated to the relevant HTTP specialist

### Codecs (3)
- `circe-specialist`
- `jsoniter-specialist`
- `play-json-specialist`

### Protocols (2)
- `scalapb-specialist` — gRPC + protobuf in Scala
- `avro4s-specialist`

### Databases (3)
- `doobie-specialist`
- `quill-specialist`
- `slick-specialist`

## Why no skills directory

Sub-agents (invoked via the Agent tool) cannot dynamically activate skills — skills must be preloaded via the `skills:` frontmatter field, which means every invocation pays the token cost of every preloaded skill. For a multi-library agent like a generic `db-reviewer` with Doobie + Quill + Slick skills preloaded, two-thirds of every invocation is wasted context. Splitting into per-library specialists is cheaper and sharper.

If shared content emerges (e.g., "how to detect framework from build.sbt" repeated across many agents), revisit. Don't speculate.

## Adding a new specialist

1. Create `agents/<name>-specialist.md` with frontmatter:
   ```yaml
   ---
   name: <name>-specialist
   description: <what this agent specializes in — used for context-based routing>
   tools: Read, Grep, Glob, Bash
   ---
   ```
2. Body: tight, focused on one library or concern. Target <150 lines.
3. Update the inventory in this file.

## What this plugin is NOT

- Not a guide to writing Scala. It assumes the user knows the language.
- Not a curriculum. Specialists offer review and implementation help; they don't teach the basics.
- Not opinionated on effect-system choice. Pekko, ZIO, and Cats Effect specialists coexist as peers.
