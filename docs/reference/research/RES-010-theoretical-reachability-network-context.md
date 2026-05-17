---
doc_id: RES-010
project: Cadastre
title: Theoretical Reachability and Network Context Research Report for Cadastre
status: research-report
inspection_date: 2026-05-16
related_docs:
  - RES-006
  - RES-008
---

## 1. Executive verdict

Cadastre must keep theoretical reachability out of MVP. The current PRD boundary is correct: `observed_connection` may exist as an MVP graph edge only when derived from `observed_network_flow_fact` and `FlowRoleEvidence`, while `has_theoretical_reachability` must remain rejected until a later accepted PRD or NLSpec defines the full input contract, completeness model, source authority model, solver capability contract, and result semantics.[^1][^2][^3]

The core finding is that no inspected external system provides a universal reachability oracle. Batfish provides offline, snapshot-based network configuration analysis and can produce example flows and traces under header and path constraints. AWS, Google Cloud, and Azure provider analyzers are useful scoped diagnostic tools, but each has explicit analysis-mode, resource, permission, unsupported-feature, and representative-path limits. Kubernetes and Cilium policy semantics are workload-policy contracts, not application-access guarantees. NIST Zero Trust Architecture makes network location insufficient for access. BloodHound/OpenGraph shows that graph traversability is a separate analysis property, not source truth or network reachability.[^11][^14][^17][^20][^25][^26][^31][^35]

A future Cadastre reachability phase may be safe only if it is analysis-first, evidence-scoped, and graph-neutral by default. The default `graph_effect` for all proposed reachability contracts must be `none`. GoldFact or graph-edge emission must require an explicit `ReachabilityClaimPolicy`, complete evidence references, source completeness decisions, source authority profiles, and a solver capability matrix that proves every relevant resource, policy, route, translation, workload, identity, and limitation is representable.

Recommended future architecture:

```text
NetworkContextSnapshot
  -> ReachabilitySourceDatasetProfile
  -> ReachabilitySourceCompletenessProfile
  -> ReachabilityModelCapabilityMatrix
  -> ReachabilityQuery
  -> EvaluateReachability(...)
  -> ReachabilityResult
  -> ReachabilityAnalysisArtifact
  -> optional future GoldFact/GraphEdge only through ReachabilityClaimPolicy
```

The PRD should be strengthened, but not expanded into MVP reachability. The highest-value PRD changes are future-domain contracts, result-state vocabulary, prohibited-claim rules, graph traversal eligibility, analyzer-boundary rules, and validation fixtures.

## 2. Source inventory and inspection limits

### 2.1 Evidence class definitions

| Evidence class | Meaning in this report |
| --- | --- |
| `confirmed source fact` | Directly supported by official documentation, repository documentation, source code, schema, standard, or uploaded Cadastre project file inspected for this report. |
| `source-grounded inference` | Inferred from one source family’s confirmed facts, documented architecture, or documented limitations. |
| `Cadastre applicability inference` | Design implication derived by comparing confirmed facts to the governing Cadastre PRD. |
| `speculative transfer` | Plausible Cadastre use requiring deeper implementation inspection, benchmark, or product/domain authority before PRD adoption. |
| `open question` | Requires further source inspection, runtime testing, source-specific data, or product governance. |

### 2.2 Source inventory and inspection limits

| Source family | Sources inspected | Version, release, or freshness | Inspection mode | Confirmed facts versus limits |
| --- | --- | --- | --- | --- |
| Batfish and pybatfish | Batfish and pybatfish official documentation | pybatfish docs identify version `0.36.0`; docs copyright through 2024 | Documentation inspected only; no clone, snapshot, question execution, or lab validation | Confirmed packet-forwarding, reachability, traceroute, ACL, bidirectional, multipath, and supported-feature documentation. Runtime behavior was not tested. |
| AWS VPC Reachability Analyzer and Network Access Analyzer | AWS EC2/VPC official docs | Public AWS docs inspected 2026-05-16 | Documentation inspected only; no AWS account/API execution | Confirmed configuration-analysis semantics, no-packet behavior, supported resources, path limits, representative findings, and unsupported resources. API permissions and behavior were not tested. |
| Google Cloud Connectivity Tests | Google Cloud Network Intelligence Center docs and official Google Cloud blog | Public docs inspected 2026-05-16 | Documentation inspected only; no GCP project/API execution | Confirmed configuration analysis, live data-plane analysis limitations, result states, load-balancer/backend limitations, and permission gaps. Runtime behavior was not tested. |
| Azure Network Watcher and Azure Virtual Network Manager | Microsoft Learn docs for IP Flow Verify, Next Hop, Effective Security Rules, Connection Troubleshoot, security admin rules | Public docs inspected 2026-05-16 | Documentation inspected only; no Azure tenant/API execution | Confirmed diagnostic decomposition, admin-rule precedence, permissions, protocol bounds, preview caveat for agentless connection troubleshoot, and eventual consistency. Runtime behavior was not tested. |
| Kubernetes NetworkPolicy | Kubernetes official docs | Public docs inspected 2026-05-16 | Documentation inspected only; no cluster execution | Confirmed ingress/egress isolation, additive rules, selector semantics, CNI dependency, IPBlock and rewrite caveats. CNI-specific behavior was not tested. |
| Cilium network policy | Cilium stable docs | Stable docs identified as `1.19.4` | Documentation inspected only; no cluster execution | Confirmed policy enforcement modes, default-deny behavior, deny precedence, L3/L4/L7, DNS/FQDN, and version caveats. Runtime behavior was not tested. |
| NIST SP 800-207 Zero Trust Architecture | NIST CSRC official page and publication record | SP 800-207, August 2020; stable standard | Official documentation inspected only | Confirmed no implicit trust from network location or asset ownership, subject/device authorization before session, and PDP/PEP concepts. Implementation profiles were not evaluated. |
| OpenConfig ACL and related YANG models | OpenConfig ACL YANG browser material | Public OpenConfig ACL schema inspected 2026-05-16 | Documentation/schema inspected only; no device execution | Confirmed ACL sets, sequence-ordered entries, match fields, actions, and operational counters. Vendor realization was not tested. |
| NetKAT and formal verification work | NetKAT official site, Google NetKAT repository, NetKAT publications list | Public site/repository inspected 2026-05-16; Google repo has no releases published | Documentation/repository page inspected only; no verifier execution | Confirmed formal packet-forwarding DSL, KAT foundation, reachability/isolation/equivalence verification, and implementation caveat. Practical Cadastre integration remains unproven. |
| BloodHound and OpenGraph | Official BloodHound/OpenGraph docs plus prior Cadastre RES-008 | BloodHound/OpenGraph docs inspected 2026-05-16; RES-008 inspected BloodHound v9.1.0 and SharpHound v2.12.0 | Docs and prior research inspected; no local BloodHound execution | Confirmed structured versus generic graph behavior, traversability, pathfinding eligibility, post-processed edge behavior, and graph-authority cautions. No runtime pathfinding was executed. |
| Source-adapter and ingestion systems | Prior Cadastre RES-006 | RES-006 inspected 2026-05-16 | Prior research used as evidence, not authority | Confirmed Cadastre-relevant raw-first adapter patterns, state/completeness receipts, and live-probe non-authority. Original systems were not re-executed here. |

### 2.3 General source limits

No external repository was cloned, built, tested, or executed for this report. No cloud-provider API was invoked. No provider analyzer result was generated in a live tenant. No Kubernetes, Cilium, Batfish, BloodHound, NetKAT, or OpenConfig device/runtime fixture was executed. Public documentation and prior uploaded Cadastre research reports are sufficient for PRD-level boundary recommendations, but not sufficient for production implementation compatibility claims.

## 3. Cadastre baseline and non-transferable MVP boundary

Cadastre’s governing baseline already states that the lakehouse is the system of record, graph output is a replaceable projection, `CadastreSilverObservation` is the authoritative silver contract, and source-adapter patterns must not change the raw-first lakehouse-authoritative boundary.[^4]

The PRD already requires `FlowRoleEvidence` for network-flow direction and states that graph direction must come only from Cadastre flow-role rules.[^1] It also rejects `has_theoretical_reachability` in the default MVP graph projection profile until a later PRD or NLSpec defines the complete theoretical reachability contract.[^1][^3]

The PRD’s non-scope language is explicit: the product must not treat observed network traffic as proof of complete theoretical reachability, must not treat absence of evidence as negative evidence unless a source contract defines absence as meaningful, and must not guarantee lateral movement pathing unless firewall policy, routing, NAT, identity privilege, and endpoint access context are present and modeled.[^2]

The graph edge baseline is also explicit. `GraphEdgeSemantics` must define one row per edge type, including source fact type, source predicate, direction rule, evidence requirements, allowed assertion states, temporal policy, confidence policy, non-implication rules, and no-op conditions. The MVP row for `observed_connection` says it does not imply theoretical reachability, and `has_theoretical_reachability` is not a default MVP edge.[^5]

| Boundary | Required Cadastre behavior |
| --- | --- |
| `observed_connection` | May be emitted only from `observed_network_flow_fact` with qualifying `FlowRoleEvidence`. It must not imply theoretical reachability.[^5] |
| `has_theoretical_reachability` | Must fail or no-op in MVP with `THEORETICAL_REACHABILITY_SCOPE_ERROR` until a future accepted PRD or NLSpec exists.[^3][^6] |
| Direction | Must come from Cadastre flow-role rules, not OCSF endpoint field order, provider field names, or graph source/destination convention.[^7] |
| Absence | Must require `SourceCompletenessReceipt`, active `SourceCompletenessProfile`, source authority, and where applicable `CoverageAssertion`; unsafe completeness states must block absence, retraction, cleanup, and watermark advancement.[^8] |
| Adapter output | Adapter stages may emit raw records, completeness receipts, and adapter errors only; they must not directly emit silver, gold, graph deltas, graph apply results, or reachability truth.[^9] |
| Provider analyzer output | Must be scoped evidence or analysis artifact only, never direct Cadastre gold truth. |
| Graph traversal | Must not imply source truth, effective privilege, complete network path, or application access. |

## 4. Required terminology

### 4.1 Observed traffic versus modeled reachability terminology

| Term | Required definition | Positive result means | Negative result means | Cadastre graph or fact implication |
| --- | --- | --- | --- | --- |
| `observed_connection` | A Cadastre fact or edge derived from observed network-flow evidence and `FlowRoleEvidence`. | Traffic matching the evidence was observed within the evidence’s valid/known interval. | No negative claim unless the source contract defines complete coverage and meaningful absence. | MVP edge allowed only under `GraphEdgeSemantics`; must not imply theoretical reachability.[^5] |
| `modeled_reachability` | A solver result over a declared network context snapshot, source completeness set, authority profile, and capability matrix. | The model found a scoped path satisfying the query and modeled constraints. | Only meaningful inside declared scope and only if all required evidence classes and model capabilities are complete. | Future analysis artifact by default; GoldFact/edge emission requires `ReachabilityClaimPolicy`. |
| `packet_reachability` | Whether a packet path exists for a protocol, address tuple, port tuple, and direction. | A modeled forward packet can traverse all represented forwarding, filtering, and translation components. | `not_reachable` only if route, policy, NAT, topology, and capability evidence are complete for the modeled packet domain. | Not service access; not identity access. |
| `session_reachability` | Whether forward and return path or stateful-return behavior are modeled for a session. | Forward path and modeled reverse/session state permit a response. | Negative only if both forward and return or stateful-return evidence are complete. | Stronger than packet reachability; still not application access. |
| `service_access` | Whether endpoint listener, service, proxy, load balancer, backend health, and host/workload policy permit application-level access. | The model includes network path plus service/listener/backend context. | Negative only when service and endpoint access evidence are complete. | Must not be inferred from firewall allow or route alone. |
| `identity_conditioned_service_access` | Whether subject, service account, group, role, device posture, PDP/PEP policy, and session context allow access. | Access is modeled under named identity and posture conditions. | Negative only when identity and access-policy sources are complete and authoritative. | Requires zero-trust conditions; network location alone is insufficient. |
| `unknown_partial` | Result when evidence is missing, stale, partial, permission-limited, or outside completeness scope. | No positive or negative reachability claim is safe. | Same. | GoldFact and graph edge emission prohibited. |
| `unsupported` | Result when solver cannot represent a relevant resource, route, policy, appliance, workload, identity condition, or translation behavior. | The query cannot be evaluated under the declared model. | Same. | GoldFact and graph edge emission prohibited. |
| `conditional` | Result when the claim is true only under explicitly named protocol, route, identity, posture, time, or policy conditions. | Reachability/access holds only under the returned conditions. | Outside those conditions, no claim is made unless separately evaluated. | Fact/edge emission prohibited by default unless a future policy encodes conditions in the fact. |

### 4.2 Packet/session/service/identity-conditioned access comparison

| Claim kind | Required evidence classes | Includes return path | Includes listener/backend | Includes identity/access policy | Default result when evidence missing | Positive claim may support | Negative claim may support |
| --- | --- | ---: | ---: | ---: | --- | --- | --- |
| `packet_reachability` | Endpoint identity, packet selector, valid/known time, topology, route, firewall/policy, NAT/translation when present, source completeness, authority, model capability | No | No | No | `unknown_partial` or `unsupported` | Analysis artifact; future fact only with policy | Only with complete scoped evidence and capability |
| `session_reachability` | All packet classes plus reverse path or stateful-return behavior | Yes | No | No | `unknown_partial` | Analysis artifact; future fact only with policy | Only with complete forward and return/session evidence |
| `service_access` | All session classes plus listener, load balancer, backend health, service, host firewall, workload policy | Yes | Yes | No | `unknown_partial` | Analysis artifact; future fact only with policy | Only with complete service and endpoint context |
| `identity_conditioned_service_access` | All service classes plus subject, service account, group, role, device posture, PDP/PEP, session authorization | Yes | Yes | Yes | `unknown_partial` or `conditional` | Analysis artifact; future fact only with policy and explicit conditions | Only with complete identity and policy context |

## 5. Independent source analyses

### 5.1 Batfish

#### 5.1.1 Source inventory and inspection limits

Batfish and pybatfish were inspected through official documentation only. pybatfish docs identify version `0.36.0`. No Batfish snapshot was built, no vendor configuration was parsed locally, and no question was executed.

#### 5.1.2 Problem solved and non-purpose

Batfish solves offline, snapshot-based network configuration analysis. It models traffic, forwarding, filters, ACLs, routes, and device/network behavior from a parsed snapshot. It does not observe live traffic, perform live probes, prove enterprise-wide access, or model identity-conditioned application access by default.[^11]

#### 5.1.3 Architecture and operational model

Batfish uses a network snapshot and query-driven questions. Packet-forwarding questions answer how traffic is forwarded and whether endpoints can communicate. Traceroute is directional and explicitly decouples forward and reverse connectivity. Reachability questions search for flows matching path and header constraints and return examples of flows. Header constraints, path constraints, actions, maximum traces, inverted search, and filter-ignore options make the query semantics explicit. Bidirectional reachability performs forward and return analyses and can model sessions created by the forward pass before searching return flows.[^11]

ACL/filter analysis includes whether traffic is permitted or denied and whether ACL lines can match, including shadowed or empty lines.[^12]

#### 5.1.4 Data model and schema model

| Concept | Batfish concept | Cadastre mapping |
| --- | --- | --- |
| Snapshot | Parsed network configuration state | `NetworkContextSnapshot` input |
| Header constraints | Packet fields and protocol constraints | `ReachabilityQuery.packet_selector` |
| Path constraints | Location/path requirements | `ReachabilityQuery.path_constraints` |
| Flow disposition/action | Outcome of modeled flow | `ReachabilityResult.result_state` candidate input |
| Trace | Sequence of hops and transformations | `ReachabilityPathStep[]` |
| ACL/filter result | Permit/deny and line behavior | `NetworkPolicyRuleObservation` or evidence row |
| Unsupported feature | Feature not represented by Batfish | `ReachabilityModelCapabilityMatrix.unsupported_component` |

#### 5.1.5 Reachability semantics

A positive Batfish reachability result means the modeled snapshot contains at least one example flow satisfying the query. It is packet/model-level, not observed traffic. It does not guarantee service access, endpoint listener availability, live data-plane state, or identity-conditioned access.[^11]

A negative result is meaningful only to the extent the snapshot, parser support, modeled features, and query constraints are complete. If Cadastre uses Batfish output, it must preserve whether the answer is example-path evidence or exhaustive proof.

#### 5.1.6 Validation and completeness model

Batfish supported-feature documentation lists supported vendors and features and identifies limited support or unsupported areas for other platforms and constructs. Unsupported features must map to `unsupported`, not `not_reachable`.[^13]

Cadastre must require:

```text
snapshot_id
snapshot_valid_time
parser_version
vendor_feature_support_manifest
unsupported_feature_set
source_config_completeness_receipts
query_hash
answer_hash
path_enumeration_mode
```

#### 5.1.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Snapshot-based analysis | `adopt` | `NetworkContextSnapshot` |
| Header and path constraints | `adopt` | `ReachabilityQuery` |
| Flow dispositions and traces | `adapt` | `ReachabilityResult`, `ReachabilityPathStep` |
| Bidirectional reachability distinction | `adopt` | `claim_kind = session_reachability` |
| ACL/filter line analysis | `adapt` | `NetworkPolicyRuleObservation`, validation fixtures |
| Unsupported-feature visibility | `adopt` | `ReachabilityModelCapabilityMatrix` |
| Example flows/traces | `cautionary_reference` | `path_enumeration_mode = representative` |

#### 5.1.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Treating a Batfish positive as live reachability | Modeled configuration is not observed or probed traffic | Store as `ReachabilityAnalysisArtifact` |
| Treating example traces as complete path enumeration | Example evidence does not prove all paths | Require `path_enumeration_mode = representative` |
| Treating unsupported features as pass or fail | Unsupported is neither reachable nor unreachable | Return `unsupported` |
| Treating forward reachability as session reachability | Return path is separate | Require bidirectional/session claim kind |

### 5.2 AWS VPC Reachability Analyzer and Network Access Analyzer

#### 5.2.1 Source inventory and inspection limits

AWS official docs for VPC Reachability Analyzer and Network Access Analyzer were inspected. No AWS account, IAM permission, VPC configuration, or API call was tested.

#### 5.2.2 Problem solved and non-purpose

Reachability Analyzer is a configuration-analysis tool for testing connectivity from a source to a destination in VPC scope. It builds a model of network configuration and does not send packets. A reachable result provides hop-by-hop virtual path details; an unreachable result identifies blocking components.[^14]

Network Access Analyzer performs automated reasoning over AWS network access and uses static analysis; it does not send packets and ignores transient or service failures. Its findings can be representative rather than exhaustive.[^16]

#### 5.2.3 Architecture and operational model

Reachability Analyzer query shape is source, destination, protocol/port, and optional path. It is scoped by region, VPC connectivity, accounts, and supported resources. AWS documents supported sources/destinations, intermediate resources, and path components including EC2 instances, ENIs, security groups, NACLs, routes, load balancers, NAT gateways, AWS Network Firewall, transit gateways, and VPN components.[^15]

Network Access Analyzer uses access scopes and can produce findings, but docs state unsupported resources, unsupported configurations, path-length bounds, unidirectional limits, no cross-account/region paths, no target-health consideration, and possible finding changes as AWS support changes.[^16]

#### 5.2.4 Data model and schema model

| AWS concept | Cadastre mapping |
| --- | --- |
| Path analysis | `ReachabilityAnalysisArtifact` |
| Hop-by-hop path | `ReachabilityPathStep[]` |
| Blocking component | `ReachabilityResult.blocking_components[]` |
| Explanation code | `ReachabilityResult.diagnostics[]` |
| Access scope | `ReachabilityQuery.path_constraints` or `ReachabilityClaimPolicy.scope` |
| Unsupported resource/config | `ReachabilityModelCapabilityMatrix.unsupported_components[]` |
| IAM/API permission | `ReachabilitySourceCompletenessProfile.permission_scope` |

#### 5.2.5 Reachability semantics

A positive Reachability Analyzer result means AWS’s configuration model found a reachable path inside the supported scope. It does not mean packets were sent, target health was checked, endpoint listener state exists, identity policy permits access, or enterprise-wide reachability exists. AWS explicitly documents target-health omissions and unsupported Network Firewall rule-group limitations.[^15]

A negative result means no path was found in the modeled supported scope or a blocking component was identified. It must not be generalized outside supported resources, permissions, region/account/VPC scope, or unsupported-feature limits.

#### 5.2.6 Validation and completeness model

Cadastre must capture:

```text
aws_account_id
aws_region
vpc_scope
source_resource_arn
destination_resource_arn
protocol
port
analysis_id
analysis_time
supported_resource_manifest
permission_profile_ref
unsupported_component_list
explanation_codes
path_steps
```

#### 5.2.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Hop-by-hop modeled path | `adapt` | `ReachabilityPathStep` |
| Blocking component output | `adopt` | `ReachabilityResult.blocking_components` |
| Supported-resource matrix | `adopt` | `ReachabilityModelCapabilityMatrix` |
| Access scopes/findings | `adapt` | `ReachabilityClaimPolicy`, `ReachabilityQuery` |
| Static/no-packet boundary | `adopt` | User-facing non-implication rules |
| Representative findings | `cautionary_reference` | `path_enumeration_mode` |

#### 5.2.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| AWS positive implies enterprise-wide reachability | Provider scope is bounded | Label as AWS-scoped configuration analysis |
| AWS negative implies no path anywhere | Unsupported resources and scope gaps exist | Return `unknown_partial` or `unsupported` when outside model |
| Network Access Analyzer finding is Cadastre GoldFact | Provider output is analysis artifact | Require source authority and evidence refs |
| Security-group allow implies service access | Listener/backend/identity context missing | Require service or identity claim kind evidence |

### 5.3 Google Cloud Connectivity Tests

#### 5.3.1 Source inventory and inspection limits

Google Cloud official Connectivity Tests documentation was inspected. No GCP project, IAM permission, or test execution was performed.

#### 5.3.2 Problem solved and non-purpose

Connectivity Tests evaluates network connectivity using configuration analysis and, where supported, live data-plane analysis. It diagnoses possible paths, firewall/routing decisions, and load-balancer backend paths. It is not a universal delivery proof. Google explicitly states that configuration analysis is based entirely on configuration and may not reflect the actual data plane; a deliver state does not guarantee delivery.[^17]

#### 5.3.3 Architecture and operational model

Connectivity Tests supports many GCP constructs including VPC networks, peering, Shared VPC, routes, NAT except NAT64, firewall policies, policy-based routes, IPv6, and load balancers. Live data-plane analysis has significant unsupported cases including non-GCP sources, GKE pods, certain load balancers, Cloud VPN, and NAT64.[^17]

Load-balancer configuration analysis may produce one trace per possible backend path, does not consider backend health check status as live availability, and may omit some backends when there are too many.[^19]

#### 5.3.4 Data model and schema model

| GCP concept | Cadastre mapping |
| --- | --- |
| Test result state | `ReachabilityResult.result_state` |
| Trace | `ReachabilityPathStep[]` |
| Resource card with no permission | `ReachabilityEvidenceRow.permission_gap` |
| Forward state | `ReachabilityPathStep.forwarding_state` |
| Firewall/routing result | Network policy and route evidence |
| Multiple backend traces | `path_enumeration_mode = representative` or `bounded_subset` |
| Live data-plane result | Non-authoritative probe evidence |

#### 5.3.5 Reachability semantics

Google result states include `Reachable`, `Unreachable`, `Ambiguous`, and `Undetermined`; undetermined can result from permission errors.[^18]

A positive configuration-analysis deliver state means the simulated packet could be delivered according to represented configuration. It does not guarantee data-plane delivery. Live data-plane analysis supplements configuration analysis only for supported routes and is not exhaustive for load-balanced backends or ECMP-like conditions.[^17]

#### 5.3.6 Validation and completeness model

Cadastre must capture:

```text
project_id
region_or_location
test_id
analysis_mode = configuration | live_data_plane | combined
source_endpoint
destination_endpoint
protocol
port
trace_count
omitted_backend_count
permission_gap_refs
unsupported_config_refs
verified_time
```

#### 5.3.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Explicit `Ambiguous`/`Undetermined` states | `adopt` | `ReachabilityResult.result_state` |
| Deliver does not guarantee delivery | `adopt` | User-facing summary rules |
| Permission gap surfaced in result | `adopt` | Completeness/authority decisions |
| LB/backend trace limits | `adapt` | `path_enumeration_mode` |
| Config versus live analysis separation | `adopt` | `analysis_mode` field |

#### 5.3.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| `Deliver` equals service reachable | Google says config deliver is not delivery guarantee | Use “configuration permits modeled delivery” |
| Live probe success equals complete reachability | Live analysis is scoped and incomplete | Store live probe separately |
| Permission-limited result is negative proof | Permission gaps may hide resources | Return `unknown_partial` |
| Load-balancer trace set is exhaustive | Backends can be omitted | Mark representative or bounded subset |

### 5.4 Azure Network Watcher and Azure Virtual Network Manager

#### 5.4.1 Source inventory and inspection limits

Microsoft Learn docs were inspected for IP Flow Verify, Next Hop, Effective Security Rules, Connection Troubleshoot, Network Watcher, and Azure Virtual Network Manager security admin rules. No Azure tenant, VM, VNet, NSG, or API call was tested.

#### 5.4.2 Problem solved and non-purpose

Azure decomposes network diagnostics into specialized tools.

| Azure tool | Problem solved | Non-purpose |
| --- | --- | --- |
| IP Flow Verify | Checks whether a packet to/from a VM is allowed or denied by configured security/admin rules | Not complete path or service access |
| Next Hop | Identifies next hop type/IP/route table for destination IP | Not firewall/policy proof |
| Effective Security Rules | Shows aggregated security rules applied to NIC | Not route or listener proof |
| Connection Troubleshoot | Runs connectivity diagnostics and probes | Not complete theoretical proof |
| Security admin rules | Applies higher-priority global allow/deny/always-allow guardrails | Not universal to all services/resources |

IP Flow Verify evaluates TCP/UDP packet parameters against VM NIC security rules and returns allowed/denied plus the rule. Next Hop provides next-hop type, IP, and route table ID for a VM/destination pair. Effective Security Rules shows aggregated inbound/outbound NSG and admin rules applied to a NIC. Connection Troubleshoot provides root-cause insights, NSG/UDR/blocked-port checks, latency, topology, and failed probe counts; Microsoft labels an agentless implementation as preview. Security admin rules are evaluated before NSGs; `Always Allow` and `Deny` terminate evaluation while ordinary `Allow` proceeds to NSG evaluation. Microsoft also documents eventual consistency and excluded resources/services.[^20][^21][^22][^23][^24]

#### 5.4.3 Architecture and operational model

Azure’s model is diagnostic composition, not a single oracle. Cadastre must not collapse IP Flow Verify, Next Hop, and Connection Troubleshoot into one boolean. A safe Cadastre analysis must retain each diagnostic as evidence with its own scope, time, permissions, and limitation metadata.

#### 5.4.4 Data model and schema model

| Azure concept | Cadastre mapping |
| --- | --- |
| IP Flow Verify allow/deny | `NetworkPolicyRuleObservation` or policy evidence |
| Next Hop type | `RouteEvidenceRow` |
| Effective security rules | `NetworkPolicyRuleObservation[]` |
| Connection Troubleshoot probes | Non-authoritative live-probe evidence |
| Admin rule precedence | Policy evaluation order model |
| Excluded resource/service list | `ReachabilityModelCapabilityMatrix.unsupported_components` |
| Eventual consistency | `SourceCompletenessProfile.staleness_rule` |

#### 5.4.5 Reachability semantics

A positive IP Flow Verify result means a configured security/admin rule permits a packet at the VM NIC rule-evaluation boundary. It does not prove route existence, return path, destination listener, load-balancer backend, host firewall, identity policy, or service access.

A positive Next Hop result means routing selects a next hop. It does not prove policy allow or service access.

Connection Troubleshoot combines diagnostic checks and probes, but live probe success or failure must not become complete theoretical reachability.

#### 5.4.6 Validation and completeness model

Cadastre must capture:

```text
subscription_id
resource_group
vm_or_nic_resource_id
network_watcher_region
diagnostic_tool
protocol
direction
local_ip
remote_ip
local_port
remote_port
rule_id
route_table_id
next_hop_type
probe_count
failed_probe_count
preview_feature_flag
eventual_consistency_window
unsupported_resource_refs
```

#### 5.4.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Diagnostic decomposition | `adopt` | Evidence-family matrix |
| Admin rule precedence | `adopt` | Policy evaluation order |
| Effective rules as evidence | `adapt` | `NetworkPolicyRuleObservation` |
| Next hop as route evidence | `adapt` | `RouteEvidenceRow` |
| Preview/live probe caveat | `cautionary_reference` | Live-probe non-authority |
| Eventual consistency | `adopt` | Completeness/staleness rules |

#### 5.4.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| IP Flow Verify allow equals reachability | Route, NAT, listener, return path absent | Use policy-evidence row only |
| Next Hop exists equals access | Policy and endpoint context absent | Use route-evidence row only |
| Connection Troubleshoot probe equals theory | Live probe is not exhaustive | Store as diagnostic/probe artifact |
| Admin allow bypasses all controls | NSG may still evaluate ordinary allow; excluded resources exist | Preserve precedence and exclusion metadata |

### 5.5 Kubernetes NetworkPolicy

#### 5.5.1 Source inventory and inspection limits

Kubernetes official documentation was inspected. No Kubernetes cluster, CNI plugin, or policy was executed.

#### 5.5.2 Problem solved and non-purpose

Kubernetes NetworkPolicy specifies L3/L4 traffic rules for pods in a cluster. A cluster must use a NetworkPolicy-supporting plugin; otherwise creating NetworkPolicy resources has no effect. NetworkPolicy does not prove application access, service mesh authorization, listener health, identity-conditioned access, or node/network-appliance reachability.[^25]

#### 5.5.3 Architecture and operational model

NetworkPolicy selects pods through `podSelector`, applies direction-specific `policyTypes`, and defines ingress/egress allow rules. Policies are additive. A connection is allowed only when both source egress and destination ingress policy requirements allow it, if either side is isolated. Selector semantics are YAML-sensitive: namespace and pod selectors can be combined in one peer or listed as separate peers with different meanings. IPBlock is intended for cluster-external IPs. Source/destination rewriting before or after NetworkPolicy evaluation is not defined consistently and varies by plugin, cloud provider, and Service implementation.[^25]

#### 5.5.4 Data model and schema model

| Kubernetes concept | Cadastre mapping |
| --- | --- |
| Namespace | Workload scope evidence |
| Pod selector | Endpoint/workload selector resolution |
| Namespace selector | Scope selector resolution |
| Policy type | Directional isolation evidence |
| Ingress rule | Destination policy evidence |
| Egress rule | Source policy evidence |
| IPBlock | External CIDR policy evidence with rewrite caveat |
| CNI plugin support | Model capability prerequisite |
| Service/NodePort/Ingress | Service/proxy evidence, not NetworkPolicy alone |

#### 5.5.5 Reachability semantics

A missing NetworkPolicy does not always imply full reachability in Cadastre. Kubernetes default semantics allow non-isolated pods, but only if a NetworkPolicy-capable plugin is present, no other enforcement layer applies, and source/destination rewrite behavior, service routing, host firewall, node policy, and CNI-specific behavior are known.

NetworkPolicy is direction-specific and additive. It is a workload policy input, not a graph reachability edge.

#### 5.5.6 Validation and completeness model

Cadastre must capture:

```text
cluster_id
api_server_version
cni_plugin_name
cni_plugin_version
network_policy_support_state
namespace_snapshot
pod_snapshot
service_snapshot
endpoint_slice_snapshot
network_policy_snapshot
selector_resolution_manifest
ip_rewrite_behavior
policy_valid_time
source_completeness_receipts
```

#### 5.5.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Direction-specific isolation | `adopt` | Workload reachability evidence |
| Additive allow rules | `adopt` | Policy evaluation algorithm |
| Selector resolution | `adopt` | `NetworkContextSnapshot` |
| CNI dependency | `adopt` | Capability matrix |
| Rewrite caveats | `adopt` | Unsupported/unknown result handling |
| Default allow/no policy | `cautionary_reference` | Prohibited-claim matrix |

#### 5.5.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Missing NetworkPolicy equals full reachability | CNI, defaults, service routing, mesh, and host policies may differ | Require capability and completeness evidence |
| Ingress allow equals connection allowed | Egress side may still block | Evaluate both directions |
| Pod selector match equals service access | Listener/service/backend context absent | Require service access claim kind |
| IPBlock semantics universal | Rewrite order varies | Mark unsupported or conditional unless known |

### 5.6 Cilium network policy

#### 5.6.1 Source inventory and inspection limits

Cilium stable documentation was inspected. The docs page identifies stable version `1.19.4`. No Cilium cluster, agent, Envoy proxy, DNS proxy, or policy was executed.[^26]

#### 5.6.2 Problem solved and non-purpose

Cilium implements Kubernetes NetworkPolicy plus CiliumNetworkPolicy and CiliumClusterwideNetworkPolicy with L3/L4/L7 controls. It is a workload policy engine, not a complete enterprise reachability oracle.[^26]

#### 5.6.3 Architecture and operational model

Cilium policy enforcement modes include `default`, `always`, and `never`. In default mode, endpoints are unrestricted until selected by policy. In `never`, all traffic is allowed even if rules select endpoints. Cilium deny policies are enabled by default since Cilium 1.9 and deny rules take precedence over allow rules across policy types. Cilium supports endpoint/service/entity/node/IP/CIDR/DNS-based L3 policy, L4 policy, and L7 policy. DNS/FQDN policy uses DNS responses to generate CIDR rules and depends on the DNS proxy and L7 DNS allow rules. L7 rules are embedded in L4 policies; L7 request allowance requires a matching rule, conflicting L7 parser types are rejected, and some features have host-policy or protocol limitations.[^27][^28][^29][^30]

#### 5.6.4 Data model and schema model

| Cilium concept | Cadastre mapping |
| --- | --- |
| Enforcement mode | `ReachabilityModelCapabilityMatrix.policy_enforcement_mode` |
| Endpoint selector | Workload selector evidence |
| CiliumNetworkPolicy | Policy evidence |
| Clusterwide policy | Cluster-scope policy evidence |
| Deny policy | Policy precedence row |
| L7 HTTP/DNS/FQDN | Service or identity-conditioned evidence, depending claim |
| DNS proxy | Capability prerequisite |
| Generated CIDR from DNS | Dynamic translation evidence with TTL |

#### 5.6.5 Reachability semantics

A Cilium allow rule does not imply reachability when deny rules match, endpoint is in default-deny, egress or ingress counterpart is missing, DNS/FQDN translation is stale or unsupported, or L7 policy blocks the application request.

A positive Cilium policy evaluation may support packet, session, or service-access claims only when the relevant policy layer, endpoint selector resolution, enforcement mode, DNS proxy state, and L7 conditions are modeled.

#### 5.6.6 Validation and completeness model

Cadastre must capture:

```text
cilium_version
policy_enforcement_mode
policy_types_present
deny_policy_snapshot
allow_policy_snapshot
endpoint_identity_snapshot
service_snapshot
dns_proxy_enabled
fqdn_cache_snapshot
fqdn_ttl_basis
l7_proxy_enabled
envoy_policy_snapshot
unsupported_l7_feature_refs
```

#### 5.6.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Enforcement modes | `adopt` | Capability matrix |
| Deny-before-allow | `adopt` | Deterministic evaluation |
| Clusterwide policy | `adapt` | Scope model |
| L7 HTTP/DNS/FQDN | `adapt` | Service access conditions |
| DNS-to-CIDR dynamic behavior | `cautionary_reference` | Temporal translation evidence |
| Version-specific caveats | `adopt` | Version manifest |

#### 5.6.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Cilium allow means application access | Deny/L7/listener/backend may block | Evaluate full claim kind |
| DNS name match equals stable reachability | DNS-derived CIDRs are dynamic and TTL-bound | Require FQDN cache evidence |
| Latest docs imply production behavior | Docs can be version-specific | Pin Cilium version and docs |
| Policy enforcement mode ignored | `never` mode allows traffic despite policy definitions | Require mode in snapshot |

### 5.7 NIST Zero Trust Architecture

#### 5.7.1 Source inventory and inspection limits

NIST SP 800-207 official CSRC material was inspected. No zero-trust product or implementation profile was evaluated.

#### 5.7.2 Problem solved and non-purpose

NIST SP 800-207 defines Zero Trust Architecture concepts. It states that no implicit trust is granted based solely on network location or asset ownership, and that subject and device authentication and authorization occur before a session to an enterprise resource is established. It does not provide a reachability solver, graph model, or Cadastre data model.[^31]

#### 5.7.3 Architecture and operational model

Zero trust distinguishes subjects, devices, resources, policy decision points, policy enforcement points, continuous evaluation, and session authorization. For Cadastre, this is a non-implication reference: subnet/IP reachability is not service access.

#### 5.7.4 Data model and schema model

| NIST concept | Cadastre mapping |
| --- | --- |
| Subject | Identity evidence |
| Device | Endpoint posture evidence |
| Resource | Destination/service evidence |
| PDP | Access policy decision source |
| PEP | Enforcement point evidence |
| Session | Conditional access result |
| Device posture | `condition_context.device_posture` |
| Policy context | `required_conditions[]` |

#### 5.7.5 Reachability semantics

A network path is insufficient for access. `identity_conditioned_service_access` must require subject identity, device posture, resource, policy, enforcement point, and session context.

#### 5.7.6 Validation and completeness model

Cadastre must return `conditional` or `unknown_partial` unless all required identity and access evidence is present and within validity windows.

#### 5.7.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| No network-location implicit trust | `adopt` | Prohibited claims |
| PDP/PEP separation | `adapt` | Identity-conditioned access model |
| Subject/device/resource/session authorization | `adopt` | `condition_context` |
| Posture and dynamic policy | `study` | Future access model |

#### 5.7.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Zero-trust label means access denied or allowed | Architecture concept is not source evidence | Require PDP/PEP evidence |
| Network reachability implies resource access | NIST rejects implicit trust | Use identity-conditioned claim |
| Identity group alone implies endpoint access | Policy/resource/posture missing | Require full condition context |

### 5.8 OpenConfig ACL

#### 5.8.1 Source inventory and inspection limits

OpenConfig ACL YANG documentation/schema was inspected. No device, vendor implementation, or NETCONF/gNMI retrieval was executed.

#### 5.8.2 Problem solved and non-purpose

OpenConfig ACL defines a vendor-neutral ACL schema with ACL sets, ordered entries, match fields, actions, and operational state. It does not solve reachability by itself because routing, NAT, topology, stateful behavior, load balancing, and endpoint context are outside an individual ACL rule.[^32]

#### 5.8.3 Architecture and operational model

ACL sets are configured with names and types; type determines allowed fields. ACL entries are keyed by sequence ID, and sequence ID determines rule order, applied from lowest to highest. Entries include L2/L3/L4 match fields such as MACs, IPv4 source/destination prefixes, DSCP sets, protocol, hop-limit, and actions. Operational state includes counters such as matched packets and octets.[^32]

#### 5.8.4 Data model and schema model

| OpenConfig concept | Cadastre mapping |
| --- | --- |
| ACL set | `NetworkPolicyObservation.policy_set_id` |
| ACL type | Policy family |
| Sequence ID | Deterministic evaluation order |
| Match fields | Packet selector fields |
| Forwarding action | Permit/deny/reject policy action |
| Log action | Diagnostic/audit setting |
| Counters | Observed policy-hit evidence, not reachability proof |

#### 5.8.5 Reachability semantics

An ACL permit means a packet matching the ACL entry is allowed by that ACL stage. It does not prove routing, NAT, return path, service availability, endpoint listener state, identity policy, or complete path reachability.

#### 5.8.6 Validation and completeness model

Cadastre must require:

```text
device_id
acl_set_name
acl_type
entry_sequence_id
ordered_entry_snapshot
match_field_snapshot
action_snapshot
operational_state_snapshot_optional
vendor_translation_profile
unsupported_match_field_refs
```

#### 5.8.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| ACL set/entry structure | `adopt` | `NetworkPolicyRuleObservation` |
| Sequence-ordered evaluation | `adopt` | Deterministic algorithm |
| Match/action schema | `adapt` | Packet selector and policy evidence |
| Operational counters | `cautionary_reference` | Observed evidence only |
| Vendor-neutral normalization | `adopt` | Policy normalization profile |

#### 5.8.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| ACL allow equals reachability | ACL is one stage only | Combine route/NAT/topology/endpoint evidence |
| Counter hit equals complete coverage | Counter is observed hit evidence | Use as supporting observation only |
| Vendor mapping is lossless | Vendor features can exceed model | Use unsupported/partial state |

### 5.9 NetKAT

#### 5.9.1 Source inventory and inspection limits

NetKAT official site, Google NetKAT repository page, and publication descriptions were inspected. No NetKAT verifier was executed. The Google C++ implementation page states no releases are published and warns that the implementation may diverge from academic NetKAT and is not an officially supported Google product.[^33][^34]

#### 5.9.2 Problem solved and non-purpose

NetKAT is a domain-specific language for specifying and reasoning about packet-forwarding networks. It is grounded in Kleene Algebra with Tests and provides a sound and complete equational theory for reasoning about network behavior. It supports verification of reachability, isolation, and program equivalence. NetKAT is not a Cadastre production reachability engine by itself, not a source collector, not a cloud diagnostics tool, and not a user-facing graph semantics model.[^33]

#### 5.9.3 Architecture and operational model

NetKAT is a formal language and reasoning framework. It models packet predicates and forwarding programs algebraically, enabling automatic verification and equivalence checking. Cadastre can use this as a semantic reference for solver contracts and acceptance tests, not as governing architecture.

#### 5.9.4 Data model and schema model

| NetKAT concept | Cadastre mapping |
| --- | --- |
| Packet tests/predicates | `ReachabilityQuery.packet_selector` |
| Packet transformations | NAT/translation or forwarding step abstraction |
| Sequential/choice/star composition | Route and policy program composition reference |
| Reachability property | Validation scenario or solver capability |
| Isolation property | Negative reachability validation scenario |
| Program equivalence | Replay/normalization equivalence test |

#### 5.9.5 Reachability semantics

A NetKAT reachability result is formal packet-program semantics under the encoded model. It says nothing about unmodeled resources, live data-plane state, identity-conditioned access, endpoint listeners, workload policy, or application success.

#### 5.9.6 Validation and completeness model

Cadastre may use NetKAT-like scenarios for:

```text
packet selector determinism
route-policy composition
deny-over-allow proofs
isolation validation
equivalence between normalized vendor policy and source-native policy
```

But any solver must declare supported packet fields, transformations, statefulness, NAT behavior, path enumeration behavior, and unsupported constructs.

#### 5.9.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Formal packet semantics | `study` | Solver semantics |
| Reachability/isolation/equivalence verification | `adapt` | Validation fixtures |
| Algebraic equivalence | `study` | Policy normalization tests |
| Sound/complete theory under model | `cautionary_reference` | Capability-bound claims |

#### 5.9.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Formal proof over encoded model equals real-world access | Model may omit critical context | Require model capability and evidence completeness |
| NetKAT graph semantics replaces Cadastre graph | Cadastre graph is fact projection | Keep NetKAT as solver/test reference |
| Packet reachability equals service access | Listener/identity absent | Use separate claim kinds |

### 5.10 BloodHound and OpenGraph

#### 5.10.1 Source inventory and inspection limits

Official BloodHound/OpenGraph docs and prior Cadastre RES-008 were inspected. No BloodHound instance or OpenGraph upload was executed.

#### 5.10.2 Problem solved and non-purpose

BloodHound/OpenGraph solves identity-path graph analysis and pathfinding eligibility, not network theoretical reachability. OpenGraph supports custom nodes, edges, and properties and distinguishes generic graphs from structured graphs. Structured graphs use extension definition schemas to enable enhanced features; generic graphs support basic exploration and search but not full structured pathfinding.[^35]

#### 5.10.3 Architecture and operational model

OpenGraph FAQ states that structured graphs support Search and Pathfinding, while generic graphs support Search only; pathfinding, findings, and analysis require traversable relationship kinds defined in an extension definition schema. Generic graph data is treated as non-traversable. BloodHound traversable edges represent relationships where the starting node can take control of the ending node to a degree that enables abuse of outgoing edges. Non-traversable edges can combine into traversable post-processed edges, and pathfinding includes only traversable edges.[^36][^37]

RES-008 identifies the strongest transferable ideas as explicit traversability, separation between generic graph ingestion and structured pathfinding eligibility, stable IDs, and post-processed edges as generated path edges from supporting facts rather than primary evidence.[^10]

#### 5.10.4 Data model and schema model

| BloodHound/OpenGraph concept | Cadastre mapping |
| --- | --- |
| Generic graph | Non-authoritative graph input/reference |
| Structured graph | `GraphReadModelSchemaProfile` analogue |
| `is_traversable` | Future `GraphEdgeSemantics.traversal_eligibility` |
| Post-processed edge | Derived edge through `DerivationRuleBundle`, not source truth |
| Source kind | Source metadata; not deletion authority |
| Pathfinding | Analysis rule output, not fact authority |

#### 5.10.5 Reachability semantics

BloodHound graph pathfinding is identity/attack-path traversability, not network reachability. A traversable path does not prove route, firewall, NAT, listener, workload policy, or zero-trust access.

#### 5.10.6 Validation and completeness model

Cadastre must require that graph traversal eligibility is declared per edge type and claim family. Traversability must never be inferred from edge existence.

#### 5.10.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| First-class traversability | `adopt` | `GraphEdgeSemantics.traversal_eligibility` |
| Generic versus structured graph separation | `adopt` | Graph schema/profile activation |
| Post-processed generated edges | `adapt` | Derivation rule boundary |
| Search versus pathfinding distinction | `adopt` | Query contract |
| Source-kind deletion hazard | `cautionary_reference` | Graph apply/delete safety |

#### 5.10.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Graph path implies network reachability | Identity path lacks network context | Separate graph traversal from reachability result |
| Traversable edge is source truth | Traversability is analysis eligibility | Preserve source fact and derivation refs |
| Generic graph edge enters pathfinding | Structured eligibility required | Default `traversal_eligibility = none` |
| Post-processed edge as primary evidence | Derived edge hides supporting facts | Require supporting evidence refs |

### 5.11 Source-adapter and ingestion patterns

#### 5.11.1 Source inventory and inspection limits

RES-006 was inspected as prior Cadastre research. It examined CloudQuery, Steampipe, NiFi, Data Prepper, and Debezium. Those systems were not re-executed for this report.[^9]

#### 5.11.2 Problem solved and non-purpose

Source-adapter systems solve collection, state, pagination, incremental sync, CDC, pipeline, and ingestion workflow problems. They do not directly produce Cadastre reachability truth.

RES-006 states that adapter collection stages are permitted to emit only `RawRecord[]`, `SourceCompletenessReceipt[]`, and adapter error records; they must not emit silver observations, coverage assertions, identity decisions, unresolved target references, gold facts, graph deltas, CIM projection results, or graph apply results.[^9]

#### 5.11.3 Architecture and operational model

The transferable model is raw-first collection with explicit source state, source-call policy, completeness receipts, diagnostic records, and version-manifested state. `StageStateRecord` must be explicit, canonicalized, hashable, replayable, and included in `VersionManifest` when it affects output.[^9]

#### 5.11.4 Data model and schema model

| Adapter concept | Cadastre reachability input role |
| --- | --- |
| Raw source record | Evidence input only |
| Completeness receipt | Completeness decision input |
| Stage state | Replay and deterministic input |
| Source call policy | Collection validity input |
| Live source probe | Non-authoritative diagnostic |
| Acknowledgment/provenance | Ingestion lineage only by default |
| CDC offset/heartbeat | Liveness/state only by default |

#### 5.11.5 Reachability semantics

Adapter output cannot directly assert reachability. It may collect firewall rules, routes, NAT tables, cloud analyzer outputs, workload policies, identity policies, or endpoint context, but each dataset must be evaluated through source completeness, source authority, and model capability before a reachability result can be emitted.

#### 5.11.6 Validation and completeness model

`SourceCompletenessReceipt` is not self-authorizing truth. RES-006 states that absence, retraction, cleanup, and watermark advancement require evaluation against an active `SourceCompletenessProfile`; unsafe states default to no absence, no retraction, no cleanup, and no source watermark advancement.[^9]

#### 5.11.7 Transferability to Cadastre

| Finding | Transfer class | Cadastre contract strengthened |
| --- | --- | --- |
| Raw-first collection | `adopt` | Reachability evidence ingestion |
| Completeness receipts | `adopt` | `ReachabilitySourceCompletenessProfile` |
| Stage state/version manifest | `adopt` | Replay determinism |
| Live probe non-authority | `adopt` | Prohibited claims |
| Destination cleanup rejection | `adopt` | Source authority boundary |
| Completeness evidence rows | `adapt` | Minimum evidence model |

#### 5.11.8 Concepts that must not transfer

| Unsafe transfer | Protected Cadastre boundary | Safer alternative |
| --- | --- | --- |
| Adapter output directly emits reachability GoldFact | Violates raw-first boundary | Evaluate in reachability analysis stage |
| Live query/probe row becomes reachability truth | Non-authoritative by default | Store as diagnostic/probe artifact |
| Ack/provenance becomes source completeness | Delivery is not source coverage | Require completeness profile |
| CDC offset/heartbeat becomes fact time | Connector state is not domain time | Use only as collection state |

## 6. Cross-source comparison

### 6.1 Adopt/adapt/reject/study/cautionary-reference decision matrix

| Source family | Adopt | Adapt | Study | Reject | Cautionary reference |
| --- | --- | --- | --- | --- | --- |
| Batfish | Snapshot, header/path constraints, bidirectional distinction, unsupported-feature metadata | Traces, ACL analysis, flow dispositions | Direct solver integration | Example trace as exhaustive proof | Representative path evidence |
| AWS analyzers | Supported-resource limits, blocking components, no-packet boundary | Hop-by-hop paths, access scopes | API integration | Provider result as GoldFact | Representative findings and unsupported configs |
| GCP Connectivity Tests | `Reachable`/`Unreachable`/`Ambiguous`/`Undetermined`, config/live separation | Load-balancer traces, permission gaps | API integration | `Deliver` as data-plane guarantee | Backend omissions and live limits |
| Azure diagnostics | Diagnostic decomposition, admin precedence, eventual consistency | Effective rules, next-hop, probes | API integration | IP Flow Verify as reachability | Preview/probe limitations |
| Kubernetes NetworkPolicy | Direction isolation, additive rules, selector resolution, CNI requirement | Workload evidence model | CNI-specific expansion | Missing policy equals full reachability | Rewrite behavior |
| Cilium | Enforcement mode, deny precedence, L7/DNS/FQDN caveats | FQDN cache and L7 policy evidence | Deep Cilium integration | Allow rule equals service access | Version-specific docs |
| NIST ZTA | No implicit network trust, PDP/PEP conditions | Access-condition matrix | Product-specific ZTA integrations | Network reachability equals access | Identity/posture incompleteness |
| OpenConfig ACL | Ordered rule model, match/action schema | Vendor-neutral policy observations | Policy normalization | ACL allow equals reachability | Vendor translation gaps |
| NetKAT | Formal semantics as validation inspiration | Reachability/isolation/equivalence fixtures | Solver semantics | NetKAT as Cadastre graph semantics | Model-only proof |
| BloodHound/OpenGraph | Traversability field, structured path eligibility | Derived-edge boundary | Graph analysis ergonomics | Graph path as network reachability | Pathfinding-edge non-truth |
| Source-adapter systems | Raw-first, state, completeness, replay | Reachability evidence ingestion profiles | Adapter authoring patterns | Adapter as reachability oracle | Probe/ack/provenance non-authority |

### 6.2 Network policy/routing/NAT/load-balancer evidence-family matrix

| Evidence family | Required content | Positive claim allowed | Negative claim allowed | Missing/partial/stale behavior | Primary source families |
| --- | --- | --- | --- | --- | --- |
| Topology | Nodes, interfaces, links, zones, VPC/VNet/subnet/namespace/workload relationships | Packet/session only when complete for scope | Only with complete scoped topology | `unknown_partial` | Batfish, cloud configs, Kubernetes |
| Route | Route tables, next hops, priorities, policy-based routes, dynamic route state when applicable | Packet/session | Only with route completeness | `unknown_partial` | Batfish, AWS, GCP, Azure |
| Firewall/policy | Ordered rules, precedence, action, direction, selectors, admin/deny precedence | Packet/session/service depending layer | Only with complete policy and precedence | `unknown_partial` or `unsupported` | OpenConfig, AWS, GCP, Azure, Kubernetes, Cilium |
| NAT/translation | Translation rules, before/after addresses/ports, statefulness, proxy behavior | Packet/session if modeled | Only if translations complete | `unknown_partial` | AWS NAT, GCP NAT, Azure/NVA, service/proxy |
| Load balancer/proxy | Forwarding rules, listeners, backend selection, health behavior, proxy termination, source rewrite | Service only when backend/listener complete | Only if backend/health/context complete | `conditional` or `unknown_partial` | AWS, GCP, Azure, Kubernetes Services |
| Endpoint access | Host firewall, listener, process/service, workload policy, endpoint health | Service | Only with endpoint completeness | `unknown_partial` | Endpoint/EDR, Kubernetes, Cilium |
| Identity access | Subject, service account, groups, roles, posture, PDP/PEP, session authorization | Identity-conditioned service access | Only with authoritative identity/access data | `conditional` or `unknown_partial` | NIST ZTA, IAM, service mesh |
| Observed traffic | Flow logs, packet/session logs, counters | `observed_connection` only | Not by default | No negative claim | Flow sources, OpenConfig counters |
| Live probes | Probe attempts/results | Diagnostic only by default | Diagnostic only by default | No reachability truth | GCP live analysis, Azure Connection Troubleshoot |

## 7. Minimum evidence before theoretical reachability claims

### 7.1 Minimum evidence before reachability claim emission

| Evidence class | Required content | Required source datasets | Required completeness proof | Required authority proof | Missing/partial/stale/unsupported behavior | Required for packet | Required for session | Required for service | Required for identity-conditioned access | Positive claim support | Negative claim support |
| --- | --- | --- | --- | --- | --- | ---: | ---: | ---: | ---: | --- | --- |
| Endpoint identity evidence | Source and destination selectors resolved to concrete entities or addresses with scope | Asset inventory, cloud resource inventory, workload inventory | Complete selector resolution for query scope | Source authority for endpoint identity | `unknown_partial` | Yes | Yes | Yes | Yes | Both | Both if complete |
| Packet selector | Protocol, src/dst IP, dst port, optional source port, address family | Query request | Request validation, not source completeness | Caller/query authority | `VALIDATION_ERROR` if absent for claim kind requiring it | Yes | Yes | Yes | Yes | Both | Both |
| Valid and known time | `as_of_valid_time`, `as_of_known_time`, source snapshot times | Version manifest, table snapshots | Snapshot availability | Table/profile authority | `VALIDATION_ERROR` or `unknown_partial` | Yes | Yes | Yes | Yes | Both | Both |
| Topology evidence | Interfaces, links, zones, VPC/VNet/subnet, namespace/workload topology | Network inventory, cloud config, Kubernetes inventory | Scope-complete topology receipt | Source authority for topology | `unknown_partial` | Yes | Yes | Yes | Yes | Both | Both |
| Route evidence | Route tables, priorities, dynamic/static route state, next hops | Routing control plane, cloud route config | Route table completeness by scope | Route source authority | `unknown_partial` | Yes | Yes | Yes | Yes | Both | Both |
| Firewall and policy evidence | Ordered rules, selectors, priorities, actions, statefulness, admin/deny precedence | Firewall policy, SG/NACL, NSG, NetworkPolicy, Cilium policy, ACLs | Policy scope completeness | Policy source authority | `unknown_partial` or `unsupported` | Yes | Yes | Yes | Yes | Both | Both |
| NAT and translation evidence | NAT/proxy rules, address/port rewrite, stateful behavior, translation order | NAT gateways, firewall configs, load balancers, service mesh | Translation completeness | Translation source authority | `unknown_partial` | When present | When present | Yes when present | Yes when present | Both | Both |
| Load balancer and backend evidence | Listeners, pools, backend selection, health-check policy, proxy termination | LB configs, service endpoints, health APIs | Backend set completeness | LB source authority | `unknown_partial` or `conditional` | No | Sometimes | Yes | Yes | Positive only when complete | Negative only when health/backend complete |
| Endpoint access context | Listener, host firewall, workload endpoint, process/service, endpoint health | Endpoint telemetry, host firewall, workload inventory | Endpoint context completeness | Endpoint source authority | `unknown_partial` | No | No | Yes | Yes | Positive and negative if complete | Both if complete |
| Identity privilege context | Subject, groups, roles, service account, resource policy, posture, PDP/PEP | IAM, ZTA, service mesh, directory, access broker | Identity/access completeness | Identity/access source authority | `conditional` or `unknown_partial` | No | No | No | Yes | Conditional positive | Conditional negative if complete |
| Source completeness | Receipt plus active profile per dataset/scope/time | Completeness receipts and evidence rows | `EvaluateSourceCompleteness = complete` or equivalent | Profile permits claim kind | `unknown_partial` | Yes | Yes | Yes | Yes | Both | Required for negative |
| Source authority | Fact/predicate/source authority rows | SourceAuthorityProfile | Profile active and scoped | Authority permits claim | `unknown_partial` | Yes | Yes | Yes | Yes | Both | Both |
| Model capability | Supported resources, policies, protocols, transformations, appliances | Solver capability manifest | Capability covers every required component | Solver profile active | `unsupported` | Yes | Yes | Yes | Yes | Both | Both |
| Directionality | Forward, reverse, stateful-return, initiator/responder rules | Flow-role, routing, policy, stateful evidence | Direction completeness | Cadastre rules | `unknown_partial` or `ambiguous` | Yes | Yes | Yes | Yes | Positive | Negative only with complete direction |
| Evidence lineage | Raw/silver/source refs for every claim input | Raw records, observations, artifacts | Every ref available or retention-qualified | Evidence refs authorized | Fact emission prohibited | Yes | Yes | Yes | Yes | Required for fact | Required for fact |
| Version manifest | Analyzer, snapshot, source, profile, solver, rules, hashes | VersionManifest | All refs present and checksummed | Active profile refs | `VERSION_MANIFEST_MISMATCH` | Yes | Yes | Yes | Yes | Required for replay | Required for replay |

### 7.2 Result-state matrix with GoldFact and graph-edge eligibility

| `result_state` | Definition | GoldFact emission allowed | Graph edge emission allowed | Required user-facing wording | Default `graph_effect` |
| --- | --- | ---: | ---: | --- | --- |
| `reachable` | Complete modeled evidence supports the requested claim in scope | Only if `ReachabilityClaimPolicy` permits and all evidence refs exist | Only if future edge semantics and claim policy permit | “Modeled reachable within the declared scope and evidence set.” | `none` |
| `not_reachable` | Complete modeled evidence proves no path/access for the requested scoped claim | Only if policy permits negative facts and completeness is complete | Only if future edge semantics permit negative edge; default no | “Modeled not reachable within the declared scope; this is not a statement outside that scope.” | `none` |
| `conditional` | Claim holds only under explicit returned conditions | Prohibited by default | Prohibited by default | “Reachability is conditional on the listed protocol, route, identity, posture, time, or policy conditions.” | `none` |
| `unknown_partial` | Evidence, completeness, permission, freshness, or scope is insufficient | No | No | “Cadastre cannot determine reachability because required evidence is missing, stale, partial, permission-limited, or outside scope.” | `none` |
| `unsupported` | Solver cannot represent a relevant component or behavior | No | No | “Cadastre cannot evaluate this query because the model does not support the listed components or behaviors.” | `none` |
| `conflicted` | Evidence sources conflict and authority rules cannot resolve | No | No | “Cadastre found conflicting evidence and did not choose a reachability result.” | `none` |

## 8. Required prohibited claims

| Claim attempt | Required analysis | Required Cadastre result |
| --- | --- | --- |
| Observed flow implies reachability. | Observed traffic proves only that traffic was observed under the evidence conditions. It does not prove complete theoretical reachability.[^2][^5] | Reject. |
| No observed flow implies no reachability. | Absence of observed traffic is not negative evidence unless the source contract defines complete coverage and meaningful absence.[^2] | Reject by default. |
| Firewall allow implies reachability. | Routing, NAT/translation, destination policy, endpoint context, and direction may still block. | Reject unless all required evidence is complete. |
| Route exists implies reachability. | Policy, NAT, translation, endpoint policy, and service context may still block. | Reject unless complete. |
| One-way path implies bidirectional session. | Return path or stateful-return behavior must be modeled. | Reject for session claims. |
| Security group allow implies service access. | Listener, load balancer, backend, host firewall, and identity policy may be missing. | Reject unless service context complete. |
| Kubernetes missing NetworkPolicy always implies full reachability. | CNI behavior, enforcement mode, namespace scope, service routing, host policies, and selectors must be known. | Reject. |
| Identity group membership implies endpoint access. | Access policy, resource, device posture, and network path must be modeled. | Reject. |
| Provider analyzer positive result implies enterprise-wide reachability. | Provider analyzers are bounded by resource, account, region, permission, and feature support. | Reject outside provider scope. |
| Provider analyzer negative result implies no reachability. | Unsupported resources, stale config, partial scope, or unmodeled appliances can exist. | Reject outside supported scope. |
| Structural nodes such as `Internet`, `Everyone`, `AnyHost`, or `AnyNetwork` imply universal reachability. | Structural/global aliases are modeling conveniences and PRD MVP emits none.[^2] | Reject. |
| Live probe success implies complete theoretical reachability. | A probe is one observation, not exhaustive theoretical coverage. | Reject. |
| Live probe failure implies no theoretical reachability. | Alternate paths or conditions may exist. | Reject unless complete model proves no path. |
| Analyzer output without evidence refs becomes GoldFact. | Fact emission requires evidence lineage and authority. | Reject. |

## 9. Future Cadastre reachability architecture

### 9.1 Contract summary

| Contract | Purpose | Inputs | Outputs | Required fields | Default values | Bounds | Error behavior | Source lineage | Version-manifest requirements | Authority boundary | Graph effect |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `NetworkContextSnapshot` | Immutable modeled network context for one analysis | Topology, routes, policies, NAT, LB, workload, endpoint, identity context refs | Snapshot ID and checksums | `snapshot_id`, `valid_time`, `known_time`, dataset refs, unsupported refs | none | Snapshot must be immutable | Missing ref -> `SNAPSHOT_INCOMPLETE` | Every input ref retained | Include all dataset refs and hashes | Not truth by itself | `none` |
| `ReachabilityModelCapabilityMatrix` | Declares what solver can represent | Solver version, feature list | Capability decision | Supported protocols, resources, policy types, NAT, LB, identity, unsupported handling | unsupported -> `unsupported` | Closed feature enum | Unsupported -> `unsupported` | Capability artifact ref | Solver/version/hash required | Capability not evidence | `none` |
| `ReachabilitySourceDatasetProfile` | Defines dataset shape for reachability inputs | Source category/dataset | Dataset profile | Dataset type, schema, scope, time, evidence classes | no default authority | Must declare source category | Invalid -> `DATASET_PROFILE_ERROR` | Source dataset refs | Profile version required | Dataset is evidence input only | `none` |
| `ReachabilitySourceCompletenessProfile` | Defines completeness for each evidence class | Receipts, evidence rows | Completeness decisions | Evidence class, scope, valid/known time, allowed claims | unsafe -> no claim | Must cover each required class | Gap -> `unknown_partial` | Receipt/evidence refs | Profile and decision refs | Completeness not source authority | `none` |
| `ReachabilityQuery` | User/API request | Selectors, claim kind, protocol, port, time, constraints | Query hash | Required request fields in Section 10 | `max_paths=10`, `timeout=30`, `graph_effect=none` | `max_paths 0..100`, timeout `1..300` | Invalid -> `VALIDATION_ERROR` | Caller/query refs | Query hash required | Query is not claim | `none` |
| `ReachabilityResult` | Analysis result | Query, snapshot, completeness, capability, authority | Result state and artifacts | Fields in Section 10 | `graph_effect=none`, `paths=[]` | Path count <= `max_paths` | Timeout -> `unknown_partial` with timeout code | Evidence refs when requested | Result hash required | Not fact unless policy permits | `none` |
| `ReachabilityPathStep` | One modeled path hop/decision | Solver trace | Step rows | component, action, input/output packet, evidence refs | none | Ordered index required | Unsupported step -> result `unsupported` | Component evidence refs | Included in result hash | Step not fact | `none` |
| `ReachabilityEvidenceRow` | Normalized evidence supporting analysis | Raw/silver/artifact refs | Evidence row | evidence class, source, scope, time, authority limit | authority limit = evidence_only | Must name class | Missing -> `EVIDENCE_REF_MISSING` | Required | Hash required | Evidence row not authority unless profiled | `none` |
| `ReachabilityClaimPolicy` | Controls whether result may emit future facts/edges | Result, authority, completeness, edge semantics | Allow/deny decision | claim kind, permitted states, evidence requirements, graph permission | deny | Closed allowed states | Deny -> no fact/edge | Decision refs | Policy version required | Sole fact/edge gate | `none` unless explicitly permitted |
| `ReachabilityAnalysisArtifact` | Persisted analysis output | Query/result/path/evidence | Artifact ID | query hash, result hash, solver, snapshot, limitations | none | Immutable | Hash mismatch -> reject | Full lineage | Required for replay | Analysis only | `none` |
| `ReachabilityValidationScenario` | Test fixture | Given context/query | Expected result/errors | scenario ID, inputs, expected state, expected diagnostics | none | Deterministic expected output | Mismatch -> fail | Fixture refs | Scenario checksum | Validation only | `none` |

### 9.2 Contract relationships

```text
ReachabilityQuery
  -> NetworkContextSnapshot
  -> ReachabilitySourceCompletenessProfile decisions
  -> ReachabilityModelCapabilityMatrix
  -> SourceAuthorityProfile
  -> EvaluateReachability(...)
  -> ReachabilityResult
  -> ReachabilityAnalysisArtifact
  -> ReachabilityClaimPolicy
  -> optional future GoldFact/GraphEdge only when policy permits
```

No proposed contract may bypass raw, silver, source completeness, source authority, or version manifest requirements.

## 10. Reachability query and result contract

### 10.1 Interface

```text
EvaluateReachability(request) -> ReachabilityResult
```

### 10.2 Request contract

| Field | Type | Required | Default | Validation |
| --- | ---: | ---: | --- | --- |
| `source_selector` | object | Yes | none | Must resolve to one or more scoped endpoints or identities. |
| `destination_selector` | object | Yes | none | Must resolve to one or more scoped endpoints, services, or resources. |
| `claim_kind` | enum | Yes | none | `packet_reachability`, `session_reachability`, `service_access`, `identity_conditioned_service_access`. |
| `protocol` | enum | Yes for packet/session/service | none | `tcp`, `udp`, `icmp`, `sctp`, `other_declared`. |
| `destination_port` | integer or null | Required for TCP/UDP/SCTP service claims | null | `1..65535`; null only when protocol does not use ports. |
| `source_port` | integer or null | No | null | `1..65535` when supplied. |
| `direction` | enum | Yes | `source_to_destination` | `source_to_destination`, `destination_to_source`, `bidirectional`. |
| `as_of_valid_time` | timestamp | Yes | none | RFC3339 UTC. |
| `as_of_known_time` | timestamp | Yes | none | RFC3339 UTC. |
| `condition_context` | object | No | `{}` | Required for identity-conditioned access. |
| `path_constraints` | object | No | `{}` | Must use declared constraint keys only. |
| `max_paths` | integer | No | `10` | `0..100`; `0` means no path details, result only. |
| `timeout_seconds` | integer | No | `30` | `1..300`. |
| `include_evidence` | boolean | No | `false` | If true, include evidence refs. |
| `include_unsupported` | boolean | No | `true` | If true, include unsupported components. |

### 10.3 Response contract

| Field | Type | Required | Default | Validation |
| --- | ---: | ---: | --- | --- |
| `result_state` | enum | Yes | none | One of `reachable`, `not_reachable`, `conditional`, `unknown_partial`, `unsupported`, `conflicted`. |
| `claim_kind` | enum | Yes | copied | Must match request. |
| `query_hash` | sha256_hex | Yes | none | Hash of canonical request. |
| `network_context_snapshot_id` | id | Yes | none | Must reference snapshot. |
| `model_capability_matrix_id` | id | Yes | none | Must reference solver capabilities. |
| `source_completeness_decision_refs` | array[id] | Yes | `[]` | Must include every required completeness decision. |
| `source_authority_profile_refs` | array[id] | Yes | `[]` | Must include authority refs used. |
| `path_enumeration_mode` | enum | Yes | `none` | `none`, `complete`, `representative`, `bounded_subset`, `unknown`. |
| `paths` | array[`ReachabilityPathStep[]`] | Yes | `[]` | Count <= `max_paths`. |
| `blocking_components` | array[object] | Yes | `[]` | Required for `not_reachable` when known. |
| `unsupported_components` | array[object] | Yes | `[]` | Required for `unsupported` or when `include_unsupported=true`. |
| `required_conditions` | array[object] | Yes | `[]` | Required for `conditional`. |
| `evidence_refs` | array[id] | Yes | `[]` | Required for fact-emitting policy; optional in analysis. |
| `user_facing_summary` | string | Yes | none | Must follow result-state wording rules. |
| `graph_effect` | enum | Yes | `none` | `none`, `eligible_future_fact`, `eligible_future_edge`; default and MVP must be `none`. |

### 10.4 Query hash canonicalization

```text
CanonicalReachabilityQueryHash(request):
  1. Normalize Unicode strings to NFC.
  2. Canonicalize timestamps to UTC RFC3339 with 9 fractional digits and trailing Z.
  3. Sort object keys lexically at every depth.
  4. Sort arrays only when the field contract declares unordered semantics.
  5. Preserve array order for path_constraints that declare ordered semantics.
  6. Serialize as canonical JSON with no insignificant whitespace.
  7. SHA-256 hash the UTF-8 bytes.
```

## 11. Deterministic evaluation and replay algorithm

```text
ReachabilityResult = EvaluateReachability(
  query,
  network_context_snapshot,
  completeness_decision_set,
  model_capability_matrix,
  source_authority_profile
)
```

### 11.1 Validation order

```text
1. Validate request schema.
2. Canonicalize request and compute query_hash.
3. Validate claim_kind-specific required fields.
4. Validate source_selector and destination_selector.
5. Validate as_of_valid_time and as_of_known_time.
6. Resolve NetworkContextSnapshot for the requested times.
7. Validate VersionManifest references for every snapshot input.
8. Validate source completeness decisions for every required evidence class.
9. Validate source authority profiles for every required evidence class.
10. Validate model capability for every component, protocol, route, policy, translation, workload, identity, and solver feature.
11. Evaluate topology.
12. Evaluate route and forwarding.
13. Evaluate firewall/policy in deterministic precedence order.
14. Evaluate NAT, proxy, and translation.
15. Evaluate load balancer, service, and backend context when claim kind requires it.
16. Evaluate endpoint listener, host firewall, workload policy, and service mesh context when required.
17. Evaluate identity, posture, PDP/PEP, and session authorization when required.
18. Produce result_state.
19. Order paths deterministically.
20. Emit result hash and analysis artifact.
21. Apply ReachabilityClaimPolicy only after analysis completes.
```

### 11.2 Evaluation order for route, policy, NAT, workload, and identity

| Order | Evaluation domain | Required behavior |
| ---: | --- | --- |
| 1 | Endpoint selector resolution | Reject ambiguous selectors unless query permits fan-out; unresolved selectors -> `unknown_partial`. |
| 2 | Packet selector | Missing protocol/port for relevant claim -> `VALIDATION_ERROR`. |
| 3 | Topology | Missing topology for any candidate path -> `unknown_partial`. |
| 4 | Route | Evaluate route precedence, next hop, and route validity; missing route evidence -> `unknown_partial`. |
| 5 | Policy | Evaluate ordered/priority rules; deny precedence before allow where source semantics require it. |
| 6 | NAT/translation | Apply declared translation order; unsupported translation -> `unsupported`. |
| 7 | Load balancer/proxy | Evaluate listener, target selection, backend set, proxy termination, and source/destination rewrite. |
| 8 | Workload policy | Resolve namespace/pod/service selectors and evaluate ingress plus egress. |
| 9 | Endpoint/service | Evaluate listener and host/workload endpoint context. |
| 10 | Identity/access | Evaluate subject, device posture, PDP/PEP, session authorization, and conditions. |

### 11.3 Negative-claim requirements

A `not_reachable` result may be emitted only when:

```text
all required evidence classes for the claim kind are present;
all required completeness decisions are complete for the query scope;
source authority permits negative claims for the evidence class;
model capability covers every relevant component;
no unsupported component can affect the result;
no permission gap can affect the result;
no stale input can affect the result;
path_enumeration_mode is complete or the solver proves absence without enumeration.
```

Otherwise the result must be `unknown_partial`, `unsupported`, or `conflicted`.

### 11.4 Unsupported-feature handling

| Condition | Required state | Required diagnostic |
| --- | --- | --- |
| Relevant resource type unsupported | `unsupported` | `UNSUPPORTED_RESOURCE_TYPE` |
| Relevant policy type unsupported | `unsupported` | `UNSUPPORTED_POLICY_TYPE` |
| Relevant NAT/proxy behavior unsupported | `unsupported` | `UNSUPPORTED_TRANSLATION_BEHAVIOR` |
| Relevant workload policy unsupported | `unsupported` | `UNSUPPORTED_WORKLOAD_POLICY` |
| Relevant identity/access policy unsupported | `unsupported` | `UNSUPPORTED_IDENTITY_POLICY` |
| Unsupported feature not on any possible path | Continue only if solver proves irrelevance | `UNSUPPORTED_COMPONENT_IRRELEVANT` |

### 11.5 Partial-result handling

If the solver returns representative paths rather than complete enumeration, the result must set:

```text
path_enumeration_mode = representative
```

The user-facing summary must not use “all paths,” “complete reachability,” “no other paths,” or equivalent language.

### 11.6 Path ordering

```text
Order paths by:
  1. shortest modeled path length
  2. lowest count of conditional steps
  3. lowest count of unsupported-but-irrelevant annotations
  4. highest minimum source authority across path evidence, descending
  5. highest minimum completeness confidence across path evidence, descending
  6. lexical sequence of component IDs
  7. lexical sequence of normalized packet transformations
```

### 11.7 Snapshot and replay hashing

```text
SnapshotHash(snapshot):
  1. Collect all input dataset refs.
  2. For each ref, use table-format-native snapshot or artifact checksum.
  3. Include source dataset profile IDs and versions.
  4. Include completeness profile IDs and decision IDs.
  5. Include source authority profile IDs and versions.
  6. Include solver capability matrix ID and solver version.
  7. Sort rows lexically by dataset role, source instance ID, dataset ID, and ref ID.
  8. Serialize canonical JSON.
  9. SHA-256 hash bytes.
```

Replay must reject with `VERSION_MANIFEST_MISMATCH` or a more specific reachability error before output when any hash, profile version, capability matrix, completeness decision, source authority profile, or snapshot ref differs.

### 11.8 Timeout behavior

| Timeout point | Required behavior |
| --- | --- |
| Before solver starts | Return `TIMEOUT_BEFORE_EVALUATION` and `unknown_partial`. |
| During evidence validation | Return `TIMEOUT_DURING_VALIDATION` and `unknown_partial`; no partial path claims. |
| During solver execution with no safe partial | Return `TIMEOUT_DURING_SOLVER` and `unknown_partial`. |
| During solver execution with representative partial enabled | Return `unknown_partial`, include `paths` only as diagnostics, and set `path_enumeration_mode = representative`. |
| During claim policy evaluation | Return analysis result but prohibit GoldFact/edge emission. |

### 11.9 Error codes

| Error code | Meaning |
| --- | --- |
| `VALIDATION_ERROR` | Request schema or field validation failed. |
| `REACHABILITY_QUERY_MISSING_PROTOCOL` | Protocol required but absent. |
| `REACHABILITY_QUERY_MISSING_PORT` | Destination port required but absent. |
| `SNAPSHOT_INCOMPLETE` | Required snapshot input missing. |
| `COMPLETENESS_GAP` | Required completeness decision missing or unsafe. |
| `SOURCE_AUTHORITY_GAP` | Required authority profile missing or does not permit claim. |
| `UNSUPPORTED_RESOURCE_TYPE` | Solver cannot represent resource type. |
| `UNSUPPORTED_POLICY_TYPE` | Solver cannot represent policy type. |
| `UNSUPPORTED_TRANSLATION_BEHAVIOR` | NAT/proxy/load-balancer behavior unsupported. |
| `UNSUPPORTED_IDENTITY_POLICY` | Identity/access policy unsupported. |
| `EVIDENCE_CONFLICT` | Evidence sources conflict. |
| `VERSION_MANIFEST_MISMATCH` | Replay/version refs mismatch. |
| `TIMEOUT_DURING_SOLVER` | Solver timed out. |
| `CLAIM_POLICY_DENIED` | Analysis result cannot emit fact/edge. |
| `THEORETICAL_REACHABILITY_SCOPE_ERROR` | MVP or inactive profile attempted graph/fact reachability emission. |

## 12. Graph semantics changes

### 12.1 Why `has_theoretical_reachability` must not reuse `observed_connection`

`observed_connection` and `has_theoretical_reachability` have different evidence, semantics, time models, direction rules, confidence policies, and non-implications. Reusing `observed_connection` would collapse observed evidence with modeled evidence and violate the PRD’s explicit boundary that observed traffic does not prove complete theoretical reachability.[^2][^5]

### 12.2 Graph-edge semantics matrix for `observed_connection` and future `has_theoretical_reachability`

| Edge type | Source fact type | Source predicate | Direction rule | Evidence requirements | Temporal policy | Confidence policy | Traversal eligibility | Non-implication rules | No-op behavior | MVP behavior | Future behavior |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `observed_connection` | `observed_network_flow_fact` | `observed_connection` | From `FlowRoleEvidence`; if undirected allowed, lexical pair with `direction_state=ambiguous`; otherwise no-op ambiguous | Flow observation, flow role evidence, source authority for observed traffic | Valid/known interval from observed evidence | Derived from flow source and evidence confidence | `network_observation_only` | Does not imply theoretical reachability, service access, allowed policy, route existence, endpoint compromise, or identity access | No-op when direction cannot be determined and undirected not permitted | Allowed in MVP under closed edge table | Continue as observed-only edge |
| `has_theoretical_reachability` | Future `modeled_reachability_fact` | Future `has_theoretical_reachability` | From solver result and query direction; bidirectional/session claims require return or stateful-return proof | Complete evidence classes in Section 7, completeness decisions, authority profile, solver capability, evidence lineage | Snapshot valid/known interval plus claim conditions | From source authority and model confidence policy | `network_modeled_reachability` only if future profile permits | Does not imply observed traffic, service access, identity-conditioned access, exploitability, or lateral movement by itself | No-op or validation failure when any evidence/capability incomplete | Forbidden; validation fails | Future only through accepted PRD/NLSpec and active claim policy |

### 12.3 Proposed `traversal_eligibility` enum

| Enum value | Meaning | Allowed for `observed_connection` | Allowed for future `has_theoretical_reachability` |
| --- | --- | ---: | ---: |
| `none` | Edge must not be used in pathfinding. | Yes | Yes by default |
| `network_observation_only` | Edge may be traversed only for observed-flow exploration, not reachability or attack paths. | Yes | No |
| `network_modeled_reachability` | Edge may be traversed for modeled network reachability views. | No | Future only |
| `identity_attack_path` | Edge may be traversed for identity attack-path analysis. | No | No |
| `exposure_analysis` | Edge may be traversed for declared exposure rules. | Future conditional | Future conditional |
| `service_access_analysis` | Edge may be traversed for service access analysis with listener/backend context. | No | Future conditional |
| `identity_conditioned_access` | Edge may be traversed for zero-trust/identity-conditioned access. | No | Future conditional |

## 13. Source completeness and source authority model

### 13.1 Completeness rules

Reachability completeness must be evaluated per evidence class, source scope, valid time, known time, and claim kind. A complete firewall-policy dataset does not make route evidence complete. A complete route dataset does not make NAT complete. A provider analyzer result does not make endpoint access complete.

### 13.2 Source authority rules

| Source category | May support | Must not support by itself |
| --- | --- | --- |
| `cloud_network_config` | Cloud topology, routes, SG/NACL/NSG, load-balancer config | Endpoint listener, identity-conditioned access, non-cloud appliances |
| `firewall_policy` | Firewall rule evaluation and policy action | Route existence, service access, identity access |
| `routing_control_plane` | Next hop, route selection, routing domains | Firewall allow, service access |
| `nat_translation_policy` | Address/port translation, proxy termination | Endpoint listener, identity access |
| `load_balancer_proxy_policy` | Listener/backend/proxy path conditions | Backend health unless collected, identity access |
| `endpoint_access_policy` | Host firewall/listener/endpoint policy | Network route unless collected |
| `orchestration_network_policy` | Pod/workload ingress/egress policy | CNI enforcement unless collected, listener access |
| `identity_access_policy` | Subject/resource/device posture access | Network path unless collected |
| `reachability_analyzer_output` | Scoped analysis artifact | Direct GoldFact or graph edge |

### 13.3 Source completeness decision states

| State | Meaning | Reachability behavior |
| --- | --- | --- |
| `complete` | Source scope fully evaluated for declared dataset/time | May support positive and negative claims if authority/capability also complete |
| `empty_complete` | Source scope fully evaluated and empty | May support negative only when source contract defines meaningful absence |
| `partial_known_gap` | Known missing scopes/pages/resources | `unknown_partial` |
| `partial_unknown_gap` | Missing unknown scope/pages/resources | `unknown_partial` |
| `source_unavailable` | Source unavailable | `unknown_partial` |
| `scope_unavailable` | Scope unavailable or permission-limited | `unknown_partial` |
| `not_authoritative_for_absence` | Presence-only source | Positive only when evidence supports; negative prohibited |
| `not_attempted` | Collection not attempted | `unknown_partial` |
| `not_applicable` | Evidence class not applicable to claim | Omit from requirement only if claim contract permits |

## 14. Workload, Kubernetes, Cilium, and zero-trust access model

### 14.1 Kubernetes/Cilium policy semantics matrix

| Semantics case | Kubernetes NetworkPolicy | Cilium | Cadastre requirement |
| --- | --- | --- | --- |
| No policy selects pod | Pod is non-isolated for that direction by default | Depends on enforcement mode; default unrestricted until selected, `always` differs | Must capture enforcement mode/CNI before positive claim |
| Ingress isolation | Any selecting ingress policy isolates ingress | Cilium endpoint selected by ingress policy enters default deny for ingress | Evaluate destination ingress |
| Egress isolation | Any selecting egress policy isolates egress | Cilium endpoint selected by egress policy enters default deny for egress | Evaluate source egress |
| Additive allow | Policies are additive | Union of allow rules | Merge allows after denies/precedence |
| Deny precedence | Native NetworkPolicy has allow-only model | Cilium deny policies take precedence over allow | Apply Cilium deny first |
| Pod selector | Label-selected pods | Endpoint selectors and identities | Resolve selector snapshot |
| Namespace selector | Namespace labels | Namespace/endpoint labels | Resolve namespace snapshot |
| IPBlock | External CIDR with rewrite caveat | CIDR/IP policies, DNS/FQDN policies | Mark rewrite unknown unless behavior known |
| L7 HTTP/DNS/FQDN | Not native | Supported with proxy/DNS caveats | Required for service/FQDN claims |
| Version/CNI dependency | Requires supporting plugin | Requires Cilium version/mode | Capability matrix required |

### 14.2 Zero-trust access-condition matrix

| Condition | Required content | Required for claim kind | Missing behavior |
| --- | --- | --- | --- |
| Subject identity | User/service account/principal ID, scope, groups/roles | Identity-conditioned service access | `unknown_partial` |
| Device identity | Device ID, posture, compliance state | Identity-conditioned service access | `conditional` or `unknown_partial` |
| Resource identity | Application/resource/service ID | Service and identity-conditioned access | `unknown_partial` |
| PDP policy | Policy version, conditions, decision logic | Identity-conditioned service access | `unknown_partial` |
| PEP enforcement | Enforcement point identity and scope | Identity-conditioned service access | `unknown_partial` |
| Session context | Session issuance, duration, re-auth, revocation | Identity-conditioned service access | `conditional` |
| Time condition | Valid time, policy time windows | Conditional access | `conditional` |
| Network condition | Network location or route condition | Conditional access | Must not become implicit trust |

### 14.3 Required workload resolution

A future reachability analysis must resolve:

```text
pod selector resolution
namespace selector resolution
service account and workload identity
ingress isolation
egress isolation
CNI capability and version
Cilium deny-before-allow precedence
Cilium L7 HTTP/DNS/FQDN behavior
Service and EndpointSlice resolution
source/destination rewrite behavior
NodePort, Service, ingress, egress gateway, and proxy behavior
PDP/PEP access control
identity and device posture conditions
session authorization
```

Subnet or IP reachability alone must not satisfy workload or application access.

## 15. Cloud-provider analyzer boundary model

Provider-native analyzer output must be stored as scoped evidence or `ReachabilityAnalysisArtifact`, not direct Cadastre truth.

```text
provider analyzer output
  -> RawRecord
  -> parsed analyzer artifact
  -> ReachabilityAnalysisArtifact
  -> optional evidence row
  -> EvaluateReachability may consume as supporting scoped evidence
  -> GoldFact/graph edge prohibited unless ReachabilityClaimPolicy permits
```

### 15.1 Provider analyzer boundary matrix

| Provider | Analysis mode | Query shape | Resource scope | Supported resources | Unsupported resources | Permission requirements | Snapshot/model time | Path output | Blocking output | Live probe support | Negative-result meaning | Result limitation metadata | Cadastre evidence class | Cadastre non-transfer rule |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AWS Reachability Analyzer | Configuration analysis, no packets | Source, destination, protocol/port, optional path | Same region and supported VPC connectivity; account/org limits | EC2, ENI, SG, NACL, routes, LB, NAT, TGW, VPN, Network Firewall subsets | IPv6, TGW Connect, some Network Firewall rules, target health, traffic mirroring, advanced cases | EC2/VPC analysis permissions | AWS configuration at analysis time | Hop-by-hop virtual path | Blocking components/explanations | No | No modeled path or blocking component inside scope | Unsupported resource/config docs and explanations | Provider analysis artifact | Must not become GoldFact directly |
| AWS Network Access Analyzer | Static automated reasoning, no packets | Access scope | Account/region/path length bounds | Many VPC access paths | API Gateway, Global Accelerator, Traffic Mirroring, Wavelength, Direct Connect, EC2 forwarding appliances, unsupported GWLB/NFW cases | IAM permissions plus service data | AWS configuration at analysis time | Findings, representative by design | Access findings/violations | No | Scoped finding absence only, not universal negative | Unsupported and path-limit docs | Provider analysis artifact | Representative findings must not imply exhaustive enumeration |
| Google Connectivity Tests | Configuration analysis and optional live data-plane analysis | Source, destination, protocol/port | GCP resources supported by Network Intelligence Center | VPC, routes, firewall policies, NAT except NAT64, LBs with caveats | Many live-analysis gaps: GKE pods, some LBs, VPN, NAT64, non-running VMs | Project/network permissions; gaps may appear in result | Config state and verified time | Traces, sometimes multiple | Unreachable/ambiguous/undetermined states | Yes, scoped | `Unreachable` only within represented scope; `Undetermined` for gaps | Result state, trace details, permission gaps | Provider analysis/probe artifact | `Deliver` must not be worded as guaranteed delivery |
| Azure Network Watcher/VNM | Diagnostic decomposition | Tool-specific packet/routing/security/probe inputs | Azure VM/NIC/VNet/resource scopes | NSG/admin rules, next hop, effective rules, connection probes | Excluded services/resources, protocol bounds, preview agentless mode | Network Watcher/resource permissions | Config and diagnostic time | Next hop/topology/probe results | Allow/deny/rule, UDR/NSG issues | Connection Troubleshoot probes | Tool-specific, not complete negative | Rule names, route table, probe count, preview, eventual consistency | Diagnostic evidence row | IP Flow Verify/Next Hop must not imply reachability |

### 15.2 Required provider analyzer metadata

| Metadata field | Required behavior |
| --- | --- |
| `provider` | `aws`, `gcp`, or `azure`. |
| `analysis_tool` | Exact provider tool name. |
| `analysis_mode` | `configuration`, `live_probe`, `static_reasoning`, `diagnostic`, or `combined`. |
| `provider_scope` | Account/project/subscription, region, VPC/VNet/network, resource scope. |
| `query_shape` | Source, destination, protocol, port, direction. |
| `supported_resource_manifest` | Tool-supported resources/features at analysis time. |
| `unsupported_components` | Components affecting result or declared irrelevant. |
| `permission_context` | Permissions available and gaps. |
| `analysis_time` | Provider analysis timestamp. |
| `path_output_mode` | `none`, `single_shortest`, `representative`, `multiple_backend`, `complete`, `unknown`. |
| `negative_result_scope` | Exact scope where negative result is meaningful. |
| `raw_output_ref` | Evidence lineage to source artifact. |

## 16. Validation matrix and fixtures

### 16.1 Model capability matrix with unsupported-component behavior

| Component class | Capability value | Required behavior when unsupported |
| --- | --- | --- |
| IPv4 routing | `supported`, `partial`, `unsupported` | `unsupported` if relevant; otherwise annotate irrelevant |
| IPv6 routing | `supported`, `partial`, `unsupported` | Reject IPv6 query if unsupported |
| Security groups / NSGs / NACLs | `supported`, `partial`, `unsupported` | `unsupported` for policy-dependent claim |
| Ordered ACLs | `supported`, `partial`, `unsupported` | `unsupported` unless all matched fields/actions supported |
| Stateful firewall | `supported`, `partial`, `unsupported` | Session claim prohibited if unsupported |
| NAT | `supported`, `partial`, `unsupported` | `unsupported` when translation may affect path |
| Load balancer | `supported`, `partial`, `unsupported` | Service claim `unknown_partial` or `unsupported` |
| Network virtual appliance | `supported`, `partial`, `unsupported` | `unsupported` unless appliance model declared |
| Kubernetes NetworkPolicy | `supported`, `partial`, `unsupported` | Workload claim prohibited if unsupported |
| Cilium L7/FQDN | `supported`, `partial`, `unsupported` | Service/FQDN claim conditional or unsupported |
| Identity/ZTA policy | `supported`, `partial`, `unsupported` | Identity-conditioned access prohibited |
| Live probe | `diagnostic_only` | Must not emit theoretical reachability |

### 16.2 Validation fixture matrix

| Fixture ID | Scenario | Given | Expected result |
| --- | --- | --- | --- |
| `VR-001` | MVP reachability edge rejection | Graph projection attempts `has_theoretical_reachability` | Fail with `THEORETICAL_REACHABILITY_SCOPE_ERROR` |
| `VR-002` | Observed flow non-implication | Flow log exists only | `observed_connection` allowed; reachability query `unknown_partial` |
| `VR-003` | Missing protocol | Query omits protocol | `REACHABILITY_QUERY_MISSING_PROTOCOL` |
| `VR-004` | Missing destination port | TCP service claim omits port | `REACHABILITY_QUERY_MISSING_PORT` |
| `VR-005` | Missing route evidence | Topology and firewall exist, routes missing | `unknown_partial` |
| `VR-006` | Missing firewall evidence | Routes exist, policy missing | `unknown_partial` |
| `VR-007` | Missing NAT/proxy evidence | Route crosses NAT but NAT state absent | `unknown_partial` |
| `VR-008` | Unsupported NVA | Path crosses unmodeled appliance | `unsupported` |
| `VR-009` | One-way path session claim | Forward path only | `unknown_partial` for session |
| `VR-010` | Kubernetes ingress only | Destination ingress allows, source egress isolated/no allow | `not_reachable` only if egress completeness complete; otherwise `unknown_partial` |
| `VR-011` | Cilium deny overrides allow | Matching allow and deny | Deny wins; path blocked |
| `VR-012` | Identity claim without identity context | Network/service complete, identity missing | `unknown_partial` |
| `VR-013` | AWS positive outside scope | AWS reachable for one VPC, query asks enterprise scope | Provider artifact stored; enterprise claim rejected |
| `VR-014` | AWS negative with unsupported resource | Analyzer negative plus unsupported NVA | `unsupported` or `unknown_partial`, no negative fact |
| `VR-015` | GCP deliver wording | Config result `Deliver` | Summary says modeled config could deliver, not guaranteed delivery |
| `VR-016` | Azure IP Flow Verify allow | Policy allow, route missing | `unknown_partial` |
| `VR-017` | Stale completeness receipt | Receipt expired | `unknown_partial` |
| `VR-018` | Deterministic path ordering | Multiple equal paths | Lexical component sequence tie-break |
| `VR-019` | Missing model capability metadata | Solver lacks matrix | `MODEL_CAPABILITY_MISSING` |
| `VR-020` | Fact-emitting result without evidence refs | Claim policy requests fact, no evidence refs | `CLAIM_POLICY_DENIED` |
| `VR-021` | Version manifest replay mismatch | Snapshot hash differs | `VERSION_MANIFEST_MISMATCH` |
| `VR-022` | Structural alias | `AnyHost` used as universal endpoint | Reject structural alias claim |
| `VR-023` | Live probe success | Probe succeeds but model incomplete | Diagnostic only; reachability `unknown_partial` |
| `VR-024` | Live probe failure | Probe fails but alternate modeled path exists unknown | No negative claim |

## 17. PRD improvement map

| PRD improvement | Current PRD state | Source basis | Recommended contract change | Why it strengthens Cadastre | Why out of MVP or analysis-only | Required validation scenario | Acceptance criterion |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Future `ReachabilityAnalysis` domain | PRD declares future theoretical reachability but not full future domain | Batfish, cloud analyzers, GCP/Azure diagnostics | Add future non-MVP domain section with analysis-only default | Prevents ad hoc reachability facts | Out of MVP by PRD | `VR-001` | MVP emits no reachability edges |
| `NetworkContextSnapshot` | Not defined | Batfish snapshots; cloud config analyzers | Add immutable snapshot contract | Makes model inputs replayable | Future only | `VR-021` | Snapshot hash mismatch rejects replay |
| `ReachabilityModelCapabilityMatrix` | Not defined | All analyzer unsupported-feature docs | Add solver capability contract | Prevents unsupported false negatives | Future only | `VR-019` | Missing capability matrix rejects |
| `ReachabilityQuery` | Not defined | Batfish/AWS/GCP query shapes | Add query contract with protocol/port/time/constraints | Prevents ambiguous evaluation | Analysis-only | `VR-003`, `VR-004` | Missing fields fail |
| `ReachabilityResult` | Not defined | GCP states; provider diagnostics | Add closed result-state vocabulary | Prevents boolean collapse | Analysis-only | All state fixtures | Each fixture maps to closed state |
| `ReachabilityPathStep` | Not defined | Batfish traces, AWS/GCP paths, Azure topology | Add path-step schema | Enables path evidence without overclaiming | Analysis-only | `VR-018` | Paths deterministic |
| `ReachabilityEvidenceRow` | Not defined | Source adapter research | Add evidence row with authority limits | Prevents adapter truth leakage | Future only | `VR-020` | Fact requires evidence refs |
| `ReachabilityClaimPolicy` | Not defined | PRD SourceAuthorityProfile and GraphEdgeSemantics | Add sole future fact/edge gate | Separates analysis from truth | Future only | `VR-020` | Denied claim emits no fact |
| `ReachabilitySourceDatasetProfile` | Not defined | Source-adapter research | Add dataset input profile | Makes evidence classes explicit | Future only | `VR-005` | Missing dataset -> unknown |
| `ReachabilitySourceCompletenessProfile` | Source completeness exists, not reachability-specific | RES-006, PRD completeness | Add evidence-class completeness rows | Enables scoped negatives only | Future only | `VR-017` | Stale receipt -> unknown |
| `GraphEdgeSemantics.traversal_eligibility` | Not present in PRD fields | BloodHound/OpenGraph | Add closed enum | Stops edge existence from implying path eligibility | Can be added now for non-reachability graph discipline | `VR-002` | Observed edge not reachability-traversable |
| Future `has_theoretical_reachability` row | Explicitly forbidden in MVP | PRD baseline, all source families | Add placeholder future row with `status=future_prohibited` | Makes rejection deterministic | Out of MVP | `VR-001` | Projection fails |
| Source category `cloud_network_config` | Source categories implied | AWS/GCP/Azure | Add category for cloud network config evidence | Improves source authority mapping | Collection only in MVP if no reachability emission | `VR-013` | Provider scope retained |
| Source category `firewall_policy` | Not reachability-specific | OpenConfig/Azure/AWS | Add category | Policy evidence separated from reachability | Future claim only | `VR-006` | Missing policy -> unknown |
| Source category `routing_control_plane` | Not reachability-specific | Batfish/Azure/GCP | Add category | Route evidence separated from policy | Future claim only | `VR-005` | Missing route -> unknown |
| Source category `nat_translation_policy` | Not reachability-specific | AWS/GCP/Azure | Add category | Translation prevents false positives | Future claim only | `VR-007` | Missing NAT -> unknown |
| Source category `load_balancer_proxy_policy` | Not reachability-specific | AWS/GCP/Azure/K8s | Add category | Differentiates service access from packet path | Future/service claim only | `VR-015` | LB result not delivery guarantee |
| Source category `endpoint_access_policy` | Not reachability-specific | NIST/Cilium/endpoint sources | Add category | Enables service-access boundary | Future only | `VR-012` | Missing endpoint policy -> unknown |
| Source category `orchestration_network_policy` | Not reachability-specific | Kubernetes/Cilium | Add category | Workload policy captured explicitly | Future only | `VR-010`, `VR-011` | Ingress+egress/deny precedence pass |
| Source category `identity_access_policy` | Not reachability-specific | NIST ZTA | Add category | Prevents network reachability from implying access | Future only | `VR-012` | Missing identity -> unknown |
| Source category `reachability_analyzer_output` | Not defined | AWS/GCP/Azure | Add analysis-artifact source category | Keeps provider output scoped | Analysis-only | `VR-013`, `VR-014` | No direct GoldFact |

## 18. Concepts that must not transfer

| Concept | Source family | Why unsafe | Safer Cadastre alternative |
| --- | --- | --- | --- |
| Example path as complete path set | Batfish, GCP, AWS NAA | Representative outputs do not prove exhaustiveness | `path_enumeration_mode = representative` |
| Provider positive as enterprise-wide reachability | AWS/GCP/Azure | Provider scope and supported-resource limits | Scoped analysis artifact |
| Provider negative as universal absence | AWS/GCP/Azure | Unsupported resources and permission gaps | `unknown_partial` or `unsupported` |
| Firewall/security group allow as service access | AWS/Azure/OpenConfig | Route/NAT/listener/identity absent | Require service-access evidence classes |
| Route/next-hop as reachability | Azure/Batfish/cloud | Policy/NAT/service absent | Route evidence only |
| Missing Kubernetes policy as universal allow | Kubernetes/Cilium | CNI/enforcement/mesh/host policy unknown | Require CNI and policy completeness |
| Cilium allow as application access | Cilium | Deny/L7/listener/backend may block | Evaluate deny and L7 context |
| Network location as access | NIST ZTA | No implicit trust | Identity-conditioned access claim |
| Graph traversability as network reachability | BloodHound/OpenGraph | Identity path semantics differ | Separate traversal eligibility |
| Adapter live probe as truth | Source-adapter systems | Probe is non-authoritative | Diagnostic artifact |
| Structural aliases as universal endpoints | JupiterOne/OpenGraph analogues; PRD boundary | Alias nodes collapse scope | Reject unless bounded structural policy exists |

## 19. Gaps, risks, stale assumptions, and unresolved questions

| Type | Item | Required resolution |
| --- | --- | --- |
| Gap | No external analyzer was executed. | Run controlled fixtures before any PRD normative acceptance beyond research. |
| Gap | Exact source category schemas for routes, NAT, LB, identity policy, and endpoint access are not defined. | Add future source dataset profiles. |
| Gap | No solver selected. | Define solver-agnostic capability matrix before choosing implementation. |
| Gap | No formal policy normalization profile exists. | Add OpenConfig-inspired `NetworkPolicyRuleObservation`. |
| Risk | Provider docs and supported-resource sets change. | Pin provider API/docs version or capture support manifests per run. |
| Risk | Cloud analyzers may silently improve or change findings. | Store analyzer version/support metadata and raw output. |
| Risk | Kubernetes/Cilium behavior is CNI/version-specific. | Require CNI/Cilium version and enforcement mode in snapshot. |
| Risk | L7/FQDN policy depends on dynamic DNS caches and proxies. | Require TTL/cache/proxy evidence and condition output. |
| Risk | Live probes can bias users toward overclaiming. | Enforce non-authoritative live-probe summary wording. |
| Stale assumption | Cilium stable docs observed as 1.19.4. | Recheck before production PRD amendment. |
| Stale assumption | Cloud provider feature support observed 2026-05-16. | Recheck provider docs and APIs before implementation. |
| Open question | Whether Cadastre should implement its own solver or orchestrate Batfish/provider analyzers. | Decide after future solver evaluation. |
| Open question | Whether negative `not_reachable` facts should ever be GoldFacts. | Requires product governance and risk policy. |
| Open question | Whether graph should ever store modeled reachability edges or only analysis artifacts. | Requires future graph product decision. |

## 20. Acceptance criteria

| ID | Criterion | Pass condition | Fail condition |
| --- | --- | --- | --- |
| `AC-TR-001` | Default MVP rejection of `has_theoretical_reachability` | Any default MVP projection attempt fails with `THEORETICAL_REACHABILITY_SCOPE_ERROR` | Any default MVP graph edge or GoldFact is emitted |
| `AC-TR-002` | `observed_connection` non-satisfaction | An observed flow alone cannot satisfy reachability evidence requirements | Observed flow produces modeled reachability |
| `AC-TR-003` | Missing protocol validation | Request missing required protocol returns `REACHABILITY_QUERY_MISSING_PROTOCOL` | Solver executes |
| `AC-TR-004` | Missing port validation | TCP/UDP/SCTP service query missing destination port returns `REACHABILITY_QUERY_MISSING_PORT` | Solver executes |
| `AC-TR-005` | Missing route evidence | Route gap returns `unknown_partial` | Result returns `reachable` or `not_reachable` |
| `AC-TR-006` | Missing firewall/policy evidence | Policy gap returns `unknown_partial` | Result returns `reachable` or `not_reachable` |
| `AC-TR-007` | Missing NAT/proxy/LB evidence | Relevant translation gap returns `unknown_partial` | Result ignores translation gap |
| `AC-TR-008` | Unsupported NVA | Unsupported relevant appliance returns `unsupported` | Negative reachability is emitted |
| `AC-TR-009` | One-way versus bidirectional/session claims | Forward-only evidence cannot satisfy `session_reachability` | Session result returns `reachable` |
| `AC-TR-010` | Kubernetes ingress plus egress | Evaluation requires both destination ingress and source egress when isolated | Ingress-only allow produces connection allow |
| `AC-TR-011` | Cilium deny precedence | Matching deny overrides allow | Allow wins over deny |
| `AC-TR-012` | Identity-conditioned access without identity context | Missing identity context returns `unknown_partial` | Access allowed/denied claim emitted |
| `AC-TR-013` | Provider positive outside supported scope | Provider-positive result outside scope is rejected as enterprise claim | Enterprise reachability emitted |
| `AC-TR-014` | Provider negative with unsupported resources | Negative provider result plus unsupported resource returns `unsupported` or `unknown_partial` | Negative Cadastre fact emitted |
| `AC-TR-015` | Stale or partial completeness | Stale/partial receipt returns `unknown_partial` | Negative claim emitted |
| `AC-TR-016` | Deterministic path ordering | Same inputs produce byte-identical ordered paths | Path order varies |
| `AC-TR-017` | Model capability metadata | Missing capability matrix rejects evaluation | Solver executes without capability matrix |
| `AC-TR-018` | Evidence refs for fact-emitting results | Fact/edge policy requires evidence refs | Fact/edge emitted without evidence refs |
| `AC-TR-019` | Version-manifest replay rejection | Any output-affecting hash mismatch rejects before output | Replay emits output |
| `AC-TR-020` | Graph non-implication rules | `observed_connection` summary says it does not imply theoretical reachability | UI/API implies reachability |
| `AC-TR-021` | Structural alias rejection | `AnyHost`, `AnyNetwork`, `Internet`, `Everyone` cannot imply reachability | Structural alias supports reachability claim |
| `AC-TR-022` | Live probe success non-transfer | Probe success remains diagnostic unless complete model supports claim | Probe success emits reachability |
| `AC-TR-023` | Live probe failure non-transfer | Probe failure does not emit no-reachability unless complete model proves absence | Probe failure emits negative reachability |
| `AC-TR-024` | Representative path wording | Representative output never claims complete path enumeration | User-facing summary says all paths enumerated |
| `AC-TR-025` | Analyzer output without evidence refs | Analyzer output cannot become GoldFact without raw/evidence refs and claim policy | GoldFact emitted directly from analyzer output |

## Sources

[^1]: `PRD-Cadastre.md`, Section 4.1 and source/graph scope requirements, especially lines 180-225 in the uploaded draft. The PRD requires `FlowRoleEvidence`, source completeness controls, graph edge semantics enforcement, and default MVP rejection of `has_theoretical_reachability`.

[^2]: `PRD-Cadastre.md`, Section 4.2 Non-Scope, especially lines 292-320 in the uploaded draft. The PRD prohibits treating observed traffic as complete theoretical reachability, absence of evidence as negative evidence by default, and lateral movement pathing without firewall, routing, NAT, identity privilege, and endpoint access context.

[^3]: `PRD-Cadastre.md`, GoldFact and MVP theoretical reachability rules, especially lines 1761-1795 in the uploaded draft. The PRD states `observed_connection` must reference `FlowRoleEvidence` and that `has_theoretical_reachability` is future non-MVP requiring firewall, route, NAT, zone, endpoint, identity, completeness, authority, lineage, direction, and unknown behavior.

[^4]: `PRD-Cadastre.md`, Document Status, especially lines 49-63 in the uploaded draft. The PRD states Cadastre follows `nlspec-spec.md`, establishes `CadastreSilverObservation` as the authoritative silver contract, and preserves raw-first lakehouse authority for source-adapter research.

[^5]: `PRD-Cadastre.md`, `GraphEdgeSemantics`, especially lines 4308-4378 in the uploaded draft. The PRD defines graph edge semantics fields, the MVP `observed_connection` row, and non-implication of theoretical reachability.

[^6]: `PRD-Cadastre.md`, graph projection enforcement and closed edge table, especially lines 7608-7645 in the uploaded draft. The PRD states `has_theoretical_reachability` is not in the default closed MVP graph edge table and must be rejected before graph delta emission.

[^7]: `PRD-Cadastre.md`, Network Flow Projection Direction, especially lines 7685-7705 in the uploaded draft. The PRD requires projection to use `FlowRoleEvidence` and forbids direction inference from OCSF endpoint positions.

[^8]: `PRD-Cadastre.md`, source authority and future reachability rows, especially lines 7088-7110 in the uploaded draft. The PRD states observed traffic does not prove full reachability and future theoretical reachability is out of MVP.

[^9]: `RES-006-source-adapter-ingestion-patterns.md`, Executive summary and Cadastre baseline, especially lines 17-56 in the uploaded draft. The report states adapters may emit raw records and completeness receipts but not downstream truth, and that completeness receipts are not self-authorizing.

[^10]: `RES-008-identity-graph-attack-paths.md`, Executive verdict and graph semantics findings, especially lines 11-23 and 407-415 in the uploaded draft. The report identifies traversability, generic versus structured graph separation, stable IDs, post-processed edge boundaries, and pathfinding non-authority as transferable concepts.

[^11]: Batfish documentation, “Forwarding and reachability” notebook, inspected 2026-05-16. Used for snapshot-based packet forwarding, reachability questions, traceroute directionality, header/path constraints, and bidirectional reachability semantics.

[^12]: Batfish documentation, “Filters” notebook, inspected 2026-05-16. Used for ACL/filter line analysis, permit/deny behavior, and line-match semantics.

[^13]: Batfish documentation, supported devices and supported features, inspected 2026-05-16. Used for unsupported-feature and partial-support boundary requirements.

[^14]: AWS documentation, VPC Reachability Analyzer overview, inspected 2026-05-16. Used for configuration-analysis, no-packet, hop-by-hop path, and blocking-component semantics.

[^15]: AWS documentation, “How VPC Reachability Analyzer works,” inspected 2026-05-16. Used for supported-resource and target-health/unsupported-resource limitations.

[^16]: AWS documentation, Network Access Analyzer “How Network Access Analyzer works,” inspected 2026-05-16. Used for static analysis, representative findings, unsupported resources, path length, account/region, and target-health limitations.

[^17]: Google Cloud documentation, Connectivity Tests overview and concepts, inspected 2026-05-16. Used for configuration-analysis and live data-plane analysis separation, `Deliver` limitation, supported resources, and live-analysis unsupported cases.

[^18]: Google Cloud documentation, Connectivity Tests state tables, inspected 2026-05-16. Used for `Reachable`, `Unreachable`, `Ambiguous`, and `Undetermined` result-state concepts.

[^19]: Google Cloud documentation, Connectivity Tests load-balancer testing concepts, inspected 2026-05-16. Used for backend trace limits, health-check limitation, and backend omission behavior.

[^20]: Microsoft Learn, Azure Network Watcher IP Flow Verify overview, inspected 2026-05-16. Used for packet-rule allow/deny diagnostic semantics.

[^21]: Microsoft Learn, Azure Network Watcher Next Hop overview, inspected 2026-05-16. Used for next-hop diagnostic semantics.

[^22]: Microsoft Learn, Azure Network Watcher Effective Security Rules overview, inspected 2026-05-16. Used for aggregated NIC security rule evidence.

[^23]: Microsoft Learn, Azure Network Watcher Connection Troubleshoot overview, inspected 2026-05-16. Used for live-probe/diagnostic and preview-boundary behavior.

[^24]: Microsoft Learn, Azure Virtual Network Manager security admin rules concepts, inspected 2026-05-16. Used for admin-rule precedence, `Always Allow`, `Deny`, ordinary `Allow`, excluded resources, and eventual consistency.

[^25]: Kubernetes documentation, Network Policies concept page, inspected 2026-05-16. Used for ingress/egress isolation, additive policy semantics, CNI dependency, selector semantics, IPBlock guidance, and rewrite caveats.

[^26]: Cilium documentation, stable `1.19.4` policy overview, inspected 2026-05-16. Used for Cilium network policy scope and stable-version context.

[^27]: Cilium documentation, policy intro and enforcement modes, inspected 2026-05-16. Used for `default`, `always`, and `never` enforcement mode semantics.

[^28]: Cilium documentation, deny policies, inspected 2026-05-16. Used for deny-policy precedence over allow policy.

[^29]: Cilium documentation, L3 policy, DNS, and FQDN policy sections, inspected 2026-05-16. Used for DNS/FQDN policy and dynamic CIDR behavior.

[^30]: Cilium documentation, L7 policy sections, inspected 2026-05-16. Used for L7 HTTP/DNS/FQDN policy, parser conflict, and proxy requirements.

[^31]: NIST SP 800-207, Zero Trust Architecture, official CSRC publication page, inspected 2026-05-16. Used for no implicit trust from network location or asset ownership and subject/device/resource/session authorization concepts.

[^32]: OpenConfig ACL YANG schema documentation, inspected 2026-05-16. Used for ACL set, ordered entries, sequence ID, match fields, forwarding/log actions, and operational counters.

[^33]: NetKAT official site, inspected 2026-05-16. Used for formal packet-forwarding semantics, reachability, isolation, and equivalence verification concepts.

[^34]: Google NetKAT repository page, inspected 2026-05-16. Used for implementation caveats, no published releases, and lack of official support warranty.

[^35]: BloodHound OpenGraph overview, official documentation, inspected 2026-05-16. Used for generic versus structured graph behavior and OpenGraph node/edge/property model.

[^36]: BloodHound OpenGraph FAQ, official documentation, inspected 2026-05-16. Used for Search versus Pathfinding behavior and generic graph non-traversability.

[^37]: BloodHound traversable edges documentation, inspected 2026-05-16. Used for traversability, pathfinding eligibility, and post-processed edge concepts.
