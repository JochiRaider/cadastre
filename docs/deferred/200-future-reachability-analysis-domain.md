---
doc_id: CADASTRE-DEFERRED-200
title: Future Reachability Analysis Domain
doc_type: deferred-nlspec
status: inactive_deferred
---

## Authority

This document is inactive. It preserves future reachability contract material and has no MVP implementation effect. No active Cadastre spec may emit theoretical reachability facts, graph edges, graph properties, or unqualified reachability claims from this document.

## Activation Rule

This document may become active only after a new accepted NLSpec promotion updates `docs/nlspec/000-cadastre-spec-index.md`, assigns authoritative owners, resolves all reachability open decisions, and adds passing validation fixtures in `docs/nlspec/120-validation-fixtures-and-acceptance.md`.

## Volatility rule while deferred

All current reachability content remains `inactive_future_domain` and has no MVP implementation effect.

Future stable reachability contracts must be promoted into active owner specs before implementation. Future solver profiles, reachability source dataset profiles, completeness profiles, capability matrices, query/result policies, claim policies, and fixtures must be separately classified by volatility class before activation.

Future graph or fact effects must require active `GraphProjectionProfile`, `SourceAuthorityProfile`, `TemporalSemanticsPolicy`, and `ReachabilityClaimPolicy` artifact refs where applicable.

This document must not define production activation behavior while deferred.

### Structured Input Repository Rule While Deferred

Future reachability structured inputs may be authored in a repository only as inactive future-domain material.

Repository merge, validation, materialization, package release creation, or package-set inclusion must not activate reachability while this document remains `inactive_deferred`.

Future activation must update `docs/nlspec/000-cadastre-spec-index.md`, assign active owner specs, classify volatility, add active owner contracts, add package and manifest gates when package-supplied, and add passing `120` fixtures before any reachability fact, graph edge, graph property, API output, package activation effect, or user-facing reachability claim can execute.

### FutureReachabilitySourceDatasetBlockHandoff

Any active `020.future_reachability` source-dataset row while this document remains `inactive_deferred` must be an exact deterministic block row. It must emit no production read target, fact, graph edge, graph property, API output, package activation effect, production validation pass for reachability, or unqualified reachability wording.

A `future_reachability` row may appear only as inactive future-domain material or validation-only negative fixture input. Repository merge, materialization, package release creation, package-set inclusion, graph profile activation, API wording, or validation report creation must not convert that row into runtime reachability behavior while this document remains deferred.

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

Expected active-spec failure codes:

| Code | Required use |
| --- | --- |
| `THEORETICAL_REACHABILITY_SCOPE_ERROR` | Active graph, gold, or analysis code attempts theoretical reachability behavior in MVP. |
| `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | Active spec attempts to emit a deferred reachability fact, edge, property, or API output. |
| `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` | API/UI/export wording uses unqualified reachable/not-reachable language. |

## Deferred Reachability Validation Contracts

### ReachabilityDeferredValidationPointer

`120` must include negative validation rows proving that active MVP specs fail or no-op for the following prohibited outputs.

| Prohibited output | Required active-spec validation row ID | Expected result |
| --- | --- | --- |
| `has_theoretical_reachability` edge | `val-090-theoretical-reachability-prohibited`; `fixture-090-theoretical-reachability-edge` | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op |
| `modeled_reachability_fact` | `val-080-modeled-reachability-fact-prohibited`; `fixture-200-active-reachability-output` | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` or no-op |
| Boolean reachability property | `val-090-boolean-reachability-property-prohibited`; `val-110-reachability-wording-prohibited` | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` |
| unqualified `reachable` or `not reachable` wording | `val-110-reachability-wording-prohibited` | `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` |

### DeferredStatusConsistency

| Condition | Required behavior |
| --- | --- |
| `000` status is `inactive_deferred` | This document remains non-runtime and active specs must reject prohibited outputs. |
| `000` status is not `inactive_deferred` | Activation rule in this document must be satisfied before any reachability runtime behavior can execute. |
| Any active spec emits prohibited MVP output | `ValidateSpecSet` fails before promotion. |

This document must not define production solver behavior, source authority, graph effect, or result semantics beyond preserved future candidates.

### Future Activation Required Decisions

| Decision area | Required future owner decision |
| --- | --- |
| source authority | Define reachability-specific source authority and absence authority rows. |
| completeness | Define topology, route, policy, NAT, workload, endpoint, and identity completeness dimensions. |
| solver capability | Define capability matrix and unsupported-component behavior. |
| result semantics | Define claim kinds, result states, conditions, limitations, and negative-claim scope. |
| graph effect | Define graph node, edge, property, traversal, and non-implication rows. |
| API wording | Define qualified user-facing language and forbidden claims. |
| validation fixtures | Define positive, negative, unknown, unsupported, conditional, and permission-limited fixtures. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `200-CLEANUP-AC-001` | No banned reference class remains. |
| `200-CLEANUP-AC-002` | The document remains `inactive_deferred`. |
| `200-CLEANUP-AC-003` | Active graph and gold specs still cannot emit theoretical reachability facts, graph edges, graph properties, or unqualified reachability claims from this document. |
| `200-CLEANUP-AC-004` | Future activation still requires accepted NLSpec promotion, assigned owners, resolved future activation decisions, and passing validation fixtures. |
| `200-VOLATILITY-AC-001` | Deferred reachability candidates remain inactive and cannot become production behavior without volatility classification, owner assignment, and validation rows. |

| `200-DEFERRED-CONSISTENCY-AC-001` | `ValidateSpecSet` fails when `000` status and this document's activation rule disagree, or when any active spec emits prohibited reachability output. |

### Structured input deferred reachability acceptance criteria

| ID | Criterion |
| --- | --- |
| `200-STRUCTURED-INPUT-DEFERRED-AC-001` | Repository-authored reachability artifacts produce no MVP fact, graph edge, graph property, package activation effect, solver selection, or unqualified reachability claim while this document remains `inactive_deferred`. |

## Definition of Done for Future Activation

| ID | Criterion |
| --- | --- |
| `200-AC-001` | All reachability contracts have active owners and dependency declarations. |
| `200-AC-002` | Capability matrix fixtures cover unsupported, unknown partial, conditional, positive, and negative cases. |
| `200-AC-003` | User-facing wording distinguishes observed traffic, modeled-within-scope analysis, representative paths, diagnostic probes, unknown partial evidence, and unsupported model components. |
| `200-AC-004` | Graph and gold outputs remain disabled until `ReachabilityClaimPolicy` authorizes a narrower output and validation passes. |
