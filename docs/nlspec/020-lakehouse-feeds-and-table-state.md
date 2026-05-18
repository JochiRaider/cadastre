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

- `RawRecord`
- `ComputeRawRecordId`
- `CoreRecordValidationAlgorithm`

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

`RawRecordImportRun` must supply every field required by `040.RawRecordSchema` and must import lakehouse raw feed rows or objects into deterministic `RawRecord` records that pass `040.ValidateCoreRecord` before commit. The import run must record feed profile ID, read policy ID, feed manifest ID, source dataset, scope keys, payload hash algorithm, import package artifact, input table or object refs, and every input required by `040.ComputeRawRecordId`.

RawRecord ID computation is owned by `040.ComputeRawRecordId`. `020` supplies feed, target, manifest, source dataset, scope, supplier identity, payload hash, and import profile inputs. `020` must not define a competing raw ID input order or collision policy and must import `RAW_RECORD_ID_COLLISION` from `040.CoreRecordErrorCodeSet`.

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
8. Persist only `RawRecord` rows that pass `040.ValidateCoreRecord`, or persist deterministic import errors.
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

### LakehouseReadPolicy payload limits

| Field | Required behavior | Default | Bounds |
| --- | --- | --- | --- |
| `max_payload_byte_length` | Maximum raw payload bytes a read/import step may accept for one `RawRecord`. | `104857600` bytes | Minimum `1`; maximum `1073741824`; feed profile may set a lower bound. |
| `raw_byte_canonicalization_mode` | Byte handling before `canonical_payload_hash` computation. | `none` | Closed enum: `none`; future modes require new schema version and fixtures. |
| `payload_ref` | Must reference declared table, object, partition, manifest, or dataset inputs. | none | Must not contain credentials, concrete private routes, or raw payload bytes. |

Raw payload bytes remain in raw/bronze lakehouse storage. `RawRecord` persists bounded metadata, refs, byte lengths, checksums, visibility state, and lineage only.

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

#### RawFeedManifest nested object schemas

`RawFeedManifest` nested objects must use the following closed shapes before manifest ID or checksum computation. Omitted optional arrays default to `[]`; omitted required nested objects fail with `RAW_FEED_MANIFEST_INVALID`.

##### `object_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `object_ref_id` | Yes | Stable object reference scoped to the manifest; not an object-store credential or private route. |
| `object_uri_hash` | Yes | SHA-256 lowercase hex over the redacted canonical URI reference. |
| `byte_count` | Yes | Non-negative `uint64`. |
| `checksum_algorithm` | Yes | Default and only active value `sha256`. |
| `object_checksum` | Yes | SHA-256 lowercase hex over object bytes after declared compression boundary. |
| `media_type` | Yes | Declared MIME-like media type token or `application/octet-stream`. |
| `compression` | Yes | Closed token; default `none`. |
| `encryption_ref` | No | Omitted when not encrypted; present value must be a redacted reference. |
| `redaction_state` | Yes | Closed token imported from `110` redaction policy. |

##### `partition_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `partition_key` | Yes | Canonical JSON object; keys sorted lexically. |
| `partition_bounds` | Yes | Canonical JSON object describing inclusive/exclusive bounds. |
| `schema_ref_id` | Yes | Must match one `schema_refs[].schema_ref_id`. |
| `partition_checksum` | Yes | SHA-256 lowercase hex over canonical partition inventory or table-format-native partition ref. |
| `row_count` | Yes | Non-negative `uint64`; `0` is valid only when the profile permits empty partitions. |
| `byte_count` | Yes | Non-negative `uint64`. |

##### `schema_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `schema_ref_id` | Yes | Stable manifest-local schema reference. |
| `schema_family` | Yes | Vendor-neutral schema family or `unknown`. |
| `schema_version` | Yes | Exact source schema version or `unknown`. |
| `schema_checksum` | Yes | SHA-256 lowercase hex over schema bytes or canonical schema descriptor. |
| `schema_uri_hash` | Yes | SHA-256 lowercase hex over the redacted schema URI reference. |

##### `time_bounds`

| Field | Required | Rule |
| --- | ---: | --- |
| `source_time_from` | No | RFC3339 UTC or omitted when unavailable. |
| `source_time_to` | No | RFC3339 UTC or omitted when unavailable. |
| `supplier_collection_from` | No | RFC3339 UTC or omitted when unavailable. |
| `supplier_collection_to` | No | RFC3339 UTC or omitted when unavailable. |
| `lakehouse_commit_time` | Yes | RFC3339 UTC table/object commit or manifest materialization time; not fact time by itself. |
| `quality` | Yes | Closed quality token; `unknown` is allowed only when profile permits. |

##### `supplier_lineage`

| Field | Required | Rule |
| --- | ---: | --- |
| `supplier_batch_id` | Yes | Supplier-scoped batch ID or canonical null sentinel when unavailable. |
| `supplier_run_id` | Yes | Supplier-scoped run ID or canonical null sentinel when unavailable. |
| `supplier_profile_id` | Yes | Must match the manifest's `supplier_profile_id`. |
| `redacted_access_ref_hash` | Yes | SHA-256 lowercase hex over the redacted access-reference descriptor. |

### ComputeRawFeedManifestId

```text
ComputeRawFeedManifestId(manifest):
1. Materialize manifest defaults.
2. Exclude `manifest_id` and `manifest_checksum`.
3. Serialize the remaining manifest object with `040.CanonicalJSON`.
4. Return `rfm_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.
5. If a prior different canonical manifest byte string has the same ID, emit `RAW_FEED_MANIFEST_ID_COLLISION` and commit no manifest or raw import output.
```

`manifest_checksum` must be SHA-256 lowercase hex over `040.CanonicalJSON` bytes for the materialized manifest excluding only `manifest_checksum`. `manifest_id` is included in `manifest_checksum`.

### RawRecord identity import

`020` imports `040.RawRecordSchema`, `040.ComputeRawRecordId`, and `040.CoreRecordValidationAlgorithm`. The former `020` raw ID input-order table is consolidated into `040.CoreRecordIdPolicy`. Raw import fixtures in `120` must compute expected raw IDs through `040.ComputeRawRecordId`.

| Import concern | Required behavior |
| --- | --- |
| ID input source | `RawRecordImportRun` supplies only the inputs named by `040.ComputeRawRecordId`. |
| Validation order | Schema validation and ID/checksum validation happen before raw persistence, absence evaluation, cleanup, or watermark decisions. |
| Collision behavior | `RAW_RECORD_ID_COLLISION` is imported from `040`; `020` must not define a local collision code. |
| Private binding | `payload_ref` and supplier metadata must fail with `PRIVATE_BINDING_LEAK` when they expose private routes, credentials, or tenant host lists. |

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

| Fixture class | Required fixture ID | Required owner validation row | Expected outcome |
| --- | --- | --- | --- |
| minimal valid object-batch manifest | `feed-020-valid-object-batch-manifest` | `val-020-manifest-valid-object-batch` | Manifest ID and checksum match expected TODO checksum. |
| malformed manifest | `feed-020-malformed-manifest` | `val-020-manifest-malformed` | `manifest_invalid`; no raw import commit. |
| manifest checksum mismatch | `feed-020-manifest-checksum-mismatch` | `val-020-manifest-checksum-mismatch` | `manifest_invalid`; no raw import commit. |
| omitted object for table snapshot | `feed-020-table-omitted-object` | `val-020-table-omitted-object` | Fail read unless declared subset profile permits partial. |
| omitted object for object batch | `feed-020-object-batch-known-gap` | `val-020-object-batch-known-gap` | `read_partial_known_gap`; no absence authority. |
| raw ID replay | `feed-020-raw-id-replay` | `val-020-raw-id-replay` | Expected raw ID computed through `040.ComputeRawRecordId`. |
| raw ID collision | `feed-020-raw-id-collision` | `val-020-raw-id-collision` | `RAW_RECORD_ID_COLLISION`; commit no colliding record. |
| no absence from manifest | `feed-020-no-absence-from-manifest` | `val-020-no-absence-from-manifest` | Missing object or row yields no absence without `060` gates. |
| CDC tombstone | `feed-020-cdc-tombstone-non-authority` | `val-020-cdc-tombstone-non-authority` | No retraction without `060` and `080` policy. |
| no direct source call | `feed-010-direct-source-call` | `neg-010-direct-source-call` | `DIRECT_SOURCE_CALL_FORBIDDEN`. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `020-CLEANUP-AC-001` | No banned reference class remains. |
| `020-CLEANUP-AC-002` | `LakehouseReadCompletenessReceipt` remains necessary but not sufficient for source-level absence, retraction, cleanup, graph expiry, or watermark advancement. |
| `020-CLEANUP-AC-003` | Table maintenance remains governed by `ReplayRetentionPolicy`, `TableMaintenancePolicy`, and `ReplayRetentionDecision`. |
| `020-CLEANUP-AC-004` | No direct enterprise source-call behavior is introduced. |
| `020-SCHEMA-PATCH-AC-001` | `020` contains no normative duplicate of `RawRecord` field schema or ID input order. |
| `020-SCHEMA-PATCH-AC-002` | Raw import produces only records that pass `040.ValidateCoreRecord`. |
| `020-SCHEMA-PATCH-AC-003` | `LakehouseReadPolicy` declares max payload byte length and raw byte canonicalization mode. |
| `020-SCHEMA-PATCH-AC-004` | Raw payload refs cannot expose private routes, credentials, or raw payload bytes in public artifacts. |

| `020-DOMAIN-CONSISTENCY-AC-001` | `domain.md` routes raw feed manifest and raw record ID behavior to `020` and `040` without marking them unresolved or restating `040` ID input order. |
| `020-MANIFEST-AC-001` | `ComputeRawFeedManifestId` and `manifest_checksum` produce byte-identical values across replay from the same manifest bytes. |

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
