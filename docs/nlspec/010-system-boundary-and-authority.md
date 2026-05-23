---
doc_id: CADASTRE-NLSPEC-010
title: System Boundary and Authority
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define what Cadastre is, what it must not do, what can be authoritative, and which outputs are projections.

## Explicit Non-Scope

- Lakehouse read field schemas.
- Graph backend adapter mechanics.
- Identity resolver algorithms.
- Package supply-chain mechanics.

## Imports

- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeDimension`

## Exports

- `AuthorityClass`
- `PublicPrivateBoundaryRule`
- `ProjectionAuthorityRule`
- `RuntimeTelemetryAuthorityRule`
- `DirectSourceProhibition`
- `SourceOfRecordRule`
- `VolatilityBoundaryRule`
- `ScopeSelectorPublicBindingRule`

## Boundary Contract

Cadastre must be a lakehouse-fed interpretation, normalization, identity, fact, and projection system. It must not collect directly from configured enterprise source systems in the default production boundary.

| Input class | Production status | Required handling |
| --- | --- | --- |
| Declared lakehouse table snapshot | Allowed | Must be named by `LakehouseSnapshotRef` imported from `020`. |
| Declared dataset version | Allowed | Must be named by `DatasetVersionRef` imported from `020`. |
| Object-store raw batch | Allowed | Must be named by `RawFeedManifest` imported from `020`. |
| Supplier-provided metadata | Allowed | Must be interpreted only through active authority, completeness, temporal, and mapping policies. |
| Source-effect closure catalog, deterministic block catalog, validation report, package-set manifest, or closure health record | Activation and validation artifact only | May authorize interpretation only through the owner specs that export the selected rows. Must not become raw source evidence, direct enterprise source evidence, graph truth, package activation authority by itself, external-system authority, or system-of-record state. |
| Structured input authoring repository snapshot | Allowed for authoring, validation, and provenance only | Must be named by `030.StructuredInputRepositorySnapshot`; must not satisfy raw evidence, source completeness, source authority, package activation, graph authority, production approval, or rollback eligibility. |
| Maintenance SDK/CLI output | Validation or materialization evidence only | Must be bound to exact repository snapshot, selected paths, tool contract, input checksum, output checksum, redaction refs, and validation refs; must not satisfy activation, authority, production approval, or rollback. |
| Repository template conformance | Validation evidence only | Must not substitute for owner row refs, package release refs, package-set activation, source authority, graph authority, or `VersionManifest` completeness. |
| Producer CI status | Validation evidence only | Valid only for the exact commit, tree, selected paths, file manifest checksum, toolchain refs, validation matrix refs, and output checksums named by `030.StructuredInputRepositoryCIContract`. |
| Producer publication manifest | Candidate package evidence only | Must be imported and verified by `100.ImportStructuredInputPublicationManifest` against immutable artifact bytes, digest, size, media type, package type, validation refs, redaction refs, and release manifest fields. |
| Manifest-driven sync record | Candidate discovery and audit only | Must not mutate active package set, owner rows, watermarks, graph state, source authority, source completeness, rollback targets, or system-of-record state. |
| Enterprise source API call | Forbidden | Must fail with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| Source scanner poll, source syslog receive, source CDC connector operation | Forbidden | Must be performed by an external supplier, not by Cadastre production. |
| Validation-only source exploration | Allowed only when explicitly declared | Must emit only `SourceExplorationResult` or `ProbeDiagnosticRecord`; it must not satisfy production evidence. |

## Authority Classes

| Authority class | Meaning | Default owner |
| --- | --- | --- |
| `system_of_record` | Record class that can determine product truth. | Authoritative lakehouse records only. |
| `derived_projection` | Rebuildable output from authoritative records. | Graph, CIM, OCSF export, consumer outputs. |
| `supporting_evidence` | Source or supplier data that can support a decision only through an active profile. | Raw records, upstream completeness evidence, lineage, diagnostics. |
| `non_authoritative_analysis` | Findings, metrics, enrichment, and registry metadata with no mutation authority. | Analysis and governance records. |
| `inactive_future_domain` | Preserved candidate material with no MVP effect. | Future reachability. |

Structured-input repository snapshots default to `supporting_evidence` or `runtime_state_record` as routed by the owner spec. They must never be classified as `system_of_record` and must not become source, identity, graph, package, or production approval authority by existence.

A maintenance tool, repository template, CI status, publication manifest, or sync record must fail with `STRUCTURED_INPUT_AUTHORITY_VIOLATION` when used as source authority, source completeness, identity authority, fact authority, graph authority, package activation, production approval, rollback eligibility, validation acceptance by itself, or system-of-record state.

## Projection Authority Rule

Graph state, Splunk CIM output, OCSF export output, metadata graphs, lineage graphs, analysis outputs, and registry labels must not create or modify authoritative facts. A projection may be served only when its derivation profile, input refs, output checksum, and lag state are persisted.

### RuntimeTelemetryAuthorityRule

Runtime telemetry, including traces, spans, metrics, structured logs, baggage, exporter state, Collector state, sampling decisions, dashboards, and telemetry-derived alerts, is operational diagnostic material only.

Runtime telemetry must not create, modify, retract, expire, merge, split, authorize, validate, project, replay, or persist authoritative Cadastre domain records.

Runtime telemetry must not create or modify `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `IdentityDecision`, `GoldFact`, `GraphDeltaSet`, `GraphApplyResult`, `SourceAuthorityProfileRow`, `LakehouseFeedCompletenessProfileRow`, `CoverageAssertion`, `PackageActivationFailureEvent`, `ProductionPackageSetManifest`, `WatermarkCommitRecord`, `VersionManifest`, or `AuditEvent`.

Telemetry may inform `110.OperationalHealthStatus` only through `140.TelemetryHealthMappingPolicy` and `110.EndpointOutcomeMatrix`.

Telemetry loss, exporter failure, or Collector failure must not invalidate already persisted authoritative Cadastre records.

## Volatility Boundary Rule

`stable_core_contract` owns the product boundary, authority classes, source-of-record rule, direct-source prohibition, canonical record semantics, temporal axes, identity semantics, graph projection boundary, package activation boundary, API state-label boundary, and validation ownership boundary.

`activation_controlled_artifact` may instantiate exported behavior through versioned rows, profiles, package refs, registry records, fixtures, and mapping tables. It must not define new authority classes, omission states, canonical fields, ID algorithms, temporal axes, identity semantics, graph authority, API state labels, or package trust semantics.

Activation-controlled row catalogs must instantiate owner-exported row schemas through `030.ActivationControlledRowSchema`, `030.ActivationControlledRowField`, `030.ActivationControlledRowRef`, and `030.ActivationControlledRowSetSchema`. A row catalog must not redefine row field types, defaults, null behavior, omit behavior, ID inputs, checksum inputs, selector behavior, lifecycle behavior, error precedence, authority semantics, row sorting, duplicate policy, or extension behavior. Private bindings may map concrete private values to public redacted refs or hashes, but they must not change row validation, row selection, row sorting, checksum participation, missing-field behavior, invalid-field behavior, or owner error precedence.

Git-authored activation-controlled artifacts may instantiate owner-exported behavior only after exact snapshot validation, immutable materialization, package-set membership when required, and `030.VersionManifest` inclusion. A mutable Git ref, branch, tag, pull request, repository URL, hook result, or commit timestamp must not satisfy production activation.

If an activation-controlled artifact conflicts with a stable core contract, the artifact fails activation before any production output. If two stable core contracts conflict, the spec set fails validation. If a stable core spec embeds concrete volatile rows, the rows must be non-normative examples or `TODO:` blockers unless represented as an activation-controlled artifact.

## Public and Private Source Binding

The public NLSpec set must define vendor-neutral feed contracts. Concrete vendor names, private source binding artifacts, routes, credentials, and environment-specific bindings must live in private implementation artifacts and must not appear in public canonical records.

Core schemas may name vendor-neutral source categories and redacted refs only. A public `RawRecord`, `EvidenceRef`, graph delta, API response, validation report, package report, resolver row catalog, or export artifact that exposes a private source binding must fail with `PRIVATE_BINDING_LEAK` before publication or persistence.

| Artifact | Public canonical model status |
| --- | --- |
| `RawSupplierProfile` | Public, vendor-neutral. |
| `LakehouseFeedProfile` | Public, vendor-neutral. |
| `SourceDatasetCatalogRowSet` | Public, vendor-neutral. |
| Concrete source-dataset-to-vendor/product/route binding | Private only. It may be represented through private implementation artifacts or redacted refs; it must never appear in public catalog bytes. |
| `PrivateSourceFeedBinding` | Private only. |
| `PrivateFeedSchemaInventory` | Private only. |
| `PrivateCompletenessEvidenceInventory` | Private only. |
| `PrivateGoldenCorpus` | Private only unless redacted into `LakehouseFeedFixture`. |

A public source-dataset catalog row must not contain a vendor product name, private route, credential, tenant inventory, scanner site, directory tenant list, cloud account list, host list, or private schema payload. A private implementation may bind concrete upstream systems to a public `source_dataset` token only when the binding does not alter public row selection, row checksum, default missing-row behavior, error precedence, activation scope, validation requirements, or `030.VersionManifest` inclusion.

Public structured-input repositories must not contain concrete private routes, credentials, tenant IDs, private inventories, source-native secrets, unredacted private schema payloads, or raw private fixture bytes. Private structured-input repositories may bind concrete private systems to public rows, but those bindings must not alter public row-selection order, defaults, error precedence, authority semantics, or activation gates.

| Structured input repository field class | Public canonical status | Required behavior |
| --- | --- | --- |
| repository profile ID, snapshot ID, validation status, materialization status | Public only as redacted refs or checksums | May appear in public validation or API output when `110` redaction permits. |
| branch name, file path, repository URL, commit SHA | Diagnostic only | Must be redacted, hashed, or bounded before public output according to `110` and `140`. |
| private route, credential, tenant inventory, host list, account list, private sample bytes | Private only | Must fail with `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` or `PRIVATE_BINDING_LEAK` before public persistence, publication, export, API response, audit response, telemetry export, or validation-report materialization. |

### ScopeSelectorPublicBindingRule

`ScopeSelectorPublicBindingRule` applies to every `030.ScopeSelector` and `030.ActivationScope` before persistence, publication, export, API output, audit output, telemetry export, package report materialization, validation report materialization, or public structured-input materialization.

Public selector dimension values may contain only the value kinds permitted by `030.ScopeDimension.value_kind`: `enum_token`, `cadastre_id`, `external_ref_id`, `redacted_ref`, and `sha256_hex`. A public selector must not contain raw private tenant IDs, source routes, scanner sites, host lists, account lists, credentials, private inventories, raw private fixture bytes, source-native identity values, backend internal IDs, private endpoint URLs, private repository routes, or concrete environment-specific source target lists.

| Selector value class | Public status | Required behavior |
| --- | --- | --- |
| Vendor-neutral enum token | Allowed | Validate through the owner `030.ScopeSelectorContext`; do not reinterpret as private binding. |
| Cadastre ID | Allowed | Must reference a public Cadastre record or redacted public record ref. |
| External ref ID | Allowed when redacted or non-sensitive | Must not encode private route, credential, tenant inventory, host list, account list, scanner site, or backend internal ID. |
| Redacted ref | Allowed | Must be opaque and irreversible from public bytes. |
| SHA-256 hex | Allowed | May prove equality of private values only when the raw value is not exposed and the owner redaction rule permits the hash. |
| Raw private binding value | Forbidden | Fail with `PRIVATE_BINDING_LEAK` or a more specific owner code before public output. |

Private binding artifacts may map concrete values to public redacted refs or hashes. They must not alter `030.NormalizeScopeSelector`, `030.ScopeSelectorEquals`, `030.ScopeSelectorCovers`, `030.CompareScopeSpecificity`, `030.ResolveScopedRow`, default exactness, subset eligibility, error precedence, ambiguity behavior, activation gates, or owner non-scope predicates.

### SourceAuthorityRowPublicBindingRule

Public source-authority row catalogs may contain only vendor-neutral `source_category`, `source_dataset`, row IDs, redacted artifact refs, canonical scope selectors, lifecycle status, validation refs, and checksummed public policy refs.

`SourceAuthorityProfileRow`, `SourceAuthorityProfileRowSet`, `LakehouseFeedCategoryClosureRow`, `LakehouseFeedCategoryClosureRowSet`, `SourceAuthorityClosureMatrixRow`, `SourceAuthorityClosureMatrixRowSet`, `ExternalSchemaAuthoritySignalMappingRow`, `ExternalSchemaAuthoritySignalMappingRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ControlResultMappingRow`, `SupplierCollectionVisibilityProfile`, `SourceHistoryRetentionProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `LakehouseFeedCompletenessProfileRow`, `SourceAuthorityClosureMatrix` validation results, source-closure validation report rows, and source-closure acceptance report rows public instances must not contain concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventory, zone inventory, account lists, or environment-specific source target lists.

A source-closure row, deterministic block row, validation report, `VersionManifest` ref, API output, export output, audit output, or persisted public artifact that contains a concrete product name, tenant ID, private route, credential, host list, scanner site name, directory tenant inventory, zone inventory, account list, or environment-specific source target list must fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response materialization, or validation-report materialization.

Private source binding artifacts may map concrete upstream systems to public vendor-neutral datasets. Private mappings must not alter the public `060` row-resolution algorithm, matching specificity, default omission behavior, error precedence, or allowed effect semantics.

### MappingArtifactPublicBindingRule

Public OCSF mapping row catalogs, external enum rule catalogs, profile-resolution manifests, base-event policy sets, source-extension rule sets, observation-type validation matrices, and canonical validation output summaries may contain only vendor-neutral `source_category`, `source_dataset`, observation type tokens, row IDs, redacted artifact refs, field paths, checksummed public policy refs, lifecycle status, activation scope, and validation refs.

Public instances of these artifacts must not contain concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventories, zone inventories, account lists, raw fixture bytes, source-native secrets, or environment-specific source target lists.

Any public mapping artifact, validation report, API output, export output, audit output, or `VersionManifest` ref that leaks those values must fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response materialization, or validation-report materialization.

Private artifacts may bind concrete upstream source systems and private source schemas to public mapping rows. Private bindings must not alter public `050.ResolveOCSFMapping` row-selection order, default behavior, error precedence, source-extension policy, enum policy, OCSF non-authority behavior, or `cadastre_only` behavior.

### IdentityResolverRowPublicBindingRule

Public identity resolver row catalogs may contain only vendor-neutral source categories, source datasets, row IDs, redacted artifact refs, canonical scope selectors, lifecycle status, validation refs, and checksummed policy refs.

`ResolverProfileRow`, `IdentifierScope`, `IdentifierEvidenceClass`, `IdentityHardBlockerRow`, `IdentityHardBlockerRowSet`, `AssetGenerationBoundary`, `CandidateGenerationProfile`, `TargetSelectorSafetyPolicy`, `IdentityReviewRoutingPolicy`, `SplitPolicy`, `IdentityConfidenceBand`, `ResolverDecisionMatrix`, `ResolverExplanationPolicy`, `ResolverActivationReport`, `ResolverShadowRun`, and resolver activation catalog closure validation artifacts public instances must not contain concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventory, zone inventory, account lists, environment-specific source target lists, source-native identity values in public validation reports, or private scanner site names.

A public resolver row, package activation report, validation report, export, API response, or persisted public artifact that contains a concrete product name, tenant ID, private route, credential, host list, scanner site name, directory tenant inventory, zone inventory, account list, or environment-specific source target list must fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response materialization, package report materialization, or validation-report materialization.

Private source binding artifacts may map concrete upstream systems to public resolver scopes. Private mappings must not alter the public `070` resolver row-selection algorithm, evidence authority defaults, blocker precedence, decision semantics, review totality, split policy semantics, selector safety, or explanation checksum policy.

Forbidden identity-specific leak examples include private scanner site names, directory tenant inventories, cloud account lists, environment-specific source target lists, private routes, credentials, host lists, and source-native identity values in public resolver activation reports, validation reports, package reports, API responses, export outputs, or resolver row catalogs.

### ActivationCatalogPublicBindingRule

This rule applies to every activation-controlled row catalog in `000.MVPActivationCatalogClosurePack`, including source-dataset catalogs, deterministic source-dataset block rows, feed-category catalogs, OCSF mapping catalogs, source-authority closure catalogs, underlying `060` row sets, resolver catalogs, graph profile catalogs, graph backend catalogs, package policy rows, package deprecation rows, package release manifests, production package-set manifests, validation-output catalogs, acceptance reports, package reports, API outputs, audit outputs, and telemetry-visible diagnostics that reference source-dataset or source-authority closure.

| Public catalog field class | Public status | Required behavior |
| --- | --- | --- |
| vendor-neutral category or dataset token | allowed | May appear in public rows when the owning spec defines the token. |
| redacted artifact ref, package-set ref, validation ref, lifecycle status, checksum, canonical selector | allowed | May appear only as bounded refs or checksums and must not reveal private route, tenant, account, host, credential, or raw fixture bytes. |
| concrete vendor, product, tenant ID, account list, host list, directory tenant inventory, scanner site name, route, credential, private schema payload, backend credential, private source-native object ID, or private sample byte | forbidden | Must fail before persistence, publication, API response, export, audit output, telemetry export, package report, or validation-report materialization. |
| Git snapshot, validation run, repository branch, repository tag, package label, backend default, research report, or ADR | non-authority | Must not satisfy activation-catalog closure, source authority, package activation, graph backend activation, production approval, or `VersionManifest` completeness. |

Private implementation artifacts may bind concrete sources, private schemas, backend endpoints, deployment credentials, or package repositories to public rows. Those private bindings may map concrete values to redacted refs or hashes only. They must not alter row selection, row checksum, error precedence, effect default, validation requirement, activation scope, package-set requirement, manifest inclusion, source-authority closure outcome, or public missing-row behavior.

If an activation-catalog publication leak is covered by `PRIVATE_BINDING_LEAK`, `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK`, `FEED_PROFILE_REPOSITORY_PRIVATE_BINDING_LEAK`, `RESOLVER_REPOSITORY_PRIVATE_BINDING_LEAK`, or another more specific owner code, the most specific existing code must be used. `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` is emitted only when no more specific private-binding error covers the closure-pack catalog family.

## Cross-Domain Invariants

- Missing lakehouse rows must not imply source absence.
- Successful feed read must not imply source completeness.
- Source completeness must not imply source authority.
- Source authority, source completeness, source scope, and public resolver row presence must not imply identity merge, creation, or attachment authority.
- Identity resolution must not mutate graph state directly.
- Graph apply success must not imply fact correctness.
- Package signature verification must not imply package activation eligibility.
- Analysis findings must not mutate facts, graph state, watermarks, or completeness.
- A missing source-authority closure row must not imply source absence, cleanup permission, graph expiry, retraction, pass/fail state, source-history no-change, or watermark advancement.
- Runtime telemetry must not imply source authority, source completeness, identity, fact correctness, graph correctness, audit persistence, replay equivalence, package activation, or watermark eligibility.
- A Git commit, branch, tag, pull request, merge, repository URL, hook result, or commit timestamp must not imply source authority, source completeness, identity authority, fact authority, graph authority, package activation, production approval, or rollback eligibility.

## Required Errors

| Error code | Emitted when |
| --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | Production execution attempts to call or authenticate to an enterprise source system. |
| `PROJECTION_AUTHORITY_VIOLATION` | A derived projection attempts to create authoritative records. |
| `PRIVATE_BINDING_LEAK` | A public artifact, including identity resolver rows and package-supplied resolver artifacts, contains a concrete private vendor/source binding. |
| `TELEMETRY_AUTHORITY_VIOLATION` | Runtime telemetry attempts to create, modify, authorize, validate, replay, or persist a non-telemetry authoritative Cadastre record. |
| `TELEMETRY_PRIVATE_BINDING_LEAK` | Telemetry contains a private source binding, private route, credential, tenant inventory, or environment-specific private value before redaction/export. |
| `STRUCTURED_INPUT_AUTHORITY_VIOLATION` | A structured-input repository value attempts to satisfy source authority, source completeness, identity authority, fact authority, graph authority, package activation, production approval, rollback eligibility, or system-of-record state. |
| `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` | A public structured-input repository artifact, API response, validation output, telemetry event, audit output, package report, or manifest leaks private repository routes, credentials, tenant inventories, private source bindings, source-native secrets, raw fixture bytes, or raw structured input bytes. |
| `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` | A public activation-catalog closure-pack row, package report, validation output, API response, audit output, export, or manifest leaks a private binding and no more specific private-binding error covers the artifact. |
| `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | A branch, tag, pull request ref, repository URL, mutable default branch, or rebuilt repository tip is used as a production activation, rollback, manifest, or authority target. |
| `UNDECLARED_AUTHORITY_CLASS` | A record is written without an owner declaring its authority class. |
| `VOLATILITY_BOUNDARY_VIOLATION` | An activation-controlled artifact attempts to define product authority or stable core semantics. |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | An activation-controlled artifact conflicts with the owner stable core contract. |
| `VOLATILE_ROW_IN_CORE_SPEC` | A stable core spec embeds production-active volatile rows without artifact classification, example-only marking, or blocking `TODO:`. |

Error precedence:

| Condition | Required precedence |
| --- | --- |
| Caller is not authorized to know the object exists and the response could reveal a private binding | `AUTHORIZATION_ERROR` must precede `PRIVATE_BINDING_LEAK` in caller-visible API responses. |
| Public docs, exports, validation reports, or persisted public artifacts contain private binding data | `PRIVATE_BINDING_LEAK` must precede generic validation errors. |
| Public activation-catalog closure-pack rows contain private binding data and no specific owner private-binding code applies | `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` must precede generic validation, activation, package, graph, resolver, and manifest errors. |
| Projection attempts authoritative mutation | `PROJECTION_AUTHORITY_VIOLATION` must precede generic validation errors. |
| Activation artifact attempts stable-core override | `VOLATILITY_BOUNDARY_VIOLATION` or `ACTIVATION_ARTIFACT_CORE_CONFLICT` must precede owner-specific artifact validation errors. |
| Production code attempts direct enterprise source access | `DIRECT_SOURCE_CALL_FORBIDDEN` must precede package-local, transport, or source-client errors. |

## Authority Validation Contracts

### PrivateBindingLeakValidationRule

Public artifacts must be scanned before publication, API response emission, export, and validation-report materialization.

| Rejected artifact class | Rejected field or value pattern | Error precedence | Required behavior |
| --- | --- | --- | --- |
| Public NLSpec or README | Concrete enterprise source name, private route, credential material, tenant-specific binding, private inventory entry | `PRIVATE_BINDING_LEAK` before generic validation errors | Reject the artifact and record the owner spec. |
| Public API response | Inaccessible asset identifier, private source route, raw credential, unredacted raw payload | `AUTHORIZATION_ERROR` before `PRIVATE_BINDING_LEAK` when caller access is the root cause | Return redacted or fail closed as `110` defines. |
| Export artifact | Private source binding, environment-specific host list, private golden corpus sample | `PRIVATE_BINDING_LEAK` | Reject export before write. |
| Validation report | Private fixture bytes or unreduced corpus contents | `PRIVATE_BINDING_LEAK` | Store only redacted fixture refs and checksums. |
| Public resolver artifact, activation report, package report, API response, or export | Concrete scanner site, directory tenant inventory, cloud account list, route, credential, host list, environment-specific target list, or source-native identity value | `PRIVATE_BINDING_LEAK` | Reject before public artifact, report, response, or export materialization. |

### SourceOfRecordRule

| Output class | Authority class | Runtime owner | Default if owner row missing |
| --- | --- | --- | --- |
| `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `IdentityDecision`, `GoldFact` | `system_of_record` | `040`, `070`, `080` as applicable | Fail with `UNDECLARED_AUTHORITY_CLASS`. |
| `EvidenceRef`, supplier metadata, lineage, diagnostics, upstream completeness evidence | `supporting_evidence` | `020`, `040`, `060`, `110`, `130` as applicable | Diagnostic or pointer only unless owner profile consumes it. |
| `GraphNodeDeltaShape`, `GraphEdgeDeltaShape`, `GraphDeltaSet`, `GraphApplyResult`, graph read model, CIM output, OCSF export | `derived_projection` | `040`, `050`, `090`, `110` | Fail or no-op as owner specifies. |
| `AnalysisFinding`, `AnalysisMetric`, enrichment, registry metadata | `non_authoritative_analysis` | `130` | No mutation authority. |
| Reachability candidate contracts | `inactive_future_domain` | `200` | No MVP effect. |

### ValidationOnlySourceExplorationBoundary

`source_exploration` and validation-only probe modes may emit only `SourceExplorationResult`, `ProbeDiagnosticRecord`, validation diagnostics, or redacted fixture candidates. They must not satisfy production raw, completeness, silver, identity, gold, graph, health, watermark, manifest, or acceptance-report contracts.

### BoundaryValidationTraceability

| Error code | Required `120` validation row | Expected result |
| --- | --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | `neg-010-direct-source-call` | Fail before output and before source transport side effects. |
| `PRIVATE_BINDING_LEAK` | `neg-010-private-binding-leak-public-artifact` | Reject or redact according to artifact class and `110` response rules. |
| `PROJECTION_AUTHORITY_VIOLATION` | `neg-010-projection-authority-violation` | No authoritative mutation and no projection commit. |
| `TELEMETRY_AUTHORITY_VIOLATION` | `120-OBSERVABILITY-NONAUTH-*` | Telemetry authority attempt fails before forbidden mutation. |
| `STRUCTURED_INPUT_AUTHORITY_VIOLATION` | `010-STRUCTURED-INPUT-BOUNDARY-AC-001` | Git-as-authority attempt fails before output or mutation. |
| `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` | `010-STRUCTURED-INPUT-BOUNDARY-AC-002` | Private structured-input values are rejected or redacted before public output. |
| `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | `010-STRUCTURED-INPUT-BOUNDARY-AC-003` | Mutable Git refs fail before activation, rollback, or manifest satisfaction. |
| `UNDECLARED_AUTHORITY_CLASS` | `neg-010-undeclared-authority-class` | Reject record write before persistence. |
| `VOLATILITY_BOUNDARY_VIOLATION` | `neg-010-activation-artifact-core-override` | Reject artifact activation before production output. |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | `neg-010-activation-artifact-core-conflict` | Reject artifact activation before production output. |
| `VOLATILE_ROW_IN_CORE_SPEC` | `neg-010-volatile-row-in-core-spec` | Fail spec-set validation before promotion. |

### Error ownership export

| Error code | Owning spec | Shared registry owner |
| --- | --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | `010` | `110` |
| `PROJECTION_AUTHORITY_VIOLATION` | `010` | `110` |
| `PRIVATE_BINDING_LEAK` | `010` | `110` |
| `TELEMETRY_AUTHORITY_VIOLATION` | `010` | `110` |
| `TELEMETRY_PRIVATE_BINDING_LEAK` | `010` | `110` |
| `STRUCTURED_INPUT_AUTHORITY_VIOLATION` | `010` | `110` |
| `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` | `010` | `110` |
| `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | `010` | `110` |
| `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` | `010` | `110` |

| `UNDECLARED_AUTHORITY_CLASS` | `010` | `110` |
| `VOLATILITY_BOUNDARY_VIOLATION` | `010` | `110` |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | `010` | `110` |
| `VOLATILE_ROW_IN_CORE_SPEC` | `010` | `110` |

### BoundaryErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | `010` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-direct-source-call-forbidden` |
| `PROJECTION_AUTHORITY_VIOLATION` | `010` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-projection-authority-violation` |
| `PRIVATE_BINDING_LEAK` | `010` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `010.BoundaryErrorContext` | `error-registry-010-private-binding-leak` |
| `TELEMETRY_AUTHORITY_VIOLATION` | `010` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-telemetry-authority-violation` |
| `TELEMETRY_PRIVATE_BINDING_LEAK` | `010` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-telemetry-private-binding-leak` |
| `STRUCTURED_INPUT_AUTHORITY_VIOLATION` | `010` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-structured-input-authority-violation` |
| `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` | `010` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `010.BoundaryErrorContext` | `error-registry-010-structured-input-private-binding-leak` |
| `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | `010` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-structured-input-mutable-ref-forbidden` |
| `ACTIVATION_CATALOG_PRIVATE_BINDING_LEAK` | `010` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `010.BoundaryErrorContext` | `error-registry-010-activation-catalog-private-binding-leak` |
| `UNDECLARED_AUTHORITY_CLASS` | `010` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `010.BoundaryErrorContext` | `error-registry-010-undeclared-authority-class` |
| `VOLATILITY_BOUNDARY_VIOLATION` | `010` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-volatility-boundary-violation` |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | `010` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `010.BoundaryErrorContext` | `error-registry-010-activation-artifact-core-conflict` |
| `VOLATILE_ROW_IN_CORE_SPEC` | `010` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `010.BoundaryErrorContext` | `error-registry-010-volatile-row-in-core-spec` |

### BoundaryErrorContext

`BoundaryErrorContext` is the owner context schema for every `010.BoundaryErrorRegistryFragment` row.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `010` context schema version. |
| `owner_spec` | Yes | Must be `010`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `direct_source_call`, `projection_authority`, `private_binding`, `activation_catalog_private_binding`, `telemetry_authority`, `undeclared_authority`, `volatility_boundary`, or `activation_artifact_conflict`. |
| `operation` | Yes | Operation that attempted publication, persistence, projection, telemetry export, or activation. |
| `affected_record_type` | Yes | Public artifact or record type; null only when no record exists. |
| `field_path` | Yes | Exact public field path when known; null only when the violation is artifact-wide. |
| `artifact_refs` | Yes | Redacted refs for public artifact, activation artifact, telemetry artifact, or projection profile. |
| `blocking_reason` | Yes | Bounded reason explaining why the output is forbidden or blocked. |
| `validation_refs` | Yes | Exact `120` validation refs proving the boundary behavior. |
| `redaction_classes` | Yes | Must map private binding values, credentials, routes, tenant inventories, source-native identity values, and raw payload bytes to `always_forbidden`. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `010-CLEANUP-AC-001` | No banned reference class remains. |
| `010-CLEANUP-AC-002` | Direct source calls still fail before output with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-CLEANUP-AC-003` | Private binding leakage validation still rejects public artifacts that expose private source binding artifacts. |
| `010-CLEANUP-AC-004` | Every output class still maps to exactly one authority class. |
| `010-MAPPING-PRIVATE-BINDING-AC-001` | Public mapping catalogs, source-extension rule sets, validation summaries, and `VersionManifest` refs that expose private bindings, raw fixture bytes, tenant inventories, private routes, credentials, or source-native secrets fail with `PRIVATE_BINDING_LEAK` before persistence or publication. |
| `010-SCHEMA-PATCH-AC-001` | Every 040 exported record maps to exactly one authority class. |
| `010-SCHEMA-PATCH-AC-002` | `EvidenceRef` remains supporting evidence and cannot become raw payload storage or fact authority by itself. |
| `010-SCHEMA-PATCH-AC-003` | Graph delta primitive shapes remain derived projection records. |
| `010-VOLATILITY-AC-001` | No activation-controlled artifact can define product authority. |
| `010-VOLATILITY-AC-002` | Activation artifacts that conflict with stable core fail before production output. |
| `010-VOLATILITY-AC-003` | Product authority, identity semantics, temporal semantics, and projection boundaries remain stable-core owned. |
| `010-SCOPE-SELECTOR-PUBLIC-BINDING-AC-001` | Public `030.ScopeSelector` and `030.ActivationScope` values containing raw tenant IDs, private routes, scanner sites, host lists, account lists, credentials, backend internal IDs, private endpoint URLs, private inventories, or raw private fixture bytes fail before persistence, publication, export, API output, audit output, telemetry export, package report materialization, or validation report materialization. |
| `010-SCOPE-SELECTOR-PUBLIC-BINDING-AC-002` | Private bindings may map concrete values to redacted refs or hashes, but selector normalization, coverage, specificity, ambiguity, default exactness, subset eligibility, and activation gates remain byte-identical to public selector behavior. |
| `010-SOURCE-CLOSURE-PRIVATE-BINDING-AC-001` | A public source-authority row containing a private route or concrete tenant inventory fails before persistence, publication, export, or validation-report materialization. |
| `010-SOURCE-CLOSURE-PRIVATE-BINDING-AC-002` | A public source-closure row, deterministic block row, closure validation result, or closure acceptance report containing a private route, concrete product name, tenant inventory, scanner site, zone inventory, account list, host list, or credential fails with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response materialization, or validation-report materialization. |
| `010-IDENTITY-PRIVATE-BINDING-AC-001` | Public resolver profile rows, identifier scope rows, and package-supplied resolver artifacts containing concrete tenant inventories, private routes, credentials, scanner site names, or account lists fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response, package report, or validation-report materialization. |
| `010-IDENTITY-PRIVATE-BINDING-AC-002` | A public `ResolverActivationReport` containing a concrete scanner site, directory tenant inventory, cloud account list, route, credential, host list, environment-specific source target list, or source-native identity value fails with `PRIVATE_BINDING_LEAK`; no public artifact, package report, validation report, API response, or export output is materialized. |
| `010-ACTIVATION-CATALOG-PUBLIC-BINDING-AC-001` | Public activation-catalog closure-pack rows fail before materialization when they contain private routes, concrete vendors, tenant inventories, scanner site names, host lists, account lists, backend credentials, private source-native object IDs, private schema payloads, or raw private fixture bytes. |
| `010-TELEMETRY-AUTHORITY-AC-001` | Runtime telemetry cannot mutate authoritative records, replace audit events, satisfy replay equivalence, activate packages, advance watermarks, or prove source, identity, fact, or graph correctness. |
| `010-TELEMETRY-PRIVATE-BINDING-AC-001` | Telemetry private-binding leaks fail before export or publication. |
| `010-AC-005` | Every error exported by `010` appears in `110.ErrorCodeRegistry`, uses `110.StandardErrorCallerFields` and `110.StandardErrorAuditFields`, has no `TODO` values, and appears in `120.Required negative tests by owner`. |

### Structured input repository acceptance criteria

| ID | Criterion |
| --- | --- |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-001` | Git commit, branch, tag, pull request, merge, repository URL, hook result, or commit timestamp cannot satisfy source authority, source completeness, identity authority, fact authority, graph authority, package activation, production approval, or rollback eligibility. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-002` | Public structured-input repository artifacts reject or redact private routes, credentials, tenant inventories, source-native secrets, raw fixture bytes, and raw structured input bytes before persistence or output. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-003` | Mutable Git refs fail with `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` before activation, rollback, or `VersionManifest` satisfaction. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-004` | Maintenance SDK/CLI success does not activate output and cannot satisfy source authority, validation acceptance, package activation, production approval, rollback eligibility, or system-of-record state. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-005` | Repository template conformance does not activate feed, mapping, source-authority, resolver, temporal, graph, registry, or package output. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-006` | Producer CI success is stale and rejected unless bound to the exact commit, tree, selected paths, file manifest checksum, toolchain refs, and validation matrix refs. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-007` | Publication manifest digest, size, media type, package type, validation, redaction, or materialization mismatch blocks package release handoff before activation. |
| `010-STRUCTURED-INPUT-BOUNDARY-AC-008` | Candidate sync records cannot activate, roll back, mutate, or authorize production state. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `010-AC-001` | A production run attempting an enterprise source API call fails before any output write with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-AC-002` | Every output class is classified as `system_of_record`, `derived_projection`, `supporting_evidence`, `non_authoritative_analysis`, or `inactive_future_domain`. |
| `010-AC-003` | No active spec permits graph, CIM, OCSF export, analysis, enrichment, lineage, registry, or package tooling output to become authoritative without an imported named authority contract. |
| `010-AC-004` | Public artifacts fail validation when they contain private source binding artifacts, route names, credential fields, or environment-specific source target lists. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
