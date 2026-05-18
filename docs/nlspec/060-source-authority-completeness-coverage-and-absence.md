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
- `LakehouseFeedCompletenessProfileRow`
- `ProgressSignalInterpretationPolicy`
- `ProjectionWatermarkPolicy`
- `WatermarkCommitRecord`
- `EvaluateLakehouseFeedCompleteness`
- `SourceAuthorityArtifactLifecycleGuardRows`
- `SourceAuthorityClosureMatrix`

## Source Authority Contract

Gold derivation must use an active `SourceAuthorityProfileRow` for the exact fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, staleness requirement, coverage requirement, and requested absence authority.

A broad source-category ranking must not act as fallback authority when the exact row is missing.

`DeriveAbsenceOrUnknown` may populate `040.GoldFact.absence_outcome` only through `040.GoldFactSchema` and only after exact authority, completeness, staleness, and coverage gates pass. Coverage-sensitive fact types or predicates must emit a `GoldFact` with non-empty `coverage_assertion_refs`; `coverage_assertion_refs = []` fails with `GOLD_FACT_COVERAGE_REQUIRED` for coverage-sensitive output.

### Correction authority handoff to 080

`080` must call `DeriveAbsenceOrUnknown` for any correction candidate based on missing evidence, stale evidence, source delete evidence, cleanup evidence, source-history no-change, control result absence, vulnerability fixed state, DNS absence, DHCP/IPAM expiry, flow non-observation, cloud disappearance, or CDC tombstone.

`060` decides whether negative, stale, cleanup, tombstone, or no-change evidence is authorized for a requested effect. `080` decides the correction operation. `090` decides projection and apply output. A correction or graph projection that consumes negative evidence directly, without an `AbsenceDerivationResult` ref for the requested effect, must fail before mutation.

### SourceAuthorityProfileRow schema

`SourceAuthorityProfileRow` is the stable row interface for fact authority and absence-sensitive effect authority. Concrete row instances are activation-controlled artifacts. Wildcards for `fact_type`, `predicate`, `source_dataset`, and requested effect are forbidden. Dataset-default rows may match only when `instance_default_allowed = true`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the active row set. |
| `fact_type` | Yes | none | Exact fact type. Wildcards are forbidden. |
| `predicate` | Yes | none | Exact predicate. Wildcards are forbidden. |
| `source_category` | Yes | none | Vendor-neutral source category. |
| `source_dataset` | Yes | none | Vendor-neutral source dataset. Wildcards are forbidden. |
| `source_instance_selector` | Yes | `dataset_default` only when `instance_default_allowed = true` | Exact source instance selector or dataset-default selector. |
| `instance_default_allowed` | Yes | `false` | `true` permits dataset-default matching only when no exact source-instance row exists. |
| `subject_scope_selector` | Yes | none | Canonical selector covering the subject scope. |
| `object_scope_selector` | Yes | none | Canonical selector covering the object scope. |
| `authority_mode` | Yes | none | One of `positive_only`, `positive_and_absence`, `control_result_only`, `history_only`, `non_authoritative`. |
| `allowed_effects` | Yes | `[]` | Closed array of `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`. Empty permits positive observations only. |
| `required_completeness_profile_row_refs` | Yes when any effect is allowed | `[]` only for `positive_only` | Exact `LakehouseFeedCompletenessProfileRow` refs. |
| `required_coverage_dimension_profile_refs` | Yes when coverage-sensitive | `[]` only for non-coverage-sensitive positive output | Exact `CoverageDimensionProfile` refs. |
| `required_staleness_policy_refs` | Yes when output or effect depends on stale state | none | Exact `SourceStalenessPolicy` refs. |
| `required_progress_signal_policy_refs` | Yes when progress signals are consulted | `[]` | Exact `ProgressSignalInterpretationPolicy` refs. |
| `required_control_result_mapping_refs` | Required for `control_result_only` or control output | `[]` for other modes | Exact `ControlResultMappingRowSet` refs. |
| `required_source_history_retention_refs` | Required for `history_only` or source-history no-change | `[]` for other modes | Exact `SourceHistoryRetentionProfile` refs. |
| `required_supplier_visibility_profile_refs` | Required for permission-sensitive absence | `[]` when visibility cannot affect the effect | Exact `SupplierCollectionVisibilityProfile` refs. |
| `absence_source_state_mapping` | Required when `allowed_effects` is non-empty | none | Map from source state to `040.FactAbsenceOutcome` token or blocking reason. |
| `validation_refs` | Yes | none | Non-empty refs proving positive, blocked, missing-row, ambiguous-row, and manifest behavior. |
| `activation_scope` | Yes | none | Vendor-neutral scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

Closed `authority_mode` values are:

```text
positive_only
positive_and_absence
control_result_only
history_only
non_authoritative
```

Closed `allowed_effects` values are:

```text
absence
cleanup
retraction
graph_expiry
watermark
```

### ResolveSourceAuthorityProfileRow

```text
ResolveSourceAuthorityProfileRow(request, active_row_set):
1. Validate row-set ref through `030.ActivationControlledArtifactRef`.
2. Keep only active rows whose activation scope covers the request.
3. Match exact fact_type, predicate, source_category, source_dataset, requested effect, subject scope, and object scope.
4. If request names a source instance, select only rows for that exact instance unless no specific row exists.
5. Permit a dataset-default row only when `instance_default_allowed = true`.
6. If zero rows remain, return `SOURCE_AUTHORITY_ROW_MISSING`.
7. If more than one equally specific row remains, return `SOURCE_AUTHORITY_ROW_AMBIGUOUS`.
8. Return the selected row and require its ref and checksum in `VersionManifest`.
```

## Completeness Contract

`LakehouseFeedCompletenessProfile` maps feed-read completeness and supplier-provided upstream evidence into source completeness decisions. A `LakehouseReadCompletenessReceipt` alone must not authorize absence, retraction, cleanup, graph expiry, or watermark advancement.

`LakehouseFeedCompletenessProfileRow` is the activation-controlled row shape that makes feed-read completeness decisions scope-exact. Missing rows, missing `allowed_effects`, or missing upstream evidence must not be interpreted as permission for absence, cleanup, retraction, graph expiry, or watermark advancement.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `profile_row_id` | Yes | none | Stable row ID scoped to `060` and included in the row-set checksum. |
| `feed_category` | Yes | none | Must match `020.LakehouseFeedCategoryClosureRow.feed_category`. |
| `source_dataset` | Yes | none | Vendor-neutral dataset token. |
| `scope_selector` | Yes | none | Canonical selector for the source/feed scope; broad source-category scope is not a fallback. |
| `read_target_kind` | Yes | none | Closed read target kind imported from `020`. |
| `receipt_state` | Yes | none | One `020.LakehouseReadCompletenessReceipt` state. |
| `upstream_evidence_class` | Yes | none | Supplier evidence class required by the category row. |
| `upstream_evidence_state` | Yes | none | Closed token: `sufficient`, `insufficient`, `missing`, `permission_limited`, `scope_unavailable`, `source_unavailable`, `stale`, `not_applicable`. |
| `coverage_domain` | No | null | Required when the requested effect is coverage-sensitive. |
| `completeness_decision` | Yes | none | One completeness decision from the completeness table. |
| `allowed_effects` | No | `[]` | Closed effect tokens: `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`. Missing or empty means no effects are allowed. |
| `blocking_reason` | Required when decision blocks an effect | null | Most specific blocking code from the blocking precedence table. |
| `source_authority_row_refs` | No | `[]` | Exact authority refs required for effects; broad source-category rankings are invalid. |
| `source_staleness_policy_refs` | No | `[]` | Exact staleness policy refs required for effects. |
| `validation_refs` | Yes | none | Non-empty refs proving the row's positive and negative behavior. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |

A row with `completeness_decision in {complete, empty_complete}` may enter absence evaluation only when `allowed_effects` contains the requested effect and all exact authority, coverage, and staleness refs validate.

For every active absence-sensitive feed profile, source dataset, scope selector, read target kind, requested effect, and upstream evidence class, the active row set must cover every `LakehouseReadCompletenessReceipt` state and every `upstream_evidence_state`. Missing combinations emit `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING`. Receipt states are `read_complete`, `read_partial_known_gap`, `read_partial_unknown_gap`, `read_unavailable`, `schema_unavailable`, and `manifest_invalid`. Upstream evidence states are `sufficient`, `insufficient`, `missing`, `permission_limited`, `scope_unavailable`, `source_unavailable`, `stale`, and `not_applicable`.

| Completeness decision | Absence authority default | Cleanup/retraction default |
| --- | --- | --- |
| `complete` | May be considered only when `LakehouseFeedCompletenessProfileRow.allowed_effects` contains `absence`. | May be considered only when `allowed_effects` contains the requested effect. |
| `empty_complete` | May be considered only when `LakehouseFeedCompletenessProfileRow.allowed_effects` contains `absence`. | May be considered only when `allowed_effects` contains the requested effect. |
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

Two or more progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, or live-probe signals that individually lack authority must not combine into stronger authority unless exactly one active `ProgressSignalInterpretationPolicy` row grants the exact combined signal set and requested effect.

## Coverage Contract

Coverage-sensitive facts require `CoverageDimensionProfile` and a current `CoverageAssertion` before absence, pass/fail/unknown, no-change, or negative claims may be emitted.

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. The coverage catalog in this document supplies default dimensions. Concrete active source-specific row instances are activation-controlled and validation-blocked until present; missing rows fail closed through `SourceAuthorityClosureMatrix` and `120` validation rows.

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
| `LakehouseFeedCompletenessProfile` and `LakehouseFeedCompletenessProfileRow` row sets | `activation_controlled_artifact` | Must evaluate feed-read and upstream evidence before absence effects and must declare allowed effects explicitly. |
| `ProgressSignalInterpretationPolicy` | `activation_controlled_artifact` | Weak signals remain non-authoritative unless an active row grants the exact effect. |
| `ProjectionWatermarkPolicy` | `activation_controlled_artifact` | Must be active before watermark advancement. |
| `WatermarkCommitRecord` | `runtime_state_record` | Records attempted watermark outcome; does not grant authority. |
| `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` | `stable_core_contract` | Algorithms validate activation refs before output. |

### SourceAuthorityClosureMatrix

`SourceAuthorityClosureMatrix` is a stable validation view over active `060` activation-controlled row sets and imported `020` feed-category closure rows. It is not a runtime authority class and must not substitute for the underlying row refs.

For every active `LakehouseFeedProfile` and requested absence-sensitive effect, the matrix must prove that exactly one active validated row chain exists for fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, completeness, coverage, staleness, progress-signal interpretation, control-result mapping when applicable, source-history retention when applicable, absence derivation, and watermark policy when applicable.

If the row chain is absent, ambiguous, inactive, checksum-mismatched, out of scope, stale, or unvalidated, the effect must not execute. The result must be `unknown`, `not_applicable`, `source_stale`, `no_op`, or the most specific deterministic error. It must not emit absence, cleanup, retraction, graph expiry, pass/fail, no-change proof, or watermark advancement.

The matrix output must include the requested effect token, feed category, source dataset, selected row refs, row checksums, validation refs, blocking reason when blocked, and a `VersionManifest` ref. A closure validation result may be referenced by `030.VersionManifest`, but the underlying activation-controlled artifact refs and checksums remain required.

### SourceAuthorityArtifactLifecycleGuardRows

`060` policy row sets use `030.ActivationControlledArtifactLifecycleMachine.v1` for activation. Runtime completeness, absence, and watermark decisions remain algorithmic unless a future owner patch defines a lifecycle subject, closed states, closed events, a total transition matrix, and validation rows.

| Artifact class | Lifecycle binding | Required owner guards |
| --- | --- | --- |
| `SourceAuthorityProfile` and row set | `030.ActivationControlledArtifactLifecycleMachine.v1` | Exact fact, predicate, dataset, and scope coverage; no broad fallback; validation refs. |
| `LakehouseFeedCompletenessProfileRow` row set | Generic artifact lifecycle | Exact feed category, read target, receipt state, upstream evidence, allowed effects, and validation refs. |
| `CoverageDimensionProfile` source-specific rows | Generic artifact lifecycle | Coverage domain, dimension states, staleness rules, permission behavior, and fixture refs. |
| `ProgressSignalInterpretationPolicy` | Generic artifact lifecycle | Signal class, default non-authority, exact granted effect, and weak-signal combination fixture. |
| `ProjectionWatermarkPolicy` | Generic artifact lifecycle | Required completeness, apply status, and no advancement after failed or partial apply. |
| `AbsenceDerivationPolicy` | Generic artifact lifecycle | Absence, no-op, stale, error result mapping, and blocking precedence fixture. |
| `SourceAuthorityClosureMatrix` | Stable validation view | Exact row-chain coverage, ambiguity detection, checksum validation, deterministic block rows, and validation refs. |

### Pure algorithm lifecycle non-substitution rule

`EvaluateLakehouseFeedCompleteness`, `DeriveAbsenceOrUnknown`, and watermark evaluation are pure deterministic algorithms for MVP. They must not be converted into lifecycle machines unless a future owner patch defines a lifecycle subject, closed states, closed events, total transition matrix, and validation rows.

## EvaluateLakehouseFeedCompleteness Algorithm

```text
EvaluateLakehouseFeedCompleteness(receipt, upstream_evidence_set, active_feed_completeness_profile_row_set, feed_category_closure_row, authority_profile_rows, coverage_assertions, staleness_policy_refs, requested_effect):
1. Validate every required activation artifact ref through `030.ActivationControlledArtifactRef`.
2. Select candidate `LakehouseFeedCompletenessProfileRow` rows by exact feed category, source dataset, scope selector, read target kind, receipt state, upstream evidence class, and activation scope.
3. If no row matches, return `not_authoritative_for_absence` with `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING`.
4. Apply `CompletenessBlockingPrecedence` to receipt state, upstream evidence state, manifest validity, source availability, permission, partial gaps, staleness, authority, coverage, and not-applicable state.
5. If a blocking condition exists, return exactly one completeness decision, selected row refs, and the most specific blocking reason.
6. If `requested_effect` is absent from `allowed_effects`, return the row decision with `COMPLETENESS_EFFECT_NOT_ALLOWED` and no effect.
7. Validate exact authority profile row refs, coverage assertion refs when required, and staleness policy refs.
8. Emit exactly one completeness decision: `complete`, `empty_complete`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `permission_limited`, `not_attempted`, `not_authoritative_for_absence`, or `not_applicable`.
9. Include selected row refs, upstream evidence refs, authority refs, coverage refs, staleness refs, blocking reason, requested effect, and validation refs in the result and `VersionManifest`.
```

### CompletenessBlockingPrecedence

| Precedence | Blocking condition | Required decision | Blocking reason |
| ---: | --- | --- | --- |
| 1 | Required activation artifact missing, inactive, checksum-mismatched, out of scope, or unvalidated. | deterministic error or `not_authoritative_for_absence` | owner-specific activation error |
| 2 | Manifest invalid or required schema unavailable. | `partial_unknown_gap` or deterministic error | `manifest_invalid` or `schema_unavailable` |
| 3 | Source or lakehouse target unavailable. | `source_unavailable` | `source_unavailable` |
| 4 | Scope unavailable. | `scope_unavailable` | `scope_unavailable` |
| 5 | Permission limited or visibility limited. | `permission_limited` | `permission_limited` |
| 6 | Partial unknown gap. | `partial_unknown_gap` | `partial_unknown_gap` |
| 7 | Partial known gap. | `partial_known_gap` | `partial_known_gap` |
| 8 | Stale source state blocks requested effect. | `not_authoritative_for_absence` | `stale` |
| 9 | Exact authority, coverage, or effect permission missing. | `not_authoritative_for_absence` | most specific missing authority, coverage, or effect code |
| 10 | Not applicable by exact profile row. | `not_applicable` | `not_applicable` |
| 11 | Empty complete with effect allowed. | `empty_complete` | none |
| 12 | Complete with effect allowed. | `complete` | none |

Only `complete` and `empty_complete` may enter absence evaluation, and only when `allowed_effects` contains the requested effect.

## DeriveAbsenceOrUnknown Algorithm

```text
DeriveAbsenceOrUnknown(candidate, requested_effect, evidence_set, receipt_set, upstream_evidence_set, completeness_profile, authority_profile, coverage_assertions, staleness_policy):
1. Validate every required activation artifact ref through `030.ActivationControlledArtifactRef`.
2. Fail before absence, cleanup, retraction, graph expiry, or watermark advancement when an authority, staleness, coverage, completeness, progress, control-result, source-history, visibility, or absence policy artifact is missing, inactive, checksum-mismatched, out of scope, or unvalidated.
3. Validate `requested_effect` against the requested-effect gate table. Missing, unknown, or omitted effects block with `COMPLETENESS_EFFECT_NOT_ALLOWED`.
4. Resolve exactly one active `SourceAuthorityProfileRow` through `ResolveSourceAuthorityProfileRow`; missing or ambiguous rows block the requested effect.
5. Reject when required `LakehouseReadCompletenessReceipt` is missing or not tied to the active feed profile and manifest refs.
6. Evaluate `UpstreamCompletenessEvidence` and the read receipt through `EvaluateLakehouseFeedCompleteness` for the requested effect.
7. If the completeness decision is not `complete` or `empty_complete`, return unknown, not applicable, stale, no-op, or deterministic error with the selected blocking reason.
8. Reject when the selected completeness row's `allowed_effects` omits the requested effect.
9. Evaluate `SourceStalenessPolicy` using declared time-input precedence.
10. Require `CoverageAssertion` when the fact type or predicate is coverage-sensitive.
11. Reject weak progress signals unless one active `ProgressSignalInterpretationPolicy` row grants the exact effect.
12. Reject source-history no-change unless an active `SourceHistoryRetentionProfile` row covers the exact dataset, scope, and history window.
13. Reject supplier visibility or permission-sensitive absence unless required `SupplierCollectionVisibilityProfile` refs validate.
14. Reject external-schema classification, status, severity, confidence, observables, enrichments, field presence, field absence, endpoint order, CDC tombstone, source delete marker, or cleanup evidence as authority unless an active `060` row maps the exact signal to the exact requested effect.
15. Emit exactly one `AbsenceDerivationResult` whose `result_state` maps one-to-one to `040.FactAbsenceOutcome` when a fact-level outcome is produced.
16. Include requested effect, allowed effects, authority profile row refs, completeness row refs, policy refs, artifact checksums, feed manifest refs, input evidence refs, result checksum, and validation refs in the result and `VersionManifest`.
```

### AbsenceDerivationResult schema

| Field | Required | Rule |
| --- | ---: | --- |
| `result_state` | Yes | Closed token from `040.FactAbsenceOutcome` when fact-level outcome is produced; `no_op` or owner error when no fact-level outcome is produced. |
| `absence_authorized` | Yes | Boolean; true only when authority, completeness, staleness, and coverage gates pass. |
| `blocking_reason` | Required when `absence_authorized = false` | Most specific blocking code. |
| `authority_row_ref` | Required when evaluated | Exact `SourceAuthorityProfileRow` ref or null when missing. |
| `completeness_decision_ref` | Required when evaluated | Lakehouse feed completeness decision ref. |
| `coverage_assertion_refs` | Required for coverage-sensitive outputs | Non-empty when coverage-sensitive output is authorized. |
| `staleness_decision_ref` | Required when staleness evaluated | Source staleness policy decision ref. |
| `version_manifest_ref` | Required for production output | Version manifest ref containing all output-affecting inputs. |
| `requested_effect` | Yes | One requested-effect token from the gate table. |
| `allowed_effects` | Yes | Closed set selected from the active authority and completeness rows. Empty means no absence-sensitive effect is authorized. |
| `absence_policy_ref` | Required when absence policy evaluated | Active `AbsenceDerivationPolicy` row ref or null when blocked before policy selection. |
| `progress_signal_policy_refs` | Yes | Canonically sorted consulted progress-signal policy refs; empty when no progress signals were consulted. |
| `control_result_mapping_refs` | Yes | Canonically sorted consulted control-result mapping refs. |
| `source_history_retention_refs` | Yes | Canonically sorted consulted source-history retention refs. |
| `supplier_visibility_refs` | Yes | Canonically sorted consulted visibility refs. |
| `input_evidence_refs` | Yes | Canonically sorted raw, silver, source-state, delete, tombstone, cleanup, or source-history evidence refs. |
| `result_checksum` | Yes | SHA-256 over canonical result bytes excluding only `result_checksum`. |

### Requested-effect gate table

| Requested effect | Required authorization | Default when gate fails | Downstream owner |
| --- | --- | --- | --- |
| `absence` | Exact authority row with `allowed_effects` containing `absence`, complete or empty-complete decision, required coverage, and non-stale state unless policy permits. | `unknown` or deterministic owner error; no negative fact. | `060`, consumed by `080`. |
| `cleanup` | Exact authority row and completeness row with `allowed_effects` containing `cleanup`; cleanup evidence must be authorized, not merely destination cleanup. | no-op with no cleanup. | `060`, consumed by `080` and `090`. |
| `retraction` | Exact authority row and completeness row with `allowed_effects` containing `retraction`; delete or tombstone evidence must pass all gates. | no-op with no retraction. | `060`, consumed by `080`. |
| `graph_expiry` | Exact authority row and completeness row with `allowed_effects` containing `graph_expiry`; graph expiry is never authorized by graph state alone. | no-op with no graph delta. | `060`, consumed by `090`. |
| `watermark` | Exact authority row, completeness row, and `ProjectionWatermarkPolicy` permitting the exact watermark target and effect. | no watermark advancement. | `060`. |

Stale-source output may create a stale known-state only when the active `SourceStalenessPolicy` row permits stale state materialization. Stale source state does not authorize retraction, cleanup, graph expiry, or watermark by default.

CDC delete markers and tombstones are raw delete evidence only. They authorize retraction only when exact authority, completeness, coverage, staleness, source-history when applicable, and requested-effect gates pass. Unauthorized CDC tombstones must produce no retraction, cleanup, graph expiry, absence, or watermark advancement.

### Authority, coverage, and absence error codes

| Error/no-op code | Emitted when |
| --- | --- |
| `SOURCE_AUTHORITY_ROW_MISSING` | No exact active authority row covers fact type, predicate, dataset, scopes, and requested absence authority. |
| `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | More than one equally specific active authority row covers the requested fact, predicate, dataset, scopes, and requested effect. |
| `SOURCE_STALENESS_POLICY_ROW_MISSING` | No active staleness policy row covers the requested source dataset, fact type, predicate, scope, and effect. |
| `SOURCE_STALENESS_POLICY_ROW_AMBIGUOUS` | More than one equally specific staleness policy row covers the request. |
| `COVERAGE_DIMENSION_ROW_MISSING` | No source-specific coverage dimension row covers the requested coverage domain, dataset, fact type, predicate, and scope. |
| `COVERAGE_DIMENSION_ROW_AMBIGUOUS` | More than one equally specific coverage dimension row covers the request. |
| `CONTROL_RESULT_MAPPING_ROW_MISSING` | No active control-result mapping row covers the external vocabulary, rule polarity, external state, fact type, predicate, and scope. |
| `CONTROL_RESULT_MAPPING_ROW_AMBIGUOUS` | More than one equally specific control-result mapping row covers the request. |
| `PROGRESS_SIGNAL_POLICY_ROW_MISSING` | A progress or weak-signal effect is requested without an active exact progress-signal policy row. |
| `PROGRESS_SIGNAL_POLICY_ROW_AMBIGUOUS` | More than one equally specific progress-signal policy row covers the request. |
| `SOURCE_HISTORY_RETENTION_ROW_MISSING` | Source-history interpretation is requested without an active retention row covering dataset, scope, and history window. |
| `SOURCE_HISTORY_RETENTION_ROW_AMBIGUOUS` | More than one equally specific source-history retention row covers the request. |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_MISSING` | Permission-sensitive absence is requested without an active visibility profile row. |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_AMBIGUOUS` | More than one equally specific visibility profile row covers the request. |
| `SOURCE_STATE_MAPPING_ROW_MISSING` | No active absence source-state mapping covers the selected source state and requested effect. |
| `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE` | `SourceAuthorityClosureMatrix` cannot prove exactly one validated row chain for an active absence-sensitive request. |
| `SOURCE_AUTHORITY_CLOSURE_AMBIGUOUS` | `SourceAuthorityClosureMatrix` finds more than one equally specific validated row chain. |
| `COVERAGE_ASSERTION_REQUIRED` | Coverage-sensitive output lacks a current coverage assertion. |
| `COVERAGE_DIMENSION_UNRESOLVED` | Required coverage dimension has no active source-specific row. |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | Progress/liveness/freshness/lineage signal is used without an active policy row granting the effect. |
| `COMPLETENESS_DECISION_UNSAFE_FOR_ABSENCE` | Completeness decision is partial, unavailable, permission-limited, not attempted, not authoritative, or otherwise unsafe. |
| `SOURCE_AUTHORITY_ARTIFACT_MISSING` | A required source authority, staleness, coverage, completeness, progress, control-result, or absence policy artifact ref is missing. |
| `SOURCE_AUTHORITY_ARTIFACT_INACTIVE` | A required source authority artifact is not active for production execution. |
| `SOURCE_AUTHORITY_ARTIFACT_CHECKSUM_MISMATCH` | A required source authority artifact checksum mismatches the active ref or manifest. |
| `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` | An active source authority row set exists but no row covers the requested scope. |
| `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | No active completeness profile row covers feed category, dataset, scope selector, target kind, receipt state, upstream evidence class, and activation scope. |
| `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | Required upstream evidence is missing, stale, permission-limited, scope-limited, or insufficient for the requested effect. |
| `COMPLETENESS_EFFECT_NOT_ALLOWED` | The selected completeness row does not list the requested effect in `allowed_effects`. |
| `COMPLETENESS_BLOCKING_PRECEDENCE_APPLIED` | A higher-precedence blocking condition selected the completeness decision and blocked a lower-precedence effect. |
| `EMPTY_SCOPE_NOT_AUTHORIZED_FOR_ABSENCE` | An empty scope is complete for the feed read but not authorized for absence by exact completeness, authority, coverage, and staleness rows. |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | Watermark advancement is requested while completeness, upstream evidence, authority, coverage, or staleness gates block the effect. |
| `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | CDC tombstone or delete marker attempts retraction without exact authority, completeness, coverage, staleness, and requested-effect authorization. |
| `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | A gold derivation, absence derivation, control-state output, cleanup, retraction, graph expiry, or watermark decision attempts to use external schema classification, status, severity, confidence, field presence, field absence, observable, enrichment, or endpoint order as authority without an active `060` row. |

## Watermarks

Every attempted production source, projection, graph-apply, or presence-only watermark change must be evaluated through `ProjectionWatermarkPolicy` and persisted as exactly one `WatermarkCommitRecord`. Failed, partial, aborted, schema-preflight-failed, or completeness-blocked runs must not advance watermarks.

## Authority-to-GoldFact linkage

`060` owns absence and coverage authorization causes but does not restate `040.GoldFactSchema`. The following cause codes may be recorded on a 040-shaped `GoldFact` validation or derivation failure.

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `GOLD_FACT_COVERAGE_REQUIRED` | `060` | A coverage-sensitive fact or predicate attempts output with `coverage_assertion_refs = []`. |
| `GOLD_FACT_AUTHORITY_ROW_MISSING` | `060` | No exact active `SourceAuthorityProfileRow` covers the requested fact, predicate, dataset, scopes, staleness, coverage, and absence authority. |

Missing rows, stale states, partial states, permission-limited states, weak progress signals, destination cleanup, omitted fields, missing lakehouse rows, and OCSF field absence must remain unable to create absence by themselves.

### External schema non-authority boundary

OCSF class, category, activity, type, object paths, observables, enrichments, status, severity, confidence, endpoint order, field presence, and field absence are normalized observation metadata only. They must not satisfy `SourceAuthorityProfileRow`, `CoverageAssertion`, `SourceStalenessPolicy`, `LakehouseFeedCompletenessProfileRow`, `AbsenceDerivationPolicy`, or `ControlResultMappingRow` requirements unless an active `060`-owned row explicitly maps the exact signal to the exact effect.

A `060` row that maps an external-schema signal must name the external schema profile ref, field path, source dataset, fact type, predicate, requested effect, source authority row ref, coverage row ref when applicable, staleness row ref, validation refs, activation scope, and lifecycle status. Omission of any required ref means the external-schema signal remains non-authoritative.

Vulnerability Finding status must not become Cadastre `assertion_state` without a `060` or `080` derivation path. DNS and DHCP field absence must not become absence without exact completeness, coverage, staleness, and authority rows. OCSF endpoint order must not become graph direction authority.

## Source Authority Contract Details

### CoverageDimensionProfile catalog

`CoverageDimensionProfile` rows are stable at the schema level and activation-controlled at the source-specific row level.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `coverage_profile_row_id` | Yes | none | Stable row ID scoped to the row set. |
| `coverage_domain` | Yes | none | One of the domains declared in the catalog below or a deferred domain with explicit inactive status. |
| `source_category` | Yes | none | Vendor-neutral source category. |
| `source_dataset` | Yes | none | Vendor-neutral source dataset. |
| `fact_type` | Yes | none | Exact fact type. |
| `predicate` | Yes | none | Exact predicate. |
| `scope_selector` | Yes | none | Canonical source/feed scope selector. |
| `required_dimensions` | Yes | none | Non-empty ordered set of required dimensions for the domain. |
| `authorized_dimension_states` | Yes | `covered` only | Authorized states are `covered` and explicitly permitted `empty_covered` only. |
| `blocking_dimension_states` | Yes | all blocking states | Must include `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `stale`, `unsupported`, `not_checked`, and `error` unless a narrower owner row maps a non-negative result. |
| `freshness_policy_ref` | Yes | none | Exact `SourceStalenessPolicy` ref for coverage freshness. |
| `visibility_profile_ref` | Required when permissions affect visibility | null only when permissions cannot affect coverage | Exact `SupplierCollectionVisibilityProfile` ref. |
| `validation_refs` | Yes | none | Non-empty positive and negative validation refs. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

`ResolveCoverageDimensionProfile(request, active_row_set)` must validate the row set through `030.ActivationControlledArtifactRef`, select active rows whose activation scope covers the request, match exact coverage domain, source category, source dataset, fact type, predicate, and scope selector, return `COVERAGE_DIMENSION_ROW_MISSING` when no row matches, return `COVERAGE_DIMENSION_ROW_AMBIGUOUS` when more than one equally specific row matches, and include the selected ref and checksum in `VersionManifest`.

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

### FeedCategoryCoverageEffectMatrix

This table aligns to `020.LakehouseFeedCategoryClosureRequirementTable` without restating feed-read target mechanics. Exact activation-controlled rows must provide the source-specific refs before effects are allowed.

| Feed category | Required completeness, coverage, and authority dimensions before effects | Effects that may be allowed by exact row | Missing or insufficient evidence result |
| --- | --- | --- | --- |
| `endpoint_inventory` | endpoint scope, enrollment or inventory visibility, collection window, failed-scope evidence, source authority, staleness, coverage assertion | `absence`, `cleanup`, `graph_expiry`, `watermark` | `not_authoritative_for_absence` or specific partial/permission state |
| `configuration_inventory` | configuration scope, object type, source authority, staleness, coverage assertion for affected domain | `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark` | `not_authoritative_for_absence` |
| `vulnerability_scan` | scanner target, credential/auth status, plugin/check set, scan window, failed-target/exclusion evidence, coverage assertion, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `COVERAGE_ASSERTION_REQUIRED` |
| `control_evaluation` | benchmark/check scope, applicability, control result mapping, evaluation status, coverage assertion, staleness, authority rows | `absence`, `retraction`, `watermark` | `not_checked`, `unknown`, or exact blocking reason; no pass/fail by default |
| `directory_inventory` | tenant/domain, object type, hidden-object permission, page/delta completion, coverage, staleness, authority rows | `absence`, `cleanup`, `graph_expiry`, `watermark` | `permission_limited`, `partial_known_gap`, or `not_authoritative_for_absence` |
| `directory_membership` | tenant/domain, group, member type, direct/transitive mode, hidden-membership permission, page/delta completion, AD primary-group handling, coverage, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, or `not_authoritative_for_absence` |
| `dns_record_set` | zone/source scope, authoritative source, TTL basis, collection window, coverage, staleness, authority rows | `absence`, `retraction`, `watermark` | TTL expiry alone maps to `not_authoritative_for_absence` |
| `dhcp_ipam_assignment` | DHCP/IPAM scope, lease window, authoritative-system evidence, coverage, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | lease expiry alone maps to `not_authoritative_for_absence` |
| `network_flow` | sensor scope, collection point, time window, role evidence, coverage, staleness, authority rows | `watermark` only when exact row permits; absence effects default blocked | missing flow maps to `unknown` |
| `cloud_asset_inventory` | account/project/subscription, region, resource type, permission visibility, source-history requirement when applicable, coverage, staleness, authority rows | `absence`, `cleanup`, `graph_expiry`, `watermark` | `permission_limited`, `stale`, or `not_authoritative_for_absence` |
| `source_history` | source-native history window, retention profile, query scope, authority rows, staleness rows | `absence`, `watermark` only within supported window and exact row permission | outside-window no-result maps to `not_authoritative_for_absence` |
| `future_reachability` | inactive deferred reachability contracts only | none in MVP | deterministic no-op or deferred reachability error |

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

Two or more weak progress signals must not combine into stronger authority unless exactly one active `ProgressSignalInterpretationPolicy` row grants the exact combined signal set and requested effect.

### SourceStalenessPolicy

`SourceStalenessPolicy` rows must keep source event time, known time, table time, connector time, watermark time, replay time, and graph apply time distinct. Graph derived-view lag must not become source staleness.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to the row set. |
| `source_category` | Yes | none | Vendor-neutral source category. |
| `source_dataset` | Yes | none | Vendor-neutral source dataset. |
| `fact_type` | Yes | none | Exact fact type. |
| `predicate` | Yes | none | Exact predicate. |
| `scope_selector` | Yes | none | Canonical scope selector. |
| `time_input_precedence` | Yes | none | Ordered time-input list. Current platform time is forbidden unless explicitly named and validation-covered for a non-authority diagnostic. |
| `required_time_inputs` | Yes | none | Non-empty set unless `staleness_not_applicable = true`. |
| `max_age_seconds` or `expiry_rule` | Yes | none | Exactly one staleness bound mechanism must be present unless `staleness_not_applicable = true`. |
| `staleness_not_applicable` | No | `false` | `true` maps staleness to `not_applicable` for the row scope only. |
| `missing_time_behavior` | Yes | `unknown` | Closed enum: `unknown`, `stale`, `deterministic_error`. |
| `stale_effects` | Yes | block all absence-sensitive effects | Default blocks `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. |
| `version_manifest_inclusion` | Yes | required | Selected row ref, checksum, and selected time input must appear in `VersionManifest`. |
| `validation_refs` | Yes | none | Non-empty refs for missing, malformed, stale, current, and replay cases. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

Closed `missing_time_behavior` values are:

```text
unknown
stale
deterministic_error
```

Missing time input must never fall back to current platform time. A stale result blocks `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark` unless a narrower exact owner row explicitly maps a non-negative result.

### ControlResultMappingRow

`ControlResultMappingRow` must map external control states without collapsing unknown, error, not evaluated, not checked, or not applicable into pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `mapping_row_id` | Yes | none | Stable row ID scoped to the row set. |
| `external_vocabulary` | Yes | none | External vocabulary or profile identifier. |
| `rule_polarity` | Yes | none | Exact control polarity or owner-defined polarity token. |
| `external_state` | Yes | none | Exact source state token. |
| `cadastre_state` | Yes | none | One closed Cadastre state from the default table or owner extension. |
| `applicability_condition` | Yes | none | Exact condition under which the mapping applies. |
| `required_coverage_refs` | Yes | `[]` only when the control is not coverage-sensitive | Exact coverage refs. |
| `required_authority_refs` | Yes | none | Exact authority refs. |
| `required_staleness_refs` | Yes | none | Exact staleness refs. |
| `validation_refs` | Yes | none | Non-empty refs for each mapped and unmapped state. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

| External state | Cadastre state | Default effect |
| --- | --- | --- |
| `pass` | `pass` | Output only when exact authority, coverage, staleness, applicability, and completeness rows pass. |
| `fail` | `fail` | Output only when exact authority, coverage, staleness, applicability, and completeness rows pass. |
| `unknown` | `unknown` | No pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark. |
| `error` | `error` | No pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark. |
| `not evaluated` | `not_checked` | No pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark. |
| `not checked` | `not_checked` | No pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark. |
| `not applicable` | `not_applicable` | No pass/fail unless a row explicitly permits a not-applicable fact. |

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
| `complete` | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | observed or authorized_not_observed | `060` |
| `empty_complete` | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | authorized_not_observed | `060` |
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
| `060-ABSENCE-OUTPUT-AC-001` | `DeriveAbsenceOrUnknown` emits an `AbsenceDerivationResult` with authority, completeness, coverage, staleness, requested effect, allowed effects, result checksum, and manifest refs or a specific blocking reason. |
| `060-CORRECTION-HANDOFF-AC-001` | `080` correction candidates based on missing, stale, delete, cleanup, no-change, fixed-state, DNS, DHCP/IPAM, flow non-observation, cloud disappearance, or CDC tombstone evidence carry an `AbsenceDerivationResult` ref before correction output. |
| `060-REQUESTED-EFFECT-AC-001` | Missing or omitted requested effect blocks absence, cleanup, retraction, graph expiry, and watermark. |
| `060-STALE-NO-EFFECT-AC-001` | Stale source state without a staleness row permitting materialization emits no retraction, cleanup, graph expiry, or watermark. |
| `060-CDC-TOMBSTONE-AC-001` | Unauthorized CDC tombstone evidence emits no absence, retraction, cleanup, graph expiry, or watermark. |
| `060-VOLATILITY-AC-001` | Missing exact source-authority row, inactive row set, staleness checksum mismatch, coverage row absence, and weak-signal combination cases fail or no-op with no absence, cleanup, retraction, graph expiry, or watermark advancement. |
| `060-VOLATILITY-AC-002` | Every output-affecting source-authority artifact ref appears in `VersionManifest` with checksum and validation refs. |
| `060-FEED-CLOSURE-AC-001` | Missing `LakehouseFeedCompletenessProfileRow` maps to `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` and no absence-sensitive effect. |
| `060-FEED-CLOSURE-AC-002` | Missing upstream completeness evidence maps to `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `not_authoritative_for_absence`, not absence. |
| `060-FEED-CLOSURE-AC-003` | `allowed_effects = []` or omitted allowed effects blocks absence, cleanup, retraction, graph expiry, and watermark. |
| `060-FEED-CLOSURE-AC-004` | Blocking precedence is deterministic and selects the same blocking reason for identical evidence, rows, refs, and requested effect. |
| `060-FEED-CLOSURE-AC-005` | Complete or empty-complete decisions authorize effects only through exact active authority, completeness, coverage, and staleness rows. |
| `060-OCSF-NONAUTH-AC-001` | OCSF status, severity, confidence, observables, enrichments, class, activity, field presence, and field absence cannot satisfy authority, completeness, coverage, staleness, control-result, or absence gates by themselves. |
| `060-OCSF-NONAUTH-AC-002` | Vulnerability Finding status does not become Cadastre `assertion_state` without a `060` or `080` owned derivation path. |
| `060-OCSF-NONAUTH-AC-003` | DNS or DHCP field absence does not become absence without exact completeness, coverage, staleness, and authority rows. |
| `060-LIFECYCLE-AC-001` | Source authority, completeness, coverage, progress, absence, and watermark artifacts can become active only through `030.LifecycleTransitionEvidence`, while `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` remain pure deterministic algorithms. |
| `060-SOURCE-CLOSURE-AC-001` | Missing exact source-authority row blocks the requested effect and emits no absence, cleanup, retraction, graph expiry, pass/fail, no-change proof, or watermark advancement. |
| `060-SOURCE-CLOSURE-AC-002` | Ambiguous equally specific source-authority row blocks the requested effect with `SOURCE_AUTHORITY_ROW_AMBIGUOUS`. |
| `060-SOURCE-CLOSURE-AC-003` | Dataset-default authority rows do not match a source-instance-specific request unless `instance_default_allowed = true` and no exact instance row exists. |
| `060-SOURCE-CLOSURE-AC-004` | Missing source-specific coverage row blocks negative, pass/fail, not-checked, not-applicable, and no-change output. |
| `060-SOURCE-CLOSURE-AC-005` | Missing staleness time input never falls back to current platform time. |
| `060-SOURCE-CLOSURE-AC-006` | OVAL, XCCDF, or similar `unknown`, `error`, `not evaluated`, `not checked`, and `not applicable` states never become pass, fail, absence, remediation, cleanup, retraction, graph expiry, or watermark by default. |
| `060-SOURCE-CLOSURE-AC-007` | DNS TTL expiry maps to stale or unknown, not domain deletion, host deletion, or negative DNS fact. |
| `060-SOURCE-CLOSURE-AC-008` | DHCP lease expiry maps to stale or expired assignment state, not host absence. |
| `060-SOURCE-CLOSURE-AC-009` | Missing flow evidence maps to `unknown` and emits no absence edge. |
| `060-SOURCE-CLOSURE-AC-010` | Directory permission gaps, hidden-membership gaps, AD primary-group gaps, direct-only membership queries, delta resets, and page incompletion block nonmembership. |
| `060-SOURCE-CLOSURE-AC-011` | Weak-signal combinations cannot authorize absence unless exactly one active policy row grants the exact combined signal set and requested effect. |
| `060-SOURCE-CLOSURE-AC-012` | All output-affecting closure refs appear in `VersionManifest`; omission fails with `VERSION_MANIFEST_INCOMPLETE` or a more specific owner code. |
| `060-SOURCE-CLOSURE-AC-013` | `SourceAuthorityClosureMatrix` fails closure when a row chain is absent, ambiguous, inactive, checksum-mismatched, out of scope, stale, or unvalidated. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `060-AC-001` | Missing, stale, partial, permission-limited, unsupported, or unavailable evidence cannot become absence unless an active profile authorizes it. |
| `060-AC-002` | Weak progress signals cannot be combined into stronger authority than each active policy row permits. |
| `060-AC-003` | Every coverage-dependent fact has a current coverage assertion or emits a deterministic unknown/error outcome. |
| `060-AC-004` | Source staleness and derived-view lag are exposed as distinct states. |
| `060-AC-005` | Every watermark attempt emits exactly one `WatermarkCommitRecord`. |
| `060-AC-006` | Gold derivation, cleanup, retraction, graph expiry, and watermark advancement persist completeness row refs and blocking reasons before producing absence-sensitive effects. |
| `060-AC-007` | External schema metadata cannot create authority, absence, assertion state, compliance state, risk state, cleanup, graph expiry, retraction, or watermark effects without exact `060` rows. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

TODO: Concrete active source-specific row instances for vendor-neutral `source_dataset` values remain outside this stable contract. Missing row instances are deterministic activation or validation blockers and must not be inferred from source category, vendor product, research report, or private inventory.
