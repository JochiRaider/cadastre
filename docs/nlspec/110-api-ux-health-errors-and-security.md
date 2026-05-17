---
doc_id: CADASTRE-NLSPEC-110
title: API, UX, Health, Errors, and Security
doc_type: candidate-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: docs/archive/PRD-Cadastre.revised-draft.md
source_prd_sha256: 99437d5ec12d52752a0003577ac37f8a6c6f1221ac3ae3b7cce713b003aeae55
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

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

`ErrorRecord` must contain stable error code, message, severity, retryability, owner spec, source artifact refs, affected record IDs, correlation ID, and redaction state. Domain-specific codes must be owned by the domain spec; this spec owns the shared shape and generated registry.

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

- Raw payload exposure requires explicit raw-evidence permission.
- Graph properties derived from raw payloads must pass `GraphPropertyEvidencePolicy` and redaction policy.
- API responses must not leak existence of an inaccessible asset through partial detail responses.
- Public artifacts must not expose private source bindings, route names, credentials, or environment-specific inventories.
- Audit events must record caller, authorization context checksum, operation, input checksum, output object classes, redaction summary, and error code when emitted.

## Migration finalization contracts

### ErrorCodeOwnershipMatrix

| Owner spec | Error prefix or namespace | Shared shape fields | Owner-specific codes | Retryability owner | Redaction owner | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- |
| `010` | boundary/authority | `ErrorRecord` | `DIRECT_SOURCE_CALL_FORBIDDEN`, `PROJECTION_AUTHORITY_VIOLATION`, `PRIVATE_BINDING_LEAK`, `UNDECLARED_AUTHORITY_CLASS` | owner row | `110` | boundary negative rows |
| `020` | feed/table | `ErrorRecord` | feed read, manifest, raw identity, availability, maintenance errors | owner row | `110` | feed fixture rows |
| `030` | dag/lifecycle | `ErrorRecord` | `FORBIDDEN_STAGE_OUTPUT`, lifecycle, run-lock, manifest errors | owner row | `110` | DAG negative rows |
| `040` | canonical data | `ErrorRecord` | unknown field, checksum, ID collision errors | owner row | `110` | canonical rows |
| `050` | mapping/schema | `ErrorRecord` | OCSF, enum, extension, CIM errors | owner row | `110` | mapping rows |
| `060` | authority/completeness | `ErrorRecord` | authority, coverage, progress, watermark errors | owner row | `110` | absence rows |
| `070` | identity | `ErrorRecord` | resolver, review, selector errors | owner row | `110` | identity rows |
| `080` | temporal/replay | `ErrorRecord` | temporal, correction, replay errors | owner row | `110` | event sequence rows |
| `090` | graph | `ErrorRecord` | backend, query, rebuild, drift errors | owner row | `110` | graph rows |
| `100` | package | `ErrorRecord` | activation, trust, rollback, quarantine errors | owner row | `110` | package rows |
| `120` | validation | `ErrorRecord` | validation and acceptance errors | owner row | `110` | validation rows |
| `130` | analysis/registry | `ErrorRecord` | mutation, lineage, registry errors | owner row | `110` | analysis rows |
| `200` | reachability deferred | `ErrorRecord` | reachability prohibited output errors | owner row | `110` | reachability prohibition rows |

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

### Page token canonicalization

API page tokens must be generated from `040.CanonicalJSON` over query checksum, authorization context checksum, owner profile refs, ordering keys, page boundary, derived-view state when applicable, and expiration. Backend cursors or internal IDs are forbidden as page token identity.

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
| asset detail | inaccessible entity | none | `AUTHORIZATION_ERROR` | no existence leak | fail closed | n/a | n/a | n/a | `110` |
| evidence drillback | raw payload not permitted | metadata plus redaction marker | none | raw redacted | raw permission required | owner label | owner label | canonical token | `110` |
| graph query | derived view stale and compliance/audit class | reject | `DERIVED_VIEW_LAG_ERROR` | applied | required | reject by default | owner label | canonical token | `090`, `110` |
| export | private binding detected | none | `PRIVATE_BINDING_LEAK` | fail closed | export permission required | owner label | owner label | n/a | `010`, `110` |

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `110-PATCH-AC-001` | Every observable API outcome has deterministic success, empty, redaction, authorization, stale, partial, and error behavior. |
| `110-PATCH-AC-002` | Error records use the most specific owner code and include owner spec. |
| `110-PATCH-AC-003` | Public artifacts and API responses fail or redact private source bindings. |
| `110-PATCH-AC-004` | Compliance and audit query classes reject stale graph-derived state by default. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `110-AC-001` | Every observable API outcome has deterministic success, empty-result, redaction, authorization, stale-state, partial-state, and error behavior. |
| `110-AC-002` | Raw payload requests without permission return metadata plus redaction marker, not raw bytes. |
| `110-AC-003` | Source stale and derived-view stale states are never collapsed. |
| `110-AC-004` | Error records include owner spec and use the most specific available error code. |
| `110-AC-005` | Compliance and audit query classes reject stale graph-derived state by default. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| docs/archive/PRD-Cadastre.revised-draft.md | `User-Facing Behavior` | lines 1015-1533 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Operational Health` | lines 1313-1501 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Source State Labels and Export State Separation` | lines 1502-1533 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Error Model` | lines 11873-12315 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Edge Cases and Required Behavior` | lines 12316-12485 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Bounds and Limit Conditions` | lines 12796-12825 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Security Requirements` | lines 12826-12853 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Configuration and Bounds Acceptance` | lines 13053-13067 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Error and Edge-Case Acceptance` | lines 13068-13087 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Security Acceptance` | lines 13105-13121 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
