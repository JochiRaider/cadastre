---
doc_id: CADASTRE-NLSPEC-020
title: Lakehouse Feeds and Table State
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

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

Deterministic raw record identity must be computed by `ComputeRawRecordId` using the `Raw record ID input order` table in this document and `040.CanonicalJSON`.

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

## Feed and Table State Contract Details

### LakehouseFeedFeasibilityAssessment

`LakehouseFeedFeasibilityAssessment` is exported by this spec. A feed may become active only when the feed has a passing assessment or an explicit blocking row.

| Feed category | Required metadata | Timestamp sufficiency | Identifier sufficiency | Replayability | Fixture coverage | Completeness evidence | Parser/mapping readiness | Activation result |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `table_snapshot` | Snapshot ref, schema ref, scope keys, supplier profile | Required | Required for source-native or payload-hash identity | Required | Required | Required when absence-sensitive | Required | `blocked_until_all_pass` |
| `dataset_version` | Dataset version ref, table-set checksum, schema refs | Required | Required | Required | Required | Required when absence-sensitive | Required | `blocked_until_all_pass` |
| `object_batch` | Object refs, byte totals, object checksums, manifest checksum | Required or explicit `unknown` | Required or payload-hash identity | Required | Required | Optional unless absence-sensitive | Required | `blocked_until_all_pass` |
| `partition_set` | Partition refs, partition checksums, partition bounds | Required | Required | Required | Required | Required when absence-sensitive | Required | `blocked_until_all_pass` |
| `manifest_list` | Manifest refs, manifest checksums, schema refs | Required | Required if raw import follows | Required | Required | Required when absence-sensitive | Required | `blocked_until_all_pass` |

### RawSupplierProfile

| Supplier class | Public/private status | Allowed metadata | Prohibited private fields | Validation behavior |
| --- | --- | --- | --- | --- |
| `external_transport_supplier` | public vendor-neutral class | supplier class, feed scope, delivery batch ref, redacted access-ref hash | concrete vendor route, credential, tenant host list | Reject private values with `PRIVATE_BINDING_LEAK`. |
| `lakehouse_export_supplier` | public vendor-neutral class | export window, object refs, schema refs, upstream completeness refs | source credentials, source API URLs | Require manifest validation. |
| `validation_fixture_supplier` | public redacted class | fixture ID, source category, redaction summary, checksum | raw private payload unless redacted fixture permits | Validation-only, no production evidence. |
| `private_bound_supplier` | private implementation artifact only | none in public docs | all concrete bindings | Public artifact must fail if present. |

### Read target policy table

| Target kind | Required reference | Checksum inputs | Timeout default | Retry behavior | Failed-object behavior | Omitted-object behavior | Output classes | Replay requirement |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `table_snapshot` | `LakehouseSnapshotRef` | table profile, snapshot ID, schema ID, partition/delete refs | `300s` | retry metadata read 3 times | fail read | fail unless profile permits partial | raw rows, receipt | snapshot ref retained |
| `dataset_version` | `DatasetVersionRef` | dataset version, table-set checksum, schema refs | `300s` | retry metadata read 3 times | fail read | fail unless subset profile permits | raw rows, receipt | dataset ref retained |
| `object_batch` | `RawFeedManifest.object_refs` | object URI hash, size, checksum, manifest checksum | `600s` | retry object read 3 times | emit `read_partial_known_gap` | emit `read_partial_known_gap` | raw objects, receipt | object refs retained |
| `partition_set` | `RawFeedManifest.partition_refs` | partition keys, schema, bounds, manifest checksum | `600s` | retry partition read 3 times | emit `read_partial_known_gap` | emit `read_partial_known_gap` | raw rows, receipt | partition refs retained |
| `manifest_list` | `RawFeedManifest` | manifest bytes, schema refs, object/partition refs | `120s` | retry manifest read 3 times | `manifest_invalid` | `manifest_invalid` | manifest validation, receipt | manifest retained |

### RawFeedManifest schema

| Field | Type | Required | Default | Rule |
| --- | --- | ---: | --- | --- |
| `manifest_id` | string | Yes | none | Deterministic ID from canonical manifest bytes. |
| `feed_profile_id` | string | Yes | none | Must reference active `LakehouseFeedProfile`. |
| `supplier_profile_id` | string | Yes | none | Must reference active `RawSupplierProfile`. |
| `read_target_kind` | enum | Yes | none | One of the declared read target kinds. |
| `object_refs` | array | Required for object targets | `[]` | Each row includes ref, byte count, checksum, media type. |
| `partition_refs` | array | Required for partition targets | `[]` | Each row includes partition key, bounds, schema ref, checksum. |
| `schema_refs` | array | Yes | none | Every payload schema used by the manifest. |
| `record_count` | integer | Yes | `0` | Non-negative. |
| `byte_total` | integer | Yes | `0` | Non-negative. |
| `time_bounds` | object | Yes | none | Declared source, supplier, and lakehouse time bounds with quality. |
| `supplier_lineage` | object | Yes | none | Supplier batch, run, and redacted access-reference hashes. |
| `upstream_completeness_refs` | array | No | `[]` | References only; no absence authority by itself. |
| `hash_algorithm` | enum | Yes | `sha256` | Only `sha256` is active for MVP. |
| `manifest_checksum` | string | Yes | none | SHA-256 over canonical manifest bytes excluding this field. |

### Raw record ID input order

| Order | Input name | Required | Normalization | Missing-value behavior | Collision handling |
| ---: | --- | ---: | --- | --- | --- |
| 1 | `feed_profile_id` | Yes | canonical string | reject | reject duplicate candidate |
| 2 | `read_target_kind` | Yes | enum token | reject | reject duplicate candidate |
| 3 | `source_dataset` | Yes | canonical string | reject | reject duplicate candidate |
| 4 | `scope_key_set` | Yes | canonical JSON sorted by key | reject | reject duplicate candidate |
| 5 | `supplier_batch_or_object_identity` | Yes | canonical JSON | reject | reject duplicate candidate |
| 6 | `source_native_record_identity` | No | canonical JSON | materialize `null` sentinel | use payload hash tiebreak |
| 7 | `canonical_payload_hash` | Yes | lowercase SHA-256 hex | reject | collision emits `RAW_RECORD_ID_COLLISION` |
| 8 | `import_profile_version` | Yes | canonical string | reject | reject duplicate candidate |

### ComputeRawRecordId

```text
ComputeRawRecordId(inputs):
1. Validate all required inputs from the raw record ID input order table.
2. Materialize the optional source-native identity as JSON null when absent.
3. Serialize the ordered input array using `040.CanonicalJSON`.
4. Compute SHA-256 over the UTF-8 canonical bytes.
5. Return `raw_` plus the lowercase hex digest.
6. If two different canonical input arrays produce the same ID, emit `RAW_RECORD_ID_COLLISION` and commit no raw record.
```

### LakehouseFeedAvailabilityFreshnessPolicy

| Target kind | TTL default | Invalidation trigger | Stale behavior |
| --- | ---: | --- | --- |
| `table_snapshot` | `24h` | schema ref change, table profile change, snapshot ref change | fail read with stale availability unless profile permits refresh. |
| `dataset_version` | `24h` | dataset version ref or table-set checksum change | fail read. |
| `object_batch` | `12h` | object list, object checksum, or manifest checksum change | revalidate before read. |
| `partition_set` | `12h` | partition set, schema ref, or manifest checksum change | revalidate before read. |
| `manifest_list` | `6h` | manifest checksum or schema ref change | revalidate before read. |

### CDC-shaped feed metadata mapping

| CDC metadata | Cadastre placement | Authority effect | Validation fixture requirement |
| --- | --- | --- | --- |
| source offset, LSN, or binlog position | `RawRecord.source_event_metadata` and `StageStateRecord` | replay/progress only | positive and offset mismatch cases |
| schema-history ref | `CDCReplayStateContract` | replay sufficiency only | missing and stale schema-history cases |
| tombstone/delete marker | raw event subtype plus temporal/correction input | no retraction without `060` and `080` policy | tombstone non-authority case |
| heartbeat | `StageStateRecord` diagnostic | no fact time, absence, cleanup, or watermark by itself | heartbeat no-authority case |
| transaction metadata | raw metadata and replay ordering input | ordering only if policy permits | out-of-order case |

### Feed-fixture coverage matrix

| Fixture class | Required owner validation row |
| --- | --- |
| valid table snapshot | `120` feed-read success row |
| malformed manifest | `120` manifest rejection row |
| duplicate payload | `120` raw identity duplicate row |
| stale availability | `120` availability stale row |
| partial known gap | `120` no-absence row |
| permission-limited supplier evidence | `120` no-absence row |
| CDC tombstone | `120` CDC non-authority row |
| no direct source call | `120` direct-source negative row |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `020-CLEANUP-AC-001` | No banned reference class remains. |
| `020-CLEANUP-AC-002` | `LakehouseReadCompletenessReceipt` remains necessary but not sufficient for source-level absence, retraction, cleanup, graph expiry, or watermark advancement. |
| `020-CLEANUP-AC-003` | Table maintenance remains governed by `ReplayRetentionPolicy`, `TableMaintenancePolicy`, and `ReplayRetentionDecision`. |
| `020-CLEANUP-AC-004` | No direct enterprise source-call behavior is introduced. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `020-AC-001` | Raw import can be replayed from declared feed/table/object refs without enterprise source calls. |
| `020-AC-002` | Missing rows or objects never authorize absence without an imported completeness decision from `060`. |
| `020-AC-003` | Every production read and write has table-format-native refs when it can affect output or replay. |
| `020-AC-004` | Destructive maintenance is refused when any protected manifest, snapshot, replay window, or legal hold would be invalidated. |
| `020-AC-005` | Feed profile activation fails while feed feasibility, raw supplier profiles, read policy catalog, raw manifest schema, or feed fixture coverage is `TODO:` for the target feed. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
