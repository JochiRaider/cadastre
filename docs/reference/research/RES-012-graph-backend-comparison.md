---
doc_id: RES-012
project: Cadastre
title: Graph Backend Comparison for Cadastre
status: research-report
inspection_date: 2026-05-16
---

## Executive verdict

Cadastre can evaluate Neo4j, Memgraph, JanusGraph, and ArangoDB only as replaceable graph-serving read models and graph-delta apply targets. The graph backend must not become the authoritative raw evidence store, silver observation store, gold fact store, identity resolver, source-completeness authority, source-authority arbiter, temporal ledger, or graph-rebuild authority. The governing PRD states that the lakehouse is the system of record, while graph output is a replaceable projection, and that graph rebuild, graph lag, replay retention, and destructive maintenance are Cadastre correctness concerns governed by explicit Cadastre contracts rather than backend defaults.[^1]

This report does not select a final production backend. It ranks fit only by bounded use-case axes, and every ranking requires clone, build, runtime, workload, driver, security, and failure-mode validation before a production decision.

| Axis | Strongest apparent fit | Assumptions | Required validation before decision |
| --- | --- | --- | --- |
| Fastest proof-of-concept | Neo4j, then Memgraph | Cadastre starts with a labeled-property-graph read model and Cypher-like authoring expectations. | Build a minimal `GraphDeltaSet` apply harness, query harness, schema preflight, and stale-read response contract against both. |
| Strongest Cypher / labeled-property-graph fit | Neo4j | Cadastre wants mature Cypher, constraints, indexes, cluster routing, driver bookmarks, and explainable plans. | Validate Cypher 25 behavior, index plans, write retries, cluster failover, constraint errors, and deterministic path ordering. |
| Strongest real-time graph analytics fit | Memgraph | Cadastre wants Cypher-compatible serving with streaming/import-oriented projection inputs and optional algorithm modules. | Validate storage mode, HA mode, memory pressure, Cypher compatibility, schema preflight, and on-disk limitations under Cadastre apply semantics. |
| Strongest scale-out fit | JanusGraph | Cadastre expects very large graphs, distributed storage, and Gremlin/TinkerPop workloads. | Validate Gremlin query contract translation, backend consistency, uniqueness locking, mixed-index freshness, super-node behavior, and operational complexity. |
| Strongest graph/document co-modeling fit | ArangoDB | Cadastre wants document-plus-graph serving metadata in one engine without making it authoritative. | Validate named-graph integrity, raw collection bypass controls, AQL traversal ordering, cluster transaction limits, and graph/document boundary enforcement. |
| Lowest operational complexity for MVP | Neo4j or Memgraph single-primary | MVP prioritizes current-state and recent-history graph serving over full historical graph serving. | Validate backup/restore, health, metrics, access control, schema rebuild, and single-writer apply behavior. |
| Strongest deterministic graph-apply verifiability | Neo4j or ArangoDB single-shard / OneShard-style deployment | Apply uses one explicit transaction boundary, deterministic IDs, unique selectors, and preflighted schema. | Validate complete rollback on failure, constraint-error mapping, read-after-write evidence, and persisted backend schema fingerprints. |
| Lowest lock-in risk | No backend by itself | Lock-in risk is minimized by Cadastre-owned query, projection, delta, schema, apply, rebuild, and lag contracts, not by backend choice. | Validate that every backend adapter implements the same `QueryGraph`, `ApplyGraphDelta`, `GraphReadModelSchemaProfile`, and `GraphApplyResult` contracts. |
| Highest PRD-contract burden | JanusGraph | Gremlin/TinkerPop differs most from Cypher-like expectations, and distributed backend consistency is backend-dependent. | Create a complete Gremlin translation table, index consistency policy, uniqueness policy, and super-node mitigation policy. |

The high-level transfer stance is:

| Backend | Bounded Cadastre stance |
| --- | --- |
| Neo4j | `adapt` as the mature Cypher/labeled-property-graph reference. Reject internal IDs, APOC/GDS/APOC-like convenience, and Neo4j-specific storage semantics as Cadastre truth. |
| Memgraph | `study` to `adapt` as a Cypher-compatible high-performance graph-serving and real-time analytics candidate. Reject assumptions of full Neo4j compatibility and reject in-memory state as historical authority. |
| JanusGraph | `study` as the scale-out and distributed graph reference. Adapt schema/index/vertex-centric/super-node lessons, but reject uncontracted Gremlin semantics and backend eventual consistency as Cadastre defaults. |
| ArangoDB | `adapt` as a multi-model graph/document serving candidate. Use named graph integrity and AQL traversal patterns cautiously; reject multi-model convenience as a reason to store authoritative raw/silver/gold facts in the graph backend. |

## Source limits and inspection method

Inspection date: 2026-05-16.

Primary sources used were official repositories, official documentation, release notes, manuals, and selected official examples. No repository was cloned locally. No backend was built. No database was started. No driver was exercised. No benchmark, failover test, backup/restore test, import job, schema migration, or graph apply harness was run. Therefore, all runtime, performance, compatibility, and failure-mode claims are either documentation facts, source-grounded inferences, Cadastre applicability inferences, or open validation requirements.

| Evidence class | Meaning in this report |
| --- | --- |
| `confirmed source fact` | Directly supported by inspected source code, repository structure, official docs, release notes, examples, tests, or official manuals. |
| `documentation fact` | Directly supported by official documentation but not verified by local execution. |
| `source-grounded inference` | Inferred from multiple confirmed facts in one backend. |
| `Cadastre applicability inference` | Design implication derived by comparing backend behavior to the Cadastre PRD. |
| `speculative transfer` | Promising idea requiring local clone, build, benchmark, runtime test, or domain authority before PRD amendment. |
| `open question` | Required follow-up that cannot be resolved from inspected sources. |

Report method:

1. Establish the Cadastre baseline from the PRD and NLSpec standard.
2. Perform a source discovery pass for each backend.
3. Analyze each backend independently before cross-backend comparison.
4. Map backend behavior to Cadastre graph-serving, schema, apply, rebuild, lag, and non-transfer contracts.
5. Produce backend-neutral PRD improvement recommendations with defaults, bounds, errors, and binary acceptance criteria.

The NLSpec standard used for report quality requires behavioral completeness, unambiguous interfaces, explicit defaults and boundaries, mapping tables for translation, and binary acceptance criteria.[^2]

## Cadastre baseline and graph-serving boundary

Cadastre’s graph backend is a derived serving substrate. It may represent projected graph nodes, typed graph edges, edge properties, evidence references, assertion states, confidence, expiry scopes, no-op outcomes, backend schema fingerprints, apply results, rebuild manifests, index checks, and derived-view lag state. It must not determine what facts exist.

The governing boundary is:

```text
Authoritative lakehouse records
  -> GraphProjectionProfile
  -> persisted GraphDeltaSet
  -> GraphApplyProfile
  -> GraphApplyResult
  -> Graph read model
```

The backend may optimize storage, query planning, traversal, indexing, and apply execution. It must not create unpersisted authoritative graph mutations, infer identity, decide source completeness, decide fact retraction, define bitemporal semantics, or repair drift by itself. The PRD requires graph deltas to be generated through active `GraphProjectionProfile` mapping rows or declared synthetic structural rules, preflights graph read-model constraints and indexes through `GraphReadModelSchemaProfile`, and applies graph deltas only through `ApplyGraphDelta(GraphDeltaSet, GraphApplyProfile) -> GraphApplyResult`.[^3]

The graph query contract requires neighbor expansion, bounded path search, node-type and edge-type filters, assertion-state filters, confidence filters, valid-time filters, known-time filters, optional evidence references, current-state defaults, depth bounds from 1 through 6, default depth 3, page-size default 100, page-size maximum 1000, timeout default 30 seconds, timeout maximum 300 seconds, and deterministic path ordering by path length, minimum edge confidence, maximum edge `last_seen`, and lexical node ID sequence.[^4]

The rebuild and serving contracts are also already explicit. A `GraphRebuildManifest` must record rebuild inputs from authoritative lakehouse state and active graph profiles; `GraphIndexConsistencyCheck` must validate schema, indexes, constraints, nodes, edges, and evidence indexes after rebuild or apply; and every graph-serving response using graph read-model state must include or reference a `DerivedViewState` with the served dataset or snapshot version and graph apply or rebuild reference.[^5]

## Preliminary source discovery results

### Discovery summary

| Backend | Repository and current-source areas inspected | High-value source areas inspected | Not inspected | Freshness and reliability risk |
| --- | --- | --- | --- | --- |
| Neo4j | `neo4j/neo4j` repository README; current operations manual; current Cypher manual; constraints; import; clustering; driver manual; monitoring/security docs; release notes. | Labeled property graph purpose, Cypher 25 behavior, constraints, import modes, clustering, driver sessions/bookmarks, monitoring/RBAC. | Full source tree, tests, kernel storage implementation, APOC/GDS internals, local cluster, import runtime, benchmark workloads. | High reliability for official docs; medium risk for version-specific behavior because Neo4j 2025/2026 release line is active. |
| Memgraph | `memgraph/memgraph` repository README; release notes; storage modes; indexes/constraints; storage access; replication; import; monitoring/security docs. | Storage modes, in-memory/on-disk implications, Cypher-compatible claims, index/constraint support, replication modes, HA health checks, monitoring metrics. | Full C/C++ source, tests, query planner internals, MAGE procedures, custom modules, local failover, import stress, on-disk HA validation. | High reliability for official docs; medium risk for compatibility details and Enterprise feature boundaries. |
| JanusGraph | `JanusGraph/janusgraph` repository README; release/changelog; docs for storage/index backends, data model, transactions, eventual consistency, schema/indexing, Gremlin basics, server/configuration. | TinkerPop/Gremlin property graph model, storage backend abstraction, index backend abstraction, transaction limitations, locking/consistency, vertex-centric indexes, super-node concerns. | Local JanusGraph Server, Cassandra/HBase/Scylla/Bigtable deployments, Gremlin translation harness, index rebuild, current bulk loader details, backend-specific security/monitoring in depth. | Medium-high reliability for official docs; higher risk because behavior depends on selected storage and index backends. |
| ArangoDB | `arangodb/arangodb` repository README; docs for graphs, traversals, shortest path, transactions, indexes, ArangoSearch, import/backup, sharding, SmartGraphs, server options, security search results. | Multi-model model, named graph integrity, `_from`/`_to`, traversal semantics, shortest path tie behavior, transaction limitations, edge indexes, inverted-index consistency, backup/import. | Full source tree, local cluster, SmartGraph/EnterpriseGraph runtime, OneShard transaction validation, AQL query-plan harness, vector/geo/deep index validation. | High reliability for official docs; medium risk for Enterprise feature boundaries and cluster/transaction edge cases. |

### Follow-up inspection plan

| Backend | Follow-up inspection required before PRD amendment |
| --- | --- |
| Neo4j | Clone `neo4j/neo4j`; inspect kernel/schema/index/transaction tests; run driver transaction and failover tests; validate Cypher 25 query set; validate `neo4j-admin database import` full and incremental rebuild; capture explain plans and schema fingerprints. |
| Memgraph | Clone `memgraph/memgraph`; inspect storage mode tests, Cypher parser/planner tests, constraint/index tests, replication tests; run the same Cadastre graph apply harness in each supported storage mode; validate MAGE/procedure non-use boundaries. |
| JanusGraph | Clone JanusGraph; run with BerkeleyDB, Cassandra/CQL or Scylla, and one mixed-index backend; validate Gremlin translation, composite and mixed indexes, uniqueness/locking, ghost vertices, stale index behavior, vertex-centric indexes, batch load, and super-node mitigation. |
| ArangoDB | Clone ArangoDB or use official distribution; validate named graph integrity, raw collection bypass prevention, AQL traversal order contract, shortest path tie behavior, stream transaction boundaries, cluster transaction failure, import/dump/restore rebuild, and index/search freshness. |

## Neo4j independent analysis

### 1. Source inventory

| Field | Content |
| --- | --- |
| Repository URL | `https://github.com/neo4j/neo4j` |
| Documentation URLs | Neo4j Operations Manual, Cypher Manual, Import docs, Driver Manual, Clustering docs, Monitoring docs, Authentication/Authorization docs. |
| Release, branch, tag, or commit inspected | Public repository and current docs inspected through web on 2026-05-16. Current release page identified Neo4j `2026.04.0` in the 2025/2026 release line. Exact repository commit was not pinned.[^8] |
| Evidence type | `confirmed source fact`, `documentation fact`, `source-grounded inference`, `Cadastre applicability inference`. |
| Scope inspected | Repository README, purpose statement, Cypher docs, Cypher 25 notes, constraints, import, clustering, driver sessions, CDC internal-ID warning, monitoring/security docs. |
| Scope not inspected | Full kernel source, internal tests, APOC/GDS source, storage internals, local import, local cluster, benchmark behavior. |
| Reliability and freshness risk | High for official docs; medium for exact runtime behavior and release-specific edge cases not executed. |

### 2. Project purpose and non-purpose

Neo4j is built as a graph database using a labeled property graph model and Cypher. Its README describes a flexible network structure of nodes and relationships, high-performance query execution, and ACID transactions.[^9]

For Cadastre, Neo4j is useful as the mature labeled-property-graph and Cypher reference backend. It does not solve Cadastre raw evidence preservation, source completeness, source authority, identity resolution, temporal fact correction, lakehouse replay, graph rebuild authority, or backend-neutral query semantics.

### 3. Repository and system architecture

| Area | Neo4j role | Cadastre relevance |
| --- | --- | --- |
| Core database/kernel | Native graph storage, transactions, schema, indexes, records, query execution. | Backend implementation only; not Cadastre truth. |
| Cypher engine | Pattern matching, traversal, write statements, planning. | Strongest reference for Cadastre Cypher-like query syntax, but Cadastre must define its own query contract. |
| Schema and indexes | Constraints and indexes over labels, relationship types, and properties. | Strong candidate for `GraphReadModelSchemaProfile` rows. |
| Cluster architecture | Primaries and secondaries, failover, replication, routing. | Must be reflected in `GraphApplyProfile` writer routing and `GraphApplyResult`. |
| Import tooling | `neo4j-admin database import`, full and incremental import, CSV/Parquet, dry-run, schema files. | Useful for graph rebuild strategy, but import success must not prove rebuild correctness. |
| Drivers | Sessions, transaction functions, bookmarks, routing, causal consistency. | Required for deterministic apply, retries, and read-after-write checks. |
| Monitoring/security | Metrics, logs, query logs, security logs, RBAC, LDAP/OIDC/SSO. | Operational health and access-control surface; must not replace graph-property redaction policy. |

### 4. Graph data model

Neo4j’s model is a labeled property graph. Nodes have labels and properties. Relationships have a type, direction, properties, and identity. Cypher patterns express labels and relationship types. Neo4j constraints include node property uniqueness, relationship property uniqueness, node and relationship property existence, property type constraints, and key constraints, with edition-specific feature availability for some constraints.[^10]

Cadastre mapping:

| Neo4j concept | Cadastre-safe mapping | Hazard |
| --- | --- | --- |
| Node label | `GraphNodeDelta.node_type` plus backend label emitted by `GraphTaxonomyTranslationPolicy`. | Backend labels must not become canonical entity types. |
| Relationship type | `GraphEdgeDelta.edge_type` mapped by `GraphEdgeSemantics`. | Relationship type direction must not override Cadastre edge direction rules. |
| Node property | Graph read-model property from `GraphPropertyMappingRow`. | Properties must not expose raw payload or secrets. |
| Relationship property | Edge property from `GraphEdgeSemantics` and projection row. | Edge property must not create unsupported fact semantics. |
| Internal node/relationship ID or `elementId` | Forbidden as Cadastre ID. | Neo4j CDC docs warn internal element IDs are not safe outside a single transaction and require logical keys via key constraints.[^11] |
| Key constraints | Backend schema enforcement for deterministic selectors. | Constraint success proves backend state shape, not identity truth. |

### 5. Query language and traversal model

Cypher is Neo4j’s query language. Current Neo4j docs identify Cypher 25 as the current language line, state that new features as of 2025.06 are added exclusively to Cypher 25, and state that new databases as of 2026.02 must explicitly use Cypher 25.[^12]

Cadastre must therefore not specify “Cypher” as a monolith. It must declare a backend query dialect version and a translation profile. Production `QueryGraph` must not rely on Neo4j-specific functions, APOC procedures, GDS algorithms, implicit path tie-breaking, internal IDs, or storage-level traversal semantics unless a Cadastre query contract maps them explicitly.

Required Neo4j query contract checks:

| Concern | Required Cadastre behavior |
| --- | --- |
| Neighbor expansion | Use deterministic ordering after backend results. |
| Bounded path search | Apply Cadastre depth bounds and path ordering independent of backend return order. |
| Reverse path search | Translate by relationship direction in `GraphEdgeSemantics`, not by label naming. |
| Pagination | Use stable sort keys and cursor tokens owned by Cadastre. |
| Timeout | Enforce request timeout in Cadastre and backend transaction timeout when available. |
| Evidence inclusion | Fetch by Cadastre evidence properties, not by graph storage IDs. |
| Authorization filtering | Apply graph-property redaction and authorization rules before returning data. |
| Plan introspection | Persist explain/profile summary only as diagnostics. |

### 6. Transaction, consistency, and apply model

Neo4j supports ACID transactions and driver sessions with causal consistency. Driver docs describe sessions as query channels and state that causal consistency is enforced for sessions, while sessions are lightweight but not thread-safe.[^13]

Cluster docs describe fault tolerance through primaries and secondaries. For a single database, a cluster with three primaries can maintain write availability through one primary failure; five primaries tolerate two failures. Secondaries use asynchronous replication and are used for read scaling.[^14]

Cadastre apply rule for Neo4j:

```text
A Neo4j GraphApplyProfile must target a writer route, preflight schema, apply ordered_delta_refs inside explicit transaction batches, persist bookmarks or causal-consistency evidence, and record constraint errors, transaction IDs when exposed, bookmark state, writer member identity, database name, schema fingerprint, input GraphDeltaSet checksum, and output read-after-write verification in GraphApplyResult.
```

Partial-apply failure must be forbidden unless a batch boundary is declared, the failed batch has no committed writes, or committed batch IDs are recorded and idempotent reapply can prove all committed deltas match the requested graph state.

### 7. Indexes, constraints, and schema preflight

Neo4j supports property uniqueness, node/relationship key constraints, property existence constraints, and type constraints. Relationship property uniqueness and relationship key constraints are especially relevant for Cadastre edge selectors.[^10]

Required `GraphReadModelSchemaProfile` rows for Neo4j:

| Schema object | Neo4j expression target | Required preflight |
| --- | --- | --- |
| Unique graph node selector | Node key or uniqueness constraint over `node_id`. | `SHOW CONSTRAINTS` must match name, label, properties, and type. |
| Unique graph edge selector | Relationship key or uniqueness constraint over `edge_id`. | Must exist before edge upsert. |
| Evidence lookup | Index over evidence reference properties. | Must exist and be online before evidence drillback queries. |
| Temporal lookup | Range index over valid/known time properties. | Must be used or query plan must be accepted by plan policy. |
| Assertion state lookup | Index or composite index over assertion state and type. | Required for filters. |
| Backend schema fingerprint | Canonicalized `SHOW INDEXES` and `SHOW CONSTRAINTS` output. | Must match `GraphApplyProfile.expected_backend_schema_fingerprint`. |

### 8. Bulk load, import, export, and rebuild

Neo4j import docs state that `neo4j-admin database import` writes CSV data to native Neo4j format, supports Parquet, and is intended for initial loads, large imports, and scenarios where offline import is more efficient than transactional insertion. Docs also distinguish full and incremental modes, note that import changes are not captured by CDC, and document a dry-run mode that validates input and estimates database size without writing data.[^15]

Cadastre rebuild implications:

| Neo4j capability | Cadastre rebuild use | Boundary |
| --- | --- | --- |
| Full offline import | Candidate for full graph rebuild from authoritative lakehouse graph-delta export. | Must produce `GraphRebuildManifest`; import success is not correctness proof. |
| Incremental import | Candidate for large batch apply to an offline/read-only staging DB. | Must not bypass `GraphDeltaSet` or apply result evidence. |
| CSV/Parquet import | Useful lakehouse export format. | File checksums and canonical row order required. |
| Dry run | Useful schema/input validation. | Dry run is validation evidence only. |
| Schema file import | Useful to create schema before import. | Must be generated from `GraphReadModelSchemaProfile`. |

### 9. Operations, monitoring, and security

Neo4j operations docs expose monitoring through metrics, currently executing query inspection, logs, security event logs, query logs, and Causal Cluster analysis.[^16] Neo4j also supports RBAC and fine-grained privileges in Enterprise-oriented docs.[^17]

Cadastre must layer its own graph authorization and redaction contract on top. Backend RBAC may restrict operator and service accounts, but it must not decide graph-property redaction, raw-payload exposure, evidence permissions, or source authority.

### 10. Testing and validation patterns

Cadastre should add Neo4j-specific fixtures:

| Fixture | Required assertion |
| --- | --- |
| Constraint preflight missing | `GRAPH_SCHEMA_NOT_READY` before mutation. |
| Stale schema fingerprint | `GRAPH_SCHEMA_FINGERPRINT_MISMATCH` before mutation. |
| Duplicate node selector | Constraint error mapped to non-retryable apply failure. |
| Duplicate edge selector | Relationship uniqueness violation mapped to non-retryable apply failure. |
| Retryable transaction failure | Transaction function retries only idempotent batch. |
| Cluster writer failover | Apply either commits once with recorded writer/bookmark or fails without partial ambiguity. |
| Path tie case | Cadastre sorts paths deterministically after backend return. |
| Rebuild import | Same `GraphDeltaSet` and schema profile produce same output checksum or same declared failure. |

### 11. Confirmed findings

| Finding | Evidence class |
| --- | --- |
| Neo4j is a graph database using nodes, relationships, Cypher, and ACID transactions. | `confirmed source fact`[^9] |
| Current Cypher docs distinguish Cypher 25 from older Cypher 5 behavior. | `documentation fact`[^12] |
| Neo4j supports node and relationship constraints over uniqueness, existence, type, and keys. | `documentation fact`[^10] |
| `neo4j-admin database import` supports full and incremental import, CSV/Parquet, and dry-run behavior. | `documentation fact`[^15] |
| Cluster write/read availability depends on primary/secondary topology and failure tolerance. | `documentation fact`[^14] |

### 12. Source-grounded inferences

| Inference | Confidence |
| --- | --- |
| Neo4j is the strongest reference backend for Cadastre’s labeled-property-graph MVP. | High |
| Neo4j can represent Cadastre projected nodes, typed edges, properties, evidence refs, assertion state, confidence, valid/known time, and schema fingerprints when the projection schema is explicit. | High |
| Deterministic path ordering must be Cadastre-owned, because backend traversal order and shortest-path tie behavior cannot be treated as the API contract. | High |
| Neo4j import is promising for full rebuild but must be wrapped by `GraphRebuildManifest` and `GraphIndexConsistencyCheck`. | High |

### 13. Cadastre transfer candidates

| Backend pattern | Cadastre contract area | Transfer stance |
| --- | --- | --- |
| Labeled property graph | `GraphProjectionProfile`, `GraphEdgeSemantics` | `adapt` |
| Cypher schema/constraints | `GraphReadModelSchemaProfile` | `adopt` as backend rows |
| Driver bookmarks and causal consistency | `GraphApplyResult`, `DerivedViewState` | `adapt` |
| Offline and incremental import | `GraphRebuildManifest` | `adapt` |
| RBAC and monitoring | Operations and health | `adapt` |
| GDS/APOC procedures | Analysis references only | `cautionary_reference` |

### 14. Concepts that must not transfer

| Neo4j-native concept | Must not define Cadastre behavior |
| --- | --- |
| Internal node ID, relationship ID, or `elementId` | Cadastre canonical IDs, graph node IDs, edge IDs, evidence refs, or replay selectors. |
| Cypher query result order | Cadastre deterministic path or pagination order. |
| APOC/GDS result | Gold fact truth, identity, risk scoring, or source authority. |
| Constraint success | Identity decision or source completeness. |
| Import success | Rebuild correctness or lakehouse replay eligibility. |
| Cluster replication success | Authoritative table state, source completeness, or graph rebuild validity. |

### 15. Gaps, risks, stale assumptions, and unresolved questions

| Item | Required verification step |
| --- | --- |
| Cypher 25 behavior and GQL compatibility details | Run Cadastre query suite in Cypher 25 and explicitly compare any Cypher 5 compatibility mode. |
| Enterprise feature boundaries for relationship key constraints and fine-grained security | Verify license/edition requirements against deployment profile. |
| Constraint/index creation blocking behavior under production load | Run schema migration and concurrent query/write tests. |
| Cluster failover and idempotent retries | Run writer failover while applying staged `GraphDeltaSet` batches. |
| Import rebuild determinism | Run canonical lakehouse export to import twice and compare graph checksums. |

## Memgraph independent analysis

### 1. Source inventory (Memgraph)

| Field | Content |
| --- | --- |
| Repository URL | `https://github.com/memgraph/memgraph` |
| Documentation URLs | Memgraph docs for fundamentals, storage modes, indexes, constraints, storage access, high availability, import, monitoring, authentication and authorization. |
| Release, branch, tag, or commit inspected | Public repository and current docs inspected through web on 2026-05-16. Release notes exposed Memgraph `v3.10.1` on 2026-05-15. Exact commit was not pinned.[^18] |
| Evidence type | `confirmed source fact`, `documentation fact`, `source-grounded inference`, `Cadastre applicability inference`. |
| Scope inspected | Repository README, release notes, data durability/storage modes, on-disk memory behavior, storage accessors, indexes/constraints, replication, import, monitoring/security docs. |
| Scope not inspected | Full C/C++ source tree, parser/planner tests, MAGE modules, custom procedure runtime, stream ingestion runtime, HA failover under load, benchmarks. |
| Reliability and freshness risk | High for official docs; medium for Cypher compatibility and Enterprise/community feature boundaries. |

### 2. Project purpose and non-purpose (Memgraph)

Memgraph is built as a high-performance graph database with Cypher compatibility, in-memory and on-disk storage modes, graph algorithms, custom modules, streaming ingestion, and high-availability features. The repository README describes it as a high-performance in-memory graph database with ACID-compliant transactions, Cypher compatibility, high availability, MAGE algorithms, custom modules, and import support.[^18]

For Cadastre, Memgraph is a candidate for high-performance graph serving and real-time graph analytics. It does not provide Cadastre’s raw-first lakehouse, bitemporal fact model, identity resolver, source completeness authority, graph rebuild authority, or backend-neutral query contract.

### 3. Repository and system architecture (Memgraph)

| Area | Memgraph role | Cadastre relevance |
| --- | --- | --- |
| Storage engine | Vertices, edges, properties, labels, accessors, in-memory and on-disk modes. | Must be captured in `GraphApplyProfile.storage_mode`. |
| Cypher engine | Cypher-compatible query execution. | Candidate for Cypher-like query contract, but compatibility must be proven. |
| Index and constraint subsystem | Label, label-property, composite, edge-type, edge-property indexes; uniqueness/existence constraints. | Backend-specific schema profile rows. |
| Storage modes | `IN_MEMORY_TRANSACTIONAL`, `IN_MEMORY_ANALYTICAL`, `ON_DISK_TRANSACTIONAL`. | Apply and rebuild behavior changes by mode. |
| Replication / HA | Main/replica and coordinator features with SYNC, STRICT_SYNC, ASYNC behaviors. | Must be represented in apply result and lag policy. |
| Streams/import | Kafka/Pulsar/Redpanda/file/import surfaces. | Projection inputs only; must not be authoritative source inputs by default. |
| MAGE/custom modules | Algorithms and custom procedures. | Analysis reference only. |
| Monitoring/security | Metrics, auth, roles, database-specific privileges. | Ops surface; not graph-property redaction authority. |

### 4. Graph data model (Memgraph)

Memgraph uses a labeled property graph model with nodes, labels, relationships/edges, properties, indexes, constraints, and Cypher-like queries. It can represent Cadastre graph node IDs, edge IDs, node types, edge types, evidence properties, assertion state, confidence, valid-time fields, known-time fields, expired/retracted state, and no-op outcomes as properties when Cadastre defines the projection schema explicitly.

Storage mode is a first-class Cadastre concern. Memgraph docs describe three storage modes: in-memory transactional, in-memory analytical, and on-disk transactional. In-memory transactional uses periodic snapshots and WAL; in-memory analytical omits periodic snapshots/WAL but permits manual snapshots; on-disk uses RocksDB WAL.[^19]

### 5. Query language and traversal model (Memgraph)

Memgraph claims Cypher compatibility, but Cadastre must treat compatibility as a validation target, not a contract. Differences in functions, procedures, constraints, indexes, path matching, ordering, planner hints, query memory limits, and cancellation can affect `QueryGraph`.

Required Memgraph query contract checks:

| Concern | Required Cadastre behavior |
| --- | --- |
| Cypher compatibility | Run every Cadastre query scenario and pin supported query subset. |
| Path search | Apply deterministic Cadastre ordering after backend traversal. |
| Query memory | Set and record query memory limit when available; map exceeded memory to `GRAPH_QUERY_RESOURCE_LIMIT`. |
| Query cancellation | Enforce Cadastre timeout; record backend cancellation evidence. |
| Procedures/MAGE | Disabled by default for production graph serving unless an analysis-only profile permits read-only execution. |
| Streams/triggers | Disabled for graph mutation unless future PRD allows projection-input stream mode. |

### 6. Transaction, consistency, and apply model (Memgraph)

Memgraph storage access docs describe storage accessors that provide a transactional view and concurrency-safety guarantees. Replication docs describe main/replica operation and replication modes including synchronous, strict synchronous, and asynchronous modes.[^20]

Storage-mode implications for Cadastre:

| Storage mode | Apply implication | Rebuild implication | Required default |
| --- | --- | --- | --- |
| `IN_MEMORY_TRANSACTIONAL` | ACID-oriented graph apply with WAL/snapshots. | Rebuild can load then snapshot. | Allowed for MVP if durability and backup policy pass. |
| `IN_MEMORY_ANALYTICAL` | Not safe for production graph apply requiring transactional durability. | Useful for validation/import shadow only. | Production default `reject`. |
| `ON_DISK_TRANSACTIONAL` | Durable on-disk apply, but on-disk limitations and HA support must be validated. | Candidate for durable serving. | Allowed only after on-disk HA and memory tests. |

Memgraph docs state that in on-disk mode indexes and constraints are stored in RocksDB, a single transaction must still fit in memory, and on-disk storage lacks replication and high availability in the inspected docs.[^21]

### 7. Indexes, constraints, and schema preflight (Memgraph)

Memgraph docs describe existence and uniqueness constraints. Uniqueness constraints cover label-property pairs and do not automatically create label-property indexes. Index docs describe label, label-property, composite, edge-type, and edge-property indexes, and state that index creation is non-blocking for reads while writes may briefly pause.[^22]

Required `GraphReadModelSchemaProfile` rows for Memgraph:

| Schema concern | Memgraph row |
| --- | --- |
| Unique node selector | Uniqueness constraint over label and `node_id`; explicit label-property index also required. |
| Unique edge selector | Edge property index plus application-level uniqueness unless backend uniqueness supports required relationship semantics. |
| Evidence lookup | Label-property or composite index over evidence refs. |
| Edge lookup | Edge-type and edge-property indexes when supported and feature flag enabled. |
| Schema fingerprint | Canonical `SHOW INDEX INFO` / `SHOW CONSTRAINTS` output plus storage mode and database settings. |
| Missing index | Reject production apply with `GRAPH_SCHEMA_NOT_READY`; do not rely on planner fallback. |

### 8. Bulk load, import, export, and rebuild (Memgraph)

Memgraph import docs describe `LOAD CSV`, concurrent import, compressed files, analytical mode for faster import with reduced ACID behavior, and edge import mode for on-disk storage.[^23]

Cadastre rebuild implications:

| Memgraph capability | Cadastre use | Boundary |
| --- | --- | --- |
| `LOAD CSV` | Rebuild from canonical graph-delta export. | Must be driven by `GraphRebuildManifest`, not source data directly. |
| Analytical mode | Shadow validation or rapid rebuild staging only. | Not production serving unless Cadastre explicitly accepts durability limits. |
| Edge import mode | Large edge load optimization. | Must preserve deterministic edge IDs and schema checks. |
| Streams | Projection-input optimization only. | Source streams are not source authority or completeness. |

### 9. Operations, monitoring, and security (Memgraph)

Memgraph monitoring docs state that Enterprise metrics can track high availability, transactions, query latencies, snapshot recovery latencies, triggers, Bolt messages, indexes, streams, memory, operators, and sessions through an HTTP server.[^24] Security docs describe authentication and authorization, users, roles, fine-grained access control, and external auth integrations. Search results also indicate Community limitations for auth/authorization and Enterprise support for roles, SAML/OIDC/LDAP, and multi-database roles.[^25]

Cadastre must record Memgraph edition, license, storage mode, auth mode, monitoring availability, HA mode, and stream/procedure enablement in `VersionManifest` or deployment profile when any affects production behavior.

### 10. Testing and validation patterns (Memgraph)

Cadastre should add Memgraph-specific fixtures:

| Fixture | Required assertion |
| --- | --- |
| Storage mode matrix | Same `GraphDeltaSet` has identical graph checksum or declared unsupported result across each enabled mode. |
| Analytical mode rejection | Production apply rejects analytical mode unless explicitly shadow-only. |
| On-disk transaction memory limit | Large batch fails before mutation or records committed batch boundary. |
| Constraint/index preflight | Missing uniqueness/index rows fail before apply. |
| Edge index availability | Edge lookups use declared indexes or fail plan validation. |
| Replication mode | Strict sync/sync/async apply results include replication evidence and lag. |
| Stream forbidden output | Stream ingestion cannot write authoritative graph state outside `GraphDeltaSet`. |

### 11. Confirmed findings (Memgraph)

| Finding | Evidence class |
| --- | --- |
| Memgraph positions itself as a high-performance, Cypher-compatible graph database with ACID transactions, HA, import, algorithms, and custom modules. | `confirmed source fact`[^18] |
| Memgraph supports multiple storage modes with different durability properties. | `documentation fact`[^19] |
| On-disk mode has memory and HA limitations in inspected docs. | `documentation fact`[^21] |
| Memgraph supports existence/uniqueness constraints and multiple index types including edge indexes. | `documentation fact`[^22] |
| Memgraph supports main/replica replication modes including synchronous and asynchronous variants. | `documentation fact`[^20] |

### 12. Source-grounded inferences (Memgraph)

| Inference | Confidence |
| --- | --- |
| Memgraph can likely serve Cadastre’s current-state/recent-history graph read model when schema and storage mode are tightly constrained. | Medium-high |
| Memgraph’s storage modes are not interchangeable for Cadastre apply and rebuild semantics. | High |
| Memgraph’s Neo4j compatibility must be tested query-by-query rather than assumed. | High |
| Streams and MAGE modules are useful analysis/projection references but unsafe as authoritative Cadastre data paths. | High |

### 13. Cadastre transfer candidates (Memgraph)

| Backend pattern | Cadastre contract area | Transfer stance |
| --- | --- | --- |
| Storage-mode declaration | `GraphApplyProfile`, `VersionManifest` | `adopt` |
| Edge indexes | `GraphReadModelSchemaProfile` | `adapt` |
| Strict replication modes | `GraphApplyResult`, `DerivedViewLagPolicy` | `adapt` |
| Monitoring metrics | Operational health | `adapt` |
| Streams | Projection-input optimization only | `study` |
| MAGE/custom procedures | Read-only analysis references | `cautionary_reference` |

### 14. Concepts that must not transfer (Memgraph)

| Memgraph concept | Must not define Cadastre behavior |
| --- | --- |
| In-memory state or snapshots | Historical truth or replay eligibility. |
| Cypher-compatible claim | Cadastre query contract. |
| Stream ingestion success | Source completeness or graph apply success. |
| MAGE algorithm output | Gold facts, identity, or risk truth. |
| Storage mode default | Cadastre durability or apply behavior without explicit profile row. |
| Replication state | Lakehouse replay eligibility or rebuild correctness. |

### 15. Gaps, risks, stale assumptions, and unresolved questions (Memgraph)

| Item | Required verification step |
| --- | --- |
| Exact Cypher deviations from Neo4j | Run Cadastre query compatibility suite and record unsupported constructs. |
| On-disk HA limitations | Validate current release docs/source and run HA tests. |
| Community versus Enterprise security/monitoring | Pin edition boundaries in deployment profile. |
| Query cancellation/memory limits | Execute timeout and memory-pressure fixtures. |
| Edge import and graph rebuild determinism | Rebuild twice from canonical export and compare checksums. |

## JanusGraph independent analysis

### 1. Source inventory (JanusGraph)

| Field | Content |
| --- | --- |
| Repository URL | `https://github.com/JanusGraph/janusgraph` |
| Documentation URLs | JanusGraph docs for data model, Gremlin basics, storage backends, index backends, transactions, eventual consistency, schema/indexing, server, configuration, changelog. |
| Release, branch, tag, or commit inspected | Public repository and docs inspected through web on 2026-05-16. Changelog/release sources identify JanusGraph `1.1.0` as released 2024-11-07; exact repository commit was not pinned.[^26] |
| Evidence type | `confirmed source fact`, `documentation fact`, `source-grounded inference`, `Cadastre applicability inference`. |
| Scope inspected | Repository README, release/changelog, storage/index backend docs, data model, basic Gremlin usage, transactions, eventual consistency, indexing performance, advanced schema. |
| Scope not inspected | Full source, JanusGraph Server runtime, backend-specific runtime for Cassandra/HBase/Scylla/Bigtable/BerkeleyDB, current bulk loading internals, local Gremlin execution, mixed-index synchronization. |
| Reliability and freshness risk | Medium-high for docs; high risk for runtime semantics because behavior depends on selected storage and index backends. |

### 2. Project purpose and non-purpose (JanusGraph)

JanusGraph is built for scale-out graph storage and traversal over large graphs. Its README describes it as a scalable graph database optimized for billions of vertices and edges distributed across multi-machine clusters, supporting transactional database use, concurrent users, complex traversals, and analytic graph queries.[^27]

For Cadastre, JanusGraph is the strongest reference for very large graph serving and distributed storage abstraction. It does not provide a Cadastre lakehouse, Cypher-compatible graph API, bitemporal truth model, identity resolver, source completeness authority, or deterministic graph apply contract.

### 3. Repository and system architecture (JanusGraph)

| Area | JanusGraph role | Cadastre relevance |
| --- | --- | --- |
| TinkerPop/Gremlin layer | Property graph and traversal API. | Requires precise translation from Cadastre `QueryGraph` to Gremlin. |
| Storage backend abstraction | Cassandra/CQL, HBase, BerkeleyDB, ScyllaDB, and other documented backends. | Operational and consistency behavior depends on backend. |
| Index backend abstraction | Composite indexes in storage backend; mixed indexes via Elasticsearch, Solr, or Lucene. | Schema profile must include storage and index backend state. |
| Schema management | Vertex labels, edge labels, property keys, cardinality, multiplicity, indexes, vertex-centric indexes. | Strong reference for typed graph schema and super-node mitigation. |
| Transaction and consistency | Thread-local transactions, backend-dependent ACID, locking options. | Apply semantics require stricter Cadastre wrapper. |
| Batch loading | Load optimization that may disable consistency checks/locking. | Rebuild only under strict validation. |
| JanusGraph Server | Remote Gremlin execution. | Deployment/API surface, not source authority. |

### 4. Graph data model (JanusGraph)

JanusGraph uses Apache TinkerPop’s property graph model. Its docs describe an adjacency-list model in which each vertex stores an adjacency list containing incident edges and properties; each edge is stored twice, once in the adjacency list of each incident vertex, and sort order enables efficient vertex-centric indexes.[^28]

Cadastre mapping:

| JanusGraph concept | Cadastre-safe mapping | Hazard |
| --- | --- | --- |
| Vertex | Graph node. | Vertex IDs must not be Cadastre canonical IDs unless user-supplied and governed. |
| Edge | Graph edge. | Edge multiplicity/direction must be mapped to `GraphEdgeSemantics`. |
| Property key | Graph property field. | Cardinality and data type must be explicit. |
| Vertex label | Node type. | Labels cannot define Cadastre entity truth. |
| Edge label | Edge type. | Edge label cannot define non-implication semantics by itself. |
| Composite index | Backend lookup support. | Exact query shape must be designed for it. |
| Mixed index | Search/range/full-text backend support. | Stale index and backend availability must be checked. |
| Vertex-centric index | Super-node traversal optimization. | Strong transfer candidate for high-degree nodes. |

### 5. Query language and traversal model (JanusGraph)

JanusGraph queries use Gremlin/TinkerPop traversals rather than Cypher pattern matching. Gremlin is a traversal/data-flow language, so Cypher-like patterns must be translated into ordered traversal steps with explicit direction, edge labels, filters, limits, and sort behavior. TinkerPop supports OLTP graph traversal and OLAP graph processing concepts; JanusGraph supports complex traversals and analytic graph queries, but Cadastre must not assume a Cypher-like query semantics.[^29]

Required translation contract:

| Cadastre query construct | Gremlin translation requirement |
| --- | --- |
| Neighbor expansion | `V(seed).bothE()/outE()/inE()` based on `GraphEdgeSemantics.direction`, then vertex projection. |
| Bounded path search | `repeat(...).times(max_depth)` or bounded traversal with explicit simple-path policy. |
| Reverse path search | Reverse edge direction based on semantics, not label name. |
| Edge-type filtering | `hasLabel(edge_type)` or edge property filters with exhaustive mapping. |
| Temporal/confidence/assertion filters | `has()` predicates on edge/node properties; index requirements explicit. |
| Deterministic ordering | Materialize path tuples and sort by Cadastre order after traversal. |
| Pagination | Cursor from sorted path/node/edge tuple, not Gremlin iterator state. |
| Timeout | JanusGraph Server / traversal timeout plus Cadastre timeout. |

### 6. Transaction, consistency, and apply model (JanusGraph)

JanusGraph transaction docs state that all graph interactions occur in transactions, but ACID behavior is backend-dependent. Docs explicitly state JanusGraph can be configured for ACID with BerkeleyDB but generally cannot provide ACID transactions with Cassandra or HBase.[^30]

Eventual consistency docs state that with eventually consistent storage backends such as Cassandra and HBase, JanusGraph provides additional consistency features for locks and uniqueness, but transactions are not isolated, locks are not enabled by default, and locking is not robust against all failure scenarios such as quorum loss; strict/frequent uniqueness constraints should use a backend that provides transactional isolation.[^31]

Cadastre apply implication:

```text
A JanusGraph GraphApplyProfile must declare storage_backend_kind, index_backend_kind, consistency_mode, uniqueness_enforcement_mode, lock_mode, batch_loading_mode, transaction_boundary, retry policy, and stale-index-read policy. Production apply must reject backend combinations whose declared consistency cannot guarantee idempotent reapply and selector uniqueness for the graph delta class being applied.
```

### 7. Indexes, constraints, and schema preflight (JanusGraph)

JanusGraph composite indexes are supported through the primary storage backend; mixed indexes require an index backend such as Elasticsearch, Solr, or Lucene.[^32] JanusGraph docs also describe vertex-centric indexes and warn that unconstrained traversals over high-degree vertices are slow; index order and reindex state must be handled explicitly.[^33]

Required `GraphReadModelSchemaProfile` rows for JanusGraph:

| Schema concern | JanusGraph row |
| --- | --- |
| Unique node selector | Composite index plus backend consistency/locking declaration. |
| Unique edge selector | Edge ID property with composite/mixed lookup plus application-level idempotency. |
| Evidence lookup | Composite or mixed index based on exact query shape. |
| Temporal range lookup | Mixed index if range queries exceed storage-backed composite support. |
| Assertion/confidence lookup | Composite index for equality/band filters; mixed index for range/faceted search. |
| Path query support | Vertex-centric index for high-degree edge labels and temporal sorting. |
| Schema fingerprint | Graph schema definitions plus storage/index backend config plus index status. |
| Index consistency check | Verify composite/mixed index status and synchronization before serving. |

### 8. Bulk load, import, export, and rebuild (JanusGraph)

JanusGraph batch-loading docs in older official versions describe a mode that can disable consistency checks and locking for performance, while placing responsibility for data consistency on the user and requiring schema to be pre-created.[^34] Because this specific page is from older JanusGraph docs, it must be revalidated against the current release before PRD amendment.

Cadastre rebuild implications:

| JanusGraph capability | Cadastre use | Boundary |
| --- | --- | --- |
| Batch loading | Candidate for full rebuild into empty graph. | Only from validated `GraphDeltaSet` export and with post-load consistency checks. |
| Storage backend snapshot/restore | Disaster recovery for serving state. | Must not substitute for `GraphRebuildManifest`. |
| Transaction log | Operational change observation. | Transaction log order and at-least-once processors must not become Cadastre delta truth. |
| Reindex | Required after schema/index changes. | Must be included in `GraphIndexConsistencyCheck`. |

### 9. Operations, monitoring, and security (JanusGraph)

JanusGraph operational complexity is backend-compositional. A deployment’s backup, restore, monitoring, security, HA, and failure modes depend on JanusGraph Server plus the selected storage backend and index backend. Official configuration docs identify storage and index backend options as configuration-controlled; JanusGraph Server provides remote Gremlin execution over WebSocket/HTTP.[^35]

Cadastre must treat JanusGraph as a deployment family rather than a single backend. `GraphApplyProfile` must name storage backend, index backend, consistency model, and operational topology.

### 10. Testing and validation patterns (JanusGraph)

Cadastre should add JanusGraph-specific fixtures:

| Fixture | Required assertion |
| --- | --- |
| Backend matrix | Each supported storage/index backend either passes all apply/query tests or is rejected by profile. |
| Uniqueness conflict | Concurrent duplicate `node_id` and `edge_id` creates exactly one object or fails without ambiguity. |
| Stale mixed index | Serving rejects when index backend lags or stale-index policy fails. |
| Super-node traversal | High-degree node query uses vertex-centric index or fails profile validation. |
| Ghost vertex / edge fork | Drift check detects and refuses repair-by-backend. |
| Gremlin translation parity | Every `QueryGraph` scenario matches expected node/edge/path set and deterministic order. |
| Batch load consistency | Full rebuild post-load manifest and index check match expected checksum. |

### 11. Confirmed findings (JanusGraph)

| Finding | Evidence class |
| --- | --- |
| JanusGraph targets very large distributed graphs and supports OLTP/OLAP-style graph workloads. | `confirmed source fact`[^27] |
| JanusGraph stores graph data through pluggable storage backends. | `documentation fact`[^32] |
| Composite indexes use storage backend support; mixed indexes require an index backend such as Elasticsearch, Solr, or Lucene. | `documentation fact`[^32] |
| Transaction/ACID behavior is backend-dependent; Cassandra/HBase-style backends do not generally provide ACID transactions. | `documentation fact`[^30] |
| Eventually consistent backends require special consistency/locking handling and still carry failure risks. | `documentation fact`[^31] |

### 12. Source-grounded inferences (JanusGraph)

| Inference | Confidence |
| --- | --- |
| JanusGraph is the strongest scale-out candidate but imposes the highest contract burden. | High |
| JanusGraph is not a drop-in Cypher backend; Cadastre must define a Gremlin translation contract. | High |
| Storage/index backend selection materially changes apply, consistency, index freshness, and operational behavior. | High |
| JanusGraph is better suited for current-state/recent-history graph serving at large scale than for full historical graph serving unless Cadastre makes retention/query semantics explicit. | Medium-high |

### 13. Cadastre transfer candidates (JanusGraph)

| Backend pattern | Cadastre contract area | Transfer stance |
| --- | --- | --- |
| Vertex-centric indexes | `GraphReadModelSchemaProfile` path-query rows | `adopt` |
| Storage/index backend abstraction | `GraphApplyProfile`, `VersionManifest` | `adopt` |
| Mixed index status checks | `GraphIndexConsistencyCheck` | `adapt` |
| Multiplicity/cardinality schema | `GraphEdgeSemantics`, schema profile | `adapt` |
| Gremlin traversal model | `GraphTaxonomyTranslationPolicy`, query translation | `study` |
| Batch loading | `GraphRebuildManifest` | `study` with caution |

### 14. Concepts that must not transfer (JanusGraph)

| JanusGraph concept | Must not define Cadastre behavior |
| --- | --- |
| Gremlin traversal output | Cadastre fact truth, risk scoring, or deterministic path order. |
| Backend vertex ID | Cadastre canonical ID or graph node ID unless explicitly user-assigned and governed. |
| Storage backend consistency | Cadastre apply semantics without explicit profile row. |
| Mixed-index search results | Identity, edge, or fact evidence by themselves. |
| Batch-loader success | Rebuild correctness or source completeness. |
| Transaction log | Cadastre `GraphDeltaSet` or authoritative temporal ledger. |

### 15. Gaps, risks, stale assumptions, and unresolved questions (JanusGraph)

| Item | Required verification step |
| --- | --- |
| Current supported storage/index backend matrix | Pin JanusGraph release and verify docs/changelog for current backend versions. |
| Gremlin translation correctness | Build full `QueryGraph` translation and golden corpus. |
| ACID/idempotent apply | Test with each candidate storage backend under concurrent writes and failures. |
| Stale index behavior | Create mixed-index lag scenarios and validate serving refusal. |
| Batch loading current semantics | Recheck current docs/source and run rebuild/load tests. |
| Security/monitoring | Validate JanusGraph Server and backend security/monitoring stack. |

## ArangoDB independent analysis

### 1. Source inventory (ArangoDB)

| Field | Content |
| --- | --- |
| Repository URL | `https://github.com/arangodb/arangodb` |
| Documentation URLs | ArangoDB 3.12 docs for data modeling/graphs, AQL traversals, shortest paths, transactions, indexes, ArangoSearch, import, backup/restore, sharding, SmartGraphs, server options/security. |
| Release, branch, tag, or commit inspected | Public repository and current 3.12 docs inspected through web on 2026-05-16. Docs/release sources surfaced 3.12.x content and 3.12.9 release notes. Exact commit was not pinned.[^36] |
| Evidence type | `confirmed source fact`, `documentation fact`, `source-grounded inference`, `Cadastre applicability inference`. |
| Scope inspected | Repository README, graph docs, traversal/shortest-path docs, transaction docs, stream transaction docs, indexes/search docs, backup/import docs, sharding/SmartGraph docs, server options. |
| Scope not inspected | Full C++ source, local cluster, EnterpriseGraph/SmartGraph runtime, OneShard transaction harness, AQL optimizer plans, vector/geo deep tests, access-control runtime. |
| Reliability and freshness risk | High for official docs; medium for Enterprise boundaries and cluster-specific behavior. |

### 2. Project purpose and non-purpose (ArangoDB)

ArangoDB is built as a multi-model database with document, key-value, graph, and search capabilities through a single engine and AQL. The repository README describes scalable graph data modeling, native graphs, integrated search, JSON documents, and a single query language.[^37]

For Cadastre, ArangoDB is a multi-model graph/document serving candidate. It can co-locate graph vertices, edges, and derived graph metadata documents, but this convenience must not blur Cadastre raw/silver/gold/projection boundaries.

### 3. Repository and system architecture (ArangoDB)

| Area | ArangoDB role | Cadastre relevance |
| --- | --- | --- |
| Document collections | JSON document storage; vertices are documents. | Candidate for projected node documents and non-authoritative serving metadata. |
| Edge collections | Edges with `_from` and `_to`. | Candidate for graph edges with deterministic Cadastre `edge_id`. |
| Named graphs | Edge definitions, vertex collections, graph integrity functions. | Strong candidate for schema/integrity wrapper. |
| AQL engine | Traversals, path search, shortest paths, filtering, updates. | Query translation target. |
| Indexes | `_key`, edge `_from`/`_to`, persistent, inverted, TTL, geo/vector/multidimensional variants. | Backend schema profile rows. |
| Transactions | AQL, stream, JavaScript transactions; cluster limitations. | Apply semantics depend on shard topology. |
| Cluster/sharding | Shards across DB-Servers; SmartGraphs/EnterpriseGraphs/OneShard/SatelliteGraph features. | Performance and transaction semantics depend on topology. |
| Import/backup | `arangoimport`, `arangodump`, `arangorestore`, hot backup. | Rebuild and restore mechanisms, not truth. |

### 4. Graph data model (ArangoDB)

ArangoDB graph vertices are documents in vertex collections. Edges are documents in edge collections and contain `_from` and `_to`. Named graph docs state that graph modules provide integrity checks by validating edge endpoints, deleting connected edges when a node is deleted through graph interfaces, and preventing incompatible edge definitions, but these guarantees apply only when using named graph interfaces; raw collection/document APIs can bypass them, and existing inconsistencies are not automatically corrected.[^38]

Cadastre mapping:

| ArangoDB concept | Cadastre-safe mapping | Hazard |
| --- | --- | --- |
| Vertex document `_key` | Deterministic `node_id` if generated by Cadastre. | Auto-generated `_key` must not become Cadastre ID. |
| Edge document `_key` | Deterministic `edge_id` if generated by Cadastre. | Raw collection inserts can bypass graph integrity. |
| `_from`, `_to` | Cadastre `src_node_id` and `dst_node_id` references. | Direction must follow `GraphEdgeSemantics`. |
| Named graph | Backend schema/integrity wrapper. | Named graph integrity is bypassable through raw collection APIs. |
| Document fields | Graph properties. | Document convenience must not store authoritative raw/silver/gold. |
| ArangoSearch/inverted indexes | Search support for serving. | Eventually consistent search results must not be evidence. |

### 5. Query language and traversal model (ArangoDB)

AQL supports graph traversals with explicit `OUTBOUND`, `INBOUND`, or `ANY` direction, depth bounds, graph names or edge collections, `PRUNE`, path variables, and traversal options. Traversal docs state that DFS, BFS, and weighted traversal orders are available, and that equal-cost weighted paths are returned in non-deterministic order.[^39]

Shortest path docs state that if multiple shortest paths exist, ArangoDB returns one path of lowest weight, or a random one if there are several with the same weight.[^40]

Cadastre implication: AQL can implement Cadastre path search, but Cadastre must never rely on backend tie order. Cadastre must materialize returned candidates within bounded query limits and apply its own path sort.

### 6. Transaction, consistency, and apply model (ArangoDB)

ArangoDB transaction docs state that single-server transactions are fully ACID, single-document operations in clusters are fully ACID, and multi-document or multi-collection cluster transactions are limited unless a single-shard or OneShard-style arrangement applies. Cluster transaction limitations state that commits are atomic per involved DB-Server, but there is no global consensus; multi-DB-server commit atomicity is not guaranteed.[^41]

Stream transaction docs state that the client starts a transaction, specifies collections upfront, commits or aborts explicitly, keeps transactions short, and must respect idle timeout and transaction size limits.[^42]

Cadastre apply rule for ArangoDB:

```text
An ArangoDB GraphApplyProfile must declare deployment_mode, graph_kind, vertex_collections, edge_collections, shard_policy, single_shard_requirement, transaction_kind, collection_read_write_set, transaction_size_limit, idle_timeout_seconds, and cluster_atomicity_policy. Production apply must reject multi-shard graph mutations when the declared atomicity cannot prove deterministic all-or-nothing or committed-batch idempotence.
```

### 7. Indexes, constraints, and schema preflight (ArangoDB)

ArangoDB index docs state that `_key` is automatically indexed for every collection and edge collections automatically index `_from` and `_to`. Index docs also describe persistent, inverted, TTL, and multidimensional indexes; search/index docs indicate inverted indexes and ArangoSearch views are eventually consistent, while regular indexes are immediately consistent.[^43]

ArangoDB 3.12 feature docs also describe vector-index improvements, including sparse vector indexes in 3.12.6.[^44]

Required `GraphReadModelSchemaProfile` rows for ArangoDB:

| Schema concern | ArangoDB row |
| --- | --- |
| Unique node selector | `_key = node_id` with graph vertex collection and optional persistent index over `node_type`. |
| Unique edge selector | `_key = edge_id` in edge collection; persistent index over `edge_type` and source/target fields. |
| Evidence lookup | Persistent index over evidence ref fields; inverted index only when stale-search policy allows. |
| Temporal range lookup | Persistent index over valid/known fields. |
| Edge traversal | Automatic `_from`/`_to` indexes plus edge collection partitioning/sharding policy. |
| Search/vector fields | Disabled by default for authority-sensitive queries; enabled only as non-authoritative analysis. |
| Schema fingerprint | Collection names/options, graph definitions, indexes, shard settings, search index consistency settings. |

### 8. Bulk load, import, export, and rebuild (ArangoDB)

ArangoDB docs describe `arangoimport` for JSON, CSV, and TSV imports, and `arangodump`/`arangorestore` plus hot backup behavior for backup/restore.[^45]

Cadastre rebuild implications:

| ArangoDB capability | Cadastre use | Boundary |
| --- | --- | --- |
| `arangoimport` | Rebuild from canonical graph node/edge export. | Must use deterministic `_key` values and validate named graph integrity. |
| `arangodump`/`arangorestore` | Serving-state backup/restore. | Backup success does not prove source completeness or lakehouse replay. |
| Hot backup | Operational recovery. | Must still be paired with `GraphRebuildManifest` when validating authoritative rebuild. |
| Named graph validation | Post-load integrity check. | Does not repair raw collection bypass inconsistencies. |

### 9. Operations, monitoring, and security (ArangoDB)

ArangoDB clustering divides collection data into shards across DB-Servers; SmartGraphs use value-based sharding to increase traversal locality, while normal graph queries can still use the same query surface.[^46] Server options include RocksDB encryption-at-rest key configuration options, and official docs cover authentication/authorization areas, but detailed auth/RBAC behavior was not deeply inspected in this pass.[^47]

Cadastre must capture ArangoDB deployment mode, cluster/shard policy, named graph definition, transaction type, search/index consistency settings, backup profile, and access-control mode as graph-serving profile inputs when they affect output, consistency, or redaction.

### 10. Testing and validation patterns (ArangoDB)

Cadastre should add ArangoDB-specific fixtures:

| Fixture | Required assertion |
| --- | --- |
| Named graph bypass | Raw collection insert that violates graph definition is detected by consistency check and cannot be served current. |
| Multi-shard transaction failure | Apply fails with explicit partial-apply status or is rejected before mutation. |
| OneShard/single-shard apply | Apply commits all-or-nothing and records transaction evidence. |
| Equal shortest paths | Cadastre order is stable despite backend random/tie behavior. |
| Inverted-search staleness | Search-backed graph query is marked stale or rejected according to policy. |
| Import rebuild | `arangoimport` rebuild produces expected counts/checksum and graph integrity check. |
| TTL/delete behavior | TTL cleanup cannot expire facts or graph edges without Cadastre projection rule. |

### 11. Confirmed findings (ArangoDB)

| Finding | Evidence class |
| --- | --- |
| ArangoDB is a multi-model database with graph, document, search, JSON, and AQL capabilities. | `confirmed source fact`[^37] |
| Named graphs provide integrity guarantees only when graph interfaces are used; raw collection APIs can bypass them. | `documentation fact`[^38] |
| AQL traversals support direction, bounded depth, pruning, and traversal options, with some nondeterministic equal-cost behavior. | `documentation fact`[^39] |
| Shortest path tie behavior can return one of multiple equal paths randomly. | `documentation fact`[^40] |
| Cluster transaction atomicity is limited across DB-Servers. | `documentation fact`[^41] |
| `_key` and edge `_from`/`_to` indexes are automatic, and inverted/search indexes are eventually consistent. | `documentation fact`[^43] |

### 12. Source-grounded inferences (ArangoDB)

| Inference | Confidence |
| --- | --- |
| ArangoDB can serve Cadastre graph read-model nodes and edges as documents, but transaction determinism depends on shard topology. | High |
| Named graph integrity is valuable but insufficient unless raw collection writes are controlled. | High |
| ArangoSearch/inverted/vector features are useful for analysis/exploration but must not define identity or evidence. | High |
| ArangoDB is attractive for graph/document co-modeling, but this is also its strongest boundary risk for Cadastre. | High |

### 13. Cadastre transfer candidates (ArangoDB)

| Backend pattern | Cadastre contract area | Transfer stance |
| --- | --- | --- |
| Named graph definitions | `GraphReadModelSchemaProfile` | `adapt` |
| Deterministic `_key` IDs | `GraphProjectionProfile`, `GraphApplyProfile` | `adopt` with Cadastre-owned key generation |
| AQL traversal patterns | `QueryGraph` backend adapter | `adapt` |
| Stream transactions | `GraphApplyProfile` | `study` |
| Single-shard / OneShard topology | Deterministic apply profile | `study` |
| Document-plus-graph metadata | Graph serving metadata only | `cautionary_reference` |

### 14. Concepts that must not transfer (ArangoDB)

| ArangoDB concept | Must not define Cadastre behavior |
| --- | --- |
| Raw collection document state | Canonical raw/silver/gold truth. |
| Auto-generated `_key` | Cadastre node/edge/canonical IDs. |
| Named graph integrity alone | Graph rebuild correctness or source completeness. |
| TTL indexes | Fact retraction, graph expiry, or cleanup permission. |
| ArangoSearch/vector/geo result | Identity, fact evidence, or graph edge truth. |
| Multi-model convenience | Permission to co-store authoritative Cadastre records in graph backend. |

### 15. Gaps, risks, stale assumptions, and unresolved questions (ArangoDB)

| Item | Required verification step |
| --- | --- |
| EnterpriseGraph/SmartGraph/OneShard feature and license boundaries | Verify official 3.12/3.13 feature matrix and run topology tests. |
| AQL plan and traversal performance | Run Cadastre query suite with explain/profiling and high-degree nodes. |
| Cluster transaction behavior | Fail multi-shard and single-shard apply under fault injection. |
| Search/vector/geo index consistency | Validate stale-read policy and query labels. |
| Raw collection bypass prevention | Test service account privileges and consistency scans. |

## Cross-backend comparison matrices

### Backend fit summary

| Backend | Best-fit Cadastre use | Strongest architectural value | Strongest caution | Transfer stance |
| --- | --- | --- | --- | --- |
| Neo4j | MVP labeled-property-graph serving, Cypher-like graph query, deterministic batch apply with mature schema constraints. | Mature Cypher, constraints, drivers, import, clustering, monitoring. | Neo4j-specific Cypher, APOC/GDS, internal IDs, and cluster/import semantics can leak into Cadastre contracts. | `adapt` |
| Memgraph | High-performance current-state/recent-history graph serving and real-time graph analytics exploration. | Cypher-compatible model, storage modes, edge indexes, replication, streaming/import orientation. | Storage-mode differences and incomplete Neo4j compatibility can break deterministic apply/query. | `study` to `adapt` |
| JanusGraph | Very large distributed graph serving where scale-out is more important than Cypher convenience. | TinkerPop scale-out abstraction, storage/index backend flexibility, vertex-centric indexes. | Highest semantic and operational burden: Gremlin translation, backend consistency, index freshness, super-nodes. | `study` |
| ArangoDB | Graph/document serving where derived graph metadata and traversal queries benefit from one engine. | Multi-model documents plus graph, AQL traversals, named graphs, import/backup, flexible indexes. | Raw collection bypass and cluster transaction limits can violate graph integrity/apply determinism. | `adapt` with caution |

### Query language translation matrix

| Cadastre query capability | Neo4j / Cypher | Memgraph / Cypher-compatible | JanusGraph / Gremlin | ArangoDB / AQL | Required Cadastre contract |
| --- | --- | --- | --- | --- | --- |
| Neighbor expansion | `MATCH (n)-[e]-(m)` with direction by edge type. | Same shape if supported. | `bothE/outE/inE` then `otherV/inV/outV`. | `FOR v,e IN 1..d OUTBOUND/INBOUND/ANY`. | `QueryGraph.neighbor_expansion` must define direction, filters, depth, authorization, order. |
| Bounded path search | Variable-length path; shortest/all shortest where supported. | Cypher-compatible bounded path. | `repeat` / `emit` / `times` traversal. | AQL traversal with min/max depth. | Backend result order ignored; Cadastre sorts paths. |
| Reverse path search | Reverse relationship direction in pattern. | Same if supported. | Use opposite traversal step. | Use opposite AQL direction. | Reverse semantics are declared per edge type. |
| Edge-type filtering | Relationship type or property filter. | Relationship type or property filter. | Edge label or edge property. | Edge collection/type property. | Closed edge-type mapping table. |
| Node-type filtering | Labels / node properties. | Labels / node properties. | Vertex label / property. | Vertex collection/type property. | Closed node-type mapping table. |
| Assertion-state filtering | Edge/node property filter. | Edge/node property filter. | `has('assertion_state', ...)`. | `FILTER e.assertion_state == ...`. | Indexed assertion-state property. |
| Confidence filtering | Range predicate. | Range predicate. | Gremlin range predicate. | AQL range filter. | Confidence stored as canonical decimal or sortable numeric. |
| Valid-time filtering | Range/interval predicate. | Range/interval predicate. | `has` predicates; may need mixed index. | AQL interval filter. | Half-open interval rule and temporal index row. |
| Known-time filtering | Range/interval predicate. | Range/interval predicate. | `has` predicates; may need mixed index. | AQL interval filter. | Same as valid-time; backend-independent semantics. |
| Evidence-reference inclusion | Follow evidence properties or query by evidence index. | Same. | Vertex/edge properties plus evidence index. | Document properties plus persistent index. | Evidence returned as Cadastre refs, not backend IDs. |
| Deterministic path ordering | Must post-sort; Cypher tie order not authority. | Must post-sort. | Must post-sort Gremlin results. | Must post-sort because AQL equal-cost/shortest path ties may be nondeterministic. | Sort by PRD path ordering. |
| Pagination | Stable sort key plus cursor; do not use backend natural order. | Same. | Same. | Same. | Cursor token encodes sorted tuple and query checksum. |
| Timeout | Driver/server tx timeout plus Cadastre timeout. | Query timeout/cancel plus Cadastre timeout. | Server traversal timeout plus Cadastre timeout. | AQL query/transaction timeout plus Cadastre timeout. | Default 30s, max 300s unless profile narrows. |
| Authorization filtering | Backend RBAC plus Cadastre redaction. | Backend auth plus Cadastre redaction. | Server/backend auth plus Cadastre redaction. | Users/permissions plus Cadastre redaction. | Backend auth never replaces `GraphPropertyEvidencePolicy`. |
| Graph-property redaction | Projection/redaction in return. | Projection/redaction in return. | Projection/redaction in traversal result. | AQL `RETURN` shape/redaction. | Redaction must run before response and be testable. |

### Graph model mapping matrix

| Cadastre concept | Neo4j | Memgraph | JanusGraph | ArangoDB | Mapping risk | Required mitigation |
| --- | --- | --- | --- | --- | --- | --- |
| Graph node ID | Property plus uniqueness/key constraint. | Property plus uniqueness constraint/index. | Vertex property plus composite index/lock. | Document `_key` or property. | Backend-generated IDs leak. | Cadastre-generated deterministic ID; reject backend-generated ID. |
| Graph edge ID | Relationship property with uniqueness/key where supported. | Edge property plus edge index/application uniqueness. | Edge property plus index/application uniqueness. | Edge document `_key`. | Edge uniqueness differs by backend. | Declare unique edge selector row and idempotent upsert rule. |
| Node type | Label/property. | Label/property. | Vertex label/property. | Collection/type property. | Taxonomy leakage. | `GraphTaxonomyTranslationPolicy`. |
| Edge type | Relationship type. | Relationship type. | Edge label. | Edge collection/type property. | Direction/non-implication leakage. | `GraphEdgeSemantics`. |
| Edge direction | Native relationship direction. | Native relationship direction. | Out/in edge. | `_from` to `_to`. | Backend direction mistaken for fact semantics. | Direction selected by projection row only. |
| Edge properties | Relationship properties. | Edge properties. | Edge properties. | Edge document fields. | Raw/secret leakage. | `GraphPropertyEvidencePolicy`. |
| Node labels or kinds | Labels. | Labels. | Vertex labels. | Collections/type fields. | External labels become canonical. | Projection metadata only unless mapped. |
| Evidence reference properties | Properties. | Properties. | Properties/index. | Document fields/index. | Missing evidence drillback. | Required evidence lookup index. |
| Assertion state | Property. | Property. | Property. | Property. | Backend cleanup deletes state. | State transitions only from graph deltas. |
| Confidence | Property. | Property. | Property. | Property. | Numeric precision drift. | Canonical decimal serialization and sortable numeric mirror if needed. |
| Valid-time fields | Properties. | Properties. | Properties; mixed index may be needed. | Properties. | Range query performance. | Required temporal index row. |
| Known-time fields | Properties. | Properties. | Properties; mixed index may be needed. | Properties. | Same. | Same. |
| Expired edge state | Property or edge expiration. | Property or edge expiration. | Property; avoid TTL as authority. | Property; avoid TTL as authority. | Backend deletion mistaken for fact expiry. | Persist `operation=expire` delta and state property. |
| Retracted edge state | Property or delete/recreate. | Property or delete/recreate. | Property. | Property. | Lost retraction evidence. | Retain retraction state and evidence refs. |
| No-op delta | Not stored unless Cadastre records apply result. | Same. | Same. | Same. | No-op lost. | Persist no-op in `GraphDeltaSet` and `GraphApplyResult`. |
| Backend schema fingerprint | `SHOW INDEXES/CONSTRAINTS`. | `SHOW INDEX INFO/CONSTRAINTS` plus settings. | Schema API plus backend/index status. | Collections/graph definitions/indexes/shards. | Non-comparable fingerprints. | Backend-specific canonicalization profile. |

### Apply semantics matrix

| Apply concern | Neo4j | Memgraph | JanusGraph | ArangoDB | Required `GraphApplyProfile` rule |
| --- | --- | --- | --- | --- | --- |
| Transaction boundary | ACID tx; cluster writer routing. | Transactional by storage mode; transaction must fit memory in some modes. | Backend-dependent transaction behavior. | Single-server ACID; cluster multi-shard limits. | Declare batch size, tx type, topology, and all-or-nothing guarantees. |
| Batched upsert | Cypher `MERGE` batches. | Cypher `MERGE` batches. | Gremlin upsert traversal or backend-specific pattern. | AQL/stream transaction upsert. | Ordered batches from `ordered_delta_refs` only. |
| Retryable errors | Driver retryable tx errors. | Storage/query/replication retry classes. | Backend/index-specific temporary failures. | Transaction conflicts/timeouts/transient cluster errors. | Closed retryable error table per backend. |
| Non-retryable errors | Constraint/schema/query syntax. | Constraint/schema/mode unsupported. | Schema/consistency/index unsupported. | Schema/transaction atomicity/index missing. | Closed non-retryable error table. |
| Partial apply | Avoid inside tx; record committed batches if any. | Storage-mode dependent. | High risk on eventual backend. | High risk in multi-shard cluster. | Default `partial_apply_behavior = reject`; batch commits recorded if allowed. |
| Idempotent reapply | Deterministic IDs and `MERGE`. | Deterministic IDs and `MERGE`. | Deterministic property keys and uniqueness policy. | Deterministic `_key`/upsert. | Same input and schema must produce same final graph checksum. |
| Missing schema | Reject before mutation. | Reject before mutation. | Reject before mutation. | Reject before mutation. | `schema_readiness_required = true`. |
| Missing index | Reject when required by profile. | Reject. | Reject or shadow only. | Reject. | Missing required index = `GRAPH_SCHEMA_NOT_READY`. |
| Constraint violation | Non-retryable unless idempotent same-state match. | Non-retryable unless same-state match. | Non-retryable or profile rejection. | Non-retryable unless same document same content. | Map to `GRAPH_CONSTRAINT_VIOLATION`; record delta IDs. |
| Stale backend schema fingerprint | Reject. | Reject. | Reject. | Reject. | Expected fingerprint required in apply request. |
| Cluster failover | Driver routing/bookmark evidence. | Replication/coordinator evidence. | Backend-specific failure evidence. | Cluster/shard tx evidence. | Apply result records writer/topology/failure. |
| Read-after-write behavior | Bookmark/causal read. | Verify query against committed batch. | Verify with consistency policy/index freshness. | Verify after commit within transaction or post-commit. | Required verification query and checksum. |
| Graph apply result evidence | Tx/bookmark/db/schema. | Storage mode/replication/schema. | Backend/index/lock/status. | Tx ID/status/shard/schema. | Persist backend-specific evidence rows. |

### Index and constraint matrix

| Schema concern | Neo4j | Memgraph | JanusGraph | ArangoDB | Required `GraphReadModelSchemaProfile` row |
| --- | --- | --- | --- | --- | --- |
| Unique node selector | Node key/uniqueness constraint. | Uniqueness constraint plus index. | Composite index plus consistency/lock policy. | `_key` or unique persistent index. | `unique_node_selector`. |
| Unique edge selector | Relationship key/uniqueness where available. | Edge property index and uniqueness policy. | Edge property index/application uniqueness. | Edge `_key`. | `unique_edge_selector`. |
| Relationship/edge property lookup | Relationship property index. | Edge-type/edge-property index. | Composite/mixed index. | Edge collection persistent index. | `edge_property_lookup_index`. |
| Evidence lookup index | Node/relationship property index. | Label-property/composite index. | Composite/mixed index. | Persistent index. | `evidence_lookup_index`. |
| Temporal range lookup | Range index. | Label-property/composite where supported. | Mixed index often required. | Persistent index. | `temporal_range_index`. |
| Assertion-state lookup | Equality/composite index. | Label-property/composite index. | Composite index. | Persistent index. | `assertion_state_index`. |
| Confidence-band lookup | Equality/range index. | Label-property/composite. | Composite/mixed. | Persistent index. | `confidence_band_index`. |
| Path query support | Relationship type/label indexes and planner. | Edge indexes. | Vertex-centric indexes for high-degree vertices. | `_from`/`_to` edge indexes and graph sharding. | `path_query_support_row`. |
| Graph type taxonomy support | Labels/types. | Labels/types. | Vertex/edge labels. | Collections/type fields. | `taxonomy_backend_mapping_row`. |
| Index creation blocking behavior | Must be measured per index. | Non-blocking reads; writes may briefly pause. | Reindex may be required and unavailable until complete. | Depends on index; search views eventually consistent. | `index_creation_mode` and serving gate. |
| Index consistency check | `SHOW INDEXES` online state. | `SHOW INDEX INFO` plus settings. | Index status/reindex completion. | Index list and search freshness. | `GraphIndexConsistencyCheck`. |
| Schema fingerprinting | Canonical constraints/indexes. | Canonical indexes/constraints/settings. | Schema definitions/backend config/index status. | Graph definitions/collections/indexes/shards. | `backend_schema_fingerprint_algorithm`. |

### Rebuild and import matrix

| Rebuild concern | Neo4j | Memgraph | JanusGraph | ArangoDB | Required `GraphRebuildManifest` rule |
| --- | --- | --- | --- | --- | --- |
| Full rebuild from lakehouse graph deltas | Offline import or transactional apply. | CSV import / LOAD CSV / bulk staging. | Batch loading into empty graph. | `arangoimport` or AQL bulk apply. | Rebuild inputs are authoritative snapshot refs and persisted graph deltas. |
| Incremental apply from graph deltas | Driver transactions. | Driver transactions. | Gremlin transactions. | AQL/stream transactions. | Incremental apply must emit `GraphApplyResult`. |
| Offline import | Strong support. | Import mode support. | Batch load candidate. | `arangoimport`. | Offline import must not serve until checks pass. |
| Online import | Transactional Cypher. | `LOAD CSV`/transactional writes. | Gremlin writes. | AQL/stream writes. | Online import is just graph apply with apply profile. |
| Export/dump/restore | Neo4j admin/backup tools. | Backup/snapshot tools. | Storage/index backend tools. | `arangodump`/restore/hot backup. | Restore is operational, not rebuild correctness. |
| Validation before serve | Schema/index checks. | Schema/index/storage checks. | Schema/index/backend consistency checks. | Named graph/index/shard checks. | `GraphIndexConsistencyCheck.status = passed`. |
| Evidence index rebuild | Index rebuild/recreate. | Index recreate. | Mixed-index reindex. | Persistent/inverted index rebuild. | Evidence lookup count and checksum must match. |
| Schema/index rebuild | Schema file/commands. | Index/constraint commands. | Schema management and reindex. | Graph/index creation. | Schema fingerprint must equal manifest. |
| Rebuild output checksum | Cadastre-owned graph checksum. | Same. | Same. | Same. | Canonical contents checksum required. |
| Derived-view lag update | Apply/rebuild result updates state. | Same. | Same. | Same. | `DerivedViewState.served_dataset_version_ref_id` updated after success. |

### Operations matrix

| Operational concern | Neo4j | Memgraph | JanusGraph | ArangoDB | Cadastre implication |
| --- | --- | --- | --- | --- | --- |
| Deployment topology | Single, cluster, primaries/secondaries. | Single, replication/HA, storage modes. | JanusGraph Server plus storage/index clusters. | Single server or cluster with shards/SmartGraph/OneShard. | Topology must be in backend profile. |
| Clustering/HA | Mature causal cluster. | Replication and HA modes; storage-mode limits. | Depends on storage/index backend. | Cluster DB-Servers/coordinators/agents; multi-shard limits. | HA success does not prove apply determinism. |
| Storage durability | Native database storage, backups/import. | WAL/snapshots or RocksDB depending mode. | Backend-dependent. | RocksDB/document collections/backups. | Durability class recorded in `GraphApplyResult`. |
| Memory model | Native DB cache/page cache. | In-memory and on-disk modes; tx memory limits. | Backend-specific. | RocksDB caches/shards. | Batch size and resource limits are profile fields. |
| Backup/restore | Neo4j admin tools. | Snapshots/backups. | Storage/index backend backups. | Dump/restore/hot backup. | Backup is serving recovery only. |
| Monitoring | Metrics/logs/query/security logs. | Metrics HTTP Enterprise, probes, HA metrics. | JanusGraph plus backend monitoring. | Server/cluster metrics, logs, backups. | Health records map backend metrics to Cadastre health. |
| Access control | Native/RBAC/LDAP/OIDC depending edition. | Enterprise auth/roles/SSO; Community limitations. | JanusGraph Server/backend-specific. | Users/permissions/auth options. | Backend ACL is defense-in-depth, not Cadastre redaction. |
| Upgrade behavior | Active 2025/2026 release line. | Active 3.x line. | 1.1.0 current inspected stable. | 3.12.x current inspected line. | Version manifest must pin backend/driver/schema versions. |
| License/edition boundary | Constraints/security/import features may be edition-specific. | Monitoring/security/HA may be Enterprise. | Mostly OSS plus backend license constraints. | SmartGraphs/EnterpriseGraphs/OneShard/hot backup may be edition-specific. | Feature availability must be profile-gated. |
| Operational complexity | Medium. | Medium. | High. | Medium-high. | Complexity increases validation matrix size. |
| Failure mode most likely to violate Cadastre contracts | Internal ID/Cypher/import semantics treated as truth. | Storage-mode/compatibility assumptions. | Eventual consistency/stale indexes/super-nodes. | Raw collection bypass/multi-shard partial apply/search staleness. | Add rejection tests for each. |

## Backend-neutral graph serving contract implications

### Backend adapter interface

Cadastre needs a backend-neutral adapter interface that every candidate backend must implement.

```text
PreflightGraphBackend(schema_profile, apply_profile) -> GraphBackendPreflightResult
ApplyGraphDelta(graph_delta_set, apply_profile, expected_schema_fingerprint) -> GraphApplyResult
QueryGraph(query_request, query_profile, derived_view_policy) -> QueryGraphResponse
CheckGraphIndexConsistency(schema_profile, expected_counts, expected_checksum) -> GraphIndexConsistencyCheck
RebuildGraphServingState(rebuild_manifest_request) -> GraphRebuildManifest
ComputeBackendSchemaFingerprint(schema_profile) -> BackendSchemaFingerprint
```

Defaults:

| Field | Default |
| --- | --- |
| `schema_readiness_required` | `true` |
| `partial_apply_behavior` | `reject` |
| `idempotent_reapply_required` | `true` |
| `backend_generated_ids_allowed` | `false` |
| `backend_query_order_trusted` | `false` |
| `raw_collection_writes_allowed` | `false` for graph-serving service accounts unless the apply profile explicitly maps them to graph deltas. |
| `stale_search_index_authoritative` | `false` |

### Deterministic schema fingerprint algorithm

```text
ComputeBackendSchemaFingerprint(input):
1. Read backend schema objects required by GraphReadModelSchemaProfile.
2. Normalize every object to a backend-neutral row:
   (object_kind, logical_name, backend_name, target_type, properties, uniqueness, existence, index_kind, status, backend_options).
3. Reject rows whose status is not ready/online unless the schema profile explicitly permits shadow-only status.
4. Sort rows by object_kind, logical_name, backend_name, property list, and backend_options canonical JSON.
5. Serialize as canonical JSON with sorted keys.
6. Return sha256_hex of the canonical bytes plus backend name, backend version, driver version, and schema profile checksum.
```

Errors:

| Error | Required behavior |
| --- | --- |
| `GRAPH_SCHEMA_NOT_READY` | Return before graph mutation when required index/constraint/schema object is missing or not ready. |
| `GRAPH_SCHEMA_FINGERPRINT_MISMATCH` | Return before mutation when actual schema fingerprint differs from expected. |
| `GRAPH_BACKEND_VERSION_UNSUPPORTED` | Return when backend or driver version does not match profile. |
| `GRAPH_BACKEND_FEATURE_UNAVAILABLE` | Return when license/edition/storage/topology lacks required feature. |

### Deterministic apply algorithm

```text
ApplyGraphDelta(graph_delta_set, apply_profile, expected_schema_fingerprint):
1. Verify graph_delta_set.status = validated.
2. Verify graph_delta_set.ordered_delta_refs are sorted exactly by Cadastre ordering.
3. Run PreflightGraphBackend.
4. Reject if backend schema fingerprint mismatches expected_schema_fingerprint.
5. Split ordered_delta_refs into batches using apply_profile.batch_size and dependency boundaries.
6. For each batch:
   a. Start backend transaction using declared transaction_kind.
   b. Apply mutations in ordered_delta_refs order.
   c. Convert backend errors through the apply_profile error map.
   d. Commit only if every mutation in the batch succeeds or is an explicit idempotent no-op.
   e. Record committed batch checksum, transaction evidence, writer identity, and verification query result.
7. Run read-after-write verification for changed node and edge selectors.
8. Run GraphIndexConsistencyCheck if apply_profile.requires_post_apply_check = true.
9. Persist GraphApplyResult before marking DerivedViewState current.
```

Default bounds:

| Bound | Default | Required profile override behavior |
| --- | ---: | --- |
| `max_batch_deltas` | `1000` | May lower per backend; may raise only after performance and transaction-size validation. |
| `max_apply_transaction_seconds` | `30` | May raise to 300 maximum only with explicit profile row. |
| `max_apply_retries` | `3` | Retry only closed retryable error classes. |
| `max_read_after_write_verification_seconds` | `30` | Must fail apply if verification cannot complete. |

### Deterministic query algorithm

```text
QueryGraph(request):
1. Validate request bounds: depth 1..6, page_size 1..1000, timeout <= 300 seconds.
2. Resolve DerivedViewState and lag policy.
3. If lag_state = stale_forbidden, return DERIVED_VIEW_LAG_ERROR with no graph nodes/edges.
4. Translate request to backend query using backend query translation profile.
5. Apply backend timeout and Cadastre wall-clock timeout.
6. Materialize candidate nodes, edges, and paths up to backend_candidate_limit.
7. Apply authorization and graph-property redaction.
8. Sort using Cadastre order, not backend natural order.
9. Page using Cadastre cursor token.
10. Return derived_view_state, stale flags, graph apply/rebuild refs, and evidence refs when requested.
```

## PRD improvement recommendations

| recommendation_id | contract_area | current_gap | source_basis | proposed_contract | default | bounds | errors | acceptance_criteria | transfer_stance |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `GBR-001` | `GraphReadModelSchemaProfile` | Backend schema fingerprinting is named but not backend-specific enough. | Neo4j/Memgraph schema commands, JanusGraph backend-dependent schema, Arango graph/index definitions. | Add `backend_schema_fingerprint_row` with backend name, backend version, driver version, schema object inventory, readiness states, feature/edition requirements, and canonical hash algorithm. | Fingerprint required before production apply. | One active fingerprint row per backend/profile/version. | `GRAPH_SCHEMA_FINGERPRINT_MISMATCH`, `GRAPH_BACKEND_FEATURE_UNAVAILABLE`. | Applying with mismatched fingerprint fails before mutation for all four backends. | `adopt` |
| `GBR-002` | `GraphApplyProfile` | Apply transaction semantics differ by topology/storage mode. | Neo4j clusters, Memgraph storage modes, JanusGraph backend ACID limits, Arango cluster transaction limits. | Add `transaction_semantics_row` with `transaction_kind`, `topology`, `storage_mode`, `all_or_nothing_guarantee`, `partial_apply_behavior`, `committed_batch_evidence_required`. | `partial_apply_behavior = reject`. | Batch max 1000 deltas unless profile overrides after validation. | `GRAPH_PARTIAL_APPLY_UNSUPPORTED`, `GRAPH_TRANSACTION_BOUNDARY_ERROR`. | Fault injection either commits no ambiguous batch or records committed batch checksums. | `adopt` |
| `GBR-003` | `GraphApplyResult` | Backend evidence needed to prove what happened is not exhaustive. | Driver bookmarks, replication modes, index statuses, transaction limits. | Persist `backend_evidence_rows` with transaction/bookmark/topology/storage/index/replication/write-member/read-after-write data. | Required for every production apply. | Evidence row count >= committed batch count. | `GRAPH_APPLY_EVIDENCE_MISSING`. | Replay can prove whether every batch committed, failed, or was not attempted. | `adopt` |
| `GBR-004` | `GraphProjectionProfile` | Backend-generated IDs remain a lock-in hazard. | Neo4j `elementId` warning, Arango `_key`, JanusGraph vertex IDs. | Add `backend_generated_id_policy = forbidden` and require deterministic `node_id` and `edge_id` formulas in every node/edge projection row. | `forbidden`. | No exceptions in MVP. | `GRAPH_BACKEND_ID_FORBIDDEN`. | Any projection row using backend-generated ID fails validation. | `adopt` |
| `GBR-005` | `GraphEdgeSemantics` | Query traversability and path eligibility need backend-independent contract rows. | Neo4j/Memgraph relationship types, JanusGraph edge labels/vertex-centric indexes, Arango directions. | Add `traversal_eligibility`, `reverse_traversal_eligibility`, `pathfinding_role`, and `non_implication_rule` to each edge semantics row. | Non-traversable unless row permits traversal. | One row per edge type. | `GRAPH_EDGE_SEMANTICS_ERROR`. | Path query cannot use an edge type lacking traversal eligibility. | `adapt` |
| `GBR-006` | `GraphTaxonomyTranslationPolicy` | Query-language and backend label/type/kind translations are not exhaustive by backend. | Cypher labels/types, Gremlin labels, Arango collections/type fields. | Add exhaustive `backend_taxonomy_mapping_rows` from Cadastre node/edge types to backend labels, edge labels, relationship types, collections, and properties. | Reject unmapped taxonomy labels. | One complete table per active backend. | `GRAPH_TAXONOMY_TRANSLATION_ERROR`. | Unknown backend label or edge type cannot appear in served output. | `adopt` |
| `GBR-007` | `GraphRebuildManifest` | Import tools can be mistaken for rebuild proof. | Neo4j import, Memgraph LOAD CSV, JanusGraph batch load, Arango import/dump. | Add `import_mechanism_row` with canonical input file checksums, row order, backend import command version, dry-run evidence, post-import schema and graph checksum. | Backend import success is validation evidence only. | Rebuild checksum required for all production rebuilds. | `GRAPH_REBUILD_VALIDATION_ERROR`. | Rebuild cannot become current until output checksum and index checks pass. | `adapt` |
| `GBR-008` | `GraphIndexConsistencyCheck` | Index freshness and search-index staleness differ. | JanusGraph mixed indexes, Arango inverted/search eventual consistency, Neo4j/Memgraph index readiness. | Add `index_consistency_mode` values: `immediate`, `eventual_requires_refresh`, `eventual_forbidden_for_authority`, `not_applicable`. | Authority-sensitive queries require immediate or verified refreshed indexes. | Every required index has one mode. | `GRAPH_INDEX_CONSISTENCY_ERROR`. | Stale mixed/search index prevents current serving. | `adopt` |
| `GBR-009` | `DerivedViewLagPolicy` | Backend read-after-write lag and async replication require query response state. | Neo4j causal consistency, Memgraph async replication, JanusGraph eventual indexes, Arango search/index freshness. | Extend `DerivedViewState` with backend apply result, backend consistency evidence, index freshness evidence, and serving replica/member identity. | Stale graph-derived compliance/audit output forbidden. | Max lag remains 300s default, 0..86400. | `DERIVED_VIEW_LAG_ERROR`. | Query response identifies served dataset, graph apply/rebuild, and stale-read state. | `adopt` |
| `GBR-010` | `ValidationMatrix` | Backend-specific rejection tests are not enumerated. | All four backend failure modes. | Add backend fixture classes: missing schema, stale fingerprint, duplicate selectors, partial apply, failover, stale index, nondeterministic path tie, raw-bypass, storage-mode rejection. | All active backend fixture classes required. | One positive and one negative case per enabled backend feature. | `VALIDATION_MATRIX_INCOMPLETE`. | Backend profile cannot become active until every fixture passes. | `adopt` |
| `GBR-011` | `VersionManifest` | Backend version, driver, storage mode, schema fingerprint, and graph state refs must be complete. | All backend docs and PRD manifest boundary. | Add `graph_backend_version_refs`, `graph_driver_version_refs`, `graph_backend_topology_refs`, `graph_backend_schema_fingerprints`, `graph_storage_mode_refs`, and `graph_index_freshness_refs`. | Required when graph serving enabled. | Empty arrays only when graph serving disabled. | `VERSION_MANIFEST_MISMATCH`. | Replay rejects if any backend-affecting value differs. | `adopt` |
| `GBR-012` | `GraphReadModelDriftCheck` | Drift checks may tempt unauthorized repair. | Backend drift/index/graph integrity hazards. | Add `allowed_outputs = operational_health_only` and `repair_behavior = forbidden` unless future contract defines repair via graph deltas. | Drift check cannot mutate graph. | No production repair in MVP. | `GRAPH_DRIFT_REPAIR_FORBIDDEN`. | Injected drift emits health record but no graph mutation. | `adopt` |

## Concepts Cadastre must not transfer

| External concept | Required Cadastre rule | Rationale |
| --- | --- | --- |
| Backend graph database state | Must not become canonical truth. | Only lakehouse-backed raw/silver/gold and persisted graph deltas are authoritative. |
| Backend internal node or edge IDs | Must not become Cadastre canonical IDs. | Internal IDs are backend-local and may be unstable, reused, or unsafe for replay. |
| Backend query language semantics | Must not define Cadastre graph API semantics unless explicitly mapped. | Cypher, Gremlin, and AQL have different path, traversal, ordering, and pagination behavior. |
| Backend pathfinding output | Must not become risk scoring or fact truth. | Pathfinding is analysis over projected state, not evidence. |
| Backend constraints | Must not replace Cadastre identity decisions or source authority. | Constraints enforce graph serving shape only. |
| Backend stale-delete, TTL, or cleanup behavior | Must not authorize fact retraction, graph expiry, source absence, or cleanup by itself. | Expiry and retraction require Cadastre completeness, authority, and projection rules. |
| Backend full-text/search/vector results | Must not become identity, graph edge, or fact evidence by themselves. | Search/vector similarity is retrieval context, not authoritative evidence. |
| Backend import success | Must not prove source completeness or rebuild correctness. | Rebuild correctness requires manifest, checksums, counts, and index consistency. |
| Backend HA replication success | Must not prove lakehouse replay eligibility. | HA replicates serving state; it does not preserve authoritative input state. |
| Multi-model document storage | Must not blur raw, silver, gold, and projection boundaries. | Arango-style convenience can collapse evidence layers if not prohibited. |
| Gremlin/Cypher/AQL query output | Must not become source evidence unless a Cadastre derivation rule explicitly consumes persisted Cadastre facts. | Query output is derived read-model state. |

## Gaps, risks, stale assumptions, and unresolved questions

| ID | Scope | Gap/risk/question | Required resolution |
| --- | --- | --- | --- |
| `GAP-001` | All | No backend was cloned, built, or run. | Run local or controlled-cloud proof-of-concept harness. |
| `GAP-002` | All | Driver versions and API behavior were not pinned. | Pin driver versions and run retry/bookmark/transaction/cancellation tests. |
| `GAP-003` | All | Production graph size, edge skew, and high-degree node distribution are unknown. | Build representative Cadastre graph workload and benchmark. |
| `GAP-004` | Neo4j | Cypher 25 versus older Cypher behavior may affect query portability. | Run query compatibility suite and pin dialect. |
| `GAP-005` | Neo4j | Enterprise feature boundaries for constraints/RBAC/import may affect feasibility. | Verify license/edition feature matrix. |
| `GAP-006` | Memgraph | Neo4j compatibility claims may not cover every Cadastre query. | Run exact Cadastre query set. |
| `GAP-007` | Memgraph | Storage mode changes durability, HA, and apply behavior. | Validate all enabled modes; reject unvalidated modes. |
| `GAP-008` | JanusGraph | Backend consistency and index freshness are deployment-specific. | Validate storage/index backend matrix. |
| `GAP-009` | JanusGraph | Gremlin translation can change query semantics. | Create exhaustive translation table and golden corpus. |
| `GAP-010` | ArangoDB | Raw collection writes can bypass named graph integrity. | Enforce service-account permissions and drift check. |
| `GAP-011` | ArangoDB | Multi-shard cluster transactions may be non-atomic. | Validate topology and reject unsafe apply profiles. |
| `GAP-012` | All | Security/authorization was inspected only at overview level. | Perform redaction, permission, RBAC, and audit-log review. |
| `GAP-013` | All | Backup/restore was not runtime-tested. | Restore from backup and compare graph checksum to manifest. |
| `GAP-014` | All | Performance tuning and query planning were not benchmarked. | Capture explain/profile plans and latency percentiles. |

## Backend-specific validation matrix

| Test ID | Backend | Fixture | Expected result |
| --- | --- | --- | --- |
| `VAL-NEO-001` | Neo4j | Missing node uniqueness constraint. | `GRAPH_SCHEMA_NOT_READY`; no mutation. |
| `VAL-NEO-002` | Neo4j | Duplicate `edge_id` relationship under uniqueness constraint. | Non-retryable `GRAPH_CONSTRAINT_VIOLATION`; delta refs recorded. |
| `VAL-NEO-003` | Neo4j | Writer failover during apply. | Commit exactly once with bookmark/writer evidence or fail without ambiguous batch. |
| `VAL-NEO-004` | Neo4j | Equal path tie. | Response path order matches Cadastre order, not backend natural order. |
| `VAL-NEO-005` | Neo4j | Offline import rebuild. | `GraphRebuildManifest.output_checksum` stable across two rebuilds. |
| `VAL-MEM-001` | Memgraph | `IN_MEMORY_ANALYTICAL` production apply. | Rejected unless shadow-only profile explicitly permits. |
| `VAL-MEM-002` | Memgraph | On-disk transaction exceeds memory limit. | Fails before mutation or records committed batch boundary. |
| `VAL-MEM-003` | Memgraph | Missing edge index required by profile. | `GRAPH_SCHEMA_NOT_READY`. |
| `VAL-MEM-004` | Memgraph | Async replication lag after apply. | `DerivedViewState` marks stale or within policy with lag evidence. |
| `VAL-MEM-005` | Memgraph | Stream attempts graph mutation outside delta path. | `GRAPH_DELTA_REQUIRED` or equivalent rejection. |
| `VAL-JAN-001` | JanusGraph | Cassandra-backed duplicate node selector under concurrency. | Exactly one vertex or profile rejected as unsafe. |
| `VAL-JAN-002` | JanusGraph | Mixed index stale after write. | Serving rejected or marked stale per policy. |
| `VAL-JAN-003` | JanusGraph | High-degree node path expansion without vertex-centric index. | Schema/profile validation fails. |
| `VAL-JAN-004` | JanusGraph | Gremlin query translation for every `QueryGraph` scenario. | Returned set and order match golden corpus. |
| `VAL-JAN-005` | JanusGraph | Batch load rebuild. | Post-load checksum and index checks pass before serve. |
| `VAL-ARA-001` | ArangoDB | Raw collection edge insert bypasses named graph. | Drift check detects inconsistency; graph cannot be marked current. |
| `VAL-ARA-002` | ArangoDB | Multi-shard transaction failure. | Apply rejected before mutation unless committed-batch evidence is profile-approved. |
| `VAL-ARA-003` | ArangoDB | Equal shortest paths. | Cadastre response sort is deterministic despite backend tie behavior. |
| `VAL-ARA-004` | ArangoDB | Inverted/search index stale. | Query rejected or labeled stale according to `DerivedViewLagPolicy`. |
| `VAL-ARA-005` | ArangoDB | `arangoimport` rebuild. | Deterministic `_key` values, expected counts, graph integrity, and checksum pass. |
| `VAL-ALL-001` | All | Backend-generated ID appears in graph delta. | `GRAPH_BACKEND_ID_FORBIDDEN`. |
| `VAL-ALL-002` | All | Missing backend schema fingerprint in manifest. | `VERSION_MANIFEST_MISMATCH`. |
| `VAL-ALL-003` | All | Query exceeds 300-second timeout request. | `VALIDATION_ERROR`; no backend query executed. |
| `VAL-ALL-004` | All | Unauthorized raw-derived graph property. | Redacted or rejected according to `GraphPropertyEvidencePolicy`. |
| `VAL-ALL-005` | All | Graph drift check attempts repair. | `GRAPH_DRIFT_REPAIR_FORBIDDEN`; operational health only. |

## Definition of done and acceptance criteria

| ID | Acceptance criterion | Status in this report |
| --- | --- | --- |
| `AC-001` | Each backend is analyzed independently before cross-backend comparison. | Passed. |
| `AC-002` | Every backend section includes source inventory, inspected scope, uninspected scope, and freshness risk. | Passed. |
| `AC-003` | Every backend section distinguishes confirmed facts from source-grounded and Cadastre applicability inferences. | Passed through evidence tables and inference tables. |
| `AC-004` | The report evaluates graph backend behavior only as replaceable graph serving behavior. | Passed. |
| `AC-005` | The report does not let backend graph state define canonical truth, identity, completeness, temporal semantics, or source authority. | Passed. |
| `AC-006` | Query-language differences across Cypher, Gremlin, and AQL are mapped explicitly. | Passed. |
| `AC-007` | Graph model differences are mapped explicitly. | Passed. |
| `AC-008` | Apply semantics are mapped explicitly to `GraphApplyProfile` and `GraphApplyResult`. | Passed. |
| `AC-009` | Index and constraint requirements are mapped explicitly to `GraphReadModelSchemaProfile` and `GraphIndexConsistencyCheck`. | Passed. |
| `AC-010` | Rebuild/import/export behavior is mapped explicitly to `GraphRebuildManifest`. | Passed. |
| `AC-011` | Operational risks are mapped explicitly to Cadastre health, lag, replay, and validation contracts. | Passed. |
| `AC-012` | Concepts that must not transfer are listed with rationale. | Passed. |
| `AC-013` | PRD improvement recommendations include concrete contract text, defaults, bounds, errors, and binary acceptance criteria. | Passed. |
| `AC-014` | Every matrix covers all four required backends. | Passed. |
| `AC-015` | The report states where local clone, build, runtime test, benchmark, or proof-of-concept validation is required. | Passed. |
| `AC-016` | The report does not choose a single winning backend unless separately requested. | Passed. |

## Sources

[^1]: `PRD-Cadastre.md`, Section 1, lines 35-45. The PRD states that graph serving is a query-optimized relationship read model, the lakehouse is the system of record, graph and CIM outputs are replaceable projections, and graph rebuild/lag/replay/maintenance are explicit Cadastre correctness concerns.
[^2]: `nlspec-spec.md`, lines 40-72 and 116-120. The NLSpec standard defines behavioral completeness, precise interfaces, explicit defaults and boundaries, mapping tables, binary acceptance criteria, and the recreatability test.
[^3]: `PRD-Cadastre.md`, Section 2 contract table, lines 88-101 and lines 133-142. The PRD defines `GraphProjectionProfile`, `GraphReadModelSchemaProfile`, `GraphApplyProfile`, `GraphApplyResult`, `GraphEdgeSemantics`, `GraphTaxonomyTranslationPolicy`, `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy`.
[^4]: `PRD-Cadastre.md`, Section 8.4, lines 1017-1069. The PRD defines graph query capabilities, depth/page/timeout bounds, deterministic path ordering, and derived-view state behavior.
[^5]: `PRD-Cadastre.md`, Sections 10.13.11 through 10.13.13, lines 2393-2491. The PRD defines `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy`/`DerivedViewState`.
[^8]: Neo4j release notes and docs inspected 2026-05-16. Web results identified Neo4j `2026.04.0` as current release context; exact repository commit not pinned.
[^9]: Neo4j repository README, `https://github.com/neo4j/neo4j`, inspected 2026-05-16. Web open lines 251-266 described Neo4j as a graph database with Cypher, ACID transactions, and nodes/relationships.
[^10]: Neo4j Cypher Manual, constraints page, inspected 2026-05-16. Web open lines 315-335 described node and relationship uniqueness, existence, type, and key constraints, including edition-specific notes.
[^11]: Neo4j CDC docs, inspected 2026-05-16. Web open lines 177-178 warned that internal `elementId` is not safe outside a single transaction and advised logical/business keys via key constraints.
[^12]: Neo4j Cypher Manual introduction, inspected 2026-05-16. Web open lines 314-320 described Cypher as Neo4j’s query language and the Cypher 25 versus Cypher 5 behavior in the 2025/2026 line.
[^13]: Neo4j Driver Manual, transactions/session behavior, inspected 2026-05-16. Web open lines 181-191 described sessions, causal consistency, and session thread-safety limits.
[^14]: Neo4j clustering docs, inspected 2026-05-16. Web open lines 479-502 and 541-558 described primary/secondary topology, write fault tolerance, and asynchronous replication to secondaries.
[^15]: Neo4j import docs, inspected 2026-05-16. Web open lines 431-458, 886-890, 1059-1065, 1527-1546, and 2045-2071 described `neo4j-admin database import`, CSV/Parquet, full/incremental import, dry run, staged import, and schema files.
[^16]: Neo4j Operations Manual, monitoring page, inspected 2026-05-16. Web search result described metrics, executing-query management, logs, security event logs, query logs, and cluster monitoring.
[^17]: Neo4j Operations Manual, authentication/authorization and RBAC pages, inspected 2026-05-16. Web search results described system-graph security model, RBAC, fine-grained privileges, LDAP, OIDC/SSO, and Enterprise feature context.
[^18]: Memgraph repository README and release notes, `https://github.com/memgraph/memgraph`, inspected 2026-05-16. Web open lines 383-401 described Memgraph as high-performance, Cypher-compatible, ACID-compliant, highly available, and algorithm/module/import capable; release notes exposed v3.10.1 dated 2026-05-15.
[^19]: Memgraph data durability / storage mode docs, inspected 2026-05-16. Web open lines 543-550 described `IN_MEMORY_ANALYTICAL`, `IN_MEMORY_TRANSACTIONAL`, and `ON_DISK_TRANSACTIONAL`, including WAL/snapshot differences.
[^20]: Memgraph storage access and replication docs, inspected 2026-05-16. Web open lines 434-449 described storage accessors and transactional views; web open lines 444-476 described main/replica replication and `SYNC`, `STRICT_SYNC`, and `ASYNC` modes.
[^21]: Memgraph storage memory / on-disk docs, inspected 2026-05-16. Web open lines 549-555 described on-disk indexes/constraints in RocksDB, transaction memory requirement, and lack of replication/HA for on-disk storage in inspected docs.
[^22]: Memgraph constraints and indexes docs, inspected 2026-05-16. Web open lines 441-494 described existence and uniqueness constraints; web open lines 498-705 described label, label-property, composite, edge-type, and edge-property indexes plus creation behavior.
[^23]: Memgraph import docs, inspected 2026-05-16. Web open lines 593-601 described analytical mode for faster import, concurrent `LOAD CSV`, compressed inputs, and edge import mode.
[^24]: Memgraph monitoring docs, inspected 2026-05-16. Web search result described Enterprise HTTP metrics for HA, transactions, query latency, snapshot recovery, triggers, Bolt messages, indexes, streams, memory, operators, and sessions.
[^25]: Memgraph authentication and authorization docs, inspected 2026-05-16. Web search results described users, roles, fine-grained access control, external auth integrations, Enterprise SSO/LDAP/OIDC, and Community auth limitations.
[^26]: JanusGraph releases and changelog, inspected 2026-05-16. Web open lines 219-231 and changelog lines 383-391 identified JanusGraph 1.1.0 release context; exact repository commit not pinned.
[^27]: JanusGraph repository README, `https://github.com/JanusGraph/janusgraph`, inspected 2026-05-16. Web open lines 408-419 described a scalable graph database optimized for billions of vertices/edges, transactional use, concurrent users, complex traversals, and analytic graph queries.
[^28]: JanusGraph data model docs, inspected 2026-05-16. Web open lines 96-113 described adjacency-list storage, edge duplication, and vertex-centric indexing implications.
[^29]: Apache TinkerPop / Gremlin and JanusGraph basic usage docs, inspected 2026-05-16. JanusGraph basic docs showed Gremlin usage; TinkerPop sources described Gremlin as graph traversal/data-flow language and OLTP/OLAP traversal context.
[^30]: JanusGraph transaction docs, inspected 2026-05-16. Web open lines 106-110 and 114-149 described transaction behavior and backend-dependent ACID limits.
[^31]: JanusGraph eventual consistency docs, inspected 2026-05-16. Web open lines 101-126 described Cassandra/HBase consistency features, lack of transaction isolation, locking not enabled by default, and locking failure risks.
[^32]: JanusGraph storage and index backend docs, inspected 2026-05-16. Web open lines 86-92 described pluggable storage backends; web open lines 88-91 described composite indexes through storage backend and mixed indexes via Elasticsearch, Solr, or Lucene.
[^33]: JanusGraph indexing performance docs, inspected 2026-05-16. Web open lines 292-309 described vertex-centric indexes, reindex requirements, and high-degree traversal risks.
[^34]: JanusGraph batch-loading docs from older official documentation, inspected 2026-05-16. Web open lines 31-48 described batch loading that disables consistency checks/locking and requires user-managed consistency. This source is stale relative to JanusGraph 1.1.0 and must be revalidated before PRD amendment.
[^35]: JanusGraph Server and configuration docs, inspected 2026-05-16. Web search results described JanusGraph Server remote Gremlin execution and configuration as defining storage/index backend components and operational aspects.
[^36]: ArangoDB release/docs context, inspected 2026-05-16. Web sources exposed ArangoDB 3.12.x docs and 3.12.9 release-note context; exact repository commit not pinned.
[^37]: ArangoDB repository README, `https://github.com/arangodb/arangodb`, inspected 2026-05-16. Web open lines 381-407 described scalable graph data modeling, native graphs, integrated search, JSON documents, and AQL.
[^38]: ArangoDB graph docs, inspected 2026-05-16. Web open lines 66-89 described named graphs, graph integrity guarantees through graph interfaces, and raw collection bypass risk.
[^39]: ArangoDB AQL traversal docs, inspected 2026-05-16. Web open lines 49-90 and 207-230 described traversal syntax, direction, bounds, `PRUNE`, options, and nondeterministic equal-cost weighted traversal behavior.
[^40]: ArangoDB shortest path docs, inspected 2026-05-16. Web open lines 19-25 and 54-66 described shortest path syntax/direction and random tie behavior among equally shortest paths.
[^41]: ArangoDB transaction docs, inspected 2026-05-16. Web open lines 44-51 and 68-78 described single-server ACID behavior, cluster single-document ACID, and limitations for multi-document/multi-shard transactions.
[^42]: ArangoDB stream transaction docs, inspected 2026-05-16. Web open lines 23-54 described start/commit/abort, declaring collections, short-lived transactions, timeout/size limits, and no concurrent requests inside one stream transaction.
[^43]: ArangoDB index and ArangoSearch docs, inspected 2026-05-16. Web open lines 22-44 described automatic `_key` and edge `_from`/`_to` indexes and index families; web open lines 475-490 described regular indexes as immediately consistent and inverted/search indexes as eventually consistent.
[^44]: ArangoDB 3.12 feature notes, inspected 2026-05-16. Web search result described sparse vector indexes and vector index improvements in 3.12.6.
[^45]: ArangoDB import and backup docs, inspected 2026-05-16. Web open lines 18-24 described `arangoimport`; backup docs lines 17-32 and 43-55 described logical dumps/restores and hot backup behavior.
[^46]: ArangoDB sharding and SmartGraph docs, inspected 2026-05-16. Web open lines 18-37 described sharding across DB-Servers; SmartGraph docs lines 17-33 described value-based sharding and same query surface.
[^47]: ArangoDB server options docs, inspected 2026-05-16. Web search result described RocksDB encryption-at-rest options and server configuration options.
