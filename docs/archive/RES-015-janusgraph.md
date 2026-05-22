---
doc_id: RES-015
project: Cadastre
title: JanusGraph Repository Research Report
status: Archive
inspection_date: 2026-05-18
related_cadastre_specs:
  - docs/nlspec/090-graph-projection-serving-and-backends.md
  - docs/nlspec/100-packages-supply-chain-and-activation.md
  - docs/nlspec/110-api-ux-health-errors-and-security.md
  - docs/nlspec/120-validation-fixtures-and-acceptance.md
  - docs/nlspec/domain.md
related_research:
  - docs/reference/research/RES-012-graph-backend-comparison.md
---

Repository: `JanusGraph/janusgraph`

Inspection target: default branch `master`, latest inspected commit `346f5a46a9b01c1979272f14de95435d4777c160`.

Source access limit: direct container cloning was unavailable because the execution environment could not resolve `github.com`. Repository evidence in this report comes from GitHub connector file fetches, GitHub connector search results, and selected rendered GitHub pages. The report does not claim exhaustive inspection of every source file. It reconstructs the architecture from root metadata, module POMs, representative source files, official repository documentation, CI workflows, configuration definitions, backend adapters, index adapters, server code, and schema-initialization code.

## 1. Executive summary

Confirmed from project documentation: JanusGraph presents itself as a highly scalable transactional graph database for very large graphs, with support for concurrent users, complex traversals, and analytic graph queries.[^1] Confirmed from source: in this repository, JanusGraph is implemented as a Maven multi-module Java project whose core module exposes a TinkerPop-facing graph API, builds an embedded graph database engine around transaction-scoped state, and delegates persistence to storage-manager SPIs and advanced search to index-provider SPIs.[^2]

Confirmed from source: the root POM identifies the inspected source version as `1.2.0-SNAPSHOT`, uses Maven aggregation with many JanusGraph modules, sets Apache TinkerPop integration through `tinkerpop.version=3.7.3`, and declares Java 8 compilation defaults with a Java 11 profile.[^2][^3][^5] The source includes a compatibility surface with Titan through root properties and configuration keys such as `titan.compatible-versions` and the Titan-compatible ID-store migration option.[^2][^21]

The most important architectural fact is that JanusGraph is two systems that share one implementation core:

1. Embedded JanusGraph, opened through `JanusGraphFactory`, where user code receives a `JanusGraph` instance and directly creates transactions, management transactions, queries, mutations, and commits.
2. JanusGraph Server, a wrapper around Apache TinkerPop Gremlin Server, where graph instances are configured in server YAML, exposed through Gremlin remoting, and optionally accompanied by a JanusGraph gRPC management surface.[^10][^24][^25]

Confirmed from source: the public `JanusGraph` interface extends TinkerPop `Transaction`, declares TinkerPop compliance opt-ins and explicit opt-outs, and exposes graph-specific operations such as `newTransaction`, `buildTransaction`, `openManagement`, `close`, `isOpen`, and `getDBCacheInvalidationService`.[^9] Confirmed from source: the concrete core runtime is centered on `StandardJanusGraph`, which owns configuration, `Backend`, ID assignment, serialization, index serialization, schema cache, management logging, traversal strategies, and open transactions.[^13]

The core write path is transaction-centric. `StandardJanusGraphTx` holds the transaction-local vertex cache, added and deleted relations, index cache, lock state, and new type cache. On commit it delegates modified relations to `StandardJanusGraph.commit`; the graph then prepares relation mutations, composite-index updates, mixed-index updates, transaction logs, storage commits, and secondary index/user-log commits.[^13][^14]

Confirmed from source: the backend boundary is explicit. `Backend` orchestrates primary key-column-value storage, index providers, log managers, caches, ID authority, locking, scanner jobs, global/user configuration stores, and wrapped `BackendTransaction` objects.[^15] Storage adapters must satisfy `KeyColumnValueStoreManager` and `KeyColumnValueStore`; index adapters must satisfy `IndexProvider`.[^17]

A developer modifying JanusGraph must understand these boundaries first:

- Public API and TinkerPop integration live mainly in `janusgraph-core`, with server and remote access layered above it.
- Schema, ID allocation, relation serialization, query planning, index selection, cache behavior, and commit coordination are core concerns and must not be changed as isolated backend details.
- Storage backends implement persistence shape and capabilities, but the core decides relation layout, index-update semantics, locking requirements, and transaction flow.
- Mixed-index adapters implement external query and mutation semantics for Elasticsearch, Solr, and Lucene, but the core decides when a mixed index is used and how element IDs and property fields are translated.
- Server mode is not a separate database implementation. It is a deployment and remote-access layer around graph instances managed by TinkerPop and JanusGraph manager code.

Source-grounded inference: the repository is intentionally organized to make backend and index adapters replaceable without letting them define the graph model. The graph model is centralized in core interfaces, schema/type classes, serializers, and transaction logic, while backends supply storage capabilities and operational integrations.

## 2. Source inventory and research method

The research method was source-first. Repository files were treated as primary evidence, official documentation in the repository as secondary evidence, and external or rendered GitHub information only as support for repository identity and freshness. The report marks findings using these labels:

- Confirmed from source: directly supported by Java, POM, YAML, script, workflow, or generated-contract source in the repository.
- Confirmed from project documentation: directly supported by repository documentation such as `README.md`, `BUILDING.md`, `TESTING.md`, or `docs/**`.
- Source-grounded inference: inferred from source structure, dependencies, control flow, class names, tests, and configuration, but not explicitly stated in a single source line.
- Not determined from inspected sources: not resolved from the inspected material.

The following evidence table summarizes the inspected source areas. The table lists evidence paths rather than footnote markers; formal source citations appear in the prose and final Sources section.

| Source area or file path | Purpose | Architectural relevance | Confidence |
| --- | --- | --- | --- |
| `README.md` | Project identity and high-level claims | Establishes documented intent and user-facing positioning | High for documentation claims |
| Root `pom.xml` | Maven aggregator, module list, dependency management, plugin enforcement, profiles | Defines build graph and dependency architecture | High |
| Module POMs for core, server, driver, gRPC | Module responsibilities and intermodule dependencies | Establishes API/server/remote-access layering | High |
| Module POMs for storage backends | Backend dependency footprints and test profiles | Establishes adapter families and operational assumptions | High |
| Module POMs for index backends | Search/mixed-index dependency footprints | Establishes indexing subsystem boundaries | High |
| `JanusGraph`, `JanusGraphFactory`, `ConfiguredGraphFactory`, `JanusGraphManagement` | Public API and factory/management surfaces | Establishes embedded opening, dynamic server graph opening, and schema-management contracts | High |
| `StandardJanusGraph`, `StandardJanusGraphTx` | Core runtime and transaction implementation | Establishes control flow for transactions, query execution, commit, logs, indexes, and caches | High |
| `Backend`, `BackendTransaction` | Storage/index orchestration and transaction proxy | Establishes backend boundary and read/write behavior | High |
| Storage and index SPI interfaces | Required contracts for custom adapters | Establishes extension points | High |
| Representative storage adapters | CQL, Scylla, HBase, BerkeleyJE, in-memory behavior | Establishes adapter divergence and operational assumptions | Medium-high |
| Representative index adapters | Elasticsearch, Solr, Lucene behavior and feature surfaces | Establishes search-backend divergence | Medium-high |
| `GraphDatabaseConfiguration` | Configuration keys, defaults, mutability, operational risks | Establishes configuration surface | High for inspected options |
| Schema initialization source | Startup schema initialization and JSON strategy | Establishes schema-init workflow and edge cases | High |
| Server source and docs | Gremlin Server wrapping, gRPC enablement, server startup and environment | Establishes remote and operational model | High |
| Build, testing, CI, distribution docs | Developer workflow, test taxonomy, release and package assumptions | Establishes validation and packaging model | High |

Important areas not fully resolved:

- Not determined from inspected sources: the full contents of every generated gRPC class and every proto file were not inspected. The report uses gRPC POM generation and the `graph_manager.proto` contract as representative evidence.
- Not determined from inspected sources: every test fixture, script, Dockerfile, and distribution configuration file was not read line by line. The report uses distribution documentation, server documentation, CI workflow, module POM test dependencies, and representative server files.
- Not determined from inspected sources: exact runtime behavior of every storage backend feature flag across all environment combinations was not exhaustively traced. The report uses each representative adapter’s POM and key manager class where inspected.

## 3. Repository identity and top-level layout

Confirmed from source: the root POM declares `groupId=org.janusgraph`, `artifactId=janusgraph`, `packaging=pom`, and `version=1.2.0-SNAPSHOT`.[^2] Confirmed from source: it aggregates these JanusGraph modules: `janusgraph-grpc`, `janusgraph-driver`, `janusgraph-core`, `janusgraph-server`, `janusgraph-backend-testutils`, `janusgraph-test`, `janusgraph-inmemory`, `janusgraph-berkeleyje`, `cassandra-hadoop-util`, `scylla-hadoop-util`, `janusgraph-cql`, `janusgraph-hadoop`, `janusgraph-hbase`, `janusgraph-bigtable`, `janusgraph-es`, `janusgraph-lucene`, `janusgraph-all`, `janusgraph-dist`, `janusgraph-benchmark`, `janusgraph-doc`, `janusgraph-solr`, `janusgraph-examples`, `janusgraph-mixed-index-utils`, and `janusgraph-scylla`.[^2]

### Module-family classification

| Module or directory | Category | Likely responsibility | Key dependencies | Key exported or consumed abstractions | Runtime classification |
| --- | --- | --- | --- | --- | --- |
| `janusgraph-core` | Core graph database engine | Public API, transactions, schema, query planning, serialization, backend orchestration | TinkerPop, JanusGraph driver, metrics, configuration, cache libraries | `JanusGraph`, `JanusGraphFactory`, `StandardJanusGraph`, `StandardJanusGraphTx`, `Backend`, schema and query classes | Runtime-critical |
| `janusgraph-driver` | Remote/driver support | Gremlin-driver library additions and JanusGraph client-side utilities | TinkerPop Gremlin core/driver/groovy, spatial and JSON utilities | Driver-facing graph/serialization utilities | Runtime for clients and core dependency |
| `janusgraph-grpc` | gRPC generated-contract module | Protobuf/gRPC service definitions and generated Java code | `protobuf-java`, `grpc-protobuf`, `grpc-stub` | `JanusGraphManagerService` proto contracts and generated service bindings | Runtime for optional gRPC server/client surfaces |
| `janusgraph-server` | Server and remote access | JanusGraph Server wrapper around Gremlin Server and optional gRPC services | Core, gRPC module, Gremlin Server, gRPC runtime | `JanusGraphServer`, `JanusGraphSettings`, gRPC service implementations | Runtime-critical for server mode |
| `janusgraph-cql` | Storage backend adapter | Cassandra-compatible CQL storage | Core, Cassandra Java driver, optional Hadoop utility, Testcontainers | `CQLStoreManager`, CQL key-column-value stores | Integration-specific runtime |
| `janusgraph-scylla` | Storage backend adapter | Scylla-specific CQL derivative | CQL module with Cassandra driver exclusions, Scylla driver | `ScyllaStoreManager` extending CQL manager | Integration-specific runtime |
| `janusgraph-hbase` | Storage backend adapter | HBase storage and optional Hadoop integration | Core, HBase shaded client/mapreduce, test infrastructure | `HBaseStoreManager` and HBase KCV stores | Integration-specific runtime |
| `janusgraph-bigtable` | Storage backend adapter | Google Cloud Bigtable HBase-compatible dependency layer | Bigtable HBase shaded client | Bigtable/HBase compatibility surface | Integration-specific runtime; implementation detail not fully resolved |
| `janusgraph-berkeleyje` | Storage backend adapter | Local Berkeley DB Java Edition storage | Core, Sleepycat JE | `BerkeleyJEStoreManager`, ordered key-value stores | Runtime for local/persistent mode |
| `janusgraph-inmemory` | Storage backend adapter | In-memory storage manager | Core | `InMemoryStoreManager` | Runtime for tests/dev and possible ephemeral embedded use |
| `janusgraph-es` | Mixed-index adapter | Elasticsearch-backed mixed indexes | Core, mixed-index utils, Elasticsearch REST client, Testcontainers | `ElasticSearchIndex` | Integration-specific runtime |
| `janusgraph-solr` | Mixed-index adapter | Solr/SolrCloud-backed mixed indexes | Core, SolrJ, Lucene, Kerberos test utilities, Testcontainers | `SolrIndex` | Integration-specific runtime |
| `janusgraph-lucene` | Mixed-index adapter | Local Lucene-backed mixed indexes | Core, Lucene modules | `LuceneIndex` | Runtime for local search indexing |
| `janusgraph-mixed-index-utils` | Mixed-index support | Shared mixed-index utilities | Core, Lucene core | Utility classes for mixed-index behavior | Runtime support |
| `janusgraph-hadoop` | OLAP/Hadoop integration | Hadoop/GraphComputer related utilities | TinkerPop Hadoop/Spark families as managed dependencies | Hadoop-backed analysis and scan support | Integration-specific runtime |
| `cassandra-hadoop-util`, `scylla-hadoop-util` | Backend utility modules | Hadoop utility support for CQL/Scylla families | Backend-specific Hadoop dependencies | Utility classes consumed by storage/Hadoop modules | Integration-specific support |
| `janusgraph-backend-testutils` | Test infrastructure | Shared backend test utilities and category definitions | Test dependencies | Feature flags, backend test scaffolding | Test-only |
| `janusgraph-test` | Test module | Cross-module tests and TinkerPop conformance support | Core/backend modules and test dependencies | Test suites | Test-only |
| `janusgraph-benchmark` | Benchmarking | Performance and benchmark workloads | Benchmark frameworks and JanusGraph modules | Benchmark harnesses | Benchmark-only |
| `janusgraph-examples` | Examples | Executable examples and demonstration code | Core/server/backend modules | Sample configurations and usage | Example/documentation support |
| `janusgraph-doc`, `docs` | Documentation | MkDocs sources and generated/reference docs | Documentation tooling | User/operator/developer documentation | Documentation-oriented |
| `janusgraph-dist` | Distribution and packaging | ZIP archive, package tests, scripts, Docker image context | Aggregated runtime artifacts | Distribution layout and scripts | Distribution-only/runtime packaging |
| `janusgraph-all` | Aggregation/shading style module | Combined dependency/artifact boundary | Many JanusGraph modules | Aggregated artifact | Build/distribution boundary |

Confirmed from source: the module classifications above are consistent with the root POM module list and module-level POM dependencies.[^2][^6][^7][^8]

## 4. Maven, dependency, and build architecture

Confirmed from source: the root POM is the Maven aggregator and parent. It declares the Maven prerequisite `3.2.5`, Java compiler defaults of source/target `1.8`, a Java 11 profile, dependency-management entries, and plugin management for compiler, surefire, failsafe, release, RAT, checkstyle, CycloneDX, and enforcer behavior.[^2][^3][^4][^5]

Confirmed from source: the root dependency model centralizes third-party versions. Major dependency families include Apache TinkerPop, logging, Jackson, Netty, Dropwizard Metrics, gRPC/protobuf, Elasticsearch client, Solr/Lucene, HBase/Hadoop, Cassandra/Scylla, Testcontainers, JUnit, Mockito, Caffeine, spatial libraries, and Guava.[^2][^3][^4][^5]

Confirmed from source: dependency enforcement includes Maven Enforcer rules for upper-bound dependencies and dependency convergence, with explicit excludes for some Jackson artifacts. The root POM also includes RAT checks, checkstyle configuration, and CycloneDX aggregate BOM generation.[^2][^3]

### Build commands and workflow

Confirmed from project documentation: `BUILDING.md` states that Java 8 and Maven 3 are required, `mvn clean install -DskipTests=true` builds without tests, `mvn clean install` builds with default tests, `mvn clean install -Dtest.skip.tp=false` adds TinkerPop tests, and `mvn clean install -Pjanusgraph-release -Dgpg.skip=true -DskipTests=true` builds a distribution archive under `janusgraph-dist/target/janusgraph-$VERSION.zip`.[^26]

Confirmed from source: the root POM’s surefire configuration separates default tests from TinkerPop test execution. TinkerPop tests are included through an explicit `tinkerpop-test` execution and controlled by `test.skip.tp`.[^3]

Confirmed from source and project documentation: the CI workflow uses GitHub Actions, builds `janusgraph-all`, verifies with coverage, tests core modules across Java 8 and Java 11 matrices, and runs module-specific verification for core, driver, server, test, inmemory, BerkeleyJE, and Lucene.[^26]

### Build structure and runtime architecture relationship

Source-grounded inference: the Maven module graph reinforces the runtime architecture. Core is upstream of backend and index adapters. Server depends on core and gRPC. Storage and index modules depend on core but do not own public graph semantics. Distribution modules depend on built artifacts and scripts. Test utility modules sit beside backend modules to normalize integration testing.

The Maven layout is not merely packaging. It encodes architectural separations:

- Core owns graph semantics, transaction semantics, schema, query planning, and SPI definitions.
- Backends plug into the key-column-value storage abstraction.
- Mixed indexes plug into the index-provider abstraction.
- Server and gRPC expose core graph instances remotely.
- Distribution builds an executable deployment surface from the modules.

## 5. Architectural overview

Confirmed from source: `StandardJanusGraph` registers JanusGraph traversal strategies into TinkerPop’s traversal strategy registry and initializes the concrete runtime around configuration, `Backend`, ID management, serialization, schema cache, index serializer, management logger, and optional Gremlin script engine.[^13]

### Layered model

```text
Caller / Gremlin traversal / JanusGraph API
  -> TinkerPop-facing public API (`JanusGraph`, traversal strategies, driver/server surfaces)
  -> Graph-opening layer (`JanusGraphFactory`, `ConfiguredGraphFactory`, `GraphFactoryUtils`)
  -> Graph configuration (`GraphDatabaseConfiguration`, local properties, global storage-backed config)
  -> Concrete graph runtime (`StandardJanusGraph`)
  -> Transaction state (`StandardJanusGraphTx`, transaction builder, vertex and relation caches)
  -> Schema and management (`JanusGraphManagement`, `ManagementSystem`, schema cache, management log)
  -> Query planning and execution (`GraphCentricQueryBuilder`, `VertexCentricQueryBuilder`, index selection, processors)
  -> Serialization and ID handling (`EdgeSerializer`, `IndexSerializer`, `IDManager`, relation identifiers)
  -> Backend abstraction (`Backend`, `BackendTransaction`)
  -> Primary storage SPI (`KeyColumnValueStoreManager`, `KeyColumnValueStore`)
  -> Mixed-index SPI (`IndexProvider`, `IndexTransaction`)
  -> Adapter implementations (CQL, Scylla, HBase, BerkeleyJE, in-memory; Elasticsearch, Solr, Lucene)
  -> Operational wrapper (`JanusGraphServer`, Gremlin Server, optional gRPC services, distribution scripts)
```

### Embedded versus server usage

Confirmed from source: embedded graph opening uses `JanusGraphFactory.open(...)`. That path accepts a file path, shorthand, Apache Commons `Configuration`, `ReadConfiguration`, or builder, loads local configuration, optionally registers the graph with `JanusGraphManager` when `graph.graphname` is present, and constructs the graph through schema initialization before returning it.[^10]

Confirmed from source: server usage constructs `JanusGraphServer`, reads `JanusGraphSettings` from YAML, creates a TinkerPop `GremlinServer`, optionally creates a gRPC server when enabled, starts Gremlin Server, and configures the JanusGraph manager after the TinkerPop server starts.[^24]

Confirmed from project documentation: JanusGraph Server uses the Gremlin Server engine as its server component, provides remote execution of Gremlin traversals against one or more hosted JanusGraph instances, and can be configured for WebSocket and HTTP endpoint interactions.[^25]

## 6. Core abstractions and internal object model

### Abstraction map

| Abstraction | Path | Responsibility | Depends on | Depended on by | API class |
| --- | --- | --- | --- | --- | --- |
| `JanusGraph` | `janusgraph-core/src/main/java/org/janusgraph/core/JanusGraph.java` | Public graph interface, transaction entry points, management entry point, lifecycle | TinkerPop graph/transaction concepts | User code, `StandardJanusGraph`, server bindings | Public API |
| `JanusGraphFactory` | `janusgraph-core/.../JanusGraphFactory.java` | Embedded graph opening, local config loading, shorthand parsing, graph drop, transaction recovery helpers | Configuration classes, `GraphFactoryUtils`, `JanusGraphManager`, `Backend` | User code, server graph opening | Public API |
| `ConfiguredGraphFactory` | `janusgraph-core/.../ConfiguredGraphFactory.java` | Dynamically create/open/drop graphs by stored configuration | `ConfigurationManagementGraph`, `JanusGraphManager`, `JanusGraphFactory` | Server-managed dynamic graph workflows | Public API for server-style graph management |
| `JanusGraphManagement` | `janusgraph-core/.../schema/JanusGraphManagement.java` | Schema, graph indexes, relation indexes, consistency, management transactions | Transaction and schema concepts | `ManagementSystem`, user schema code | Public API |
| `StandardJanusGraph` | `janusgraph-core/.../database/StandardJanusGraph.java` | Concrete graph runtime, transaction creation, backend operations, commit coordination | Configuration, backend, serializers, schema cache, ID manager | Transactions, factories, server | Implementation detail with core importance |
| `StandardJanusGraphTx` | `janusgraph-core/.../transaction/StandardJanusGraphTx.java` | Transaction-scoped graph operations, caches, query processors, commit/rollback | `StandardJanusGraph`, backend transaction, query builders | User operations during transaction | Internal implementation/API bridge |
| `GraphDatabaseConfiguration` | `janusgraph-core/.../configuration/GraphDatabaseConfiguration.java` | Defines configuration namespace, defaults, mutability classes, setup logic | Config classes, backend/index enum classes | Graph construction, backend construction, docs generation | Core configuration contract |
| `Backend` | `janusgraph-core/.../diskstorage/Backend.java` | Orchestrates storage, indexing, logs, ID authority, caches, locks, scanner, backend transactions | Storage and index SPI, configuration | `StandardJanusGraph` | Internal backend coordinator |
| `BackendTransaction` | `janusgraph-core/.../diskstorage/BackendTransaction.java` | Wraps storage and index transactions; exposes read/write proxy operations | Cache transaction, KCVS caches, index transactions | `StandardJanusGraphTx`, `StandardJanusGraph` | Internal backend transaction facade |
| `KeyColumnValueStoreManager` | `janusgraph-core/.../keycolumnvalue/KeyColumnValueStoreManager.java` | Storage manager SPI for stores and multi-store mutations | Store and transaction classes | Storage adapters, `Backend` | SPI |
| `KeyColumnValueStore` | `janusgraph-core/.../keycolumnvalue/KeyColumnValueStore.java` | Row/column/value storage operations | Static buffers, entries, transactions | Storage adapters, caches | SPI |
| `IndexProvider` | `janusgraph-core/.../indexing/IndexProvider.java` | External/mixed-index SPI | Index query/mutation information | Index adapters, `Backend`, `IndexTransaction` | SPI |
| `EdgeSerializer` | `janusgraph-core/.../database/EdgeSerializer.java` | Relation storage serialization/deserialization | Serializer, type inspector, ID handler | Commit/read path | Internal core implementation |
| `IndexSerializer` | `janusgraph-core/.../database/IndexSerializer.java` | Composite and mixed-index update/query translation | Edge serializer, serializer, index metadata | Query planner, commit path | Internal core implementation |

Confirmed from source: these abstractions and their roles are directly visible in the inspected class definitions and constructors.[^9][^10][^11][^12][^13][^14][^15][^16][^17][^22][^23]

### Runtime participation

`JanusGraphFactory` is the primary embedded entry point. It resolves configuration, decides whether graph names are registered with the manager, invokes schema initialization, and returns a graph instance.[^10]

`StandardJanusGraph` is the concrete runtime root. It owns the `Backend`, `IDManager`, `VertexIDAssigner`, `IndexSerializer`, `EdgeSerializer`, `Serializer`, query caches, schema caches, management logger, index selector, and open transaction set.[^13]

`StandardJanusGraphTx` is the mutable execution context. It holds vertex and subquery caches, added/deleted relations, new index entries, unique locks, and query processors. It is responsible for local transaction-visible reads, query execution, and commit/rollback delegation.[^14]

`Backend` translates graph configuration into physical storage and index provider instances. It opens fixed stores such as `edgestore` and `graphindex`, configures caches, creates ID authority, opens system/user configs and logs, and returns `BackendTransaction` objects.[^15]

`BackendTransaction` is the transaction facade over primary storage and external index transactions. It provides mutation methods for edges and composite indexes, lock acquisition, edge-store reads, index reads, raw mixed-index queries, aggregation queries, storage commit, index commit, and rollback.[^16]

## 7. Data model and graph representation

Confirmed from source: JanusGraph’s public graph model is TinkerPop-facing, but the internal data model is a JanusGraph-specific property graph representation with schema types, ID management, key-column-value storage, relation serialization, composite indexes, mixed indexes, and vertex-centric indexes.

### Conceptual data model

| Concept | TinkerPop-level meaning | JanusGraph-specific storage/schema meaning |
| --- | --- | --- |
| Vertex | Graph element with identity and incident edges/properties | Stored as an ID-keyed row whose relations are represented as entries in the edge store |
| Edge | Directed relation between vertices | Serialized relation entry with label/type ID, direction encoding, relation ID, adjacent vertex ID, sort/signature properties, and metadata |
| Vertex property | Property value on a vertex | Serialized as a relation entry using a property-key relation type and value encoding |
| Edge label | Name/type of an edge | Schema relation type with multiplicity and sort/signature behavior |
| Property key | Name/type of property values | Schema relation type with data type and cardinality/multiplicity behavior |
| Vertex label | Type label for vertices | Schema type managed through management transactions |
| Composite index | JanusGraph-maintained key-column-value index | Stored in the `graphindex` store through `mutateIndex` calls |
| Mixed index | External index-backed query support | Delegated to configured `IndexProvider` transactions |
| Vertex-centric index | Relation-type-local index | Schema-managed index on a relation type for efficient adjacency queries |

Confirmed from source: cardinality has three JanusGraph values: `SINGLE`, `LIST`, and `SET`; it also converts to and from TinkerPop `VertexProperty.Cardinality`.[^22] Confirmed from source: multiplicity supports `MULTI`, `SIMPLE`, `ONE2MANY`, `MANY2ONE`, and `ONE2ONE`, and maps property cardinalities onto multiplicity variants for storage/type semantics.[^22]

### Key-column-value storage model

Confirmed from source: the storage SPI describes the data store as a BigTable-like representation with rows identified by keys and column-value pairs in each row, where subsets of column-value pairs can be retrieved by column intervals.[^17] Confirmed from source: `Backend` opens fixed stores named `edgestore` and `graphindex`, and comments state that the edge store contains all edges and properties while the property index contains an inverted index from attribute value to vertex.[^15]

### Relation serialization

Confirmed from source: `EdgeSerializer` parses relation entries by reading relation type and direction, resolving the relation type through the transaction/type inspector, applying multiplicity, reading adjacent vertex IDs or property values, reading relation IDs, reading sort keys/signatures/inline properties, and converting entry metadata into implicit keys when present.[^22]

Confirmed from source: `EdgeSerializer.writeRelation` writes the relation type and direction ID, sort-key values for unconstrained multiplicity, adjacent vertex ID or property value, relation ID, signature properties, remaining relation properties, and value-position metadata.[^22]

Source-grounded inference: JanusGraph stores both edges and vertex properties as relations in a row-oriented adjacency-list layout. This allows vertex-centric retrieval through slice queries while preserving schema-specific ordering and uniqueness behavior.

### ID assignment and encoding

Confirmed from source: configuration exposes ID block allocation size, renew timeout, renew percentage, conflict-avoidance mode, conflict-avoidance tag bits, conflict-avoidance tags, and a Titan-compatible ID-store-name option.[^21] Confirmed from source: `Backend` creates a `ConsistentKeyIDAuthority` when the backend is key-consistent, and otherwise rejects storage that cannot guarantee proper ID allocations.[^15]

Not determined from inspected sources: the full bit-level ID encoding contract in `IDManager` and `IDHandler` was not exhaustively inspected in this pass. The report relies on configuration and serializer evidence for ID behavior rather than claiming a complete binary encoding map.

## 8. Schema system, validation, and initialization

Confirmed from source: `JanusGraphManagement` defines the management transaction contract for schema creation, schema lookup, graph indexes, relation indexes, consistency modifiers, schema constraints, and commit/rollback.[^12] Confirmed from source: `GraphDatabaseConfiguration` defines automatic schema behavior through `schema.default` and exposes shorthands `default`, `tp3`, `none`, and `ignore-prop`, with a custom `DefaultSchemaMaker` class path allowed.[^21]

Confirmed from source: `schema.constraints` defaults to `false`; when enabled and `schema.default=none`, violations throw an `IllegalArgumentException`; when enabled with automatic schema creation, constraints are automatically created as described by the default schema maker.[^21]

Confirmed from source: schema initialization is a startup path. `GraphDatabaseConfiguration` defines `schema.init.strategy`, `schema.init.drop-before-startup`, JSON input file/string options, skip flags for elements and indexes, index activation mode, force-close-other-instances, and index-status await timeout. Defaults include strategy `none`, drop-before-startup `false`, JSON skip-elements `false`, JSON skip-indices `false`, JSON activation type `REINDEX_AND_ENABLE_NON_ENABLED`, force-close-other-instances `false`, and await-index-status-timeout of three minutes.[^21]

Confirmed from source: `SchemaInitializationManager` optionally drops schema/data before initialization, resolves the selected strategy by shorthand or class name, runs the strategy, and constructs `StandardJanusGraph` when the strategy returns null.[^28] Confirmed from source: `JsonSchemaInitStrategy` opens `StandardJanusGraph`, requires either JSON string or file when JSON initialization is enabled, closes the graph on initialization failure, can force-close other instances and roll back active transactions, commits created schema, then processes index activation through reindex/enable or force-enable behavior.[^28]

### Schema concepts

| Concept | Source path or class | Purpose | Validation behavior | Operational implications |
| --- | --- | --- | --- | --- |
| Management transaction | `JanusGraphManagement` / `ManagementSystem` | Isolates schema mutations from normal graph transactions | Must commit or rollback | Schema changes can affect all graph instances and index status |
| Default schema maker | `GraphDatabaseConfiguration.AUTO_TYPE` | Controls implicit type creation | Validates shorthand or class implementing `DefaultSchemaMaker` | Choosing implicit schema can hide modeling errors; disabling it requires explicit schema |
| Schema constraints | `GraphDatabaseConfiguration.SCHEMA_CONSTRAINTS` | Enforces schema constraints | Defaults to disabled | Enabling constraints changes mutation validation behavior |
| JSON schema initialization | `JsonSchemaInitStrategy` | Initializes schema from file or string | Throws when JSON input is missing under JSON strategy | Useful for reproducible deployments; index activation can reindex existing data |
| Index activation | `IndicesActivationType` used by JSON strategy | Determines reindex/enable behavior | Unknown activation type throws | Can trigger expensive reindexing or force-enable indexes without reindexing |
| Force-close other instances | JSON schema-init option | Removes other graph instances before initialization | Defaults false | Dangerous operation intended for zombie-instance cases |
| Drop-before-startup | Schema init option | Drops schema and graph data before initialization | Defaults false | Destructive; applies regardless of selected init strategy |

## 9. Runtime flow

### Opening a graph with `JanusGraphFactory`

Confirmed from source:

1. Entry point: `JanusGraphFactory.open(String)` or related overload.
2. The string argument is interpreted as a file path when readable; otherwise it can be interpreted as a backend shorthand such as `[BACKEND]:[ARG]`.
3. Relative storage and index directory/config-file settings in a properties file are rewritten relative to the config file directory.
4. If `graph.graphname` is present, the graph is registered/opened through `JanusGraphManager` and schema initialization; otherwise it is opened without manager tracking.
5. `drop(JanusGraph)` removes the graph/traversal from manager if present, closes the graph, opens its backend, clears storage, and closes the backend. The source warns this is irreversible.[^10]

Failure modes visible from source:

- Missing or unreadable local configuration file raises `IllegalArgumentException`.
- Unknown backend shorthand raises `IllegalArgumentException`.
- `drop` requires a closed or closable JanusGraph instance and can throw backend exceptions during clear.

### Opening or creating a graph with `ConfiguredGraphFactory`

Confirmed from source:

1. `ConfiguredGraphFactory.create(name)` requires a template configuration and no existing graph configuration.
2. It copies the template, sets graph-name metadata, stores the graph configuration, and opens the graph.
3. `open(name)` resolves stored configuration, opens through `JanusGraphManager`, and returns the graph.
4. `drop(name)` removes configuration, evicts graph state, and calls `JanusGraphFactory.drop`.[^11]

Configuration dependency: the code expects a configuration-management graph to be present in the server context under the key `ConfigurationManagementGraph`.[^11]

### Creating a transaction

Confirmed from source:

1. `StandardJanusGraph.newTransaction()` delegates to `buildTransaction().start()`.
2. `newTransaction(TransactionConfiguration)` checks the graph is open, creates `StandardJanusGraphTx`, begins a backend transaction through `Backend.beginTransaction`, attaches it to the transaction, tracks open transactions, and returns it.[^13]
3. `StandardJanusGraphTx` initializes transaction-local caches, added/deleted relation containers, index caches, lock maps, and behavioral flags based on transaction configuration.[^14]

Failure modes visible from source:

- Opening a transaction on a closed graph fails.
- Read-only transactions reject write verification.
- Dirty vertex cache sizing and vertex cache behavior depend on transaction config.

### Executing a traversal or graph query

Confirmed from source: `StandardJanusGraph` registers JanusGraph traversal optimization strategies with TinkerPop, and `StandardJanusGraphTx.query()` returns a `GraphCentricQueryBuilder` with `IndexSerializer` and index selection strategy.[^13][^14]

Confirmed from source: graph-centric query execution uses `JointIndexQuery` when available. It streams from the first index query, prepares remaining subqueries for intersection, and uses `IndexSerializer.query` through `indexCache`. If no suitable index exists and `query.force-index` is enabled, it throws; otherwise it logs a warning and executes a full scan over vertices, edges, or properties.[^14]

Configuration dependencies:

- `query.force-index` controls whether non-indexed graph scans are allowed.
- `query.index-select-strategy` controls index selection behavior.
- `query.batch.*` options control traversal multi-query and property prefetching behavior.[^21]

### Reading vertices, edges, and properties

Confirmed from source: vertex-centric query execution loads relations with `graph.edgeQuery(v.id(), sliceQuery, txHandle)` and turns `EntryList` results into relations through `RelationConstructor.readRelation`. Multi-query paths collect vertex IDs, call `graph.edgeMultiQuery`, and load results into `CacheVertex` relation caches.[^14]

Confirmed from source: `BackendTransaction.edgeStoreQuery` chooses cached or no-cache reads based on transaction cache settings and wraps backend reads in retry logic with `BackendOperation.execute` and configured maximum read time.[^16]

### Writing mutations and committing

Confirmed from source:

1. `StandardJanusGraphTx.commit()` checks open state, increments metrics when configured, and delegates modified relation sets to `StandardJanusGraph.commit`; otherwise it commits the backend transaction directly.[^14]
2. `StandardJanusGraph.commit` sets a commit timestamp, assigns IDs, optionally writes transaction-log precommit entries, prepares mutation containers, writes primary storage, commits storage, commits secondary index/user-log operations, writes log status, and rolls back on exceptions.[^13]
3. Composite-index updates are translated into `mutateIndex` calls against the graph index store. Mixed-index updates are applied through named `IndexTransaction` objects.[^13][^23]

Failure modes visible from source:

- Commit exceptions trigger backend rollback attempts; rollback failure after commit failure becomes a JanusGraph exception.[^14]
- Storage and index commits are coordinated but not a single cross-system distributed transaction; the presence of write-ahead transaction logging and recovery helpers indicates explicit handling for partially failed transactions.[^10][^13][^21]

### Server-side request handling

Confirmed from source and documentation: JanusGraph Server wraps TinkerPop Gremlin Server. It reads YAML settings, starts Gremlin Server, optionally starts a gRPC server on the configured gRPC port, and then configures a JanusGraph-specific graph manager if the TinkerPop graph manager is a `JanusGraphManager`.[^24][^25]

## 10. Storage backend architecture

Confirmed from source: storage backends are selected by `storage.backend`, which can be a registered shorthand or a fully qualified custom `StoreManager` implementation. Registered store managers include BerkeleyJE, CQL, HBase, in-memory, and Scylla.[^18][^21]

Confirmed from source: the primary storage contract is the key-column-value store. A manager opens named stores and mutates multiple stores; a store provides slice reads, multi-slice reads, mutations, lock acquisition, key scans, and close.[^17]

### Backend comparison

| Backend family | Module path | Adapter responsibility | Storage abstraction | Dependency footprint | Operational assumptions | Test strategy evidence | Notable implementation differences |
| --- | --- | --- | --- | --- | --- | --- | --- |
| CQL/Cassandra | `janusgraph-cql` | Cassandra-compatible distributed storage | `KeyColumnValueStoreManager` via `CQLStoreManager` | Cassandra Java driver, optional Hadoop utility, Testcontainers | Remote CQL hosts, keyspace creation, replication strategy, back pressure, metrics | Cassandra 3/4, Murmur/byteordered, SSL/client auth profiles | Computes default back-pressure limit from driver profile and node count; initializes keyspace if absent |
| Scylla | `janusgraph-scylla` | Scylla-specific CQL storage variant | `ScyllaStoreManager` extends `CQLStoreManager` | Scylla driver, CQL module with Cassandra driver excluded | Scylla docker image/profile; same CQL core behavior by inheritance | Scylla profile in POM/test docs | Source class delegates to CQL manager; divergence is mainly dependency/driver/profile level |
| HBase | `janusgraph-hbase` | HBase distributed storage | `HBaseStoreManager` | HBase shaded client/mapreduce, optional Hadoop | HBase/ZooKeeper environment, table/CF creation unless schema check skipped | HBase Docker properties in tests | HBase-specific table, column-family, compression, region, snapshot options |
| Bigtable | `janusgraph-bigtable` | Google Cloud Bigtable HBase-compatible support | Not fully resolved from inspected source | Bigtable HBase 2.x shaded client | Source-grounded inference: relies on HBase compatibility | Not fully resolved | Only module POM inspected; no adapter class was resolved in this pass |
| BerkeleyJE | `janusgraph-berkeleyje` | Local persistent storage | Ordered key-value manager adapted to KCVS | Sleepycat JE | Local directory, transactional or non-transactional JE environment | Backend testutils and test-jar packaging | Ordered key-value store, local environment configuration, JE cache/isolation options |
| In-memory | `janusgraph-inmemory` | Ephemeral in-memory storage | `KeyColumnValueStoreManager` | Core only | Non-persistent stores in process memory | TinkerPop tests enabled in CI matrix | `persists(false)`, ordered/unordered scan support, snapshot/restore helpers |

Confirmed from source: the storage comparison is grounded in module POMs, registered manager classes, and representative manager implementations.[^7][^18][^19]

### Backend operational notes

- CQL and Scylla are distributed storage adapters and require remote-compatible operational configuration such as hosts, keyspace, replication, Docker/test versions, and driver configuration.[^7][^19]
- HBase exposes HBase-specific schema/table behavior, including column-family shortening, compression, schema-check skipping, table naming, snapshots, and region sizing.[^19]
- BerkeleyJE is local and persistent, uses Sleepycat JE environment/database configuration, and advertises ordered scans and transactional features through `StoreFeatures`.[^19]
- In-memory storage advertises `persists(false)`, which makes it unsuitable as durable production storage unless externally snapshotted and restored by custom operational logic.[^19]

## 11. Mixed-index and search backend architecture

Confirmed from source: external/mixed indexes are selected by `index.<name>.backend`, with built-in shorthands `lucene`, `elasticsearch`/`es`, and `solr`; a custom implementation can be supplied as a class implementing `IndexProvider`.[^18][^21]

Confirmed from source: `IndexProvider` defines index registration, mutation, restore, structured query, raw query, totals, aggregation, transaction creation, close, clear storage, clear store, and existence checks.[^17]

### Index-backend comparison

| Backend | Module | Adapter class | Dependency footprint | Query/mapping capabilities visible from source | Operational assumptions | Notable risks |
| --- | --- | --- | --- | --- | --- | --- |
| Elasticsearch | `janusgraph-es` | `ElasticSearchIndex` | Elasticsearch REST client, mixed-index utils, Testcontainers | Text, comparison, geo, mapping parameters, aggregations, raw query, REST client settings | Remote Elasticsearch cluster, REST client interface, health wait, SSL/auth options | Mapping/version differences across ES major versions; external index consistency |
| Solr | `janusgraph-solr` | `SolrIndex` | SolrJ, Lucene analyzers, Kerberos utilities, Testcontainers | Text/string dynamic-field suffixes, geo, comparison, SolrCloud/HTTP modes, Kerberos | Remote Solr/SolrCloud, ZooKeeper, HTTP settings | Dynamic schema vs explicit schema, SolrCloud collection creation, Kerberos complexity |
| Lucene | `janusgraph-lucene` | `LuceneIndex` | Lucene core/analyzers/queryparser/spatial | Text/string mappings, cardinality support, custom analyzer, nanoseconds, geo contains | Local filesystem index directory | Local disk durability and file permissions; not a distributed search cluster |
| Mixed-index utils | `janusgraph-mixed-index-utils` | Utility module | Core and Lucene core | Shared mixed-index helper behavior | Shared by adapters | Coupling risk if adapter-specific behavior leaks into utilities |

Confirmed from source: `ElasticSearchIndex`, `SolrIndex`, and `LuceneIndex` implement `IndexProvider` and define backend-specific configuration and feature behavior.[^20]

### Composite versus mixed indexes

Confirmed from source: composite index updates use the internal `graphindex` key-column-value store through `mutateIndex`, while mixed-index updates are routed to configured `IndexTransaction` objects backed by external `IndexProvider` implementations.[^13][^16][^23]

Source-grounded inference: composite indexes are part of JanusGraph’s primary storage model and participate in backend storage mutation coordination; mixed indexes are external-system integrations and introduce cross-system consistency and operational complexity.

## 12. Query planning, traversal, and optimization

Confirmed from source: JanusGraph integrates with TinkerPop traversal optimization by registering strategies such as adjacent-vertex filtering, local query optimization, has-step optimization, multi-query, mixed-index aggregation/count, JanusGraph step strategy, and IO registration.[^13]

Confirmed from source: `GraphDatabaseConfiguration` exposes query configuration for force-index behavior, property prefetching, smart limits, hard max limits, index-selection strategy, optimizer backend access, and multiple batch-query modes for repeat, has, properties, label, and drop steps.[^21]

### Graph-centric query path

Sequence:

1. Caller creates a traversal or calls `tx.query()`.
2. Query builder constructs a graph-centric query and asks the index-selection machinery for usable indexes.
3. `StandardJanusGraphTx.elementProcessor` executes a `JointIndexQuery` when present.
4. It streams results from the first subquery, intersects remaining subquery results as necessary, and converts returned IDs into vertices, edges, or properties.
5. If no index is available, behavior depends on `query.force-index`: throw when enabled, full-scan with warning when disabled.[^14][^21]

### Vertex-centric query path

Sequence:

1. Caller queries around a vertex.
2. `VertexCentricQueryBuilder` builds slice queries over the vertex’s adjacency list.
3. `StandardJanusGraphTx.edgeProcessor` checks transaction-local additions/deletions and multiplicity replacement effects.
4. Backend reads are executed through `graph.edgeQuery` or `graph.edgeMultiQuery`.
5. Returned entries are deserialized into relations and loaded into the vertex relation cache.[^14]

### Performance-oriented patterns

Source-grounded inference: performance patterns include transaction-local caches, vertex relation caches, multi-key backend queries, batching traversal strategies, query-cache usage for index subqueries, smart/full-scan limits, optional prefetching, and backend-level thread pools. These are performance inferences from source structure and configuration names, not measured benchmark claims.

## 13. Server, remote access, and API surfaces

Confirmed from project documentation: JanusGraph Server is Gremlin Server used as the server component, extended with JanusGraph convenience features, and supports remote execution of Gremlin traversals against one or more hosted JanusGraph instances.[^25]

Confirmed from source: `JanusGraphServer` reads a YAML configuration into `JanusGraphSettings`, constructs `GremlinServer`, optionally constructs a gRPC server, starts both, and configures JanusGraph-specific graph bindings after Gremlin Server startup.[^24]

Confirmed from source: `JanusGraphSettings` extends TinkerPop `Settings`, adds `grpcServer` settings, defaults gRPC to disabled, and defaults the gRPC port to `10182`.[^24]

Confirmed from source: `JanusGraphManager` implements TinkerPop `GraphManager`, tracks graph and traversal-source maps, opens graphs declared in server settings, treats `ConfigurationManagementGraph` specially, periodically binds configured graphs to the Gremlin executor, and handles commit/rollback across graph sources.[^24]

### Public API and remote boundaries

| Surface | Entry point | Protocol or call style | Main responsibility | Boundary risk |
| --- | --- | --- | --- | --- |
| Embedded API | `JanusGraphFactory.open`, `JanusGraph` methods | In-process Java | Direct graph transactions, schema, queries | Application shares JVM lifecycle and classpath with JanusGraph |
| Configured graph API | `ConfiguredGraphFactory` | In-process/server Java API | Dynamic graph configuration, open/create/drop | Requires configuration-management graph and server manager state |
| Gremlin Server | `JanusGraphServer` / TinkerPop Gremlin Server | WebSocket/HTTP per server configuration | Remote Gremlin traversals | Server YAML and graph property files must align |
| gRPC management | `JanusGraphManagerService` proto and service impls | gRPC | Graph context and version management, schema service implementations | Optional; disabled by default in settings |
| Driver module | `janusgraph-driver` | TinkerPop Gremlin driver dependency surface | Client-side driver utilities and serialization support | Compatibility with TinkerPop driver versions |

Confirmed from source and documentation: server configuration and operational mode are not a separate database engine; they wrap and expose graph instances.[^24][^25]

## 14. Configuration surfaces

Confirmed from source: `GraphDatabaseConfiguration` is the canonical source of graph configuration keys, defaults, mutability classes, namespaces, and many operational warnings. It defines graph, transaction, query, schema, cache, storage, lock, ID, index, log, attribute, and metrics namespaces.[^21]

### Configuration-surface table

| Configuration surface | Example source path | Scope | Mutability | Default behavior | Operational risk | Relevant modules |
| --- | --- | --- | --- | --- | --- | --- |
| Graph properties | `GraphDatabaseConfiguration` and `*.properties` consumed by `JanusGraphFactory` | Graph instance | Depends on option type | `storage.backend` is required; many options have explicit defaults | Misconfigured fixed/global options can block startup or corrupt expectations | Core, all backends |
| `graph.graphname` | `GraphDatabaseConfiguration.GRAPH_NAME` | Graph/server manager | Local | Optional; enables configured graph lookup when supplied | Duplicate/missing names affect server binding | Core, server |
| `schema.default` | `GraphDatabaseConfiguration.AUTO_TYPE` | Schema | Maskable | `default` | Implicit schema can create unexpected types | Core |
| `schema.constraints` | `GraphDatabaseConfiguration.SCHEMA_CONSTRAINTS` | Schema validation | Global offline | `false` | Enabling changes mutation validation | Core |
| Schema initialization | `schema.init.*` | Startup schema | Local | `strategy=none`, destructive options disabled | Drop/reindex/force-enable options can be expensive or destructive | Core |
| Storage backend | `storage.backend`, `storage.hostname`, `storage.directory` | Persistence | Mostly local/maskable | Backend required; hosts default to loopback for host-based settings | Wrong backend/host can fail startup or write to unintended data store | Core, storage adapters |
| Storage read/write wait | `storage.read-time`, `storage.write-time` | Backend operations | Maskable | Read 10s, write 100s | Too low causes transient failures; too high delays failover | Core, storage adapters |
| Cache | `cache.db-cache`, `cache.db-cache-size`, `cache.db-cache-time`, `cache.tx-cache-size` | Instance/transaction | Maskable | DB cache disabled, DB cache size 0.3, cache time 10s, tx cache 20000 | Stale reads or memory pressure | Core |
| Locking | `storage.lock.*` | Consistency | Mixed | Backend `consistentkey`, retries 3, wait 100ms, expiry 300s | Misconfiguration can cause contention or stale locks | Core, distributed storage |
| ID allocation | `ids.*` | Cluster/graph | Mixed, often global offline/fixed | Block size 10000, renew timeout 120s, CAV bits 4 | Changing fixed ID settings can corrupt data | Core, storage |
| Mixed index | `index.<name>.*` | Search backend | Global offline/maskable | Backend default `elasticsearch` when namespace exists | External index mismatch causes missed/slow queries | Core, index adapters |
| Metrics | `metrics.*` | Instance/transaction | Maskable | Basic metrics disabled | Measurement overhead or missing telemetry | Core, adapters |
| Server YAML | `JanusGraphSettings`, docs server YAML references | Server process | Runtime file | gRPC disabled by default, port 10182 when enabled | Wrong graph paths or manager class prevents graph binding | Server, distribution |
| Distribution environment | Server docs env table | Process startup | Environment | Defaults under `JANUSGRAPH_HOME` | Wrong classpath, PID, logs, JVM options | Distribution/server |

### Configuration precedence and overrides

Confirmed from source: local properties files are loaded through Apache Commons Configuration, and relative `storage.*.(directory|conf-file)` and `index.*.(directory|conf-file)` values are rewritten relative to the configuration file directory.[^10]

Confirmed from source: `GraphDatabaseConfiguration` differentiates configuration option types such as `LOCAL`, `MASKABLE`, `GLOBAL`, `GLOBAL_OFFLINE`, and `FIXED`; managed options can be stored in the backend and can conflict with local copies, with `graph.allow-stale-config` defaulting to true.[^21]

Not determined from inspected sources: this pass did not reconstruct the complete precedence algorithm for every configuration backend and every managed option type. The report identifies the major surfaces and key observed rules.

## 15. Operational model

### Embedded JVM mode

Confirmed from source: embedded operation uses `JanusGraphFactory.open` and returns a `JanusGraph` instance in the caller’s JVM.[^10] Operational dependencies are determined by `storage.backend`, index namespaces, cache settings, ID settings, and schema settings. The application owns graph lifecycle and must close the graph when done.

Operational risks:

- Using an in-memory backend has no durable persistence unless explicitly snapshotted by custom logic.[^19]
- Using distributed storage requires unique instance IDs and consistent global settings.[^21]
- Transaction commit can affect storage, composite indexes, mixed indexes, logs, and caches, so partial-failure behavior matters.[^13]

### JanusGraph Server mode

Confirmed from documentation and source: server mode starts JanusGraph Server through `bin/janusgraph-server.sh`, reads `conf/gremlin-server/gremlin-server.yaml` by default, and exposes hosted graph instances through Gremlin Server.[^25] Source confirms that the server is a TinkerPop `GremlinServer` wrapper with optional gRPC services.[^24]

Operational risks:

- Server YAML `graphs` entries must point to valid graph properties files containing `gremlin.graph=org.janusgraph.core.JanusGraphFactory` for the documented configuration path.[^25]
- `JanusGraphManager` must be active for `ConfiguredGraphFactory` and dynamic graph binding semantics.[^24]
- gRPC is disabled by default but has a default port when enabled.[^24]

### Packaged distribution mode

Confirmed from project documentation: distribution archives are created by the release profile and written to `janusgraph-dist/target/janusgraph-$VERSION.zip`. Without test skipping, distribution integration tests may start and stop Cassandra, Elasticsearch, and HBase and require `unzip` and `expect`.[^27]

### Containerized mode

Confirmed from project documentation: Docker image building has been moved into `janusgraph-dist/docker`, and the release-profile build command can also build Docker images.[^26]

Not determined from inspected sources: this pass did not inspect every Dockerfile and entrypoint file, so exact container environment variables and startup behavior are not fully enumerated beyond documentation and server-script environment variables.

### Distributed backend and search-index mode

Confirmed from source: distributed storage and index backends introduce external service dependencies. CQL needs CQL hosts/keyspace/replication behavior; HBase needs HBase/ZooKeeper/table/column-family behavior; Elasticsearch and Solr need external search clusters or local Lucene directories depending on backend choice.[^19][^20][^21]

Source-grounded inference: production deployments must treat JanusGraph as a coordinator over multiple systems, not as a single-process database. Correct operation depends on storage backend availability, index backend availability, consistent cluster-wide config, lock/ID behavior, and index lifecycle management.

## 16. Extension points and customization

### Extension-point table

| Extension point | Implemented through | Contract to satisfy | Tests/utilities support | Affects | Risks |
| --- | --- | --- | --- | --- | --- |
| Storage backend SPI | `KeyColumnValueStoreManager`, `KeyColumnValueStore`, `StoreFeatures` | Open stores, begin transactions, mutate many, scan, lock as supported, expose features | Backend test utilities and feature flags | Persistence, scan behavior, locking, ID allocation | Incorrect feature flags or mutation semantics can corrupt graph behavior |
| Index provider SPI | `IndexProvider`, `IndexInformation`, `IndexFeatures` | Register keys, mutate documents, query, raw query, totals, aggregation, clear, transaction handling | Mixed-index adapter tests | Mixed-index querying and indexing | Field mapping or consistency bugs cause missed query results |
| Custom schema maker | `DefaultSchemaMaker` class path through `schema.default` | Automatic schema behavior | Core schema tests | Implicit type creation and validation | Hidden modeling mistakes or unexpected implicit schema |
| Schema initialization strategy | `SchemaInitStrategy` class path through `schema.init.strategy` | Initialize schema and return graph or null | JSON strategy source and tests | Startup schema creation/reindexing | Destructive drop/reindex/force-enable behavior |
| Custom attribute serializer | `attribute.custom.*` configuration | Attribute class and serializer class registration | Serializer-related tests | Data serialization | Incompatible serializers can break reads |
| Log manager | `log.*.backend` custom `LogManager` | Constructor accepting configuration and log semantics | Log/recovery code | Transaction and management logs | Recovery and schema coordination risk |
| Parallel backend executor | `storage.parallel-backend-executor-service.class` | ExecutorService or constructor contract | Backend executor tests | Parallel storage operations | Threading/backpressure bugs |
| Graph manager/server | TinkerPop `GraphManager`, `JanusGraphManager` | Server graph binding and transaction coordination | Server integration tests | Server lifecycle and remote graph exposure | Wrong manager breaks `ConfiguredGraphFactory` |

Confirmed from source: storage and index shorthands can be replaced with fully qualified implementation classes; schema maker and schema-init strategy can be custom class paths; log backend and executor service configuration also accept custom implementations.[^17][^18][^21]

## 17. Design patterns and implementation patterns

| Pattern | Source-grounded example | Why it matters |
| --- | --- | --- |
| Factory pattern | `JanusGraphFactory`, `ConfiguredGraphFactory` | Centralizes graph opening, configuration loading, manager registration, and destructive drop behavior |
| Adapter/SPI pattern | Storage managers and index providers | Allows backend-specific behavior without moving graph semantics out of core |
| Transaction-scoped state | `StandardJanusGraphTx` caches, added/deleted relations, locks | Separates transaction-visible data from persisted backend data |
| Backend facade | `Backend` and `BackendTransaction` | Coordinates storage, indexes, logs, locks, caches, and retry behavior behind a core-facing API |
| Schema-management transactions | `JanusGraphManagement` / `ManagementSystem` | Separates schema mutation lifecycle from normal graph mutation lifecycle |
| Configuration as contract | `GraphDatabaseConfiguration` | Makes defaults, mutability, and operational warnings part of source-defined behavior |
| Strategy registration | TinkerPop traversal strategies in `StandardJanusGraph` | Integrates JanusGraph-specific query planning into Gremlin traversal execution |
| Index translation | `IndexSerializer` | Separates graph/index schema semantics from backend-specific index providers |
| Test harness reuse | backend test utilities and CI matrices | Normalizes behavior across adapters and feature combinations |
| Distribution as integration boundary | `janusgraph-dist` | Tests scripts, archives, and runtime packaging rather than just library code |

Confirmed from source: these patterns are visible in the inspected classes, module POMs, and build/test documentation.[^6][^7][^8][^10][^13][^14][^15][^16][^17][^21][^26][^27]

## 18. Validation, testing, and benchmarking

Confirmed from project documentation: JanusGraph uses JUnit, and the standard `mvn clean install` command compiles, packages, and runs the default suite. Memory and performance tests are disabled by default; test tags include `MEMORY_TESTS`, `PERFORMANCE_TESTS`, and untagged default tests.[^26]

Confirmed from project documentation: backend and index tests often require Docker. CQL tests use Testcontainers and can run Cassandra or Scylla profiles. Solr and Elasticsearch tests run against external containers and support version/profile customization. HBase tests also require Docker and expose Docker-specific properties.[^26]

Confirmed from source: CI runs a core workflow across Java 8 and Java 11, builds `janusgraph-all`, verifies with coverage, and tests module matrices including core, driver, server, test, inmemory, BerkeleyJE, and Lucene.[^26]

Confirmed from source: module POMs include backend-specific test dependency patterns. CQL includes Testcontainers Cassandra and Docker profile properties; Elasticsearch includes Testcontainers Elasticsearch/Cassandra; Solr includes Testcontainers Solr/Cassandra and Kerberos test dependencies; server includes failsafe integration-test/verify; BerkeleyJE packages test jars.[^6][^7][^8]

Source-grounded inference: the test strategy is designed around reusable behavioral tests plus backend-specific integration matrices. Storage and index adapters are validated through module-local tests and shared test utilities, while TinkerPop compliance is handled as a separate test path controlled by Maven properties.

## 19. Documentation and examples as architecture evidence

Confirmed from project documentation: `README.md` establishes the project’s graph-database identity and the existence of documentation, community, and project metadata.[^1] Confirmed from project documentation: `BUILDING.md`, `TESTING.md`, server docs, and distribution docs describe workflows that align with source-level Maven and server implementation.[^25][^26][^27]

Documentation-source alignment:

| Topic | Documentation evidence | Source corroboration | Alignment |
| --- | --- | --- | --- |
| Server wraps Gremlin Server | `docs/operations/server.md` says JanusGraph uses Gremlin Server as its server component | `JanusGraphServer` constructs TinkerPop `GremlinServer` | Strong |
| Embedded graph properties use JanusGraphFactory | Server docs require `gremlin.graph=org.janusgraph.core.JanusGraphFactory` in graph property files | `JanusGraphFactory` implements graph opening | Strong |
| Build/test commands | `BUILDING.md`, `TESTING.md` list Maven commands and tags | Root POM defines surefire, TinkerPop test execution, profiles | Strong |
| Distribution archives | `BUILDING.md`, `janusgraph-dist/README.md` describe release profile archive output | Root POM has release profile and dist module | Strong |
| Schema initialization | Config source defines JSON init strategy | Source implements JSON schema init | Strong source evidence; documentation details not fully inspected |

Not determined from inspected sources: the examples module was not read line-by-line. The report treats examples as a documented module family based on the root module list and POM structure, not as detailed evidence for API workflows.

## 20. Lineage, ecosystem relationship, and comparison analysis

### Titan relationship

Confirmed from source: the root POM defines `titan.compatible-versions=1.0.0,1.1.0-SNAPSHOT`.[^2] Confirmed from source: `GraphDatabaseConfiguration` contains a hidden `graph.titan-version` option for Titan version backward compatibility and an `ids.store-name` option whose description says the ID store can be overridden to facilitate migration from Titan because the prior KCV store was named `titan_ids`.[^21]

Source-grounded inference: the repository preserves migration/compatibility mechanisms from Titan at the storage/configuration level, especially around version metadata and ID-store naming. This does not imply source identity or code-copy claims beyond the inspected compatibility hooks.

### Apache TinkerPop relationship

Confirmed from source: TinkerPop is a central dependency family with root-managed version `3.7.3`, dependencies across core, driver, server, and tests, and TinkerPop test-suite execution in the root POM.[^2][^3][^6]

Confirmed from source: `JanusGraph` declares TinkerPop compliance opt-ins and specific opt-outs, `StandardJanusGraph` registers JanusGraph traversal strategies, and `JanusGraphServer` wraps TinkerPop Gremlin Server.[^9][^13][^24]

Source-grounded inference: JanusGraph is not merely compatible with TinkerPop; it uses TinkerPop as public API surface, traversal engine integration point, driver ecosystem, server runtime, and compliance test frame.

### Internal module-family comparison

| Comparison | Overlap | Divergence | Migration/operational implication |
| --- | --- | --- | --- |
| Core engine vs storage adapters | Both participate in persistence behavior | Core defines graph semantics and commit planning; adapters implement physical storage | Backend changes must preserve KCVS semantics and feature flags |
| Storage adapters vs mixed-index adapters | Both are backend integrations | Storage stores authoritative graph data; mixed indexes optimize/expose search | Index loss/mismatch can be rebuilt; storage loss is graph data loss |
| Embedded vs server mode | Both use `StandardJanusGraph` | Embedded is direct JVM API; server wraps graph instances in Gremlin Server/gRPC | Server configs must bind graphs and traversal sources correctly |
| CQL vs Scylla | Scylla extends CQL manager behavior | Scylla diverges in module dependency/driver profile | Most behavioral changes in CQL may affect Scylla unless overridden |
| Elasticsearch vs Solr vs Lucene | All implement `IndexProvider` | ES/Solr are remote search systems; Lucene is local filesystem | Query features and operational failure modes differ substantially |
| Source modules vs docs/dist modules | Same repository identity | Docs/dist encode workflow and packaging, not core graph behavior | Docs are useful but must be source-corroborated for behavior claims |

Repository-to-repository comparison status: only one repository was provided. No second repository has been invented or compared. If a second repository is later supplied, this section must be extended with overlap, divergence, lineage, naming/package similarities, dependency differences, API differences, implementation differences, operational differences, compatibility implications, and evidence for each claim.

## 21. Developer onboarding map

### Read first

1. `README.md`, root `pom.xml`, `BUILDING.md`, and `TESTING.md` for repository identity, build/test mechanics, and module list.
2. `janusgraph-core/src/main/java/org/janusgraph/core/JanusGraph.java` for public API shape.
3. `JanusGraphFactory` and `ConfiguredGraphFactory` for graph opening and manager behavior.
4. `StandardJanusGraph` for the runtime root and commit flow.
5. `StandardJanusGraphTx` for transaction-local behavior and query processors.
6. `Backend`, `BackendTransaction`, `KeyColumnValueStore*`, and `IndexProvider` for backend boundaries.
7. `GraphDatabaseConfiguration` for configuration defaults and operational consequences.
8. Representative backend or index module matching the area being changed.

### Tests to run by change type

| Change type | Minimum local test target | Additional target | Rationale |
| --- | --- | --- | --- |
| Core transaction/query/schema change | `mvn verify --projects janusgraph-core -Pcoverage` | TinkerPop tests with `-Dtest.skip.tp=false` where relevant | Core changes affect public graph behavior |
| Storage SPI change | Core tests plus affected backend module | Backend integration module with Docker if required | Feature flags and KCVS semantics are cross-backend |
| CQL change | `mvn clean install -pl janusgraph-cql` | Cassandra/Scylla profiles as relevant | CQL has version and partitioner matrix |
| Scylla change | `mvn clean install -pl janusgraph-scylla` | CQL tests where inherited behavior changes | Scylla inherits CQL behavior |
| HBase change | `mvn clean install -pl janusgraph-hbase` | HBase Docker configuration variants | HBase table/schema assumptions are operational |
| BerkeleyJE change | `mvn clean install -pl janusgraph-berkeleyje` | Core/storage tests | Local ordered store behavior affects KCVS abstraction |
| Mixed-index change | Affected module: `janusgraph-es`, `janusgraph-solr`, or `janusgraph-lucene` | Core query/index tests | Index providers affect graph-centric queries |
| Server change | `mvn verify --projects janusgraph-server` | Distribution package tests if scripts/configs change | Server behavior involves YAML, Gremlin Server, and optional gRPC |
| Distribution/script change | `janusgraph-dist` build with release profile | Package integration tests | Scripts and ZIP layout are integration boundaries |

### Common traps

- Treating a storage backend as if it owns graph semantics. It does not; core serializers, schema, ID handling, and commit logic own graph semantics.
- Assuming mixed indexes are authoritative storage. They are external query support and must be kept consistent with primary storage.
- Ignoring configuration mutability. Fixed/global/global-offline options can have cluster-wide effects and cannot be treated as simple local toggles.
- Modifying transaction commit behavior without considering WAL, storage commit, index commit, user log, and cache invalidation.
- Assuming server mode is a separate database engine. It is a Gremlin Server wrapper over graph instances.
- Forgetting TinkerPop compatibility and opt-outs. Public behavior may be constrained by TinkerPop tests and declared deviations.

### Debugging map

| Symptom | First files/classes to inspect | Likely boundary |
| --- | --- | --- |
| Graph will not open | `JanusGraphFactory`, `GraphDatabaseConfiguration`, backend manager | Configuration or backend initialization |
| Dynamic server graph not visible | `ConfiguredGraphFactory`, `JanusGraphManager`, server YAML | Graph manager binding |
| Schema creation fails | `JanusGraphManagement`, `ManagementSystem`, schema init strategy | Management transaction/schema status |
| Query unexpectedly scans | `StandardJanusGraphTx.elementProcessor`, `IndexSerializer`, `GraphDatabaseConfiguration.FORCE_INDEX_USAGE` | Index selection/query planning |
| Vertex adjacency read is slow | Vertex-centric query builders, `edgeProcessor`, backend multi-query support | Storage multi-query/cache behavior |
| Commit partially fails | `StandardJanusGraph.commit`, `BackendTransaction`, transaction log settings | Storage/index/log coordination |
| Mixed-index query misses results | `IndexSerializer`, affected `IndexProvider`, index lifecycle/status | Index update/query mapping |
| Backend-specific integration fails | Affected `*StoreManager`, module POM test properties | External service configuration |

## 22. Risks, ambiguities, and unresolved questions

A Cadastre profile, package, deployment, query translation row, or validation fixture must not depend on any `Not determined` JanusGraph area until that area has an owner decision and passing validation evidence.

| Unknown or ambiguity | Why it matters | Source to inspect next | Affects |
| --- | --- | --- | --- |
| Full Bigtable adapter behavior was not resolved beyond module POM dependency | Bigtable operational behavior may be inherited or configured through HBase-compatible classes not inspected here | Full `janusgraph-bigtable` tree and docs | Architecture, operations |
| Full gRPC schema service contract was not enumerated | gRPC behavior may expose more management/schema methods than the graph manager proto inspected here | All `janusgraph-grpc/src/main/proto/**` and server service implementations | API, operations |
| Complete configuration precedence algorithm was not reconstructed for every option type | Operators need exact local/global/backend-hosted override semantics | `BasicConfiguration`, `KCVSConfiguration`, `GraphDatabaseConfiguration` validation methods | Operations |
| Every query optimizer strategy was not traced | Traversal performance and query planning can depend on strategy-specific behavior | `graphdb/tinkerpop/optimize/**`, query index strategy classes | Query implementation |
| Full relation/ID binary encoding was not exhaustively mapped | Storage migration and low-level debugging may require exact binary formats | `IDManager`, `IDHandler`, `RelationIdentifier`, serializer tests | Storage, migration |
| Docker entrypoint behavior was not fully inspected | Containerized operations depend on entrypoint/env behavior | `janusgraph-dist/docker/**` | Operations |
| Examples were not used as primary evidence | Examples may demonstrate supported usage patterns and edge cases | `janusgraph-examples/**` | Onboarding, docs |
| Full release process was not reconstructed | Release engineering may require signing/publishing/versioning details | `RELEASING.md`, release workflows, plugin configs | Build/release |

## Cadastre transfer ledger

This section is non-normative. Runtime behavior must be adopted only through owner NLSpecs.

| Finding | Owning spec for any transfer | Transfer status | Required validation before production |
| --- | --- | --- | --- |
| JanusGraph as MVP default provider candidate | `090` | evidence only until profile row is active | default-provider, package, schema, index, query, apply, and rebuild fixtures |
| Gremlin/TinkerPop query surface | `090` | mapping input | Gremlin translation parity fixtures |
| Storage and mixed-index adapter families | `090`, `100` | provider-profile and package-gate input | storage/index capability and freshness fixtures |
| Schema defaults and schema initialization behavior | `090` | schema strategy input | implicit-schema rejection and fingerprint fixtures |
| Transaction and partial-commit behavior | `090` | graph apply evidence input | partial-apply and resume-safety fixtures |
| Packaging, distribution, and dependency evidence | `100` | package-gate input | package release, SBOM, provenance, compatibility, and rollback fixtures |

## 23. Evidence ledger

### Confirmed findings from source

| Finding | File path | Relevant class/function/config/doc section | Confidence |
| --- | --- | --- | --- |
| Repository is a Maven multi-module project at version `1.2.0-SNAPSHOT` | `pom.xml` | root coordinates and modules | High |
| TinkerPop version is root-managed as `3.7.3` | `pom.xml` | properties/dependency management | High |
| Core public API is TinkerPop-facing and declares compliance opt-ins/opt-outs | `janusgraph-core/.../JanusGraph.java` | interface annotations and methods | High |
| Embedded opening is through `JanusGraphFactory` | `janusgraph-core/.../JanusGraphFactory.java` | `open`, `getLocalConfiguration`, `drop` | High |
| Dynamic configured graphs use `ConfiguredGraphFactory` and `JanusGraphManager` | `ConfiguredGraphFactory.java`, `JanusGraphManager.java` | `create`, `open`, manager graph cache | High |
| Core runtime is `StandardJanusGraph` | `StandardJanusGraph.java` | constructor, transaction, commit methods | High |
| Transaction state and query processors live in `StandardJanusGraphTx` | `StandardJanusGraphTx.java` | fields, processors, commit/rollback | High |
| Backend orchestration lives in `Backend` | `Backend.java` | constructor, initialize, beginTransaction | High |
| Backend transactions wrap storage and index transactions | `BackendTransaction.java` | storage/index commit/query/mutate methods | High |
| Storage SPI uses key-column-value abstractions | `KeyColumnValueStoreManager.java`, `KeyColumnValueStore.java` | SPI methods | High |
| Mixed-index SPI uses `IndexProvider` | `IndexProvider.java` | register, mutate, query, raw query, transaction methods | High |
| Built-in storage shorthands include BerkeleyJE, CQL, HBase, in-memory, Scylla | `StandardStoreManager.java` | enum entries | High |
| Built-in index shorthands include Lucene, Elasticsearch/ES, Solr | `StandardIndexProvider.java` | enum entries | High |
| CQL, Scylla, HBase, BerkeleyJE, and in-memory adapters implement storage boundaries | representative manager classes and POMs | `CQLStoreManager`, `ScyllaStoreManager`, `HBaseStoreManager`, `BerkeleyJEStoreManager`, `InMemoryStoreManager` | Medium-high |
| Elasticsearch, Solr, and Lucene implement `IndexProvider` | representative index classes | `ElasticSearchIndex`, `SolrIndex`, `LuceneIndex` | High |
| JSON schema initialization exists | `GraphDatabaseConfiguration`, `SchemaInitializationManager`, `JsonSchemaInitStrategy` | `schema.init.*`, strategy resolution, JSON implementation | High |
| JanusGraph Server wraps TinkerPop Gremlin Server and optional gRPC | `JanusGraphServer.java`, `JanusGraphSettings.java` | `start`, `createGrpcServer`, defaults | High |
| gRPC graph-manager service has context and version RPCs | `graph_manager.proto` | service definition | High |
| Build/test workflow includes Java 8 default, Java 11 profile, TinkerPop test switch, CI module matrix | `pom.xml`, `BUILDING.md`, `TESTING.md`, `.github/workflows/ci-core.yml` | build/profile/test docs and workflow | High |

### Confirmed findings from project documentation

| Finding | Documentation path | Confidence | Source corroborates it |
| --- | --- | --- | --- |
| JanusGraph is documented as a scalable transactional graph database for large graphs | `README.md` | High for documentation claim | Yes, root/module source shows graph database implementation |
| Required build tools are Java 8 and Maven 3 | `BUILDING.md` | High | Yes, root POM and CI Java setup corroborate |
| Distribution archive is produced under `janusgraph-dist/target/janusgraph-$VERSION.zip` | `BUILDING.md`, `janusgraph-dist/README.md` | High | Yes, release profile and dist module corroborate |
| JanusGraph Server uses Gremlin Server as the server component | `docs/operations/server.md` | High | Yes, `JanusGraphServer` constructs `GremlinServer` |
| Server script defaults and environment variables are documented | `docs/operations/server.md` | Medium-high | Script files not fully inspected in this pass |
| Backend/index integration tests require Docker in common cases | `TESTING.md` | High | Yes, module POMs include Testcontainers and Docker properties |

### Source-grounded inferences

| Inference | Evidence | Reasoning | Confidence | What would disconfirm it |
| --- | --- | --- | --- | --- |
| Backends are intended to be replaceable without moving graph semantics out of core | SPI interfaces, registered manager enum, backend modules | Core owns serializers/transaction/query; adapters implement storage | High | Backend classes defining graph-schema semantics outside SPI |
| Scylla behavior mostly inherits CQL behavior | `ScyllaStoreManager` extends `CQLStoreManager`; Scylla POM swaps driver dependencies | Minimal class override suggests dependency/profile divergence | Medium-high | Additional Scylla-specific code paths not inspected here |
| Composite indexes are more tightly coupled to primary storage than mixed indexes | `StandardJanusGraph` and `BackendTransaction` mutation paths | Composite uses `graphindex` store; mixed uses `IndexTransaction` | High | Alternative composite index provider path not inspected |
| Production deployment complexity comes from multi-system coordination | source config and adapters for storage/index/log/ID/locks | Multiple external systems and consistency settings must align | High | Single-process-only production mode, not supported by source evidence |
| Query performance is controlled by strategies, index selection, caching, and backend multi-query features | traversal strategy registration, config keys, transaction query processors | Source includes explicit knobs and multi-query pathways | High | Benchmark evidence showing these are irrelevant |

### Not determined

| Question | Attempted evidence | Why unresolved |
| --- | --- | --- |
| Full Bigtable implementation details | `janusgraph-bigtable/pom.xml`, code search | No adapter class was resolved in this pass |
| Complete gRPC schema service API | gRPC POM and graph manager proto | Only graph manager proto was fetched; all protos not enumerated |
| Exact container entrypoint behavior | Build docs and server docs | Docker directory files were not fully inspected |
| Full ID binary encoding | Config and relation serializer | `IDManager`/`IDHandler` were not fully traced |
| Full query optimizer strategy behavior | Strategy registration and transaction processors | Individual strategy classes were not fully inspected |
| Complete examples behavior | Root module list and POM context | Examples module source was not inspected line by line |

### Sources

[^1]: JanusGraph/janusgraph, `README.md`, lines 1-44, fetched from GitHub connector on 2026-05-18. The report’s detailed source inspection is otherwise anchored to commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^2]: JanusGraph/janusgraph, `pom.xml`, lines 1-260, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^3]: JanusGraph/janusgraph, `pom.xml`, lines 260-620, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^4]: JanusGraph/janusgraph, `pom.xml`, lines 620-1100, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^5]: JanusGraph/janusgraph, `pom.xml`, lines 1100-1650, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^6]: JanusGraph/janusgraph, `janusgraph-core/pom.xml`, lines 1-260; `janusgraph-server/pom.xml`, lines 1-260; `janusgraph-driver/pom.xml`, lines 1-220; `janusgraph-grpc/pom.xml`, lines 1-260; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^7]: JanusGraph/janusgraph, `janusgraph-cql/pom.xml`, lines 1-520; `janusgraph-scylla/pom.xml`, lines 1-260; `janusgraph-hbase/pom.xml`, lines 1-260; `janusgraph-bigtable/pom.xml`, lines 1-260; `janusgraph-berkeleyje/pom.xml`, lines 1-220; `janusgraph-inmemory/pom.xml`, lines 1-180; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^8]: JanusGraph/janusgraph, `janusgraph-es/pom.xml`, lines 1-260; `janusgraph-solr/pom.xml`, lines 1-260; `janusgraph-lucene/pom.xml`, lines 1-220; `janusgraph-mixed-index-utils/pom.xml`, lines 1-180; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^9]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/JanusGraph.java`, lines 1-220, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^10]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/JanusGraphFactory.java`, lines 1-620, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^11]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/ConfiguredGraphFactory.java`, lines 1-300, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^12]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/schema/JanusGraphManagement.java`, lines 1-260, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^13]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/graphdb/database/StandardJanusGraph.java`, lines 140-1160, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^14]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/graphdb/transaction/StandardJanusGraphTx.java`, lines 1-620 and 1280-1800, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^15]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/diskstorage/Backend.java`, lines 1-760, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^16]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/diskstorage/BackendTransaction.java`, lines 1-760, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^17]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/diskstorage/keycolumnvalue/KeyColumnValueStoreManager.java`, lines 1-260; `janusgraph-core/src/main/java/org/janusgraph/diskstorage/keycolumnvalue/KeyColumnValueStore.java`, lines 1-260; `janusgraph-core/src/main/java/org/janusgraph/diskstorage/indexing/IndexProvider.java`, lines 1-320; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^18]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/diskstorage/StandardStoreManager.java`, lines 1-220; `janusgraph-core/src/main/java/org/janusgraph/diskstorage/StandardIndexProvider.java`, lines 1-180; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^19]: JanusGraph/janusgraph, `janusgraph-cql/src/main/java/org/janusgraph/diskstorage/cql/CQLStoreManager.java`, lines 1-260; `janusgraph-scylla/src/main/java/org/janusgraph/diskstorage/cql/ScyllaStoreManager.java`, lines 1-220; `janusgraph-hbase/src/main/java/org/janusgraph/diskstorage/hbase/HBaseStoreManager.java`, lines 1-220; `janusgraph-berkeleyje/src/main/java/org/janusgraph/diskstorage/berkeleyje/BerkeleyJEStoreManager.java`, lines 1-240; `janusgraph-inmemory/src/main/java/org/janusgraph/diskstorage/inmemory/InMemoryStoreManager.java`, lines 1-220; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^20]: JanusGraph/janusgraph, `janusgraph-es/src/main/java/org/janusgraph/diskstorage/es/ElasticSearchIndex.java`, lines 1-260; `janusgraph-solr/src/main/java/org/janusgraph/diskstorage/solr/SolrIndex.java`, lines 1-240; `janusgraph-lucene/src/main/java/org/janusgraph/diskstorage/lucene/LuceneIndex.java`, lines 1-240; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^21]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/graphdb/configuration/GraphDatabaseConfiguration.java`, lines 1-1320, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^22]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/Multiplicity.java`, lines 1-220; `janusgraph-core/src/main/java/org/janusgraph/core/Cardinality.java`, lines 1-160; `janusgraph-core/src/main/java/org/janusgraph/graphdb/database/EdgeSerializer.java`, lines 1-320; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^23]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/graphdb/database/IndexSerializer.java`, lines 1-260, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^24]: JanusGraph/janusgraph, `janusgraph-server/src/main/java/org/janusgraph/graphdb/server/JanusGraphServer.java`, lines 1-280; `janusgraph-server/src/main/java/org/janusgraph/graphdb/server/JanusGraphSettings.java`, lines 1-280; `janusgraph-core/src/main/java/org/janusgraph/graphdb/management/JanusGraphManager.java`, lines 1-300; `janusgraph-grpc/src/main/proto/janusgraph/v1/graph_manager.proto`, lines 1-240; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^25]: JanusGraph/janusgraph, `docs/operations/server.md`, lines 1-260, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^26]: JanusGraph/janusgraph, `BUILDING.md`, lines 1-240; `TESTING.md`, lines 1-260; `.github/workflows/ci-core.yml`, lines 1-220; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^27]: JanusGraph/janusgraph, `janusgraph-dist/README.md`, lines 1-260, commit `346f5a46a9b01c1979272f14de95435d4777c160`.
[^28]: JanusGraph/janusgraph, `janusgraph-core/src/main/java/org/janusgraph/core/schema/SchemaInitializationManager.java`, lines 1-260; `janusgraph-core/src/main/java/org/janusgraph/core/schema/JsonSchemaInitStrategy.java`, lines 1-320; all at commit `346f5a46a9b01c1979272f14de95435d4777c160`.
