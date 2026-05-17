---
doc_id: CADASTRE-NLSPEC-010
title: System Boundary and Authority
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define what Cadastre is, what it must not do, what can be authoritative, and which outputs are projections.

## Explicit Non-Scope

- Lakehouse read field schemas.
- Graph backend adapter mechanics.
- Identity resolver algorithms.
- Package supply-chain mechanics.

## Imports

- None.

## Exports

- `AuthorityClass`
- `PublicPrivateBoundaryRule`
- `ProjectionAuthorityRule`
- `DirectSourceProhibition`
- `SourceOfRecordRule`

## Boundary Contract

Cadastre must be a lakehouse-fed interpretation, normalization, identity, fact, and projection system. It must not collect directly from configured enterprise source systems in the default production boundary.

| Input class | Production status | Required handling |
| --- | --- | --- |
| Declared lakehouse table snapshot | Allowed | Must be named by `LakehouseSnapshotRef` imported from `020`. |
| Declared dataset version | Allowed | Must be named by `DatasetVersionRef` imported from `020`. |
| Object-store raw batch | Allowed | Must be named by `RawFeedManifest` imported from `020`. |
| Supplier-provided metadata | Allowed | Must be interpreted only through active authority, completeness, temporal, and mapping policies. |
| Enterprise source API call | Forbidden | Must fail with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| Source scanner poll, source syslog receive, source CDC connector operation | Forbidden | Must be performed by an external supplier, not by Cadastre production. |
| Validation-only source exploration | Allowed only when explicitly declared | Must emit only `SourceExplorationResult` or `ProbeDiagnosticRecord`; it must not satisfy production evidence. |

## Authority Classes

| Authority class | Meaning | Default owner |
| --- | --- | --- |
| `system_of_record` | Record class that can determine product truth. | Authoritative lakehouse records only. |
| `derived_projection` | Rebuildable output from authoritative records. | Graph, CIM, OCSF export, consumer outputs. |
| `supporting_evidence` | Source or supplier data that can support a decision only through an active profile. | Raw records, upstream completeness evidence, lineage, diagnostics. |
| `non_authoritative_analysis` | Findings, metrics, enrichment, and registry metadata with no mutation authority. | Analysis and governance records. |
| `inactive_future_domain` | Preserved candidate material with no MVP effect. | Future reachability. |

## Projection Authority Rule

Graph state, Splunk CIM output, OCSF export output, metadata graphs, lineage graphs, analysis outputs, and registry labels must not create or modify authoritative facts. A projection may be served only when its derivation profile, input refs, output checksum, and lag state are persisted.

## Public and Private Source Binding

The public NLSpec set must define vendor-neutral feed contracts. Concrete vendor names, private source binding artifacts, routes, credentials, and environment-specific bindings must live in private implementation artifacts and must not appear in public canonical records.

Core schemas may name vendor-neutral source categories and redacted refs only. A public `RawRecord`, `EvidenceRef`, graph delta, API response, validation report, or export artifact that exposes a private source binding must fail with `PRIVATE_BINDING_LEAK` before publication or persistence.

| Artifact | Public canonical model status |
| --- | --- |
| `RawSupplierProfile` | Public, vendor-neutral. |
| `LakehouseFeedProfile` | Public, vendor-neutral. |
| `PrivateSourceFeedBinding` | Private only. |
| `PrivateFeedSchemaInventory` | Private only. |
| `PrivateCompletenessEvidenceInventory` | Private only. |
| `PrivateGoldenCorpus` | Private only unless redacted into `LakehouseFeedFixture`. |

## Cross-Domain Invariants

- Missing lakehouse rows must not imply source absence.
- Successful feed read must not imply source completeness.
- Source completeness must not imply source authority.
- Source authority must not imply identity merge authority.
- Identity resolution must not mutate graph state directly.
- Graph apply success must not imply fact correctness.
- Package signature verification must not imply package activation eligibility.
- Analysis findings must not mutate facts, graph state, watermarks, or completeness.

## Required Errors

| Error code | Emitted when |
| --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | Production execution attempts to call or authenticate to an enterprise source system. |
| `PROJECTION_AUTHORITY_VIOLATION` | A derived projection attempts to create authoritative records. |
| `PRIVATE_BINDING_LEAK` | A public artifact contains a concrete private vendor/source binding. |
| `UNDECLARED_AUTHORITY_CLASS` | A record is written without an owner declaring its authority class. |

## Authority Validation Contracts

### PrivateBindingLeakValidationRule

Public artifacts must be scanned before publication, API response emission, export, and validation-report materialization.

| Rejected artifact class | Rejected field or value pattern | Error precedence | Required behavior |
| --- | --- | --- | --- |
| Public NLSpec or README | Concrete enterprise source name, private route, credential material, tenant-specific binding, private inventory entry | `PRIVATE_BINDING_LEAK` before generic validation errors | Reject the artifact and record the owner spec. |
| Public API response | Inaccessible asset identifier, private source route, raw credential, unredacted raw payload | `AUTHORIZATION_ERROR` before `PRIVATE_BINDING_LEAK` when caller access is the root cause | Return redacted or fail closed as `110` defines. |
| Export artifact | Private source binding, environment-specific host list, private golden corpus sample | `PRIVATE_BINDING_LEAK` | Reject export before write. |
| Validation report | Private fixture bytes or unreduced corpus contents | `PRIVATE_BINDING_LEAK` | Store only redacted fixture refs and checksums. |

### SourceOfRecordRule

| Output class | Authority class | Runtime owner | Default if owner row missing |
| --- | --- | --- | --- |
| `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `IdentityDecision`, `GoldFact` | `system_of_record` | `040`, `070`, `080` as applicable | Fail with `UNDECLARED_AUTHORITY_CLASS`. |
| `EvidenceRef`, supplier metadata, lineage, diagnostics, upstream completeness evidence | `supporting_evidence` | `020`, `040`, `060`, `110`, `130` as applicable | Diagnostic or pointer only unless owner profile consumes it. |
| `GraphNodeDeltaShape`, `GraphEdgeDeltaShape`, `GraphDeltaSet`, `GraphApplyResult`, graph read model, CIM output, OCSF export | `derived_projection` | `040`, `050`, `090`, `110` | Fail or no-op as owner specifies. |
| `AnalysisFinding`, `AnalysisMetric`, enrichment, registry metadata | `non_authoritative_analysis` | `130` | No mutation authority. |
| Reachability candidate contracts | `inactive_future_domain` | `200` | No MVP effect. |

### ValidationOnlySourceExplorationBoundary

`source_exploration` and validation-only probe modes may emit only `SourceExplorationResult`, `ProbeDiagnosticRecord`, validation diagnostics, or redacted fixture candidates. They must not satisfy production raw, completeness, silver, identity, gold, graph, health, watermark, manifest, or acceptance-report contracts.

### Error ownership export

| Error code | Owning spec | Shared registry owner |
| --- | --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | `010` | `110` |
| `PROJECTION_AUTHORITY_VIOLATION` | `010` | `110` |
| `PRIVATE_BINDING_LEAK` | `010` | `110` |
| `UNDECLARED_AUTHORITY_CLASS` | `010` | `110` |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `010-CLEANUP-AC-001` | No banned reference class remains. |
| `010-CLEANUP-AC-002` | Direct source calls still fail before output with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-CLEANUP-AC-003` | Private binding leakage validation still rejects public artifacts that expose private source binding artifacts. |
| `010-CLEANUP-AC-004` | Every output class still maps to exactly one authority class. |
| `010-SCHEMA-PATCH-AC-001` | Every 040 exported record maps to exactly one authority class. |
| `010-SCHEMA-PATCH-AC-002` | `EvidenceRef` remains supporting evidence and cannot become raw payload storage or fact authority by itself. |
| `010-SCHEMA-PATCH-AC-003` | Graph delta primitive shapes remain derived projection records. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `010-AC-001` | A production run attempting an enterprise source API call fails before any output write with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-AC-002` | Every output class is classified as `system_of_record`, `derived_projection`, `supporting_evidence`, `non_authoritative_analysis`, or `inactive_future_domain`. |
| `010-AC-003` | No active spec permits graph, CIM, OCSF export, analysis, enrichment, lineage, registry, or package tooling output to become authoritative without an imported named authority contract. |
| `010-AC-004` | Public artifacts fail validation when they contain private source binding artifacts, route names, credential fields, or environment-specific source target lists. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
