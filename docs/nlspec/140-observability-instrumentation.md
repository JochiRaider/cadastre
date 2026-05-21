---
doc_id: CADASTRE-NLSPEC-140
title: Observability, Instrumentation, and Runtime Telemetry
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

`140` owns runtime observability instrumentation only. It does not own source authority, identity, graph projection, package activation, API rendering, audit persistence, validation harness behavior, or replay checksum algorithms outside the telemetry handoffs named in this spec.

## Purpose

Define runtime telemetry signals, trace context propagation, telemetry correlation, metric instruments, telemetry attributes, telemetry redaction, exporter behavior, telemetry runtime state, telemetry health mapping, telemetry replay exclusion, and the telemetry non-authority rule.

## Explicit Non-Scope

- Domain record authority.
- Source completeness, source absence, coverage, watermarks, or source authority.
- Identity evidence, canonical identity, merge, split, or resolver scoring.
- Graph projection, graph apply, graph query, graph drift repair, or backend mutation authority.
- Package trust, compatibility, activation, rollback, or quarantine behavior.
- Caller-visible API schemas, health response rendering, audit event persistence, authorization, or final redaction policy.
- Validation fixture schemas and acceptance report mechanics.

## Imports

- `010.AuthorityClass`
- `010.RuntimeTelemetryAuthorityRule`
- `030.ProcessingStageDAG`
- `030.StageStateRecord`
- `030.VersionManifest`
- `030.ActivationControlledArtifactRef`
- `060.ProgressSignalInterpretationPolicy`
- `070.ResolverProfile`
- `080.ReplayEquivalencePolicy`
- `090.GraphBackendProfile`
- `090.GraphApplyResult`
- `090.DerivedViewState`
- `100.ProductionPackageSetManifest`
- `110.OperationalHealthStatus`
- `110.ErrorRecord`
- `110.AuditEvent`
- `110.RedactionPolicy`
- `110.AuthorizationDecision`
- `120.ValidationMatrix`

## Exports

- `ObservabilityInstrumentationProfile`
- `TelemetrySignalPolicy`
- `TraceContextPolicy`
- `TelemetryCorrelationPolicy`
- `MetricInstrumentCatalog`
- `MetricInstrumentRow`
- `TelemetryAttributePolicy`
- `TelemetryRedactionPolicy`
- `TelemetryExporterProfile`
- `TelemetryRuntimeState`
- `TelemetryHealthMappingPolicy`
- `TelemetryReplayExclusionPolicy`
- `TelemetryNonAuthorityRule`
- `EmitTelemetry`
- `ValidateTelemetryProfile`

## Telemetry Non-Authority

`TelemetryNonAuthorityRule` imports the global boundary from `010.RuntimeTelemetryAuthorityRule` and applies it to every telemetry signal owned by this spec.

Runtime telemetry, including traces, spans, metrics, structured logs, baggage, exporter state, Collector state, sampling decisions, dashboards, alerts, dropped-signal counts, and telemetry backend records, is operational diagnostic material only.

Telemetry must not create, modify, authorize, validate, retract, expire, merge, split, project, replay, or persist authoritative Cadastre domain records.

Telemetry must not create or modify `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `IdentityDecision`, `GoldFact`, `GraphDeltaSet`, `GraphApplyResult`, `SourceAuthorityProfileRow`, `LakehouseFeedCompletenessProfileRow`, `CoverageAssertion`, `PackageActivationFailureEvent`, `ProductionPackageSetManifest`, `WatermarkCommitRecord`, `VersionManifest`, or `AuditEvent`.

Telemetry may affect caller-visible health only through `TelemetryHealthMappingPolicy` and the `110.OperationalHealthStatus` handoff. Telemetry loss, exporter failure, Collector failure, or sampling must not invalidate already persisted authoritative Cadastre records.

Run-lock telemetry is diagnostic material only. Metrics, traces, structured logs, dashboards, or alerts about lock acquisition, heartbeat, stale recovery, fencing, and lock loss must not become lock authority, source authority, source completeness, graph authority, package activation authority, audit evidence, replay evidence, or watermark authority. Persisted `030` lock evidence is the only run-lock evidence that may gate domain output.

## Default Instrumentation Profile

`ObservabilityInstrumentationProfile` is an activation-controlled artifact. When telemetry is enabled and no more specific active profile covers the execution scope, the implementation must materialize the following candidate defaults before validation.

| Field | Default value | Rule |
| --- | --- | --- |
| `telemetry_standard` | `opentelemetry` | OpenTelemetry compatibility is the default instrumentation standard. OTel does not define Cadastre domain authority. |
| `telemetry_protocol` | `otlp` | OTLP-compatible export is the default protocol when export is enabled. |
| `telemetry_signals` | `traces`, `metrics`, `structured_logs` | Baggage is propagation context only unless the signal policy explicitly permits it. |
| `telemetry_delivery_mode` | `bounded_best_effort_nonblocking` | Emission must not block domain writes beyond bounded in-process validation. |
| `telemetry_export_failure_domain_effect` | `none` | Export failure must not alter authoritative domain outputs. |
| `telemetry_export_failure_health_effect` | `degraded` | Default health effect is observability degradation. |
| `telemetry_trace_id_replay_checksum_effect` | `excluded` | Trace IDs are excluded from domain replay checksums by default. |
| `telemetry_span_id_replay_checksum_effect` | `excluded` | Span IDs are excluded from domain replay checksums by default. |
| `raw_payload_in_telemetry` | `forbidden` | Raw payload bytes fail before export. |
| `private_source_binding_in_telemetry` | `forbidden` | Private binding values fail before export. |
| `backend_internal_id_in_telemetry` | `forbidden` | Backend-generated IDs fail before export. |

A profile that requires production export must include an active `TelemetryExporterProfile`. If the exporter profile is omitted, telemetry export is disabled; local signal construction may still run for validation-only diagnostics and must not emit outside the process.

## Signal Policy

`TelemetrySignalPolicy` is a closed activation-controlled policy. A signal not listed below must fail with `TELEMETRY_SIGNAL_FORBIDDEN` unless the policy row declares `validation_only_non_production = true` and the execution mode is validation-only.

| Signal token | Production status | Default export behavior | Non-authority rule |
| --- | --- | --- | --- |
| `trace` | Allowed when profile enables traces. | May export redacted spans through `EmitTelemetry`. | Must not prove stage success, fact correctness, graph correctness, audit persistence, package activation, or replay equivalence. |
| `metric` | Allowed when profile enables metrics. | May export instruments declared in `MetricInstrumentCatalog`. | Metric values, including zero values, must not prove absence, completeness, cleanup permission, or control pass/fail. |
| `structured_log` | Allowed when profile enables structured logs. | May export allowlisted attributes after redaction. | Log presence or absence must not prove domain success, failure, completeness, or audit persistence. |
| `baggage` | Propagation-only by default. | Export forbidden unless a policy row allows a specific allowlisted key. | Baggage must not contain identity evidence, source bindings, raw values, credentials, or authority state. |

Signals must be normalized to these tokens before validation. Unknown case variants, provider-specific signal names, and custom signal names must not be accepted by string similarity.

## Trace Context Policy

`TraceContextPolicy` governs propagation across Cadastre runtime boundaries.

| Context condition | Required behavior |
| --- | --- |
| Inbound trace context present and valid | Preserve parent context for diagnostic correlation only. |
| Inbound trace context absent | Create a new root context only when traces or structured logs are enabled; otherwise no-op. |
| Parent-child stage relationship | A child span may be created only after the stage execution context exists and before telemetry emission. It must not affect stage ordering. |
| Async stage handoff | Persist only the allowed opaque correlation context required by the stage runner; raw trace IDs and span IDs must not become stage state or replay state. |
| Sampled trace | May emit spans according to exporter profile and redaction policy. |
| Unsampled trace | Must preserve propagation context but may emit no span. Unsampled state must not change domain output. |
| Context loss | Must emit a telemetry diagnostic when telemetry is enabled; must not fail the domain operation. |
| Baggage key not allowlisted | Reject or drop the key before propagation according to `TelemetryAttributePolicy`; do not export it. |

Trace context values are runtime volatile fields. They are excluded from domain replay checksums by default.

## Telemetry Correlation Policy

`TelemetryCorrelationPolicy` controls operator-facing diagnostic correlation.

`diagnostic_correlation_ref` is an opaque Cadastre support reference. It is not a trace ID, span ID, evidence ref, audit ID, identity ID, graph ID, package ID, or replay key.

| Output condition | Default behavior |
| --- | --- |
| API or audit output does not request a diagnostic ref | Omit or null the ref. |
| Policy lacks an approved derivation algorithm | Do not expose a diagnostic correlation ref. |
| Raw trace ID or raw span ID requested for caller-visible output | Reject with `TELEMETRY_ATTRIBUTE_FORBIDDEN` unless both `140.TelemetryCorrelationPolicy` and `110.RedactionPolicy` permit the exact data class. |
| Diagnostic ref emitted in API envelope | `110.AuditEvent` must include the same opaque ref when `110` requires it. |
| Diagnostic ref present during replay | Exclude from domain replay equivalence by default. |

TODO: Product security governance must choose the exact opaque `diagnostic_correlation_ref` derivation algorithm before production exposure. Until that TODO is resolved in an active `TelemetryCorrelationPolicy`, raw trace IDs and span IDs remain forbidden and caller-visible diagnostic refs default to null.

## Metric Instrument Catalog

`MetricInstrumentCatalog` is a closed activation-controlled registry for production metrics. An emitted metric name must match exactly one active catalog row. Unknown metrics fail with `TELEMETRY_SIGNAL_FORBIDDEN` before export.

Default metric rows:

| Metric name | Type | Unit | Allowed attributes | Cardinality class | Aggregation | Validation refs |
| --- | --- | --- | --- | --- | --- | --- |
| `cadastre.stage.started` | counter | `{stage}` | `stage_id`, `stage_class`, `run_id`, `version_manifest_ref`, `package_set_ref` | `bounded_medium` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*` |
| `cadastre.stage.completed` | counter | `{stage}` | `stage_id`, `stage_class`, `run_id`, `version_manifest_ref`, `package_set_ref`, `owner_error_code` | `bounded_medium` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*` |
| `cadastre.stage.duration` | histogram | `ms` | `stage_class`, `owner_error_code` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*` |
| `cadastre.api.request.duration` | histogram | `ms` | `endpoint_class`, `health_scope`, `owner_error_code` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*` |
| `cadastre.graph.apply.duration` | histogram | `ms` | `graph_operation`, `owner_error_code` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*` |
| `cadastre.telemetry.exporter.failures` | counter | `{failure}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.exporter.queue_depth` | gauge | `{item}` | `telemetry_profile_ref`, `exporter_profile_ref` | `bounded_low` | last value | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.dropped_spans` | counter | `{span}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.dropped_metrics` | counter | `{metric}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.run_lock.acquired` | counter | `{lock}` | `lock_scope_class`, `output_class`, `owner_error_code` | `bounded_low` | monotonic sum | `120-RUNLOCK-OBSERVABILITY-*` |
| `cadastre.run_lock.heartbeat` | counter | `{heartbeat}` | `lock_scope_class`, `result`, `owner_error_code` | `bounded_low` | monotonic sum | `120-RUNLOCK-OBSERVABILITY-*` |
| `cadastre.run_lock.recovery_attempt` | counter | `{attempt}` | `lock_scope_class`, `result`, `owner_error_code` | `bounded_low` | monotonic sum | `120-RUNLOCK-OBSERVABILITY-*` |
| `cadastre.run_lock.lost` | counter | `{lock}` | `lock_scope_class`, `output_class`, `owner_error_code` | `bounded_low` | monotonic sum | `120-RUNLOCK-OBSERVABILITY-*` |
| `cadastre.run_lock.lease_remaining` | gauge | `s` | `lock_scope_class`, `output_class` | `bounded_low` | last value | `120-RUNLOCK-OBSERVABILITY-*` |

The active metric catalog may add rows only through activation-controlled artifacts. A package-supplied catalog must pass `ValidateTelemetryProfile` and package activation before production use.

Run-lock metric rows must not allow attributes containing raw lock keys, source-native identifiers, canonical IDs, private source bindings, raw fencing token values, raw idempotency key values, or backend IDs. Only redacted refs or checksums may be emitted when `TelemetryAttributePolicy`, `TelemetryRedactionPolicy`, and `110.RedactionPolicy` permit them.

## Metric Instrument Row Schema

`MetricInstrumentRow` is the stable row interface inside `MetricInstrumentCatalog`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `metric_name` | Yes | none | Stable lowercase dotted name. Unknown names fail before export. |
| `instrument_type` | Yes | none | Closed token: `counter`, `gauge`, `histogram`. |
| `unit` | Yes | none | UCUM-compatible unit string or bounded literal such as `{item}`. |
| `description` | Yes | none | Maximum 512 Unicode scalar values after normalization. |
| `allowed_attribute_keys` | Yes | `[]` only when no attributes allowed | Every key must resolve through `TelemetryAttributePolicy`. |
| `cardinality_class` | Yes | none | Closed token: `static`, `bounded_low`, `bounded_medium`, `forbidden_unbounded`. |
| `max_distinct_attribute_sets_per_run` | Required for bounded classes | `1` for `static` | `bounded_low` maximum is 64; `bounded_medium` maximum is 1024; `forbidden_unbounded` cannot be active. |
| `aggregation` | Yes | none | Closed token: `monotonic_sum`, `last_value`, `explicit_bucket_histogram`. |
| `export_temporality` | No | `cumulative` | Closed token: `cumulative` or `delta`. |
| `validation_refs` | Yes | none | Non-empty `120-OBSERVABILITY-*` rows. |
| `activation_scope` | Yes | none | Scope in which the row may affect telemetry output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

## Telemetry Attribute Policy

`TelemetryAttributePolicy` is an activation-controlled allowlist for telemetry attributes. Attribute keys and values must be normalized before validation.

Default allowed attribute keys:

| Attribute key | Allowed value class | Maximum distinct values per run | Rule |
| --- | --- | ---: | --- |
| `stage_id` | Cadastre stage ID | 1024 | Must not encode private route or source-native ID. |
| `stage_class` | Closed `030` stage class | 64 | Must match `030` stage class vocabulary. |
| `run_id` | Opaque run ref | 1024 | Runtime ref only; excluded from domain replay when volatile. |
| `version_manifest_ref` | `030.VersionManifest` ref | 1024 | Ref only; no manifest bytes. |
| `package_set_ref` | `100.ProductionPackageSetManifest` ref | 1024 | Ref only; no package payload bytes. |
| `activation_artifact_ref` | `030.ActivationControlledArtifactRef` ref | 4096 | Ref and checksum only. |
| `owner_error_code` | Owner error code | 256 | Error code only; no raw owner context. |
| `lifecycle_transition_evidence_ref` | `030.LifecycleTransitionEvidence` ref | 4096 | Ref and checksum only. |
| `validation_row_ref` | `120.ValidationMatrix` row ref | 4096 | Row ref only. |
| `health_scope` | `110.OperationalHealthRequest.health_scope` token | 32 | Includes `observability`. |
| `telemetry_profile_ref` | `ObservabilityInstrumentationProfile` ref | 64 | Ref and checksum only. |
| `exporter_profile_ref` | `TelemetryExporterProfile` ref | 64 | Ref and checksum only. |
| `graph_operation` | Closed graph operation class | 64 | Diagnostic only; must not include provider query text. |
| `endpoint_class` | Public endpoint class | 64 | Diagnostic only; must not include route names or private paths. |

Forbidden values include raw payload bytes, raw payload hashes when not authorized by `110`, private source bindings, private routes, credentials, secrets, tokens, tenant inventories, environment-specific source target lists, backend internal IDs, source-native IDs, canonical entity IDs, raw hostnames, IP addresses, user names, provider-native query text, raw graph property values, package payload bytes, and unredacted source identifiers.

A forbidden key or value must fail before export with the most specific telemetry error code.

## Telemetry Redaction Policy

`TelemetryRedactionPolicy` imports `110.RedactionPolicy` for caller-visible and audit-visible redaction classes. This spec owns telemetry pre-export rejection and redaction.

| Data class in telemetry | Default behavior | Error code when not redacted or rejected |
| --- | --- | --- |
| Raw payload bytes | Reject before export. | `TELEMETRY_RAW_PAYLOAD_FORBIDDEN` |
| Private source binding, private route, credential, tenant inventory | Reject before export. | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` |
| Backend-generated graph node, edge, vertex, document, element, or relationship ID | Reject before export. | `TELEMETRY_BACKEND_ID_FORBIDDEN` |
| Source-native ID | Reject unless a future owner policy allows a bounded hashed diagnostic form. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Canonical Cadastre entity or identity ID | Reject unless a future owner policy allows a bounded hashed diagnostic form. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Hostname, IP address, user name, account name | Reject unless a future owner policy allows a bounded redacted form. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Provider-native query text | Reject by default. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Exporter endpoint or Collector route | Redact to opaque ref before telemetry or API output. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |

Redaction that changes a metric attribute value must re-run metric cardinality validation on the redacted value set before export.

## Telemetry Exporter Profile

`TelemetryExporterProfile` is an activation-controlled artifact. A production exporter profile must name endpoint refs, delivery bounds, failure behavior, health mapping refs, redaction refs, and validation refs. Omitted required fields fail profile validation.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `exporter_profile_id` | Yes | none | Stable ID scoped to `140`. |
| `protocol` | Yes | none | `otlp` for the default profile. |
| `endpoint_ref` | Required when export enabled | Export disabled when omitted and export not required | Opaque ref only; raw URL, credential, or private route is forbidden in public artifacts. |
| `collector_required` | No | `true` for production export | Direct exporter endpoints are validation-only unless policy permits them. |
| `timeout_ms` | Yes when export enabled | none | Range `100..30000`. |
| `retry_policy` | Yes when export enabled | none | Must define max attempts, backoff bounds, and retryable exporter errors. |
| `queue_max_items` | Yes when export enabled | none | Range `1..65536`; queue overflow uses `TELEMETRY_EXPORTER_QUEUE_OVERFLOW`. |
| `backpressure_behavior` | Yes | `drop_newest_nonblocking` | Closed token: `drop_newest_nonblocking`, `drop_oldest_nonblocking`. Blocking domain writes is forbidden. |
| `shutdown_flush_timeout_ms` | Yes when export enabled | none | Range `0..30000`; failure does not mutate domain output. |
| `failure_health_effect` | No | `degraded` | Closed token: `none`, `degraded`, `blocked` for observability health only. |
| `validation_refs` | Yes | none | Non-empty `120-OBSERVABILITY-EXPORTER-*` rows. |
| `activation_scope` | Yes | none | Scope in which export may run. |
| `lifecycle_status` | Yes | none | Production export requires `active`. |

TODO: Product operations governance must select production default exporter numeric values before production export is active. Until that TODO is resolved by an active exporter profile, production export remains blocked when export is required.

## Telemetry Runtime State

`TelemetryRuntimeState` is a runtime state record. It records telemetry delivery and loss only. It does not grant domain authority.

| Field | Required | Rule |
| --- | ---: | --- |
| `telemetry_runtime_state_id` | Yes | Deterministic ID over profile ref, exporter profile ref, run ref, state window, and canonical counters; excludes wall-clock display fields. |
| `profile_ref` | Yes | Active `ObservabilityInstrumentationProfile` ref. |
| `exporter_profile_ref` | Required when export enabled | Null only when export disabled by active profile. |
| `dropped_span_count` | Yes | Unsigned integer. |
| `dropped_metric_count` | Yes | Unsigned integer. |
| `dropped_structured_log_count` | Yes | Unsigned integer. |
| `exporter_queue_depth` | Required when export enabled | Unsigned integer; omit only when export disabled. |
| `exporter_failure_state` | Yes | Closed token: `none`, `unavailable`, `queue_overflow`, `timeout`, `redaction_failure`, `profile_invalid`. |
| `collector_reachability_state` | Yes | Closed token: `not_required`, `reachable`, `unreachable`, `unknown`. |
| `last_successful_export_at` | No | RFC3339 UTC or null; excluded from domain replay unless health-visible output includes it through policy. |
| `health_affecting_refs` | Yes | Refs to health mapping, exporter profile, redaction policy, and owner error rows that affected health. |
| `version_manifest_ref` | Required when state affects health, API, audit, validation, or operator-visible diagnostics | Must be included by `030.VersionManifest`. |

## Telemetry Health Mapping Policy

`TelemetryHealthMappingPolicy` maps telemetry runtime conditions to `110.OperationalHealthStatus` inputs. It must not mutate domain outputs.

| Telemetry condition | Default `health_scope = observability` effect | Forbidden domain effect |
| --- | --- | --- |
| Exporter unavailable | `degraded` | No source, identity, fact, graph, package, audit, replay, or watermark mutation. |
| Exporter queue overflow | `degraded` | No domain mutation. |
| Telemetry profile missing when telemetry required | `blocked` | No domain mutation. |
| Collector unreachable | `degraded` | No domain mutation. |
| Telemetry disabled by active profile | `healthy` for observability-disabled mode | No domain mutation. |
| Dropped telemetry counts exceed active policy bound | `degraded` | No domain mutation. |
| Run-lock telemetry missing | `degraded` for observability only | No lock, source, graph, package, audit, replay, or watermark mutation. |
| Run-lock metric cardinality overrun | `degraded` or `blocked` for observability only by policy | No domain mutation. |

System health aggregation remains owned by `110.EndpointOutcomeMatrix`.

## Telemetry Replay Exclusion Policy

`TelemetryReplayExclusionPolicy` defines telemetry fields excluded from authoritative domain replay equivalence by default.

| Field class | Default replay effect | Inclusion condition |
| --- | --- | --- |
| Trace ID | Excluded. | May not be included in authoritative domain checksums. |
| Span ID | Excluded. | May not be included in authoritative domain checksums. |
| Sampled flag | Excluded. | May affect telemetry emission only. |
| Runtime duration | Excluded. | May be included only in telemetry diagnostic output rows. |
| Runtime start time | Excluded. | May be included only in telemetry diagnostic output rows. |
| Exporter queue state | Excluded from domain replay. | Included only when `TelemetryRuntimeState` is health-visible and manifest-tracked. |
| Exporter delivery result | Excluded from domain replay. | Included only when `TelemetryRuntimeState` is health-visible and manifest-tracked. |
| Collector state | Excluded from domain replay. | Included only when `TelemetryRuntimeState` is health-visible and manifest-tracked. |
| Dropped telemetry counts | Excluded from domain replay. | Included only when `TelemetryRuntimeState` is health-visible and manifest-tracked. |
| Telemetry backend-generated IDs | Excluded and forbidden in domain outputs. | No production inclusion. |

A replay checksum that includes a trace ID, span ID, sampled flag, backend telemetry ID, exporter queue state, or Collector state outside an allowed telemetry health diagnostic row must fail with `TELEMETRY_REPLAY_FIELD_FORBIDDEN`.

## EmitTelemetry Algorithm

```text
EmitTelemetry(signal, attributes, value, context, profile, runtime_state):
1. Validate that `profile` is present, active, in scope, and names one `TelemetrySignalPolicy`.
2. If telemetry is disabled by the active profile, return no-op and do not mutate domain output.
3. Normalize `signal` to one of `trace`, `metric`, `structured_log`, or `baggage`.
4. If the normalized signal is not permitted for production, fail with `TELEMETRY_SIGNAL_FORBIDDEN` before export.
5. Validate the active trace context. If context is missing and tracing or structured logs are enabled, create runtime diagnostic context only.
6. Canonicalize attribute keys and values.
7. Reject any attribute key or value not allowlisted by `TelemetryAttributePolicy`.
8. Run `TelemetryRedactionPolicy` and `110.RedactionPolicy` preflight; reject raw payload bytes, private bindings, credentials, backend IDs, and provider-native query text before export.
9. If `signal = metric`, resolve exactly one `MetricInstrumentRow`, validate instrument type, unit, allowed attributes, cardinality class, and per-run cardinality bound.
10. Enforce `TelemetryNonAuthorityRule`; if telemetry attempts to create, modify, authorize, validate, replay, or persist a non-telemetry authoritative record, fail with `TELEMETRY_AUTHORITY_VIOLATION` before the forbidden effect.
11. Submit the telemetry item through `TelemetryExporterProfile` using bounded non-blocking behavior when export is enabled.
12. On exporter unavailable, timeout, Collector unreachable, or queue overflow, update `TelemetryRuntimeState` counters and failure state; do not change the domain result.
13. Map telemetry runtime state to health only through `TelemetryHealthMappingPolicy` when health output is requested.
14. Return the updated `TelemetryRuntimeState` ref when runtime state is health/API/audit/validation visible; otherwise return telemetry no-op status.
```

Forbidden telemetry content errors are blocking for the telemetry item and must occur before export. Export delivery failures are non-blocking for domain output and health-affecting only through policy.

## ValidateTelemetryProfile Algorithm

```text
ValidateTelemetryProfile(profile, artifact_refs, package_set_ref, metric_catalog, exporter_profile, validation_scope):
1. Validate `profile` through `030.ActivationControlledArtifactRef` with owner `140`.
2. Validate profile lifecycle status, activation scope, checksum, package-set ref when package-supplied, and validation refs.
3. Resolve exactly one active `TelemetrySignalPolicy` for each enabled signal.
4. Validate `TraceContextPolicy` when traces or structured logs are enabled.
5. Validate `TelemetryCorrelationPolicy` before any caller-visible correlation ref is emitted.
6. Validate `MetricInstrumentCatalog` and every `MetricInstrumentRow` when metrics are enabled.
7. Validate `TelemetryAttributePolicy` and `TelemetryRedactionPolicy` for every enabled signal.
8. Validate `TelemetryExporterProfile` when export is enabled or required.
9. Validate exporter timeout, retry, queue, backpressure, shutdown flush, drop behavior, and failure health effect bounds.
10. Validate `TelemetryHealthMappingPolicy` when telemetry can affect `110.OperationalHealthStatus`.
11. Validate `TelemetryReplayExclusionPolicy` when replay, API, audit, validation, or operator-visible diagnostics are in scope.
12. Reject any policy that permits raw payload bytes, private binding leakage, backend ID export, unbounded metric labels, domain authority effects, audit substitution, graph repair, source absence, identity evidence, package activation, watermark advancement, or replay checksum inclusion of forbidden telemetry fields.
13. Require `100.ProductionPackageSetManifest` refs when telemetry artifacts are package-supplied.
14. Require `030.VersionManifest` telemetry policy refs when telemetry policy or runtime state affects health, API, audit, validation, or operator-visible diagnostics.
15. Return pass only when every required `120-OBSERVABILITY-*` validation ref is present and non-blocking.
```

## Error Codes

Telemetry errors feed `110.ErrorCodeRegistry`. `140` owns telemetry-specific causes and owner context except where the row imports the authority error from `010`.

| Error code | Owner | Emitted when |
| --- | --- | --- |
| `TELEMETRY_PROFILE_MISSING` | `140` | A required observability instrumentation profile is absent. |
| `TELEMETRY_PROFILE_INACTIVE` | `140` | A required profile exists but is not active for the execution scope. |
| `TELEMETRY_SIGNAL_FORBIDDEN` | `140` | A signal is unknown, disabled, or production-forbidden. |
| `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `140` | A telemetry attribute key or value is not allowlisted or cannot be safely redacted. |
| `TELEMETRY_CARDINALITY_VIOLATION` | `140` | A metric row or emitted attribute set exceeds the active cardinality policy. |
| `TELEMETRY_RAW_PAYLOAD_FORBIDDEN` | `140` | Raw payload bytes appear in telemetry before export. |
| `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` | `140` | Private source binding, credential, private route, or tenant inventory appears in telemetry before export. |
| `TELEMETRY_BACKEND_ID_FORBIDDEN` | `140` | Backend-generated IDs appear in telemetry before export. |
| `TELEMETRY_EXPORTER_UNAVAILABLE` | `140` | Exporter or Collector is unavailable. |
| `TELEMETRY_EXPORTER_QUEUE_OVERFLOW` | `140` | Telemetry queue is full and a signal is dropped. |
| `TELEMETRY_AUTHORITY_VIOLATION` | `010` imported by `140` | Telemetry attempts a non-telemetry authoritative effect. |
| `TELEMETRY_REPLAY_FIELD_FORBIDDEN` | `140` | A forbidden telemetry runtime field is included in a domain replay checksum or output identity. |
| `TELEMETRY_VERSION_MANIFEST_INCOMPLETE` | `140` | Telemetry policy refs or runtime state refs required by `030.VersionManifest` are missing. |

### TelemetryErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns caller-visible severity, retry class, final redaction, and generic-code precedence.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `TELEMETRY_PROFILE_MISSING` | `140` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-profile-missing` |
| `TELEMETRY_PROFILE_INACTIVE` | `140` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-profile-inactive` |
| `TELEMETRY_SIGNAL_FORBIDDEN` | `140` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-signal-forbidden` |
| `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `140` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-attribute-forbidden` |
| `TELEMETRY_CARDINALITY_VIOLATION` | `140` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-cardinality-violation` |
| `TELEMETRY_RAW_PAYLOAD_FORBIDDEN` | `140` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-raw-payload-forbidden` |
| `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` | `140` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-private-binding-forbidden` |
| `TELEMETRY_BACKEND_ID_FORBIDDEN` | `140` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-backend-id-forbidden` |
| `TELEMETRY_EXPORTER_UNAVAILABLE` | `140` | `error` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-exporter-unavailable` |
| `TELEMETRY_EXPORTER_QUEUE_OVERFLOW` | `140` | `error` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-exporter-queue-overflow` |
| `TELEMETRY_REPLAY_FIELD_FORBIDDEN` | `140` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-replay-field-forbidden` |
| `TELEMETRY_VERSION_MANIFEST_INCOMPLETE` | `140` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `140.TelemetryErrorContext` | `error-registry-140-telemetry-version-manifest-incomplete` |

### TelemetryErrorContext

`TelemetryErrorContext` is the owner context schema for `140` telemetry profile, signal, attribute, cardinality, exporter, replay-exclusion, and telemetry-manifest registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `140` context schema version. |
| `owner_spec` | Yes | Must be `140`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `telemetry_profile`, `signal_policy`, `attribute_policy`, `cardinality`, `forbidden_data_class`, `exporter`, `queue`, `replay_exclusion`, or `version_manifest`. |
| `operation` | Yes | Telemetry profile validation, telemetry emission, attribute validation, metric emission, export, health mapping, replay validation, or manifest validation. |
| `affected_record_type` | Yes | Telemetry profile, signal policy, attribute policy, metric row, exporter profile, runtime state, replay exclusion policy, or version manifest type. |
| `field_path` | Yes | Exact telemetry attribute path, profile field, replay field, or null for artifact-wide failures. |
| `telemetry_profile_ref` | No | Required when profile resolution is involved. |
| `signal_token` | No | Required for signal-policy failures. |
| `attribute_path` | No | Required for attribute failures. |
| `exporter_ref` | No | Required for exporter failures. |
| `queue_state_class` | No | Required for queue overflow. |
| `forbidden_data_class` | No | Required for raw payload, private binding, backend ID, or replay-field failures. |
| `runtime_state_ref` | No | Required when telemetry runtime state affects health/API/audit/validation diagnostics. |
| `validation_refs` | Yes | Exact `120` observability fixture refs. |
| `redaction_classes` | Yes | Raw payload bytes, private bindings, credentials, backend IDs, source-native identity values, canonical IDs, hostnames, IPs, usernames, provider-native query text, exporter credentials, and raw queue payloads must map to `always_forbidden`. |

## Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `140-PROFILE-DEFAULTS-AC-001` | The default instrumentation profile materializes OpenTelemetry-compatible, OTLP-compatible traces, metrics, structured logs, bounded best-effort nonblocking delivery, no domain effect from export failure, degraded health effect from export failure, replay exclusion for trace/span IDs, and forbidden raw payload, private binding, and backend ID telemetry. |
| `140-SIGNAL-POLICY-AC-001` | `trace`, `metric`, `structured_log`, and `baggage` are the only production signal tokens; unknown or disabled signals fail with `TELEMETRY_SIGNAL_FORBIDDEN` before export. |
| `140-TRACE-CONTEXT-AC-001` | Trace context propagation, parent-child spans, async handoff, sampled/unsampled behavior, baggage restrictions, and context loss never alter stage order, output IDs, checksums, replay equivalence, package activation, or domain output. |
| `140-ATTRIBUTE-REDACTION-AC-001` | Raw payload bytes, private bindings, credentials, backend IDs, source-native IDs, canonical IDs, hostnames, IPs, user names, tenant inventories, route names, and provider-native query text are rejected or redacted before export according to `TelemetryAttributePolicy`, `TelemetryRedactionPolicy`, and `110.RedactionPolicy`. |
| `140-CARDINALITY-AC-001` | Every emitted metric resolves exactly one `MetricInstrumentRow`; unbounded attributes and cardinality overrun fail with `TELEMETRY_CARDINALITY_VIOLATION` before export. |
| `140-EXPORTER-AC-001` | Exporter outage, Collector outage, timeout, and queue overflow update `TelemetryRuntimeState`, may degrade observability health through policy, and do not mutate authoritative domain records. |
| `140-HEALTH-AC-001` | Telemetry runtime state affects `110.OperationalHealthStatus` only through `TelemetryHealthMappingPolicy` and never repairs, invalidates, or mutates domain outputs. |
| `140-REPLAY-AC-001` | Trace IDs, span IDs, sampled flags, runtime duration, exporter queue state, exporter result, Collector state, dropped counts, and backend telemetry IDs are excluded from authoritative domain replay checksums by default. |
| `140-NONAUTH-AC-001` | Telemetry cannot create, modify, authorize, validate, retract, expire, merge, split, project, replay, or persist non-telemetry authoritative Cadastre records. |
| `140-VERSION-MANIFEST-AC-001` | Health, API, audit, validation, or operator-visible diagnostics that depend on telemetry policy or runtime state include required telemetry policy refs and runtime state refs in `030.VersionManifest` or fail with `TELEMETRY_VERSION_MANIFEST_INCOMPLETE`. |
| `140-PACKAGE-AC-001` | Package-supplied telemetry profiles, metric catalogs, redaction policies, exporter profiles, health policies, and replay exclusion policies fail closed unless `100.ProductionPackageSetManifest`, package type policy, validation refs, and artifact checksums pass. |
| `140-RUNLOCK-TELEMETRY-AC-001` | Run-lock acquisition, heartbeat, recovery, loss, and lease-remaining metrics use only bounded low-cardinality attributes and validation refs from `120-RUNLOCK-OBSERVABILITY-*`. |
| `140-RUNLOCK-NONAUTH-AC-001` | Dropping all run-lock telemetry does not change lock outcome, domain output, replay checksum, package activation, graph apply, table maintenance, or watermark state. |

## Definition of Done

- Every exported telemetry contract has one owner in `000.Owner Contract Registry` and one volatility classification.
- `EmitTelemetry` and `ValidateTelemetryProfile` reject forbidden telemetry before export.
- Telemetry-enabled and telemetry-disabled runs produce byte-identical authoritative domain outputs for the same inputs.
- Telemetry exporter failure degrades observability health only through policy and never mutates domain records.
- `120-OBSERVABILITY-*` validation rows cover non-authority, redaction, cardinality, exporter failure, health mapping, replay, audit independence, version manifest completeness, and package activation.

## Open Questions

| ID | Question | Blocking scope |
| --- | --- | --- |
| `140-TODO-CORRELATION-REF-ALGORITHM` | Product security governance must choose the exact opaque diagnostic correlation ref derivation before caller-visible production exposure. | Blocks exposed non-null `diagnostic_correlation_ref`. |
| `140-TODO-EXPORTER-DEFAULT-BOUNDS` | Product operations governance must choose production default exporter timeout, retry, queue, and shutdown flush values. | Blocks required production export when no active exporter profile supplies the values. |
