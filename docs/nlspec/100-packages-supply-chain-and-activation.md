---
doc_id: CADASTRE-NLSPEC-100
title: Packages, Supply Chain, and Activation
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define package artifact identity, release manifests, package-set activation, trust, compatibility, rollback, quarantine, and emergency behavior.

## Explicit Non-Scope

- Package implementation language.
- Registry product selection.
- Graph backend product selection.
- Domain behavior outside package activation and execution permissions.

## Imports

- `AuthorityClass`
- `LifecycleStatus`
- `ValidationMatrix`
- `VersionManifest`
- `ActivationControlledArtifactRef`

## Exports

- `PackageArtifact`
- `PackageType`
- `PackageTypePolicyRow`
- `PackageTypePolicyRowSet`
- `ResolvePackageTypePolicyRow`
- `PackageReleaseManifest`
- `ProductionPackageSetManifest`
- `PackageCohesionGroup`
- `PackageRepositoryModel`
- `PackageRepositoryModelRow`
- `PackageTrustPolicy`
- `PackageTrustPolicyRow`
- `PackageRepositoryMetadata`
- `PackageRepositorySnapshot`
- `PackageRepositoryFreshnessProof`
- `PackageRepositoryAntiRollbackState`
- `PackageTransparencyEvidencePolicy`
- `PackageTransparencyEvidencePolicyRow`
- `PackageSignatureVerificationResult`
- `PackageAttestationSet`
- `PackageProvenancePolicy`
- `PackageProvenancePolicyRow`
- `PackageBuildProvenance`
- `PackageSBOMPolicy`
- `PackageSBOMPolicyRow`
- `PackageSBOMRef`
- `PackageDependencyLockPolicy`
- `PackageDependencyLockPolicyRow`
- `PackageCompatibilityMatrix`
- `PackageCompatibilityMatrixRow`
- `PackagePromotionGatePolicy`
- `PackagePromotionRecord`
- `PackageActivationFailureEvent`
- `PackageDeploymentRevision`
- `LastKnownGoodHealthGate`
- `LastKnownGoodPackageSet`
- `RollbackCompatibilityPolicy`
- `PackageRollbackPlan`
- `PackageRollbackResult`
- `QuarantineScopePolicy`
- `PackageQuarantineRecord`
- `EmergencyPackageOverrideRecord`
- `PackageStageBinding`
- `PackageDeveloperContract`
- `ActivatePackageSet`
- `RollbackPackageSet`
- `QuarantinePackage`
- `PackageSetActivationLifecycleMachine`
- `PackageDeprecationWindowPolicy`
- `PackageDeprecationWindowPolicyRow`

## Activation Unit

Production package runtime behavior may activate only through an immutable `ProductionPackageSetManifest`. A single `PackageArtifact`, package version string, deployment revision label, mutable repository ref, scalar signature status, dependency lock, SBOM existence, provenance existence, or successful validation run is not a production activation target.

Before a package release can participate in validation, compatibility, stage binding, rollback, quarantine, emergency override, or health gating, activation must resolve exactly one `PackageTypePolicyRow` for `PackageReleaseManifest.package_type` and the target environment. Unknown package types, known package types with no active policy row, and ambiguous package type policy resolution fail before any candidate production output.

## Package Type Policy Contract

`PackageType` is a lower snake-case enum token used only for package activation policy resolution. Package names, module names, artifact filenames, repository paths, version strings, and developer labels must not be used as package type substitutes.

`PackageType` is the confirmed closed enum below. Unknown tokens fail with `PACKAGE_TYPE_UNKNOWN`.

```text
lakehouse_feed_package
parser_package
mapping_bundle
source_schema_import_profile
semantic_overlay_artifact
mapping_validation_rule_set
mapping_project_manifest
mapping_compiler_pipeline
canonical_validation_output_schema
validation_scenario
toolchain_dependency_review
external_tool_capability_evidence
mapping_toolchain_package
external_schema_profile
lakehouse_read_policy
lakehouse_feed_completeness_profile
declared_dag_subset_profile
resolver_profile
analysis_rule_bundle
analysis_rule_row_set
rule_graph_compatibility_matrix
derivation_rule_bundle
derived_graph_edge_rule_set
threat_intel_enrichment_profile
threat_intel_distribution_mapping_policy
threat_intel_artifact_package
lineage_facet_mapping_policy
artifact_class_policy_row_set
registry_governance_artifact
registry_custom_property_schema
registry_classification_policy
policy_bundle
target_selector_safety_policy
graph_edge_semantics
graph_taxonomy_translation_policy
graph_projection_profile
graph_read_model_schema_profile
graph_apply_profile
graph_backend_provider_package
graph_provider_adapter_package
graph_backend_driver_package
graph_storage_backend_adapter_package
graph_index_backend_adapter_package
graph_backend_runtime_distribution
graph_backend_deployment_profile
structural_global_node_alias_policy
cim_projection_profile
projection_loss_policy
```

Broad package labels such as `deployment_profile`, module names, artifact filenames, repository paths, and developer labels are not package types. They must fail with `PACKAGE_TYPE_UNKNOWN` unless mapped by a future owner-approved enum patch.

Package type resolution error codes are:

| Error code | Required use |
| --- | --- |
| `PACKAGE_TYPE_UNKNOWN` | `PackageReleaseManifest.package_type` is not one confirmed `PackageType` token. |
| `PACKAGE_TYPE_POLICY_MISSING` | The package type is known but no active `PackageTypePolicyRow` covers it for the target environment. |
| `PACKAGE_TYPE_POLICY_AMBIGUOUS` | More than one equally specific active policy row covers the package type and target environment. |

### PackageTypePolicyRow schema

`PackageTypePolicyRowSet` is an activation-controlled artifact represented by `030.ActivationControlledArtifactRef` with `artifact_class = package_type_policy_row_set`. Concrete rows instantiate stable package behavior; they must not define new authority classes, lifecycle states, stage classes, record classes, trust semantics, graph semantics, identity semantics, or API states.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable ID scoped to the active row set. |
| `package_type` | Yes | none | One confirmed `PackageType` enum token. |
| `public_api_boundary` | Yes | none | Prose contract or imported owner contract ref. |
| `runtime_protocol` | No | `none` | Required for executable packages. |
| `allowed_stage_classes` | No | `[]` | Empty means no production stage execution. |
| `allowed_output_record_classes` | No | `[]` | Empty means no production records may be emitted. |
| `dependency_lock_policy` | No | `required_when_dependencies_affect_output_or_validation` | `required` for code-bearing executable packages. |
| `validation_matrix_requirements` | Yes | none | Must include positive, negative, replay, rollback, forbidden-output, and package-specific validation classes. |
| `schema_compatibility_inputs` | No | `[]` | Required when schema output can change. |
| `graph_compatibility_inputs` | No | `[]` | Required when graph output, graph query, graph apply, or analysis output can change. |
| `trust_policy_requirement` | No | `required` | Trust policy is required unless a stable owner row explicitly blocks activation. |
| `attestation_requirement` | No | `required` | Declarative-only exceptions must use `not_applicable_declarative_only`. |
| `sbom_requirement` | No | `required_for_code_or_dependencies` | Declarative-only exceptions must be explicit. |
| `repository_form_allowlist` | No | `oci_digest`, `tuf_compatible_metadata`, `local_bundle` | `git_tree_snapshot` is validation-only or provenance-only for MVP. |
| `deprecation_policy_ref` | Yes | none | Ref to active `PackageDeprecationWindowPolicyRow`. |
| `rollback_compatibility_policy_ref` | Yes | none | Ref to active `RollbackCompatibilityPolicy`. |
| `last_known_good_health_gate_ref` | Yes | none | Ref to active `LastKnownGoodHealthGate`. |
| `quarantine_scope_policy_ref` | Yes | none | Ref to active `QuarantineScopePolicy`. |
| `emergency_override_allowed_effects` | Yes | none | Subset of allowed emergency effects in this spec. |
| `validation_refs` | Yes | none | Non-empty refs to passing validation rows. |
| `activation_scope` | Yes | none | Target environment and package scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### ResolvePackageTypePolicyRow

```text
ResolvePackageTypePolicyRow(package_type, target_environment, active_row_set):
1. Validate active row-set ref through `030.ActivationControlledArtifactRef`.
2. If `package_type` is not one confirmed `PackageType` token, fail with `PACKAGE_TYPE_UNKNOWN`.
3. Keep only active rows whose `activation_scope` covers `target_environment`.
4. Match `package_type` exactly.
5. If zero rows remain, fail with `PACKAGE_TYPE_POLICY_MISSING`.
6. If more than one equally specific row remains, fail with `PACKAGE_TYPE_POLICY_AMBIGUOUS`.
7. Return the selected row and require its ref and checksum in `030.VersionManifest.included_refs`.
```

Every confirmed `PackageType` token must have exactly one active policy row in the active row set for each target environment in which package activation is in scope. A package activation implementation must run this resolver before compatibility checks, validation checks, stage binding, rollback preflight, quarantine evaluation, or health gating.

### Graph backend package type policy rows

These rows are package type policies for confirmed `PackageType` tokens. They must not define graph semantics; they gate runtime artifacts required by `090.GraphBackendProfile`.

| Package type | Public API boundary | Runtime protocol | Required compatibility inputs |
| --- | --- | --- | --- |
| `graph_backend_provider_package` | `090.GraphBackendProfile` and provider adapter contract | provider runtime | provider version, license, SBOM, vulnerabilities, storage/index compatibility |
| `graph_provider_adapter_package` | `090.GraphProviderAdapterContract` | driver/client protocol | query translation, apply, rebuild, errors, raw-write bypass |
| `graph_backend_driver_package` | provider driver API | TinkerPop/Gremlin driver for JanusGraph MVP | driver version, provider version, serialization compatibility |
| `graph_storage_backend_adapter_package` | provider storage adapter | storage backend API | durable mode, topology, lock/ID behavior, failure modes |
| `graph_index_backend_adapter_package` | provider index adapter | index provider API | index freshness, mapping, query feature compatibility |
| `graph_backend_runtime_distribution` | runtime distribution | server/container/distribution | package checks, startup config, health, rollback |
| `graph_backend_deployment_profile` | `090.GraphBackendProfile` deployment refs | deployment configuration | server mode, storage config, index config, schema-init config, health |

### Analysis, enrichment, lineage, and registry package type policy rows

These rows are package type policies for confirmed `PackageType` tokens. They make package-supplied `130` artifacts eligible only through package type policy, package-set membership, compatibility rows, validation refs, trust evidence, and `030.VersionManifest` inclusion.

| Package type | Public API boundary | Allowed stage classes | Allowed output record classes | Required activation artifact refs |
| --- | --- | --- | --- | --- |
| `analysis_rule_bundle` | `130.AnalysisRuleBundle` | `analysis` | `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord`, analysis execution summary | `analysis_rule_bundle`, `rule_graph_compatibility_matrix`, `analysis_output_replay_policy_row_set` |
| `analysis_rule_row_set` | `130.AnalysisRule` row set inside bundle | `analysis` | Same as parent bundle, only through bundle activation | `analysis_rule_row_set`, parent `analysis_rule_bundle` |
| `rule_graph_compatibility_matrix` | `130.RuleGraphCompatibilityMatrix` | none or validation-only | No production record output | `rule_graph_compatibility_matrix` |
| `derivation_rule_bundle` | `130.DerivationRuleBundle` and `080.DeriveFacts` handoff | `gold_derivation` | No direct package output; may invoke `080` only | `derivation_rule_bundle` |
| `derived_graph_edge_rule_set` | `130.DerivedGraphEdgeRule` and `090` handoff | `analysis`, `graph_projection` only through owner handoff | No direct graph mutation | `derived_graph_edge_rule_row_set` |
| `threat_intel_enrichment_profile` | `130.ThreatIntelEnrichmentProfile` | `enrichment` | `ThreatIntelEnrichmentRecord` | `threat_intel_enrichment_profile` |
| `threat_intel_distribution_mapping_policy` | `130.ThreatIntelDistributionMappingPolicy` | none or `enrichment` support only | No direct production output | `threat_intel_distribution_mapping_policy` |
| `threat_intel_artifact_package` | `130.ThreatIntelArtifactRef` through active profile | `enrichment` | `ThreatIntelEnrichmentRecord` only through active profile | `threat_intel_artifact_ref_policy`, `threat_intel_enrichment_profile` |
| `lineage_facet_mapping_policy` | `130.LineageFacetMappingPolicy` | `lineage` | `RunDatasetIOContract` | `lineage_facet_mapping_policy` |
| `artifact_class_policy_row_set` | `130.ArtifactClassPolicy` | none or validation-only | No production record output | `artifact_class_policy_row_set` |
| `registry_governance_artifact` | `130.RegistryArtifactGovernance` | none | Activation metadata only | `registry_governance_artifact` |
| `registry_custom_property_schema` | `130.RegistryCustomPropertySchema` | none | Activation metadata only | `registry_custom_property_schema` |
| `registry_classification_policy` | `130.RegistryClassificationPolicy` | none | Activation metadata only | `registry_classification_policy` |

The unresolved global `PackageType` enum blocker applies to every row in this subsection. Validation fixtures may use these candidate rows to prove fail-closed behavior, but authoritative package activation must fail until product governance confirms the canonical enum.

## Package Release Manifest

`PackageReleaseManifest` must name one immutable package release, including package type, artifact digest, size, media type, subject digest, release version, dependency locks, public API version when applicable, package developer contract, stage bindings, validation matrix refs, supply-chain evidence refs, compatibility refs, and release checksum.

## Package Set Manifest

`ProductionPackageSetManifest` must name the full coherent set of package release manifests for a production activation. It must include cohesion groups, target environment, activation mode, validation report refs, trust policy refs, compatibility matrix refs, rollback target refs, approval refs, and package-set checksum.

Partial package-set activation is forbidden unless a future accepted NLSpec defines partial semantics, output boundaries, rollback rules, and acceptance criteria.

## Package-supplied activation artifact boundary

`ProductionPackageSetManifest` may include activation-controlled artifact refs only as immutable `030.ActivationControlledArtifactRef` rows. A package set grants execution eligibility, not product authority. Runtime behavior still comes from owner specs and active artifact rows.

Package manifests must not define new canonical fields, authority classes, identity semantics, temporal semantics, graph semantics, API states, validation report shapes, or trust semantics. A package set that contains a valid package release but invalid artifact refs must fail activation or block the dependent stage before output.

A `GraphBackendProfile` that references a package release or package set whose trust, SBOM, provenance, dependency, license, vulnerability, compatibility, rollback, quarantine, or health gate fails must fail graph backend preflight with `GRAPH_BACKEND_PACKAGE_GATE_FAILED` before graph mutation, query serving, rebuild promotion, or drift check.

Package-supplied resolver artifacts must not redefine stable `070` identity semantics, hard-blocker precedence, decision states, confidence-band semantics, review terminality, split-policy semantics, selector safety, explanation checksum policy, or weak-evidence defaults. A package may instantiate active resolver row sets only through artifact classes permitted by `030.ActivationControlledArtifactRef`, owner spec `070`, exact checksums, active lifecycle status, activation scope, and passing identity validation refs.

Package-supplied analysis, enrichment, lineage, registry, and derived-edge artifacts must not redefine stable `130` non-authority rules, `080` gold derivation behavior, `090` graph projection behavior, `110` API/error/redaction behavior, or `120` validation shapes. A package may instantiate active `130` row sets only through the package type policy rows in this spec, immutable `ProductionPackageSetManifest` membership, `030.ActivationControlledArtifactRef`, exact checksums, active lifecycle status, activation scope, package compatibility rows, and passing `120` validation refs. Missing policy, missing package-set ref, unknown package type, ambiguous policy, compatibility failure, or manifest omission fails before production output.

## Trust Verification

`PackageSignatureVerificationResult` must be structured evidence. Scalar summaries such as `signature_status = verified` must not authorize activation.

| Evidence | Required behavior |
| --- | --- |
| Artifact digest, size, media type | Must match acquired bytes and release manifest. |
| Subject digest | Must match package artifact or OCI subject when applicable. |
| Signer identity | Must match `PackageTrustPolicyRow` authorized signer row. |
| Trust root | Must be active and scoped to package namespace or repository. |
| Threshold rule | Must pass when policy requires multiple signatures. |
| Repository metadata | Must pass freshness and anti-rollback checks for supported repository forms. |
| Transparency evidence | Required when the selected `PackageTransparencyEvidencePolicyRow` requires it. |
| Attestation subject | Must match release subject digest. |
| SBOM subject | Must match release subject digest. |

A cryptographically valid signature from an unauthorized signer must fail closed with `PACKAGE_SIGNER_UNAUTHORIZED`. A valid signature under an inactive trust root must fail with `PACKAGE_TRUST_ROOT_INACTIVE`. A threshold failure must fail with `PACKAGE_SIGNATURE_THRESHOLD_FAILED`.

### PackageTrustPolicyRow schema

| Field | Required behavior |
| --- | --- |
| `policy_row_id` | Stable row ID scoped to the trust policy row set. |
| `owner` | Product governance owner for the trust policy. |
| `trust_roots` | Non-empty active trust root refs and allowed root scopes. |
| `signer_identities` | Authorized signer identities scoped by package namespace, repository, and package type. |
| `issuer_claim_constraints` | Required issuer, subject, audience, and claim constraints when identity claims are used. |
| `threshold_rules` | Required signature counts, role separation, and signer independence rules. |
| `rotation_overlap` | New roots must overlap old roots for the declared overlap window before old root deactivation. |
| `deactivation_behavior` | Inactive roots and inactive signers fail new activation. |
| `approval_authority_separation` | Approval refs must not be emitted by the same authority class that supplies cryptographic verification. |
| `compromise_behavior` | Compromise rows are non-retryable for new activation until a successor policy row is active. |
| `validation_refs` | Non-empty refs proving authorized signer, unauthorized signer, inactive root, threshold, rotation, and compromise cases. |

### PackageTransparencyEvidencePolicyRow schema

| Field | Required behavior |
| --- | --- |
| `policy_row_id` | Stable row ID scoped to the transparency policy row set. |
| `transparency_required` | Boolean. Default `true` when package type policy requires transparency. |
| `accepted_proof_forms` | Closed list of accepted proof classes, such as provider log bundle, signed bundle, or owner-approved offline checkpoint. |
| `subject_digest` | Must equal release subject digest or package artifact digest as declared by the repository form. |
| `issuer` | Must match trust policy issuer constraints when issuer claims are used. |
| `timestamp` | Must be inside policy validity and must not be used as artifact freshness by itself. |
| `log_checkpoint` | Required when proof form uses a transparency log checkpoint. |
| `offline_verification_behavior` | `fail_closed` by default when the required proof cannot be verified offline. |
| `unsupported_provider_behavior` | `fail_closed` unless the row explicitly permits `not_applicable_declarative_only`. |
| `expiration` | Expired transparency evidence fails activation. |

### PackageProvenancePolicyRow schema

| Field | Required behavior |
| --- | --- |
| `policy_row_id` | Stable row ID scoped to the provenance policy row set. |
| `predicate_type` | Required predicate or attestation class. |
| `builder_identity` | Required authorized builder identity or explicit `not_applicable_declarative_only`. |
| `build_type` | Required build type allowlist. |
| `material_rules` | Required material digest, source, and dependency inclusion rules. |
| `product_subject_matching` | Provenance subject digest must equal release subject digest. |
| `allowed_build_systems` | Required allowlist when build systems affect provenance trust. |
| `expiration` | Expired provenance fails activation. |

### PackageSBOMPolicyRow schema

| Field | Required behavior |
| --- | --- |
| `policy_row_id` | Stable row ID scoped to the SBOM policy row set. |
| `supported_formats` | Required list of accepted SBOM formats; this spec does not select a concrete format. |
| `subject_matching` | SBOM subject digest must equal release subject digest. |
| `component_completeness` | Required component coverage rule. |
| `dependency_completeness` | Required dependency graph coverage rule. |
| `opaque_dependency_behavior` | Default `fail_closed` for code-bearing packages unless the row explicitly permits a bounded exception. |
| `license_gate` | Required when license policy affects activation. |
| `vulnerability_gate` | Required when vulnerability policy affects activation. |
| `expiration` | Expired SBOM evidence fails activation. |

### PackageDependencyLockPolicyRow schema

Dependency locks are reproducibility evidence only. A dependency lock must not satisfy trust, provenance, SBOM, compatibility, validation, or activation proof.

| Field | Required behavior |
| --- | --- |
| `policy_row_id` | Stable row ID scoped to the dependency lock policy row set. |
| `requirement_mode` | `required`, `required_when_dependencies_affect_output_or_validation`, or `not_applicable_declarative_only`. |
| `lock_subject` | Package release, build input, or runtime dependency graph covered by the lock. |
| `lock_checksum` | Required when a lock is required. |
| `resolved_dependency_digests` | Required for code-bearing packages when dependencies affect output or validation. |
| `live_resolution_behavior` | Production live dependency resolution is forbidden and fails with `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN`. |
| `mismatch_behavior` | Missing lock fails with `PACKAGE_DEPENDENCY_LOCK_MISSING`; checksum or digest mismatch fails with `PACKAGE_DEPENDENCY_LOCK_MISMATCH`. |

## Activation Algorithm

```text
ActivatePackageSet(candidate_set, current_active_set):
1. Verify candidate_set immutable checksum and release manifest checksums.
2. Verify package cohesion groups are complete.
3. For each release, resolve exactly one PackageTypePolicyRow through ResolvePackageTypePolicyRow.
4. Verify repository_form is one form allowed by the selected PackageTypePolicyRow.
5. Reject production git_tree_snapshot releases with PACKAGE_REPOSITORY_FORM_UNSUPPORTED.
6. Verify repository metadata, freshness proof, anti-rollback state, descriptor, artifact digest, size, media type, and subject digest.
7. Verify PackageTrustPolicyRow, signatures, signer authorization, active trust roots, threshold rules, and transparency evidence.
8. Verify required attestations, build provenance, SBOM, license, vulnerability, dependency lock, and dependency graph gates.
9. Verify PackageCompatibilityMatrixRow coverage for every axis required by the selected PackageTypePolicyRow.
10. Verify PackageDeveloperContract and PackageStageBinding refs.
11. Verify every package-supplied activation-controlled artifact ref through 030.ActivationControlledArtifactRef.
12. Verify each artifact owner spec and artifact class is permitted by the selected PackageTypePolicyRow.
13. Verify artifact checksums, lifecycle status, validation refs, activation scope, and package-set membership.
14. When package-supplied artifacts have owner spec 070, verify identity artifact compatibility: permitted artifact class, stable weak-evidence defaults unchanged, no graph-key-only or weak-only merge authority, no hard-blocker weakening, no decision-state redefinition, no review terminality weakening, no split or explanation checksum policy redefinition, and required identity validation refs present.
15. Verify validation matrix and required negative fixtures.
16. Verify VersionManifest includes every package-set, release, repository, trust, transparency, attestation, provenance, SBOM, dependency lock, compatibility, validation, stage binding, rollback, quarantine, emergency, health, lifecycle, and approval ref that can affect output.
17. Persist PackagePromotionRecord with artifact refs.
18. If any check fails, keep current_active_set active, write no candidate production output, and emit PackageActivationFailureEvent with the most specific package failure code.
19. If all checks pass, activate package set and record PackageDeploymentRevision with artifact refs.
20. Do not mark LastKnownGoodPackageSet until every required LastKnownGoodHealthGate passes.
```

Version strings never authorize activation by themselves. Dependency locks never authorize activation by themselves. Emergency override records never satisfy package signature, trust, repository metadata, repository freshness, transparency, or anti-rollback verification for a new production activation.

## Package Set Activation Lifecycle

Machine ID: `100.PackageSetActivationLifecycleMachine.v1`.

States:

```text
candidate
validated
shadow_running
canary_running
activating
active
last_known_good
activation_failed
rollback_pending
deprecated
retired
quarantined
```

Events:

```text
promotion_checks_passed
promotion_checks_failed
start_shadow
shadow_passed
shadow_failed
start_canary
canary_passed
canary_failed
activate
activation_committed
activation_failed
post_activation_health_passed
post_activation_health_failed
rollback_requested
rollback_checks_passed
rollback_checks_failed
deprecate
retire
quarantine
retry_activation
replay_same_event
```

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `candidate` | `promotion_checks_passed` | `validated` | Persist `PackagePromotionRecord`. |
| `candidate` | `promotion_checks_failed` | `activation_failed` | Preserve current active set; no candidate production output. |
| `validated` | `start_shadow` | `shadow_running` | Shadow output only. |
| `shadow_running` | `shadow_passed` | `validated` | Persist shadow pass evidence. |
| `shadow_running` | `shadow_failed` | `activation_failed` | No production output. |
| `validated` | `start_canary` | `canary_running` | Canary output only. |
| `canary_running` | `canary_passed` | `validated` | Persist canary pass evidence. |
| `canary_running` | `canary_failed` | `activation_failed` | Current active set preserved. |
| `validated` | `activate` | `activating` | Activation lock and all gates required. |
| `activating` | `activation_committed` | `active` | Persist `PackageDeploymentRevision`. |
| `activating` | `activation_failed` | `activation_failed` | Current active set preserved. |
| `active` | `post_activation_health_passed` | `last_known_good` | Persist `LastKnownGoodPackageSet`. |
| `active` | `post_activation_health_failed` | `rollback_pending` | Do not mark last-known-good. |
| `active`, `last_known_good`, `activation_failed`, or `quarantined` | `rollback_requested` | `rollback_pending` | Immutable verified rollback target required. |
| `rollback_pending` | `rollback_checks_passed` | `active` | Persist `PackageRollbackResult`. |
| `rollback_pending` | `rollback_checks_failed` | prior state | No target activation. |
| `active` or `last_known_good` | `deprecate` | `deprecated` | Persist deprecation record. |
| `deprecated` | `retire` | `retired` | Persist retirement evidence. |
| any non-retired state | `quarantine` | `quarantined` | Block dependent activation. |
| `activation_failed` | `retry_activation` | `candidate` | Candidate checksum unchanged and retryable failure class required. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

Canary and shadow states must not write current production output, advance watermarks, or mark last-known-good.

## Rollback and Emergency Behavior

Rollback target must be an immutable verified `ProductionPackageSetManifest` checksum. Mutable tags, branches, ranges, deployment revision labels alone, rebuilt artifacts, or tip-of-branch states are forbidden rollback targets.

Emergency override may quarantine, retire, abort candidate activation, roll back to verified last-known-good, or extend deprecation windows. Emergency override must not bypass signature, trust policy, repository metadata, freshness, transparency, or anti-rollback verification for a new production activation.

### LastKnownGoodHealthGate schema

A package set may be active but not last-known-good. `LastKnownGoodPackageSet` may be written only after every required health gate passes.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `health_gate_id` | Yes | none | Stable ID scoped to the gate row set. |
| `package_type_scope` | No | all package types in package set | Scope used for gate evaluation. |
| `enabled_output_classes` | Yes | derived from package set before checksum | Must include every output class the package set can affect. |
| `required_successful_run_count` | No | `1` per enabled output class | Must be at least `1`. |
| `max_health_window` | No | `24h` | Health evidence outside the window does not mark LKG. |
| `critical_error_codes` | Yes | none | Package, validation, graph, replay, manifest, and security codes that block LKG. |
| `version_manifest_required` | No | `true` | Missing manifest blocks LKG. |
| `graph_apply_required_when_graph_enabled` | No | `true` | Graph-enabled output requires graph apply success. |
| `watermark_advancement_required` | No | `false` | When `false`, lack of watermark advancement alone does not block LKG. |
| `rollback_on_failure` | No | `deployment_profile_defined` | Automatic rollback is forbidden unless the deployment profile requires it and rollback gates pass. |

Post-activation health failure must not mark `LastKnownGoodPackageSet`. Automatic rollback is forbidden unless the deployment profile explicitly requires it and `RollbackCompatibilityPolicy` gates pass.

### RollbackCompatibilityPolicy schema

Rollback is a preflight-gated package operation, not a pointer switch.

| Gate | Required behavior |
| --- | --- |
| target identity | Target must be an immutable verified `ProductionPackageSetManifest` checksum. |
| trust | Target releases must pass current trust or an explicit rollback trust-allow row. |
| quarantine | Quarantine blocks rollback unless a row explicitly permits rollback. |
| dependency locks | Target locks must be available and checksum-valid. |
| state schema | Stage state, replay inputs, and schema refs must be compatible, or rollback is read-only historical replay. |
| replay retention | Required manifests, snapshots, commits, and datasets must be available. |
| graph | Graph rollback must use `090.GraphRebuildManifest` or validated graph delta reapply. |
| authority | Authority, absence, identity, graph semantics, and source completeness changes require compatibility rows proving no illegal widening. |
| health | Rollback target becomes LKG only after health gates pass. |

```text
RollbackPackageSet(target_manifest_ref, current_active_set, rollback_policy):
1. Validate target_manifest_ref is an immutable verified ProductionPackageSetManifest checksum.
2. Validate target release checksums, repository evidence, trust evidence, dependency locks, and quarantine state.
3. Validate state schema, replay retention, graph, authority, completeness, identity, and compatibility gates.
4. If any gate fails, preserve current_active_set and emit the most specific rollback package error.
5. If all gates pass, activate the rollback target and persist PackageRollbackResult.
6. Mark target last-known-good only after LastKnownGoodHealthGate passes.
```

### QuarantineScopePolicy schema

Quarantine records block dependent activation without mutating artifact bytes, signature verification results, repository metadata, or prior activation evidence.

Allowed `target_kind` values are:

```text
artifact_digest
release_manifest
package_set
signer
trust_root
repository_snapshot
package_namespace
package_type
```

| Target kind | Default blocks |
| --- | --- |
| `artifact_digest` | Any release manifest referencing the digest. |
| `release_manifest` | Any package set referencing the release manifest. |
| `package_set` | Activation, rollback selection, canary, shadow, and LKG marking for the set. |
| `signer` | New activation for packages requiring the signer; rollback only if policy permits. |
| `trust_root` | New activation under that root; rollback only if policy permits. |
| `repository_snapshot` | Acquisition, activation, rollback, and replay depending on that snapshot. |
| `package_namespace` | All packages in the namespace. |
| `package_type` | All packages of that type in the target environment. |

| Quarantine reason | Default expiration |
| --- | --- |
| `security_compromise` | null |
| `operational_quarantine` | `72h` |

Clearing quarantine must create a successor `PackageQuarantineRecord`. Clearing quarantine must not mutate the original record and must not transition directly to active.

```text
QuarantinePackage(record, active_policy):
1. Validate target_kind is allowed.
2. Validate target_ref checksum and package scope.
3. Materialize default block scope from target_kind unless an active policy row narrows it.
4. Persist immutable PackageQuarantineRecord.
5. Block dependent activation, rollback selection, canary, shadow, and LKG marking according to the materialized scope.
```

### EmergencyPackageOverrideRecord schema

Emergency package override is bounded operational control, not activation proof.

| Field | Required | Rule |
| --- | ---: | --- |
| `emergency_override_id` | Yes | Stable ID over target, type, approvals, TTL, and reason. |
| `override_type` | Yes | One allowed override type from the table below. |
| `target_ref` | Yes | Immutable target ref or package scope ref. |
| `requested_by` | Yes | Operator or service principal ref. |
| `quorum_approval_rows` | Yes | Non-empty approval refs. |
| `ttl_expires_at` | Yes | Must not exceed the type default or maximum below. |
| `forbidden_bypass_assertion` | Yes | Must be `true`. |
| `audit_reason` | Yes | Bounded string recorded in audit. |
| `created_at` | Yes | RFC3339 UTC. |
| `checksum` | Yes | SHA-256 over canonical record bytes excluding `checksum`. |

| Override type | Default TTL | Allowed effect |
| --- | ---: | --- |
| `quarantine_package` | `72h` | Block activation and optionally rollback or replay. |
| `retire_active_package` | `24h` | Block new production writes after safe handoff. |
| `abort_candidate_activation` | `24h` | Stop candidate activation and preserve current set. |
| `rollback_to_lkg` | `24h` | Select prior verified immutable manifest after rollback gates. |
| `extend_deprecation_window` | `30d max` | Permit replay, rollback, or migration use only. |

Emergency override must not bypass package signature, trust policy, repository metadata, repository freshness, transparency, or anti-rollback verification for a new production activation. Emergency override must not modify artifact bytes, rewrite verification result bytes, rewrite repository metadata, or mark a failed candidate successful.

## Package Activation Contract Details

### PackageRepositoryModel catalog

| Repository form | Resolver rule | Metadata rule | Descriptor rule | Digest rule | Evidence-attachment rule | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `oci_digest` | resolve by immutable digest only | repository metadata and referrers required when policy says | OCI descriptor required | subject digest and artifact digest must match manifest | signatures, attestations, SBOM attach to subject | candidate |
| `tuf_compatible_metadata` | resolve through trusted metadata roles | timestamp, snapshot, targets, delegation evidence required | target metadata required | target hash and length verified | trust evidence persists | candidate |
| `local_bundle` | resolve by local immutable checksum | local repository metadata required | bundle descriptor required | bundle checksum verified | evidence bundled and checksummed | candidate |
| `git_tree_snapshot` | validation-only or provenance-only in MVP | no production metadata authority | no production descriptor authority | no production digest authority | evidence may be retained for audit but not activation | `inactive_for_mvp`; reject production activation with `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` |

A package acquired from Git must be materialized into `oci_digest`, `tuf_compatible_metadata`, or `local_bundle` before production activation. A `PackageReleaseManifest` that directly references `repository_form = git_tree_snapshot` for production activation fails with `PACKAGE_REPOSITORY_FORM_UNSUPPORTED`.

Git commit timestamp, branch name, tag name, repository URL, or tree hash must not satisfy freshness, anti-rollback, artifact digest, signature, SBOM, provenance, compatibility, or rollback-target requirements for production activation.

### PackageTrustRootGovernance

| Field | Required behavior |
| --- | --- |
| trust root owner | Named product governance owner. |
| authorized signers | Explicit signer rows scoped by package namespace and repository. |
| rotation | New root must overlap old root and record effective time. |
| deactivation | Deactivated signer must fail new activation. |
| threshold rules | Policy-defined; failure is non-retryable until evidence changes. |
| issuers and scopes | OIDC/PKI issuers and claims must match policy. |
| approval authority | Separate from cryptographic verification. |

### PackageTransparencyEvidencePolicy

| Evidence form | Required behavior |
| --- | --- |
| provider transparency log bundle | Verify offline inclusion or fail closed when policy requires. |
| signed bundle | Verify subject, issuer, timestamp, and log checkpoint. |
| no proof supplied | Non-retryable failure when policy requires transparency. |
| unsupported provider | Fail closed unless policy row explicitly permits. |

### PackageRepositoryFreshnessAntiRollbackPolicy

| Repository type | Freshness default | Anti-rollback state |
| --- | --- | --- |
| `oci_digest` | metadata checked at acquisition and activation | highest-seen digest metadata when registry metadata is used |
| `tuf_compatible_metadata` | timestamp expiration enforced | highest-seen timestamp/snapshot/targets versions |
| `local_bundle` | bundle timestamp plus owner approval | bundle checksum allowlist |
| `git_tree_snapshot` | no production freshness proof in MVP | no production anti-rollback authority; fail `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` |

### Package evidence policies

| Policy | Required behavior |
| --- | --- |
| `PackageTrustPolicy` | Defines owners, trust roots, signer identities, namespace and repository scopes, issuer constraints, threshold rules, rotation, deactivation, authority separation, and compromise behavior. |
| `PackageTransparencyEvidencePolicy` | Defines whether transparency is required, accepted proof forms, subject digest, issuer, timestamp, log checkpoint, offline verification behavior, unsupported-provider behavior, and expiration. |
| `PackageProvenancePolicy` | Defines predicate type, builder identity, build type, material rules, subject matching, allowed build systems, and expiration. |
| `PackageSBOMPolicy` | Defines supported SBOM formats, subject matching, component/dependency completeness, opaque dependency behavior, license gates, vulnerability gates, and expiration. |
| `PackageDependencyLockPolicy` | Defines dependency lock requirement mode, lock checksum, resolved dependency digests, mismatch behavior, and live resolution prohibition. |
| `PackageCompatibilityMatrix` | Defines public API, runtime protocol, dependency lock, validation checksum, schema compatibility, graph compatibility, trust, attestation, SBOM, deployment target, and deprecation window per package type. |
| `PackageCohesionGroup` | Defines packages that activate, roll back, quarantine, and retire together. |
| `PackagePromotionGatePolicy` | Defines approvals, shadow outputs, canary thresholds, health gates, and failure precedence. |
| `LastKnownGoodHealthGate` | Defines post-activation health rows before last-known-good marking. |
| `RollbackCompatibilityPolicy` | Defines immutable rollback target and replay/schema/graph/trust/quarantine gates. |
| `EmergencyPackageOverrideRecord` | Allows quarantine, retirement, abort, verified rollback, or deprecation extension only. |
| `QuarantineScopePolicy` | Defines target scope, default blocks, expiration, and successor records. |

### PackageCompatibilityMatrixRow schema

`PackageCompatibilityMatrix` must be evaluated as a deterministic compatibility gate. SemVer, package version strings, deployment revision labels, and dependency locks must not authorize activation by themselves.

| Field | Required when output-affecting |
| --- | --- |
| `package_type` | Always required. |
| `package_version` | Always required and informational unless a row declares a comparison rule. |
| `artifact_digest` | Always required. |
| `contract_version` | Required when the package public contract can affect output. |
| `runtime_protocol_version` | Required for executable packages. |
| `dependency_lock_checksum` | Required when dependencies affect output or validation. |
| `source_schema_checksum` | Required when source schema output can change. |
| `validation_output_checksum` | Required when validation output can change. |
| `target_environment` | Always required. |
| `source_stage_binding` | Required when the package can execute in a stage. |
| `package_public_api_version` | Required when a public API boundary exists. |
| `graph_projection_profile_version` | Required when graph output can change. |
| `graph_read_model_schema_profile_version` | Required when graph serving schema can change. |
| `graph_apply_profile_version` | Required when graph apply can change. |
| `external_schema_profile_version` | Required when external schema validation can change. |
| `trust_policy_version` | Always required. |
| `attestation_policy_version` | Required when attestation is required. |
| `sbom_policy_version` | Required when SBOM is required. |
| `decision` | Required; closed values are `pass` or `fail`. |
| `decision_reason` | Required bounded explanation. |
| `validation_refs` | Non-empty refs. |

Every output-affecting axis named by `PackageTypePolicyRow` must have a passing compatibility row. A missing, expired, mismatched, or failed compatibility row emits `PACKAGE_COMPATIBILITY_FAILED` before activation or rollback.

### JanusGraph backend compatibility axes

| Compatibility axis | Required for JanusGraph MVP |
| --- | --- |
| provider runtime version | pinned immutable package or release ref |
| TinkerPop version | must match `090.GraphBackendProfile` |
| Gremlin driver version | must match provider adapter contract |
| storage adapter | must match selected storage backend kind/version |
| index adapter | must match selected index backend kind/version |
| schema profile | must match `090.GraphReadModelSchemaProfile` checksum |
| query translation | must match `090.GraphQueryTranslationProfile` checksum |
| graph apply | must match `090.GraphApplyProfile` checksum |
| license/SBOM/vulnerability | must pass package policy |
| rollback | must preserve graph provider compatibility or block rollback |

### Package type compatibility closure

Broad labels such as `feed reader`, `mapping`, `projection package`, and `validation package` must not be used for production compatibility selection. Compatibility inputs must come from the selected `PackageTypePolicyRow` and `PackageCompatibilityMatrixRow` keyed by a confirmed `PackageType` token.

A package type with no active policy row fails before activation with `PACKAGE_TYPE_POLICY_MISSING`. A known package type with multiple equally specific active rows fails before activation with `PACKAGE_TYPE_POLICY_AMBIGUOUS`. A package type not in the confirmed token list fails before policy resolution with `PACKAGE_TYPE_UNKNOWN`.

For every package-supplied `130` artifact, compatibility rows must include the selected package type policy row, public API boundary, allowed stage classes, allowed output record classes, required `030` artifact-class refs, package-set checksum, validation refs, and `VersionManifest` omission test. A package that supplies `130` artifacts without matching compatibility rows fails with `PACKAGE_COMPATIBILITY_FAILED` or `PACKAGE_VERSION_MANIFEST_INCOMPLETE` before output.

TODO: Replace broad legacy package labels in every package fixture and active package-set manifest with confirmed `PackageType` tokens before authoritative handoff.

### PackageDeprecationWindowPolicy

`PackageDeprecationWindowPolicyRow` is keyed by confirmed `PackageType`, not by broad package labels.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the policy row set. |
| `package_type` | Yes | none | One confirmed `PackageType` token. |
| `default_window` | Yes | none | Duration used when a release enters `deprecated`. |
| `minimum_window` | Yes | none | May be `0d` for immediate retirement or quarantine. |
| `maximum_window` | Yes | none | Emergency extension must not exceed this value. |
| `emergency_retirement_behavior` | Yes | none | Required quarantine or retirement behavior for trust, correctness, schema, graph, identity, or safety failures. |
| `compatibility_override_behavior` | Yes | none | Override must require explicit owner approval and validation evidence, and must not bypass trust. |
| `validation_refs` | Yes | none | Non-empty deprecation, expiry, emergency-extension, and rollback fixtures. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

A deprecated package must not be newly selected after its deprecation window expires except as an immutable verified rollback target allowed by `PackageRollbackPlan` and `RollbackCompatibilityPolicy`.

TODO: Product governance must provide one active `PackageDeprecationWindowPolicyRow` for every confirmed `PackageType` token before authoritative package activation handoff.

### PackageHealthApiHandoff

Package activation, rollback, quarantine, emergency, and last-known-good states are observable through health, audit, admin API, and package activation output. They must use structured owner errors and must not leak private artifact payloads.

| Package condition | Recommended API/health state | Required behavior |
| --- | --- | --- |
| candidate activation failure | `blocked` or `error` by failure code | Preserve current active package set and emit `PackageActivationFailureEvent`. |
| active but not last-known-good | `blocked` | Active package may serve according to activation result, but it is not rollback-eligible as last-known-good. |
| rollback blocked | `blocked` | Preserve current active set; emit rollback owner error. |
| quarantine blocked | `blocked` | No package activation or artifact mutation. |
| emergency trust bypass forbidden | `security_error` | New production activation must not bypass signature, trust, repository freshness, transparency, or anti-rollback verification. |

Caller-visible outputs must not leak private artifact payloads, private repository paths, signer secrets, raw SBOM bytes, unauthorized package evidence, or private source bindings.

### Package activation failure codes

| Error code | Emitted when |
| --- | --- |
| `PACKAGE_TYPE_UNKNOWN` | Release manifest package type is not one confirmed `PackageType` token. |
| `PACKAGE_TYPE_POLICY_MISSING` | Known package type has no active policy row covering the target environment. |
| `PACKAGE_TYPE_POLICY_AMBIGUOUS` | More than one equally specific active package type policy row covers the release. |
| `PACKAGE_ACTIVATION_ARTIFACT_MISSING` | A required package-supplied activation artifact ref is missing. |
| `PACKAGE_ACTIVATION_ARTIFACT_OWNER_MISMATCH` | A package-supplied artifact class is not permitted by the owner spec or selected package type policy row. |
| `PACKAGE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | A package-supplied artifact checksum mismatches the release, artifact ref, or package-set manifest. |
| `PACKAGE_ACTIVATION_ARTIFACT_SCOPE_MISMATCH` | A package-supplied artifact does not cover the activation scope. |
| `PACKAGE_ACTIVATION_ARTIFACT_VALIDATION_MISSING` | Required artifact validation refs are missing or not passing. |
| `PACKAGE_ACTIVATION_ARTIFACT_CORE_CONFLICT` | Package-supplied activation artifact conflicts with an owner stable core contract; maps to `010.ACTIVATION_ARTIFACT_CORE_CONFLICT` for shared boundary handling. |
| `PACKAGE_SET_CHECKSUM_MISMATCH` | Candidate package set checksum or release manifest checksum mismatches. |
| `PACKAGE_COHESION_INCOMPLETE` | A cohesion group is incomplete. |
| `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | Repository form is inactive or unsupported for MVP production activation. |
| `PACKAGE_REPOSITORY_ROLLBACK_DETECTED` | Repository metadata regresses under anti-rollback state. |
| `PACKAGE_REPOSITORY_METADATA_EXPIRED` | Required repository metadata, timestamp, snapshot, target metadata, or local bundle metadata is expired. |
| `PACKAGE_SIGNER_UNAUTHORIZED` | Cryptographic signature is valid but signer is not authorized. |
| `PACKAGE_TRUST_ROOT_INACTIVE` | Required trust root is inactive, deactivated, expired, or outside scope. |
| `PACKAGE_SIGNATURE_THRESHOLD_FAILED` | Required signature threshold rule does not pass. |
| `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | Required transparency proof is missing, unsupported, expired, or unverifiable. |
| `PACKAGE_ATTESTATION_MISSING` | Required attestation is missing. |
| `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | Attestation subject does not match release subject digest. |
| `PACKAGE_PROVENANCE_POLICY_FAILED` | Build provenance predicate, builder, material, product, build system, or expiration rule fails. |
| `PACKAGE_SBOM_MISSING` | Required SBOM ref is missing. |
| `PACKAGE_SBOM_SUBJECT_MISMATCH` | SBOM subject does not match release subject digest. |
| `PACKAGE_SBOM_POLICY_FAILED` | SBOM format, component, dependency, opaque dependency, license, vulnerability, or expiration rule fails. |
| `PACKAGE_DEPENDENCY_LOCK_MISSING` | Required dependency lock is missing. |
| `PACKAGE_DEPENDENCY_LOCK_MISMATCH` | Dependency lock checksum or resolved dependency digest mismatches. |
| `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | Production activation attempts live dependency resolution. |
| `PACKAGE_COMPATIBILITY_FAILED` | Any required compatibility matrix axis is missing, expired, mismatched, or failed. |
| `PACKAGE_VALIDATION_FAILED` | Required validation row fails, is blocked, or is missing. |
| `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | Required package activation refs are missing from `030.VersionManifest.included_refs`. |
| `PACKAGE_LKG_HEALTH_GATE_FAILED` | Post-activation health gate fails or required health evidence is missing within the health window. |
| `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | Package lifecycle event is invalid for the prior state. |
| `PACKAGE_ACTIVATION_IDEMPOTENCY_CONFLICT` | Package activation idempotency key is reused with different input checksums. |
| `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | Canary execution attempts current production output, watermark advancement, or last-known-good state. |
| `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | Shadow execution attempts current production output, watermark advancement, or last-known-good state. |
| `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | Rollback target is mutable, missing, quarantined without permission, or not a verified package-set manifest. |
| `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | Rollback state schema, stage state, or replay inputs are incompatible. |
| `PACKAGE_ROLLBACK_REPLAY_INPUT_INSUFFICIENT` | Required rollback manifests, snapshots, commits, datasets, or replay inputs are unavailable. |
| `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | Required graph rebuild or graph delta reapply evidence is missing or incompatible. |
| `PACKAGE_ROLLBACK_TRUST_INCOMPATIBLE` | Rollback target fails current trust or lacks an explicit rollback trust-allow row. |
| `PACKAGE_ROLLBACK_QUARANTINE_BLOCKED` | Quarantine blocks rollback target selection. |
| `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | Dependent activation references a quarantined package, artifact, signer, trust root, repository snapshot, namespace, or package type. |
| `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | Deprecated artifact is selected after the applicable deprecation window without verified rollback authorization. |
| `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | Emergency override attempts to bypass package signature, trust, repository metadata, freshness, transparency, or anti-rollback verification for new production activation. |
| `PACKAGE_RETRY_NOT_ALLOWED` | Retry is requested for a non-retryable activation failure or changed candidate checksum. |

### PackageErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | default_severity | default_retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `PACKAGE_TYPE_UNKNOWN` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-type-unknown` |
| `PACKAGE_TYPE_POLICY_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-type-policy-missing` |
| `PACKAGE_TYPE_POLICY_AMBIGUOUS` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-type-policy-ambiguous` |
| `PACKAGE_ACTIVATION_ARTIFACT_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-missing` |
| `PACKAGE_ACTIVATION_ARTIFACT_OWNER_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-owner-mismatch` |
| `PACKAGE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-checksum-mismatch` |
| `PACKAGE_ACTIVATION_ARTIFACT_SCOPE_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-scope-mismatch` |
| `PACKAGE_ACTIVATION_ARTIFACT_VALIDATION_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-validation-missing` |
| `PACKAGE_ACTIVATION_ARTIFACT_CORE_CONFLICT` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-artifact-core-conflict` |
| `PACKAGE_SET_CHECKSUM_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-set-checksum-mismatch` |
| `PACKAGE_COHESION_INCOMPLETE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-cohesion-incomplete` |
| `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-repository-form-unsupported` |
| `PACKAGE_REPOSITORY_ROLLBACK_DETECTED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-repository-rollback-detected` |
| `PACKAGE_REPOSITORY_METADATA_EXPIRED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-repository-metadata-expired` |
| `PACKAGE_SIGNER_UNAUTHORIZED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-signer-unauthorized` |
| `PACKAGE_TRUST_ROOT_INACTIVE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-trust-root-inactive` |
| `PACKAGE_SIGNATURE_THRESHOLD_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-signature-threshold-failed` |
| `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-transparency-evidence-missing` |
| `PACKAGE_ATTESTATION_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-attestation-missing` |
| `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-attestation-subject-mismatch` |
| `PACKAGE_PROVENANCE_POLICY_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-provenance-policy-failed` |
| `PACKAGE_SBOM_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-sbom-missing` |
| `PACKAGE_SBOM_SUBJECT_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-sbom-subject-mismatch` |
| `PACKAGE_SBOM_POLICY_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-sbom-policy-failed` |
| `PACKAGE_DEPENDENCY_LOCK_MISSING` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-dependency-lock-missing` |
| `PACKAGE_DEPENDENCY_LOCK_MISMATCH` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-dependency-lock-mismatch` |
| `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-dependency-live-resolution-forbidden` |
| `PACKAGE_COMPATIBILITY_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-compatibility-failed` |
| `PACKAGE_VALIDATION_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-validation-failed` |
| `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-version-manifest-incomplete` |
| `PACKAGE_LKG_HEALTH_GATE_FAILED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-lkg-health-gate-failed` |
| `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-lifecycle-illegal-transition` |
| `PACKAGE_ACTIVATION_IDEMPOTENCY_CONFLICT` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-activation-idempotency-conflict` |
| `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-canary-output-forbidden` |
| `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-shadow-output-forbidden` |
| `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-target-unverified` |
| `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-state-incompatible` |
| `PACKAGE_ROLLBACK_REPLAY_INPUT_INSUFFICIENT` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-replay-input-insufficient` |
| `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-graph-incompatible` |
| `PACKAGE_ROLLBACK_TRUST_INCOMPATIBLE` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-trust-incompatible` |
| `PACKAGE_ROLLBACK_QUARANTINE_BLOCKED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-rollback-quarantine-blocked` |
| `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-quarantine-blocked-activation` |
| `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-deprecation-window-expired` |
| `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-emergency-package-bypass-forbidden` |
| `PACKAGE_RETRY_NOT_ALLOWED` | `100` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `100.PackageErrorContext` | `package-error-package-retry-not-allowed` |

### ProductionPackageSetManifest schema

| Field | Required | Rule |
| --- | ---: | --- |
| `package_release_refs` | Yes | Canonically sorted release manifest refs and checksums. |
| `package_type_policy_refs` | Yes | Selected `PackageTypePolicyRow` refs and checksums for every release. |
| `repository_model_refs` | Yes | Repository model row refs, metadata refs, snapshot refs, freshness proof refs, and anti-rollback state refs. |
| `supply_chain_policy_refs` | Yes | Trust, transparency, provenance, SBOM, and dependency lock policy row refs. |
| `supply_chain_evidence_refs` | Yes | Signature verification result, transparency evidence, attestation, provenance, SBOM, and dependency lock refs. |
| `health_gate_refs` | Yes | Selected `LastKnownGoodHealthGate` refs and checksums. |
| `quarantine_scope_policy_refs` | Yes | Selected quarantine scope policy refs and checksums. |
| `emergency_override_refs` | Required when emergency action affects eligibility | Emergency override record refs and checksums. |
| `cohesion_groups` | Yes | Every group complete before activation. |
| `environment` | Yes | Target environment token. |
| `activation_mode` | Yes | production, canary, shadow, rollback, or emergency bounded action. |
| `activation_artifact_refs` | Yes | Immutable `030.ActivationControlledArtifactRef` rows for package-supplied artifacts. |
| `artifact_owner_specs` | Yes | Stable core owner specs for package-supplied artifacts. |
| `artifact_validation_refs` | Yes | Passing validation refs for package-supplied artifacts. |
| `artifact_compatibility_refs` | Yes | Package/artifact compatibility rows. |
| `artifact_scope` | Yes | Activation scope covered by artifact refs. |
| `artifact_registry_snapshot_ref` | Yes | Immutable registry snapshot containing artifact rows and checksums. |
| `trust_refs` | Yes | Trust policies and verification results. |
| `validation_refs` | Yes | Required validation report refs. |
| `compatibility_refs` | Yes | Compatibility matrix refs. |
| `rollback_refs` | Yes | Immutable rollback target refs or explicit null when not applicable. |
| `approval_refs` | Yes | Product approval refs; approval does not bypass trust. |
| `package_set_checksum` | Yes | SHA-256 over canonical manifest bytes excluding this field. |

### PackageActivationFailureEvent schema

| Field | Required | Rule |
| --- | ---: | --- |
| `event_id` | Yes | Deterministic event ID from candidate set checksum, failed check, and timestamp evidence. |
| `candidate_package_set_checksum` | Yes | Candidate checksum. |
| `current_active_package_set_checksum` | Yes | Current active set preserved after failure. |
| `failed_check` | Yes | Most specific failure code. |
| `failure_detail_ref` | Yes | Redacted evidence ref. |
| `candidate_output_visibility` | Yes | Must be `none` for production output. |
| `watermark_effect` | Yes | Must be `no_advancement`. |

### Package-set activation failure precedence

| Failed check | Emitted event | Current-active-set behavior | Candidate-output behavior | Retryability | Last-known-good effect |
| --- | --- | --- | --- | --- | --- |
| immutable checksum mismatch | `PackageActivationFailureEvent` | keep current | write none | no | unchanged |
| incomplete cohesion group | same | keep current | write none | no until manifest changes | unchanged |
| repository freshness/anti-rollback | same | keep current | write none | yes after metadata refresh unless rollback detected | unchanged |
| unauthorized signer | same | keep current | write none | no | unchanged |
| transparency/provenance/SBOM mismatch | same | keep current | write none | no until evidence changes | unchanged |
| compatibility failure | same | keep current | write none | no | unchanged |
| validation matrix failure | same | keep current | write no production output | yes after candidate changes | unchanged |
| post-activation health failure | same | keep activated only if policy permits, otherwise rollback plan | no new candidate output | policy-defined | do not mark last-known-good |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `100-ANALYSIS-REGISTRY-PACKAGE-TYPE-AC-001` | Each candidate `130` package type has exactly one active `PackageTypePolicyRow` in validation scope, and missing-policy, unknown-type, and ambiguous-policy fixtures fail closed before package activation. |
| `100-ANALYSIS-REGISTRY-PACKAGE-SET-AC-001` | Package-supplied `130` artifacts require immutable package-set membership, exact `030.ActivationControlledArtifactRef`, compatibility row, validation refs, and `VersionManifest` inclusion before output. |
| `100-ANALYSIS-REGISTRY-STAGE-OUTPUT-AC-001` | Package policy permits only the stage classes and output record classes listed in the `130` package type table and fails with `FORBIDDEN_STAGE_OUTPUT` or package-policy errors for every other output. |
| `100-CLEANUP-AC-001` | No banned reference class remains. |
| `100-CLEANUP-AC-002` | Production activation still targets immutable package sets, not individual package artifacts or scalar signature summaries. |
| `100-CLEANUP-AC-003` | Unauthorized signer with a cryptographically valid signature still fails closed. |
| `100-CLEANUP-AC-004` | Emergency override still cannot bypass signature or trust verification for a new production activation. |
| `100-PACKAGE-TYPE-POLICY-AC-001` | Every confirmed `PackageType` token has exactly one active `PackageTypePolicyRow` in the active row set for the target environment. |
| `100-PACKAGE-TYPE-POLICY-AC-002` | Unknown package type fails with `PACKAGE_TYPE_UNKNOWN`. |
| `100-PACKAGE-TYPE-POLICY-AC-003` | Known package type with no active policy row fails with `PACKAGE_TYPE_POLICY_MISSING`. |
| `100-PACKAGE-TYPE-POLICY-AC-004` | Ambiguous package type policy resolution fails with `PACKAGE_TYPE_POLICY_AMBIGUOUS`. |
| `100-PACKAGE-TYPE-ENUM-AC-001` | The confirmed enum has no duplicate tokens, contains no generic `deployment_profile`, and rejects unknown or legacy broad labels before package release validation. |
| `100-REPOSITORY-FORM-AC-001` | `git_tree_snapshot` is inactive for MVP production activation and fails with `PACKAGE_REPOSITORY_FORM_UNSUPPORTED`. |
| `100-REPOSITORY-FORM-AC-002` | A candidate release using `git_tree_snapshot` as production repository form fails with `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` and writes no candidate production output. |
| `100-PACKAGE-SET-MANIFEST-AC-001` | `ProductionPackageSetManifest` includes package release refs, selected package type policy refs, repository refs, supply-chain policy/evidence refs, cohesion groups, environment, activation mode, trust refs, validation refs, compatibility refs, rollback refs, quarantine refs, health refs, approval refs, and checksum. |
| `100-GRAPH-BACKEND-PACKAGE-GATE-AC-001` | Missing JanusGraph provider, adapter, driver, storage adapter, index adapter, runtime distribution, SBOM, provenance, compatibility, or package-set refs fail graph backend preflight with `GRAPH_BACKEND_PACKAGE_GATE_FAILED`. |
| `100-GRAPH-BACKEND-PACKAGE-GATE-AC-002` | Storage adapter version mismatch, index adapter version mismatch, or TinkerPop/driver compatibility mismatch fails candidate activation and preserves the current active package set. |
| `100-GRAPH-BACKEND-PACKAGE-GATE-AC-003` | Rollback to an incompatible graph backend package set fails before active state changes. |
| `100-COMPATIBILITY-AC-001` | Missing or failed compatibility on any required output-affecting axis fails activation or rollback with `PACKAGE_COMPATIBILITY_FAILED`. |
| `100-SBOM-AC-001` | Missing, subject-mismatched, expired, or policy-failed SBOM evidence fails activation with `PACKAGE_SBOM_MISSING`, `PACKAGE_SBOM_SUBJECT_MISMATCH`, or `PACKAGE_SBOM_POLICY_FAILED`. |
| `100-ATTESTATION-AC-001` | Missing or subject-mismatched attestation fails activation with `PACKAGE_ATTESTATION_MISSING` or `PACKAGE_ATTESTATION_SUBJECT_MISMATCH`. |
| `100-DEPENDENCY-LOCK-AC-001` | Missing dependency lock, dependency lock mismatch, or live dependency resolution fails with the corresponding dependency lock error and writes no production output. |
| `100-TRANSPARENCY-AC-001` | Missing required transparency evidence fails with `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING`. |
| `100-TRUST-THRESHOLD-AC-001` | Inactive trust root or failed signature threshold fails with `PACKAGE_TRUST_ROOT_INACTIVE` or `PACKAGE_SIGNATURE_THRESHOLD_FAILED`. |
| `100-DEPRECATION-WINDOW-AC-001` | Every package type has explicit deprecation defaults, minimums, and maximums. |
| `100-DEPRECATION-WINDOW-AC-002` | Expired deprecation window blocks new activation with `PACKAGE_DEPRECATION_WINDOW_EXPIRED` unless verified rollback policy permits the target. |
| `100-LKG-HEALTH-AC-001` | `LastKnownGoodPackageSet` is not written until every required health gate passes. |
| `100-LKG-HEALTH-AC-002` | Post-activation health failure preserves active-state evidence but does not mark last-known-good. |
| `100-ROLLBACK-AC-001` | Mutable rollback targets fail before active state changes. |
| `100-ROLLBACK-AC-002` | Immutable target with incompatible state, graph, trust, replay, dependency lock, or authority refs fails before active state changes. |
| `100-QUARANTINE-AC-001` | Quarantine over any allowed target kind blocks dependent activation before output. |
| `100-QUARANTINE-AC-002` | Clearing quarantine creates successor evidence and cannot clear directly to active. |
| `100-EMERGENCY-AC-001` | Emergency signature or trust bypass for new production activation fails with `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN`. |
| `100-VERSION-MANIFEST-AC-001` | Package activation output fails with `PACKAGE_VERSION_MANIFEST_INCOMPLETE` when any required package-set, release, trust, repository, SBOM, provenance, compatibility, rollback, quarantine, emergency, health, lifecycle, or validation ref is omitted from `030.VersionManifest.included_refs`. |
| `100-VOLATILITY-AC-001` | Package set with mapping bundle wrong owner spec, resolver profile checksum mismatch, missing graph projection profile, or invalid artifact validation refs keeps the current active set and writes no candidate production output. |
| `100-VOLATILITY-AC-002` | `ProductionPackageSetManifest` includes activation artifact refs, owner specs, validation refs, compatibility refs, activation scope, and artifact registry snapshot refs when package-supplied artifacts can affect output. |
| `100-RESOLVER-ARTIFACT-AC-001` | A package-supplied resolver artifact that permits weak-only, selector-only, mapped-target-only, source-native-merge-history-only, or graph-key-only create, attach, or merge fails activation and preserves the current active set. |
| `100-RESOLVER-ARTIFACT-AC-002` | A package-supplied resolver artifact with missing identity validation refs fails activation before candidate production output. |
| `100-RESOLVER-ARTIFACT-AC-003` | Resolver profile checksum mismatch in a candidate package set fails activation, keeps the current active set, and writes no candidate production output. |
| `100-LIFECYCLE-AC-001` | Package activation lifecycle matrix is total. |
| `100-LIFECYCLE-AC-002` | Failed candidate activation preserves current active package set and writes no candidate production output. |
| `100-LIFECYCLE-AC-003` | Canary and shadow never mark last-known-good or advance watermarks. |
| `100-LIFECYCLE-AC-004` | Rollback target must be an immutable verified manifest. |
| `100-LIFECYCLE-AC-005` | Quarantine blocks dependent activation and cannot clear directly to active. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `100-AC-001` | No production package can activate unless immutable artifact, package type policy, repository form, trust, freshness, anti-rollback, transparency, attestation, provenance, SBOM, dependency lock, compatibility, validation, and package-set evidence pass. |
| `100-AC-002` | Candidate activation failure preserves the current active package set and writes no candidate production output. |
| `100-AC-003` | Rollback targets immutable verified package-set manifests only and passes rollback compatibility gates before active state changes. |
| `100-AC-004` | Emergency override cannot bypass trust verification for new production activation. |
| `100-AC-005` | Package repository model, trust root governance, transparency, SBOM/provenance, dependency lock, compatibility matrix, health gate, rollback, quarantine, emergency, and cohesion group catalogs have exact catalog rows or explicit blocking rows before authoritative status. |
| `100-AC-006` | `AcceptanceReport` fails while any package activation fixture checksum, expected output checksum, expected error, package-specific validation ref, or package `VersionManifest` inclusion check remains `TODO`, blocked, failed, not run, absent, or checksum-mismatched. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `100-RESOLVED-PACKAGE-TYPE-ENUM` | Canonical `PackageType` enum tokens are confirmed by this spec. | Package type token parsing and unknown-token rejection. | `100.PackageType`; validation by `120-PACKAGE-TYPE-*`. | Unknown or legacy broad labels fail with `PACKAGE_TYPE_UNKNOWN`. |
| `100-TODO-PACKAGE-TYPE-POLICY-ROWS` | TODO: Every confirmed package type must have exactly one active `PackageTypePolicyRow` per target environment before package-set activation can affect production output. | Package type policy, deprecation rows, compatibility rows, validation fixtures, and manifest inclusion. | Product governance plus `000`, `030`, `100`, and `120` validation refs. | Package activation remains blocked with `PACKAGE_TYPE_POLICY_MISSING` or `PACKAGE_TYPE_POLICY_AMBIGUOUS` until policy rows validate. |
| `100-TODO-DEPRECATION-ROWS` | TODO: Provide active `PackageDeprecationWindowPolicyRow` instances for every confirmed package type. | Deprecation expiry, emergency extension, rollback eligibility, and migration windows. | `100` package governance and `120` package deprecation fixtures. | Missing row fails with `PACKAGE_DEPRECATION_WINDOW_EXPIRED` or policy-missing package validation, and promotion remains blocked. |
