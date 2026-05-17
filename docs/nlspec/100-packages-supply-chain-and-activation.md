---
doc_id: CADASTRE-NLSPEC-100
title: Packages, Supply Chain, and Activation
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Packages, Supply Chain, and Activation

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

## Definition of Done

| ID | Criterion |
| --- | --- |
| `100-AC-001` | No production package can activate unless immutable artifact, trust, freshness, anti-rollback, compatibility, validation, and package-set evidence pass. |
| `100-AC-002` | Candidate activation failure preserves the current active package set and writes no candidate production output. |
| `100-AC-003` | Rollback targets immutable verified package-set manifests only. |
| `100-AC-004` | Emergency override cannot bypass trust verification for new production activation. |
| `100-AC-005` | Package repository model, trust root governance, SBOM/provenance policy, compatibility matrix, and cohesion group catalogs are resolved before authoritative status. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `PackageArtifact` | lines 5032-5144 |
| PRD-Cadastre.md | `PackageReleaseManifest` | lines 5145-5188 |
| PRD-Cadastre.md | `ProductionPackageSetManifest` | lines 5189-5220 |
| PRD-Cadastre.md | `PackageCohesionGroup` | lines 5221-5244 |
| PRD-Cadastre.md | `PackageTrustPolicy` | lines 5245-5283 |
| PRD-Cadastre.md | `Package Repository Evidence` | lines 5284-5343 |
| PRD-Cadastre.md | `PackageSignatureVerificationResult` | lines 5344-5376 |
| PRD-Cadastre.md | `PackageAttestationSet, PackageBuildProvenance, and PackageSBOMRef` | lines 5377-5431 |
| PRD-Cadastre.md | `PackageCompatibilityMatrix` | lines 5432-5507 |
| PRD-Cadastre.md | `PackagePromotionRecord and PackageActivationFailureEvent` | lines 5508-5545 |
| PRD-Cadastre.md | `PackageDeploymentRevision, LastKnownGoodPackageSet, PackageRollbackPlan, and PackageRollbackResult` | lines 5546-5566 |
| PRD-Cadastre.md | `PackageQuarantineRecord and EmergencyPackageOverrideRecord` | lines 5567-5618 |
| PRD-Cadastre.md | `Package Supply-Chain Authority Boundary` | lines 5619-5650 |
| PRD-Cadastre.md | `PackageStageBinding` | lines 6523-6561 |
| PRD-Cadastre.md | `PackageDeveloperContract` | lines 7388-7412 |
| PRD-Cadastre.md | `Extension Package Lifecycle` | lines 11583-11872 |
| PRD-Cadastre.md | `Package Artifact Acceptance` | lines 13206-13219 |
| PRD-Cadastre.md | `Package Supply-Chain Acceptance` | lines 13575-13609 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
