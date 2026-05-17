---
doc_id: RES-006
project: Cadastre
title: Source Adapter and Ingestion Patterns Research Report for Cadastre
status: research-report
inspection_date: 2026-05-16
primary_sources:
  - cloudquery/cloudquery
  - turbot/steampipe
  - apache/nifi
  - opensearch-project/data-prepper
  - debezium/debezium
---

## Executive summary

Cadastre should adopt a stricter source-adapter contract than any inspected external system. The strongest transferable patterns are CloudQuery's plugin/table/resolver decomposition and incremental-state discipline, Steampipe's live API table ergonomics and predicate/projection pushdown, NiFi's FlowFile provenance and queue/backpressure taxonomy, Data Prepper's explicit pipeline source-buffer-processor-sink model and end-to-end acknowledgment boundary, and Debezium's durable-change-log snapshot/stream/offset/schema-history model. None of the inspected systems should become governing architecture for Cadastre. Cadastre's PRD already establishes the controlling boundary: the lakehouse is the system of record, adapters may emit raw records and source completeness receipts, and graph/CIM outputs are replaceable deterministic projections.[^1]

The highest-value PRD changes are:

| Rank | Recommendation | Source basis | Cadastre contract area | Transfer stance |
| ---: | --- | --- | --- | --- |
| 1 | Add collection patterns for `poll_with_cursor`, `incremental_table_sync`, `snapshot_then_incremental`, `cdc_initial_snapshot`, `cdc_stream`, `snapshot_then_stream`, `pipeline_acknowledged_batch`, `flowfile_batch`, and `live_source_probe`. | CloudQuery incremental tables; Debezium snapshots/streams; Data Prepper acks; NiFi FlowFiles; Steampipe live tables. | `CollectionStepPattern`, `SourceStageStep` | `adopt`/`adapt` by pattern |
| 2 | Extend `SourceCallPolicy` with cursor commit rules, retry-after handling, duplicate cursor behavior, stream resume behavior, acknowledgment timeout, and schema-history dependency fields. | CloudQuery state backend and at-least-once duplicates; Data Prepper acks; Debezium offsets/schema history; Steampipe rate limiters. | `SourceCallPolicy`, `StageStateRecord` | `adopt` |
| 3 | Extend `StageStateRecord` with state classes for page tokens, API cursors, sync-group cursors, CDC offsets, schema history refs, source-coordination leases, and ack batch IDs. | CloudQuery cursors; Debezium offsets; Data Prepper source coordination; NiFi state. | `StageStateRecord`, `VersionManifest` | `adopt` |
| 4 | Extend `SourceCompletenessReceipt` with evidence classes for cursor exhaustion, expected count match, stream heartbeat, schema history availability, end-to-end ack, provenance closure, table commit success, permission-scope proof, and source-declared partiality. | All five systems expose partial-completion, state, or replay signals, but none is sufficient alone. | `SourceCompletenessReceipt`, `SourceCompletenessProfile`, `CoverageAssertion` | `adapt` |
| 5 | Add `source_exploration` or `live_source_probe` mode as non-authoritative tooling. | Steampipe's live SQL query model. | `SourceStageDAG`, `PackageStageBinding`, `RecordedSourceFixture` | `adapt` |
| 6 | Add CDC-specific fixture and validation classes. | Debezium CDC event envelope, snapshot modes, incremental snapshots, schema history, offsets, tombstones. | `RecordedSourceFixture`, `ValidationMatrix`, `RawRecord` | `adopt` |
| 7 | Add ingestion-provenance event taxonomy distinct from source evidence. | NiFi provenance events and replay behavior. | `DiagnosticRecord`, `RunDatasetIOContract`, `RawRecord`, `EvidenceRef` | `adapt` |
| 8 | Reject destination-authoritative cleanup, live-query evidence, provenance-as-truth, ack-as-completeness, and CDC-time-as-valid-time. | CloudQuery destination write modes; Steampipe live queries; NiFi replay; Data Prepper acknowledgments; Debezium offsets and timestamps. | Entire raw/silver/gold/projection boundary | `reject` or `cautionary_reference` |

Bottom line: Cadastre should define adapters as deterministic source-evidence collectors with explicit state, call policy, completeness evidence, validation fixtures, and version-manifest references. It must not let adapters become graph writers, destination cleanup engines, opaque sync services, or live query backends.

## Evidence classes and inspection limits

| Evidence class | Meaning in this report |
| --- | --- |
| `confirmed source fact` | Directly supported by source code, official documentation, release metadata, examples, or inspected repository structure. |
| `documentation fact` | Directly supported by official documentation or README material but not verified in a local build. |
| `source-grounded inference` | Inferred from multiple confirmed facts in one source. |
| `Cadastre applicability inference` | Design implication derived by comparing one source to the Cadastre PRD. |
| `speculative transfer` | Promising concept requiring deeper implementation inspection, benchmark, or domain authority before PRD change. |

No repository was cloned or built locally for this report. No tests, examples, Docker services, connectors, or local databases were run. Public repository pages, official documentation, release pages, and source snippets were inspected on 2026-05-16. Release, branch, and commit facts are freshness-sensitive and must be rechecked before implementation or PRD amendment.

## Cadastre baseline used for comparison

Cadastre's PRD is authoritative for this report. It defines Cadastre as a temporal-lakehouse-backed graph asset intelligence platform. The raw/bronze, silver, and gold lakehouse layers are authoritative; graph serving and Splunk CIM are replaceable projections; and projection is the deterministic contract from authoritative records to serving outputs.[^1] The PRD explicitly rejects direct-to-graph or graph-authoritative synchronization and treats external tools as evidence sources, not governing systems.[^2]

The PRD already names the key adapter-adjacent contracts: `SourceInstance`, `SourceInstanceConfigSchema`, `ConnectivityValidationArtifact`, `SourceStageDAG`, `SourceStageStep`, `CollectionStepPattern`, `SourcePackageDeveloperContract`, `PackageStageBinding`, `SourceCallPolicy`, `StageStateRecord`, `SourceCompletenessProfile`, `SourceCompletenessReceipt`, `CoverageAssertion`, `VersionManifest`, `RecordedSourceFixture`, and `DiagnosticRecord`.[^3]

The PRD's `RawRecord` contract preserves source-native payloads, source identity, source dataset/type, native IDs, schema version, source event time quality, payload format, exact payload hash, adapter/collector versions, run lineage, duplicate/quarantine status, and partition key.[^4] Adapter collection stages are currently permitted to emit only `RawRecord[]`, `SourceCompletenessReceipt[]`, and adapter error records. They must not emit silver observations, coverage assertions, identity decisions, unresolved target references, gold facts, graph deltas, CIM projection results, or graph apply results.[^5]

The PRD already defines `SourceCompletenessReceipt` as observed completeness state, not self-authorizing truth. Absence, retraction, cleanup, and watermark advancement require evaluation against an active `SourceCompletenessProfile`; unsafe states such as partial gaps, unavailable sources, unavailable scopes, and not-attempted scopes default to no absence, no retraction, no cleanup, and no source watermark advancement.[^6]

`StageStateRecord` already requires output-affecting execution state to be explicit, canonicalized, hashable, replayable, and included in `VersionManifest`.[^7] `SourceStageStep` already requires every adapter collection step to declare exactly one `CollectionStepPattern`, and every source-call step must reference one active `SourceCallPolicy`.[^8] `SourceCallPolicy` already defines pagination, retry, rate limit, timeout, duplicate-page, partial-page, credential-error, and visibility-loss behavior with concrete defaults and bounds.[^9]

## CloudQuery independent analysis

### Source inventory and inspection limits

| Field | Required content |
| --- | --- |
| Source name | `cloudquery/cloudquery`; CloudQuery official docs; CloudQuery Go source plugin SDK docs; CloudQuery destination and incremental-sync docs. |
| URL or path | Public repository and docs pages inspected through web. |
| Evidence class | `source_code`, `official_docs`, `release_metadata`, `example`, `inference`. |
| Commit, tag, branch, or version | Repository branch `main` was inspected from GitHub page. Exact commit was not pinned. Docs represent current public docs as of inspection. |
| Inspection date | 2026-05-16. |
| Scope inspected | Repository root/module layout; README; Architecture docs; source plugin Go SDK docs; source plugin layout docs; incremental-table docs; destination write mode docs; transformer behavior docs; configuration examples. |
| Scope not inspected | Full repository source tree, generated plugin protocol definitions, every plugin, destination implementation source, tests, CI, package signing/verification code, local CLI behavior, builds, sync runs, and destination write behavior under load. |
| Reliability | High for documented architecture and configuration surfaces; medium for operational semantics not validated by local execution. |

CloudQuery receives full-depth treatment because it is the most directly relevant source-adapter architecture. The repository page identifies a monorepo with CLI, plugins, scaffolding, scripts, and versioned sites. The README describes CloudQuery as syncing cloud and SaaS asset data from AWS, Azure, GCP, and many sources into queryable destinations.[^10]

### Project purpose and non-purpose

CloudQuery is an ELT-style asset data collection and synchronization system. It is designed to connect source plugins to destination plugins, optionally run transformer plugins, and write tabular records into destinations. Its purpose is not bitemporal fact derivation, Cadastre identity resolution, gold fact arbitration, source completeness authorization, or graph projection.

For Cadastre, CloudQuery is valuable as a reference for source plugin boundaries, table/resolver organization, sync state, schema declaration, incremental collection, destination fan-out, and operational diagnostics. Its destination modes and stale deletion are not safe as Cadastre truth mechanisms.

### Repository layout and major modules

Confirmed repository-level areas include `cli`, `plugins`, `scaffold`, `scripts`, and `sites/versions` in the public repository. The source plugin SDK documentation describes a Go source plugin project layout with `main.go`, `go.mod`, `plugin/`, `client/`, `spec/`, `resources/`, `services/`, and tests. It shows a separation between CLI entrypoint, plugin configuration, source client construction, table definitions, and resolvers.[^11]

| Area | CloudQuery role | Cadastre relevance |
| --- | --- | --- |
| CLI | Orchestrates sync execution and gRPC communication with plugins. | Reference for `SourceStageDAG` runtime boundary and package execution. |
| Source plugins | Define tables, clients, resolvers, incremental tables, and source config. | Strong reference for `SourcePackageDeveloperContract` and `SourceStageStep`. |
| Transformer plugins | Mutate/filter records between source and destination. | Caution: Cadastre transformer-like behavior must be parser/normalizer stages, not adapter output mutation. |
| Destination plugins | Receive transformed source records and apply write modes. | Caution: destination behavior must not determine Cadastre truth or cleanup. |
| Plugin SDK | Provides table/resolver/state abstractions and code generation or scaffolding. | Reference for developer workflow and validation patterns. |
| Plugin tests/examples | Validate plugin tables, resolvers, and sync behavior. | Reference for `RecordedSourceFixture` and `ValidationMatrix`. |

### Architecture and component boundaries

CloudQuery's official architecture describes the CLI as the orchestrator. Source, destination, and transformer integrations run as plugins and communicate with the CLI over gRPC. Data flow is source plugin to CLI, optional transformer chain, then one or more destination plugins. Every record flows through the CLI, which augments records with metadata such as sync time, source name, and sync group ID.[^12]

Cadastre transfer: the plugin runtime boundary is useful, but Cadastre must split CloudQuery's broad sync pipeline into bounded stages. In Cadastre, the source plugin runtime may perform source collection only; parser, mapper, resolver, gold derivation, projection, and graph apply remain separate stages with separate output permissions.

### Data model and record/event/schema model

CloudQuery models source data as tables. Tables have columns, primary keys, resolvers, parent/child relationships, and optional incremental behavior. Plugin docs show table resolver functions receiving context, client metadata, optional parent resource, and a result stream object. Resolver functions send rows through the result stream, which allows streaming and partial emission while a resolver is still executing.[^13]

CloudQuery destinations add system metadata columns, and destination configuration supports primary-key modes. The destination documentation states that default `write_mode` is `overwrite-delete-stale`, `migrate_mode` is `safe`, and primary-key mode defaults to a CloudQuery ID mode in some destination docs. Destination modes include `overwrite-delete-stale`, `overwrite`, and `append`; overwrite-delete-stale deletes rows no longer observed in the current sync for a table.[^14]

Cadastre transfer: table declarations are useful as source-dataset declarations. Primary keys and CloudQuery system IDs are not Cadastre canonical identity. Destination metadata columns are useful as raw collection metadata analogues, but Cadastre raw IDs and payload hashes must be product-owned.

### Adapter, plugin, connector, or processor interface

The Go source plugin interface includes `Configure`, source client creation, table definitions, resolvers, table testing, and state backend access for incremental tables. The documentation shows `Configure` parsing and validating the plugin spec and returning a client. It also shows incremental table support through table options, an incremental key, and a state backend with `GetKey`, `SetKey`, and `Flush`; failure to flush state can cause refetching.[^11]

Cadastre transfer:

| CloudQuery interface concept | Cadastre-safe analogue |
| --- | --- |
| Source client | Package-internal source client component. Must not emit production records directly. |
| Table | `SourceStageStep.source_dataset` plus source dataset schema. |
| Resolver | Collection-step handler. Must emit only raw records/completeness/errors in adapter stage. |
| Parent/child table | `fetch_child_dataset` or future `child_table_sync` pattern. |
| Incremental table | New `incremental_table_sync` or `poll_with_cursor` pattern. |
| State backend | `StageStateRecord` with manifest inclusion. |
| Transform plugin | Parser/normalizer stage only, not adapter output mutation. |
| Destination plugin | Not part of adapter authority. |

### Configuration surfaces, defaults, and bounds

CloudQuery configuration includes source plugin specs, destination specs, transformer lists, write modes, migration modes, primary-key modes, and state backend settings. The source plugin guide examples show config structs with defaults, validation constraints, and secret-like fields. The destination docs identify `write_mode`, `migrate_mode`, `pk_mode`, `send_sync_summary`, and transformer behavior.[^11][^14]

Cadastre needs a more restrictive surface. Production `SourceInstanceConfigSchema` must declare required fields, optional fields, defaults, secret fields, credential references, redacted hashes, scope keys, and validation status. Live user config, environment variables, and destination settings must not affect production output unless materialized into `SourceInstance`, `StageStateRecord`, and `VersionManifest`.

### Control flow and data flow

CloudQuery sync flow can be summarized as:

```text
source plugin Configure
  -> source client construction
  -> table graph selection
  -> resolver execution
  -> source rows streamed to CLI
  -> CloudQuery metadata columns added
  -> optional transformer chain
  -> destination writes using configured write mode
  -> state backend updates for incremental tables
```

Cadastre-safe flow must be:

```text
SourceInstance + SourceCallPolicy + SourceStageStep
  -> source client call
  -> RawRecord persistence
  -> SourceCompletenessReceipt persistence
  -> StageStateRecord persistence when state affects output
  -> parser stage
  -> normalizer stage
  -> identity/gold/projection/apply governed stages
```

### State, cursor, offset, snapshot, watermark, or replay model

CloudQuery incremental-table docs state that incremental tables fetch only records that changed since a prior sync, read cursor state from a state backend, and provide at-least-once delivery with no gaps, which can create duplicate records. Append mode requires downstream deduplication because duplicates may occur.[^15]

Cadastre transfer: CloudQuery's at-least-once/no-gap language is a useful state contract template. Cadastre must adapt it into explicit state and duplicate behavior:

| Required Cadastre field or rule | Definition |
| --- | --- |
| `StageStateRecord.state_kind = cursor` | Holds source cursor, table name, scope key, start cursor, end cursor candidate, and commit status. |
| `cursor_commit_rule` | Cursor may be committed only after raw records and completeness receipt persist successfully. |
| `duplicate_cursor_behavior` | Closed enum: `reject`, `record_duplicate_raw_status`, `dedupe_by_source_native_id`, `dedupe_by_payload_hash`. Default `record_duplicate_raw_status` for at-least-once sources. |
| `gap_prevention_rule` | A cursor may not advance past an unpersisted page, failed child scope, failed schema-history dependency, or source-declared partial result. |
| `VersionManifest` requirement | Every output-affecting cursor state hash must be present before production replay. |

### Error handling, retry, rate limit, partial collection, and backpressure

CloudQuery docs and plugin guides include retry and resolver behavior patterns, but this report did not inspect all retry implementation code. The key relevant behavior is incremental duplicate handling and destination write behavior, not a complete source-call policy implementation.

Cadastre must not inherit implicit CloudQuery behavior. It must define source-call outcomes through `SourceCallPolicy` and `SourceCompletenessReceipt`:

| Condition | Cadastre behavior |
| --- | --- |
| Retryable source error before any page persists | Retry according to `SourceCallPolicy`; on exhaustion emit adapter error and `source_unavailable` or `partial_unknown_gap` receipt. |
| Retryable error after some pages persist | Persist collected raw records, emit receipt state `partial_known_gap` when missing page or child scope is known, else `partial_unknown_gap`; block absence, cleanup, and watermark. |
| Duplicate page/cursor | Use declared `duplicate_cursor_behavior`; default records duplicate raw status rather than silently deduplicating. |
| Incremental cursor lost | Emit `STATE_RECORD_MISSING`; execute full resync only if source profile declares safe full fallback. |

### Source completeness and absence semantics

CloudQuery's `overwrite-delete-stale` destination behavior can delete rows no longer observed in the current sync. That is not source completeness. It is destination-state cleanup based on the observed result set. Cadastre must not convert this into absence, retraction, cleanup, or watermark advancement unless an active `SourceCompletenessProfile` and `SourceCompletenessReceipt` prove the source dataset and scope are complete under Cadastre rules.

Cadastre should copy only the idea that stale cleanup must be a declared mode. It should not copy CloudQuery destination cleanup authority.

### Validation, tests, fixtures, and golden-output patterns

CloudQuery plugin docs include source plugin testing guidance and table test resources. The most transferable pattern is validating source tables/resolvers in isolation before integration runtime. Cadastre should require fixture classes for full sync, incremental table sync, cursor loss, duplicate cursor, duplicate page, partial page, rate limit, timeout, credential error, permission loss, and state flush failure.

### Operational model, deployment, observability, and security

CloudQuery's operational model is CLI-driven and plugin-driven. The architecture docs place the CLI in the control path and the plugins behind gRPC. The system supports multiple destinations from one source collection and optional transformer chains.[^12]

Cadastre adaptation: one source collection may feed multiple downstream stages, but the fan-out point must be persisted raw records and explicit dataset refs, not an in-memory streaming path. Metrics, logs, traces, exit codes, partial run results, state writes, and destination summaries must become `DiagnosticRecord`, `SourceStageLifecycleResult`, `SourceCompletenessReceipt`, and `VersionManifest` references when they affect output or replay.

### Extension points and developer workflow

CloudQuery offers plugin SDKs, source plugin project layout, table definitions, configuration validation, resolvers, state access, transformer plugins, destination plugins, and scaffolding. Cadastre should adapt the developer ergonomics while constraining production outputs.

Cadastre `SourcePackageDeveloperContract` should explicitly separate:

| Package component | Permitted behavior |
| --- | --- |
| source client | May call the source and return source-native responses to collection steps. Must not emit production records. |
| collection step | May emit `RawRecord`, `SourceCompletenessReceipt`, and adapter errors only. |
| parser | May parse raw records and emit parse results/errors only. |
| mapper/normalizer | May emit `CadastreSilverObservation`, `CoverageAssertion`, and `UnresolvedTargetReference` only when stage permits. |
| state provider | May read/write declared `StageStateRecord` only. |
| validation component | May emit validation artifacts and diagnostics only. |

### Confirmed findings

| Finding | Evidence class | Contract implication |
| --- | --- | --- |
| CloudQuery runs source, transformer, and destination integrations behind a CLI-orchestrated gRPC architecture. | official docs | Cadastre may use a plugin runtime boundary but must restrict adapter outputs. |
| CloudQuery source collection can feed multiple destinations and optional transformers. | official docs | Cadastre fan-out must occur from persisted raw/lakehouse state, not destination streams. |
| Incremental tables use state backends and aim for no gaps with possible duplicates. | official docs | Cadastre should add cursor-state and duplicate handling contracts. |
| Destination stale deletion is a declared write mode. | official docs | Cadastre must reject destination cleanup as source absence proof. |

### Source-grounded inferences

1. CloudQuery's table/resolver structure is a strong source-dataset authoring model for Cadastre.
2. CloudQuery's state backend flush model implies cursor state can be output-affecting and must be version-manifested in Cadastre.
3. CloudQuery's multi-destination architecture is operationally useful but architecturally unsafe if copied into Cadastre without raw-first persistence.

### Cadastre transfer candidates

| Candidate | Cadastre contract change | Stance |
| --- | --- | --- |
| `incremental_table_sync` pattern | Add to `CollectionStepPattern` with cursor state, duplicate behavior, and end-of-collection proof. | `adopt` |
| `poll_with_cursor` pattern | Add to `CollectionStepPattern` for API cursors independent of table abstraction. | `adopt` |
| table/resolver package layout | Add to `SourcePackageDeveloperContract` as recommended internal decomposition. | `adapt` |
| state backend | Convert to `StageStateRecord` plus `VersionManifest`. | `adopt` |
| transformer chain | Keep as parser/normalizer stages only. | `adapt` |

### Concepts that must not transfer

| Pattern | Why unsafe | Cadastre-safe alternative |
| --- | --- | --- |
| Destination `overwrite-delete-stale` authorizes absence | Destination state is not source completeness. | `SourceCompletenessProfile` plus `SourceCompletenessReceipt` authorizes absence/cleanup. |
| Append-mode duplicate tolerance without explicit raw duplicate status | Silent duplicate handling breaks replay and evidence accounting. | `RawRecord.record_status = duplicate` or declared dedupe rule. |
| Source plugin emits normalized graph-like records | Violates adapter output boundary. | Adapter emits raw evidence only. |
| Transformer plugin changes authoritative meaning before raw persistence | Obscures evidence lineage. | Persist raw first; transform in parser/normalizer stages. |

### Gaps, risks, stale assumptions, and unresolved questions

| Type | Item |
| --- | --- |
| Gap | Exact CloudQuery plugin protocol schema and source/destination code paths were not inspected. |
| Risk | Cursor state may be scoped by table, sync group, source config, and destination behavior; Cadastre must scope its cursor state explicitly. |
| Stale assumption | Current destination defaults and plugin SDK behavior may change; recheck before PRD amendment. |
| Open question | Whether CloudQuery's signed plugin acquisition or verification mechanisms provide patterns worth adapting into `PackageArtifact` was not established. |

## Steampipe independent analysis

### Source inventory and inspection limits (Steampipe)

| Field | Required content |
| --- | --- |
| Source name | `turbot/steampipe`; Steampipe official docs; plugin authoring docs. |
| URL or path | Public repository and docs pages inspected through web. |
| Evidence class | `source_code`, `official_docs`, `example`, `inference`. |
| Commit, tag, branch, or version | Repository branch `develop` was inspected from GitHub page. Exact commit was not pinned. |
| Inspection date | 2026-05-16. |
| Scope inspected | Repository README/layout; architecture docs; writing plugins docs; concurrency/rate-limit docs; query/mod docs; CLI timeout documentation. |
| Scope not inspected | Full FDW source, plugin manager source, SQLite/export modes, every plugin, tests, release artifacts, local query execution, real source APIs. |
| Reliability | High for official architecture and plugin behavior; medium for distribution modes not deeply inspected. |

### Project purpose and non-purpose (Steampipe)

Steampipe is a live query and exploration system. It exposes external APIs as SQL tables through a Postgres foreign data wrapper and plugins. Its purpose is low-friction source inspection, security/control query packs, and query-driven API access. It is not a durable raw evidence lake, not a deterministic replay engine, not a source completeness engine, and not a bitemporal fact store.

### Repository layout and major modules (Steampipe)

The public repository contains command and package areas such as `cmd`, `pkg`, and `tests`, and the README presents Steampipe as a zero-ETL system for querying cloud services and APIs in real time with SQL.[^16]

Steampipe plugin authoring docs structure plugins around table definition files, columns, list/get/hydrate functions, connection configuration, and generated documentation/examples. This structure is valuable for developer ergonomics but must remain tooling-side in Cadastre unless production capture contracts are added.

### Architecture and component boundaries (Steampipe)

Steampipe architecture docs describe a Postgres FDW architecture. The FDW delegates API-specific behavior to plugins, and plugins communicate over gRPC. The plugin provides tables and functions that hydrate table rows on demand.[^17]

Cadastre implication: this runtime boundary is useful for live source inspection and mapping-bundle authoring, not for production collection. Cadastre production collection must be stage-driven and raw-record-persistent, not query-demand-driven.

### Data model and record/event/schema model (Steampipe)

Steampipe exposes remote APIs as SQL tables. Plugin docs define tables with columns and types, including JSON columns. A foreign table stores no data; table contents are hydrated when a query executes.[^18]

Steampipe's execution flow routes SQL to Postgres, then FDW, then plugin hydrate functions. `List` or `Get` functions produce base rows, while hydrate functions populate additional columns. Column projection means only hydrate functions required by the selected columns are invoked; key qualifiers can select `Get` instead of `List`, improving API efficiency.[^18]

Cadastre transfer: table-like source API modeling can improve source-schema import, mapping authoring, and fixture generation. Live table rows must not become `RawRecord` unless Cadastre explicitly captures the exact request, response, source config, caller, scope, timestamp, payload bytes, and validation artifacts through a production or validation capture contract.

### Adapter, plugin, connector, or processor interface (Steampipe)

Steampipe plugin interfaces include:

| Interface element | Behavior | Cadastre applicability |
| --- | --- | --- |
| table definition | Declares table name, columns, list/get/hydrate functions. | Useful as source-dataset shape for authoring. |
| `List` function | Fetches many rows, often with pagination. | Comparable to collection step, but live-query-only by default. |
| `Get` function | Fetches one row when key qualifiers are present. | Useful source-call optimization pattern. |
| hydrate functions | Populate columns, declare dependencies and concurrency. | Useful mapping authoring pattern; unsafe for production unless captured deterministically. |
| transform functions | Convert raw API data into SQL column values. | Mapping reference only; must not bypass Cadastre raw/silver contracts. |
| connection config | Resolves credentials and source-specific settings. | Reference for `SourceInstanceConfigSchema`. |

### Configuration surfaces, defaults, and bounds (Steampipe)

Steampipe supports connection configuration and credentials, plugin-specific settings, query execution, and rate/concurrency limiters. CLI query documentation identifies query timeout behavior, including a default of zero for no timeout in one query command context.[^19] Plugin docs and checklists require credential/config validation, pagination, retry/backoff, context cancellation, and account-based tests.[^20]

Cadastre must not inherit an unbounded default timeout. Every production source call already requires bounded `SourceCallPolicy.per_request_timeout_seconds`. Steampipe's no-timeout behavior is acceptable only for local interactive source exploration, not production collection.

### Control flow and data flow (Steampipe)

```text
SQL query
  -> Postgres planner
  -> Steampipe FDW
  -> plugin gRPC call
  -> List/Get function
  -> hydrate dependencies based on projected columns and predicates
  -> SQL row result
```

Cadastre-safe tool mode:

```text
source_exploration request
  -> live_source_probe package binding
  -> connection config validation
  -> bounded List/Get/hydrate execution
  -> non-authoritative probe result + diagnostic records
  -> optional captured fixture only when redaction and fixture policy pass
```

### State, cursor, offset, snapshot, watermark, or replay model (Steampipe)

Steampipe is live-query-oriented. It does not provide durable Cadastre-like raw evidence, source completeness, or deterministic replay by default. Any API pagination or rate-limit behavior occurs inside plugin functions and query execution.

Cadastre transfer: `live_source_probe` must have `state_kind = none` by default and `production_writes_allowed = false`. When a probe captures a replay fixture, the fixture must store request/response bytes, redaction decisions, source-call policy, plugin version, config hash, and recorded time.

### Error handling, retry, rate limit, partial collection, and backpressure (Steampipe)

Steampipe has explicit concurrency and rate limiter concepts. Official docs describe HCL and Go limiters, `max_concurrency`, token buckets, scope values, and diagnostic `_ctx` information that can show hydrate functions, scope values, limiters, and delay.[^21]

Cadastre transfer:

| Steampipe pattern | PRD refinement |
| --- | --- |
| predicate pushdown | Add `source_filter_pushdown_rule` as optional source-call optimization metadata. It must not change output semantics. |
| projection pushdown | Add `requested_field_set` to live probe diagnostics; production collection must declare all output-affecting fields. |
| rate limiter scopes | Add `rate_limit_scope_key` and `rate_limit_scope_value_rule` to `SourceCallPolicy`. |
| `_ctx` diagnostics | Add stable diagnostic fields for source-call function, limiter, delay, retry count, and scope. |

Partial live query results must not authorize completeness. A canceled or timed-out live query may emit probe diagnostics, not source completeness receipts, unless a Cadastre capture profile explicitly maps the run to a validation-only fixture.

### Source completeness and absence semantics (Steampipe)

Steampipe live query results are not source completeness evidence by default. A query returning zero rows does not prove absence; it may reflect query predicate, projection, permission, API failure, rate limit, time, plugin behavior, or partial hydration.

Cadastre rule: `live_source_probe` outputs must have `authority_class = non_authoritative_metadata` unless a separate production `adapter_collection` run captures raw records and completeness receipts under the normal PRD contracts.

### Validation, tests, fixtures, and golden-output patterns (Steampipe)

Steampipe's plugin release checklist emphasizes configuration checks, pagination, retry/backoff, concurrency, context cancellation, and tests using real accounts.[^20] Cadastre should adapt these into validation matrix rows, but Cadastre needs stronger replay and negative tests:

| Required Cadastre fixture | Reason |
| --- | --- |
| predicate pushdown fixture | Proves pushed filter equals post-filtered full result or is marked optimization-only. |
| projection pushdown fixture | Proves omitted hydrate columns do not affect production output. |
| rate limiter fixture | Proves limiter delay/diagnostic behavior. |
| canceled query fixture | Proves no completeness or raw production output. |
| permission-limited query fixture | Proves visibility loss maps to `SourceCompletenessReceipt` only in production collection, not live probe. |

### Operational model, deployment, observability, and security (Steampipe)

Steampipe's operational model is operator-driven SQL exploration and mod/control execution. Query packs and controls are useful as read-only source exploration and mapping-development aids. Credentials and connection configuration are resolved by the plugin environment.

Cadastre must keep these workflows out of production mutation. A live probe may support mapping authors and operators, but must not feed gold facts, identity decisions, graph deltas, or completeness directly.

### Extension points and developer workflow (Steampipe)

Steampipe's strongest developer patterns are table-per-file organization, column metadata, List/Get separation, hydrate dependencies, retry/backoff conventions, context cancellation, and query examples. Cadastre can adapt these into source package authoring guidance and source-schema import tooling.

### Confirmed findings (Steampipe)

| Finding | Evidence class | Contract implication |
| --- | --- | --- |
| Steampipe exposes APIs as SQL tables through Postgres FDW and plugin gRPC. | official docs | Add non-authoritative `live_source_probe`; do not use for production raw by default. |
| Foreign tables store no data and rows are hydrated at query time. | official docs | Live rows are not durable raw evidence. |
| List/Get/Hydrate functions and predicate/projection behavior minimize source calls. | official docs | Add optimization metadata and diagnostics. |
| Rate limiters and `_ctx` diagnostic data expose limiter scopes and delays. | official docs | Add stable rate-limit diagnostics. |

### Source-grounded inferences (Steampipe)

1. Steampipe's table model can improve Cadastre source-schema authoring and source API introspection.
2. Query-driven execution creates nondeterminism for production collection unless every request/response and execution setting is captured.
3. Hydrate dependency graphs are useful for mapping bundle development but unsafe as hidden production dependencies.

### Cadastre transfer candidates (Steampipe)

| Candidate | PRD change | Stance |
| --- | --- | --- |
| `live_source_probe` | Add collection pattern or separate tool mode with no production writes. | `adapt` |
| List/Get distinction | Add optional source-call optimization metadata. | `adapt` |
| hydrate dependencies | Add developer-tooling concept; forbid hidden production dependencies. | `adapt` |
| `_ctx` diagnostics | Add source-call diagnostic fields. | `adopt` |
| query packs/controls | Use as non-authoritative source inspection aids. | `study` |

### Concepts that must not transfer (Steampipe)

| Pattern | Why unsafe | Cadastre-safe alternative |
| --- | --- | --- |
| Live query result as `RawRecord` | No durable payload hash, replay manifest, completeness, or source-call receipt by default. | Capture through `RecordedSourceFixture` or production adapter collection. |
| Zero-row SQL result as absence proof | Query may be partial, filtered, permission-limited, or canceled. | `SourceCompletenessReceipt` plus active profile. |
| Hydrate function side effect as production mapping | Hidden dependency breaks replay. | Declare every output-affecting dependency in `SourceStageStep` and `VersionManifest`. |

### Gaps, risks, stale assumptions, and unresolved questions (Steampipe)

| Type | Item |
| --- | --- |
| Gap | SQLite extension and export tooling were not deeply inspected. |
| Risk | Query-time behavior can change with predicates, projection, plugin version, credentials, and API rate limits. |
| Stale assumption | Plugin SDK details may change; verify exact runtime for any production tooling transfer. |
| Open question | Whether Steampipe's mod/control model should become a Cadastre validation scenario authoring interface requires separate study. |

## Apache NiFi independent analysis

### Source inventory and inspection limits (Apache NiFi)

| Field | Required content |
| --- | --- |
| Source name | `apache/nifi`; Apache NiFi official documentation, user guide, developer guide, administration guide, release metadata. |
| URL or path | Public repository, release notes, and docs inspected through web. |
| Evidence class | `source_code`, `official_docs`, `release_metadata`, `test`, `example`, `inference`. |
| Commit, tag, branch, or version | Repository branch `main` inspected from GitHub page; current release metadata showed Apache NiFi 2.9.0 released 2026-04-10. Exact repository commit was not pinned. |
| Inspection date | 2026-05-16. |
| Scope inspected | Repository README/layout; FlowFile overview; In Depth docs; user guide queue/backpressure/retry settings; developer guide provenance and state management; REST API docs; admin guide state providers and repositories. |
| Scope not inspected | Full Java source, processor implementations, cluster code, provenance repository internals, test suites, local flow execution, Registry implementation details. |
| Reliability | High for official docs and release metadata; medium for implementation internals not source-inspected. |

### Project purpose and non-purpose (Apache NiFi)

NiFi is an ingestion and flow-management system. It provides processor-based dataflows, queues, FlowFiles, provenance, replay, state management, process groups, controller services, versioned flows, REST APIs, access control, and operational observability. It is not a Cadastre raw/silver/gold temporal asset intelligence model, and NiFi provenance is not a substitute for Cadastre source evidence or bitemporal fact lineage.

### Repository layout and major modules (Apache NiFi)

The public repository contains NiFi runtime modules, extension bundles, API/service components, docs, and tests. The README describes NiFi as a system for automated data movement, transformation, routing, retry/backoff, provenance, versioned pipelines, and clustering.[^22]

For Cadastre, the relevant modules are conceptual rather than source-file-specific: FlowFile repository, content repository, provenance repository, processor API, controller services, state management, queues, process groups, REST API, access control, and registry/versioned-flow support.

### Architecture and component boundaries (Apache NiFi)

NiFi's dataflow architecture is processor-centered. Data moves as FlowFiles through processors connected by relationships and queues. Connections are bounded buffers and can enforce prioritization and backpressure.[^23]

Architecture boundary comparison:

| NiFi component | NiFi responsibility | Cadastre-safe analogue |
| --- | --- | --- |
| FlowFile | Content pointer plus attributes. | Raw collection unit or batch metadata, not raw payload contract by itself. |
| Processor | Performs source, transform, route, sink, or utility logic. | Stage step or implementation mechanism below `SourceStageDAG`. |
| Relationship | Named output path from processor. | Failure/success/partial routing state in `SourceStageLifecycleResult`. |
| Queue/connection | Buffer with backpressure/expiration/prioritizer. | Runtime mechanism, not source completeness. |
| Provenance repository | Records lineage events and replay metadata. | Ingestion-provenance telemetry distinct from `EvidenceRef`. |
| StateManager | Local or cluster-scoped processor state. | `StageStateRecord` when output-affecting. |
| Process group | Logical flow composition. | DAG/stage grouping, if versioned and explicit. |

### Data model and record/event/schema model (Apache NiFi)

NiFi's FlowFile model is two-part: attributes and content. In-depth docs describe a FlowFile record as a pointer to content plus attributes, with separate repositories for FlowFile metadata, content, and provenance. Content is treated with copy-on-write semantics.[^24]

Developer docs define provenance event types including receive, fetch, send, route, fork, join, attributes modified, remote invocation, replay, expire, and unknown.[^25]

Cadastre transfer: NiFi's provenance taxonomy should inform a Cadastre `IngestionProvenanceEvent`, but this taxonomy must be distinct from source evidence. A provenance event can prove what the ingestion system did, not what the external source authoritatively asserted.

### Adapter, plugin, connector, or processor interface (Apache NiFi)

NiFi processors receive FlowFiles or create them, read/update attributes, write content, route to relationships, record provenance, and access state through local or cluster scope. Controller services provide reusable external clients, credentials, and shared services. Extension APIs allow custom processors and services.

Cadastre transfer: a NiFi-like runtime may implement an adapter, but every observable production output must still be captured by Cadastre contracts. A processor's route to `success` does not imply source completeness; a route to `failure` must be mapped to stable diagnostics and completeness states.

### Configuration surfaces, defaults, and bounds (Apache NiFi)

NiFi user documentation identifies queue backpressure defaults of 10,000 objects and 1 GB data size, default expiration of zero seconds, default penalty duration of 30 seconds, and default yield duration of 1 second. It also documents auto-retry behavior and prioritizers.[^26]

Cadastre transfer: backpressure and routing settings are runtime execution settings. They may appear in `SourceStageLifecycleResult` diagnostics, but only output-affecting settings must be in `VersionManifest`. Queue settings must not authorize completeness or absence.

### Control flow and data flow (Apache NiFi)

```text
source processor creates FlowFile
  -> FlowFile attributes + content claim
  -> queue connection
  -> downstream processors
  -> relationships route success/failure/retry
  -> provenance events record flow lineage
  -> optional replay from provenance/content
```

Cadastre-safe mapping:

```text
implementation flow event
  -> IngestionProvenanceEvent or DiagnosticRecord
  -> RawRecord only if payload persisted with Cadastre fields and hash
  -> SourceCompletenessReceipt only if source profile evaluates the source scope
  -> downstream Cadastre stages from lakehouse, not from transient queue alone
```

### State, cursor, offset, snapshot, watermark, or replay model (Apache NiFi)

NiFi state management supports local and cluster-scoped state through a `StateManager`. Admin docs identify local and cluster providers such as a write-ahead local state provider and ZooKeeper-backed clustered state.[^27]

NiFi provenance replay can replay content from provenance events when content is available and access policy permits. User docs also tie replay privileges to access policies for provenance and data.[^28]

Cadastre transfer: state and replay are useful patterns but insufficient as Cadastre replay. A NiFi replay event must not replace `RawRecord`, `VersionManifest`, `LakehouseSnapshotRef`, `StageStateRecord`, or `SourceCompletenessReceipt`. If a NiFi-backed adapter uses processor state or queue offsets to affect output, those values must be materialized as `StageStateRecord` rows.

### Error handling, retry, rate limit, partial collection, and backpressure (Apache NiFi)

NiFi's queue, penalty, yield, retry, expiration, and relationship routing model is valuable for operational robustness. It should not appear directly as Cadastre source truth. Cadastre should represent it as:

| NiFi behavior | Cadastre record |
| --- | --- |
| queue backpressure exceeded | `DiagnosticRecord` with `BACKPRESSURE_LIMIT_EXCEEDED`; stage may pause or fail according to lifecycle. |
| FlowFile expired | `DiagnosticRecord` and possibly adapter error; raw completeness must fail or mark gap if source data was lost. |
| penalized FlowFile | Retry diagnostic; no output change until terminal route. |
| processor yielded | Operational health signal; not a completeness state. |
| failure relationship | Adapter/parser/processor diagnostic mapped to stable error code. |
| replay event | Ingestion provenance, not source evidence. |

### Source completeness and absence semantics (Apache NiFi)

NiFi can prove that a FlowFile traversed a flow or that a processor emitted a route. It cannot by itself prove the external source was complete. A complete queue drain or a provenance join does not authorize absence or cleanup.

Cadastre should add a `provenance_event_closure` evidence class to `SourceCompletenessReceipt`, but the receipt still needs source-call, source-scope, source-permission, expected-count, cursor-exhaustion, or source-declared completeness evidence.

### Validation, tests, fixtures, and golden-output patterns (Apache NiFi)

NiFi extension development supports unit-style processor tests and provenance/state APIs. Cadastre should adapt:

| Fixture class | Required Cadastre test |
| --- | --- |
| `flowfile_batch` | Given a batch of FlowFiles, raw persistence must preserve payload hashes and attributes selected by policy. |
| `flowfile_fork_join` | Parent/child provenance must not create identity or graph edges. |
| `queue_backpressure` | Stage must pause/fail deterministically and emit diagnostic. |
| `flowfile_replay` | Replay must not bypass Cadastre manifest checks. |
| `processor_state_loss` | Missing state must emit `STATE_RECORD_MISSING` and block watermark. |

### Operational model, deployment, observability, and security (Apache NiFi)

NiFi provides UI-driven flow configuration, REST APIs for programmatic control, provenance viewing, replay, clustering, and access policies. Docs identify REST API access for realtime command/control and security policies for provenance and content replay.[^29]

Cadastre caution: UI-driven flow configuration must not become a production contract unless every observable behavior is captured in versioned artifacts, checksums, lifecycle states, validation matrix rows, and `VersionManifest`.

### Extension points and developer workflow (Apache NiFi)

NiFi extensions include processors, controller services, reporting tasks, and custom services. Cadastre may use a NiFi-based runtime as an implementation mechanism only when:

1. Flow definitions are versioned and checksummed.
2. Processor versions and controller service configs appear in `VersionManifest` when output-affecting.
3. FlowFile content becomes Cadastre `RawRecord` before downstream truth derivation.
4. Provenance is stored as non-authoritative ingestion provenance.
5. Replay cannot bypass PRD replay gates.

### Confirmed findings (Apache NiFi)

| Finding | Evidence class | Contract implication |
| --- | --- | --- |
| FlowFile consists of attributes and content, with repository separation. | official docs | Cadastre may model ingestion provenance separately from raw evidence. |
| Connections are bounded queues with prioritizers and backpressure. | official docs | Add runtime diagnostics, not completeness authority. |
| Provenance event types include RECEIVE, FETCH, SEND, ROUTE, FORK, JOIN, REPLAY, and EXPIRE. | official docs | Add ingestion-provenance taxonomy. |
| NiFi has local and cluster-scoped state providers. | official docs | Map output-affecting state to `StageStateRecord`. |

### Source-grounded inferences (Apache NiFi)

1. NiFi's provenance model is a rich operational lineage source but is not source evidence.
2. FlowFiles can support batch and parent/child source collection patterns when the external source evidence is separately preserved.
3. Queue/backpressure controls should influence lifecycle and health, not gold truth.

### Cadastre transfer candidates (Apache NiFi)

| Candidate | PRD change | Stance |
| --- | --- | --- |
| ingestion-provenance event taxonomy | Add `IngestionProvenanceEvent` or `DiagnosticRecord` subtypes. | `adapt` |
| `flowfile_batch` fixture class | Add to `RecordedSourceFixture`. | `adopt` |
| backpressure diagnostics | Add stable diagnostic codes and lifecycle handling. | `adapt` |
| local/cluster state model | Add scope fields to `StageStateRecord`. | `adapt` |

### Concepts that must not transfer (Apache NiFi)

| Pattern | Why unsafe | Cadastre-safe alternative |
| --- | --- | --- |
| Provenance replay as Cadastre replay | It replays flow content, not Cadastre raw/silver/gold/projection contracts. | Cadastre replay from persisted raw and `VersionManifest`. |
| Queue drain as completeness | It proves internal queue state, not source scope coverage. | `SourceCompletenessReceipt` evaluated against source profile. |
| UI flow as production contract | UI state can be mutable and underspecified. | Versioned, checksummed `SourceStageDAG` and package artifacts. |
| FlowFile attributes as canonical facts | Attributes are transport metadata. | Parse into silver and derive gold separately. |

### Gaps, risks, stale assumptions, and unresolved questions (Apache NiFi)

| Type | Item |
| --- | --- |
| Gap | Full NiFi processor source, Registry internals, and cluster failure semantics were not inspected. |
| Risk | Replay and provenance are compelling enough that implementers may over-trust them as source evidence. |
| Stale assumption | NiFi defaults and access-control behavior should be rechecked against the exact deployed version. |
| Open question | Whether a NiFi-backed Cadastre adapter runtime is worth specifying as an optional implementation profile requires performance and security review. |

## Data Prepper independent analysis

### Source inventory and inspection limits (Data Prepper)

| Field | Required content |
| --- | --- |
| Source name | `opensearch-project/data-prepper`; Data Prepper official docs, examples, and release metadata. |
| URL or path | Public repository and docs pages inspected through web. |
| Evidence class | `source_code`, `official_docs`, `release_metadata`, `example`, `inference`. |
| Commit, tag, branch, or version | Repository branch `main` inspected from GitHub page. Search/release metadata surfaced Data Prepper 2.15.0 on 2026-04-06; exact commit/tag was not pinned. |
| Inspection date | 2026-05-16. |
| Scope inspected | Repository root/module layout; README; pipeline and key concept docs; configuration docs; end-to-end acknowledgment docs; source coordination docs; peer forwarding docs; expression syntax docs. |
| Scope not inspected | Full Java source, plugin implementations, tests, OpenSearch sink code, Iceberg CDC examples, local pipelines, admin API behavior under runtime. |
| Reliability | High for official pipeline model and configuration defaults; medium for source-specific examples and release metadata not fully source-verified. |

### Project purpose and non-purpose (Data Prepper)

Data Prepper is an ingestion pipeline service for observability/security data that receives, buffers, transforms, routes, aggregates, and sinks events, commonly to OpenSearch. It is relevant to Cadastre for pipeline-stage configuration, plugin boundaries, buffer/ack semantics, conditional routing, source coordination, and distributed operation. It is not a Cadastre lakehouse, not a source completeness authority, not a graph model, and not a bitemporal fact engine.

### Repository layout and major modules (Data Prepper)

The public repository includes modules such as `data-prepper-api`, `data-prepper-core`, `data-prepper-event`, `data-prepper-expression`, `data-prepper-plugin-framework`, `data-prepper-plugins`, `examples`, `e2e-test`, and test modules. The README describes Data Prepper as a server-side data collector that can accept, filter, transform, enrich, and route data at scale.[^30]

### Architecture and component boundaries (Data Prepper)

Data Prepper key concepts define a pipeline as one source, one or more sinks, an optional buffer, and optional processors. The default buffer is `bounded_blocking`; if processors are omitted, the default processor behavior is no-op. Pipelines are YAML-defined and may be chained.[^31]

Cadastre transfer: this model is useful below `SourceStageDAG`. A Data Prepper pipeline may implement an adapter stage, but the PRD stage boundary remains authoritative. Data Prepper's `source`, `buffer`, `processor`, and `sink` must not be mapped one-for-one to Cadastre raw/silver/gold/projection authority.

### Data model and record/event/schema model (Data Prepper)

Data Prepper processes events through sources, buffers, processors, and sinks. Its expression language can inspect event fields, run functions, and support conditional routing. Official docs show JSON pointer-style field access, operators, and routing expressions.[^32]

Cadastre transfer: conditional expressions are useful for routing validation and mapping tests, but production routing rules must be versioned, deterministic, typed, and included in `VersionManifest` when output-affecting.

### Adapter, plugin, connector, or processor interface (Data Prepper)

Data Prepper plugin boundaries are source, buffer, processor, and sink. Sources ingest events, buffers store in-flight events, processors mutate/filter/aggregate events, and sinks emit to external destinations. The plugin framework and schema modules support plugin configuration and validation.

Cadastre-safe mapping:

| Data Prepper plugin | Cadastre interpretation |
| --- | --- |
| source | Implementation of adapter source calls or report reads. |
| buffer | Runtime queue; not a PRD data layer. |
| processor | Parser/normalizer implementation mechanism only when bound to stage output permissions. |
| sink | Writes to Cadastre raw table or diagnostics only in adapter stage; OpenSearch sink assumptions do not transfer. |
| pipeline | Implementation DAG below `SourceStageDAG`; must be versioned if production. |

### Configuration surfaces, defaults, and bounds (Data Prepper)

Pipeline docs identify fields such as `workers`, `delay`, `buffer`, processors, sinks, and routes. Defaults include `workers = 1`, `delay = 3000 ms`, default bounded blocking buffer, and no-op processor behavior when processors are absent. Configuration docs also identify shutdown timeout and peer-forwarder defaults such as port, request timeout, and thread counts.[^33]

Cadastre transfer: when a Data Prepper-like runtime affects output order, timing, retries, aggregation, or ack behavior, these settings must appear in `VersionManifest` or a referenced runtime profile. Cadastre should not inherit default no-op processors or delays without explicit versioned defaults.

### Control flow and data flow (Data Prepper)

```text
source plugin
  -> buffer
  -> processor chain in declared order
  -> conditional routing
  -> sink plugin
  -> optional end-to-end acknowledgement to source
```

Cadastre-safe adaptation:

```text
source event batch
  -> Cadastre adapter source-call policy
  -> optional runtime buffer
  -> raw record persistence sink
  -> ack may be emitted after raw persistence and receipt write
  -> downstream stages consume lakehouse rows
```

### State, cursor, offset, snapshot, watermark, or replay model (Data Prepper)

Data Prepper source coordination docs describe distributed partition assignment and lease-state management. Source coordination can prevent duplicate work across nodes and uses partition states such as assigned, unassigned, closed, and completed.[^34] Peer forwarding supports stateful aggregation across nodes for selected processors.[^35]

Cadastre transfer: source coordination leases and aggregation state are output-affecting when they decide which source partitions are read or when aggregation changes emitted data. They must be represented as `StageStateRecord` rows with `state_kind = source_coordination_lease` or `processor_state` and included in `VersionManifest`.

### Error handling, retry, rate limit, partial collection, and backpressure (Data Prepper)

Data Prepper pipeline docs describe end-to-end acknowledgments: a source can monitor batches and wait for positive acknowledgment after records reach final sinks; negative acknowledgments, missing acknowledgments, and timeouts can affect source retry or delete behavior.[^33] Data Prepper also provides buffer and shutdown settings.

Cadastre transfer: acknowledgments should be modeled explicitly, but an ack is not source completeness. Add fields:

| Proposed field | Location | Default | Behavior |
| --- | --- | --- | --- |
| `ack_required` | `SourceCallPolicy` or `SourceStageStep` | `false` | When true, batch is not terminal until ack completes. |
| `ack_timeout_seconds` | `SourceCallPolicy` | `300`, bounds `1..3600` | Timeout emits `ACK_TIMEOUT` and blocks completeness/watermark. |
| `ack_state_ref` | `StageStateRecord` | required when ack affects retry/delete | Records batch ID, source partition, ack status, and sink persistence ref. |
| `ack_evidence_class` | `SourceCompletenessReceipt.table_completeness_evidence_refs` or new field | optional | May support delivery evidence but not source completeness by itself. |

### Source completeness and absence semantics (Data Prepper)

End-to-end ack means a batch reached a sink or final processing stage, not that the source was completely read. Conditional routing and dropped events can make the gap worse: an ack can succeed while events were intentionally dropped or routed away. Therefore an ack may support delivery or raw-persistence evidence, but it cannot authorize absence, cleanup, retraction, or source watermark advancement alone.

### Validation, tests, fixtures, and golden-output patterns (Data Prepper)

Cadastre should add fixture classes for:

| Fixture class | Required test |
| --- | --- |
| `pipeline_batch` | Given source batch, processors, and routes, raw output and diagnostics are deterministic. |
| `pipeline_ack_success` | Ack occurs only after raw persistence and receipt write. |
| `pipeline_ack_timeout` | Timeout emits `ACK_TIMEOUT` and blocks watermark. |
| `conditional_route` | Every route predicate has deterministic truth table cases. |
| `stateful_aggregation` | Processor state is materialized and replay-compatible. |
| `source_coordination_lease` | Duplicate lease acquisition cannot produce duplicate production raw records without duplicate status. |

### Operational model, deployment, observability, and security (Data Prepper)

Data Prepper supports pipeline YAML, worker threads, buffers, peer forwarding, metrics, admin API, circuit breakers, source coordination, and distributed operation. For Cadastre, these are implementation mechanisms and health signals unless a specific field affects output.

### Extension points and developer workflow (Data Prepper)

Data Prepper plugins and schemas are useful references for package validation and runtime extension. Cadastre should not adopt OpenSearch sink assumptions as storage, graph, or lakehouse contracts. A Data Prepper sink writing to OpenSearch is a serving or observability mechanism, not a Cadastre authoritative table write.

### Confirmed findings (Data Prepper)

| Finding | Evidence class | Contract implication |
| --- | --- | --- |
| A pipeline has one source, optional buffer, optional processors, and one or more sinks. | official docs | Useful implementation mechanism below `SourceStageDAG`. |
| Default buffer is bounded blocking and default processor is no-op. | official docs | Defaults must be explicit if adopted. |
| End-to-end acknowledgments can signal positive/negative/timeout delivery outcomes. | official docs | Add ack fields but do not equate with completeness. |
| Source coordination leases partitions across nodes. | official docs | Add state records for leases when output-affecting. |

### Source-grounded inferences (Data Prepper)

1. Pipeline ack can protect delivery but not source coverage.
2. Conditional routing expressions can improve validation when their syntax and evaluation are fixed and fixture-tested.
3. Stateful aggregation and peer forwarding are high-risk for replay unless state is captured.

### Cadastre transfer candidates (Data Prepper)

| Candidate | PRD change | Stance |
| --- | --- | --- |
| `pipeline_acknowledged_batch` pattern | Add collection pattern with ack status and timeout. | `adapt` |
| conditional routing expression | Add mapping-validation or event-routing rule artifact, not source truth. | `adapt` |
| source coordination leases | Add `StageStateRecord.state_kind`. | `adopt` |
| stateful aggregation state | Add processor-state record and validation fixtures. | `adapt` |

### Concepts that must not transfer (Data Prepper)

| Pattern | Why unsafe | Cadastre-safe alternative |
| --- | --- | --- |
| End-to-end ack as source completeness | Ack only proves delivery to sink path, not source enumeration. | Completeness profile and receipt with source evidence. |
| OpenSearch sink as Cadastre storage | Sink is destination-specific and serving-oriented. | Authoritative lakehouse write contract. |
| Conditional route as omission/source authority rule | Route may be implementation-specific. | Versioned mapping rule with fixture coverage. |
| Peer-forwarded aggregation as deterministic output without state | Distributed aggregation state can be hidden. | `StageStateRecord` plus replay validation. |

### Gaps, risks, stale assumptions, and unresolved questions (Data Prepper)

| Type | Item |
| --- | --- |
| Gap | Data Prepper plugin source and Iceberg/CDC examples were not deeply inspected. |
| Risk | Pipeline YAML can hide output-affecting behavior unless every processor/route/default is versioned. |
| Stale assumption | Defaults and release version must be rechecked before adoption. |
| Open question | Whether Data Prepper's admin API metrics should map into Cadastre health records requires source/API inspection. |

## Debezium independent analysis

### Source inventory and inspection limits (Debezium)

| Field | Required content |
| --- | --- |
| Source name | `debezium/debezium`; Debezium official documentation; Debezium MySQL connector docs; signaling/incremental snapshot docs; release metadata. |
| URL or path | Public repository and docs pages inspected through web. |
| Evidence class | `source_code`, `official_docs`, `release_metadata`, `example`, `inference`. |
| Commit, tag, branch, or version | Repository branch was inspected from GitHub page. Official docs surfaced stable Debezium 3.5 and development 3.6 in current docs. Exact commit was not pinned. |
| Inspection date | 2026-05-16. |
| Scope inspected | Repository README; architecture docs; Kafka Connect connector architecture; MySQL connector snapshot modes; change-event envelope; transaction metadata; heartbeat/error/default configuration; incremental snapshot/signaling docs. |
| Scope not inspected | Full connector source, Postgres/SQL Server/Oracle connector internals, test harnesses, schema history implementation code, Kafka Connect worker code, local CDC runtime. |
| Reliability | High for official CDC architecture and connector docs; medium for source-specific defaults not checked in code. |

### Project purpose and non-purpose (Debezium)

Debezium is a change data capture platform. It monitors database transaction logs and emits committed row-level changes through Kafka Connect source connectors. It is relevant to Cadastre for database-backed sources with durable change logs, initial snapshots, incremental snapshots, offsets, schema history, heartbeats, transaction metadata, duplicate semantics, and CDC envelopes. It is not a general SaaS API ingestion model and must not be generalized to sources without durable logs and schema-history guarantees.

### Repository layout and major modules (Debezium)

The public repository includes connector modules, core components, documentation, examples, tests, and integration areas. The README describes Debezium as CDC software that monitors databases and lets applications consume row-level changes, with committed changes visible and source database logs providing ordered change streams.[^36]

### Architecture and component boundaries (Debezium)

Debezium runs as Kafka Connect source connectors. Official architecture docs describe connectors streaming change events from source databases into Kafka topics, with connector-specific integrations such as MySQL binlog and PostgreSQL logical replication.[^37]

Cadastre transfer: Kafka Connect connector lifecycle and offsets are useful for `database_change_log` sources, but Cadastre adapter output remains raw CDC event records and completeness/heartbeat receipts, not gold facts.

### Data model and record/event/schema model (Debezium)

Debezium emits structured change events. MySQL connector docs show an envelope with `before`, `after`, `source`, `op`, and timestamp fields. Operation codes include create, update, delete, and tombstone behavior. The `source` block contains connector, database, table, file, position, row, and snapshot metadata for MySQL events. Transaction metadata can identify BEGIN/END boundaries and event ordering inside transactions.[^38]

Deletes can emit a delete event and then a tombstone event with the same key and a null value.[^39] Primary key updates can be represented as delete/tombstone/create style event sequences in connector-specific behavior.[^38]

Cadastre transfer: raw CDC change events, schema-change events, tombstones, and heartbeats should become explicit `RawRecord.source_event_subtype` values. They must not by themselves determine `GoldFact.valid_time`, `GoldFact.known_time`, or source observation time.

### Adapter, plugin, connector, or processor interface (Debezium)

Debezium connector interfaces are Kafka Connect source connectors with connector configuration, snapshot mode, offset storage, schema history, topic routing, heartbeat settings, error handling, and database log reading. The common interface relevant to Cadastre is:

| Connector element | Cadastre analogue |
| --- | --- |
| connector config | `SourceInstanceConfigSchema` and `SourceInstance` fields. |
| initial snapshot | `cdc_initial_snapshot` collection pattern. |
| incremental snapshot | `incremental_table_sync` or `cdc_incremental_snapshot` pattern. |
| streaming log | `cdc_stream` or `database_change_log` pattern. |
| offset | `StageStateRecord.state_kind = cdc_offset`. |
| schema history | `StageStateRecord.state_kind = schema_history_ref` and `VersionManifest` ref. |
| heartbeat | `SourceCompletenessReceipt` evidence class for stream liveness, not completeness alone. |
| signaling table/topic | Control-plane input, not source evidence unless captured. |

### Configuration surfaces, defaults, and bounds (Debezium)

Debezium MySQL docs describe snapshot modes such as `always`, `initial`, `initial_only`, `no_data`, `never`, `recovery`, `when_needed`, `configuration_based`, and `custom`. Snapshot completion is recorded in offsets. Error/default docs show examples such as `event.processing.failure.handling.mode = fail`, heartbeat interval defaulting to `0` ms, incremental snapshot chunk size defaulting to `1024`, and incremental snapshot schema-change allowance defaults.[^40]

Cadastre transfer: add database-source configuration fields and default rules only as Cadastre-owned profiles. Do not copy all connector defaults blindly. The PRD should define closed enums for CDC pattern and snapshot mode selected by adapter packages.

### Control flow and data flow (Debezium)

```text
connector config validation
  -> optional initial snapshot
  -> snapshot completion offset
  -> transaction log streaming
  -> event envelope production
  -> offset commit
  -> schema history update when schema changes
  -> heartbeat and transaction metadata events when configured
```

Cadastre-safe flow:

```text
SourceInstance + CDC collection pattern
  -> raw CDC event persistence
  -> cdc offset StageStateRecord
  -> schema history StageStateRecord or artifact ref
  -> heartbeat/completeness receipt
  -> parser normalizes CDC events into source observations
  -> gold derivation uses Cadastre temporal rules, not log time alone
```

### State, cursor, offset, snapshot, watermark, or replay model (Debezium)

Debezium offsets include transaction-log positions such as binlog file/position for MySQL, LSN for PostgreSQL, or other connector-specific log positions. Restart behavior depends on offset and schema history availability. Docs warn that offset storage and schema history recovery matter; clearing offset storage can force snapshots depending on mode and state.[^40]

Incremental snapshots use chunking, signaling, and watermarking while streaming continues; the docs describe execute/pause/resume signals and chunk processing.[^41]

Cadastre transfer:

| State type | Required Cadastre representation |
| --- | --- |
| initial snapshot ID | `StageStateRecord.state_kind = cdc_snapshot_state`; include table, chunk, low/high watermark when available. |
| streaming offset | `StageStateRecord.state_kind = cdc_offset`; include connector, database, table, partition, log position, transaction ID when available. |
| schema history | `StageStateRecord.state_kind = schema_history_ref`; include schema history topic/table, version/ref, checksum, DDL parse status. |
| heartbeat | completeness evidence row `stream_heartbeat`; includes offset and timestamp, but no absence authority by itself. |
| signaling command | control-plane record; must be captured if output-affecting. |

### Error handling, retry, rate limit, partial collection, and backpressure (Debezium)

Debezium connector docs state that connectors can fail on invalid configuration, database connection problems, missing log positions, schema history issues, and event processing failures. The MySQL docs identify default error handling modes and heartbeat defaults.[^40]

Cadastre must map failures to stable diagnostics:

| Failure | Cadastre diagnostic | Completeness effect |
| --- | --- | --- |
| missing offset | `CDC_OFFSET_LOST` | Block watermark; require snapshot fallback or fail. |
| unavailable binlog/LSN | `CDC_LOG_POSITION_UNAVAILABLE` | Block stream continuity; partial gap. |
| schema history missing | `CDC_SCHEMA_HISTORY_MISSING` | Reject parsing/replay until restored. |
| unparseable DDL | `CDC_SCHEMA_CHANGE_UNPARSEABLE` | Quarantine schema event; block dependent mapping. |
| heartbeat timeout | `CDC_HEARTBEAT_TIMEOUT` | Mark stream liveness unknown; no absence proof. |
| duplicate event after restart | `CDC_DUPLICATE_EVENT` or duplicate raw status | Deduplicate only by declared key/offset rule. |

### Source completeness and absence semantics (Debezium)

CDC offsets, heartbeats, and snapshot completion are strong evidence for stream progress but not universal absence. A log position proves the connector consumed through a durable source-log position if the log and connector semantics are valid. It does not prove a vulnerability is absent, an asset is gone, a control passed, or a SaaS API scope was complete. Such facts require Cadastre source authority and coverage profiles.

For database-backed source tables with a durable change log, Cadastre may permit table-level source watermark advancement when:

1. Initial snapshot or incremental snapshot completed for the declared table/scope.
2. Streaming offset is continuous from snapshot boundary.
3. Schema history is available and compatible.
4. Connector heartbeat or log position proves stream liveness.
5. `SourceCompletenessProfile` permits the dataset and fact type.

### Validation, tests, fixtures, and golden-output patterns (Debezium)

Cadastre must add CDC fixture classes:

| Fixture class | Required cases |
| --- | --- |
| `cdc_initial_snapshot` | snapshot complete, snapshot interrupted, snapshot resumed, snapshot mode rejected. |
| `cdc_incremental_snapshot` | chunk overlap, low/high watermark, duplicate update during chunk, pause/resume signal. |
| `cdc_stream` | create/update/delete/tombstone, transaction boundaries, heartbeat, duplicate after restart. |
| `cdc_schema_history` | schema change, schema history missing, unparseable DDL, incompatible schema. |
| `cdc_offset_loss` | missing log position, offset reset, unsupported fallback. |

### Operational model, deployment, observability, and security (Debezium)

Debezium operates through Kafka Connect workers and connector configs. Cadastre should not require Kafka Connect as an implementation mechanism, but should require equivalent observable state and events for any CDC adapter. Secrets must remain credential references, and connector config hashes must be in `VersionManifest` when output-affecting.

### Extension points and developer workflow (Debezium)

Debezium connectors are database-specific. The source guarantees differ by database engine and connector version. Cadastre should define CDC collection patterns only for sources with durable ordered change logs and schema history. The PRD must state that CDC patterns are not transferable to SaaS APIs, vulnerability scanners, endpoint tools, or exports without equivalent log guarantees.

### Confirmed findings (Debezium)

| Finding | Evidence class | Contract implication |
| --- | --- | --- |
| Debezium uses Kafka Connect source connectors to stream committed database changes. | official docs | Add CDC-specific collection patterns and state records. |
| Change events include envelope fields such as `before`, `after`, `source`, `op`, and connector/source timestamps. | official docs | Add raw source-event subtypes and parser rules. |
| Snapshot modes are explicit and snapshot completion is recorded in offsets. | official docs | Add snapshot state and validation. |
| Incremental snapshots use signals, chunks, and watermarks while streaming can continue. | official docs | Add chunk/watermark state and duplicate handling. |
| Heartbeats default off in at least one documented config context. | official docs | Cadastre heartbeat requirement must be explicit, not assumed. |

### Source-grounded inferences (Debezium)

1. Debezium provides the strongest state/replay model among inspected systems, but only for durable-log database sources.
2. Schema history is as output-affecting as offsets because it determines how raw events are interpreted.
3. CDC event time, transaction log time, and connector processing time must be separated in Cadastre.

### Cadastre transfer candidates (Debezium)

| Candidate | PRD change | Stance |
| --- | --- | --- |
| `cdc_initial_snapshot` | Add to `CollectionStepPattern`. | `adopt` |
| `cdc_stream` | Add to `CollectionStepPattern` with offset and schema-history state. | `adopt` |
| `snapshot_then_stream` | Add to `CollectionStepPattern` for snapshot-to-log continuity. | `adopt` |
| CDC raw subtypes | Add `source_event_subtype` to `RawRecord` or a CDC extension record. | `adapt` |
| schema history state | Add `StageStateRecord.state_kind = schema_history_ref`. | `adopt` |

### Concepts that must not transfer (Debezium)

| Pattern | Why unsafe | Cadastre-safe alternative |
| --- | --- | --- |
| CDC offset as valid time/known time | Offset is log position, not domain validity or platform knowledge interval. | Gold facts set valid/known time through Cadastre temporal rules. |
| Snapshot time as observation time | Snapshot is collection state, not necessarily source event time. | Preserve source event time quality separately. |
| Heartbeat as absence proof | Heartbeat proves liveness, not absence of fact in a covered domain. | Require coverage and completeness profile. |
| Generalizing CDC to SaaS APIs | APIs may lack durable ordered logs/schema history. | Use paginated API or report patterns with source completeness receipts. |

### Gaps, risks, stale assumptions, and unresolved questions (Debezium)

| Type | Item |
| --- | --- |
| Gap | Connector source and test harnesses were not inspected. |
| Risk | CDC terminology can obscure that schema history and offsets are both required for replay. |
| Stale assumption | Debezium defaults vary by connector/version; exact selected connector docs must be pinned. |
| Open question | Cadastre should specify database-specific offset shapes only after selecting supported source database families. |

## Cross-source comparison matrices

### Adapter boundary matrix

Declared scope: CloudQuery, Steampipe, NiFi, Data Prepper, Debezium, Cadastre.

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre requirement |
| --- | --- | --- | --- | --- | --- | --- |
| adapter or plugin unit | Source plugin table/resolver package. | Plugin table with List/Get/Hydrate functions. | Processor and controller service. | Source/buffer/processor/sink plugin. | Kafka Connect source connector. | `SourcePackageDeveloperContract` package component and `SourceStageStep`. |
| runtime boundary | CLI orchestrates plugins over gRPC. | Postgres FDW invokes plugin over gRPC. | Runtime invokes processors through flow graph. | Pipeline runtime invokes plugins. | Kafka Connect worker invokes connector. | Stage runtime may use plugins, but PRD stage output permissions govern outputs. |
| input contract | Plugin spec, tables, source config, state backend. | SQL query, connection config, plugin table schema. | FlowFile attributes/content, processor config, state. | Pipeline YAML, events, routes, plugin config. | Connector config, DB log, offsets, schema history. | `SourceInstance`, `SourceInstanceConfigSchema`, `SourceCallPolicy`, `StageStateRecord`. |
| output contract | Rows to CLI/destinations plus metadata. | SQL rows. | Routed FlowFiles and provenance. | Events to sinks and acks. | CDC events to Kafka topics. | Adapter may emit only `RawRecord`, `SourceCompletenessReceipt`, and adapter errors. |
| allowed side effects | Destination writes, state updates, migrations. | Live API calls, no durable table storage by default. | Queue/state/provenance/content repository writes. | Buffer writes, sink writes, source acks. | Offset commits, topic writes, schema history writes. | Source calls, declared state, raw persistence, completeness receipts; no graph/silver/gold side effects. |
| state ownership | CloudQuery state backend for incremental tables. | Mostly query/runtime/plugin; not durable raw state. | Processor state local/cluster; repositories. | Source coordination, aggregation, buffer, ack state. | Kafka Connect offsets and schema history. | Cadastre owns output-affecting state as `StageStateRecord`. |
| credential handling | Plugin config/spec; destination/source config. | Connection config and plugin credential resolution. | Controller services, sensitive properties, access policies. | Plugin config and source/sink credentials. | Connector config and worker secrets. | Credential references only; no inline secrets in manifests or raw/silver/gold. |
| config validation | Plugin spec validation and destination config. | Plugin config checks and release checklist. | Processor validation and flow validation. | Plugin schema and pipeline config validation. | Connector config validation. | `SourceInstanceConfigSchema`, `ConnectivityValidationArtifact`, `ValidationMatrix`. |
| schema declaration | Source tables/columns, primary keys. | SQL table columns and types. | FlowFile attributes; processor-specific schemas. | Event schemas and plugin config schemas. | CDC envelope and source schema history. | Source schema import, raw record type, silver mapping profile, OCSF external profile. |
| source-call behavior | Resolvers call APIs, paginate, stream rows. | List/Get/Hydrate functions call APIs. | Processors call sources or read incoming FlowFiles. | Source plugins receive/read events. | Connector reads DB snapshot/log. | Every source read must have `SourceCallPolicy`. |
| retry behavior | Plugin/source-specific plus incremental state. | Plugin-specific retry/backoff checklist. | Penalty, yield, retry relationships. | Source retry and ack retry behavior. | Connector/worker retry and failure modes. | Closed retry policy with bounds and diagnostics. |
| rate-limit behavior | Source-specific; not fully inspected. | Explicit rate limiter scopes and diagnostics. | Processor-specific throttling/yield. | Source/plugin-specific, buffer backpressure. | DB/connect backpressure, connector retry. | Source-specific mappings to platform rate-limit diagnostics and policy. |
| pagination or streaming behavior | Table pagination and incremental cursors. | List pagination; Get by key; hydrate. | FlowFile streams and queues. | Batch/stream pipelines. | Initial snapshot, incremental snapshot, streaming log. | Closed `CollectionStepPattern` plus state refs. |
| partial result behavior | Incremental duplicates possible; destination modes. | Live partial/canceled query possible. | Failure routes, expired FlowFiles, replay. | Negative/no ack, dropped/routed events. | Snapshot interruption, log gap, schema failure. | Partial states block absence/cleanup/watermark unless profile permits. |
| test fixture strategy | Plugin tests and table resources. | Plugin tests, real account tests, query examples. | Processor tests/provenance tests. | Pipeline/e2e tests and plugin tests. | Connector test harnesses and CDC fixtures. | `RecordedSourceFixture` classes by pattern plus validation matrix. |
| production activation behavior | Plugin acquisition/versioning and config. | Plugin install/connection; query execution. | Flow deployment and access control. | Pipeline deployment/config. | Connector deployment in Kafka Connect. | `PackageArtifact`, `PackageStageBinding`, lifecycle, validation, manifest. |
| Cadastre transfer stance | `adapt` plugin/table/state; `reject` destination authority. | `adapt` live probe/tooling; `reject` live evidence. | `adapt` provenance/backpressure; `reject` provenance-as-truth. | `adapt` ack/routing; `reject` ack-as-completeness. | `adopt` CDC patterns for durable logs; reject generalization. | Baseline authority. |

### State, cursor, offset, and replay matrix

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre equivalent and required PRD change |
| --- | --- | --- | --- | --- | --- | --- |
| state primitive | Incremental cursor/state backend, sync group/table. | Query context; plugin pagination state transient. | FlowFile repository, content claim, provenance, processor state. | Buffer, ack set, source-coordination lease, aggregation state. | Kafka Connect offset, snapshot state, schema history, transaction log position. | `StageStateRecord` with new `state_kind` values. |
| persistence location | State backend and destination/state storage. | Not durable by default for live query. | FlowFile/content/provenance/state repositories. | Runtime buffer/lease store/coordination store. | Offset storage and schema history topics/storage. | Cadastre lakehouse state table and manifest references. |
| monotonicity rule | Cursor should advance without gaps; duplicates possible. | None by default; query-dependent. | Queue ordering/prioritizers; provenance sequence. | Source partition states progress assigned/closed/completed. | Log offsets monotonic per partition/source log; snapshot chunk watermarks. | Define monotonicity per state kind; default no advance on failure. |
| idempotence behavior | At-least-once; destination modes handle duplicates differently. | Query rerun may re-fetch live data. | Replay/route may duplicate unless controlled. | Ack and source retry may duplicate batches. | At-least-once on failure; duplicates can occur after restart. | Raw IDs include payload/source native/offset; duplicate behavior closed enum. |
| duplicate behavior | Duplicates expected in append incremental mode. | Duplicate SQL rows possible depending API/query. | Duplicate FlowFiles possible under replay/retry. | Duplicate delivery possible after negative/no ack. | Duplicate CDC events possible after restart/fault. | `RawRecord.record_status = duplicate` or declared deterministic dedupe. |
| gap behavior | Designed for no gaps if state works; state loss causes refetch/gap risk. | No gap proof. | Expired/lost FlowFile creates internal gap, not source gap proof. | Ack failure/lease loss can create delivery gaps. | Missing log position/schema history creates gap. | `partial_known_gap`/`partial_unknown_gap` receipts. |
| restart behavior | Resume from state backend. | Re-executes live query. | Repositories/state restore flow. | Source coordination/ack retries. | Resume from offsets and schema history. | Restart requires manifest state resolution before output. |
| replay compatibility | Not Cadastre replay; state and destination influence results. | Not replayable by default. | Provenance replay limited by content availability/access. | Replay depends on buffer/source/ack state. | Replay depends on offsets, retained logs, schema history. | Replay from persisted raw and manifest; source state required only to recollect. |
| schema dependency | Table schema/PK/source config. | Table columns/hydrate transforms. | Processor/FlowFile attribute conventions. | Event schema and processor config. | Schema history and connector event envelope. | Include source schema/import/profile and schema history refs. |
| manifest requirement | CloudQuery state not enough. | Live query requires capture to be manifestable. | Flow state/provenance only if output-affecting. | Ack/lease/processor state if output-affecting. | Offsets/schema history always output-affecting. | `VersionManifest` must include every output-affecting state hash/ref. |
| Cadastre equivalent | `poll_with_cursor`, `incremental_table_sync`. | `live_source_probe`. | `flowfile_batch`, provenance diagnostics. | `pipeline_acknowledged_batch`, source coordination state. | `cdc_initial_snapshot`, `cdc_stream`, `snapshot_then_stream`. | Add state kinds, fixture classes, diagnostics. |
| required PRD change or no change | Add cursor fields. | Add non-authoritative probe mode. | Add provenance event taxonomy. | Add ack/lease fields. | Add CDC state fields. | Existing manifest/receipt rules remain authoritative. |

### Source completeness and cleanup safety matrix

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre modeling rule |
| --- | --- | --- | --- | --- | --- | --- |
| how the system knows it has all records | Full sync/table traversal or incremental cursor; destination stale modes. | It generally does not; live query returns current query result. | Flow/provenance can show internal flow completion. | Ack can show batch delivery; source coordination can show partition completion. | Snapshot completion, log continuity, offsets, heartbeats, schema history. | Only active `SourceCompletenessProfile` plus receipt may authorize completeness. |
| how it represents partial collection | State gaps, errors, destination summary. | Query cancellation/error/partial hydration. | Failure routes, expired FlowFiles, provenance gaps. | Negative/no ack/timeout, dropped/routed events, lease failures. | Snapshot interruption, missing log position, schema history failure. | Closed completeness states: partial known/unknown gap, unavailable, not attempted. |
| whether absence is meaningful | Only if source/table traversal contract proves it; destination stale delete not enough. | No by default. | No by default. | No by ack alone. | Table-level absence may be meaningful for durable DB tables under snapshot/log profile. | Profile-defined by dataset/fact/scope. |
| whether stale deletion or cleanup is supported | Yes, destination `overwrite-delete-stale`. | No persistent cleanup by default. | FlowFile expiration/repository cleanup. | Buffer/sink/source delete or ack behavior. | Tombstones/log deletes represent source DB deletes. | Cleanup only after receipt/profile authorization. |
| whether cleanup depends on destination state | CloudQuery destination mode can. | Not applicable. | Repository and queue state. | Sink/source ack state. | Kafka compacted topics/tombstones possible downstream; source delete in DB log. | Destination state must not authorize Cadastre cleanup. |
| whether expected counts exist | Source-specific. | Query result count only. | Queue/content counts internal. | Batch/event counts and ack counts. | Snapshot chunks/events; source DB counts source-specific. | `records_expected_known` and expected-count evidence class. |
| whether source visibility is proven | Source-specific config/permissions. | Not by query result alone. | Processor credentials/access policy internal. | Source plugin-specific. | DB privileges and connector access. | `visibility_state` and `source_permission_state` required. |
| how permission loss is handled | Source-specific errors. | Plugin error/empty result risk. | Access policies and failure routes. | Source/sink errors, negative ack. | Connector errors/failures. | Specific credential/permission states, not generic partial. |
| how Cadastre should model evidence | Cursor exhaustion, state refs, source-call logs. | Probe diagnostics only. | Provenance event closure as supporting operational evidence. | Ack evidence as delivery evidence. | Offset/log/schema/heartbeat evidence. | Receipt evidence classes are supporting; profile computes booleans. |
| transfer stance | `adapt`; reject destination cleanup authority. | `reject` for completeness; `adapt` for probe. | `adapt` provenance only. | `adapt` ack only. | `adopt` for DB log sources only. | Baseline. |

### Pipeline and flow-control matrix

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre equivalent / required acceptance criteria |
| --- | --- | --- | --- | --- | --- | --- |
| stage model | Source-transformer-destination sync. | Query plan/hydrate function graph. | Processor graph/process groups. | YAML pipeline. | Snapshot and streaming connector lifecycle. | `SourceStageDAG` with stage output permissions. |
| processor or step model | Table resolvers. | List/Get/Hydrate. | Processors. | Processors in order. | Connector tasks and snapshot/log readers. | `SourceStageStep` with exactly one pattern. |
| buffer or queue model | CLI streaming and destination batches. | FDW/query execution buffers. | Queues/connections with backpressure. | Explicit/default buffer. | Kafka Connect queues/topics. | Runtime mechanism; output-affecting buffer state must be explicit. |
| backpressure model | Destination/source-specific. | Rate/concurrency limiters. | Object/data backpressure thresholds. | Buffer capacity and worker delay. | Connect/Kafka/source DB pressure. | Diagnostics and lifecycle handling; no completeness by itself. |
| retry/yield/penalty model | Source-specific and state. | Plugin retry/backoff and cancellation. | Penalty/yield/retry relationships. | Ack retry and source retry. | Connector retries/failure modes. | `SourceCallPolicy` with bounded retry and stable diagnostics. |
| acknowledgment model | Destination summary/state. | SQL result returned. | FlowFile route/provenance. | End-to-end ack positive/negative/timeout. | Offset commits/producer delivery. | `ack_state_ref`; ack is delivery, not completeness. |
| conditional routing model | Transformer chain could filter. | SQL predicates and hydrate dependencies. | Relationships route FlowFiles. | Expression-based conditional routing. | Topic routing, SMTs, connector filters. | Routing rules must be deterministic, versioned, fixture-tested. |
| provenance model | Logs/sync summaries. | Query context diagnostics. | Rich provenance repository and replay. | Metrics/admin/ack events. | Source block, transaction metadata, offsets. | Distinguish source evidence, ingestion provenance, and lineage. |
| distributed execution model | CLI/plugins/destinations. | Query engine/plugins. | Clustered NiFi, cluster state. | Peer forwarding/source coordination. | Kafka Connect distributed workers. | `RunLockSet`, stage state, partition/lease records. |
| Cadastre equivalent | Adapter stage implementation. | Non-authoritative live probe. | Optional flow runtime. | Optional pipeline runtime. | CDC adapter profile. | Every accepted implementation must pass replay and forbidden-output criteria. |
| required acceptance criteria | State refs deterministic; stale delete rejected as absence. | Live query cannot emit production raw/completeness. | Provenance cannot satisfy evidence by itself. | Ack cannot satisfy completeness by itself. | Offsets/schema history required for CDC replay. | Two implementations produce identical outputs for same persisted inputs/manifest. |

### Schema and mapping matrix

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre mapping rule |
| --- | --- | --- | --- | --- | --- | --- |
| schema declaration mechanism | Source tables/columns/PKs. | SQL tables/columns/types. | Processor-specific attributes/content schemas. | Event/plugin schemas and pipeline config. | CDC event envelope plus source schema history. | `SourceSchemaImportProfile`, mapping bundle, external schema artifact. |
| source table/event/object model | Tables and rows. | Live API rows. | FlowFiles. | Events. | Change events and schema-change events. | Raw event subtypes and source dataset contract. |
| field metadata | Column metadata/system columns. | Column descriptions, transform metadata. | FlowFile attributes. | Event fields and expressions. | `source` block, op, timestamps, before/after. | `field_quality` and source-state mapping. |
| primary key or identity model | Table PK/CQ ID. | Key columns/qualifiers. | FlowFile UUID/internal refs. | Event identity source-specific. | Key schema, DB primary key, log position. | Source-native ID only; no canonical identity by itself. |
| raw field preservation | Destination rows may not preserve full source payload. | Raw JSON can be exposed as column if plugin chooses. | Content repository preserves content while retained. | Event payload may be mutated by processors. | Envelope preserves before/after/source; raw DB row not necessarily original bytes. | `RawRecord.payload` and exact `payload_sha256` required. |
| unknown field behavior | Plugin/table-specific. | Plugin-specific JSON/columns. | Attributes/content processor-specific. | Event processors may drop/route. | Schema history governs. | Unknown field handling must be mapping-defined. |
| enum behavior | Table-specific. | Table/plugin-specific. | Attribute/content-specific. | Event-specific. | Source DB values plus schema. | Unknown enum raw value preserved as `unknown`. |
| schema evolution | Plugin/schema versions. | Plugin versions/table changes. | Flow/processor version changes. | Plugin config/event schema changes. | Schema history topics and DDL changes. | Schema artifact/profile/version manifest refs. |
| validation mechanism | Plugin tests/config validation. | Plugin checks/query examples. | Processor validation/tests. | Plugin schema and e2e tests. | Connector tests/schema history. | `ValidationMatrix`, golden corpus, replay, negative tests. |
| mapping to Cadastre raw/silver/contracts | Source rows become raw records only after payload capture. | Live rows are probe output unless captured. | FlowFiles can become raw if payload persisted. | Events can become raw at sink boundary. | CDC events can become raw event subtypes. | Parser/normalizer produce silver only after raw stage. |
| do-not-transfer risk | PK/destination ID as identity. | SQL row as evidence. | FlowFile attribute as fact. | Processor route as authority. | Offset/time as valid/known time. | PRD boundary protects identity/gold/projection. |

### Operational lifecycle matrix

| Row | CloudQuery | Steampipe | NiFi | Data Prepper | Debezium | Cadastre implications |
| --- | --- | --- | --- | --- | --- | --- |
| package or plugin acquisition | CloudQuery plugins/SDK/scaffolding. | Plugin install/manage. | NAR bundles/extensions. | Plugin modules/distribution. | Connector artifacts/Kafka Connect plugins. | `PackageArtifact` with materialization separate from activation. |
| versioning | Plugin versions/source tables/dest modes. | Plugin and CLI versions. | NiFi version, flow version, extension versions. | Data Prepper version, plugin versions, pipeline config. | Connector version, database version, schema history. | Every output-affecting version in `VersionManifest`. |
| trust or verification | Not fully inspected. | Plugin ecosystem trust not deeply inspected. | Extension signing/security not deeply inspected. | Plugin trust not deeply inspected. | Connector artifacts and Kafka Connect deployment trust not deeply inspected. | Require checksum/signature/dependency review before production activation. |
| dependency locking | Plugin and module dependencies. | Plugin dependencies. | Extension bundles and runtime libs. | Plugin/runtime dependencies. | Connector/runtime dependencies. | Lock and digest all production packages. |
| config validation | Plugin/destination validation. | Connection/plugin checks. | Processor/flow validation. | Pipeline/plugin schema validation. | Connector config validation. | `SourceInstanceConfigSchema` and connectivity artifact. |
| local development workflow | SDK and scaffolding. | Plugin authoring and SQL examples. | UI/processor test/dev guides. | Pipeline YAML/examples/e2e tests. | Connector integration tests. | Developer tooling allowed only as non-authoritative unless validated. |
| CI/test model | Plugin tests not fully inspected. | Plugin release checklist. | Processor/tests and release process. | e2e tests and plugin tests. | Connector compatibility tests. | `ValidationMatrix` rows per pattern. |
| release model | CloudQuery releases/plugins. | CLI/plugins/mods. | Apache releases. | OpenSearch project releases. | Debezium stable releases. | Recheck current versions before adoption. |
| runtime observability | Logs/metrics/traces/summaries. | `_ctx`, SQL output, diagnostics. | Provenance, queues, UI, REST API. | Metrics/admin API, acks, circuit breakers. | Connector metrics, heartbeats, offsets. | Stable `DiagnosticRecord` and health dimensions. |
| failure recovery | State resume, destination modes. | Query rerun. | Replay/provenance/state recovery. | Ack retry/source coordination. | Offset/schema history restart. | Recovery must not bypass manifest/completeness gates. |
| Cadastre `PackageArtifact`, `PackageStageBinding`, `ValidationMatrix`, and `VersionManifest` implications | Bind table/resolver packages; reject destination authority. | Bind live probe packages as non-production by default. | Bind flow/runtime artifacts and extension versions. | Bind pipeline YAML/plugin versions/ack policies. | Bind connector config/offset/schema-history refs. | Active lifecycle, package binding, validation, manifest required before production writes. |

## Cadastre transfer analysis

### High-value findings table

| Finding ID | Source | Evidence class | Confirmed or inferred | Finding | Cadastre PRD area | Transfer stance | Recommended PRD improvement | New or revised contract fields | Default behavior | Error behavior | Acceptance criteria | Risks | Open questions |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RES006-F01 | CloudQuery | official_docs | confirmed + inferred | Incremental table sync uses state and can produce duplicates while aiming to avoid gaps. | `CollectionStepPattern`, `SourceCallPolicy`, `StageStateRecord`, `RawRecord` | `adopt` | Add `incremental_table_sync` and `poll_with_cursor` patterns. Cursor state must commit only after raw persistence and completeness receipt persistence. | `cursor_state_ref`, `cursor_start`, `cursor_end_candidate`, `cursor_commit_status`, `duplicate_cursor_behavior`, `gap_prevention_rule`. | Duplicate cursor behavior defaults to `record_duplicate_raw_status`; cursor commit defaults to `after_raw_and_receipt_commit`. | `CURSOR_STATE_MISSING`, `CURSOR_GAP_DETECTED`, `CURSOR_DUPLICATE_UNDECLARED`. | Fixture replay with duplicate page produces duplicate raw status and no watermark advancement unless profile permits. | Cursor semantics vary by source. | Which source adapters require source-owned cursor vs Cadastre-owned cursor? |
| RES006-F02 | CloudQuery | official_docs | confirmed | Destination `overwrite-delete-stale` can delete destination rows not observed in a sync. | `SourceCompletenessReceipt`, `SourceCompletenessProfile`, `GraphProjectionProfile` | `reject` | Add explicit do-not-transfer rule: destination stale deletion is never absence, cleanup, retraction, or watermark proof. | `destination_cleanup_observed` diagnostic field only. | Destination cleanup ignored for completeness. | `DESTINATION_CLEANUP_NOT_COMPLETENESS` when cleanup evidence is submitted as absence proof. | Validation rejects any mapping from destination stale delete to absence. | Implementers may want convenient cleanup semantics. | None. |
| RES006-F03 | CloudQuery | official_docs | inferred | Source table/resolver layout improves adapter package authoring. | `SourcePackageDeveloperContract` | `adapt` | Define package-internal components: source client, table/dataset resolver, parser, mapper, state provider, validation provider. | `component_kind`, `allowed_outputs`, `state_access_mode`, `source_dataset_ref`. | Components are output-forbidden unless stage binding permits. | `FORBIDDEN_PACKAGE_COMPONENT_OUTPUT`. | Unit fixture proves source client cannot emit production records. | Over-specification of internal mechanism. | Should this be mandatory or recommended? |
| RES006-F04 | Steampipe | official_docs | confirmed | Live API tables are useful for source exploration but are not durable evidence. | `SourceStageDAG`, `PackageStageBinding`, `RecordedSourceFixture` | `adapt` | Add `live_source_probe` mode with non-authoritative output class and no production writes. | `probe_mode`, `probe_result_authority_class`, `capture_fixture_allowed`, `probe_timeout_seconds`. | `production_writes_allowed = false`; default timeout 30 seconds, max 300 seconds. | `LIVE_PROBE_NOT_PRODUCTION_EVIDENCE`. | Live probe returning zero rows cannot emit completeness receipt or gold facts. | Users may confuse SQL results with inventory truth. | Should probe results be stored in health table or separate tooling table? |
| RES006-F05 | Steampipe | official_docs | confirmed | Predicate/projection pushdown minimizes API calls and exposes useful diagnostics. | `SourceCallPolicy`, `DiagnosticRecord` | `adapt` | Add optional optimization metadata that cannot change semantics. | `source_filter_pushdown_rule`, `source_projection_pushdown_rule`, `optimization_semantics = equivalent_or_diagnostic_only`, `rate_limit_scope_key`. | Optimization disabled unless declared. | `PUSHDOWN_SEMANTICS_UNDECLARED`, `RATE_LIMIT_SCOPE_ERROR`. | Full fetch plus post-filter and pushed fetch must match in fixture or be diagnostic-only. | Source APIs may make equivalence hard to prove. | Which source categories need predicate pushdown in MVP? |
| RES006-F06 | NiFi | official_docs | confirmed | FlowFile provenance gives detailed ingestion lineage and replay signals. | `DiagnosticRecord`, `RunDatasetIOContract`, `RawRecord`, `EvidenceRef` | `adapt` | Add an ingestion-provenance event taxonomy distinct from source evidence. | `ingestion_provenance_event_id`, `event_type`, `flowfile_ref`, `parent_refs`, `content_claim_ref`, `route`, `processor_id`, `authority_class`. | Authority class defaults to `lineage_only`. | `PROVENANCE_NOT_SOURCE_EVIDENCE` when used as raw/gold evidence. | Provenance replay fixture cannot satisfy evidence drillback without `RawRecord`. | Provenance retention may expire content. | Should provenance events be queryable by users or only operators? |
| RES006-F07 | NiFi | official_docs | confirmed | Queue backpressure, penalty, yield, expiration, and failure routes are useful operational controls. | `SourceStageLifecycleResult`, `DiagnosticRecord`, `SourceCallPolicy` | `adapt` | Add diagnostics for backpressure and internal flow loss. | `runtime_backpressure_state`, `queue_object_count`, `queue_byte_count`, `flowfile_expiration_count`, `yield_duration_seconds`. | Backpressure affects health/lifecycle only. | `BACKPRESSURE_LIMIT_EXCEEDED`, `FLOWFILE_EXPIRED_BEFORE_RAW_PERSIST`. | Expired FlowFile blocks completeness and emits diagnostic. | Runtime details may be implementation-specific. | Whether to specify runtime-independent names only. |
| RES006-F08 | Data Prepper | official_docs | confirmed | End-to-end ack is a useful delivery guarantee but not source completeness. | `SourceStageLifecycleResult`, `SourceCompletenessReceipt`, `StageStateRecord` | `adapt` | Add ack fields and receipt evidence class with explicit non-authority rule. | `ack_required`, `ack_timeout_seconds`, `ack_state_ref`, `ack_status`, `ack_evidence_class`. | `ack_required = false`; ack evidence cannot authorize absence. | `ACK_TIMEOUT`, `ACK_NOT_COMPLETENESS`. | Ack timeout fixture blocks watermark and completeness. | Acks can be over-trusted. | Which source protocols require ack before deleting upstream messages? |
| RES006-F09 | Data Prepper | official_docs | confirmed | Source coordination leases and stateful aggregation can affect output in distributed runs. | `StageStateRecord`, `VersionManifest`, `RunLockSet` | `adopt` | Add state kinds for `source_coordination_lease`, `processor_state`, and `ack_batch`. | `lease_owner`, `lease_state`, `lease_scope`, `lease_expires_at`, `processor_state_checksum`. | Missing lease state rejects production replay. | `SOURCE_COORDINATION_STATE_MISSING`, `PROCESSOR_STATE_MISMATCH`. | Two workers cannot emit same source partition without duplicate status or lock conflict. | Distributed state can be high-cardinality. | Should lease states be summarized or fully materialized? |
| RES006-F10 | Debezium | official_docs | confirmed | Durable database change logs require snapshot and stream patterns. | `CollectionStepPattern`, `SourceCallPolicy`, `StageStateRecord`, `RawRecord` | `adopt` | Add `cdc_initial_snapshot`, `cdc_stream`, `snapshot_then_stream`, and `database_change_log`. | `cdc_connector_kind`, `snapshot_mode`, `snapshot_boundary_ref`, `stream_offset_ref`, `log_position_kind`, `source_transaction_ref`. | CDC disabled unless package declares durable-log support. | `CDC_PATTERN_UNSUPPORTED`, `CDC_LOG_POSITION_UNAVAILABLE`. | Snapshot-then-stream fixture proves no gap between snapshot boundary and first stream offset. | Database-specific offsets differ. | Which databases are MVP-supported? |
| RES006-F11 | Debezium | official_docs | confirmed | Schema history is required for CDC replay compatibility. | `StageStateRecord`, `VersionManifest`, `ValidationMatrix` | `adopt` | Add `schema_history_ref` state kind and diagnostics. | `schema_history_artifact_ref`, `schema_history_checksum`, `ddl_event_ref`, `schema_history_status`. | Required for CDC streams; missing ref rejects parsing/replay. | `CDC_SCHEMA_HISTORY_MISSING`, `CDC_SCHEMA_CHANGE_UNPARSEABLE`. | Replay with missing schema history fails before output. | Schema histories may include secrets or DDL comments. | Redaction policy for DDL comments. |
| RES006-F12 | Debezium | official_docs | confirmed | CDC envelope has create/update/delete/tombstone/schema/heartbeat/transaction metadata event types. | `RawRecord`, `RecordedSourceFixture`, `DiagnosticRecord` | `adapt` | Add `source_event_subtype` or CDC extension on raw record. | `source_event_subtype`, `cdc_operation`, `cdc_source_position`, `cdc_transaction_id`, `cdc_schema_version_ref`. | `source_event_subtype = source_record` for non-CDC; CDC must specify subtype. | `CDC_EVENT_SUBTYPE_MISSING`. | Fixtures cover create/update/delete/tombstone/heartbeat/schema-change. | Adding field to `RawRecord` may affect existing model. | Better as extension object or core field? |
| RES006-F13 | All | official_docs + inference | inferred | Source-call diagnostics need stable cross-system vocabulary. | `DiagnosticRecord`, `SourceCallPolicy`, `ValidationMatrix` | `adopt` | Add diagnostic codes for state loss, partial pages, duplicate pages, ack timeout, schema history loss, live probe non-replayability, provenance gaps, rate limit scope, and permission loss. | `diagnostic_code`, `source_call_attempt_id`, `state_ref`, `scope_key`, `retry_count`, `terminal_behavior`. | Diagnostics are required for every terminal source-call failure. | Specific code must be used when available; generic failure rejected in validation. | Negative fixtures assert exact diagnostic code. | Too many codes can drift. | Should diagnostic registry be global or package-local with mapping? |
| RES006-F14 | All | Cadastre applicability inference | inferred | Production package activation needs pattern-specific validation matrix rows. | `ValidationMatrix`, `PackageStageBinding`, `RecordedSourceFixture` | `adopt` | Add validation classes for paginated API, live query, flowfile, pipeline batch, CDC snapshot, CDC stream, file report set. | `fixture_class`, `required_case_rows`, `forbidden_output_cases`, `replay_determinism_required`. | Package cannot become active until all declared pattern cases pass. | `VALIDATION_MATRIX_COVERAGE_ERROR`. | Active package with `cdc_stream` must have offset-loss and schema-history-loss tests. | Validation scope expands. | Which fixture classes are mandatory for MVP packages? |
| RES006-F15 | All | Cadastre baseline | inferred | Adapter collection must remain raw-evidence-first. | `SourceStageDAG`, `PackageStageBinding`, `GraphProjectionProfile` | `cautionary_reference` | Re-state forbidden output matrix for new patterns and implementation runtimes. | `adapter_output_class`, `forbidden_output_violation_code`. | Forbidden outputs rejected before persistence. | `FORBIDDEN_STAGE_OUTPUT`. | Any adapter stage attempting silver/gold/graph output fails validation and runtime. | Teams may import graph-first connector patterns. | None. |

### Do-not-transfer table

| Source | Pattern that must not transfer | Why it is unsafe for Cadastre | PRD contract protected | Required Cadastre-safe alternative |
| --- | --- | --- | --- | --- |
| CloudQuery | Destination write modes or stale-delete behavior authorizing absence, cleanup, retraction, or watermark advancement. | Destination state is not source scope completeness and can be shaped by partial sync, destination mode, schema, or previous destination contents. | `SourceCompletenessProfile`, `SourceCompletenessReceipt`, `CoverageAssertion`, `GraphProjectionProfile`. | Evaluate completeness through source evidence, active profile, receipt, and coverage before cleanup/absence/watermark. |
| Steampipe | Live query results becoming production evidence, completeness proof, replayable raw records, gold facts, or graph evidence. | Live query rows are not durable payload-hashed source evidence and depend on SQL, projection, predicates, permissions, time, and plugin behavior. | `RawRecord`, `SourceCompletenessReceipt`, `GoldFact`, `GraphProjectionProfile`. | Use `live_source_probe` as non-authoritative tooling or capture through `RecordedSourceFixture`/production adapter collection. |
| NiFi | Provenance and replay replacing Cadastre raw evidence, table snapshots, bitemporal facts, or graph projection lineage. | Provenance records flow activity; replay depends on repository retention/access and does not prove source authority. | `RawRecord`, `EvidenceRef`, `VersionManifest`, `GoldFact`, `GraphDeltaSet`. | Store NiFi-style provenance as ingestion-provenance diagnostics; derive facts from Cadastre raw/silver/gold. |
| Data Prepper | End-to-end acknowledgment treated as source completeness. | Ack proves delivery to a final sink or processing path, not that source scope was enumerated or absence is meaningful. | `SourceCompletenessReceipt`, `SourceCompletenessProfile`, `CoverageAssertion`. | Model ack as delivery evidence and require independent source completeness proof. |
| Debezium | CDC offsets, snapshot times, transaction-log positions, connector event times, or heartbeats becoming valid time, known time, source observation time, or completeness by themselves. | These are connector/log/runtime states; they do not encode domain validity or source authority for Cadastre facts. | `GoldFact`, `RawRecord.source_event_time_quality`, `SourceCompletenessReceipt`, `CoverageAssertion`. | Preserve as raw/StageState metadata; derive temporal facts through Cadastre rules and source authority. |
| All | Direct-to-graph or direct-to-destination adapter architecture bypassing raw, silver, identity, gold, graph-delta, and graph-apply stages. | It makes graph/destination the de facto truth source and breaks replay/evidence drillback. | Entire Cadastre lakehouse-authoritative boundary. | Adapter emits raw records, completeness receipts, and errors only; graph deltas derive from gold facts. |
| CloudQuery + Data Prepper | Transformer/processor mutation before Cadastre raw persistence. | It can erase source-native payload and obscure field quality/omission evidence. | `RawRecord`, `CadastreSilverObservation`, `VersionManifest`. | Persist exact source payload before transformation; run parser/normalizer as governed stages. |
| Steampipe + Debezium | Source-native keys, SQL key columns, DB primary keys, or graph keys becoming canonical identity. | Source identifiers are scope-local and may be weak or mutable. | `IdentityDecision`, `TargetSelectorSafetyPolicy`. | Treat as source-scoped identifiers and require resolver decisions with evidence. |

## PRD improvement map

| PRD area | Required evaluation | Recommendation | Observable contract additions |
| --- | --- | --- | --- |
| `CollectionStepPattern` | Evaluate patterns such as `poll_with_cursor`, `incremental_table_sync`, `cdc_initial_snapshot`, `cdc_stream`, `snapshot_then_stream`, `live_source_probe`, `pipeline_acknowledged_batch`, and `file_report_set`. | Add all except `live_source_probe` as production-capable only under declared package/stage binding; add `live_source_probe` as non-authoritative by default. | Closed enum values, required fields, output permissions, state refs, completeness eligibility table, forbidden outputs. |
| `SourceCallPolicy` | Evaluate ack timeout, cursor state, source rate-limit scopes, retry-after, duplicate cursor, stream resume, schema-history dependency, source-visible completeness proofs. | Refine with source-call state and CDC/ack fields. | `retry_after_header_rule`, `rate_limit_scope_key`, `cursor_commit_rule`, `duplicate_cursor_behavior`, `stream_resume_rule`, `ack_timeout_seconds`, `schema_history_required`, `source_completeness_evidence_rule`. |
| `StageStateRecord` | Model page tokens, cursors, CDC offsets, schema history refs, snapshot IDs, checkpoint IDs, queue offsets, and processor state. | Extend `state_kind` table. | `page_token`, `api_cursor`, `sync_cursor`, `cdc_offset`, `cdc_snapshot_state`, `schema_history_ref`, `source_coordination_lease`, `ack_batch`, `processor_state`, `queue_offset`. |
| `SourceCompletenessReceipt` | Add evidence classes for cursor exhaustion, expected-count match, stream heartbeat, schema-history availability, end-to-end ack, provenance event closure, table commit success, permission-scope proof. | Add `completeness_evidence_rows` with closed evidence classes. | `evidence_class`, `evidence_ref`, `authority_limit`, `permits_absence_candidate`, `requires_profile_permission`, `source_scope_visibility_ref`. |
| `SourcePackageDeveloperContract` | Further separate source clients, table resolvers, hydrate functions, processors, parsers, mappers, and state providers. | Add component role table and forbidden-output rules. | `component_kind`, `allowed_stage_types`, `allowed_outputs`, `state_access`, `source_call_access`, `credential_access`, `validation_only`. |
| `RecordedSourceFixture` | Fixture classes should differ for paginated API, live query, flowfile, pipeline batch, CDC snapshot, CDC stream, and file report sources. | Add class-specific required cases and redaction rules. | `fixture_class = paginated_api \| cursor_api \| live_query \| flowfile_batch \| pipeline_ack_batch \| cdc_snapshot \| cdc_stream \| cdc_schema_history \| report_file_set`. |
| `ValidationMatrix` | Add positive, negative, replay, partial, duplicate, rate-limit, timeout, cursor-loss, schema-history-loss, permission-loss tests. | Add pattern-to-test coverage table. | For each declared pattern, list required test IDs and forbidden-output tests; activation fails on missing row. |
| `VersionManifest` | Include source-call policies, state records, offsets, cursor refs, schema-history refs, source fixtures, pipeline acknowledgments when output-affecting. | Extend manifest refs for new state and policy types. | `source_call_state_ref_ids`, `cdc_offset_state_ids`, `schema_history_ref_ids`, `ack_state_ref_ids`, `source_coordination_state_ids`, `probe_capture_fixture_ids`. |
| `RawRecord` | Evaluate explicit source-event subtypes for API rows, live query rows, FlowFiles, pipeline events, CDC change events, schema changes, tombstones, and heartbeats. | Add `source_event_subtype` or nested `source_event_metadata` extension. | Closed subtype enum: `api_row`, `api_page`, `report_file_record`, `flowfile_content`, `pipeline_event`, `cdc_change`, `cdc_schema_change`, `cdc_tombstone`, `cdc_heartbeat`, `live_probe_capture`. Default `api_row` only when source category declares API row semantics. |
| `DiagnosticRecord` | Stable diagnostics for source-call failures, state loss, duplicate pages, partial pages, schema-history loss, unparseable DDL, live-query non-replayability, provenance gaps, ack timeout. | Add source adapter diagnostic registry. | Codes: `SOURCE_CALL_TIMEOUT`, `RATE_LIMITED`, `DUPLICATE_PAGE`, `PARTIAL_PAGE`, `CURSOR_STATE_MISSING`, `CDC_SCHEMA_HISTORY_MISSING`, `CDC_OFFSET_LOST`, `LIVE_PROBE_NOT_PRODUCTION_EVIDENCE`, `PROVENANCE_GAP`, `ACK_TIMEOUT`, `PERMISSION_SCOPE_LOSS`. |

### Proposed closed `CollectionStepPattern` additions

| New pattern | Permitted stage type | Required state | Required completeness evidence candidate | Production default | Forbidden behavior |
| --- | --- | --- | --- | --- | --- |
| `poll_with_cursor` | `adapter_collection` | API cursor state. | Cursor exhaustion or source-declared complete. | Enabled only when source call policy declares cursor commit. | Must not advance cursor after partial page. |
| `incremental_table_sync` | `adapter_collection` | Table cursor and sync scope. | Cursor progression and duplicate policy. | Duplicates recorded unless declared dedupe. | Must not infer absence from missing current rows. |
| `snapshot_then_incremental` | `adapter_collection` | Snapshot ID plus incremental cursor. | Snapshot complete and incremental continuity. | Requires continuity check. | Must not emit completeness if gap between snapshot and cursor exists. |
| `cdc_initial_snapshot` | `adapter_collection` | Snapshot state and chunk state. | Snapshot completion, expected chunks or source-declared complete. | Disabled unless durable log source. | Must not use snapshot time as valid time. |
| `cdc_stream` | `adapter_collection` | Offset and schema history refs. | Log continuity, schema history availability, optional heartbeat. | Requires schema history. | Must not emit absence from heartbeat only. |
| `snapshot_then_stream` | `adapter_collection` | Snapshot boundary plus stream offset. | Snapshot-stream continuity proof. | Requires boundary proof. | Must not proceed with unresolved boundary. |
| `pipeline_acknowledged_batch` | `adapter_collection` | Ack batch state. | Ack success as delivery evidence only. | Ack not required unless declared. | Ack must not authorize completeness. |
| `flowfile_batch` | `adapter_collection` | FlowFile/provenance refs when output-affecting. | Provenance closure as operational evidence only. | Non-authoritative provenance by default. | FlowFile replay must not bypass raw persistence. |
| `live_source_probe` | `source_exploration` or validation-only stage | None unless fixture captured. | None. | Production writes forbidden. | Probe result must not emit raw/completeness/gold/graph. |
| `file_report_set` | `adapter_collection` | Report manifest/checkpoint. | Report file set complete, expected count, checksum. | Requires report manifest. | Missing file blocks absence and cleanup. |

### Deterministic source completeness decision algorithm additions

The current `EvaluateSourceCompleteness(receipt, profile)` should retain authority. Add the following pre-checks before any profile-specific permission is evaluated:

```text
1. If receipt.completeness_state is not complete or empty_complete, set all absence, retraction, cleanup, and source watermark booleans to false.
2. If any required state ref for the declared CollectionStepPattern is missing, set completeness_state = partial_unknown_gap and emit STATE_RECORD_MISSING.
3. If any source-call policy marks a page, cursor, stream, lease, ack, or schema-history dependency as failed, set completeness_state to the most specific unsafe state.
4. If an evidence row has authority_limit = delivery_only, lineage_only, diagnostic_only, or probe_only, it must not contribute to absence, retraction, cleanup, or source watermark permission.
5. If all source-scope and source-visibility evidence rows required by the profile are present and the profile permits the fact/dataset/scope, compute the booleans according to the active profile.
6. Emit a deterministic decision record containing the evaluated profile version, receipt ID, evidence rows used, evidence rows ignored, decision booleans, and diagnostic refs.
```

Acceptance criteria:

1. Given the same receipt, profile, state refs, and evidence rows, two implementations must compute byte-identical decision records.
2. A delivery-only ack, provenance closure, live query result, or destination cleanup signal must never set `absence_semantics_valid = true` by itself.
3. Missing schema history in a CDC stream must force `watermark_advance_allowed = false`.
4. A cursor commit attempted before raw persistence and receipt persistence must fail with `CURSOR_COMMIT_ORDER_ERROR`.

## Concepts that should remain non-authoritative tooling

| Concept | Tooling value | Required boundary |
| --- | --- | --- |
| Steampipe-like source exploration | Lets mapping authors inspect live source APIs, predicates, and raw-ish JSON columns. | Must run as `live_source_probe`, emit non-authoritative probe records, and never emit production raw/completeness/gold/graph output. |
| CloudQuery-like table scaffolding | Helps package authors structure source datasets, parent/child relationships, and resolvers. | Must compile into Cadastre package metadata and stage steps; table rows are not canonical facts. |
| NiFi-like visual flow design | Useful for prototyping ingestion flow, queue behavior, and routing. | Production flow must be versioned and checksummed; UI state must not be production contract. |
| Data Prepper-like pipeline YAML | Useful implementation mechanism for source/buffer/process/sink runtime. | Cadastre `SourceStageDAG` remains the external contract; pipeline defaults must not leak into output semantics. |
| Debezium connector runtime | Useful for database CDC adapters. | CDC applies only to durable-log sources with schema history; offsets remain state, not Cadastre time or truth. |
| Query packs, controls, and live checks | Useful for authoring validation scenarios and source-inspection examples. | Must not satisfy coverage, completeness, evidence, identity, or gold fact requirements. |

## Gaps, risks, stale assumptions, contradictions, and unresolved questions

### Cross-source gaps

| Gap ID | Gap | Impact | Required next step |
| --- | --- | --- | --- |
| GAP-001 | Exact source code for plugin protocols, ack internals, processor internals, and connector tests was not fully inspected. | Implementation-level transfer may miss edge cases. | Before PRD amendment, inspect exact code paths or constrain recommendation to PRD-level observable contracts. |
| GAP-002 | Package acquisition, signature verification, and plugin trust mechanisms were not deeply compared. | `PackageArtifact` security requirements may remain under-specified. | Perform package supply-chain follow-up report. |
| GAP-003 | Current release versions and defaults may drift after inspection. | Freshness-sensitive facts may become stale. | Recheck official releases and docs before production implementation. |
| GAP-004 | Database-specific CDC offset shapes differ across connectors. | A single generic offset schema may be too vague. | Add source-family-specific offset mapping tables when MVP source DBs are chosen. |
| GAP-005 | Source completeness for SaaS APIs cannot be generalized from CDC or destination stale deletion. | Unsafe absence facts or cleanup could be emitted. | Require per-source `SourceCompletenessProfile` rows and fixtures. |

### Contradictions found

No direct contradiction was established among the inspected sources and the Cadastre PRD. The conflict is architectural rather than textual: several external systems permit direct destination writes, live query results, or graph-like outputs, while Cadastre forbids those as authoritative production outputs unless a Cadastre contract captures and validates them.

### Unresolved questions requiring domain authority

| Question ID | Question | Why it requires authority |
| --- | --- | --- |
| OQ-001 | Which Cadastre MVP sources are database-backed and have durable CDC logs? | CDC patterns should be active only for those sources. |
| OQ-002 | Should `RawRecord.source_event_subtype` become a core field or an extension object? | This changes core raw schema compatibility. |
| OQ-003 | Should live source probes be stored at all, or only emitted to local tooling? | Storage affects privacy, access control, and evidence confusion risk. |
| OQ-004 | Which source-call optimizer rules are acceptable without full-fetch equivalence proof? | Predicate/projection pushdown can change source results if APIs behave unexpectedly. |
| OQ-005 | Should a NiFi/Data Prepper runtime be supported as an implementation profile? | Requires operational, security, and replay review. |

## Report acceptance criteria and Definition of Done

| ID | Acceptance criterion | Status in this report |
| --- | --- | --- |
| AC-001 | Each primary repository is analyzed independently before any comparison section. | Satisfied. |
| AC-002 | Each primary source section records source inventory, freshness, and inspection limits. | Satisfied, with explicit no-build/no-test caveat. |
| AC-003 | Non-trivial factual claims are cited to source docs, repository evidence, release metadata, or Cadastre PRD. | Satisfied at report level through footnote citations; implementation must recheck exact source lines before PRD patching. |
| AC-004 | Confirmed findings are separated from source-grounded inferences. | Satisfied in every source section. |
| AC-005 | The report does not claim local builds, tests, services, examples, or clones were run. | Satisfied. |
| AC-006 | CloudQuery receives full-depth treatment of adapter contracts, table modeling, incremental sync, state/cursor behavior, destination modes, and runtime boundaries. | Satisfied. |
| AC-007 | Steampipe receives targeted treatment of live API tables, plugin ergonomics, source-query efficiency, rate limiting, and non-persistence limits. | Satisfied. |
| AC-008 | NiFi receives targeted treatment of FlowFiles, processors, queues, provenance, replay, lineage, backpressure, and operator configuration. | Satisfied. |
| AC-009 | Data Prepper receives targeted treatment of pipeline stages, buffering, processors, routing, acknowledgments, stateful aggregation, and plugin boundaries. | Satisfied. |
| AC-010 | Debezium receives targeted treatment of CDC snapshots, streams, offsets, schema history, heartbeats, watermarking, event envelopes, and failure modes. | Satisfied. |
| AC-011 | Comparison matrices cover every source in declared scope. | Satisfied. |
| AC-012 | Transfer analysis maps high-value findings to PRD areas or rejects them. | Satisfied. |
| AC-013 | Do-not-transfer table has at least one row per primary source and protects lakehouse-authoritative boundary. | Satisfied. |
| AC-014 | Adopted/adapted recommendations include observable contract changes. | Satisfied. |
| AC-015 | Adopted/adapted recommendations include binary acceptance criteria. | Satisfied in transfer table and algorithm section. |
| AC-016 | Gaps, stale assumptions, and unresolved questions are identified without guessing. | Satisfied. |
| AC-017 | NLSpec rigor is preserved through explicit fields, defaults, errors, deterministic algorithms, and acceptance criteria. | Satisfied at research-report level. |
| AC-018 | The report is self-contained for first-pass implementer review. | Satisfied. |

## Source discipline ledger

| Source name | URL or path | Evidence class | Commit, tag, branch, or version | Inspection date | Scope inspected | Scope not inspected | Reliability |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cadastre PRD | `/mnt/data/PRD-Cadastre.md` | `Cadastre_baseline` | Draft in project context | 2026-05-16 | Product summary, document status, source contracts, raw records, completeness, stage DAG, state, step patterns, call policy | Full PRD not line-by-line revalidated in this report | High for Cadastre baseline |
| NLSpec grounding | `/mnt/data/nlspec-spec.md` | `Cadastre_baseline` | Version 0.2.2 | 2026-05-16 | Completeness, defaults, mapping tables, acceptance criteria | Full doc not cited line-by-line | High for report quality standard |
| CloudQuery repository/docs | Public GitHub repository and docs | `source_code`, `official_docs`, `example` | Branch `main` from repository page; exact commit not pinned | 2026-05-16 | README/layout, architecture, Go source plugin docs, incremental docs, destination modes | Full source, tests, releases, local syncs | High for documented behavior; medium for runtime internals |
| Steampipe repository/docs | Public GitHub repository and docs | `source_code`, `official_docs`, `example` | Branch `develop` from repository page; exact commit not pinned | 2026-05-16 | README/layout, architecture, plugin authoring, rate limiting, CLI query docs | Full FDW source, tests, plugin internals, local queries | High for documented plugin model; medium for runtime internals |
| Apache NiFi repository/docs | Public GitHub repository and docs | `source_code`, `official_docs`, `release_metadata`, `example` | Branch `main`; release metadata included NiFi 2.9.0 | 2026-05-16 | README, release page, overview, in-depth docs, user/developer/admin docs, REST docs | Full Java source, local runtime, tests | High for official docs; medium for internals |
| Data Prepper repository/docs | Public GitHub repository and docs | `source_code`, `official_docs`, `release_metadata`, `example` | Branch `main`; release metadata included 2.15.0 | 2026-05-16 | README, key concepts, pipeline config, ack docs, source coordination, peer forwarding, expressions | Full Java source, plugin tests, local pipelines | High for official docs; medium for internals |
| Debezium repository/docs | Public GitHub repository and docs | `source_code`, `official_docs`, `release_metadata`, `example` | Current docs exposed 3.5 stable / 3.6 development; exact commit not pinned | 2026-05-16 | README, architecture, MySQL connector docs, snapshots, CDC envelope, transaction metadata, heartbeat/error defaults, incremental snapshots | Full connector source, local Kafka Connect, tests, every DB connector | High for official docs; medium for connector-specific implementation details |

## Sources

[^1]: `/mnt/data/PRD-Cadastre.md`, Product Summary, lines 15-43. The PRD states Cadastre's temporal lakehouse, raw/silver/gold layers, deterministic graph deltas, replaceable graph/CIM projections, and table-format-native lakehouse reference requirement.
[^2]: `/mnt/data/PRD-Cadastre.md`, Document Status, lines 47-59. The PRD states NLSpec governance, non-authoritative source-schema tooling, rejection of direct-to-graph or graph-authoritative sync, and limits on external lakehouse/lineage/workflow artifacts.
[^3]: `/mnt/data/PRD-Cadastre.md`, normative contracts table, lines 61-80 and adjacent rows. Evidence used for baseline source-instance, DAG, package, state, source-call, completeness, and coverage contracts.
[^4]: `/mnt/data/PRD-Cadastre.md`, `RawRecord`, lines 1273-1306. Evidence used for raw source-native payload preservation and source event time quality.
[^5]: `/mnt/data/PRD-Cadastre.md`, `SourceStageDAG` stage output permissions, lines 2715-2798. Evidence used for adapter-stage permitted and forbidden outputs.
[^6]: `/mnt/data/PRD-Cadastre.md`, `SourceCompletenessReceipt`, lines 2611-2662 and 2664-2673. Evidence used for receipt fields, completeness states, visibility states, and non-self-authorizing completeness behavior.
[^7]: `/mnt/data/PRD-Cadastre.md`, `StageStateRecord`, lines 4186-4228. Evidence used for output-affecting state being explicit, canonicalized, hashable, replayable, and manifest-included.
[^8]: `/mnt/data/PRD-Cadastre.md`, `SourceStageStep and CollectionStepPattern`, lines 4497-4536. Evidence used for closed collection-step patterns and source-call policy linkage.
[^9]: `/mnt/data/PRD-Cadastre.md`, `SourceCallPolicy`, lines 4624-4666. Evidence used for pagination, retry, rate-limit, timeout, duplicate-page, partial-page, credential-error, and visibility-loss defaults and bounds.
[^10]: `https://github.com/cloudquery/cloudquery`, repository page and README inspected 2026-05-16. Evidence used for repository identity, branch page, module layout, and CloudQuery product purpose.
[^11]: CloudQuery official docs, Go source plugin creation and project layout pages, inspected 2026-05-16. Evidence used for source plugin layout, `Configure`, client/spec/table/resolver structure, testing resources, and incremental table SDK examples.
[^12]: CloudQuery official Architecture docs, inspected 2026-05-16. Evidence used for CLI-orchestrated source/transformer/destination architecture, gRPC plugin runtime, transformer chain, multi-destination flow, and metadata columns.
[^13]: CloudQuery official source plugin resolver docs and Go SDK examples, inspected 2026-05-16. Evidence used for resolver signatures, parent-resource handling, and row streaming behavior.
[^14]: CloudQuery official destination docs and destination write-mode documentation, inspected 2026-05-16. Evidence used for `write_mode`, `overwrite-delete-stale`, `overwrite`, `append`, `migrate_mode`, `pk_mode`, and destination cleanup behavior.
[^15]: CloudQuery official incremental-table docs, inspected 2026-05-16. Evidence used for incremental state backend, cursor resume, at-least-once delivery, duplicates, and no-gap claims.
[^16]: `https://github.com/turbot/steampipe`, repository page and README inspected 2026-05-16. Evidence used for repository identity, branch page, module layout, and live SQL/API query purpose.
[^17]: Steampipe official architecture docs, inspected 2026-05-16. Evidence used for Postgres FDW architecture and gRPC plugin boundary.
[^18]: Steampipe official plugin-authoring docs, inspected 2026-05-16. Evidence used for table definitions, columns, List/Get/Hydrate functions, query flow, predicate/key handling, and projection-driven hydrate behavior.
[^19]: Steampipe official CLI query reference, inspected 2026-05-16. Evidence used for query timeout behavior and interactive/batch execution context.
[^20]: Steampipe official plugin release checklist and plugin development docs, inspected 2026-05-16. Evidence used for config checks, pagination, retry/backoff, max concurrency, context cancellation, and real-account tests.
[^21]: Steampipe official concurrency and rate-limiting docs, inspected 2026-05-16. Evidence used for limiters, scopes, token bucket behavior, `max_concurrency`, and `_ctx` diagnostics.
[^22]: `https://github.com/apache/nifi`, repository page, README, and Apache NiFi release notes inspected 2026-05-16. Evidence used for project purpose, release context, runtime features, and repository identity.
[^23]: Apache NiFi official Overview docs, inspected 2026-05-16. Evidence used for FlowFile, processor, relationship, connection, queue, prioritizer, and backpressure concepts.
[^24]: Apache NiFi official In Depth docs, inspected 2026-05-16. Evidence used for FlowFile attributes/content, content claims, repositories, and copy-on-write model.
[^25]: Apache NiFi official Developer Guide, inspected 2026-05-16. Evidence used for provenance event types, provenance reporter, processor state, and extension-development concepts.
[^26]: Apache NiFi official User Guide, inspected 2026-05-16. Evidence used for queue backpressure defaults, expiration, penalty duration, yield duration, auto retry, and prioritizers.
[^27]: Apache NiFi official Administration Guide, inspected 2026-05-16. Evidence used for local and cluster state management providers and repository/state behavior.
[^28]: Apache NiFi official User Guide access-control and provenance/replay documentation, inspected 2026-05-16. Evidence used for provenance viewing, content access, and replay access policy boundaries.
[^29]: Apache NiFi official REST API documentation, inspected 2026-05-16. Evidence used for programmatic command/control, provenance, content, and replay surfaces.
[^30]: `https://github.com/opensearch-project/data-prepper`, repository page and README inspected 2026-05-16. Evidence used for repository/module layout and product purpose.
[^31]: Data Prepper official Key Concepts and pipeline docs, inspected 2026-05-16. Evidence used for source-buffer-processor-sink model, default bounded blocking buffer, no-op processor, and pipeline YAML.
[^32]: Data Prepper official expression syntax docs, inspected 2026-05-16. Evidence used for conditional routing and event expression behavior.
[^33]: Data Prepper official pipeline configuration and end-to-end acknowledgment docs/blog, inspected 2026-05-16. Evidence used for workers, delay, shutdown timeout context, positive/negative/no-ack, and timeout behavior.
[^34]: Data Prepper official source coordination docs, inspected 2026-05-16. Evidence used for distributed partition coordination, lease stores, and partition states.
[^35]: Data Prepper official peer forwarding docs, inspected 2026-05-16. Evidence used for peer forwarding and stateful aggregation/distributed operation.
[^36]: `https://github.com/debezium/debezium`, repository README and Debezium release/docs pages inspected 2026-05-16. Evidence used for CDC purpose, committed row-level change streaming, and current documentation version context.
[^37]: Debezium official architecture docs, inspected 2026-05-16. Evidence used for Kafka Connect source connector architecture and database-log integration.
[^38]: Debezium official MySQL connector docs, change event value/envelope examples, inspected 2026-05-16. Evidence used for `before`, `after`, `source`, `op`, timestamp fields, source block, transaction metadata, and primary-key update behavior.
[^39]: Debezium official MySQL connector docs, delete and tombstone event behavior, inspected 2026-05-16. Evidence used for delete event and tombstone records.
[^40]: Debezium official MySQL connector docs, snapshot modes, offsets, failure handling, heartbeat, and incremental snapshot configuration defaults, inspected 2026-05-16. Evidence used for snapshot mode enum, offset behavior, `event.processing.failure.handling.mode`, heartbeat default, and chunk-size defaults.
[^41]: Debezium official signaling and incremental snapshot docs, inspected 2026-05-16. Evidence used for incremental snapshot chunking, watermarking, execute/pause/resume signaling, and streaming continuity.
