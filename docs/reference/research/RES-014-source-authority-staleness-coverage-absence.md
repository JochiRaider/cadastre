---
doc_id: RES-014
project: Cadastre
title: Source Authority, Staleness, Coverage, and Absence Semantics Research Report for Cadastre
status: research-report
inspection_date: 2026-05-16
---

## Executive summary

Cadastre should adopt a stricter source-authority and absence contract than any inspected external system. The dominant finding is that external systems expose many useful **signals** but very few transferable **truth authorities**. ServiceNow IRE shows how to separate identification from attribute reconciliation. CloudQuery, Steampipe, NiFi, Data Prepper, Debezium, Flink, Beam, Kafka Streams, and dbt expose state, cursor, progress, acknowledgment, provenance, and freshness signals that are useful operational evidence but unsafe as source truth. Vulnerability scanners, control standards, directory APIs, DNS, DHCP, flow telemetry, and cloud inventory systems show that absence is meaningful only under source-specific coverage, visibility, permission, and staleness contracts.

The governing Cadastre boundary remains correct: the lakehouse is the system of record; graph, CIM, metadata, lineage, and external projections are replaceable derived views; and source adapters may emit only the outputs permitted by the PRD. The highest-value PRD work is to make that boundary mechanically complete by adding first-class staleness, coverage-dimension, progress-signal, control-result, source-history, and total absence-authorization contracts.

| Rank | Recommendation | Target Cadastre contract | Transfer stance |
| ---: | --- | --- | --- |
| 1 | Require `SourceAuthorityProfile` rows at fact type, predicate, source category, source dataset, source instance override, subject scope, object scope, and absence-authority granularity. | `SourceAuthorityProfile` | `adopt` |
| 2 | Add first-class `SourceStalenessPolicy` with source/fact-specific time-input precedence and expiry effects. | `SourceStalenessPolicy`, `VersionManifest` | `adopt` |
| 3 | Add `CoverageDimensionProfile` rows for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, and source history coverage. | `CoverageAssertion` | `adopt` |
| 4 | Add a total `ProgressSignalInterpretationPolicy` that makes cursor exhaustion, delta token, CDC offset, heartbeat, watermark, queue drain, ack, provenance closure, dbt freshness, graph index state, and destination cleanup non-authoritative by default. | `StageStateRecord`, `SourceCompletenessEvidenceRow`, `VersionManifest` | `adopt` |
| 5 | Add `ControlResultMappingRow` for OVAL/XCCDF polarity and result-state mapping. | `ControlResultMappingRow`, `GoldFact` | `adopt` |
| 6 | Add a deterministic `DeriveAbsenceOrUnknown` algorithm that rejects absence from missing rows, zero live-query results, TTL expiry, lease expiry, queue drain, ack success, destination cleanup, watermark, heartbeat, freshness-only signals, graph-index state, and provenance closure unless an active Cadastre profile row permits that source state. | Gold derivation, `SourceCompletenessProfile`, `CoverageAssertion` | `adopt` |
| 7 | Add source-specific UI/API states for stale, unknown, not applicable, not observed, conflicted, ambiguous, source unavailable, scope unavailable, permission limited, partial known gap, and partial unknown gap. | Search, asset detail, graph query, compliance export, audit evidence | `adopt` |

Bottom line: Cadastre should define source authority as a deterministic product-owned contract. External systems may contribute observations, evidence, state, health, lineage, and source-native status, but they must not become accidental authority for fact truth, source absence, graph state, cleanup, lineage closure, destination state, or freshness.

## 1. Source limits and confidence

Report date: **2026-05-16**.

Web sources were checked against current official documentation, specifications, vendor docs, and public standards pages where external behavior is freshness-sensitive. No external repository was cloned. No external system was built, installed, executed, benchmarked, or tested. No vendor API was invoked. Runtime claims are therefore limited to official documentation, standards, uploaded project files, and source-grounded inference.

Uploaded project files used:

- `PRD-Cadastre.md`, treated as governing Cadastre baseline.
- `nlspec-spec.md`, treated as governing specification-quality standard.
- `RES-006-source-adapter-ingestion-patterns.md`, `RES-011-temporal-semantics.md`, `RES-007-normalization-and-schema-standards.md`, `RES-009-identity-resolution-governance.md`, `RES-010-theoretical-reachability-network-context.md`, `RES-008-identity-graph-attack-paths.md`, `RES-005-Lakehouse-Derived-Graph-Architecture.md`, `RES-012-graph-backend-comparison.md`, and related uploaded reports, treated as prior Cadastre research evidence, not authority.

| Evidence class | Meaning |
| --- | --- |
| `confirmed source fact` | Directly supported by inspected source code, official docs, standards, schemas, tests, examples, release metadata, or uploaded project files. |
| `source-grounded inference` | Inferred from one source family’s confirmed facts. |
| `Cadastre applicability inference` | Derived by comparing confirmed facts to the Cadastre PRD. |
| `speculative transfer` | Plausible but requires deeper source inspection, execution, domain authority, or product governance before PRD adoption. |
| `open question` | Requires further source inspection, runtime validation, source-specific data, or product decision. |

The governing Cadastre baseline already establishes the strongest architectural constraint: the lakehouse is the system of record; graph, CIM, metadata, lineage, and other serving outputs are replaceable projections; and table, graph, replay, maintenance, freshness, and lineage artifacts must be governed by Cadastre contracts rather than external defaults.[^1] The NLSpec standard requires the final PRD amendment work to be prescriptive, complete, unambiguous, defaulted, bounded, mapped through exhaustive tables, and testable by binary acceptance criteria.[^13]

## 2. Preliminary source discovery pass

### 2.1 Discovery summary

The discovery pass prioritized sources that expose concrete authority, freshness, coverage, progress, cursor, result-state, visibility, or omission models. Sources were demoted when they mostly provide product marketing, broad taxonomy, graph UI behavior, or general metadata governance without directly sharpening source authority, stale observation, coverage, or absence semantics.

| Priority | Source family | Selected sources | Why selected | Main Cadastre contract affected | Inspection depth |
| ---: | --- | --- | --- | --- | --- |
| 1 | Cadastre governing baseline | `PRD-Cadastre.md`; `nlspec-spec.md` | Governing architecture, current contract vocabulary, omission states, source authority, completeness, coverage, graph projection, and specification-quality requirements. | All target PRD contracts. | Local file inspection with line references. |
| 2 | Cadastre prior research | RES-006, RES-011, RES-007, RES-009, RES-010, RES-008, RES-005, RES-012 | Existing Cadastre research already covers adapter patterns, temporal semantics, schema standards, identity, reachability, graph boundaries, lakehouse replay, and derived-view risk. | Evidence only, not authority. | Uploaded report inspection. No original-source re-execution. |
| 3 | CMDB and reconciliation systems | ServiceNow IRE identification and reconciliation docs | Clear distinction between identification rules, reconciliation rules, source update authority, and simulation. | `SourceAuthorityProfile`, `ResolverProfile`, `CoverageAssertion`. | Official docs only. |
| 4 | Source adapter and ingestion systems | CloudQuery, Steampipe, Apache NiFi, OpenSearch Data Prepper, Debezium, Kafka Connect concepts | Concrete models for plugins, live probes, cursors, destination stale delete, provenance, queue/backpressure, acks, offsets, schema history, snapshots, and streams. | `SourceCallPolicy`, `StageStateRecord`, `SourceCompletenessEvidenceRow`, `ProgressSignalInterpretationPolicy`. | Official docs, prior Cadastre report. No execution. |
| 5 | Stream progress and freshness | Apache Flink, Apache Beam, Kafka Streams concepts, dbt source freshness | Watermarks, allowed lateness, stream time, freshness artifacts, and late data are common accidental-authority hazards. | `SourceStalenessPolicy`, `LateArrivalPolicy`, progress-signal non-authority. | Official docs. No pipeline execution. |
| 6 | Vulnerability scanner and scan coverage systems | Tenable, Qualys, Rapid7 | Concrete scanner state terms such as fixed, reopened, resurfaced, authenticated scan, plugin/check coverage, target coverage, and scan progress. | `CoverageAssertion`, vulnerability absence dimensions, scanner health metrics. | Official docs. No scanner API execution. |
| 7 | Control and compliance result standards | OVAL Results, XCCDF | Explicit result vocabularies include unknown, error, not evaluated, not applicable, and not checked. | `ControlResultMappingRow`, omission mapping, compliance export behavior. | Official standards/docs. No validator execution. |
| 8 | Endpoint, identity, and directory APIs | Microsoft Graph group membership and delta, Entra hidden membership behavior, AD `memberOf`, Kubernetes object identity, OpenTelemetry Entity Data Model | Permission-limited rows, delta tokens, hidden membership, AD primary group caveat, identifying vs descriptive attributes, reusable names versus UIDs. | `CollectionMethodVisibilityProfile`, `ResolverProfile`, directory membership coverage. | Official docs. No tenant or cluster API invocation. |
| 9 | DNS, DHCP, IPAM, flow, and host telemetry | RFC 1035 DNS TTL, RFC 2131 DHCP, Zeek `conn.log`, osquery differential results, NetBox demoted | TTL, leases, flow UIDs, differential query semantics, and intended-state IPAM hazards. | DNS/DHCP/flow staleness, `FlowRoleEvidence`, absence safety. | Official standards/docs. No server logs or sensors executed. |
| 10 | Cloud asset inventory and history | Google Cloud Asset Inventory, AWS Config, Azure Resource Graph change analysis | Source-native history windows, configuration histories, history retention, and change windows. | `SourceHistoryRetentionProfile`, source-history staleness and retention rules. | Official docs. No cloud API execution. |
| 11 | Metadata, lineage, and freshness artifacts | DataHub, OpenLineage, OpenMetadata, dbt artifacts | Strong examples of derived graph/search rebuild, lineage facets, and freshness artifacts that must not become authority. | `RunDatasetIOContract`, `LineageFacetMappingPolicy`, `ArtifactClassPolicy`, `DerivedViewLagPolicy`. | Official docs. No backend execution. |
| 12 | Graph and reachability boundary references | BloodHound/OpenGraph, Batfish, AWS Reachability Analyzer, Google Connectivity Tests, Azure Network Watcher | Useful only to clarify modeled analysis, unsupported scope, representative paths, graph traversability, and unsafe negative evidence. | `GraphEdgeSemantics`, reachability non-implication, graph-output eligibility. | Official docs and prior Cadastre research. No analyzer execution. |

### 2.2 Excluded or demoted source families

| Source family | Disposition | Reason |
| --- | --- | --- |
| Product marketing pages without contracts | Excluded | They do not define result states, schemas, coverage dimensions, state machines, or validation behavior. |
| Generic graph backends | Demoted | Graph backend behavior is relevant to derived-view lag and non-authority, but not to source authority or source absence. |
| Supply-chain frameworks | Demoted | Useful for package activation and trust, but not central to source observation staleness or absence semantics. |
| NetBox/IPAM product docs | Demoted | Useful for intended-state modeling, but the most critical transfer is negative: intended allocation must not be conflated with observed host identity or live assignment. |
| Secondary blog commentary | Excluded except when official/vendor-authored and used narrowly | The report prioritizes official documentation, standards, schemas, and source files. |
| Unexecuted APIs or private product internals | Excluded | No API calls were made; claims requiring tenant-specific behavior remain open questions. |

### 2.3 Refined follow-up source list

The final analysis focuses on the following concrete questions:

| Source family | Follow-up question asked |
| --- | --- |
| ServiceNow IRE | How should Cadastre separate identity matching, attribute authority, source update permission, and absence authority? |
| CloudQuery and Steampipe | What is safe to transfer from plugin/table/live-query models, and what must remain non-authoritative? |
| NiFi and Data Prepper | Which runtime delivery, provenance, queue, and ack signals are useful evidence, and which must be non-authoritative? |
| Debezium and Kafka Connect | Which offsets, schema-history states, snapshots, tombstones, and heartbeats are replay inputs rather than fact time or absence proof? |
| Flink, Beam, Kafka Streams, dbt | How must watermarks, allowed lateness, stream progress, freshness artifacts, and run artifacts be prevented from becoming source completeness? |
| Tenable, Qualys, Rapid7 | Which scan states and coverage dimensions are required before vulnerability absence may be asserted? |
| OVAL and XCCDF | How should control-result vocabularies map to Cadastre states without collapsing unknown/error/notchecked into absence? |
| Microsoft Graph, Entra, AD | Which permission, hidden membership, delta, and primary-group caveats block membership absence? |
| DNS, DHCP, Zeek, osquery | What time and coverage inputs are required before a stale resolution, expired lease, or missing flow can affect facts? |
| Cloud inventory systems | How must source-native history windows be bounded and kept separate from Cadastre retention? |
| DataHub, OpenLineage, OpenMetadata | How should lineage and metadata artifacts remain governance or derived-view artifacts rather than source evidence? |
| Batfish and provider reachability tools | What result states and unsupported-scope limits prevent unsafe negative reachability claims? |

## 3. Cadastre baseline

### 3.1 What the PRD already gets right

Cadastre’s PRD already defines the correct system boundary: raw/bronze, silver, and gold lakehouse records are authoritative, while graph and Splunk CIM are replaceable projections. It also requires table-format-native state references for authoritative lakehouse reads and writes and says graph rebuild, graph lag, replay retention, and destructive table maintenance are Cadastre correctness concerns rather than backend defaults.[^1]

The PRD already prevents the major accidental-authority hazards. It rejects direct-to-graph synchronization, external metadata graphs, freshness artifacts, lineage facets, live source probes, destination cleanup, acknowledgments, queue drain, provenance closure, CDC offsets, CDC heartbeats, schema-history timestamps, graph state, and source-native merge history as source authority unless an explicit Cadastre contract grants the authority.[^2]

The PRD already defines the key contract surface needed for this topic: `SourceCallPolicy`, `SourceCompletenessProfile`, `CollectionMethodVisibilityProfile`, `SourceCompletenessReceipt`, `SourceCompletenessEvidenceRow`, `CoverageAssertion`, `SourceAuthorityProfile`, `ResolverProfile`, and `GraphEdgeSemantics`.[^3] It also defines a closed omission-state vocabulary and explicitly prevents optionality, nullable schema declarations, and source field presence from deciding omission semantics by themselves.[^4]

### 3.2 Residual gap table

| Area | Existing PRD coverage | Residual implementation-divergence risk | Required refinement |
| --- | --- | --- | --- |
| Lakehouse system-of-record boundary | Strong. The PRD defines the lakehouse as authoritative and graph/CIM as projections.[^1] | Low conceptual risk, medium mechanical risk when source progress signals are adjacent to table state. | Add a total `ProgressSignalInterpretationPolicy` that declares every progress/freshness/lineage artifact non-authoritative by default. |
| `SourceAuthorityProfile` | Strong. Gold derivation must use active profiles and missing profiles fail derivation.[^8] | Profiles can remain too coarse if they cover fact type but not predicate, source scope, dataset, and absence authority separately. | Require profile rows at fact type, predicate, source category, source dataset, source instance override, subject scope, and absence-authority granularity. |
| `SourceStalenessPolicy` | Partly implied through `stale_threshold_policy` in `SourceAuthorityProfile`.[^8] | Implementers can choose different clocks for source event time, scan end time, collection time, TTL, lease expiry, last sync, graph apply time, and source history time. | Add first-class `SourceStalenessPolicy` with source/fact-specific time-input precedence and expiry behavior. |
| `SourceCompletenessReceipt` | Strong. Receipt is run evidence and must not self-authorize absence.[^10] | Receipts can still be confused with coverage if dimensions are underspecified. | Add receipt-to-coverage constraints and require explicit `coverage_assertion_id` for coverage-dependent absence or pass/fail facts. |
| `SourceCompletenessProfile` | Strong. Only complete and empty-complete states may authorize absence, retraction, cleanup, or watermark by profile row.[^10] | Some source-specific partiality, permission, and unsupported states lack category-specific mappings. | Add source-category-specific default unsafe states for scanner, directory, DNS, DHCP/IPAM, flow, cloud history, and control sources. |
| `SourceCompletenessEvidenceRow` | Strong. Evidence classes already limit delivery, lineage, liveness, probe, and diagnostic signals.[^7] | Implementers may still combine weak evidence rows into completeness without a deterministic aggregation rule. | Add aggregation precedence: any blocking unsafe row must dominate candidate completeness evidence. |
| `CoverageAssertion` | Present and required for scanner/control/directory/network coverage-sensitive facts.[^3] | Required coverage dimensions are not yet exhaustive by source category. | Add `CoverageDimensionProfile` with required dimensions per source class. |
| Omission states | Field-level states are defined.[^4] | Higher-level absence states such as `source_unavailable`, `scope_unavailable`, `permission_limited`, and `partial_gap` are not fully mapped to field omission states. | Add an absence outcome vocabulary for fact derivation separate from field omission vocabulary. |
| `StageStateRecord` | Requires output-affecting state to be manifest-included.[^6] | Cursor, delta token, CDC offset, schema history, queue state, and ack state may be treated differently by implementations. | Add closed `state_kind` rows and commit-order rules for each progress state. |
| `SourceCallPolicy` | Defines pagination, retry, rate limit, timeout, duplicate-page, partial-page, credential-error, and visibility-loss behavior.[^3] | Cursor exhaustion and page completion can be overinterpreted as coverage. | Require cursor exhaustion to be candidate completeness evidence only, never absence authority by itself. |
| `CollectionMethodVisibilityProfile` | Present and relevant to volatile or permission-dependent collection methods.[^3] | Scanner auth, directory hidden membership, local group/session volatility, flow capture point, and plugin/check selection need explicit dimensions. | Add method-specific visibility rows and required fixture classes. |
| `GoldFact` | Strong bitemporal model with assertion state, evidence refs, authority, and coverage-sensitive fact notes.[^5] | Missing-row absence and stale fact behavior can diverge. | Require `DeriveAbsenceOrUnknown` before any negative, stale, or expired assertion is emitted. |
| `ResolverProfile` | Strong identity authority boundary.[^3] | Directory, scanner, graph, and source-native merge history can leak into identity authority. | Add identity-specific source authority rows for visibility-limited membership and source-native merge history. |
| `GraphEdgeSemantics` | Strong graph edge meaning, traversal, evidence, and non-implication boundary.[^9] | Graph index state, graph drift checks, and graph cleanup can still become de facto truth operationally. | Add explicit rejection rows in `ProgressSignalInterpretationPolicy` and UI labels for graph stale reads. |
| `DerivedViewLagPolicy` | Strong stale read handling and default stale-read rejection for compliance/audit/rule contexts.[^12] | Derived-view lag can be mistaken for source staleness. | Require API responses to distinguish `source_stale` from `derived_view_stale`. |
| `VersionManifest` | Very strong. Production output requires manifest capture of every output-affecting profile, state, dataset, coverage, authority, and graph policy.[^6] | New staleness and progress policies must be manifest-included. | Add `SourceStalenessPolicy`, `CoverageDimensionProfile`, `ProgressSignalInterpretationPolicy`, `ControlResultMappingRow`, and `SourceHistoryRetentionProfile` refs. |

## 4. Independent source analyses

### 4.1 CMDB and reconciliation systems

#### 4.1.1 ServiceNow Identification and Reconciliation Engine

##### Source overview (4.2.1)

ServiceNow IRE solves CMDB CI identification and attribute reconciliation. It does not solve Cadastre source authority, canonical identity, bitemporal fact derivation, coverage proof, or absence semantics. Its value is the separation of **identification rules**, **reconciliation rules**, **data-source update permission**, and **simulation**.[^15]

##### Architecture and operational model (4.2.1)

ServiceNow IRE processes incoming CMDB payloads before inserts or updates, applies identification rules to decide whether a CI matches an existing record, and applies reconciliation rules to decide which source may update which attributes. Data-source rules can authorize or block inserts or updates independently from identification. Simulation can evaluate payloads without committing changes.[^15]

##### Data, schema, and state model (4.2.1)

| Source concept | Model | Cadastre-safe analogue |
| --- | --- | --- |
| Identification rule | Class-scoped rule that identifies a CI using required or unique attributes. | `ResolverProfile` decision row, not source authority. |
| Reconciliation rule | Attribute-level data-source priority or authorization. | `SourceAuthorityProfile` row for a fact predicate. |
| Data-source rule | Source may update but not insert, or insert but not update. | Source permission metadata; must not imply absence authority. |
| Dependent CI | CI identification requires parent or relationship context. | Coverage or identity rule requiring subject scope and relationship evidence. |
| Simulation | Non-mutating rule evaluation. | `ValidationScenario`, `ResolverShadowRun`, or profile activation test. |

##### Source authority model (4.2.1)

ServiceNow distinguishes identity authority from attribute update authority. Cadastre must copy that separation. A source that may update a CMDB attribute is not automatically authoritative for Cadastre fact presence, absence, retraction, cleanup, or coverage.

| Authority dimension | ServiceNow behavior | Cadastre implication |
| --- | --- | --- |
| Identity authority | Identification rules match CIs. | Must remain `ResolverProfile`, never imported as-is. |
| Attribute authority | Reconciliation controls attribute writes. | Transfer to `SourceAuthorityProfile` by predicate and scope. |
| Source update authority | Data-source rules govern insert/update permission. | Permission only; not absence authority. |
| Absence authority | Not an IRE guarantee. | Must require Cadastre completeness and coverage. |
| Retraction/cleanup authority | CMDB update/delete behavior is source-internal. | Must not become Cadastre cleanup authority. |

##### Staleness and freshness model (4.2.1)

IRE is not a staleness model for Cadastre. CMDB updated time or reconciliation winner time may inform source metadata, but it must not become Cadastre fact valid time, known time, or absence expiry.

##### Coverage model (4.2.1)

ServiceNow identification can validate payload completeness for a matching operation, but it does not prove source-scope coverage. Cadastre must not infer that all CIs, all attributes, or all relationships were observed because an IRE payload was accepted.

##### Absence and omission semantics (4.2.1)

A missing attribute in a CMDB payload may mean absent, omitted, unchanged, not authorized, not collected, or not applicable. Cadastre must map it to `unknown` unless the source profile, completeness receipt, and coverage assertion authorize a stronger state.

##### Validation, fixture, and acceptance patterns (4.2.1)

Simulation is the best transferable pattern. Cadastre should require non-mutating profile tests for attribute authority, identity candidate generation, permission-limited payloads, missing required identifiers, and rejected absence.

##### Confirmed findings versus inferences (4.2.1)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | IRE separates identification and reconciliation, and reconciliation governs which source may update attributes. | ServiceNow docs.[^15] | Add explicit separation between identity authority and attribute authority in PRD amendments. |
| Source-grounded inference | Source update permission can be narrower or broader than source truth. | ServiceNow reconciliation and data-source rules.[^15] | Update permission must not imply presence, absence, or coverage authority. |
| Cadastre applicability inference | CMDB reconciliation winner must not become Cadastre truth. | PRD non-authority boundary plus ServiceNow docs.[^2][^15] | Treat CMDB outputs as source observations subject to Cadastre authority profiles. |
| Speculative transfer | Simulation payload generation can seed Cadastre validation scenarios. | ServiceNow simulation docs.[^15] | Use only after Cadastre defines scenario schemas. |
| Open question | Exact ServiceNow runtime behavior under conflicting dynamic/static reconciliation rules was not executed. | No instance used. | Keep transfer conceptual. |

##### Transferability to Cadastre (4.2.1)

| Transferable idea | Target Cadastre contract | Stance | Required PRD change | Default behavior | Error behavior | Validation | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Attribute-level authority | `SourceAuthorityProfile` | `adapt` | Add predicate and scope row requirements. | Missing row blocks gold derivation. | `SOURCE_AUTHORITY_PROFILE_ERROR`. | Authority row coverage fixture. | Same input emits same authority decision and checksum. |
| Non-mutating simulation | `ValidationScenario` | `adapt` | Add profile simulation cases for identity and source authority. | Simulation writes no production records. | `VALIDATION_SCENARIO_FAILED`. | Golden scenarios. | No production table changes occur. |
| Data-source insert/update separation | `SourceAuthorityProfile`, `SourceCallPolicy` | `adapt` | Add `source_update_permission_only` flag. | Permission metadata is non-authoritative. | `NOT_AUTHORITATIVE_FOR_ABSENCE`. | Permission-limited fixture. | Update permission alone cannot emit absence. |

##### Concepts that must not transfer (4.2.1)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| ServiceNow CMDB reconciliation winner as truth | It is a CMDB write rule, not Cadastre bitemporal authority. | Treat as source observation. Derive gold only through `SourceAuthorityProfile`. |
| Source update permission as absence authority | Permission to update does not prove coverage. | Require completeness and coverage. |
| CMDB missing attribute as negative fact | Payload may be partial or unchanged. | Default `unknown`. |
| CMDB delete/update as Cadastre cleanup | External cleanup can delete local source state, not Cadastre facts. | Cleanup requires Cadastre profile and evidence. |

### 4.2 Source adapter and ingestion systems

#### 4.2.1 CloudQuery

##### Source overview (4.2.2)

CloudQuery is an ELT-style source/destination plugin system. It is valuable for plugin/table/resolver structure, incremental state, and destination-mode cautions. It is not a Cadastre authority model. Prior Cadastre research identified CloudQuery’s plugin/table/resolver decomposition and incremental-state discipline as transferable, while rejecting destination-authoritative cleanup.[^14]

##### Architecture and operational model (4.2.2)

CloudQuery source plugins collect tables through resolvers and write results to destinations. Incremental tables use state backends and cursors. Destination `overwrite-delete-stale` can delete rows in a destination that no longer appear in a sync result.[^16]

##### Data, schema, and state model (4.2.2)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Source plugin table | `SourceStageStep.source_dataset` | Dataset shape only. |
| Resolver | Collection step | May emit raw/completeness/errors only. |
| Incremental cursor | `StageStateRecord.state_kind = api_cursor` or `sync_cursor` | Progress state, not fact time. |
| At-least-once duplicate behavior | Duplicate raw status or deterministic dedupe rule | Does not prove absence. |
| Destination stale delete | Diagnostic-only cleanup observation | Never source absence. |

##### Source authority model (4.2.2)

CloudQuery does not define Cadastre fact authority. Its tables are source collection units; destination state is not source truth. Cadastre must treat CloudQuery-like stale deletion as a forbidden absence input unless a source profile separately proves complete coverage.

##### Staleness and freshness model (4.2.2)

CloudQuery sync time and cursor state may inform collection progress. They must not become `GoldFact.valid_time`, `GoldFact.known_time`, stale fact expiry, cleanup permission, or source completeness by themselves.

##### Coverage model (4.2.2)

Cursor exhaustion and expected counts can be candidate completeness evidence only when the source API contract, source scope, permission scope, and dataset scope are covered. Cursor exhaustion alone is not coverage proof.

##### Absence and omission semantics (4.2.2)

A row missing from a destination after `overwrite-delete-stale` means the destination no longer has that row after the sync mode. It does not mean the source subject was observed and the condition was absent.

##### Validation, fixture, and acceptance patterns (4.2.2)

Cadastre should require fixtures for full sync, incremental duplicate page, cursor loss, cursor commit failure, partial page, expected count mismatch, permission loss, and destination cleanup rejection.

##### Confirmed findings versus inferences (4.2.2)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | CloudQuery incremental tables use cursor state and may provide at-least-once behavior with possible duplicates. | CloudQuery docs.[^16] | `StageStateRecord` must capture cursor state and duplicate behavior. |
| Confirmed source fact | CloudQuery destination `overwrite-delete-stale` deletes rows no longer present in a sync result. | CloudQuery docs.[^16] | Destination cleanup must be diagnostic-only. |
| Cadastre applicability inference | Destination stale delete must not authorize Cadastre absence. | PRD non-scope and prior research.[^2][^14] | Add explicit rejection metric and acceptance test. |

##### Transferability to Cadastre

| Transferable idea | Target Cadastre contract | Stance | Required PRD change | Default behavior | Error behavior | Validation | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Incremental cursor state | `StageStateRecord` | `adopt` | Add cursor state kinds and commit ordering. | Cursor commit after raw and receipt persistence only. | `CURSOR_COMMIT_ORDER_ERROR`. | Cursor fixture. | Cursor cannot advance past failed page. |
| Plugin table/resolver decomposition | `SourcePackageDeveloperContract` | `adapt` | Specify internal decomposition as allowed, not required. | Adapter emits raw/completeness/errors only. | `FORBIDDEN_STAGE_OUTPUT`. | Package fixture. | Resolver cannot emit silver/gold/graph. |
| Destination cleanup rejection | `ProgressSignalInterpretationPolicy` | `adopt` | Add row for `destination_cleanup_observed`. | Diagnostic-only. | `DESTINATION_CLEANUP_NOT_SOURCE_EVIDENCE`. | Cleanup fixture. | Destination delete cannot emit absence. |

##### Concepts that must not transfer (4.2.2)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Destination `overwrite-delete-stale` as source absence | Destination state is not source coverage. | Diagnostic-only. |
| Cursor exhaustion as total coverage | Cursor completion can be scoped, partial, or permission-limited. | Candidate completeness only. |
| Sync time as fact time | It is collection time. | Use `SourceStalenessPolicy` time precedence. |
| Destination primary key as canonical identity | Destination key is storage-specific. | Resolver decision required. |

#### 4.2.2 Steampipe

##### Source overview (4.2.3)

Steampipe exposes live source APIs as SQL tables. It is useful for source exploration, predicate pushdown, projection pushdown, and developer ergonomics. It is unsafe as production evidence or absence authority by default.[^17]

##### Architecture and operational model (4.2.3)

Steampipe uses a query-driven model. A SQL query invokes plugin `List`, `Get`, or hydrate functions against live APIs. A zero-row result can mean no matching row, insufficient permissions, filter mismatch, API failure, plugin limitation, rate limit, or unsupported qualifier.

##### Data, schema, and state model (4.2.3)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Live SQL table | `SourceExplorationResult` | Non-authoritative. |
| `List`/`Get` | Live probe or source call pattern | No production writes by default. |
| Hydrate function | Optional fetch expansion | Must not mutate authoritative meaning before raw persistence. |
| Predicate pushdown | Source-call optimization metadata | Must not change semantics. |
| Zero rows | Probe result | Never absence by default. |

##### Source authority model (4.2.3)

Steampipe does not provide source-scope completeness. Query results are live inspection outputs. Cadastre must permit Steampipe-like behavior only as `source_exploration` or `live_source_probe` unless a separate production capture contract persists exact raw payloads, request shape, scopes, timestamps, source-call policy, and completeness evidence.

##### Staleness and freshness model (4.2.3)

Live query time is probe time. It is not proof that a source observation is fresh, stale, or absent.

##### Coverage model (4.2.3)

Coverage is not proven by a live query. Permission scope, qualifiers, page completion, API errors, plugin limitations, and target scope must be represented explicitly.

##### Absence and omission semantics (4.2.3)

Zero rows must map to `unknown` or `no-op` by default, not `not_observed`.

##### Validation, fixture, and acceptance patterns (4.2.3)

Cadastre should test that live zero-row probes cannot emit `RawRecord`, `GoldFact`, `CoverageAssertion`, `SourceCompletenessReceipt`, graph deltas, or production health state.

##### Confirmed findings versus inferences (4.2.3)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Steampipe is a zero-ETL live API query system. | Steampipe docs.[^17] | Treat as live probe/source exploration by default. |
| Cadastre applicability inference | Live zero-row result is not absence. | PRD live-probe non-authority boundary.[^2] | Add acceptance test and rejection metric. |
| Speculative transfer | Predicate pushdown can improve source-call efficiency. | Steampipe docs.[^17] | Safe only if declared non-semantic optimization. |

##### Concepts that must not transfer (4.2.3)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Live zero-row result as absence | Query may be incomplete, filtered, or permission-limited. | Default `unknown` or no-op. |
| Hydrated row as production raw evidence | Exact request/response may not be persisted. | Persist raw through adapter collection only. |
| SQL table schema as source authority | It is a query abstraction. | Source dataset contract required. |

#### 4.2.3 Apache NiFi

##### Source overview (4.2.4)

NiFi is a flow-based ingestion system with FlowFiles, queues, backpressure, provenance events, and replay. It is valuable for runtime lineage and queue/backpressure patterns. It does not prove source truth or completeness.[^18]

##### Architecture and operational model (4.2.4)

NiFi processors move FlowFiles through queues. Provenance records events such as receive, route, fetch, send, drop, replay, and fork/join. Queues and backpressure represent pipeline state, not source state.[^18]

##### Data, schema, and state model (4.2.4)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| FlowFile content | `RawRecord.source_event_subtype = flowfile_content` | Raw delivery evidence only. |
| Provenance event | `IngestionProvenanceEvent` | Lineage-only by default. |
| Queue drain | `SourceCompletenessEvidenceRow.evidence_class = queue_drain_observed` | Lineage-only. |
| Replay | Diagnostic or recovery event | Not source re-observation. |
| Backpressure resolution | Operational health | Not completeness. |

##### Source authority model (4.2.4)

NiFi provenance describes pipeline movement. It does not decide whether a source observed all subjects or whether a missing record is a negative observation.

##### Staleness and freshness model (4.2.4)

Provenance event time is ingestion runtime time. It may support operational health, but it must not become source event time, fact time, or absence time.

##### Coverage model (4.2.4)

A drained queue proves only that the queue is empty at a point in the ingestion pipeline. It does not prove the source had no additional data, that upstream collection finished, or that all scopes were reachable.

##### Absence and omission semantics (4.2.4)

Missing FlowFile content maps to pipeline gap or source unavailable depending on evidence, never to source absence by default.

##### Validation, fixture, and acceptance patterns (4.2.4)

Cadastre should test FlowFile replay, expired FlowFile before raw persistence, queue drain, partial queue, and provenance closure rejection.

##### Confirmed findings versus inferences (4.2.4)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | NiFi has FlowFile queues, provenance, and replay/backpressure concepts. | NiFi docs.[^18] | Model as ingestion lineage and operational health. |
| Cadastre applicability inference | Provenance closure cannot prove source completeness. | PRD `SourceCompletenessEvidenceRow` limits.[^7] | Add rejection metric. |

##### Concepts that must not transfer (4.2.4)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Provenance closure as source truth | It describes pipeline events only. | `IngestionProvenanceEvent` lineage-only. |
| Queue drain as completeness | It proves queue state, not source scope. | `queue_drain_observed` lineage-only. |
| Replay as re-observation | Replay repeats pipeline data, not source observation. | Diagnostic/recovery only. |

#### 4.2.4 OpenSearch Data Prepper

##### Source overview (4.2.5)

Data Prepper defines a source-buffer-processor-sink pipeline. End-to-end acknowledgment can notify a source when delivery succeeds, and source plugins may then remove or commit upstream state. This is delivery evidence, not source completeness.[^19]

##### Architecture and operational model (4.2.5)

A pipeline has source, optional buffer, optional processors, and sinks. End-to-end acknowledgments track successful delivery to sinks when supported. Ack failure, timeout, or limitations indicate delivery state, not source coverage.[^19]

##### Data, schema, and state model (4.2.5)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Source plugin | Adapter source call | Collection evidence. |
| Buffer | Pipeline state | Operational health. |
| Processor | Transformation stage | Must not alter authority before raw persistence. |
| Sink ack | `ack_success` or `ack_timeout` evidence row | Delivery-only. |
| End-to-end ack limitation | Completeness blocker | Unsafe state. |

##### Source authority model (4.2.5)

Data Prepper acks are sink delivery facts. They are not source observations, coverage assertions, or absence authority.

##### Staleness and freshness model (4.2.5)

Ack time is delivery time. It must not become source event time, scan end time, or fact known time.

##### Coverage model (4.2.5)

An ack means a batch reached a sink. It does not prove all source partitions, all pages, all subjects, or all checks completed.

##### Absence and omission semantics (4.2.5)

Ack success maps to no absence. Ack timeout maps to `source_unavailable`, `partial_unknown_gap`, or unsafe completeness for the affected scope.

##### Validation, fixture, and acceptance patterns (4.2.5)

Test ack success, ack timeout, peer-forwarding limitation, sink failure, and processor drop before raw persistence.

##### Confirmed findings versus inferences (4.2.5)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Data Prepper pipelines include sources, buffers, processors, and sinks. | Data Prepper docs.[^19] | Model pipeline components as runtime lineage only. |
| Confirmed source fact | End-to-end acknowledgment indicates delivery to sinks when supported. | Data Prepper docs.[^19] | `ack_success` must be delivery-only evidence. |
| Cadastre applicability inference | Ack success must not authorize source completeness. | PRD evidence limits.[^7] | Add acceptance test. |

##### Concepts that must not transfer (4.2.5)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Sink acknowledgment as source completeness | Delivery success is not source-scope completion. | Delivery-only evidence. |
| Processor success as source truth | It is runtime processing success. | Diagnostic or health only. |
| Buffer drain as absence | It is pipeline state. | No-op for facts. |

#### 4.2.5 Debezium and Kafka Connect

##### Source overview (4.3.1)

Debezium is a CDC platform built on Kafka Connect concepts. It streams database changes, tracks offsets, uses schema history, snapshots, tombstones, and heartbeats. These are replay and ingestion signals, not Cadastre fact time, absence proof, or coverage proof.[^20]

##### Architecture and operational model (4.3.1)

A connector monitors a database and writes change events to Kafka topics. Offsets track log position. Schema history is required to interpret changes. Snapshots can establish initial state. Tombstones and deletes reflect source database row changes. Heartbeats indicate connector liveness.[^20]

##### Data, schema, and state model (4.3.1)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Initial snapshot | Collection pattern | Source DB row state only within declared scope. |
| Log offset / LSN / binlog position | `StageStateRecord.state_kind = cdc_offset` | Replay position only. |
| Schema history | `StageStateRecord.state_kind = cdc_schema_history_ref` | Parsing/replay dependency. |
| Tombstone/delete | `RawRecord.source_event_subtype = cdc_tombstone` | Raw delete evidence, not automatic retraction. |
| Heartbeat | `RawRecord.source_event_subtype = cdc_heartbeat` | Liveness only. |

##### Source authority model (4.3.1)

CDC event presence may be strong evidence that a source row changed. It is not automatically authoritative for Cadastre gold facts unless the source dataset and predicate have an active `SourceAuthorityProfile`. CDC delete or tombstone is not automatic fact retraction unless source authority, completeness, and staleness rules permit it.

##### Staleness and freshness model (4.3.1)

Offsets, connector timestamps, heartbeat timestamps, snapshot timestamps, and schema-history timestamps must not determine `GoldFact.valid_time`, `GoldFact.known_time`, source observation time, or absence by themselves. Cadastre’s PRD already forbids that substitution.[^2]

##### Coverage model (4.3.1)

A CDC offset can prove that a connector processed to a log position. It does not prove all source tables or predicates were complete unless the CDC profile defines the table set, schema history, snapshot mode, offset ranges, partitions, and gap behavior.

##### Absence and omission semantics (4.3.1)

A missing CDC event is not no change unless the connector state, offset, schema history, topic partition, source database, table scope, and retention window are proven complete.

##### Validation, fixture, and acceptance patterns (4.3.1)

Cadastre should require fixtures for initial snapshot, snapshot interruption, schema-history missing, tombstone handling, heartbeat lag, offset commit failure, log gap, connector restart, and source history outside retained log window.

##### Confirmed findings versus inferences (4.3.1)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Debezium streams database changes and uses Kafka Connect connector concepts. | Debezium docs.[^20] | Add CDC-specific raw subtypes, state kinds, and replay dependencies. |
| Source-grounded inference | Offsets and schema history are replay dependencies. | Debezium docs.[^20] | They must be manifest-included but non-authoritative for absence. |
| Cadastre applicability inference | Heartbeat cannot prove no source changes. | PRD CDC non-authority boundary.[^2] | Add heartbeat rejection test. |

##### Concepts that must not transfer

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| CDC offset as fact time | Log position is progress state. | Use only for replay. |
| CDC heartbeat as no-change proof | Heartbeat is liveness. | Liveness-only. |
| Schema history timestamp as source event time | It is parsing metadata. | Schema dependency only. |
| Tombstone as automatic retraction | Retraction requires authority and coverage. | Raw delete evidence only. |

### 4.3 Stream progress, freshness, metadata, and lineage systems

#### 4.3.1 Apache Flink

##### Source overview (4.4.1)

Flink defines event time, processing time, windows, allowed lateness, and watermarks. Watermarks are event-time progress signals. They must not become source-scope completeness or absence proof in Cadastre.[^21]

##### Architecture and operational model (4.4.1)

Flink operators process streams and advance event-time watermarks. Late events can arrive after watermarks and are handled by window and allowed-lateness policy.

##### Data, schema, and state model (4.4.1)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Event time | Candidate source event time | Must be resolved by source policy. |
| Processing time | Runtime time | Not fact time. |
| Watermark | Progress signal | Not source completeness. |
| Allowed lateness | Late-arrival policy | Cannot discard authoritative evidence by default. |
| Window close | Derived output trigger | Not absence. |

##### Source authority model (4.4.1)

Flink does not define source fact authority. A watermark says the stream processor believes event time has progressed; it does not prove the upstream source has no missing records.

##### Staleness and freshness model (4.4.1)

Watermark time can inform late-arrival routing. It must not set stale fact expiry or absence.

##### Coverage model (4.4.1)

A watermark is not target, subject, predicate, permission, or scan coverage. Cadastre coverage must come from `CoverageAssertion`.

##### Absence and omission semantics (4.4.1)

No event before a watermark does not mean `not_observed`.

##### Validation, fixture, and acceptance patterns (4.4.1)

Test a watermark advancing past a missing vulnerability record; the system must not emit absence unless a source completeness and coverage profile independently authorizes it.

##### Confirmed findings versus inferences (4.4.1)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Flink watermarks represent event-time progress and late events can occur. | Flink docs.[^21] | Watermarks are progress signals only. |
| Cadastre applicability inference | Watermark-as-absence must be rejected. | PRD non-authority boundary.[^2] | Add `WATERMARK_AS_ABSENCE_REJECTED`. |

#### 4.3.2 Apache Beam

Beam similarly distinguishes event time, processing time, triggers, watermarks, and late data. Beam watermarks are progress estimates and may be heuristic; they are not source completeness.[^22]

| Dimension | Cadastre handling |
| --- | --- |
| Beam event time | Candidate event time only if source policy accepts it. |
| Beam processing time | Runtime metadata only. |
| Beam watermark | Progress signal only. |
| Beam late data | Must be preserved as raw evidence; late authoritative records trigger correction evaluation. |
| Beam trigger pane | Derived-output timing only. |

Confirmed finding: Beam docs define triggers and late data after watermarks, and Java docs describe watermarks as lower-bound progress estimates.[^22] Cadastre implication: production default must preserve late authoritative evidence rather than discard it.

#### 4.3.3 dbt source freshness

dbt source freshness artifacts are workflow outputs that assess freshness of declared sources. They are useful as health signals, not coverage proof. A freshness artifact must not authorize absence, retraction, cleanup, or source completeness.[^29]

| dbt concept | Cadastre default |
| --- | --- |
| `sources.json` freshness result | Health metadata only. |
| Source freshness timestamp | Staleness candidate only if `SourceStalenessPolicy` maps it. |
| Freshness pass | Not coverage. |
| Freshness fail | Health degradation; not retraction. |

#### 4.3.4 DataHub, OpenLineage, and OpenMetadata

DataHub is most useful for its separation between persisted metadata aspects and derived graph/search indexes; index restore can rebuild graph/search from authoritative persisted metadata, but the graph/search index is not the source of truth.[^30] OpenLineage models run, job, dataset, and facets. OpenMetadata models metadata entities, change events, and lineage. These are metadata and lineage systems, not Cadastre evidence or source completeness systems.[^30]

| Source | Useful concept | Cadastre handling |
| --- | --- | --- |
| DataHub | Rebuild derived graph/search from persisted source-of-truth table. | Adapt to `GraphRebuildManifest`; graph index remains derived. |
| OpenLineage | Run/job/dataset/facet event model. | `RunDatasetIOContract`, `LineageFacetMappingPolicy`; non-authoritative by default. |
| OpenMetadata | Metadata change events and lineage relationships. | Governance metadata only; not gold fact authority. |
| dbt | Workflow artifact separation. | Health and artifact-class boundary only. |

### 4.4 Vulnerability scanners and control standards

#### 4.4.1 Tenable, Qualys, and Rapid7

##### Source overview

Vulnerability scanners provide evidence about vulnerability presence, fixed/resolved/reopened/resurfaced states, scan status, authenticated scan state, target inclusion, plugin/check coverage, and scan progress. They do not provide unconditional absence authority.

##### Architecture and operational model

Tenable documents scan progress/status and vulnerability lifecycle states such as active, fixed, reopened, and resurfaced.[^23] Qualys documents active, fixed, and reopened behavior after subsequent detections.[^24] Rapid7 documentation emphasizes credentials/authenticated scans and coverage considerations for dynamic asset inventories and recurring vulnerability content.[^24]

##### Data, schema, and state model

| Scanner concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Finding active/open | Vulnerability presence observation | Presence authority only if source profile permits. |
| Fixed/mitigated | Candidate negative observation | Requires scan coverage and staleness policy. |
| Reopened/resurfaced | Positive observation after prior fixed | Must reopen or supersede prior absence through correction. |
| Scan completed | Run state | Not coverage by itself. |
| Authenticated scan | Coverage dimension | Required where checks depend on authentication. |
| Plugin/check inclusion | Coverage dimension | Required for vulnerability/control absence. |
| Target inclusion | Coverage dimension | Required subject scope. |
| Port/service coverage | Coverage dimension | Required for network/service vulnerability claims. |

##### Source authority model

A scanner may be authoritative for vulnerability presence for the assets and checks it observed. Absence requires stronger conditions: authenticated status where applicable, plugin/check enabled, target included, scan finished, no blocking error, relevant ports/services covered, source dataset scope complete, and staleness not expired.

##### Staleness and freshness model

Scan end time is the default staleness basis for scanner findings. Detection time may be useful for valid-time of a finding. Scanner publish time, dashboard update time, and API sync time must not replace scan end time unless a profile row explicitly chooses them.

##### Coverage model

Coverage must include at least:

```text
scanner_instance
scan_id
scan_policy_id
target_scope
credential/authentication status
plugin/check set
port/service scan coverage
agent/network sensor scope
excluded targets
failed targets
scan_start
scan_end
result publication status
permission scope
```

##### Absence and omission semantics

| Source state | Cadastre mapping |
| --- | --- |
| Finding missing from latest scan without coverage | `unknown` or `not_authoritative_for_absence`. |
| Scanner `fixed` with qualifying coverage | Candidate `not_observed` or retraction subject to authority and staleness. |
| Scanner `fixed` without plugin/check coverage | `unknown`. |
| Scanner source unavailable | `source_unavailable`. |
| Scan partial | `partial_known_gap` or `partial_unknown_gap`. |
| Auth failure | `permission_limited` or `scope_unavailable`. |
| Reopened/resurfaced | Positive presence correction; prior absence must close known interval. |

##### Validation, fixture, and acceptance patterns

Cadastre must test authenticated scan failure, plugin disabled, target excluded, partial scan, scan completed but publishing incomplete, fixed without coverage, reopened after fixed, and stale fixed state.

##### Confirmed findings versus inferences

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Tenable and Qualys expose fixed/reopened/resurfaced-like lifecycle states. | Vendor docs.[^23][^24] | Native scanner status must map through Cadastre result rules. |
| Confirmed source fact | Rapid7 documents authenticated scanning and coverage concerns. | Rapid7 docs.[^24] | Coverage dimensions must include authentication and target coverage. |
| Cadastre applicability inference | Scanner `fixed` is not automatic absence. | PRD source authority table and coverage rules.[^11] | Require `CoverageAssertion`. |

##### Concepts that must not transfer (4.4.1)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Scanner `fixed` as automatic absence | Fixed can depend on coverage, plugin, auth, and recency. | Require coverage and staleness. |
| Completed scan as complete coverage | Scan may exclude targets/checks or fail auth. | Coverage dimensions required. |
| Reopened/resurfaced as duplicate only | It changes temporal state after fixed. | Emit correction/change-set. |
| Scanner asset merge as identity authority | Scanner correlation can be wrong or scoped. | Resolver profile required. |

#### 4.4.2 OVAL and XCCDF

##### Source overview (4.4.2)

OVAL Results and XCCDF define control/check result vocabularies. Their value is explicit non-binary states. They are not Cadastre fact authority, and their scores or result states do not prove source coverage.

##### Data, schema, and state model (4.4.2)

OVAL result vocabularies include `true`, `false`, `unknown`, `error`, `not evaluated`, and `not applicable` states.[^25] XCCDF result vocabularies include pass/fail/error/unknown/notchecked/notapplicable and related states, depending on schema version and rule context.[^25]

| Source state | Cadastre mapping |
| --- | --- |
| OVAL `true` | Candidate control pass/fail depending on rule polarity. |
| OVAL `false` | Candidate inverse state depending on rule polarity. |
| OVAL `unknown` | `unknown`. |
| OVAL `error` | `unknown` plus error evidence. |
| OVAL `not evaluated` | `not_observed` is forbidden; map `not_checked`. |
| OVAL `not applicable` | `not_applicable`. |
| XCCDF `pass` | Candidate pass, polarity-controlled. |
| XCCDF `fail` | Candidate fail, polarity-controlled. |
| XCCDF `error` | `unknown` plus error evidence. |
| XCCDF `unknown` | `unknown`. |
| XCCDF `notchecked` | `not_checked`, not absence. |
| XCCDF `notapplicable` | `not_applicable`. |

##### Authority, staleness, coverage, and absence model

Control pass/fail/unknown facts require a control-result mapping row, source authority profile, staleness policy, and coverage assertion. Unknown, error, not evaluated, and not checked must never map to negative compliance or absence by default.

##### Validation patterns

Cadastre must define `ControlResultMappingRow` with rule polarity, expected evidence refs, result vocabulary, not-applicable conditions, error behavior, unknown behavior, and coverage requirements.

##### Concepts that must not transfer (4.4.2)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| OVAL unknown as absence | Unknown means insufficient or unavailable evaluation. | Map to `unknown`. |
| OVAL error as failure or absence | Error is evaluation failure. | Error evidence and unknown state. |
| XCCDF score as factual state | Score aggregates checks and weights. | Analysis/compliance output only. |
| XCCDF notchecked as pass | It means not checked. | `not_checked`/`unknown`, no absence. |

### 4.5 Endpoint, identity, and directory APIs

#### 4.5.1 Microsoft Graph, Entra ID, and Active Directory

##### Source overview (4.5.1)

Microsoft Graph and Entra APIs expose device, group, membership, and delta query surfaces. They are highly relevant because permission limitations and hidden membership can make missing rows unsafe. Active Directory `memberOf` has primary-group limitations.[^26]

##### Architecture and operational model (4.5.1)

Microsoft Graph delta queries return pages and state tokens. Group membership APIs can return limited information or require additional permissions for hidden membership. Active Directory `memberOf` excludes the user’s primary group, which is represented separately by `primaryGroupID`.[^26]

##### Data, schema, and state model (4.5.1)

| Source concept | Cadastre analogue | Authority limit |
| --- | --- | --- |
| Group member row | Membership observation | Presence only unless membership scope covered. |
| Limited-info member row | Permission-limited observation | `permission_limited`. |
| Hidden membership permission | Visibility dimension | Required for absence. |
| Delta token | `StageStateRecord.state_kind = delta_token` | Progress state, not completeness by itself. |
| Delta reset/token expiry | Coverage gap or full resync required | Blocks absence. |
| AD `memberOf` | Partial membership list | Excludes primary group. |
| `primaryGroupID` | Separate membership evidence | Required to cover primary group. |

##### Source authority model (4.5.1)

Directory sources can be authoritative for group membership presence and absence only when the source profile covers tenant/domain, group, hidden membership visibility, transitive/direct membership mode, delta validity or full enumeration, and AD primary group handling.

##### Staleness and freshness model (4.5.1)

Directory membership staleness should use source observation time when provided, otherwise collection time, but only within the declared staleness policy. Delta token time is progress metadata, not fact valid time.

##### Coverage model (4.5.1)

Coverage dimensions:

```text
tenant_or_domain
group_id
membership_mode = direct | transitive
hidden_membership_permission
limited_information_permission
page_completion
delta_token_validity
full_sync_or_delta_sync_mode
primary_group_handling for AD
permission error state
```

##### Absence and omission semantics (4.5.1)

| Source state | Cadastre mapping |
| --- | --- |
| Empty membership list with hidden membership permission missing | `permission_limited`, not `not_observed`. |
| Limited-info row | Presence of membership may be known; member detail is `unknown`. |
| Delta token reset | `partial_unknown_gap` until full resync. |
| AD `memberOf` missing primary group | `partial_known_gap`. |
| Source unavailable | `source_unavailable`. |
| Direct membership query for transitive claim | `not_authoritative_for_absence`. |

##### Confirmed findings versus inferences (4.5.1)

| Type | Finding | Evidence | Cadastre implication |
| --- | --- | --- | --- |
| Confirmed source fact | Microsoft Graph membership APIs can require hidden membership permission and delta queries use state tokens. | Microsoft docs.[^26] | Coverage must include permission and delta validity. |
| Confirmed source fact | AD `memberOf` excludes primary group; primary group is represented separately. | Microsoft docs.[^26] | AD membership coverage requires primary-group handling. |
| Cadastre applicability inference | Hidden membership omission is not absence. | PRD omission and coverage rules.[^4][^10] | Map to `permission_limited`. |

##### Concepts that must not transfer (4.5.1)

| Source concept | Why unsafe for Cadastre | Required Cadastre handling |
| --- | --- | --- |
| Microsoft Graph limited-info row as full member detail | Permission may hide fields. | Membership presence and detail quality separate. |
| Hidden membership omission as absence | Permission can hide membership. | Require hidden-membership permission. |
| AD `memberOf` as complete membership | Primary group is excluded. | Require `primaryGroupID`. |
| Delta token as coverage | Token is progress state. | Page/delta coverage profile required. |

#### 4.5.2 Kubernetes and OpenTelemetry Entity Data Model

Kubernetes distinguishes object names and UIDs. Names are scoped and reusable; UIDs identify object occurrences. Labels and annotations are user-provided metadata. OpenTelemetry distinguishes identifying attributes from descriptive attributes and treats identity as minimally sufficient within an entity model.[^27]

| Source concept | Cadastre handling |
| --- | --- |
| Kubernetes UID | Strong for same Kubernetes object occurrence in cluster scope; not host identity by itself. |
| Kubernetes name | Selector only unless paired with UID/generation and scope. |
| Label/annotation | Descriptive or candidate selector only. |
| OTel identifying attribute | Useful evidence-class pattern; not automatic Cadastre identity. |
| OTel descriptive attribute | Not identity authority. |

Concepts that must not transfer: Kubernetes name as durable identity; labels as identity; telemetry entity identity as Cadastre canonical identity.

### 4.6 DNS, DHCP, IPAM, flow, and host telemetry

#### 4.6.1 DNS TTL

RFC 1035 defines TTL as a cache lifetime interval before consulting the source again. TTL expiry means cached data should expire; it does not mean the DNS record was deleted or that the name no longer resolves.[^28]

| DNS state | Cadastre mapping |
| --- | --- |
| DNS answer observed | `dns_resolution_observation` presence. |
| TTL active | Freshness window for that answer. |
| TTL expired | `stale` or expired for current graph, not deletion. |
| Missing DNS answer from non-authoritative query | `unknown`. |
| NXDOMAIN from authoritative source with coverage | Candidate explicit negative only if source contract permits. |

#### 4.6.2 DHCP lease

DHCP leases assign network parameters for a lease interval. Lease expiry means the assignment is no longer current under that lease unless renewed. It does not prove host deletion, durable host identity, or absence from the network.[^28]

| DHCP state | Cadastre mapping |
| --- | --- |
| Active lease | IP assignment fact within lease interval. |
| Expired lease | Stale/expired assignment, not host absence. |
| DHCP scope unavailable | `scope_unavailable`. |
| Missing lease outside authoritative DHCP scope | `unknown`. |
| Static IPAM allocation | Intended-state assignment, not observed host identity. |

#### 4.6.3 Zeek flow logs

Zeek `conn.log` uses a `uid` to correlate related activity in other logs. A missing flow does not mean no communication occurred unless flow logging coverage is complete for sensor placement, time window, protocols, and drop behavior.[^28]

| Flow state | Cadastre mapping |
| --- | --- |
| Flow observed | `observed_network_flow_fact` candidate. |
| Missing flow | `unknown` by default. |
| Sensor offline | `source_unavailable` or `scope_unavailable`. |
| Packet loss or capture gap | `partial_known_gap` or `partial_unknown_gap`. |
| Flow UID | Correlation ID only, not identity. |

#### 4.6.4 osquery differential results

osquery scheduled query logging can produce differential added/removed results between runs. Differential removal is not necessarily source absence unless the query scope, schedule, table semantics, and host visibility are complete.[^28]

| osquery state | Cadastre mapping |
| --- | --- |
| Added row | Presence observation. |
| Removed row | Candidate change evidence; not automatic deletion. |
| Query failed | `unknown` plus diagnostic. |
| Host offline | `source_unavailable` or stale. |
| Schedule gap | `partial_known_gap`. |

### 4.7 Cloud inventory and source-history systems

#### 4.7.1 Google Cloud Asset Inventory

Google Cloud Asset Inventory keeps asset metadata history for a bounded time window and can query history within that window. Current official documentation describes history windows in the five-week or 35-day range depending on page and operation wording.[^29]

#### 4.7.2 AWS Config

AWS Config records supported resource configuration changes and keeps configuration history under a retention period that can be defaulted or configured within documented bounds.[^29]

#### 4.7.3 Azure Resource Graph change analysis

Azure Resource Graph change analysis exposes a bounded change-history window. It is useful as source-native history evidence, not Cadastre retention.[^29]

| Source concept | Cadastre handling |
| --- | --- |
| Cloud asset presence | Presence observation when source authority allows. |
| Cloud delete event | Candidate retraction only with source authority and coverage. |
| Source history window | `SourceHistoryRetentionProfile`, not Cadastre retention. |
| Query outside source history window | `source_history_window_expired`. |
| No history result | `unknown` unless source proves complete coverage for that window. |

Concepts that must not transfer: cloud inventory history window as Cadastre retention; cloud config recorder time as fact valid time; missing history outside supported window as no change.

### 4.8 Graph and reachability systems as boundary references

#### 4.8.1 Batfish and provider reachability analyzers

Batfish performs modeled network configuration analysis and can answer reachability and traceroute-style questions over snapshots. AWS Reachability Analyzer performs static configuration analysis and does not send packets. Google Connectivity Tests distinguishes configuration analysis and live data-plane analysis. Azure Network Watcher decomposes diagnostics into IP Flow Verify, Effective Security Rules, Next Hop, and Connection Troubleshoot.[^31]

These tools are useful to define non-implication rules:

| Tool output | Cadastre interpretation |
| --- | --- |
| Modeled reachable path | Analysis artifact only by default. |
| Modeled unreachable | Negative only within declared supported scope and complete evidence. |
| Representative path | Example, not exhaustive path enumeration. |
| Unsupported resource | `unsupported`, not reachable/unreachable. |
| Permission gap | `permission_limited`. |
| Live probe success | Probe evidence only, not source completeness. |
| Live probe failure | `unknown` or diagnostic unless source contract permits stronger state. |

#### 4.8.2 BloodHound and OpenGraph

BloodHound/OpenGraph are relevant only to graph semantics and non-implication. Traversable graph edges are pathfinding eligibility, not Cadastre source authority. Source-kind deletion and external graph endpoint matching must not define Cadastre identity, cleanup, or absence. The PRD already rejects graph state, external graph traversability, relationship findings, collector output, and source-kind deletion as authority unless a Cadastre contract maps them.[^2]

## 5. Cross-source comparison

### 5.1 Authority dimension matrix

Legend: `Yes` means potentially authoritative when the active Cadastre profile permits it. `No` means forbidden by default. `Profile only` means the signal is not self-authorizing and must be combined with the named Cadastre authority profile. `Health only` means operational metadata only.

| Signal or artifact | Presence authority | Absence authority | Retraction authority | Cleanup authority | Coverage authority | Staleness authority | Cadastre default |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| Source record presence | Profile only | No | No | No | No | Profile only | Candidate observation. |
| Missing row | No | No | No | No | No | No | `unknown` or no-op. |
| Missing field | No | No | No | No | No | No | Field omission mapping. |
| Explicit negative observation | Profile only | Profile only | Profile only | No | No | Profile only | Candidate `not_observed` only with authority, completeness, and coverage. |
| Scan fixed state | Profile only | Profile only | Profile only | No | No | Scan-end policy | Candidate negative only with scan coverage. |
| Scanner reopened/resurfaced state | Profile only | No | Profile only | No | No | Scan-end policy | Positive correction candidate. |
| Control pass/fail | Profile only | No | No | No | Requires coverage | Evaluation-time policy | Candidate control fact. |
| Control unknown/error/notchecked | No | No | No | No | No | No | `unknown` or diagnostic. |
| API cursor exhaustion | No | No | No | No | Profile only | No | Candidate completeness evidence only. |
| Delta token | No | No | No | No | No | No | Progress state only. |
| CDC offset | No | No | No | No | No | No | Replay state only. |
| CDC heartbeat | No | No | No | No | No | Health only | Liveness only. |
| Schema history | No | No | No | No | No | No | Parsing/replay dependency. |
| Stream watermark | No | No | No | No | No | No | Progress only. |
| Queue drain | No | No | No | No | No | No | Ingestion lineage only. |
| Ack success | No | No | No | No | No | No | Delivery-only evidence. |
| Provenance closure | No | No | No | No | No | No | Lineage-only evidence. |
| dbt freshness | No | No | No | No | No | Health only | Freshness health only. |
| Destination stale delete | No | No | No | No | No | No | Diagnostic-only rejection. |
| Graph index state | No | No | No | No | No | Derived-view only | Derived read-model state. |
| Source history window | Profile only | No | Profile only | No | No | Profile only | Bounded source-history input. |
| DNS TTL expiry | No | No | No | No | No | Yes for DNS answer | Stale cached answer, not deletion. |
| DHCP lease expiry | No | No | No | No | No | Yes for lease assignment | Expired assignment, not host absence. |
| Flow log absence | No | No | No | No | No | No | `unknown`. |
| Live probe success/failure | No | No | No | No | No | Health/probe only | Non-authoritative probe. |

### 5.2 Source-category authority and absence matrix

| Fact category | Presence authority default | Absence may be meaningful only when | Staleness basis | Unsafe default |
| --- | --- | --- | --- | --- |
| Vulnerability presence | Vulnerability scanner or EDR source through active `SourceAuthorityProfile`. | Authenticated scan where required, target included, plugin/check included, scan complete, no partial gap, current `CoverageAssertion`. | Scan end time unless profile chooses source detection time. | Treating missing finding or `fixed` as absence. |
| Vulnerability fixed or absent | Scanner `fixed` or explicit negative only through profile. | Same as above plus absence semantics row. | Scan end time and staleness policy. | Fixed without scan coverage. |
| Control pass/fail/unknown | Configuration integrity or endpoint management through profile. | Coverage assertion for rule/check, source scope, target, and evaluation time. | Evaluation end time. | Mapping unknown/error/notchecked to pass/fail/absence. |
| Endpoint management state | Endpoint management source through profile. | Device enrollment scope and permissions complete. | Device last sync or collection time by policy. | Treating stale device as deleted. |
| Directory group membership | Identity directory through profile. | Group scope, direct/transitive mode, hidden membership permission, full page/delta coverage, AD primary group handling. | Directory observation or collection time. | Missing membership row as absence. |
| DNS resolution | DNS telemetry through source contract. | Authoritative DNS source, query scope, time window, and negative response semantics proven. | TTL or configured DNS staleness threshold. | TTL expiry as deletion. |
| DHCP/IPAM assignment | DHCP/IPAM or endpoint telemetry through profile. | DHCP scope authority, lease state, source availability, and time window proven. | Lease expiry or collection time. | Lease expiry as host absence. |
| Firewall/network flow | Network firewall/gateway or endpoint telemetry presence only. | Absence generally not meaningful unless a future complete capture contract exists. | Flow observation time and sensor staleness. | Missing flow as no communication. |
| Cloud asset inventory | Cloud inventory source through profile. | Source service supports complete scope enumeration, permissions, and current history window. | Asset update time, snapshot time, or collection time by policy. | History window as Cadastre retention. |
| Source history/change history | Source-native history API through profile. | Query within supported history window and scope. | Source-native event time if provided. | Missing history outside window as no change. |

### 5.3 Staleness basis matrix

| Source/fact type | Staleness input | Required fields | Default at expiry | Must not imply |
| --- | --- | --- | --- | --- |
| Vulnerability finding | `scan_end_time` | scan ID, policy, target, plugin/check set, scan end, coverage ID | `stale` | Vulnerability absent or remediated. |
| Vulnerability absence | `scan_end_time` plus coverage expiry | coverage ID, authority row, staleness row | `unknown_staleness` or `stale` | Current absence. |
| Control result | evaluation end time | control ID, rule polarity, check set, coverage ID | `stale` | Pass or fail after expiry. |
| Directory membership | source observation time or collection time | group, member scope, permission state, delta/full sync state | `stale` | Non-membership. |
| DNS answer | DNS TTL or source policy | answer, TTL, observed time, source authority | `stale` or `expired_for_current_graph` | Record deletion. |
| DHCP lease | lease end time | scope, lease start/end, server authority | `expired_for_current_graph` | Host deletion. |
| Flow observation | flow end time or sensor observation time | sensor, capture point, flow time, role evidence | `stale` | No communication. |
| Cloud asset | source update time or snapshot time | source scope, asset ID, history window | `stale` | Asset deletion unless delete event authoritative. |
| CDC row | source row event time only if source emits it | table, operation, schema history, offset | `unknown_staleness` if no source event time | No change. |
| Graph derived view | graph apply/rebuild time | `DerivedViewState` | `DERIVED_VIEW_LAG_ERROR` or stale label | Source fact staleness. |
| dbt freshness | freshness artifact time | artifact ID, source, generated time | health degradation | Coverage or absence. |
| OpenLineage run | event time | run/job/dataset IDs | lineage stale | Source completeness. |

### 5.4 Coverage dimensions matrix

| Coverage class | Required dimensions | Default missing-dimension behavior | Required evidence refs |
| --- | --- | --- | --- |
| Vulnerability scan | scanner instance, scan ID, target scope, credential/auth status, plugin/check set, port/service scope, scan start/end, failed targets, exclusions, publish state | `not_authoritative_for_absence` | scan config, target inventory, scan result, credential result, plugin manifest |
| Control evaluation | control ID, rule polarity, target scope, evaluator version, check set, error states, applicability rules, evaluation window | `unknown` | rule artifact, evaluation result, coverage assertion |
| Endpoint inventory | management tenant, device scope, enrollment state, permission scope, last sync, collection method | `unknown` | source call receipt, permission evidence |
| Directory membership | tenant/domain, group, member type, direct/transitive mode, hidden-membership permission, page completion, delta validity, AD primary-group handling | `permission_limited` or `partial_known_gap` | membership API response, permission proof, delta/full sync state |
| DNS | authoritative source, query name/type/class, resolver/zone scope, TTL, negative response semantics, collection window | `unknown` | DNS answer, zone/source authority, TTL |
| DHCP/IPAM | server or IPAM scope, subnet/pool, lease interval, reservation/static allocation class, source availability | `unknown` | lease record, scope config, source receipt |
| Flow logging | sensor, interface/tap, capture window, protocol support, packet loss/drop state, aggregation mode, NAT/proxy role evidence | `unknown` | sensor health, flow record, role evidence |
| Cloud inventory | provider account/project/subscription, region, resource type, permission scope, history window, full enumeration state | `unknown` | inventory export/query, permission evidence, source history policy |
| Source history | source history window, query time range, object scope, permissions, retention policy, unsupported types | `source_history_window_expired` or `unknown` | history API response, retention policy, source docs snapshot |

### 5.5 Omission-state mapping matrix

| Source state | Cadastre field omission state | Cadastre fact absence outcome | Default behavior |
| --- | --- | --- | --- |
| Source field missing | `absent` | No absence | Preserve missing-field state. |
| Source field present empty string | `empty` | No absence | Preserve empty value. |
| Source field present empty array | `empty` unless field contract says negative | No absence by default | Preserve collection shape. |
| Source field present null | `explicit_null` | No absence | Preserve null. |
| Source field malformed | `malformed` | No absence | Preserve raw value evidence and diagnostic. |
| Source field not applicable by record type | `not_applicable` | `not_applicable` | Do not emit negative fact. |
| Explicit negative observation | `not_observed` candidate | `not_observed` only with authority, completeness, coverage, staleness | Otherwise `unknown`. |
| Undocumented enum | `unknown` | `unknown` | Preserve raw enum. |
| Permission withheld | `unknown` | `permission_limited` | Block absence. |
| Redacted by source | `unknown` | `permission_limited` or `unknown` | Block absence. |
| Unsupported resource/check | `not_applicable` or `unknown` by contract | `unsupported` | Block absence. |
| Source unavailable | Not a field omission | `source_unavailable` | Block absence/retraction/cleanup. |
| Source declared partial | Not a field omission | `partial_known_gap` or `partial_unknown_gap` | Block absence unless profile narrows safely. |
| Not checked | `unknown` or `not_applicable` by result vocabulary | `unknown` | Block pass/fail/absence. |
| Error | `unknown` plus diagnostic | `unknown` or `source_unavailable` | Block absence. |
| Expired TTL | Not field omission | `stale` | Do not emit deletion. |
| Expired DHCP lease | Not field omission | `expired_for_current_graph` | Do not emit host absence. |
| Missing source row | Not field omission | `unknown` | No-op unless total absence algorithm authorizes. |

## 6. Required PRD improvement analysis

| Priority | Finding | Target PRD contract | Proposed change | Why it improves determinism | Acceptance criterion |
| ---: | --- | --- | --- | --- | --- |
| 1 | Authority rows must be predicate and scope specific. | `SourceAuthorityProfile` | Require rows keyed by fact type, predicate, source category, source dataset, source instance override, subject scope, object scope, and absence authority. | Prevents broad “source is authoritative” interpretations. | Missing row fails with `SOURCE_AUTHORITY_PROFILE_ERROR`. |
| 2 | Staleness must be first-class. | `SourceStalenessPolicy` | Add policy with source/fact time-input precedence, expiry behavior, stale output state, and graph effect. | Prevents conflicting use of scan end, TTL, lease, sync time, or graph time. | Same record and policy always produce same staleness state. |
| 3 | Coverage dimensions must be profile-driven. | `CoverageAssertion` | Add `CoverageDimensionProfile` per source category. | Prevents incomplete coverage from authorizing absence. | Absence fails when any required dimension is missing. |
| 4 | Completeness evidence must aggregate deterministically. | `SourceCompletenessEvidenceRow` | Add blocking precedence: unsafe evidence rows dominate candidate evidence rows. | Prevents weak evidence combination from authorizing absence. | `source_declared_partial` blocks absence despite cursor exhaustion. |
| 5 | Permission-limited states need explicit outcomes. | `CollectionMethodVisibilityProfile` | Add `permission_limited`, `hidden_membership_missing`, `limited_info_row`, `credential_scope_missing`, and `redacted_by_source` states. | Prevents permission omissions from becoming absence. | Hidden group membership without permission cannot emit nonmembership. |
| 6 | Progress signals need a total interpretation table. | `ProgressSignalInterpretationPolicy` | Add rows for cursor, delta token, CDC offset, heartbeat, watermark, ack, queue drain, provenance, destination cleanup, dbt freshness, graph index state. | Prevents accidental authority leakage. | Every listed progress artifact maps to an authority limit. |
| 7 | Control result vocabularies must be explicit. | `ControlResultMappingRow` | Define OVAL/XCCDF result mapping, polarity, unknown/error/notchecked behavior, and applicability. | Prevents result polarity and unknown-state drift. | OVAL `unknown` always maps to Cadastre `unknown`. |
| 8 | Vulnerability absence requires scanner dimensions. | `CoverageAssertion` | Add vulnerability scan dimensions: target, auth, plugin/check, port/service, failed targets, exclusions, scan window, publish state. | Prevents scanner `fixed` from becoming automatic absence. | Fixed without plugin coverage emits `unknown`. |
| 9 | Directory membership coverage needs visibility dimensions. | `CollectionMethodVisibilityProfile` | Add hidden membership, limited-info rows, transitive/direct mode, AD primary-group coverage, delta validity. | Prevents hidden omissions and AD primary-group gaps. | AD `memberOf` alone cannot prove absence from primary group. |
| 10 | DNS, DHCP, and flow need staleness policies. | `SourceStalenessPolicy` | Add TTL, lease, flow observation, sensor health, and capture-window policies. | Prevents TTL/lease/missing-flow negative claims. | Expired TTL maps to stale, not deletion. |
| 11 | Cloud history windows need retention profiles. | `SourceHistoryRetentionProfile` | Add source-native history window, query bounds, outside-window behavior, and retention refs. | Prevents external history windows from replacing Cadastre retention. | Query outside source window returns `source_history_window_expired`. |
| 12 | Total absence authorization must be algorithmic. | `DeriveAbsenceOrUnknown` | Add deterministic algorithm with explicit rejections for missing rows and progress-only signals. | Prevents ad hoc absence derivation. | Missing row without coverage emits `unknown` or no-op. |
| 13 | UI/API states must distinguish stale, unknown, not applicable, not observed, conflicted, permission-limited. | Asset search/detail, graph, evidence, exports | Add response-state vocabulary and default display behavior. | Prevents user-facing collapse of non-negative states. | Compliance export includes unknown/error states separately. |
| 14 | Health metrics must protect contracts. | Operational health | Add warning/failure thresholds for stale, unsafe completeness, missing coverage, permission-limited visibility, token reset, CDC gaps, graph lag, scanner auth failure, and forbidden-output attempts. | Makes unsafe states visible and testable. | Each metric has threshold, action, and protected contract. |
| 15 | Version manifests must include new policies. | `VersionManifest` | Add `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `CoverageDimensionProfile`, `ControlResultMappingRow`, `SourceHistoryRetentionProfile`. | Replay uses the same authority and staleness rules. | Replay rejects missing or mismatched policy ref. |

## 7. Required deterministic algorithms

### 7.1 `EvaluateSourceAuthority`

Inputs:

```text
candidate fact
source observation
fact type
predicate
source category
source dataset
source scope
SourceAuthorityProfile
source instance overrides
conflict policy
```

Output:

```text
authority decision
reason code
selected authority row
conflict disposition
deterministic checksum input
```

Pseudocode:

```text
function EvaluateSourceAuthority(input):
    checksum_input = canonical_json(input excluding non-deterministic runtime fields)

    rows = SourceAuthorityProfile.rows where
        row.lifecycle_status = active
        and row.fact_type = input.fact_type
        and row.predicate = input.predicate
        and row.source_category = input.source_category
        and row.source_dataset = input.source_dataset
        and row.source_scope matches input.source_scope
        and row.valid_time_range contains candidate_fact.valid_time

    rows = apply_source_instance_overrides(rows, source_instance_overrides)

    if rows is empty:
        return {
            authority_decision: not_authoritative,
            reason_code: SOURCE_AUTHORITY_PROFILE_ERROR,
            selected_authority_row: null,
            conflict_disposition: no_decision,
            deterministic_checksum_input: checksum_input
        }

    if any row.authority_status = authority_unresolved:
        return {
            authority_decision: blocked,
            reason_code: AUTHORITY_UNRESOLVED,
            selected_authority_row: deterministic_first(rows),
            conflict_disposition: no_decision,
            deterministic_checksum_input: checksum_input
        }

    eligible_rows = rows where
        row.observation_state_allowed contains source_observation.state
        and row.permission_state_allowed contains source_observation.permission_state
        and row.coverage_requirement is satisfied or row.coverage_required = false
        and row.staleness_requirement is satisfied or row.staleness_allowed = true

    if eligible_rows is empty:
        return {
            authority_decision: not_authoritative,
            reason_code: SOURCE_NOT_ELIGIBLE_FOR_PREDICATE,
            selected_authority_row: deterministic_first(rows),
            conflict_disposition: no_decision,
            deterministic_checksum_input: checksum_input
        }

    selected = sort eligible_rows by:
        1. row.authority_rank ascending
        2. row.source_instance_specificity descending
        3. row.scope_specificity descending
        4. row.profile_row_id lexical ascending
      take first

    competing = eligible_rows excluding selected where
        row.authority_rank = selected.authority_rank
        and row.source_instance_specificity = selected.source_instance_specificity
        and row.scope_specificity = selected.scope_specificity

    if competing not empty and facts_conflict(candidate_fact, competing):
        disposition = apply_conflict_policy(conflict_policy, selected, competing)
        if disposition = block:
            return {
                authority_decision: conflicted,
                reason_code: AUTHORITATIVE_SOURCE_CONFLICT,
                selected_authority_row: selected,
                conflict_disposition: conflicted,
                deterministic_checksum_input: checksum_input
            }

    return {
        authority_decision: authoritative,
        reason_code: AUTHORITY_ROW_SELECTED,
        selected_authority_row: selected,
        conflict_disposition: disposition or none,
        deterministic_checksum_input: checksum_input
    }
```

### 7.2 `EvaluateStaleness`

Inputs:

```text
source observation
fact type
source category
SourceStalenessPolicy
valid or known time
scan end time
last sync time
TTL or lease interval
collection window
source snapshot time
graph apply time if relevant
```

Output:

```text
fresh
stale
expired_for_current_graph
unknown_staleness
reason code
```

Pseudocode:

```text
function EvaluateStaleness(input):
    policy = lookup active SourceStalenessPolicy by
        fact_type, source_category, source_dataset, predicate, source_scope

    if policy missing:
        return unknown_staleness, SOURCE_STALENESS_POLICY_MISSING

    candidate_times = []

    for basis in policy.time_basis_precedence:
        if basis = source_event_time and source_observation.source_event_time_quality = source_time_valid:
            candidate_times.append(source_observation.source_event_time)
        if basis = scan_end_time and input.scan_end_time present:
            candidate_times.append(input.scan_end_time)
        if basis = ttl_expiry and ttl_or_lease_interval.kind = dns_ttl:
            candidate_times.append(source_observation.observed_at + ttl_or_lease_interval)
        if basis = lease_expiry and ttl_or_lease_interval.kind = dhcp_lease:
            candidate_times.append(ttl_or_lease_interval.lease_end)
        if basis = source_snapshot_time and source_snapshot_time present:
            candidate_times.append(source_snapshot_time)
        if basis = collection_window_end and collection_window.end present:
            candidate_times.append(collection_window.end)
        if basis = last_sync_time and last_sync_time present:
            candidate_times.append(last_sync_time)

    if candidate_times empty:
        return unknown_staleness, STALENESS_INPUT_UNAVAILABLE

    selected_time = first candidate_times by policy.time_basis_precedence

    if policy.forbidden_basis contains graph_apply_time:
        ignore graph_apply_time for source staleness

    expiry_time = selected_time + policy.max_age_seconds unless selected_time already represents expiry

    if selected_time represents ttl_expiry or lease_expiry:
        expiry_time = selected_time

    if current_time <= expiry_time:
        return fresh, FRESH_WITHIN_POLICY

    if policy.expiry_effect = expired_for_current_graph:
        return expired_for_current_graph, EXPIRED_FOR_CURRENT_GRAPH

    return stale, STALE_BY_POLICY
```

Rules:

```text
graph_apply_time may determine derived-view lag only.
dbt freshness time may determine health only unless SourceStalenessPolicy explicitly maps it.
CDC offset, heartbeat, schema-history time, queue drain time, ack time, and provenance event time must not be staleness inputs unless the policy explicitly maps the source state and authority limit.
```

### 7.3 `EvaluateCoverageAssertion`

Inputs:

```text
candidate subject
fact type
predicate
source scope
source dataset
coverage assertion
coverage profile
time window
required dimensions
```

Output:

```text
coverage valid or invalid
missing dimensions
stale dimensions
permission gaps
skipped or unsupported checks
reason code
```

Pseudocode:

```text
function EvaluateCoverageAssertion(input):
    profile = coverage_profile.active_row_for(fact_type, predicate, source_dataset, source_scope)

    if profile missing:
        return invalid, all required_dimensions, [], [], [], COVERAGE_PROFILE_MISSING

    required = profile.required_dimensions union input.required_dimensions
    present = coverage_assertion.dimension_map.keys

    missing = required - present

    stale = []
    permission_gaps = []
    skipped_or_unsupported = []

    for dim in required intersect present:
        value = coverage_assertion.dimension_map[dim]

        if value.status in [permission_limited, redacted, hidden_membership_missing]:
            permission_gaps.append(dim)

        if value.status in [skipped, unsupported, not_checked, not_evaluated]:
            skipped_or_unsupported.append(dim)

        if value.valid_from > time_window.start or value.valid_to < time_window.end:
            stale.append(dim)

        if value.staleness_state in [stale, expired_for_current_graph, unknown_staleness]:
            stale.append(dim)

    if missing not empty:
        return invalid, missing, stale, permission_gaps, skipped_or_unsupported, COVERAGE_DIMENSION_MISSING

    if permission_gaps not empty:
        return invalid, missing, stale, permission_gaps, skipped_or_unsupported, COVERAGE_PERMISSION_LIMITED

    if skipped_or_unsupported not empty:
        return invalid, missing, stale, permission_gaps, skipped_or_unsupported, COVERAGE_UNSUPPORTED_OR_SKIPPED

    if stale not empty:
        return invalid, missing, stale, permission_gaps, skipped_or_unsupported, COVERAGE_STALE

    if coverage_assertion.subject_scope does not cover candidate_subject:
        return invalid, missing, stale, permission_gaps, skipped_or_unsupported, COVERAGE_SUBJECT_OUT_OF_SCOPE

    return valid, [], [], [], [], COVERAGE_VALID
```

### 7.4 `DeriveAbsenceOrUnknown`

Inputs:

```text
candidate
authority profile
completeness receipt
completeness profile
coverage assertion
staleness policy
source-state mapping
field-quality mapping
```

Output:

```text
not_observed
unknown
not_applicable
stale
source_unavailable
scope_unavailable
partial_known_gap
partial_unknown_gap
not_authoritative_for_absence
no-op
reason code
evidence refs
checksum
```

Pseudocode:

```text
function DeriveAbsenceOrUnknown(input):
    checksum = canonical_json(input)

    if candidate.source_state in [
        missing_row,
        zero_live_query_rows,
        expired_dns_ttl,
        expired_dhcp_lease,
        queue_drain,
        ack_success,
        destination_cleanup,
        stream_watermark,
        cdc_heartbeat,
        dbt_freshness_only,
        graph_index_state,
        provenance_closure
    ]:
        permitted = active_profile_row_explicitly_permits_absence_from_source_state(
            input.authority_profile,
            input.completeness_profile,
            input.source_state_mapping,
            candidate.source_state
        )
        if not permitted:
            return {
                outcome: no-op if candidate.source_state = missing_row else unknown,
                reason_code: SOURCE_STATE_NOT_AUTHORIZED_FOR_ABSENCE,
                evidence_refs: collect_refs(input),
                checksum: checksum
            }

    authority = EvaluateSourceAuthority(...)
    if authority.authority_decision != authoritative:
        return not_authoritative_for_absence, authority.reason_code, refs, checksum

    completeness = EvaluateSourceCompleteness(completeness_receipt, completeness_profile)
    if completeness.state = source_unavailable:
        return source_unavailable, SOURCE_UNAVAILABLE_BLOCKS_ABSENCE, refs, checksum
    if completeness.state = scope_unavailable:
        return scope_unavailable, SCOPE_UNAVAILABLE_BLOCKS_ABSENCE, refs, checksum
    if completeness.state = partial_known_gap:
        return partial_known_gap, PARTIAL_KNOWN_GAP_BLOCKS_ABSENCE, refs, checksum
    if completeness.state = partial_unknown_gap:
        return partial_unknown_gap, PARTIAL_UNKNOWN_GAP_BLOCKS_ABSENCE, refs, checksum
    if completeness.state in [not_attempted, not_applicable, not_authoritative_for_absence]:
        return not_authoritative_for_absence, COMPLETENESS_NOT_AUTHORIZED_FOR_ABSENCE, refs, checksum

    if completeness.state not in [complete, empty_complete]:
        return unknown, COMPLETENESS_STATE_UNSAFE, refs, checksum

    coverage = EvaluateCoverageAssertion(...)
    if coverage.coverage_valid = false:
        return unknown, coverage.reason_code, refs, checksum

    stale = EvaluateStaleness(...)
    if stale.state = unknown_staleness:
        return unknown, UNKNOWN_STALENESS_BLOCKS_ABSENCE, refs, checksum
    if stale.state in [stale, expired_for_current_graph]:
        return stale.state, STALE_SOURCE_BLOCKS_CURRENT_ABSENCE, refs, checksum

    mapped = source_state_mapping.lookup(candidate.source_state)

    if mapped = not_applicable:
        return not_applicable, SOURCE_STATE_NOT_APPLICABLE, refs, checksum

    if mapped = explicit_negative_observation:
        return not_observed, ABSENCE_AUTHORIZED, refs, checksum

    if mapped = missing_row and completeness.state = empty_complete:
        return not_observed only if completeness_profile.empty_complete_authorizes_absence = true
        else return unknown, EMPTY_COMPLETE_NOT_ABSENCE_AUTHORIZED, refs, checksum

    return unknown, SOURCE_STATE_MAPPING_UNKNOWN, refs, checksum
```

## 8. Required UI and API implications

| Surface | Default behavior | Error behavior |
| --- | --- | --- |
| Asset search | Include assets with stale, unknown, conflicted, and permission-limited facts unless filters exclude them. Show state counts separately. | Invalid filter state returns `VALIDATION_ERROR`; graph lag beyond policy returns `DERIVED_VIEW_LAG_ERROR` for graph-derived filters. |
| Asset detail | Show stale facts with `assertion_state = stale`; show unknown and permission-limited facts as explicit non-negative states; show `not_observed` only when authorized. | Missing evidence refs return `LINEAGE_ERROR`; raw permission failure redacts payload but keeps metadata. |
| Graph query | Use graph read model only with `DerivedViewState`; distinguish `source_stale` from `derived_view_stale`; pathfinding uses only allowed traversal classes. | Excess graph lag returns `DERIVED_VIEW_LAG_ERROR` unless stale reads are allowed for query class. |
| Evidence drillback | Show authority row, completeness receipt, coverage assertion, staleness decision, and source-state mapping for absence or stale facts. | Missing authority/completeness/coverage refs return `LINEAGE_ERROR` or `AUTHORITY_EVIDENCE_MISSING`. |
| Health dashboards | Show unsafe completeness, stale coverage, permission-limited visibility, token reset, CDC gaps, graph lag, scanner auth failure, and forbidden-output attempts. | Health unknown when required telemetry missing; production writes fail when protected contract threshold is exceeded. |
| Compliance exports | Export pass, fail, unknown, error, not checked, not applicable, stale, permission-limited, and not observed separately. | Stale graph-derived exports are forbidden by default; return `DERIVED_VIEW_LAG_ERROR`. |
| Audit evidence queries | Use lakehouse-backed evidence by default; include version manifest and policy IDs. | Reject when required `VersionManifest`, `SourceAuthorityProfile`, `CoverageAssertion`, or `SourceStalenessPolicy` is missing. |
| Analysis rule outputs | Mark findings as analysis outputs, not gold facts; include stale/unknown/coverage state in rule result. | Rule execution fails if graph profile or coverage state is incompatible with `RuleGraphCompatibilityMatrix`. |

## 9. Required health metrics

Default threshold convention:

```text
warning threshold = first value that indicates production correctness risk.
failure threshold = value that requires blocking protected output or marking protected feature unhealthy.
default action = fail closed for production-affecting outputs, except exploratory views may show warning labels when policy permits.
```

| Metric | Warning threshold | Failure threshold | Default action | Protected contract |
| --- | ---: | ---: | --- | --- |
| Source freshness lag | > `0.8 * SourceStalenessPolicy.max_age_seconds` | > `SourceStalenessPolicy.max_age_seconds` | Mark facts stale; block current absence. | `SourceStalenessPolicy` |
| Source completeness unsafe states | Any unsafe state for coverage-sensitive source | Unsafe state used for absence attempt | Reject absence/retraction/cleanup/watermark. | `SourceCompletenessProfile` |
| Coverage assertion missing or stale | Any missing required dimension | Coverage-dependent fact attempted | Reject fact or absence. | `CoverageAssertion` |
| Permission-limited visibility | Any permission-limited dimension | Permission-limited absence attempt | Map to `permission_limited`; block absence. | `CollectionMethodVisibilityProfile` |
| Delta token reset | Any token reset | Absence attempted before full resync | Block absence; require full sync. | `StageStateRecord` |
| CDC schema-history missing | Any missing required schema history | CDC parse/replay attempted | Quarantine or reject CDC output. | CDC state contract |
| CDC heartbeat lag | > policy warning | > policy failure | Mark connector unhealthy; no absence. | CDC health |
| Cursor gap | Any skipped page or cursor discontinuity | Cursor advanced past gap | Block cursor commit and completeness. | `SourceCallPolicy` |
| Cursor commit failure | Any failure | State committed before raw/receipt persistence | Reject run output. | `StageStateRecord` |
| Destination cleanup rejection | Any attempted use | Any production absence from cleanup | Emit critical health event. | `ProgressSignalInterpretationPolicy` |
| Watermark-as-absence rejection | Any attempted use | Any emitted absence from watermark | Reject output. | `ProgressSignalInterpretationPolicy` |
| Stale absence attempt | Any stale source used for absence | Any stale absence emitted | Reject output. | `SourceStalenessPolicy` |
| Live probe forbidden-output attempt | Any live probe tries production output | Any write succeeds | Fail package validation. | Stage output permissions |
| Provenance-not-source-evidence rejection | Any attempted provenance-as-completeness | Any completeness authorized by provenance alone | Reject output. | `SourceCompletenessEvidenceRow` |
| Graph derived-view lag | > policy warning | > `DerivedViewLagPolicy.max_lag_seconds` | Label or reject by query class. | `DerivedViewLagPolicy` |
| Scanner authenticated-scan failure | Any required auth failure | Absence attempted for auth-required check | Block vulnerability absence. | `CoverageAssertion` |
| Scanner plugin/check coverage missing | Any missing required plugin/check | Absence attempted | Block vulnerability/control absence. | `CoverageDimensionProfile` |
| Directory hidden-membership permission missing | Any group requiring hidden membership | Membership absence attempted | Map to `permission_limited`. | `CollectionMethodVisibilityProfile` |
| DNS TTL stale rate | > 10% active DNS facts stale | > 25% active DNS facts stale or stale DNS used as current | Mark stale; block current graph edge if policy requires freshness. | DNS staleness |
| DHCP lease expiry without authoritative scope | Any expired lease outside authoritative scope used | Any host absence emitted | Reject absence; mark assignment expired. | DHCP staleness |
| Source history outside supported window | Any query outside window | Any no-change derived from outside-window query | Emit `source_history_window_expired`; block negative fact. | `SourceHistoryRetentionProfile` |

## 10. Concepts that should not be transferred

| External concept | Tempting interpretation | Required Cadastre interpretation | Reason |
| --- | --- | --- | --- |
| ServiceNow CMDB reconciliation winner | Cadastre truth | Source observation subject to Cadastre authority | CMDB reconciliation is source-local update policy. |
| Source update permission | Absence authority | Permission metadata only | Update permission does not prove coverage. |
| CloudQuery destination stale delete | Source absence | Diagnostic-only rejection | Destination state is not source scope. |
| Steampipe live zero-row result | Absence | Live probe result, no-op or unknown | Query may be filtered, incomplete, or permission-limited. |
| NiFi provenance closure | Source truth | Ingestion lineage only | Pipeline event, not source observation. |
| Queue drain | Completeness | Queue state only | Does not prove upstream source complete. |
| Data Prepper or sink acknowledgment | Source completeness | Delivery-only evidence | Sink delivery is not source coverage. |
| Flink/Beam watermark | Source completeness | Stream progress only | Late data may still exist; source scope not proven. |
| CDC heartbeat | No-change proof | Liveness only | Does not prove source table unchanged. |
| CDC offset | Fact time | Replay position only | Offset is log/progress state. |
| dbt freshness | Coverage | Health/freshness artifact only | Freshness is not subject/predicate coverage. |
| Scanner `fixed` | Automatic absence | Candidate negative only with scan coverage | Fixed state depends on scan scope, auth, checks, recency. |
| XCCDF scoring | Factual state | Compliance analysis output | Score is aggregation, not fact presence. |
| OVAL unknown | Absence | `unknown` | Unknown means insufficient/evaluation failure. |
| Microsoft Graph limited-info row | Full membership detail | Membership presence with unknown details | Permissions can hide object properties. |
| Hidden group membership omission | Absence | `permission_limited` | Permission can hide members. |
| AD `memberOf` | Complete group membership | Partial membership evidence | Primary group excluded. |
| Kubernetes name | Durable identity | Scoped selector only | Names can be reused. |
| DNS TTL expiry | Deletion | Stale cache/answer | TTL is cache lifetime. |
| DHCP lease | Host identity | Temporal IP assignment | Lease does not identify host durably. |
| Missing flow | No communication | `unknown` | Sensor/capture coverage may be incomplete. |
| Cloud inventory history window | Cadastre retention | Source-native query bound | External history window is not Cadastre replay retention. |
| Graph backend state | Source truth | Derived read-model state | Lakehouse remains system of record. |

## 11. Gaps, risks, stale assumptions, and unresolved questions

| Category | Gap or risk | Impact | Required owner or follow-up | Default safe behavior |
| --- | --- | --- | --- | --- |
| Product decision | Exact `SourceStalenessPolicy` defaults per source category are unresolved. | Implementations may choose different clocks. | Product owner plus source-domain owners. | `unknown_staleness`; block absence. |
| Product decision | Host ownership authority remains unresolved in PRD. | Owner facts may be emitted inconsistently. | Product governance. | No owner fact. |
| Source-specific | Scanner plugin/check coverage differs by product. | Unsafe vulnerability absence. | Vulnerability domain owner. | Require explicit coverage dimension; otherwise unknown. |
| Source-specific | Microsoft Graph delta token expiration varies by resource. | Delta gaps can be missed. | Directory source owner. | Full resync required after reset/expiry. |
| Source-specific | AD primary group handling often omitted. | False nonmembership. | Directory source owner. | `partial_known_gap`. |
| Source-specific | DNS negative response authority requires zone/source contract. | False DNS absence. | DNS owner. | `unknown`. |
| Source-specific | DHCP lease semantics vary by server/config. | False host absence or stale IP fact. | Network services owner. | Expired assignment only. |
| Source-specific | Flow logging capture points and loss are environment-specific. | False no-communication claims. | Network telemetry owner. | Missing flow is `unknown`. |
| External docs | Vendor docs may change after 2026-05-16. | Stale source assumptions. | Source owner before PRD amendment. | Re-verify official docs. |
| Scanner ambiguity | `fixed`, `mitigated`, `resolved`, `reopened`, `resurfaced` vary by product. | Wrong temporal correction. | Vulnerability domain owner. | Product-specific mapping rows required. |
| Control polarity | OVAL true/false and XCCDF pass/fail depend on rule polarity. | Inverted pass/fail facts. | Compliance domain owner. | Require `ControlResultMappingRow`. |
| History windows | Cloud inventory/history APIs have bounded retention. | False no-change outside window. | Cloud source owner. | `source_history_window_expired`. |
| Graph/UI | Derived graph stale reads may be mistaken for source stale facts. | User confusion and wrong exports. | API/UI owner. | Label or reject by query class. |
| Replay | New staleness and progress policies not included in manifest. | Replay divergence. | Platform owner. | Replay reject. |
| Lineage | OpenLineage/DataHub/OpenMetadata artifacts are tempting evidence substitutes. | Authority leakage. | Platform owner. | `lineage_only`. |
| Live probes | Source exploration zero-row results are operationally convenient. | False absence. | Adapter/tooling owner. | Non-authoritative only. |

## 12. Binary acceptance criteria

| ID | Acceptance criterion | Fails if |
| --- | --- | --- |
| AC-001 | A missing source row with no `SourceCompletenessReceipt` and no `CoverageAssertion` emits no absence fact and returns `unknown` or no-op. | Any `not_observed`, retraction, cleanup, or graph expiry is emitted. |
| AC-002 | A partial completeness receipt blocks absence, retraction, cleanup, and source watermark advancement. | Any completeness-sensitive effect occurs from `partial_known_gap` or `partial_unknown_gap`. |
| AC-003 | `source_unavailable` maps to `source_unavailable` and blocks negative facts. | Missing rows during outage become absence. |
| AC-004 | Permission-limited visibility maps to `permission_limited` and blocks absence. | Hidden, redacted, or limited rows are treated as complete absence. |
| AC-005 | Scanner `fixed` without target, auth, plugin/check, and scan coverage emits `unknown`. | It emits vulnerability absence. |
| AC-006 | Scanner reopened/resurfaced after fixed emits a new positive correction and closes or supersedes prior absence by known time. | It is ignored as a duplicate or leaves prior absence current. |
| AC-007 | XCCDF `unknown`, `error`, and `notchecked` map to non-positive, non-negative states. | They map to pass, fail, or absence without mapping row authorization. |
| AC-008 | OVAL `unknown`, `error`, and `not evaluated` map to `unknown` or not-checked outcomes. | They map to absence. |
| AC-009 | Microsoft Graph hidden membership without required permission cannot prove nonmembership. | Empty membership result emits absence. |
| AC-010 | Microsoft Graph delta token reset or equivalent delta reset blocks membership absence until full resync. | Delta reset scope emits absence before resync. |
| AC-011 | AD `memberOf` without `primaryGroupID` handling records a partial known gap. | It asserts complete group membership absence. |
| AC-012 | DNS TTL expiry marks the DNS resolution stale or expired for current graph only. | It emits DNS record deletion or host absence. |
| AC-013 | DHCP lease expiry expires the IP assignment interval only. | It emits host deletion or durable identity change. |
| AC-014 | No flow logs for a subject/time window map to `unknown` unless a future complete-flow contract explicitly authorizes absence. | It emits no-communication fact. |
| AC-015 | Destination stale delete is rejected as source evidence. | It emits absence, retraction, cleanup, or watermark advancement. |
| AC-016 | Live source probe zero rows cannot emit production raw, completeness, coverage, gold, or graph records. | Any production record is written. |
| AC-017 | Stream watermark cannot authorize source absence. | Watermark alone emits absence or cleanup. |
| AC-018 | CDC heartbeat cannot prove no change. | Heartbeat alone emits no-change, absence, cleanup, or retraction. |
| AC-019 | CDC schema history missing blocks CDC parsing/replay output for affected records. | CDC output proceeds without required schema-history ref. |
| AC-020 | Queue drain is lineage-only and cannot satisfy completeness. | Queue drain emits complete receipt or absence. |
| AC-021 | Ack success is delivery-only and cannot satisfy completeness. | Ack success authorizes absence, cleanup, or watermark. |
| AC-022 | Provenance closure is lineage-only and cannot satisfy source evidence or source completeness. | Provenance closure emits source truth. |
| AC-023 | A stale graph derived view returns `DERIVED_VIEW_LAG_ERROR` for compliance exports unless the deployment policy explicitly permits stale export. | Stale graph output is exported unlabeled. |
| AC-024 | Conflicting authoritative sources with equal priority emit `conflicted` and no silent winner unless conflict policy defines a deterministic winner. | One source silently overwrites the other. |
| AC-025 | Omission states `absent`, `empty`, `explicit_null`, `malformed`, `not_applicable`, `not_observed`, and `unknown` remain distinct in persisted silver and validation output. | Any two states are coerced into one value without a mapping row. |
| AC-026 | Missing `SourceAuthorityProfile` fails gold derivation with `SOURCE_AUTHORITY_PROFILE_ERROR`. | Gold fact is emitted without profile. |
| AC-027 | Missing required `CoverageAssertion` fails coverage-dependent absence and control pass/fail/unknown derivation. | Coverage-dependent fact emits without assertion. |
| AC-028 | Missing `SourceStalenessPolicy` returns `unknown_staleness` and blocks current absence. | Implementation chooses an implicit staleness clock. |
| AC-029 | Source-native merge history is stored as lineage/evidence only and cannot create `IdentityDecision` without active `ResolverProfile`. | Merge history alone auto-merges canonical entities. |
| AC-030 | Source history query outside supported window emits `source_history_window_expired` and no negative no-change fact. | Missing history outside window becomes no change. |
| AC-031 | API cursor exhaustion is candidate completeness evidence only and cannot authorize absence without profile, coverage, and permission scope. | Cursor exhaustion alone emits absence. |
| AC-032 | A stale absence attempt is rejected before graph projection. | Stale absence creates graph expiration. |
| AC-033 | A `VersionManifest` missing authority, coverage, completeness, staleness, source-state, or replay policy refs blocks production replay. | Replay proceeds with missing refs. |
| AC-034 | Derived graph index state cannot create or retract facts. | Graph index drift repair mutates raw, silver, gold, or identity state. |
| AC-035 | Control result polarity must be declared before mapping pass/fail to gold facts. | A control result emits pass/fail without `ControlResultMappingRow`. |

## Sources

[^1]: `PRD-Cadastre.md`, Product Summary, lines 19-47. Establishes Cadastre’s temporal lakehouse architecture, lakehouse system-of-record boundary, graph/CIM projection boundary, table-format-native read/write references, and graph/replay/maintenance correctness constraints.

[^2]: `PRD-Cadastre.md`, Document Status and Non-Scope boundaries, lines 51-71 and 382-405. Establishes non-authority boundaries for direct-to-graph sync, lineage/freshness artifacts, destination cleanup, live probes, acknowledgments, queue drain, provenance, CDC offsets, CDC heartbeats, graph state, external graph systems, and source-native merge history.

[^3]: `PRD-Cadastre.md`, normative contract table, lines 90-120. Defines `SourceCallPolicy`, `SourceCompletenessProfile`, `CollectionMethodVisibilityProfile`, `SourceCompletenessReceipt`, `SourceCompletenessEvidenceRow`, `CoverageAssertion`, `SourceAuthorityProfile`, `ResolverProfile`, and `GraphEdgeSemantics`.

[^4]: `PRD-Cadastre.md`, Omission Semantics, lines 1454-1493. Defines `absent`, `empty`, `explicit_null`, `malformed`, `not_applicable`, `not_observed`, and `unknown`, and requires exhaustive source-state mapping.

[^5]: `PRD-Cadastre.md`, `GoldFact`, lines 2329-2390. Defines canonical bitemporal facts, assertion state, evidence refs, authority, supported MVP fact types, and coverage-sensitive vulnerability/control/network facts.

[^6]: `PRD-Cadastre.md`, `VersionManifest`, lines 2630-2649. Requires production output and replay to include every output-affecting source, state, profile, coverage, authority, resolver, graph, projection, and dataset reference.

[^7]: `PRD-Cadastre.md`, `SourceCompletenessEvidenceRow`, lines 3655-3699. Defines authority-limited evidence classes, including cursor exhaustion, expected count, heartbeat, schema history, ack, provenance closure, source partiality, destination cleanup, live probe, and queue drain.

[^8]: `PRD-Cadastre.md`, `SourceAuthorityProfile`, lines 3787-3834. Defines executable source authority arbitration, coverage requirements, absence semantics, stale-threshold policy, conflict handling, and gold-derivation failure when authority is unresolved or missing.

[^9]: `PRD-Cadastre.md`, graph projection and graph object eligibility, lines 3985-4056. Requires graph expiration/retraction to be source-authority and completeness governed, and requires `GraphEdgeSemantics` and graph eligibility rows before traversal or analysis use.

[^10]: `PRD-Cadastre.md`, `SourceCompletenessProfile`, lines 4831-4965. Defines completeness profiles, evidence classes, authority limits, default decision matrix, and deterministic evaluation rules that block absence from unsafe states.

[^11]: `PRD-Cadastre.md`, initial source authority table, lines 8214-8248. Defines initial authority defaults for endpoint last seen, directory registration, group membership, vulnerability presence/absence, controls, DNS, IP assignment, observed traffic, future reachability, and host ownership.

[^12]: `PRD-Cadastre.md`, `DerivedViewLagPolicy`, lines 3078-3119. Defines derived-view lag policy, stale-read defaults, graph query response requirements, and `DERIVED_VIEW_LAG_ERROR`.

[^13]: `nlspec-spec.md`, Version and NLSpec nature/quality requirements, lines 1-7, 11-25, 28-72, 120-122, and 142-154. Defines NLSpec as prescriptive/generative and requires behavioral completeness, unambiguous interfaces, explicit defaults, mapping tables, binary acceptance criteria, completeness, precision, and define-once economy.

[^14]: `RES-006-source-adapter-ingestion-patterns.md`, Executive summary and evidence limits, lines 15-54. Prior Cadastre research identifying CloudQuery, Steampipe, NiFi, Data Prepper, and Debezium patterns, and rejecting destination cleanup, live-query evidence, provenance-as-truth, ack-as-completeness, and CDC-time-as-valid-time. Used as evidence, not governing authority.

[^15]: ServiceNow official documentation inspected 2026-05-16. IRE documentation describes identification and reconciliation before CMDB insert/update, reconciliation source authorization, and identification rules. Source inspected through official ServiceNow documentation.

[^16]: CloudQuery official documentation inspected 2026-05-16. Documentation describes incremental table cursor state, at-least-once behavior with possible duplicates, `overwrite-delete-stale`, and write-mode defaults/configuration.

[^17]: Steampipe official documentation/repository information inspected 2026-05-16. Sources describe Steampipe as a zero-ETL/live API query system using SQL-style tables.

[^18]: Apache NiFi official documentation inspected 2026-05-16. Documentation describes FlowFiles, queues, provenance/replay, and backpressure.

[^19]: OpenSearch Data Prepper official documentation inspected 2026-05-16. Documentation describes pipelines made of source, buffer, processor, and sink components, and end-to-end acknowledgments with limitations.

[^20]: Debezium official documentation/repository information inspected 2026-05-16. Sources describe Debezium as a CDC platform using Kafka/Kafka Connect concepts, connectors, source database monitoring, and change-event streaming.

[^21]: Apache Flink official documentation inspected 2026-05-16. Documentation describes watermarks as event-time progress and late events after watermark advancement.

[^22]: Apache Beam official documentation inspected 2026-05-16. Documentation describes triggers, late data after watermarks, and watermarks as lower-bound progress estimates in the Java API docs.

[^23]: Tenable official documentation inspected 2026-05-16. Documentation describes scan status/progress, vulnerability states including active/fixed, and reopened/resurfaced-like semantics.

[^24]: Qualys and Rapid7 official documentation inspected 2026-05-16. Qualys documents active/fixed/reopened vulnerability states. Rapid7 documents authenticated discovery scans and coverage considerations for dynamic and recurring vulnerability coverage.

[^25]: OVAL and XCCDF official standards/schema sources inspected 2026-05-16. OVAL Results schema includes true, false, unknown, error, not evaluated, and not applicable result states; XCCDF schema materials define compliance rule result vocabularies.

[^26]: Microsoft official documentation inspected 2026-05-16. Microsoft Graph membership docs describe hidden membership permissions and limited visibility; delta docs describe state tokens; AD `memberOf` docs describe primary-group limitations.

[^27]: Kubernetes and OpenTelemetry official documentation inspected 2026-05-16. Kubernetes docs distinguish names, UIDs, labels, and annotations; OpenTelemetry docs distinguish identifying and descriptive attributes and define Resource/Entity concepts.

[^28]: DNS, DHCP, Zeek, and osquery sources inspected 2026-05-16. RFC 1035 defines DNS TTL cache lifetime; DHCP sources define lease behavior; Zeek docs describe connection `uid`; osquery docs describe differential scheduled query results.

[^29]: Cloud inventory/history and dbt official documentation inspected 2026-05-16. Google Cloud Asset Inventory documents bounded history windows; AWS Config documents resource configuration recording and retention; Azure Resource Graph change analysis documents bounded change history; dbt docs describe source freshness artifacts.

[^30]: DataHub, OpenLineage, and OpenMetadata official documentation inspected 2026-05-16. DataHub docs describe restoring/rebuilding search and graph indexes from metadata aspect state; OpenLineage docs/spec describe run/job/dataset/facet event models; OpenMetadata docs describe change events and lineage APIs.

[^31]: Batfish and cloud reachability diagnostic official documentation inspected 2026-05-16. Batfish docs describe modeled forwarding/traceroute analysis; AWS Reachability Analyzer docs describe static configuration analysis and unsupported/permission limits; Google Connectivity Tests docs distinguish configuration and live data-plane analysis; Azure Network Watcher docs describe IP Flow Verify, Effective Security Rules, Next Hop, and Connection Troubleshoot diagnostics.
