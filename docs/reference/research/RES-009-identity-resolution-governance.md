---
doc_id: RES-009
project: Cadastre
title: Cadastre Identity Resolution and Resolver Governance Research Report
status: research-report
inspection_date: 2026-05-16
---

## 1. Executive summary

This report evaluates how Cadastre should specify canonical asset identity resolution so two independent implementations, given identical raw and silver inputs, source scopes, resolver profile, lifecycle evidence, completeness evidence, and version manifest, emit the same `IdentityDecision`, review-routing, split-correction, and downstream graph-correction outputs.

The governing Cadastre PRD already establishes the correct product boundary: the lakehouse is the system of record, graph and Splunk CIM outputs are replaceable projections, and source observations flow through raw, silver, identity, gold, and graph projection stages rather than directly into graph truth.[^1] It also already requires separate `CanonicalEntity`, `SourceAsset`, `Identifier`, and `IdentityDecision` concepts, forbids IP-only, hostname-only, DNS-only, PTR-only, provider-key-only, and mapped-target-only canonical host merges, and requires negative evidence to block auto-merge.[^3]

The PRD’s remaining identity risk is not conceptual. It is operational determinism. Resolver authority is still implied across `IdentityDecision`, `TargetSelectorSafetyPolicy`, `SourceAuthorityProfile`, `VersionManifest`, `ValidationMatrix`, and Section 14 rather than fully centralized in a production `ResolverProfile` with closed evidence classes, hard blockers, decision matrix rows, lifecycle boundary rules, review routing, activation scenarios, shadow/canary gates, and explanation fields. NLSpec quality requires behaviorally complete contracts, precise interfaces, explicit defaults and bounds, exhaustive mapping tables, and binary acceptance criteria.[^4]

| Rank | Finding | PRD impact | Confidence | Source basis | Recommended transfer stance |
| ---: | --- | --- | --- | --- | --- |
| 1 | Cadastre needs a first-class `ResolverProfile` as the sole production authority for identity decisions. | Add `ResolverProfile` and make identity resolution fail when no active profile covers entity type plus source scopes. | High | PRD distributed resolver contracts and Section 14 identity rules.[^2][^3] | Adopt. |
| 2 | Identifier values must be normalized into typed `IdentityEvidenceItem` rows with durability, scope, lifecycle, reuse, source quality, and role semantics. | Add `IdentityEvidenceItem` and `IdentifierEvidenceClass` registry. | High | OpenTelemetry identifying-attribute rules, Kubernetes UID/name behavior, cloud and endpoint identifiers.[^7][^8][^9][^10][^11][^12] | Adopt. |
| 3 | Auto-merge must be rule-gated before score-gated. | Add hard-blocker evaluation before confidence computation and decision matrix selection. | High | PRD negative-evidence rule; ServiceNow identification priority; Tenable higher-priority conflict behavior.[^3][^5][^14] | Adopt. |
| 4 | IP, DNS, PTR, hostname, flow, scanner-name, and management-name evidence must default to selector or candidate-hint roles, not auto-merge authority. | Add default `no_decision` rows for weak temporal evidence. | High | PRD non-scope, RFC DNS/DHCP temporal behavior, Zeek UID correlation-only behavior, scanner docs.[^3][^16][^14][^15] | Adopt. |
| 5 | Asset generation boundaries must be first-class resolver input. | Add `AssetGenerationBoundary` with reimage, clone, VDI, agent reinstall, delete/recreate, rekey, and scanner-correlation-change cases. | High | Kubernetes object lifecycle, Rapid7 VDI and multi-NIC behavior, cloud delete/recreate, endpoint enrollment behavior.[^8][^15][^11][^12][^13] | Adopt. |
| 6 | Review workflows must be deterministic state machines that emit terminal `IdentityDecision` records and never mutate identity invisibly. | Add `IdentityReviewCase` with closed states, permitted transitions, reviewer authority, expiration, override evidence, and replay rules. | High | PRD identity conflict visibility; ServiceNow simulation as non-mutating validation pattern; Dedupe active labeling pattern adapted under Cadastre authority.[^3][^5][^17] | Adopt with stricter Cadastre governance. |
| 7 | Source-native merge histories must be lineage and evidence, not Cadastre identity authority. | Add `source_native_merge_history_policy` and evidence class default `lineage_only`. | High | Microsoft Defender `MergedDeviceIds` fields and scanner correlation behavior.[^12][^14][^15] | Adopt boundary, reject authority. |
| 8 | General ER frameworks are useful for candidate generation, blocking, labeling, explanation, and calibration but unsafe as direct production merge authorities. | Add `CandidateGenerationProfile`, `ResolverTrainingArtifact`, `ResolverShadowRun`, and deterministic wrapper requirements. | High | Splink, Dedupe, py_entitymatching, and Zingg docs.[^17] | Adapt, do not import autonomous merge. |
| 9 | Graph-first systems prove why graph keys and target selectors cannot create canonical identity. | Strengthen `UnresolvedTargetReference`, `TargetSelectorSafetyPolicy`, and graph-correction handoff. | High | JupiterOne, Cartography, BloodHound/OpenGraph research.[^18][^19][^20] | Adapt graph patterns, reject graph authority. |
| 10 | Resolver activation must be scenario-gated, not merely profile-versioned. | Add `ResolverActivationReport` with required pass/fail scenarios and output checksums. | High | PRD `ValidationMatrix`, ServiceNow simulation, ER validation practices.[^6][^5][^17] | Adopt. |

## 2. Source inventory and inspection limits

Evidence classes in this report:

| Evidence class | Meaning |
| --- | --- |
| `confirmed source fact` | Directly supported by inspected official documentation, source repository research, standards text, or uploaded project files. |
| `source-grounded inference` | Inferred from one source family’s confirmed facts. |
| `Cadastre applicability inference` | Derived by comparing confirmed facts to the governing PRD. |
| `speculative transfer` | Plausible but requires product, platform, or domain authority before PRD adoption. |
| `open question` | Requires deeper source inspection, production data, or governance decision. |

| Source family | Source name or repository identifier | Evidence type | Inspected version or freshness | Inspection date | Scope inspected | Scope not inspected | Reliability | Freshness risk |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Cadastre baseline | `/mnt/data/PRD-Cadastre.md` | Project PRD | Uploaded draft, 2026-05-16 | 2026-05-16 | Product summary, normative contracts, source flow, identity data model, Section 14, version manifest, validation matrix. | No hidden project docs or repo code. | High for governing baseline. | Medium because draft can change. |
| NLSpec standard | `/mnt/data/nlspec-spec.md` | Project standard | Version 0.2.2 | 2026-05-16 | Behavioral completeness, interfaces, defaults, mapping tables, acceptance criteria, recreatability test. | No implied project behavior beyond spec discipline. | High. | Low. |
| CMDB identity and reconciliation | ServiceNow CMDB IRE docs | Official docs | Australia/Xanadu docs, updated 2024-2026 where shown | 2026-05-16 | IRE overview, identification rules, dependent CIs, reconciliation rules, data-source insert/update rules, simulation, partial payloads. | No ServiceNow instance, no private rule tables, no execution. | High for documented concepts. | Medium. |
| Telemetry identity | OpenTelemetry entity data model and resource semantic conventions | Official docs/spec | Current docs and spec 1.56.0 surfaced by search | 2026-05-16 | Identifying attributes, descriptive attributes, minimally sufficient identity, repeatability across observers, lifespan specificity. | No SDK implementation tests. | High. | Medium. |
| Kubernetes identity | Kubernetes object names, IDs, labels, annotations, Pods | Official docs | Current docs, last-updated dates visible in docs | 2026-05-16 | Names, UID, namespace/resource scope, deleted-object name reuse, labels and annotations, pod immutability. | No API-server runtime tests. | High. | Low-medium. |
| Cloud provider identifiers | AWS EC2 docs, Azure ARM/IMDS docs, Google Cloud Asset Inventory docs | Official docs | Current docs, Google CAI last updated 2026-05-13 | 2026-05-16 | AWS instance/IP/ENI/EIP/tags, Azure ARM scopes/resource IDs/vmId, Google full resource names and CAI history. | No provider API execution; GCP instance numeric ID reuse not confirmed from official docs. | High for documented behavior. | Medium. |
| Endpoint, MDM, EDR, agents | Microsoft Intune, Defender XDR, Microsoft Graph devices, osquery, Fleet | Official docs | Current docs | 2026-05-16 | Managed device IDs, Azure/Entra device IDs, Defender DeviceInfo IDs and merge fields, osquery node_key enrollment, Fleet enroll/reenroll behavior. | No tenant data, no agent internals beyond docs. | High for schema fields. | Medium. |
| Vulnerability scanners | Tenable Security Center, Rapid7 InsightVM docs | Official docs | Tenable SC 6.8.x docs, Rapid7 current docs | 2026-05-16 | Tenable ordered asset tracking and conflict behavior; Rapid7 agent UUID, site linking, VDI, multi-NIC behavior. | No scanner API data, no Tenable.io/Tenable VM deep API inspection. | High for documented correlation behavior. | Medium. |
| DNS, DHCP, IPAM, flows | RFC 2131, RFC 1035, Zeek docs, NetBox docs | Standards and official docs | RFCs stable; Zeek current docs; NetBox docs 2025 | 2026-05-16 | DHCP lease intervals, DNS TTL, Zeek `uid`, NetBox intended-state boundary. | No DHCP server logs or authoritative DNS server traces. | High. | Low for RFCs, medium for tools. |
| Entity resolution frameworks | Splink, Dedupe, py_entitymatching, Zingg | Official docs/repositories | Current web docs, mixed release freshness | 2026-05-16 | Probabilistic linkage, blocking, active labeling, thresholds, model settings, explainability and blocking diagnostics. | No local model training or benchmark. | Medium-high. | Medium. |
| Adjacent graph systems | JupiterOne/Starbase uploaded research | Prior research based on primary docs/source | RES-002, observed 2026-05-15 | 2026-05-16 | Graph-first ingestion, provider-scoped keys, mapped relationships, testing, graph output. | Primary sources were not re-inspected in this report beyond prior research. | Medium-high. | Medium. |
| Adjacent graph systems | Cartography uploaded research | Prior research based on source/doc inspection | RES-001, observed 2026-05-15 | 2026-05-16 | Neo4j graph ingestion, model/query builders, cleanup, source modules. | Primary repo not re-cloned or run here. | Medium-high. | Medium. |
| Adjacent graph systems | BloodHound, SharpHound, OpenGraph uploaded research | Prior research based on primary docs/source | RES-008, observed 2026-05-16 | 2026-05-16 | Stable IDs, OpenGraph endpoint matching, traversability, deletion hazards, collector output. | Primary sources not re-inspected beyond prior research. | Medium-high. | Medium. |
| Semantic overlays | Taxi uploaded research | Prior research based on source/docs | RES-003, observed 2026-05-15 | 2026-05-16 | Semantic overlays, aliases, schema converters, compiler/linter tooling. | No local build; external artifacts unverified. | Medium-high for source structure. | Medium. |

## 3. Cadastre baseline assessment

### What the PRD already gets right

Cadastre already defines the product as a temporal lakehouse-backed graph asset-intelligence platform and makes the graph a replaceable projection rather than the system of record.[^1] It already defines `SourceAuthorityProfile`, `TargetSelectorSafetyPolicy`, `GraphEdgeSemantics`, `ValidationMatrix`, and `RecordedSourceFixture` as normative contracts, which is the correct class of governance objects for resolver and projection determinism.[^2]

Cadastre already separates `CanonicalEntity`, `SourceAsset`, `Identifier`, and `IdentityDecision`, and `IdentityDecision` already carries decision type, canonical entity, source asset IDs, identifier IDs, confidence, positive evidence, negative evidence, valid time, known time, and resolver versions.[^3] The PRD also already forbids directly identifying a canonical host by mutable hostname, IP, DNS name, or PTR record, and it gives default evidence strengths that reject IP-only, hostname-only, PTR-only, semantic-overlay-only, and query-ID-only auto-merge.[^3]

Cadastre already requires split correction to close the prior decision’s knowledge interval, create a new split decision, correct affected gold facts, emit graph retractions or expirations, and preserve the prior merge plus later split evidence chain.[^3] Gold facts already require valid-time and known-time intervals, evidence refs, source authority, assertion states, and correction by creating new facts rather than overwriting existing facts.[^4]

Cadastre already requires an immutable `VersionManifest` before production output and requires it to include every output-affecting resolver profile, target selector safety policy, graph edge semantics policy, validation artifact, source configuration, completeness receipt, coverage assertion, and stage state that can affect output.[^5]

### What remains underspecified

| Area | Existing PRD coverage | Residual implementation-divergence risk | Required PRD refinement |
| --- | --- | --- | --- |
| Resolver production authority | `resolver profile` is referenced in version manifests and validation matrix rows, but the contract is not fully defined in the PRD excerpt. | Two implementations can disagree about which rule set governs evidence classes, blockers, confidence bands, and review routing. | Add `ResolverProfile` as first-class sole identity authority. |
| Evidence classes | Identifier types are defined, but typed evidence semantics are not complete. | Same identifier value can be treated as strong, weak, temporal, selector, or negative evidence inconsistently. | Add `IdentifierEvidenceClass` plus `IdentityEvidenceItem`. |
| Source scope | PRD requires source instance/scope hashes, but identity-specific scope keys are not exhaustively tied to tenant/account/project/cluster/site. | Provider-scoped IDs may be over-promoted to global IDs. | Add identity `IdentifierScope` rows and scope validation algorithm. |
| Candidate generation | Section 14 defines evidence strength but not blocking keys, candidate ordering, pair ordering, or learned candidate handling. | Resolver outputs can differ before scoring. | Add `CandidateGenerationProfile`. |
| Confidence computation | Section 14 has strengths, but no deterministic confidence algorithm or band selection rows. | Implementations may compute scores differently. | Prefer deterministic rule bands for MVP; wrap weighted/model scores as governed hints. |
| Lifecycle boundaries | Section 14 names reimage as negative evidence but does not define clone, VDI, agent reinstall, source rekey, delete/recreate, directory reenrollment, Kubernetes recreate, or scanner correlation change. | Resolver may merge across generation boundaries inconsistently. | Add `AssetGenerationBoundary` table and materialization rules. |
| Review routing | User-facing states exist, but reviewer case state, authority, terminal outputs, expiration, and replay are not defined. | Manual review can mutate identity hidden from `IdentityDecision`. | Add `IdentityReviewCase` state machine. |
| Activation gates | Validation matrix exists, but default resolver activation scenarios are not enumerated. | A resolver profile can become active without proving rejection and split cases. | Add `ResolverActivationReport`. |
| Source-native merge history | Defender/Tenable/Rapid7-like merge histories are not explicitly typed. | Source-native merges may be treated as Cadastre authority. | Add merge-history policy with default `lineage_only`. |
| Graph correction after identity correction | Split behavior names graph retractions/expirations, but exact handoff contract is not tied to resolver output. | Projection can fail to retract stale edges after identity split. | Add graph-correction handoff fields in `ResolverExplanation` and split policy. |

## 4. Independent source analyses

### 4.1 CMDB identity and reconciliation systems

#### Source inventory and inspection limits

| Field | Content |
| --- | --- |
| Source name | ServiceNow CMDB Identification and Reconciliation Engine, identification rules, dependent CI rules, reconciliation rules, data-source rules, identification simulation. |
| Evidence type | Official documentation. |
| Inspected version or freshness | Public docs updated in 2024-2026 where pages expose dates. |
| Scope inspected | Identification/reconciliation concepts, independent/dependent CIs, priority/order, reconciliation, data-source insert blocking, simulation, partial payloads. |
| Scope not inspected | No ServiceNow instance, no actual rule export, no API execution. |
| Reliability | High for conceptual and documented rule behavior. |
| Freshness risk | Medium. ServiceNow docs and release families change. |

#### Source overview

ServiceNow IRE solves CMDB CI identification and attribute reconciliation, not Cadastre canonical identity. IRE centralizes identification and reconciliation for different data sources; identification rules decide which CI a payload refers to, and reconciliation rules decide which data sources may update which attributes.[^5]

#### Architecture and operational model

ServiceNow identification rules are scoped by class and use identifier entries composed of unique or required attributes. Child classes inherit parent identification rules when no child-specific rule exists. Dependent CIs require parent or relationship context for identification.[^5]

Reconciliation rules are separate from identification. They define which discovery sources are authorized to update attributes and use source priority or dynamic rules. Data-source rules can allow a data source to update existing CIs while preventing it from inserting new CIs for a class.[^5]

ServiceNow simulation is valuable because it constructs valid payloads, including dependent CIs, and does not commit updates.[^5]

#### Data, schema, and identity model

| Concept | Source behavior | Cadastre transfer |
| --- | --- | --- |
| Identification rule | Class-scoped rule that identifies a CI using attributes. | `ResolverProfile.decision_matrix_rows` with entity type and source scope coverage. |
| Dependent CI rule | Identification depends on parent CI or relationship context. | `ResolverProfile.dependent_identity_rows` for child resources such as network interfaces, cloud disks, Kubernetes pods, scanner findings. |
| Reconciliation rule | Attribute update authority, distinct from identity. | `SourceAuthorityProfile.attribute_authority_rows`; do not mix with identity matching. |
| Data-source insert rule | Update allowed but insert blocked. | `source_permission_rows` that allow evidence contribution without canonical creation. |
| Simulation | Rule testing without committing updates. | `ResolverShadowRun` and `ResolverActivationReport`. |

#### Identifier behavior

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive identity evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CI class identifier entry | CMDB class and source payload | CI generation | Depends on attributes | Depends on attributes | Depends on attributes | Only under active profile row | Conflict with higher-priority rule | Reference pattern only; do not copy ServiceNow rules. |
| Dependent CI parent reference | Parent CI or relationship context | Dependent CI lifecycle | Strong when parent identified | Parent relationship can change | Medium | Supporting context | Missing required parent blocks identification | Adapt as dependent identity row. |
| Data-source name | CMDB source | Rule lifecycle | Not identity | Mutable config | Medium | None by itself | Missing permission blocks insert/merge | Source permission only. |

#### Matching, correlation, and reconciliation behavior

The transferable pattern is ordered rule evaluation with explicit class scope and dependency context. The rejected pattern is treating CMDB CI identity rules as Cadastre truth. Cadastre must not import a CMDB rule that merges host identity by name or weak attributes unless the Cadastre `ResolverProfile` separately permits those evidence classes.

#### Validation, review, and governance behavior

Cadastre should transfer simulation as a non-mutating activation gate. `ResolverActivationReport` must run required scenarios and must emit expected `IdentityDecision` outputs without mutating canonical identity, gold facts, graph deltas, watermarks, or production health.

#### Confirmed findings versus inferences

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Identification rules are class-scoped and use required/unique attributes; dependent CIs require parent or relationship context; reconciliation controls attribute update authority; data-source rules can block inserts but allow updates.[^5] | Rule order and scope separation are essential to preventing duplicate CIs and overbroad authority. | Cadastre should separate identity matching, attribute authority, completeness, and source insert permission. | Simulated resolver payload generation may inform Cadastre validation scenario generation. | Exact ServiceNow priority algorithms and CMDB 360 dynamic behavior were not executed. |

#### Concepts that must not transfer

| Rejected concept | Reason |
| --- | --- |
| CMDB CI identity as Cadastre canonical identity | CMDB rules are source configuration, not Cadastre resolver authority. |
| Attribute reconciliation as identity matching | Attribute authority does not prove entity equality. |
| Source insert permission as evidence strength | Insert/update permissions govern source behavior, not identifier durability. |
| Partial payload merge as Cadastre merge authority | Partial payloads require Cadastre evidence refs and profile rules. |

### 4.2 Telemetry and resource identity standards

#### Source inventory and inspection limits (telemetry)

| Field | Content |
| --- | --- |
| Source name | OpenTelemetry Entity Data Model, OpenTelemetry resource/entity semantic conventions, Kubernetes object names/IDs/labels/annotations/pods. |
| Evidence type | Official documentation/specification. |
| Scope inspected | Identifying versus descriptive attributes, minimal sufficient identity, observer repeatability, lifespan specificity, Kubernetes UID/name/scope/reuse. |
| Scope not inspected | No OTel SDK instrumentation, Kubernetes API-server runtime, or CRD-specific lifecycle behavior. |
| Reliability | High. |
| Freshness risk | Low-medium. |

#### Source overview (telemetry)

OpenTelemetry formalizes telemetry entity identity, not enterprise canonical asset identity. Its transferable principle is that identifying attributes must be minimal, sufficient, stable for the entity lifespan, and repeatable across independent observers. Descriptive attributes may change and must not define identity.[^7]

Kubernetes formalizes object identity in an API-server scope. Names are unique only for a resource type and namespace at a time; deleted object names can be reused; UIDs are generated for each object occurrence across the cluster lifetime. Labels are user-defined, mutable, non-unique selectors; annotations are non-identifying metadata.[^8]

#### Architecture and operational model (telemetry)

Telemetry identity is observer-side and data-plane oriented. It makes telemetry records linkable but does not decide whether an endpoint, VM, container, Kubernetes object, or workload should be merged into a Cadastre canonical host.

Kubernetes identity is API-object identity. A Pod UID identifies an object occurrence in a cluster, not a durable host. Workload objects can create replacement Pods when pod templates change, and Pod metadata such as namespace/name/UID is mostly immutable after creation.[^8]

#### Identifier behavior (telemetry)

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive identity evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| OTel identifying attribute set | Discovery context and entity type | Entity lifespan | Strong only inside OTel entity model | Must not change for lifespan | Low when complete | Supporting identity classification | Missing required attrs means entity type should not be emitted | Use to classify `IdentifierEvidenceClass`, not canonical merge authority. |
| OTel descriptive attribute | Entity record | Observation interval | Weak | Mutable | High | None | None | Descriptive only. |
| Kubernetes UID | Cluster, resource type, namespace as applicable | Object occurrence | Strong for object occurrence | Immutable | Low within cluster lifetime | Positive for same K8s object occurrence | Conflicting UID blocks same object identity | Auto-merge only for same cluster/resource type/object generation; not host identity by itself. |
| Kubernetes name | Namespace and resource type | Until delete/recreate | Medium selector | Can be reused after deletion | High | Candidate selector | Name reuse boundary | Selector only unless paired with UID/generation. |
| Kubernetes label | Object scope | Label lifetime | Weak | Mutable | High | None by default | None by default | Selector/candidate generation only. |
| Kubernetes annotation | Object scope | Annotation lifetime | Weak | Mutable | High | None | None | Context/lineage only. |

#### Matching, correlation, and reconciliation behavior (telemetry)

OpenTelemetry’s key contribution is a negative requirement: if an observer cannot supply minimally sufficient identifying attributes, it must not emit that entity type. Cadastre should mirror this by refusing production merge decisions when the evidence item set lacks the required evidence classes for the selected `decision_matrix_row`.[^7]

#### Confirmed findings versus inferences (telemetry)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| OTel separates identifying and descriptive attributes; identifying attributes must not change, must be minimally sufficient, and should be repeatable across observers.[^7] Kubernetes names are scoped and reusable after deletion; UIDs distinguish object occurrences; labels and annotations are non-unique or non-identifying metadata.[^8] | Telemetry entity identity is narrower than Cadastre canonical identity. | Cadastre should define `identity_role` and `lifecycle_scope` per evidence class. | OTel entity model may inform a source telemetry adapter’s source-local identity rows. | CRD-specific identity behavior requires per-resource type profiles. |

#### Concepts that must not transfer (telemetry)

| Rejected concept | Reason |
| --- | --- |
| Telemetry entity identity as Cadastre canonical identity | It is observer and resource-context identity, not enterprise canonical asset identity. |
| Kubernetes name as durable identity | Names can be reused after deletion. |
| Kubernetes label or annotation equality as identity | Labels and annotations are mutable and non-unique. |
| Workload identity as durable host identity | Pods and workloads are generations of execution, not physical or durable endpoint identity by default. |

### 4.3 Cloud provider and source-specific asset identifiers

#### Source inventory and inspection limits (cloud)

| Field | Content |
| --- | --- |
| Source name | AWS EC2, Azure ARM/IMDS, Google Cloud Asset Inventory. |
| Evidence type | Official cloud provider documentation. |
| Scope inspected | EC2 instance metadata, private/public/EIP assignment, ENIs, tags; Azure management scopes/resource IDs/vmId; Google full resource names/history. |
| Scope not inspected | No API execution, no provider-specific delete/recreate empirical test. |
| Reliability | High for documented fields and scopes. |
| Freshness risk | Medium. |

#### Source overview (cloud)

Cloud providers supply durable source-native IDs, scoped resource paths, network interface IDs, tags/labels, addresses, and region/zone/project/account context. These identifiers are strong within provider scope but do not become global Cadastre canonical identity without resolver governance.

AWS documents EC2 instance IDs, private IP assignment to network interfaces, public IP release/reassignment behavior, Elastic IP association movement, and tags as mutable metadata with no EC2 semantic meaning.[^9] Azure ARM resources are organized under management scopes such as subscriptions, resource groups, and resources, and ARM resource IDs encode subscription, resource group, provider, resource type, and resource name; Azure IMDS exposes `vmId` as a VM unique ID.[^10] Google Cloud Asset Inventory uses full resource names and stores asset metadata history for five weeks.[^11]

#### Identifier behavior (cloud)

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive identity evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AWS EC2 `instance-id` | AWS account plus region | Instance lifetime | Strong | Immutable | Low in provider scope | Positive source asset identity | Conflicting IDs block same instance | Strong source-native ID; not global canonical identity. |
| AWS ENI ID / MAC | Account plus region/VPC | Network interface lifetime | Strong for interface | Interface attachment can move | Medium | Dependent identity for NIC | Attachment conflicts | Dependent `SourceAsset`; not host merge alone. |
| AWS private IPv4 | VPC/subnet/interface interval | Assignment interval | Temporal | Reassignable under rules | High | Temporal address fact | Overlap conflict inside assignment interval | `AddressAssignmentEvidence`; no merge. |
| AWS public IPv4 | AWS public address pool / instance interval | Assignment interval | Temporal | Released on stop/termination unless EIP | High | Temporal observation | Conflict inside assignment interval | Selector only. |
| AWS Elastic IP | AWS account plus region | Allocation until release, association interval | Strong for address allocation, not host | Association mutable | Medium | IP allocation fact | Conflicting association interval | Temporal IP fact, not host identity. |
| AWS tag | Resource scope | Tag lifetime | Weak | Editable/removable | High | None by default | None | Descriptive/context. |
| Azure ARM resource ID | Tenant/subscription/resource group/provider/type/name | Resource path lifetime | Strong path ID | Resource may be moved/renamed depending type; names usually scoped | Medium | Source asset identity | Scope/type conflict | Strong source asset ID; requires generation evidence for delete/recreate. |
| Azure VM `vmId` | Azure VM | VM creation lifetime | Strong | Immutable | Low | Positive source asset identity | Conflicting `vmId` | Strong source-native evidence in Azure scope. |
| Google full resource name | Organization/folder/project/location/type/name | Resource path lifetime | Strong path | Name/path may be reused after delete for some resources | Medium | Source asset identity | Scope/type conflict | Strong source asset selector; require create/delete history for generation. |
| Cloud labels/tags | Resource scope | Label/tag lifetime | Weak | Mutable | High | None | None | Context only. |

#### Matching, correlation, and reconciliation behavior (cloud)

Cloud provider paths must be treated as scoped source-native asset IDs. Provider/account/subscription/project, region/zone/location, resource type, and source instance are part of the identity scope. IP addresses and tags must be temporal attributes or selectors, not identity authority. Network-interface identity must remain distinct from instance identity because interfaces can have independent attachments, IPs, and lifecycle.

#### Confirmed findings versus inferences (cloud)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| AWS private IPv4 remains associated with the network interface across stop/start and is released on termination; public IPv4 can be released on stop/termination; EIPs can be moved; tags are metadata with no EC2 semantic meaning.[^9] Azure exposes management scopes, ARM IDs, and `vmId` for VM uniqueness.[^10] Google CAI full resource names support history and Pub/Sub change monitoring; CAI keeps five-week metadata history.[^11] | Provider identifiers are strong only in provider scope and lifecycle. | Cadastre should require `IdentifierScope` for provider/account/subscription/project/region/zone/resource type. | Google CAI history can feed lifecycle boundary detection. | Provider-specific reuse guarantees must be confirmed per resource type before auto-merge. |

#### Concepts that must not transfer (cloud)

| Rejected concept | Reason |
| --- | --- |
| Cloud source-native ID as global canonical identity | Provider IDs are scoped; global host identity can span sources and lifecycles. |
| Resource path/name as durable generation identity | Paths/names can be reused or moved depending source. |
| IP assignment as identity | Address assignment is temporal and reassignable. |
| Tags/labels as identity | Tags/labels are mutable metadata. |

### 4.4 Endpoint, EDR, MDM, directory, and agent systems

#### Source inventory and inspection limits (scanners)

| Field | Content |
| --- | --- |
| Source name | Microsoft Intune managedDevice, Defender XDR DeviceInfo, Microsoft Graph device, osquery, Fleet. |
| Evidence type | Official documentation. |
| Scope inspected | Device IDs, names, Entra device IDs, Defender device IDs and merge fields, agent/node enrollment. |
| Scope not inspected | No tenant, no reimage/re-enrollment empirical data. |
| Reliability | High for schema fields. |
| Freshness risk | Medium. |

#### Source overview (scanners)

Endpoint systems expose parallel identities for the same host: MDM device record IDs, directory device IDs, EDR/XDR device IDs, agent UUIDs/node keys, serial numbers, hardware UUIDs, MAC addresses, and mutable display names. Cadastre must preserve them as typed source-scoped evidence rather than collapse them into one untyped identifier.

Intune `managedDevice` exposes a read-only unique `id`, a read-only `deviceName`, `azureADDeviceId`, serial number, Wi-Fi MAC, Ethernet MAC, and a mutable user-friendly `managedDeviceName`.[^12] Defender XDR `DeviceInfo` exposes `DeviceId`, FQDN-like `DeviceName`, public IP that may be NAT/proxy, Entra device ID, `ReportId` that must be combined with DeviceName and Timestamp for uniqueness, and source-native merge fields such as `MergedDeviceIds` and `MergedToDeviceId`.[^12] Microsoft Graph device records distinguish directory object `id`, `deviceId`, and display name.[^12]

osquery remote enrollment returns a `node_key`, stored locally and used for future config/log requests; the server can invalidate it and trigger re-enrollment.[^13] Fleet host enrollment requires URL and enrollment secret; Fleet docs advise deleting a host before re-enrolling to remove pending state and original host vitals.[^13]

#### Identifier behavior (scanners)

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Intune managedDevice `id` | Intune tenant | Enrollment/device record | Strong source-local | Read-only | Medium after reenrollment | Positive source asset | Conflicting active ID | Strong source asset, not global by itself. |
| Entra `deviceId` / `azureADDeviceId` | Entra tenant | Directory device object lifecycle | Strong source-local | Read-only | Medium after reenrollment | Strong when paired with endpoint evidence | Conflict hard blocker | Positive evidence within tenant. |
| Defender `DeviceId` | Defender service | Defender device record | Strong source-local | Read-only | Source-native merge possible | Strong source-local | Conflict with merge history | Source asset ID; merge history not authority. |
| Defender `MergedDeviceIds` | Defender service | Source merge lifecycle | Lineage-only | Source-controlled | Medium | None by default | Contradiction evidence | Store as `source_native_merge_history_row`. |
| osquery `node_key` | TLS enrollment server | Agent enrollment | Strong source-local | Reissued on re-enrollment | Medium | Source-local agent identity | Reinstall/re-enroll conflict | Positive only with lifecycle continuity. |
| Fleet enroll secret | Fleet team/tenant | Enrollment authorization | Not asset identity | Rotatable | High | None | None | Credential/config, not identity evidence. |
| Serial number | Hardware/vendor scope | Device lifetime | Medium-high | Usually stable, source quality varies | Medium due replacement/reuse/data quality | Supporting strong evidence | Conflict hard blocker when authoritative | Positive but requires clone/duplicate checks. |
| Hardware UUID | Firmware/virtualization scope | Image/device generation | Medium-high | May duplicate in clones | High in clone/golden image scenarios | Supporting strong evidence | Clone suspicion hard blocker | Never auto-merge when duplicate active. |
| Management name/deviceName/displayName | Source/tenant | Name lifetime | Weak | Mutable | High | Candidate hint | None by itself | Selector only. |

#### Matching, correlation, and reconciliation behavior (scanners)

Endpoint identity must account for reimage, clone, agent reinstall, directory reenrollment, golden image, VDI, and mutable display names. Source-native EDR merge histories must remain evidence about that source’s internal correlation model. They may explain why a source record changed, but they must not authorize Cadastre canonical merges.

#### Confirmed findings versus inferences (scanners)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Intune, Defender, and Graph device schemas expose multiple IDs and mutable names; Defender includes source-native merged-device fields; osquery/Fleet enrollment creates agent-local keys.[^12][^13] | Endpoint sources frequently expose parallel and sometimes conflicting identifiers for the same machine. | Cadastre must store evidence class and lifecycle scope for every endpoint identifier. | Defender merge fields can feed split-validation fixtures. | Exact reinstall/reimage behavior differs by product and deployment. |

#### Concepts that must not transfer (scanners)

| Rejected concept | Reason |
| --- | --- |
| EDR source-native merge as Cadastre merge | It is the source’s model, not Cadastre canonical authority. |
| Agent node key as durable host across reinstall | Re-enrollment can create new keys and stale state. |
| Hardware UUID as unconditional auto-merge | Clones and golden images can duplicate it. |
| Management display name as identity | Names are mutable and reused. |

### 4.5 Vulnerability scanner identity and asset correlation

#### Source inventory and inspection limits (endpoint)

| Field | Content |
| --- | --- |
| Source name | Tenable Security Center, Rapid7 InsightVM. |
| Evidence type | Official documentation. |
| Scope inspected | Tenable IA priority/conflict behavior, duplicate causes, IP/agent tracking; Rapid7 asset correlation, UUIDs, site linking, VDI, multi-NIC behavior. |
| Scope not inspected | No live scanner, no API record corpus, no scanner version-by-version algorithm diff. |
| Reliability | High for documented behavior. |
| Freshness risk | Medium. |

#### Source overview (endpoint)

Scanner identity models optimize vulnerability result correlation, not enterprise canonical host identity. They can be excellent evidence for vulnerability observation lineage and coverage, but scanner asset IDs and scanner correlation algorithms must be limited by source authority and completeness.

Tenable Security Center tracks imported assets by attributes in descending priority: Tenable UUID, BIOS UUID, MAC address, NetBIOS name, FQDN, IP. It verifies no conflicting higher-priority attributes before merge and can create duplicate assets when non-credentialed scans cannot access stronger identifiers or when multi-interface assets expose different MAC addresses.[^14]

Rapid7 InsightVM correlates scan data into unique asset records using attributes such as hostname, IP, MAC, UUIDs, VM IDs, and network interfaces. Agent UUID correlation improves accuracy when scan data is otherwise insufficient. Rapid7’s multi-NIC documentation explicitly describes previous over-correlation failure modes that could incorrectly remediate vulnerabilities when different interfaces were scanned.[^15]

#### Identifier behavior (endpoint)

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Tenable UUID | Tenable repository/source instance | Scanner asset/credentialed host | Strong source-local | Source-controlled | Medium | Source asset identity | Higher-priority conflict behavior | Scanner `SourceAsset`, not canonical merge alone. |
| BIOS UUID from scanner | Scanner observed host | Device/generation | Medium-high | Clone risk | High in VM clone | Supporting evidence | Clone suspicion | Positive only with stronger scoped evidence. |
| Scanner MAC | Scanner observed interface | Interface/scan | Medium | Multi-NIC, virtual NIC changes | High | Supporting selector | Multi-NIC conflict | Candidate only unless paired. |
| Scanner hostname/FQDN/IP | Scanner observed endpoint | Observation interval | Weak/temporal | Mutable/reassignable | High | Candidate hint | None by itself | Selector only. |
| Rapid7 asset ID | InsightVM console/site | Scanner asset record | Strong source-local | Source-controlled | Algorithm changes/cleanup | Source asset identity | Correlation-change conflict | Scanner `SourceAsset`. |
| Rapid7 Agent UUID/R7CorrelationIdentifier | Rapid7 environment | Agent or VDI correlation lifecycle | Strong source-local | Configurable | Medium, especially golden image | Source-local evidence | VDI/golden-image blocker | Strong only under source scope and lifecycle boundary. |
| Network interface identity | Scanner observed interface | Interface scan interval | Medium | Multi-NIC changes | Medium | Vulnerability endpoint correlation | Interface conflict | Dependent evidence for vulnerabilities. |

#### Matching, correlation, and reconciliation behavior (endpoint)

The transferable pattern is Tenable’s ordered matching with higher-priority conflicts blocking lower-priority merges. The rejected pattern is importing scanner merge output as Cadastre merge authority. Scanner identity is evidence about scanner result grouping and coverage prerequisites, not canonical host truth.

#### Confirmed findings versus inferences (endpoint)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Tenable uses ordered identifiers and higher-priority conflict checks; Agent assets are tracked by UUID.[^14] Rapid7 correlates UUIDs, site/linking behavior, VDI correlation identifiers, and multi-NIC vulnerability correlation.[^15] | Scanner correlation algorithms optimize scan result grouping and can change over time. | Cadastre must store scanner `SourceAsset` identity and preserve scanner conflict/algorithm-change evidence. | Scanner IA priority can inform validation scenarios. | Exact Rapid7 asset-correlation algorithm internals were not inspected. |

#### Concepts that must not transfer (endpoint)

| Rejected concept | Reason |
| --- | --- |
| Scanner asset ID as canonical host identity | Scanner records are source-local and algorithm-dependent. |
| Scanner correlation as Cadastre merge authority | It can over- or under-correlate due scan method, credentials, multi-NIC, and VDI behavior. |
| Vulnerability absence without coverage | Vulnerability absence requires `CoverageAssertion` plus source completeness and authority. |
| IP-only scanner match | DHCP/dynamic IP and scan scope make this unsafe. |

### 4.6 DNS, DHCP, IPAM, and network-flow systems

#### Source inventory and inspection limits (network)

| Field | Content |
| --- | --- |
| Source name | RFC 2131 DHCP, RFC 1035 DNS, Zeek conn.log, NetBox. |
| Evidence type | Internet standards and official docs. |
| Scope inspected | Lease times, DNS TTL/caching, Zeek UID, NetBox desired-state boundary. |
| Scope not inspected | No authoritative DNS/DHCP logs, no NetBox deployment. |
| Reliability | High. |
| Freshness risk | Low for RFCs, medium for tools. |

#### Source overview (network)

Network evidence is temporal and contextual. DHCP leases identify address assignments during lease intervals. DNS records are cacheable for TTL intervals and can be stale or cached. PTR records are reverse DNS assertions, not host identity. Zeek connection UIDs are log correlation IDs for a connection, not asset identity. NetBox is intended to represent desired network state, not live observed state by default.[^16]

#### Identifier behavior (network)

| Identifier | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DHCP lease | DHCP server/scope | Lease interval | Temporal | Renew/rebind/reassign | High after expiry | Address assignment evidence | Expired lease blocks dependent observation | `AddressAssignmentEvidence`, no merge. |
| DNS A/AAAA/CNAME | Zone/resolver/record | TTL interval | Temporal | Mutable | High after TTL | DNS resolution evidence | TTL expired blocks dependent identity | `DnsResolutionEvidence`, no merge. |
| PTR record | Reverse DNS zone | TTL interval | Weak temporal | Mutable | High | Candidate hint | None by itself | `no_decision` by default. |
| IP address | Network scope | Assignment interval | Temporal | Reusable | High | Temporal fact | Overlap conflict inside lease/association | Address entity/fact, not host identity. |
| Zeek `uid` | Zeek sensor/log set | Connection | Log correlation | Immutable per log record | None for asset; unique per connection | None for identity | None for identity | `FlowEndpointEvidence` lineage/correlation. |
| NetBox object ID | NetBox tenant/site | Intended-state object lifecycle | Strong intended-state | Operator editable | Medium | Intended-state evidence | Conflict with observed evidence is review case | `IpamIntentEvidence`, not observed host identity. |

#### Matching, correlation, and reconciliation behavior (network)

Cadastre default for IP-only, DNS-only, PTR-only, and flow-only evidence must be `no_decision`. These evidence classes may create candidate pairs and temporal facts, but they cannot create or merge canonical hosts without stronger scoped identifiers and valid-time evidence.

#### Confirmed findings versus inferences (network)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| DHCP clients record lease duration and must discontinue prior address use when the lease expires; DNS TTL specifies how long RRs may be cached; Zeek `uid` links related logs for a connection; NetBox intends desired state and discourages automated import of live state without vetting.[^16] | Network identifiers are inherently temporal or intended-state. | Cadastre should define `AddressAssignmentEvidence`, `DnsResolutionEvidence`, `FlowEndpointEvidence`, and `IpamIntentEvidence`. | NetBox intended-state diffs may become review hints. | NAT/proxy/firewall collection-point semantics require source-specific evidence rows. |

#### Concepts that must not transfer (network)

| Rejected concept | Reason |
| --- | --- |
| IPAM intended state as observed host identity | Intended state is not observation. |
| DNS current answer as durable identity | TTL and cache behavior limit validity. |
| Flow UID as host identity | It is a connection correlation key. |
| PTR as host identity | PTR is mutable reverse DNS. |

### 4.7 General entity-resolution frameworks

#### Source inventory and inspection limits (entity-resolution)

| Field | Content |
| --- | --- |
| Source name | Splink, Dedupe, py_entitymatching, Zingg. |
| Evidence type | Official docs and repository docs. |
| Scope inspected | Probabilistic linkage, Fellegi-Sunter, blocking, active labeling, training settings, thresholds, explainability, blocking diagnostics. |
| Scope not inspected | No local training, no production ER datasets. |
| Reliability | Medium-high. |
| Freshness risk | Medium. |

#### Source overview (frameworks)

General ER frameworks solve record linkage across datasets with statistical, supervised, or learned methods. They are valuable for candidate generation, blocking, review sampling, labeling workflows, score explanation, and calibration. They do not know Cadastre’s source scopes, lifecycle boundaries, authority boundaries, completeness, or graph-correction obligations.

Splink is probabilistic record linkage based on Fellegi-Sunter and performs best with multiple non-highly-correlated columns; its docs warn correlation among columns can be problematic.[^17] Dedupe uses active learning and returns confidence scores/clusters; it can save learned settings and has active labeling workflows.[^17] py_entitymatching identifies blocking and matching as two main EM steps; Zingg exposes blocking diagnostics and warns that bias, insufficient labels, nulls, or non-differentiating columns can degrade blocking.[^17]

#### Identifier behavior (frameworks)

| ER artifact | Source scope | Lifecycle scope | Durability | Mutability | Reuse risk | Positive evidence | Negative evidence | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Blocking key | Candidate generation profile | Profile version | Deterministic if configured | Profile changes | High if weak fields | None by itself | None by itself | Candidate-generation only. |
| Match score | Model/profile run | Model artifact version | Output score | Changes with model/data | Calibration risk | Score hint only | Low score can route review | Non-authoritative input to decision matrix. |
| Training label | Labeling workflow | Training artifact | Durable artifact if checksummed | Reviewer correction possible | Human error | May calibrate model | May define negative examples | `ResolverTrainingArtifact`, not production decision. |
| Active label `unsure/not sure` | Review workflow | Label session | Durable review input | Can be resolved later | Human uncertainty | None | Review route | Map to `needs_more_evidence`. |
| Explanation report | Model output | Run | Derived | Model changes | Medium | Audit support | Audit support | Non-normative unless mapped to `ResolverExplanation` fields. |

#### Matching, correlation, and reconciliation behavior (frameworks)

Cadastre must not allow a probabilistic model to directly emit production merges. Probabilistic output may produce candidates, a `score_hint`, or a proposed confidence input only when the active `ResolverProfile` declares the model artifact checksum, feature set, input canonicalization, blocked fields, calibration scenario set, confidence mapping, hard blockers, and decision-matrix rows.

Correlated weak identifiers violate naive independence assumptions. Hostname, FQDN, DNS answer, PTR record, IP address, scanner name, and management display name often derive from the same administrative naming/addressing scheme or from each other. A probabilistic model that treats them as independent features can overstate confidence.

#### Confirmed findings versus inferences (frameworks)

| Confirmed source facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Splink implements probabilistic linkage and warns about correlated columns; Dedupe uses active learning and confidence scores; py_entitymatching separates blocking and matching; Zingg validates blocking model quality.[^17] | ER frameworks are useful before and around merge decisions, not as final authority. | Cadastre should wrap ER models in deterministic resolver governance. | Learned blocking may improve performance in shadow. | Cadastre-specific calibration requires production-like golden identity scenarios. |

#### Concepts that must not transfer (frameworks)

| Rejected concept | Reason |
| --- | --- |
| Probabilistic model output as direct production merge | It lacks Cadastre source authority, blockers, lifecycle, and graph correction. |
| Naive feature independence across weak identifiers | Weak fields are often correlated. |
| Review UI label as identity mutation | Only emitted `IdentityDecision` may mutate identity. |
| Model threshold as policy | Thresholds must be governed by `ResolverProfile`. |

### 4.8 Adjacent graph and asset-intelligence systems

#### Source inventory and inspection limits (graph)

| Field | Content |
| --- | --- |
| Source name | JupiterOne/Starbase, Cartography, BloodHound/SharpHound/OpenGraph, Taxi semantic overlays. |
| Evidence type | Uploaded research reports based on source/docs inspection; not re-inspected here except through those reports. |
| Scope inspected | Graph-first ingestion, provider-scoped keys, target matching, graph cleanup hazards, traversability, source-kind deletion, semantic overlay alias risks. |
| Scope not inspected | No local builds or source execution in this report. |
| Reliability | Medium-high as prior research evidence; not governing. |
| Freshness risk | Medium. |

#### Source overview (graph)

JupiterOne and Starbase are graph-first integration/synchronization references; provider-scoped `_key` values and mapped relationship target creation are graph synchronization mechanisms, not deterministic Cadastre identity decisions.[^18] Cartography is a Neo4j-backed infrastructure asset graph with provider modules, graph schema declarations, query generation, cleanup, and analysis jobs; it is a graph ingestion/analysis tool, not a temporal lakehouse fact system.[^19] BloodHound/OpenGraph provides strong graph semantics such as stable IDs, endpoint matching, traversability, structured versus generic graph behavior, and source-kind deletion hazards, but must remain analysis/projection reference material rather than identity authority.[^20] Taxi semantic overlays are valuable source-schema authoring aids but must not become identity evidence or Cadastre semantics authority.[^21]

#### Identifier behavior (graph)

| External graph identifier | Scope | Durability | Mutability | Reuse risk | Cadastre default treatment |
| --- | --- | --- | --- | --- | --- |
| JupiterOne `_key` | Provider/integration graph | Strong graph key | Integration-defined | Medium | Graph/source asset key only; no canonical merge. |
| Mapped relationship target filter | Graph sync target selection | Weak to medium | Query/filter-defined | High | `UnresolvedTargetReference`; must not create/merge. |
| Cartography node `id` | Neo4j model/provider | Strong graph key | Module-defined | Medium | Source graph key, not canonical identity. |
| OpenGraph node `id` | Uploaded graph extension/source | Intended stable source ID | Source-defined | Medium | May create `UnresolvedTargetReference`; no merge. |
| OpenGraph property/name matching | Graph endpoint match | Weak selector | Mutable | High | Reject or unresolved selector. |
| Taxi semantic type/alias | Schema overlay | Meaning mapping | Tooling-defined | High for identity | Zero positive identity by default. |

#### Matching, correlation, and reconciliation behavior (graph)

The transferable pattern is explicit graph-edge semantics, traversability, target selector safety, and source-kind deletion safety. The rejected pattern is graph-backend cleanup, graph sync, or graph key equality as identity authority.

#### Confirmed findings versus inferences (graph)

| Confirmed/prior-research facts | Source-grounded inferences | Cadastre applicability inferences | Speculative transfers | Open questions |
| --- | --- | --- | --- | --- |
| Prior research identifies JupiterOne provider-scoped graph keys and mapped relationships as graph mechanisms, Cartography as direct Neo4j graph ingestion, BloodHound/OpenGraph stable ID and endpoint matching behavior, and Taxi overlays as non-authoritative authoring aids.[^18][^19][^20][^21] | Graph systems often solve projection and analysis, not canonical identity. | Cadastre must keep `UnresolvedTargetReference`, `TargetSelectorSafetyPolicy`, and graph-derived hints separate from `IdentityDecision`. | OpenGraph traversability can improve `GraphEdgeSemantics`. | Current graph system releases should be rechecked before PRD text quotes exact versions. |

#### Concepts that must not transfer (graph)

| Rejected concept | Reason |
| --- | --- |
| Graph node IDs as canonical entity IDs | Graph IDs are projection/source keys. |
| Mapped target references creating entities | Target matching is not identity resolution. |
| Graph cleanup as absence or split authority | Backend cleanup is not source completeness or canonical correction. |
| Semantic overlay equality as identity | Overlay equality is field meaning, not entity equality. |

## 5. Cross-source synthesis

### Durable patterns

| Pattern | Sources | Cadastre consequence |
| --- | --- | --- |
| Identity rules must be scoped. | ServiceNow class rules; OTel discovery context; Kubernetes namespace/resource scope; cloud provider account/subscription/project/region/zone scopes.[^5][^7][^8][^9][^10][^11] | Every identity evidence item must carry `source_scope` and `lifecycle_scope`. |
| Strong IDs still have lifecycle boundaries. | Kubernetes UID vs name reuse; Azure/GCP/AWS resource lifecycles; endpoint reenrollment; scanner algorithm changes.[^8][^9][^10][^11][^12][^15] | Add `AssetGenerationBoundary` and do not treat same value as continuous entity without boundary evaluation. |
| Attribute authority differs from identity authority. | ServiceNow reconciliation versus identification; PRD `SourceAuthorityProfile` for gold facts.[^5][^2] | Identity resolver must not use attribute-authority rows as match rules. |
| Temporal evidence must retain validity windows. | DHCP leases; DNS TTL; public/private/EIP assignments; flow logs.[^9][^16] | IP/DNS/flow evidence can produce temporal facts and candidates, not auto-merge. |
| Ordered matching requires hard blockers. | Tenable higher-priority conflict checks; PRD negative evidence; CMDB identification priority.[^14][^3][^5] | Evaluate blockers before confidence. |
| Review and activation must be non-mutating until terminal output. | ServiceNow simulation; ER active labeling; PRD validation matrix.[^5][^17][^6] | `ResolverShadowRun` and `IdentityReviewCase` must not mutate identity until `IdentityDecision` emitted. |
| Graph systems need explicit target safety. | JupiterOne, Cartography, OpenGraph, PRD target selector policy.[^18][^19][^20][^2] | Graph-derived hints remain unresolved unless resolver emits qualifying decisions. |

### Non-transferable assumptions

| Assumption | Why unsafe | Default Cadastre boundary |
| --- | --- | --- |
| Source-native asset IDs are global identity. | Source IDs are scoped and may reflect source correlation algorithms. | Source-scoped strong evidence only. |
| Hostname/FQDN/DNS/IP/PTR are independent signals. | They are correlated and mutable. | Candidate generation only unless paired with strong evidence. |
| Scanner asset records are canonical hosts. | Scanner correlation optimizes vulnerability reporting and can over/under-correlate. | Scanner `SourceAsset` plus vulnerability evidence. |
| Graph key equality resolves identity. | Graph keys are projection/source-model artifacts. | `UnresolvedTargetReference` until resolver decision. |
| Review UI state can mutate identity. | Hidden mutation breaks replay. | Terminal `IdentityDecision` required. |
| Source-native merge history proves Cadastre merge. | It describes source’s model, not Cadastre authority. | Store as `lineage_only` or contradiction evidence. |
| IPAM intended state proves observed host identity. | Intended state may differ from operational state. | `IpamIntentEvidence` only. |
| OTel/Kubernetes resource identity is enterprise canonical identity. | It is resource/telemetry object identity, scoped and lifespan-specific. | Source/resource identity only. |

### Source-specific transfer map

| Source kind | Strong identifiers | Weak or mutable identifiers | Temporal evidence | Negative evidence to preserve | Default Cadastre decision | Non-transferable source behavior |
| --- | --- | --- | --- | --- | --- | --- |
| Endpoint agent | Agent UUID/node key in source scope | Agent hostname/display name | Agent last_seen | Reinstall/re-enrollment, duplicate active node key | Merge only with lifecycle continuity and no blocker; otherwise candidate/review | Agent ID as durable host across reinstall. |
| MDM and directory | ManagedDevice ID, Entra directory device ID | Device name, managed display name | Enrollment and sync times | Directory ID conflict, reenrollment boundary | Strong source-scoped evidence | Directory display name as identity. |
| EDR and XDR | EDR DeviceId, AadDeviceId | DeviceName, tags, public IP | First/last seen, report timestamp | Source-native merge contradiction | Source asset evidence; merge history lineage only | EDR merge history as Cadastre authority. |
| Cloud provider | Instance/resource/vmId IDs scoped by account/subscription/project/location/type | Names, tags, labels | IP/EIP/ENI attachment intervals | Delete/recreate, resource-scope conflict | Strong source asset; canonical merge only by profile | Cloud path as global identity. |
| Kubernetes | UID in cluster/resource type | Name, labels, annotations | Object create/delete/generation | Name reuse, object recreate | Strong object-generation evidence | Pod UID as durable host identity. |
| Vulnerability scanner | Scanner asset ID, scanner UUID | Hostname, FQDN, IP, scanner name | Scan time, interface scan time | Higher-priority conflict, multi-NIC ambiguity, algorithm change | Scanner source asset only | Scanner correlation as canonical host identity. |
| DNS | Authoritative record plus TTL | Cached answer, PTR, alias | TTL interval | TTL expired before dependent observation | DNS temporal fact/candidate | DNS-only merge. |
| DHCP/IPAM | Lease record; IPAM object ID for intended state | Reservation label/name | Lease interval | Expired lease, lease overlap, intended/observed mismatch | Address assignment or intent evidence | IPAM intended state as observed identity. |
| Firewall and Zeek | Flow 5-tuple plus Zeek `uid` for log correlation | NATed endpoint, proxy endpoint | Flow interval | NAT/proxy/collector ambiguity | Flow endpoint evidence, no identity merge | Flow-only identity. |
| Graph-first systems | Source graph key within source | Name/property match, mapped target filter | Graph sync timestamps | Source-kind deletion, graph cleanup | Unresolved target/reference only | Graph key or target matching as identity. |
| Semantic overlays | Declared evidence-class type classification | Alias/type equality | None by default | Overlay-only equality | Zero positive evidence by default | Overlay equality as entity identity. |

## 6. Identifier evidence registry recommendation

Cadastre should add a closed `IdentifierEvidenceClass` registry and materialize each observed value as an `IdentityEvidenceItem`. The table below is exhaustive for the declared scope of this report.

| Identifier or evidence class | Default scope | Durability class | Lifecycle scope | Mutability | Reuse risk | Source quality | Source authority | Positive-evidence eligibility | Negative-evidence eligibility | Auto-merge eligibility | Default Cadastre treatment | Required validation scenario |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Source-native durable asset ID | Source instance + tenant/account/project/site + asset type | `source_durable` | Source asset record generation | Low | Low-medium | Source-declared | Source asset only | Yes | Yes | Same source scope only, no blockers | Strong source-scoped evidence | Same ID same source; same ID different scope; conflicting IDs. |
| Agent UUID or node key | Agent server/tenant/source instance | `source_durable` | Agent enrollment generation | Low until reinstall | Medium | Source-declared | Source asset only | Yes with continuity | Yes | Conditional | Strong, but boundary-sensitive | Agent reinstall with and without continuity. |
| Directory device ID | Directory tenant | `source_durable` | Directory device object lifecycle | Low | Medium after reenrollment | Source-declared | Source asset and endpoint evidence | Yes | Yes | Conditional with endpoint evidence | Strong tenant-scoped evidence | Directory ID conflict and reenrollment. |
| Cloud instance/resource ID | Provider + account/subscription/project + region/zone + resource type | `source_durable` | Cloud resource generation | Low | Medium on path reuse | Provider-declared | Source asset only | Yes | Yes | Same provider scope/resource type only | Strong source asset evidence | Delete/recreate and scope mismatch. |
| Kubernetes UID | Cluster + resource type + namespace when namespaced | `object_occurrence` | Kubernetes object generation | Immutable | Low in cluster lifetime | API-server declared | Object identity only | Yes for same object | Yes | Same cluster/resource type/object only | Strong workload/object evidence | Same name new UID after delete. |
| Hardware UUID | Hardware/firmware/virtualization source | `hardware_claim` | Device or VM image generation | Usually low | High in clone/golden image | Variable | Supporting only | Yes | Yes | No when duplicate active | Supporting strong, clone-sensitive | Cloned hardware UUID. |
| Serial number | Vendor/device | `hardware_claim` | Device lifetime | Low, quality varies | Medium | Variable | Supporting only | Yes | Yes | Conditional | Supporting strong | Serial conflict and missing/placeholder serial. |
| MAC address | L2 interface/vendor | `interface_claim` | Interface generation | Medium | Medium-high with virtualization/NIC replacement | Variable | Interface evidence | Supporting only | Yes | No by itself | Supporting/candidate | MAC-only rejection and multi-NIC. |
| FQDN | DNS zone/source observation | `weak_name` | Name assignment/TTL/observation | Mutable | High | Variable | Selector only | Supporting with MAC/time | Limited | No by itself | Candidate selector | FQDN+MAC+overlap and FQDN-only rejection. |
| Hostname | Source/source domain | `weak_name` | Name assignment/observation | Mutable | High | Variable | Selector only | Candidate only | Limited | No | Candidate selector | Hostname-only rejection and hostname reuse. |
| IP address | Network/scope | `temporal_address` | Assignment interval | Reusable | High | Variable | Address fact only | Candidate only | Yes for interval conflict | No | Temporal address fact | IP-only rejection and DHCP reassignment. |
| PTR record | Reverse DNS zone/resolver | `weak_name` | TTL interval | Mutable | High | Variable | Selector only | Candidate only | Limited | No | Candidate selector | PTR-only rejection. |
| Vulnerability scanner asset ID | Scanner source instance/repository/site | `source_durable` | Scanner asset record | Source-controlled | Medium | Scanner-declared | Scanner observations only | Source asset only | Yes | No by itself | Scanner `SourceAsset` | Scanner duplicate/over-correlation. |
| Firewall or Zeek flow UID | Sensor/log set | `correlation_id` | Connection/log event | Immutable per log | None for asset | Sensor-declared | Lineage only | No | No | Never | Flow correlation only | Flow-only rejection. |
| DNS record | Authoritative zone or resolver cache | `temporal_name_resolution` | TTL interval | Mutable | High | Depends authoritative/cached | DNS fact only | Candidate only | TTL-expiry blocker | No | DNS temporal fact | DNS-only rejection and TTL expiry. |
| DHCP lease | DHCP server/scope | `temporal_address_assignment` | Lease interval | Renewable/reassignable | High after expiry | Server-declared | Address assignment only | Candidate only | Lease conflict/expiry | No | Address assignment evidence | Lease expired before dependent observation. |
| IPAM/DCIM object ID | IPAM tenant/site | `intended_state` | Intended object lifecycle | Operator editable | Medium | Human-vetted intended state | Intent only | Context only | Intent/observed conflict | No | `IpamIntentEvidence` | Intended state mismatch. |
| Source-native merge-history row | Source service | `lineage_event` | Source merge event | Source-controlled | Medium | Source-declared | Lineage only | No by default | Yes when contradicts stronger evidence | Never | Lineage/contradiction evidence | Defender-style merged device IDs. |
| Transient or ephemeral marker | Source scope | `lifecycle_marker` | Workload/session/generation | Source-controlled | High | Source-declared | Boundary only | No by default | Yes | Never | Blocks auto-merge by default | Ephemeral cloud workload/VDI marker. |
| Semantic overlay alias or type equality | Mapping/overlay artifact | `semantic_classification` | Overlay version | Tooling-defined | High | Tooling-derived | Classification only | No | No | Never | Zero identity evidence | Overlay-only equality rejection. |

## 7. Resolver governance contract recommendations

### 7.1 `ResolverProfile`

Purpose: `ResolverProfile` is the sole production authority for identity decision behavior. Production identity resolution must fail with `RESOLVER_PROFILE_NOT_ACTIVE` when no active `ResolverProfile` covers the requested `entity_type`, source scopes, evidence classes, lifecycle boundary types, and resolver run mode.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `resolver_profile_id` | id | Yes | none | Namespace `resolver_profile`. |
| `profile_version` | string | Yes | none | Immutable semantic version after activation. |
| `profile_status` | enum | Yes | `draft` | One `LifecycleStatus`; only `active` may produce production decisions. |
| `entity_type` | enum | Yes | none | Default host MVP value: `host`. |
| `source_scope_coverage_rows` | array[row] | Yes | none | Exhaustively covers allowed source categories and scope keys. |
| `candidate_generation_profile_id` | id | Yes | none | Must reference active profile. |
| `evidence_class_registry_version` | string | Yes | none | Must include all evidence classes used by rows. |
| `identifier_canonicalization_registry_version` | string | Yes | none | Must define canonicalization for all identifier types. |
| `negative_evidence_blocker_rows` | array[row] | Yes | required | Closed blocker table; IDs unique. |
| `lifecycle_boundary_rows` | array[row] | Yes | required | Covers every boundary type in Section 8.3. |
| `decision_matrix_rows` | array[row] | Yes | required | Exhaustive for default host resolver matrix. |
| `dependent_identity_rows` | array[row] | Yes | `[]` | Required for NICs, pods, volumes, scanner findings, interfaces. |
| `confidence_algorithm` | enum | Yes | `deterministic_rule_band` | Closed values below. |
| `confidence_band_rows` | array[row] | Yes | required | Total mapping from confidence/rule band to band enum. |
| `review_policy_id` | id | Yes | none | Must reference active `IdentityReviewPolicy` or embedded review rows. |
| `split_policy_rows` | array[row] | Yes | required | Must specify known-time correction and graph handoff. |
| `source_native_merge_history_policy` | enum | Yes | `lineage_only` | `lineage_only`, `contradiction_evidence`, `review_hint`; never merge authority in MVP. |
| `activation_scenario_set_id` | id | Yes | none | Must include required scenarios. |
| `shadow_requirements` | object | Yes | required | Declares minimum shadow run inputs and output comparisons. |
| `canary_requirements` | object | Yes | required | Declares allowed canary outputs and no-mutation rules. |
| `explanation_output_requirements` | object | Yes | required | Must require structured `ResolverExplanation`. |
| `checksum` | sha256_hex | Yes | computed | Canonical JSON hash excluding volatile fields. |

Closed `confidence_algorithm` enum:

| Value | Meaning | Production default |
| --- | --- | ---: |
| `deterministic_rule_band` | Decision row assigns confidence band and value from closed table. | Yes |
| `weighted_score` | Profile declares deterministic weights and blocker precedence. | No |
| `external_model_wrapped` | External model produces score hint only; profile maps score to decisions after blockers. | No |
| `source_native_confidence_wrapped` | Source score is input evidence, not Cadastre authority. | No |
| `reviewer_confidence_only` | Reviewer supplies confidence for review terminal decision under authority rules. | No |

### 7.2 `IdentityEvidenceItem`

Purpose: normalized typed evidence row for every positive, negative, context, selector, candidate-generation, and lineage-only evidence item.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `identity_evidence_item_id` | id | Yes | none | Namespace `identity_evidence_item`; deterministic from source asset, identifier, evidence class, valid interval, observed time, and evidence refs. |
| `source_asset_id` | id or null | Yes | null | Required unless identifier-only evidence. |
| `identifier_id` | id or null | Yes | null | Required unless source-asset-only evidence. |
| `evidence_class` | enum | Yes | none | One value from active registry. |
| `evidence_role` | enum | Yes | none | `positive_identity`, `negative_identity`, `context`, `selector`, `candidate_hint`, `lifecycle_boundary`, `lineage_only`. |
| `source_scope` | object | Yes | none | Must include required scope keys for evidence class. |
| `valid_interval` | interval or null | Yes | null | Required for temporal evidence. |
| `observed_time` | timestamp | Yes | none | Source observation or collected time per evidence class. |
| `known_time` | timestamp | Yes | none | Platform knowledge time. |
| `durability_class` | enum | Yes | derived | Must match evidence class registry. |
| `reuse_risk` | enum | Yes | derived | `low`, `medium`, `high`, `unknown`. |
| `source_quality` | enum | Yes | `unknown` | `authoritative_source`, `credentialed_observation`, `uncredentialed_observation`, `cached`, `intended_state`, `source_native_merge`, `unknown`. |
| `confidence_contribution` | decimal_0_1 | Yes | `0.0000` | Must be zero for non-positive roles. |
| `blocker_code` | enum or null | Yes | null | Required when role is `negative_identity`. |
| `evidence_refs` | array[EvidenceRef] | Yes | none | Must contain at least one source evidence ref. |

### 7.3 `AssetGenerationBoundary`

Purpose: lifecycle boundary evidence and default resolver behavior.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `asset_generation_boundary_id` | id | Yes | none | Namespace `asset_generation_boundary`. |
| `boundary_type` | enum | Yes | none | One value from Section 8.3. |
| `subject_source_asset_ids` | array[id] | Yes | none | At least one. |
| `source_scope` | object | Yes | none | Must match boundary type. |
| `valid_from` | timestamp or null | Yes | null | Required when source provides boundary valid time. |
| `valid_to` | timestamp or null | Yes | null | Half-open interval when present. |
| `known_from` | timestamp | Yes | none | Platform knowledge start. |
| `evidence_class` | enum | Yes | none | Must be lifecycle-compatible. |
| `evidence_refs` | array[EvidenceRef] | Yes | none | At least one. |
| `default_resolver_effect` | enum | Yes | `block_auto_merge` | `block_auto_merge`, `force_review`, `split_existing_merge`, `candidate_only`, `no_effect`. |
| `review_required` | boolean | Yes | true | Must be true for destructive/split effects. |

### 7.4 `IdentityReviewCase`

Purpose: deterministic candidate, conflict, and split review state. Review state must not mutate identity unless a terminal state emits an `IdentityDecision`.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `review_case_id` | id | Yes | none | Namespace `identity_review_case`. |
| `case_type` | enum | Yes | none | `candidate`, `conflict`, `split`, `appeal`, `manual_override`. |
| `state` | enum | Yes | `queued` | One closed state below. |
| `candidate_pair_key` | string | Yes | none | Canonical ordered pair or cluster key. |
| `resolver_profile_id` | id | Yes | none | Must reference active/canary profile at case creation. |
| `required_reviewer_authority` | enum | Yes | `identity_reviewer` | `identity_reviewer`, `senior_identity_reviewer`, `security_architect`, `product_governance`. |
| `assigned_reviewer_id` | id or null | Yes | null | Required in `assigned`. |
| `created_at` | timestamp | Yes | none | Platform time. |
| `expires_at` | timestamp | Yes | profile default | Must be after `created_at`. |
| `terminal_decision_id` | id or null | Yes | null | Required for terminal states that emit decision. |
| `evidence_snapshot_checksum` | sha256_hex | Yes | none | Hash of reviewed evidence items and refs. |
| `manual_override_reason_code` | enum or null | Yes | null | Required for override. |
| `audit_evidence_refs` | array[EvidenceRef] | Yes | `[]` | Required for terminal outputs. |

Closed states and transitions:

| From state | Permitted event | To state | Terminal output |
| --- | --- | --- | --- |
| `queued` | `assign` | `assigned` | none |
| `queued` | `expire` | `expired` | `no_decision` review outcome |
| `assigned` | `approve_merge` | `approved_merge` | `IdentityDecision.merge` |
| `assigned` | `reject_match` | `rejected_match` | `IdentityDecision.reject` |
| `assigned` | `confirm_conflict` | `confirmed_conflict` | `IdentityDecision.conflict` |
| `assigned` | `approve_split` | `approved_split` | `IdentityDecision.split` plus graph correction handoff |
| `assigned` | `request_more_evidence` | `needs_more_evidence` | none |
| `needs_more_evidence` | `evidence_arrived` | `queued` | none |
| `needs_more_evidence` | `expire` | `expired` | `no_decision` review outcome |
| Any non-terminal | `close_no_action` | `closed_no_action` | none |

### 7.5 `ResolverActivationReport`

Purpose: validation gates before a resolver profile becomes active.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `resolver_activation_report_id` | id | Yes | none | Namespace `resolver_activation_report`. |
| `resolver_profile_id` | id | Yes | none | Candidate profile. |
| `profile_version` | string | Yes | none | Must match profile. |
| `scenario_results` | array[row] | Yes | none | Must include every required scenario in Section 11. |
| `shadow_run_ids` | array[id] | Yes | `[]` | Required before `active`. |
| `canary_run_ids` | array[id] | Yes | `[]` | Required when canary gate enabled. |
| `blocking_coverage_status` | enum | Yes | none | `pass` or `fail`. |
| `determinism_status` | enum | Yes | none | `pass` or `fail`. |
| `promote_allowed` | boolean | Yes | false | True only when all mandatory scenarios pass. |
| `checksum` | sha256_hex | Yes | computed | Canonical activation output. |

### 7.6 `CandidateGenerationProfile`

Purpose: defines blocking keys, candidate ordering, source scopes, allowed heuristics, prohibited selectors, deterministic pair ordering, and learned-candidate boundaries.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `candidate_generation_profile_id` | id | Yes | none | Namespace `candidate_generation_profile`. |
| `profile_version` | string | Yes | none | Immutable after activation. |
| `blocking_key_rows` | array[row] | Yes | none | Must list key type, evidence classes, canonicalization, source scopes, and max block size. |
| `allowed_heuristics` | array[enum] | Yes | `[]` | `exact_strong_id`, `same_source_asset`, `temporal_name_pair`, `learned_blocking_hint`, etc. |
| `prohibited_selectors` | array[enum] | Yes | weak-only set | Must include IP-only, DNS-only, PTR-only, flow-only, semantic-overlay-only for auto-merge. |
| `candidate_pair_order` | enum | Yes | `lexical_pair_key` | Pair key is sorted source asset IDs then identifier IDs. |
| `max_candidates_per_asset` | integer | Yes | 100 | Minimum 1, maximum profile-defined; overflow routes review/no-decision. |
| `learned_artifact_policy` | enum | Yes | `candidate_hint_only` | Learned output cannot be final merge authority. |

### 7.7 `ResolverShadowRun`

Purpose: candidate resolver profiles run against production-like inputs without mutating canonical identity, gold facts, graph deltas, watermarks, production health, or production version manifests.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `resolver_shadow_run_id` | id | Yes | none | Namespace `resolver_shadow_run`. |
| `resolver_profile_id` | id | Yes | none | Profile under test. |
| `input_manifest_id` | id | Yes | none | Production-like manifest or validation manifest. |
| `output_identity_decision_set_checksum` | sha256_hex | Yes | none | Hash of shadow decisions. |
| `mutation_allowed` | boolean | Yes | false | Must be false. |
| `comparison_baseline_id` | id or null | Yes | null | Required for regression comparison. |
| `difference_summary` | object | Yes | `{}` | Must be deterministic. |

### 7.8 `ResolverExplanation`

Purpose: structured explanation fields are normative. Optional human text is non-normative.

| Field | Type | Required | Default | Validation |
| --- | --- | ---: | --- | --- |
| `resolver_explanation_id` | id | Yes | none | Namespace `resolver_explanation`. |
| `decision_id` | id | Yes | none | Must reference `IdentityDecision`. |
| `primary_evidence_class` | enum or null | Yes | null | Null only for `no_decision`. |
| `supporting_evidence_classes` | array[enum] | Yes | `[]` | Sorted lexical. |
| `negative_evidence_classes` | array[enum] | Yes | `[]` | Sorted lexical. |
| `blocked_by_negative_evidence` | boolean | Yes | false | True when any hard blocker fired. |
| `confidence_algorithm` | enum | Yes | none | Must match resolver profile. |
| `confidence_inputs_checksum` | sha256_hex | Yes | none | Canonical evidence/confidence input hash. |
| `confidence_value` | decimal_0_1 | Yes | none | Must equal decision confidence. |
| `confidence_band` | enum | Yes | none | Must equal decision band. |
| `auto_merge_eligible` | boolean | Yes | false | False when review required or blocker fired. |
| `decision_matrix_row_id` | string | Yes | none | Selected row ID or `none`. |
| `review_required` | boolean | Yes | false | True when routed. |
| `split_policy_row_id` | string or null | Yes | null | Required for split/correction outputs. |
| `activation_profile_version` | string | Yes | none | Resolver profile version. |
| `evidence_refs` | array[EvidenceRef] | Yes | none | Must include all normative evidence refs. |
| `human_explanation` | string | No | empty | Non-normative, max 2000 chars. |

### 7.9 Error codes

| Error code | Required behavior |
| --- | --- |
| `RESOLVER_PROFILE_NOT_ACTIVE` | Fail before materializing production identity decisions when no active covering profile exists. |
| `RESOLVER_SCOPE_UNCOVERED` | Fail or route all candidates to `no_decision` when source scopes are outside profile coverage. |
| `EVIDENCE_CLASS_UNREGISTERED` | Reject evidence item materialization. |
| `IDENTIFIER_CANONICALIZATION_ERROR` | Reject malformed identifier unless mapping declares malformed evidence. |
| `NEGATIVE_EVIDENCE_BLOCKER_TRIGGERED` | Block auto-merge and emit blocker explanation. |
| `LIFECYCLE_BOUNDARY_CONFLICT` | Block auto-merge and route per boundary row. |
| `DECISION_MATRIX_NO_MATCH` | Emit `no_decision` with explanation. |
| `REVIEW_AUTHORITY_INSUFFICIENT` | Reject review terminal action. |
| `REVIEW_STATE_TRANSITION_ERROR` | Reject illegal review transition without mutating identity. |
| `SOURCE_NATIVE_MERGE_AUTHORITY_ERROR` | Reject profile attempting to use source-native merge history as merge authority. |
| `RESOLVER_REPLAY_MISMATCH` | Reject production replay when decisions or explanations differ for same manifest/profile. |
| `GRAPH_CORRECTION_HANDOFF_MISSING` | Reject split decision that lacks required graph correction handoff. |

## 8. Default host resolver decision matrix

### 8.1 Resolver decision matrix

| Evidence condition | Required decision | Confidence band | Hard blockers | Review routing | Explanation requirement | Rationale | Source basis |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Same durable source-native asset ID in same source scope | `merge` or update existing canonical link | `certain` | Source scope conflict, source rekey boundary | None unless prior conflict exists | Primary source-native ID, scope, profile row | Strong source-local identity | PRD, ServiceNow, cloud/endpoint docs.[^3][^5][^9][^10][^11][^12] |
| Same agent UUID or node key | `merge` if lifecycle continuity proven; otherwise `candidate` | `very_high` with continuity; `medium` without | Agent reinstall, duplicate active key, clone marker | Review if no continuity | Agent evidence and boundary state | Agent IDs are enrollment-scoped | osquery/Fleet docs.[^13] |
| Same directory device ID plus endpoint evidence | `merge` | `very_high` | Directory ID conflict, reenrollment boundary | None unless conflicting endpoint IDs | Directory ID plus endpoint evidence refs | Strong tenant-scoped ID | Microsoft docs.[^12] |
| Same Kubernetes UID in same cluster/resource type | `merge` for object occurrence only | `very_high` | Cluster/resource type mismatch | None | UID, cluster, resource type | UID distinguishes object occurrence | Kubernetes docs.[^8] |
| Same cloud instance/resource ID in same provider scope/resource type | `merge` for source asset | `very_high` | Provider/account/project/region/type mismatch; delete/recreate boundary | Review on generation ambiguity | Provider ID and scope refs | Strong source asset ID | AWS/Azure/GCP docs.[^9][^10][^11] |
| FQDN + MAC + overlapping valid time | `candidate` or `merge` only if profile permits | `high` | Incompatible durable IDs, expired DNS, MAC conflict | Review default; auto only with active row | FQDN, MAC, time interval | Stronger than name alone but still weak/correlated | PRD; DNS and scanner docs.[^3][^16][^14] |
| Hostname + domain + overlapping observation | `candidate` | `medium` | Incompatible durable identifiers, hostname reuse | Queue candidate review | Hostname/domain/time | Names are mutable/reused | PRD.[^3] |
| DNS name + current IP + endpoint observation | `candidate` | `medium_low` | TTL expired, IP reassigned, incompatible durable IDs | Queue candidate review | DNS TTL, IP interval, endpoint evidence | Temporal correlated weak evidence | RFC DNS/DHCP.[^16] |
| MAC-only match | `candidate` | `medium` | Multi-NIC ambiguity, active interface conflict | Review | MAC evidence and source quality | Interface not host | Tenable/Rapid7.[^14][^15] |
| IP-only match | `no_decision` | `none` | Not applicable | None unless review requested | IP-only rejection reason | Reassignable address | PRD/RFC DHCP.[^3][^16] |
| Hostname-only match | `no_decision` | `none` | Not applicable | None | Hostname-only rejection | Mutable/reused | PRD.[^3] |
| DNS-only or PTR-only match | `no_decision` | `none` | TTL expired if dependent evidence attempted | None | DNS-only rejection | Cached/TTL/reverse DNS ambiguity | PRD/RFC DNS.[^3][^16] |
| Source-native merge history | `candidate` or `conflict` | `source_hint` | Contradicts stronger evidence | Review when contradiction exists | Merge-history refs and policy | Source’s own model only | Defender/Rapid7/Tenable docs.[^12][^14][^15] |
| Hardware UUID duplicate across active cloned VMs | `conflict` | `blocked` | Hardware clone suspicion | Confirm conflict review | Duplicate UUID refs | Clone/golden image risk | Endpoint/scanner docs.[^12][^14][^15] |
| Reimage boundary | `candidate` or `split` if prior merge crosses boundary | `blocked` | Boundary evidence | Split review | Boundary evidence refs | Generation changed | PRD negative evidence.[^3] |
| Agent reinstall with evidence | `candidate` or continuity merge by row | `medium_high` | Conflicting durable IDs | Review default | Reinstall evidence, old/new agent IDs | Reinstall changes agent identity | osquery/Fleet.[^13] |
| Agent reinstall without evidence | `no_decision` | `none` | Missing continuity | None or review if high value asset | Missing continuity explanation | Unsafe continuity assumption | osquery/Fleet.[^13] |
| VDI pool reuse | `candidate` or `no_decision` | `low` | VDI/golden image marker | Review only when strong business key | VDI marker | Pool reuse undermines continuity | Rapid7 VDI docs.[^15] |
| Ephemeral cloud workload | `no_decision` for durable host merge; source asset only | `low` | Ephemeral marker | None by default | Ephemeral marker | Short-lived workload | Cloud/K8s docs.[^8][^11] |
| Scanner correlation conflict | `conflict` for scanner source asset, no canonical merge | `blocked` | Higher-priority ID conflict, multi-NIC ambiguity | Review if affects vulnerability coverage | Scanner conflict refs | Scanner algorithm not canonical | Tenable/Rapid7.[^14][^15] |
| Prior merge contradicted by stronger evidence | `split` | `blocked` | Strong conflict evidence | Split review unless profile allows automatic split | Prior decision, new evidence, graph handoff | Correct known-time intervals | PRD split rule.[^3] |

### 8.2 Negative evidence hard-blocker table

| Blocker code | Negative evidence condition | Evidence class | Default effect | Required output | Review behavior | Split behavior | Source basis | Required validation scenario |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `DURABLE_SOURCE_ID_CONFLICT` | Different durable source-native IDs in same source scope for same candidate | Source-native durable asset ID | Block auto-merge | `conflict` or `candidate` | Queue conflict review | Split if prior merge crossed conflict | PRD/ServiceNow/Tenable | Conflicting source IDs. |
| `AGENT_ID_CONFLICT` | Two active assets present different agent IDs without reinstall boundary | Agent UUID/node key | Block auto-merge | `conflict` | Review | Split if prior merge relied on old agent | osquery/Fleet | Agent conflict. |
| `DIRECTORY_ID_CONFLICT` | Different directory device IDs in same valid interval | Directory device ID | Block auto-merge | `conflict` | Review | Split | PRD/Microsoft | Directory ID conflict. |
| `HARDWARE_UUID_CLONE_SUSPECTED` | Same hardware UUID across active cloned VMs or source reports clone/golden-image | Hardware UUID | Block auto-merge | `conflict` | Confirm clone conflict | Split if merged | Endpoint/scanner | Cloned UUID. |
| `SERIAL_CONFLICT` | Different serial numbers in same valid interval with authoritative source | Serial number | Block auto-merge | `conflict` | Review | Split if merged | PRD | Serial conflict. |
| `MAC_ACTIVE_INTERFACE_CONFLICT` | Same MAC or conflicting MAC across active interfaces when source asserts one interface | MAC/interface evidence | Block auto-merge | `candidate`/`conflict` | Review multi-NIC | Split only if prior merge depended on MAC | Tenable/Rapid7 | Multi-NIC MAC conflict. |
| `HOSTNAME_INCOMPATIBLE_DURABLE_IDS` | Same hostname with incompatible durable identifiers | Hostname + durable IDs | Block auto-merge | `candidate` or `conflict` | Review | Split if prior merge by hostname | PRD | Hostname reuse. |
| `IP_LEASE_INTERVAL_CONFLICT` | Same IP assigned to incompatible assets inside lease/association interval | DHCP/IP assignment | Block IP-dependent merge | `no_decision` or `conflict` | Review if coverage impacted | Split only if prior merge used IP | RFC DHCP/AWS | DHCP reassignment conflict. |
| `DHCP_LEASE_EXPIRED` | DHCP lease expired before dependent observation | DHCP lease | Block dependent evidence | `no_decision` | None | None | RFC DHCP | Expired lease. |
| `DNS_TTL_EXPIRED` | DNS TTL expired before dependent observation | DNS record | Block dependent evidence | `no_decision` | None | None | RFC DNS | Expired TTL. |
| `TRANSIENT_EPHEMERAL_SOURCE` | Source reports transient/ephemeral workload/session | Lifecycle marker | Block durable host auto-merge | `source_asset` only or `candidate` | Review only if profile permits | Split if durable merge exists | K8s/cloud/Rapid7 VDI | Ephemeral workload. |
| `SCANNER_MULTI_NIC_AMBIGUITY` | Scanner cannot associate vulnerability/interface uniquely or scanner reports multi-NIC caveat | Scanner/interface evidence | Block scanner-only merge | `candidate`/`conflict` | Review coverage | No split unless merge relied on scanner | Rapid7/Tenable | Multi-NIC scanner fixture. |
| `SCANNER_ALGORITHM_CHANGE` | Scanner correlation algorithm/version changed and affects IDs | Scanner source evidence | Block auto-merge until activation validation | `candidate` | Review profile change | Split if contradicted | Rapid7/Tenable | Algorithm change. |
| `SOURCE_PERMISSION_INCOMPLETE` | Source lacks permission to see required identity scope | Source completeness/permission | Block auto-merge and absence | `no_decision` | Operator remediation | None | ServiceNow/PRD completeness | Permission incomplete. |
| `SOURCE_SCOPE_INCOMPLETE` | Source scope does not cover tenant/account/project/site | Source scope | Block auto-merge | `no_decision` | Operator remediation | None | PRD source scopes | Scope incomplete. |
| `SOURCE_NATIVE_MERGE_CONTRADICTION` | Source merge history contradicts stronger evidence | Merge-history row | Block auto-merge | `conflict` | Review | Split if needed | Defender/Rapid7 | Merge history contradiction. |
| `PRIOR_DECISION_EVIDENCE_UNREPRODUCIBLE` | Prior resolver decision lacks reproducible evidence refs or manifest | Version/evidence | Block reliance on prior decision | `no_decision` or review | Audit review | Split requires governance | PRD VersionManifest | Missing evidence refs. |
| `SEMANTIC_OVERLAY_ONLY` | Equality exists only through semantic overlay alias/type | Semantic overlay | Block auto-merge | `no_decision` | None | None | PRD/Taxi | Overlay-only equality. |

### 8.3 Lifecycle and generation boundary table

| Boundary type | Meaning | Required evidence | Default resolver effect | Auto-merge eligibility | Review routing | Split behavior | Validation scenarios |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `reimage` | Same physical/virtual device may have new OS/agent/generation. | Source lifecycle event, install timestamp reset, agent reinstall, OS image marker. | Block auto-merge across boundary unless continuity row permits. | No by default. | Split/candidate review. | Split prior merge if known-time evidence invalidates continuity. | Reimage with same hostname; reimage with same serial. |
| `agent_reinstall` | Agent identity regenerated. | Old/new agent IDs, install time, host continuity evidence. | Candidate unless continuity proven. | Conditional. | Review default. | Split if old/new agent IDs were merged without continuity. | With evidence, without evidence. |
| `golden_image_clone` | Multiple assets share cloned hardware UUID/agent/hostname. | Duplicate hardware UUID, image marker, VDI/source marker. | Hard blocker. | Never until unique evidence supplied. | Conflict review. | Split active cloned assets. | Cloned hardware UUID. |
| `vdi_pool_reuse` | Non-persistent VDI sessions reset/recreate. | VDI marker, R7CorrelationIdentifier, pool ID, session timestamps. | Candidate/no-decision. | No by default. | Review only for durable pool entity. | Split if per-session assets merged incorrectly. | VDI pool reuse. |
| `ephemeral_cloud_workload` | Short-lived cloud/serverless/container workload. | Cloud lifecycle, K8s UID, autoscaling/serverless marker. | Source asset only, no durable host merge. | No by default. | None unless policy. | Expire prior workload link if reused. | Ephemeral workload. |
| `hostname_reuse` | Hostname reused by different assets/generations. | Same hostname with conflicting durable IDs or non-overlap delete/create events. | Block hostname merge. | No. | Review if candidate. | Split prior hostname-based merge. | Hostname reuse. |
| `ip_reuse` | IP reassigned to another asset. | DHCP lease, cloud IP assignment, EIP association, NetBox/observed conflict. | Block IP-dependent merge. | No. | Review if coverage impacted. | Split IP-based merge. | DHCP reassignment. |
| `source_asset_rekey` | Source changes asset ID for same or different asset. | Source rekey event, merge history, old/new ID mapping. | Review required. | No by default. | Conflict/rekey review. | Split or continuity decision. | Source asset rekey. |
| `delete_recreate` | Resource path/name reused after deletion. | Source delete/create events, create time, provider generation ID. | New generation; block continuity unless strong evidence. | Conditional. | Review default. | Split across recreate. | Delete/recreate same name/path. |
| `directory_reenrollment` | Directory/MDM device object recreated. | Old/new directory IDs, enrollment event. | Candidate unless continuity evidence. | Conditional. | Review. | Split if prior merge invalid. | Directory reenrollment. |
| `kubernetes_object_recreate` | Same kind/name recreated with new UID. | Kubernetes UID/name/delete/create. | New object generation. | No name-only merge. | None/review. | Split old object occurrence. | Same name new UID. |
| `cloud_resource_recreate` | Same cloud name/path recreated or path changed. | Cloud delete/create, provider ID, creation timestamp. | New generation by default. | Conditional with vmId/instance ID continuity only. | Review. | Split if name/path merge invalid. | Cloud resource recreate. |
| `scanner_correlation_change` | Scanner algorithm/source grouping changes asset records. | Scanner version/algorithm setting, merge/asset history. | Block scanner-only auto-merge. | No. | Review scanner coverage. | Split if prior scanner merge contradicted. | Rapid7/Tenable algorithm change. |

## 9. Deterministic resolver algorithm

```text
ResolveIdentity(source_assets, identifiers, evidence, resolver_profile, as_of_valid_time, as_of_known_time)
  -> IdentityDecision[]

1. Validate profile.
   1.1 Require resolver_profile.profile_status = active for production.
   1.2 Require entity_type coverage for every source asset and identifier.
   1.3 Require resolver_profile.checksum present in VersionManifest.
   1.4 If no active covering profile exists, return RESOLVER_PROFILE_NOT_ACTIVE before output.

2. Validate source scopes.
   2.1 Canonicalize every source scope object by sorted keys.
   2.2 Require each source scope to match exactly one source_scope_coverage_row.
   2.3 If any scope is uncovered, emit no production decisions and return RESOLVER_SCOPE_UNCOVERED.

3. Canonicalize identifiers.
   3.1 For each identifier, apply identifier_canonicalization_registry rule.
   3.2 Normalize strings to NFC, IPs to canonical IP form, FQDNs to lowercase IDNA no trailing dot, MACs to lowercase colon form.
   3.3 Reject malformed identifiers with IDENTIFIER_CANONICALIZATION_ERROR unless the evidence class permits malformed evidence.
   3.4 Sort identifiers by (identifier_type, canonical_value, identifier_id).

4. Materialize IdentityEvidenceItem rows.
   4.1 For each source observation, map to exactly one evidence_class.
   4.2 Assign evidence_role from evidence class registry and source quality.
   4.3 Require at least one EvidenceRef for every item.
   4.4 Sort items by (source_scope_hash, source_asset_id, identifier_id, evidence_class, valid_from, valid_to, evidence_item_id).

5. Materialize AssetGenerationBoundary rows.
   5.1 Detect boundary evidence from lifecycle observations, source delete/create events, merge history, reinstall records, clone markers, VDI markers, scanner algorithm changes, DHCP/IP reuse, DNS TTL expiry, and source rekey events.
   5.2 Sort boundaries by (boundary_type, source_scope_hash, valid_from, known_from, boundary_id).

6. Generate candidate pairs.
   6.1 Load CandidateGenerationProfile from resolver_profile.
   6.2 Build blocking keys only from permitted evidence classes.
   6.3 Prohibited selectors may generate audit diagnostics but not candidate merge authority.
   6.4 Learned/probabilistic candidate generation may add `candidate_hint` only.
   6.5 Enforce max_candidates_per_asset; overflow emits no-decision or review per profile.

7. Order candidate pairs.
   7.1 Candidate pair key = sorted tuple of source_asset_id(s) and identifier_id(s).
   7.2 Sort by (candidate_pair_key, strongest_evidence_class_rank descending, earliest valid_from, lexical evidence_item_id list).
   7.3 Ties are resolved lexically by canonical pair key.

8. Evaluate negative-evidence blockers.
   8.1 For each candidate in order, evaluate blocker rows in ascending blocker_priority then blocker_code lexical order.
   8.2 A hard blocker sets blocked_by_negative_evidence = true and prevents auto-merge.
   8.3 All fired blockers are recorded in ResolverExplanation.

9. Evaluate lifecycle boundaries.
   9.1 Evaluate boundary rows in ascending boundary_priority then boundary_type lexical order.
   9.2 Boundary effects override confidence but do not erase evidence.
   9.3 Boundary requiring split routes to split policy before graph correction handoff.

10. Compute confidence.
   10.1 If blocked, confidence band = blocked and auto_merge_eligible = false.
   10.2 If confidence_algorithm = deterministic_rule_band, select confidence value from matching decision row.
   10.3 If weighted_score or external_model_wrapped, compute or read score only after blockers; map through deterministic band rows.
   10.4 Source-native confidence and reviewer confidence are inputs only; Cadastre decision authority remains ResolverProfile.

11. Select decision matrix row.
   11.1 Match rows by evidence condition, entity_type, source_scope, blocker state, lifecycle state, and confidence band.
   11.2 If multiple rows match, choose the lowest row_priority; ties fail profile validation.
   11.3 If no row matches, emit `no_decision` with DECISION_MATRIX_NO_MATCH.

12. Route review.
   12.1 If selected row requires review, create or update IdentityReviewCase.
   12.2 Review case state is sorted and deterministic; no identity mutation occurs until terminal state emits IdentityDecision.
   12.3 Expired review emits closed/no-action or no-decision per review policy.

13. Apply split correction.
   13.1 If selected decision is split, close known_to on prior IdentityDecision.
   13.2 Create new split IdentityDecision.
   13.3 Emit affected GoldFact correction refs and GraphCorrectionHandoff.
   13.4 Reject split with GRAPH_CORRECTION_HANDOFF_MISSING if handoff is absent.

14. Persist explanations.
   14.1 Emit exactly one ResolverExplanation per IdentityDecision.
   14.2 Explanation fields are normative except human_explanation.

15. Include VersionManifest refs.
   15.1 Include resolver profile, evidence registry, canonicalization registry, candidate generation profile, training/model artifacts, review policy, split policy, activation report, source scopes, evidence refs, and input dataset refs.
   15.2 Replay must reject mismatches before output.

16. Hand off graph correction.
   16.1 Identity decisions do not mutate graph directly.
   16.2 Graph projection consumes identity/gold correction records and emits graph deltas under GraphProjectionProfile and GraphEdgeSemantics.
```

No-op behavior: an input evidence item that cannot be assigned to a registered evidence class must not be silently ignored. It must produce `EVIDENCE_CLASS_UNREGISTERED` or be explicitly materialized as `lineage_only` by a profile row. A candidate pair with no matching decision matrix row must emit `no_decision`, not disappear.

## 10. Review, activation, and replay model

### Review model

Review cases are deterministic state machines. A review UI may display, filter, assign, and annotate, but identity mutation occurs only through emitted terminal `IdentityDecision` records. Reviewer authority is checked against `required_reviewer_authority` at terminal transition time. Manual override requires `manual_override_reason_code`, evidence snapshot checksum, reviewer authority, and audit refs.

Terminal review outputs:

| Terminal state | Required output |
| --- | --- |
| `approved_merge` | `IdentityDecision.merge`, `ResolverExplanation`, VersionManifest refs. |
| `rejected_match` | `IdentityDecision.reject`, explanation. |
| `confirmed_conflict` | `IdentityDecision.conflict`, explanation, hard blockers. |
| `approved_split` | `IdentityDecision.split`, corrected known-time intervals, graph correction handoff. |
| `expired` | `closed_no_action` or `no_decision` per profile; no merge. |
| `closed_no_action` | No identity mutation. |

### Activation gates

A `ResolverProfile` cannot become active unless `ResolverActivationReport.promote_allowed = true`, all required scenarios pass, shadow outputs are deterministic, and canary outputs, if enabled, are non-authoritative. Production activation requires the profile checksum, evidence registry, candidate generation profile, review policy, split policy, and activation report IDs to appear in `VersionManifest` before output.

Required activation scenarios appear in Section 11. The expected output must be binary: exact `decision_type`, `confidence_band`, blocker codes, review routing, split handoff presence, and graph correction outcome.

### Shadow and canary behavior

`ResolverShadowRun` may read production-like inputs and emit shadow decision sets, but must not mutate canonical identity, unresolved targets, gold facts, graph deltas, graph serving state, CIM, health-critical production state, production watermarks, or production `VersionManifest`. Canary output is also non-authoritative unless a future deployment profile permits a bounded write class. MVP canary identity output remains shadow-only.

### Replay behavior

Replay must be byte-equivalent for all normative outputs when given identical input snapshots, resolver profile, evidence registry, canonicalization registry, source scopes, lifecycle evidence, completeness evidence, training artifacts, review terminal decisions, and version manifest. Replay must reject output with `RESOLVER_REPLAY_MISMATCH` when any `IdentityDecision`, `ResolverExplanation`, review terminal output, or graph correction handoff differs.

## 11. Validation scenarios and acceptance criteria

### 11.1 Required resolver activation scenarios

| Scenario ID | Given | Expected output |
| --- | --- | --- |
| `RID-SAME-SOURCE-ID` | Same durable source-native asset ID in same source scope, no blockers. | `merge`, `certain`, no review. |
| `RID-IP-ONLY` | Shared IP only. | `no_decision`, no review, blocker absent, explanation says IP-only not sufficient. |
| `RID-HOSTNAME-ONLY` | Shared hostname only. | `no_decision`. |
| `RID-DNS-ONLY` | Shared DNS A/PTR only. | `no_decision`. |
| `RID-PTR-ONLY` | PTR-only match. | `no_decision`. |
| `RID-FQDN-MAC-OVERLAP` | FQDN + MAC + overlapping valid time, no durable conflicts. | `candidate` by default, or `merge` only if explicit row permits. |
| `RID-CLONED-HWUUID` | Same hardware UUID across active cloned VMs. | `conflict`, blocker `HARDWARE_UUID_CLONE_SUSPECTED`. |
| `RID-AGENT-REINSTALL-WITH-EVIDENCE` | Old/new agent IDs with continuity evidence. | `candidate` or profile-permitted `merge`; explanation names boundary. |
| `RID-AGENT-REINSTALL-WITHOUT-EVIDENCE` | Old/new agent IDs without continuity. | `no_decision` or `candidate`, no auto-merge. |
| `RID-DHCP-REASSIGNMENT` | Same IP assigned to different assets in non-overlapping leases. | Separate address facts, no host merge. |
| `RID-DNS-TTL-EXPIRY` | DNS evidence after TTL expired. | Dependent identity evidence rejected; `no_decision`. |
| `RID-DEFENDER-MERGE-HISTORY` | Source-native merged device IDs only. | `candidate` or `lineage_only`, no merge authority. |
| `RID-TENABLE-HIGHER-PRIORITY-CONFLICT` | Lower-priority MAC/FQDN match with higher-priority UUID conflict. | `conflict`, hard blocker. |
| `RID-RAPID7-MULTI-NIC` | Same asset/vulnerability ambiguity across NICs. | No canonical host merge; scanner conflict/candidate. |
| `RID-PRIOR-MERGE-DURABLE-CONFLICT` | Prior merge contradicted by durable ID conflict. | `split`, prior known_to closed, graph correction handoff present. |
| `RID-SOURCE-SCOPE-INCOMPLETE` | Source scope missing tenant/account/project/site. | `no_decision` or profile error `RESOLVER_SCOPE_UNCOVERED`. |
| `RID-SOURCE-PERMISSION-INCOMPLETE` | Source cannot see required identity scope. | `no_decision`, blocker `SOURCE_PERMISSION_INCOMPLETE`. |
| `RID-SEMANTIC-OVERLAY-ONLY` | Equality only through semantic alias/type equality. | `no_decision`, blocker `SEMANTIC_OVERLAY_ONLY` or zero evidence. |

### 11.2 Binary acceptance criteria by contract

| AC ID | Contract | Criterion |
| --- | --- | --- |
| `AC-RP-001` | `ResolverProfile` | Production resolution fails with `RESOLVER_PROFILE_NOT_ACTIVE` when no active profile covers entity type and source scopes. |
| `AC-RP-002` | `ResolverProfile` | Every decision matrix row has unique row ID and priority; ties fail profile validation. |
| `AC-RP-003` | `ResolverProfile` | Hard blockers are evaluated before confidence; a fired hard blocker prevents auto-merge even when score exceeds threshold. |
| `AC-RP-004` | `ResolverProfile` | Profile checksum changes when any row affecting output changes. |
| `AC-IEI-001` | `IdentityEvidenceItem` | Every item has exactly one evidence class, role, source scope, evidence ref set, durability class, and reuse risk. |
| `AC-IEI-002` | `IdentityEvidenceItem` | IP-only, DNS-only, PTR-only, flow-only, semantic-overlay-only rows contribute zero auto-merge authority. |
| `AC-AGB-001` | `AssetGenerationBoundary` | Reimage, clone, VDI, ephemeral workload, hostname reuse, IP reuse, agent reinstall, source rekey, delete/recreate, directory reenrollment, Kubernetes recreate, cloud recreate, and scanner correlation change are all represented with default effects. |
| `AC-IRC-001` | `IdentityReviewCase` | Illegal review transition fails with `REVIEW_STATE_TRANSITION_ERROR` and does not mutate identity. |
| `AC-IRC-002` | `IdentityReviewCase` | Terminal merge, reject, conflict, and split states emit the matching `IdentityDecision` or no identity mutation occurs. |
| `AC-RAR-001` | `ResolverActivationReport` | A profile cannot become active unless every required activation scenario has expected output and `promote_allowed = true`. |
| `AC-CGP-001` | `CandidateGenerationProfile` | Candidate pair ordering is deterministic and independent of input order. |
| `AC-CGP-002` | `CandidateGenerationProfile` | Learned or probabilistic candidate generation can produce only `candidate_hint` unless a future profile declares stricter authority. |
| `AC-RSR-001` | `ResolverShadowRun` | Shadow run does not mutate production raw, silver, identity, gold, graph, health, watermark, or version manifest state. |
| `AC-REX-001` | `ResolverExplanation` | Every `IdentityDecision` has exactly one structured explanation with confidence input checksum and decision matrix row ID. |
| `AC-SPLIT-001` | Split policy | Prior merge decision has `known_to` closed; new split decision exists; affected gold facts get corrected intervals; graph correction handoff exists. |
| `AC-REPLAY-001` | Resolver replay | Same inputs and same profile produce byte-equivalent `IdentityDecision[]` and `ResolverExplanation[]`. |
| `AC-GRAPH-CORR-001` | Graph correction handoff | Identity resolver emits no direct graph mutation; graph correction is handed to projection through persisted correction records. |

## 12. Concepts not to transfer

| Concept | Rejection |
| --- | --- |
| Using source-native asset IDs as global canonical identity | Must not transfer. Source IDs are scoped and source-controlled. |
| Using graph keys as canonical identity | Must not transfer. Graph keys are projection or source graph keys. |
| Using mapped target references to create or merge canonical entities | Must not transfer. Target references remain unresolved until resolver decision. |
| Hostname-only, IP-only, DNS-only, PTR-only, or flow-only auto-merge | Must not transfer. These are weak temporal or selector evidence. |
| Source-native merge history as Cadastre merge authority | Must not transfer. Store as lineage or contradiction evidence. |
| Scanner correlation as canonical host identity | Must not transfer. Scanner algorithms optimize vulnerability reporting. |
| IPAM intended state as observed host identity | Must not transfer. Intended state is not observed state. |
| Telemetry resource identity as Cadastre canonical identity | Must not transfer. Telemetry identity is resource/observer scope. |
| Semantic overlay type equality or alias equality as identity evidence by itself | Must not transfer. Overlay equality classifies meaning only. |
| Probabilistic ER model output as direct production merge authority | Must not transfer. It must be wrapped by `ResolverProfile`. |
| Reviewer UI state without emitted `IdentityDecision` | Must not transfer. Hidden mutation breaks replay. |
| Graph/backend cleanup as source absence or split authority | Must not transfer. Cleanup is not completeness or correction evidence. |

## 13. Gaps, risks, stale assumptions, and unresolved questions

| Category | Gap or risk | Required authority or follow-up |
| --- | --- | --- |
| Product decision | Whether any FQDN+MAC+overlap condition may auto-merge in MVP. | Product/security architecture decision. Default remains review/candidate. |
| Product decision | Whether canonical hosts should represent VDI pool, VDI session, golden image, or each recreated desktop. | Endpoint/VDI domain authority. |
| Product decision | Which cloud serverless/container workloads are canonical hosts versus workload generations. | Cloud platform architecture. |
| Product decision | Whether Azure ARM path continuity or `vmId` continuity governs Azure VM generation in Cadastre. | Cloud platform and source adapter design. |
| Product decision | How long source-native merge history remains reviewable evidence. | Retention and audit governance. |
| Source evidence | Exact Rapid7 and Tenable correlation internals can vary by version and settings. | Scanner source-specific adapter profiles and fixtures. |
| Source evidence | GCP per-resource delete/recreate and numeric instance ID reuse guarantees were not fully established from official docs in this run. | GCP source adapter follow-up. |
| Source evidence | Hardware UUID and serial quality differs by vendor, virtualization platform, and clone practice. | Endpoint domain authority and validation corpus. |
| Review governance | Reviewer authority levels and override conditions are product decisions. | Product/security governance. |
| Replay risk | Review decisions require persistent evidence snapshot checksums. | PRD must add review evidence snapshot retention requirements. |
| Model risk | Probabilistic ER calibration on weak correlated identifiers can overstate confidence. | Golden identity corpus and shadow-run calibration. |
| Graph risk | Split correction can fail if projection profile cannot retract/expire affected edges. | Add graph correction handoff validation and projection scenario tests. |

## 14. Highest-value PRD improvement map

| Priority | Finding | Target PRD area | Proposed contract | Exact PRD improvement | Acceptance criterion |
| --- | --- | --- | --- | --- | --- |
| High | Resolver authority is distributed. | Identity Resolution Requirements, Data Model, VersionManifest, ValidationMatrix | `ResolverProfile` | Add first-class resolver authority contract and fail production resolution when absent. | `AC-RP-001`. |
| High | Identifier semantics are under-typed. | `Identifier`, `IdentityDecision`, Section 14 | `IdentityEvidenceItem`, `IdentifierEvidenceClass` | Replace raw positive/negative evidence JSON with typed evidence items. | `AC-IEI-001`, `AC-IEI-002`. |
| High | Candidate generation can diverge. | Section 14 | `CandidateGenerationProfile` | Define blocking keys, prohibited selectors, ordering, max candidates, learned candidate boundaries. | `AC-CGP-001`, `AC-CGP-002`. |
| High | Negative evidence needs exhaustive hard blockers. | Section 14.3 | `negative_evidence_blocker_rows` | Add blocker code table in Section 8.2. | `AC-RP-003`. |
| High | Lifecycle boundaries not first-class. | Section 14.3/14.4 | `AssetGenerationBoundary` | Add boundary table and materialization algorithm. | `AC-AGB-001`. |
| High | Review state can mutate hidden identity if not constrained. | Identity Conflict Review | `IdentityReviewCase` | Add closed state machine, reviewer authority, terminal outputs, expiration. | `AC-IRC-001`, `AC-IRC-002`. |
| High | Resolver activation scenarios absent. | ValidationMatrix | `ResolverActivationReport` | Add required scenario set and pass/fail outputs. | `AC-RAR-001`. |
| Medium | Source-native merge history needs policy. | `IdentityDecision` and evidence | `source_native_merge_history_policy` | Store merge history as lineage/contradiction/review hint only. | Activation scenarios `RID-DEFENDER-MERGE-HISTORY`. |
| Medium | Explanations are not normative enough. | `IdentityDecision` | `ResolverExplanation` | Require structured explanation fields and input checksums. | `AC-REX-001`. |
| Medium | Graph correction handoff is implied. | Split Behavior, Graph Projection | `GraphCorrectionHandoff` field in split policy | Tie split to gold correction and graph retraction/expiration outputs. | `AC-GRAPH-CORR-001`. |
| Medium | ER frameworks need deterministic wrapper. | Resolver governance | `ResolverTrainingArtifact`, `ResolverShadowRun` | Permit probabilistic outputs only as governed hints. | `AC-RSR-001`. |

## Sources

[^1]: `/mnt/data/PRD-Cadastre.md`, Product Summary and Document Status, lines 16-62. The PRD defines the temporal lakehouse architecture, graph as replaceable projection, OCSF-aligned silver, direct-to-graph rejection, and raw-first boundary.
[^2]: `/mnt/data/PRD-Cadastre.md`, normative contract table and scope rows, lines 64-110 and 178-194. The PRD defines SourceAuthorityProfile, TargetSelectorSafetyPolicy, GraphEdgeSemantics, ValidationMatrix, RecordedSourceFixture, and source/projection governance.
[^3]: `/mnt/data/PRD-Cadastre.md`, identity conflict review and identity model, lines 1038-1062, 1481-1588, and 6945-7005. The PRD defines visible identity states, CanonicalEntity, SourceAsset, Identifier, IdentityDecision, default evidence strength, negative evidence, and split behavior.
[^4]: `/mnt/data/nlspec-spec.md`, NLSpec structural requirements, lines 30-72 and 116-120. The NLSpec standard requires behavioral completeness, unambiguous interfaces, defaults and boundaries, mapping tables, binary acceptance criteria, and recreatability.
[^5]: ServiceNow official documentation inspected through web: CMDB IRE overview, identification rules, dependent relationship rules, reconciliation rules, data-source insert/update governance, and identification simulation. citeturn934047view0turn934047view1turn733688view0turn733688view1turn733688view2turn733688view3
[^6]: `/mnt/data/PRD-Cadastre.md`, ValidationMatrix and golden corpus requirements, lines 7760-7805.
[^7]: OpenTelemetry Entity Data Model and resource/entity semantic convention docs. citeturn934047view2turn934047view3
[^8]: Kubernetes object names/IDs, labels, annotations, and pod lifecycle docs. citeturn442556view0turn442556view1turn442556view2turn442556view3
[^9]: AWS EC2 official documentation for instance metadata, IP addressing, Elastic IP addresses, ENIs, and tags. citeturn254581view1turn848391view1turn848391view0turn848391view2turn254581view0
[^10]: Azure ARM, resource ID/scopes, resource naming, and VM ID/IMDS official docs. citeturn984238view0turn984238view1turn984238view2turn984238view3turn984238view4turn272477view0turn272477view1
[^11]: Google Cloud Asset Inventory and Compute Engine docs. citeturn272477view2turn272477view3turn272477view4
[^12]: Microsoft Intune managedDevice, Defender XDR DeviceInfo, and Microsoft Graph device official docs. citeturn191238view0turn191238view1turn191238view2turn159361view4
[^13]: osquery deployment remote enrollment and Fleet host enrollment docs. citeturn191238view3turn159361view3
[^14]: Tenable Security Center Asset Tracking official docs. citeturn562369view0
[^15]: Rapid7 InsightVM asset correlation, multi-NIC, VDI, and cross-site linking docs. citeturn403100view0turn403100view1turn403100view2turn403100view3turn403100view4
[^16]: RFC 2131 DHCP, RFC 1035 DNS, Zeek docs, and NetBox documentation. citeturn923781view0turn923781view1turn532586search0turn532586search20
[^17]: Splink, Dedupe, py_entitymatching, and Zingg docs/repositories. citeturn743856view0turn743856view1turn743856view3turn386342search0turn386342search1turn743856view2turn743856view4
[^18]: `/mnt/data/RES-002-jupiterone.md`, executive summary and architecture contrast, lines 10-36, plus Starbase runtime orientation, lines 84-100 and 120-150. Prior research evidence, not governing authority.
[^19]: `/mnt/data/RES-001-cartography.md`, executive architectural map and purpose, lines 20-105. Prior research evidence, not governing authority.
[^20]: `/mnt/data/RES-008-identity-graph-attack-paths.md`, executive verdict, inspection limits, OpenGraph identity/endpoint matching, deletion hazards, and graph state table, lines 9-36, 42-70, 96-110, 120-154, and 206-230. Prior research evidence, not governing authority.
[^21]: `/mnt/data/RES-003-taxi-lang.md`, executive verdict and source inventory, lines 10-42. Prior research evidence, not governing authority.
