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

- `CanonicalEntitySchema`
- `SourceAssetSchema`
- `IdentifierSchema`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`
- `CanonicalJSON`
- `DecimalPrecisionPolicy.confidence_0_1`

## Exports

- `ResolverProfile`
- `ResolverProfileRow`
- `ResolverDecisionMatrix`
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

## Resolver Authority

`ResolverProfile` is the sole production authority for identity resolution. A resolver run must fail when no active profile covers resolver run mode, entity type, source scopes, evidence classes, and lifecycle boundary types.

Identity inputs must be materialized as typed `IdentityEvidenceItem` rows before candidate generation, blocker evaluation, review routing, creation, attachment, merge, split, reject, conflict, or no-decision output.

A resolver may create or update `CanonicalEntity`, `SourceAsset`, or `Identifier` records only by emitting records that pass the corresponding `040` schema and `040.ValidateCoreRecord`. `creation_identity_decision_id`, `resolver_profile_id`, and `identity_policy_version` must be present when a `CanonicalEntity` is created. `SourceAsset.source_scope`, `SourceAsset.source_native_identity`, and `SourceAsset.source_asset_type` must be sufficient for `040.SourceAssetSchema`; under-scoped source identity fails before candidate generation. `Identifier` outputs must include typed scope, quality, validity, known time, and evidence refs. `ResolverExplanation` must reference core record IDs and checksums rather than duplicate core fields.

### Resolver Determinism Closure

Each production resolver run must resolve exactly one active `ResolverProfileRow` before candidate generation. Resolution must fail before mutation when any row, artifact ref, checksum, source scope, identifier scope, lifecycle status, activation scope, or validation ref is missing, inactive, ambiguous, mismatched, under-scoped, or outside the run scope. The selected row must name every output-affecting resolver artifact, and every selected artifact and runtime state ref must appear in `030.VersionManifest`.

A production resolver must not use implementation-local defaults for evidence class authority, blocking keys, candidate caps, hard blocker precedence, confidence bands, review routing, split partitioning, explanation checksum inputs, or learned artifacts. Omitted values must use the defaults in this spec or must emit the most specific resolver error before identity mutation.

### ResolverProfileRow schema

`ResolverProfileRow` is the row-level executable interface inside `ResolverProfile`. Concrete row sets are activation-controlled artifacts; stable decision semantics remain owned by this spec.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the resolver profile row set. |
| `resolver_profile_id` | Yes | none | Stable profile ID. |
| `profile_version` | Yes | none | Immutable owner version included in explanation and manifest checksums. |
| `run_mode` | Yes | none | Closed token; production selection requires `production` unless a validation/shadow/canary mode is explicitly requested. |
| `entity_type` | Yes | none | Must match one row in `ResolverProfileCoverageMatrix`. |
| `source_scope_selector` | Yes | none | Canonical selector over source/feed scopes; concrete private source bindings are forbidden in public rows. |
| `source_scope_match_policy` | Yes | `exact_scope_required` | Closed enum: `exact_scope_required`, `scope_subset_allowed_for_review_only`. Subset policy must not create, attach, or merge. |
| `evidence_class_set_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = identifier_evidence_class_row_set`. |
| `identifier_scope_row_refs` | Yes | none | Non-empty canonical array of active `IdentifierScope` row refs. |
| `candidate_generation_profile_ref` | Yes | none | `artifact_class = candidate_generation_profile`. |
| `hard_blocker_row_refs` | Yes | none | Non-empty canonical array of active `AssetGenerationBoundary` or hard-blocker rows. |
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
| `activation_scope` | Yes | none | Vendor-neutral scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

## Resolver artifact activation boundary

Stable identity semantics, weak-evidence defaults, hard-blocker precedence, review terminality, and selector safety are owned by this spec. Resolver profiles, decision rows, candidate bounds, and selector policy material are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `ResolverProfile` | `activation_controlled_artifact` | Sole production resolver behavior authority; must validate before candidate generation. |
| `IdentifierEvidenceClass` row set | stable default semantics plus activation-controlled extensions | Extensions must not allow weak evidence auto-merge unless stable semantics permit. |
| `IdentityEvidenceItem` | `runtime_state_record` | Materialized evidence input. |
| `IdentifierScope` row set | `activation_controlled_artifact` | Scope registry under stable canonicalization rules. |
| `CandidateGenerationProfile` | `activation_controlled_artifact` | Defines blocking keys, pair ordering, caps, overflow, and learned-artifact policy. |
| `AssetGenerationBoundary` row set | `activation_controlled_artifact` | Must preserve stable blocker semantics. |
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

### ResolverArtifactLifecycleGuardRows

Resolver activation artifacts use `030.ActivationControlledArtifactLifecycleMachine.v1`. `070` owns identity-specific guard rows and review-case state-machine requirements.

| Artifact class | Lifecycle binding | Required owner guards |
| --- | --- | --- |
| `ResolverProfile` | `030.ActivationControlledArtifactLifecycleMachine.v1` | Coverage for run mode, entity type, source scopes, evidence classes, lifecycle boundary types, activation scenarios, and blocker coverage. |
| `IdentifierEvidenceClass` row set | Generic artifact lifecycle | Stable weak-evidence defaults preserved; extensions cannot permit weak auto-merge unless stable semantics permit. |
| `IdentifierScope` row set | Generic artifact lifecycle | Scope keys, canonicalization, uncovered-scope behavior, and fixtures. |
| `CandidateGenerationProfile` | Generic artifact lifecycle | Blocking keys, deterministic pair ordering, candidate caps, overflow behavior, and learned-artifact fixtures. |
| `AssetGenerationBoundary` row set | Generic artifact lifecycle | Reimage, clone, VDI, reinstall, delete/recreate, scanner correlation, and lifecycle blocker fixtures. |
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
| Durable cloud provider resource ID | May create only under exact provider plus account/project/subscription scope, region when applicable, compatible generation, exact normalized value, and no blocker. | May attach under the same exact scope and no blocker. | May merge only when the durable ID bundle proves the same source object under exact scope and no blocker. | provider, account/project/subscription, region when applicable, generation key when available | `positive_evidence` | Global identity outside the exact scope. |
| Endpoint agent durable ID | May create under exact enrollment scope and no blocker. | May attach under exact enrollment scope and no blocker. | May merge only when the same durable ID is observed in the exact enrollment scope or when a qualifying durable provider ID bundle also matches. | enrollment tenant, deployment, agent namespace, generation when available | `positive_evidence` | Merge across reinstall, clone, or VDI boundary without qualifying durable provider evidence. |
| Directory user ID | May create for `user` under exact directory tenant/domain/source scope and no blocker. | May attach under exact directory tenant/domain/source scope and no blocker. | May merge only under exact durable directory identity and no delete/recreate or reenrollment blocker. | directory tenant/domain/source and immutable user identifier | `positive_evidence` | Name, mail, UPN, or display-label merge by itself. |
| Service account durable ID | May create for `service_account` under exact provider/account/tenant scope and no blocker. | May attach under exact provider/account/tenant scope and no blocker. | May merge only under exact durable provider or directory ID and no rekey/delete blocker. | provider/account/tenant or directory tenant plus service account ID | `positive_evidence` | Secret name, display name, or role-name merge by itself. |
| Kubernetes UID | Must not create Cadastre canonical host identity by default. | May attach only as source-object identity when scope is cluster, namespace, resource kind, UID, and generation. | Never by default. | cluster, namespace, resource kind, UID, generation when available | `source_object_identity` | Name-based workload or host identity merge. |
| IP address only | Never. | Never. | Never. | address family, observed interval, source scope | `candidate_hint` | Any create, attach, or merge. |
| Hostname only | Never. | Never. | Never. | source scope and observed interval | `candidate_hint` | Any create, attach, or merge. |
| DNS name only or PTR only | Never. | Never. | Never. | zone/source scope and observed interval | `candidate_hint` | Any create, attach, or merge. |
| Flow ID only | Never. | Never. | Never. | sensor scope and observation interval | `correlation_hint` | Identity creation or relationship endpoint merge. |
| Scanner name only | Never. | Never. | Never. | scanner scope and scan interval | `candidate_hint` | Asset identity merge. |
| Source-native merge history only | Never. | Never. | Never. | source scope and source history ref | `lineage_only` | Cadastre identity authority by itself. |
| Semantic overlay only | Never. | Never. | Never. | mapping artifact scope | `lineage_only` | Identity evidence. |
| Mapped target only | Never. | Never. | Never. | selector source scope | `selector` | Identity evidence. |
| Graph key only | Never. | Never. | Never. | graph backend profile scope | `selector` | Identity evidence. |

## Candidate Generation

`CandidateGenerationProfile` must define blocking keys, allowed heuristics, prohibited selectors, deterministic pair ordering, candidate caps, overflow behavior, and learned-artifact policy. Learned candidate generation may propose candidates only when `learned_artifact_policy != disabled`; it must not override hard blockers, lifecycle boundaries, decision matrix rows, confidence bands, review routing, or split policy.

Candidate generation must materialize blocking keys before candidate pairs. A blocking key whose required scope input is absent must be rejected as under-scoped and must not produce candidates. A blocking key whose member count exceeds `max_members_per_blocking_key` must emit `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` and must not produce auto-merge output from that block.

| Block kind rank | Blocking key class | Canonical key string | Scope hash inputs | Generation handling | Default output when key is weak or selector-only |
| ---: | --- | --- | --- | --- | --- |
| 10 | durable provider ID | `provider_id:<provider>:<normalized_value>` | provider, account/project/subscription, region when applicable | append `generation:<asset_generation_key>` when present; missing generation remains eligible only if no generation blocker fires | may create, attach, or merge only through decision matrix |
| 20 | endpoint agent ID | `endpoint_agent_id:<namespace>:<normalized_value>` | enrollment tenant, deployment, agent namespace | append generation when present; reinstall blocker wins before confidence | may create or attach; merge requires same enrollment scope or provider durable bundle |
| 30 | directory user ID | `directory_user_id:<directory_namespace>:<normalized_value>` | directory tenant/domain/source | delete/recreate or reenrollment blocker wins | may create, attach, or merge only for `user` |
| 40 | service account durable ID | `service_account_id:<namespace>:<normalized_value>` | provider/account/tenant or directory tenant | rekey/delete blocker wins | may create, attach, or merge only for `service_account` |
| 50 | Kubernetes UID | `k8s_uid:<normalized_uid>` | cluster, namespace, resource kind | UID plus generation identifies source object only by default | source-object identity only; no canonical merge by default |
| 60 | weak hostname | `hostname:<nfc_lowercase_hostname>` | source scope and observed interval | hostname reuse blocker wins | candidate or no_decision only |
| 70 | weak IP | `ip:<address_family>:<canonical_ip>` | source scope and observed interval | IP reuse blocker wins | candidate or no_decision only |
| 80 | weak DNS/PTR | `dns:<canonical_fqdn>` or `ptr:<canonical_ptr>` | zone/source scope and observed interval | conflicting resolution blocker wins | candidate or no_decision only |
| 90 | selector-only | `selector:<mechanism>:<normalized_value>` | selector source scope | no generation authority | unresolved target or no_decision only |

`scope_hash` must be `sha256` over `040.CanonicalJSON` of the ordered scope inputs declared for the block kind. Candidate pair ordering must sort by block kind rank, blocking key UTF-8 bytes, lexical left source asset ID, lexical right source asset ID, and evidence item set checksum. The left source asset ID is the lexically smaller source asset ID; ties use lexical evidence item ID order.

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

## ResolveIdentity Algorithm

```text
ResolveIdentity(evidence_items, resolver_profile, source_authority_context, version_manifest):
1. Validate `ResolverProfile`, `ResolverProfileRow`, `IdentifierEvidenceClass`, `IdentifierScope`, `CandidateGenerationProfile`, `AssetGenerationBoundary`, `ResolverDecisionMatrix`, `IdentityConfidenceBand`, `IdentityReviewRoutingPolicy`, `SplitPolicy`, `ResolverExplanationPolicy`, and `TargetSelectorSafetyPolicy` refs through `030.ActivationControlledArtifactRef`.
2. Fail before candidate generation when any required row or artifact is inactive, missing, ambiguous, checksum-mismatched, out of scope, superseded, or unvalidated.
3. Resolve exactly one active `ResolverProfileRow` for run mode, entity type, source scopes, evidence classes, lifecycle boundary types, and activation scope.
4. Normalize identifiers into `IdentifierScope`-aware `IdentityEvidenceItem` rows and materialize defaults before checksum computation.
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

## Identity Resolution Contract Details

### ResolverProfileCoverageMatrix

| Entity type | Run mode | Source scope coverage | Evidence classes | Lifecycle boundary classes | Allowed decision outputs | Validation scenario families |
| --- | --- | --- | --- | --- | --- | --- |
| `host` | `production`, `shadow`, `canary`, `validation` | provider/account/project/subscription, region when applicable, endpoint enrollment scope, scanner/feed scope, cluster scope when present | durable provider ID, endpoint agent ID, Kubernetes UID, weak hostname, weak IP, weak DNS/PTR, scanner name, graph key selector, source-native merge history | reimage, clone, VDI reuse, agent reinstall, provider delete/recreate, hostname reuse, IP reuse, Kubernetes recreate, scanner correlation change | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-weak-rejection`, `identity-hard-blocker`, `identity-candidate-overflow`, `identity-split-handoff`, `identity-explanation-checksum` |
| `user` | `production`, `shadow`, `canary`, `validation` | directory tenant/domain/source, provider tenant when federated | directory user ID, durable provider principal ID, weak name/mail/UPN hint, group-membership hint, graph key selector, source-native merge history | delete/recreate, reenrollment, hidden membership or permission-limited evidence | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-review-state-machine`, `identity-selector-safety`, `identity-replay` |
| `service_account` | `production`, `shadow`, `canary`, `validation` | provider/account/tenant, directory tenant, workload namespace when applicable | service account durable ID, provider principal ID, directory service principal ID, weak display-name hint, graph key selector, source-native merge history | rekey, delete/recreate, directory reenrollment, namespace recreation | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` | `identity-create`, `identity-attach`, `identity-durable-merge`, `identity-hard-blocker`, `identity-confidence-band`, `identity-package-artifact-core-conflict` |
| `unsupported_entity_type` | any | any | any | any | `no_decision` only | `identity-unsupported-entity-type` |

The `unsupported_entity_type` row is closed behavior. It must emit `RESOLVER_ENTITY_TYPE_UNSUPPORTED`, must emit `no_decision`, must not open review, and must not mutate `CanonicalEntity`, `SourceAsset`, or `Identifier`.

### IdentifierScope canonicalization

| Source class | Identifier class | Required scope keys | Normalization | Uncovered-scope behavior |
| --- | --- | --- | --- | --- |
| cloud provider | provider resource ID | provider, account/project/subscription, region when applicable, generation when available | provider lowercased; ID value exact after NFC; region lowercased | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| endpoint agent | agent durable ID | enrollment tenant, deployment, agent namespace, generation when available | exact ID string after NFC | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| directory | user or service account durable ID | directory tenant/domain/source and immutable object ID | exact ID string after NFC; display names excluded | `IDENTITY_EVIDENCE_UNDER_SCOPED`; no candidates |
| Kubernetes | UID | cluster, namespace, resource kind, UID, generation when available | exact UID after NFC | source-object identity only by default |
| network | IP address | address family, observed interval, source scope | canonical textual IP | weak evidence only |
| DNS | name/PTR | zone/source scope, observed interval | lowercase FQDN with trailing-dot normalization | weak evidence only |
| graph | backend key | backend profile and graph object scope | exact string after NFC | selector only; no identity |

### IdentifierEvidenceClass registry

| Evidence class | Durability | Scope | Reuse risk | Default role | Creation/attachment/merge authority | Negative evidence effect | Review routing |
| --- | --- | --- | --- | --- | --- | --- | --- |
| durable provider ID | high | exact provider/account/project/subscription, region when applicable | medium after delete/recreate | `positive_evidence` | create, attach, and merge only under exact scope, compatible generation, exact normalized value, and no blocker | generation or conflict blocker wins | no review unless conflict or split |
| endpoint agent ID | high | exact enrollment tenant/deployment/namespace | medium after reinstall, clone, VDI | `positive_evidence` | create or attach; merge only under exact enrollment scope or qualifying durable provider bundle | reinstall/clone/VDI blocker wins | review on conflict |
| directory user ID | high | exact directory tenant/domain/source | medium after delete/recreate or reenrollment | `positive_evidence` | create, attach, or merge for `user` only under exact durable ID and no blocker | reenrollment/delete blocker wins | review on conflict or hidden-membership gap |
| service account durable ID | high | exact provider/account/tenant or directory tenant | medium after rekey/delete | `positive_evidence` | create, attach, or merge for `service_account` only under exact durable ID and no blocker | rekey/delete blocker wins | review on conflict |
| Kubernetes UID | high for source object, not canonical host | exact cluster/namespace/resource/generation | medium after recreate | `source_object_identity` | no canonical merge by default | recreate blocker wins | no_decision unless profile opens review |
| hostname | low | source/time scoped | high | `candidate_hint` | never | reuse blocks weak merge | review candidate only |
| IP address | low | source/time scoped | high | `candidate_hint` | never | reuse blocks weak merge | review candidate only |
| DNS/PTR | low | zone/time scoped | high | `candidate_hint` | never | conflicting resolution blocks weak merge | review candidate only |
| flow ID | low | sensor scoped | high | `correlation_hint` | never | none by itself | no_decision |
| graph key | low | backend scoped | high | `selector` | never | none by itself | no_decision |
| source-native merge history | medium | source scoped | medium | `lineage_only` | never | contradiction may block | review candidate only |
| learned hint | model-artifact scoped | resolver scope | model drift risk | `candidate_hint` | never | none by itself | review candidate only |

### Resolver decision matrix

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

MVP confidence is rule-banded, not probabilistic. Scores must be persisted as `040.DecimalPrecisionPolicy.confidence_0_1` values with six fractional digits and no rounding.

| Band | Score | Selection condition | Permitted decision classes |
| --- | ---: | --- | --- |
| `blocked` | `0.000000` | Any non-overridable hard blocker fired. | `rejected`, `no_decision`, `conflicted` |
| `selector_only` | `0.000000` | Only selector evidence exists. | `no_decision` |
| `weak_hint_only` | `0.250000` | Only weak hostname/IP/DNS/PTR/scanner hint exists. | `candidate`, `rejected`, `no_decision` |
| `learned_hint_only` | `0.300000` | Only learned or model-generated hint exists. | `candidate`, `rejected`, `no_decision` |
| `single_durable_creation` | `0.900000` | One exact durable evidence bundle under exact scope and no existing canonical candidate. | `canonical_created` |
| `exact_durable_attach` | `0.950000` | One exact durable evidence bundle attaches to exactly one existing canonical entity and no blocker fires. | `source_asset_attached` |
| `exact_durable_merge` | `1.000000` | Exact compatible durable evidence proves the same object across existing canonical entities and no blocker fires. | `auto_merged` |
| `conflicting_durable` | `0.500000` | Durable evidence conflicts or equally specific rows disagree. | `conflicted`, `candidate`, `rejected` |

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

### Review routing defaults

| Candidate condition | Default routing | Reviewer authority limit |
| --- | --- | --- |
| Weak-only candidate | May open review when policy permits. | Reviewer must not approve merge without new durable evidence materialized as `IdentityEvidenceItem`. |
| Learned-only candidate | May open review when policy permits. | Reviewer must not approve merge without new durable evidence and passing replay fixture. |
| Conflicting durable evidence | Must open review when profile permits review; otherwise emit `conflicted`. | Reviewer may approve only after conflict-resolving durable evidence snapshot matches the case. |
| Candidate overflow | May open diagnostic review only when profile permits. | Reviewer must not approve merge from an overflowed block; rerun under non-overflow conditions is required. |
| Hard blocker fired | Review may document blocker only. | Reviewer must not override non-overridable blockers. |
| Selector-only evidence | No review by default. | No creation, attachment, merge, or split authority. |

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

### Identity closed enum tables

| Enum family | Closed values |
| --- | --- |
| identity decision states | `canonical_created`, `source_asset_attached`, `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` |
| review case states | `opened`, `in_review`, `blocked`, `expired`, `cancelled`, `terminal_decision_emitted` |
| review events | `assign`, `approve_merge`, `reject_merge`, `approve_split`, `request_more_evidence`, `evidence_supplied`, `expire`, `cancel`, `replay_same_event` |
| evidence roles | `positive_evidence`, `negative_evidence`, `candidate_hint`, `selector`, `lineage_only`, `correlation_hint`, `source_object_identity` |
| selector mechanisms | `mapped_target`, `opengraph_property_matching`, `graph_key`, `hostname`, `ip_address`, `dns_name`, `ptr_name` |

### IdentityApiHandoff

Resolver and target-selector outcomes that surface through asset detail, graph query, evidence drillback, health, audit, or export must use structured owner context and the generated `110.ErrorCodeRegistry`.

| Identity condition | Recommended `110` label or outcome | Required behavior |
| --- | --- | --- |
| resolver profile missing | `error` | Emit `RESOLVER_PROFILE_MISSING`; no identity mutation. |
| resolver row ambiguous | `ambiguous` | Emit `RESOLVER_PROFILE_ROW_AMBIGUOUS`; no implementation-order selection. |
| selector-only unresolved target | diagnostic or no output | No identity, no graph endpoint, no canonical entity creation. |
| graph endpoint unresolved | graph owner error consumed by `110` | Emit `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; no dependent graph edge. |
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
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | Entity type maps to the unsupported row. |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | Required scope keys are absent or non-canonical. |
| `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | Evidence class lacks a covering resolver profile row. |
| `IDENTITY_HARD_BLOCKER_TRIGGERED` | Hard blocker or generation boundary prevents create, attach, or merge. |
| `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | A blocking key exceeds `max_members_per_blocking_key`. |
| `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | Candidate pairs exceed per-source-asset or resolver-partition caps. |
| `RESOLVER_DECISION_ROW_MISSING` | No decision matrix row covers a candidate after blocker evaluation. |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | No confidence band row covers the candidate decision state. |
| `RESOLVER_REVIEW_ROUTING_MISSING` | Review is required but no active review routing policy row covers the candidate condition. |
| `RESOLVER_SPLIT_POLICY_MISSING` | A split decision is selected without an active split policy. |
| `RESOLVER_EXPLANATION_INCOMPLETE` | Required explanation fields or checksum inputs are missing. |
| `IDENTITY_REVIEW_TRANSITION_INVALID` | Review event is illegal for the current state. |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | Review event evidence snapshot checksum does not match the case snapshot. |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | Reviewer lacks authority for the requested review event. |
| `TARGET_SELECTOR_UNSAFE` | Selector attempts a resolution state beyond `TargetSelectorSafetyPolicy`. |
| `DEPRECATED_NAME_MATCHING_FORBIDDEN` | Deprecated name matching is used in production. |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | Graph projection requests endpoint identity from unresolved target hints, selector-only evidence, graph keys, or non-qualifying identity material. |

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

| Selector mechanism | Maximum resolution state | Evidence required | Auto-merge allowed | Graph projection effect | Error/no-op behavior |
| --- | --- | --- | ---: | --- | --- |
| mapped target | `UnresolvedTargetReference` | selector source and scope | No | hint only | no-op if unresolved |
| OpenGraph property matching | `UnresolvedTargetReference` | active selector policy | No | no projection unless owner permits | reject deprecated name matching |
| graph key | selector only | graph profile and owner row | No | no identity influence | no-op |
| hostname/IP/DNS selector | candidate hint | temporal source scope | No | no graph edge by itself | candidate or no_decision |

### GraphEndpointIdentityHandoff validation rows

| Validation row | Required behavior |
| --- | --- |
| `val-070-graph-endpoint-requires-identity-decision` | A projected graph endpoint requires a qualifying `IdentityDecision` and `CanonicalEntity` ref. |
| `val-070-unresolved-target-no-graph-endpoint` | `UnresolvedTargetReference` emits no graph endpoint and no edge. |
| `val-070-opengraph-property-match-no-graph-identity` | OpenGraph-style property matching remains selector-only and cannot create endpoint identity. |
| `val-070-split-handoff-no-resolver-graph-mutation` | Split handoff routes affected facts to `090` and the resolver emits no graph mutation. |

### IdentityErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | default_severity | default_retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `RESOLVER_ARTIFACT_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-artifact-missing` |
| `RESOLVER_ARTIFACT_INACTIVE` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-artifact-inactive` |
| `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-artifact-checksum-mismatch` |
| `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-artifact-scope-mismatch` |
| `RESOLVER_PROFILE_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-profile-missing` |
| `RESOLVER_PROFILE_ROW_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-profile-row-missing` |
| `RESOLVER_PROFILE_ROW_AMBIGUOUS` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-profile-row-ambiguous` |
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-entity-type-unsupported` |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-evidence-under-scoped` |
| `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-evidence-class-unsupported` |
| `IDENTITY_HARD_BLOCKER_TRIGGERED` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-hard-blocker-triggered` |
| `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-candidate-block-overflow` |
| `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-candidate-partition-overflow` |
| `RESOLVER_DECISION_ROW_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-decision-row-missing` |
| `RESOLVER_CONFIDENCE_BAND_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-confidence-band-missing` |
| `RESOLVER_REVIEW_ROUTING_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-review-routing-missing` |
| `RESOLVER_SPLIT_POLICY_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-split-policy-missing` |
| `RESOLVER_EXPLANATION_INCOMPLETE` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-resolver-explanation-incomplete` |
| `IDENTITY_REVIEW_TRANSITION_INVALID` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-review-transition-invalid` |
| `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-review-evidence-snapshot-mismatch` |
| `IDENTITY_REVIEW_AUTHORITY_MISSING` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-identity-review-authority-missing` |
| `TARGET_SELECTOR_UNSAFE` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-target-selector-unsafe` |
| `DEPRECATED_NAME_MATCHING_FORBIDDEN` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-deprecated-name-matching-forbidden` |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `070` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `070.IdentityErrorContext` | `identity-error-graph-endpoint-identity-unresolved` |

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
| `070-MVP-MATRIX-AC-001` | `host`, `user`, `service_account`, and `unsupported_entity_type` are covered by the MVP coverage matrix with closed decision outputs. |
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
| `070-VOLATILITY-AC-001` | Inactive resolver profile, target selector artifact omission, resolver artifact checksum mismatch, and package artifact core conflict fail before identity mutation. |
| `070-LIFECYCLE-AC-001` | Resolver profile activation emits generic artifact lifecycle transition evidence and resolver-specific guard results before identity mutation. |

| `070-GRAPH-ENDPOINT-HANDOFF-AC-001` | Graph endpoint projection requires a qualifying identity decision and canonical entity ref. |
| `070-GRAPH-ENDPOINT-HANDOFF-AC-002` | Selector-only, graph-key-only, mapped-target-only, and OpenGraph property-match-only inputs cannot create graph endpoint identity. |
| `070-GRAPH-ENDPOINT-HANDOFF-AC-003` | Unresolved endpoint identity is reported to `090` as a projection blocker and never mutates identity. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `070-AC-001` | Same inputs, active resolver row, source scopes, blockers, artifact refs, and version manifest produce byte-identical identity decisions and explanations. |
| `070-AC-002` | IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-only, mapped-target-only, learned-only, and selector-only evidence never create, attach, or merge. |
| `070-AC-003` | Hard blockers and lifecycle boundaries override confidence scores and reviewer notes. |
| `070-AC-004` | Manual review cannot mutate identity without terminal `IdentityDecision`, `IdentityReviewCase`, `ResolverExplanation`, and `VersionManifest` refs. |
| `070-AC-005` | Every identity split that affects gold or graph output emits `GraphCorrectionHandoff` and no resolver graph mutation. |
| `070-AC-006` | Authoritative promotion fails while any resolver closure validation row is missing, blocked, not run, failed, checksum-mismatched, or has a `TODO` expected output. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
