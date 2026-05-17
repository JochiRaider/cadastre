---
doc_id: CADASTRE-NLSPEC-070
title: Identity Resolution and Target Selectors
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Identity Resolution and Target Selectors

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

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

## Resolver Authority

`ResolverProfile` is the sole production authority for identity resolution. A resolver run must fail when no active profile covers resolver run mode, entity type, source scopes, evidence classes, and lifecycle boundary types.

Identity inputs must be materialized as typed `IdentityEvidenceItem` rows before candidate generation, blocker evaluation, review routing, merge, split, reject, conflict, or no-decision output.

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
1. Validate active resolver_profile coverage for entity type, source scopes, run mode, evidence classes, and lifecycle boundaries.
2. Normalize identifiers into IdentifierScope-aware IdentityEvidenceItem rows.
3. Reject uncovered or under-scoped evidence with the most specific resolver error.
4. Generate candidate pairs through CandidateGenerationProfile.
5. Sort candidate pairs by deterministic pair ordering.
6. Evaluate hard blockers and lifecycle generation boundaries.
7. Select decision matrix rows only after blocker evaluation.
8. Compute deterministic confidence band or governed score as profile defines.
9. Emit one IdentityDecision per candidate: auto_merged, candidate, rejected, split, conflicted, or no_decision.
10. Persist IdentityReviewCase when manual review is required.
11. Persist exactly one ResolverExplanation per IdentityDecision.
12. Emit GraphCorrectionHandoff for every split that affects prior gold facts or graph output.
```

## Review Contract

Manual review must not mutate canonical identity directly. `IdentityReviewCase` must define closed states, reviewer authority, evidence snapshot checksums, transitions, expiration, terminal decision outputs, and illegal-transition behavior. Identity mutation may occur only through terminal `IdentityDecision` records.

## Target Selector Safety

`UnresolvedTargetReference` represents relationship hints without creating or merging canonical entities. `TargetSelectorSafetyPolicy` must define selector-specific maximum resolution state and evidence requirements.

OpenGraph-style property matching, name matching, source-kind matching, environment-scoped matching, cross-source reference matching, or mapped-target matching must not create canonical identity without a qualifying identity decision. Deprecated name matching is forbidden in production.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `070-AC-001` | Same inputs, active resolver profile, source scopes, blockers, and version manifest produce identical identity decisions and explanations. |
| `070-AC-002` | IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-only, and mapped-target-only evidence never auto-merges. |
| `070-AC-003` | Hard blockers and lifecycle boundaries override confidence scores and reviewer notes. |
| `070-AC-004` | Manual review cannot mutate identity without terminal `IdentityDecision`, `IdentityReviewCase`, `ResolverExplanation`, and `VersionManifest` refs. |
| `070-AC-005` | Every identity split that affects gold or graph output emits `GraphCorrectionHandoff`. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `CanonicalEntity` | lines 2124-2155 |
| PRD-Cadastre.md | `SourceAsset` | lines 2156-2170 |
| PRD-Cadastre.md | `Identifier` | lines 2171-2213 |
| PRD-Cadastre.md | `IdentifierScope` | lines 2214-2273 |
| PRD-Cadastre.md | `IdentityDecision` | lines 2274-2338 |
| PRD-Cadastre.md | `IdentifierEvidenceClass` | lines 2339-2385 |
| PRD-Cadastre.md | `IdentityEvidenceItem` | lines 2386-2418 |
| PRD-Cadastre.md | `ResolverProfile` | lines 2419-2478 |
| PRD-Cadastre.md | `CandidateGenerationProfile` | lines 2479-2504 |
| PRD-Cadastre.md | `AssetGenerationBoundary` | lines 2505-2551 |
| PRD-Cadastre.md | `IdentityReviewCase` | lines 2552-2608 |
| PRD-Cadastre.md | `ResolverActivationReport` | lines 2609-2631 |
| PRD-Cadastre.md | `ResolverShadowRun` | lines 2632-2653 |
| PRD-Cadastre.md | `ResolverExplanation` | lines 2654-2692 |
| PRD-Cadastre.md | `GraphCorrectionHandoff` | lines 2693-2718 |
| PRD-Cadastre.md | `UnresolvedTargetReference` | lines 4672-4727 |
| PRD-Cadastre.md | `TargetSelectorSafetyPolicy` | lines 6562-6637 |
| PRD-Cadastre.md | `Identity Resolution Requirements` | lines 11032-11317 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
