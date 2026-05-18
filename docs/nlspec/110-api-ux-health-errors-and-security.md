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
| `include_raw_payload` | false | Raw payload returned only with raw-evidence permission. |

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

## Error Model

`ErrorRecord` must contain stable error code, message, severity, retryability, owner spec, affected record type, field path when applicable, record ID when available, source artifact refs, error correlation ID, owner error context, redaction state, and caller-visible field set. Domain-specific codes must be owned by the domain spec; this spec owns the shared shape and generated registry. Every 040 error must include owner spec, affected record type, field path when applicable, record ID when available, retryability, severity, redaction state, and source artifact refs.

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

| Owner spec | Error prefix or namespace | Shared shape fields | Owner-specific codes | Retryability owner | Redaction owner | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- |
| `010` | boundary/authority | `ErrorRecord` | `DIRECT_SOURCE_CALL_FORBIDDEN`, `PROJECTION_AUTHORITY_VIOLATION`, `PRIVATE_BINDING_LEAK`, `UNDECLARED_AUTHORITY_CLASS` | owner row | `110` | boundary negative rows |
| `020` | feed/table | `ErrorRecord` | feed read, manifest, raw identity, availability, maintenance errors | owner row | `110` | feed fixture rows |
| `030` | dag/lifecycle | `ErrorRecord` | `FORBIDDEN_STAGE_OUTPUT`, lifecycle, run-lock, manifest errors | owner row | `110` | DAG negative rows |
| `040` | canonical data | `ErrorRecord` | `CORE_UNKNOWN_FIELD`, `CORE_REQUIRED_FIELD_MISSING`, `CORE_NULL_FORBIDDEN`, `CORE_FIELD_TYPE_INVALID`, `CORE_FIELD_BOUNDS_INVALID`, `CORE_RECORD_ID_MISMATCH`, `CORE_RECORD_CHECKSUM_MISMATCH`, `CORE_SCHEMA_VERSION_UNSUPPORTED`, record collision errors, `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN`, `GRAPH_BACKEND_ID_FORBIDDEN` | owner row | `110` | core schema rows |
| `050` | mapping/schema | `ErrorRecord` | OCSF, enum, extension, CIM errors | owner row | `110` | mapping rows |
| `060` | authority/completeness | `ErrorRecord` | authority, coverage, progress, watermark errors | owner row | `110` | absence rows |
| `070` | identity | `ErrorRecord` | resolver, review, selector errors | owner row | `110` | identity rows |
| `080` | temporal/replay | `ErrorRecord` | temporal, correction, replay errors | owner row | `110` | event sequence rows |
| `090` | graph | `ErrorRecord` | backend, query, rebuild, drift errors | owner row | `110` | graph rows |
| `100` | package | `ErrorRecord` | activation, trust, rollback, quarantine errors | owner row | `110` | package rows |
| `120` | validation | `ErrorRecord` | validation and acceptance errors | owner row | `110` | validation rows |
| `130` | analysis/registry | `ErrorRecord` | mutation, lineage, registry errors | owner row | `110` | analysis rows |
| `200` | reachability deferred | `ErrorRecord` | reachability prohibited output errors | owner row | `110` | reachability prohibition rows |

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

### Page token canonicalization

API page tokens must be generated from `040.CanonicalJSON` over query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. Backend cursors or internal IDs are forbidden as page token identity.

### PageTokenPolicy

| Policy field | Required behavior |
| --- | --- |
| token input fields | Query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. |
| default expiration | TODO: owner decision required before authoritative promotion. |
| maximum expiration | TODO: owner decision required before authoritative promotion. |
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

## Definition of Done

| ID | Criterion |
| --- | --- |
| `110-AC-001` | Every observable API outcome has deterministic success, empty-result, redaction, authorization, stale-state, partial-state, and error behavior. |
| `110-AC-002` | Raw payload requests without permission return metadata plus redaction marker, not raw bytes. |
| `110-AC-003` | Source stale and derived-view stale states are never collapsed. |
| `110-AC-004` | Error records include owner spec and use the most specific available error code. |
| `110-AC-005` | Compliance and audit query classes reject stale graph-derived state by default. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
