---
doc_id: CADASTRE-NLSPEC-060
title: Source Authority, Completeness, Coverage, and Absence
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

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

- `GoldFactSchema`
- `FactAbsenceOutcome`
- `EvidenceRef`
- `ActivationControlledArtifactRef`

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

`DeriveAbsenceOrUnknown` may populate `040.GoldFact.absence_outcome` only through `040.GoldFactSchema` and only after exact authority, completeness, staleness, and coverage gates pass. Coverage-sensitive fact types or predicates must emit a `GoldFact` with non-empty `coverage_assertion_refs`; `coverage_assertion_refs = []` fails with `GOLD_FACT_COVERAGE_REQUIRED` for coverage-sensitive output.

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

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. The coverage catalog in this document supplies default dimensions; source-specific unresolved rows remain blocking until closed in the gap artifact and validation matrix.

## Source authority artifact activation boundary

`060` algorithms are stable core contracts. Source-specific authority, staleness, coverage, completeness, progress, control-result, history, watermark, and absence rows are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `SourceAuthorityProfile` and `SourceAuthorityProfileRow` row sets | `activation_controlled_artifact` | Must be active and exact before authority or absence effects. |
| `SourceStalenessPolicy` | `activation_controlled_artifact` | Must be checksummed and scoped before stale effects. |
| `SupplierCollectionVisibilityProfile` | `activation_controlled_artifact` | Must be active before visibility-dependent absence. |
| `ControlResultMappingRow` | `activation_controlled_artifact` | Must be active before control-state output. |
| `SourceHistoryRetentionProfile` | `activation_controlled_artifact` | Must be active before source-history interpretation. |
| `AbsenceDerivationPolicy` | `activation_controlled_artifact` | Must be active before absence/no-op/stale result selection. |
| `CoverageDimensionProfile` | stable catalog plus activation-controlled source-specific rows | Source-specific rows must be active before coverage-sensitive output. |
| `CoverageAssertion` | `runtime_state_record` | Evidence record consumed only through active profiles. |
| `LakehouseFeedCompletenessProfile` | `activation_controlled_artifact` | Must evaluate feed-read and upstream evidence before absence effects. |
| `ProgressSignalInterpretationPolicy` | `activation_controlled_artifact` | Weak signals remain non-authoritative unless an active row grants the exact effect. |
| `ProjectionWatermarkPolicy` | `activation_controlled_artifact` | Must be active before watermark advancement. |
| `WatermarkCommitRecord` | `runtime_state_record` | Records attempted watermark outcome; does not grant authority. |
| `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` | `stable_core_contract` | Algorithms validate activation refs before output. |

## DeriveAbsenceOrUnknown Algorithm

```text
DeriveAbsenceOrUnknown(candidate, evidence_set, receipt_set, upstream_evidence_set, completeness_profile, authority_profile, coverage_assertions, staleness_policy):
1. Validate every required activation artifact ref through `030.ActivationControlledArtifactRef`.
2. Fail before absence, cleanup, retraction, graph expiry, or watermark advancement when an authority, staleness, coverage, completeness, progress, control-result, or absence policy artifact is missing, inactive, checksum-mismatched, out of scope, or unvalidated.
3. Reject when no active SourceAuthorityProfileRow covers the fact type, predicate, source dataset, scopes, and requested absence authority.
4. Reject when required LakehouseReadCompletenessReceipt is missing or not tied to the active feed profile and manifest refs.
5. Evaluate UpstreamCompletenessEvidence through LakehouseFeedCompletenessProfile.
6. If any blocking unsafe completeness decision is present, return unknown with the most specific unsafe state.
7. Evaluate SourceStalenessPolicy using declared time-input precedence.
8. Require CoverageAssertion when the fact type or predicate is coverage-sensitive.
9. Reject weak progress signals unless one active ProgressSignalInterpretationPolicy row grants the exact effect.
10. Emit exactly one AbsenceDerivationResult: absence authorized, unknown, not applicable, stale, no-op, or deterministic error.
11. Include authority profile row refs, policy refs, artifact checksums, feed manifest refs, and validation refs in the result and `VersionManifest`.
```

### AbsenceDerivationResult schema

| Field | Required | Rule |
| --- | ---: | --- |
| `result_state` | Yes | Closed token: absence authorized, unknown, not applicable, stale, no-op, or deterministic error. |
| `absence_authorized` | Yes | Boolean; true only when authority, completeness, staleness, and coverage gates pass. |
| `blocking_reason` | Required when `absence_authorized = false` | Most specific blocking code. |
| `authority_row_ref` | Required when evaluated | Exact `SourceAuthorityProfileRow` ref or null when missing. |
| `completeness_decision_ref` | Required when evaluated | Lakehouse feed completeness decision ref. |
| `coverage_assertion_refs` | Required for coverage-sensitive outputs | Non-empty when coverage-sensitive output is authorized. |
| `staleness_decision_ref` | Required when staleness evaluated | Source staleness policy decision ref. |
| `version_manifest_ref` | Required for production output | Version manifest ref containing all output-affecting inputs. |

### Authority, coverage, and absence error codes

| Error/no-op code | Emitted when |
| --- | --- |
| `SOURCE_AUTHORITY_ROW_MISSING` | No exact active authority row covers fact type, predicate, dataset, scopes, and requested absence authority. |
| `COVERAGE_ASSERTION_REQUIRED` | Coverage-sensitive output lacks a current coverage assertion. |
| `COVERAGE_DIMENSION_UNRESOLVED` | Required coverage dimension has no active source-specific row. |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | Progress/liveness/freshness/lineage signal is used without an active policy row granting the effect. |
| `COMPLETENESS_DECISION_UNSAFE_FOR_ABSENCE` | Completeness decision is partial, unavailable, permission-limited, not attempted, not authoritative, or otherwise unsafe. |
| `SOURCE_AUTHORITY_ARTIFACT_MISSING` | A required source authority, staleness, coverage, completeness, progress, control-result, or absence policy artifact ref is missing. |
| `SOURCE_AUTHORITY_ARTIFACT_INACTIVE` | A required source authority artifact is not active for production execution. |
| `SOURCE_AUTHORITY_ARTIFACT_CHECKSUM_MISMATCH` | A required source authority artifact checksum mismatches the active ref or manifest. |
| `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` | An active source authority row set exists but no row covers the requested scope. |

## Watermarks

Every attempted production source, projection, graph-apply, or presence-only watermark change must be evaluated through `ProjectionWatermarkPolicy` and persisted as exactly one `WatermarkCommitRecord`. Failed, partial, aborted, schema-preflight-failed, or completeness-blocked runs must not advance watermarks.

## Authority-to-GoldFact linkage

`060` owns absence and coverage authorization causes but does not restate `040.GoldFactSchema`. The following cause codes may be recorded on a 040-shaped `GoldFact` validation or derivation failure.

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `GOLD_FACT_COVERAGE_REQUIRED` | `060` | A coverage-sensitive fact or predicate attempts output with `coverage_assertion_refs = []`. |
| `GOLD_FACT_AUTHORITY_ROW_MISSING` | `060` | No exact active `SourceAuthorityProfileRow` covers the requested fact, predicate, dataset, scopes, staleness, coverage, and absence authority. |

Missing rows, stale states, partial states, permission-limited states, weak progress signals, destination cleanup, omitted fields, missing lakehouse rows, and OCSF field absence must remain unable to create absence by themselves.

## Source Authority Contract Details

### CoverageDimensionProfile catalog

| Row ID | Domain | Dimension | Required evidence | Freshness policy | Permission-limited behavior | Partial behavior | Stale behavior | Default output | CoverageClosureStatus |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cov-vulnerability-default` | vulnerability | scanner target, credentialed status, plugin/check scope, scan window | coverage assertion plus scanner evidence | `SourceStalenessPolicy` | `permission_limited` blocks absence | `partial_known_gap` or `partial_unknown_gap` | stale blocks absence | unknown | `requires_source_specific_row` |
| `cov-control-default` | control | benchmark/check scope, applicability, evaluation status | `ControlResultMappingRow` and coverage assertion | owner row | not checked/unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-endpoint-default` | endpoint | asset scope, enrollment visibility, collection window | feed completeness plus authority row | owner row | blocks absence | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-directory-default` | directory | tenant/domain, group/member scope, hidden membership visibility | directory completeness evidence | owner row | blocks nonmembership | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dns-default` | DNS | zone/source scope, TTL, authoritative source | DNS feed evidence | TTL-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dhcp-ipam-default` | DHCP/IPAM | scope, lease window, authoritative system | lease/IPAM evidence | lease-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-flow-default` | flow | sensor scope, collection point, time window, role evidence | flow feed evidence | window policy | unknown | partial | stale | unknown | `requires_source_specific_row` |
| `cov-cloud-inventory-default` | cloud inventory | account/project/subscription, region, resource type | inventory feed and source history | source-specific | permission_limited | partial | stale | unknown | `requires_source_specific_row` |
| `cov-source-history-default` | source history | source-native history window, retention | history retention profile | history-window policy | unknown | unknown | outside window no proof | unknown | `requires_source_specific_row` |
| `cov-reachability-deferred` | deferred reachability | topology, route, policy, NAT, workload, identity context | inactive `200` placeholders | inactive | no MVP claim | no MVP claim | no MVP claim | no-op | `inactive_deferred` |

`CoverageClosureStatus` is a closed enum: `closed_default`, `requires_source_specific_row`, and `inactive_deferred`. Source-specific absence-sensitive outputs remain blocked until a `requires_source_specific_row` domain has a matching active source-specific coverage row.

### Coverage-sensitive blocked outputs

| Missing coverage domain | Blocked output |
| --- | --- |
| vulnerability | vulnerability absence and fixed-state absence. |
| control | control pass, fail, unknown, not-checked, and not-applicable facts. |
| endpoint | endpoint disappearance, non-observation, and cleanup. |
| directory | group non-membership, user non-membership, hidden-membership absence. |
| DNS | DNS absence, host deletion from TTL expiry, negative DNS fact. |
| DHCP/IPAM | DHCP lease absence, IPAM absence, host deletion from lease expiry. |
| flow | flow non-observation and observed-connection absence. |
| cloud inventory | missing cloud resource and resource deletion inference. |
| source history | no-change proof outside supported history window. |

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

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `060-CLEANUP-AC-001` | No banned reference class remains. |
| `060-CLEANUP-AC-002` | Broad source-category ranking still cannot act as fallback authority when the exact `SourceAuthorityProfileRow` is missing. |
| `060-CLEANUP-AC-003` | Progress, liveness, lineage, freshness, acknowledgment, CDC, graph-derived, and live-probe signals remain non-authoritative by default. |
| `060-CLEANUP-AC-004` | Missing rows, stale states, partial states, and permission-limited states still cannot authorize absence unless an active policy permits it. |
| `060-SCHEMA-PATCH-AC-001` | Coverage-sensitive `GoldFact` outputs fail when `coverage_assertion_refs = []`. |
| `060-SCHEMA-PATCH-AC-002` | `absence_outcome` can be materialized only by `DeriveAbsenceOrUnknown`. |
| `060-SCHEMA-PATCH-AC-003` | `060` does not restate `GoldFact` field definitions; it references `040.GoldFactSchema` by exact name. |

| `060-COVERAGE-CLOSURE-AC-001` | Every coverage catalog row has a `CoverageDimensionProfileRowId` and `CoverageClosureStatus`. |
| `060-ABSENCE-OUTPUT-AC-001` | `DeriveAbsenceOrUnknown` emits an `AbsenceDerivationResult` with authority, completeness, coverage, staleness, and manifest refs or a specific blocking reason. |
| `060-VOLATILITY-AC-001` | Missing exact source-authority row, inactive row set, staleness checksum mismatch, coverage row absence, and weak-signal combination cases fail or no-op with no absence, cleanup, retraction, graph expiry, or watermark advancement. |
| `060-VOLATILITY-AC-002` | Every output-affecting source-authority artifact ref appears in `VersionManifest` with checksum and validation refs. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `060-AC-001` | Missing, stale, partial, permission-limited, unsupported, or unavailable evidence cannot become absence unless an active profile authorizes it. |
| `060-AC-002` | Weak progress signals cannot be combined into stronger authority than each active policy row permits. |
| `060-AC-003` | Every coverage-dependent fact has a current coverage assertion or emits a deterministic unknown/error outcome. |
| `060-AC-004` | Source staleness and derived-view lag are exposed as distinct states. |
| `060-AC-005` | Every watermark attempt emits exactly one `WatermarkCommitRecord`. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
