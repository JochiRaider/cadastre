---
doc_id: RES-013
project: Cadastre
title: Extension Package Supply Chain Report for Cadastre
status: research-report
inspection_date: 2026-05-16
---

## Executive Verdict

Cadastre must move from package artifact activation to **verified package-set activation**. The existing PRD already requires immutable `PackageArtifact` digests, `VersionManifest` inclusion, lifecycle states, validation matrices, dependency locks, source package developer contracts, and stage bindings.[^1][^2][^3][^4][^5] The gap is that a production release is not one package. It is a cohesive set of adapter, parser, mapping, resolver, policy, projection, validation, dependency, trust, provenance, SBOM, and runtime compatibility artifacts. The safe unit for promotion, production activation, canary, rollback, emergency retirement, and last-known-good behavior must therefore be an immutable `ProductionPackageSetManifest` that references immutable `PackageReleaseManifest` records.

The report recommends these PRD changes:

| Rank | Recommendation | Reason | Transfer stance |
| --- | --- | --- | --- |
| 1 | Add `PackageTrustPolicy`, `PackageRepositoryMetadata`, `PackageRepositorySnapshot`, `PackageRepositoryFreshnessProof`, `PackageRepositoryAntiRollbackState`, and `PackageSignatureVerificationResult`. | TUF separates repository roles, thresholds, snapshot/timestamp freshness, rollback protection, mix-and-match protection, and target hash/size verification; Cadastre needs the same observable controls for package acquisition and verification.[^9] | `adopt` |
| 2 | Add `PackageAttestationSet`, `PackageBuildProvenance`, and `PackageSBOMRef` as first-class release evidence. | SLSA and in-toto describe build provenance, builders, materials, products, layouts, and link metadata; SPDX and CycloneDX describe SBOM component, dependency, license, vulnerability, and evidence models.[^10][^11][^20][^21] | `adopt` |
| 3 | Add `PackageReleaseManifest` and `ProductionPackageSetManifest` as immutable activation targets. | OCI descriptors and referrers show digest-addressed artifact identity and attached evidence; GitOps systems show desired-state activation over a set of resources rather than isolated objects.[^13][^18] | `adopt` |
| 4 | Add `PackagePromotionRecord`, `PackageDeploymentRevision`, `LastKnownGoodPackageSet`, `PackageRollbackPlan`, `PackageRollbackResult`, and `PackageActivationFailureEvent`. | Helm, OPA bundles, Argo Rollouts, Argo CD, and Flux expose revisions, keep-current behavior, canary/analysis gates, rollback/remediation, and desired-state sync patterns.[^16][^17][^18] | `adapt` |
| 5 | Add `PackageCompatibilityMatrix`, type-specific public API definitions, and package-type-specific deprecation windows. | SemVer is meaningful only after the public API is declared; Terraform, Buildpacks, Kubernetes, and OpenTelemetry all separate version strings from compatibility, protocols, APIs, or stability states.[^15][^19] | `adopt` |
| 6 | Treat emergency override as bounded retirement, quarantine, or rollback only; forbid signature bypass for production activation. | Sigstore, TUF, Notation, and OPA patterns show that signature verification and policy admission are core trust controls, not optional annotations.[^9][^12][^14][^17] | `tightening` |

The default production rule must be fail-closed:

```text
If candidate activation fails, the current active package set remains active, no production output is written by the failed candidate, and PackageActivationFailureEvent is emitted.
```

The highest-priority PRD correction is operational: `signature_status = verified` must not remain a scalar. It must be replaced or supplemented by a structured verification result that names the artifact subject, digest, media type, repository metadata snapshot, trust policy, trust root, signer identity, threshold rule, transparency evidence, attestation subject, and failure code.

## Source Inventory and Inspection Limits

The report uses NLSpec quality constraints for behavioral completeness, explicit interfaces, defaults, mapping tables, and binary acceptance criteria.[^6]

## Evidence class definitions

| Evidence class | Definition |
| --- | --- |
| `confirmed source fact` | Directly supported by inspected official documentation, specification text, public repository pages, source snippets, or Cadastre project files. |
| `source-grounded inference` | Derived from multiple confirmed facts within one source family. |
| `Cadastre applicability inference` | Derived by comparing confirmed source facts to the governing Cadastre PRD. |
| `speculative transfer` | Plausible transfer requiring product governance, runtime validation, or deeper source inspection before PRD adoption. |
| `open question` | Requires clone/build/run, production data, source owner confirmation, or Cadastre product governance. |

## Inspection limits

All external sources were inspected through public official documentation, specifications, release pages, public repository pages, and rendered source snippets on 2026-05-16. No external repository was cloned, built, installed, executed, benchmarked, or tested in this run. No package signing command, registry resolver, admission controller, Helm release, OPA bundle activation, Argo rollout, Flux reconciliation, Terraform provider installation, SPDX validator, CycloneDX validator, SLSA verifier, or in-toto verifier was run. Runtime behavior claims are therefore limited to inspected source material and labeled inferences.

Cadastre PRD claims are cited only to `/mnt/data/PRD-Cadastre.md`. Prior Cadastre research is cited separately and used as context, not as governing specification.

## Source inventory

| Source name | URL or path | Evidence class | Version/release/inspection date | Material inspected | Material not inspected | Confidence | Freshness risk |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cadastre PRD | `/mnt/data/PRD-Cadastre.md` | governing PRD | uploaded draft, inspected 2026-05-16 | Product summary, contract table, VersionManifest, PackageArtifact, lifecycle | Hidden project docs and implementation repo | High | Medium, draft may change |
| NLSpec guidance | `/mnt/data/nlspec-spec.md` | project standard | 0.2.2 | Behavioral completeness, interfaces, defaults, mappings, acceptance criteria | No implied product behavior beyond spec discipline | High | Low |
| Taxi prior research | `/mnt/data/RES-003-taxi-lang.md` | prior research | 2026-05-15/16 report | Package-manager and direct-runtime-dependency cautions | Original repo not re-executed here | Medium-high | Medium |
| Source-adapter prior research | `/mnt/data/RES-006-source-adapter-ingestion-patterns.md` | prior research | 2026-05-16 report | Adapter state, completeness, live-probe non-authority | Original systems not re-executed here | Medium-high | Medium |
| The Update Framework | TUF spec | official spec | current spec inspected 2026-05-16 | Goals, attacks, roles, thresholds, snapshot/timestamp metadata | Implementation source, client execution | High | Low-medium |
| SLSA and in-toto | SLSA v1.2 docs; SLSA provenance; in-toto docs | official specs/docs | current docs inspected 2026-05-16 | Provenance predicate, builder, materials, layout, link metadata | Verifier execution, layouts from real builds | High | Medium |
| Sigstore/Cosign/Rekor/policy-controller | Official Sigstore docs | official docs | current docs inspected 2026-05-16 | Keyless signing, OIDC identity, bundles, transparency logs, admission policy | CLI execution, Rekor inclusion verification, cluster admission tests | High | Medium |
| OCI artifacts and Notation/Notary | OCI image/distribution specs; Notation docs | official specs/docs | current docs inspected 2026-05-16 | Descriptors, digest, size, mediaType, subject/referrers, trust stores/policies | Registry runtime, Notation verification command | High | Medium |
| Terraform provider lifecycle | Terraform docs | official docs | current docs inspected 2026-05-16 | Version constraints, lock files, checksums, protocol versions | Provider installation execution | High | Medium |
| Helm charts | Helm docs | official docs | current docs inspected 2026-05-16 | Chart metadata, appVersion, kubeVersion, provenance, revisions, rollback | Cluster execution | High | Medium |
| OPA bundles and discovery | OPA docs | official docs | current docs inspected 2026-05-16 | Bundle signing, activation failure, persistence, discovery immutability | OPA runtime execution | High | Medium |
| Argo CD / Rollouts / Flux | Official docs | official docs | current docs inspected 2026-05-16 | Desired-state sync, self-heal, canary steps, analysis abort, Helm rollback remediation | Cluster execution | High | Medium |
| SemVer / Kubernetes / OpenTelemetry / Buildpacks | Official specs/docs | official specs/docs | current docs inspected 2026-05-16 | Public API, deprecation, stability, API compatibility matrices | Runtime compatibility tests | High | Medium |
| SPDX and CycloneDX | Official specs/docs | official specs/docs | SPDX 3.0.1; CycloneDX 1.7 docs inspected 2026-05-16 | SBOM packages, components, dependencies, vulnerabilities, licenses, evidence | Validator execution, generated SBOMs | High | Medium |

## Cadastre Baseline and Immediate Gaps

Cadastre’s PRD already establishes the controlling boundary. The lakehouse is the system of record; graph and Splunk CIM outputs are replaceable projections; authoritative reads and writes must be represented by table-format-native state references; and external systems must not become fact, completeness, identity, graph, or production-approval authority unless a Cadastre contract grants that authority.[^1] The PRD also already names `PackageArtifact`, `PackageStageBinding`, `SourcePackageDeveloperContract`, `LifecycleStatus`, `LifecycleStateMachineDefinition`, `ValidationMatrix`, `VersionManifest`, `ToolchainDependencyReview`, `ExternalToolCapabilityEvidence`, `MappingProjectManifest`, `MappingCompilerPipeline`, `CanonicalValidationOutput`, `StageStateRecord`, `GraphProjectionProfile`, `GraphApplyProfile`, `GraphApplyResult`, `SourceCompletenessProfile`, `SourceCompletenessReceipt`, and `ResolverProfile` as governing contract areas.[^2]

The PRD’s current `VersionManifest` is broad enough to include package artifact digests, dependency locks, resolved dependency digests, validation matrices, source package developer contracts, package stage bindings, source call policies, graph profiles, deployment profiles, and many output-affecting refs.[^3] `PackageArtifact` already requires immutable digests, rejects mutable branches and tags, forbids setup/materialization from writing production outputs, and requires dependency locks when dependencies affect output or validation.[^4] The package lifecycle section already defines package types, required package metadata, lifecycle states, immutable activation, failure-closed contradictory activation evidence, and rollback from deprecated to active only with explicit approval and compatibility.[^5]

The immediate gap is not absence of package lifecycle. The gap is insufficient **supply-chain proof structure** and insufficient **package-set release authority**.

## Gap hypothesis validation

| Gap hypothesis | Verdict | Required PRD refinement |
| --- | --- | --- |
| `signature_status = verified` is operationally incomplete. | Verified. TUF, Sigstore, Notation, Helm, and OPA all distinguish cryptographic validity from trust-root, signer identity, repository metadata, freshness, transparency, and policy authorization.[^9][^12][^14][^16][^17] | Add `PackageSignatureVerificationResult` and `PackageTrustPolicy`; make scalar `signature_status` derived only. |
| Package versioning is not fully package-type-specific. | Verified. SemVer requires a declared public API; Terraform and Buildpacks require protocol/API compatibility, not only version strings.[^15][^19] | Add public API rows per package type and `PackageCompatibilityMatrix`. |
| Promotion is not represented as a package-set manifest. | Verified. GitOps and Helm/Flux represent desired release state and histories around coherent releases; Cadastre packages are interdependent.[^16][^18] | Add `PackageReleaseManifest`, `ProductionPackageSetManifest`, and `PackageCohesionGroup`. |
| Rollback evidence is underspecified. | Verified. Helm rollback and Flux rollback are useful but narrower than Cadastre’s replay/state/schema/graph completeness boundary.[^16][^18] | Add `PackageRollbackPlan` and `PackageRollbackResult` with immutable target manifests, replay, dependency, schema, state, and graph gates. |
| Emergency override is not bounded enough. | Verified. Trust bypass would defeat the lifecycle and supply-chain model. | Permit only quarantine, retirement, activation abort, or rollback to a previously verified manifest; forbid production signature bypass. |
| Deprecation windows are not package-type-specific. | Verified. OTel, Kubernetes, Terraform, and package-class semantics show different compatibility horizons.[^15][^19] | Add `DeprecatedPackageCompatibilityWindowPolicy` by package type. |
| Build provenance and SBOM state are not first-class package evidence. | Verified. SLSA, in-toto, SPDX, and CycloneDX require separate subject, builder, material, product, component, vulnerability, and license evidence.[^10][^11][^20][^21] | Add `PackageAttestationSet`, `PackageBuildProvenance`, and `PackageSBOMRef`. |
| Dependency locks are not enough by themselves. | Verified. Terraform locks constrain provider versions and checksums, but HashiCorp docs explicitly require trust review of checksum signers and lock contents.[^15] | Define lock evidence as reproducibility evidence only, not trust or compatibility proof. |
| Package activation failure behavior needs stronger fail-safe semantics. | Verified. OPA keep-current-on-failure and canary abort patterns show fail-safe activation behavior.[^17][^18] | Add `PackageActivationFailureEvent`, `LastKnownGoodPackageSet`, and fail-closed activation algorithm. |
| Rollback must target immutable package-set manifests. | Verified. OCI tags and Git branches are mutable; TUF and OCI digest identity require immutable subject references.[^9][^13] | Rollback target must be an immutable `ProductionPackageSetManifest` checksum and release checksums, not tags, branches, or rebuild-from-tip. |

## Independent Source Analyses

## The Update Framework Independent Analysis

### Source inventory and inspection limits

Official TUF specification inspected through docs. No implementation source, client, repository, or metadata verification command was run. Scope inspected: goals, attacks, repository roles, threshold signing, root/targets/delegation/snapshot/timestamp metadata, target hash/size, expiration, freeze, rollback, and mix-and-match protections. [^9]

### Problem solved and non-purpose

TUF solves secure software update metadata over untrusted mirrors. It does not solve build provenance, SBOM accuracy, runtime compatibility, package correctness, validation quality, or Cadastre production activation by itself. [^9]

### Architecture and major components

Major components are root, targets, delegated targets, snapshot, timestamp, repository metadata, target files, client trusted metadata, and threshold key rules. Root defines trusted keys and roles. Targets authorize target metadata and delegation. Snapshot fixes a coherent set of metadata versions. Timestamp protects freshness of the snapshot. [^9]

### Data model and artifact model

Artifact identity is target path plus hashes and length. Metadata identity is role metadata version, expiration, signatures, and delegated scopes. Package release identity in Cadastre must be artifact digest plus repository metadata snapshot and target metadata, not mutable package name alone. [^9]

### Control flow and operational model

Client acquires trusted root, refreshes timestamp, fetches snapshot, fetches targets/delegated targets, verifies signatures, thresholds, expiry, rollback counters, hash and length, then downloads target and verifies hash/length. Promotion is outside TUF. [^9]

### Trust, signature, and provenance model

Trust roots are root role keys. Delegation constrains target paths. Thresholds require multiple authorized signatures. Snapshot and timestamp metadata protect mix-and-match, rollback, and freeze attacks. No build provenance is asserted. [^9]

### Versioning and compatibility model

TUF versions metadata and expires it. It is not SemVer and does not define runtime protocol compatibility. Cadastre must adapt metadata version and expiration as repository freshness and anti-rollback state. [^9]

### Promotion, rollback, and emergency behavior

TUF informs safe acquisition. It does not provide canary, shadow, activation, rollback plans, or emergency override behavior. [^9]

### Validation, tests, and acceptance strategies

Validation evidence is cryptographic and metadata-structural: signatures, thresholds, expiration, snapshot consistency, target hash/size. Persisted verification result must be replayable. [^9]

### Confirmed findings versus inferences

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| TUF defines root, targets, snapshot, timestamp roles; target hash/length verification; threshold signing; expiration; and defenses against freeze, rollback, and mix-and-match attacks. | Cadastre can use TUF as the strongest source for repository trust metadata, freshness proof, and anti-rollback state. | Add `PackageRepositoryMetadata`, `PackageRepositorySnapshot`, `PackageRepositoryFreshnessProof`, `PackageRepositoryAntiRollbackState`, and `PackageTrustPolicy`. | Use TUF delegation shape for package-maintainer ownership only after product governance defines package namespace ownership. | Exact repository format and whether Cadastre will implement TUF-compatible metadata or a TUF-inspired profile remains open. |

### Transferability to Cadastre

`adopt` repository trust roles, thresholds, freshness, target hash/size, anti-rollback. `reject` treating TUF as package correctness or compatibility proof.

### Concepts that must not transfer

Do not transfer mutable target paths as package identity; do not treat unexpired timestamp as validation success; do not let repository role ownership authorize runtime stage output.

## SLSA and in-toto Independent Analysis

### Source inventory and inspection limits (slsa-in-toto)

SLSA v1.2 docs, SLSA provenance predicate, and in-toto metadata model were inspected through official docs. No builder, attestation generator, layout verification, or provenance verifier was executed. [^10][^11]

### Problem solved and non-purpose (slsa-in-toto)

SLSA defines supply-chain security levels and provenance expectations. in-toto defines layouts and link metadata for supply-chain steps. They do not prove Cadastre package correctness, source completeness, runtime compatibility, or graph-safety by themselves. [^10][^11]

### Architecture and major components (slsa-in-toto)

SLSA provenance records a subject and predicate describing builder, build type, invocation, parameters, materials, and dependencies. in-toto layouts define expected steps, authorized functionaries, thresholds, materials, products, and inspections; link metadata records evidence for performed steps. [^10][^11]

### Data model and artifact model (slsa-in-toto)

Artifact subject identity is digest-addressed output. Materials and products are named artifacts with digests. Cadastre must require attestation subject digest to equal the package artifact digest or OCI subject digest. [^10][^11]

### Control flow and operational model (slsa-in-toto)

Build step emits provenance or link metadata. Verification checks subject match, builder authorization, buildType allowlist, materials policy, and step/layout consistency. Cadastre validation remains separate. [^10][^11]

### Trust, signature, and provenance model (slsa-in-toto)

Builder identity, functionary keys, thresholds, and attestations are trust inputs. They must be authorized by `PackageTrustPolicy` or `PackageAttestationPolicy` before activation. [^10][^11]

### Versioning and compatibility model (slsa-in-toto)

Build provenance does not define package public API versions. It may record buildType version and builder.id. Compatibility belongs in `PackageCompatibilityMatrix`. [^10][^11]

### Promotion, rollback, and emergency behavior (slsa-in-toto)

No native canary or rollback. Transfer applies to release evidence only. [^10][^11]

### Validation, tests, and acceptance strategies (slsa-in-toto)

Verification evidence must persist subject digest, predicate type, builder.id, buildType, materials digest set, policy ID, verifier version, and failure code. [^10][^11]

### Confirmed findings versus inferences (slsa-in-toto)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| SLSA provenance contains subject, builder.id, buildType, invocation/materials/dependencies. in-toto link metadata records materials and products; layout steps define authorized functionaries and thresholds. | Build provenance and in-toto links are necessary but not sufficient release evidence. | Add `PackageAttestationSet` and `PackageBuildProvenance`; require subject match and authorized builder policy. | Use in-toto layouts for Cadastre package build workflows after package build authority is defined. | Which SLSA level is required for each package type requires product governance. |

### Transferability to Cadastre (slsa-in-toto)

`adopt` subject match, builder authorization, buildType allowlist, materials/products verification. `reject` provenance-as-validation.

### Concepts that must not transfer (slsa-in-toto)

Do not treat existence of provenance as proof of deterministic output, absence of tampering, runtime compatibility, or correctness.

## Sigstore, Cosign, Rekor, and Policy Controller Independent Analysis

### Source inventory and inspection limits (sigstore-cosign)

Official Sigstore/Cosign and Policy Controller docs were inspected. No `cosign` command, Rekor proof verification, Fulcio certificate validation, Kubernetes cluster, or admission controller was executed. [^12]

### Problem solved and non-purpose (sigstore-cosign)

Sigstore provides identity-bound signing, keyless certificate flows, transparency logging, and admission-policy enforcement. It does not define Cadastre package stage compatibility, validation matrices, or package-set promotion. [^12]

### Architecture and major components (sigstore-cosign)

Cosign signs and verifies artifacts and can carry bundles containing signature, certificate, timestamp, and transparency-log proof. Fulcio issues short-lived certificates bound to OIDC identities. Rekor logs signing events. Policy Controller enforces signature and attestation policies at admission. [^12]

### Data model and artifact model (sigstore-cosign)

Key artifact evidence includes certificate identity, OIDC issuer, signature, Rekor log entry, inclusion proof or signed entry timestamp, bundle digest, and subject digest. [^12]

### Control flow and operational model (sigstore-cosign)

Verifier obtains artifact bytes or digest, signature bundle, certificate identity, issuer, and transparency proof. Admission policy evaluates ClusterImagePolicy-like authorities and attestation policies before permitting workload admission. [^12]

### Trust, signature, and provenance model (sigstore-cosign)

Trust roots include Fulcio root and Rekor public key. Authorized signer policy must include certificate identity, OIDC issuer, repository/project scope, and optional transparency requirements. [^12]

### Versioning and compatibility model (sigstore-cosign)

Sigstore does not define package versioning. Policy and trust-root versions affect verification and must appear in `VersionManifest`. [^12]

### Promotion, rollback, and emergency behavior (sigstore-cosign)

Policy Controller is admission-time enforcement. Cadastre must adapt it as activation-time and execution-time gates, not Kubernetes admission by default. [^12]

### Validation, tests, and acceptance strategies (sigstore-cosign)

Persist `PackageSignatureVerificationResult` with identity, issuer, bundle digest, Rekor proof status, trust root ID, policy row ID, verifier version, and failure code. [^12]

### Confirmed findings versus inferences (sigstore-cosign)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Sigstore keyless signing binds OIDC identities to artifact signatures through Fulcio certificates and logs events in Rekor; Policy Controller uses policy gates over signatures and attestations. | Cadastre must verify both cryptographic signature and signer authorization; one without the other is insufficient. | Add authorized signer rows to `PackageTrustPolicy` and transparency proof fields to `PackageSignatureVerificationResult`. | Use admission-style policy syntax only if Cadastre maps every predicate to deterministic activation behavior. | Whether Cadastre will require Rekor inclusion proof, SET, or offline bundle for all production packages remains open. |

### Transferability to Cadastre (sigstore-cosign)

`adopt` identity-bound signing and transparency evidence. `adapt` admission policy to package activation. `reject` signature existence as verification.

### Concepts that must not transfer (sigstore-cosign)

Do not allow a cryptographically valid signature from an unauthorized identity; do not allow identity issuer wildcard by default; do not bypass transparency requirements silently.

## OCI Artifacts, Referrers, and Notation/Notary Independent Analysis

### Source inventory and inspection limits (oci-notation)

OCI image-spec descriptor/referrers material and Notation trust policy docs were inspected. No registry, descriptor resolution, notation verification, or artifact push/pull was executed. [^13][^14]

### Problem solved and non-purpose (oci-notation)

OCI defines content-addressed artifact descriptors and referrers. Notation/Notary defines X.509 trust stores, trust policies, identities, and signature verification. They do not define Cadastre release compatibility or validation success. [^13][^14]

### Architecture and major components (oci-notation)

OCI descriptors carry mediaType, digest, size, and optional artifactType/subject. Referrers attach artifacts such as signatures, SBOMs, and provenance to a subject. Notation trust policy maps registry scopes to trust stores, signature verification level, and trusted identities. [^13][^14]

### Data model and artifact model (oci-notation)

Artifact identity is descriptor digest plus size and mediaType. Tag names are locators only. Referrer identity must include subject digest, artifact descriptor digest, artifact mediaType, and evidence class. [^13][^14]

### Control flow and operational model (oci-notation)

Resolve descriptor by digest, verify mediaType and size, fetch bytes, compute digest, discover referrers by subject digest, verify attached signatures/SBOM/provenance by policy. [^13][^14]

### Trust, signature, and provenance model (oci-notation)

Trust policy selects applicable registry scope, trust store roots, trusted identity rules, and signature verification requirements. Absence of applicable policy must fail closed. [^13][^14]

### Versioning and compatibility model (oci-notation)

OCI digests are content identity, not package version. Media type and artifactType are compatibility axes. Registry tag is not version authority. [^13][^14]

### Promotion, rollback, and emergency behavior (oci-notation)

OCI distribution provides storage and attached evidence, not promotion. Cadastre must wrap OCI artifacts in release manifests and package-set manifests. [^13][^14]

### Validation, tests, and acceptance strategies (oci-notation)

Persist descriptor digest, size, mediaType, artifactType, subject digest, registry scope, referrer digest, signature result, and trust policy decision. [^13][^14]

### Confirmed findings versus inferences (oci-notation)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| OCI descriptors include mediaType, digest, and size; referrers associate artifacts with subjects. Notation trust policies define trust stores, registry scopes, signature verification, and trusted identities. | Cadastre must use OCI descriptor identity for `PackageArtifactRef` and attached evidence graph when OCI packaging is selected, and must reject tag identity. | Add `PackageArtifactRef` fields to release manifests and evidence attachment rows for SBOM/provenance/signatures. | Use OCI artifact manifests for all package types only if local bundle and Git tree packaging can be mapped without loss. | Whether Cadastre package repositories must be OCI registries or may be TUF-backed non-OCI repositories remains open. |

### Transferability to Cadastre (oci-notation)

`adopt` digest/size/mediaType/subject/referrer. `adapt` Notation trust policies. `reject` OCI tag identity.

### Concepts that must not transfer (oci-notation)

Do not activate by image tag, registry path, tag plus digest mismatch, or referrer existence without subject match.

## Terraform Provider Lifecycle Independent Analysis

### Source inventory and inspection limits (terraform-provider)

Terraform dependency lock, provider lock, and plugin protocol docs were inspected. No provider install, checksum verification, or plugin protocol negotiation was executed. [^15]

### Problem solved and non-purpose (terraform-provider)

Terraform solves provider version selection, provider checksum locking, and CLI/provider protocol compatibility. It does not solve package trust, provenance, SBOM, validation, or Cadastre stage safety. [^15]

### Architecture and major components (terraform-provider)

Provider requirements contain version constraints. `.terraform.lock.hcl` records selected provider versions and checksums. `terraform providers lock` can pre-populate platform checksums. Plugin protocol major versions define CLI/provider compatibility. [^15]

### Data model and artifact model (terraform-provider)

Dependency identity is provider source address, version, platform, checksum set, and signature signer review. Runtime compatibility is protocol version, not package SemVer alone. [^15]

### Control flow and operational model (terraform-provider)

Resolve constraints, select provider version, verify against lock/checksums, then negotiate plugin protocol. Upgrade requires explicit lock update. [^15]

### Trust, signature, and provenance model (terraform-provider)

Lock files constrain reproducibility but are not full trust. Checksum signer trust still needs review and policy. [^15]

### Versioning and compatibility model (terraform-provider)

Version constraints and protocol versions are separate axes. Cadastre packages need constraints, locks, resolved digests, and runtime protocol compatibility matrix rows. [^15]

### Promotion, rollback, and emergency behavior (terraform-provider)

No staged production promotion or rollback model beyond lock selection and state management. [^15]

### Validation, tests, and acceptance strategies (terraform-provider)

Validate dependency locks before package validation or runtime execution. Reject unresolved dependency, checksum mismatch, platform-missing lock, or protocol incompatibility. [^15]

### Confirmed findings versus inferences (terraform-provider)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Terraform lock files record selected provider versions/checksums; provider protocol major versions define CLI/plugin compatibility; provider lock docs warn review of signatures/checksums remains necessary. | Cadastre dependency locks must be reproducibility evidence only. | Add `PackageDependencyConstraint`, `PackageDependencyLock`, and runtime protocol rows in `PackageCompatibilityMatrix`. | Terraform lock syntax is not necessarily the right Cadastre serialization. | Cadastre package dependency solver strategy and offline mirror policy remain open. |

### Transferability to Cadastre (terraform-provider)

`adopt` lock/checksum/protocol separation. `reject` lock-as-trust.

### Concepts that must not transfer (terraform-provider)

Do not accept live dependency resolution at activation or replay; do not let dependency constraints override an immutable lock.

## Helm Charts Independent Analysis

### Source inventory and inspection limits (helm)

Helm chart, provenance, history, and rollback docs were inspected. No chart install, verify, rollback, or cluster test was executed. [^16]

### Problem solved and non-purpose (helm)

Helm packages Kubernetes manifests as charts, tracks release revisions, supports provenance files, and supports rollback by revision. It does not perform Cadastre replay, graph validation, source completeness, or package-set compatibility. [^16]

### Architecture and major components (helm)

Chart metadata includes chart version, appVersion, kubeVersion constraints, dependencies, and deprecation flag. Provenance files sign chart package hash and metadata. Release history records revisions and statuses. Rollback restores a prior release revision. [^16]

### Data model and artifact model (helm)

Chart version and appVersion are distinct. Release revision is deployment history, not package artifact identity. Provenance file is signature evidence over chart archive. [^16]

### Control flow and operational model (helm)

Package chart, optionally sign; verify chart/provenance; install or upgrade; record release revision; rollback to prior revision when requested. [^16]

### Trust, signature, and provenance model (helm)

Helm provenance uses key-based signing. It does not define repository-level TUF freshness or authorized signer policy unless wrapped by Cadastre. [^16]

### Versioning and compatibility model (helm)

Chart SemVer describes chart package changes. appVersion is informational. kubeVersion is a target-platform constraint. [^16]

### Promotion, rollback, and emergency behavior (helm)

Rollback and revision history are transferable as deployment revision evidence, but Cadastre rollback must be package-set and replay-aware. [^16]

### Validation, tests, and acceptance strategies (helm)

Validate chart package hash/provenance, chart version, dependency lock, kubeVersion-like target constraints, dry-run output, and revision evidence only as bounded evidence. [^16]

### Confirmed findings versus inferences (helm)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Helm chart version differs from appVersion; kubeVersion uses SemVer constraints; provenance files verify chart origin/integrity; release history and rollback use revisions. | Cadastre package version axes must separate package version, artifact digest, app/runtime target, and deployment revision. | Add `PackageDeploymentRevision` and use Helm rollback only as a cautionary reference for `PackageRollbackPlan`. | Helm-style release history may inform UI for package deployment history. | Whether Cadastre deployment revisions are global or per-environment needs product governance. |

### Transferability to Cadastre (helm)

`adapt` chart/app version separation and release revisions. `reject` Helm rollback as Cadastre rollback.

### Concepts that must not transfer (helm)

Do not rollback by mutable chart repository state; do not infer Cadastre replay safety from Helm revision success.

## OPA Bundles and Discovery Independent Analysis

### Source inventory and inspection limits (opa)

OPA bundle, discovery, and CLI docs were inspected. No OPA server, bundle signing, discovery activation, or persisted-bundle recovery was executed. [^17]

### Problem solved and non-purpose (opa)

OPA bundles distribute policy/data with optional signatures and activation behavior. Discovery bootstraps OPA configuration. OPA does not validate Cadastre source packages beyond policy-bundle scope. [^17]

### Architecture and major components (opa)

Bundle service delivers tar/gzip bundles. Signed bundles include signatures metadata over files. Activation verifies signatures when configured. Discovery can load remote configuration and may persist activated bundles. [^17]

### Data model and artifact model (opa)

Bundle identity is download ETag/manifest/signature/file hashes. Scope and key ID constrain signatures. Persisted active bundle is last-known-good policy state for OPA only. [^17]

### Control flow and operational model (opa)

Download bundle, verify signatures when configured, parse/evaluate, activate on success, continue current active bundle on verification or activation failure. Discovery keys cannot be modified by discovery-generated config. [^17]

### Trust, signature, and provenance model (opa)

Signing keys and scope configure trust. Missing signatures fail when signing is configured. Trust-root immutability in discovery is highly transferable. [^17]

### Versioning and compatibility model (opa)

OPA capabilities can validate policy compatibility with OPA versions. Bundle revision is not SemVer by itself. [^17]

### Promotion, rollback, and emergency behavior (opa)

The strongest transfer is keep-current-on-failure and persisted current bundle. No broad package-set rollback model. [^17]

### Validation, tests, and acceptance strategies (opa)

Activation failure must be evented and must not mutate active behavior. Persist exact bundle digest/signature result and active bundle ID. [^17]

### Confirmed findings versus inferences (opa)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| OPA signed bundles activate only if verification succeeds; failed activation continues existing bundle; discovery signing keys are not modifiable through discovery; persisted activated bundles support recovery. | Cadastre must keep current package set active on failed candidate activation. | Add `LastKnownGoodPackageSet` and fail-safe activation semantics. | OPA capability documents may inspire package runtime capability matrices. | Cadastre policy bundle signing scope conventions require governance. |

### Transferability to Cadastre (opa)

`adopt` keep-current-on-failure and trust-root immutability. `reject` OPA bundle success as universal package success.

### Concepts that must not transfer (opa)

Do not allow failed candidate package to write production outputs; do not let discovery config mutate its own trust root.

## Argo CD, Argo Rollouts, and Flux Independent Analysis

### Source inventory and inspection limits (argo-flux)

Official Argo CD automated sync, Argo Rollouts canary/analysis, and Flux HelmRelease docs were inspected. No cluster, controller, rollout, analysis run, or reconciliation test was executed. [^18]

### Problem solved and non-purpose (argo-flux)

These systems solve desired-state reconciliation, sync, rollout strategies, metric-based analysis, abort, remediation, and drift correction for Kubernetes resources. They do not define Cadastre facts or package validation correctness. [^18]

### Architecture and major components (argo-flux)

Argo CD compares Git desired state to live cluster state and can auto-sync/prune/self-heal. Argo Rollouts executes canary steps and analyses that can abort. Flux HelmRelease reconciles chart sources and release state and supports install/upgrade remediation including rollback. [^18]

### Data model and artifact model (argo-flux)

Desired state is Git commit/ref plus rendered manifests or chart artifact. Rollout state includes steps, weights, analysis runs, abort/degraded status. Flux status records release history and last successful releases. [^18]

### Control flow and operational model (argo-flux)

Reconcile desired state, perform sync or rollout steps, run analysis gates, abort or rollback on failure based on remediation policy, record status/history. [^18]

### Trust, signature, and provenance model (argo-flux)

GitOps trust is not artifact trust. It must be combined with artifact digest/signature/provenance verification for Cadastre. [^18]

### Versioning and compatibility model (argo-flux)

Git commit ref and chart artifact revision are deployment identity, not package public API compatibility. [^18]

### Promotion, rollback, and emergency behavior (argo-flux)

Transfer desired-state package-set promotion, canary/shadow gates, analysis abort, last successful release, and rollback remediation. [^18]

### Validation, tests, and acceptance strategies (argo-flux)

Persist promotion record, approvals, canary steps, metric results, abort reasons, shadow result checksums, and target manifest ID. [^18]

### Confirmed findings versus inferences (argo-flux)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Argo CD syncs Git desired state to live state and supports self-heal/prune controls. Argo Rollouts canary steps and analysis can abort. Flux HelmRelease includes remediation/rollback strategy and release history. | Cadastre promotion must record the desired immutable manifest, not only operator intent. | Add `PackagePromotionRecord`, canary/shadow gates, approval quorum, and rollback-to-last-known-good behavior. | Use controller-like reconciliation loop only if Cadastre defines idempotent activation and lock boundaries. | Metric definitions for Cadastre canary gates require product and domain authority. |

### Transferability to Cadastre (argo-flux)

`adapt` GitOps desired state and rollout gates. `reject` drift correction as Cadastre fact correction.

### Concepts that must not transfer (argo-flux)

Do not let Git branch head, controller status, or live drift correction become package truth or source fact authority.

## SemVer, Kubernetes Deprecation Policy, OpenTelemetry Stability, and Cloud Native Buildpacks Independent Analysis

### Source inventory and inspection limits (semver-k8s-otel-buildpacks)

SemVer, Kubernetes deprecation policy, OpenTelemetry stability/versioning, Buildpacks Buildpack/Platform API, and lifecycle README were inspected. No API compatibility test was run. [^19]

### Problem solved and non-purpose (semver-k8s-otel-buildpacks)

These sources define public API versioning, deprecation windows, stability lifecycle, and protocol/API compatibility matrices. They do not define Cadastre package trust or validation gates. [^19]

### Architecture and major components (semver-k8s-otel-buildpacks)

SemVer requires a declared public API before major/minor/patch has meaning. Kubernetes defines removal windows by stability level. OpenTelemetry defines component lifecycle states. Buildpacks separate Buildpack API, Platform API, and lifecycle support matrices. [^19]

### Data model and artifact model (semver-k8s-otel-buildpacks)

Compatibility identity includes public API, stability state, lifecycle/deprecation state, protocol/API version, and supported target matrix. [^19]

### Control flow and operational model (semver-k8s-otel-buildpacks)

Producers declare API/stability/version. Consumers compare compatibility matrix and deprecation window before use. Lifecycle supports buildpacks when both declare a common API version. [^19]

### Trust, signature, and provenance model (semver-k8s-otel-buildpacks)

No signing/provenance model. [^19]

### Versioning and compatibility model (semver-k8s-otel-buildpacks)

This is the core source for Cadastre package-type-specific versioning. Version strings are not eligibility decisions; compatibility matrices are. [^19]

### Promotion, rollback, and emergency behavior (semver-k8s-otel-buildpacks)

Deprecation and removal windows must be package-type-specific and tied to rollback/replay eligibility. [^19]

### Validation, tests, and acceptance strategies (semver-k8s-otel-buildpacks)

Activation must evaluate declared public API diff, compatibility target, runtime protocol, deprecation window, and matrix row. [^19]

### Confirmed findings versus inferences (semver-k8s-otel-buildpacks)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| SemVer requires declared public API and defines major/minor/patch triggers. Kubernetes and OpenTelemetry define stability/deprecation/removal policies. Buildpacks define Platform API and Buildpack API compatibility between lifecycle and buildpacks. | Cadastre cannot interpret package version without package-type public API rows. | Add package-type versioning matrix and `PackageCompatibilityMatrix`. | Default deprecation windows need product governance and may differ by package class. | Cadastre contract-version scheme across packages remains open. |

### Transferability to Cadastre (semver-k8s-otel-buildpacks)

`adopt` public API first, compatibility matrix second, version string third.

### Concepts that must not transfer (semver-k8s-otel-buildpacks)

Do not treat SemVer as meaningful when public API is undefined; do not activate incompatible protocol versions because version range matches.

## SPDX and CycloneDX SBOM Standards Independent Analysis

### Source inventory and inspection limits (spdx-cyclonedx)

SPDX 3.0.1 package/SBOM/vulnerability/package verification/license docs and CycloneDX overview/v1.7 reference/release docs were inspected. No SBOM validator or generator was run. [^20][^21]

### Problem solved and non-purpose (spdx-cyclonedx)

SPDX and CycloneDX model bills of materials, components/packages, dependencies, licenses, vulnerabilities, evidence, and related transparency data. They do not prove package trust, build authenticity, or runtime compatibility. [^20][^21]

### Architecture and major components (spdx-cyclonedx)

SPDX has profiles/classes for packages, SBOMs, vulnerabilities, license identifiers, integrity/content identifiers, and relationships. CycloneDX has BOM metadata, components, services, dependencies, vulnerabilities, compositions/completeness, evidence, external references, and formulation/citations. [^20][^21]

### Data model and artifact model (spdx-cyclonedx)

SBOM identity must include format, spec version, SBOM document digest, target package/artifact digest, component refs, dependency graph, license data, vulnerability data, and completeness declaration when available. [^20][^21]

### Control flow and operational model (spdx-cyclonedx)

Generate SBOM from build/materials or analysis; attach SBOM to artifact; verify SBOM document digest and subject match; evaluate license/vulnerability/completeness policy; persist result. [^20][^21]

### Trust, signature, and provenance model (spdx-cyclonedx)

SBOM trust depends on subject match, producer identity, signature/provenance, and policy. SBOM contents are inventory evidence, not proof of safety. [^20][^21]

### Versioning and compatibility model (spdx-cyclonedx)

SBOM spec version and component versions are separate axes. Vulnerability data may change faster than package release. [^20][^21]

### Promotion, rollback, and emergency behavior (spdx-cyclonedx)

SBOM gates can block promotion or require waiver, but rollback cannot rely on SBOM alone. [^20][^21]

### Validation, tests, and acceptance strategies (spdx-cyclonedx)

Activation must reject missing required SBOM, subject mismatch, unsupported format, invalid checksum, disallowed license, critical vulnerability without waiver, or opaque dependency graph when policy requires completeness. [^20][^21]

### Confirmed findings versus inferences (spdx-cyclonedx)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| SPDX packages may represent archives, modules, container images, or Git snapshots; SPDX SBOM describes content/composition/provenance/licenses/security issues. CycloneDX models BOM metadata, components, services, dependencies, vulnerabilities, evidence, and dependency graph opacity when omitted. | Cadastre SBOMs must be attached evidence with explicit subject match and policy evaluation. | Add `PackageSBOMRef` and SBOM gate rows in `PackagePromotionRecord`. | Use both SPDX and CycloneDX formats only if profile-specific validation is defined. | Default accepted SBOM formats and minimum fields require product governance. |

### Transferability to Cadastre (spdx-cyclonedx)

`adopt` component/dependency/license/vulnerability inventory. `reject` SBOM-as-trust.

### Concepts that must not transfer (spdx-cyclonedx)

Do not treat a lock file or SBOM as security approval; do not infer dependency-free from a missing dependency graph.

## Taxi and Prior Cadastre Adapter/Package Research Independent Analysis

### Source inventory and inspection limits (taxi-adapter-research)

Prior Cadastre reports RES-003 and RES-006 were inspected through local files. The original external sources in those reports were not re-executed here. [^7][^8]

### Problem solved and non-purpose (taxi-adapter-research)

Prior research informs Cadastre package-manager gaps, mapping-bundle tooling, dependency locks, source package boundaries, adapter state, completeness evidence, and non-authoritative live probes. It is not governing specification. [^7][^8]

### Architecture and major components (taxi-adapter-research)

Taxi package patterns include name/version/dependencies/repositories/plugins/compiler options, but prior research warns Cadastre needs lifecycle state, owner, checksum, signature status, effective intervals, compatibility targets, locks, hashes, schema checksums, and golden corpus. Source-adapter research recommends deterministic collectors with explicit state and completeness evidence. [^7][^8]

### Data model and artifact model (taxi-adapter-research)

Relevant Cadastre package evidence includes mapping project manifests, dependency locks, source schema checksums, semantic overlay checksums, compiler/linter versions, validation output checksums, fixture IDs, and stage state records. [^7][^8]

### Control flow and operational model (taxi-adapter-research)

Authoring package -> validate mapping/compiler/linter -> materialize immutable artifact -> bind to stage -> include in VersionManifest -> execute only within stage output permissions. [^7][^8]

### Trust, signature, and provenance model (taxi-adapter-research)

Prior research does not supply external trust roots; it confirms Cadastre must add stronger package trust controls. [^7][^8]

### Versioning and compatibility model (taxi-adapter-research)

Taxi-like versioned packages are useful but insufficient without public API, locks, checksums, compatibility, lifecycle, and validation outputs. [^7][^8]

### Promotion, rollback, and emergency behavior (taxi-adapter-research)

Prior research supports golden corpus, shadow execution, and stricter activation evidence, but not full package-set promotion. [^7][^8]

### Validation, tests, and acceptance strategies (taxi-adapter-research)

Use mapping compiler pipelines, canonical validation output, recorded source fixtures, source-call behavior fixtures, and forbidden-output tests. [^7][^8]

### Confirmed findings versus inferences (taxi-adapter-research)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Taxi direct runtime dependency is rejected until buildability, artifact availability, exact version pinning, deterministic package validation, and security review are verified. Source-adapter research rejects destination cleanup, live-query evidence, provenance-as-truth, ack-as-completeness, and CDC-time-as-valid-time. | Cadastre package supply chain must extend current package materialization with trust, provenance, SBOM, and package-set gates. | Tighten `PackageArtifact`, `VersionManifest`, `SourcePackageDeveloperContract`, `MappingProjectManifest`, `ValidationMatrix`, and `StageStateRecord`. | Some Taxi package-manager mechanics may inform authoring workflow only. | Exact package repository implementation remains open. |

### Transferability to Cadastre (taxi-adapter-research)

`adapt` authoring/dependency lessons and state/completeness fixtures. `reject` external package manager as production authority.

### Concepts that must not transfer (taxi-adapter-research)

Do not take direct Taxi runtime dependency; do not let live source probes or source package helpers mutate production output outside stage contracts.

## Optional Package Ecosystems Independent Analysis

### Source inventory and inspection limits (ecosystems)

Optional ecosystems such as Nix/Guix, npm/pnpm, Cargo, Go modules, Maven, Debian APT, Kubernetes Operator Lifecycle Manager, Kyverno, and Gatekeeper were not deeply inspected in this report. They are intentionally deprioritized because the priority sources already supply the key contracts needed for identity, trust, provenance, locking, compatibility, promotion, rollback, and policy gates. [^15][^19][^20][^21]

### Problem solved and non-purpose (ecosystems)

Optional ecosystems may expose useful lock, reproducible build, module checksum, repository metadata, policy admission, or operator upgrade patterns. They must not be imported without source-specific inspection. [^15][^19][^20][^21]

### Architecture and major components (ecosystems)

No detailed architecture claim is made for uninspected optional systems. [^15][^19][^20][^21]

### Data model and artifact model (ecosystems)

No uninspected optional-system field is proposed as a Cadastre contract field. [^15][^19][^20][^21]

### Control flow and operational model (ecosystems)

No uninspected optional-system workflow is proposed as Cadastre behavior. [^15][^19][^20][^21]

### Trust, signature, and provenance model (ecosystems)

Trust patterns from uninspected ecosystems remain open questions. [^15][^19][^20][^21]

### Versioning and compatibility model (ecosystems)

Versioning patterns from uninspected ecosystems remain open questions. [^15][^19][^20][^21]

### Promotion, rollback, and emergency behavior (ecosystems)

Promotion/rollback patterns from uninspected ecosystems remain open questions. [^15][^19][^20][^21]

### Validation, tests, and acceptance strategies (ecosystems)

Optional systems may be added only through a future source discovery pass with primary-source evidence. [^15][^19][^20][^21]

### Confirmed findings versus inferences (ecosystems)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| No primary-source claims are made in this report for optional systems beyond the family names listed by the user. | The already inspected sources cover the required Cadastre pressure points for this report. | Keep optional ecosystems as follow-up research only. | Nix/Guix may add stronger reproducibility patterns; npm/pnpm/Cargo/Go/Maven may add language lock nuances; OLM/Kyverno/Gatekeeper may add admission/policy patterns. | Which optional ecosystem to study next depends on Cadastre’s implementation language and package repository choice. |

### Transferability to Cadastre (ecosystems)

`study` only.

### Concepts that must not transfer (ecosystems)

Do not cite uninspected ecosystem behavior as PRD basis.

## Cross-Source Comparison

| Control area | Strongest external pattern | Cadastre-safe interpretation | Default Cadastre stance |
| --- | --- | --- | --- |
| Artifact identity | OCI descriptor digest, size, mediaType; TUF target hash/length | Identity must be digest-addressed and media-type checked. Mutable tags and branches are locators only. | `adopt` immutable digest identity |
| Repository trust | TUF roles and Notation trust policy | Repository metadata and signature policy must be separate from artifact bytes. | `adopt` trust policy and freshness proof |
| Signing | Sigstore/Cosign, Helm provenance, OPA bundles | Signature must bind artifact subject to authorized signer and trust root. | `adopt` structured verification result |
| Transparency | Rekor log entry, inclusion proof, SET | Transparency evidence is required only when the active trust policy requires it. | `adapt` |
| Provenance | SLSA provenance and in-toto link/layout | Build proof must match subject digest and authorized builder. | `adopt` |
| SBOM | SPDX/CycloneDX | SBOM is component/dependency/license/vulnerability evidence, not trust. | `adopt with constraints` |
| Dependency lock | Terraform lock | Lock is reproducibility evidence and checksum constraint, not signer/trust/validation proof. | `adopt with boundary` |
| Compatibility | Terraform protocol, Buildpack API matrix, Kubernetes/OTel stability | Compatibility matrix determines eligibility; SemVer helps only after public API is defined. | `adopt` |
| Promotion | GitOps desired state and rollout records | Promotion target must be immutable package-set manifest plus approval and gate evidence. | `adapt` |
| Canary/shadow | Argo Rollouts/analysis | Canary success is a gate, not universal correctness. | `adapt` |
| Activation failure | OPA keep-current-on-failure | Failed candidate must not alter active production behavior. | `adopt` |
| Rollback | Helm revision / Flux rollback | Rollback target must be immutable package-set manifest plus replay/compatibility checks. | `adapt` |
| Emergency | Trust systems fail closed | Emergency may block, quarantine, retire, or rollback; it must not bypass signature/trust for a new candidate. | `tightening` |

## Transferable Architecture and Data-Model Patterns

## Package release graph

Cadastre must model package release evidence as a directed, immutable evidence graph:

```text
PackageReleaseManifest
  -> PackageArtifactRef[]
  -> PackageSignatureVerificationResult[]
  -> PackageRepositorySnapshot
  -> PackageRepositoryFreshnessProof
  -> PackageRepositoryAntiRollbackState
  -> PackageAttestationSet
  -> PackageBuildProvenance
  -> PackageSBOMRef[]
  -> PackageDependencyLock[]
  -> PackageCompatibilityMatrix
  -> ValidationMatrix
  -> PackageStageBinding or non-stage authority binding
```

`ProductionPackageSetManifest` is then the production activation unit:

```text
ProductionPackageSetManifest
  -> PackageReleaseManifest[]
  -> PackageCohesionGroup[]
  -> PackageCompatibilityMatrix aggregate decision
  -> LastKnownGoodPackageSet target
  -> rollback_target_manifest_id
  -> activation gates
```

## Required source-to-Cadastre mapping tables

### Trust and signature mapping

| External concept | Cadastre analogue | Required Cadastre constraint | Transfer stance |
| --- | --- | --- | --- |
| TUF root role | `PackageTrustPolicy.trust_root_set` | Must identify root keys, role versions, thresholds, expiration, and rotation rules. | `adopt` |
| TUF targets role | `PackageRepositoryMetadata.targets_role_ref` | Must authorize artifact target metadata and path namespace. | `adopt` |
| TUF delegated target role | `PackageRepositoryMetadata.delegation_rows` | Must constrain package namespace and authorized maintainers. | `adapt` |
| TUF snapshot metadata | `PackageRepositorySnapshot` | Must bind a coherent set of repository metadata versions. | `adopt` |
| TUF timestamp metadata | `PackageRepositoryFreshnessProof` | Must prove snapshot freshness and reject expired metadata. | `adopt` |
| TUF target hash and size | `PackageArtifactRef.digest` and `size_bytes` | Must match exact bytes before activation. | `adopt` |
| Sigstore certificate identity | `PackageTrustPolicy.authorized_signer.identity` | Must match exact identity or policy-approved pattern. Wildcard default is forbidden. | `adopt` |
| Sigstore OIDC issuer | `PackageTrustPolicy.authorized_signer.oidc_issuer` | Must match configured issuer. | `adopt` |
| Sigstore transparency log entry | `PackageSignatureVerificationResult.transparency_log_ref` | Required when trust policy requires transparency. | `adapt` |
| Cosign signature | `PackageSignatureVerificationResult.signature_ref` | Must bind artifact subject digest to signer evidence. | `adopt` |
| Rekor inclusion proof or SET | `PackageSignatureVerificationResult.rekor_evidence` | Must verify inclusion or SET when configured. | `adapt` |
| Helm provenance file | `PackageSignatureVerificationResult.provenance_signature_ref` | May serve as signature evidence only under Helm package policy. | `adapt` |
| OPA signed bundle | `PackageSignatureVerificationResult.bundle_signature_ref` | May serve as bundle signature evidence only for policy bundles. | `adapt` |
| OCI descriptor digest | `PackageArtifactRef.digest` | Must be the immutable artifact identity. | `adopt` |
| OCI descriptor mediaType | `PackageArtifactRef.media_type` | Must match package type allowlist. | `adopt` |
| OCI descriptor size | `PackageArtifactRef.size_bytes` | Must match fetched bytes and bounds. | `adopt` |
| OCI referrer | `PackageEvidenceAttachment` | Must reference exact subject digest and evidence digest. | `adopt` |

### Provenance and attestation mapping

| External concept | Cadastre analogue | Required Cadastre constraint | Transfer stance |
| --- | --- | --- | --- |
| SLSA provenance subject | `PackageBuildProvenance.subject_digest` | Must equal package artifact digest or release subject digest. | `adopt` |
| `builder.id` | `PackageBuildProvenance.builder_id` | Must match authorized builder policy for package type. | `adopt` |
| `buildType` | `PackageBuildProvenance.build_type` | Must match allowlist and version constraints. | `adopt` |
| source materials | `PackageBuildProvenance.source_materials` | Must include immutable refs and digests when build output depends on source. | `adopt` |
| dependency materials | `PackageBuildProvenance.dependency_materials` | Must reconcile with dependency lock and resolved digest set. | `adopt` |
| in-toto layout | `PackageAttestationPolicy.layout_ref` | May define expected steps and functionary thresholds. | `adapt` |
| in-toto link metadata | `PackageAttestationSet.link_metadata_refs` | Must bind materials/products to package subject and authorized step. | `adapt` |
| attestation predicate | `PackageAttestationSet.predicate_ref` | Must declare predicate type, schema, subject, verifier, and policy result. | `adopt` |
| SBOM component | `PackageSBOMRef.component_rows` | Must identify component, version, purl/CPE when available, and evidence source. | `adopt` |
| SBOM vulnerability or license metadata | `PackageSBOMRef.policy_result_rows` | Must map to vulnerability/license gates; must not be trust proof. | `adopt with boundary` |

### Versioning and compatibility mapping

| External concept | Cadastre analogue | Required Cadastre constraint | Transfer stance |
| --- | --- | --- | --- |
| SemVer public API | `PackagePublicAPIProfile` | SemVer may be interpreted only after public API is declared. | `adopt` |
| Terraform version constraint | `PackageDependencyConstraint` | Must constrain dependency identity and allowed versions. | `adopt` |
| Terraform dependency lock | `PackageDependencyLock` | Must record resolved version/checksums/platforms and be immutable for activation. | `adopt` |
| Terraform provider protocol | `PackageCompatibilityMatrix.runtime_protocol` | Must pass independent of version string. | `adopt` |
| Helm chart version | `PackageReleaseManifest.package_version` | Package version axis; not deployment revision. | `adapt` |
| Helm appVersion | `PackageReleaseManifest.application_version_hint` | Informational unless compatibility matrix consumes it. | `adapt` |
| Helm kubeVersion constraint | `PackageCompatibilityMatrix.target_environment_constraint` | Must gate target environment compatibility. | `adapt` |
| Buildpacks Platform API | `PackageCompatibilityMatrix.platform_api_version` | Target environment must support declared API. | `adopt` |
| Buildpacks Buildpack API | `PackageCompatibilityMatrix.package_api_version` | Runtime/lifecycle must support package API. | `adopt` |
| Kubernetes deprecation window | `DeprecatedPackageCompatibilityWindowPolicy.window_rule` | Must define support/removal by package class. | `adapt` |
| OpenTelemetry stability state | `LifecycleStatus` plus package stability metadata | Stability controls activation and deprecation behavior. | `adapt` |

### Promotion and rollback mapping

| External concept | Cadastre analogue | Required Cadastre constraint | Transfer stance |
| --- | --- | --- | --- |
| GitOps desired state | `ProductionPackageSetManifest` | Desired production state must be immutable and checksum-addressed. | `adapt` |
| Git commit ref | `PackageRepositorySnapshot.source_commit_ref` | Must be immutable commit plus artifact digest, not branch. | `adapt` |
| Argo CD sync | `PromotePackageSet` and `ActivatePackageSet` | Must apply only after validation and approval gates. | `adapt` |
| Argo Rollouts canary step | `PackagePromotionRecord.canary_gate_rows` | Must define traffic/sample scope, metric, bounds, and abort behavior. | `adapt` |
| Argo Rollouts analysis abort | `AbortPackageActivation` | Must keep current active set and emit failure event. | `adopt` |
| Flux HelmRelease rollback | `PackageRollbackPlan` | May inform remediation strategy; target must be immutable package-set manifest. | `adapt` |
| Helm release revision | `PackageDeploymentRevision` | Records deployment history; not artifact identity. | `adapt` |
| OPA current active bundle | `LastKnownGoodPackageSet` | Failed candidate keeps current active set. | `adopt` |
| OPA failed activation behavior | `PackageActivationFailureEvent` | Must fail closed with no production output. | `adopt` |
| last-known-good package set | `LastKnownGoodPackageSet` | Must reference immutable package-set checksum and successful activation result. | `adopt` |

## Package Supply-Chain Contract Recommendations

## Contract conventions

Every contract below must use canonical JSON for checksum inputs, UTF-8 NFC strings, UTC RFC3339 timestamps with exactly 9 fractional second digits, lowercase snake_case enums, and SHA-256 lower-hex checksums unless the field explicitly names an OCI digest. Every contract that can affect production output, package activation, rollback, replay, graph apply, validation, visibility, or health gating must be referenced by `VersionManifest` before production output is written.[^3]

## PackageReleaseManifest

**Contract purpose:** Defines one immutable released package and all evidence required before it can enter a package set.

**Authority boundary:** May authorize eligibility for package-set inclusion; must not authorize production activation alone.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| package_release_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_type | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| artifact_refs | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| trust_policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| signature_result_ids | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| attestation_set_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| sbom_ref_ids | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_lock_ids | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| compatibility_matrix_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| validation_matrix_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| release_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageReleaseManifest.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageReleaseManifest` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageReleaseManifest` field fails activation or replay before production output.

## PackageTrustPolicy

**Contract purpose:** Defines trust roots, authorized signers, transparency requirements, repository policy, and attestation authority.

**Authority boundary:** May authorize signature/provenance trust decisions; must not authorize validation, compatibility, or runtime stage output.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| package_trust_policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| policy_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| trust_root_refs | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| authorized_signer_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| threshold_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| transparency_required | boolean | Yes | false | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| repository_scope_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| attestation_authority_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| status | LifecycleStatus | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageTrustPolicy.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageTrustPolicy` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageTrustPolicy` field fails activation or replay before production output.

## PackageRepositoryMetadata

**Contract purpose:** Records verified repository role metadata, delegated ownership, and applicable package namespace scope.

**Authority boundary:** May authorize repository metadata provenance; must not prove package bytes safe without artifact digest verification.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| repository_metadata_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| repository_uri | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| metadata_format | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| role_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| delegation_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| snapshot_ref | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| timestamp_ref | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| metadata_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRepositoryMetadata.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRepositoryMetadata` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRepositoryMetadata` field fails activation or replay before production output.

## PackageRepositorySnapshot

**Contract purpose:** Records one coherent repository metadata snapshot and target metadata set.

**Authority boundary:** May authorize metadata coherence; must not authorize production activation.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| repository_snapshot_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| snapshot_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| snapshot_expires_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_metadata_refs | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| metadata_versions | object | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| snapshot_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| verified_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRepositorySnapshot.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRepositorySnapshot` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRepositorySnapshot` field fails activation or replay before production output.

## PackageRepositoryFreshnessProof

**Contract purpose:** Records timestamp/freshness verification and expiry decision.

**Authority boundary:** May authorize freshness; must not authorize signer trust or compatibility.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| freshness_proof_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| repository_snapshot_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| timestamp_metadata_ref | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| expires_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| verified_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| max_age_seconds | int | Yes | 86400 | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| freshness_status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRepositoryFreshnessProof.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRepositoryFreshnessProof` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRepositoryFreshnessProof` field fails activation or replay before production output.

## PackageSignatureVerificationResult

**Contract purpose:** Records one structured signature verification decision.

**Authority boundary:** May authorize signature gate only when paired with trust policy; must not authorize package correctness.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| signature_verification_result_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| artifact_digest | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| artifact_size_bytes | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| artifact_media_type | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| signature_ref | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| signer_identity | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| oidc_issuer | string | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| trust_policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| transparency_status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| verification_status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| failure_code | enum | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageSignatureVerificationResult.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageSignatureVerificationResult` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageSignatureVerificationResult` field fails activation or replay before production output.

## PackageAttestationSet

**Contract purpose:** Records all required attestations for a package release.

**Authority boundary:** May authorize attestation gate; must not replace validation matrix or compatibility matrix.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| attestation_set_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| subject_digest | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| predicate_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| provenance_id | id | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| in_toto_link_refs | array | Yes | [] | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| verification_result_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| policy_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageAttestationSet.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageAttestationSet` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageAttestationSet` field fails activation or replay before production output.

## PackageBuildProvenance

**Contract purpose:** Records build provenance for the released artifact.

**Authority boundary:** May prove build origin/materials under policy; must not prove runtime correctness.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| build_provenance_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| subject_digest | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| builder_id | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| build_type | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| source_materials | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_materials | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| resolved_dependencies_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| provenance_predicate_type | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| verification_status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageBuildProvenance.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageBuildProvenance` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageBuildProvenance` field fails activation or replay before production output.

## PackageSBOMRef

**Contract purpose:** Records an SBOM document and its subject mapping.

**Authority boundary:** May supply component/license/vulnerability gates; must not be trust proof.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| package_sbom_ref_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| sbom_format | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| sbom_spec_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| sbom_digest | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| subject_digest | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| component_count | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_graph_status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| license_policy_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| vulnerability_policy_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageSBOMRef.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageSBOMRef` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageSBOMRef` field fails activation or replay before production output.

## PackageDependencyConstraint

**Contract purpose:** Defines allowed dependency ranges before resolution.

**Authority boundary:** May constrain dependency solving; must not override lock identity.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| dependency_constraint_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_name | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_source | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| allowed_versions | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| excluded_versions | array | Yes | [] | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| reason | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageDependencyConstraint.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageDependencyConstraint` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageDependencyConstraint` field fails activation or replay before production output.

## PackageDependencyLock

**Contract purpose:** Defines resolved dependency identities and checksums.

**Authority boundary:** May authorize reproducible dependency resolution; must not authorize trust, compatibility, or safety by itself.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| dependency_lock_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| lock_format_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| platform_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| resolved_digest_set_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| lock_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageDependencyLock.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageDependencyLock` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageDependencyLock` field fails activation or replay before production output.

## PackageCompatibilityMatrix

**Contract purpose:** Defines package eligibility against target runtime, protocol, schema, graph, source, dependency, and contract axes.

**Authority boundary:** May authorize compatibility gate; must not authorize trust or validation.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| compatibility_matrix_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_type | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_public_api_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| contract_version | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| runtime_protocol_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_environment_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| dependency_lock_checksum | sha256_hex | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| source_schema_checksum | sha256_hex | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| validation_output_checksum | sha256_hex | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| decision | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageCompatibilityMatrix.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageCompatibilityMatrix` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageCompatibilityMatrix` field fails activation or replay before production output.

## ProductionPackageSetManifest

**Contract purpose:** Defines immutable production activation target set.

**Authority boundary:** May authorize activation target when all gates pass; must not permit partial activation unless explicitly declared.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| package_set_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| environment_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| release_manifest_refs | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| cohesion_group_refs | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_set_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| rollback_target_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| approval_policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| lkg_candidate | boolean | Yes | false | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `ProductionPackageSetManifest.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `ProductionPackageSetManifest` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `ProductionPackageSetManifest` field fails activation or replay before production output.

## PackagePromotionRecord

**Contract purpose:** Records promotion request, gates, approvals, and decision.

**Authority boundary:** May authorize transition to canary or active when every referenced gate passes.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| promotion_record_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| candidate_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_environment_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| requested_by | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| approval_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| shadow_gate_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| canary_gate_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| validation_gate_result | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| promotion_decision | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackagePromotionRecord.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackagePromotionRecord` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackagePromotionRecord` field fails activation or replay before production output.

## PackageDeploymentRevision

**Contract purpose:** Records environment-visible deployment revision.

**Authority boundary:** May record history; must not be package identity or rollback target alone.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| deployment_revision_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| environment_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_set_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| revision_number | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| activated_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| activation_result_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageDeploymentRevision.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageDeploymentRevision` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageDeploymentRevision` field fails activation or replay before production output.

## LastKnownGoodPackageSet

**Contract purpose:** Records most recent package set that passed activation and health gates.

**Authority boundary:** May be rollback target only when still compatible and not quarantined.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| last_known_good_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| environment_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_set_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| activation_result_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| health_gate_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| recorded_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| valid_for_rollback_until | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `LastKnownGoodPackageSet.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `LastKnownGoodPackageSet` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `LastKnownGoodPackageSet` field fails activation or replay before production output.

## PackageRollbackPlan

**Contract purpose:** Defines proposed rollback from current set to immutable target.

**Authority boundary:** May authorize rollback only after compatibility/replay/quarantine gates pass.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| rollback_plan_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| current_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| reason | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| compatibility_check_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| replay_check_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| state_migration_required | boolean | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| approval_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRollbackPlan.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRollbackPlan` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRollbackPlan` field fails activation or replay before production output.

## PackageRollbackResult

**Contract purpose:** Records executed rollback outcome.

**Authority boundary:** May update active package set only on success.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| rollback_result_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| rollback_plan_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| started_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| completed_at | timestamp | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| failure_code | enum | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| active_manifest_after | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| production_writes_blocked | boolean | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRollbackResult.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRollbackResult` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRollbackResult` field fails activation or replay before production output.

## EmergencyPackageOverrideRecord

**Contract purpose:** Records break-glass action.

**Authority boundary:** May quarantine, retire, abort, or rollback; must not bypass signature/trust for new production activation.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| emergency_override_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| override_type | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_ref | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| requested_by | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| quorum_approval_rows | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| ttl_expires_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| forbidden_bypass_assertion | boolean | Yes | true | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| audit_reason | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `EmergencyPackageOverrideRecord.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `EmergencyPackageOverrideRecord` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `EmergencyPackageOverrideRecord` field fails activation or replay before production output.

## DeprecatedPackageCompatibilityWindowPolicy

**Contract purpose:** Defines package-type-specific deprecation and rollback windows.

**Authority boundary:** May allow deprecated use for replay/rollback/migration only.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| deprecation_policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| package_type | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| default_window_days | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| max_window_days | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| allowed_use_classes | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| retirement_action | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| waiver_required_after | timestamp | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `DeprecatedPackageCompatibilityWindowPolicy.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `DeprecatedPackageCompatibilityWindowPolicy` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `DeprecatedPackageCompatibilityWindowPolicy` field fails activation or replay before production output.

## PackageCohesionGroup

**Contract purpose:** Defines packages that must promote/rollback together.

**Authority boundary:** May require atomic package-set inclusion; must not hide individual package evidence failures.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| cohesion_group_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| group_kind | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| member_package_ids | array | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| cohesion_rule | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| minimum_member_count | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| compatibility_matrix_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageCohesionGroup.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageCohesionGroup` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageCohesionGroup` field fails activation or replay before production output.

## PackageActivationFailureEvent

**Contract purpose:** Records failed candidate activation.

**Authority boundary:** May trigger abort and health state; must not mutate active set.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| activation_failure_event_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| candidate_manifest_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| failure_precedence | int | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| failure_class | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| failure_code | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| emitted_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| current_manifest_preserved | boolean | Yes | true | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageActivationFailureEvent.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageActivationFailureEvent` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageActivationFailureEvent` field fails activation or replay before production output.

## PackageQuarantineRecord

**Contract purpose:** Blocks package/release/set use.

**Authority boundary:** May block activation, canary, rollback, or replay as configured.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| quarantine_record_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_kind | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| reason | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| scope | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| created_by | string | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| expires_at | timestamp | No | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| status | enum | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageQuarantineRecord.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageQuarantineRecord` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageQuarantineRecord` field fails activation or replay before production output.

## PackageRepositoryAntiRollbackState

**Contract purpose:** Records highest-seen repository metadata and target versions.

**Authority boundary:** May reject older metadata or target versions; must not block explicit rollback to verified immutable package set unless policy says so.

| Required field | Type | Required | Default | Bounds | Validation |
| --- | --- | --- | --- | --- | --- |
| anti_rollback_state_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| repository_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| role_version_map | object | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| target_version_map | object | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| last_snapshot_checksum | sha256_hex | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| updated_at | timestamp | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |
| policy_id | id | Yes | none | contract-defined | Must be present, canonicalized, and included in checksum when required. |

**Derived fields:** `PackageRepositoryAntiRollbackState.checksum` must be SHA-256 over canonical JSON excluding mutable audit timestamps unless a timestamp is explicitly part of the contract identity. Any scalar status summary must be derived from the structured rows above.

**State transitions:** Valid status-bearing contracts use `draft -> validated -> canary -> active -> deprecated -> retired` unless the contract declares itself immutable evidence. Immutable evidence contracts have only `recorded`, `superseded`, and `quarantined` states.

**Failure behavior:** Missing required field, checksum mismatch, expired policy, unresolved reference, or incompatible lifecycle status must reject activation with the most specific package error code; if no specific code exists, use `PACKAGE_SUPPLY_CHAIN_ERROR`.

**VersionManifest inclusion:** `PackageRepositoryAntiRollbackState` IDs and checksums must appear in `VersionManifest` whenever the contract can affect activation, validation, production output, rollback, replay, graph apply, visibility, or health gating.[^3]

**Replay behavior:** Replay must recompute the checksum and reject mismatch before production output. Mutable external references must be resolved through immutable digests or snapshots.

**Acceptance criteria:** A fixture that removes, mutates, expires, or checksum-mismatches any required `PackageRepositoryAntiRollbackState` field fails activation or replay before production output.

## Package Versioning and Compatibility Recommendations

SemVer must not be interpreted until the package type’s public API is defined. Production eligibility must be determined by `PackageCompatibilityMatrix`, not by package version string alone. The matrix must treat these axes as separate when they affect behavior:

```text
package version
artifact digest
contract version
runtime protocol version
dependency lock checksum
source schema checksum
validation output checksum
compatibility target
source stage binding
package public API version
graph projection/read/apply profile version
external schema profile version
trust policy version
attestation policy version
SBOM policy version
```

## Package-type-specific versioning table

| Package type | Public API for versioning | Major-version trigger | Minor-version trigger | Patch-version trigger | Rollback constraint |
| --- | --- | --- | --- | --- | --- |
| adapter | Source datasets, call patterns, emitted raw/completeness/error records | Changes output permissions, source state semantics, source dataset identity, or completeness effects | Adds non-breaking source dataset or optional config with default | Bug fix with byte-equivalent output for existing fixtures | Rollback requires cursor/state compatibility and raw replay safety |
| parser | Raw input schema to parse result mapping | Changes parse output shape, omission mapping, or error classes | Adds supported raw record subtype or optional parsed field | Fixes parser bug without schema change | Rollback requires raw fixture replay and no loss of parser error semantics |
| mapping_bundle | Silver/gold mapping rules and output schema refs | Changes emitted observation/fact semantics or required fields | Adds optional mapping or source extension field | Fixes deterministic mapping bug | Rollback requires validation output checksum compatibility |
| source_schema_import_profile | Source schema import behavior and unsupported construct mapping | Changes canonical imported schema model | Adds supported source schema construct | Fixes diagnostics only | Rollback requires source schema checksum compatibility |
| semantic_overlay_artifact | Non-authoritative overlay symbol set | Renames/removes active symbols used by mappings | Adds symbols or annotations | Fixes metadata only | Rollback requires mapping bundle overlay checksum match |
| mapping_validation_rule_set | Rule IDs, severities, diagnostics | Changes required/forbidden rule or severity affecting activation | Adds warning or optional rule | Fixes diagnostic wording without code change | Rollback requires canonical validation output schema match |
| mapping_project_manifest | Source roots, compiler config, dependency refs | Changes output-affecting config surface | Adds optional non-output-affecting metadata | Fixes manifest metadata | Rollback requires dependency lock and compiler version match |
| mapping_compiler_pipeline | Compiler phases and normalized validation output | Changes phase ordering, output schema, or diagnostics codes | Adds phase that emits non-breaking diagnostics | Fixes deterministic compiler bug | Rollback requires golden corpus and validation output compatibility |
| canonical_validation_output_schema | Validation output fields and checksum inputs | Removes/renames fields or changes checksum rules | Adds optional field excluded from checksum by default | Clarifies schema without behavior change | Rollback requires stored output readable by target schema |
| validation_scenario | Given/input/expected output assertions | Changes expected production result or rejection | Adds new scenario | Fixes expected diagnostic text only | Rollback requires scenario checksum match or waiver |
| toolchain_dependency_review | External tool approval boundary | Changes approval decision or required evidence | Adds reviewed capability or platform | Extends expiry or corrects metadata | Rollback requires non-expired approval |
| external_tool_capability_evidence | Executable capability proof | Changes capability semantics or artifact subject | Adds new capability proof | Corrects evidence metadata | Rollback requires exact artifact/capability match |
| mapping_toolchain_package | Compiler/linter/generator runtime API | Changes generated artifacts or validation result | Adds non-breaking command or output | Fixes tool bug | Rollback requires dependency lock and validation replay |
| external_schema_profile | External schema artifact and class mapping | Changes class/field/enum semantics | Adds class/profile/extension row | Fixes metadata or allowed enum description | Rollback requires compiled artifact and mapping compatibility |
| source_call_policy | Pagination/retry/rate-limit/timeout behavior | Changes cursor commit or visibility-loss behavior | Adds optional policy row | Adjusts non-output diagnostics | Rollback requires stage state compatibility |
| source_completeness_profile | Absence/retraction/cleanup/watermark authority | Changes what states authorize absence or cleanup | Adds dataset/scope coverage row | Fixes diagnostic text | Rollback must not invalidate emitted absence/retraction facts |
| declared_dag_subset_profile | Safe subset execution behavior | Changes permitted outputs or cleanup/watermark rules | Adds safe subset | Fixes metadata | Rollback requires no forbidden outputs in prior run |
| resolver_profile | Identity evidence, blockers, decisions | Changes merge/split/blocker behavior | Adds source scope or review route | Fixes explanation metadata | Rollback requires graph correction handoff compatibility |
| analysis_rule_bundle | Read-only analysis rule API | Changes rule query result semantics | Adds rule disabled by default or non-breaking rule | Fixes rule metadata | Rollback requires graph compatibility matrix |
| derivation_rule_bundle | Gold derivation rule API | Changes emitted GoldFact semantics | Adds new derivation fact type | Fixes deterministic bug | Rollback requires fact correction plan |
| policy_bundle | Operational/promotion/security policy rules | Changes authorization decision | Adds policy row with default deny | Fixes diagnostics | Rollback requires active trust/approval policy compatibility |
| target_selector_safety_policy | Selector max resolution and evidence requirements | Changes selector authority or allowed state | Adds stricter selector row | Fixes text | Rollback requires unresolved target replay compatibility |
| graph_edge_semantics | Edge meaning, direction, traversal class | Changes edge semantics or pathfinding eligibility | Adds edge type disabled by default | Fixes metadata | Rollback requires graph delta retraction plan |
| graph_taxonomy_translation_policy | External taxonomy label mapping | Changes label meaning or allowed output | Adds non-authoritative label mapping | Fixes display text | Rollback requires graph property compatibility |
| graph_projection_profile | Gold-to-graph delta mapping | Changes emitted node/edge/property semantics | Adds optional projection row | Fixes deterministic no-op | Rollback requires graph apply rollback/rebuild plan |
| graph_read_model_schema_profile | Backend constraints/indexes/schema | Changes required selectors or indexes | Adds optional index | Fixes schema metadata | Rollback requires schema preflight and rebuild compatibility |
| graph_apply_profile | Apply ordering/batching/idempotency | Changes transaction boundary or retry semantics | Adds optional batch mode | Fixes diagnostics | Rollback requires idempotent reapply and previous GraphApplyResult |
| rule_graph_compatibility_matrix | Rule-to-graph profile compatibility | Changes pass/fail eligibility | Adds rule/profile case | Fixes expected hash metadata | Rollback requires rule output checksum match |
| structural_global_node_alias_policy | Global node/alias behavior | Enables or changes alias semantics | Adds disabled alias candidate | Fixes metadata | Rollback requires graph no-op or retraction |
| CIM_projection_profile | Splunk CIM projection mapping | Changes projected CIM fields or loss semantics | Adds optional CIM output | Fixes projection metadata | Rollback affects CIM only unless deployment requires CIM |
| projection_loss_policy | Loss accounting and budgets | Changes allowed loss or failure threshold | Adds budget row | Fixes diagnostics | Rollback requires projection loss manifest compatibility |
| deployment_profile | Environment targets, gates, policy refs | Changes production eligibility or required outputs | Adds environment optional capability | Fixes metadata | Rollback requires target environment compatibility |

## Promotion, Canary, Activation, and Rollback Recommendations

`PromotePackageSet` must create or select a `ProductionPackageSetManifest` and write a `PackagePromotionRecord`. `ActivatePackageSet` must be the only algorithm that changes the active package set for an environment. `RollbackPackageSet` must target a prior immutable `ProductionPackageSetManifest`, not individual packages, tags, branches, or rebuilt artifacts.

## Deterministic package activation failure precedence

| Precedence | Failure class | Required result |
| --- | --- | --- |
| 1 | Artifact digest, size, media type, or subject mismatch | Reject candidate. |
| 2 | Signature, trust-root, signer, transparency, or repository metadata failure | Reject candidate. |
| 3 | Repository freshness, expiration, freeze, mix-and-match, or anti-rollback failure | Reject candidate. |
| 4 | Quarantine or emergency block | Reject candidate. |
| 5 | Missing developer contract or stage binding | Reject candidate. |
| 6 | Dependency lock mismatch, unresolved dependency, or live resolution attempt | Reject candidate. |
| 7 | Attestation subject, builder, buildType, material, or predicate-policy failure | Reject candidate. |
| 8 | SBOM missing, subject mismatch, disallowed license, vulnerability gate failure, or opaque dependency graph when completeness required | Reject candidate. |
| 9 | Compatibility matrix failure | Reject candidate. |
| 10 | Validation matrix or required negative test failure | Reject candidate. |
| 11 | Golden corpus or replay failure | Reject candidate. |
| 12 | Shadow or canary failure | Reject candidate or abort canary. |
| 13 | Missing rollback target, invalid last-known-good target, or rollback plan incompatibility | Reject candidate. |
| 14 | Missing, expired, or insufficient approval | Reject candidate. |
| 15 | VersionManifest mismatch or missing required package-set references | Reject candidate before output. |

## Production activation rules

```text
Activation must be idempotent for the same candidate_manifest_id and environment_id.
Activation must not write production output until all failure-precedence checks pass.
Activation must set the candidate as active only after deployment revision, VersionManifest refs, and LastKnownGood candidate records are persisted atomically or through a declared two-phase activation record.
Activation failure must preserve the current active package set and emit PackageActivationFailureEvent.
```

## Emergency Override and Deprecation Recommendations

Emergency override must be constrained to one of these types:

| Override type | Allowed target | Allowed effect | Forbidden effect | Default TTL |
| --- | --- | --- | --- | --- |
| `quarantine_package` | Package artifact, release manifest, package set | Blocks new activation and optionally blocks rollback/replay. | Must not modify artifact bytes or trust result. | 72 hours |
| `retire_active_package` | Active package release | Moves lifecycle toward retired and blocks new writes after safe handoff. | Must not activate unverified replacement. | 24 hours |
| `abort_candidate_activation` | Candidate package-set manifest | Stops candidate activation and preserves current set. | Must not mark candidate successful. | 24 hours |
| `rollback_to_lkg` | LastKnownGoodPackageSet | Activates prior verified immutable manifest after rollback gates. | Must not resolve dependencies or rebuild from tip. | 24 hours |
| `extend_deprecation_window` | Deprecated package class or release | Permits replay/rollback/migration use only. | Must not permit new production activation without compatibility gates. | 30 days max default |

Emergency signature bypass for a new package activation is forbidden. A break-glass action may retire, quarantine, abort, or rollback only.

## Concepts That Must Not Transfer

| Source pattern | Why unsafe for Cadastre | Required Cadastre-safe alternative |
| --- | --- | --- |
| OCI tag as package identity | Tags are mutable. | Immutable digest identity. |
| Git branch as production package identity | Branch heads move. | Immutable commit plus artifact digest. |
| Signature existence as verification | Signature may not prove authorized signer or trust root. | Structured verification result plus trust policy. |
| Provenance existence as validation | Provenance describes build origin, not correctness. | Provenance plus validation matrix and policy. |
| Dependency lock as trust | Lock proves reproducibility, not safety. | Lock plus trust, provenance, SBOM, and validation. |
| Helm rollback as Cadastre rollback | Helm rollback lacks Cadastre replay, state, graph, and completeness checks. | `PackageRollbackPlan` and `PackageRollbackResult`. |
| OPA bundle success as general package success | OPA bundle scope is narrower. | Package-type-specific validation and compatibility gates. |
| GitOps drift correction as Cadastre fact correction | Deployment drift is not source truth. | Operational health only unless Cadastre contracts authorize changes. |
| SemVer without public API | Version string has no binding semantics. | Package public API plus compatibility matrix. |
| Canary success as universal correctness | Canary does not prove historical replay or all edge cases. | Canary plus replay, golden corpus, and validation. |
| Emergency signature bypass | Bypasses core integrity guarantees. | Emergency retirement, quarantine, or rollback to previous verified manifest only. |
| SBOM as safety approval | Inventory does not prove trust, correctness, or compatibility. | SBOM plus policy gates, trust, provenance, and validation. |
| Transparency log entry as authorization | Transparency proves logging, not authorization. | Transparency plus trust policy signer authorization. |
| Repository freshness as package compatibility | Fresh metadata does not prove runtime compatibility. | Freshness plus compatibility matrix and validation matrix. |

## PRD Improvement Map

| Finding | Source basis | Current PRD area | Proposed PRD change | Strengthens | Corrects | Constrains | Complicates | Priority |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Scalar `signature_status` is insufficient | TUF, Sigstore, Notation, Helm, OPA | `PackageArtifact` | Add `PackageSignatureVerificationResult` and derive `signature_status` from it. | Package integrity | Ambiguous verification | Activation gates | More evidence storage | P0 / tightening + new contract |
| Package repository trust is absent | TUF, Notation | `PackageArtifact`, `VersionManifest` | Add repository metadata, snapshot, freshness proof, anti-rollback state, trust policy refs. | Repository trust | Mutable/unfresh metadata | Repository use | More repository metadata | P0 / new contract |
| Package release is not set-based | GitOps, Helm/Flux, PRD package types | Extension Package Lifecycle | Add `PackageReleaseManifest`, `ProductionPackageSetManifest`, `PackageCohesionGroup`. | Activation determinism | Single-package rollback risk | Partial activation | Set management | P0 / new contract |
| Build provenance is not first-class | SLSA, in-toto | `ToolchainDependencyReview`, `ExternalToolCapabilityEvidence` | Add `PackageBuildProvenance` and `PackageAttestationSet`. | Build evidence | Digest-only assurance | Builder policy | Attestation verification | P0 / additive |
| SBOM is not first-class package evidence | SPDX, CycloneDX | `ToolchainDependencyReview`, `ValidationMatrix` | Add `PackageSBOMRef` with policy gate fields. | Inventory, license, vulnerability gates | Unknown dependencies | Promotion | SBOM validation | P1 / additive |
| Locks are conflated with trust risk | Terraform | `dependency_lock_id`, `resolved_dependency_digests` | Define locks as reproducibility evidence only; require trust/provenance/SBOM separately. | Replay | Lock-as-trust | Dependency resolution | More gates | P0 / tightening |
| Runtime protocol compatibility is not explicit enough | Terraform, Buildpacks | `PackageStageBinding`, `SourcePackageDeveloperContract` | Add `PackageCompatibilityMatrix` runtime protocol and target environment rows. | Interoperability | Version-only eligibility | Activation | Matrix maintenance | P0 / additive |
| Canary/shadow gates need package-set records | Argo Rollouts, Argo CD, Flux | `LifecycleStatus`, `LifecycleStateMachineDefinition` | Add promotion record gates with shadow/canary result checksums. | Safe rollout | Implicit gates | Promotion | Metrics definition | P1 / additive |
| Activation failure must keep current | OPA bundles | `LifecycleStateMachineDefinition` | Add `PackageActivationFailureEvent` and keep-current default. | Fail-safe behavior | Candidate side effects | Activation | Eventing | P0 / tightening |
| Rollback target identity underspecified | Helm, Flux, OCI, TUF | Extension Package Lifecycle | Rollback target must be immutable package-set manifest checksum plus release checksums. | Rollback safety | Mutable rollback | Rollback by tag | Rollback plan burden | P0 / semantic change |
| Emergency override too broad | TUF, Sigstore, OPA | `LifecycleStatus` | Add `EmergencyPackageOverrideRecord`; forbid new-candidate signature bypass. | Trust boundary | Break-glass ambiguity | Override | Quorum/TTL governance | P0 / tightening |
| Deprecation not package-class-specific | Kubernetes, OpenTelemetry | `LifecycleStatus` | Add `DeprecatedPackageCompatibilityWindowPolicy` by package type. | Compatibility windows | Uniform deprecation | Deprecated use | Policy table | P1 / additive |
| Package version API not defined | SemVer | Extension package lifecycle package types | Add package-type public API rows and versioning triggers table. | Version correctness | Meaningless SemVer | Activation by version | More spec text | P0 / tightening |
| `VersionManifest` package refs need supply-chain expansion | PRD, OCI, TUF, SLSA, SPDX | `VersionManifest` | Add package release/set/trust/signature/attestation/SBOM/freshness/anti-rollback refs. | Replay | Incomplete replay | Manifest size | More refs | P0 / additive |
| `ValidationMatrix` needs package supply-chain tests | All priority sources | `ValidationMatrix` | Add negative tests for signatures, trust, locks, provenance, SBOM, compatibility, rollback, emergency override. | Binary gating | Untested lifecycle | Activation | Fixture burden | P0 / acceptance-criteria addition |
| `PackageStageBinding` must consume compatibility matrix | Terraform, Buildpacks | `PackageStageBinding` | Require active package-stage binding to name compatible package types, output classes, runtime protocols, and contract versions. | Stage safety | Protocol mismatch | Stage execution | Matrix maintenance | P0 / tightening |
| `SourcePackageDeveloperContract` must bind to package release evidence | PRD, RES-006 | `SourcePackageDeveloperContract` | Require contract checksum and release manifest inclusion before stage execution. | Adapter boundary | Unbound package execution | Stage output | More refs | P0 / tightening |
| `MappingProjectManifest` and compiler packages need supply-chain evidence | Taxi, SLSA, Terraform | `MappingProjectManifest`, `MappingCompilerPipeline` | Require release manifest, dependency lock, attestation, and validation output checksum. | Mapping determinism | Tooling drift | Promotion | Toolchain evidence | P1 / tightening |
| `CanonicalValidationOutput` must be release-gated | Taxi, PRD | `CanonicalValidationOutput` | Include schema version and output checksum in package release and package set. | Replay | Validation drift | Activation | Checksum management | P1 / additive |
| `StageStateRecord` rollback interaction underspecified | RES-006, Terraform, OPA | `StageStateRecord` | Rollback plan must check cursor/state schema compatibility and state migration need. | State safety | Unsafe rollback | Rollback | State validation | P1 / additive |
| `GraphApplyProfile` and `GraphApplyResult` need package-set coupling | PRD, RES-012 | `GraphApplyProfile`, `GraphApplyResult` | Graph apply package/profile versions must be part of package set compatibility and rollback plan. | Graph safety | Profile drift | Graph mutation | Graph plan checks | P1 / additive |
| `DerivedViewLagPolicy`, `ReplayRetentionPolicy`, `TableMaintenancePolicy` interact with rollback | RES-005, RES-011 | Named policies | Rollback plan must verify replay retention and derived-view lag policy before activation. | Replay-safe rollback | Retention drift | Rollback | More preflight checks | P1 / additive |

## Proposed Deterministic Algorithms

## Shared algorithm rules

All algorithms below must use canonical JSON serialization for checksum inputs, lexically sorted arrays unless order is semantically declared, exact byte digests for artifact checks, and the failure precedence table in this report. Every algorithm must be idempotent for the same inputs. Any retry must re-read immutable records by ID/checksum and must not re-resolve mutable tags, branches, or dependency ranges.

## VerifyPackageArtifact(candidate_artifact_ref, trust_policy)

| Property | Requirement |
| --- | --- |
| Inputs | candidate_artifact_ref, trust_policy |
| Outputs | artifact_verification_result |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes verification result only; does not write package activation state. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load artifact ref by immutable digest.
2. Fetch bytes only from allowed repository or local materialization root.
3. Verify byte length equals declared size.
4. Verify digest equals declared digest.
5. Verify mediaType is allowed for package type.
6. Verify subject digest if artifact is an evidence referrer.
7. Call VerifyPackageSignature and repository freshness checks.
8. Return structured result.
```

## VerifyPackageSignature(package_artifact, trust_policy)

| Property | Requirement |
| --- | --- |
| Inputs | package_artifact, trust_policy |
| Outputs | PackageSignatureVerificationResult |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes signature verification result; does not update artifact status except through derived field. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load signatures or signature bundle refs.
2. Reject if required signature absent.
3. Verify cryptographic signature over exact subject digest.
4. Verify signer identity and OIDC issuer against authorized signer rows.
5. Verify trust root and certificate chain or key.
6. Verify threshold rule if multiple signatures required.
7. Verify transparency proof when required.
8. Return pass/fail with specific code.
```

## VerifyPackageAttestations(package_artifact, attestation_policy)

| Property | Requirement |
| --- | --- |
| Inputs | package_artifact, attestation_policy |
| Outputs | PackageAttestationSet verification decision |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes attestation verification rows only. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load required attestation predicates.
2. Verify each attestation signature or envelope.
3. Verify subject digest equals package artifact digest.
4. Verify builder.id, buildType, materials, dependencies, and predicate type against policy.
5. Verify in-toto layout/link thresholds when configured.
6. Return attestation policy result.
```

## ResolvePackageDependencies(package_release_manifest)

| Property | Requirement |
| --- | --- |
| Inputs | package_release_manifest |
| Outputs | resolved dependency set |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes no production state; may write validation diagnostic. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load dependency constraints and locks.
2. Reject live resolution for production activation.
3. Verify every dependency row has resolved digest and checksum.
4. Verify platform target rows required for target environment.
5. Verify lock checksum.
6. Return sorted dependency digest set.
```

## ValidatePackageCompatibility(package_release_manifest, target_environment)

| Property | Requirement |
| --- | --- |
| Inputs | package_release_manifest, target_environment |
| Outputs | compatibility decision |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes compatibility check result only. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load PackageCompatibilityMatrix.
2. Verify package type, public API version, contract version, runtime protocol version, dependency lock checksum, source schema checksum, validation output checksum, and target environment row.
3. Reject any matrix row with decision not equal pass.
4. Return compatibility result.
```

## BuildProductionPackageSet(candidate_release_manifests)

| Property | Requirement |
| --- | --- |
| Inputs | candidate_release_manifests |
| Outputs | ProductionPackageSetManifest |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes package set manifest if all inputs pass; no active state mutation. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Sort candidate release manifests by package_type then package_id.
2. Verify every release manifest checksum.
3. Verify cohesion groups contain required members.
4. Reject duplicate package type when group policy forbids duplicates.
5. Compute aggregate compatibility.
6. Compute package-set checksum.
7. Write immutable package set manifest.
```

## PromotePackageSet(candidate_manifest_id, target_environment)

| Property | Requirement |
| --- | --- |
| Inputs | candidate_manifest_id, target_environment |
| Outputs | PackagePromotionRecord |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes promotion record; does not activate by itself unless ActivatePackageSet is called after pass. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load candidate manifest by ID and checksum.
2. Verify target environment deployment profile.
3. Run validation, shadow, and canary gates as required by lifecycle status.
4. Verify approval quorum and non-expired approvals.
5. Write promotion record with decision.
```

## ActivatePackageSet(candidate_manifest_id)

| Property | Requirement |
| --- | --- |
| Inputs | candidate_manifest_id |
| Outputs | activation result or PackageActivationFailureEvent |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes activation result only after all checks pass; failure writes event and preserves active set. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. See detailed algorithm below.
```

## AbortPackageActivation(candidate_manifest_id, failure)

| Property | Requirement |
| --- | --- |
| Inputs | candidate_manifest_id, failure |
| Outputs | PackageActivationFailureEvent |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes failure event and abort state only. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load candidate and current active manifest.
2. Classify failure by precedence.
3. Persist failure event.
4. Mark candidate activation attempt aborted.
5. Verify current active manifest remains active.
```

## RollbackPackageSet(current_manifest_id, target_manifest_id)

| Property | Requirement |
| --- | --- |
| Inputs | current_manifest_id, target_manifest_id |
| Outputs | PackageRollbackResult |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes rollback plan/result and active manifest only on success. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load current and target manifests by checksum.
2. Verify target is prior verified active/LKG or explicitly approved deprecated active-compatible set.
3. Verify no quarantine blocks target.
4. Verify compatibility, dependency locks, schema/state compatibility, replay retention, and graph rollback/rebuild plan.
5. Require approvals.
6. Activate target through same activation checks with rollback context.
```

## ApplyEmergencyPackageOverride(override_record)

| Property | Requirement |
| --- | --- |
| Inputs | override_record |
| Outputs | override result |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes emergency override record and limited target state. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Verify override type is allowed.
2. Verify quorum, TTL, target, and audit reason.
3. Reject forbidden bypass types, especially new-candidate signature bypass.
4. Apply only quarantine, retire, abort, rollback-to-LKG, or deprecation-window extension.
5. Persist audit record.
```

## EvaluateDeprecatedPackageUse(package_id, requested_use)

| Property | Requirement |
| --- | --- |
| Inputs | package_id, requested_use |
| Outputs | deprecation decision |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | Writes no state unless waiver use is audited. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Load package lifecycle status and deprecation policy.
2. Verify requested use class is replay, rollback, migration, or historical read-only exception.
3. Verify compatibility window and waiver.
4. Reject production new activation by default.
```

## ComputePackageReleaseChecksum(package_release_manifest)

| Property | Requirement |
| --- | --- |
| Inputs | package_release_manifest |
| Outputs | sha256_hex |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | No side effects. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Remove derived checksum field.
2. Canonicalize JSON.
3. Sort arrays by declared rules.
4. Hash UTF-8 bytes with SHA-256.
```

## ComputePackageSetManifestChecksum(package_set_manifest)

| Property | Requirement |
| --- | --- |
| Inputs | package_set_manifest |
| Outputs | sha256_hex |
| Preconditions | Inputs exist by immutable ID/checksum; caller holds required run lock when production state may be written. |
| Failure precedence | Use package activation failure precedence table; most specific code wins. |
| Idempotency | Same inputs and same persisted state produce byte-identical output or same failure code. |
| Side effects | No side effects. |
| State written | Only the state named in side effects. |
| State not written | Raw, silver, identity, gold, graph, CIM, source completeness, and production output state unless algorithm explicitly says otherwise. |
| VersionManifest references | Every output-affecting input and result must be included before production output. |
| Replay behavior | Replay recomputes checksums and rejects mismatch before output. |

```text
1. Remove derived checksum field.
2. Canonicalize release manifest refs by release checksum lexical order.
3. Canonicalize cohesion group refs and policy refs.
4. Hash UTF-8 bytes with SHA-256.
```

## Detailed ActivatePackageSet(candidate_manifest_id)

| Property | Requirement |
| --- | --- |
| Inputs | `candidate_manifest_id`, implicit `target_environment_id` from manifest, current active package set for environment. |
| Outputs | `PackageDeploymentRevision` and activation result on success; `PackageActivationFailureEvent` on failure. |
| Preconditions | Candidate manifest exists, is immutable, and caller holds package activation lock for the environment. |
| Failure precedence | Use the activation failure precedence table exactly. The first failing class wins. |
| Idempotency | Re-running activation for an already active manifest returns the existing deployment revision when all referenced checksums match. |
| Side effects | Success writes active package-set state and deployment revision. Failure writes only activation failure event and abort marker. |
| State written | `PackageDeploymentRevision`, `LastKnownGoodPackageSet` candidate when health gate passes, activation audit record, `VersionManifest` refs. |
| State not written | Candidate raw/silver/identity/gold/graph/CIM production outputs before successful activation. |
| VersionManifest references | Must include candidate package set checksum, release checksums, artifact digests, trust policy, signature results, repository metadata/freshness/anti-rollback, attestation set, SBOM refs, locks, compatibility matrix, validation matrix, developer contracts, stage bindings, promotion record, rollback target, quarantine checks, and approvals. |
| Replay behavior | Replay must recompute every checksum and reject mismatches before output. |

```text
1. Load candidate ProductionPackageSetManifest by candidate_manifest_id.
2. Verify package_set_manifest_checksum.
3. Load each PackageReleaseManifest and verify release_checksum.
4. For each artifact ref, verify artifact digest.
5. For each artifact ref, verify artifact size.
6. For each artifact ref, verify artifact media type against package-type allowlist.
7. Verify signature verification result exists, matches artifact digest, and has verification_status = passed.
8. Verify PackageTrustPolicy authorizes signer identity, trust root, threshold, repository scope, and transparency requirement.
9. Verify PackageRepositoryFreshnessProof is not expired and matches PackageRepositorySnapshot.
10. Verify PackageRepositoryAntiRollbackState permits repository metadata versions and target versions.
11. Verify every attestation subject digest equals the artifact digest or declared release subject digest.
12. Verify SBOM refs exist when policy requires them, match subject digest, and pass license/vulnerability/dependency-graph gates.
13. Verify PackageDependencyLock checksum and resolved dependency digests for every package whose dependencies affect output or validation.
14. Verify SourcePackageDeveloperContract exists, is active, and checksum matches for every stage-executing package.
15. Verify PackageStageBinding exists, is active, and permits package type, package version, stage, and output classes.
16. Verify PackageCompatibilityMatrix passes for package type, public API, contract version, runtime protocol, dependency lock, source schema, validation output, graph profile, and target environment.
17. Verify ValidationMatrix exists and every required positive, negative, forbidden-output, rollback, replay, and supply-chain test passes.
18. Verify golden corpus result checksums.
19. Verify production replay result checksums for required replay scenarios.
20. Verify shadow execution result checksums when required by promotion policy.
21. Verify canary execution metrics and failure bounds when required by promotion policy.
22. Verify rollback_target_manifest_id exists, is immutable, and is not quarantined.
23. Verify no PackageQuarantineRecord or EmergencyPackageOverrideRecord blocks candidate, any release, any artifact, any signer, or any trust root.
24. Verify approval quorum, roles, expiry, and environment scope.
25. Acquire or verify active package-set activation lock.
26. Persist activation VersionManifest references.
27. Persist PackageDeploymentRevision with status = activating.
28. Atomically set environment active package set to candidate manifest, or use declared two-phase activation record with old and new manifest IDs.
29. Mark deployment revision status = active.
30. Record LastKnownGoodPackageSet only after required post-activation health gates pass.
31. Return success.

Failure rule:
- On any failed step, call AbortPackageActivation(candidate_manifest_id, failure).
- Current active package set remains active.
- Candidate writes no production output.
- PackageActivationFailureEvent.current_manifest_preserved must be true.
```

## Binary Acceptance Criteria

| ID | Pass/fail criterion |
| --- | --- |
| PKG-SIG-001 | A package artifact whose bytes hash to a digest different from `PackageArtifactRef.digest` fails activation with `artifact_digest_mismatch` before signature verification. |
| PKG-SIG-002 | A package with a cryptographically valid signature from a signer not authorized by `PackageTrustPolicy` fails activation with `unauthorized_signer`. |
| PKG-SIG-003 | A package whose trust policy requires transparency evidence fails activation with `transparency_evidence_missing` when no valid Rekor inclusion proof, SET, or approved offline bundle is present. |
| PKG-TRUST-001 | A package repository snapshot whose timestamp metadata is expired fails activation with `repository_metadata_expired`. |
| PKG-TRUST-002 | A package repository metadata set whose snapshot version is lower than `PackageRepositoryAntiRollbackState` permits fails activation with `repository_rollback_detected`. |
| PKG-ATTEST-001 | A provenance attestation whose subject digest does not equal the package artifact digest fails activation with `attestation_subject_mismatch`. |
| PKG-SBOM-001 | A package that requires an SBOM fails activation with `sbom_missing` when no `PackageSBOMRef` matching the artifact subject digest exists. |
| PKG-LOCK-001 | A dependency listed in `PackageDependencyConstraint` but absent from `PackageDependencyLock` fails activation with `dependency_unresolved`. |
| PKG-LOCK-002 | A resolved dependency whose downloaded bytes do not match the locked checksum fails activation with `dependency_checksum_mismatch`. |
| PKG-COMPAT-001 | A package whose runtime protocol version has no passing row in `PackageCompatibilityMatrix` for the target environment fails activation with `runtime_protocol_incompatible`. |
| PKG-PROMOTE-001 | A candidate package set with a failed required validation matrix row cannot produce a `PackagePromotionRecord.promotion_decision = approved`. |
| PKG-PROMOTE-002 | A candidate package set whose shadow execution output checksum differs from the expected checksum fails promotion with `shadow_output_mismatch`. |
| PKG-PROMOTE-003 | A candidate package set whose canary metric violates a declared abort threshold emits `PackageActivationFailureEvent` and preserves the current active package set. |
| PKG-ROLLBACK-001 | Rollback to a package set referenced by mutable tag, mutable branch, unresolved dependency range, or rebuild-from-tip fails with `rollback_target_not_immutable`. |
| PKG-ROLLBACK-002 | Rollback to a prior package set with incompatible stage state schema fails with `rollback_state_incompatible` before active state changes. |
| PKG-EMERG-001 | An emergency override that attempts to activate a new package while bypassing signature verification fails with `emergency_bypass_forbidden`. |
| PKG-EMERG-002 | An emergency quarantine record for a package artifact blocks activation of every package release that references that artifact digest. |
| PKG-COHESION-001 | A `ProductionPackageSetManifest` missing a required member of an active `PackageCohesionGroup` fails build with `cohesion_group_incomplete`. |
| PKG-VM-001 | Production output is rejected with `VERSION_MANIFEST_MISMATCH` when the active package set checksum is absent from `VersionManifest`. |
| PKG-VM-002 | Production replay is rejected before output when any release manifest checksum referenced by the package set differs from the recorded `VersionManifest` checksum. |
| PKG-DEPREC-001 | A deprecated package used outside its `DeprecatedPackageCompatibilityWindowPolicy.allowed_use_classes` fails activation with `deprecated_use_forbidden`. |
| PKG-RETIRED-001 | A retired package fails production, shadow, replay, rollback, and validation-promotion execution unless a read-only historical replay exception explicitly names the package and manifest. |

## Gaps, Risks, Stale Assumptions, and Open Questions

| Type | Item | Required resolution |
| --- | --- | --- |
| Open question | Exact package repository implementation | Decide whether Cadastre uses TUF-compatible repositories, OCI registries with Notation, or a hybrid. |
| Open question | Required SLSA level by package class | Product governance must assign minimum level and exceptions. |
| Open question | Accepted SBOM formats | Decide SPDX, CycloneDX, or both; define validation profiles and minimum fields. |
| Open question | Transparency evidence requirement | Decide whether Rekor inclusion proof/SET/offline bundle is mandatory for all packages or selected package classes. |
| Open question | Default deprecation windows | Approve per-package-type defaults and maximums. |
| Risk | Package-set manifest expansion increases VersionManifest size | Use digest refs and canonical manifests; do not inline large SBOM/provenance documents. |
| Risk | Emergency override could become hidden bypass | Require quorum, TTL, audit reason, forbidden bypass assertion, and post-event review. |
| Risk | Compatibility matrix maintenance burden | Generate matrix rows from declared package public APIs and target runtime inventory where possible, but persist the generated result. |
| Stale assumption | External docs and releases can change | Re-run source discovery before PRD amendment or implementation. |
| Source limit | No external command was executed | Do not treat this report as proof of tool behavior under local runtime conditions. |

## Sources

[^1]: /mnt/data/PRD-Cadastre.md, Product Summary and Document Status, lines 19-67.
[^2]: /mnt/data/PRD-Cadastre.md, normative contracts table, lines 73-128. Direct line capture was partially displayed by shell output but the cited range is the uploaded file range inspected locally.
[^3]: /mnt/data/PRD-Cadastre.md, VersionManifest, lines 2476-2650.
[^4]: /mnt/data/PRD-Cadastre.md, PackageArtifact, lines 4172-4244.
[^5]: /mnt/data/PRD-Cadastre.md, Extension Package Lifecycle, lines 9092-9233.
[^6]: /mnt/data/nlspec-spec.md, NLSpec definition and structural requirements, lines 11-72.
[^7]: /mnt/data/RES-003-taxi-lang.md, direct runtime dependency caution, lines 18-21; Taxi package-system lessons and Cadastre package gaps, lines 522-533.
[^8]: /mnt/data/RES-006-source-adapter-ingestion-patterns.md, source-adapter contract and state/completeness recommendations, lines 15-32; live-source probe production boundary, lines 384-404.
[^9]: The Update Framework Specification, <https://theupdateframework.github.io/specification/latest/>, inspected 2026-05-16; web inspection lines 77-143 and 171-204 covered goals, threat model, root/targets/snapshot/timestamp roles, thresholds, and metadata protections.
[^10]: SLSA v1.2 and SLSA Build Provenance documentation, <https://slsa.dev/spec/v1.2/> and <https://slsa.dev/provenance/v1>, inspected 2026-05-16; web inspection lines 41-100 of the SLSA spec and lines 67-159, 233-254, and 294-318 of provenance covered levels, provenance predicate, subjects, builder.id, buildType, and materials/dependencies.
[^11]: in-toto metadata model documentation, <https://in-toto.readthedocs.io/en/latest/model.html>, inspected 2026-05-16; web inspection lines 414-454 and 495-504, 747-779 covered link metadata, materials, products, layouts, steps, thresholds, and artifact rules.
[^12]: Sigstore Cosign and Policy Controller official docs, <https://docs.sigstore.dev/cosign/> and <https://docs.sigstore.dev/policy-controller/>, inspected 2026-05-16; web inspection covered keyless signing, Fulcio, Rekor, bundles, OIDC identity, admission policy, and policy authorities.
[^13]: OCI Image Specification descriptor and OCI Distribution 1.1 referrers documentation, <https://github.com/opencontainers/image-spec> and <https://github.com/opencontainers/distribution-spec>, inspected 2026-05-16; web inspection covered descriptors, digest, size, mediaType, subject, and referrers.
[^14]: Notation trust policy documentation, <https://notaryproject.dev/docs/user-guides/trust-store-trust-policy/>, inspected 2026-05-16; web inspection covered trust stores, trusted identities, registry scopes, and signature verification prerequisites.
[^15]: Terraform dependency lock file, provider lock, and provider plugin protocol documentation, <https://developer.hashicorp.com/terraform/language/files/dependency-lock>, <https://developer.hashicorp.com/terraform/cli/commands/providers/lock>, and <https://developer.hashicorp.com/terraform/plugin/terraform-plugin-protocol>, inspected 2026-05-16.
[^16]: Helm chart, provenance, history, and rollback documentation, <https://helm.sh/docs/topics/charts/>, <https://helm.sh/docs/topics/provenance/>, <https://helm.sh/docs/helm/helm_history/>, and <https://helm.sh/docs/helm/helm_rollback/>, inspected 2026-05-16.
[^17]: OPA bundle, discovery, and CLI documentation, <https://www.openpolicyagent.org/docs/latest/management-bundles/>, <https://www.openpolicyagent.org/docs/latest/configuration/#discovery>, and <https://www.openpolicyagent.org/docs/latest/cli/>, inspected 2026-05-16.
[^18]: Argo CD automated sync docs, Argo Rollouts canary and analysis docs, and Flux HelmRelease API docs, inspected 2026-05-16; source URLs include <https://argo-cd.readthedocs.io/>, <https://argo-rollouts.readthedocs.io/>, and <https://fluxcd.io/flux/components/helm/helmreleases/>.
[^19]: SemVer 2.0.0, Kubernetes API deprecation policy, OpenTelemetry versioning/status docs, and Cloud Native Buildpacks Buildpack/Platform API and lifecycle README, inspected 2026-05-16; source URLs include <https://semver.org/>, <https://kubernetes.io/docs/reference/using-api/deprecation-policy/>, <https://opentelemetry.io/docs/specs/otel/versioning-and-stability/>, and <https://buildpacks.io/docs/reference/spec/buildpack-api/>.
[^20]: SPDX Specification 3.0.1, SPDX License List, SPDX Package, SBOM, Vulnerability, and PackageVerificationCode pages, <https://spdx.github.io/spdx-spec/v3.0.1/> and <https://spdx.org/licenses/>, inspected 2026-05-16.
[^21]: CycloneDX Specification Overview, v1.7 reference, and v1.7 release note, <https://cyclonedx.org/specification/overview/>, <https://cyclonedx.org/docs/1.7/proto/>, and <https://cyclonedx.org/news/cyclonedx-v1.7-released/>, inspected 2026-05-16.
