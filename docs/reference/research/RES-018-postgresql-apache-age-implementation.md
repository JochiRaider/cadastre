---
doc_id: RES-018
project: Cadastre
title: PostgreSQL and Apache AGE Implementation Refinement Research Report
status: research-report
inspection_date: 2026-05-22
---

## 1. Executive verdict

[Cadastre applicability inference] Cadastre can replace the omitted-backend MVP default `mvp-janusgraph.v1` with a PostgreSQL relational graph profile only if the later NLSpec patch makes the PostgreSQL profile an activation-controlled backend profile with complete schema, query translation, apply, package, provider capability, preflight, validation, observability, and error closure. The safest implementation posture is:

| Candidate profile | Recommended status | Reason |
| --- | --- | --- |
| `mvp-postgresql-relational-graph.v1` | Default after gates pass | PostgreSQL can enforce application-owned node and edge IDs, endpoint integrity, edge-type closure, temporal fields, evidence lookup, transaction boundaries, read-only query roles, query timeouts, backup/restore, and schema fingerprinting through relational contracts. It remains validation-dependent for traversal performance, high-degree behavior, query-plan regressions, restore rehearsals, and exact Cadastre query parity. |
| `mvp-postgresql-age.v1` | Validation-only optional profile | Apache AGE adds graph labels, vertices, directed edges, property maps, and Cypher-through-SQL execution, but managed-provider support is uneven, extension lifecycle and upgrade behavior add operational risk, graph-native uniqueness and endpoint constraints are not established from inspected primary sources, and SQL/Cypher mutation and namespace bypass controls require runtime validation. |
| `mvp-janusgraph.v1` | Remove as omitted default; retain as explicit non-default profile until retirement is separately accepted | Current `090` defaulting behavior names JanusGraph as the omitted-backend selection token, but the same spec says selection does not authorize mutation/query/rebuild/promotion and remains blocked until backend, package, storage, index, schema, and validation gates close.[^1] |

[Cadastre applicability inference] The default replacement must be PostgreSQL relational, not AGE. The relational profile directly matches Cadastre’s MVP read-model contract: the active edge set is exactly `observed_connection`, theoretical reachability is prohibited, backend-generated IDs are forbidden, and Cadastre-owned ordering, candidate limits, authorization, redaction, page-token identity, and error selection happen after backend candidate materialization.[^2]

[operational risk] Apache AGE is not rejected. It remains a valid experimental profile for validating graph-extension ergonomics and Cypher query authoring. It must not become the default until provider availability, extension versioning, graph namespace security, direct SQL bypass prevention, query semantics, indexing, restore, upgrade, and benchmark gates pass with exact PostgreSQL and AGE versions.

[open question] This report does not prove production performance, operational safety, tenant-specific cloud availability, extension upgrade safety, RLS behavior over AGE-created graph tables, or query-plan adequacy. No PostgreSQL database was created, no AGE extension was installed, no repository was cloned, and no benchmark or failover test was executed.

## 2. Source inventory and inspection limits

The uploaded `postgresql-apache-age-research-report.md` was used only as Cadastre prior context and an open-question checklist. It states that no repository was cloned, built, installed, executed, benchmarked, or tested, and it labels all production behavior as subject to runtime validation.[^3]

### Evidence label contract

| Label | Meaning in this report |
| --- | --- |
| `confirmed primary-source fact` | Directly supported by inspected official PostgreSQL docs, PostgreSQL source-facing docs, Apache AGE docs, Apache AGE repository or release metadata, or cloud-provider official docs. |
| `source-grounded inference` | Derived from confirmed primary-source facts with reasoning stated. |
| `Cadastre applicability inference` | Derived by comparing primary-source facts to Cadastre graph, package, API, validation, observability, and authority contracts. |
| `operational risk` | A deployment, reliability, upgrade, security, performance, supportability, or managed-service risk. |
| `open question` | Cannot be resolved from inspected primary sources alone. |
| `requires runtime validation` | Must be tested in a real environment before Cadastre can rely on the claim. |

### External primary source inventory

| Source family | Source URL | Inspection date | Material inspected | Limits |
| --- | --- | --- | --- | --- |
| PostgreSQL recursive queries | https://www.postgresql.org/docs/current/queries-with.html | 2026-05-22 | Recursive CTE execution model, search order, cycle detection, materialization notes. | Did not run queries. |
| PostgreSQL constraints and generated columns | https://www.postgresql.org/docs/current/ddl-constraints.html and https://www.postgresql.org/docs/current/sql-createtable.html | 2026-05-22 | Primary keys, uniqueness, foreign keys, checks, exclusion constraints, generated columns. | Did not execute DDL. |
| PostgreSQL indexes and JSONB | https://www.postgresql.org/docs/current/indexes.html and https://www.postgresql.org/docs/current/datatype-json.html | 2026-05-22 | B-tree, GIN, partial/expression indexes, JSON/JSONB behavior, JSONB GIN operator classes. | Did not inspect every operator class or benchmark indexes. |
| PostgreSQL transactions, locks, RLS, timeouts, backup, EXPLAIN, extensions | PostgreSQL current documentation pages | 2026-05-22 | Isolation, explicit/advisory locks, RLS, statement timeout, backup/PITR, extension packaging, `CREATE EXTENSION`, `ALTER EXTENSION`, `EXPLAIN`. | Did not inspect every operational chapter. |
| Apache AGE setup, graph model, Cypher, SQL/Cypher composition | https://age.apache.org/age-manual/master/ | 2026-05-22 | Build/install, `CREATE EXTENSION`, `LOAD`, `search_path`, graph namespaces, labels, vertices, edges, `ag_catalog.cypher()`, JOIN/CTE constraints. | Documentation version freshness is inconsistent across pages. |
| Apache AGE repository and releases | https://github.com/apache/age and https://github.com/apache/age/releases | 2026-05-22 | README compatibility statement, feature list, releases for PostgreSQL 17 and 18, upgrade warnings for AGE releases. | Release assets not installed; source not cloned. |
| Azure Database for PostgreSQL Flexible Server | Microsoft Learn PostgreSQL extension pages | 2026-05-22 | AGE preview status, extension-version matrix, required configuration, in-place major-version-upgrade limitation. | No Azure tenant tested. |
| AWS RDS and Aurora PostgreSQL | AWS official extension-version pages | 2026-05-22 | Supported-extension list behavior, extension upgrade behavior, absence of `age` in inspected extension lists. | No AWS account or private-preview check. |
| Google Cloud SQL for PostgreSQL | Google Cloud official extension page | 2026-05-22 | Supported-extension list behavior, supported-extension installation role, absence of `age` in inspected extension list. | No GCP project or private-preview check. |

### Source limits

[confirmed primary-source fact] PostgreSQL documentation establishes a mature relational database surface, but this report did not prove any Cadastre-specific schema, query, migration, backup, restore, RLS, or connection-pool behavior by execution. Apache AGE documentation and releases establish extension capabilities and risks, but this report did not prove actual AGE behavior under Cadastre workloads.

[open question] Commercial support for AGE-specific production incidents, tenant-specific provider availability, regional availability, provider support response scope, and private-preview availability were not established from inspected primary sources.

## 3. Cadastre baseline and non-transferable boundaries

[confirmed primary-source fact] Cadastre `090` owns graph projection, backend profiles, graph query, graph apply, rebuild, lag, and drift without making the backend authoritative. It exports `GraphBackendProfile`, `GraphProviderCapabilityMatrix`, `GraphBackendPreflightResult`, `BackendSchemaFingerprint`, `GraphBackendTaxonomyMappingProfile`, `GraphQueryTranslationProfile`, `GraphReadModelSchemaProfile`, `GraphApplyProfile`, `GraphApplyResult`, `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, `DerivedViewLagPolicy`, and `DerivedViewState`.[^4]

[confirmed primary-source fact] Cadastre graph state is a derived read model. A graph backend must not create authoritative facts, infer identity, decide source completeness, decide fact retraction, define bitemporal semantics, repair drift by itself, or expose backend internal IDs as Cadastre IDs.[^5]

[confirmed primary-source fact] The active MVP graph edge set is exactly `observed_connection`; all other edge types, including theoretical reachability, require inactive or deterministic block rows. Missing or ambiguous `FlowRoleEvidence`, unresolved endpoint identity, and OCSF endpoint order cannot emit graph output.[^6]

[confirmed primary-source fact] `100` defines production package runtime activation as an immutable `ProductionPackageSetManifest`; a single artifact, version string, deployment label, mutable ref, scalar signature status, dependency lock, SBOM, provenance record, or validation run is not a production activation target.[^7]

[confirmed primary-source fact] `100.PackageType` is a closed lower-snake-case enum. Existing graph package tokens already include provider package, adapter package, driver package, storage backend adapter, index backend adapter, runtime distribution, and deployment profile tokens.[^8]

[confirmed primary-source fact] `110` sets graph API bounds: `max_depth` defaults to `3` and is bounded `1..6`, graph `page_size` defaults to `100` and is bounded `1..1000`, graph timeout defaults to `30` seconds and is bounded `1..300`, and page-token TTL defaults to `900` seconds and is bounded `60..3600`.[^9]

[confirmed primary-source fact] `120` already requires graph negative tests for backend ID leakage, active MVP edge set closure, missing or ambiguous `FlowRoleEvidence`, OCSF endpoint-order rejection, unresolved endpoint identity, theoretical reachability prohibition, candidate limits, page tokens, missing provider profile fields, unsupported provider capability, unsafe storage mode, stale schema fingerprint, missing package gate, raw-write bypass, backend-generated ID leakage, and provider-specific query bypass.[^10]

[confirmed primary-source fact] `140` makes runtime telemetry diagnostic only and forbids telemetry from creating or modifying authoritative domain records. Its default instrumentation profile forbids raw payload bytes, private source bindings, and backend internal IDs in telemetry; the telemetry redaction policy rejects backend-generated graph IDs and provider-native query text by default.[^11]

## 4. PostgreSQL relational graph projection analysis

### Relational fit for MVP serving

[confirmed primary-source fact] PostgreSQL supports recursive `WITH` queries that can refer to their own output. Its documented recursive-query algorithm evaluates a non-recursive term, then iteratively evaluates the recursive term until no rows remain. PostgreSQL also documents search-order and cycle-detection techniques while warning that result evaluation order is implementation-dependent unless final ordering is explicitly specified.[^12]

[confirmed primary-source fact] PostgreSQL supports primary keys, unique constraints, foreign keys, check constraints, exclusion constraints, generated columns, indexes, JSON/JSONB types, JSONB GIN indexes, row-level security, transaction isolation, locks, advisory locks, backup and PITR, extensions, and query-plan inspection through `EXPLAIN`.[^13]

[Cadastre applicability inference] A pure PostgreSQL relational graph projection can satisfy Cadastre MVP graph serving without Apache AGE when Cadastre limits serving to current-state and recent-history graph queries over the active `observed_connection` edge set and requires every query result to pass Cadastre-owned ordering, authorization, redaction, page-token, state-label, and error selection after backend candidate materialization.

### Recommended relational read-model shape

| Artifact | Required implementation refinement |
| --- | --- |
| `graph_node` | Store only Cadastre application-owned `graph_node_id` as the primary key. Store canonical entity refs and node type as constrained relational columns. Do not store or expose database OIDs, tuple IDs, sequence values, or backend-generated IDs as Cadastre IDs. |
| `graph_edge` | Store only Cadastre application-owned `graph_edge_id` as the primary key. Store `edge_type`, `traversal_class`, `from_graph_node_id`, `to_graph_node_id`, `source_gold_fact_id`, `assertion_state`, `confidence`, valid-time, known-time, and `last_seen` as constrained relational columns. |
| `graph_edge_evidence_ref` | Store one row per edge-evidence ref for deterministic evidence drillback and evidence lookup. Use `(graph_edge_id, evidence_ref_id)` uniqueness. |
| `graph_node_property` or bounded JSONB column | Use only declared graph property paths from `GraphBackendTaxonomyMappingProfile` and `GraphPropertyEvidencePolicy`. For MVP, prefer typed relational columns for query-driving properties and allow JSONB only for bounded non-query or validation-backed properties. |
| `graph_apply_state` | Persist apply transaction, batch, idempotency key, input checksum, schema fingerprint, read-after-write proof, and terminal status. This state is Cadastre runtime evidence, not source truth. |
| `graph_schema_fingerprint_state` | Persist deterministic fingerprint inventory and checksum over tables, columns, constraints, indexes, triggers, RLS policies, functions, extension versions when used, and schema-profile refs. |

### Required relational constraints

| Constraint target | PostgreSQL mechanism | Cadastre effect |
| --- | --- | --- |
| Duplicate node ID | Primary key or unique constraint on `graph_node_id` | Enforces application-owned node identity. |
| Duplicate edge ID | Primary key or unique constraint on `graph_edge_id` | Enforces application-owned edge identity. |
| Endpoint existence | Foreign keys from `graph_edge.from_graph_node_id` and `graph_edge.to_graph_node_id` to `graph_node.graph_node_id` | Prevents orphan edges before apply commit. |
| MVP edge set | Check constraint or reference table that allows only `observed_connection` while MVP profile is active | Prevents accidental activation of non-MVP edges. |
| Traversal class | Check constraint or reference table resolved from `GraphTraversalClass` | Prevents parallel backend-specific traversal eligibility. |
| Assertion state | Check constraint or reference table imported from Cadastre state vocabulary | Prevents backend-local assertion-state tokens. |
| Confidence | Check constraint matching Cadastre decimal confidence bounds | Prevents backend-local confidence semantics. |
| Valid/known intervals | Check constraints for timestamp nullability and interval order | Preserves temporal filter semantics. |
| Evidence refs | Foreign-key-like relational lookup or constrained ref table | Enables deterministic drillback. |
| JSONB properties | JSONB validity plus check constraints, generated columns, expression indexes, and validation queries | Permits bounded properties without making JSON shape implicit. |

[Cadastre applicability inference] PostgreSQL can directly enforce most MVP structural constraints. Cadastre must still enforce source authority, identity, bitemporal interpretation, omission semantics, endpoint identity resolution, graph edge semantics, authorization, redaction, page-token identity, and candidate ordering outside PostgreSQL.

### Required indexes

| Query or operation | Required relational index class |
| --- | --- |
| Node detail by `graph_node_id` | Unique B-tree primary key. |
| Node lookup by canonical entity ref | B-tree index on canonical entity ref and node type. |
| Edge detail by `graph_edge_id` | Unique B-tree primary key. |
| Outbound neighbor expansion | B-tree index on `(from_graph_node_id, edge_type, traversal_class, assertion_state, last_seen DESC, graph_edge_id)`. |
| Inbound neighbor expansion | B-tree index on `(to_graph_node_id, edge_type, traversal_class, assertion_state, last_seen DESC, graph_edge_id)`. |
| Bounded path traversal | Same adjacency indexes, plus partial indexes for active assertion states and allowed traversal classes. |
| Valid-time and known-time filters | Composite or range-aware indexes over endpoint, edge type, and time columns. Exact form requires benchmark validation. |
| Evidence drillback | B-tree index on `source_gold_fact_id` and edge-evidence bridge indexes on `evidence_ref_id`. |
| JSONB property filters | GIN or expression indexes only for declared property paths used by active query profiles. |

### Mapping to Cadastre contracts

| Cadastre contract | PostgreSQL relational refinement |
| --- | --- |
| `GraphBackendProfile` | Add provider `postgresql_relational`, query language `sql`, storage backend `postgresql_table`, index backend `postgresql_indexes`, and required PostgreSQL major/version refs. |
| `GraphReadModelSchemaProfile` | Define exact relational tables, columns, constraints, RLS policy requirements, indexes, generated columns, triggers if used, and forbidden implicit schema creation. |
| `GraphQueryTranslationProfile` | Define SQL templates for `node_detail`, `neighbor_expansion`, `bounded_path`, `evidence_drillback`, and `analysis_read_only`; every template must return candidate rows containing only Cadastre IDs and declared properties. |
| `GraphApplyProfile` | Define transaction mode, batch size, delta ordering, `ON CONFLICT` behavior, idempotency semantics, retry bounds, lock behavior, and partial-apply resume evidence. |
| `GraphApplyResult` | Persist PostgreSQL transaction evidence, batch outcomes, conflict counts, constraint errors, schema fingerprint used, and read-after-write proof. |
| `BackendSchemaFingerprint` | Compute from catalog-visible schema objects sorted by stable owner-defined keys; exclude physical OIDs and volatile statistics. |
| `GraphIndexConsistencyCheck` | Validate required indexes exist, are usable, match expected definitions, and pass fixture queries. It must not treat index success as fact authority. |
| `DerivedViewState` | Reference the `GraphApplyResult`, dataset/snapshot refs, schema fingerprint, backend profile ref, query translation profile ref, and freshness/lag state. |
| `GraphBackendPreflightResult` | Validate connection, PostgreSQL major/version, schema fingerprint, role grants, RLS posture when required, search path, statement timeout setting, package refs, and fixture coverage before query or mutation. |
| `VersionManifest` | Include graph backend profile, PostgreSQL version ref, driver/package refs, schema fingerprint, query translation profile, apply profile, package set, validation rows, and runtime state refs that affect graph output. |

### PostgreSQL-only limitations

| Limitation | Classification | Required handling |
| --- | --- | --- |
| Recursive traversal performance depends on degree distribution, filters, depth, and indexes | requires runtime validation | Benchmark depth `1..6`, high-degree nodes, filter selectivity, concurrent load, and timeout behavior. |
| SQL path queries are more verbose than Cypher | Cadastre applicability inference | Keep SQL hidden behind `GraphQueryTranslationProfile`; do not expose SQL as user-authored graph behavior. |
| JSONB shape constraints are not equivalent to a typed graph schema unless explicitly validated | source-grounded inference | Prefer typed query-driving columns; use validation queries for JSONB paths. |
| RLS and authorization cannot replace Cadastre authorization/redaction | Cadastre applicability inference | Use database privileges as defense-in-depth only; keep final authorization and redaction in Cadastre. |
| Query plans can regress after statistics, data, or version changes | operational risk | Add query-plan regression fixtures and operational health thresholds. |

## 5. Apache AGE backend profile analysis

### Confirmed AGE behavior relevant to Cadastre

[confirmed primary-source fact] Apache AGE is a PostgreSQL extension. Its official setup documentation describes build dependencies, `make install`, `CREATE EXTENSION age`, per-session `LOAD 'age'`, and adding `ag_catalog` to `search_path`; it also documents a non-superuser setup path requiring `GRANT USAGE ON SCHEMA ag_catalog`.[^14]

[confirmed primary-source fact] AGE graph documentation says graphs have vertices and edges with property maps, edges are directed, graphs are created through `ag_catalog.create_graph`, and each graph is associated with a PostgreSQL namespace. AGE creates `_ag_label_vertex` and `_ag_label_edge` parent tables and label-specific tables, tracks labels in `ag_catalog.ag_label`, and recommends that DML and DDL not be executed in the reserved graph namespace.[^15]

[confirmed primary-source fact] AGE Cypher queries are invoked through `ag_catalog.cypher(graph_name, query_string, parameters)`, return PostgreSQL `SETOF` records, require a column definition list for returned rows, and document that the parameters map is only available for prepared statements. AGE’s advanced SQL/Cypher page documents CTE and JOIN composition, but states that `CREATE`, `SET`, and `REMOVE` clauses cannot be used in SQL JOINs because they affect the PostgreSQL transaction system.[^16]

[confirmed primary-source fact] Apache AGE release notes show current PostgreSQL 17 and PostgreSQL 18 release lines as of inspection, but they also show material upgrade caveats: the AGE 1.7.0 PostgreSQL 17 upgrade script from 1.6.0 may take a while for large graphs because it creates indexes; PostgreSQL 18 1.7.0 had no upgrade path because it was a new extension for PostgreSQL 18; AGE 1.6.0 PostgreSQL 15 and 14 notes warn that GIN operator changes require dropping and recreating GIN indexes and recommend backups before upgrade.[^17]

[confirmed primary-source fact] The AGE README states support for PostgreSQL 11 through 18 and describes features including Cypher, hybrid querying, multiple graphs, hierarchical labels, property indexes on vertices and edges, and use of PostgreSQL storage and relational capabilities. That README compatibility statement is broader than the master manual setup page and therefore must be validated against the exact release and provider.[^18]

### AGE suitability by use

| Use class | Status | Required Cadastre behavior |
| --- | --- | --- |
| Candidate materialization only | Validation-only allowed | AGE may return candidate node, edge, and path IDs for Cadastre post-processing. Returned IDs must be application-owned Cadastre IDs, not AGE graph object IDs. |
| Read-only query substrate | Validation-only allowed | Use read-only database role and read-only transaction. Reject or sandbox raw Cypher. Validate that `CREATE`, `SET`, `REMOVE`, `DELETE`, graph creation, and graph drop cannot execute from read-only query paths. |
| Graph mutation substrate | Not production-default; runtime validation required | Only allowed if `ApplyGraphDelta` writes through Cadastre-controlled AGE templates, enforces relational anchor constraints first or in the same PostgreSQL transaction, proves no direct namespace bypass, and persists apply evidence. |
| Relational anchor plus AGE mirror | Preferred AGE experiment | Store Cadastre-owned IDs and constraints in relational tables; mirror only declared labels/edges/properties into AGE. Query candidates through AGE, then rejoin/validate relational anchors before serving. |
| Production backend after gates pass | Possible optional profile | Requires provider support, exact version compatibility, extension install/restore rehearsal, permission bypass validation, query performance benchmarks, upgrade rehearsals, and validation fixtures. |
| Candidate default profile | Not recommended now | AGE does not yet have enough primary-source and runtime evidence to replace relational PostgreSQL as the default. |

### AGE-specific risks

| Risk | Classification | Required validation or contract closure |
| --- | --- | --- |
| Extension lifecycle | operational risk | Preflight must check `age` extension existence/version, `ag_catalog` accessibility, graph existence, provider parameters, and `search_path` behavior. |
| Managed-provider availability | operational risk | Provider row must be exact by provider, region, PostgreSQL major version, AGE version, preview/GA status, and support boundary. |
| Graph-native constraints | open question | AGE docs inspected do not establish graph-native uniqueness, endpoint, or property-shape enforcement equivalent to relational constraints. Use relational anchors or validation queries. |
| Direct SQL namespace bypass | operational risk | Database roles must prevent application and read-only roles from DML/DDL on AGE graph namespaces. Negative fixtures must attempt bypass. |
| Cypher mutation bypass | requires runtime validation | Read-only API role must be unable to call mutating Cypher or graph management functions. |
| Search path hazards | operational risk | Use fully qualified `ag_catalog.cypher` in generated SQL; preflight must reject unsafe `search_path` configurations. |
| Internal graph IDs | Cadastre applicability inference | AGE vertex, edge, graph, label, path, or agtype-internal IDs must not become Cadastre IDs, cursors, selectors, drillback refs, evidence refs, response IDs, or replay keys. |
| Upgrade and rollback | operational risk | Rehearse exact extension upgrade path and rollback or rebuild path before optional production activation. |

## 6. Deployment-provider and extension-support matrix

[confirmed primary-source fact] Azure Flexible Server documents `age` as a preview extension with AGE 1.7.0 for PostgreSQL 18 and 17, AGE 1.6.0 for PostgreSQL 16, AGE 1.5.0 for PostgreSQL 15, 14, and 13, and no support for PostgreSQL 12 or 11. Azure’s AGE how-to requires setting `azure.extensions`, adding AGE to `shared_preload_libraries`, restarting, creating the extension, and setting `search_path`. Azure extension considerations state that in-place major version upgrade does not support Apache AGE.[^19]

[confirmed primary-source fact] AWS RDS and Amazon Aurora PostgreSQL extension documentation state that extension versions do not automatically upgrade during engine upgrades and list supported extensions by PostgreSQL version. The inspected RDS and Aurora pages did not list `age` or Apache AGE. Google Cloud SQL states that users can install only supported extensions and that `cloudsqlsuperuser` can create supported extensions; the inspected Cloud SQL extension page did not list `age` or Apache AGE.[^20]

| Provider or deployment mode | Source URL | Inspection date | Supported PostgreSQL versions | Supported AGE versions | Required flags or preload libraries | Privilege model | Upgrade behavior | Support status | Cadastre viability status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Self-managed PostgreSQL | https://www.postgresql.org/docs/current/ and https://github.com/apache/age | 2026-05-22 | PostgreSQL current docs list supported major versions 18, 17, 16, 15, and 14. AGE README states PostgreSQL 11 through 18 support; release notes must be matched to exact major. | AGE release notes include AGE 1.7.0 for PostgreSQL 17 and 18 and 1.6.0/1.5.0 for other majors. | Build/install AGE with PostgreSQL development files and `pg_config`; `CREATE EXTENSION`, `LOAD`, and `search_path` setup required. | Superuser or equivalent extension-install authority; least-privilege app roles must be separate. | PostgreSQL major upgrade plus AGE upgrade scripts must be rehearsed. | Supported by self-operation only. | Relational PostgreSQL viable. AGE viable only as optional validation or production-after-gates profile. |
| Azure Database for PostgreSQL Flexible Server | https://learn.microsoft.com/en-us/azure/postgresql/extensions/concepts-extensions-versions and https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-use-pg-age | 2026-05-22 | Azure matrix lists AGE support for PostgreSQL 18, 17, 16, 15, 14, and 13; not for 12 or 11. | AGE 1.7.0 for PG18/17, 1.6.0 for PG16, 1.5.0 for PG15/14/13. | `azure.extensions`, `shared_preload_libraries`, restart, `CREATE EXTENSION`, `search_path`. | Azure-managed extension allowlist and database role model; exact app-role grants require runtime validation. | Microsoft docs state in-place major version upgrade does not support Apache AGE. | Preview. | Relational PostgreSQL viable. AGE optional validation-only; production use blocked until preview/support/upgrade risk accepted. |
| AWS RDS for PostgreSQL | https://docs.aws.amazon.com/AmazonRDS/latest/PostgreSQLReleaseNotes/postgresql-extensions.html | 2026-05-22 | AWS extension pages cover RDS PostgreSQL engine versions by provider matrix. | `age` not found in inspected official extension list. | Provider-supported listed extensions only; exact flags not applicable because AGE not listed. | RDS extension model; `SHOW rds.extensions` can show available extensions. | Extension versions do not automatically upgrade during engine upgrade. | Not listed in inspected source. | Relational PostgreSQL viable. AGE not viable without provider exception, private preview, or self-managed alternative. |
| Amazon Aurora PostgreSQL | https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Extensions.html | 2026-05-22 | Aurora extension pages cover Aurora PostgreSQL versions by provider matrix. | `age` not found in inspected official extension list. | Provider-supported listed extensions only; exact flags not applicable because AGE not listed. | Aurora/RDS extension model. | Extension versions do not automatically upgrade during engine upgrades. | Not listed in inspected source. | Relational Aurora viable if Cadastre accepts Aurora-specific operation. AGE not viable without provider exception or private preview. |
| Google Cloud SQL for PostgreSQL | https://cloud.google.com/sql/docs/postgres/extensions | 2026-05-22 | Cloud SQL supports provider-specific PostgreSQL versions and listed extensions. | `age` not found in inspected official extension list. | Only listed supported extensions may be installed. | Users with `cloudsqlsuperuser` can use `CREATE EXTENSION` for supported extensions. | Read replicas inherit primary extension; exact AGE behavior not applicable because AGE is not listed. | Not listed in inspected source. | Relational PostgreSQL viable. AGE unsupported/not listed for Cadastre production. |
| Kubernetes/self-hosted PostgreSQL | PostgreSQL and AGE official docs/source | 2026-05-22 | Same as self-managed, bounded by chosen image and operator. | Same as self-managed, bounded by packaged extension version. | Image must include AGE binaries, control files, SQL scripts, and PostgreSQL-compatible build. | Cluster/operator role model; app, migration, validation, and read-only roles must be separate. | Image upgrade, PostgreSQL major upgrade, AGE upgrade, backup/restore, and failover must be rehearsed. | Self-supported. | Relational viable. AGE optional after package provenance, restore, upgrade, and security gates. |

## 7. Graph schema, identifiers, constraints, and fingerprints

### Identifier strategy

[Cadastre applicability inference] Both PostgreSQL relational and AGE profiles must treat Cadastre application-owned IDs as the only response IDs, selector IDs, cursor identity inputs, evidence refs, replay keys, drillback keys, and graph delta IDs. PostgreSQL sequence values, tuple identifiers, OIDs, AGE graph object IDs, AGE labels, agtype vertices, agtype edges, paths, backend cursors, and provider-native query handles must remain internal and unexposed.

| Identifier | Required source | PostgreSQL relational enforcement | AGE plus relational anchor enforcement |
| --- | --- | --- | --- |
| `graph_node_id` | Cadastre projection algorithm over stable graph delta inputs | Primary key on `graph_node.graph_node_id`; unique constraint over declared canonical-source tuple when needed. | Relational anchor primary key. AGE vertex property may duplicate value only after anchor insert succeeds. |
| `graph_edge_id` | Cadastre projection algorithm over edge type, endpoints, direction, source fact refs, temporal inputs, and projection refs | Primary key on `graph_edge.graph_edge_id`. | Relational anchor primary key. AGE edge property may duplicate value only after anchor insert succeeds. |
| Endpoint refs | Resolved canonical graph node IDs | Foreign keys from edge to node. | Relational anchor FK required; AGE endpoint IDs must be verified against anchor rows. |
| Evidence refs | `040.EvidenceRef` or gold/silver/raw metadata refs | Bridge table with constrained refs and deterministic sort. | Relational evidence bridge remains source of drillback; AGE stores no authoritative evidence. |
| Backend schema fingerprint | Cadastre computed runtime state | Deterministic inventory over relational schema and profile refs. | Inventory must include relational anchors plus AGE extension version, graph namespaces, labels, property indexes, and validation queries. |

### Enforcement comparison

| Requirement | PostgreSQL direct enforcement | Cadastre pre-mutation validation | Requires runtime validation |
| --- | --- | --- | --- |
| Duplicate Cadastre node ID | Yes, primary key or unique constraint. | Also validate delta-set duplicates before DB write. | No, except performance and error mapping. |
| Duplicate Cadastre edge ID | Yes, primary key or unique constraint. | Also validate delta-set duplicates before DB write. | No, except performance and error mapping. |
| Endpoint exists | Yes, foreign keys. | Validate endpoints before apply to emit Cadastre-specific error. | No for relational; yes for AGE mirror consistency. |
| Edge type is `observed_connection` for MVP | Yes, check constraint or ref table. | Validate graph profile and semantics before apply. | No. |
| Property type constraints | Yes for typed columns; partial for JSONB with checks/generated columns. | Required for JSONB and AGE property maps. | Yes for AGE property handling and query semantics. |
| Label and relationship type mapping | Not native for SQL tables; represented as columns/checks/ref tables. | Required through `GraphBackendTaxonomyMappingProfile`. | Yes for AGE label creation and migration. |
| Valid-time and known-time lookup | Yes through timestamp/range columns and indexes. | Required to preserve Cadastre time policy. | Benchmark index form. |
| Assertion-state and confidence filtering | Yes through constrained columns and indexes. | Required to preserve Cadastre state rules. | Benchmark selectivity. |
| Backend internal ID rejection | Yes by absence from public schema and API response. | Required in API, telemetry, query translation, and validation fixtures. | Yes for AGE agtype and provider query results. |

### BackendSchemaFingerprint requirements

`BackendSchemaFingerprint` for `mvp-postgresql-relational-graph.v1` must include at least:

- PostgreSQL major and patch version ref.
- Database schema name or normalized schema ref.
- Table names, column names, data types, nullability, defaults, generated-column expressions, and check constraints.
- Primary key, unique, foreign key, exclusion, and check constraint definitions.
- Index definitions, including expression, partial, covering, GIN, and operator-class choices when used.
- Trigger and function definitions when output-affecting.
- RLS enabled/forced state and policy definitions when tenant or role isolation relies on RLS.
- `GraphReadModelSchemaProfile`, `GraphBackendTaxonomyMappingProfile`, `GraphQueryTranslationProfile`, `GraphApplyProfile`, package-set refs, and validation refs.

`BackendSchemaFingerprint` for `mvp-postgresql-age.v1` must include all relational anchor rows above plus AGE extension version, graph namespace names, graph labels, AGE label table inventory, property-index inventory, graph creation/drop policy, `ag_catalog` grants, `search_path` policy, and direct namespace DML/DDL bypass validation refs.

## 8. Query translation and candidate materialization

[confirmed primary-source fact] PostgreSQL ordering is not guaranteed by recursive evaluation order; Cadastre must impose final ordering outside backend natural order. PostgreSQL `statement_timeout` can abort statements that run longer than the specified time, and a zero value disables that timeout.[^21]

[Cadastre applicability inference] Query translation must materialize backend candidates, then apply Cadastre-owned authorization, redaction, deterministic ordering, page-token creation, response shape, derived-view state labeling, and error selection. Backend cursor values, SQL row order, AGE path object order, and AGE graph IDs must not determine response identity or ordering.

| Cadastre query class | PostgreSQL lookup strategy | AGE lookup strategy | Required indexes | Candidate limit | Timeout behavior | Ordering and page-token inputs | Unsupported behavior | Failure code recommendation | Validation fixture recommendation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `node_detail` | Select by `graph_node_id` or authorized canonical ref from `graph_node`; use candidate limit 2 to detect duplicates when lookup is not primary-key based. | `ag_catalog.cypher()` matches vertex by application-owned `graph_node_id` property, then joins or validates relational anchor. | Node primary key; canonical ref index; AGE property index if AGE used. | `2` for duplicate detection; exactly one for success. | Set per-query `statement_timeout` to request timeout bounded by `110`; application cancellation also required. | Cadastre node ID, graph profile, derived-view state, auth/redaction context. | Missing node returns empty or owner error according to endpoint policy; duplicate candidates reject. | `GRAPH_NODE_CANDIDATE_DUPLICATE`, `GRAPH_BACKEND_ID_FORBIDDEN`, `GRAPH_QUERY_TIMEOUT`. | Primary-key success, missing node, duplicate anchor, AGE internal ID leakage, unauthorized node. |
| `neighbor_expansion` | Query outbound and inbound adjacency from `graph_edge`, filtered by edge type, traversal class, assertion state, confidence, valid/known time. | Cypher expansion by allowed edge label, then return application-owned edge/node IDs only and validate relational anchors. | Outbound/inbound adjacency indexes; time/filter indexes. | TODO: product governance must set backend over-fetch multiplier and hard cap. Recommendation: explicit cap by query class, never unbounded. | Server `statement_timeout` plus application cancellation; partial results only when endpoint outcome permits. | Endpoint graph node ID, traversal classes, filters, page size, page token context, auth/redaction context. | Unsupported filter or query class rejects before backend query; empty traversal class returns empty result without backend traversal. | `GRAPH_QUERY_TRANSLATION_UNSUPPORTED`, `GRAPH_QUERY_CANDIDATE_LIMIT_EXCEEDED`. | High-degree node, empty result, filters, stale derived view, unauthorized neighbor, page-token context mismatch. |
| `bounded_path` | Recursive CTE over adjacency edges with max depth `1..6`, cycle guard, traversal-class filter, and candidate cap. Final sort by Cadastre path ordering after candidate materialization. | AGE variable-length traversal over allowed relationship label and traversal filters; return application IDs; revalidate against relational anchors and Cadastre ordering. | Adjacency indexes; partial indexes for active traversal classes/assertion states; optional time-filter indexes. | TODO: explicit path candidate cap by depth and query class. Must fail closed on overflow. | Request timeout must map to `statement_timeout` and application cancellation. Depth outside `1..6` fails before backend. | Start/end IDs, max depth, traversal class set, filters, Cadastre ordering tuple, page token context. | Theoretical reachability remains prohibited; unsupported traversal class returns no paths or owner error per profile. | `GRAPH_PATH_DEPTH_INVALID`, `GRAPH_PATH_CANDIDATE_LIMIT_EXCEEDED`, `THEORETICAL_REACHABILITY_SCOPE_ERROR`. | Depth 1 through 6, cycle graph, nondeterministic path tie, high-degree overflow, reachability-prohibited query. |
| `evidence_drillback` | Join `graph_node` or `graph_edge` to `source_gold_fact_id` and evidence bridge; return evidence metadata refs only unless raw permission allows more. | AGE object-to-application-ID lookup only; relational evidence tables supply drillback chain. | Edge evidence refs, source gold fact ID, node source object refs. | Page size + one for pagination; hard cap from `110`. | Same API timeout; no AGE-only evidence path. | Drillback ref, evidence refs, authorization decision, redaction context, page-token context. | AGE internal object IDs are invalid drillback refs. | `GRAPH_DRILLBACK_REF_INVALID`, `GRAPH_BACKEND_ID_FORBIDDEN`, `LINEAGE_ERROR`. | Node/edge drillback, missing gold fact, raw payload redaction, AGE internal ID rejection. |
| `analysis_read_only` | Execute only validated read-only SQL templates or imported analysis rules mapped through `GraphQueryTranslationProfile`; use read-only transaction and read-only role. | AGE read-only Cypher only after mutation prohibition proof; mutating Cypher forbidden. | Same as query class. | Per rule profile; TODO concrete caps. | `statement_timeout` plus application cancellation. | Analysis rule ref, graph compatibility matrix, auth/redaction, derived-view state. | Raw SQL/Cypher text not accepted unless validation import maps it into active profile. | `ANALYSIS_MUTATION_FORBIDDEN`, `GRAPH_PROVIDER_QUERY_BYPASS_FORBIDDEN`. | Mutating SQL/Cypher rejection, raw provider query import rejection, read-only transaction proof. |

## 9. Graph apply, rebuild, restore, and migration behavior

### PostgreSQL relational apply model

[Cadastre applicability inference] PostgreSQL native transactions are sufficient to provide atomicity for one committed relational apply batch. They are not sufficient by themselves to satisfy Cadastre replay, resume, schema-fingerprint, derived-view, read-after-write, and activation evidence requirements. Cadastre must persist explicit runtime state records for those effects.

| Apply concern | Required relational behavior |
| --- | --- |
| Transaction boundary | Default: one transaction for a complete `GraphDeltaSet` when size and timeout permit. For large deltas, batch transactions are allowed only when `GraphApplyProfile` defines deterministic batch boundaries and resume evidence. |
| Write batching | Batch order must be deterministic by delta operation class and Cadastre graph delta IDs. TODO: `090` must add a provider-neutral graph delta apply ordering table if it is not already closed. |
| Idempotent reapply | Reapplying the same `GraphDeltaIdempotencyKey` and input checksum must produce byte-identical `GraphApplyResult` or explicit idempotent no-op. Same key with different checksum must fail before mutation. |
| Duplicate insert behavior | Duplicate ID with identical canonical row checksum may be idempotent; duplicate ID with different checksum fails with a Cadastre graph collision error before derived-view advancement. |
| Partial apply | Forbidden unless `GraphApplyProfile` declares committed-batch state, resume boundary, read-after-write proof, and rollback behavior. |
| Resume behavior | Resume starts at the first uncommitted batch only after verifying schema fingerprint, input delta checksum, prior batch checksums, and run-lock/fencing evidence. |
| Read-after-write proof | Query selected nodes, edges, and counts after commit under the same schema fingerprint and record proof in `GraphApplyResult`. |
| Schema fingerprint validation | Compare expected `BackendSchemaFingerprint` before every mutation; stale fingerprint fails before DML. |
| Rebuild import | Rebuild from authoritative lakehouse state and persisted graph deltas only; never from PostgreSQL graph state alone. |
| Drift check | May emit operational health only; must not repair or mutate graph state. |
| Rollback | Prefer rebuild from lakehouse and persisted deltas. SQL transaction rollback is sufficient only for uncommitted batches. Committed batch rollback requires explicit compensating deltas or rebuild. |

### AGE apply model

[requires runtime validation] AGE graph mutation can be considered only after validating that controlled Cypher writes, relational anchor writes, transaction rollback, read-after-write proof, role restrictions, and direct namespace bypass controls behave correctly for the exact PostgreSQL and AGE versions.

| AGE apply concern | Required behavior |
| --- | --- |
| Anchor-first enforcement | Insert or validate relational anchor rows with constraints before AGE vertex/edge creation, preferably in the same PostgreSQL transaction. |
| AGE mutation templates | Only precompiled adapter templates may call `ag_catalog.cypher()` for mutation. Raw user-authored Cypher is forbidden. |
| Internal ID rejection | AGE-created vertex/edge IDs must be discarded after validating the application-owned property and anchor. |
| Bypass detection | Direct DML/DDL against AGE graph namespace by application roles must fail. Validation role must attempt bypass. |
| Extension availability | Preflight must fail if extension is absent, wrong version, unloaded, inaccessible, or unsupported by provider. |
| Upgrade | Exact AGE upgrade scripts must be rehearsed with graph data and indexes. If no upgrade path exists, graph rebuild from lakehouse must be the migration path. |

### Backup, restore, and disaster recovery

[confirmed primary-source fact] PostgreSQL documents SQL dump, filesystem-level backup, and continuous archiving as backup approaches. PostgreSQL extension packaging documentation states that `pg_dump` dumps `CREATE EXTENSION` rather than individual extension member objects, and extension files must be available on restore.[^22]

[Cadastre applicability inference] Relational PostgreSQL graph state can be backed up and restored through normal PostgreSQL operational mechanisms, but Cadastre graph-serving promotion after restore must still require `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, `BackendSchemaFingerprint`, `GraphApplyResult`, and `DerivedViewState` validation. For AGE, restore also requires matching AGE extension binaries, control files, SQL scripts, provider support, and extension version compatibility.

### Migration behavior

| Migration | PostgreSQL relational requirement | AGE requirement |
| --- | --- | --- |
| Graph schema migration | DDL migration plus fingerprint diff, validation fixtures, and rollback/rebuild plan. | Relational anchor migration plus AGE label/property/index migration; query parity fixtures. |
| Index rebuild | Non-authoritative maintenance; serving promotion blocked until index consistency check passes. | AGE property index rebuild validation plus relational anchor consistency. |
| PostgreSQL major upgrade | Rehearse `pg_upgrade` or dump/restore path, schema fingerprint diff, query plans, and restore. | Same plus AGE major-version compatibility and upgrade scripts. |
| Extension upgrade | Not applicable for relational-only unless using extensions. | Required exact AGE upgrade rehearsal; no implicit extension upgrade assumption. |
| Restore into new environment | Restore, validate schema fingerprint, apply package refs, run graph index consistency, and verify query parity. | Same plus extension files and graph namespaces. |
| Rollback after failed migration | Transaction rollback for uncommitted DDL only; otherwise restore/rebuild. | Restore/rebuild preferred; extension downgrade must not be assumed. |

## 10. Security, authorization, redaction, and bypass prevention

[confirmed primary-source fact] PostgreSQL row-level security can restrict row visibility and mutation by policy; if RLS is enabled and no policy exists, default-deny applies, while superusers and roles with `BYPASSRLS` bypass RLS. PostgreSQL also documents that `search_path` determines object resolution for unqualified names and that the default path is suitable only for a single user or a few mutually trusting users.[^23]

### Role model

| Role | Required permissions | Forbidden permissions |
| --- | --- | --- |
| Migration/admin role | Own schema migration, extension installation where allowed, controlled DDL, package-set approved setup. | Must not be used by API request handling or analysis queries. |
| Graph apply role | Insert/update/delete only through graph read-model tables or approved AGE mutation functions/templates; execute required preflight queries. | Direct schema DDL, extension management, graph drop, unsafe search path, raw AGE namespace DML unless explicitly owned and validated. |
| Read-only API role | Select through approved views/templates; read-only transactions. | DML, DDL, AGE mutation clauses, graph creation/drop, extension management. |
| Validation role | Execute negative fixtures in non-production and inspect catalog state. | Production mutation unless validation scenario explicitly runs in isolated environment. |
| Restricted role | Minimal role for bypass tests. | Any graph namespace DML/DDL, `ag_catalog.cypher()` mutation, direct table mutation, extension operations. |
| Operator role | View health and preflight evidence; trigger approved rebuild/migration workflows. | Direct SQL or Cypher mutation outside activation-controlled workflow. |

### Required negative security scenarios for `120`

| Scenario ID recommendation | Backend profile | Required outcome |
| --- | --- | --- |
| `120-POSTGRES-REL-DIRECT-DML-READONLY-FORBIDDEN` | PostgreSQL relational | Read-only API role attempts `INSERT`, `UPDATE`, or `DELETE` on graph tables; fails with no mutation. |
| `120-POSTGRES-REL-SCHEMA-DDL-APP-FORBIDDEN` | PostgreSQL relational | Apply or API role attempts schema DDL; fails before mutation. |
| `120-POSTGRES-REL-RLS-BYPASS-FORBIDDEN` | PostgreSQL relational | Non-admin role cannot bypass tenant or scope policy; superuser/BYPASSRLS role is never used by API. |
| `120-POSTGRES-REL-SEARCH-PATH-HIJACK` | PostgreSQL relational | Unsafe `search_path` or shadowed function/table name fails preflight. |
| `120-AGE-CYPHER-MUTATION-READONLY-FORBIDDEN` | AGE | Read-only role attempts `CREATE`, `SET`, `REMOVE`, `DELETE`, `DROP`, or graph creation through Cypher; fails. |
| `120-AGE-GRAPH-NAMESPACE-DML-FORBIDDEN` | AGE | Restricted role attempts DML/DDL against AGE graph namespace; fails. |
| `120-AGE-GRAPH-DROP-FORBIDDEN` | AGE | App role attempts `drop_graph` or destructive graph DDL; fails. |
| `120-AGE-INTERNAL-ID-LEAK` | AGE | Cypher returns AGE vertex/edge/path IDs; response materialization rejects them before caller-visible output. |
| `120-GRAPH-PROVIDER-QUERY-TEXT-REDACTION` | Both | Telemetry or API diagnostics include provider-native SQL/Cypher text; redaction rejects or removes it. |
| `120-GRAPH-TENANT-CROSS-SCOPE-FORBIDDEN` | Both | Query attempts cross-tenant or cross-scope read through backend filters; Cadastre auth/redaction prevents leakage after candidate materialization. |

### Redaction and audit

[Cadastre applicability inference] Provider-native query text, SQL literals, Cypher strings, query plans with literal values, graph property values, backend-generated IDs, private source bindings, and raw payload bytes must not appear in public API responses, telemetry, validation reports, package reports, audit exports, or health output unless an active `110`/`140` policy allows a bounded redacted form. Telemetry and health evidence must not become graph authority.

## 11. Package and supply-chain implications

### Package type sufficiency

[Cadastre applicability inference] The current `100.PackageType` enum is sufficient for PostgreSQL relational and AGE backend profiles. No new package type token is required for the backend provider, adapter, driver, runtime distribution, storage adapter, index adapter, deployment profile, schema profile, query translation profile, apply profile, taxonomy mapping, or validation rows. New tokens must not be invented unless a required artifact cannot be represented by the closed enum; no such artifact was identified in this research.

| Artifact | Package type | Package identity | Version source | Trust evidence | Compatibility evidence | SBOM/provenance need | Package-set activation need | Rollback effect | Validation fixture class |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PostgreSQL server runtime | `graph_backend_provider_package` or `graph_backend_runtime_distribution` depending on packaging boundary | PostgreSQL distribution image/package digest or managed provider profile ref | PostgreSQL version, provider version, image digest, or service release ref | Trust policy, repository metadata, provider evidence, SBOM when packaged | PostgreSQL major/patch, schema profile, driver, migration, backup/restore, query plan | Required for self-hosted/runtime package; managed provider evidence still needed | Required before backend preflight | Rollback to last-known-good package set or rebuild environment | Provider preflight, schema fingerprint, restore. |
| PostgreSQL adapter | `graph_provider_adapter_package` | Cadastre PostgreSQL graph adapter package digest | Adapter release manifest | Trust, provenance, SBOM, dependency lock | Query translation, apply, rebuild, errors, raw-write bypass | Required | Required | Rollback adapter and query/apply profile as package cohort | SQL query parity, apply idempotency, bypass negative. |
| PostgreSQL driver | `graph_backend_driver_package` | Driver package digest | Driver upstream release or internal build | Trust/provenance/SBOM | PostgreSQL protocol, transaction behavior, pooling behavior, TLS support | Required for code-bearing package | Required | Rollback with adapter | Connection, timeout, cancellation, transaction fixtures. |
| Relational storage schema/profile | `graph_read_model_schema_profile` | Activation-controlled row-set ref | Owner profile version | Package trust if package-supplied | Schema fingerprint, constraints, indexes, RLS, migration | Declarative SBOM only if packaged policy requires | Required | Rollback to prior schema profile or rebuild | Schema fingerprint, stale schema, duplicate IDs. |
| Relational index profile | `graph_index_backend_adapter_package` if separate adapter; otherwise schema profile | Index adapter or index policy ref | Profile or package version | Trust/provenance if packaged | Query class support, plan regression, index freshness | Required if code-bearing | Required | Rollback or rebuild indexes | Index missing/stale/plan regression. |
| AGE extension runtime | `graph_backend_runtime_distribution` plus `graph_backend_provider_package` | AGE extension binary/control/SQL scripts or managed provider extension ref | AGE release tag/version and PostgreSQL major version | Trust/provenance/SBOM for packaged extension or provider evidence for managed service | Exact PostgreSQL major, AGE release, upgrade path, restore, provider support | Required for self-hosted/Kubernetes; provider evidence for managed | Required for AGE profile activation | Rollback may require restore/rebuild; downgrade not assumed | Extension install, restore, upgrade, graph namespace. |
| AGE adapter | `graph_provider_adapter_package` | Cadastre AGE adapter package digest | Adapter release manifest | Trust/provenance/SBOM | Cypher templates, relational anchor handoff, bypass controls, query parity | Required | Required | Rollback adapter with AGE query/apply profiles | Cypher read-only, mutation, internal ID leak, anchor consistency. |
| Backend deployment profile | `graph_backend_deployment_profile` | Environment-specific redacted deployment profile ref | Package or private deployment profile version | Trust and private-binding redaction evidence | Provider support, region, service tier, roles, backup, restore, upgrade | Required by policy | Required | Rollback to prior deployment profile only if compatible | Provider support and restore/failover fixtures. |

[TODO: 100 owner approval] If implementation wants to package PostgreSQL connection-pool configuration or database migration tooling as a separate activation-controlled runtime artifact and cannot represent it through `graph_backend_driver_package`, `graph_provider_adapter_package`, `graph_backend_deployment_profile`, or `policy_bundle`, the `100` owner must decide whether a new package type is required.

## 12. Observability and health implications

### Health states

| Condition | Health scope | Required visible state | Domain effect |
| --- | --- | --- | --- |
| PostgreSQL connection unavailable | `graph` | `blocked` or `error` depending on endpoint outcome policy | No graph query or mutation; no domain mutation. |
| PostgreSQL schema fingerprint stale | `graph` | `blocked` | Apply/query/rebuild promotion blocked before mutation or serving. |
| Required index missing or stale | `graph` | `blocked` for affected query class | No serving for affected query class; no fact mutation. |
| Query timeout exceeded | `graph` or `api` | endpoint error or partial only if endpoint policy permits partial | No graph mutation. |
| AGE extension missing/wrong version | `graph` | `blocked` for AGE profile | AGE profile cannot serve, apply, rebuild, or promote. |
| AGE provider unsupported | `graph` and `package` | `blocked` | AGE profile activation blocked. |
| Telemetry exporter failure | `observability` | `degraded` by default | No domain output mutation. |

### Telemetry attribute policy refinement

| Attribute class | Default behavior |
| --- | --- |
| PostgreSQL server major/patch version | Allowed only as bounded low-cardinality operational metadata when profile permits. |
| AGE extension version | Allowed only as bounded low-cardinality operational metadata when AGE profile permits. |
| Backend schema fingerprint ref | Allowed as opaque ref. |
| Query translation profile ref | Allowed as opaque ref. |
| SQL query text | Reject or redact by default. |
| Cypher query text | Reject or redact by default. |
| Query plan with literals | Reject unless literals are removed and plan class is allowed by policy. |
| AGE vertex/edge/path IDs | Reject. |
| PostgreSQL OID/tuple IDs | Reject. |
| Canonical graph node/edge IDs | Reject unless future owner policy allows a bounded hashed diagnostic form. |

[Cadastre applicability inference] `140` does not need a new telemetry authority model. It needs PostgreSQL/AGE rows in existing telemetry attribute and redaction allowlists/rejection tables, plus fixtures that prove backend internal IDs and provider-native query text do not leak.

## 13. Performance-validation plan

[requires runtime validation] Performance adequacy cannot be claimed without a representative benchmark. The benchmark must compare PostgreSQL relational and AGE profiles under identical Cadastre query classes, data shapes, package refs, schema profiles, query translation profiles, and API bounds.

### Benchmark graph size tiers

| Tier | Nodes | Edges | Purpose |
| --- | ---: | ---: | --- |
| `tiny-fixture` | 100 | 500 | Deterministic correctness and replay. |
| `small-validation` | 10,000 | 100,000 | CI-scale query and apply validation. |
| `medium-mvp` | TODO: product governance | TODO: product governance | Expected MVP production-like tenant. |
| `large-stress` | TODO: product governance | TODO: product governance | Headroom, high-degree, bloat, timeout, and plan regression. |

### Required data distributions

| Dimension | Required cases |
| --- | --- |
| Degree distribution | Low degree, power-law-like distribution, and explicit high-degree nodes. |
| High-degree node | At least one node with TODO product-specified degree threshold; query must prove overflow and timeout behavior. |
| Edge/property cardinality | `observed_connection` only for MVP; property paths include declared assertion state, confidence, valid/known time, last_seen, evidence refs. |
| Valid-time and known-time filters | Current default, valid-at, known-at, expired interval, open interval, malformed interval rejection. |
| Assertion-state and confidence filters | Active, stale, conflicted, unknown, threshold edge cases. |
| Evidence drillback density | Sparse evidence, dense evidence, missing evidence, redacted evidence, unauthorized evidence. |
| Path depth | Bounded path depth 1, 2, 3, 4, 5, and 6. |
| Cycles | Cycle and self-loop cases that must not produce nondeterministic infinite traversal. |

### Required workloads

| Workload | Measurement |
| --- | --- |
| `node_detail` | p50, p95, p99 latency; duplicate detection; authorization/redaction overhead. |
| `neighbor_expansion` | p50/p95/p99 by degree bucket; candidate overflow rate; page-token correctness. |
| `bounded_path` | p50/p95/p99 by depth 1..6; path count; timeout and cancellation. |
| `evidence_drillback` | latency by evidence density and redaction state. |
| Concurrent reads | Throughput and latency under parallel graph queries. |
| Concurrent graph apply | Apply latency, lock wait, query isolation, read-after-write proof. |
| Rebuild/import | Total time, WAL volume, index build time, vacuum/bloat, schema fingerprint validation. |
| Timeout/cancellation | Statement timeout, application cancellation, retry/no-retry behavior. |

### Pass/fail thresholds

TODO: Product governance must supply concrete thresholds for p95/p99 query latency, maximum rebuild time, maximum apply time per delta set, maximum high-degree node response time, maximum acceptable bloat, maximum WAL volume, concurrent query throughput, timeout budget, and provider cost ceiling. Until thresholds exist, benchmark rows must be `blocked`, not `pass`.

## 14. Runtime-validation matrix

| Validation area | Profile | Required scenario | Evidence required | Pass condition | Fail condition |
| --- | --- | --- | --- | --- | --- |
| Backend preflight | PostgreSQL relational | Missing profile, inactive profile, checksum mismatch, missing required field, unpinned version. | `GraphBackendPreflightResult`, error record, no mutation proof. | Most-specific owner error, no backend mutation. | Any backend mutation or generic error. |
| Schema fingerprint | PostgreSQL relational | Stale table/index/constraint/RLS fingerprint. | Expected vs actual fingerprint diff and no mutation proof. | Apply/query/rebuild promotion blocked. | Query or mutation proceeds. |
| Duplicate IDs | PostgreSQL relational | Duplicate node/edge IDs in input and existing DB. | Constraint error mapped to Cadastre error. | No derived-view advancement; deterministic error. | Silent overwrite or backend ID leak. |
| Recursive path | PostgreSQL relational | Depth 1 through 6, cycles, tie ordering. | Query outputs and checksums. | Cadastre deterministic ordering and cycle handling. | Backend natural order drives output. |
| High-degree traversal | Both | High-degree node neighbor/path queries. | Latency, timeout, candidate cap evidence. | Meets TODO thresholds and cap behavior. | Timeout without owner error or unbounded candidate materialization. |
| Query timeout | Both | Statement exceeds API timeout. | Timeout error and cancellation evidence. | `GRAPH_QUERY_TIMEOUT` or owner error; no mutation. | Partial state mutation or hung query. |
| Apply idempotency | Both | Reapply same delta set and same idempotency key. | Byte-identical result or idempotent no-op. | Deterministic result. | Duplicate mutation or divergent result. |
| Partial apply resume | Both | Failure after committed batch. | Batch state, resume result, read-after-write proof. | Resume at correct batch only. | Reapply wrong batch or advance derived view without proof. |
| AGE install | AGE | Extension absent/wrong version/unloaded/unsupported. | Preflight record. | AGE profile blocked. | AGE query executes. |
| AGE read-only | AGE | Read-only API role attempts Cypher mutation. | Role grants, query error, no mutation proof. | Mutation forbidden. | Mutation succeeds. |
| AGE namespace bypass | AGE | Direct DML/DDL against graph namespace. | Role grants and failure evidence. | Bypass fails. | Direct namespace mutation succeeds. |
| AGE internal IDs | AGE | Cypher returns vertex/edge/path IDs. | Response materialization proof. | Internal IDs rejected or discarded. | Internal IDs appear in API/telemetry/cursor. |
| Backup restore | Both | Restore into clean environment. | Restore logs, package refs, fingerprint, query parity. | Clean restore passes gates. | Hidden state or missing extension blocks unhandled. |
| PostgreSQL major upgrade | Both | Upgrade exact version pair. | Upgrade logs, query parity, fingerprint diff. | Upgrade passes or rebuild path works. | Silent semantic drift or no rollback/rebuild. |
| Provider support | AGE | Azure/AWS/GCP exact provider-region-tier support check. | Provider docs or tenant validation evidence. | Profile activation matches provider support. | Profile activates where unsupported. |
| Observability redaction | Both | Backend query text/internal IDs in telemetry. | Telemetry export record. | Rejected/redacted. | Query text or IDs exported. |

## 15. NLSpec patch-impact map

| File | Exact section likely affected | Issue or gap | Required contract closure | Rows/tables likely needed | TODOs that must remain unresolved | Acceptance criteria needed |
| --- | --- | --- | --- | --- | --- | --- |
| `docs/nlspec/090-graph-projection-serving-and-backends.md` | `GraphBackendSelectionPolicy`; `GraphBackendProfile schema`; backend capability, schema, query, apply, preflight, blocker, and acceptance sections. | Omitted backend currently materializes `mvp-janusgraph.v1`; JanusGraph-specific fixtures and package gates are embedded in default closure. | Replace omitted default with `mvp-postgresql-relational-graph.v1` only if adopted; add relational PostgreSQL profile, AGE validation-only profile, PostgreSQL schema fingerprinting, SQL query translation, apply semantics, AGE extension blocker set, provider matrix refs, and JanusGraph non-default profile posture. | Backend profile rows, capability matrix rows, taxonomy mapping rows, schema profile rows, SQL/AGE query translation rows, apply profile rows, preflight rows, blocker rows. | TODO candidate limits; TODO benchmark thresholds; TODO exact PostgreSQL version; TODO exact AGE version; TODO provider deployment scope; TODO apply batch size; TODO migration strategy. | PostgreSQL default selection; relational schema fingerprint; SQL query parity; AGE extension blocked where unsupported; JanusGraph explicit non-default; no backend ID leakage. |
| `docs/nlspec/100-packages-supply-chain-and-activation.md` | `PackageTypePolicyRow` graph package rows; graph backend compatibility axes; package gate acceptance criteria. | Existing package tokens are sufficient but JanusGraph-specific driver/storage/index rows must generalize to PostgreSQL and AGE. | Add package policy rows for PostgreSQL server/runtime, adapter, driver, schema/profile packages, AGE extension/runtime distribution, deployment profiles, provider support evidence, restore/upgrade evidence, and compatibility axes. | Package compatibility matrix rows for PostgreSQL major/version, AGE extension version, provider support, driver, connection pool, migration tool, backup/restore, query plan, schema fingerprint. | TODO if separate connection-pool or migration-tool package type is needed; owner must approve if existing tokens cannot represent it. | Package activation fails without PostgreSQL/AGE package refs, compatibility rows, SBOM/provenance where required, provider support evidence, and package-set membership. |
| `docs/nlspec/110-api-ux-health-errors-and-security.md` | Graph errors, health states, security/redaction, GraphQuery response behavior. | PostgreSQL/AGE error and health states are not closed. | Add user-visible errors for PostgreSQL schema stale, duplicate ID, query timeout, provider unsupported, AGE extension missing, AGE mutation forbidden, namespace bypass, restore/upgrade blocked, query-plan regression. Map health states to `healthy`, `degraded`, `blocked`, `error`. | Error registry rows and owner context schemas for PostgreSQL/AGE graph failures. | TODO exact wording for operator-facing provider support and preview warnings. | API renders derived-view state, graph blocked state, timeout, duplicate, and provider unsupported without exposing SQL/Cypher text or backend IDs. |
| `docs/nlspec/120-validation-fixtures-and-acceptance.md` | `GraphBackendProviderValidationMatrix`; required negative test classes; graph fixture families. | Current backend rows are JanusGraph-specific and checksum TODO-bearing. | Replace or supplement JanusGraph fixture rows with PostgreSQL relational rows and AGE rows. Keep provider portability rows. | Fixtures for schema fingerprint stale, duplicate IDs, FK failure, SQL recursive path, high-degree timeout, direct DML, RLS, AGE extension unavailable, AGE mutation forbidden, AGE namespace bypass, AGE internal ID leak, restore, upgrade, provider unsupported. | TODO fixture checksums and expected output checksums until runtime tests exist; TODO performance thresholds. | Acceptance cannot pass while new PostgreSQL/AGE rows are `TODO`, blocked, failed, not run, or missing mutation-prohibition proof. |
| `docs/nlspec/140-observability-instrumentation.md` | Telemetry attribute policy, metric catalog, telemetry redaction policy. | PostgreSQL/AGE telemetry attributes are not explicitly categorized. | Add allowlist/rejection rows for PostgreSQL version, AGE extension version, schema fingerprint ref, provider profile ref, query plan class, SQL text, Cypher text, AGE IDs, OIDs, tuple IDs, and query literals. | Telemetry attribute policy rows and validation rows. | TODO exact metric threshold names and allowed low-cardinality tags. | Telemetry exports reject provider-native query text, internal IDs, raw graph properties, private bindings, and unbounded labels. |
| `docs/nlspec/domain.md` | Resolved terminology decisions; graph concepts; owner-routing tables. | Domain vocabulary lacks PostgreSQL relational graph and AGE optional profile routing. | Add domain-only definitions for PostgreSQL relational graph projection, relational anchor table, AGE graph namespace, AGE internal graph object ID, application-owned graph ID, provider extension support. Route runtime behavior to `090`/`100`/`110`/`120`/`140`. | Domain distinction table rows. | TODO only if owner specs conflict on terminology. | Domain remains non-runtime and routes backend behavior to owner specs. |
| `docs/nlspec/000-cadastre-spec-index.md` | Research registry, source-of-truth map, graph backend default acceptance criteria. | If report becomes `RES-017`, the registry and closure gates must reference it as research only, not runtime authority. | Add RES-017 reference row; update graph backend default acceptance row if default changes. | Registry row and source-of-truth map row. | TODO until patch planning decides whether to add report to corpus. | Research cannot be cited as runtime authority; PostgreSQL default gate references owner specs and `120` rows only. |
| `MANIFEST.md` | Research report list. | `RES-017` is not currently listed. | Add `docs/reference/research/RES-017-postgresql-and-apache-age-implementation-refinement.md` if report is adopted into corpus. | Manifest row. | TODO until file is added to repository. | Manifest path is normalized, unique, and repository-relative. |

## 16. Open questions and TODOs

| ID | Classification | Owner | Required resolution |
| --- | --- | --- | --- |
| `TODO-RES017-001` | open question | Product governance, `090` | Decide whether omitted graph backend default changes to `mvp-postgresql-relational-graph.v1` or whether PostgreSQL remains explicit until validation closes. |
| `TODO-RES017-002` | requires runtime validation | `090`, `120` | Select exact PostgreSQL major/patch version and run relational query/apply/rebuild fixtures. |
| `TODO-RES017-003` | requires runtime validation | `090`, `120` | Define backend candidate limits for node detail, neighbor expansion, bounded path, evidence drillback, and analysis read-only. |
| `TODO-RES017-004` | requires runtime validation | `090`, `120` | Define pass/fail performance thresholds for latency, throughput, rebuild, import, apply, high-degree traversal, bloat, WAL, and timeout behavior. |
| `TODO-RES017-005` | requires runtime validation | `090`, `100`, `120` | Select exact PostgreSQL driver, connection pool, migration tool, deployment profile, and package compatibility evidence. |
| `TODO-RES017-006` | requires runtime validation | `090`, `120` | Validate PostgreSQL recursive CTE path semantics, cycle detection, candidate overflow, and query-plan stability. |
| `TODO-RES017-007` | requires runtime validation | `090`, `120` | Rehearse backup, PITR, restore into new environment, failover, and major-version upgrade for relational PostgreSQL graph. |
| `TODO-RES017-008` | open question | `100` | Decide whether connection pool and migration tooling require new package type tokens or can be represented by existing graph driver/adapter/deployment package tokens. |
| `TODO-RES017-009` | requires runtime validation | `090`, `100`, `120` | For AGE, select exact AGE release and PostgreSQL major version; validate install, extension load, provider support, restore, upgrade, graph namespace security, and Cypher query parity. |
| `TODO-RES017-010` | operational risk | Product governance | Decide whether Azure AGE preview status and major-version-upgrade limitation are acceptable for any production deployment scope. |
| `TODO-RES017-011` | operational risk | Product governance | Decide whether AWS/GCP managed AGE absence blocks AGE for those deployment scopes or whether self-managed/Kubernetes deployment is acceptable. |
| `TODO-RES017-012` | requires runtime validation | `110`, `140`, `120` | Validate SQL/Cypher text redaction, backend internal ID rejection, query-plan redaction, and telemetry allowlist behavior. |
| `TODO-RES017-013` | requires runtime validation | `090`, `120` | Validate AGE graph-native constraints, property indexes, and direct SQL namespace bypass behavior; if constraints cannot be proven, require relational anchors. |
| `TODO-RES017-014` | open question | `090`, `120` | Decide whether AGE can ever be graph mutation substrate or only read-only candidate materialization for MVP. |

## 17. Final recommendation

[Cadastre applicability inference] Specify `mvp-postgresql-relational-graph.v1` as the replacement default candidate for Cadastre MVP graph serving, with production effects blocked until `090`, `100`, `110`, `120`, `140`, and `domain.md` close the profile, schema, query, apply, package, health, error, observability, and validation rows. The relational profile is the best default path because it can keep all MVP graph identity, endpoint, edge-type, evidence, time, assertion, confidence, schema, and apply constraints in PostgreSQL relational objects while preserving Cadastre’s non-authoritative graph boundary.

[Cadastre applicability inference] Specify `mvp-postgresql-age.v1` only as a validation-only optional graph-extension profile. AGE may become an optional production profile only after exact-version runtime validation proves extension installation, provider availability, graph namespace security, direct bypass prevention, application-owned ID enforcement, query parity, benchmark thresholds, restore, failover, and upgrade behavior. AGE must not become the omitted-backend default in the next patch.

[Cadastre applicability inference] Remove `mvp-janusgraph.v1` as the omitted-backend default in later NLSpec patching, but retain it as an explicit non-default profile until product governance retires it. The retirement decision must be separate from this research report and must pass provider-neutral graph portability fixtures.

[operational risk] The later patch must not claim PostgreSQL or AGE production adequacy from documentation alone. Production activation must remain impossible until validation rows contain concrete fixture checksums, expected output checksums, mutation-prohibition proofs, package-set refs, schema fingerprints, provider-support evidence, restore/upgrade evidence, and benchmark pass/fail evidence.

## Sources

[^1]: `docs/nlspec/090-graph-projection-serving-and-backends.md`, `MVP Graph Backend Selection Closure Requirements`, lines 234-258; `GraphBackendSelectionPolicy`, lines 278-303; `Definition of Done`, lines 1148-1157.
[^2]: `docs/nlspec/090-graph-projection-serving-and-backends.md`, `Projection Authority`, lines 97-103; `MVP Graph Active Profile Closure Requirements`, lines 220-232; `Definition of Done`, lines 1171-1185.
[^3]: `postgresql-apache-age-research-report.md`, lines 1-15 and 17-36. Used as prior Cadastre context and open-question checklist only, not as proof of external technical behavior.
[^4]: `docs/nlspec/090-graph-projection-serving-and-backends.md`, `Purpose`, lines 12-14; `Exports`, lines 55-95.
[^5]: `docs/nlspec/090-graph-projection-serving-and-backends.md`, `Projection Authority`, lines 97-103.
[^6]: `docs/nlspec/090-graph-projection-serving-and-backends.md`, `MVP Graph Active Profile Closure Requirements`, lines 220-232; `GraphEdgeDeltaIdPolicy` and related graph delta rules, lines 735-745.
[^7]: `docs/nlspec/100-packages-supply-chain-and-activation.md`, `Activation Unit`, lines 100-104.
[^8]: `docs/nlspec/100-packages-supply-chain-and-activation.md`, `Package Type Policy Contract`, lines 106-110; closed enum graph package tokens, lines 180-191; enum closure behavior, lines 197-205.
[^9]: `docs/nlspec/110-api-ux-health-errors-and-security.md`, `API Bounds`, lines 90-104.
[^10]: `docs/nlspec/120-validation-fixtures-and-acceptance.md`, `Required Negative Test Classes`, lines 63-86; `GraphBackendProviderValidationMatrix`, lines 1373-1407.
[^11]: `docs/nlspec/140-observability-instrumentation.md`, `Telemetry Non-Authority`, lines 75-87; default instrumentation profile, lines 89-107; telemetry redaction policy rows, lines 260-270.
[^12]: PostgreSQL official documentation, recursive queries, search order, cycle detection, and materialization behavior. citeturn637572view0
[^13]: PostgreSQL official documentation for constraints, indexes, JSON/JSONB, JSONB GIN, transaction isolation, locks, row-level security, backup/PITR, extension packaging, and `EXPLAIN`. citeturn821280view0turn821280view1turn821280view2turn821280view3turn821280view4turn821280view5turn771535view0turn771535view2turn821280view6turn821280view7turn771535view3turn771535view4turn469994view0turn771535view7turn771535view6turn469994view1
[^14]: Apache AGE official setup documentation, build/install, Docker, `LOAD`, `search_path`, and non-superuser grant guidance. citeturn914545view0
[^15]: Apache AGE official graph documentation for vertices, edges, graph creation/drop, graph namespace, `_ag_label_vertex`, `_ag_label_edge`, label tables, and reserved namespace warning. citeturn914545view1
[^16]: Apache AGE official Cypher and advanced SQL/Cypher documentation for `ag_catalog.cypher()`, return behavior, parameter map, CTE/JOIN behavior, and mutation limitations inside joins. citeturn914545view2turn914545view3
[^17]: Apache AGE official GitHub release notes for PostgreSQL 17 and 18 AGE 1.7.0 and AGE 1.6.0 PostgreSQL 14/15 upgrade caveats. citeturn526976view0turn526976view2
[^18]: Apache AGE official README compatibility and feature statements. citeturn526976view6
[^19]: Microsoft Learn official Azure PostgreSQL extension and AGE documentation for AGE preview version matrix, required `azure.extensions`, `shared_preload_libraries`, restart, `CREATE EXTENSION`, `search_path`, and in-place major-version-upgrade limitation. citeturn709206view2turn709206view0turn709206view1
[^20]: AWS official RDS and Aurora PostgreSQL extension documentation and Google Cloud SQL PostgreSQL extension documentation. The inspected pages did not list Apache AGE or `age`; Google documents that only supported extensions can be installed. citeturn709206view4turn429827view1turn311311view0turn311311view2turn709206view6turn311311view4
[^21]: PostgreSQL official documentation for recursive query ordering/materialization and client connection defaults including `statement_timeout`, `search_path`, transaction read-only defaults, and lock timeouts. citeturn637572view0turn235066view0
[^22]: PostgreSQL official backup documentation and extension packaging documentation. citeturn771535view7turn771535view6
[^23]: PostgreSQL official row-level security documentation and client connection defaults documentation for `search_path`. citeturn469994view0turn235066view0
