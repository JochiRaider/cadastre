---
doc_id: domain
title: Cadastre Domain Vocabulary and Boundaries
doc_type: domain-vocabulary
status: candidate
---

## 1. Status and Authority

`domain.md` is the root Cadastre domain vocabulary and boundary document. It owns repository-wide domain interpretation, canonical term selection, forbidden substitutions, high-risk distinction tables, bounded-context routing, and the domain-level owner map for Cadastre concepts.

`domain.md` does not own runtime algorithms, data schemas, persistence behavior, graph backend behavior, API behavior, package activation behavior, validation harness behavior, implementation module structure, private source bindings, or external-system behavior. Those behaviors must remain owned by the active Cadastre owner specs.

`domain.md` must not add, widen, narrow, or replace runtime behavior. It may state domain interpretation rules and route implementers to the owner spec that owns behavior.

`domain.md` is listed by `docs/nlspec/000-cadastre-spec-index.md` as a `candidate_spec` with `candidate` status. It may route vocabulary and owner lookup during review, but it must not drive implementation until `000` promotes it to `authoritative`.

Cadastre source authority is governed by the supplied source-of-truth map and status vocabulary. The supplied index states that candidate Cadastre NLSpecs may not drive implementation, that documents marked `authoritative` may drive implementation for owned contracts, and that research reports are supporting evidence only, never runtime authority.

If `domain.md` conflicts with a primary owner spec, the owner spec governs the behavior it owns. The conflict must be recorded as an owner-spec `TODO:` or a validation failure in `120`. If two owner specs conflict, the contradiction is a corpus defect. `domain.md` must not silently choose a side.

The normative words `must`, `must not`, `may`, and `default` in this document govern vocabulary use, domain interpretation, owner lookup, review discipline, and coding-agent context only. This document does not use `should` as a runtime norm.

Research reports, examples, the uploaded template, implementation modules, candidate drafts, external repositories, backend products, package registries, UI labels, and storage artifacts are not Cadastre domain authority unless an active Cadastre owner spec explicitly grants that authority.

Content excluded from the uploaded template: Cartulary-specific product terms, workbook-native incident workspace framing, workbook surfaces, built-in tabs, saved views, parties, artifacts, notes, timeline rows, task requests, decisions, object blobs, import sessions, snapshots, releases, `view_schema_id`, `record_id`, `party_id`, and `cartulary.*` identifiers are not Cadastre requirements and are not imported into this document.

### Runtime exports

`domain.md` exports no runtime records, schemas, algorithms, interfaces, lifecycle machines, error codes, validation harness behavior, graph backend behavior, package activation behavior, or API behavior.

`domain.md` exports vocabulary terms, owner-routing tables, boundary distinctions, and review rules only.

## 2. Purpose

`domain.md` must provide the root domain context that lets implementers, reviewers, specification authors, and coding agents use Cadastre terminology consistently without turning implementation representations into domain truth.

This document must:

- define canonical Cadastre domain terms and forbidden substitutions;
- map domain concepts to primary owner specs or owner sections;
- define bounded-context relationship types, context-map edge routing, crossing-artifact vocabulary, and default no-effect behavior for unowned or invalid cross-context dependencies;
- distinguish domain concepts from implementation modules, storage artifacts, UI labels, graph backend labels, package names, route names, and external-system terms;
- preserve Cadastre authority boundaries and owner routing;
- provide stable context for implementers, reviewers, specification authors, and coding agents;
- provide binary acceptance criteria for the root domain document.

This document is not an API reference, schema reference, persistence spec, graph backend spec, UI design guide, implementation guide, operating handbook, or test plan. It may point to those artifacts when they own detailed behavior.

## 3. Scope and non-scope

### 3.1 Root domain document owns

| Domain area | Root `domain.md` ownership |
| --- | --- |
| Domain vocabulary | Canonical term, definition, forbidden interpretation, and owner lookup. |
| Domain concepts | Domain-level distinction between evidence, observations, facts, identities, projections, packages, analysis, and external concepts. |
| Core entities | Conceptual entity list, identity basis category, lifecycle basis category, relationship summary, and owner spec. |
| Entity relationships | Relationship families, domain meaning, evidence requirements at a summary level, and non-implication rules. |
| Lifecycle vocabulary | Domain-level state vocabulary routing and non-substitution rules. Transition tables remain owner-specific. |
| Identity and naming rules | Domain-level distinction among canonical identity, source-native identity, graph identity, storage identity, display labels, external schema identifiers, package identifiers, and private binding names. |
| Graph concepts | Domain boundary between graph concepts and authoritative facts when graph applicability is established by Cadastre owner specs. |
| Workflow vocabulary | Domain distinction between processing stages, lifecycle states, validation artifacts, analysis outputs, and user-facing states. |
| Spec-owner routing | The primary owner spec for each domain concept and boundary. |
| Context map | Bounded-context relationship type vocabulary, context-map edge routing, crossing-artifact owner lookup, prohibited dependency classes, and default root-domain no-effect interpretation. |
| Review discipline | Rules for agents and reviewers when terminology or ownership is unresolved. |
| Volatility vocabulary | Canonical names and owner routing for stable core contracts, activatable artifacts, runtime state records, reference evidence, rationale, and inactive future material. |

### 3.2 Root domain document does not own

| Area not owned by root `domain.md` | Owner artifact to consult |
| --- | --- |
| Product boundary, authority classes, and direct-source prohibition | `010-system-boundary-and-authority.md` |
| Lakehouse feed reads, raw imports, table-state refs, replay retention, and maintenance safety | `020-lakehouse-feeds-and-table-state.md` |
| Processing DAG execution, stage output permissions, lifecycle machines, run locks, and version manifests | `030-processing-dag-lifecycle-and-versioning.md` |
| Core record shapes, scalar rules, canonical serialization, omission states, raw/silver/gold primitive fields, and graph delta primitive shapes | `040-canonical-data-observation-and-fact-model.md` |
| Parser behavior, normalization, OCSF/CIM/external schema mapping, source extension fields, mapping compiler behavior | `050-normalization-external-schema-and-mapping.md` |
| Source authority, completeness, staleness, coverage, absence, control result mapping, and watermarks | `060-source-authority-completeness-coverage-and-absence.md` |
| Identity resolver behavior, target selector safety, review cases, identity splits, and graph correction handoff | `070-identity-resolution-and-target-selectors.md` |
| Temporal semantics, fact-time resolution, bitemporal query modes, gold derivation, corrections, late arrival, replay | `080-temporal-corrections-replay-and-gold-derivation.md` |
| Graph projection, graph deltas, edge semantics, traversal, backend profiles, graph apply, graph query, rebuild, drift | `090-graph-projection-serving-and-backends.md` |
| Package artifact identity, package release manifests, package-set activation, trust, compatibility, rollback, quarantine | `100-packages-supply-chain-and-activation.md` |
| API behavior, UX states, operational health, shared errors, redaction, authorization, audit | `110-api-ux-health-errors-and-security.md` |
| Fixtures, validation matrices, golden corpus, acceptance reports, validation acceptance | `120-validation-fixtures-and-acceptance.md` |
| Analysis rules, enrichment, lineage mapping, artifact-class boundaries, registry governance | `130-analysis-enrichment-and-registry-governance.md` |
| Observability instrumentation, telemetry signals, telemetry attributes, metric catalogs, trace context, exporter behavior, telemetry health mapping, and telemetry replay exclusion | `140-observability-instrumentation.md` |
| Future theoretical reachability candidate contracts | `200-future-reachability-analysis-domain.md`, inactive until promoted |
| Runtime effects of context-map edges, crossing-artifact schemas, handoff algorithms, failure codes, mutation behavior, query behavior, activation behavior, telemetry behavior, and validation harness execution | The owner spec named by the context-map edge and `120-validation-fixtures-and-acceptance.md` for validation evidence only. |
| Volatility activation algorithms, artifact schemas, package-set behavior, and version-manifest field schemas | `000`, `010`, `030`, `100`, `120`, and the domain owner specs named for the artifact class |
| Implementation code structure, package/module names, route names, database schemas, CSS classes, local config, private source binding artifacts | Implementation repository and private artifacts; not domain authority by default |

## 4. Domain overview

Cadastre is a lakehouse-fed interpretation, normalization, identity, fact, and projection system for temporal asset intelligence. It consumes externally supplied lakehouse-resident raw feeds, preserves raw evidence, normalizes observations into Cadastre-owned silver envelopes, resolves canonical identity through governed resolver profiles, derives bitemporal gold facts through source authority and temporal policies, and projects those facts into graph serving and optional external outputs.

Temporal, correction, late-arrival, replay, correction-snapshot, no-op/error, and graph-handoff behavior is closed by `080` and instantiated by activation-controlled rows. `domain.md` routes terms to owner specs and must not duplicate `080` algorithms, correction matrices, replay failure precedence, replay output-class rows, or graph handoff behavior.

The controlling domain interpretation rule is:

> Cadastre domain truth is the product-owned interpretation of lakehouse-resident evidence through active Cadastre contracts, not the implementation representation that happens to store, display, transport, project, or query that interpretation.

Consequences:

- A lakehouse table, graph backend node, CIM record, OCSF field, source-native ID, package artifact, route, UI label, validation fixture, or research-report term must not become the definition of a Cadastre domain term.
- The lakehouse is the system-of-record substrate for authoritative Cadastre records, but lakehouse product defaults, catalog names, snapshot timestamps, object-store commits, and table-maintenance defaults are not Cadastre domain authority unless an owner spec maps them.
- Graph, CIM, OCSF export, metadata, lineage, analysis, registry, and user-facing outputs are projections, analysis artifacts, or governance metadata unless an owner spec grants a narrower authority class.

## 5. Document relationship and ownership map

| Artifact or spec | Owns | `domain.md` role | Must not be used for |
| --- | --- | --- | --- |
| `domain.md` | Root vocabulary, domain boundaries, owner lookup, high-risk distinctions, reviewer/coding-agent domain rules. | Defines canonical language and routes behavior to owner specs. | Runtime algorithms, schemas, persistence, APIs, graph backend selection, package activation, validation harnesses. |
| `000-cadastre-spec-index.md` | Documentation governance, source-of-truth map, status vocabulary, dependency policy, promotion gates, archival rules. | Must follow its status and owner map; must surface unresolved root-domain map gaps. | Product runtime behavior or domain record fields. |
| `010-system-boundary-and-authority.md` | Cadastre product boundary, authority classes, forbidden production inputs, projection authority, public/private boundary. | Imports authority vocabulary and global non-authority rules. | Lakehouse table fields, graph algorithms, identity decisions. |
| `020-lakehouse-feeds-and-table-state.md` | Lakehouse feeds, raw imports, read policies, completeness receipts, table-state refs, maintenance and retention. | Uses its terms for lakehouse-fed input and table-state domain boundaries. | Source absence interpretation, gold truth, identity resolution, graph apply. |
| `030-processing-dag-lifecycle-and-versioning.md` | Processing stage DAG, output permissions, lifecycle machines, run locks, version manifests, stage state. | Uses its lifecycle and stage vocabulary for domain routing. | Parser/gold/identity/graph/package-specific behavior. |
| `040-canonical-data-observation-and-fact-model.md` | Core record shapes, canonical serialization, omission states, raw/silver/core entity/gold/evidence primitives. | Uses its record names as canonical domain vocabulary. | Source authority algorithms, resolver algorithms, external mapping rows, graph projection algorithms. |
| `050-normalization-external-schema-and-mapping.md` | Parser, normalization, OCSF/CIM/external schema profiles, source extension fields, mapping compiler pipeline. | Routes external schema and mapping vocabulary. `050` owns OCSF mapping interfaces and algorithms; activation catalogs and validation fixtures instantiate them. | Gold authority, identity merge decisions, graph projection behavior. |
| `060-source-authority-completeness-coverage-and-absence.md` | Source authority, completeness, staleness, coverage, absence derivation, progress-signal interpretation, watermarks. | Routes evidence/absence/coverage vocabulary and non-implication rules. | Resolver merge rules or graph apply behavior. |
| `070-identity-resolution-and-target-selectors.md` | Resolver profiles, identity evidence, identity decisions, review, split behavior, target selector safety. | Routes identity and naming vocabulary. | Source authority for non-identity facts, direct graph mutation. |
| `080-temporal-corrections-replay-and-gold-derivation.md` | Temporal semantics, gold derivation, correction, late arrivals, replay equivalence, deterministic side effects. | Routes bitemporal and correction vocabulary. | Backend graph apply or external schema mappings. |
| `090-graph-projection-serving-and-backends.md` | Graph projection, graph delta identity, graph edge semantics, traversal, backend profiles, apply, query, rebuild, drift. | Defines graph-domain boundary and non-authority rules at root level. | Fact existence, identity resolution, source completeness, backend product selection. |
| `100-packages-supply-chain-and-activation.md` | Package artifacts, release manifests, package-set activation, trust, compatibility, rollback, quarantine, emergency behavior. | Routes package and activation vocabulary. | Package business behavior outside activation and execution permissions. |
| `110-api-ux-health-errors-and-security.md` | Public APIs, UX state labels, health, shared errors, redaction, authorization, audit. | Routes user-facing label and error vocabulary without redefining domain behavior. | Domain-specific behavior owned elsewhere. |
| `120-validation-fixtures-and-acceptance.md` | Fixtures, validation matrices, golden corpus, shadow execution, replay validation, acceptance reports. | Uses validation terms for Definition of Done and acceptance criteria. | New behavioral requirements not owned by domain specs. |
| `130-analysis-enrichment-and-registry-governance.md` | Analysis rules, findings, metrics, enrichment, lineage facet mapping, artifact-class policy, registry governance. | Routes non-authoritative analysis/enrichment/registry vocabulary. | Fact, identity, graph, completeness, package, or watermark authority. |
| `140-observability-instrumentation.md` | Runtime telemetry contracts, signal policy, trace context, metric catalog, telemetry attributes, telemetry redaction, exporter behavior, telemetry runtime state, telemetry health mapping, telemetry replay exclusion, and telemetry non-authority. | Routes runtime observability vocabulary and boundary distinctions. | Domain truth, source authority, identity, graph projection, package activation, API response schema, validation harness behavior. |
| `200-future-reachability-analysis-domain.md` | Inactive future reachability candidate contracts. | Labels reachability as deferred and blocks active MVP reachability claims. | Active MVP behavior, solver selection, negative reachability facts, graph reachability edges. |
| `nlspec-spec.md` | Specification-quality standard: behavioral completeness, unambiguous interfaces, explicit defaults, mapping tables, binary acceptance criteria. | Governs writing quality and acceptance posture. | Cadastre product behavior. |
| Uploaded `domain.md` template | Structure, voice, and pattern examples only. | Provides non-authoritative structural pattern for root-domain writing. | Any Cadastre terminology, workflow, identifiers, entity model, owner model, lifecycle, acceptance criteria, or product thesis. |
| `RES-001-cartography.md` | Research report on Cartography. | Supporting evidence for graph-source boundary risks only when adopted by owner specs. | Runtime authority or Cadastre graph model. |
| `RES-002-jupiterone.md` | Research report on JupiterOne and Starbase. | Supporting evidence for graph-first and identity-key hazards only when adopted by owner specs. | Runtime authority, identity authority, graph authority. |
| `RES-003-taxi-lang.md` | Research report on Taxi semantic overlays and tooling. | Supporting evidence for semantic-overlay and tooling boundaries only when adopted by owner specs. | Cadastre canonical semantics, runtime dependency, identity, graph, or gold authority. |
| `RES-004-ocsf-schema.md` | Research report on OCSF schema. | Supporting evidence for OCSF as external schema profile only when adopted by mapping spec. | Cadastre truth layer, identity resolver, graph model, bitemporal model. |
| `RES-005-Lakehouse-Derived-Graph-Architecture.md` | Research report on lakehouse and derived graph architecture. | Supporting evidence for lakehouse-as-system-of-record and graph projection boundaries. | Selecting a lakehouse product or graph backend, runtime authority. |
| `RES-006-source-adapter-ingestion-patterns.md` | Research report on source adapter and ingestion patterns. | Supporting evidence for adapter state/completeness patterns after lakehouse-fed translation. | Direct enterprise-source collection authority. |
| `RES-007-normalization-and-schema-standards.md` | Research report on normalization standards. | Supporting evidence for external schema, mapping, analysis, enrichment, lineage, registry boundaries. | External schema authority over Cadastre identity, facts, graph, completeness. |
| `RES-008-identity-graph-attack-paths.md` | Research report on identity graphs and attack paths. | Supporting evidence for graph traversal, OpenGraph, selector, and attack-path non-authority. | Cadastre identity, risk, remediation, source completeness authority. |
| `RES-009-identity-resolution-governance.md` | Research report on identity resolver governance. | Supporting evidence for resolver profiles, evidence classes, hard blockers. | Direct production resolver authority unless owner spec adopts it. |
| `RES-010-theoretical-reachability-network-context.md` | Research report on future reachability. | Supporting evidence for keeping theoretical reachability inactive in MVP. | Active reachability facts or graph edges. |
| `RES-011-temporal-semantics.md` | Research report on temporal semantics and replay. | Supporting evidence for temporal/gold/replay boundary refinements. | Backend or external system authority for Cadastre time. |
| `RES-012-graph-backend-comparison.md` | Research report on graph backends. | Supporting evidence for backend-neutral graph-serving boundaries. | Selecting production graph backend or backend-specific domain truth. |
| `RES-013-extension-package-supply-chain.md` | Research report on package supply chain. | Supporting evidence for package-set activation and trust refinements. | Registry/product selection or activation authority by itself. |
| `RES-014-source-authority-staleness-coverage-absence.md` | Research report on authority, staleness, coverage, absence. | Supporting evidence for source authority and absence boundary refinements. | Source truth unless adopted by `060`. |
| `RES-015-janusgraph.md` | Research report on JanusGraph. | Supporting evidence for provider-specific backend risk analysis and provider-neutral graph backend contracts when adopted by owner specs. | Runtime authority, graph model authority, identity authority, source authority, package activation authority, backend selection authority, or future-provider exclusion. |
| Implementation repository files, when present | Code, package structure, local module names, tests, comments. | May map implementation objects to domain terms after exact inspection. | Defining Cadastre domain concepts by package/module/table/route names. |

## 6. Domain inclusion rule

A term belongs in root `domain.md` when misunderstanding it would cause an implementer, reviewer, specification author, or coding agent to build the wrong behavior, address the wrong owner contract, mutate the wrong source of truth, collapse distinct concepts, or write misleading specification text.

A term belongs outside root `domain.md` when it is only an implementation package, module, function, table, index, route, component, CSS class, object-store key, deployment path, test-harness detail, or product-specific backend mechanism.

An implementation detail may appear only as an ambiguity-preventing mapping. It must not become the domain definition.

| Term or concern | Belongs in root `domain.md` | Belongs outside root `domain.md` | Rationale-free rule |
| --- | --- | --- | --- |
| `GoldFact` | Yes. It is a core Cadastre fact concept with source authority, temporal, evidence, and graph implications. | Field-level schema and correction algorithm. | Define term and owner; route behavior to `040` and `080`. |
| Graph backend product name | No, unless used to state forbidden substitution or owner mapping. | Backend adapter implementation, driver config, product selection. | Graph backend is not domain truth. |
| `GraphEdgeSemantics` | Yes. Misuse creates false graph relationships. | Full edge-type rows and projection implementation. | Define domain boundary; route detailed rows to `090`. |
| SQL table name | No by default. | Persistence implementation. | A table name is not a domain concept unless an owner spec makes it contract-visible. |
| `LakehouseReadCompletenessReceipt` | Yes. Misuse creates false absence claims. | Receipt schema fields and read algorithm. | Define non-implication and owner. |
| `OCSF` | Yes as an external schema boundary. | Full OCSF schema, class mapping rows, compiler details. | External schema terms must not become Cadastre truth. |
| UI label | No by default. | UI rendering and localization. | User-facing labels must map to stable domain terms owned by `110`. |
| `ResolverProfile` | Yes. It is the identity authority boundary. | Resolver decision matrix implementation details. | Define as identity owner, not a generic score. |
| Package name | No by default. | Package registry and build system. | Package names do not define package activation authority. |
| `ProductionPackageSetManifest` | Yes. It defines the activation unit boundary. | Trust verification internals and artifact acquisition. | Define activation target; route behavior to `100`. |
| Future reachability terms | Yes only as inactive/deferred boundary language. | Active MVP facts, graph edges, solver behavior. | Label inactive and route to `200`. |

## 7. Resolved terminology decisions

| Issue | Canonical Cadastre interpretation | Forbidden interpretation | Primary owner | Source evidence | Status |
| --- | --- | --- | --- | --- | --- |
| Cadastre product boundary | Cadastre consumes lakehouse-resident feeds, interprets, normalizes, resolves, derives, and projects. | Cadastre as default enterprise source collector, scanner, syslog receiver, API poller, or CDC connector. | `010`, `020` | `010` Boundary Contract | Resolved. |
| System of record versus projection | Authoritative Cadastre records live in the lakehouse-backed record flow. Graph, CIM, OCSF export, metadata, lineage, and analysis outputs are projections or non-authoritative artifacts unless owner spec grants authority. | Graph backend, CIM output, OCSF export, metadata graph, lineage graph, or analysis output as source of truth. | `010`, `040`, `090`, `130` | `010` Authority Classes; `090` Projection Authority | Resolved. |
| Raw evidence versus source completeness | Raw evidence and feed read receipts can support completeness decisions only through active profiles and active feed-category closure rows. | Missing rows, missing profile rows, successful feed reads, or private completeness capabilities as source absence by themselves. | `020`, `060` | `020` Completeness Receipt Contract and `LakehouseFeedCategoryClosureRow`; `060` Completeness Contract and `LakehouseFeedCompletenessProfileRow` | Concept resolved; owner closure rows still gate activation. |
| Raw observation versus normalized observation | `RawRecord` preserves source-native evidence; `CadastreSilverObservation` is a normalized Cadastre silver envelope. | Parser output or external schema fields as canonical truth. | `040`, `050` | `040` Core Records; `050` Parser and Silver Normalization Contracts | Resolved. |
| Silver observation versus gold fact | Silver observations preserve normalized evidence and Cadastre-owned metadata; `GoldFact` is a canonical bitemporal assertion derived through authority and temporal policies. | Treating every silver observation as a canonical fact. | `040`, `060`, `080` | `040` Core Records; `060` Source Authority Contract; `080` Gold Derivation | Resolved. |
| Canonical entity versus source asset | `CanonicalEntity` is product-owned identity. `SourceAsset` is source/feed-scoped asset identity. | Source-native ID, provider key, graph key, or display label as canonical identity. | `040`, `070` | `040` Core Records; `070` Resolver Authority | Resolved. |
| Identifier versus identity decision | `Identifier` records typed observed identifiers; `IdentityDecision` is the governed resolver output. | Identifier equality as automatic canonical identity. | `040`, `070` | `040` Core Records; `070` Evidence Roles | Resolved. |
| Target hint versus identity | `UnresolvedTargetReference` is a relationship hint and must not create or merge canonical entities. | Mapped target, OpenGraph-style property matching, name matching, or graph key as identity. | `070` | `070` Target Selector Safety | Resolved. |
| Source authority versus source completeness | Source completeness is coverage or read completeness evidence; source authority is permission to derive or assert a fact. | Complete source scope as automatic authority for every fact type or predicate. | `060` | `060` Source Authority Contract and Completeness Contract | Resolved. |
| Absence versus unknown | Absence may be emitted only through active absence authority, completeness, coverage, staleness, and allowed-effect gates. Unknown preserves lack of authority, missing closure rows, or insufficient evidence. | Missing value, missing row, TTL expiry, lease expiry, partial feed, permission limit, or `not_authoritative_for_absence` as absence. | `060`, `040`, `110` | `060` DeriveAbsenceOrUnknown; `040` Omission Semantics; `110` OwnerStateToSourceStateLabelMapping | Concept resolved; owner closure rows still gate activation. |
| Graph edge versus domain relationship | A graph edge is a projected read-model object governed by `GraphEdgeSemantics`. A domain relationship is the underlying meaning supported by facts and evidence. | Backend edge existence as source fact, identity proof, reachability proof, risk proof, or completeness proof. | `090`, `040`, `080` | `090` Edge Semantics and Traversal | Resolved. |
| Graph traversal versus reachability | `GraphTraversalClass` controls path eligibility. Theoretical reachability is inactive and must not be emitted in MVP. | Any graph path as network reachability or service access. | `090`, `200` | `090` Definition of Done; `200` Prohibited MVP Outputs | Resolved for MVP. |
| External schema term versus Cadastre term | OCSF and CIM terms are external schema/projection vocabulary mapped through profiles. | OCSF/CIM fields as Cadastre identity, authority, completeness, fact, or graph authority. | `050`, `040`, `060` | `050` External Schema Profile and CIM Projection Contract | Resolved. |
| Analysis finding versus fact | `AnalysisFinding`, `AnalysisMetric`, and `RiskAcceptanceRecord` are analysis or workflow outputs. | Analysis output as remediation, risk reduction, fact retraction, graph edge removal, or source completeness. | `130` | `130` Analysis Rules and Non-Authority Rule | Resolved. |
| Package artifact versus package activation | A package artifact is immutable release material; production activation occurs only through a `ProductionPackageSetManifest`. | Single artifact, version string, signature status, SBOM, provenance, dependency lock, or validation run as activation authority. | `100` | `100` Activation Unit and Package Set Manifest | Resolved. |
| `structured_input_repository` | A Git-backed authoring and provenance surface for high-change structured inputs. Production behavior may change only through owner validation, immutable materialization, package release creation when required, package-set activation when required, and `030.VersionManifest` inclusion. | Git branch, tag, pull request, hook result, repository URL, path name, commit timestamp, or repository working tree as production authority. | `030`, `100`, `110`, `120` | `030.StructuredInputRepositoryProfile`; `030.StructuredInputRepositorySnapshot`; `100.StructuredInputMaterializationResult`; `110.StructuredInputRepositoryAccessPolicy`; `110.StructuredInputRepositoryRedactionPolicy`; `120.StructuredInputRepositoryValidationMatrix` | `blocked_validation` until owner rows and fixtures pass. |
| `structured_input_repository_profile` | Owner-routed activation-controlled repository governance row set declaring allowed origins, refs, path roots, artifact classes, branch policy, merge policy, and public/private classification. | Repository URL, organization name, branch name, tag, path prefix, PR label, or hook result as the profile. | `030`, `110`, `120` | `030.StructuredInputRepositoryProfile`; `110.StructuredInputRepositoryAccessPolicy`; `120-STRUCTURED-INPUT-*` rows | `blocked_validation` until validation rows exist. |
| Scope selector terms | `ScopeSelector`, `ActivationScope`, `ScopeSelectorContext`, scope coverage, scope specificity, and scope ambiguity are runtime selector terms owned by `030`; owner specs own only their context rows and non-scope predicates. | Private binding as public scope, row order as ambiguity tiebreaker, graph selector as identity, dataset-level row as universal wildcard, telemetry scope as domain authority, or package environment string as an unnormalized activation selector. | `030`, owner specs | `030.ScopeSelector`; `030.ActivationScope`; owner `ScopeSelectorContext` rows | Resolved as vocabulary routing only. |
| `structured_input_repository_snapshot` | Exact runtime state record over one resolved commit, tree hash, parent commits, selected paths, normalized file manifest, file checksums, and validation refs. | Branch tip, tag, PR number, repository URL, commit timestamp, merge event, or hook log as equivalent snapshot authority. | `030`, `040`, `120` | `030.StructuredInputRepositorySnapshot`; `040.EvidenceArtifactClassRegistry`; `120.StructuredInputRepositoryValidationMatrix` | `blocked_validation` until deterministic snapshot fixtures pass. |
| `structured_input_change_proposal` | Reviewable proposed repository change whose lifecycle state is authoring and validation evidence only. It must not activate production behavior. | Pull request approval, merge approval, branch update, hook success, or reviewer comment as production package-set activation. | `030`, `110`, `120` | `030.StructuredInputChangeProposal`; `110.StructuredInputChangeProposalRequest`; `120-STRUCTURED-INPUT-*` rows | `blocked_validation` until lifecycle and API fixtures pass. |
| `structured_input_materialization` | Deterministic owner-routed conversion from a validated exact repository snapshot into immutable package artifact bytes or activation artifact bytes. | Git tree, branch, tag, repository path, hook result, validation report, or package name as activated release material. | `100`, `030`, `110`, `120` | `100.StructuredInputMaterializationResult`; `030.VersionManifest`; `110.RedactionDataClassMatrix`; `120.StructuredInputRepositoryValidationMatrix` | `blocked_validation` until materialization and package-set fixtures pass. |

| Graph backend selection default | Any named graph backend selection default may exist only as an owner-routed `090.GraphBackendSelectionPolicy` result with required `090`, `100`, and `120` gates. | Any backend product as Cadastre graph semantics, fact authority, identity authority, source completeness authority, activation authority, or future-provider exclusion. | `090`, `100`, `120` | `090.GraphBackendSelectionPolicy`; `090.GraphBackendProfile`; `100.ProductionPackageSetManifest`; `120` graph backend validation rows | Resolved as owner-routed selection; activation remains validation-gated. |
| Lifecycle status versus lifecycle machine | `LifecycleStatus` names shared states; runtime lifecycle behavior comes from a declared machine or pure deterministic algorithm authority. | Inferring transitions from diagrams, process status, logs, caches, or names. | `030` | `030` Lifecycle Contract | Resolved. |
| `docs/nlspec/domain.md` authority | Root domain document authority is `candidate` until `000` marks it `authoritative`. | Treating this candidate document as implementation authority without registry status. | `000` | `000` Document Registry. | Resolved. |
| Runtime telemetry versus domain truth | Runtime telemetry is operational diagnostic material emitted through `140`; it may support health and troubleshooting only through owner policies. | Trace, span, metric, structured log, exporter success, Collector state, or dashboard state as fact authority, identity evidence, source completeness, graph truth, audit ledger, or replay input. | `010`, `140`, `110`, `120` | `010.RuntimeTelemetryAuthorityRule`; `140.TelemetryNonAuthorityRule`; `110.TelemetryHealthMappingHandoff`; `120-OBSERVABILITY-*` rows | Resolved; validation-gated by `120-OBSERVABILITY-*`. |
| Telemetry correlation versus audit | `diagnostic_correlation_ref` is an opaque support ref when exposed; `AuditEvent` remains the audit record. | Trace ID or span ID as the audit event, evidence ref, or replay identity. | `110`, `140` | `110.CommonApiResponseEnvelope`; `110.AuditEventSchema`; `140.TelemetryCorrelationPolicy` | Resolved; validation-gated by `120-API-SCHEMA-TOTAL-*`, `120-AUTH-REDACTION-*`, and `120-OBSERVABILITY-*`. |
| Metric zero versus absence | A metric value of zero is runtime telemetry and does not prove source absence. | Zero rows, zero events, no spans, no logs, or no errors as absence, completeness, cleanup permission, graph expiry, or watermark authority. | `060`, `140` | `060.ProgressSignalInterpretationPolicy`; `140.TelemetryNonAuthorityRule` | Resolved; validation-gated by `120-SOURCE-CLOSURE-*` and `120-OBSERVABILITY-*`. |
| Telemetry resource identity versus canonical identity | Telemetry resource attributes identify runtime telemetry resources only unless an active resolver profile maps separate source evidence. | OTel resource attributes, service names, host names, process IDs, container IDs, or backend IDs as canonical entity identity. | `070`, `140` | `070.TelemetryIdentityNonAuthorityHandoff`; `140.TelemetryAttributePolicy` | Resolved; validation-gated by `120-IDENTITY-CLOSURE-*` and `120-OBSERVABILITY-*`. |

## 8. Core distinctions

| Distinction | Required interpretation | Common failure | Primary owner |
| --- | --- | --- | --- |
| Source evidence and authoritative facts | Source evidence supports decisions only through active Cadastre contracts; authoritative facts are Cadastre outputs with authority, temporal, evidence, and manifest refs. | Treating source rows, scanner results, collector output, or external schema fields as facts. | `040`, `060`, `080` |
| Raw observations and normalized observations | Raw observations preserve source-native payload and lineage. Normalized observations are Cadastre silver envelopes created by parser/mapping behavior. | Skipping raw preservation or asserting truth during parsing. | `040`, `050` |
| Normalized observations and canonical facts | Silver observations are evidence-bearing observations. Gold facts are canonical bitemporal assertions. | Treating a valid OCSF-normalized silver event as a gold fact. | `040`, `060`, `080` |
| Canonical entities and source-native assets | Canonical entities are product-owned identity outputs. Source assets are source/feed-scoped assets. | Using provider ID, hostname, IP, graph key, or source-native key as canonical entity ID. | `040`, `070` |
| Identity evidence and identity decisions | Evidence items are resolver inputs. Identity decisions are governed resolver outputs. | Auto-merging from strong-looking but unprofiled identifiers. | `070` |
| Graph read model and system of record | Graph state is a rebuildable projection. Authoritative records remain in Cadastre lakehouse-backed records. | Repairing truth from graph state or treating graph drift as fact drift. | `010`, `090` |
| Graph edge and source fact | A graph edge is a projected object with edge semantics. A source fact or gold fact is a product assertion. | Treating edge existence as source evidence, identity proof, or reachability proof. | `090`, `080` |
| Lifecycle state and workflow/UI state | Lifecycle state controls production eligibility through owner machines; UI state labels communicate observable conditions. | Treating user-facing labels as transition authority. | `030`, `110` |
| Source completeness and source authority | Completeness indicates scope/evidence sufficiency. Authority grants fact/predicate permission. | Treating complete inventory as authority for all predicates. | `060` |
| Absence and unknown | Absence requires active authority and sufficient completeness/coverage. Unknown is the default when evidence cannot authorize a claim. | Rendering unknown as negative fact or compliance pass/fail. | `060`, `110` |
| Implementation module and domain concept | Module names may implement or map domain terms; they must not define them. | Letting package names, table names, route names, or functions redefine domain terms. | `docs/nlspec/domain.md`, implementation repository after exact inspection |
| External schema term and Cadastre term | External terms map through profiles. Cadastre terms remain product-owned. | Letting OCSF/CIM/ECS/Sigma/MISP/OpenLineage/OpenMetadata labels become Cadastre identity, fact, graph, or authority. | `050`, `130`, `060`, `090` |
| OCSF mapping row family and active mapping row catalog | Stable specs may define required row families and row interfaces. Concrete mapping rows, enum rules, profile-resolution manifests, base-event policies, and source-extension rules affect production only through activation-controlled artifact refs. | Treating example rows, research text, or incomplete row-family matrices as active production mappings. | `000`, `030`, `050`, `100`, `120` |
| Source freshness and derived-view lag | Source stale state belongs to source staleness policy. Derived-view lag belongs to graph/API state. | Collapsing stale source evidence and stale graph view into one status. | `060`, `090`, `110` |
| Structured input repository and package repository | A structured-input repository is an authoring and provenance surface. A package repository is an acquisition surface for immutable release material. | Treating a Git authoring repository or repository URL as a package release, package repository metadata snapshot, or production activation target. | `030`, `100`, `120` |
| Git commit/tree snapshot and package artifact digest | A repository snapshot records exact authored content. A package artifact digest records immutable materialized release bytes. | Treating a tree hash, branch tip, or commit timestamp as the package artifact digest or release checksum. | `030`, `100`, `040` |
| Pull request approval and production package-set activation | Review approval may authorize a change proposal to merge or revalidate. Production behavior changes only through `100.ProductionPackageSetManifest` when package-supplied artifacts affect output. | Treating PR approval, merge approval, hook success, or CI success as production activation. | `030`, `100`, `110`, `120` |
| Merge to protected branch and active `ProductionPackageSetManifest` | A protected-branch merge is provenance. The active package set is the production activation target. | Treating protected branch state, tag movement, or repository default branch as current production state. | `030`, `100`, `120` |

| Package verification and package activation | Verification evidence supports activation decisions. Activation is package-set governed. | Treating valid signature, SBOM, provenance, or dependency lock as activation. | `100` |
| Future reachability and MVP graph behavior | Future reachability is inactive and graph-neutral by default. MVP graph must not emit theoretical reachability. | Emitting `has_theoretical_reachability`, boolean reachability, or unqualified reachable wording. | `090`, `200` |
| Runtime telemetry versus audit | Telemetry records runtime diagnostics; audit records authorized operation evidence. | Treating exporter delivery as audit persistence. | `110`, `140` |
| Runtime telemetry versus replay | Telemetry runtime IDs and sampling/exporter state are excluded from domain replay by default. | Including trace IDs, span IDs, or runtime duration in replay equivalence. | `080`, `140` |
| Runtime telemetry versus graph health | Graph metrics may report backend or derived-view state, but cannot repair graph drift or authorize graph mutation. | Treating graph telemetry as graph truth. | `090`, `140` |
| GoldFact subject ref versus identity evidence | `GoldFact.subject_ref` is a core reference to an eligible canonical entity, source asset, or identifier. | Treating raw identity evidence, selector hints, graph keys, source-native strings, or unresolved targets as `GoldFact.subject_ref`. | `040`, `070`, `080` |
| GoldFact object value versus external schema object | `GoldFact.object_value` is a typed object value governed by `040` and permitted by `080`. | Treating OCSF objects, observables, enrichments, unmapped fields, or external taxonomies as object authority by themselves. | `040`, `050`, `060`, `080` |
| EvidenceRef artifact ref versus raw payload | `EvidenceRef.artifact_id` is a checksummed reference, not payload. | Inlining raw payload bytes, package bytes, SBOM bytes, provenance bytes, or object-store path strings as evidence artifacts. | `040`, `020`, `030`, `100`, `130` |
| EvidenceRef artifact class versus activation artifact class | `EvidenceRef.artifact_class` selects evidence citation rules; activation artifact classes govern activatable rows and package-supplied artifacts. | Treating activation eligibility as evidence authority or adding evidence artifact ID kinds through owner artifacts. | `040`, `030`, `100`, `130` |
| Null object value versus omitted object value versus absence outcome | `null_value`, field omission, and `GoldFact.absence_outcome` are distinct concepts owned by different contracts. | Rendering omitted object value as null, null as absence, or absence outcome as a missing object. | `040`, `060`, `080`, `110` |

### 8.1 Volatility class versus authority class

`volatility_class` is not the same concept as `AuthorityClass`.

`AuthorityClass` answers whether an output may determine product truth. `volatility_class` answers whether material is stable core contract text, separately activatable row/profile/package/fixture material, runtime state, reference evidence, rationale, or inactive future material.

`domain.md` may name volatility terms and route them to owners. It must not define activation algorithms, artifact field schemas, package-set behavior, or version-manifest behavior.

| Volatility class | Required interpretation | Forbidden interpretation | Primary owner route | Runtime authority effect | Status |
| --- | --- | --- | --- | --- | --- |
| `stable_core_contract` | Long-lived owner NLSpec text that defines product boundaries, interfaces, defaults, algorithms, errors, and acceptance criteria. | Treating mutable row instances or package payloads as stable contract definitions. | `000` for governance and `010` for product authority boundary. | May define runtime behavior only for contracts exported by the authoritative owner spec. | canonical |
| `activation_controlled_artifact` | Versioned row set, profile, package release, fixture, mapping table, or registry artifact that instantiates exported behavior. | Defining new authority classes, core fields, identity rules, temporal axes, graph semantics, API states, or trust semantics. | `030` for manifest refs, `100` for package-supplied activation, and the domain owner spec for behavior. | No authority by itself; output effect exists only through the owner stable contract. | canonical |
| `runtime_state_record` | Persisted execution, replay, commit, snapshot, lifecycle, health, validation, or derived-view state. | Source truth, production activation, or product authority by mere existence. | Domain owner and `030.VersionManifest`. | No authority by itself; may be replay or audit evidence when owner permits. | canonical |
| `reference_evidence` | Research, standards, reports, examples, and evidence used to support owner-spec decisions. | Runtime authority, profile activation, or behavior definition. | `000` as non-runtime evidence. | None. | canonical |
| `rationale` | ADR or decision explanation that records why a choice was made. | Field shape, algorithm, default, error, or acceptance criterion. | `000` as non-runtime ADR rationale. | None. | canonical |
| `inactive_future_domain` | Deferred candidate material with no MVP production effect. | Active behavior, production output, solver selection, or graph/gold authority. | `000` and `200`. | None while deferred. | canonical |

| Volatility class | Owner-routing rule |
| --- | --- |
| `stable_core_contract` | Route to `000` for governance and `010` for product authority boundary. |
| `activation_controlled_artifact` | Route to `030` for manifest refs, `100` for package-set activation when package-supplied, and the domain owner spec for behavior. |
| `runtime_state_record` | Route to the domain owner and `030.VersionManifest`. |
| `reference_evidence` | Route to `000` as non-runtime evidence. |
| `rationale` | Route to `000` as non-runtime ADR rationale. |
| `inactive_future_domain` | Route to `000` and `200`. |

### 8.2 Run-lock signal distinction

`domain.md` routes run-lock behavior to owner specs only. It must not duplicate lock TTL values, algorithm steps, schema fields, or error-code tables.

| Distinction | Required domain meaning | Forbidden substitution | Owner |
| --- | --- | --- | --- |
| Run-lock heartbeat versus source heartbeat | Run-lock heartbeat is orchestration ownership evidence. | Source completeness, source liveness, no-change proof, absence, cleanup, graph expiry, or watermark authority. | `030`, `060`, `140` |
| Run-lock lease lifecycle and stale recovery | Lock ownership behavior is specified by `030` and remains validation-blocked until `120-RUNLOCK-*` rows pass. | Root-domain timing defaults, lock-store algorithm ownership, source authority, graph authority, or package activation authority. | `030`, `120` |

## 9. Bounded contexts

| Bounded context | Owns domain language for | Must not own | Primary owner |
| --- | --- | --- | --- |
| Documentation governance | Spec status, source-of-truth rows, dependencies, promotion state, archival policy. | Product runtime behavior. | `000` |
| System boundary and authority | Product boundary, authority classes, projection authority, public/private source binding boundary. | Data schemas, identity algorithms, graph algorithms. | `010` |
| Lakehouse feed and table state | Feed profiles, raw feed manifests, read policies, completeness receipts, table snapshot/commit/dataset refs, maintenance safety. | Source absence authority, identity, graph projection. | `020` |
| Processing and versioning | Stage classes, permitted outputs, lifecycle status, lifecycle machines, run locks, version manifests, stage state. | Domain-specific semantics for parser/identity/gold/graph/package. | `030` |
| Core data model | Raw records, silver observations, canonical entities, source assets, identifiers, gold facts, evidence refs, omission states, canonical serialization. | Source authority, resolver behavior, graph projection algorithms. | `040` |
| Normalization and external schema | Parser outputs, silver mapping, OCSF/CIM/external schema profiles, source extension fields, mapping validation. | Identity merge, gold derivation, graph apply. | `050` |
| Source authority and absence | Source authority profiles, completeness profiles, staleness policies, coverage assertions, absence derivation, watermarks. | Resolver merge or graph apply. | `060` |
| Identity resolution | Resolver profiles, identity evidence, identity decisions, review cases, target selector safety, split handoff. | Source authority for non-identity facts or direct graph mutation. | `070` |
| Temporal and gold facts | Fact-time resolution, bitemporal modes, gold derivation, correction, late arrival, replay equivalence. | Backend graph apply, external schema mapping. | `080` |
| Graph projection and serving | Graph projection profiles, graph deltas, graph edge semantics, traversal, graph backend profiles, query, apply, rebuild, drift. | Fact existence, identity decisions, source completeness. | `090` |
| Package activation | Package artifacts, release manifests, package-set activation, trust, compatibility, rollback, quarantine, emergency behavior. | Business behavior outside activation and execution permissions. | `100` |
| API, UX, health, error, security | API request/response contracts, state labels, health, errors, redaction, authorization, audit. | Domain-specific behavior owned by source, identity, temporal, graph, or package specs. | `110` |
| Validation and acceptance | Validation matrices, fixtures, golden corpus, acceptance reports, validation acceptance. | New behavioral requirements not owned by a domain spec. | `120` |
| Analysis, enrichment, lineage, registry | Read-only analysis, findings, metrics, enrichment records, lineage mapping, artifact-class policy, registry governance. | Fact, identity, graph, completeness, package, or watermark authority. | `130` |
| Observability and runtime telemetry | Runtime telemetry signals, trace context, metric instruments, telemetry attributes, telemetry redaction, exporter behavior, telemetry runtime state, telemetry health mapping, and telemetry replay exclusion. | Domain truth, source authority, identity, graph projection, package activation, API response schema, audit persistence, validation harness behavior, or replay checksum algorithms outside named telemetry handoffs. | `140` |
| Future reachability | Deferred reachability claim kinds, result states, model capability, analysis artifacts, claim policy. | Active MVP graph/gold behavior, solver selection, negative reachability facts. | `200`, inactive |

### 9.1 Context map authority boundary

A bounded context defines a semantic ownership boundary. A context-map edge defines how domain language, artifacts, authority, projection, validation, activation, diagnostics, or deferred material may cross from one bounded context to another.

The context map is a root-domain routing artifact. It must not define runtime record schemas, algorithms, failure precedence, lifecycle transitions, graph mutation behavior, package activation behavior, validation harness execution, API response behavior, or telemetry export behavior.

Every runtime-affecting context-map edge must name exactly one primary behavior owner spec and at least one exported owner contract. If no owner spec exports the required handoff, the edge is not active. The root-domain default is `no_implicit_effect`.

When a crossing artifact is missing, inactive, stale, ambiguous, checksum-invalid, out of scope, not covered by the active owner contract, or omitted from the declared edge row, `domain.md` must route the issue to the owner spec or Section 25 closure ledger. It must not infer fallback behavior.

`all_active_contexts` means every bounded context in Section 9 whose primary owner is not marked inactive. `none` means the edge is prohibited or deferred and has no active target context. `external_boundary` means non-Cadastre source material or product/system behavior outside the public Cadastre model.

### 9.2 Context map relationship type vocabulary

`ContextMapRelationshipType` is a closed root-domain vocabulary. A relationship type not listed in this table is invalid until a future `domain.md` revision adds it and `120-DOMAIN-CMAP-*` validation rows cover it.

| Relationship type | Required root-domain meaning | Forbidden interpretation |
| --- | --- | --- |
| `owner_import` | One context imports an exported contract name from another owner spec. | Copying or redefining the imported owner behavior. |
| `published_language` | One context supplies stable terminology or record names used by other contexts. | Treating terminology as runtime authorization. |
| `anti_corruption_translation` | External or adjacent terminology is translated into Cadastre-owned terms before use. | Letting external schemas, graph labels, source keys, or tool outputs define Cadastre truth. |
| `authority_handoff` | A context provides an owner-approved authorization or denial artifact consumed by another context. | Downstream inference without the handoff artifact. |
| `projection_handoff` | Authoritative records or owner-approved change records become derived projection input. | Projection state becoming source of record. |
| `activation_gate` | Production behavior may execute only after package, profile, policy, lifecycle, and manifest gates pass. | Single artifact, version string, signature scalar, or validation run as activation. |
| `validation_gate` | Validation proves behavior owned elsewhere through fixtures and acceptance reports. | Validation inventing new owner behavior. |
| `read_only_query` | A downstream context may query declared read models without mutation authority. | Query output mutating facts, identity, graph state, completeness, packages, or watermarks. |
| `telemetry_handoff` | Runtime diagnostics may support health or troubleshooting only through named telemetry and API/health policies. | Telemetry becoming fact, identity, completeness, graph, package, audit, watermark, or replay authority. |
| `deferred_no_effect` | Future-domain material is preserved but has no MVP production effect. | Deferred material emitting active facts, graph edges, graph properties, or user-facing claims. |
| `prohibited_dependency` | A tempting or unsafe dependency is explicitly forbidden. | Treating absence of a positive edge as permission by omission. |

### 9.3 Context map edge row shape

The context-map matrix is the root-domain interface for cross-context routing. It is not a runtime API, schema, event, graph edge, package dependency, processing DAG, or validation fixture.

The relationship matrix is split into two tables for readability. A context-map edge is complete only when its `edge_id` appears exactly once in both tables in Sections 9.4 and 9.5. Each edge row must populate every required field in this shape.

| Field | Required behavior |
| --- | --- |
| `edge_id` | Stable lower-snake-case identifier unique within Section 9. |
| `from_context` | One bounded context named in Section 9, `all_active_contexts`, or `external_boundary` for non-Cadastre inputs. |
| `to_context` | One bounded context named in Section 9, `all_active_contexts`, or `none` for prohibited or deferred edges. |
| `relationship_type` | One token from `ContextMapRelationshipType`. |
| `crossing_artifacts` | Exact exported contract names, exact owner-state names, or `none`. Broad prose such as “profile permits” is invalid. |
| `authority_class` | One `010.AuthorityClass` token when the edge concerns authority, otherwise `not_applicable`. |
| `behavior_owner` | Exact owner spec and exported contract that owns the primary runtime behavior, or `none` when the edge is prohibited, deferred, or root-domain-only. |
| `allowed_root_domain_effect` | One closed value: `owner_lookup`, `term_translation`, `boundary_warning`, `validation_route`, `deferred_no_effect`, or `none`. |
| `forbidden_runtime_effects` | Closed list of runtime effects that the edge must not imply. Empty lists are invalid for `anti_corruption_translation`, `prohibited_dependency`, `telemetry_handoff`, and `deferred_no_effect`. |
| `default_when_missing_or_invalid` | Default must be `no_implicit_effect` unless the referenced owner spec exports a stricter named default. Deferred material must use `deferred_no_effect`. |
| `validation_route` | Exact `120` validation row family, or `TODO:` when the row family is not yet defined. |
| `status` | Closed value: `resolved_owner_routed`, `blocked_validation`, `blocked_owner_todo`, or `inactive_deferred`. |

### 9.4 Context map relationship matrix: identity and owner routing

| `edge_id` | `from_context` | `to_context` | `relationship_type` | `crossing_artifacts` | `authority_class` | `behavior_owner` |
| --- | --- | --- | --- | --- | --- | --- |
| `doc_governance_to_all_contexts` | Documentation governance | `all_active_contexts` | `owner_import` | `SpecStatus`, `DocumentClass`, `SourceOfTruthRow`, `VolatilityClass` | `not_applicable` | `000.SourceOfTruthRow` |
| `system_boundary_to_all_contexts` | System boundary and authority | `all_active_contexts` | `published_language` | `AuthorityClass`, `DirectSourceProhibition`, `ProjectionAuthorityRule`, `RuntimeTelemetryAuthorityRule` | `not_applicable` | `010.AuthorityClass` |
| `lakehouse_feed_to_processing` | Lakehouse feed and table state | Processing and versioning | `authority_handoff` | `LakehouseFeedProfile`, `RawFeedManifest`, `LakehouseSnapshotRef`, `DatasetVersionRef`, `LakehouseCommitRef`, `LakehouseReadCompletenessReceipt` | `supporting_evidence` | `020.LakehouseReadCompletenessReceipt` |
| `processing_to_stage_owners` | Processing and versioning | `all_active_contexts` | `activation_gate` | `ProcessingStageDAG`, `StageStateRecord`, `VersionManifest`, `ActivationControlledArtifactRef`, `RunLockSet`, `RunLockLeasePolicy`, `RunLockOperationEvidence`, `RunLockRecoveryEvidence`, `RunLockCommitGuard` | `not_applicable` | `030.ExecuteProcessingStageDAG` |
| `core_model_to_all_contexts` | Core data model | `all_active_contexts` | `published_language` | `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `GoldFact`, `EvidenceRef`, `FactAbsenceOutcome`, `CanonicalJSON` | `not_applicable` | `040.CoreRecordSchema` |
| `normalization_to_source_authority` | Normalization and external schema | Source authority and absence | `anti_corruption_translation` | `CadastreSilverObservation`, `ExternalSchemaProfile`, `ObservationToOCSFMappingRowSet`, `SourceExtensionFieldRuleSet`, `ExternalSchemaAuthoritySignalMappingRow` | `supporting_evidence` | `050.NormalizeObservation` |
| `normalization_to_identity` | Normalization and external schema | Identity resolution | `anti_corruption_translation` | `CadastreSilverObservation`, `ExternalSchemaProfile`, `ObservationToOCSFMappingRowSet`, `SourceExtensionFieldRuleSet` | `supporting_evidence` | `050.NormalizeObservation` |
| `normalization_to_gold` | Normalization and external schema | Temporal and gold facts | `anti_corruption_translation` | `CadastreSilverObservation`, `ExternalSchemaProfile`, `ObservationToOCSFMappingRowSet`, `SourceExtensionFieldRuleSet` | `supporting_evidence` | `050.NormalizeObservation` |
| `normalization_to_graph` | Normalization and external schema | Graph projection and serving | `anti_corruption_translation` | `CadastreSilverObservation`, `ExternalSchemaProfile`, `ObservationToOCSFMappingRowSet`, `SourceExtensionFieldRuleSet` | `supporting_evidence` | `050.NormalizeObservation` |
| `source_authority_to_gold` | Source authority and absence | Temporal and gold facts | `authority_handoff` | `SourceAuthorityClosureMatrixRow`, `SourceAuthorityProfileRow`, `LakehouseFeedCompletenessProfileRow`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `AbsenceDerivationResult`, `ControlResultMappingRow` | `system_of_record` | `060.DeriveAbsenceOrUnknown` |
| `source_authority_to_graph_expiry` | Source authority and absence | Graph projection and serving | `authority_handoff` | `AbsenceDerivationResult` plus exact `SourceAuthorityClosureMatrixRow` or deterministic source-authority block row | `derived_projection` | `060.DeriveAbsenceOrUnknown` |
| `identity_to_gold_subjects` | Identity resolution | Temporal and gold facts | `authority_handoff` | `IdentityDecision`, `ResolverExplanation`, `CanonicalEntity`, `SourceAsset`, `Identifier` | `system_of_record` | `070.ResolveIdentity` |
| `identity_to_graph_projection` | Identity resolution | Graph projection and serving | `projection_handoff` | `IdentityDecision`, `GraphCorrectionHandoff` | `derived_projection` | `070.GraphCorrectionHandoff` |
| `gold_to_graph_projection` | Temporal and gold facts | Graph projection and serving | `projection_handoff` | `GoldFact`, `GoldFactChangeSet`, `TemporalObservationTimeResolution` | `derived_projection` | `080.GoldFactChangeSet` |
| `graph_to_api_serving` | Graph projection and serving | API, UX, health, error, security | `read_only_query` | `DerivedViewState`, `GraphQueryTranslationProfile`, `GraphApplyResult`, `GraphQueryResponse` | `derived_projection` | `090.QueryGraph` |
| `package_activation_to_stage_runtime` | Package activation | `all_active_contexts` | `activation_gate` | `ProductionPackageSetManifest`, `PackageReleaseManifest`, `PackageStageBinding`, `PackageCompatibilityMatrix`, `PackagePromotionRecord` | `not_applicable` | `100.ActivatePackageSet` |
| `validation_to_all_contexts` | Validation and acceptance | `all_active_contexts` | `validation_gate` | `ValidationMatrix`, `ValidationScenario`, `GoldenCorpus`, `AcceptanceReport` | `not_applicable` | `120.RunValidationMatrix` |
| `analysis_to_graph_read_model` | Analysis, enrichment, lineage, registry | Graph projection and serving | `read_only_query` | `AnalysisRuleBundle`, `RuleGraphCompatibilityMatrix`, `AnalysisFinding`, `AnalysisMetric` | `non_authoritative_analysis` | `130.AnalyzeGraph` |
| `analysis_to_api_rendering` | Analysis, enrichment, lineage, registry | API, UX, health, error, security | `read_only_query` | `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord` | `non_authoritative_analysis` | `130.AnalysisFinding` |
| `observability_to_health` | Observability and runtime telemetry | API, UX, health, error, security | `telemetry_handoff` | `TelemetryRuntimeState`, `TelemetryHealthMappingPolicy`, `TelemetrySignalPolicy`, `ObservabilityInstrumentationProfile` | `not_applicable` | `140.TelemetryHealthMappingPolicy` |
| `future_reachability_to_no_active_context` | Future reachability | `none` | `deferred_no_effect` | `ReachabilityResult`, `ReachabilityAnalysisArtifact`, `ReachabilityClaimPolicy` | `inactive_future_domain` | `none` |

### 9.5 Context map relationship matrix: effects, defaults, validation, and status

| `edge_id` | `allowed_root_domain_effect` | `forbidden_runtime_effects` | `default_when_missing_or_invalid` | `validation_route` | `status` |
| --- | --- | --- | --- | --- | --- |
| `doc_governance_to_all_contexts` | `owner_lookup` | Product runtime behavior; domain record fields; owner behavior promotion by implication. | `no_implicit_effect`; candidate or unregistered text must not drive implementation. | `120-DOMAIN-CMAP-*` | `blocked_validation` |
| `system_boundary_to_all_contexts` | `owner_lookup` | Data schemas; identity algorithms; graph algorithms; package activation; telemetry authority by implication. | `no_implicit_effect`; unowned authority claims fail or route to `TODO:`. | `120-DOMAIN-CMAP-*` | `blocked_validation` |
| `lakehouse_feed_to_processing` | `owner_lookup` | Raw import; absence; cleanup; retraction; graph expiry; watermark effect by read success alone. | `no_implicit_effect`; feed-read evidence alone has no source-level effect. | `120-DOMAIN-CMAP-*`; `120-SOURCE-CLOSURE-*` | `blocked_validation` |
| `processing_to_stage_owners` | `owner_lookup` | Stage output outside owner-permitted classes; owner algorithm substitution; package activation by stage presence; source authority, absence, cleanup, graph expiry, or watermark by run-lock signal. | `no_implicit_effect`; stages cannot emit outside permitted owner outputs and run-lock signals remain orchestration-only unless an owner handoff consumes persisted `030` evidence. | `120-DOMAIN-CMAP-*`; stage-specific validation rows; `120-RUNLOCK-CLOSE-*` | `blocked_validation` |
| `core_model_to_all_contexts` | `owner_lookup` | Runtime authorization; owner algorithm selection; source authority; identity merge; graph projection by record name alone. | `no_implicit_effect`; core names do not authorize owner behavior. | `120-DOMAIN-CMAP-*` | `blocked_validation` |
| `normalization_to_source_authority` | `term_translation` | Source authority; completeness; absence; cleanup; watermark advancement by parser or external schema output. | `no_implicit_effect`; mapped silver output is supporting evidence only. | `120-DOMAIN-CMAP-*`; `120-OCSF-MAP-*`; `120-SOURCE-CLOSURE-*` | `blocked_validation` |
| `normalization_to_identity` | `term_translation` | Canonical identity; create; attach; merge; split; reject; score by parser or external schema output. | `no_implicit_effect`; identity requires `070` resolver ownership. | `120-DOMAIN-CMAP-*`; identity validation rows | `blocked_validation` |
| `normalization_to_gold` | `term_translation` | Gold fact creation; fact-time selection; correction; replay authority by parser or external schema output. | `no_implicit_effect`; gold derivation requires authority and temporal owners. | `120-DOMAIN-CMAP-*`; `120-TEMPORAL-CORRECTION-*` | `blocked_validation` |
| `normalization_to_graph` | `term_translation` | Graph node; graph edge; graph direction; pathfinding; traversal; graph expiry by parser or external schema output. | `no_implicit_effect`; graph projection requires `090` owner rows. | `120-DOMAIN-CMAP-*`; `120-GRAPH-PROFILE-CLOSURE-*` | `blocked_validation` |
| `source_authority_to_gold` | `owner_lookup` | Direct downstream use of negative evidence; broad source-category fallback; correction without exact absence handoff. | `no_implicit_effect`; missing exact authority or absence handoff blocks fact or correction effect. | `120-DOMAIN-CMAP-*`; `120-SOURCE-CLOSURE-*`; `120-TEMPORAL-CORRECTION-*` | `blocked_validation` |
| `source_authority_to_graph_expiry` | `owner_lookup` | Graph expiry; graph cleanup; graph retraction from missing graph object, stale derived view, backend cleanup, or drift check. | `no_implicit_effect`; no graph expiry or cleanup without exact checksum-valid authorization. | `120-DOMAIN-CMAP-*`; `120-GRAPH-PROFILE-CLOSURE-*` | `blocked_validation` |
| `identity_to_gold_subjects` | `owner_lookup` | Fact subject creation from unresolved, weak, selector-only, review-only, or blocked identity. | `no_implicit_effect`; unresolved or blocked identity does not become a fact subject. | `120-DOMAIN-CMAP-*`; identity validation rows | `blocked_validation` |
| `identity_to_graph_projection` | `owner_lookup` | Direct graph mutation; graph edge projection; backend taxonomy selection; source authority for non-identity facts. | `no_implicit_effect`; resolver must not mutate graph state directly. | `120-DOMAIN-CMAP-*`; graph correction validation rows | `blocked_validation` |
| `gold_to_graph_projection` | `owner_lookup` | Backend graph truth; unsupported graph delta; unauthorized projection; graph edge as source fact. | `no_implicit_effect`; unsupported or unauthorized handoff emits no graph delta. | `120-DOMAIN-CMAP-*`; `120-GRAPH-PROFILE-CLOSURE-*` | `blocked_validation` |
| `graph_to_api_serving` | `owner_lookup` | Source truth; fact truth; identity creation; source completeness; audit persistence by graph read state. | `no_implicit_effect`; graph read model cannot create source truth, fact truth, or identity. | `120-DOMAIN-CMAP-*`; graph/API validation rows | `blocked_validation` |
| `package_activation_to_stage_runtime` | `owner_lookup` | Production activation from package artifact, version string, signature scalar, SBOM, provenance, dependency lock, or validation run alone. | `no_implicit_effect`; package artifacts do not activate production behavior alone. | `120-DOMAIN-CMAP-*`; package validation rows | `blocked_validation` |
| `validation_to_all_contexts` | `validation_route` | New domain behavior; owner behavior invention; runtime mutation by fixture or acceptance report. | `no_implicit_effect`; validation verifies owner behavior and cannot invent behavior. | `120-DOMAIN-CMAP-*` | `blocked_validation` |
| `analysis_to_graph_read_model` | `owner_lookup` | Fact mutation; graph mutation; completeness mutation; watermark mutation; identity mutation; package mutation; source-authority mutation. | `no_implicit_effect`; findings and metrics are non-authoritative. | `120-DOMAIN-CMAP-*`; analysis validation rows | `blocked_validation` |
| `analysis_to_api_rendering` | `owner_lookup` | API output that represents analysis as remediation, fact retraction, graph edge removal, or source completeness change. | `no_implicit_effect`; API rendering must preserve analysis non-authority. | `120-DOMAIN-CMAP-*`; analysis/API validation rows | `blocked_validation` |
| `observability_to_health` | `owner_lookup` | Domain truth; source authority; identity; graph projection; package activation; audit persistence; replay input. | `no_implicit_effect`; telemetry may affect caller-visible health only through named health mapping and must not affect authoritative records. | `120-DOMAIN-CMAP-*`; `120-OBSERVABILITY-*` | `blocked_validation` |
| `future_reachability_to_no_active_context` | `deferred_no_effect` | MVP facts; graph edges; graph properties; solver selection; negative reachability facts; unqualified reachability claims. | `deferred_no_effect`; no MVP effect exists. | `120-DOMAIN-CMAP-*`; reachability negative rows | `inactive_deferred` |

### 9.6 Prohibited context dependencies

Each row in this table is a `prohibited_dependency` context-map edge. Unless the row states otherwise, its `crossing_artifacts` value is `none`, its `authority_class` is `not_applicable`, its `allowed_root_domain_effect` is `boundary_warning`, its `default_when_missing_or_invalid` is `no_implicit_effect`, its `validation_route` is `120-DOMAIN-CMAP-PROHIBITED-*`, and its `status` is `blocked_validation`.

| `edge_id` | `from_context` | `to_context` | `behavior_owner` | Forbidden effect | Required routing |
| --- | --- | --- | --- | --- | --- |
| `prohibit_enterprise_source_direct_call` | `external_boundary` | System boundary and authority | `010.DirectSourceProhibition` | Raw evidence, completeness, identity, fact, graph, or production output authority from a production enterprise source call. | `010.DirectSourceProhibition`; `020` lakehouse feed contracts. |
| `prohibit_external_schema_as_authority` | `external_boundary` | Normalization and external schema | `050.ExternalSchemaProfile` | Canonical identity, source authority, completeness, gold fact, graph edge, or temporal semantics from OCSF, CIM, external schemas, observables, labels, or fields. | `050` mapping profile first, then consuming owner. |
| `prohibit_weak_progress_as_absence` | Lakehouse feed and table state | Source authority and absence | `060.DeriveAbsenceOrUnknown` | Negative fact, cleanup, retraction, graph expiry, or watermark advancement from missing feed-category rows, missing source-authority closure rows, source-history outside-window no-results, missing fields, TTL expiry, queue drain, heartbeat, watermark, ack, or freshness signal. | `060.DeriveAbsenceOrUnknown`; `020.LakehouseFeedCategoryClosureRow`; `120-SOURCE-CLOSURE-*`. |
| `prohibit_resolver_direct_graph_mutation` | Identity resolution | Graph projection and serving | `070.GraphCorrectionHandoff` | Graph node/edge creation, expiry, cleanup, or graph apply directly from resolver output. | `070.GraphCorrectionHandoff`; `090.ProjectGraphDeltas`. |
| `prohibit_graph_backend_as_record_truth` | Graph projection and serving | `all_active_contexts` | `090.ProjectionAuthority` | Fact creation, identity inference, completeness decision, fact retraction, bitemporal semantics, drift repair as truth, or backend internal ID leakage. | `090` derived read-model contracts. |
| `prohibit_package_artifact_as_activation` | Package activation | `all_active_contexts` | `100.ActivatePackageSet` | Production runtime behavior from a package artifact, version string, signature scalar, SBOM, provenance, dependency lock, or validation run alone. | `100.ProductionPackageSetManifest`. |
| `prohibit_analysis_as_mutation_authority` | Analysis, enrichment, lineage, registry | `all_active_contexts` | `130.Non-Authority Rule` | Fact, graph, completeness, watermark, identity, package, or source-authority mutation from analysis finding, metric, enrichment, lineage facet, registry label, or risk acceptance. | `130` non-authority rule and owner-specific named interface. |
| `prohibit_telemetry_as_domain_authority` | Observability and runtime telemetry | `all_active_contexts` | `140.TelemetryNonAuthorityRule` | Fact, identity, completeness, graph, package, audit, replay, or watermark authority from span, metric, structured log, exporter state, Collector state, dashboard, trace ID, or span ID. | `140.TelemetryNonAuthorityRule`; `110` health handoff only. |
| `prohibit_future_reachability_mvp_effect` | Future reachability | `none` | `200.Activation Rule` | `has_theoretical_reachability`, negative reachability, service-access claim, solver-selected truth, active graph edge, or active gold fact. | `200` inactive deferred boundary. |

## 10. Domain vocabulary

| Term | Definition | Not this | Canonical identifiers or tokens | Primary owner | Status |
| --- | --- | --- | --- | --- | --- |
| Bounded context | Semantic ownership boundary for domain language, owner routing, and forbidden substitutions. | Implementation module, package, table, route, or runtime service boundary. | Section 9 bounded-context names | `domain.md`, owner specs | Resolved. |
| Context map | Root-domain routing map of relationships between bounded contexts. | Runtime dependency graph, processing DAG, package dependency graph, graph read model, or validation harness. | Section 9 context-map edge rows | `domain.md` | Resolved. |
| Context-map edge | Documented relationship between two bounded contexts, with relationship type, crossing artifacts, behavior owner, no-effect default, and validation route. | Runtime API, schema, event, package dependency, graph edge, or source relationship. | `edge_id` in Section 9 | `domain.md`; runtime behavior owner named by row | Resolved. |
| Context-map relationship type | Closed root-domain vocabulary describing how a context-map edge crosses a boundary. | Free-form dependency prose or implementation coupling. | `ContextMapRelationshipType` tokens in Section 9 | `domain.md` | Resolved. |
| Crossing artifact | Exact exported contract, state name, or owner-approved artifact named by a context-map edge. | Broad prose, implementation object, private binding, external product field, or implicit dependency. | Exact owner-exported contract names | Named owner spec | Resolved owner-routed. |
| Cadastre | A lakehouse-fed interpretation, normalization, identity, fact, and projection system for temporal asset intelligence. | Enterprise source collector, SIEM replacement, graph-first sync engine. | `Cadastre` | `010` | Resolved. |
| Enterprise source system | External operational system that supplies or originates security, identity, endpoint, vulnerability, configuration, network, DNS, DHCP, IPAM, or related observations. | Cadastre production component. | None defined in public model. | `010`, `020` | Resolved as external. |
| External raw supplier | External pipeline, tool, team, or supplier class that delivers raw feeds into the lakehouse. | Cadastre production collector. | `RawSupplierProfile` | `020` | Resolved. |
| Lakehouse | Authoritative storage substrate for Cadastre raw/bronze, silver, identity, gold, graph-delta, health, and related record classes. | Graph backend, metadata graph, source system. | `LakehouseTableProfile`, `LakehouseSnapshotRef`, `DatasetVersionRef`, `LakehouseCommitRef` | `020`, `010` | Resolved. |
| Lakehouse feed | Vendor-neutral lakehouse-resident raw input feed that Cadastre may read. | Direct API collection or source exploration. | `LakehouseFeedProfile`, `RawFeedManifest` | `020` | Resolved. |
| Lakehouse feed profile schema | Stable schema for public feed profile fields, defaults, branch inputs, validation refs, and activation scope. | Missing profile row as default partial permission. | `LakehouseFeedProfileSchema` | `020` | Resolved as owner-routed behavior. |
| Lakehouse feed category closure row | Activation-controlled row that closes feed-category read branches, absence-sensitive domains, upstream evidence requirements, default missing-row result, and deterministic block/no-op behavior. | Feed read success as source completeness, or category name as implicit permission. | `LakehouseFeedCategoryClosureRow`, `LakehouseFeedCategoryClosureRowSet` | `020` | Resolved as owner-routed behavior; active rows still gate use. |
| Lakehouse feed feasibility assessment | Runtime state record proving feed readiness or deterministic blocking reasons before activation. | Runtime authority by existence, or private source inventory. | `LakehouseFeedFeasibilityAssessment` | `020` | Resolved as owner-routed behavior. |
| Lakehouse feed completeness profile row | Exact `060` row that maps feed-read state, upstream evidence, scope, coverage, authority, staleness, and allowed effects into one completeness decision. | Private completeness capability as public source authority, or missing `allowed_effects` as permission. | `LakehouseFeedCompletenessProfileRow` | `060` | Resolved as owner-routed behavior; active rows still gate effects. |
| Raw feed manifest | Per-batch or per-snapshot feed identity, counts, byte totals, hashes, time bounds, schema refs, supplier lineage, and completeness evidence refs. | Source completeness decision by itself. | `RawFeedManifest` | `020` | Resolved. |
| Lakehouse read completeness receipt | Evidence that Cadastre read a declared lakehouse feed target completely or partially. | Source-level absence, retraction, cleanup, graph expiry, or watermark authority by itself. | `LakehouseReadCompletenessReceipt`; states in Section 14. | `020`, `060` | Resolved. |
| Upstream completeness evidence | Supplier-provided evidence about upstream source collection, permission scope, partiality, or failure. | Cadastre source completeness decision by itself. | `UpstreamCompletenessEvidence` | `020`, `060` | Resolved. |
| Raw record | Cadastre record preserving source-native evidence and import lineage. | Parsed output, normalized observation, gold fact. | `RawRecord` | `040`, `020` | Resolved. |
| Cadastre silver observation | Cadastre-owned normalized observation envelope produced from parse and mapping stages. | Canonical fact or identity decision. | `CadastreSilverObservation` | `040`, `050` | Resolved. |
| `normalized_fields` | Silver observation member aligned to active OCSF profile unless observation is declared `cadastre_only`. | Cadastre-owned omission, lineage, identity, temporal, confidence, or flow-role authority. | `normalized_fields` | `050`, `040` | Resolved. |
| `source_extension_fields` | Declared, typed, bounded, namespaced source-specific fields in a silver observation. | Arbitrary external custom fields or OCSF `unmapped`. | `SourceExtensionFieldRuleShape`, `SourceExtensionFieldRule` | `040`, `050` | Resolved. |
| External schema profile | Governed external schema baseline used for validation and mapping. | Cadastre canonical fact, identity, source authority, graph, or persistence model. | `ExternalSchemaProfile`, `ExternalSchemaArtifactRef` | `050` | Resolved. |
| OCSF | External schema profile target for supported `normalized_fields`. | Cadastre truth layer or identity resolver. | OCSF profile identifiers owned by `050`. | `050` | Resolved. |
| CIM projection | Lossy deterministic Splunk CIM-facing projection. | Authoritative silver schema or proof that no information was lost. | `CIMProjectionProfile`, `ProjectionLossManifest` | `050` | Resolved. |
| Canonical entity | Product-owned canonical entity identity. | Source-native asset, graph node, hostname, IP, DNS name, display label. | `CanonicalEntity` | `040`, `070` | Resolved. |
| Source asset | Source/feed-scoped asset identity with source-native evidence. | Canonical entity. | `SourceAsset` | `040`, `070` | Resolved. |
| Identifier | Typed normalized observed identifier with scope and validity. | Identity decision. | `Identifier`, `IdentifierScope` | `040`, `070` | Resolved. |
| Identity evidence item | Typed resolver input with evidence class, scope, time, durability, reuse risk, contribution, blockers, and evidence refs. | Identity decision or canonical entity. | `IdentityEvidenceItem`, `IdentifierEvidenceClass` | `070` | Resolved. |
| Resolver profile | Sole production authority for identity resolution behavior. | Generic scoring heuristic or implementation module. | `ResolverProfile` | `070` | Resolved. |
| Identity decision | Governed resolver output that creates, attaches, merges, rejects, splits, conflicts, or declines identity relationships. | Identifier equality, source-native merge history, graph key, reviewer note. | `IdentityDecision`; decision states owned by `070` | `070`, `040` | Resolved. |
| `canonical_created` | Routing-only identity decision vocabulary for a new canonical entity created by resolver-owned durable evidence rules. | Domain permission to create identity outside `070`. | `canonical_created` | `070` | Resolved as owner-routed vocabulary. |
| `source_asset_attached` | Routing-only identity decision vocabulary for attaching a source asset to exactly one existing canonical entity. | Merge of two canonical entities. | `source_asset_attached` | `070` | Resolved as owner-routed vocabulary. |
| `auto_merged` | Routing-only identity decision vocabulary for resolver-owned canonical merge. | Weak evidence merge or reviewer note. | `auto_merged` | `070` | Resolved as owner-routed vocabulary. |
| `identity_candidate` | Routing-only identity decision vocabulary for a non-mutating candidate. | Canonical identity mutation. | `candidate` | `070` | Resolved as owner-routed vocabulary. |
| `identity_rejected` | Routing-only identity decision vocabulary for rejected candidate evidence. | Source absence or graph cleanup. | `rejected` | `070` | Resolved as owner-routed vocabulary. |
| `identity_split` | Routing-only identity decision vocabulary for split output and graph correction handoff. | Direct graph mutation. | `split` | `070`, `080`, `090` | Resolved as owner-routed vocabulary. |
| `identity_conflicted` | Routing-only identity decision vocabulary for unresolved conflicting identity evidence. | Authorized merge. | `conflicted` | `070` | Resolved as owner-routed vocabulary. |
| `identity_no_decision` | Routing-only identity decision vocabulary for unsupported, weak, selector-only, overflowed, or out-of-scope evidence. | Negative fact or deletion. | `no_decision` | `070` | Resolved as owner-routed vocabulary. |
| Unresolved target reference | Relationship hint that has not created or merged canonical identity. | Weak entity or automatic relationship target. | `UnresolvedTargetReference` | `070` | Resolved. |
| Source authority profile | Executable authority arbitration for gold fact type and predicate. | Source category preference, source completeness, or staleness policy. | `SourceAuthorityProfile`, `SourceAuthorityProfileRow` | `060` | Resolved. |
| Coverage assertion | Source-backed coverage state required before coverage-dependent facts or negative claims may be emitted. `CoverageAssertion` consumes `CoverageDomainToken` through `060`-owned coverage profiles; it does not define coverage-domain token identity. | Scanner presence, source presence, feed read success by itself, or coverage-domain token definition. | `CoverageAssertion`, `CoverageDimensionProfile` | `060` | Resolved at stable contract level; active source-specific row instances are activation-controlled and validation-blocked until present. |
| Coverage domain token | Canonical lower-snake-case token used to identify the coverage domain evaluated by source authority, feed category closure, coverage profiles, and validation. | Feed category token, display label, OCSF class, source dataset, package type, UI label, or legacy spelling. | `CoverageDomainToken`, `CoverageDomainCatalog` | `060` | Resolved as owner-routed behavior; active rows and validation still gate effects. |
| Omission state | Explicit field-level state for present, empty, not provided, not applicable, permission limited, unavailable, unsupported, malformed, or redacted values. | Nullable schema field or missing source field. | `observed_present`, `observed_empty`, `not_provided`, `not_applicable`, `permission_limited`, `source_unavailable`, `unsupported`, `malformed`, `redacted` | `040` | Resolved. |
| Fact absence outcome | Fact-level absence or unknown outcome derived through authority, completeness, coverage, and staleness gates. | Field omission state or caller-visible source-state label. | `FactAbsenceOutcome`, `AbsenceDerivationResult` | `040`, `060` | Resolved at stable contract level; `040` owns tokens and `060` owns production derivation. |
| Gold fact | Canonical bitemporal assertion with subject, predicate, object/value, valid time, known time, assertion state, confidence, authority, evidence, and correction refs. | Silver observation or graph edge. | `GoldFact`; assertion states in Section 14. | `040`, `060`, `080` | Resolved. |
| Evidence ref | Product-owned reference from an output to supporting artifact, role, checksum, and visibility state. | Raw payload itself or external lineage facet. | `EvidenceRef` | `040`, `130` | Resolved. |
| Temporal observation time resolution | Persisted resolution of the time inputs selected for fact derivation. | Implicit current-time fallback. | `TemporalObservationTimeResolution` | `080` | Resolved. |
| ScopeSelector | Runtime selector object used by owners to express request scope or row scope. | Public/private binding, wildcard string, row-order tiebreaker, graph identity, telemetry authority, or package environment string. | `ScopeSelector` | `030` | Resolved as owner-routed vocabulary. |
| ActivationScope | Exact alias of `ScopeSelector` for artifact and row activation. | A second selector schema or owner-local activation-scope object. | `ActivationScope` | `030` | Resolved as owner-routed vocabulary. |
| ScopeSelectorContext | Owner-supplied context that declares required dimensions, optional dimensions, subset-allowed keys, specificity diagnostics, and owner error mapping for `030` selector algorithms. | A place for owners to redefine selector equality, coverage, or ambiguity rules. | `ScopeSelectorContext` | `030`; owner specs instantiate contexts | Resolved as owner-routed vocabulary. |
| Scope coverage | Whether a row selector covers a request selector under `030.ScopeSelectorCovers`. | Broad source-category fallback, dataset-level universal wildcard, row order, or private binding match. | `ScopeSelectorCovers` | `030` | Resolved as owner-routed vocabulary. |
| Scope specificity | Partial-order comparison of covering selectors under `030.CompareScopeSpecificity`. | Lexical row ID, file order, package order, backend order, insertion order, or validation order tiebreaking. | `CompareScopeSpecificity` | `030` | Resolved as owner-routed vocabulary. |
| Scope ambiguity | State where more than one maximal scoped row remains or selectors are incomparable under `030`. | Conflict, authorized absence, source truth, graph mutation permission, or implementation-local tiebreaking. | `ResolveScopedRow`; `ScopeSelectorErrorCodeSet` | `030`, `110` for rendering | Resolved as owner-routed vocabulary. |
| Version manifest | Immutable manifest of every output-affecting profile, policy, package, state, lakehouse ref, completeness receipt, coverage assertion, schema artifact, temporal policy, resolver profile, graph profile, graph apply result, and acceptance report that can affect output or replay. | Build number, run ID, or deployment label by itself. | `VersionManifest` | `030`, cross-cutting | Resolved. |
| Processing stage DAG | Acyclic deterministic stage graph with declared inputs, outputs, profiles, state kinds, lifecycle, and failure behavior. | Implementation scheduler or package module layout. | `ProcessingStageDAG` | `030` | Resolved. |
| Lifecycle status | Shared lifecycle status vocabulary for activation-controlled records. | Full transition table or UI status label. | `draft`, `validated`, `canary`, `active`, `deprecated`, `retired`, `quarantined` | `030` | Resolved. |
| Graph projection profile | Mapping from gold facts and declared synthetic structural rules to graph nodes, edges, properties, expiry scopes, and no-op outcomes. | Graph backend schema or query language. | `GraphProjectionProfile` | `090` | Resolved. |
| Graph delta set | Persisted set of graph node and edge deltas produced by projection. | Backend mutation log or authoritative facts. | `GraphDeltaSet` | `090`, `040` | Resolved. |
| Graph edge semantics | One active row per graph edge type defining source fact type, predicate, direction, evidence, states, temporal policy, confidence policy, traversal class, non-implications, and no-op conditions. | Backend relationship type or external graph taxonomy. | `GraphEdgeSemantics`, `GraphTraversalClass` | `090` | Resolved. |
| Graph read model | Derived graph-serving state rebuilt from authoritative records and persisted graph deltas. | System of record. | `DerivedViewState`, `GraphBackendProfile` | `090`, `010` | Resolved. |
| Package artifact | Immutable package materialization subject to trust and release evidence. | Production activation target by itself. | `PackageArtifact` | `100` | Resolved. |
| Core record schema registry | Registry of field schemas, ID policies, checksum policies, null/omit/default rules, extension policies, and validation behavior for exported core records. | Runtime behavior in downstream specs or a planning document. | `CoreRecordSchemaRegistry`, `CoreRecordSchema`, `CoreRecordFieldRow` | `040` | Resolved. |
| Common record header | Shared persisted-record header for record type, schema version, record ID, checksum, and version manifest. | Record-specific business fields or API response envelope. | `CommonRecordHeader` | `040` | Resolved. |
| Gold fact semantic key | Stable comparable semantic key for a fact assertion. | Immutable knowledge assertion record ID. | `gold_fact_key_id` | `040`, consumed by `080` | Resolved. |
| Gold fact record ID | Immutable record ID for one knowledge assertion version. | Semantic fact key or mutable correction handle. | `gold_fact_id` | `040`, consumed by `080` | Resolved. |
| Evidence ref | Bounded evidence pointer. It is not raw payload storage. | Raw payload bytes, source payload body, or unrestricted drillback. | `EvidenceRef` | `040`, exposure governed by `110` | Resolved. |
| Graph delta primitive shape | Primitive graph delta record shape, not graph projection authority. | Graph projection profile, graph apply policy, or backend storage schema. | `GraphNodeDeltaShape`, `GraphEdgeDeltaShape` | `040`, runtime semantics in `090` | Resolved. |
| Production package set manifest | Immutable coherent set of package releases that is the only production activation target. | Single package version, tag, signature status, SBOM, provenance, or validation result. | `ProductionPackageSetManifest` | `100` | Resolved. |
| Validation matrix | Validation artifact mapping contract, scenario, fixture, expected output, expected error/no-op, owner spec, and pass/fail evidence. | Behavior owner or test plan replacing domain specs. | `ValidationMatrix` | `120` | Resolved. |
| Analysis finding | Read-only non-authoritative analysis output. Runtime closure is validation-gated by `120`. | Gold fact, remediation, risk reduction proof, source completeness. | `AnalysisFinding` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Analysis metric | Read-only non-authoritative dashboard or report metric. Runtime closure is validation-gated by `120`. | Authoritative risk score, fact, source authority, remediation, graph mutation. | `AnalysisMetric` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Risk acceptance record | Workflow metadata tied to analysis findings. Runtime closure is validation-gated by `120`. | Remediation, retraction, risk reduction proof, source completeness, graph edge removal. | `RiskAcceptanceRecord` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Threat-intel enrichment profile | Non-authoritative enrichment activation profile. Runtime closure is validation-gated by `120`. | Identity authority, source authority, source completeness, fact authority, graph mutation, coverage, watermark. | `ThreatIntelEnrichmentProfile` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Threat-intel enrichment record | Enrichment context for indicators, sightings, taxonomies, galaxies, object templates, confidence labels, distribution controls. Runtime closure is validation-gated by `120`. | Identity, source authority, gold fact, graph edge, absence, cleanup, retraction, coverage, or watermark authority. | `ThreatIntelEnrichmentRecord` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Threat-intel distribution mapping policy | Non-authoritative mapping from external distribution or TLP-like labels to visibility and redaction. Runtime closure is validation-gated by `120`. | Source authority, source completeness, production approval, or export authority by itself. | `ThreatIntelDistributionMappingPolicy` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Threat-intel artifact ref | Immutable ref and checksum for imported threat-intel source artifacts. Runtime closure is validation-gated by `120`. | Identity evidence, fact evidence, source authority, package activation authority. | `ThreatIntelArtifactRef` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Lineage facet mapping policy | Non-authoritative lineage facet routing and rejection policy. Runtime closure is validation-gated by `120`. | Evidence authority, table snapshot proof, source completeness, graph rebuild proof. | `LineageFacetMappingPolicy` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Run dataset IO contract | Runtime lineage state for dataset inputs and outputs. Runtime closure is validation-gated by `120`. | Evidence authority, source completeness, table snapshot proof, source authority. | `RunDatasetIOContract` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Artifact class policy | Governance boundary preventing artifact substitution. Runtime closure is validation-gated by `120`. | Package activation authority, fact authority, graph authority, source completeness. | `ArtifactClassPolicy` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Registry governance artifact | Governance metadata for owners, domains, classifications, glossary labels, policies, lifecycle, custom properties, checksums. Runtime closure is validation-gated by `120`. | Cadastre fact or graph authority. | `RegistryArtifactGovernance` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Registry custom property schema | Typed bounded metadata schema for registry custom properties. Runtime closure is validation-gated by `120`. | Core field schema, fact field, source-extension field, authority grant by itself. | `RegistryCustomPropertySchema` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Registry classification policy | Governance classification and glossary label policy. Runtime closure is validation-gated by `120`. | Fact authority, graph authority, source authority, production approval by itself. | `RegistryClassificationPolicy` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Derived graph edge rule | Non-authoritative or owner-routed derived edge rule. Runtime closure is validation-gated by `120`. | Direct graph mutation, unsupported generated edge, gold output outside `080`, graph output outside `090`. | `DerivedGraphEdgeRule` | `130` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| Future reachability | Inactive future analysis domain for modeled packet/session/service/identity-conditioned access. | Active MVP graph edge, boolean reachability property, or negative reachability fact. | `ReachabilityResult`, `ReachabilityClaimPolicy`; inactive. | `200`, `090` | Inactive deferred. |
| Private source binding | Private implementation mapping from concrete vendor/product/source route to public vendor-neutral feed contract. | Public canonical domain concept. | `PrivateSourceFeedBinding` | `010` | Resolved as private-only. |
| Source authority closure row chain | Owner-routed validation chain proving exact authority, completeness, coverage, staleness, visibility, progress-signal, control-result, source-history, absence-policy, and watermark-policy refs for one requested absence-sensitive effect. Missing, ambiguous, inactive, checksum-mismatched, out-of-scope, stale, unvalidated, or `TODO` rows must not be interpreted as permission. | Runtime authority, schema fields, row-resolution algorithm, or validation harness behavior. | `SourceAuthorityClosureMatrixRow`, `SourceAuthorityClosureMatrixRowSet` | `060`; validation owner `120` | Vocabulary routed; runtime behavior owner-gated. |
| Deterministic source-authority block row | Explicit active row that blocks a category, dataset, fact, predicate, scope, or requested effect and proves mutation prohibition for the blocked scope. | Positive authorization, absence authorization, cleanup permission, graph expiry, retraction, watermark advancement, compliance pass/fail, or no-change proof. | `LakehouseFeedCategoryClosureRow`, `SourceAuthorityClosureMatrixRow` | `020` for feed-category block rows; `060` for source-authority block rows | Vocabulary routed; block behavior owner-gated. |
| Source-history no-change proof | Bounded source-native history interpretation requiring `SourceHistoryRetentionProfile`, source authority, staleness, completeness, and source-history coverage. Outside-window no-results must not become no-change proof. | Cadastre replay retention, source absence by default, graph expiry, cleanup, or watermark advancement by source-history silence. | `SourceHistoryRetentionProfile`, `CoverageDimensionProfile`, `AbsenceDerivationResult` | `060` | Vocabulary routed; validation-gated by `120`. |
| Absence-sensitive effect | Closed effect set `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`; every active use must route to `060` and `030.VersionManifest` refs. | Additional effect tokens by inference; positive-only observation permission; graph edge emission. | `SourceAuthorityClosureMatrixRow.requested_effect` | `060` | Vocabulary routed; closed effect set imported by `020`, `030`, `080`, `090`, `100`, `110`, and `120`. |

## 10A. DomainTermRegistry

`DomainTermRegistry` defines canonical terminology and owner routing. It does not define runtime behavior.

| Term | Canonical meaning | Forbidden substitution | Owner spec | Runtime owner | Validation owner | Glossary status |
| --- | --- | --- | --- | --- | --- | --- |
| `source_of_truth` | The document-status and ownership decision made by `000`. | README, manifest, glossary, ADR, research, private binding artifact, or archive. | `000` | `000` governance only. | `120` | canonical |
| `domain_term` | A vocabulary item routed by `domain.md`. | Runtime field, API label, backend label, package name, route name. | `domain` | Owning behavior spec named in this table. | `120` | canonical |
| `runtime_behavior` | Observable behavior owned by exactly one active NLSpec. | Rationale, research evidence, glossary prose. | owner spec | Owner spec named by `000`. | `120` | canonical |
| `source_authority` | Permission to treat source evidence as truth for a fact, predicate, scope, and absence class. | Feed-read success, freshness, graph state, lineage, scanner state. | `060` | `060` | `120` | canonical |
| `coverage_domain_token` | Canonical lower-snake-case token used by source authority, feed-category closure, coverage profiles, and validation to identify a coverage domain. | Feed category token, display label, OCSF class, source dataset, package type, UI label, or legacy spelling. | `060` | `060.CoverageDomainToken`, `060.CoverageDomainCatalog` | `120-COVERAGE-DOMAIN-*` | canonical |
| `canonical_serialization` | Byte-stable serialization used for IDs, checksums, replay, validation, graph deltas, audit, and package gates. | Backend natural order, language JSON defaults, unordered maps. | `040` | `040` | `120` | canonical |
| `core_record_schema_registry` | Registry of field schemas, ID policies, checksum policies, null/omit/default rules, extension policies, and validation behavior for exported core records. | Downstream owner field schema, DTO inventory, or planning artifact. | `040` | `040` | `120` | canonical |
| `common_record_header` | Shared persisted-record header for record type, schema version, record ID, checksum, and version manifest. | API envelope, graph object label, or package-local metadata. | `040` | `040` | `120` | canonical |
| `gold_fact_key_id` | Stable comparable semantic key for a fact assertion. | Immutable knowledge assertion record ID. | `040` | `040`, consumed by `080` | `120` | canonical |
| `gold_fact_id` | Immutable record ID for one knowledge assertion version. | Comparable fact key or correction mutable handle. | `040` | `040`, consumed by `080` | `120` | canonical |
| `raw_feed_manifest` | Feed batch, object, partition, schema, lineage, time-bound, checksum, and completeness-reference record. | Supplier delivery summary or private binding artifact. | `020` | `020` | `120` | canonical |
| `resolver_profile` | Sole production authority for identity resolution behavior. | Confidence score, graph key, source-native merge history, selector. | `070` | `070` | `120` | canonical |
| `replay_equivalence` | Deterministic equality rule for replay outputs and required refs. | Ad hoc checksum or current-state comparison. | `080` | `080` | `120` | canonical |
| `graph_backend_profile` | Activation profile for a graph serving backend and its preflight requirements. | Backend selection, backend docs, benchmark result. | `090` | `090` | `120` | canonical |
| `package_set` | Immutable coherent activation target for production packages. | Single package artifact, version string, signature scalar, dependency lock. | `100` | `100` | `120` | canonical |
| `package_type` | Canonical package behavior category used for activation policy resolution. | Package name, module name, artifact filename, repository path, version string, broad package label, grouping bundle, or runtime alias. | `100` | `100.PackageType` | `120-PACKAGE-TYPE-*` | Resolved owner-routed; package activation remains validation-gated. |
| `package_type_policy` | Row-set-controlled activation policy selected for a package type. | Broad package category prose, `policy_bundle`, grouping bundle, or implementation-local default. | `100` | `100` | `120` | canonical |
| `last_known_good_package_set` | Verified rollback candidate after activation and health gates pass. | Recently activated package set by itself. | `100` | `100` | `120` | canonical |
| `package_quarantine` | Immutable block record over package activation targets. | Artifact mutation, signature rewrite, or hidden denylist. | `100` | `100` | `120` | canonical |
| `emergency_package_override` | Bounded emergency action record. | Trust bypass or unverified activation. | `100` | `100` | `120` | canonical |
| `validation_matrix` | Executable row set proving success, rejection, no-op, edge, replay, redaction, authorization, and negative authority-boundary behavior. | Test-plan prose or aggregate acceptance claim. | `120` | `120` | `120` | canonical |
| `analysis_finding` | Non-authoritative analysis result routing. | Fact authority, identity authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `analysis_metric` | Non-authoritative analysis metric routing. | Authoritative risk score, fact authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `risk_acceptance_record` | Non-authoritative workflow routing for accepted analysis risk context. | Remediation, retraction, risk reduction proof, fact authority, source completeness, graph mutation, package activation authority, or watermark authority. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `threat_intel_enrichment_profile` | Non-authoritative threat-intel enrichment activation profile routing. | Fact authority, identity authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `threat_intel_enrichment_record` | Non-authoritative threat-intel enrichment output routing. | Fact authority, identity authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `threat_intel_distribution_mapping_policy` | Non-authoritative distribution and redaction routing for threat-intel labels. | Source completeness, source authority, export authority by itself, production approval, graph mutation, identity authority, or fact authority. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `threat_intel_artifact_ref` | Immutable ref and checksum routing for threat-intel source artifacts. | Identity evidence, source authority, package activation authority, fact evidence, graph mutation, completeness, or watermark authority by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `run_dataset_io_contract` | Runtime lineage state routing for dataset inputs and outputs. | Evidence authority, source completeness, table snapshot proof, graph rebuild proof, fact authority, or source authority by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `lineage_facet_mapping_policy` | Non-authoritative lineage facet mapping and rejection routing. | Evidence authority, source completeness, table snapshot proof, graph rebuild proof, fact authority, or source authority by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `artifact_class_policy` | Artifact substitution boundary routing. | New runtime behavior, package activation authority, fact authority, graph authority, or source completeness by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `registry_artifact_governance` | Non-authoritative registry governance metadata routing. | Fact authority, identity authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `registry_custom_property_schema` | Typed bounded custom-property metadata routing. | Core field schema, fact authority, source-extension field rule, graph mutation, package activation authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `registry_classification_policy` | Non-authoritative classification and glossary label routing. | Fact authority, identity authority, source authority, completeness, graph mutation, package activation authority, watermark authority, or production approval by itself. | `130` | `130` | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `derived_graph_edge_rule` | Owner-routed derived-edge rule vocabulary. | Direct graph mutation, graph authority outside `090`, gold output outside `080`, identity authority, source completeness, package activation authority, or watermark authority by itself. | `130` | `130`, routed through `080` or `090` when effect is permitted | `120` | Vocabulary resolved; runtime closure validation-gated by `120`. |
| `deferred_reachability` | Future theoretical reachability domain preserved inactive with no MVP graph or gold effect. | Observed connection, graph traversal, provider analyzer truth. | `200` | none while deferred. | `120` | deferred |
| `stable_core_contract` | Long-lived owner NLSpec contract text that defines product behavior. | Profile row, mapping table, package payload, fixture, runtime state, or research report. | `000`, `010`, and named owner spec | Owner spec only. | `120` | canonical |
| `activation_controlled_artifact` | Versioned artifact that instantiates stable owner behavior through rows, profiles, packages, fixtures, or registries. | New runtime behavior, new authority class, or override of core semantics. | `030`, `100` when package-supplied, and named owner spec | No authority except through owner contract. | `120` | canonical |
| `runtime_state_record` | Persisted execution, replay, commit, snapshot, lifecycle, health, derived-view, or validation state. | Source truth or activation authority by existence. | Domain owner and `030` | No authority except as owner-permitted state evidence. | `120` | canonical |
| `reference_evidence` | Non-runtime evidence such as research reports, standards, examples, or source notes. | Implementation authority or production behavior. | `000` | none. | `120` | canonical |
| `rationale` | Non-runtime decision explanation such as an ADR. | Runtime field, algorithm, default, error, or acceptance criterion. | `000` | none. | `120` | canonical |
| `inactive_future_domain` | Deferred future material with no MVP implementation effect. | Active product behavior or production output. | `000`, `200` | none while deferred. | `120` | canonical |

## 10B. RootDomainConflictRule

If `docs/nlspec/domain.md` conflicts with an owner spec for runtime behavior, the owner spec controls the behavior and the conflict must be recorded as a owner-spec `TODO:` or validation failure in `120`. If two owner specs conflict, the conflict is a corpus defect. `domain.md` must not choose a side unless the affected behavior is caller-equivalent.

## 10C. TermPromotionRule

A glossary term may become canonical only when `docs/nlspec/domain.md` adds or updates the term and the relevant owner spec confirms the runtime owner. A glossary-only term remains reference-only and must not be cited as implementation authority.

## 11. Core entities

| Entity | Definition | Identity basis | Lifecycle basis | Key relationships | Primary owner | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Cadastre system | The product boundary that consumes lakehouse feeds and emits interpreted, normalized, identity, fact, graph, API, and optional projection outputs. | Product boundary, not a persisted entity ID. | Spec status and deployment/package lifecycle, as applicable. | Reads lakehouse feeds, writes authoritative records, serves projections. | `010`, `100`, `030` | Resolved. |
| External raw supplier | External provider of lakehouse-resident raw feed data and supplier metadata. | Supplier class/profile identity, not concrete vendor route by default. | Feed/profile activation owned by lakehouse and package specs. | Supplies lakehouse feed; may provide upstream completeness evidence. | `020`, `010` | Resolved. |
| Lakehouse feed | Vendor-neutral lakehouse-resident raw input feed. | `LakehouseFeedProfile` plus feed scope and read target refs. | Feed profile activation and availability checks. | Produces raw feed rows/objects and completeness receipt. | `020` | Resolved. |
| Raw feed manifest | Batch/snapshot manifest for lakehouse feed input. | Manifest checksum and declared feed identity. | Feed read/import lifecycle. | Referenced by raw import, completeness evidence, version manifest. | `020` | Resolved. |
| Raw record | Preserved source-native evidence record imported from declared feed input. | Deterministic raw record ID from `040.ComputeRawRecordId`. | Duplicate/quarantine/import status; table retention. | Supports parse results, silver observations, evidence refs. | `040`, `020` | Resolved; field schema and ID policy owned by `040`. |
| Cadastre silver observation | Cadastre-owned normalized observation envelope. | Observation ID from raw refs, observation type, mapping bundle, source scope, normalized payload checksum. | Parse/normalize stage status and mapping validation. | Supports identity evidence, authority evaluation, gold derivation, CIM/OCSF output. | `040`, `050` | Resolved. |
| Canonical entity | Product-owned canonical asset/security entity. | Product-owned canonical entity ID; must not be inferred from labels, source IDs, graph IDs, or backend IDs. | Entity lifecycle state, valid/known bounds, identity decision refs. | Related to source assets, identifiers, gold facts, graph nodes. | `040`, `070` | Resolved. |
| Source asset | Source/feed-scoped asset record. | Source/feed scope plus source-native identity evidence. | Source asset lifecycle evidence. | Related to canonical entity through identity decisions. | `040`, `070` | Resolved. |
| Identifier | Typed observed identifier. | Identifier type, normalized value, source scope, validity interval. | Valid/known time and quality. | Feeds identity evidence items and resolver decisions. | `040`, `070` | Resolved. |
| Identity evidence item | Materialized resolver input. | Evidence item identity from evidence class, identifier/scope/time/evidence refs as defined by `070`. | Resolver input lifecycle. | Feeds candidate generation, blockers, decisions, explanations. | `070` | Resolved. |
| Identity decision | Resolver output. | Decision ID and checksum as defined by resolver profile and version manifest. | Active/terminal review and split behavior through identity owner. | Creates/rejects/splits/conflicts canonical identity relationships; may trigger graph correction handoff. | `070` | Resolved. |
| Gold fact | Canonical bitemporal fact. | `gold_fact_key_id` identifies the comparable semantic assertion; `gold_fact_id` identifies one immutable knowledge assertion version. | Assertion state and correction change sets. | Supports graph projection, API detail, analysis, audit. | `040`, `060`, `080` | Resolved. |
| Evidence ref | Product-owned bounded metadata pointer, not raw payload storage. | Referenced artifact class, artifact ID, role, checksum. | Visibility/redaction state. | Links graph, gold, silver, raw, validation, lineage as permitted. | `040`, `110`, `130` | Resolved. |
| Coverage assertion | Coverage-sensitive authorization input. | Coverage assertion ID and dimension profile refs. | Current/stale/expired coverage state through `060`. | Required before coverage-dependent absence or negative claims. | `060` | Resolved with coverage catalog TODO. |
| Version manifest | Immutable replay and output-affecting manifest. | Manifest checksum over declared output-affecting refs. | Production/replay/activation lifecycle. | References profiles, policies, packages, state, lakehouse refs, validation reports. | `030` | Resolved. |
| Graph node delta | Primitive graph node projection delta. | Cadastre-owned deterministic graph node ID. | Delta set and graph apply lifecycle. | Derived from gold facts or declared structural rules; applied to graph read model. | `040`, `090` | Resolved. |
| Graph edge delta | Primitive graph edge projection delta. | Cadastre-owned deterministic graph edge ID; not backend ID. | Delta set and graph apply lifecycle. | Governed by graph edge semantics and projected from facts/rules. | `040`, `090` | Resolved. |
| Graph read-model object | Derived graph serving object after graph apply. | Cadastre graph ID translated through backend profile. | Derived view state and graph apply result. | Served by graph query; drillback to gold/silver/raw. | `090`, `110` | Resolved. |
| Package release | Immutable package release record. | Artifact digest, subject digest, package type, release version, release checksum. | Package lifecycle and package-set activation. | Included in production package set manifest. | `100` | Resolved. |
| `package_type` | Canonical package behavior category used for activation policy resolution. | Package name, module name, artifact filename, repository path, version string, broad package label, grouping bundle, or runtime alias. | Confirmed `100.PackageType` token. | Selects package type policy before activation. | `100` | Resolved owner-routed; active policy rows still gate activation. |
| `package_type_policy` | Row-set-controlled activation policy selected for a package type. | Broad package category prose, `policy_bundle`, grouping bundle, or implementation-local default. | `PackageTypePolicyRow`, `PackageTypePolicyRowSet`. | Selects package evidence, compatibility, stage, rollback, health, quarantine, and emergency requirements. | `100` | Resolved as owner-routed behavior; active rows still gate use. |
| Production package set | Coherent activation unit for package runtime behavior. | Immutable package-set checksum. | Promotion, activation, rollback, quarantine, last-known-good. | Provides package roles for DAG stages and validation. | `100`, `030` | Resolved. |
| `last_known_good_package_set` | Verified rollback candidate after activation and health gates pass. | Recently activated package set by itself. | `LastKnownGoodPackageSet`, `LastKnownGoodHealthGate`. | Permits rollback eligibility only after health gates pass. | `100` | Resolved as owner-routed behavior. |
| `package_quarantine` | Immutable block record over package activation targets. | Artifact mutation, signature rewrite, or hidden denylist. | `PackageQuarantineRecord`, `QuarantineScopePolicy`. | Blocks dependent activation or rollback according to target kind and scope. | `100` | Resolved as owner-routed behavior. |
| `emergency_package_override` | Bounded emergency action record. | Trust bypass or unverified activation. | `EmergencyPackageOverrideRecord`. | Allows quarantine, retirement, abort, verified rollback, or deprecation-window extension only. | `100` | Resolved as owner-routed behavior. |
| Analysis finding | Non-authoritative analysis output. | Finding identity as defined by analysis owner. | Analysis/workflow lifecycle only. | May reference graph query and evidence but cannot mutate facts or graph state. | `130` | Resolved. |
| Registry governance artifact | Governance metadata artifact. | Registry artifact checksum and schema refs. | Registry lifecycle. | May annotate owners/classifications/policies but not define fact authority. | `130` | Resolved. |
| Reachability analysis artifact | Future inactive reachability analysis output. | Future query/result/artifact checksum when activated. | Inactive deferred until promoted. | Must not emit MVP facts, graph edges, or graph properties. | `200` | Inactive deferred. |

## 12. Entity relationships

Section 12 defines domain relationship families among Cadastre concepts, records, and evidence chains. Section 9 defines bounded-context relationship edges among owner contexts. A relationship family in Section 12 must not be used to infer a runtime dependency, handoff, projection, query, validation, activation, or telemetry permission unless Section 9 names the context-map edge or an owner spec exports the required handoff.

| Relationship family | Domain meaning | Authoritative representation | Direction rule | Evidence requirement | Primary owner | Non-implication rule |
| --- | --- | --- | --- | --- | --- | --- |
| Supplier-to-feed | External supplier provides lakehouse-resident feed data. | `RawSupplierProfile`, `LakehouseFeedProfile`, `RawFeedManifest`. | Supplier to feed. | Feed profile, manifest, supplier metadata. | `020`, `010` | Does not imply source completeness, source authority, or production source-call permission. |
| Feed-to-raw-record | Declared feed input is imported into preserved raw records. | `RawRecordImportRun`, `RawRecord`. | Feed input to raw record. | Lakehouse target refs, manifest refs, payload hashes, import profile. | `020`, `040` | Does not imply parsing success, source absence, or canonical truth. |
| Raw-to-silver | Raw evidence is parsed and normalized into a silver observation. | `ParseRawBatch`, `NormalizeObservation`, `CadastreSilverObservation`. | Raw record to silver observation. | Raw refs, parser profile, mapping bundle, external schema profile. | `050`, `040` | Does not imply identity merge, source authority, or gold fact. |
| Silver-to-identity-evidence | Silver observations can provide typed identity evidence. | `IdentityEvidenceItem`, `Identifier`, `IdentifierScope`. | Observation/identifier to evidence item. | Evidence class, scope, valid/known time, evidence refs. | `070`, `040` | Does not imply canonical identity without `IdentityDecision`. |
| Source-asset-to-canonical-entity | Resolver decides relationship between source-scoped asset evidence and product-owned canonical entity. | `IdentityDecision`, `ResolverExplanation`. | Decision row defines direction and effect. | Active resolver profile, evidence items, blockers, version manifest. | `070` | Does not imply graph mutation or non-identity source authority. |
| Silver/evidence-to-gold-fact | Authorized evidence supports canonical bitemporal fact derivation. | `GoldFact`, `GoldFactChangeSet`, `TemporalObservationTimeResolution`. | Subject/predicate/object direction defined by fact type and derivation profile. | Source authority row, temporal resolution, evidence refs, coverage when required. | `060`, `080`, `040` | Does not imply graph edge until projection owner emits graph delta. |
| Gold-fact-correction | Later knowledge changes fact assertion without in-place mutation. | `GoldFactChangeSet`, known-time closure, new fact rows. | Prior fact to correction/new fact. | Correction policy, old/new snapshot refs, authority, temporal resolution. | `080` | Does not delete prior evidence or rewrite history. |
| Gold-fact-to-graph-delta | Graph projection maps authoritative facts to graph nodes/edges/properties. | `GraphProjectionProfile`, `GraphDeltaSet`. | Edge direction from `GraphEdgeSemantics`; node identity from graph projection profile. | Gold facts, identity decisions, projection profile, graph property evidence policy. | `090` | Graph edge does not imply identity, authority, reachability, risk, remediation, or completeness unless owner row grants it. |
| Graph-delta-to-read-model | Graph apply materializes deltas in graph serving backend. | `ApplyGraphDelta`, `GraphApplyResult`, `DerivedViewState`. | Delta apply order from graph apply profile. | Backend preflight, schema fingerprint, idempotency key, apply result. | `090` | Backend state is not system of record and cannot repair truth. |
| Evidence drillback | Served outputs link back to supporting gold, silver, raw, and evidence metadata. | `EvidenceDrillbackResponse`, `EvidenceRef`. | Output to evidence chain. | Caller permission and lineage refs. | `110`, `040` | Raw payload exposure is not implied; redaction may apply. |
| Package-to-stage | Package release executes only in permitted DAG stage roles. | `PackageStageBinding`, `ProductionPackageSetManifest`, `ProcessingStageDAG`. | Package set to stage binding to output class. | Package activation evidence, validation matrix, package developer contract. | `100`, `030` | Package artifact does not authorize production output by itself. |
| Analysis-to-fact/graph | Analysis may read declared models and emit findings or metrics. | `AnalysisRuleBundle`, `AnalysisFinding`, `AnalysisMetric`. | Query or rule defined by analysis owner. | Rule compatibility matrix, graph profile, validation. | `130` | Analysis output does not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority. |
| Threat-intel-to-observation/fact | Threat-intel enrichment may provide context. | `ThreatIntelEnrichmentRecord`, `ThreatIntelEnrichmentProfile`. | Enrichment target defined by profile. | Artifact refs, distribution policy, redaction. | `130` | Does not imply identity, source authority, gold fact, graph edge, absence, cleanup, retraction, coverage, or watermark authority. |
| Registry-to-domain-artifact | Registry governance may classify or annotate artifacts. | `RegistryArtifactGovernance`, `RegistryClassificationPolicy`. | Governance artifact to governed artifact. | Checksums, schema, owner, lifecycle. | `130` | Registry labels do not define source authority, graph semantics, or production approval by themselves. |
| Future reachability | Deferred analysis result may relate modeled context to reachability claim. | `ReachabilityResult`, `ReachabilityClaimPolicy`, inactive. | Future query and result policy. | Future complete evidence, authority, completeness, capability matrix. | `200` | No active MVP graph/gold implication. |

## 13. Graph concepts

Cadastre source documents establish graph-domain concepts. The graph is a derived serving model, not the system of record. Graph applicability is active for graph projection and serving, but theoretical reachability is inactive and deferred.

Graph-domain boundary rules:

- A graph node is a projected read-model object. It is not automatically a `CanonicalEntity`, `SourceAsset`, source-native asset, or external graph object.
- A graph edge is a projected read-model relationship governed by `GraphEdgeSemantics`. It is not automatically a source fact, gold fact, identity proof, risk proof, reachability proof, remediation proof, or completeness proof.
- Graph projection owns graph-delta construction. Graph apply owns backend materialization. Core facts remain owned by `040`, `060`, and `080`.
- Graph traversal vocabulary must come from `GraphTraversalClass`. Parallel traversal eligibility labels are forbidden.
- Backend-generated node, edge, relationship, vertex, document, element, transaction, shard, or native cursor IDs are forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity.

| Graph concept | Domain interpretation | Not this | Owner spec | Default implication |
| --- | --- | --- | --- | --- |
| Graph read model | Derived serving substrate rebuilt from authoritative records and graph deltas. | System of record. | `090`, `010` | Derived projection only. |
| Graph projection profile | Contract mapping facts/rules to graph nodes, edges, properties, expiry scopes, and no-op outcomes. | Backend schema or query implementation. | `090` | Projection authority only. |
| Graph node delta | Persisted projected node change with Cadastre-owned deterministic ID. | Canonical entity by itself or backend node ID. | `040`, `090` | No identity implication beyond projection profile. |
| Graph edge delta | Persisted projected edge change with Cadastre-owned deterministic ID. | Source fact, gold fact, or backend relationship ID. | `040`, `090` | No relationship implication beyond `GraphEdgeSemantics`. |
| Graph edge semantics | Edge-type contract for meaning, direction, evidence, temporal policy, confidence, traversal class, non-implications. | Backend relationship label or external taxonomy edge. | `090` | Only declared edge semantics apply. |
| Graph traversal class | Sole traversal eligibility authority. | Pathfinding role, external traversability, reachability, risk, privilege. | `090` | Empty allowed traversal classes means no path traversal. |
| Graph backend profile | Activation and preflight contract for a backend product/driver/dialect/topology/storage mode. | Backend product selection or domain truth. | `090` | Backend may serve only after preflight. |
| Backend schema fingerprint | Deterministic backend schema inventory checksum. | Proof of fact correctness. | `090` | Schema readiness only. |
| Graph apply result | Persisted result of applying a graph delta set. | Fact derivation result or source completeness. | `090` | Derived-view state candidate only. |
| Derived view state | Served dataset/snapshot version and graph apply/rebuild reference. | Source freshness or fact validity. | `090`, `110` | Indicates graph serving state and lag only. |
| Graph rebuild manifest | Evidence that graph serving state was rebuilt from authoritative inputs and active profiles. | Backend import success or source truth. | `090`, `080` | Rebuild evidence only. |
| Structural global node | Bounded future graph node concept for global structural objects. | Implicit `Internet`, `Everyone`, wildcard identity, complete exposure, or universal reachability. | `090` | Default MVP emits none. |
| Theoretical reachability edge | Deferred future graph edge or fact candidate. | Active MVP graph output. | `200`, `090` | Forbidden in MVP. |

## 14. Lifecycle concepts and state vocabulary

Lifecycle concepts in root `domain.md` identify the domain meaning and owner of state vocabularies. Full transition tables remain owner-specific.

| Lifecycle concept | Applies to | State vocabulary owner | Domain meaning | Must not imply |
| --- | --- | --- | --- | --- |
| Spec status | Cadastre documentation artifacts. | `000` | Whether a document may drive implementation or is draft/candidate/deferred/archive material. | Runtime behavior unless status is `authoritative` and owner spec grants behavior. |
| Lifecycle status | Packages, profiles, policies, lifecycle machines, validation artifacts, activation-controlled records. | `030`, `100`, specific owner specs. | Production eligibility state shared across activation-controlled artifacts. | Transition behavior without lifecycle machine. |
| Gold fact assertion state | `GoldFact` rows. | `040`, corrections in `080`. | Current assertion meaning of a bitemporal fact; transition behavior is owned by `080`. | UI state, graph state, source completeness, identity decision. |
| Lakehouse read receipt state | `LakehouseReadCompletenessReceipt`. | `020`. | Whether Cadastre read a declared lakehouse feed target completely or failed/partially read it. | Source absence or source completeness by itself. |
| Completeness decision | Source/feed completeness evaluation. | `060`. | Whether source-level completeness may be considered under active profiles. | Source authority, identity, graph truth, or absence without additional gates. |
| API/source/export state labels | API, UI, evidence drillback, compliance export, audit export, analysis output. | `110`, source domain owners. | User-facing state separation. | Authorized negative facts or compliance pass/fail unless owner output exists. |
| Future reachability result state | Inactive future reachability results. | `200`. | Deferred analysis state for modeled reachability. | Active graph or gold output before promotion. |

Lifecycle runtime behavior is owner-routed. `domain.md` names machine IDs and domain meaning only; it must not restate transition matrices.

| Lifecycle machine | Domain meaning | Owner spec | Runtime table location |
| --- | --- | --- | --- |
| `030.ActivationControlledArtifactLifecycleMachine.v1` | Generic artifact activation, canary, deprecation, retirement, quarantine, and idempotent no-op behavior. | `030` plus artifact owner guard rows | `030`; owner guard rows in artifact owner spec |
| `030.ProductionRunExecutionLifecycleMachine.v1` | Production run preflight, lock, run, failure, completion, and abort. | `030` | `030` |
| `030.StageExecutionLifecycleMachine.v1` | Stage readiness, running, retry, success, no-op, isolated or non-isolated failure, and skip. | `030` plus stage owner event derivation | `030`; feed derivation in `020` |
| `090.GraphApplyLifecycleMachine.v1` | Graph apply preflight, apply, partial failure, resume, and idempotent reapply. | `090` | `090` |
| `100.PackageSetActivationLifecycleMachine.v1` | Package-set promotion, shadow, canary, activation, failure, rollback, last-known-good, deprecation, and quarantine. | `100` | `100` |
| `120.ValidationAcceptanceLifecycleMachine.v1` | Validation acceptance status transitions. | `120` | `120` |
| `070.IdentityReviewCaseStateMachine.v1` | Manual identity review states and terminal decision routing. | `070` | `070`; closed by `070.IdentityReviewCaseStateMachineBinding` after identity closure validation passes |

| State token | Applies to | Meaning | Terminal | Default transition authority | Owner |
| --- | --- | --- | --- | --- | --- |
| `draft` | Spec status | Authoring text that is not review-complete. | No. | Index promotion process. | `000` |
| `candidate` | Spec status | Candidate NLSpec text that may be reviewed and validated. | No. | Index promotion process. | `000` |
| `authoritative` | Spec status | Accepted NLSpec text whose validation criteria pass and whose registry row marks it active. | Yes for owned contracts. | Index promotion process. | `000` |
| `superseded` | Spec status | Replaced by newer document or version. | Yes for replaced version. | Index promotion process. | `000` |
| `archived` | Spec status | Historical reference only. | Yes. | Index promotion process. | `000` |
| `inactive_deferred` | Spec status | Future-domain candidate with no active MVP effect. | No active effect. | Promotion process. | `000`, `200` |
| `draft` | Lifecycle status | Cannot affect production output. | No. | Declared lifecycle machine. | `030` |
| `validated` | Lifecycle status | May be used only in validation or canary gates. | No. | Declared lifecycle machine. | `030` |
| `canary` | Lifecycle status | May produce canary or shadow output only. | No. | Declared lifecycle machine. | `030` |
| `active` | Lifecycle status | May affect production output when dependencies are active. | No. | Declared lifecycle machine. | `030` |
| `deprecated` | Lifecycle status | May remain in existing output but must not be newly selected unless policy permits. | No. | Declared lifecycle machine. | `030` |
| `retired` | Lifecycle status | Must not execute or affect new output. | Yes by default. | Declared lifecycle machine. | `030` |
| `quarantined` | Lifecycle status | Must not execute and must block dependent package-set activation. | Yes until owner action. | Declared lifecycle machine. | `030`, `100` |
| `active` | Gold assertion state | Fact is asserted current for query interval. | No. | Gold correction policy. | `040`, `080` |
| `stale` | Gold assertion state | Fact is retained but stale under active staleness policy. | No. | Source staleness and correction policies. | `040`, `060`, `080` |
| `retracted` | Gold assertion state | Fact has been explicitly retracted by authorized evidence. | Owner-defined. | Gold correction policy. | `040`, `080` |
| `superseded` | Gold assertion state | Fact was replaced by correction or stronger evidence. | Owner-defined. | Gold correction policy. | `040`, `080` |
| `conflicted` | Gold assertion state | Concurrent evidence prevents a single resolved assertion. | No. | Gold derivation/correction policy. | `040`, `080` |
| `unknown` | Gold assertion state | Evidence exists but does not authorize presence or absence. | No. | Authority/completeness/correction policy. | `040`, `060`, `080` |
| `no_op` | Gold assertion state | Candidate derivation intentionally emitted no fact. | Yes for candidate. | Gold derivation policy. | `040`, `080` |
| `read_complete` | Read receipt | Declared lakehouse target was read completely. | Yes for receipt. | Read algorithm. | `020` |
| `read_partial_known_gap` | Read receipt | Known row/object/partition/range not read. | Yes for receipt. | Read algorithm. | `020` |
| `read_partial_unknown_gap` | Read receipt | Target may be incomplete with unknown missing scope. | Yes for receipt. | Read algorithm. | `020` |
| `read_unavailable` | Read receipt | Declared lakehouse target could not be read. | Yes for receipt. | Read algorithm. | `020` |
| `schema_unavailable` | Read receipt | Required schema ref unavailable. | Yes for receipt. | Read algorithm. | `020` |
| `manifest_invalid` | Read receipt | Manifest checksum/count/shape failed validation. | Yes for receipt. | Read algorithm. | `020` |
| `complete` | Completeness decision | May be considered only when profile permits. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `empty_complete` | Completeness decision | Empty scope may be considered only when profile permits. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `partial_known_gap` | Completeness decision | Known gap blocks absence, cleanup, retraction. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `partial_unknown_gap` | Completeness decision | Unknown gap blocks absence, cleanup, retraction. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `source_unavailable` | Completeness or UI/source label | Source/feed evidence unavailable. | Decision-specific. | `060` or `110` context. | `060`, `110` |
| `scope_unavailable` | Completeness or UI/source label | Required scope unavailable. | Decision-specific. | `060` or `110` context. | `060`, `110` |
| `permission_limited` | Omission/completeness/UI label | Visibility did not permit observation. | Decision-specific. | `040`, `060`, or `110` context. | `040`, `060`, `110` |
| `not_attempted` | Completeness decision | Required source/feed scope was not attempted. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `not_authoritative_for_absence` | Completeness decision | Source/scope lacks absence authority. | Decision-specific. | `EvaluateLakehouseFeedCompleteness`. | `060` |
| `not_applicable` | Omission/completeness/UI label | Active profile states field/fact/scope does not apply. | Decision-specific. | Owner policy. | `040`, `060`, `110` |
| `reachable_within_model` | Future reachability result | Positive only inside declared model scope. | Result-specific. | Future activation. | `200` inactive |
| `not_reachable_within_complete_model` | Future reachability result | Negative only with complete evidence and solver capability. | Result-specific. | Future activation. | `200` inactive |
| `unknown_partial` | Future reachability result | Missing/stale/partial/permission-limited/out-of-scope evidence. | Result-specific. | Future activation. | `200` inactive |
| `unsupported` | Omission/future reachability | Parser/source/solver cannot represent field or relevant component. | Context-specific. | Owner policy. | `040`, `200` |
| `ambiguous` | API/future reachability | Multiple unresolved interpretations exist. | Context-specific. | Owner policy. | `110`, `200` |
| `conditional` | Future reachability result | Result holds only under named conditions. | Result-specific. | Future activation. | `200` inactive |
| `diagnostic_only` | Future reachability result | Provider/probe output cannot support claims. | Yes for claim. | Future activation. | `200` inactive |

## 15. Identity and naming rules

Stable IDs must not be inferred from labels, display text, backend-generated IDs, storage keys, route names, file names, package names, external-system identifiers, or source-native IDs unless an owner spec explicitly permits the exact use.

Resolver artifact identity is activation-artifact identity. It is not canonical entity identity, source-native identity, graph identity, package identity, storage identity, or display identity.

| Name or identifier class | Domain target | Stable use | Not this | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| Canonical domain identity | Product-owned canonical entity or fact identity. | Stable reference inside Cadastre outputs and evidence chains. | Source-native ID, graph backend ID, display label. | `040`, `070`, `080` | Resolved. |
| Source-native identity | Identifier emitted by external source or supplier feed. | Evidence input scoped by source/feed and resolver/authority policy. | Cadastre canonical identity by default. | `040`, `070`, `020` | Resolved. |
| Source/feed scope key | Scope boundary for feed, supplier, source dataset, resolver, authority, completeness, coverage. | Defines where source evidence is valid and comparable. | Global identity by itself. | `020`, `060`, `070` | Resolved. |
| Raw record ID | `RawRecord`. | Deterministic raw evidence reference computed by `040.ComputeRawRecordId`; `020` supplies feed, target, manifest, source dataset, scope, supplier identity, payload hash, and import-profile inputs. | Source completeness, canonical identity, or a `domain.md` restatement of raw ID input order. | `040`, `020` | Resolved; field schema and ID policy are owned by `040`, and feed/import input contribution is owned by `020`. |
| Silver observation ID | `CadastreSilverObservation`. | Deterministic normalized observation reference. | Gold fact ID or identity decision ID. | `040`, `050` | Resolved. |
| Canonical entity ID | `CanonicalEntity`. | Product-owned identity selected by resolver policy. | Source asset ID, hostname, IP, DNS name, graph node ID. | `040`, `070` | Resolved. |
| Source asset ID | `SourceAsset`. | Source/feed-scoped asset reference. | Product canonical entity ID. | `040`, `070` | Resolved. |
| Identifier ID | `Identifier`. | Typed normalized observed identifier reference. | Identity decision. | `040`, `070` | Resolved. |
| Identity decision ID | `IdentityDecision`. | Resolver output and replay/audit reference. | Identifier equality or reviewer note. | `070` | Resolved. |
| Gold fact ID | `GoldFact`. | Canonical fact identity across time/evidence/authority state. | Silver observation ID or graph edge ID. | `040`, `080` | Resolved. |
| Evidence ref ID | `EvidenceRef`. | Drillback and evidence role reference. | Raw payload bytes. | `040`, `110` | Resolved. |
| Graph node/edge ID | Graph read-model object. | Cadastre-owned deterministic graph identity and response identity. | Backend internal ID, vertex ID, relationship ID, document ID. | `090`, `040` | Resolved. |
| Graph backend internal ID | Backend storage object. | Backend-local execution evidence only when captured as backend evidence. | Cadastre ID, selector, evidence ref, replay key, pagination identity. | `090` | Resolved as forbidden for Cadastre IDs. |
| Display label | User-facing text. | Presentation only. | Stable identity or behavior selector. | `110`, owner concept spec | Resolved. |
| External schema identifier | OCSF/CIM/external schema category, class, activity, field, enum, profile, extension. | Mapping and validation through `ExternalSchemaProfile` and mapping rows. | Cadastre identity, fact, authority, completeness, graph semantics. | `050` | Resolved. |
| Package artifact digest | Immutable package artifact subject. | Artifact acquisition, trust verification, release manifest. | Production activation target alone. | `100` | Resolved. |
| Production package set checksum | Package-set activation target. | Production activation, rollback, last-known-good reference. | Single package version, branch, tag, deployment label. | `100` | Resolved. |
| Version manifest checksum | Replay/output-affecting input set. | Replay equivalence and audit reference. | Runtime log or build number. | `030`, `080` | Resolved. |
| Private source binding name | Concrete vendor/product/source route binding in private implementation repository. | Private mapping to public feed contracts. | Public canonical domain term. | `010`, `020` | Resolved as private-only. |
| User-facing name | UI/API label for a domain object. | Display and filtering when owner spec maps it. | Stable ID or domain definition. | `110` | Resolved. |

## 16. Domain invariants

| ID | Invariant | Owner | Acceptance evidence |
| --- | --- | --- | --- |
| `DOM-INV-001` | Cadastre production must be lakehouse-fed and must not call configured enterprise source systems directly. | `010`, `020` | Production source API call fails before output with `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| `DOM-INV-002` | Missing lakehouse rows, missing objects, or successful feed reads must not imply source absence. | `020`, `060` | Absence derivation requires active completeness and authority gates. |
| `DOM-INV-003` | Source completeness must not imply source authority. | `060` | Exact `SourceAuthorityProfileRow` is required for gold derivation and absence authority. |
| `DOM-INV-004` | Source authority, source completeness, source scope, and public resolver row presence must not imply identity creation, attachment, or merge authority. | `060`, `070`, `010` | Identity creation, attachment, and merge require active `ResolverProfileRow`, durable evidence where required, exact scope, no hard blockers, and passing identity closure validation. |
| `DOM-INV-005` | Identity evidence must be materialized before resolver output. | `070` | Resolver run rejects uncovered or under-scoped evidence. |
| `DOM-INV-006` | Weak-only, selector-only, graph-key-only, source-native-merge-history-only, semantic-overlay-only, and mapped-target-only evidence must never create, attach, or merge canonical identity. | `070` | Required identity closure validation cases fail creation, attachment, and merge. |
| `DOM-INV-007` | Parser and normalization stages must not assert canonical truth, absence, identity, gold facts, graph deltas, health state, watermark state, or package state. | `050`, `030` | Forbidden output class fails with `FORBIDDEN_STAGE_OUTPUT`. |
| `DOM-INV-008` | Every gold fact must carry valid time, known time, assertion state, evidence, confidence, and authority references. | `040`, `080` | Core record validation and gold derivation validation pass. |
| `DOM-INV-009` | Gold corrections must not mutate existing `GoldFact` rows in place. | `080` | Correction emits known-time closures, inserts, interval splits, conflicts, no-ops, or errors. |
| `DOM-INV-010` | Graph state must remain a rebuildable derived read model. | `090`, `010` | Graph rebuild manifest and index consistency check pass from authoritative inputs. |
| `DOM-INV-011` | Backend-generated graph IDs must not become Cadastre IDs, selectors, evidence refs, replay keys, response IDs, or pagination identity. | `090`, `040` | Backend internal ID negative fixture fails. |
| `DOM-INV-012` | Every active graph edge type must have exactly one `GraphEdgeSemantics` row. | `090` | Graph profile activation rejects missing or duplicate edge semantics rows. |
| `DOM-INV-013` | OCSF/CIM/external schema fields must not define Cadastre identity, authority, completeness, gold facts, graph edges, or temporal semantics by default. | `050`, `060`, `090` | External schema mapping validations and forbidden-inference tests pass. |
| `DOM-INV-014` | Analysis, enrichment, lineage, and registry records must not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active spec grants a named interface. | `130` | Analysis/enrichment negative mutation cases fail. |
| `DOM-INV-015` | Production package runtime behavior may activate only through immutable `ProductionPackageSetManifest`. | `100` | Single artifact/version/signature/SBOM/provenance activation attempt fails. |
| `DOM-INV-016` | Production output without a complete immutable `VersionManifest` must fail. | `030` | Missing manifest negative fixture fails with `VERSION_MANIFEST_INCOMPLETE`. |
| `DOM-INV-017` | Runtime randomness, wall-clock reads, generated IDs, unordered iteration, external calls, or backend-discovered values must not affect production output unless captured by `DeterministicSideEffectRecord`. | `080`, `030` | Replay validation rejects unrecorded nondeterminism. |
| `DOM-INV-018` | Future reachability terms must remain inactive and graph-neutral until promoted. | `200`, `090` | MVP graph profiles cannot emit `has_theoretical_reachability` or equivalent properties. |
| `DOM-INV-019` | Root `domain.md` must not define runtime behavior owned by active owner specs. | `domain.md`, `000` | Review confirms every root requirement is vocabulary, boundary, or owner-routing only. |
| `DOM-INV-020` | Template-specific Cartulary content must not become Cadastre vocabulary or requirements. | `domain.md` | Generated document contains no template-specific imported model except the exclusion note. |
| `DOM-INV-021` | Every runtime-affecting cross-context claim must be routed by a Section 9 context-map edge or by an exact owner-exported contract. | `domain.md`, named owner spec | Context-map validation rejects unlisted runtime-affecting dependencies. |
| `DOM-INV-022` | Missing, inactive, stale, ambiguous, checksum-invalid, out-of-scope, or owner-uncovered crossing artifacts default to `no_implicit_effect`. | `domain.md`, named owner spec | Negative validation proves no fallback authority or mutation occurs. |
| `DOM-INV-023` | Context-map edges must not create runtime interfaces, schemas, errors, algorithms, lifecycle transitions, graph mutation behavior, package activation behavior, validation harness execution, API response behavior, or telemetry export behavior not exported by owner specs. | `domain.md`, `000`, `120` | Document validation rejects runtime restatement. |
| `DOM-INV-024` | Observability and runtime telemetry must remain diagnostic and must not become domain truth, source authority, identity evidence, graph authority, package activation, audit persistence, or replay input. | `140`, `110`, `010` | Observability negative validation rows pass. |

## 17. Defaults and omitted cases with domain significance

| Domain area | Default or omitted-case behavior | Owner | If omitted source evidence is missing |
| --- | --- | --- | --- |
| Root domain authority | Default: `docs/nlspec/domain.md` is a candidate vocabulary document and may not drive implementation until marked `authoritative` by `000`. | `000`, `docs/nlspec/domain.md` | Use for vocabulary routing only while candidate. |
| Production input boundary | Default: enterprise source calls are forbidden; production reads declared lakehouse feeds and supplier metadata only. | `010`, `020` | Fail before output with owner-specific error. |
| Source evidence absence | Default: missing lakehouse row, source field, object, partition, or feed item means no stronger interpretation than unknown/not-provided unless owner profile grants more. | `020`, `040`, `060` | Emit explicit unknown/not-provided/unsafe state, not absence. |
| Source completeness receipt | Default: read completeness receipt is necessary but not sufficient for source absence, cleanup, retraction, graph expiry, or watermark advancement. | `020`, `060` | Absence/retraction/cleanup/watermark forbidden. |
| Source authority row | Default: no broad source-category fallback when exact authority row is missing. | `060` | Gold derivation or absence authority must reject. |
| Coverage-sensitive fact | Default: require `CoverageDimensionProfile` and current `CoverageAssertion`. | `060` | Emit deterministic unknown/error; no negative claim. |
| Omission state | Default: optionality, nullable schema, external field absence, CIM/OCSF absence, or source row absence cannot decide omission semantics. | `040` | Use explicit omission state or reject unresolved field interpretation. |
| External schema field | Default: external schema fields are mapping/validation terms only. | `050` | Do not derive identity/fact/graph/authority without owner mapping. |
| Identity weak evidence | Default: IP-only, hostname-only, DNS-only, PTR-only, flow-only, graph-key-only, source-native-merge-only, semantic-overlay-only, mapped-target-only evidence has no auto-merge authority. | `070` | Emit rejected, candidate, conflict, no-decision, or deterministic error per resolver profile. |
| Temporal fallback | Default: implicit current-time fallback is forbidden. | `080` | Emit temporal error or no candidate fact. |
| Gold correction | Default: corrections are append-only knowledge transitions. | `080` | Missing correction policy or snapshot refs blocks correction. |
| Graph traversal | Default: empty `allowed_traversal_classes` returns no paths rather than traversing all edges. | `090` | Return no paths or validation error per query contract, never broad traversal. |
| Graph backend IDs | Default: backend internal IDs are forbidden as Cadastre IDs/selectors. | `090` | Reject response, selector, replay key, or pagination identity. |
| Derived-view lag | Default: compliance, audit, and production rule queries reject stale graph state. | `090`, `110` | Return `DERIVED_VIEW_LAG_ERROR` or owner-specific stale-state behavior. |
| Package activation | Default: candidate activation failure keeps current active package set and writes no candidate production output. | `100` | Emit `PackageActivationFailureEvent`. |
| Analysis output | Default: analysis/enrichment/lineage/registry outputs are non-authoritative. | `130` | No mutation of facts, graph state, completeness, identity, package state, source authority, or watermarks. |
| Future reachability | Default: graph effect is `none`; active MVP cannot emit theoretical reachability. | `200`, `090` | Fail/no-op with future-scope boundary error. |
| TODO values | Default: downstream implementation must not resolve `TODO:` by inference. | `domain.md`, `120` | Block affected implementation or document update until owner decision exists. |
| Context-map edge omitted | Default: unlisted runtime-affecting dependency is not permitted. | `domain.md`, `120` | Add `TODO:` or fail validation; do not infer dependency. |
| Context-map relationship type omitted | Default: edge row is invalid. | `domain.md`, `120` | Validation fails before the edge can be used for owner routing. |
| Crossing artifact omitted | Default: `no_implicit_effect`. | Named owner spec | No authority, mutation, projection, activation, validation pass, or telemetry effect may be inferred. |
| Behavior owner omitted | Default: edge is non-runtime only or invalid. | `domain.md`, `000`, `120` | Runtime-affecting claim must fail document validation. |
| Missing behavior owner for subject, object, or evidence refs | Default: `no_implicit_effect`. | `040`, owner spec, `120` | No subject ref, object value, evidence ref, authority, mutation, graph projection, package activation, or validation pass may be inferred. |
| Validation route omitted | Default: `blocked_validation`. | `120` | Section 25 ledger must carry a blocker until validation rows exist. |

## 18. Boundaries to adjacent specs

### 18.1 Domain model versus data model

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Data model | Record shapes, scalar rules, canonical JSON, field-level omission states, primitive record identities. | Term definitions, concept distinctions, and owner lookup. | Treating field schema as domain definition when root or owner defines a concept. |

### 18.2 Domain model versus graph model

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Graph model | Projection rows, graph delta construction, graph edge semantics, traversal classes, backend profiles, query/apply/rebuild behavior. | Graph-versus-domain distinction and non-authority rules. | Treating backend graph labels, relationship types, internal IDs, traversal order, or graph edges as domain truth. |

### 18.3 Domain model versus workflow specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Processing/lifecycle workflow | DAG execution, permitted outputs, lifecycle machines, run locks, version manifests. | Stage/lifecycle vocabulary boundaries and owner routing. | Treating UI workflow state, process status, or lifecycle diagram as lifecycle authority. |

### 18.4 Domain model versus persistence or lakehouse specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Lakehouse/persistence | Feed profiles, raw imports, table snapshots, commits, dataset versions, retention, maintenance. | System-of-record vocabulary and table-state non-substitution rule. | Treating lakehouse product defaults, table names, snapshot times, object keys, or catalog names as fact authority. |

### 18.5 Domain model versus package activation specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Package activation | Package type policy, package release manifests, package-set activation, trust, repository evidence, compatibility, health gates, rollback, quarantine, and emergency override. | Package vocabulary routing and package-versus-domain distinction. | Treating package name, module name, artifact filename, repository ref, dependency lock, SBOM existence, signature scalar, or emergency override as activation authority. |

### 18.6 Domain model versus API/UI specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| API/UI | Request/response shapes, state labels, redaction, authorization, health, errors, audit. | Stable term mapping and label-versus-identity rule. | Treating UI labels, displayed order, page tokens, or labels as stable domain identity. |

### 18.7 Domain model versus validation and fixture specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Validation/fixtures | Validation matrices, fixtures, golden corpus, acceptance reports, validation acceptance. | Binary root-domain acceptance criteria. | Treating test fixture shape as domain behavior owner or using validation artifact to invent behavior. |

### 18.8 Domain model versus package and implementation details

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Packages/implementation | Package releases, package-set activation, trust, code modules, local functions, package names, build and deployment mechanisms. | Package-domain boundary and implementation-mapping exception. | Treating module/package/function/table/route names as domain definitions or production activation authority. |

### 18.9 Cadastre domain concepts versus external schema or external-system concepts

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| External schemas/systems | External terminology, source-native identifiers, provider metadata, schema fields, analyzer outputs, lineage facets, taxonomies, registry labels. | Vendor-neutral Cadastre interpretation and non-authority rule. | Treating external concepts as Cadastre identity, authority, completeness, graph, persistence, lifecycle, validation, or production approval authority by default. |

## 19. Mapping tables

This section defines cross-representation mappings not already defined in Section 5. The domain concept to primary owner spec mapping is defined once in Section 5 and must not be duplicated here.

### 19.1 Domain entity to graph concept

| Domain entity | Graph concept | Mapping rule | Owner | Status |
| --- | --- | --- | --- | --- |
| `CanonicalEntity` | Graph node candidate. | May project only through active `GraphProjectionProfile` and graph node delta rules. | `090`, `040` | Resolved. |
| `SourceAsset` | Graph node or property candidate. | May project only when projection profile declares source-asset representation. | `090`, `070` | Owner mapping required per graph profile. |
| `Identifier` | Graph property or node candidate. | Must not become identity or graph selector unless projection and selector policy permit. | `090`, `070` | Resolved boundary. |
| `GoldFact` | Graph node, edge, or property source. | May project only through active graph projection row and `GraphEdgeSemantics` when edge. | `090`, `080` | Resolved. |
| `EvidenceRef` | Graph property or drillback ref. | May be exposed only through evidence and redaction policy. | `090`, `110`, `040` | Resolved. |
| `GoldFact.subject_ref` | Graph endpoint or graph property source only after owner routing. | May project only when `040` shape, `070` identity eligibility, `080` predicate contract, and `090` projection rules permit it. | `040`, `070`, `080`, `090` | Vocabulary routed. |
| `GoldFact.object_value` | Graph endpoint or graph property source only after owner routing. | May project only when `040` shape, `080` predicate contract, `060` authority, and `090` projection rules permit it. | `040`, `060`, `080`, `090` | Vocabulary routed. |
| `EvidenceRef.artifact_id` | Drillback ref, not graph identity. | May be exposed only as checksummed evidence metadata; raw payload and backend IDs remain forbidden. | `040`, `110` | Vocabulary routed. |
| `AnalysisFinding` | Analysis result, not default graph node. | May appear in graph only if future owner mapping grants it. | `130`, `090` | No default graph authority. |
| `ReachabilityResult` | No active graph concept in MVP. | Must not project until `ReachabilityClaimPolicy` is active. | `200`, `090` | Inactive deferred. |

### 19.2 Domain relationship to graph edge family

| Domain relationship | Graph edge family | Required mapping | Owner | Status |
| --- | --- | --- | --- | --- |
| Observed network relationship | `observed_connection`. | MVP active graph relationship output is limited to observed network relationships routed through `observed_connection`; runtime behavior, edge semantics, direction, traversal, properties, and output eligibility are owned by `090`. | `090`, `050`, `080` | Owner-routed for MVP; future reachability remains inactive/deferred. |
| Identity relationship | Owner-defined identity/projected relationship edge. | Must derive from identity decisions and/or gold facts through projection profile. | `070`, `090` | Requires graph profile row. |
| Asset ownership/containment/use | Owner-defined edge type. | Must derive from gold facts or declared synthetic structural rules. | `080`, `090` | Requires graph edge semantics row. |
| Threat-intel enrichment relationship | Analysis/enrichment edge only if declared. | Must not become gold/graph authority without separate contract. | `130`, `090` | No default authority. |
| Future reachability relationship | `has_theoretical_reachability` or equivalent. | Forbidden in MVP. Future activation requires `ReachabilityClaimPolicy`. | `200`, `090` | Inactive deferred. |
| MVP active graph edge-family routing | `observed_connection` only for MVP observed network relationship output. | Active runtime behavior must be read from `090.GraphEdgeSemanticsRegistry`, `090.MVPActiveGraphProjectionProfile`, and `120.GraphActiveProfileClosureValidationMatrix`, not from `domain.md`. | `090`, `120` | Routing resolved; authoritative closure remains validation-gated until `120` graph closure rows have non-TODO fixture and expected-output checksums. |
| GoldFact subject/object reference relationship | Owner-defined projection relationship only when predicate and projection rows permit it. | Runtime behavior must be read from `040` one-of registries, `070` reference eligibility, `080` predicate contracts, `060` authority, and `090` projection rows. | `040`, `060`, `070`, `080`, `090` | Vocabulary routed; no runtime behavior defined here. |
| Evidence artifact drillback relationship | Evidence drillback, not a graph edge by default. | Runtime behavior must be read from `040.EvidenceArtifactClassRegistry`, owner artifact handoffs, and `110` evidence/redaction rules. | `040`, `020`, `030`, `100`, `110`, `130` | Vocabulary routed; no runtime behavior defined here. |

### 19.3 Source evidence concept to Cadastre observation or fact concept

| Source evidence concept | Cadastre mapping | Must not map to | Owner | Status |
| --- | --- | --- | --- | --- |
| Source-native payload | `RawRecord` payload and metadata. | Gold fact or canonical identity. | `020`, `040` | Resolved. |
| Source-native event/object field | Parser output then `CadastreSilverObservation` field. | Authority or identity by field name alone. | `050`, `040` | Resolved. |
| Supplier completeness signal | `UpstreamCompletenessEvidence`, then profile evaluation. | Absence/cleanup/retraction by itself. | `020`, `060` | Resolved. |
| Feed profile or category closure row | Owner-routed activation-controlled feed configuration and closure evidence. | Source completeness, source authority, or absence by itself. | `020`, `060` | Resolved boundary; active owner rows and validation still required. |
| Cursor, CDC offset, heartbeat, queue ack, provenance closure, freshness artifact | `StageStateRecord`, diagnostic, or progress signal interpretation only. | Source authority, fact time, source absence, cleanup, graph mutation authority. | `030`, `060`, `080` | Resolved. |
| Identity-like source value | `Identifier` and `IdentityEvidenceItem` when scoped and typed. | Auto-merge by value equality alone. | `040`, `070` | Resolved. |
| Coverage evidence | `CoverageAssertion` when dimension profile permits. | Negative fact without authority. | `060` | Partial TODO catalog. |
| External analyzer output | Analysis artifact or raw evidence only unless owner grants authority. | Gold truth by itself. | `130`, `200`, `060` | Resolved boundary. |

### 19.4 External schema concept to Cadastre concept

| External schema concept | Cadastre concept | Default mapping | Owner | Status |
| --- | --- | --- | --- | --- |
| OCSF class/category/activity/type | `CadastreSilverObservation.normalized_fields` classification. | Requires exact mapping row or blocking `TODO:`. | `050` | Resolved with mapping rows required. |
| OCSF object/attribute | `normalized_fields` object/field. | Validated against active compiled profile. | `050` | Resolved. |
| OCSF `raw_data`/`unmapped` | Disabled unless bounded by base-event field policy. | No raw evidence authority by default. | `050`, `040` | Resolved. |
| OCSF observables/enrichments | Non-authoritative mapping/enrichment hints. | No graph node or identity by default. | `050`, `130`, `070` | Resolved. |
| CIM field | Lossy external projection field. | Output only through `CIMProjectionProfile` and `ProjectionLossManifest`. | `050` | Resolved. |
| Sigma-like rule | `AnalysisRule` inside `AnalysisRuleBundle`. | Read-only and compatibility-gated. | `130` | Resolved. |
| MISP/STIX-like indicator | `ThreatIntelEnrichmentRecord`. | Enrichment context only by default. | `130` | Resolved. |
| OpenLineage-like facet | `RunDatasetIOContract` or lineage metadata only when mapped. | Not evidence/completeness authority. | `130`, `020` | Resolved. |
| External graph taxonomy label/key | `GraphTaxonomyTranslationPolicy` input only. | Not identity, fact, or edge semantics by itself. | `090`, `070` | Resolved. |

### 19.5 Lifecycle state to domain meaning

| Lifecycle family | State tokens | Domain mapping | Owner | Status |
| --- | --- | --- | --- | --- |
| Spec status | `draft`, `candidate`, `authoritative`, `superseded`, `archived`, `inactive_deferred`, `active_standard` | Documentation authority and implementation eligibility. | `000` | Resolved. |
| Activation lifecycle | `draft`, `validated`, `canary`, `active`, `deprecated`, `retired`, `quarantined` | Production eligibility of activation-controlled records. | `030`, `100` | Resolved as shared vocabulary. |
| Gold assertion | `active`, `stale`, `retracted`, `superseded`, `conflicted`, `unknown`, `no_op` | Fact assertion meaning. | `040`, `080` | Resolved. |
| Future reachability result | `reachable_within_model`, `not_reachable_within_complete_model`, `unknown_partial`, `unsupported`, `ambiguous`, `conditional`, `diagnostic_only` | Inactive future result state. | `200` | Inactive deferred. |

### 19.6 User-facing label to stable domain term

| User-facing or API label | Stable domain term | Owner | Forbidden substitution | Status |
| --- | --- | --- | --- | --- |
| `source_stale` | Source staleness state. | `060`, `110` | Derived-view lag. | Resolved. |
| `derived_view_stale` | Graph/API derived view lag state. | `090`, `110` | Source staleness. | Resolved. |
| `unknown` | Unknown/unauthorized fact state or user-facing state. | `040`, `060`, `110` | Negative fact or compliance pass/fail. | Resolved. |
| `authorized_not_observed` | Absence or not-observed output authorized by owner policy. | `060`, `110` | Missing row by default. | Resolved. |
| `not_authoritative_for_absence` | Completeness decision that maps to `unknown` or owner-context detail. | `060`, `110` | Authorized negative fact. | Resolved as non-negative. |
| `permission_limited` | Visibility-limited observation/state. | `040`, `060`, `110` | Absence or not applicable. | Resolved. |
| `conflicted` | Concurrent evidence or fact conflict. | `040`, `080`, `110` | Error or resolved fact. | Resolved. |
| Display name/title | Owner-specific display label. | `110` plus concept owner | Stable ID. | Resolved. |

### 19.7 Private source binding to public vendor-neutral concept

| Private concern | Public vendor-neutral concept | Boundary | Owner | Status |
| --- | --- | --- | --- | --- |
| Concrete vendor/product/source route | `PrivateSourceFeedBinding` to `LakehouseFeedProfile`. | Private artifact only; must not appear in public canonical model. | `010`, `020` | Resolved. |
| Private observed source fields | `PrivateFeedSchemaInventory` and source schema import/mapping artifacts. | Private inventory may inform mappings; public outputs require declared profiles. | `050`, `020` | Resolved. |
| Private completeness capabilities | `PrivateCompletenessEvidenceInventory` and `UpstreamCompletenessEvidence`. | Supplier evidence must carry authority limit; active public completeness and category closure rows decide effect. | `020`, `060` | Resolved. |
| Private golden examples | Redacted `LakehouseFeedFixture` when public validation needs evidence. | Direct enterprise API recordings must not replace lakehouse fixtures. | `120`, `020` | Resolved. |

### 19.8 API label and outcome owner routing

These entries route vocabulary only. Runtime behavior remains owned by `110` and the source owner specs named in each row.

| Domain term | Primary owner | Required domain interpretation | Must not be used for |
| --- | --- | --- | --- |
| API state label | `110` | Caller-visible rendering of owner state. | Source authority, fact truth, compliance pass/fail, graph cleanup, remediation. |
| Export state label | `110` | Export-visible rendering of owner state. | Domain truth or owner-source outcome. |
| Endpoint outcome | `110` | Observable API success, empty, denial, stale, partial, conflict, ambiguity, redaction, pagination, and error behavior. | Owner algorithm behavior. |
| Response envelope | `110` | Common API response wrapper and checksum surface. | Core record schema or owner state semantics. |
| Error registry row | `110` plus owner fragments | Generated caller-visible error metadata from owner codes. | Error cause ownership. |
| Authorization decision | `110` | Runtime authorization result for an API/export operation. | Domain identity or source authority. |
| Redaction context | `110` | Runtime data-class redaction decision for caller-visible and audit output. | Record existence, fact truth, or source completeness. |

### 19.9 High-risk API state distinction table

| Concept | Owner and producer | Domain distinction | Forbidden substitution |
| --- | --- | --- | --- |
| `FactAbsenceOutcome` | Owned by `040`; produced by `060`. | Fact-level absence outcome. | Must not be treated as a caller-visible label without `110` mapping. |
| `GoldFact.assertion_state` | Owned by `040`; transitioned by `080`. | Assertion state of a core fact. | Must not be treated as source completeness or API state by itself. |
| `SourceStateLabel` | Owned by `110`. | API/export label for owner state rendering. | Must not be treated as domain truth, compliance pass/fail, graph cleanup, retraction, watermark, or remediation. |
| Feed category versus coverage domain token | Feed categories are lakehouse feed closure inputs owned by `020`; coverage-domain tokens are coverage authority inputs owned by `060`. | Treating `dns_record_set`, `DNS`, and `dns` as interchangeable without the `020` mapping table and `060` token validation. | |
| UI display label versus coverage domain token | UI display labels such as DNS and DHCP/IPAM may be rendered by API/UI layers but must not be accepted as runtime coverage-domain tokens. | Treating display labels, legacy spellings, source datasets, package names, or OCSF classes as `CoverageDomainToken` values. | |
| compliance pass/fail | Emitted only by the applicable owner output. | Owner-emitted result, not label-inferred. | Must not be inferred from `unknown`, `error`, `not_checked`, `not_applicable`, `conflicted`, or `ambiguous`. |

## 20. External-system and implementation boundaries

| External or adjacent concern | Domain boundary | Required language | Forbidden language |
| --- | --- | --- | --- |
| Enterprise source systems | External suppliers or feeds may provide evidence; Cadastre production must not collect directly. | “lakehouse-resident feed evidence,” “supplier-provided metadata.” | “Cadastre polls source API in production,” unless validation-only source exploration. |
| Lakehouse product/catalog | System-of-record substrate and table-state refs are Cadastre-governed. | “`LakehouseSnapshotRef` identifies authoritative read state.” | “table snapshot time is fact time,” “catalog branch is approval.” |
| OCSF | External schema profile for `normalized_fields`. | “OCSF-aligned normalized fields through active profile.” | “OCSF object is Cadastre entity,” “OCSF observable is graph node.” |
| Splunk CIM | Lossy deterministic projection target. | “CIM projection with loss manifest.” | “CIM is silver source of truth.” |
| Semantic overlays and Taxi-like tooling | Non-authoritative authoring and validation aids. | “semantic overlay supports mapping validation.” | “semantic type is identity evidence by itself.” |
| External graph systems | Cautionary references and taxonomy inputs only when mapped. | “external graph taxonomy translation.” | “external graph key resolves identity,” “traversable edge proves reachability.” |
| Graph backends | Replaceable read-model apply/query targets. A named backend selection default may be used only through active `090` backend selection/profile contracts and `100` package-set gates. | “backend selection policy,” “backend profile and preflight,” “package-gated backend activation.” | “backend product defines Cadastre semantics,” “backend default is graph truth,” “future providers are prohibited.” |
| Package registries and signing systems | Evidence sources for package acquisition/trust. | “package signature verification result under trust policy.” | “valid signature means active package.” |
| Threat-intel systems | Enrichment context by default. | “threat-intel enrichment record.” | “indicator equality creates asset identity.” |
| Lineage and metadata systems | Governance/lineage metadata unless mapped. | “lineage facet mapping policy.” | “facet proves evidence or completeness.” |
| UI labels | Presentation and state communication. | “user-facing label maps to stable domain term.” | “label is ID,” “visible order determines behavior.” |
| Implementation modules/functions/routes | Implementation details unless owner spec makes them observable. | “module implements domain term.” | “module name defines domain term.” |
| Storage artifacts/object keys | Implementation/persistence realization. | “storage key maps to artifact ref when owner permits.” | “object-store key is evidence identity by default.” |
| Research reports | Supporting evidence only. | “research report informed owner spec.” | “research report grants active behavior.” |
| Candidate drafts | Candidate text until statused. | “candidate pending owner review.” | “candidate is authoritative.” |
| Future reachability analyzers | Deferred analysis evidence only. | “inactive future reachability candidate.” | “reachable,” “not reachable,” “has theoretical reachability” in MVP. |

## 21. Normative requirements

Root-domain-owned requirements only:

| ID | Requirement | Owner | Verification method |
| --- | --- | --- | --- |
| `DOM-REQ-001` | `docs/nlspec/domain.md` must define canonical Cadastre terms without importing template-specific domain concepts. | `docs/nlspec/domain.md` | Review vocabulary and exclusion note. |
| `DOM-REQ-002` | `domain.md` must route each high-risk term to exactly one primary owner or mark it with `TODO:`. | `domain.md` | Owner columns in Sections 7, 10, 11, 12, 15. |
| `DOM-REQ-003` | `domain.md` must not add runtime behavior outside vocabulary interpretation, boundary rules, and owner lookup. | `domain.md`, `000` | Diff review against owner specs. |
| `DOM-REQ-004` | `domain.md` must treat implementation modules, storage artifacts, UI labels, graph backend labels, package names, route names, and external-system terms as mappings, not definitions. | `domain.md` | Boundary tables in Sections 18 and 20. |
| `DOM-REQ-005` | `domain.md` must surface unresolved contradictions and missing owner decisions as `TODO:` entries. | `domain.md` | Section 25 contains each blocking ambiguity. |
| `DOM-REQ-006` | `domain.md` must preserve the Cadastre source-of-truth map and must not silently promote candidate specs, research, or drafts into implementation authority. | `domain.md`, `000` | Sections 1 and 5 state status limits. |
| `DOM-REQ-007` | `domain.md` must define graph-domain concepts without making graph state authoritative. | `domain.md`, `090` | Section 13 and graph boundary checks. |
| `DOM-REQ-008` | `domain.md` must define identity naming classes without permitting weak evidence to auto-merge. | `domain.md`, `070` | Section 15 matches resolver boundaries. |
| `DOM-REQ-009` | `domain.md` must define external-system boundaries without allowing external schema, taxonomy, lineage, registry, package, or analyzer terms to become Cadastre authority by default. | `domain.md` | Section 20 table. |
| `DOM-REQ-010` | `domain.md` must include binary Definition of Done criteria sufficient to judge root-domain completeness. | `domain.md` | Section 26. |
| `DOM-REQ-011` | `domain.md` must route volatility classes to owner specs and must not define activation algorithms, artifact schemas, version-manifest schemas, or package-set behavior. | `domain.md`, `000`, `010`, `030`, `100`, `120` | Sections 8.1, 10A, and 24. |
| `DOM-REQ-012` | `domain.md` must define a closed context-map relationship type vocabulary and must reject unknown relationship types. | `domain.md` | Section 9 relationship type table and `120-DOMAIN-CMAP-*` validation. |
| `DOM-REQ-013` | Every runtime-affecting cross-context claim must name a Section 9 edge, an exact behavior owner, exact crossing artifacts, a no-effect default, and a validation route. | `domain.md`, named owner spec | Context-map matrix review and owner export lookup. |
| `DOM-REQ-014` | Context-map edges must default to `no_implicit_effect` when crossing artifacts are missing, inactive, stale, ambiguous, checksum-invalid, out of scope, or not covered by the owner contract. | `domain.md`, named owner spec | Negative validation rows prove no fallback effect. |
| `DOM-REQ-015` | `domain.md` must include `140` as a bounded context and must route telemetry through `telemetry_handoff` or `prohibited_dependency` edges only. | `domain.md`, `140`, `110`, `010` | Section 9 coverage and observability negative validation rows. |
| `DOM-REQ-016` | `domain.md` must keep graph backend product selection owner-routed and must not make a backend product a domain concept, default authority, graph semantics source, activation source, or future-provider exclusion. | `domain.md`, `090`, `100`, `120` | Section 5, Section 7, Section 20, and Section 25 wording review. |

## 22. Intentional implementation latitude

| Area | Latitude granted | Boundary | Reason it is caller-equivalent |
| --- | --- | --- | --- |
| Markdown file placement before index update | Implementers may place the candidate root document in any repo path for review. | It must not claim source-of-truth status until `000` maps it. | Path does not affect domain semantics before authority assignment. |
| Internal formatting of implementation maps | Implementers may maintain local module-to-domain mapping files. | They must not redefine domain terms or owners. | Caller-visible domain behavior depends on owner specs, not local formatting. |
| Graph backend product | Implementers may evaluate different graph backends. | Active backend profile and graph contracts must preserve Cadastre IDs, ordering, and non-authority rules. | Backend choice is caller-equivalent only through `090` contract conformance. |
| UI display names | Implementers may choose labels, localization, or layout. | Labels must map to stable domain terms and must not act as IDs. | Users see different text but domain/API behavior remains stable. |
| Package implementation language | Package authors may use different languages/tools. | Package activation, stage binding, outputs, validation, and trust contracts must pass. | Runtime behavior is determined by package contracts, not language. |
| Validation harness implementation | Validation may be implemented with different tools. | `ValidationMatrix`, fixtures, and acceptance reports must be deterministic. | Pass/fail evidence remains caller-equivalent. |
| Private source bindings | Operators may bind different concrete vendors/sources to public feed profiles. | Public model remains vendor-neutral and private bindings do not leak. | Cadastre consumes the same public contract shape. |
| Documentation examples | Authors may include examples when needed. | Examples must not introduce new terms, authority, or behavior. | Examples do not affect implementation obligations. |

## 23. Reviewer and coding-agent rules

Reviewers, agents, and specification authors must:

- identify the bounded context before asserting behavior;
- identify the Section 9 context-map edge before asserting any cross-context dependency;
- classify the dependency with exactly one `ContextMapRelationshipType`;
- reject cross-context behavior claims when the crossing artifact, behavior owner, default missing behavior, or validation route is absent;
- use canonical Cadastre terms from Section 10;
- consult the primary owner spec before asserting behavior;
- use stable identifiers instead of labels, route names, module names, graph backend IDs, storage keys, package names, or external-system identifiers;
- treat implementation modules and storage artifacts as mappings, not definitions;
- use `TODO:` for unresolved owner decisions, contradictions, missing source evidence, or missing mapping rows;
- reject candidate text that defines high-risk terms without owners;
- reject text that imports template-specific concepts, workflows, identifiers, entity models, lifecycle states, or acceptance criteria;
- reject text that creates synonyms for exact Cadastre tokens unless an owner spec defines the synonym and canonical replacement;
- reject text that uses a Section 12 domain relationship family as a substitute for a Section 9 context-map edge;
- reject text that turns future, deferred, research, external, candidate, draft, or implementation content into active domain authority;
- reject graph, CIM, OCSF, lineage, metadata, analysis, package, or UI wording that implies authority not granted by an owner spec;
- reject source absence, identity merge, graph traversal, reachability, package activation, or lifecycle claims that lack the required owner profile or state machine.

## 24. Maintenance and drift-control rules

`domain.md` must be updated, or an explicit “domain vocabulary unchanged” statement must be recorded, when a change modifies any of the following:

- core entity membership;
- domain relationship families;
- bounded-context membership;
- context-map relationship types;
- context-map edge rows;
- crossing-artifact owner routing;
- prohibited context dependencies;
- canonical identifiers or identifier classes;
- closed vocabulary tokens;
- lifecycle vocabulary;
- graph-domain concepts;
- source authority concepts;
- identity and naming rules;
- external-system boundaries;
- owner-spec assignments;
- high-risk terminology decisions;
- root-domain acceptance criteria.

Drift-control rules:

- `domain.md` must point to owner specs instead of copying full owner contracts.
- Copied exact token lists must identify their owner and must be updated when the owner changes.
- New glossary entries must include definition, forbidden interpretation when ambiguity is plausible, canonical identifiers when applicable, owner reference, and status.
- Future-only or deferred terms must be labeled and must not appear as active behavior.
- Deprecated terms must name canonical replacements and change owner.
- Near-duplicate terms are defects unless the distinction is explicit in Section 8 or Section 10.
- Research report language must be translated through owner specs before it appears as active Cadastre language.
- External product names must remain external references unless an owner spec declares a vendor-neutral contract mapping.
- `TODO:` rows must not be deleted without an owner decision or source update that resolves them.
- Any change that reclassifies a contract, profile, row set, package, fixture, registry object, or runtime state by volatility class must update `000`, `010`, `030`, `100` when package activation is involved, and `120` validation rows. `domain.md` must route the change and must not define the runtime behavior.
- Any change that adds a runtime-affecting dependency between owner specs must update Section 9 or add a `TODO:` ledger row. The change must not rely on Section 12 relationship families, implementation dependencies, package dependencies, graph edges, validation fixtures, or telemetry signals as substitutes for a context-map edge.

## 25. Open questions, blockers, and required owner confirmation

Section 25 is a closure ledger for root-domain status only. It must not define runtime behavior and must not duplicate owner schemas, algorithms, defaults, failure precedence, activation rows, or validation harness behavior. Each row must resolve to exactly one owner contract or validation row family.

Closed `domain_closure_state` values are `resolved_owner_routed`, `blocked_validation`, `blocked_owner_todo`, `blocked_owner_catalogs`, and `inactive_deferred`.

Closed `domain_behavior_until_closed` values are `owner governs`, `validation fails`, `deferred no-op`, and `not runtime`.

The `owner_contract_scope` column is required for every row. It must distinguish stable owner contract, activation-controlled row set, validation-only family, or inactive deferred material. It must not describe runtime behavior.

### Owner-routing sentence form

`<ImportedContract>` is owned by `<owner spec>`. This section routes to `<ImportedContract>` and defines no additional schema, default, algorithm, failure precedence, activation rule, validation harness behavior, or runtime side effect.

| domain_row_id | owner_contract | owner_contract_scope | domain_closure_state | remaining_blocker | domain_behavior_until_closed | validation_target |
| --- | --- | --- | --- | --- | --- | --- |
| `DOM-TODO-ACTIVATION-CATALOG-CLOSURE` | `000.MVPActivationCatalogClosureGate`, `030.ActivationCatalogClosureManifestRule`, `120.ActivationCatalogClosureValidationMatrix`, and owner-local row catalogs in `020`, `050`, `060`, `070`, `090`, and `100` | root-domain routing only for activation-catalog closure; no runtime schema, default, algorithm, validation harness behavior, or side effect | `blocked_owner_catalogs` | owner specs must materialize active row-set refs or exact deterministic block refs for feed category closure, OCSF mapping closure, source-authority closure, resolver closure, graph profile/backend closure, and package activation closure | not runtime | `120-DOMAIN-SECTION25-*`; `120-ACTIVATION-CATALOG-CLOSURE-*` |
| `DOM-TODO-ACTIVATION-ROW-SCHEMA-PRECISION` | `030.ActivationControlledRowSchema`, `030.ActivationControlledRowRef`, and owner-local row field tables | activation-controlled row-schema precision routing only; no runtime schema, default, algorithm, validation harness behavior, or side effect | `blocked_validation` | owner specs must convert production-affecting row families to complete `030.ActivationControlledRowField` tables or mark them inactive, deferred, validation-only, non-production, or deterministically blocked | not runtime | `120-DOMAIN-SECTION25-*`; `120-ACTIVATION-ROW-SCHEMA-*` |
| `DOM-TODO-STRUCTURED-INPUT-REPOSITORY` | `030.StructuredInputRepositoryProfile`, `030.StructuredInputRepositorySnapshot`, `030.StructuredInputChangeProposal`, `100.StructuredInputMaterializationResult`, `110.StructuredInputRepositoryAccessPolicy`, `110.StructuredInputRepositoryRedactionPolicy`, and `120.StructuredInputRepositoryValidationMatrix` | structured-input workflow validation and materialization handoff | `blocked_validation` | repository profile refs, snapshot refs, validation run refs, materialization refs, package release refs, package-set refs, redaction fixtures, audit fixtures, telemetry fixtures, and manifest refs must pass with non-`TODO` checksums before production-affecting structured-input repository workflow is authoritative | validation fails | `120-DOMAIN-STRUCTURED-INPUT-*`; `120-STRUCTURED-INPUT-*` |
| `DOM-TODO-005` | `060.SourceAuthorityClosureMatrix` | activation-controlled source-authority closure row-chain catalogs | `blocked_owner_catalogs` | routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE`; `060` remains the runtime owner and must materialize active source-authority row-chain refs or deterministic block refs | validation fails | `120-DOMAIN-SECTION25-*`; `120-SOURCE-CLOSURE-*` |
| `DOM-RESOLVED-COVERAGE-DOMAIN-TOKEN` | `060.CoverageDomainToken`, `060.CoverageDomainCatalog`, and `060.CoverageDomainAliasRejectionTable` | vocabulary routing only for coverage-domain token identity; no runtime grammar, alias table, error semantics, validation algorithm, or mapping behavior is defined by `domain.md` | `resolved_owner_routed` | none | owner governs | `120-DOMAIN-SECTION25-*`; `120-COVERAGE-DOMAIN-*` |
| `DOM-RESOLVED-006` | `080.CorrectionSnapshotRefPolicy` | stable correction snapshot reference policy | `resolved_owner_routed` | none | owner governs | `120-DOMAIN-SECTION25-*`; `120-TEMPORAL-CORRECTION-*` |
| `DOM-RESOLVED-007` | `080.ReplayEquivalencePolicy` | stable replay equivalence policy | `resolved_owner_routed` | none | owner governs | `120-DOMAIN-SECTION25-*`; `120-REPLAY-OUTPUT-CLASS-*` |
| `DOM-TODO-008` | `050.ObservationToOCSFMappingRowSet` | activation-controlled OCSF mapping row catalog | `blocked_owner_catalogs` | routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE`; `050` remains the owner for mapping row schemas, compiled artifact refs, and `cadastre_only` selection | validation fails | `120-DOMAIN-SECTION25-*`; `120-OCSF-MAP-*` |
| `DOM-TODO-009` | `090.GraphEdgeSemanticsRegistry` | stable graph edge semantics plus MVP graph profile validation | `blocked_owner_catalogs` | routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE`; `090` remains the owner for graph edge semantics, active profile closure, and graph output eligibility | validation fails | `120-DOMAIN-SECTION25-*`; `120-GRAPH-PROFILE-CLOSURE-*` |
| `DOM-TODO-010` | `030.RequiredLifecycleMachineBindings` plus owner-specific machine bindings | lifecycle validation handoff family | `blocked_validation` | lifecycle validation rows for `030`, `070`, `090`, `100`, and `120` machines | validation fails | `120-DOMAIN-SECTION25-*`; `120-LIFECYCLE-*` |
| `DOM-TODO-020` | `030.RunLockSet`, `030.RunLockLeasePolicy`, `030.RunLockOperationEvidence`, `030.RunLockRecoveryEvidence`, and `030.RunLockCommitGuard` | run-lock closure validation handoff family | `blocked_validation` | owner behavior specified in `030`; validation requires `120-RUNLOCK-*` | validation fails | `120-DOMAIN-SECTION25-*`; `120-RUNLOCK-CLOSE-*` |
| `DOM-TODO-011` | `domain.md.Runtime exports` | validation-only absence of runtime exports | `resolved_owner_routed` | none | not runtime | `120-DOMAIN-SECTION25-*` |
| `DOM-RESOLVED-012` | `100.PackageType` | closed package type enum identity only | `resolved_owner_routed` | package policy, deprecation, compatibility, validation, package-set, trust, and manifest blockers are routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE` and remain owner-local to `100` | owner governs | `120-DOMAIN-SECTION25-*`; `120-PACKAGE-TYPE-*` |
| `DOM-TODO-013` | `090.GraphBackendSelectionPolicy` | graph backend default selection plus production activation gating | `blocked_owner_catalogs` | routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE`; `090`, `100`, and `120` own backend/profile/package validation rows | validation fails | `120-DOMAIN-SECTION25-*`; `120-GRAPH-BACKEND-SELECTION-*`; `120-GRAPH-BACKEND-PACKAGE-GATE-*` |
| `DOM-TODO-014` | `domain.md.ContextMapRelationshipType` | validation-only context-map relationship vocabulary | `blocked_validation` | `120-DOMAIN-CMAP-RELATIONSHIP-TYPE-*` rows validating closed vocabulary, unknown-token rejection, and no runtime restatement | validation fails | `120-DOMAIN-CMAP-*` |
| `DOM-TODO-015` | `domain.md.ContextMapEdgeMatrix` | validation-only context-map edge matrix | `blocked_validation` | `120-DOMAIN-CMAP-EDGE-*` rows validating edge coverage, unique edge IDs, owner coverage, crossing-artifact exactness, no-effect defaults, and prohibited dependency rows | validation fails | `120-DOMAIN-CMAP-*` |
| `DOM-TODO-016` | `140.TelemetryNonAuthorityRule` and `110.TelemetryHealthMappingPolicy` | telemetry non-authority and observability health validation handoff | `blocked_validation` | observability context-map rows and telemetry non-authority negative cases | validation fails | `120-DOMAIN-CMAP-*`; `120-OBSERVABILITY-*` |
| `DOM-TODO-018` | `040.CoreOneOfRegistry` and `040.EvidenceArtifactClassRegistry` | validation-only core one-of and evidence artifact closure family | `blocked_validation` | `120-CORE-ONEOF-CLOSURE-*`, `120-CORE-EVIDENCE-ARTIFACT-*`, `120-CORE-NULL-OMISSION-*`, `120-CORE-ID-REPLAY-ONEOF-*`, and `120-CORE-ERROR-REGISTRY-*` rows must pass | validation fails | `120-DOMAIN-SECTION25-*`; `120-CORE-ONEOF-CLOSURE-*`; `120-CORE-EVIDENCE-ARTIFACT-*` |
| `DOM-TODO-019` | `070.ResolverProfileRowSet` and resolver activation artifacts | activation-controlled identity resolver artifact catalog | `blocked_owner_catalogs` | routed through `DOM-TODO-ACTIVATION-CATALOG-CLOSURE`; `070` remains the owner for resolver row schemas, weak-evidence defaults, candidate generation, blockers, decisions, review, split, selector, and explanation behavior | validation fails | `120-DOMAIN-SECTION25-*`; `120-IDENTITY-CLOSURE-*`; `120-IDENTITY-REPLAY-*`; `120-IDENTITY-REVIEW-*`; `120-IDENTITY-SPLIT-*`; `120-IDENTITY-EXPLANATION-*` |

Stable source-authority behavior is owned by `060.SourceAuthorityClosureMatrix`. Production promotion remains blocked until every active absence-sensitive feed category has exact closure rows or deterministic block rows validated by `120-SOURCE-CLOSURE-*`.

`CoverageDomainToken` and `CoverageDomainCatalog` are owned by `060`. This section routes vocabulary to those contracts and defines no additional token grammar, alias behavior, validation algorithm, error semantics, mapping behavior, or runtime side effect.

Activation-controlled row-schema precision is owned by `030.ActivationControlledRowSchema`, `030.ActivationControlledRowRef`, and owner-local row field tables. Domain behavior remains not runtime until `120-ACTIVATION-ROW-SCHEMA-*` rows pass.

Correction snapshot behavior is owned by `080.CorrectionSnapshotRefPolicy`. No domain action remains.

Replay equivalence behavior is owned by `080.ReplayEquivalencePolicy`, with output-class field selection imported from owner specs. No domain action remains.

OCSF mapping behavior is owned by `050.ObservationToOCSFMappingRowSet`. Production activation remains blocked until active MVP row sets and `120-OCSF-MAP-*` fixtures and checksums exist.

MVP edge semantics are owner-routed to `090.GraphEdgeSemanticsRegistry`. The active edge set is `observed_connection` only, but validation evidence remains blocked until `120-GRAPH-PROFILE-CLOSURE-*` rows pass.

Lifecycle behavior is owner-routed to `030` lifecycle machines and owner-specific machine bindings. Validation remains blocked until lifecycle matrix rows pass.

Run-lock lease lifecycle and stale recovery behavior is owner-routed to `030.RunLockSet`, `030.RunLockLeasePolicy`, `030.RunLockOperationEvidence`, `030.RunLockRecoveryEvidence`, and `030.RunLockCommitGuard`. Domain status remains `blocked_validation` until `120-RUNLOCK-CLOSE-*` passes.

`domain.md` exports no runtime records.

`100.PackageType` owns the canonical package type enum. Package type policy rows and deprecation rows remain owner-local activation blockers in `100` and do not re-open the package type enum row.

`090.GraphBackendSelectionPolicy` owns default backend selection. A backend selection token is not domain truth and is not production activation. Production activation remains blocked until the selected `090` backend profile, `100` package-set gates, and `120` backend/package validation rows pass.

Gold fact subject refs, gold fact object values, and evidence artifact refs are vocabulary-routed to `040` for closed shape, `080` for predicate-specific fact semantics, `060` for source authority, `070` for identity reference eligibility, `090` for graph consumption, `110` for API/error/redaction, and `120` for validation. Missing owner behavior has `no_implicit_effect`.

A downstream implementation must not resolve a blocker by inference. `ValidateSpecSet` must fail with `DOMAIN_LEDGER_OWNER_DUPLICATE` when two Section 25 rows point to the same owner contract without distinct owner scope and distinct validation row family. `ValidateSpecSet` must fail with `DOMAIN_OWNER_STATUS_CONTRADICTION` when this ledger marks a row resolved while a named owner-local blocker remains, or marks a row blocked-owner when the owner contract is closed and required validation rows pass. `ValidateSpecSet` must fail with `DOMAIN_RUNTIME_RESTATEMENT` when this ledger copies runtime schema, algorithm, default, failure precedence, activation behavior, or validation harness behavior from an owner spec instead of routing by exact owner contract name.

## 26. Definition of Done

| ID | Criterion |
| --- | --- |
| `DOMAIN-SCOPE-SELECTOR-VOCABULARY-AC-001` | `domain.md` routes `ScopeSelector`, `ActivationScope`, `ScopeSelectorContext`, scope coverage, scope specificity, and scope ambiguity to `030` without restating schema fields, algorithms, defaults, or failure precedence. |
| `DOMAIN-SCOPE-SELECTOR-VOCABULARY-AC-002` | Forbidden interpretations for selector terms include private binding as public scope, row order as ambiguity tiebreaker, graph selector as identity, dataset-level row as universal wildcard, telemetry scope as domain authority, and package environment string as unnormalized activation selector. |
| `DOM-AC-001` | The document states its authority, non-authority, relation to the source-of-truth map, conflict behavior, contradiction reporting, normative-language scope, and research/template non-authority. |
| `DOM-AC-002` | Every root-domain-owned concept is defined once or has a blocking `TODO:`. |
| `DOM-AC-003` | Every high-risk term in Section 7 has a canonical interpretation, forbidden interpretation, primary owner, source evidence class, and status. |
| `DOM-AC-004` | Every core entity in Section 11 has identity basis, lifecycle basis, key relationships, primary owner, and status or a blocking `TODO:`. |
| `DOM-AC-005` | Every relationship family in Section 12 has meaning, authoritative representation, direction rule, evidence requirement, owner, and non-implication rule or a blocking `TODO:`. |
| `DOM-AC-006` | Graph applicability is resolved as active for graph projection/serving and inactive for theoretical reachability, with deferred reachability blocked by `TODO:` or inactive status. |
| `DOM-AC-007` | Lifecycle vocabulary is sourced, owned, and not copied from the template. |
| `DOM-AC-008` | Defaults and omitted cases are specified where root domain owns interpretation or are routed to owner specs. |
| `DOM-AC-009` | Mapping tables are exhaustive for their declared scope or contain explicit `TODO:` rows. |
| `DOM-AC-010` | Implementation details, UI labels, storage names, graph backend IDs, route names, package names, and external-system terms do not become domain definitions. |
| `DOM-AC-011` | Adjacent spec boundaries are explicit for data, graph, workflow, persistence/lakehouse, API/UI, validation, package/implementation, and external schema/system areas. |
| `DOM-AC-012` | Rationale is separated from normative requirements; normative requirements are listed in Section 21. |
| `DOM-AC-013` | No Cartulary-specific content remains except the short exclusion note in Section 1. |
| `domain-source-closure-status-consistency` | `domain.md` must not mark source-category coverage unresolved when `060` marks the stable contract closed, and must not mark it resolved if `060` still contains owner-local blockers for the stable contract. |
| `domain-identity-closure-status-consistency` | `domain.md` must not mark `070.IdentityReviewCaseStateMachine.v1` blocked when `070` closes it, and must not mark it closed if `070` reintroduces owner-local blockers. |
| `DOM-AC-014` | No Cadastre requirement is invented from the uploaded template. |
| `DOM-AC-015` | Every row in the Section 25 closure ledger matches exactly one owner-local closure state in `000.Owner Contract Registry` and one validation row family in `120.DomainSection25StatusValidationMatrix`. |
| `DOM-AC-016` | The document passes the two-independent-implementers test for domain vocabulary, owner lookup, entity interpretation, relationship interpretation, and boundary decisions. |
| `DOM-AC-017` | The document passes the recreatability test for the root Cadastre domain model and owner map without relying on the uploaded template. |
| `DOM-AC-018` | The document is small enough for prompt context but complete enough to prevent semantic drift at root-domain level. |
| `DOM-AC-019` | No external-draft evidence table remains in this document. |
| `DOMAIN-SCHEMA-PATCH-AC-001` | Root vocabulary routes all core record field schemas to `040`. |
| `DOMAIN-SCHEMA-PATCH-AC-002` | Root vocabulary distinguishes semantic fact key from immutable fact record ID. |
| `DOM-PACKAGE-TERMS-AC-001` | `domain.md` routes `package_type`, `package_type_policy`, `last_known_good_package_set`, `package_quarantine`, and `emergency_package_override` to `100`, rejects broad labels and grouping bundles as vocabulary substitutes, and defines no runtime schema or algorithm for them. |
| `DOMAIN-SCHEMA-PATCH-AC-003` | Root vocabulary states that `EvidenceRef` is not raw payload storage. |
| `DOM-AC-020` | The root-domain owner map uses owner specs and exported contract names only. |
| `DOM-AC-021` | Section 25 has one row per closure issue; any duplicate owner contract has a distinct `owner_contract_scope` and distinct validation family, or validation fails with `DOMAIN_LEDGER_OWNER_DUPLICATE`. |
| `DOM-LIFECYCLE-AC-001` | `domain.md` routes lifecycle machines to owner specs by exact machine ID and does not restate transition matrices. |
| `DOMAIN-CLEANUP-AC-001` | The document contains no banned reference class. |
| `DOMAIN-CLEANUP-AC-002` | Every owner-routing table uses the cleaned status vocabulary from `000`. |
| `DOMAIN-CLEANUP-AC-003` | Conflict handling uses local `TODO:` rows or validation failure. |
| `DOMAIN-CLEANUP-AC-004` | No external-draft evidence table remains. |
| `DOMAIN-CLEANUP-AC-005` | Private implementation artifacts are described without deprecated artifact-index wording. |
| `DOM-VOLATILITY-AC-001` | Every volatility term in `domain.md` has a canonical definition, forbidden interpretation, owner route, and non-runtime boundary. |
| `DOM-VOLATILITY-AC-002` | `domain.md` contains no activation algorithm, artifact field schema, version-manifest field schema, or package-set activation behavior. |
| `DOM-CMAP-AC-001` | Section 9 includes every active bounded context owner named in the document relationship map, including `140`. |
| `DOM-CMAP-AC-002` | Section 9 defines a closed `ContextMapRelationshipType` vocabulary and rejects unknown relationship types. |
| `DOM-CMAP-AC-003` | Every context-map edge has `edge_id`, `from_context`, `to_context`, `relationship_type`, `crossing_artifacts`, `authority_class`, `behavior_owner`, `allowed_root_domain_effect`, `forbidden_runtime_effects`, `default_when_missing_or_invalid`, `validation_route`, and `status`. |
| `DOM-CMAP-AC-004` | Every bounded context appears in at least one context-map edge or is explicitly marked inactive/deferred. |
| `DOM-CMAP-AC-005` | Every runtime-affecting context-map edge names exactly one primary behavior owner or carries a blocking Section 25 `TODO:` ledger row. |
| `DOM-CMAP-AC-006` | Every `anti_corruption_translation`, `authority_handoff`, `projection_handoff`, `activation_gate`, `validation_gate`, `telemetry_handoff`, `deferred_no_effect`, and `prohibited_dependency` edge has at least one negative validation route. |
| `DOM-CMAP-AC-007` | Section 9 and Section 12 are not interchangeable: context-map edges define inter-context routing, and entity relationship families define domain concept relationships. |
| `DOM-CMAP-AC-008` | Missing, inactive, stale, ambiguous, checksum-invalid, out-of-scope, or owner-uncovered crossing artifacts default to `no_implicit_effect` or `deferred_no_effect`. |
| `DOM-CMAP-AC-009` | Graph backend product names do not define root-domain defaults, graph semantics, identity, source authority, package activation, or future-provider exclusion. |
| `DOM-CMAP-AC-010` | The context-map revision adds no runtime exports to `domain.md`. |
| `DOMAIN-RUNLOCK-STATUS-AC-001` | `DOMAIN_OWNER_STATUS_CONTRADICTION` fires if `domain.md`, `000`, `030`, or `120` disagree on run-lock closure status. |
