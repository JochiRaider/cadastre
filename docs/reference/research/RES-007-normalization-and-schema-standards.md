---
doc_id: RES-007
project: Cadastre
title: Normalization and Schema Standards Research Report for Cadastre
status: research-report
inspection_date: 2026-05-16
---

## 1. Executive verdict

Cadastre should **adapt OCSF as the primary external silver-schema validation profile**, **adapt ECS as a naming, field-set, datatype, and artifact-generation reference**, **adapt Sigma only for analysis-rule metadata and translation validation**, **adapt MISP only for threat-intel enrichment and sharing controls**, **adapt OpenLineage only for pipeline dataset IO lineage**, and **adapt OpenMetadata only for registry, governance, classification, glossary, policy, and metadata-standard patterns**. None of these systems should define Cadastre identity, source authority, source completeness, gold facts, bitemporal semantics, graph edges, or production approval. Cadastre’s PRD already states the correct boundary: `CadastreSilverObservation.normalized_fields` must align to a pinned OCSF profile unless an observation is explicitly `cadastre_only`, while Cadastre-owned envelope fields preserve omission, lineage, source identity, field quality, temporal semantics, confidence, identity inputs, and graph derivation.[^1]

### 1.1 Ranked top findings

| Rank | Finding | Decision | Primary PRD target |
| ---: | --- | --- | --- |
| 1 | Cadastre needs an explicit `ExternalSchemaProfile` governance contract that pins OCSF source tag, source commit, compiler, validator, compiled artifact checksum, class allowlist, profile set, extension set, deprecated-field policy, and enum mapping behavior. | adopt | `ExternalSchemaProfile`, `ExternalSchemaArtifactRef`, `ProfileResolutionManifest` |
| 2 | `CadastreSilverObservation.normalized_fields` should be validated against compiled OCSF artifacts, but OCSF `raw_data`, `unmapped`, `observables`, `enrichments`, `status`, `severity`, and `confidence` must remain non-authoritative or disabled by explicit policy. | adopt | `CadastreSilverObservation.normalized_fields`, `OCSFBaseEventFieldPolicy` |
| 3 | ECS should sharpen Cadastre field dictionary structure, field-set naming, type conventions, custom/source-specific field policy, and artifact generation, but ECS must not become a secondary canonical schema. | adapt | `CadastreSilverObservation.source_extension_fields`, `SourceSchemaImportProfile`, `MappingCompilerPipeline` |
| 4 | Sigma should inform `AnalysisRuleBundle`, `RuleGraphCompatibilityMatrix`, log-source mapping, rule status, false-positive metadata, severity labels, validation schemas, and backend translation ambiguity tests. | adapt | `AnalysisRuleBundle`, `RuleGraphCompatibilityMatrix`, `MappingValidationRule` |
| 5 | MISP should inform threat-intel enrichment records, indicators, sightings, object-template validation, taxonomies, galaxies, confidence/classification tags, and distribution controls. | adapt | threat-intel enrichment boundary, `GraphPropertyEvidencePolicy`, visibility/redaction policies |
| 6 | OpenLineage should improve `RunDatasetIOContract` and `LineageFacetMappingPolicy`, especially immutable custom facet schema URLs, job/run/dataset identity, parent-run facets, source-code facets, schema facets, and dataset-version facets. | adapt | `RunDatasetIOContract`, `LineageFacetMappingPolicy`, `VersionManifest` |
| 7 | OpenMetadata should improve registry and governance contracts: ownership, domains, data products, glossaries, classifications, policies, custom properties, schema standards, metadata APIs, and lineage APIs. | adapt | `ExternalSchemaProfile`, `ArtifactClassPolicy`, registry governance |
| 8 | Unknown fields, unknown enums, deprecated fields, custom fields, profile selection, extension selection, and compiled artifact checksums must be handled uniformly across all external schema profiles. | adopt | `ExternalEnumMappingRule`, `MappingValidationRule`, `CanonicalValidationOutput` |
| 9 | Cadastre should add an external-schema profile upgrade gate that requires diff, replay, shadow, class allowlist, enum, deprecated-field, profile, extension, and golden-corpus evidence before activation. | adopt | `OCSFProfileUpgradeReport`, `VersionManifest` |
| 10 | Cadastre should explicitly reject every pattern that treats external observables, detections, indicators, sightings, lineage facets, metadata graphs, taxonomy labels, or dataset names as identity or fact authority. | adopt | `SourceAuthorityProfile`, `CoverageAssertion`, `EvidenceRef`, `GraphEdgeSemantics` |

### 1.2 Adopt, adapt, reject, study decision table

| Source | Decision | Cadastre use | Non-transferable boundary |
| --- | --- | --- | --- |
| OCSF | adopt with constraints | Primary external schema profile for `normalized_fields`. | Not identity, completeness, source authority, gold facts, graph, bitemporal, or raw evidence. |
| ECS | adapt | Field-set structure, naming, datatypes, artifact-generation tooling, custom-field collision policy. | Not canonical fact, identity, OCSF replacement, graph, or lakehouse schema. |
| Sigma | adapt | Detection-rule metadata, logsource mapping, rule validation, translation ambiguity tests. | Not asset schema, source truth, source completeness, or gold derivation by itself. |
| MISP | adapt | Threat-intel enrichment, indicators, sightings, taxonomies, galaxies, object templates, distribution controls. | Not enterprise asset identity, completeness, source authority, gold fact, or graph edge authority. |
| OpenLineage | adapt | Pipeline run/job/dataset lineage and non-authoritative facets. | Not evidence authority, source completeness, table snapshot proof, identity, gold, or graph evidence. |
| OpenMetadata | adapt | Governance registries, classifications, glossaries, policies, ownership, metadata standards, connector metadata. | Not Cadastre canonical fact graph, source authority, evidence lineage, or graph serving. |
| OCSF extensions | study before production | Declared OCSF extension set only after compiled artifact validation. | Must not be confused with Cadastre `source_extension_fields`. |
| ECS as validation profile | reject for MVP | Use only as naming/tooling reference. | Secondary schema validation would create OCSF/ECS ambiguity. |
| Sigma graph querying | study | Permit only after `RuleGraphCompatibilityMatrix` proves graph profile compatibility. | Rules must remain read-only. |
| MISP/STIX conversion | study | Projection or import/export compatibility only. | STIX/MISP graph semantics must not define enterprise asset identity. |

### 1.3 Highest-risk boundary mistakes

| Boundary mistake | Contract at risk | Required guard |
| --- | --- | --- |
| Treating OCSF objects or observables as Cadastre graph nodes. | `GraphProjectionProfile`, `IdentityDecision` | Observables disabled by default; graph nodes derive only from gold facts or declared structural rules. |
| Treating OCSF `raw_data` or `unmapped` as Cadastre raw evidence. | `RawRecord`, `EvidenceRef` | Raw payloads remain in bronze; unknown fields go to raw evidence or declared `source_extension_fields`. |
| Treating ECS custom fields as uncontrolled `source_extension_fields`. | `CadastreSilverObservation.source_extension_fields` | Every source extension field must be declared, typed, bounded, and validated. |
| Treating Sigma rule matches as source truth. | `SourceAuthorityProfile`, `GoldFact` | Sigma results are analysis outputs unless a separate derivation rule consumes them. |
| Treating MISP indicator equality as asset identity. | `IdentityDecision` | Indicators are enrichment context unless resolver rules convert non-MISP evidence into identity evidence. |
| Treating OpenLineage dataset names or facets as table snapshot proof. | `LakehouseSnapshotRef`, `EvidenceRef` | Dataset lineage may reference snapshot refs, but cannot replace them. |
| Treating OpenMetadata metadata graph as Cadastre graph serving. | `GraphReadModelSchemaProfile`, `GraphEdgeSemantics` | Metadata graph is governance context only. |

### 1.4 Highest-value PRD changes

Cadastre should add or refine the following contracts first:

1. `ExternalSchemaProfile` and `ExternalSchemaArtifactRef` for compiled, checksummed external schema governance.
2. `ProfileResolutionManifest` for OCSF inheritance, profiles, required fields, recommended fields, constraints, and object-path resolution.
3. `ExternalEnumMappingRule` for enum ID/name siblings, unknown values, `Other`, raw preservation, and deprecated enum behavior.
4. `OCSFBaseEventFieldPolicy` for `raw_data`, `raw_data_hash`, `unmapped`, `observables`, `enrichments`, severity, status, confidence, metadata, message, and timezone fields.
5. `SourceSchemaImportProfile`, `SemanticOverlayArtifact`, `MappingProjectManifest`, `MappingCompilerPipeline`, `MappingValidationRule`, and `CanonicalValidationOutput` for schema-import and mapping-bundle determinism.
6. `AnalysisRuleBundle` and `RuleGraphCompatibilityMatrix` for Sigma-derived detection content.
7. `RunDatasetIOContract` and `LineageFacetMappingPolicy` for OpenLineage-style lineage without evidence-authority leakage.
8. `ArtifactClassPolicy` for preventing static DAG, executed run, freshness, semantic, lineage, validation, table-state, and graph-rebuild artifact substitution.

### 1.5 Deeper validation required before specification changes

The PRD must not finalize production OCSF mapping rows until Cadastre compiles the exact OCSF source tree, records compiler and validator versions, captures the compiled artifact checksum, and validates every MVP observation mapping against the class allowlist. The PRD already records this as an open product decision for the exact compiled OCSF schema artifact, OCSF compiler/validator toolchain, profile behavior verification, source-corpus coverage, extension policy, and base-event field policy.[^2]

## 2. Source inventory and inspection limits

Inspection date for this report: **2026-05-16**.

| Source family | Repository or docs inspected | Version, tag, branch, release, or commit observed | Evidence class | Limits |
| --- | --- | --- | --- | --- |
| OCSF | `ocsf/ocsf-schema`, release page, README, raw `1.8.0` schema files, compiler README, validator README, extension docs | Stable release `1.8.0`, tag `1.8.0`, commit `6fa6499`; release page dated March 18, 2026. Default-branch commit was not established in this run. | Repository source and official docs | No local clone. Compiler and validator were not run. Compiled schema artifact was not produced. |
| ECS | `elastic/ecs` docs, field reference, release notes, GitHub releases, generator tooling docs/source snippets | Official ECS docs field reference version `9.4.0`; GitHub main version file rendered as `9.5.0-dev`; GitHub releases page exposed `v9.4.0-rc1` and `v9.3.0`. Stable `v9.4.0` repo tag commit was not verified. | Official docs and repository-source snippets | No local generator run. Generated CSV/template artifacts were not downloaded or checksummed. |
| Sigma | `SigmaHQ/sigma`, `SigmaHQ/sigma-specification`, rules spec, JSON schema, correlation/modifier/tag docs, release page | Sigma specification `v2.1.0`; release page shows tag `v2.1.0`, commit `0c857d0`; rules specification release date August 2, 2025; rendered release page date September 12, 2025. | Repository source and official specification docs | Rule repository current commit not established. pySigma backend behavior was not independently inspected except where source docs mention conversion assumptions. |
| MISP | `MISP/MISP`, `MISP/misp-objects`, `MISP/misp-taxonomies`, `MISP/misp-galaxy`, MISP published standards | MISP core release `v2.5.37`, commit `be735cf`, April 29, 2026. MISP core format Internet-Draft draft-20 dated November 26, 2025. Object/taxonomy/galaxy repo exact current commits were not established. | Repository release/docs and published standards | No MISP instance, PyMISP client, enrichment module, or STIX conversion tool was run. |
| OpenLineage | `OpenLineage/OpenLineage`, OpenLineage spec docs, object model, naming conventions, facets, custom facets | Docs version `1.47.1`; exact release commit was not established from the rendered pages in this run. | Official docs and repository README | No client libraries, integrations, or backend were run. API schema was not downloaded and checksummed. |
| OpenMetadata | `open-metadata/OpenMetadata`, OpenMetadata docs, OpenMetadata Standards, OpenMetadata Standards repository | Platform release `1.12.8`, tag `1.12.8-release`, commit `5d66de8`, May 13, 2026. Standards site version `1.13.0`, April 2026. | Repository release/docs and standards docs | No OpenMetadata server, ingestion connector, search index, lineage API, or schema generation was run. Platform release and Standards version differ and are treated separately. |
| Cadastre PRD | Uploaded `PRD-Cadastre.md` | Draft uploaded 2026-05-16 in project context | Governing baseline | Used only as Cadastre target contract, not as external-source evidence. |
| Prior Cadastre research | Uploaded `RES-005-Lakehouse-Derived-Graph-Architecture.md` | Research report inspected only for existing Cadastre lineage/lakehouse context | Project context | Not used to establish external facts for this report. |

### 2.1 Source limits that affect confidence

No repository was cloned locally. No schema compiler, validator, mapping generator, ECS generator, Sigma validator, MISP validation script, OpenLineage client, or OpenMetadata code generator was run. Claims about runtime behavior are limited to official source files, official docs, release pages, and source-grounded inference. Any production PRD change that depends on compiled artifacts must require executable evidence and artifact checksums before activation.

## 3. Cadastre baseline and normalization boundary

Cadastre’s governing boundary is:

```text
Raw / bronze source evidence
  -> CadastreSilverObservation
  -> OCSF-aligned normalized_fields when applicable
  -> Cadastre-owned omission, lineage, confidence, source authority, identity, temporal, and field-quality metadata
  -> Gold bitemporal facts
  -> deterministic graph deltas and optional external projections
```

The PRD establishes these normative constraints:

| Layer | Cadastre contract |
| --- | --- |
| Raw / bronze | Must preserve source-native evidence, source identity, collection metadata, payload hashes, and ingest lineage. It must not contain canonical identity decisions or derived truth. |
| Silver | Must use `CadastreSilverObservation` envelopes. `normalized_fields` may align to OCSF, but omission states, field quality, lineage, source metadata, identity inputs, confidence, temporal semantics, and flow-role evidence remain Cadastre-owned. |
| Gold | Must represent canonical bitemporal facts with valid time, known time, evidence, confidence, source authority, and assertion state. |
| Projection | Must deterministically derive graph deltas and external projections. External projections must not create authoritative facts. |
| Graph serving | Must be a replaceable read model. Graph nodes and edges must have drillback to gold, silver, and raw evidence metadata. |

Cadastre must therefore decide **where each external-schema concept belongs**:

| External concept class | Cadastre placement |
| --- | --- |
| OCSF event classes, objects, profiles, enum sibling pairs | `normalized_fields` and `ExternalSchemaProfile` only. |
| OCSF `raw_data`, `unmapped`, `observables`, `enrichments`, status, severity, confidence | Governed by `OCSFBaseEventFieldPolicy`; not authoritative by default. |
| ECS field sets and types | Mapping guidance and optional source-extension-field discipline; not a validation profile by default. |
| Sigma rules | `AnalysisRuleBundle` and validation artifacts only. |
| MISP indicators, objects, taxonomies, galaxies, sightings | Threat-intel enrichment and sharing controls only. |
| OpenLineage run/job/dataset/facets | `RunDatasetIOContract` and `LineageFacetMappingPolicy` only. |
| OpenMetadata entities, classifications, glossaries, policies, ownership, custom properties | Governance registry metadata only. |

## 4. OCSF independent analysis

### 4.1 Problem solved

OCSF solves a cybersecurity event and observation normalization problem. Its repository describes OCSF as an open standard made of categories, event classes, data types, and an attribute dictionary, with core schema definitions for consistent representation across tools and platforms.[^3]

### 4.2 Scope and non-scope

| Scope item | OCSF status |
| --- | --- |
| Event class taxonomy | Strong. Categories and event classes are primary schema structures. |
| Attribute dictionary | Strong. `dictionary.json` is a shared attribute reference. |
| Objects and profiles | Strong. Objects are reusable nested structures; profiles augment classes or objects. |
| Extensions | Strong. OCSF supports schema extension by attributes, objects, categories, profiles, and event classes. |
| Validation of schema-source files | Strong. OCSF has metaschemas and an official validator. |
| Runtime ingestion | Non-scope. OCSF is schema, not a collection system. |
| Storage format | Non-scope. OCSF is format-agnostic. |
| Identity resolution | Non-scope for Cadastre. OCSF objects contain identifiers but do not resolve Cadastre canonical entities. |
| Source completeness, bitemporal gold facts, graph projection | Non-scope for Cadastre. |

### 4.3 Repository or system layout

OCSF repository layout includes `events/`, `objects/`, `profiles/`, `extensions/`, `metaschema/`, `templates/`, `categories.json`, `dictionary.json`, and `version.json`.[^3]

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| `categories.json` | Category registry. | Class allowlist and category UID/name verification. |
| `dictionary.json` | Attribute definitions, types, enum sibling relationships. | Field dictionary and enum mapping. |
| `events/` | Event class source files. | MVP observation type mapping. |
| `objects/` | Reusable nested object definitions. | Device, endpoint, user, vulnerability, package, DNS answer, network endpoint mappings. |
| `profiles/` | Profile mix-ins. | Explicit profile selection and profile resolution manifest. |
| `extensions/` | Extension definitions. | Governed OCSF extension set, distinct from Cadastre source-extension fields. |
| `metaschema/` | JSON Schema files for OCSF source schema documents. | Validation of OCSF schema source and custom extensions. |
| Compiler and validator | Compile and validate schema source. | Required before active `ExternalSchemaProfile`. |

### 4.4 Major components and data model

OCSF base events require classification, occurrence, and metadata fields. In the raw OCSF `1.8.0` base event file, required base-event attributes include `activity_id`, `category_uid`, `class_uid`, `metadata`, `severity_id`, `time`, and `type_uid`; optional or recommended context includes `raw_data`, `raw_data_hash`, `unmapped`, `observables`, `enrichments`, status fields, message, and timezone offset.[^4]

The dictionary defines enum ID/name sibling conventions. For example, `activity_id` is a normalized integer with sibling `activity_name`, and `Other` values direct the producer to preserve a data-source-specific value in the sibling field.[^5]

### 4.5 Schema model

OCSF schema source is JSON. Validation uses metaschemas and repository consistency checks. The official validator documentation states that individual OCSF source files can be incomplete because include mechanisms require repository-aware validation; validator checks include JSON validity, directory structure, include/profile/extends targets, required attributes, unrecognized attributes, dictionary usage, and name collisions.[^6]

### 4.6 Field or attribute dictionary

OCSF’s attribute dictionary is the strongest external reference for Cadastre silver field names. Cadastre should use it only inside `normalized_fields`. It must not use OCSF dictionary attributes to define Cadastre gold fact predicates, source authority, identity decisions, or graph edge semantics.

### 4.7 Type system or taxonomy

OCSF has a taxonomy of categories and event classes. It also has object types, data types, enum definitions, requirements, constraints, profiles, and extension definitions. Cadastre should treat the OCSF taxonomy as an external classification attached to silver observations, not as Cadastre’s canonical fact taxonomy.

### 4.8 Extension model

OCSF extensions can add attributes, objects, categories, profiles, and event classes. OCSF contribution docs require extension UID/name reservation to avoid collisions, and extension directories carry an `extension.json` with extension name, UID, and version.[^7]

Cadastre rule:

```text
OCSF extensions and Cadastre source_extension_fields are distinct mechanisms.
An OCSF extension may affect normalized_fields only when compiled into the active OCSF artifact.
A Cadastre source_extension_field may preserve source-specific material without becoming an OCSF field.
```

### 4.9 Versioning and compatibility model

The stable production baseline should be pinned to an OCSF release, not `main`. The OCSF release page shows stable `1.8.0` as a release from March 18, 2026, tag `1.8.0`, commit `6fa6499`.[^8]

### 4.10 Validation strategy and artifact generation

The official schema compiler compiles OCSF schema source into a JSON-compatible compiled schema, and the validator checks schema-source consistency.[^9]

Cadastre validation sequence:

```text
ValidateOCSFNormalizedFields(observation, external_schema_profile) -> result

1. Load ExternalSchemaProfile by schema_profile_id.
2. Verify profile_status = active.
3. Verify external_schema_name = ocsf.
4. Verify external_schema_version, source tag, source commit, compiler version, validator version, profile set, extension set, and compiled artifact checksum.
5. Load ExternalSchemaArtifactRef bytes and verify compiled_schema_sha256.
6. Resolve class row from class_allowlist by observation_type, category_uid, class_uid, activity_id, and type_uid.
7. Verify normalized_fields category_uid, class_uid, activity_id, type_uid, and sibling names match the class row and ExternalEnumMappingRule.
8. Apply ProfileResolutionManifest for inherited fields, required fields, recommended fields, object paths, profiles, and constraints.
9. Reject deprecated fields unless a non-expired waiver permits them.
10. Apply OCSFBaseEventFieldPolicy for raw_data, raw_data_hash, unmapped, observables, enrichments, status, severity, confidence, metadata, message, and timezone fields.
11. Validate normalized_fields against the compiled artifact.
12. Emit CanonicalValidationOutput with deterministic diagnostics and checksum.
```

### 4.11 Mapping or translation workflow

Cadastre should require a mapping row for each MVP observation type:

```text
Cadastre observation type
  -> OCSF category UID/name
  -> OCSF class UID/name
  -> activity_id/activity_name mapping
  -> type_uid/type_name mapping
  -> required object paths
  -> profile set
  -> enum sibling mapping rules
  -> deprecated-field policy
  -> base-event field policy rows
  -> Cadastre-owned envelope fields retained outside OCSF
```

### 4.12 Unknown-field behavior

OCSF `unmapped` exists, but Cadastre must not use it as a generic dump. Unknown source fields must remain in raw evidence or declared `source_extension_fields`, not in `normalized_fields.unmapped` unless an explicit policy and validation scenario allow it. The PRD already defines this as a default policy.[^10]

### 4.13 Unknown-enum behavior

Cadastre must not invent OCSF enum IDs. It may use OCSF `Other` only when the compiled enum contains an `Other` value and an active `ExternalEnumMappingRule` permits it. The raw source value must be preserved through `field_quality.raw_value_ref` or source-extension evidence. If no valid OCSF enum value can be emitted, the mapping must either reject the OCSF output or declare the observation `cadastre_only`.

### 4.14 Deprecated-field behavior

Default production behavior must be `reject`. A waiver may allow a deprecated field only when it names the replacement, owner, expiration, reason, validation scenario IDs, and golden-corpus cases.

### 4.15 Custom-field or source-specific-field behavior

Cadastre should prefer `source_extension_fields` over OCSF extension authoring for source-specific material. OCSF extensions should be reserved for durable, cross-source schema needs and must be compiled into the active artifact.

### 4.16 Lineage or provenance model

OCSF has metadata, raw-data-related fields, and product context, but those must not replace Cadastre `RawRecord`, `EvidenceRef`, `VersionManifest`, `SourceCompletenessReceipt`, or `RunDatasetIOContract`.

### 4.17 Operational model

OCSF is a schema profile, not an operation runtime. Cadastre must own ingestion, mapping, validation, replay, evidence, completeness, and graph projection.

### 4.18 Public APIs and developer interfaces

OCSF developer interfaces include source schema files, schema browser, compiler, validator, metaschemas, releases, and contribution workflows. Cadastre should use these as toolchain inputs only after `ToolchainDependencyReview` and `ExternalToolCapabilityEvidence`.

### 4.19 Test and validation assets

Cadastre must build its own golden corpus. OCSF validation proves schema conformity, not Cadastre correctness.

### 4.20 Examples relevant to Cadastre

| Cadastre observation type | Candidate OCSF family |
| --- | --- |
| `host_identifier_observation` | Discovery / Inventory Info |
| `device_state_observation` | Discovery / Inventory Info |
| `software_observation` | Discovery / Software Inventory Info |
| `vulnerability_finding_observation` | Findings / Vulnerability Finding |
| `identity_membership_observation` | IAM / Group Management |
| `user_host_activity_observation` | IAM / Authentication or related IAM activity |
| `dns_resolution_observation` | Network / DNS Activity |
| `ip_assignment_observation` | Network / DHCP Activity or Discovery fallback |
| `firewall_flow_observation` | Network / Network Activity |
| `control_state_observation` | Findings family with security-control profile, exact class TBD by compiled artifact |

### 4.21 Confirmed findings

| Finding | Evidence |
| --- | --- |
| OCSF is a schema framework with categories, event classes, data types, dictionary, objects, profiles, extensions, metaschema, and templates. | Confirmed from OCSF repository README. |
| OCSF `1.8.0` is an inspectable stable release with tag and commit. | Confirmed from OCSF release page. |
| Base events include required classification and occurrence fields plus optional/recommended raw, unmapped, observable, enrichment, status, severity, and confidence-related fields. | Confirmed from raw `1.8.0` base event source. |
| OCSF has official compiler and validator tooling. | Confirmed from tool README pages. |
| Extensions require UID/name reservation to avoid collisions. | Confirmed from OCSF contribution docs. |

### 4.22 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| OCSF is the strongest external candidate for Cadastre silver `normalized_fields`. | High |
| Cadastre must pin compiled OCSF artifacts rather than validating against mutable source files or docs. | High |
| OCSF `Other` plus sibling strings provide a useful pattern for unknown enum preservation, but Cadastre must add field-quality evidence. | High |
| OCSF profiles are useful, but production activation must be explicit and compiled. | High |

### 4.23 Concepts transferable to Cadastre

| Concept | Cadastre transfer |
| --- | --- |
| Category/class/activity/type identifiers | Adopt for silver external classification. |
| Dictionary attributes | Adopt inside OCSF-aligned `normalized_fields`. |
| Required/recommended/optional fields | Adapt into profile validation; do not equate “recommended” with Cadastre correctness. |
| ID/name enum siblings | Adopt through `ExternalEnumMappingRule`. |
| Metaschema validation | Adopt for extension/source schema validation. |
| Compiler + compiled artifact | Adopt as `ExternalSchemaArtifactRef`. |
| Extension UID registry | Adapt for collision avoidance. |

### 4.24 Concepts that must not transfer to Cadastre

| OCSF concept | Must not transfer as |
| --- | --- |
| OCSF event classes | Cadastre gold fact types. |
| OCSF objects | Cadastre canonical entities. |
| OCSF observables | Graph nodes or identity decisions. |
| OCSF `raw_data` | Cadastre raw evidence. |
| OCSF `unmapped` | Uncontrolled source-extension dump. |
| OCSF severity/status/confidence | Cadastre risk, assertion state, or confidence without policy translation. |
| OCSF metadata | `VersionManifest` or source lineage. |

### 4.25 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Exact compiled OCSF artifact for production | Compile and checksum OCSF `1.8.0` with declared profiles/extensions. |
| Exact profile set for MVP | Product Architecture must decide after class mapping compile validation. |
| Control-state observation class | Validate exact OCSF class candidate in compiled artifact. |
| Network flow initiator semantics | Preserve Cadastre `FlowRoleEvidence`; do not rely only on OCSF source/destination. |
| Unknown enum rule coverage | Define `ExternalEnumMappingRule` for every enum field used by the MVP. |

## 5. ECS independent analysis

### 5.1 Problem solved

Elastic Common Schema provides a common field dictionary for events ingested into Elastic-oriented search, observability, and security pipelines. Official ECS docs state that ECS defines field names, datatypes, descriptions, examples, field levels, naming guidelines, and custom-field guidance.[^11]

### 5.2 Scope and non-scope

| Scope item | ECS status |
| --- | --- |
| Field sets | Strong. ECS organizes fields into field sets. |
| Naming conventions | Strong. Lowercase, underscore-separated, field-set-prefixed names. |
| Datatypes | Strong. ECS defines mapping-oriented datatypes. |
| Generated artifacts | Strong. Tooling generates templates, Beats configs, CSV, and docs. |
| Custom fields | Strong as guidance. ECS is permissive and allows custom fields. |
| Event class taxonomy | Partial. ECS has `event.*` fields, not an OCSF-like class taxonomy. |
| Identity resolution | Non-scope for Cadastre. |
| Gold facts and graph model | Non-scope for Cadastre. |
| Lakehouse schema | Non-scope for Cadastre. |

### 5.3 Repository or system layout

ECS field reference docs state that Base fields are the only root fields and all other field sets are objects; generated CSV is available for a single-page representation.[^12] ECS generator tooling loads schema files, validates YAML, applies field reuse/subset/exclude filters, and generates flat/nested schemas plus CSV, Elasticsearch templates, Beats configs, and Markdown docs.[^13]

### 5.4 Major components

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| Base fields | Root event fields. | Naming discipline only. |
| `event.*` | Event metadata and dataset naming. | Source-dataset and analysis mapping guidance. |
| `host.*`, `user.*`, `process.*`, `file.*` | Entity-context field sets. | Mapping-table support for common source fields. |
| `network.*`, `source.*`, `destination.*`, `client.*`, `server.*` | Network and flow field sets. | Flow-role mapping evidence reference, not graph direction authority. |
| `cloud.*`, `container.*`, `orchestrator.*` | Cloud/container context. | Source extension and normalized field support. |
| Generated artifacts | CSV, templates, docs. | Pattern for Cadastre schema artifact generation. |

### 5.5 Data model

ECS is field-set based. It does not require a single event-class model. It is permissive and allows additional unmapped data via custom field names.[^14]

### 5.6 Schema model

ECS schema source can generate multiple downstream artifacts. This is directly useful for Cadastre’s `MappingCompilerPipeline` because Cadastre also needs deterministic source schemas, field dictionaries, validation outputs, and generated documentation.

### 5.7 Field or attribute dictionary

ECS field sets are useful for source mapping tables where source products expose host, user, process, network, cloud, container, file, DNS, and event concepts under inconsistent names. ECS field paths can help mapping authors choose stable, conventional names for `source_extension_fields`.

### 5.8 Type system or taxonomy

ECS datatypes are search/indexing-oriented. Cadastre should adapt the naming and type discipline, but it must translate into Cadastre scalar types and OCSF types where applicable.

### 5.9 Extension model and custom-field behavior

ECS custom field docs state that ECS does not define custom fields; they are additional fields defined by users or integrations, and this flexibility is by design. ECS also warns about collision risk with future ECS fields and provides strategies to reduce it.[^15]

Cadastre transfer:

```text
source_extension_fields must be declared in a source-owned namespace.
source_extension_fields must not use OCSF-reserved field names.
source_extension_fields must not use uncontrolled ECS-style permissiveness.
```

### 5.10 Versioning and compatibility model

ECS follows Semantic Versioning and the docs/reference expose versioned field references. The inspected docs expose ECS `9.4.0`, while the GitHub main version file rendered as `9.5.0-dev`; production Cadastre references must therefore pin release artifacts, not main branch docs.[^16]

### 5.11 Validation strategy and artifact generation

ECS generator tooling is a strong reference for Cadastre artifact generation. Cadastre should not import ECS validation as a production runtime schema check for silver observations because that would conflict with OCSF-aligned `normalized_fields`.

### 5.12 Mapping or translation workflow

ECS can sharpen Cadastre mapping workflow by requiring:

```text
source field
  -> Cadastre silver envelope field or normalized_fields path
  -> optional ECS reference field-set path for authoring guidance
  -> datatype translation
  -> unknown-field behavior
  -> source_extension_fields namespace
  -> collision policy
```

### 5.13 Unknown-field behavior

ECS permits custom fields. Cadastre should not. Cadastre must require declared source-extension fields with type, bounds, lineage, unknown behavior, and redaction policy.

### 5.14 Unknown-enum behavior

ECS does not provide a universal enum sibling model comparable to OCSF. Cadastre should use OCSF `ExternalEnumMappingRule` for OCSF fields and Cadastre-owned enum mapping for envelope fields.

### 5.15 Deprecated-field behavior

ECS versioned docs and release notes can identify schema changes, but Cadastre should own deprecation behavior. Every source-extension or ECS-reference field must have a mapping validation rule for deprecated references.

### 5.16 Lineage or provenance model

ECS includes event/source context fields but does not provide Cadastre evidence lineage, table lineage, or run lineage.

### 5.17 Operational model and APIs

ECS assumes Elastic-compatible ingestion and indexing contexts. Cadastre must not import Elasticsearch mapping assumptions as lakehouse or graph-serving requirements.

### 5.18 Test and validation assets

ECS generated artifacts and tooling structure are useful references. Cadastre must still generate its own `CanonicalValidationOutput`.

### 5.19 Examples relevant to Cadastre

| ECS field-set concept | Cadastre use |
| --- | --- |
| `host.*` | Host identifier and device-state mapping-table support. |
| `user.*`, `group.*` | Identity membership and user-host activity authoring support. |
| `event.*` | Source dataset, event kind, event category metadata guidance. |
| `network.*`, `source.*`, `destination.*` | Flow-role authoring support; not direction authority. |
| `cloud.*`, `container.*` | Cloud/container context in `source_extension_fields` or OCSF objects. |

### 5.20 Confirmed findings

| Finding | Evidence |
| --- | --- |
| ECS defines field names, datatypes, descriptions, examples, levels, naming guidelines, and custom-field guidance. | Official ECS docs. |
| ECS field reference is organized by field sets; Base is root, other sets are objects. | ECS field reference. |
| ECS tools generate CSV, Elasticsearch templates, Beats configs, and docs. | ECS tooling docs and generator source. |
| ECS is permissive and allows custom fields. | ECS docs. |

### 5.21 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| ECS should improve Cadastre mapping authoring, not production silver validation. | High |
| ECS custom-field guidance is useful but too permissive without Cadastre declaration and validation. | High |
| ECS artifact generation is a strong pattern for deterministic Cadastre mapping compiler outputs. | Medium-high |

### 5.22 Concepts transferable to Cadastre

| Concept | Transfer |
| --- | --- |
| Field sets | Adapt for mapping tables and source-extension structure. |
| Naming rules | Adapt for Cadastre-owned source extensions. |
| Datatype conventions | Adapt only after translation into Cadastre scalar types. |
| Artifact generation | Adopt as a pattern for deterministic outputs. |
| Custom-field collision avoidance | Adapt with stricter declaration. |

### 5.23 Concepts that must not transfer to Cadastre

| ECS concept | Must not transfer as |
| --- | --- |
| ECS event fields | Gold fact model. |
| ECS field sets | OCSF replacement. |
| ECS custom fields | Uncontrolled Cadastre source-extension fields. |
| Elasticsearch mappings | Lakehouse table schema or graph schema. |
| ECS search conventions | Cadastre query contract. |

### 5.24 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Exact ECS stable source artifact | Pin a release tag and generated CSV if ECS is used in tooling references. |
| ECS/OCSF conflict handling | Add mapping validation rules that prioritize Cadastre and OCSF. |
| Source-extension namespace policy | Define collision-safe namespace and casing rules. |

## 6. Sigma independent analysis

### 6.1 Problem solved

Sigma solves portable detection-rule expression across heterogeneous log sources and backend query languages. The main Sigma repository describes itself as the main rule repository for detection engineers and threat hunters, with more than 3000 rules.[^17]

### 6.2 Scope and non-scope

| Scope item | Sigma status |
| --- | --- |
| Detection rule metadata | Strong. |
| Logsource abstraction | Strong. |
| Detection expression structure | Strong. |
| Status lifecycle | Strong. |
| Backend translation assumptions | Strong as a design pressure. |
| Rule schema validation | Strong. |
| Asset schema | Non-scope. |
| Identity, source truth, completeness, gold facts | Non-scope. |

### 6.3 Repository or system layout

Sigma has a rule corpus repository and a separate specification repository. The specification repository contains the rule, correlation, filter, modifier, tag, taxonomy, and convention materials. The current specification release page exposes `v2.1.0` with commit `0c857d0`.[^18]

### 6.4 Major components

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| Rule metadata | Title, ID, related rules, name, taxonomy, status, description, references, author, dates. | `AnalysisRuleBundle` metadata. |
| `logsource` | Product/service/category/definition abstraction. | Source-category and source-dataset mapping. |
| `detection` | Search identifiers and conditions. | Read-model query validation, not source truth. |
| Conditions and modifiers | Portable detection logic. | Translation ambiguity tests. |
| Tags | ATT&CK, CVE, D3FEND, TLP, etc. | Analysis context only. |
| `falsepositives` and `level` | Analyst metadata. | Finding metadata; not Cadastre risk by default. |
| Correlation | Multi-event or meta-rule behavior. | Future rule bundle capability only after compatibility matrix. |

### 6.5 Data model and schema model

The Sigma rule specification says rule files are YAML and contain mandatory `title`, `logsource`, and `detection` sections, with optional metadata and arbitrary custom fields.[^19] The JSON schema requires `title`, `logsource`, and `detection`; it defines status values `stable`, `test`, `experimental`, `deprecated`, and `unsupported` and level values `informational`, `low`, `medium`, `high`, and `critical`.[^20]

### 6.6 Field or attribute dictionary

Sigma does not define an asset field dictionary. Its taxonomy concept can define field names, field values, and logsource names, and non-default taxonomies must be handled or transformed by tools.[^21]

Cadastre implication:

```text
A Sigma-derived rule must declare the taxonomy it expects.
A Sigma-derived rule must not run against Cadastre graph output until RuleGraphCompatibilityMatrix maps every required field, node, edge, direction, and property.
```

### 6.7 Type system or taxonomy

Sigma’s taxonomy and tags are rule metadata and translation aids. They should not become Cadastre classification authority.

### 6.8 Extension model

Sigma permits arbitrary custom fields in YAML. Cadastre must not allow arbitrary rule metadata to affect execution. Every production-affecting field must be declared in `AnalysisRuleBundle`.

### 6.9 Versioning and compatibility model

Sigma rule statuses and related-rule relationships create a useful lifecycle pattern. Cadastre should map Sigma `stable/test/experimental/deprecated/unsupported` into Cadastre lifecycle states only through a policy table, not automatically.

### 6.10 Validation strategy

Sigma uses JSON schema and repository conventions. Cadastre must add graph compatibility validation:

```text
ValidateAnalysisRuleBundle(bundle, graph_profile) -> result

1. Validate Sigma YAML against declared Sigma spec version and JSON schema.
2. Reject unsupported or deprecated rules unless policy permits validation-only use.
3. Normalize rule metadata into AnalysisRuleBundle.
4. Resolve logsource rows to Cadastre source categories, datasets, observation types, or graph profiles.
5. Compile detection expression into a declared query target.
6. Verify every referenced field, node type, edge type, edge direction, graph property, temporal field, and assertion-state filter through RuleGraphCompatibilityMatrix.
7. Run positive, negative, stale, conflicted, unauthorized, and empty-result validation scenarios.
8. Emit CanonicalValidationOutput.
```

### 6.11 Mapping or translation workflow

Sigma translation ambiguity is a direct lesson for Cadastre: every source-to-common-language translation must use explicit mapping rows. Sigma’s field usage section states that mapped field names should be used when field mappings exist, otherwise raw source field names may be used after stripping spaces.[^22] Cadastre should be stricter: no production rule may rely on unmapped raw field names.

### 6.12 Unknown-field behavior

A Sigma rule field without a Cadastre graph/profile mapping must reject production activation.

### 6.13 Unknown-enum behavior

Sigma string and modifier semantics are backend-dependent. Unknown Cadastre enum values must be handled through rule-specific expected behavior: reject, ignore, include as unknown, or require explicit filter behavior.

### 6.14 Deprecated-field behavior

Sigma `deprecated` status should map to non-production or replay-only behavior unless Cadastre grants a waiver.

### 6.15 Custom-field behavior

Custom Sigma fields may be stored as non-authoritative rule metadata. They must not affect query execution unless declared in `AnalysisRuleBundle`.

### 6.16 Lineage or provenance model

Sigma references can cite rule inspiration, but they do not satisfy Cadastre evidence lineage.

### 6.17 Operational model and APIs

Sigma rules usually run through converters or pySigma-like ecosystems. Cadastre should treat converters as build-time or validation-time tools subject to dependency review.

### 6.18 Test and validation assets

Cadastre should require rule validation scenarios and graph compatibility matrices, not just Sigma schema validation.

### 6.19 Examples relevant to Cadastre

| Sigma concept | Cadastre use |
| --- | --- |
| `logsource.category = process_creation` | Map to observation/source datasets or graph profile if process events are supported later. |
| `logsource.category = firewall` | Map to `firewall_flow_observation` or graph flow read model only after field compatibility validation. |
| `level = high` | Rule severity metadata; not Cadastre risk score. |
| `falsepositives` | Analyst explanation metadata. |
| `status = deprecated` | Lifecycle policy input. |

### 6.20 Confirmed findings

| Finding | Evidence |
| --- | --- |
| Sigma rule structure has mandatory title/logsource/detection and optional metadata. | Sigma rules spec. |
| Sigma logsource uses product, service, category, and definition. | Sigma rules spec. |
| Sigma has JSON schemas and status/level vocabularies. | Sigma JSON schema and spec. |
| Sigma taxonomy may define field names, values, and logsource names; non-default taxonomy must be transformed by tooling. | Sigma rules spec. |

### 6.21 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| Sigma should influence analysis rules, not schema normalization. | High |
| Sigma translation ambiguity should force Cadastre mapping-table completeness. | High |
| Sigma rules may query Cadastre graph read models only after compatibility validation. | High |

### 6.22 Concepts transferable to Cadastre

| Concept | Transfer |
| --- | --- |
| Rule metadata | Adopt in `AnalysisRuleBundle`. |
| Logsource abstraction | Adapt for source-category/dataset mapping. |
| Status lifecycle | Adapt through lifecycle policy. |
| Rule schemas | Adapt for validation artifacts. |
| Backend translation ambiguity | Adopt as rejection tests for mapping compiler. |

### 6.23 Concepts that must not transfer to Cadastre

| Sigma concept | Must not transfer as |
| --- | --- |
| Detection match | Source truth. |
| Logsource | Source completeness proof. |
| Rule severity | Cadastre risk score without policy. |
| Tags | Asset classifications or graph edges. |
| Custom rule fields | Production-affecting fields without declaration. |

### 6.24 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Rule execution target | Decide which Cadastre graph profiles Sigma-derived rules may query. |
| Backend translation | Define supported translator or query compiler and test corpus. |
| Correlation rules | Add explicit state, window, ordering, and deduplication contracts before use. |

## 7. MISP independent analysis

### 7.1 Problem solved

MISP solves threat-intelligence sharing, event/attribute exchange, indicator context, object templates, taxonomies, galaxies, sightings, distribution, and sharing control. MISP published standards describe the core format as JSON for exchanging indicators and threat information between MISP instances.[^23]

### 7.2 Scope and non-scope

| Scope item | MISP status |
| --- | --- |
| Threat-intel events and attributes | Strong. |
| Indicators and observables | Strong. |
| Object templates | Strong. |
| Taxonomies and machine tags | Strong. |
| Galaxies and clusters | Strong. |
| Sightings | Strong. |
| Distribution and sharing groups | Strong. |
| Enterprise asset identity | Non-scope for Cadastre. |
| Source completeness and gold facts | Non-scope for Cadastre. |

### 7.3 Repository or system layout

| Repository or standard | Role |
| --- | --- |
| `MISP/MISP` | Core platform and REST/API surfaces. |
| `MISP/misp-objects` | Object templates. |
| `MISP/misp-taxonomies` | Machine tag vocabularies. |
| `MISP/misp-galaxy` | Galaxies and clusters for threat context. |
| MISP core format | JSON event/attribute exchange standard. |
| MISP object template format | Object-template standard. |
| MISP taxonomy format | Taxonomy/machine tag standard. |
| MISP galaxy format | Galaxy/cluster standard. |
| SightingDB format | Sighting count/observability standard. |

MISP release `v2.5.37` introduced an event templating system, STIX 2 stack changes, and security/performance fixes.[^24]

### 7.4 Major components

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| Event | Container for coherent threat-intel context. | Enrichment batch or source record context. |
| Attribute | Category/type/value triplet. | Indicator or observable enrichment. |
| Object template | Structured multi-attribute object. | Threat-intel enrichment validation. |
| Taxonomy | Machine tags/triple tags. | Classification metadata and analysis context. |
| Galaxy | Clusters for threat actors, malware, campaigns, tools, etc. | Enrichment context only. |
| Sighting | Observability/count/time window. | Enrichment signal, not completeness. |
| Distribution/sharing group | Access/dissemination semantics. | Evidence visibility and redaction policy. |
| REST/PyMISP | API interface. | Adapter/importer reference only. |

### 7.5 Data model

MISP events are JSON objects containing metadata and attributes. The MISP core format states that an event UUID must be preserved, and attributes are category-type-value triplets.[^25]

### 7.6 Schema model

MISP object templates define structured objects. The object-template standard describes public object-template directories and definitions, while the repository shows object templates with attributes such as MISP attribute type, categories, multiple value support, UI priority, disable-correlation flags, allowed values, defaults, and `to_ids` behavior.[^26]

### 7.7 Field or attribute dictionary

MISP’s field dictionary is threat-intel-specific: attribute categories, attribute types, object-template attributes, galaxy values, taxonomy predicates/values, and sighting fields.

### 7.8 Type system or taxonomy

MISP taxonomies are JSON machine-tag vocabularies. The taxonomy standard describes namespace directories, `machinetag.json`, and a manifest with metadata, version, license, URL, and path.[^27]

### 7.9 Extension model

MISP object templates, taxonomies, galaxies, and event templates are dynamic governance artifacts. Cadastre should adapt this model for enrichment package registries, but every enrichment artifact must be versioned, checksummed, lifecycle-controlled, and non-authoritative by default.

### 7.10 Versioning and compatibility model

MISP core and related standards publish versioned drafts or schemas. Cadastre must pin exact template/taxonomy/galaxy versions when enrichment output affects analysis results or graph properties.

### 7.11 Validation strategy

Cadastre should validate MISP-derived enrichment as:

```text
ValidateThreatIntelEnrichment(input, profile) -> result

1. Verify source instance, MISP format version, object-template version, taxonomy version, galaxy version, and distribution policy.
2. Validate event UUID and attribute category/type/value structure.
3. Validate object-template fields and allowed values.
4. Validate taxonomy namespace, predicate, and value.
5. Validate galaxy cluster references and confidence fields.
6. Validate sighting time/count/TTL fields if present.
7. Apply distribution and sharing controls to visibility labels and graph property redaction.
8. Emit enrichment records only; do not emit identity decisions, source completeness receipts, gold facts, or graph deltas.
```

### 7.12 Mapping or translation workflow

| MISP concept | Cadastre placement |
| --- | --- |
| Attribute indicator | Enrichment record or silver observation only when source record is itself an observed indicator. |
| Object template | Enrichment object schema. |
| Taxonomy tag | Classification/analysis context. |
| Galaxy cluster | Threat context. |
| Sighting | Enrichment observability signal. |
| Distribution | Visibility/redaction constraint. |

### 7.13 Unknown-field behavior

Unknown MISP object-template fields must be preserved as enrichment raw context or rejected by the enrichment profile. They must not enter Cadastre gold facts by default.

### 7.14 Unknown-enum behavior

Unknown taxonomy, galaxy, category, or type values must preserve raw value and template/version context. Cadastre may classify them as `unknown` enrichment context; it must not coerce them into known classifications.

### 7.15 Deprecated-field behavior

Deprecated MISP templates, taxonomies, or galaxies may support historical replay. Production use requires a non-expired waiver or migration mapping.

### 7.16 Custom-field behavior

MISP local taxonomies or local object templates must be declared as enrichment artifacts with owner, version, checksum, distribution policy, and lifecycle status.

### 7.17 Lineage or provenance model

MISP provenance includes event UUIDs, references, distribution, sharing groups, sightings, and taxonomy/galaxy sources. Cadastre must map this to enrichment lineage only.

### 7.18 Operational model and APIs

MISP uses REST APIs internally and externally, and PyMISP is described by MISP glossary material as the de-facto standard client surface.[^28]

### 7.19 Test and validation assets

Cadastre should include enrichment validation scenarios for: known indicator, unknown indicator type, taxonomy unknown value, galaxy cluster missing, sighting TTL expired, distribution restricted, and raw value redacted.

### 7.20 Examples relevant to Cadastre

| MISP source | Cadastre use |
| --- | --- |
| MISP indicator attribute for IP/domain/hash | Threat-intel enrichment linked to a Cadastre entity or identifier by explicit evidence. |
| MISP sighting for indicator | Enrichment observability, not source completeness. |
| MISP galaxy threat actor | Analysis context, not graph edge by default. |
| MISP taxonomy TLP or classification | Visibility and sharing label input. |
| MISP distribution/sharing group | Evidence visibility and graph property redaction. |

### 7.21 Confirmed findings

| Finding | Evidence |
| --- | --- |
| MISP core format defines JSON exchange of indicators and threat information. | MISP standards. |
| MISP events preserve UUIDs and contain attribute structures. | MISP core format. |
| MISP object templates, taxonomies, galaxies, and sightings have published standards. | MISP standards. |
| MISP distribution values define sharing/dissemination scope. | MISP core format. |
| MISP core release `2.5.37` is current as inspected and includes event templating and STIX-stack changes. | MISP release page. |

### 7.22 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| MISP should improve Cadastre enrichment and visibility controls, not asset identity. | High |
| MISP sightings are observability signals but not source completeness. | High |
| MISP galaxy labels are threat context but not graph edges by themselves. | High |
| MISP object templates can inspire enrichment schema artifact governance. | Medium-high |

### 7.23 Concepts transferable to Cadastre

| Concept | Transfer |
| --- | --- |
| Event and attribute model | Adapt for threat-intel enrichment imports. |
| Object templates | Adapt for enrichment schema validation. |
| Taxonomies | Adapt for classification and analysis context. |
| Galaxies | Adapt for threat context. |
| Sightings | Adapt for enrichment observability. |
| Distribution/sharing | Adopt for visibility/redaction policy inputs. |

### 7.24 Concepts that must not transfer to Cadastre

| MISP concept | Must not transfer as |
| --- | --- |
| Indicator equality | Asset identity. |
| Sightings | Source completeness. |
| Galaxy clusters | Graph edges by default. |
| Taxonomy labels | Source authority. |
| Distribution | Authorization by itself without Cadastre policy. |
| STIX conversion graph | Enterprise asset graph. |

### 7.25 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Enrichment record schema | Define threat-intel enrichment envelope and its relation to silver/gold. |
| Distribution mapping | Define mapping from MISP distribution/TLP/sharing groups to Cadastre visibility and redaction. |
| Indicator identity | Define strict rules for when an indicator may link to identifiers without resolving assets. |
| STIX/MISP conversion | Study only as import/export projection, not identity model. |

## 8. OpenLineage independent analysis

### 8.1 Problem solved

OpenLineage standardizes pipeline lineage around jobs, runs, datasets, events, and facets. Its object model is designed to observe datasets as they move through pipelines; it includes Jobs, Runs, Datasets, and lineage events.[^29]

### 8.2 Scope and non-scope

| Scope item | OpenLineage status |
| --- | --- |
| Run/job/dataset identity | Strong. |
| Run lifecycle events | Strong. |
| Input/output dataset lists | Strong. |
| Facets | Strong. |
| Custom facet naming and schema URLs | Strong. |
| Pipeline lineage | Strong. |
| Raw evidence authority | Non-scope for Cadastre. |
| Source completeness | Non-scope for Cadastre. |
| Bitemporal facts and identity | Non-scope for Cadastre. |
| Graph serving | Non-scope for Cadastre. |

### 8.3 Repository or system layout

OpenLineage includes a core specification, object model, naming conventions, run cycle, facets/extensibility, producers, schemas, examples, client libraries, and integrations.

### 8.4 Major components

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| `RunEvent` | Runtime state update for a job. | Stage run lineage and `RunDatasetIOContract`. |
| `Job` | Defined work unit. | `SourceStageDAG` or mapping compiler job metadata. |
| `Run` | Execution instance, client-generated UUID. | `run_id` mapping, but not `VersionManifest` replacement. |
| `Dataset` | Abstract data collection with namespace/name. | Dataset lineage reference, not table snapshot proof. |
| Input/output datasets | Datasets read or written by runs. | `RunDatasetIOContract`. |
| Facets | Extensible metadata on run/job/dataset. | `LineageFacetMappingPolicy`. |
| Parent, source-code, schema, dataset-version facets | Specific lineage metadata. | Non-authoritative context. |

### 8.5 Data model

OpenLineage object model says Jobs are identified by namespace/name, Runs are instances with client-generated UUIDs, and Datasets have namespace/name derived from physical location or data system.[^30]

### 8.6 Schema model

OpenLineage facets are JSON-schema-governed extension objects. Dataset schema facets can carry dataset field names, types, and descriptions; source-code facets can carry repository/path/version/branch/tag metadata.[^31]

### 8.7 Field or attribute dictionary

OpenLineage does not provide Cadastre asset field dictionaries. It provides lineage metadata fields.

### 8.8 Type system or taxonomy

OpenLineage core types are run, job, dataset, and facet types. They are lineage taxonomy, not asset taxonomy.

### 8.9 Extension model

OpenLineage custom facets must use a distinct prefix to avoid collision; they have `_schemaURL` pointing to the facet schema, and the versioned URL must be immutable, with a tag or git SHA rather than a branch name.[^32]

Cadastre transfer:

```text
LineageFacetMappingPolicy must require immutable schema URLs for production-accepted external facets.
Mutable branch URLs must be rejected for production-affecting facets.
Custom facet names must use collision-resistant prefixes.
```

### 8.10 Versioning and compatibility model

OpenLineage docs were inspected at version `1.47.1`. Cadastre should record OpenLineage spec version and facet schema URLs in every accepted lineage event.

### 8.11 Validation strategy

Cadastre validation sequence:

```text
ValidateLineageEvent(event, policy) -> result

1. Verify lineage_system = openlineage.
2. Verify event type, run, job, dataset, producer, and schema URL fields.
3. Map run/job/dataset fields to RunDatasetIOContract.
4. For every facet, resolve LineageFacetMappingPolicy row by facet name and schema URL.
5. Reject production-affecting unmapped facets.
6. Store permitted facets as run context, dataset IO, diagnostic metadata, or non-authoritative metadata.
7. Forbid any facet from satisfying EvidenceRef, SourceCompletenessReceipt, CoverageAssertion, IdentityDecision, GoldFact, GraphDeltaSet, GraphRebuildManifest, or GraphIndexConsistencyCheck unless another Cadastre authority contract supplies the evidence.
```

### 8.12 Mapping or translation workflow

| OpenLineage concept | Cadastre target |
| --- | --- |
| `run.runId` | Run lineage field, not `VersionManifest` identity by itself. |
| `job.namespace/name` | Stage/job context. |
| `inputs` and `outputs` | `RunDatasetIOContract` rows. |
| Dataset version facet | `DatasetVersionRef` pointer only when paired with table snapshot refs. |
| Schema facet | Lineage schema metadata, not source schema profile authority. |
| Parent run facet | Stage-parent metadata. |
| Source-code facet | Package/source context in `VersionManifest` only if validated. |

### 8.13 Unknown-field behavior

Unknown facets must follow `unmapped_facet_behavior`. Production-affecting unknown facets must reject.

### 8.14 Unknown-enum behavior

Unknown OpenLineage lifecycle/event values must reject unless a policy maps them to non-authoritative metadata.

### 8.15 Deprecated-field behavior

Deprecated facet schema URLs must require migration or waiver. Mutable schema URLs must reject in production.

### 8.16 Custom-field behavior

Custom facets may be stored only when collision-resistant naming, immutable schema URL, and policy mapping pass.

### 8.17 Lineage or provenance model

OpenLineage is a pipeline lineage model. It should improve pipeline lineage, but it must not replace source evidence lineage, lakehouse snapshot lineage, graph projection lineage, or evidence drillback.

### 8.18 Operational model and APIs

OpenLineage events may be emitted by pipeline integrations and accepted by lineage backends. Cadastre should persist its own lineage records and may emit OpenLineage-compatible projections.

### 8.19 Test and validation assets

Cadastre must test mapped facets, unmapped facets, mutable schema URLs, dataset version without snapshot ref, parent-run handling, source-code version handling, and failure events.

### 8.20 Examples relevant to Cadastre

| OpenLineage facet or field | Cadastre use |
| --- | --- |
| Parent run facet | Source stage parent/child run context. |
| Source-code location facet | Package source context if immutable and validated. |
| Schema dataset facet | Dataset schema lineage only. |
| Dataset version facet | May point to `DatasetVersionRef`, not table proof by itself. |
| Error message facet | Diagnostic metadata. |

### 8.21 Confirmed findings

| Finding | Evidence |
| --- | --- |
| OpenLineage object model is Job, Run, Dataset, event, facet oriented. | Official object model docs. |
| Run events describe pipeline state updates and include job, run, and datasets. | Official object model docs. |
| Jobs and Datasets have namespaces; Runs use client-generated UUIDs. | Naming docs. |
| Custom facets require distinct prefixes and immutable schema URLs. | Facets/extensibility docs. |

### 8.22 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| OpenLineage is directly useful for `RunDatasetIOContract`. | High |
| OpenLineage facets must be non-authoritative by default. | High |
| Dataset version facets cannot prove lakehouse snapshot identity by themselves. | High |

### 8.23 Concepts transferable to Cadastre

| Concept | Transfer |
| --- | --- |
| Run/job/dataset identity | Adapt into `RunDatasetIOContract`. |
| Input/output datasets | Adopt as lineage rows. |
| Parent run | Adapt into stage run context. |
| Source-code location | Adapt into version manifest/package context. |
| Facet collision avoidance | Adopt for `LineageFacetMappingPolicy`. |
| Immutable schema URLs | Adopt for external facet governance. |

### 8.24 Concepts that must not transfer to Cadastre

| OpenLineage concept | Must not transfer as |
| --- | --- |
| Run events | Production run success proof by themselves. |
| Dataset names | Table snapshot proof. |
| Dataset version facet | `LakehouseSnapshotRef` by itself. |
| Facets | Evidence authority or completeness. |
| Lineage graph | Cadastre graph serving. |

### 8.25 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Facet allowlist | Define which facets Cadastre accepts. |
| Dataset version mapping | Require pairing with `LakehouseSnapshotRef` or `DatasetVersionRef`. |
| External lineage projection | Decide whether Cadastre emits OpenLineage-compatible events. |
| Run event lifecycle mapping | Define total mapping from OpenLineage events to Cadastre lifecycle results. |

## 9. OpenMetadata independent analysis

### 9.1 Problem solved

OpenMetadata solves metadata governance: technical metadata, data quality, lineage, column-level lineage, ownership, usage, policies, conversations, glossaries, classifications, metrics, domains, and data products in a unified metadata knowledge graph with connectors, standards, APIs, SDKs, and search.[^33]

### 9.2 Scope and non-scope

| Scope item | OpenMetadata status |
| --- | --- |
| Metadata entity model | Strong. |
| JSON schemas and standards | Strong. |
| Ownership, domains, data products | Strong. |
| Glossaries and classifications | Strong. |
| Policies | Strong. |
| Custom properties | Strong. |
| Lineage APIs | Strong. |
| Change events and webhooks | Strong. |
| Search/indexing | Strong. |
| Asset-intelligence gold facts | Non-scope for Cadastre. |
| Cadastre identity/source authority/evidence completeness | Non-scope. |
| Cadastre graph serving | Non-scope. |

### 9.3 Repository or system layout

OpenMetadata platform docs and release pages show an application platform; OpenMetadata Standards provides schemas, ontologies, and API specs. The platform release inspected is `1.12.8`, while the Standards site exposes `1.13.0` materials.[^34]

### 9.4 Major components

| Component | Role | Cadastre relevance |
| --- | --- | --- |
| Entity model | Data assets, services, governance, teams, users, quality, lineage, usage. | Registry and governance inspiration. |
| JSON Schemas | Canonical metadata validation. | External profile and registry schemas. |
| RDF/OWL/SHACL/JSON-LD/PROV-O | Semantic web standards. | Governance metadata export or validation reference. |
| Custom properties | Extensible metadata. | Cadastre registry extension policy. |
| Classifications and tags | Governance classification. | Label/context policy, not source authority. |
| Glossaries | Business terms. | Registry context and UI metadata. |
| Policies | Access/governance policy. | Authorization and visibility policy inspiration. |
| Lineage API | Manage lineage relationships. | Governance lineage, not Cadastre graph serving. |
| Change events/webhooks | Metadata change notifications. | Operational events only. |
| Connectors | Metadata ingestion. | Source package metadata inspiration. |

### 9.5 Data model

OpenMetadata Standards docs state that the standards provide 700+ JSON Schemas spanning entities, relationships, events, and configuration, plus RDF/OWL, SHACL, JSON-LD, and PROV-O support.[^35]

### 9.6 Schema model

The standards are JSON-schema-first and API-first. Cadastre should adapt this into registry schemas for `ExternalSchemaProfile`, mapping artifacts, source packages, lifecycle machines, and profile governance.

### 9.7 Field or attribute dictionary

OpenMetadata metadata fields include ownership, domains, tags, glossaries, service references, fully qualified names, custom properties, versions, and entity status. These fields are useful for governance registries but not Cadastre asset facts.

### 9.8 Type system or taxonomy

OpenMetadata has entity schemas, relationships, classifications, tags, custom properties, and type systems. Cadastre should borrow governance structure, not semantics of assets.

### 9.9 Extension model

OpenMetadata custom properties and standards provide a controlled extension pattern. Cadastre should adopt explicit schema-bound custom registry properties but not arbitrary custom fact properties.

### 9.10 Versioning and compatibility model

OpenMetadata entity APIs include versions and update flows; OpenMetadata Standards version is separate from the platform release. Cadastre must record both platform/tool versions and standard/schema versions when using OpenMetadata-derived artifacts.

### 9.11 Validation strategy

Cadastre should adapt:

```text
ValidateGovernanceRegistryArtifact(artifact, policy) -> result

1. Validate JSON schema.
2. Validate owner, domain, classification, glossary, and policy references.
3. Validate custom properties against registry extension policy.
4. Reject any metadata graph edge used as Cadastre source authority, evidence, identity, gold fact, or graph serving edge.
5. Emit artifact-class validation result and checksum.
```

### 9.12 Mapping or translation workflow

| OpenMetadata concept | Cadastre placement |
| --- | --- |
| Ownership | Registry owner and approval metadata. |
| Classification/tag | Governance context or visibility policy input. |
| Glossary term | Human-facing semantic label. |
| Domain/data product | Source instance or table-profile governance grouping. |
| Custom property | Declared registry metadata extension. |
| Lineage API edge | Governance lineage only. |
| Change event/webhook | Operational event only. |

### 9.13 Unknown-field behavior

Unknown custom properties must reject unless declared by a registry extension schema.

### 9.14 Unknown-enum behavior

Unknown classification/tag/policy values must be stored as non-authoritative metadata or rejected according to registry policy. They must not become source authority.

### 9.15 Deprecated-field behavior

Deprecated registry schema fields require migration or waiver before production activation.

### 9.16 Custom-field behavior

Custom properties must be declared, typed, bounded, owner-assigned, and checksummed. They must not extend Cadastre gold facts unless a separate Cadastre contract defines the field.

### 9.17 Lineage or provenance model

OpenMetadata lineage captures relationships between entities, including column-level mappings and pipeline references.[^36] Cadastre must distinguish this from source evidence lineage, lakehouse snapshot lineage, and graph projection lineage.

### 9.18 Operational model and APIs

OpenMetadata provides APIs for classifications, lineage, data products, and many other entities. It also supports change events and webhooks as part of its standards and platform model.[^35]

### 9.19 Test and validation assets

Cadastre should create registry validation scenarios for classification, glossary, owner missing, policy missing, custom property unknown, lineage edge non-authority, and metadata graph substitution rejection.

### 9.20 Examples relevant to Cadastre

| OpenMetadata concept | Cadastre use |
| --- | --- |
| Owner refs | Owner for profiles, packages, policies, schema profiles, mappings. |
| Classifications | Governance labels and visibility policy inputs. |
| Glossaries | Human-readable semantic dictionary. |
| Domains/data products | Source package and dataset grouping. |
| Custom properties | Declared registry metadata extensions. |
| Lineage API | Governance lineage, not graph serving. |

### 9.21 Confirmed findings

| Finding | Evidence |
| --- | --- |
| OpenMetadata connects metadata, quality, lineage, ownership, policies, glossaries, classifications, metrics, domains, and data products in a metadata knowledge graph. | OpenMetadata repository README. |
| OpenMetadata Standards provides 700+ JSON Schemas and semantic-web standards. | Standards docs. |
| OpenMetadata lineage API can manage lineage relationships and column-level mappings. | Lineage API docs. |
| Platform `1.12.8` and Standards `1.13.0` are separate observed version contexts. | Release/docs pages. |

### 9.22 Source-grounded inferences

| Inference | Confidence |
| --- | ---: |
| OpenMetadata should improve Cadastre registries and governance metadata. | High |
| OpenMetadata metadata graph must not replace Cadastre graph serving. | High |
| Custom property governance is transferable only for registry metadata. | High |

### 9.23 Concepts transferable to Cadastre

| Concept | Transfer |
| --- | --- |
| JSON-schema-first metadata standards | Adopt for registry artifacts. |
| Custom properties | Adapt under strict extension policy. |
| Ownership | Adopt for governance objects. |
| Classifications and glossaries | Adapt for metadata context. |
| Policies | Adapt for visibility/governance policy. |
| Lineage API model | Adapt only for metadata lineage. |
| Change events/webhooks | Adapt as operational notifications. |

### 9.24 Concepts that must not transfer to Cadastre

| OpenMetadata concept | Must not transfer as |
| --- | --- |
| Metadata graph | Canonical Cadastre fact graph. |
| Lineage API edge | Cadastre graph edge or evidence drillback. |
| Classification/tag | Source authority. |
| Custom property | Gold fact field without Cadastre contract. |
| Search index | Source of truth. |
| Entity version | Bitemporal gold fact. |

### 9.25 Gaps and unresolved questions

| Gap | Required resolution |
| --- | --- |
| Registry schema governance | Define which Cadastre registry artifacts use OpenMetadata-inspired patterns. |
| Classification policy | Define which classifications affect visibility or redaction. |
| Custom property policy | Define extension schema, owner, checksum, and activation gates. |
| Metadata lineage boundary | Add validation that prevents metadata lineage from satisfying evidence lineage. |

## 10. Secondary references, if used

No secondary reference changed the report’s conclusions. DataHub, OpenCTI, and STIX were not independently expanded into separate source analyses. MISP/STIX conversion was considered only as a MISP-adjacent import/export boundary because the MISP release notes mention STIX-stack changes; no STIX or OpenCTI graph semantics were used to establish Cadastre requirements.

## 11. Comparative matrices

### 11.1 Matrix 1: Normalization concept coverage

Cell values are restricted to `strong`, `partial`, `weak`, `absent`, `not_applicable`, or `unknown`.

| Concept | OCSF | ECS | Sigma | MISP | OpenLineage | OpenMetadata |
| --- | --- | --- | --- | --- | --- | --- |
| schema version | strong: releases, version files, compiled artifacts | strong: versioned docs/releases | strong: spec version and rule schema | strong: core release and standards | strong: spec/docs version | strong: platform and standards versions |
| field dictionary | strong: dictionary | strong: field sets | partial: detection fields depend on taxonomy | partial: attribute/object/template dictionaries | weak: lineage fields only | strong: metadata schemas |
| type taxonomy | strong: data types/objects | strong: datatypes | partial: rule/logsource/status types | strong: attribute types, object templates, taxonomies | partial: run/job/dataset/facet types | strong: entity/types/custom properties |
| event or observation class taxonomy | strong: categories/classes | weak: event fields, no class taxonomy | partial: rule/logsource taxonomy | partial: threat events/attributes | absent: pipeline lineage only | partial: metadata entity taxonomy |
| object model | strong: objects | partial: field-set objects | weak: rule structures | strong: objects, galaxies, events | strong: run/job/dataset | strong: entities/relationships |
| profile model | strong: profiles | partial: field levels/maturity | absent | partial: templates/taxonomies/galaxies | partial: facets | strong: standards/entities/custom properties |
| extension rules | strong: extensions and UID reservation | partial: custom field guidance | partial: arbitrary fields and custom taxonomy | strong: local templates/taxonomies/galaxies | strong: custom facets | strong: custom properties/schema standards |
| custom/source-specific fields | partial: extensions/unmapped but governed | strong but permissive | partial: custom YAML fields | strong: local templates/tags | strong: custom facets | strong: custom properties |
| unknown field handling | partial: validator catches schema-source unknowns; event unknowns need policy | strong but permissive | partial: backend-dependent | partial: template/profile-dependent | partial: policy needed | partial: schema/custom property policy |
| unknown enum handling | strong: ID/name sibling and Other conventions | weak: no universal enum model | weak: backend-dependent strings | partial: raw taxonomy/type preservation | weak: event/facet enums | partial: schema enum validation |
| deprecated field handling | strong: deprecation metadata | partial: version/release notes | strong: deprecated status | partial: template/version dependent | partial: spec/version dependent | partial: schema/version dependent |
| required/recommended/optional fields | strong | partial: field levels not presence contract | strong: required rule sections | strong: standards/templates | strong: event payload requirements | strong: JSON schemas |
| validation artifacts | strong: metaschema, validator, compiler | strong: generator artifacts | strong: JSON schema | strong: schemas/templates | strong: JSON schemas/facet URLs | strong: JSON schemas/standards |
| generated artifacts | strong: compiled schema | strong: CSV/templates/docs | partial: schemas/rule packages | partial: generated docs/templates | partial: spec/client-generated events | strong: generated classes/API schemas |
| source-to-common-schema mapping | strong: intended use | partial: field mapping reference | partial: detection translation | partial: import/export/enrichment | weak: lineage only | partial: metadata connectors |
| lineage to raw input | weak | weak | absent | partial: event provenance | partial: pipeline inputs | partial: metadata lineage |
| pipeline lineage | absent | absent | absent | absent | strong | strong but governance-oriented |
| metadata governance | weak | weak | weak | partial: distribution/tags | partial: facets | strong |
| threat-intel enrichment | partial: threat/security fields | weak | partial: rule tags | strong | absent | partial: governance context |
| analysis-rule metadata | weak | weak | strong | partial | absent | partial |

### 11.2 Matrix 2: Cadastre transfer decision

| Source | Concept | Cadastre target contract | Decision | Why it transfers or does not transfer | New precision added to PRD | New complexity introduced | Required validation | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| OCSF | Compiled schema artifact | `ExternalSchemaArtifactRef` | adopt | Production validation needs immutable artifact. | Exact source tag, commit, compiler, validator, checksum. | Toolchain governance. | Compile and checksum artifact. | Mismatched artifact rejects before output. |
| OCSF | Class allowlist | `ExternalSchemaProfile` | adopt | Prevents accidental class drift. | Observation-type to class/activity rows. | Per-class validation. | Golden corpus per row. | Unsupported class rejects activation. |
| OCSF | Enum siblings and `Other` | `ExternalEnumMappingRule` | adopt | Preserves unknown enum source value. | ID/name sibling behavior. | More mapping rows. | Unknown enum cases. | Unknown value never invents enum ID. |
| OCSF | Base-event raw/unmapped/observable fields | `OCSFBaseEventFieldPolicy` | adopt | Prevents authority leakage. | Field-level allow/forbid table. | Policy/golden corpus. | Policy row per emitted field. | Forbidden field rejects mapping activation. |
| ECS | Field sets and naming | `SourceSchemaImportProfile`, `source_extension_fields` | adapt | Improves source mapping consistency. | Naming and datatype conventions. | OCSF/ECS conflict handling. | Collision tests. | Undeclared source extension rejects. |
| ECS | Artifact generator | `MappingCompilerPipeline` | adapt | Strong generator pattern. | Deterministic generated outputs. | Build tooling. | Byte-stable outputs. | Same inputs produce same checksum. |
| Sigma | Rule structure | `AnalysisRuleBundle` | adapt | Strong detection metadata. | Rule fields, lifecycle, false positives. | Rule compiler. | Schema validation. | Invalid rule fails before activation. |
| Sigma | Logsource | `RuleGraphCompatibilityMatrix` | adapt | Logsource maps to source datasets/graph profiles. | Compatibility rows. | Matrix maintenance. | Positive/negative graph scenarios. | Rule cannot run without matrix match. |
| MISP | Indicators | Threat-intel enrichment contract | adapt | Useful context. | Indicator schema and lineage. | Enrichment store. | Unknown type and redaction tests. | Indicator equality cannot merge assets. |
| MISP | Sightings | Threat-intel enrichment contract | adapt | Useful observability. | Sighting TTL/count/time fields. | TTL semantics. | Expired/unknown sighting tests. | Sighting cannot authorize completeness. |
| MISP | Distribution/sharing | `GraphPropertyEvidencePolicy` | adapt | Visibility/redaction protection. | Distribution-to-visibility mapping. | Access policy integration. | Restricted sharing cases. | Restricted enrichment redacts graph properties. |
| OpenLineage | Run/job/dataset | `RunDatasetIOContract` | adapt | Needed pipeline lineage. | IO rows and authority class. | Dataset version refs. | Dataset mapping tests. | `lineage_only` cannot satisfy evidence. |
| OpenLineage | Custom facets | `LineageFacetMappingPolicy` | adopt | Collision and immutable schema URL rules are strong. | Facet mapping table. | Facet registry. | Mutable URL rejection. | Production unmapped facet rejects. |
| OpenMetadata | Classifications/glossaries | Registry governance | adapt | Useful governance metadata. | Owner/classification/glossary refs. | Governance registry. | Unknown classification tests. | Tags cannot satisfy source authority. |
| OpenMetadata | Custom properties | Registry extension policy | adapt | Useful controlled extensibility. | Typed custom property schema. | Schema registry. | Unknown property rejection. | Undeclared custom property rejects. |
| OpenMetadata | Metadata graph | none | reject | Would replace Cadastre graph/facts incorrectly. | Explicit non-authority wording. | None. | Artifact substitution tests. | Metadata graph edge cannot be graph edge evidence. |

### 11.3 Matrix 3: Do-not-transfer list

| External concept | Why tempting | Why forbidden | Cadastre contract at risk | Required PRD wording or validation gate |
| --- | --- | --- | --- | --- |
| OCSF as gold fact model | Rich event/object taxonomy. | OCSF is event normalization, not bitemporal canonical truth. | `GoldFact` | Gold facts derive only through Cadastre gold derivation. |
| OCSF `raw_data` as Cadastre raw evidence | Field exists in base event. | Raw evidence needs payload hash, retention, access, and lineage. | `RawRecord`, `EvidenceRef` | `raw_data` forbidden in production normalized fields by default. |
| OCSF `observables` as graph nodes | Observables look like extraction candidates. | Graph nodes must derive from gold facts or declared structural rules. | `GraphProjectionProfile`, `IdentityDecision` | Observables disabled by default; cannot create nodes. |
| ECS as canonical fact model | Broad field dictionary. | Search-oriented field sets do not define facts or identity. | `GoldFact`, `SourceAuthorityProfile` | ECS is naming/tooling reference only. |
| ECS custom fields as uncontrolled `source_extension_fields` | ECS permits custom fields. | Cadastre requires declared, typed, bounded source fields. | `CadastreSilverObservation.source_extension_fields` | Undeclared source extension rejects. |
| Sigma detections as source truth | Rule matches look like findings. | Detections are analysis outputs, not source evidence. | `SourceAuthorityProfile`, `GoldFact` | Analysis rules are read-only by default. |
| Sigma logsource as source completeness proof | Names a log source. | Does not prove coverage or absence. | `SourceCompletenessReceipt` | Logsource cannot authorize absence or watermark. |
| MISP indicator equality as asset identity | Indicators match observed identifiers. | Indicator equality does not prove enterprise asset identity. | `IdentityDecision` | MISP signals contribute zero identity evidence by default. |
| MISP sightings as source completeness | Sighting counts imply observability. | Counts do not prove complete source scope. | `CoverageAssertion`, `SourceCompletenessReceipt` | Sighting cannot authorize absence/retraction/cleanup. |
| MISP galaxy labels as graph edges | Threat actor/campaign clusters look relational. | Labels are context; graph edges need fact evidence and semantics. | `GraphEdgeSemantics` | Galaxy labels stored as enrichment context only. |
| OpenLineage facets as evidence authority | Facets carry rich metadata. | Facets are lineage metadata, not raw or fact evidence. | `EvidenceRef` | `LineageFacetMappingPolicy` forbids evidence substitution. |
| OpenLineage dataset names as table snapshot proof | Dataset identity names tables. | Snapshot proof needs table-format-native refs. | `LakehouseSnapshotRef` | Dataset ref must pair with snapshot refs. |
| OpenMetadata metadata graph as canonical fact graph | Metadata graph has entities and relationships. | It is governance metadata, not Cadastre facts. | `GraphReadModelSchemaProfile`, `GoldFact` | Metadata graph cannot mark graph serving current. |
| OpenMetadata lineage API as Cadastre graph serving | Lineage API returns graph-like edges. | Cadastre graph requires evidence, temporal filters, edge semantics, and projection profiles. | `GraphEdgeSemantics`, `QueryGraph` | Lineage API edge cannot satisfy graph edge evidence. |
| OpenMetadata classifications as source authority | Tags look policy-relevant. | Tags do not prove source truth. | `SourceAuthorityProfile` | Classification labels require policy translation and cannot arbitrate facts. |
| MISP/STIX graph semantics as enterprise asset identity | Threat-intel graphs link observables and actors. | Threat-intel graph identity is not enterprise asset identity. | `IdentityDecision` | STIX/MISP graph imports remain enrichment-only unless a Cadastre resolver rule consumes independent evidence. |

### 11.4 Matrix 4: External schema profile governance

| Governance concern | OCSF | ECS | MISP object templates | OpenMetadata Standards | OpenLineage facets | Cadastre improvement |
| --- | --- | --- | --- | --- | --- | --- |
| version pinning | strong: release/tag/commit | strong docs, but stable artifact must be verified | partial: template/taxonomy/galaxy versioning | strong: standards/platform versions | strong: docs/facet versions | `ExternalSchemaProfile` must pin source, version, release, artifact. |
| artifact checksums | strong if compiled artifact is captured | strong if generated CSV/templates captured | partial; Cadastre must checksum templates | strong for JSON schemas if captured | strong if schema URL immutable and bytes captured | `ExternalSchemaArtifactRef` must include byte checksum. |
| schema generation | compiler | generator | docs/templates validation | generated classes/API schemas | schemas/facets | `MappingCompilerPipeline` must produce deterministic artifacts. |
| schema validation | validator/metaschema | generator validation | object/template/schema checks | JSON Schema/SHACL | JSON Schema facets | `CanonicalValidationOutput` must normalize diagnostics. |
| extension naming | UID/name reservation | custom namespace/capitalization guidance | local template/taxonomy namespace | custom properties | custom facet prefix | `ArtifactClassPolicy` and profile extension rows. |
| extension collision avoidance | extension registry | future-field collision guidance | namespace/version management | schema-bound custom properties | distinct facet prefix | `SourceSchemaImportProfile` collision checks. |
| deprecated element handling | deprecation metadata | release/version docs | template/version dependent | schema/version dependent | schema version dependent | `MappingValidationRule` rejects deprecated fields by default. |
| unknown element behavior | validator catches source-schema unknown; event unknown needs policy | permissive | profile/template dependent | schema custom property policy | unmapped facet policy | Unknown fields reject unless declared. |
| custom fields | OCSF extensions or `unmapped` | permissive custom fields | local templates/tags | custom properties | custom facets | Cadastre `source_extension_fields` declared separately. |
| profile/extension selection | explicit profiles/extensions | not true profile model | template/taxonomy/galaxy selection | standards/schema selection | facet selection | `ProfileResolutionManifest` and `LineageFacetMappingPolicy`. |
| compatibility upgrade workflow | compare compiled artifacts | compare generated fields | template/taxonomy diff | schema/API diff | facet schema URL diff | `OCSFProfileUpgradeReport` and profile upgrade gates. |

Concrete contract mapping:

| Cadastre contract | Required improvement |
| --- | --- |
| `ExternalSchemaProfile` | Add external source version, release/tag/commit, profile set, extension set, class allowlist, deprecated-field policy, unknown behavior. |
| `ExternalSchemaArtifactRef` | Store compiled/generated artifact bytes checksum and toolchain identity. |
| `ProfileResolutionManifest` | Resolve OCSF inherited fields, profiles, constraints, required/recommended fields, and object paths. |
| `ExternalEnumMappingRule` | Define ID/name siblings, unknown enum, raw preservation, `Other`, and no-invented-ID behavior. |
| `OCSFBaseEventFieldPolicy` | Govern raw/unmapped/observable/enrichment/status/severity/confidence fields. |
| `OCSFProfileUpgradeReport` | Require drift, compatibility, replay, shadow, enum, deprecated-field, profile, and extension checks. |
| `SourceSchemaImportProfile` | Add format-specific unknown, unsupported, source-state, and semantic metadata handling. |
| `SemanticOverlayArtifact` | Keep overlays non-authoritative and checksummed. |
| `MappingProjectManifest` | Include source roots, source schema refs, dependency locks, compiler options. |
| `MappingCompilerPipeline` | Define deterministic phases and failure diagnostics. |
| `CanonicalValidationOutput` | Byte-stable validation output. |
| `LineageFacetMappingPolicy` | Map external facets to allowed Cadastre lineage or diagnostic fields. |
| `ArtifactClassPolicy` | Forbid artifact substitution. |

### 11.5 Matrix 5: Silver observation boundary

All OCSF class candidates below are **candidates pending compiled OCSF artifact verification**.

| MVP observation type | Best OCSF class candidate | ECS field-set support | Sigma relevant | MISP relevant | OpenLineage relevant | OpenMetadata relevant | Fields that must remain Cadastre-owned | Known gaps | Validation cases required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `host_identifier_observation` | Discovery / Inventory Info | `host.*`, `agent.*`, `cloud.*` | no | indicator enrichment only | run/source dataset lineage | owner/domain metadata | identity hints, omission, field quality, source asset IDs | OCSF device object is not canonical host | weak identifier non-merge; unknown hostname; conflicting serial |
| `device_state_observation` | Discovery / Inventory Info | `host.*`, `device.*`, `os.*` | no | enrichment only | collection run lineage | ownership/classification | lifecycle state, source authority, valid/known time | OCSF status not Cadastre assertion state | stale device; source unavailable; permission-limited |
| `vulnerability_finding_observation` | Findings / Vulnerability Finding | `vulnerability.*`, `host.*`, `package.*` | possible analysis only | strong enrichment | scan run lineage | owner/policy context | coverage, absence, assertion state, confidence, source authority | OCSF finding status not Cadastre vulnerability state | present, absent with coverage, stale scanner, conflicting severity |
| `software_observation` | Discovery / Software Inventory Info | `package.*`, `host.*`, `process.*` | no | enrichment for malicious package/hash | inventory run lineage | data product/owner context | package identity, source-native package IDs, omission, coverage | package ecosystem normalization | unknown package manager; malformed version; duplicate package |
| `control_state_observation` | Findings family with security-control profile or Compliance/Security Finding candidate | `event.*`, `rule.*`, `host.*` | possible rule metadata | enrichment/context | control evaluation run lineage | policy/glossary/classification | control scope, pass/fail/unknown, coverage assertion, source authority | exact OCSF class requires compile validation | pass, fail, unknown, no coverage, waived deprecated field |
| `identity_membership_observation` | IAM / Group Management | `user.*`, `group.*` | possible analysis only | enrichment only | directory sync lineage | ownership/policy context | membership temporal interval, identity decision, source authority | OCSF group object not canonical group | add/remove, nested group, partial directory scope |
| `user_host_activity_observation` | IAM / Authentication or related IAM activity | `user.*`, `host.*`, `event.*` | strong for detection rules | enrichment only | auth log run lineage | governance context | no ownership inference, confidence, source authority | user activity does not imply membership or ownership | successful login, failed login, service account, unknown host |
| `dns_resolution_observation` | Network / DNS Activity | `dns.*`, `source.*`, `destination.*` | possible DNS detections | indicator enrichment | DNS collection lineage | data-domain context | TTL/staleness, DNS evidence, identity non-merge | DNS name/IP relationship not host identity | TTL expired, NXDOMAIN, multiple answers, malformed RR |
| `ip_assignment_observation` | Network / DHCP Activity or Discovery fallback | `host.*`, `network.*`, `source.*`, `destination.*` | no | indicator enrichment | DHCP/IPAM run lineage | owner/domain | lease interval, assignment authority, IP non-merge | DHCP vs IPAM semantics differ | lease start/end, static assignment, partial scope |
| `firewall_flow_observation` | Network / Network Activity | `network.*`, `source.*`, `destination.*`, `client.*`, `server.*` | strong for detection rules | IOC enrichment | flow collection lineage | policy context | `FlowRoleEvidence`, direction, NAT/proxy, collection point, aggregation | OCSF source/destination insufficient for graph direction | known initiator, unknown initiator, NAT, proxy, bidirectional flow |

## 12. Highest-value transferable concepts

| Rank | Concept | Transfer contract | Requirement |
| ---: | --- | --- | --- |
| 1 | Compiled external schema artifact | `ExternalSchemaArtifactRef` | Every production OCSF validation must use a checksummed compiled artifact. |
| 2 | External schema profile governance | `ExternalSchemaProfile` | Profile must declare class allowlist, profile set, extension set, compiler, validator, and checksums. |
| 3 | Profile resolution | `ProfileResolutionManifest` | Inheritance, profiles, requirements, constraints, and object paths must be resolved before mapping activation. |
| 4 | Enum sibling mapping | `ExternalEnumMappingRule` | Unknown enums must preserve raw value and may use `Other` only under policy. |
| 5 | ECS artifact-generation discipline | `MappingCompilerPipeline` | Mapping validation must emit byte-stable outputs. |
| 6 | ECS custom-field collision awareness | `source_extension_fields` policy | Source extension fields must be declared and collision-safe. |
| 7 | Sigma rule lifecycle and logsource abstraction | `AnalysisRuleBundle` | Rule status, logsource, false positives, tags, and level must be retained as analysis metadata. |
| 8 | Sigma backend translation ambiguity | `RuleGraphCompatibilityMatrix` | Rule execution requires graph profile compatibility matrix and scenarios. |
| 9 | MISP enrichment and distribution | threat-intel enrichment, `GraphPropertyEvidencePolicy` | Indicators, sightings, tags, galaxies, and distribution must remain enrichment context with redaction. |
| 10 | OpenLineage custom facet governance | `LineageFacetMappingPolicy` | Facets require immutable schema URLs and explicit allowed use. |
| 11 | OpenMetadata registry governance | registry contracts | Owners, domains, classifications, glossaries, policies, and custom properties should govern packages and profiles. |
| 12 | Artifact class separation | `ArtifactClassPolicy` | No static, freshness, lineage, semantic, validation, table, or graph artifact can substitute for another. |

## 13. Concepts Cadastre must not transfer

Cadastre must reject every external concept that could weaken the lakehouse-first, bitemporal, evidence-backed boundary.

| Concept family | Rejection rule |
| --- | --- |
| External normalization schemas | Must not define gold facts, identity, source authority, source completeness, or graph projection. |
| External threat-intel graphs | Must not define enterprise asset identity or graph edges. |
| External detection rules | Must not define source truth. |
| External metadata graphs | Must not become Cadastre canonical fact graph. |
| External lineage events | Must not replace evidence refs, completeness receipts, table snapshot refs, or graph rebuild proof. |
| External custom fields | Must not bypass Cadastre declaration, typing, bounds, redaction, and validation. |
| External severity/confidence/status | Must not become Cadastre risk, confidence, or assertion state without policy translation. |
| External dataset names | Must not replace `DatasetVersionRef` or `LakehouseSnapshotRef`. |

## 14. Gaps, risks, stale assumptions, and unresolved questions

| ID | Gap or risk | Impact | Required resolution |
| --- | --- | --- | --- |
| GAP-001 | Exact OCSF compiled artifact not produced in this report. | Production profile cannot be finalized. | Compile OCSF `1.8.0`, record compiler/validator versions and checksum. |
| GAP-002 | ECS stable `9.4.0` source tag/commit was not verified; docs and repo main differ. | ECS tooling reference could drift. | Pin exact ECS artifact or generated CSV before use. |
| GAP-003 | Sigma rule repository current commit was not established. | Rule corpus freshness cannot be guaranteed. | Pin rule package release or commit before importing rules. |
| GAP-004 | MISP object/taxonomy/galaxy current commits were not established. | Enrichment artifact freshness uncertain. | Pin each template/taxonomy/galaxy artifact checksum. |
| GAP-005 | OpenLineage release commit not established. | Facet schema refs must be pinned separately. | Record spec version and facet schema URL bytes. |
| GAP-006 | OpenMetadata platform and standards versions differ. | Governance schema version ambiguity. | Treat platform and standards as separate dependencies. |
| GAP-007 | Control-state OCSF class candidate unresolved. | MVP mapping row incomplete. | Compile OCSF and verify best class/profile. |
| GAP-008 | MISP/STIX conversion not inspected. | Import/export semantics unknown. | Study only if Cadastre needs STIX projection/import. |
| GAP-009 | External tools were not run. | Tool behavior is unverified. | Require `ExternalToolCapabilityEvidence` before production use. |
| GAP-010 | Unknown enum behavior requires per-field rows. | Mapping ambiguity. | Populate `ExternalEnumMappingRule` for every enum path. |

## 15. Concrete PRD improvement map

| Target PRD area | Current gap or ambiguity | Source evidence | Proposed contract improvement | Observable behavior changed | Acceptance criterion | Risk or downside | Priority |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ExternalSchemaProfile` | Needs complete external schema governance. | OCSF/ECS/OpenMetadata/OpenLineage | Require source tag, commit, artifact checksum, class allowlist, profile set, extension set, deprecated policy. | Profile activation rejects incomplete metadata. | Active profile has complete immutable refs. | More governance work. | high |
| `ExternalSchemaArtifactRef` | Validation must not depend on mutable docs/source. | OCSF compiler/validator, ECS generator | Store compiled/generated artifact bytes checksum and tool versions. | Replay rejects checksum mismatch. | Same artifact bytes produce same checksum. | Toolchain pinning required. | high |
| `ProfileResolutionManifest` | OCSF inherited/profile fields may be ambiguous. | OCSF profiles/metaschema | Record resolved fields, constraints, inherited requirements, object paths. | Mapping validation uses resolved manifest. | Missing manifest rejects profile-dependent mapping. | Compile complexity. | high |
| `ExternalEnumMappingRule` | Unknown enum behavior can drift. | OCSF sibling/Other model | Define known, unknown, Other, raw preservation, no invented IDs. | Unknown enum has deterministic output or rejection. | Unknown enum test passes. | Per-field rows. | high |
| `OCSFBaseEventFieldPolicy` | Base fields can leak authority. | OCSF base event, PRD policy | Keep default forbid/limited policy for raw/unmapped/observables/enrichments/status/severity/confidence. | Forbidden base field rejects activation. | Golden corpus proves policy. | Some source context excluded. | high |
| `OCSFProfileUpgradeReport` | Upgrade drift not fully mechanized. | OCSF releases | Require class, enum, profile, extension, deprecated, replay, shadow diff. | Profile upgrade cannot activate silently. | Upgrade report passes all gates. | Upgrade effort. | high |
| `CadastreSilverObservation.normalized_fields` | Needs compiled-artifact validation. | OCSF compiler/validator | Validate only against active `ExternalSchemaArtifactRef`. | Invalid normalized fields reject before write. | Invalid required field fails deterministically. | Requires compiled artifact. | high |
| `CadastreSilverObservation.source_extension_fields` | Needs stricter policy. | ECS custom-field guidance, OCSF extension distinction | Declare namespace, type, bounds, redaction, collision behavior. | Undeclared fields reject. | Collision test rejects. | Mapping author burden. | high |
| `CadastreSilverObservation.field_quality` | Unknown/omitted values must be exhaustive. | OCSF enum behavior, ECS custom fields, Sigma null/empty | Require rows for absent, empty, null, malformed, unknown, redacted, permission-withheld. | Field quality present for every nullable/optional output. | Missing row rejects mapping. | More mapping rows. | high |
| `FlowRoleEvidence` | OCSF/ECS source-destination can mislead graph direction. | OCSF/ECS network fields | Preserve initiator/responder/NAT/proxy/aggregation/collection-point outside OCSF. | Ambiguous flows produce ambiguous facts or no directional edge. | Unknown initiator test cannot emit directed edge. | More flow evidence modeling. | high |
| `SourceSchemaImportProfile` | Need format-specific source schema import behavior. | ECS generator, OpenMetadata standards | Add source schema checksum, unsupported constructs, unknown fields, semantic metadata keys. | Unsupported construct rejects unless handled. | Unsupported `oneOf` or unknown enum case rejects. | More importer policy. | high |
| `SemanticOverlayArtifact` | Must remain non-authoritative. | ECS/Sigma/OpenMetadata custom models | Require checksum, symbol table, alias graph, unsupported constructs, non-authority rule. | Overlay cannot affect identity/facts by itself. | Alias-only identity test rejects. | Authoring complexity. | medium |
| `MappingProjectManifest` | Needs source roots/dependency/toolchain determinism. | ECS generator, Sigma schemas | Record source roots, schema refs, dependency locks, compiler options, plugin refs. | Validation checksum changes when manifest changes. | Same manifest/input gives same output. | Build discipline. | high |
| `MappingCompilerPipeline` | Need deterministic phases. | ECS generator, OCSF compiler, Sigma schema validation | Define parse, import, overlay, map, lint, external schema validation, scenario phases. | Phase failures produce stable diagnostics. | Same invalid mapping yields same diagnostic code. | Compiler implementation. | high |
| `MappingValidationRule` | Need required rules across schemas. | Sigma/ECS/OCSF lessons | Add rules for unknown fields, unknown enums, deprecated fields, extension collisions, OCSF policy. | Activation rejects policy violations. | Required linter rules cannot be demoted below failure. | Rule registry upkeep. | high |
| `CanonicalValidationOutput` | Need byte-stable validation artifact. | ECS artifacts, Sigma schema | Normalize diagnostics, sorted keys, stable checksums. | CI/editor/prod validation parity. | Same inputs produce byte-identical output. | Deterministic serialization. | high |
| `ValidationScenario` | Need explicit tests for edge cases. | Sigma rules, OCSF unknowns, MISP distributions | Add positive, negative, unknown, deprecated, redaction, authorization, no-op scenarios. | Promotion requires scenario pass. | Every active mapping has required rejection cases. | Scenario maintenance. | high |
| `AnalysisRuleBundle` | Sigma content needs boundary. | Sigma spec | Store rule metadata, logsource, detection, false positives, level, tags, status; read-only. | Analysis rule cannot mutate production state. | Mutation attempt rejects. | Rule compiler needed. | high |
| `RuleGraphCompatibilityMatrix` | Graph query compatibility must be explicit. | Sigma translation ambiguity | Require node/edge/property/direction/temporal compatibility rows. | Rule cannot run against incompatible graph profile. | Missing graph property rejects. | Matrix upkeep. | high |
| `RunDatasetIOContract` | Lineage must not become evidence. | OpenLineage | Default authority class `lineage_only`; reference dataset versions. | Lineage-only rows cannot satisfy evidence/completeness. | Evidence substitution test rejects. | More lineage rows. | high |
| `LineageFacetMappingPolicy` | External facets need explicit mapping. | OpenLineage custom facets | Map facet to target, authority class, allowed/forbidden uses; immutable schema URLs. | Production unmapped facet rejects. | Mutable branch schema URL rejects. | Facet registry. | high |
| `ArtifactClassPolicy` | Artifact substitution risk. | OpenLineage/OpenMetadata/previous PRD | Forbid static DAG, freshness, lineage, semantic, validation, table, graph artifact substitutions. | Wrong artifact class rejects. | Freshness artifact cannot authorize completeness. | More artifact typing. | high |
| `GraphPropertyEvidencePolicy` | Enrichment and metadata can leak. | MISP distribution, OpenMetadata classifications | Add visibility/redaction rules for enrichment-derived graph properties. | Restricted enrichment redacted. | Restricted MISP distribution test redacts. | Policy complexity. | medium |
| `SourceAuthorityProfile` | External confidence/status/tags could leak into facts. | OCSF/MISP/Sigma/OpenMetadata | Require explicit translation rows before external status/confidence/classification influences facts. | External labels do not arbitrate facts by default. | Missing authority row rejects derivation. | More policy work. | high |
| `CoverageAssertion` | MISP sightings or Sigma logsource could be misused. | MISP sightings, Sigma logsource | Explicitly forbid enrichment/rule/logsource as coverage proof. | Absence cannot derive from sightings/rules. | Sighting-only absence rejects. | None. | high |
| `EvidenceRef` | External lineage/provenance may be overused. | OpenLineage, MISP, OCSF metadata | Evidence refs require bronze/silver/gold/external_reference with role; lineage facets default lineage-only. | Facet cannot satisfy evidence. | Facet-only evidence rejects. | More drillback discipline. | high |
| `VersionManifest` | External schema and toolchain refs must be replay-relevant. | OCSF/ECS/Sigma/OpenLineage/OpenMetadata | Add schema artifact refs, mapping output checksums, rule bundle versions, facet policies, enrichment artifacts. | Replay rejects missing/mismatched external artifact refs. | Missing profile checksum rejects production replay. | Manifest size. | high |

## 16. Acceptance criteria for adopting findings into the PRD

```text
AC-REPORT-001: Every primary source is analyzed independently before cross-source comparison.
AC-REPORT-002: Every non-trivial claim about an external source is cited to a primary source or explicitly marked as inference.
AC-REPORT-003: Every recommended transfer maps to a named Cadastre PRD contract or declares that a new contract is required.
AC-REPORT-004: Every do-not-transfer warning identifies the Cadastre contract it protects.
AC-REPORT-005: Every schema translation recommendation includes explicit handling for unknown fields, unknown enums, deprecated fields, custom/source-specific fields, and extension behavior.
AC-REPORT-006: Every OCSF recommendation distinguishes normalized_fields from Cadastre-owned envelope fields.
AC-REPORT-007: Every Sigma recommendation is scoped to detection, rule, logsource, or analysis metadata and does not define assets or facts.
AC-REPORT-008: Every MISP recommendation distinguishes threat-intel enrichment from enterprise asset identity.
AC-REPORT-009: Every OpenLineage recommendation distinguishes pipeline lineage from evidence authority, source completeness, and table snapshot proof.
AC-REPORT-010: Every OpenMetadata recommendation distinguishes metadata governance from Cadastre canonical bitemporal facts.
AC-REPORT-011: The report contains a complete do-not-transfer matrix.
AC-REPORT-012: The report contains a concrete PRD improvement map with priority and acceptance criteria.
AC-REPORT-013: The report states all source limits, unverified tools, inaccessible artifacts, and unrun builds.
```

Additional PRD adoption criteria:

```text
AC-PRD-SCHEMA-001: An active ExternalSchemaProfile cannot be created without a compiled artifact checksum.
AC-PRD-SCHEMA-002: An active mapping bundle cannot emit an OCSF enum field without an ExternalEnumMappingRule row.
AC-PRD-SCHEMA-003: An active mapping bundle cannot emit a deprecated external field unless a non-expired waiver passes validation scenarios.
AC-PRD-SCHEMA-004: An active mapping bundle cannot emit source_extension_fields unless every field is declared, typed, bounded, and namespaced.
AC-PRD-SCHEMA-005: A Sigma-derived AnalysisRuleBundle cannot run unless RuleGraphCompatibilityMatrix passes for the active graph profile.
AC-PRD-SCHEMA-006: A MISP-derived enrichment cannot create or merge CanonicalEntity records by itself.
AC-PRD-SCHEMA-007: An OpenLineage facet cannot satisfy EvidenceRef, SourceCompletenessReceipt, CoverageAssertion, IdentityDecision, GoldFact, GraphDeltaSet, GraphRebuildManifest, or GraphIndexConsistencyCheck.
AC-PRD-SCHEMA-008: An OpenMetadata classification, glossary, policy, custom property, lineage edge, or metadata graph edge cannot satisfy SourceAuthorityProfile or GraphEdgeSemantics by itself.
AC-PRD-SCHEMA-009: CanonicalValidationOutput is byte-identical for the same mapping inputs, external schema artifacts, source schema artifacts, dependency locks, compiler version, and validation scenarios.
AC-PRD-SCHEMA-010: Production replay rejects before output when any external schema artifact, mapping validation checksum, lineage facet policy, analysis rule bundle, enrichment artifact, or registry artifact referenced by VersionManifest is absent or mismatched.
```

## 17. Source notes and citations

This report distinguishes confirmed source facts from source-grounded inferences in every primary source section. It did not run builds, compilers, validators, importers, enrichment modules, graph services, OpenLineage integrations, or OpenMetadata connectors. It therefore recommends production gates that require executable validation before any external tool, compiled artifact, generated artifact, imported schema, rule bundle, lineage facet, enrichment template, or metadata standard affects Cadastre production behavior.

## Sources

[^1]: Cadastre PRD, product summary, document status, and normalization boundary. The PRD states that `CadastreSilverObservation.normalized_fields` must align to a pinned OCSF schema profile unless `cadastre_only`, that Splunk CIM is non-authoritative, and that semantic overlays do not replace Cadastre silver, gold, identity, omission, temporal, confidence, lineage, graph, or CIM contracts. Source file: `PRD-Cadastre.md`.

[^2]: Cadastre PRD, required product decisions before final NLSpec. The PRD identifies exact compiled OCSF schema artifact, compiler/validator toolchain, profile behavior verification, profile coverage, extension policy, and OCSF base-event field policy as unresolved. Source file: `PRD-Cadastre.md`.

[^3]: OCSF repository README. OCSF describes itself as an open standard for cybersecurity event logging and normalization, made of categories, event classes, data types, and an attribute dictionary, with repository structure for events, objects, profiles, extensions, metaschema, templates, categories, dictionary, and version files. URL: <https://github.com/ocsf/ocsf-schema>.

[^4]: OCSF `1.8.0` raw `events/base_event.json`. The base event includes required `activity_id`, `category_uid`, `class_uid`, `metadata`, `severity_id`, `time`, and `type_uid`, plus optional or recommended fields including `raw_data`, `raw_data_hash`, `unmapped`, `observables`, `enrichments`, message, status fields, severity, and timezone offset. URL: <https://raw.githubusercontent.com/ocsf/ocsf-schema/1.8.0/events/base_event.json>.

[^5]: OCSF `1.8.0` raw `dictionary.json`. The dictionary describes itself as the authoritative reference for standardized attributes and shows enum ID/name sibling conventions such as `activity_id`/`activity_name` and `Other` values directing source-specific values to sibling attributes. URL: <https://raw.githubusercontent.com/ocsf/ocsf-schema/1.8.0/dictionary.json>.

[^6]: OCSF validator README. The validator is needed because include/profile/extends mechanisms make individual files incomplete, and it checks JSON validity, directory structure, include/profile/extends targets, required attributes, unrecognized attributes, dictionary usage, and name collisions. URL: <https://github.com/ocsf/ocsf-validator>.

[^7]: OCSF contribution docs for extensions. Extensions reserve unique UID/name entries to avoid collisions and use extension directories with `extension.json` metadata. URL: <https://github.com/ocsf/ocsf-schema/blob/main/CONTRIBUTING.md>.

[^8]: OCSF GitHub release page. Release `1.8.0` is shown with tag `1.8.0`, commit `6fa6499`, and release date March 18, 2026. URL: <https://github.com/ocsf/ocsf-schema/releases>.

[^9]: OCSF schema compiler README. The compiler is a Python library/CLI for compiling OCSF schema source into a JSON-compatible compiled schema artifact. URL: <https://github.com/ocsf/ocsf-schema-compiler>.

[^10]: Cadastre PRD, `OCSFBaseEventFieldPolicy`. The PRD policy forbids or constrains OCSF `raw_data`, `raw_data_hash`, `unmapped`, `observables`, `enrichments`, status, severity, confidence, metadata, message, and timezone fields, and states that OCSF base-event fields must not replace Cadastre raw evidence, omission states, identity decisions, bitemporal facts, assertion states, graph projection, CIM loss accounting, source authority, or confidence policy. Source file: `PRD-Cadastre.md`.

[^11]: Elastic ECS docs. ECS defines field names, datatypes, descriptions, examples, field levels, naming guidelines, and custom-field guidance. URL: <https://www.elastic.co/docs/reference/ecs>.

[^12]: ECS field reference. ECS version `9.4.0` docs define field sets; Base is root and other field sets are objects. URL: <https://www.elastic.co/docs/reference/ecs/ecs-field-reference>.

[^13]: ECS generator/tooling docs and source. ECS tooling generates artifacts from schemas and custom field definitions, including CSV, Elasticsearch templates, Beats configs, and docs; generator source describes loading/validating YAML schemas and producing flat/nested schemas and artifacts. URL: <https://github.com/elastic/ecs/blob/main/USAGE.md>.

[^14]: ECS overview. ECS is described as permissive, and events with additional data that cannot be mapped to ECS may add custom field names. URL: <https://www.elastic.co/docs/reference/ecs>.

[^15]: ECS custom fields docs. ECS custom fields are additional fields defined by users or integrations, with flexibility by design and collision risk with future ECS fields. URL: <https://www.elastic.co/docs/reference/ecs/ecs-custom-fields-in-ecs>.

[^16]: ECS release/version context. Official field reference docs show ECS `9.4.0`; GitHub release search exposed `v9.4.0-rc1` and `v9.3.0`; GitHub main version file rendered as `9.5.0-dev`. URL: <https://www.elastic.co/docs/reference/ecs/ecs-field-reference>.

[^17]: Sigma main repository README. The rule repository is for detection engineers and threat hunters and contains more than 3000 rules. URL: <https://github.com/sigmahq/sigma>.

[^18]: Sigma specification release page. `SigmaHQ/sigma-specification` release `v2.1.0` is shown with commit `0c857d0`. URL: <https://github.com/SigmaHQ/sigma-specification/releases>.

[^19]: Sigma rules specification. The rules are YAML and include mandatory `title`, `logsource`, and `detection` plus optional metadata and arbitrary custom fields. URL: <https://raw.githubusercontent.com/SigmaHQ/sigma-specification/main/specification/sigma-rules-specification.md>.

[^20]: Sigma JSON schema. The schema requires `title`, `logsource`, and `detection`; it defines status values and level values. URL: <https://raw.githubusercontent.com/SigmaHQ/sigma-specification/main/json-schema/sigma-detection-rule-schema.json>.

[^21]: Sigma rules specification, taxonomy and status sections. Sigma taxonomy can define field names, field values, and logsource names; non-default taxonomies must be handled by tooling or transformed. URL: <https://raw.githubusercontent.com/SigmaHQ/sigma-specification/main/specification/sigma-rules-specification.md>.

[^22]: Sigma rules specification, field usage and special values. The spec says mapped field names should be used when mappings exist, raw source fields may be used otherwise, and empty/null values are backend-dependent. URL: <https://raw.githubusercontent.com/SigmaHQ/sigma-specification/main/specification/sigma-rules-specification.md>.

[^23]: MISP published standards. MISP core format describes JSON exchange of indicators and threat information; object template, taxonomy, galaxy, and SightingDB formats are separately described. URL: <https://www.misp-standard.org/standards/>.

[^24]: MISP release page and release notes. MISP `v2.5.37`, commit `be735cf`, was released April 29, 2026 and includes event templating, Suricata attribute type, STIX 2 stack changes, security fixes, and performance tooling. URL: <https://github.com/MISP/MISP/releases>.

[^25]: MISP core format draft. The core format describes MISP events as JSON objects, requires event UUID preservation, defines distribution values, and describes attributes as category-type-value triplets. URL: <https://www.misp-standard.org/rfc/misp-standard-core.html>.

[^26]: MISP object templates. The repository and object-template standard describe public object-template directories and template attributes such as MISP attribute types, categories, multiple values, disabled correlation, allowed values, defaults, and `to_ids`. URL: <https://github.com/misp/misp-objects>.

[^27]: MISP taxonomy and galaxy standards. Taxonomy format uses namespace directories, `machinetag.json`, manifests, predicates, and values; galaxy format defines galaxies/clusters and relationship fields. URL: <https://www.misp-standard.org/rfc/misp-standard-taxonomy-format.html>.

[^28]: MISP glossary. MISP uses a RESTful API internally and externally, and PyMISP is described as the de-facto standard client surface. URL: <https://misp.gitbooks.io/misp-book/content/GLOSSARY.html>.

[^29]: OpenLineage object model docs. The docs state that OpenLineage observes datasets through pipelines, defines Jobs and Datasets, and has Job Run State Updates, Job Metadata Updates, and Dataset Metadata Updates. URL: <https://openlineage.io/docs/spec/object-model/>.

[^30]: OpenLineage naming docs. Jobs and datasets use namespaces; Runs are client-generated UUIDs maintained throughout the run cycle. URL: <https://openlineage.io/docs/spec/naming/>.

[^31]: OpenLineage facet docs. Dataset schema facets contain dataset schema fields, and source-code location facets include source code location and version metadata. URL: <https://openlineage.io/docs/spec/facets/dataset-facets/schema/>.

[^32]: OpenLineage facets and extensibility docs. Custom facets require distinct prefixes, `_schemaURL`, immutable versioned schema URLs, and collision-avoidance naming patterns. URL: <https://openlineage.io/docs/spec/facets/>.

[^33]: OpenMetadata repository README. OpenMetadata connects technical metadata, data quality, lineage, ownership, usage, policies, glossaries, classifications, metrics, domains, and data products into a unified metadata knowledge graph with connectors, standards, APIs, SDKs, and search. URL: <https://github.com/open-metadata/openmetadata>.

[^34]: OpenMetadata release and Standards pages. Platform release `1.12.8` is shown with tag `1.12.8-release`, commit `5d66de8`, and date May 13, 2026; OpenMetadata Standards site exposes version `1.13.0`, April 2026. URLs: <https://github.com/open-metadata/OpenMetadata/releases>, <https://docs.open-metadata.org/>.

[^35]: OpenMetadata Standards docs. The Standards provide 700+ JSON schemas, REST and event schemas, RDF/OWL, SHACL, JSON-LD, and PROV-O materials for cataloging, governance, lineage, quality, and observability. URL: <https://docs.open-metadata.org/v1.12.x/api-reference/main-concepts/metadata-standard>.

[^36]: OpenMetadata lineage API docs. The lineage API manages lineage relationships between entities, including column-level mappings and pipeline references. URL: <https://docs.open-metadata.org/latest/how-to-guides/data-lineage>.
