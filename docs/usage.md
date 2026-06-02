# Usage

This plugin ships 31 specialist agents. Claude Code routes to them automatically based on each agent's `description` field — you don't usually need to name them explicitly. This page documents what triggers each one, what they can do, and how they compose.

## How routing works

When you ask Claude Code something, it scans the available agents' `description` fields and picks the best match. You influence routing by what you write:

- "review this Doobie query" → `doobie-specialist`
- "is this Scala 3 ADT idiomatic?" → `scala3-fp-reviewer`
- "wire up a Tapir endpoint over zio-http" → `tapir-specialist` **and** `zio-http-specialist` (composed)

You can also force-select an agent by name in your prompt ("use the `quill-specialist` to..."). Useful when routing picks the wrong one.

## Read-only vs read-write

| Capability | Agents |
| --- | --- |
| Read-only (review, navigate) | `codebase-explorer`, `scala2-fp-reviewer`, `scala3-fp-reviewer` |
| Read-write (implement + review) | All 28 library specialists |

Read-only agents have `tools: Read, Grep, Glob, Bash`. Read-write agents add `Write, Edit`.

## Composition patterns

The plugin's design assumption is that real tasks cross library boundaries. The main agent invokes 2–3 specialists in sequence rather than one bloated agent. Common compositions:

| Task | Agents engaged |
| --- | --- |
| Tapir endpoint with Circe codecs on http4s | `tapir-specialist` → `circe-specialist` → `http4s-specialist` |
| ZIO service with typed errors and zio-test coverage | `zio-core-specialist` → `zio-test-specialist` |
| Pekko Kafka consumer decoding Avro | `pekko-kafka-specialist` → `avro4s-specialist` |
| Doobie repository tested with MUnit + cats-effect-munit | `doobie-specialist` → `munit-specialist` |
| Cassandra-backed Pekko Persistence event store | `pekko-persistence-specialist` (DB schema is out of scope — see "Not in scope" below) |
| Scala 3 domain model with Quill repository | `scala3-fp-reviewer` → `quill-specialist` |
| ScalaPB-generated gRPC service on fs2-grpc | `scalapb-specialist` → `fs2-specialist` |

If you're unsure where to start, ask Claude Code first ("scope this task — which specialists should we use?") and it will route to `codebase-explorer` to detect framework and modules.

## Agent catalog

### Cross-cutting

#### `codebase-explorer`
**Read-only.** Scala codebase navigation. Detects framework (Pekko/ZIO/CE), Scala version, build tool from `build.sbt` or `build.mill`. Maps module structure, traces imports, finds usages.
**Triggers:** "scope this task", "where is X defined", "which files reference Y", "what framework does this repo use".

#### `scala2-fp-reviewer`
**Read-only.** Reviews Scala 2.13 code for language-level FP discipline — purity, immutability, totality, ADT design, type-class hygiene.
**Triggers:** Reviewing pure Scala 2.13 code, domain models, libraries, or code that hasn't picked an effect type. Does **not** review effect-system idioms — delegate to `zio-core` / `ce-core` / `pekko-*` specialists for those.

#### `scala3-fp-reviewer`
**Read-only.** Reviews Scala 3 code for FP discipline — purity, immutability, totality, idiomatic use of `enum`, `extension`, `given`/`using`, `derives`, opaque types, union/intersection types.
**Triggers:** Reviewing pure Scala 3 code, domain models, or code without an effect type. Does **not** review effect-system idioms.

### Build

#### `sbt-specialist`
sbt build configuration — `build.sbt`, `project/*.sbt`, `plugins.sbt`, multi-module projects, cross-builds, custom tasks/commands, dependency resolution, sbt-native-packager.
**Triggers:** Any sbt build question or change.

#### `mill-specialist`
Mill build configuration — `build.mill`, modules, custom tasks, cross-builds, publishing.
**Triggers:** Any Mill build question or change.

### Effect systems — Pekko

#### `pekko-actor-specialist`
Pekko Typed actors — behaviors, message protocols, supervision, ask pattern, `ActorContext`, typed sharding. Covers `pekko-actor-typed` and `pekko-cluster-sharding-typed`.
**Triggers:** Anything to do with typed actors. **Does not cover** streams (`pekko-streams-specialist`) or persistence (`pekko-persistence-specialist`).

#### `pekko-streams-specialist`
Pekko Streams — `Source`/`Flow`/`Sink` graphs, GraphDSL, backpressure, materialization, supervision, throttling, batching, async boundaries. Includes `pekko-connectors` (Alpakka) integrations.
**Triggers:** Any pekko-streams code.

#### `pekko-persistence-specialist`
Pekko Persistence — `EventSourcedBehavior`, `DurableStateBehavior`, snapshots, event adapters, persistence query, `pekko-projection`, CQRS read-side. Covers Cassandra and JDBC journal/snapshot plugins at the Pekko-config level (not the database schema itself).
**Triggers:** Event sourcing or durable state in Pekko.

#### `pekko-http-specialist`
Pekko HTTP — Route DSL, directives, marshalling/unmarshalling, server config, client (`Http().singleRequest`), WebSockets, streaming responses.
**Triggers:** Code using `pekko-http` directly. If the project uses Tapir with the pekko-http interpreter, also engage `tapir-specialist`.

#### `pekko-kafka-specialist`
`pekko-connectors-kafka` (Alpakka Kafka) — Consumer/Producer sources/sinks, committable/at-least-once semantics, partitioned sources, transactional producer, rebalance handling.
**Triggers:** Any pekko-kafka code. For schema-registry codecs, also engage `avro4s-specialist` or `scalapb-specialist`.

### Effect systems — ZIO

#### `zio-core-specialist`
ZIO core idioms — `ZIO[R, E, A]`, `ZLayer` wiring, typed errors via the `E` channel, services, for-comprehensions, error recovery (`catchAll`/`mapError`/`refineToOrDie`), `Ref`/`Promise`/`Queue`/`Hub`, scoped resources.
**Triggers:** ZIO effect code. **Does not cover** ZIO Streams, ZIO HTTP, or zio-test — delegate to those specialists.

#### `zio-streams-specialist`
ZIO Streams — `ZStream`/`ZSink`/`ZPipeline`, chunking, backpressure, scoped streams, parallel transforms, broadcasting, grouping.
**Triggers:** ZStream code. Distinguishes ZStream patterns from ZIO core effects.

#### `zio-http-specialist`
ZIO HTTP — `Routes`/`Handler`/`HttpApp`, middleware, request/response, WebSockets, client API, server config.
**Triggers:** Code using zio-http directly. If the project uses Tapir with the zio-http interpreter, also engage `tapir-specialist`.

#### `zio-kafka-specialist`
zio-kafka — Consumer/Producer with `ZStream`, `Subscription`/`Offset` semantics, transactional producer, partition stream patterns, schema registry integration.
**Triggers:** Any zio-kafka code. For Avro/Proto specifics, also engage `avro4s-specialist` or `scalapb-specialist`.

### Effect systems — Cats Effect

#### `ce-core-specialist`
Cats Effect 3 idioms — `IO` and tagless final `F[_]: Sync/Async/Concurrent`, `Resource` composition, `Ref`/`Deferred`/`Semaphore`, `MonadCancel`/`MonadError`, fiber lifecycle, `IORuntime` config.
**Triggers:** CE3 effect code. **Does not cover** fs2 (`fs2-specialist`) or http4s (`http4s-specialist`).

#### `fs2-specialist`
fs2 streams — `Stream`/`Pipe`/`Pull`, chunking, concurrency primitives, scoped resources, error handling, parallel evaluation, topic/queue patterns.
**Triggers:** Any fs2 code. If it's fs2-kafka, prefer `fs2-kafka-specialist`.

#### `http4s-specialist`
http4s — `HttpRoutes`, middleware, `EntityEncoder`/`Decoder`, server (Ember/Blaze), client (`EmberClient`), error handling, route composition, content negotiation.
**Triggers:** http4s code. If the project uses Tapir with the http4s interpreter, also engage `tapir-specialist`.

#### `fs2-kafka-specialist`
fs2-kafka — `KafkaConsumer`/`KafkaProducer` with fs2 `Stream`, committable offsets, transactional producer, schema-registry codec wiring, partitioned streams.
**Triggers:** fs2-kafka code. For Avro/Proto codec specifics, also engage `avro4s-specialist` or `scalapb-specialist`.

### API design

#### `tapir-specialist`
Tapir endpoints — endpoint definition, input/output/error schemas, security, codec selection, schema derivation, OpenAPI/AsyncAPI/redoc generation.
**Triggers:** Any Tapir endpoint work. **Binding-agnostic** — for the interpreter (pekko-http/http4s/zio-http/Netty), also engage the relevant HTTP specialist.

### Codecs

#### `circe-specialist`
Circe — `Encoder`/`Decoder` derivation (semi-auto vs auto, magnolia, `derives` in Scala 3), custom codecs, JSON traversal/transformation via `Cursor`, parser selection (jawn/jackson), generic-extras config.
**Triggers:** Any Circe code.

#### `jsoniter-specialist`
jsoniter-scala — `JsonValueCodec` macros, `CodecMakerConfig` tuning, performance-oriented patterns, integration with Tapir/http4s/Pekko HTTP, custom codec implementations.
**Triggers:** jsoniter-scala code, especially performance-critical JSON paths.

#### `play-json-specialist`
Play JSON — `Reads`/`Writes`/`Format` derivation, `Json.format` macro, manual codecs, `JsPath` traversal, validation via `JsResult`, custom transformers.
**Triggers:** Any Play JSON code.

### Protocols

#### `scalapb-specialist`
ScalaPB and gRPC in Scala — `.proto` design (field numbers, evolution, `oneof`, well-known types), generated code usage, custom options, gRPC service definitions, integration with `pekko-grpc` / `fs2-grpc` / `zio-grpc`, `sealed_oneof` for ADTs, `java_conversions`.
**Triggers:** Anything touching `.proto` files or generated gRPC code.

#### `avro4s-specialist`
avro4s — `SchemaFor`/`Encoder`/`Decoder` derivation, schema evolution rules, `@AvroNamespace`/`@AvroAlias`/`@AvroDefault` annotations, union types, enum handling, Confluent / Apicurio schema registry integration.
**Triggers:** avro4s code or Avro schema design.

### Databases

#### `doobie-specialist`
Doobie — `sql`/`fr` interpolators, `ConnectionIO` composition, transactor configuration, `Read`/`Write`/`Get`/`Put` instances, custom mappings, streaming queries, fragment composition, transactional boundaries, error handling.
**Triggers:** Any Doobie code. CE-based, but used across all effect systems via interop.

#### `quill-specialist`
Quill — quoted DSL, `MappedEncoding`, schema mapping, dynamic vs compile-time queries, context selection (jdbc/cassandra/zio/pekko), naming strategies, lifting and infix.
**Triggers:** Quill code. Note macro implementation differs significantly between Scala 2 and Scala 3 — the agent handles both.

#### `slick-specialist`
Slick — `TableQuery`, lifted embedding, plain SQL, schema codegen, `DBIO` composition, transactional combinators, async API integration with `Future`/`IO`/`ZIO` via `slick-effect` or sttp adapters.
**Triggers:** Slick code. Scala 3 support is recent and has rough edges — agent will flag known issues.

### Testing

#### `scalatest-specialist`
ScalaTest — choice of style (`FunSuite`/`AnyFlatSpec`/etc.), matchers, fixtures, async testing, property-based testing via scalatestplus-scalacheck, mocking integration, `BeforeAndAfter` hooks.
**Triggers:** ScalaTest-based codebases.

#### `munit-specialist`
MUnit — `test`/`testFailing`, fixtures (`FunFixture`/`Fixture`), assertions, async tests via `Future`/`IO`, property-based testing via munit-scalacheck, integration with `cats-effect-munit` and `zio-munit`.
**Triggers:** MUnit-based codebases.

#### `weaver-specialist`
Weaver tests (CE-based effect-aware test framework) — `IOSuite`/`SimpleIOSuite`, shared resources, parallel execution model, expectations, integration with cats-effect.
**Triggers:** Weaver-based codebases.

#### `zio-test-specialist`
zio-test — `ZIOSpecDefault`, `test`/`suite`, assertions and `assert`, `TestEnvironment` (`TestClock`/`TestRandom`/`TestConsole`), `TestAspect` composition, sharing resources, property-based testing via `Gen`.
**Triggers:** zio-test-based codebases.

## Not in scope

These specialists deliberately stop at library boundaries. If your task is outside, you'll need to handle it yourself or with a different tool:

- **Database schema design / DDL** — `doobie`/`quill`/`slick` cover Scala-side query code, not table design or migrations.
- **Cassandra / Postgres operational concerns** — `pekko-persistence-specialist` handles the Pekko config layer; the database itself is out of scope.
- **Effect-system selection** — no agent will recommend Pekko vs ZIO vs CE. They coexist as peers.
- **Teaching Scala** — the plugin assumes you know the language. Specialists review and implement; they don't tutor.
- **Akka** — Akka is not Pekko. None of the Pekko specialists will help with Akka code.

## Tips

- **Be specific about the library.** "Format this JSON" is ambiguous; "Format this Circe `Json`" routes correctly.
- **Mention the effect type if relevant.** "Add a streaming endpoint" → ambiguous. "Add a streaming endpoint in zio-http" → unambiguous.
- **Chain explicitly when needed.** "Review this Tapir endpoint, then have the http4s specialist check the binding" forces the composition you want.
- **Use `codebase-explorer` first on unfamiliar repos.** It's cheap and prevents wrong-specialist routing.
