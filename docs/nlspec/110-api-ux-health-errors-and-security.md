---
doc_id: CADASTRE-NLSPEC-110
title: API, UX, Health, Errors, and Security
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define observable API behavior, user-facing states, health, shared error records, security, redaction, and audit behavior.

## Explicit Non-Scope

- Backend query execution internals.
- Raw/silver/gold derivation algorithms.
- Package activation.
- Domain-specific behavior owned by source, identity, temporal, graph, or package specs.

## Imports

- `GraphQueryTranslationProfile`
- `DerivedViewState`
- `IdentityDecision`
- `SourceAuthorityProfile`
- `CadastreSilverObservation`
- `GoldFact`
- `PackageActivationFailureEvent`

- `CoreRecordErrorCodeSet`
- `EvidenceRef`
- `CommonRecordHeader`
- `ActivationControlledArtifactRef`

## Exports

- `GraphQueryRequest`
- `GraphQueryResponse`
- `AssetSearchRequest`
- `AssetDetailResponse`
- `EvidenceDrillbackResponse`
- `OperationalHealthStatus`
- `SourceStateLabel`
- `ExportStateLabel`
- `ErrorRecord`
- `ErrorCodeRegistry`
- `AuditEvent`
- `RedactionPolicy`
- `AuthorizationDecision`

## Roles

| Role | Required capabilities |
| --- | --- |
| `analyst` | Search assets, view graph relationships, inspect evidence metadata, view conflicts and stale facts. |
| `engineer` | View parser, mapping, resolver, projection, and scoring outputs; run replay in authorized non-production contexts. |
| `operator` | Configure feed profiles, schedules, package versions, deployment profiles, and health alerts. |
| `auditor` | View immutable lineage, version manifests, evidence chains, and access audit records. |
| `admin` | Manage users, roles, retention profiles, production package promotion, and raw evidence access. |

## API Bounds

| API field | Default | Bounds |
| --- | ---: | --- |
| Asset search `query` | empty string | Maximum 256 Unicode scalar values after normalization. |
| Asset search `page_size` | 100 | 1 through 1000. |
| Graph query `max_depth` | 3 | 1 through 6. |
| Graph query `page_size` | 100 | 1 through 1000. |
| Graph query `timeout_seconds` | 30 | 1 through 300. |
| Graph page token TTL seconds | 900 | 60 through 3600. |
| `include_raw_payload` | false | Raw payload returned only with raw-evidence permission. |

### GraphQueryRequest schema

`GraphQueryRequest` must validate API bounds before calling `090.QueryGraph` or any backend adapter.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `query_class` | Yes | none | Closed token: `node_detail`, `neighbor_expansion`, `bounded_path`, `evidence_drillback`, `analysis_read_only`. |
| `graph_projection_profile_ref` | Yes | none | Must match the active graph profile used by the derived view. |
| `graph_query_translation_profile_ref` | Yes | none | Must be active and in scope. |
| `backend_taxonomy_mapping_profile_ref` | Yes | none | Must be active and in scope. |
| `allowed_traversal_classes` | Required for `bounded_path` | Omission fails with `GRAPH_TRAVERSAL_CLASS_REQUIRED`; empty array returns no paths. | Values must be closed `090.GraphTraversalClass` tokens. |
| `max_depth` | Required for path queries | `3` | Bounds from API table. |
| `page_size` | No | `100` | Bounds from API table. |
| `timeout_seconds` | No | `30` | Bounds from API table. |
| `page_token` | No | null | Must be a Cadastre page token; backend cursors are rejected. |
| `include_evidence` | No | `false` | Evidence metadata only unless authorization permits more. |
| `include_raw_payload` | No | `false` | Raw payload returned only with raw-evidence permission. |

### GraphQueryResponse graph profile context

Every graph response must include or reference the profile and state context that determined observable output.

| Response field | Required behavior |
| --- | --- |
| `derived_view_state_ref` | Required for every response using graph read-model state. |
| `graph_projection_profile_ref` | Required and must match the profile used to project the served graph objects. |
| `graph_query_translation_profile_ref` | Required and must match request and manifest refs. |
| `backend_taxonomy_mapping_profile_ref` | Required and must match backend materialization refs. |
| `output_eligibility_context_ref` | Required row-set checksum or refs proving search, neighbor, pathfinding, analysis, metrics, and identity-influence eligibility. |
| `redaction_context_ref` | Required authorization/redaction checksum. |
| `page_token` | Cadastre token only; token identity must include profile refs, derived-view state, ordering keys, page boundary, authorization context, and expiry. |

`observed_connection` detail text must state that the edge represents observed traffic only. It must not imply theoretical reachability, service access, policy allow, compromise, exploitability, lateral movement, or identity access.

MVP API and UI text must not use unqualified `reachable`, `not reachable`, `allowed path`, `service accessible`, or `lateral movement path` for graph output unless `200` is promoted and the active owner policies authorize the exact claim.

## Source and Export State Labels

API, UI, evidence drillback, compliance export, audit export, and analysis output must distinguish:

```text
source_stale
derived_view_stale
unknown
not_applicable
authorized_not_observed
permission_limited
source_unavailable
scope_unavailable
partial_known_gap
partial_unknown_gap
not_checked
error
conflicted
ambiguous
```

These labels must not be rendered as authorized negative facts or compliance pass/fail states unless the owning domain spec has emitted the corresponding authoritative output.

`not_authoritative_for_absence` is an owner completeness decision, not an authorized negative fact. Caller-visible output must map it to `unknown` with owner context unless an endpoint exposes owner diagnostics separately.

Mapping-blocked observations must surface as `error`, not `unknown`. Source-extension redaction failures must surface as security errors. OCSF non-authority blocks must not render as `authorized_not_observed`, compliance pass, compliance fail, remediation, risk reduction, or graph mutation.

Temporal resolution failures must render as `error` with owner context, not as `unknown`. Unauthorized negative evidence must render as `unknown` or owner diagnostic, never as an authorized negative fact. Replay failures must render as replay or audit errors and must not be converted into source-state labels. Duplicate no-op correction output must render as no output change with audit evidence. Stale known-state output must render as `source_stale`, not `derived_view_stale`.

### SourceAuthorityClosureStateMapping

Each `060` closure outcome or error class must map to exactly one caller-visible label for the response context. A mapping row must not render a blocked, unknown, stale, permission-limited, not-checked, not-applicable, error, or ambiguous state as an authorized negative fact.

| `060` outcome/error class | Caller label | Authorized negative? | Compliance export behavior | Graph query behavior | Health severity | Required audit fields |
| --- | --- | ---: | --- | --- | --- | --- |
| `authorized_absent` | `authorized_not_observed` or owner-specific absence label | Yes, only for predicates permitting negative facts. | owner fact result | graph effect only when `060` and `090` authorize | informational | authority, completeness, coverage, staleness, effect refs |
| `authorized_not_observed` | `authorized_not_observed` | Yes, only for predicates permitting negative facts. | owner fact result | graph effect only when `060` and `090` authorize | informational | authority, completeness, coverage, staleness, effect refs |
| `SOURCE_AUTHORITY_ROW_MISSING` | `unknown` | No | not pass and not fail | no absence edge, no expiry | blocked | row selector, requested effect, redacted refs |
| `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | `ambiguous` | No | not pass and not fail | no absence edge, no expiry | error | matching row refs when authorized |
| `COVERAGE_DIMENSION_ROW_MISSING` | `unknown` or `partial_unknown_gap` by owner mapping | No | not pass and not fail | no absence edge, no expiry | blocked | coverage domain and redacted refs |
| `COVERAGE_ASSERTION_REQUIRED` | `unknown` | No | not pass and not fail | no absence edge, no expiry | blocked | coverage assertion refs |
| `SOURCE_STALENESS_POLICY_ROW_MISSING` | `source_stale` or `unknown` by `060` blocking reason | No | stale or unknown, not pass/fail | no absence edge, no expiry | blocked | time basis and policy refs |
| `permission_limited` state | `permission_limited` | No | not pass and not fail | no absence edge, no expiry | blocked | visibility row refs |
| `partial_known_gap` | `partial_known_gap` | No | not pass and not fail | no absence edge, no expiry | blocked | known-gap refs |
| `partial_unknown_gap` | `partial_unknown_gap` | No | not pass and not fail | no absence edge, no expiry | blocked | unknown-gap refs |
| `CONTROL_RESULT_MAPPING_ROW_MISSING` | `error` | No | error | no control graph effect | error | external vocabulary, state, redacted refs |
| `not_checked` | `not_checked` | No | not checked | no absence edge, no expiry | diagnostic | mapping row refs |
| `not_applicable` | `not_applicable` | No | not applicable | no absence edge, no expiry | diagnostic | applicability refs |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | `unknown` with owner context | No | not pass and not fail | no absence edge, no expiry | diagnostic | signal refs and requested effect |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | health diagnostic only | No | no export effect | no graph effect | blocked | watermark target, blocking reason, refs |

## Error Model

`ErrorRecord` must contain stable error code, message, severity, retryability, owner spec, affected record type, field path when applicable, record ID when available, source artifact refs, error correlation ID, owner error context, redaction state, and caller-visible field set. Domain-specific codes must be owned by the domain spec; this spec owns the shared shape and generated registry. Identity resolver error codes are owned by `070`; this spec defines only observable handling, redaction, severity, retryability, caller-visible fields, and audit-visible refs. Every 040 error must include owner spec, affected record type, field path when applicable, record ID when available, retryability, severity, redaction state, and source artifact refs.

`message` must be at most 1024 Unicode scalar values after normalization. Caller-visible `ErrorRecord` fields are code, message, severity, retryability, owner spec, affected record type, field path, redaction state, and error correlation ID. Audit-visible fields may additionally include record ID, source artifact refs, owner error context, input checksum, and secure diagnostic refs.

Generic error codes must not be used when a more specific domain code exists.

## Evidence Drillback

Graph evidence chain:

```text
Graph node or edge
  -> GoldFact
  -> CadastreSilverObservation or declared derivation input
  -> RawRecord metadata
  -> raw payload only when authorized
```

Missing required lineage refs must return `LINEAGE_ERROR`. Raw payloads must be redacted unless the caller has raw-evidence permission.

## Security and Redaction

- `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` is non-retryable until the payload is removed or reclassified.
- `GRAPH_BACKEND_ID_FORBIDDEN` is non-retryable for the submitted record and must not leak backend-native ID values to unauthorized callers.
- Schema validation errors must not be collapsed into `unknown`, `not_checked`, or compliance pass/fail.
- Audit events for rejected schema records must include input checksum and redaction summary.
- Raw payload exposure requires explicit raw-evidence permission.
- Graph properties derived from raw payloads must pass `GraphPropertyEvidencePolicy` and redaction policy.
- API responses must not leak existence of an inaccessible asset through partial detail responses.
- Public artifacts must not expose private source bindings, route names, credentials, or environment-specific inventories.
- Audit events must record caller, authorization context checksum, operation, input checksum, output object classes, redaction summary, and error code when emitted.
- Source-extension values that fail redaction, secret scan, namespace validation, or OCSF reserved-name policy must be redacted from caller-visible errors.
- External-schema non-authority failures must expose owner, external field path, and redaction state without exposing raw source values.
- Resolver error context must not expose private scope selectors, concrete tenant inventories, credentials, raw payload values, private routes, scanner site names, directory tenant inventories, zone inventories, account lists, or environment-specific source bindings.

## API, UX, Health, Error, and Security Contract Details

### ErrorCodeOwnershipMatrix

The generated error registry must include every code exported by `040.CoreRecordErrorCodeSet`. The rows below define required observable handling for core schema errors.

| Error code | Owner | Severity | Retryability | Redaction |
| --- | --- | --- | --- | --- |
| `CORE_UNKNOWN_FIELD` | `040` | error | no, until schema or input changes | field path allowed, value redacted |
| `CORE_REQUIRED_FIELD_MISSING` | `040` | error | no, until input changes | field path allowed |
| `CORE_NULL_FORBIDDEN` | `040` | error | no, until input changes | field path allowed |
| `CORE_FIELD_TYPE_INVALID` | `040` | error | no, until input changes | value redacted |
| `CORE_FIELD_BOUNDS_INVALID` | `040` | error | no, until input changes | value redacted |
| `CORE_RECORD_ID_MISMATCH` | `040` | error | no, until input changes | computed ID may be logged only in secure audit |
| `CORE_RECORD_CHECKSUM_MISMATCH` | `040` | error | no, until input changes | checksum visible |
| `CORE_SCHEMA_VERSION_UNSUPPORTED` | `040` | error | no, until schema activation changes | schema version visible |
| `RAW_RECORD_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `SILVER_OBSERVATION_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `CANONICAL_ENTITY_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `SOURCE_ASSET_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `IDENTIFIER_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `GOLD_FACT_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `EVIDENCE_REF_ID_COLLISION` | `040` | error | no, until input changes | record IDs visible; colliding inputs redacted |
| `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` | `040` | security error | no | payload redacted |
| `GRAPH_BACKEND_ID_FORBIDDEN` | `040`/`090` | security error | no | backend ID redacted unless admin audit permits |

The generated error registry must include the following mapping, source-extension, external-schema non-authority, and graph-direction codes.

| Error code | Owner | Severity | Retryability | Redaction |
| --- | --- | --- | --- | --- |
| `MAP_OCSF_ROW_MISSING` | `050` | error | no, until mapping bundle/profile changes | observation type visible; source values redacted |
| `MAP_OCSF_ROW_AMBIGUOUS` | `050` | error | no, until mapping rows change | row IDs visible; raw source values redacted |
| `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | `050` | error | no, until parser/mapping changes | field path visible |
| `OCSF_REQUIRED_OBJECT_PATH_MISSING` | `050` | error | no, until mapping/input changes | object path visible |
| `OCSF_FORBIDDEN_FIELD_EMITTED` | `050` | security error | no | field path visible; value redacted |
| `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED` | `050` | error | no, until observation type or mapping changes | observation type visible |
| `SOURCE_EXTENSION_RULESET_MISSING` | `050` | error | no, until mapping row or rule set changes | source-extension path visible; value redacted |
| `SOURCE_EXTENSION_NAMESPACE_INVALID` | `050` | security error | no | path visible; value redacted |
| `SOURCE_EXTENSION_REDACTION_POLICY_MISSING` | `050` | security error | no | path visible |
| `SOURCE_EXTENSION_SECRET_SCAN_FAILED` | `050` | security error | no | path visible; value redacted |
| `SOURCE_EXTENSION_OCSF_RESERVED_COLLISION` | `050` | security error | no | path visible |
| `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `060` | error | no, until derivation policy changes | external field path visible |
| `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` | `090` | diagnostic | no, until evidence or projection changes | graph IDs visible only when authorized |

| Owner spec | Error prefix or namespace | Shared shape fields | Owner-specific codes | Retryability owner | Redaction owner | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- |
| `010` | boundary/authority | `ErrorRecord` | `DIRECT_SOURCE_CALL_FORBIDDEN`, `PROJECTION_AUTHORITY_VIOLATION`, `PRIVATE_BINDING_LEAK`, `UNDECLARED_AUTHORITY_CLASS` | owner row | `110` | boundary negative rows |
| `020` | feed/table | `ErrorRecord` | feed read, manifest, raw identity, availability, maintenance errors | owner row | `110` | feed fixture rows |
| `030` | dag/lifecycle | `ErrorRecord` | `FORBIDDEN_STAGE_OUTPUT`, lifecycle, run-lock, manifest errors, `ACTIVATION_ARTIFACT_INCOMPLETE` | owner row | `110` | DAG negative rows |
| `040` | canonical data | `ErrorRecord` | `CORE_UNKNOWN_FIELD`, `CORE_REQUIRED_FIELD_MISSING`, `CORE_NULL_FORBIDDEN`, `CORE_FIELD_TYPE_INVALID`, `CORE_FIELD_BOUNDS_INVALID`, `CORE_RECORD_ID_MISMATCH`, `CORE_RECORD_CHECKSUM_MISMATCH`, `CORE_SCHEMA_VERSION_UNSUPPORTED`, record collision errors, `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN`, `GRAPH_BACKEND_ID_FORBIDDEN` | owner row | `110` | core schema rows |
| `050` | mapping/schema | `ErrorRecord` | OCSF row resolution, enum, source extension, base-event field, and CIM errors | owner row | `110` | mapping rows |
| `060` | authority/completeness | `ErrorRecord` | authority, coverage, progress, watermark, and external-schema non-authority errors | owner row | `110` | absence rows |
| `070` | identity | `ErrorRecord` | resolver, review, selector errors | owner row | `110` | identity rows |
| `080` | temporal/replay | `ErrorRecord` | temporal, correction, replay errors | owner row | `110` | event sequence rows |
| `090` | graph | `ErrorRecord` | backend, query, rebuild, drift, and flow-role evidence errors | owner row | `110` | graph rows |
| `100` | package | `ErrorRecord` | activation, trust, rollback, quarantine errors | owner row | `110` | package rows |
| `120` | validation | `ErrorRecord` | validation and acceptance errors | owner row | `110` | validation rows |
| `130` | analysis/registry | `ErrorRecord` | mutation, lineage, registry errors | owner row | `110` | analysis rows |
| `200` | reachability deferred | `ErrorRecord` | reachability prohibited output errors | owner row | `110` | reachability prohibition rows |

### IdentityResolverErrorObservableMapping

`070` owns resolver error semantics. `110` maps resolver errors to observable API, health, validation, and audit behavior. Generic error codes must fail validation when a more specific resolver code applies.

| Error code | Owner | Severity | Retryability | Redaction | Caller-visible fields | Audit-visible refs |
| --- | --- | --- | --- | --- | --- | --- |
| `RESOLVER_PROFILE_ROW_MISSING` | `070` | blocked | no, until profile row activates | row selector redacted | code, severity, retryability, owner, affected record type, redaction state, correlation ID | resolver profile ref, redacted selector hash, version manifest ref |
| `RESOLVER_PROFILE_ROW_AMBIGUOUS` | `070` | error | no, until row set changes | matching row refs redacted unless authorized | code, severity, retryability, owner, redaction state, correlation ID | matching row refs, row-set checksum, version manifest ref |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | `070` | error | no, until evidence changes | scope values redacted; missing key names visible | code, field path, severity, owner, redaction state, correlation ID | evidence item refs, identifier scope refs, input checksum |
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | `070` | diagnostic | yes only after supported row activates | entity type visible | code, severity, retryability, owner, affected record type, correlation ID | resolver row ref and validation ref |
| `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | `070` | blocked | yes after profile bounds or input partition changes | blocking key value redacted; block kind visible | code, severity, retryability, owner, redaction state, correlation ID | block kind, member count, cap, profile ref |
| `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | `070` | blocked | yes after profile bounds or input partition changes | source asset IDs redacted unless authorized | code, severity, retryability, owner, redaction state, correlation ID | partition checksum, candidate cap, profile ref |
| `RESOLVER_DECISION_ROW_MISSING` | `070` | error | no, until decision matrix changes | candidate refs redacted | code, severity, retryability, owner, correlation ID | decision matrix ref, blocker result refs, candidate checksum |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | `070` | error | no, until band row changes | score visible, evidence redacted | code, severity, retryability, owner, correlation ID | confidence band ref, decision row ref, explanation ref |
| `RESOLVER_REVIEW_ROUTING_MISSING` | `070` | error | no, until routing policy changes | candidate refs redacted | code, severity, retryability, owner, correlation ID | routing policy ref, candidate checksum |
| `RESOLVER_SPLIT_POLICY_MISSING` | `070` | error | no, until split policy activates | split refs redacted | code, severity, retryability, owner, correlation ID | split decision ref, resolver explanation ref, version manifest ref |
| `RESOLVER_EXPLANATION_INCOMPLETE` | `070` | error | no, until output or policy changes | missing field names visible; values redacted | code, field path, severity, owner, redaction state, correlation ID | explanation policy ref, output decision ref, checksum inputs |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | `070` | error | yes after reviewer reloads current evidence | evidence values redacted | code, severity, retryability, owner, redaction state, correlation ID | review case ref, expected checksum, supplied checksum |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | `070` | security error | no, until authorization changes | reviewer identity redacted from caller | code, severity, retryability, owner, redaction state, correlation ID | reviewer auth context checksum, case ref, event |
| `TARGET_SELECTOR_UNSAFE` | `070` | security error | no, until selector policy changes | selector value redacted; mechanism visible | code, severity, retryability, owner, redaction state, correlation ID | selector policy ref, unresolved target ref, mechanism |

### Lifecycle error registry rows

The generated `ErrorCodeRegistry` must include lifecycle errors with owner, severity, retryability, redaction, and fixture ID. Owner-specific lifecycle errors must be selected before generic lifecycle fallback codes.

| Error code | Owner | Severity | Retryability | Redaction | Caller-visible behavior |
| --- | --- | --- | --- | --- | --- |
| `LIFECYCLE_MACHINE_MISSING` | `030` | error | no until spec or artifact changes | artifact refs redacted | block dependent output |
| `LIFECYCLE_MACHINE_INACTIVE` | `030` | blocked | no until lifecycle changes | artifact refs redacted | block dependent output |
| `LIFECYCLE_MACHINE_CHECKSUM_MISMATCH` | `030` | error | no until artifact or manifest changes | checksums policy-visible | block output or replay |
| `LIFECYCLE_EVENT_INVALID` | owner machine | error | no until request changes | event visible | reject transition |
| `LIFECYCLE_STATE_DERIVATION_FAILED` | owner machine | error | owner-defined | state refs redacted | reject transition |
| `LIFECYCLE_ILLEGAL_TRANSITION` | owner machine | error | no until prior state or event changes | subject refs redacted | no mutation |
| `LIFECYCLE_IDEMPOTENCY_CONFLICT` | `030` | error | no until idempotency key or input changes | input checksums visible only by policy | no mutation |
| `LIFECYCLE_TRANSITION_EVIDENCE_INCOMPLETE` | `030` | error | no until evidence exists | evidence refs redacted | block output or replay |
| `LIFECYCLE_PARENT_CHILD_STATE_INVALID` | `030` | error | owner-defined | child refs redacted | reject parent transition |
| `LIFECYCLE_GUARD_REFUSED` | owner machine | blocked | owner-defined | guard context redacted | no mutation |

### Feed closure error registry rows

The generated `ErrorCodeRegistry` must include the following owner-specific rows. Caller-visible fields are code, message, severity, retryability, owner spec, affected record type, field path when applicable, redaction state, and error correlation ID. Audit-visible fields may add artifact refs, selected branch, requested effect, validation refs, checksums, and secure diagnostic refs.

| Error code | Owner | Severity | Retryability | Redaction | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | `020` | error | no, until profile changes | field path allowed, value redacted | `feed-020-profile-schema-incomplete` |
| `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | `020` | blocked | no, until artifact activation changes | category visible; private refs redacted | `feed-020-category-row-missing` |
| `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | `020` | blocked | no, until profile or row changes | branch name visible; payload redacted | `feed-020-profile-branch-unresolved` |
| `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | `020` | blocked | no, until subset profile activates | subset scope redacted | `feed-020-declared-subset-required` |
| `UPSTREAM_COMPLETENESS_EVIDENCE_REQUIRED` | `020` | blocked | owner-defined | evidence class visible; private refs redacted | `feed-category-missing-upstream-completeness-*` |
| `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | `020` | diagnostic | no, until policy changes | scope redacted | `feed-020-empty-scope-not-authoritative` |
| `DECLARED_DAG_SUBSET_PROFILE_MISSING` | `030` | blocked | no, until artifact activation changes | subset scope redacted | `fixture-030-subset-profile-missing` |
| `DECLARED_DAG_SUBSET_SCOPE_MISMATCH` | `030` | error | no, until profile or request changes | scope redacted | `fixture-030-subset-scope-mismatch` |
| `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | `030` | error | no, until profile or request changes | output class visible | `fixture-030-subset-output-forbidden` |
| `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | `060` | blocked | no, until artifact activation changes | row selector redacted | `feed-category-missing-profile-row-*` |
| `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | `060` | blocked | owner-defined | evidence refs redacted | `feed-category-missing-upstream-completeness-*` |
| `COMPLETENESS_EFFECT_NOT_ALLOWED` | `060` | blocked | no, until profile row changes | effect token visible | `fixture-060-effect-not-allowed` |
| `COMPLETENESS_BLOCKING_PRECEDENCE_APPLIED` | `060` | diagnostic | no | blocking reason visible; private refs redacted | `fixture-060-blocking-precedence` |
| `EMPTY_SCOPE_NOT_AUTHORIZED_FOR_ABSENCE` | `060` | blocked | no, until profile row changes | scope redacted | `feed-category-empty-complete-*` |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | `060` | blocked | owner-defined | watermark target redacted | `feed-category-watermark-blocked-*` |

### SourceAuthorityClosureErrorRegistryRows

The generated `ErrorCodeRegistry` must include every `060` source-authority closure code with owner, severity, retryability, redaction, and fixture ID. These rows extend the feed closure rows and must be selected before generic activation, validation, or unknown-state codes.

| Error code | Owner | Severity | Retryability | Redaction | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| `SOURCE_AUTHORITY_ROW_MISSING` | `060` | blocked | no, until row set changes | selector redacted | `source-authority-row-resolution-missing` |
| `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | `060` | error | no, until row set changes | matching rows redacted unless audit permits | `source-authority-row-resolution-ambiguous` |
| `SOURCE_STALENESS_POLICY_ROW_MISSING` | `060` | blocked | no, until policy activates | policy selector redacted | `staleness-policy-missing-time-input` |
| `SOURCE_STALENESS_POLICY_ROW_AMBIGUOUS` | `060` | error | no, until policy rows change | matching rows redacted | `staleness-policy-ambiguous` |
| `COVERAGE_DIMENSION_ROW_MISSING` | `060` | blocked | no, until coverage row activates | coverage selector redacted | `coverage-dimension-missing-source-specific` |
| `COVERAGE_DIMENSION_ROW_AMBIGUOUS` | `060` | error | no, until coverage rows change | matching rows redacted | `coverage-dimension-ambiguous` |
| `CONTROL_RESULT_MAPPING_ROW_MISSING` | `060` | error | no, until mapping row activates | external state visible; private refs redacted | `control-result-mapping-unmapped` |
| `CONTROL_RESULT_MAPPING_ROW_AMBIGUOUS` | `060` | error | no, until mapping rows change | matching rows redacted | `control-result-mapping-ambiguous` |
| `PROGRESS_SIGNAL_POLICY_ROW_MISSING` | `060` | blocked | no, until policy activates | signal refs redacted | `progress-signal-weak-combination-blocked` |
| `PROGRESS_SIGNAL_POLICY_ROW_AMBIGUOUS` | `060` | error | no, until policy rows change | matching rows redacted | `progress-signal-ambiguous` |
| `SOURCE_HISTORY_RETENTION_ROW_MISSING` | `060` | blocked | no, until retention row activates | history scope redacted | `staleness-policy-source-history-outside-window` |
| `SOURCE_HISTORY_RETENTION_ROW_AMBIGUOUS` | `060` | error | no, until retention rows change | matching rows redacted | `source-history-retention-ambiguous` |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_MISSING` | `060` | blocked | no, until visibility row activates | scope redacted | `directory-visibility-hidden-membership` |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_AMBIGUOUS` | `060` | error | no, until visibility rows change | matching rows redacted | `directory-visibility-ambiguous` |
| `SOURCE_STATE_MAPPING_ROW_MISSING` | `060` | blocked | no, until mapping row activates | source state visible; refs redacted | `source-authority-state-mapping-missing` |
| `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE` | `060` | blocked | no, until closure rows and validation refs exist | closure refs redacted | `manifest-closure-omitted-authority-row-set-ref` |
| `SOURCE_AUTHORITY_CLOSURE_AMBIGUOUS` | `060` | error | no, until closure rows change | matching rows redacted | `source-authority-row-resolution-ambiguous` |

### TemporalCorrectionReplayErrorRegistryRows

The generated registry must include every `080` temporal, correction, late-arrival, snapshot, no-op, and replay error below. Each row must produce a stable `ErrorRecord` and must not collapse into a generic error when the owner-specific code applies.

| Error code | Owner | Severity | Retryability | Redaction | Caller-visible behavior | Audit-visible fields | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `TEMPORAL_POLICY_UNRESOLVED` | `080` | error | no, until policy row activates | source selector redacted | temporal error | temporal tuple, policy row-set refs | `temporal-resolution-missing-policy` |
| `TEMPORAL_INTERVAL_MODEL_MISSING` | `080` | error | no, until policy row changes | policy payload redacted | temporal error | policy row ref, missing field | `temporal-resolution-interval-model-missing` |
| `SOURCE_TIME_NOT_AUTHORIZED` | `080` | error | no, until policy or input changes | source value redacted | temporal error | selected path, policy ref | `temporal-resolution-source-time-not-authorized` |
| `TEMPORAL_RESOLUTION_REQUIRED` | `080` | error | no, until resolution exists | source refs redacted | temporal error | observation ref, required output class | `temporal-resolution-required` |
| `TEMPORAL_ARTIFACT_MISSING` | `080` | blocked | no, until artifact ref exists | artifact payload redacted | blocked | artifact class, manifest ref | `temporal-artifact-missing` |
| `LATE_ARRIVAL_POLICY_MISSING` | `080` | blocked | no, until policy row exists | policy payload redacted | blocked | dataset, fact type, predicate | `late-arrival-policy-missing` |
| `LATE_ARRIVAL_DISCARD_FORBIDDEN` | `080` | error | no, until route policy changes | evidence refs redacted | error with evidence preserved | route row, temporal ref | `late-arrival-discard-forbidden` |
| `CORRECTION_POLICY_MISSING` | `080` | blocked | no, until policy row exists | policy payload redacted | blocked | fact type, predicate | `gold-correction-policy-missing` |
| `GOLD_CORRECTION_TRANSITION_UNDEFINED` | `080` | error | no, until transition row exists | evidence refs redacted | error | prior state, event, policy ref | `assertion-transition-missing` |
| `CORRECTION_SNAPSHOT_REF_MISSING` | `080` | blocked | no, until snapshot refs exist | snapshot refs redacted by policy | blocked | correction class, old/new role | `correction-snapshot-ref-missing` |
| `MUTABLE_BRANCH_REF_FOR_REPLAY` | `080` | error | no, until immutable ref supplied | mutable ref redacted | replay/correction error | ref type, artifact class | `replay-mutable-ref` |
| `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | `080` | diagnostic | no, until authority changes | tombstone payload redacted | unknown or owner diagnostic, not authorized negative | tombstone ref, `060` blocking reason | `gold-correction-unauthorized-cdc-tombstone` |
| `REPLAY_POLICY_ARTIFACT_MISSING` | `080` | blocked | no, until row exists | artifact payload redacted | replay blocked | output class, manifest ref | `replay-output-class-row-missing` |
| `REPLAY_INPUT_INSUFFICIENT` | `080` | blocked | owner-defined after inputs restore | refs redacted by class | replay blocked | missing refs, sufficiency check | `replay-input-insufficient` |
| `REPLAY_CHECKSUM_MISMATCH` | `080` | error | no, until input or expected output changes | checksums visible only by policy | replay error | expected and actual checksum refs | `replay-checksum-mismatch` |

Temporal, correction, and replay errors must be audit-visible even when caller-visible fields are redacted. Replay errors must not mutate source labels, facts, graph state, watermarks, or exports.

### Activation artifact error handling

Artifact activation failures must use the most specific owner code when available. `030.ACTIVATION_ARTIFACT_INCOMPLETE` is the generic fallback only when no domain-specific code exists.

Caller-visible activation artifact errors must include owner spec, affected artifact class, redaction state, retryability, and error correlation ID. Audit-visible errors may include artifact ID, checksum, package-set ref, activation scope, validation refs, and secure diagnostic refs.

Caller-visible output must not reveal private source bindings, private routes, raw fixture bytes, unauthorized package evidence, or unauthorized artifact payload locations.

| Error source | Caller-visible required fields | Audit-visible additional fields | Source-state label rule |
| --- | --- | --- | --- |
| inactive artifact | owner spec, artifact class, retryability, redaction state, correlation ID | artifact ID, lifecycle status, validation refs | Health state only; do not add a source-state label. |
| checksum mismatch | owner spec, artifact class, retryability, redaction state, correlation ID | expected checksum, actual checksum, package-set ref | Health state only. |
| validation refs missing | owner spec, artifact class, retryability, redaction state, correlation ID | missing validation refs and activation scope | Health state only. |
| package-set artifact mismatch | owner spec, artifact class, retryability, redaction state, correlation ID | package-set ref and release manifest refs | Health state only. |
| owner spec mismatch | owner spec, artifact class, retryability, redaction state, correlation ID | declared and required owner specs | Health state only. |

### PackageActivationErrorObservableMapping

Every package activation, rollback, quarantine, last-known-good, emergency, and package manifest error owned by `100` must appear in `ErrorCodeRegistry`. Package errors must expose caller-visible diagnostics without leaking private package evidence.

Caller-visible package error fields are code, owner, severity, retryability, affected package-set checksum or redacted ref class, redaction state, and error correlation ID. Audit-visible fields may include package type policy row refs, repository metadata refs, repository snapshot refs, trust refs, signature verification refs, transparency refs, attestation refs, provenance refs, SBOM refs, dependency lock refs, compatibility refs, rollback refs, quarantine refs, emergency refs, manifest refs, lifecycle refs, and validation refs.

Artifact payload locations, signer secrets, private repository paths, raw SBOM contents, private source bindings, unauthorized package evidence, and private package registry credentials must not be caller-visible.

| Error code | Owner | Severity | Retryability | Caller-visible behavior | Audit-visible refs |
| --- | --- | --- | --- | --- | --- |
| `PACKAGE_TYPE_UNKNOWN` | `100` | error | no, until release manifest changes | activation blocked; current active set preserved | release manifest ref, supplied token redacted when private |
| `PACKAGE_TYPE_POLICY_MISSING` | `100` | blocked | no, until policy row activates | activation blocked | package type, target environment, policy row-set ref |
| `PACKAGE_TYPE_POLICY_AMBIGUOUS` | `100` | error | no, until policy rows change | activation blocked | matching policy row refs when authorized |
| `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | `100` | error | no, until repository form changes | activation blocked | release manifest ref, repository form |
| `PACKAGE_REPOSITORY_ROLLBACK_DETECTED` | `100` | security error | no, until repository metadata is remediated | activation blocked | anti-rollback state ref, metadata refs |
| `PACKAGE_REPOSITORY_METADATA_EXPIRED` | `100` | blocked | yes after metadata refresh | activation blocked | metadata refs, expiration evidence |
| `PACKAGE_SIGNER_UNAUTHORIZED` | `100` | security error | no, until trust policy or signer changes | activation blocked | signature verification result, signer ref redacted by policy |
| `PACKAGE_SIGNATURE_THRESHOLD_FAILED` | `100` | security error | no, until signature evidence changes | activation blocked | threshold row, verification refs |
| `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | `100` | blocked | no, until evidence changes | activation blocked | transparency policy row ref, release subject digest ref |
| `PACKAGE_TRUST_ROOT_INACTIVE` | `100` | security error | no, until trust policy changes | activation blocked | trust root ref, policy row ref |
| `PACKAGE_ATTESTATION_MISSING` | `100` | blocked | no, until evidence changes | activation blocked | attestation policy row ref |
| `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | `100` | error | no, until evidence or release changes | activation blocked | attestation ref, expected subject digest ref |
| `PACKAGE_PROVENANCE_POLICY_FAILED` | `100` | error | no, until provenance or policy changes | activation blocked | provenance policy row ref, build provenance ref |
| `PACKAGE_SBOM_MISSING` | `100` | blocked | no, until evidence changes | activation blocked | SBOM policy row ref |
| `PACKAGE_SBOM_SUBJECT_MISMATCH` | `100` | error | no, until evidence or release changes | activation blocked | SBOM ref, expected subject digest ref |
| `PACKAGE_SBOM_POLICY_FAILED` | `100` | error | no, until SBOM or policy changes | activation blocked | SBOM policy row ref, SBOM ref |
| `PACKAGE_DEPENDENCY_LOCK_MISSING` | `100` | blocked | no, until lock evidence changes | activation blocked | dependency lock policy row ref |
| `PACKAGE_DEPENDENCY_LOCK_MISMATCH` | `100` | error | no, until lock or manifest changes | activation blocked | lock checksum refs, dependency digest refs |
| `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | `100` | security error | no | activation blocked | attempted dependency source redacted |
| `PACKAGE_COMPATIBILITY_FAILED` | `100` | blocked | no, until compatibility rows change | activation or rollback blocked | compatibility row refs and failed axis |
| `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | `100` | blocked | no, until policy or release changes | new activation blocked | deprecation policy row ref, release refs |
| `PACKAGE_LKG_HEALTH_GATE_FAILED` | `100` | blocked | owner-defined | active-but-not-last-known-good | health gate ref, failed output class refs |
| `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | `100` | blocked | no, until rollback target or state changes | rollback blocked; active set unchanged | rollback policy ref, state refs |
| `PACKAGE_ROLLBACK_REPLAY_INPUT_INSUFFICIENT` | `100` | blocked | yes after inputs restore | rollback blocked; active set unchanged | replay input refs, manifest refs |
| `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | `100` | blocked | no, until graph evidence changes | rollback blocked; active set unchanged | graph rebuild or delta reapply refs |
| `PACKAGE_ROLLBACK_TRUST_INCOMPATIBLE` | `100` | security error | no, until trust policy changes | rollback blocked; active set unchanged | trust refs, rollback trust-allow refs |
| `PACKAGE_ROLLBACK_QUARANTINE_BLOCKED` | `100` | blocked | no, until quarantine successor record permits | rollback blocked; active set unchanged | quarantine record refs |
| `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | `100` | blocked | no, until successor quarantine record permits | activation blocked | quarantine target refs and scope policy refs |
| `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | `100` | security error | no | no activation output and current active set preserved | emergency override record ref, attempted bypass class |
| `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | `100` | blocked | no, until manifest changes | output rejected before visibility | missing package manifest ref classes |

### GraphQueryErrorRegistryRows

| Error code | Owner | Severity | Retryability | Caller-visible behavior | Audit-visible refs |
| --- | --- | --- | --- | --- | --- |
| `GRAPH_EDGE_SEMANTICS_ROW_MISSING` | `090` | error | no, until graph profile changes | graph output blocked | graph profile ref, edge type, validation refs |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `090`, with identity evidence from `070` | diagnostic or error by query class | yes after identity resolution | no endpoint node or dependent edge | unresolved target refs, redacted identity decision context |
| `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | `090` | error | yes after narrower query | no partial output unless query class permits | query checksum, candidate limit, profile refs |
| `GRAPH_PAGE_TOKEN_EXPIRED` | `090`, observable handling by `110` | error | yes with fresh request | no backend query execution | token checksum, expiry, request checksum |
| `GRAPH_PAGE_TOKEN_INVALID` | `090`, observable handling by `110` | security | no for supplied token | no backend query execution and no existence leak | token checksum, mismatch class, authorization checksum |
| `GRAPH_TRAVERSAL_CLASS_REQUIRED` | `090`, observable handling by `110` | error | yes with corrected request | request rejected before backend query | query class and request checksum |
| `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` | `200`, observable handling by `110` | error | no until wording or policy changes | output wording rejected | output checksum and owner policy refs |

### Shared error codes

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `AUTHORIZATION_ERROR` | `110` | Caller lacks permission and the response must not reveal existence. |
| `LINEAGE_ERROR` | `110` | Required lineage or evidence drillback refs are absent or invalid. |
| `PAGE_TOKEN_INVALID` | `110` | Page token is malformed, uses backend cursor identity, or authorization context mismatches. |
| `PAGE_TOKEN_EXPIRED` | `110` | Page token expiration has passed. |
| `API_BOUNDS_INVALID` | `110` | API field exceeds bounds or violates required shape. |
| `RAW_PAYLOAD_PERMISSION_REQUIRED` | `110` | Raw payload was requested and metadata-only redaction is not permitted for the endpoint. |
| `DERIVED_VIEW_LAG_ERROR` | `090`, `110` | Query class requires current graph-derived state and derived view is stale. |

All owner-specific errors must appear in the generated registry before promotion. A generic shared code must not be selected when an owner-specific code precisely covers the failure.

### SourceStateLabelMapping

| Owner state | API label | Compliance label | Audit label | Graph label | Allowed negative interpretation | Default |
| --- | --- | --- | --- | --- | --- | --- |
| `unknown` | `unknown` | `unknown` | `unknown` | `unknown` | none | unknown |
| `permission_limited` | `permission_limited` | `unknown` | `permission_limited` | `permission_limited` | none | permission_limited |
| `partial_known_gap` | `partial_known_gap` | `unknown` | `partial_known_gap` | `partial_known_gap` | none | partial_known_gap |
| `partial_unknown_gap` | `partial_unknown_gap` | `unknown` | `partial_unknown_gap` | `partial_unknown_gap` | none | partial_unknown_gap |
| `source_unavailable` | `source_unavailable` | `error` | `source_unavailable` | `source_unavailable` | none | source_unavailable |
| `scope_unavailable` | `scope_unavailable` | `unknown` | `scope_unavailable` | `scope_unavailable` | none | scope_unavailable |
| `not_checked` | `not_checked` | `not checked` | `not_checked` | `not_checked` | none | not_checked |
| `error` | `error` | `error` | `error` | `error` | none | error |
| `source_stale` | `source_stale` | `stale` | `source_stale` | `source_stale` | none | source_stale |
| `derived_view_stale` | `derived_view_stale` | reject by default | `derived_view_stale` | `derived_view_stale` | none | derived_view_stale |
| `authorized_not_observed` | `authorized_not_observed` | owner-defined | `authorized_not_observed` | owner-defined | only when `060` authorizes | owner-defined |
| `not_applicable` | `not_applicable` | `not applicable` | `not_applicable` | not projected by default | none | not_applicable |

### OwnerStateToSourceStateLabelMapping

| Owner state | Caller-visible label | Required owner context | Authorized negative interpretation |
| --- | --- | --- | --- |
| `not_authoritative_for_absence` | `unknown` | completeness decision, owner spec, blocking reason, and redacted artifact refs when diagnostic access permits | forbidden |
| `partial_known_gap` | `partial_known_gap` | known missing object, partition, or row-range metadata when permitted | forbidden |
| `partial_unknown_gap` | `partial_unknown_gap` | unknown-gap diagnostic and redacted feed refs | forbidden |
| `permission_limited` | `permission_limited` | permission or visibility limitation class | forbidden |
| `source_unavailable` | `source_unavailable` | unavailable source/feed target class | forbidden |
| `scope_unavailable` | `scope_unavailable` | unavailable scope selector class | forbidden |
| stale source output | `source_stale` | staleness policy ref and selected time basis when permitted | forbidden |
| authorized empty or negative output with `060.AbsenceDerivationResult.absence_authorized = true` | `authorized_not_observed` | exact authority, completeness, coverage, and staleness refs | allowed only for the authorized predicate and scope |

### PrivateBindingLeakResponse

Public docs, APIs, exports, and validation reports must fail closed or redact when private source bindings appear. Public API responses must not reveal inaccessible asset existence through partial detail responses.

### ComplianceExportStatePolicy

| Source state | Compliance export result default |
| --- | --- |
| authorized pass/fail from owner fact | pass/fail as owner emits |
| `unknown`, `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `source_stale`, `derived_view_stale` | not pass and not fail; emit mapped non-authoritative state or reject if query class requires current state |
| `not_checked` | not checked |
| `error` | error |
| `not_applicable` | not applicable |
| omitted/redacted | omitted/redacted with audit event |

### HealthStatusMapping

| Owner source | User-visible state | Retryability | Production impact | Alert class | Redaction behavior |
| --- | --- | --- | --- | --- | --- |
| feed availability | `source_unavailable` or `partial_unknown_gap` | retryable when lakehouse evidence can refresh | blocks absence and may block import | feed_health | private refs redacted |
| schema validation | `error` | not retryable until schema or input changes | blocks affected output | schema_health | values redacted |
| source staleness | `source_stale` | owner-defined | blocks absence unless policy permits | data_freshness | source refs redacted |
| derived view lag | `derived_view_stale` | retryable after graph apply/rebuild | blocks compliance/audit graph queries | graph_health | backend details redacted |
| package activation failure | `error` | owner-defined | preserves current active package set | package_health | supply-chain evidence redacted |
| validation acceptance | `blocked`, `not_run`, or `fail` | owner-defined | prevents promotion | validation_health | fixture bytes redacted |
| inactive activation artifact | `blocked` | no, until artifact lifecycle changes | blocks dependent production output | activation_health | artifact payload refs redacted |
| activation artifact checksum mismatch | `error` | no, until artifact or manifest changes | blocks dependent production output | activation_health | checksums visible only when policy permits |
| activation artifact validation refs missing | `blocked` | owner-defined | prevents activation or stage output | activation_health | fixture refs redacted |
| package-set artifact mismatch | `error` | no, until package set changes | preserves current active package set | package_health | package evidence redacted |
| activation artifact owner spec mismatch | `error` | no, until artifact metadata changes | blocks activation or stage output | activation_health | owner names visible; private refs redacted |
| feed activation blocked | `blocked` | no, until profile, category row, feasibility assessment, or validation refs change | blocks feed activation and dependent output | feed_health | profile and artifact refs redacted |
| feed category closure missing | `blocked` | no, until category closure row set activates | blocks category-dependent feed reads | feed_health | category visible; private refs redacted |
| upstream completeness unavailable | `unknown` | owner-defined | blocks absence-sensitive effects | data_freshness | evidence refs redacted |
| completeness effect blocked | `unknown` | no, until completeness row changes | blocks absence, cleanup, retraction, graph expiry, or watermark for requested effect | completeness_health | effect token visible; private refs redacted |
| watermark blocked by completeness | `blocked` | owner-defined | blocks watermark advancement only; must not mutate facts or graph state | watermark_health | watermark target redacted |
| lifecycle illegal transition | `error` | no until event or state changes | no mutation; dependent output blocked | lifecycle_health | subject refs redacted |
| lifecycle idempotency conflict | `error` | no until idempotency conflict resolved | no mutation | lifecycle_health | input checksum policy-visible |
| lifecycle guard refusal | `blocked` | owner-defined | output blocked; no state mutation | lifecycle_health | guard context redacted |
| lifecycle machine inactive or missing | `blocked` | no until artifact activation | output blocked | activation_health | artifact refs redacted |
| package lifecycle failure | `error` or `blocked` | owner-defined | current active package set preserved | package_health | package evidence redacted |
| graph apply lifecycle partial failure | `blocked` | retry only when resume-safe | derived view not advanced | graph_health | backend evidence redacted |
| temporal policy missing | `error` or `blocked` | no, until policy row activates | blocks gold, correction, replay, and dependent graph output | temporal_health | policy refs redacted |
| correction policy missing | `blocked` | no, until policy row activates | blocks correction output | temporal_health | policy refs redacted |
| replay policy missing | `blocked` | no, until output-class row activates | blocks replay output only | replay_health | artifact refs redacted |
| late discard forbidden | `error` | no, until route policy changes | preserves evidence; blocks discard | temporal_health | evidence refs redacted |
| snapshot-ref failure | `blocked` | no, until immutable old/new refs exist | blocks correction and replay output | replay_health | snapshot refs redacted |

| package type policy missing | `blocked` | no, until policy row activates | activation blocked; current active package set preserved | package_health | policy refs redacted |
| unsupported repository form | `error` | no, until release materializes into supported form | activation blocked; current active package set preserved | package_health | repository refs redacted |
| repository freshness blocked | `blocked` | yes after metadata refresh unless rollback detected | activation blocked; current active package set preserved | package_health | metadata refs redacted |
| trust/signature blocked | `error` | no, until trust or signature evidence changes | activation blocked; current active package set preserved | package_health | signer and trust refs redacted |
| attestation/provenance blocked | `blocked` | no, until evidence changes | activation blocked; current active package set preserved | package_health | provenance refs redacted |
| SBOM/license/vulnerability blocked | `blocked` | no, until SBOM or policy changes | activation blocked; current active package set preserved | package_health | raw SBOM bytes redacted |
| compatibility blocked | `blocked` | no, until compatibility rows change | activation or rollback blocked; active set unchanged | package_health | failed axis visible when authorized |
| deprecation window expired | `blocked` | no, until policy changes or verified rollback applies | new activation blocked | package_health | release refs redacted |
| post-activation health failed | `blocked` | owner-defined | active-state evidence preserved; last-known-good not marked | package_health | health gate refs redacted |
| last-known-good not marked | `blocked` | owner-defined | active but not rollback-eligible as LKG | package_health | health refs redacted |
| rollback compatibility blocked | `blocked` | no, until rollback target or evidence changes | active set unchanged | package_health | rollback refs redacted |
| quarantine blocked activation | `blocked` | no, until successor quarantine record permits | no artifact mutation and no activation output | package_health | quarantine target refs redacted |
| emergency bypass forbidden | `error` | no | security error; no activation output | package_health | emergency override refs redacted |
| VersionManifest package refs missing | `blocked` | no, until manifest changes | output rejected before visibility | package_health | missing ref classes visible; private refs redacted |

### Page token canonicalization

API page tokens must be generated from `040.CanonicalJSON` over query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. Backend cursors or internal IDs are forbidden as page token identity.

### PageTokenPolicy

| Policy field | Required behavior |
| --- | --- |
| token input fields | Query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. |
| default expiration | `900` seconds. |
| minimum expiration | `60` seconds. |
| maximum expiration | `3600` seconds. |
| expired token behavior | Fail with `GRAPH_PAGE_TOKEN_EXPIRED` or `PAGE_TOKEN_EXPIRED` before owner query execution. |
| promotion behavior | `110-PAGE-TOKEN-DEFAULTS-AC-001` passes only when token bounds are materialized in this table and in `090.GraphQueryPageTokenPolicy`. |
| stale derived-view behavior | If token references a stale derived-view state and query class requires current state, fail with `DERIVED_VIEW_LAG_ERROR`. |
| authorization context mismatch | Fail with `PAGE_TOKEN_INVALID`; do not reveal whether the original result still exists. |
| backend cursor supplied | Reject with `PAGE_TOKEN_INVALID`; backend cursors are not Cadastre token identity. |

### AuditEventSchema

| Field | Required | Rule |
| --- | ---: | --- |
| `audit_event_id` | Yes | Canonical ID, not backend ID. |
| `caller` | Yes | Caller identity or service principal. |
| `authorization_context_checksum` | Yes | Checksum of evaluated auth context. |
| `operation` | Yes | Endpoint or export operation. |
| `input_checksum` | Yes | Canonical request checksum. |
| `output_object_classes` | Yes | Classes emitted or redacted. |
| `redaction_summary` | Yes | Counts and classes only, no private bytes. |
| `error_correlation_id` | Required when error emitted | Links to `ErrorRecord`. |
| `version_manifest_ref` | Required when production output relies on manifest | Manifest ref. |

### Observable API outcome matrix

| Endpoint | Condition | Output | Error | Redaction | Authorization | Stale behavior | Partial behavior | Pagination | Owner spec |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| asset search | no matches | empty result | none | applied | omit unauthorized | label stale when present | label partial when present | canonical token | `110`, owner states |
| asset search | malformed query | none | `API_BOUNDS_INVALID` or owner validation code | n/a | required | n/a | n/a | n/a | `110` |
| asset search | page size out of range | none | `API_BOUNDS_INVALID` | n/a | required | n/a | n/a | n/a | `110` |
| asset detail | inaccessible entity | none | `AUTHORIZATION_ERROR` | no existence leak | fail closed | n/a | n/a | n/a | `110` |
| evidence drillback | missing lineage | none | `LINEAGE_ERROR` | applied | required | owner label | owner label | n/a | `110` |
| evidence drillback | raw payload not permitted | metadata plus redaction marker | none | raw redacted | raw permission required | owner label | owner label | canonical token | `110` |
| evidence drillback | raw payload requested without permission and endpoint requires hard failure | none | `RAW_PAYLOAD_PERMISSION_REQUIRED` | raw redacted | raw permission required | owner label | owner label | n/a | `110` |
| graph query | graph max depth out of range | none | `API_BOUNDS_INVALID` | n/a | required | n/a | n/a | n/a | `090`, `110` |
| graph query | graph timeout | none unless partial allowed | `GRAPH_QUERY_TIMEOUT` | applied | required | owner label | partial only when allowed | canonical token | `090`, `110` |
| graph query | derived view stale and compliance/audit class | reject | `DERIVED_VIEW_LAG_ERROR` | applied | required | reject by default | owner label | canonical token | `090`, `110` |
| graph query | bounded path omits traversal classes | none | `GRAPH_TRAVERSAL_CLASS_REQUIRED` | n/a | required | n/a | n/a | n/a | `090`, `110` |
| graph query | empty traversal classes | empty path result | none | applied | required | owner label | no paths | canonical token | `090`, `110` |
| graph query | candidate limit reached | none unless partial explicitly permitted | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | applied | required | owner label | no partial by default | canonical token | `090`, `110` |
| graph query | expired or mismatched graph page token | none | `GRAPH_PAGE_TOKEN_EXPIRED` or `GRAPH_PAGE_TOKEN_INVALID` | no backend cursor leak | required | n/a | n/a | n/a | `090`, `110` |
| compliance query | stale source or derived view | reject unless owner permits non-authoritative state | `DERIVED_VIEW_LAG_ERROR` or owner stale code | applied | required | reject by default | n/a | canonical token | `060`, `090`, `110` |
| export | private binding detected | none | `PRIVATE_BINDING_LEAK` | fail closed | export permission required | owner label | owner label | n/a | `010`, `110` |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `110-CLEANUP-AC-001` | No banned reference class remains. |
| `110-CLEANUP-AC-002` | API and UI state labels remain distinct and cannot imply authorized negative facts unless the owning domain spec emitted the authoritative output. |
| `110-CLEANUP-AC-003` | Raw payload exposure remains false by default and requires raw-evidence permission. |
| `110-CLEANUP-AC-004` | Generic error codes remain forbidden when a more specific owner error code exists. |
| `110-SCHEMA-PATCH-AC-001` | Every 040 error code appears in the generated error registry with owner, severity, retryability, and redaction behavior. |
| `110-SCHEMA-PATCH-AC-002` | Raw payload leakage through `EvidenceRef` or graph properties fails closed and produces a redacted audit event. |
| `110-SCHEMA-PATCH-AC-003` | Backend-native IDs are never returned as Cadastre identity, evidence identity, pagination identity, or drillback identity. |

| `110-ERROR-RECORD-AC-001` | `ErrorRecord` has no duplicate field definitions, has `error_correlation_id` and `owner_error_context`, and separates caller-visible from audit-visible fields. |
| `110-PAGE-TOKEN-AC-001` | Page tokens reject backend cursors, authorization-context mismatches, expired tokens, and stale derived-view state when the query class requires current state. |
| `110-TEMPORAL-ERROR-REGISTRY-AC-001` | Every `080` temporal, correction, late-arrival, snapshot, no-op, and replay error code appears in `ErrorCodeRegistry` with owner, severity, retryability, redaction, caller-visible behavior, audit-visible fields, and validation fixture. |
| `110-TEMPORAL-LABEL-AC-001` | Temporal errors render as `error`, unauthorized negatives do not render as authorized absence, replay errors do not mutate output, duplicate no-ops render as no output change with audit evidence, and stale known states render as `source_stale`. |
| `110-ACTIVATION-ERROR-AC-001` | Activation artifact errors expose owner, artifact class, retryability, and redaction state. |
| `110-ACTIVATION-ERROR-AC-002` | Activation artifact errors never collapse into `unknown`, `not_checked`, pass, or fail. |
| `110-PAGE-TOKEN-DEFAULTS-AC-001` | Page-token default and maximum expiration are explicit or promotion is blocked. |
| `110-FEED-CLOSURE-AC-001` | `not_authoritative_for_absence` renders as `unknown` with owner context and never as an authorized negative fact. |
| `110-FEED-CLOSURE-AC-002` | `partial_known_gap` remains caller-visible as `partial_known_gap` and does not collapse to `unknown` when that owner state is available. |
| `110-SOURCE-CLOSURE-STATE-AC-001` | Every `060` closure outcome maps to exactly one caller-visible label and never to authorized negative unless `060` authorized it. |
| `110-SOURCE-CLOSURE-ERROR-AC-001` | Every new `060` error code appears in `ErrorCodeRegistry`. |
| `110-SOURCE-CLOSURE-COMPLIANCE-AC-001` | `unknown`, `error`, `not_checked`, and `not_applicable` never render as pass or fail by default. |
| `110-FEED-CLOSURE-AC-003` | Every new `020`, `030`, and `060` feed-closure error code appears in the generated registry with severity, retryability, redaction, owner, and fixture ID. |
| `110-FEED-CLOSURE-AC-004` | Feed activation, category closure, upstream completeness, effect-blocked, and watermark-blocked health rows are diagnostics only and do not mutate source authority, facts, graph state, or watermarks. |

| `110-LIFECYCLE-ERROR-AC-001` | Lifecycle errors appear in `ErrorCodeRegistry` with owner, severity, retryability, redaction, and fixture ID. |
| `110-LIFECYCLE-ERROR-AC-002` | Lifecycle errors never collapse to `unknown`, `not_checked`, pass, or fail. |
| `110-LIFECYCLE-HEALTH-AC-001` | Lifecycle health rows are diagnostic and do not mutate facts, graph state, package state, completeness, or watermarks. |
| `110-OCSF-MAP-ERROR-AC-001` | Every new `050`, `060`, and `090` OCSF mapping, source-extension, external-schema non-authority, and flow-role evidence code appears in `ErrorCodeRegistry` with severity, retryability, redaction, owner, and caller-visible behavior. |
| `110-PACKAGE-ERROR-AC-001` | Every `100` package activation, rollback, quarantine, last-known-good, and emergency error code appears in `ErrorCodeRegistry`. |
| `110-PACKAGE-ERROR-AC-002` | Package activation errors expose audit refs but do not leak private artifact payloads, private source bindings, raw SBOM bytes, unauthorized package evidence, signer secrets, or private repository paths. |
| `110-PACKAGE-HEALTH-AC-001` | Active-but-not-last-known-good, rollback-blocked, quarantine-blocked, and emergency-bypass-forbidden states render distinctly. |
| `110-OCSF-MAP-ERROR-AC-002` | Mapping-blocked observations render as `error`, source-extension redaction failures render as security errors, and OCSF non-authority blocks never render as authorized negative facts or compliance pass/fail states. |
| `110-IDENTITY-ERROR-AC-001` | Every new `070` resolver error appears in `ErrorCodeRegistry` with owner, severity, retryability, redaction, caller-visible behavior, audit-visible refs, and fixture ID. |
| `110-IDENTITY-ERROR-AC-002` | Resolver errors must use the specific `070` code when applicable and must not collapse to generic validation, unknown, pass, or fail states. |
| `110-IDENTITY-REDACTION-AC-001` | Resolver error responses redact private source bindings, concrete tenant inventories, credentials, raw payload values, and environment-specific source target lists. |
| `110-IDENTITY-NO-MUTATION-AC-001` | Resolver error rendering does not mutate identity, graph, package, completeness, watermark, or source-authority state. |
| `110-GRAPH-RESPONSE-CONTEXT-AC-001` | Every graph response includes derived-view state, graph profile, query translation, backend taxonomy mapping, output eligibility, and redaction context refs. |
| `110-GRAPH-TRAVERSAL-AC-001` | Path query traversal omission rejects with `GRAPH_TRAVERSAL_CLASS_REQUIRED`; empty traversal classes return no paths. |
| `110-GRAPH-PAGE-TOKEN-AC-001` | Expired and invalid graph page tokens reject before backend query execution. |
| `110-GRAPH-WORDING-AC-001` | MVP graph output cannot render unqualified reachability, service accessibility, allowed path, lateral movement, or equivalent claims. |
| `110-GRAPH-NON-IMPLICATION-AC-001` | `observed_connection` detail text states the non-implication contract. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `110-AC-001` | Every observable API outcome has deterministic success, empty-result, redaction, authorization, stale-state, partial-state, and error behavior. |
| `110-AC-002` | Raw payload requests without permission return metadata plus redaction marker, not raw bytes. |
| `110-AC-003` | Source stale and derived-view stale states are never collapsed. |
| `110-AC-004` | Error records include owner spec and use the most specific available error code. |
| `110-AC-005` | Compliance and audit query classes reject stale graph-derived state by default. |
| `110-AC-006` | API, health, evidence drillback, compliance export, and audit export preserve distinctions among unknown, not authoritative, partial, permission-limited, stale, and authorized-not-observed states. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
