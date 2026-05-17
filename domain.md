---
doc_id: domain
title: Cadastre Domain Vocabulary and Boundaries
doc_type: domain vocabulary and boundary document
status: migration_active
generated_on: 2026-05-17
---

## 1. Status and authority

`domain.md` is the root Cadastre domain vocabulary and boundary document. It owns repository-wide domain interpretation, canonical term selection, forbidden substitutions, high-risk distinction tables, bounded-context routing, and the domain-level owner map for Cadastre concepts.

`domain.md` does not own runtime algorithms, data schemas, persistence behavior, graph backend behavior, API behavior, package activation behavior, validation harness behavior, implementation module structure, private source bindings, or external-system behavior. Those behaviors must remain owned by the active Cadastre owner specs.

`domain.md` must not add, widen, narrow, or replace runtime behavior. It may state domain interpretation rules and route implementers to the owner spec that owns behavior.

'TODO: Cadastre root domain authority unresolved. The supplied Cadastre Spec Index does not list a root domain document. Add `domain.md` to the source-of-truth map, assign a status, and define whether it is an active NLSpec, companion reference, or generated migration artifact before it may become implementation authority.'

Cadastre source authority is governed by the supplied source-of-truth map and status vocabulary. The supplied index states that active Cadastre NLSpecs are currently `migration_active`, that documents marked `authoritative` may drive implementation, and that research reports are supporting evidence only, never runtime authority. Until the index marks a document `authoritative`, the original PRD remains the upstream intent container and generated NLSpecs are migration evidence.

'TODO: PRD status contradiction unresolved. The uploaded PRD front matter states `status: archived`, while the supplied source-of-truth map states that the original PRD remains upstream intent until the NLSpec set becomes authoritative and that archival occurs after migration acceptance. This document must not resolve that conflict by inference.'

If `domain.md` conflicts with a primary owner spec, the owner spec governs the behavior it owns. The conflict is documentation drift and must be reported in Section 25. If two owner specs conflict, the contradiction is a corpus defect. `domain.md` must not silently choose a side.

The normative words `must`, `must not`, `may`, and `default` in this document govern vocabulary use, domain interpretation, owner lookup, review discipline, and coding-agent context only. This document does not use `should` as a runtime norm.

Research reports, examples, the uploaded template, implementation modules, generated drafts, external repositories, backend products, package registries, UI labels, and storage artifacts are not Cadastre domain authority unless an active Cadastre owner spec explicitly grants that authority.

Content excluded from the uploaded template: Cartulary-specific product terms, workbook-native incident workspace framing, workbook surfaces, built-in tabs, saved views, parties, artifacts, notes, timeline rows, task requests, decisions, object blobs, import sessions, snapshots, releases, `view_schema_id`, `record_id`, `party_id`, and `cartulary.*` identifiers are not Cadastre requirements and are not imported into this document.

## 2. Purpose

`domain.md` must provide the root domain context that lets implementers, reviewers, specification authors, and coding agents use Cadastre terminology consistently without turning implementation representations into domain truth.

This document must:

- define canonical Cadastre domain terms and forbidden substitutions;
- map domain concepts to primary owner specs or owner sections;
- distinguish domain concepts from implementation modules, storage artifacts, UI labels, graph backend labels, package names, route names, and external-system terms;
- preserve the Cadastre authority model and current migration status limits;
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
| Review discipline | Rules for agents and reviewers when terminology or ownership is unresolved. |

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
| Fixtures, validation matrices, golden corpus, acceptance reports, migration acceptance | `120-validation-fixtures-and-acceptance.md` |
| Analysis rules, enrichment, lineage mapping, artifact-class boundaries, registry governance | `130-analysis-enrichment-and-registry-governance.md` |
| Future theoretical reachability candidate contracts | `200-future-reachability-analysis-domain.md`, inactive until promoted |
| Implementation code structure, package/module names, route names, database schemas, CSS classes, local config, private source inventory | Implementation repository and private artifacts; not domain authority by default |

## 4. Domain overview

Cadastre is a lakehouse-fed interpretation, normalization, identity, fact, and projection system for temporal asset intelligence. It consumes externally supplied lakehouse-resident raw feeds, preserves raw evidence, normalizes observations into Cadastre-owned silver envelopes, resolves canonical identity through governed resolver profiles, derives bitemporal gold facts through source authority and temporal policies, and projects those facts into graph serving and optional external outputs.

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
| `000-cadastre-spec-index.md` | Documentation governance, source-of-truth map, status vocabulary, dependency policy, migration ledger pointers, archival rules. | Must follow its status and owner map; must surface unresolved root-domain map gaps. | Product runtime behavior or domain record fields. |
| `010-system-boundary-and-authority.md` | Cadastre product boundary, authority classes, forbidden production inputs, projection authority, public/private boundary. | Imports authority vocabulary and global non-authority rules. | Lakehouse table fields, graph algorithms, identity decisions. |
| `020-lakehouse-feeds-and-table-state.md` | Lakehouse feeds, raw imports, read policies, completeness receipts, table-state refs, maintenance and retention. | Uses its terms for lakehouse-fed input and table-state domain boundaries. | Source absence interpretation, gold truth, identity resolution, graph apply. |
| `030-processing-dag-lifecycle-and-versioning.md` | Processing stage DAG, output permissions, lifecycle machines, run locks, version manifests, stage state. | Uses its lifecycle and stage vocabulary for domain routing. | Parser/gold/identity/graph/package-specific behavior. |
| `040-canonical-data-observation-and-fact-model.md` | Core record shapes, canonical serialization, omission states, raw/silver/core entity/gold/evidence primitives. | Uses its record names as canonical domain vocabulary. | Source authority algorithms, resolver algorithms, external mapping rows, graph projection algorithms. |
| `050-normalization-external-schema-and-mapping.md` | Parser, normalization, OCSF/CIM/external schema profiles, source extension fields, mapping compiler pipeline. | Routes external schema and mapping vocabulary. | Gold authority, identity merge decisions, graph projection behavior. |
| `060-source-authority-completeness-coverage-and-absence.md` | Source authority, completeness, staleness, coverage, absence derivation, progress-signal interpretation, watermarks. | Routes evidence/absence/coverage vocabulary and non-implication rules. | Resolver merge rules or graph apply behavior. |
| `070-identity-resolution-and-target-selectors.md` | Resolver profiles, identity evidence, identity decisions, review, split behavior, target selector safety. | Routes identity and naming vocabulary. | Source authority for non-identity facts, direct graph mutation. |
| `080-temporal-corrections-replay-and-gold-derivation.md` | Temporal semantics, gold derivation, correction, late arrivals, replay equivalence, deterministic side effects. | Routes bitemporal and correction vocabulary. | Backend graph apply or external schema mappings. |
| `090-graph-projection-serving-and-backends.md` | Graph projection, graph delta identity, graph edge semantics, traversal, backend profiles, apply, query, rebuild, drift. | Defines graph-domain boundary and non-authority rules at root level. | Fact existence, identity resolution, source completeness, backend product selection. |
| `100-packages-supply-chain-and-activation.md` | Package artifacts, release manifests, package-set activation, trust, compatibility, rollback, quarantine, emergency behavior. | Routes package and activation vocabulary. | Package business behavior outside activation and execution permissions. |
| `110-api-ux-health-errors-and-security.md` | Public APIs, UX state labels, health, shared errors, redaction, authorization, audit. | Routes user-facing label and error vocabulary without redefining domain behavior. | Domain-specific behavior owned elsewhere. |
| `120-validation-fixtures-and-acceptance.md` | Fixtures, validation matrices, golden corpus, shadow execution, replay validation, acceptance reports. | Uses validation terms for Definition of Done and acceptance criteria. | New behavioral requirements not owned by domain specs. |
| `130-analysis-enrichment-and-registry-governance.md` | Analysis rules, findings, metrics, enrichment, lineage facet mapping, artifact-class policy, registry governance. | Routes non-authoritative analysis/enrichment/registry vocabulary. | Fact, identity, graph, completeness, package, or watermark authority. |
| `200-future-reachability-analysis-domain.md` | Inactive future reachability candidate contracts. | Labels reachability as deferred and blocks active MVP reachability claims. | Active MVP behavior, solver selection, negative reachability facts, graph reachability edges. |
| `PRD-Cadastre..md` | Upstream product intent and source PRD material; status conflict unresolved. | Provides domain thesis, baseline product boundary, and archived/upstream intent evidence subject to Section 1 TODO. | Direct implementation authority after NLSpec cutover; resolving conflicts without owner decision. |
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
| Cadastre product boundary | Cadastre consumes lakehouse-resident feeds, interprets, normalizes, resolves, derives, and projects. | Cadastre as default enterprise source collector, scanner, syslog receiver, API poller, or CDC connector. | `010`, `020` | PRD Product Summary; `010` Boundary Contract | Resolved. |
| System of record versus projection | Authoritative Cadastre records live in the lakehouse-backed record flow. Graph, CIM, OCSF export, metadata, lineage, and analysis outputs are projections or non-authoritative artifacts unless owner spec grants authority. | Graph backend, CIM output, OCSF export, metadata graph, lineage graph, or analysis output as source of truth. | `010`, `040`, `090`, `130` | PRD Product Summary; `010` Authority Classes; `090` Projection Authority | Resolved. |
| Raw evidence versus source completeness | Raw evidence and feed read receipts can support completeness decisions only through active profiles. | Missing rows or successful feed reads as source absence by themselves. | `020`, `060` | `020` Completeness Receipt Contract; `060` Completeness Contract | Resolved. |
| Raw observation versus normalized observation | `RawRecord` preserves source-native evidence; `CadastreSilverObservation` is a normalized Cadastre silver envelope. | Parser output or external schema fields as canonical truth. | `040`, `050` | `040` Core Records; `050` Parser and Silver Normalization Contracts | Resolved. |
| Silver observation versus gold fact | Silver observations preserve normalized evidence and Cadastre-owned metadata; `GoldFact` is a canonical bitemporal assertion derived through authority and temporal policies. | Treating every silver observation as a canonical fact. | `040`, `060`, `080` | `040` Core Records; `060` Source Authority Contract; `080` Gold Derivation | Resolved. |
| Canonical entity versus source asset | `CanonicalEntity` is product-owned identity. `SourceAsset` is source/feed-scoped asset identity. | Source-native ID, provider key, graph key, or display label as canonical identity. | `040`, `070` | `040` Core Records; `070` Resolver Authority | Resolved. |
| Identifier versus identity decision | `Identifier` records typed observed identifiers; `IdentityDecision` is the governed resolver output. | Identifier equality as automatic canonical identity. | `040`, `070` | `040` Core Records; `070` Evidence Roles | Resolved. |
| Target hint versus identity | `UnresolvedTargetReference` is a relationship hint and must not create or merge canonical entities. | Mapped target, OpenGraph-style property matching, name matching, or graph key as identity. | `070` | `070` Target Selector Safety | Resolved. |
| Source authority versus source completeness | Source completeness is coverage or read completeness evidence; source authority is permission to derive or assert a fact. | Complete source scope as automatic authority for every fact type or predicate. | `060` | `060` Source Authority Contract and Completeness Contract | Resolved. |
| Absence versus unknown | Absence may be emitted only through active absence authority, completeness, coverage, and staleness gates. Unknown preserves lack of authority or insufficient evidence. | Missing value, missing row, TTL expiry, lease expiry, partial feed, or permission limit as absence. | `060`, `040` | `060` DeriveAbsenceOrUnknown; `040` Omission Semantics | Resolved. |
| Graph edge versus domain relationship | A graph edge is a projected read-model object governed by `GraphEdgeSemantics`. A domain relationship is the underlying meaning supported by facts and evidence. | Backend edge existence as source fact, identity proof, reachability proof, risk proof, or completeness proof. | `090`, `040`, `080` | `090` Edge Semantics and Traversal | Resolved. |
| Graph traversal versus reachability | `GraphTraversalClass` controls path eligibility. Theoretical reachability is inactive and must not be emitted in MVP. | Any graph path as network reachability or service access. | `090`, `200` | `090` Definition of Done; `200` Prohibited MVP Outputs | Resolved for MVP. |
| External schema term versus Cadastre term | OCSF and CIM terms are external schema/projection vocabulary mapped through profiles. | OCSF/CIM fields as Cadastre identity, authority, completeness, fact, or graph authority. | `050`, `040`, `060` | `050` External Schema Profile and CIM Projection Contract | Resolved. |
| Analysis finding versus fact | `AnalysisFinding`, `AnalysisMetric`, and `RiskAcceptanceRecord` are analysis or workflow outputs. | Analysis output as remediation, risk reduction, fact retraction, graph edge removal, or source completeness. | `130` | `130` Analysis Rules and Non-Authority Rule | Resolved. |
| Package artifact versus package activation | A package artifact is immutable release material; production activation occurs only through a `ProductionPackageSetManifest`. | Single artifact, version string, signature status, SBOM, provenance, dependency lock, or validation run as activation authority. | `100` | `100` Activation Unit and Package Set Manifest | Resolved. |
| Lifecycle status versus lifecycle machine | `LifecycleStatus` names shared states; runtime lifecycle behavior comes from a declared machine or pure deterministic algorithm authority. | Inferring transitions from diagrams, process status, logs, caches, or names. | `030` | `030` Lifecycle Contract | Resolved. |
| `domain.md` authority | Root domain document authority is not yet listed in the supplied source-of-truth map. | Treating this generated document as implementation authority without index status. | `000` plus TODO owner decision | `000` Source-of-Truth Map lacks root `domain.md` row. | `TODO:` unresolved. |
| PRD status | Uploaded PRD says `archived`; source-of-truth map says original PRD remains upstream intent until NLSpec set authoritative. | Treating the conflict as resolved by this document. | `000` plus owner decision | PRD front matter and `000` Authoritative Source Rule. | `TODO:` unresolved. |

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
| Implementation module and domain concept | Module names may implement or map domain terms; they must not define them. | Letting package names, table names, route names, or functions redefine domain terms. | `domain.md`, implementation repository after exact inspection |
| External schema term and Cadastre term | External terms map through profiles. Cadastre terms remain product-owned. | Letting OCSF/CIM/ECS/Sigma/MISP/OpenLineage/OpenMetadata labels become Cadastre identity, fact, graph, or authority. | `050`, `130`, `060`, `090` |
| Source freshness and derived-view lag | Source stale state belongs to source staleness policy. Derived-view lag belongs to graph/API state. | Collapsing stale source evidence and stale graph view into one status. | `060`, `090`, `110` |
| Package verification and package activation | Verification evidence supports activation decisions. Activation is package-set governed. | Treating valid signature, SBOM, provenance, or dependency lock as activation. | `100` |
| Future reachability and MVP graph behavior | Future reachability is inactive and graph-neutral by default. MVP graph must not emit theoretical reachability. | Emitting `has_theoretical_reachability`, boolean reachability, or unqualified reachable wording. | `090`, `200` |

## 9. Bounded contexts

| Bounded context | Owns domain language for | Must not own | Primary owner |
| --- | --- | --- | --- |
| Documentation governance | Spec status, source-of-truth rows, dependencies, migration state, archival policy. | Product runtime behavior. | `000` |
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
| Validation and acceptance | Validation matrices, fixtures, golden corpus, acceptance reports, migration acceptance. | New behavioral requirements not owned by a domain spec. | `120` |
| Analysis, enrichment, lineage, registry | Read-only analysis, findings, metrics, enrichment records, lineage mapping, artifact-class policy, registry governance. | Fact, identity, graph, completeness, package, or watermark authority. | `130` |
| Future reachability | Deferred reachability claim kinds, result states, model capability, analysis artifacts, claim policy. | Active MVP graph/gold behavior, solver selection, negative reachability facts. | `200`, inactive |

## 10. Domain vocabulary

| Term | Definition | Not this | Canonical identifiers or tokens | Primary owner | Status |
| --- | --- | --- | --- | --- | --- |
| Cadastre | A lakehouse-fed interpretation, normalization, identity, fact, and projection system for temporal asset intelligence. | Enterprise source collector, SIEM replacement, graph-first sync engine. | `Cadastre` | `010`, PRD | Resolved. |
| Enterprise source system | External operational system that supplies or originates security, identity, endpoint, vulnerability, configuration, network, DNS, DHCP, IPAM, or related observations. | Cadastre production component. | None defined in public model. | `010`, `020` | Resolved as external. |
| External raw supplier | External pipeline, tool, team, or supplier class that delivers raw feeds into the lakehouse. | Cadastre production collector. | `RawSupplierProfile` | `020` | Resolved. |
| Lakehouse | Authoritative storage substrate for Cadastre raw/bronze, silver, identity, gold, graph-delta, health, and related record classes. | Graph backend, metadata graph, source system. | `LakehouseTableProfile`, `LakehouseSnapshotRef`, `DatasetVersionRef`, `LakehouseCommitRef` | `020`, `010` | Resolved. |
| Lakehouse feed | Vendor-neutral lakehouse-resident raw input feed that Cadastre may read. | Direct API collection or source exploration. | `LakehouseFeedProfile`, `RawFeedManifest` | `020` | Resolved. |
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
| Identity decision | Governed resolver output that creates, rejects, splits, conflicts, or declines identity relationships. | Identifier equality, source-native merge history, graph key, reviewer note. | `IdentityDecision` | `070`, `040` | Resolved. |
| Unresolved target reference | Relationship hint that has not created or merged canonical identity. | Weak entity or automatic relationship target. | `UnresolvedTargetReference` | `070` | Resolved. |
| Source authority profile | Executable authority arbitration for gold fact type and predicate. | Source category preference, source completeness, or staleness policy. | `SourceAuthorityProfile`, `SourceAuthorityProfileRow` | `060` | Resolved. |
| Coverage assertion | Source-backed coverage state required before coverage-dependent facts or negative claims may be emitted. | Scanner presence, source presence, or feed read success by itself. | `CoverageAssertion`, `CoverageDimensionProfile` | `060` | Resolved with TODO catalog. |
| Omission state | Explicit field-level state for present, empty, not provided, not applicable, permission limited, unavailable, unsupported, malformed, or redacted values. | Nullable schema field or missing source field. | `observed_present`, `observed_empty`, `not_provided`, `not_applicable`, `permission_limited`, `source_unavailable`, `unsupported`, `malformed`, `redacted` | `040` | Resolved. |
| Fact absence outcome | Fact-level absence or unknown outcome derived through authority, completeness, coverage, and staleness gates. | Field omission state. | `FactAbsenceOutcome`, `AbsenceDerivationResult` | `040`, `060` | Resolved at concept level. |
| Gold fact | Canonical bitemporal assertion with subject, predicate, object/value, valid time, known time, assertion state, confidence, authority, evidence, and correction refs. | Silver observation or graph edge. | `GoldFact`; assertion states in Section 14. | `040`, `060`, `080` | Resolved. |
| Evidence ref | Product-owned reference from an output to supporting artifact, role, checksum, and visibility state. | Raw payload itself or external lineage facet. | `EvidenceRef` | `040`, `130` | Resolved. |
| Temporal observation time resolution | Persisted resolution of the time inputs selected for fact derivation. | Implicit current-time fallback. | `TemporalObservationTimeResolution` | `080` | Resolved. |
| Version manifest | Immutable manifest of every output-affecting profile, policy, package, state, lakehouse ref, completeness receipt, coverage assertion, schema artifact, temporal policy, resolver profile, graph profile, graph apply result, and acceptance report that can affect output or replay. | Build number, run ID, or deployment label by itself. | `VersionManifest` | `030`, cross-cutting | Resolved. |
| Processing stage DAG | Acyclic deterministic stage graph with declared inputs, outputs, profiles, state kinds, lifecycle, and failure behavior. | Implementation scheduler or package module layout. | `ProcessingStageDAG` | `030` | Resolved. |
| Lifecycle status | Shared lifecycle status vocabulary for activation-controlled records. | Full transition table or UI status label. | `draft`, `validated`, `canary`, `active`, `deprecated`, `retired`, `quarantined` | `030` | Resolved. |
| Graph projection profile | Mapping from gold facts and declared synthetic structural rules to graph nodes, edges, properties, expiry scopes, and no-op outcomes. | Graph backend schema or query language. | `GraphProjectionProfile` | `090` | Resolved. |
| Graph delta set | Persisted set of graph node and edge deltas produced by projection. | Backend mutation log or authoritative facts. | `GraphDeltaSet` | `090`, `040` | Resolved. |
| Graph edge semantics | One active row per graph edge type defining source fact type, predicate, direction, evidence, states, temporal policy, confidence policy, traversal class, non-implications, and no-op conditions. | Backend relationship type or external graph taxonomy. | `GraphEdgeSemantics`, `GraphTraversalClass` | `090` | Resolved. |
| Graph read model | Derived graph-serving state rebuilt from authoritative records and persisted graph deltas. | System of record. | `DerivedViewState`, `GraphBackendProfile` | `090`, `010` | Resolved. |
| Package artifact | Immutable package materialization subject to trust and release evidence. | Production activation target by itself. | `PackageArtifact` | `100` | Resolved. |
| Production package set manifest | Immutable coherent set of package releases that is the only production activation target. | Single package version, tag, signature status, SBOM, provenance, or validation result. | `ProductionPackageSetManifest` | `100` | Resolved. |
| Validation matrix | Validation artifact mapping contract, scenario, fixture, expected output, expected error/no-op, owner spec, and pass/fail evidence. | Behavior owner or test plan replacing domain specs. | `ValidationMatrix` | `120` | Resolved. |
| Analysis finding | Read-only analysis or workflow output. | Gold fact, remediation, risk reduction proof, source completeness. | `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord` | `130` | Resolved. |
| Threat-intel enrichment | Enrichment context for indicators, sightings, taxonomies, galaxies, object templates, confidence labels, distribution controls. | Identity, source authority, gold fact, graph edge, absence, cleanup, retraction, coverage, or watermark authority. | `ThreatIntelEnrichmentRecord`, `ThreatIntelEnrichmentProfile` | `130` | Resolved. |
| Registry governance artifact | Governance metadata for owners, domains, classifications, glossary labels, policies, lifecycle, custom properties, checksums. | Cadastre fact or graph authority. | `RegistryArtifactGovernance` | `130` | Resolved. |
| Future reachability | Inactive future analysis domain for modeled packet/session/service/identity-conditioned access. | Active MVP graph edge, boolean reachability property, or negative reachability fact. | `ReachabilityResult`, `ReachabilityClaimPolicy`; inactive. | `200`, `090` | Inactive deferred. |
| Private source binding | Private implementation mapping from concrete vendor/product/source route to public vendor-neutral feed contract. | Public canonical domain concept. | `PrivateSourceFeedBinding` | `010`, PRD | Resolved as private-only. |

## 11. Core entities

| Entity | Definition | Identity basis | Lifecycle basis | Key relationships | Primary owner | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Cadastre system | The product boundary that consumes lakehouse feeds and emits interpreted, normalized, identity, fact, graph, API, and optional projection outputs. | Product boundary, not a persisted entity ID. | Spec status and deployment/package lifecycle, as applicable. | Reads lakehouse feeds, writes authoritative records, serves projections. | `010`, `100`, `030` | Resolved. |
| External raw supplier | External provider of lakehouse-resident raw feed data and supplier metadata. | Supplier class/profile identity, not concrete vendor route by default. | Feed/profile activation owned by lakehouse and package specs. | Supplies lakehouse feed; may provide upstream completeness evidence. | `020`, `010` | Resolved. |
| Lakehouse feed | Vendor-neutral lakehouse-resident raw input feed. | `LakehouseFeedProfile` plus feed scope and read target refs. | Feed profile activation and availability checks. | Produces raw feed rows/objects and completeness receipt. | `020` | Resolved. |
| Raw feed manifest | Batch/snapshot manifest for lakehouse feed input. | Manifest checksum and declared feed identity. | Feed read/import lifecycle. | Referenced by raw import, completeness evidence, version manifest. | `020` | Resolved. |
| Raw record | Preserved source-native evidence record imported from declared feed input. | Deterministic raw record ID from feed scope, dataset, supplier/object identity, optional source-native ID, payload hash, import profile version. | Duplicate/quarantine/import status; table retention. | Supports parse results, silver observations, evidence refs. | `040`, `020` | Resolved with raw ID ordering TODO in `020`. |
| Cadastre silver observation | Cadastre-owned normalized observation envelope. | Observation ID from raw refs, observation type, mapping bundle, source scope, normalized payload checksum. | Parse/normalize stage status and mapping validation. | Supports identity evidence, authority evaluation, gold derivation, CIM/OCSF output. | `040`, `050` | Resolved. |
| Canonical entity | Product-owned canonical asset/security entity. | Product-owned canonical entity ID; must not be inferred from labels, source IDs, graph IDs, or backend IDs. | Entity lifecycle state, valid/known bounds, identity decision refs. | Related to source assets, identifiers, gold facts, graph nodes. | `040`, `070` | Resolved. |
| Source asset | Source/feed-scoped asset record. | Source/feed scope plus source-native identity evidence. | Source asset lifecycle evidence. | Related to canonical entity through identity decisions. | `040`, `070` | Resolved. |
| Identifier | Typed observed identifier. | Identifier type, normalized value, source scope, validity interval. | Valid/known time and quality. | Feeds identity evidence items and resolver decisions. | `040`, `070` | Resolved. |
| Identity evidence item | Materialized resolver input. | Evidence item identity from evidence class, identifier/scope/time/evidence refs as defined by `070`. | Resolver input lifecycle. | Feeds candidate generation, blockers, decisions, explanations. | `070` | Resolved. |
| Identity decision | Resolver output. | Decision ID and checksum as defined by resolver profile and version manifest. | Active/terminal review and split behavior through identity owner. | Creates/rejects/splits/conflicts canonical identity relationships; may trigger graph correction handoff. | `070` | Resolved. |
| Gold fact | Canonical bitemporal fact. | Fact ID from canonical subject, predicate, object/value, valid interval, assertion state, authority profile, evidence set. | Assertion state and correction change sets. | Supports graph projection, API detail, analysis, audit. | `040`, `060`, `080` | Resolved. |
| Evidence ref | Product-owned evidence reference. | Referenced artifact class, artifact ID, role, checksum. | Visibility/redaction state. | Links graph, gold, silver, raw, validation, lineage as permitted. | `040`, `110`, `130` | Resolved. |
| Coverage assertion | Coverage-sensitive authorization input. | Coverage assertion ID and dimension profile refs. | Current/stale/expired coverage state through `060`. | Required before coverage-dependent absence or negative claims. | `060` | Resolved with coverage catalog TODO. |
| Version manifest | Immutable replay and output-affecting manifest. | Manifest checksum over declared output-affecting refs. | Production/replay/activation lifecycle. | References profiles, policies, packages, state, lakehouse refs, validation reports. | `030` | Resolved. |
| Graph node delta | Primitive graph node projection delta. | Cadastre-owned deterministic graph node ID. | Delta set and graph apply lifecycle. | Derived from gold facts or declared structural rules; applied to graph read model. | `040`, `090` | Resolved. |
| Graph edge delta | Primitive graph edge projection delta. | Cadastre-owned deterministic graph edge ID; not backend ID. | Delta set and graph apply lifecycle. | Governed by graph edge semantics and projected from facts/rules. | `040`, `090` | Resolved. |
| Graph read-model object | Derived graph serving object after graph apply. | Cadastre graph ID translated through backend profile. | Derived view state and graph apply result. | Served by graph query; drillback to gold/silver/raw. | `090`, `110` | Resolved. |
| Package release | Immutable package release record. | Artifact digest, subject digest, package type, release version, release checksum. | Package lifecycle and package-set activation. | Included in production package set manifest. | `100` | Resolved. |
| Production package set | Coherent activation unit for package runtime behavior. | Immutable package-set checksum. | Promotion, activation, rollback, quarantine, last-known-good. | Provides package roles for DAG stages and validation. | `100`, `030` | Resolved. |
| Analysis finding | Non-authoritative analysis output. | Finding identity as defined by analysis owner. | Analysis/workflow lifecycle only. | May reference graph query and evidence but cannot mutate facts or graph state. | `130` | Resolved. |
| Registry governance artifact | Governance metadata artifact. | Registry artifact checksum and schema refs. | Registry lifecycle. | May annotate owners/classifications/policies but not define fact authority. | `130` | Resolved. |
| Reachability analysis artifact | Future inactive reachability analysis output. | Future query/result/artifact checksum when activated. | Inactive deferred until promoted. | Must not emit MVP facts, graph edges, or graph properties. | `200` | Inactive deferred. |

## 12. Entity relationships

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
| Spec status | Cadastre documentation artifacts. | `000` | Whether a document may drive implementation or is draft/migration/deferred/archive material. | Runtime behavior unless status is `authoritative` and owner spec grants behavior. |
| Lifecycle status | Packages, profiles, policies, lifecycle machines, validation artifacts, activation-controlled records. | `030`, `100`, specific owner specs. | Production eligibility state shared across activation-controlled artifacts. | Transition behavior without lifecycle machine. |
| Gold fact assertion state | `GoldFact` rows. | `040`, corrections in `080`. | Current assertion meaning of a bitemporal fact. | UI state, graph state, source completeness, identity decision. |
| Lakehouse read receipt state | `LakehouseReadCompletenessReceipt`. | `020`. | Whether Cadastre read a declared lakehouse feed target completely or failed/partially read it. | Source absence or source completeness by itself. |
| Completeness decision | Source/feed completeness evaluation. | `060`. | Whether source-level completeness may be considered under active profiles. | Source authority, identity, graph truth, or absence without additional gates. |
| API/source/export state labels | API, UI, evidence drillback, compliance export, audit export, analysis output. | `110`, source domain owners. | User-facing state separation. | Authorized negative facts or compliance pass/fail unless owner output exists. |
| Future reachability result state | Inactive future reachability results. | `200`. | Deferred analysis state for modeled reachability. | Active graph or gold output before promotion. |

| State token | Applies to | Meaning | Terminal | Default transition authority | Owner |
| --- | --- | --- | --- | --- | --- |
| `draft` | Spec status | Authoring text not migration-complete. | No. | Index/migration process. | `000` |
| `migration_active` | Spec status | Generated or migrated text under active decomposition; not implementation authority except as migration evidence. | No. | Index/migration process. | `000` |
| `candidate_authoritative` | Spec status | Reviewed candidate with owned PRD rows dispositioned; controlled spikes only. | No. | Index/migration process. | `000` |
| `authoritative` | Spec status | Acceptance criteria pass and index marks active. | No. | Index/migration process. | `000` |
| `superseded` | Spec status | Replaced by newer document or version. | Yes for replaced version. | Index/migration process. | `000` |
| `archived` | Spec status | Historical traceability only. | Yes. | Index/migration process. | `000` |
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

| Name or identifier class | Domain target | Stable use | Not this | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| Canonical domain identity | Product-owned canonical entity or fact identity. | Stable reference inside Cadastre outputs and evidence chains. | Source-native ID, graph backend ID, display label. | `040`, `070`, `080` | Resolved. |
| Source-native identity | Identifier emitted by external source or supplier feed. | Evidence input scoped by source/feed and resolver/authority policy. | Cadastre canonical identity by default. | `040`, `070`, `020` | Resolved. |
| Source/feed scope key | Scope boundary for feed, supplier, source dataset, resolver, authority, completeness, coverage. | Defines where source evidence is valid and comparable. | Global identity by itself. | `020`, `060`, `070` | Resolved. |
| Raw record ID | `RawRecord`. | Deterministic raw evidence reference. | Source completeness or canonical identity. | `040`, `020` | `TODO:` exact input ordering unresolved in `020`. |
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
| `DOM-INV-004` | Source authority must not imply identity merge authority. | `060`, `070` | Identity merge requires active `ResolverProfile` and no hard blockers. |
| `DOM-INV-005` | Identity evidence must be materialized before resolver output. | `070` | Resolver run rejects uncovered or under-scoped evidence. |
| `DOM-INV-006` | IP-only, hostname-only, DNS-only, PTR-only, graph-key-only, source-native-merge-only, and mapped-target-only evidence must never auto-merge. | `070` | Required negative identity validation cases fail merge. |
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

## 17. Defaults and omitted cases with domain significance

| Domain area | Default or omitted-case behavior | Owner | If omitted source evidence is missing |
| --- | --- | --- | --- |
| Root domain authority | Default: `domain.md` is a generated root-domain candidate and may not drive implementation until listed and statused by the source-of-truth map. | `000`, `domain.md` | `TODO:` blocking authority decision. |
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

### 18.5 Domain model versus API/UI specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| API/UI | Request/response shapes, state labels, redaction, authorization, health, errors, audit. | Stable term mapping and label-versus-identity rule. | Treating UI labels, displayed order, page tokens, or labels as stable domain identity. |

### 18.6 Domain model versus validation and fixture specs

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Validation/fixtures | Validation matrices, fixtures, golden corpus, acceptance reports, migration acceptance. | Binary root-domain acceptance criteria. | Treating test fixture shape as domain behavior owner or using validation artifact to invent behavior. |

### 18.7 Domain model versus package and implementation details

| Adjacent area | What it owns | What root `domain.md` owns | Forbidden substitution |
| --- | --- | --- | --- |
| Packages/implementation | Package releases, package-set activation, trust, code modules, local functions, package names, build and deployment mechanisms. | Package-domain boundary and implementation-mapping exception. | Treating module/package/function/table/route names as domain definitions or production activation authority. |

### 18.8 Cadastre domain concepts versus external schema or external-system concepts

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
| `AnalysisFinding` | Analysis result, not default graph node. | May appear in graph only if future owner mapping grants it. | `130`, `090` | No default graph authority. |
| `ReachabilityResult` | No active graph concept in MVP. | Must not project until `ReachabilityClaimPolicy` is active. | `200`, `090` | Inactive deferred. |

### 19.2 Domain relationship to graph edge family

| Domain relationship | Graph edge family | Required mapping | Owner | Status |
| --- | --- | --- | --- | --- |
| Observed network relationship | `observed_connection` or owner-defined edge type. | Must derive from qualifying gold fact and flow-role evidence; must not imply theoretical reachability. | `090`, `050`, `080` | Edge details owned by `090`; MVP reachability blocked. |
| Identity relationship | Owner-defined identity/projected relationship edge. | Must derive from identity decisions and/or gold facts through projection profile. | `070`, `090` | Requires graph profile row. |
| Asset ownership/containment/use | Owner-defined edge type. | Must derive from gold facts or declared synthetic structural rules. | `080`, `090` | Requires graph edge semantics row. |
| Threat-intel enrichment relationship | Analysis/enrichment edge only if declared. | Must not become gold/graph authority without separate contract. | `130`, `090` | No default authority. |
| Future reachability relationship | `has_theoretical_reachability` or equivalent. | Forbidden in MVP. Future activation requires `ReachabilityClaimPolicy`. | `200`, `090` | Inactive deferred. |
| TODO: Complete active edge-type list | TODO: exact graph edge family rows. | Required source: active `GraphEdgeSemantics` registry for MVP graph profile. | `090` | Blocking for exhaustive edge-family map. |

### 19.3 Source evidence concept to Cadastre observation or fact concept

| Source evidence concept | Cadastre mapping | Must not map to | Owner | Status |
| --- | --- | --- | --- | --- |
| Source-native payload | `RawRecord` payload and metadata. | Gold fact or canonical identity. | `020`, `040` | Resolved. |
| Source-native event/object field | Parser output then `CadastreSilverObservation` field. | Authority or identity by field name alone. | `050`, `040` | Resolved. |
| Supplier completeness signal | `UpstreamCompletenessEvidence`, then profile evaluation. | Absence/cleanup/retraction by itself. | `020`, `060` | Resolved. |
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
| Spec status | `draft`, `migration_active`, `candidate_authoritative`, `authoritative`, `superseded`, `archived`, `inactive_deferred` | Documentation authority and implementation eligibility. | `000` | Resolved with PRD status contradiction TODO. |
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
| `permission_limited` | Visibility-limited observation/state. | `040`, `060`, `110` | Absence or not applicable. | Resolved. |
| `conflicted` | Concurrent evidence or fact conflict. | `040`, `080`, `110` | Error or resolved fact. | Resolved. |
| Display name/title | Owner-specific display label. | `110` plus concept owner | Stable ID. | Resolved. |

### 19.7 Private source binding to public vendor-neutral concept

| Private concern | Public vendor-neutral concept | Boundary | Owner | Status |
| --- | --- | --- | --- | --- |
| Concrete vendor/product/source route | `PrivateSourceFeedBinding` to `LakehouseFeedProfile`. | Private artifact only; must not appear in public canonical model. | `010`, `020` | Resolved. |
| Private observed source fields | `PrivateFeedSchemaInventory` and source schema import/mapping artifacts. | Private inventory may inform mappings; public outputs require declared profiles. | `050`, `020` | Resolved. |
| Private completeness capabilities | `PrivateCompletenessEvidenceInventory` and `UpstreamCompletenessEvidence`. | Supplier evidence must carry authority limit; profile decides effect. | `020`, `060` | Resolved. |
| Private golden examples | Redacted `LakehouseFeedFixture` when public validation needs evidence. | Direct enterprise API recordings must not replace lakehouse fixtures. | `120`, `020` | Resolved. |

## 20. External-system and implementation boundaries

| External or adjacent concern | Domain boundary | Required language | Forbidden language |
| --- | --- | --- | --- |
| Enterprise source systems | External suppliers or feeds may provide evidence; Cadastre production must not collect directly. | “lakehouse-resident feed evidence,” “supplier-provided metadata.” | “Cadastre polls source API in production,” unless validation-only source exploration. |
| Lakehouse product/catalog | System-of-record substrate and table-state refs are Cadastre-governed. | “`LakehouseSnapshotRef` identifies authoritative read state.” | “table snapshot time is fact time,” “catalog branch is approval.” |
| OCSF | External schema profile for `normalized_fields`. | “OCSF-aligned normalized fields through active profile.” | “OCSF object is Cadastre entity,” “OCSF observable is graph node.” |
| Splunk CIM | Lossy deterministic projection target. | “CIM projection with loss manifest.” | “CIM is silver source of truth.” |
| Semantic overlays and Taxi-like tooling | Non-authoritative authoring and validation aids. | “semantic overlay supports mapping validation.” | “semantic type is identity evidence by itself.” |
| External graph systems | Cautionary references and taxonomy inputs only when mapped. | “external graph taxonomy translation.” | “external graph key resolves identity,” “traversable edge proves reachability.” |
| Graph backends | Replaceable read-model apply/query targets. | “backend profile and preflight.” | “Neo4j/Memgraph/JanusGraph/ArangoDB defines Cadastre semantics.” |
| Package registries and signing systems | Evidence sources for package acquisition/trust. | “package signature verification result under trust policy.” | “valid signature means active package.” |
| Threat-intel systems | Enrichment context by default. | “threat-intel enrichment record.” | “indicator equality creates asset identity.” |
| Lineage and metadata systems | Governance/lineage metadata unless mapped. | “lineage facet mapping policy.” | “facet proves evidence or completeness.” |
| UI labels | Presentation and state communication. | “user-facing label maps to stable domain term.” | “label is ID,” “visible order determines behavior.” |
| Implementation modules/functions/routes | Implementation details unless owner spec makes them observable. | “module implements domain term.” | “module name defines domain term.” |
| Storage artifacts/object keys | Implementation/persistence realization. | “storage key maps to artifact ref when owner permits.” | “object-store key is evidence identity by default.” |
| Research reports | Supporting evidence only. | “research report informed owner spec.” | “research report grants active behavior.” |
| Generated drafts | Migration evidence until statused. | “generated draft pending owner review.” | “draft is authoritative.” |
| Future reachability analyzers | Deferred analysis evidence only. | “inactive future reachability candidate.” | “reachable,” “not reachable,” “has theoretical reachability” in MVP. |

## 21. Normative requirements

Root-domain-owned requirements only:

| ID | Requirement | Owner | Verification method |
| --- | --- | --- | --- |
| `DOM-REQ-001` | `domain.md` must define canonical Cadastre terms without importing template-specific domain concepts. | `domain.md` | Review vocabulary and exclusion note. |
| `DOM-REQ-002` | `domain.md` must route each high-risk term to exactly one primary owner or mark it with `TODO:`. | `domain.md` | Owner columns in Sections 7, 10, 11, 12, 15. |
| `DOM-REQ-003` | `domain.md` must not add runtime behavior outside vocabulary interpretation, boundary rules, and owner lookup. | `domain.md`, `000` | Diff review against owner specs. |
| `DOM-REQ-004` | `domain.md` must treat implementation modules, storage artifacts, UI labels, graph backend labels, package names, route names, and external-system terms as mappings, not definitions. | `domain.md` | Boundary tables in Sections 18 and 20. |
| `DOM-REQ-005` | `domain.md` must surface unresolved contradictions and missing owner decisions as `TODO:` entries. | `domain.md` | Section 25 contains each blocking ambiguity. |
| `DOM-REQ-006` | `domain.md` must preserve the Cadastre source-of-truth map and must not silently promote migration-active specs, research, or drafts into implementation authority. | `domain.md`, `000` | Sections 1 and 5 state status limits. |
| `DOM-REQ-007` | `domain.md` must define graph-domain concepts without making graph state authoritative. | `domain.md`, `090` | Section 13 and graph boundary checks. |
| `DOM-REQ-008` | `domain.md` must define identity naming classes without permitting weak evidence to auto-merge. | `domain.md`, `070` | Section 15 matches resolver boundaries. |
| `DOM-REQ-009` | `domain.md` must define external-system boundaries without allowing external schema, taxonomy, lineage, registry, package, or analyzer terms to become Cadastre authority by default. | `domain.md` | Section 20 table. |
| `DOM-REQ-010` | `domain.md` must include binary Definition of Done criteria sufficient to judge root-domain completeness. | `domain.md` | Section 26. |

## 22. Intentional implementation latitude

| Area | Latitude granted | Boundary | Reason it is caller-equivalent |
| --- | --- | --- | --- |
| Markdown file placement before index update | Implementers may place the generated root document in any repo path for review. | It must not claim source-of-truth status until `000` maps it. | Path does not affect domain semantics before authority assignment. |
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
- use canonical Cadastre terms from Section 10;
- consult the primary owner spec before asserting behavior;
- use stable identifiers instead of labels, route names, module names, graph backend IDs, storage keys, package names, or external-system identifiers;
- treat implementation modules and storage artifacts as mappings, not definitions;
- use `TODO:` for unresolved owner decisions, contradictions, missing source evidence, or missing mapping rows;
- reject generated text that defines high-risk terms without owners;
- reject text that imports template-specific concepts, workflows, identifiers, entity models, lifecycle states, or acceptance criteria;
- reject text that creates synonyms for exact Cadastre tokens unless an owner spec defines the synonym and canonical replacement;
- reject text that turns future, deferred, research, external, generated, draft, or implementation content into active domain authority;
- reject graph, CIM, OCSF, lineage, metadata, analysis, package, or UI wording that implies authority not granted by an owner spec;
- reject source absence, identity merge, graph traversal, reachability, package activation, or lifecycle claims that lack the required owner profile or state machine.

## 24. Maintenance and drift-control rules

`domain.md` must be updated, or an explicit “domain vocabulary unchanged” statement must be recorded, when a change modifies any of the following:

- core entity membership;
- domain relationship families;
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
- Deprecated terms must name canonical replacements and migration owner.
- Near-duplicate terms are defects unless the distinction is explicit in Section 8 or Section 10.
- Research report language must be translated through owner specs before it appears as active Cadastre language.
- External product names must remain external references unless an owner spec declares a vendor-neutral contract mapping.
- `TODO:` rows must not be deleted without an owner decision or source update that resolves them.

## 25. Unresolved questions

| ID | Question | Blocking scope | Required source or owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `DOM-TODO-001` | TODO: Add `domain.md` to the Cadastre source-of-truth map and assign status, owner, dependencies, imports, and exports. | Root domain authority. | `000` owner decision. | `domain.md` remains generated candidate vocabulary evidence only. |
| `DOM-TODO-002` | TODO: Resolve conflict between uploaded PRD `status: archived` and source-of-truth map statement that original PRD remains upstream intent until NLSpec set is authoritative. | Authority and migration status. | `000` owner decision and PRD/archive disposition. | Do not treat either statement as silently superseding the other. |
| `DOM-TODO-003` | TODO: Confirm whether active `migration_active` NLSpec files may be used as owner specs for generated root-domain vocabulary before authoritative cutover. | Owner lookup strength. | `000` and `120` acceptance decision. | Use them as migration evidence and owner candidates; do not claim implementation authority. |
| `DOM-TODO-004` | TODO: Finalize exact raw feed manifest schema and raw record identity input ordering. | Raw record identity and replay. | `020` owner decision. | Raw identity basis remains conceptually defined but exact ordering is blocking. |
| `DOM-TODO-005` | TODO: Complete source-category-specific coverage dimension catalog. | Coverage-sensitive absence and negative claims. | `060` owner decision. | Coverage-dependent negative claims remain blocked where dimensions are unresolved. |
| `DOM-TODO-006` | TODO: Finalize old/new table snapshot roles, table-set checksums, and retention-protection rows by correction class. | Gold correction and replay. | `080` owner decision. | Affected correction classes remain blocked. |
| `DOM-TODO-007` | TODO: Finalize replay equivalence policy catalog. | Production replay and deterministic rebuild. | `080` owner decision. | Production replay remains blocked for unresolved output classes. |
| `DOM-TODO-008` | TODO: Complete active MVP observation type to OCSF category/class/activity/type mapping rows. | External schema mapping and silver validation. | `050` owner decision. | Any missing row is a blocking `TODO:` for that observation type. |
| `DOM-TODO-009` | TODO: Provide exhaustive active graph edge family registry and edge semantics rows for the MVP graph profile. | Relationship-family and graph mapping completeness. | `090` owner decision. | Graph edge mapping remains only concept-level; missing edge rows block activation. |
| `DOM-TODO-010` | TODO: Confirm lifecycle transition tables for every production-affecting lifecycle machine. | Lifecycle behavior. | `030` plus package/profile owner decisions. | `LifecycleStatus` tokens remain shared names only. |
| `DOM-TODO-011` | TODO: Confirm whether `domain.md` must export any named records or only vocabulary/owner maps. | Spec index and imports/exports. | `000` owner decision. | Treat `domain.md` as no-runtime-export vocabulary reference. |
| `DOM-TODO-012` | TODO: Confirm exact canonical filename and path for root domain document. | Repository placement and citations. | Project owner decision. | Generated artifact is `cadastre-domain.md` in this workspace, not an authoritative repo path. |

A downstream implementation must not resolve a `TODO:` by inference.

## 26. Definition of Done

| ID | Criterion |
| --- | --- |
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
| `DOM-AC-014` | No Cadastre requirement is invented from the uploaded template. |
| `DOM-AC-015` | Every unresolved ambiguity that affects interoperability is surfaced as `TODO:` in Section 25. |
| `DOM-AC-016` | The document passes the two-independent-implementers test for domain vocabulary, owner lookup, entity interpretation, relationship interpretation, and boundary decisions. |
| `DOM-AC-017` | The document passes the recreatability test for the root Cadastre domain model and owner map without relying on the uploaded template. |
| `DOM-AC-018` | The document is small enough for prompt context but complete enough to prevent semantic drift at root-domain level. |
| `DOM-AC-019` | Every source in the source traceability table was inspected, and no unseen text is claimed as read or verified. |
| `DOM-AC-020` | The root-domain owner map includes every supplied Cadastre source-of-truth or adjacent artifact inspected for this generation. |

## Source traceability

| Source | Section or heading | Used for | Limits |
| --- | --- | --- | --- |
| `PRD-Cadastre..md` | Product Summary, lines 8-52; Document Status, lines 54-104 | Domain thesis, lakehouse-fed boundary, projection non-authority, research non-authority boundary, public/private feed concepts. | Uploaded file name has double dot; front matter says `status: archived`, which conflicts with `000` migration language and is reported as `TODO:`. |
| `000-cadastre-spec-index.md` | Authority, lines 13-17; Purpose, lines 19-21; Document Status Vocabulary, lines 41-51; Authoritative Source Rule, lines 53-63; Source-of-Truth Map, lines 65-83; Dependency Matrix, lines 91-109 | Authority model, owner map, status vocabulary, migration limits, research non-authority. | Does not list root `domain.md`; this is reported as `TODO:`. |
| `010-system-boundary-and-authority.md` | Boundary Contract, lines 43-55; Authority Classes, lines 57-68; Projection Authority Rule, lines 70-74; Public and Private Source Binding, lines 76-87; Cross-Domain Invariants, lines 89-97 | Product boundary, authority classes, projection boundary, private/public source binding, invariants. | Migration-active candidate, not marked authoritative in supplied index. |
| `020-lakehouse-feeds-and-table-state.md` | Purpose, lines 19-21; Exports, lines 36-55; Feed Read Contract, lines 57-70; Completeness Receipt Contract, lines 79-94; Table State Contract, lines 96-109; Maintenance Safety, lines 111-118 | Lakehouse feed terms, raw import, completeness receipt, table-state refs, maintenance and retention boundaries. | Contains unresolved raw record identity ordering TODO. |
| `030-processing-dag-lifecycle-and-versioning.md` | Stage Graph Contract, lines 56-71; Lifecycle Contract, lines 73-87; Run Locks, lines 89-91; Version Manifest Contract, lines 93-97; Execution Algorithm, lines 103-119 | Processing stage, lifecycle, version manifest, run lock, output permission vocabulary. | Full lifecycle transition tables are owner-specific and not copied. |
| `040-canonical-data-observation-and-fact-model.md` | Canonical Serialization, lines 53-67; Omission Semantics, lines 69-83; Core Records, lines 85-95; Assertion States, lines 97-109; Graph Delta Primitive Shapes, lines 111-113 | Core records, entity definitions, omission states, assertion states, graph delta primitive boundary. | Field schemas are not duplicated; behavior remains owned by this spec. |
| `050-normalization-external-schema-and-mapping.md` | Parser Contract, lines 59-71; Silver Normalization Contract, lines 73-77; External Schema Profile Contract, lines 79-83; OCSF Mapping Requirements, lines 85-94; Source Extension Fields, lines 96-98; CIM Projection Contract, lines 114-116 | Parser/silver/external schema/OCSF/CIM/source extension vocabulary and boundaries. | Exact MVP OCSF mapping rows are not supplied in this root doc. |
| `060-source-authority-completeness-coverage-and-absence.md` | Source Authority Contract, lines 58-62; Completeness Contract, lines 64-80; Progress Signal Interpretation, lines 82-94; Coverage Contract, lines 96-100; DeriveAbsenceOrUnknown, lines 102-114; Watermarks, lines 116-118 | Source authority, completeness, progress signal, coverage, absence, watermark vocabulary and non-implication rules. | Coverage dimension catalog has an explicit TODO in source. |
| `070-identity-resolution-and-target-selectors.md` | Resolver Authority, lines 57-61; Evidence Roles, lines 63-79; Candidate Generation, lines 81-83; Hard Blocker Precedence, lines 85-99; ResolveIdentity, lines 101-117; Review Contract, lines 119-121; Target Selector Safety, lines 123-127 | Identity evidence, resolver authority, weak-evidence defaults, hard blockers, review, target selector boundaries. | Does not define all concrete resolver matrix rows in root doc. |
| `080-temporal-corrections-replay-and-gold-derivation.md` | Temporal Axes, ResolveFactTime, Gold Derivation, Correction, Late Arrival, Replay, Deterministic Side Effects, lines 57-107 | Temporal/gold/correction/replay concepts, current-time fallback prohibition, append-only correction boundary. | Contains TODOs for correction snapshot roles and replay equivalence catalog. |
| `090-graph-projection-serving-and-backends.md` | Projection Authority, lines 70-74; Graph Delta Identity, lines 76-78; Edge Semantics and Traversal, lines 80-84; Backend Preflight, lines 86-88; QueryGraph Contract, lines 104-125; Derived View Lag, lines 127-129; Graph Rebuild, lines 131-133; Drift Check Boundary, lines 135-137 | Graph concepts, graph non-authority, graph ID, edge semantics, traversal, backend, query, lag, rebuild, drift boundaries. | Active edge-type registry is not reproduced; missing exhaustive edge mapping remains TODO. |
| `100-packages-supply-chain-and-activation.md` | Activation Unit, Package Release Manifest, Package Set Manifest, Trust Verification, Activation Algorithm, Rollback and Emergency Behavior, lines 65-121 | Package artifact versus activation, package-set authority, trust verification, fail-closed activation, rollback boundaries. | Registry/product choices and package policies remain owner-specific. |
| `110-api-ux-health-errors-and-security.md` | Roles, API Bounds, Source and Export State Labels, Error Model, Evidence Drillback, Security and Redaction, lines 58-126 | User-facing state labels, API/UI label boundary, errors, evidence drillback, redaction. | API shapes are not duplicated beyond domain-level terms. |
| `120-validation-fixtures-and-acceptance.md` | Validation Ownership Rule, Validation Artifact Model, Required Negative Test Classes, Acceptance Aggregation, Migration Acceptance Criteria, lines 57-95; Definition of Done, lines 97-105 | Validation/acceptance vocabulary, binary acceptance criteria, migration acceptance, negative test classes. | Test implementation mechanics are not included. |
| `130-analysis-enrichment-and-registry-governance.md` | Non-Authority Rule, lines 64-66; Analysis Rules, lines 68-72; Derivation Rule Boundary, lines 74-76; Threat-Intel Enrichment, lines 78-82; Lineage and Artifact Boundary, lines 84-88; Registry Governance, lines 90-92 | Analysis, enrichment, lineage, artifact class, registry governance boundaries. | Does not grant authority beyond owner-specified interfaces. |
| `200-future-reachability-analysis-domain.md` | Authority, lines 13-19; Explicit Non-Scope, lines 21-29; Preserved Contract Candidates, lines 31-42; Claim Kinds, lines 44-52; Result States, lines 54-63; Prohibited MVP Outputs, lines 65-71 | Inactive future reachability domain, result-state vocabulary, prohibited MVP reachability outputs. | Inactive deferred candidate; no active MVP implementation effect. |
| `nlspec-spec.md` | Nature of Artifact, lines 11-25; Why NLSpecs Work, lines 28-72; What an NLSpec Is Not, lines 75-89 | NLSpec quality bar: prescriptive language, two-independent-implementers test, interfaces, defaults, mapping tables, acceptance criteria. | Specification-quality standard only; no Cadastre product behavior. |
| Uploaded `domain.md` template | Status/authority and relationship-map pattern, lines 1-80 inspected | Structure, voice, and pattern source only. | Non-authoritative for Cadastre; template domain terms excluded. |
| `RES-001-cartography.md` | Front matter and Metadata, lines 1-16 | Research-report status and graph-system adjacency row. | Not used to define Cadastre requirements. |
| `RES-002-jupiterone.md` | Executive Summary, lines 1-18 | Research-report status and graph/identity adjacency row. | Not used to define Cadastre requirements. |
| `RES-003-taxi-lang.md` | Executive verdict, lines 1-18 | Research-report status and semantic-overlay/tooling adjacency row. | Not used to define Cadastre requirements. |
| `RES-004-ocsf-schema.md` | Executive summary, lines 1-18 | Research-report status and OCSF adjacency row. | Not used to define Cadastre requirements beyond owner-spec-adopted concepts. |
| `RES-005-Lakehouse-Derived-Graph-Architecture.md` | Executive summary, lines 1-10 | Research-report status and lakehouse/derived graph adjacency row. | Not used to define Cadastre requirements. |
| `RES-006-source-adapter-ingestion-patterns.md` | Front matter and Executive summary, lines 1-17 | Research-report status and ingestion-pattern adjacency row. | Not used to define Cadastre requirements; owner specs provide authoritative adopted terms. |
| `RES-007-normalization-and-schema-standards.md` | Front matter and Executive verdict, lines 1-20 | Research-report status and normalization-schema adjacency row. | Not used to define Cadastre requirements; owner specs provide authoritative adopted terms. |
| `RES-008-identity-graph-attack-paths.md` | Front matter and Executive verdict, lines 1-20 | Research-report status and attack-path/graph adjacency row. | Not used to define Cadastre requirements; owner specs provide authoritative adopted terms. |
| `RES-009-identity-resolution-governance.md` | Front matter and Executive summary, lines 1-20 | Research-report status and identity-governance adjacency row. | Not used to define Cadastre requirements; owner specs provide authoritative adopted terms. |
| `RES-010-theoretical-reachability-network-context.md` | Executive verdict, lines 1-18 | Research-report status and reachability adjacency row. | Not used as active behavior authority. |
| `RES-011-temporal-semantics.md` | Source limits summary, lines 1-16 | Research-report status and temporal/replay adjacency row. | Not used as active behavior authority. |
| `RES-012-graph-backend-comparison.md` | Executive verdict, lines 1-18 | Research-report status and graph-backend adjacency row. | Not used for backend selection. |
| `RES-013-extension-package-supply-chain.md` | Executive Verdict, lines 1-18 | Research-report status and package-supply-chain adjacency row. | Not used for package registry or mechanism selection. |
| `RES-014-source-authority-staleness-coverage-absence.md` | Executive summary, lines 1-18 | Research-report status and source-authority adjacency row. | Not used as direct authority. |
