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

## Exports

- `ResolverProfile`
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

Identity inputs must be materialized as typed `IdentityEvidenceItem` rows before candidate generation, blocker evaluation, review routing, merge, split, reject, conflict, or no-decision output.

A resolver may create or update `CanonicalEntity`, `SourceAsset`, or `Identifier` records only by emitting records that pass the corresponding `040` schema and `040.ValidateCoreRecord`. `creation_identity_decision_id`, `resolver_profile_id`, and `identity_policy_version` must be present when a `CanonicalEntity` is created. `SourceAsset.source_scope`, `SourceAsset.source_native_identity`, and `SourceAsset.source_asset_type` must be sufficient for `040.SourceAssetSchema`; under-scoped source identity fails before candidate generation. `Identifier` outputs must include typed scope, quality, validity, known time, and evidence refs. `ResolverExplanation` must reference core record IDs and checksums rather than duplicate core fields.

## Resolver artifact activation boundary

Stable identity semantics, weak-evidence defaults, hard-blocker precedence, review terminality, and selector safety are owned by this spec. Resolver profiles, decision rows, candidate bounds, and selector policy material are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `ResolverProfile` | `activation_controlled_artifact` | Sole production resolver behavior authority; must validate before candidate generation. |
| `IdentifierEvidenceClass` row set | stable default semantics plus activation-controlled extensions | Extensions must not allow weak evidence auto-merge unless stable semantics permit. |
| `IdentityEvidenceItem` | `runtime_state_record` | Materialized evidence input. |
| `IdentifierScope` row set | `activation_controlled_artifact` | Scope registry under stable canonicalization rules. |
| `CandidateGenerationProfile` | `activation_controlled_artifact` | Must define bounds or remain `blocked_owner_todo`. |
| `AssetGenerationBoundary` row set | `activation_controlled_artifact` | Must preserve stable blocker semantics. |
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
| `CandidateGenerationProfile` | Generic artifact lifecycle | Blocking keys, deterministic pair ordering, candidate caps, and overflow behavior. |
| `AssetGenerationBoundary` row set | Generic artifact lifecycle | Reimage, clone, VDI, reinstall, delete/recreate, scanner correlation, and lifecycle blocker fixtures. |
| `TargetSelectorSafetyPolicy` | Generic artifact lifecycle | Selector maximum resolution state and forbidden identity side effects. |

### IdentityReviewCaseStateMachineBinding

Machine ID: `070.IdentityReviewCaseStateMachine.v1`.

`IdentityReviewCase` uses an owner-local deterministic state machine rather than shared `LifecycleStatus`. Review state changes must emit owner-local transition evidence and must not mutate canonical identity except through a terminal `IdentityDecision` record.

TODO: `070` must add closed review states, closed events, a total transition matrix, reviewer authority guard order, expiration behavior, terminal identity-decision output rules, illegal-transition behavior, idempotency key handling, and validation rows for `070.IdentityReviewCaseStateMachine.v1` before authoritative promotion. The existing `IdentityReviewCase state machine` table remains a partial transition catalog and does not close this TODO.

## Evidence Roles

| Evidence class default | Auto-merge authority |
| --- | --- |
| Durable cloud provider resource ID within exact provider scope | Profile-defined. |
| Endpoint agent durable ID within enrollment scope | Profile-defined. |
| Kubernetes UID within cluster/resource/generation scope | Source-object identity only by default. |
| IP address only | Never. |
| Hostname only | Never. |
| DNS name only | Never. |
| PTR only | Never. |
| Flow ID only | Never. |
| Scanner name only | Never. |
| Source-native merge history only | Never. |
| Semantic overlay only | Never. |
| Mapped target only | Never. |
| Graph key only | Never. |

## Candidate Generation

`CandidateGenerationProfile` must define blocking keys, allowed heuristics, prohibited selectors, deterministic pair ordering, candidate caps, overflow behavior, and learned-artifact policy. Learned candidate generation may propose candidates but must not override hard blockers, lifecycle boundaries, or decision matrix rows.

## Hard Blocker Precedence

Hard blockers and lifecycle generation boundaries must be evaluated before confidence computation and before any decision matrix row can permit auto-merge.

| Boundary | Default effect |
| --- | --- |
| Reimage evidence | Blocks auto-merge across boundary unless profile permits split-aware continuation. |
| Clone or VDI reuse | Blocks auto-merge across generation. |
| Agent reinstall | Blocks unless source scope and durable external evidence repair the generation. |
| Delete/recreate | Blocks across provider object generations. |
| Hostname reuse | Blocks hostname-only and weak merge. |
| IP reuse | Blocks IP-only and time-overlapping weak merge. |
| Directory reenrollment | Blocks source-local identity unless profile row permits. |
| Kubernetes recreate | Blocks name-based identity without UID/generation evidence. |
| Scanner correlation change | Blocks scanner-name-only and scanner-merge-only identity. |

## ResolveIdentity Algorithm

```text
ResolveIdentity(evidence_items, resolver_profile, source_authority_context, version_manifest):
1. Validate `ResolverProfile`, `CandidateGenerationProfile`, `IdentifierScope`, `IdentifierEvidenceClass`, `AssetGenerationBoundary`, and `TargetSelectorSafetyPolicy` refs through `030.ActivationControlledArtifactRef`.
2. Fail before candidate generation when any required artifact is inactive, missing, checksum-mismatched, out of scope, or unvalidated.
3. Validate active resolver_profile coverage for entity type, source scopes, run mode, evidence classes, and lifecycle boundaries.
4. Normalize identifiers into IdentifierScope-aware IdentityEvidenceItem rows.
5. Reject uncovered or under-scoped evidence with the most specific resolver error.
6. Generate candidate pairs through CandidateGenerationProfile.
7. Sort candidate pairs by deterministic pair ordering.
8. Evaluate hard blockers and lifecycle generation boundaries.
9. Select decision matrix rows only after blocker evaluation.
10. Compute deterministic confidence band or governed score as profile defines.
11. Emit one IdentityDecision per candidate: auto_merged, candidate, rejected, split, conflicted, or no_decision.
12. Persist IdentityReviewCase when manual review is required.
13. Persist exactly one ResolverExplanation per IdentityDecision.
14. Emit GraphCorrectionHandoff for every split that affects prior gold facts or graph output.
15. Include every output-affecting resolver artifact ref in `VersionManifest`.
```

## Review Contract

Manual review must not mutate canonical identity directly. `IdentityReviewCase` must define closed states, reviewer authority, evidence snapshot checksums, transitions, expiration, terminal decision outputs, and illegal-transition behavior. Identity mutation may occur only through terminal `IdentityDecision` records.

## Target Selector Safety

`UnresolvedTargetReference` represents relationship hints without creating or merging canonical entities. `TargetSelectorSafetyPolicy` must define selector-specific maximum resolution state and evidence requirements.

OpenGraph-style property matching, name matching, source-kind matching, environment-scoped matching, cross-source reference matching, or mapped-target matching must not create canonical identity without a qualifying identity decision. Deprecated name matching is forbidden in production.

## Identity Resolution Contract Details

### ResolverProfileCoverageMatrix

| Entity type | Run mode | Source scopes | Evidence classes | Lifecycle boundary types | Decision output classes | Validation scenarios |
| --- | --- | --- | --- | --- | --- | --- |
| `host` | `production` | provider/account/tenant/site/cluster as applicable | durable IDs plus weak evidence classes | reimage, clone, VDI, reinstall, delete/recreate, reuse | auto_merged, candidate, rejected, split, conflicted, no_decision | weak evidence rejection, hard blocker, split handoff |
| `user` | `production` | directory tenant/domain/source | directory IDs plus names and memberships | delete/recreate, reenrollment | candidate, rejected, conflicted, no_decision unless profile permits merge | permission-limited and hidden-membership cases |
| `service_account` | `production` | provider/account/tenant | durable provider IDs and directory IDs | delete/recreate, rekey | candidate, rejected, no_decision | scope mismatch cases |
| `unsupported_entity_type` | any | any | any | any | no_decision only | emit `RESOLVER_ENTITY_TYPE_UNSUPPORTED`; no identity mutation |

### IdentifierScope canonicalization

| Source class | Identifier class | Required scope keys | Normalization | Uncovered-scope behavior |
| --- | --- | --- | --- | --- |
| cloud provider | provider resource ID | provider, account/project/subscription, region when applicable | provider-lowercase, exact ID string | reject as under-scoped |
| endpoint agent | agent durable ID | vendor-neutral enrollment scope, tenant, deployment | exact ID string | reject as under-scoped |
| Kubernetes | UID | cluster, namespace, resource kind, generation where applicable | exact UID | source-object identity only by default |
| network | IP address | address family, observed time, source scope | canonical textual IP | weak evidence only |
| DNS | name/PTR | zone/source scope, observed time | lowercase FQDN with trailing-dot normalization | weak evidence only |
| graph | backend key | backend profile and graph object scope | exact string | selector only, no identity |

### IdentifierEvidenceClass registry

| Evidence class | Durability | Scope | Reuse risk | Default role | Auto-merge authority | Negative evidence effect | Review routing |
| --- | --- | --- | --- | --- | --- | --- | --- |
| durable provider ID | high | provider-scoped | medium after delete/recreate | positive evidence | profile-defined | lifecycle blocker can override | no review unless conflict |
| endpoint agent ID | high | enrollment-scoped | medium after reinstall | positive evidence | profile-defined | reinstall can block | review on conflict |
| hostname | low | source/time-scoped | high | candidate hint | never | reuse can block weak merge | review candidate only |
| IP address | low | time-scoped | high | candidate hint | never | reuse can block weak merge | review candidate only |
| DNS/PTR | low | zone/time-scoped | high | candidate hint | never | conflicting resolution blocks weak merge | review candidate only |
| flow ID | low | sensor-scoped | high | correlation hint | never | none by itself | no_decision |
| graph key | low | backend-scoped | high | selector | never | none by itself | no_decision |
| source-native merge history | medium | source-scoped | medium | lineage only | never | may become blocker if contradicting | review candidate |

### Resolver decision matrix

| Decision | Required condition | Output behavior |
| --- | --- | --- |
| `auto_merged` | Profile row permits, hard blockers absent, evidence class sufficient, confidence band satisfied | Emit terminal `IdentityDecision`. |
| `candidate` | Evidence suggests match but authority insufficient or review required | Emit `IdentityDecision` plus `IdentityReviewCase`. |
| `rejected` | Candidate fails rule or blocker fires | Emit rejection with explanation. |
| `split` | Prior merge invalidated by split evidence | Emit split decision and `GraphCorrectionHandoff`. |
| `conflicted` | Concurrent evidence prevents deterministic resolution | Emit conflicted decision and review case when permitted. |
| `no_decision` | Evidence is uncovered, weak, selector-only, or out of profile scope | Emit no decision; no identity mutation. |

### IdentityReviewCase state machine

| Current state | Event | Next state | Illegal transition behavior |
| --- | --- | --- | --- |
| `opened` | reviewer assigns | `in_review` | `IDENTITY_REVIEW_TRANSITION_INVALID` |
| `opened` | expires | `expired` | same |
| `in_review` | approve merge | `terminal_decision_emitted` | same |
| `in_review` | reject merge | `terminal_decision_emitted` | same |
| `in_review` | request more evidence | `blocked` | same |
| `blocked` | evidence supplied | `in_review` | same |
| any terminal state | any mutation event | unchanged | same and no mutation |

Manual review must never mutate canonical identity outside terminal `IdentityDecision` records.

`IdentityReviewCase` transition coverage is total. Any state/event pair not listed as an allowed transition must reject mutation, preserve the current state, and emit `IDENTITY_REVIEW_TRANSITION_INVALID`. Expiration routes the case to `expired` and must not mutate canonical identity.

### Identity closed enum tables

| Enum family | Closed values |
| --- | --- |
| identity decision states | `auto_merged`, `candidate`, `rejected`, `split`, `conflicted`, `no_decision` |
| review case states | `opened`, `in_review`, `blocked`, `expired`, `terminal_decision_emitted` |
| review events | `assign`, `approve_merge`, `reject_merge`, `request_more_evidence`, `evidence_supplied`, `expire`, `cancel` |
| evidence roles | `positive_evidence`, `negative_evidence`, `candidate_hint`, `selector`, `lineage_only`, `correlation_hint` |
| selector mechanisms | `mapped_target`, `opengraph_property_matching`, `graph_key`, `hostname`, `ip_address`, `dns_name`, `ptr_name` |

### Resolver error codes

| Error code | Emitted when |
| --- | --- |
| `RESOLVER_ARTIFACT_MISSING` | A required resolver, candidate, scope, evidence-class, boundary, or selector policy artifact ref is missing. |
| `RESOLVER_ARTIFACT_INACTIVE` | A required resolver artifact is not active for production execution. |
| `RESOLVER_ARTIFACT_CHECKSUM_MISMATCH` | A resolver artifact checksum mismatches the active ref or manifest. |
| `RESOLVER_ARTIFACT_SCOPE_MISMATCH` | A resolver artifact does not cover resolver run mode, entity type, source scope, evidence class, or lifecycle boundary. |
| `RESOLVER_PROFILE_MISSING` | No active resolver profile covers run mode, entity type, source scopes, evidence classes, and lifecycle boundaries. |
| `RESOLVER_ENTITY_TYPE_UNSUPPORTED` | Entity type maps to the unsupported row. |
| `IDENTITY_EVIDENCE_UNDER_SCOPED` | Required scope keys are absent or non-canonical. |
| `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | Evidence class lacks a covering resolver profile row. |
| `IDENTITY_HARD_BLOCKER_TRIGGERED` | Hard blocker or generation boundary prevents auto-merge. |
| `IDENTITY_REVIEW_TRANSITION_INVALID` | Review event is illegal for the current state. |
| `TARGET_SELECTOR_UNSAFE` | Selector attempts a resolution state beyond `TargetSelectorSafetyPolicy`. |
| `DEPRECATED_NAME_MATCHING_FORBIDDEN` | Deprecated name matching is used in production. |

### CandidateGenerationProfile bounds

| Field | Default | Maximum | Overflow behavior | Closure state |
| --- | --- | --- | --- | --- |
| `candidate_cap` | TODO: owner decision required | TODO: owner decision required | Sort candidate pairs by deterministic pair ordering, emit overflow output state, and do not auto-merge overflowed candidates. Missing owner values block authoritative resolver promotion. | blocked_owner_todo |

### ResolverExplanation required fields

| Field | Required behavior |
| --- | --- |
| decision row | Reference the selected resolver decision matrix row. |
| blockers evaluated | Include fired and non-fired blocker row IDs. |
| evidence refs | Reference `IdentityEvidenceItem` rows and source evidence refs. |
| confidence band | Include deterministic band or governed score inputs. |
| review routing | Include review case ref when manual review is required. |
| checksum | SHA-256 over canonical explanation bytes. |

### ResolverActivationReport requirements

| Requirement | Default |
| --- | --- |
| scenario pass/fail | required for every evidence class and blocker class |
| shadow run | required before canary |
| canary | required before active production unless owner explicitly defers |
| blocker coverage | every hard blocker has positive and negative fixture |
| deterministic output checksums | required for decisions and explanations |
| promotion eligibility | blocked while any required scenario fails or is missing |

### Hard blocker matrix

| Boundary | Evidence trigger | Default effect | Override allowance | Required validation rows |
| --- | --- | --- | --- | --- |
| reimage | source or temporal evidence crosses generation | block auto-merge | profile row only | blocker override rejection |
| clone or VDI reuse | repeated agent/image evidence | block auto-merge | profile row with split-aware continuation | clone rejection |
| agent reinstall | new enrollment generation | block unless durable external evidence repairs | profile row | reinstall case |
| delete/recreate | provider object generation changes | block cross-generation merge | profile row | delete/recreate case |
| hostname/IP reuse | reused weak value | block weak merge | none for weak-only | weak evidence no-merge |
| Kubernetes recreate | same name, different UID | block name-based identity | UID/generation evidence only | recreate case |

### Selector safety matrix

| Selector mechanism | Maximum resolution state | Evidence required | Auto-merge allowed | Graph projection effect | Error/no-op behavior |
| --- | --- | --- | ---: | --- | --- |
| mapped target | `UnresolvedTargetReference` | selector source and scope | No | hint only | no-op if unresolved |
| OpenGraph property matching | `UnresolvedTargetReference` | active selector policy | No | no projection unless owner permits | reject deprecated name matching |
| graph key | selector only | graph profile and owner row | No | no identity influence | no-op |
| hostname/IP/DNS selector | candidate hint | temporal source scope | No | no graph edge by itself | candidate or no_decision |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `070-CLEANUP-AC-001` | No banned reference class remains. |
| `070-CLEANUP-AC-002` | Identity resolution still fails when no active `ResolverProfile` covers the run mode, entity type, source scopes, evidence classes, and lifecycle boundary types. |
| `070-CLEANUP-AC-003` | Hard blockers and lifecycle boundaries still run before confidence computation and decision matrix selection. |
| `070-CLEANUP-AC-004` | Target selectors still cannot create canonical identity by themselves. |
| `070-SCHEMA-PATCH-AC-001` | Every emitted `CanonicalEntity`, `SourceAsset`, and `Identifier` passes `040.ValidateCoreRecord`. |
| `070-SCHEMA-PATCH-AC-002` | `070` does not restate core record fields except by exact schema name reference. |
| `070-SCHEMA-PATCH-AC-003` | Weak evidence cannot produce a canonical entity or identifier merge by bypassing the `040` and `070` validation sequence. |

| `070-REVIEW-TOTALITY-AC-001` | Every `IdentityReviewCase` state/event pair has an allowed transition or emits `IDENTITY_REVIEW_TRANSITION_INVALID` without identity mutation. |
| `070-EXPLANATION-AC-001` | Every `IdentityDecision` has exactly one `ResolverExplanation` with decision row, blockers, evidence refs, confidence band, review routing, and checksum. |
| `070-VOLATILITY-AC-001` | Inactive resolver profile, target selector artifact omission, and resolver artifact checksum mismatch fail before identity mutation. |
| `070-VOLATILITY-AC-002` | Candidate cap overflow does not auto-merge overflowed candidates and blocks authoritative promotion while numeric caps remain `TODO:`. |
| `070-VOLATILITY-AC-003` | A resolver profile that attempts graph-key-only auto-merge fails before candidate decision output. |
| `070-LIFECYCLE-AC-001` | Resolver profile activation emits generic artifact lifecycle transition evidence and resolver-specific guard results before identity mutation. |
| `070-LIFECYCLE-AC-002` | `070.IdentityReviewCaseStateMachine.v1` remains blocked by explicit TODO rows until review states, events, total transitions, idempotency, expiration, and validation rows are closed. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `070-AC-001` | Same inputs, active resolver profile, source scopes, blockers, and version manifest produce identical identity decisions and explanations. |
| `070-AC-002` | IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-only, and mapped-target-only evidence never auto-merges. |
| `070-AC-003` | Hard blockers and lifecycle boundaries override confidence scores and reviewer notes. |
| `070-AC-004` | Manual review cannot mutate identity without terminal `IdentityDecision`, `IdentityReviewCase`, `ResolverExplanation`, and `VersionManifest` refs. |
| `070-AC-005` | Every identity split that affects gold or graph output emits `GraphCorrectionHandoff`. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
