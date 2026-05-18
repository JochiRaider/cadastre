---
doc_id: CADASTRE-NLSPEC-060
title: Source Authority, Completeness, Coverage, and Absence
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define when Cadastre may treat observations, missing rows, stale states, control results, and coverage states as authoritative.

## Explicit Non-Scope

- Identity merge logic.
- Parser output shape.
- Lakehouse read mechanics.
- Graph apply mechanics.

## Imports

- `AuthorityClass`
- `LakehouseReadCompletenessReceipt`
- `UpstreamCompletenessEvidence`
- `RawRecord`
- `CadastreSilverObservation`
- `StageStateRecord`

- `GoldFactSchema`
- `FactAbsenceOutcome`
- `EvidenceRef`
- `ActivationControlledArtifactRef`

## Exports

- `SourceAuthorityProfile`
- `SourceAuthorityProfileRow`
- `SourceStalenessPolicy`
- `SupplierCollectionVisibilityProfile`
- `ControlResultMappingRow`
- `SourceHistoryRetentionProfile`
- `DeriveAbsenceOrUnknown`
- `AbsenceDerivationPolicy`
- `AbsenceDerivationResult`
- `CoverageDimensionProfile`
- `CoverageAssertion`
- `LakehouseFeedCompletenessProfile`
- `LakehouseFeedCompletenessProfileRow`
- `ProgressSignalInterpretationPolicy`
- `ProjectionWatermarkPolicy`
- `WatermarkCommitRecord`
- `EvaluateLakehouseFeedCompleteness`
- `SourceAuthorityArtifactLifecycleGuardRows`

## Source Authority Contract

Gold derivation must use an active `SourceAuthorityProfileRow` for the exact fact type, predicate, source category, source dataset, source instance override when present, subject scope, object scope, staleness requirement, coverage requirement, and requested absence authority.

A broad source-category ranking must not act as fallback authority when the exact row is missing.

`DeriveAbsenceOrUnknown` may populate `040.GoldFact.absence_outcome` only through `040.GoldFactSchema` and only after exact authority, completeness, staleness, and coverage gates pass. Coverage-sensitive fact types or predicates must emit a `GoldFact` with non-empty `coverage_assertion_refs`; `coverage_assertion_refs = []` fails with `GOLD_FACT_COVERAGE_REQUIRED` for coverage-sensitive output.

## Completeness Contract

`LakehouseFeedCompletenessProfile` maps feed-read completeness and supplier-provided upstream evidence into source completeness decisions. A `LakehouseReadCompletenessReceipt` alone must not authorize absence, retraction, cleanup, graph expiry, or watermark advancement.

`LakehouseFeedCompletenessProfileRow` is the activation-controlled row shape that makes feed-read completeness decisions scope-exact. Missing rows, missing `allowed_effects`, or missing upstream evidence must not be interpreted as permission for absence, cleanup, retraction, graph expiry, or watermark advancement.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `profile_row_id` | Yes | none | Stable row ID scoped to `060` and included in the row-set checksum. |
| `feed_category` | Yes | none | Must match `020.LakehouseFeedCategoryClosureRow.feed_category`. |
| `source_dataset` | Yes | none | Vendor-neutral dataset token. |
| `scope_selector` | Yes | none | Canonical selector for the source/feed scope; broad source-category scope is not a fallback. |
| `read_target_kind` | Yes | none | Closed read target kind imported from `020`. |
| `receipt_state` | Yes | none | One `020.LakehouseReadCompletenessReceipt` state. |
| `upstream_evidence_class` | Yes | none | Supplier evidence class required by the category row. |
| `upstream_evidence_state` | Yes | none | Closed token: `sufficient`, `insufficient`, `missing`, `permission_limited`, `scope_unavailable`, `source_unavailable`, `stale`, `not_applicable`. |
| `coverage_domain` | No | null | Required when the requested effect is coverage-sensitive. |
| `completeness_decision` | Yes | none | One completeness decision from the completeness table. |
| `allowed_effects` | No | `[]` | Closed effect tokens: `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`. Missing or empty means no effects are allowed. |
| `blocking_reason` | Required when decision blocks an effect | null | Most specific blocking code from the blocking precedence table. |
| `source_authority_row_refs` | No | `[]` | Exact authority refs required for effects; broad source-category rankings are invalid. |
| `source_staleness_policy_refs` | No | `[]` | Exact staleness policy refs required for effects. |
| `validation_refs` | Yes | none | Non-empty refs proving the row's positive and negative behavior. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |

A row with `completeness_decision in {complete, empty_complete}` may enter absence evaluation only when `allowed_effects` contains the requested effect and all exact authority, coverage, and staleness refs validate.

| Completeness decision | Absence authority default | Cleanup/retraction default |
| --- | --- | --- |
| `complete` | May be considered only when `LakehouseFeedCompletenessProfileRow.allowed_effects` contains `absence`. | May be considered only when `allowed_effects` contains the requested effect. |
| `empty_complete` | May be considered only when `LakehouseFeedCompletenessProfileRow.allowed_effects` contains `absence`. | May be considered only when `allowed_effects` contains the requested effect. |
| `partial_known_gap` | Forbidden. | Forbidden. |
| `partial_unknown_gap` | Forbidden. | Forbidden. |
| `source_unavailable` | Forbidden. | Forbidden. |
| `scope_unavailable` | Forbidden. | Forbidden. |
| `permission_limited` | Forbidden. | Forbidden. |
| `not_attempted` | Forbidden. | Forbidden. |
| `not_authoritative_for_absence` | Forbidden. | Forbidden. |
| `not_applicable` | No absence claim; may emit not-applicable state when profile permits. | Forbidden. |

## Progress Signal Interpretation

Progress, liveness, lineage, freshness, acknowledgment, queue, CDC, graph-derived, source-history, destination-cleanup, and live-probe signals are non-authoritative by default. They may affect output only through an active `ProgressSignalInterpretationPolicy` row.

| Signal class | Default effect |
| --- | --- |
| Cursor exhaustion | Candidate evidence only; no absence by itself. |
| CDC offset or heartbeat | Replay/progress state only; no fact time, absence, or cleanup. |
| Queue drain or ack success | Delivery diagnostic only. |
| Provenance closure | Lineage diagnostic only. |
| dbt or freshness artifact | Freshness diagnostic only. |
| Graph apply success | Derived-view state only. |
| Destination stale delete | Non-authoritative; no source absence. |
| Live source probe result | Validation/exploration only. |

## Coverage Contract

Coverage-sensitive facts require `CoverageDimensionProfile` and a current `CoverageAssertion` before absence, pass/fail/unknown, no-change, or negative claims may be emitted.

Coverage dimensions must be explicit for vulnerability, control, endpoint, directory, DNS, DHCP/IPAM, flow, cloud inventory, source history, and future reachability domains. The coverage catalog in this document supplies default dimensions; source-specific unresolved rows remain blocking until closed in the gap artifact and validation matrix.

## Source authority artifact activation boundary

`060` algorithms are stable core contracts. Source-specific authority, staleness, coverage, completeness, progress, control-result, history, watermark, and absence rows are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `SourceAuthorityProfile` and `SourceAuthorityProfileRow` row sets | `activation_controlled_artifact` | Must be active and exact before authority or absence effects. |
| `SourceStalenessPolicy` | `activation_controlled_artifact` | Must be checksummed and scoped before stale effects. |
| `SupplierCollectionVisibilityProfile` | `activation_controlled_artifact` | Must be active before visibility-dependent absence. |
| `ControlResultMappingRow` | `activation_controlled_artifact` | Must be active before control-state output. |
| `SourceHistoryRetentionProfile` | `activation_controlled_artifact` | Must be active before source-history interpretation. |
| `AbsenceDerivationPolicy` | `activation_controlled_artifact` | Must be active before absence/no-op/stale result selection. |
| `CoverageDimensionProfile` | stable catalog plus activation-controlled source-specific rows | Source-specific rows must be active before coverage-sensitive output. |
| `CoverageAssertion` | `runtime_state_record` | Evidence record consumed only through active profiles. |
| `LakehouseFeedCompletenessProfile` and `LakehouseFeedCompletenessProfileRow` row sets | `activation_controlled_artifact` | Must evaluate feed-read and upstream evidence before absence effects and must declare allowed effects explicitly. |
| `ProgressSignalInterpretationPolicy` | `activation_controlled_artifact` | Weak signals remain non-authoritative unless an active row grants the exact effect. |
| `ProjectionWatermarkPolicy` | `activation_controlled_artifact` | Must be active before watermark advancement. |
| `WatermarkCommitRecord` | `runtime_state_record` | Records attempted watermark outcome; does not grant authority. |
| `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` | `stable_core_contract` | Algorithms validate activation refs before output. |

### SourceAuthorityArtifactLifecycleGuardRows

`060` policy row sets use `030.ActivationControlledArtifactLifecycleMachine.v1` for activation. Runtime completeness, absence, and watermark decisions remain algorithmic unless a future owner patch defines a lifecycle subject, closed states, closed events, a total transition matrix, and validation rows.

| Artifact class | Lifecycle binding | Required owner guards |
| --- | --- | --- |
| `SourceAuthorityProfile` and row set | `030.ActivationControlledArtifactLifecycleMachine.v1` | Exact fact, predicate, dataset, and scope coverage; no broad fallback; validation refs. |
| `LakehouseFeedCompletenessProfileRow` row set | Generic artifact lifecycle | Exact feed category, read target, receipt state, upstream evidence, allowed effects, and validation refs. |
| `CoverageDimensionProfile` source-specific rows | Generic artifact lifecycle | Coverage domain, dimension states, staleness rules, permission behavior, and fixture refs. |
| `ProgressSignalInterpretationPolicy` | Generic artifact lifecycle | Signal class, default non-authority, exact granted effect, and weak-signal combination fixture. |
| `ProjectionWatermarkPolicy` | Generic artifact lifecycle | Required completeness, apply status, and no advancement after failed or partial apply. |
| `AbsenceDerivationPolicy` | Generic artifact lifecycle | Absence, no-op, stale, error result mapping, and blocking precedence fixture. |

### Pure algorithm lifecycle non-substitution rule

`EvaluateLakehouseFeedCompleteness`, `DeriveAbsenceOrUnknown`, and watermark evaluation are pure deterministic algorithms for MVP. They must not be converted into lifecycle machines unless a future owner patch defines a lifecycle subject, closed states, closed events, total transition matrix, and validation rows.

## EvaluateLakehouseFeedCompleteness Algorithm

```text
EvaluateLakehouseFeedCompleteness(receipt, upstream_evidence_set, active_feed_completeness_profile_row_set, feed_category_closure_row, authority_profile_rows, coverage_assertions, staleness_policy_refs, requested_effect):
1. Validate every required activation artifact ref through `030.ActivationControlledArtifactRef`.
2. Select candidate `LakehouseFeedCompletenessProfileRow` rows by exact feed category, source dataset, scope selector, read target kind, receipt state, upstream evidence class, and activation scope.
3. If no row matches, return `not_authoritative_for_absence` with `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING`.
4. Apply `CompletenessBlockingPrecedence` to receipt state, upstream evidence state, manifest validity, source availability, permission, partial gaps, staleness, authority, coverage, and not-applicable state.
5. If a blocking condition exists, return exactly one completeness decision, selected row refs, and the most specific blocking reason.
6. If `requested_effect` is absent from `allowed_effects`, return the row decision with `COMPLETENESS_EFFECT_NOT_ALLOWED` and no effect.
7. Validate exact authority profile row refs, coverage assertion refs when required, and staleness policy refs.
8. Emit exactly one completeness decision: `complete`, `empty_complete`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `permission_limited`, `not_attempted`, `not_authoritative_for_absence`, or `not_applicable`.
9. Include selected row refs, upstream evidence refs, authority refs, coverage refs, staleness refs, blocking reason, requested effect, and validation refs in the result and `VersionManifest`.
```

### CompletenessBlockingPrecedence

| Precedence | Blocking condition | Required decision | Blocking reason |
| ---: | --- | --- | --- |
| 1 | Required activation artifact missing, inactive, checksum-mismatched, out of scope, or unvalidated. | deterministic error or `not_authoritative_for_absence` | owner-specific activation error |
| 2 | Manifest invalid or required schema unavailable. | `partial_unknown_gap` or deterministic error | `manifest_invalid` or `schema_unavailable` |
| 3 | Source or lakehouse target unavailable. | `source_unavailable` | `source_unavailable` |
| 4 | Scope unavailable. | `scope_unavailable` | `scope_unavailable` |
| 5 | Permission limited or visibility limited. | `permission_limited` | `permission_limited` |
| 6 | Partial unknown gap. | `partial_unknown_gap` | `partial_unknown_gap` |
| 7 | Partial known gap. | `partial_known_gap` | `partial_known_gap` |
| 8 | Stale source state blocks requested effect. | `not_authoritative_for_absence` | `stale` |
| 9 | Exact authority, coverage, or effect permission missing. | `not_authoritative_for_absence` | most specific missing authority, coverage, or effect code |
| 10 | Not applicable by exact profile row. | `not_applicable` | `not_applicable` |
| 11 | Empty complete with effect allowed. | `empty_complete` | none |
| 12 | Complete with effect allowed. | `complete` | none |

Only `complete` and `empty_complete` may enter absence evaluation, and only when `allowed_effects` contains the requested effect.

## DeriveAbsenceOrUnknown Algorithm

```text
DeriveAbsenceOrUnknown(candidate, evidence_set, receipt_set, upstream_evidence_set, completeness_profile, authority_profile, coverage_assertions, staleness_policy):
1. Validate every required activation artifact ref through `030.ActivationControlledArtifactRef`.
2. Fail before absence, cleanup, retraction, graph expiry, or watermark advancement when an authority, staleness, coverage, completeness, progress, control-result, or absence policy artifact is missing, inactive, checksum-mismatched, out of scope, or unvalidated.
3. Reject when no active SourceAuthorityProfileRow covers the fact type, predicate, source dataset, scopes, and requested absence authority.
4. Reject when required LakehouseReadCompletenessReceipt is missing or not tied to the active feed profile and manifest refs.
5. Evaluate UpstreamCompletenessEvidence and the read receipt through `EvaluateLakehouseFeedCompleteness` for the requested effect.
6. If the completeness decision is not `complete` or `empty_complete`, return unknown, not applicable, stale, no-op, or deterministic error with the selected blocking reason.
7. Reject when the selected completeness row's `allowed_effects` omits the requested effect.
8. Evaluate SourceStalenessPolicy using declared time-input precedence.
9. Require CoverageAssertion when the fact type or predicate is coverage-sensitive.
10. Reject weak progress signals unless one active ProgressSignalInterpretationPolicy row grants the exact effect.
11. Reject external-schema classification, status, severity, confidence, observables, enrichments, field presence, field absence, or endpoint order as authority unless an active `060` row maps the exact signal to the exact requested effect.
12. Emit exactly one AbsenceDerivationResult: absence authorized, unknown, not applicable, stale, no-op, or deterministic error.
13. Include authority profile row refs, completeness row refs, policy refs, artifact checksums, feed manifest refs, requested effect, and validation refs in the result and `VersionManifest`.
```

### AbsenceDerivationResult schema

| Field | Required | Rule |
| --- | ---: | --- |
| `result_state` | Yes | Closed token: absence authorized, unknown, not applicable, stale, no-op, or deterministic error. |
| `absence_authorized` | Yes | Boolean; true only when authority, completeness, staleness, and coverage gates pass. |
| `blocking_reason` | Required when `absence_authorized = false` | Most specific blocking code. |
| `authority_row_ref` | Required when evaluated | Exact `SourceAuthorityProfileRow` ref or null when missing. |
| `completeness_decision_ref` | Required when evaluated | Lakehouse feed completeness decision ref. |
| `coverage_assertion_refs` | Required for coverage-sensitive outputs | Non-empty when coverage-sensitive output is authorized. |
| `staleness_decision_ref` | Required when staleness evaluated | Source staleness policy decision ref. |
| `version_manifest_ref` | Required for production output | Version manifest ref containing all output-affecting inputs. |

### Authority, coverage, and absence error codes

| Error/no-op code | Emitted when |
| --- | --- |
| `SOURCE_AUTHORITY_ROW_MISSING` | No exact active authority row covers fact type, predicate, dataset, scopes, and requested absence authority. |
| `COVERAGE_ASSERTION_REQUIRED` | Coverage-sensitive output lacks a current coverage assertion. |
| `COVERAGE_DIMENSION_UNRESOLVED` | Required coverage dimension has no active source-specific row. |
| `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` | Progress/liveness/freshness/lineage signal is used without an active policy row granting the effect. |
| `COMPLETENESS_DECISION_UNSAFE_FOR_ABSENCE` | Completeness decision is partial, unavailable, permission-limited, not attempted, not authoritative, or otherwise unsafe. |
| `SOURCE_AUTHORITY_ARTIFACT_MISSING` | A required source authority, staleness, coverage, completeness, progress, control-result, or absence policy artifact ref is missing. |
| `SOURCE_AUTHORITY_ARTIFACT_INACTIVE` | A required source authority artifact is not active for production execution. |
| `SOURCE_AUTHORITY_ARTIFACT_CHECKSUM_MISMATCH` | A required source authority artifact checksum mismatches the active ref or manifest. |
| `SOURCE_AUTHORITY_ROW_SCOPE_MISMATCH` | An active source authority row set exists but no row covers the requested scope. |
| `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | No active completeness profile row covers feed category, dataset, scope selector, target kind, receipt state, upstream evidence class, and activation scope. |
| `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | Required upstream evidence is missing, stale, permission-limited, scope-limited, or insufficient for the requested effect. |
| `COMPLETENESS_EFFECT_NOT_ALLOWED` | The selected completeness row does not list the requested effect in `allowed_effects`. |
| `COMPLETENESS_BLOCKING_PRECEDENCE_APPLIED` | A higher-precedence blocking condition selected the completeness decision and blocked a lower-precedence effect. |
| `EMPTY_SCOPE_NOT_AUTHORIZED_FOR_ABSENCE` | An empty scope is complete for the feed read but not authorized for absence by exact completeness, authority, coverage, and staleness rows. |
| `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | Watermark advancement is requested while completeness, upstream evidence, authority, coverage, or staleness gates block the effect. |
| `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | A gold derivation, absence derivation, control-state output, cleanup, retraction, graph expiry, or watermark decision attempts to use external schema classification, status, severity, confidence, field presence, field absence, observable, enrichment, or endpoint order as authority without an active `060` row. |

## Watermarks

Every attempted production source, projection, graph-apply, or presence-only watermark change must be evaluated through `ProjectionWatermarkPolicy` and persisted as exactly one `WatermarkCommitRecord`. Failed, partial, aborted, schema-preflight-failed, or completeness-blocked runs must not advance watermarks.

## Authority-to-GoldFact linkage

`060` owns absence and coverage authorization causes but does not restate `040.GoldFactSchema`. The following cause codes may be recorded on a 040-shaped `GoldFact` validation or derivation failure.

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `GOLD_FACT_COVERAGE_REQUIRED` | `060` | A coverage-sensitive fact or predicate attempts output with `coverage_assertion_refs = []`. |
| `GOLD_FACT_AUTHORITY_ROW_MISSING` | `060` | No exact active `SourceAuthorityProfileRow` covers the requested fact, predicate, dataset, scopes, staleness, coverage, and absence authority. |

Missing rows, stale states, partial states, permission-limited states, weak progress signals, destination cleanup, omitted fields, missing lakehouse rows, and OCSF field absence must remain unable to create absence by themselves.


### External schema non-authority boundary

OCSF class, category, activity, type, object paths, observables, enrichments, status, severity, confidence, endpoint order, field presence, and field absence are normalized observation metadata only. They must not satisfy `SourceAuthorityProfileRow`, `CoverageAssertion`, `SourceStalenessPolicy`, `LakehouseFeedCompletenessProfileRow`, `AbsenceDerivationPolicy`, or `ControlResultMappingRow` requirements unless an active `060`-owned row explicitly maps the exact signal to the exact effect.

A `060` row that maps an external-schema signal must name the external schema profile ref, field path, source dataset, fact type, predicate, requested effect, source authority row ref, coverage row ref when applicable, staleness row ref, validation refs, activation scope, and lifecycle status. Omission of any required ref means the external-schema signal remains non-authoritative.

Vulnerability Finding status must not become Cadastre `assertion_state` without a `060` or `080` derivation path. DNS and DHCP field absence must not become absence without exact completeness, coverage, staleness, and authority rows. OCSF endpoint order must not become graph direction authority.

## Source Authority Contract Details

### CoverageDimensionProfile catalog

| Row ID | Domain | Dimension | Required evidence | Freshness policy | Permission-limited behavior | Partial behavior | Stale behavior | Default output | CoverageClosureStatus |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cov-vulnerability-default` | vulnerability | scanner target, credentialed status, plugin/check scope, scan window | coverage assertion plus scanner evidence | `SourceStalenessPolicy` | `permission_limited` blocks absence | `partial_known_gap` or `partial_unknown_gap` | stale blocks absence | unknown | `requires_source_specific_row` |
| `cov-control-default` | control | benchmark/check scope, applicability, evaluation status | `ControlResultMappingRow` and coverage assertion | owner row | not checked/unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-endpoint-default` | endpoint | asset scope, enrollment visibility, collection window | feed completeness plus authority row | owner row | blocks absence | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-directory-default` | directory | tenant/domain, group/member scope, hidden membership visibility | directory completeness evidence | owner row | blocks nonmembership | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dns-default` | DNS | zone/source scope, TTL, authoritative source | DNS feed evidence | TTL-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-dhcp-ipam-default` | DHCP/IPAM | scope, lease window, authoritative system | lease/IPAM evidence | lease-aware policy | unknown | unknown | stale | unknown | `requires_source_specific_row` |
| `cov-flow-default` | flow | sensor scope, collection point, time window, role evidence | flow feed evidence | window policy | unknown | partial | stale | unknown | `requires_source_specific_row` |
| `cov-cloud-inventory-default` | cloud inventory | account/project/subscription, region, resource type | inventory feed and source history | source-specific | permission_limited | partial | stale | unknown | `requires_source_specific_row` |
| `cov-source-history-default` | source history | source-native history window, retention | history retention profile | history-window policy | unknown | unknown | outside window no proof | unknown | `requires_source_specific_row` |
| `cov-reachability-deferred` | deferred reachability | topology, route, policy, NAT, workload, identity context | inactive `200` placeholders | inactive | no MVP claim | no MVP claim | no MVP claim | no-op | `inactive_deferred` |

`CoverageClosureStatus` is a closed enum: `closed_default`, `requires_source_specific_row`, and `inactive_deferred`. Source-specific absence-sensitive outputs remain blocked until a `requires_source_specific_row` domain has a matching active source-specific coverage row.

### Coverage-sensitive blocked outputs

| Missing coverage domain | Blocked output |
| --- | --- |
| vulnerability | vulnerability absence and fixed-state absence. |
| control | control pass, fail, unknown, not-checked, and not-applicable facts. |
| endpoint | endpoint disappearance, non-observation, and cleanup. |
| directory | group non-membership, user non-membership, hidden-membership absence. |
| DNS | DNS absence, host deletion from TTL expiry, negative DNS fact. |
| DHCP/IPAM | DHCP lease absence, IPAM absence, host deletion from lease expiry. |
| flow | flow non-observation and observed-connection absence. |
| cloud inventory | missing cloud resource and resource deletion inference. |
| source history | no-change proof outside supported history window. |

### FeedCategoryCoverageEffectMatrix

This table aligns to `020.LakehouseFeedCategoryClosureRequirementTable` without restating feed-read target mechanics. Exact activation-controlled rows must provide the source-specific refs before effects are allowed.

| Feed category | Required completeness, coverage, and authority dimensions before effects | Effects that may be allowed by exact row | Missing or insufficient evidence result |
| --- | --- | --- | --- |
| `endpoint_inventory` | endpoint scope, enrollment or inventory visibility, collection window, failed-scope evidence, source authority, staleness, coverage assertion | `absence`, `cleanup`, `graph_expiry`, `watermark` | `not_authoritative_for_absence` or specific partial/permission state |
| `configuration_inventory` | configuration scope, object type, source authority, staleness, coverage assertion for affected domain | `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark` | `not_authoritative_for_absence` |
| `vulnerability_scan` | scanner target, credential/auth status, plugin/check set, scan window, failed-target/exclusion evidence, coverage assertion, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `COVERAGE_ASSERTION_REQUIRED` |
| `control_evaluation` | benchmark/check scope, applicability, control result mapping, evaluation status, coverage assertion, staleness, authority rows | `absence`, `retraction`, `watermark` | `not_checked`, `unknown`, or exact blocking reason; no pass/fail by default |
| `directory_inventory` | tenant/domain, object type, hidden-object permission, page/delta completion, coverage, staleness, authority rows | `absence`, `cleanup`, `graph_expiry`, `watermark` | `permission_limited`, `partial_known_gap`, or `not_authoritative_for_absence` |
| `directory_membership` | tenant/domain, group, member type, direct/transitive mode, hidden-membership permission, page/delta completion, AD primary-group handling, coverage, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, or `not_authoritative_for_absence` |
| `dns_record_set` | zone/source scope, authoritative source, TTL basis, collection window, coverage, staleness, authority rows | `absence`, `retraction`, `watermark` | TTL expiry alone maps to `not_authoritative_for_absence` |
| `dhcp_ipam_assignment` | DHCP/IPAM scope, lease window, authoritative-system evidence, coverage, staleness, authority rows | `absence`, `cleanup`, `retraction`, `watermark` | lease expiry alone maps to `not_authoritative_for_absence` |
| `network_flow` | sensor scope, collection point, time window, role evidence, coverage, staleness, authority rows | `watermark` only when exact row permits; absence effects default blocked | missing flow maps to `unknown` |
| `cloud_asset_inventory` | account/project/subscription, region, resource type, permission visibility, source-history requirement when applicable, coverage, staleness, authority rows | `absence`, `cleanup`, `graph_expiry`, `watermark` | `permission_limited`, `stale`, or `not_authoritative_for_absence` |
| `source_history` | source-native history window, retention profile, query scope, authority rows, staleness rows | `absence`, `watermark` only within supported window and exact row permission | outside-window no-result maps to `not_authoritative_for_absence` |
| `future_reachability` | inactive deferred reachability contracts only | none in MVP | deterministic no-op or deferred reachability error |

### LakehouseFeedCompletenessProfile table

| Feed-read state | Upstream evidence state | Completeness decision | Source absence effect |
| --- | --- | --- | --- |
| `read_complete` | sufficient and current | `complete` or `empty_complete` as profile defines | May be considered only with authority and coverage. |
| `read_complete` | missing or insufficient | `not_authoritative_for_absence` | Forbidden. |
| `read_partial_known_gap` | any | `partial_known_gap` | Forbidden. |
| `read_partial_unknown_gap` | any | `partial_unknown_gap` | Forbidden. |
| `read_unavailable` | any | `source_unavailable` | Forbidden. |
| `schema_unavailable` | any | `source_unavailable` | Forbidden. |
| `manifest_invalid` | any | `partial_unknown_gap` or deterministic error | Forbidden. |

### ProgressSignalInterpretationPolicy total matrix

| Signal | Default interpretation | May authorize absence by default |
| --- | --- | ---: |
| cursor exhaustion | progress candidate only | No |
| delta token | resume state only | No |
| CDC offset | replay state only | No |
| CDC heartbeat | liveness diagnostic only | No |
| schema-history availability | replay sufficiency input only | No |
| queue drain | delivery diagnostic only | No |
| ack success | delivery diagnostic only | No |
| provenance closure | lineage diagnostic only | No |
| freshness artifact | freshness diagnostic only | No |
| graph apply | derived-view state only | No |
| destination cleanup | non-authoritative cleanup signal | No |
| live source probe | validation/exploration only | No |
| table commit success | lakehouse write evidence only | No |
| graph rebuild success | derived-view rebuild evidence only | No |

Weak progress signals must not combine into stronger authority.

### SourceStalenessPolicy

| Policy field | Required behavior |
| --- | --- |
| Time-input precedence | Declared per source category, dataset, fact type, predicate, and scope. |
| Missing-time behavior | Emit stale/unknown/error as policy states; no implicit current time. |
| Expiry behavior | May mark facts stale; must not authorize absence unless authority and completeness permit. |
| Output effects | API labels import `110`; graph lag remains separate from source staleness. |

### ControlResultMappingRow

| External state | Cadastre state | Default compliance export |
| --- | --- | --- |
| pass | pass fact only when authority and coverage permit | pass |
| fail | fail fact only when authority and coverage permit | fail |
| unknown | unknown | unknown |
| error | error | error |
| not evaluated | not_checked | not checked |
| not checked | not_checked | not checked |
| not applicable | not_applicable | not applicable |

### SourceHistoryRetentionProfile

| History condition | Required behavior |
| --- | --- |
| Inside supported source-native history window | May be evidence only when source authority row permits. |
| Outside supported source-native history window | Must not become no-change proof. |
| Source history unavailable | Emit unknown or error; no absence. |
| Cadastre replay retention available | Separate from source-native history. |

### Completeness-to-effect matrix

| Completeness decision | Absence authority | Cleanup authority | Retraction authority | Graph expiry authority | Watermark authority | Default API state | Owner |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `complete` | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | observed or authorized_not_observed | `060` |
| `empty_complete` | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | allowed_effects-gated | authorized_not_observed | `060` |
| `partial_known_gap` | forbidden | forbidden | forbidden | forbidden | forbidden | partial_known_gap | `060` |
| `partial_unknown_gap` | forbidden | forbidden | forbidden | forbidden | forbidden | partial_unknown_gap | `060` |
| `source_unavailable` | forbidden | forbidden | forbidden | forbidden | forbidden | source_unavailable | `060` |
| `scope_unavailable` | forbidden | forbidden | forbidden | forbidden | forbidden | scope_unavailable | `060` |
| `permission_limited` | forbidden | forbidden | forbidden | forbidden | forbidden | permission_limited | `060` |
| `not_attempted` | forbidden | forbidden | forbidden | forbidden | forbidden | not_checked | `060` |
| `not_authoritative_for_absence` | forbidden | forbidden | forbidden | forbidden | forbidden | unknown | `060` |
| `not_applicable` | no absence claim | forbidden | forbidden | forbidden | forbidden | not_applicable | `060` |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `060-CLEANUP-AC-001` | No banned reference class remains. |
| `060-CLEANUP-AC-002` | Broad source-category ranking still cannot act as fallback authority when the exact `SourceAuthorityProfileRow` is missing. |
| `060-CLEANUP-AC-003` | Progress, liveness, lineage, freshness, acknowledgment, CDC, graph-derived, and live-probe signals remain non-authoritative by default. |
| `060-CLEANUP-AC-004` | Missing rows, stale states, partial states, and permission-limited states still cannot authorize absence unless an active policy permits it. |
| `060-SCHEMA-PATCH-AC-001` | Coverage-sensitive `GoldFact` outputs fail when `coverage_assertion_refs = []`. |
| `060-SCHEMA-PATCH-AC-002` | `absence_outcome` can be materialized only by `DeriveAbsenceOrUnknown`. |
| `060-SCHEMA-PATCH-AC-003` | `060` does not restate `GoldFact` field definitions; it references `040.GoldFactSchema` by exact name. |

| `060-COVERAGE-CLOSURE-AC-001` | Every coverage catalog row has a `CoverageDimensionProfileRowId` and `CoverageClosureStatus`. |
| `060-ABSENCE-OUTPUT-AC-001` | `DeriveAbsenceOrUnknown` emits an `AbsenceDerivationResult` with authority, completeness, coverage, staleness, and manifest refs or a specific blocking reason. |
| `060-VOLATILITY-AC-001` | Missing exact source-authority row, inactive row set, staleness checksum mismatch, coverage row absence, and weak-signal combination cases fail or no-op with no absence, cleanup, retraction, graph expiry, or watermark advancement. |
| `060-VOLATILITY-AC-002` | Every output-affecting source-authority artifact ref appears in `VersionManifest` with checksum and validation refs. |
| `060-FEED-CLOSURE-AC-001` | Missing `LakehouseFeedCompletenessProfileRow` maps to `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` and no absence-sensitive effect. |
| `060-FEED-CLOSURE-AC-002` | Missing upstream completeness evidence maps to `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `not_authoritative_for_absence`, not absence. |
| `060-FEED-CLOSURE-AC-003` | `allowed_effects = []` or omitted allowed effects blocks absence, cleanup, retraction, graph expiry, and watermark. |
| `060-FEED-CLOSURE-AC-004` | Blocking precedence is deterministic and selects the same blocking reason for identical evidence, rows, refs, and requested effect. |
| `060-FEED-CLOSURE-AC-005` | Complete or empty-complete decisions authorize effects only through exact active authority, completeness, coverage, and staleness rows. |
| `060-OCSF-NONAUTH-AC-001` | OCSF status, severity, confidence, observables, enrichments, class, activity, field presence, and field absence cannot satisfy authority, completeness, coverage, staleness, control-result, or absence gates by themselves. |
| `060-OCSF-NONAUTH-AC-002` | Vulnerability Finding status does not become Cadastre `assertion_state` without a `060` or `080` owned derivation path. |
| `060-OCSF-NONAUTH-AC-003` | DNS or DHCP field absence does not become absence without exact completeness, coverage, staleness, and authority rows. |
| `060-LIFECYCLE-AC-001` | Source authority, completeness, coverage, progress, absence, and watermark artifacts can become active only through `030.LifecycleTransitionEvidence`, while `EvaluateLakehouseFeedCompleteness` and `DeriveAbsenceOrUnknown` remain pure deterministic algorithms. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `060-AC-001` | Missing, stale, partial, permission-limited, unsupported, or unavailable evidence cannot become absence unless an active profile authorizes it. |
| `060-AC-002` | Weak progress signals cannot be combined into stronger authority than each active policy row permits. |
| `060-AC-003` | Every coverage-dependent fact has a current coverage assertion or emits a deterministic unknown/error outcome. |
| `060-AC-004` | Source staleness and derived-view lag are exposed as distinct states. |
| `060-AC-005` | Every watermark attempt emits exactly one `WatermarkCommitRecord`. |
| `060-AC-006` | Gold derivation, cleanup, retraction, graph expiry, and watermark advancement persist completeness row refs and blocking reasons before producing absence-sensitive effects. |
| `060-AC-007` | External schema metadata cannot create authority, absence, assertion state, compliance state, risk state, cleanup, graph expiry, retraction, or watermark effects without exact `060` rows. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
