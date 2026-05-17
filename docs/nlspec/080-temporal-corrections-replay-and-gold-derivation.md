---
doc_id: CADASTRE-NLSPEC-080
title: Temporal Corrections, Replay, and Gold Derivation
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

Define temporal semantics, bitemporal facts, late arrivals, corrections, replay equivalence, and deterministic gold derivation.

## Explicit Non-Scope

- Graph apply implementation.
- Backend query translation.
- Package trust.
- External schema mappings.

## Imports

- `RawRecord`
- `CadastreSilverObservation`
- `GoldFact`
- `SourceAuthorityProfile`
- `LakehouseSnapshotRef`
- `DatasetVersionRef`
- `VersionManifest`
- `IdentityDecision`

## Exports

- `TemporalSemanticsPolicy`
- `KnowledgeTimeImportPolicy`
- `TemporalObservationTimeResolution`
- `LateArrivalPolicy`
- `BitemporalQueryMode`
- `GoldFactCorrectionPolicy`
- `GoldFactChangeSet`
- `CorrectionSnapshotRefPolicy`
- `ReplayEquivalencePolicy`
- `ReplayInputSufficiencyCheck`
- `GraphRebuildEquivalencePolicy`
- `EventSequenceValidationCorpus`
- `ResolveFactTime`
- `EvaluateLateArrival`
- `ApplyGoldCorrection`
- `DeriveFacts`
- `ComputeReplayEquivalenceChecksum`

## Temporal Axes

Cadastre must keep source event time, source observation time, supplier collection time, supplier delivery time, lakehouse commit time, table snapshot time, CDC connector time, CDC heartbeat time, schema-history time, graph apply time, replay time, and platform current time distinct.

A `GoldFact` candidate may be created only after `ResolveFactTime` emits exactly one `TemporalObservationTimeResolution` row or a deterministic temporal error.

## ResolveFactTime Algorithm

```text
ResolveFactTime(observation, temporal_policy, knowledge_time_policy):
1. Validate temporal_policy covers source dataset, observation type, and fact type.
2. Select valid-time input using declared precedence.
3. Reject absent, malformed, ambiguous, or non-authoritative time unless policy names an explicit fallback.
4. Select known-time input using KnowledgeTimeImportPolicy.
5. Reject reconstructed historical known time unless policy permits and required source-known-time evidence validates.
6. Emit TemporalObservationTimeResolution with selected fields, quality, source refs, policy refs, and checksum.
```

Implicit current-time fallback is forbidden.

## Gold Derivation

`DeriveFacts` must consume silver observations, identity decisions, source authority decisions, coverage assertions, temporal resolutions, and active derivation profiles. It must emit `GoldFact`, `GoldFactChangeSet`, no-op, conflict, or deterministic error. It must not mutate existing `GoldFact` rows in place.

## Correction Contract

Gold corrections are append-only knowledge transitions. `ApplyGoldCorrection` must close prior `known_to` intervals and emit new facts, interval splits, explicit retractions, conflicts, no-ops, or deterministic errors.

`CorrectionSnapshotRefPolicy` must require old and new lakehouse snapshot refs for every correction class. `TODO: finalize old/new table snapshot roles, table-set checksums, and retention-protection rows by correction class.`

## Assertion-State Transition Matrix

Every correction policy must define a total transition matrix across assertion states. An omitted transition is illegal and must emit `GOLD_CORRECTION_TRANSITION_UNDEFINED`.

## Late Arrival

Late evidence must be routed through `EvaluateLateArrival` using late-arrival policy, temporal resolution, source authority, completeness, and watermark state. Silently discarding late authoritative evidence in production is forbidden.

## Replay Contract

Production replay must run `ReplayInputSufficiencyCheck`, `ReplayEquivalencePolicy`, `DecideReplayMode`, and `ComputeReplayEquivalenceChecksum` before any replay output is written.

`ReplayEquivalencePolicy` must define included fields, excluded volatile fields, hash algorithms, canonical ordering, failure precedence, and shadow-output behavior by output class. `TODO: finalize replay equivalence policy catalog.`

## Deterministic Side Effects

Runtime randomness, wall-clock reads, generated IDs, unordered iteration, external calls, or backend-discovered values must not affect production output unless a declared `DeterministicSideEffectRecord` captures the value and replay behavior.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `080-AC-001` | Current, valid-at, known-at, corrected-history, replay, and audit queries are reconstructable from persisted records. |
| `080-AC-002` | Every gold fact candidate has a temporal resolution row or deterministic temporal error. |
| `080-AC-003` | Corrections never mutate existing `GoldFact` rows in place. |
| `080-AC-004` | Replay rejects missing, mutable-only, retention-ineligible, schema-incompatible, authority-mismatched, or checksum-mismatched inputs before writing output. |
| `080-AC-005` | Event-sequence validation covers temporal resolution, late arrival, correction, replay, side effects, and graph rebuild equivalence. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `TemporalSemanticsPolicy` | lines 1953-1983 |
| PRD-Cadastre.md | `KnowledgeTimeImportPolicy` | lines 1984-2010 |
| PRD-Cadastre.md | `TemporalObservationTimeResolution` | lines 2011-2058 |
| PRD-Cadastre.md | `LateArrivalPolicy` | lines 2059-2087 |
| PRD-Cadastre.md | `BitemporalQueryMode` | lines 2088-2123 |
| PRD-Cadastre.md | `GoldFact` | lines 2719-2800 |
| PRD-Cadastre.md | `GoldFactCorrectionPolicy` | lines 2801-2831 |
| PRD-Cadastre.md | `GoldFactChangeSet` | lines 2832-2871 |
| PRD-Cadastre.md | `CorrectionSnapshotRefPolicy` | lines 2872-2897 |
| PRD-Cadastre.md | `Gold Correction Assertion-State Transition Matrix` | lines 2898-2913 |
| PRD-Cadastre.md | `ReplayEquivalencePolicy` | lines 3781-3806 |
| PRD-Cadastre.md | `ReplayInputSufficiencyCheck` | lines 3807-3853 |
| PRD-Cadastre.md | `DeterministicSideEffectRecord` | lines 3854-3879 |
| PRD-Cadastre.md | `GraphRebuildEquivalencePolicy` | lines 3880-3906 |
| PRD-Cadastre.md | `Gold Fact Derivation Interface` | lines 9612-9729 |
| PRD-Cadastre.md | `Temporal Resolution, Late Arrival, and Correction Interfaces` | lines 9645-9692 |
| PRD-Cadastre.md | `Replay, Side-Effect, Graph Resume, and Watermark Interfaces` | lines 9693-9729 |
| PRD-Cadastre.md | `Temporal, Correction, Replay, Watermark, CDC, and Idempotency Acceptance` | lines 13538-13574 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
