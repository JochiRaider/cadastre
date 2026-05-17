---
doc_id: RES-005
title: Lakehouses and Derived Graph Architectures Research Report for Cadastre
Inspection date: 2026-05-15
status: research-report
---

## 1. Executive summary

Cadastre’s governing architectural boundary is correct: the temporal lakehouse is the system of record, graph and CIM outputs are replaceable projections, and projection is the deterministic contract between authoritative records and serving outputs.[^1] The highest-value improvement from the lakehouse and metadata-system research is not to replace that boundary, but to make it more mechanically complete. The PRD already names `VersionManifest`, `SourceCompletenessProfile`, `GraphProjectionProfile`, `GraphDeltaSet`, `GraphApplyProfile`, and graph edge semantics. It still needs more exact contracts for table-format snapshot identity, table-set version identity, retention-driven replay eligibility, maintenance safety, and graph rebuild evidence.[^2]

### 1.1 Top findings

| Rank | Finding | PRD impact | Adoption stance | Main risk reduced |
| ---: | --- | --- | --- | --- |
| 1 | Cadastre must record table-format-native snapshot, commit, transaction-log, metadata-file, and catalog-reference identifiers in `VersionManifest` through explicit `LakehouseSnapshotRef` and `LakehouseCommitRef` rows. Iceberg table metadata, snapshots, manifests, and atomic metadata-file replacement, and Delta’s serial integer log versions and MVCC snapshots, show that reproducible reads depend on table-format-native state identifiers, not just a run ID.[^18][^20] | `VersionManifest`, temporal lakehouse table contract, replay, rollback, restore | `adopt` | Replay ambiguity and non-reproducible historical reads |
| 2 | Cadastre must define a `ReplayRetentionPolicy` and `TableMaintenancePolicy` that make replay eligibility explicit before snapshot expiration, vacuum, cleaning, orphan deletion, garbage collection, or Hudi restore. Iceberg snapshot expiration, Delta vacuum, Hudi cleaner, and lakeFS garbage collection can remove historical data needed for time travel or replay.[^19][^22][^23][^28] | raw/bronze retention, silver/gold replay, operational maintenance | `adopt` | Silent destruction of replay evidence |
| 3 | Graph rebuild must be specified as a lakehouse-to-derived-view contract with a persisted `GraphRebuildManifest`, not as a graph-backend operation. DataHub’s restore-index pattern rebuilds graph/search indexes from an authoritative metadata-aspect table, while Cadastre’s PRD already requires persisted graph deltas and forbids unpersisted graph-authoritative mutation.[^14][^5] | graph rebuild, graph read-model consistency, disaster recovery | `adopt` | Graph-authority leakage and unverifiable read-model state |
| 4 | Cadastre must distinguish pipeline lineage, source evidence lineage, table snapshot lineage, and graph projection lineage. OpenLineage’s run/job/dataset/facet model is useful, but it is additive pipeline metadata and too narrow for Cadastre source authority, bitemporal validity, evidence roles, completeness, identity, and graph projection lineage.[^16][^2] | `SourceStageDAG`, `VersionManifest`, `SourceCompletenessReceipt`, evidence refs | `adapt` | Treating OpenLineage as evidence or fact authority |
| 5 | Source completeness should be sharpened with table-format and connector evidence classes: page exhaustion, expected-count match, scope enumeration, permission visibility, table commit success, and snapshot/tag retention. The current PRD already defines `EvaluateSourceCompleteness`; it should add table-format receipt rows and maintenance eligibility gates.[^4][^23] | `SourceCompletenessProfile`, `CoverageAssertion`, watermarks, graph expiry | `adopt` | Unsafe absence, cleanup, and watermark advancement |
| 6 | Cross-table consistency must be a first-class optional contract. Nessie’s branch, tag, commit, content-key, and multi-table transaction semantics are a strong reference, but Cadastre should adapt them into `CrossTableCommitProfile` and `CatalogBranchPromotionPolicy` only when multiple Cadastre tables must be read as one atomic version set.[^24] | `VersionManifest`, staging, validation, promotion, table-set replay | `adapt` | Mixed-table snapshots and partial promotion |
| 7 | Metadata systems show useful event and aspect patterns, but Cadastre must not adopt current-state metadata graphs as canonical bitemporal facts. DataHub aspects and MCL/MCP streams, OpenMetadata entity/version/change-event surfaces, and Marquez lineage graph APIs are derived or metadata-oriented references, not Cadastre truth models.[^12][^13][^25][^26] | gold fact storage, graph projection, analysis rules | `cautionary_reference` | Metadata graph mistaken for fact authority |
| 8 | dbt artifacts demonstrate deterministic workflow-artifact value: `manifest.json`, `run_results.json`, `sources.json`, and `semantic_manifest.json` are useful narrow references for DAG, run result, freshness, and semantic artifact separation, but they must not define Cadastre lineage or graph semantics.[^27] | `SourceStageDAG`, validation scenarios, mapping compiler outputs | `adapt` | Blending authoring artifacts with production truth |

### 1.2 PRD areas strengthened, corrected, or constrained

| PRD area | Impact |
| --- | --- |
| `VersionManifest` | Add table-format-native snapshot and commit references, catalog branch/tag/commit references, cross-table snapshot set checksum, maintenance-policy versions, and replay-retention decision refs. |
| Temporal lakehouse table contract | Add `LakehouseTableProfile`, `LakehouseSnapshotRef`, `LakehouseCommitRef`, and `DatasetVersionRef` contracts. |
| Raw/bronze retention and replay | Add replay eligibility gates before maintenance operations and make payload, manifest, snapshot, and catalog retention consequences observable. |
| Silver replay and schema validation | Require exact table snapshot plus schema/profile refs for every replay input. |
| Gold fact storage | Require bitemporal correction to reference old and new table snapshots, not only logical facts. |
| Source completeness | Add completeness evidence rows for table commits, table scans, table snapshot availability, and maintenance-safety decisions. |
| Graph projection and apply | Add `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy`. |
| Analysis rules | Require rule compatibility to name graph rebuild manifest and schema fingerprint, not only graph profile version. |
| Operational maintenance | Define snapshot expiration, vacuum, cleaner, compaction, orphan cleanup, and object GC as replay-affecting operations. |

### 1.3 Biggest risks

The largest correctness risk is retention drift: a maintenance process can make historical replay impossible while all current-state tables still look healthy. The second risk is graph-authority leakage: operational convenience can cause the graph backend, search index, or metadata UI to become the de facto truth source. The third risk is cross-table inconsistency: Cadastre’s raw, silver, identity, gold, graph delta, graph apply, and health tables can be read from different table snapshots unless table-set versioning is explicitly required. The fourth risk is lineage conflation: OpenLineage and Marquez lineage represent pipeline/job/dataset flow, not source authority, bitemporal fact validity, identity resolution, or graph projection evidence.

### 1.4 Recommended next actions

1. Add `LakehouseTableProfile`, `LakehouseSnapshotRef`, `LakehouseCommitRef`, `DatasetVersionRef`, `TableMaintenancePolicy`, and `ReplayRetentionPolicy` to the PRD.
2. Extend `VersionManifest` to include table snapshot refs, table commit refs, catalog refs, cross-table snapshot set checksum, maintenance-policy refs, and replay-retention decision refs.
3. Add `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy` to make graph rebuilds provable from authoritative lakehouse state.
4. Add `RunDatasetIOContract` and `LineageFacetMappingPolicy` to adapt OpenLineage without letting facets replace Cadastre evidence or completeness contracts.
5. Commission a follow-up table-format selection report only after the PRD’s observable lakehouse contracts are defined.

## 2. Evidence inventory and freshness

### 2.1 Evidence class definitions

| Evidence class | Meaning |
| --- | --- |
| Confirmed source fact | Directly supported by source code, repository docs, schemas, tests, specifications, or official documentation. |
| Source-grounded inference | Inferred from multiple confirmed facts, source structure, named components, or documented control flow. |
| Cadastre applicability inference | A design implication compared against `PRD-Cadastre.md`. |
| Speculative transfer | Possible Cadastre use requiring deeper validation before PRD adoption. |
| Open question | A question requiring deeper source, operational, or domain review. |

### 2.2 Source inventory

| Source name | URL or path | Source type | Inspection date | Version, tag, commit, or release | Evidence inspected | Reliability | Limits |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cadastre PRD | `/mnt/data/PRD-Cadastre.md` | Project PRD | 2026-05-15 | Draft uploaded in project context | Architecture, scope, `VersionManifest`, completeness, projection, graph delta/apply, graph edge semantics | High for target contract | Treated as target artifact, not proof of external systems. |
| NLSpec standard | `/mnt/data/nlspec-spec.md` | Project standard | 2026-05-15 | Version 0.2.2 | Behavioral completeness, interfaces, defaults, mapping tables, acceptance criteria | High | Used as evaluation standard, not external system source. |
| Cartography research report | `/mnt/data/RES-001-cartography-research-report.md` | Prior research report | 2026-05-15 | RES-001 | Direct-to-graph ingestion, Neo4j graph, freshness cleanup, update tags, cleanup hazards | Medium-high | Context source; original repository not re-cloned in this report. |
| JupiterOne research report | `/mnt/data/RES-002-jupiterone-research-report.md` | Prior research report | 2026-05-15 | RES-002 | Starbase, JupiterOne SDK, graph-first synchronization, graph taxonomy cautions | Medium-high | Context source; public JupiterOne internals remain limited. |
| Taxi research report | `/mnt/data/RES-003-taxi-lang-research-report.md` | Prior research report | 2026-05-15 | RES-003 | Semantic overlay and compiler/tooling boundaries | Medium-high | Used only where schema-overlay tooling intersects lakehouse or projection contracts. |
| OCSF research report | `/mnt/data/RES-004-ocsf-schema-research-report.md` | Prior research report | 2026-05-15 | RES-004, OCSF target 1.8.0 | External normalization boundary and OCSF non-authority cautions | Medium-high | Used only to keep OCSF out of storage/identity/graph authority. |
| DataHub | `https://github.com/datahub-project/datahub`, `https://docs.datahub.com/` | Repository, docs, spec docs | 2026-05-15 | GitHub releases page showed `v1.5.0.6`, commit `d0fce94`, 2026-05-11 | Metadata model, aspects, entity registry, MCP/MCL, ingestion, graph/search rebuild, aspect retention | High for docs; medium for release metadata | Did not locally build DataHub or inspect every module. |
| OpenLineage | `https://github.com/OpenLineage/OpenLineage`, `https://openlineage.io/` | Repository, spec, API, docs | 2026-05-15 | Release `1.47.1`, commit `0331f2e`, 2026-05-13 | Run/job/dataset model, events, facets, custom facet rules, API schema, parent runs | High | Did not inspect every integration implementation. |
| Apache Iceberg | `https://github.com/apache/iceberg`, `https://iceberg.apache.org/spec/` | Repository, table spec, docs | 2026-05-15 | GitHub releases page showed `apache-iceberg-1.10.1`, commit `ccb8bc4`, 2025-12-22 | Table spec, snapshots, manifests, metadata files, schema IDs, deletes, commit scheme, releases | High | Did not run Iceberg; some docs opened from 1.9.1 for Nessie integration. |
| Delta Lake | `https://github.com/delta-io/delta`, `https://docs.delta.io/` | Repository, protocol spec, docs | 2026-05-15 | GitHub releases page showed `4.2.0`, 2026-04-16, commit `9f600c4` | Transaction protocol, MVCC, actions, txn app IDs, protocol evolution, checkpoints, vacuum | High | Did not inspect all implementation modules or table-feature docs. |
| Apache Hudi | `https://github.com/apache/hudi`, `https://hudi.apache.org/docs/` | Repository, docs, technical spec | 2026-05-15 | Docs included Hudi CLI examples for 1.1.1; release search indicated current 1.1.x line | Timeline, instants, states, rollback, restore, savepoint, cleaning, incremental queries | High for docs | Did not inspect source code or run Hudi table services. |
| Project Nessie | `https://github.com/projectnessie/nessie`, `https://projectnessie.org/` | Repository, docs, spec | 2026-05-15 | GitHub/docs release surface indicated 0.107.x compatibility line | Branches, tags, commits, content keys, content IDs, cross-table transactions, Iceberg integration | High for docs | Did not inspect release artifacts or run Nessie. |
| OpenMetadata | `https://github.com/open-metadata/OpenMetadata`, `https://docs.open-metadata.org/` | Repository, docs, API docs | 2026-05-15 | Docs release page showed 1.12.8 on 2026-05-13 | Entity store, ingestion framework, search index, lineage API, change events, webhooks, JSON schema standards | High for docs | Did not inspect database schema implementation or run indexing. |
| Marquez | `https://github.com/MarquezProject/marquez`, `https://marquezproject.ai/` | Repository, docs, API docs | 2026-05-15 | Latest release not established from primary release page; public project appears OpenLineage reference backend | OpenLineage HTTP backend, lineage API, web UI, runs/jobs/datasets, lineage graph | Medium-high | Search collisions with people named Marquez; release freshness not established. |
| dbt Core artifacts | `https://docs.getdbt.com/reference/artifacts/` | Docs, artifact schemas | 2026-05-15 | Docs current as of 2026-05-14/15; dbt-core release search showed v1.11.10 on 2026-05-14 | `manifest.json`, `run_results.json`, `sources.json`, `semantic_manifest.json` | High for docs | Narrow targeted artifact study only. |
| lakeFS | `https://github.com/treeverse/lakeFS`, `https://docs.lakefs.io/` | Repository, docs | 2026-05-15 | Docs current; release line not deeply inspected | Git-like data lake versioning, commits, branches, zero-copy, rollback, GC retention rules | High for docs | Included as optional expansion only for object-storage-level versioning comparison. |

## 3. Independent source reconstruction

### 3.1 DataHub

#### Problem solved (OpenLineage)

DataHub is a metadata platform that models metadata as entities and aspects, supports ingestion through pull and push pipelines, persists metadata state, emits metadata change events, and derives graph/search serving surfaces from persisted metadata. Its highest relevance to Cadastre is the separation between persisted metadata aspects and derived graph/search indexes.

#### Repository or system layout (OpenLineage)

DataHub’s public docs expose the main concepts Cadastre needs: a schema-first metadata model, a YAML entity registry, aspect schemas, MCP/MCL event streams, ingestion pipelines, metadata service, graph/search services, and index restore utilities. DataHub’s release page surfaced `v1.5.0.6` as latest during this inspection.[^29]

#### Major components (OpenLineage)

| Component | Responsibility | Cadastre relevance |
| --- | --- | --- |
| Entity and aspect model | Metadata entity types are composed of PDL-defined aspects. | Pattern for typed records, but not a Cadastre gold fact model. |
| Entity registry | YAML registry maps entities and aspects and validates model bindings at startup. | Reference for profile/contract registries. |
| Metadata Change Proposal | Write proposal for changing one entity aspect. | Reference for staged metadata writes. |
| Metadata Change Log | Event emitted after a metadata aspect changes. | Reference for derived index updates. |
| GMS / metadata service | Validates and persists aspect writes. | Reference for authoritative write path. |
| Graph/search services | Serve relationships and search from metadata state. | Reference for derived read-model rebuild. |
| RestoreIndices | Replays persisted aspects into graph/search indices. | Strong reference for Cadastre graph rebuild manifests. |

#### Data model (OpenLineage)

DataHub uses a schema-first model with Pegasus PDL schemas. The documentation says the metadata model is the schema-first foundation on which storage, serving, indexing, and ingestion operate; it also describes an entity registry and validated PDL aspect schemas.[^12] Aspects are associated with entities, and DataHub supports latest, versioned, and timeseries aspects. Aspect version zero represents the latest value; immutable aspect updates create new versions.[^12][^14]

Cadastre should adapt the aspect idea only as a record-versioning and derived-view-update reference. Cadastre must not collapse bitemporal `GoldFact` intervals into current-state aspect versions.

#### Control flow (OpenLineage)

DataHub accepts push and pull ingestion. Its docs describe ingestion sources and sinks, including Python ingestion sending metadata over HTTP or Kafka, and an MCP/MCL pipeline where proposals become logs after GMS persistence.[^13][^15]

Control-flow reconstruction:

```text
source connector or producer
  -> metadata change proposal
  -> metadata service validation
  -> persisted aspect write
  -> metadata change log
  -> graph/search index update
  -> graph/search/API serving
```

#### Persistence model

DataHub’s key transferable pattern is rebuildability. Its restore-index documentation states that search infrastructure and graph services can become out of sync and can be rebuilt from the authoritative `metadata_aspect_v2` table by replaying latest aspects as MCL events.[^14] It also documents database-backed aspect version retention policies: indefinite, latest-N, and time-based retention, with the latest version retained.[^14]

Cadastre implication: graph/search rebuilds must consume authoritative persisted lakehouse records and must produce explicit rebuild evidence. A graph backend must not be the source of truth.

#### Extension points

DataHub extension points include ingestion sources/sinks, metadata models/aspects, entity registry bindings, GraphQL/REST APIs, and MCL consumers. Cadastre should transfer the registry and event-stream patterns only when the output of the extension remains non-authoritative or governed by PRD contracts.

#### Validation strategy

The model-first PDL and entity registry provide schema validation at the metadata boundary. Cadastre needs stronger binary acceptance criteria: table snapshot refs, graph rebuild checksums, no-op graph delta coverage, completeness receipts, and retention eligibility.

#### Operational model

DataHub’s operational model includes ingestion pipelines, Kafka/HTTP metadata proposals, graph/search index update consumers, index restore utilities, and aspect retention policies. The high-value pattern is operational repair of derived views from persisted truth, not the specific graph backend.

#### Public APIs

DataHub exposes GraphQL/REST metadata APIs, MCP/MCL event streams, ingestion recipes, and graph/search query surfaces. Cadastre should not import API shapes; it should import the separation between write proposals, persisted state, and derived read models.

#### Governance artifacts

DataHub uses PDL schemas, entity registry definitions, docs, release notes, ingestion recipes, and restore-index operations. Cadastre should require each graph projection profile and rebuild manifest to be governed by explicit profile versions and acceptance criteria.

#### Tests and examples

This report did not inspect DataHub tests. DataHub docs and restore utilities provide enough source evidence for architectural transfer but not for implementation-level compatibility claims.

#### Concepts not to transfer

| Concept | Must not transfer | Rationale and Cadastre contract affected |
| --- | --- | --- |
| Current-state aspect graph as canonical truth | Must not replace `GoldFact` bitemporal facts. | Cadastre requires valid-time and known-time fact intervals, corrections, evidence, and assertion states. |
| MCL as complete evidence lineage | Must not replace `EvidenceRef`, `VersionManifest`, or source completeness. | MCL records metadata changes, not source authority or raw evidence preservation. |
| Search/graph index as authoritative | Must not become graph truth. | PRD requires graph deltas and graph apply from authoritative records only.[^5] |
| Aspect retention defaults | Must not imply Cadastre replay retention. | Cadastre replay eligibility depends on raw, silver, gold, table snapshots, and graph deltas. |

### 3.2 OpenLineage

#### Problem solved

OpenLineage standardizes pipeline lineage events around jobs, runs, datasets, event lifecycles, and facets. It gives Cadastre a strong vocabulary for pipeline-run context, parent runs, input/output datasets, source-code versions, and extensible facets. It does not solve source evidence authority, source completeness, identity resolution, bitemporal fact validity, or graph projection lineage.

#### Repository or system layout

OpenLineage includes a specification, API schema, facet specifications, governance around standard and custom facets, clients, and integrations. The current docs and GitHub release page showed release `1.47.1` on 2026-05-13.[^16][^29]

#### Major components

| Component | Responsibility | Cadastre relevance |
| --- | --- | --- |
| `RunEvent` | Event describing a pipeline run transition. | Source-stage run lineage and `VersionManifest` context. |
| `Job` | Logical process definition. | `SourceStageDAG` or stage/job identity analogue. |
| `Run` | Execution instance identified by UUID. | `run_id`/adapter stage execution analogue. |
| `Dataset` | Input or output data object. | `DatasetVersionRef` and run IO mapping. |
| Facets | Extensible metadata on jobs, runs, and datasets. | Controlled extension mechanism. |
| Transport | HTTP, Kafka, file, or integration-defined event delivery. | Non-authoritative event ingestion reference. |

#### Data model

OpenLineage distinguishes jobs and runs and identifies runs with UUIDs. Its run event lifecycle includes START, RUNNING, COMPLETE, ABORT, FAIL, and OTHER, and events include run, job, input datasets, output datasets, event time, producer, and schema URL.[^16] Facets attach metadata to runs, jobs, and datasets, custom facets use namespace prefixes, and facet schemas use immutable `_schemaURL` references.[^16]

#### Control flow

```text
pipeline engine or integration
  -> RunEvent START/RUNNING/COMPLETE/ABORT/FAIL
  -> transport
  -> lineage backend such as Marquez
  -> lineage graph and run/dataset history
```

#### Data flow

OpenLineage events carry pipeline metadata, not raw source records. They can name input/output datasets and versions. For Cadastre, this means OpenLineage-like records may describe `SourceStageDAG` run IO, but `RawRecord`, `CadastreSilverObservation`, `GoldFact`, `SourceCompletenessReceipt`, and `GraphDeltaSet` remain Cadastre-owned.

#### Persistence model (OpenLineage)

OpenLineage is a spec, not a storage engine. Backends such as Marquez persist and visualize it. Cadastre should persist its own lineage records and may expose OpenLineage-compatible events as an external projection.

#### Extension points (OpenLineage)

OpenLineage facets are the main extension point. Standard facets include parent run and source-code-related facets; custom facets must use distinct naming and immutable schemas.[^16][^17]

#### Validation strategy (OpenLineage)

OpenLineage uses JSON schemas and schema URLs. Cadastre must add stricter validation: every custom or analogous facet must map to a Cadastre contract row and must not satisfy evidence, completeness, or identity by itself.

#### Operational model (OpenLineage)

OpenLineage operationally depends on integrations emitting lifecycle events and transports delivering them. Cadastre should treat missing or failed OpenLineage transport as lineage telemetry failure, not as proof that source collection did or did not occur.

#### Public APIs (OpenLineage)

The OpenLineage API supports event and batch submission, with the run, job, input, and output fields encoded in `RunEvent`.[^16]

#### Governance artifacts (OpenLineage)

The OpenLineage spec, API schema, facets docs, integrations, and release notes govern behavior. Cadastre should define `LineageFacetMappingPolicy` so only declared facets are admitted.

#### Tests and examples (OpenLineage)

OpenLineage examples show START/COMPLETE events with job, run, source-code version, inputs, outputs, and schema metadata.[^16]

#### Concepts not to transfer (OpenLineage)

| Concept | Must not transfer | Rationale and Cadastre contract affected |
| --- | --- | --- |
| Facets as Cadastre evidence roles | Must not replace `EvidenceRef.evidence_role`. | Facets are extensible metadata, not source evidence authority. |
| Dataset version as raw evidence retention proof | Must not replace table snapshot refs or payload refs. | Cadastre replay requires exact raw and table-state availability. |
| Pipeline lineage as source authority | Must not decide gold facts or absence. | Cadastre requires `SourceAuthorityProfile` and `SourceCompletenessProfile`. |
| OpenLineage backend as graph read model | Must not satisfy Cadastre graph serving. | Cadastre graph has bitemporal filters, evidence drillback, assertion states, and deterministic ordering. |

### 3.3 Apache Iceberg

#### Problem solved (Apache Iceberg)

Iceberg defines table semantics for large analytic tables: immutable metadata files, snapshots, manifest lists, manifests, data files, schema IDs, partition evolution, row-level deletes, snapshot isolation, atomic commits, and time travel. It is the strongest reference for Cadastre’s reproducible historical table reads.

#### Repository or system layout (Apache Iceberg)

The relevant source is the Iceberg table specification and docs. The GitHub releases page showed `apache-iceberg-1.10.1`, commit `ccb8bc4`, as the latest release entry inspected.[^29]

#### Major components (Apache Iceberg)

| Component | Responsibility | Cadastre relevance |
| --- | --- | --- |
| Table metadata file | Authoritative table state document. | Required in `LakehouseSnapshotRef`. |
| Snapshot | Table state at a point in time. | Required in `VersionManifest` for every table read. |
| Manifest list | Snapshot-level list of manifest files. | Replay and audit input. |
| Manifest | Tracks data/delete files and partitions. | Evidence for file-level table contents. |
| Schema ID / field ID | Stable schema evolution mechanism. | Required for Cadastre schema evolution. |
| Partition spec ID | Partition evolution mechanism. | Needed for reproducible scan planning. |
| Delete file | Position/equality delete representation. | Retraction and mutation evidence. |
| Snapshot refs | Branches/tags for retained snapshots. | Retention and promotion reference. |

#### Data model (Apache Iceberg)

Iceberg tracks table state in metadata files. The spec states that every change creates a new table metadata file and atomically swaps the old metadata pointer; snapshots represent table state, and manifests track data files.[^18] Table metadata includes fields such as format version, table UUID, location, last sequence number, schema lists and current schema ID, partition specs, current snapshot ID, snapshot log, and metadata log.[^18]

#### Control flow (Apache Iceberg)

```text
writer creates data/delete files
  -> writer creates manifests and manifest list
  -> writer creates new metadata file
  -> catalog/metastore atomic pointer swap
  -> readers load current or selected snapshot
```

Iceberg’s commit scheme uses atomic swap or compare-and-set over metadata-file pointers; readers use table snapshots while writers use optimistic concurrency and retry if the base is no longer current.[^18]

#### Data flow (Apache Iceberg)

The table’s observable content derives from the selected snapshot. Cadastre must record the table metadata file, snapshot ID, manifest list location, schema ID, partition spec ID, and catalog reference for any run that reads or writes Iceberg-backed tables.

#### Persistence model (Apache Iceberg)

Iceberg metadata and data files are immutable until deleted. Row-level deletes are tracked through delete files, including position and equality deletes.[^19] Snapshot expiration removes old snapshots from metadata and eliminates time travel to those snapshots when data files are no longer referenced.[^19]

#### Extension points (Apache Iceberg)

Iceberg supports catalogs, schemas, partition evolution, sort orders, snapshot refs, delete files, and format versions. Cadastre should not depend on one implementation but must require an adapter to expose these observable identifiers.

#### Validation strategy (Apache Iceberg)

Iceberg’s correctness depends on metadata-file, snapshot, manifest, schema, partition, and file-level invariants. Cadastre should validate that `LakehouseSnapshotRef` rows resolve to a consistent table view before replay or graph rebuild.

#### Operational model (Apache Iceberg)

Operational maintenance includes snapshot expiration, orphan-file removal, compaction/rewrite procedures, and branch/tag retention. Cadastre must make such maintenance ineligible unless replay-retention checks prove no required replay scope is damaged.

#### Public APIs (Apache Iceberg)

Iceberg exposes table APIs through engines and catalogs. Cadastre should specify the observable identifiers required from any Iceberg-compatible implementation instead of binding to one API.

#### Governance artifacts (Apache Iceberg)

The Iceberg table spec is the main governance artifact. Release notes and docs describe features and implementation behavior.

#### Tests and examples (Apache Iceberg)

This report did not inspect Iceberg test suites. The table spec is sufficient to support PRD-level contract recommendations.

#### Concepts not to transfer (Apache Iceberg)

| Concept | Must not transfer | Rationale and Cadastre contract affected |
| --- | --- | --- |
| Iceberg snapshot as Cadastre fact validity | Must not replace valid-time/known-time. | Snapshot time is table state, not domain validity. |
| Partition spec as source scope | Must not replace `SourceCompletenessProfile.scope_key`. | Partitioning is storage layout, not source authority or coverage. |
| Snapshot expiration defaults | Must not govern Cadastre retention by default. | Cadastre needs replay eligibility and evidence retention. |
| Table metadata as domain model | Must not define asset, identity, or graph semantics. | Iceberg is a storage/table format. |

### 3.4 Delta Lake

#### Problem solved (Delta Lake)

Delta Lake defines a transaction-log protocol for ACID table semantics over files and object stores. It uses MVCC, serial integer table versions, JSON log actions, checkpoints, protocol versions/table features, idempotent app transactions, deletion vectors, and vacuum retention.

#### Repository or system layout (Delta Lake)

The relevant source is `PROTOCOL.md`, utility docs, and release metadata. The release page exposed Delta Lake `4.2.0` in April 2026.[^29]

#### Major components (Delta Lake)

| Component | Responsibility | Cadastre relevance |
| --- | --- | --- |
| `_delta_log` | Transaction log directory. | Commit/source-of-truth for table versions. |
| Versioned JSON log entries | Atomic table versions. | `LakehouseCommitRef` input. |
| `add` actions | Add logical data files. | File inclusion evidence. |
| `remove` actions | Tombstone logical files. | Delete/retraction evidence. |
| `txn` actions | Application transaction IDs and versions. | Idempotent write support. |
| Protocol action | Required reader/writer versions and features. | Compatibility check. |
| Checkpoints | Compacted transaction-log state. | Rebuild/read performance and replay evidence. |
| Vacuum | Physical deletion of no-longer-current files. | Replay eligibility risk. |

#### Data model (Delta Lake)

Delta’s protocol is a file-backed ACID transaction protocol. Its overview says readers get a consistent snapshot by using the transaction log, writers optimistically write files and then commit by adding a new log entry, and table versions are contiguous monotonically increasing integers.[^20] The log supports incremental processing by tailing the Delta log.[^20]

`add` and `remove` actions modify the table by adding or removing logical files, and deletion vectors can represent row-level logical deletion while data files remain physically present.[^21] The `txn` action stores an application ID and version and lets an application make transactions idempotent at the table level.[^21]

#### Control flow (Delta Lake)

```text
writer writes candidate files
  -> writer validates current table version and protocol
  -> writer appends next transaction-log JSON file
  -> checkpoint may compact log state
  -> reader reconstructs snapshot from checkpoint plus log tail
```

#### Persistence model (Delta Lake)

Delta persists a single serial history of atomic table versions. Checkpoints encode state for efficient reads, with V1/V2 checkpoint forms and sidecar files.[^21] Vacuum physically removes data files that are no longer needed after a retention period; docs state that time travel older than retention can be lost after vacuum.[^22]

#### Extension points (Delta Lake)

Delta’s protocol evolution uses min reader/writer protocol versions and feature lists. Clients must respect required reader/writer features.[^20] Cadastre should require protocol and feature data in `LakehouseTableProfile`.

#### Validation strategy (Delta Lake)

Cadastre should validate that a Delta-backed `LakehouseSnapshotRef` can reconstruct the requested version from checkpoint and log files, and that the required protocol version and table features are supported before replay, rollback, or graph rebuild.

#### Operational model (Delta Lake)

Delta supports batch/streaming semantics, incremental log tailing, checkpoints, protocol evolution, vacuum, and time travel. Cadastre must treat vacuum as a replay-affecting maintenance operation.

#### Public APIs (Delta Lake)

Delta APIs are engine-specific; the transferable contract is the table protocol and observable log state, not Spark-only APIs.

#### Governance artifacts (Delta Lake)

`PROTOCOL.md` and docs are the authoritative protocol references. Release notes identify current version context.

#### Tests and examples (Delta Lake)

This report did not inspect Delta test suites.

#### Concepts not to transfer (Delta Lake)

| Concept | Must not transfer | Rationale and Cadastre contract affected |
| --- | --- | --- |
| Delta table version as fact validity | Must not replace bitemporal intervals. | A table version is storage state. |
| Vacuum retention default | Must not become Cadastre replay retention. | Replay windows are product contracts. |
| `txn` app ID as Cadastre run identity | Must not replace `VersionManifest.run_id`. | It is table-format idempotence metadata, not complete Cadastre run lineage. |
| Protocol feature flags as domain semantics | Must not define Cadastre asset/identity/graph meaning. | They only govern transaction-log interpretation. |

### 3.5 Apache Hudi targeted reconstruction

Hudi solves lakehouse ingestion, mutation, incremental query, and table-service problems using a table timeline. Hudi timeline actions include commits, delta commits, replace commits, cleaning, compaction, clustering, rollback, savepoint, and restore. Timeline actions move through requested, inflight, and completed states, and Hudi describes timeline state transitions as atomic and timeline-consistent.[^23]

Hudi’s strongest Cadastre relevance is failure and lifecycle handling. Rollback automatically cleans up partially failed writes; readers ignore non-completed commits; future commits can detect and clean failed writes, including through heartbeat timeout in multi-writer cases.[^23] Savepoints prevent the cleaner from deleting saved file slices and allow restore; restore can delete data files and timeline files after the savepoint and requires operational caution.[^23]

Hudi also offers snapshot, incremental, and read-optimized query modes. Incremental queries read records written since a begin instant and are useful for change-stream-style processing.[^23]

#### Concepts not to transfer (Apache Hudi)

| Concept | Must not transfer |
| --- | --- |
| Hudi instant time as Cadastre known time | Instant time is table-timeline state, not Cadastre known-time. |
| Automatic rollback as sufficient replay guarantee | Rollback handles failed table writes, not Cadastre raw/silver/gold/graph replay. |
| Cleaner defaults as evidence-retention policy | Cleaning balances storage cost and history; Cadastre must define replay retention explicitly. |
| Incremental query output as graph delta truth | Graph deltas must derive from gold facts under `GraphProjectionProfile`. |

### 3.6 Project Nessie targeted reconstruction

Nessie provides Git-like catalog versioning for data lakes. It manages branches, tags, commits, content keys, and content IDs, and supports Iceberg tables and views. Its docs describe cross-table transactions either through branches and merge or through single commits containing many object changes; it also exposes APIs for isolation levels and records operations over table objects.[^24]

Nessie’s specification distinguishes content keys, which resolve symbolic names such as table names, from content IDs, which are immutable per content object. For Iceberg tables, a Nessie commit refers to an Iceberg table metadata pointer plus on-reference state such as snapshot ID, schema ID, partition spec ID, and sort order ID.[^24]

Cadastre should adapt Nessie concepts into table-set versioning contracts only when needed. A catalog branch is a useful staging and promotion mechanism, but it must not become an approval workflow unless the PRD defines approval, validation, promotion, rollback, and audit semantics.

#### Concepts not to transfer (Project Nessie)

| Concept | Must not transfer |
| --- | --- |
| Branch name as production approval | Branch semantics are catalog visibility, not governance approval. |
| Nessie commit hash as full replay proof | Cadastre also needs table snapshots, raw refs, schema/profile refs, package refs, and graph deltas. |
| Content key as canonical domain identity | Content keys name tables/views, not Cadastre assets. |
| Catalog rollback as full product rollback | Product rollback must cover raw/silver/gold/identity/graph/apply/health artifacts. |

### 3.7 OpenMetadata targeted reconstruction

OpenMetadata is a metadata platform with an API, UI, ingestion framework, entity store, search engine, JSON-schema-based metadata standard, lineage APIs, change events, and webhooks. Its high-level design docs identify the API as the main interface, the ingestion framework as the connector foundation, an entity store backed by MySQL for entity/relationship state, and Elasticsearch for UI search indexing.[^25]

OpenMetadata lineage APIs can query, create, delete, and export lineage between entities, including column-level mappings and pipeline references.[^25] It captures technical and business metadata changes as new entity versions, and metadata changes generate events; webhooks can receive metadata event notifications.[^25]

OpenMetadata is useful as a comparative metadata graph and ingestion reference. Cadastre should not adopt its metadata graph as canonical cyber asset truth.

#### Concepts not to transfer (OpenMetadata)

| Concept | Must not transfer |
| --- | --- |
| Metadata entity versions as gold bitemporal facts | Entity versioning is metadata lifecycle, not Cadastre valid/known-time fact correction. |
| Search index as storage authority | Search is derived from entity store. Cadastre graph/search must remain derived. |
| Lineage API edge as Cadastre graph edge | Cadastre edge semantics require fact type, evidence, direction, confidence, temporal policy, and non-implication rules. |
| Governance tags as source authority | Tags do not satisfy `SourceAuthorityProfile` or `CoverageAssertion`. |

### 3.8 Marquez targeted reconstruction

Marquez is an OpenLineage reference backend. The project describes itself as a service for consuming, storing, and visualizing OpenLineage metadata, with a real-time metadata server exposing an OpenLineage-compatible endpoint.[^26] Its UI shows dependencies between jobs and datasets, and run metadata for current and previous job runs.[^26] Its lineage graph API uses node IDs for datasets, dataset fields, and jobs, with configurable graph depth defaulting to 20.[^26]

Marquez is a useful backend reference for lineage visibility and run history. It is not a Cadastre evidence store, asset graph, source-completeness engine, identity resolver, or bitemporal fact store.

#### Concepts not to transfer (Marquez)

| Concept | Must not transfer |
| --- | --- |
| Marquez lineage graph as Cadastre graph serving | Lacks Cadastre graph edge semantics, evidence drillback, bitemporal filters, and graph apply contracts. |
| OpenLineage event acceptance as source completeness | Event receipt does not prove source scope completeness. |
| Dataset version as table snapshot proof | Dataset lineage version is not equivalent to table-format snapshot refs. |
| UI lineage depth defaults | Must not define Cadastre graph query bounds. |

### 3.9 dbt artifact targeted reconstruction

dbt’s artifacts provide a narrow, high-value reference for deterministic workflow artifacts. `manifest.json` contains a full representation of a dbt project’s resources and includes node configurations, sources, parent and child maps, selectors, and disabled resources.[^27] `run_results.json` contains completed invocation information, timing, and status for executed nodes only.[^27] `sources.json` records source freshness results with `max_loaded_at`, `snapshotted_at`, freshness criteria, and source node IDs.[^27] `semantic_manifest.json` contains Semantic Layer information, lives in `target`, and is used by MetricFlow as the integration point for semantic query planning.[^27]

Cadastre should adapt the artifact separation: static project DAG, executed-run results, source freshness, and semantic graph artifacts must be separate records with separate checksums.

#### Concepts not to transfer (dbt artifact)

| Concept | Must not transfer |
| --- | --- |
| dbt DAG as source-stage DAG authority | dbt DAG is transformation-specific and lacks Cadastre stage output permissions. |
| Source freshness as source completeness | Freshness does not prove absence, coverage, or permission visibility. |
| Semantic manifest as graph model | Semantic graph artifacts are query-planning inputs, not Cadastre graph serving. |
| Executed nodes only in run results as complete run evidence | Cadastre must record skipped, forbidden, partial, and no-op stage outcomes. |

### 3.10 lakeFS targeted reconstruction

lakeFS provides Git-like version control over object-storage data lakes. Its docs describe branching for isolated versions, committing for reproducible points in time, merging atomically, rollback, zero-copy branching, and configurable garbage collection.[^28] lakeFS acts as a metadata layer that determines which objects should be fetched for a version while clients read/write through the underlying object store.[^28]

Its garbage-collection docs state that by default lakeFS keeps all objects forever for time travel, but GC can remove committed objects that were deleted or replaced after retention, and uncommitted inaccessible objects. GC rules determine how long deleted/replaced objects are retained, including branch-specific retention.[^28]

Cadastre should use lakeFS as an optional object-storage-level versioning comparison. It must not substitute for table-format snapshots or Cadastre domain lineage.

#### Concepts not to transfer (lakeFS)

| Concept | Must not transfer |
| --- | --- |
| Object-storage commit as table snapshot | Object refs do not encode table-format schema, manifest, delete, or transaction semantics. |
| lakeFS branch as production workflow | Branch/merge must be wrapped in Cadastre promotion contracts. |
| Default forever retention as sufficient replay policy | Cadastre must define bounded retention and explicit replay eligibility. |
| Object path identity as asset identity | Object paths are storage identities, not Cadastre entities. |

## 4. Source-specific do-not-transfer analysis

| Source | Source pattern | Required caution | Cadastre contract that would be weakened |
| --- | --- | --- | --- |
| DataHub | Current-state metadata aspects and graph/search indexes | Do not treat metadata aspects or indexes as gold facts. | `GoldFact`, `GraphProjectionProfile`, `GraphDeltaSet` |
| OpenLineage | Run/job/dataset/facet events | Do not treat facets as evidence authority, completeness, or bitemporal validity. | `EvidenceRef`, `SourceCompletenessProfile`, `VersionManifest` |
| Iceberg | Table metadata and snapshots | Do not let table-format state define asset, identity, source authority, or valid time. | `GoldFact`, `IdentityDecision`, `SourceAuthorityProfile` |
| Delta Lake | Transaction-log versions and app transaction IDs | Do not let log version or `txn` app ID replace run, evidence, or cross-table version manifest. | `VersionManifest`, `RunDatasetIOContract` |
| Hudi | Automatic rollback and cleaner | Do not assume table rollback or cleaner safety preserves Cadastre replay. | `ReplayRetentionPolicy`, `TableMaintenancePolicy` |
| Nessie | Branches, tags, commits | Do not treat catalog branch promotion as production approval unless PRD defines approval contract. | `CatalogBranchPromotionPolicy`, lifecycle machines |
| OpenMetadata | Entity store and metadata knowledge graph | Do not import its entity graph as Cadastre canonical graph. | `GraphEdgeSemantics`, `GraphTaxonomyTranslationPolicy` |
| Marquez | Lineage graph over jobs/datasets | Do not use it as Cadastre graph serving or evidence drillback. | `GraphReadModelSchemaProfile`, `EvidenceRef` |
| dbt artifacts | Manifest/run/freshness artifacts | Do not treat freshness or run artifacts as source completeness. | `SourceCompletenessReceipt`, `CoverageAssertion` |
| lakeFS | Object-level versioning | Do not equate object versioning with table-format or bitemporal fact versioning. | `LakehouseSnapshotRef`, `GoldFact` |
| Cartography | Direct-to-graph ingestion and cleanup | Do not bypass raw/silver/gold/identity/projection contracts. | Entire PRD source-of-truth boundary |
| JupiterOne/Starbase | Graph entities and relationships as integration output | Do not use graph sync as authoritative mutation model. | `GraphDeltaSet`, `SourceStageDAG`, `GoldFact` |
| Taxi | Semantic aliases and TaxiQL | Do not use semantic overlay equality as identity or graph semantics. | `IdentityDecision`, `GraphEdgeSemantics` |
| OCSF | Event/object normalization taxonomy | Do not treat OCSF as storage, graph, identity, lineage, or bitemporal fact model. | `CadastreSilverObservation`, `GoldFact`, `GraphProjectionProfile` |

## 5. Transferable architectural concepts

| Concept | Finding | Source evidence | Transfer mechanism | Cadastre area improved | Adoption stance | Complexity introduced | PRD revision needed | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| System-of-record vs derived view | DataHub rebuilds graph/search from persisted metadata aspects; Cadastre already declares lakehouse authoritative. | DataHub restore-index docs; PRD graph boundary.[^14][^1] | Add `GraphRebuildManifest` and `GraphIndexConsistencyCheck`. | Graph rebuild, disaster recovery | `adopt` | Medium | Add graph rebuild contracts. | Given a gold snapshot set, rebuild produces the same graph checksum as a clean apply or fails with `GRAPH_INDEX_CONSISTENCY_ERROR`. |
| Lakehouse table state | Iceberg metadata/snapshots/manifests and Delta transaction log versions are precise table-state identifiers. | Iceberg spec and Delta protocol.[^18][^20] | Add `LakehouseSnapshotRef`, `LakehouseCommitRef`, `LakehouseTableProfile`. | Replay, temporal reads | `adopt` | Medium | Extend `VersionManifest`. | Replay rejects if any required snapshot or commit ref cannot be resolved byte-identically. |
| Cross-table versioning | Nessie enables catalog-level branches/commits and cross-table visibility. | Nessie transactions/spec docs.[^24] | Add optional `CrossTableCommitProfile`. | Multi-table consistency | `adapt` | Medium-high | Add table-set commit contract. | A run that reads multiple authoritative tables records one table-set version checksum or rejects production replay. |
| Derived graph rebuilding | DataHub restore-index pattern proves derived indexes can be rebuilt from authoritative persistence. | DataHub restore-index docs.[^14] | Require graph rebuild from gold facts and graph delta sets. | Graph serving | `adopt` | Medium | Add rebuild manifest. | Rebuild manifests list input snapshots, projection profile, schema profile, output checksum, and drift result. |
| Metadata event streams | DataHub MCP/MCL and OpenMetadata change events represent metadata change streams. | DataHub MCP/MCL; OpenMetadata change events.[^13][^25] | Adapt as non-authoritative operational event streams. | Health, derived view update | `adapt` | Medium | Define event-vs-fact boundary. | Dropping an event can delay derived view update but cannot delete or create authoritative facts. |
| Lineage/provenance | OpenLineage run/job/dataset and facets provide pipeline run context. | OpenLineage spec/API.[^16] | Add `RunDatasetIOContract` and `LineageFacetMappingPolicy`. | SourceStageDAG, VersionManifest | `adapt` | Medium | Define facet mapping rows. | Each emitted external lineage event maps to Cadastre run, stage, dataset IO, and table snapshot refs, or is rejected. |
| Source ingestion frameworks | DataHub/OpenMetadata ingestion and dbt artifacts separate source config, execution, artifacts. | DataHub ingestion; OpenMetadata design; dbt artifact docs.[^15][^25][^27] | Reinforce `SourceStageDAG` and validation artifacts. | Ingestion, validation | `adapt` | Medium | Add artifact checksums for run IO. | Same source inputs and artifact refs produce byte-equivalent validation output. |
| Adapter decomposition | JupiterOne SDK and Cartography show client/collector/converter/step separation and risks. | Prior reports.[^9][^8] | Keep Cadastre output permissions strict. | Source packages | `adapt` | Low | No new contract; tighten `SourcePackageDeveloperContract`. | Adapter tests prove collection cannot emit silver/gold/graph records. |
| Source completeness | Hudi rollback, dbt freshness, Cartography cleanup guards, and PRD completeness all show unsafe absence risks. | Hudi rollback; dbt freshness; Cartography/PRD.[^23][^27][^8][^4] | Add table and source API evidence rows. | Completeness, cleanup, watermarks | `adopt` | Medium | Extend `SourceCompletenessProfile`. | Unsafe states cannot emit absence, cleanup, retraction, or watermark advancement. |
| Replay/time travel | Iceberg, Delta, Hudi, Nessie, lakeFS all expose time-travel/rollback primitives with retention limits. | Table docs.[^18][^20][^23][^24][^28] | Define replay eligibility independently. | Replay, DR | `adopt` | Medium | Add `ReplayRetentionPolicy`. | Maintenance cannot proceed if it invalidates any protected replay window. |
| Schema evolution | Iceberg schema IDs/field IDs and Delta protocol/column mapping show format-specific evolution metadata. | Iceberg spec; Delta protocol.[^18][^20] | Require table-format schema metadata in `LakehouseTableProfile`. | Schema compatibility | `adopt` | Medium | Add replay compatibility checks. | Replay fails before output if schema ID/protocol/field-ID mapping mismatch. |
| Incremental processing | Delta log tailing and Hudi incremental queries are useful for projection updates. | Delta protocol; Hudi querying docs.[^20][^23] | Use only as input optimization; output must match full recomputation. | Graph projection | `adapt` | Medium-high | Add incremental/full equivalence criterion. | Incremental projection over a table version interval equals full projection diff over same endpoints. |
| Graph projection | Cartography/JupiterOne show direct graph loading hazards; Cadastre already requires persisted deltas. | Prior reports and PRD.[^8][^9][^5] | Strengthen no-op and rebuild evidence. | Graph determinism | `adopt` | Medium | Add graph rebuild/consistency contracts. | Every gold fact is projected to node, edge, or explicit no-op. |
| Validation and governance | NLSpec requires defaults, mappings, and binary acceptance criteria. External systems show registries and schemas. | NLSpec and source docs.[^7] | Add acceptance criteria to new PRD contracts. | NLSpec completeness | `adopt` | Low | Add mapping tables and algorithms. | Two implementations produce interchangeable table refs, lineage records, and graph rebuild outcomes. |
| Operational maintenance | Table services and GC affect replay. | Delta vacuum, Hudi cleaner, lakeFS GC, Iceberg expiration.[^19][^22][^23][^28] | Add maintenance preflight and evidence. | Retention and DR | `adopt` | Medium | Add `TableMaintenancePolicy`. | Maintenance emits a decision record and refuses unsafe deletions. |
| Failure and rollback | Hudi rollback/restore and PRD graph apply lifecycle show partial failure must be observable. | Hudi rollback; PRD graph apply.[^23][^5] | Add table-write failure receipts and graph rebuild failure evidence. | Failure recovery | `adopt` | Medium | Extend run and table commit records. | Failed commits never advance watermarks and record first failed table/delta ID. |

## 6. Required comparison matrices

### 6.1 Lakehouse storage and catalog semantics matrix

Declared scope: Iceberg, Delta Lake, Hudi, Nessie, lakeFS.

| Row | Iceberg | Delta Lake | Hudi | Nessie | lakeFS |
| --- | --- | --- | --- | --- | --- |
| authoritative state representation | Table metadata file pointing to current snapshot, manifests, schemas, specs. | `_delta_log` serial table versions plus checkpoints. | Timeline instants and file slices. | Catalog commit graph over content objects and keys. | Object-level repository commits and metadata refs. |
| commit identity | New metadata file and catalog atomic swap. | Monotonic integer log version. | Instant time/action/state. | Commit hash/ref state. | Commit ID. |
| snapshot identity | Snapshot ID. | Table version number. | Completed instant / file slice state. | For Iceberg, Nessie commit plus Iceberg snapshot ID/on-reference state. | Commit/ref and object version set. |
| table version identity | Table metadata location + snapshot ID + schema/spec IDs. | Delta log version + protocol/table features. | Completed instant. | Content key + content ID + commit + table-format state. | Repository + branch/tag/commit. |
| cross-table transaction support | Table-level by default. | Table-level by default. | Table-level by default. | Strong catalog-level reference through branch/merge and multi-object commit. | Object/repository-level atomic commit/merge, not table-format-aware. |
| schema evolution mechanism | Schema IDs and field IDs. | Protocol/table features, metadata schema, column mapping. | Timeline plus table schema metadata. | Catalog records table-format metadata refs. | Format-agnostic; does not define table schema semantics. |
| partition evolution mechanism | Partition spec IDs. | Partition metadata in transaction log. | Partition paths and timeline/file-slice metadata. | Stores Iceberg partition spec ID for Iceberg tables. | Format-agnostic object paths. |
| row-level mutation support | Position/equality delete files. | Remove/add actions, deletion vectors, row tracking where enabled. | Upsert/delete by record key and table type. | Delegates to table format. | Does not define row-level table semantics. |
| delete/retraction representation | Delete files and snapshot changes. | `remove` actions and deletion vectors. | Deletes/replace commits/rollback. | Catalog updates to content object state. | Object deletion/replacement in commits. |
| incremental-read mechanism | Snapshot diff/scan planning and engine APIs. | Tail Delta log / change data feed where enabled. | Incremental queries since instant. | Catalog commit history plus table-format diff. | Commit diff/object changes. |
| time-travel mechanism | Snapshot ID/timestamp while retained. | Version/timestamp while log/data retained. | Instant time/savepoint while retained. | Branch/tag/commit references to table states. | Branch/tag/commit refs while objects retained. |
| rollback/restore mechanism | Rollback to snapshot / catalog pointer update. | Restore/time travel/write new version, implementation-specific. | Savepoint and restore; rollback failed commits. | Branch reset/merge/transplant semantics depending API. | Revert/rollback branch HEAD. |
| retention and cleanup mechanism | Snapshot expiration and orphan-file deletion. | Vacuum and log retention/checkpoint cleanup. | Cleaner, compaction, clustering, rollback, restore. | Nessie GC/management plus table-format retention. | GC rules per branch/repository. |
| orphan-file behavior | Orphan file deletion can remove unreferenced files. | Vacuum removes unneeded data files after retention. | Cleaner removes older file slices; rollback removes failed write files. | Delegates data-file cleanup to table format/object store policies. | GC removes inaccessible uncommitted and expired deleted/replaced objects. |
| concurrent-write model | Optimistic commit with atomic pointer swap. | Optimistic commit to next log version. | Timeline states, lock/heartbeat behavior. | Catalog isolation levels and commit primitives. | Branch/commit/merge semantics. |
| idempotent write support | Engine/catalog-specific. | `txn` app ID/version action. | Record keys/timeline rollback; engine-specific. | Commit primitives; app idempotence external. | Commit identity external; object-level. |
| metadata-file or transaction-log shape | Metadata JSON, manifest lists, manifests. | JSON log actions, checkpoints, sidecars. | Timeline files, metadata table, file groups/slices. | Commit metadata, content objects, keys, refs. | Repository metadata, commits, object pointers. |
| reader isolation | Snapshot isolation. | Snapshot isolation. | Snapshot/read-optimized/incremental query modes. | Ref/commit-consistent catalog view. | Commit/ref-consistent object view. |
| writer isolation | Serializable/optimistic table commit. | Serializable writes and snapshot reads by protocol goal. | Timeline-consistent actions with concurrency controls. | Configurable optimistic isolation primitives. | Branch isolation; merge conflicts possible. |
| catalog dependency | Requires catalog/metastore for current metadata pointer. | Self-describing log; catalog optional depending engine. | Table path/timeline; metastore optional. | Catalog itself. | Object-store metadata layer. |
| replay implications for Cadastre | Record metadata file, snapshot ID, manifest list, schema/spec IDs. | Record version, checkpoint/log range, protocol/features, txn action. | Record instant, action, state, savepoint/cleaner policy. | Record branch/tag/commit/content key/content ID/table on-reference state. | Record repo/ref/commit and GC retention decision. |
| PRD contract likely affected | `LakehouseSnapshotRef`, `ReplayRetentionPolicy`. | `LakehouseCommitRef`, `TableMaintenancePolicy`. | `TableMaintenancePolicy`, completeness and rollback. | `CrossTableCommitProfile`, `CatalogBranchPromotionPolicy`. | `ReplayRetentionPolicy`, object retention. |
| transfer recommendation | `adopt` table-state identifiers. | `adopt` log/version/idempotence identifiers. | `adapt` failure/rollback/cleaning semantics. | `adapt` catalog-level table-set semantics. | `cautionary_reference` object-versioning only. |

### 6.2 Metadata graph and derived index matrix

Declared scope: DataHub, OpenMetadata, Marquez, Cartography, JupiterOne/Starbase, Cadastre PRD.

| Row | DataHub | OpenMetadata | Marquez | Cartography | JupiterOne/Starbase | Cadastre PRD |
| --- | --- | --- | --- | --- | --- | --- |
| authoritative persistence layer | Metadata aspects in DB/KV; graph/search derived. | Entity store in MySQL plus search index. | Marquez database for OpenLineage metadata. | Neo4j graph itself is persistence for Cartography graph state. | JupiterOne/Neo4j graph sync outputs. | Temporal lakehouse raw/silver/gold; graph derived. |
| metadata/event write model | MCP proposals and MCL logs. | Entity APIs, versions, change events/webhooks. | OpenLineage event ingestion and API writes. | Provider modules write graph directly with generated Cypher. | Integrations emit graph entities/relationships for sync. | `SourceStageDAG` writes layer-specific records; graph via `GraphDeltaSet` only. |
| entity model | URN entities + aspects. | JSON-schema entities and relationships. | Jobs, runs, datasets, fields. | Neo4j labels/properties. | `_class`, `_type`, `_key`, graph metadata. | `CanonicalEntity`, `SourceAsset`, `Identifier`, `GoldFact`, graph nodes. |
| relationship model | Relationships extracted from aspects, graph service. | Entity relationship store and lineage edges. | Job/dataset lineage graph. | Declarative relationship schemas. | Direct/mapped graph relationships. | `GraphEdgeSemantics` from gold facts. |
| lineage model | Dataset and entity lineage metadata. | Lineage API with column mappings/pipelines. | OpenLineage run/job/dataset. | Graph relationships and analysis jobs. | Graph relationships and metadata. | Separate source evidence, pipeline, table, and projection lineage. |
| graph index or graph store | Graph service backed by ES or Neo4j depending deployment. | Knowledge graph over entity store; search via Elasticsearch. | Lineage graph API. | Neo4j. | Neo4j or JupiterOne platform. | Replaceable graph read model. |
| search index | Elasticsearch/OpenSearch. | Elasticsearch/OpenSearch. | API/UI search limited by project. | Neo4j queries/rules. | JupiterOne query/search. | Optional graph/search read model under schema profile. |
| API surfaces | REST, GraphQL, ingestion APIs. | REST, SDKs, lineage APIs, webhooks. | REST/Lineage API, UI. | CLIs and Neo4j queries. | SDK/CLI/platform APIs. | Product API, graph query, evidence drillback. |
| ingestion framework | Pull/push, recipes. | Connector ingestion workflows. | OpenLineage HTTP backend/integrations. | Provider intel modules. | Integration SDK/Starbase. | Source instances, packages, adapters, DAG stages. |
| extension mechanism | Aspects, entities, ingestion sources. | JSON schemas, connectors, custom properties. | OpenLineage facets/events. | Provider modules, graph schemas, rules. | SDK integrations, graph taxonomy. | Package artifacts, profiles, mappings, policies. |
| schema or model registry | Entity registry and PDL. | OpenMetadata Standards JSON schemas. | OpenLineage schemas. | Python schema classes. | Data model docs/OpenAPI/SDK. | PRD/NLSpec contracts and registries. |
| rebuildability of graph/search/indexes | Restore indices from `metadata_aspect_v2`. | Reindex from entity store through API/connectors. | Rebuild lineage views from stored lineage events where retained. | No separate lakehouse; graph is storage. | Backend sync opaque; not Cadastre-style replay. | Must rebuild from lakehouse and persisted deltas. |
| current-state versus historical model | Latest/versioned/timeseries aspects, not Cadastre bitemporal facts. | Entity versions/change events, not Cadastre bitemporal facts. | Run/dataset history, not asset facts. | `firstseen`/`lastupdated`, cleanup, not bitemporal. | Graph lifecycle timestamps, not Cadastre intervals. | Bitemporal gold facts plus current/recent graph read model. |
| evidence drillback support | Metadata lineage/drilldown, not raw evidence. | Entity/lineage metadata, not raw evidence lake. | Run/facet metadata. | Limited graph records and source module context. | `_rawData` exists but in graph-object sync context. | Required graph->gold->silver->raw chain. |
| authorization/redaction considerations | Platform-specific. | Governance/access/search policies. | Backend/API-specific. | Neo4j/user deployment-specific. | JupiterOne platform-specific. | Raw permission, graph property redaction, evidence policy. |
| testing and validation model | Schema/model validation and ingestion tests. | JSON schemas, API tests, connector tests. | OpenLineage compatibility. | Exact Cypher and integration tests. | SDK client/converter/step tests. | `ValidationMatrix` and golden corpus. |
| Cadastre transfer recommendation | Adopt derived-index rebuild concept. | Adapt entity schema/change-event concepts. | Adapt lineage visibility only. | Cautionary direct-graph reference. | Cautionary graph-first reference. | Governing target. |

### 6.3 Lineage event and run model matrix

Declared scope: OpenLineage, Marquez, DataHub lineage/event concepts, dbt artifacts, Cadastre PRD concepts.

| Row | OpenLineage | Marquez | DataHub | dbt artifacts | Cadastre PRD |
| --- | --- | --- | --- | --- | --- |
| run identity | UUID run ID, UUIDv7 recommended in docs. | Stores/retrieves run metadata. | Ingestion/pipeline run metadata in platform concepts. | Invocation represented in artifacts; `run_results` per command. | `run_id` in `VersionManifest`, stage runs, graph apply runs. |
| job identity | Namespace/name job. | Job nodes in lineage graph. | Data jobs/pipelines as entities. | Nodes/resources in manifest. | `SourceStageDAG` and stage IDs. |
| dataset identity | Namespace/name datasets. | Dataset node IDs. | Dataset URNs/entities. | Sources/models by unique IDs. | `DatasetVersionRef`, table refs, raw/silver/gold table refs. |
| source identity | Producer and integration context. | Lineage API source endpoints. | Ingestion source and platform. | Source definitions in manifest/sources. | `SourceInstance`, source category/dataset/scope. |
| event type | START, RUNNING, COMPLETE, ABORT, FAIL, OTHER. | OpenLineage events and API operations. | MCP/MCL change types. | Command artifacts; executed node statuses. | Stage lifecycle, run manifest status, graph apply status. |
| event time | `eventTime`. | Stored event/run timestamps. | Change-event timestamps. | Invocation and freshness timestamps. | `processing_window`, collected/ingested/known/valid times. |
| input/output datasets | Explicit arrays. | Lineage graph jobs to datasets. | Dataset lineage/aspect relationships. | Parent/child maps and artifacts. | `RunDatasetIOContract` proposed. |
| dataset version | Dataset facets / backend-specific. | Dataset versions through API. | Versioned aspects/metadata. | Manifest/schema versions. | `DatasetVersionRef` proposed with table snapshot refs. |
| schema version | Dataset schema facets. | Dataset schema metadata. | Aspect/schema versions. | Artifact schema versions. | Cadastre schema contract and external schema artifact refs. |
| source-code version | Standard job/run facets. | Facet storage. | Ingestion metadata possible. | Project commit external, not inherent in artifact. | Package artifact digest and mapping/compiler versions. |
| parent run/job | ParentRunFacet. | Stored as facet where emitted. | Pipeline lineage concepts. | DAG parent/child maps, not run parent. | `SourceStageDAG` dependencies and lifecycle evidence. |
| custom extension mechanism | Custom facets with namespace and `_schemaURL`. | Stores OpenLineage facets. | Custom aspects/properties. | Project metadata and artifacts. | `LineageFacetMappingPolicy`, source-extension fields. |
| transport | HTTP, Kafka, file, integrations. | HTTP API/backend. | Kafka/HTTP MCP/MCL. | Files in `target`. | Persisted lakehouse records; optional external projections. |
| storage backend | Spec-independent. | Marquez DB. | Metadata DB/KV. | Files/artifacts. | Lakehouse tables and registries. |
| failure events | ABORT, FAIL. | Run states. | Change/failure via platform events. | Node statuses in run results. | Stage lifecycle, error records, completeness unsafe states. |
| additive versus replacing metadata | Facets can replace same-named facet; events additive in lifecycle. | Backend-specific. | Aspect updates create new versions/logs. | New artifact per invocation. | Immutable manifests/facts with correction rules. |
| replay support | Not sufficient by itself. | Backend run history, not Cadastre replay. | Aspect replay for indices, not Cadastre data replay. | Artifact replay if inputs retained. | Replay from persisted inputs and manifest refs. |
| bitemporal support | No Cadastre bitemporal fact model. | No. | Versioned metadata, not Cadastre bitemporal. | No. | Required for `GoldFact`. |
| Cadastre gap | Need mapping from facets to Cadastre lineage types. | Need separation between lineage backend and evidence store. | Need no graph-authority leakage. | Need artifact/run separation. | Add `RunDatasetIOContract`, `DatasetVersionRef`, `LineageFacetMappingPolicy`. |
| PRD revision proposal | Add facet mapping policy. | Treat Marquez as optional lineage projection. | Add graph rebuild manifest. | Add deterministic artifact refs to validation. | Implement proposed contracts. |

### 6.4 Graph projection and rebuild matrix

Declared scope: DataHub, OpenMetadata, Cartography, JupiterOne/Starbase, Cadastre PRD.

| Row | DataHub | OpenMetadata | Cartography | JupiterOne/Starbase | Cadastre PRD recommended requirement |
| --- | --- | --- | --- | --- | --- |
| graph is authoritative or derived | Derived from metadata aspects. | Knowledge graph over entity store; search derived. | Graph is primary persisted asset state. | Graph output/sync is central. | Graph must be derived from lakehouse gold/deltas. |
| input to graph mutation | MCL/aspect updates. | Entity/lineage updates and change events. | Provider transformed dictionaries. | Integration graph entities/rels. | Validated `GraphDeltaSet` only. |
| graph node identity | Entity URNs/relationships. | Entity FQNs/IDs. | Neo4j labels + stable IDs. | `_key`, `_type`, `_class`, `_id`. | Deterministic graph node IDs from projection rows. |
| graph edge identity | Relationship extraction. | Lineage/relationship store. | Cypher MERGE pattern. | Direct/mapped relationship keys. | Deterministic edge ID from edge type, src, dst, discriminator. |
| relationship direction semantics | Metadata relationship rules. | Lineage direction and relationship semantics. | Schema direction via `LinkDirection`. | Relationship classes/directions. | `GraphEdgeSemantics` row required. |
| evidence references | Metadata-level references. | Metadata lineage/context. | Limited to graph/source module. | `_rawData` in graph objects. | Graph->gold->silver->raw evidence chain. |
| index preflight | Restore/reindex utilities. | Search index/entity store behavior. | Index creation before writes. | Platform/backend-specific. | `GraphReadModelSchemaProfile` preflight before mutation. |
| apply ordering | Event/consumer ordering. | Event/indexing ordering platform-specific. | Provider stage order and Cypher write order. | Sync backend diffing. | Required operation order in `GraphDeltaSet`/`GraphApplyProfile`. |
| idempotent reapply | Reindex from source of truth. | Reindex/replay from entity store. | MERGE idempotence if IDs stable. | Backend sync. | Same delta/profile must produce same served state and no duplicate/watermark double-advance. |
| partial apply behavior | Restore utility can rebuild. | Event retries/indexing behavior. | Provider-specific hazards. | Backend-specific. | Persist `GraphApplyResult`; fail/rollback/resume per profile. |
| cleanup/expiry behavior | Aspect/version retention and index restore. | Entity deletion and change events. | `lastupdated` cleanup. | Backend diff/delete. | Expiry only through completeness-authorized `GraphExpiryScopeRow`. |
| rebuild from authoritative state | Yes, restore indices from aspect table. | Possible via entity store APIs/reindex. | No separate authoritative lakehouse. | No Cadastre-style evidence. | Required `GraphRebuildManifest`. |
| failure recovery | Restore/reindex. | Reindex/retry. | Provider cleanup guards. | Sync retry/backend. | Graph apply lifecycle and rebuild manifest. |
| query API | Graph/search APIs. | Lineage/search APIs. | Neo4j/Cypher/rules. | JupiterOne query. | Bounded graph query with evidence, bitemporal filters, authorization. |
| authorization | Platform-specific. | Governance policies. | Deployment-specific. | Platform-specific. | Graph property evidence/redaction policies. |
| recommended Cadastre requirement | Rebuild evidence and source-of-truth separation. | Change-event and index-drillback caution. | Use as cleanup hazard reference. | Use as graph-first caution. | Add graph rebuild and consistency contracts. |

## 7. Cadastre applicability analysis

| Finding | Strengthens PRD | Corrects or sharpens existing requirement | Constrains under-specified behavior | Improves data model | Improves graph model | Improves ingestion/normalization | Improves lineage/replay/audit | Improves maintainability | Complexity/risk | Conflict with direction |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Table-format snapshot refs | Yes | Yes | Yes | Yes | Indirect | Indirect | Yes | Yes | Medium | No |
| Replay retention and maintenance policy | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Medium | No |
| Graph rebuild manifest | Yes | Yes | Yes | Indirect | Yes | No | Yes | Yes | Medium | No |
| OpenLineage facet mapping | Yes | Yes | Yes | No | No | Yes | Yes | Yes | Medium | No, if bounded |
| Cross-table commit profile | Yes | Yes | Yes | Yes | Indirect | Indirect | Yes | Yes | Medium-high | No, optional |
| Metadata event stream adaptation | Yes | Yes | Yes | No | Yes | No | Yes | Yes | Medium | No, if non-authoritative |
| dbt artifact separation | Yes | Yes | Yes | No | No | Yes | Yes | Yes | Low | No |
| Graph-first cautions from Cartography/JupiterOne | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Low | No |

Decision table:

| Recommendation | Meaning in this report |
| --- | --- |
| `adopt` | Add a direct Cadastre requirement or contract pattern. |
| `adapt` | Use the concept but change semantics or boundary. |
| `cautionary_reference` | Use as a hazard or comparison, not a design import. |
| `do_not_transfer` | Reject because it conflicts with Cadastre architecture or NLSpec precision. |

## 8. NLSpec-oriented gap analysis

### Gap 1: Table-format-native snapshot identity is not explicit enough

```text
Gap: VersionManifest is comprehensive but does not define a normalized, table-format-specific LakehouseSnapshotRef row for every read table.
Why it matters: Two implementations could record different snapshot identifiers for the same table and both claim replay support.
Evidence: Iceberg requires snapshot IDs, table metadata, schemas, partition specs, and manifests; Delta requires log versions/protocol; Cadastre VersionManifest already requires every output-affecting input but not a table-format mapping table.
Affected PRD section or contract: VersionManifest; temporal lakehouse table contract.
Required decision: Define table-format reference rows for Iceberg, Delta, Hudi, Nessie, lakeFS, and other_declared.
Proposed requirement: Every production run must record one LakehouseSnapshotRef per authoritative table read, including table_format, table_identifier, snapshot_identity, schema_identity, partition_identity when applicable, metadata_file_or_log_range, catalog_ref, and retention_status.
Required mapping table: LakehouseSnapshotRef by table format.
Acceptance criteria: Replay with a missing or mismatched LakehouseSnapshotRef fails before output with LAKEHOUSE_SNAPSHOT_REF_ERROR.
Risk if omitted: Non-reproducible historical reads and unverifiable graph rebuilds.
```

### Gap 2: Table commits and idempotent writes need explicit commit refs

```text
Gap: VersionManifest records many package/profile versions, but table commit identity is not isolated as a reusable contract.
Why it matters: Delta has txn app IDs and integer versions; Iceberg has metadata-file commits; Hudi has instants. Cadastre needs one contract for write evidence.
Evidence: Delta logs next atomic table versions and txn app IDs; Iceberg commits via metadata-file pointer swap; Hudi writes timeline instants.
Affected PRD section or contract: VersionManifest, raw/silver/gold write stages, graph delta persistence.
Required decision: Add LakehouseCommitRef.
Proposed requirement: Every table write stage must emit LakehouseCommitRef rows for attempted, succeeded, failed, rolled_back, restored, or maintenance commits.
Required mapping table: Commit identity by table format.
Acceptance criteria: A succeeded output write without a matching LakehouseCommitRef is rejected with LAKEHOUSE_COMMIT_REF_ERROR.
Risk if omitted: Partial table writes can be mistaken for successful stage output.
```

### Gap 3: Retention, vacuum, cleaning, snapshot expiration, orphan deletion, and GC must become replay eligibility decisions

```text
Gap: PRD discusses retention broadly, but table maintenance does not yet have a deterministic replay-safety gate.
Why it matters: Iceberg snapshot expiration, Delta vacuum, Hudi cleaner, and lakeFS GC can remove data needed for time travel and replay.
Evidence: Table-format docs explicitly tie maintenance to history/time-travel retention.
Affected PRD section or contract: Operational maintenance and retention; Raw/bronze retention and replay; Graph expiry.
Required decision: Add ReplayRetentionPolicy and TableMaintenancePolicy.
Proposed requirement: A maintenance operation must compute ReplayRetentionDecision before deleting any data, delete file, manifest, checkpoint, log, object, or timeline file.
Required mapping table: Maintenance operation -> replay impact -> required refusal condition.
Acceptance criteria: Maintenance refuses when it would invalidate any protected VersionManifest replay window.
Risk if omitted: Silent loss of forensic and replay evidence.
```

### Gap 4: Cross-table consistency is optional but under-specified

```text
Gap: Cadastre has many tables that can form one logical run state, but the PRD does not require a table-set snapshot identity when multiple tables are consumed together.
Why it matters: A graph rebuild may read gold facts at one snapshot and identity decisions at another.
Evidence: Nessie provides catalog-level visibility and cross-table transaction patterns.
Affected PRD section or contract: VersionManifest, replay, graph rebuild.
Required decision: Add CrossTableCommitProfile for deployments that require atomic table-set visibility.
Proposed requirement: A run that declares cross_table_consistency = required must record table_set_ref and table_set_snapshot_checksum before production output.
Required mapping table: consistency mode -> allowed outputs and failure behavior.
Acceptance criteria: Rebuild rejects mixed snapshot sets when cross_table_consistency = required.
Risk if omitted: Graph read model can represent a state that never existed in the authoritative lakehouse.
```

### Gap 5: Graph rebuild evidence is not yet a first-class artifact

```text
Gap: PRD defines GraphDeltaSet and GraphApplyResult, but not a manifest for rebuilding the entire graph read model from authoritative snapshots.
Why it matters: A derived graph must be provably aligned to a lakehouse state after index loss, backend migration, or corruption.
Evidence: DataHub restores graph/search from persisted metadata aspects; Cadastre already forbids graph-backend authoritative mutation.
Affected PRD section or contract: GraphReadModelSchemaProfile, GraphApplyProfile, disaster recovery.
Required decision: Add GraphRebuildManifest and GraphIndexConsistencyCheck.
Proposed requirement: Every production graph rebuild must record authoritative input snapshot refs, projection profile refs, schema profile refs, rebuild algorithm version, output object counts, checksum, drift checks, and error records.
Required mapping table: rebuild mode -> required inputs -> acceptable result states.
Acceptance criteria: Rebuild is accepted only when graph consistency checks pass or the previous graph remains authoritative for serving.
Risk if omitted: Graph drift and backend repair become unverifiable.
```

### Gap 6: Derived-view lag should be explicit

```text
Gap: Operational health includes projection lag, but no bounded DerivedViewLagPolicy defines allowed staleness and failure semantics.
Why it matters: Derived graph/search/index reads can lag lakehouse state without being incorrect if the lag is declared and observable.
Evidence: DataHub uses asynchronous MCL/index update patterns; Cadastre graph apply may fail or lag.
Affected PRD section or contract: Operational health, graph serving API, graph apply.
Required decision: Add DerivedViewLagPolicy.
Proposed requirement: Every graph query response must include or be governed by a derived_view_state whose lakehouse_snapshot_ref, graph_apply_result_id, max_lag_seconds, and staleness state are known.
Required mapping table: lag state -> query behavior.
Acceptance criteria: Query responses exceeding max_lag_seconds return DERIVED_VIEW_LAG_ERROR or a declared stale read flag.
Risk if omitted: Users cannot distinguish stale graph reads from current facts.
```

### Gap 7: OpenLineage mapping should be bounded

```text
Gap: The PRD names lineage and version manifest concepts but not a mapping from pipeline-lineage facets to Cadastre records.
Why it matters: OpenLineage facets are attractive but can be overused as evidence, completeness, or authority.
Evidence: OpenLineage facets are custom metadata attached to jobs, runs, and datasets; Cadastre requires separate evidence and completeness semantics.
Affected PRD section or contract: SourceStageDAG, VersionManifest, evidence refs.
Required decision: Add LineageFacetMappingPolicy and RunDatasetIOContract.
Proposed requirement: Each accepted facet must map to exactly one Cadastre lineage field, diagnostic field, or ignored field; facets default to ignored.
Required mapping table: facet -> Cadastre field -> authority class -> forbidden uses.
Acceptance criteria: An unmapped custom facet in production lineage output is rejected or stored as non-authoritative metadata only.
Risk if omitted: Pipeline lineage leaks into fact authority.
```

### Gap 8: Source completeness needs table-format maintenance and table-read evidence rows

```text
Gap: SourceCompletenessProfile is strong but focused on source collection; lakehouse reads and maintenance decisions also affect absence, cleanup, and replay.
Why it matters: A complete source collection is insufficient if the table snapshot read for gold derivation is missing delete files, manifests, or retained data.
Evidence: Iceberg/Delta/Hudi table state is reconstructable only when required metadata and data files remain available.
Affected PRD section or contract: SourceCompletenessProfile, CoverageAssertion, replay.
Required decision: Add evidence classes for lakehouse_snapshot_resolved, metadata_file_resolved, transaction_log_range_resolved, delete_file_resolved, retention_window_protected.
Proposed requirement: Completeness-sensitive derivation must require both source completeness and table snapshot completeness when using historical table reads.
Required mapping table: table format -> required read completeness evidence.
Acceptance criteria: A gold derivation read missing required table metadata emits SOURCE_COMPLETENESS_ERROR or LAKEHOUSE_SNAPSHOT_REF_ERROR and cannot emit absence.
Risk if omitted: Absence can be derived from incomplete lakehouse state.
```

### Gap 9: Schema compatibility needs table-format mappings

```text
Gap: PRD defines many schema refs, but table-format schema evolution must be tied to Cadastre fact meaning.
Why it matters: Iceberg field IDs and Delta column mapping/protocol features can preserve or change column interpretation across versions.
Evidence: Iceberg table metadata carries schema IDs and last column ID; Delta protocol and column mapping govern reader behavior.
Affected PRD section or contract: Silver replay, gold fact table storage, external schema artifacts.
Required decision: Add LakehouseSchemaCompatibilityCheck.
Proposed requirement: Before replay, rollback, graph rebuild, or profile activation, Cadastre must verify table-format schema identity and Cadastre contract version compatibility.
Required mapping table: table format -> schema identity fields -> incompatible change conditions.
Acceptance criteria: Replay with incompatible schema ID/field mapping fails before output.
Risk if omitted: Replayed facts can change meaning while IDs remain stable.
```

### Gap 10: dbt-like artifacts should inform but not define Cadastre run artifacts

```text
Gap: Cadastre needs clearer separation between static DAG, executed run results, source freshness, and semantic artifacts.
Why it matters: dbt shows that manifests, run results, sources freshness, and semantic manifests answer different questions.
Evidence: dbt docs define separate `manifest.json`, `run_results.json`, `sources.json`, and `semantic_manifest.json` artifacts.
Affected PRD section or contract: SourceStageDAG, ValidationScenario, MappingProjectManifest, CanonicalValidationOutput.
Required decision: Add artifact class boundaries to mapping and execution validation.
Proposed requirement: A production run must not use a freshness artifact as a completeness receipt or a static DAG artifact as executed-run proof.
Required mapping table: artifact class -> allowed Cadastre use -> forbidden use.
Acceptance criteria: Validation rejects artifact substitution with DAG_ARTIFACT_CLASS_ERROR.
Risk if omitted: Validation can pass with the wrong artifact type.
```

## 9. Special focus questions

### 9.1 Lakehouse system of record

Iceberg maps most directly to Cadastre’s `VersionManifest` and gold/silver table reproducibility needs: table metadata file, snapshot ID, manifest list, manifests, schema ID, partition spec ID, sort order ID, delete files, and catalog pointer must be available as `LakehouseSnapshotRef` fields. Delta maps through table version, `_delta_log` range, checkpoint refs, protocol action, table features, add/remove/deletion-vector actions, and optional app `txn` action. Hudi maps through timeline instant, action, state, commit/delta-commit/replace-commit, savepoint, rollback, restore, cleaner, and incremental-read begin/end instants. Nessie maps through branch/tag/commit, content key, content ID, and table-format on-reference state. lakeFS maps through repository, branch/tag/commit, object version set, and GC policy.

Mandatory table-format semantics for Cadastre correctness are snapshot identity, commit identity, schema identity, delete/retraction representation, reader isolation, retention eligibility, and reproducible reconstruction of table contents. Implementation choices include whether the deployment uses Iceberg, Delta, Hudi, Nessie, lakeFS, or another table/catalog format. `VersionManifest` must include `LakehouseSnapshotRef` rows for every authoritative input table and `LakehouseCommitRef` rows for every output table write. For table-set atomicity, it must include a `CrossTableCommitProfile` table-set ref when required.

Retention, snapshot expiration, vacuum, cleaner, orphan deletion, and GC must make replay eligibility a visible state. A protected replay window must prevent physical deletion of data, metadata, delete files, checkpoints, transaction logs, or object versions that a referenced `VersionManifest` needs. Table-format schema evolution must be handled by mapping table-format schema IDs/field IDs/protocol features to Cadastre schema-contract versions; replay must fail when schema identity changes alter fact meaning.

### 9.2 Derived graph architecture

DataHub treats persisted metadata aspects as authoritative before graph/search indexes are updated. OpenMetadata treats its entity store as authoritative before search/index/event delivery. Cartography and JupiterOne/Starbase are cautionary because graph writes or graph sync are primary outputs. Cadastre must specify that authoritative state is raw/silver/identity/gold lakehouse state, and that graph mutation occurs only from persisted, validated `GraphDeltaSet`.

Cadastre should specify graph delta ordering exactly as the PRD already does: edge retractions, edge expirations, node retractions, node expirations, node upserts, edge upserts, noops, with lexical ID ordering inside each group.[^5] It should add graph rebuild evidence: input lakehouse snapshot refs, projection profile, schema profile, graph apply profile, expected index/constraint fingerprint, output node/edge counts, lineage coverage count, no-op count, checksum, drift result, and failure records.

### 9.3 Lineage and run modeling

OpenLineage `Run` maps to Cadastre execution instances only when wrapped in `RunDatasetIOContract`. `Job` maps to declared stage/job identity, not source adapter code by itself. `Dataset` maps to a declared dataset ref and must include table-format `DatasetVersionRef` when it affects replay. Parent-run facets map to source-stage parentage or orchestration lineage, not proof of source completeness.

Directly useful OpenLineage facets include parent run, source code, source code location, job ownership, schema, dataset version, and error-message facets. Cadastre-specific analogous records are required for source completeness, coverage assertion, evidence roles, source authority, identity decisions, graph projection lineage, table snapshot refs, table commit refs, and graph rebuild manifests. OpenLineage is too narrow wherever Cadastre needs cyber asset intelligence semantics: source authority, weak identity non-merge rules, bitemporal correction, negative evidence, graph edge non-implication rules, and raw-evidence drillback.

### 9.4 Ingestion, completeness, and replay

The sources detect completeness in different ways. dbt `sources.json` records freshness checks but not source coverage. Hudi detects failed writes through timeline state and heartbeats. Delta and Iceberg define commit/snapshot consistency but not source API page completeness. Cartography’s cleanup hazards show that stale cleanup is unsafe under partial visibility. Cadastre should adopt page exhaustion, expected count, scope enumeration, permission visibility, report-file-set completeness, table snapshot resolved, log-range resolved, and retention-protected evidence classes.

Cadastre should reject patterns that treat absence unsafely: missing graph node means deleted, missing source record means absent, freshness success means complete, event receipt means complete, search index no-hit means fact absence, or graph sync diff means authoritative deletion. Source completeness must interact with table snapshots and graph expiry: absence/retraction/cleanup/watermark advancement requires both source completeness authorization and a resolvable retained table snapshot for the relevant window.

### 9.5 Schema and compatibility

Iceberg encodes schema evolution through schema IDs and field IDs. Delta encodes protocol and table features, metadata schemas, column mapping, type widening, and reader/writer requirements. OCSF uses pinned schema artifacts and compiled profiles. OpenMetadata uses JSON schemas and generated classes. dbt artifacts use artifact schema versions.

Cadastre should specify schema evolution for `RawRecord`, `CadastreSilverObservation`, `GoldFact`, `GraphDeltaSet`, and external projections through compatibility checks that run before replay, rollback, graph rebuild, profile activation, and source package activation. Required checks include table-format schema identity, Cadastre contract version, external schema artifact checksum, graph projection profile compatibility, graph read-model schema fingerprint, mapping compiler output checksum, and rule graph compatibility.

## 10. Required PRD improvement output

### 10.1 Ranked finding list

| Rank | Finding | Correctness | Replayability | Source-of-truth clarity | Graph determinism | Complexity | Risk reduction |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | Add `LakehouseSnapshotRef` / `LakehouseCommitRef` | 5 | 5 | 5 | 3 | 3 | 5 |
| 2 | Add `ReplayRetentionPolicy` / `TableMaintenancePolicy` | 5 | 5 | 4 | 3 | 3 | 5 |
| 3 | Add `GraphRebuildManifest` / `GraphIndexConsistencyCheck` | 4 | 5 | 5 | 5 | 3 | 5 |
| 4 | Add `CrossTableCommitProfile` when table-set consistency is required | 4 | 5 | 4 | 4 | 4 | 4 |
| 5 | Add `LineageFacetMappingPolicy` / `RunDatasetIOContract` | 4 | 4 | 4 | 2 | 3 | 4 |
| 6 | Add `DerivedViewLagPolicy` | 3 | 3 | 3 | 4 | 2 | 4 |
| 7 | Extend `SourceCompletenessProfile` with table-read and maintenance evidence | 5 | 4 | 4 | 4 | 3 | 5 |
| 8 | Add `LakehouseSchemaCompatibilityCheck` | 5 | 5 | 4 | 3 | 3 | 4 |

### 10.2 PRD revision map

| Finding | PRD section or contract | Required change | Adoption stance | Acceptance criterion |
| --- | --- | --- | --- | --- |
| Table snapshot refs | `VersionManifest`, Data Model | Add `LakehouseSnapshotRef`, `LakehouseCommitRef`, `DatasetVersionRef`. | `adopt` | Replay fails before output if any required ref is absent or unresolved. |
| Retention and maintenance | Operational maintenance, replay | Add `ReplayRetentionPolicy`, `TableMaintenancePolicy`, `ReplayRetentionDecision`. | `adopt` | Maintenance refuses deletion that invalidates protected replay windows. |
| Graph rebuild evidence | Graph projection/apply | Add `GraphRebuildManifest`, `GraphIndexConsistencyCheck`. | `adopt` | Rebuild output checksum matches expected or graph remains unavailable/stale. |
| Cross-table versioning | `VersionManifest`, staging/promotion | Add optional `CrossTableCommitProfile`. | `adapt` | Mixed snapshots are rejected when consistency required. |
| Catalog branch promotion | Lifecycle/staging | Add optional `CatalogBranchPromotionPolicy`. | `adapt` | Branch/tag promotion cannot become active without validation result and table-set checksum. |
| OpenLineage mapping | Lineage/evidence | Add `LineageFacetMappingPolicy`, `RunDatasetIOContract`. | `adapt` | Unmapped production facet cannot affect facts/completeness/identity. |
| Derived view lag | Graph serving health/API | Add `DerivedViewLagPolicy`. | `adopt` | Queries beyond allowed lag return declared stale/error behavior. |
| Table schema compatibility | Schema/replay | Add `LakehouseSchemaCompatibilityCheck`. | `adopt` | Replay/rollback/rebuild rejects incompatible schema identities. |

### 10.3 Proposed contract additions

#### 10.3.1 `LakehouseTableProfile`

```text
Contract name: LakehouseTableProfile
Purpose: Defines table-format-specific observable identifiers, supported operations, compatibility requirements, retention hooks, and replay support for one authoritative Cadastre table.
Inputs: table_identifier, table_format, catalog_kind, deployment_profile, supported_operation_set.
Outputs: validated table profile and checksum.
Required fields: table_profile_id, table_name, cadastre_layer, table_format, catalog_kind, required_snapshot_ref_fields, required_commit_ref_fields, required_schema_identity_fields, supported_maintenance_operations, replay_retention_policy_id, checksum.
Defaults: supported_maintenance_operations = []; replay_required = true for raw, silver, identity, gold, graph_delta tables; false only for explicitly non-authoritative health scratch tables.
Bounds: table_name maximum 128 characters; required fields arrays must be non-empty for authoritative tables.
Errors: LAKEHOUSE_TABLE_PROFILE_ERROR, LAKEHOUSE_FORMAT_UNSUPPORTED.
Deterministic algorithm: validate profile rows by table_format mapping; reject missing required identifier fields; compute canonical checksum from sorted fields.
Mapping tables: table_format -> required snapshot fields; table_format -> required commit fields.
Acceptance criteria: Every authoritative table referenced by VersionManifest has exactly one active LakehouseTableProfile.
```

#### 10.3.2 `LakehouseSnapshotRef`

```text
Contract name: LakehouseSnapshotRef
Purpose: Records exact table state read by a run, replay, graph rebuild, or validation scenario.
Inputs: table profile, runtime table metadata, catalog reference, read timestamp.
Outputs: immutable snapshot reference row.
Required fields: snapshot_ref_id, table_profile_id, table_format, table_identifier, snapshot_identity, metadata_ref, schema_identity, partition_identity, delete_or_tombstone_identity, catalog_ref, retention_status, resolved_at, checksum.
Defaults: retention_status = unknown until ReplayRetentionPolicy evaluates it.
Bounds: one row per authoritative table read; checksum is SHA-256 over canonical JSON.
Errors: LAKEHOUSE_SNAPSHOT_REF_ERROR, LAKEHOUSE_SCHEMA_COMPATIBILITY_ERROR.
Deterministic algorithm: collect format-specific fields, sort nested refs lexically, compute checksum, validate required fields.
Mapping tables: Iceberg, Delta, Hudi, Nessie, lakeFS, other_declared snapshot fields.
Acceptance criteria: Two readers resolving the same table state produce identical snapshot_ref checksum.
```

#### 10.3.3 `LakehouseCommitRef`

```text
Contract name: LakehouseCommitRef
Purpose: Records attempted and completed table writes, including table-format commit identifiers and idempotence metadata.
Inputs: table profile, write operation, run ID, writer identity, table-format commit response.
Outputs: immutable commit ref row.
Required fields: commit_ref_id, table_profile_id, operation, attempted_at, completed_at, status, commit_identity, previous_snapshot_ref_id, new_snapshot_ref_id, idempotence_key, error_refs, checksum.
Defaults: status = attempted until completed, failed, rolled_back, or restored.
Bounds: operation one of append, overwrite, delete, merge, maintenance, rollback, restore, compact, expire, vacuum, clean, gc.
Errors: LAKEHOUSE_COMMIT_REF_ERROR.
Deterministic algorithm: validate operation against table profile; bind previous/new snapshot refs; compute checksum.
Mapping tables: table format -> commit identity fields.
Acceptance criteria: Any production table write without a succeeded or failed LakehouseCommitRef fails stage completion.
```

#### 10.3.4 `TableMaintenancePolicy`

```text
Contract name: TableMaintenancePolicy
Purpose: Controls snapshot expiration, vacuum, cleaner, compaction, orphan-file deletion, restore, rollback, and object GC.
Inputs: table profile, maintenance request, protected VersionManifest set, ReplayRetentionPolicy.
Outputs: maintenance decision and allowed operation parameters.
Required fields: policy_id, table_format, allowed_operations, default_retention_days, protected_layers, preflight_required, refusal_conditions, checksum.
Defaults: preflight_required = true; destructive maintenance forbidden until policy active.
Bounds: default_retention_days minimum 0 only for non-authoritative scratch tables; authoritative tables require deployment-defined minimum.
Errors: TABLE_MAINTENANCE_POLICY_ERROR, REPLAY_RETENTION_VIOLATION.
Deterministic algorithm: evaluate requested deletion candidates against protected snapshot refs and replay windows; allow only safe candidates.
Mapping tables: operation -> required preflight evidence -> allowed/refused behavior.
Acceptance criteria: Maintenance refuses to delete any file/log/object required by an active replay window.
```

#### 10.3.5 `ReplayRetentionPolicy`

```text
Contract name: ReplayRetentionPolicy
Purpose: Defines which historical raw payloads, table snapshots, transaction logs, manifests, delete files, graph deltas, graph apply results, and lineage records must remain available for replay.
Inputs: deployment profile, table profiles, run manifests, regulatory retention rules.
Outputs: replay retention decisions.
Required fields: policy_id, protected_run_scope, protected_time_window, protected_layers, minimum_table_history, evidence_retention_mode, raw_payload_retention_mode, exception_rows, checksum.
Defaults: protect all production VersionManifest refs for the declared retention window.
Bounds: protected_time_window must be explicit; unbounded allowed only when deployment declares indefinite retention.
Errors: REPLAY_RETENTION_POLICY_ERROR, REPLAY_RETENTION_VIOLATION.
Deterministic algorithm: enumerate all protected manifest refs; mark each maintenance candidate allowed or forbidden.
Mapping tables: layer -> required retained artifacts.
Acceptance criteria: Replay eligibility can be answered as yes/no for every historical run.
```

#### 10.3.6 `GraphRebuildManifest`

```text
Contract name: GraphRebuildManifest
Purpose: Records a full or partial graph read-model rebuild from authoritative lakehouse inputs.
Inputs: LakehouseSnapshotRef set, GraphProjectionProfile, GraphReadModelSchemaProfile, GraphApplyProfile, rebuild mode.
Outputs: rebuild manifest and consistency checks.
Required fields: rebuild_manifest_id, rebuild_run_id, rebuild_mode, input_snapshot_refs, graph_projection_profile_id, graph_schema_profile_id, graph_apply_profile_id, projection_algorithm_version, expected_counts, output_counts, output_checksum, graph_index_consistency_check_ids, status, errors.
Defaults: rebuild_mode = full for disaster recovery; partial requires explicit scope.
Bounds: input_snapshot_refs must include gold, identity, graph profile registry, and required silver/raw metadata refs.
Errors: GRAPH_REBUILD_ERROR, GRAPH_INDEX_CONSISTENCY_ERROR.
Deterministic algorithm: resolve snapshot refs; project full graph delta set; apply to empty or declared target; compute canonical node/edge/property/evidence checksum.
Mapping tables: rebuild_mode -> required inputs and allowed status.
Acceptance criteria: A rebuild from the same inputs produces the same checksum or the same declared failure.
```

#### 10.3.7 `GraphIndexConsistencyCheck`

```text
Contract name: GraphIndexConsistencyCheck
Purpose: Proves graph read-model indexes and constraints match the graph schema profile and authoritative delta state.
Inputs: graph backend schema fingerprint, GraphReadModelSchemaProfile, GraphRebuildManifest or GraphApplyResult.
Outputs: consistency check row.
Required fields: check_id, graph_backend_kind, schema_profile_id, backend_schema_fingerprint, expected_schema_fingerprint, node_count_hash, edge_count_hash, evidence_index_hash, status, errors.
Defaults: status = failed until every required check passes.
Bounds: Must run before production serving after rebuild and before graph apply when schema changed.
Errors: GRAPH_INDEX_CONSISTENCY_ERROR, GRAPH_READ_MODEL_SCHEMA_ERROR.
Deterministic algorithm: compare backend schema, required indexes, counts, and evidence references to expected profile values.
Mapping tables: backend_kind -> required index/constraint checks.
Acceptance criteria: Graph serving remains unavailable or stale when consistency status is not passed.
```

#### 10.3.8 `DerivedViewLagPolicy`

```text
Contract name: DerivedViewLagPolicy
Purpose: Defines allowed lag between authoritative lakehouse state and graph/search/read-model state.
Inputs: authoritative snapshot refs, latest graph apply result, query request, deployment profile.
Outputs: derived view state and query behavior.
Required fields: policy_id, view_kind, max_lag_seconds, stale_read_allowed, stale_label_required, fail_behavior, checksum.
Defaults: stale_read_allowed = true for analyst exploratory graph queries with visible stale label; false for compliance export unless deployment says otherwise.
Bounds: max_lag_seconds minimum 0, maximum deployment-defined.
Errors: DERIVED_VIEW_LAG_ERROR.
Deterministic algorithm: compute lag from latest authoritative snapshot/update time to latest successful apply; choose behavior from table.
Mapping tables: lag_state -> allowed query response.
Acceptance criteria: Query response identifies stale read or fails when lag exceeds policy.
```

#### 10.3.9 `LineageFacetMappingPolicy`

```text
Contract name: LineageFacetMappingPolicy
Purpose: Maps OpenLineage standard/custom facets or analogous records to Cadastre lineage metadata without granting fact authority.
Inputs: lineage event, facet schemas, SourceStageDAG context, table snapshot refs.
Outputs: accepted lineage record or diagnostic.
Required fields: policy_id, facet_rows, default_unmapped_behavior, forbidden_authority_uses, checksum.
Defaults: default_unmapped_behavior = store_non_authoritative_metadata in non-production; reject in production when facet affects output.
Bounds: Every production-affecting facet must have exactly one mapping row.
Errors: LINEAGE_FACET_MAPPING_ERROR.
Deterministic algorithm: validate schemaURL, facet name, namespace; map fields; enforce forbidden uses.
Mapping tables: facet -> Cadastre field -> authority class -> forbidden uses.
Acceptance criteria: A custom facet cannot affect source completeness, identity, gold facts, or graph projection unless an explicit Cadastre contract permits it.
```

#### 10.3.10 `DatasetVersionRef`

```text
Contract name: DatasetVersionRef
Purpose: Gives a dataset-level version identity that may compose table-format snapshot refs, artifact checksums, and external lineage dataset versions.
Inputs: dataset identifier, layer, table snapshot refs or artifact refs.
Outputs: dataset version row.
Required fields: dataset_version_ref_id, dataset_id, layer, version_kind, table_snapshot_ref_ids, artifact_checksums, external_dataset_version, checksum.
Defaults: version_kind = table_snapshot_set for Cadastre authoritative tables.
Bounds: authoritative dataset version refs must include at least one table snapshot ref.
Errors: DATASET_VERSION_REF_ERROR.
Deterministic algorithm: sort refs, compute checksum.
Mapping tables: layer -> required version source.
Acceptance criteria: Every RunDatasetIO row names a DatasetVersionRef when dataset state affects replay.
```

#### 10.3.11 `RunDatasetIOContract`

```text
Contract name: RunDatasetIOContract
Purpose: Records input and output datasets for a Cadastre run or source stage, separated from evidence authority.
Inputs: stage run, dataset refs, OpenLineage-like event fields, table refs.
Outputs: IO rows and optional external lineage projection.
Required fields: run_dataset_io_id, run_id, stage_id, io_role, dataset_version_ref_id, lineage_event_ref, evidence_role, authority_class, checksum.
Defaults: authority_class = lineage_only.
Bounds: io_role one of input, output, read_only_context, validation_input, shadow_output.
Errors: RUN_DATASET_IO_ERROR.
Deterministic algorithm: resolve datasets, validate version refs, bind IO to stage and manifest.
Mapping tables: io_role -> allowed downstream uses.
Acceptance criteria: An IO record with authority_class = lineage_only cannot satisfy SourceCompletenessReceipt or GoldFact evidence requirements.
```

#### 10.3.12 `CrossTableCommitProfile`

```text
Contract name: CrossTableCommitProfile
Purpose: Defines when multiple table commits or reads must be treated as one atomic table-set version.
Inputs: table set, catalog refs, commit refs, snapshot refs, deployment consistency mode.
Outputs: table_set_ref and checksum.
Required fields: profile_id, consistency_mode, required_tables, catalog_kind, branch_or_tag_policy, atomicity_requirement, table_set_checksum, checksum.
Defaults: consistency_mode = not_required unless graph rebuild, gold derivation, or deployment profile declares table-set consistency required.
Bounds: required_tables must be non-empty when consistency_mode != not_required.
Errors: CROSS_TABLE_COMMIT_ERROR.
Deterministic algorithm: validate all required tables share one catalog commit/ref or compute sorted snapshot set; reject missing tables.
Mapping tables: consistency_mode -> required catalog/table refs.
Acceptance criteria: A required table-set read cannot mix snapshots outside the declared table set.
```

#### 10.3.13 `CatalogBranchPromotionPolicy`

```text
Contract name: CatalogBranchPromotionPolicy
Purpose: Governs staging, validation, promotion, and rollback when catalog branches/tags are used.
Inputs: source branch, target branch, validation manifests, table-set checksum, lifecycle machine.
Outputs: promotion decision.
Required fields: policy_id, catalog_kind, source_ref, target_ref, required_validation_results, required_table_set_checksum, rollback_ref_behavior, checksum.
Defaults: promotion forbidden without validation result.
Bounds: mutable branch names cannot be production refs unless paired with immutable commit/hash.
Errors: CATALOG_BRANCH_PROMOTION_ERROR.
Deterministic algorithm: compare source and target refs, validate table-set checksum, require lifecycle transition evidence.
Mapping tables: event -> allowed transition -> required evidence.
Acceptance criteria: Branch merge/promotion cannot make production visible without a passing validation manifest.
```

### 10.4 Recommended follow-up research reports

| Follow-up report | Justification |
| --- | --- |
| Lakehouse table-format selection for Cadastre | This report defines transferable contracts but does not select Iceberg, Delta, Hudi, Nessie, or lakeFS. Selection requires deployment constraints, engines, object store, query patterns, and operational team requirements. |
| Graph database apply engines and deterministic graph delta materialization | The PRD must remain graph-backend-neutral, but implementation requires comparing Neo4j, Memgraph, JanusGraph, TigerGraph, and possibly relational/Elasticsearch graph-serving approaches against `GraphApplyProfile`. |
| OpenLineage-to-Cadastre lineage contract design | Needed to produce exact facet mappings, schema URLs, and external lineage projections without conflating lineage and evidence. |
| Catalog-level branching and promotion workflows | Needed if Cadastre wants Nessie or lakeFS-style staging, write-audit-publish, or table-set promotion. |
| Incremental view maintenance for graph and search projections | Needed to define algorithms that prove incremental updates equal full recomputation. |
| Retention, vacuum, snapshot expiration, and replay eligibility | Needed to turn retention requirements into operational policy for selected storage and catalog tools. |

## 11. Evidence discipline notes

This report separates project artifacts, prior Cadastre research reports, official external docs/specs, and web release pages. It does not merge them as if they shared authority. The Cadastre PRD is the target artifact. External sources are evidence for transferable patterns or hazards. Prior research reports are context and boundary evidence, not fresh repository verification. Web release pages establish freshness where accessible; no external repository was cloned or built during this report.

## 12. Acceptance criteria self-check

| ID | Result |
| --- | --- |
| AC-001 | Pass. DataHub, OpenLineage, Iceberg, and Delta are independently reconstructed before comparison. |
| AC-002 | Pass. Marquez, Hudi, Nessie, OpenMetadata, dbt, and lakeFS are analyzed to targeted depth. |
| AC-003 | Pass. Source inventory includes inspected versions/releases where available, evidence, reliability, and limits. |
| AC-004 | Pass. All four required comparison matrices are included. |
| AC-005 | Pass. High-value findings map to PRD contracts or revision areas. |
| AC-006 | Pass. PRD gap proposals include requirements and binary acceptance criteria. |
| AC-007 | Pass. Storage/table, metadata graph, lineage, ingestion/completeness, and graph projection semantics are separated. |
| AC-008 | Pass. Concepts not to transfer are included for every source. |
| AC-009 | Pass. No graph-authoritative or direct-to-graph recommendation is made for Cadastre. |
| AC-010 | Pass. No external source is treated as Cadastre canonical domain model. |
| AC-011 | Pass. Retention, expiration, vacuum, cleaning, rollback, restore, and replay consequences are addressed. |
| AC-012 | Pass. Follow-up reports are justified by unresolved decisions or scope limits. |
| AC-013 | Pass. Material claims are cited to source evidence. |
| AC-014 | Pass. Final synthesis includes ranked findings, revision map, proposed changes, risks, unresolved decisions, and implementation-guidance-only concepts. |

## Sources

[^1]: `PRD-Cadastre.md`, product architecture and document status, lines 25-52.
[^2]: `PRD-Cadastre.md`, scope and non-scope requirements, lines 132-168; `VersionManifest`, lines 1531-1593.
[^4]: `PRD-Cadastre.md`, `SourceCompletenessReceipt` and `SourceCompletenessProfile`, lines 1908-2024 and 2889-3001.
[^5]: `PRD-Cadastre.md`, `GraphProjectionProfile`, `GraphDeltaSet`, `GraphApplyProfile`, and graph apply interface, lines 2172-2351, 4025-4103, and 4803-4828.
[^7]: `nlspec-spec.md`, NLSpec definition and required properties, lines 13-72; structure guidance, lines 184-205.
[^8]: `RES-001-cartography-research-report.md`, executive architecture and cleanup/freshness analysis. Evidence is from the uploaded prior report; original repository was not re-cloned in this report.
[^9]: `RES-002-jupiterone-research-report.md`, executive summary and central architectural contrast. Evidence is from the uploaded prior report; public/private JupiterOne implementation limits remain as stated there.
[^12]: DataHub documentation, Metadata Model, lines 102-178 from opened web source `turn704266view0`; DataHub aspect documentation, lines 14-20 from `turn704266view4`.
[^13]: DataHub documentation, MCP/MCL event stream, lines 123-251 from `turn704266view2`; DataHub MCL event docs, lines 121-145 from `turn704266view3`.
[^14]: DataHub documentation, Restore Indices and aspect retention/versioning, lines 116-159 from `turn229355view0`, lines 118-139 from `turn229355view1`, and lines 117-132 from `turn229355view3`.
[^15]: DataHub ingestion documentation, lines 122-134 from `turn229355view4`, and recipe docs lines 107-137 from `turn229355view5`.
[^16]: OpenLineage specification and API docs, lines 252-335 from `turn316332view0`, lines 56-83 from `turn316332view1`, and lines 6-53 from `turn316332view5`.
[^17]: OpenLineage parent/run facets, lines 67-71 from `turn316332view3` and lines 67-70 from `turn316332view4`.
[^18]: Apache Iceberg table specification, lines 1501-1514 and 2163-2297 from `turn718741view0`.
[^19]: Apache Iceberg table specification and maintenance docs, lines 491-517 and 974-1003 from `turn718741view1`; Iceberg maintenance/search results on snapshot expiration and time travel from `turn372194search0`, `turn372194search19`.
[^20]: Delta Lake protocol, lines 364-376, 379, 786-794, 947-1028 from `turn221175view7`, `turn851942view4`, and `turn851942view5`.
[^21]: Delta Lake protocol action/checkpoint details, lines 637-704 and 1840-1941 from `turn851942view1`, `turn851942view2`, and `turn851942view3`; `txn` and protocol evolution details from `turn490359view1`, lines 3276-3320.
[^22]: Delta Lake utility docs, `vacuum` and time-travel retention, lines 104-105 from `turn221175view0`.
[^23]: Apache Hudi docs, timeline actions and states, lines 116-150 from `turn514277view0`; rollback, lines 98-115 from `turn514277view2`; cleaning, lines 98-116 from `turn514277view3`; querying/incremental queries, lines 73-101 from `turn514277view5`; savepoint/restore, lines 100-104 from `turn514277view1`.
[^24]: Project Nessie docs, transactions and isolation, lines 624-642 from `turn221175view1`; Nessie spec content key/content ID/on-reference state, lines 640-703 from `turn221175view4`; Iceberg Nessie integration, lines 1302-1308 from `turn221175view5`.
[^25]: OpenMetadata docs and repository documentation, high-level design lines 177-205 from `turn205568view0`; knowledge graph lines 422 and 648-688 from `turn470846view4`; lineage API lines 152-183 from `turn221175view3`; change events and webhooks lines 86-88 from `turn205568view1` and `turn205568view2`; release page lines 61-76 from `turn205568view3`.
[^26]: Marquez project docs and repository README, project description lines 16-29 from `turn494755view0`, README lines 456-463 from `turn494755view1`, lineage graph API lines 47-61 and 82-143 from `turn494755view2`, and Marquez API blog lines 30-39 from `turn494755view4`.
[^27]: dbt artifact docs, `manifest.json` lines 94-116 from `turn494755view7`; `run_results.json` lines 84-87 from `turn494755view5`; `sources.json` lines 79-97 from `turn494755view6`; `semantic_manifest.json` lines 79-90 from `turn470846view1` and deployment use lines 543-581 from `turn470846view2`.
[^28]: lakeFS docs, Git-like data versioning and zero-copy/metadata layer, lines 198-236 from `turn494755view8`; garbage collection rules and retention, lines 203-225 from `turn470846view0`.
[^29]: External release pages inspected on 2026-05-15: DataHub releases lines 172-213 from `turn227903view0`; OpenLineage releases lines 178-201 and 882-889 from `turn227903view1`; Iceberg releases lines 172-186 and 296-304 from `turn227903view2`; Delta releases from `turn227903view3`; OpenMetadata releases lines 59-76 from `turn205568view3`. Release pages were used for freshness context only and were not treated as implementation behavior evidence.
