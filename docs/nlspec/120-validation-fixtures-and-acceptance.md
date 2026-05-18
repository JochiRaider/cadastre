---
doc_id: CADASTRE-NLSPEC-120
title: Validation, Fixtures, and Acceptance
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define fixtures, validation matrices, golden corpus, shadow execution, replay validation, required negative tests, and binary acceptance reports for the NLSpec set.

## Explicit Non-Scope

- Owning domain behavior.
- New behavior not exported by a domain spec.
- Implementation-specific test harness mechanisms unless observable validation output is affected.

## Imports

- `All active domain specs`

- `All exported 040 core record schemas`
- `CoreRecordValidationAlgorithm`

## Exports

- `ValidationMatrix`
- `LakehouseFeedFixture`
- `ValidationScenario`
- `EventSequenceValidationCorpus`
- `GoldenCorpus`
- `ShadowExecutionReport`
- `ReplayValidationReport`
- `AcceptanceReport`
- `ValidateSpecSet`
- `RunValidationMatrix`
- `LifecycleValidationMatrix`
- `ValidationAcceptanceLifecycleMachine`

## Validation Ownership Rule

This spec verifies behavior owned by domain specs. It must not create new behavioral requirements unless the requirement is a validation interface, fixture record, acceptance report, or promotion gate.

## Validation Artifact Model

| Artifact | Required purpose |
| --- | --- |
| `ValidationMatrix` | Rows mapping contract, scenario, fixture, expected output, expected error/no-op, owner spec, and pass/fail evidence. |
| `LakehouseFeedFixture` | Redacted replayable feed rows, objects, manifests, supplier metadata, and expected read/import outcomes. |
| `ValidationScenario` | Validation-only scenario with given records, expected outputs, expected diagnostics, no-op assertions, and mutation prohibition. |
| `EventSequenceValidationCorpus` | Ordered event sequences for temporal, correction, replay, watermark, CDC, graph apply, rebuild, lifecycle transition, idempotency, no-op, rollback, quarantine, and validation acceptance behavior. |
| `GoldenCorpus` | Canonical set of input fixtures and expected outputs used for regression and promotion. |
| `AcceptanceReport` | Immutable report that records criteria pass/fail status, checksums, owner, environment, and timestamp. |

## Required Negative Test Classes

Every active domain spec must include at least one negative validation case for each forbidden authority boundary it declares. Required classes include:

| Boundary | Required negative case |
| --- | --- |
| Direct source calls | Production package attempts enterprise source API call and fails before output. |
| Feed completeness | Missing feed row attempts absence and fails. |
| Feed profile closure | Missing feed profile field, missing category closure row, unresolved profile branch, invalid empty-scope authorization, or missing subset profile fails before absence-sensitive effects. |
| Source authority | Missing exact authority row, ambiguous authority row, missing source-specific coverage row, missing staleness row, missing control-result mapping row, weak-signal combination, or checksum-mismatched closure row attempts output and fails or no-ops with no forbidden mutation. |
| Completeness effect gate | Missing completeness profile row, missing upstream evidence, omitted allowed effect, weak-signal combination, or completeness-blocked watermark fails or no-ops with no absence-sensitive effect. |
| Identity | Weak-only create/attach/merge attempts, selector-only attempts, source-native-merge-history-only attempts, candidate overflow auto-merge attempts, reviewer override of hard blocker, missing explanation, missing resolver row, ambiguous resolver row, and package-supplied weak-default override fail before identity mutation. |
| Graph | Backend internal ID appears in response or selector and fails; active MVP edge set is exactly `observed_connection`; missing or ambiguous `FlowRoleEvidence` emits no edge; OCSF endpoint order cannot determine direction; unresolved endpoint identity emits no edge; generic external graph payload is not pathfinding; theoretical reachability and boolean reachability properties are prohibited; query candidate limits and page tokens fail closed. |
| Package | Missing package-type policy, ambiguous package-type policy, unknown package type, unsupported repository form, unauthorized signer, missing transparency evidence, missing attestation, missing SBOM, dependency live resolution, compatibility failure, deprecation expiry, failed post-activation health gate, mutable rollback target, rollback compatibility failure, quarantine target block, emergency trust bypass, and package `VersionManifest` omission fail or no-op with no forbidden mutation. |
| Mapping | Missing mapping row, ambiguous mapping row, missing activity discriminator, unknown enum, forbidden OCSF field, IPAM/DHCP split, and undeclared source extension field fail before silver output. |
| Temporal | Missing temporal policy attempts current-time fallback and fails. |
| Analysis | Analysis rule attempts mutation and fails. |
| Reachability | MVP graph profile attempts `has_theoretical_reachability` and fails. |

## Required Volatility Boundary Test Classes

Validation must prove that volatile material cannot redefine stable behavior and cannot affect production output without activation refs.

| Volatility boundary | Required negative case | Expected result |
| --- | --- | --- |
| core/artifact boundary | Mapping bundle attempts to define a new core field. | Fails before persistence. |
| artifact activation | Inactive resolver profile, missing candidate generation profile, checksum-mismatched evidence class row set, out-of-scope split policy, omitted resolver explanation policy, or missing `VersionManifest` resolver artifact refs are referenced. | Fails before identity output or replay. |
| artifact checksum | Source-authority row checksum mismatch. | Fails before gold derivation. |
| artifact omission | Graph projection profile or OCSF mapping row set omitted from `VersionManifest`. | Fails before dependent output. |
| external schema non-authority | OCSF profile attempts to treat observables, status, severity, confidence, field absence, or endpoint order as authority. | Fails before authority or graph effect. |
| package/artifact mismatch | Active package set includes artifact checksum that does not match the artifact ref. | Package activation fails and current active set remains. |
| appendix/reference non-authority | Research or ADR text is referenced as runtime authority. | Spec-set validation fails. |
| stable core conflict | Activation artifact attempts to redefine identity, temporal, omission, or graph authority. | Fails with volatility-boundary error before production output. |
| runtime state substitution | `GraphRebuildManifest`, `LakehouseCommitRef`, lineage facet, or validation report is used as source truth. | Fails before authority effect. |

## Acceptance Aggregation

`RunValidationMatrix` must produce a deterministic `AcceptanceReport` containing row ID, owner spec, fixture checksum, input checksum, expected output checksum, actual output checksum, result, failure code, and version manifest ref.

## Spec Set Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `SPECSET-AC-001` | Every implementation-relevant contract has exactly one owning spec. |
| `SPECSET-AC-002` | Every active spec declares imports, exports, dependencies, explicit non-scope, and Definition of Done. |
| `SPECSET-AC-003` | No active NLSpec restates another active NLSpec requirement except by exact named reference. |
| `SPECSET-AC-004` | Every production input path remains lakehouse-fed and has no direct enterprise source-call behavior. |
| `SPECSET-AC-005` | Every domain spec has binary acceptance criteria and at least one negative validation case for each forbidden authority boundary it declares. |
| `SPECSET-AC-006` | The deferred reachability document is inactive and no active MVP graph or gold spec can emit theoretical reachability output. |
| `SPECSET-AC-007` | The recreatability test passes using only active NLSpecs, deferred NLSpecs where explicitly referenced, and accepted standards. |

| `SPECSET-AC-008` | Domain unresolved/resolved status and owner-local closure states are consistent. |
| `SPECSET-AC-009` | ADR statuses are members of `000.SpecStatus` and accepted ADRs use `accepted_rationale`. |
| `SPECSET-AC-010` | Every manifest path has exactly one registry row or explicit non-registry reason. |
| `SPECSET-AC-011` | Every implementation-relevant contract, profile, row set, package, fixture, registry object, and runtime state record has exactly one volatility class. |
| `SPECSET-AC-012` | No activation-controlled artifact can define new runtime behavior not exported by a stable core contract. |
| `SPECSET-AC-013` | Every output-affecting activation-controlled artifact is represented by `030.ActivationControlledArtifactRef` and appears in `VersionManifest`. |
| `SPECSET-AC-014` | Stable core specs contain no production-active volatile rows except non-normative examples or blocking `TODO:` rows. |
| `SPECSET-AC-015` | Every active absence-sensitive feed category has `SourceAuthorityClosureMatrix` validation rows or deterministic block rows. |

## Validation Contract Details

### CoreRecordSchemaValidationMatrix

| Row class | Required coverage |
| --- | --- |
| `040-schema-success` | Minimal valid record for every 040 exported record emits expected canonical bytes and checksum. |
| `040-required-field-rejection` | Missing required field fails with owner-specific missing-field code. |
| `040-null-rejection` | Explicit null in a non-nullable field fails with `CORE_NULL_FORBIDDEN`. |
| `040-unknown-field-rejection` | Unknown field outside extension maps fails with `CORE_UNKNOWN_FIELD`. |
| `040-default-materialization` | Required default `[]`, `{}`, enum, or null is materialized exactly as specified. |
| `040-array-sort` | Arrays with declared sort keys produce byte-identical output regardless of input order. |
| `040-id-collision` | Simulated collision fails with owner collision error and commits no record. |
| `040-checksum-replay` | Same record input produces same checksum across replay. |
| `040-raw-payload-boundary` | `EvidenceRef` and graph deltas cannot inline raw payload bytes. |
| `040-backend-id-rejection` | Backend-generated graph IDs fail. |
| `040-gold-fact-key-id-separation` | Comparable semantic fact key remains stable while immutable fact record ID changes across knowledge versions. |
| `040-two-independent-implementers` | Two implementations produce byte-identical IDs and checksums for every core record fixture. |
| `040-source-extension-rule-shape` | Valid, missing-field, unknown-field, bounds, non-canonical ordering, checksum replay, and pre-activation rejection fixtures for `SourceExtensionFieldRuleShape`. |

### CoreRecordSchema fixture families

| Fixture ID family | Records |
| --- | --- |
| `core-minimal-valid-*` | One minimal valid fixture per exported record. |
| `core-required-missing-*` | One missing-field fixture per record. |
| `core-null-forbidden-*` | One null-forbidden fixture per nullable or non-null boundary class. |
| `core-unknown-field-*` | Unknown field outside extension map. |
| `core-extension-map-*` | Declared extension allowed; undeclared extension rejected. |
| `core-array-sort-*` | Lexical and ID sort validation. |
| `core-checksum-*` | Checksum inclusion/exclusion validation. |
| `core-id-collision-*` | Collision simulation fixture. |
| `core-evidence-raw-payload-*` | Raw payload leak rejection. |
| `core-graph-backend-id-*` | Backend ID rejection. |
| `core-goldfact-correction-*` | Key ID versus immutable ID replay sequence. |

### ValidationMatrixRow schema

| Field | Required | Rule |
| --- | ---: | --- |
| `row_id` | Yes | Stable unique ID. |
| `owner_spec` | Yes | Exact owner spec ID. |
| `contract` | Yes | Named exported contract. |
| `scenario` | Yes | Success, rejection, no-op, edge, replay, redaction, authorization, or negative authority-boundary. |
| `fixture` | Yes | Fixture ID and checksum. |
| `inputs` | Yes | Canonical input refs and checksums. |
| `expected_outputs` | Required unless no-op/error expected | Canonical expected output checksum. |
| `expected_no_op` | Required when no-op expected | Boolean plus explanation. |
| `expected_error` | Required when error expected | Owner-specific code. |
| `mutation_prohibition` | Yes | Production mutation classes forbidden by the row. |
| `checksums` | Yes | Input, fixture, expected, and actual checksums. |
| `pass_fail_evidence` | Yes after run | `pass`, `fail`, `blocked`, or `not_run`. |
| `volatility_class` | No | Required when the row validates a volatility boundary. |
| `activation_artifact_refs` | No | Required refs and checksums for output-affecting activation-controlled artifacts. |
| `stable_core_owner` | No | Owner stable contract when validating activation artifacts. |
| `artifact_owner_spec` | No | Artifact owner spec when different from stable core owner. |
| `version_manifest_requirement` | No | Expected `VersionManifest` inclusion rule. |
| `artifact_mutation_prohibition` | No | Artifact classes and stable-core fields that the row forbids mutating. |

### OwnerLocalStatusConsistencyRule validation rows

| Row ID | Scenario | Expected failure code | Mutation prohibition |
| --- | --- | --- | --- |
| `domain-owner-status-raw-feed-manifest-closed` | Domain marks `RawFeedManifest` unresolved while `020` is `closed_local`. | `DOMAIN_OWNER_STATUS_CONTRADICTION` | no production output |
| `domain-owner-status-raw-record-id-closed` | Domain marks raw record ID unresolved while `040.ComputeRawRecordId` and `020` import contribution are `closed_local`. | `DOMAIN_OWNER_STATUS_CONTRADICTION` | no production output |
| `domain-runtime-restatement-raw-id-order` | Domain copies raw ID input order instead of routing to `040.ComputeRawRecordId`. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `owner-spec-contradiction-same-runtime-contract` | Two owner specs define incompatible runtime behavior for one named contract. | `OWNER_SPEC_CONTRADICTION` | no production output |
| `adr-status-unregistered` | ADR status is not in `000.SpecStatus`. | `ADR_STATUS_UNREGISTERED` | no promotion |
| `registry-manifest-path-mismatch` | Manifest path lacks exactly one registry row or explicit non-registry reason. | `REGISTRY_MANIFEST_PATH_MISMATCH` | no promotion |

### ValidationCoverageMatrix

| Owner spec | Required success | Required rejection | Required no-op | Required edge | Required replay | Required redaction | Required authorization | Required negative authority-boundary | Required volatility-boundary |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | authority class | direct source call | validation exploration no-op | private/public boundary | n/a | private leak | API/private boundary | direct source, projection authority | required when owner declares activation-controlled artifacts. |
| `020` | feed read/import and category closure | malformed manifest, profile schema incomplete, category row missing | partial no-absence, empty-scope no-absence | CDC metadata and closure branch behavior | raw ID replay | private binding | n/a | no direct source call and unresolved profile branch | required when owner declares activation-controlled artifacts. |
| `030` | DAG execution | forbidden output | safe subset no-op | lifecycle illegal transition | manifest replay | n/a | n/a | subset authority | required when owner declares activation-controlled artifacts. |
| `040` | canonical ID/checksum | unknown field | omission distinction | collision | checksum replay | redaction state | n/a | backend ID rejection | required when owner declares activation-controlled artifacts. |
| `050` | OCSF mapping row success by row family | missing row, ambiguous row, missing discriminator, forbidden field, deprecated field, unknown enum, IPAM split, artifact mismatch | cadastre-only null-profile no-op when row permits | unknown enum and mapping discriminator branches | mapping replay | source extension redaction | n/a | undeclared extension and source-extension wildcard | required when owner declares activation-controlled artifacts. |
| `060` | absence authorized and empty-complete authorized | missing authority row, ambiguous authority row, missing completeness row, missing upstream evidence, missing coverage row, missing staleness row, missing control-result mapping, and external-schema authority attempt | weak signal no-op, weak-signal combination blocked, effect-not-allowed no-op, OCSF non-authority no-op, DNS TTL no deletion, DHCP lease no host absence, and missing-flow unknown | blocking precedence, partial completeness, source-history outside-window, and directory visibility gaps | watermark replay, closure-manifest replay, and watermark blocked | n/a | n/a | missing coverage, weak-signal combination, source-history no-change, DNS/DHCP/flow negative non-authority, and OCSF status/severity/confidence non-authority | required when owner declares activation-controlled artifacts. |
| `070` | creation, attachment, exact durable merge, split, and no-decision | weak-only, under-scoped, blocker, missing row, ambiguous row, and overflow | selector-only and unsupported entity | split handoff | decision/explanation checksum replay | resolver private-binding redaction through `010`/`110` | reviewer authority | review cannot mutate identity without terminal decision | inactive/missing/checksum-mismatched/out-of-scope resolver artifacts and package core conflict. |
| `080` | fact derivation | temporal missing | duplicate correction no-op | late arrival | replay mismatch | n/a | n/a | no current-time fallback | required when owner declares activation-controlled artifacts. |
| `090` | graph query/apply and positive observed-connection projection | backend ID rejection, OCSF endpoint-order direction rejection, unresolved endpoint rejection, backend taxonomy unmapped rejection, query candidate-limit rejection, and page-token rejection | empty traversal no-path, missing-flow-role no-edge, ambiguous-flow-role no-edge, and generic external graph payload no-pathfinding | active profile exact edge set, edge semantics, output eligibility, and apply order | rebuild equivalence and graph profile replay | raw property redaction and backend ID redaction | graph auth and output eligibility | reachability prohibition, endpoint-order non-authority, selector-only endpoint non-authority, and backend-state non-authority | required when owner declares activation-controlled artifacts. |
| `100` | package type policy, supported repository forms, trust, transparency, attestation, provenance, SBOM, dependency lock, compatibility, deprecation, health, rollback, quarantine, emergency, and package manifest success | unknown type, missing policy, ambiguous policy, unsupported Git tree snapshot, unauthorized signer, threshold failure, missing transparency, missing attestation, missing SBOM, live dependency resolution, compatibility failure, expired deprecation, failed health gate, mutable rollback target, rollback incompatibility, quarantine block, emergency bypass, and package manifest omission | failed candidate keep-current, health-fail not-LKG, quarantine no mutation, and shadow/canary no current output | rollback preflight and emergency bounded actions | package-set replay and manifest completeness | package evidence redaction through `110` | promotion auth and emergency quorum | signature/trust bypass forbidden, dependency lock non-authority, and mutable Git ref non-authority | required when owner declares activation-controlled artifacts. |
| `110` | API success | stale compliance rejection | empty result | state label | page token replay | raw payload redaction | inaccessible asset | state-label non-collapse | required when owner declares activation-controlled artifacts. |
| `130` | analysis read-only | mutation rejection | risk scoring disabled | lineage facet | registry replay | lineage redaction | registry auth | analysis non-authority | required when owner declares activation-controlled artifacts. |
| `200` | n/a while deferred | reachability prohibited | no-op | no graph effect | n/a | n/a | n/a | deferred reachability | required when owner declares activation-controlled artifacts. |

### ObservationTypeExternalMappingValidationMatrix

Every active `ObservationToOCSFMappingRow` must have one validation row in this matrix for a positive mapping case and one or more negative rows for discriminator, enum, object-path, forbidden-field, source-extension, and non-authority boundaries that the row can encounter.

| Row field | Required content |
| --- | --- |
| `owner_spec` | `050` for mapping behavior; `060` or `090` when the row proves a non-authority or graph-direction boundary. |
| `contract` | `ResolveOCSFMapping`, `SourceExtensionFieldRuleSet`, `DeriveAbsenceOrUnknown`, or `ProjectGraphDeltas`. |
| `scenario` | `success`, `rejection`, `no-op`, `redaction`, `replay`, or `negative authority-boundary`. |
| `fixture` | Exact fixture ID and checksum. |
| `expected_outputs` | Expected silver output checksum for success; omitted for rejection/no-op. |
| `expected_error` | Exact owner error code for rejection. |
| `mutation_prohibition` | `no silver output` for mapping rejection; `no authoritative mutation` for non-authority; `no graph mutation` for direction rejection. |
| `version_manifest_refs` | Every artifact ref required by `030.VersionManifestCompletenessMatrix` for the scenario. |
| `acceptance_criterion` | Exact `050-OCSF-*`, `050-SOURCE-EXT-*`, `060-OCSF-*`, or `090-OCSF-*` criterion. |
| `blocking_status` | `blocking` until fixture checksum, expected output checksum or expected error, and version-manifest refs are filled. |

Required `050` fixture families:

```text
ocsf-mapping-row-success-*
ocsf-mapping-row-missing-*
ocsf-mapping-row-ambiguous-*
ocsf-activity-discriminator-missing-*
ocsf-required-object-path-missing-*
ocsf-forbidden-field-*
ocsf-unknown-enum-*
ocsf-other-enum-not-permitted-*
ocsf-deprecated-field-*
source-extension-rule-success-*
source-extension-rule-undeclared-*
source-extension-wildcard-rejected-*
source-extension-secret-scan-*
cadastre-only-null-profile-*
dhcp-ipam-split-required-*
```

Required `060` fixture families:

```text
ocsf-status-non-authority-*
ocsf-severity-non-authority-*
ocsf-confidence-non-authority-*
ocsf-observable-non-authority-*
ocsf-field-absence-non-authority-*
```

Required `090` fixture families:

```text
ocsf-endpoint-order-no-graph-direction-*
flow-role-evidence-required-*
```

### GraphActiveProfileClosureValidationMatrix

Rows in this matrix validate behavior owned by `090` and cross-owner handoffs required by `040`, `050`, `060`, `070`, `080`, `110`, and `130`. Because the uploaded patch plan does not provide canonical fixture bytes or expected output checksums, checksum cells are explicit blockers. `AcceptanceReport` must fail while any non-deferred graph closure row has a `TODO:` checksum, status `blocked`, status `not_run`, or missing mutation-prohibition proof.

| Row ID | Owner | Fixture ID | Fixture checksum | Expected result | Expected output checksum | Mutation prohibition |
| --- | --- | --- | --- | --- | --- | --- |
| `val-090-mvp-active-profile-exact-edge-set` | `090` | `fixture-090-mvp-active-profile-exact-edge-set` | TODO: required before acceptance | Only `observed_connection` active; all other edge types rejected. | TODO: required before acceptance | no unauthorized graph mutation |
| `val-090-observed-connection-positive` | `090` | `fixture-090-observed-connection-positive` | TODO: required before acceptance | One observed-connection edge plus resolved canonical endpoint nodes. | TODO: required before acceptance | no backend ID, no reachability output |
| `val-090-observed-connection-missing-flow-role` | `090` | `fixture-090-observed-connection-missing-flow-role` | TODO: required before acceptance | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op. | TODO: required before acceptance | no edge, no expiry, no absence |
| `val-090-observed-connection-ambiguous-flow-role` | `090` | `fixture-090-observed-connection-ambiguous-flow-role` | TODO: required before acceptance | ambiguous or no-op owner diagnostic. | TODO: required before acceptance | no graph mutation |
| `val-090-ocsf-endpoint-order-no-direction` | `090`, `050` | `fixture-090-ocsf-endpoint-order-no-direction` | TODO: required before acceptance | no observed-connection edge from OCSF endpoint order. | TODO: required before acceptance | no graph mutation |
| `val-090-unresolved-endpoint-no-edge` | `090`, `070` | `fixture-090-unresolved-endpoint-no-edge` | TODO: required before acceptance | `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. | TODO: required before acceptance | no graph node, no edge, no identity mutation |
| `val-090-stale-edge-not-pathfinding-default` | `090`, `060`, `110` | `fixture-090-stale-edge-not-pathfinding-default` | TODO: required before acceptance | stale display only when eligibility permits; no pathfinding by default. | TODO: required before acceptance | no expiry or cleanup |
| `val-090-empty-traversal-classes` | `090`, `110` | `fixture-090-empty-traversal-classes` | TODO: required before acceptance | empty path result. | TODO: required before acceptance | no backend traversal mutation |
| `val-090-observed-connection-path-explicit` | `090`, `110` | `fixture-090-observed-connection-path-explicit` | TODO: required before acceptance | path query traverses only `observed_connection_path`. | TODO: required before acceptance | no reachability claim |
| `val-090-theoretical-reachability-prohibited` | `090`, `200` | `fixture-090-theoretical-reachability-edge` | TODO: required before acceptance | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op. | TODO: required before acceptance | no graph/gold/API output |
| `val-090-boolean-reachability-property-prohibited` | `090`, `110`, `200` | `fixture-090-boolean-reachability-property` | TODO: required before acceptance | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`. | TODO: required before acceptance | no graph property, no API claim |
| `val-090-generic-external-payload-not-pathfinding` | `090`, `130` | `fixture-090-generic-external-payload-not-pathfinding` | TODO: required before acceptance | payload is non-emitting or detail-only and not pathfinding-ready. | TODO: required before acceptance | no finding, no metric, no identity influence |
| `val-090-backend-taxonomy-unmapped` | `090` | `fixture-090-backend-taxonomy-unmapped` | TODO: required before acceptance | unmapped backend label/type/property rejected. | TODO: required before acceptance | no serving output |
| `val-090-query-candidate-limit` | `090`, `110` | `fixture-090-query-candidate-limit` | TODO: required before acceptance | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`. | TODO: required before acceptance | no partial output unless explicitly permitted |
| `val-090-page-token-expired` | `090`, `110` | `fixture-090-page-token-expired` | TODO: required before acceptance | `GRAPH_PAGE_TOKEN_EXPIRED`. | TODO: required before acceptance | no backend query execution |
| `val-090-page-token-invalid` | `090`, `110` | `fixture-090-page-token-invalid` | TODO: required before acceptance | `GRAPH_PAGE_TOKEN_INVALID`. | TODO: required before acceptance | no backend query execution and no existence leak |
| `val-090-apply-order-replay` | `090`, `040` | `fixture-090-apply-order-replay` | TODO: required before acceptance | byte-identical canonical delta ordering. | TODO: required before acceptance | no nondeterministic apply |
| `val-090-partial-apply-no-derived-view` | `090` | `fixture-090-partial-apply-no-derived-view` | TODO: required before acceptance | partial apply records evidence and does not advance derived view. | TODO: required before acceptance | no derived-view advancement |
| `val-090-rebuild-equivalence` | `090`, `080` | `fixture-090-rebuild-equivalence` | TODO: required before acceptance | rebuild equivalence includes profile, schema, eligibility, taxonomy, query, property, index, and output checksums. | TODO: required before acceptance | no rebuild promotion on mismatch |
| `val-090-source-kind-cleanup-cross-source-reject` | `090`, `060` | `fixture-090-source-kind-cleanup-cross-source-reject` | TODO: required before acceptance | cross-source cleanup rejected unless exact `060` cleanup authorization exists. | TODO: required before acceptance | no cross-source deletion |
| `val-040-graph-node-source-object-ref-canonical-entity` | `040` | `fixture-040-graph-node-source-object-ref-canonical-entity` | TODO: required before acceptance | valid node source ref. | TODO: required before acceptance | no graph apply before validation |
| `val-040-graph-edge-source-object-ref-gold-fact` | `040` | `fixture-040-graph-edge-source-object-ref-gold-fact` | TODO: required before acceptance | valid edge source ref. | TODO: required before acceptance | no graph apply before validation |
| `val-050-flow-role-explicit-positive` | `050` | `fixture-050-flow-role-explicit-positive` | TODO: required before acceptance | explicit source direction emits `FlowRoleEvidence`. | TODO: required before acceptance | no graph mutation in mapping |
| `val-060-missing-flow-role-evidence-unknown` | `060` | `fixture-060-missing-flow-role-evidence-unknown` | TODO: required before acceptance | unknown/no-op. | TODO: required before acceptance | no absence, expiry, cleanup, retraction, or watermark |
| `val-070-graph-endpoint-requires-identity-decision` | `070` | `fixture-070-graph-endpoint-requires-identity-decision` | TODO: required before acceptance | graph endpoint requires identity decision. | TODO: required before acceptance | no graph endpoint from selector-only evidence |
| `val-080-graph-delta-replay-field-handoff` | `080` | `fixture-080-graph-delta-replay-field-handoff` | TODO: required before acceptance | replay includes imported `090` graph fields. | TODO: required before acceptance | no replay output on mismatch |
| `val-110-graph-response-profile-context` | `110` | `fixture-110-graph-response-profile-context` | TODO: required before acceptance | graph response contains profile context. | TODO: required before acceptance | no backend ID leak |
| `val-130-rule-graph-compat-observed-connection` | `130` | `fixture-130-rule-graph-compat-observed-connection` | TODO: required before acceptance | analysis read-only compatibility passes for named observed-connection query. | TODO: required before acceptance | no mutation |

### SourceExtensionFieldRuleShapeValidationRows

| Fixture family | Required coverage | Expected result |
| --- | --- | --- |
| `source-extension-shape-valid-*` | Minimal valid `040.SourceExtensionFieldRuleShape`. | Valid canonical bytes and checksum. |
| `source-extension-shape-missing-field-*` | Each required field omitted. | `CORE_REQUIRED_FIELD_MISSING`; no activation. |
| `source-extension-shape-unknown-field-*` | Unknown field outside the shape. | `CORE_UNKNOWN_FIELD`; no activation. |
| `source-extension-shape-noncanonical-*` | Non-canonical ordering or sort keys. | Checksum mismatch or canonicalization failure. |
| `source-extension-shape-invalid-bounds-*` | Bounds exceed `040` primitive limits. | `CORE_FIELD_BOUNDS_INVALID`; no activation. |
| `source-extension-shape-checksum-replay-*` | Same rule shape input replayed. | Byte-identical checksum. |

### FeedCategoryClosureValidationMatrix

Each fixture family in this matrix applies to every active feed category in `020.LakehouseFeedCategoryClosureRequirementTable`. `future_reachability` must use deterministic block/no-op fixtures for MVP instead of positive reachability output.

| Fixture family | Required coverage | Expected result |
| --- | --- | --- |
| `feed-category-valid-complete-*` | One complete positive case per non-blocked active feed category and requested effect. | Complete or empty-complete only when exact rows and refs permit. |
| `feed-category-missing-profile-row-*` | Category has no active `LakehouseFeedCompletenessProfileRow` or `LakehouseFeedCategoryClosureRow`. | Owner-specific missing-row error; no effect. |
| `feed-category-missing-upstream-completeness-*` | Required upstream evidence omitted, stale, insufficient, permission-limited, or scope-limited. | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `not_authoritative_for_absence`; no effect. |
| `feed-category-partial-known-gap-*` | Known missing object, partition, row range, or scope. | `partial_known_gap`; no absence, cleanup, retraction, graph expiry, or watermark. |
| `feed-category-partial-unknown-gap-*` | Missing scope cannot be identified. | `partial_unknown_gap`; no absence-sensitive effect. |
| `feed-category-permission-limited-*` | Supplier or source visibility is permission-limited. | `permission_limited`; no negative fact. |
| `feed-category-stale-*` | Source or evidence is stale under active staleness policy. | `source_stale` or blocking reason; no effect unless exact row permits. |
| `feed-category-empty-complete-*` | Empty scope or empty complete read with exact upstream evidence. | Authorized effect only through exact `060` row; otherwise empty-scope block. |
| `feed-category-weak-progress-*` | Cursor, ack, heartbeat, freshness, provenance, graph, or destination cleanup signal without active policy. | Weak-signal no-authority error or unknown no-op. |
| `feed-category-watermark-blocked-*` | Watermark requested while completeness gate blocks effect. | `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED`; no watermark advancement. |

| Feed category | Required fixture status |
| --- | --- |
| `endpoint_inventory` | Positive and negative closure fixtures required. |
| `configuration_inventory` | Positive and negative closure fixtures required. |
| `vulnerability_scan` | Positive and negative closure fixtures required, including scanner coverage dimensions. |
| `control_evaluation` | Positive and negative closure fixtures required, including control-result mapping. |
| `directory_inventory` | Positive and negative closure fixtures required, including hidden-object permission. |
| `directory_membership` | Positive and negative closure fixtures required, including direct/transitive mode and AD primary-group handling. |
| `dns_record_set` | Positive and negative closure fixtures required, including TTL non-authority. |
| `dhcp_ipam_assignment` | Positive and negative closure fixtures required, including lease expiry non-authority. |
| `network_flow` | Positive observed-flow and missing-flow-unknown fixtures required; absence fixture must block. |
| `cloud_asset_inventory` | Positive and negative closure fixtures required, including permission-limited inventory. |
| `source_history` | Positive and negative closure fixtures required, including outside-window no-proof. |
| `future_reachability` | MVP block/no-op fixture required; positive reachability output forbidden. |

### TemporalCorrectionReplayValidationMatrix

Rows in this matrix are required when `080` behavior is in implementation scope. Each row must name fixture checksum, input checksum, expected output checksum or no-op, expected error when applicable, mutation-prohibition proof, and `VersionManifest` ref.

| Fixture family | Required coverage |
| --- | --- |
| `temporal-resolution-*` | exact policy row, missing policy, absent time, malformed time, ambiguous time, unauthorized source time, authorized fallback, forbidden current-time fallback, manifest omission. |
| `knowledge-time-import-*` | `current_import`, historical valid-time-only import, reconstruction success with persisted source-known-time evidence, reconstruction rejected without evidence. |
| `late-arrival-*` | on-time authoritative, late authoritative correction, late non-authoritative quarantine, discard forbidden, watermark no-effect default. |
| `gold-correction-*` | replacement, retraction, interval split, stale state, conflict mark, conflict resolution, duplicate no-op, confidence-only same value, confidence-only changed value. |
| `assertion-transition-*` | Every default assertion-state and correction-event pair defined by `080`. |
| `correction-snapshot-ref-*` | old snapshot missing, new snapshot missing, table-set checksum missing, retention ineligible, mutable ref rejected. |
| `gold-correction-no-op-error-*` | missing temporal resolution, missing authority, ambiguous authority, missing correction policy, missing transition row, unauthorized negative, unauthorized CDC tombstone, replay output-class missing. |
| `replay-equivalence-*` | output-class rows for raw, silver, identity, gold, gold correction, graph delta, graph apply, graph rebuild, API response, export projection, analysis output, and validation acceptance; checksum match and mismatch. |
| `graph-handoff-*` | `none`, `reproject_fact_key`, authorized expiry, authorized cleanup, denied expiry, conflict visibility default, identity split handoff. |
| `temporal-version-manifest-*` | missing temporal resolution ref, missing correction snapshot refs, missing replay sufficiency check, missing graph handoff refs, checksum mismatch. |

| Validation row | Owner | Required assertion |
| --- | --- | --- |
| `120-TEMPORAL-CORRECTION-MISSING-POLICY` | `080` | Missing temporal policy emits `TEMPORAL_POLICY_UNRESOLVED` and no gold output. |
| `120-TEMPORAL-CORRECTION-CURRENT-TIME` | `080` | Current platform time fallback is rejected. |
| `120-ASSERTION-TRANSITION-TOTALITY` | `080` | Every assertion-state/correction-event pair has one expected result. |
| `120-REPLAY-OUTPUT-CLASS-COVERAGE` | `080` | Replay rows exist for every required output class. |
| `120-GRAPH-HANDOFF-COVERAGE` | `080`, `090` | Every graph handoff effect has success, error, or no-op validation. |
| `120-NOOP-ERROR-COVERAGE` | `080`, `060`, `090` | No-op and deterministic error cases prove no forbidden mutation. |
| `120-PROJECTION-REPLAY-COVERAGE` | `050`, `080` | Projection replay exact match, profile mismatch, loss-manifest mismatch, redaction mismatch, and volatile-field-only difference are validated. |
| `120-ANALYSIS-REPLAY-COVERAGE` | `130`, `080` | Analysis replay exact match, rule mismatch, graph compatibility mismatch, derived-view mismatch, authorization mismatch, shadow-only result, and mutation attempt are validated. |
| `120-API-TEMPORAL-ERROR-COVERAGE` | `110`, `080` | Every new `080` error is registered and label behavior is non-collapsing. |

### SourceAuthorityClosureValidationMatrix

Every fixture family in this matrix is required before `SourceAuthorityClosureMatrix` may satisfy promotion for an active absence-sensitive feed category. `120` validates owner behavior; it must not define source-authority behavior beyond validation artifact shapes, fixture IDs, expected outputs, and acceptance aggregation.

| Fixture family | Required cases |
| --- | --- |
| `source-authority-row-resolution-*` | exact match success, missing row, ambiguous row, dataset-default allowed, dataset-default forbidden, source-instance override exact match, source-instance mismatch |
| `lakehouse-feed-completeness-totality-*` | all six `020` receipt states crossed with all eight upstream evidence states for at least one absence-sensitive category |
| `coverage-dimension-*` | missing source-specific coverage row, missing required dimension, permission-limited dimension, partial known gap, stale coverage, covered success |
| `staleness-policy-*` | declared time precedence, missing time input, malformed time input, TTL expiry, DHCP lease expiry, source-history outside-window, no current-time fallback |
| `control-result-mapping-*` | pass, fail, unknown, error, not evaluated, not checked, not applicable, unmapped external state |
| `progress-signal-*` | cursor exhaustion alone, ack success alone, queue drain alone, CDC heartbeat alone, graph apply success alone, destination cleanup alone, weak-signal combination blocked, exact combined policy success |
| `dns-dhcp-flow-*` | DNS TTL expiry not deletion, DHCP lease expiry not host absence, missing flow unknown |
| `directory-visibility-*` | hidden membership, limited-information rows, direct-only membership query, AD primary-group gap, delta reset, page incompletion |
| `manifest-closure-*` | omitted authority row set ref, omitted staleness ref, omitted coverage ref, omitted progress policy, omitted control mapping, omitted source-history retention, checksum mismatch |
| `graph-expiry-*` | expiry authorized success, expiry missing `060` effect ref rejected, derived-view stale does not expire graph object |

A validation row in this matrix must include fixture checksum, expected `AbsenceDerivationResult` checksum when applicable, expected `VersionManifest` checksum or expected manifest error, expected output checksum when output is allowed, expected no-op when output is blocked, expected error code when rejected, and mutation-prohibition proof for raw, silver, identity, gold, graph, watermark, compliance export, and API label mutation classes affected by the case.

### PackageActivationValidationMatrix

Every row in this matrix validates behavior owned by `100`, manifest inclusion owned by `030`, observable error handling owned by `110`, or acceptance aggregation owned by `120`. Rows with `TODO` fixture checksums or expected output checksums are blocking and must prevent authoritative package activation handoff.

| Row class | Required coverage |
| --- | --- |
| `100-package-type-policy` | Unknown, missing, ambiguous, inactive, scope-mismatched, and manifest-omitted package type policy rows. |
| `100-repository-form` | Supported repository forms and `git_tree_snapshot` rejection. |
| `100-trust` | Unauthorized signer, inactive trust root, threshold failure, transparency evidence missing. |
| `100-attestation-sbom` | Missing, mismatched, expired, and policy-failed attestation and SBOM refs. |
| `100-dependency-lock` | Missing lock, checksum mismatch, live dependency resolution attempt. |
| `100-compatibility` | Missing, failed, expired, and mismatched compatibility rows across required axes. |
| `100-deprecation` | In-window use, expired use, emergency extension, and forbidden new activation. |
| `100-health-lkg` | Health pass marks LKG, health fail does not. |
| `100-rollback` | Mutable target rejection, immutable target pass, incompatible state/schema/graph/trust/replay rejection. |
| `100-quarantine` | Artifact, release, set, signer, trust root, snapshot, namespace, and package type quarantine blocking. |
| `100-emergency` | Allowed bounded override actions and forbidden trust bypass. |
| `100-version-manifest` | Omitted package activation refs reject output. |

A `PackageActivationValidationMatrix` row must include fixture checksum, expected output checksum or expected no-op, expected error when rejected, mutation prohibition, owner spec, acceptance criterion, and `VersionManifest` requirement. The row must identify the package activation ref classes required by `030.VersionManifestCompletenessMatrix` and the observable error mapping required by `110.PackageActivationErrorObservableMapping`.

Required `100` fixture rows:

| Row ID | Fixture ID | Fixture checksum | Expected error or no-op | Expected output checksum | Mutation prohibition | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `val-100-package-type-unknown` | `fixture-100-package-type-unknown` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-002` | blocking |
| `val-100-package-type-policy-missing` | `fixture-100-package-type-policy-missing` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `val-100-package-type-policy-ambiguous` | `fixture-100-package-type-policy-ambiguous` | TODO | `PACKAGE_TYPE_POLICY_AMBIGUOUS` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-004` | blocking |
| `val-100-git-tree-snapshot-unsupported` | `fixture-100-git-tree-snapshot-unsupported` | TODO | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | TODO | no candidate production output | `100-REPOSITORY-FORM-AC-002` | blocking |
| `val-100-package-set-checksum-mismatch` | `fixture-100-package-set-checksum-mismatch` | TODO | `PACKAGE_SET_CHECKSUM_MISMATCH` | TODO | current active set unchanged | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `val-100-unauthorized-signer` | `fixture-100-unauthorized-signer` | TODO | `PACKAGE_SIGNER_UNAUTHORIZED` | TODO | no package activation | `100-CLEANUP-AC-003` | blocking |
| `val-100-transparency-evidence-missing` | `fixture-100-transparency-evidence-missing` | TODO | `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | TODO | no package activation | `100-TRANSPARENCY-AC-001` | blocking |
| `val-100-attestation-missing` | `fixture-100-attestation-missing` | TODO | `PACKAGE_ATTESTATION_MISSING` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `val-100-attestation-subject-mismatch` | `fixture-100-attestation-subject-mismatch` | TODO | `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `val-100-sbom-missing` | `fixture-100-sbom-missing` | TODO | `PACKAGE_SBOM_MISSING` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `val-100-sbom-subject-mismatch` | `fixture-100-sbom-subject-mismatch` | TODO | `PACKAGE_SBOM_SUBJECT_MISMATCH` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `val-100-dependency-lock-missing` | `fixture-100-dependency-lock-missing` | TODO | `PACKAGE_DEPENDENCY_LOCK_MISSING` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `val-100-dependency-live-resolution-forbidden` | `fixture-100-dependency-live-resolution-forbidden` | TODO | `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `val-100-compatibility-failed` | `fixture-100-compatibility-failed` | TODO | `PACKAGE_COMPATIBILITY_FAILED` | TODO | no package activation | `100-COMPATIBILITY-AC-001` | blocking |
| `val-100-deprecation-window-expired` | `fixture-100-deprecation-window-expired` | TODO | `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | TODO | no new activation | `100-DEPRECATION-WINDOW-AC-002` | blocking |
| `val-100-lkg-health-gate-failed` | `fixture-100-lkg-health-gate-failed` | TODO | `PACKAGE_LKG_HEALTH_GATE_FAILED` | TODO | no last-known-good mark | `100-LKG-HEALTH-AC-002` | blocking |
| `val-100-mutable-rollback-target` | `fixture-100-mutable-rollback-target` | TODO | `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | TODO | no package activation | `100-ROLLBACK-AC-001` | blocking |
| `val-100-rollback-state-incompatible` | `fixture-100-rollback-state-incompatible` | TODO | `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `val-100-rollback-graph-incompatible` | `fixture-100-rollback-graph-incompatible` | TODO | `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `val-100-quarantine-artifact-blocks-release` | `fixture-100-quarantine-artifact-blocks-release` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent activation | `100-QUARANTINE-AC-001` | blocking |
| `val-100-quarantine-signer-blocks-activation` | `fixture-100-quarantine-signer-blocks-activation` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent activation | `100-QUARANTINE-AC-001` | blocking |
| `val-100-quarantine-clear-not-active` | `fixture-100-quarantine-clear-not-active` | TODO | no-op | TODO | no direct active transition | `100-QUARANTINE-AC-002` | blocking |
| `val-100-emergency-bypass-forbidden` | `fixture-100-emergency-bypass-forbidden` | TODO | `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | TODO | no package activation | `100-EMERGENCY-AC-001` | blocking |
| `val-100-version-manifest-package-refs-missing` | `fixture-100-version-manifest-package-refs-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `100-VERSION-MANIFEST-AC-001` | blocking |
| `val-100-canary-output-isolation` | `fixture-100-canary-output-isolation` | TODO | `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | TODO | no current production output | `100-LIFECYCLE-AC-003` | blocking |
| `val-100-shadow-output-isolation` | `fixture-100-shadow-output-isolation` | TODO | `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | TODO | no current production output | `100-LIFECYCLE-AC-003` | blocking |

### TwoIndependentImplementersCheck

| Field | Required content |
| --- | --- |
| comparison scope | Every authoritative owner contract and validation row in the spec set. |
| allowed variance | Non-observable implementation structure only. |
| output equivalence | Byte-identical canonical records or deterministic equivalent observable outputs. |
| failure classification | spec ambiguity, implementation defect, fixture defect, or deferred issue. |
| acceptance threshold | All non-deferred authoritative rows pass. |
| core record fixture coverage | Raw, silver, canonical entity, source asset, identifier, gold, evidence, and graph delta fixtures exported by `040`. |

### RecreatabilityCheck

The recreatability check passes only when the intended behavior can be derived from active NLSpecs, accepted standards, and explicitly deferred documents where referenced.

Any required lookup outside those inputs is a failure unless the lookup is trace-only and does not determine behavior.

### AcceptanceReport statuses

| Status | Meaning | Failure precedence |
| --- | --- | --- |
| `fail` | Row ran and produced mismatched output, forbidden mutation, or wrong error. | Highest. |
| `blocked` | Row cannot run because required artifact, owner decision, fixture, or dependency is missing. | Second. |
| `not_run` | Row exists but has no execution evidence. | Third. |
| `pass` | Row ran and all expected evidence matched. | Lowest. |

Lifecycle transition evidence is required for acceptance status changes that affect promotion. A promotion-affecting `AcceptanceReport` status change without matching `120.ValidationAcceptanceLifecycleMachine.v1` transition evidence is invalid and must fail before promotion.

### Required negative tests by owner

| Owner spec | Forbidden boundary | Fixture ID | Fixture checksum | Expected error/no-op | Expected output checksum | Mutation prohibition | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | direct source call | `fixture-010-direct-source-call` | TODO | `DIRECT_SOURCE_CALL_FORBIDDEN` | TODO | no production output | `010-CLEANUP-AC-002` | blocking |
| `010` | private binding leak | `fixture-010-private-binding-leak` | TODO | `PRIVATE_BINDING_LEAK` | TODO | no public artifact write | `010-CLEANUP-AC-003` | blocking |
| `010` | projection authority violation | `fixture-010-projection-authority-violation` | TODO | `PROJECTION_AUTHORITY_VIOLATION` | TODO | no authoritative mutation | `010-CLEANUP-AC-004` | blocking |
| `020` | CDC metadata as authority | `fixture-020-cdc-metadata-non-authority` | TODO | no-op | TODO | no absence, no retraction, no watermark | `020-CLEANUP-AC-004` | blocking |
| `020` | malformed feed manifest | `feed-020-malformed-manifest` | TODO | `manifest_invalid` | TODO | no raw import commit | `020-MANIFEST-AC-001` | blocking |
| `020` | missing feed profile field | `feed-020-profile-schema-incomplete` | TODO | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | TODO | no feed read | `020-FEED-CLOSURE-AC-001` | blocking |
| `020` | missing category closure row | `feed-020-category-row-missing` | TODO | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | TODO | no feed read | `020-FEED-CLOSURE-AC-002` | blocking |
| `020` | unresolved profile branch | `feed-020-profile-branch-unresolved` | TODO | `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | TODO | no production output | `020-FEED-CLOSURE-AC-003` | blocking |
| `020` | invalid empty-scope authorization | `feed-020-empty-scope-not-authoritative` | TODO | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | TODO | no absence and no watermark | `020-FEED-CLOSURE-AC-006` | blocking |
| `020` | declared subset missing | `feed-020-declared-subset-required` | TODO | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | TODO | no subset output | `020-FEED-CLOSURE-AC-005` | blocking |
| `030` | forbidden stage output | `fixture-030-forbidden-stage-output` | TODO | `FORBIDDEN_STAGE_OUTPUT` | TODO | no production output | `030-CLEANUP-AC-003` | blocking |
| `030` | duplicate stage ID | `fixture-030-duplicate-stage-id` | TODO | `DAG_STAGE_ID_DUPLICATE` | TODO | no production output | `030-DAG-ORDER-AC-001` | blocking |
| `030` | subset output forbidden | `fixture-030-subset-output-forbidden` | TODO | `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | TODO | no production output and no watermark | `030-FEED-CLOSURE-AC-002` | blocking |
| `040` | unknown field | `fixture-040-unknown-field` | TODO | `CORE_UNKNOWN_FIELD` | TODO | no record commit | `040-CLEANUP-AC-002` | blocking |
| `040` | one-of invalid | `fixture-040-one-of-invalid` | TODO | `CORE_ONE_OF_INVALID` | TODO | no record commit | `120-SCHEMA-PATCH-AC-001` | blocking |
| `050` | undeclared extension | `fixture-050-source-extension-undeclared` | TODO | `SOURCE_EXTENSION_FIELD_UNDECLARED` | TODO | no silver output | `050-CLEANUP-AC-002` | blocking |
| `050` | OCSF artifact mismatch | `fixture-050-ocsf-artifact-mismatch` | TODO | `OCSF_ARTIFACT_MISMATCH` | TODO | no normalized output | `050-OCSF-BASELINE-AC-001` | blocking |
| `050` | missing OCSF mapping row | `ocsf-mapping-row-missing-*` | TODO | `MAP_OCSF_ROW_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-001` | blocking |
| `050` | ambiguous OCSF mapping row | `ocsf-mapping-row-ambiguous-*` | TODO | `MAP_OCSF_ROW_AMBIGUOUS` | TODO | no silver output | `050-OCSF-MAP-AC-001` | blocking |
| `050` | missing OCSF activity discriminator | `ocsf-activity-discriminator-missing-*` | TODO | `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-005` | blocking |
| `050` | OCSF required object path missing | `ocsf-required-object-path-missing-*` | TODO | `OCSF_REQUIRED_OBJECT_PATH_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-002` | blocking |
| `050` | OCSF forbidden field emitted | `ocsf-forbidden-field-*` | TODO | `OCSF_FORBIDDEN_FIELD_EMITTED` | TODO | no silver output | `050-BASE-FIELD-AC-001` | blocking |
| `050` | DHCP/IPAM split required | `dhcp-ipam-split-required-*` | TODO | `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED` | TODO | no silver output | `050-OCSF-MAP-AC-006` | blocking |
| `050` | source-extension wildcard rejected | `source-extension-wildcard-rejected-*` | TODO | `SOURCE_EXTENSION_NAMESPACE_INVALID` | TODO | no silver output | `050-SOURCE-EXT-AC-003` | blocking |
| `050` | source-extension secret scan | `source-extension-secret-scan-*` | TODO | `SOURCE_EXTENSION_SECRET_SCAN_FAILED` | TODO | no caller-visible secret output | `050-SOURCE-EXTENSION-AC-001` | blocking |
| `060` | weak progress absence | `fixture-060-weak-progress-absence` | TODO | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` or unknown no-op | TODO | no absence, no cleanup, no watermark | `060-CLEANUP-AC-003` | blocking |
| `060` | missing coverage | `fixture-060-coverage-required` | TODO | `COVERAGE_ASSERTION_REQUIRED` | TODO | no coverage-sensitive fact | `060-COVERAGE-CLOSURE-AC-001` | blocking |
| `060` | missing completeness profile row | `feed-category-missing-profile-row-*` | TODO | `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | TODO | no absence-sensitive effect | `060-FEED-CLOSURE-AC-001` | blocking |
| `060` | missing upstream completeness evidence | `feed-category-missing-upstream-completeness-*` | TODO | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | TODO | no absence-sensitive effect | `060-FEED-CLOSURE-AC-002` | blocking |
| `060` | effect not allowed | `fixture-060-effect-not-allowed` | TODO | `COMPLETENESS_EFFECT_NOT_ALLOWED` | TODO | no absence, cleanup, retraction, graph expiry, or watermark | `060-FEED-CLOSURE-AC-003` | blocking |
| `060` | weak-signal combination | `feed-category-weak-progress-*` | TODO | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` or unknown no-op | TODO | no stronger authority | `060-CLEANUP-AC-003` | blocking |
| `060` | watermark blocked by completeness | `feed-category-watermark-blocked-*` | TODO | `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | TODO | no watermark advancement | `060-FEED-CLOSURE-AC-004` | blocking |
| `060` | OCSF status as authority | `ocsf-status-non-authority-*` | TODO | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | TODO | no authoritative mutation | `060-OCSF-NONAUTH-AC-001` | blocking |
| `060` | OCSF field absence as absence | `ocsf-field-absence-non-authority-*` | TODO | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | TODO | no absence, cleanup, retraction, graph expiry, or watermark | `060-OCSF-NONAUTH-AC-003` | blocking |
| `070` | weak evidence auto-merge | `fixture-070-weak-evidence-auto-merge` | TODO | `no_decision` | TODO | no identity mutation | `070-CLEANUP-AC-002` | blocking |
| `070` | invalid review transition | `fixture-070-invalid-review-transition` | TODO | `IDENTITY_REVIEW_TRANSITION_INVALID` | TODO | no identity mutation | `070-REVIEW-TOTALITY-AC-001` | blocking |
| `080` | missing temporal policy | `temporal-resolution-missing-policy` | TODO | `TEMPORAL_POLICY_UNRESOLVED` | TODO | no gold fact | `080-TEMPORAL-POLICY-AC-001` | blocking |
| `080` | absent source time | `temporal-resolution-absent-time` | TODO | temporal error or unknown per active row | TODO | no unauthorized fallback | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | malformed source time | `temporal-resolution-malformed-time` | TODO | temporal error | TODO | no gold fact | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | ambiguous source time | `temporal-resolution-ambiguous-time` | TODO | temporal error | TODO | no gold fact | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | forbidden current-time fallback | `fixture-080-current-time-fallback` | TODO | `SOURCE_TIME_NOT_AUTHORIZED` or temporal error | TODO | no gold fact | `080-CLEANUP-AC-003` | blocking |
| `080` | historical known-time reconstruction rejected | `knowledge-time-import-reconstruction-rejected` | TODO | rejected reconstruction | TODO | no historical known-time claim | `080-KNOWLEDGE-TIME-AC-001` | blocking |
| `080` | missing late-arrival row | `late-arrival-policy-missing` | TODO | `LATE_ARRIVAL_POLICY_MISSING` | TODO | no correction route | `080-LATE-ARRIVAL-AC-001` | blocking |
| `080` | late discard forbidden | `late-arrival-discard-forbidden` | TODO | `LATE_ARRIVAL_DISCARD_FORBIDDEN` | TODO | evidence preserved | `080-LATE-ARRIVAL-AC-001` | blocking |
| `080` | missing correction policy | `gold-correction-policy-missing` | TODO | `CORRECTION_POLICY_MISSING` | TODO | no correction | `080-CORRECTION-MATRIX-AC-001` | blocking |
| `080` | missing correction transition | `assertion-transition-missing` | TODO | `GOLD_CORRECTION_TRANSITION_UNDEFINED` | TODO | no correction or graph handoff | `080-CORRECTION-MATRIX-AC-001` | blocking |
| `080` | missing old snapshot | `correction-snapshot-old-missing` | TODO | `CORRECTION_SNAPSHOT_REF_MISSING` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | missing new snapshot | `correction-snapshot-new-missing` | TODO | `CORRECTION_SNAPSHOT_REF_MISSING` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | mutable snapshot ref | `correction-snapshot-mutable-ref` | TODO | `MUTABLE_BRANCH_REF_FOR_REPLAY` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | duplicate candidate no-op | `gold-correction-duplicate-no-op` | TODO | `no_op_duplicate` | TODO | no graph mutation | `080-CORRECTION-NOOP-AC-001` | blocking |
| `080` | unauthorized retraction | `gold-correction-unauthorized-retraction` | TODO | owner `060` blocking reason | TODO | no retraction, no graph expiry, no watermark | `080-UNSAFE-NEGATIVE-AC-001` | blocking |
| `080` | unauthorized CDC tombstone | `gold-correction-unauthorized-cdc-tombstone` | TODO | `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | TODO | no retraction, cleanup, expiry, or watermark | `080-UNSAFE-NEGATIVE-AC-001` | blocking |
| `080` | confidence-only same value no-op | `gold-correction-confidence-same-no-op` | TODO | `no_op_duplicate` | TODO | no new active fact and no graph delta | `080-CORRECTION-NOOP-AC-001` | blocking |
| `080` | confidence-only changed value | `gold-correction-confidence-changed` | TODO | `insert_fact` or policy-defined update | TODO | append-only change set only | `040-CONFIDENCE-CANONICAL-AC-001` | blocking |
| `080` | missing replay output-class row | `replay-output-class-row-missing` | TODO | `REPLAY_POLICY_ARTIFACT_MISSING` | TODO | no replay output | `080-REPLAY-OUTPUT-CLASS-AC-001` | blocking |
| `080` | replay mutable ref | `replay-mutable-ref` | TODO | `MUTABLE_BRANCH_REF_FOR_REPLAY` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay retention failure | `replay-retention-ineligible` | TODO | `REPLAY_INPUT_INSUFFICIENT` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay authority mismatch | `replay-authority-mismatch` | TODO | `REPLAY_INPUT_INSUFFICIENT` or `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay temporal mismatch | `replay-temporal-mismatch` | TODO | `REPLAY_INPUT_INSUFFICIENT` or `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay side-effect mismatch | `replay-side-effect-mismatch` | TODO | `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay checksum mismatch | `fixture-080-replay-mismatch` | TODO | `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-CLEANUP-AC-004` | blocking |
| `090` | theoretical reachability edge | `fixture-090-theoretical-reachability-edge` | TODO | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op | TODO | no graph mutation | `090-CLEANUP-AC-004` | blocking |
| `090` | backend internal ID | `fixture-090-backend-id` | TODO | `GRAPH_BACKEND_ID_FORBIDDEN` | TODO | no graph response identity leak | `090-CLEANUP-AC-003` | blocking |
| `090` | graph apply unsafe resume | `fixture-090-lifecycle-graph-apply-resume-unsafe` | TODO | `GRAPH_APPLY_RESUME_UNSAFE` | TODO | no graph mutation and no derived-view advancement | `090-LIFECYCLE-AC-003` | blocking |
| `090` | graph apply identical reapply | `fixture-090-lifecycle-identical-reapply` | TODO | idempotent no-op | TODO | no duplicate graph object | `090-LIFECYCLE-AC-002` | blocking |
| `090` | OCSF endpoint order no graph direction | `ocsf-endpoint-order-no-graph-direction-*` | TODO | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op | TODO | no graph mutation | `090-OCSF-DIRECTION-AC-001` | blocking |
| `090` | missing flow-role evidence | `flow-role-evidence-required-*` | TODO | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op | TODO | no graph mutation | `090-OCSF-DIRECTION-AC-002` | blocking |
| `100` | package type unknown | `fixture-100-package-type-unknown` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-002` | blocking |
| `100` | package type policy missing | `fixture-100-package-type-policy-missing` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `100` | package type policy ambiguous | `fixture-100-package-type-policy-ambiguous` | TODO | `PACKAGE_TYPE_POLICY_AMBIGUOUS` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-004` | blocking |
| `100` | unsupported Git tree snapshot | `fixture-100-git-tree-snapshot-unsupported` | TODO | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | TODO | no candidate production output | `100-REPOSITORY-FORM-AC-002` | blocking |
| `100` | unauthorized signer | `fixture-100-unauthorized-signer` | TODO | `PACKAGE_SIGNER_UNAUTHORIZED` | TODO | no package activation | `100-CLEANUP-AC-003` | blocking |
| `100` | missing transparency evidence | `fixture-100-transparency-evidence-missing` | TODO | `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | TODO | no package activation | `100-TRANSPARENCY-AC-001` | blocking |
| `100` | missing attestation | `fixture-100-attestation-missing` | TODO | `PACKAGE_ATTESTATION_MISSING` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `100` | missing SBOM | `fixture-100-sbom-missing` | TODO | `PACKAGE_SBOM_MISSING` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `100` | dependency live resolution | `fixture-100-dependency-live-resolution-forbidden` | TODO | `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `100` | compatibility failed | `fixture-100-compatibility-failed` | TODO | `PACKAGE_COMPATIBILITY_FAILED` | TODO | no package activation | `100-COMPATIBILITY-AC-001` | blocking |
| `100` | deprecation window expired | `fixture-100-deprecation-window-expired` | TODO | `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | TODO | no new activation | `100-DEPRECATION-WINDOW-AC-002` | blocking |
| `100` | LKG health gate failed | `fixture-100-lkg-health-gate-failed` | TODO | `PACKAGE_LKG_HEALTH_GATE_FAILED` | TODO | no last-known-good mark | `100-LKG-HEALTH-AC-002` | blocking |
| `100` | mutable rollback target | `fixture-100-mutable-rollback-target` | TODO | `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | TODO | no package activation | `100-ROLLBACK-AC-001` | blocking |
| `100` | rollback compatibility failure | `fixture-100-rollback-state-incompatible` | TODO | `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `100` | quarantine blocks dependent activation | `fixture-100-quarantine-artifact-blocks-release` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent package activation | `100-QUARANTINE-AC-001` | blocking |
| `100` | emergency bypass forbidden | `fixture-100-emergency-bypass-forbidden` | TODO | `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | TODO | no package activation | `100-EMERGENCY-AC-001` | blocking |
| `100` | package manifest refs missing | `fixture-100-version-manifest-package-refs-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `100-VERSION-MANIFEST-AC-001` | blocking |
| `100` | canary output isolation | `fixture-100-canary-output-isolation` | TODO | `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | TODO | no current production output or LKG | `100-LIFECYCLE-AC-003` | blocking |
| `100` | shadow output isolation | `fixture-100-shadow-output-isolation` | TODO | `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | TODO | no current production output or LKG | `100-LIFECYCLE-AC-003` | blocking |
| `110` | state-label collapse | `fixture-110-state-label-collapse` | TODO | reject/non-collapse | TODO | no incorrect API state | `110-CLEANUP-AC-002` | blocking |
| `110` | page token authorization mismatch | `fixture-110-page-token-auth-mismatch` | TODO | `PAGE_TOKEN_INVALID` | TODO | no data disclosure | `110-PAGE-TOKEN-AC-001` | blocking |
| `110` | new owner state label non-collapse | `fixture-110-feed-closure-state-labels` | TODO | reject/non-collapse | TODO | no authorized negative fact rendering | `110-FEED-CLOSURE-AC-001` | blocking |
| `130` | analysis mutation | `fixture-130-analysis-mutation` | TODO | `ANALYSIS_MUTATION_FORBIDDEN` | TODO | no mutation | `130-CLEANUP-AC-003` | blocking |
| `130` | lineage facet checksum mismatch | `fixture-130-lineage-facet-checksum-mismatch` | TODO | `LINEAGE_FACET_CHECKSUM_MISMATCH` | TODO | no lineage activation | `130-AC-003` | blocking |
| `200` | active reachability output | `fixture-200-active-reachability-output` | TODO | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | TODO | no gold, graph, API output | `200-CLEANUP-AC-003` | blocking |
| volatility | activation artifact redefines stable core behavior; activation artifact omitted from `VersionManifest`; runtime state substituted as source truth | `fixture-120-volatility-boundary` | TODO | owner-specific volatility error or no-op | TODO | no production mutation | `120-VOLATILITY-AC-001` | blocking |

### LifecycleValidationMatrix

Lifecycle validation rows must be executable ordered event sequences. Each row must name expected transition evidence, expected output checksum or no-op, expected error when applicable, and mutation-prohibition proof.

| Validation row | Owner | Required assertion |
| --- | --- | --- |
| `val-030-lifecycle-artifact-totality` | `030` | Every artifact lifecycle state/event pair resolves to legal transition, no-op, or illegal-transition evidence. |
| `val-030-lifecycle-idempotent-replay` | `030` | Same event key and same inputs produce byte-identical evidence. |
| `val-030-lifecycle-idempotency-conflict` | `030` | Same key with changed input checksum fails with `LIFECYCLE_IDEMPOTENCY_CONFLICT`. |
| `val-030-lifecycle-production-run` | `030` | Non-isolated stage failure stops downstream stages and emits no downstream production output. |
| `val-030-lifecycle-stage-execution` | `030` | Stage execution machine has total state/event coverage and expected terminal states. |
| `val-030-lifecycle-feed-stage` | `020`, `030` | Partial feed read emits valid lifecycle result with blocked effects and no absence or watermark. |
| `val-090-lifecycle-graph-apply-totality` | `090` | Graph apply machine has total state/event coverage. |
| `val-090-lifecycle-graph-apply-resume-safe` | `090` | Partial apply with committed-batch proof resumes after last compatible batch. |
| `val-090-lifecycle-graph-apply-resume-unsafe` | `090` | Partial apply without proof fails with `GRAPH_APPLY_RESUME_UNSAFE`. |
| `val-090-lifecycle-graph-apply-identical-reapply` | `090` | Identical reapply returns no-op and does not mutate graph. |
| `val-100-lifecycle-package-activation` | `100` | Candidate activation failure preserves current active package set. |
| `val-100-lifecycle-canary-isolation` | `100` | Canary and shadow emit no current production output, watermark, or last-known-good state. |
| `val-100-lifecycle-rollback-immutable-target` | `100` | Mutable rollback targets fail. |
| `val-100-lifecycle-quarantine-blocks-dependent-activation` | `100` | Quarantine blocks dependent activation and cannot clear directly to active. |
| `val-100-lifecycle-deprecation-window-expiry` | `100` | Deprecated artifact cannot be newly selected after window expiry except verified rollback. |
| `val-120-lifecycle-validation-acceptance` | `120` | Acceptance reaches `passed` only when every required row passes and no owner TODO remains. |
| `val-domain-lifecycle-todo-resolved` | `domain`, `000`, `120` | `DOM-TODO-010` is resolved only when all owner machines and validation rows exist and pass. |

### ValidationAcceptanceLifecycleMachine

Machine ID: `120.ValidationAcceptanceLifecycleMachine.v1`.

States:

```text
pending
running
passed
failed
blocked
expired
superseded
```

Events:

```text
start_validation
all_rows_passed
row_failed
owner_todo_found
acceptance_expired
superseded_by_new_report
replay_same_event
```

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `pending` | `start_validation` | `running` | Matrix, fixture refs, owner specs, and input checksums validate. |
| `running` | `all_rows_passed` | `passed` | Persist `AcceptanceReport`. |
| `running` | `row_failed` | `failed` | Acceptance report includes failing rows. |
| `running` | `owner_todo_found` | `blocked` | Persist blocking report. |
| `passed` | `acceptance_expired` | `expired` | Persist expiry evidence. |
| `passed`, `failed`, `blocked`, or `expired` | `superseded_by_new_report` | `superseded` | Persist supersession evidence. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

### ExpectedMutationProhibition rows

| Prohibition token | Required proof |
| --- | --- |
| no production output | Output manifest contains no production-visible record refs. |
| no graph mutation | Graph apply result absent or no-op; derived-view state unchanged. |
| no watermark advancement | No `WatermarkCommitRecord` with advanced value. |
| no package activation | Current active package set checksum unchanged. |
| no identity mutation | No new terminal `IdentityDecision` that mutates canonical identity. |
| no raw payload exposure | API/export/audit caller-visible fields contain no raw payload bytes. |

### Owner-specific fixture families

| Owner | Required fixture families |
| --- | --- |
| `020` | feed read, feed lifecycle event derivation, manifest validation, read completeness, raw ID replay, omitted object, partial known gap, feed category closure, feasibility assessment, missing profile field, target-kind omission, declared subset, empty-scope, object/partition known gap, unknown gap, private-binding leak. |
| `030` | DAG ordering, lifecycle transition, lifecycle totality, lifecycle idempotency, lifecycle conflict, version manifest ID/checksum, run lock failure, forbidden output, declared subset profile missing, subset scope mismatch, subset output forbidden. |
| `050` | OCSF mapping row success, missing row, ambiguous row, activity discriminator missing, required object path missing, forbidden field, unknown enum, OCSF Other not permitted, deprecated field, source extension rule success, undeclared source extension, wildcard rejection, secret scan, cadastre-only null profile, DHCP/IPAM split required. |
| `060` | absence, coverage, progress signal, source staleness, control result mapping, feed completeness row, effect gate, blocking precedence, source-specific coverage domains, watermark gating, OCSF status non-authority, OCSF severity non-authority, OCSF confidence non-authority, OCSF observable non-authority, OCSF field absence non-authority. |
| `070` | identity-create, identity-attach, identity-durable-merge, identity-weak-rejection, identity-hard-blocker, identity-candidate-overflow, identity-confidence-band, identity-review-state-machine, identity-selector-safety, identity-split-handoff, identity-explanation-checksum, identity-replay, identity-package-artifact-core-conflict. |
| `080` | temporal-resolution, knowledge-time-import, late-arrival, gold-correction, assertion-transition, correction-snapshot-ref, gold-correction-no-op-error, replay-equivalence, graph-handoff, temporal-version-manifest. |
| `090` | active profile exact edge set, observed-connection positive projection, missing/ambiguous flow-role no-edge, unresolved endpoint no-edge, traversal behavior, graph apply lifecycle, graph apply ordering, page-token behavior, query candidate limit, rebuild equivalence, backend taxonomy rejection, reachability prohibition, backend ID rejection, OCSF endpoint-order no graph direction, generic external payload non-pathfinding, and source-kind cleanup safety. |
| `100` | package type policy, repository form, trust, transparency, attestation, SBOM, dependency lock, compatibility, deprecation, health/LKG, rollback, quarantine, emergency, package manifest completeness, canary and shadow isolation. |
| `110` | API outcome, redaction, paging, state labels, authorization non-leakage. |
| `130` | analysis mutation, lineage facet, registry non-authority, threat-intel no-identity, risk scoring boundary. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `120-CLEANUP-AC-001` | No banned reference class remains. |
| `120-CLEANUP-AC-002` | `SPECSET-AC-*` rows cover ownership, imports/exports, define-once, lakehouse boundary, negative tests, deferred reachability, and recreatability. |
| `120-CLEANUP-AC-003` | `RunValidationMatrix` output remains deterministic and includes row ID, owner spec, fixture checksum, input checksum, expected output checksum, actual output checksum, result, failure code, and version manifest ref. |
| `120-SCHEMA-PATCH-AC-001` | Every 040 exported core record has at least one success fixture and one required-field rejection fixture. |
| `120-SCHEMA-PATCH-AC-002` | Every nullable field has a null acceptance or null rejection fixture matching its 040 schema row. |
| `120-SCHEMA-PATCH-AC-003` | Every declared default has a materialization fixture. |
| `120-SCHEMA-PATCH-AC-004` | Every record ID algorithm has a replay fixture and collision fixture. |
| `120-SCHEMA-PATCH-AC-005` | Every record checksum policy has a replay-equivalence fixture. |
| `120-SCHEMA-PATCH-AC-006` | The two-independent-implementer check covers raw, silver, canonical entity, source asset, identifier, gold, evidence, and graph delta fixtures. |
| `120-SCHEMA-PATCH-AC-007` | The aggregate `AcceptanceReport` cannot pass while any 040 schema row, fixture checksum, expected output checksum, or owner error code remains `TODO:`. |
| `120-CLEANUP-AC-004` | Required negative tests by owner remain present and do not depend on external product drafts, decomposition-control artifacts, private binding indexes, or promotion-readiness gates. |
| `120-TEMPORAL-CORRECTION-AC-001` | Aggregate acceptance fails while any `080` temporal, correction, late-arrival, replay, no-op/error, graph handoff, or manifest fixture is blocked, missing checksum, missing expected output, or not run. |
| `120-ASSERTION-TRANSITION-AC-001` | Every assertion-state/correction-event pair has one expected result. |
| `120-REPLAY-OUTPUT-CLASS-AC-001` | Replay rows exist for raw, silver, identity, gold, gold correction, graph delta, graph apply, graph rebuild, API response, export projection, analysis output, and validation acceptance. |
| `120-GRAPH-HANDOFF-AC-001` | Every graph handoff effect has success, error, or no-op validation. |
| `120-NOOP-ERROR-AC-001` | No-op and deterministic error cases prove no forbidden fact, graph, export, replay, or watermark mutation. |

| `120-STATUS-CONSISTENCY-AC-001` | Owner/domain status contradiction, domain runtime restatement, owner-spec contradiction, ADR status, and manifest path mismatch validation rows exist and fail with exact codes. |
| `120-MUTATION-PROHIBITION-AC-001` | Every negative validation row has a mutation-prohibition proof for each affected mutation class. |
| `120-ACCEPTANCE-PRECEDENCE-AC-001` | Aggregate `AcceptanceReport` cannot pass while any non-deferred required row is `fail`, `blocked`, or `not_run`, or while any required fixture checksum or expected output checksum is `TODO`. |
| `120-VOLATILITY-AC-001` | Aggregate `AcceptanceReport` fails while any non-deferred volatility row is `fail`, `blocked`, `not_run`, or missing expected checksum. |
| `120-VOLATILITY-AC-002` | Validation rows cover inactive, missing, mismatched, out-of-scope, and runtime-restating activation artifacts. |
| `120-FEED-CLOSURE-AC-001` | Aggregate acceptance fails while any active feed category lacks a closure row, deterministic block row, validation refs, or fixture refs. |
| `120-FEED-CLOSURE-AC-002` | Every profile-dependent branch in `020`, `030`, and `060` has a validation row with expected receipt state, error/no-op, user-facing label, and mutation prohibition. |
| `120-FEED-CLOSURE-AC-003` | Every active feed category has positive and negative fixtures for complete, partial, stale, permission-limited, missing-profile-row, missing-upstream-evidence, weak-progress, empty-complete, and watermark-blocked cases. |
| `120-FEED-CLOSURE-AC-004` | Every absence-sensitive output has exact authority, completeness, coverage, staleness, and watermark fixtures where applicable. |
| `120-SOURCE-CLOSURE-AC-001` | Every active absence-sensitive feed category has closure validation rows or deterministic block rows. |
| `120-SOURCE-CLOSURE-AC-002` | Every missing, ambiguous, inactive, or checksum-mismatched source-authority closure component produces the expected owner error or no-op and no forbidden mutation. |
| `120-SOURCE-CLOSURE-AC-003` | Two independent implementations produce byte-identical `AbsenceDerivationResult`, `VersionManifest`, and expected output checksums for the same closure fixtures. |
| `120-SOURCE-CLOSURE-AC-004` | `ValidateSpecSet` fails promotion when `domain.md`, `000`, `020`, `030`, `040`, `060`, `080`, `090`, or `110` disagree on source-authority closure state. |

| `120-LIFECYCLE-AC-001` | Every production-affecting lifecycle machine has executable event-sequence fixtures. |
| `120-LIFECYCLE-AC-002` | Aggregate `AcceptanceReport` cannot pass while any required lifecycle fixture is blocked, not run, failed, missing checksum, or missing expected output. |
| `120-LIFECYCLE-AC-003` | Validation acceptance lifecycle matrix is total and idempotent. |
| `120-OCSF-MAP-AC-001` | `AcceptanceReport` cannot pass while any active `ObservationToOCSFMappingRow` lacks positive and negative fixtures. |
| `120-OCSF-MAP-AC-002` | `AcceptanceReport` cannot pass while any mapping fixture checksum or expected output checksum is `TODO`. |
| `120-OCSF-MAP-AC-003` | `AcceptanceReport` cannot pass while any active mapping row lacks `VersionManifest` artifact refs. |
| `120-SOURCE-EXT-AC-001` | `AcceptanceReport` cannot pass while any emitted source-extension path lacks a matching rule fixture and undeclared-path rejection fixture. |
| `120-OCSF-NONAUTH-AC-001` | `AcceptanceReport` cannot pass unless OCSF status, severity, confidence, observables, enrichments, and field absence non-authority fixtures pass. |
| `120-OCSF-DIRECTION-AC-001` | `AcceptanceReport` cannot pass unless OCSF endpoint-order graph-direction rejection fixtures pass. |
| `120-LIFECYCLE-AC-004` | Domain/owner lifecycle closure status contradiction fails with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| `120-IDENTITY-CLOSURE-AC-001` | Aggregate acceptance fails while any `070` resolver row coverage, evidence class, scope, decision matrix, confidence band, review routing, split policy, explanation policy, activation report, or selector safety fixture is missing, blocked, failed, not run, checksum-mismatched, or has a `TODO` expected output. |
| `120-IDENTITY-CLOSURE-AC-002` | Validation proves deterministic resolver pair ordering and byte-identical decision/explanation checksums across two independent implementations. |
| `120-IDENTITY-CLOSURE-AC-003` | Overflow fixtures prove no identity mutation and no auto-merge from overflowed candidates. |
| `120-IDENTITY-REVIEW-AC-001` | Every review state/event pair is covered and illegal transitions emit no mutation. |
| `120-IDENTITY-SPLIT-AC-001` | Split handoff fixtures prove required `GraphCorrectionHandoff` fields, no resolver graph mutation, and deterministic replay. |
| `120-IDENTITY-EXPLANATION-AC-001` | Resolver explanation checksum fixtures include every required output-affecting field and exclude non-output volatile fields. |
| `120-IDENTITY-PACKAGE-AC-001` | Package-supplied resolver artifact weakening fails activation and preserves the current active package set. |
| `120-GRAPH-PROFILE-CLOSURE-AC-001` | Aggregate acceptance fails while any non-deferred `GraphActiveProfileClosureValidationMatrix` row is failed, blocked, not run, missing fixture checksum, missing expected-output checksum, or missing mutation-prohibition proof. |
| `120-GRAPH-QUERY-TRANSLATION-AC-001` | Graph query candidate-limit, traversal-class, page-token expiry, and page-token mismatch fixtures pass before graph serving promotion. |
| `120-GRAPH-OUTPUT-ELIGIBILITY-AC-001` | Output eligibility fixtures prove generic external payloads and synthetic structural objects are not pathfinding, finding, metric, or identity inputs in MVP. |
| `120-GRAPH-APPLY-ORDER-AC-001` | Apply-order fixtures prove byte-identical graph delta ordering across two independent implementations. |
| `120-GRAPH-REBUILD-EQUIVALENCE-AC-001` | Rebuild fixtures prove profile, schema, index, query, eligibility, taxonomy, property, and output checksum equivalence. |
| `120-GRAPH-ENDPOINT-IDENTITY-AC-001` | Endpoint identity fixtures prove unresolved target hints, graph keys, and selector-only evidence emit no graph endpoint. |
| `120-GRAPH-PAGE-TOKEN-AC-001` | Page-token fixtures prove TTL, expiry, mismatch, authorization context, derived-view state, and profile-ref identity behavior. |
| `120-PACKAGE-ACTIVATION-AC-001` | Aggregate acceptance fails while any required package activation fixture is blocked, not run, failed, missing fixture checksum, missing expected-output checksum, missing expected error, or missing mutation-prohibition proof. |
| `120-PACKAGE-MATRIX-AC-001` | Every `PackageActivationValidationMatrix` row has fixture checksum, expected output checksum, expected error or no-op, mutation prohibition, owner spec, and version manifest requirement. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `120-AC-001` | Every domain NLSpec has binary acceptance criteria and validation matrix rows for success, rejection, no-op, and edge behavior. |
| `120-AC-002` | Every required negative authority-boundary case is executable and produces the expected owner-specific error code. |
| `120-AC-003` | Golden corpus replay produces byte-identical expected outputs or deterministic failure records. |
| `120-AC-004` | Validation artifacts are redacted, checksummed, and replayable. |
| `120-AC-005` | The aggregate acceptance report is sufficient to decide whether the NLSpec set can be promoted to `authoritative`. |
| `120-AC-006` | `RunValidationMatrix` emits blocked or fail for any unresolved active feed category, unresolved profile branch, missing fixture checksum, or missing expected output checksum. |
| `120-AC-007` | Authoritative promotion is forbidden while any non-deferred identity closure row is `fail`, `blocked`, `not_run`, missing checksum, or has a `TODO` expected output. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
