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
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ScopeSelectorCovers`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`

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

### RunLockTelemetryValidationHandoff

`val-140-runlock-telemetry-cardinality` must prove that run-lock metric attributes are bounded low-cardinality and that raw lock keys, raw fencing tokens, raw idempotency keys, source-native IDs, canonical IDs, private bindings, and backend IDs are rejected or redacted before export. Rejected telemetry items must not mutate domain output.

`val-140-runlock-telemetry-nonauthority` must prove that dropping all run-lock telemetry changes no lock outcome, domain output, replay checksum, package activation, graph apply, table maintenance, watermark state, or audit evidence substitution.

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

## TelemetryScopeSelectorContext

`TelemetryScopeSelectorContext` is the owner context family for instrumentation profile, signal policy, metric catalog, exporter profile, attribute policy, redaction policy, health mapping, and replay exclusion policy activation. It instantiates `030.ScopeSelectorContext`; it does not grant domain authority.

| Telemetry artifact family | Required dimensions | Optional dimensions | Default subset behavior | Scope-match effect |
| --- | --- | --- | --- | --- |
| instrumentation profile | `stage_class`, `execution_mode` | `package_set_ref`, `deployment_scope` | none | Enables local telemetry construction only. |
| signal policy | `signal_token`, `stage_class` | `execution_mode` | none | Determines whether the signal may be emitted. |
| metric catalog | `metric_name`, `stage_class` | `deployment_scope` | none | Determines whether the metric row may export. |
| exporter profile | `exporter_profile_id`, `deployment_scope` | `stage_class` | none | Determines export eligibility and health effect. |
| attribute policy | `attribute_key`, `signal_token` | `stage_class` | none | Determines attribute retention or rejection. |
| redaction policy | `data_class`, `signal_token` | `stage_class` | none | Determines telemetry redaction only. |
| health mapping | `telemetry_condition`, `health_scope` | `deployment_scope` | none | Determines observability health mapping only. |
| replay exclusion policy | `field_class`, `output_class` | `stage_class` | none | Determines telemetry replay exclusion only. |

Telemetry selector matches determine telemetry emission, export, and health-mapping eligibility only. They must not affect domain outputs, replay checksums for non-telemetry output, watermarks, graph apply, package activation, identity, facts, completeness, source authority, or source absence.

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

### diagnostic_correlation_ref_v1

`diagnostic_correlation_ref_v1(policy_row, activation_scope_checksum, policy_row_checksum, diagnostic_correlation_secret_ref, telemetry_context) -> string | null` is the only production derivation algorithm for caller-visible diagnostic correlation refs.

Inputs:

| Input | Required behavior |
| --- | --- |
| `policy_row` | Active `TelemetryCorrelationPolicy` row selected for the output context. |
| `activation_scope_checksum` | Checksum of the selected activation scope. It must equal `policy_row.activation_scope.selector_checksum`. |
| `policy_row_checksum` | Checksum of `policy_row` after defaults materialize. It must equal `policy_row.row_checksum`. |
| `diagnostic_correlation_secret_ref` | Private secret reference selected from the policy row. The secret bytes are never serialized in public artifacts. |
| `telemetry_context` | In-memory runtime telemetry context for the request or stage handoff. |

The algorithm must return `null` without fallback when telemetry is disabled, the selected policy row is inactive, the selected policy row is outside the activation scope, emission is disabled for the requested output context, the secret is missing, invalid, expired, checksum-mismatched, or the caller lacks permission for the output context. Omission of `diagnostic_correlation_secret_ref` returns `null`; it must not expose a raw trace ID, raw span ID, baggage value, audit ID, graph ID, evidence ref, package proof, or replay key.

When no `server_correlation_nonce` exists in `telemetry_context`, the runtime must generate exactly 128 bits from a CSPRNG and store it only in that in-memory telemetry context. The nonce may be copied only to an asynchronous in-memory runtime handoff envelope for the same request or stage execution. It must not be persisted as a `030.StageStateRecord`, `030.VersionManifest` input, metric attribute, API identifier, audit identifier, graph selector, replay key, evidence ref, package proof, or validation output outside validation-owned fixed-nonce fixtures.

The HMAC preimage is the byte concatenation below. `NUL` means a single `0x00` byte.

```text
"cadastre.diagnostic_correlation_ref.v1" NUL
policy_row.row_checksum NUL
policy_row.activation_scope.selector_checksum NUL
server_correlation_nonce_bytes
```

The algorithm computes `HMAC-SHA-256(secret_bytes, preimage)`, takes the first 16 bytes of the HMAC output, encodes those bytes as lowercase unpadded base32hex, and returns `"dcr1-" + encoded`.

The returned value is diagnostic support material only. It must match the regular expression `^dcr1-[0-9a-v]{26}$`, must not be caller-supplied as authority, and must remain excluded from domain output IDs, domain output checksums, and authoritative replay equivalence unless an owner output-class row explicitly includes it only as a diagnostic display field.

### TelemetryCorrelationPolicy field precision

`TelemetryCorrelationPolicy` is an activation-controlled row family and must use the `030.ActivationControlledRowField` column order below.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty, owner-scoped stable token | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected row ref and checksum required when diagnostic refs can be emitted | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `algorithm` | `enum` | `yes` | `diagnostic_correlation_ref_v1` | `no` | `yes` | exactly `diagnostic_correlation_ref_v1` | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | selected algorithm value and policy row checksum required | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_PROFILE_INVALID` |
| `emit_diagnostic_correlation_ref` | `boolean` | `yes` | `false` | `no` | `yes` | `true` or `false` | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | required when API or audit output may emit the ref | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_PROFILE_INVALID` |
| `diagnostic_correlation_secret_ref` | `private_secret_ref` | `conditional:emit_diagnostic_correlation_ref=true` | `null` | `yes` | `yes` | opaque private ref plus secret checksum; raw secret bytes forbidden | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | private ref checksum required when non-null refs can be emitted | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` |
| `secret_validation_policy_ref` | `030.ActivationControlledArtifactRef` | `conditional:diagnostic_correlation_secret_ref!=null` | `null` | `yes` | `yes` | active secret validation policy ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | required for non-null diagnostic refs | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_PROFILE_INVALID` |
| `allowed_output_contexts` | `array<enum>` | `yes` | `[]` | `no` | `yes` | values from `api_response`, `audit_event`, `operator_diagnostic`, `validation_diagnostic`; empty means no emission | `canonical_set` | `reject` | `value` | `no` | `yes` | `closed` | `110` | selected contexts required when visible output emits ref | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_PROFILE_INVALID` |
| `raw_trace_span_fallback` | `enum` | `yes` | `forbidden` | `no` | `yes` | exactly `forbidden` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required for visible diagnostics | `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| `nonce_persistence_policy` | `enum` | `yes` | `in_memory_only` | `no` | `yes` | exactly `in_memory_only` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected policy ref required when non-null refs can be emitted | `TELEMETRY_CORRELATION_REF_UNAVAILABLE` | `TELEMETRY_REPLAY_FIELD_FORBIDDEN` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty; must include fixed-nonce and null-secret validation rows before production emission | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover output request through `TelemetryScopeSelectorContext` | `n/a` | `reject` | `n/a` | `ordered:99` | `yes` | `closed` | `110` | selector context ref and selector checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production selection requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle transition evidence required when selection changes | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex over canonical row bytes after defaults, excluding only `row_checksum` | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |

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
| `cadastre.graph.backend.preflight.count` | counter | `{preflight}` | `graph_backend_profile_ref`, `graph_backend_provider_token`, `graph_backend_preflight_status`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-BACKEND-*` |
| `cadastre.graph.query.duration_ms` | histogram | `ms` | `graph_backend_profile_ref`, `graph_backend_provider_token`, `graph_query_class`, `query_plan_class`, `package_set_ref`, `validation_row_ref` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-QUERY-*` |
| `cadastre.graph.apply.batch.size` | histogram | `{delta}` | `graph_backend_profile_ref`, `graph_backend_provider_token`, `package_set_ref`, `validation_row_ref` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-APPLY-*` |
| `cadastre.graph.apply.duration_ms` | histogram | `ms` | `graph_backend_profile_ref`, `graph_backend_provider_token`, `package_set_ref`, `validation_row_ref` | `bounded_low` | explicit bucket histogram | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-APPLY-*` |
| `cadastre.graph.restore.rehearsal.status` | counter | `{rehearsal}` | `graph_backend_profile_ref`, `restore_rehearsal_ref`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-RESTORE-UPGRADE-*` |
| `cadastre.graph.upgrade.rehearsal.status` | counter | `{rehearsal}` | `graph_backend_profile_ref`, `upgrade_rehearsal_ref`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-RESTORE-UPGRADE-*` |
| `cadastre.graph.benchmark.status` | counter | `{benchmark}` | `graph_backend_profile_ref`, `benchmark_threshold_row_ref`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-BENCHMARK-*` |
| `cadastre.graph.provider.support.status` | counter | `{support}` | `graph_backend_profile_ref`, `graph_backend_provider_token`, `provider_support_status`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-BACKEND-*` |
| `cadastre.graph.schema_fingerprint.status` | counter | `{fingerprint}` | `graph_backend_profile_ref`, `backend_schema_fingerprint_ref`, `package_set_ref`, `validation_row_ref` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-CARDINALITY-*`, `120-GRAPH-POSTGRES-SCHEMA-*` |
| `cadastre.telemetry.exporter.failures` | counter | `{failure}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.exporter.queue_depth` | gauge | `{item}` | `telemetry_profile_ref`, `exporter_profile_ref` | `bounded_low` | last value | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.dropped_spans` | counter | `{span}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.telemetry.dropped_metrics` | counter | `{metric}` | `telemetry_profile_ref`, `exporter_profile_ref`, `owner_error_code` | `bounded_low` | monotonic sum | `120-OBSERVABILITY-EXPORTER-*` |
| `cadastre.structured_input.snapshot.validated` | counter | `{snapshot}` | `repository_profile_ref`, `structured_input_operation`, `artifact_class`, `owner_spec`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-*` |
| `cadastre.structured_input.validation.failed` | counter | `{failure}` | `repository_profile_ref`, `structured_input_operation`, `artifact_class`, `owner_spec`, `owner_error_code`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-*` |
| `cadastre.structured_input.materialization.completed` | counter | `{materialization}` | `repository_profile_ref`, `structured_input_operation`, `artifact_class`, `owner_spec`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-*` |
| `cadastre.structured_input.materialization.failed` | counter | `{failure}` | `repository_profile_ref`, `structured_input_operation`, `artifact_class`, `owner_spec`, `owner_error_code`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-*` |
| `cadastre.structured_input.private_binding.rejected` | counter | `{rejection}` | `repository_profile_ref`, `structured_input_operation`, `artifact_class`, `owner_spec`, `owner_error_code` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-*` |
| `cadastre.structured_input.template.validation.completed` | counter | `{validation}` | `repository_profile_ref`, `structured_input_operation`, `template_contract_ref`, `artifact_class`, `owner_spec`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-TEMPLATE-*` |
| `cadastre.structured_input.producer_ci.validation.failed` | counter | `{failure}` | `repository_profile_ref`, `structured_input_operation`, `producer_ci_contract_ref`, `artifact_class`, `owner_spec`, `owner_error_code`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-CI-*` |
| `cadastre.structured_input.publication.import.completed` | counter | `{import}` | `repository_profile_ref`, `structured_input_operation`, `publication_manifest_ref`, `artifact_class`, `owner_spec`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-PUBLICATION-*` |
| `cadastre.structured_input.publication.import.failed` | counter | `{failure}` | `repository_profile_ref`, `structured_input_operation`, `publication_manifest_ref`, `artifact_class`, `owner_spec`, `owner_error_code`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-PUBLICATION-*` |
| `cadastre.structured_input.sync.candidate.discovered` | counter | `{candidate}` | `repository_profile_ref`, `structured_input_operation`, `candidate_sync_record_ref`, `artifact_class`, `owner_spec`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-SYNC-*` |
| `cadastre.structured_input.repository_group.validation.failed` | counter | `{failure}` | `repository_profile_ref`, `structured_input_operation`, `repository_group_ref`, `artifact_class`, `owner_spec`, `owner_error_code`, `validation_status` | `bounded_low` | monotonic sum | `120-STRUCTURED-INPUT-MULTIREPO-*` |
| `cadastre.run_lock.acquired` | counter | `{lock}` | `lock_scope_class`, `output_class`, `owner_error_code` | `bounded_low` | monotonic sum | `val-140-runlock-telemetry-cardinality` |
| `cadastre.run_lock.heartbeat` | counter | `{heartbeat}` | `lock_scope_class`, `result`, `owner_error_code` | `bounded_low` | monotonic sum | `val-140-runlock-telemetry-cardinality` |
| `cadastre.run_lock.recovery_attempt` | counter | `{attempt}` | `lock_scope_class`, `result`, `owner_error_code` | `bounded_low` | monotonic sum | `val-140-runlock-telemetry-cardinality` |
| `cadastre.run_lock.lost` | counter | `{lock}` | `lock_scope_class`, `output_class`, `owner_error_code` | `bounded_low` | monotonic sum | `val-140-runlock-telemetry-cardinality` |
| `cadastre.run_lock.lease_remaining` | gauge | `s` | `lock_scope_class`, `output_class` | `bounded_low` | last value | `val-140-runlock-telemetry-cardinality` |

The active metric catalog may add rows only through activation-controlled artifacts. A package-supplied catalog must pass `ValidateTelemetryProfile` and package activation before production use.

Run-lock metric rows must not allow attributes containing raw lock keys, source-native identifiers, canonical IDs, private source bindings, raw fencing token values, raw idempotency key values, or backend IDs. Only redacted refs or checksums may be emitted when `TelemetryAttributePolicy`, `TelemetryRedactionPolicy`, and `110.RedactionPolicy` permit them.

## Metric Instrument Row Schema

`MetricInstrumentRow` is the stable row interface inside `MetricInstrumentCatalog`. It is an activation-controlled row family and must use the `030.ActivationControlledRowField` column order below.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `metric_name` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | lowercase dotted token, `3..128` characters | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected metric row ref and checksum required when emitted or health/API/audit/validation visible | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `instrument_type` | `enum` | `yes` | `none` | `no` | `no` | `counter`, `gauge`, or `histogram` | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `unit` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | UCUM-compatible token or bounded literal such as `{item}` | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `description` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | maximum 512 Unicode scalar values | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `allowed_attribute_keys` | `array<040.ScalarType.string>` | `yes` | `[]` | `no` | `yes` | each key must resolve through `TelemetryAttributePolicy`; unbounded identifiers forbidden | `canonical_set` | `reject` | `value` | `no` | `yes` | `closed` | `110` | selected attribute policy refs required for non-empty set | `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `TELEMETRY_CARDINALITY_VIOLATION` |
| `cardinality_class` | `enum` | `yes` | `bounded_low` unless default mapping below narrows or widens | `no` | `yes` | `static`, `bounded_low`, `bounded_medium`; `forbidden_unbounded` cannot be active | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_CARDINALITY_VIOLATION` |
| `max_distinct_attribute_sets_per_run` | `integer` | `yes` | derived from default mapping below | `no` | `yes` | `1..1024`; `static = 1`, `bounded_low <= 64`, `bounded_medium <= 1024` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized value required before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_CARDINALITY_VIOLATION` |
| `histogram_bucket_profile` | `enum` | `conditional:instrument_type=histogram` | `duration_ms_buckets` for `unit = ms`; `count_item_buckets` for count-like units | `no` | `yes` | `duration_ms_buckets` or `count_item_buckets` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected profile included in row checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `duration_ms_buckets` | `array<integer>` | `conditional:histogram_bucket_profile=duration_ms_buckets` | `[1,5,10,25,50,100,250,500,1000,2500,5000,10000,30000]` | `no` | `yes` | ascending positive integer milliseconds, max `300000` | `ordered_sequence` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `count_item_buckets` | `array<integer>` | `conditional:histogram_bucket_profile=count_item_buckets` | `[1,2,5,10,25,50,100,250,500,1000,5000]` | `no` | `yes` | ascending positive integer counts, max `65536` | `ordered_sequence` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `aggregation` | `enum` | `yes` | `monotonic_sum` for counters, `last_value` for gauges, `explicit_bucket_histogram` for histograms | `no` | `yes` | one valid aggregation for the instrument type | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `export_temporality` | `enum` | `no` | `cumulative` | `no` | `yes` | `cumulative` or `delta` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `attribute_set_overrun_behavior` | `enum` | `yes` | `drop_telemetry_item_nonblocking` | `no` | `yes` | exactly `drop_telemetry_item_nonblocking` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | runtime overrun state visible only through telemetry runtime state refs | `TELEMETRY_CARDINALITY_VIOLATION` | `TELEMETRY_CARDINALITY_VIOLATION` |
| `exemplar_policy` | `enum` | `yes` | `forbidden` | `no` | `yes` | exactly `forbidden` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty `120-OBSERVABILITY-*` refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover emission scope | `n/a` | `reject` | `n/a` | `ordered:99` | `yes` | `closed` | `110` | selector context ref and selector checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required when status affects selection | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex over canonical row bytes after defaults, excluding only `row_checksum` | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |

Metric cardinality defaults must materialize before metric row checksum computation.

| Cardinality input | Materialized `max_distinct_attribute_sets_per_run` |
| --- | ---: |
| `static` | 1 |
| `bounded_low` | 64 |
| `bounded_medium` | 1024 |
| `cadastre.stage.started` | 1024 |
| `cadastre.stage.completed` | 1024 |
| any other default metric row | 64 unless narrowed by an active row |

Metric exemplars are disabled for MVP. A metric point containing exemplar trace IDs, span IDs, sampled flags, exemplar-filter attributes, or any trace-context-derived value must fail before export with `TELEMETRY_ATTRIBUTE_FORBIDDEN` and must not mutate domain output.

## Telemetry Attribute Policy

`TelemetryAttributePolicy` is an activation-controlled allowlist for telemetry attributes. Attribute keys and values must be normalized before validation.

Default allowed attribute keys:

| Attribute key | Allowed value class | Maximum distinct values per run | Rule |
| --- | --- | ---: | --- |
| `repository_profile_ref` | Structured input repository profile ref | 64 | Ref and checksum only; no repository URL or private route. |
| `structured_input_operation` | Closed operation token | 64 | One of `snapshot`, `validate`, `template_validate`, `producer_ci_validate`, `tool_invoke`, `materialize`, `publication_import`, `candidate_sync`, `repository_group_validate`, `package_release_handoff`, `package_set_candidate_stage`, `promote`, `rollback`, `private_binding_reject`; raw branch names are forbidden. |
| `artifact_class` | Closed `030.ActivationControlledArtifactRef.artifact_class` token | 256 | Token only; no file path or package filename. |
| `owner_spec` | Owner spec token | 32 | One of registered owner spec IDs. |
| `validation_status` | Closed validation status token | 16 | One of `pass`, `fail`, `blocked`, `not_run`, `stale`; must not include diagnostic text. |
| `stage_id` | Cadastre stage ID | 1024 | Must not encode private route or source-native ID. |
| `stage_class` | Closed `030` stage class | 64 | Must match `030` stage class vocabulary. |
| `run_id` | Opaque run ref | 1024 | Runtime ref only; excluded from domain replay when volatile. |
| `version_manifest_ref` | `030.VersionManifest` ref | 1024 | Ref only; no manifest bytes. |
| `package_set_ref` | `100.ProductionPackageSetManifest` ref | 1024 | Ref only; no package payload bytes. |
| `activation_artifact_ref` | `030.ActivationControlledArtifactRef` ref | 4096 | Ref and checksum only. |
| `publication_manifest_ref` | Ref/checksum only | 1024 | No manifest bytes or artifact payload bytes. |
| `candidate_sync_record_ref` | Ref/checksum only | 1024 | Sync telemetry is diagnostic only and must not imply activation. |
| `repository_group_ref` | Ref/checksum only | 1024 | No private member repository route or URL. |
| `template_contract_ref` | Ref/checksum only | 1024 | Template conformance telemetry is diagnostic only. |
| `producer_ci_contract_ref` | Ref/checksum only | 1024 | No raw CI logs or branch names. |
| `maintenance_tool_contract_ref` | Ref/checksum only | 1024 | No raw tool logs, command lines with private values, or generated bytes. |
| `owner_error_code` | Owner error code | 256 | Error code only; no raw owner context. |
| `lifecycle_transition_evidence_ref` | `030.LifecycleTransitionEvidence` ref | 4096 | Ref and checksum only. |
| `validation_row_ref` | `120.ValidationMatrix` row ref | 4096 | Row ref only. |
| `health_scope` | `110.OperationalHealthRequest.health_scope` token | 32 | Includes `observability`. |
| `telemetry_profile_ref` | `ObservabilityInstrumentationProfile` ref | 64 | Ref and checksum only. |
| `exporter_profile_ref` | `TelemetryExporterProfile` ref | 64 | Ref and checksum only. |
| `graph_operation` | Closed graph operation class | 64 | Diagnostic only; must not include provider query text. |
| `graph_backend_profile_ref` | Ref/checksum only | 64 | No provider config bytes. |
| `graph_backend_provider_token` | Closed provider token | 16 | One of `postgresql`, `postgresql_age`, explicit non-default providers. |
| `graph_backend_preflight_status` | Closed preflight token | 16 | One of `pass`, `fail`, `blocked`, `not_run`, `stale`. |
| `backend_schema_fingerprint_ref` | Ref/checksum only | 1024 | No schema DDL text. |
| `postgresql_version_ref` | Bounded version ref | 32 | No connection string. |
| `age_extension_version_ref` | Bounded version ref | 32 | Only when AGE is selected. |
| `graph_query_class` | Closed query class | 32 | No SQL, Cypher, or SQL/Cypher text. |
| `provider_support_status` | Closed support-status token | 16 | No account, tenant, database, region-private, or provider-private identifiers. |
| `query_plan_class` | Closed diagnostic class | 64 | No literal values or raw plan text. |
| `endpoint_class` | Public endpoint class | 64 | Diagnostic only; must not include route names or private paths. |

Forbidden values include raw payload bytes, raw payload hashes when not authorized by `110`, private source bindings, private routes, credentials, secrets, tokens, tenant inventories, environment-specific source target lists, backend internal IDs, source-native IDs, canonical entity IDs, raw hostnames, IP addresses, user names, provider-native query text, raw SQL text, raw Cypher text, SQL/Cypher composed text, SQL literals, Cypher literals, query plans containing literals, connection strings, private database names, private schema names, PostgreSQL OIDs, tuple IDs, sequence values, prepared statement names, cursor names, AGE vertex IDs, AGE edge IDs, AGE path IDs, AGE graph namespace names when private, AGE graph/label IDs, agtype object IDs, raw graph property values, package payload bytes, and unredacted source identifiers.

A forbidden key or value must fail before export with the most specific telemetry error code.

Structured-input telemetry must not contain raw branch names unless redacted to a bounded class, raw file paths unless hashed or redacted, raw schema bytes, private routes, credentials, source-native identifiers, tenant inventories, raw fixture bytes, raw structured input bytes, or commit messages containing private data. Commit SHA may be emitted only as a checksum-like diagnostic when `110.StructuredInputRepositoryRedactionPolicy` and `TelemetryAttributePolicy` permit it; it must not become replay evidence, audit persistence evidence, activation authority, rollback authority, or source-authority evidence.

Structured-input telemetry for template validation, producer CI validation, tool invocation, publication import, candidate sync, repository group validation, package release handoff, and package-set candidate staging is diagnostic material only. It must not prove validation, activation, audit persistence, replay equivalence, package activation, source authority, graph output, or domain output.

### Graph backend telemetry attribute closure

Graph backend telemetry attributes are diagnostic-only and must use closed tokens, refs, or checksums. The allowlist below is exhaustive for PostgreSQL/AGE backend diagnostics until a later active `TelemetryAttributePolicy` row adds a bounded key.

| Attribute key | Allowed value class | Maximum distinct values per run | Required behavior |
| --- | --- | ---: | --- |
| `graph_backend_profile_ref` | Ref/checksum | 64 | No provider config bytes. |
| `graph_backend_provider_token` | Closed token | 16 | `postgresql`, `postgresql_age`, or explicit non-default provider token. |
| `graph_backend_preflight_status` | Closed token | 16 | `pass`, `fail`, `blocked`, `not_run`, or `stale`. |
| `backend_schema_fingerprint_ref` | Ref/checksum | 1024 | No DDL text or private schema name. |
| `postgresql_version_ref` | Ref/checksum | 64 | No connection string. |
| `age_extension_version_ref` | Ref/checksum | 64 | Present only when AGE is selected or in validation scope. |
| `graph_query_class` | Closed query class | 32 | No SQL, Cypher, SQL/Cypher text, or literals. |
| `provider_support_status` | Closed token | 16 | `supported`, `unsupported`, `preview`, `unknown`, `expired`, or `not_run`. |
| `query_plan_class` | Closed diagnostic class | 64 | No literal-bearing plan text. |
| `benchmark_threshold_row_ref` | Ref/checksum | 1024 | Required only when benchmark gates are in scope. |
| `restore_rehearsal_ref` | Ref/checksum | 1024 | No raw restore log bytes. |
| `upgrade_rehearsal_ref` | Ref/checksum | 1024 | No raw migration SQL or provider log bytes. |
| `package_set_ref` | Ref/checksum | 1024 | No package payload bytes. |
| `validation_row_ref` | Ref/checksum | 4096 | No raw fixture bytes. |

A telemetry row that carries raw SQL, raw Cypher, SQL/Cypher text, query literals, literal-bearing query plans, connection strings, private database names, private schema names, PostgreSQL OIDs, tuple IDs, sequence values, physical row locators, prepared statement names, cursor names, AGE vertex/edge/path/graph/label IDs, agtype IDs, backend-generated graph IDs, raw graph property values, credentials, private bindings, hostnames, IP addresses, usernames, source-native IDs, or raw package evidence must fail before export with the most specific telemetry error.

Graph backend telemetry health mapping rows must preserve non-authority boundaries:

| Telemetry condition | Required health behavior | Forbidden substitution |
| --- | --- | --- |
| Exporter failure during backend diagnostics | May affect observability health only. | Must not mark graph backend unhealthy by itself. |
| Backend preflight telemetry says `pass` | Diagnostic only. | Must not mark backend healthy without `090` preflight and validation rows. |
| Benchmark telemetry present | Diagnostic only. | Must not substitute for benchmark threshold validation rows. |
| Restore or upgrade telemetry present | Diagnostic only. | Must not substitute for `090.GraphRestoreRehearsalEvidenceRow` or `090.GraphUpgradeRehearsalEvidenceRow`. |
| Provider support telemetry present | Diagnostic only. | Must not substitute for `090.GraphProviderSupportEvidenceRow`. |

### TelemetryActivationControlledRowFieldClosure

The row families `TelemetryAttributePolicy`, `TelemetryRedactionPolicy`, and `TelemetryHealthMappingPolicy` must be represented by `030.ActivationControlledRowField` tables using the exact `030` column order before production telemetry export or health mapping can depend on them.

| Row family | Required field paths | Defaults and bounds | Manifest and checksum behavior | Missing or invalid error |
| --- | --- | --- | --- | --- |
| `TelemetryAttributePolicy` | `attribute_key`, `signal_token`, `value_type`, `allowed_value_class`, `max_distinct_values_per_run`, `cardinality_class`, `redaction_behavior`, `public_private_constraint`, `package_set_ref`, `validation_refs`, `activation_scope`, `lifecycle_status` | `cardinality_class` defaults are forbidden; production rows must choose `static`, `bounded_low`, or `bounded_medium`; `max_distinct_values_per_run` maximum is `4096` and may be narrowed by key; unbounded labels are forbidden | Attribute key, signal token, value type, value class, cardinality, redaction behavior, package-set ref, validation refs, and activation scope are checksum inputs and must appear in `030.VersionManifest` when health, API, audit, validation, or export visibility depends on the row | `TELEMETRY_ATTRIBUTE_FORBIDDEN`, `TELEMETRY_CARDINALITY_VIOLATION`, or `TELEMETRY_PROFILE_INVALID` |
| `TelemetryRedactionPolicy` | `data_class`, `signal_token`, `redaction_transform`, `forbidden_value_classes`, `public_visibility`, `audit_visibility`, `package_set_ref`, `validation_refs`, `activation_scope`, `lifecycle_status` | Default transform for forbidden backend data is `reject_before_export`; omission of transform is invalid; arrays use `canonical_set` with duplicate rejection | Selected redaction row refs, checksums, package-set refs, and validation refs must appear in `030.VersionManifest` when visible diagnostics depend on redaction | `TELEMETRY_ATTRIBUTE_FORBIDDEN`, `TELEMETRY_BACKEND_ID_FORBIDDEN`, or `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` |
| `TelemetryHealthMappingPolicy` | `telemetry_condition`, `health_scope`, `required_health_effect`, `forbidden_domain_effect`, `owner_error_refs`, `package_set_ref`, `validation_refs`, `activation_scope`, `lifecycle_status` | `required_health_effect` is one of `healthy`, `degraded`, `blocked`, or `error`; `forbidden_domain_effect` must be `no_domain_mutation`; omission is invalid | Selected health mapping row refs, telemetry runtime state refs, validation refs, generated error registry checksum, and package-set refs when supplied must appear in `030.VersionManifest` for health/API/audit/validation-visible diagnostics | `TELEMETRY_PROFILE_INVALID` or owner health error |

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
| Raw SQL text, raw Cypher text, or SQL/Cypher composed text | Reject before export. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| SQL literal, Cypher literal, or query plan containing literal values | Reject before export. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Connection string, private database name, private schema name, or private AGE graph namespace name | Reject before export. | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` |
| PostgreSQL OID, tuple ID, sequence value, physical row locator, prepared statement name, or cursor name | Reject before export. | `TELEMETRY_BACKEND_ID_FORBIDDEN` |
| AGE vertex ID, AGE edge ID, AGE path ID, AGE graph ID, AGE label ID, or agtype object ID | Reject before export. | `TELEMETRY_BACKEND_ID_FORBIDDEN` |
| Raw graph property value | Reject before export. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| Exporter endpoint or Collector route | Redact to opaque ref before telemetry or API output. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |

Redaction that changes a metric attribute value must re-run metric cardinality validation on the redacted value set before export.

## Telemetry Exporter Profile

`TelemetryExporterProfile` is an activation-controlled artifact. A production exporter profile must name endpoint refs, delivery bounds, failure behavior, health mapping refs, redaction refs, and validation refs. Omitted fields that have defaults in this section must materialize before row checksum computation and before exporter startup. Omitted required fields with no default fail profile validation.

### Default exporter profile

| Field | Default value | Bounds or required behavior |
| --- | --- | --- |
| `protocol` | `otlp` | Only `otlp` is active for MVP. |
| `endpoint_ref` | `null` unless export is enabled | When export is enabled, the value must be an opaque ref. Raw URLs, credentials, headers, and private routes are forbidden in public artifacts. |
| `collector_required` | `true` | Direct vendor export is forbidden unless a future active policy permits it. |
| `timeout_ms` | `2000` | Integer range `100..30000`. |
| `retry_policy.enabled` | `true` | `false` disables retries but not the first attempt. |
| `retry_policy.max_retries` | `5` | Integer range `0..10`. |
| `retry_policy.max_elapsed_ms` | `30000` | Integer range `0..300000`; `0` disables retries. |
| `retry_policy.initial_backoff_ms` | `100` | Integer range `50..30000`. |
| `retry_policy.max_backoff_ms` | `5000` | Integer range `100..60000`. |
| `retry_policy.backoff_multiplier` | `"2.0"` | Decimal string range `1.0..5.0`. |
| `retry_policy.jitter` | `full_jitter` | Validation fixtures may inject fixed jitter samples; runtime configuration must not inject deterministic jitter. |
| `retry_policy.retryable_classes` | listed below | Canonical set with duplicate rejection. |
| `queue_max_items` | `2048` | Integer range `1..65536`. |
| `export_batch_max_items` | `512` | Integer range `1..queue_max_items`. |
| `schedule_delay_ms.trace` | `5000` | Integer range `100..300000`. |
| `schedule_delay_ms.metric` | `60000` | Integer range `100..300000`. |
| `schedule_delay_ms.structured_log` | `1000` | Integer range `100..300000`. |
| `backpressure_behavior` | `drop_newest_nonblocking` | Blocking domain writes is forbidden. |
| `shutdown_flush_timeout_ms` | `5000` | Integer range `0..30000`. |
| `failure_health_effect` | `degraded` | Observability health only. Domain effect is always `none`. |
| `metric_exemplars_enabled` | `false` | Any `true` value fails profile validation. |
| `otel_environment_passthrough` | `false` | OpenTelemetry SDK environment variables and autoconfiguration must not determine Cadastre behavior. |

Retryable classes are exactly the canonical set below:

```text
transient_transport_error
http_429
http_502
http_503
http_504
grpc_unavailable
grpc_deadline_exceeded
grpc_resource_exhausted
```

Permanent rejection is terminal and must not retry.

### Exporter retry envelope

The first export attempt has no delay. Retry attempt index starts at `1`. For each retry, compute:

```text
base_interval_ms = min(max_backoff_ms, initial_backoff_ms * backoff_multiplier^(retry_attempt_index - 1))
retry_delay_ms = uniform_integer(0, base_interval_ms)
```

No retry may start after `max_elapsed_ms`. `max_elapsed_ms = 0` disables retries. No new retry loop may start after shutdown begins. A retry already in progress may finish only within `shutdown_flush_timeout_ms`; after that timeout the exporter must record shutdown flush timeout and continue shutdown without blocking domain output. Validation fixtures may inject fixed jitter samples by fixture-owned input only. Production runtime configuration must not supply fixed jitter samples or deterministic random seeds.

### Exporter runtime condition mapping

| Exporter condition | `TelemetryRuntimeState.exporter_failure_state` | Health effect | Domain effect |
| --- | --- | --- | --- |
| export disabled by active profile | `export_disabled` | healthy for telemetry-disabled profile; not an exporter failure | none |
| exporter endpoint unavailable | `endpoint_unavailable` | degraded unless active health mapping narrows to blocked for observability diagnostics | none |
| Collector unreachable | `collector_unreachable` | degraded unless active health mapping narrows to blocked for observability diagnostics | none |
| export attempt timeout | `timeout` | degraded | none |
| queue overflow | `queue_overflow` | degraded unless active health mapping narrows to blocked for observability diagnostics | none |
| redaction rejection | `redaction_rejection` | degraded for observability diagnostics when non-recursive runtime state can be recorded | none |
| invalid required exporter profile | `profile_invalid` | blocked for observability health and telemetry-visible output | none |
| shutdown flush timeout | `shutdown_flush_timeout` | degraded | none |
| permanent rejection | `permanent_rejection` | degraded or blocked according to active health mapping | none |
| retry max elapsed exhausted | `retry_max_elapsed_exhausted` | degraded | none |

### TelemetryExporterProfile field precision

`TelemetryExporterProfile` must use the `030.ActivationControlledRowField` column order below.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `exporter_profile_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty, owner-scoped stable token | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected profile ref and checksum required when export is enabled, required, or health-visible | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `protocol` | `enum` | `yes` | `otlp` | `no` | `yes` | exactly `otlp` for MVP | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `endpoint_ref` | `030.ActivationControlledArtifactRef` | `conditional:export_enabled=true` | `null` | `yes` | `yes` | opaque endpoint ref only; raw URLs and credentials forbidden | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | private endpoint binding ref required when export enabled | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` |
| `collector_required` | `boolean` | `yes` | `true` | `no` | `yes` | `true` for production export | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected value required before export | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `timeout_ms` | `integer` | `yes` | `2000` | `no` | `yes` | `100..30000` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `retry_policy` | `object` | `yes` | default object above | `no` | `yes` | nested fields bounded by default exporter profile | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized nested object required before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `queue_max_items` | `integer` | `yes` | `2048` | `no` | `yes` | `1..65536` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_EXPORTER_QUEUE_OVERFLOW` |
| `export_batch_max_items` | `integer` | `yes` | `512` | `no` | `yes` | `1..queue_max_items` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `schedule_delay_ms` | `object` | `yes` | default object above | `no` | `yes` | each member `100..300000` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `backpressure_behavior` | `enum` | `yes` | `drop_newest_nonblocking` | `no` | `yes` | exactly `drop_newest_nonblocking` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_EXPORTER_QUEUE_OVERFLOW` | `TELEMETRY_AUTHORITY_VIOLATION` |
| `shutdown_flush_timeout_ms` | `integer` | `yes` | `5000` | `no` | `yes` | `0..30000` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `failure_health_effect` | `enum` | `yes` | `degraded` | `no` | `yes` | `none`, `degraded`, or `blocked`; observability health only | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected mapping refs required when health-visible | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `metric_exemplars_enabled` | `boolean` | `yes` | `false` | `no` | `yes` | exactly `false` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before metric export | `TELEMETRY_ATTRIBUTE_FORBIDDEN` | `TELEMETRY_ATTRIBUTE_FORBIDDEN` |
| `otel_environment_passthrough` | `boolean` | `yes` | `false` | `no` | `yes` | exactly `false` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before startup/export | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty exporter validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover deployment scope | `n/a` | `reject` | `n/a` | `ordered:99` | `yes` | `closed` | `110` | selector context ref and selector checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production export requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required when status affects selection | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex over canonical row bytes after defaults, excluding only `row_checksum` | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |

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
| `exporter_failure_state` | Yes | Closed token: `none`, `export_disabled`, `endpoint_unavailable`, `collector_unreachable`, `queue_overflow`, `timeout`, `redaction_rejection`, `profile_invalid`, `shutdown_flush_timeout`, `permanent_rejection`, `retry_max_elapsed_exhausted`. |
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

The `TelemetryHealthMappingPolicy` input condition vocabulary is total over the exporter runtime condition mapping in `TelemetryExporterProfile`. A missing health mapping row for any condition that is health/API/audit/validation visible fails before rendering with `TELEMETRY_PROFILE_INVALID` or `TELEMETRY_VERSION_MANIFEST_INCOMPLETE`.

| Telemetry condition | Required `health_scope = observability` status | System aggregation may consider it | Forbidden domain effect |
| --- | --- | --- | --- |
| `export_disabled_by_active_profile` | `healthy` when the selected profile disables export; otherwise `blocked` when export is required | yes | none |
| `exporter_endpoint_unavailable` | `degraded` unless active policy maps to `blocked` for observability diagnostics | yes | none |
| `collector_unreachable` | `degraded` unless active policy maps to `blocked` | yes | none |
| `export_attempt_timeout` | `degraded` | yes | none |
| `exporter_queue_overflow` | `degraded` unless active policy maps to `blocked` | yes | none |
| `redaction_rejection` | `degraded` when runtime state can be recorded without recursion; otherwise `blocked` for observability diagnostics | yes | none |
| `invalid_required_exporter_profile` | `blocked` | yes | none |
| `shutdown_flush_timeout` | `degraded` | yes | none |
| `permanent_rejection` | `degraded` unless active policy maps to `blocked` | yes | none |
| `retry_max_elapsed_exhausted` | `degraded` | yes | none |
| `otel_environment_attempted_egress_contained` | `degraded` for observability diagnostics when recorded; startup validation may instead fail the profile before export | yes | none |

System health aggregation remains owned by `110.EndpointOutcomeMatrix`.

### TelemetryDomainClosureHandoff

`140.TelemetryNonAuthorityRule` and `140.TelemetryHealthMappingPolicy` are the owner contracts for telemetry non-authority and observability health mapping. `domain.md` may route to these contracts by exact name only. Domain context-map validation remains blocked until `120-OBSERVABILITY-*` and `120-DOMAIN-CMAP-*` rows pass.

This handoff defines no source authority, identity authority, graph authority, package activation authority, API response schema, audit persistence rule, replay checksum algorithm, or validation harness behavior outside the telemetry contracts owned by this file.

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

The excluded volatile telemetry field set also includes `diagnostic_correlation_ref`, `server_correlation_nonce`, `retry_jitter_sample_ms`, `export_attempt_start_time`, `export_attempt_end_time`, `shutdown_flush_start_time`, `shutdown_flush_end_time`, exporter runtime random source state, and telemetry SDK provider runtime instance ID. These fields may appear only in telemetry diagnostic output classes when permitted by an active `TelemetryReplayExclusionPolicy`, and even then only with `TelemetryRuntimeState` refs and `030.VersionManifest` refs.

### TelemetryReplayExclusionPolicy row schema

`TelemetryReplayExclusionPolicy` is an activation-controlled row family. The default rows below must exist when `080.ReplayEquivalencePolicyOutputClassRow.output_class = telemetry_health_diagnostic` is active. A row affects telemetry replay only; it must not grant domain authority.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `field_class` | Yes | none | One of `trace_id`, `span_id`, `sampled_flag`, `runtime_duration`, `runtime_start_time`, `exporter_queue_state`, `exporter_delivery_result`, `collector_state`, `dropped_telemetry_count`, or `backend_telemetry_id`. |
| `default_replay_effect` | Yes | `excluded_from_domain_replay` | Domain replay must not include the field. |
| `allowed_inclusion_condition` | Yes | `telemetry_health_diagnostic_only` | Inclusion is allowed only for health-visible telemetry diagnostics with `TelemetryRuntimeState` refs and manifest refs. |
| `health_visible_condition` | Yes | `requires_telemetry_runtime_state_ref` | Health-visible fields require a persisted runtime state ref. |
| `forbidden_domain_output_condition` | Yes | `always_for_domain_outputs` | Inclusion in raw, silver, identity, gold, correction, graph, API, export, analysis, validation, or run-lock domain checksums fails. |
| `validation_refs` | Yes | none | Non-empty refs proving trace/span changes do not alter domain replay and forbidden inclusion fails. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults, excluding only `row_checksum`. |

## EmitTelemetry Algorithm

```text
EmitTelemetry(signal, attributes, value, context, profile, runtime_state):
1. Validate that `profile` is present, active, in scope, and names one `TelemetrySignalPolicy`.
2. If telemetry is disabled by the active profile, return no-op and do not mutate domain output.
3. Normalize `signal` to one of `trace`, `metric`, `structured_log`, or `baggage`.
4. If the normalized signal is not permitted for production, fail with `TELEMETRY_SIGNAL_FORBIDDEN` before export.
5. Validate the active trace context. If context is missing and tracing or structured logs are enabled, create runtime diagnostic context only.
6. When the active profile and an API, audit, operator-diagnostic, or validation output context requires an opaque support ref, call `diagnostic_correlation_ref_v1`; otherwise keep `diagnostic_correlation_ref = null`.
7. Reject raw trace ID and raw span ID exposure before export and before API, audit, health, validation, graph, package, or evidence materialization.
8. Ignore OpenTelemetry SDK environment variables, SDK autoconfiguration, default OTLP endpoints, default headers, sampler settings, resource detectors, processors, readers, plugin activation, and config files as behavior sources unless a future active Cadastre policy explicitly imports an exact bounded value. MVP `otel_environment_passthrough` must be `false`.
9. Canonicalize attribute keys and values.
10. Reject any attribute key or value not allowlisted by `TelemetryAttributePolicy`.
11. Run `TelemetryRedactionPolicy` and `110.RedactionPolicy` preflight; reject raw payload bytes, private bindings, credentials, backend IDs, provider-native query text, raw SQL text, raw Cypher text, SQL/Cypher composed text, SQL/Cypher literals, literal-bearing query plans, connection strings, PostgreSQL internal IDs, SQL cursor or prepared statement names, AGE internal IDs, and raw graph property values before export.
12. If `signal = metric`, resolve exactly one `MetricInstrumentRow`, validate instrument type, unit, bucket profile, allowed attributes, cardinality class, per-run cardinality bound, `attribute_set_overrun_behavior`, and `exemplar_policy`.
13. Reject metric exemplars before export. `metric_exemplars_enabled` must be `false`; exemplar trace IDs, span IDs, sampled flags, and exemplar attributes must not be exported.
14. Enforce `TelemetryNonAuthorityRule`; if telemetry attempts to create, modify, authorize, validate, replay, or persist a non-telemetry authoritative record, fail with `TELEMETRY_AUTHORITY_VIOLATION` before the forbidden effect.
15. Materialize `TelemetryExporterProfile` defaults before exporter handoff when export is enabled or required.
16. Enforce exporter queue capacity using `drop_newest_nonblocking`; a full queue must drop the newest telemetry item, update runtime state when non-recursive, and never block domain writes.
17. Submit the telemetry item through `TelemetryExporterProfile` using bounded non-blocking behavior when export is enabled.
18. On endpoint unavailable, timeout, Collector unreachable, queue overflow, redaction rejection, permanent rejection, retry exhaustion, invalid profile, or shutdown flush timeout, update `TelemetryRuntimeState` counters and failure state; do not change the domain result.
19. Map telemetry runtime state to health only through `TelemetryHealthMappingPolicy` when health output is requested.
20. Return the updated `TelemetryRuntimeState` ref when runtime state is health/API/audit/validation visible; otherwise return telemetry no-op status.
```

Forbidden telemetry content errors are blocking for the telemetry item and must occur before export. Export delivery failures are non-blocking for domain output and health-affecting only through policy.

## ValidateTelemetryProfile Algorithm

```text
ValidateTelemetryProfile(profile, artifact_refs, package_set_ref, metric_catalog, exporter_profile, validation_scope):
1. Validate `profile` through `030.ActivationControlledArtifactRef` with owner `140`.
2. Validate profile lifecycle status, activation scope, checksum, package-set ref when package-supplied, and validation refs.
3. Fail if any output-affecting telemetry row family lacks a complete `030.ActivationControlledRowField` table with the exact required columns.
4. Resolve exactly one active `TelemetrySignalPolicy` for each enabled signal.
5. Validate `TraceContextPolicy` when traces or structured logs are enabled.
6. Validate `TelemetryCorrelationPolicy` before any caller-visible correlation ref is emitted.
7. Fail if `otel_environment_passthrough != false`.
8. Fail if `metric_exemplars_enabled != false`.
9. Validate `MetricInstrumentCatalog` and every `MetricInstrumentRow` when metrics are enabled.
10. Validate `TelemetryAttributePolicy` and `TelemetryRedactionPolicy` for every enabled signal.
11. Validate `TelemetryExporterProfile` when export is enabled or required; fail if an exporter profile is required but absent.
12. Materialize every defaulted exporter numeric field before exporter row checksum computation; fail when a required default cannot materialize.
13. Validate exporter timeout, retry, queue, batch, schedule delay, backpressure, shutdown flush, drop behavior, and failure health effect bounds against this file.
14. Validate `TelemetryHealthMappingPolicy` when telemetry can affect `110.OperationalHealthStatus`.
15. Validate `TelemetryReplayExclusionPolicy` when replay, API, audit, validation, or operator-visible diagnostics are in scope.
16. Reject any policy that permits raw payload bytes, private binding leakage, backend ID export, raw trace IDs, raw span IDs, unbounded metric labels, domain authority effects, audit substitution, graph repair, source absence, identity evidence, package activation, watermark advancement, or replay checksum inclusion of forbidden telemetry fields.
17. Require `100.ProductionPackageSetManifest` refs when telemetry artifacts are package-supplied.
18. Require `030.VersionManifest` telemetry policy refs and runtime-state refs when telemetry policy or runtime state affects health, API, audit, validation, or operator-visible diagnostics.
19. Return pass only when every required `120-OBSERVABILITY-*` validation ref is present, non-blocking, checksum-backed, package-set checked when applicable, and manifest-included.
```

When structured-input repository workflow telemetry is enabled, `ValidateTelemetryProfile` must validate every structured-input metric row, allowed attribute key, forbidden attribute pattern, commit-SHA diagnostic rule, branch-name redaction rule, file-path redaction rule, private-route rejection rule, and package/report/audit telemetry handoff before export.

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
| `artifact_refs` | Yes | Canonically sorted refs to instrumentation profiles, signal policies, metric catalogs, exporter profiles, attribute policies, redaction policies, runtime state refs, health mapping policies, replay exclusion policies, validation fixtures, package-set refs, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `telemetry_profile_ref` | No | Required when profile resolution is involved. |
| `signal_token` | No | Required for signal-policy failures. |
| `attribute_path` | No | Required for attribute failures. |
| `exporter_ref` | No | Required for exporter failures. |
| `queue_state_class` | No | Required for queue overflow. |
| `forbidden_data_class` | No | Required for raw payload, private binding, backend ID, or replay-field failures. |
| `runtime_state_ref` | No | Required when telemetry runtime state affects health/API/audit/validation diagnostics. |
| `validation_refs` | Yes | Exact `120` observability fixture refs. |
| `redaction_classes` | Yes | Raw payload bytes, private bindings, credentials, backend IDs, source-native identity values, canonical IDs, hostnames, IPs, usernames, provider-native query text, exporter credentials, and raw queue payloads must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |

## ActivationControlledRowSchemaPrecisionHandoff

The following `140` row families can affect telemetry construction, signal export, attribute retention, metric cardinality, exporter behavior, health mapping, replay exclusion, or telemetry redaction. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `ObservabilityInstrumentationProfile` | output_affecting for telemetry behavior | Closed by `ObservabilityInstrumentationProfile field precision`; production still requires concrete row refs, validation refs, package-set refs when supplied, and manifest inclusion. |
| `TelemetrySignalPolicy` | output_affecting for telemetry emission | Closed by `TelemetrySignalPolicy field precision`; unknown signals, omitted export behavior, and validation-only exceptions are deterministic. |
| `TelemetryCorrelationPolicy` | output_affecting for API/audit-visible diagnostics | Closed by `TelemetryCorrelationPolicy field precision`; non-null refs require `diagnostic_correlation_ref_v1`, private secret validation, and nonce non-persistence. |
| `MetricInstrumentRow` | output_affecting for metrics | Closed by `MetricInstrumentRow` field precision; exemplars are forbidden and cardinality caps materialize before checksum computation. |
| `TelemetryAttributePolicy` | output_affecting for telemetry output | Closed for graph backend diagnostics by `TelemetryActivationControlledRowFieldClosure`; production still requires concrete row refs, package-set refs when supplied, validation refs, and manifest inclusion. |
| `TelemetryRedactionPolicy` | output_affecting for telemetry output | Closed for graph backend diagnostics by `TelemetryActivationControlledRowFieldClosure`; forbidden backend values reject before export. |
| `TelemetryExporterProfile` | output_affecting for export and health | Closed by `TelemetryExporterProfile field precision`; defaults, null/omit behavior, retry, queue, shutdown, OTel environment containment, and exemplar prohibition are deterministic. |
| `TelemetryHealthMappingPolicy` | health_affecting | Closed for graph backend diagnostics by `TelemetryActivationControlledRowFieldClosure` and by the total condition mapping in this file; health effects are limited to `healthy`, `degraded`, `blocked`, or `error` and have no domain mutation effect. |
| `TelemetryReplayExclusionPolicy` | replay_affecting | Closed by `TelemetryReplayExclusionPolicy row schema`; activation remains blocked when validation refs, package-set refs, or manifest refs are missing or TODO-bearing. |

`telemetry_signals`, `allowed_attribute_keys`, enabled signal sets, and validation refs must use `canonical_set` semantics with duplicate rejection unless an owner row declares `ordered_sequence`. `retry_policy` must be a typed nested object or named owner union with bounds for maximum attempts, backoff, queue behavior, and retryable exporter errors. Telemetry rows must remain diagnostic and must not grant source, identity, graph, package, audit, replay, or watermark authority.

Telemetry validation, export, health mapping, or replay exclusion must fail before telemetry-visible output when any selected `140` row family remains `TODO:`, uses an unbounded label, exposes raw/private values, uses a bare string row ref, or omits selected row refs from `030.VersionManifest`.

### Telemetry activation-controlled row-family closure

The tables below close the remaining output-affecting telemetry row families. They use the exact `030.ActivationControlledRowField` column order and are authoritative for defaults, null behavior, omission behavior, bounds, checksum participation, manifest requirements, and owner errors.

#### ObservabilityInstrumentationProfile field precision

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `profile_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty, owner-scoped stable token | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected profile ref and checksum required when telemetry is enabled or visible | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `telemetry_standard` | `enum` | `yes` | `opentelemetry` | `no` | `yes` | exactly `opentelemetry` for MVP | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `telemetry_protocol` | `enum` | `yes` | `otlp` | `no` | `yes` | exactly `otlp` for MVP when export is enabled | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | materialized before checksum | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `telemetry_signals` | `array<enum>` | `yes` | `[trace, metric, structured_log]` | `no` | `yes` | members from `trace`, `metric`, `structured_log`, `baggage`; baggage export forbidden by default | `canonical_set` | `reject` | `value` | `no` | `yes` | `closed` | `110` | selected signal refs required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `telemetry_delivery_mode` | `enum` | `yes` | `bounded_best_effort_nonblocking` | `no` | `yes` | exactly `bounded_best_effort_nonblocking` for MVP | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_AUTHORITY_VIOLATION` |
| `telemetry_export_failure_domain_effect` | `enum` | `yes` | `none` | `no` | `yes` | exactly `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required in manifest for visible diagnostics | `TELEMETRY_AUTHORITY_VIOLATION` | `TELEMETRY_AUTHORITY_VIOLATION` |
| `telemetry_export_failure_health_effect` | `enum` | `yes` | `degraded` | `no` | `yes` | `none`, `degraded`, or `blocked` for observability health only | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected health mapping refs required when visible | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `telemetry_trace_id_replay_checksum_effect` | `enum` | `yes` | `excluded` | `no` | `yes` | exactly `excluded` for domain replay | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | replay exclusion refs required when replay is in scope | `TELEMETRY_REPLAY_FIELD_FORBIDDEN` | `TELEMETRY_REPLAY_FIELD_FORBIDDEN` |
| `telemetry_span_id_replay_checksum_effect` | `enum` | `yes` | `excluded` | `no` | `yes` | exactly `excluded` for domain replay | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | replay exclusion refs required when replay is in scope | `TELEMETRY_REPLAY_FIELD_FORBIDDEN` | `TELEMETRY_REPLAY_FIELD_FORBIDDEN` |
| `raw_payload_in_telemetry` | `enum` | `yes` | `forbidden` | `no` | `yes` | exactly `forbidden` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_RAW_PAYLOAD_FORBIDDEN` | `TELEMETRY_RAW_PAYLOAD_FORBIDDEN` |
| `private_source_binding_in_telemetry` | `enum` | `yes` | `forbidden` | `no` | `yes` | exactly `forbidden` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN` |
| `backend_internal_id_in_telemetry` | `enum` | `yes` | `forbidden` | `no` | `yes` | exactly `forbidden` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before export | `TELEMETRY_BACKEND_ID_FORBIDDEN` | `TELEMETRY_BACKEND_ID_FORBIDDEN` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty observability profile validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover stage and execution scope | `n/a` | `reject` | `n/a` | `ordered:99` | `yes` | `closed` | `110` | selector context ref and selector checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required when status affects selection | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex over canonical row bytes after defaults, excluding only `row_checksum` | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |

#### TelemetrySignalPolicy field precision

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty, owner-scoped stable token | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected row ref and checksum required when signal may emit | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `signal_token` | `enum` | `yes` | `none` | `no` | `no` | `trace`, `metric`, `structured_log`, or `baggage` | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | selected signal row checksum required | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `stage_class` | `enum` | `yes` | `none` | `no` | `no` | active `030` stage class token or `api`/`operator` diagnostic context | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `production_status` | `enum` | `yes` | `allowed` for listed production signal rows | `no` | `yes` | `allowed`, `forbidden`, or `validation_only_non_production` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required before emission | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `default_export_behavior` | `enum` | `yes` | `export_after_redaction` for trace/metric/structured_log; `propagation_only` for baggage | `no` | `yes` | `export_after_redaction`, `local_only`, `propagation_only`, `forbidden` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selected row checksum required | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_PROFILE_INVALID` |
| `validation_only_non_production` | `boolean` | `yes` | `false` | `no` | `yes` | `true` permitted only outside production execution mode | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | value required when custom signal is validation-only | `TELEMETRY_SIGNAL_FORBIDDEN` | `TELEMETRY_SIGNAL_FORBIDDEN` |
| `non_authority_rule_ref` | `string` | `yes` | `TelemetryNonAuthorityRule` | `no` | `yes` | exact exported non-authority rule name | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | non-authority refs required when signal visible | `TELEMETRY_AUTHORITY_VIOLATION` | `TELEMETRY_AUTHORITY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty signal validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover stage and execution scope | `n/a` | `reject` | `n/a` | `ordered:99` | `yes` | `closed` | `110` | selector context ref and selector checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required when status affects selection | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex over canonical row bytes after defaults, excluding only `row_checksum` | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | selected row checksum required | `TELEMETRY_PROFILE_INVALID` | `TELEMETRY_PROFILE_INVALID` |

## Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `140-SCOPE-TELEMETRY-AC-001` | Exact telemetry export scope, scope mismatch health degradation or blocking as policy declares, private exporter scope leak, and no domain mutation from telemetry scope failures fixtures pass. |
| `140-SCOPE-TELEMETRY-AC-002` | Telemetry scope matching affects only telemetry emission, export, health mapping, and telemetry replay exclusion. |
| `140-PROFILE-DEFAULTS-AC-001` | The default instrumentation profile materializes OpenTelemetry-compatible, OTLP-compatible traces, metrics, structured logs, bounded best-effort nonblocking delivery, no domain effect from export failure, degraded health effect from export failure, replay exclusion for trace/span IDs, and forbidden raw payload, private binding, and backend ID telemetry. |
| `140-SIGNAL-POLICY-AC-001` | `trace`, `metric`, `structured_log`, and `baggage` are the only production signal tokens; unknown or disabled signals fail with `TELEMETRY_SIGNAL_FORBIDDEN` before export. |
| `140-TRACE-CONTEXT-AC-001` | Trace context propagation, parent-child spans, async handoff, sampled/unsampled behavior, baggage restrictions, and context loss never alter stage order, output IDs, checksums, replay equivalence, package activation, or domain output. |
| `140-ATTRIBUTE-REDACTION-AC-001` | Raw payload bytes, private bindings, credentials, backend IDs, source-native IDs, canonical IDs, hostnames, IPs, user names, tenant inventories, route names, provider-native query text, raw SQL, raw Cypher, SQL/Cypher text, SQL/Cypher literals, literal-bearing query plans, connection strings, PostgreSQL internal IDs, SQL cursor or prepared statement names, AGE internal IDs, and raw graph property values are rejected or redacted before export according to `TelemetryAttributePolicy`, `TelemetryRedactionPolicy`, and `110.RedactionPolicy`. |
| `140-CARDINALITY-AC-001` | Every emitted metric resolves exactly one `MetricInstrumentRow`; unbounded attributes and cardinality overrun fail with `TELEMETRY_CARDINALITY_VIOLATION` before export. |
| `140-EXPORTER-AC-001` | Exporter outage, Collector outage, timeout, and queue overflow update `TelemetryRuntimeState`, may degrade observability health through policy, and do not mutate authoritative domain records. |
| `140-HEALTH-AC-001` | Telemetry runtime state affects `110.OperationalHealthStatus` only through `TelemetryHealthMappingPolicy` and never repairs, invalidates, or mutates domain outputs. |
| `140-REPLAY-AC-001` | Trace IDs, span IDs, sampled flags, runtime duration, exporter queue state, exporter result, Collector state, dropped counts, and backend telemetry IDs are excluded from authoritative domain replay checksums by default. |
| `140-NONAUTH-AC-001` | Telemetry cannot create, modify, authorize, validate, retract, expire, merge, split, project, replay, or persist non-telemetry authoritative Cadastre records. |
| `140-VERSION-MANIFEST-AC-001` | Health, API, audit, validation, or operator-visible diagnostics that depend on telemetry policy or runtime state include required telemetry policy refs and runtime state refs in `030.VersionManifest` or fail with `TELEMETRY_VERSION_MANIFEST_INCOMPLETE`. |
| `140-PACKAGE-AC-001` | Package-supplied telemetry profiles, metric catalogs, redaction policies, exporter profiles, health policies, and replay exclusion policies fail closed unless `100.ProductionPackageSetManifest`, package type policy, validation refs, and artifact checksums pass. |
| `140-RUNLOCK-TELEMETRY-AC-001` | `val-140-runlock-telemetry-cardinality` proves run-lock acquisition, heartbeat, recovery, loss, and lease-remaining metrics use only bounded low-cardinality attributes and reject or redact raw lock keys, raw fencing tokens, raw idempotency keys, source-native IDs, canonical IDs, private bindings, and backend IDs before export. |
| `140-RUNLOCK-NONAUTH-AC-001` | `val-140-runlock-telemetry-nonauthority` proves dropping all run-lock telemetry does not change lock outcome, domain output, replay checksum, package activation, graph apply, table maintenance, watermark state, or audit evidence substitution. |
| `140-CORRELATION-AC-001` | `diagnostic_correlation_ref_v1` fixed-nonce validation produces the exact expected `dcr1-*` output for the selected policy row checksum, activation scope checksum, secret, and nonce. |
| `140-CORRELATION-AC-002` | Missing, inactive, invalid, expired, checksum-mismatched, unauthorized, or omitted correlation secret returns `diagnostic_correlation_ref = null` and never falls back to raw trace/span IDs. |
| `140-EXPORTER-DEFAULTS-AC-001` | Exporter timeout, retry, queue, batch, schedule delay, shutdown, health effect, exemplar, and OTel environment defaults materialize before checksum computation and reject out-of-bounds values. |
| `140-EXPORTER-RETRY-AC-001` | Retry attempts follow the full-jitter envelope, fixtures may inject fixed jitter samples, production configuration cannot inject deterministic jitter, and no retry begins after shutdown. |
| `140-OTEL-ENV-AC-001` | OTel SDK environment variables and autoconfiguration cannot enable endpoint, header, sampler, resource, reader, processor, plugin, exemplar, or config-file behavior. |
| `140-EXEMPLAR-AC-001` | Metric exemplars and exemplar trace/span data are rejected before export. |
| `140-ROW-FIELD-PRECISION-AC-001` | Every output-affecting telemetry row family has a complete `030.ActivationControlledRowField` table and row-set validation contract. |

### Structured input telemetry acceptance criteria

| ID | Criterion |
| --- | --- |
| `140-STRUCTURED-INPUT-TELEMETRY-AC-001` | Structured-input telemetry cannot affect validation, package activation, audit persistence, replay equivalence, source authority, graph output, or domain output. |
| `140-STRUCTURED-INPUT-REDACTION-AC-001` | Private repository values, raw file paths, raw schema bytes, raw fixture bytes, raw structured input bytes, credentials, private routes, and tenant inventories are rejected or redacted before telemetry export. |
| `140-STRUCTURED-INPUT-CARDINALITY-AC-001` | File path, branch-name, commit-message, repository URL, source-native identifier, and private-route cardinality cannot become metric labels. |
| `140-STRUCTURED-INPUT-TOKEN-CLOSURE-AC-001` | Template validation, producer CI validation, tool invocation, publication import, candidate sync, repository group validation, package release handoff, and package-set candidate staging use only closed operation tokens. |
| `140-STRUCTURED-INPUT-PUBLICATION-NONAUTH-AC-001` | Publication import telemetry does not prove package release candidacy or package activation. |
| `140-STRUCTURED-INPUT-SYNC-NONAUTH-AC-001` | Candidate sync telemetry does not mutate active state, prove validation, or authorize rollback. |

## Definition of Done

- Every exported telemetry contract has one owner in `000.Owner Contract Registry` and one volatility classification.
- `diagnostic_correlation_ref_v1` is fully specified and fixed-nonce validation rows pass.
- Exporter timeout, retry, queue, batch, schedule delay, shutdown, failure-health, exemplar, and OTel environment defaults are fully specified and materialized before checksums.
- OTel environment passthrough is exactly disabled for MVP.
- Metric exemplars are exactly disabled for MVP.
- Every output-affecting telemetry row family has a complete `030.ActivationControlledRowField` table.
- `EmitTelemetry` and `ValidateTelemetryProfile` reject forbidden telemetry before export.
- Telemetry-enabled and telemetry-disabled runs produce byte-identical authoritative domain outputs for the same inputs.
- Telemetry exporter failure degrades observability health only through policy and never mutates domain records.
- `120-OBSERVABILITY-*` validation rows cover correlation, exporter defaults, retry, queue overflow, redaction rejection, cardinality, exemplar rejection, OTel environment containment, health mapping, replay, audit linkage, version manifest completeness, package activation, and enabled/disabled domain parity.

## Open Questions

No production-blocking observability open questions remain in this file. Supporting-material fixture bytes, private diagnostic-correlation secret bindings, private exporter endpoint bindings, Collector route bindings, dashboards, alerts, runbooks, and any OpenTelemetry source snapshot or SDK package matrix remain outside this core file until registered by repository-relative path and accepted through `120` validation.
