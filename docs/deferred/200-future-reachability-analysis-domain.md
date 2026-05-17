---
doc_id: CADASTRE-DEFERRED-200
title: Future Reachability Analysis Domain
status: inactive_deferred
doc_type: deferred-nlspec-candidate
generated_on: 2026-05-17
source_prd: docs/archive/PRD-Cadastre.revised-draft.md
source_prd_sha256: 99437d5ec12d52752a0003577ac37f8a6c6f1221ac3ae3b7cce713b003aeae55
---

## Authority

This document is inactive. It preserves future reachability contract material and has no MVP implementation effect. No active Cadastre spec may emit theoretical reachability facts, graph edges, graph properties, or unqualified reachability claims from this document.

## Activation Rule

This document may become active only after a new accepted NLSpec promotion updates `docs/nlspec/000-cadastre-spec-index.md`, assigns authoritative owners, resolves all reachability open decisions, and adds passing validation fixtures in `docs/nlspec/120-validation-fixtures-and-acceptance.md`.

## Explicit Non-Scope While Deferred

- MVP graph projection.
- MVP `has_theoretical_reachability` output.
- Production solver selection.
- Provider analyzer compatibility claims.
- Negative reachability facts.
- Service access claims.
- Identity-conditioned access claims.

## Preserved Contract Candidates

| Candidate | Required future behavior |
| --- | --- |
| `NetworkContextSnapshot` | Must contain topology, routes, policies, translations, workloads, endpoint context, identity-access context, dataset refs, and snapshot hashes. |
| `ReachabilitySourceDatasetProfile` | Must define source-dataset coverage, authority, freshness, and evidence requirements by reachability evidence family. |
| `ReachabilityLakehouseFeedCompletenessProfile` | Must prevent partial evidence from becoming negative reachability. |
| `ReachabilityModelCapabilityMatrix` | Must map unsupported relevant components to `unsupported`, not `not_reachable`. |
| `ReachabilityQuery` | Must include claim kind, selectors, packet/session/service/identity constraints, time bounds, and query hash. |
| `ReachabilityResult` | Must use closed result states, limitation metadata, deterministic path ordering, and blocking components. |
| `ReachabilityAnalysisArtifact` | Must preserve provider analyzer output as analysis evidence only. |
| `ReachabilityClaimPolicy` | Must explicitly authorize any future fact, graph edge, property, or user-facing claim. |

## Claim Kinds

| Claim kind | Default graph effect |
| --- | --- |
| `packet_reachability` | `none` |
| `session_reachability` | `none` |
| `service_access` | `none` |
| `identity_conditioned_service_access` | `none` |

## Result States

| State | Meaning |
| --- | --- |
| `reachable_within_model` | Positive only within declared model scope. |
| `not_reachable_within_complete_model` | Negative only when evidence and solver capability are complete for the claim. |
| `unknown_partial` | Evidence missing, stale, partial, permission-limited, or outside completeness scope. |
| `unsupported` | Solver cannot represent a relevant component. |
| `ambiguous` | Multiple unresolved interpretations exist. |
| `conditional` | Result holds only under named conditions. |
| `diagnostic_only` | Provider or probe output cannot support claims. |

## Prohibited MVP Outputs

- `has_theoretical_reachability` edge.
- `modeled_reachability_fact`.
- Boolean reachability property.
- Unqualified `reachable` or `not reachable` wording.
- Any graph property that implies packet, session, service, or identity-conditioned access.

## Migration finalization contracts

### ReachabilityDeferredValidationPointer

`120` must include negative validation rows proving that active MVP specs fail or no-op for the following prohibited outputs.

| Prohibited output | Required active-spec validation pointer | Expected result |
| --- | --- | --- |
| `has_theoretical_reachability` edge | `090` and `120` negative graph projection row | fail or no-op |
| `modeled_reachability_fact` | `080`, `090`, and `120` negative derivation/projection row | fail or no-op |
| Boolean reachability property | `090`, `110`, and `120` API/projection row | fail or no-op |
| unqualified `reachable` or `not reachable` wording | `110` API wording row | reject wording or qualify as unsupported/deferred |

### Future activation owner-decision placeholders

| Decision area | Required future owner decision |
| --- | --- |
| source authority | Define reachability-specific source authority and absence authority rows. |
| completeness | Define topology, route, policy, NAT, workload, endpoint, and identity completeness dimensions. |
| solver capability | Define capability matrix and unsupported-component behavior. |
| result semantics | Define claim kinds, result states, conditions, limitations, and negative-claim scope. |
| graph effect | Define graph node, edge, property, traversal, and non-implication rows. |
| API wording | Define qualified user-facing language and forbidden claims. |
| validation fixtures | Define positive, negative, unknown, unsupported, conditional, and permission-limited fixtures. |

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `200-PATCH-AC-001` | The document remains `inactive_deferred`. |
| `200-PATCH-AC-002` | Active graph and gold specs contain prohibition validation rows. |
| `200-PATCH-AC-003` | Future activation still requires index promotion, owners, open-decision resolution, and passing validation. |

## Definition of Done for Future Activation

| ID | Criterion |
| --- | --- |
| `200-AC-001` | All reachability contracts have active owners and dependency declarations. |
| `200-AC-002` | Capability matrix fixtures cover unsupported, unknown partial, conditional, positive, and negative cases. |
| `200-AC-003` | User-facing wording distinguishes observed traffic, modeled-within-scope analysis, representative paths, diagnostic probes, unknown partial evidence, and unsupported model components. |
| `200-AC-004` | Graph and gold outputs remain disabled until `ReachabilityClaimPolicy` authorizes a narrower output and validation passes. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| docs/archive/PRD-Cadastre.revised-draft.md | `Future Non-MVP Reachability Analysis Domain` | lines 8752-9160 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Future Reachability Analysis Interface` | lines 10232-10279 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Future Reachability Configuration Defaults` | lines 10597-10619 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Future Reachability Boundary Acceptance` | lines 13506-13537 |
| Decomposition plan | Current user prompt | Deferred reachability target and activation boundary. |
