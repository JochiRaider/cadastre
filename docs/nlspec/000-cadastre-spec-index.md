---
doc_id: CADASTRE-NLSPEC-000
title: Cadastre Spec Index and Source-of-Truth Map
doc_type: nlspec
status: candidate
---

## Authority

This document owns Cadastre documentation governance, document status vocabulary, document classes, source-of-truth ownership, dependency policy, promotion gates, document registry consistency, spec-set version records, and archival rules.

This document must not define product runtime behavior. Runtime behavior must be owned by the specific Cadastre NLSpec that exports the relevant contract.

## Purpose

Define the self-contained Cadastre documentation set, source-of-truth ownership, spec status vocabulary, dependency policy, acceptance dependencies, promotion rules, and archival rules.

## Explicit Non-Scope

- Domain runtime behavior.
- Data model fields except documentation-governance records.
- Package, graph, identity, source, API, or validation algorithms.
- Product decisions that are still open in owner-local `TODO:` rows.

## Imports

- `docs/reference/standards/nlspec-spec.md`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.NormalizeScopeSelector`
- `030.ScopeSelectorCovers`
- `030.CompareScopeSpecificity`
- `030.ResolveScopedRow`

## Exports

- `SpecStatus`
- `DocumentClass`
- `SourceOfTruthRow`
- `SpecDependency`
- `SpecSetVersion`
- `PromotionGate`
- `DocumentRegistryConsistencyRule`
- `VolatilityClass`
- `VolatilityClassificationRow`
- `VolatilityRegistryConsistencyRule`
- `DefineOnceClosureInventory`

## Document Status Vocabulary

`SpecStatus` is a closed vocabulary for document authority. A status value not listed in this table must not be used by the document registry.

| Status | Meaning | May drive implementation |
| --- | --- | --- |
| `draft` | Authoring text that is not review-complete. | No. |
| `candidate` | Candidate NLSpec text that may be reviewed and validated. | No, except controlled validation or implementation spikes explicitly named by an acceptance report. |
| `authoritative` | Accepted NLSpec text whose validation criteria pass and whose registry row marks it active. | Yes, for owned contracts only. |
| `superseded` | Replaced by a newer registered version. | No. |
| `archived` | Retained for historical reference only. | No. |
| `accepted_rationale` | Accepted decision rationale with no runtime authority. | No. |
| `inactive_deferred` | Preserved future-domain candidate with no active MVP effect. | No. |
| `active_standard` | Reference standard for NLSpec quality only. | No product runtime authority. |

## Document Class Registry

| Document class | Scope | Runtime authority | Promotion rule |
| --- | --- | ---: | --- |
| `authoritative_spec` | Active NLSpecs with status `authoritative`. | Yes, for owned contracts only. | Requires passing `120` evidence, no unresolved owner `TODO:` rows, and an active registry row. |
| `candidate_spec` | NLSpecs under candidate review. | No. | May become `authoritative_spec` only through `PromotionGate`. |
| `deferred_spec` | Inactive future-domain material. | No. | Requires future accepted promotion, owners, decisions, and fixtures. |
| `reference` | Research reports, standards, cleanup references, and other evidence documents. | No. | Must not be promoted without conversion into an owning NLSpec. |
| `rationale` | ADRs and decision rationale. | No. | Remains rationale only. |
| `archive` | Superseded product drafts and historical documents. | No. | Historical reference only. |
| `navigation` | Manifest and navigation documents. | No. | Must point to this index for status when status is needed. |

Reference and rationale documents never provide runtime authority unless their content is converted into an owning NLSpec that exports the relevant contract and passes acceptance.

## Authoritative Source Rule

Implementation authority must be determined in this order:

1. Active Cadastre NLSpecs listed in this index with status `authoritative`.
2. Validation and acceptance reports referenced by `docs/nlspec/120-validation-fixtures-and-acceptance.md` for acceptance state only.
3. ADRs only for rationale, never for runtime behavior.
4. Research reports only as supporting evidence, never as runtime authority.
5. Archived product documents only for historical reference, never for implementation authority.

Until this index marks a document `authoritative`, the NLSpec set remains candidate text and must not be treated as production implementation authority except for controlled validation or implementation spikes explicitly named by an acceptance report.

## Promotion Gates

| From status | To status | Required gate | Blocking conditions |
| --- | --- | --- | --- |
| `candidate` | `authoritative` | Passing owner `AcceptanceReport`; no unresolved owner `TODO:` rows; registry row marks the file active. | Failed, blocked, missing, or stale validation evidence; unresolved owner `TODO:` rows; duplicate owner for a runtime contract. |
| `authoritative` | `superseded` | New accepted `SpecSetVersion` names the replacement. | Missing replacement checksum or missing registry row. |

Promotion must fail when any active absence-sensitive feed category or category row with non-empty `allowed_effects` can affect output and the spec set lacks registered, classified, validated, and manifest-included source-authority closure row-catalog refs or deterministic block catalog refs for every out-of-scope family.

Promotion must fail when any active MVP OCSF mapping row set exists and the spec set lacks registered, classified, validated, and manifest-included refs for `ExternalSchemaProfile`, `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `ObservationToOCSFMappingRowSet`, `ExternalEnumMappingRuleSet`, `OCSFBaseEventFieldPolicySet`, `SourceExtensionFieldRuleSet`, `ObservationTypeExternalMappingValidationMatrix`, and `CanonicalValidationOutput`.

Additional promotion gates:

| Gate | Blocking condition |
| --- | --- |
| identity resolver closure | When `IdentityDecision`, `ResolverProfile`, `SourceAsset`, `Identifier`, or `CanonicalEntity` output is in implementation scope, promotion fails if any required resolver artifact row set, activation report, validation row, fixture checksum, expected output checksum, manifest ref, package weakening row, or decision-ambiguity closure row is missing, stale, failed, blocked, not run, checksum-mismatched, or `TODO`. |
| package activation registry closure | When `PackageReleaseManifest`, `ProductionPackageSetManifest`, package activation, rollback, quarantine, emergency override, package stage binding, or package-supplied activation artifacts are in implementation scope, promotion fails if any required package activation registry ref, policy row set, compatibility row, deprecation row, validation row, fixture checksum, expected output checksum, error parity row, manifest ref, or package type coverage row is missing, stale, failed, blocked, not run, checksum-mismatched, or `TODO`. |
| run-lock closure | When run-lock governed output is in implementation scope, promotion fails if run-lock lease policy, stale recovery behavior, fencing-token semantics, commit guard semantics, operation evidence, recovery evidence, error registry rows, or validation refs are missing, blocked, not run, failed, stale, checksum-mismatched, or `TODO`-bearing. |
| scope selector closure | Promotion fails if any active scoped owner family restates selector schema, equality, coverage, subset matching, specificity, ambiguity, or tie-breaking instead of importing the exact `030` selector contracts, or if required `120-SCOPE-SELECTOR-CLOSURE-*` fixtures are missing, blocked, not run, failed, stale, checksum-mismatched, or `TODO`-bearing. |

### MVP Activation Catalog Closure Gate

`MVPActivationCatalogClosurePack` is a promotion-time set of refs. It is not a runtime authority, a row schema, a package label, a summary report, or a substitute for owner-local activation rows. The pack exists only to prove that every selected production scope has either active validated row-set refs or deterministic block refs for each activation-controlled catalog family that can affect output.

| Catalog family | Required member refs |
| --- | --- |
| `feed_category_closure` | `020` source-dataset catalog refs and `LakehouseFeedCategoryClosureRowSet` refs for every declared feed category. |
| `ocsf_mapping_closure` | `050` `ExternalSchemaProfile`, compiled artifact ref, profile-resolution manifest refs, observation-to-OCSF mapping row set, enum rule set, base-event field policy set, source-extension rule set, observation-type validation matrix, and canonical validation output. |
| `source_authority_closure` | `060.SourceAuthorityClosureMatrixRowSet` plus every underlying source authority, completeness, coverage, staleness, progress-signal, supplier-visibility, control-result, source-history, absence, external-schema-authority-signal, and watermark row set required by the selected effect. |
| `resolver_catalog_closure` | `070.ResolverProfileRowSet` and every resolver artifact row set, including `ResolverActivationReportPolicy`. |
| `graph_active_profile_closure` | `090.GraphActiveProfileClosure`, `GraphEdgeSemantics` row set, `GraphObjectOutputEligibilityRowSet`, active graph profile refs, and deterministic inactive/block rows for out-of-scope edge types. |
| `graph_backend_selection_closure` | `090.GraphBackendSelectionDefaultDecision`, backend selection policy refs, graph backend profile refs when supplied, and `GraphBackendActivationBlockerSet` refs for unresolved provider, storage, index, schema, fixture, and package gates. |
| `package_activation_registry_closure` | `100.PackageTypePolicyRowSet`, package deprecation window policy row set, compatibility rows, trust/provenance/SBOM policy rows, package release manifests, and production package-set manifests. |

Closed closure result states are:

| State | Promotion behavior |
| --- | --- |
| `closed_active` | May pass when every member ref is active, checksum-valid, validation-backed, and manifest-included where output-affecting. |
| `closed_deterministically_blocked` | May pass only for the exact blocked scope and only when mutation-prohibition validation proves no production mutation, output, watermark, graph apply, package activation, or visibility effect can occur. |
| `blocked_missing_ref` | Must fail promotion. |
| `blocked_validation` | Must fail promotion. |
| `blocked_todo` | Must fail promotion. |
| `blocked_checksum` | Must fail promotion. |
| `blocked_package_set` | Must fail promotion. |
| `blocked_manifest` | Must fail promotion. |

Promotion to `authoritative` may pass this gate only when every closure-pack family in the selected production scope is `closed_active` or `closed_deterministically_blocked`. A deterministic block row must be exact to the blocked family, source dataset, row set, effect, package scope, graph scope, or resolver scope and must prove no production mutation for that scope. Closure summaries, validation reports, package labels, repository snapshots, branches, tags, ADRs, research reports, and owner prose must not substitute for the underlying owner row refs.

The required validation rows for this gate are `val-000-activation-catalog-closure-pack-complete`, `val-000-activation-catalog-closure-pack-member-manifested`, `val-000-activation-catalog-closure-pack-block-row-no-mutation`, and `val-000-activation-catalog-closure-pack-todo-blocks-promotion`.

### Structured input repository promotion gate

| Gate | Blocking condition |
| --- | --- |
| structured input repository closure | When structured-input repository workflow is in implementation scope, promotion fails if any repository profile ref, snapshot ref, exact tree or file checksum, validation run ref, redaction validation ref, materialization result ref, package release ref, package-set ref, access policy ref, error registry row, audit fixture, `VersionManifest` ref, fixture checksum, expected output checksum, or expected error is missing, failed, stale, checksum-mismatched, `TODO`, `blocked`, or `not_run`. |

## SpecSetVersion Record

| Field | Type | Required | Default | Rule |
| --- | --- | ---: | --- | --- |
| `spec_set_version` | string | Yes | none | Monotonic version assigned by this index. |
| `spec_set_checksum` | sha256 string | Yes for authoritative handoff | none | Computed over included authoritative docs using canonical path order and exact bytes. |
| `included_docs` | array of path/checksum rows | Yes | empty | Must include every authoritative doc. |
| `deferred_docs` | array of path/checksum rows | Yes | empty | Must include `docs/deferred/200-future-reachability-analysis-domain.md` while it remains deferred. |
| `acceptance_report_id` | string | Yes for authoritative handoff | none | Must reference a passing `AcceptanceReport`. |
| `supersedes` | string or null | No | null | Must reference the prior spec-set version when replacing it. |
| `effective_from` | timestamp or null | Yes for authoritative handoff | null | RFC3339 UTC timestamp assigned when the spec-set version becomes authoritative. |
| `open_exclusions` | array | Yes | empty | Explicit deferred or unresolved issues implementation must not infer. |
| `volatility_registry_checksum` | sha256 string | Yes for authoritative handoff | none | SHA-256 over canonical volatility classification rows in path and artifact order. |
| `activation_artifact_registry_refs` | array | Yes | empty | Refs to activation-controlled artifact registries included in the spec-set version. For every active production scope, this array must include the `MVPActivationCatalogClosurePack` member refs by family: `feed_category_closure`, `ocsf_mapping_closure`, `source_authority_closure`, `resolver_catalog_closure`, `graph_active_profile_closure`, `graph_backend_selection_closure`, and `package_activation_registry_closure`. Each member must be either an active owner row-set ref or an exact deterministic block ref with validation evidence. Closure summaries, validation reports, repository snapshots, branches, tags, package labels, ADRs, research reports, and owner prose must not substitute for underlying owner refs. Missing, inactive, checksum-mismatched, package-set-mismatched, or unmanifested member refs fail promotion with the corresponding closure result state. |
| `open_volatility_exclusions` | array | Yes | empty | Explicit volatility classification exclusions that implementation must not infer. |
| `validation_matrix_refs` | array | Yes | empty | Required owner validation rows from `120`, including `120-DOMAIN-SECTION25-*`, `120-CORE-ONEOF-CLOSURE-*`, `120-CORE-EVIDENCE-ARTIFACT-*`, `120-CORE-NULL-OMISSION-*`, `120-CORE-ID-REPLAY-ONEOF-*`, and `120-CORE-ERROR-REGISTRY-*` before authoritative handoff for core one-of closure and domain closure-ledger status consistency, including feed-closure rows before authoritative handoff when feed profiles are active. Source-authority closure handoff must include `120-SOURCE-CLOSURE-*` and `SourceAuthorityClosureMatrix` validation refs before authoritative handoff when any absence-sensitive feed profile is active. OCSF/external-schema mapping handoff must include `120-OCSF-MAP-*`, `120-SOURCE-EXT-*`, `120-OCSF-NONAUTH-*`, and `120-OCSF-DIRECTION-*` rows when any MVP mapping row set is active. Lifecycle-affecting handoff must include `val-030-lifecycle-*`, `val-090-lifecycle-*`, `val-100-lifecycle-*`, `val-120-lifecycle-*`, and `val-domain-lifecycle-todo-resolved`. Run-lock closure handoff must include `120-RUNLOCK-CLOSE-*`, `val-030-runlock-*`, `val-020-runlock-*`, `val-060-runlock-*`, `val-080-runlock-*`, `val-090-runlock-*`, `val-100-runlock-*`, `val-110-runlock-*`, and `val-140-runlock-*` rows before authoritative handoff when run-lock governed output is in scope. Temporal/correction/replay handoff must include `120-TEMPORAL-CORRECTION-*`, `120-ASSERTION-TRANSITION-*`, `120-REPLAY-OUTPUT-CLASS-*`, `120-GRAPH-HANDOFF-*`, and `120-NOOP-ERROR-*` rows before authoritative handoff when `080` output is in scope. Identity output handoff must include `120-IDENTITY-CLOSURE-*`, `120-IDENTITY-REPLAY-*`, `120-IDENTITY-REVIEW-*`, `120-IDENTITY-SPLIT-*`, `120-IDENTITY-EXPLANATION-*`, and identity package resolver-artifact weakening rows before authoritative handoff when `IdentityDecision`, `ResolverProfile`, `SourceAsset`, `Identifier`, or `CanonicalEntity` output is in implementation scope. Graph profile closure handoff must include `120-GRAPH-PROFILE-CLOSURE-*`, `120-GRAPH-QUERY-TRANSLATION-*`, `120-GRAPH-OUTPUT-ELIGIBILITY-*`, `120-GRAPH-APPLY-ORDER-*`, `120-GRAPH-REBUILD-EQUIVALENCE-*`, `120-GRAPH-ENDPOINT-IDENTITY-*`, `120-GRAPH-PAGE-TOKEN-*`, `120-OCSF-DIRECTION-*`, reachability-prohibition rows, `120-GRAPH-BACKEND-SELECTION-*`, `120-GRAPH-BACKEND-PREFLIGHT-*`, `120-GRAPH-PROVIDER-CAPABILITY-*`, `120-GRAPH-JANUSGRAPH-*`, `120-GRAPH-PROVIDER-PORTABILITY-*`, `120-GRAPH-INDEX-FRESHNESS-*`, `120-GRAPH-PARTIAL-APPLY-*`, and `120-GRAPH-BACKEND-PACKAGE-GATE-*` rows before authoritative handoff when graph projection, graph apply, graph query, graph rebuild, graph-serving output, backend defaulting, or provider activation is in implementation scope. Package activation handoff must include `120-PACKAGE-TYPE-*`, `120-PACKAGE-REPOSITORY-*`, `120-PACKAGE-TRUST-*`, `120-PACKAGE-ATTESTATION-SBOM-*`, `120-PACKAGE-COMPATIBILITY-*`, `120-PACKAGE-DEPRECATION-*`, `120-PACKAGE-LKG-*`, `120-PACKAGE-ROLLBACK-*`, `120-PACKAGE-QUARANTINE-*`, `120-PACKAGE-EMERGENCY-*`, and `120-PACKAGE-VERSION-MANIFEST-*` rows before authoritative handoff when `PackageReleaseManifest`, `ProductionPackageSetManifest`, package activation, rollback, quarantine, emergency override, package stage binding, or package-supplied activation artifacts are in implementation scope. API and observable-state handoff must include `120-API-SCHEMA-TOTAL-*`, `120-STATE-LABEL-TOTAL-*`, `120-ENDPOINT-OUTCOME-*`, `120-ERROR-REGISTRY-*`, `120-AUTH-REDACTION-*`, and `120-PAGE-TOKEN-*` rows before authoritative handoff when API, export, health, audit, compliance, or graph-query response output is in implementation scope. Observability handoff must include `120-OBSERVABILITY-SIGNAL-*`, `120-OBSERVABILITY-ATTRIBUTE-*`, `120-OBSERVABILITY-REDACTION-*`, `120-OBSERVABILITY-CARDINALITY-*`, `120-OBSERVABILITY-EXPORTER-*`, `120-OBSERVABILITY-HEALTH-*`, `120-OBSERVABILITY-REPLAY-*`, `120-OBSERVABILITY-NONAUTH-*`, and `120-OBSERVABILITY-VERSION-MANIFEST-*` rows before authoritative handoff when telemetry, observability health, API diagnostics, audit diagnostics, validation diagnostics, or telemetry runtime state visibility is in implementation scope. |

`SpecSetVersion.validation_matrix_refs` must include `120.ActivationCatalogClosureValidationMatrix` and the member families `120-FEED-CLOSURE-*`, `120-OCSF-MAP-*`, `120-SOURCE-EXT-*`, `120-OCSF-NONAUTH-*`, `120-OCSF-DIRECTION-*`, `120-SOURCE-CLOSURE-*`, `120-IDENTITY-CLOSURE-*`, `120-GRAPH-PROFILE-CLOSURE-*`, `120-GRAPH-BACKEND-*`, `120-PACKAGE-*`, `120-VERSION-MANIFEST-*`, `120-ERROR-REGISTRY-*`, and `120-PRIVATE-BINDING-*` before authoritative promotion for any active closure-pack family.

| `implementation_scope` | array | Yes | empty | Contracts, interfaces, algorithms, errors, defaults, and mappings covered. |
| `feedback_rule` | string | Yes | `spec_change_required` | Implementation discoveries that affect behavior must create a spec change before or alongside code. |

When structured-input repositories are enabled, `activation_artifact_registry_refs` must include active structured-input repository profile row sets, repository access policy row sets, repository redaction policy row sets, materialization validation refs, and deterministic block refs for any structured-input family intentionally out of scope. These refs must also appear in owner `VersionManifest` records whenever repository-authored artifacts affect output.

### Identity resolver spec-set registry refs

When `IdentityDecision`, `ResolverProfile`, `SourceAsset`, `Identifier`, or `CanonicalEntity` output is in implementation scope, `SpecSetVersion.activation_artifact_registry_refs` must include active registry refs for the following identity resolver artifacts, or an owner `TODO:` blocker that explicitly keeps identity output out of implementation scope:

| Required registry ref | Owner |
| --- | --- |
| `ResolverProfileRowSet` | `070` |
| `IdentifierEvidenceClassRowSet` | `070` |
| `IdentifierScopeRowSet` | `070` |
| `CandidateGenerationProfile` | `070` |
| `IdentityHardBlockerRowSet` | `070` |
| `AssetGenerationBoundaryRowSet` | `070` |
| `ResolverDecisionMatrixRowSet` | `070` |
| `IdentityConfidenceBandRowSet` | `070` |
| `IdentityReviewRoutingPolicy` | `070` |
| `IdentitySplitPolicy` | `070` |
| `ResolverExplanationPolicy` | `070` |
| `TargetSelectorSafetyPolicy` | `070` |

Missing refs block promotion before authoritative handoff. Registry refs must match the artifact names and classes used by `030`, `070`, `100`, and `120`.

### Package activation spec-set registry refs

When `PackageReleaseManifest`, `ProductionPackageSetManifest`, package activation, rollback, quarantine, emergency override, package stage binding, or package-supplied activation artifacts are in implementation scope, `SpecSetVersion.activation_artifact_registry_refs` must include active registry refs for the following package activation artifacts, or explicit owner `TODO:` blockers that prevent promotion:

| Required registry ref | Owner |
| --- | --- |
| `PackageTypePolicyRowSet` | `100` |
| `PackageTypePolicyRowCoverageMatrix` | `100` |
| `PackageRepositoryModelRowSet` | `100` |
| `PackageTrustPolicy` | `100` |
| `PackageTransparencyEvidencePolicy` when consulted | `100` |
| `PackageProvenancePolicy` when provenance is required | `100` |
| `PackageSBOMPolicy` when SBOM is required | `100` |
| `PackageDependencyLockPolicy` when dependencies affect output or validation | `100` |
| `PackageCompatibilityMatrix` | `100` |
| `PackageDeprecationWindowPolicy` | `100` |
| `LastKnownGoodHealthGate` | `100` |
| `RollbackCompatibilityPolicy` | `100` |
| `QuarantineScopePolicy` | `100` |
| `PackagePromotionGatePolicy` | `100` |
| `PackageReleaseManifest` registry or manifest refs | `100` |
| `ProductionPackageSetManifest` registry or manifest refs | `100` |

Package activation registry refs must match the artifact names and classes used by `030`, `100`, `110`, and `120`. `PackageType` enum closure is not sufficient for package-set activation; active package type policy rows, deprecation rows, compatibility rows, package evidence, manifest completeness, error-registry parity, and validation fixtures remain promotion blockers.

## Volatility Classification Governance

`VolatilityClass` is a closed governance vocabulary. Every implementation-relevant exported contract, profile, row set, package, fixture, validation artifact, registry artifact, and runtime state class must have exactly one volatility classification.

| Volatility class | May define runtime behavior | May be separately activated | Must appear in `VersionManifest` | Must have lifecycle status | May appear in `ProductionPackageSetManifest` | Required owner |
| --- | --- | --- | --- | --- | --- | --- |
| `stable_core_contract` | Yes, only through the authoritative owner NLSpec. | No. | Yes when the contract version affects output or replay. | Yes when promotion or runtime eligibility depends on lifecycle. | No. | Owner NLSpec registered in `000`. |
| `activation_controlled_artifact` | No. It may instantiate exported stable behavior only. | Yes. | Yes when it can affect output, validation, visibility, replay, or activation. | Yes. | Yes when package-supplied. | Stable owner NLSpec plus artifact owner row. |
| `runtime_state_record` | No. It records execution, replay, commit, snapshot, lifecycle, health, or validation state. | No. | Yes when output-affecting or replay-affecting. | Only when the state record itself has production eligibility semantics. | No, except as referenced evidence. | Domain owner and `030.VersionManifest`. |
| `reference_evidence` | No. | No. | No, except as cited non-runtime evidence in acceptance material. | No. | No. | `000`. |
| `rationale` | No. | No. | No. | No. | No. | `000`. |
| `inactive_future_domain` | No while inactive. | No while inactive. | Deferred docs must be recorded in `SpecSetVersion.deferred_docs`. | Future activation requires owner assignment. | No while inactive. | `000` and deferred owner. |

Rules:

- `stable_core_contract` may define runtime behavior only when its owner NLSpec is authoritative.
- `activation_controlled_artifact` must not define new runtime behavior. It may instantiate behavior exported by a stable core contract.
- `runtime_state_record` records execution, replay, commit, snapshot, lifecycle, or health state and must not create authority by itself.
- `reference_evidence`, `rationale`, and `inactive_future_domain` must not drive runtime behavior.
- Every `Owner Contract Registry` row must include `volatility_class`.
- Every exported contract, profile, row set, package, fixture, validation artifact, registry artifact, and runtime state class must have exactly one volatility classification.
- If an activation-controlled artifact requires behavior not exported by its owner stable core contract, promotion or activation must fail.

`VolatilityClassificationRow` fields are `artifact_or_contract`, `volatility_class`, `owner_spec`, `stable_core_owner`, `may_affect_output`, `version_manifest_requirement`, `package_set_manifest_requirement`, `lifecycle_requirement`, and `validation_row_refs`.

`VolatilityRegistryConsistencyRule` must fail before promotion using the most specific failure code.

| Failure code | Emitted when |
| --- | --- |
| `VOLATILITY_CLASS_MISSING` | An implementation-relevant artifact or contract lacks a volatility class. |
| `VOLATILITY_CLASS_CONFLICT` | More than one volatility class is assigned to the same artifact or contract. |
| `VOLATILE_ROW_IN_STABLE_CORE` | A stable core spec contains production-active volatile row instances rather than non-normative examples or blocking `TODO:` rows. |
| `ACTIVATION_ARTIFACT_OWNER_MISMATCH` | An activation-controlled artifact does not reference the stable core owner contract that exports its behavior. |
| `ACTIVATION_ARTIFACT_RUNTIME_RESTATEMENT` | An activation-controlled artifact restates or defines runtime behavior instead of instantiating owner-exported behavior. |

### Required API and observable-state volatility classifications

The following `110` contracts must have exactly one owner and one volatility class before API scope can be promoted. This section routes governance only and must not define runtime behavior.

| Contract or record | Owner | Volatility class | Required validation refs |
| --- | --- | --- | --- |
| `CommonApiResponseEnvelope` | `110` | `stable_core_contract` | `120-API-SCHEMA-TOTAL-*` |
| `EndpointOutcomeMatrix` | `110` | `stable_core_contract` | `120-ENDPOINT-OUTCOME-*` |
| `AuthorizationPermissionMatrix` | `110` | `stable_core_contract` | `120-AUTH-REDACTION-*` |
| `RedactionDataClassMatrix` | `110` | `stable_core_contract` | `120-AUTH-REDACTION-*` |
| `ErrorCodeRegistryRow` | `110` | `stable_core_contract` | `120-ERROR-REGISTRY-*` |
| `GeneratedErrorCodeRegistry` | `110` | `runtime_state_record` | `120-ERROR-REGISTRY-*` |
| `SourceStateLabelMapping` | `110` | `stable_core_contract` | `120-STATE-LABEL-TOTAL-*` |
| `AuthorizationDecision` | `110` | `runtime_state_record` | `120-AUTH-REDACTION-*` |
| `AuditEvent` | `110` | `runtime_state_record` | `120-AUTH-REDACTION-*` |
| `ErrorRecord` | `110` | `runtime_state_record` | `120-ERROR-REGISTRY-*` |

`SpecSetVersion.validation_matrix_refs` must include `120-API-SCHEMA-TOTAL-*`, `120-STATE-LABEL-TOTAL-*`, `120-ENDPOINT-OUTCOME-*`, `120-ERROR-REGISTRY-*`, `120-AUTH-REDACTION-*`, and `120-PAGE-TOKEN-*` before `110` can move out of candidate or validation-blocked status.

### Required OCSF mapping volatility classifications

The following mapping artifacts must have exactly one volatility classification before authoritative handoff. The rows classify the artifact interface, not concrete row bytes.

| artifact_or_contract | volatility_class | owner_spec | stable_core_owner | may_affect_output | version_manifest_requirement |
| --- | --- | --- | --- | --- | --- |
| `ExternalSchemaProfile` | stable row interface plus activation-controlled profile row set | `050` | `050` | yes | required for OCSF output |
| `ExternalSchemaArtifactRef` | `activation_controlled_artifact` | `050` | `050` | yes | required for OCSF output |
| `ProfileResolutionManifest` | `activation_controlled_artifact` | `050` | `050` | yes | required for OCSF output |
| `ObservationToOCSFMappingRow` | stable row interface plus activation-controlled row set membership | `050` | `050` | yes | required for silver output |
| `ObservationToOCSFMappingRowSet` | `activation_controlled_artifact` | `050` | `050` | yes | required for silver output |
| `ExternalEnumMappingRule` | stable row interface plus activation-controlled row set membership | `050` | `050` | yes | required when enum mapping affects output |
| `ExternalEnumMappingRuleSet` | `activation_controlled_artifact` | `050` | `050` | yes | required when enum mapping affects output |
| `OCSFBaseEventFieldPolicy` | stable row interface plus activation-controlled policy set membership | `050` | `050` | yes | required for OCSF output |
| `OCSFBaseEventFieldPolicySet` | `activation_controlled_artifact` | `050` | `050` | yes | required for OCSF output |
| `SourceExtensionFieldRule` | stable row interface plus activation-controlled rule set membership | `050` | `050` | yes when source extensions may emit | required when source extensions may emit |
| `SourceExtensionFieldRuleSet` | `activation_controlled_artifact` | `050` | `050` | yes when source extensions may emit | required when source extensions may emit |
| `OCSFProfileUpgradeReport` | `runtime_state_record` or activation evidence record according to upgrade use | `050` | `050` | yes when profile version changes | required when the active external schema profile differs from the prior production profile |
| `CanonicalValidationOutput` for mapping activation | `runtime_state_record` | `050` and `120` | `050` and `120` | yes for promotion | required for mapping activation |
| `ObservationTypeExternalMappingValidationMatrix` | `activation_controlled_artifact` | `050` and `120` | `050` and `120` | yes for promotion | required for mapping activation |
| `ExternalSchemaAuthoritySignalMappingRowSet` | `activation_controlled_artifact` | `060` | `060` | only when external schema signals are authority inputs | required for authority effects only |

### Required source-authority closure volatility classifications

The following source-authority closure artifacts must have exactly one volatility classification before authoritative handoff. The rows classify artifact interfaces and validation views, not concrete private source bindings.

| artifact_or_contract | volatility_class | owner_spec | stable_core_owner | may_affect_output | version_manifest_requirement |
| --- | --- | --- | --- | --- | --- |
| `SourceAuthorityClosureMatrix` | `stable_core_contract` | `060` | `060` | yes for promotion and blocking decisions | closure validation result may be referenced, but underlying row refs remain required |
| `SourceAuthorityProfileRow` | `activation_controlled_artifact` | `060` | `060` | yes | required when authority affects output |
| `SourceAuthorityProfileRowSet` | `activation_controlled_artifact` | `060` | `060` | yes | required when authority affects output |
| `CoverageDimensionProfile` source-specific rows | `activation_controlled_artifact` | `060` | `060` | yes | required for coverage-sensitive output |
| `SourceStalenessPolicy` row set | `activation_controlled_artifact` | `060` | `060` | yes | required when stale state can affect output |
| `ProgressSignalInterpretationPolicy` row set | `activation_controlled_artifact` | `060` | `060` | yes | required when progress signals are consulted |
| `ControlResultMappingRowSet` | `activation_controlled_artifact` | `060` | `060` | yes | required for control-state output |
| `SourceHistoryRetentionProfile` row set | `activation_controlled_artifact` | `060` | `060` | yes | required for source-history interpretation |
| `SupplierCollectionVisibilityProfile` row set | `activation_controlled_artifact` | `060` | `060` | yes | required for permission-sensitive absence |
| `AbsenceDerivationPolicy` row set | `activation_controlled_artifact` | `060` | `060` | yes | required for absence-sensitive output |
| `CorrectionSnapshotRefPolicy` | `activation_controlled_artifact` | `080` | `080` | yes | required for correction output |
| `ReplayEquivalencePolicy` | `activation_controlled_artifact` | `080` | `080` | yes | required for replay output |
| `GraphRebuildEquivalencePolicy` | `activation_controlled_artifact` | `080` and `090` | `080` and `090` | yes | required for graph rebuild replay/promotion |
| `EventSequenceValidationCorpus` | `activation_controlled_artifact` | `120` | `120` | yes for promotion | required for temporal/correction/replay validation |
| `TemporalObservationTimeResolution` | `runtime_state_record` | `080` | `080` | yes | required for gold, correction, graph handoff, audit, and replay when source observation time affects output |
| `GoldFactChangeSet` | `runtime_state_record` | `080` | `080` | yes | required for correction output and graph handoff |
| `ReplayInputSufficiencyCheck` | `runtime_state_record` | `080` | `080` | yes | required before replay output |

### Required source-authority closure row-catalog classifications

These row-catalog families are activation-controlled artifacts. They instantiate stable behavior owned by `020` or `060`; they must not define new authority classes, effect tokens, omission states, validation behavior, or manifest fields.

| Artifact or contract | Volatility class | Owner spec | Stable core owner | VersionManifest requirement |
| --- | --- | --- | --- | --- |
| `SourceAuthorityClosureMatrixRowSet` | `activation_controlled_artifact` | `060` | `060.SourceAuthorityClosureMatrix` | Required for absence-sensitive promotion and for any run that executes an absence-sensitive effect. |
| `LakehouseFeedCategoryClosureRowSet` | `activation_controlled_artifact` | `020` | `020.LakehouseFeedCategoryClosureRow` | Required before active feed read/import for any known feed category. |
| `LakehouseFeedCompletenessProfileRowSet` | `activation_controlled_artifact` | `060` | `060.LakehouseFeedCompletenessProfile` | Required for absence, cleanup, retraction, graph expiry, or watermark effects. |
| `SourceAuthorityProfileRowSet` | `activation_controlled_artifact` | `060` | `060.SourceAuthorityProfileRow` | Required before source authority or absence-sensitive effects. |
| `CoverageDimensionProfileRowSet` | `activation_controlled_artifact` | `060` | `060.CoverageDimensionProfile` | Required for coverage-sensitive outputs. |
| `SourceStalenessPolicyRowSet` | `activation_controlled_artifact` | `060` | `060.SourceStalenessPolicy` | Required whenever stale state can affect output. |
| `ProgressSignalInterpretationPolicyRowSet` | `activation_controlled_artifact` | `060` | `060.ProgressSignalInterpretationPolicy` | Required when progress signals are consulted. |
| `SupplierCollectionVisibilityProfileRowSet` | `activation_controlled_artifact` | `060` | `060.SupplierCollectionVisibilityProfile` | Required for permission-sensitive absence. |
| `ControlResultMappingRowSet` | `activation_controlled_artifact` | `060` | `060.ControlResultMappingRow` | Required for control-state output. |
| `SourceHistoryRetentionProfileRowSet` | `activation_controlled_artifact` | `060` | `060.SourceHistoryRetentionProfile` | Required for source-history no-change interpretation. |
| `AbsenceDerivationPolicyRowSet` | `activation_controlled_artifact` | `060` | `060.AbsenceDerivationPolicy` | Required for absence-sensitive output. |
| `ProjectionWatermarkPolicyRowSet` | `activation_controlled_artifact` | `060` | `060.ProjectionWatermarkPolicy` | Required for watermark attempts. |
| `ExternalSchemaAuthoritySignalMappingRowSet` | `activation_controlled_artifact` | `060` | `060.ExternalSchemaAuthoritySignalMappingRow` | Required only when external schema signals are consulted by authority logic. |

`ValidateSpecSet` must reject a spec set when `000`, `domain.md`, `020`, `030`, `060`, or `120` use different names for the same source-closure row-catalog family.

### Required structured-input repository volatility classifications

Structured-input repository artifacts must have exactly one owner and one volatility classification before authoritative handoff. Concrete repository rows are activation-controlled only where this table states so; runtime state records must not become activation targets.

| Contract or artifact | Owner | Volatility class | Required validation refs |
| --- | --- | --- | --- |
| `Structured input repository governance` | `030` | `stable_core_contract` | `120-STRUCTURED-INPUT-*` |
| `StructuredInputRepositoryProfile` | `030` | stable row interface plus `activation_controlled_artifact` concrete rows | `120-STRUCTURED-INPUT-PROFILE-*` |
| `StructuredInputRepositorySnapshot` | `030` | `runtime_state_record` | `120-STRUCTURED-INPUT-SNAPSHOT-*` |
| `StructuredInputChangeProposal` | `030` | `runtime_state_record` | `120-STRUCTURED-INPUT-LIFECYCLE-*` |
| `StructuredInputValidationRun` | `120` | `validation_artifact` | `120-STRUCTURED-INPUT-VALIDATION-*` |
| `StructuredInputMaterializationResult` | `100` | `runtime_state_record` | `120-STRUCTURED-INPUT-MATERIALIZATION-*` |
| `StructuredInputRepositoryAccessPolicy` | `110` | stable interface plus `activation_controlled_artifact` policy rows | `120-STRUCTURED-INPUT-AUTH-*` |
| `StructuredInputRepositoryRedactionPolicy` | `110` | stable interface plus `activation_controlled_artifact` policy rows | `120-STRUCTURED-INPUT-REDACTION-*` |

A repository branch, tag, pull request, hook result, repository URL, or commit timestamp is not a volatility class and must not satisfy any activation-controlled artifact, runtime state record, validation artifact, package release, package-set, or `VersionManifest` requirement.

### Required graph projection volatility classifications

The following graph closure artifacts must have exactly one volatility classification before authoritative handoff. The rows classify artifact interfaces and runtime state, not backend product selection.

| artifact_or_contract | volatility_class | owner_spec | stable_core_owner | may_affect_output | version_manifest_requirement |
| --- | --- | --- | --- | --- | --- |
| `GraphProjectionProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required for graph projection, apply, query, rebuild, and replay |
| `GraphProjectionProfileRowSet` | `activation_controlled_artifact` | `090` | `090` | yes | required for graph delta output |
| `GraphEdgeSemantics` row set | `activation_controlled_artifact` | `090` | `090` | yes | required for every active edge type |
| `GraphTraversalClass` row refs | stable semantics plus `activation_controlled_artifact` inclusion rows | `090` | `090` | yes | required for path queries and analysis compatibility |
| `GraphObjectOutputEligibilityRow` row set | `activation_controlled_artifact` | `090` | `090` | yes | required for search, neighbor expansion, pathfinding, analysis, metrics, and identity-influence gates |
| `GraphBackendProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required before graph mutation, query, rebuild, drift check, serving promotion, and default backend materialization |
| `GraphBackendSelectionPolicy` | stable core selection contract plus activation-controlled default rows where applicable | `090` | `090` | yes when graph serving is enabled | required when omitted backend selection materializes a default |
| `GraphProviderCapabilityMatrix` | `activation_controlled_artifact` | `090` | `090` | yes | required before any provider profile can become active |
| `GraphBackendTaxonomyMappingProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required before backend labels, relationship types, collections, or properties are accepted |
| `GraphQueryTranslationProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required before query execution and page-token generation |
| `GraphReadModelSchemaProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required before apply, query serving, rebuild, or promotion |
| `GraphApplyProfile` | `activation_controlled_artifact` | `090` | `090` | yes | required before graph apply |
| `DerivedViewLagPolicy` | `activation_controlled_artifact` | `090` | `090` | yes | required before serving graph responses |
| `GraphRebuildManifest` | `runtime_state_record` | `090` | `090` | yes for rebuild promotion | required when rebuild is involved |
| `GraphIndexConsistencyCheck` | `runtime_state_record` | `090` | `090` | yes for apply/query/rebuild promotion | required after apply or rebuild |
| `DerivedViewState` | `runtime_state_record` | `090` | `090` | yes | required for every graph-serving response |

### Required identity resolver volatility classifications

The following identity resolver artifacts and runtime state records must have exactly one volatility classification before authoritative handoff. The rows classify artifact interfaces and state records, not private source bindings.

| artifact_or_contract | volatility_class | owner_spec | stable_core_owner | may_affect_output | version_manifest_requirement |
| --- | --- | --- | --- | --- | --- |
| `ResolverProfileRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before identity output |
| `IdentifierEvidenceClassRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before identity output |
| `IdentifierScopeRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before candidate generation |
| `AssetGenerationBoundaryRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before blocker evaluation |
| `IdentityHardBlockerRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before blocker evaluation |
| `CandidateGenerationProfile` | `activation_controlled_artifact` | `070` | `070` | yes | required before candidate generation |
| `TargetSelectorSafetyPolicy` | `activation_controlled_artifact` | `070` | `070` | yes | required before selectors influence output |
| `ResolverDecisionMatrixRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before decision output |
| `IdentityConfidenceBandRowSet` | `activation_controlled_artifact` | `070` | `070` | yes | required before confidence selection |
| `IdentityReviewRoutingPolicy` | `activation_controlled_artifact` | `070` | `070` | yes | required when review can open or transition |
| `IdentitySplitPolicy` | `activation_controlled_artifact` | `070` | `070` | yes | required before split output |
| `ResolverExplanationPolicy` | `activation_controlled_artifact` | `070` | `070` | yes | required before identity decision visibility |
| `IdentityDecision` | `runtime_state_record` | `070` | `070` | yes | required for identity output and replay |
| `IdentityReviewCase` | `runtime_state_record` | `070` | `070` | yes when review is opened or transitioned | required for review output and replay |
| `ResolverExplanation` | `runtime_state_record` | `070` | `070` | yes | required for every identity decision and replay |
| `GraphCorrectionHandoff` | `runtime_state_record` | `070` | `070` | yes when split affects gold or graph output | required for split correction/projection |
| `ResolverActivationReport` | `runtime_state_record` | `070` | `070` | yes for activation and promotion | required when activation or promotion is in scope |
| `ResolverShadowRun` | `runtime_state_record` | `070` | `070` | yes for shadow/canary promotion | required when promotion uses shadow or canary evidence |

### Required package activation volatility classifications

The following package activation contracts, row sets, policies, runtime state records, and release evidence classes must have exactly one volatility classification before authoritative package activation handoff. Runtime behavior remains owned by `100`; activation-controlled artifacts instantiate that behavior and must be referenced through `030.ActivationControlledArtifactRef` when output-affecting.

| Artifact or contract | Volatility class | Owner spec | Stable core owner | VersionManifest requirement |
| --- | --- | --- | --- | --- |
| `PackageType` | `stable_core_contract` | `100` | `100` | required when package activation affects output |
| `PackageTypePolicyRowSet` | `activation_controlled_artifact` | `100` | `100` | required |
| `PackageRepositoryModelRowSet` | `activation_controlled_artifact` | `100` | `100` | required |
| `PackageTrustPolicy` | `activation_controlled_artifact` | `100` | `100` | required |
| `PackageTransparencyEvidencePolicy` | `activation_controlled_artifact` | `100` | `100` | required when policy consulted |
| `PackageProvenancePolicy` | `activation_controlled_artifact` | `100` | `100` | required when provenance required |
| `PackageSBOMPolicy` | `activation_controlled_artifact` | `100` | `100` | required when SBOM required |
| `PackageDependencyLockPolicy` | `activation_controlled_artifact` | `100` | `100` | required when dependency lock required |
| `PackageCompatibilityMatrix` | `activation_controlled_artifact` | `100` | `100` | required |
| `PackageDeprecationWindowPolicy` | `activation_controlled_artifact` | `100` | `100` | required |
| `LastKnownGoodHealthGate` | `activation_controlled_artifact` | `100` | `100` | required |
| `RollbackCompatibilityPolicy` | `activation_controlled_artifact` | `100` | `100` | required for rollback |
| `QuarantineScopePolicy` | `activation_controlled_artifact` | `100` | `100` | required when quarantine may affect activation |
| `EmergencyPackageOverrideRecord` | `runtime_state_record` | `100` | `100` | required when emergency action affects eligibility |
| `PackageSignatureVerificationResult` | `runtime_state_record` | `100` | `100` | required |
| `PackageSBOMRef` | `runtime_state_record` or immutable release evidence | `100` | `100` | required when SBOM policy requires it |
| `PackageBuildProvenance` | `runtime_state_record` or immutable release evidence | `100` | `100` | required when provenance policy requires it |
| `PackageActivationFailureEvent` | `runtime_state_record` | `100` | `100` | required on failure |

`PackageType` enum closure is not sufficient for package-set activation. Active package type policy rows, deprecation rows, compatibility rows, package evidence, manifest completeness, and validation fixtures remain promotion blockers until the required `100`, `030`, `110`, and `120` rows pass.

### Required observability volatility classifications

The following `140` contracts must have exactly one owner and one volatility class before observability scope can be promoted. This section routes governance only and must not define runtime behavior.

| Artifact or contract | Volatility class | Owner spec | Stable core owner | VersionManifest requirement |
| --- | --- | --- | --- | --- |
| `EmitTelemetry` | `stable_core_contract` | `140` | `140` | Required when telemetry affects health, API output, audit output, validation output, or runtime state visibility. |
| `ValidateTelemetryProfile` | `stable_core_contract` | `140` | `140` | Required for telemetry profile activation. |
| `TelemetryNonAuthorityRule` | `stable_core_contract` | `140` and boundary import from `010` | `140` | Required when telemetry is enabled. |
| `TelemetryReplayExclusionPolicy` | `stable_core_contract` plus activation-controlled output rows | `140` and replay handoff to `080` | `140` | Required when telemetry is enabled and replay is in scope. |
| `ObservabilityInstrumentationProfile` | `activation_controlled_artifact` | `140` | `140` | Required when telemetry is enabled. |
| `TelemetrySignalPolicy` | `activation_controlled_artifact` | `140` | `140` | Required when telemetry is enabled. |
| `TraceContextPolicy` | `activation_controlled_artifact` | `140` | `140` | Required when traces or structured logs are enabled. |
| `TelemetryCorrelationPolicy` | `activation_controlled_artifact` | `140` | `140` | Required when caller-visible correlation refs are emitted. |
| `MetricInstrumentCatalog` | `activation_controlled_artifact` | `140` | `140` | Required when metrics are enabled. |
| `MetricInstrumentRow` | `activation_controlled_artifact` | `140` | `140` | Required when metrics are enabled. |
| `TelemetryAttributePolicy` | `activation_controlled_artifact` | `140` | `140` | Required when any telemetry signal is enabled. |
| `TelemetryRedactionPolicy` | `activation_controlled_artifact` | `140` | `140` | Required when any telemetry signal is enabled. |
| `TelemetryExporterProfile` | `activation_controlled_artifact` | `140` | `140` | Required when export is enabled. |
| `TelemetryHealthMappingPolicy` | `activation_controlled_artifact` | `140` | `140` | Required when telemetry can affect health output. |
| `TelemetryRuntimeState` | `runtime_state_record` | `140` | `140` | Required when telemetry state affects health, API, audit, validation, or operator-visible diagnostics. |

## Document Registry

| ID | Path | Document class | Status | Owner | May drive implementation | Source-of-truth role |
| --- | --- | --- | --- | --- | --- | --- |
| `MANIFEST` | MANIFEST.md | navigation | `draft` | 000 | No | Path inventory and navigation only. |
| `domain` | docs/nlspec/domain.md | candidate_spec | `candidate` | domain | No | Root vocabulary, domain boundaries, owner routing, and status ledger; exports no runtime contracts. |
| `000` | docs/nlspec/000-cadastre-spec-index.md | candidate_spec | `candidate` | 000 | No | Governance owner. |
| `010` | docs/nlspec/010-system-boundary-and-authority.md | candidate_spec | `candidate` | 010 | No | Product boundary owner. |
| `020` | docs/nlspec/020-lakehouse-feeds-and-table-state.md | candidate_spec | `candidate` | 020 | No | Lakehouse feed owner. |
| `030` | docs/nlspec/030-processing-dag-lifecycle-and-versioning.md | candidate_spec | `candidate` | 030 | No | DAG and lifecycle owner. |
| `040` | docs/nlspec/040-canonical-data-observation-and-fact-model.md | candidate_spec | `candidate` | 040 | No | Core data owner. |
| `050` | docs/nlspec/050-normalization-external-schema-and-mapping.md | candidate_spec | `candidate` | 050 | No | Normalization owner. |
| `060` | docs/nlspec/060-source-authority-completeness-coverage-and-absence.md | candidate_spec | `candidate` | 060 | No | Authority and absence owner. |
| `070` | docs/nlspec/070-identity-resolution-and-target-selectors.md | candidate_spec | `candidate` | 070 | No | Identity owner. |
| `080` | docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md | candidate_spec | `candidate` | 080 | No | Temporal and gold owner. |
| `090` | docs/nlspec/090-graph-projection-serving-and-backends.md | candidate_spec | `candidate` | 090 | No | Graph owner. |
| `100` | docs/nlspec/100-packages-supply-chain-and-activation.md | candidate_spec | `candidate` | 100 | No | Package owner. |
| `110` | docs/nlspec/110-api-ux-health-errors-and-security.md | candidate_spec | `candidate` | 110 | No | API and security owner. |
| `120` | docs/nlspec/120-validation-fixtures-and-acceptance.md | candidate_spec | `candidate` | 120 | No | Validation owner. |
| `130` | docs/nlspec/130-analysis-enrichment-and-registry-governance.md | candidate_spec | `candidate` | 130 | No | Analysis and registry owner. |
| `140` | docs/nlspec/140-observability-instrumentation.md | candidate_spec | `candidate` | 140 | No | Observability, instrumentation, telemetry signal, exporter, health-mapping, and telemetry non-authority owner. |
| `200` | docs/deferred/200-future-reachability-analysis-domain.md | deferred_spec | `inactive_deferred` | 200 | No | Inactive future reachability. |
| `nlspec-standard` | docs/reference/standards/nlspec-spec.md | reference | `active_standard` | 000 | No | Quality standard only. |
| `ADR-0001` | docs/adr/ADR-0001-lakehouse-fed-boundary.md | rationale | `accepted_rationale` | 010,020 | No | Accepted rationale only; runtime owned by `010` and `020`. |
| `ADR-0002` | docs/adr/ADR-0002-graph-as-derived-projection.md | rationale | `accepted_rationale` | 090 | No | Accepted rationale only; runtime owned by `090`. |
| `ADR-0003` | docs/adr/ADR-0003-ocsf-as-silver-profile.md | rationale | `accepted_rationale` | 050 | No | Accepted rationale only; runtime owned by `050`. |
| `ADR-0004` | docs/adr/ADR-0004-package-set-activation.md | rationale | `accepted_rationale` | 100 | No | Accepted rationale only; runtime owned by `100`. |
| `research-reports` | docs/reference/research/*.md | reference | `draft` | 000 | No | Evidence only. |
| `doc-cleanup` | docs/doc_clean-up/*.md | reference | `draft` | 000 | No | Formatting reference only. |
| `product-archive` | docs/archive/PRD-Cadastre.revised-draft.md | archive | `archived` | 000 | No | Historical reference only. |

## Owner Contract Registry

| Contract | Owner spec | Imported by | Runtime authority class | Volatility class | Validation owner | Owner-local closure state | Open blocker status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Documentation governance | `000` | `MANIFEST.md` and registered documentation artifacts | governance | `stable_core_contract` | 120 | blocked_validation | open until registry consistency rows pass |
| Root domain vocabulary | `domain` | all owner specs | vocabulary | `stable_core_contract` | 120 | blocked_validation | open until domain status ledger rows validate and no runtime restatement exists |
| DomainSection25ClosureLedger | `domain` plus validation in `120` | `000`, all owner specs | none | `stable_core_contract` | 120 | blocked_validation | every row must map to one owner-local closure state and one `120-DOMAIN-SECTION25-*` validation row |
| Boundary and authority classes | `010` | 020,060,090,110,130 | runtime_boundary | `stable_core_contract` | 120 | closed_local | validation rows required |
| Lakehouse feed and table-state contracts | `020` | 030,040,060,080,120 | runtime_data_input | `stable_core_contract` | 120 | blocked_validation | open until feed profile schema completion, feed category closure row validation, feasibility assessment export/schema alignment, manifest fixtures, and checksum rows close |
| RawFeedManifest | `020` | 030,040,060,120 | runtime_data_input | `runtime_state_record` | 120 | closed_local | validation fixture checksums required |
| Raw record identity import | `020` | 040,120,domain | runtime_data_input | `stable_core_contract` | 120 | closed_local | imports `040.ComputeRawRecordId`; no local ID order blocker |
| LakehouseFeedProfileSchema | `020` | 020,030,120,domain | runtime_data_input | `stable_core_contract` | 120 | blocked_validation | feed profile schema validation and target-kind omission fixtures required |
| LakehouseFeedFeasibilityAssessment | `020` | 020,030,120,domain | runtime_data_input | `runtime_state_record` | 120 | blocked_validation | assessment schema, blocking reason, and activation-result fixtures required |
| LakehouseFeedCategoryClosureRowSet | `020` | 020,030,060,110,120,domain | runtime_data_input | `activation_controlled_artifact` | 120 | blocked_validation | every active feed category requires closure row or deterministic block row |
| DAG lifecycle and version manifest | `030` | 020,080,090,100,120 | runtime_orchestration | `stable_core_contract` | 120 | blocked_validation | run-lock lease timing, heartbeat, stale recovery, fencing, idempotency, and commit guards are specified in `030`; authoritative handoff remains blocked until `120-RUNLOCK-*` and lifecycle validation rows pass. |
| Lifecycle machine mechanics | `030` | all lifecycle owners | runtime_orchestration | `stable_core_contract` | 120 | closed_local | validation rows required |
| ActivationControlledArtifactLifecycleMachine | `030` plus artifact owner | all artifact owners | runtime_orchestration | `stable_core_contract` | 120 | closed_local | owner guard validation rows required |
| ProductionRunExecutionLifecycleMachine | `030` | `120`, domain | runtime_orchestration | `stable_core_contract` | 120 | closed_local | lifecycle fixtures required |
| StageExecutionLifecycleMachine | `030`, feed event derivation by `020` | `020`, `120`, domain | runtime_orchestration | `stable_core_contract` | 120 | closed_local | feed-stage fixtures required |
| ActivationControlledArtifactRef | `030` | 020,050,060,070,080,090,100,110,120,130 | runtime_orchestration | `stable_core_contract` | 120 | blocked_validation | activation artifact ref validation rows required |
| DeclaredDAGSubsetProfile | `030` | 020,060,080,090,120 | runtime_orchestration | `activation_controlled_artifact` | 120 | blocked_validation | subset output, watermark, absence, cleanup, retraction, and graph-expiry negative fixtures required |
| Core record schema registry, scalar registry, canonical serialization, IDs, checksums, omission states, and evidence refs | `040` | 020,030,050,060,070,080,090,110,120 | runtime_data_shape | `stable_core_contract` | 120 | blocked_validation | owner enum and one-of validation rows required |
| ComputeRawRecordId | `040` | 020,120,domain | runtime_data_shape | `stable_core_contract` | 120 | closed_local | validation fixture checksums required |
| External schema and mapping | `050` | 040,100,120,130 | runtime_normalization | `stable_core_contract` | 120 | blocked_validation | stable mapping row schema, row selection, defaults, errors, and non-authority boundaries are closed; concrete active row-set refs and fixture checksums remain validation-blocked until represented as activation-controlled artifacts |
| ObservationToOCSFMappingRowSet | `050` | 030,040,060,090,110,120 | runtime_normalization | `activation_controlled_artifact` | 120 | blocked_validation | stable owner contract exists in `050`; production activation remains blocked until active row sets, compiled artifacts, fixture refs, expected checksums, and `120-OCSF-MAP-*` rows pass |
| SourceExtensionFieldRuleSet | `050` | 030,040,110,120 | runtime_normalization | `activation_controlled_artifact` | 120 | blocked_validation | every emitted source-extension path requires exact rule, redaction, secret-scan, collision, and reserved-name fixtures |
| ObservationTypeExternalMappingValidationMatrix | `050`, `120` | 000,030,050,060,090,110 | validation | `activation_controlled_artifact` | 120 | blocked_validation | required OCSF mapping fixture checksums, expected output checksums, expected errors, and version-manifest refs remain blocking until filled |
| Source authority and absence | `060` | 010,020,080,090,110,130 | runtime_authority | `stable_core_contract` | 120 | blocked_validation | stable schemas, matching algorithms, error precedence, and fail-closed behavior are closed by `060`; active source-specific row instances and fixture checksums remain validation-blocked |
| `SourceAuthorityClosureMatrix` | `060` | `020`, `030`, `080`, `090`, `110`, `120`, `domain` | `runtime_authority_validation` | `stable_core_contract` | 120 | blocked_validation | stable owner contract exists in `060`; production activation remains blocked until every active absence-sensitive feed category has exact closure rows or deterministic block rows |
| LakehouseFeedCompletenessProfileRow | `060` | 020,030,080,090,110,120,domain | runtime_authority | `activation_controlled_artifact` | 120 | blocked_validation | every absence-sensitive active category/effect requires exact completeness rows and fixtures |
| Identity resolution | `070` | 040,060,080,090,100,110,120,domain | runtime_identity | `stable_core_contract` | 120 | blocked_validation | stable resolver rows, active row instances, identity hard blocker row set, activation report scenario evidence, fixture checksums, decision ambiguity closure, review expiration, selector safety, confidence bands, split handoff, and weak-evidence defaults are closed by `070`; active row instances and fixture checksums remain validation-blocked |
| IdentityReviewCaseStateMachine | `070` | `030`, `110`, `120`, `domain` | runtime_identity | `stable_core_contract` | 120 | blocked_validation | runtime owner is `070`; generic lifecycle behavior imports `030`; validation owner is `120` |
| Temporal, gold, replay | `080` | 030,040,060,090,120 | runtime_gold | `stable_core_contract` | 120 | closed_local | validation rows required; concrete activation-controlled row instances remain blockers only when selected for production scope |
| Graph projection and serving | `090` | 040,050,060,070,080,110,120,130 | derived_projection | `stable_core_contract` | 120 | blocked_validation | active profile closure is owner-closed only when `090` has no graph-profile TODOs and `120` graph profile, query, eligibility, endpoint identity, page-token, apply-order, rebuild, and reachability-prohibition rows pass |
| GraphEdgeSemanticsRegistry | `090` | `050`, `060`, `070`, `080`, `110`, `120`, `domain` | derived_projection | `stable_core_contract` | 120 | blocked_validation | MVP edge set closure is `closed_local` for `observed_connection` only; production promotion remains blocked until graph profile validation rows pass |
| GraphBackendSelectionPolicy | `090` | `030`, `100`, `110`, `120`, `domain` | derived_projection | stable core selection contract plus activation-controlled default rows where applicable | 120 | blocked_validation | defaulting behavior is owned by `090`; `mvp-janusgraph.v1` may be a selection default only; activation remains blocked until backend/package validation rows pass |
| GraphApplyLifecycleMachine | `090` | `030`, `080`, `110`, `120` | derived_projection | `stable_core_contract` | 120 | closed_local | graph apply fixtures required |
| PackageType | `100` | `000`, `030`, `090`, `110`, `120`, `domain` | runtime_activation | `stable_core_contract` | 120 | closed_local | canonical enum is closed; policy-row activation remains `blocked_validation` |
| Package-set activation | `100` | 030,050,090,110,120 | runtime_activation | `stable_core_contract` | 120 | blocked_validation | package type policy coverage, repository form closure, supply-chain evidence rows, rollback/quarantine/emergency rows, health gates, manifest completeness, and fixture checksums remain TODO |
| PackageSetActivationLifecycleMachine | `100` | `030`, `110`, `120` | runtime_activation | `stable_core_contract` | 120 | closed_local | package lifecycle, LKG, rollback, quarantine, and emergency fixtures required |
| API, errors, security | `110` | all owner specs | runtime_api | `stable_core_contract` | 120 | blocked_validation | page-token expiration bounds are closed locally; graph response context, reachability wording, and error-registry fixture checksums remain validation-blocked |
| Validation and acceptance | `120` | 000 and all owner specs | validation | `stable_core_contract` | 120 | blocked_validation | fixture and expected-output checksums remain TODO |
| ValidationAcceptanceLifecycleMachine | `120` | `000`, `030`, domain | validation | `stable_core_contract` | 120 | closed_local | validation lifecycle fixtures required |
| Analysis, enrichment, registry | `130` | 050,060,090,110 | non_authoritative_analysis | `stable_core_contract` | 120 | blocked_validation | fixture checksums required |
| Observability instrumentation and runtime telemetry | `140` | 010,030,060,070,080,090,100,110,120,domain | runtime_observability | `stable_core_contract` | 120 | blocked_validation | stable telemetry contracts exist only after `140` is added; production activation remains blocked until `120-OBSERVABILITY-*` validation rows pass |
| Deferred reachability | `200` | 090,110,120 | inactive_future_domain | `inactive_future_domain` | 120 | inactive_deferred | deferred |

Owner-local closure states are closed values: `closed_local`, `blocked_owner_todo`, `blocked_validation`, `inactive_deferred`, and `candidate_not_promoted`. A `closed_local` owner contract may still be prevented from promotion by candidate document status or missing validation evidence.

### Structured input repository owner contract rows

| Contract | Owner spec | Imported by | Runtime authority class | Volatility class | Validation owner | Owner-local closure state | Open blocker status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Structured input repository governance | `030` | `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, `140`, `domain` | runtime_orchestration | `stable_core_contract` | 120 | blocked_validation | repository profile, snapshot, validation, access, redaction, materialization, package, manifest, and fixture rows required |
| StructuredInputRepositoryProfile | `030` | `010`, `020`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, `140`, `domain` | runtime_orchestration | stable interface plus `activation_controlled_artifact` concrete rows | 120 | blocked_validation | profile row-set fixtures and private-binding validation required |
| StructuredInputRepositorySnapshot | `030` | `040`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, `140` | runtime_orchestration | `runtime_state_record` | 120 | blocked_validation | deterministic snapshot, path normalization, mutable ref, force-push, and manifest fixtures required |
| StructuredInputChangeProposal | `030` with observable behavior in `110` | `100`, `110`, `120` | runtime_orchestration | `runtime_state_record` | 120 | blocked_validation | lifecycle and merge-revalidation fixtures required |
| StructuredInputValidationRun | `120` | `030`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `130`, `140` | validation | `validation_artifact` | 120 | blocked_validation | exact snapshot, stale validation, private leak, redaction, negative, and materialization fixtures required |
| StructuredInputMaterializationResult | `100` | `030`, `040`, `050`, `060`, `070`, `080`, `090`, `110`, `120`, `130`, `140` | runtime_activation | `runtime_state_record` | 120 | blocked_validation | materialization, package release handoff, direct Git rejection, rollback, and manifest fixtures required |
| StructuredInputRepositoryAccessPolicy | `110` | `010`, `030`, `100`, `120`, `140` | runtime_api_security | stable interface plus `activation_controlled_artifact` policy rows | 120 | blocked_validation | authorization, no-existence-leak, audit, and endpoint outcome fixtures required |
| StructuredInputRepositoryRedactionPolicy | `110` | `010`, `030`, `040`, `100`, `120`, `140` | runtime_api_security | stable interface plus `activation_controlled_artifact` policy rows | 120 | blocked_validation | validation-diagnostic, file-path, commit, payload, private-route, and telemetry redaction fixtures required |

## Dependency Rule

A spec may depend on another spec only through a named record, named interface, named algorithm, named error code, named lifecycle state, named acceptance report, or named validation matrix row.

A spec must not restate an imported contract except for a one-sentence reference and the exact imported name.

`140` may import owner contracts by exact name but must not restate domain runtime behavior owned by `020`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, or `130`.

### Core schema dependency map

| Importing spec | Required 040 imports |
| --- | --- |
| `020` | `RawRecordSchema`, `ComputeRawRecordId`, `ValidateCoreRecord`. |
| `030` | `CoreRecordSchema`, `CoreRecordValidationAlgorithm`, `CoreRecordChecksumPolicy`. |
| `050` | `CadastreSilverObservationSchema`, `ValidateCoreRecord`. |
| `060` | `GoldFactSchema`, `FactAbsenceOutcome`, `EvidenceRef`. |
| `070` | `CanonicalEntitySchema`, `SourceAssetSchema`, `IdentifierSchema`. |
| `080` | `GoldFactSchema`, `ComputeGoldFactKeyId`, `ComputeGoldFactId`. |
| `090` | `GraphNodeDeltaShape`, `GraphEdgeDeltaShape`, `EvidenceRef`. |
| `110` | `CoreRecordErrorCodeSet`, `EvidenceRef`, `CommonRecordHeader`. |
| `120` | All exported record schemas and validation algorithms. |
| `docs/nlspec/domain.md` | Vocabulary and owner routing only. |

## Document Registry Consistency Rule

Every registry row must reference a normalized repository-relative path or explicit glob. A registry path must not contain an absolute prefix, backslash, empty segment, `.` segment, or `..` segment.

Every target file in the NLSpec set must have exactly one registry row. Duplicate rows for the same target path are invalid. A manifest path outside the NLSpec set may be represented by an explicit reference, rationale, archive, or navigation row.

No non-040 file may define a field schema for a core record exported by `040`. A downstream spec may import an exact schema name and define behavior for its own stage, but it must not restate field paths, defaults, nullability, ID inputs, checksum inputs, or extension policy for the 040-owned record.

### DefineOnceClosureInventory

`DefineOnceClosureInventory` is a non-runtime governance record. It is produced by `ValidateSpecSet` before promotion and must not be used as product runtime behavior.

| Field | Required behavior |
| --- | --- |
| `contract_name` | Exact runtime contract, validation contract, or owner-routed contract name being checked. |
| `owner_spec` | Exact owner spec ID expected to export the contract. |
| `owner_file_path` | Normalized repository-relative path of the owner file. |
| `owner_export_present` | Boolean result after checking the owner `Exports` list and declared owner-export alias tables. |
| `imported_by` | Canonically sorted list of files that import or route to the contract. Empty array means owner-only. |
| `non_owner_reference_locations` | Canonically sorted file/heading locations where the contract is referenced outside its owner. |
| `permitted_reference_form` | Exact imported contract name plus at most one owner-routing sentence, unless the owner spec explicitly grants a wider validation form. |
| `duplicate_or_contradiction_class` | One closed class from the table below. |
| `required_action` | One of `none`, `add_owner_export`, `remove_restatement`, `merge_domain_rows`, `split_domain_scope`, `resolve_owner_conflict`, or `mark_unresolved_todo`. |
| `validation_row_id` | Exact `120` row that proves the inventory result. |

Closed `duplicate_or_contradiction_class` values are:

```text
none
non_owner_runtime_restatement
duplicate_domain_ledger_row
duplicate_owner_export
owner_runtime_contradiction
unexported_owner_contract_reference
reference_or_rationale_runtime_authority
```

`ValidateSpecSet` must build the inventory deterministically by path order, then heading order, then lexical contract name. The same spec bytes must produce byte-identical inventory rows.

Every runtime contract name used in `Imports`, `domain.md` Section 25, `120` validation rows, or `SpecSetVersion.validation_matrix_refs` must appear in exactly one owner `Exports` list or in one declared owner-export alias table. A missing owner export fails with `OWNER_CONTRACT_REF_UNEXPORTED`. More than one owner export for the same runtime contract fails with `DUPLICATE_OWNER_EXPORT` unless one row is an explicit alias owned by the same file.

A non-owner document may contain only the exact imported contract name plus one owner-routing sentence. It must not copy row schemas, algorithms, defaults, failure precedence, activation behavior, validation harness behavior, field tables, or acceptance-owner decisions.

Two Section 25 rows may point to the same owner contract only when they name different `owner_contract_scope` values and different validation row families. Otherwise validation fails with `DOMAIN_LEDGER_OWNER_DUPLICATE`.

Supporting duplicate inventories, runtime-restatement workbooks, concrete fixture bytes, expected-output checksums, and private source bindings belong in supporting material. Their path is `TODO:` until registered in `MANIFEST.md`.

### Scope selector define-once closure

`DefineOnceClosureInventory` must include the row family `scope_selector_closure` before promotion.

| Contract | Sole runtime owner | Required validation row |
| --- | --- | --- |
| `ScopeSelector` | `030` | `val-000-scope-selector-owner-export-present` |
| `ActivationScope` | `030` | `val-000-scope-selector-owner-export-present` |
| `ScopeSelectorContext` | `030` | `val-000-scope-selector-owner-export-present` |
| `NormalizeScopeSelector` | `030` | `val-000-scope-selector-owner-export-present` |
| `ScopeSelectorCovers` | `030` | `val-000-scope-selector-owner-export-present` |
| `CompareScopeSpecificity` | `030` | `val-000-scope-selector-owner-export-present` |
| `ResolveScopedRow` | `030` | `val-000-scope-selector-owner-export-present` |

Promotion must fail when any active non-`030` owner file uses `activation_scope`, `scope_selector`, `source_scope_selector`, `subject_scope_selector`, `object_scope_selector`, target-environment matching, the phrase `covers the request`, the phrase `exact scope`, or the phrase `equally specific` as selector runtime behavior without importing the exact `030` contract and routing selector behavior to it by name.

Promotion must also fail when any active non-`030` file defines selector schema, selector equality, selector coverage, subset matching, specificity comparison, ambiguity resolution, row-order tie breaking, package-order tie breaking, backend-order tie breaking, file-order tie breaking, or validation-order tie breaking for selector selection instead of using `030.ScopeSelector`, `030.ActivationScope`, `030.ScopeSelectorContext`, `030.NormalizeScopeSelector`, `030.ScopeSelectorCovers`, `030.CompareScopeSpecificity`, and `030.ResolveScopedRow`.

Required validation row families are `120-SCOPE-SELECTOR-CLOSURE-*`, `120-SCOPE-SELECTOR-OWNER-IMPORT-*`, `120-SCOPE-SELECTOR-NO-RESTATEMENT-*`, and `120-SCOPE-SELECTOR-FIXTURE-COVERAGE-*`.

### OwnerLocalStatusConsistencyRule

`ValidateSpecSet` must evaluate owner-local closure state independently from document authority status. The following failure codes are closed for this rule.

| Failure code | Emitted when |
| --- | --- |
| `DOMAIN_OWNER_STATUS_CONTRADICTION` | `domain.md` marks a ledger row unresolved while the owner is `closed_local` and required validation rows pass, or marks a row resolved while any named owner has an unresolved owner TODO or required validation row is `blocked`, `not_run`, or `fail`. |
| `DOMAIN_RUNTIME_RESTATEMENT` | `domain.md` copies row schemas, algorithms, defaults, failure precedence, validation harness behavior, or activation behavior from owners instead of routing by exact owner contract name. |
| `OWNER_SPEC_CONTRADICTION` | Two owner specs define incompatible runtime behavior for the same named contract. |
| `DOMAIN_LEDGER_OWNER_DUPLICATE` | `domain.md` Section 25 has two rows for the same owner contract without distinct owner contract scope and distinct validation row family. |
| `OWNER_CONTRACT_REF_UNEXPORTED` | A contract name used by an import, Section 25 row, validation row, or `SpecSetVersion.validation_matrix_refs` is not exported by exactly one owner or declared owner-export alias. |
| `DUPLICATE_OWNER_EXPORT` | More than one owner exports the same runtime contract name without a same-owner alias declaration. |
| `ADR_STATUS_UNREGISTERED` | An ADR file or registry row uses a status outside `SpecStatus`. |
| `REGISTRY_MANIFEST_PATH_MISMATCH` | A manifest path lacks exactly one registry row or explicit non-registry reason. |
| `VOLATILITY_CLASS_MISSING` | An implementation-relevant artifact or contract lacks exactly one volatility class. |
| `VOLATILITY_CLASS_CONFLICT` | An artifact or contract has conflicting volatility classes. |
| `VOLATILE_ROW_IN_STABLE_CORE` | A stable core spec contains production-active volatile rows rather than non-normative examples or blocking `TODO:` rows. |
| `ACTIVATION_ARTIFACT_OWNER_MISMATCH` | An activation-controlled artifact lacks a matching owner stable core contract. |
| `ACTIVATION_ARTIFACT_RUNTIME_RESTATEMENT` | An activation-controlled artifact defines runtime behavior instead of instantiating exported owner behavior. |

Required consistency checks:

| Check | Required behavior |
| --- | --- |
| Domain unresolved and owners closed | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| Domain resolved and owner blocked | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| Domain runtime restatement | Fail with `DOMAIN_RUNTIME_RESTATEMENT`; `domain.md` must use owner-routing language only and must not copy row schemas, algorithms, defaults, or failure precedence from owners. |
| Domain Section 25 unresolved with owner closed | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION` when owner state is `closed_local` and matching validation rows pass. |
| Domain Section 25 resolved with owner blocked | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION` when any named owner TODO remains or required validation rows are `blocked`, `not_run`, or `fail`. |
| Owner runtime contradiction | Fail with `OWNER_SPEC_CONTRADICTION`; `domain.md` must not pick a side. |
| Domain ledger duplicate owner row | Fail with `DOMAIN_LEDGER_OWNER_DUPLICATE` unless the duplicate rows have distinct `owner_contract_scope` values and distinct validation row families. |
| Owner contract ref unexported | Fail with `OWNER_CONTRACT_REF_UNEXPORTED` when a referenced runtime contract does not resolve to exactly one owner export or owner-export alias. |
| Duplicate owner export | Fail with `DUPLICATE_OWNER_EXPORT` when two owners export the same runtime contract name. |
| ADR status mismatch | Fail with `ADR_STATUS_UNREGISTERED` unless both file and registry row use `accepted_rationale`. |
| Manifest path mismatch | Fail with `REGISTRY_MANIFEST_PATH_MISMATCH`; every manifest path must have one registry row or explicit non-registry reason. |

## Registry and Acceptance Control

No document may be marked `authoritative` when it has unresolved owner `TODO:` rows, duplicate runtime ownership, missing validation evidence, or a missing registry entry.

Promotion to `authoritative` must include the `120.CoreRecordSchemaValidationMatrix` rows for every `040` exported record in `SpecSetVersion.validation_matrix_refs`. Unresolved owner `TODO:` rows in core schemas, owner error codes, fixture checksums, expected output checksums, or downstream cross-references remain implementation exclusions.

Promotion to `authoritative` must include `120-CORE-ONEOF-CLOSURE-*`, `120-CORE-EVIDENCE-ARTIFACT-*`, `120-CORE-NULL-OMISSION-*`, `120-CORE-ID-REPLAY-ONEOF-*`, and `120-CORE-ERROR-REGISTRY-*` row families. `ValidateSpecSet` must fail when `040.CoreOneOfRegistry` contains unresolved rows or when any required core one-of closure validation row is absent, blocked, not run, failed, stale, checksum-mismatched, or `TODO`-bearing.

Promotion to `authoritative` must fail when `040.EvidenceArtifactClassRegistry` pairings for declared artifact classes are missing, ambiguous, inactive, checksum-mismatched, out of scope, or lack validation refs.

`ValidateSpecSet` must run `DefineOnceClosureInventory` before owner acceptance aggregation. Promotion to `authoritative` must fail when any non-deferred inventory row has `duplicate_or_contradiction_class` other than `none`, when a referenced owner contract is unexported, when a runtime contract has duplicate owners, or when a non-owner restates runtime behavior instead of routing by exact contract name.

`SpecSetVersion.validation_matrix_refs` must include `020-FEED-CLOSURE-AC-*`, `030-FEED-CLOSURE-AC-*`, `060-FEED-CLOSURE-AC-*`, `110-FEED-CLOSURE-AC-*`, and `120-FEED-CLOSURE-AC-*` rows before authoritative handoff when any production feed category is active.

`SpecSetVersion.validation_matrix_refs` must include `120-SOURCE-CLOSURE-*` rows and `SourceAuthorityClosureMatrix` validation refs before authoritative handoff when any active feed profile has non-empty `absence_sensitive_domains` or any category row has non-empty `allowed_effects`.

`SpecSetVersion.validation_matrix_refs` must include `120-DOMAIN-SECTION25-*` rows before authoritative handoff. Every row in `domain.md` Section 25 must have one matching owner-local closure state and one validation row family in `120.DomainSection25StatusValidationMatrix`.

`SpecSetVersion.validation_matrix_refs` must include `120-OCSF-MAP-*`, `120-SOURCE-EXT-*`, `120-OCSF-NONAUTH-*`, and `120-OCSF-DIRECTION-*` rows before authoritative handoff when any MVP observation-to-OCSF mapping row set is active.

`SpecSetVersion.validation_matrix_refs` must include analysis/enrichment/lineage/registry handoff rows before authoritative handoff when analysis, enrichment, lineage, registry, derived graph edge, threat-intel, or analysis API output is in implementation scope. Required row families are `120-ANALYSIS-OUTPUT-AUTHORITY-*`, `120-ANALYSIS-REPLAY-*`, `120-THREAT-INTEL-*`, `120-LINEAGE-FACET-*`, `120-REGISTRY-GOVERNANCE-*`, `120-DERIVED-GRAPH-EDGE-*`, `120-ARTIFACT-CLASS-POLICY-*`, and `120-130-ERROR-REGISTRY-*`.

A document that is not marked `authoritative` must not be used as product runtime authority. Candidate documents may be used only for validation or implementation spikes explicitly named by an acceptance report.

`ValidateSpecSet` must fail before promotion when any exported implementation-relevant artifact lacks a volatility class, an activation-controlled artifact has no owner spec, a volatile production row is embedded in a stable core section without being marked example-only or `TODO:`, or a reference, ADR, or archive document is used as runtime authority.

`SpecSetVersion.validation_matrix_refs` must include lifecycle validation rows for every lifecycle machine that can affect production output, activation, graph apply, replay, watermark eligibility, or CI gating. Authoritative handoff must fail when lifecycle ownership or closure state is inconsistent with `domain.md` unresolved or resolved lifecycle status.

`SpecSetVersion.validation_matrix_refs` must include graph closure validation rows before authoritative handoff when graph projection, graph apply, graph query, graph rebuild, graph serving, analysis graph reads, or API graph responses are in implementation scope. Required row families are `120-GRAPH-PROFILE-CLOSURE-*`, `120-GRAPH-QUERY-TRANSLATION-*`, `120-GRAPH-OUTPUT-ELIGIBILITY-*`, `120-GRAPH-APPLY-ORDER-*`, `120-GRAPH-REBUILD-EQUIVALENCE-*`, `120-GRAPH-ENDPOINT-IDENTITY-*`, `120-GRAPH-PAGE-TOKEN-*`, `120-OCSF-DIRECTION-*`, and active reachability-prohibition rows.

`ValidateSpecSet` must fail before promotion when a graph profile, edge semantics row set, output eligibility row set, backend taxonomy mapping profile, query translation profile, read-model schema profile, apply profile, lag policy, graph rebuild manifest, graph index consistency check, or derived-view state lacks a volatility classification, an activation-controlled artifact ref where required, or a matching `030.VersionManifest` requirement.

`SpecSetVersion.validation_matrix_refs` must include `120-TEMPORAL-CORRECTION-*`, `120-ASSERTION-TRANSITION-*`, `120-REPLAY-OUTPUT-CLASS-*`, `120-GRAPH-HANDOFF-*`, and `120-NOOP-ERROR-*` rows before authoritative handoff when temporal, gold, correction, late-arrival, replay, export replay, analysis replay, or graph handoff behavior is in implementation scope.

`ValidateSpecSet` must fail with `DOMAIN_OWNER_STATUS_CONTRADICTION` or a more specific registry error when `000` marks temporal/gold/replay closed while `080` contains blocking placeholder rows in `TemporalSemanticsPolicy`, `GoldFactCorrectionPolicy`, `LateArrivalPolicy`, `ReplayEquivalencePolicy`, `CorrectionSnapshotRefPolicy`, or assertion-state transitions. It must also fail when `080` is closed but required `120` temporal/correction/replay validation rows are missing, or when a replay/correction artifact lacks a volatility class or `030.VersionManifest` representation.

`ValidateSpecSet` must fail before promotion when `130` contains unresolved owner TODO rows, when any required `130` fixture family is absent, blocked, failed, not run, stale, or TODO-bearing, or when any `130` activation-controlled artifact lacks a volatility class, `030.ActivationControlledArtifactRef`, package-set ref when package-supplied, or `030.VersionManifest` requirement.

## Archival Policy

Archived documents are historical reference only and never implementation authority.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `000-SCOPE-SELECTOR-DEFINE-ONCE-AC-001` | `ValidateSpecSet` fails when any non-`030` active spec defines selector schema, equality, subset, specificity, coverage, or ambiguity tie-breaking instead of routing to the exact `030` selector contracts. |
| `000-SCOPE-SELECTOR-DEFINE-ONCE-AC-002` | Every active scoped owner imports or routes to `030.ScopeSelector`, `030.ActivationScope`, `030.ScopeSelectorContext`, `030.ScopeSelectorCovers`, `030.CompareScopeSpecificity`, and `030.ResolveScopedRow` by exact name when its rows use scope matching. |
| `000-AC-001` | Every implementation-relevant behavior has exactly one owning spec. |
| `000-AC-002` | Every active spec declares imports, exports, dependencies, non-scope, and Definition of Done. |
| `000-AC-003` | Archived and reference documents are not cited as implementation authority. |
| `000-DEFINE-ONCE-AC-001` | `ValidateSpecSet` emits a deterministic `DefineOnceClosureInventory` and fails when any runtime contract reference is unexported, duplicate-owned, contradicted, or restated by a non-owner. |
| `000-DEFINE-ONCE-AC-002` | `domain.md` Section 25 contains no duplicate owner-contract rows unless each duplicate has a distinct owner-contract scope and distinct validation row family. |
| `000-DEFINE-ONCE-AC-003` | Every runtime contract in `Imports`, Section 25, `120` validation rows, and `SpecSetVersion.validation_matrix_refs` resolves to exactly one owner export or owner-export alias. |
| `000-AC-004` | Research reports, ADRs, navigation files, and manifest files are marked non-authoritative for runtime behavior. |
| `000-CLEANUP-AC-001` | The file contains no banned reference class. |
| `000-CLEANUP-AC-002` | `SpecStatus` is total for `draft`, `candidate`, `authoritative`, `superseded`, `archived`, `accepted_rationale`, `inactive_deferred`, and `active_standard`. |
| `000-CLEANUP-AC-003` | No export name contains a banned reference class. |
| `000-CLEANUP-AC-004` | The registry has one row for each target file and no row for decomposition-control artifacts. |
| `000-CLEANUP-AC-005` | Promotion to `authoritative` depends only on registry state, owner review, and passing validation evidence. |
| `000-SCHEMA-PATCH-AC-001` | The owner registry names `040` as the sole owner of exported core record schemas. |
| `000-SCHEMA-PATCH-AC-002` | No downstream spec is registered as owner of a 040 core record field schema. |
| `000-SCHEMA-PATCH-AC-003` | Promotion to `authoritative` depends on passing 040 schema validation rows in `120`. |
| `000-VOLATILITY-AC-001` | Every implementation-relevant artifact has exactly one volatility class. |
| `000-VOLATILITY-AC-002` | Activation-controlled artifacts reference an owner stable core contract. |
| `000-VOLATILITY-AC-003` | Stable core specs do not contain production-active volatile row instances. |
| `000-VOLATILITY-AC-004` | Research, ADR, reference, and archive documents remain non-runtime even when cited as evidence. |
| `000-FEED-CLOSURE-AC-001` | `ValidateSpecSet` fails with registry or volatility errors if `LakehouseFeedProfileSchema`, `LakehouseFeedFeasibilityAssessment`, `LakehouseFeedCategoryClosureRowSet`, `LakehouseFeedCompletenessProfileRow`, or `DeclaredDAGSubsetProfile` lacks exactly one owner and volatility classification. |
| `000-FEED-CLOSURE-AC-002` | `SpecSetVersion.validation_matrix_refs` includes the feed-closure acceptance rows before authoritative handoff for any active feed category. |
| `000-TEMPORAL-REGISTRY-AC-001` | `ValidateSpecSet` fails when `000` says temporal/gold/replay is `closed_local` but `080` contains blocking temporal, correction, late-arrival, replay, snapshot-ref, or assertion transition placeholder rows. |
| `000-TEMPORAL-REGISTRY-AC-002` | `ValidateSpecSet` fails when `080` is closed but required `120` temporal/correction/replay validation rows are missing. |
| `000-TEMPORAL-REGISTRY-AC-003` | Every replay/correction artifact has exactly one volatility class and every output-affecting activation-controlled artifact appears in `030.VersionManifest`. |
| `000-OCSF-MAP-AC-001` | `ValidateSpecSet` fails when a new mapping row set lacks a volatility classification, an artifact class exists in `030` but not in `000`, a validation row exists in `120` but is missing from required promotion refs, or a research report is referenced as runtime authority. |
| `000-OCSF-MAP-AC-002` | Every new OCSF mapping row set and policy set is classified as activation-controlled and tied to required `120` validation refs before authoritative handoff. |
| `000-SOURCE-CLOSURE-AC-001` | Promotion fails when an active absence-sensitive feed category lacks `120` validation refs for `SourceAuthorityClosureMatrix`. |
| `000-SOURCE-CLOSURE-AC-002` | Promotion fails when an active absence-sensitive feed category has no active `SourceAuthorityClosureMatrixRowSet` ref or deterministic block row ref. |
| `000-SOURCE-CLOSURE-AC-003` | Promotion fails when any source-closure row catalog is active but lacks a volatility classification, validation refs, activation scope, lifecycle status, or `VersionManifest` inclusion rule. |
| `000-SOURCE-CLOSURE-AC-004` | Promotion fails when `000`, `domain.md`, `020`, `030`, `060`, or `120` use different names for the same source-closure row catalog family. |
| `000-IDENTITY-CLOSURE-AC-001` | Promotion fails when identity output is in implementation scope and `SpecSetVersion.validation_matrix_refs` lacks required identity closure, replay, review, split, explanation, and package weakening validation rows. |
| `000-IDENTITY-CLOSURE-AC-002` | Promotion fails when unresolved `070` identity TODOs exist or when `domain.md`, `040`, `070`, `110`, and `120` disagree on identity decision enum closure or review-state-machine closure. |
| `000-IDENTITY-VOLATILITY-AC-001` | Every identity resolver row-set artifact is classified as activation-controlled and every identity runtime decision/explanation/handoff record is classified as runtime state in exactly one place. |
| `000-GRAPH-PROFILE-CLOSURE-AC-001` | Promotion fails when graph projection, apply, query, rebuild, or serving is in scope and `SpecSetVersion.validation_matrix_refs` lacks required graph closure row families. |
| `000-GRAPH-VOLATILITY-AC-001` | Every graph profile, edge semantics, output eligibility, taxonomy mapping, query translation, schema, apply, lag, rebuild, index consistency, and derived-view state artifact has exactly one volatility class. |
| `000-GRAPH-BACKEND-DEFAULT-AC-001` | Promotion fails when graph serving is in scope and `GraphBackendProfile`, `GraphBackendSelectionPolicy`, provider capability rows, JanusGraph default rows, package gates, or required `120-GRAPH-BACKEND-*` validation refs are missing, inactive, checksum-mismatched, or unresolved. |
| `000-ANALYSIS-REGISTRY-CLOSURE-AC-001` | Promotion fails when analysis, enrichment, lineage, registry, threat-intel, derived-edge, or analysis API output is in implementation scope and required `120` row families for `130` closure are absent, blocked, failed, not run, stale, or fixture-incomplete. |
| `000-ANALYSIS-REGISTRY-TODO-BLOCK-AC-001` | Promotion fails when `130` contains unresolved owner TODO rows or when any `130` error, fixture, acceptance, manifest, package, or volatility row remains TODO-bearing. |
| `000-ANALYSIS-REGISTRY-VOLATILITY-AC-001` | Promotion fails when any `130` activation-controlled artifact lacks a volatility class, `030.ActivationControlledArtifactRef`, package-set ref when package-supplied, or `030.VersionManifest` inclusion rule. |
| `000-PAGE-TOKEN-CLOSURE-AC-001` | Page-token default, minimum, and maximum expiration bounds are closed in `110`; promotion still requires passing page-token validation refs. |
| `000-RESEARCH-RUNTIME-AUTHORITY-AC-001` | Research reports, ADRs, reference, and archive documents fail validation when referenced as runtime authority. |
| `000-DOMAIN-GRAPH-STATUS-AC-001` | `domain.md` graph edge-family status agrees with `090` owner closure and `120` graph validation status. |
| `000-STATUS-CONSISTENCY-AC-001` | `ValidateSpecSet` fails domain/owner closure-state contradictions with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| `000-STATUS-CONSISTENCY-AC-002` | Every row in `domain.md` Section 25 has exactly one matching owner-local closure state and one matching `120` validation row family. |
| `000-STATUS-CONSISTENCY-AC-003` | `ValidateSpecSet` fails runtime restatement in `domain.md` with `DOMAIN_RUNTIME_RESTATEMENT`. |
| `000-STATUS-CONSISTENCY-AC-004` | `ValidateSpecSet` fails owner-spec runtime contradictions with `OWNER_SPEC_CONTRADICTION`. |
| `000-STATUS-CONSISTENCY-AC-005` | ADR status values in files and registry rows are members of `SpecStatus` and use `accepted_rationale` for accepted non-runtime rationale. |
| `000-STATUS-CONSISTENCY-AC-006` | Every manifest path has exactly one registry row or explicit non-registry reason. |
| `000-PACKAGE-CLOSURE-AC-001` | Promotion fails when any required package activation validation group is absent, blocked, not run, failed, or contains `TODO` checksums. |
| `000-PACKAGE-VOLATILITY-AC-001` | Every package activation contract, row set, policy, runtime state record, and release evidence class has exactly one volatility class. |
| `000-PACKAGE-REGISTRY-REFS-AC-001` | Promotion fails when package activation is in scope and `SpecSetVersion.activation_artifact_registry_refs` lacks required package activation registry refs or explicit owner TODO blockers that prevent promotion. |
| `000-PACKAGE-ENUM-STATUS-AC-001` | `ValidateSpecSet` fails if any owner spec marks `PackageType` unresolved while `000` marks it `closed_local`, or marks package-set activation closed while required package validation rows remain blocked. |
| `000-CORE-ONEOF-CLOSURE-AC-001` | Promotion fails when `040.CoreOneOfRegistry` contains unresolved rows or when any `120` core one-of closure validation row is absent, blocked, not run, failed, stale, checksum-mismatched, or `TODO`-bearing. |
| `000-CORE-EVIDENCE-ARTIFACT-CLOSURE-AC-001` | Promotion fails when `EvidenceRef` artifact class/kind pairing rows are missing, ambiguous, inactive, checksum-mismatched, out of scope, or lack validation refs. |
| `000-RUNLOCK-CLOSURE-AC-001` | Promotion fails when run-lock lease policy, stale recovery behavior, fencing-token semantics, commit guard semantics, operation evidence, recovery evidence, error registry rows, or validation refs are missing, blocked, not run, failed, stale, checksum-mismatched, or `TODO`-bearing. |

| `000-ACTIVATION-CATALOG-CLOSURE-AC-001` | Every active production scope has either active row-set refs or exact deterministic block refs for each required closure-pack family, every member ref is validation-backed, and every output-affecting member ref appears in `030.VersionManifest.included_refs`. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
