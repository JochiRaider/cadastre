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

## Validation Ownership Rule

This spec verifies behavior owned by domain specs. It must not create new behavioral requirements unless the requirement is a validation interface, fixture record, acceptance report, or promotion gate.

## Validation Artifact Model

| Artifact | Required purpose |
| --- | --- |
| `ValidationMatrix` | Rows mapping contract, scenario, fixture, expected output, expected error/no-op, owner spec, and pass/fail evidence. |
| `LakehouseFeedFixture` | Redacted replayable feed rows, objects, manifests, supplier metadata, and expected read/import outcomes. |
| `ValidationScenario` | Validation-only scenario with given records, expected outputs, expected diagnostics, no-op assertions, and mutation prohibition. |
| `EventSequenceValidationCorpus` | Ordered event sequences for temporal, correction, replay, watermark, CDC, graph apply, and rebuild behavior. |
| `GoldenCorpus` | Canonical set of input fixtures and expected outputs used for regression and promotion. |
| `AcceptanceReport` | Immutable report that records criteria pass/fail status, checksums, owner, environment, and timestamp. |

## Required Negative Test Classes

Every active domain spec must include at least one negative validation case for each forbidden authority boundary it declares. Required classes include:

| Boundary | Required negative case |
| --- | --- |
| Direct source calls | Production package attempts enterprise source API call and fails before output. |
| Feed completeness | Missing feed row attempts absence and fails. |
| Source authority | Missing exact authority row attempts gold derivation and fails. |
| Identity | IP-only/hostname-only/DNS-only/PTR-only/graph-key-only merge attempt fails. |
| Graph | Backend internal ID appears in response or selector and fails. |
| Package | Unauthorized signer with valid cryptographic signature fails activation. |
| Mapping | Undeclared source extension field fails validation. |
| Temporal | Missing temporal policy attempts current-time fallback and fails. |
| Analysis | Analysis rule attempts mutation and fails. |
| Reachability | MVP graph profile attempts `has_theoretical_reachability` and fails. |

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

| Owner spec | Required success | Required rejection | Required no-op | Required edge | Required replay | Required redaction | Required authorization | Required negative authority-boundary |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | authority class | direct source call | validation exploration no-op | private/public boundary | n/a | private leak | API/private boundary | direct source, projection authority |
| `020` | feed read/import | malformed manifest | partial no-absence | CDC metadata | raw ID replay | private binding | n/a | no direct source call |
| `030` | DAG execution | forbidden output | safe subset no-op | lifecycle illegal transition | manifest replay | n/a | n/a | subset authority |
| `040` | canonical ID/checksum | unknown field | omission distinction | collision | checksum replay | redaction state | n/a | backend ID rejection |
| `050` | OCSF mapping | artifact mismatch | unmapped no-op | unknown enum | mapping replay | source extension redaction | n/a | undeclared extension |
| `060` | absence authorized | missing authority row | weak signal no-op | partial completeness | watermark replay | n/a | n/a | missing coverage |
| `070` | resolver decision | weak merge rejection | selector no-merge | hard blocker | identity replay | n/a | n/a | manual review terminality |
| `080` | fact derivation | temporal missing | duplicate correction no-op | late arrival | replay mismatch | n/a | n/a | no current-time fallback |
| `090` | graph query/apply | backend ID rejection | empty traversal no-path | edge semantics | rebuild equivalence | raw property redaction | graph auth | reachability prohibition |
| `100` | package activation | unauthorized signer | failed candidate keep-current | rollback | package-set replay | n/a | promotion auth | emergency no trust bypass |
| `110` | API success | stale compliance rejection | empty result | state label | page token replay | raw payload redaction | inaccessible asset | state-label non-collapse |
| `130` | analysis read-only | mutation rejection | risk scoring disabled | lineage facet | registry replay | lineage redaction | registry auth | analysis non-authority |
| `200` | n/a while deferred | reachability prohibited | no-op | no graph effect | n/a | n/a | n/a | deferred reachability |

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

### Required negative tests by owner

| Owner spec | Forbidden boundary | Fixture ID | Fixture checksum | Expected error/no-op | Expected output checksum | Mutation prohibition | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | direct source call | `fixture-010-direct-source-call` | TODO | `DIRECT_SOURCE_CALL_FORBIDDEN` | TODO | no production output | `010-CLEANUP-AC-002` | blocking |
| `010` | private binding leak | `fixture-010-private-binding-leak` | TODO | `PRIVATE_BINDING_LEAK` | TODO | no public artifact write | `010-CLEANUP-AC-003` | blocking |
| `010` | projection authority violation | `fixture-010-projection-authority-violation` | TODO | `PROJECTION_AUTHORITY_VIOLATION` | TODO | no authoritative mutation | `010-CLEANUP-AC-004` | blocking |
| `020` | CDC metadata as authority | `fixture-020-cdc-metadata-non-authority` | TODO | no-op | TODO | no absence, no retraction, no watermark | `020-CLEANUP-AC-004` | blocking |
| `020` | malformed feed manifest | `feed-020-malformed-manifest` | TODO | `manifest_invalid` | TODO | no raw import commit | `020-MANIFEST-AC-001` | blocking |
| `030` | forbidden stage output | `fixture-030-forbidden-stage-output` | TODO | `FORBIDDEN_STAGE_OUTPUT` | TODO | no production output | `030-CLEANUP-AC-003` | blocking |
| `030` | duplicate stage ID | `fixture-030-duplicate-stage-id` | TODO | `DAG_STAGE_ID_DUPLICATE` | TODO | no production output | `030-DAG-ORDER-AC-001` | blocking |
| `040` | unknown field | `fixture-040-unknown-field` | TODO | `CORE_UNKNOWN_FIELD` | TODO | no record commit | `040-CLEANUP-AC-002` | blocking |
| `040` | one-of invalid | `fixture-040-one-of-invalid` | TODO | `CORE_ONE_OF_INVALID` | TODO | no record commit | `120-SCHEMA-PATCH-AC-001` | blocking |
| `050` | undeclared extension | `fixture-050-source-extension-undeclared` | TODO | `SOURCE_EXTENSION_FIELD_UNDECLARED` | TODO | no silver output | `050-CLEANUP-AC-002` | blocking |
| `050` | OCSF artifact mismatch | `fixture-050-ocsf-artifact-mismatch` | TODO | `OCSF_ARTIFACT_MISMATCH` | TODO | no normalized output | `050-OCSF-BASELINE-AC-001` | blocking |
| `060` | weak progress absence | `fixture-060-weak-progress-absence` | TODO | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` or unknown no-op | TODO | no absence, no cleanup, no watermark | `060-CLEANUP-AC-003` | blocking |
| `060` | missing coverage | `fixture-060-coverage-required` | TODO | `COVERAGE_ASSERTION_REQUIRED` | TODO | no coverage-sensitive fact | `060-COVERAGE-CLOSURE-AC-001` | blocking |
| `070` | weak evidence auto-merge | `fixture-070-weak-evidence-auto-merge` | TODO | `no_decision` | TODO | no identity mutation | `070-CLEANUP-AC-002` | blocking |
| `070` | invalid review transition | `fixture-070-invalid-review-transition` | TODO | `IDENTITY_REVIEW_TRANSITION_INVALID` | TODO | no identity mutation | `070-REVIEW-TOTALITY-AC-001` | blocking |
| `080` | current-time fallback | `fixture-080-current-time-fallback` | TODO | temporal error | TODO | no gold fact | `080-CLEANUP-AC-003` | blocking |
| `080` | replay mismatch | `fixture-080-replay-mismatch` | TODO | replay mismatch error | TODO | no replay output | `080-CLEANUP-AC-004` | blocking |
| `090` | theoretical reachability edge | `fixture-090-theoretical-reachability-edge` | TODO | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op | TODO | no graph mutation | `090-CLEANUP-AC-004` | blocking |
| `090` | backend internal ID | `fixture-090-backend-id` | TODO | `GRAPH_BACKEND_ID_FORBIDDEN` | TODO | no graph response identity leak | `090-CLEANUP-AC-003` | blocking |
| `100` | unauthorized signer | `fixture-100-unauthorized-signer` | TODO | `PACKAGE_SIGNER_UNAUTHORIZED` | TODO | no package activation | `100-CLEANUP-AC-003` | blocking |
| `100` | mutable rollback target | `fixture-100-mutable-rollback-target` | TODO | activation failure | TODO | no package activation | `100-AC-003` | blocking |
| `110` | state-label collapse | `fixture-110-state-label-collapse` | TODO | reject/non-collapse | TODO | no incorrect API state | `110-CLEANUP-AC-002` | blocking |
| `110` | page token authorization mismatch | `fixture-110-page-token-auth-mismatch` | TODO | `PAGE_TOKEN_INVALID` | TODO | no data disclosure | `110-PAGE-TOKEN-AC-001` | blocking |
| `130` | analysis mutation | `fixture-130-analysis-mutation` | TODO | `ANALYSIS_MUTATION_FORBIDDEN` | TODO | no mutation | `130-CLEANUP-AC-003` | blocking |
| `130` | lineage facet checksum mismatch | `fixture-130-lineage-facet-checksum-mismatch` | TODO | `LINEAGE_FACET_CHECKSUM_MISMATCH` | TODO | no lineage activation | `130-AC-003` | blocking |
| `200` | active reachability output | `fixture-200-active-reachability-output` | TODO | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | TODO | no gold, graph, API output | `200-CLEANUP-AC-003` | blocking |

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
| `020` | feed read, manifest validation, read completeness, raw ID replay, omitted object, partial known gap. |
| `030` | DAG ordering, lifecycle transition, version manifest ID/checksum, run lock failure, forbidden output. |
| `050` | OCSF mapping, enum handling, source extension, disabled base-event fields, cadastre-only null profile. |
| `060` | absence, coverage, progress signal, source staleness, control result mapping. |
| `070` | weak evidence, hard blocker, split handoff, review state machine, selector safety. |
| `080` | event sequence, fact time, late arrival, correction, replay mismatch, graph rebuild equivalence. |
| `090` | edge semantics, graph apply, query ordering, reachability prohibition, backend ID rejection. |
| `100` | package activation, trust, rollback, emergency behavior, repository unsupported form. |
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

| `120-STATUS-CONSISTENCY-AC-001` | Owner/domain status contradiction, domain runtime restatement, owner-spec contradiction, ADR status, and manifest path mismatch validation rows exist and fail with exact codes. |
| `120-MUTATION-PROHIBITION-AC-001` | Every negative validation row has a mutation-prohibition proof for each affected mutation class. |
| `120-ACCEPTANCE-PRECEDENCE-AC-001` | Aggregate `AcceptanceReport` cannot pass while any non-deferred required row is `fail`, `blocked`, or `not_run`, or while any required fixture checksum or expected output checksum is `TODO`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `120-AC-001` | Every domain NLSpec has binary acceptance criteria and validation matrix rows for success, rejection, no-op, and edge behavior. |
| `120-AC-002` | Every required negative authority-boundary case is executable and produces the expected owner-specific error code. |
| `120-AC-003` | Golden corpus replay produces byte-identical expected outputs or deterministic failure records. |
| `120-AC-004` | Validation artifacts are redacted, checksummed, and replayable. |
| `120-AC-005` | The aggregate acceptance report is sufficient to decide whether the NLSpec set can be promoted to `authoritative`. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
