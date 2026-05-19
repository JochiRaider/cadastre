---
doc_id: RES-016
project: Cadastre
title: Batfish Engineering Research Report
status: research-report
inspection_date: 2026-05-19
---

## 1. Executive Summary

Batfish is a network configuration analysis system. The repository contains a Java-centered analysis engine, a coordinator service, a Java client, question plugins and templates, a symbolic reachability subsystem, a symbolic route-policy subsystem, a vendored JavaBDD implementation, Python lockfiles, documentation, example networks, and test fixtures. The inspected repository is `batfish/batfish` on branch `master`, at commit `497c920d648f14cfe41d5cb5e13c133b024cd416`, with repository version file `projects/VERSION` reporting `0.36.0` and Bazel version `9.1.0`.[^1]

The central architectural pattern is a staged compiler-like pipeline:

```text
raw snapshot input
  -> format detection
  -> ANTLR parsing
  -> parse-tree extraction
  -> vendor-specific VendorConfiguration
  -> vendor-independent Configuration objects
  -> conversion finalization and validation
  -> topology and data-plane computation
  -> forwarding analysis, traceroute, BDD reachability, or question-specific analysis
  -> Answer / TableAnswerElement / serialized result
```

The repository is not a generic service application. Its main technical artifact is a set of normalized network models and analysis engines. The top-level `projects/batfish` subproject owns the worker-side analysis pipeline and data-plane computation. `projects/common` owns shared interfaces, models, topology abstractions, grammar support, BDD utilities, and serialization-facing data structures. `projects/coordinator` owns network and snapshot storage orchestration, question storage, work queue management, and REST resources. `projects/question` owns many user-facing Java question plugins and answerers. `projects/symbolic` provides symbolic state and ingress abstractions used by BDD reachability. `projects/bdd` vendors the `net.sf.javabdd` implementation. `projects/minesweeper` provides symbolic route-policy and route-advertisement analysis rather than packet-forwarding reachability. `projects/allinone` composes client, coordinator, and worker execution into a single process launcher.[^2]

The most important implementation conclusion is that Batfish separates vendor-specific parsing/extraction from vendor-independent analysis. Vendor support is not just grammar work. It requires grammar changes, extractor changes, vendor representation changes, conversion logic, warnings/structure tracking, and tests. The repository’s developer documentation explicitly describes the stages from parsing through forwarding analysis, and the source confirms these stages through `ParseVendorConfigurationJob`, `VendorConfiguration`, `ConvertConfigurationJob`, `IncrementalDataPlanePlugin`, `IncrementalBdpEngine`, `Answerer`, and the question plugin interfaces.[^3]

The main operational model is a snapshot-based service. A client initializes a network, uploads a snapshot packaged with configuration directories, stores questions, queues work, and retrieves answers. The coordinator validates and stores snapshot input, assigns IDs, computes work details, queues parsing/data-plane/question work, and returns serialized answers through REST resources. The all-in-one launcher starts a coordinator and Batfish worker thread and optionally runs the Java client. The repository README and build docs also point to Pybatfish and Docker as primary user-facing operational paths, with Docker image building maintained in a separate `batfish/docker` repository according to the official build documentation.[^4]

## 2. Scope and Evidence

### Repository researched

Primary repository:

```text
https://github.com/batfish/batfish
```

Only one primary repository was supplied. Adjacent ecosystem repositories and packages are discussed only where directly referenced by the repository README or official Batfish documentation. Pybatfish and Docker packaging are ecosystem context, not peer repositories in this report.[^4]

### Inspected revision

The inspection used branch `master` and commit:

```text
497c920d648f14cfe41d5cb5e13c133b024cd416
```

Inspection date: May 19, 2026.

### Methods used

The inspection combined repository metadata, GitHub-rendered directory listings, official repository documentation, build files, representative production source files, and representative tests. The strongest evidence in this report comes from source files and build files. Documentation is used to explain intended architecture, terminology, and workflows, but source-confirmed behavior is preferred when documentation and source differ.

The local container could not resolve GitHub over DNS, so repository inspection used GitHub web views and the available GitHub connector rather than a local clone. That means this report did not run a full build, execute tests, inspect every generated grammar artifact, or perform local grep over the entire tree. The report avoids claiming full repository-wide coverage where only representative files were inspected.

### Evidence-quality rules applied

Findings are classified as follows:

| Classification | Meaning |
| --- | --- |
| Confirmed | Directly supported by inspected source, build files, tests, or official repository documentation. |
| Source-grounded inference | A conclusion drawn from multiple confirmed observations, with the inference boundary stated. |
| Unresolved | A relevant detail where inspected evidence was insufficient, contradictory, or too partial for a firm claim. |

### Source limits that affect interpretation

The strongest source limit is generated and grammar-related code. The report inspected grammar support classes, parser/extractor documentation, conversion code, and a large parser test, but it did not inspect every vendor ANTLR grammar, every generated parser, or every vendor extractor. Claims about the grammar system are therefore architectural and workflow-level, not exhaustive per-vendor claims.

A second source limit is documentation freshness. For example, the official topology documentation discusses a `TopologyContainer` with Layer 2 topology, but the inspected `TopologyContainer` source exposes Layer 1 topologies, Layer 3 topology, L3 adjacencies, and protocol/overlay topologies, not a `getLayer2Topology()` accessor. This report treats that as a documentation/source mismatch and follows the inspected source when describing the container’s public interface.[^5]

## 3. Repository Layout

### Top-level repository map

| Path | Primary role | Production, test, docs, tooling, or examples | Notes |
| --- | --- | --- | --- |
| `projects/` | Main Java subprojects and core build units | Production and tests | Contains `batfish`, `common`, `coordinator`, `client`, `question`, `symbolic`, `bdd`, `minesweeper`, and `allinone`.[^2] |
| `docs/` | Developer and user-facing documentation | Docs | Includes architecture, parsing, extraction, conversion, post-processing, data-plane, topology, symbolic engine, forwarding analysis, development, and build/run docs.[^2] |
| `questions/` | JSON question templates | User-facing question templates | Split into `stable` and `experimental`; templates are loaded by clients/coordinator and map to Java question classes through JSON `class` and instance metadata.[^6] |
| `python/` | Python dependency lockfiles and Python build metadata | Tooling and ecosystem support | Contains `requirements.in` and `requirements.txt`; README states the lockfile is generated from `requirements.in` with hashes.[^2] |
| `tests/` | External-style fixtures and reference networks | Test fixtures | Contains fixtures such as `aws`, `basic`, `logging`, `parsing-errors-tests`, `parsing-tests`, `questions`, and `roles`.[^2] |
| `tools/` | Developer scripts, benchmarks, profilers, format tooling, generated data tooling | Tooling | Contains scripts for Java formatting, checkstyle, reference updates, profiling, benchmarks, BDD tools, JOL, and Palo Alto application generation.[^2] |
| `networks/` | Example and sample networks | Examples and fixtures | Contains example networks such as `example`, `iptables-firewall`, `hybrid-cloud-aws`, and cloud-specific examples.[^2] |
| `MODULE.bazel` | Bazel module and dependency declaration | Build configuration | Declares Java/Python/Bazel dependencies, Maven artifacts, Python 3.10 toolchain, and Maven/pip lock usage.[^7] |
| `.bazelversion` | Bazelisk version pin | Build configuration | Pins Bazel to `9.1.0`.[^1] |
| `.pre-commit-config.yaml` | Local formatting/lint hooks | Developer tooling | Configures Java format, buildifier, Python formatting, and related hooks.[^7] |
| `README.md` | Project overview and getting-started entry point | Documentation | Describes Batfish, core analysis inputs, Docker all-in-one usage, and Pybatfish installation.[^1] |

### Production versus support material

The production Java code is concentrated under `projects/*/src/main/java`. The core worker and analysis engine live under `projects/batfish`. Shared data models, plugin interfaces, topology abstractions, grammar base classes, BDD utility classes, and common serialization models live under `projects/common`. Service/coordinator code lives under `projects/coordinator`. Java client code lives under `projects/client`. Java question implementations live mainly under `projects/question` and partly under `projects/minesweeper` for route-policy symbolic questions. Symbolic state abstractions live under `projects/symbolic`. The BDD implementation itself lives under `projects/bdd/src/main/java/net/sf/javabdd`.[^2]

The user-facing JSON question templates in `questions/` are not Java production code, but they are part of the operational question surface. The coordinator loads question templates by walking configured template directories, parsing JSON, extracting `instance` metadata, lowercasing the instance name for the template key, and hiding templates whose names begin with `__` unless verbose mode is requested.[^8]

The example networks under `networks/` and the fixtures under `tests/` provide realistic input material and reference data for testing, but they are not the primary implementation. The tools directory is developer support, including formatting, benchmark, profiling, and generated data update scripts.[^2]

### How the layout supports the Batfish workflow

The top-level layout mirrors the staged workflow. `questions/` and `projects/question` define what users ask. `projects/coordinator` and `projects/client` define how work is submitted and retrieved. `projects/batfish` defines how snapshots are parsed, converted, analyzed, and answered. `projects/common` defines the shared data and plugin contracts that let those pieces interact. `projects/symbolic`, `projects/bdd`, and `projects/minesweeper` provide specialized analysis foundations. `docs/`, `tests/`, `networks/`, and `tools/` provide development guidance, fixtures, examples, and automation around that core.[^2]

## 4. Architectural Overview

### Main subsystems

| Subsystem | Repository location | Responsibility | Boundary |
| --- | --- | --- | --- |
| Worker and analysis engine | `projects/batfish` | Parse snapshots, extract vendor models, convert to vendor-independent models, compute data planes, build topology, run analysis engines | Owns execution-heavy analysis jobs and data-plane algorithms. |
| Shared model and APIs | `projects/common` | Data model, question model, answer model, plugin APIs, grammar base classes, topology abstractions, BDD utility wrappers | Shared by client, coordinator, worker, question, and symbolic code. |
| Coordinator service | `projects/coordinator` | REST service, network/snapshot/question storage, ID management, work queue, answer retrieval | Owns persistent orchestration and API-facing service behavior. |
| Java client | `projects/client` | CLI-style command processing, question variable validation, snapshot upload helper flows, coordinator interaction | User-facing Java client, distinct from Pybatfish ecosystem client. |
| Question implementations | `projects/question` | Java `Question`, `QuestionPlugin`, and `Answerer` implementations for user-facing analyses | Maps templates and serialized questions to Java answer logic. |
| Symbolic state layer | `projects/symbolic` | `StateExpr`, state visitor, ingress locations, symbolic reachability state vocabulary | Supplies symbolic state names and traversal abstractions to BDD reachability. |
| BDD implementation | `projects/bdd` | Vendored `net.sf.javabdd` classes and tests | Low-level BDD factory, variables, domains, pairings, and operations. |
| Minesweeper | `projects/minesweeper` | Symbolic route-policy and route-advertisement analysis using BDDs and atomic predicates | Complements packet reachability by analyzing routing policies and BGP-route transformations. |
| All-in-one launcher | `projects/allinone` | Starts client, coordinator, and worker in one process-style composition | Operational composition, not a separate analysis engine. |

### High-level architecture diagram

```text
                     +----------------------------+
                     | User / Pybatfish / Client  |
                     +-------------+--------------+
                                   |
                                   v
                     +----------------------------+
                     | Coordinator REST service   |
                     | WorkMgrServiceV2/resources |
                     +-------------+--------------+
                                   |
                  network/snapshot/question IDs, storage, work queue
                                   |
                                   v
                     +----------------------------+
                     | Batfish worker             |
                     | projects/batfish           |
                     +-------------+--------------+
                                   |
             +---------------------+---------------------+
             |                                           |
             v                                           v
+---------------------------+             +-----------------------------+
| Parse/extract/convert     |             | Question dispatch            |
| ParseVendorConfiguration  |             | QuestionPlugin / Answerer     |
| VendorConfiguration       |             +--------------+--------------+
| ConvertConfigurationJob   |                            |
+-------------+-------------+                            |
              |                                          |
              v                                          |
+-----------------------------+                          |
| Vendor-independent model    |                          |
| Configuration / Interface   |                          |
| VRF / ACL / policy / route  |                          |
+-------------+---------------+                          |
              |                                          |
              v                                          v
+-----------------------------+             +-----------------------------+
| Data plane and topology     |             | Concrete and symbolic answers |
| IncrementalDataPlanePlugin  |             | Tables, traces, BDD results   |
| IncrementalBdpEngine        |             +--------------+--------------+
+-------------+---------------+                            |
              |                                            v
              v                              +-----------------------------+
+-----------------------------+              | Answer serialization/storage |
| DataPlane / Topology        |              +-----------------------------+
| ForwardingAnalysis / FIBs   |
+-----------------------------+
```

### Control-flow summary

Control begins outside the worker. A client initializes a network, uploads a snapshot, loads a question, or asks an existing question. The coordinator’s REST service dispatches to resources and the `WorkMgr`, which validates network and snapshot existence, stores inputs, assigns IDs, and queues work. `WorkMgr.computeWorkDetails` classifies work as parsing, data-planing, or one of the answer types by inspecting the `WorkItem` and the parsed `Question` properties `getIndependent()` and `getDataPlane()`.[^8]

When a worker runs analysis, the parsing stage detects configuration formats, instantiates vendor-specific parsers and extractors, and creates vendor-specific `VendorConfiguration` objects. The conversion stage calls `VendorConfiguration.toVendorIndependentConfigurations()` and then finalizes each `Configuration`, checking references, pruning invalid structures, setting immutable maps, validating interfaces and protocols, and aggregating warnings. Data-plane computation loads configurations, external BGP advertisements, and initial topologies, then invokes the iBDP engine to compute RIBs, FIBs, topology updates, forwarding analysis, and related structures.[^9]

Question answering uses `QuestionPlugin` registration to map a serialized Java question class to an `Answerer`. An answerer receives an `IBatfish` instance and a `NetworkSnapshot`, loads the models it needs, computes rows or structured answer elements, and returns an `AnswerElement` or `Answer`. Differential answering is supported at the base `Answerer` level for table answers by running base and reference answers and applying table diff logic, unless a question overrides or rejects differential behavior.[^10]

### Data-flow summary

Data moves through a series of increasingly semantic representations:

| Stage | Data shape | Owner | Validation and normalization |
| --- | --- | --- | --- |
| Snapshot input | Files under a packaged snapshot directory | Coordinator storage, then worker | Snapshot packaging must contain at least one recognized config directory; exactly one top-level folder is expected for uploads.[^8] |
| Parse input | Text files and detected `ConfigurationFormat` | `ParseVendorConfigurationJob` | Empty, ignored, unknown, and unimplemented formats are handled distinctly. Parser errors and warnings are captured.[^11] |
| Parse tree | ANTLR parse tree | Vendor parser | Base combined parser sets prediction mode, error listeners, and optional recovery.[^12] |
| Vendor-specific model | `VendorConfiguration` subclass | Extractor and representation package | Extractor walks parse tree; `VendorConfiguration` tracks filenames, warnings, structure references, overlays, secondary files, runtime data, and conversion context.[^13] |
| Vendor-independent model | `Configuration` and related datamodel classes | Conversion job and vendor conversion code | `ConvertConfigurationJob.finalizeConfiguration` validates defaults, references, interfaces, protocol structures, static routes, tracks, ACL invariants, and immutability.[^14] |
| Data plane | `DataPlane`, RIBs, FIBs, `TopologyContainer`, `ForwardingAnalysis` | iBDP engine | Fixed-point routing/topology computation with BGP, IGP, tunnels, VXLAN, tracks, IP ownership, and oscillation checks.[^15] |
| Analysis output | `Answer`, `TableAnswerElement`, traces, BDD models, route-policy results | Question answerers and analysis engines | Answerers build table metadata, rows, traces, reachability results, or symbolic route-policy witness routes.[^10] |

## 5. End-to-End Runtime Flow

### Sequence narrative

The generic sequence requested in the prompt refines to this source-grounded path:

```text
User / client
  -> WorkMgrServiceV2 or Java client helper
  -> WorkMgr network/snapshot/question storage
  -> WorkQueueMgr and work executor
  -> Batfish worker service
  -> ParseVendorConfigurationJob
  -> BatfishCombinedParser + vendor parser/extractor
  -> VendorConfiguration
  -> ConvertConfigurationJob
  -> Configuration objects and warnings
  -> topology provider + IncrementalDataPlanePlugin
  -> IncrementalBdpEngine, Node, VirtualRouter, RIB/FIB computation
  -> DataPlane + TopologyContainer + ForwardingAnalysis
  -> QuestionPlugin / Answerer / BDD or concrete analysis engine
  -> Answer or serialized JSON result
  -> Coordinator answer retrieval resource or client result display
```

### Snapshot creation

Snapshot creation is coordinator-owned. `WorkMgr.initNetwork` generates a `NetworkId`, initializes storage, assigns the human network name, and returns the network name. `WorkMgr.initSnapshot` validates the uploaded snapshot directory, moves `runtime_data.json` into `batfish/` if needed, migrates deprecated interface blacklist input into runtime data, stores snapshot input objects, merges reference library data if present, writes snapshot metadata, and assigns the snapshot name to a generated `SnapshotId`.[^8]

Snapshot packaging has observable rules. The upload root must contain exactly one non-ignored top-level directory, and that snapshot directory must contain at least one of the recognized configuration directories: host configs, network configurations, AWS configs, SONIC configs, or Azure configs. If the required packaging is absent, `WorkMgr` raises an exception with the official packaging instructions URL.[^8]

### Service receives work

The coordinator starts a Grizzly/Jersey REST service using `WorkMgrServiceV2` and a configured work V2 port. `WorkMgrServiceV2` is annotated as a REST resource and exposes endpoints for networks, question templates, network resources, and version information. It uses an API-key header with a default API key and registers Jackson, an exception mapper, and authentication filter features.[^17]

Work classification happens in `WorkMgr.computeWorkDetails`. Parsing, data-planing, and answering are mutually constrained. Composite parsing-plus-data-plane or answering-plus-other work is rejected. For questions, `Question.parseQuestion` is used to inspect whether the question is independent, data-plane-dependent, or parsing-dependent.[^8]

### Raw configuration files become parse trees

`ParseVendorConfigurationJob` receives a map from filenames to file text and an expected format. It detects format if needed, handles empty and ignored text, rejects or flags unknown/unimplemented formats, and then selects vendor-specific parser and extractor combinations from a large switch over `ConfigurationFormat`. For nested formats such as Juniper, VyOS, and Palo Alto, the job may flatten text before parsing. For SONiC, it has a multi-file processing branch rather than a single text parser.[^11]

The parser stack uses `BatfishCombinedParser` as a base wrapper around an ANTLR lexer and parser. It configures parser prediction, lexer and parser error listeners, optional recovery infrastructure, and token-mode tracking. `BatfishLexer` provides shared lexer recovery and mode behavior. This is why parser support in Batfish is not just `.g4` files; it is an ANTLR grammar plus a shared parser runtime policy.[^12]

### Parse trees become vendor-specific models

Extraction is performed through `ControlPlaneExtractor`, which extends `BatfishExtractor`. The contract is small: process a parse tree for a snapshot and return a `VendorConfiguration`. The docs describe the standard single-file flow as parse tree walking through preprocessors and listeners, and the source confirms extractors are invoked by `ParseVendorConfigurationJob.parseFile` after successful parsing. The custom `BatfishParseTreeWalker` wraps parse-tree traversal exceptions with parse context and rethrows commit-control exceptions.[^11]

The vendor-specific result is a `VendorConfiguration` subclass. The base class stores hostname, vendor format, filename, secondary filenames, overlay configuration, warnings, conversion context, runtime data, ISP configuration, and a `StructureManager` for definitions and references. It exposes the abstract `toVendorIndependentConfigurations()` conversion method and helper methods for defining, referencing, renaming, deleting, and marking undefined vendor structures.[^13]

### Vendor-specific models become vendor-independent configurations

`ConvertConfigurationJob.call` casts the parsed object to `VendorConfiguration`, sets warnings, conversion context, and runtime data, then invokes `toVendorIndependentConfigurations()` in parallel. Each produced `Configuration` is finalized by `finalizeConfiguration`, after optional iptables overlay processing. The conversion job records the file-to-host map, saves structure information, and merges warnings deterministically.[^14]

The vendor-independent `Configuration` model is broad. It represents a network device such as a router, switch, firewall, or host. It owns maps and sets for interfaces, VRFs, ACLs, route filters, routing policies, authentication material, IPsec/IKE objects, community and AS-path structures, packet policies, generated reference books, NTP/DNS/logging/TACACS/SNMP, tracking groups, vendor family, zones, device type/model, and normal VLAN ranges.[^16]

### Post-processing finalizes the model

Model finalization is not a cosmetic step. `ConvertConfigurationJob.finalizeConfiguration` checks that default inbound and cross-zone actions are present, adds tenant VNI interfaces, removes undefined routing-policy references, simplifies and annotates routing policies, verifies interfaces, verifies OSPF areas and VRRP groups, removes invalid static routes and undefined track references, asserts valid vendor structure IDs in tests, and then converts many mutable collections into immutable or sorted maps and sets.[^14]

Post-processing also repairs or rejects partial models. Undefined IP-space references are replaced with `EmptyIpSpace` and red-flag warnings. Interfaces with undefined channel groups are removed. OSPF area references to undefined interfaces are pruned with warnings. Static routes, BGP tracks, HSRP/VRRP tracks, and other references are verified or removed depending on the structure.[^14]

### Data plane is computed

The data-plane plugin mechanism is explicit. `DataPlanePlugin` registers with `IBatfish`, and `IncrementalDataPlanePlugin` registers the `ibdp` engine with AutoService. `computeDataPlane` loads configurations and external BGP announcements, gets initial topology objects and IP owners from the topology provider, builds a `TopologyContext`, and delegates to `IncrementalBdpEngine.computeDataPlane`.[^15]

`IncrementalBdpEngine` performs fixed-point computation. It creates sorted `Node` objects and a randomized list of `VirtualRouter` instances for parallelization across VRFs, computes IGP data plane, initializes EGP computation, and then iterates topology and routing until topologies, track results, IP owners, and VXLAN autostate converge. It throws `BdpOscillationException` for non-monotonic oscillation or failure to converge within `MAX_TOPOLOGY_ITERATIONS`, which is `10` in the inspected source.[^15]

### Questions are dispatched and answered

`QuestionPlugin.pluginInitialize` creates a question instance to discover its name and canonical class name. When running under a Batfish plugin consumer, it registers an answerer with Batfish. When running under a client plugin consumer, it registers the question with the client. `Answerer.create` delegates to `IBatfish.createAnswerer`; individual answerers implement `answer(NetworkSnapshot)`.[^10]

A representative question, `IpOwners`, shows the pattern. Its plugin creates an `IpOwnersAnswerer` and an `IpOwnersQuestion`. The answerer loads configurations, gets initial IP owners from the topology provider, resolves a requested IP space through the specifier context, builds rows with node, VRF, interface, IP, mask, and active status, and returns a `TableAnswerElement` with metadata.[^10]

### Results are returned

For stored answers, the coordinator’s `AnswerResource` checks snapshot existence, loads the answer through `Main.getWorkMgr().getAnswer`, optionally filters it, and returns JSON. `WorkMgr.getAnswer` derives an `AnswerId` from network, snapshot, question, node roles, and optional reference snapshot IDs, checks answer metadata existence, and loads the serialized answer. Missing metadata means the question has not been answered.[^8]

## 6. Parsing, Extraction, and Conversion Pipeline

### Grammar architecture

Batfish parser support is ANTLR-based for device configuration DSLs and direct object mapping for structured formats. The parsing docs distinguish DSL-style configurations from formats such as JSON and describe the flow from raw text to parse tree to extractor. They also define progressive support levels: unrecognized syntax, grammar-recognized but silently ignored syntax through `_null` rules, grammar-recognized syntax that produces TODO/unimplemented extraction warnings, extracted syntax that warns or errors during conversion, and fully implemented syntax.[^12]

The base Java parser infrastructure is in `projects/common`. `projects/common/src/main/java/BUILD.bazel` groups parser-common sources, including `BatfishCombinedParser`, `BatfishLexer`, parser error listeners, grammar settings, implemented rule tracking, parse-tree walker, lexer recovery strategies, and related support. The documentation’s example vendor parser layout describes lexer grammar, parser grammar, base lexer, combined parser, extractor, and Bazel package targets. Exact vendor file names vary by vendor and must be discovered in the existing vendor package before patching.[^12]

### Lexer and parser design

The docs define naming and structural conventions: lexer token names use uppercase; parser rules use lowercase; start rules should include `EOF`; token order matters; and rules that intentionally consume syntax without extraction can use `_null` suffixes. The base combined parser sets prediction mode to SLL, can enable or disable recovery around unrecognized input, and records parser and lexer errors. `BatfishLexer` adds mode tracking and lexer recovery behavior.[^12]

This design enables a staged support model. A developer can first make syntax recognized, then decide whether it should be ignored, warned on, extracted, converted, or fully modeled. That staged model is source-grounded by the parse job’s handling of parse statuses and the warning system’s distinct parse, unimplemented, red-flag, and pedantic warning categories.[^11]

### Vendor grammars and parser support

`ParseVendorConfigurationJob.parseFiles` is the switchboard that maps detected `ConfigurationFormat` values to parser/extractor pairs and special handling. The inspected source includes many cases, including Cisco-family, Juniper, Arista, Cumulus, F5, FortiOS, Palo Alto, AWS, Azure, SONiC, host JSON, and others. Some formats are explicitly marked unimplemented. Unknown, ignored, and empty files get separate parse statuses.[^11]

This means adding vendor syntax support normally requires changes in at least four places:

1. Grammar recognition, usually in the vendor ANTLR grammar and supporting parser classes.
2. Extraction, usually in a vendor `ControlPlaneExtractor` and any preprocessing listeners.
3. Vendor representation classes, if the syntax introduces new vendor-specific structures.
4. Conversion into vendor-independent `Configuration` objects, if the syntax affects behavior.

Parser tests and conversion tests must be updated to encode the expected syntax support level.[^11]

### Extraction layer

The extraction layer consumes parse trees and produces `VendorConfiguration` objects. The docs describe single-file extraction as a sequence of preprocessors and listeners that walk the parse tree. Juniper-family multifile extraction is more complex because multiple parse trees may be merged before preprocessing and final extraction. The source interface is intentionally small: `BatfishExtractor.processParseTree` and `ControlPlaneExtractor.getVendorConfiguration`.[^11]

Preprocessing is needed when vendor syntax changes the tree before semantic extraction. The docs call out Juniper-style delete/deactivate/apply-groups/insert/replace handling and describe listeners that rewrite or annotate the tree before extraction. This is a source-grounded documentation claim, but per-vendor details must be verified in the relevant vendor packages before implementation.[^11]

### Warning and partial-support strategy

Warnings are a first-class part of parsing, extraction, and conversion. `Warnings` carries parse warnings, pedantic warnings, red flags, unimplemented warnings, and optional error details. It can create warnings for parse context, line/text/comment, TODO/unimplemented extraction paths, and red-flag conversion problems. `VendorConfiguration` stores warnings and structure-reference data, and conversion jobs merge configuration-specific warnings deterministically.[^11]

Warnings are not interchangeable:

| Warning category | Typical source | Meaning |
| --- | --- | --- |
| Parse warning | Parser or extractor | Syntax or line was recognized with a warning or partial behavior. |
| Unimplemented warning | Extractor or converter | Syntax or behavior was identified but not implemented. |
| Red-flag warning | Conversion or validation | Potentially behavior-affecting issue, such as undefined reference removal. |
| Pedantic warning | Detailed model quality warning | Lower-severity, often useful during development or strict auditing. |
| Error details | Parser or job failure | Fatal parse or conversion context. |

### Conversion layer

The conversion layer normalizes vendor-specific structures into vendor-independent models. The documentation describes conversion as translation, normalization, validation, reference resolution, and warning emission. The source confirms that `VendorConfiguration` is the base conversion contract and that `ConvertConfigurationJob` is the worker-side conversion and finalization job.[^14]

The base `VendorConfiguration` class provides structure tracking, filename and secondary-file tracking, conversion context, runtime data, overlay configuration, ISP configuration hooks, and abstract conversion to one or more vendor-independent configurations. Some vendor configurations can produce multiple `Configuration` objects, which is why `ConvertConfigurationJob.call` iterates over `toVendorIndependentConfigurations()` rather than assuming one output per input file.[^13]

### Vendor-independent model

The vendor-independent model centers on `Configuration`, `Interface`, `Vrf`, ACLs, route filters, routing policies, protocol process objects, IP spaces, packet policies, and topology/data-plane structures. `Configuration` lowercases hostnames in its builder and initializes major maps and sets in its constructor. The model uses a mix of builders, immutable collections, typed route objects, visitor-style expression trees, and JSON annotations.[^16]

Key VI model families include:

| Model family | Representative classes | Role |
| --- | --- | --- |
| Device model | `Configuration`, `DeviceType`, `ConfigurationFormat` | Device-level normalized state. |
| Interface model | `Interface`, `ConcreteInterfaceAddress`, `InterfaceType`, switchport fields | L1/L2/L3 interface behavior and dependencies. |
| VRF/routing model | `Vrf`, `BgpProcess`, `OspfProcess`, static/generated/local/connected route classes | Control-plane and RIB inputs. |
| Policy model | `RoutingPolicy`, routing-policy expressions/statements, route filters, AS-path and community structures | Route import/export, redistribution, and policy transformation. |
| ACL/security model | `IpAccessList`, `AclLine`, `AclLineMatchExpr`, `HeaderSpace`, `IpSpace` | Packet filtering and symbolic encoding input. |
| Topology model | `Layer1Topologies`, `Layer2Topology`, `Topology`, protocol topologies | Physical/logical/protocol connectivity. |
| Data-plane model | `DataPlane`, `Fib`, RIB tables, `ForwardingAnalysis` | Computed forwarding behavior. |
| Answer model | `Question`, `Answer`, `AnswerElement`, `TableAnswerElement`, `Row`, `Schema` | Serialized analysis outputs. |

### Extension workflow for new vendor syntax

A safe extension workflow is:

1. Determine whether the syntax must affect Batfish behavior. The parser docs explicitly ask whether syntax can change the control plane, forwarding behavior, security policy, or protocol establishment. If not, it may be ignored through grammar-only support.[^12]
2. Add or update lexer/parser rules in the vendor grammar. Use `_null` rules only for intentionally ignored syntax.
3. Update the vendor extractor to record the syntax in the vendor-specific representation or emit an appropriate TODO/unimplemented warning.
4. Update the vendor representation model if the syntax introduces a new vendor-specific concept.
5. Update conversion logic so the concept affects `Configuration`, `Interface`, `Vrf`, ACLs, route filters, routing policies, protocol settings, or topology inputs as appropriate.
6. Add parser tests for recognition and warning behavior.
7. Add conversion/model tests for normalized output and red-flag behavior.
8. Add data-plane or question tests if the syntax changes routing, forwarding, filtering, topology, or answer behavior.

## 7. Data Plane, Topology, and Forwarding

### Data-plane computation

The iBDP engine is the primary inspected data-plane implementation. `IncrementalDataPlanePlugin` registers plugin name `ibdp`, loads configurations and external BGP advertisements, builds a topology context from the topology provider, and calls `IncrementalBdpEngine.computeDataPlane`. The plugin’s return type is `ComputeDataPlaneResult`, which includes a `DataPlane`, a `TopologyContainer`, and a data-plane answer element through the abstract plugin base class.[^15]

`IncrementalBdpEngine` computes the data plane using fixed-point iteration. Its comments describe the run sequence as: let IGP routes converge, initialize BGP neighbors with reachability checks, let EGP routes converge, and compute FIBs. The source then iterates topology and routing state, recalculating IPsec, VXLAN, tunnel, EIGRP, BGP, L3 adjacencies, Layer 3 topology, track reachability, track routes, and IP ownership until convergence.[^15]

### Routing strategy and data structures

`VirtualRouter` is the per-node/per-VRF state holder for routing computation. It owns connected, local, static, independent, generated, main, ISIS, RIP, and other RIBs; BGP and OSPF process references; incoming route queues; cross-VRF route queues; the computed FIB; VNI settings; and prefix-tracing metadata. The iBDP engine constructs `Node` objects and flattens their virtual routers into a list used for parallel computation.[^15]

The public `DataPlane` interface exposes the major computed data products: BGP routes, backup BGP routes, EVPN routes, backup EVPN routes, FIBs, forwarding analysis, RIBs, prefix tracing information, and Layer 2 and Layer 3 VNI settings.[^16]

### Topology model

The topology model is layered:

| Layer | Representative source | Role |
| --- | --- | --- |
| Layer 1 | `Layer1Topologies` | Physical and logical edge views: user-provided, synthesized, logical, active logical, and combined. |
| Layer 2 | `Layer2Topology` | Broadcast-domain equivalence represented with union-find during construction. |
| Layer 3 | `Topology`, `L3Adjacencies` | IP adjacency and routed edge model used for forwarding and protocol topology. |
| Overlay | VXLAN, IPsec, tunnel topology objects | Dynamic overlays pruned or recomputed based on reachability. |
| Protocol | BGP, OSPF, ISIS, EIGRP topologies | Protocol-neighbor graph inputs for route exchange. |

The source-level `TopologyContainer` exposes converged data-plane topologies: BGP, EIGRP, ISIS, Layer 1 topologies, L3 adjacencies, Layer 3 topology, OSPF, VXLAN, and tunnel topology. It does not expose `getLayer2Topology()` in the inspected source. Layer 2 still exists as a source-level abstraction and is used internally in topology computation, but it is not part of the inspected `TopologyContainer` interface.[^5]

`Layer2Topology` uses a union-find builder. Adding an edge unions the two `Layer2Node`s, and building the topology creates an immutable map from each node to a representative. The API can test whether node-interface pairs are in the same broadcast domain.[^5]

### Forwarding analysis

Forwarding analysis separates RIB/FIB output from packet-disposition reasoning. The `ForwardingAnalysis` interface returns ARP replies and a map from hostname to VRF to `VrfForwardingBehavior`. `VrfForwardingBehavior` contains routable IPs, null-routed IPs, interface forwarding behavior, next-VRF behavior, and ARP-true edge IP spaces. `InterfaceForwardingBehavior` divides interface-level forwarding into accepted IPs, delivered-to-subnet IPs, exits-network IPs, neighbor-unreachable IPs, and insufficient-info IPs, defaulting missing IP spaces to `EmptyIpSpace`.[^16]

Those structures are used by both concrete and symbolic analysis. Concrete traceroute and flow analysis can use FIB and forwarding behavior directly. BDD reachability converts these forwarding behavior IP spaces to BDDs and creates graph edges to disposition states such as `Accept`, `DropNoRoute`, `DropNullRoute`, `DeliveredToSubnet`, `ExitsNetwork`, `NeighborUnreachable`, and `InsufficientInfo`.[^18]

### Caching, persistence, and reuse

The inspected source confirms that `WorkMgr` stores and loads serialized answers, snapshot input objects, topology objects, and metadata through a `StorageProvider`. The architecture docs state that data-plane results are serialized to snapshot storage and reused by data-plane-dependent questions. The precise cache invalidation and storage-key policy for every computed artifact was not exhaustively inspected; the report therefore treats persistence as confirmed at the WorkMgr/storage interface level and source-grounded for data-plane reuse through `IBatfish.loadDataPlane`, `DataPlanePlugin`, and question dependencies.[^8]

## 8. Symbolic and BDD-Based Analysis

### BDD packet model

Batfish’s packet-level symbolic reachability uses BDDs to encode packet header fields. `BDDPacket` allocates BDD variables for destination/source IPs, destination/source ports, IP protocol, ICMP code/type, TCP flags, DSCP, ECN, fragment offset, and packet length. It uses `net.sf.javabdd.JFactory`, an initial node table size of `1_000_000`, cache ratio `8`, and reserves the first packet variable index at `100` so auxiliary variables can be allocated before packet variables.[^19]

The low-level BDD implementation is vendored in `projects/bdd`. The top-level `projects/bdd/BUILD.bazel` aliases `bdd` to `//projects/bdd/src/main/java/net/sf/javabdd`, and that package builds a public `java_library` named `javabdd` from all Java sources. Source files include `BDD`, `BDDFactory`, `JFactory`, `BDDPairing`, `BDDDomain`, and related tests.[^19]

### Header-space and ACL encoding

`HeaderSpaceToBDD` converts a `HeaderSpace` to a BDD. It treats null header-space fields as unconstrained, applies positive and negative constraints for packet length, fragments, ECN, DSCP, TCP flags, ICMP, protocol, ports, and source/destination IP spaces, and then optionally negates the final BDD if `HeaderSpace.getNegate()` is set.[^19]

`IpAccessListToBdd` converts ACLs and ACL match expressions to BDDs. It memoizes ACL conversions, detects cyclic ACL references through `NonRecursiveSupplier`, computes permit and deny BDDs, and explicitly models ACL default action as denying all unmatched traffic. Its line-reachability method appends a default deny `PermitAndDenyBdds` element for unmatched packets.[^19]

### Reachability graph

`BDDReachabilityAnalysisFactory` constructs the symbolic reachability graph. It generates `StateExpr` nodes and edges with BDD or transition labels. The factory builds ACL permit/deny BDDs, ARP-true edge BDDs, null-routed and routable BDDs, interface accept BDDs, forwarding-disposition BDDs, transformation transitions, outgoing original flow filters, source managers, and optional session/last-hop state.[^18]

`BDDReachabilityAnalysis` holds a forward edge table, a memoized transposed edge table, ingress states, and a query header-space BDD. It can compute reverse reachable states from the query state, reverse reachability from explicit roots, or forward reachable states from ingress locations. The analysis description notes that ACLs are encoded as edge labels and that source NAT is tracked through transformations and reconstructed rather than represented as separate original/current source-IP variables.[^18]

The symbolic state vocabulary comes from `projects/symbolic`. `StateExpr` is a serializable visitor-based interface. `StateExprVisitor` enumerates concrete states such as accept, drops, node/interface dispositions, origination states, pre/post interface and VRF states, packet-policy states, session states, and query. `IngressLocation` represents reachability starting points as either interface-link ingress or VRF ingress.[^18]

### NAT and transformations

`TransformationToTransition` converts data-model `Transformation` objects into BDD reachability transitions. It maps source/destination IP and port fields to primed BDD variables, handles IP-address pool assignment, subnet shifting, port pool assignment, `ApplyAll`, `ApplyAny`, no-op steps, guard branches, and return-flow transformation reversal. Port assignment is constrained to protocols that have ports and is a no-op for protocols without ports.[^18]

### JavaBDD memory-management implications

BDD operations are memory-sensitive. The inspected code exposes several memory-management signals: `BDDPacket` configures JFactory node and cache sizes; `IpAccessListToBdd` comments avoid leaking BDDs when converting expression ACL lines; `TransferBDD` in Minesweeper has an `assertNoLeaks` helper that checks outstanding BDD counts; and many BDD methods use JavaBDD idioms such as `id`, `andWith`, `orWith`, and `diffEq`, where ownership matters.[^19]

### Concrete forwarding versus symbolic reachability

Concrete forwarding analysis computes behavior from actual FIBs, topology, ARP behavior, interface behavior, and flows. Symbolic reachability computes sets of packet headers that can reach states or dispositions. Concrete traceroute is useful for a small number of flows and explains hops. BDD reachability is useful for quantifying large header spaces, proving absence or presence of classes of flows, and answering reachability questions without enumerating every packet. Both depend on the same vendor-independent model and computed forwarding structures, but they use different representations and algorithms.[^16]

### Minesweeper relationship

Minesweeper is related but distinct. `BDDRoute` represents symbolic route advertisements, not packet headers. It models route attributes such as prefix, prefix length, administrative distance, communities, AS-path atomic predicates, local preference, MED, next hop, origin type, OSPF metric type, protocol history, optional route-policy environment dimensions, tunnel encapsulation, tracks, weight, and unsupported-policy state. `TransferBDD` is described as the function from an input routing advertisement to an output routing advertisement and evaluates routing-policy expressions/statements over symbolic route attributes.[^20]

The `SearchRoutePoliciesAnswerer` uses symbolic route-policy analysis to find concrete or symbolic BGP routes satisfying input/output/action constraints for route policies. It constructs atomic predicates for community and AS-path regexes, runs transfer analysis, extracts models, and cross-checks symbolic predictions against route-policy simulation. This is not the same subsystem as packet BDD reachability, though both use JavaBDD and the shared VI data model.[^20]

## 9. Question Framework and User-Facing Analysis

### Question templates

The `questions/` directory holds JSON question templates, split into `stable` and `experimental`. The questions README states that these templates are loaded by clients and the coordinator and that users can run or load questions through the client. The question development docs describe a Pybatfish call such as `bf.q.nodeProperties()` as a JSON template that eventually instantiates a Java question class and answerer.[^6]

The coordinator template loader reads JSON files under configured question-template directories, requires an `instance` object, extracts `InstanceData`, and stores the template under the lowercase instance name. Hidden templates start with `__` and are omitted unless verbose mode is requested.[^8]

### Java question classes

The base `Question` class is JSON-polymorphic through the `class` property. It carries display hints, exclusions, instance data, assertion, differential flag, and include-one-table-key behavior. It requires subclasses to implement `getDataPlane()` and `getName()`. Its `getIndependent()` default is `false`, so question subclasses must opt into independence if they do not need snapshot parsing or data-plane state.[^10]

`Question.parseQuestion` preprocesses JSON, handles variable replacement and optional-variable removal in templates, and maps the result through Batfish’s object mapper. This is the bridge from JSON question template to Java question object.[^10]

### Answerers and plugins

`QuestionPlugin` is the registration bridge. It requires subclasses to create a question and answerer. On initialization, it discovers the question name and canonical class name, then registers either an answerer with Batfish or a question with the client depending on the plugin consumer. Representative question plugins use `@AutoService(Plugin.class)`, which lets service loading discover them.[^10]

`Answerer` is the base class for analysis logic. It stores the question, `IBatfish`, and logger, and defines `answer(NetworkSnapshot)`. Its static factory method delegates to Batfish. Its default differential behavior runs the base and reference answers and applies `TableDiff` when both answers are table answers; otherwise it raises an unsupported operation.[^10]

### Parameters and schemas

Question parameters are represented in JSON templates by variables and are validated by clients before instantiation. The Java client validates required variables, unknown parameter names, type-specific constraints such as JSON path structure and Java regular-expression syntax, minimum array sizes, and allowed values. Answer schemas are expressed through `TableMetadata`, `ColumnMetadata`, `Schema`, and row builders in answerers. `IpOwnersAnswerer` is a representative example: it defines column metadata and creates table rows for owner IPs.[^10]

### Question-framework workflow map

| Step | File family | Observable behavior |
| --- | --- | --- |
| Define Java question | `projects/question/.../*QuestionPlugin.java` or specialized package | Subclass `Question`, implement `getName()` and `getDataPlane()`, expose JSON properties. |
| Define answerer | `projects/question/.../*Answerer.java` | Subclass `Answerer`, load required Batfish data, compute answer element. |
| Register plugin | `QuestionPlugin` with `@AutoService(Plugin.class)` | Plugin initialization registers answerer with Batfish and question with client. |
| Add template | `questions/stable` or `questions/experimental` | JSON template references Java `class` and declares instance metadata/variables. |
| Validate parameters | `projects/client` or Pybatfish path | Client validates missing, unknown, incorrectly typed, or disallowed variables. |
| Execute | Coordinator queues answer work; worker calls answerer | Result is serialized as `Answer` or answer element. |
| Retrieve | `AnswerResource` or client helper | Coordinator returns stored JSON answer or filtered answer. |

### Adding a new question

A developer adding a question should touch:

1. Java question package under `projects/question/src/main/java/org/batfish/question/...` or a specialized subsystem such as `projects/minesweeper` if the analysis belongs there.
2. A `Question` subclass with stable JSON properties.
3. An `Answerer` subclass that uses `IBatfish` loading APIs and returns an answer element.
4. A `QuestionPlugin` subclass annotated for service loading.
5. A JSON template under `questions/stable` when user-facing and mature, or `questions/experimental` when not stable.
6. Unit tests for question parameters, answer rows, metadata, and differential behavior if supported.

## 10. Service, Coordinator, Client, and Operations

### Runtime service model

The coordinator is a RESTful work manager. `WorkMgrServiceV2` states that it services client API calls and provides information about containers, testrigs, questions, and analyses, including create/delete or modification paths for authenticated clients. It is mounted under the V2 work-manager service path and returns JSON. `Main.startWorkManagerService` creates a Grizzly HTTP server with Jersey resources and required features.[^17]

The coordinator uses `FileBasedStorage` at the configured containers location and a storage-backed ID manager. It starts `WorkMgr`, then starts the REST service. `Main.mainRun` initializes the authorizer, initializes the work manager, triggers garbage collection, and keeps the process alive.[^8]

### Coordinator/client split

The coordinator owns persistent orchestration: networks, snapshots, questions, answers, work queue, task status, topology objects, and storage IDs. The Java client owns command processing and local input validation. The inspected `Client` source validates template variables and imports coordinator helper classes, work item builders, zip utilities, and datamodel question types. This split keeps storage and queue semantics server-side while allowing client-side convenience validation.[^10]

### REST/API surfaces visible in code

Visible REST surfaces include:

| Resource | Example source | Responsibility |
| --- | --- | --- |
| `WorkMgrServiceV2` | `projects/coordinator/src/main/java/org/batfish/coordinator/WorkMgrServiceV2.java` | Networks, containers alias, question templates, version, network resource routing, network initialization. |
| `AnswerResource` | `projects/coordinator/src/main/java/org/batfish/coordinator/resources/AnswerResource.java` | Fetch or filter a stored answer for a question/snapshot/reference snapshot. |
| Network resource tree | `WorkMgrServiceV2.getNetworkResource` returns `NetworkResource` | Nested network/snapshot/question resources; not exhaustively inspected here. |

This report does not claim the table is an exhaustive REST API reference. It identifies representative source-confirmed surfaces.[^17]

### Snapshot and storage model

The operational object hierarchy is network -> snapshot -> question -> work/answer. `WorkMgr` assigns IDs through `IdManager`, stores snapshot input objects under a `NetworkSnapshot`, stores question content, derives `AnswerId`s from network/snapshot/question/node-role/reference-snapshot IDs, and loads answers only when metadata exists. Snapshot metadata includes creation time and optional parent snapshot ID for forked snapshots.[^8]

### All-in-one packaging

`AllInOne` composes the runtime. It parses all-in-one settings, constructs Java client arguments, creates a `Client`, starts a coordinator thread, starts a Batfish worker thread through `Driver.main`, wires coordinator work execution to `Driver.getBatfishWorkerService()`, then either runs the client or sleeps indefinitely when not running the client.[^21]

### Docker and Pybatfish ecosystem context

The repository README tells users to run `batfish/allinone` Docker and then install Pybatfish to interact with the Batfish service. The build docs state that Docker images are built from the separate `batfish/docker` repository and published to Docker Hub, while this repository contains vulnerability scanning workflows and service source code. This report therefore treats Docker packaging and Pybatfish as adjacent ecosystem context, not primary repository subjects.[^4]

### Configuration and flags

The inspected sources show multiple settings classes rather than one global configuration surface: all-in-one settings construct client, coordinator, and Batfish worker argument strings; coordinator settings drive the work-bind host, work V2 port, containers location, authorizer type, and question template dirs; client settings drive log level, log file, command file, snapshot dir, and coordinator port. This report did not inspect every setting constant or CLI flag, so it does not provide an exhaustive flag reference.[^17]

## 11. Build, Dependencies, and Developer Workflow

### Bazel/Bazelisk model

The repository uses Bazel/Bazelisk, not Maven or Gradle, as the primary build system. `.bazelversion` pins Bazel to `9.1.0`. `MODULE.bazel` declares the Bazel module, Java rules, Python rules, Maven dependency installation, Python toolchain, pip lockfile, and development tool dependencies. The build docs list Bazelisk as a prerequisite and describe building all-in-one through Bazel.[^7]

### Java and Python requirements

The build and development docs list Java 17+ and Python 3.10+ as prerequisites. `MODULE.bazel` configures the Python toolchain to `3.10` and parses `python:requirements.txt` as the pip requirements lock. The development docs identify Java as the primary language, ANTLR as the parser generator, and Python as primarily Pybatfish/client-side ecosystem support.[^7]

### Dependency declaration and pinning

`MODULE.bazel` contains Maven artifacts and BOMs, including ANTLR, Jackson, Guava, Jersey/Grizzly, JavaBDD-related dependencies, JUnit, Mockito, parboiled, PMD, Log4j, and others. It uses `rules_jvm_external` with a `maven_install.json` lock file and `strict_visibility = True`. The build docs describe dependency repinning through `REPIN=1 bazel run @maven//:pin`.[^7]

### Generated code and grammar generation

ANTLR is declared through Maven artifacts and is used by parser support code. The parser docs and build files show parser grammar organization and parser common support, but this report did not inspect every ANTLR generation rule. The safe claim is that grammar-generated parser/lexer code is part of the build workflow and governed by Bazel, ANTLR artifacts, and vendor grammar packages.[^12]

### Build/test/operation cheat sheet

| Task | Representative command or source-grounded pattern | Notes |
| --- | --- | --- |
| Run all tests | `bazel test //...` | Listed in build docs. |
| Test one subtree | `bazel test //projects/batfish/...` | Build docs show subtree test pattern. |
| Test one target | `bazel test //projects/batfish/src/test/java/org/batfish/grammar/cisco_xr:XrGrammarTest` | Build docs show single-target style; exact target names must be confirmed from BUILD files. |
| Filter one Java test | `bazel test <target> --test_filter=<method-or-class-filter>` | Build docs include filter form. |
| Build all-in-one | `bazel build //projects/allinone:allinone_main` | Build docs use this target. |
| Run all-in-one service | `tools/bazel_run.sh run //projects/allinone:allinone_main -- -runclient false ...` | Build docs give service-run pattern with templatedirs and containers location. |
| Re-pin Maven deps | `REPIN=1 bazel run @maven//:pin` | Build docs and `MODULE.bazel` lockfile model. |
| Format Java | `./tools/fix_java_format.sh --replace` | Pre-commit hook invokes this script. |
| Run buildifier fix | `bazel --noblock_for_lock run //:buildifier.fix` | Pre-commit hook invokes this command. |
| Python lock update | `pip-compile` from `python/requirements.in` to hashed `requirements.txt` | Python README describes this lockfile workflow. |

### Formatting, linting, and pre-commit

The pre-commit configuration includes local Java formatting through `tools/fix_java_format.sh --replace`, buildifier fixes, and Python formatting/linting hooks such as black, isort, and autoflake. The tools directory also contains checkstyle and reference-update scripts. This supports a Bazel-first developer workflow with local formatting hooks rather than Maven/Gradle lifecycle plugins.[^7]

## 12. Testing and Validation Strategy

### Test organization

Testing is distributed across project-specific `src/test/java` trees and top-level fixtures under `tests/`. The top-level `tests/` directory includes parsing tests, parsing-error tests, question fixtures, roles, logging, basic, and AWS fixtures. `projects/bdd` and `projects/minesweeper` include their own unit and regression tests. Parser/conversion behavior is heavily represented in vendor grammar tests such as `XrGrammarTest`, which imports parse-warning matchers, configuration matchers, structure-reference matchers, route/filter/ACL matchers, data-plane classes, and vendor representation constants.[^22]

### Parser and reference tests

Parser tests validate more than syntactic acceptance. The representative Cisco XR grammar test imports matchers for parse warnings, undefined references, defined structures, red-flag warnings, route filters, ACL acceptance/rejection, configuration format, interfaces, BGP/OSPF route properties, and tracking groups. This indicates that grammar tests often combine syntax recognition, extraction, structure tracking, conversion output, and warning expectations.[^22]

### Conversion and data-plane tests

The conversion job contains many `@VisibleForTesting` methods and assertions, including IP-space reference collection, invalid vendor-structure ID checks, and finalization helpers. Data-plane code exposes testing hooks for VXLAN autostate and track comparison. The build docs support targeted tests by package, target, and filter, which is important because full network/data-plane tests can be expensive.[^14]

### BDD and symbolic tests

`projects/bdd` has direct JavaBDD tests and regression tests. `projects/minesweeper` has tests for model generation, transfer BDD logic, community/AS-path BDD translation, route-policy transfer behavior, and transfer-BDD validation questions. Packet-level BDD reachability has its own source-level helpers and is tied into Batfish question/analysis tests, although this report did not enumerate every reachability test target.[^22]

### Golden-output and reference behavior

The repository uses reference fixtures and question/test networks. The tools directory includes an `update_refs.sh` script, which indicates reference-output maintenance workflows. This report did not inspect every golden file or reference update target, so it treats golden/reference testing as confirmed by repository layout and tooling but not exhaustively mapped.[^2]

### Edge-case strategy

The codebase encodes edge cases as explicit warnings, parse statuses, finalization checks, and test matchers. Examples include empty/ignored/unknown/unimplemented formats in parsing, undefined IP-space references converted to empty IP spaces with warnings, undefined routing-policy and track references removed with warnings, invalid OSPF interface references pruned, ACL cyclic-reference detection in BDD conversion, and BDP oscillation exceptions in data-plane computation.[^11]

## 13. Internal Component Relationship Analysis

### Core versus common

`projects/batfish` is the worker-side implementation. It owns jobs, engines, parser dispatch, data-plane computation, BDD reachability factory integration, and analysis execution. `projects/common` is the shared contract and model layer. It contains `IBatfish`, `DataPlanePlugin`, `Question`, `Answerer`, data model classes, topology interfaces, grammar support base classes, and BDD helper classes. The boundary is not perfectly clean because `projects/common/src/main/java/BUILD.bazel` comments that much common code is cyclically dependent and must be compiled as one unit. That is a source-confirmed warning against overstating modular purity.[^23]

### Coordinator versus client

`projects/coordinator` is server-side state and work orchestration. It validates and stores networks, snapshots, questions, answers, metadata, and work status. `projects/client` is a Java client interface that validates parameters and coordinates user commands with the service. The coordinator exposes REST resources; the client uses helper classes and settings to submit work and interpret responses. All-in-one composes them but does not collapse their responsibilities.[^17]

### `projects/question` versus `questions/`

`projects/question` contains Java question plugins and answerers. `questions/` contains JSON templates that users and clients load. A template identifies a Java question class and declares user-facing instance metadata and variables. The Java class defines behavior; the JSON template defines the user-facing invocation surface.[^6]

### Symbolic, BDD, and Minesweeper boundaries

`projects/bdd` provides the low-level JavaBDD implementation. `projects/symbolic` provides symbolic state and ingress abstractions used by packet reachability. `projects/common` provides packet BDD wrappers such as `BDDPacket`, `HeaderSpaceToBDD`, and `IpAccessListToBdd`. `projects/batfish` contains the BDD reachability graph factory and analysis. `projects/minesweeper` provides route-policy/control-plane symbolic analysis over route advertisements. These are related through BDDs and the VI model, but their modeled domains differ: packets for reachability, routes for Minesweeper.[^18]

### All-in-one composition

`projects/allinone` is an operational composition layer. It starts the client, starts the coordinator, starts a Batfish worker, wires the coordinator work executor to the worker service, and optionally runs the client. It is not a separate model or analysis layer.[^21]

### Apparent lineage and subsystem boundaries

The code and docs retain legacy terms such as “container” and “testrig” alongside newer “network” and “snapshot” terminology. `WorkMgrServiceV2` has deprecated container endpoints that redirect to network equivalents, and `WorkMgr` comments still use testrig terminology in some contexts. This indicates a gradual terminology migration rather than a clean historical reset.[^17]

### Ecosystem context

The README and official build docs identify two adjacent ecosystem components: Pybatfish as the Python SDK used to interact with the service, and a separate `batfish/docker` repository for Docker image construction. The current repository’s `python/` directory contains dependency lockfiles, but the README’s Pybatfish link points outside the primary repository. The report does not analyze Pybatfish or Docker packaging as peer subjects because the user supplied only `batfish/batfish`.[^4]

## 14. Design Patterns and Abstractions

### Compiler-like pipeline

Batfish has a source-to-model pipeline analogous to a compiler: lexical/parsing, parse tree, semantic extraction, vendor-specific IR, vendor-independent IR, validation/finalization, and executable analysis. The pattern is source-grounded by parser/extractor docs, `ParseVendorConfigurationJob`, `ControlPlaneExtractor`, `VendorConfiguration`, and `ConvertConfigurationJob`.[^3]

### Plugin and registry patterns

Plugins are used for data-plane engines and questions. `DataPlanePlugin` registers with Batfish during initialization. `QuestionPlugin` registers answerers with Batfish and questions with the client. AutoService annotations on plugins allow service-loader discovery. `IBatfish` exposes registration methods for answerers, BGP table plugins, data-plane plugins, and external BGP advertisement plugins.[^10]

### ANTLR listener/visitor patterns

Parsing and extraction are ANTLR-centered. Grammar docs prescribe lexer/parser conventions. `ControlPlaneExtractor` can extend an ANTLR base listener or wrap listeners. `BatfishParseTreeWalker` customizes parse-tree walking and error context. Conversion and BDD encoding also use visitor patterns over ACLs, IP spaces, routing-policy structures, transformations, and symbolic states.[^11]

### Job/task abstractions and parallel processing

Parsing and conversion are represented as jobs. `ParseVendorConfigurationJob` and `ConvertConfigurationJob` return result objects with elapsed time, logs, warnings, and model data or exceptions. Conversion parallelizes vendor-independent configurations from a single vendor configuration, and data-plane computation uses parallel streams over virtual routers in places. Work coordination is separate through coordinator queues and task handles.[^8]

### Immutable and builder-style data models

Many data structures use builders, immutable collections, and JSON annotations. `Configuration.Builder` constructs configurations; `ConvertConfigurationJob.finalizeConfiguration` converts many maps and sets into immutable sorted structures. Forwarding behavior classes use immutable maps and builder defaults. BDD and symbolic objects also use immutable maps/tables for computed results.[^16]

### Warning and error aggregation

Warnings are aggregated rather than discarded. Parsing captures parse warnings and error details. Vendor configurations carry warnings and structure managers. Conversion merges per-configuration warnings deterministically. Many validators remove invalid or undefined references and emit red flags. This supports partial modeling while making partial support visible.[^11]

### Snapshot-based workflow

All analysis is snapshot-scoped. `NetworkSnapshot` is a recurring API type. Coordinator storage keys include network, snapshot, question, answer, node roles, and optional reference snapshot IDs. Differential questions use base and reference snapshots.[^8]

### Graph modeling

Topology and reachability are graph models. Layer 2 uses union-find to compute broadcast-domain representatives. Layer 3 topology is edge-based. Protocol topologies are graph-like neighbor structures. BDD reachability builds a state graph with `StateExpr` nodes and transition-labeled edges. Data-plane computation also treats topology as an iterative graph product that can change with routing reachability.[^5]

### Serialization and schema patterns

Questions and answers are JSON-serializable. `Question` uses Jackson class metadata. Tables use `TableMetadata`, `ColumnMetadata`, `Schema`, and `Row`. The coordinator persists and returns serialized answers. The Java client validates JSON template variable types. This gives Batfish a schema-driven answer surface even when internal models are complex.[^10]

### Where to change the code

| Change goal | Primary files or packages | Secondary files or tests |
| --- | --- | --- |
| Add vendor syntax support | Vendor grammar and parser package; vendor `ControlPlaneExtractor`; vendor representation; conversion code; `ParseVendorConfigurationJob` if a new format or parser dispatch is needed | Parser tests, conversion tests, warning tests, reference fixtures, data-plane/question tests if behavior changes. |
| Add a new question | `projects/question/src/main/java/...` or specialized subsystem package; `Question`, `Answerer`, `QuestionPlugin`; `questions/stable` or `questions/experimental` template | Question tests, template tests, table metadata assertions, differential tests if applicable. |
| Modify data-plane behavior | `projects/batfish/src/main/java/org/batfish/dataplane/ibdp`; `IncrementalBdpEngine`, `VirtualRouter`, protocol helpers, RIB classes | Data-plane tests, topology tests, protocol-specific tests, question tests that depend on routes/FIBs. |
| Modify topology behavior | `projects/common/src/main/java/org/batfish/common/topology`; topology utilities and protocol topology packages; iBDP topology context | Topology tests, data-plane tests, forwarding/traceroute tests. |
| Modify symbolic reachability | `BDDReachabilityAnalysisFactory`, `BDDReachabilityAnalysis`, transition classes, `projects/symbolic`, common BDD converters | BDD reachability tests, ACL/HeaderSpace BDD tests, transformation/NAT tests, question tests. |
| Modify route-policy symbolic analysis | `projects/minesweeper/src/main/java/org/batfish/minesweeper/bdd`; route-policy question packages | Minesweeper BDD tests, search/compare route-policy question tests, model-generation tests. |
| Modify service/client behavior | `WorkMgrServiceV2`, resource classes, `WorkMgr`, `Client`, `BfCoordWorkHelper`, all-in-one settings | Coordinator resource tests, client command tests, work queue tests, answer retrieval tests. |

## 15. Applicability and Transferable Lessons

### Network validation architecture

Batfish shows how a validation system can avoid device access by analyzing configuration snapshots and optional external artifacts. The useful transferable pattern is snapshot isolation: every analysis has explicit input state and can be rerun, diffed, cached, or forked. The tradeoff is that snapshot packaging and input normalization become critical correctness boundaries.[^8]

### Compiler-like translation from DSL to normalized models

The parser/extractor/converter pipeline is broadly reusable for systems that ingest many vendor DSLs. It allows staged support, warnings for partial support, and a stable internal model. The tradeoff is implementation cost: adding support requires grammar, extraction, representation, conversion, and tests rather than one parser function.[^12]

### Vendor abstraction and normalization

The VI model decouples network analysis from vendor syntax. This is useful in any multi-provider system where behavior matters more than configuration syntax. The tradeoff is semantic loss or approximation risk: conversion must preserve behavior, and unsupported or ambiguous vendor features must be surfaced through warnings rather than silently normalized incorrectly.[^14]

### Question/answer framework

The question framework separates user-facing templates, Java question classes, answerers, schemas, and serialized answers. This pattern is reusable for analysis systems that need a stable API surface over many internal analyses. The tradeoff is dual maintenance: templates and Java classes must remain aligned.[^10]

### Symbolic reachability

BDD reachability shows how to replace packet enumeration with symbolic sets of packet headers. This is valuable when input spaces are huge and correctness requires universal or existential reasoning over many flows. The tradeoff is BDD memory sensitivity, variable-order effects, and the need for careful ownership of BDD objects.[^19]

### Graph/topology modeling

Layered topology modeling is transferable to other infrastructure-analysis domains. Physical, logical, overlay, and protocol graphs should not be collapsed prematurely because each layer has different failure modes and computation rules. The tradeoff is cross-layer complexity: changes in overlay reachability can feed back into Layer 3 topology and data-plane convergence.[^15]

### Regression and golden-output testing

Batfish’s testing culture shows the value of parser tests that assert warnings, structures, conversion results, and data-plane behavior, not just parse success. This is transferable to any DSL ingestion system. The tradeoff is maintenance cost when intended behavior changes, because reference outputs and fixture expectations must be updated deliberately.[^22]

### Operational packaging

All-in-one composition and Docker-based service startup are useful for complex systems whose production deployment has multiple components. The tradeoff is that all-in-one can hide service boundaries during development; source changes still need to respect coordinator, worker, and client separation.[^21]

## 16. Confirmed Findings vs Inferences

| Finding | Classification | Evidence | Confidence | Notes |
| --- | --- | --- | --- | --- |
| The repository is a single primary Batfish repository with top-level `projects`, `docs`, `questions`, `python`, `tests`, `tools`, and `networks`. | Confirmed | GitHub repo root and directory pages. | High | Only one primary repo was supplied. |
| The inspected commit is `497c920d648f14cfe41d5cb5e13c133b024cd416`. | Confirmed | GitHub connector repository/compare metadata. | High | Local clone was unavailable. |
| Build is Bazel/Bazelisk-centered and `.bazelversion` pins `9.1.0`. | Confirmed | `.bazelversion`, `MODULE.bazel`, build docs. | High | Not Maven/Gradle-centered. |
| Batfish uses a staged parsing, extraction, conversion, post-processing, data-plane, forwarding-analysis architecture. | Confirmed | Architecture docs and source jobs/interfaces. | High | Source and docs align at high level. |
| `projects/batfish` owns worker-side parser dispatch, conversion jobs, and data-plane computation. | Confirmed | `ParseVendorConfigurationJob`, `ConvertConfigurationJob`, `IncrementalDataPlanePlugin`, `IncrementalBdpEngine`. | High | Representative source inspected. |
| `projects/common` owns shared data models, plugin APIs, grammar base classes, topology abstractions, and BDD helpers. | Confirmed | `IBatfish`, `DataPlanePlugin`, `Question`, `Configuration`, grammar base, topology, BDD utility sources. | High | Build file warns common code has cyclic dependencies. |
| `projects/bdd` is a vendored JavaBDD implementation. | Confirmed | `projects/bdd/BUILD.bazel` alias and `net/sf/javabdd` build/source paths. | High | Direct source/build evidence. |
| `projects/symbolic` provides symbolic state and ingress abstractions. | Confirmed | `StateExpr`, `StateExprVisitor`, `IngressLocation`, `projects/symbolic/BUILD.bazel`. | High | Direct source/build evidence. |
| `projects/minesweeper` models symbolic route advertisements and route-policy transfer, not packet forwarding. | Confirmed | `BDDRoute`, `TransferBDD`, `SearchRoutePoliciesAnswerer`. | High | It uses route attributes and route-policy classes. |
| Data-plane computation uses fixed-point iteration and caps topology iterations at 10. | Confirmed | `IncrementalBdpEngine.MAX_TOPOLOGY_ITERATIONS`. | High | Source constant and convergence logic inspected. |
| Question templates map to Java question classes and answerers. | Confirmed | `questions/README`, question development docs, `Question`, `QuestionPlugin`, `Answerer`. | High | Direct docs and source evidence. |
| Coordinator storage and work orchestration are snapshot/network/question ID-based. | Confirmed | `WorkMgr`, `WorkMgrServiceV2`, `AnswerResource`. | High | Representative service and storage paths inspected. |
| The architecture is compiler-like. | Source-grounded inference | Stages and source classes align with compiler pipeline concepts. | High | “Compiler-like” is an analytic description, not repository terminology in every source. |
| Topology docs and `TopologyContainer` source may be out of sync regarding Layer 2 topology exposure. | Confirmed mismatch | Topology docs describe L2 topology in container; inspected source lacks `getLayer2Topology`. | Medium-high | More source history could explain whether docs are stale or interface changed. |
| Full parser support for every vendor was not exhaustively reconstructed. | Unresolved | Only representative parser docs/source/tests were inspected. | Medium | Requires per-vendor grammar/extractor walk. |
| Full REST API surface was not exhaustively reconstructed. | Unresolved | Representative resources inspected; nested resource tree not fully enumerated. | Medium | Requires full coordinator resources scan. |
| All generated code behavior was not verified locally. | Unresolved | No local clone/build/test due network resolution failure in container. | Medium | GitHub source inspection still supports architecture-level findings. |

## 17. Open Questions and Further Investigation

### Areas needing deeper source inspection

1. Full vendor grammar inventory. A complete vendor-support report would enumerate every grammar, extractor, representation package, conversion package, and test class per vendor.
2. Full REST API resource tree. This report inspected `WorkMgrServiceV2`, `AnswerResource`, `WorkMgr`, and coordinator main flow, but not every nested resource class.
3. Data-plane protocol internals. This report inspected the iBDP engine and `VirtualRouter` state but did not fully walk every BGP, OSPF, ISIS, EIGRP, RIP, redistribution, and RIB helper class.
4. Every question plugin. The report uses `IpOwners` and Minesweeper route-policy questions as representative examples, but it does not enumerate every question class and template.
5. Serialization and cache keys for all computed artifacts. `WorkMgr` and `IBatfish` expose storage/loading behavior, but complete cache invalidation and artifact dependency rules require deeper storage-provider inspection.

### Ambiguous subsystem boundaries

The `projects/common` build file’s comment about cyclic dependencies means “common” is not a pure interface-only module. It contains shared contracts, shared implementation helpers, and broad data-model code compiled together. Developers should verify dependency direction before moving code into or out of `projects/common`.[^23]

The BDD subsystem spans multiple projects. Low-level BDD classes are in `projects/bdd`, packet BDD helpers are in `projects/common`, symbolic states are in `projects/symbolic`, packet reachability graph construction is in `projects/batfish`, and symbolic route-policy analysis is in `projects/minesweeper`. Treating “BDD” as one module would hide important responsibility boundaries.[^18]

### Potentially outdated docs

The clearest source/documentation mismatch found is the Layer 2 topology exposure in `TopologyContainer`. The topology docs discuss a container with Layer 2 topology, while the inspected source interface lacks a Layer 2 accessor. A maintainer should decide whether documentation should be updated or whether another code path exposes Layer 2 topology outside the inspected interface.[^5]

### External companion repositories

The README and official build docs identify Pybatfish and Docker packaging as ecosystem components. A full ecosystem report would inspect Pybatfish and `batfish/docker`, but those repositories were not primary inputs and are intentionally outside this report’s scope.[^4]

## Sources

[^1]: `batfish/batfish` GitHub repository root, branch and directory listing, GitHub web view lines 150-320; README excerpt lines 330-388; GitHub connector repository metadata and compare metadata for commit `497c920d648f14cfe41d5cb5e13c133b024cd416`; `projects/VERSION`, fetched lines 1-5; `.bazelversion`, fetched line 1. Source limit: metadata was inspected remotely, not from a local clone.
[^2]: GitHub directory pages for `projects/`, `docs/`, `python/`, `questions/`, `tests/`, `tools/`, and `networks/`, inspected through GitHub web views and GitHub connector. Relevant observed listings included `projects` subdirectories `allinone`, `batfish`, `bdd`, `client`, `common`, `coordinator`, `minesweeper`, `question`, `symbolic`; docs directories for architecture/parsing/extraction/conversion/data-plane/topology/symbolic/development/build; and top-level fixture/tool/example directories.
[^3]: `docs/architecture/README.md`, GitHub web view lines 253-308; `projects/batfish/src/main/java/org/batfish/job/ParseVendorConfigurationJob.java`, fetched lines 1-320, 320-720, and 720-860; `projects/common/src/main/java/org/batfish/vendor/VendorConfiguration.java`, fetched lines 1-260 and 260-520; `projects/batfish/src/main/java/org/batfish/job/ConvertConfigurationJob.java`, fetched lines 1-320, 320-720, and 1000-1260; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalDataPlanePlugin.java`, fetched lines 1-220; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalBdpEngine.java`, fetched lines 1-260 and 260-620.
[^4]: `README.md`, GitHub web view lines 367-388 for all-in-one Docker and Pybatfish; `docs/building_and_running/README.md`, GitHub web view lines 387-431 for Docker image repository context; `questions/README.md`, GitHub web view lines 271-273 for Pybatfish recommendation and Java-client deprecation note.
[^5]: `docs/topology/README.md`, GitHub web views covering topology layers, Layer 1 views, Layer 2 union-find, Layer 3 topology, overlays, protocol topologies, and topology workflow; `projects/common/src/main/java/org/batfish/common/topology/TopologyContainer.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/common/topology/Layer1Topologies.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/common/topology/Layer2Topology.java`, fetched lines 1-220. Source limit: the mismatch is between inspected docs and inspected interface only; history was not inspected.
[^6]: `questions/README.md`, GitHub web view lines 221-288; `docs/question_development/README.md`, GitHub web view lines 277-322; `projects/coordinator/src/main/java/org/batfish/coordinator/Main.java`, fetched lines 1-280, for question-template loading.
[^7]: `.bazelversion`, fetched line 1; `MODULE.bazel`, fetched lines 1-240; `docs/building_and_running/README.md`, GitHub web view lines 244-252, 322-385; `docs/development/README.md`, GitHub web view lines 264-291; `.pre-commit-config.yaml`, GitHub web view lines 304-372; `python/README.md`, GitHub web view lines 221-258.
[^8]: `projects/coordinator/src/main/java/org/batfish/coordinator/WorkMgr.java`, fetched lines 1-320, 320-760, and 760-1260; `projects/coordinator/src/main/java/org/batfish/coordinator/Main.java`, fetched lines 1-280 and 280-420; `projects/coordinator/src/main/java/org/batfish/coordinator/resources/AnswerResource.java`, fetched lines 1-240.
[^9]: `projects/batfish/src/main/java/org/batfish/job/ParseVendorConfigurationJob.java`, fetched lines 1-320, 320-720, and 720-860; `projects/common/src/main/java/org/batfish/vendor/VendorConfiguration.java`, fetched lines 1-260 and 260-520; `projects/batfish/src/main/java/org/batfish/job/ConvertConfigurationJob.java`, fetched lines 1-320, 320-720, and 1000-1260; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalDataPlanePlugin.java`, fetched lines 1-220; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalBdpEngine.java`, fetched lines 1-260 and 260-620.
[^10]: `projects/common/src/main/java/org/batfish/common/Answerer.java`, fetched lines 1-200; `projects/question/src/main/java/org/batfish/question/QuestionPlugin.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/datamodel/questions/Question.java`, fetched lines 1-240 and 241-340; `projects/question/src/main/java/org/batfish/question/ipowners/IpOwnersQuestionPlugin.java`, fetched lines 1-220; `projects/question/src/main/java/org/batfish/question/ipowners/IpOwnersAnswerer.java`, fetched lines 1-220; `projects/client/src/main/java/org/batfish/client/Client.java`, fetched lines 1-260.
[^11]: `docs/parsing/README.md`, GitHub web view lines 267-320 and 650-802; `docs/extraction/README.md`, GitHub web views lines 243-343 and 398-520; `projects/common/src/main/java/org/batfish/grammar/ControlPlaneExtractor.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/grammar/BatfishExtractor.java`, fetched lines 1-160; `projects/common/src/main/java/org/batfish/grammar/BatfishParseTreeWalker.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/common/Warnings.java`, fetched lines 1-260 and 260-380; `projects/batfish/src/main/java/org/batfish/job/ParseVendorConfigurationJob.java`, fetched lines 1-320, 320-720, and 720-860.
[^12]: `docs/parsing/README.md`, GitHub web view lines 300-530 and 650-802; `projects/common/src/main/java/BUILD.bazel`, fetched lines 1-180; `projects/common/src/main/java/org/batfish/grammar/BatfishCombinedParser.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/grammar/BatfishLexer.java`, fetched lines 1-180.
[^13]: `projects/common/src/main/java/org/batfish/vendor/VendorConfiguration.java`, fetched lines 1-260 and 260-520; `docs/extraction/README.md`, GitHub web view lines 243-343.
[^14]: `docs/conversion/README.md`, GitHub web view lines 248-450; `projects/batfish/src/main/java/org/batfish/job/ConvertConfigurationJob.java`, fetched lines 1-320, 320-720, and 1000-1260.
[^15]: `docs/data_plane/README.md`, GitHub web views lines 252-383, 410-550, 496-682, and 688-739; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalDataPlanePlugin.java`, fetched lines 1-220; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/IncrementalBdpEngine.java`, fetched lines 1-260 and 260-620; `projects/batfish/src/main/java/org/batfish/dataplane/ibdp/VirtualRouter.java`, fetched lines 1-220.
[^16]: `projects/common/src/main/java/org/batfish/datamodel/Configuration.java`, fetched lines 1-220 and 220-430; `projects/common/src/main/java/org/batfish/datamodel/DataPlane.java`, fetched lines 1-220; `projects/common/src/main/java/org/batfish/datamodel/ForwardingAnalysis.java`, fetched lines 1-240; `projects/common/src/main/java/org/batfish/datamodel/VrfForwardingBehavior.java`, fetched lines 1-240; `projects/common/src/main/java/org/batfish/datamodel/InterfaceForwardingBehavior.java`, fetched lines 1-220.
[^17]: `projects/coordinator/src/main/java/org/batfish/coordinator/WorkMgrServiceV2.java`, fetched lines 1-220; `projects/coordinator/src/main/java/org/batfish/coordinator/Main.java`, fetched lines 1-280 and 280-420; `projects/coordinator/src/main/java/org/batfish/coordinator/WorkMgr.java`, fetched lines 1-320, 320-760, and 760-1260; `projects/coordinator/src/main/java/org/batfish/coordinator/resources/AnswerResource.java`, fetched lines 1-240; `projects/client/src/main/java/org/batfish/client/Client.java`, fetched lines 1-260.
[^18]: `projects/batfish/src/main/java/org/batfish/bddreachability/BDDReachabilityAnalysisFactory.java`, fetched lines 1-280 and 280-620; `projects/batfish/src/main/java/org/batfish/bddreachability/BDDReachabilityAnalysis.java`, fetched lines 1-260; `projects/batfish/src/main/java/org/batfish/bddreachability/transition/TransformationToTransition.java`, fetched lines 1-300; `projects/symbolic/src/main/java/org/batfish/symbolic/state/StateExpr.java`, fetched lines 1-160; `projects/symbolic/src/main/java/org/batfish/symbolic/state/StateExprVisitor.java`, fetched lines 1-260; `projects/symbolic/src/main/java/org/batfish/symbolic/IngressLocation.java`, fetched lines 1-260; `projects/symbolic/BUILD.bazel`, fetched lines 1-200.
[^19]: `projects/common/src/main/java/org/batfish/common/bdd/BDDPacket.java`, fetched lines 1-260; `projects/common/src/main/java/org/batfish/common/bdd/IpAccessListToBdd.java`, fetched lines 1-320; `projects/common/src/main/java/org/batfish/common/bdd/IpAccessListToBddImpl.java`, fetched lines 1-280; `projects/common/src/main/java/org/batfish/common/bdd/HeaderSpaceToBDD.java`, fetched lines 1-260; `projects/bdd/BUILD.bazel`, fetched lines 1-200; `projects/bdd/src/main/java/net/sf/javabdd/BUILD.bazel`, fetched lines 1-200; GitHub connector search results for `projects/bdd/src/main/java/net/sf/javabdd/BDD.java`, `BDDFactory.java`, `JFactory.java`, `BDDPairing.java`, and tests.
[^20]: `projects/minesweeper/src/main/java/org/batfish/minesweeper/bdd/BDDRoute.java`, fetched lines 1-260; `projects/minesweeper/src/main/java/org/batfish/minesweeper/bdd/TransferBDD.java`, fetched lines 1-260; `projects/minesweeper/src/main/java/org/batfish/minesweeper/question/searchroutepolicies/SearchRoutePoliciesAnswerer.java`, fetched lines 1-260; GitHub connector search results for Minesweeper tests and route-policy question packages.
[^21]: `projects/allinone/src/main/java/org/batfish/allinone/AllInOne.java`, fetched lines 1-260.
[^22]: `tests/` directory listing, GitHub web view lines 221-260; `projects/batfish/src/test/java/org/batfish/grammar/cisco_xr/XrGrammarTest.java`, fetched lines 1-220; `docs/building_and_running/README.md`, GitHub web view lines 352-375; GitHub connector search results for `projects/bdd/src/test/java/...` and `projects/minesweeper/src/test/java/...`.
[^23]: `projects/common/src/main/java/BUILD.bazel`, fetched lines 1-180, especially the `common_lib` comment that common code is cyclically dependent and compiled as one unit.
