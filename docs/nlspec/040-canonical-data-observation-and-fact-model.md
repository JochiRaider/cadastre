---
doc_id: CADASTRE-NLSPEC-040
title: Canonical Data, Observation, and Fact Model
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

## Purpose

Own Cadastre core record shapes, scalar rules, identifiers, omission states, evidence references, and canonical serialization.

## Explicit Non-Scope

- Source absence authorization algorithm.
- Resolver algorithm.
- OCSF mapping rows.
- Graph projection algorithms.
- Package lifecycle.

## Imports

- `AuthorityClass`
- `VersionManifest`

## Exports

- `ScalarType`
- `CanonicalJSON`
- `OmissionState`
- `FactAbsenceOutcome`
- `RawRecord`
- `CadastreSilverObservation`
- `SourceExtensionFieldRuleShape`
- `CanonicalEntity`
- `SourceAsset`
- `Identifier`
- `GoldFact`
- `EvidenceRef`
- `GraphNodeDeltaShape`
- `GraphEdgeDeltaShape`

## Canonical Serialization

All records owned by this spec must serialize through canonical JSON when their bytes affect IDs, checksums, replay equivalence, package activation, validation output, graph delta identity, or audit evidence.

Canonical JSON rules:

| Rule | Required behavior |
| --- | --- |
| Object key order | Lexical Unicode code point order. |
| Omitted field | Excluded from serialized bytes unless field default is explicitly materialized. |
| Null value | Distinct from omission and serialized as JSON `null` only when the schema allows null. |
| Timestamp | RFC3339 UTC with `Z`; no offset-preserving variants. |
| Decimal | Base-10 string when precision affects output; binary floating point is forbidden for checksummed values. |
| Arrays | Input order is preserved unless the field declares canonical sort keys. |
| Unknown field | Rejected unless the record declares an extension field map. |

## Omission Semantics

Omission states must be explicit. Optionality, nullable schema declarations, absent source field, OCSF field absence, CIM field absence, or source row absence must not decide Cadastre omission semantics by themselves.

| Omission state | Meaning |
| --- | --- |
| `observed_present` | Source evidence asserted a value. |
| `observed_empty` | Source evidence asserted an empty value. |
| `not_provided` | Source did not provide a value and no stronger interpretation is authorized. |
| `not_applicable` | Active profile states the field does not apply. |
| `permission_limited` | Source or supplier visibility did not permit observation. |
| `source_unavailable` | Source or required feed evidence was unavailable. |
| `unsupported` | Source or parser cannot represent the field. |
| `malformed` | Source value was present but invalid for the target field. |
| `redacted` | Value exists but must not be exposed to caller or downstream projection. |

## Core Records

| Record | Required identity | Required fields or field groups |
| --- | --- | --- |
| `RawRecord` | Deterministic raw record ID from feed scope, source dataset, supplier/object identity, source-native ID when present, payload hash, and import profile version. | Source identity, source dataset/type, native ID, event metadata, payload format, payload hash, collector/import versions, run lineage, duplicate/quarantine status, partition key. |
| `CadastreSilverObservation` | Observation ID from raw evidence refs, observation type, mapping bundle, source scope, and normalized payload checksum. | Raw refs, observation type, observed time, collected/imported time, normalized fields, source extension fields, field quality, lineage, confidence hints, flow role evidence when applicable. |
| `CanonicalEntity` | Product-owned canonical entity ID. | Entity type, lifecycle state, valid/known bounds, current status, source decision refs. |
| `SourceAsset` | Source-scoped asset ID. | Source/feed scope, source-native identity fields, source asset type, evidence refs, lifecycle evidence. |
| `Identifier` | Identifier ID from identifier type, normalized value, source scope, and validity interval. | Type, normalized value, scope, raw value refs, validity/known time, quality. |
| `GoldFact` | Fact ID from canonical subject, predicate, object/value, valid interval, assertion state, authority profile, and evidence set. | Subject, predicate, object/value, valid time, known time, assertion state, confidence, authority ref, evidence refs, correction refs. |
| `EvidenceRef` | Evidence ref ID from referenced artifact class, artifact ID, role, and checksum. | Artifact class, artifact ID, evidence role, raw/silver/gold link, checksum, visibility state. |

## Assertion States

`GoldFact.assertion_state` must be a closed enum. Default MVP states are:

| State | Meaning |
| --- | --- |
| `active` | Fact is asserted current for the query interval. |
| `stale` | Fact is retained but stale under an active staleness policy. |
| `retracted` | Fact has been explicitly retracted by authorized evidence. |
| `superseded` | Fact was replaced by correction or stronger evidence. |
| `conflicted` | Concurrent evidence prevents a single resolved assertion. |
| `unknown` | Evidence exists but does not authorize presence or absence. |
| `no_op` | Candidate derivation intentionally emitted no fact. |

## Graph Delta Primitive Shapes

This spec owns only the primitive record shape of `GraphNodeDelta` and `GraphEdgeDelta`. Projection rules, graph apply, backend mapping, traversal, and query behavior are owned by `090`.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `040-AC-001` | Two implementations serialize, hash, compare, and validate every core record shape identically. |
| `040-AC-002` | Omitted, null, empty, redacted, permission-limited, unknown, and unsupported values are distinguishable in serialized output. |
| `040-AC-003` | Graph delta primitive records cannot be created from backend internal IDs. |
| `040-AC-004` | Every `GoldFact` carries valid-time, known-time, assertion-state, evidence, confidence, and authority references. |
| `040-AC-005` | Unknown fields are rejected unless declared through a namespaced extension field rule owned by `050`. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `Scalar Types` | lines 1582-1599 |
| PRD-Cadastre.md | `Omission Semantics` | lines 1621-1690 |
| PRD-Cadastre.md | `RawRecord` | lines 1691-1805 |
| PRD-Cadastre.md | `CadastreSilverObservation` | lines 1806-1886 |
| PRD-Cadastre.md | `CanonicalEntity` | lines 2124-2155 |
| PRD-Cadastre.md | `SourceAsset` | lines 2156-2170 |
| PRD-Cadastre.md | `Identifier` | lines 2171-2213 |
| PRD-Cadastre.md | `GoldFact` | lines 2719-2800 |
| PRD-Cadastre.md | `EvidenceRef` | lines 2914-2927 |
| PRD-Cadastre.md | `GraphNodeDelta` | lines 2928-2947 |
| PRD-Cadastre.md | `GraphEdgeDelta` | lines 2948-2977 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
