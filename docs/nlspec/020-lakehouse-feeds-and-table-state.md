---
doc_id: CADASTRE-NLSPEC-020
title: Lakehouse Feeds and Table State
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Lakehouse Feeds and Table State

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

## Purpose

Define how Cadastre reads lakehouse-resident raw feeds, imports raw records, references table state, and preserves replay inputs.

## Explicit Non-Scope

- Source authority interpretation.
- Parser-to-silver behavior.
- Identity resolution.
- Graph projection or apply behavior.
- Package trust.

## Imports

- `DirectSourceProhibition`
- `AuthorityClass`

## Exports

- `RawSupplierProfile`
- `LakehouseFeedProfile`
- `LakehouseFeedProfileSchema`
- `LakehouseFeedAvailabilityCheck`
- `RawFeedManifest`
- `LakehouseReadPolicy`
- `RawRecordImportRun`
- `LakehouseReadCompletenessReceipt`
- `UpstreamCompletenessEvidence`
- `LakehouseTableProfile`
- `LakehouseSnapshotRef`
- `DatasetVersionRef`
- `LakehouseCommitRef`
- `ReplayRetentionPolicy`
- `TableMaintenancePolicy`
- `ReplayRetentionDecision`
- `CrossTableCommitProfile`
- `CatalogBranchPromotionPolicy`

## Feed Read Contract

A production feed read must start from an active `LakehouseFeedProfile`. The profile must name the supplier class, read target kind, scope keys, schema refs, object or table refs, package bindings, replay-relevant configuration hashes, and redacted access-reference hashes.

| Read target kind | Required reference | Output allowed |
| --- | --- | --- |
| `table_snapshot` | `LakehouseSnapshotRef` | Raw feed rows and read completeness receipt. |
| `dataset_version` | `DatasetVersionRef` | Raw feed rows and read completeness receipt. |
| `object_batch` | `RawFeedManifest` object refs | Raw feed objects and read completeness receipt. |
| `partition_set` | `RawFeedManifest` partition refs | Raw feed rows and read completeness receipt. |
| `manifest_list` | `RawFeedManifest` | Manifest validation result and read completeness receipt. |

`LakehouseFeedAvailabilityCheck` must validate catalog, table, object, partition, manifest, and schema availability without enterprise source calls or observation-producing side effects.

## Raw Import Contract

`RawRecordImportRun` must import lakehouse raw feed rows or objects into deterministic `RawRecord` identities. The import run must record feed profile ID, read policy ID, feed manifest ID, source dataset, scope keys, payload hash algorithm, import package artifact, and input table or object refs.

Deterministic raw record identity must be computed from declared feed scope, supplier batch or object identity, source-native record identity when present, canonical payload hash, source dataset, and import run profile version. `TODO: finalize exact raw feed manifest schema and raw record identity input ordering.`

## Completeness Receipt Contract

`LakehouseReadCompletenessReceipt` proves only that Cadastre read the declared feed target. It is necessary but not sufficient for source-level absence, retraction, cleanup, graph expiry, or watermark advancement.

| Receipt state | Meaning | May authorize absence by itself |
| --- | --- | --- |
| `read_complete` | Cadastre read the declared lakehouse feed target completely. | No. |
| `read_partial_known_gap` | Known partition, object, or row range was not read. | No. |
| `read_partial_unknown_gap` | Feed target may be incomplete but the missing scope is unknown. | No. |
| `read_unavailable` | Declared lakehouse target could not be read. | No. |
| `schema_unavailable` | Required schema ref was unavailable. | No. |
| `manifest_invalid` | Manifest checksum, count, or shape failed validation. | No. |

Supplier evidence must be persisted as `UpstreamCompletenessEvidence` with an explicit `authority_limit`. Supplier evidence is diagnostic-only unless imported by `060` through an active completeness profile.

## Table State Contract

Every authoritative read that can affect production output, replay, graph rebuild, or maintenance eligibility must emit a `LakehouseSnapshotRef` or `DatasetVersionRef`. Every authoritative write attempt, success, failure, rollback, restore, or maintenance commit must emit a `LakehouseCommitRef`.

| Reference | Required purpose |
| --- | --- |
| `LakehouseTableProfile` | Names table format, catalog binding, replay requirement, maintenance policy, schema compatibility policy, and checksum behavior. |
| `LakehouseSnapshotRef` | Names table-format-native snapshot, metadata/log, schema, partition, delete/tombstone, catalog, and checksum identity. |
| `DatasetVersionRef` | Names logical dataset or table-set version consumed or produced by a run. |
| `LakehouseCommitRef` | Names attempted or completed write and its table-format-native commit evidence. |

## Maintenance Safety

`TableMaintenancePolicy` must evaluate snapshot expiration, vacuum, cleaner, orphan deletion, checkpoint or transaction-log cleanup, object garbage collection, restore, rollback, compaction deletion, and catalog garbage collection before execution.

`ReplayRetentionDecision` must refuse destructive maintenance when candidate deletion or rewrite would invalidate any protected production `VersionManifest`, graph rebuild, replay window, legal hold, or retention policy.

## Feed Read Algorithm

```text
ReadLakehouseFeed(profile, read_policy, target_ref):
1. Verify profile lifecycle status is active.
2. Verify no direct enterprise source endpoint is referenced.
3. Verify LakehouseFeedAvailabilityCheck is current when required by profile.
4. Validate target_ref against read_policy target kind and checksum rules.
5. Read only declared table snapshots, dataset versions, objects, partitions, or manifests.
6. Persist read checkpoints as StageStateRecord when output-affecting.
7. Persist RawRecordImportRun inputs.
8. Persist RawRecord rows or deterministic import errors.
9. Persist LakehouseReadCompletenessReceipt.
10. Persist VersionManifest refs for every output-affecting profile, policy, target, schema, state, and manifest.
```

## Definition of Done

| ID | Criterion |
| --- | --- |
| `020-AC-001` | Raw import can be replayed from declared feed/table/object refs without enterprise source calls. |
| `020-AC-002` | Missing rows or objects never authorize absence without an imported completeness decision from `060`. |
| `020-AC-003` | Every production read and write has table-format-native refs when it can affect output or replay. |
| `020-AC-004` | Destructive maintenance is refused when any protected manifest, snapshot, replay window, or legal hold would be invalidated. |
| `020-AC-005` | Feed profile activation fails while feed feasibility, raw supplier profiles, read policy catalog, raw manifest schema, or feed fixture coverage is `TODO:` for the target feed. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `Lakehouse-Fed Boundary Amendment` | lines 101-277 |
| PRD-Cadastre.md | `Lakehouse Feed Corpus Contracts` | lines 6152-6329 |
| PRD-Cadastre.md | `LakehouseFeedCompletenessProfile` | lines 6330-6522 |
| PRD-Cadastre.md | `LakehouseReadPolicy` | lines 7637-7667 |
| PRD-Cadastre.md | `Lakehouse Table State Interface` | lines 10114-10134 |
| PRD-Cadastre.md | `Table Maintenance and Replay Retention Interface` | lines 10135-10152 |
| PRD-Cadastre.md | `Configuration, Defaults, and Bounds` | lines 10280-10646 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
