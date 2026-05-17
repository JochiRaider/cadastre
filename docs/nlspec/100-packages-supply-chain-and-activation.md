---
doc_id: CADASTRE-NLSPEC-100
title: Packages, Supply Chain, and Activation
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

## Exports

- `PackageArtifact`
- `PackageReleaseManifest`
- `ProductionPackageSetManifest`
- `PackageCohesionGroup`
- `PackageTrustPolicy`
- `PackageRepositoryMetadata`
- `PackageRepositorySnapshot`
- `PackageRepositoryFreshnessProof`
- `PackageRepositoryAntiRollbackState`
- `PackageSignatureVerificationResult`
- `PackageAttestationSet`
- `PackageBuildProvenance`
- `PackageSBOMRef`
- `PackageCompatibilityMatrix`
- `PackagePromotionRecord`
- `PackageActivationFailureEvent`
- `PackageDeploymentRevision`
- `LastKnownGoodPackageSet`
- `PackageRollbackPlan`
- `PackageRollbackResult`
- `PackageQuarantineRecord`
- `EmergencyPackageOverrideRecord`
- `PackageStageBinding`
- `PackageDeveloperContract`
- `ActivatePackageSet`
- `RollbackPackageSet`
- `QuarantinePackage`

## Activation Unit

Production package runtime behavior may activate only through an immutable `ProductionPackageSetManifest`. A single `PackageArtifact`, package version string, deployment revision label, mutable repository ref, scalar signature status, dependency lock, SBOM existence, provenance existence, or successful validation run is not a production activation target.

## Package Release Manifest

`PackageReleaseManifest` must name one immutable package release, including package type, artifact digest, size, media type, subject digest, release version, dependency locks, public API version when applicable, package developer contract, stage bindings, validation matrix refs, supply-chain evidence refs, compatibility refs, and release checksum.

## Package Set Manifest

`ProductionPackageSetManifest` must name the full coherent set of package release manifests for a production activation. It must include cohesion groups, target environment, activation mode, validation report refs, trust policy refs, compatibility matrix refs, rollback target refs, approval refs, and package-set checksum.

Partial package-set activation is forbidden unless a future accepted NLSpec defines partial semantics, output boundaries, rollback rules, and acceptance criteria.

## Trust Verification

`PackageSignatureVerificationResult` must be structured evidence. Scalar summaries such as `signature_status = verified` must not authorize activation.

| Evidence | Required behavior |
| --- | --- |
| Artifact digest, size, media type | Must match acquired bytes and release manifest. |
| Subject digest | Must match package artifact or OCI subject when applicable. |
| Signer identity | Must match `PackageTrustPolicy` authorized signer row. |
| Trust root | Must be active and scoped to package namespace/repository. |
| Threshold rule | Must pass when policy requires multiple signatures. |
| Repository metadata | Must pass freshness and anti-rollback checks. |
| Transparency evidence | Required when package policy requires it. |
| Attestation subject | Must match release subject digest. |
| SBOM subject | Must match release subject digest. |

A cryptographically valid signature from an unauthorized signer must fail closed.

## Activation Algorithm

```text
ActivatePackageSet(candidate_set, current_active_set):
1. Verify candidate_set immutable checksum and release manifest checksums.
2. Verify package cohesion groups are complete.
3. Verify repository metadata, freshness proof, anti-rollback state, and artifact bytes.
4. Verify signatures, signer authorization, trust roots, thresholds, and transparency evidence.
5. Verify required attestations, build provenance, SBOM, license, vulnerability, and dependency-graph gates.
6. Verify package compatibility matrix rows.
7. Verify package developer contracts and package stage bindings.
8. Verify validation matrix and required negative fixtures.
9. Persist PackagePromotionRecord.
10. If any check fails, keep current_active_set active, write no candidate production output, and emit PackageActivationFailureEvent.
11. If all checks pass, activate package set and record PackageDeploymentRevision.
12. Mark LastKnownGoodPackageSet only after post-activation health gates pass.
```

## Rollback and Emergency Behavior

Rollback target must be an immutable verified `ProductionPackageSetManifest` checksum. Mutable tags, branches, ranges, deployment revision labels alone, rebuilt artifacts, or tip-of-branch states are forbidden rollback targets.

Emergency override may quarantine, retire, abort candidate activation, roll back to verified last-known-good, or extend deprecation windows. Emergency override must not bypass signature, trust policy, repository metadata, freshness, transparency, or anti-rollback verification for a new production activation.

## Migration finalization contracts

### PackageRepositoryModel catalog

| Repository form | Resolver rule | Metadata rule | Descriptor rule | Digest rule | Evidence-attachment rule | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `oci_digest` | resolve by immutable digest only | repository metadata and referrers required when policy says | OCI descriptor required | subject digest and artifact digest must match manifest | signatures, attestations, SBOM attach to subject | candidate |
| `tuf_compatible_metadata` | resolve through trusted metadata roles | timestamp, snapshot, targets, delegation evidence required | target metadata required | target hash and length verified | trust evidence persists | candidate |
| `local_bundle` | resolve by local immutable checksum | local repository metadata required | bundle descriptor required | bundle checksum verified | evidence bundled and checksummed | candidate |
| `git_tree_snapshot` | resolve by immutable commit/tree checksum only | anti-rollback state required | tree descriptor required | tree checksum verified | evidence refs required | TODO blocking until policy rows exist |

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
| `git_tree_snapshot` | commit timestamp is evidence only; owner approval required | highest-seen approved commit per package namespace |

### Package evidence policies

| Policy | Required behavior |
| --- | --- |
| `PackageProvenancePolicy` | Defines predicate type, builder identity, build type, material rules, subject matching, and expiration. |
| `PackageSBOMPolicy` | Defines supported SBOM formats, subject matching, component/dependency completeness, license and vulnerability gates. |
| `PackageCompatibilityMatrix` | Defines public API, runtime protocol, dependency lock, validation checksum, schema compatibility, graph compatibility, trust, attestation, SBOM, and deprecation window per package type. |
| `PackageCohesionGroup` | Defines packages that activate, roll back, quarantine, and retire together. |
| `PackagePromotionGatePolicy` | Defines approvals, shadow outputs, canary thresholds, health gates, and failure precedence. |
| `LastKnownGoodHealthGate` | Defines post-activation health rows before last-known-good marking. |
| `RollbackCompatibilityPolicy` | Defines immutable rollback target and replay/schema/graph gates. |
| `EmergencyOverrideGovernance` | Allows quarantine, retirement, abort, verified rollback, or deprecation extension only. |
| `QuarantineScopePolicy` | Defines target scope, default blocks, expiration, and successor records. |

### Package type compatibility table

| Package type | Public API | Runtime protocol | Dependency lock | Validation checksum | Schema compatibility | Graph compatibility | Trust policy | Attestation | SBOM | Deprecation window |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| feed reader | package developer contract | feed stage protocol | required when dependencies affect output | required | feed schema refs | none | required | policy-defined | policy-defined | TODO |
| parser | parser contract | parse stage protocol | required | required | raw/schema refs | none | required | policy-defined | policy-defined | TODO |
| mapping | mapping compiler pipeline | normalize stage protocol | required | required | external schema artifact | none | required | policy-defined | policy-defined | TODO |
| resolver profile | resolver contract | identity stage protocol | n/a | required | evidence classes | graph handoff compatibility | required | policy-defined | policy-defined | TODO |
| projection package | graph projection/apply contract | graph stage protocol | required | required | graph schema profile | required | required | policy-defined | policy-defined | TODO |
| validation package | validation matrix contract | validation protocol | required | required | fixture schema | graph fixture when applicable | required | policy-defined | policy-defined | TODO |

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

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `100-PATCH-AC-001` | No package set activates unless immutable artifact, trust, freshness, anti-rollback, compatibility, validation, and package-set evidence pass. |
| `100-PATCH-AC-002` | Candidate activation failure preserves current active package set and writes no candidate production output. |
| `100-PATCH-AC-003` | Rollback targets immutable verified package-set manifests only. |
| `100-PATCH-AC-004` | Package supply-chain validation fixtures cover positive and negative package-set activation cases. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `100-AC-001` | No production package can activate unless immutable artifact, trust, freshness, anti-rollback, compatibility, validation, and package-set evidence pass. |
| `100-AC-002` | Candidate activation failure preserves the current active package set and writes no candidate production output. |
| `100-AC-003` | Rollback targets immutable verified package-set manifests only. |
| `100-AC-004` | Emergency override cannot bypass trust verification for new production activation. |
| `100-AC-005` | Package repository model, trust root governance, SBOM/provenance policy, compatibility matrix, and cohesion group catalogs have exact catalog rows or explicit blocking rows before authoritative status. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageArtifact` | lines 5032-5144 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageReleaseManifest` | lines 5145-5188 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ProductionPackageSetManifest` | lines 5189-5220 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageCohesionGroup` | lines 5221-5244 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageTrustPolicy` | lines 5245-5283 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Package Repository Evidence` | lines 5284-5343 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageSignatureVerificationResult` | lines 5344-5376 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageAttestationSet, PackageBuildProvenance, and PackageSBOMRef` | lines 5377-5431 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageCompatibilityMatrix` | lines 5432-5507 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackagePromotionRecord and PackageActivationFailureEvent` | lines 5508-5545 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageDeploymentRevision, LastKnownGoodPackageSet, PackageRollbackPlan, and PackageRollbackResult` | lines 5546-5566 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageQuarantineRecord and EmergencyPackageOverrideRecord` | lines 5567-5618 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Package Supply-Chain Authority Boundary` | lines 5619-5650 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageStageBinding` | lines 6523-6561 |
| docs/archive/PRD-Cadastre.revised-draft.md | `PackageDeveloperContract` | lines 7388-7412 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Extension Package Lifecycle` | lines 11583-11872 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Package Artifact Acceptance` | lines 13206-13219 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Package Supply-Chain Acceptance` | lines 13575-13609 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
