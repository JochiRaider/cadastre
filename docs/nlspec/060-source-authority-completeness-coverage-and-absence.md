---
doc_id: CADASTRE-NLSPEC-060
title: Source Authority, Completeness, Coverage, and Absence
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

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. The coverage catalog in this document supplies default dimensions; source-specific unresolved rows remain blocking until closed in the gap ledger and validation matrix.

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

## Migration finalization contracts

### CoverageDimensionProfile catalog

| Domain | Dimension | Required evidence | Freshness policy | Permission-limited behavior | Partial behavior | Stale behavior | Default output |
| --- | --- | --- | --- | --- | --- | --- | --- |
| vulnerability | scanner target, credentialed status, plugin/check scope, scan window | coverage assertion plus scanner evidence | `SourceStalenessPolicy` | `permission_limited` blocks absence | `partial_known_gap` or `partial_unknown_gap` | stale blocks absence | unknown |
| control | benchmark/check scope, applicability, evaluation status | `ControlResultMappingRow` and coverage assertion | owner row | not checked/unknown | unknown | stale | unknown |
| endpoint | asset scope, enrollment visibility, collection window | feed completeness plus authority row | owner row | blocks absence | unknown | stale | unknown |
| directory | tenant/domain, group/member scope, hidden membership visibility | directory completeness evidence | owner row | blocks nonmembership | unknown | stale | unknown |
| DNS | zone/source scope, TTL, authoritative source | DNS feed evidence | TTL-aware policy | unknown | unknown | stale | unknown |
| DHCP/IPAM | scope, lease window, authoritative system | lease/IPAM evidence | lease-aware policy | unknown | unknown | stale | unknown |
| flow | sensor scope, collection point, time window, role evidence | flow feed evidence | window policy | unknown | partial | stale | unknown |
| cloud inventory | account/project/subscription, region, resource type | inventory feed and source history | source-specific | permission_limited | partial | stale | unknown |
| source history | source-native history window, retention | history retention profile | history-window policy | unknown | unknown | outside window no proof | unknown |
| deferred reachability | topology, route, policy, NAT, workload, identity context | inactive `200` placeholders | inactive | no MVP claim | no MVP claim | no MVP claim | no-op |

### LakehouseFeedCompletenessProfile table

| Feed-read state | Upstream evidence state | Completeness decision | Source absence effect |
| --- | --- | --- | --- |
| `read_complete` | sufficient and current | `complete` or `empty_complete` as profile defines | May be considered only with authority and coverage. |
| `read_complete` | missing or insufficient | `not_authoritative_for_absence` | Forbidden. |
| `read_partial_known_gap` | any | `partial_known_gap` | Forbidden. |
| `read_partial_unknown_gap` | any | `partial_unknown_gap` | Forbidden. |
| `read_unavailable` | any | `source_unavailable` | Forbidden. |
| `schema_unavailable` | any | `source_unavailable` | Forbidden. |
| `manifest_invalid` | any | `partial_unknown_gap` or deterministic error | Forbidden. |

### ProgressSignalInterpretationPolicy total matrix

| Signal | Default interpretation | May authorize absence by default |
| --- | --- | ---: |
| cursor exhaustion | progress candidate only | No |
| delta token | resume state only | No |
| CDC offset | replay state only | No |
| CDC heartbeat | liveness diagnostic only | No |
| schema-history availability | replay sufficiency input only | No |
| queue drain | delivery diagnostic only | No |
| ack success | delivery diagnostic only | No |
| provenance closure | lineage diagnostic only | No |
| freshness artifact | freshness diagnostic only | No |
| graph apply | derived-view state only | No |
| destination cleanup | non-authoritative cleanup signal | No |
| live source probe | validation/exploration only | No |
| table commit success | lakehouse write evidence only | No |
| graph rebuild success | derived-view rebuild evidence only | No |

Weak progress signals must not combine into stronger authority.

### SourceStalenessPolicy

| Policy field | Required behavior |
| --- | --- |
| Time-input precedence | Declared per source category, dataset, fact type, predicate, and scope. |
| Missing-time behavior | Emit stale/unknown/error as policy states; no implicit current time. |
| Expiry behavior | May mark facts stale; must not authorize absence unless authority and completeness permit. |
| Output effects | API labels import `110`; graph lag remains separate from source staleness. |

### ControlResultMappingRow

| External state | Cadastre state | Default compliance export |
| --- | --- | --- |
| pass | pass fact only when authority and coverage permit | pass |
| fail | fail fact only when authority and coverage permit | fail |
| unknown | unknown | unknown |
| error | error | error |
| not evaluated | not_checked | not checked |
| not checked | not_checked | not checked |
| not applicable | not_applicable | not applicable |

### SourceHistoryRetentionProfile

| History condition | Required behavior |
| --- | --- |
| Inside supported source-native history window | May be evidence only when source authority row permits. |
| Outside supported source-native history window | Must not become no-change proof. |
| Source history unavailable | Emit unknown or error; no absence. |
| Cadastre replay retention available | Separate from source-native history. |

### Completeness-to-effect matrix

| Completeness decision | Absence authority | Cleanup authority | Retraction authority | Graph expiry authority | Watermark authority | Default API state | Owner |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `complete` | profile-gated | profile-gated | profile-gated | profile-gated | profile-gated | observed or authorized_not_observed | `060` |
| `empty_complete` | profile-gated | profile-gated | profile-gated | profile-gated | profile-gated | authorized_not_observed | `060` |
| `partial_known_gap` | forbidden | forbidden | forbidden | forbidden | forbidden | partial_known_gap | `060` |
| `partial_unknown_gap` | forbidden | forbidden | forbidden | forbidden | forbidden | partial_unknown_gap | `060` |
| `source_unavailable` | forbidden | forbidden | forbidden | forbidden | forbidden | source_unavailable | `060` |
| `scope_unavailable` | forbidden | forbidden | forbidden | forbidden | forbidden | scope_unavailable | `060` |
| `permission_limited` | forbidden | forbidden | forbidden | forbidden | forbidden | permission_limited | `060` |
| `not_attempted` | forbidden | forbidden | forbidden | forbidden | forbidden | not_checked | `060` |
| `not_authoritative_for_absence` | forbidden | forbidden | forbidden | forbidden | forbidden | unknown | `060` |
| `not_applicable` | no absence claim | forbidden | forbidden | forbidden | forbidden | not_applicable | `060` |

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `060-PATCH-AC-001` | Every coverage-sensitive negative or absence claim requires a current coverage assertion. |
| `060-PATCH-AC-002` | Every progress signal has one default interpretation and cannot authorize absence unless a policy row says so. |
| `060-PATCH-AC-003` | Source staleness and derived-view lag remain separate states. |
| `060-PATCH-AC-004` | Every watermark attempt emits exactly one `WatermarkCommitRecord`. |

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
| docs/archive/PRD-Cadastre.revised-draft.md | `SourceAuthorityProfile` | lines 4307-4671 |
| docs/archive/PRD-Cadastre.revised-draft.md | `SourceStalenessPolicy` | lines 4413-4477 |
| docs/archive/PRD-Cadastre.revised-draft.md | `SupplierCollectionVisibilityProfile` | lines 4478-4520 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ControlResultMappingRow` | lines 4521-4563 |
| docs/archive/PRD-Cadastre.revised-draft.md | `SourceHistoryRetentionProfile` | lines 4564-4595 |
| docs/archive/PRD-Cadastre.revised-draft.md | `DeriveAbsenceOrUnknown` | lines 4596-4671 |
| docs/archive/PRD-Cadastre.revised-draft.md | `LakehouseFeedCompletenessProfile` | lines 6330-6522 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ProgressSignalInterpretationPolicy` | lines 6406-6472 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ProjectionWatermarkPolicy` | lines 6473-6498 |
| docs/archive/PRD-Cadastre.revised-draft.md | `WatermarkCommitRecord` | lines 6499-6522 |
| docs/archive/PRD-Cadastre.revised-draft.md | `CoverageDimensionProfile and CoverageAssertion` | lines 7492-7636 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Source Authority Profiles` | lines 10678-10707 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Source Completeness Acceptance` | lines 13149-13160 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Source Authority Acceptance` | lines 13161-13171 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Source Authority, Staleness, Coverage, Progress, and Absence Acceptance` | lines 13610-13634 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
