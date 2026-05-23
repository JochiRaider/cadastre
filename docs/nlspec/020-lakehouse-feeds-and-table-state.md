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
- `ActivationControlledArtifactRef`
- `EvidenceRef`
- `EvidenceArtifactIdKindRegistry`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ResolveScopedRow`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `060.CoverageDomainToken`
- `060.ValidateCoverageDomainTokenArray`
- `060.CoverageDomainCatalog`
- `080.GoldFactPredicateContractRow`

## Exports

- `RawSupplierProfile`
- `LakehouseFeedProfile`
- `LakehouseFeedProfileSchema`
- `LakehouseFeedFeasibilityAssessment`
- `SourceDatasetCatalogRow`
- `SourceDatasetCatalogRowSet`
- `ResolveSourceDatasetCatalogRow`
- `SourceDatasetCatalogErrorCodeSet`
- `MVPSourceDatasetPredicateSupportClosure`
- `LakehouseFeedCategoryClosureRow`
- `LakehouseFeedCategoryClosureRowSet`
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
- `DecideTableMaintenance`
- `RawFeedManifestRuntimeStateSchema`
- `LakehouseTableStateErrorRegistryFragment`
- `FeedStageLifecycleEventDerivation`

## Feed Read Contract

A production feed read must start from an active `LakehouseFeedProfile`. The profile must name the supplier class, read target kind, scope keys, schema refs, object or table refs, package bindings, replay-relevant configuration hashes, and redacted access-reference hashes.

A production feed read must validate the profile against `LakehouseFeedProfileSchema`, must resolve the profile `source_dataset` through exactly one active `SourceDatasetCatalogRow` or exactly one deterministic source-dataset block row, and must resolve exactly one active `LakehouseFeedCategoryClosureRow` from an active `LakehouseFeedCategoryClosureRowSet` before reading or importing raw records. The phrase `profile permits` is not runtime behavior. Every profile-dependent branch must resolve to a named profile field, an active activation-controlled artifact ref, a deterministic branch result, and a validation row.

| Read target kind | Required reference | Output allowed |
| --- | --- | --- |
| `table_snapshot` | `LakehouseSnapshotRef` | Raw feed rows and read completeness receipt. |
| `dataset_version` | `DatasetVersionRef` | Raw feed rows and read completeness receipt. |
| `object_batch` | `RawFeedManifest` object refs | Raw feed objects and read completeness receipt. |
| `partition_set` | `RawFeedManifest` partition refs | Raw feed rows and read completeness receipt. |
| `manifest_list` | `RawFeedManifest` | Manifest validation result and read completeness receipt. |

`LakehouseFeedAvailabilityCheck` must validate catalog, table, object, partition, manifest, and schema availability without enterprise source calls or observation-producing side effects.

### LakehouseFeedProfileSchema

`LakehouseFeedProfileSchema` is the stable schema for every `LakehouseFeedProfile`. A profile with a missing required field must fail before lakehouse availability checks, feed reads, raw import, completeness evaluation, absence evaluation, cleanup, graph expiry, retraction, or watermark advancement.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `feed_profile_id` | Yes | none | Stable profile identifier scoped to `020`; must not encode a private source route. |
| `feed_category` | Yes | none | Closed category token from `LakehouseFeedCategoryClosureRequirementTable`; unknown tokens fail closed until an active category closure row set and validation rows exist. |
| `source_category` | Yes | none | Vendor-neutral source category; private vendor/product names are forbidden in public profile bytes. |
| `source_dataset` | Yes | none | Vendor-neutral lower-snake-case dataset token used by `060` authority, coverage, staleness, and completeness rows. Production activation requires a selected `SourceDatasetCatalogRow` ref and checksum. |
| `source_dataset_catalog_row_ref` | Yes for production activation | none | Structured `030.ActivationControlledRowRef` for the selected `SourceDatasetCatalogRow` or deterministic source-dataset block row. Bare string refs fail before read/import. |
| `source_dataset_catalog_row_checksum` | Yes for production activation | none | SHA-256 of the selected row after defaults materialize. A mismatch fails before read/import, completeness evaluation, mapping activation, source-authority closure, graph handoff, analysis handoff, API filtering, or validation acceptance. |
| `supplier_profile_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = raw_supplier_profile`. |
| `read_target_kind` | Yes | none | One of `table_snapshot`, `dataset_version`, `object_batch`, `partition_set`, or `manifest_list`; omission fails with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE`. |
| `read_policy_ref` | Yes | none | Active `030.ActivationControlledArtifactRef` with `artifact_class = lakehouse_read_policy`. |
| `scope_key_schema` | Yes | none | Declares the feed-owned dimensions used to materialize `020.LakehouseFeedScopeSelectorContext`. Empty scope key schema is forbidden for production feeds. |
| `absence_sensitive_domains` | No | `[]` | Array of `060.CoverageDomainToken` values. Omission defaults to `[]`. Values must validate through `060.ValidateCoverageDomainTokenArray`, reject duplicates, sort lexically before checksum computation, and must not include display labels or aliases. Empty means the feed may preserve positive observations but must not support absence-sensitive effects. |
| `partial_read_policy` | No | `positive_records_only` | Closed enum: `reject_read`, `positive_records_only`, `declared_subset_required`. The default permits positive raw import from known partial reads only and forbids absence-sensitive effects. |
| `empty_scope_policy` | No | `not_authoritative_for_absence` | Closed enum: `not_authoritative_for_absence`, `empty_complete_requires_060`, `reject_empty_scope`. |
| `availability_check_required` | No | `true` | `false` is allowed only for validation fixtures and must not be used for production reads. |
| `availability_refresh_policy` | No | `fail_if_stale` | Closed enum: `fail_if_stale`, `refresh_before_read`; refresh may validate lakehouse artifacts only. |
| `upstream_completeness_required` | No | derived | Materializes to `true` when `absence_sensitive_domains` is non-empty or the category row requires upstream evidence; otherwise `false`. |
| `coverage_profile_refs` | No | `[]` | Refs to active `060.CoverageDimensionProfile` rows required for absence-sensitive domains. Missing required refs block effects. |
| `source_authority_profile_refs` | No | `[]` | Refs to active `060.SourceAuthorityProfileRow` row sets. Missing refs block absence-sensitive effects. |
| `source_staleness_policy_refs` | No | `[]` | Refs to active `060.SourceStalenessPolicy` rows required by category and effect. |
| `parser_mapping_refs` | No | `[]` | Parser and mapping artifact refs required before raw records may advance past import into parsing or normalization. |
| `fixture_refs` | Yes | none | Non-empty refs to `120.LakehouseFeedFixture` rows covering the category and target kind. |
| `validation_refs` | Yes | none | Non-empty refs to passing validation rows for profile schema, branch behavior, category closure, and private-binding leak rejection. |
| `activation_scope` | Yes | none | `030.ActivationScope` for the feed profile. It must not contain concrete tenant inventories, routes, credentials, or private source lists. |

Profile validation must materialize defaults before checksum computation. Unknown fields fail before activation unless the owning profile schema version declares an extension map.

### SourceDatasetCatalogRowSet

`SourceDatasetCatalogRowSet` is the activation-controlled public row catalog for `source_dataset` token identity. It is represented by `030.ActivationControlledArtifactRef` with `artifact_class = source_dataset_catalog_row_set`. The catalog is vendor-neutral. It must not contain concrete vendor names, product names, private source routes, credentials, tenant inventories, scanner sites, directory tenant lists, cloud account lists, host lists, private schema payloads, or raw private fixture bytes.

`SourceDatasetCatalogRow` is the row-level public contract for one allowed `source_dataset` token. Concrete production rows are activation-controlled artifacts or package-supplied artifacts. Core prose must not embed private production row instances.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_set_id` | Yes | none | Stable catalog row-set ID. |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable version included in row checksum. |
| `source_dataset` | Yes | none | Vendor-neutral lower-snake-case token. It must not encode vendor, route, table path, tenant, scanner site, account, host list, source-native product, or private inventory. |
| `feed_category` | Yes | none | One token from `LakehouseFeedCategoryClosureRequirementTable`. |
| `source_category` | Yes | none | Vendor-neutral source category. |
| `scope_key_schema_ref` | Yes | none | Exact schema ref for required public scope dimensions. |
| `allowed_read_target_kinds` | Yes unless deterministically blocked | `[]` only when `deterministic_block_code` is present | Canonical set from the closed read-target kind set. Duplicate values fail. |
| `supported_fact_predicate_refs` | Yes | `[]` only when the row is positive-raw-only or deterministically blocked | Canonical set of active `080.GoldFactPredicateContractRow` refs. |
| `coverage_domain_tokens` | Yes | `[]` only when no absence-sensitive coverage can be supported | Canonical set of `060.CoverageDomainToken` values. Duplicates fail. Unsupported feed-category/token pairs fail with `SOURCE_DATASET_UNSUPPORTED_FOR_FEED_CATEGORY` before activation. |
| `absence_sensitive_effects` | Yes | `[]` | Canonical set selected from `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. Empty means positive-only for source-effect behavior. |
| `default_missing_row_result` | Yes | none | One of `unknown`, `not_authoritative_for_absence`, `no_op`, or an owner deterministic error. It must not be `authorized_absent`. |
| `private_binding_policy` | Yes | `public_row_no_private_binding` | The public row cannot contain private bindings. Concrete bindings are private-only and must not alter row selection, checksums, missing-row behavior, error precedence, activation scope, or validation requirements. |
| `deterministic_block_code` | Required when intentionally blocked | null only for active non-block rows | Exact owner code used when the dataset is intentionally blocked. A block row must emit no absence-sensitive mutation. |
| `validation_refs` | Yes | none | Non-empty source-dataset catalog validation refs. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

#### SourceDatasetCatalogErrorCodeSet

| Error code | Required use |
| --- | --- |
| `SOURCE_DATASET_CATALOG_ROW_MISSING` | No active row or deterministic block row matches the requested `source_dataset`, source category, feed category, and activation scope. |
| `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` | More than one equally specific active row matches after scoped resolution. |
| `SOURCE_DATASET_CATALOG_ROW_INACTIVE` | A matching row exists but lifecycle status is not `active`. |
| `SOURCE_DATASET_CATALOG_ROW_CHECKSUM_MISMATCH` | The selected row checksum, row-set checksum, package-set checksum, or manifest checksum does not match. |
| `SOURCE_DATASET_CATALOG_PRIVATE_BINDING_LEAK` | A public catalog row exposes a private binding value, private inventory, route, credential, raw private fixture byte, or private schema payload. |
| `SOURCE_DATASET_UNSUPPORTED_FOR_FEED_CATEGORY` | The catalog row names a feed-category/token or coverage-domain combination not permitted by `FeedCategoryToCoverageDomainTokenMapping`. |
| `SOURCE_DATASET_DETERMINISTICALLY_BLOCKED` | The selected row intentionally blocks the dataset or effect and emits no absence-sensitive mutation. |
| `SOURCE_DATASET_CATALOG_ROW_TODO` | The selected row, validation ref, package ref, or manifest ref contains `TODO:`. |
| `SOURCE_DATASET_CATALOG_ROW_UNVALIDATED` | The selected row lacks non-empty passing validation refs for the requested scope. |

#### ResolveSourceDatasetCatalogRow

```text
ResolveSourceDatasetCatalogRow(request, active_catalog):
1. Validate row-set activation ref, row-set checksum, package-set ref when supplied, lifecycle status, and validation refs.
2. Normalize request scope with `030.NormalizeScopeSelector` under `020.LakehouseFeedScopeSelectorContext`.
3. Match exact `source_dataset`, `source_category`, `feed_category`, and activation scope.
4. If zero rows match, fail with `SOURCE_DATASET_CATALOG_ROW_MISSING` and emit no absence-sensitive effect.
5. If more than one maximal row matches, fail with `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS` and emit no absence-sensitive effect.
6. If the selected row is inactive, checksum-mismatched, package-set-mismatched, out of scope, `TODO:`-bearing, private-leaking, or unvalidated, fail with the most specific `SourceDatasetCatalogErrorCodeSet` code.
7. If the selected row has `deterministic_block_code`, return `SOURCE_DATASET_DETERMINISTICALLY_BLOCKED`, the selected block row ref/checksum, and no absence-sensitive effect.
8. Include the selected row ref, row checksum, row-set ref, row-set checksum, selector checksum, validation refs, and package-set ref when package-supplied in `030.VersionManifest`.
9. Return the selected row and manifest inclusion requirement.
```

The resolver is deterministic. It must not infer a dataset token from a runtime route, table path, package name, source-native product, OCSF class, coverage-domain token, feed category, validation fixture name, or private binding.

### MappingSourceDatasetCatalogHandoff

Any `050.ObservationToOCSFMappingRow`, `050.ExternalSchemaProfile`, `050.SourceExtensionFieldRule`, or OCSF validation fixture that filters, selects, blocks, or validates by `source_dataset` must include the selected `020.SourceDatasetCatalogRow` ref/checksum or the exact deterministic source-dataset block row ref/checksum.

| Mapping source-dataset condition | Required behavior |
| --- | --- |
| Selected active source-dataset row | Mapping activation may proceed only when the row ref, row checksum, row-set ref, row-set checksum, selector checksum, validation refs, package-set refs when package-supplied, and `VersionManifest` refs are present. |
| Missing or ambiguous row | Mapping activation and silver output fail before `ResolveOCSFMapping` emits output. |
| Inactive, checksum-mismatched, private-leaking, package-set-mismatched, unvalidated, unmanifested, or `TODO:` row | OCSF mapping activation, source-extension activation, and validation acceptance fail with the most specific source-dataset or mapping owner error. |
| Deterministic block row | Emits no silver output unless the owner validation row expects a deterministic no-op or error and proves mutation prohibition. |
| Bare `source_dataset` string | Insufficient for row selection, checksum, validation acceptance, package activation, API filtering, graph handoff, or replay. |

OCSF mapping rows must not use private source routes, source-native products, feed profile names, package labels, source table paths, OCSF classes, coverage-domain tokens, or validation fixture names as row-selection authority for `source_dataset`.

#### SourceDatasetDeterministicBlockDefaults

Concrete public source-dataset rows are not supplied by this core spec. Until a selected production scope supplies active catalog rows or exact deterministic block rows, every referenced `source_dataset` resolves to `SOURCE_DATASET_CATALOG_ROW_MISSING` and no absence, cleanup, retraction, graph expiry, watermark, control pass/fail, source-history no-change proof, graph delta, compliance negative output, or source-effect validation pass may be emitted.

A selected deterministic source-dataset block row is a terminal no-effect decision for the exact dataset, scope, and effect named by the row. It must emit no read target, absence, cleanup, retraction, graph expiry, watermark, pass/fail, no-change proof, graph delta, compliance negative output, package activation effect, or validation pass except the mutation-prohibition validation evidence required by `120`.

### MVPSourceDatasetPredicateSupportClosure

`MVPSourceDatasetPredicateSupportClosure` defines the public MVP source-dataset support closure for predicate-catalog handoff. It instantiates `SourceDatasetCatalogRow` and does not embed private routes, vendor products, tenant inventories, scanner sites, account lists, host lists, private schema payloads, or raw fixture bytes.

| Field | Required value or behavior |
| --- | --- |
| `row_set_id` | `sdc-mvp-public-source-datasets-v1`. |
| `row_set_lifecycle_status` | Production use requires `active`. |
| `source_dataset_catalog_row_set_ref` | Required and manifest-included. |
| `row_set_checksum` | Required before production feed activation, mapping activation, source-authority closure, graph/analysis handoff, API filtering, validation acceptance, or replay output can depend on any selected row. |
| `package_set_ref` | Required when the row set is package-supplied. |
| `validation_refs` | Must include `120-SOURCE-DATASET-CATALOG-*`, `120-GOLD-PREDICATE-CATALOG-*`, `120-SOURCE-CLOSURE-*`, `120-PACKAGE-*`, and `120-VERSION-MANIFEST-*` rows. |

The active MVP public source-dataset rows must support only the exact `080` predicate rows listed below. A dataset row may list a strict subset when product governance keeps the missing predicate family out of the selected production scope. It must not list a predicate by fact-type string alone, wildcard, source-native product term, OCSF class, coverage-domain token, private route, or package label.

| Public `source_dataset` token | Feed category | Required `supported_fact_predicate_refs` | Required default effect posture |
| --- | --- | --- | --- |
| `endpoint_inventory` | `endpoint_inventory` | `gfp-mvp-host-id-resolved-to-canonical-v1`, `gfp-mvp-host-id-has-identifier-v1`, `gfp-mvp-host-attr-has-os-v1`, `gfp-mvp-host-attr-lifecycle-v1`, `gfp-mvp-host-attr-management-v1`, `gfp-mvp-user-logged-on-to-v1`, `gfp-mvp-user-used-device-v1` | Positive observations only unless exact `060` rows authorize absence-sensitive effects. |
| `configuration_inventory` | `configuration_inventory` | `gfp-mvp-host-attr-has-os-v1`, `gfp-mvp-host-attr-lifecycle-v1`, `gfp-mvp-host-attr-management-v1`, `gfp-mvp-host-software-runs-software-v1` | Positive observations only unless exact `060` rows authorize absence-sensitive effects. |
| `vulnerability_scan` | `vulnerability_scan` | `gfp-mvp-host-vuln-has-vulnerability-v1` | Coverage-sensitive; absence or fixed-state effects require exact `060` coverage, completeness, staleness, authority, and absence rows. |
| `control_evaluation` | `control_evaluation` | `gfp-mvp-control-failed-v1`, `gfp-mvp-control-passed-v1`, `gfp-mvp-control-unknown-v1` | Control output requires exact `060.ControlResultMappingRow` refs. |
| `directory_inventory` | `directory_inventory` | `[]` | Resolver input only. May support raw import, silver observation, and `070.IdentityEvidenceItem` candidate generation when `070` resolver rows cover the scope. It must not emit gold predicate output, membership facts, nonmembership, deletion, source absence, cleanup, graph expiry, or graph edge removal by itself. |
| `directory_membership` | `directory_membership` | `gfp-mvp-identity-member-of-v1` | Requires `070` group resolver coverage and exact `060` membership authority rows. |
| `dns_record_set` | `dns_record_set` | `gfp-mvp-host-dns-has-dns-name-v1`, `gfp-mvp-host-dns-resolved-to-ip-v1` | DNS absence or TTL-derived effects require exact `060` staleness and authority rows. |
| `dhcp_ipam_assignment` | `dhcp_ipam_assignment` | `gfp-mvp-host-ip-had-ip-v1` | DHCP/IPAM lease expiry must not imply host absence without exact `060` rows. |
| `network_flow` | `network_flow` | `gfp-mvp-flow-observed-connection-v1` | Positive observed flow only. Absence-sensitive flow effects remain blocked unless a future exact `060` row permits them. |
| `cloud_asset_inventory` | `cloud_asset_inventory` | `gfp-mvp-host-id-resolved-to-canonical-v1`, `gfp-mvp-host-id-has-identifier-v1`, `gfp-mvp-host-attr-lifecycle-v1`, `gfp-mvp-host-attr-management-v1`, `gfp-mvp-exposure-observed-v1` | Positive observations only unless exact `060` rows authorize absence-sensitive effects. |
| `source_history` | `source_history` | `[]` | Supporting evidence only. No no-change proof, disappearance proof, negative output, cleanup, retraction, graph expiry, watermark, or compliance negative output is allowed without exact `060.SourceHistoryRetentionProfile`, coverage, staleness, completeness, authority, absence, validation, package-set, and manifest refs. |
| `future_reachability` | deterministic block row | `[]` | `deterministic_block_code = REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`; no production read target, no fact, no graph edge, no graph property, and no API reachability claim. |

`network_flow` may support `gfp-mvp-flow-observed-connection-v1` only for positive observed traffic. `network_flow` must not support flow absence, no-change proof, observed-connection absence edges, theoretical reachability, service access, or identity-conditioned access in MVP.

Rows with `TODO:` in selected row refs, checksums, validation refs, expected outputs, mutation-prohibition proofs, or package-set refs are not active for gold output. They may remain as planning blockers only. `ValidateSpecSet` must classify them as `blocked_todo` and promotion must fail if their dataset is in selected gold-derivation scope.

### MVPSourceDatasetCatalogClosureInventory

The MVP public source-dataset inventory is closed to the tokens in this table. Production selection must resolve each referenced token through exactly one active `020.SourceDatasetCatalogRow` or exactly one deterministic source-dataset block row. Runtime table paths, feed profile names, feed categories, coverage-domain tokens, OCSF classes, source extension fields, package labels, package names, private routes, source-native products, and private bindings must not infer `source_dataset` identity.

| Public `source_dataset` token | Required row presence | Permitted production use | Gold predicate support | Default absence-sensitive posture | Deterministic block code when blocked | Required `060` closure refs when active | Package-set requirement when package-supplied | Validation row family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `endpoint_inventory` | Active catalog row required when selected. | `raw_import`, `silver_observation`, positive gold candidates, resolver evidence when `070` permits. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Positive-only unless exact `060` rows authorize the requested effect. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for absence, cleanup, retraction, graph expiry, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-FEED-CLOSURE-*`; `120-SOURCE-CLOSURE-*`. |
| `configuration_inventory` | Active catalog row required when selected. | `raw_import`, `silver_observation`, positive configuration and software gold candidates. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Positive-only unless exact `060` rows authorize the requested effect. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for configuration absence, cleanup, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-FEED-CLOSURE-*`; `120-SOURCE-CLOSURE-*`. |
| `vulnerability_scan` | Active catalog row required when selected. | `raw_import`, `silver_observation`, vulnerability gold candidates. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Coverage-sensitive; fixed or absent output requires exact `060` closure. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for vulnerability absence, fixed state, cleanup, retraction, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`; `120-COVERAGE-DOMAIN-*`. |
| `control_evaluation` | Active catalog row required when selected. | `raw_import`, `silver_observation`, control result candidates. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Control pass/fail/unknown requires exact control-result mapping and source closure. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for pass, fail, unknown, not-checked, not-applicable, absence, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`; `120-ERROR-REGISTRY-*`. |
| `directory_inventory` | Active catalog row required for resolver input; deterministic block row required for gold-output attempts. | `raw_import`, `silver_observation`, `resolver_input_only`. | `[]` for gold output. | Not authoritative for absence, deletion, membership, or nonmembership. | `DIRECTORY_INVENTORY_GOLD_OUTPUT_BLOCKED`. | None for gold output; exact `070` resolver rows required for identity evidence. | Required when package-supplied. | `120-SOURCE-DATASET-CATALOG-*`; `120-IDENTITY-CLOSURE-*`. |
| `directory_membership` | Active catalog row required when selected. | `raw_import`, `silver_observation`, positive membership gold candidates when `070` and `060` permit. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Membership absence and nonmembership require exact `060` closure and hidden-permission coverage. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for membership absence, cleanup, retraction, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-IDENTITY-CLOSURE-*`; `120-SOURCE-CLOSURE-*`. |
| `dns_record_set` | Active catalog row required when selected. | `raw_import`, `silver_observation`, DNS positive facts. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | DNS absence or TTL-derived effects require exact `060` closure. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for DNS absence, staleness, cleanup, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`. |
| `dhcp_ipam_assignment` | Active catalog row required when selected. | `raw_import`, `silver_observation`, positive IP assignment facts. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Lease expiry must not imply host absence without exact `060` closure. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for assignment absence, cleanup, retraction, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`. |
| `network_flow` | Active catalog row required for observed flow; deterministic block row required for flow-absence attempts. | Positive observed traffic only. | `gfp-mvp-flow-observed-connection-v1` only. | Flow non-observation, missing role evidence, ambiguous role evidence, and OCSF endpoint order are blocked by default. | `FLOW_ABSENCE_BLOCKED_MVP`. | Required only for positive fact authority or future explicitly active flow absence rows. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-GRAPH-PROFILE-CLOSURE-*`; `120-OCSF-DIRECTION-*`. |
| `cloud_asset_inventory` | Active catalog row required when selected. | `raw_import`, `silver_observation`, positive cloud asset facts. | Must match `MVPSourceDatasetPredicateSupportClosure` row for the same token. | Cloud disappearance, cleanup, and graph expiry require exact `060` closure. | `SOURCE_DATASET_CATALOG_ROW_MISSING` when omitted. | Required for disappearance, cleanup, graph expiry, source-history consult, or watermark. | Required. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`; `120-GRAPH-PROFILE-CLOSURE-*`. |
| `source_history` | Active catalog row may exist only as supporting evidence unless exact no-change closure exists. | `supporting_evidence_only`. | `[]` for gold output by default. | No no-change proof, disappearance proof, or negative output by silence or outside-window history. | `SOURCE_HISTORY_NO_CHANGE_PROOF_BLOCKED`. | Required for any future no-change proof: retention, coverage, staleness, completeness, authority, absence, validation, and manifest refs. | Required when package-supplied. | `120-SOURCE-DATASET-CATALOG-*`; `120-SOURCE-CLOSURE-*`; `120-COVERAGE-DOMAIN-*`. |
| `future_reachability` | Exact deterministic block row required while `200` is `inactive_deferred`. | Inactive future-domain material or validation-only negative fixture input. | `[]`. | No production read target, fact, graph edge, graph property, API claim, package activation effect, or production reachability validation pass. | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`. | None while deferred. | Required when package-supplied. | `120-SOURCE-DATASET-CATALOG-*`; reachability-prohibition rows. |

### LakehouseFeedScopeSelectorContext

`LakehouseFeedScopeSelectorContext` is the owner context family for `020` feed scopes. It instantiates `030.ScopeSelectorContext`; it does not define selector schema, equality, coverage, subset matching, specificity, or ambiguity behavior.

| Feed operation | Required context dimensions | Optional context dimensions | Default subset behavior | Required behavior |
| --- | --- | --- | --- | --- |
| feed read request | `source_category`, `source_dataset`, `read_target_kind` | `feed_profile_id`, `supplier_profile_ref`, `object_batch_ref`, `partition_set_ref`, `manifest_ref`, `dataset_version_ref`, `table_snapshot_ref` | none | Materialize an exact `030.ScopeSelector` before availability, read, import, and completeness decisions. |
| object batch read | `source_category`, `source_dataset`, `read_target_kind`, `object_batch_ref` | `supplier_profile_ref`, `manifest_ref` | none | Object refs remain lakehouse refs and must not contain private source routes. |
| partition set read | `source_category`, `source_dataset`, `read_target_kind`, `partition_set_ref` | `dataset_version_ref`, `table_snapshot_ref` | none | Missing required partition scope fails before read completeness output. |
| dataset version read | `source_category`, `source_dataset`, `read_target_kind`, `dataset_version_ref` | `table_snapshot_ref` | none | Dataset version scope is exact by default. |
| manifest list read | `source_category`, `source_dataset`, `read_target_kind`, `manifest_ref` | `supplier_profile_ref` | none | Manifest validation and read completeness use the same selector checksum. |

Feed diagnostics field `scope_selector_checksum` must contain the normalized `030.ScopeSelector.selector_checksum`. Raw selector values remain forbidden in diagnostics when they are private under `010.ScopeSelectorPublicBindingRule`.

Every selected feed profile, source-dataset catalog row or deterministic block row, feed category closure row, read policy row, completeness profile row, and feed-scope context that can affect output must add the selected row ref, row checksum, `ScopeSelectorContext.context_ref`, request selector checksum, selected row selector checksum, and activation artifact ref to `030.VersionManifest`.

### StructuredInputRepositoryFeedProfileHandoff

Repository-authored `LakehouseFeedProfile`, `RawSupplierProfile`, `LakehouseReadPolicy`, `SourceDatasetCatalogRowSet`, and feed category closure row catalogs are inert until materialized into activation-controlled artifact refs and, when package-supplied, included in an active `100.ProductionPackageSetManifest`.

`StructuredInputRepositorySnapshot` must not be a `read_target_kind`. The closed `read_target_kind` set remains `table_snapshot`, `dataset_version`, `object_batch`, `partition_set`, and `manifest_list`.

Repository-authored feed profile validation order is:

```text
ValidateRepositoryAuthoredFeedProfile(candidate_profile, repository_snapshot, version_manifest):
1. Validate `030.StructuredInputRepositorySnapshot` and selected path refs.
2. Validate `030.StructuredInputRepositoryTemplateContract` when the repository profile declares `template_required = true`.
3. Validate `030.StructuredInputRepositoryCIContract` for the exact snapshot when producer CI evidence is supplied or accepted by the repository profile.
4. Validate `030.StructuredInputValidationRun` for the exact snapshot, selected paths, file manifest checksum, template refs, CI refs, and validation matrix refs.
5. Validate private-binding redaction through `010` and `110` before materialization.
6. Validate `LakehouseFeedProfileSchema`.
7. Resolve feed category closure rows and lakehouse read policy refs.
8. Validate `100.StructuredInputPublicationManifest` when feed profile artifacts or source-dataset row catalogs were published by a remote repository.
9. Validate `030.StructuredInputCandidateSyncRecord` only as candidate discovery or audit evidence; sync evidence must not activate feed behavior.
10. Require package release and package-set refs when the feed profile or source-dataset row catalog is package-supplied.
11. Run `LakehouseFeedAvailabilityCheck` only after steps 1 through 10 pass.
12. Require materialized activation refs and `030.VersionManifest` inclusion before production feed read or raw import.
```

Repository-authored feed profiles must still use vendor-neutral `source_category` and `source_dataset`. They must not encode concrete private routes, credentials, tenant IDs, private inventories, scanner site names, directory tenant inventories, cloud account lists, host lists, private source names, or raw private fixture bytes.

A repository merge, pull request approval, branch update, validation run, or hook success must not authorize feed read, raw import, absence evaluation, cleanup, graph expiry, retraction, watermark advancement, or source-authority closure. Those effects require active `020`, `060`, and `030` refs and the relevant owner validation refs.

| Error code | Required use |
| --- | --- |
| `FEED_PROFILE_REPOSITORY_SNAPSHOT_MISSING` | Repository-authored feed profile validation lacks an exact `030.StructuredInputRepositorySnapshot` ref or selected path manifest checksum. |
| `FEED_PROFILE_REPOSITORY_PRIVATE_BINDING_LEAK` | Repository-authored feed profile or validation output exposes a private binding or raw private fixture value. |
| `FEED_PROFILE_REPOSITORY_ACTIVATION_MISSING` | Repository-authored feed profile exists only in Git or validation output and lacks materialized activation refs, package-set refs when package-supplied, or `VersionManifest` refs. |
| `FEED_PROFILE_REPOSITORY_TEMPLATE_MISMATCH` | Repository-authored feed profile layout, generated-output roots, or declared artifact classes do not match the selected `030.StructuredInputRepositoryTemplateContract`. |
| `FEED_PROFILE_REPOSITORY_CI_STALE` | Producer CI evidence is missing, stale, or not bound to the exact commit, tree, selected paths, file manifest checksum, toolchain refs, and validation matrix refs. |
| `FEED_PROFILE_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest digest, size, media type, package type, materialization ref, validation ref, redaction ref, or release input checksum does not match feed profile artifacts. |
| `FEED_PROFILE_SYNC_RECORD_NONAUTHORITY` | A candidate sync record is used as feed activation, feed read permission, absence authorization, cleanup permission, graph-expiry permission, retraction permission, or watermark authority. |

### LakehouseFeedCategoryClosureRow

`LakehouseFeedCategoryClosureRowSet` is an activation-controlled artifact represented by `030.ActivationControlledArtifactRef` with `artifact_class = lakehouse_feed_category_closure_row_set`. The row set instantiates the stable category-closure contract; it must not define new read target kinds, receipt states, authority classes, omission states, or effect tokens.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to `LakehouseFeedCategoryClosureRowSet`; required for refs, checksums, and validation output. |
| `row_version` | Yes | none | Immutable owner version. |
| `source_dataset_allowlist_ref` | Yes | none | Canonical set of structured `030.ActivationControlledRowRef` objects for selected `SourceDatasetCatalogRow` rows or exact deterministic source-dataset block rows. Bare string refs fail before feed activation, source-authority closure, validation acceptance, or absence-sensitive effects. |
| `category_activation_state` | Yes | none | Closed enum: `active_for_effects`, `active_positive_only`, `deterministically_blocked`. |
| `feed_category` | Yes | none | Must be one category from `LakehouseFeedCategoryClosureRequirementTable`. |
| `allowed_read_target_kinds` | Yes | none | Non-empty array of closed read target kinds unless `deterministic_block_code` blocks the category. |
| `absence_sensitive_domains` | No | `[]` | Array of `060.CoverageDomainToken` values whose negative or not-observed outputs require `060` completeness, authority, coverage, and staleness gates. Omission defaults to `[]`. Values must validate through `060.ValidateCoverageDomainTokenArray`. |
| `required_upstream_evidence_classes` | No | `[]` | Supplier evidence classes that must be present before `060` may evaluate effects. |
| `required_coverage_domains` | No | `[]` | Array of `060.CoverageDomainToken` values required for effects. Omission defaults to `[]`. Values must be a subset of `FeedCategoryToCoverageDomainTokenMapping` unless the category is `configuration_inventory` and the active row explicitly declares the affected domain. Unsupported values fail with `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY`. |
| `required_authority_refs` | No | `[]` | Exact `060.SourceAuthorityProfileRow` refs or row-set refs required by the category. |
| `required_staleness_refs` | No | `[]` | Exact `060.SourceStalenessPolicy` refs required by the category. |
| `allowed_effects` | No | `[]` | Closed effect tokens imported from `060`: `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`. Empty means positive observations only. |
| `blocked_effects` | No | all effects not in `allowed_effects` | Map from blocked effect token to deterministic block code. Missing map entries fail closure validation. |
| `source_authority_closure_matrix_ref` | Required when any absence-sensitive effect is allowed | null only for positive-only or category-blocked rows | Ref to the active `060.SourceAuthorityClosureMatrixRowSet` or validation result for this category, effect, and scope. |
| `closure_row_checksum` | Yes | none | SHA-256 over the canonical row bytes after defaults materialize. |
| `required_060_closure_refs` | Required when `allowed_effects` is non-empty | `[]` only when the row permits positive observations only | Canonically sorted refs to required `060` row-set artifacts for completeness, authority, coverage, staleness, progress signals, control results, source history, absence policy, and watermark policy. |
| `effect_closure_requirements` | Yes | none | Total map over `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. For each allowed effect, the row must name exact required `060` artifact classes, row-set refs, validation row IDs, and manifest inclusion requirements. For each blocked effect, the row must name a deterministic block code and mutation-prohibition validation ref. |
| `visibility_profile_refs` | Required when category depends on permissions or hidden objects | `[]` | Exact `060.SupplierCollectionVisibilityProfile` refs required for permission-sensitive absence. |
| `control_result_mapping_required` | Required for `control_evaluation` | `false` for all other categories | `true` requires an active `060.ControlResultMappingRowSet` before control output. |
| `source_history_retention_required` | Required for `source_history` and history-backed cloud inventory | `false` | `true` requires `060.SourceHistoryRetentionProfile`. |
| `default_missing_row_result` | Yes | none | Deterministic result when a required upstream object, partition, row, or source-scope record is missing. It must not be `absence` by default. |
| `deterministic_block_code` | Required when category is blocked | null | Error or no-op code used when the category exists but is intentionally blocked for MVP. |
| `validation_refs` | Yes | none | Non-empty refs to category-specific positive and negative validation rows. |

A missing active row for a known feed category fails with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING`. A known category that product governance removes from MVP must still have an active closure row with `allowed_effects = []`, a deterministic block code, and validation refs.

`020.allowed_effects` is necessary but not sufficient. It permits `060` to evaluate the effect; it must not authorize the effect without exact active `060` closure rows.

`effect_closure_requirements` must be total for every closed effect token. Missing allowed-effect refs, missing blocked-effect codes, missing mutation-prohibition refs, or missing manifest inclusion rules fail closure validation and must not be interpreted as positive-only behavior.

### Feed category to source-authority closure handoff

A feed category with non-empty `absence_sensitive_domains` must not enter active production scope unless each absence-sensitive effect has exactly one of these closure paths:

| Closure path | Required behavior |
| --- | --- |
| validated closure row chain | The category names a validated `060.SourceAuthorityClosureMatrix` row chain for the effect, including authority, completeness, coverage, staleness, progress-signal, control-result, history, absence, and watermark artifacts when applicable. |
| deterministic block row | The category has `allowed_effects = []`, a deterministic block code, validation refs, and mutation-prohibition evidence proving no absence-sensitive effect can occur. |

Missing source-authority closure rows must block absence, cleanup, retraction, graph expiry, and watermark advancement. Missing rows must not be interpreted as permission to use broad source-category authority, default lakehouse read completeness, provider freshness, destination cleanup, graph derived-view state, or implementation-local policy.

### SourceAuthorityClosureImportBoundary

`020` owns feed-category closure and read/import eligibility. It imports `060.SourceAuthorityClosureMatrix` by exact name and defines no source-authority row schema, absence algorithm, staleness algorithm, coverage algorithm, control-result mapping, graph-expiry authorization, or watermark authorization.

### MVPFeedCategoryClosureCatalogCompleteness

The active `LakehouseFeedCategoryClosureRowSet` must contain exactly one row for every category in `LakehouseFeedCategoryClosureRequirementTable`.

| Condition | Required behavior |
| --- | --- |
| Known category row missing | Fail with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` before feed read/import. |
| Known category row duplicated | Fail with the owner duplicate-row validation code before feed read/import. |
| Category outside MVP but known to the table | Use `category_activation_state = deterministically_blocked` or `category_activation_state = active_positive_only` with all absence-sensitive effects blocked. |
| Missing row for any known category | Must not mean positive-only, absence, cleanup, graph expiry, retraction, or watermark permission. |
| Deterministic block row selected | Emit no raw-import side effect beyond owner-declared diagnostics and mutation-prohibition evidence for absence-sensitive effects. |

### Concrete Source Dataset and Feed Category Catalog Materialization Plan

Stable source-dataset and feed-category row schemas remain owned by this spec. Concrete active row instances may live only in registered supporting material or package-supplied row catalogs. The public spec must not invent private rows, private source bindings, raw fixture bytes, tenant inventories, scanner sites, account lists, host lists, table paths, or route names.

| Materialization target | Required closure |
| --- | --- |
| Every public MVP `source_dataset` token in `MVPSourceDatasetCatalogClosureInventory` | Exactly one active `SourceDatasetCatalogRow` or exactly one exact deterministic source-dataset block row. |
| Every feed category in `LakehouseFeedCategoryClosureRequirementTable` | Exactly one active `LakehouseFeedCategoryClosureRow` or exactly one exact deterministic category block row. |
| Every `effect_closure_requirements` map | Total over `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`; omitted effect keys fail with `LAKEHOUSE_FEED_CATEGORY_EFFECT_MAP_INCOMPLETE`. |
| Every `requires_060_closure` cell | Exact `060.SourceAuthorityClosureMatrixRow` refs/checksums or exact deterministic block row refs/checksums. A bare effect name, source-dataset token, validation summary, package label, or row-set checksum is not sufficient. |
| Concrete supporting material path | Registered supporting artifact path. If absent, the row-set must contain `TODO: registered supporting artifact path` and validation must classify the row set as blocked. |

Deterministic block rows must use exact block codes for the blocked scope. The defaults below apply until product governance supplies concrete rows.

| Blocked scope | Required deterministic block code | Required no-effect behavior |
| --- | --- | --- |
| `directory_inventory` direct gold output | `DIRECTORY_INVENTORY_GOLD_OUTPUT_BLOCKED` | No membership fact, nonmembership, deletion, absence, cleanup, graph expiry, graph edge removal, or gold output. |
| `network_flow` absence or non-observation | `FLOW_ABSENCE_BLOCKED_MVP` | No observed-connection absence edge, graph expiry, cleanup, retraction, watermark, or theoretical reachability. |
| `source_history` no-change proof without exact retention and coverage refs | `SOURCE_HISTORY_NO_CHANGE_PROOF_BLOCKED` | No no-change proof, negative output, cleanup, retraction, graph expiry, watermark, control state, or compliance negative output. |
| `future_reachability` while `200` is inactive | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | No read target, fact, graph edge, graph property, API claim, package activation effect, or production validation pass for reachability. |

Production feed activation, read/import, mapping activation, source-authority closure, graph handoff, API filtering, validation acceptance, and replay output must consume selected row refs/checksums. They must not consume token strings, feed profile names, private bindings, table names, package labels, validation summaries, or owner prose as row identity.

`TODO:` materialization tasks are blocking product-governance tasks, not implementation latitude. A selected row catalog with `TODO:` fixture checksums, expected-output checksums, package-set refs, validation refs, row refs, row checksums, mutation-prohibition refs, or registered supporting-artifact paths must fail promotion and emit no absence-sensitive mutation.

### MVP Feed Category Closure Catalog Requirements

The selected MVP scope must provide one active `LakehouseFeedCategoryClosureRowSet` ref or an exact deterministic block row for every category in `LakehouseFeedCategoryClosureRequirementTable`. This section does not embed private vendor rows; concrete row instances remain activation-controlled supporting material.

| Required member | Requirement |
| --- | --- |
| `source_dataset` catalog ref | Every `LakehouseFeedProfile.source_dataset` must resolve to a vendor-neutral dataset catalog ref or an exact deterministic block proof before feed read/import. |
| category row coverage | Exactly one `LakehouseFeedCategoryClosureRow` must resolve for every category in `LakehouseFeedCategoryClosureRequirementTable`. |
| `future_reachability` | Must resolve to a deterministic block row in MVP. |
| out-of-scope category or effect | Must resolve to a deterministic block row or an `active_positive_only` row with every absence-sensitive effect blocked. |
| package-supplied row set | Must include immutable package release refs, an active package-set ref, validation refs, and `030.VersionManifest` inclusion before production use. |

Every row must materialize `row_id`, `row_version`, `source_dataset_allowlist_ref`, `category_activation_state`, `feed_category`, `allowed_read_target_kinds`, `required_upstream_evidence_classes`, `effect_closure_requirements`, `validation_refs`, `activation_scope`, `lifecycle_status`, package-set ref when package-supplied, and row checksum before validation output is computed.

`effect_closure_requirements` must be total over the closed effect set:

| Effect token | Required closure behavior |
| --- | --- |
| `absence` | Requires exact `060` closure refs or a deterministic block row. |
| `cleanup` | Requires exact `060` closure refs or a deterministic block row. |
| `retraction` | Requires exact `060` closure refs or a deterministic block row. |
| `graph_expiry` | Requires exact `060` closure refs and `090` graph handoff validation, or a deterministic block row. |
| `watermark` | Requires exact `060.ProjectionWatermarkPolicy` refs or a deterministic block row. |

```text
ValidateMVPFeedCategoryClosure(row_set, source_dataset_catalog, selected_scope):
1. Validate row-set activation ref.
2. Validate source-dataset catalog ref or deterministic block proof.
3. Materialize defaults and compute row checksums.
4. For each category in LakehouseFeedCategoryClosureRequirementTable, resolve exactly one row.
5. Reject missing or duplicate rows.
6. Validate effect closure requirements for every allowed effect.
7. Require 060 closure refs or deterministic block refs for absence-sensitive effects.
8. Require package-set refs when package-supplied.
9. Require VersionManifest inclusion.
10. Emit closed, deterministically_blocked, or owner error.
```

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

### EvidenceRef lakehouse artifact handoff

`020` runtime records that are persisted Cadastre records may be referenced by `EvidenceRef.artifact_id.kind = cadastre_record_ref` with `artifact_checksum` equal to the referenced record checksum. Lakehouse external artifacts that are not Cadastre records must use `EvidenceRef.artifact_id.kind = lakehouse_artifact_ref` with `artifact_checksum` computed over immutable bytes or owner-declared canonical metadata bytes.

| Lakehouse artifact or record | Required `EvidenceRef.artifact_id.kind` | Required checksum basis |
| --- | --- | --- |
| `RawFeedManifest` persisted as a Cadastre record | `cadastre_record_ref` | Referenced record checksum. |
| `RawFeedManifest` referenced manifest or object bytes | `lakehouse_artifact_ref` | Immutable manifest or object bytes, or owner-declared canonical metadata bytes. |
| `LakehouseSnapshotRef` | `cadastre_record_ref` | Referenced record checksum. |
| `DatasetVersionRef` | `cadastre_record_ref` | Referenced record checksum. |
| `LakehouseCommitRef` | `cadastre_record_ref` | Referenced record checksum. |
| `LakehouseReadCompletenessReceipt` | `cadastre_record_ref` | Referenced record checksum. |
| `UpstreamCompletenessEvidence` | `cadastre_record_ref` | Referenced record checksum. |
| Raw object ref inside `RawFeedManifest` | `lakehouse_artifact_ref` | Immutable object bytes or owner-declared canonical metadata bytes. |
| Partition ref inside `RawFeedManifest` | `lakehouse_artifact_ref` | Immutable partition manifest bytes or owner-declared canonical metadata bytes. |

Raw payload bytes must not be inlined into `EvidenceRef`. Object-store path alone is not sufficient evidence identity; `artifact_checksum` is mandatory. Mutable table refs such as `latest`, branch name alone, unpinned catalog ref, unresolved object prefix, and unpinned partition prefix must not satisfy `EvidenceRef`. Output-affecting reads and writes must still be represented by `LakehouseSnapshotRef`, `DatasetVersionRef`, or `LakehouseCommitRef`.

## Feed and Table Volatility Classification

Feed and table contracts split stable behavior from activatable profiles and runtime state.

| Artifact or record | Volatility class | Authority class | Required production handling |
| --- | --- | --- | --- |
| `RawSupplierProfile` | `activation_controlled_artifact` | supporting evidence | Must be active before dependent feed profiles execute. |
| `LakehouseFeedProfile` | `activation_controlled_artifact` | supporting evidence | Must be referenced by `030.ActivationControlledArtifactRef`. |
| `LakehouseFeedProfileSchema` | `stable_core_contract` | runtime data input | Owned by `020`; package or profile rows must not redefine it. |
| `LakehouseFeedCategoryClosureRow` | `activation_controlled_artifact` | supporting evidence | Row inside an active `LakehouseFeedCategoryClosureRowSet`; must instantiate the stable closure schema only. |
| `LakehouseFeedCategoryClosureRowSet` | `activation_controlled_artifact` | supporting evidence | Must be active before category-dependent feed reads or absence-sensitive effects. |
| `LakehouseFeedAvailabilityCheck` | `runtime_state_record` | supporting evidence | Must be recorded when required by the active profile. |
| `LakehouseFeedFeasibilityAssessment` | `runtime_state_record` | activation evidence | Must record activation readiness or deterministic blocking reasons for the feed profile and category row. |
| `RawFeedManifest` | `runtime_state_record` | supporting evidence | Must be included in `VersionManifest` when output-affecting. |
| `LakehouseReadPolicy` | `activation_controlled_artifact` | supporting evidence | Must be active and manifest-recorded before read. |
| `RawRecordImportRun` | `runtime_state_record` | supporting evidence | Must record deterministic import inputs and refs. |
| `LakehouseReadCompletenessReceipt` | `runtime_state_record` | supporting evidence | Necessary but not sufficient for absence, cleanup, retraction, graph expiry, or watermark. |
| `UpstreamCompletenessEvidence` | `runtime_state_record` | supporting evidence | May be consumed only through `060` policy. |
| `LakehouseTableProfile` | `activation_controlled_artifact` | system of record table governance | Must be active before production table read/write. |
| `LakehouseSnapshotRef` | `runtime_state_record` | table-state evidence | Must identify table-format-native read state. |
| `DatasetVersionRef` | `runtime_state_record` | table-state evidence | Must identify logical dataset or table-set version. |
| `LakehouseCommitRef` | `runtime_state_record` | table-state evidence | Must identify attempted or completed write state. |
| `ReplayRetentionPolicy` | `activation_controlled_artifact` | replay governance | Must be active before retention-sensitive maintenance. |
| `TableMaintenancePolicy` | `activation_controlled_artifact` | maintenance governance | Must be active before destructive or rewrite maintenance. |
| `ReplayRetentionDecision` | `runtime_state_record` | maintenance evidence | Must be persisted for each maintenance candidate set. |
| `CrossTableCommitProfile` | `activation_controlled_artifact` | table-set governance | Must be active when coherent table-set reads are required. |
| `CatalogBranchPromotionPolicy` | `activation_controlled_artifact` | catalog promotion governance | Must be active when catalog versioning controls production visibility. |

Production feed reads and table maintenance decisions must fail closed when any required activation-controlled artifact is inactive, checksum-mismatched, outside activation scope, missing validation refs, or omitted from `VersionManifest`.

## Maintenance Safety

`TableMaintenancePolicy` must evaluate snapshot expiration, vacuum, cleaner, orphan deletion, checkpoint or transaction-log cleanup, object garbage collection, restore, rollback, compaction deletion, and catalog garbage collection before execution.

`ReplayRetentionDecision` must refuse destructive maintenance when candidate deletion or rewrite would invalidate any protected production `VersionManifest`, graph rebuild, replay window, legal hold, or retention policy.

### RunLock maintenance guard handoff

Destructive or rewrite table maintenance is output-affecting and must consume the `030` run-lock contract without redefining it.

| Condition | Required behavior |
| --- | --- |
| Destructive or rewrite maintenance attempt | Call `030.AssertRunLockHeldBeforeCommit` before the destructive commit. |
| Commit scope | Include table profile, candidate maintenance set checksum, replay-retention decision ref, and requested maintenance effect. |
| Successful maintenance commit | `LakehouseCommitRef` and `VersionManifest` must include the `030.RunLockCommitGuard` ref. |
| Lock loss before destructive commit | Emit imported `030.RUN_LOCK_LOST` or `030.RUN_LOCK_FENCING_TOKEN_STALE`; no destructive commit may occur. |
| Lock loss after safe no-mutation preflight | Diagnostics may be recorded only; no table mutation is implied. |
| Maintenance failure due to lock loss | Must not advance source, projection, graph-apply, or presence-only watermarks. |

## Feed Read Algorithm

```text
ReadLakehouseFeed(profile, read_policy, target_ref):
1. Validate `030.ActivationControlledArtifactRef` for `LakehouseFeedProfile`, `RawSupplierProfile`, `LakehouseReadPolicy`, `LakehouseFeedCategoryClosureRowSet`, and every required table, retention, maintenance, cross-table, or catalog policy artifact.
2. Validate `profile` against `LakehouseFeedProfileSchema` after default materialization.
3. Fail with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` when any required profile field, including `read_target_kind`, is omitted or invalid.
4. Resolve exactly one active `LakehouseFeedCategoryClosureRow` for `profile.feed_category` and fail with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` when no row exists.
5. Fail with the row's `deterministic_block_code` when the category row is intentionally blocked for MVP.
6. Fail with `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` when a branch depends on an unmaterialized profile field, missing closure row field, missing validation ref, or missing owner-defined result.
7. Verify no direct enterprise source endpoint, private route, credential, or environment-specific source list is referenced.
8. Verify `LakehouseFeedAvailabilityCheck` is current when `availability_check_required = true`; if stale, apply `availability_refresh_policy`.
9. Validate `target_ref` against `read_policy`, `profile.read_target_kind`, `allowed_read_target_kinds`, and checksum rules.
10. For a declared subset read, validate an active `030.DeclaredDAGSubsetProfile`; otherwise fail with `LAKEHOUSE_DECLARED_SUBSET_REQUIRED`.
11. Read only declared table snapshots, dataset versions, objects, partitions, or manifests.
12. Map every missing, unreadable, omitted, empty, or invalid target condition through `ReadTargetBranchPolicy`.
13. Persist read checkpoints as `StageStateRecord` when output-affecting.
14. Persist `RawRecordImportRun` inputs.
15. Persist only positive `RawRecord` rows that pass `040.ValidateCoreRecord` and are permitted by `partial_read_policy`; otherwise persist deterministic import errors.
16. Persist `LakehouseReadCompletenessReceipt` with the branch-selected receipt state.
17. Persist `VersionManifest` refs for every output-affecting activation artifact, profile, policy, target, schema, state, manifest, feasibility assessment, and category closure row set.
18. Return no absence, cleanup, retraction, graph expiry, or watermark result from this algorithm.
```

### FeedStageLifecycleEventDerivation

`FeedStageLifecycleEventDerivation` maps feed-read branch outcomes into `030.StageExecutionLifecycleMachine.v1` events. It must not restate the generic stage lifecycle transition matrix. Every feed-read branch must emit a `030.ProcessingStageLifecycleResult` or fail before production output.

| Feed read outcome | Lifecycle event | Terminal state expectation | Required side effects |
| --- | --- | --- | --- |
| Profile schema missing or invalid | `nonretryable_error` | `failed_nonisolated` unless stage isolation permits `failed_isolated` | No read, no raw import, no watermark. |
| No active category closure row | `nonretryable_error` | `failed_nonisolated` unless stage isolation permits `failed_isolated` | No read, no raw import. |
| Category closure deterministic no-op block | `stage_no_output` | `no_op` | Emit block/no-op evidence only. |
| Category closure deterministic error block | `nonretryable_error` | `failed_nonisolated` unless isolated | Emit owner error; no production output. |
| Branch unresolved | `nonretryable_error` | `failed_nonisolated` unless isolated | No read. |
| Declared subset missing | `nonretryable_error` | `failed_nonisolated` unless isolated | No subset output. |
| Declared subset output forbidden | `forbidden_output` | `failed_nonisolated` | No output commit. |
| Missing table, dataset, object, partition, or schema refs before read | `nonretryable_error` unless read policy declares the condition retryable | `failed_nonisolated` or `retry_wait` | No raw import until retry or success. |
| Manifest invalid | `nonretryable_error` | `failed_nonisolated` unless isolated | Receipt or diagnostic only; no raw import commit. |
| Partial known gap with positive records allowed | `stage_success` | `succeeded` | Persist raw positives, receipt, and `blocked_effects = [absence, cleanup, retraction, graph_expiry, watermark]`. |
| Partial unknown gap with positive records allowed | `stage_success` | `succeeded` | Persist raw positives, receipt, and `blocked_effects = [absence, cleanup, retraction, graph_expiry, watermark]`. |
| Empty target scope under `not_authoritative_for_absence` | `stage_success` | `succeeded` | Persist receipt and diagnostic; absence and watermark remain blocked. |
| Empty target scope under `reject_empty_scope` | `nonretryable_error` | `failed_nonisolated` unless isolated | No read output. |
| Successful full read | `stage_success` | `succeeded` | Persist raw rows or manifest validation, receipt, and state records. |
| Successful manifest-list validation that produces no raw rows | `stage_no_output` or `stage_success` as profile declares | `no_op` or `succeeded` | Persist receipt and validation refs; no absence-sensitive effects unless `060` later permits them. |

Feed-related `030.ProcessingStageLifecycleResult` usage must add `feed_receipt_state_ref`. The field must reference the persisted `LakehouseReadCompletenessReceipt` when the feed stage reaches `succeeded`, `no_op`, or an isolated failure after receipt emission.

`020` remains non-authoritative for absence, cleanup, retraction, graph expiry, and watermark authorization. Those effects may be unblocked only by `060` after the feed stage result records the blocked effects.

## Feed and Table State Contract Details

### LakehouseFeedFeasibilityAssessment

`LakehouseFeedFeasibilityAssessment` is exported by this spec. It is a runtime state record that proves activation readiness or records deterministic blocking reasons. A feed may become active only when the assessment exists, references the active feed profile and category closure row, and has `activation_result = pass`; otherwise activation is blocked.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `assessment_id` | Yes | none | Deterministic ID over feed profile ref, category row ref, policy refs, readiness statuses, and blocking reasons. |
| `feed_profile_ref` | Yes | none | Active `LakehouseFeedProfile` ref. |
| `feed_category_row_ref` | Yes | none | Active `LakehouseFeedCategoryClosureRow` ref from the active row set. |
| `read_policy_ref` | Yes | none | Active `LakehouseReadPolicy` ref. |
| `payload_fidelity_status` | Yes | none | Closed token: `pass`, `blocked`, `not_applicable`. |
| `metadata_sufficiency_status` | Yes | none | Must pass when required target refs, schema refs, manifest refs, and supplier lineage are required. |
| `timestamp_sufficiency_status` | Yes | none | Must pass before downstream temporal processing; `unknown` is not a pass. |
| `identifier_sufficiency_status` | Yes | none | Must pass when raw records can affect identity, target selectors, graph projection, or source authority. |
| `replayability_status` | Yes | none | Must pass when target refs, object refs, state refs, and manifest refs are retained for replay. |
| `fixture_coverage_status` | Yes | none | Must pass when `fixture_refs` cover category, target kind, branch behavior, and private-binding leak cases. |
| `completeness_evidence_status` | Yes | none | Must pass for absence-sensitive domains and may be `not_applicable` only when `absence_sensitive_domains = []`. |
| `parser_mapping_readiness_status` | Yes | none | Must pass before parser, mapping, normalization, or downstream output stages can use the feed. |
| `activation_result` | Yes | none | `pass` only when every required status is `pass` or explicitly `not_applicable`; otherwise `blocked_until_all_pass`. |
| `blocking_reasons` | Yes | `[]` | Required non-empty when `activation_result != pass`; sorted by blocking code then artifact ref. |

`activation_result = pass` is forbidden when any required dimension is omitted, `blocked`, `unknown`, stale, checksum-mismatched, outside activation scope, or missing validation refs.

### RawSupplierProfile

| Supplier class | Public/private status | Allowed metadata | Prohibited private fields | Validation behavior |
| --- | --- | --- | --- | --- |
| `external_transport_supplier` | public vendor-neutral class | supplier class, feed scope, delivery batch ref, redacted access-ref hash | concrete vendor route, credential, tenant host list | Reject private values with `PRIVATE_BINDING_LEAK`. |
| `lakehouse_export_supplier` | public vendor-neutral class | export window, object refs, schema refs, upstream completeness refs | source credentials, source API URLs | Require manifest validation. |
| `validation_fixture_supplier` | public redacted class | fixture ID, source category, redaction summary, checksum | raw private payload unless redacted fixture permits | Validation-only, no production evidence. |
| `private_bound_supplier` | private implementation artifact only | none in public docs | all concrete bindings | Public artifact must fail if present. |

### RawSupplierProfileRow schema

`RawSupplierProfile` is an activation-controlled row family. Concrete rows instantiate the supplier boundary only; they do not grant source authority, source completeness, identity authority, graph authority, or package activation authority by existence.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version included in `row_checksum`. |
| `supplier_profile_id` | Yes | none | Stable public ID; must not encode vendor, route, tenant, host list, credential, or private inventory. |
| `supplier_class` | Yes | none | Closed enum: `external_transport_supplier`, `lakehouse_export_supplier`, `validation_fixture_supplier`. Public use of `private_bound_supplier` fails with `PRIVATE_BINDING_LEAK`. |
| `allowed_feed_categories` | Yes | none | Canonical non-empty set of `LakehouseFeedCategoryClosureRequirementTable` tokens; duplicates fail; values sort lexically before checksum computation. |
| `allowed_read_target_kinds` | Yes | none | Canonical set over `table_snapshot`, `dataset_version`, `object_batch`, `partition_set`, and `manifest_list`. |
| `allowed_metadata_fields` | No | `[]` | Canonical set of public supplier metadata paths. Private route, credential, and raw payload paths are forbidden. |
| `prohibited_metadata_classes` | No | `private_route`, `credential`, `tenant_inventory`, `host_list`, `raw_payload_bytes`, `source_native_secret` | Canonical set; an active row may add classes but must not remove a default class. |
| `redacted_access_ref_hash_required` | No | `true` | Production feed reads must fail when a required redacted access reference hash is absent. |
| `upstream_completeness_evidence_classes` | No | `[]` | Canonical set of supplier evidence classes. Presence is evidence only and grants no authority by itself. |
| `availability_evidence_required` | No | `true` for production, `false` for validation-only fixtures | Production reads must validate lakehouse availability evidence before output. |
| `validation_refs` | Yes | none | Non-empty refs proving supplier class, private-binding rejection, redacted-hash behavior, metadata allowlist behavior, and manifest requirements. |
| `activation_scope` | Yes | none | `030.ActivationScope`; private binding values are forbidden. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Owner errors for this row family are `RAW_SUPPLIER_PROFILE_MISSING`, `RAW_SUPPLIER_PROFILE_INACTIVE`, `RAW_SUPPLIER_PROFILE_UNSUPPORTED_CLASS`, `PRIVATE_BINDING_LEAK`, `RAW_SUPPLIER_REDACTED_ACCESS_HASH_MISSING`, and `RAW_SUPPLIER_VALIDATION_REFS_MISSING`. A feed read that selects a missing, inactive, unsupported, private-leaking, checksum-mismatched, unvalidated, or unmanifested supplier profile must emit no raw output.

### ReadTargetBranchPolicy

The read target branch table is total for production feed reads. A branch not covered by this table must fail with `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` before read, import, completeness evaluation, absence evaluation, cleanup, retraction, graph expiry, or watermark advancement.

| Condition | Required receipt state | Required error or no-op | Output classes | VersionManifest requirement |
| --- | --- | --- | --- | --- |
| `read_target_kind` omitted or outside the closed enum | none | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | none | rejected profile validation refs only |
| `feed_category` has no active closure row | none | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | none | rejected profile and row-set refs |
| Category closure row has `deterministic_block_code` | none | row-defined deterministic block/no-op code | none | category row ref and validation refs |
| `table_snapshot` missing `LakehouseSnapshotRef`, schema ref, or required table profile ref | `read_unavailable` or `schema_unavailable` as applicable | hard read failure | receipt and diagnostics only | failed target refs and read policy ref |
| `dataset_version` missing `DatasetVersionRef`, schema refs, or table-set checksum | `read_unavailable` or `schema_unavailable` as applicable | hard read failure | receipt and diagnostics only | failed target refs and read policy ref |
| Manifest checksum, count, object shape, partition shape, or schema ref invalid | `manifest_invalid` | `manifest_invalid` | manifest validation result and receipt only | manifest ref and validation error refs |
| Manifest-listed object or partition is unreadable after retry and its identity is known | `read_partial_known_gap` | no absence, cleanup, retraction, graph expiry, or watermark | positive raw records only when `partial_read_policy = positive_records_only` | missing object or partition ref and checkpoint refs |
| Missing scope is suspected but the missing object, partition, row range, or table region cannot be identified | `read_partial_unknown_gap` | no absence, cleanup, retraction, graph expiry, or watermark | diagnostics and any already committed positive raw records permitted by policy | partial state and blocking reason refs |
| Declared subset requested without active `030.DeclaredDAGSubsetProfile` | none | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | none | subset request ref and rejected profile refs |
| Declared subset emits an output forbidden by `030.DeclaredDAGSubsetProfile` | none | `030.DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | none | subset profile ref and stage output refs |
| Empty target scope with `empty_scope_policy = not_authoritative_for_absence` | `read_complete` | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | receipt and diagnostics; no absence-sensitive effects | empty scope evidence and profile ref |
| Empty target scope with `empty_scope_policy = empty_complete_requires_060` | `read_complete` | no feed-read error; `060` must decide effects | receipt only | upstream evidence refs and completeness profile refs |
| Empty target scope with `empty_scope_policy = reject_empty_scope` | none | hard read failure | diagnostics only | profile and target refs |
| Successful full read of declared target | `read_complete` | none | raw rows or manifest validation, receipt, state records | all target, manifest, policy, category, and state refs |

### LakehouseFeedCategoryClosureRequirementTable

This table defines the MVP feed-category closure catalog. It does not activate a category row by itself. Each non-blocked category must have exactly one active `LakehouseFeedCategoryClosureRow` in the active row set before production use. Unknown or future categories fail closed until an active row set, `060` completeness rows, and `120` validation rows exist.

| Feed category | Allowed read target kinds in active row | Absence-sensitive uses | Required upstream evidence classes | Required `060` coverage domains | Default missing-row behavior | MVP result |
| --- | --- | --- | --- | --- | --- | --- |
| `endpoint_inventory` | Any closed read target kind when the active row permits it. | Endpoint non-observation, cleanup, graph expiry, watermark. | scope enumeration, permission visibility, collection window, failed-scope evidence. | endpoint | `not_authoritative_for_absence` | Requires active closure row. |
| `configuration_inventory` | Any closed read target kind when the active row permits it. | Configuration absence, stale configuration, cleanup, watermark. | configuration scope, collection window, failed-item evidence, source permission state. | Active row declares one or more `CoverageDomainToken` values. | `not_authoritative_for_absence` | Requires active closure row. |
| `vulnerability_scan` | Any closed read target kind when the active row permits it. | Vulnerability absence, fixed-state absence, scan watermark. | scanner target scope, credential/auth status, plugin/check set, scan window, failed-target and exclusion evidence. | vulnerability | `not_authoritative_for_absence` | Requires active closure row. |
| `control_evaluation` | Any closed read target kind when the active row permits it. | Control pass, fail, unknown, not checked, not applicable, watermark. | benchmark/check scope, applicability evidence, evaluation status, result mapping refs. | control | `not_authoritative_for_absence` | Requires active closure row. |
| `directory_inventory` | Any closed read target kind when the active row permits it. | Principal, group, device, and directory-object absence. | tenant/domain scope, page or delta completion, hidden-object permission state, failed-scope evidence. | directory | `not_authoritative_for_absence` | Requires active closure row. |
| `directory_membership` | Any closed read target kind when the active row permits it. | Group non-membership, user non-membership, membership cleanup, watermark. | tenant/domain, group, member type, direct/transitive mode, hidden-membership permission, page/delta completion, AD primary-group evidence. | directory | `not_authoritative_for_absence` | Requires active closure row. |
| `dns_record_set` | Any closed read target kind when the active row permits it. | DNS absence and stale DNS state. | zone/source scope, authoritative source evidence, TTL basis, collection window. | `dns` | `not_authoritative_for_absence` | Requires active closure row. |
| `dhcp_ipam_assignment` | Any closed read target kind when the active row permits it. | Lease absence, assignment absence, host cleanup, watermark. | DHCP/IPAM scope, lease window, authoritative-system evidence, failed-scope evidence. | `dhcp_ipam` | `not_authoritative_for_absence` | Requires active closure row. |
| `network_flow` | Any closed read target kind when the active row permits it. | Positive observed-flow evidence only by default. | sensor scope, collection point, time window, role evidence, packet/flow field completeness. | flow | `unknown`; missing flow must not imply no flow. | Requires active closure row; absence effects default blocked. |
| `cloud_asset_inventory` | Any closed read target kind when the active row permits it. | Cloud resource absence, deletion inference, graph expiry, watermark. | account/project/subscription, region, resource type, permission visibility, source-history evidence when required. | `cloud_inventory`; `source_history` when source-history evidence is consulted. | `not_authoritative_for_absence` | Requires active closure row. |
| `source_history` | Any closed read target kind when the active row permits it. | No-change proof only within supported source-native history window. | history window, retention profile, query scope, outside-window evidence. | `source_history` | `unknown`; outside-window no-result is not proof. | Requires active closure row. |
| `future_reachability` | none | none in MVP. | inactive deferred reachability evidence only. | `reachability` with deterministic block. | deterministic no-op/block. | Blocked for MVP with `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` or owner no-op. |

### FeedCategoryToCoverageDomainTokenMapping

This table maps feed-category tokens to imported `060.CoverageDomainToken` values. `020` owns the feed-category side of the mapping and does not define token grammar, aliases, token catalog membership, or coverage-domain error precedence. Unsupported category/token pairs fail before feed activation, raw import, completeness evaluation, absence-sensitive effects, package activation, and validation acceptance.

| Feed category | Required `060.CoverageDomainToken` values | Mapping rule |
| --- | --- | --- |
| `endpoint_inventory` | `endpoint` | Exact. |
| `configuration_inventory` | One or more tokens from `060.CoverageDomainCatalog` as declared by the active row. | No implicit default. Active row must identify the affected fact or predicate domain. |
| `vulnerability_scan` | `vulnerability` | Exact. |
| `control_evaluation` | `control` | Exact. |
| `directory_inventory` | `directory` | Exact. |
| `directory_membership` | `directory` | Exact. |
| `dns_record_set` | `dns` | Exact. |
| `dhcp_ipam_assignment` | `dhcp_ipam` | Exact. |
| `network_flow` | `flow` | Exact. |
| `cloud_asset_inventory` | `cloud_inventory`; `source_history` only when source-history evidence is consulted. | History token is conditional, not automatic. |
| `source_history` | `source_history` | Exact. |
| `future_reachability` | `reachability` | Deterministic block only while `200` remains inactive deferred. |

`future_reachability` remains a feed-category token only. It must not be accepted as a coverage-domain token. Feed-category validation must call `060.ValidateCoverageDomainTokenArray` before read/import and must fail with `COVERAGE_DOMAIN_UNSUPPORTED_FOR_FEED_CATEGORY` when an otherwise valid token is not permitted by this table.

### FeedCategoryToSourceAuthorityClosureRequirements

This table maps every feed category in `LakehouseFeedCategoryClosureRequirementTable` to the `060` artifact classes required before an absence-sensitive effect may execute. The table contains no concrete vendor, product, tenant, route, scanner-site, zone, account, or private inventory rows.

| Feed category | Required `060` artifact classes before absence-sensitive effects | Required `120` validation row family | Additional condition |
| --- | --- | --- | --- |
| `endpoint_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | `SupplierCollectionVisibilityProfile` is required when enrollment or inventory visibility is permission-limited. |
| `configuration_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Coverage domain is selected by the active row for the affected configuration fact. |
| `vulnerability_scan` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Scanner target, credential, plugin/check, and scan-window coverage must resolve through `060`. |
| `control_evaluation` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ControlResultMappingRowSet`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Pass, fail, unknown, not checked, and not applicable output require control-result mapping. |
| `directory_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Hidden-object and limited-information states block negative output unless exact rows authorize. |
| `directory_membership` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Direct/transitive mode, hidden membership, page completion, delta reset, and AD primary-group handling must resolve through exact rows. |
| `dns_record_set` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | TTL expiry maps to stale or unknown unless exact rows authorize a narrower effect. |
| `dhcp_ipam_assignment` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Lease expiry maps to stale or expired assignment state, not host absence, unless exact rows authorize. |
| `network_flow` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Missing flow defaults to `unknown`; `watermark` may be allowed only by exact rows, and absence effects remain blocked unless exact rows authorize a future narrower effect. |
| `cloud_asset_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | `cloud_inventory` is required; `source_history` and `SourceHistoryRetentionProfile` are required when source-history no-change or disappearance is consulted. |
| `source_history` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SourceHistoryRetentionProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Outside-window no-result must not become no-change proof; no-change proof requires source-history coverage plus retention. |
| `future_reachability` | deterministic block row only | `120-SOURCE-CLOSURE-*` deterministic block row only | MVP behavior is deterministic no-op or deferred reachability error; activation of any read target or absence-sensitive effect is prohibited. |

### MVPCategoryEffectDefaultPosture

This table is total for every feed category in `LakehouseFeedCategoryClosureRequirementTable` and every closed requested-effect token. A cell value is a default posture only; it does not authorize an effect. `requires_060_closure` means the effect may execute only after `020`, `060`, `030`, `100` when package-supplied, and `120` refs resolve. `positive_only`, `deterministic_block`, `validation_only`, and `inactive_deferred` emit no absence-sensitive mutation.

| Feed category | `absence` | `cleanup` | `retraction` | `graph_expiry` | `watermark` |
| --- | --- | --- | --- | --- | --- |
| `endpoint_inventory` | `requires_060_closure` | `requires_060_closure` | `deterministic_block` | `requires_060_closure` | `requires_060_closure` |
| `configuration_inventory` | `requires_060_closure` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `vulnerability_scan` | `requires_060_closure` | `deterministic_block` | `requires_060_closure` | `deterministic_block` | `requires_060_closure` |
| `control_evaluation` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `directory_inventory` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `directory_membership` | `requires_060_closure` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `dns_record_set` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `dhcp_ipam_assignment` | `requires_060_closure` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `network_flow` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `cloud_asset_inventory` | `requires_060_closure` | `requires_060_closure` | `deterministic_block` | `requires_060_closure` | `requires_060_closure` |
| `source_history` | `requires_060_closure` | `deterministic_block` | `deterministic_block` | `deterministic_block` | `requires_060_closure` |
| `future_reachability` | `inactive_deferred` | `inactive_deferred` | `inactive_deferred` | `inactive_deferred` | `inactive_deferred` |

### LakehouseReadPolicy payload limits

| Field | Required behavior | Default | Bounds |
| --- | --- | --- | --- |
| `max_payload_byte_length` | Maximum raw payload bytes a read/import step may accept for one `RawRecord`. | `104857600` bytes | Minimum `1`; maximum `1073741824`; feed profile may set a lower bound. |
| `raw_byte_canonicalization_mode` | Byte handling before `canonical_payload_hash` computation. | `none` | Closed enum: `none`; future modes require new schema version and fixtures. |
| `payload_ref` | Must reference declared table, object, partition, manifest, or dataset inputs. | none | Must not contain credentials, concrete private routes, or raw payload bytes. |

Raw payload bytes remain in raw/bronze lakehouse storage. `RawRecord` persists bounded metadata, refs, byte lengths, checksums, visibility state, and lineage only.

### LakehouseReadPolicyRow schema

`LakehouseReadPolicy` is an activation-controlled row family. A production read must resolve exactly one active row before reading a table snapshot, dataset version, object batch, partition set, or manifest list.

| Field | Required | Default | Bounds or rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `read_policy_id` | Yes | none | Stable public policy ID. |
| `max_payload_byte_length` | No | `104857600` | Integer range `1..1073741824`; feed profile may only narrow. |
| `raw_byte_canonicalization_mode` | No | `none` | Closed enum: `none` for MVP. |
| `metadata_timeout_seconds` | No | `30` | Integer range `1..300`. |
| `read_operation_timeout_seconds` | No | `300` | Integer range `1..3600`. |
| `whole_read_timeout_seconds` | No | `3600` | Integer range `1..86400`. |
| `max_retry_attempts` | No | `3` | Integer range `1..10`. |
| `retry_schedule` | No | `exponential_no_jitter` | Deterministic; jitter is forbidden unless a future policy records side-effect inputs. |
| `initial_retry_delay_ms` | No | `1000` | Integer range `0..60000`. |
| `max_retry_delay_ms` | No | `30000` | Integer range `1000..600000`; must be greater than or equal to `initial_retry_delay_ms`. |
| `projection_pushdown_policy` | No | `forbidden_for_raw_import` | Pushdown omission must not authorize absence. |
| `read_checkpoint_required` | No | derived | Materializes to `true` for production or replay-affecting reads. |
| `target_ref_requirements` | Yes | none | Total map over every read target kind in the table below. |
| `object_gap_behavior_by_target_kind` | Yes | none | Total map in the object-gap table below. |
| `retryable_error_classes` | Yes | default table below | Total map from error class to retryability. |
| `checksum_behavior` | Yes | default table below | Total map from checksum state to read result. |
| `failed_object_behavior` | Yes | default table below | Total map from failed-object state to receipt/result. |
| `omitted_object_behavior` | Yes | default table below | Total map from omitted-object state to receipt/result. |
| `deterministic_ordering_rule` | Yes | default table below | Defines row/object ordering before checksum and import output. |
| `validation_refs` | Yes | none | Non-empty refs for target kinds, bounds, retries, checksums, omitted object handling, and ordering. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

#### LakehouseReadPolicy target_ref_requirements

| Read target kind | Required refs |
| --- | --- |
| `table_snapshot` | `LakehouseSnapshotRef`, active `LakehouseTableProfile` ref/checksum, and schema ref/checksum. |
| `dataset_version` | `DatasetVersionRef`, table-set checksum, schema refs/checksums, and active `CrossTableCommitProfile` when consistency is required. |
| `object_batch` | `RawFeedManifest.object_refs[]` with object checksums, byte counts, media type, compression boundary, and schema refs. |
| `partition_set` | `RawFeedManifest.partition_refs[]` with partition bounds, schema refs, row counts, byte counts, and checksums. |
| `manifest_list` | `RawFeedManifest` ref/checksum and manifest validation result. |

#### LakehouseReadPolicy object gap and retry maps

| Map | Key | Required value |
| --- | --- | --- |
| `object_gap_behavior_by_target_kind` | each closed read target kind | `known_gap_to_partial_known`, `unknown_gap_to_partial_unknown`, or `gap_forbidden`. |
| `retryable_error_classes` | `transient_metadata_unavailable` | retryable until `max_retry_attempts` or timeout. |
| `retryable_error_classes` | `transient_object_unavailable` | retryable until `max_retry_attempts` or timeout. |
| `retryable_error_classes` | `checksum_mismatch` | nonretryable. |
| `retryable_error_classes` | `schema_unavailable` | nonretryable unless an active schema refresh policy is named. |
| `checksum_behavior` | `checksum_match` | continue. |
| `checksum_behavior` | `checksum_mismatch` | fail with `LAKEHOUSE_READ_CHECKSUM_MISMATCH`; no absence-sensitive effects. |
| `checksum_behavior` | `checksum_missing_required` | fail before output. |
| `failed_object_behavior` | `known_failed_object` | emit `read_partial_known_gap` and preserve positive raw output only when profile permits. |
| `failed_object_behavior` | `unknown_failed_object` | emit `read_partial_unknown_gap`; no absence-sensitive effects. |
| `omitted_object_behavior` | `object_not_listed_in_manifest` | fail with `LAKEHOUSE_READ_OMITTED_OBJECT`; no implicit absence. |
| `omitted_object_behavior` | `projection_pushdown_omitted_object` | forbidden for raw import. |

#### LakehouseReadPolicy deterministic ordering

Records must sort by the first available key in this order: `object_ref_id`, canonical `partition_key`, stable row ordinal, byte offset, and declared source row key. If no deterministic order exists, the read must fail before output with `LAKEHOUSE_READ_ORDER_UNDETERMINED`.

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

### RawFeedManifestRuntimeStateSchema

`RawFeedManifest` is a `020` runtime-state record. It must not be represented as an activation-controlled row. Manifest validation policy may be activation-controlled, but each manifest instance is runtime evidence from a feed read.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `manifest_schema_version` | Yes | none | Immutable runtime manifest schema version. |
| `source_dataset_catalog_row_ref` | Yes | none | Selected `020.SourceDatasetCatalogRow` or deterministic block row ref. |
| `source_dataset_catalog_row_checksum` | Yes | none | Must match selected row bytes after defaults. |
| `feed_category_row_ref` | Yes | none | Selected `LakehouseFeedCategoryClosureRow` ref. |
| `feed_category_row_checksum` | Yes | none | Must match selected category row. |
| `read_policy_ref` | Yes | none | Selected `LakehouseReadPolicy` activation artifact ref. |
| `read_policy_checksum` | Yes | none | Must match selected policy row or artifact checksum. |
| `table_profile_refs` | Required for table-backed reads | `[]` for object-only manifest lists | Canonical set of `LakehouseTableProfile` row refs/checksums. |
| `snapshot_ref` | Required for `table_snapshot` | null otherwise | Table-format-native snapshot ref/checksum. |
| `dataset_version_ref` | Required for `dataset_version` or coherent table sets | null otherwise | Dataset version ref/checksum. |
| `manifest_source_kind` | Yes | none | Closed enum below. |
| `manifest_entry_bounds` | Yes | none | Counts, byte totals, target count bounds, and allowed zero-row policy. |
| `canonical_array_semantics` | Yes | none | Must equal the array semantics table below. |
| `duplicate_entry_policy` | Yes | `reject` | Duplicate canonical keys fail with `RAW_FEED_MANIFEST_DUPLICATE_ENTRY`. |
| `manifest_validation_result` | Yes | none | Closed enum below. |
| `raw_import_allowed` | Yes | `false` unless validation result is `valid` and target permits import | Validation-only manifests must not import raw records. |
| `replay_materialization_refs` | Yes | `[]` | Canonical set of refs needed to recreate manifest reads. |
| `record_checksum` | Yes | none | SHA-256 over canonical runtime-state bytes excluding `record_checksum`. |
| `version_manifest_ref` | Required when output-affecting | none | Must be included in `030.VersionManifest` before raw import or replay-affecting output. |

Closed `manifest_source_kind` values are:

```text
supplier_manifest
cadastre_materialized_manifest
validation_fixture_manifest
```

Closed `manifest_validation_result` values are:

```text
valid
manifest_invalid
checksum_mismatch
schema_unavailable
private_binding_leak
bounds_exceeded
```

| Array | Semantics | Duplicate policy | Sort key |
| --- | --- | --- | --- |
| `object_refs[]` | `canonical_set` | `reject` | `object_ref_id` |
| `partition_refs[]` | `canonical_set` | `reject` | canonical `partition_key`, then `schema_ref_id` |
| `schema_refs[]` | `canonical_set` | `reject` | `schema_ref_id` |
| `upstream_completeness_refs[]` | `canonical_set` | `reject` | referenced record ID |

### LakehouseTableProfileRow schema

`LakehouseTableProfile` is the activation-controlled row family for authoritative table governance. A production read or write of an authoritative table must resolve exactly one active table profile before output.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `table_profile_id` | Yes | none | Stable public table profile ID. |
| `logical_table_role` | Yes | none | Closed enum below. |
| `table_format` | Yes | none | Closed enum below. |
| `table_format_version` | Yes | none | Exact table-format version or protocol version. |
| `catalog_binding_ref` | Yes | none | Redacted catalog binding ref; mutable branch labels alone are forbidden. |
| `catalog_binding_checksum` | Yes | none | SHA-256 over canonical catalog binding descriptor. |
| `table_native_identity_hash` | Yes | none | Hash over table-format-native table identity. |
| `table_location_hash` | Yes | none | Hash over redacted table location; raw private paths are forbidden. |
| `required_snapshot_identity_fields` | Yes | none | Canonical set from `TableFormatNativeIdentityMatrix`. |
| `required_commit_identity_fields` | Yes | none | Canonical set from `TableFormatNativeIdentityMatrix`. |
| `schema_compatibility_policy_ref` | Yes | none | Active schema compatibility policy or check ref. |
| `replay_required` | No | `true` for authoritative tables | `false` is allowed only for validation-only or non-authoritative tables. |
| `replay_retention_policy_ref` | Yes | none | Active `ReplayRetentionPolicy` row ref. |
| `maintenance_policy_ref` | Yes | none | Active `TableMaintenancePolicy` row ref. |
| `write_mode` | Yes | none | Closed owner token for append, replace, correction, maintenance, or validation-only write mode. |
| `delete_or_rewrite_guard_required` | No | `true` | Must be `true` for destructive or rewrite-capable maintenance. |
| `validation_refs` | Yes | none | Non-empty refs for format identity, schema compatibility, replay, retention, maintenance, and private-binding tests. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Closed `logical_table_role` values are:

```text
raw_bronze
silver
identity
gold
graph_delta
graph_apply
registry
health
validation
audit
```

Closed `table_format` values are:

```text
iceberg
delta
hudi
opaque_manifested_table
```

`opaque_manifested_table` may support positive raw import only. It must not satisfy replay, graph rebuild, destructive maintenance, authoritative gold correction, or table-set promotion until a future active profile defines full native identity.

### TableFormatNativeIdentityMatrix

| Format | Snapshot ref must include | Commit ref must include |
| --- | --- | --- |
| `iceberg` | snapshot ID, metadata file URI hash, metadata file checksum, manifest-list URI hash, manifest-list checksum, schema ID, partition spec ID, sort order ID, delete-file set checksum, catalog ref/checksum. | previous metadata pointer checksum, new metadata file checksum, commit operation, manifest-list checksum, catalog compare-and-swap evidence. |
| `delta` | table version, log range, checkpoint ref/checksum when used, protocol min reader/writer versions, table features, schema checksum, deletion-vector set checksum when used, catalog ref/checksum. | committed log version, action-set checksum, app transaction ref when present, protocol feature refs, checkpoint effect when present. |
| `hudi` | instant time, instant action, completed-state proof, timeline checksum, table config checksum, base-file set checksum, log-file set checksum, savepoint ref when protected. | instant time, action, prior timeline checksum, completed-state proof, rollback/restore refs when applicable. |
| `opaque_manifested_table` | manifest ref/checksum and schema checksum. | manifest write ref/checksum only; positive-read-only for MVP. |

### ReplayRetentionPolicyRow schema

`ReplayRetentionPolicy` is an activation-controlled row family. Until product governance supplies an active production retention duration, the default retention mode is `deterministic_block_all_maintenance`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `retention_policy_id` | Yes | none | Stable public policy ID. |
| `protected_artifact_classes` | Yes | none | Canonical set of manifest, snapshot, commit, schema, graph rebuild, and replay input classes protected by the policy. |
| `protected_run_classes` | Yes | none | Canonical set of run classes protected by the policy. |
| `retention_mode` | Yes | `deterministic_block_all_maintenance` | Closed enum below. |
| `duration_days` | Required when `retention_mode = duration_window` | null otherwise | Positive integer; omitted duration in duration mode blocks activation. |
| `legal_hold_refs` | No | `[]` | Canonical set; any present legal hold blocks destructive candidates. |
| `graph_rebuild_protection` | No | `true` | When `true`, graph rebuild input refs are deletion/rewrite blockers. |
| `partial_candidate_behavior` | No | `refuse_candidate_set` | Split eligibility must be explicit. |
| `outside_window_behavior` | No | `refuse_until_policy_active` | Outside-window candidates remain blocked until policy rows define eligibility. |
| `deletion_eligibility_rule` | Yes | none | Candidate is eligible only under the rule below. |
| `validation_refs` | Yes | none | Non-empty refs for legal hold, protected manifest, graph rebuild, replay window, and eligible-candidate cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Closed `retention_mode` values are:

```text
protect_all_manifested_refs
duration_window
legal_hold_only
deterministic_block_all_maintenance
```

A maintenance candidate is eligible only when it is not referenced by protected manifests, replay windows, graph rebuilds, legal holds, schema compatibility, table-state checksums, or active retention rules.

### TableMaintenancePolicyRow schema

`TableMaintenancePolicy` is an activation-controlled row family for destructive and rewrite-capable maintenance preflight.

| Field | Required | Default | Bounds or rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `maintenance_policy_id` | Yes | none | Stable public policy ID. |
| `allowed_actions` | No | `[]` | Canonical set from the closed enum below. Empty blocks all maintenance. |
| `candidate_enumeration_rule` | Yes | none | Deterministic rule that enumerates candidate refs before decision. |
| `candidate_set_checksum` | Yes | none | SHA-256 over canonical candidate set. |
| `replay_retention_policy_ref` | Yes | none | Active `ReplayRetentionPolicy` row ref. |
| `run_lock_required` | No | `true` | Destructive and rewrite-capable actions must require `030.RunLockCommitGuard`. |
| `commit_guard_ref_required` | No | `true` | Missing guard fails before mutation. |
| `max_candidates_per_decision` | No | `100000` | Integer range `1..1000000`. |
| `maintenance_timeout_seconds` | No | `3600` | Integer range `1..86400`. |
| `partial_eligibility_mode` | No | `refuse_whole_candidate_set` | Split eligibility must be explicit. |
| `idempotency_key_inputs` | Yes | none | Exact ordered inputs used to identify retries. |
| `no_candidate_behavior` | No | `emit_no_op_decision` | No candidates must not mutate table state. |
| `commit_preconditions` | Yes | none | Exact preconditions for mutation, including commit ref and run-lock guard. |
| `validation_refs` | Yes | none | Non-empty refs for default block, run lock, candidate checksum, partial eligibility, timeout, no-op, and commit-ref cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Closed `allowed_actions` values are:

```text
snapshot_expiration
vacuum
cleaner
orphan_delete
checkpoint_log_cleanup
object_gc
restore
rollback
compaction_rewrite
catalog_gc
```

### DecideTableMaintenance

```text
DecideTableMaintenance(request, table_profiles, retention_policy, maintenance_policy, candidate_set, run_lock_guard):
1. Validate active `LakehouseTableProfile`, `ReplayRetentionPolicy`, and `TableMaintenancePolicy` refs and checksums.
2. Validate requested action appears in `maintenance_policy.allowed_actions`.
3. Canonicalize the candidate set and verify `candidate_set_checksum`.
4. Validate `030.RunLockCommitGuard` for destructive or rewrite-capable actions.
5. Sort candidates by `table_profile_id`, `ref_kind`, `ref_id`, and `object_path_hash`.
6. Evaluate blockers in this order: legal hold, protected `VersionManifest`, protected graph rebuild input, replay retention, table-state checksum invalidation, schema compatibility, policy-specific blockers.
7. If any candidate is refused and `partial_eligibility_mode = refuse_whole_candidate_set`, emit `ReplayRetentionDecision(decision = refuse)` and perform no mutation.
8. If split eligibility is allowed, emit separate eligible and refused candidate sets and checksums.
9. Commit maintenance only after an eligible decision, valid run-lock guard, and valid `LakehouseCommitRef`.
10. Include policy refs, candidate checksum, retention decision, run-lock guard, commit ref, and package-set refs in `VersionManifest`.
```

### ReplayRetentionDecision schema

`ReplayRetentionDecision` is a runtime-state record. It must not be represented as an activation-controlled row.

| Field | Required | Rule |
| --- | ---: | --- |
| `decision_id` | Yes | Deterministic ID over policy refs, table refs, candidate set checksum, blocker summary, and decision. |
| `retention_policy_ref` | Yes | Selected `ReplayRetentionPolicy` ref/checksum. |
| `maintenance_policy_ref` | Yes | Selected `TableMaintenancePolicy` ref/checksum. |
| `table_profile_refs` | Yes | Canonical set of selected `LakehouseTableProfile` refs/checksums. |
| `candidate_set_checksum` | Yes | SHA-256 over sorted candidates. |
| `eligible_candidate_refs` | Yes | Canonical set; empty unless `decision` permits mutation or no candidates exist. |
| `refused_candidate_refs` | Yes | Canonical set with refusal reasons. |
| `refusal_reasons` | Yes | Canonical map from refused candidate ref to ordered blocker reasons. |
| `decision` | Yes | Closed enum below. |
| `mutation_prohibition_refs` | Required for `refuse` and `no_op` | Proof that no mutation occurred. |
| `checksum` | Yes | SHA-256 over canonical decision bytes excluding `checksum`. |
| `version_manifest_ref` | Required when output-affecting | Must include decision and every consulted ref. |

Closed `decision` values are:

```text
eligible
refuse
eligible_partial
no_op
```

### CrossTableCommitProfileRow schema

`CrossTableCommitProfile` is an activation-controlled profile required whenever a production run, replay, rebuild, validation, or promotion requires coherent multi-table state.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `cross_table_profile_id` | Yes | none | Stable public profile ID. |
| `required_table_profile_refs` | Yes | none | Canonical non-empty set of `LakehouseTableProfile` refs/checksums. |
| `consistency_mode` | Yes | none | Closed enum in table below. |
| `atomicity_requirement` | Yes | none | Closed enum in table below. |
| `catalog_semantics` | Yes | `none` | Closed enum in table below. |
| `table_set_checksum_algorithm` | No | `sha256` | Only `sha256` is active for MVP. |
| `snapshot_set_sort_key` | No | `table_profile_id` | Duplicates rejected. |
| `missing_table_behavior` | No | `fail_before_output` | Missing required tables must fail before output. |
| `mixed_snapshot_behavior` | No | `fail_before_output` unless `manifested_snapshot_set` and checksum match | Mixed snapshots fail before output unless explicitly manifested. |
| `dataset_version_ref_required` | No | `true` | Production table sets require dataset version refs. |
| `validation_refs` | Yes | none | Non-empty refs for missing table, mixed snapshot, checksum mismatch, and coherent accepted cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

| Field | Values |
| --- | --- |
| `consistency_mode` | `single_catalog_commit`, `manifested_snapshot_set`, `same_catalog_branch_commit`, `opaque_dataset_version` |
| `atomicity_requirement` | `atomic_required`, `consistent_snapshot_set_required`, `validation_only`; production table-set output cannot use `validation_only`. |
| `catalog_semantics` | `none`, `branch_commit`, `tag_commit`, `transaction_id`, `dataset_version_manifest` |

### CatalogBranchPromotionPolicyRow schema

`CatalogBranchPromotionPolicy` is an activation-controlled row family. It governs visibility transitions only when catalog versioning controls production visibility.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the row set. |
| `row_version` | Yes | none | Immutable schema/content version. |
| `promotion_policy_id` | Yes | none | Stable public policy ID. |
| `catalog_profile_ref` | Yes | none | Redacted catalog profile ref/checksum. |
| `promotion_source_ref_kind` | Yes | none | Closed enum below. |
| `source_commit_ref` | Yes | none | Immutable commit ref; mutable branch or tag alone is forbidden. |
| `target_visibility_ref` | Yes | none | Ref for production visibility target. |
| `cross_table_commit_profile_ref` | Required when table-set consistency is required | null otherwise | Selected `CrossTableCommitProfile` row ref/checksum. |
| `required_validation_refs` | Yes | none | Non-empty validation refs that must pass before promotion. |
| `approval_refs` | Yes | none | Non-empty for production visibility changes. |
| `rollback_target_requirement` | Yes | none | Rollback target must be immutable and checksum-valid. |
| `target_advanced_behavior` | No | `preserve_current_production_visibility` | If target advanced unexpectedly, preserve current production visibility and fail candidate promotion. |
| `post_promotion_verification` | Yes | none | Required post-promotion checksum and visibility verification refs. |
| `promotion_lock_required` | No | `true` | Production promotion requires a lock/guard. |
| `failure_behavior` | No | `preserve_current_production_visibility` | Candidate failure must not advance target visibility. |
| `validation_refs` | Yes | none | Non-empty refs for immutable ref, approval, rollback, target advanced, and preserve-current cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Closed `promotion_source_ref_kind` values are:

```text
commit
tag_resolved_to_commit
branch_resolved_to_commit
```

Mutable branch or tag alone is forbidden. A catalog promotion success changes visibility only through the refs required by this row and by `030.VersionManifest`; it does not create source authority.

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
| source or CDC heartbeat | `StageStateRecord` diagnostic | no fact time, absence, cleanup, or watermark by itself; distinct from `030` run-lock heartbeat | heartbeat no-authority case |
| transaction metadata | raw metadata and replay ordering input | ordering only if policy permits | out-of-order case |

### Feed-fixture coverage matrix

| Fixture class | Required fixture ID | Required owner validation row | Expected outcome |
| --- | --- | --- | --- |
| activation artifacts | `feed-020-activation-artifacts` | `val-020-activation-artifacts` | Inactive feed profile, stale read policy artifact, table profile checksum mismatch, maintenance policy omitted from `VersionManifest`, and replay retention policy inactive during maintenance fail before output or destructive action. |
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
| profile schema incomplete | `feed-020-profile-schema-incomplete` | `val-020-feed-profile-schema-incomplete` | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE`; no read. |
| missing category closure row | `feed-020-category-row-missing` | `val-020-category-row-missing` | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING`; no read. |
| unresolved profile branch | `feed-020-profile-branch-unresolved` | `val-020-profile-branch-unresolved` | `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED`; no output. |
| declared subset required | `feed-020-declared-subset-required` | `val-020-declared-subset-required` | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED`; no output. |
| empty scope not authoritative | `feed-020-empty-scope-not-authoritative` | `val-020-empty-scope-not-authoritative` | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE`; no absence or watermark. |
| feed category closure catalog | `feed-020-category-closure-*` | `val-020-category-closure-*` | Each MVP category has one positive or deterministic block row and one missing-row negative case. |

### Feed activation artifact errors

| Error code | Emitted when |
| --- | --- |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_MISSING` | A required feed, table, read, retention, maintenance, cross-table, or catalog policy artifact ref is missing. |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_INACTIVE` | A required artifact exists but lifecycle status is not allowed for production execution. |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | A required artifact checksum differs from the active artifact ref or manifest. |
| `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | A feed profile omits a required `LakehouseFeedProfileSchema` field, uses an invalid target kind, or leaves a required branch input unresolved. |
| `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | No active `LakehouseFeedCategoryClosureRow` covers the profile's `feed_category`. |
| `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | A feed read branch still depends on unmaterialized profile or closure-row policy. |
| `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | A declared subset read is requested or required but no active `030.DeclaredDAGSubsetProfile` covers the request. |
| `UPSTREAM_COMPLETENESS_EVIDENCE_REQUIRED` | The profile or category row requires upstream completeness evidence and the manifest or evidence refs omit it. |
| `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | An empty target scope is read under the default empty-scope policy and must not be interpreted as source absence. |

### LakehouseErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_MISSING` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-missing` |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_INACTIVE` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-inactive` |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | `020` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-checksum-mismatch` |
| `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | `020` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-feed-profile-schema-incomplete` |
| `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-feed-category-row-missing` |
| `LAKEHOUSE_FEED_CATEGORY_ROW_AMBIGUOUS` | `020` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-feed-category-row-ambiguous` |
| `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-profile-branch-unresolved` |
| `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-declared-subset-required` |
| `UPSTREAM_COMPLETENESS_EVIDENCE_REQUIRED` | `020` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-upstream-completeness-evidence-required` |
| `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | `020` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-empty-scope-not-authoritative` |
| `RAW_FEED_MANIFEST_INVALID` | `020` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-raw-feed-manifest-invalid` |
| `RAW_FEED_MANIFEST_ID_COLLISION` | `020` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-raw-feed-manifest-id-collision` |

### LakehouseTableStateErrorRegistryFragment

This owner fragment extends `LakehouseErrorRegistryFragment` for read-policy, supplier, manifest runtime-state, table profile, retention, maintenance, cross-table, and catalog-promotion failures. Every row must be generated into `110.ErrorCodeRegistryRow` before caller-visible diagnostics may be emitted.

| Error group | Required codes or code families |
| --- | --- |
| Raw supplier | `RAW_SUPPLIER_PROFILE_MISSING`, `RAW_SUPPLIER_PROFILE_INACTIVE`, `RAW_SUPPLIER_PROFILE_UNSUPPORTED_CLASS`, `PRIVATE_BINDING_LEAK`, `RAW_SUPPLIER_REDACTED_ACCESS_HASH_MISSING`, `RAW_SUPPLIER_VALIDATION_REFS_MISSING`. |
| Read policy | `LAKEHOUSE_READ_POLICY_MISSING`, `LAKEHOUSE_READ_TIMEOUT`, `LAKEHOUSE_READ_RETRY_EXHAUSTED`, `LAKEHOUSE_READ_CHECKSUM_MISMATCH`, `LAKEHOUSE_READ_OMITTED_OBJECT`, `LAKEHOUSE_READ_ORDER_UNDETERMINED`, `LAKEHOUSE_PAYLOAD_BOUND_EXCEEDED`. |
| Raw feed manifest | `RAW_FEED_MANIFEST_INVALID`, `RAW_FEED_MANIFEST_DUPLICATE_ENTRY`, `RAW_FEED_MANIFEST_CHECKSUM_MISMATCH`, `RAW_FEED_MANIFEST_BOUNDS_EXCEEDED`, `RAW_FEED_MANIFEST_EMPTY_TARGET_INVALID`, `RAW_FEED_MANIFEST_ID_COLLISION`, `RAW_FEED_MANIFEST_RUNTIME_STATE_OMITTED`. |
| Table profile | `LAKEHOUSE_TABLE_PROFILE_MISSING`, `LAKEHOUSE_TABLE_FORMAT_UNSUPPORTED`, `LAKEHOUSE_SNAPSHOT_IDENTITY_FIELD_MISSING`, `LAKEHOUSE_COMMIT_IDENTITY_FIELD_MISSING`, `LAKEHOUSE_SCHEMA_INCOMPATIBLE`, `OPAQUE_TABLE_REPLAY_FORBIDDEN`. |
| Retention | `REPLAY_RETENTION_POLICY_MISSING`, `REPLAY_RETENTION_PROTECTED_REF_REFUSED`, `REPLAY_RETENTION_LEGAL_HOLD_PROTECTED`, `REPLAY_RETENTION_WINDOW_PROTECTED`, `REPLAY_RETENTION_GRAPH_REBUILD_PROTECTED`. |
| Maintenance | `TABLE_MAINTENANCE_POLICY_MISSING`, `TABLE_MAINTENANCE_ACTION_NOT_ALLOWED`, `TABLE_MAINTENANCE_RUN_LOCK_GUARD_MISSING`, `TABLE_MAINTENANCE_CANDIDATE_CHECKSUM_MISMATCH`, `TABLE_MAINTENANCE_PARTIAL_ELIGIBILITY_REFUSED`, `TABLE_MAINTENANCE_TIMEOUT`, `TABLE_MAINTENANCE_COMMIT_REF_MISSING`. |
| Cross-table | `CROSS_TABLE_PROFILE_MISSING`, `CROSS_TABLE_MISSING_TABLE`, `CROSS_TABLE_MIXED_SNAPSHOT_REJECTED`, `CROSS_TABLE_SET_CHECKSUM_MISMATCH`. |
| Catalog promotion | `CATALOG_PROMOTION_MUTABLE_REF_FORBIDDEN`, `CATALOG_ROLLBACK_MUTABLE_TARGET_FORBIDDEN`, `CATALOG_PROMOTION_APPROVAL_MISSING`, `CATALOG_PROMOTION_VALIDATION_MISSING`, `CATALOG_PROMOTION_TARGET_ADVANCED`, `CATALOG_PROMOTION_POST_CHECKSUM_MISMATCH`. |

Generic activation, bounds, authorization, or manifest errors must not substitute for a more specific code in this fragment.

### LakehouseErrorContext

`LakehouseErrorContext` is the owner context schema for `020` feed, manifest, table-state, and upstream-completeness error rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `020` context schema version. |
| `owner_spec` | Yes | Must be `020`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `feed_profile`, `category_closure`, `manifest`, `declared_subset`, `upstream_completeness`, `empty_scope`, or `activation_artifact`. |
| `operation` | Yes | Feed read, manifest validation, raw import, declared-subset validation, or table-state operation. |
| `affected_record_type` | Yes | Feed profile, manifest, table-state ref, raw import run, or completeness receipt type. |
| `field_path` | Yes | Exact field path when applicable; null only for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to the selected feed profile, feed category closure row set, read policy, fixture, manifest, snapshot, dataset version, object batch, partition set, raw import run, or table-state artifact consulted by the error; empty only when no artifact was consulted. |
| `feed_profile_ref` | No | Redacted feed profile ref when consulted. |
| `read_target_kind` | No | Closed read target token when known. |
| `source_dataset` | No | Vendor-neutral dataset token only. |
| `scope_selector_checksum` | No | Checksum of the scope selector; raw selector values are forbidden when private. |
| `manifest_refs` | No | Canonically sorted manifest, snapshot, dataset, or object refs. |
| `required_upstream_evidence_classes` | No | Closed evidence-class tokens required by the category row. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason that explains blocked output; otherwise null or omitted. Diagnostic rows may include a bounded reason only when it changes caller or audit interpretation. |
| `validation_refs` | Yes | Exact `120` fixture refs. |
| `redaction_classes` | Yes | Private routes, credentials, host lists, raw payload bytes, and source-native IDs must map to `always_forbidden`. |

### LakehouseApiStateLabelGuidance

Profile, category, manifest, table-state, and activation errors render through `110` as `error` or health-blocked states. Partial read, declared-subset, manifest, and empty-scope errors must not render as authorized absence. Upstream completeness insufficiency must map through `060` before any absence-sensitive API, export, graph, or watermark effect.

### ActivationControlledRowSchemaPrecisionHandoff

The following `020` row families can affect feed activation, feed read, raw import, absence-sensitive eligibility, replay, retention, maintenance, table-set consistency, or catalog promotion. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `LakehouseFeedProfileSchema` | output_affecting | Closed for source-effect selection by the field table above plus required `source_dataset_catalog_row_ref`, row checksum, validation refs, activation scope, lifecycle status, and `VersionManifest` inclusion. Remaining non-source-effect precision work in this table still blocks unrelated production feed behavior. |
| `LakehouseFeedCategoryClosureRow` | output_affecting | Closed for source-effect selection by the field table above, structured `source_dataset_allowlist_ref`, total `effect_closure_requirements`, deterministic block refs, validation refs, activation scope, lifecycle status, and `VersionManifest` inclusion. Remaining non-source-effect precision work in this table still blocks unrelated production feed behavior. |
| `RawSupplierProfile` | output_affecting when referenced by feed profile | Closed by `RawSupplierProfileRow schema`; production selection requires structured refs/checksums, active lifecycle, validation refs, private-binding rejection, and `VersionManifest` inclusion. |
| `LakehouseReadPolicy` | output_affecting | Closed by `LakehouseReadPolicyRow schema`; target refs, payload bounds, retries, timeouts, checksum behavior, object gap behavior, omitted-object behavior, and deterministic ordering are explicit. |
| `LakehouseTableProfile` | output_affecting | Closed by `LakehouseTableProfileRow schema` and `TableFormatNativeIdentityMatrix`; table identity, format, catalog binding, replay, maintenance, schema compatibility, and checksum behavior are explicit. |
| `ReplayRetentionPolicy` | output_affecting | Closed by `ReplayRetentionPolicyRow schema`; protected artifacts, legal hold, graph rebuild protection, retention mode, and deletion eligibility are explicit. |
| `TableMaintenancePolicy` | output_affecting | Closed by `TableMaintenancePolicyRow schema` and `DecideTableMaintenance`; destructive and rewrite maintenance preflight, refusal, idempotence, timeout, candidate checksum, and decision refs are explicit. |
| `CrossTableCommitProfile` | output_affecting when table-set consistency is required | Closed by `CrossTableCommitProfileRow schema`; required table groups, consistency mode, atomicity requirement, catalog semantics, and table-set checksum behavior are explicit. |
| `CatalogBranchPromotionPolicy` | output_affecting when catalog versioning controls production visibility | Closed by `CatalogBranchPromotionPolicyRow schema`; immutable source commit, validation, approval, rollback, failure behavior, and post-promotion verification are explicit. |
| `RawFeedManifest` | runtime_state_record | Closed as `RawFeedManifestRuntimeStateSchema`; it remains runtime evidence and must not be modeled as an activation-controlled row. |

Ref arrays including `object_refs`, `partition_refs`, `schema_refs`, `coverage_profile_refs`, `source_authority_profile_refs`, and `source_staleness_policy_refs` must declare `canonical_set` or `ordered_sequence`, duplicate policy, canonical sort key, and checksum participation. `blocked_effects` and `effect_closure_requirements` must be typed maps with total key coverage over declared effect tokens.

A production feed read, raw import, completeness evaluation, absence-sensitive effect, replay retention decision, destructive maintenance decision, cross-table read, or catalog promotion must fail before output when any selected `020` row family remains `TODO:` or lacks a structured `030.ActivationControlledRowRef`.

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
| `020-VOLATILITY-AC-001` | A production feed read with an inactive `LakehouseFeedProfile` emits no `RawRecord`. |
| `020-VOLATILITY-AC-002` | Identical feed bytes with different active `LakehouseReadPolicy` checksums produce different `VersionManifest` refs or fail before output. |
| `020-VOLATILITY-AC-003` | Table maintenance fails before destructive action when the required policy artifact is inactive or omitted from `VersionManifest`. |
| `020-FEED-CLOSURE-AC-001` | Feed profile activation and production read fail with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` when any required profile schema field is omitted, including `read_target_kind`. |
| `020-FEED-CLOSURE-AC-002` | Every active feed category either has an active `LakehouseFeedCategoryClosureRow` with validation refs or a deterministic MVP block row. |
| `020-FEED-CLOSURE-AC-003` | Every profile-dependent feed-read branch resolves to an explicit field, active artifact ref, receipt state, error/no-op, and validation row. |
| `020-FEED-CLOSURE-AC-004` | Omitted or invalid target-kind input never defaults to a table, object, partition, manifest, or dataset read. |
| `020-FEED-CLOSURE-AC-005` | Declared subset reads fail before output unless an active `030.DeclaredDAGSubsetProfile` covers requested outputs and effects. |
| `020-FEED-CLOSURE-AC-006` | Partial known gaps, partial unknown gaps, empty scopes under the default policy, and missing lakehouse rows never authorize absence, cleanup, retraction, graph expiry, or watermark advancement. |
| `020-SOURCE-CLOSURE-AC-001` | Every category row with non-empty `allowed_effects` declares `effect_closure_requirements`. |
| `020-SOURCE-CLOSURE-AC-002` | `allowed_effects` without matching `060` row refs permits no absence-sensitive effect. |
| `020-SOURCE-CLOSURE-AC-003` | Each feed category in `LakehouseFeedCategoryClosureRequirementTable` appears exactly once in `FeedCategoryToSourceAuthorityClosureRequirements`. |
| `020-SOURCE-CLOSURE-AC-004` | The active `LakehouseFeedCategoryClosureRowSet` contains exactly one row for every category in `LakehouseFeedCategoryClosureRequirementTable`. |
| `020-SOURCE-CLOSURE-AC-005` | Every category row's effect closure map is total across `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. |
| `020-SOURCE-CLOSURE-AC-006` | `source_history` rows require `CoverageDimensionProfile` plus `SourceHistoryRetentionProfile` before no-change proof. |
| `020-SOURCE-CLOSURE-AC-007` | `future_reachability` resolves to a deterministic block row and emits no MVP runtime effect. |
| `020-SOURCE-CLOSURE-HANDOFF-AC-001` | Every active feed category that names an absence-sensitive domain has exact `060` closure refs and `120` validation refs, or a deterministic block row. Missing, ambiguous, inactive, or checksum-mismatched refs fail before read/import output can authorize absence-sensitive effects. |
| `020-COVERAGE-DOMAIN-FEED-MAP-AC-001` | Every feed category in `LakehouseFeedCategoryClosureRequirementTable` maps to canonical coverage-domain tokens or deterministic block behavior. |
| `020-COVERAGE-DOMAIN-FEED-MAP-AC-002` | `configuration_inventory` has no implicit default coverage domain. |
| `020-COVERAGE-DOMAIN-FEED-MAP-AC-003` | `cloud_asset_inventory` requires `source_history` only when source-history evidence is consulted. |
| `020-COVERAGE-DOMAIN-FEED-MAP-AC-004` | `future_reachability` maps to `reachability` only for deterministic no-op/block behavior while `200` is inactive deferred. |
| `020-EVIDENCE-LAKEHOUSE-AC-001` | `LakehouseSnapshotRef`, `DatasetVersionRef`, `LakehouseCommitRef`, `LakehouseReadCompletenessReceipt`, and `UpstreamCompletenessEvidence` evidence use `cadastre_record_ref` with referenced record checksum. |
| `020-EVIDENCE-LAKEHOUSE-AC-002` | Raw object, manifest, and partition evidence use `lakehouse_artifact_ref` with immutable bytes or owner-declared canonical metadata bytes. |
| `020-EVIDENCE-LAKEHOUSE-AC-003` | Missing artifact checksum, mutable `latest` refs, branch-only refs, unresolved prefixes, and raw payload bytes fail before `EvidenceRef` ID computation. |
| `020-LIFECYCLE-AC-001` | Every feed-read branch maps to exactly one `030.StageExecutionLifecycleMachine.v1` event, terminal-state expectation, and mutation-prohibition rule. |
| `020-LIFECYCLE-AC-002` | Partial known and partial unknown gaps may commit positive raw records and receipts but must record blocked absence, cleanup, retraction, graph-expiry, and watermark effects. |
| `020-LIFECYCLE-AC-003` | Feed lifecycle results for `succeeded`, `no_op`, and isolated receipt-emitting failures contain `feed_receipt_state_ref`. |
| `020-RUNLOCK-MAINTENANCE-AC-001` | Destructive maintenance fails before mutation when the required `030.RunLockCommitGuard` is missing, stale, or fenced; successful maintenance includes the guard ref in `VersionManifest`. |
| `020-LAKEHOUSE-ROW-PRECISION-AC-001` | Every selected `020` activation-controlled row family has complete field precision, row checksum behavior, validation refs, activation scope, lifecycle status, and `VersionManifest` inclusion. |
| `020-RAW-SUPPLIER-AC-001` | Missing, inactive, unsupported, private-leaking, unredacted, or unvalidated supplier profile emits no raw output. |
| `020-READ-POLICY-AC-001` | Read policy defaults and bounds materialize deterministically, and out-of-bound timeout, retry, payload, or checksum behavior fails before read output. |
| `020-MANIFEST-AC-002` | `RawFeedManifest` remains runtime state, enforces canonical array semantics, rejects duplicates, validates bounds, and appears in `VersionManifest` when output-affecting. |
| `020-TABLE-PROFILE-AC-001` | Table profiles require table-format-native snapshot and commit identity fields before production read, replay, rebuild, correction, or maintenance. |
| `020-RETENTION-AC-001` | Retention preflight refuses any candidate protected by manifests, replay windows, graph rebuild inputs, legal holds, schema compatibility, table-state checksums, or active retention rules. |
| `020-MAINTENANCE-AC-001` | Destructive or rewrite-capable maintenance mutates no table state unless retention decision, candidate checksum, run-lock guard, commit ref, and manifest refs all validate. |
| `020-CROSS-TABLE-AC-001` | Missing table, mixed snapshot, and table-set checksum mismatch fail before coherent table-set output. |
| `020-CATALOG-PROMOTION-AC-001` | Catalog promotion rejects mutable branch/tag-only refs, missing validation, missing approval, mutable rollback targets, target-advanced candidates, and post-promotion checksum mismatch before visibility changes. |
| `020-NONAUTHORITY-AC-001` | Manifest validity, table commit success, maintenance success, and catalog promotion success remain operational evidence only and do not authorize source absence or cleanup by themselves. |

### Structured input feed acceptance criteria

| ID | Criterion |
| --- | --- |
| `020-STRUCTURED-INPUT-FEED-AC-001` | Repository-authored feed profile changes produce no feed read, raw import, absence evaluation, cleanup, graph expiry, retraction, or watermark advancement until the profile is materialized, activated, and manifest-included. |
| `020-STRUCTURED-INPUT-FEED-AC-002` | `StructuredInputRepositorySnapshot` cannot satisfy `read_target_kind`, `LakehouseSnapshotRef`, `DatasetVersionRef`, `RawFeedManifest`, feed category closure, or lakehouse read policy refs. |
| `020-STRUCTURED-INPUT-FEED-AC-003` | Repository-authored feed profiles that leak private bindings fail with `FEED_PROFILE_REPOSITORY_PRIVATE_BINDING_LEAK` or imported `PRIVATE_BINDING_LEAK` before publication or validation-report materialization. |
| `020-STRUCTURED-INPUT-FEED-AC-004` | Template-only or sync-only feed profile candidates create no feed read, raw import, absence evaluation, cleanup, graph expiry, retraction, or watermark advancement. |
| `020-STRUCTURED-INPUT-FEED-AC-005` | Stale producer CI, publication manifest mismatch, or missing package-set refs block repository-authored feed profile activation before lakehouse availability checks and before read/import. |
| `020-SOURCE-DATASET-CATALOG-AC-001` | Every active `LakehouseFeedProfile.source_dataset` resolves to exactly one active `SourceDatasetCatalogRow` or one exact deterministic source-dataset block row before feed activation or read/import. |
| `020-SOURCE-DATASET-CATALOG-AC-002` | Public source-dataset catalog rows that contain private route names, tenant IDs, scanner sites, account lists, host lists, raw sample bytes, source-native secret values, or private schema payloads fail with `SOURCE_DATASET_CATALOG_PRIVATE_BINDING_LEAK`. |
| `020-SOURCE-DATASET-CATALOG-AC-003` | `030.VersionManifest` includes selected source-dataset row refs/checksums, row-set refs/checksums, selector checksums, validation refs, and package-set refs when package-supplied. |
| `020-SOURCE-DATASET-CATALOG-AC-004` | `future_reachability` source-dataset rows resolve only to deterministic block or inactive-deferred behavior while `200` remains inactive. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `020-SCOPE-FEED-AC-001` | Feed read requests materialize a normalized `030.ScopeSelector` before availability, read, import, and completeness decisions. |
| `020-SCOPE-FEED-AC-002` | Exact feed scope selection, subset-disallowed feed scope, missing required key, duplicate key, private-binding leak, and manifest-inclusion fixtures pass for every active production feed context. |
| `020-AC-001` | Raw import can be replayed from declared feed/table/object refs without enterprise source calls. |
| `020-AC-002` | Missing rows or objects never authorize absence without an imported completeness decision from `060`. |
| `020-AC-003` | Every production read and write has table-format-native refs when it can affect output or replay. |
| `020-AC-004` | Destructive maintenance is refused when any protected manifest, snapshot, replay window, or legal hold would be invalidated. |
| `020-AC-005` | Feed profile activation fails while feed feasibility, raw supplier profiles, read policy catalog, raw manifest schema, or feed fixture coverage is `TODO:` for the target feed. |
| `020-AC-006` | Every active production feed category has an active closure row set, a passing feasibility assessment, fixture refs, validation refs, and deterministic missing-row behavior. |
| `020-AC-007` | Destructive or rewrite maintenance cannot commit without a valid `030.RunLockCommitGuard` and cannot advance watermarks after lock loss. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `020-TODO-SOURCE-DATASET-CATALOG` | TODO: supporting artifact path and concrete public row catalog are not supplied in the uploaded files. The schema and deterministic missing-row behavior are defined by `SourceDatasetCatalogRowSet`; production requires active catalog rows or exact deterministic source-dataset block rows. | Feed activation, mapping activation, source-authority closure, graph/analysis handoff, API filtering, package-supplied catalogs, and validation. | Product governance plus `020`, `030`, `060`, `100`, and `120`. | Referenced datasets without selected catalog or block rows fail with `SOURCE_DATASET_CATALOG_ROW_MISSING` and emit no absence-sensitive mutation. |
| `020-TODO-FEED-CATEGORY-CLOSURE-ROW-SET` | TODO: Provide an active `LakehouseFeedCategoryClosureRowSet` or deterministic block row for every category in `LakehouseFeedCategoryClosureRequirementTable`, with validation refs and `VersionManifest` inclusion. | Production feed activation and absence-sensitive effects. | Product governance plus `020`, `060`, `100`, and `120` validation refs. | Production feed categories without active rows fail with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` or deterministic block behavior. |
