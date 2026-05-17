# Open Decision Ledger

This ledger is generated from PRD Section 24. Each row blocks authoritative status for the owning spec until resolved, explicitly deferred, or removed by a later accepted decision record.

| Decision | Required resolution before implementation | Owner |
| --- | --- | --- |
| Feed feasibility catalog | Define the feed feasibility assessment rows for every MVP feed: availability, payload fidelity, metadata sufficiency, timestamp sufficiency, identifier sufficiency, schema stability, replayability, fixture coverage, completeness evidence, and parser/mapping readiness. | `020,120` |
| Raw supplier profile set | Define public supplier classes and private supplier-binding rules without exposing sensitive source inventory in the public PRD. | `010,020` |
| Lakehouse read policy catalog | Define supported read target kinds, manifest validation policies, checksum policies, timeout bounds, failed-object behavior, and replay policy rows. | `020` |
| Raw feed manifest schema | Finalize required counts, hashes, time bounds, schema refs, supplier lineage, upstream completeness evidence refs, and manifest checksum serialization. | `020` |
| Feed completeness evidence policy | Define which upstream evidence classes are sufficient for each source category, dataset, fact type, predicate, and scope. | `060` |
| CDC-shaped feed metadata mapping | Define how upstream CDC metadata contained in lakehouse feeds maps to `RawRecord.source_event_metadata`, replay state, parsing requirements, and non-authority rules. | `020,030,080` |
| Lakehouse feed fixture corpus | Build positive, malformed, unsupported, duplicate, stale, partial, permission-limited, upstream-completeness-available, and upstream-completeness-unavailable fixtures from redacted lakehouse feed rows and manifests. | `020,120` |
| Private source-feed binding inventory | Maintain concrete source and supplier route mappings only in the private implementation repository. | `010,020,110` |
| Feed availability check freshness | Define TTL and invalidation rules for table-backed, object-backed, partition-backed, and manifest-backed feeds. | `020` |
| Parser and mapping readiness | Prove that active parser and mapping bundles can map feed records into OCSF-aligned silver observations and Cadastre source-extension fields without vendor-specific leakage into gold facts. | `050,120` |
| Completeness UI wording | Define user-facing wording for feed missing, feed stale, supplier degraded, upstream completeness unavailable, source absence not authorized, and unknown/not observed states. | `060,110` |
| Acceptance test harness | Add no-direct-source-call, feed manifest, raw import identity, completeness rejection, replay-without-source, private binding leak, and feed fixture validation cases to CI and production readiness checks. | `120` |
| Temporal policy catalog | Define source-dataset-specific `TemporalSemanticsPolicy` rows, allowed time fields, fallback behavior, historical import behavior, and malformed/ambiguous handling. | `080` |
| Knowledge-time import mode | Decide which datasets, if any, may reconstruct prior known time and what source-known-time evidence is sufficient. | `080` |
| Bitemporal query mode catalog | Define active query modes, assertion-state visibility, default times, audit requirements, page-token scope, and ordering. | `080,110` |
| Gold correction policy catalog | Define comparable keys, interval overlap behavior, confidence-only changes, delete evidence behavior, conflict behavior, stale behavior, and sort keys by fact type and predicate. | `080` |
| Correction snapshot ref policy | Define old/new table snapshot roles, table-set checksum requirements, retention protection, and mutable-ref rejection for every correction class. | `080` |
| Late-arrival policy catalog | Define allowed lateness and authoritative/non-authoritative routing by dataset, source authority class, and fact type. | `080` |
| Replay equivalence policy catalog | Define output-class-specific included manifest fields, excluded volatile fields, hash algorithms, failure precedence, and shadow-output rules. | `030,080,120` |
| Replay input sufficiency matrix activation | Confirm required replay inputs and specific failure codes for raw, silver, temporal resolution, identity, gold, correction, graph delta, graph apply, graph rebuild, projections, analysis outputs, completeness decisions, and maintenance decisions. | `030,080,120` |
| Deterministic side-effect policy | Decide which side-effect kinds are allowed, which must be rejected, and which may be replayed from recorded values. | `080` |
| Progress signal authority matrix activation | Confirm the total matrix of progress signals and whether any environment-specific signal requires a new row before production activation. | `060` |
| Projection watermark policy catalog | Define watermark kinds, completeness gates, delta-validation gates, apply-success gates, graph consistency requirements, and lag bounds. | `060,080,090` |
| CDC replay state requirements | Define required CDC state fields, schema-history refs, snapshot handoff behavior, commit-order rules, heartbeat handling, and tombstone behavior for every CDC-shaped feed. | `020,030,080` |
| Graph delta idempotency and resume policy | Define idempotency key inputs, resume checkpoint granularity, prior-result compatibility, and backend-specific no-op evidence. | `080,090` |
| Graph rebuild equivalence policy | Define included inputs, excluded volatile fields, checksum fields, canonical ordering, and consistency-check requirements for rebuild promotion. | `080,090,120` |
| Event-sequence validation corpus activation set | Finalize the exact active cases, expected outputs, expected errors/no-ops, checksums, and fixture serialization for production readiness. | `080,120` |
| Package repository model | Decide whether production packages use OCI, TUF-compatible metadata, local bundles, Git tree snapshots, or a supported combination, and define exact resolver, metadata, descriptor, digest, and evidence attachment rules. | `100` |
| Package trust root governance | Define trust root owners, authorized signer rows, signer rotation, signer deactivation, threshold rules, issuer constraints, repository scopes, and approval authority. | `100` |
| Package transparency evidence policy | Decide when transparency evidence is required, which provider or bundle forms are accepted, how offline verification works, and which missing-proof failures are non-retryable. | `100` |
| Package repository freshness and anti-rollback policy | Define freshness TTLs, timestamp expiration handling, highest-seen state scope, freeze detection, mix-and-match detection, and metadata rollback behavior by repository type. | `100` |
| Package provenance policy | Define required provenance predicate type, minimum builder policy, allowed build types, material policies, and builder authorization by package type. | `100` |
| Package SBOM policy | Define supported SBOM formats, required package types, subject matching, component/dependency graph completeness, license thresholds, vulnerability thresholds, and opaque dependency graph behavior. | `100` |
| Package compatibility matrix catalog | Finalize matrix axes, runtime protocol versions, package public API versions, graph profile compatibility, validation output checksums, trust policy versions, attestation policy versions, and SBOM policy versions by package type. | `100` |
| Package cohesion group catalog | Define which packages must activate, roll back, quarantine, or retire together for MVP and for future package classes. | `100` |
| Package-set promotion gate policy | Define required approval rows, shadow outputs, canary thresholds, health gates, failure precedence, and promotion evidence for package-set activation. | `100` |
| Package last-known-good health gates | Define exact post-activation health dimensions and durations required before a package set can become last-known-good. | `100` |
| Package rollback compatibility policy | Define rollback state, schema, dependency, validation, graph rollback or rebuild, replay retention, and approval requirements by package type. | `100` |
| Package emergency override governance | Define quorum, TTL, post-event review, allowed target refs, emergency override audit fields, and environment-specific break-glass authority while preserving signature bypass prohibition. | `100` |
| Package quarantine scope policy | Define which target kinds may be quarantined, whether rollback or replay are blocked by default, and how quarantine expiration or successor records are handled. | `100` |
| Package supply-chain validation fixtures | Build positive and negative fixtures for package-set activation, trust, signatures, repository freshness, anti-rollback, attestation, provenance, SBOM, compatibility, rollback, quarantine, emergency override, and activation failure precedence. | `100,120` |

Source: `PRD-Cadastre.md`, lines 13635-13681 if present.
