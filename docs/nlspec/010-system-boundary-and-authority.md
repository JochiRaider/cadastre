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

- None.

## Exports

- `AuthorityClass`
- `PublicPrivateBoundaryRule`
- `ProjectionAuthorityRule`
- `RuntimeTelemetryAuthorityRule`
- `DirectSourceProhibition`
- `SourceOfRecordRule`
- `VolatilityBoundaryRule`

## Boundary Contract

Cadastre must be a lakehouse-fed interpretation, normalization, identity, fact, and projection system. It must not collect directly from configured enterprise source systems in the default production boundary.

| Input class | Production status | Required handling |
| --- | --- | --- |
| Declared lakehouse table snapshot | Allowed | Must be named by `LakehouseSnapshotRef` imported from `020`. |
| Declared dataset version | Allowed | Must be named by `DatasetVersionRef` imported from `020`. |
| Object-store raw batch | Allowed | Must be named by `RawFeedManifest` imported from `020`. |
| Supplier-provided metadata | Allowed | Must be interpreted only through active authority, completeness, temporal, and mapping policies. |
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

If an activation-controlled artifact conflicts with a stable core contract, the artifact fails activation before any production output. If two stable core contracts conflict, the spec set fails validation. If a stable core spec embeds concrete volatile rows, the rows must be non-normative examples or `TODO:` blockers unless represented as an activation-controlled artifact.

## Public and Private Source Binding

The public NLSpec set must define vendor-neutral feed contracts. Concrete vendor names, private source binding artifacts, routes, credentials, and environment-specific bindings must live in private implementation artifacts and must not appear in public canonical records.

Core schemas may name vendor-neutral source categories and redacted refs only. A public `RawRecord`, `EvidenceRef`, graph delta, API response, validation report, package report, resolver row catalog, or export artifact that exposes a private source binding must fail with `PRIVATE_BINDING_LEAK` before publication or persistence.

| Artifact | Public canonical model status |
| --- | --- |
| `RawSupplierProfile` | Public, vendor-neutral. |
| `LakehouseFeedProfile` | Public, vendor-neutral. |
| `PrivateSourceFeedBinding` | Private only. |
| `PrivateFeedSchemaInventory` | Private only. |
| `PrivateCompletenessEvidenceInventory` | Private only. |
| `PrivateGoldenCorpus` | Private only unless redacted into `LakehouseFeedFixture`. |

### SourceAuthorityRowPublicBindingRule

Public source-authority row catalogs may contain only vendor-neutral `source_category`, `source_dataset`, row IDs, redacted artifact refs, canonical scope selectors, lifecycle status, validation refs, and checksummed public policy refs.

`SourceAuthorityProfileRow`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ControlResultMappingRow`, `SupplierCollectionVisibilityProfile`, `SourceHistoryRetentionProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, and `LakehouseFeedCompletenessProfileRow` public instances must not contain concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventory, zone inventory, account lists, or environment-specific source target lists.

A public row, validation report, export, API response, or persisted public artifact that contains a concrete product name, tenant ID, private route, credential, host list, scanner site name, directory tenant inventory, zone inventory, account list, or environment-specific source target list must fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, or validation-report materialization.

Private source binding artifacts may map concrete upstream systems to public vendor-neutral datasets. Private mappings must not alter the public `060` row-resolution algorithm, matching specificity, default omission behavior, error precedence, or allowed effect semantics.

### IdentityResolverRowPublicBindingRule

Public identity resolver row catalogs may contain only vendor-neutral source categories, source datasets, row IDs, redacted artifact refs, canonical scope selectors, lifecycle status, validation refs, and checksummed policy refs.

`ResolverProfileRow`, `IdentifierScope`, `IdentifierEvidenceClass`, `AssetGenerationBoundary`, `CandidateGenerationProfile`, `TargetSelectorSafetyPolicy`, `IdentityReviewRoutingPolicy`, `SplitPolicy`, `IdentityConfidenceBand`, `ResolverDecisionMatrix`, and `ResolverExplanationPolicy` public instances must not contain concrete product names, tenant IDs, private routes, credentials, host lists, scanner site names, directory tenant inventory, zone inventory, account lists, or environment-specific source target lists.

A public resolver row, package activation report, validation report, export, API response, or persisted public artifact that contains a concrete product name, tenant ID, private route, credential, host list, scanner site name, directory tenant inventory, zone inventory, account list, or environment-specific source target list must fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response materialization, package report materialization, or validation-report materialization.

Private source binding artifacts may map concrete upstream systems to public resolver scopes. Private mappings must not alter the public `070` resolver row-selection algorithm, evidence authority defaults, blocker precedence, decision semantics, review totality, split policy semantics, selector safety, or explanation checksum policy.

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

## Required Errors

| Error code | Emitted when |
| --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | Production execution attempts to call or authenticate to an enterprise source system. |
| `PROJECTION_AUTHORITY_VIOLATION` | A derived projection attempts to create authoritative records. |
| `PRIVATE_BINDING_LEAK` | A public artifact, including identity resolver rows and package-supplied resolver artifacts, contains a concrete private vendor/source binding. |
| `TELEMETRY_AUTHORITY_VIOLATION` | Runtime telemetry attempts to create, modify, authorize, validate, replay, or persist a non-telemetry authoritative Cadastre record. |
| `TELEMETRY_PRIVATE_BINDING_LEAK` | Telemetry contains a private source binding, private route, credential, tenant inventory, or environment-specific private value before redaction/export. |
| `UNDECLARED_AUTHORITY_CLASS` | A record is written without an owner declaring its authority class. |
| `VOLATILITY_BOUNDARY_VIOLATION` | An activation-controlled artifact attempts to define product authority or stable core semantics. |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | An activation-controlled artifact conflicts with the owner stable core contract. |
| `VOLATILE_ROW_IN_CORE_SPEC` | A stable core spec embeds production-active volatile rows without artifact classification, example-only marking, or blocking `TODO:`. |

Error precedence:

| Condition | Required precedence |
| --- | --- |
| Caller is not authorized to know the object exists and the response could reveal a private binding | `AUTHORIZATION_ERROR` must precede `PRIVATE_BINDING_LEAK` in caller-visible API responses. |
| Public docs, exports, validation reports, or persisted public artifacts contain private binding data | `PRIVATE_BINDING_LEAK` must precede generic validation errors. |
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
| `UNDECLARED_AUTHORITY_CLASS` | `010` | `110` |
| `VOLATILITY_BOUNDARY_VIOLATION` | `010` | `110` |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | `010` | `110` |
| `VOLATILE_ROW_IN_CORE_SPEC` | `010` | `110` |

### BoundaryErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | default_severity | default_retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `DIRECT_SOURCE_CALL_FORBIDDEN` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-direct-source-call-forbidden` |
| `PROJECTION_AUTHORITY_VIOLATION` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-projection-authority-violation` |
| `PRIVATE_BINDING_LEAK` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-private-binding-leak` |
| `TELEMETRY_AUTHORITY_VIOLATION` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-telemetry-authority-violation` |
| `TELEMETRY_PRIVATE_BINDING_LEAK` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-telemetry-private-binding-leak` |
| `UNDECLARED_AUTHORITY_CLASS` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-undeclared-authority-class` |
| `VOLATILITY_BOUNDARY_VIOLATION` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-volatility-boundary-violation` |
| `ACTIVATION_ARTIFACT_CORE_CONFLICT` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-activation-artifact-core-conflict` |
| `VOLATILE_ROW_IN_CORE_SPEC` | `010` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `010.BoundaryErrorContext` | `boundary-error-volatile-row-in-core-spec` |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `010-CLEANUP-AC-001` | No banned reference class remains. |
| `010-CLEANUP-AC-002` | Direct source calls still fail before output with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-CLEANUP-AC-003` | Private binding leakage validation still rejects public artifacts that expose private source binding artifacts. |
| `010-CLEANUP-AC-004` | Every output class still maps to exactly one authority class. |
| `010-SCHEMA-PATCH-AC-001` | Every 040 exported record maps to exactly one authority class. |
| `010-SCHEMA-PATCH-AC-002` | `EvidenceRef` remains supporting evidence and cannot become raw payload storage or fact authority by itself. |
| `010-SCHEMA-PATCH-AC-003` | Graph delta primitive shapes remain derived projection records. |
| `010-VOLATILITY-AC-001` | No activation-controlled artifact can define product authority. |
| `010-VOLATILITY-AC-002` | Activation artifacts that conflict with stable core fail before production output. |
| `010-VOLATILITY-AC-003` | Product authority, identity semantics, temporal semantics, and projection boundaries remain stable-core owned. |
| `010-SOURCE-CLOSURE-PRIVATE-BINDING-AC-001` | A public source-authority row containing a private route or concrete tenant inventory fails before persistence, publication, export, or validation-report materialization. |
| `010-IDENTITY-PRIVATE-BINDING-AC-001` | Public resolver profile rows, identifier scope rows, and package-supplied resolver artifacts containing concrete tenant inventories, private routes, credentials, scanner site names, or account lists fail with `PRIVATE_BINDING_LEAK` before persistence, publication, export, API response, package report, or validation-report materialization. |
| `010-TELEMETRY-AUTHORITY-AC-001` | Runtime telemetry cannot mutate authoritative records, replace audit events, satisfy replay equivalence, activate packages, advance watermarks, or prove source, identity, fact, or graph correctness. |
| `010-TELEMETRY-PRIVATE-BINDING-AC-001` | Telemetry private-binding leaks fail before export or publication. |

| `010-AC-005` | Every error exported by `010` appears in `110.ErrorCodeRegistry` and in `120.Required negative tests by owner`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `010-AC-001` | A production run attempting an enterprise source API call fails before any output write with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `010-AC-002` | Every output class is classified as `system_of_record`, `derived_projection`, `supporting_evidence`, `non_authoritative_analysis`, or `inactive_future_domain`. |
| `010-AC-003` | No active spec permits graph, CIM, OCSF export, analysis, enrichment, lineage, registry, or package tooling output to become authoritative without an imported named authority contract. |
| `010-AC-004` | Public artifacts fail validation when they contain private source binding artifacts, route names, credential fields, or environment-specific source target lists. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
