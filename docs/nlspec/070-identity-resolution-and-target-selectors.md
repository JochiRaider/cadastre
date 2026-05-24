---
doc_id: CADASTRE-NLSPEC-070
title: Identity Resolution and Target Selectors
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define canonical identity behavior, resolver determinism, manual review, split behavior, graph correction handoff, and target selector safety.

## Explicit Non-Scope

- Source authority for non-identity facts.
- Graph edge projection.
- External graph taxonomy semantics.
- Direct graph mutation.

## Imports

- `CanonicalEntity`
- `SourceAsset`
- `Identifier`
- `CadastreSilverObservation`
- `SourceAuthorityProfile`
- `VersionManifest`
- `TelemetryAttributePolicy`
- `TelemetryNonAuthorityRule`

- `CanonicalEntitySchema`
- `SourceAssetSchema`
- `IdentifierSchema`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`
- `CanonicalJSON`
- `DecimalPrecisionPolicy.confidence_0_1`
- `GoldFactSubjectRefKindRegistry`
- `GoldFactObjectValueKindRegistry`
- `GoldFactPredicateContractRow`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ResolveScopedRow`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.SourceDatasetCatalogRow`
- `020.SourceDatasetCatalogRowSet`

## Exports

- `ResolverProfile`
- `ResolverProfileRow`
- `ResolverDecisionMatrix`
- `IdentityHardBlockerRow`
- `IdentityConfidenceBand`
- `IdentityReviewRoutingPolicy`
- `SplitPolicy`
- `IdentifierEvidenceClass`
- `IdentityEvidenceItem`
- `IdentifierScope`
- `CandidateGenerationProfile`
- `AssetGenerationBoundary`
- `IdentityDecision`
- `IdentityReviewCase`
- `ResolverActivationReport`
- `ResolverShadowRun`
- `ResolverExplanation`
- `GraphCorrectionHandoff`
- `UnresolvedTargetReference`
- `TargetSelectorSafetyPolicy`
- `ResolveIdentity`
- `ResolverArtifactLifecycleGuardRows`
- `IdentityReviewCaseStateMachineBinding`
- `ResolverProfileRowSet`
- `IdentifierEvidenceClassRowSet`
- `IdentifierScopeRowSet`
- `IdentityHardBlockerRowSet`
- `AssetGenerationBoundaryRowSet`
- `ResolverDecisionMatrixRowSet`
- `IdentityConfidenceBandRowSet`
- `IdentitySplitPolicy`
- `ResolverExplanationPolicy`
- `ResolverActivationReportPolicy`
- `TargetSelectorSafetyPolicyRowSet`

### Exported resolver artifact aliases

The row-set artifact exports below are exact contract names for activation-controlled resolver artifacts. They map to stable row schemas and artifact classes already owned by this file. They do not define second schemas.

| Exported artifact name | Stable row schema or policy | Activation-controlled artifact class |
| --- | --- | --- |
| `ResolverProfileRowSet` | `ResolverProfileRow` | `resolver_profile` |
| `IdentifierEvidenceClassRowSet` | `IdentifierEvidenceClass` | `identifier_evidence_class_row_set` |
| `IdentifierScopeRowSet` | `IdentifierScope` | `identifier_scope_row_set` |
| `IdentityHardBlockerRowSet` | `IdentityHardBlockerRow` | `identity_hard_blocker_row_set` |
| `AssetGenerationBoundaryRowSet` | `AssetGenerationBoundary` | `asset_generation_boundary_row_set` |
| `ResolverDecisionMatrixRowSet` | `ResolverDecisionMatrix` | `resolver_decision_matrix_row_set` |
| `IdentityConfidenceBandRowSet` | `IdentityConfidenceBand` | `identity_confidence_band_row_set` |
| `IdentityReviewRoutingPolicy` | `IdentityReviewRoutingPolicy` | `identity_review_routing_policy` |
| `IdentitySplitPolicy` | `SplitPolicy` | `identity_split_policy` |
| `ResolverExplanationPolicy` | `ResolverExplanation` output policy | `resolver_explanation_policy` |
| `ResolverActivationReportPolicy` | `ResolverActivationReport` activation policy | `resolver_activation_report_policy` |
| `TargetSelectorSafetyPolicyRowSet` | `TargetSelectorSafetyPolicy` | `target_selector_safety_policy` |

A downstream spec may import these artifact names by exact name. It must not infer row-set names from package type labels, implementation module names, or owner prose.

## Resolver Authority

`ResolverProfile` is the sole production authority for identity resolution. A resolver run must fail when no active profile covers resolver run mode, entity type, source scopes, evidence classes, and lifecycle boundary types.

Identity inputs must be materialized as typed `IdentityEvidenceItem` rows before candidate generation, blocker evaluation, review routing, creation, attachment, merge, split, reject, conflict, or no-decision output.

A resolver may create or update `CanonicalEntity`, `SourceAsset`, or `Identifier` records only by emitting records that pass the corresponding `040` schema and `040.ValidateCoreRecord`. `creation_identity_decision_id`, `resolver_profile_id`, and `identity_policy_version` must be present when a `CanonicalEntity` is created. `SourceAsset.source_scope`, `SourceAsset.source_native_identity`, and `SourceAsset.source_asset_type` must be sufficient for `040.SourceAssetSchema`; under-scoped source identity fails before candidate generation. `Identifier` outputs must include typed scope, quality, validity, known time, and evidence refs. `ResolverExplanation` must reference core record IDs and checksums rather than duplicate core fields.

### DirectoryInventoryResolverInputHandoff

A `directory_inventory` source-dataset row may produce `IdentityEvidenceItem` candidates only when all of the following hold:

1. The selected `020.SourceDatasetCatalogRow` ref/checksum and row-set ref/checksum are present, checksum-valid, active with `permitted_production_use = resolver_input_only`, and included in `030.VersionManifest`.
2. The active `ResolverProfileRow` covers the directory scope, entity type, resolver run mode, evidence classes, identifier scopes, lifecycle boundary types, and activation scope.
3. The selected `IdentifierEvidenceClass` row is active and allows the evidence role produced by the directory inventory observation.
4. No lifecycle blocker, hard blocker, under-scoped source identity, private-binding leak, validation blocker, package-set blocker, or manifest blocker applies.

`directory_inventory` does not by itself authorize `identity_membership_fact.member_of`, nonmembership, user deletion, group deletion, source absence, cleanup, graph expiry, graph edge removal, canonical merge, or gold output. Directory display names, group names, usernames, labels, hidden-membership hints, permission-limited states, and directory object presence may be evidence only as typed resolver inputs. `directory_membership`, not `directory_inventory`, is the positive membership dataset for `gfp-mvp-identity-member-of-v1`.

Directory inventory admissibility is total over the states below.

| `directory_inventory` catalog state | Identity evidence behavior | Non-resolver effect behavior |
| --- | --- | --- |
| Active selected row with `permitted_production_use = resolver_input_only` | May produce `IdentityEvidenceItem` candidates when resolver rows, evidence-class rows, validation refs, package-set refs when package-supplied, and manifest refs all validate. | No direct gold, absence, deletion, cleanup, graph expiry, graph edge removal, or membership output. |
| Deterministic block row for direct gold, membership absence, deletion, cleanup, graph expiry, or graph edge removal | Must not produce identity evidence unless the block row explicitly names resolver-input allowance. Resolver-input allowance is not defaulted. | May satisfy proof that the blocked non-resolver effect emits no mutation when mutation-prohibition refs validate. |
| Missing, ambiguous, inactive, checksum-mismatched, unvalidated, package-set-mismatched, unmanifested, private-leaking, or `TODO:` row | Must produce no identity evidence candidate. | Must produce no gold, graph, cleanup, expiry, deletion, absence, or authorized-negative API output. |

`120` must include a directory-inventory direct-gold negative validation row proving that a deterministic block for direct gold output creates no `IdentityEvidenceItem` unless an explicit resolver-input allowance exists.

### TelemetryIdentityNonAuthorityHandoff

Runtime telemetry resource attributes, trace attributes, span attributes, metric attributes, structured-log attributes, service names, process names, host names, container IDs, pod names, runtime instance IDs, exporter IDs, Collector IDs, telemetry backend IDs, backend internal IDs, trace IDs, span IDs, `diagnostic_correlation_ref`, `server_correlation_nonce`, telemetry runtime state IDs, exporter profile runtime IDs, and Collector runtime IDs must not create, attach, merge, split, reject, or score canonical identity by themselves.

Telemetry attributes may appear only as diagnostic context unless a future active `ResolverProfileRow` explicitly consumes a non-telemetry `IdentityEvidenceItem` derived from authoritative Cadastre evidence. The telemetry attribute itself must not be the evidence ref.

A `diagnostic_correlation_ref` must not be materialized as an `IdentityEvidenceItem`. A resolver artifact that maps `diagnostic_correlation_ref` or `server_correlation_nonce` to any durable evidence class fails activation. A resolver run that attempts to consume either value fails before candidate generation with `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` and emits no identity mutation.

A resolver run that attempts to use telemetry-only attributes as durable identity evidence must fail before candidate generation with `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN`.

### Resolver Determinism Closure

Each production resolver run must resolve exactly one active `ResolverProfileRow` before candidate generation. Resolution must fail before mutation when any row, artifact ref, checksum, source scope, identifier scope, lifecycle status, activation scope, or validation ref is missing, inactive, ambiguous, mismatched, under-scoped, or outside the run scope. The selected row must name every output-affecting resolver artifact, and every selected artifact and runtime state ref must appear in `030.VersionManifest`.

A production resolver must not use implementation-local defaults for evidence class authority, blocking keys, candidate caps, hard blocker row sets, generation boundary row sets, confidence bands, review routing, split partitioning, explanation checksum inputs, or learned artifacts. Omitted values must use the defaults in this spec or must emit the most specific resolver error before identity mutation.

### StructuredInputRepositoryResolverHandoff

Repository-authored resolver profiles, evidence class row sets, hard blockers, decision matrices, generation boundaries, review routing policies, split policies, target-selector policies, confidence bands, and resolver activation policies are inert until materialized, validated for the exact repository snapshot, package-set activated when package-supplied, and included in `030.VersionManifest`.

Repository snapshot refs may appear in `ResolverActivationReport` as provenance only. A change proposal, branch update, pull request approval, merge, validation run, or hook success must not create, attach, merge, split, reject, score, or review identity.

Resolver activation for repository-authored artifacts must include exact repository snapshot refs, file manifest checksums, materialization refs, package release refs when created, package-set refs when package-supplied, scenario-gated validation refs, hard-blocker precedence fixtures, weak-evidence rejection fixtures, and manifest refs.

Repository-authored resolver activation must also include repository template contract refs when required, producer CI exact-snapshot validation refs when accepted, maintenance tool contract and invocation refs when resolver artifacts are generated, publication manifest refs when remote publication is consumed, candidate sync record refs when imported by sync, and repository group refs when resolver artifacts depend on mapping or source-authority artifacts from another repository.

A maintenance tool may generate resolver rows only when the tool contract is active, the invocation is checksummed, redaction validation passes, and generated row-set bytes match the materialization result. Template conformance, producer CI success, generated diagnostics, publication manifest existence, and sync status must not create, attach, merge, split, reject, score, or review identity.

Private source bindings in resolver row catalogs fail under `010` and `110` redaction rules before activation, API output, audit output, package reports, telemetry export, or validation-report materialization.

| Error code | Required use |
| --- | --- |
| `RESOLVER_REPOSITORY_ARTIFACT_INACTIVE` | Repository-authored resolver artifact lacks active materialized artifact refs, package-set refs when package-supplied, or manifest refs. |
| `RESOLVER_REPOSITORY_VALIDATION_STALE` | Resolver validation run does not match exact repository snapshot, file manifest checksum, resolver artifact checksum, hard-blocker row set, or scenario output checksum. |
| `RESOLVER_REPOSITORY_PRIVATE_BINDING_LEAK` | Repository-authored resolver artifact or validation output leaks private source binding values or raw private fixture bytes. |
| `RESOLVER_REPOSITORY_TEMPLATE_MISMATCH` | Repository-authored resolver artifact layout, generated output roots, or artifact classes do not match the selected template contract. |
| `RESOLVER_REPOSITORY_CI_STALE` | Producer CI evidence is stale or not exact-snapshot-bound. |
| `RESOLVER_REPOSITORY_TOOL_OUTPUT_MISMATCH` | Tool-generated resolver row-set bytes, invocation checksum, or materialization result checksum do not match. |
| `RESOLVER_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used to create, attach, merge, split, reject, score, or review identity. |

### ResolverProfileRow schema

`ResolverProfileRow` is the row-level executable interface inside `ResolverProfile`. Concrete row sets are activation-controlled artifacts; stable decision semantics remain owned by this spec.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the resolver profile row set. |
| `resolver_profile_id` | Yes | none | Stable profile ID. |
| `profile_version` | Yes | none | Immutable owner version included in explanation and manifest checksums. |
| `run_mode` | Yes | none | Closed token; production selection requires `production` unless a validation/shadow/canary mode is explicitly requested. |
| `entity_type` | Yes | none | Must match one row in `ResolverProfileCoverageMatrix`. |
| `source_scope_selector` | Yes | none | `030.ScopeSelector` over source/feed scopes; concrete private source bindings are forbidden in public rows. |
| `source_scope_match_policy` | Yes | `exact_scope_required` | Closed enum: `exact_scope_required`, `scope_subset_allowed_for_review_only`. `exact_scope_required` means the selected `030.ScopeSelectorContext` has no subset-allowed keys. `scope_subset_allowed_for_review_only` means subset keys may exist only in the selected context and selected row; subset matches may route only to review or no-decision and must not create, attach, merge, or split. |
| `evidence_class_set_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = identifier_evidence_class_row_set`. |
| `identifier_scope_row_refs` | Yes | none | Non-empty canonical array of active `IdentifierScope` row refs. |
| `candidate_generation_profile_ref` | Yes | none | `artifact_class = candidate_generation_profile`. |
| `identity_hard_blocker_row_set_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = identity_hard_blocker_row_set`; required before blocker evaluation and explanation emission. |
| `asset_generation_boundary_row_set_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = asset_generation_boundary_row_set`; required before lifecycle/generation boundary evaluation. |
| `decision_matrix_ref` | Yes | none | `artifact_class = resolver_decision_matrix_row_set`. |
| `confidence_band_ref` | Yes | none | `artifact_class = identity_confidence_band_row_set`. |
| `review_routing_policy_ref` | Yes | none | `artifact_class = identity_review_routing_policy`. |
| `split_policy_ref` | Yes | none | `artifact_class = identity_split_policy`. |
| `explanation_policy_ref` | Yes | none | `artifact_class = resolver_explanation_policy`. |
| `target_selector_policy_ref` | Yes | none | `artifact_class = target_selector_safety_policy`. |
| `allowed_decision_classes` | Yes | none | Non-empty subset of the closed identity decision states. |
| `creation_policy` | Yes | `durable_evidence_creation_allowed` for `host`, `user`, and `service_account`; `creation_forbidden` for unsupported types | Creation still requires durable evidence, exact scope, no existing canonical candidate, no blocker, and selected decision row. |
| `merge_policy` | Yes | `durable_evidence_merge_allowed` | Merge still requires exact durable evidence, exact scope, no blocker, and selected decision row. |
| `learned_artifact_policy` | No | `disabled` | Learned artifacts may propose candidates only when active; they must not override blockers or emit create, attach, or merge decisions by themselves. |
| `validation_refs` | Yes | none | Non-empty refs proving creation, attachment, durable merge, weak rejection, blockers, overflow, review totality, split handoff, selector safety, explanation checksum, and replay. |
| `activation_scope` | Yes | none | `030.ActivationScope`; selected through `070.IdentifierScopeSelectorContext` before the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

## Resolver artifact activation boundary

Stable identity semantics, weak-evidence defaults, hard-blocker precedence, review terminality, and selector safety are owned by this spec. Resolver profiles, decision rows, candidate bounds, and selector policy material are activation-controlled artifacts.

Resolver artifacts must not weaken stable weak-evidence defaults by treating telemetry resource attributes as durable identity evidence. Any package-supplied resolver artifact that maps telemetry-only values to auto-merge authority fails activation with `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` or the most specific resolver artifact conflict code.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `ResolverProfile` | `activation_controlled_artifact` | Sole production resolver behavior authority; must validate before candidate generation. |
| `IdentifierEvidenceClass` row set | stable default semantics plus activation-controlled extensions | Extensions must not allow weak evidence auto-merge unless stable semantics permit. |
| `IdentityEvidenceItem` | `runtime_state_record` | Materialized evidence input. |
| `IdentifierScope` row set | `activation_controlled_artifact` | Scope registry under stable canonicalization rules. |
| `CandidateGenerationProfile` | `activation_controlled_artifact` | Defines blocking keys, pair ordering, caps, overflow, and learned-artifact policy. |
| `IdentityHardBlockerRowSet` | `activation_controlled_artifact` | Instantiates all hard blocker families and precedence rows; must be total for the active precedence table. |
| `AssetGenerationBoundary` row set | `activation_controlled_artifact` | Instantiates lifecycle and generation boundary rows only; it must not substitute for the hard blocker row set. |
| `ResolverDecisionMatrix` row set | `activation_controlled_artifact` | Instantiates the closed decision conditions without redefining decision semantics. |
| `IdentityConfidenceBand` row set | `activation_controlled_artifact` | Instantiates the closed confidence-band table. |
| `IdentityReviewRoutingPolicy` | `activation_controlled_artifact` | Routes review only under closed review-state semantics. |
| `SplitPolicy` | `activation_controlled_artifact` | Instantiates split partitioning and graph handoff behavior. |
| `ResolverExplanationPolicy` | `activation_controlled_artifact` | Selects included/excluded explanation checksum fields from this spec. |
| `IdentityDecision` | `runtime_state_record` | System-of-record output governed by resolver profile. |
| `IdentityReviewCase` | `runtime_state_record` | Review state; mutation only through terminal identity decisions. |
| `ResolverActivationReport` | `runtime_state_record` | Validation/activation state record. |
| `ResolverShadowRun` | `runtime_state_record` | Non-mutating comparison state. |
| `ResolverExplanation` | `runtime_state_record` | Runtime explanation output. |
| `GraphCorrectionHandoff` | `runtime_state_record` | Handoff record; resolver does not mutate graph. |
| `UnresolvedTargetReference` | `runtime_state_record` | Runtime hint record; no identity by itself. |
| `TargetSelectorSafetyPolicy` | `activation_controlled_artifact` | Must validate before selectors influence resolver or projection behavior. |
| `ResolveIdentity` | `stable_core_contract` | Algorithm validates artifact refs before candidate generation. |

### MVP Resolver Catalog Closure Requirements

The MVP resolver activation catalog is not a separate runtime authority. It is the concrete set of activation artifacts selected by exactly one active `ResolverProfileRow`. A production resolver run must resolve every catalog member below by exact `030.ActivationControlledArtifactRef`, include every selected ref and checksum in `030.VersionManifest`, and fail before identity mutation when a required member is missing, inactive, ambiguous, checksum-mismatched, scope-mismatched, unvalidated, or package-set mismatched.

| Catalog member | Artifact or runtime class | Omission behavior |
| --- | --- | --- |
| resolver profile row set | `resolver_profile` | `RESOLVER_PROFILE_ROW_MISSING` |
| evidence class row set | `identifier_evidence_class_row_set` | `RESOLVER_ARTIFACT_MISSING` |
| identifier scope row set | `identifier_scope_row_set` | `RESOLVER_ARTIFACT_MISSING` |
| candidate generation profile | `candidate_generation_profile` | `RESOLVER_ARTIFACT_MISSING` |
| hard blocker row set | `identity_hard_blocker_row_set` | `RESOLVER_HARD_BLOCKER_ROW_MISSING` |
| generation boundary row set | `asset_generation_boundary_row_set` | `RESOLVER_ARTIFACT_MISSING` |
| decision matrix row set | `resolver_decision_matrix_row_set` | `RESOLVER_DECISION_ROW_MISSING` or `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| confidence band row set | `identity_confidence_band_row_set` | `RESOLVER_CONFIDENCE_BAND_MISSING` |
| review routing policy | `identity_review_routing_policy` | `RESOLVER_REVIEW_ROUTING_MISSING` |
| split policy | `identity_split_policy` | `RESOLVER_SPLIT_POLICY_MISSING` |
| explanation policy | `resolver_explanation_policy` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| selector safety policy | `target_selector_safety_policy` | `TARGET_SELECTOR_UNSAFE` |
| activation report | `ResolverActivationReport` | production promotion forbidden; emit `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` when promotion requires the report |
| activation report policy | `resolver_activation_report_policy` | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` when the policy row set or validation refs are missing |
| shadow/canary output | `ResolverShadowRun` | required only when promotion uses shadow or canary; omission blocks promotion |

The catalog closure check must execute after profile row selection and before candidate generation. A catalog member must not be inferred from package type labels, owner prose, research reports, implementation defaults, or another artifact with a similar name.

### MVPResolverCatalogClosurePack

`MVPResolverCatalogClosurePack` is a promotion-time and resolver-preflight proof object. It is not runtime identity authority and must not substitute for any selected `ResolverProfileRow`, row-set artifact, policy artifact, validation row, package-set ref, or `030.VersionManifest` entry.

For each selected production resolver scope, the pack must contain active, checksum-valid, validation-backed, scope-valid, package-set-consistent when package-supplied, and manifest-included refs for every member below. Public resolver catalog bytes must be vendor-neutral. A catalog row, row-set envelope, validation output, activation report, package report, API output, audit output, or telemetry export that exposes private tenant inventories, directory tenant names, scanner sites, host lists, raw private fixture bytes, credentials, or private source binding values must fail before promotion and before identity output.

| Required catalog member | Required active catalog closure |
| --- | --- |
| `ResolverProfileRowSet` | Provide one active row for every supported `entity_type × run_mode` tuple: `host`, `user`, `service_account`, `group`, and `unsupported_entity_type`; `production`, `shadow`, `canary`, and `validation`. Wildcard run modes are forbidden. |
| `IdentifierEvidenceClassRowSet` | Provide exactly one active row for every evidence class named by `Evidence Roles`, `IdentifierEvidenceClass registry`, `Candidate Generation`, target selector safety, and `120` identity closure rows. The MVP registry includes durable, weak, selector, learned, graph-key, mapped-target, source-native-merge-history, semantic-overlay, group-membership, and telemetry-forbidden classes. |
| `IdentifierScopeRowSet` | Provide rows for cloud provider IDs, endpoint agent IDs, directory user IDs, directory group IDs, service account IDs, Kubernetes UIDs, IP addresses, DNS/PTR names, graph keys, mapped targets, learned hints, and telemetry-forbidden inputs. |
| `CandidateGenerationProfile` | Provide deterministic blocking-key ranks, canonical key strings, scope hash inputs, pair ordering, candidate caps, and overflow behavior. |
| `IdentityHardBlockerRowSet` | Provide exactly one most-specific active row for every hard-blocker family in the precedence table. |
| `AssetGenerationBoundaryRowSet` | Provide active rows for reimage, clone, VDI reuse, agent reinstall, provider delete/recreate, directory reenrollment, Kubernetes recreate, source rekey, hostname reuse, IP reuse, and scanner correlation change. |
| `ResolverDecisionMatrixRowSet` | Provide exactly one active row for each post-blocker decision condition: `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, and `no_decision`. |
| `IdentityConfidenceBandRowSet` | Provide the eight MVP rule-banded confidence rows with exact six-decimal canonical scores. |
| `IdentityReviewRoutingPolicy` | Provide review-opening, reviewer-authority, expiration, terminal-output, and illegal-transition rows. |
| `IdentitySplitPolicy` | Provide deterministic partition, retained-canonical selection, new-canonical creation, affected-fact selection, ambiguous-partition behavior, and handoff requirements. |
| `ResolverExplanationPolicy` | Provide included and excluded checksum-field rows for every externally visible `IdentityDecision`. |
| `TargetSelectorSafetyPolicyRowSet` | Provide one active row for every selector mechanism: `mapped_target`, `opengraph_node_id_equality`, `opengraph_property_matching`, `deprecated_name_matching`, `graph_key`, `hostname`, `ip_address`, `dns_name`, `ptr_name`, and `weak_endpoint_selector`. |
| `ResolverActivationReportPolicy` | Provide scenario gates for creation, attachment, durable merge, weak rejection, blocker precedence, overflow, review totality, split handoff, selector safety, explanation replay, and shadow/canary determinism. |

A selected resolver catalog member whose supporting artifact bytes, fixture checksum, expected-output checksum, package-set ref when package-supplied, or `VersionManifest` entry is missing must emit the most specific resolver artifact, package, manifest, validation, or activation-report error before candidate generation. Prose closure, registry labels, package type tokens, row-set checksums without selected row refs, validation summaries, and research reports are not substitutes for concrete selected refs and checksums.

The total resolver artifact coverage states are closed:

| Result state | Required meaning |
| --- | --- |
| `active_required` | The artifact must be active, checksum-valid, validation-backed, in scope, and manifest-included before identity output. |
| `deterministically_blocked` | The exact resolver scope is intentionally blocked and validation proves no identity mutation. |
| `validation_only` | The artifact may be used only in validation, shadow, or canary output and must not produce production identity mutations. |
| `not_in_scope_blocked` | The artifact is outside the selected resolver scope and must not be selected as fallback. |

Weak evidence defaults are total for MVP. Weak-only, selector-only, learned-only, source-native-merge-history-only, graph-key-only, mapped-target-only, OpenGraph property-match-only, deprecated-name-match-only, and telemetry-only inputs may emit only the active row behavior in the table below. They must not create `CanonicalEntity`, mutate `SourceAsset`, mutate `Identifier`, attach a source asset, merge canonical entities, split canonical entities, create a graph endpoint, or become gold-reference eligible by themselves. A non-mutating `IdentityDecision` with decision class `candidate`, `rejected`, or `no_decision` is allowed only when the active decision matrix row permits that class.

| Weak or non-authoritative input | Required active behavior | Forbidden production effect |
| --- | --- | --- |
| IP-only | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| Hostname-only | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| DNS-only or PTR-only | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| Scanner-name-only | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| Weak management name | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| Weak user name, mail, or UPN | `no_decision` or non-mutating `candidate`. | No identity mutation. |
| Weak group name | `no_decision` or non-mutating `candidate`. | No identity mutation and no membership fact by itself. |
| Group-membership hint | Correlation or review context only. | No identity mutation and no `identity_membership_fact.member_of` output by itself. |
| Graph-key-only | Selector only. | No identity mutation and no graph endpoint identity. |
| Mapped-target-only | `UnresolvedTargetReference` only. | No identity mutation. |
| OpenGraph property match | `UnresolvedTargetReference` only. | No identity mutation. |
| Deprecated name matching | Forbidden in production. | Fail with `DEPRECATED_NAME_MATCHING_FORBIDDEN`. |
| Source-native merge history | `lineage_only`; may block or route to review. | Never merge by itself. |
| Learned-only | `candidate_hint` only. | No identity mutation. |
| Telemetry-only | Forbidden as durable identity evidence. | Fail with `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` when used as identity evidence or scoring authority. |

### ResolverArtifactLifecycleGuardRows

Resolver activation artifacts use `030.ActivationControlledArtifactLifecycleMachine.v1`. `070` owns identity-specific guard rows and review-case state-machine requirements.

| Artifact class | Lifecycle binding | Required owner guards |
| --- | --- | --- |
| `ResolverProfile` | `030.ActivationControlledArtifactLifecycleMachine.v1` | Coverage for run mode, entity type, source scopes, evidence classes, lifecycle boundary types, activation scenarios, and blocker coverage. |
| `IdentifierEvidenceClass` row set | Generic artifact lifecycle | Stable weak-evidence defaults preserved; extensions cannot permit weak auto-merge unless stable semantics permit. |
| `IdentifierScope` row set | Generic artifact lifecycle | Scope keys, canonicalization, uncovered-scope behavior, and fixtures. |
| `CandidateGenerationProfile` | Generic artifact lifecycle | Blocking keys, deterministic pair ordering, candidate caps, overflow behavior, and learned-artifact fixtures. |
| `IdentityHardBlockerRowSet` | Generic artifact lifecycle | All hard blocker families, precedence values, trigger conditions, override policies, explanation keys, fired fixtures, and not-fired fixtures. |
| `AssetGenerationBoundary` row set | Generic artifact lifecycle | Reimage, clone, VDI, reinstall, delete/recreate, scanner correlation, and lifecycle generation-boundary fixtures only. |
| `ResolverDecisionMatrix` row set | Generic artifact lifecycle | Creation, attachment, durable merge, candidate, rejection, split, conflict, and no-decision fixtures. |
| `IdentityConfidenceBand` row set | Generic artifact lifecycle | Closed band values, decimal canonicalization, and replay fixtures. |
| `IdentityReviewRoutingPolicy` | Generic artifact lifecycle | Review totality, reviewer authority, and snapshot checksum fixtures. |
| `SplitPolicy` | Generic artifact lifecycle | Split partitioning, ambiguous partition handling, and graph handoff fixtures. |
| `ResolverExplanationPolicy` | Generic artifact lifecycle | Explanation checksum inclusion/exclusion and replay fixtures. |
| `TargetSelectorSafetyPolicy` | Generic artifact lifecycle | Selector maximum resolution state and forbidden identity side effects. |

### IdentityReviewCaseStateMachineBinding

Machine ID: `070.IdentityReviewCaseStateMachine.v1`.

`IdentityReviewCase` uses an owner-local deterministic state machine rather than shared `LifecycleStatus`. Review state changes must emit owner-local transition evidence and must not mutate canonical identity except through a terminal `IdentityDecision` record.

The machine states are `opened`, `in_review`, `blocked`, `expired`, `cancelled`, and `terminal_decision_emitted`. The machine events are `assign`, `approve_merge`, `reject_merge`, `approve_split`, `request_more_evidence`, `evidence_supplied`, `expire`, `cancel`, and `replay_same_event`. The total transition matrix in `IdentityReviewCase state machine` is closed for this machine version. Any state/event pair not explicitly allowed by that matrix must emit `IDENTITY_REVIEW_TRANSITION_INVALID`, preserve the current state, write no identity mutation, and produce deterministic transition evidence.

## Evidence Roles

Stable evidence defaults are closed by this spec. A `ResolverProfileRow` may narrow authority or route to review, but it must not grant create, attach, or merge authority to a class whose stable default forbids it.

| Evidence class default | Creation authority | Attachment authority | Merge authority | Required exact scope | Default role | Forbidden use |
| --- | --- | --- | --- | --- | --- | --- |
| `durable_provider_id` | May create only under exact provider plus account/project/subscription scope, region when applicable, compatible generation, exact normalized value, and no blocker. | May attach under the same exact scope and no blocker. | May merge only when the durable ID bundle proves the same source object under exact scope and no blocker. | provider, account/project/subscription, region when applicable, generation key when available | `positive_evidence` | Global identity outside the exact scope. |
| `endpoint_agent_id` | May create under exact enrollment scope and no blocker. | May attach under exact enrollment scope and no blocker. | May merge only when the same durable ID is observed in the exact enrollment scope or when a qualifying durable provider ID bundle also matches. | enrollment tenant, deployment, agent namespace, generation when available | `positive_evidence` | Merge across reinstall, clone, or VDI boundary without qualifying durable provider evidence. |
| `directory_user_id` | May create for `user` under exact directory tenant/domain/source scope and no blocker. | May attach under exact directory tenant/domain/source scope and no blocker. | May merge only under exact durable directory identity and no delete/recreate or reenrollment blocker. | directory tenant/domain/source and immutable user identifier | `positive_evidence` | Name, mail, UPN, or display-label merge by itself. |
| `service_account_durable_id` | May create for `service_account` under exact provider/account/tenant scope and no blocker. | May attach under exact provider/account/tenant scope and no blocker. | May merge only under exact durable provider or directory ID and no rekey/delete blocker. | provider/account/tenant or directory tenant plus service account ID | `positive_evidence` | Secret name, display name, or role-name merge by itself. |
| `kubernetes_uid` | Must not create Cadastre canonical host identity by default. | May attach only as source-object identity when scope is cluster, namespace, resource kind, UID, and generation. | Never by default. | cluster, namespace, resource kind, UID, generation when available | `source_object_identity` | Name-based workload or host identity merge. |
| `ip_address` | Never. | Never. | Never. | address family, observed interval, source scope | `candidate_hint` | Any create, attach, or merge. |
| `hostname` | Never. | Never. | Never. | source scope and observed interval | `candidate_hint` | Any create, attach, or merge. |
| `dns_or_ptr` | Never. | Never. | Never. | zone/source scope and observed interval | `candidate_hint` | Any create, attach, or merge. |
| `flow_id` | Never. | Never. | Never. | sensor scope and observation interval | `correlation_hint` | Identity creation or relationship endpoint merge. |
| `scanner_name` | Never. | Never. | Never. | scanner scope and scan interval | `candidate_hint` | Asset identity merge. |
| `weak_management_name` | Never. | Never. | Never. | management-plane source scope and observation interval | `candidate_hint` | Any create, attach, merge, split, or gold-reference eligibility by itself. |
| `weak_user_name_mail_upn` | Never. | Never. | Never. | directory tenant/domain/source and observation interval | `candidate_hint` | Any create, attach, merge, split, or gold-reference eligibility by itself. |
| `group_membership_hint` | Never. | Never. | Never. | directory tenant/domain/source, group scope, and observation interval | `correlation_hint` | Any create, attach, merge, split, or gold-reference eligibility by itself. |
| `weak_service_account_display_name` | Never. | Never. | Never. | provider/account/tenant or directory tenant plus observation interval | `candidate_hint` | Any create, attach, merge, split, or gold-reference eligibility by itself. |
| `source_native_merge_history` | Never. | Never. | Never. | source scope and source history ref | `lineage_only` | Cadastre identity authority by itself. |
| `semantic_overlay` | Never. | Never. | Never. | mapping artifact scope | `lineage_only` | Identity evidence. |
| `mapped_target` | Never. | Never. | Never. | selector source scope | `selector` | Identity evidence. |
| `graph_key` | Never. | Never. | Never. | graph backend profile scope | `selector` | Identity evidence. |

## Candidate Generation

`CandidateGenerationProfile` must define blocking keys, allowed heuristics, prohibited selectors, deterministic pair ordering, candidate caps, overflow behavior, and learned-artifact policy. Learned candidate generation may propose candidates only when `learned_artifact_policy != disabled`; it must not override hard blockers, lifecycle boundaries, decision matrix rows, confidence bands, review routing, or split policy.

Candidate generation must materialize blocking keys before candidate pairs. A blocking key whose required scope input is absent must be rejected as under-scoped and must not produce candidates. A blocking key whose member count exceeds `max_members_per_blocking_key` must emit `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` and must not produce auto-merge output from that block.

| Block kind rank | Blocking key class | Canonical key string | Scope hash inputs | Generation handling | Default output when key is weak or selector-only |
| ---: | --- | --- | --- | --- | --- |
| 10 | `durable_provider_id` | `provider_id:<provider>:<normalized_value>` | provider, account/project/subscription, region when applicable | append `generation:<asset_generation_key>` when present; missing generation remains eligible only if no generation blocker fires | may create, attach, or merge only through decision matrix |
| 20 | `endpoint_agent_id` | `endpoint_agent_id:<namespace>:<normalized_value>` | enrollment tenant, deployment, agent namespace | append generation when present; reinstall blocker wins before confidence | may create or attach; merge requires same enrollment scope or provider durable bundle |
| 30 | `directory_user_id` | `directory_user_id:<directory_namespace>:<normalized_value>` | directory tenant/domain/source | delete/recreate or reenrollment blocker wins | may create, attach, or merge only for `user` |
| 35 | `directory_group_id` | `directory_group_id:<directory_namespace>:<normalized_value>` | directory tenant/domain/source plus immutable group ID | delete/recreate or reenrollment blocker wins | may create, attach, or merge only for `group` through the decision matrix |
| 40 | `service_account_durable_id` | `service_account_id:<namespace>:<normalized_value>` | provider/account/tenant or directory tenant | rekey/delete blocker wins | may create, attach, or merge only for `service_account` |
| 50 | `kubernetes_uid` | `k8s_uid:<normalized_uid>` | cluster, namespace, resource kind | UID plus generation identifies source object only by default | source-object identity only; no canonical merge by default |
| 60 | weak hostname | `hostname:<nfc_lowercase_hostname>` | source scope and observed interval | hostname reuse blocker wins | candidate or no_decision only |
| 70 | weak IP | `ip:<address_family>:<canonical_ip>` | source scope and observed interval | IP reuse blocker wins | candidate or no_decision only |
| 80 | weak DNS/PTR | `dns:<canonical_fqdn>` or `ptr:<canonical_ptr>` | zone/source scope and observed interval | conflicting resolution blocker wins | candidate or no_decision only |
| 82 | weak scanner name | `scanner_name:<nfc_lowercase_name>` | scanner scope and scan interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; no cross-scanner block merge | candidate or no_decision only |
| 84 | weak management name | `management_name:<nfc_lowercase_name>` | management-plane source scope and observation interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; management names are weak hints only | candidate or no_decision only |
| 86 | weak user name/mail/UPN | `weak_user:<kind>:<nfc_lowercase_value>` where `kind` is `name`, `mail`, or `upn` | directory tenant/domain/source and observation interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; name/mail/UPN evidence must not create, attach, or merge by itself | candidate or no_decision only |
| 87 | `weak_group_name` | `weak_group:<nfc_lowercase_name>` | directory tenant/domain/source and observation interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; weak group names must not create, attach, merge, split, or become gold-reference eligible by themselves | candidate or no_decision only |
| 88 | group membership hint | `group_membership:<nfc_group_ref>:<nfc_member_hint>` | directory tenant/domain/source, group scope, and observation interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; group membership supplies review context only | correlation context only |
| 89 | weak service-account display name | `service_account_display_name:<nfc_lowercase_name>` | provider/account/tenant or directory tenant plus observation interval | scope omission emits `IDENTITY_EVIDENCE_UNDER_SCOPED`; display names are weak hints only | candidate or no_decision only |
| 90 | selector-only | `selector:<mechanism>:<normalized_value>` | selector source scope | no generation authority | unresolved target or no_decision only |

`scope_hash` must be `sha256` over `040.CanonicalJSON` of the ordered scope inputs declared for the block kind. Scope omission for any blocking key in this table must emit `IDENTITY_EVIDENCE_UNDER_SCOPED` and must produce no candidate pairs for that key. Candidate pair ordering must sort by block kind rank, blocking key UTF-8 bytes, lexical left source asset ID, lexical right source asset ID, and evidence item set checksum. The left source asset ID is the lexically smaller source asset ID; ties use lexical evidence item ID order.

## Hard Blocker Precedence

Hard blockers and lifecycle generation boundaries must be evaluated before confidence computation, review approval, and any decision matrix row that can create, attach, or merge. A fired blocker must persist in the `ResolverExplanation`; confidence bands and reviewer notes must not override it.

| Precedence | Blocker family | Default effect | Override allowance |
| ---: | --- | --- | --- |
| 1 | Scope mismatch or under-scoped evidence | Reject create, attach, merge, and split partitioning from the affected evidence. | Never. |
| 2 | Prior split or revoked merge | Block re-merge across the split boundary unless the split policy emits a new terminal merge decision with new durable evidence after the split known time. | Never for replay of the old event. |
| 3 | Generation boundary | Block auto-merge across reimage, clone, VDI reuse, agent reinstall, provider delete/recreate, directory reenrollment, Kubernetes recreate, or source rekey. | Only a split-aware continuation row may attach source assets without merging canonical entities. |
| 4 | Conflicting durable IDs | Emit `conflicted` or review; do not create, attach, or merge until conflict is resolved by durable evidence. | Never for weak or learned evidence. |
| 5 | Weak identifier reuse | Reject weak-only create, attach, and merge; weak values may remain candidate hints. | Never. |
| 6 | Source-native merge contradiction | Emit blocker or review; source-native merge history remains lineage only. | Never as standalone authority. |
| 7 | Candidate overflow | Emit deterministic overflow error or diagnostic review; no overflowed candidate may auto-merge. | Never inside the overflowed run. |

### IdentityHardBlockerRow schema

`IdentityHardBlockerRow` is the row-level interface for general hard blockers. `IdentityHardBlockerRowSet` is an activation-controlled artifact with `artifact_class = identity_hard_blocker_row_set`. It is separate from `AssetGenerationBoundaryRowSet`; generation boundary is one blocker family, not the full blocker catalog.

| Field | Required behavior |
| --- | --- |
| `row_id` | Stable row ID scoped to the hard blocker row set. |
| `blocker_family` | One closed value from the hard blocker precedence table and hard blocker matrix. |
| `precedence` | Required integer that must match the blocker precedence table. |
| `trigger_condition` | Exact Boolean condition over evidence roles, identifier scope, source scope, candidate state, prior decisions, and profile policy. Prose-only trigger conditions are invalid. |
| `blocked_decision_classes` | Closed subset of identity decision states blocked when the row fires. |
| `override_policy` | Closed enum: `never`, `split_aware_attach_only`, or `diagnostic_review_only`. |
| `explanation_field_key` | Required key included in `ResolverExplanation` when the row is evaluated. |
| `validation_refs` | Non-empty refs to fired and not-fired blocker fixtures. |
| `activation_scope` | Vendor-neutral scope in which the row may affect output. |
| `lifecycle_status` | Production use requires `active`. |

The active hard blocker row set must contain exactly one most-specific active row for every blocker family in the precedence table. Missing coverage emits `RESOLVER_HARD_BLOCKER_ROW_MISSING`. More than one equally specific active blocker row for the same blocker event emits `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS`. Either error blocks confidence selection, review routing, decision matrix selection, identity mutation, and graph correction handoff.

## ResolveIdentity Algorithm

```text
ResolveIdentity(evidence_items, resolver_profile, source_authority_context, version_manifest):
1. Validate `ResolverProfile`, `ResolverProfileRow`, `IdentifierEvidenceClass`, `IdentifierScope`, `CandidateGenerationProfile`, `IdentityHardBlockerRowSet`, `AssetGenerationBoundary`, `ResolverDecisionMatrix`, `IdentityConfidenceBand`, `IdentityReviewRoutingPolicy`, `SplitPolicy`, `ResolverExplanationPolicy`, and `TargetSelectorSafetyPolicy` refs through `030.ActivationControlledArtifactRef`.
2. Fail before candidate generation when any required row or artifact is inactive, missing, ambiguous, checksum-mismatched, out of scope, superseded, or unvalidated.
3. Resolve exactly one active `ResolverProfileRow` for run mode, entity type, evidence classes, lifecycle boundary types, source scope, and activation scope by applying owner non-scope predicates and then calling `030.ResolveScopedRow` over `source_scope_selector` and `activation_scope`.
4. Normalize identifiers into `IdentifierScopeSelectorContext`-aware `IdentityEvidenceItem` rows and materialize defaults before checksum computation.
5. Reject uncovered or under-scoped evidence with the most specific resolver error.
6. Generate blocking keys from the canonical blocking-key table; reject under-scoped keys before candidate pair materialization.
7. For each blocking key, sort members lexically by source asset ID and evidence item checksum, generate unordered candidate pairs, and stop at the first overflow bound crossed.
8. Sort candidate pairs by block kind rank, blocking key UTF-8 bytes, lexical left source asset ID, lexical right source asset ID, and evidence item set checksum.
9. Enforce per-source-asset and resolver-partition candidate caps after sorting; overflow emits the overflow code and no mutation for overflowed candidates.
10. Evaluate hard blockers and lifecycle generation boundaries in precedence order before confidence selection.
11. Select exactly one resolver decision matrix row per candidate after blocker evaluation.
12. Select exactly one deterministic confidence band from `IdentityConfidenceBand`; persist the canonical six-digit decimal score from `040.DecimalPrecisionPolicy.confidence_0_1`.
13. Apply review routing. Review output may open `IdentityReviewCase`, but no review action may create, attach, merge, or split except through a terminal `IdentityDecision`.
14. Emit exactly one `IdentityDecision` per candidate: `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, or `no_decision`.
15. Persist exactly one `ResolverExplanation` per `IdentityDecision` using the explanation checksum fields in this spec.
16. Emit `GraphCorrectionHandoff` for every split that affects prior gold facts or graph output; the resolver must not mutate graph state.
17. Include every output-affecting resolver artifact ref, runtime decision ref, explanation checksum, review case ref, and graph correction handoff ref in `VersionManifest`.
```

## Review Contract

Manual review must not mutate canonical identity directly. `IdentityReviewCase` must define closed states, reviewer authority, evidence snapshot checksums, transitions, expiration, terminal decision outputs, and illegal-transition behavior. Identity mutation may occur only through terminal `IdentityDecision` records.

## Target Selector Safety

`UnresolvedTargetReference` represents relationship hints without creating or merging canonical entities. `TargetSelectorSafetyPolicy` must define selector-specific maximum resolution state and evidence requirements.

OpenGraph-style property matching, name matching, source-kind matching, environment-scoped matching, cross-source reference matching, or mapped-target matching must not create canonical identity without a qualifying identity decision. Deprecated name matching is forbidden in production.

### GraphEndpointIdentityHandoff

A graph endpoint may become `090.projected_canonical_node` only after a qualifying `IdentityDecision` creates or references a `CanonicalEntity` allowed by the active resolver profile. Graph projection must treat unresolved endpoint identity as a projection blocker, not as an identity mutation request.

| Endpoint input class | MVP graph endpoint identity behavior | Required `090` handoff |
| --- | --- | --- |
| `CanonicalEntity` ref produced or referenced by qualifying `IdentityDecision` | Eligible for `projected_canonical_node` when the active graph profile permits the node type. | Include canonical entity ref and selected identity decision ref in graph projection inputs. |
| `UnresolvedTargetReference` | Hint only; no endpoint identity. | Emit no endpoint node and block dependent edge with `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. |
| Mapped target reference, graph key, OpenGraph property match, hostname/IP/DNS/PTR selector-only evidence, learned candidate, or source-native merge history | No canonical graph endpoint identity in MVP. | Emit no endpoint node unless a later resolver run emits a qualifying identity decision. |
| Identity split handoff | Resolver emits `GraphCorrectionHandoff` metadata only. | `090` routes affected facts through projection; resolver must not mutate graph state. |

Handoff records consumed by graph projection must include endpoint canonical refs, selected identity decision refs, and resolver explanation checksum refs when those fields affect projection, replay, or audit. Omitted handoff fields block graph delta persistence with the most specific `070`, `090`, or manifest error.

### GoldFactReferenceEligibilityHandoff

A `CanonicalEntity`, `SourceAsset`, or `Identifier` may appear in `GoldFact.subject_ref` or `GoldFact.object_value` only when the referenced record has passed its `040` schema, the producing or attaching `IdentityDecision` is terminal or otherwise permitted by the active `ResolverProfileRow`, and the selected `080.GoldFactPredicateContractRow` permits that reference kind.

`IdentityEvidenceItem` is not a valid `GoldFact.subject_ref` or `GoldFact.object_value`. `UnresolvedTargetReference` is not a valid subject or object reference. `TargetSelectorSafetyPolicy` can restrict selector influence but cannot create a subject or object reference.

IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-history-only, mapped-target-only, learned-only, telemetry-only, and selector-only evidence must not become `canonical_entity_ref`, `source_asset_ref`, or `identifier_ref` by themselves. A package-supplied resolver artifact that attempts to weaken this rule fails activation before identity or gold output.

`Identifier` reference eligibility requires typed scope and validated `IdentifierScope`. `SourceAsset` reference eligibility requires sufficient `source_scope`, `source_native_identity`, and `source_asset_type`. `CanonicalEntity` reference eligibility requires a qualifying identity decision and resolver explanation ref when those fields affect replay, projection, or audit.

`identity_membership_fact.member_of` requires qualifying identity decisions for both the member ref and the group ref. The group ref must resolve through the `group` row in `ResolverProfileCoverageMatrix`; ad hoc group strings, weak group names, directory display names, graph keys, and membership hints are not gold-reference eligible by themselves.

## Identity Resolution Contract Details

### ResolverProfileCoverageMatrix

| Entity type | Run mode | Source scope coverage | Evidence classes | Lifecycle boundary classes | Allowed decision outputs | Validation scenario families |
| --- | --- | --- | --- | --- | --- | --- |
| `host` | `production`, `shadow`, `canary`, `validation` | provider/account/project/subscription, region when applicable, endpoint enrollment scope, scanner/feed scope, cluster scope when present | `durable_provider_id`, `endpoint_agent_id`, `kubernetes_uid`, `hostname`, `ip_address`, `dns_or_ptr`, `scanner_name`, `weak_management_name`, `graph_key`, `source_native_merge_history` | reimage, clone, VDI reuse, agent reinstall, provider delete/recreate, hostname reuse, IP reuse, Kubernetes recreate, scanner correlation change | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-weak-rejection`, `identity-hard-blocker`, `identity-candidate-overflow`, `identity-split-handoff`, `identity-explanation-checksum` |
| `user` | `production`, `shadow`, `canary`, `validation` | directory tenant/domain/source, provider tenant when federated | `directory_user_id`, `durable_provider_id`, `weak_user_name_mail_upn`, `group_membership_hint`, `graph_key`, `source_native_merge_history` | delete/recreate, reenrollment, hidden membership or permission-limited evidence | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-review-state-machine`, `identity-selector-safety`, `identity-replay` |
| `service_account` | `production`, `shadow`, `canary`, `validation` | provider/account/tenant, directory tenant, workload namespace when applicable | `service_account_durable_id`, `durable_provider_id`, `directory_user_id`, `weak_service_account_display_name`, `graph_key`, `source_native_merge_history` | rekey, delete/recreate, directory reenrollment, namespace recreation | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-hard-blocker`, `identity-confidence-band`, `identity-package-artifact-core-conflict` |
| `group` | `production`, `shadow`, `canary`, `validation` | directory tenant/domain/source | `directory_group_id`, `weak_group_name`, `group_membership_hint`, `graph_key`, `source_native_merge_history` | delete/recreate, reenrollment, hidden membership or permission-limited evidence | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-group-create`, `identity-group-durable-merge`, `identity-group-weak-rejection`, `identity-membership-hidden-permission`, `identity-member-of-fact` |
| `unsupported_entity_type` | any | any | any | any | `no_decision` only | `identity-unsupported-entity-type` |

The `unsupported_entity_type` row is closed behavior. It must emit `RESOLVER_ENTITY_TYPE_UNSUPPORTED`, must emit `no_decision`, must not open review, and must not mutate `CanonicalEntity`, `SourceAsset`, or `Identifier`.

### IdentifierScopeSelectorContext

`IdentifierScopeSelectorContext` converts identity source-scope and identifier-scope matching to `030.ScopeSelectorContext` rows. It does not define selector schema, equality, coverage, specificity, subset rules, or ambiguity behavior.

| Source class | Identifier class | Required selector dimensions | Optional dimensions | Subset behavior | Uncovered-scope behavior |
| --- | --- | --- | --- | --- | --- |
| cloud provider | provider resource ID | `provider`, `account_or_project_or_subscription` | `region`, `generation` | none | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates. |
| endpoint agent | agent durable ID | `enrollment_tenant`, `deployment`, `agent_namespace` | `generation` | none | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates. |
| directory | user or service account durable ID | `directory_tenant_or_domain`, `directory_source`, `immutable_object_id` | none | none | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates. |
| directory | group durable ID | `directory_tenant_or_domain`, `directory_source`, `immutable_group_id` | none | none | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates. |
| Kubernetes | UID | `cluster`, `namespace`, `resource_kind`, `uid` | `generation` | none | source-object identity only by default. |
| network | `ip_address` | `address_family`, `observed_interval`, `source_scope` | none | none | weak evidence only. |
| DNS | name/PTR | `zone_or_source_scope`, `observed_interval` | none | none | weak evidence only. |
| graph | backend key | `backend_profile`, `graph_object_scope` | none | none | selector only; no identity. |

Identifier normalization remains owner-local after selector validation: provider tokens are lowercased where the existing identity evidence rule requires it, exact durable IDs remain exact after NFC, FQDN normalization remains owner-local, and IP textual normalization remains owner-local. Selector context validation occurs before candidate generation.

Shared selector errors map to resolver errors as follows.

| Shared selector error | Resolver error |
| --- | --- |
| `SCOPE_REQUEST_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `SCOPE_SELECTOR_UNSUPPORTED_DIMENSION` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `SCOPE_SUBSET_NOT_ALLOWED` | `RESOLVER_SCOPE_SUBSET_MUTATION_FORBIDDEN` |
| `SCOPE_SELECTOR_AMBIGUOUS` | `RESOLVER_PROFILE_ROW_AMBIGUOUS` |
| `SCOPE_SELECTOR_PRIVATE_BINDING_LEAK` | `RESOLVER_REPOSITORY_PRIVATE_BINDING_LEAK` or `PRIVATE_BINDING_LEAK` by publication context |

### IdentifierScope canonicalization

| Source class | Identifier class | Required scope keys | Normalization | Uncovered-scope behavior |
| --- | --- | --- | --- | --- |
| cloud provider | provider resource ID | provider, account/project/subscription, region when applicable, generation when available | provider lowercased; ID value exact after NFC; region lowercased | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| endpoint agent | agent durable ID | enrollment tenant, deployment, agent namespace, generation when available | exact ID string after NFC | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| directory | user or service account durable ID | directory tenant/domain/source and immutable object ID | exact ID string after NFC; display names excluded | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| directory | group durable ID | directory tenant/domain/source and immutable group ID | exact ID string after NFC; display names excluded | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| Kubernetes | UID | cluster, namespace, resource kind, UID, generation when available | exact UID after NFC | source-object identity only by default |
| network | `ip_address` | address family, observed interval, source scope | canonical textual IP | weak evidence only |
| DNS | name/PTR | zone/source scope, observed interval | lowercase FQDN with trailing-dot normalization | weak evidence only |
| graph | backend key | backend profile and graph object scope | exact string after NFC | selector only; no identity |

### IdentifierEvidenceClass registry

| Evidence class | Durability | Scope | Reuse risk | Default role | Creation/attachment/merge authority | Negative evidence effect | Review routing |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `durable_provider_id` | high | exact provider/account/project/subscription, region when applicable | medium after delete/recreate | `positive_evidence` | create, attach, and merge only under exact scope, compatible generation, exact normalized value, and no blocker | generation or conflict blocker wins | no review unless conflict or split |
| `endpoint_agent_id` | high | exact enrollment tenant/deployment/namespace | medium after reinstall, clone, VDI | `positive_evidence` | create or attach; merge only under exact enrollment scope or qualifying durable provider bundle | reinstall/clone/VDI blocker wins | review on conflict |
| `directory_user_id` | high | exact directory tenant/domain/source | medium after delete/recreate or reenrollment | `positive_evidence` | create, attach, or merge for `user` only under exact durable ID and no blocker | reenrollment/delete blocker wins | review on conflict or hidden-membership gap |
| `directory_group_id` | high | exact directory tenant/domain/source | medium after delete/recreate or reenrollment | `positive_evidence` | create, attach, or merge for `group` only under exact durable ID and no blocker | reenrollment/delete blocker wins | review on conflict or hidden-membership gap |
| `service_account_durable_id` | high | exact provider/account/tenant or directory tenant | medium after rekey/delete | `positive_evidence` | create, attach, or merge for `service_account` only under exact durable ID and no blocker | rekey/delete blocker wins | review on conflict |
| `kubernetes_uid` | high for source object, not canonical host | exact cluster/namespace/resource/generation | medium after recreate | `source_object_identity` | no canonical merge by default | recreate blocker wins | no_decision unless profile opens review |
| `hostname` | low | source/time scoped | high | `candidate_hint` | never | reuse blocks weak merge | review candidate only |
| `ip_address` | low | source/time scoped | high | `candidate_hint` | never | reuse blocks weak merge | review candidate only |
| `dns_or_ptr` | low | zone/time scoped | high | `candidate_hint` | never | conflicting resolution blocks weak merge | review candidate only |
| `flow_id` | low | sensor scoped | high | `correlation_hint` | never | none by itself | no_decision |
| `scanner_name` | low | scanner scope and scan interval | high | `candidate_hint` | never | weak reuse may block | candidate review only when policy permits |
| `weak_management_name` | low | management-plane source/time scoped | high | `candidate_hint` | never | weak reuse may block | candidate review only when policy permits |
| `weak_user_name_mail_upn` | low | directory tenant/domain/source and observation interval | high | `candidate_hint` | never | weak reuse may block | candidate review only when policy permits |
| `weak_group_name` | low | directory tenant/domain/source and observation interval | high | `candidate_hint` | never | weak reuse may block | candidate review only when policy permits; never by itself for membership facts |
| `group_membership_hint` | low | directory tenant/domain/source, group scope, and observation interval | high | `correlation_hint` | never | none by itself | review context only |
| `weak_service_account_display_name` | low | provider/account/tenant or directory tenant plus observation interval | high | `candidate_hint` | never | weak reuse may block | candidate review only when policy permits |
| `graph_key` | low | backend scoped | high | `selector` | never | none by itself | no_decision |
| `mapped_target` | low | selector source scoped | high | `selector` | never | none by itself | no_decision |
| `source_native_merge_history` | medium | source scoped | medium | `lineage_only` | never | contradiction may block | review candidate only |
| `semantic_overlay` | low | mapping artifact scoped | high | `lineage_only` | never | none by itself | no_decision |
| `learned_hint` | model-artifact scoped | resolver scope | model drift risk | `candidate_hint` | never | none by itself | review candidate only |

No row in this registry may create, attach, merge, split, or become gold-reference eligible by itself when its `Creation/attachment/merge authority` is `never`. A resolver artifact that weakens one of these stable defaults fails activation before production identity output.

### Resolver decision matrix

`ResolverDecisionMatrix` row sets are activation-controlled artifacts with `artifact_class = resolver_decision_matrix_row_set`. Stable decision states remain closed by this spec. The resolver must select exactly one active decision row after hard blocker evaluation. If no row matches, it must emit `RESOLVER_DECISION_ROW_MISSING`. If more than one equally specific active row matches the same candidate after blocker evaluation, it must emit `RESOLVER_DECISION_ROW_AMBIGUOUS`. Either error blocks identity mutation.

| Field | Required behavior |
| --- | --- |
| `decision_row_id` | Stable row ID scoped to the decision matrix row set. |
| `decision_class` | One closed identity decision state. |
| `decision_priority` | Required integer used only after row specificity has selected candidate rows; lower numeric value wins only when the active row set declares priority ordering for non-equally-specific rows. |
| `required_condition` | Exact Boolean condition over evidence roles, hard blockers, generation boundaries, source scope, identifier scope, candidate state, profile policy, and review state. |
| `required_confidence_band` | One band from the active `IdentityConfidenceBandRow`. |
| `terminality` | Closed enum: `terminal_mutating`, `terminal_non_mutating`, or `non_terminal_reviewable`. |
| `mutation_limit` | Exact allowed mutation classes: `none`, `create_canonical_entity`, `attach_source_asset`, `merge_canonical_entities`, `emit_split_handoff`, or a closed subset. |
| `forbidden_evidence_roles` | Closed evidence-role set that cannot satisfy the row. Weak, selector, lineage-only, correlation-only, source-object-only, telemetry-only, and learned-only roles must not satisfy mutating rows. |
| `validation_refs` | Non-empty positive, rejection, replay, and mutation-prohibition fixture refs. |

#### Resolver decision rows

| Decision | Required condition | Output behavior | Mutation limit |
| --- | --- | --- | --- |
| `canonical_created` | Durable evidence class permits creation; exact source and identifier scopes validate; no existing canonical candidate remains after sorted candidate evaluation; `creation_policy = durable_evidence_creation_allowed`; no blocker fires; confidence band is `single_durable_creation`. | Emit terminal `IdentityDecision` and create one `CanonicalEntity` plus required `SourceAsset`/`Identifier` records that pass `040`. | Must not merge with any existing canonical entity. |
| `source_asset_attached` | Exactly one existing canonical entity matches exact durable evidence under exact scope; no blocker fires; decision row permits attachment; confidence band is `exact_durable_attach`. | Emit terminal `IdentityDecision` and attach the source asset or identifier to that canonical entity. | Must not merge two existing canonical entities. |
| `auto_merged` | Two or more existing canonical entities have exact compatible durable evidence under exact scope; no blocker fires; decision row permits merge; confidence band is `exact_durable_merge`. | Emit terminal `IdentityDecision` linking merged canonical refs and supersession refs. | Must not use weak, selector-only, learned-only, or source-native merge-history-only evidence. |
| `candidate` | Evidence suggests a possible match, but stable authority is insufficient, review is required, evidence is weak-only, learned-only, or policy routes to review. | Emit non-mutating `IdentityDecision`; open review only when review routing permits. | No canonical mutation. |
| `rejected` | Candidate fails scope, evidence class, hard blocker, lifecycle, selector safety, or decision row condition. | Emit rejection with explanation. | No canonical mutation. |
| `split` | Prior merge is invalidated by split evidence, approved split review, or deterministic split policy; affected facts or graph projection may change. | Emit terminal split `IdentityDecision` and `GraphCorrectionHandoff`. | Resolver must not mutate graph. |
| `conflicted` | Conflicting durable evidence or equally specific decision rows prevent deterministic creation, attachment, merge, or split. | Emit conflicted decision and review case when routing permits. | No canonical mutation before terminal review decision. |
| `no_decision` | Evidence is uncovered, unsupported entity type, selector-only, out of profile scope, overflowed, or otherwise ineligible. | Emit no-decision with explanation or deterministic error. | No canonical mutation. |

### Deterministic confidence bands

`IdentityConfidenceBandRow` is the row-level interface for the active confidence band row set. Concrete row sets are activation-controlled artifacts with `artifact_class = identity_confidence_band_row_set`. Persisted scores must import `040.DecimalPrecisionPolicy.confidence_0_1` and must use canonical six-fractional-digit decimal strings. Binary floating point and rounded values are forbidden.

| Field | Required behavior |
| --- | --- |
| `band_id` | Stable band token. |
| `score` | Canonical decimal string from `0.000000` through `1.000000` inclusive. |
| `selection_condition` | Exact Boolean condition over blockers, evidence roles, durable evidence scope, candidate state, and decision row. |
| `permitted_decision_classes` | Closed subset of identity decision states. |
| `selection_order` | Required integer matching the order in this table. |
| `validation_refs` | Non-empty refs proving selection, non-selection, decimal canonicalization, and replay. |

MVP confidence is rule-banded, not probabilistic. Scores must be persisted as `040.DecimalPrecisionPolicy.confidence_0_1` values with six fractional digits and no rounding.

| Band | Score | Selection condition | Permitted decision classes |
| --- | ---: | --- | --- |
| `blocked` | `0.000000` | Any non-overridable hard blocker fired. | `rejected`, `no_decision`, `conflicted` |
| `selector_only` | `0.000000` | Only selector evidence exists. | `no_decision` |
| `weak_hint_only` | `0.250000` | Only weak hostname, IP, DNS, PTR, scanner, management name, user name/mail/UPN, group name, service-account display name, group-membership hint, or equivalent weak evidence exists. | `candidate`, `rejected`, `no_decision` |
| `learned_hint_only` | `0.300000` | Only learned or model-generated hint exists. | `candidate`, `rejected`, `no_decision` |
| `conflicting_durable` | `0.500000` | Durable evidence conflicts or equally specific rows disagree. | `conflicted`, `candidate`, `rejected` |
| `single_durable_creation` | `0.900000` | One exact durable evidence bundle under exact scope and no existing canonical candidate. | `canonical_created` |
| `exact_durable_attach` | `0.950000` | One exact durable evidence bundle attaches to exactly one existing canonical entity and no blocker fires. | `source_asset_attached` |
| `exact_durable_merge` | `1.000000` | Exact compatible durable evidence proves the same object across existing canonical entities and no blocker fires. | `auto_merged` |

If more than one band condition matches, the selected band must be the first row in this table whose selection condition is true.

### IdentityReviewCase state machine

Reviewer authority guards must execute before transition selection in this order: reviewer authorization, case lifecycle status, evidence snapshot checksum match, hard blocker result preservation, idempotency key match, and terminal-state check. A review event whose evidence snapshot checksum does not equal the case snapshot checksum must emit `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` and must not transition. A review event without required reviewer authority must emit `IDENTITY_REVIEW_AUTHORITY_MISSING` and must not transition.

| Current state | `assign` | `approve_merge` | `reject_merge` | `approve_split` | `request_more_evidence` | `evidence_supplied` | `expire` | `cancel` | `replay_same_event` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `opened` | `in_review` | illegal | `terminal_decision_emitted` | illegal | `blocked` | illegal | `expired` | `cancelled` | idempotent no-op |
| `in_review` | `in_review` | `terminal_decision_emitted` | `terminal_decision_emitted` | `terminal_decision_emitted` | `blocked` | illegal | `expired` | `cancelled` | idempotent no-op |
| `blocked` | illegal | illegal | `terminal_decision_emitted` | illegal | `blocked` | `in_review` | `expired` | `cancelled` | idempotent no-op |
| `expired` | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | idempotent no-op |
| `cancelled` | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | idempotent no-op |
| `terminal_decision_emitted` | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | terminal no-op | idempotent no-op |

Illegal transitions must emit `IDENTITY_REVIEW_TRANSITION_INVALID`, preserve current state, write no `IdentityDecision`, write no `CanonicalEntity`, write no `SourceAsset`, write no `Identifier`, and write no `GraphCorrectionHandoff`. Terminal no-op transitions must emit deterministic transition evidence and no mutation. `approve_merge` must be illegal when only weak, learned, selector-only, source-native-merge-history-only, mapped-target-only, or graph-key-only evidence exists.

### IdentityReviewCaseLifecycleClosure

`070` owns identity review states, events, transition rows, and terminal identity outputs. `030` owns generic lifecycle evidence fields and `ApplyLifecycleEvent`. `120` owns validation rows.

| Field | Required value |
| --- | --- |
| `machine_id` | `070.IdentityReviewCaseStateMachine.v1` |
| `generic_lifecycle_owner` | `030.LifecycleStateMachineDefinition` |
| `domain_closure_state` | `closed_local_pending_validation` |
| `required_validation_rows` | `val-070-identity-review-totality`, `val-070-identity-review-idempotency`, `val-070-identity-review-illegal-no-mutation`, `val-070-identity-review-terminal-decision`, `val-070-identity-review-evidence-snapshot` |
| `mutation_prohibition` | Review transitions must not mutate identity except through terminal `IdentityDecision` records. |
| `version_manifest_requirement` | Include machine ID, machine checksum, transition evidence refs, review-case refs, terminal decision refs, and resolver explanation refs whenever review affects output. |

### Review routing defaults

| Candidate condition | Default routing | Reviewer authority limit |
| --- | --- | --- |
| Weak-only candidate | May open review when policy permits. | Reviewer must not approve merge without new durable evidence materialized as `IdentityEvidenceItem`. |
| Learned-only candidate | May open review when policy permits. | Reviewer must not approve merge without new durable evidence and passing replay fixture. |
| Conflicting durable evidence | Must open review when profile permits review; otherwise emit `conflicted`. | Reviewer may approve only after conflict-resolving durable evidence snapshot matches the case. |
| Candidate overflow | May open diagnostic review only when profile permits. | Reviewer must not approve merge from an overflowed block; rerun under non-overflow conditions is required. |
| Hard blocker fired | Review may document blocker only. | Reviewer must not override non-overridable blockers. |
| Selector-only evidence | No review by default. | No creation, attachment, merge, or split authority. |

### IdentityReviewRoutingPolicy row schema

`IdentityReviewRoutingPolicy` is an activation-controlled artifact with `artifact_class = identity_review_routing_policy`. It governs review opening, expiration, and reviewer terminal authority; it must not redefine review state names or bypass terminal `IdentityDecision` output.

| Field | Required behavior |
| --- | --- |
| `policy_id` | Stable policy ID. |
| `case_expiration_seconds` | Default `1209600`; bounds `86400..7776000`. |
| `expiration_effect` | Default `expired_no_mutation`; expiration emits transition evidence and no identity mutation. |
| `weak_only_review_behavior` | Must not approve merge unless new durable evidence is materialized as `IdentityEvidenceItem` and replay fixtures pass. |
| `hard_blocker_review_behavior` | Review may document a blocker but must not override a non-overridable blocker. |
| `terminal_decision_requirement` | Identity mutation may occur only through terminal `IdentityDecision`. |
| `validation_refs` | Non-empty review-opening, weak-only, hard-blocker, expiration, terminal-decision, and illegal-transition fixtures. |

### SplitPolicy schema and deterministic split algorithm

`SplitPolicy` is an activation-controlled artifact with `artifact_class = identity_split_policy`. It must name trigger conditions, partition key rules, retained canonical partition selection, new canonical partition creation rules, ambiguous partition behavior, affected fact selection, and `GraphCorrectionHandoff` emission rules.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `split_policy_id` | Yes | none | Stable policy ID. |
| `trigger_decision_classes` | Yes | none | Non-empty subset containing `split`; review-approved split must emit terminal `split`. |
| `partition_key_rule` | Yes | none | Canonical rule over durable evidence scope, generation boundary, known time, and source asset refs. |
| `retained_canonical_partition_rule` | Yes | none | Deterministic rule; default is partition containing the oldest known active durable evidence, ties by lexical canonical entity ID. |
| `new_canonical_partition_rule` | Yes | none | Creates one canonical entity per non-retained partition only through `040.CanonicalEntitySchema`. |
| `ambiguous_partition_behavior` | No | `emit_conflicted_no_split` | Closed enum: `emit_conflicted_no_split`, `open_review`, `reject_split`. |
| `affected_fact_selection` | Yes | none | Exact fact refs or deterministic affected-fact selection checksum required. |
| `graph_correction_handoff_required` | Yes | `true` | Must be true when gold facts or graph output may be affected. |
| `validation_refs` | Yes | none | Split partition, ambiguous partition, handoff, replay, and no-direct-graph-mutation fixtures. |

Split execution must sort source assets by lexical source asset ID, compute each partition key using `040.CanonicalJSON`, select the retained partition, create new canonical partitions only for non-retained partitions, select affected facts using the policy rule, emit a `split` `IdentityDecision`, and emit `GraphCorrectionHandoff` when affected facts or graph output are non-empty. `GraphCorrectionHandoff` must include split decision ref, split policy ref, retained canonical entity ref, new canonical entity refs, split partition refs, affected fact refs or affected fact-selection checksum, resolver explanation checksum, and version manifest ref. The resolver must never mutate graph state.

### GraphCorrectionHandoff field schema

`GraphCorrectionHandoff` is the immutable identity-split-to-gold-and-graph handoff. It is a runtime state record. The resolver emits it only for a terminal `split` decision whose effects can change prior gold facts or graph projection output. `080` and `090` must consume these exact field names and must not reconstruct resolver split state.

| Field | Required behavior |
| --- | --- |
| `graph_correction_handoff_id` | Deterministic ID over split decision, split policy, partitions, affected facts or selection checksum, explanation checksum, and manifest ref. |
| `split_identity_decision_ref` | Required terminal `split` decision. |
| `split_policy_ref` | Required active `identity_split_policy`. |
| `retained_canonical_entity_ref` | Required canonical entity retained by the split policy. |
| `new_canonical_entity_refs` | Canonically sorted; non-empty when the split creates new canonical entities. |
| `split_partition_refs` | Canonically sorted refs or checksums for each split partition. |
| `affected_fact_refs` | Exact refs, or null only when `affected_fact_selection_checksum` is present. |
| `affected_fact_selection_checksum` | Required when exact affected fact refs are not enumerated. |
| `resolver_explanation_checksum` | Required checksum for the split decision explanation. |
| `version_manifest_ref` | Required `030.VersionManifest` ref. |
| `validation_refs` | Required split handoff fixtures. |

Missing, null-forbidden, non-canonical, checksum-mismatched, or schema-invalid handoff fields must block correction, projection, and replay with the most specific owner error. A split handoff must not contain graph backend IDs or private source binding values.

### Identity closed enum tables

| Enum family | Closed values |
| --- | --- |
| identity decision states | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` |
| review case states | `opened`, `in_review`, `blocked`, `expired`, `cancelled`, `terminal_decision_emitted` |
| review events | `assign`, `approve_merge`, `reject_merge`, `approve_split`, `request_more_evidence`, `evidence_supplied`, `expire`, `cancel`, `replay_same_event` |
| evidence roles | `positive_evidence`, `negative_evidence`, `candidate_hint`, `selector`, `lineage_only`, `correlation_hint`, `source_object_identity` |
| selector mechanisms | `mapped_target`, `opengraph_node_id_equality`, `opengraph_property_matching`, `deprecated_name_matching`, `graph_key`, `hostname`, `ip_address`, `dns_name`, `ptr_name`, `weak_endpoint_selector` |

### IdentityApiHandoff

Resolver and target-selector outcomes that surface through asset detail, graph query, evidence drillback, health, audit, or export must use structured owner context and the generated `110.ErrorCodeRegistry`.

| Identity condition | Recommended `110` label or outcome | Required behavior |
| --- | --- | --- |
| resolver profile missing | `error` | Emit `RESOLVER_PROFILE_MISSING`; no identity mutation. |
| resolver row ambiguous | `ambiguous` | Emit `RESOLVER_PROFILE_ROW_AMBIGUOUS`; no implementation-order selection. |
| selector-only unresolved target | diagnostic or no output | No identity, no graph endpoint, no canonical entity creation. |
| graph endpoint unresolved | imported graph owner error consumed by `110` | `090` emits `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; `070` emits no identity mutation and owns no generated registry row for this code. |
| review required | non-mutating review state | Preserve state until terminal `IdentityDecision`; no hidden identity mutation. |
| private source scope selector | redacted diagnostic | Caller-visible responses must not expose tenant inventories, scanner sites, directory inventories, zone inventories, account lists, raw payload values, or private bindings. |

### Resolver error codes

| Error code | Emitted when |
| --- | --- |
| `RESOLVER_ARTIFACT_MISSING` | A required resolver, candidate, scope, evidence-class, boundary, decision, confidence, review, split, explanation, or selector policy artifact ref is missing. |
| `RESOLVER_ARTIFACT_INACTIVE` | A required resolver artifact is not active for production execution. |
| `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | A resolver artifact checksum mismatches the active ref or manifest. |
| `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | A resolver artifact does not cover resolver run mode, entity type, source scope, evidence class, lifecycle boundary, or activation scope. |
| `RESOLVER_PROFILE_MISSING` | No active resolver profile covers run mode, entity type, source scopes, evidence classes, and lifecycle boundaries. |
| `RESOLVER_PROFILE_ROW_MISSING` | No active `ResolverProfileRow` covers the resolver run tuple. |
| `RESOLVER_PROFILE_ROW_AMBIGUOUS` | More than one equally specific active `ResolverProfileRow` covers the resolver run tuple. |
| `RESOLVER_SCOPE_SUBSET_MUTATION_FORBIDDEN` | A subset scope match attempts creation, attachment, merge, split, or another mutating decision. |
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | Entity type maps to the unsupported row. |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | Required scope keys are absent or non-canonical. |
| `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | Evidence class lacks a covering resolver profile row. |
| `IDENTITY_HARD_BLOCKER_TRIGGERED` | Hard blocker or generation boundary prevents create, attach, or merge. |
| `RESOLVER_HARD_BLOCKER_ROW_MISSING` | No active hard blocker row covers a required blocker family. |
| `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` | More than one equally specific active hard blocker row covers a blocker event. |
| `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | A blocking key exceeds `max_members_per_blocking_key`. |
| `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | Candidate pairs exceed per-source-asset or resolver-partition caps. |
| `RESOLVER_DECISION_ROW_MISSING` | No decision matrix row covers a candidate after blocker evaluation. |
| `RESOLVER_DECISION_ROW_AMBIGUOUS` | More than one equally specific active decision row covers a candidate after blocker evaluation. |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | No confidence band row covers the candidate decision state. |
| `RESOLVER_REVIEW_ROUTING_MISSING` | Review is required but no active review routing policy row covers the candidate condition. |
| `RESOLVER_SPLIT_POLICY_MISSING` | A split decision is selected without an active split policy. |
| `RESOLVER_EXPLANATION_INCOMPLETE` | Required explanation fields or checksum inputs are missing. |
| `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | Required resolver activation scenario, checksum, validation ref, or result field is missing. |
| `RESOLVER_ACTIVATION_REPORT_FAILED` | Required resolver activation scenario gate failed. |
| `IDENTITY_REVIEW_TRANSITION_INVALID` | Review event is illegal for the current state. |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | Review event evidence snapshot checksum does not match the case snapshot. |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | Reviewer lacks authority for the requested review event. |
| `TARGET_SELECTOR_UNSAFE` | Selector attempts a resolution state beyond `TargetSelectorSafetyPolicy`. |
| `DEPRECATED_NAME_MATCHING_FORBIDDEN` | Deprecated name matching is used in production. |
| `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` | Telemetry-only attributes or identifiers are used as durable identity evidence, candidate scoring authority, create/attach/merge authority, split authority, rejection authority, or hard-blocker authority. |

### IdentityErrorContext

`IdentityErrorContext` is the owner context schema for `070` resolver, target-selector, review, split, explanation, and telemetry-identity boundary registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `070` context schema version. |
| `owner_spec` | Yes | Must be `070`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `resolver_artifact`, `resolver_profile`, `identity_evidence`, `hard_blocker`, `candidate_generation`, `decision_matrix`, `review`, `split`, `explanation`, `activation_report`, `target_selector`, or `telemetry_identity_non_authority`. |
| `operation` | Yes | Resolver profile selection, evidence materialization, candidate generation, blocker evaluation, review transition, split handling, explanation emission, or target selector validation. |
| `affected_record_type` | Yes | Resolver artifact, identity evidence item, review case, target selector, or identity decision type. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to resolver profiles, evidence class row sets, identifier scope rows, hard blocker row sets, candidate generation profiles, decision matrices, confidence bands, review routing policies, split policies, explanation policies, target selector policies, activation reports, fixtures, package-set refs, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `entity_type` | No | Required when resolver scope was evaluated. |
| `source_scope_checksum` | No | Checksum only; raw tenant, directory, scanner, route, or account values are forbidden in caller context. |
| `evidence_class` | No | Required when the failure is evidence-class-specific. |
| `resolver_profile_ref` | No | Required when a resolver profile was consulted. |
| `review_case_ref` | No | Required when the failure is review-specific. |
| `validation_refs` | Yes | Exact `120` identity fixture refs. |
| `redaction_classes` | Yes | Source-native identity values, telemetry resource values, private bindings, credentials, raw payload bytes, and graph/backend IDs must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |

### CandidateGenerationProfile bounds

| Field | Default | Maximum | Overflow behavior | Closure state |
| --- | --- | --- | --- | --- |
| `max_members_per_blocking_key` | `256` | `1024` | Emit `RESOLVER_CANDIDATE_BLOCK_OVERFLOW`; emit no mutation from the overflowed block; do not auto-merge overflowed candidates. | closed_local |
| `max_candidate_pairs_per_source_asset` | `128` | `512` | Emit `RESOLVER_CANDIDATE_PARTITION_OVERFLOW`; keep non-overflowed prior pairs only when their block did not overflow; do not auto-merge overflowed candidates. | closed_local |
| `max_candidate_pairs_per_resolver_partition` | `1000000` | `5000000` | Emit `RESOLVER_CANDIDATE_PARTITION_OVERFLOW`; stop pair generation at deterministic boundary; no mutation for overflowed partition. | closed_local |
| `learned_artifact_policy` | `disabled` | n/a | Learned artifacts cannot produce candidates when disabled; when enabled, learned candidates remain `candidate_hint` only. | closed_local |

Profile rows may set lower limits. Setting a value above the maximum or omitting a required cap fails artifact activation before production resolver output.

### ResolverExplanation required fields

`ResolverExplanation` checksum must include output-affecting fields and exclude non-output volatile fields.

| Inclusion class | Fields | Checksum rule |
| --- | --- | --- |
| Required included fields | resolver profile row ref/checksum, candidate generation profile ref/checksum, identifier scope refs/checksums, evidence class refs/checksums, hard blocker row refs/checksums, decision matrix ref/checksum, confidence band ref/checksum, review routing policy ref/checksum, split policy ref/checksum, explanation policy ref/checksum, target selector policy ref/checksum, source scope selector, run mode, entity type, candidate pair ID, blocking key, evidence item refs/checksums, hard blocker results, selected decision row, selected confidence band, canonical score, review routing result, graph correction handoff ref when applicable, output decision ref, version manifest ref | Included in `ResolverExplanation` checksum. |
| Required excluded fields | runtime duration, worker ID, thread ID, wall-clock evaluation time, UI labels, reviewer display name, non-output diagnostic ordering, request correlation ID | Excluded from `ResolverExplanation` checksum and must not affect replay equivalence. |

Missing any required included field emits `RESOLVER_EXPLANATION_INCOMPLETE` before the `IdentityDecision` becomes externally visible.

### ResolverActivationReport requirements

| Scenario gate | Required behavior |
| --- | --- |
| creation | At least one scenario proves durable evidence creation and weak-only creation rejection. |
| attachment | At least one scenario proves exact durable attach and rejection of attachment that would merge two canonical entities. |
| durable merge | At least one scenario proves exact durable merge and blocker-precedence rejection. |
| weak evidence rejection | IP-only, hostname-only, DNS-only, PTR-only, scanner-name-only, graph-key-only, mapped-target-only, and source-native-merge-history-only attempts must fail create, attach, and merge. |
| blocker precedence | Every hard blocker has fired and non-fired fixture pairs and proves it precedes confidence and review. |
| overflow | Block overflow and partition overflow emit deterministic errors and no mutation. |
| review totality | Every review state/event pair is covered and illegal transitions emit no mutation. |
| split handoff | Split emits `GraphCorrectionHandoff` with required refs and no direct graph mutation. |
| selector safety | Selector mechanisms cannot exceed maximum resolution state. |
| explanation checksum replay | Same inputs produce byte-identical explanation checksum. |
| shadow/canary determinism | Shadow and canary runs match production decision/explanation checksums before activation. |

`ResolverActivationReport` must contain the following promotion fields when a resolver profile or resolver artifact row set is promoted to `active`.

| Field | Required behavior |
| --- | --- |
| `activation_report_id` | Deterministic ID over selected resolver catalog refs, scenario results, fixture checksums, expected output checksums, and candidate package-set ref when present. |
| `resolver_profile_row_ref` | Exact selected `ResolverProfileRow` ref and checksum. |
| `resolver_catalog_refs` | Canonically sorted refs for every member in `Resolver activation catalog closure`. |
| `scenario_results` | One row per required scenario gate; each row must be `pass`, `fail`, `blocked`, or `not_run`. |
| `fixture_checksums` | Non-empty checksums for every scenario fixture required for promotion. |
| `expected_output_checksums` | Non-empty expected decision, explanation, review, handoff, no-op, or error checksums for every scenario required for promotion. |
| `shadow_run_refs` | Required when promotion mode is `shadow` or `canary`; otherwise empty. |
| `promotion_eligibility` | `eligible` only when every required scenario result is `pass`, no required field is `TODO`, and every selected artifact ref is active, validated, manifest-included, and package-set consistent. |
| `failure_precedence` | `fail` outranks `blocked`, `blocked` outranks `not_run`, and `not_run` outranks `pass` for aggregate promotion status. |

If a required field, scenario row, checksum, validation ref, or artifact ref is absent, the report is incomplete and promotion must emit `RESOLVER_ACTIVATION_REPORT_INCOMPLETE`. If any required scenario result is `fail`, promotion must emit `RESOLVER_ACTIVATION_REPORT_FAILED` and preserve the current active resolver artifacts.

### Hard blocker matrix

| Boundary | Evidence trigger | Default effect | Override allowance | Required validation rows |
| --- | --- | --- | --- | --- |
| scope mismatch or under-scoped evidence | missing required scope key or scope hash mismatch | reject create, attach, merge, and split from affected evidence | never | `identity-under-scoped-rejection` |
| prior split or revoked merge | prior terminal split or revoked merge covers the candidate pair | block re-merge of old event | never for replay of old event | `identity-prior-split-blocker` |
| generation boundary | reimage, clone, VDI, reinstall, delete/recreate, reenrollment, rekey, Kubernetes recreate | block auto-merge across generation | split-aware continuation may attach only when durable evidence permits | `identity-generation-boundary` |
| conflicting durable IDs | incompatible durable IDs under same exact scope | emit `conflicted` or review; no create/attach/merge | never with weak or learned evidence | `identity-conflicting-durable` |
| weak identifier reuse | reused hostname/IP/DNS/PTR/scanner value | block weak create, attach, and merge | never | `identity-weak-reuse-rejection` |
| source-native merge contradiction | source-native merge history contradicts Cadastre durable evidence | preserve lineage and block or review | never by itself | `identity-source-native-merge-contradiction` |
| candidate overflow | block or partition cap exceeded | overflow diagnostic/review only; no mutation | never in overflowed run | `identity-candidate-overflow` |

### Selector safety matrix

`TargetSelectorSafetyPolicyRowSet` must be total over every selector mechanism in the closed selector mechanism enum. Missing, inactive, ambiguous, checksum-mismatched, package-set-mismatched, or unmanifested selector rows fail before unresolved-target output, identity mutation, graph endpoint output, graph edge output, review routing, or validation acceptance.

| Selector mechanism | Maximum resolution state | Production mutation | Error or no-op behavior |
| --- | --- | --- | --- |
| `mapped_target` | `UnresolvedTargetReference` | No identity mutation. | Emit unresolved target or no-op according to active policy. |
| `opengraph_node_id_equality` | `UnresolvedTargetReference` | No identity mutation. | Emit unresolved target or no-op according to active policy. |
| `opengraph_property_matching` | `UnresolvedTargetReference` | No identity mutation. | Emit unresolved target or no-op according to active policy. |
| `deprecated_name_matching` | forbidden | No identity mutation. | Fail with `DEPRECATED_NAME_MATCHING_FORBIDDEN`. |
| `graph_key` | selector only | No identity influence. | No-op unless a non-mutating selector diagnostic is permitted. |
| `hostname` | candidate hint only | No graph edge or identity mutation by itself. | `candidate`, `rejected`, or `no_decision` only. |
| `ip_address` | candidate hint only | No graph edge or identity mutation by itself. | `candidate`, `rejected`, or `no_decision` only. |
| `dns_name` | candidate hint only | No graph edge or identity mutation by itself. | `candidate`, `rejected`, or `no_decision` only. |
| `ptr_name` | candidate hint only | No graph edge or identity mutation by itself. | `candidate`, `rejected`, or `no_decision` only. |
| `weak_endpoint_selector` | candidate hint only | No graph edge or identity mutation by itself. | `candidate`, `rejected`, or `no_decision` only. |

### GraphEndpointIdentityHandoff validation rows

| Validation row | Required behavior |
| --- | --- |
| `val-070-graph-endpoint-requires-identity-decision` | A projected graph endpoint requires a qualifying `IdentityDecision` and `CanonicalEntity` ref. |
| `val-070-unresolved-target-no-graph-endpoint` | `UnresolvedTargetReference` emits no graph endpoint and no edge. |
| `val-070-opengraph-property-match-no-graph-identity` | OpenGraph-style property matching remains selector-only and cannot create endpoint identity. |
| `val-070-split-handoff-no-resolver-graph-mutation` | Split handoff routes affected facts to `090` and the resolver emits no graph mutation. |

### IdentityErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `RESOLVER_ARTIFACT_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-artifact-missing` |
| `RESOLVER_ARTIFACT_INACTIVE` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-artifact-inactive` |
| `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `070` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-artifact-checksum-mismatch` |
| `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-artifact-scope-mismatch` |
| `RESOLVER_PROFILE_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-profile-missing` |
| `RESOLVER_PROFILE_ROW_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-profile-row-missing` |
| `RESOLVER_PROFILE_ROW_AMBIGUOUS` | `070` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-profile-row-ambiguous` |
| `RESOLVER_SCOPE_SUBSET_MUTATION_FORBIDDEN` | `070` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `070.IdentityErrorContext` | `error-registry-070-resolver-scope-subset-mutation-forbidden` |
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-entity-type-unsupported` |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-identity-evidence-under-scoped` |
| `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-identity-evidence-class-unsupported` |
| `IDENTITY_HARD_BLOCKER_TRIGGERED` | `070` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-identity-hard-blocker-triggered` |
| `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-hard-blocker-row-missing` |
| `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` | `070` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-hard-blocker-row-ambiguous` |
| `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-candidate-block-overflow` |
| `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | `070` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-candidate-partition-overflow` |
| `RESOLVER_DECISION_ROW_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-decision-row-missing` |
| `RESOLVER_DECISION_ROW_AMBIGUOUS` | `070` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-decision-row-ambiguous` |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-confidence-band-missing` |
| `RESOLVER_REVIEW_ROUTING_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-review-routing-missing` |
| `RESOLVER_SPLIT_POLICY_MISSING` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-split-policy-missing` |
| `RESOLVER_EXPLANATION_INCOMPLETE` | `070` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-explanation-incomplete` |
| `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `070` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-activation-report-incomplete` |
| `RESOLVER_ACTIVATION_REPORT_FAILED` | `070` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-resolver-activation-report-failed` |
| `IDENTITY_REVIEW_TRANSITION_INVALID` | `070` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-identity-review-transition-invalid` |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | `070` | `error` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `070.IdentityErrorContext` | `error-registry-070-identity-review-evidence-snapshot-mismatch` |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | `070` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `070.IdentityErrorContext` | `error-registry-070-identity-review-authority-missing` |
| `TARGET_SELECTOR_UNSAFE` | `070` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `070.IdentityErrorContext` | `error-registry-070-target-selector-unsafe` |
| `DEPRECATED_NAME_MATCHING_FORBIDDEN` | `070` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `070.IdentityErrorContext` | `error-registry-070-deprecated-name-matching-forbidden` |
| `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` | `070` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `070.IdentityErrorContext` | `error-registry-070-telemetry-identity-evidence-forbidden` |

### ActivationControlledRowSchemaPrecisionHandoff

The following field tables close the stable row-schema precision for every `070` output-affecting resolver row family. Concrete row bytes remain activation-controlled artifacts; this section defines the executable row interfaces that those artifacts must satisfy. The selected row ref, selected row checksum, containing row-set checksum, validation refs, package-set ref when package-supplied, and resolver runtime refs that consume the row must appear in `030.VersionManifest` before any identity output, review output, resolver explanation, or split handoff becomes externally visible.

Rows in these tables use `040.CanonicalJSON` and `030.ActivationControlledRowRef` mechanics. Unknown fields are forbidden. Bare string refs are forbidden. Arrays marked `canonical_set` reject duplicates after canonicalization and sort by the declared canonical sort key before checksum computation. A selected row with any missing required field, invalid enum value, out-of-bounds scalar, non-canonical array, checksum mismatch, package-set mismatch, private-binding leak, or manifest omission must fail before candidate generation or mutation with the most specific `070`, `030`, `100`, or manifest error.

#### ResolverProfileRow field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable ID scoped to row set | n/a | reject | n/a | ordered:1 | yes | closed | `110` | selected row ref/checksum | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `resolver_profile_id` | `040.ScalarType.identifier` | yes | none | no | no | stable profile ID | n/a | reject | n/a | ordered:2 | yes | closed | `110` | profile ref/checksum | `RESOLVER_PROFILE_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `profile_version` | `040.ScalarType.version_token` | yes | none | no | no | immutable owner version | n/a | reject | n/a | ordered:3 | yes | closed | `110` | profile version included | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `run_mode` | owner enum `ResolverRunMode` | yes | none | no | no | `production`, `shadow`, `canary`, `validation` | n/a | reject | n/a | ordered:4 | yes | closed | `110` | selected row run mode | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `entity_type` | owner enum `ResolverEntityType` | yes | none | no | no | `host`, `user`, `service_account`, `group`, `unsupported_entity_type` | n/a | reject | n/a | ordered:5 | yes | closed | `110` | selected row entity type | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ENTITY_TYPE_UNSUPPORTED` |
| `source_scope_selector` | `030.ScopeSelector` | yes | none | no | no | context `070.IdentifierScopeSelectorContext`; private values forbidden | n/a | reject | n/a | ordered:6 | yes | closed | `110` | selector context ref and checksum | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `source_scope_match_policy` | owner enum `SourceScopeMatchPolicy` | no | `exact_scope_required` | no | yes | `exact_scope_required`, `scope_subset_allowed_for_review_only` | n/a | reject | n/a | no | yes | closed | `110` | selected row checksum | none | `RESOLVER_SCOPE_SUBSET_MUTATION_FORBIDDEN` |
| `evidence_class_set_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identifier_evidence_class_row_set` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum and package-set ref when package-supplied | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `identifier_scope_row_refs` | array of `030.ActivationControlledRowRef` | yes | none | no | no | length `1..128`; row family `IdentifierScope` | canonical_set | reject | `row_id` | no | yes | closed | `110` | every selected scope row ref/checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `candidate_generation_profile_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `candidate_generation_profile` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `identity_hard_blocker_row_set_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identity_hard_blocker_row_set` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `asset_generation_boundary_row_set_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `asset_generation_boundary_row_set` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `decision_matrix_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `resolver_decision_matrix_row_set` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| `confidence_band_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identity_confidence_band_row_set` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `review_routing_policy_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identity_review_routing_policy` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `split_policy_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identity_split_policy` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `explanation_policy_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `resolver_explanation_policy` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `target_selector_policy_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `target_selector_safety_policy` | n/a | reject | n/a | no | yes | closed | `110` | artifact ref/checksum | `TARGET_SELECTOR_UNSAFE` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `allowed_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | non-empty subset of closed identity decision states | canonical_set | reject | enum token | no | yes | closed | `110` | selected row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `creation_policy` | owner enum `CreationPolicy` | no | derived:`entity_type` | no | yes | `durable_evidence_creation_allowed`, `creation_forbidden` | n/a | reject | n/a | no | yes | closed | `110` | selected row checksum | none | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `merge_policy` | owner enum `MergePolicy` | no | `durable_evidence_merge_allowed` | no | yes | `durable_evidence_merge_allowed`, `merge_forbidden` | n/a | reject | n/a | no | yes | closed | `110` | selected row checksum | none | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `learned_artifact_policy` | owner enum `LearnedArtifactPolicy` | no | `disabled` | no | yes | `disabled`, `candidate_hint_only` | n/a | reject | n/a | no | yes | closed | `110` | selected row checksum | none | `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` when telemetry-only learned inputs are used |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | length `1..256`; passing `120-IDENTITY-CLOSURE-*` refs | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs included | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral scope only | n/a | reject | n/a | no | yes | closed | `110` | activation scope checksum | `RESOLVER_PROFILE_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle evidence ref | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### IdentifierEvidenceClass field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` |
| `evidence_class` | owner enum `IdentifierEvidenceClassToken` | yes | none | no | no | exactly one MVP evidence class token; duplicates forbidden across row set | n/a | reject | n/a | ordered:2 | yes | closed | `110` | selected evidence class ref/checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` |
| `durability` | owner enum `EvidenceDurability` | yes | none | no | no | `high`, `medium`, `low`, `model_artifact_scoped`, `telemetry_forbidden` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` when telemetry is durable |
| `default_role` | owner enum `EvidenceRole` | yes | none | no | no | closed evidence roles plus `telemetry_forbidden` for telemetry-only rows | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `required_scope_keys` | array of `040.ScalarType.identifier` | yes | none | no | no | length `1..32` unless class is explicitly selector-only with selector scope | canonical_set | reject | key token | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `creation_authority` | owner enum `EvidenceMutationAuthority` | yes | none | no | no | `never`, `durable_exact_scope_only` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `attachment_authority` | owner enum `EvidenceMutationAuthority` | yes | none | no | no | `never`, `durable_exact_scope_only`, `source_object_identity_only` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `merge_authority` | owner enum `EvidenceMutationAuthority` | yes | none | no | no | `never`, `durable_exact_scope_only` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `gold_reference_eligible_by_itself` | `040.ScalarType.boolean` | no | `false` | no | yes | `true` only for durable classes when profile and `080` predicate permit | n/a | reject | n/a | no | yes | closed | `110` | row checksum | none | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `review_routing_default` | owner enum `ReviewRoutingDefault` | yes | none | no | no | `no_decision`, `candidate_review_allowed`, `conflict_review_required`, `lineage_only`, `forbidden` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | non-empty totality and weak-default validation refs | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### IdentifierScope field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `source_class` | owner enum `IdentifierSourceClass` | yes | none | no | no | `cloud provider`, `endpoint agent`, `directory`, `Kubernetes`, `network`, `DNS`, `graph`, `selector`, `learned`, `telemetry` | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `identifier_class` | owner enum `IdentifierClass` | yes | none | no | no | closed identifier classes from `IdentifierScope canonicalization` plus mapped target and learned hint | n/a | reject | n/a | ordered:3 | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `required_scope_keys` | array of `040.ScalarType.identifier` | yes | none | no | no | length `1..32` | canonical_set | reject | key token | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `optional_scope_keys` | array of `040.ScalarType.identifier` | no | `[]` | no | yes | length `0..32` | canonical_set | reject | key token | no | yes | closed | `110` | row checksum | none | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `normalization_rule` | owner enum `IdentifierNormalizationRule` | yes | none | no | no | exact owner rule token; prose-only normalization forbidden | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `scope_hash_inputs` | array of `040.ScalarType.identifier` | yes | none | no | no | non-empty subset of required plus optional scope keys | ordered_sequence | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `uncovered_scope_behavior` | owner enum `UncoveredScopeBehavior` | yes | none | no | no | `under_scoped_no_candidates`, `weak_evidence_only`, `selector_only_no_identity`, `source_object_identity_only`, `telemetry_forbidden` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `IDENTITY_EVIDENCE_UNDER_SCOPED` | `TELEMETRY_IDENTITY_EVIDENCE_FORBIDDEN` when telemetry is used |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | non-empty scope coverage and under-scoped fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### CandidateGenerationProfile field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `candidate_generation_profile_id` | `040.ScalarType.identifier` | yes | none | no | no | stable profile ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | profile ref/checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `blocking_key_rows` | array of owner object `BlockingKeyRow` | yes | none | no | no | length `1..64`; ranks unique; required MVP ranks must be present | canonical_set | reject | `block_kind_rank` | no | yes | closed | `110` | profile checksum and validation refs | `RESOLVER_ARTIFACT_MISSING` | `IDENTITY_EVIDENCE_UNDER_SCOPED` |
| `max_members_per_blocking_key` | `040.ScalarType.uint64` | no | `256` | no | yes | `1..1024` | n/a | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` |
| `max_candidate_pairs_per_source_asset` | `040.ScalarType.uint64` | no | `128` | no | yes | `1..512` | n/a | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` |
| `max_candidate_pairs_per_resolver_partition` | `040.ScalarType.uint64` | no | `1000000` | no | yes | `1..5000000` | n/a | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` |
| `pair_ordering` | ordered array of owner enum `CandidatePairSortKey` | no | block kind rank, blocking-key UTF-8 bytes, lexical left source asset ID, lexical right source asset ID, evidence item set checksum | no | yes | exactly five sort keys in the declared order | ordered_sequence | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `overflow_behavior` | owner enum `CandidateOverflowBehavior` | no | `emit_error_no_mutation` | no | yes | `emit_error_no_mutation`; no weaker value allowed for MVP | n/a | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` |
| `learned_artifact_policy` | owner enum `LearnedArtifactPolicy` | no | `disabled` | no | yes | `disabled`, `candidate_hint_only` | n/a | reject | n/a | no | yes | closed | `110` | profile checksum | none | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | non-empty pair-order, overflow, learned-disabled, and group-key fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `profile_checksum` | `040.ScalarType.sha256` | yes | derived:`profile_body_excluding_profile_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | profile checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### IdentityHardBlockerRow field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` |
| `blocker_family` | owner enum `HardBlockerFamily` | yes | none | no | no | seven families from hard blocker precedence table | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` |
| `precedence` | `040.ScalarType.uint64` | yes | none | no | no | exact `1..7` matching blocker family | n/a | reject | n/a | ordered:3 | yes | closed | `110` | row checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` |
| `trigger_condition` | owner object `BooleanCondition` | yes | none | no | no | canonical condition over evidence roles, scopes, candidate state, prior decisions, and profile policy | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` |
| `blocked_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | non-empty subset of decision states | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `override_policy` | owner enum `HardBlockerOverridePolicy` | no | `never` | no | yes | `never`, `split_aware_attach_only`, `diagnostic_review_only` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | none | `IDENTITY_HARD_BLOCKER_TRIGGERED` |
| `explanation_field_key` | `040.ScalarType.identifier` | yes | none | no | no | must match `ResolverExplanationPolicy.included_field_rows` | n/a | reject | n/a | no | yes | closed | `110` | explanation policy ref/checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | fired and not-fired fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### AssetGenerationBoundary field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `boundary_type` | owner enum `AssetGenerationBoundaryType` | yes | none | no | no | reimage, clone, VDI reuse, agent reinstall, provider delete/recreate, directory reenrollment, Kubernetes recreate, source rekey, hostname reuse, IP reuse, scanner correlation change | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row checksum | `RESOLVER_ARTIFACT_MISSING` | `IDENTITY_HARD_BLOCKER_TRIGGERED` |
| `trigger_condition` | owner object `BooleanCondition` | yes | none | no | no | canonical condition over lifecycle evidence and durable/weak evidence | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_ARTIFACT_MISSING` | `IDENTITY_HARD_BLOCKER_TRIGGERED` |
| `generation_reset_behavior` | owner enum `GenerationResetBehavior` | yes | none | no | no | `block_auto_merge`, `source_object_identity_only`, `split_aware_attach_only`, `review_only` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_ARTIFACT_MISSING` | `IDENTITY_HARD_BLOCKER_TRIGGERED` |
| `blocked_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | non-empty subset | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `RESOLVER_ARTIFACT_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | boundary fired and not-fired fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### ResolverDecisionMatrix field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `decision_row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| `decision_class` | owner enum `IdentityDecisionState` | yes | none | no | no | one closed identity decision state | n/a | reject | n/a | ordered:2 | yes | closed | `110` | selected row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| `decision_priority` | `040.ScalarType.uint64` | yes | none | no | no | `0..1000000`; lower wins only after non-equal specificity | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| `required_condition` | owner object `BooleanCondition` | yes | none | no | no | exact canonical condition; prose-only invalid | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_DECISION_ROW_AMBIGUOUS` |
| `required_confidence_band_ref` | `030.ActivationControlledRowRef` | yes | none | no | no | row family `IdentityConfidenceBand` | n/a | reject | n/a | no | yes | closed | `110` | confidence band row ref/checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `terminality` | owner enum `DecisionTerminality` | yes | none | no | no | `terminal_mutating`, `terminal_non_mutating`, `non_terminal_reviewable` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `mutation_limit` | array of owner enum `MutationClass` | yes | none | no | no | subset of `none`, `create_canonical_entity`, `attach_source_asset`, `merge_canonical_entities`, `emit_split_handoff` | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `forbidden_evidence_roles` | array of owner enum `EvidenceRole` | yes | none | no | no | weak, selector, lineage-only, correlation-only, source-object-only, telemetry-forbidden, and learned-only roles must appear for mutating rows | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `RESOLVER_DECISION_ROW_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | positive, rejection, replay, and mutation-prohibition fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### IdentityConfidenceBand field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `band_id` | owner enum `IdentityConfidenceBandToken` | yes | none | no | no | eight MVP band tokens exactly once per row set | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_CONFIDENCE_BAND_MISSING` |
| `score` | `040.DecimalPrecisionPolicy.confidence_0_1` | yes | none | no | no | canonical string `0.000000..1.000000`; no binary float | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `selection_condition` | owner object `BooleanCondition` | yes | none | no | no | exact canonical condition | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_CONFIDENCE_BAND_MISSING` |
| `permitted_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | non-empty subset | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `selection_order` | `040.ScalarType.uint64` | yes | none | no | no | exact `1..8`; first matching row wins | n/a | reject | n/a | ordered:3 | yes | closed | `110` | row checksum | `RESOLVER_CONFIDENCE_BAND_MISSING` | `RESOLVER_CONFIDENCE_BAND_MISSING` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | selection, non-selection, decimal, replay fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### IdentityReviewRoutingPolicy field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.identifier` | yes | none | no | no | stable policy ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | policy ref/checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `case_expiration_seconds` | `040.ScalarType.uint64` | no | `1209600` | no | yes | `86400..7776000` | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | none | `IDENTITY_REVIEW_TRANSITION_INVALID` |
| `expiration_effect` | owner enum `ReviewExpirationEffect` | no | `expired_no_mutation` | no | yes | `expired_no_mutation` only for MVP | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | none | `IDENTITY_REVIEW_TRANSITION_INVALID` |
| `weak_only_review_behavior` | owner enum `WeakOnlyReviewBehavior` | yes | none | no | no | `review_allowed_no_merge_without_new_durable_evidence`, `no_review` | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `IDENTITY_REVIEW_AUTHORITY_MISSING` |
| `hard_blocker_review_behavior` | owner enum `HardBlockerReviewBehavior` | yes | none | no | no | `document_only_no_override`, `no_review` | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `IDENTITY_HARD_BLOCKER_TRIGGERED` |
| `terminal_decision_requirement` | owner enum `TerminalDecisionRequirement` | yes | `identity_mutation_only_through_terminal_decision` | no | no | exact MVP token | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_REVIEW_ROUTING_MISSING` | `IDENTITY_REVIEW_TRANSITION_INVALID` |
| `state_event_transition_rows` | array of owner object `ReviewTransitionRow` | yes | none | no | no | total over every review state/event pair | canonical_set | reject | `current_state,event` | no | yes | closed | `110` | transition row refs/checksums | `RESOLVER_REVIEW_ROUTING_MISSING` | `IDENTITY_REVIEW_TRANSITION_INVALID` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | review-opening, weak-only, hard-blocker, expiration, terminal, illegal-transition fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `policy_checksum` | `040.ScalarType.sha256` | yes | derived:`policy_body_excluding_policy_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | policy checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### SplitPolicy field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `split_policy_id` | `040.ScalarType.identifier` | yes | none | no | no | stable policy ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | split policy ref/checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `trigger_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | non-empty subset containing `split` | canonical_set | reject | enum token | no | yes | closed | `110` | policy checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `partition_key_rule` | owner object `PartitionKeyRule` | yes | none | no | no | canonical rule over durable evidence scope, generation boundary, known time, and source asset refs | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `retained_canonical_partition_rule` | owner enum `RetainedPartitionRule` | yes | `oldest_active_durable_evidence_then_lexical_canonical_entity_id` | no | no | exact default or stricter deterministic rule | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `new_canonical_partition_rule` | owner enum `NewCanonicalPartitionRule` | yes | none | no | no | `one_canonical_entity_per_non_retained_partition` | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `ambiguous_partition_behavior` | owner enum `AmbiguousSplitPartitionBehavior` | no | `emit_conflicted_no_split` | no | yes | `emit_conflicted_no_split`, `open_review`, `reject_split` | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | none | `RESOLVER_SPLIT_POLICY_MISSING` |
| `affected_fact_selection` | owner union `AffectedFactSelection` | yes | none | no | no | exact fact refs or deterministic affected-fact selection checksum | n/a | reject | n/a | no | yes | closed | `110` | affected fact refs or checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `graph_correction_handoff_required` | `040.ScalarType.boolean` | yes | `true` | no | no | must be true when gold or graph output may be affected | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | partition, ambiguous partition, handoff, replay, and no-graph-mutation fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `policy_checksum` | `040.ScalarType.sha256` | yes | derived:`policy_body_excluding_policy_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | policy checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### ResolverExplanationPolicy field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.identifier` | yes | none | no | no | stable policy ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | policy ref/checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| `included_checksum_fields` | array of `040.ScalarType.identifier` | yes | none | no | no | exact fields listed in `ResolverExplanation required fields`; wildcards forbidden | canonical_set | reject | field path | no | yes | closed | `110` | policy checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| `excluded_volatile_fields` | array of `040.ScalarType.identifier` | yes | none | no | no | exact volatile fields listed in `ResolverExplanation required fields`; wildcards forbidden | canonical_set | reject | field path | no | yes | closed | `110` | policy checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | explanation checksum and volatile-field exclusion fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `policy_checksum` | `040.ScalarType.sha256` | yes | derived:`policy_body_excluding_policy_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | policy checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### TargetSelectorSafetyPolicy field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.identifier` | yes | none | no | no | stable row ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | row ref/checksum | `TARGET_SELECTOR_UNSAFE` | `TARGET_SELECTOR_UNSAFE` |
| `selector_mechanism` | owner enum `SelectorMechanism` | yes | none | no | no | one closed selector mechanism; every mechanism exactly once per active set | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row checksum | `TARGET_SELECTOR_UNSAFE` | `DEPRECATED_NAME_MATCHING_FORBIDDEN` for deprecated name matching in production |
| `maximum_resolution_state` | owner enum `SelectorMaximumResolutionState` | yes | none | no | no | `UnresolvedTargetReference`, `selector_only`, `candidate_hint_only`, or `forbidden` | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `TARGET_SELECTOR_UNSAFE` | `TARGET_SELECTOR_UNSAFE` |
| `permitted_decision_classes` | array of owner enum `IdentityDecisionState` | yes | none | no | no | subset of `candidate`, `rejected`, `no_decision`; empty allowed only when forbidden | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `TARGET_SELECTOR_UNSAFE` | `TARGET_SELECTOR_UNSAFE` |
| `forbidden_identity_roles` | array of owner enum `EvidenceRole` | yes | none | no | no | must include mutating and durable roles for selector-only mechanisms | canonical_set | reject | enum token | no | yes | closed | `110` | row checksum | `TARGET_SELECTOR_UNSAFE` | `TARGET_SELECTOR_UNSAFE` |
| `production_mutation_effect` | owner enum `SelectorProductionMutationEffect` | yes | `none` | no | no | `none` for all MVP rows | n/a | reject | n/a | no | yes | closed | `110` | row checksum | `TARGET_SELECTOR_UNSAFE` | `TARGET_SELECTOR_UNSAFE` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | selector totality, deprecated-name rejection, OpenGraph no-identity, and graph endpoint no-op fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`row_body_excluding_row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### ResolverActivationReportPolicy field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.identifier` | yes | none | no | no | stable policy ID | n/a | reject | n/a | ordered:1 | yes | closed | `110` | policy ref/checksum | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `required_scenario_gates` | array of owner enum `ResolverActivationScenarioGate` | yes | none | no | no | creation, attachment, durable merge, weak rejection, blocker precedence, overflow, review totality, split handoff, selector safety, explanation replay, shadow/canary determinism | canonical_set | reject | enum token | no | yes | closed | `110` | policy checksum | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `promotion_eligibility_rule` | owner enum `ResolverPromotionEligibilityRule` | yes | `all_required_scenarios_pass_no_todo_manifest_included_package_consistent` | no | no | exact MVP token | n/a | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `failure_precedence` | ordered array of owner enum `ActivationReportResult` | yes | `fail`, `blocked`, `not_run`, `pass` | no | no | exactly four tokens in declared order | ordered_sequence | reject | n/a | no | yes | closed | `110` | policy checksum | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | scenario gate validation refs | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | public, vendor-neutral | n/a | reject | n/a | no | yes | closed | `110` | scope checksum | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `RESOLVER_ARTIFACT_SCOPE_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle refs | `RESOLVER_ARTIFACT_INACTIVE` | `RESOLVER_ARTIFACT_INACTIVE` |
| `policy_checksum` | `040.ScalarType.sha256` | yes | derived:`policy_body_excluding_policy_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | derived | no | closed | `110` | policy checksum | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |

#### GraphCorrectionHandoff runtime-state field table

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `graph_correction_handoff_id` | `040.ScalarType.identifier` | yes | derived:`split_decision_policy_partitions_affected_facts_explanation_manifest` | no | no | deterministic ID; backend IDs forbidden | n/a | reject | n/a | derived | yes | closed | `110` | handoff ref/checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `split_identity_decision_ref` | `030.ActivationControlledRowRef` or owner runtime ref | yes | none | no | no | terminal `split` decision only | n/a | reject | n/a | ordered:1 | yes | closed | `110` | decision ref/checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `split_policy_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact class `identity_split_policy` | n/a | reject | n/a | ordered:2 | yes | closed | `110` | split policy ref/checksum | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` |
| `retained_canonical_entity_ref` | owner runtime ref `CanonicalEntity` | yes | none | no | no | one valid `040.CanonicalEntity` ref | n/a | reject | n/a | ordered:3 | yes | closed | `110` | retained canonical ref | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `new_canonical_entity_refs` | array of owner runtime ref `CanonicalEntity` | yes | `[]` | no | yes | required non-empty when split creates new canonicals | canonical_set | reject | canonical entity ref | ordered:4 | yes | closed | `110` | new canonical refs sorted | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `split_partition_refs` | array of owner runtime ref `SplitPartition` | yes | none | no | no | length `1..1024`; refs or checksums sorted canonically | canonical_set | reject | partition ref or checksum | ordered:5 | yes | closed | `110` | partition refs/checksums | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `affected_fact_refs` | array of owner runtime ref `GoldFact` | conditional:`affected_fact_selection_checksum absent` | none | yes | yes | null allowed only when `affected_fact_selection_checksum` is present | canonical_set | reject | fact ref | ordered:6 | yes | closed | `110` | affected fact refs when enumerated | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `affected_fact_selection_checksum` | `040.ScalarType.sha256` | conditional:`affected_fact_refs null or omitted` | none | yes | yes | SHA-256 lowercase hex; null allowed only when exact refs are enumerated | n/a | reject | n/a | ordered:7 | yes | closed | `110` | selection checksum when refs omitted | `RESOLVER_SPLIT_POLICY_MISSING` | `RESOLVER_SPLIT_POLICY_MISSING` |
| `resolver_explanation_checksum` | `040.ScalarType.sha256` | yes | none | no | no | checksum of split decision explanation | n/a | reject | n/a | ordered:8 | yes | closed | `110` | explanation checksum | `RESOLVER_EXPLANATION_INCOMPLETE` | `RESOLVER_EXPLANATION_INCOMPLETE` |
| `version_manifest_ref` | owner runtime ref `030.VersionManifest` | yes | none | no | no | producing manifest ref | n/a | reject | n/a | ordered:9 | yes | closed | `110` | manifest ref | `VERSION_MANIFEST_INCOMPLETE` | `VERSION_MANIFEST_INCOMPLETE` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | split handoff fixtures | canonical_set | reject | artifact ref | no | yes | closed | `110` | validation refs | `RESOLVER_ACTIVATION_REPORT_INCOMPLETE` | `RESOLVER_ACTIVATION_REPORT_FAILED` |

`GraphCorrectionHandoff` must not contain graph backend IDs, graph backend labels, backend-generated vertex or edge IDs, private source binding values, raw payload bytes, reviewer display names, UI labels, or request correlation IDs. `new_canonical_entity_refs`, `split_partition_refs`, and `affected_fact_refs` when present must be canonically sorted before ID and checksum computation.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `070-CLEANUP-AC-001` | No banned reference class remains. |
| `070-CLEANUP-AC-002` | Identity resolution fails when no active `ResolverProfileRow` covers run mode, entity type, source scopes, evidence classes, lifecycle boundary types, and activation scope. |
| `070-CLEANUP-AC-003` | Hard blockers and lifecycle boundaries run before confidence computation, review approval, and decision matrix selection. |
| `070-CLEANUP-AC-004` | Target selectors cannot create, attach, or merge canonical identity by themselves. |
| `070-SCHEMA-PATCH-AC-001` | Every emitted `CanonicalEntity`, `SourceAsset`, and `Identifier` passes `040.ValidateCoreRecord`. |
| `070-SCHEMA-PATCH-AC-002` | `070` does not restate core record fields except by exact schema name reference. |
| `070-SCHEMA-PATCH-AC-003` | Weak evidence cannot produce a canonical entity, source asset attachment, or identifier merge by bypassing the `040` and `070` validation sequence. |
| `070-RESOLVER-ROW-AC-001` | Exactly one active `ResolverProfileRow` is selected before candidate generation, and missing, ambiguous, inactive, checksum-mismatched, or out-of-scope rows fail before mutation. |
| `070-MVP-MATRIX-AC-001` | `host`, `user`, `service_account`, `group`, and `unsupported_entity_type` are covered by the MVP coverage matrix with closed decision outputs. |
| `070-DURABLE-MERGE-AC-001` | Exact durable evidence under exact scope can create, attach, or merge only through the matching decision row and confidence band. |
| `070-WEAK-REJECTION-AC-001` | Weak-only, selector-only, learned-only, source-native-merge-history-only, and graph-key-only evidence never create, attach, or merge. |
| `070-PAIR-ORDER-AC-001` | Candidate pair ordering is byte-identical across implementations for the same blocking keys and evidence item checksums. |
| `070-OVERFLOW-AC-001` | Block and partition overflow emit deterministic errors and no identity mutation. |
| `070-BLOCKER-PRECEDENCE-AC-001` | Blockers execute before confidence selection and reviewer authority, and non-overridable blockers cannot be overridden. |
| `070-REVIEW-TOTALITY-AC-001` | Every `IdentityReviewCase` state/event pair has an allowed transition, idempotent no-op, terminal no-op, or `IDENTITY_REVIEW_TRANSITION_INVALID` without identity mutation. |
| `070-EXPLANATION-AC-001` | Every `IdentityDecision` has exactly one `ResolverExplanation` with all required included fields and a canonical checksum. |
| `070-SPLIT-HANDOFF-AC-001` | Every identity split affecting gold or graph output emits `GraphCorrectionHandoff` with split policy, partition, affected fact, resolver explanation, and version manifest refs. |
| `070-MANIFEST-AC-001` | `VersionManifest` contains all consulted resolver artifacts, runtime decision refs, review refs, explanation checksums, and graph correction handoff refs. |
| `070-PACKAGE-WEAKENING-AC-001` | Package-supplied resolver artifacts that weaken stable weak-evidence defaults or redefine identity semantics fail activation before production output. |
| `070-TELEMETRY-IDENTITY-NONAUTH-AC-001` | Telemetry-only service, host, container, process, runtime, trace, span, exporter, Collector, or backend identifiers cannot create, attach, merge, split, score, or reject canonical identity. |
| `070-VOLATILITY-AC-001` | Inactive resolver profile, target selector artifact omission, resolver artifact checksum mismatch, and package artifact core conflict fail before identity mutation. |
| `070-LIFECYCLE-AC-001` | Resolver profile activation emits generic artifact lifecycle transition evidence and resolver-specific guard results before identity mutation. |
| `070-LIFECYCLE-CLOSURE-AC-001` | `IdentityReviewCaseLifecycleClosure` names `070.IdentityReviewCaseStateMachine.v1`, routes generic lifecycle evidence to `030`, and lists the required `120` identity-review validation rows. |
| `070-GRAPH-ENDPOINT-HANDOFF-AC-001` | Graph endpoint projection requires a qualifying identity decision and canonical entity ref. |
| `070-GRAPH-ENDPOINT-HANDOFF-AC-002` | Selector-only, graph-key-only, mapped-target-only, and OpenGraph property-match-only inputs cannot create graph endpoint identity. |
| `070-GRAPH-ENDPOINT-HANDOFF-AC-003` | Unresolved endpoint identity is reported to `090` as a projection blocker and never mutates identity. |
| `070-GOLD-REF-ELIGIBILITY-AC-001` | Terminal or explicitly permitted identity decisions allow eligible `canonical_entity_ref`, `source_asset_ref`, or `identifier_ref` only when the selected `080` predicate contract permits the reference kind. |
| `070-GOLD-REF-ELIGIBILITY-AC-002` | Weak-only, selector-only, unresolved-target, graph-key-only, source-native-merge-history-only, learned-only, and telemetry-only inputs cannot become `GoldFact.subject_ref` or `GoldFact.object_value` references. |
| `070-GOLD-REF-ELIGIBILITY-AC-003` | `Identifier` refs require typed scope and validated `IdentifierScope`; `SourceAsset` refs require sufficient source scope, native identity, and asset type. |
| `070-GOLD-REF-ELIGIBILITY-AC-004` | Package-supplied resolver artifacts cannot weaken gold reference eligibility defaults. |
| `070-EVIDENCE-REGISTRY-TOTALITY-AC-001` | Every evidence class named by `ResolverProfileCoverageMatrix`, `Evidence Roles`, `IdentifierEvidenceClass registry`, `Candidate Generation`, selector policy, and validation rows appears exactly once in the active evidence class row set. |
| `070-EVIDENCE-WEAK-ROW-MISSING-AC-001` | Missing rows for `scanner_name`, `weak_management_name`, `weak_user_name_mail_upn`, `weak_group_name`, `group_membership_hint`, or `weak_service_account_display_name` block resolver artifact activation. |
| `070-HARD-BLOCKER-ROWSET-TOTALITY-AC-001` | Every hard blocker family has exactly one active most-specific `IdentityHardBlockerRow`; missing or ambiguous rows fail before confidence, review, or decision selection. |
| `070-DECISION-ROW-AMBIGUITY-AC-001` | Equally specific active decision rows emit `RESOLVER_DECISION_ROW_AMBIGUOUS` and no identity mutation. |
| `070-REVIEW-EXPIRATION-AC-001` | Review expiration uses `case_expiration_seconds` default `1209600`, respects bounds `86400..7776000`, emits transition evidence, and performs no identity mutation. |
| `070-GRAPH-CORRECTION-HANDOFF-SCHEMA-AC-001` | Every split handoff has all required `GraphCorrectionHandoff` fields and rejects missing refs, missing checksums, or non-canonical ordering before correction or graph projection. |
| `070-SELECTOR-SAFETY-TOTALITY-AC-001` | `opengraph_node_id_equality`, `opengraph_property_matching`, `deprecated_name_matching`, `mapped_target`, `graph_key`, and weak endpoint selectors have total maximum-resolution behavior and cannot create identity by themselves. |
| `070-ACTIVATION-REPORT-GATE-AC-001` | Resolver promotion is forbidden unless `ResolverActivationReport` contains passing scenario gates, fixture checksums, expected output checksums, manifest refs, and no `TODO` values. |
| `070-PACKAGE-ROWSET-WEAKENING-AC-001` | Package-supplied resolver row sets that weaken stable weak-evidence defaults fail activation and preserve the current active resolver artifact set. |

### Structured input resolver acceptance criteria

| ID | Criterion |
| --- | --- |
| `070-STRUCTURED-INPUT-RESOLVER-AC-001` | Repository-authored resolver profile missing activation refs, materialization refs, package-set refs when package-supplied, scenario validation refs, or manifest refs fails before identity output. |
| `070-STRUCTURED-INPUT-RESOLVER-AC-002` | Repository-authored resolver profile that weakens hard-blocker precedence, weak-evidence defaults, review terminality, split policy, or selector safety fails before activation and before identity mutation. |
| `070-STRUCTURED-INPUT-RESOLVER-AC-003` | Change proposal, merge, branch update, PR approval, validation run, and hook success produce no identity mutation by themselves. |
| `070-STRUCTURED-INPUT-RESOLVER-AC-004` | Stale CI cannot activate resolver rows; tool-generated weakening of hard blockers fails before identity mutation; sync-only resolver candidates create no identity mutation. |
| `070-STRUCTURED-INPUT-RESOLVER-AC-005` | Private resolver bindings are rejected before API, audit, telemetry, package report, or validation output. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `070-SCOPE-IDENTITY-AC-001` | Exact durable evidence, missing required identifier-scope key, provider/account scope mismatch, subset review-only route, subset create/merge rejection, private resolver scope leak, ambiguous resolver row, and manifest-inclusion fixtures pass. |
| `070-SCOPE-IDENTITY-AC-002` | Candidate generation does not start until resolver row scope and every identity evidence scope normalize under the active `030.ScopeSelectorContext`. |
| `070-AC-001` | Same inputs, active resolver row, source scopes, blockers, artifact refs, and version manifest produce byte-identical identity decisions and explanations. |
| `070-AC-002` | IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-only, mapped-target-only, learned-only, and selector-only evidence never create, attach, or merge. |
| `070-AC-003` | Hard blockers and lifecycle boundaries override confidence scores and reviewer notes. |
| `070-AC-004` | Manual review cannot mutate identity without terminal `IdentityDecision`, `IdentityReviewCase`, `ResolverExplanation`, and `VersionManifest` refs. |
| `070-AC-005` | Every identity split that affects gold or graph output emits `GraphCorrectionHandoff` and no resolver graph mutation. |
| `070-AC-006` | Authoritative promotion fails while any resolver closure validation row is missing, blocked, not run, failed, checksum-mismatched, or has a `TODO` expected output. |
| `070-AC-007` | Two independent implementations produce byte-identical candidate ordering, identity decisions, resolver explanations, split handoffs, manifest refs, and replay checksums from the same active resolver catalog and inputs. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `070-TODO-RESOLVER-ACTIVE-ROW-SETS` | TODO: Provide active resolver profile row sets, evidence class row sets, identifier scope row sets, candidate generation profiles, hard blocker row sets, generation boundary row sets, decision matrix row sets, confidence band row sets, review routing policy, split policy, explanation policy, selector safety policy, activation report policy, fixture checksums, and expected output checksums. | Production `IdentityDecision`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `IdentityReviewCase`, `ResolverExplanation`, and `GraphCorrectionHandoff` output. | `070` resolver governance plus `100` package-set activation and `120` identity closure fixtures. | Identity output remains blocked with the most specific resolver artifact error. |
