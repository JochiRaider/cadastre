---
doc_id: CADASTRE-NLSPEC-060
title: Source Authority, Completeness, Coverage, and Absence
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

Define when Cadastre may treat observations, missing rows, stale states, control results, and coverage states as authoritative.

## Explicit Non-Scope

- Identity merge logic.
- Parser output shape.
- Lakehouse read mechanics.
- Graph apply mechanics.

## Imports

- `AuthorityClass`
- `LakehouseReadCompletenessReceipt`
- `UpstreamCompletenessEvidence`
- `RawRecord`
- `CadastreSilverObservation`
- `StageStateRecord`

## Exports

- `SourceAuthorityProfile`
- `SourceAuthorityProfileRow`
- `SourceStalenessPolicy`
- `SupplierCollectionVisibilityProfile`
- `ControlResultMappingRow`
- `SourceHistoryRetentionProfile`
- `DeriveAbsenceOrUnknown`
- `AbsenceDerivationPolicy`
- `AbsenceDerivationResult`
- `CoverageDimensionProfile`
- `CoverageAssertion`
- `LakehouseFeedCompletenessProfile`
- `ProgressSignalInterpretationPolicy`
- `ProjectionWatermarkPolicy`
- `WatermarkCommitRecord`
- `EvaluateLakehouseFeedCompleteness`

## Source Authority Contract

Gold derivation must use an active `SourceAuthorityProfileRow` for the exact fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, staleness requirement, coverage requirement, and requested absence authority.

A broad source-category ranking must not act as fallback authority when the exact row is missing.

## Completeness Contract

`LakehouseFeedCompletenessProfile` maps feed-read completeness and supplier-provided upstream evidence into source completeness decisions. A `LakehouseReadCompletenessReceipt` alone must not authorize absence, retraction, cleanup, graph expiry, or watermark advancement.

| Completeness decision | Absence authority default | Cleanup/retraction default |
| --- | --- | --- |
| `complete` | May be considered only when profile row permits. | May be considered only when profile row permits. |
| `empty_complete` | May be considered only when profile row permits. | May be considered only when profile row permits. |
| `partial_known_gap` | Forbidden. | Forbidden. |
| `partial_unknown_gap` | Forbidden. | Forbidden. |
| `source_unavailable` | Forbidden. | Forbidden. |
| `scope_unavailable` | Forbidden. | Forbidden. |
| `permission_limited` | Forbidden. | Forbidden. |
| `not_attempted` | Forbidden. | Forbidden. |
| `not_authoritative_for_absence` | Forbidden. | Forbidden. |
| `not_applicable` | No absence claim; may emit not-applicable state when profile permits. | Forbidden. |

## Progress Signal Interpretation

Progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, and live-probe signals are non-authoritative by default. They may affect output only through an active `ProgressSignalInterpretationPolicy` row.

| Signal class | Default effect |
| --- | --- |
| Cursor exhaustion | Candidate evidence only; no absence by itself. |
| CDC offset or heartbeat | Replay/progress state only; no fact time, absence, or cleanup. |
| Queue drain or ack success | Delivery diagnostic only. |
| Provenance closure | Lineage diagnostic only. |
| dbt or freshness artifact | Freshness diagnostic only. |
| Graph apply success | Derived-view state only. |
| Destination stale delete | Non-authoritative; no source absence. |
| Live source probe result | Validation/exploration only. |

## Coverage Contract

Coverage-sensitive facts require `CoverageDimensionProfile` and a current `CoverageAssertion` before absence, pass/fail/unknown, no-change, or negative claims may be emitted.

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. `TODO: complete source-category-specific coverage dimension catalog.`

## DeriveAbsenceOrUnknown Algorithm

```text
DeriveAbsenceOrUnknown(candidate, evidence_set, receipt_set, upstream_evidence_set, completeness_profile, authority_profile, coverage_assertions, staleness_policy):
1. Reject when no active SourceAuthorityProfileRow covers the fact type, predicate, source dataset, scopes, and requested absence authority.
2. Reject when required LakehouseReadCompletenessReceipt is missing or not tied to the active feed profile and manifest refs.
3. Evaluate UpstreamCompletenessEvidence through LakehouseFeedCompletenessProfile.
4. If any blocking unsafe completeness decision is present, return unknown with the most specific unsafe state.
5. Evaluate SourceStalenessPolicy using declared time-input precedence.
6. Require CoverageAssertion when the fact type or predicate is coverage-sensitive.
7. Reject weak progress signals unless an active ProgressSignalInterpretationPolicy grants the exact effect.
8. Emit exactly one AbsenceDerivationResult: absence authorized, unknown, not applicable, stale, no-op, or deterministic error.
```

## Watermarks

Every attempted production source, projection, graph-apply, or presence-only watermark change must be evaluated through `ProjectionWatermarkPolicy` and persisted as exactly one `WatermarkCommitRecord`. Failed, partial, aborted, schema-preflight-failed, or completeness-blocked runs must not advance watermarks.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `060-AC-001` | Missing, stale, partial, permission-limited, unsupported, or unavailable evidence cannot become absence unless an active profile authorizes it. |
| `060-AC-002` | Weak progress signals cannot be combined into stronger authority than each active policy row permits. |
| `060-AC-003` | Every coverage-dependent fact has a current coverage assertion or emits a deterministic unknown/error outcome. |
| `060-AC-004` | Source staleness and derived-view lag are exposed as distinct states. |
| `060-AC-005` | Every watermark attempt emits exactly one `WatermarkCommitRecord`. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `SourceAuthorityProfile` | lines 4307-4671 |
| PRD-Cadastre.md | `SourceStalenessPolicy` | lines 4413-4477 |
| PRD-Cadastre.md | `SupplierCollectionVisibilityProfile` | lines 4478-4520 |
| PRD-Cadastre.md | `ControlResultMappingRow` | lines 4521-4563 |
| PRD-Cadastre.md | `SourceHistoryRetentionProfile` | lines 4564-4595 |
| PRD-Cadastre.md | `DeriveAbsenceOrUnknown` | lines 4596-4671 |
| PRD-Cadastre.md | `LakehouseFeedCompletenessProfile` | lines 6330-6522 |
| PRD-Cadastre.md | `ProgressSignalInterpretationPolicy` | lines 6406-6472 |
| PRD-Cadastre.md | `ProjectionWatermarkPolicy` | lines 6473-6498 |
| PRD-Cadastre.md | `WatermarkCommitRecord` | lines 6499-6522 |
| PRD-Cadastre.md | `CoverageDimensionProfile and CoverageAssertion` | lines 7492-7636 |
| PRD-Cadastre.md | `Source Authority Profiles` | lines 10678-10707 |
| PRD-Cadastre.md | `Source Completeness Acceptance` | lines 13149-13160 |
| PRD-Cadastre.md | `Source Authority Acceptance` | lines 13161-13171 |
| PRD-Cadastre.md | `Source Authority, Staleness, Coverage, Progress, and Absence Acceptance` | lines 13610-13634 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
