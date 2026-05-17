---
doc_id: CADASTRE-NLSPEC-120
title: Validation, Fixtures, and Acceptance
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
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
| PRD-Cadastre.md | `ValidationMatrix` | lines 5780-5954 |
| PRD-Cadastre.md | `LakehouseFeedFixture` | lines 7450-7491 |
| PRD-Cadastre.md | `ValidationScenario` | lines 8535-8613 |
| PRD-Cadastre.md | `EventSequenceValidationCorpus` | lines 8568-8613 |
| PRD-Cadastre.md | `Golden Corpus` | lines 12488-12696 |
| PRD-Cadastre.md | `Shadow Execution` | lines 12697-12749 |
| PRD-Cadastre.md | `Replay` | lines 3311-3335 |
| PRD-Cadastre.md | `Acceptance Criteria` | lines 12869-13634 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
