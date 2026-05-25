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
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ResolveScopedRow`
- `030.CompareScopeSpecificity`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.SourceDatasetCatalogRow`
- `020.SourceDatasetCatalogRowSet`
- `020.ResolveSourceDatasetCatalogRow`

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
- `CoverageDomainToken`
- `CoverageDomainCatalog`
- `CoverageDomainAliasRejectionTable`
- `ValidateCoverageDomainToken`
- `ValidateCoverageDomainTokenArray`
- `CoverageDomainErrorCodeSet`
- `LakehouseFeedCompletenessProfile`
- `LakehouseFeedCompletenessProfileRow`
- `ProgressSignalInterpretationPolicy`
- `ProjectionWatermarkPolicy`
- `WatermarkCommitRecord`
- `EvaluateLakehouseFeedCompleteness`
- `SourceAuthorityArtifactLifecycleGuardRows`
- `SourceAuthorityClosureMatrix`
- `SourceAuthorityClosureMatrixRowSet`
- `MVPGoldPredicateSourceAuthorityClosure`
- `ExternalSchemaAuthoritySignalMappingRow`
- `ExternalSchemaAuthoritySignalMappingRowSet`
- `MVPSourceAuthorityClosureTupleInventory`

## Source Authority Contract

Gold derivation must use an active `SourceAuthorityProfileRow` for the exact fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, subject reference kind, object value kind, object kind, null-object state, structured object schema ref when present, staleness requirement, coverage requirement, and requested absence authority.

A broad source-category ranking must not act as fallback authority when the exact row is missing.

`DeriveAbsenceOrUnknown` may populate `040.GoldFact.absence_outcome` only through `040.GoldFactSchema` and only after exact authority, completeness, staleness, and coverage gates pass. Coverage-sensitive fact types or predicates must emit a `GoldFact` with non-empty `coverage_assertion_refs`; `coverage_assertion_refs = []` fails with `GOLD_FACT_COVERAGE_REQUIRED` for coverage-sensitive output.

### SourceDatasetCatalogAuthorityGate

Every source-authority row family that uses `source_dataset` must carry the selected source-dataset catalog row as a structured ref and checksum. The selected row must be resolved through `020.ResolveSourceDatasetCatalogRow` before source-authority profile selection, completeness evaluation, coverage evaluation, staleness evaluation, progress-signal interpretation, control-result mapping, source-history interpretation, absence derivation, external-schema authority mapping, watermark policy selection, or closure matrix evaluation.

| Row family | Required source-dataset fields | Default when missing or invalid |
| --- | --- | --- |
| `SourceAuthorityProfileRow` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block requested effect before authority selection. |
| `LakehouseFeedCompletenessProfileRow` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block absence-sensitive completeness decision. |
| `CoverageDimensionProfile` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block coverage-sensitive output. |
| `SourceStalenessPolicy` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block staleness-dependent output; no current-time fallback. |
| `ProgressSignalInterpretationPolicy` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Treat signal as diagnostic-only. |
| `ControlResultMappingRow` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block control pass/fail/not-checked output. |
| `SupplierCollectionVisibilityProfile` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block permission-sensitive negative output. |
| `SourceHistoryRetentionProfile` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Block no-change proof and outside-window interpretation. |
| `AbsenceDerivationPolicy` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Emit `unknown`, `no_op`, or owner error; no authorized absence. |
| `ProjectionWatermarkPolicy` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Emit watermark no-op; no advancement. |
| `ExternalSchemaAuthoritySignalMappingRow` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Keep normalized external-schema signal non-authoritative. |
| `SourceAuthorityClosureMatrixRow` | `source_dataset_catalog_row_ref`, `source_dataset_catalog_row_checksum` | Closure outcome maps to `000.blocked_missing_ref`, `blocked_checksum`, `blocked_todo`, `blocked_package_set`, or `blocked_manifest` as applicable. |

A missing, inactive, ambiguous, private-leaking, checksum-mismatched, package-set-mismatched, out-of-scope, unvalidated, unmanifested, `TODO:`-bearing, or deterministically blocked source-dataset catalog row must emit no fact, cleanup, retraction, graph expiry, watermark, compliance pass/fail, control state, no-change proof, graph delta, or API authorized-negative output.

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
| `subject_scope_selector` | Yes | none | `030.ScopeSelector` covering the subject scope under `060.SourceAuthorityScopeSelectorContextSet`. |
| `object_scope_selector` | Yes | none | `030.ScopeSelector` covering the object scope under `060.SourceAuthorityScopeSelectorContextSet`. |
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
| `activation_scope` | Yes | none | `030.ActivationScope`; selected through `060.SourceAuthorityScopeSelectorContextSet` before the row may affect output. |
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

### StructuredGoldObjectAuthorityClosure

Structured values are authorized only by exact `080.StructuredValueSchemaRow` refs, exact predicate refs, exact source-dataset refs, and exact source-authority closure rows. A parseable structured value is not sufficient authority.

| Structured schema | Required `060` closure |
| --- | --- |
| `080.struct.os_descriptor.v1` | Exact positive authority and staleness policy for OS descriptor output. |
| `080.struct.host_lifecycle_state.v1` | Exact lifecycle-state authority; source delete and decommission states require absence/correction refs before any retraction, cleanup, graph expiry, or lifecycle boundary output. |
| `080.struct.host_management_state.v1` | Exact management-plane authority; labels alone are non-authoritative. |
| `080.struct.vulnerability_instance_key.v1` | Exact scanner authority, coverage, staleness, and absence/fixed-state closure. |
| `080.struct.software_instance_key.v1` | Exact inventory authority and staleness policy. |
| `080.struct.control_evaluation_key.v1` | Exact `ControlResultMappingRow` and coverage closure. |
| `080.struct.observed_exposure_key.v1` | Exact positive observed-exposure authority; no reachability, service-access, or identity-conditioned access negative inference. |

`SourceAuthorityProfileRow.structured_object_authority_refs` must contain exact structured schema refs from `080`; schema-family strings, external schema object names, OCSF class names, package labels, and private binding names are not substitutes. Progress signals remain diagnostic-only by default. `ControlResultMappingRow` must map every external control state to exactly one of pass, fail, unknown, not_checked, not_applicable, error, or deterministic block; no default pass or fail mapping is permitted.

### SourceAuthorityScopeSelectorContextSet

`SourceAuthorityScopeSelectorContextSet` is the owner context family for scoped rows in `060`. Each context instantiates `030.ScopeSelectorContext`; no row in this section defines selector schema, equality, coverage, subset matching, specificity, or ambiguity behavior.

| Context name | Row families covered | Required dimensions | Optional dimensions | Subset-allowed dimensions | Owner-local predicate that remains outside selector matching |
| --- | --- | --- | --- | --- | --- |
| `source_authority_activation_scope` | `SourceAuthorityProfileRow` activation | `source_category`, `source_dataset`, `fact_type`, `predicate`, `requested_effect` | `source_instance`, `subject_scope`, `object_scope` | `source_instance` only when owner row also has `instance_default_allowed = true` | fact type, predicate, object kind, subject ref kind, object value kind, authority mode, lifecycle status. |
| `subject_scope` | `SourceAuthorityProfileRow.subject_scope_selector` | owner-declared subject keys | owner-declared subject refinements | none unless exact context row permits | subject reference kind validation. |
| `object_scope` | `SourceAuthorityProfileRow.object_scope_selector` | owner-declared object keys | owner-declared object refinements | none unless exact context row permits | object value kind, null-object, structured-object validation. |
| `coverage_scope` | `CoverageDimensionProfile` | `coverage_domain`, `source_category`, `source_dataset`, `fact_type`, `predicate` | owner-declared coverage dimensions | none | coverage state mapping and freshness policy. |
| `staleness_scope` | `SourceStalenessPolicy` | `source_category`, `source_dataset`, `fact_type`, `predicate` | owner-declared time-basis keys | none | time-input precedence and expiry effects. |
| `visibility_scope` | `SupplierCollectionVisibilityProfile` | `source_category`, `source_dataset`, `collection_method` | owner-declared permission keys | none | permission-state interpretation. |
| `progress_signal_scope` | `ProgressSignalInterpretationPolicy` | `source_category`, `source_dataset`, `signal_kind` | owner-declared signal keys | none | allowed effect and non-authority mapping. |
| `source_history_scope` | `SourceHistoryRetentionProfile` | `source_category`, `source_dataset`, `history_kind` | owner-declared retention keys | none | source-native history-window handling. |
| `control_result_scope` | `ControlResultMappingRow` | `source_category`, `source_dataset`, `control_vocab` | benchmark/check scope | none | external vocabulary result mapping. |
| `external_schema_authority_scope` | `ExternalSchemaAuthoritySignalMappingRow` | `external_schema_profile_ref`, `field_path`, `source_dataset`, `fact_type`, `predicate`, `requested_effect` | owner-declared schema refinements | none | external-schema authority handoff refs. |
| `watermark_scope` | `ProjectionWatermarkPolicy` | `source_category`, `source_dataset`, `requested_effect` | owner-declared watermark keys | none | watermark advancement rule. |

Dataset-default behavior is not a wildcard. A dataset-default source-authority row may cover a source-instance request only when the selected context lists `source_instance` in `subset_allowed_dimension_keys` and the candidate row has `instance_default_allowed = true`. Otherwise omission of `source_instance` returns the owner-mapped `SCOPE_SUBSET_NOT_ALLOWED` error before authority effects.

Owner selector errors map as follows unless a narrower owner row maps the error more specifically.

| Shared selector error | Default `060` owner error |
| --- | --- |
| `SCOPE_SELECTOR_MISMATCH` | `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` |
| `SCOPE_REQUEST_UNDER_SCOPED` | `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` |
| `SCOPE_SUBSET_NOT_ALLOWED` | `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` |
| `SCOPE_SELECTOR_UNSUPPORTED_DIMENSION` | `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` |
| `SCOPE_SELECTOR_AMBIGUOUS` | `SOURCE_AUTHORITY_ROW_AMBIGUOUS` |
| selector structural validation errors | most specific `060` owner validation error, or `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` when no narrower code exists |

### ResolveSourceAuthorityProfileRow

```text
ResolveSourceAuthorityProfileRow(request, active_row_set):
1. Validate row-set ref through `030.ActivationControlledArtifactRef`.
2. Apply exact non-scope predicates for fact_type, predicate, source_category, source_dataset, requested effect, subject reference kind, object value kind, object kind, null-object state, structured schema refs, authority mode, lifecycle status, and validation refs.
3. Materialize request activation, subject, and object selectors under `060.SourceAuthorityScopeSelectorContextSet`.
4. Select candidate rows by calling `030.ResolveScopedRow` for `activation_scope`, `subject_scope_selector`, and `object_scope_selector` using the selected contexts and owner error map.
5. If request names a source instance, exact source-instance rows are non-scope preferred. Dataset-default rows may cover only when `instance_default_allowed = true` and the context permits `source_instance` subset matching.
6. If zero rows remain after selector resolution, return `SOURCE_AUTHORITY_ROW_MISSING` or `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` according to the mapped selector failure.
7. If more than one maximal row remains under `030.CompareScopeSpecificity`, return `SOURCE_AUTHORITY_ROW_AMBIGUOUS`. No lexical, file-order, package-order, or validation-order tie break is permitted.
8. Return the selected row and require its ref, row checksum, selector context refs, selector checksums, and activation artifact ref in `VersionManifest`.
```

### StructuredInputRepositorySourceAuthorityHandoff

Repository-authored source-authority, completeness, coverage, staleness, progress-signal, source-history, control-result, absence-policy, and watermark row catalogs are inert until materialized into activation-controlled row sets, validated for the exact snapshot, included in the owner `VersionManifest`, and activated through package-set membership when package-supplied.

`StructuredInputRepositorySnapshot` may be cited as provenance, but it must not satisfy `SourceAuthorityClosureMatrix`, `SourceAuthorityProfileRowSet`, `LakehouseFeedCompletenessProfileRowSet`, `CoverageDimensionProfileRowSet`, `SourceStalenessPolicyRowSet`, `ProgressSignalInterpretationPolicyRowSet`, `ControlResultMappingRowSet`, `SourceHistoryRetentionProfileRowSet`, `AbsenceDerivationPolicyRowSet`, or `ProjectionWatermarkPolicyRowSet`.

Remote-maintained source-authority artifacts require the repository-authored source-authority closure input set below before any absence, cleanup, retraction, graph-expiry, watermark, pass/fail, no-change proof, or authorized-negative API output can occur.

| Closure input | Required when | Default when missing |
| --- | --- | --- |
| repository profile ref | repository-authored | no source-authority effect |
| exact snapshot ref | repository-authored | no source-authority effect |
| file manifest checksum | repository-authored | no source-authority effect |
| repository template contract ref | profile declares `template_required = true` | no source-authority effect |
| producer CI validation ref | producer CI evidence is supplied or accepted | no source-authority effect when stale or mismatched |
| materialization result ref | repository output is materialized | no source-authority effect |
| publication manifest ref | remote publication is consumed | no source-authority effect |
| package release ref | package release is created | no source-authority effect |
| package-set ref | artifact is package-supplied | no source-authority effect |
| candidate sync record ref | sync imported the candidate | discovery/audit only; no effect |
| selected row-set refs and row checksums | every source-authority row family | no source-authority effect |
| repository group ref | cross-repository artifacts are declared | no effect until group coherence passes |
| `VersionManifest` refs | every output-affecting ref | no source-authority effect |

Template conformance maps to validation evidence only. Stale producer CI maps to no source-authority effect. Sync records map to candidate discovery only. Cross-repository source-authority artifacts require `030.StructuredInputRepositoryGroup` coherence before source-authority closure.

Closure for repository-authored row catalogs requires active row-set refs, exact row-set checksums, lifecycle status permitted for production, validation refs for the exact repository snapshot and row bytes, materialization refs when packaged, package-set refs when package-supplied, and `030.VersionManifest` inclusion.

Public repository-authored row catalogs must satisfy `010` public/private rules. Concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventories, zone inventories, account lists, source-native secrets, raw private fixture bytes, and environment-specific source target lists fail with `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` or `PRIVATE_BINDING_LEAK` before publication, validation-report materialization, API response, export, audit output, or telemetry export.

A repository merge, pull request approval, branch update, validation run, or hook success must not authorize absence, cleanup, retraction, graph expiry, or watermark advancement.

| Error code | Required use |
| --- | --- |
| `SOURCE_AUTHORITY_REPOSITORY_ROWSET_INACTIVE` | Repository-authored row catalog exists but is not an active materialized row set with required lifecycle, validation, materialization, package, and manifest refs. |
| `SOURCE_AUTHORITY_REPOSITORY_SNAPSHOT_ONLY_FORBIDDEN` | A `StructuredInputRepositorySnapshot` is used as source-authority closure, completeness, coverage, staleness, absence, cleanup, graph-expiry, retraction, or watermark authority. |
| `SOURCE_AUTHORITY_REPOSITORY_TEMPLATE_MISMATCH` | Repository-authored source-authority catalog does not match required template contract. |
| `SOURCE_AUTHORITY_REPOSITORY_CI_STALE` | Producer CI evidence is stale or not bound to exact snapshot, selected paths, file manifest checksum, toolchain refs, and validation refs. |
| `SOURCE_AUTHORITY_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest does not match materialized source-authority artifacts, package type, validation refs, or release refs. |
| `SOURCE_AUTHORITY_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used as source-authority, completeness, coverage, absence, cleanup, graph-expiry, retraction, or watermark authority. |
| `SOURCE_AUTHORITY_REPOSITORY_GROUP_MISMATCH` | Cross-repository source-authority artifacts are not coherent under the selected repository group. |
| `SOURCE_AUTHORITY_REPOSITORY_PRIVATE_BINDING_LEAK` | Repository-authored authority catalog or validation output exposes private binding data or raw private fixture bytes. |

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

### LakehouseTableStateSignalNonAuthorityHandoff

Lakehouse table-state and maintenance success signals are operational evidence only unless exact active `060` rows grant a narrower effect.

| Signal | Required non-authority behavior |
| --- | --- |
| `RawFeedManifest.manifest_validation_result = valid` | Manifest-shape evidence only; it is not source completeness. |
| `LakehouseReadCompletenessReceipt.receipt_state = read_complete` | Necessary but not sufficient for absence-sensitive effects. |
| `LakehouseCommitRef.status = success` | Table-write evidence only. |
| `ReplayRetentionDecision(decision = eligible)` | Permits maintenance preflight only; it does not authorize absence, cleanup, graph expiry, retraction, or watermark advancement. |
| `TableMaintenancePolicy` success | Operational table-state evidence only. |
| `CatalogBranchPromotionPolicy` success | Visibility transition evidence only through `020` and `030` refs; it does not create source authority. |

`020` maintenance and catalog promotion must not depend on any `060` row family that remains `TODO:` for absence-sensitive effects. Missing exact active `060` authority, completeness, coverage, staleness, absence, progress-signal, visibility, control-result, history, or watermark rows keep table-state signals at `diagnostic_only`.

## Progress Signal Interpretation

Progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, live-probe, and run-lock signals are non-authoritative by default. They may affect output only through an active `ProgressSignalInterpretationPolicy` row.

Runtime telemetry signals are progress or diagnostic signals only. Trace presence, trace absence, metric value, structured-log presence, structured-log absence, exporter success, exporter failure, Collector state, dashboard state, alert state, sampling decision, and dropped telemetry counts must not authorize source absence, source completeness, coverage, cleanup, retraction, graph expiry, watermark advancement, control pass/fail, or source-history no-change.

Telemetry may be consulted only through `140.TelemetryHealthMappingPolicy` for operational health or through an active `ProgressSignalInterpretationPolicy` row whose allowed effect is `diagnostic_only`. An active row must not grant `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark` effects from telemetry signals.

Run-lock signals owned by `030` must not satisfy `SourceAuthorityProfileRow`, `LakehouseFeedCompletenessProfileRow`, `CoverageAssertion`, `SourceStalenessPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy`, or `ControlResultMappingRow`. A run-lock signal used without a specific non-authoritative diagnostic policy emits `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY`; it must not produce absence, cleanup, retraction, graph expiry, source watermark, projection watermark, control pass/fail, source-history no-change, or source completeness.

### RunLockSignalNonAuthorityValidationHandoff

`val-060-runlock-signal-nonauthority` must cover heartbeat, lease expiry, stale recovery, conflict, release, lock loss, heartbeat uncertainty, and stale fencing. For each signal, the expected effect must be diagnostic-only or `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY`.

The required mutation-prohibition proof must enumerate all forbidden effects below:

| Forbidden effect class | Required proof |
| --- | --- |
| source absence | No `FactAbsenceOutcome.authorized_absent` or `authorized_not_observed` may be emitted from a run-lock signal. |
| cleanup | No cleanup permission may be emitted. |
| retraction | No fact retraction or correction may be emitted. |
| graph expiry | No `graph_expiry` effect may be authorized. |
| source completeness | No completeness decision may be satisfied. |
| coverage assertion | No `CoverageAssertion` may be created or satisfied. |
| control output | No control pass, fail, not-checked, or not-applicable output may be authorized. |
| source-history no-change proof | No no-change proof may be emitted. |
| watermark advancement | No source, projection, graph-apply, or presence-only watermark may advance. |

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
| Run-lock heartbeat | Orchestration ownership diagnostic only. |
| Run-lock lease expiry | Orchestration stale-lock candidate only. |
| Run-lock stale recovery | Orchestration ownership transfer evidence only. |
| Run-lock conflict | Orchestration blocking condition only. |
| Run-lock release | Orchestration release evidence only. |

Two or more progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, or live-probe signals that individually lack authority must not combine into stronger authority unless exactly one active `ProgressSignalInterpretationPolicy` row grants the exact combined signal set and requested effect.

## Coverage Contract

Coverage-sensitive facts require `CoverageDimensionProfile` and a current `CoverageAssertion` before absence, pass/fail/unknown, no-change, or negative claims may be emitted.

Coverage dimensions must use `CoverageDomainToken` values for `vulnerability`, `control`, `endpoint`, `directory`, `dns`, `dhcp_ipam`, `flow`, `cloud_inventory`, `source_history`, and inactive deferred `reachability`. Display labels such as `DNS`, `DHCP/IPAM`, `cloud inventory`, and `source history` are not runtime tokens. The coverage catalog in this document supplies default dimensions. Concrete active source-specific row instances are activation-controlled and validation-blocked until present; missing rows fail closed through `SourceAuthorityClosureMatrix` and `120` validation rows.

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
| `CoverageDomainToken` | `stable_core_contract` | Sole runtime token space for coverage domains; non-owners may import by exact name only. |
| `CoverageDomainCatalog` | `stable_core_contract` | Closed catalog of canonical coverage-domain tokens, display labels, default row IDs, and domain status. |
| `CoverageDomainAliasRejectionTable` | `stable_core_contract` | Known display and legacy spellings are rejected at runtime, not accepted as aliases. |
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

0. `020.SourceDatasetCatalogRow`.
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

`SourceAuthorityClosureMatrix` is a stable validation view over active `060` activation-controlled row sets and imported `020` source-dataset catalog rows and feed-category closure rows. It is not a runtime authority class and must not substitute for the underlying row refs.

For every active `LakehouseFeedProfile` and requested absence-sensitive effect, the matrix must prove that exactly one active validated row chain exists for fact type, predicate, source category, selected source-dataset catalog row, source instance override when present, subject scope, object scope, completeness, coverage, staleness, progress-signal interpretation, control-result mapping when applicable, source-history retention when applicable, absence derivation, and watermark policy when applicable.

If the row chain is absent, ambiguous, inactive, checksum-mismatched, out of scope, stale, or unvalidated, the effect must not execute. The result must be `unknown`, `not_applicable`, `source_stale`, `no_op`, or the most specific deterministic error. It must not emit absence, cleanup, retraction, graph expiry, pass/fail, no-change proof, or watermark advancement.

The matrix output must include the requested effect token, feed category, source dataset, selected `020.SourceDatasetCatalogRow` ref/checksum, selected row refs, row checksums, validation refs, package-set refs when package-supplied, blocking reason when blocked, and a `VersionManifest` ref. A closure validation result may be referenced by `030.VersionManifest`, but the underlying activation-controlled artifact refs and checksums remain required.

### MVPSourceAuthorityClosureTupleInventory

`MVPSourceAuthorityClosureTupleInventory` is the tuple-granular materialization inventory for source authority and absence-sensitive effects. `SourceAuthorityClosureMatrix` may summarize closure only after all underlying refs validate. It must not satisfy authority by itself.

The tuple key is exactly:

```text
source_dataset_catalog_row_ref
fact_type
predicate
source_category
subject_scope_selector_checksum
object_scope_selector_checksum
subject_ref_kind
object_value_kind
structured_object_schema_ref_or_null
staleness_policy_ref
coverage_dimension_profile_refs
requested_effect
```

For each tuple in implementation scope, the inventory must contain exactly one selected closure matrix row or exactly one deterministic block row. The selected row must name every consulted `060` row family.

| Consulted row family | Required selected refs |
| --- | --- |
| Authority | Selected `SourceAuthorityProfileRow` ref/checksum and row-set checksum. |
| Completeness | Selected `LakehouseFeedCompletenessProfileRow` refs/checksums and upstream evidence requirements. |
| Coverage | Selected `CoverageDimensionProfile` refs/checksums and coverage-domain token checksum. |
| Staleness | Selected `SourceStalenessPolicy` refs/checksums. |
| Progress signal | Selected `ProgressSignalInterpretationPolicy` refs/checksums for every consulted progress, liveness, lineage, freshness, CDC, graph, source-history, destination-cleanup, or live-probe signal. |
| Supplier visibility | Selected `SupplierCollectionVisibilityProfile` refs/checksums for permission-sensitive or visibility-sensitive effects. |
| Control result | Selected `ControlResultMappingRow` refs/checksums when control output, pass/fail/not-checked, or compliance output is in scope. |
| Source history retention | Selected `SourceHistoryRetentionProfile` refs/checksums when no-change proof or source-history outside-window behavior is consulted. |
| Absence derivation | Selected `AbsenceDerivationPolicy` and `AbsenceDerivationResult` refs/checksums for absence-sensitive outcomes. |
| External-schema authority signal | Selected `ExternalSchemaAuthoritySignalMappingRow` refs/checksums when normalized external-schema signals are consulted. |
| Watermark policy | Selected `ProjectionWatermarkPolicy` refs/checksums when watermark advancement is requested. |

A tuple whose concrete supporting rows are not supplied must be represented by `TODO: product governance must supply source-authority closure tuple row bytes and checksum` and must resolve to `blocked_todo`. A broad source category, source dataset token, completeness receipt, graph state, source-history label, external schema field, validation report, owner prose, or private binding must not substitute for selected tuple refs.

Acceptance criteria:

| ID | Requirement |
| --- | --- |
| `060-SOURCE-AUTHORITY-TUPLE-CLOSURE-AC-001` | `ValidateSpecSet` fails when any source-authority tuple in implementation scope lacks exactly one selected closure matrix row or deterministic block row. |
| `060-SOURCE-AUTHORITY-TUPLE-CLOSURE-AC-002` | `ValidateSpecSet` fails when a selected closure row omits any consulted underlying row family, selected row checksum, validation ref, package ref when package-supplied, or `030.VersionManifest` requirement. |
| `060-SOURCE-AUTHORITY-TUPLE-CLOSURE-AC-003` | `ValidateSpecSet` fails when `SourceAuthorityClosureMatrix` is used as authority without validated underlying refs. |

### SourceAuthorityClosureMatrixRow schema

`SourceAuthorityClosureMatrixRow` is the executable row-level closure proof for one feed category, source dataset, fact type, predicate, subject scope, object scope, and requested absence-sensitive effect. Concrete rows are activation-controlled artifacts or deterministic block rows. Wildcards are forbidden for `fact_type`, `predicate`, `source_dataset`, and `requested_effect`.

The row must use `030.ActivationControlledRowRef` for every selected row reference. Bare string refs fail closure validation. A closure summary, validation report, package label, owner prose, row-set checksum, or manifest ref alone must not substitute for selected row refs and selected row checksums.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_set_id` | Yes | none | Stable row-set ID for the active `SourceAuthorityClosureMatrixRowSet`. |
| `row_id` | Yes | none | Stable row ID scoped to `row_set_id`. |
| `row_version` | Yes | none | Immutable owner version included in `row_checksum`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize, including selected row refs, selected row checksums, closure outcome, validation refs, activation scope, and lifecycle status. |
| `source_dataset_catalog_row_ref` | Yes | none | Structured `030.ActivationControlledRowRef` selected by `020.ResolveSourceDatasetCatalogRow`. |
| `source_dataset_catalog_row_checksum` | Yes | none | Checksum of the selected `020.SourceDatasetCatalogRow`. A mismatch blocks the requested effect before authority row selection. |
| `feed_category` | Yes | none | Must match the selected `020.SourceDatasetCatalogRow.feed_category` and the selected `020.LakehouseFeedCategoryClosureRow.feed_category`. |
| `required_lakehouse_feed_category_closure_row_ref` | Yes | none | Structured `030.ActivationControlledRowRef` for the exact active `020.LakehouseFeedCategoryClosureRow`, or the exact deterministic block row for the selected feed category and requested effect. |
| `source_dataset` | Yes | none | Vendor-neutral dataset token. Must match the selected `020.SourceDatasetCatalogRow.source_dataset`; private routes, table paths, tenant IDs, account lists, scanner sites, host lists, and vendor product names are forbidden in public row bytes. |
| `requested_effect` | Yes | none | One of `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark`. Omission blocks the effect. Each effect must be independently authorized or independently blocked. |
| `fact_type` | Yes | none | Exact fact type. Wildcards are forbidden. |
| `predicate` | Yes | none | Exact predicate. Wildcards are forbidden. |
| `source_instance_policy` | Yes | none | One of `exact_instance_required`, `dataset_default_allowed`, or `blocked`. `dataset_default_allowed` must not widen source authority unless the selected underlying authority row permits dataset-default matching. |
| `subject_scope_selector` | Yes | none | Canonical `030.ScopeSelector` for the subject scope. The selector checksum must be included in manifest requirements when the row affects output. |
| `object_scope_selector` | Yes | none | Canonical `030.ScopeSelector` for the object scope. The selector checksum must be included in manifest requirements when the row affects output. |
| `required_source_authority_row_refs` | Required unless deterministically blocked before authority selection | `[]` only when the block scope prevents authority selection | Structured refs for exact `SourceAuthorityProfileRow` rows or exact deterministic block rows. |
| `required_completeness_profile_row_refs` | Required when the requested effect depends on completeness | `[]` only when the row is positive-only or deterministically blocked before completeness evaluation | Structured refs for exact `LakehouseFeedCompletenessProfileRow` rows or exact deterministic block rows. |
| `required_coverage_dimension_profile_refs` | Required when coverage-sensitive | `[]` only for non-coverage-sensitive effects or deterministic block rows whose `block_scope` is earlier than coverage | Structured refs for exact `CoverageDimensionProfile` rows or exact deterministic block rows. |
| `required_staleness_policy_refs` | Required when staleness can affect output | `[]` only when staleness is not consulted or the row is deterministically blocked before staleness evaluation | Structured refs for exact `SourceStalenessPolicy` rows or exact deterministic block rows. |
| `required_progress_signal_policy_refs` | Required when progress signals are consulted | `[]` when no progress signal is consulted | Structured refs for exact `ProgressSignalInterpretationPolicy` rows. Weak progress signals default to diagnostic-only unless an exact active row grants the exact requested effect. |
| `required_control_result_mapping_refs` | Required for control-result output | `[]` for non-control output | Structured refs for exact `ControlResultMappingRow` rows. Missing external result-state mappings must block or map to an explicit non-authoritative state; no pass/fail default is allowed. |
| `required_source_history_retention_refs` | Required for source-history no-change claims | `[]` when source history is not consulted | Structured refs for exact `SourceHistoryRetentionProfile` rows. Source-history silence must not become no-change proof without this row family. |
| `required_external_schema_authority_signal_mapping_refs` | Required when external schema signals are consulted | `[]` when external schema signals are not consulted | Structured refs for exact `ExternalSchemaAuthoritySignalMappingRow` rows. Mapping rows and normalized fields remain non-authoritative without these refs. |
| `required_absence_derivation_policy_refs` | Required for absence-sensitive outputs | `[]` only when the selected row is deterministically blocked before absence derivation | Structured refs for exact `AbsenceDerivationPolicy` rows. |
| `required_watermark_policy_refs` | Required for watermark effects | `[]` for non-watermark requested effects | Structured refs for exact `ProjectionWatermarkPolicy` rows. |
| `underlying_row_refs` | Yes | none | Canonically sorted structured refs for every row family required by `SourceAuthorityRowCatalogClosure`, including all required row refs listed above. This field is the aggregate closure inventory and must not replace the named required-ref fields. |
| `underlying_row_checksums` | Yes | none | Checksum map keyed by structured row ref. Every selected underlying row ref must have exactly one checksum entry. The map participates in `row_checksum`. |
| `closure_outcome` | Yes | none | Owner-local state: `closed`, `deterministically_blocked`, or `blocked_validation`. Promotion maps this value through `000.SourceAuthorityClosureStateCrosswalk`. |
| `deterministic_block_code` | Required when `closure_outcome = deterministically_blocked` | null only when `closure_outcome = closed` or `blocked_validation` | Exact owner code for deterministic no-op/block rows. |
| `block_scope` | Required when `closure_outcome = deterministically_blocked` | null only when `closure_outcome = closed` or `blocked_validation` | One of `category`, `source_dataset`, `fact_predicate`, `scope`, or `requested_effect`. |
| `mutation_prohibition_validation_refs` | Required for deterministic block and no-op rows | `[]` only for closed active rows | Refs proving no fact, correction, graph delta, cleanup, watermark, compliance pass/fail, no-change proof, package activation, visibility effect, or backend mutation is emitted for the blocked scope. |
| `package_set_ref` | Required when any selected row is package-supplied | null otherwise | Active `100.ProductionPackageSetManifest` ref. A package-supplied row without this ref blocks closure with the package-set owner error. |
| `version_manifest_requirement` | Yes | none | Exact required `030.VersionManifest` members for the selected source-dataset catalog row, feed-category closure row, source-authority closure row, all underlying row refs, row checksums, deterministic block rows, validation refs, package refs, selector checksums, and runtime state refs. |
| `validation_refs` | Yes | none | Non-empty refs to `120-SOURCE-CLOSURE-*` and `120-SOURCE-DATASET-CATALOG-*` validation rows covering positive authorization, missing rows, ambiguous rows, deterministic blocks, private-binding rejection, manifest inclusion, checksum mismatch, and mutation prohibition. |
| `activation_scope` | Yes | none | `030.ActivationScope`. Production selection requires the request scope to be covered through the selected `030.ScopeSelectorContext`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. Inactive, retired, quarantined, out-of-scope, TODO-bearing, unvalidated, or checksum-mismatched rows block the requested effect before mutation. |

When `closure_outcome = closed`, the row authorizes only the exact `requested_effect` for the exact category, source dataset, fact type, predicate, subject scope, and object scope named by the row. Authorization of one requested effect must not authorize any other requested effect.

When `closure_outcome = deterministically_blocked`, the row must include `deterministic_block_code`, `block_scope`, `mutation_prohibition_validation_refs`, empty allowed effects for the blocked scope, and exact deterministic block refs in `version_manifest_requirement`. Deterministic block rows emit no fact, correction, cleanup, graph expiry, graph delta, retraction, watermark, compliance pass/fail, package activation effect, visibility effect, or no-change proof.

When `closure_outcome = blocked_validation`, the row must reject before any absence-sensitive mutation and must identify the failed or missing validation refs through `validation_refs` or the owner error context. `blocked_validation` must not be rendered as a deterministic no-op and must not satisfy promotion closure.

Rows with missing, inactive, ambiguous, stale, checksum-mismatched, out-of-scope, package-set-mismatched, manifest-incomplete, unvalidated, private-leaking, or `TODO:` dependencies must emit no absence-sensitive mutation and must resolve to the most specific deterministic owner error before `DeriveAbsenceOrUnknown` produces an output.

### RowCatalogPlacementRule

Stable source-authority behavior remains in this file. Concrete active source-specific row instances must be activation-controlled artifacts or deterministic block rows. This file must not embed production-active private source rows, tenant inventories, scanner site names, host lists, account lists, credentials, private routes, raw private fixture bytes, or private source binding values.

### MVP Source Authority Closure Catalog Requirements

The selected MVP scope must provide active row sets or exact deterministic block rows for every row-set family below. Concrete row instances are activation-controlled artifacts and must not be embedded as private production rows in this core spec.

| Row-set family | Required closure behavior |
| --- | --- |
| `SourceAuthorityProfileRowSet` | Required for positive authority and every absence-sensitive requested effect. |
| `LakehouseFeedCompletenessProfileRowSet` | Required whenever feed completeness can affect absence, cleanup, retraction, graph expiry, or watermark. |
| `CoverageDimensionProfileRowSet` | Required for every coverage-sensitive fact, predicate, or negative/control output. |
| `SourceStalenessPolicyRowSet` | Required whenever stale state can affect output or block an effect. |
| `ProgressSignalInterpretationPolicyRowSet` | Required whenever a progress, liveness, freshness, lineage, CDC, queue, ack, graph, source-history, destination-cleanup, or live-probe signal is consulted. |
| `SupplierCollectionVisibilityProfileRowSet` | Required whenever permission or hidden-object visibility can affect output. |
| `ControlResultMappingRowSet` | Required before control pass/fail/unknown/not-checked/not-applicable output. |
| `SourceHistoryRetentionProfileRowSet` | Required before source-history no-change proof or outside-window handling. |
| `AbsenceDerivationPolicyRowSet` | Required before `DeriveAbsenceOrUnknown` emits any absence-sensitive outcome. |
| `ProjectionWatermarkPolicyRowSet` | Required before source, projection, graph, or presence-only watermark effects. |
| `ExternalSchemaAuthoritySignalMappingRowSet` | Required only when external schema signals are consulted for authority effects. |
| `SourceAuthorityClosureMatrixRowSet` | Required as the closure proof row set or deterministic block row set for the exact requested effect. |

Requested effect coverage is total:

| `requested_effect` | Required result |
| --- | --- |
| `absence` | Full row chain or deterministic block row. |
| `cleanup` | Full row chain or deterministic block row. |
| `retraction` | Full row chain or deterministic block row. |
| `graph_expiry` | Full row chain plus graph handoff validation, or deterministic block row. |
| `watermark` | Full row chain including `ProjectionWatermarkPolicy`, or deterministic block row. |

#### MVPGoldPredicateSourceAuthorityClosure

For MVP, every active `080.MVPGoldFactPredicateContractRowSetClosure` row must have at least one exact `SourceAuthorityProfileRow` or one exact deterministic source-authority block row before the predicate can emit production output. The source-authority row must set `required_gold_fact_predicate_contract_ref` to the exact `080` row ID.

| `080` predicate row | Minimum `060` closure requirement | Additional required refs |
| --- | --- | --- |
| `gfp-mvp-host-id-resolved-to-canonical-v1` | Exact positive authority row or deterministic block. | Source-dataset catalog row, resolver eligibility refs, validation refs. |
| `gfp-mvp-host-id-has-identifier-v1` | Exact positive authority row or deterministic block. | Identifier scope refs and resolver eligibility refs. |
| `gfp-mvp-host-attr-has-os-v1` | Exact positive authority row or deterministic block. | Structured schema ref `080.struct.os_descriptor.v1`; source staleness refs when stale state can block output. |
| `gfp-mvp-host-attr-lifecycle-v1` | Exact positive authority row or deterministic block. | Structured schema ref `080.struct.host_lifecycle_state.v1`; source staleness refs when stale state can block output. |
| `gfp-mvp-host-attr-management-v1` | Exact positive authority row or deterministic block. | Structured schema ref `080.struct.host_management_state.v1`. |
| `gfp-mvp-host-ip-had-ip-v1` | Exact positive authority row or deterministic block. | DNS/DHCP/IPAM staleness policy refs when lease or assignment time affects output. |
| `gfp-mvp-host-dns-has-dns-name-v1` | Exact positive authority row or deterministic block. | DNS staleness policy refs when TTL or observation time affects output. |
| `gfp-mvp-host-dns-resolved-to-ip-v1` | Exact positive authority row or deterministic block. | DNS staleness policy refs when TTL or observation time affects output. |
| `gfp-mvp-host-vuln-has-vulnerability-v1` | Exact positive authority row plus exact coverage closure, or deterministic block. | `CoverageDimensionProfile`, `CoverageAssertion`, source staleness refs, absence derivation refs for absence or fixed-state effects. |
| `gfp-mvp-host-software-runs-software-v1` | Exact positive authority row or deterministic block. | Structured schema ref `080.struct.software_instance_key.v1`. |
| `gfp-mvp-control-failed-v1` | Exact control-result authority row or deterministic block. | `ControlResultMappingRowSet`, coverage refs, source staleness refs. |
| `gfp-mvp-control-passed-v1` | Exact control-result authority row or deterministic block. | `ControlResultMappingRowSet`, coverage refs, source staleness refs. |
| `gfp-mvp-control-unknown-v1` | Exact control-result authority row or deterministic block. | `ControlResultMappingRowSet`, coverage refs, source staleness refs. |
| `gfp-mvp-identity-member-of-v1` | Exact directory membership authority row or deterministic block. | Directory membership coverage refs and supplier visibility refs when hidden or permission-limited membership can affect output. |
| `gfp-mvp-user-logged-on-to-v1` | Exact positive authority row or deterministic block. | Supplier visibility refs when logged-on evidence is permission-limited. |
| `gfp-mvp-user-used-device-v1` | Exact positive authority row or deterministic block. | Supplier visibility refs when device-use evidence is permission-limited. |
| `gfp-mvp-flow-observed-connection-v1` | Exact positive authority row may exist; absence-sensitive effects are blocked by default. | `020.network_flow` catalog row, `050.FlowRoleEvidence` handoff, graph projection refs when graph output is requested. |
| `gfp-mvp-exposure-observed-v1` | Exact positive observed-exposure authority row or deterministic block. | Must not authorize theoretical reachability or service access. |

For coverage-sensitive facts, the selected source-authority row must name exact `CoverageDimensionProfile` refs and the producing run must include exact `CoverageAssertion` refs. For control facts, the selected row must name exact `ControlResultMappingRowSet` refs. For DNS, DHCP, IPAM, and flow rows, exact staleness policy refs are required when freshness can affect output. For vulnerability absence, fixed-state, cleanup, retraction, graph expiry, or no-change effects, the selected closure must include coverage, completeness, staleness, and absence-derivation refs.

Positive `observed_network_flow_fact.observed_connection` authority may exist for observed traffic only. Flow non-observation, missing flow rows, weak flow progress signals, graph derived-view lag, and graph backend state remain blocked for absence, cleanup, retraction, graph expiry, and watermark by default.

Theoretical reachability and host ownership remain deterministic blocks in MVP. A source-authority row must not authorize `modeled_reachability_fact`, `has_theoretical_reachability`, `host_ownership_fact.owned_by`, or an owner-like host predicate unless a later active owner spec replaces the block rows and adds passing validation.

Every `SourceAuthorityClosureMatrixRow` must either resolve the full row chain for the exact scope and effect or select one deterministic block row for the exact blocked scope and effect. Missing, duplicated, ambiguous, inactive, checksum-mismatched, out-of-scope, stale, unvalidated, or `TODO` member rows block the requested effect and emit no absence, cleanup, retraction, graph expiry, watermark, control pass/fail, source-history no-change, or compliance negative output.

### MVPSourceAuthorityClosureInventory

The MVP source-authority closure inventory is total over every active tuple of feed category, source dataset, fact type, predicate, subject scope, object scope, and requested effect. For each tuple in the selected production scope, the active closure row set must contain exactly one `SourceAuthorityClosureMatrixRow` with complete underlying refs or exactly one deterministic block row for the exact blocked tuple. Missing rows, broad source-category fallbacks, wildcard predicates, validation summaries, row-set checksums alone, package labels, or private bindings must not satisfy closure.

| Tuple family | Required closure decision | Default behavior when exact closure is absent | Required refs before active output |
| --- | --- | --- | --- |
| Active absence-sensitive tuple | One closure row or deterministic block row for the exact feed category, source dataset, fact type, predicate, subject scope, object scope, and requested effect. | `SOURCE_AUTHORITY_CLOSURE_INCOMPLETE`; no absence, cleanup, retraction, graph expiry, watermark, pass/fail, no-change proof, or compliance negative output. | Selected `020.SourceDatasetCatalogRow`, source-authority row, completeness row, coverage row when required, staleness row, absence policy, validation refs, package-set refs when package-supplied, and `VersionManifest` refs. |
| `directory_inventory` gold-output attempt | Deterministic block unless exact `070` resolver rows and exact `060` authority rows separately authorize a downstream predicate. | `DIRECTORY_INVENTORY_GOLD_OUTPUT_BLOCKED`; resolver input may remain evidence only. | Exact resolver input refs for evidence generation; no gold refs unless separate owner rows exist. |
| `source_history` no-change proof | Deterministic block unless `SourceHistoryRetentionProfile`, coverage, staleness, completeness, authority, absence, validation, package-set, and manifest refs all cover the exact scope and history window. | `SOURCE_HISTORY_NO_CHANGE_PROOF_BLOCKED`; emit unknown or no-op. | Source-history retention row, coverage assertion, completeness decision, staleness policy, source authority row, absence policy, closure row, and validation refs. |
| `network_flow` non-observation | Deterministic block by default for MVP. | `FLOW_ABSENCE_BLOCKED_MVP`; emit unknown/no-op and no graph mutation. | Positive observed-connection authority for positive facts only; future flow absence requires a new exact closure row and validation. |
| `future_reachability` | Deterministic block while `200` is `inactive_deferred`. | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`; no fact, graph edge, graph property, API claim, package activation effect, or production validation pass. | Deferred validation-only negative fixture refs only. |

A closure row for `directory_inventory`, `source_history`, `network_flow`, or `future_reachability` must include mutation-prohibition refs proving that blocked states produce no fact, no correction, no graph delta, no cleanup, no watermark, no pass/fail, no no-change proof, no compliance negative output, and no authorized-negative API output.

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

#### SourceHistoryNoChangeProofPolicy

Default source-history no-change behavior is no proof. No new `requested_effect` token is introduced. A no-change fact may be considered only as `requested_effect = absence` when all of the following hold:

1. An active `080.GoldFactPredicateContractRow` defines the exact no-change predicate.
2. The selected `SourceHistoryRetentionProfile` covers the exact dataset, scope, and history window.
3. Exact source-history `CoverageDimensionProfile` and `CoverageAssertion` refs validate.
4. Exact completeness, staleness, authority, absence-policy, closure-row, validation, package-set when package-supplied, and `VersionManifest` refs validate.
5. `DeriveAbsenceOrUnknown` emits `absence_authorized = true` for the exact predicate and effect.

When any condition is missing, stale, ambiguous, checksum-mismatched, package-set-mismatched, unmanifested, or `TODO:`-bearing, the result must be `unknown`, diagnostic-only, or explicit no-op. It must emit no no-change proof, fact, correction, graph delta, cleanup, retraction, graph expiry, watermark, control state, compliance negative output, or authorized-negative API label.

#### EvaluateSourceEffectClosure

```text
EvaluateSourceEffectClosure(request, row_sets, runtime_state, version_manifest):
1. Normalize the tuple key: feed_category, source_dataset, fact_type, predicate, subject_scope_selector_checksum, object_scope_selector_checksum, requested_effect.
2. Resolve exactly one selected `020.SourceDatasetCatalogRow` or exact deterministic source-dataset block row through `020.ResolveSourceDatasetCatalogRow`.
3. Resolve exactly one selected `020.LakehouseFeedCategoryClosureRow` or exact deterministic feed-category block row for `feed_category` and `requested_effect`.
4. Resolve exactly one `060.SourceAuthorityClosureMatrixRow` or exact deterministic source-authority block row through `ResolveSourceAuthorityClosureMatrixRow`.
5. If a deterministic block row is selected at steps 2, 3, or 4, validate block scope, block checksum, validation refs, mutation-prohibition refs, package-set refs when package-supplied, and `VersionManifest` refs; then emit exact no-op and no owner output.
6. Resolve and validate every required `SourceAuthorityProfileRow`, `LakehouseFeedCompletenessProfileRow`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `SupplierCollectionVisibilityProfile`, `ControlResultMappingRow`, `SourceHistoryRetentionProfile`, `ExternalSchemaAuthoritySignalMappingRow`, `AbsenceDerivationPolicy`, and `ProjectionWatermarkPolicy` named by the closure row.
7. Evaluate `EvaluateLakehouseFeedCompleteness` when the requested effect depends on completeness.
8. Evaluate coverage, staleness, progress-signal, visibility, control-result, source-history, external-schema authority-signal, absence-policy, and watermark gates in that order when applicable.
9. If any required row is missing, inactive, ambiguous, checksum-mismatched, package-set-mismatched, unvalidated, unmanifested, stale, out-of-scope, or `TODO:`-bearing, emit exactly one owner error or exact no-op with mutation-prohibition evidence and no output effect.
10. If `requested_effect` is `absence`, `cleanup`, `retraction`, or `graph_expiry`, call `DeriveAbsenceOrUnknown` and require exactly one `AbsenceDerivationResult` ref before output.
11. If `requested_effect` is `watermark`, evaluate `ProjectionWatermarkPolicy` and emit exactly one `WatermarkCommitRecord` or watermark no-op.
12. Add selected `020` refs, selected `060` refs, every consulted underlying row ref/checksum, selector checksums, validation refs, runtime state refs, package release refs, package-set refs, mutation-prohibition refs, and owner error refs to `VersionManifest` before any output, no-op visibility, health visibility, API visibility, graph handoff, or validation acceptance.
13. Emit exactly one of: `AbsenceDerivationResult`, `WatermarkCommitRecord`, exact no-op, or exact owner error.
```

This algorithm is the only `060` source-effect closure path. Missing rows must not fall back to broad source category, lakehouse read success, graph backend state, destination cleanup, OCSF fields, source-history silence, weak progress signals, telemetry, package labels, validation summaries, or implementation-local policy.

### SourceAuthorityArtifactLifecycleGuardRows

`060` policy row sets use `030.ActivationControlledArtifactLifecycleMachine.v1` for activation. Runtime completeness, absence, and watermark decisions remain algorithmic unless a future accepted owner specification update defines a lifecycle subject, closed states, closed events, a total transition matrix, and validation rows.

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

`EvaluateLakehouseFeedCompleteness`, `DeriveAbsenceOrUnknown`, and watermark evaluation are pure deterministic algorithms for MVP. They must not be converted into lifecycle machines unless a future accepted owner specification update defines a lifecycle subject, closed states, closed events, total transition matrix, and validation rows.

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
| `source_dataset_catalog_row_ref` | Required when evaluated | Exact `020.SourceDatasetCatalogRow` ref or deterministic source-dataset block row ref. |
| `source_dataset_catalog_row_checksum` | Required when evaluated | Checksum of the selected source-dataset catalog row or deterministic block row. |
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

### Requested-effect authorization table

| Requested effect | Required positive authorization | Default when gate fails | Downstream owner |
| --- | --- | --- | --- |
| `absence` | Exact source-dataset catalog row, exact authority, completeness `complete` or `empty_complete`, staleness pass, coverage pass when coverage-sensitive, and `AbsenceDerivationPolicy`. | `unknown` or deterministic owner error; no negative fact. | `060`, consumed by `080`. |
| `cleanup` | All `absence` gates plus explicit cleanup permission in authority and completeness rows; authoritative lakehouse record deletion is forbidden. | no-op with no cleanup. | `060`, consumed by `080` and `090`. |
| `retraction` | Exact `080` correction route plus `AbsenceDerivationResult` with `requested_effect = retraction`; delete or tombstone evidence must pass all gates. | no-op with no retraction. | `060`, consumed by `080`. |
| `graph_expiry` | Exact graph-expiry permission, `AbsenceDerivationResult`, and downstream `090` graph effect gate. | no-op with no graph delta. | `060`, consumed by `090`. |
| `watermark` | Exact `ProjectionWatermarkPolicy`, source-dataset catalog row, completeness refs, progress-signal refs, blocking-state refs, and commit refs. Weak progress signals are diagnostic-only unless exact row permits. | no watermark advancement. | `060`. |

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
| `COVERAGE_DOMAIN_REQUIRED` | Required coverage-domain token field is omitted or null. |
| `COVERAGE_DOMAIN_TOKEN_INVALID` | Coverage-domain value is not a valid lower-snake-case token string. |
| `COVERAGE_DOMAIN_ALIAS_REJECTED` | Coverage-domain value is a known display or legacy spelling. |
| `COVERAGE_DOMAIN_UNKNOWN` | Coverage-domain value is syntactically valid but absent from `CoverageDomainCatalog`. |
| `COVERAGE_DOMAIN_DUPLICATE` | Coverage-domain token array contains duplicate canonical tokens. |
| `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` | `020` feed category names a token not permitted by the feed-category mapping. |
| `COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT` | Absence-sensitive effect lacks required coverage-domain tokens. |
| `COVERAGE_DOMAIN_INACTIVE_DEFERRED` | `reachability` is used outside the inactive deferred deterministic-block path. |
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
| `COVERAGE_DOMAIN_REQUIRED` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-required` |
| `COVERAGE_DOMAIN_TOKEN_INVALID` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-token-invalid` |
| `COVERAGE_DOMAIN_ALIAS_REJECTED` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-alias-rejected` |
| `COVERAGE_DOMAIN_UNKNOWN` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-unknown` |
| `COVERAGE_DOMAIN_DUPLICATE` | `060` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-duplicate` |
| `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-feed-category-unsupported` |
| `COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT` | `060` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-required-for-effect` |
| `COVERAGE_DOMAIN_INACTIVE_DEFERRED` | `060` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `060.SourceAuthorityErrorContext` | `error-registry-060-coverage-domain-reachability-deferred` |
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

Lock loss, heartbeat uncertainty, stale recovery failure, active lock conflict, or stale fencing blocks watermark advancement for the affected run. These states do not themselves decide whether the source is complete, absent, stale, or covered.

### SourceAuthorityErrorContext

`SourceAuthorityErrorContext` is the owner context schema for `060` authority, completeness, coverage, absence, source-history, control-result, progress-signal, and watermark registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `060` context schema version. |
| `owner_spec` | Yes | Must be `060`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `authority`, `completeness`, `coverage`, `coverage_domain`, `staleness`, `control_result`, `progress_signal`, `source_history`, `visibility`, `absence`, `watermark`, `external_schema_non_authority`, or `cdc_tombstone`. |
| `operation` | Yes | Authority resolution, completeness evaluation, absence derivation, coverage validation, source staleness evaluation, control result mapping, watermark decision, or correction authority handoff. |
| `affected_record_type` | Yes | Authority row, completeness profile row, coverage assertion, staleness policy, progress policy, control mapping, absence result, watermark record, or gold fact candidate. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to source authority rows, completeness profile rows, coverage dimension rows, coverage assertions, staleness policies, progress-signal policies, control-result mappings, source-history profiles, absence policies, external-schema authority-signal rows, watermark policies, fixtures, closure rows, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `fact_type` | No | Required when a fact or absence candidate was evaluated. |
| `predicate` | No | Required when a fact or absence candidate was evaluated. |
| `source_dataset` | No | Vendor-neutral dataset token only. |
| `subject_scope_checksum` | No | Checksum only; raw scope values remain redacted when private. |
| `object_scope_checksum` | No | Checksum only; raw scope values remain redacted when private. |
| `requested_effect` | No | One of `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark` when an effect was requested. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason for blocked output; otherwise null or omitted. Diagnostic, ambiguous, or no-mutation output may include a bounded reason only when it changes caller or audit interpretation. |
| `validation_refs` | Yes | Exact `120` source-authority fixture refs. |
| `redaction_classes` | Yes | External schema raw values, private bindings, credentials, source-native IDs, raw payload bytes, and supplier-private metadata must map to `always_forbidden`. |

### SourceAuthorityErrorNameCanonicalization

`060` is the sole owner for source-dataset catalog registry codes. The canonical source-dataset catalog codes are `SOURCE_DATASET_CATALOG_ROW_MISSING` and `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS`. The alias `SOURCE_DATASET_CATALOG_MISSING` is invalid input to `110.GenerateErrorCodeRegistry` and must fail validation before API, export, health, audit, or validation-visible output.

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

`ExternalSchemaAuthoritySignalMappingRow` is the only `060` row that can make a normalized external-schema field visible to source-authority logic. Wildcards are forbidden. The row is an activation-controlled row and must satisfy the `030.ActivationControlledRowField` table below before any authority, coverage, staleness, control, absence, cleanup, retraction, graph expiry, or watermark effect may be attempted.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.string` | yes | none | no | no | 1..256 Unicode scalar values | n/a | reject | n/a | ordered:1 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `row_version` | `040.ScalarType.string` | yes | none | no | no | 1..64 Unicode scalar values | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row schema version | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `external_schema_profile_ref` | `030.ActivationControlledRowRef` or `030.ActivationControlledArtifactRef` | yes | none | no | no | exact active `050.ExternalSchemaProfile` | n/a | reject | n/a | ordered:3 | yes | closed | `110` | external schema profile ref/checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `field_path` | `040.ScalarType.field_path` | yes | none | no | no | exact normalized field path; wildcards forbidden | n/a | reject | n/a | ordered:4 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `source_dataset` | `040.ScalarType.enum_token` | yes | none | no | no | vendor-neutral token only | n/a | reject | n/a | ordered:5 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `source_dataset_catalog_row_ref` | `030.ActivationControlledRowRef` | yes | none | no | no | selected `020.SourceDatasetCatalogRow` or deterministic block row | n/a | reject | n/a | no | yes | closed | `110` | selected source-dataset row ref/checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `SOURCE_DATASET_CATALOG_ROW_CHECKSUM_MISMATCH` |
| `source_dataset_catalog_row_checksum` | `040.ScalarType.sha256_hex` | yes | none | no | no | SHA-256 hex | n/a | reject | n/a | no | yes | closed | `110` | source-dataset row checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `SOURCE_DATASET_CATALOG_ROW_CHECKSUM_MISMATCH` |
| `fact_type` | `040.ScalarType.enum_token` | yes | none | no | no | exact fact type | n/a | reject | n/a | ordered:6 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `predicate` | `040.ScalarType.enum_token` | yes | none | no | no | exact predicate | n/a | reject | n/a | ordered:7 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `requested_effect` | owner enum | yes | none | no | no | `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`, `control_state`, or `authority_metadata_consult` | n/a | reject | n/a | ordered:8 | yes | closed | `110` | selected row ref and checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `source_authority_closure_matrix_row_ref` | `030.ActivationControlledRowRef` | yes | none | no | no | exact active closure row or deterministic block row | n/a | reject | n/a | no | yes | closed | `110` | closure row ref/checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `SOURCE_AUTHORITY_CLOSURE_CHECKSUM_MISMATCH` |
| `source_authority_row_ref` | `030.ActivationControlledRowRef` | yes | none | no | no | exact active `SourceAuthorityProfileRow` | n/a | reject | n/a | no | yes | closed | `110` | authority row ref/checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `SOURCE_AUTHORITY_ROW_INVALID` |
| `coverage_dimension_profile_ref` | `030.ActivationControlledRowRef` | conditional:coverage-sensitive effect | null | yes means not coverage-sensitive | no | exact active row | n/a | reject | n/a | no | yes | closed | `110` | coverage row ref/checksum when coverage-sensitive | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `COVERAGE_DIMENSION_PROFILE_INVALID` |
| `source_staleness_policy_ref` | `030.ActivationControlledRowRef` | yes | none | no | no | exact active row | n/a | reject | n/a | no | yes | closed | `110` | staleness row ref/checksum | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `SOURCE_STALENESS_POLICY_INVALID` |
| `control_result_mapping_ref` | `030.ActivationControlledRowRef` | conditional:control output requested | null | yes means no control output requested | no | exact active row | n/a | reject | n/a | no | yes | closed | `110` | control row ref/checksum when control output requested | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `CONTROL_RESULT_MAPPING_INVALID` |
| `absence_derivation_policy_ref` | `030.ActivationControlledRowRef` | conditional:absence-sensitive output requested | null | yes means no absence-sensitive output requested | no | exact active row | n/a | reject | n/a | no | yes | closed | `110` | absence policy ref/checksum when required | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `ABSENCE_DERIVATION_POLICY_INVALID` |
| `projection_watermark_policy_ref` | `030.ActivationControlledRowRef` | conditional:watermark requested | null | yes means no watermark requested | no | exact active row | n/a | reject | n/a | no | yes | closed | `110` | watermark row ref/checksum when required | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | `PROJECTION_WATERMARK_POLICY_INVALID` |
| `validation_refs` | `array<030.ActivationControlledRowRef>` | yes | none | no | no | 1..1024 refs | canonical_set | reject | `row_family,row_id` | no | yes | closed | `110` | validation refs | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | `060.SourceAuthorityScopeSelectorContextSet` | n/a | reject | n/a | no | yes | closed | `110` | selector context and selector checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `SCOPE_SELECTOR_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle transition evidence refs | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INVALID` |
| `row_checksum` | `040.ScalarType.sha256_hex` | yes | derived:`040.CanonicalJSON` | no | no | SHA-256 hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` | `SOURCE_AUTHORITY_CLOSURE_CHECKSUM_MISMATCH` |

Missing, ambiguous, inactive, checksum-mismatched, out-of-scope, package-set-mismatched, unvalidated, unmanifested, `TODO:`-bearing, or deterministic-blocked external-schema authority signal rows leave the external schema signal non-authoritative and block the attempted authority effect.

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

### CoverageDomainToken

`CoverageDomainToken` is the only valid runtime token space for coverage domains. A runtime coverage-domain value must be exactly one canonical token from `CoverageDomainCatalog` after validation by `ValidateCoverageDomainToken`.

Token grammar is `^[a-z][a-z0-9]*(?:_[a-z0-9]+)*$`. Maximum token length is 64 Unicode scalar values. Tokens are case-sensitive. Implementations must not trim, lowercase, uppercase, Unicode-normalize, slash-replace, hyphen-replace, or space-replace token input before validation.

Non-ASCII letters, whitespace, `/`, `-`, punctuation, empty strings, and null are invalid unless the owning schema explicitly allows omission. Unknown lower-snake-case tokens fail with `COVERAGE_DOMAIN_UNKNOWN`. Known display or legacy spellings fail with `COVERAGE_DOMAIN_ALIAS_REJECTED`. Runtime aliases are forbidden. Optional arrays default to `[]` only where the owning schema explicitly says so. Domain arrays must reject duplicates and sort accepted canonical tokens lexically before checksum computation.

`reachability` is an inactive deferred coverage-domain token. It may validate only when `context.coverage_domain_context = inactive_deferred_deterministic_block`; every other use fails with `COVERAGE_DOMAIN_INACTIVE_DEFERRED` until `200` is promoted through the active-spec promotion path.

#### CoverageDomainCatalog

| Canonical token | Display label | Default row ID | Domain status |
| --- | --- | --- | --- |
| `vulnerability` | Vulnerability | `cov-vulnerability-default` | `active_requires_source_specific_row` |
| `control` | Control | `cov-control-default` | `active_requires_source_specific_row` |
| `endpoint` | Endpoint | `cov-endpoint-default` | `active_requires_source_specific_row` |
| `directory` | Directory | `cov-directory-default` | `active_requires_source_specific_row` |
| `dns` | DNS | `cov-dns-default` | `active_requires_source_specific_row` |
| `dhcp_ipam` | DHCP/IPAM | `cov-dhcp-ipam-default` | `active_requires_source_specific_row` |
| `flow` | Flow | `cov-flow-default` | `active_requires_source_specific_row` |
| `cloud_inventory` | Cloud inventory | `cov-cloud-inventory-default` | `active_requires_source_specific_row` |
| `source_history` | Source history | `cov-source-history-default` | `active_requires_source_specific_row` |
| `reachability` | Deferred reachability | `cov-reachability-deferred` | `inactive_deferred` |

`CoverageDomainCatalog` is closed for this spec version. Adding, removing, or renaming a canonical token requires a `060` spec change, `000` volatility and define-once updates, `020` feed-category mapping updates when applicable, generated `110` error-registry parity, and concrete `120-COVERAGE-DOMAIN-*` validation rows.

#### CoverageDomainAliasRejectionTable

This table is a runtime rejection table, not an alias table. The canonical-token column exists only to explain the rejected value and must not be used for conversion.

| Rejected value | Canonical token | Required runtime result |
| --- | --- | --- |
| `DNS` | `dns` | `COVERAGE_DOMAIN_ALIAS_REJECTED` |
| `DHCP/IPAM` | `dhcp_ipam` | `COVERAGE_DOMAIN_ALIAS_REJECTED` |
| `cloud inventory` | `cloud_inventory` | `COVERAGE_DOMAIN_ALIAS_REJECTED` |
| `source history` | `source_history` | `COVERAGE_DOMAIN_ALIAS_REJECTED` |
| `deferred reachability` | `reachability` | `COVERAGE_DOMAIN_ALIAS_REJECTED` |
| `future_reachability` | `reachability` | `COVERAGE_DOMAIN_ALIAS_REJECTED` when used as a coverage-domain value; remains valid only as a `020` feed-category token. |

#### ValidateCoverageDomainToken

```text
ValidateCoverageDomainToken(value, context):
1. If value is omitted or null:
   a. If context requires a token, return COVERAGE_DOMAIN_REQUIRED.
   b. If context is an optional token array, materialize [] and return the materialized array to the caller.
2. If value is not a JSON string, return COVERAGE_DOMAIN_TOKEN_INVALID.
3. If value exactly matches a row in CoverageDomainAliasRejectionTable, return COVERAGE_DOMAIN_ALIAS_REJECTED.
4. If value does not match ^[a-z][a-z0-9]*(?:_[a-z0-9]+)*$, return COVERAGE_DOMAIN_TOKEN_INVALID.
5. If value is not in CoverageDomainCatalog, return COVERAGE_DOMAIN_UNKNOWN.
6. If value = reachability and context.coverage_domain_context is not inactive_deferred_deterministic_block, return COVERAGE_DOMAIN_INACTIVE_DEFERRED.
7. Return value unchanged.
```

#### ValidateCoverageDomainTokenArray

```text
ValidateCoverageDomainTokenArray(values, context):
1. If values is omitted, materialize [] only where the owning field default is [].
2. Validate each element with ValidateCoverageDomainToken.
3. Reject duplicates with COVERAGE_DOMAIN_DUPLICATE.
4. Sort accepted tokens lexically before row checksum computation.
5. If the array is empty and the row allows an absence-sensitive effect, reject with COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT unless the selected SourceAuthorityClosureMatrix row declares coverage not applicable for that exact effect.
6. Return the sorted array unchanged by any aliasing, display-label conversion, case conversion, or Unicode normalization.
```

#### CoverageDomainErrorCodeSet

| Error code | Required use |
| --- | --- |
| `COVERAGE_DOMAIN_REQUIRED` | Required token field is omitted or null. |
| `COVERAGE_DOMAIN_TOKEN_INVALID` | Value is not a valid lower-snake-case token string. |
| `COVERAGE_DOMAIN_ALIAS_REJECTED` | Value is a known display or legacy spelling. |
| `COVERAGE_DOMAIN_UNKNOWN` | Value is syntactically valid but absent from the catalog. |
| `COVERAGE_DOMAIN_DUPLICATE` | Token array contains duplicate canonical tokens. |
| `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` | `020` feed category names a token not permitted by the feed-category mapping. |
| `COVERAGE_DOMAIN_REQUIRED_FOR_EFFECT` | Absence-sensitive effect lacks required coverage-domain tokens. |
| `COVERAGE_DOMAIN_INACTIVE_DEFERRED` | `reachability` is used outside the deferred deterministic-block path. |

### CoverageDimensionProfile catalog

`CoverageDimensionProfile` rows are stable at the schema level and activation-controlled at the source-specific row level.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `coverage_profile_row_id` | Yes | none | Stable row ID scoped to the row set. |
| `coverage_domain` | Yes | none | Required `CoverageDomainToken` validated by `ValidateCoverageDomainToken`; display labels, legacy spellings, aliases, null, empty string, and unknown tokens fail with `CoverageDomainErrorCodeSet` owner errors. |
| `source_category` | Yes | none | Vendor-neutral source category. |
| `source_dataset` | Yes | none | Vendor-neutral source dataset. |
| `fact_type` | Yes | none | Exact fact type. |
| `predicate` | Yes | none | Exact predicate. |
| `scope_selector` | Yes | none | `030.ScopeSelector` for the source/feed scope under `060.SourceAuthorityScopeSelectorContextSet`. |
| `required_dimensions` | Yes | none | Non-empty ordered set of required dimensions for the domain. |
| `authorized_dimension_states` | Yes | `covered` only | Authorized states are `covered` and explicitly permitted `empty_covered` only. |
| `blocking_dimension_states` | Yes | all blocking states | Must include `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `stale`, `unsupported`, `not_checked`, and `error` unless a narrower owner row maps a non-negative result. |
| `freshness_policy_ref` | Yes | none | Exact `SourceStalenessPolicy` ref for coverage freshness. |
| `visibility_profile_ref` | Required when permissions affect visibility | null only when permissions cannot affect coverage | Exact `SupplierCollectionVisibilityProfile` ref. |
| `validation_refs` | Yes | none | Non-empty positive and negative validation refs. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

`ResolveCoverageDimensionProfile(request, active_row_set)` must validate the row set through `030.ActivationControlledArtifactRef`, apply exact non-scope predicates for coverage domain, source category, source dataset, fact type, and predicate, then call `030.ResolveScopedRow` over `scope_selector` using the `coverage_scope` context. Missing rows return `COVERAGE_DIMENSION_ROW_MISSING`; multiple maximal rows under `030.CompareScopeSpecificity` return `COVERAGE_DIMENSION_ROW_AMBIGUOUS`. The selected ref, checksum, context ref, selector checksum, and activation artifact ref must appear in `VersionManifest`.

| Row ID | Domain | Dimension | Required evidence | Freshness policy | Permission-limited behavior | Partial behavior | Stale behavior | Default output | CoverageClosureStatus |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cov-vulnerability-default` | vulnerability | scanner target, credentialed status, plugin/check scope, scan window | coverage assertion plus scanner evidence | `SourceStalenessPolicy` | `permission_limited` blocks absence | `partial_known_gap` or `partial_unknown_gap` | stale blocks absence | unknown | `requires_source_specific_row` |
| `cov-control-default` | control | benchmark/check scope, applicability, evaluation status | `ControlResultMappingRow` and coverage assertion | owner row | not checked/unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-endpoint-default` | endpoint | asset scope, enrollment visibility, collection window | feed completeness plus authority row | owner row | blocks absence | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-directory-default` | directory | tenant/domain, group/member scope, hidden membership visibility | directory completeness evidence | owner row | blocks nonmembership | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dns-default` | dns | zone/source scope, TTL, authoritative source | DNS feed evidence | TTL-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dhcp-ipam-default` | dhcp_ipam | scope, lease window, authoritative system | lease/IPAM evidence | lease-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-flow-default` | flow | sensor scope, collection point, time window, role evidence | flow feed evidence | window policy | unknown | partial | stale | unknown | `requires_source_specific_row` |
| `cov-cloud-inventory-default` | cloud_inventory | account/project/subscription, region, resource type | inventory feed and source history | source-specific | permission_limited | partial | stale | unknown | `requires_source_specific_row` |
| `cov-source-history-default` | source_history | source-native history window, retention | history retention profile | history-window policy | unknown | unknown | outside window no proof | unknown | `requires_source_specific_row` |
| `cov-reachability-deferred` | reachability | topology, route, policy, NAT, workload, identity context | inactive `200` placeholders | inactive | no MVP claim | no MVP claim | no MVP claim | no-op | `inactive_deferred` |

`CoverageClosureStatus` is a closed enum: `closed_default`, `requires_source_specific_row`, and `inactive_deferred`. Source-specific absence-sensitive outputs remain blocked until a `requires_source_specific_row` domain has a matching active source-specific coverage row.

### Coverage-sensitive blocked outputs

| Missing coverage domain | Blocked output |
| --- | --- |
| vulnerability | vulnerability absence and fixed-state absence. |
| control | control pass, fail, unknown, not-checked, and not-applicable facts. |
| endpoint | endpoint disappearance, non-observation, and cleanup. |
| directory | group non-membership, user non-membership, hidden-membership absence. |
| dns | DNS absence, host deletion from TTL expiry, negative DNS fact. |
| dhcp_ipam | DHCP lease absence, IPAM absence, host deletion from lease expiry. |
| flow | flow non-observation and observed-connection absence. |
| cloud_inventory | missing cloud resource and resource deletion inference. |
| source_history | no-change proof outside supported history window. |

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

`ProgressSignalInterpretationPolicy` must be total for the closed signal families below. The default `authority_limit` is `diagnostic_only`; the default `allowed_effects` is `[]`. A row that grants any effect must name the exact `source_dataset_catalog_row_ref`, requested effect, evidence inputs, validation refs, and `VersionManifest` requirements.

| Signal family | Default authority limit | Default allowed effects | Required non-authority behavior |
| --- | --- | --- | --- |
| `cursor_exhaustion` | `diagnostic_only` | `[]` | Progress candidate only; no absence by default. |
| `delta_token_complete` | `diagnostic_only` | `[]` | Resume state only. |
| `cdc_offset` | `diagnostic_only` | `[]` | Replay state only; not fact time or completeness. |
| `cdc_heartbeat` | `diagnostic_only` | `[]` | Liveness diagnostic only. |
| `queue_drain` | `diagnostic_only` | `[]` | Delivery diagnostic only. |
| `end_to_end_ack` | `diagnostic_only` | `[]` | Delivery diagnostic only. |
| `provenance_closure` | `diagnostic_only` | `[]` | Lineage diagnostic only. |
| `freshness_artifact` | `diagnostic_only` | `[]` | Freshness diagnostic only. |
| `graph_apply_success` | `diagnostic_only` | `[]` | Derived-view state only. |
| `graph_index_state` | `diagnostic_only` | `[]` | Graph readiness diagnostic only. |
| `graph_drift_check` | `diagnostic_only` | `[]` | Drift diagnostic only; no repair or authority. |
| `destination_cleanup` | `diagnostic_only` | `[]` | Non-authoritative cleanup signal only. |
| `source_history_no_result` | `diagnostic_only` | `[]` | No-change proof forbidden without exact source-history and coverage rows. |
| `live_source_probe_zero_rows` | `diagnostic_only` | `[]` | Validation/exploration only. |
| `lakehouse_read_complete` | `diagnostic_only` | `[]` | Feed-read evidence only; no absence by default. |
| `raw_feed_manifest_valid` | `diagnostic_only` | `[]` | Manifest-shape evidence only. |
| `table_snapshot_available` | `diagnostic_only` | `[]` | Table-state availability only. |
| `table_commit_success` | `diagnostic_only` | `[]` | Table-write evidence only. |
| `table_maintenance_success` | `diagnostic_only` | `[]` | Maintenance evidence only; no source cleanup permission. |
| `catalog_promotion_success` | `diagnostic_only` | `[]` | Visibility transition evidence only. |
| `telemetry_metric` | `diagnostic_only` | `[]` | Operational health input only through `140` and `110`. |

Two or more weak progress signals must not combine into stronger authority unless exactly one active `ProgressSignalInterpretationPolicy` row grants the exact combined signal set and requested effect. Missing signal-family rows default to diagnostic-only and must not authorize absence, cleanup, retraction, graph expiry, watermark, pass/fail, or no-change proof.

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

Every external result state in the selected vocabulary must resolve through exactly one active `ControlResultMappingRow`, an explicit `unknown`, `not_checked`, `not_applicable`, or `error` mapping, or a deterministic block row. Missing external result states must not default to pass or fail. Omitted states block control-state output before compliance export and must not authorize absence-sensitive effects.

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

`SourceHistoryRetentionProfile` rows define source-native history-window interpretation. They do not define Cadastre replay retention and they do not introduce a `no_change_proof` requested-effect token.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `profile_row_id` | Yes | none | Stable row ID scoped to the source-history retention row set. |
| `source_dataset_catalog_row_ref` | Yes | none | Selected `020.SourceDatasetCatalogRow` ref. |
| `source_dataset_catalog_row_checksum` | Yes | none | Must match the selected catalog row. |
| `history_kind` | Yes | none | Closed owner token for the source-native history family. |
| `scope_selector` | Yes | none | `030.ScopeSelector` under `source_history_scope`. |
| `history_window_start_input` | Yes | none | Exact source or feed time field. Current platform time is forbidden. |
| `history_window_end_input` | Yes | none | Exact source or feed time field, or open-ended only when the row explicitly permits it. |
| `minimum_retention_seconds` | Yes | none | Positive integer; zero is invalid for production no-change interpretation. |
| `outside_window_behavior` | Yes | `unknown` | Closed enum: `unknown`, `diagnostic_only`, `explicit_no_op`, or `deterministic_error`. |
| `no_result_inside_window_behavior` | Yes | `unknown` | May authorize a no-change fact only through `requested_effect = absence`, exact predicate contract refs, and exact absence closure refs. |
| `required_coverage_refs` | Yes | none | Exact source-history `CoverageDimensionProfile` refs. |
| `required_absence_policy_refs` | Required when no-change fact output is attempted | `[]` only for diagnostic/no-op behavior | Exact `AbsenceDerivationPolicy` refs. |
| `validation_refs` | Yes | none | Non-empty refs for inside-window, outside-window, missing-history, permission-limited, and replay cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

Closed `outside_window_behavior` values are `unknown`, `diagnostic_only`, `explicit_no_op`, and `deterministic_error`. Outside-window no-result must not become no-change proof, absence, cleanup, graph expiry, retraction, watermark, compliance pass/fail, or authorized-negative API output.

### ProjectionWatermarkPolicy schema

`ProjectionWatermarkPolicy` is the only `060` row family that may authorize source/projection watermark advancement. It is an activation-controlled artifact. Missing or blocked policy rows emit an explicit watermark no-op and must not advance source, projection, graph, compliance, or package watermarks.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to the policy row set. |
| `source_dataset_catalog_row_ref` | Yes | none | Selected `020.SourceDatasetCatalogRow` ref. |
| `requested_effect` | Yes | none | Must equal `watermark` for watermark advancement. |
| `time_basis` | Yes | none | Exact source or feed time basis. Current platform time is forbidden unless the row declares diagnostic-only output. |
| `required_completeness_refs` | Yes | none | Exact completeness rows and decisions required before advancement. |
| `required_progress_signal_refs` | Yes | `[]` only when no progress signal is consulted | Exact progress-signal rows; weak signals default diagnostic-only. |
| `blocking_states` | Yes | none | Total set of states that force no-op, including missing catalog row, stale source, partial gaps, permission limits, failed validation, checksum mismatch, and TODO rows. |
| `commit_refs` | Yes | none | Lakehouse or projection commit refs required before advancement is visible. |
| `no_op_behavior` | Yes | none | Explicit no-op record and owner error when advancement is blocked. |
| `validation_refs` | Yes | none | Non-empty watermark positive, blocked, replay, manifest, and mutation-prohibition refs. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

Source-history no-change proof requires both `SourceHistoryRetentionProfile` and `CoverageDimensionProfile` for source-history coverage unless an accepted owner specification update removes source history from the coverage domain list. Outside-window no-result maps to `unknown` or deterministic no-op, never no-change proof.

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

### SourceEffectClosureRowFamilyFieldClosure

The row-family tables below close the `030.ActivationControlledRowField` precision requirement for the `060` families that can still affect source-effect closure. The common fields apply to every row family in this subsection and must be present even when not repeated in the family-specific table.

| Common field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `row_id` or family-specific stable row ID | Yes | none | Unique inside the row set after canonicalization. Included in row ID and checksum. |
| `row_set_id` | Yes | none | Stable row-set ID. Included in row-set checksum. |
| `row_version` | Yes | none | Immutable owner schema version. Included in row checksum. |
| `source_dataset_catalog_row_ref` | Yes | none | Structured `030.ActivationControlledRowRef`; bare strings fail. Included in `VersionManifest`. |
| `source_dataset_catalog_row_checksum` | Yes | none | SHA-256; mismatch blocks output. |
| `validation_refs` | Yes | none | Non-empty validation refs; `TODO:` blocks selection. Canonical set sorted by ref. |
| `package_set_ref` | Required when package-supplied | null only for non-package rows | Must match `100.ProductionPackageSetManifest`; mismatch blocks output. |
| `activation_scope` | Yes | none | `030.ActivationScope`; selected through `060.SourceAuthorityScopeSelectorContextSet`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize and excluding only `row_checksum`. |

#### LakehouseFeedCompletenessProfileRow field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `feed_category` | Yes | none | Closed `020` feed category token. |
| `scope_selector` | Yes | none | `030.ScopeSelector`; selector checksum included in manifest. |
| `read_target_kind` | Yes | none | Closed `020` read-target kind. |
| `receipt_state` | Yes | none | One of `read_complete`, `read_partial_known_gap`, `read_partial_unknown_gap`, `read_unavailable`, `schema_unavailable`, or `manifest_invalid`. |
| `upstream_evidence_class` | Yes | none | Closed owner token; wildcards forbidden. |
| `upstream_evidence_state` | Yes | none | One of `sufficient`, `insufficient`, `missing`, `permission_limited`, `scope_unavailable`, `source_unavailable`, `stale`, or `not_applicable`. |
| `allowed_effects` | Yes | `[]` | Canonical set over `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`; duplicates rejected. |
| `completeness_decision` | Yes | none | One closed completeness decision; missing combination blocks with `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING`. |
| `blocking_reason` | Conditional when decision blocks an effect | null only when effect is allowed | Most specific owner error. |

#### CoverageDimensionProfile field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `coverage_domain` | Yes | none | One `CoverageDomainToken`; aliases and display labels rejected. |
| `fact_type` | Yes | none | Exact fact type; wildcard forbidden. |
| `predicate` | Yes | none | Exact predicate; wildcard forbidden. |
| `scope_selector` | Yes | none | `030.ScopeSelector` under `coverage_scope`. |
| `required_dimensions` | Yes | none | Non-empty canonical set; duplicates rejected. |
| `authorized_dimension_states` | Yes | `covered` only | Canonical set; may include `empty_covered` only when explicitly validated. |
| `blocking_dimension_states` | Yes | all blocking states | Must include permission, partial, unavailable, stale, unsupported, not-checked, error, and missing states unless exact row maps otherwise. |
| `freshness_policy_ref` | Yes | none | Structured `SourceStalenessPolicy` row ref. |
| `visibility_profile_ref` | Conditional when permission affects coverage | null only when permission cannot affect coverage | Structured `SupplierCollectionVisibilityProfile` row ref. |

#### SourceStalenessPolicy field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `fact_type` | Yes | none | Exact fact type. |
| `predicate` | Yes | none | Exact predicate. |
| `scope_selector` | Yes | none | `030.ScopeSelector` under `staleness_scope`. |
| `time_input_precedence` | Yes | none | Ordered sequence. Current platform time is forbidden unless the row declares diagnostic-only use. |
| `required_time_inputs` | Yes | none unless `staleness_not_applicable = true` | Canonical set; duplicates rejected. |
| `max_age_seconds` or `expiry_rule` | Yes | none | Exactly one bound unless `staleness_not_applicable = true`. |
| `missing_time_behavior` | Yes | `unknown` | One of `unknown`, `stale`, or `deterministic_error`; no implicit current-time fallback. |
| `stale_effects` | Yes | block all absence-sensitive effects | Canonical set of blocked or permitted effects. |

#### SupplierCollectionVisibilityProfile field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `collection_method` | Yes | none | Closed owner token for supplier-reported collection method. |
| `scope_selector` | Yes | none | `030.ScopeSelector` under `visibility_scope`. |
| `visibility_state` | Yes | none | One of `visible`, `hidden_or_permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `cache_only`, or `unsupported`. |
| `volatility_class` | Yes | none | Closed owner token; volatile methods cannot authorize absence without exact row permission. |
| `permission_behavior` | Yes | `blocks_negative_output` | Must state whether permission limits block or allow a named diagnostic. |
| `failed_member_behavior` | Yes | `blocks_negative_output` | Missing failed-member evidence blocks negative output. |
| `absence_behavior` | Yes | `not_authoritative_for_absence` | Must not default to authorized absence. |

#### SourceHistoryRetentionProfile field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `history_kind` | Yes | none | Closed source-history family token. |
| `scope_selector` | Yes | none | `030.ScopeSelector` under `source_history_scope`. |
| `history_window_start_input` | Yes | none | Exact persisted source/feed time input. |
| `history_window_end_input` | Yes | none | Exact persisted source/feed time input or explicit open-window allowance. |
| `minimum_retention_seconds` | Yes | none | Positive integer. |
| `outside_window_behavior` | Yes | `unknown` | `unknown`, `diagnostic_only`, `explicit_no_op`, or `deterministic_error`; no no-change proof. |
| `no_result_inside_window_behavior` | Yes | `unknown` | No-change fact output only through `requested_effect = absence` and exact closure refs. |
| `required_coverage_refs` | Yes | none | Exact source-history coverage row refs. |

#### AbsenceDerivationPolicy field closure

| Field | Required | Default or omission behavior | Bounds and deterministic behavior |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to absence policy row set. |
| `fact_type` | Yes | none | Exact fact type. |
| `predicate` | Yes | none | Exact predicate. |
| `requested_effect` | Yes | none | One of `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark`. |
| `allowed_effects` | Yes | `[]` | Canonical set; selected result must contain `requested_effect` before effect output. |
| `source_state_mapping` | Yes | none | Total map over declared input states or explicit block rows for omitted states. |
| `authorized_outcome` | Conditional when absence fact can be emitted | null when no fact outcome allowed | One `040.FactAbsenceOutcome`; `no_change_proof` is invalid. |
| `blocking_precedence` | Yes | none | Ordered sequence of owner blocking reasons. |
| `owner_error_mapping` | Yes | none | Most specific owner error for missing, stale, permission-limited, unsupported, deterministic block, and manifest failures. |

### ActivationControlledRowSchemaPrecisionHandoff

The following `060` row families can affect source authority, completeness, staleness, coverage, absence, cleanup, retraction, graph expiry, control result state, supplier visibility, source-history interpretation, progress-signal interpretation, or watermarks. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `SourceAuthorityProfileRow` | output_affecting | Closed for source-effect selection by the existing field table plus required `source_dataset_catalog_row_ref`, row checksum, effect token, fact/predicate, underlying refs, deterministic block refs, package refs when supplied, and manifest requirements. |
| `LakehouseFeedCompletenessProfileRow` | absence_sensitive | Closed by `SourceEffectClosureRowFamilyFieldClosure`; completeness states, allowed effects, upstream evidence refs, feed-read refs, blocking behavior, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `SourceAuthorityClosureMatrixRow` | absence_sensitive | Closed for source-effect selection by the `SourceAuthorityClosureMatrixRow schema`, including selected source-dataset catalog row, fact type, predicate, requested effect, closure outcome, underlying structured row refs, deterministic block refs, package refs, row checksum, mutation-prohibition refs, and manifest requirements. |
| `ExternalSchemaAuthoritySignalMappingRow` | output_affecting when external schema signals are consulted | Closed by the full `030.ActivationControlledRowField` table in `ExternalSchemaAuthoritySignalMappingRow schema`; missing, ambiguous, inactive, checksum-mismatched, out-of-scope, unvalidated, unmanifested, package-set-mismatched, deterministic-blocked, or `TODO:` rows produce no authority effect. |
| `CoverageDimensionProfile` | coverage_sensitive | Closed by `SourceEffectClosureRowFamilyFieldClosure`; coverage domain, dimension states, stale behavior, permission behavior, missing-dimension behavior, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `SourceStalenessPolicy` | output_affecting | Closed by `SourceEffectClosureRowFamilyFieldClosure`; time input precedence, stale effects, expiry behavior, missing input behavior, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `ControlResultMappingRow` | output_affecting for control facts | Closed for source-effect selection by total mapping requirement over external control result states or explicit blocked omitted states. |
| `SupplierCollectionVisibilityProfile` | absence_sensitive | Closed by `SourceEffectClosureRowFamilyFieldClosure`; method visibility, volatility, permission, cache, expected-count, failed-member, absence behavior, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `ProgressSignalInterpretationPolicy` | output_affecting when progress signals are consulted | Closed for the signal families in `ProgressSignalInterpretationPolicy total matrix`; default `authority_limit = diagnostic_only` and `allowed_effects = []`. |
| `SourceHistoryRetentionProfile` | output_affecting for history/no-change claims | Closed by `SourceEffectClosureRowFamilyFieldClosure` and `SourceHistoryRetentionProfile`; native history windows, outside-window behavior, source-history refs, non-authority defaults, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `AbsenceDerivationPolicy` | absence_sensitive | Closed by `SourceEffectClosureRowFamilyFieldClosure`; source-state mapping, requested effects, allowed effects, blocking states, output outcome, owner errors, source-dataset refs, package refs, validation refs, checksums, and manifest requirements are defined. |
| `ProjectionWatermarkPolicy` | watermark_affecting | Closed for source-effect selection by `ProjectionWatermarkPolicy schema`, including time basis, completeness refs, progress refs, blocking states, commit refs, validation refs, and no-op behavior. |

`allowed_effects`, `required_*_refs`, `subject_ref_kind_scope`, `object_value_kind_scope`, `authorized_dimension_states`, `blocking_dimension_states`, and `stale_effects` must use `canonical_set` unless this spec declares `ordered_sequence`. `time_input_precedence` must use `ordered_sequence`. State mappings for absence, control results, progress signals, and watermarks must be total over declared input states or explicitly block omitted states.

Missing row and ambiguous row errors must remain distinct and owner-specific. Source-authority evaluation must fail before gold derivation, correction, graph handoff, or watermark advancement when any selected `060` row family remains `TODO:` or lacks a structured `030.ActivationControlledRowRef`.

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
| `060-RUNLOCK-NONAUTH-AC-001` | `val-060-runlock-signal-nonauthority` proves that run-lock heartbeat, lease expiry, stale recovery, conflict, release, lock loss, heartbeat uncertainty, and stale fencing signals cannot authorize absence, cleanup, retraction, graph expiry, source completeness, coverage, control pass/fail, source-history no-change, or watermark advancement; unauthorized use emits `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` and has concrete no-mutation proof. |
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
| `060-SOURCE-CLOSURE-AC-015` | Missing active row catalogs or deterministic block rows for the full source-closure row chain emit no absence, cleanup, retraction, graph expiry, watermark, control pass/fail, source-history no-change, or compliance negative output. |
| `060-SOURCE-CLOSURE-AC-018` | Every active absence-sensitive effect resolves exactly one active closure row chain or one deterministic block row before `DeriveAbsenceOrUnknown` can emit output. |
| `060-SOURCE-DATASET-CATALOG-AC-001` | Every active absence-sensitive effect validates the selected `020.SourceDatasetCatalogRow` ref and checksum before authority profile selection. |
| `060-SOURCE-DATASET-CATALOG-AC-002` | Missing, ambiguous, inactive, checksum-mismatched, package-set-mismatched, TODO-bearing, unvalidated, or unmanifested source-dataset catalog rows emit no fact, cleanup, retraction, graph expiry, watermark, control pass/fail, source-history no-change proof, or compliance negative output. |
| `060-WEAK-SIGNAL-DIAGNOSTIC-AC-001` | Every weak progress signal family defaults to `diagnostic_only` with `allowed_effects = []`. |
| `060-SOURCE-CLOSURE-AC-016` | Deterministic block rows emit no fact, cleanup, graph expiry, retraction, watermark, compliance pass/fail, or no-change proof. |
| `060-SOURCE-CLOSURE-AC-017` | Source-history no-change proof fails without both retention and coverage rows. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-001` | External schema signals remain non-authoritative without an exact active `ExternalSchemaAuthoritySignalMappingRow`. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-002` | Missing, inactive, checksum-mismatched, or out-of-scope external-schema authority signal row sets block authority effects and do not reinterpret normalized metadata as source authority. |
| `060-EXTERNAL-SCHEMA-AUTHORITY-AC-003` | Ambiguous external-schema authority signal rows emit `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_AMBIGUOUS` and no authority, absence, cleanup, graph expiry, control-state, or watermark effect. |
| `060-ERROR-REGISTRY-CLOSURE-AC-001` | All `060` source-closure owner errors generate complete `110.ErrorCodeRegistryRow` rows with no `TODO` values. |
| `060-COVERAGE-DOMAIN-TOKEN-AC-001` | The catalog contains exactly the ten canonical tokens `vulnerability`, `control`, `endpoint`, `directory`, `dns`, `dhcp_ipam`, `flow`, `cloud_inventory`, `source_history`, and `reachability`, with no mixed-case, slash-containing, or spaced runtime values. |
| `060-COVERAGE-DOMAIN-TOKEN-AC-002` | `CoverageDimensionProfile.coverage_domain` rejects `DNS`, `DHCP/IPAM`, `cloud inventory`, `source history`, `future_reachability`, and `deferred reachability` with owner-specific coverage-domain errors. |
| `060-COVERAGE-DOMAIN-TOKEN-AC-003` | Domain arrays reject duplicates, sort lexically before checksum computation, and reject empty arrays when an absence-sensitive effect requires coverage. |
| `060-COVERAGE-DOMAIN-REACHABILITY-AC-001` | `reachability` is accepted only for inactive deferred deterministic-block behavior until `200` is promoted. |
| `060-GOLD-SHAPE-AUTHORITY-AC-001` | A source-authority row cannot widen `080.allowed_subject_ref_kinds` or `080.allowed_object_value_kinds`; widening attempts fail before authority selection. |
| `060-GOLD-SHAPE-AUTHORITY-AC-002` | A source-authority row cannot permit `null_value` when the selected predicate contract forbids it. |
| `060-GOLD-SHAPE-AUTHORITY-AC-003` | External schema field presence or absence remains non-authoritative without exact `050`, `060`, and `080` handoff rows. |
| `060-GOLD-SHAPE-AUTHORITY-AC-004` | Missing, inactive, checksum-mismatched, or out-of-scope predicate contract refs block source authority before gold output. |
| `060-GOLD-SHAPE-AUTHORITY-REPLAY-AC-001` | Authority checksum changes and predicate-contract checksum changes are replay-affecting for gold output. |
| `060-LAKEHOUSE-TABLESTATE-NONAUTH-AC-001` | Manifest validity, read completeness, table commit success, maintenance success, and catalog promotion success cannot authorize absence-sensitive effects without exact active `060` rows. |
| `060-LAKEHOUSE-PROGRESS-SIGNAL-AC-001` | All new lakehouse table-state signals default to `diagnostic_only` and `allowed_effects = []`. |

### Structured input source authority acceptance criteria

| ID | Criterion |
| --- | --- |
| `060-STRUCTURED-INPUT-SOURCE-AUTHORITY-AC-001` | Source-closure validation fails when a repository-authored row catalog exists only in Git and is not an active materialized row set. |
| `060-STRUCTURED-INPUT-SOURCE-AUTHORITY-AC-002` | Repository snapshot provenance, exact row-set refs, closure rows, validation refs, materialization refs when packaged, package-set refs when package-supplied, and manifest refs are all required when source-authority rows are repository-authored. |
| `060-STRUCTURED-INPUT-SOURCE-AUTHORITY-AC-003` | Repository-authored private binding leaks fail before absence, cleanup, retraction, graph expiry, watermark, API, export, audit, telemetry, or validation-report output. |
| `060-STRUCTURED-INPUT-SOURCE-AUTHORITY-AC-004` | Stale CI, missing publication manifest, sync-record-only authority attempt, template-only authority attempt, and multi-repository group mismatch produce no absence, cleanup, retraction, graph expiry, or watermark mutation. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `060-SCOPE-AUTHORITY-AC-001` | Exact row selection, dataset-default subset selection, dataset-default subset refusal, ambiguous maximal rows, missing required subject/object dimensions, coverage scope mismatch, source-instance override precedence, and no mutation on ambiguity fixtures pass through `030.ResolveScopedRow`. |
| `060-SCOPE-AUTHORITY-AC-002` | Every output-affecting source-authority scoped selection includes selected row refs, row checksums, selector context refs, selector checksums, and activation artifact refs in `VersionManifest`. |
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
| `060-TODO-SOURCE-CLOSURE-ROW-SETS` | TODO: Provide active row sets or deterministic block rows for `SourceAuthorityClosureMatrixRowSet`, `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfileRowSet`, `SourceStalenessPolicyRowSet`, `ProgressSignalInterpretationPolicyRowSet`, `SupplierCollectionVisibilityProfileRowSet`, `ControlResultMappingRowSet`, `SourceHistoryRetentionProfileRowSet`, `AbsenceDerivationPolicyRowSet`, `ProjectionWatermarkPolicyRowSet`, and `ExternalSchemaAuthoritySignalMappingRowSet` when external schema signals are consulted. | Absence, cleanup, retraction, graph expiry, watermark, control pass/fail, source-history no-change, and compliance negative output. | Product governance plus `020`, `060`, `100`, and `120` validation refs. | Requested effects resolve to deterministic block, unknown, not_applicable, source_stale, no_op, or the most specific owner error; no forbidden mutation. |
