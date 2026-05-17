---
doc_id: RES-011
project: Cadastre
title: Temporal Semantics, Corrections, Replay, and Deterministic Rebuild Cadastre Research Report
status: research-report
inspection_date: 2026-05-16
related_docs:
  - RES-006
  - RES-008
---

## Source limits summary

This report uses `PRD-Cadastre.md` as the governing Cadastre baseline. The uploaded Cadastre research reports are used as evidence and hypothesis material only. Public external sources were inspected through official documentation, specifications, release pages, public repository pages, and source snippets reachable through web inspection. No external repository was cloned in this run. No external system was built, installed, executed, benchmarked, or tested. No source connector, CDC stream, lakehouse table, workflow, graph backend, projection engine, or metadata platform was run locally. Runtime behavior claims are therefore limited to inspected source material and labeled inferences.

The strongest Cadastre conclusion is stable: Cadastre already has the right authority boundary. The lakehouse must remain the system of record, graph and CIM outputs must remain replaceable projections, and every production-affecting table, replay, correction, and graph-serving decision must be tied to Cadastre-owned contracts instead of external defaults or backend state.[^1]

### 1. Executive verdict

| Rank | Finding | Source basis | Cadastre contract area | Transfer stance | Main risk reduced |
| ---: | --- | --- | --- | --- | --- |
| 1 | Add first-class temporal semantics contracts so source event time, known time, table time, connector time, watermark time, replay time, and graph apply time cannot be confused. | XTDB, Datomic, stream-time systems, CDC systems, PRD boundary | `TemporalSemanticsPolicy`, `TemporalObservationTimeResolution`, `GoldFact` | `adopt` | Fact-time corruption and inconsistent corrections |
| 2 | Treat corrections as append-only knowledge transitions, never in-place mutation. | XTDB/Datomic, EventStoreDB/KurrentDB, Differential Dataflow, PRD correction rule | `GoldFactCorrectionPolicy`, `ApplyGoldCorrection` | `adopt` | Lost audit history and irreproducible “as known then” views |
| 3 | Make late-arrival handling authority-aware. Production default must preserve evidence and evaluate correction impact, not silently discard late records. | Flink, Beam, Kafka Streams, PRD completeness/authority model | `LateArrivalPolicy`, `EvaluateLateArrival`, `SourceCompletenessProfile` | `adapt` | Loss of authoritative late evidence and graph drift |
| 4 | Capture CDC offsets, schema history, transaction metadata, tombstones, and heartbeats as replay state and raw evidence only. | Debezium MySQL/PostgreSQL, Kafka Connect, PRD raw-event subtype contract | `CDCReplayStateContract`, `StageStateRecord`, `VersionManifest` | `adopt` | Treating log position, heartbeat, or connector time as fact time or completeness |
| 5 | Define production replay as a preflight-gated output mode with explicit failure precedence. | Temporal.io deterministic replay, Cadence, lakehouse refs, PRD `VersionManifest` | `ReplayEquivalencePolicy`, `ReplayInputSufficiencyCheck` | `adopt` | Silent output drift and manifest substitution |
| 6 | Use event-sourcing projection patterns for graph deltas but keep graph truth in the lakehouse. | EventStoreDB/KurrentDB, DataHub restore-index pattern, PRD graph boundary | `GraphDeltaSet`, `GraphApplyResult`, `ProjectionWatermarkPolicy` | `adapt` | Graph backend becoming de facto authority |
| 7 | Model corrections and graph deltas as deterministic signed effects, not mutable current-state rewrites. | Differential Dataflow, EventStoreDB/KurrentDB | `GoldFactChangeSet`, `GraphDeltaIdempotencyKey` | `adapt` | Non-idempotent graph apply and correction replay divergence |
| 8 | Require table-format-native snapshot and commit refs for every production-affecting read/write, replay, rebuild, or maintenance decision. | Iceberg, Delta Lake, Hudi, Nessie, prior lakehouse report | `LakehouseSnapshotRef`, `LakehouseCommitRef`, `DatasetVersionRef` | `adopt` | Unreproducible table reads and unsafe maintenance |
| 9 | Keep metadata, lineage, dbt freshness, search indexes, and graph indexes as derived or governance artifacts unless Cadastre explicitly maps authority. | DataHub, OpenLineage, OpenMetadata, Marquez, dbt artifacts | `RunDatasetIOContract`, `ArtifactClassPolicy`, `LineageFacetMappingPolicy` | `cautionary_reference` | Artifact substitution and false completeness |
| 10 | Treat graph-adjacent systems as cautionary references for direct-to-graph authority and key-based identity hazards. | Cartography, JupiterOne/Starbase, BloodHound/OpenGraph prior reports | `GraphProjectionProfile`, `GraphTaxonomyTranslationPolicy`, `GraphRebuildManifest` | `cautionary_reference` | Direct graph synchronization, graph-key identity, and graph cleanup leakage |

Source basis for the executive verdict comes from official bitemporal, stream-processing, CDC, workflow, event-store, differential-dataflow, lakehouse, metadata, and graph-adjacent sources inspected for this report, plus the governing PRD and existing Cadastre reports used only as context.[^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21][^22][^23][^24][^25][^26][^27][^28][^29][^30][^31][^32][^33][^34][^35]

### 2. Evidence inventory and source limits

#### 2.1 Evidence classes

| Evidence class | Definition |
| --- | --- |
| `confirmed source fact` | Directly supported by inspected source code, official docs, specs, tests, schemas, examples, or release metadata. |
| `source-grounded inference` | Inferred from multiple confirmed facts in one source family. |
| `Cadastre applicability inference` | Derived by comparing confirmed source facts with the Cadastre PRD. |
| `speculative transfer` | Promising but requiring deeper validation before PRD adoption. |
| `open question` | Requires additional source inspection, execution, operational data, product governance, or domain authority. |

#### 2.2 Source inventory

| Source name | URL or path | Evidence type | Version, tag, commit, or release | Inspection date | Scope inspected | Scope not inspected | Reliability | Freshness risk |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Cadastre PRD | `/mnt/data/PRD-Cadastre.md` | spec | uploaded draft | 2026-05-16 | Product summary, raw/silver/gold, completeness, version manifest, lakehouse refs, graph rebuild/apply contracts | Hidden project docs, implementation repo | High: governing baseline | Medium: draft can change |
| NLSpec standard | `/mnt/data/nlspec-spec.md` | spec | 0.2.2 | 2026-05-16 | Behavioral completeness, interface precision, defaults, mapping tables, acceptance criteria | No project behavior beyond quality standard | High | Low |
| XTDB | Official docs, repo/docs pages | official_docs, spec-like docs | current docs; exact release not pinned | 2026-05-16 | Bitemporality, valid time, system time, documents, transactions | Build, tests, source internals | High for documented model | Medium |
| Datomic | Official docs and release notes | official_docs, release_metadata | Pro release page current; exact runtime not installed | 2026-05-16 | History DB, transaction time, datoms, assertions/retractions, release pages | Runtime, peer/client behavior, tests | High for documented model | Medium |
| Apache Flink | Official docs and release/download pages | official_docs, release_metadata | stable docs inspected; release pages showed active 2.x line | 2026-05-16 | Event time, processing time, watermarks, windows, allowed lateness | Runtime execution and source code | High for docs | Medium-high due release/docs drift |
| Apache Beam | Official programming guide, Javadocs, releases | official_docs, release_metadata | Beam 2.73.0 surfaced in releases | 2026-05-16 | Event time, processing time, watermarks, triggers, late data, allowed lateness | Pipeline execution, runner-specific behavior | High for docs | Medium |
| Kafka Streams | Official Apache Kafka docs and release pages; Confluent docs for explanatory detail | official_docs, secondary_source | Kafka 4.1.x line surfaced | 2026-05-16 | Event time, stream time, grace windows, state stores, late record behavior | Source code, local app execution | Medium-high | Medium |
| Debezium MySQL | Official Debezium docs | official_docs | Debezium 3.5 stable surfaced | 2026-05-16 | Initial snapshots, binlog offsets, schema history, delete/tombstone, transactions | Connector execution, source code | High | Medium |
| Debezium PostgreSQL | Official Debezium docs | official_docs | Debezium 3.5 stable surfaced | 2026-05-16 | Snapshots, WAL/LSN, schema behavior, deletes, tombstones, replica identity caveats | Connector execution, source code | High | Medium |
| Kafka Connect offsets/schema history | Kafka/Debezium docs | official_docs | not pinned | 2026-05-16 | Offset and schema-history persistence concepts | Worker execution, exact storage topics | Medium-high | Medium |
| Temporal.io | Official docs | official_docs | current docs; exact server version not pinned | 2026-05-16 | Workflow history, deterministic replay, side effects, patching/versioning, nondeterminism errors | Server/source/test execution | High | Medium |
| Cadence | Official docs/source docs | official_docs | current docs; exact version not pinned | 2026-05-16 | Workflow replay and side-effect recording comparison | Runtime/source execution | Medium-high | Medium |
| EventStoreDB/KurrentDB | Official docs | official_docs | current docs; exact release not pinned | 2026-05-16 | Append-only streams, expected revision, idempotent event IDs, projections, checkpoints, parked messages | Runtime/source/tests | High | Medium |
| Differential Dataflow | Official docs and repository | official_docs, source_code docs | current docs; exact crate version not pinned | 2026-05-16 | Collections as `(data, time, diff)`, frontiers, compaction, arrangements, incremental computation | Runtime benchmarks/source deep dive | High for model | Medium |
| Apache Iceberg | Table spec, docs, releases | spec, official_docs, release_metadata | 1.10.x line surfaced | 2026-05-16 | Metadata files, snapshots, manifests, delete files, snapshot refs, expiration | Runtime, source code, engine-specific APIs | High | Medium |
| Delta Lake | Protocol spec, docs, release pages | spec, official_docs, release_metadata | 4.2.0 surfaced | 2026-05-16 | Transaction log, versions, actions, protocol features, checkpoints, vacuum | Runtime/source/tests | High | Medium |
| Apache Hudi | Docs, technical specs, releases | official_docs, release_metadata | 1.1.1 surfaced | 2026-05-16 | Timeline instants, commit states, rollback, savepoint, cleaner, restore, incremental query | Runtime/source/tests | High | Medium |
| Project Nessie | Official docs/releases | official_docs, release_metadata | 0.107.x line surfaced | 2026-05-16 | Branch/tag/commit, content keys/IDs, cross-table commits, Iceberg integration | Runtime/source/tests | High | Medium |
| lakeFS | Official docs/repo | official_docs | current docs; exact release not pinned | 2026-05-16 | Object-store commits, branches, GC, rollback comparison | Runtime/source/tests | Medium-high | Medium |
| DataHub | Official docs/repo/release pages | official_docs, source_code docs | current docs; exact release not pinned in this run | 2026-05-16 | Aspects, metadata change proposals/logs, graph/search index restore | Runtime/source/tests | High for docs | Medium |
| OpenLineage | Official spec/docs/repo | spec, official_docs | 1.47.x line surfaced | 2026-05-16 | Run/job/dataset/facet model, event lifecycle, custom facets | Integrations/runtime | High | Medium |
| OpenMetadata | Official docs/repo | official_docs | 1.12.x line surfaced | 2026-05-16 | Entity/version/change events, lineage APIs, search index | Runtime/source/tests | High | Medium |
| Marquez | Official docs/repo | official_docs | exact release unknown | 2026-05-16 | OpenLineage backend, run/job/dataset lineage graph | Runtime/source/tests | Medium-high | Medium |
| dbt artifacts | Official dbt artifact docs | official_docs, schema docs | docs current; exact dbt release not pinned | 2026-05-16 | `manifest.json`, `run_results.json`, `sources.json`, semantic artifacts | dbt execution | High | Medium |
| Cartography | Uploaded RES-001 plus context source family | research_report, source-grounded context | RES-001 inspected 2026-05-15 | 2026-05-16 | Direct-to-Neo4j ingestion, cleanup, graph model hazards | Primary repo not re-inspected in this run | Medium-high | Medium |
| JupiterOne/Starbase | Uploaded RES-002 plus context source family | research_report, source-grounded context | RES-002 inspected 2026-05-15 | 2026-05-16 | SDK graph-object sync, mapped relationships, graph-key hazards | Primary repo not re-inspected in this run | Medium-high | Medium |
| BloodHound/SharpHound/OpenGraph | Uploaded RES-008 plus context source family | research_report, source-grounded context | RES-008 inspected 2026-05-16 | 2026-05-16 | Graph identity, traversability, deletion/source-kind hazards | Primary repo not re-inspected in this run | Medium-high | Medium |

### 3. Cadastre baseline assessment

Cadastre already specifies that the product must create a temporal lakehouse-backed asset intelligence graph and that the lakehouse is the system of record. The graph and Splunk CIM output are replaceable projections, and projection is the deterministic contract from authoritative Cadastre records to serving outputs.[^1]

Cadastre also already forbids several dangerous substitutions: table snapshot time, transaction-log time, Hudi instant time, catalog commit time, object-store commit time, CDC offsets, connector timestamps, schema-history timestamps, CDC heartbeat, stream heartbeat, queue drain, processor success, acknowledgment, provenance closure, live query rows, and graph-backend state must not become Cadastre fact time, completeness, evidence authority, cleanup authority, retraction authority, or graph mutation authority unless a Cadastre contract explicitly permits that use.[^2]

The PRD’s `RawRecord` contract already distinguishes source event subtypes, including API rows/pages, report-file records, FlowFile content, pipeline events, CDC changes, CDC schema changes, CDC tombstones, CDC heartbeats, and live-probe captures. It also already states that CDC offsets, transaction-log positions, connector timestamps, schema-history timestamps, snapshot timestamps, and heartbeat timestamps must not determine Cadastre fact time, absence, cleanup, retraction, or watermark advancement by themselves.[^4]

`CadastreSilverObservation` already separates source observation time, collected time, normalized time, external schema classification, source extension fields, field quality, and Cadastre-owned flow-role evidence from canonical truth.[^5]

`GoldFact` already defines bitemporal half-open valid-time and known-time intervals, assertion states, evidence references, source authority, and the correction rule that closes `known_to` and emits a new fact rather than overwriting.[^6]

`VersionManifest` already has a broad manifest capture requirement across source instances, source config, stage state, completeness receipts, lakehouse refs, graph apply results, schema refs, policies, package artifacts, rule bundles, and projection profiles. It also already requires production replay to reject mismatches before writing production output.[^7]

`SourceCompletenessReceipt`, `SourceCompletenessProfile`, and `EvaluateSourceCompleteness` already establish that completeness evidence is evaluated through a profile and that unsafe states must not authorize absence, retraction, cleanup, or watermark advancement.[^8][^9]

`GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy` already establish that graph rebuild is a lakehouse-to-derived-view process and that graph query behavior must disclose or reject stale derived-view state.[^10]

#### 3.1 Baseline gap table

| Area | Existing PRD coverage | Residual implementation-divergence risk | Required refinement |
| --- | --- | --- | --- |
| Lakehouse system-of-record boundary | Explicitly defined. | Low conceptual risk, medium mechanical risk if table refs are incomplete. | Require all temporal/replay algorithms to use `LakehouseSnapshotRef`, `LakehouseCommitRef`, and `DatasetVersionRef` only as table-state refs, not fact-time refs. |
| Graph and CIM projection boundary | Explicitly defined. | Graph apply/rebuild idempotency and watermark advancement can still diverge. | Add `ProjectionWatermarkPolicy`, `WatermarkCommitRecord`, `GraphDeltaIdempotencyKey`, and `GraphRebuildEquivalencePolicy`. |
| `RawRecord` source time and CDC subtype behavior | Strong CDC subtype and non-authority rules already exist. | CDC replay sufficiency, schema-history dependency, and tombstone-to-retraction gating are underspecified. | Add `CDCReplayStateContract` and CDC-specific replay failure precedence. |
| `CadastreSilverObservation.observed_at` | Defines observed time and quality. | Source-time resolution rules can vary for absent, malformed, ambiguous, connector-derived, or historical-import times. | Add `TemporalSemanticsPolicy` and `ResolveFactTime`. |
| `GoldFact` bitemporal intervals | Defines valid and known intervals and correction append rule. | Comparable-fact correction behavior, interval splitting, confidence-only changes, and duplicate no-ops can diverge. | Add `GoldFactCorrectionPolicy` and `ApplyGoldCorrection`. |
| `GoldFact.assertion_state` | Has closed state vocabulary. | Transition causes and graph delta mapping are not total for all correction scenarios. | Add assertion-state transition table in `GoldFactCorrectionPolicy`. |
| Correction rule | Forbids in-place overwrite. | Does not yet define deterministic no-op records, error records, or snapshot refs for old/new facts. | Add `CorrectionSnapshotRefPolicy` and `GoldFactChangeSet`. |
| `VersionManifest` | Broad coverage. | Included/excluded fields for replay-equivalence checksums are not fully defined. | Add `ReplayEquivalencePolicy`, `ReplayInputSufficiencyCheck`, and `ComputeReplayEquivalenceChecksum`. |
| `StageStateRecord` | Requires output-affecting state to be hashable, replayable, and manifest-included. | CDC offsets, schema history, workflow side effects, watermark commits, and graph apply checkpoints need specific state kinds. | Add state-kind rows for CDC, temporal resolution, deterministic side effects, and watermarks. |
| `SourceCompletenessReceipt` | Strong source completeness model. | Late data and watermark interactions can diverge. | Add `LateArrivalPolicy` and `WatermarkCommitRecord`. |
| `SourceCompletenessProfile` | Strong authority gate. | Progress signals and source-completeness permissions need a total mapping table. | Add watermark/completeness/absence authority table as normative PRD appendix. |
| `EvaluateSourceCompleteness` | Pure deterministic authority for MVP. | Lack of explicit relationship to Beam/Flink/Kafka/Differential frontiers. | Define external progress signals as non-authoritative by default. |
| `CoverageAssertion` | Used for coverage-dependent absence/control/vulnerability facts. | Late authoritative evidence after coverage assertions can diverge. | Add late authoritative evidence correction rules. |
| `SourceAuthorityProfile` | Required for gold derivation. | Interaction with late records, delete evidence, and competing authoritative sources can diverge. | Add source-authority-aware correction matrix. |
| `LakehouseSnapshotRef` and `LakehouseCommitRef` | Defined. | Need mandatory old/new refs for corrections and table-set refs for replay/rebuild. | Add `CorrectionSnapshotRefPolicy` and stricter `ReplayInputSufficiencyCheck`. |
| `ReplayRetentionPolicy` and `TableMaintenancePolicy` | Defined. | Maintenance eligibility algorithm needs deterministic candidate refusal. | Add `DecideMaintenanceEligibility`. |
| `GraphDeltaSet` | Projection exists. | Delta idempotency key and signed-effect semantics need stronger contract. | Add `GraphDeltaIdempotencyKey` and graph delta algebra rows. |
| `GraphApplyProfile` and `GraphApplyResult` | Defined. | Resume after partial apply and watermark advancement need deterministic behavior. | Add `ResumeGraphApply` and `AdvanceProjectionWatermark`. |
| `GraphRebuildManifest` and `GraphIndexConsistencyCheck` | Defined. | Equivalence definition for rebuild from same refs/profiles needs explicit checksum rules. | Add `GraphRebuildEquivalencePolicy`. |
| `DerivedViewLagPolicy` | Defined. | Needs coupling to projection watermark and graph apply result. | Add `ProjectionWatermarkPolicy`. |

### 4. Independent source analyses

#### 4.1 Bitemporal databases and append-only fact systems

##### 4.1.1 XTDB

| Required subsection | Analysis |
| --- | --- |
| Source overview | XTDB is a bitemporal database reference. Its problem is querying data by valid time and transaction/system time. Its non-purpose for Cadastre is source completeness, lakehouse retention, identity resolution, or graph projection authority. |
| Architecture and operational model | XTDB persists immutable records and supports queries over a transaction/system-time dimension and an application valid-time dimension. Cadastre should treat this as a model for knowledge timelines, not as a required backend. |
| Data, schema, and temporal model | The transferable model is two independent temporal axes: valid time for when a fact is true in the domain, and system or transaction time for when the database knew it. |
| Correction, late-arrival, replay, or rebuild behavior | A correction can be represented as a new assertion at a later knowledge/system time that changes valid-time history while preserving previous knowledge. Cadastre must adapt this into append-only `GoldFact` rows with known-time closure. |
| Interfaces and extension points | Query modes such as current, valid-at, system-at, and bitemporal history are transferable as `BitemporalQueryMode` rows. |
| Validation and tests | No XTDB tests were run. Cadastre must create its own golden event-sequence corpus. |

###### Confirmed findings versus inferences

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| XTDB documentation describes valid time and transaction/system time. | Its bitemporal model preserves “as known then” and corrected history. | Cadastre should add explicit bitemporal query modes and correction lineage. | Query-mode naming could be reused only if product governance accepts it. | Exact XTDB v2/v1 differences were not exhaustively inspected. |

###### Transfer candidates

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Valid time and system time | `GoldFact.valid_*`, `GoldFact.known_*` | `adopt` | Valid time must describe domain validity; known time must describe Cadastre knowledge validity. | Corpus cases prove current, historical-valid, historical-known, and corrected-history queries. |
| Bitemporal query modes | `BitemporalQueryMode` | `adapt` | Modes must specify defaults, filters, interval semantics, and assertion-state inclusion. | Query fixtures produce byte-identical result IDs and ordering. |

###### Concepts that must not transfer

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Database system time as Cadastre known time by default | Cadastre known time is a product contract and may involve import policy, manifest closure, and persistence rules. | `KnowledgeTimeImportPolicy` and `ResolveFactTime`. |
| Database transaction time as source event time | Transaction time is persistence progress, not source occurrence. | `TemporalObservationTimeResolution.source_event_time`. |

Source basis: XTDB official bitemporal documentation was inspected; no XTDB build, query, or tests were executed.[^16]

##### 4.1.2 Datomic

| Required subsection | Analysis |
| --- | --- |
| Source overview | Datomic is an immutable database with datoms, transactions, history, assertions, and retractions. Its relevance is immutable knowledge history and transaction metadata. |
| Architecture and operational model | Datomic databases are values; historical database values can be queried. Transactions add assertions and retractions rather than mutating history in place. |
| Data, schema, and temporal model | Datomic’s transaction dimension is useful as a knowledge-history reference. Cadastre must not conflate Datomic transaction time with source valid time. |
| Correction, late-arrival, replay, or rebuild behavior | Assertions and retractions provide a pattern for append-only corrections. Cadastre must emit new facts, no-op records, or errors rather than overwrite. |
| Interfaces and extension points | The source pattern maps to `GoldFactCorrectionPolicy`, `KnowledgeTimeImportPolicy`, and audit reconstruction. |
| Validation and tests | No Datomic runtime or test suite was run. |

###### Confirmed findings versus inferences (Datomic)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Datomic documentation describes a history database and assertions/retractions. | Retractions preserve historical transaction context. | Cadastre should represent retraction as explicit assertion-state transition, not row deletion. | Transaction metadata patterns could inform `EvidenceRef` enrichment. | Exact Datomic Cloud versus Pro operational differences were not inspected. |

###### Transfer candidates (Datomic)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Assertion/retraction | `GoldFactChangeSet` | `adopt` | Retraction must close knowledge interval and emit explicit retraction row or change-set entry. | Prior and corrected states are queryable by known time. |
| History query | `BitemporalQueryMode.as_known_at` | `adapt` | Query must return the facts Cadastre knew at `known_at`, including superseded and retracted rows per mode. | Historical audit fixture matches expected rows. |

###### Concepts that must not transfer (Datomic)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Datomic transaction entity as complete Cadastre manifest | It does not capture table refs, source profiles, schema artifacts, graph apply state, or retention. | `VersionManifest`. |
| Retraction as downstream graph retraction authority by itself | Source authority and projection profile must authorize graph-visible effects. | `SourceAuthorityProfile` plus `GraphProjectionProfile`. |

Source basis: Datomic official history and transaction documentation and release pages were inspected; no local Datomic runtime was executed.[^17]

#### 4.2 Stream-time and out-of-order event systems

##### 4.2.1 Apache Flink

| Required subsection | Analysis |
| --- | --- |
| Source overview | Flink is a stream processor with event time, processing time, watermarks, windows, allowed lateness, and state. Its non-purpose is Cadastre fact authority. |
| Architecture and operational model | Operators process keyed streams, maintain state, use watermarks to model event-time progress, and can retain window state for allowed late records. |
| Data, schema, and temporal model | Event time belongs to records; processing time belongs to the runtime; watermarks estimate progress and out-of-orderness. |
| Correction, late-arrival, replay, or rebuild behavior | Flink allowed lateness is useful for late data policy, but Cadastre production must preserve evidence rather than use stream discard as default. |
| Interfaces and extension points | Watermark strategies and window policies transfer to `LateArrivalPolicy` only as external progress signals. |
| Validation and tests | No Flink job was run. Cadastre must validate late-arrival behavior with golden event sequences. |

###### Confirmed findings versus inferences (Apache Flink)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Flink docs define event time, processing time, watermarks, windows, and allowed lateness. | Watermarks are progress indicators, not evidence of source completeness. | Cadastre should use watermarks to route late records, never to authorize absence by default. | Flink side-output late records could inspire quarantine/shadow outputs. | Exact defaults vary by API and version; recheck before implementation. |

###### Transfer candidates (Apache Flink)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Watermark | `WatermarkCommitRecord` | `adapt` | A watermark may indicate progress only after `SourceCompletenessProfile` permits the dataset/scope. | Watermark denial fixture emits no source completeness, no retraction, no cleanup. |
| Allowed lateness | `LateArrivalPolicy` | `adapt` | Default production action after cutoff must preserve authoritative records as correction/quarantine/shadow, not silently discard. | Late authoritative and non-authoritative cases diverge as specified. |

###### Concepts that must not transfer (Apache Flink)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Watermark as absence proof | A watermark is stream progress, not source-scope completeness. | `EvaluateSourceCompleteness`. |
| Late-data discard as production default | It destroys evidence and can suppress corrections. | Preserve raw evidence and evaluate late arrival through policy. |

Source basis: Flink official event-time, watermark, and window/allowed-lateness documentation and release pages were inspected.[^18]

##### 4.2.2 Apache Beam

| Required subsection | Analysis |
| --- | --- |
| Source overview | Beam is a unified batch/stream model with event time, processing time, watermarks, triggers, windowing, and allowed lateness across runners. |
| Architecture and operational model | Beam pipelines transform `PCollection` values; windows, triggers, accumulation modes, and watermarks determine when results are emitted. |
| Data, schema, and temporal model | Beam distinguishes event time from processing time and models watermarks as progress estimates. |
| Correction, late-arrival, replay, or rebuild behavior | Beam late panes and allowed lateness are useful patterns for multiple emitted results. Cadastre should use this as a model for correction output, not as a source authority model. |
| Interfaces and extension points | Windowing/triggers transfer only as `LateArrivalPolicy` and `TemporalSemanticsPolicy` concepts. |
| Validation and tests | No Beam pipeline or runner was executed. |

###### Confirmed findings versus inferences (Apache Beam)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Beam docs define event time, processing time, watermarks, triggers, and allowed lateness. | Late panes can model repeated updates to a derived result. | Cadastre should expose late authoritative evidence as correction, not overwrite. | Accumulating/discarding pane ideas can inform graph delta no-op versus retraction. | Runner-specific behavior was not inspected. |

###### Transfer candidates (Apache Beam)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Late pane | `GoldFactChangeSet` | `adapt` | Late evidence must emit a new change set with deterministic assertion-state transitions. | Replayed late pane cases match byte-identical change sets. |
| Triggering model | `ProjectionWatermarkPolicy` | `study` | Projection watermark may advance only after completeness, delta validation, and apply success. | Watermark cases pass after graph apply and fail before. |

###### Concepts that must not transfer (Apache Beam)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Beam watermark as source complete | Beam progress is runner-stream progress, not source absence. | `SourceCompletenessReceipt` plus profile. |
| Dropped late data as acceptable production behavior | Cadastre replay and audit require preserved evidence. | `LateArrivalPolicy.default_after_cutoff_action`. |

Source basis: Beam official programming guide, watermarks/triggers material, Javadocs, and release pages were inspected.[^19]

##### 4.2.3 Kafka Streams

| Required subsection | Analysis |
| --- | --- |
| Source overview | Kafka Streams is a stream-processing library that uses record timestamps, stream time, window stores, grace periods, changelogs, and state stores. |
| Architecture and operational model | Applications consume Kafka topics, maintain local state stores backed by changelogs, and emit derived records. |
| Data, schema, and temporal model | Record timestamps and stream time are progress concepts. Grace periods bound how late records are accepted for a window. |
| Correction, late-arrival, replay, or rebuild behavior | Changelog-backed state stores are useful for replayable derived state, but Cadastre must persist authoritative outputs in the lakehouse. |
| Interfaces and extension points | Grace periods transfer as `LateArrivalPolicy.cutoff` inputs only. State-store changelogs transfer as `StageStateRecord` patterns only when output-affecting. |
| Validation and tests | No Kafka Streams app was executed. |

###### Confirmed findings versus inferences (Kafka Streams)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Kafka Streams docs describe stream processing over Kafka records, state stores, changelogs, timestamps, and window/grace behavior. | Grace periods manage out-of-order records, not source authority. | Cadastre should not treat stream-time progress or changelog state as fact validity. | Changelog checkpoints can inspire replay diagnostics. | Exact late-record default behavior varies by API version and topology. |

###### Transfer candidates (Kafka Streams)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Grace period | `LateArrivalPolicy.allowed_lateness` | `adapt` | Grace must affect route/action only, not raw evidence preservation. | Late record corpus cases produce expected route. |
| State-store changelog | `StageStateRecord` | `adapt` | Only output-affecting state store refs may affect replay, and they must be hashable and manifest-included. | Replay rejects missing state ref. |

###### Concepts that must not transfer (Kafka Streams)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Stream time as known time | Stream time is derived from observed record timestamps, not Cadastre knowledge validity. | `known_from` from `ResolveFactTime`. |
| Changelog offset as source completeness | Offset progress is transport progress, not source absence. | `EvaluateSourceCompleteness`. |

Source basis: Apache Kafka official documentation and release pages were inspected; explanatory Kafka Streams grace-period docs were treated as lower-weight support.[^20]

#### 4.3 CDC and source-log replay systems

##### 4.3.1 Debezium MySQL connector

| Required subsection | Analysis |
| --- | --- |
| Source overview | Debezium MySQL captures row-level changes from a consistent snapshot and then the MySQL binlog. It is relevant to Cadastre raw CDC evidence and replay state. |
| Architecture and operational model | The connector snapshots tables, stores offsets, reads binlog changes, emits create/update/delete events, may emit tombstones, and depends on schema history. |
| Data, schema, and temporal model | Binlog filename/position or GTID, transaction metadata, connector timestamps, and schema-history state are replay and ordering data, not Cadastre fact time. |
| Correction, late-arrival, replay, or rebuild behavior | Source deletes and tombstones are raw delete evidence only. Downstream retraction requires `SourceAuthorityProfile` and completeness. |
| Interfaces and extension points | Connector config, offset storage, snapshot modes, heartbeat config, and schema history map to `CDCReplayStateContract`. |
| Validation and tests | No connector was run; no MySQL instance or binlog was inspected. |

###### Confirmed findings versus inferences (Debezium MySQL connector)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Debezium MySQL docs describe snapshots, binlog reading, offsets, schema history, delete events, tombstones, and heartbeats. | CDC positions are necessary for resumability and replay sufficiency. | Cadastre must include CDC offset and schema-history refs in `VersionManifest`. | Debezium transaction metadata may inform batch grouping. | Exact connector defaults must be rechecked by version. |

###### Transfer candidates (Debezium MySQL connector)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Initial snapshot plus binlog handoff | `CDCReplayStateContract` | `adopt` | State must include snapshot marker, source partition, log position, schema history ref, and handoff marker. | Replay rejects missing handoff or schema history. |
| Tombstone | `RawRecord.source_event_subtype = cdc_tombstone` | `adopt` | Tombstone must not retract a `GoldFact` unless authority and completeness gates pass. | Tombstone-only corpus emits raw delete evidence and no gold retraction. |

###### Concepts that must not transfer (Debezium MySQL connector)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Binlog position as valid time | Log position orders database changes; it is not the domain occurrence time. | `TemporalObservationTimeResolution`. |
| Heartbeat as completeness | Heartbeat indicates connector liveness/progress, not source-scope absence. | `SourceCompletenessReceipt` with profile gate. |

Source basis: Debezium official MySQL connector and state-storage documentation was inspected.[^21]

##### 4.3.2 Debezium PostgreSQL connector

| Required subsection | Analysis |
| --- | --- |
| Source overview | Debezium PostgreSQL captures changes from an initial snapshot and WAL logical decoding with LSN-based offsets. |
| Architecture and operational model | The connector snapshots table data, streams WAL changes, persists offsets, emits row-change events, and handles delete and tombstone behavior. |
| Data, schema, and temporal model | LSN, transaction ID, connector timestamp, and schema metadata support ordering/replay, not Cadastre valid or known time. Replica identity settings can affect delete/update detail. |
| Correction, late-arrival, replay, or rebuild behavior | Deletes and tombstones are evidence. Retraction authority is downstream and profile-gated. |
| Interfaces and extension points | Slot, publication, offset, schema, and heartbeat settings map to CDC replay inputs. |
| Validation and tests | No PostgreSQL source, slot, publication, or connector was executed. |

###### Confirmed findings versus inferences (Debezium PostgreSQL connector)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Debezium PostgreSQL docs describe snapshots, WAL/LSN offsets, delete/tombstone behavior, schema handling, and replica identity caveats. | LSN is required for deterministic CDC replay but insufficient as source authority. | Cadastre must reject production replay when LSN/schema-history refs are missing or unresolved. | Replica identity state could be recorded as schema compatibility input. | Exact replica identity effects should be tested with fixtures. |

###### Transfer candidates (Debezium PostgreSQL connector)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| WAL LSN | `StageStateRecord.state_kind = cdc_offset` | `adopt` | LSN must be persisted, hashed, and included in `VersionManifest` when output-affecting. | Replay without LSN fails `CDC_LOG_POSITION_UNAVAILABLE`. |
| Replica identity caveat | `LakehouseSchemaCompatibilityCheck` / CDC schema check | `adapt` | CDC delete/update payload sufficiency must be validated per table. | Delete fixture with insufficient replica identity emits deterministic error. |

###### Concepts that must not transfer (Debezium PostgreSQL connector)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| WAL LSN time as known time | LSN orders database log entries and does not represent Cadastre knowledge interval. | `known_from` policy. |
| Connector snapshot timestamp as source occurrence | Snapshot timestamp is collection state. | Explicit source event time or fallback policy. |

Source basis: Debezium official PostgreSQL connector documentation was inspected.[^22]

##### 4.3.3 Kafka Connect offsets and schema history

| Required subsection | Analysis |
| --- | --- |
| Source overview | Kafka Connect offset and schema-history behavior provides durable connector progress and schema interpretation state. |
| Architecture and operational model | Workers and connectors store offset state and schema history in configured stores/topics so connectors can resume and parse future records. |
| Data, schema, and temporal model | Offsets and schema-history records are replay and parsing inputs, not source occurrence, absence, or fact authority. |
| Correction, late-arrival, replay, or rebuild behavior | Missing offsets can force resnapshot or replay gaps; missing schema history can make historical CDC records unparseable. |
| Interfaces and extension points | Offset storage, schema-history storage, connector config, and task state map to `CDCReplayStateContract` and `StageStateRecord`. |
| Validation and tests | No Kafka Connect worker was run. |

###### Confirmed findings versus inferences (Kafka Connect offsets and schema history)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Kafka Connect and Debezium docs describe offset and schema-history stores for resumability and record interpretation. | Replay sufficiency depends on both data offsets and schema history. | `VersionManifest` must include schema-history refs when CDC parsing affects output. | Store compaction behavior may need retention protection. | Exact offset-store serialization was not inspected. |

###### Transfer candidates (Kafka Connect offsets and schema history)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Offset store | `CDCReplayStateContract.offset_ref` | `adopt` | Required for CDC replay when output depends on CDC ordering or resume. | Missing offset ref rejects production replay. |
| Schema history | `CDCReplayStateContract.schema_history_ref` | `adopt` | Required when CDC payload interpretation depends on schema history. | Schema-history mismatch rejects before output. |

###### Concepts that must not transfer (Kafka Connect offsets and schema history)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Offset commit as raw persistence proof | Offset commit can occur outside Cadastre raw persistence unless contracted. | Cursor/offset commit ordering in `CDCReplayStateContract`. |
| Schema-history timestamp as fact time | Schema-history time describes schema evolution, not source event validity. | Source event time resolution policy. |

Source basis: Kafka Connect and Debezium offset/schema-history documentation was inspected.[^21][^22]

#### 4.4 Deterministic workflow replay

##### 4.4.1 Temporal.io

| Required subsection | Analysis |
| --- | --- |
| Source overview | Temporal is a durable workflow system whose relevance is deterministic replay of workflow event history, nondeterminism control, side-effect recording, and patch/versioning. |
| Architecture and operational model | Workflows execute deterministic code, record command/event history, replay workflow code against history, and run activities for side effects. |
| Data, schema, and temporal model | Workflow event history is durable execution state. It is not source evidence and not Cadastre fact time. |
| Correction, late-arrival, replay, or rebuild behavior | Temporal replay mismatch is the key pattern: production replay should fail before output when command/history equivalence breaks. |
| Interfaces and extension points | Side effects, activities, patch/versioning, and nondeterminism errors map to `DeterministicSideEffectRecord` and `ReplayEquivalencePolicy`. |
| Validation and tests | No Temporal worker or replay tests were executed. |

###### Confirmed findings versus inferences (Temporal.io)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Temporal docs describe workflow event history, deterministic replay, activities, side-effect recording, patching, and nondeterminism errors. | Replay safety depends on recording every output-affecting nondeterministic decision. | Cadastre should persist deterministic side-effect records for non-source runtime choices. | Temporal patching can inspire mapping/profile version migration. | Exact SDK-specific replay diagnostics differ by language. |

###### Transfer candidates (Temporal.io)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Workflow event-history matching | `ReplayEquivalencePolicy` | `adopt` | Replayed production run must reject mismatched command/input/output-affecting state before output. | Manifest mismatch fixture fails with deterministic error. |
| SideEffect | `DeterministicSideEffectRecord` | `adapt` | Nondeterministic choices affecting output must be recorded and replayed, not recomputed. | Replayed random/time-dependent choice yields same output or fails. |

##### Concepts that must not transfer (Temporal.io)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Workflow history as source evidence | Workflow history proves execution, not source assertions. | `RawRecord` and `EvidenceRef`. |
| Activity success as source completeness | Activity success is process outcome, not source-scope completeness. | `EvaluateSourceCompleteness`. |

Source basis: Temporal official workflow determinism, side-effect, versioning/patching, and nondeterminism documentation was inspected.[^23]

#### 4.4.2 Cadence

| Required subsection | Analysis |
| --- | --- |
| Source overview | Cadence is a durable workflow system and useful comparison point for deterministic workflow replay and side-effect recording. |
| Architecture and operational model | Similar to Temporal, workflows replay deterministic histories and record side-effect values in history. |
| Data, schema, and temporal model | Workflow history is durable execution state, not source evidence. |
| Correction, late-arrival, replay, or rebuild behavior | The transferable pattern is recording nondeterministic side effects so replay sees stable values. |
| Interfaces and extension points | Side effects map to `DeterministicSideEffectRecord`; workflow history maps to replay diagnostics, not Cadastre evidence. |
| Validation and tests | No Cadence runtime was executed. |

##### Confirmed findings versus inferences (Cadence)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Cadence docs describe side-effect recording and replay behavior. | Durable replay systems require separating deterministic workflow code from external side effects. | Cadastre must record output-affecting nondeterminism in versioned records. | Cadence event history diagnostics can inform replay error detail. | No Cadence/Temporal API compatibility conclusion is made. |

##### Transfer candidates (Cadence)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Recorded side effects | `DeterministicSideEffectRecord` | `adapt` | Every nondeterministic value affecting production output must have a stable record ID and checksum. | Missing side-effect record rejects production replay. |

##### Concepts that must not transfer (Cadence)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Workflow replay success as product replay success | Cadastre output equivalence also requires raw, table, schema, policy, and graph refs. | `ReplayInputSufficiencyCheck`. |

Source basis: Cadence official side-effect/replay documentation was inspected.[^24]

### 4.5 Event sourcing and projection/read-model systems

#### 4.5.1 EventStoreDB / KurrentDB

| Required subsection | Analysis |
| --- | --- |
| Source overview | EventStoreDB/KurrentDB is an event-sourcing system with append-only streams, expected revision, idempotent append behavior, projections, checkpoints, retries, and parked messages. |
| Architecture and operational model | Events are appended to streams with IDs and expected revisions. Projections/read models consume streams with checkpoints and may reset from beginning. |
| Data, schema, and temporal model | Event position, revision, and checkpoint are stream progress. They are not Cadastre valid time or known time. |
| Correction, late-arrival, replay, or rebuild behavior | Append-only streams and projection reset are strong references for graph delta apply and graph read-model rebuild. |
| Interfaces and extension points | Expected revision maps to concurrency/idempotency. Parked messages map to persisted graph apply errors. Projection reset maps to graph rebuild from authoritative inputs. |
| Validation and tests | No EventStoreDB/KurrentDB server was run. |

##### Confirmed findings versus inferences (EventStoreDB / KurrentDB)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Official docs describe expected revision, idempotent event IDs, persistent subscription checkpoints, parked messages, and projection reset behavior. | Projection consumers must own checkpoints and retry/error handling. | Cadastre graph apply must persist checkpoints, idempotency keys, and partial failure results. | Expected revision could inspire graph apply compare-and-set behavior. | Exact KurrentDB/EventStoreDB naming drift should be checked before spec text. |

##### Transfer candidates (EventStoreDB / KurrentDB)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Expected revision | `GraphDeltaIdempotencyKey` / apply precondition | `adapt` | Graph apply must reject or no-op repeated/old deltas deterministically. | Reapplying same delta set returns idempotent result. |
| Parked message | `GraphApplyResult.error_records` | `adapt` | Failed deltas must be persisted with stable error codes and retry eligibility. | Partial failure fixture resumes without duplicate mutation. |
| Projection reset | `GraphRebuildManifest` | `adapt` | Graph rebuild must reset derived view from authoritative refs, not mutate gold facts. | Rebuild from same refs produces same checksum. |

##### Concepts that must not transfer (EventStoreDB / KurrentDB)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Projection-owned output stream as graph truth | Projection output is derived and can lag or fail. | `GraphDeltaSet` from gold facts plus lakehouse refs. |
| Event stream revision as known time | Revision is stream position, not Cadastre knowledge validity. | `known_from`/`known_to`. |

Source basis: EventStoreDB/KurrentDB official append, persistent subscription, checkpoint, parked message, and projection documentation was inspected.[^25]

### 4.6 Incremental computation and correction algebra

#### 4.6.1 Differential Dataflow

| Required subsection | Analysis |
| --- | --- |
| Source overview | Differential Dataflow is an incremental computation framework where collections are updated by `(data, time, diff)` records and frontiers indicate completion/progress. |
| Architecture and operational model | It incrementally maintains derived collections using arrangements, logical times, positive and negative differences, and progress/frontier tracking. |
| Data, schema, and temporal model | Logical time and diff are computation semantics, not Cadastre source event time or fact authority. |
| Correction, late-arrival, replay, or rebuild behavior | Positive/negative diffs provide a strong algebra for deterministic correction and graph delta effects. |
| Interfaces and extension points | Signed updates map to `GoldFactChangeSet` and graph delta operations. Frontiers map to non-authoritative progress signals. |
| Validation and tests | No differential-dataflow program was run. |

##### Confirmed findings versus inferences (Differential Dataflow)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Differential Dataflow docs describe collections as updates with data, time, and integer differences, along with frontiers and arrangements. | Signed effects are a compact model for retractions and incremental corrections. | Cadastre should model correction output as deterministic signed effects but expose gold facts, not implementation diffs, as authority. | Arrangements may inspire materialized intermediate indexes. | Public graph API diff exposure should be rejected unless explicitly specified. |

##### Transfer candidates (Differential Dataflow)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Positive/negative diff | `GoldFactChangeSet`, `GraphDeltaSet` | `adapt` | Every correction must emit signed effects or deterministic no-op/error records. | Change-set checksum is stable across replay. |
| Frontier | `WatermarkCommitRecord` | `cautionary_reference` | Completion/frontier may indicate computation progress only, not source completeness. | Frontier-only case cannot retract or cleanup. |

##### Concepts that must not transfer (Differential Dataflow)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Differential diff as public graph API | Implementation diffs are not domain facts and can expose unstable internal algebra. | Cadastre graph deltas with source fact refs. |
| Completion frontier as source completeness | Frontier is computation progress. | `SourceCompletenessProfile`. |

Source basis: Differential Dataflow documentation and repository documentation were inspected.[^26]

### 4.7 Lakehouse table formats and catalog versioning

#### 4.7.1 Apache Iceberg

| Required subsection | Analysis |
| --- | --- |
| Source overview | Iceberg defines table snapshots, metadata files, manifests, manifest lists, schema IDs, partition specs, delete files, snapshot refs, and atomic metadata pointer updates. |
| Architecture and operational model | Writers create data/delete files and metadata, then commit a new table metadata file through catalog atomicity. Readers use a selected snapshot. |
| Data, schema, and temporal model | Snapshot IDs, metadata-file locations, manifest-list paths, sequence numbers, schema IDs, partition spec IDs, and delete files are table-state refs, not fact time. |
| Correction, late-arrival, replay, or rebuild behavior | Snapshot refs are essential for deterministic replay and graph rebuild. Snapshot expiration can remove replay inputs. |
| Interfaces and extension points | Catalogs, branches/tags, schema evolution, delete files, and snapshot refs map to `LakehouseSnapshotRef`, `LakehouseCommitRef`, and `ReplayRetentionPolicy`. |
| Validation and tests | No Iceberg table was created or read. |

##### Confirmed findings versus inferences (Apache Iceberg)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Iceberg specs/docs describe metadata files, snapshots, manifest lists, manifests, delete files, schema IDs, and snapshot expiration. | Replayable reads require exact table-format snapshot identity. | Cadastre must record Iceberg-native refs in every production-affecting manifest. | Iceberg branches/tags may support protected replay refs. | Exact engine behavior must be validated per selected implementation. |

##### Transfer candidates (Apache Iceberg)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Snapshot ID and metadata file | `LakehouseSnapshotRef` | `adopt` | Required for every authoritative Iceberg table read that affects output. | Replay rejects unresolved snapshot or metadata checksum mismatch. |
| Snapshot expiration | `ReplayRetentionPolicy` | `adopt` | Maintenance must refuse deleting snapshots protected by replay, graph rebuild, legal hold, or retention policy. | Candidate expiration fixture emits refusal decision. |

##### Concepts that must not transfer (Apache Iceberg)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Iceberg snapshot time as valid time | Snapshot time is table state. | `GoldFact.valid_from` from `ResolveFactTime`. |
| Partition spec as source scope | Partitioning is storage layout. | `SourceCompletenessProfile.scope_key`. |

Source basis: Iceberg table specification, docs, and release pages were inspected.[^27]

#### 4.7.2 Delta Lake

| Required subsection | Analysis |
| --- | --- |
| Source overview | Delta Lake defines a transaction-log protocol with serial table versions, actions, checkpoints, protocol features, deletion vectors, and vacuum. |
| Architecture and operational model | Writers append transaction log entries; readers reconstruct table snapshots from checkpoint plus log tail. |
| Data, schema, and temporal model | Delta versions and log action timestamps are table-state/protocol state, not Cadastre domain time. |
| Correction, late-arrival, replay, or rebuild behavior | Delta table versions and protocol features are required replay inputs. Vacuum can destroy time-travel/replay inputs. |
| Interfaces and extension points | Add/remove actions, `txn` actions, checkpoints, protocol feature flags, and vacuum retention map to lakehouse refs and maintenance policy. |
| Validation and tests | No Delta table was read or written. |

##### Confirmed findings versus inferences (Delta Lake)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Delta protocol/docs describe transaction log versions, add/remove actions, checkpoints, protocol versions/features, deletion vectors, and vacuum. | Exact Delta replay depends on log/checkpoint availability and protocol support. | Cadastre must reject replay when required log/version/protocol refs are missing or incompatible. | Delta `txn` action can inspire idempotent writer refs. | Engine-specific vacuum behavior requires implementation tests. |

##### Transfer candidates (Delta Lake)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Transaction-log version | `LakehouseCommitRef` / `LakehouseSnapshotRef` | `adopt` | Record exact version and required log/checkpoint refs for reads/writes. | Replay from same version produces same table checksum. |
| Vacuum | `TableMaintenancePolicy` | `adopt` | Destructive maintenance must be refused when protected replay refs would be invalidated. | Vacuum candidate fixture fails retention check. |

##### Concepts that must not transfer (Delta Lake)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Delta version as known time | Table version is storage-state order, not knowledge interval. | `GoldFact.known_from`. |
| Delta `txn` app ID as complete run identity | It is table-format idempotence metadata only. | `VersionManifest.run_id` and `LakehouseCommitRef`. |

Source basis: Delta Lake protocol documentation, docs, and release pages were inspected.[^28]

#### 4.7.3 Apache Hudi

| Required subsection | Analysis |
| --- | --- |
| Source overview | Hudi defines a table timeline with instants for commits, delta commits, compaction, cleaning, rollback, restore, savepoint, and clustering. |
| Architecture and operational model | Timeline instants move through requested/inflight/completed states; readers use completed instants; table services clean, compact, roll back, or restore. |
| Data, schema, and temporal model | Hudi instant time is table timeline state, not Cadastre fact valid or known time. |
| Correction, late-arrival, replay, or rebuild behavior | Savepoints and cleaner behavior are important retention and replay references. Restore/rollback are table operations, not product rollback by themselves. |
| Interfaces and extension points | Instants, cleaner policy, rollback/restore/savepoint refs map to `LakehouseCommitRef`, `ReplayRetentionPolicy`, and `TableMaintenancePolicy`. |
| Validation and tests | No Hudi table was run. |

##### Confirmed findings versus inferences (Apache Hudi)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Hudi docs describe timeline instants, commit states, rollback, restore, savepoints, cleaning, and incremental queries. | Timeline refs are necessary for replay/maintenance safety but not domain time. | Cadastre must protect Hudi instants needed for replay and graph rebuild. | Savepoints can inspire protected replay checkpoints. | Exact cleaner defaults must be pinned by implementation. |

##### Transfer candidates (Apache Hudi)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Timeline instant | `LakehouseCommitRef` | `adopt` | Record instant ID, action, state, schema, and table role for output-affecting writes. | Replay rejects missing instant. |
| Savepoint/cleaner | `ReplayRetentionDecision` | `adapt` | Cleaning must refuse candidates that delete protected instants or file slices. | Protected-instant cleanup fixture refused. |

##### Concepts that must not transfer (Apache Hudi)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Hudi instant time as known time | Instant is table operation time. | `KnowledgeTimeImportPolicy`. |
| Hudi rollback as product rollback | It only rolls back table writes. | Cadastre rollback/replay manifests across all layers. |

Source basis: Hudi official timeline, table-service, rollback, restore, savepoint, cleaner, and release documentation was inspected.[^29]

#### 4.7.4 Project Nessie

| Required subsection | Analysis |
| --- | --- |
| Source overview | Nessie provides Git-like catalog versioning with branches, tags, commits, content keys, content IDs, and multi-table commit semantics. |
| Architecture and operational model | Catalog operations reference content objects such as Iceberg tables through commits on branches/tags. |
| Data, schema, and temporal model | Branch names are mutable catalog references. Commit hashes and table-format snapshot IDs are required for immutable replay. |
| Correction, late-arrival, replay, or rebuild behavior | Cross-table consistency can be represented by catalog commits or table-set refs, but branch names alone are not production approval. |
| Interfaces and extension points | Branch/tag/commit and content key/ID concepts map to `CrossTableCommitProfile` and `CatalogBranchPromotionPolicy`. |
| Validation and tests | No Nessie server was run. |

##### Confirmed findings versus inferences (Project Nessie)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Nessie docs describe branches, tags, commits, content keys/IDs, and Iceberg integration. | Multi-table consistency needs immutable catalog state and underlying table snapshot refs. | Cadastre must reject mutable branch-only snapshot refs for production replay. | Nessie commits can simplify table-set checksum construction. | Selected catalog implementation may not be Nessie. |

##### Transfer candidates (Project Nessie)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Catalog commit | `CrossTableCommitProfile` | `adapt` | Use only when paired with immutable table-format snapshot refs and checksum. | Branch-only case rejects; commit+snapshot case passes. |
| Branch/tag promotion | `CatalogBranchPromotionPolicy` | `adapt` | Branch movement must be governed by validation, approval, rollback, and immutable refs. | Promotion fixture records immutable commit and approval refs. |

##### Concepts that must not transfer (Project Nessie)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Branch name as production approval | Branch is catalog visibility, not governance. | `CatalogBranchPromotionPolicy`. |
| Content key as asset identity | Content key names catalog object, not enterprise asset. | Cadastre `CanonicalEntity` and `SourceAsset`. |

Source basis: Project Nessie official docs and release pages were inspected.[^30]

#### 4.7.5 lakeFS

| Required subsection | Analysis |
| --- | --- |
| Source overview | lakeFS provides object-store versioning with commits, branches, rollback, and garbage collection over data lake objects. |
| Architecture and operational model | It versions object-store namespaces, not necessarily table-format semantic snapshots unless wrapping table refs. |
| Data, schema, and temporal model | Object-store commit time is object version state, not table-format snapshot proof and not fact time. |
| Correction, late-arrival, replay, or rebuild behavior | lakeFS is useful as object-GC comparison and as a warning that object versioning must wrap table-format refs. |
| Interfaces and extension points | Branch/commit/GC concepts map only to `LakehouseSnapshotRef` when paired with table-format-native identities. |
| Validation and tests | No lakeFS repository or object store was run. |

##### Confirmed findings versus inferences (lakeFS)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| lakeFS docs describe commits, branches, object versioning, rollback, and GC. | Object versioning can preserve bytes but not prove table semantic state by itself. | Cadastre must require table-format snapshot refs even when object-store versioning exists. | lakeFS GC policies can inform retention preflight. | No selected object versioning product is assumed. |

##### Transfer candidates (lakeFS)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Object commit | `LakehouseSnapshotRef.object_version_ref` | `study` | May supplement but not replace table-format metadata/log refs. | Object-only ref fails replay preflight. |
| GC preflight | `ReplayRetentionDecision` | `adapt` | Object GC must not remove protected table data/log/metadata. | GC candidate set denied when protected refs exist. |

##### Concepts that must not transfer (lakeFS)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Object-store commit as table snapshot proof | Object commits do not by themselves prove table metadata, schema, partition, and delete/tombstone identity. | `LakehouseSnapshotRef` wrapping native table snapshot. |
| Object commit time as known time | Object-store progress is not Cadastre knowledge validity. | `GoldFact.known_from`. |

Source basis: lakeFS official docs and repository pages were inspected.[^31]

### 4.8 Metadata, lineage, and derived read-model systems

#### 4.8.1 DataHub

| Required subsection | Analysis |
| --- | --- |
| Source overview | DataHub is a metadata platform with entities/aspects, metadata change proposals/logs, graph/search indexing, and restore-index behavior. |
| Architecture and operational model | Metadata writes persist aspects; derived search/graph indexes can be rebuilt from persisted metadata-aspect storage. |
| Data, schema, and temporal model | Aspect versioning and MCL/MCP event times are metadata lifecycle state, not Cadastre bitemporal facts. |
| Correction, late-arrival, replay, or rebuild behavior | Restore-index is a strong pattern for derived read-model rebuild from authoritative state. |
| Interfaces and extension points | Entity/aspect registry, MCP/MCL, graph/search index restore map to `ArtifactClassPolicy` and `GraphRebuildManifest`. |
| Validation and tests | No DataHub instance or restore was executed. |

##### Confirmed findings versus inferences (DataHub)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| DataHub docs describe aspect persistence, MCP/MCL, and graph/search index restore from persisted aspects. | Derived indexes can drift and require explicit restore from persisted truth. | Cadastre graph rebuild should be lakehouse-to-derived-view with explicit manifest/check. | Aspect registries can inform Cadastre registry governance. | Runtime restore behavior not observed. |

##### Transfer candidates (DataHub)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Restore index from persisted aspects | `GraphRebuildManifest` | `adopt` | Rebuild reads authoritative lakehouse refs and emits manifest/checksum; graph backend is target only. | Same refs/profile produce same rebuild checksum. |
| Metadata change log | `RunDatasetIOContract` / diagnostics | `adapt` | Lineage events may describe run IO but must not become source evidence. | MCL-only case cannot emit gold/graph. |

##### Concepts that must not transfer (DataHub)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Metadata graph as gold facts | Metadata lifecycle is not Cadastre fact validity. | `GoldFact` bitemporal contract. |
| Search index as authority | Search is derived and may drift. | `GraphIndexConsistencyCheck`. |

Source basis: DataHub official docs and prior Cadastre lakehouse research were inspected.[^11][^32]

#### 4.8.2 OpenLineage

| Required subsection | Analysis |
| --- | --- |
| Source overview | OpenLineage defines run, job, dataset, event, and facet schemas for pipeline lineage. |
| Architecture and operational model | Producers emit run events with job/run/dataset metadata and facets to a lineage backend. |
| Data, schema, and temporal model | Event time and dataset facets are lineage metadata, not table snapshot proof, source completeness, or fact authority. |
| Correction, late-arrival, replay, or rebuild behavior | Lineage events can support audit but not replay sufficiency by themselves. |
| Interfaces and extension points | Custom facets map to `LineageFacetMappingPolicy`; dataset refs map to `RunDatasetIOContract` only when paired with Cadastre refs. |
| Validation and tests | No OpenLineage client or backend was run. |

##### Confirmed findings versus inferences (OpenLineage)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| OpenLineage docs/specs describe run/job/dataset events, lifecycle event types, facets, and custom facet schemas. | Lineage is pipeline metadata, not source evidence. | Cadastre should admit lineage only through mapping policy and artifact class checks. | Immutable facet schema URLs are a good schema identity pattern. | Backend-specific lineage storage was not inspected. |

##### Transfer candidates (OpenLineage)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Run/job/dataset | `RunDatasetIOContract` | `adapt` | May record run IO with explicit `authority_class`; cannot replace table snapshot refs. | Lineage-only run cannot satisfy replay. |
| Custom facet schema | `LineageFacetMappingPolicy` | `adopt` | Accepted facets require namespace, immutable schema URL, schema checksum, and collision rule. | Mutable or missing schema checksum rejects. |

##### Concepts that must not transfer (OpenLineage)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Run complete as source completeness | A pipeline finishing does not prove source scope completeness. | `SourceCompletenessReceipt`. |
| Dataset name as table snapshot proof | Names are not immutable table-state refs. | `LakehouseSnapshotRef` and `DatasetVersionRef`. |

Source basis: OpenLineage official specification and docs were inspected.[^33]

#### 4.8.3 OpenMetadata

| Required subsection | Analysis |
| --- | --- |
| Source overview | OpenMetadata is a metadata platform with entity schemas, versions, lineage APIs, search indexing, ingestion connectors, classifications, and governance metadata. |
| Architecture and operational model | It stores metadata entities and relationships, emits change events, and indexes derived search/lineage views. |
| Data, schema, and temporal model | Metadata entity versions are governance/metadata lifecycle state, not Cadastre `GoldFact` valid/known intervals. |
| Correction, late-arrival, replay, or rebuild behavior | Search/index restore and entity versioning are useful derived-read-model and registry references only. |
| Interfaces and extension points | Custom properties, glossary/classification, lineage API, ingestion connectors map to registry governance and artifact-class contracts. |
| Validation and tests | No OpenMetadata server or ingestion was run. |

##### Confirmed findings versus inferences (OpenMetadata)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| OpenMetadata docs describe entities, versions, lineage, search indexing, ingestion, classifications, and governance artifacts. | Metadata entity lifecycle is distinct from source evidence and fact authority. | Cadastre should use OpenMetadata-like governance only to gate artifacts, not facts. | Custom-property schema governance can inform registry contracts. | Runtime search restore and API semantics not tested. |

##### Transfer candidates (OpenMetadata)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Custom property schema | `RegistryCustomPropertySchema` | `adapt` | Custom properties may gate governance metadata only. | Governance metadata cannot emit gold/graph. |
| Lineage API edge | `RunDatasetIOContract` | `cautionary_reference` | Metadata lineage edge can map to run IO only under policy. | Lineage edge alone fails evidence/completeness checks. |

##### Concepts that must not transfer (OpenMetadata)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Metadata graph edge as Cadastre graph edge | Cadastre graph edges require gold facts, edge semantics, evidence, and projection profile. | `GraphEdgeSemantics`. |
| Glossary/classification as source authority | Governance labels do not prove source facts. | `SourceAuthorityProfile`. |

Source basis: OpenMetadata official docs and prior normalization research were inspected.[^34]

#### 4.8.4 Marquez

| Required subsection | Analysis |
| --- | --- |
| Source overview | Marquez is an OpenLineage-oriented metadata backend for storing and visualizing lineage. |
| Architecture and operational model | It ingests OpenLineage-compatible events and exposes run/job/dataset lineage APIs and UI graphs. |
| Data, schema, and temporal model | Run and dataset lineage timestamps are pipeline metadata, not Cadastre source-event or known time. |
| Correction, late-arrival, replay, or rebuild behavior | Marquez lineage graphs are derived metadata read models and not source-completeness or replay sufficiency. |
| Interfaces and extension points | OpenLineage HTTP endpoint and lineage API map only to external lineage projection or `RunDatasetIOContract`. |
| Validation and tests | No Marquez server was run. |

##### Confirmed findings versus inferences (Marquez)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Marquez docs describe an OpenLineage-compatible backend and lineage graph UI/API. | Backend lineage graph is a derived metadata view. | Cadastre must not let lineage graph state satisfy graph rebuild or gold facts. | It can be an external projection target. | Exact current release was not established. |

##### Transfer candidates (Marquez)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Lineage backend | external lineage projection | `study` | May receive Cadastre run IO as non-authoritative projection only. | Projection failure does not affect authoritative output. |

##### Concepts that must not transfer (Marquez)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Lineage graph as Cadastre graph serving | It lacks Cadastre fact, evidence, assertion-state, and bitemporal graph contracts. | `GraphProjectionProfile` and `GraphReadModelSchemaProfile`. |

Source basis: Marquez official docs and OpenLineage context were inspected.[^33][^35]

#### 4.8.5 dbt artifacts

| Required subsection | Analysis |
| --- | --- |
| Source overview | dbt artifacts separate static project manifests, execution results, source freshness results, and semantic artifacts. |
| Architecture and operational model | dbt produces JSON artifacts describing DAGs, compiled nodes, run outcomes, source freshness, and semantic model metadata. |
| Data, schema, and temporal model | Artifact timestamps and freshness outputs are workflow metadata and source-test results, not Cadastre source completeness. |
| Correction, late-arrival, replay, or rebuild behavior | Artifact separation is a useful pattern for `ArtifactClassPolicy` and validation output separation. |
| Interfaces and extension points | Manifest, run results, sources, and semantic manifests map to typed artifact classes only. |
| Validation and tests | No dbt project was executed. |

##### Confirmed findings versus inferences (dbt artifacts)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| dbt docs describe `manifest.json`, `run_results.json`, `sources.json`, and semantic artifacts. | Static DAG, run result, freshness, and semantic artifacts solve different problems. | Cadastre should prevent artifact substitution. | dbt artifact schemas can inform validation outputs. | No dbt run was performed. |

##### Transfer candidates (dbt artifacts)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Artifact class separation | `ArtifactClassPolicy` | `adopt` | Static DAG, executed run, freshness, semantic, lineage, table-state, and graph-rebuild artifacts must not substitute for each other. | Substitution fixture rejects with stable error. |

##### Concepts that must not transfer (dbt artifacts)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Freshness artifact as completeness proof | Freshness is not scoped source completeness or absence proof. | `EvaluateSourceCompleteness`. |
| Manifest as executed run evidence | Static DAG does not prove execution. | `RunDatasetIOContract` plus `VersionManifest`. |

Source basis: dbt official artifact documentation was inspected.[^36]

### 4.9 Cadastre-adjacent graph systems

This section uses existing Cadastre research reports for context and does not treat those systems as governing architecture. Primary sources were not re-inspected in this run unless needed to support temporal/replay/projection questions.

#### 4.9.1 Cartography

| Required subsection | Analysis |
| --- | --- |
| Source overview | Cartography is a graph-backed asset inventory/security exposure tool using provider modules and Neo4j. Its non-purpose for Cadastre is lakehouse-authoritative bitemporal fact storage. |
| Architecture and operational model | Provider modules collect, transform, load, and cleanup graph nodes/relationships; Neo4j is the serving/storage graph. |
| Data, schema, and temporal model | Update tags, graph node IDs, and cleanup scopes are graph synchronization metadata, not Cadastre valid/known time. |
| Correction, late-arrival, replay, or rebuild behavior | Direct graph cleanup is a cautionary reference for why Cadastre graph changes must be derived and replayable from lakehouse state. |
| Interfaces and extension points | Provider modules and schema-driven Cypher generation are useful implementation references, not authority models. |
| Validation and tests | Existing report inspected representative tests, but this run did not execute them. |

##### Confirmed findings versus inferences (Cartography)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| The existing Cartography report identifies provider modules, graph schema, Cypher generation, cleanup, and Neo4j-backed graph storage. | Direct-to-graph ingestion can make cleanup and identity graph-state-dependent. | Cadastre must keep graph as projection and require graph rebuild from lakehouse refs. | Schema-driven graph query generation can inform graph delta validation. | Current Cartography primary sources were not re-inspected here. |

##### Transfer candidates (Cartography)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Schema-driven graph writes | `GraphReadModelSchemaProfile` | `adapt` | Use only after graph deltas are derived from gold facts. | Backend schema preflight passes before apply. |
| Cleanup safety concerns | `GraphApplyProfile` | `cautionary_reference` | Cleanup/expire must be delta-driven and idempotent. | Direct graph cleanup attempt rejects. |

###### Concepts that must not transfer (Cartography)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Direct-to-Neo4j ingestion | It bypasses raw/silver/gold and replay manifest. | Lakehouse-first processing and graph delta projection. |
| Update tag as fact time | It is sync metadata. | Cadastre valid/known times. |

Source basis: Existing Cadastre Cartography research report was used as context; no Cartography repository execution occurred in this run.[^14]

##### 4.9.2 JupiterOne / Starbase / JupiterOne SDK

| Required subsection | Analysis |
| --- | --- |
| Source overview | JupiterOne/Starbase and the SDK are graph-security integration references with graph object outputs and SDK step patterns. |
| Architecture and operational model | Starbase runs integrations and pushes graph objects to Neo4j/JupiterOne; the SDK uses integration steps and graph entities/relationships. |
| Data, schema, and temporal model | Provider-scoped graph keys and graph object lifecycle timestamps are graph synchronization metadata, not Cadastre canonical identity or bitemporal facts. |
| Correction, late-arrival, replay, or rebuild behavior | Backend diffing/sync is not Cadastre replay. Cadastre must produce deterministic graph deltas from gold facts. |
| Interfaces and extension points | Step DAG and testing models are useful for source packages, but output must be raw/silver/gold/projection separated. |
| Validation and tests | Existing report inspected SDK docs and source snippets; no local SDK execution occurred. |

###### Confirmed findings versus inferences (JupiterOne / Starbase / JupiterOne SDK)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Existing JupiterOne report identifies graph-first outputs, SDK steps, provider keys, mapped relationships, and graph synchronization behavior. | Graph keys are useful stable local identifiers but unsafe as canonical identity. | Cadastre must reject graph-object sync as graph mutation authority. | SDK step testing can inform adapter validation. | Primary sources were not re-inspected in this run. |

###### Transfer candidates (JupiterOne / Starbase / JupiterOne SDK)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Integration step DAG | `SourceStageDAG` / package authoring | `adapt` | Step outputs must obey Cadastre stage output permissions. | Forbidden graph output fixture rejects. |
| Graph object testing | `ValidationMatrix` | `adapt` | Expected output tests must target raw/silver/gold/delta contracts, not graph truth. | Golden output validation passes exact checksums. |

###### Concepts that must not transfer (JupiterOne / Starbase / JupiterOne SDK)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| Provider-scoped `_key` as canonical identity | Graph key equality is not entity resolution. | `IdentityDecision`. |
| Backend graph diff as mutation authority | It bypasses lakehouse facts and replay refs. | `GraphDeltaSet` plus `GraphApplyProfile`. |

Source basis: Existing Cadastre JupiterOne research report was used as context; no JupiterOne repository execution occurred in this run.[^15]

##### 4.9.3 BloodHound / SharpHound / OpenGraph

| Required subsection | Analysis |
| --- | --- |
| Source overview | BloodHound/OpenGraph are high-value graph and path semantics references, especially for traversability, stable IDs, and deletion/source-kind hazards. |
| Architecture and operational model | BloodHound serves a graph and pathfinding experience over ingested collector/OpenGraph data. SharpHound collects AD-oriented data. |
| Data, schema, and temporal model | OpenGraph node IDs, endpoint matching, traversability flags, source kind, and collector metadata are external graph/collector concepts, not Cadastre identity or fact authority. |
| Correction, late-arrival, replay, or rebuild behavior | Source-kind deletion and post-processed edge regeneration are cautionary references for projection-owned output and deletion hazards. |
| Interfaces and extension points | Traversability can inform `GraphEdgeSemantics`; OpenGraph payloads can inform graph fixture design. |
| Validation and tests | Existing report inspected docs/source metadata; no BloodHound or SharpHound execution occurred in this run. |

###### Confirmed findings versus inferences (BloodHound / SharpHound / OpenGraph)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Existing BloodHound report identifies stable IDs, endpoint matching, traversability, source-kind deletion, post-processed edges, and graph/pathfinding boundaries. | Graph traversability is analysis eligibility, not fact truth. | Cadastre should adopt explicit edge semantics but reject graph as authority. | Traversability taxonomy can inform analysis rules. | Current primary sources were not re-inspected here. |

###### Transfer candidates (BloodHound / SharpHound / OpenGraph)

| Source concept | Cadastre target | Transfer stance | Required Cadastre contract behavior | Acceptance evidence |
| --- | --- | --- | --- | --- |
| Traversability | `GraphEdgeSemantics` | `adapt` | Edge traversability must be explicit, evidence-backed, and non-implicative. | Edge semantics validation covers every edge type. |
| Post-processed edges | `DerivationRuleBundle` / `GraphDeltaSet` | `adapt` | Generated edges must have supporting fact refs and deterministic rule versions. | Rebuild regenerates identical generated edges. |

###### Concepts that must not transfer (BloodHound / SharpHound / OpenGraph)

| Source concept | Why unsafe for Cadastre | Cadastre-safe alternative |
| --- | --- | --- |
| OpenGraph endpoint matching as identity | Endpoint matching is graph ingest/linking, not resolver authority. | `UnresolvedTargetReference` and `IdentityDecision`. |
| Source-kind deletion as authoritative cleanup | It can delete shared/referenced graph data incorrectly. | Source-scoped graph delta expiration through projection profile. |

Source basis: Existing Cadastre BloodHound/OpenGraph research report was used as context; no BloodHound runtime was executed in this run.[^13]

## 5. Cross-source comparison

### 5.1 Temporal model comparison

| External concept | Source families | Meaning in source | Cadastre analogue | Cadastre default behavior | Must not be confused with |
| --- | --- | --- | --- | --- | --- |
| event time | Flink, Beam, Kafka Streams | Time attached to source event for event-time processing. | Candidate source occurrence time. | Accept only when source event time is authoritative and `ResolveFactTime` validates it. | Processing time, ingestion time, known time, watermark. |
| processing time | Stream systems | Runtime clock of processor. | None by default. | May appear as diagnostic only. | Source event time, known time. |
| ingestion time | Stream/log systems | Time record entered processing system. | `RawRecord.ingested_at`. | Persist as platform persistence time only. | Valid time, known time, source event time. |
| collected time | Cadastre/source adapters | Time collector observed or fetched source record. | `RawRecord.collected_at`, silver `collected_at`. | Preserve as collection metadata. | Source event time, known time. |
| normalized time | Cadastre normalizer | Time silver observation was produced. | `CadastreSilverObservation.normalized_at`. | Preserve as stage output time only. | Known time, valid time. |
| valid time | XTDB, Cadastre, bitemporal systems | Time fact is true in the domain. | `GoldFact.valid_from`, `valid_to`. | Required for gold facts; half-open interval. | Table snapshot time, transaction time, CDC log time. |
| system time | XTDB | Time database/system knew a fact. | Possible input to knowledge-time import only. | Rejected as Cadastre known time unless `KnowledgeTimeImportPolicy` permits. | Source event time, table commit time. |
| transaction time | Datomic, databases | Time/order of database transaction. | Possible knowledge evidence, not default. | Non-authoritative for Cadastre fact time by default. | Valid time, known time. |
| known time | Cadastre | Time Cadastre knowledge assertion is current. | `GoldFact.known_from`, `known_to`. | Computed by Cadastre policy; not inherited from external progress by default. | Source occurrence, connector timestamp, table commit time. |
| source occurrence time | Sources/CDC payloads | Time event occurred in source domain. | Candidate source event time. | Valid only if parsed and authoritative. | CDC offset, schema-history timestamp. |
| connector time | Debezium/Kafka Connect | Time connector processed or emitted event. | Diagnostic/runtime metadata. | Must not set fact time by default. | Source occurrence time, known time. |
| snapshot time | CDC/table formats | Time snapshot/read was taken or table snapshot was created. | Table/read state ref metadata. | Must not set source event time, valid time, or known time. | `LakehouseSnapshotRef` identity. |
| table commit time | Iceberg, Delta, Hudi | Time table commit/log/instant occurred. | `LakehouseCommitRef` metadata. | Table-state only. | Known time, valid time, source event time. |
| catalog commit time | Nessie/lakeFS/catalogs | Time catalog commit occurred. | Catalog state metadata. | Must be paired with immutable table-format refs to affect replay. | Production approval, known time. |
| CDC offset/log position time | Debezium/Kafka Connect | Log ordering/resume position. | `StageStateRecord` and `CDCReplayStateContract`. | Required for replay state when output-affecting. | Source event time, valid time, completeness. |
| schema-history time | Debezium/Kafka Connect | Schema evolution/replay interpretation state. | Schema-history ref. | Required for CDC replay; non-authoritative for fact time. | Source occurrence time. |
| heartbeat time | CDC/streams | Liveness/progress signal. | Diagnostic or source completeness evidence row with authority limit. | No absence/retraction/cleanup authority by default. | Source completeness. |
| watermark time | Flink/Beam/streams | Progress estimate for event-time processing. | Watermark candidate. | May advance only through `ProjectionWatermarkPolicy` and completeness gates. | Absence proof, source authority. |
| completion frontier | Differential Dataflow | Computation progress frontier. | Computation progress candidate. | Non-authoritative by default. | Source completeness, cleanup authority. |
| graph apply time | Cadastre graph backend | Time graph apply happened. | `GraphApplyResult.applied_at`. | Graph-serving state only. | Gold known time, source event time. |
| projection watermark time | Projection systems | Derived output progress. | `WatermarkCommitRecord`. | Advance only after completeness, delta validation, apply success, and policy. | Source watermark, source completeness. |
| replay execution time | Replay/workflows | Time replay was run. | Replay diagnostic. | Excluded from equivalence checksum by default unless policy says otherwise. | Known time, valid time. |

#### 5.2 Correction model comparison

| Scenario | XTDB/Datomic pattern | Stream-time pattern | CDC pattern | Event-sourcing pattern | Lakehouse pattern | Required Cadastre behavior |
| --- | --- | --- | --- | --- | --- | --- |
| same fact repeated with new evidence | Same assertion or duplicate datom can be no-op or new transaction metadata. | Duplicate/later pane may update aggregate. | Duplicate change event or reprocessed offset possible. | Idempotent event ID no-ops. | Same row/input may be read again. | Emit deterministic `noop` when fact content and interval are unchanged; preserve evidence link if policy requires. |
| current fact superseded by later valid time | New valid interval replaces current domain state. | Later event updates window/current state. | Update event carries new values. | Append new event. | Commit new table version. | Close previous valid interval, emit new asserted fact, preserve known-time chain. |
| late historical fact with no conflict | Insert backdated valid assertion at later system time. | Late record updates prior window. | Historical CDC or snapshot record arrives after stream. | Append correction event. | New commit includes historical correction row. | Accept as correction; split intervals if needed; graph delta may update history/current only if projection scope affected. |
| late historical fact with conflict | Both histories queryable by system/known time. | Late record may be side-output/dropped depending policy. | Conflicting event order possible. | Append conflict event. | New table version with conflict rows. | Mark conflicted or quarantine according to source authority; never overwrite prior row. |
| retraction due to source delete | Retraction asserted at transaction time. | Delete/update event can retract aggregate. | Delete/tombstone emitted. | Append delete/retract event. | Delete file/remove action or new rows. | Treat delete as evidence only; retract only if source authority and completeness permit. |
| retraction due to complete authorized absence | Not a default DB concept. | Watermark alone insufficient. | Snapshot complete with absence possible only if profiled. | Append absence/retraction event. | Table scan/snapshot evidence. | Require `SourceCompletenessProfile`, receipt state, and authority. |
| stale fact with no negative evidence | Old assertion remains historically true/current until policy marks stale. | Late/no input may let state expire. | No CDC update. | No new event. | No table change. | Mark stale only by Cadastre staleness policy; do not retract. |
| conflicting authoritative sources | Multiple assertions at same valid interval. | Conflicting event streams. | Different source logs disagree. | Append conflict evidence. | Multiple table inputs. | Emit `conflicted` facts and block projection if edge semantics require unambiguous assertion. |
| ambiguous source time | Requires explicit valid time or query fallback. | Event-time parsing may fail. | Connector timestamp may exist but not occurrence. | Event metadata ambiguous. | Table time may exist. | Resolve to error, quarantine, or fallback only if `TemporalSemanticsPolicy` permits. |
| imported historical knowledge | Backdated valid facts with later system time. | Historical replay stream. | Snapshot plus log import. | Append import event. | Historical table snapshots. | Apply `KnowledgeTimeImportPolicy` and monotonic known-time guard. |
| correction after graph projection | Derived projection must update/read model. | Updated pane emitted. | Downstream consumer updates. | Projection checkpoint advances after events. | New graph delta table commit. | Emit graph delta correction or deterministic no-op; graph backend not source of truth. |
| correction after graph apply failure | Retry from checkpoint. | State retry. | Offset replay. | Parked/retry message. | Commit refs protect inputs. | Resume with idempotency key; do not advance projection watermark until success. |
| correction after table maintenance candidate | Maintenance may delete old history. | Not applicable. | Log purge can remove replay. | Event retention risk. | Snapshot expiration/vacuum/cleaning risk. | Maintenance must refuse when protected manifests/rebuild/replay refs would be invalidated. |

#### 5.3 Replay and rebuild sufficiency matrix

| Output class | Required replay inputs | Required manifest refs | Required retention refs | Allowed volatile exclusions | Production replay failure if missing |
| --- | --- | --- | --- | --- | --- |
| raw persistence | exact source payload bytes or fixture in validation mode; source instance config; adapter version; call policy; state refs | source instance, config hash, adapter, call policy, `RawRecord` refs, `StageStateRecord` | raw payload retention and table snapshot refs | wall-clock start/end diagnostics | `RAW_INPUT_UNAVAILABLE` or `VERSION_MANIFEST_MISMATCH` |
| silver observation derivation | raw refs, parser, mapping bundle, external schema artifact, source extension rules | parser/mapping versions, schema artifact, validation output checksum | raw and silver table refs | normalized_at when policy excludes | `EXTERNAL_SCHEMA_ARTIFACT_MISMATCH`, `MAPPING_ARTIFACT_MISMATCH` |
| temporal source-time resolution | raw/silver input, temporal policy, version manifest, import policy | temporal policy, source config, schema refs | raw/silver refs | diagnostic processing time | `TEMPORAL_RESOLUTION_ERROR` |
| identity decision | silver inputs, resolver profile, evidence class registry, negative evidence | resolver version/profile, selector policy, validation scenarios | identity/silver/raw refs | review UI diagnostics | `IDENTITY_REPLAY_INPUT_MISSING` |
| gold fact derivation | silver, identity, coverage, authority profile, temporal resolution, correction policy | source authority, coverage assertions, correction policy, derivation bundle | gold/silver/raw/table refs | persistence timestamp when excluded | `GOLD_REPLAY_INPUT_MISSING` |
| correction change set | comparable existing facts, candidate fact, correction policy, old/new snapshot refs | correction policy, old fact refs, new candidate refs, snapshot refs | protected old/new gold refs | change-set diagnostic text | `CORRECTION_INPUT_MISSING` |
| graph delta generation | gold facts, graph projection profile, edge semantics, taxonomy policy, schema profile | graph profile, edge semantics, taxonomy, source fact refs | graph delta table refs and gold refs | projection execution time | `GRAPH_DELTA_INPUT_MISSING` |
| graph apply result | graph delta set, apply profile, read-model schema profile, prior apply result | delta set, apply profile, schema profile, apply result | graph apply refs and delta refs | backend latency metrics | `GRAPH_APPLY_REPLAY_INPUT_MISSING` |
| graph rebuild | authoritative lakehouse snapshots, graph profiles, apply/schema profiles, rebuild policy | `GraphRebuildManifest`, `LakehouseSnapshotRef`, table-set checksum, `GraphIndexConsistencyCheck` | protected snapshot/table-set refs | rebuild runtime duration | `GRAPH_REBUILD_INPUT_MISSING` |
| CIM or external projection | silver/gold inputs, projection profile, loss manifest | CIM profile, schema artifact, projection loss manifest | projection output refs and source refs | consumer delivery time | `PROJECTION_REPLAY_INPUT_MISSING` |
| analysis rule output | graph/lakehouse query input refs, rule bundle, compatibility matrix | analysis rule checksums, graph profile, lag policy | query input refs | display ordering diagnostics if excluded | `ANALYSIS_RULE_INPUT_MISSING` |
| source completeness decision | receipt, evidence rows, completeness profile, coverage state | receipt IDs, evidence hashes, profile version | completeness/coverage refs | observer diagnostic clock | `COMPLETENESS_INPUT_MISSING` |
| maintenance eligibility decision | maintenance candidates, protected manifests, table refs, retention/legal holds | retention policy, maintenance policy, candidate set checksum | protected snapshot/commit/file refs | estimated savings | `REPLAY_RETENTION_INPUT_MISSING` |

#### 5.4 Watermark, completeness, and absence authority

| Signal | Can indicate progress? | Can authorize absence by default? | Can authorize retraction by default? | Can authorize cleanup by default? | Can advance source watermark by default? | Required Cadastre gate |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| stream watermark | yes | no | no | no | no | `SourceCompletenessProfile` plus `WatermarkCommitRecord` |
| Beam/Flink/Kafka grace or allowed lateness | yes | no | no | no | no | `LateArrivalPolicy` only |
| Differential Dataflow completion/frontier | yes | no | no | no | no | computation progress gate only |
| CDC heartbeat | yes | no | no | no | no | CDC state plus completeness profile |
| CDC offset/log position | yes | no | no | no | no | `CDCReplayStateContract` |
| schema-history availability | yes for parse readiness | no | no | no | no | schema compatibility and replay sufficiency |
| page exhaustion | yes | profile-defined only | profile-defined only | profile-defined only | profile-defined only | `SourceCompletenessReceipt` evaluation |
| expected count match | yes | profile-defined only | profile-defined only | profile-defined only | profile-defined only | count evidence row plus profile |
| empty API response | weak progress | no | no | no | no | source-declared complete or profile rule |
| source-declared empty complete | yes | profile-defined only | profile-defined only | profile-defined only | profile-defined only | receipt state `empty_complete` plus profile |
| table snapshot ref | yes for table state | no | no | no | no | `LakehouseSnapshotRef` only |
| table commit success | yes for write state | no | no | no | no | `LakehouseCommitRef` plus output contract |
| ack success | yes for delivery | no | no | no | no | acknowledgment evidence row only |
| queue drain | yes for runtime | no | no | no | no | ingestion provenance only |
| provenance closure | yes for lineage | no | no | no | no | `IngestionProvenanceEvent` only |
| graph apply success | yes for projection | no | no | no | yes for projection only | `ProjectionWatermarkPolicy` |
| graph rebuild success | yes for derived view | no | no | no | no for source watermark | `GraphRebuildManifest` and consistency check |
| lineage run complete | yes for pipeline | no | no | no | no | `RunDatasetIOContract` only |
| dbt freshness artifact | yes for test result | no | no | no | no | `ArtifactClassPolicy` |
| live probe zero-row result | weak progress | no | no | no | no | source exploration only unless future contract permits |

#### 5.5 Graph projection and idempotency comparison

| Source pattern | Cadastre graph implication | Required graph delta behavior | Required graph apply behavior | Failure/retry behavior |
| --- | --- | --- | --- | --- |
| projection-owned streams | Useful as derived read-model pattern only. | Deltas must reference source gold facts and projection profile. | Apply must persist result and checkpoint. | Reset/rebuild from lakehouse refs, not from graph backend. |
| graph backend mutation | Cautionary only. | Backend mutations must never create authoritative facts. | Backend writes only from validated deltas. | Missing schema or partial apply emits `GraphApplyResult` error. |
| idempotent event ID | Transfer to delta idempotency. | `GraphDeltaIdempotencyKey` deterministic from delta set/profile/fact refs. | Reapply same key returns deterministic no-op or same result. | Duplicate apply cannot duplicate nodes/edges. |
| expected revision | Transfer to optimistic graph apply preconditions where backend supports. | Delta includes expected prior projection watermark/apply ref. | Mismatch rejects before mutation. | Retry recomputes/resumes from persisted result. |
| checkpoint | Required for graph apply/rebuild. | Delta set order and checkpoint key deterministic. | Persist after successful batch boundary only. | Failed checkpoint resumes from last successful persisted batch. |
| parked message | Persisted error model. | Failed delta entry records stable error. | Apply result includes parked/retryable/nonretryable status. | Retry only eligible parked deltas. |
| partial failure | Must be explicit and non-advancing. | Delta set unchanged. | `GraphApplyResult.status = partial_failed`; no watermark advance. | Resume with idempotency and prior result. |
| reset from beginning | Graph rebuild. | Regenerate all deltas from authoritative refs/profile. | Apply into rebuilt derived view after schema preflight. | Consistency check gates serving. |
| positive/negative diff | Correction algebra. | `upsert`, `expire`, `retract`, `noop` signed effect. | Deterministic apply order by operation class and ID. | Replayed correction yields same effects. |
| graph rebuild from authoritative state | Required default for disaster recovery. | Deltas/rebuild output derived from lakehouse refs. | Backend import is target operation only. | Same inputs produce same rebuild checksum. |

Source basis for cross-source comparison: external primary docs/specs and Cadastre PRD sections cited in Sections 2 through 4 were used; no external runtime behavior was observed.[^1][^2][^4][^6][^7][^8][^9][^10][^16][^17][^18][^19][^20][^21][^22][^23][^25][^26][^27][^28][^29][^30][^31][^32][^33][^36]

## 6. Recommended PRD improvements

The PRD already names many of the right contracts. This section recommends refinements and new contracts only where the current PRD leaves observable behavior open to divergent implementations.

### 6.1 `TemporalSemanticsPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines which time fields may influence valid time, known time, source observation time, late-arrival routing, and replay diagnostics. |
| Observable behavior | The implementation must reject any attempt to use connector time, CDC offset, table snapshot time, watermark time, graph apply time, or workflow replay time as valid or known time unless the policy contains an explicit permission row. |
| Interfaces | Required fields: `policy_id`, `policy_version`, `status`, `source_dataset`, `allowed_source_time_fields[]`, `fallback_behavior`, `historical_import_mode`, `ambiguous_time_behavior`, `malformed_time_behavior`, `monotonic_known_time_required`, `checksum`. |
| Algorithms | Used by `ResolveFactTime`. The policy must be evaluated before a `GoldFact` candidate is created. |
| Defaults | Default `fallback_behavior = reject`; default `ambiguous_time_behavior = error`; default `malformed_time_behavior = quarantine`; default `monotonic_known_time_required = true`. |
| Errors | `SOURCE_TIME_ABSENT`, `SOURCE_TIME_MALFORMED`, `SOURCE_TIME_AMBIGUOUS`, `SOURCE_TIME_NOT_AUTHORIZED`, `TEMPORAL_FALLBACK_REJECTED`, `KNOWN_TIME_MONOTONICITY_ERROR`. |
| Interactions | Composes with `RawRecord.source_event_time_quality`, `CadastreSilverObservation.observed_at_quality`, `VersionManifest`, `GoldFactCorrectionPolicy`, and `KnowledgeTimeImportPolicy`. |
| Non-authority boundaries | Must not authorize absence, retraction, cleanup, graph mutation, replay success, or source completeness. |
| Acceptance criteria | Given the same raw/silver rows and policy, two implementations must emit byte-identical `TemporalObservationTimeResolution` rows or the same stable error code. |

#### 6.2 `TemporalObservationTimeResolution`

| Field | Required content |
| --- | --- |
| Purpose | Records the deterministic result of resolving source time into candidate fact valid-time and known-time inputs. |
| Observable behavior | The implementation must persist one row for every gold-candidate observation or emit a deterministic error before gold derivation. |
| Interfaces | Required fields: `resolution_id`, `raw_record_id`, `observation_id`, `policy_id`, `source_time_input_path`, `source_time_quality`, `resolved_valid_from`, `resolved_valid_to`, `resolved_known_from`, `resolution_mode`, `fallback_used`, `checksum`, `errors[]`. |
| Algorithms | Produced by `ResolveFactTime`; checksum computed from canonical JSON excluding diagnostic wall-clock fields. |
| Defaults | `resolved_valid_to = null` means open interval; `fallback_used = false`; no implicit current time fallback. |
| Errors | Same as `TemporalSemanticsPolicy` plus `INTERVAL_INVALID` and `RESOLUTION_CHECKSUM_MISMATCH`. |
| Interactions | Input to `GoldFact`, `ApplyGoldCorrection`, and replay equivalence. Must appear in `VersionManifest` when output-affecting. |
| Non-authority boundaries | Must not by itself authorize a fact; authority remains with `SourceAuthorityProfile`. |
| Acceptance criteria | All error/no-error outcomes and checksums match across replay. |

#### 6.3 `LateArrivalPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines how late observations or facts are routed before and after allowed lateness cutoffs. |
| Observable behavior | Production default must preserve late raw evidence and must not silently discard authoritative late records. |
| Interfaces | Required fields: `policy_id`, `dataset_scope`, `allowed_lateness_seconds`, `after_cutoff_authoritative_action`, `after_cutoff_non_authoritative_action`, `watermark_source`, `quarantine_profile_id`, `shadow_profile_id`, `checksum`. |
| Algorithms | Used by `EvaluateLateArrival`. |
| Defaults | `allowed_lateness_seconds = 0`; authoritative after-cutoff default `accept_as_correction`; non-authoritative after-cutoff default `quarantine`; discard is forbidden in production except validation-only profiles. |
| Errors | `LATE_ARRIVAL_POLICY_MISSING`, `LATE_ARRIVAL_AUTHORITY_UNRESOLVED`, `LATE_ARRIVAL_DISCARD_FORBIDDEN`. |
| Interactions | Requires `SourceCompletenessProfile`, `SourceAuthorityProfile`, temporal resolution, and watermark state. |
| Non-authority boundaries | Watermark or grace-period state must not authorize absence, cleanup, or retraction. |
| Acceptance criteria | The event corpus cases for late authoritative and late non-authoritative evidence produce the required route. |

#### 6.4 `GoldFactCorrectionPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines deterministic comparable-fact matching and append-only correction behavior. |
| Observable behavior | Existing `GoldFact` rows must never be overwritten in place. Corrections must emit `GoldFactChangeSet` rows containing new facts, known-time closures, no-ops, or errors. |
| Interfaces | Required fields: `policy_id`, `fact_type`, `predicate`, `comparable_key_fields[]`, `overlap_behavior`, `confidence_change_behavior`, `delete_evidence_behavior`, `conflict_behavior`, `stale_behavior`, `sort_key`, `checksum`. |
| Algorithms | Used by `ApplyGoldCorrection`. |
| Defaults | `delete_evidence_behavior = evidence_only`; `confidence_change_behavior = emit_new_known_state`; `conflict_behavior = mark_conflicted`; `sort_key = fact_type, subject_id, predicate, object_id, valid_from, known_from, fact_id`. |
| Errors | `CORRECTION_POLICY_MISSING`, `COMPARABLE_FACT_AMBIGUOUS`, `UNAUTHORIZED_RETRACTION`, `INTERVAL_OVERLAP_CONFLICT`, `CORRECTION_SNAPSHOT_REF_MISSING`. |
| Interactions | Requires `TemporalSemanticsPolicy`, `SourceAuthorityProfile`, `CorrectionSnapshotRefPolicy`, and `VersionManifest`. |
| Non-authority boundaries | Must not derive source absence or retraction from CDC tombstones, watermarks, or table snapshots without authority/completeness gates. |
| Acceptance criteria | Correction corpus cases produce byte-identical `GoldFactChangeSet` rows and exact expected assertion states. |

#### 6.5 `KnowledgeTimeImportPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Governs historical imports where valid time predates Cadastre ingestion or prior known-time state. |
| Observable behavior | Historical imports must not create known-time intervals earlier than Cadastre knew the imported evidence unless the policy explicitly declares imported-known-time reconstruction. |
| Interfaces | Required fields: `policy_id`, `import_mode`, `source_known_time_field`, `known_time_floor`, `monotonic_guard`, `as_known_then_allowed`, `audit_label`, `checksum`. |
| Algorithms | Used by `ResolveFactTime` and `ApplyGoldCorrection`. |
| Defaults | `import_mode = current_knowledge_import`; `as_known_then_allowed = false`; `monotonic_guard = true`. |
| Errors | `HISTORICAL_IMPORT_POLICY_MISSING`, `KNOWN_TIME_IMPORT_REJECTED`, `KNOWN_TIME_MONOTONICITY_ERROR`. |
| Interactions | Composes with `TemporalSemanticsPolicy` and `BitemporalQueryMode`. |
| Non-authority boundaries | Must not assert that Cadastre knew something before evidence was imported unless governance explicitly permits reconstruction. |
| Acceptance criteria | Historical import case emits expected `known_from` and fails monotonicity violation. |

#### 6.6 `BitemporalQueryMode`

| Field | Required content |
| --- | --- |
| Purpose | Defines query modes for current, valid-at, known-at, corrected-history, and audit reconstruction. |
| Observable behavior | Every query mode must define assertion-state inclusion, interval filters, default times, ordering, and page-token behavior. |
| Interfaces | Required fields: `mode_id`, `valid_time_selector`, `known_time_selector`, `assertion_state_filter`, `include_superseded`, `include_retracted`, `sort_order`, `default_time_behavior`, `checksum`. |
| Algorithms | Query filters must use half-open intervals: `valid_from <= t < valid_to_or_infinity` and `known_from <= t < known_to_or_infinity`. |
| Defaults | Current mode defaults to current platform time for both valid and known time; audit mode must require explicit known time. |
| Errors | `BITEMPORAL_QUERY_MODE_UNSUPPORTED`, `QUERY_TIME_INVALID`, `ASSERTION_STATE_FILTER_INVALID`. |
| Interactions | Applies to asset search, detail, graph query, evidence drillback, and audit reconstruction. |
| Non-authority boundaries | Query mode changes visibility only; it must not mutate facts. |
| Acceptance criteria | Query fixtures return exact row IDs and deterministic ordering for all modes. |

#### 6.7 `ReplayEquivalencePolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines when a replay is production-equivalent, shadow-only, or rejected. |
| Observable behavior | Production replay must reject missing, unresolved, mismatched, retention-ineligible, schema-incompatible, or authority-mismatched output-affecting inputs before writing output. |
| Interfaces | Required fields: `policy_id`, `output_class`, `included_manifest_fields[]`, `excluded_volatile_fields[]`, `field_hash_algorithms`, `failure_precedence[]`, `shadow_allowed`, `checksum`. |
| Algorithms | Used by `DecideReplayMode` and `ComputeReplayEquivalenceChecksum`. |
| Defaults | Production exact mode requires byte-equivalent included fields and immutable table refs; shadow mode required for any changed output-affecting input. |
| Errors | `VERSION_MANIFEST_MISMATCH`, `LAKEHOUSE_SNAPSHOT_REF_UNRESOLVED`, `SCHEMA_COMPATIBILITY_ERROR`, `REPLAY_RETENTION_INELIGIBLE`, `AUTHORITY_PROFILE_MISMATCH`. |
| Interactions | Composes with `VersionManifest`, lakehouse refs, CDC state, schema artifacts, mapping/authority/projection profiles. |
| Non-authority boundaries | Replay success must not imply source completeness or graph truth. |
| Acceptance criteria | Replay mode matrix returns the expected mode/error for all required output classes. |

#### 6.8 `ReplayInputSufficiencyCheck`

| Field | Required content |
| --- | --- |
| Purpose | Performs deterministic preflight over all output-affecting replay inputs. |
| Observable behavior | The implementation must evaluate the most specific available missing/mismatch/retention/schema/authority error before generic mismatch. |
| Interfaces | Required fields: `check_id`, `run_id`, `requested_output_class`, `required_inputs[]`, `resolved_inputs[]`, `missing_inputs[]`, `mismatched_inputs[]`, `retention_failures[]`, `schema_failures[]`, `authority_failures[]`, `result`, `checksum`. |
| Algorithms | Used by `DecideReplayMode`; stable sorted diagnostics. |
| Defaults | Empty required input list is invalid for production output. |
| Errors | Specific errors from replay sufficiency matrix, otherwise `VERSION_MANIFEST_MISMATCH`. |
| Interactions | Requires `ReplayEquivalencePolicy`, `VersionManifest`, `LakehouseSchemaCompatibilityCheck`, and `ReplayRetentionPolicy`. |
| Non-authority boundaries | This check permits or rejects replay output; it does not validate source truth. |
| Acceptance criteria | Missing input cases produce stable error precedence and sorted diagnostics. |

#### 6.9 `DeterministicSideEffectRecord`

| Field | Required content |
| --- | --- |
| Purpose | Records nondeterministic values that can affect production output. |
| Observable behavior | Runtime randomness, external calls, clock reads, generated IDs, unordered iteration choices, or backend-discovered values that affect output must be recorded or rejected. |
| Interfaces | Required fields: `side_effect_id`, `run_id`, `stage_id`, `side_effect_kind`, `canonical_input_hash`, `recorded_value`, `recorded_value_hash`, `producer_version`, `checksum`. |
| Algorithms | Replay must read the recorded value and must not recompute unless shadow mode is active. |
| Defaults | No side effects allowed unless declared in the active stage or policy. |
| Errors | `SIDE_EFFECT_UNDECLARED`, `SIDE_EFFECT_RECORD_MISSING`, `SIDE_EFFECT_REPLAY_MISMATCH`. |
| Interactions | Included in `VersionManifest` when output-affecting. |
| Non-authority boundaries | Side-effect records are execution state, not source evidence. |
| Acceptance criteria | Replaying a recorded side effect yields identical output or fails before output. |

### 6.10 `ProjectionWatermarkPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines when source, projection, graph-apply, and presence-only watermarks may advance. |
| Observable behavior | Projection watermarks must not advance after failed or partial graph apply. Source watermarks must not advance unless completeness policy permits. |
| Interfaces | Required fields: `policy_id`, `watermark_kind`, `required_completeness_decision`, `required_delta_validation`, `required_apply_status`, `presence_only_allowed`, `max_lag_seconds`, `checksum`. |
| Algorithms | Used by `AdvanceProjectionWatermark`. |
| Defaults | No graph-serving watermark advance unless graph apply succeeded and consistency check passed. |
| Errors | `WATERMARK_ADVANCE_DENIED`, `WATERMARK_POLICY_MISSING`, `WATERMARK_SOURCE_COMPLETENESS_DENIED`, `GRAPH_APPLY_NOT_COMPLETE`. |
| Interactions | Composes with `SourceCompletenessProfile`, `GraphApplyResult`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy`. |
| Non-authority boundaries | Watermarks must not authorize absence, retraction, cleanup, or source authority unless separately permitted. |
| Acceptance criteria | Watermark cases 24 through 26 produce exact commit/no-op/error outputs. |

### 6.11 `WatermarkCommitRecord`

| Field | Required content |
| --- | --- |
| Purpose | Persists deterministic watermark advance, no-op, or refusal decisions. |
| Observable behavior | Every attempted production watermark advance must emit exactly one commit/no-op/error record. |
| Interfaces | Required fields: `watermark_commit_id`, `watermark_kind`, `prior_watermark`, `candidate_watermark`, `committed_watermark`, `decision`, `policy_id`, `input_refs[]`, `error_code`, `checksum`. |
| Algorithms | Produced by `AdvanceProjectionWatermark`. |
| Defaults | `decision = no_op` when candidate is not greater than prior; `decision = error` when required gates fail. |
| Errors | Same as `ProjectionWatermarkPolicy`. |
| Interactions | Appears in graph serving `derived_view_state` and `VersionManifest` when output-affecting. |
| Non-authority boundaries | Commit record proves watermark state only. |
| Acceptance criteria | Repeated advance with same inputs yields byte-identical no-op or commit record. |

### 6.12 `GraphDeltaIdempotencyKey`

| Field | Required content |
| --- | --- |
| Purpose | Provides stable idempotency for graph delta generation and apply. |
| Observable behavior | Applying the same delta set and profile twice must not create duplicate graph nodes/edges or advance watermark twice. |
| Interfaces | Required fields: `idempotency_key`, `graph_delta_set_id`, `projection_profile_id`, `apply_profile_id`, `source_fact_set_checksum`, `operation_sequence_checksum`, `target_schema_fingerprint`, `checksum`. |
| Algorithms | Key computed from canonical sorted delta operations and profile refs. |
| Defaults | Required for every production graph apply. |
| Errors | `GRAPH_DELTA_IDEMPOTENCY_KEY_MISSING`, `GRAPH_DELTA_IDEMPOTENCY_CONFLICT`, `GRAPH_REAPPLY_MISMATCH`. |
| Interactions | Used by `ResumeGraphApply` and `GraphApplyResult`. |
| Non-authority boundaries | Idempotency prevents duplicate graph mutation; it does not validate fact truth. |
| Acceptance criteria | Reapply fixture emits same result or no-op and no duplicate backend mutation. |

### 6.13 `CDCReplayStateContract`

| Field | Required content |
| --- | --- |
| Purpose | Defines the required CDC replay and parsing state for CDC collection patterns. |
| Observable behavior | CDC production replay must reject missing offset, log position, snapshot handoff, schema-history, connector config, or replica-identity compatibility inputs before output. |
| Interfaces | Required fields: `cdc_state_id`, `connector_kind`, `source_partition`, `snapshot_state`, `log_position_kind`, `log_position_value`, `schema_history_ref`, `transaction_metadata_ref`, `heartbeat_ref`, `commit_order_rule`, `checksum`. |
| Algorithms | Offset may commit only after required raw records, completeness receipts, and output-affecting state records are persisted. |
| Defaults | Heartbeats are liveness only; tombstones are raw delete evidence only. |
| Errors | `CDC_SCHEMA_HISTORY_MISSING`, `CDC_LOG_POSITION_UNAVAILABLE`, `CDC_SNAPSHOT_HANDOFF_MISSING`, `CDC_OFFSET_COMMIT_ORDER_ERROR`, `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED`. |
| Interactions | Composes with `RawRecord.source_event_subtype`, `StageStateRecord`, `VersionManifest`, and `ReplayInputSufficiencyCheck`. |
| Non-authority boundaries | CDC state cannot set valid time, known time, source completeness, retraction, cleanup, or graph mutation authority by default. |
| Acceptance criteria | CDC corpus cases for update, tombstone, missing schema history, and replay mismatch emit expected results. |

### 6.14 `CorrectionSnapshotRefPolicy`

| Field | Required content |
| --- | --- |
| Purpose | Requires old and new table-state refs for corrections that affect production output or replay. |
| Observable behavior | A correction must record the source snapshot/ref set used to read comparable prior facts and the snapshot/ref set used to write new correction outputs. |
| Interfaces | Required fields: `policy_id`, `fact_type`, `old_snapshot_roles[]`, `new_snapshot_roles[]`, `required_table_set_checksum`, `retention_protection_required`, `checksum`. |
| Algorithms | `ApplyGoldCorrection` must fail before output when required old/new refs are missing or mutable-branch-only. |
| Defaults | Required for all production gold corrections. |
| Errors | `CORRECTION_SNAPSHOT_REF_MISSING`, `CORRECTION_TABLE_SET_MISMATCH`, `MUTABLE_BRANCH_REF_FOR_REPLAY`. |
| Interactions | Requires `LakehouseSnapshotRef`, `LakehouseCommitRef`, `CrossTableCommitProfile`, and `ReplayRetentionPolicy`. |
| Non-authority boundaries | Table refs identify table state only; they do not define valid or known time. |
| Acceptance criteria | Correction replay with same old/new refs yields same change-set checksum. |

### 6.15 `EventSequenceValidationCorpus`

| Field | Required content |
| --- | --- |
| Purpose | Defines golden input/output cases for temporal, correction, replay, graph apply, and watermark behavior. |
| Observable behavior | Every implementation must run the corpus and produce exact expected deterministic fields, errors, no-ops, and checksums. |
| Interfaces | Required fields per case: `case_id`, `input_sequence`, `policies`, `manifest_refs`, `expected_time_resolution`, `expected_gold_rows`, `expected_graph_behavior`, `expected_replay_behavior`, `expected_errors_or_noops`, `expected_checksum`. |
| Algorithms | Cases feed `ResolveFactTime`, `EvaluateLateArrival`, `ApplyGoldCorrection`, `DecideReplayMode`, `ResumeGraphApply`, `AdvanceProjectionWatermark`, and `DecideMaintenanceEligibility`. |
| Defaults | Corpus must include all cases in Section 8 before active production release. |
| Errors | `EVENT_SEQUENCE_CASE_MISSING`, `EVENT_SEQUENCE_OUTPUT_MISMATCH`, `EVENT_SEQUENCE_CHECKSUM_MISMATCH`. |
| Interactions | Tied to `ValidationMatrix` and `ResolverActivationReport` where applicable. |
| Non-authority boundaries | Corpus validates behavior; it must not mutate production state. |
| Acceptance criteria | All active cases pass with byte-identical deterministic output. |

### 6.16 `GraphRebuildEquivalencePolicy`

| Field | Required content |
| --- | --- |
| Purpose | Defines deterministic equivalence for graph rebuilds from authoritative lakehouse state. |
| Observable behavior | Rebuilding graph serving state from the same authoritative lakehouse refs, profiles, schema, and apply settings must produce the same graph output checksum or a deterministic error. |
| Interfaces | Required fields: `policy_id`, `included_inputs[]`, `excluded_volatile_fields[]`, `node_checksum_fields[]`, `edge_checksum_fields[]`, `ordering_rule`, `consistency_check_required`, `checksum`. |
| Algorithms | Graph rebuild checksum computed from sorted canonical node/edge/evidence-index records and excluded-field report. |
| Defaults | Exclude backend physical IDs, backend write timestamps, and runtime duration; include graph IDs, types, properties, source fact refs, assertion state, confidence, valid/known intervals, profile refs. |
| Errors | `GRAPH_REBUILD_INPUT_MISSING`, `GRAPH_REBUILD_EQUIVALENCE_MISMATCH`, `GRAPH_INDEX_CONSISTENCY_FAILED`. |
| Interactions | Requires `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, `GraphReadModelSchemaProfile`, and `DerivedViewLagPolicy`. |
| Non-authority boundaries | Rebuild output is derived graph state only; it cannot create or mutate gold facts. |
| Acceptance criteria | Same refs/profile rebuild twice produces identical checksum; changed profile produces shadow-only or mismatch according to policy. |

## 7. Deterministic algorithms

All algorithms must use canonical JSON serialization with sorted keys, UTF-8, UTC timestamps with exactly 9 fractional digits, lowercase hex SHA-256, and stable lexical sorting unless a contract defines a narrower serialization.

### 7.1 ResolveFactTime

Inputs:

```text
RawRecord
CadastreSilverObservation
TemporalSemanticsPolicy
VersionManifest
```

Output:

```text
TemporalObservationTimeResolution or deterministic error
```

Pseudocode:

```text
ResolveFactTime(raw, obs, policy, manifest):
  require raw.raw_record_id == obs.raw_record_id else error TEMPORAL_INPUT_MISMATCH
  require policy.status == active else error TEMPORAL_POLICY_INACTIVE
  require manifest contains policy checksum and raw/silver table refs else error VERSION_MANIFEST_MISMATCH

  candidate = null
  source_path = null

  if raw.source_event_time_quality == source_time_valid:
    source_path = "RawRecord.source_event_time"
    candidate = raw.source_event_time
  else if obs.observed_at_quality == valid:
    source_path = "CadastreSilverObservation.observed_at"
    candidate = obs.observed_at
  else if raw.source_event_time_quality == source_time_absent or obs.observed_at_quality == absent:
    if policy.fallback_behavior == reject:
      return error SOURCE_TIME_ABSENT
    if policy.fallback_behavior == collected_at and policy.allows_field("RawRecord.collected_at"):
      source_path = "RawRecord.collected_at"
      candidate = raw.collected_at
    else:
      return error TEMPORAL_FALLBACK_REJECTED
  else if raw.source_event_time_quality == source_time_malformed or obs.observed_at_quality == malformed:
    return error SOURCE_TIME_MALFORMED
  else if raw.source_event_time_quality == source_time_ambiguous or obs.observed_at_quality == ambiguous:
    if policy.ambiguous_time_behavior == mark_ambiguous:
      return resolution with errors=[SOURCE_TIME_AMBIGUOUS], resolution_mode="ambiguous_no_fact_time"
    else:
      return error SOURCE_TIME_AMBIGUOUS
  else if raw.source_event_time_quality == source_time_not_authoritative:
    return error SOURCE_TIME_NOT_AUTHORIZED

  if source_path not in policy.allowed_source_time_fields:
    return error SOURCE_TIME_NOT_AUTHORIZED

  if candidate is null:
    return error TEMPORAL_RESOLUTION_ERROR

  valid_from = normalize_timestamp(candidate)
  valid_to = null

  if policy.historical_import_mode == imported_known_time:
    known_from_candidate = read_required(policy.source_known_time_field)
    if known_from_candidate is absent:
      return error KNOWN_TIME_IMPORT_REJECTED
    known_from = normalize_timestamp(known_from_candidate)
  else:
    known_from = manifest.started_at

  if policy.monotonic_known_time_required:
    prior_known = max_known_from_for(raw.source_instance_id, obs.observation_type)
    if prior_known exists and known_from < prior_known:
      return error KNOWN_TIME_MONOTONICITY_ERROR

  if valid_to != null and valid_to <= valid_from:
    return error INTERVAL_INVALID

  body = canonical_json({
    raw_record_id, observation_id, policy_id,
    source_time_input_path: source_path,
    source_time_quality: raw.source_event_time_quality,
    resolved_valid_from: valid_from,
    resolved_valid_to: valid_to,
    resolved_known_from: known_from,
    resolution_mode: selected mode,
    fallback_used: source_path not in [RawRecord.source_event_time, CadastreSilverObservation.observed_at]
  })
  checksum = sha256(body)
  return TemporalObservationTimeResolution(body + checksum)
```

Required behavior coverage:

| Input condition | Required output |
| --- | --- |
| valid source event time | Resolution with that time as `resolved_valid_from` when authorized. |
| absent source time | Error unless fallback allowed. |
| malformed source time | `SOURCE_TIME_MALFORMED`. |
| ambiguous source time | `SOURCE_TIME_AMBIGUOUS` or ambiguous resolution only if policy permits. |
| source time not authoritative | `SOURCE_TIME_NOT_AUTHORIZED`. |
| fallback rejected | `TEMPORAL_FALLBACK_REJECTED`. |
| fallback allowed | Resolution marks `fallback_used = true`. |
| historical import known time | Uses import policy source-known-time field and monotonic guard. |
| monotonic import guard | Rejects known-time regression. |
| half-open interval validation | Rejects `valid_to <= valid_from`. |
| checksum computation | Canonical body hash excluding non-output diagnostics. |

### 7.2 EvaluateLateArrival

Inputs:

```text
candidate observation or fact
source completeness state
watermark state
active LateArrivalPolicy
source authority profile
```

Output:

```text
accept_as_correction
mark_ambiguous
quarantine
shadow_only
reject
or deterministic error
```

Pseudocode:

```text
EvaluateLateArrival(candidate, completeness, watermark, policy, authority):
  require policy.status == active else error LATE_ARRIVAL_POLICY_MISSING
  require authority covers candidate fact type/source else error LATE_ARRIVAL_AUTHORITY_UNRESOLVED

  candidate_time = candidate.resolved_valid_from or candidate.observed_at
  if candidate_time is null:
    return mark_ambiguous

  cutoff = watermark.source_event_time - policy.allowed_lateness_seconds
  after_cutoff = candidate_time < cutoff

  authority_class = authority.classify(candidate.source_instance_id, candidate.fact_type)
  completeness_permission = completeness.permits_late_correction(candidate.scope)

  if not after_cutoff:
    if authority_class == authoritative:
      return accept_as_correction
    if authority_class == non_authoritative:
      return mark_ambiguous
    return quarantine

  if after_cutoff:
    if authority_class == authoritative:
      if policy.after_cutoff_authoritative_action == accept_as_correction:
        return accept_as_correction
      if policy.after_cutoff_authoritative_action == shadow_only:
        return shadow_only
      if policy.after_cutoff_authoritative_action == quarantine:
        return quarantine
      if policy.after_cutoff_authoritative_action == discard:
        return error LATE_ARRIVAL_DISCARD_FORBIDDEN
    if authority_class == non_authoritative:
      action = policy.after_cutoff_non_authoritative_action
      if action == quarantine:
        return quarantine
      if action == shadow_only:
        return shadow_only
      if action == reject:
        return reject
      if action == discard and policy.validation_only == true:
        return reject
      if action == discard:
        return error LATE_ARRIVAL_DISCARD_FORBIDDEN

  return quarantine
```

### 7.3 ApplyGoldCorrection

Inputs:

```text
comparable existing GoldFact rows
candidate fact
GoldFactCorrectionPolicy
TemporalSemanticsPolicy
```

Output:

```text
GoldFactChangeSet
```

Pseudocode:

```text
ApplyGoldCorrection(existing[], candidate, correction_policy, temporal_policy):
  require correction_policy.status == active else error CORRECTION_POLICY_MISSING
  require candidate interval is half-open else error INTERVAL_INVALID

  comparable = filter existing by correction_policy.comparable_key_fields
  sort comparable by (valid_from, valid_to nulls last, known_from, fact_id)

  changes = []
  errors = []

  if duplicate_fact_exists(comparable, candidate):
    changes.append(noop(reason="duplicate_evidence", fact_id=duplicate.fact_id))
    return changeset(changes, errors)

  if only_confidence_changed(comparable.current, candidate):
    close known_to on current at candidate.known_from
    insert candidate with same valid interval and new confidence
    return changeset([known_close, insert], [])

  if no overlapping comparable fact:
    insert candidate
    if prior current fact has open valid interval and candidate.valid_from > prior.valid_from:
      close prior.valid_to at candidate.valid_from and close prior.known_to at candidate.known_from
    return changeset(sorted changes, [])

  for each overlap in comparable_overlaps(candidate):
    if candidate.valid interval ambiguous:
      errors.append(AMBIGUOUS_VALID_INTERVAL)
      continue
    if source authority conflicts and no precedence row:
      emit conflicted fact for candidate
      emit existing conflict marker if policy requires
      continue
    if candidate is unauthorized delete evidence:
      changes.append(noop(reason="unauthorized_delete_evidence", evidence_ref=candidate.evidence_refs))
      continue
    if candidate is authorized retraction:
      close known_to on affected fact
      insert retracted fact row with same valid interval and known_from candidate.known_from
      continue
    if overlap requires interval split:
      split existing into before/overlap/after half-open segments
      close known_to on original
      insert replacement segment(s) and candidate segment
      continue
    if candidate supersedes existing:
      close existing.known_to at candidate.known_from
      insert candidate with assertion_state asserted or derived

  sort all output records by operation_class, fact_type, subject_id, predicate, object_id, valid_from, known_from, fact_id
  return GoldFactChangeSet(changes, errors, checksum=sha256(canonical_json(changes, errors)))
```

Required behavior coverage:

| Condition | Required behavior |
| --- | --- |
| duplicate evidence | Deterministic `noop`. |
| confidence-only change | Close prior known interval and insert new confidence state. |
| non-overlapping later fact | Close prior valid interval if same current fact and insert later fact. |
| overlapping authoritative correction | Split intervals or supersede according to policy. |
| conflicting sources | Emit `conflicted` rows unless precedence row resolves. |
| ambiguous valid interval | Deterministic error or `ambiguous` fact. |
| stale fact | Mark stale only by policy, never retract by age alone. |
| authorized retraction | Close known interval and insert retracted row. |
| unauthorized delete evidence | No-op with evidence link. |
| interval split | Emit before/candidate/after rows with half-open intervals. |
| known-time closure | Every replaced row gets `known_to = candidate.known_from`. |
| deterministic sorting | Sort by declared keys before checksum. |

### 7.4 DecideReplayMode

Inputs:

```text
original VersionManifest
replay request
active ReplayEquivalencePolicy
retained evidence and table refs
```

Output:

```text
production_exact
shadow_recompute
reject_production
VERSION_MANIFEST_MISMATCH
or more specific error
```

Failure precedence:

| Order | Failure class | Error |
| ---: | --- | --- |
| 1 | Missing manifest or output class unsupported | `VERSION_MANIFEST_MISSING` |
| 2 | Missing immutable lakehouse snapshot/commit/dataset refs | `LAKEHOUSE_SNAPSHOT_REF_UNRESOLVED` or `LAKEHOUSE_COMMIT_REF_UNRESOLVED` |
| 3 | Retention-ineligible input | `REPLAY_RETENTION_INELIGIBLE` |
| 4 | Schema/protocol incompatibility | `SCHEMA_COMPATIBILITY_ERROR` |
| 5 | External schema artifact mismatch | `EXTERNAL_SCHEMA_ARTIFACT_MISMATCH` |
| 6 | CDC offset/schema-history insufficiency | `CDC_REPLAY_STATE_INSUFFICIENT` |
| 7 | Source authority/completeness mismatch | `AUTHORITY_PROFILE_MISMATCH` or `COMPLETENESS_PROFILE_MISMATCH` |
| 8 | Output-affecting manifest field mismatch | `VERSION_MANIFEST_MISMATCH` |
| 9 | Changed output-affecting input with shadow allowed | `shadow_recompute` |
| 10 | All included fields equivalent | `production_exact` |

Pseudocode:

```text
DecideReplayMode(original, request, policy, retained):
  require original exists else error VERSION_MANIFEST_MISSING
  require policy covers request.output_class else error REPLAY_POLICY_MISSING

  sufficiency = ReplayInputSufficiencyCheck(original, request, retained)
  if sufficiency.has_specific_error:
    return sufficiency.first_error_by_precedence

  checksum_original = ComputeReplayEquivalenceChecksum(original.output_class, policy.included_fields, policy.excluded_volatile_fields)
  checksum_request = ComputeReplayEquivalenceChecksum(request.output_class, policy.included_fields, policy.excluded_volatile_fields)

  if checksum_original == checksum_request:
    return production_exact

  if request.changed_fields subset policy.shadow_allowed_changed_fields:
    return shadow_recompute

  return error VERSION_MANIFEST_MISMATCH
```

### 7.5 ComputeReplayEquivalenceChecksum

Inputs:

```text
output class
included fields
excluded volatile fields
canonical serialization rules
```

Output:

```text
checksum
excluded field report
deterministic diagnostic rows
```

Default included and excluded fields:

| Output class | Included by default | Excluded volatile fields by default |
| --- | --- | --- |
| raw | payload hash, source config hash, call policy, adapter, source state, table refs | worker PID, runtime duration, retry sleep jitter diagnostics |
| silver | raw refs, parser, mapping, schema artifact, external profile, source extension rules, normalized deterministic fields | `normalized_at` only when policy allows exclusion |
| gold | silver refs, identity refs, authority profile, temporal resolution, correction policy, fact content, intervals | persistence wall-clock if not part of known time |
| graph delta | gold refs, projection profile, edge semantics, operations, properties, idempotency key | backend physical ID |
| graph apply | delta refs, apply profile, schema fingerprint, apply ordering, persisted errors | backend latency, connection ID |
| graph rebuild | lakehouse refs, graph profiles, node/edge/evidence-index records | rebuild duration, backend physical IDs |

Pseudocode:

```text
ComputeReplayEquivalenceChecksum(output_class, included, excluded):
  normalized = {}
  excluded_report = []
  for field in included sorted lexical:
    if field in excluded:
      excluded_report.append(field)
      continue
    value = read(field)
    normalized[field] = canonicalize(value)
  diagnostics = build_sorted_diagnostics(excluded_report, missing_optional_fields)
  checksum = sha256(canonical_json(normalized))
  return { checksum, excluded_report, diagnostics }
```

### 7.6 ResumeGraphApply

Inputs:

```text
GraphDeltaSet
optional prior GraphApplyResult
GraphApplyProfile
ProjectionWatermarkPolicy
```

Output:

```text
GraphApplyResult
```

Pseudocode:

```text
ResumeGraphApply(delta_set, prior_result, apply_profile, watermark_policy):
  require apply_profile.status == active else error GRAPH_APPLY_PROFILE_INACTIVE
  require delta_set.validation_status == valid else error GRAPH_DELTA_INVALID
  require graph schema preflight passes else error GRAPH_SCHEMA_PREFLIGHT_FAILED
  require GraphDeltaIdempotencyKey exists else error GRAPH_DELTA_IDEMPOTENCY_KEY_MISSING

  if prior_result exists:
    if prior_result.idempotency_key != delta_set.idempotency_key:
      return error GRAPH_DELTA_IDEMPOTENCY_CONFLICT
    if prior_result.status == succeeded:
      return deterministic_noop_result(reason="already_applied")
    resume_after = prior_result.last_successful_operation_sort_key
  else:
    resume_after = null

  operations = sort delta_set.operations by:
    1. operation class: retract, expire, upsert_node, upsert_edge, noop
    2. node_id or edge_id lexical
    3. valid_from
    4. known_from
    5. source_fact_id list checksum

  for op in operations after resume_after:
    if op.operation == noop:
      persist no-op result row
      continue
    begin backend transaction according to apply_profile.batch_rule
    apply op using idempotency key and expected schema fingerprint
    if backend error:
      persist error row with stable code
      persist GraphApplyResult(status=partial_failed, last_successful_operation_sort_key)
      return result
    commit transaction
    persist checkpoint after commit only

  result = GraphApplyResult(status=succeeded, idempotency_key, output_checksum)
  watermark_attempt = AdvanceProjectionWatermark(...)
  attach watermark_attempt ref
  return result
```

### 7.7 AdvanceProjectionWatermark

Inputs:

```text
source completeness decision
graph delta validation status
graph apply result
prior watermark
active ProjectionWatermarkPolicy
```

Output:

```text
WatermarkCommitRecord or deterministic no-op/error
```

Pseudocode:

```text
AdvanceProjectionWatermark(completeness, delta_validation, apply_result, prior, policy):
  require policy.status == active else error WATERMARK_POLICY_MISSING

  if policy.requires_completeness and completeness.result not in policy.allowed_completeness_results:
    return error_record(WATERMARK_SOURCE_COMPLETENESS_DENIED)

  if delta_validation != valid:
    return error_record(WATERMARK_ADVANCE_DENIED)

  if policy.watermark_kind in [projection, graph_apply] and apply_result.status != succeeded:
    return error_record(GRAPH_APPLY_NOT_COMPLETE)

  candidate = compute_candidate_watermark(policy, completeness, apply_result)
  if candidate <= prior:
    return no_op_record(prior, candidate)

  if policy.presence_only_allowed == false and completeness.authorizes_absence == false and policy.watermark_kind == source:
    return error_record(WATERMARK_SOURCE_COMPLETENESS_DENIED)

  return commit_record(prior, candidate, committed=candidate, checksum=sha256(canonical_json(inputs)))
```

Watermark advancement rules:

| Watermark kind | Advance condition |
| --- | --- |
| source | Only when completeness profile permits source watermark advancement for dataset/scope. |
| projection | Only when source/completeness gate passes, delta validation passes, and required projection outputs are persisted. |
| graph_apply | Only when graph apply result succeeded and consistency checks required by policy pass. |
| presence_only | May advance only to mark observed presence progress; must not authorize absence, cleanup, or retraction. |

### 7.8 DecideMaintenanceEligibility

Inputs:

```text
maintenance candidate set
ReplayRetentionPolicy
TableMaintenancePolicy
protected VersionManifest refs
table snapshot/commit refs
legal hold refs
```

Output:

```text
ReplayRetentionDecision
```

Pseudocode:

```text
DecideMaintenanceEligibility(candidates, retention_policy, maintenance_policy, protected_manifests, table_refs, legal_holds):
  require retention_policy.status == active else error REPLAY_RETENTION_POLICY_MISSING
  require maintenance_policy.status == active else error TABLE_MAINTENANCE_POLICY_MISSING

  sorted_candidates = sort candidates by table_id, ref_kind, ref_id, object_path
  refusal_reasons = []

  for candidate in sorted_candidates:
    if candidate intersects any legal_hold_refs:
      refusal_reasons.append({candidate, LEGAL_HOLD_PROTECTED})
      continue
    if candidate is referenced by protected production VersionManifest:
      refusal_reasons.append({candidate, PROTECTED_VERSION_MANIFEST_REF})
      continue
    if candidate is required by protected graph rebuild manifest:
      refusal_reasons.append({candidate, PROTECTED_GRAPH_REBUILD_REF})
      continue
    if candidate is within retention_policy.protected_replay_window:
      refusal_reasons.append({candidate, REPLAY_RETENTION_INELIGIBLE})
      continue
    if candidate deletion would break table snapshot, commit, schema, delete-file, log, checkpoint, manifest-list, or table-set checksum:
      refusal_reasons.append({candidate, TABLE_STATE_REF_INVALIDATION})
      continue

  if refusal_reasons not empty:
    return ReplayRetentionDecision(decision=refuse, sorted_refusals, checksum)

  return ReplayRetentionDecision(decision=eligible, candidate_checksum, checksum)
```

Refusal is mandatory when any candidate would invalidate protected replay, graph rebuild, legal hold, or retention policy. The implementation must not delete a candidate subset opportunistically unless the maintenance policy explicitly allows partial eligibility and emits separate decision records for eligible and refused partitions.

## 8. Event-sequence validation corpus

The corpus must be implemented as validation-only `ValidationScenario` artifacts. It must not mutate production state. All timestamps below are UTC RFC3339 with exactly 9 fractional digits in implementation fixtures. Shorthand:

```text
V0 = 2026-01-01T00:00:00.000000000Z
V1 = 2026-01-02T00:00:00.000000000Z
V2 = 2026-01-03T00:00:00.000000000Z
K0 = 2026-02-01T00:00:00.000000000Z
K1 = 2026-02-02T00:00:00.000000000Z
K2 = 2026-02-03T00:00:00.000000000Z
C0 = 2026-02-01T00:05:00.000000000Z
I0 = 2026-02-01T00:06:00.000000000Z
N0 = 2026-02-01T00:07:00.000000000Z
S0 = table snapshot or CDC state reference timestamp only, never fact time
```

All intervals are half-open. Deterministic expected rows must be serialized with sorted keys and sorted arrays by ID.

| Case | Input sequence | Required source-time resolution | Expected gold rows | Expected graph/projection behavior | Expected replay behavior | Expected errors/no-op |
| ---: | --- | --- | --- | --- | --- | --- |
| 1 | Raw `R1` source event time `V0`, collected `C0`, ingested `I0`, normalized `N0`. | `resolved_valid_from = V0`, `known_from = K0`, mode `source_event_time`. | Insert `F1 asserted [V0,∞) [K0,∞)`. | Emit upsert deltas for affected nodes/edges. | `production_exact` with same manifest. | none |
| 2 | `R1` asserts OS=A at `V0`; `R2` asserts OS=B at `V1`, known `K1`. | `R2.valid_from = V1`. | `F1 asserted [V0,V1) [K0,K1)`; `F2 asserted [V1,∞) [K1,∞)`. | Expire/update property edge/node according to projection profile. | Same refs exact; changed mapping shadow. | none |
| 3 | Current fact `F1 [V1,∞)`; late historical `R2` fact for `[V0,V1)` known `K1`. | Backdated valid, current known. | Insert `F2 asserted [V0,V1) [K1,∞)`; no conflict. | Historical-only graph delta if graph serves historical; current graph no-op otherwise. | Exact with retained snapshots. | optional graph `noop` |
| 4 | Existing `F1 value=A [V0,V2)`; late `R2 value=B [V1,V2)` from same authority. | `V1` valid, `K1` known. | Split `F1` to `[V0,V1)`; insert `F2 value=B [V1,V2)`; close old known. | Emit correction deltas for affected valid interval/current projection. | Exact with old/new snapshot refs. | none |
| 5 | Source time absent; policy fallback rejected. | Error. | No gold row. | No graph delta. | Replay returns same error. | `SOURCE_TIME_ABSENT` |
| 6 | Source time malformed string. | Error. | No gold row. | No graph delta. | Replay returns same error. | `SOURCE_TIME_MALFORMED` |
| 7 | Source time lacks timezone; ambiguous behavior is error. | Error. | No gold row. | No graph delta. | Replay returns same error. | `SOURCE_TIME_AMBIGUOUS` |
| 8 | CDC update has LSN/log position and connector timestamp but no source occurrence time. | `source_time_not_authoritative`; no fallback. | No gold row unless fallback policy explicitly permits. | No graph delta. | Rejects missing source-time authority. | `SOURCE_TIME_NOT_AUTHORIZED` |
| 9 | CDC tombstone for existing source row, no completeness/authority for retraction. | Tombstone time ignored as fact time. | Existing fact unchanged; emit no-op delete evidence record. | Graph unchanged. | Exact replay requires CDC offset/schema history refs. | `noop: unauthorized_delete_evidence` |
| 10 | Authorized complete empty scope after prior presence. | Empty receipt does not supply valid time; policy uses completeness decision time only for known-time closure when permitted. | Prior fact retracted or expired according to fact policy and source authority. | Emit expire/retract delta. | Exact with completeness receipt and table refs. | none |
| 11 | Partial known source gap for one page/scope. | Valid source times for present records only. | Present facts allowed; absence/retraction blocked. | Presence deltas only; no cleanup. | Exact with gap evidence. | `noop: absence_denied_partial_known_gap` |
| 12 | Source unavailable. | No candidate time. | No new facts; existing facts may become stale only if staleness policy triggers. | No cleanup/retraction; possible stale marker delta if policy. | Replay exact with source unavailable receipt. | `SOURCE_UNAVAILABLE` or no-op |
| 13 | Two authoritative sources disagree on same interval. | Both resolve same `V0`, known `K1/K2`. | Emit `conflicted` comparable facts or conflict marker per policy. | Graph edge omitted or marked conflicted according to edge semantics. | Exact with both source authority profile refs. | none |
| 14 | Flow observation has endpoints but no initiator evidence. | Time valid; flow role unresolved. | `observed_network_flow_fact` assertion_state `ambiguous`. | Graph directional edge no-op unless profile permits undirected edge. | Exact. | `noop: direction_ambiguous` |
| 15 | Graph apply fails after node upserts and before edge upserts. | Not applicable. | Gold unchanged. | `GraphApplyResult.partial_failed`; checkpoint after last committed node batch; watermark no advance. | Resume applies remaining edge ops only. | `GRAPH_APPLY_NOT_COMPLETE` for watermark |
| 16 | Production replay with identical manifest and retained snapshots. | Same resolutions. | Same gold rows/checksums. | Same graph delta/rebuild checksum. | `production_exact`. | none |
| 17 | Replay changes mapping bundle version. | Recompute differs by manifest. | Production output rejected; shadow allowed if requested. | Shadow graph output only. | `shadow_recompute` or reject production. | `VERSION_MANIFEST_MISMATCH` |
| 18 | Replay changes compiled external schema artifact. | Schema artifact mismatch. | No production output. | No graph output. | Reject before output. | `EXTERNAL_SCHEMA_ARTIFACT_MISMATCH` |
| 19 | Table maintenance would delete protected replay inputs. | Not applicable. | Gold unchanged. | Maintenance not executed. | Replay retention decision refuses. | `REPLAY_RETENTION_INELIGIBLE` |
| 20 | Graph rebuild from same snapshot refs and profiles. | Not applicable. | Gold unchanged. | Rebuild emits same node/edge/evidence-index checksum and passes consistency check. | `production_exact` rebuild equivalence. | none |
| 21 | Historical import with monotonic known-time policy. | Valid `V0`; known set to import time `K1`, not claimed original source knowledge unless allowed. | Insert fact with `known_from = K1`. | Projection updates according to current/historical scope. | Exact. | monotonicity error if `known_from < prior_known` |
| 22 | Same raw evidence reprocessed with different source-time fallback policy. | Resolution differs. | Production replay rejected; shadow only. | Shadow graph output only. | `shadow_recompute` or `VERSION_MANIFEST_MISMATCH`. | `TEMPORAL_POLICY_MISMATCH` |
| 23 | Same correction replayed after graph apply partial failure. | Same correction change set. | Gold rows unchanged; no duplicate facts. | Resume graph apply from checkpoint; same idempotency key. | Exact resume. | `noop: already_applied` for completed ops |
| 24 | Watermark advances after successful graph apply. | Not applicable. | Gold unchanged. | Commit projection/graph watermark to candidate. | Exact. | none |
| 25 | Watermark does not advance after failed or partial graph apply. | Not applicable. | Gold unchanged. | No watermark commit; emit error record. | Exact. | `GRAPH_APPLY_NOT_COMPLETE` |
| 26 | Source watermark candidate exists but completeness profile denies permission. | Not applicable. | Gold unchanged. | No source watermark advance. | Exact. | `WATERMARK_SOURCE_COMPLETENESS_DENIED` |
| 27 | Late record after cutoff from authoritative source. | Valid source time before cutoff. | Accept as correction by production default. | Emit correction delta or no-op as needed. | Exact. | none unless policy rejects |
| 28 | Late record after cutoff from non-authoritative source. | Valid source time before cutoff. | Quarantine or mark ambiguous by default; no authoritative correction. | No production graph mutation. | Exact. | `quarantine` or `mark_ambiguous` |
| 29 | Table snapshot ref resolves through mutable branch only. | Not applicable. | No production output dependent on that ref. | No graph rebuild/apply. | Reject production replay. | `MUTABLE_BRANCH_REF_FOR_REPLAY` |
| 30 | Cross-table table-set checksum mismatch. | Not applicable. | No production output. | No graph rebuild/apply. | Reject before output. | `TABLE_SET_CHECKSUM_MISMATCH` |

The corpus acceptance rule is binary: every expected deterministic field, omitted field, no-op, error code, checksum, and output ordering must match. A case that relies on optional product governance must declare the active policy row in the fixture; otherwise default behavior applies.

## 9. Non-transferable concepts

| Source concept | Why it must not transfer | Cadastre-safe alternative |
| --- | --- | --- |
| watermark as absence proof | Watermarks estimate stream or computation progress, not source-scope completeness. | `SourceCompletenessReceipt` evaluated by `SourceCompletenessProfile`. |
| late-data discard as production default | It destroys evidence and prevents correction/audit reconstruction. | Preserve raw evidence and evaluate through `LateArrivalPolicy`. |
| CDC offset as valid time | Offsets order log processing and do not represent domain validity. | `TemporalObservationTimeResolution`. |
| CDC connector timestamp as known time | Connector time is runtime metadata, not Cadastre knowledge interval. | `KnowledgeTimeImportPolicy` and manifest time rules. |
| CDC heartbeat as completeness | Heartbeat means liveness/progress only. | Completeness receipt with authority-limited evidence rows. |
| schema-history timestamp as fact time | Schema history interprets records but does not describe event occurrence. | CDC schema-history refs in `CDCReplayStateContract`. |
| table snapshot time as valid time | Snapshot time is table state. | `GoldFact.valid_from` from source-time resolution. |
| table commit time as known time | Commit time is storage operation time. | `GoldFact.known_from` by Cadastre policy. |
| catalog branch name as production approval | Branch name is mutable catalog visibility, not governance. | `CatalogBranchPromotionPolicy`. |
| object-store commit as table-format snapshot proof | Object versioning does not prove table schema, partition, metadata/log, or delete-file identity. | `LakehouseSnapshotRef` wrapping native table refs. |
| Temporal workflow event history as source evidence | Workflow history proves execution decisions, not source assertions. | `RawRecord`, `EvidenceRef`, `VersionManifest`. |
| EventStore projection stream as graph truth | Projection stream is derived and may lag or fail. | Lakehouse-derived `GraphDeltaSet` and `GraphRebuildManifest`. |
| graph backend mutation as authoritative mutation | Backend state is replaceable projection state. | Graph apply only from validated deltas. |
| Differential Dataflow diff as public graph API | Implementation diffs are not stable Cadastre fact or graph contracts. | Graph deltas with stable node/edge semantics. |
| OpenLineage run complete as source completeness | Pipeline completion does not prove source coverage or absence. | `EvaluateSourceCompleteness`. |
| dbt freshness artifact as source completeness | Freshness tests are workflow artifacts, not source absence authority. | `SourceCompletenessProfile` and coverage assertions. |
| DataHub/OpenMetadata metadata graph as Cadastre gold facts | Metadata graph is governance/metadata state. | `GoldFact` bitemporal facts. |
| live source probe zero-row result as absence proof | Live probe may be filtered, permission-limited, partial, or transient. | Production source collection plus completeness profile. |
| acknowledgment success as source completeness | Ack proves delivery/processing boundary, not source-state completeness. | Ack as `SourceCompletenessEvidenceRow` with authority limit. |
| provenance closure as source evidence | Provenance proves ingestion/runtime lineage, not source assertion. | `IngestionProvenanceEvent` plus raw source records. |

## 10. PRD improvement map

| Rank | PRD change | Current gap closed | Source basis | Required contract target | Acceptance criterion |
| ---: | --- | --- | --- | --- | --- |
| 1 | Add `TemporalSemanticsPolicy` and `TemporalObservationTimeResolution`. | Source/connector/table/watermark time confusion. | Bitemporal, stream, CDC, PRD source-time boundaries. | New temporal semantics section near raw/silver/gold time rules. | `ResolveFactTime` corpus cases 1, 5, 6, 7, 8, 21, 22 pass. |
| 2 | Add `LateArrivalPolicy`. | Late authoritative evidence may be discarded or misrouted. | Flink, Beam, Kafka Streams. | Source completeness and temporal correction contracts. | Cases 27 and 28 pass. |
| 3 | Add `GoldFactCorrectionPolicy` and `GoldFactChangeSet`. | Comparable correction behavior, interval split, conflict, duplicate no-op not total. | XTDB, Datomic, EventStoreDB, Differential Dataflow, PRD correction rule. | Gold fact correction section. | Cases 2, 3, 4, 9, 10, 13, 23 pass. |
| 4 | Add `KnowledgeTimeImportPolicy`. | Historical imports can claim impossible known times. | Bitemporal systems and PRD bitemporal facts. | Gold temporal rules. | Case 21 passes; monotonic violation fails. |
| 5 | Add `BitemporalQueryMode`. | Query-time defaults and assertion-state inclusion can diverge. | XTDB/Datomic and PRD user-facing time filters. | Query API and gold fact sections. | Current, valid-at, known-at, audit fixture ordering is exact. |
| 6 | Add `ReplayEquivalencePolicy` and `ReplayInputSufficiencyCheck`. | Replay mismatch and field inclusion/exclusion not mechanically complete. | Temporal.io/Cadence and lakehouse refs. | Version manifest/replay section. | Cases 16, 17, 18, 29, 30 pass. |
| 7 | Add `DeterministicSideEffectRecord`. | Runtime nondeterminism can affect output without replay record. | Temporal.io/Cadence side-effect patterns. | Version manifest/stage state section. | Missing side-effect fixture rejects before output. |
| 8 | Add `ProjectionWatermarkPolicy` and `WatermarkCommitRecord`. | Projection/source/graph watermarks can advance inconsistently. | Stream systems, Differential frontiers, graph apply. | Completeness and graph apply sections. | Cases 24, 25, 26 pass. |
| 9 | Add `GraphDeltaIdempotencyKey`. | Reapply/resume behavior can duplicate graph mutation. | EventStoreDB expected revision/idempotency, PRD graph apply. | Graph delta/apply section. | Case 23 and repeated apply fixture are idempotent. |
| 10 | Add `CDCReplayStateContract`. | CDC offsets/schema history/tombstones not fully replay-governed. | Debezium/Kafka Connect, PRD raw subtype. | Source adapter and replay sections. | Cases 8 and 9 pass; missing schema history rejects. |
| 11 | Add `CorrectionSnapshotRefPolicy`. | Corrections do not yet require old/new table-state refs. | Iceberg, Delta, Hudi, Nessie, prior lakehouse report. | Gold correction and lakehouse ref sections. | Correction without old/new refs rejects. |
| 12 | Add `GraphRebuildEquivalencePolicy`. | Rebuild equivalence and checksum exclusions not fully defined. | DataHub restore-index, EventStore projection reset, PRD graph rebuild. | Graph rebuild section. | Case 20 passes. |
| 13 | Add normative watermark/completeness authority table. | Progress signals may leak into absence/retraction/cleanup. | Stream, CDC, lineage, dbt, metadata systems. | Source completeness appendix. | Every signal has explicit allowed/forbidden defaults. |
| 14 | Add replay and rebuild sufficiency matrix as PRD appendix. | Output-class required inputs can diverge. | Temporal, lakehouse, CDC, graph projection sources. | Version manifest/replay section. | Every output class has required refs and failure code. |
| 15 | Add event-sequence corpus requirement. | Temporal/correction/replay behavior lacks binary fixture coverage. | NLSpec acceptance criteria standard. | Validation matrix section. | All 30 cases implemented and pass. |

Source basis: The PRD already defines broad contracts, but NLSpec-quality behavior requires deterministic algorithms, defaults, bounds, exhaustive mappings, and binary acceptance criteria.[^1][^3][^4][^5][^6][^7][^8][^9][^10]

## 11. Acceptance criteria for the report

| ID | Criterion | Result |
| --- | --- | --- |
| AC-REPORT-001 | Every source family is analyzed independently before cross-source comparison. | Pass. Section 4 analyzes each family before Section 5. |
| AC-REPORT-002 | Every factual claim about an external system is cited to a primary source or labeled as an inference. | Pass with stated limits. Claims are source-based or labeled as inferences; no runtime verification is implied. |
| AC-REPORT-003 | Every Cadastre recommendation maps to at least one Cadastre contract or proposed contract. | Pass. Section 6 and Section 10 map recommendations to contracts. |
| AC-REPORT-004 | Every proposed contract includes purpose, observable behavior, interfaces, defaults, errors, non-authority boundaries, and binary acceptance criteria. | Pass. Section 6 provides the fields. |
| AC-REPORT-005 | The report includes the required temporal model mapping table, correction model table, replay sufficiency matrix, watermark/completeness authority table, and graph idempotency comparison table. | Pass. Section 5 includes all required tables. |
| AC-REPORT-006 | The report includes deterministic pseudocode for all required algorithms. | Pass. Section 7 includes all required algorithms. |
| AC-REPORT-007 | The event-sequence validation corpus includes all required cases and exact expected outcomes. | Pass. Section 8 includes all 30 cases. |
| AC-REPORT-008 | The report explicitly separates confirmed source facts, source-grounded inferences, Cadastre applicability inferences, speculative transfers, and open questions. | Pass. Section 2 defines evidence classes; Section 4 uses the required comparison tables. |
| AC-REPORT-009 | The report identifies concepts that must not transfer and gives a Cadastre-safe alternative for each. | Pass. Sections 4 and 9. |
| AC-REPORT-010 | The report does not treat external product defaults as Cadastre policy. | Pass. External defaults are treated as references only. |
| AC-REPORT-011 | The report does not treat graph, lineage, metadata, workflow, CDC, stream, or table-format state as Cadastre authority unless a Cadastre contract explicitly permits it. | Pass. Non-authority boundaries are explicit. |
| AC-REPORT-012 | The report includes source limits for every inspected source, including uninspected files, unrun builds, unrun tests, inaccessible docs, or unverified runtime behavior. | Pass. Section 2 source inventory and source limits summary. |
| AC-REPORT-013 | The report is self-contained and does not require the reader to return to source repositories for a first-pass understanding. | Pass. Section 4 reconstructs transferable source models. |
| AC-REPORT-014 | The report uses precise normative language for recommendations and avoids vague instructions such as “handle appropriately.” | Pass. Contract recommendations use `must`, `must not`, `may`, and default behavior. |
| AC-REPORT-015 | The report contains no unsupported claims of compatibility, implementation behavior, release state, or production safety. | Pass with explicit source and execution limits. |

### Sources

[^1]: `PRD-Cadastre.md`, Section 1 Product Summary, lines 17-45. Governing Cadastre baseline: lakehouse system of record, graph/CIM as replaceable projections, and table-format-native state references.
[^2]: `PRD-Cadastre.md`, Section 2 Document Status, lines 53-65. Governing non-authority boundaries for table, CDC, connector, heartbeat, watermark, queue, provenance, live query, and graph-backend state.
[^3]: `nlspec-spec.md`, version 0.2.2, lines 13-72 and 224-270. NLSpec quality standard: prescriptive/generative specifications, behavioral completeness, unambiguous interfaces, explicit defaults/boundaries, mapping tables, and acceptance criteria.
[^4]: `PRD-Cadastre.md`, Section 10.3 `RawRecord`, lines 1394-1457. Raw event subtypes, CDC metadata, source event time quality, and CDC non-authority rules.
[^5]: `PRD-Cadastre.md`, Section 10.4 `CadastreSilverObservation`, lines 1458-1538. Silver observation fields, source/normalized time, external schema fields, field quality, and flow-role evidence.
[^6]: `PRD-Cadastre.md`, Section 10.9 `GoldFact`, lines 1721-1815. Bitemporal fact fields, assertion states, half-open intervals, network-flow rule, and correction rule.
[^7]: `PRD-Cadastre.md`, Section 10.13 `VersionManifest`, lines 1868-2018. Manifest content and replay mismatch requirements.
[^8]: `PRD-Cadastre.md`, `SourceCompletenessReceipt` section, lines 2854-2915. Completeness receipt contract and source completeness states.
[^9]: `PRD-Cadastre.md`, `SourceCompletenessProfile` and `EvaluateSourceCompleteness` section, lines 4073-4216. Completeness profile and deterministic evaluation requirements.
[^10]: `PRD-Cadastre.md`, lakehouse and graph rebuild/apply sections, including `GraphRebuildManifest`, `GraphIndexConsistencyCheck`, and `DerivedViewLagPolicy`, lines 2393-2488; graph apply/profile material around lines 5475-5524. Line ranges are approximate because the uploaded PRD is long and may shift if edited.
[^11]: `RES-005-Lakehouse-Derived-Graph-Architecture.md`, Executive summary and top findings, lines 8-49. Used as project research context only, not governing authority.
[^12]: `RES-006-source-adapter-ingestion-patterns.md`, Executive summary and CDC/state comparison material, lines 17-32 and approximately 859-1006. Used as project research context only, not governing authority.
[^13]: `RES-008-identity-graph-attack-paths.md`, Executive verdict and source inventory, lines 1-77. Used as project graph-adjacent context only.
[^14]: `RES-001-cartography.md`, Metadata and executive architectural map, lines 1-55. Used as project graph-adjacent context only.
[^15]: `RES-002-jupiterone.md`, Executive summary and SDK/Starbase analysis context, lines 1-35 and approximately 270-305. Used as project graph-adjacent context only.
[^16]: XTDB official documentation, “Time in XTDB” and bitemporality documentation, inspected 2026-05-16. URL inspected through web search: `https://docs.xtdb.com/` and XTDB bitemporality docs. No XTDB runtime was run.
[^17]: Datomic official documentation, history/tutorial/retraction docs and Datomic Pro release/change-log pages, inspected 2026-05-16. URLs inspected through web search: `https://docs.datomic.com/` and Datomic release pages. No Datomic runtime was run.
[^18]: Apache Flink official documentation for event time, watermarks, windows, and allowed lateness, plus Flink release/download pages, inspected 2026-05-16. URL family: `https://nightlies.apache.org/flink/` and `https://flink.apache.org/`. No Flink job was run.
[^19]: Apache Beam official programming guide and Javadocs for event time, watermarks, triggers, and allowed lateness, plus Beam releases/downloads, inspected 2026-05-16. URL family: `https://beam.apache.org/`. No Beam pipeline was run.
[^20]: Apache Kafka official documentation and release pages for Kafka Streams concepts, timestamps, state stores, and release status; Confluent explanatory Kafka Streams grace-period docs used only as supplementary context, inspected 2026-05-16. No Kafka Streams application was run.
[^21]: Debezium official MySQL connector and state-storage documentation, including snapshots, binlog offsets, schema history, delete/tombstone, and heartbeat behavior, inspected 2026-05-16. URL family: `https://debezium.io/documentation/`. No connector was run.
[^22]: Debezium official PostgreSQL connector documentation, including snapshots, WAL/LSN, deletes, tombstones, schema handling, and replica identity caveats, inspected 2026-05-16. URL family: `https://debezium.io/documentation/`. No connector was run.
[^23]: Temporal.io official documentation for workflow event history, deterministic replay, activities, side effects, patching/versioning, and nondeterminism, inspected 2026-05-16. URL family: `https://docs.temporal.io/`. No Temporal worker/server was run.
[^24]: Cadence official documentation for workflow replay and side-effect recording, inspected 2026-05-16. URL family: `https://cadenceworkflow.io/` and associated docs. No Cadence runtime was run.
[^25]: EventStoreDB/KurrentDB official documentation for appending events, expected revision, idempotent event IDs, persistent subscriptions, checkpoints, parked messages, and projection reset, inspected 2026-05-16. URL family: `https://docs.kurrent.io/` and `https://developers.eventstore.com/`. No server was run.
[^26]: Differential Dataflow official documentation and repository documentation describing collections as `(data, time, diff)` updates, frontiers, arrangements, and incremental computation, inspected 2026-05-16. URL family: `https://timelydataflow.github.io/differential-dataflow/` and GitHub repository docs. No program was run.
[^27]: Apache Iceberg table specification, docs, and release pages for table metadata, snapshots, manifests, delete files, schema IDs, snapshot refs, commits, and snapshot expiration, inspected 2026-05-16. URL family: `https://iceberg.apache.org/` and GitHub releases. No table was run.
[^28]: Delta Lake protocol documentation, docs, and release pages for transaction log versions, actions, checkpoints, protocol features, deletion vectors, and vacuum, inspected 2026-05-16. URL family: `https://docs.delta.io/` and GitHub releases. No table was run.
[^29]: Apache Hudi official docs and release pages for timeline instants, commit states, rollback, restore, savepoints, cleaning, and incremental queries, inspected 2026-05-16. URL family: `https://hudi.apache.org/`. No Hudi table was run.
[^30]: Project Nessie official docs and release pages for branches, tags, commits, content keys, content IDs, cross-table semantics, and Iceberg integration, inspected 2026-05-16. URL family: `https://projectnessie.org/`. No Nessie server was run.
[^31]: lakeFS official docs and repository pages for object-store commits, branches, rollback, and garbage collection, inspected 2026-05-16. URL family: `https://docs.lakefs.io/`. No lakeFS repository was run.
[^32]: DataHub official docs and repository/release pages for metadata aspects, MCP/MCL, and graph/search index restore, inspected 2026-05-16. URL family: `https://docs.datahub.com/`. No DataHub instance was run.
[^33]: OpenLineage official specification and docs for run/job/dataset events, event lifecycle, facets, and custom facets, inspected 2026-05-16. URL family: `https://openlineage.io/`. No OpenLineage client/backend was run.
[^34]: OpenMetadata official docs and repository pages for entities, versions, lineage APIs, search indexing, ingestion, classifications, and governance, inspected 2026-05-16. URL family: `https://docs.open-metadata.org/`. No server was run.
[^35]: Marquez official docs and repository pages for OpenLineage-compatible backend and lineage graph/API, inspected 2026-05-16. URL family: `https://marquezproject.ai/`. No Marquez server was run; exact current release was not established.
[^36]: dbt official artifact documentation for `manifest.json`, `run_results.json`, `sources.json`, and semantic artifacts, inspected 2026-05-16. URL family: `https://docs.getdbt.com/reference/artifacts/`. No dbt project was run.
