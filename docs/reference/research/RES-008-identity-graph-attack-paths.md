---
doc_id: RES-008
project: Cadastre
title: Identity Graph and Attack Paths Research Report for Cadastre
status: research-report
inspection_date: 2026-05-16
---

## 1. Executive verdict

[Cadastre applicability inference] BloodHound and SharpHound are high-value references for **identity-path semantics**, not for Cadastre’s authoritative model. The strongest transferable ideas are:

| Rank | Transferable concept | Transfer class | Cadastre contract to strengthen |
| ---: | --- | --- | --- |
| 1 | Explicit traversability as a first-class edge property | `adopt` | `GraphEdgeSemantics`, `GraphProjectionProfile`, `AnalysisRuleBundle` |
| 2 | Separation between generic graph ingestion, structured graph semantics, pathfinding eligibility, findings, and metrics | `adopt` | `GraphProjectionProfile`, `GraphReadModelSchemaProfile`, `GraphTaxonomyTranslationPolicy` |
| 3 | Stable ID over property/name endpoint matching, with name matching deprecated | `adopt` | `TargetSelectorSafetyPolicy`, `UnresolvedTargetReference`, `IdentityDecision` |
| 4 | Post-processed edges as generated path edges from supporting facts, not primary evidence | `adapt` | `DerivationRuleBundle`, `GraphDeltaSet`, `GoldFact` |
| 5 | Source-kind deletion and source ownership hazards | `adopt` as safety constraint | `SourceAsset`, `SourceInstance`, `GraphApplyProfile`, `GraphDeltaSet` |
| 6 | Collector output metadata, methods bitmask, type, count, version, ZIP output | `adapt` | `RawRecord`, `RecordedSourceFixture`, `VersionManifest` |
| 7 | Permission-dependent and volatile collection visibility | `adopt` | `SourceCompletenessProfile`, `SourceCompletenessReceipt` |
| 8 | Pathfinding UI ergonomics: reverse path, edge filtering, saved queries, object panels | `adapt` | `QueryGraph`, `AnalysisRuleBundle`, UI requirements |

[Cadastre applicability inference] The central boundary must remain unchanged: Cadastre must not adopt BloodHound’s graph as truth, SharpHound output as facts, OpenGraph endpoint matching as identity resolution, BloodHound pathfinding as risk scoring, or source-kind deletion as authoritative cleanup. The Cadastre PRD already states that the lakehouse is the system of record, graph output is a replaceable projection, and Cadastre must not adopt direct-to-graph or graph-authoritative synchronization.[^1]

[confirmed source fact] BloodHound CE v9.1.0 is a monolithic web application with an embedded React frontend, a Go REST API backend, a PostgreSQL application database, and a Neo4j graph database according to the repository README; its inspected release tag is `v9.1.0`, commit `6b44cc3`, released May 5, 2026.[^5]

[confirmed source fact] SharpHound v2.12.0 is the inspected collector release, tagged at commit `7faa2bb`, and its repository and docs identify it as the official C# data collector for BloodHound CE.[^6]

## 2. Source inventory and inspection limits

No repository was cloned, built, tested, or executed locally. Inspection used current public repository pages, raw source files, release metadata, and official BloodHound documentation on **2026-05-16**. Claims about runtime behavior are limited to inspected source and documentation. Offensive abuse instructions, evasion guidance, and unauthorized collection procedures were intentionally excluded.

| Source name | Source URL or identifier | Evidence type | Commit, tag, branch, release, or version | Inspection date | Scope inspected | Scope not inspected | Reliability | Freshness risk |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BloodHound repository | `SpecterOps/BloodHound` | `source_code`, `release_metadata` | `v9.1.0`, commit `6b44cc3` | 2026-05-16 | Root tree, README, `cmd/api/src`, `cmd/ui`, `schemas`, UI package, Go module, dev/test compose files | Full source, migrations internals, local build, tests, runtime, API behavior | High for repo layout and declared architecture; medium for unexecuted behavior | Medium: actively maintained release line |
| BloodHound frontend source | `cmd/ui` | `source_code` | `v9.1.0` | 2026-05-16 | UI package dependencies, `src` layout, React/Vite/Sigma/Graphology surface | Component-level graph code not deeply inspected | Medium-high | Medium |
| BloodHound backend source | `cmd/api/src` | `source_code` | `v9.1.0` | 2026-05-16 | API directory layout, Go module dependencies, database and services package presence | Handler-level semantics, migrations, DAWGS internals, API tests | Medium | Medium |
| BloodHound schemas | `schemas`, `schemas/open_graph` | `schema`, `example` | `v9.1.0` | 2026-05-16 | `valid_edges.json`, OpenGraph dog-park schema/data example | Full JSON schema definitions not exposed under guessed filenames; schema validation runtime | Medium | Medium |
| SharpHound repository | `SpecterOps/SharpHound` | `source_code`, `release_metadata` | `v2.12.0`, commit `7faa2bb` | 2026-05-16 | Root tree, README, `src` layout, `.csproj`, `Options.cs`, `Sharphound.cs`, writer/producer/runtime/client dirs | Full source, SharpHoundCommon dependency internals, local build, output generation | Medium-high for CLI/source layout; medium for output internals | Medium |
| SharpHound CE docs | Official BloodHound docs | `official_docs` | Current docs, version not stated | 2026-05-16 | Collection methods, flags, output options, permissions, data retention/completeness material | Collector runtime and actual output samples | High for documented behavior | Medium |
| BloodHound JSON format docs | Official BloodHound API docs | `official_docs`, `schema` | Current docs, version not stated | 2026-05-16 | JSON `data` and `meta` format; methods/type/count/version | Detailed per-node JSON examples beyond linked repo directory | High for documented ingest envelope | Medium |
| OpenGraph docs | Official BloodHound docs | `official_docs`, `schema` | Current docs, version not stated | 2026-05-16 | Graph data overview, graph definition, nodes, edges, metadata, API, FAQ, extension management | Runtime validation code, community extensions | High for documented extension contract | Medium-high: OpenGraph is active and still evolving |
| BloodHound Explore/Cypher docs | Official BloodHound docs | `official_docs` | Current docs, version not stated | 2026-05-16 | Search, pathfinding, Cypher, supported Cypher syntax | Actual query engine limits, ordering, timeout internals | High for documented UI behavior; medium for omitted bounds | Medium |
| BloodHound findings/posture/risk docs | Official BloodHound docs | `official_docs` | Enterprise docs, current version not stated | 2026-05-16 | Attack paths, exposure/impact, posture, risk acceptance, privilege zones | Backend scoring implementation, CE applicability for Enterprise-only pages | Medium-high; Enterprise-only caveat | Medium |
| Cadastre PRD | `/mnt/data/PRD-Cadastre.md` | Project baseline | Draft uploaded in project context | 2026-05-16 | Architecture, graph query, completeness, edge semantics, non-scope | Not an external evidence source | High for Cadastre comparison baseline | Medium: draft may change |
| NLSpec guidance | `/mnt/data/nlspec-spec.md` | Project standard | Version `0.2.2` | 2026-05-16 | Behavioral completeness, interfaces, defaults, mappings, acceptance criteria | Not an external evidence source | High for report format and recommendation standard | Low-medium |

## 3. Cadastre baseline and non-transferable boundaries

[confirmed source fact] Cadastre’s PRD defines the lakehouse as the system of record, the graph and Splunk CIM output as replaceable projections, and projection as the deterministic contract between authoritative records and serving output.[^1]

[confirmed source fact] The PRD prohibits direct-to-graph or graph-authoritative synchronization, and it defines separate contracts for source configuration, collection stages, source completeness, unresolved target references, selector safety, graph projection, graph read-model schema, graph apply, graph edge semantics, taxonomy translation, and validation.[^1]

[confirmed source fact] Cadastre’s graph query contract already requires neighbor expansion, bounded path search, filters by node type, edge type, assertion state, confidence, valid time and known time, evidence inclusion, current-state defaults, maximum depth, page-size bounds, timeouts, and deterministic path ordering.[^2]

[confirmed source fact] Cadastre’s completeness contract requires `SourceCompletenessReceipt` plus an active `SourceCompletenessProfile`; only `complete` and `empty_complete` may authorize absence, retraction, cleanup, or source watermark advancement, and partial or unavailable states must block those effects.[^3]

[confirmed source fact] Cadastre’s `GraphEdgeSemantics` contract requires one row per graph edge type, direction rules, evidence requirements, assertion-state eligibility, temporal policy, confidence policy, non-implication rules, and no-op conditions.[^4]

Non-transferable boundaries:

| External pattern | Cadastre boundary |
| --- | --- |
| BloodHound graph database as truth | Graph serving is a replaceable read model only. |
| SharpHound JSON as facts | Collector output becomes raw evidence or silver observations only after Cadastre parsing and validation. |
| BloodHound/OpenGraph node ID equality | May create unresolved references; must not create canonical identity by itself. |
| OpenGraph property or name matching | Must not resolve identity; production name matching defaults to reject. |
| BloodHound traversable edge | May inform analysis eligibility; must not become Cadastre privilege fact without evidence and source authority. |
| BloodHound post-processed edge | May map to `DerivationRuleBundle`; must not be backend-authoritative fact. |
| BloodHound attack-path severity | Analysis output only unless a Cadastre risk-scoring contract exists. |
| BloodHound risk acceptance | User workflow record only; not remediation, retraction, deletion, or proof of risk reduction. |

## 4. BloodHound independent analysis

### 4.1 Repository and architecture

[confirmed source fact] The `SpecterOps/BloodHound` root includes architecture, API, UI, schemas, local harnesses, Docker files, examples, packages, tools, Go module files, Python packaging metadata, Yarn workspaces, and development compose files.[^5]

[confirmed source fact] The repository README describes BloodHound as a monolithic web application with an embedded React frontend using Sigma.js, a Go REST API backend, a PostgreSQL application database, and a Neo4j graph database.[^5]

[confirmed source fact] The backend `cmd/api/src` tree includes packages named `api`, `auth`, `bootstrap`, `cmd`, `config`, `ctx`, `daemons`, `database`, `migrations`, `model`, `queries`, `serde`, `services`, `test`, `utils`, `vendormocks`, and `version`.[^5]

[confirmed source fact] The frontend `cmd/ui/src` tree includes `components`, `ducks`, `hooks`, `mocks`, `rendering`, `routes`, `shared`, `styles`, `views`, and test files.[^5]

[confirmed source fact] The UI package uses Vite, React 18, Redux, React Query, Material UI, Sigma, React Sigma, Graphology, Dagre, and graphology layouts.[^7]

[confirmed source fact] The Go module declares module `github.com/specterops/bloodhound`, uses Go `1.26.2`, and includes dependencies for PostgreSQL access, Neo4j, AzureHound, DAWGS graph libraries, and Gorm/Postgres.[^8]

[source-grounded inference] Application database responsibilities are associated with PostgreSQL-backed API and application state, while graph traversal/storage responsibilities are associated with the graph backend and DAWGS/Neo4j/PostgreSQL graph behavior. The README still says Neo4j graph database, while official OpenGraph docs state full OpenGraph support requires the PostgreSQL graph database and that many OpenGraph features may work in Neo4j with limitations. This is an explicit backend-transition ambiguity, not a compatibility guarantee.[^5]

[confirmed source fact] Development prerequisites include Docker, Node, Yarn, Go, Python, and `just`; development startup uses `just init` and `just bh-dev`, and the UI is served locally.[^9]

[confirmed source fact] A testing compose file defines PostgreSQL and Neo4j services for test use.[^10]

### 4.2 Data model

[documentation fact] BloodHound supports Active Directory, Azure/Entra, and OpenGraph data. Search and pathfinding docs state that BloodHound supports all search methods for structured graphs, while generic graphs support Search and Cypher search only.[^11]

[documentation fact] OpenGraph data payloads model entities as nodes and relationships as edges. BloodHound validates JSON and minimum JSON schema but does not enforce additional structure unless an extension definition schema is supplied.[^12]

[documentation fact] OpenGraph distinguishes generic graphs from structured graphs: generic graphs support search, while structured graphs support advanced analysis and pathfinding when definitions are provided.[^12]

[documentation fact] OpenGraph node IDs must be stable and globally unique; the docs prefer source-system GUIDs and warn against usernames, emails, hostnames, or autoincrement IDs. Node `kinds` are arrays with up to three values, first kind controlling primary styling; structured graph nodes must match defined node kinds.[^13]

[documentation fact] OpenGraph node properties support primitive values or homogeneous primitive arrays; nested objects and arrays of objects are unsupported. Property names are case-sensitive, `lastseen` is injected or overwritten, `reconcile` is stripped, and certain property names such as `name`, `operatingsystem`, `distinguishedname`, and `environmentid` are uppercased during ingest.[^13]

[documentation fact] OpenGraph edge records have `start`, `end`, `kind`, and optional properties. Endpoint matching supports `id`, `property`, and deprecated `name` strategies; `id` is preferred, property matching is slower and used only when no node ID is known, and name matching is deprecated and internally maps to a `name` property match.[^14]

[documentation fact] OpenGraph `source_kind` may be registered and applied globally or by node kind; it is appended to node kinds during ingestion and is required for certain structured features such as findings and metrics.[^12]

### 4.3 Edge semantics and attack paths

[documentation fact] BloodHound defines traversable edges as relationships where the starting node can take control of the ending node to a degree that allows abuse of outgoing edges; pathfinding includes only traversable edges. Non-traversable relationships can combine into traversable post-processed edges such as `DCSync`.[^15]

[documentation fact] OpenGraph relationship definitions include `is_traversable`, and the docs state that only traversable edges are included in pathfinding and considered for findings and metrics.[^16]

[documentation fact] BloodHound post-processing deletes existing post-processed edges and regenerates them; directly uploaded post-processed edges do not persist.[^14]

[documentation fact] Example built-in edges have explicit direction and traversability. `MemberOf` is source user/group/computer to destination group and traversable; `HasSession` is source computer to destination user and traversable, while the page notes that a session does not guarantee credential material is present.[^17]

[documentation fact] `CanRDP` is generated only when a user has membership in Remote Desktop Users and the `SeRemoteInteractiveLogonRight`; the docs explicitly say the edge does not guarantee privileged execution.[^18]

[documentation fact] `AZHasRole` indicates an active Entra ID role assignment, but the docs state that it does not capture all abusable possibilities for actions authorized by that role.[^19]

[documentation fact] `SyncedToADUser` models synchronization from Entra user to on-prem AD user and is traversable from `AZUser` to `User`.[^20]

### 4.4 Query and pathfinding model

[documentation fact] The Explore UI supports Search, Pathfinding, and Cypher. Search supports partial matches and node-label prefixes; pathfinding supports reverse path search and edge filtering, and ETAC limits pathfinding results to environments a user can access.[^11]

[documentation fact] Cypher search supports saved/prebuilt queries, custom queries, query import/export in JSON, and query-library material.[^21]

[documentation fact] BloodHound documents supported openCypher components translated by CySQL, including node matching, relationship matching, recursive expansion, range expansion, shortest paths, all shortest paths, order, skip, limit, entity updates, property/label changes, and entity deletion.[^22]

[open question] The inspected docs do not specify deterministic result ordering for pathfinding beyond Cypher `order by`, default path depth limits, default timeout, result truncation, evidence inclusion behavior, bitemporal filtering, assertion-state filtering, confidence filtering, or graph-property redaction. For Cadastre, these are gaps, not compatibility.

### 4.5 OpenGraph extension model

[documentation fact] An OpenGraph extension definition includes schema, node kinds, relationship kinds, environments, and relationship findings. Extension definitions enable structured graph behavior, advanced analysis, pathfinding, findings, and metrics.[^16]

[documentation fact] OpenGraph structured findings/metrics require environment kind, source kind, environment ID, `collected = true`, and privilege zone rules.[^16]

[documentation fact] Relationship findings define finding instances based on relationship kinds and remediation guidance.[^16]

[documentation fact] Deleting an OpenGraph extension removes its definition schema but leaves underlying data; the data reverts to generic graph behavior unless separately deleted.[^23]

[documentation fact] OpenGraph CE supports manual file upload, while the API payload behavior differs by edition according to the extension-management docs.[^23]

### 4.6 Deletion, ownership, and source scoping

[documentation fact] The OpenGraph FAQ says data can be deleted by source kind, and warns that source-kind metadata can mark referenced AD/AZ nodes and cause them to be deleted when that source is deleted; the documented workaround is to upload referenced built-in nodes without defining `source_kind`.[^24]

[Cadastre applicability inference] Cadastre must treat this as a deletion anti-pattern. Source-scoped deletion may expire a source’s graph projection output, but must not delete or mutate `CanonicalEntity`, `IdentityDecision`, `GoldFact`, `SourceAsset` from another source, `Identifier` from another source, or authoritative lakehouse records.

## 5. SharpHound independent analysis

### 5.1 Repository and implementation architecture

[confirmed source fact] SharpHound v2.12.0 repository root includes `.github`, `Properties`, `src`, `PowerShell`, project/solution files, a quickstart notebook, and NuGet config. The README states the project can be compiled with .NET tooling and is designed to target .NET 4.7.2.[^6]

[confirmed source fact] The `src` tree includes `Client`, `PowerShell`, `Producers`, `Runtime`, `Writers`, `BaseContext.cs`, `Options.cs`, and `Sharphound.cs`.[^6]

[confirmed source fact] The project file identifies executable output, target framework `net472`, version `2.12.0`, and references SharpHoundCommon artifacts.[^25]

[confirmed source fact] `Options.cs` defines collection methods, domain/search scope, output directory/prefix, cache and memory-cache options, zip behavior, random filenames, pretty-printing, credentials, LDAP and LDAPS options, port-check timeout, throttling, jitter, thread count, loop duration, loop interval, and metrics options.[^26]

[confirmed source fact] `Sharphound.cs` checks for .NET 4.7.2 runtime, parses CLI options, resolves collection methods, validates required credential option pairings, initializes metrics when requested, and executes a chain of collection links.[^27]

### 5.2 Collection model

[documentation fact] SharpHound CE is the official BloodHound CE collector, written in C#, and uses native Windows API functions plus LDAP namespace functions to collect data from domain controllers and domain-joined Windows systems.[^28]

[documentation fact] Running with no flags auto-determines domain and domain controller and collects group membership, domain trusts, abusable rights, GPO links, OU tree, computer/group/user properties, SQL admin links, local admin/RDP/DCOM/remote-management groups, and active sessions.[^28]

[documentation fact] Supported collection-method families include Default, All, DCOnly, ComputerOnly, Session, LoggedOn, Group, ACL, GPOLocalGroup, Trusts, Container, LocalGroup, LocalAdmin, RDP, DCOM, PSRemote, ObjectProps, UserRights, CARegistry, DCRegistry, CertServices, WebClientService, LdapServices, SmbInfo, and NTLMRegistry.[^29]

[documentation fact] Domain selection, search-forest behavior, base DN selection, LDAP filter selection, computer-file scoping, domain-controller override, and collection of all LDAP properties are exposed as options.[^29]

[documentation fact] Session data is volatile; the docs explicitly support looped session collection because sessions are not always present during one collection window.[^28]

### 5.3 Output model

[documentation fact] BloodHound JSON has top-level `data` and `meta`. `meta.methods` is a collection-method bitmask, `meta.type` describes the data-array object type, `meta.count` is the number of objects, and `meta.version` is the JSON format version. One JSON file contains only one object type.[^30]

[documentation fact] Community Edition collectors write JSON data to disk for manual upload to BloodHound via UI or `/file-upload`; Enterprise collectors continuously upload to `/ingest`.[^30]

[documentation fact] SharpHound output options include output directory, output prefix, disabling ZIP, ZIP password, ZIP filename, random filenames, pretty JSON, and CSV tracking of computer calls.[^29]

[Cadastre applicability inference] SharpHound output should enter Cadastre as `RawRecord` payloads plus `VersionManifest` collector version and option records. `meta.count` must not become completeness proof. `meta.methods` may become collection-method metadata only.

### 5.4 Completeness and visibility

[documentation fact] Active Directory structure collection can be done by authenticated users for most AD data, but restricted read permissions require additional audit role configuration.[^31]

[documentation fact] Local group membership collection requires Remote SAM or local administrator-style permissions in many environments.[^31]

[documentation fact] Session collection uses `NetWkstaUserEnum`-style visibility and requires specific permissions depending on server or desktop context.[^31]

[documentation fact] User-right assignment collection matters because RDP ability requires both RDP group membership and the `SeRemoteInteractiveLogonRight`.[^31]

[documentation fact] BloodHound Enterprise retention docs state `HasSession` is generated as a pattern of behavior rather than a momentary active session, and that a single collection absence must not be assumed to mean an object or edge no longer exists.[^32]

[Cadastre applicability inference] Successful SharpHound execution must default to `unknown_visibility` or presence-only completeness unless collection scope, permissions, returned failures, expected counts, and method-specific evidence prove completeness under a Cadastre `SourceCompletenessProfile`.

## 6. Official BloodHound documentation and OpenGraph analysis

### 6.1 Generic versus structured graph

| OpenGraph state | Can ingest? | Can search? | Can pathfind? | Can produce findings? | Can produce metrics? | Can influence Cadastre identity? |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Generic graph payload | Yes | Yes | No by default | No | No | Default `false` |
| Structured graph with definition | Yes | Yes | Yes, only via traversable relationship kinds | Yes, with relationship findings and requirements | Yes, with source/environment/collected requirements | Default `false` |
| Structured graph node with stable source ID | Yes | Yes | Depends on edges | Depends on definitions | Depends on definitions | May create `UnresolvedTargetReference`; must not merge |
| Structured graph relationship with `is_traversable=false` | Yes | Yes | No | No pathfinding role | No traversal metric role | No |
| Structured graph relationship with `is_traversable=true` | Yes | Yes | Yes | Maybe | Maybe | No identity effect by default |
| Relationship finding | Yes, as analysis definition | Searchable/reportable | Not a fact | Yes | May affect metrics | Analysis output only |
| BloodHound attack-path finding | Yes, in BloodHound Enterprise | Yes | Analysis result | Yes | Yes | Analysis output only |

### 6.2 OpenGraph payload contract

[documentation fact] Minimal OpenGraph data has `graph.nodes` and `graph.edges`, each conforming to minimum schemas. Payloads may be `.json` or `.zip`, zip archives may contain multiple payload files, and nested zip files are not supported.[^12]

[documentation fact] The BloodHound repository’s OpenGraph dog-park example declares namespaced node kinds, relationship kinds, `is_traversable`, environments, and relationship findings; the data example uses `metadata.source_kind`, node `id`, `kinds`, `environmentid`, `collected`, and edges with endpoint `match_by = id`.[^33]

### 6.3 Query library and saved queries

[documentation fact] BloodHound docs describe a community-driven BloodHound Query Library that includes both community-contributed and prebuilt queries, and BloodHound UI supports saved queries, sharing, import, and export.[^21]

[Cadastre applicability inference] Cadastre should treat query-library content as `AnalysisRuleBundle` candidates. A query result must not derive `GoldFact` unless the query is promoted into a replayable `DerivationRuleBundle` with source facts and deterministic rule versioning.

## 7. Graph data model reconstruction

### 7.1 External model

[confirmed source fact] BloodHound’s model is a labeled property graph: nodes represent objects, edges represent relationships, and pathfinding traverses selected directional relationships. The Explore docs state nodes and edges are displayed in the graph, and OpenGraph docs define `nodes[]` and `edges[]` payloads.[^11]

### 7.2 Cadastre mapping

| External concept | Cadastre mapping | Required boundary |
| --- | --- | --- |
| BloodHound AD `User`, `Group`, `Computer` node | `CadastreSilverObservation` subject/object hints, later `CanonicalEntity` only after identity decision | Node does not create canonical identity by itself |
| OpenGraph node `id` | Observed stable identifier or source-native asset ID | May create `SourceAsset` or `Identifier`, not merge |
| OpenGraph node `kinds` | External taxonomy label | Governed by `GraphTaxonomyTranslationPolicy` |
| OpenGraph edge `kind` | Candidate relationship category | Must map through `GraphEdgeSemantics` |
| BloodHound traversable edge | Candidate analysis traversal eligibility | Must not imply fact truth without source authority |
| BloodHound non-traversable supporting edge | Candidate supporting observation | May become silver/gold only with evidence and source authority |
| Post-processed edge | Derived relationship | Must map to `DerivationRuleBundle` or analysis-only |
| Attack-path finding | Analysis result | Must map to `AnalysisRuleBundle` output |
| Risk acceptance | User workflow annotation | Must not delete facts, evidence, edges, or risk |

## 8. Collector-to-ingest workflow reconstruction

### 8.1 BloodHound CE flow

[documentation fact] BloodHound CE quickstart describes a containerized CE install, file upload, standalone collectors, JSON/ZIP output, and upload through UI or `/api/v2/file-upload/`.[^34]

```text
SharpHound CE
  -> source API / LDAP / Windows API collection
  -> JSON files by object type
  -> optional ZIP archive
  -> BloodHound CE file ingest
  -> application database and graph backend
  -> analysis / pathfinding / Explore UI
```

### 8.2 Cadastre-safe adaptation

```text
SharpHound-like collector output
  -> RawRecord payload with exact bytes, source instance, collector version, collection options, payload hash
  -> SourceCompletenessReceipt with method-specific scope and visibility state
  -> Cadastre parser
  -> CadastreSilverObservation
  -> IdentityDecision only through resolver
  -> GoldFact only through source authority and derivation rules
  -> GraphDeltaSet only through GraphProjectionProfile
```

### 8.3 Required `VersionManifest` records

Cadastre must record:

```text
collector_name
collector_release
collector_commit_or_unknown
collector_config_hash
collection_methods_bitmask
collection_methods_resolved
domain_or_scope_selection
domain_controller_selection
search_forest_flag
cache_behavior
mem_cache_flag
output_zip_settings
json_format_version
parser_version
mapping_bundle_version
source_completeness_profile_version
recorded_fixture_ids when used
```

## 9. Attack-path and pathfinding semantics

[documentation fact] BloodHound Enterprise defines an Attack Path as a chain of abusable privileges and user behaviors creating direct or indirect connections between principals, and a finding as a specific high-value remediation point. Relationship-based findings identify directional paths from lower-privileged source principals to privileged target assets.[^35]

[documentation fact] BloodHound exposure and impact metrics quantify, respectively, principals that can reach a privileged asset through attack paths and potential blast radius if a finding is abused.[^35]

[documentation fact] BloodHound posture severity maps exposure percentage bands to severity levels in Enterprise posture docs.[^36]

[documentation fact] Risk acceptance records that a risk is known and temporarily tolerated; the docs explicitly state acceptance is not a fix and that the principal and related edges remain visible in other analysis views.[^37]

Cadastre transfer:

| External concept | Cadastre rule |
| --- | --- |
| Attack path | `AnalysisRuleBundle` output only |
| Finding | `analysis_finding` or future health/risk workflow record |
| Exposure | Analysis metric unless Cadastre scoring contract defines it |
| Impact | Analysis metric unless Cadastre scoring contract defines it |
| Severity | Non-authoritative label unless Cadastre risk scoring policy maps it |
| Risk acceptance | User workflow record; must not change facts or graph deltas |
| Remediation guidance | Metadata only; no automated remediation in MVP |

## 10. Extension, schema, and validation model

### 10.1 Required validation layers

Cadastre should add an explicit BloodHound/OpenGraph-inspired validation matrix:

| Layer | Required Cadastre validation |
| --- | --- |
| Raw collector output | JSON envelope valid, exact payload hash, meta fields parsed, object type consistent |
| OpenGraph-like extension payload | Node ID, kind namespace, property type, endpoint selector, edge kind, source kind, environment ID, collected-state validation |
| Selector safety | Stable-ID/property/name/cross-source selector rules enforced |
| Traversability | Every projected edge declares one traversal class |
| Graph taxonomy | External labels translated or rejected |
| Derivation | Post-processed edge inputs persisted and replayable |
| Analysis | Query compatibility matrix validated |
| Deletion | Source-scoped deletion cannot mutate authoritative lakehouse state |

### 10.2 Extension default behavior

```text
default_extension_payload_authority = non_authoritative_observation
default_selector_resolution = unresolved_target_reference
default_traversability = not_traversable
default_finding_authority = analysis_only
default_metric_authority = analysis_only
default_source_deletion_effect = graph_projection_expire_only
```

## 11. Query, UI, and operational model

### 11.1 BloodHound UI ergonomics transferable to Cadastre

[documentation fact] BloodHound Explore supports search by name/object ID, node-type prefixes, partial matches, reverse path, edge filtering, visualization layouts, table view for Cypher, context menus, and entity panels.[^11]

Cadastre should adapt:

| UI behavior | Cadastre contract |
| --- | --- |
| Search with node-type filter | `SearchGraphNode(query, node_type_filter)` |
| Pathfinding reverse | Same query with seed/target swapped; response must indicate direction |
| Edge filter | Query must reject unknown edge types and default to profile-defined eligible edge set |
| Entity panel | Must display evidence lineage and redaction state |
| Saved query | `AnalysisRuleBundle` candidate or user-saved non-authoritative query |
| Cypher import/export | Validation-only unless query compiler proves compatibility |

### 11.2 QueryGraph comparison

| Behavior | BloodHound docs | Cadastre required behavior |
| --- | --- | --- |
| Depth bounds | Not specified in inspected docs | Default `3`, allowed `1..6` |
| Page size | Not specified for pathfinding | Default `100`, max `1000` |
| Timeout | Not specified for pathfinding | Default `30s`, max `300s` |
| Valid-time filter | Not specified | Required |
| Known-time filter | Not specified | Required |
| Assertion-state filter | Not specified | Required |
| Confidence filter | Not specified | Required |
| Authorization | ETAC environment filtering documented | Must filter by caller authorization and redaction |
| Evidence inclusion | Entity panel shows details; graph-to-gold lineage not specified | Required on request |
| Deterministic ordering | Cypher `order by` supported; path order not specified | Required path ordering |
| Derived-view lag | Not specified | Required `DerivedViewState` and lag behavior |

## 12. Source completeness, visibility, and volatility analysis

| Dataset or relationship class | Source method | Scope key | Permission dependency | Volatility | Can prove absence? | Required completeness evidence | Unsafe default state |
| --- | --- | --- | --- | --- | --- | --- | --- |
| AD users/groups/computers/properties | SharpHound LDAP/ObjectProps | Domain, forest, base DN, filters | Usually authenticated read; restricted environments need additional read grants | Low-medium | Only with complete LDAP scope and no visibility errors | LDAP scope, domain/DC, paging exhaustion, permission evidence, error map | `unknown_visibility` |
| Group membership | SharpHound Group/Default | Domain | Authenticated directory visibility | Medium | Only with complete directory visibility | Group scope complete, nested membership traversal complete | `presence_only` |
| Domain trusts | Trusts/DCOnly | Forest/domain | Directory visibility | Low | Only with complete trust query scope | Trust query success, domain list, no partial errors | `unknown_visibility` |
| ACL/rights | ACL/ObjectProps/UserRights | Domain/object class | Read permissions and method-specific access | Medium | No by default | Object coverage, ACL property visibility, source authority | `partial_unknown_gap` |
| Local admin/RDP/DCOM/PSRemote groups | LocalGroup/local computer methods | Computer set | Remote SAM/local admin/GPO-dependent | Medium-high | No by default | Computer enumeration set, successful per-computer result, failed computer list | `partial_known_gap` |
| Session/logged-on | Session/LoggedOn loop | Computer set/time window | Host access and API visibility | High | No | Time window, per-computer status, loop count, failure list | `not_authoritative_for_absence` |
| CA/DC registry | CARegistry/DCRegistry | CA/DC set | Registry/WMI/admin depending method | Medium | No by default | Registry access result by target, failed target list | `partial_known_gap` |
| CertServices | CertServices LDAP | AD CS object scope | Authenticated LDAP for most | Medium | Only with full object scope | LDAP scope and object coverage | `unknown_visibility` |
| OpenGraph source-kind upload | File/API ingest | source_kind/environment | Upload scope, extension schema | N/A | No | File manifest, source_kind, node/edge counts, validation result | `non_authoritative_metadata` |
| BloodHound file ingest count | JSON `meta.count` | File | None beyond file parse | N/A | No | Count equals array length only | `count_not_absence` |

## 13. Transfer matrix

| Finding | Source | Confirmed or inferred | Transfer class | Cadastre contract affected | Required PRD improvement | Complexity introduced | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Traversable edges determine pathfinding eligibility | BloodHound edge docs/OpenGraph | Confirmed | `adopt` | `GraphEdgeSemantics` | Add traversal-class enum and default `not_traversable` | Medium | Every graph edge has exactly one traversal class or projection fails |
| Generic graph can be searched but not pathfinding-ready | OpenGraph docs | Confirmed | `adopt` | `GraphProjectionProfile` | Separate ingest/search/pathfinding/finding/metric eligibility | Medium | Generic payload cannot emit pathfinding edges |
| Stable ID preferred over property/name matching | OpenGraph edge docs | Confirmed | `adopt` | `TargetSelectorSafetyPolicy` | Add selector safety table | Low-medium | Name selector rejected in production by default |
| Name matching deprecated | OpenGraph edge docs | Confirmed | `adopt` | `TargetSelectorSafetyPolicy` | Add `DEPRECATED_SELECTOR_REJECTED` | Low | Deprecated selector fixture fails |
| Post-processed edges regenerated and uploaded ones do not persist | OpenGraph edge docs | Confirmed | `adapt` | `DerivationRuleBundle` | Require source facts plus replayable rule | Medium-high | Backend-derived edge cannot become `GoldFact` without persisted derivation inputs |
| Source-kind deletion can delete referenced built-in nodes | OpenGraph FAQ | Confirmed | `cautionary_reference` | `GraphApplyProfile`, `GraphDeltaSet` | Add deletion-safety rules | High | Deletion fixture cannot mutate cross-source or canonical records |
| JSON `meta.methods/type/count/version` | JSON format docs | Confirmed | `adapt` | `RawRecord`, `VersionManifest` | Persist collector metadata | Low | Meta parsed but count not absence proof |
| Sessions volatile and not absence proof | SharpHound docs | Confirmed | `adopt` | `SourceCompletenessReceipt` | Add session volatility rule | Low | Missing session cannot emit negative fact |
| RDP requires membership plus user right | SharpHound/BloodHound docs | Confirmed | `adapt` | `GraphEdgeSemantics` | Require composite evidence for access edges | Medium | `CanRDP` fixture without either supporting edge emits no traversable edge |
| Attack-path severity from exposure bands | Posture docs | Confirmed, Enterprise | `study` | `AnalysisRuleBundle` | Treat as analysis severity only | Low | Severity cannot become risk score without scoring policy |
| Risk acceptance is not remediation | Risk Acceptance docs | Confirmed | `adopt` | User workflow records | Add acceptance boundary | Low | Accepted finding leaves facts/evidence/edges unchanged |
| ETAC environment filtering | Explore docs | Confirmed | `adapt` | Query authorization | Include environment authorization filter | Medium | Unauthorized environment omitted and response marks access filtering |

## 14. Do-not-transfer matrix

| External concept | Source | Why it must not transfer directly | Cadastre boundary protected | Required safer alternative |
| --- | --- | --- | --- | --- |
| Graph-authoritative mutation | BloodHound graph model | Graph backend is not Cadastre truth | Lakehouse system of record | Persist gold facts, emit graph deltas |
| Direct-to-graph ingestion | BloodHound/OpenGraph | Bypasses raw/silver/identity/gold | Raw-first pipeline | Ingest as `RawRecord` or non-authoritative graph payload |
| AD-only identity worldview | BloodHound/SharpHound | Overfits to AD principals, groups, domains | Vendor-neutral identity | Source-specific mappings into Cadastre identity decisions |
| Property/name matching as identity resolution | OpenGraph selectors | Collision-prone and deprecated for name | `IdentityDecision` | Unresolved target references only |
| Source-kind deletion as identity or asset deletion | OpenGraph FAQ | Can delete referenced built-in nodes | `SourceAsset`, `CanonicalEntity`, `GoldFact` | Source-scoped graph expiry only |
| Generic graph payloads as pathfinding-ready graph data | OpenGraph docs | Generic graphs lack traversal schema | `GraphProjectionProfile` | Structured profile with traversal rules |
| Post-processed graph edges as backend-authoritative facts | OpenGraph docs | Regenerated by backend, supporting edges hidden from paths | `GoldFact` | Replayable `DerivationRuleBundle` |
| Cypher query results as gold facts | BloodHound Cypher docs | Query is read or graph mutation surface, not evidence authority | `GoldFact`, `DerivationRuleBundle` | Analysis output unless promoted to derivation rule |
| Attack-path severity as Cadastre risk score | Posture docs | Environment-specific scoring not Cadastre policy | Risk-scoring boundary | Future explicit scoring contract |
| Risk acceptance as remediation | Risk Acceptance docs | Acceptance is not a fix | Fact/evidence integrity | Workflow annotation only |
| Observed session as ownership | HasSession docs | Session does not guarantee credential material and is volatile | `IdentityDecision`, ownership facts | `user_host_activity_fact` only |
| Group membership as effective privilege without privilege semantics | MemberOf docs | Membership may grant rights only through group privileges | `GraphEdgeSemantics` | Separate membership and privilege facts |
| Observed connection as theoretical reachability | Cadastre PRD boundary | Observed traffic does not prove complete reachability | Network graph correctness | `observed_connection` only |
| Successful collection as source completeness | SharpHound docs | Permissions and visibility vary | `SourceCompletenessProfile` | Completeness receipt plus profile |
| Collector output count as absence proof | JSON format docs | Count is array length only | Absence semantics | Expected-count evidence and scope completeness |
| BloodHound risk acceptance hiding table rows | Risk Acceptance docs | Hiding default view is not deletion/remediation | Evidence retention | Accepted-analysis state only |

## 15. PRD improvement map

| Rank | PRD area | Improvement | Source basis | Contract change required | Acceptance criterion |
| ---: | --- | --- | --- | --- | --- |
| 1 | `GraphEdgeSemantics` | Add traversal-class enum | BloodHound traversable/non-traversable docs | One traversal class per edge | Edge without class fails `GRAPH_EDGE_SEMANTICS_ERROR` |
| 2 | `GraphProjectionProfile` | Distinguish graph-ingest, search, pathfinding, findings, metrics, identity influence | OpenGraph generic vs structured docs | Projection output eligibility matrix | Generic graph cannot become pathfinding-ready |
| 3 | `GraphReadModelSchemaProfile` | Add traversability and evidence indexes | OpenGraph relationship kinds and pathfinding | Schema preflight rows for edge eligibility | Query profile rejects missing traversability index |
| 4 | `TargetSelectorSafetyPolicy` | Rank ID/property/name/cross-source selectors | OpenGraph endpoint matching docs | Selector safety mapping table | Deprecated name selector rejected |
| 5 | `SourceCompletenessProfile` | Add method-specific visibility profiles | SharpHound permissions docs | Dataset/scope/permission rows | Permission-limited collection cannot assert absence |
| 6 | `SourceCompletenessReceipt` | Add session volatility and output-count non-proof states | SharpHound session/retention docs | Receipt evidence classes | Missing session cannot emit absence |
| 7 | `RecordedSourceFixture` | Add SharpHound JSON and OpenGraph fixtures | JSON/OpenGraph docs | Fixture types for meta, selectors, deletion hazards | Fixture replay deterministic |
| 8 | `VersionManifest` | Record collector/schema/query/projection versions | SharpHound repo/options, OpenGraph schemas | Add collector and graph-schema refs | Replay rejects version mismatch |
| 9 | `AnalysisRuleBundle` | Treat Cypher and attack-path findings as read-only | BloodHound Cypher/findings docs | Rule output type and graph compatibility | Analysis cannot mutate graph/facts |
| 10 | `DerivationRuleBundle` | Support post-processed edge derivation only with persisted inputs | OpenGraph post-processing docs | Source-fact and rule replay requirements | Derived edge checksum stable |
| 11 | `GraphTaxonomyTranslationPolicy` | External labels/kinds/source_kind cannot define Cadastre semantics | OpenGraph metadata docs | Translation rows and rejection rules | Unknown external kind rejected or stored non-authoritatively |
| 12 | `IdentityDecision` | Stable ID/property matching remains non-identity by default | OpenGraph selector docs | Selector-to-decision evidence boundary | Property match cannot auto-merge |
| 13 | `UnresolvedTargetReference` | Use for OpenGraph endpoint hints and cross-source refs | OpenGraph endpoints | Add selector fields and collision state | Cross-source ref waits for identity decision |

## 16. NLSpec-ready contract recommendations

### 16.1 Traversable edge semantics

```text
GraphProjectionProfile must assign exactly one traversal_class to every emitted edge.
```

| `traversal_class` | Query behavior | Default | Evidence requirement | Non-implication rule |
| --- | --- | ---: | --- | --- |
| `traversable_for_identity_path` | Eligible for identity-path search | No | Identity or privilege fact with active `GraphEdgeSemantics` row | Does not imply asset identity merge |
| `traversable_for_exposure_path` | Eligible for exposure-path search | No | Exposure or privilege fact plus source authority | Does not imply complete reachability |
| `traversable_for_membership_path` | Eligible for membership expansion | No | Membership fact and temporal validity | Does not imply effective privilege |
| `not_traversable` | Searchable but excluded from pathfinding | Yes | Any allowed source fact | Does not imply path eligibility |
| `analysis_only` | Visible only to compatible analysis rules | No | Analysis rule output or non-authoritative metadata | Does not imply gold fact truth |

Errors:

| Error | Required behavior |
| --- | --- |
| `GRAPH_EDGE_TRAVERSAL_CLASS_MISSING` | Projection fails before delta emission |
| `GRAPH_EDGE_TRAVERSAL_EVIDENCE_ERROR` | Edge emits `noop` or fails according to profile |
| `UNSUPPORTED_TRAVERSAL_CLASS` | Profile validation fails |

Acceptance criteria:

```text
AC-TES-001: Every active graph edge type has one traversal_class.
AC-TES-002: Pathfinding excludes not_traversable and analysis_only by default.
AC-TES-003: Membership traversal cannot return privilege paths unless privilege edges are explicitly eligible.
AC-TES-004: Exposure traversal cannot infer theoretical reachability from observed traffic.
```

### 16.2 Derived edge handling

```text
Backend-generated or post-processed edges must not become authoritative Cadastre facts by default.
```

| External edge source | Cadastre mapping | Default |
| --- | --- | --- |
| BloodHound post-processed edge with persisted supporting edges | `DerivationRuleBundle` candidate | `study` until rule defined |
| BloodHound post-processed edge without supporting facts | `AnalysisRuleBundle` output or `explicit no-op` | `reject` as fact |
| OpenGraph traversable edge directly uploaded | Candidate silver relationship observation | `not_traversable` until profile says otherwise |
| Cypher-created edge | Non-authoritative graph mutation | `reject` in production |
| Query result path | Analysis output | `analysis_only` |

Errors:

```text
DERIVED_EDGE_INPUT_MISSING
DERIVED_EDGE_RULE_UNDECLARED
BACKEND_DERIVED_FACT_REJECTED
```

Acceptance criteria:

```text
AC-DEH-001: A derived edge cannot emit `GoldFact` unless every input fact and derivation rule version is recorded.
AC-DEH-002: Same input facts plus same derivation rule produce byte-identical derived fact IDs.
AC-DEH-003: Backend-generated edge without persisted supporting facts emits no gold fact.
```

### 16.3 Source-kind and deletion safety

```text
External source labels, source kinds, and source-owned graph records may expire only source-scoped graph projection output.
They must not delete or mutate CanonicalEntity, IdentityDecision, GoldFact, SourceAsset from another source, Identifier from another source, RawRecord, CadastreSilverObservation, or authoritative lakehouse records.
```

Default behavior:

```text
source_kind_delete_default = graph_projection_expire_only
cross_source_reference_delete_default = reject
built_in_node_reference_delete_default = reject
```

Errors:

```text
SOURCE_KIND_DELETE_SCOPE_ERROR
CROSS_SOURCE_DELETE_REJECTED
AUTHORITATIVE_RECORD_DELETE_REJECTED
```

Acceptance criteria:

```text
AC-SDS-001: Deleting one source projection cannot remove a canonical entity.
AC-SDS-002: A source-kind delete that references another source asset fails before graph apply.
AC-SDS-003: The graph apply result records expired graph IDs, not deleted lakehouse records.
```

### 16.4 Selector safety

| External selector mechanism | Required external fields | Match behavior | Collision risk | Cadastre maximum allowed state | Required Cadastre evidence | Default production behavior |
| --- | --- | --- | --- | --- | --- | --- |
| Stable ID matching | `id`, source/system ID, environment scope | Exact match | Low | `UnresolvedTargetReference` or source-scoped `SourceAsset` | Source instance, stable ID, payload hash | Allow unresolved reference |
| Property matching | property name/value, node kind | Equality over declared property | Medium-high | `UnresolvedTargetReference` only | Collision check, selector profile | Allow only unresolved |
| Name matching | `name` | Deprecated property match | High | None | Waiver only | Reject |
| Deprecated name matching | `name` strategy | Deprecated | High | None | Not applicable | Reject |
| Environment-scoped matching | environment ID plus ID/property | Scope-limited | Medium | Unresolved or source-scoped observation | Environment authority evidence | Allow unresolved |
| Source-kind-scoped matching | `source_kind` plus selector | Source-owned match | Medium | Source-scoped projection only | Source ownership evidence | Allow graph projection only |
| Cross-source referenced node matching | ID/property of another source | Cross-source hint | High | `UnresolvedTargetReference` | Later `IdentityDecision` | Wait for identity decision |
| Generic unresolved target matching | Any weak selector | No merge | High | `UnresolvedTargetReference` | Raw evidence and selector class | Store unresolved or reject |

### 16.5 Collector completeness

```text
successful_collection != completeness_proof
output_count != absence_proof
session_non_observation != session_absence
permission_limited_collection != complete_scope
cache_hit != source_evidence unless recorded
```

Required receipt fields:

```text
collector_name
collector_version
collection_method
scope_key
scope_members_requested
scope_members_completed
scope_members_failed
permission_basis
visibility_state
records_returned_count
expected_count_basis
volatile_relationship
cache_used
cache_recorded_as_evidence
absence_semantics_valid
watermark_advance_allowed
```

Errors:

```text
COLLECTOR_COMPLETENESS_UNPROVEN
PERMISSION_VISIBILITY_UNKNOWN
VOLATILE_RELATIONSHIP_ABSENCE_REJECTED
OUTPUT_COUNT_ABSENCE_REJECTED
CACHE_EVIDENCE_MISSING
```

### 16.6 Pathfinding query contract

```text
QueryGraphPath must use Cadastre graph semantics, not BloodHound pathfinding semantics.
```

| Dimension | Contract |
| --- | --- |
| Maximum depth | Default `3`; allowed `1..6` |
| Page size | Default `100`; max `1000` |
| Timeout | Default `30s`; max `300s` |
| Edge eligibility | Only edges whose `traversal_class` is allowed by query profile |
| Path ordering | Length, minimum edge confidence descending, maximum last_seen descending, lexical node sequence |
| Temporal filters | Valid-time and known-time filters required; default current/current |
| Confidence filters | Must filter on edge and fact confidence |
| Assertion-state filters | Default excludes retracted; caller may request allowed states |
| Authorization filtering | Apply caller access before path expansion when possible and before response always |
| Evidence inclusion | Off by default; when on, include graph-to-gold lineage refs |
| Derived-view lag | Return `DERIVED_VIEW_LAG_ERROR` unless stale reads permitted |
| Unsupported query | Return `UNSUPPORTED_GRAPH_QUERY` without partial execution |

### 16.7 Attack-path findings and risk acceptance

```text
Attack-path findings are read-only analysis outputs by default.
Risk acceptance is a user workflow record.
Risk acceptance must not mean remediation, fact retraction, graph edge deletion, evidence deletion, or source cleanup.
```

| Concept | Cadastre record |
| --- | --- |
| Attack-path finding | `AnalysisFinding` or `AnalysisRuleBundleResult` |
| Exposure metric | Analysis metric |
| Impact metric | Analysis metric |
| Severity | Analysis label unless scoring policy exists |
| Remediation text | Guidance metadata |
| Risk acceptance | `RiskAcceptanceRecord` |
| Remediation | Out of MVP scope unless future contract |

Acceptance criteria:

```text
AC-APR-001: Accepted finding leaves all source facts, graph edges, and evidence refs unchanged.
AC-APR-002: Accepted finding appears only when accepted findings are requested or workflow requires it.
AC-APR-003: Severity cannot populate risk score without active risk-scoring policy.
```

## 17. Gaps, risks, stale assumptions, and unresolved questions

| Type | Item | Required verification |
| --- | --- | --- |
| Version drift | BloodHound repo README still names Neo4j graph DB, while OpenGraph docs emphasize PostgreSQL graph DB for full support | Inspect exact v9.1.0 backend graph implementation and migration docs |
| Backend ambiguity | Neo4j and PostgreSQL graph behavior may differ | Runtime test generic/structured OpenGraph ingest and pathfinding |
| AD/Entra overfitting | Built-in edge taxonomy is AD/Entra-centric | Cadastre taxonomy translation matrix |
| Generic OpenGraph payloads | Searchable but not pathfinding-ready | Fixture that generic payload cannot produce traversable path |
| Source-kind deletion side effects | Source-kind can affect referenced built-ins | Deletion-safety fixture |
| Endpoint collisions | Property/name matching collision risk | Selector collision tests |
| Session volatility | Sessions are time-window dependent | Volatile relationship completeness profile |
| Permission-limited collection | Many SharpHound methods depend on permissions | Per-method visibility evidence |
| Collector count | `meta.count` is array length only | Output-count non-proof test |
| Query results | Cypher result is not fact truth | Analysis/derivation separation |
| Risk severity | BloodHound severity is not Cadastre risk | Risk scoring policy required |
| Risk acceptance | Acceptance is not remediation | Workflow-only acceptance test |
| Community examples | OpenGraph community extensions not inspected | Treat as unreviewed examples |
| Unsupported syntax | CySQL support has caveats and differences | Do not reuse as Cadastre query contract |
| Missing tests/build | No local clone/build/run | Implementation follow-up must verify source locally |
| Offensive docs | Edge pages include abuse/OPSEC details | Cadastre report uses only schema/semantic facts |

## 18. Definition of done and acceptance criteria

| ID | Acceptance criterion | Status in this report |
| --- | --- | --- |
| AC-001 | BloodHound analyzed independently before Cadastre comparison | Pass |
| AC-002 | SharpHound analyzed independently before Cadastre comparison | Pass |
| AC-003 | Official BloodHound/OpenGraph docs analyzed separately | Pass |
| AC-004 | Repository/source claims cited and classified | Pass, within inspected scope |
| AC-005 | Inspection dates and versions stated | Pass |
| AC-006 | BloodHound architecture reconstruction included | Pass |
| AC-007 | SharpHound collector reconstruction included | Pass |
| AC-008 | Generic-versus-structured OpenGraph analysis included | Pass |
| AC-009 | Traversable-versus-non-traversable edge analysis included | Pass |
| AC-010 | Source-kind, environment, collected-state, deletion analysis included | Pass |
| AC-011 | Endpoint selector safety analysis included | Pass |
| AC-012 | Graph edge semantics mapping table included | Pass |
| AC-013 | Collector completeness table included | Pass |
| AC-014 | Do-not-transfer matrix included | Pass |
| AC-015 | Highest-value PRD improvement areas mapped | Pass |
| AC-016 | Recommendations include observable behavior, interfaces, defaults, bounds, errors, determinism, mappings, and acceptance criteria | Pass |
| AC-017 | No direct-to-graph, graph-authoritative, property-match identity, AD-only, or backend-derived fact recommendation | Pass |
| AC-018 | No exploit walkthroughs, evasion guidance, unauthorized collection steps, or offensive runbooks included | Pass |

## 19. Sources

[^1]: `PRD-Cadastre.md`, Sections 1-2 and normative contract table. Local project file inspected 2026-05-16.

[^2]: `PRD-Cadastre.md`, Section 8.4 Graph Query. Local project file inspected 2026-05-16.

[^3]: `PRD-Cadastre.md`, Source completeness authorization rules. Local project file inspected 2026-05-16.

[^4]: `PRD-Cadastre.md`, GraphEdgeSemantics fields and MVP edge semantics. Local project file inspected 2026-05-16.

[^5]: SpecterOps/BloodHound repository, tag `v9.1.0`, commit `6b44cc3`. <https://github.com/SpecterOps/BloodHound/tree/v9.1.0>

[^6]: SpecterOps/SharpHound repository and releases, tag `v2.12.0`, commit `7faa2bb`. <https://github.com/SpecterOps/SharpHound/tree/v2.12.0>

[^7]: BloodHound UI package manifest. <https://raw.githubusercontent.com/SpecterOps/BloodHound/v9.1.0/cmd/ui/package.json>

[^8]: BloodHound Go module manifest. <https://raw.githubusercontent.com/SpecterOps/BloodHound/v9.1.0/go.mod>

[^9]: BloodHound development README. <https://raw.githubusercontent.com/SpecterOps/BloodHound/v9.1.0/DEVREADME.md>

[^10]: BloodHound testing compose file. <https://raw.githubusercontent.com/SpecterOps/BloodHound/v9.1.0/docker-compose.testing.yml>

[^11]: BloodHound Explore search and pathfinding docs. <https://bloodhound.specterops.io/analyze-data/explore/search>

[^12]: BloodHound OpenGraph graph data docs. <https://bloodhound.specterops.io/opengraph/developer/graph-data>

[^13]: BloodHound OpenGraph nodes docs. <https://bloodhound.specterops.io/opengraph/developer/nodes>

[^14]: BloodHound OpenGraph edges docs. <https://bloodhound.specterops.io/opengraph/developer/edges>

[^15]: BloodHound traversable edges docs. <https://bloodhound.specterops.io/resources/edges/traversable-edges>

[^16]: BloodHound OpenGraph graph definition docs. <https://bloodhound.specterops.io/opengraph/developer/graph-definition>

[^17]: BloodHound edge docs, including `MemberOf` and `HasSession`. <https://bloodhound.specterops.io/resources/edges/member-of>

[^18]: BloodHound `CanRDP` edge docs. <https://bloodhound.specterops.io/resources/edges/can-rdp>

[^19]: BloodHound `AZHasRole` edge docs. <https://bloodhound.specterops.io/resources/edges/az-has-role>

[^20]: BloodHound `SyncedToADUser` edge docs. <https://bloodhound.specterops.io/resources/edges/synced-to-ad-user>

[^21]: BloodHound Cypher search docs. <https://bloodhound.specterops.io/analyze-data/explore/cypher-search>

[^22]: BloodHound Cypher-supported syntax docs. <https://bloodhound.specterops.io/analyze-data/explore/cypher-supported>

[^23]: BloodHound OpenGraph extension management docs. <https://bloodhound.specterops.io/opengraph/extensions/manage>

[^24]: BloodHound OpenGraph FAQ. <https://bloodhound.specterops.io/opengraph/faq>

[^25]: SharpHound project file. <https://raw.githubusercontent.com/SpecterOps/SharpHound/v2.12.0/Sharphound.csproj>

[^26]: SharpHound options source. <https://raw.githubusercontent.com/SpecterOps/SharpHound/v2.12.0/src/Options.cs>

[^27]: SharpHound entrypoint source. <https://raw.githubusercontent.com/SpecterOps/SharpHound/v2.12.0/src/Sharphound.cs>

[^28]: BloodHound SharpHound collection docs. <https://bloodhound.specterops.io/collect-data/ce-collection/sharphound>

[^29]: BloodHound SharpHound flags docs. <https://bloodhound.specterops.io/collect-data/ce-collection/sharphound-flags>

[^30]: BloodHound JSON format docs. <https://bloodhound.specterops.io/integrations/bloodhound-api/json-formats>

[^31]: BloodHound SharpHound data permissions docs. <https://bloodhound.specterops.io/collect-data/sharphound-data-permissions>

[^32]: BloodHound Enterprise data retention docs. <https://bloodhound.specterops.io/collect-data/enterprise-collection/data-retention>

[^33]: BloodHound OpenGraph dog-park schema example. <https://raw.githubusercontent.com/SpecterOps/BloodHound/v9.1.0/schemas/open_graph/dog-park-schema.json>

[^34]: BloodHound Community Edition quickstart docs. <https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart>

[^35]: BloodHound attack-path findings docs. <https://bloodhound.specterops.io/analyze-data/findings/attack-paths>

[^36]: BloodHound posture docs. <https://bloodhound.specterops.io/analyze-data/findings/posture>

[^37]: BloodHound risk acceptance docs. <https://bloodhound.specterops.io/analyze-data/findings/risk-acceptance>
