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
- `TelemetryRuntimeState`
- `TelemetryNonAuthorityRule`

- `GoldFactSchema`
- `FactAbsenceOutcome`
- `EvidenceRef`
- `ActivationControlledArtifactRef`
- `GoldFactPredicateContractRow`
- `GoldFactSubjectRefKindRegistry`
- `GoldFactObjectValueKindRegistry`

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
- `SourceAuthorityClosureMatrixRowSet`
- `ExternalSchemaAuthoritySignalMappingRow`
- `ExternalSchemaAuthoritySignalMappingRowSet`

## Source Authority Contract

Gold derivation must use an active `SourceAuthorityProfileRow` for the exact fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, subject reference kind, object value kind, object kind, null-object state, structured object schema ref when present, staleness requirement, coverage requirement, and requested absence authority.

A broad source-category ranking must not act as fallback authority when the exact row is missing.

`DeriveAbsenceOrUnknown` may populate `040.GoldFact.absence_outcome` only through `040.GoldFactSchema` and only after exact authority, completeness, staleness, and coverage gates pass. Coverage-sensitive fact types or predicates must emit a `GoldFact` with non-empty `coverage_assertion_refs`; `coverage_assertion_refs = []` fails with `GOLD_FACT_COVERAGE_REQUIRED` for coverage-sensitive output.

### Correction authority handoff to 080

`080` must call `DeriveAbsenceOrUnknown` for any correction candidate based on missing evidence, stale evidence, source delete evidence, cleanup evidence, source-history no-change, control result absence, vulnerability fixed state, DNS absence, DHCP/IPAM expiry, flow non-observation, cloud disappearance, or CDC tombstone.

`060` decides whether negative, stale, cleanup, tombstone, or no-change evidence is authorized for a requested effect. `080` decides the correction operation. `090` decides projection and apply output. A correction or graph projection that consumes negative evidence directly, without an `AbsenceDerivationResult` ref for the requested effect, must fail before mutation.

### GraphEffectAuthorizationHandoff

`060` authorizes graph expiry and graph cleanup only. It does not emit graph edges, decide graph traversal, define graph direction, decide endpoint identity, or advance derived-view state.

Positive `observed_connection` graph output consumes an already authorized positive `GoldFact`; it is not an absence-sensitive effect and does not require `AbsenceDerivationResult` unless the projection also attempts expiry, cleanup, retraction, absence, or watermark behavior.

| Graph-adjacent input or requested effect | `060` authorization behavior | Required downstream behavior |
| --- | --- | --- |
| `positive_observed_connection_source_fact` | No new graph effect authorization. The positive fact must already have exact source authority for the fact itself. | `090` may project only through the active graph profile and qualifying flow-role plus identity handoffs. |
| `graph_expiry` | Requires `AbsenceDerivationResult.requested_effect = graph_expiry` and `allowed_effects` containing `graph_expiry`. | `090` may emit expiry only when the result ref is present and checksum-valid. |
| `cleanup` | Requires `AbsenceDerivationResult.requested_effect = cleanup` and `allowed_effects` containing `cleanup`. | `090` may emit cleanup only when the result ref is present and checksum-valid. |
| `missing_flow_role_evidence` | Maps to `unknown` or deterministic no-op. | Must not authorize absence edges, graph expiry, cleanup, retraction, or watermark advancement. |
| `ambiguous_flow_role_evidence` | Maps to `ambiguous`, `unknown`, or deterministic no-op according to the owner row. | Must not authorize graph mutation. |
| `flow_non_observation` | May be considered only through normal absence evaluation for a requested effect. | Without exact authorization, emits no absence edge, expiry, cleanup, retraction, or watermark. |

Graph derived-view lag, missing graph objects, graph apply success, graph index state, backend cleanup, destination cleanup, graph drift checks, and graph query results are progress or derived-view signals only. They must not authorize source absence, graph expiry, cleanup, retraction, or watermark advancement unless an active `060` row explicitly maps the exact signal and requested effect.

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
| `required_gold_fact_predicate_contract_ref` | Yes | none | Exact active `080.GoldFactPredicateContractRow` ref covering `fact_type` and `predicate`. |
| `subject_ref_kind_scope` | Yes | none | Non-empty subset of the selected `080.allowed_subject_ref_kinds`; it must not broaden the predicate contract. |
| `object_value_kind_scope` | Yes | none | Non-empty subset of the selected `080.allowed_object_value_kinds`; it must not broaden the predicate contract. |
| `null_object_authority` | No | `false` | `true` is valid only when `080.null_object_policy = allowed`; `false` does not authorize `null_value`. |
| `structured_object_authority_refs` | Required when structured values are authoritative | `[]` when structured values are not authoritative | Exact schema refs from `080.structured_value_schema_refs` that this authority row may authorize. |
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

### GoldFactShapeAuthorityHandoff

A `SourceAuthorityProfileRow` may authorize only a `GoldFact` candidate whose `fact_type`, `predicate`, `subject_ref.kind`, `object_value.kind`, `object_kind`, null-object state, structured schema ref, source dataset, subject scope, object scope, staleness state, coverage state, and requested effect all match both the active `080.GoldFactPredicateContractRow` and the selected source-authority row.

`subject_ref_kind_scope` and `object_value_kind_scope` must be subsets of the selected `080` predicate contract row. A `060` row must not broaden the `080` allowed kind set. `null_object_authority = true` is valid only when `080.null_object_policy = allowed`. A structured-object authority ref is valid only when the referenced schema ref appears in `080.structured_value_schema_refs` for the selected predicate.

External schema field presence, external schema field absence, OCSF object metadata, progress signals, freshness signals, and source-history metadata must not satisfy object authority without an exact `060` row and a matching `080` predicate contract row. Missing, inactive, checksum-mismatched, out-of-scope, or stale predicate contract refs block authority before gold output.

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

Runtime telemetry signals are progress or diagnostic signals only. Trace presence, trace absence, metric value, structured-log presence, structured-log absence, exporter success, exporter failure, Collector state, dashboard state, alert state, sampling decision, and dropped telemetry counts must not authorize source absence, source completeness, coverage, cleanup, retraction, graph expiry, watermark advancement, control pass/fail, or source-history no-change.

Telemetry may be consulted only through `140.TelemetryHealthMappingPolicy` for operational health or through an active `ProgressSignalInterpretationPolicy` row whose allowed effect is `diagnostic_only`. An active row must not grant `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark` effects from telemetry signals.

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
| Telemetry trace presence or absence | Runtime diagnostic only. |
| Telemetry metric value | Runtime diagnostic only. |
| Telemetry structured-log presence or absence | Runtime diagnostic only. |
| Telemetry exporter or Collector state | Operational health input only through `140` and `110`. |

Two or more progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, or live-probe signals that individually lack authority must not combine into stronger authority unless exactly one active `ProgressSignalInterpretationPolicy` row grants the exact combined signal set and requested effect.

## Coverage Contract

Coverage-sensitive facts require `CoverageDimensionProfile` and a current `CoverageAssertion` before absence, pass/fail/unknown, no-change, or negative claims may be emitted.

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. The coverage catalog in this document supplies default dimensions. Concrete active source-specific row instances are activation-controlled and validation-blocked until present; missing rows fail closed through `SourceAuthorityClosureMatrix` and `120` validation rows.

## Source authority artifact activation boundary

`060` algorithms are stable core contracts. Source-specific authority, staleness, coverage, completeness, progress, control-result, history, watermark, and absence rows are activation-controlled artifacts.

`ProgressSignalInterpretationPolicy` may include telemetry-derived signals only as `diagnostic_only`; any row that grants absence-sensitive or watermark effects from telemetry fails activation with `TELEMETRY_AUTHORITY_VIOLATION`.

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
| `SourceAuthorityClosureMatrixRowSet` | `activation_controlled_artifact` | Must be active and exact before an absence-sensitive effect can execute or be deterministically blocked. |
| `ExternalSchemaAuthoritySignalMappingRowSet` | `activation_controlled_artifact` | Required only when external schema signals are consulted by authority logic. |
| `ProjectionWatermarkPolicy` | `activation_controlled_artifact` | Must be active before watermark advancement. |
| `WatermarkCommitRecord` | `runtime_state_record` | Records attempted watermark outcome; does not grant authority. |
| `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` | `stable_core_contract` | Algorithms validate activation refs before output. |

### SourceAuthorityRowCatalogClosure

For any requested absence-sensitive effect, production evaluation must resolve this row chain in order:

1. `020.LakehouseFeedCategoryClosureRow`.
2. `060.SourceAuthorityClosureMatrixRow`.
3. `060.SourceAuthorityProfileRow`.
4. `060.LakehouseFeedCompletenessProfileRow`.
5. `060.CoverageDimensionProfile` when coverage-sensitive.
6. `060.SourceStalenessPolicy`.
7. `060.ProgressSignalInterpretationPolicy` when progress signals are consulted.
8. `060.SupplierCollectionVisibilityProfile` when visibility can affect the result.
9. `060.ControlResultMappingRow` when control-state output is requested.
10. `060.SourceHistoryRetentionProfile` when source-history no-change is requested.
11. `060.ExternalSchemaAuthoritySignalMappingRow` when external schema signals are consulted.
12. `060.AbsenceDerivationPolicy`.
13. `060.ProjectionWatermarkPolicy` when watermark output is requested.

The chain must resolve exactly one active row at each required step or exactly one deterministic block row. Missing, ambiguous, inactive, checksum-mismatched, out-of-scope, stale, unvalidated, or `TODO` rows block the requested effect and must emit no fact, cleanup, graph expiry, retraction, watermark, compliance pass/fail, or no-change proof.

### SourceAuthorityClosureMatrix

`SourceAuthorityClosureMatrix` is a stable validation view over active `060` activation-controlled row sets and imported `020` feed-category closure rows. It is not a runtime authority class and must not substitute for the underlying row refs.

For every active `LakehouseFeedProfile` and requested absence-sensitive effect, the matrix must prove that exactly one active validated row chain exists for fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, completeness, coverage, staleness, progress-signal interpretation, control-result mapping when applicable, source-history retention when applicable, absence derivation, and watermark policy when applicable.

If the row chain is absent, ambiguous, inactive, checksum-mismatched, out of scope, stale, or unvalidated, the effect must not execute. The result must be `unknown`, `not_applicable`, `source_stale`, `no_op`, or the most specific deterministic error. It must not emit absence, cleanup, retraction, graph expiry, pass/fail, no-change proof, or watermark advancement.

The matrix output must include the requested effect token, feed category, source dataset, selected row refs, row checksums, validation refs, blocking reason when blocked, and a `VersionManifest` ref. A closure validation result may be referenced by `030.VersionManifest`, but the underlying activation-controlled artifact refs and checksums remain required.

#### SourceAuthorityClosureMatrixRow schema

`SourceAuthorityClosureMatrixRow` is the executable row interface for proving that one active absence-sensitive effect has a complete authority chain or a deterministic block. Wildcards are forbidden for `fact_type` and `predicate`.

| Field | Required behavior |
| --- | --- |
| `row_set_id` | Stable row-set ID. |
| `row_version` | Immutable owner version. |
| `row_checksum` | SHA-256 over canonical row bytes after defaults materialize. |
| `row_id` | Stable row ID. |
| `feed_category` | Must match `020.LakehouseFeedCategoryClosureRow.feed_category`. |
| `required_lakehouse_feed_category_closure_row_ref` | Exact active `020.LakehouseFeedCategoryClosureRow` ref. |
| `source_dataset` | Vendor-neutral dataset token. |
| `requested_effect` | One of `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark`. |
| `fact_type` | Exact fact type. Wildcards are forbidden. |
| `predicate` | Exact predicate. Wildcards are forbidden. |
| `source_instance_policy` | One of `exact_instance_required`, `dataset_default_allowed`, or `blocked`. |
| `subject_scope_selector` | Canonical subject scope selector. |
| `object_scope_selector` | Canonical object scope selector. |
| `required_source_authority_row_refs` | Exact refs or deterministic block. |
| `required_completeness_profile_row_refs` | Exact refs or deterministic block. |
| `required_coverage_dimension_profile_refs` | Required when coverage-sensitive. |
| `required_staleness_policy_refs` | Required when staleness can affect output. |
| `required_progress_signal_policy_refs` | Required when progress signals are consulted. |
| `required_control_result_mapping_refs` | Required for control-result output. |
| `required_source_history_retention_refs` | Required for source-history no-change claims. |
| `required_external_schema_authority_signal_mapping_refs` | Required when external schema signals are consulted. |
| `required_absence_derivation_policy_refs` | Required for absence-sensitive outputs. |
| `required_watermark_policy_refs` | Required for watermark effects. |
| `closure_outcome` | One of `closed`, `deterministically_blocked`, or `blocked_validation`. |
| `deterministic_block_code` | Required when blocked. |
| `block_scope` | Required when `closure_outcome = deterministically_blocked`; closed values are `category`, `source_dataset`, `fact_predicate`, `scope`, and `requested_effect`. |
| `validation_refs` | Non-empty `120-SOURCE-CLOSURE-*` refs. |
| `lifecycle_status` | Production use requires `active`. |

When `closure_outcome = deterministically_blocked`, the row must include `deterministic_block_code`, `block_scope`, mutation-prohibition validation refs, and empty allowed effects for the blocked scope. Deterministic block rows emit no fact, cleanup, graph expiry, retraction, watermark, compliance pass/fail, or no-change proof.

#### ResolveSourceAuthorityClosureMatrixRow

```text
ResolveSourceAuthorityClosureMatrixRow(request, active_closure_row_set):
1. Validate activation ref and row-set checksum through `030.ActivationControlledArtifactRef`.
2. Select active rows in scope.
3. Match exact feed category, source dataset, fact type, predicate, requested effect, subject scope, and object scope.
4. Reject zero rows with `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE`.
5. Reject multiple equally specific rows with `SOURCE_AUTHORITY_CLOSURE_AMBIGUOUS`.
6. Reject any selected row with missing required artifact refs, inactive refs, checksum mismatch, stale validation refs, or non-active lifecycle state.
7. Return the deterministic block or the selected closure row.
8. Include the closure row ref in `VersionManifest` before the requested effect can execute.
```

Public closure rows may use only vendor-neutral categories, datasets, redacted refs, and canonical scope selectors. Concrete product names, tenant IDs, routes, credentials, scanner site names, host lists, and account inventories remain private binding artifacts and must not change row-resolution behavior.

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

### SourceAuthorityApiHandoff

`060` owns source authority, completeness, coverage, staleness, progress-signal, control-result, and absence decisions. `110` owns caller-visible labels, error rendering, redaction, and audit output. A `060` result consumed by API, compliance export, audit export, graph query, health, or analysis output must select exactly one handoff row.

`SourceAuthorityApiHandoffRow` fields are: owner result or error code, result state, blocking reason, requested effect, allowed effects, authorized-negative eligibility, required audit refs, caller diagnostics visibility, and recommended source label consumed by `110`.

| Owner result or blocking reason | Result state | Requested effect | Allowed effects | Authorized-negative eligibility | Required audit refs | Caller diagnostics visibility | Recommended `110` label |
| --- | --- | --- | --- | ---: | --- | --- | --- |
| missing authority | blocked | any absence-sensitive effect | `[]` | No | authority selector, requested effect, manifest refs | owner context when authorized | `unknown` |
| ambiguous authority | blocked | any absence-sensitive effect | `[]` | No | matching row refs when authorized | limited | `ambiguous` |
| missing coverage | blocked | coverage-sensitive output | `[]` | No | coverage domain, assertion refs | limited | `partial_unknown_gap` or `unknown` by owner row |
| source stale | blocked or stale output | any effect requiring fresh source | `[]` unless row explicitly permits stale display | No | staleness policy, selected time basis | visible label, redacted refs | `source_stale` |
| permission limited | blocked | absence, cleanup, retraction, graph expiry, watermark | `[]` | No | visibility profile refs | visible label, private details redacted | `permission_limited` |
| source unavailable | blocked | any source-dependent effect | `[]` | No | source/feed availability refs | visible label, private details redacted | `source_unavailable` |
| scope unavailable | blocked | scope-dependent effect | `[]` | No | scope selector class, redacted refs | visible label, private details redacted | `scope_unavailable` |
| partial known gap | blocked | absence-sensitive effect | `[]` | No | known gap refs | visible label when permitted | `partial_known_gap` |
| partial unknown gap | blocked | absence-sensitive effect | `[]` | No | unknown gap refs | visible label when permitted | `partial_unknown_gap` |
| not applicable | terminal non-negative | declared owner context | `[]` | No | applicability row refs | visible | `not_applicable` |
| not checked | terminal non-negative | control or evaluation output | `[]` | No | control-result row refs | visible | `not_checked` |
| ambiguous source-state mapping | blocked | any absence-sensitive effect | `[]` | No | source-state mapping refs | limited | `ambiguous` |
| weak progress signal with no authority | blocked | any effect | `[]` | No | signal refs, requested effect | diagnostic only unless authorized | `unknown` |
| watermark blocked | blocked | `watermark` | `[]` | No | watermark target, completeness and authority refs | health diagnostic only | `error` or `unknown` by health row |
| authorized absence or not observed | terminal authorized | `absence` or predicate-specific negative output | exact row allowed effects | Yes only when `AbsenceDerivationResult.absence_authorized = true` | authority, completeness, coverage, staleness, effect refs | visible label; refs redacted by policy | `authorized_not_observed` |

Every `060` result or blocking reason consumed by `110` must carry structured state and blocking reason refs. API and export layers must consume these refs and must not recompute source authority from missing rows, source staleness, progress signals, or completeness receipts.

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

### SourceAuthorityErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `SOURCE_AUTHORITY_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-row-missing` |
| `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-row-ambiguous` |
| `SOURCE_STALENESS_POLICY_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-staleness-policy-row-missing` |
| `SOURCE_STALENESS_POLICY_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-staleness-policy-row-ambiguous` |
| `COVERAGE_DIMENSION_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-dimension-row-missing` |
| `COVERAGE_DIMENSION_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-dimension-row-ambiguous` |
| `CONTROL_RESULT_MAPPING_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-control-result-mapping-row-missing` |
| `CONTROL_RESULT_MAPPING_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-control-result-mapping-row-ambiguous` |
| `PROGRESS_SIGNAL_POLICY_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-progress-signal-policy-row-missing` |
| `PROGRESS_SIGNAL_POLICY_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-progress-signal-policy-row-ambiguous` |
| `SOURCE_HISTORY_RETENTION_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-history-retention-row-missing` |
| `SOURCE_HISTORY_RETENTION_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-history-retention-row-ambiguous` |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-supplier-visibility-profile-row-missing` |
| `SUPPLIER_VISIBILITY_PROFILE_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-supplier-visibility-profile-row-ambiguous` |
| `SOURCE_STATE_MAPPING_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-state-mapping-row-missing` |
| `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-closure-incomplete` |
| `SOURCE_AUTHORITY_CLOSURE_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-closure-ambiguous` |
| `COVERAGE_ASSERTION_REQUIRED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-assertion-required` |
| `COVERAGE_DIMENSION_UNRESOLVED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-dimension-unresolved` |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-weak-progress-signal-no-authority` |
| `COMPLETENESS_DECISION_UNSAFE_FOR_ABSENCE` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-completeness-decision-unsafe-for-absence` |
| `SOURCE_AUTHORITY_ARTIFACT_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-artifact-missing` |
| `SOURCE_AUTHORITY_ARTIFACT_INACTIVE` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-artifact-inactive` |
| `SOURCE_AUTHORITY_ARTIFACT_CHECKSUM_MISMATCH` | `060` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-artifact-checksum-mismatch` |
| `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-row-scope-mismatch` |
| `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-lakehouse-feed-completeness-profile-row-missing` |
| `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | `060` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-upstream-completeness-evidence-insufficient` |
| `COMPLETENESS_EFFECT_NOT_ALLOWED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-completeness-effect-not-allowed` |
| `COMPLETENESS_BLOCKING_PRECEDENCE_APPLIED` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-completeness-blocking-precedence-applied` |
| `EMPTY_SCOPE_NOT_AUTHORIZED_FOR_ABSENCE` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-empty-scope-not-authorized-for-absence` |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-watermark-advancement-completeness-blocked` |
| `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-cdc-tombstone-retraction-unauthorized` |
| `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `060` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `060.SourceAuthorityErrorContext` | `error-registry-060-external-schema-authority-forbidden` |
| `SOURCE_AUTHORITY_CLOSURE_BLOCKED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-authority-closure-blocked` |
| `SOURCE_DATASET_CATALOG_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-dataset-catalog-row-missing` |
| `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-dataset-catalog-row-ambiguous` |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-external-schema-authority-signal-row-missing` |
| `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-external-schema-authority-signal-row-ambiguous` |
| `SOURCE_HISTORY_OUTSIDE_WINDOW_NO_PROOF` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-history-outside-window-no-proof` |
| `SOURCE_HISTORY_COVERAGE_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-source-history-coverage-row-missing` |
| `DETERMINISTIC_BLOCK_ROW_SELECTED` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-deterministic-block-row-selected` |
| `GOLD_FACT_COVERAGE_REQUIRED` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-gold-fact-coverage-required` |
| `GOLD_FACT_AUTHORITY_ROW_MISSING` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-gold-fact-authority-row-missing` |

## Watermarks

Every attempted production source, projection, graph-apply, or presence-only watermark change must be evaluated through `ProjectionWatermarkPolicy` and persisted as exactly one `WatermarkCommitRecord`. Failed, partial, aborted, schema-preflight-failed, or completeness-blocked runs must not advance watermarks.

### SourceAuthorityErrorContext

`SourceAuthorityErrorContext` is the owner context schema for `060` authority, completeness, coverage, absence, source-history, control-result, progress-signal, and watermark registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `060` context schema version. |
| `owner_spec` | Yes | Must be `060`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `authority`, `completeness`, `coverage`, `staleness`, `control_result`, `progress_signal`, `source_history`, `visibility`, `absence`, `watermark`, `external_schema_non_authority`, or `cdc_tombstone`. |
| `operation` | Yes | Authority resolution, completeness evaluation, absence derivation, coverage validation, source staleness evaluation, control result mapping, watermark decision, or correction authority handoff. |
| `affected_record_type` | Yes | Authority row, completeness profile row, coverage assertion, staleness policy, progress policy, control mapping, absence result, watermark record, or gold fact candidate. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `fact_type` | No | Required when a fact or absence candidate was evaluated. |
| `predicate` | No | Required when a fact or absence candidate was evaluated. |
| `source_dataset` | No | Vendor-neutral dataset token only. |
| `subject_scope_checksum` | No | Checksum only; raw scope values remain redacted when private. |
| `object_scope_checksum` | No | Checksum only; raw scope values remain redacted when private. |
| `requested_effect` | No | One of `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark` when an effect was requested. |
| `blocking_reason` | Yes | Bounded reason for blocked, diagnostic, ambiguous, or no-mutation output. |
| `validation_refs` | Yes | Exact `120` source-authority fixture refs. |
| `redaction_classes` | Yes | External schema raw values, private bindings, credentials, source-native IDs, raw payload bytes, and supplier-private metadata must map to `always_forbidden`. |

## Authority-to-GoldFact linkage

`060` owns absence and coverage authorization causes but does not restate `040.GoldFactSchema`. The following cause codes may be recorded on a 040-shaped `GoldFact` validation or derivation failure.

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `GOLD_FACT_COVERAGE_REQUIRED` | `060` | A coverage-sensitive fact or predicate attempts output with `coverage_assertion_refs = []`. |
| `GOLD_FACT_AUTHORITY_ROW_MISSING` | `060` | No exact active `SourceAuthorityProfileRow` covers the requested fact, predicate, dataset, scopes, staleness, coverage, and absence authority. |

Missing rows, stale states, partial states, permission-limited states, weak progress signals, destination cleanup, omitted fields, missing lakehouse rows, and OCSF field absence must remain unable to create absence by themselves.

A source-authority row that references a predicate contract must not authorize a `GoldFact` candidate whose `subject_ref.kind`, `object_value.kind`, `object_kind`, null-object state, or structured schema ref is outside that predicate contract. Such a candidate must fail before `gold_fact_key_id` computation with the most specific `080` predicate-contract error or `040.CORE_ONE_OF_INVALID` when the one-of shape is invalid.

### External schema non-authority boundary

OCSF class, category, activity, type, object paths, observables, enrichments, status, severity, confidence, endpoint order, field presence, and field absence are normalized observation metadata only. They must not satisfy `SourceAuthorityProfileRow`, `CoverageAssertion`, `SourceStalenessPolicy`, `LakehouseFeedCompletenessProfileRow`, `AbsenceDerivationPolicy`, or `ControlResultMappingRow` requirements unless an active `060`-owned row explicitly maps the exact signal to the exact effect.

### ExternalSchemaAuthoritySignalMappingRow schema

`ExternalSchemaAuthoritySignalMappingRow` is the only `060` row that can make a normalized external-schema field visible to source-authority logic. Wildcards are forbidden.

| Field | Required rule |
| --- | --- |
| `row_id` | Stable row ID. |
| `external_schema_profile_ref` | Exact active `050.ExternalSchemaProfile`. |
| `field_path` | Exact normalized field path. Wildcards are forbidden. |
| `source_dataset` | Exact vendor-neutral dataset token. |
| `fact_type` | Exact fact type. |
| `predicate` | Exact predicate. |
| `requested_effect` | Exact effect token. |
| `source_authority_closure_matrix_row_ref` | Exact active closure row or deterministic block row. |
| `source_authority_row_ref` | Exact active authority row. |
| `coverage_dimension_profile_ref` | Required when coverage-sensitive. |
| `source_staleness_policy_ref` | Exact active staleness policy. |
| `control_result_mapping_ref` | Required for control-state output. |
| `absence_derivation_policy_ref` | Required for absence-sensitive output. |
| `projection_watermark_policy_ref` | Required for watermark output. |
| `validation_refs` | Non-empty. |
| `activation_scope` | Vendor-neutral scope. |
| `lifecycle_status` | Production use requires `active`. |

Missing, ambiguous, inactive, checksum-mismatched, out-of-scope, or unvalidated external-schema authority signal rows leave the external schema signal non-authoritative and block the attempted authority effect.

A `060` row that maps an external-schema signal must name the external schema profile ref, field path, source dataset, fact type, predicate, requested effect, source authority row ref, coverage row ref when applicable, staleness row ref, validation refs, activation scope, and lifecycle status. Omission of any required ref means the external-schema signal remains non-authoritative.

Vulnerability Finding status must not become Cadastre `assertion_state` without a `060` or `080` derivation path. DNS and DHCP field absence must not become absence without exact completeness, coverage, staleness, and authority rows. OCSF endpoint order must not become graph direction authority.

#### ExternalSchemaAuthoritySignalMappingRowSet default

- The default active row set is empty.
- Empty row set means every external schema signal remains non-authoritative.
- Missing row set, inactive row set, checksum mismatch, or out-of-scope row set blocks any attempted authority effect.
- Wildcard field paths, wildcard source datasets, wildcard fact types, wildcard predicates, and wildcard requested effects are forbidden.

Deterministic error handling for external-schema authority signal resolution is:

| Condition | Required result |
| --- | --- |
| No exact row | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` or imported `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN`; no authority effect. |
| More than one equally specific row | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS`; no authority effect. |
| Row missing closure refs | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN`; no authority effect. |
| Row exact and validated | Signal may be consulted only for the named fact type, predicate, source dataset, field path, and requested effect. |

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
| telemetry trace presence | runtime diagnostic only | No |
| telemetry trace absence | runtime diagnostic only | No |
| telemetry metric value | runtime diagnostic only | No |
| telemetry structured-log presence | runtime diagnostic only | No |
| telemetry structured-log absence | runtime diagnostic only | No |
| telemetry exporter success | delivery diagnostic only | No |
| telemetry exporter failure | operational health input only through `140`/`110` | No |
| telemetry Collector state | operational health input only through `140`/`110` | No |
| telemetry sampling decision | runtime diagnostic only | No |
| telemetry dropped-signal count | operational health input only through `140`/`110` | No |

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

Source-history no-change proof requires both `SourceHistoryRetentionProfile` and `CoverageDimensionProfile` for source-history coverage unless an accepted owner patch removes source history from the coverage domain list. Outside-window no-result maps to `unknown` or deterministic no-op, never no-change proof.

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
| `060-API-HANDOFF-AC-001` | Every `060` result or blocking reason consumed by `110` has exactly one `SourceAuthorityApiHandoff` row and does not authorize negative output unless `AbsenceDerivationResult.absence_authorized = true`. |
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
| `060-TELEMETRY-PROGRESS-NONAUTH-AC-001` | Telemetry traces, metrics, structured logs, exporter state, Collector state, sampling decisions, and dropped-signal counts cannot authorize absence, cleanup, retraction, graph expiry, source completeness, coverage, control pass/fail, source-history no-change, or watermark advancement. |
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
| `060-GRAPH-EFFECT-AUTH-AC-001` | Graph expiry requires `AbsenceDerivationResult.requested_effect = graph_expiry` and no graph-derived signal can authorize it by itself. |
| `060-GRAPH-EFFECT-AUTH-AC-002` | Graph cleanup requires `AbsenceDerivationResult.requested_effect = cleanup` and destination or backend cleanup summaries do not authorize it. |
| `060-GRAPH-EFFECT-AUTH-AC-003` | Missing or ambiguous flow-role evidence maps to unknown, ambiguous, or deterministic no-op and emits no graph mutation. |
| `060-GRAPH-EFFECT-AUTH-AC-004` | Flow non-observation emits no absence edge, expiry, cleanup, retraction, or watermark without exact requested-effect authorization. |
| `060-SOURCE-CLOSURE-AC-014` | Every active absence-sensitive effect resolves one `SourceAuthorityClosureMatrixRow` with complete refs and passing validation, or resolves one deterministic block row. Missing, ambiguous, inactive, checksum-mismatched, or `TODO` rows fail before any absence-sensitive effect. |
| `060-SOURCE-CLOSURE-AC-015` | Every active absence-sensitive effect resolves exactly one active closure row chain or one deterministic block row before `DeriveAbsenceOrUnknown` can emit output. |
| `060-SOURCE-CLOSURE-AC-016` | Deterministic block rows emit no fact, cleanup, graph expiry, retraction, watermark, compliance pass/fail, or no-change proof. |
| `060-SOURCE-CLOSURE-AC-017` | Source-history no-change proof fails without both retention and coverage rows. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-001` | External schema signals remain non-authoritative without an exact active `ExternalSchemaAuthoritySignalMappingRow`. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-002` | Missing, inactive, checksum-mismatched, or out-of-scope external-schema authority signal row sets block authority effects and do not reinterpret normalized metadata as source authority. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-003` | Ambiguous external-schema authority signal rows emit `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` and no authority, absence, cleanup, graph expiry, control-state, or watermark effect. |
| `060-ERROR-REGISTRY-CLOSURE-AC-001` | All `060` source-closure owner errors generate complete `110.ErrorCodeRegistryRow` rows with no `TODO` values. |
| `060-GOLD-SHAPE-AUTHORITY-AC-001` | A source-authority row cannot widen `080.allowed_subject_ref_kinds` or `080.allowed_object_value_kinds`; widening attempts fail before authority selection. |
| `060-GOLD-SHAPE-AUTHORITY-AC-002` | A source-authority row cannot permit `null_value` when the selected predicate contract forbids it. |
| `060-GOLD-SHAPE-AUTHORITY-AC-003` | External schema field presence or absence remains non-authoritative without exact `050`, `060`, and `080` handoff rows. |
| `060-GOLD-SHAPE-AUTHORITY-AC-004` | Missing, inactive, checksum-mismatched, or out-of-scope predicate contract refs block source authority before gold output. |
| `060-GOLD-SHAPE-AUTHORITY-REPLAY-AC-001` | Authority checksum changes and predicate-contract checksum changes are replay-affecting for gold output. |

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

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `060-TODO-SOURCE-DATASET-ROW-CATALOG` | TODO: Provide active vendor-neutral `source_dataset` row catalogs or deterministic block rows for every `source_dataset` referenced by active feed profiles. | Source-authority closure and absence-sensitive effects. | Product governance plus `020`, `060`, and `120`. | Requested effects remain blocked. |
