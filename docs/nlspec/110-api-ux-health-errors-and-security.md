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
- `ObservabilityInstrumentationProfile`
- `TelemetryCorrelationPolicy`
- `TelemetryHealthMappingPolicy`
- `TelemetryRuntimeState`
- `TelemetryRedactionPolicy`

- `CoreRecordErrorCodeSet`
- `EvidenceRef`
- `CommonRecordHeader`
- `ActivationControlledArtifactRef`
- `030.ScopeSelectorErrorCodeSet`
- `030.ScopeSelectorContext`

## Exports

- `CommonApiResponseEnvelope`
- `GraphQueryRequest`
- `GraphQueryResponse`
- `AssetSearchRequest`
- `AssetSearchResponse`
- `AssetDetailRequest`
- `AssetDetailResponse`
- `EvidenceDrillbackRequest`
- `EvidenceDrillbackResponse`
- `OperationalHealthRequest`
- `OperationalHealthStatus`
- `ComplianceExportRequest`
- `ComplianceExportResponse`
- `AuditExportRequest`
- `AuditExportResponse`
- `EndpointOutcomeMatrix`
- `AuthorizationPermissionMatrix`
- `RedactionDataClassMatrix`
- `ErrorCodeRegistryRow`
- `SourceStateLabel`
- `ExportStateLabel`
- `ErrorRecord`
- `ErrorCodeRegistry`
- `AuditEvent`
- `RedactionPolicy`
- `AuthorizationDecision`
- `StructuredInputRepositoryRequest`
- `StructuredInputRepositoryResponse`
- `StructuredInputChangeProposalRequest`
- `StructuredInputValidationRequest`
- `StructuredInputRepositoryAccessPolicy`
- `StructuredInputRepositoryRedactionPolicy`

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

### CommonApiResponseEnvelope schema

Every public API, health, compliance export, and audit export response must wrap its endpoint-specific payload in `CommonApiResponseEnvelope`. Endpoint-specific response tables define only the `payload` member and payload-local fields. The envelope must be emitted for success, empty result, authorization denial, redaction-only response, owner error, and page-token error outcomes.

| Field | Required | Default or omission behavior | Bounds and canonicalization | Error and checksum behavior |
| --- | ---: | --- | --- | --- |
| `api_contract_version` | Yes | Caller omission materializes to the active `110` API contract version selected by `030.SpecSetVersion`. | Lowercase version token or checksum ref. | Included in `request_checksum` and response checksum. |
| `request_checksum` | Yes | Service computes after request defaults materialize; callers must not supply it as authority. | SHA-256 over canonical request object, endpoint name, caller-visible defaults, authorization request scope, and page-token TTL. | Missing or mismatched computed value fails with `API_BOUNDS_INVALID` before owner execution. |
| `authorization_decision_ref` | Yes | None. | Ref to `AuthorizationDecision`; value may be redacted but ref presence is required. | Missing ref fails response materialization. |
| `redaction_context_ref` | Yes | None. | Ref or checksum over applied `RedactionPolicy`, data classes, caller role, and output class. | Missing ref fails response materialization. |
| `version_manifest_ref` | Yes for production-visible output | Validation-only responses may use null only when the validation row declares no production manifest. | Ref to `030.VersionManifest`. | Missing required API/export refs fail with `VERSION_MANIFEST_INCOMPLETE`. |
| `state_summary` | Yes | Empty object only when no owner state was evaluated. | Object keys are `SourceStateLabel` tokens sorted lexically. | Unknown label fails with `API_BOUNDS_INVALID`. |
| `errors` | Yes | `[]`. | Array of `ErrorRecord`; maximum 128 records per response. | Generic code rejected when owner-specific code covers the failure. |
| `audit_event_ref` | Yes | None. | Ref to `AuditEventSchema`; caller may receive redacted ref only. | Missing audit event fails response materialization. |
| `page_token` | No | null. | Cadastre token only; generated after authorization and redaction context are fixed. | Backend cursor or mismatched context fails before owner execution. |
| `diagnostic_correlation_ref` | No | null. | Opaque Cadastre correlation ref. Raw trace IDs and span IDs are forbidden unless both `140.TelemetryCorrelationPolicy` and `110.RedactionPolicy` permit the exact data class. | Excluded from domain output checksums and replay equivalence by default; included in audit when emitted. |
| `partial_result` | Yes | `false`. | Boolean. | `true` is permitted only when `EndpointOutcomeMatrix.partial_behavior` allows partial output for the endpoint and state class. |
| `redaction_summary` | Yes | Counts default to zero. | Contains data-class counts and redaction reasons only; no raw bytes or private binding values. | Missing summary fails response materialization when any field was redacted. |

### API request and response schemas

The tables below close the public API surface. Unknown fields in any request fail with `API_BOUNDS_INVALID` before owner execution unless the endpoint explicitly declares an extension map. Request checksums are computed after defaults materialize.

#### AssetSearchRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `query` | No | `""`; maximum 256 Unicode scalar values after normalization. | Out of range fails with `API_BOUNDS_INVALID`. |
| `filters` | No | `{}`; keys must be declared asset-search filter tokens. | Unknown filter fails with `API_BOUNDS_INVALID`; private filter values are redacted in audit. |
| `page_size` | No | `100`; range `1..1000`. | Out of range fails before owner execution. |
| `page_token` | No | null. | Validated by `PageTokenPolicy`; invalid token does not reveal result existence. |
| `page_token_ttl_seconds` | No | `900`; range `60..3600`. | Out of range fails with `API_BOUNDS_INVALID`; omitted value is included in checksum. |
| `include_state_summary` | No | `true`. | `false` omits display detail but does not suppress envelope `state_summary` labels required for correctness. |

#### AssetSearchResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Missing envelope fails response materialization. |
| `assets` | Yes | `[]`; maximum `page_size` rows. | Unauthorized assets are omitted without existence leak. |
| `result_count` | Yes | `0`; count of returned rows only, not total inaccessible matches. | Must not reveal inaccessible result counts. |
| `state_summary` | Yes | Mirrors envelope state labels for returned rows. | State labels must use `SourceStateLabelMapping`. |

#### AssetDetailRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `asset_ref` | Yes | None. | Inaccessible or unknown object returns `AUTHORIZATION_ERROR` with no existence leak when caller lacks visibility. |
| `include_evidence` | No | `false`. | Evidence metadata requires normal detail permission; raw payload still requires raw permission. |
| `include_raw_payload` | No | `false`. | `true` without raw permission returns redacted metadata or `RAW_PAYLOAD_PERMISSION_REQUIRED` where endpoint policy requires hard failure. |
| `include_graph_context` | No | `false`. | `true` requires graph profile refs and derived-view state when graph-derived context is served. |

#### AssetDetailResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required for success and denial. |
| `asset` | Required on success | Null only for authorization denial or error. | Redacts inaccessible fields and private source bindings. |
| `evidence_refs` | No | `[]`. | Raw payload bytes are never inlined. |
| `graph_context` | No | null. | Requires `DerivedViewState` and graph profile refs; stale graph state labels as `derived_view_stale`. |

#### EvidenceDrillbackRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `drillback_ref` | Yes | One of evidence ref, gold fact ref, silver observation ref, raw record ref, graph node ref, or graph edge ref. | Unknown or inaccessible refs use `AUTHORIZATION_ERROR` when disclosure would leak existence. |
| `include_raw_payload` | No | `false`. | Raw bytes require raw-evidence permission. |
| `page_size` | No | `100`; range `1..1000`. | Out of range fails before owner execution. |
| `page_token` | No | null. | Validated by `PageTokenPolicy`. |
| `page_token_ttl_seconds` | No | `900`; range `60..3600`. | Included in request checksum. |

#### EvidenceDrillbackResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required. |
| `chain` | Yes | `[]`; ordered graph/gold/silver/raw metadata chain. | Missing required lineage emits `LINEAGE_ERROR`. |
| `raw_payload` | No | null. | Present only with raw-evidence permission and redaction policy allow. |
| `redacted_payload_ref` | Required when raw exists but is hidden | null when no raw exists. | Must include hash/ref, not bytes. |

#### GraphQueryResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required for empty, success, partial, stale, and error outcomes. |
| `query_class` | Yes | Echo selected class. | Must match request checksum inputs. |
| `nodes` | No | `[]`; sorted by Cadastre ordering. | Backend-native IDs are forbidden. |
| `edges` | No | `[]`; sorted by Cadastre ordering. | Conflicted and stale visibility only when `090` handoff permits. |
| `paths` | No | `[]`; deterministic path ordering from `090`. | Empty traversal class returns empty paths without backend traversal. |
| `evidence` | No | `[]`. | Evidence metadata only unless raw permission and redaction allow. |
| `derived_view_state_ref` | Yes when graph read model used | None. | Missing ref fails with `DERIVED_VIEW_LAG_ERROR` or `VERSION_MANIFEST_INCOMPLETE` by context. |
| `graph_profile_context` | Yes when graph read model used | Profile refs for projection, query translation, backend taxonomy, output eligibility, and redaction. | Mismatch with request fails before owner execution. |

#### OperationalHealthRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `health_scope` | No | `system`. Closed tokens: `system`, `feed`, `mapping`, `identity`, `source_authority`, `temporal`, `graph`, `package`, `analysis`, `api`, `observability`. | Unknown token fails with `API_BOUNDS_INVALID`. |
| `include_diagnostics` | No | `false`. | Diagnostic details are redacted by permission. |
| `include_private_refs` | No | `false`. | Caller-visible responses must still redact private source bindings. |

#### OperationalHealthStatus schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required. |
| `status` | Yes | Closed token: `healthy`, `degraded`, `blocked`, `error`. | Must not collapse owner state labels. |
| `components` | Yes | `[]`; one row per evaluated component. | Private refs redacted. |
| `blocking_errors` | Yes | `[]`. | Must use generated `ErrorCodeRegistry`. |

### TelemetryHealthMappingHandoff

Telemetry runtime state may affect `OperationalHealthStatus` only through `140.TelemetryHealthMappingPolicy`.

Default mapping:

| Telemetry condition | Required health effect | Forbidden domain effect |
| --- | --- | --- |
| exporter unavailable | `status = degraded` for `health_scope = observability`; system may aggregate to degraded according to `EndpointOutcomeMatrix` | No source, identity, fact, graph, package, audit, replay, or watermark mutation. |
| exporter queue overflow | `status = degraded` unless active policy maps to `blocked` for observability diagnostics | No domain mutation. |
| telemetry profile missing when telemetry required | `status = blocked` for observability health and health output using telemetry state | No domain mutation. |
| Collector unreachable | `status = degraded` unless active policy maps to `blocked` | No domain mutation. |
| telemetry disabled by active profile | Healthy only for telemetry-disabled profile; not an exporter failure | No domain mutation. |

#### ComplianceExportRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `export_scope` | Yes | None. | Scope must be authorized before result enumeration. |
| `as_of` | No | Current authorized query mode from owner policy; no wall-clock fallback for fact time. | Temporal failures surface as owner errors. |
| `include_evidence_refs` | No | `false`. | Evidence refs redacted by permission. |
| `page_size` | No | `100`; range `1..1000`. | Out of range fails. |
| `page_token` | No | null. | Validated before export owner execution. |
| `page_token_ttl_seconds` | No | `900`; range `60..3600`. | Included in request checksum. |

#### ComplianceExportResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required. |
| `rows` | Yes | `[]`; maximum `page_size`. | `unknown`, `error`, `not_checked`, `not_applicable`, `conflicted`, and `ambiguous` must not render as pass/fail by default. |
| `export_checksum` | Yes | SHA-256 over canonical exported rows and envelope refs. | Missing checksum fails export visibility. |
| `non_pass_fail_counts` | Yes | Counts for every non-pass/fail state label. | Counts must be nonnegative. |

#### AuditExportRequest schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `audit_scope` | Yes | None. | Requires audit export permission. |
| `time_range` | Yes | Half-open known-time or audit-event interval. | Missing bounds fail with `API_BOUNDS_INVALID`. |
| `include_secure_diagnostics` | No | `false`. | Requires secure audit permission and still redacts private bindings when policy forbids display. |
| `page_size` | No | `100`; range `1..1000`. | Out of range fails. |
| `page_token` | No | null. | Validated before audit enumeration. |
| `page_token_ttl_seconds` | No | `900`; range `60..3600`. | Included in request checksum. |

#### AuditExportResponse schema

| Field | Required | Default, bounds, and omission behavior | Error and redaction behavior |
| --- | ---: | --- | --- |
| `envelope` | Yes | `CommonApiResponseEnvelope`. | Required. |
| `audit_events` | Yes | `[]`; maximum `page_size`. | Private source bindings and secure diagnostics redacted by permission. |
| `audit_export_checksum` | Yes | SHA-256 over canonical audit rows and envelope refs. | Missing checksum fails export visibility. |
| `redaction_summary` | Yes | Must include audit-specific redaction counts. | Raw payload bytes, signer secrets, private paths, and backend IDs must not appear. |

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
| `page_token` | No | null | Must be a Cadastre page token; backend cursors, SQL cursor names, prepared statement names, PostgreSQL OIDs, tuple IDs, sequence values, physical row locators, AGE vertex IDs, AGE edge IDs, AGE path IDs, AGE graph/label IDs, agtype object IDs, and provider-native query handles are rejected. |
| `page_token_ttl_seconds` | No | `900` | Bounds `60..3600`; omitted value is materialized before `request_checksum`. |
| `include_evidence` | No | `false` | Evidence metadata only unless authorization permits more. |
| `include_raw_payload` | No | `false` | Raw payload returned only with raw-evidence permission. |

Coverage-domain API filters are not part of the current `GraphQueryRequest` schema. If an API request or filter field named `coverage_domain` or `coverage_domain_filters` is added by a later owner spec, each value must be a `060.CoverageDomainToken`. Display labels and aliases are rejected with `060` owner errors, not accepted as user-friendly synonyms. UI display labels may be rendered separately and must not be sent as runtime tokens.

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
authorized_absent
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
blocked
blocked_validation
diagnostic
```

These labels must not be rendered as authorized negative facts or compliance pass/fail states unless the owning domain spec has emitted the corresponding authoritative output.

`not_authoritative_for_absence` is an owner completeness decision, not an authorized negative fact. Caller-visible output must map it to `unknown` with owner context unless an endpoint exposes owner diagnostics separately.

Mapping-blocked observations must surface as `error`, not `unknown`. Source-extension redaction failures must surface as security errors. OCSF non-authority blocks must not render as `authorized_not_observed`, compliance pass, compliance fail, remediation, risk reduction, or graph mutation.

Temporal resolution failures must render as `error` with owner context, not as `unknown`. Unauthorized negative evidence must render as `unknown` or owner diagnostic, never as an authorized negative fact. Replay failures must render as replay or audit errors and must not be converted into source-state labels. Duplicate no-op correction output must render as no output change with audit evidence. Stale known-state output must render as `source_stale`, not `derived_view_stale`.

### SourceAuthorityClosureStateMapping

Each `060` closure outcome or error class must map to exactly one caller-visible label for the response context. A mapping row must not render a blocked, unknown, stale, permission-limited, not-checked, not-applicable, error, or ambiguous state as an authorized negative fact.

| `060` outcome/error class | Caller label | Authorized negative? | Compliance export behavior | Graph query behavior | Health severity | Required audit fields |
| --- | --- | ---: | --- | --- | --- | --- |
| `authorized_absent` | `authorized_absent` | Yes, only for predicates permitting negative facts. | owner fact result | graph effect only when `060` and `090` authorize | informational | source-dataset, authority, completeness, coverage, staleness, effect refs |
| `authorized_not_observed` | `authorized_not_observed` | Yes, only for predicates permitting negative facts. | owner fact result | graph effect only when `060` and `090` authorize | informational | authority, completeness, coverage, staleness, effect refs |
| `SOURCE_AUTHORITY_ROW_MISSING` | `unknown` | No | not pass and not fail | no absence edge, no expiry | blocked | row selector, requested effect, redacted refs |
| `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | `ambiguous` | No | not pass and not fail | no absence edge, no expiry | error | matching row refs when authorized |
| `COVERAGE_DIMENSION_ROW_MISSING` | `unknown` or `partial_unknown_gap` by owner mapping | No | not pass and not fail | no absence edge, no expiry | blocked | coverage domain and redacted refs |
| `COVERAGE_ASSERTION_REQUIRED` | `unknown` | No | not pass and not fail | no absence edge, no expiry | blocked | coverage assertion refs |
| `COVERAGE_DOMAIN_REQUIRED` | `error` | No | error | no graph effect | error | token field path and owner context |
| `COVERAGE_DOMAIN_TOKEN_INVALID` | `error` | No | error | no graph effect | error | field path; invalid token redacted unless public display label |
| `COVERAGE_DOMAIN_ALIAS_REJECTED` | `error` | No | error | no graph effect | error | alias value visible only when non-private display label |
| `COVERAGE_DOMAIN_UNKNOWN` | `unknown` | No | not pass and not fail | no graph effect | blocked | token visible when public |
| `COVERAGE_DOMAIN_DUPLICATE` | `error` | No | error | no graph effect | error | duplicate token visible when public |
| `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` | `unknown` | No | not pass and not fail | no graph effect | blocked | feed category and token visible when public |
| `COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT` | `unknown` | No | not pass and not fail | no graph effect | blocked | effect and coverage-domain owner context |
| `COVERAGE_DOMAIN_INACTIVE_DEFERRED` | `unknown` with owner diagnostic | No | not pass and not fail | no reachability output | blocked or diagnostic by context | token visible |
| `SOURCE_STALENESS_POLICY_ROW_MISSING` | `source_stale` or `unknown` by `060` blocking reason | No | stale or unknown, not pass/fail | no absence edge, no expiry | blocked | time basis and policy refs |
| `permission_limited` state | `permission_limited` | No | not pass and not fail | no absence edge, no expiry | blocked | visibility row refs |
| `partial_known_gap` | `partial_known_gap` | No | not pass and not fail | no absence edge, no expiry | blocked | known-gap refs |
| `partial_unknown_gap` | `partial_unknown_gap` | No | not pass and not fail | no absence edge, no expiry | blocked | unknown-gap refs |
| `CONTROL_RESULT_MAPPING_ROW_MISSING` | `error` | No | error | no control graph effect | error | external vocabulary, state, redacted refs |
| `not_checked` | `not_checked` | No | not checked | no absence edge, no expiry | diagnostic | mapping row refs |
| `not_applicable` | `not_applicable` | No | not applicable | no absence edge, no expiry | diagnostic | applicability refs |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | `unknown` with owner context | No | not pass and not fail | no absence edge, no expiry | diagnostic | signal refs and requested effect |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | health diagnostic only | No | no export effect | no graph effect | blocked | watermark target, blocking reason, refs |
| `deterministically_blocked` | `unknown` with owner diagnostic | No | not pass and not fail | no graph delta, no expiry | diagnostic | block row ref, block scope, deterministic block code |
| `blocked_validation` | `error` | No | error | no graph delta, no expiry | blocked | validation refs and blocking reason |
| `not_authoritative_for_absence` | `unknown` | No | not pass and not fail | no absence edge, no expiry | diagnostic | completeness refs and requested effect |
| `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | `unknown` | No | not pass and not fail | no graph effect | blocked | feed category, source dataset, requested effect |
| `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | `unknown` | No | not pass and not fail | no graph effect | blocked | upstream evidence class and requested effect |
| `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE` | `unknown` | No | not pass and not fail | no graph effect | blocked | closure selector and omitted refs |
| `SOURCE_AUTHORITY_CLOSURE_AMBIGUOUS` | `ambiguous` | No | not pass and not fail | no graph effect | error | matching closure rows when authorized |
| `SOURCE_AUTHORITY_CLOSURE_BLOCKED` | `unknown` with owner diagnostic | No | not pass and not fail | no graph effect | blocked | block scope, block code, validation refs |
| `SOURCE_DATASET_CATALOG_ROW_MISSING` | `unknown` | No | not pass and not fail | no graph effect | blocked | source dataset token |
| `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` | `ambiguous` | No | not pass and not fail | no graph effect | error | matching dataset rows when authorized |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `error` for attempted authority effect | No | error | no graph effect | error | external profile ref, field path, requested effect |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` | `error` | No | error | no graph effect | error | matching signal rows when authorized |
| `SOURCE_HISTORY_OUTSIDE_WINDOW_NO_PROOF` | `unknown` | No | not pass and not fail | no graph effect | diagnostic | retention window, query scope |
| `SOURCE_HISTORY_COVERAGE_ROW_MISSING` | `unknown` | No | not pass and not fail | no graph effect | blocked | source-history coverage domain |
| `DETERMINISTIC_BLOCK_ROW_SELECTED` | `unknown` with owner diagnostic | No | no export effect | no graph effect | diagnostic | block row ref and mutation-prohibition proof |

## Error Model

`ErrorRecord` must contain stable `error_code`, message, severity, retryability, owner spec, affected record type, field path when applicable, record ID when available, source artifact refs, error correlation ID, owner error context, redaction state, and caller-visible field set. Domain-specific codes must be owned by the domain spec; this spec owns the shared shape and generated registry. Identity resolver error codes are owned by `070`; this spec defines only observable handling, redaction, severity, retryability, caller-visible fields, and audit-visible refs. Every 040 error must include owner spec, affected record type, field path when applicable, record ID when available, retryability, severity, redaction state, and source artifact refs.

`message` must be at most 1024 Unicode scalar values after normalization. Caller-visible `ErrorRecord` fields are defined by `StandardErrorCallerFields`; audit-visible fields are defined by `StandardErrorAuditFields`. Owner-specific values must be nested in `owner_error_context` and must not be added as top-level caller-visible fields.

Generic error codes must not be used when a more specific domain code exists.

### StandardErrorFieldSets

`StandardErrorCallerFields` and `StandardErrorAuditFields` are deterministic macros expanded by `GenerateErrorCodeRegistry` before canonical row serialization and checksum computation. Owner fragments may reference the macro names, but generated registry rows must contain the expanded field names.

| Field set | Expanded fields | Required use |
| --- | --- | --- |
| `110.StandardErrorCallerFields` | `error_code`, `message`, `severity`, `retryability`, `owner_spec`, `affected_record_type`, `field_path`, `redaction_state`, `error_correlation_id` | Every caller-visible owner error row. |
| `110.StandardErrorAuditFields` | `error_code`, `message`, `severity`, `retryability`, `owner_spec`, `affected_record_type`, `field_path`, `redaction_state`, `error_correlation_id`, `record_id`, `source_artifact_refs`, `owner_error_context`, `input_checksum`, `secure_diagnostic_refs`, `validation_fixture_ref`, `version_manifest_ref` | Every audit-visible owner error row. |

A generated row whose caller-visible field list contains `code`, omits `error_code`, omits one standard caller field, adds owner-specific top-level fields, or places owner-specific detail outside `owner_error_context` must fail with `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` before API, export, health, compliance, audit, or validation output.

### ErrorRedactionClassMatrix

`ErrorRedactionClassMatrix` is the closed redaction class set for generated error registry rows. Owner fragments must use these classes either directly or through a `110.StandardErrorRedactionRule` row.

| Redaction class | Caller-visible behavior | Audit-visible behavior | Required mapping |
| --- | --- | --- | --- |
| `public_token` | May be shown to any caller who may see the error record. | Visible. | `error_code`, `severity`, `retryability`, `owner_spec`, and non-sensitive enum tokens. |
| `bounded_context` | May be shown after endpoint authorization and output-class redaction. | Visible with checksum context. | `message`, `affected_record_type`, `field_path`, `redaction_state`, and bounded owner context summaries. |
| `authorized_ref` | May be shown only when the caller can access the referenced object class. | Visible as ref and checksum. | Record refs, source artifact refs, validation refs, version manifest refs, package refs, policy refs, and owner artifact refs. |
| `secure_audit_only` | Hidden from caller-visible output. | Visible only in secure audit contexts. | Secure diagnostic refs, redacted raw-value refs, backend diagnostics, and private implementation diagnostics. |
| `always_forbidden` | Never caller-visible. | Never stored as raw value in normal audit output; secure audit may store only hash, byte count, and approved sealed ref. | Raw payload bytes, credentials, private bindings, source-native identity values, backend IDs, provider-native query text, raw SBOM bytes, signer secrets, and unredacted package or registry payload bytes. |

### StandardErrorRedactionRule

| Rule ref | Required class mapping |
| --- | --- |
| `110.StandardErrorRedactionRule.owner_context` | Standard caller fields map to `public_token` or `bounded_context`; standard audit refs map to `authorized_ref`; `owner_error_context` maps to `bounded_context` unless a nested field declares a stricter class; `secure_diagnostic_refs` map to `secure_audit_only`; all `always_forbidden` data classes remain forbidden. |
| `110.StandardErrorRedactionRule.security_boundary` | Same as `owner_context`, except attempted sensitive values, private bindings, credentials, backend IDs, provider-native query text, raw payload bytes, raw SBOM bytes, and signer secrets must map to `always_forbidden` even when audit-visible context exists. |
| `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | Same as `security_boundary`; additionally, the owner context may include only data class names, redacted refs, checksums, field paths, and blocking reasons. |

### ErrorRegistryDuplicateOwnershipPolicy

`GenerateErrorCodeRegistry` must reject duplicate `error_code` values unless this table declares a shared alias and the duplicate rows are byte-identical after macro expansion. The current shared-alias set is empty.

| Error code | Final owner | Required owner behavior | Rejected duplicate owner |
| --- | --- | --- | --- |
| `PRIVATE_BINDING_LEAK` | `010` | Boundary and public/private binding violations detected by any owner must emit imported `010.PRIVATE_BINDING_LEAK` with `010.BoundaryErrorContext`. | `040` |
| `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `060` | External-schema signal authority attempts must emit imported `060.EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` with `060.SourceAuthorityErrorContext`. | `050` |
| `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | `060` | Tombstone-driven retraction attempts must call `060.DeriveAbsenceOrUnknown` and emit `060.CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` when authorization fails. | `080` |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `090` | Graph projection endpoint blockers must emit `090.GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; identity resolver code must not own this registry row. | `070` |
| `DERIVED_VIEW_LAG_ERROR` | `090` | Derived-view lag failures must emit `090.DERIVED_VIEW_LAG_ERROR`; `110` renders the generated row but must not co-own it. | `110` |

### OwnerErrorContextMinimumSchema

Every owner context schema referenced by an error fragment must include the following minimum fields. Owner specs may add owner-specific fields only inside `owner_error_context`; added fields must declare one `ErrorRedactionClassMatrix` class.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable owner context schema version. |
| `owner_spec` | Yes | Must equal the generated row owner. |
| `error_code` | Yes | Must equal the generated row error code. |
| `failure_class` | Yes | Owner-defined bounded token describing the failure family. |
| `operation` | Yes | Owner-defined bounded token for the operation that emitted the error. |
| `affected_record_type` | Yes | Null only when no record type exists. |
| `field_path` | Yes | Null only when the failure is not field-specific. |
| `artifact_refs` | Yes | Canonically sorted refs; empty array when no artifact was consulted. |
| `validation_refs` | Yes | Canonically sorted refs proving the row; empty only for runtime-only shared errors explicitly exempted by `120`. |
| `redaction_classes` | Yes | Map from owner-context field path to one redaction class. |
| `blocking_reason` | No | Required when `severity = blocked`. |

### ScopeSelectorErrorMappingHandoff

`110` renders shared selector failures only after the owning spec maps `030.ScopeSelectorErrorCodeSet` values to owner-specific error codes. Generic selector errors must not be exposed when an owner-specific selector error exists.

Caller-visible selector diagnostics may include owner spec, owner row family, selected `030.ScopeSelectorContext.context_ref`, selector checksum, redacted dimension keys, row checksum, activation artifact ref, and owner error code. Caller-visible responses must not include raw private selector values, private binding values, private routes, credentials, host lists, account lists, scanner sites, backend internal IDs, private endpoint URLs, or raw private fixture bytes.

Endpoint outcomes for selector errors are closed by this table.

| Selector failure class | Required caller-visible outcome | Required state label when state output exists |
| --- | --- | --- |
| duplicate dimension | owner-specific blocked error | `blocked` |
| duplicate value | owner-specific blocked error | `blocked` |
| unknown field | owner-specific blocked error | `blocked` |
| unsupported dimension | owner-specific blocked error | `blocked` |
| under-scoped request | owner-specific blocked error | `blocked` |
| subset disallowed | owner-specific blocked error | `blocked` |
| private-binding leak | owner-specific security error or `010.PRIVATE_BINDING_LEAK` | `blocked` |
| scope mismatch | owner-specific missing or mismatch error | `blocked` or endpoint-specific no-op when the owner declares no visible output |
| ambiguity | owner-specific ambiguity error | `ambiguous` |

Selector ambiguity must render as `SourceStateLabel.ambiguous` when a state label is emitted. It must not render as conflict, pass, fail, authorized absence, cleanup, graph expiry, retraction, watermark, remediation, or generic blocked state when owner ambiguity context is available.

### ErrorCodeRegistryRow schema

`ErrorCodeRegistry` is generated from owner fragments and shared `110` rows. Owner specs own error causes and owner context. `110` owns the caller-visible registry shape, final severity, final retry class, redaction behavior, and generic-code precedence.

`140` owns telemetry error causes and owner context. `110` owns caller-visible severity, retry class, redaction behavior, generated registry shape, and generic-code precedence for telemetry error rows.

Closed `severity` enum:

```text
informational
diagnostic
blocked
error
security_error
```

Closed `retry_class` enum:

```text
none
caller_correctable
retry_after_refresh
retry_after_owner_repair
transient_retryable
policy_change_required
```

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `error_code` | Yes | None. | Stable uppercase token. Duplicate codes across owners reject registry generation. |
| `owner_spec` | Yes | None. | Exact owning spec ID. |
| `severity` | Yes | None. | Must be one closed enum value. Owner input field `default_severity` is accepted only as a transition alias and must normalize to `severity` before checksum computation. |
| `retry_class` | Yes | None. | Must be one closed enum value. Owner input field `default_retry_class` is accepted only as a transition alias and must normalize to `retry_class` before checksum computation. |
| `redaction_rule` | Yes | None. | Must map every caller-visible and audit-visible field to a data class. |
| `caller_visible_fields` | Yes | None. | Must expand to `110.StandardErrorCallerFields`. The legacy field name `code` is forbidden. |
| `audit_visible_fields` | Yes | None. | Must expand to `110.StandardErrorAuditFields`. Owner-specific details must be nested under `owner_error_context`. |
| `owner_context_schema_ref` | Yes | None. | Ref to owner context shape; `TODO:` blocks promotion. |
| `fixture_ref` | Yes | None. | Exact `120` validation fixture ref proving the row. Legacy `fixture_family` input is accepted only as a transition alias and must normalize to exact `fixture_ref` rows before promotion. Wildcard-only fixture refs are forbidden. |
| `shared_110_override_ref` | No | null. | May narrow presentation only; must not change owner cause semantics. |
| `generated_row_checksum` | Generated | none | SHA-256 over the canonical expanded row bytes; owner fragments must not supply or override this field. |

### GenerateErrorCodeRegistry(owner_fragments, shared_110_rows)

```text
GenerateErrorCodeRegistry(owner_fragments, shared_110_rows):
1. Canonically sort owner fragments by owner spec, then error_code.
2. Normalize transition aliases `default_severity` to `severity`, `default_retry_class` to `retry_class`, and `fixture_family` to `fixture_ref` only when the normalized value is exact and non-wildcard. Reject any fragment row with a missing required field, unknown severity, unknown retry_class, missing redaction_rule, missing exact fixture_ref, wildcard-only fixture ref, `code` in `caller_visible_fields`, owner-specific top-level caller fields, or `TODO:` value.
3. Reject duplicate `error_code` values unless the duplicates are byte-identical shared aliases explicitly declared by `110`.
4. Merge each owner row with at most one shared `110` observable override selected by exact error_code.
5. Reject a shared override that changes owner_spec, owner_context_schema_ref, or fixture_ref.
6. Reject a generic shared code for a failure when an owner-specific generated row covers the same failure class and owner context.
7. Expand `StandardErrorCallerFields`, `StandardErrorAuditFields`, and `StandardErrorRedactionRule` refs before serialization.
8. Compute `generated_error_registry_checksum` over canonical row bytes sorted by `error_code`.
9. Emit `ErrorCodeRegistry` and require its checksum in `030.VersionManifest` for API, export, health, compliance, audit, validation, and telemetry-visible diagnostic output.
```

`GenerateErrorCodeRegistry` must generate exact rows for every `030` run-lock error. A generic registry row must not substitute for run-lock timing, conflict, heartbeat, stale recovery, fencing, idempotency, commit guard, assertion, or lock-loss failures.

`GenerateErrorCodeRegistry` must generate exact rows for `DOMAIN_LEDGER_OWNER_DUPLICATE`, `OWNER_CONTRACT_REF_UNEXPORTED`, and `DUPLICATE_OWNER_EXPORT`. A generic validation or registry error must not substitute for define-once closure failures.

If any owner spec emits an error code that lacks exactly one generated `ErrorCodeRegistryRow`, API/export output must fail before response visibility with `VERSION_MANIFEST_INCOMPLETE` or the most specific registry generation error. Shared codes such as `AUTHORIZATION_ERROR`, `PAGE_TOKEN_INVALID`, and `API_BOUNDS_INVALID` may be selected only when no owner-specific code covers the failure. PostgreSQL and AGE graph backend failures must use the owner-specific `090` rows for schema fingerprint, duplicate IDs, orphan edges, query timeout, unsupported query plan, direct DML, unsafe `search_path`, RLS bypass, provider support, AGE extension, AGE mutation, AGE namespace bypass, AGE internal ID, restore, and upgrade errors; generic graph or API errors are forbidden when those rows cover the failure.

Owner error fragments for activation-controlled row families must include every owner-specific missing-field, invalid-field, duplicate, checksum, manifest, package-set, and extension error used by that owner. Missing owner row-schema error fragments must fail generated registry validation before API, export, health, audit, or validation output.

### OwnerDomainRegistrySourceRule

`110.GenerateErrorCodeRegistry` is the only final registry algorithm. Owner-domain shorthand tables in this document are non-authoritative rendering examples unless they explicitly name `Shared110ErrorRegistryFragment`. Such tables must not be parsed as registry fragments, must not define final severity or retry class, and must not supply owner context, fixture refs, or registry checksums.

The generated registry must use the canonical owner fragments below.

| Failure family | Required registry source |
| --- | --- |
| Documentation governance and define-once failures | `000.GovernanceErrorRegistryFragment` |
| Lakehouse feed and table-state failures | `020.LakehouseErrorRegistryFragment` |
| Scope selector, activation-row, processing, lifecycle, run-lock, and manifest failures | `030.ProcessingErrorRegistryFragment` |
| Core record failures | `040.CoreRecordErrorRegistryFragment` |
| Mapping and external-schema failures | `050.MappingErrorRegistryFragment` |
| Source authority, coverage, completeness, absence, and watermark failures | `060.SourceAuthorityErrorRegistryFragment` |
| Identity resolver and target selector failures | `070.IdentityErrorRegistryFragment` |
| Temporal, correction, replay, and gold derivation failures | `080.TemporalErrorRegistryFragment` |
| Graph projection, query, backend, derived-view lag, and active MVP reachability-boundary failures | `090.GraphErrorRegistryFragment` |
| Package, package-set activation, rollback, quarantine, trust, and package parity failures | `100.PackageErrorRegistryFragment` |
| Shared API, security, page-token, lineage, raw-payload permission, registry-generation, and API wording failures | `110.Shared110ErrorRegistryFragment` |
| Validation-harness failures not owned by documentation governance | `120.ValidationErrorRegistryFragment` |
| Analysis, enrichment, lineage, and registry-governance failures | `130.AnalysisErrorRegistryFragment` |
| Telemetry failures | `140.TelemetryErrorRegistryFragment` |

### SharedRegistryAliasRejectionTable

These aliases are invalid registry input. `GenerateErrorCodeRegistry` must reject them before canonical row generation.

| Rejected alias | Canonical code | Owner |
| --- | --- | --- |
| `OCSF_MAPPING_ROW_MISSING` | `MAP_OCSF_ROW_MISSING` | `050` |
| `OCSF_MAPPING_ROW_AMBIGUOUS` | `MAP_OCSF_ROW_AMBIGUOUS` | `050` |
| `OCSF_COMPILED_ARTIFACT_CHECKSUM_MISMATCH` | `OCSF_ARTIFACT_MISMATCH` | `050` |
| `FEED_CATEGORY_CLOSURE_ROW_MISSING` | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | `020` |
| `SOURCE_DATASET_CATALOG_MISSING` | `SOURCE_DATASET_CATALOG_ROW_MISSING` | `060` |

### OwnerErrorFragmentCompletionRequirement

Owner fragments are incomplete unless every exported owner error code can generate one `ErrorCodeRegistryRow` through the schema above.

| owner_spec | fragment_required | required_fields | TODO_behavior | failure_code |
| --- | --- | --- | --- | --- |
| `010` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, generic substitute, legacy `code` caller field, or wildcard-only fixture ref fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `000` | true when governance errors are visible in validation, promotion, acceptance, API, health, audit, or telemetry diagnostics | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, generic substitute, legacy `code` caller field, wildcard-only fixture ref, or duplicate with `030.ACTIVATION_ARTIFACT_OWNER_MISMATCH` fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `020` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `030` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `040` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `050` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `060` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `070` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `080` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `090` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `100` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `110` | true for shared API/security rows | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, wildcard-only fixture ref, or owner-domain code in `Shared110ErrorRegistryFragment` fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `120` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, or generic substitute fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `130` | true when owner exports error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, generic substitute, legacy `code` caller field, or wildcard-only fixture ref fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |
| `140` | true when owner exports telemetry-visible error codes | severity, retry class, caller-visible fields, audit fields, redaction rule, validation fixture refs | Any `TODO`, blank required field, duplicate code, generic substitute, legacy `code` caller field, or wildcard-only fixture ref fails registry generation. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` |

Generated-registry validation for `010`, `030`, `120`, and `140` is required when those owners export visible errors. Generated-registry validation for `130` must include fixture families for `error-registry-130-lineage-facet-policy-missing`, `error-registry-130-lineage-facet-namespace-collision`, `error-registry-130-threat-intel-profile-missing`, `error-registry-130-threat-intel-distribution-unmapped`, `error-registry-130-threat-intel-artifact-checksum-mismatch`, `error-registry-130-registry-custom-property-schema-invalid`, `error-registry-130-registry-classification-authority-forbidden`, `error-registry-130-derived-graph-edge-supporting-facts-required`, `error-registry-130-derived-graph-edge-projection-forbidden`, and `error-registry-130-derived-graph-edge-replay-mismatch`. Missing, duplicate, unredacted, unfixtured, unknown-severity, unknown-retry, or owner-context-incomplete `130` rows must fail registry generation before API, export, health, compliance, audit, or validation output.

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
- Analysis, enrichment, lineage, registry, and derived-edge error context must not expose raw threat-intel values, raw lineage facet bytes, registry custom-property raw values, private activation scopes, artifact payload bytes, backend/query text, or private source binding values unless the caller has the required owner-admin, raw-evidence, or secure-audit permission.

### AuthorizationPermissionMatrix

Authorization defaults to deny. Every endpoint must evaluate authorization before owner execution when the request can reveal object existence, private source binding, raw evidence, audit evidence, package evidence, or graph-derived context. A denial must emit an `AuthorizationDecision` and `AuditEvent`; caller-visible output must not reveal inaccessible object existence.

| Endpoint or operation | Required permission | Raw-evidence permission | No-existence-leak behavior | Audit requirement |
| --- | --- | --- | --- | --- |
| `AssetSearch` | `asset.search` | Not applicable unless raw filters are explicitly permitted. | Omit unauthorized matches and do not expose hidden result counts. | Audit search scope and result redaction counts. |
| `AssetDetail` | `asset.read` for visible asset. | `raw.read` only when `include_raw_payload = true`. | Return `AUTHORIZATION_ERROR` without confirming asset existence when caller lacks visibility. | Audit requested ref and redaction classes. |
| `EvidenceDrillback` | `evidence.read`. | `raw.read` for raw bytes; metadata-only otherwise. | Hide inaccessible chain members and emit most-specific error when lineage is missing. | Audit chain refs and redaction summary. |
| `GraphQuery` | `graph.query` plus profile-specific object visibility. | `raw.read` only for raw evidence expansion. | Redact unauthorized objects after backend candidate materialization and before response emission. | Audit query checksum, profile refs, and derived-view state. |
| `OperationalHealthStatus` | `health.read`. | Not applicable. | Redact private refs from diagnostics unless caller has operator/admin diagnostic permission. | Audit requested health scope. |
| `ComplianceExport` | `compliance.export`. | Evidence refs require `evidence.read`; raw bytes remain forbidden unless explicitly permitted. | Unauthorized scope returns `AUTHORIZATION_ERROR` without row enumeration. | Audit export checksum and non-pass/fail counts. |
| `AuditExport` | `audit.export`. | Secure diagnostics require `audit.secure_diagnostics`. | Unauthorized scope returns `AUTHORIZATION_ERROR` without event enumeration. | Audit the audit export request and checksum. |
| `AnalysisReadOnly` | `analysis.read` plus graph/query permissions when graph-backed. | Raw expansion requires `raw.read`. | Mutation attempts fail before authorization-dependent output. | Audit rule bundle, compatibility refs, and redaction summary. |
| `StructuredInputRepositoryRead` | `structured_input.repository.read` | Not applicable. | Redact private repository identity and do not reveal inaccessible repository existence. | Audit repository profile ref, snapshot ref, and redaction summary. |
| `StructuredInputRepositoryWrite` | `structured_input.repository.write` | Not applicable. | Deny without exposing private repository route or file names. | Audit write scope and proposal ref. |
| `StructuredInputChangeReview` | `structured_input.change.review` | Not applicable. | Deny without exposing reviewer-only diagnostics. | Audit proposal ref, reviewer decision, and validation refs. |
| `StructuredInputChangeMerge` | `structured_input.change.merge` | Not applicable. | Deny without exposing private branch names or protected ref details. | Audit merge decision and revalidation requirement. |
| `StructuredInputValidationRun` | `structured_input.validation.run` | Not applicable. | Redact raw file bytes and private diagnostics. | Audit snapshot ref, matrix refs, and diagnostic redaction summary. |
| `StructuredInputMaterialization` | `structured_input.materialize` | Not applicable. | Deny without exposing artifact payload paths. | Audit materialization result refs and package release refs. |
| `StructuredInputPromotion` | `package.promote` plus owner-specific structured-input permission | Not applicable. | Use package no-existence-leak and redaction rules. | Audit package-set candidate and source materialization refs. |
| `StructuredInputRollback` | existing package rollback permission only | Not applicable. | Mutable Git rollback target fails before existence leak. | Audit immutable package-set target or mutable-ref rejection. |

### AuthorizationDecision schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `authorization_decision_ref` | Yes | None. | Deterministic ref over caller, endpoint, permission, scope, request checksum, and policy refs. |
| `caller_ref` | Yes | None. | Service principal or user ref; private identity attributes are audit-only. |
| `endpoint` | Yes | None. | One endpoint token from `EndpointOutcomeMatrix`. |
| `permission` | Yes | None. | Permission token selected from `AuthorizationPermissionMatrix`. |
| `decision` | Yes | `deny` when no row matches. | Closed enum: `allow`, `deny`, `redact_only`. |
| `no_existence_leak_applied` | Yes | `false`. | `true` when denial or redaction hides object existence or counts. |
| `policy_refs` | Yes | None. | Canonically sorted authorization policy refs and checksums. |
| `decision_checksum` | Yes | None. | SHA-256 over canonical decision bytes excluding display-only labels. |

### RedactionDataClassMatrix

| Data class | Default caller-visible behavior | Permission or policy required for display | Audit-visible behavior | Required redaction behavior |
| --- | --- | --- | --- | --- |
| `raw_payload_bytes` | Hidden. | `raw.read` and endpoint policy allowing raw byte display. | Hash and byte count only unless secure audit policy permits more. | Never inline in `EvidenceRef`, graph properties, errors, or page tokens. |
| `raw_payload_hashes` | Visible when evidence metadata is visible. | `evidence.read`. | Visible. | Hashes may be displayed but must not imply raw-byte access. |
| `evidence_refs` | Visible when object is visible. | `evidence.read`. | Visible. | Redact inaccessible chain members without leaking existence. |
| `private_source_bindings` | Hidden. | No public display permission. | Redacted ref/checksum only unless private implementation audit permits. | Fail with `PRIVATE_BINDING_LEAK` when unredacted in public artifacts. |
| `backend_native_graph_ids` | Forbidden. | No display permission. | Redacted diagnostic only. | Fail with `GRAPH_BACKEND_ID_FORBIDDEN` or `PAGE_TOKEN_INVALID` before response. |
| `source_extension_values` | Hidden unless rule permits display. | Active `050.SourceExtensionFieldRule` and redaction policy. | Redacted value class and field path. | Secret-scan failures render as security errors with value hidden. |
| `package_evidence` | Metadata redacted by default. | Package/admin permission by evidence class. | Evidence refs and checksums; raw SBOM bytes hidden unless policy permits. | Never leak private artifact payloads, repository paths, signer secrets, or unauthorized package evidence. |
| `raw_threat_intel_values` | Hidden. | `analysis.read` plus owner redaction policy allowing display. | Redacted value class, artifact ref, and checksum only unless secure audit policy permits more. | Hide raw indicators, sightings, taxonomy values, galaxies, object-template values, and distribution raw values in caller-visible errors and exports. |
| `raw_lineage_facet_bytes` | Hidden. | Owner-admin or secure audit policy only. | Facet ref, schema checksum, facet checksum, and byte count. | Never inline raw facet bytes in API responses, errors, page tokens, graph properties, or evidence refs. |
| `registry_custom_property_values` | Hidden unless the active schema redaction class allows display. | Active `130.RegistryCustomPropertySchema` and caller permission. | Property path, schema ref, redaction class, and checksum. | Redact values for private, raw-derived, bounded-secret, or undeclared properties. |
| `private_activation_scope` | Hidden. | No public display permission. | Redacted ref/checksum only unless private implementation audit permits. | Public artifacts and API responses must not expose concrete activation scopes or private bindings. |
| `artifact_payload_bytes` | Hidden. | Secure artifact-admin permission and owner policy. | Hash, byte count, artifact class, and package-set ref. | Never inline registry payload bytes, package payload bytes, schema payload bytes, or threat-intel artifact bytes in caller-visible output. |
| `backend_query_text` | Hidden. | Owner-admin validation context only. | Query checksum, translation profile ref, and redacted text ref. | Hide provider-native query text unless the caller has owner-admin permission and the query is validation-only or translated. |
| `telemetry_trace_context` | Hidden; expose only opaque `diagnostic_correlation_ref`. | `140.TelemetryCorrelationPolicy` plus caller permission. | Trace/span refs may be audit-visible only in redacted/hashed form when policy permits. | Never expose raw trace/span IDs by default. |
| `telemetry_attribute` | Hidden unless allowlisted. | `140.TelemetryAttributePolicy` and endpoint permission. | Attribute key, redaction class, checksum. | Reject or redact raw payloads, private bindings, credentials, backend IDs, source-native IDs, canonical IDs, hostnames, IPs, usernames, and provider-native query text by default. |
| `telemetry_runtime_state` | Summary only. | `health.read` plus operator/admin diagnostics for detail. | Runtime state refs, checksums, counts, and policy refs. | Do not expose private endpoints, exporter credentials, Collector routes, or raw queue payloads. |
| `inaccessible_asset_existence` | Hidden. | None through normal caller-visible responses. | Audit may record denied ref. | Use no-existence-leak denial and omit counts. |
| `scope_selector_dimension_key` | Redacted bounded key by default. | Diagnostic and audit contexts may show the key when it is not private. | Dimension key and owner context. | Must not expose a private route, tenant, host, account, scanner site, backend ID, or endpoint URL. |
| `scope_selector_public_value` | Visible only when the value kind is public and endpoint authorization permits it. | Visible as value, ref, or checksum according to owner policy. | Public enum tokens, Cadastre IDs, external refs, redacted refs, and SHA-256 values. | Must not include raw private values. |
| `scope_selector_redacted_value_ref` | Visible as opaque ref when policy permits. | Visible as ref and checksum. | Redacted selector value refs. | Must not be reversible from public bytes. |
| `scope_selector_checksum` | Visible when owner diagnostics permit. | Visible. | Normalized selector checksum. | Does not imply access to raw selector values. |
| `scope_selector_private_value` | Forbidden. | Hash and byte count only in secure private validation contexts when policy permits. | Raw private selector value. | Fail before public output, audit output, telemetry export, package report, or validation report. |
| `structured_input_repository_url` | Hidden. | Operator/admin diagnostic policy only. | Redacted ref/checksum. | Must not expose private routes or credentials. |
| `structured_input_branch_name` | Hidden or bounded class only. | Repository admin diagnostic policy. | Redacted value class and checksum. | Raw branch names must not become metric labels or public output. |
| `structured_input_commit_sha` | Redacted or checksum-like diagnostic when policy permits. | Repository admin diagnostic policy. | Ref/checksum only. | Must not become replay, audit, activation, rollback, or source-authority evidence. |
| `structured_input_file_path` | Hidden or hashed. | Repository admin diagnostic policy. | Hashed path and artifact class. | Raw paths must not leak private route, tenant, host, account, or source-native ID values. |
| `structured_input_file_checksum` | Visible only when artifact evidence visibility permits. | Validation/package/admin permission. | Visible as checksum. | Checksum does not imply raw file-byte access. |
| `structured_input_validation_diagnostic` | Redacted summary by default. | Operator/admin diagnostic policy. | Diagnostic class, row ref, and redaction counts. | Must not include raw bytes, secrets, private routes, or raw schema payloads. |
| `private_repository_route` | Forbidden in public output. | No public display permission. | Redacted ref/checksum only. | Fail with `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` or `PRIVATE_BINDING_LEAK` if unredacted. |
| `private_repository_secret` | Forbidden. | No display permission. | Redacted secret class only. | Fail before response, audit export, telemetry export, package report, or validation report. |
| `private_structured_input_payload` | Forbidden. | Secure private validation context only outside public artifacts. | Hash and byte count only when permitted. | Raw structured input bytes must not be exposed in public API, audit, telemetry, errors, or package reports. |

`RedactionPolicy` must apply this matrix after owner result materialization and before checksum-visible response output. Redaction must not change owner state labels, error codes, or version-manifest refs; it may only remove or replace fields whose data class requires hiding.

### StructuredInputRepositoryApiSecurity

Structured-input repository operations must use `CommonApiResponseEnvelope`. Public responses may show redacted repository profile ID, snapshot ID, file manifest checksum, validation status, materialization status, package release refs, package-set refs, and owner error codes. Public responses must not show raw file bytes, credentials, private routes, private source bindings, raw private fixture bytes, raw private schema payloads, unredacted repository URLs, raw branch names, raw file paths, or commit messages containing private data.

`StructuredInputRepositoryAccessPolicy` defines clone, push, review, merge, validation, materialization, promotion, rollback, audit, and secure diagnostic permissions. Deny is the default. A policy row must not grant production activation; package activation remains owned by `100`.

`StructuredInputRepositoryRedactionPolicy` defines forbidden and redacted fields for repository content, validation output, audit output, API output, telemetry, package reports, and `VersionManifest` rendering. Redaction must occur before response checksums, audit export, validation report materialization, package report materialization, and telemetry export.

Endpoint outcomes for structured-input operations must include success, empty, unauthorized, redaction-only, stale validation, private leak, mutable ref, materialization failure, package activation failure, and audit-only diagnostic outcomes.

| Request or response | Required fields | Forbidden fields |
| --- | --- | --- |
| `StructuredInputRepositoryRequest` | operation token, repository profile ref, optional snapshot ref, selected artifact class, request checksum, authorization scope | raw credentials, raw private route, raw file bytes |
| `StructuredInputRepositoryResponse` | envelope, redacted profile ref, snapshot ref, file manifest checksum, validation status, materialization status, package release refs, redaction summary | raw file bytes, raw private paths, raw secrets, unredacted private schema payloads |
| `StructuredInputChangeProposalRequest` | proposal ref, expected prior snapshot ref, selected paths, review action, idempotency key | production activation target, mutable rollback target |
| `StructuredInputValidationRequest` | exact snapshot ref, validation matrix refs, selected path refs, redaction policy ref | branch tip, tag, PR ref, repository URL as validation target |

`AuditEvent.operation` must include `structured_input.repository.read`, `structured_input.repository.write`, `structured_input.change.review`, `structured_input.change.merge`, `structured_input.validation.run`, `structured_input.materialize`, `structured_input.promote`, and `structured_input.rollback`. Audit records must include authorization decision ref, redaction context ref, repository profile ref, snapshot ref when present, validation refs when present, materialization refs when present, package-set refs when present, and no raw repository content.

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

### DefineOnceClosureErrorRows

Define-once closure failures are caller-visible validation and promotion failures. They must not be rendered as domain facts, source absence, compliance pass/fail, cleanup, graph expiry, retraction, watermark advancement, or risk reduction.

| Error code | Owner | Severity | Retry class | Caller-visible context | Audit-visible context |
| --- | --- | --- | --- | --- | --- |
| `DOMAIN_LEDGER_OWNER_DUPLICATE` | `000`, validated by `120` | blocked | `policy_change_required` | owner spec, contract name, Section 25 row IDs, redaction state | owner contract scope, validation row family, inventory checksum, version manifest ref |
| `OWNER_CONTRACT_REF_UNEXPORTED` | `000`, validated by `120` | blocked | `policy_change_required` | owner spec, referenced contract name, referring file class, redaction state | referring path, heading, expected owner, export inventory checksum |
| `DUPLICATE_OWNER_EXPORT` | `000`, validated by `120` | blocked | `policy_change_required` | duplicate contract name, owner specs, redaction state | owner file paths, export rows, alias status, inventory checksum |

### Core one-of and evidence artifact error rows

Closed one-of failures are observable core schema failures and must not be rendered as generic validation errors. `CORE_ONE_OF_INVALID` applies when `GoldFact.subject_ref`, `GoldFact.object_value`, `EvidenceRef.artifact_id`, `GraphNodeDeltaShape.source_object_ref`, or `GraphEdgeDeltaShape.source_object_ref` has unknown kind, missing matching value member, extra value member, forbidden null, omission, or mismatched value member. `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` applies when `EvidenceRef.artifact_class` does not permit the observed `artifact_id.kind` under `040.EvidenceArtifactClassRegistry`.

| Error condition | Required error code | Caller-visible fields | Audit-visible fields | Redaction rule | Retry class |
| --- | --- | --- | --- | --- | --- |
| Malformed `GoldFact.subject_ref` one-of shape | `CORE_ONE_OF_INVALID` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | allowed kind set, observed kind token when non-sensitive, value-member names, input checksum, validation fixture ref, version manifest ref | Raw payload bytes, private bindings, source-native values, backend IDs, and artifact payload bytes must not be exposed. | `caller_correctable` |
| Malformed `GoldFact.object_value` one-of shape | `CORE_ONE_OF_INVALID` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | allowed kind set, observed kind token when non-sensitive, value-member names, input checksum, predicate contract ref, validation fixture ref, version manifest ref | Identity-like raw strings and source-native identity values are redacted unless a specific audit policy permits them. | `caller_correctable` |
| Forbidden `null_value` under predicate contract | `CORE_NULL_FORBIDDEN` when the field is explicit null; `CORE_ONE_OF_INVALID` when `kind = null_value` is not permitted by the selected predicate contract | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | predicate contract ref, null policy, input checksum, validation fixture ref, version manifest ref | No raw object value is exposed. | `caller_correctable` |
| Malformed `EvidenceRef.artifact_id` one-of shape | `CORE_ONE_OF_INVALID` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | allowed kind set, observed kind token, value-member names, artifact class, artifact ID kind, input checksum, validation fixture ref, version manifest ref | Artifact payload bytes and external refs are redacted by evidence policy. | `caller_correctable` |
| `EvidenceRef.artifact_class` and `artifact_id.kind` mismatch | `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | artifact class, observed artifact ID kind, allowed artifact ID kind, artifact checksum, validation fixture ref, version manifest ref | Artifact payload bytes are never exposed; checksums may be audit-visible. | `caller_correctable` |
| Raw payload bytes in evidence ref | `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | artifact class, artifact ID kind, input checksum, validation fixture ref, version manifest ref | Raw payload bytes are rejected and must not appear in caller or audit output. | `none` |
| Backend ID in core-shaped output | `GRAPH_BACKEND_ID_FORBIDDEN` | code, message, severity, retryability, owner spec, affected record type, field path, redaction state, error correlation ID | backend ID class, graph profile ref, input checksum, validation fixture ref, version manifest ref | Backend IDs are hidden from caller and redacted from normal audit. | `none` |

When a missing owner registry ref, inactive owner kind set, or missing evidence artifact class row causes the failure, the generated row must use `policy_change_required` or `retry_after_owner_repair` according to the owner fragment. A generic error code is invalid when one of the specific rows above applies.

The generated error registry must include the following mapping, source-extension, external-schema non-authority, and graph-direction codes.

| Error code | Owner | Severity | Retryability | Redaction |
| --- | --- | --- | --- | --- |
| `OCSF_ARTIFACT_MISMATCH` | `050` | error | no, until schema artifact/profile changes | artifact refs and checksums redacted by policy |
| `OCSF_CLASS_NOT_ALLOWED` | `050` | blocked | no, until profile allowlist changes | class UID visible; source values redacted |
| `EXTERNAL_ENUM_UNKNOWN` | `050` | error | no, until enum rule or source mapping changes | enum path visible; raw value redacted |
| `EXTERNAL_ENUM_OTHER_NOT_PERMITTED` | `050` | error | no, until enum rule changes | enum path visible; raw value redacted |
| `EXTERNAL_ENUM_DEPRECATED` | `050` | blocked | no, until waiver/profile changes | enum path visible |
| `EXTERNAL_ENUM_SIBLING_MISMATCH` | `050` | error | no, until input or mapping changes | enum path visible; raw value redacted |
| `SOURCE_EXTENSION_FIELD_UNDECLARED` | `050` | blocked | no, until source-extension rule activates | source-extension path visible; value redacted |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `060` | error | no, until authority signal row activates | external field path visible; private refs redacted |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` | `060` | error | no, until authority signal rows change | matching row refs redacted |
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
| `130` | analysis/enrichment/lineage/registry | `ErrorRecord` | analysis mutation errors, lineage facet errors, threat-intel enrichment errors, registry governance errors, registry custom-property errors, registry classification errors, and derived graph edge routing errors | owner row | `110` | analysis, enrichment, lineage, registry, and derived-edge rows |
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
| `RESOLVER_DECISION_ROW_AMBIGUOUS` | `070` | error | no, until row set changes | candidate refs and row refs redacted unless authorized | code, severity, retryability, owner, affected record type, redaction state, correlation ID | decision matrix ref, matching row refs, candidate checksum, version manifest ref |
| `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `070` | blocked | no, until row activates | blocker family visible; private scope values redacted | code, blocker family, severity, owner, correlation ID | blocker row-set ref, profile ref, validation ref |
| `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` | `070` | error | no, until row set changes | blocker family visible; matching row refs redacted unless authorized | code, blocker family, severity, owner, redaction state, correlation ID | matching blocker row refs, row-set checksum, profile ref |
| `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `070` | blocked | no, until validation evidence changes | missing scenario class visible; private refs redacted | code, missing scenario class, owner, correlation ID | activation report ref, validation refs |
| `RESOLVER_ACTIVATION_REPORT_FAILED` | `070` | blocked | yes after candidate changes | failed scenario class visible; private refs redacted | code, failed scenario class, owner, correlation ID | activation report ref, failed row refs |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | `070` | error | no, until band row changes | score visible, evidence redacted | code, severity, retryability, owner, correlation ID | confidence band ref, decision row ref, explanation ref |
| `RESOLVER_REVIEW_ROUTING_MISSING` | `070` | error | no, until routing policy changes | candidate refs redacted | code, severity, retryability, owner, correlation ID | routing policy ref, candidate checksum |
| `RESOLVER_SPLIT_POLICY_MISSING` | `070` | error | no, until split policy activates | split refs redacted | code, severity, retryability, owner, correlation ID | split decision ref, resolver explanation ref, version manifest ref |
| `RESOLVER_EXPLANATION_INCOMPLETE` | `070` | error | no, until output or policy changes | missing field names visible; values redacted | code, field path, severity, owner, redaction state, correlation ID | explanation policy ref, output decision ref, checksum inputs |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | `070` | error | yes after reviewer reloads current evidence | evidence values redacted | code, severity, retryability, owner, redaction state, correlation ID | review case ref, expected checksum, supplied checksum |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | `070` | security error | no, until authorization changes | reviewer identity redacted from caller | code, severity, retryability, owner, redaction state, correlation ID | reviewer auth context checksum, case ref, event |
| `TARGET_SELECTOR_UNSAFE` | `070` | security error | no, until selector policy changes | selector value redacted; mechanism visible | code, severity, retryability, owner, redaction state, correlation ID | selector policy ref, unresolved target ref, mechanism |

Source-native identity values, private scope values, blocking key values, candidate IDs, and resolver row refs must be redacted in caller-visible output unless the caller is authorized for the exact data class. Audit output may include refs and checksums only after `RedactionPolicy` permits the class; raw source values remain forbidden.

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

### ActivationCatalogClosureErrorRegistryRows

The generated `ErrorCodeRegistry` must include owner-specific closure-pack errors and must select them before generic API, validation, activation, health, or unknown-state codes.

| Error code | Owner | Severity | Retryability | Redaction | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| `FEED_CATEGORY_CLOSURE_ROW_MISSING` | `020` | blocked | no, until row set changes | category visible; refs redacted | `fixture-020-category-row-missing` |
| `FEED_CATEGORY_CLOSURE_ROW_AMBIGUOUS` | `020` | error | no, until row set changes | matching rows redacted | `fixture-020-category-row-ambiguous` |
| `SOURCE_DATASET_CATALOG_ROW_MISSING` | `020` | blocked | no, until catalog activates | dataset token visible when public; refs redacted | `fixture-020-source-dataset-catalog-missing` |
| `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` | `020` | error | no, until catalog rows change | matching rows redacted unless public | `fixture-020-source-dataset-catalog-ambiguous` |
| `SOURCE_DATASET_CATALOG_ROW_INACTIVE` | `020` | blocked | no, until lifecycle changes | selected row ref and lifecycle status | `fixture-020-source-dataset-catalog-inactive` |
| `SOURCE_DATASET_CATALOG_ROW_CHECKSUM_MISMATCH` | `020` | error | no, until row or manifest changes | selected row ref, expected checksum, actual checksum | `fixture-020-source-dataset-catalog-checksum-mismatch` |
| `SOURCE_DATASET_CATALOG_PRIVATE_BINDING_LEAK` | `020` | security | no | leaked field path only; raw private value forbidden | `fixture-020-source-dataset-private-leak` |
| `SOURCE_DATASET_UNSUPPORTED_FOR_FEED_CATEGORY` | `020` | blocked | no, until catalog or category row changes | feed category, coverage-domain token when public | `fixture-020-source-dataset-unsupported-category` |
| `SOURCE_DATASET_DETERMINISTICALLY_BLOCKED` | `020` | blocked | no, until deterministic block is removed by validated row | deterministic block code and block scope | `fixture-020-source-dataset-deterministically-blocked` |
| `OCSF_COMPILED_ARTIFACT_CHECKSUM_MISMATCH` | `050` | error | no, until artifact changes | checksum policy-visible | `fixture-050-compiled-artifact-checksum-mismatch` |
| `OCSF_MAPPING_ROW_MISSING` | `050` | blocked | no, until row set changes | observation type visible | `fixture-050-mapping-row-missing` |
| `OCSF_MAPPING_ROW_AMBIGUOUS` | `050` | error | no, until row set changes | matching rows redacted | `fixture-050-mapping-row-ambiguous` |
| `SOURCE_AUTHORITY_CLOSURE_BLOCKED` | `060` | blocked | owner-defined | closure refs redacted | `fixture-060-closure-blocked` |
| `RESOLVER_CATALOG_MISSING` | `070` | blocked | no, until artifact activates | artifact class visible | `fixture-070-catalog-missing` |
| `RESOLVER_CATALOG_AMBIGUOUS` | `070` | error | no, until artifact rows change | matching rows redacted | `fixture-070-catalog-ambiguous` |
| `RESOLVER_CATALOG_INACTIVE` | `070` | blocked | no, until lifecycle changes | artifact refs redacted | `fixture-070-catalog-inactive` |
| `PACKAGE_TYPE_POLICY_ROW_MISSING` | `100` | blocked | no, until policy activates | package type visible | `fixture-100-policy-row-missing` |
| `PACKAGE_TYPE_POLICY_ROW_AMBIGUOUS` | `100` | error | no, until policy rows change | matching rows redacted | `fixture-100-policy-row-ambiguous` |
| `PACKAGE_DEPRECATION_ROW_MISSING` | `100` | blocked | no, until row activates | package type visible | `fixture-100-deprecation-row-missing` |
| `PACKAGE_DEPRECATION_ROW_AMBIGUOUS` | `100` | error | no, until rows change | matching rows redacted | `fixture-100-deprecation-row-ambiguous` |
| `GRAPH_ACTIVE_PROFILE_CLOSURE_MISSING` | `090` | blocked | no, until graph profile activates | graph scope visible | `fixture-090-active-profile-closure-missing` |
| `GRAPH_BACKEND_ACTIVATION_BLOCKER_UNRESOLVED` | `090`, `100` | blocked | no, until blocker resolves | blocker family visible | `fixture-090-backend-blocker-unresolved` |
| `ACTIVATION_CATALOG_MANIFEST_OMISSION` | `030` | blocked | no, until manifest changes | refs redacted | `fixture-030-closure-pack-manifest-omission` |
| `ACTIVATION_CATALOG_VALIDATION_BLOCKED` | `120` | blocked_validation | no, until validation passes | validation family visible | `fixture-120-closure-pack-validation-blocked` |
| `DETERMINISTIC_BLOCK_ROW_MUTATION_ATTEMPTED` | owner spec | security_error | no, until implementation changes | row refs redacted | `fixture-120-block-row-mutation-attempted` |
| `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` | `010` | security_error | none | always forbidden sensitive values | `fixture-010-activation-catalog-private-binding-leak` |

### ActivationControlledRowSchemaErrorRegistryRows

The generated `ErrorCodeRegistry` must include the generic activation-row schema errors and every owner-specific row-schema error. Generic activation-row codes must not collapse owner-specific row-family errors when an owner-specific row exists.

| Error code | Owner | Severity | Retryability | Redaction | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| `ACTIVATION_ROW_SCHEMA_INCOMPLETE` | `030` | blocked_validation | no, until owner spec changes | row family visible; row values redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-SCHEMA-INCOMPLETE` |
| `ACTIVATION_ROW_FIELD_TYPE_INVALID` | `030` | error | no, until row changes | field path visible; value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-FIELD-TYPE-INVALID` |
| `ACTIVATION_ROW_NULL_FORBIDDEN` | `030` | error | no, until row changes | field path visible; value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-NULL-FORBIDDEN` |
| `ACTIVATION_ROW_OMIT_FORBIDDEN` | `030` | error | no, until row changes | field path visible | `120-ERROR-REGISTRY-ACTIVATION-ROW-OMIT-FORBIDDEN` |
| `ACTIVATION_ROW_BOUNDS_INVALID` | `030` | error | no, until row changes | field path and bound visible; value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-BOUNDS-INVALID` |
| `ACTIVATION_ROW_ARRAY_SEMANTICS_MISSING` | `030` | blocked_validation | no, until owner spec changes | field path visible | `120-ERROR-REGISTRY-ACTIVATION-ROW-ARRAY-SEMANTICS-MISSING` |
| `ACTIVATION_ROW_DUPLICATE_FORBIDDEN` | `030` | error | no, until row set changes | duplicate key checksum visible; raw value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-DUPLICATE-FORBIDDEN` |
| `ACTIVATION_ROW_REF_INVALID` | `030` | error | no, until ref changes | ref class visible; private ref values redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-REF-INVALID` |
| `ACTIVATION_ROW_CHECKSUM_MISMATCH` | `030` | error | no, until row set changes | checksums visible | `120-ERROR-REGISTRY-ACTIVATION-ROW-CHECKSUM-MISMATCH` |
| `ACTIVATION_ROW_UNKNOWN_FIELD` | `030` | error | no, until row changes | field path visible; value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-UNKNOWN-FIELD` |
| `ACTIVATION_ROW_EXTENSION_FORBIDDEN` | `030` | security_error | no, until row changes | extension path visible; value redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-EXTENSION-FORBIDDEN` |
| `ACTIVATION_ROW_VERSION_MANIFEST_INCOMPLETE` | `030` | blocked | no, until manifest changes | missing ref class visible; refs redacted | `120-ERROR-REGISTRY-ACTIVATION-ROW-VERSION-MANIFEST-INCOMPLETE` |

Every owner-specific activation row schema error must generate exactly one `ErrorCodeRegistryRow`. If the generated registry lacks an owner row for a missing or invalid field declared by an owner `030.ActivationControlledRowField` table, registry generation must fail with `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE`.

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
| `COVERAGE_DOMAIN_REQUIRED` | `060` | error | no, until request or row changes | token field path visible; private refs redacted | `coverage-domain-required` |
| `COVERAGE_DOMAIN_TOKEN_INVALID` | `060` | error | no, until request or row changes | invalid token redacted unless value is public display label | `coverage-domain-token-invalid` |
| `COVERAGE_DOMAIN_ALIAS_REJECTED` | `060` | error | no, until row changes | alias value visible only when non-private display label | `coverage-domain-alias-rejected` |
| `COVERAGE_DOMAIN_UNKNOWN` | `060` | blocked | no, until catalog changes | token visible when public | `coverage-domain-unknown` |
| `COVERAGE_DOMAIN_DUPLICATE` | `060` | error | no, until row changes | duplicate token visible when public | `coverage-domain-duplicate` |
| `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` | `060` | blocked | no, until feed-category row changes | feed category and token visible when public | `coverage-domain-feed-category-unsupported` |
| `COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT` | `060` | blocked | no, until row changes | effect visible; private scope redacted | `coverage-domain-required-for-effect` |
| `COVERAGE_DOMAIN_INACTIVE_DEFERRED` | `060` | diagnostic or blocked by context | no, until `200` promotion | token visible | `coverage-domain-reachability-deferred` |
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
| `SOURCE_AUTHORITY_CLOSURE_BLOCKED` | `060` | blocked | no, until closure rows change | closure refs redacted | `deterministic-block-row-blocked-effect` |
| `SOURCE_DATASET_CATALOG_ROW_MISSING` | `060` | blocked | no, until catalog row activates | dataset token visible; private refs redacted | `feed-category-closure-catalog-source-dataset-missing` |
| `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` | `060` | error | no, until catalog rows change | matching rows redacted | `feed-category-closure-catalog-source-dataset-ambiguous` |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `060` | error | no, until signal row activates | external field path visible; private refs redacted | `external-schema-authority-signal-missing` |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` | `060` | error | no, until signal rows change | matching rows redacted | `external-schema-authority-signal-ambiguous` |
| `SOURCE_HISTORY_OUTSIDE_WINDOW_NO_PROOF` | `060` | diagnostic | no | source-history scope redacted | `source-history-coverage-outside-window-no-proof` |
| `SOURCE_HISTORY_COVERAGE_ROW_MISSING` | `060` | blocked | no, until coverage row activates | coverage selector redacted | `source-history-coverage-missing` |
| `DETERMINISTIC_BLOCK_ROW_SELECTED` | `060` | diagnostic | no | block refs redacted | `deterministic-block-row-selected` |

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

### GoldPredicateCatalogErrorRegistryRows

The generated registry must include every `080` gold predicate-catalog error below. Generic validation, graph, mapping, API, or activation errors must not substitute for these codes when the selected failure is predicate-catalog-owned.

| Error code | Owner | Severity | Retryability | Redaction | Caller-visible behavior | Audit-visible fields | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `GOLD_FACT_PREDICATE_CONTRACT_MISSING` | `080` | blocked | no, until row activates | fact type and predicate visible when public; refs redacted | block gold output before fact ID computation | row-set refs, candidate fact type, predicate | `val-080-gold-predicate-row-missing` |
| `GOLD_FACT_PREDICATE_CONTRACT_AMBIGUOUS` | `080` | error | no, until row set changes | matching rows redacted | block gold output before fact ID computation | matching row refs/checksums | `val-080-gold-predicate-row-ambiguous` |
| `GOLD_FACT_PREDICATE_CONTRACT_BLOCKED` | `080` | blocked | no, until block row changes | block row ref redacted | explicit no-output diagnostic only | block row ref, mutation-prohibition proof | `val-080-gold-predicate-block-row-reachability` |
| `GOLD_FACT_PREDICATE_SUBJECT_KIND_FORBIDDEN` | `080` | error | no, until candidate or row changes | subject ref redacted | block before fact ID computation | selected row ref, subject kind | `val-080-gold-predicate-subject-kind-boundary` |
| `GOLD_FACT_PREDICATE_OBJECT_KIND_FORBIDDEN` | `080` | error | no, until candidate or row changes | object value redacted | block before fact ID computation | selected row ref, object kind | `val-080-gold-predicate-object-kind-boundary` |
| `GOLD_FACT_PREDICATE_REFERENCE_ENTITY_TYPE_FORBIDDEN` | `080` | error | no, until candidate or resolver row changes | entity ref redacted | block before fact ID computation | selected row ref, entity type | `val-080-gold-predicate-reference-eligibility` |
| `GOLD_FACT_PREDICATE_REFERENCE_IDENTIFIER_TYPE_FORBIDDEN` | `080` | error | no, until candidate or resolver row changes | identifier ref redacted | block before fact ID computation | selected row ref, identifier type | `val-080-gold-predicate-reference-eligibility` |
| `GOLD_FACT_PREDICATE_NULL_FORBIDDEN` | `080` | error | no, until candidate or row changes | value redacted | block before fact ID computation | selected row ref, null policy | `val-080-gold-predicate-null-forbidden` |
| `GOLD_FACT_PREDICATE_IDENTITY_STRING_FORBIDDEN` | `080` | error | no, until candidate changes | raw string forbidden; classification visible | block before fact ID computation | selected row ref, string class | `val-080-gold-predicate-identity-like-string-rejected` |
| `GOLD_FACT_STRUCTURED_SCHEMA_MISSING` | `080` | blocked | no, until schema row activates | schema ref visible when public | block before fact ID computation | selected predicate row, required schema ref | `val-080-gold-predicate-structured-schema-required` |
| `GOLD_FACT_STRUCTURED_SCHEMA_CHECKSUM_MISMATCH` | `080` | error | no, until schema or manifest changes | checksums policy-visible | block before fact ID computation or replay output | expected and actual checksums | `val-080-gold-predicate-replay-checksum` |
| `GOLD_FACT_PREDICATE_ROW_TODO` | `080` | blocked_validation | no, until owner spec or row material changes | row family visible; row values redacted | block validation and promotion | row family, row ref | `val-080-gold-predicate-rowset-total-mvp` |
| `GOLD_FACT_PREDICATE_PACKAGE_SET_MISSING` | `080`, `100` | blocked | no, until package set activates | package refs redacted | block package activation and dependent output | package release refs, row-set ref | `val-080-gold-predicate-rowset-package-set` |
| `GOLD_FACT_PREDICATE_MANIFEST_INCOMPLETE` | `080`, `030` | blocked | retry after manifest repair | missing ref class visible; refs redacted | block visible output or replay | missing refs, manifest ref | `val-030-gold-predicate-manifest-completeness` |

### GoldPredicateCatalogErrorAcceptance

| ID | Criterion |
| --- | --- |
| `110-GOLD-PREDICATE-ERROR-AC-001` | `GenerateErrorCodeRegistry` emits exactly one generated row for every `080` predicate-catalog code in `GoldPredicateCatalogErrorRegistryRows`; missing, duplicate, wildcard-fixtured, TODO-bearing, or generic-substitute rows fail registry generation. |

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

`PackageActivationErrorObservableMapping` must be generated from `100.PackageErrorRegistryFragment` or must be validated as an exhaustive hand-maintained table whose `Error code` set exactly equals the `100` owner fragment. Missing package codes, extra unowned package codes, severity drift, retryability drift, redaction drift, or missing fixture refs fail before API, export, health, audit, or validation output.

Caller-visible package error fields are `error_code`, owner, severity, retryability, affected package-set checksum or redacted ref class, redaction state, and error correlation ID. Audit-visible fields may include package type policy row refs, package type policy row-set refs, target environment, lifecycle status, activation scope, legacy label class, repository metadata refs, repository snapshot refs, trust refs, signature verification refs, transparency refs, attestation refs, provenance refs, SBOM refs, dependency lock refs, compatibility refs, rollback refs, quarantine refs, emergency refs, manifest refs, lifecycle refs, missing ref classes, and validation refs.

Artifact payload locations, signer secrets, private repository paths, raw SBOM contents, private source bindings, unauthorized package evidence, and private package registry credentials must not be caller-visible.

| Error code | Owner | Severity | Retryability | Caller-visible behavior | Audit-visible refs |
| --- | --- | --- | --- | --- | --- |
| `PACKAGE_TYPE_UNKNOWN` | `100` | error | caller_correctable | activation blocked; current active set preserved | release manifest ref, supplied token redacted when private, `legacy_label_class` when supplied token is a broad label |
| `PACKAGE_TYPE_POLICY_MISSING` | `100` | blocked | policy_change_required | activation blocked | package type, target environment, policy row-set ref, lifecycle status when inactive, activation scope when scope-mismatched, deprecation policy ref status when policy row is not active |
| `PACKAGE_TYPE_POLICY_AMBIGUOUS` | `100` | error | retry_after_owner_repair | activation blocked | matching policy row refs when authorized |
| `PACKAGE_ACTIVATION_ARTIFACT_MISSING` | `100` | blocked | policy_change_required | activation blocked before candidate output | missing artifact class, package-set ref, validation row ref |
| `PACKAGE_ACTIVATION_ARTIFACT_OWNER_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked before candidate output | artifact class, expected owner, actual owner, policy-bundle substitution class |
| `PACKAGE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked before candidate output | artifact ref, expected checksum ref, observed checksum ref |
| `PACKAGE_ACTIVATION_ARTIFACT_SCOPE_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked before candidate output | artifact ref, target environment, activation scope |
| `PACKAGE_ACTIVATION_ARTIFACT_VALIDATION_MISSING` | `100` | blocked | policy_change_required | activation blocked before candidate output | artifact ref, missing validation refs |
| `PACKAGE_ACTIVATION_ARTIFACT_CORE_CONFLICT` | `100` | security_error | none | activation blocked; stable core conflict reported | artifact ref, owner spec, conflicting stable contract ref |
| `PACKAGE_SET_CHECKSUM_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked; current active set preserved | package-set manifest ref, release checksum refs, expected package-set checksum ref |
| `PACKAGE_COHESION_INCOMPLETE` | `100` | blocked | policy_change_required | activation blocked; current active set preserved | cohesion group ref, missing release manifest classes |
| `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | `100` | error | caller_correctable | activation blocked | release manifest ref, repository form |
| `PACKAGE_REPOSITORY_ROLLBACK_DETECTED` | `100` | security_error | none | activation blocked | anti-rollback state ref, metadata refs |
| `PACKAGE_REPOSITORY_METADATA_EXPIRED` | `100` | blocked | retry_after_refresh | activation blocked | metadata refs, expiration evidence |
| `PACKAGE_SIGNER_UNAUTHORIZED` | `100` | security_error | none | activation blocked | signature verification result, signer ref redacted by policy |
| `PACKAGE_TRUST_ROOT_INACTIVE` | `100` | security_error | none | activation blocked | trust root ref, policy row ref |
| `PACKAGE_SIGNATURE_THRESHOLD_FAILED` | `100` | security_error | none | activation blocked | threshold row, verification refs |
| `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | `100` | blocked | policy_change_required | activation blocked | transparency policy row ref, release subject digest ref |
| `PACKAGE_ATTESTATION_MISSING` | `100` | blocked | policy_change_required | activation blocked | attestation policy row ref |
| `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked | attestation ref, expected subject digest ref |
| `PACKAGE_PROVENANCE_POLICY_FAILED` | `100` | error | retry_after_owner_repair | activation blocked | provenance policy row ref, build provenance ref |
| `PACKAGE_SBOM_MISSING` | `100` | blocked | policy_change_required | activation blocked | SBOM policy row ref |
| `PACKAGE_SBOM_SUBJECT_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked | SBOM ref, expected subject digest ref |
| `PACKAGE_SBOM_POLICY_FAILED` | `100` | error | retry_after_owner_repair | activation blocked | SBOM policy row ref, SBOM ref |
| `PACKAGE_DEPENDENCY_LOCK_MISSING` | `100` | blocked | policy_change_required | activation blocked | dependency lock policy row ref |
| `PACKAGE_DEPENDENCY_LOCK_MISMATCH` | `100` | error | retry_after_owner_repair | activation blocked | lock checksum refs, dependency digest refs |
| `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | `100` | security_error | none | activation blocked | attempted dependency source redacted |
| `PACKAGE_COMPATIBILITY_FAILED` | `100` | blocked | policy_change_required | activation or rollback blocked | compatibility row refs and failed axis |
| `PACKAGE_VALIDATION_FAILED` | `100` | blocked | policy_change_required | activation blocked; candidate output invisible | failed validation row refs, fixture refs |
| `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | `100` | blocked | policy_change_required | output rejected before visibility | missing package manifest ref classes, missing `VersionManifest.included_refs` classes, package policy row refs |
| `PACKAGE_LKG_HEALTH_GATE_FAILED` | `100` | blocked | policy_change_required | active-but-not-last-known-good | health gate ref, failed output class refs |
| `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | `100` | error | retry_after_owner_repair | lifecycle transition rejected; active set unchanged | lifecycle machine ref, transition evidence ref |
| `PACKAGE_ACTIVATION_IDEMPOTENCY_CONFLICT` | `100` | error | retry_after_owner_repair | duplicate activation request rejected; active set unchanged | idempotency key ref, candidate checksum refs |
| `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | `100` | security_error | none | canary output rejected before current production visibility | canary package-set ref, attempted output class |
| `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | `100` | security_error | none | shadow output rejected before current production visibility | shadow package-set ref, attempted output class |
| `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | `100` | blocked | policy_change_required | rollback blocked; active set unchanged | rollback target ref, verification refs, mutable target class |
| `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | `100` | error | retry_after_owner_repair | rollback blocked; active set unchanged | rollback policy ref, state refs |
| `PACKAGE_ROLLBACK_REPLAY_INPUT_INSUFFICIENT` | `100` | blocked | retry_after_refresh | rollback blocked; active set unchanged | replay input refs, manifest refs |
| `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | `100` | error | retry_after_owner_repair | rollback blocked; active set unchanged | graph rebuild or delta reapply refs |
| `PACKAGE_ROLLBACK_TRUST_INCOMPATIBLE` | `100` | error | retry_after_owner_repair | rollback blocked; active set unchanged | trust refs, rollback trust-allow refs |
| `PACKAGE_ROLLBACK_QUARANTINE_BLOCKED` | `100` | blocked | policy_change_required | rollback blocked; active set unchanged | quarantine record refs |
| `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | `100` | blocked | policy_change_required | activation blocked | quarantine target refs and scope policy refs |
| `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | `100` | blocked | policy_change_required | new activation blocked | deprecation policy row ref, release refs |
| `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | `100` | security_error | none | no activation output and current active set preserved | emergency override record ref, attempted bypass class |
| `PACKAGE_RETRY_NOT_ALLOWED` | `100` | security_error | none | retry rejected; active set unchanged | prior failure ref, retry request checksum, candidate checksum |

### GraphQueryErrorRegistryRows

| Error code | Owner | Severity | Retryability | Caller-visible behavior | Audit-visible refs |
| --- | --- | --- | --- | --- | --- |
| `GRAPH_EDGE_SEMANTICS_ROW_MISSING` | `090` | error | no, until graph profile changes | graph output blocked | graph profile ref, edge type, validation refs |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `090`, with identity evidence from `070` | diagnostic or error by query class | yes after identity resolution | no endpoint node or dependent edge | unresolved target refs, redacted identity decision context |
| `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | `090` | error | yes after narrower query | no partial output unless query class permits | query checksum, candidate limit, profile refs |
| `GRAPH_PAGE_TOKEN_EXPIRED` | `090`, observable handling by `110` | error | yes with fresh request | no backend query execution | token checksum, expiry, request checksum |
| `GRAPH_PAGE_TOKEN_INVALID` | `090`, observable handling by `110` | security | no for supplied token | no backend query execution and no existence leak | token checksum, mismatch class, authorization checksum |
| `GRAPH_TRAVERSAL_CLASS_REQUIRED` | `090`, observable handling by `110` | error | yes with corrected request | request rejected before backend query | query class and request checksum |
| `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `090` | blocked | no, until profile changes | graph backend preflight blocked | profile ref, missing field path, validation refs |
| `GRAPH_BACKEND_DEFAULT_UNRESOLVED` | `090` | blocked | no, until product governance resolves default | graph backend default cannot materialize for production | selection policy ref, unresolved field paths, implementation scope |
| `GRAPH_PROVIDER_CAPABILITY_MISSING` | `090` | blocked | no, until capability matrix changes | backend activation blocked | capability matrix ref, backend profile ref |
| `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` | `090` | blocked | no, until profile or capability rows change | unsupported query/apply/rebuild capability blocked | capability row, provider ref, affected class |
| `GRAPH_BACKEND_VERSION_UNPINNED` | `090` | blocked | no, until version/package refs change | backend activation blocked | provider package refs |
| `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | `090`, evidence from `100` | blocked | yes after owner repair | graph backend blocked by package activation gate | package-set ref, gate result ref |
| `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` | `090` | blocked | no, until policy changes | backend schema preflight blocked | schema profile ref, provider config checksum |
| `GRAPH_STORAGE_MODE_UNSAFE` | `090` | blocked | no, until profile changes | backend mutation/query blocked | storage mode, profile ref |
| `GRAPH_INDEX_BACKEND_UNAVAILABLE` | `090` | blocked | yes after owner repair or provider recovery | affected query classes blocked | index backend ref, freshness check ref |
| `GRAPH_INDEX_FRESHNESS_REQUIRED` | `090` | blocked | yes after refresh | affected query classes blocked | index consistency ref |
| `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` | `090` | error | yes with narrower query or index policy | query rejected before backend traversal | query class, translation profile ref |
| `GRAPH_PROVIDER_ADAPTER_UNSUPPORTED` | `090` | blocked | no, until adapter/profile changes | backend profile activation blocked | adapter ref, provider ref |

| `GRAPH_POSTGRES_SCHEMA_FINGERPRINT_STALE` | `090` | blocked | no, until schema evidence refresh | graph backend blocked | backend schema fingerprint ref, schema profile ref |
| `GRAPH_POSTGRES_DUPLICATE_NODE_ID` | `090` | error | no, until data/profile repair | graph apply/query blocked for affected object | graph node ID checksum, profile ref |
| `GRAPH_POSTGRES_DUPLICATE_EDGE_ID` | `090` | error | no, until data/profile repair | graph apply/query blocked for affected object | graph edge ID checksum, profile ref |
| `GRAPH_POSTGRES_ORPHAN_EDGE` | `090` | error | no, until data/profile repair | edge omitted and apply/query blocked as specified by `090` | edge checksum, endpoint refs |
| `GRAPH_POSTGRES_QUERY_TIMEOUT` | `090` | error | yes under bounded retry or narrower query | no partial output unless query class permits | query checksum, timeout, profile refs |
| `GRAPH_POSTGRES_QUERY_PLAN_UNSUPPORTED` | `090` | error | yes with narrower query or index/profile change | query rejected before backend execution | query class, plan class ref, translation profile ref |
| `GRAPH_POSTGRES_DIRECT_DML_FORBIDDEN` | `090` | security_error | none | no mutation and no graph serving effect | role ref, attempted operation class |
| `GRAPH_POSTGRES_SEARCH_PATH_UNSAFE` | `090` | security_error | none | backend preflight blocked | search-path preflight ref |
| `GRAPH_POSTGRES_RLS_BYPASS_FORBIDDEN` | `090` | security_error | none | backend preflight blocked | role/RLS preflight ref |
| `GRAPH_PROVIDER_UNSUPPORTED` | `090` | blocked | no, until provider/profile changes | backend activation blocked | provider support evidence ref |
| `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` | `090` | blocked | no, until evidence supplied | backend activation blocked | provider support evidence requirement ref |
| `GRAPH_AGE_EXTENSION_MISSING` | `090` | blocked | no, until extension refs change | AGE activation blocked | extension package/version refs |
| `GRAPH_AGE_EXTENSION_VERSION_UNSUPPORTED` | `090` | blocked | no, until extension/profile changes | AGE activation blocked | extension version ref, PostgreSQL version ref |
| `GRAPH_AGE_MUTATION_FORBIDDEN` | `090` | security_error | none | no mutation and no query result | query checksum, mutation class |
| `GRAPH_AGE_NAMESPACE_BYPASS_FORBIDDEN` | `090` | security_error | none | no mutation and AGE activation blocked | namespace preflight ref |
| `GRAPH_AGE_INTERNAL_ID_FORBIDDEN` | `090` | security_error | none | output rejected before visibility | rejected ID class, response field path |
| `GRAPH_BACKEND_RESTORE_UNVERIFIED` | `090` | blocked | no, until restore proof changes | backend promotion blocked | restore rehearsal ref |
| `GRAPH_BACKEND_UPGRADE_UNVERIFIED` | `090` | blocked | no, until upgrade proof changes | backend promotion blocked | upgrade rehearsal or rebuild migration ref |
| `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` | `110` | error | `policy_change_required` | output wording rejected | output checksum and owner policy refs |

### ReachabilityWordingGuard

Public API, graph query, health, audit, compliance, and analysis responses must not emit unqualified `reachable`, `not reachable`, `reachability`, `service access`, or `lateral movement path` wording for MVP graph output. Any such wording must fail with `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` unless a future active reachability owner spec permits the exact claim kind.

### Shared110ErrorContext

`Shared110ErrorContext` is the owner context schema for shared `110` API, authorization, page-token, lineage, raw-payload permission, registry-generation, API-bounds, and MVP wording guard registry rows. It satisfies `110.OwnerErrorContextMinimumSchema`.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `110` context schema version. |
| `owner_spec` | Yes | Must be `110`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `authorization`, `lineage`, `page_token`, `api_bounds`, `raw_payload_permission`, `registry_generation`, or `reachability_wording`. |
| `operation` | Yes | API request validation, authorization, page-token validation, evidence drillback, raw-payload permission check, registry generation, or response wording validation. |
| `affected_record_type` | Yes | API request, API response, page token, authorization decision, evidence chain, raw payload ref, owner fragment row, generated registry, or wording output. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to authorization decisions, redaction policies, page-token policies, evidence refs, owner fragments, generated registry artifacts, validation fixtures, response checksums, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `validation_refs` | Yes | Exact `120` API, security, page-token, registry, or wording fixture refs; empty only for runtime authorization denials where `120` explicitly exempts the row. |
| `redaction_classes` | Yes | Map every nested owner-context field path to one `110.ErrorRedactionClassMatrix` class. Raw payload bytes, credentials, private bindings, backend IDs, provider-native query text, source-native values, and raw fixture bytes must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |
| `endpoint_name` | No | Required for endpoint-specific API bounds, authorization, or raw-payload failures. |
| `page_token_context_ref` | No | Required for page-token failures; raw token bytes are forbidden. |
| `owner_fragment_ref` | No | Required for registry-generation failures. |

### Shared110ErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. It contains only shared API/security rows owned by `110`. Owner-domain rows must remain in their owner fragments.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `AUTHORIZATION_ERROR` | `110` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `110.Shared110ErrorContext` | `error-registry-110-authorization-error` |
| `LINEAGE_ERROR` | `110` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `110.Shared110ErrorContext` | `error-registry-110-lineage-error` |
| `PAGE_TOKEN_INVALID` | `110` | `security_error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `110.Shared110ErrorContext` | `error-registry-110-page-token-invalid` |
| `PAGE_TOKEN_EXPIRED` | `110` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `110.Shared110ErrorContext` | `error-registry-110-page-token-expired` |
| `API_BOUNDS_INVALID` | `110` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `110.Shared110ErrorContext` | `error-registry-110-api-bounds-invalid` |
| `RAW_PAYLOAD_PERMISSION_REQUIRED` | `110` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `110.Shared110ErrorContext` | `error-registry-110-raw-payload-permission-required` |
| `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE` | `110` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `110.Shared110ErrorContext` | `error-registry-110-error-registry-owner-fragment-incomplete` |
| `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` | `110` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `110.Shared110ErrorContext` | `error-registry-110-reachability-unqualified-claim-forbidden` |

All owner-specific errors must appear in the generated registry before promotion. A generic shared code must not be selected when an owner-specific code precisely covers the failure.

`source_stale`, `derived_view_stale`, `unknown`, `not_applicable`, `permission_limited`, `source_unavailable`, `scope_unavailable`, `partial_known_gap`, `partial_unknown_gap`, `conflicted`, and `ambiguous` must remain distinct in `state_summary`, evidence drillback, compliance export, and audit export.

Coverage-domain token failures must route to generated `060` owner-specific `ErrorRecord` rows. They must not be rendered as authorized negative facts, compliance pass/fail, graph expiry, cleanup, retraction, reachability output, or watermark advancement.

### GraphBackendOperationalHealthComponents

| Graph health component | Required state refs | Caller-visible behavior | Audit-visible refs |
| --- | --- | --- | --- |
| provider profile | selected backend profile, selection policy, defaulting decision | `healthy`, `degraded`, or `blocked` | profile checksum |
| provider package gates | package-set, release, compatibility, SBOM, provenance gates | block graph serving when failed | package refs redacted by policy |
| schema fingerprint | backend schema fingerprint and schema profile | stale schema labels graph component blocked | fingerprint ref |
| index freshness | index consistency check and affected query classes | stale index blocks affected query classes | index check ref |
| raw-write bypass | bypass evidence | unsafe bypass blocks graph serving | bypass evidence ref |
| PostgreSQL runtime/version | PostgreSQL runtime ref, package ref, provider ref, and compatibility row | `healthy`, `degraded`, `blocked`, or `error` according to selected preflight state | PostgreSQL version ref and package checksum only |
| PostgreSQL adapter/driver package gate | adapter package, driver package, compatibility row, package-set ref | blocked when failed or missing | package refs redacted by policy |
| relational schema fingerprint | relational anchor schema fingerprint, schema profile, constraint/index inventory | blocked when stale or mismatched | fingerprint ref |
| relational constraints/index readiness | constraint/index preflight and query plan class refs | degraded or blocked for affected query classes | preflight refs |
| query translation profile | SQL or SQL/Cypher translation profile and expected checksum refs | blocked when missing, stale, or parity-failed | translation profile ref |
| apply profile and apply-state freshness | apply profile, apply state, idempotency, read-after-write, and derived-view refs | blocked when resume unsafe or stale | apply-state ref |
| provider support evidence | provider, region, engine version, service tier, extension support, restore, and upgrade support refs | blocked when missing or unsupported | provider support ref only |
| AGE extension presence/version | required only when AGE profile selected | blocked when missing or unsupported | extension version ref only |
| AGE graph namespace/security preflight | namespace, role, mutating Cypher, and bypass preflight refs | blocked on unsafe preflight | namespace/security preflight ref, no raw namespace when private |
| role/RLS/search-path readiness | role model, RLS policy, and `search_path` preflight refs | blocked when unsafe | preflight ref |
| restore rehearsal status | restore rehearsal ref and checksum | blocked when missing, stale, failed, or not run | restore ref |
| upgrade rehearsal status | upgrade rehearsal or rebuild migration ref and checksum | blocked when missing, stale, failed, or not run | upgrade ref |
| benchmark status | benchmark row refs when performance gates are in scope | blocked while thresholds are `TODO`, missing, failed, or stale | benchmark refs only |

Graph health output must normalize provider-specific details into Cadastre health states. It must not expose backend-native IDs, PostgreSQL OIDs, tuple IDs, sequence values, AGE IDs, SQL/Cypher text, credentials, private provider configuration, raw package evidence, provider exception class names, database names when private, schema names when private, or AGE graph namespace names when private to callers.

### SourceStateLabelMapping

`SourceStateLabelMapping` is total for every label declared in `Source and Export State Labels`. A label not listed here is invalid. No row may render as compliance pass/fail, authorized absence, remediation, graph expiry, cleanup, retraction, source watermark advancement, or risk reduction unless the owner output separately authorizes that exact effect.

| SourceStateLabel | API label | Compliance export label | Audit label | Graph/query label | Authorized negative interpretation | Default behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `source_stale` | `source_stale` | `stale` | `source_stale` | `source_stale` | No. | Preserve as stale source evidence; do not collapse into derived-view lag. |
| `derived_view_stale` | `derived_view_stale` | reject by default | `derived_view_stale` | `derived_view_stale` | No. | Preserve as graph/read-model lag; do not collapse into source staleness. |
| `unknown` | `unknown` | `unknown` | `unknown` | `unknown` | No. | Display as unknown with owner context when diagnostics are permitted. |
| `not_applicable` | `not_applicable` | `not applicable` | `not_applicable` | not projected by default | No. | Display only when owner row says the fact/control does not apply. |
| `authorized_absent` | `authorized_absent` | owner fact result | `authorized_absent` | owner-defined only | Yes, only when `060.AbsenceDerivationResult.absence_authorized = true` and requested effect, predicate, and scope permit. | Requires exact source-dataset, authority, completeness, coverage, staleness, absence policy, and manifest refs. |
| `authorized_not_observed` | `authorized_not_observed` | owner fact result | `authorized_not_observed` | owner-defined only | Yes, only when `060.AbsenceDerivationResult.absence_authorized = true` and predicate permits. | Requires exact authority, completeness, coverage, and staleness refs. |
| `permission_limited` | `permission_limited` | `unknown` | `permission_limited` | `permission_limited` | No. | Preserve permission-limited state and redact private permission details. |
| `source_unavailable` | `source_unavailable` | `error` or `unknown` by owner export policy | `source_unavailable` | `source_unavailable` | No. | Preserve unavailable source/feed state; no negative fact. |
| `scope_unavailable` | `scope_unavailable` | `unknown` | `scope_unavailable` | `scope_unavailable` | No. | Preserve unavailable scope; no negative fact. |
| `partial_known_gap` | `partial_known_gap` | `unknown` | `partial_known_gap` | `partial_known_gap` | No. | Preserve known gap when owner context is available. |
| `partial_unknown_gap` | `partial_unknown_gap` | `unknown` | `partial_unknown_gap` | `partial_unknown_gap` | No. | Preserve unknown gap; do not treat as complete. |
| `not_checked` | `not_checked` | `not checked` | `not_checked` | `not_checked` | No. | Display only for owner-evaluated not-checked state. |
| `error` | `error` | `error` | `error` | `error` | No. | Emit generated `ErrorRecord`; do not collapse into unknown. |
| `conflicted` | `conflicted` | `conflicted` | `conflicted` | visible only when owner graph eligibility permits | No. | Preserve conflicting assertion state; no pass/fail or cleanup by default. |
| `ambiguous` | `ambiguous` | `ambiguous` | `ambiguous` | no mutation and no path expansion by default | No. | Preserve ambiguity; require owner disambiguation or explicit error. |
| `blocked` | `blocked` | `blocked` | `blocked` | no graph mutation by default | No. | Display owner-controlled blocked validation, activation, lock, or closure state without implying negative evidence. |
| `blocked_validation` | `blocked_validation` | `blocked validation` | `blocked_validation` | no graph mutation by default | No. | Preserve owner validation, fixture, checksum, manifest, or closure blocker; do not collapse into generic blocked when owner context is available. |
| `diagnostic` | `diagnostic` | reject by default | `diagnostic` | diagnostic only | No. | Display only when endpoint context permits operational diagnostics. |

### Conflicted and ambiguous label distinction

| Label | Required meaning | Forbidden default rendering |
| --- | --- | --- |
| `conflicted` | Conflicting evidence or assertion state preserved by the owning domain, including `040.GoldFact.assertion_state = conflicted` or owner conflict output. | Must not render as pass, fail, authorized absence, cleanup, graph expiry, retraction, watermark, remediation, or ambiguity. |
| `ambiguous` | More than one equally specific interpretation, row, selector, result, or state matched. | Must not render as pass, fail, authorized absence, cleanup, graph expiry, retraction, watermark, remediation, or conflict. |

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
| `040.GoldFact.absence_outcome = authorized_absent` | `authorized_absent` | exact source-dataset, authority, completeness, coverage, staleness, absence policy, effect, predicate, scope, and manifest refs | allowed only for the authorized predicate and scope |
| `040.GoldFact.absence_outcome = authorized_not_observed` | `authorized_not_observed` | exact authority, completeness, coverage, and staleness refs | allowed only for the authorized predicate and scope |
| `run_lock_conflict` | `blocked` | lock scope class, output class, redacted lock key ref, and retry class | forbidden |
| `run_lock_lost` | `blocked` | run ID, run attempt ID, output class, and lock evidence refs | forbidden |
| `run_lock_heartbeat_uncertain` | `blocked` or `degraded` by endpoint context | heartbeat evidence ref and assertion retry state | forbidden |
| `run_lock_stale_recovered` | `diagnostic` or `blocked` for old holder | prior and new owner refs redacted, plus fencing token proof | forbidden |
| `run_lock_idempotency_conflict` | `error` | idempotency key checksum and input checksum refs | forbidden |
| `run_lock_fencing_token_stale` | `blocked` | fencing token proof refs | forbidden |
| `blocked_validation` | `blocked_validation` | owner spec, validation family, blocking row, and redacted artifact refs when diagnostic access permits | forbidden |
| `deterministically_blocked` | `blocked` or no visible output according to owner context | deterministic block code, block scope, owner spec, and validation refs when permitted | forbidden |
| `blocked_owner_todo` | `blocked` | owner TODO row, owner spec, and validation family when permitted | forbidden |
| `inactive_deferred` | no visible output | deferred owner spec and activation rule only when diagnostic access permits | forbidden |

None of the owner states in this table may authorize pass, fail, absence, cleanup, graph expiry, retraction, watermark advancement, or risk reduction. Caller-visible output must include owner context only when authorization and redaction permit it.

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
| run lock lifecycle | `blocked` for conflict, loss, fencing, or idempotency conflict; `degraded` for heartbeat uncertainty when no output was attempted | imported `030` retry class | output blocked; no source, fact, graph, package, table maintenance, or watermark mutation | run_lock_health | raw lock inputs forbidden; redacted lock key refs and checksums only |

### Page token canonicalization

API page tokens must be generated from `040.CanonicalJSON` over query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. Backend cursors or internal IDs are forbidden as page token identity. Forbidden PostgreSQL/AGE token inputs include PostgreSQL OIDs, tuple IDs, sequence values, physical row locators, SQL cursor names, prepared statement names, provider-native query handles, AGE vertex IDs, AGE edge IDs, AGE path IDs, AGE graph IDs, AGE label IDs, and agtype object IDs.

### PageTokenPolicy

| Policy field | Required behavior |
| --- | --- |
| request TTL field | Endpoints that paginate accept `page_token_ttl_seconds`; omitted value materializes to `900` before `request_checksum`. |
| minimum TTL | `60` seconds. |
| maximum TTL | `3600` seconds. |
| token input fields | Endpoint token, query checksum, authorization decision checksum, redaction context checksum, owner profile refs, graph profile refs when applicable, derived-view state when applicable, ordering keys, page boundary, TTL, and expiration. |
| malformed token | Reject with `PAGE_TOKEN_INVALID` before owner execution, backend execution, or result enumeration. |
| expired token | Reject with `PAGE_TOKEN_EXPIRED` or `GRAPH_PAGE_TOKEN_EXPIRED` before owner execution; expired tokens must not be refreshed. |
| authorization mismatch | Reject with `PAGE_TOKEN_INVALID`; response must not reveal whether the original result still exists. |
| request checksum mismatch | Reject with `PAGE_TOKEN_INVALID` before owner execution. |
| derived-view mismatch | Reject with `DERIVED_VIEW_LAG_ERROR` when current state is required; otherwise reject with `PAGE_TOKEN_INVALID` if the token identity no longer matches. |
| graph profile mismatch | Reject with `GRAPH_PAGE_TOKEN_INVALID` or `PAGE_TOKEN_INVALID` before backend query execution. |
| redaction mismatch | Reject with `PAGE_TOKEN_INVALID` before owner execution because page boundaries may differ after redaction. |
| backend cursor identity | Reject with `PAGE_TOKEN_INVALID`; backend cursors, backend-native IDs, PostgreSQL OIDs, tuple IDs, sequence values, physical row locators, SQL cursor names, prepared statement names, provider-native query handles, AGE vertex IDs, AGE edge IDs, AGE path IDs, AGE graph IDs, AGE label IDs, and agtype object IDs are forbidden token identity. |
| portability | Tokens are not portable across callers, roles, authorization decisions, graph profiles, redaction contexts, endpoint outcome states, request checksums, or derived-view states. |
| promotion behavior | `110-PAGE-TOKEN-DEFAULTS-AC-001` and `110-ENDPOINT-OUTCOME-TOTAL-AC-001` pass only when every endpoint token case has `120` fixtures. |

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
| `diagnostic_correlation_ref` | Required when emitted in response envelope | Opaque Cadastre correlation ref; raw trace/span IDs are forbidden unless permitted by `140` and redacted by `110`. |
| `version_manifest_ref` | Required when production output relies on manifest | Manifest ref. |

### Observable API outcome matrix

`EndpointOutcomeMatrix` is total for public endpoint classes in this spec. Endpoint implementations must select the row by endpoint before executing owner behavior and must apply the most-specific error precedence after owner result materialization.

Endpoint outcomes must preserve `authorized_absent` distinctly from `unknown`, `authorized_not_observed`, `not_checked`, `not_applicable`, `permission_limited`, `conflicted`, `ambiguous`, and `error`. A response may render `authorized_absent` only when owner context includes `060.AbsenceDerivationResult.absence_authorized = true`, the selected source-dataset catalog row ref/checksum, and the requested effect/predicate/scope authorization refs.

| Endpoint | Success payload | Empty result | Unauthorized behavior | Raw payload behavior | Source stale behavior | Derived-view stale behavior | Partial known gap | Partial unknown gap | Conflicted behavior | Ambiguous behavior | Page token behavior | Most-specific error precedence | Mutation prohibition |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `AssetSearch` | `AssetSearchResponse.assets[]` plus envelope. | Return `assets = []`, `result_count = 0`, and envelope; do not expose inaccessible counts. | Omit unauthorized matches; deny scope with `AUTHORIZATION_ERROR`. | Raw bytes forbidden in search results. | Label matching rows `source_stale`. | Label only when graph-derived context is included; otherwise not applicable. | Preserve `partial_known_gap`. | Preserve `partial_unknown_gap`. | Preserve `conflicted`; no pass/fail inference. | Preserve `ambiguous`; no result merging. | Validate before enumeration; token not portable across caller or redaction. | Owner-specific validation before `API_BOUNDS_INVALID` when owner context is already selected. | Must not mutate identity, facts, graph, completeness, package, or watermarks. |
| `AssetDetail` | `AssetDetailResponse.asset` plus envelope. | Visible but absent owner detail returns null detail with owner state label; inaccessible returns authorization denial. | `AUTHORIZATION_ERROR` with no existence leak. | Metadata only unless `raw.read` and endpoint policy permit. | Preserve `source_stale`. | Graph context labels `derived_view_stale`; compliance/audit detail rejects if current graph required. | Preserve. | Preserve. | Preserve assertion conflict. | Preserve selector/result ambiguity. | Detail pagination uses token when evidence or relationships are paged. | `AUTHORIZATION_ERROR` before private-binding leak in caller-visible denial. | Read-only. |
| `EvidenceDrillback` | Ordered evidence chain. | Empty chain only when owner confirms no drillback refs; missing required lineage emits `LINEAGE_ERROR`. | Hide inaccessible chain members or deny whole chain when leak risk exists. | Raw bytes require `raw.read`; otherwise emit redacted payload ref. | Preserve owner source stale. | Preserve graph context lag separately. | Preserve. | Preserve. | Preserve. | Preserve. | Validate token before expanding next chain page. | `RAW_PAYLOAD_PERMISSION_REQUIRED` when policy requires hard failure; otherwise redaction marker. | Read-only. |
| `GraphQuery` | Nodes, edges, paths, or graph detail by query class. | Empty traversal classes return no paths; no matches return empty arrays. | Apply authorization/redaction after backend candidate materialization and before response. | Raw evidence expansion requires raw permission. | Display eligible source-stale graph objects only when `090` handoff permits. | Reject or label by `DerivedViewLagPolicy` and query class. | Partial output only when `090` handoff permits. | Partial output only when `090` handoff permits. | Visibility only when `090` permits conflicted graph objects. | Reject or non-mutating empty/diagnostic by `090` handoff. | Graph token identity also includes graph profile, backend taxonomy, output eligibility, derived-view state, and redaction context. | `GRAPH_TRAVERSAL_CLASS_REQUIRED`, token errors, and candidate-limit errors precede generic API errors. | Must not mutate graph, facts, identity, or watermarks. |
| `OperationalHealthStatus` | Component health rows, including observability components when requested. | Return evaluated components with no blocking errors. | Deny unauthorized scope with no private diagnostics. | Raw bytes forbidden. | Render as health diagnostic, not pass/fail. | Render graph health lag separately. | Render as degraded/blocking by owner. | Render as degraded/blocking by owner. | Render as error/degraded by owner. | Render as error/degraded by owner. | Pagination optional for large component sets; token policy applies. | Owner health code before generic health error; telemetry degradation renders as health diagnostic through `140`. | Must not repair, mutate, or use telemetry to repair or mutate domain records. |
| `ComplianceExport` | Canonical export rows plus checksum. | `rows = []` only for authorized empty scope; no inaccessible counts. | Deny before row enumeration. | Raw bytes forbidden unless separate raw export policy exists. | Export as stale/non-pass-fail by default. | Reject stale graph-derived rows by default. | Export non-pass/fail state. | Export non-pass/fail state. | Export `conflicted`, not pass/fail. | Export `ambiguous`, not pass/fail. | Required for paged export; token not portable across manifest or policy refs. | Owner compliance errors before generic export errors. | Must not mutate facts, graph, remediation, or watermarks. |
| `AuditExport` | Audit event rows plus checksum. | `audit_events = []` only for authorized empty audit scope. | Deny before event enumeration. | Raw bytes hidden unless secure audit policy explicitly permits. | Audit state as observed; no pass/fail. | Audit derived-view state refs separately. | Audit owner context. | Audit owner context. | Audit conflict context when authorized. | Audit ambiguity context when authorized. | Required for paged audit export. | Authorization and redaction errors before generic export errors. | Must not mutate audited records. |
| `AnalysisReadOnly` | Analysis result, finding preview, metric, or execution summary. | Empty analysis result is success with envelope. | Deny if caller lacks analysis and underlying graph permissions. | Raw expansion requires raw permission. | Preserve as input state. | Reject or display stale state only when `130` handoff permits. | Preserve non-authoritative partial state. | Preserve non-authoritative partial state. | Preserve conflict without mutation. | Preserve ambiguity without mutation. | Uses API token policy when paged. | `ANALYSIS_MUTATION_FORBIDDEN` and graph compatibility errors precede generic API errors. | Must not mutate raw, silver, identity, gold, graph, package, completeness, watermarks, or manifests. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `110-API-SCHEMA-TOTAL-AC-001` | Every exported request and response object has a field table with required status, default, bounds or omitted-case behavior, redaction behavior, and error behavior. |
| `110-STATE-LABEL-TOTAL-AC-001` | Every declared `SourceStateLabel` appears exactly once in `SourceStateLabelMapping`. |
| `110-STATE-LABEL-AUTHORIZED-ABSENT-AC-001` | `authorized_absent` remains distinct from `authorized_not_observed`, `unknown`, `not_checked`, `not_applicable`, `permission_limited`, `conflicted`, `ambiguous`, and `error` in API, compliance, audit, evidence drillback, and graph-query rendering. |
| `110-SOURCE-DATASET-ERROR-AC-001` | Every source-dataset catalog error emitted by `020` has exactly one generated `ErrorCodeRegistryRow`, owner context schema, fixture ref, and manifest checksum inclusion rule. |
| `110-STATE-LABEL-CONFLICT-AC-001` | `conflicted` never renders as pass, fail, authorized absence, remediation, graph expiry, cleanup, retraction, source watermark, or ambiguity by default. |
| `110-OBSERVABILITY-HEALTH-AC-001` | Telemetry exporter, Collector, queue, profile, and dropped-signal states render through `OperationalHealthStatus` using `140.TelemetryHealthMappingPolicy` and cannot mutate domain records. |
| `110-OBSERVABILITY-REDACTION-AC-001` | Raw trace/span IDs, telemetry attributes, exporter endpoints, Collector routes, private bindings, backend IDs, provider-native query text, and raw payload values are redacted or rejected according to `140` and `110`. |
| `110-DIAGNOSTIC-CORRELATION-AC-001` | `diagnostic_correlation_ref` is opaque, optional, audit-linked when emitted, excluded from domain replay by default, and not accepted as evidence, identity, graph, completeness, package, or audit authority. |
| `110-STATE-LABEL-AMBIGUOUS-AC-001` | `ambiguous` never renders as pass, fail, authorized absence, remediation, graph expiry, cleanup, retraction, source watermark, or conflict by default. |
| `110-ERROR-REGISTRY-TOTAL-AC-001` | `GenerateErrorCodeRegistry` rejects missing, duplicate, TODO, unknown-severity, unknown-retry-class, unredacted, wildcard-fixtured, legacy `code` caller-field, generic-substitute, or unfixtured owner error rows. |
| `110-OWNER-ERROR-FRAGMENT-AC-001` | Every owner error fragment that exports error codes satisfies `OwnerErrorFragmentCompletionRequirement`; any `TODO`, blank required field, duplicate code, or generic substitute fails with `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE`. |
| `110-ANALYSIS-REGISTRY-ERROR-AC-001` | Every expanded `130` analysis, lineage, threat-intel, registry, custom-property, classification, and derived-edge error row renders through `GenerateErrorCodeRegistry` with standard caller-visible fields, audit-visible owner context, redaction rules, fixture refs, and `030.VersionManifest` checksum inclusion. |
| `110-AUTHORIZATION-MATRIX-AC-001` | Every endpoint outcome uses `AuthorizationPermissionMatrix` and emits `AuthorizationDecision` plus audit evidence for allow, deny, and redact-only decisions. |
| `110-REDACTION-MATRIX-AC-001` | Every data class in `RedactionDataClassMatrix` has caller-visible, audit-visible, and forbidden-output fixture coverage. |
| `110-ENDPOINT-OUTCOME-TOTAL-AC-001` | Every endpoint row in `EndpointOutcomeMatrix` has deterministic success, empty, unauthorized, stale, partial, conflicted, ambiguous, pagination, redaction, and error-precedence behavior. |
| `110-COMPLIANCE-NONNEGATIVE-AC-001` | Compliance export rows for `unknown`, `error`, `not_checked`, `not_applicable`, `conflicted`, and `ambiguous` are nonnegative counts and never pass/fail by default. |
| `110-GRAPH-STATE-SEPARATION-AC-001` | `source_stale` and `derived_view_stale` remain separate in graph query, asset detail graph context, compliance export, audit export, and health output. |
| `110-STATE-LABEL-PRESERVATION-AC-001` | `source_stale`, `derived_view_stale`, `unknown`, `not_applicable`, `permission_limited`, `source_unavailable`, `scope_unavailable`, `partial_known_gap`, `partial_unknown_gap`, `conflicted`, and `ambiguous` remain distinct in state summary, evidence drillback, compliance export, and audit export. |
| `110-GRAPH-BACKEND-HEALTH-AC-001` | Operational health exposes provider profile, package gate, schema fingerprint, index freshness, and raw-write bypass states without leaking backend-native IDs, credentials, private config, or raw package evidence. |
| `110-GRAPH-BACKEND-ERROR-AC-001` | New `090` graph backend errors appear in `GenerateErrorCodeRegistry` with no `TODO:` values, owner-specific precedence over generic errors, redacted caller fields, audit refs, and fixture families. |
| `110-CLEANUP-AC-001` | No banned reference class remains. |
| `110-CLEANUP-AC-002` | API and UI state labels remain distinct and cannot imply authorized negative facts unless the owning domain spec emitted the authoritative output. |
| `110-CLEANUP-AC-003` | Raw payload exposure remains false by default and requires raw-evidence permission. |
| `110-CLEANUP-AC-004` | Generic error codes remain forbidden when a more specific owner error code exists. |
| `110-SCHEMA-PATCH-AC-001` | Every 040 error code appears in the generated error registry with owner, severity, retryability, and redaction behavior. |
| `110-SCHEMA-PATCH-AC-002` | Raw payload leakage through `EvidenceRef` or graph properties fails closed and produces a redacted audit event. |
| `110-SCHEMA-PATCH-AC-003` | Backend-native IDs are never returned as Cadastre identity, evidence identity, pagination identity, or drillback identity. |
| `110-ERROR-RECORD-AC-001` | `ErrorRecord` has no duplicate field definitions, has `error_correlation_id` and `owner_error_context`, and separates caller-visible from audit-visible fields. |
| `110-PAGE-TOKEN-AC-001` | Page tokens reject backend cursors, authorization-context mismatches, expired tokens, and stale derived-view state when the query class requires current state. |
| `110-CORE-ONEOF-ERROR-AC-001` | `CORE_ONE_OF_INVALID` renders deterministic caller-visible fields for subject, object, evidence artifact, graph node, and graph edge one-of failures. |
| `110-CORE-ONEOF-ERROR-AC-002` | Audit output includes allowed/observed kind metadata and value-member names without exposing raw payload bytes, private bindings, backend IDs, source-native identity values, or artifact payload bytes. |
| `110-EVIDENCE-ARTIFACT-ERROR-AC-001` | `EvidenceRef.artifact_class`/`artifact_id.kind` mismatch renders as `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH`, not a generic validation error. |
| `110-CORE-SPECIFIC-ERROR-AC-001` | Generic codes are rejected when `CORE_ONE_OF_INVALID`, `CORE_NULL_FORBIDDEN`, `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN`, `GRAPH_BACKEND_ID_FORBIDDEN`, or `PRIVATE_BINDING_LEAK` specifically covers the failure. |
| `110-TEMPORAL-ERROR-REGISTRY-AC-001` | Every `080` temporal, correction, late-arrival, snapshot, no-op, and replay error code appears in `ErrorCodeRegistry` with owner, severity, retryability, redaction, caller-visible behavior, audit-visible fields, and validation fixture. |
| `110-IDENTITY-ERROR-MAPPING-AC-001` | Generated error registry includes every `070` resolver error and rejects generic fallbacks when a specific resolver code applies. |
| `110-TEMPORAL-LABEL-AC-001` | Temporal errors render as `error`, unauthorized negatives do not render as authorized absence, replay errors do not mutate output, duplicate no-ops render as no output change with audit evidence, and stale known states render as `source_stale`. |
| `110-ACTIVATION-ERROR-AC-001` | Activation artifact errors expose owner, artifact class, retryability, and redaction state. |
| `110-ACTIVATION-ERROR-AC-002` | Activation artifact errors never collapse into `unknown`, `not_checked`, pass, or fail. |
| `110-PAGE-TOKEN-DEFAULTS-AC-001` | Page-token default and maximum expiration are explicit or promotion is blocked. |
| `110-FEED-CLOSURE-AC-001` | `not_authoritative_for_absence` renders as `unknown` with owner context and never as an authorized negative fact. |
| `110-FEED-CLOSURE-AC-002` | `partial_known_gap` remains caller-visible as `partial_known_gap` and does not collapse to `unknown` when that owner state is available. |
| `110-SOURCE-CLOSURE-STATE-AC-001` | Every `060` closure outcome maps to exactly one caller-visible label and never to authorized negative unless `060` authorized it. |
| `110-SOURCE-CLOSURE-ERROR-AC-001` | Every new `060` error code appears in `ErrorCodeRegistry`. |
| `110-SOURCE-CLOSURE-COMPLIANCE-AC-001` | `unknown`, `error`, `not_checked`, and `not_applicable` never render as pass or fail by default. |
| `110-SOURCE-CLOSURE-STATE-AC-002` | `deterministically_blocked`, `blocked_validation`, `not_authoritative_for_absence`, source-history outside-window no-proof, and external-schema authority blocks render deterministically and never as authorized negative output. |
| `110-SOURCE-CLOSURE-ERROR-AC-002` | New source-closure error rows have no `TODO` values in severity, retryability, redaction, caller fields, audit fields, or fixture refs. |
| `110-FEED-CLOSURE-AC-003` | Every new `020`, `030`, and `060` feed-closure error code appears in the generated registry with severity, retryability, redaction, owner, and fixture ID. |
| `110-FEED-CLOSURE-AC-004` | Feed activation, category closure, upstream completeness, effect-blocked, and watermark-blocked health rows are diagnostics only and do not mutate source authority, facts, graph state, or watermarks. |
| `110-LIFECYCLE-ERROR-AC-001` | Lifecycle errors appear in `ErrorCodeRegistry` with owner, severity, retryability, redaction, and fixture ID. |
| `110-LIFECYCLE-ERROR-AC-002` | Lifecycle errors never collapse to `unknown`, `not_checked`, pass, or fail. |
| `110-LIFECYCLE-HEALTH-AC-001` | Lifecycle health rows are diagnostic and do not mutate facts, graph state, package state, completeness, or watermarks. |
| `110-OCSF-MAP-ERROR-AC-001` | Every new `050`, `060`, and `090` OCSF mapping, source-extension, external-schema non-authority, and flow-role evidence code appears in `ErrorCodeRegistry` with severity, retryability, redaction, owner, and caller-visible behavior. |
| `110-PACKAGE-ERROR-AC-001` | Every `100` package activation, rollback, quarantine, last-known-good, and emergency error code appears in `ErrorCodeRegistry`. |
| `110-PACKAGE-ERROR-AC-002` | Package activation errors expose audit refs but do not leak private artifact payloads, private source bindings, raw SBOM bytes, unauthorized package evidence, signer secrets, or private repository paths. |
| `110-PACKAGE-ERROR-PARITY-AC-001` | `PackageActivationErrorObservableMapping` has exactly the same `error_code` set as `100.PackageErrorRegistryFragment`, with no missing `100` code and no unowned `110` package row. |
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
| `110-RUNLOCK-STATE-AC-001` | A graph query, health response, API response, export, or audit output affected by `RUN_LOCK_LOST`, `RUN_LOCK_CONFLICT`, `RUN_LOCK_HEARTBEAT_UNCERTAIN`, `RUN_LOCK_FENCING_TOKEN_STALE`, or `RUN_LOCK_IDEMPOTENCY_CONFLICT` renders a blocked, degraded, or error operational state with owner context and no authorized negative interpretation. |
| `110-RUNLOCK-ERROR-REGISTRY-AC-001` | Every new `030` run-lock error appears in the generated error registry with exact fixture refs, redaction behavior, owner context, and no generic substitute. |
| `110-ERROR-REGISTRY-SHARED-AC-001` | `Shared110ErrorRegistryFragment` contains only `110` shared rows, uses closed severity and retry classes, has exact fixture refs, and does not co-own owner-domain rows such as `DERIVED_VIEW_LAG_ERROR`. |
| `110-ERROR-REGISTRY-ALIAS-AC-001` | Rejected alias codes fail before registry generation and cannot appear in API, export, health, audit, validation, telemetry, or package-visible diagnostics. |

### Structured input API and security acceptance criteria

| ID | Criterion |
| --- | --- |
| `110-STRUCTURED-INPUT-AUTH-AC-001` | Structured-input repository operations deny by default, apply no-existence-leak behavior, and require the permission named in `AuthorizationPermissionMatrix`. |
| `110-STRUCTURED-INPUT-REDACTION-AC-001` | Structured-input API, audit, telemetry, package report, and validation output reject or redact raw file bytes, private routes, credentials, raw paths, private schema payloads, and private fixture bytes. |
| `110-STRUCTURED-INPUT-AUDIT-AC-001` | Structured-input repository read, write, review, merge, validation, materialization, promotion, and rollback operations emit audit events with authorization refs, redaction refs, snapshot refs where present, and no raw repository content. |
| `110-STRUCTURED-INPUT-ERROR-AC-001` | Every structured-input error from `010`, `030`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, and `140` is rendered through generated error registry rows with owner context and redaction. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `110-SCOPE-SELECTOR-ERROR-AC-001` | Selector ambiguity renders as `SourceStateLabel.ambiguous`, not conflict or generic blocked, when owner ambiguity context is available. |
| `110-SCOPE-SELECTOR-ERROR-AC-002` | Private selector values are hidden from API, export, audit, health, telemetry, package report, and validation outputs; permitted outputs use selector checksums, redacted refs, and redacted dimension keys only. |
| `110-SCOPE-SELECTOR-ERROR-AC-003` | Generated error registry rejects generic selector errors when the owner-specific selector error exists. |
| `110-AC-001` | Every observable API outcome has deterministic success, empty-result, redaction, authorization, stale-state, partial-state, and error behavior. |
| `110-AC-002` | Raw payload requests without permission return metadata plus redaction marker, not raw bytes. |
| `110-AC-003` | Source stale and derived-view stale states are never collapsed. |
| `110-AC-004` | Error records include owner spec and use the most specific available error code. |
| `110-AC-005` | Compliance and audit query classes reject stale graph-derived state by default. |
| `110-AC-006` | API, health, evidence drillback, compliance export, and audit export preserve distinctions among unknown, not authoritative, partial, permission-limited, stale, and authorized-not-observed states. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
