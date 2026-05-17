---
doc_id: CADASTRE-NLSPEC-120
title: Validation, Fixtures, and Acceptance
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

Define fixtures, validation matrices, golden corpus, shadow execution, replay validation, required negative tests, and binary acceptance reports for the NLSpec set.

## Explicit Non-Scope

- Owning domain behavior.
- New behavior not exported by a domain spec.
- Implementation-specific test harness mechanisms unless observable validation output is affected.

## Imports

- `All active domain specs`

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

## Migration Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `MIG-AC-001` | Every PRD top-level section has a final disposition row in the migration ledger. |
| `MIG-AC-002` | Every PRD Section 10 model, Section 11 interface, Section 12 default, Section 17 error family, Section 18 edge case, Section 23 acceptance criterion, and Section 24 decision has an owner or explicit deferred/archive disposition. |
| `MIG-AC-003` | The NLSpec index identifies exactly one owner for every model, interface, algorithm, default, error code, mapping table, lifecycle state, and acceptance criterion. |
| `MIG-AC-004` | No active NLSpec restates another active NLSpec requirement except by exact named reference. |
| `MIG-AC-005` | Every active NLSpec declares dependencies, exported contracts, imported contracts, explicit non-scope, and Definition of Done. |
| `MIG-AC-006` | Every production input path remains lakehouse-fed and has no direct enterprise source-call behavior. |
| `MIG-AC-007` | Every domain spec has binary acceptance criteria and at least one required negative validation case for each forbidden authority boundary. |
| `MIG-AC-008` | The deferred reachability document is inactive and no active MVP graph or gold spec can emit theoretical reachability output. |
| `MIG-AC-009` | The archived PRD is marked superseded and implementation authority comes from the NLSpec set after cutover. |
| `MIG-AC-010` | The recreatability test passes without using the original PRD. |

## Migration finalization contracts

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

### MigrationCutoverChecklist

| Criterion | Required evidence artifact | Owner | Status | Failure code | Cutover effect |
| --- | --- | --- | --- | --- | --- |
| `MIG-AC-001` | PRD ledger final rows | `000` | `blocked` | `MIG_PRD_LEDGER_INCOMPLETE` | block cutover |
| `MIG-AC-002` | model/interface/default/error/edge/decision disposition rows | `000` | `blocked` | `MIG_DISPOSITION_INCOMPLETE` | block cutover |
| `MIG-AC-003` | owner registry | `000` | `blocked` | `MIG_OWNER_AMBIGUOUS` | block cutover |
| `MIG-AC-004` | duplicate/restatement review | `000` | `blocked` | `MIG_DEFINE_ONCE_VIOLATION` | block cutover |
| `MIG-AC-005` | per-spec imports/exports/DoD validation | `120` | `blocked` | `MIG_SPEC_SHAPE_INCOMPLETE` | block cutover |
| `MIG-AC-006` | direct-source negative tests | `010`, `020`, `120` | `blocked` | `MIG_DIRECT_SOURCE_BOUNDARY_FAIL` | block cutover |
| `MIG-AC-007` | negative authority-boundary matrix | `120` | `blocked` | `MIG_NEGATIVE_TESTS_INCOMPLETE` | block cutover |
| `MIG-AC-008` | deferred reachability negative rows | `200`, `090`, `120` | `blocked` | `MIG_REACHABILITY_ACTIVE` | block cutover |
| `MIG-AC-009` | PRD archive acceptance report | `000`, `120` | `blocked` | `MIG_PRD_ARCHIVE_NOT_ACCEPTED` | block cutover |
| `MIG-AC-010` | recreatability report | `120` | `blocked` | `MIG_RECREATABILITY_FAIL` | block cutover |

### TwoIndependentImplementersCheck

| Field | Required content |
| --- | --- |
| comparison scope | Every authoritative owner contract and validation row in the spec set. |
| allowed variance | Non-observable implementation structure only. |
| output equivalence | Byte-identical canonical records or deterministic equivalent observable outputs. |
| failure classification | spec ambiguity, implementation defect, fixture defect, or deferred issue. |
| acceptance threshold | All non-deferred authoritative rows pass. |

### RecreatabilityCheck

The recreatability check passes only when the intended behavior can be derived from NLSpecs without the PRD except for explicitly deferred or archive-trace issues. Any required PRD lookup after cutover is a failure unless the lookup is trace-only.

### AcceptanceReport statuses

| Status | Meaning | Failure precedence |
| --- | --- | --- |
| `fail` | Row ran and produced mismatched output, forbidden mutation, or wrong error. | Highest. |
| `blocked` | Row cannot run because required artifact, owner decision, fixture, or dependency is missing. | Second. |
| `not_run` | Row exists but has no execution evidence. | Third. |
| `pass` | Row ran and all expected evidence matched. | Lowest. |

### Required negative tests by owner

| Owner spec | Forbidden boundary | Fixture ID | Expected error/no-op | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- |
| `010` | direct source call | TODO | `DIRECT_SOURCE_CALL_FORBIDDEN` | `010-PATCH-AC-003` | blocking |
| `020` | CDC metadata as authority | TODO | no-op | `020-PATCH-AC-004` | blocking |
| `030` | forbidden stage output | TODO | `FORBIDDEN_STAGE_OUTPUT` | `030-PATCH-AC-001` | blocking |
| `040` | unknown field | TODO | owner error | `040-PATCH-AC-002` | blocking |
| `050` | undeclared extension | TODO | mapping error | `050-PATCH-AC-003` | blocking |
| `060` | weak progress absence | TODO | no-op/unknown | `060-PATCH-AC-002` | blocking |
| `070` | weak evidence auto-merge | TODO | no_decision | `070-PATCH-AC-002` | blocking |
| `080` | current-time fallback | TODO | temporal error | `080-PATCH-AC-001` | blocking |
| `090` | theoretical reachability edge | TODO | reachability scope error/no-op | `090-PATCH-AC-004` | blocking |
| `100` | unauthorized signer | TODO | activation failure | `100-PATCH-AC-001` | blocking |
| `110` | state-label collapse | TODO | reject/non-collapse | `110-PATCH-AC-004` | blocking |
| `130` | analysis mutation | TODO | analysis mutation error | `130-PATCH-AC-001` | blocking |
| `200` | active reachability output | TODO | no-op/rejected | `200-PATCH-AC-001` | blocking |

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `120-PATCH-AC-001` | Every active domain spec has validation rows for success, rejection, no-op, and edge behavior. |
| `120-PATCH-AC-002` | Every forbidden authority boundary has at least one executable negative case. |
| `120-PATCH-AC-003` | Acceptance reports are sufficient to decide whether NLSpecs supersede the PRD. |
| `120-PATCH-AC-004` | The two-independent-implementers and recreatability checks are explicit and binary. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `120-AC-001` | Every domain NLSpec has binary acceptance criteria and validation matrix rows for success, rejection, no-op, and edge behavior. |
| `120-AC-002` | Every required negative authority-boundary case is executable and produces the expected owner-specific error code. |
| `120-AC-003` | Golden corpus replay produces byte-identical expected outputs or deterministic failure records. |
| `120-AC-004` | Validation artifacts are redacted, checksummed, and replayable. |
| `120-AC-005` | The aggregate acceptance report is sufficient to decide whether the NLSpec set can supersede the PRD. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| docs/archive/PRD-Cadastre.revised-draft.md | `ValidationMatrix` | lines 5780-5954 |
| docs/archive/PRD-Cadastre.revised-draft.md | `LakehouseFeedFixture` | lines 7450-7491 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ValidationScenario` | lines 8535-8613 |
| docs/archive/PRD-Cadastre.revised-draft.md | `EventSequenceValidationCorpus` | lines 8568-8613 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Golden Corpus` | lines 12488-12696 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Shadow Execution` | lines 12697-12749 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Replay` | lines 3311-3335 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Acceptance Criteria` | lines 12869-13634 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
