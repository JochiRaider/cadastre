---
doc_id: CADASTRE-NLSPEC-120
title: Validation, Fixtures, and Acceptance
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define fixtures, validation matrices, golden corpus, shadow execution, replay validation, required negative tests, and binary acceptance reports for the NLSpec set.

## Explicit Non-Scope

- Owning domain behavior.
- New behavior not exported by a domain spec.
- Implementation-specific test harness mechanisms unless observable validation output is affected.

## Imports

- `All active domain specs`

- `All exported 040 core record schemas`
- `CoreRecordValidationAlgorithm`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ScopeSelectorErrorCodeSet`

## Exports

- `ValidationMatrix`
- `LakehouseFeedFixture`
- `ValidationScenario`
- `EventSequenceValidationCorpus`
- `GoldenCorpus`
- `ShadowExecutionReport`
- `ReplayValidationReport`
- `AcceptanceReport`
- `ValidateSpecSet`
- `RunValidationMatrix`
- `LifecycleValidationMatrix`
- `ValidationAcceptanceLifecycleMachine`

## Validation Ownership Rule

This spec verifies behavior owned by domain specs. It must not create new behavioral requirements unless the requirement is a validation interface, fixture record, acceptance report, or promotion gate.

## Validation Artifact Model

| Artifact | Required purpose |
| --- | --- |
| `ValidationMatrix` | Rows mapping contract, scenario, fixture, expected output, expected error/no-op, owner spec, and pass/fail evidence. |
| `LakehouseFeedFixture` | Redacted replayable feed rows, objects, manifests, supplier metadata, and expected read/import outcomes. |
| `ValidationScenario` | Validation-only scenario with given records, expected outputs, expected diagnostics, no-op assertions, and mutation prohibition. |
| `EventSequenceValidationCorpus` | Ordered event sequences for temporal, correction, replay, watermark, CDC, graph apply, rebuild, lifecycle transition, run-lock acquisition, run-lock heartbeat, stale recovery, idempotency, no-op, rollback, quarantine, and validation acceptance behavior. |
| `GoldenCorpus` | Canonical set of input fixtures and expected outputs used for regression and promotion. |
| `AcceptanceReport` | Immutable report that records criteria pass/fail status, checksums, owner, environment, and timestamp. |

## Required Negative Test Classes

Every active domain spec must include at least one negative validation case for each forbidden authority boundary it declares. Required classes include:

| Boundary | Required negative case |
| --- | --- |
| Direct source calls | Production package attempts enterprise source API call and fails before output. |
| Feed completeness | Missing feed row attempts absence and fails. |
| Feed profile closure | Missing feed profile field, missing source-dataset catalog row, ambiguous source-dataset catalog row, blocked source-dataset catalog row, missing category closure row, unresolved profile branch, invalid empty-scope authorization, or missing subset profile fails before absence-sensitive effects. |
| Source authority | Missing exact authority row, ambiguous authority row, missing source-dataset catalog row, ambiguous source-dataset catalog row, checksum-mismatched source-dataset catalog row, missing source-specific coverage row, missing staleness row, missing control-result mapping row, weak-signal combination, or checksum-mismatched closure row attempts output and fails or no-ops with no forbidden mutation. |
| Coverage-domain token closure | Alias/display values, unknown tokens, duplicate tokens, unsupported feed-category/token pairs, omitted required token fields, empty arrays for absence-sensitive effects, and inactive `reachability` usage fail before source authority, absence, cleanup, graph expiry, retraction, watermark, package activation, API output, or validation acceptance. |
| Completeness effect gate | Missing completeness profile row, missing upstream evidence, omitted allowed effect, weak-signal combination, or completeness-blocked watermark fails or no-ops with no absence-sensitive effect. |
| Identity | Weak-only create/attach/merge attempts, selector-only attempts, source-native-merge-history-only attempts, candidate overflow auto-merge attempts, reviewer override of hard blocker, missing explanation, missing resolver row, ambiguous resolver row, and package-supplied weak-default override fail before identity mutation. |
| Graph | Backend internal ID appears in response or selector and fails; active MVP edge set is exactly `observed_connection`; missing or ambiguous `FlowRoleEvidence` emits no edge; OCSF endpoint order cannot determine direction; unresolved endpoint identity emits no edge; string endpoint identity does not project; identifier and source-asset refs require resolver handoff; invalid gold subject/object kind produces no mutation; generic external graph payload is not pathfinding; theoretical reachability and boolean reachability properties are prohibited; query candidate limits and page tokens fail closed; graph backend omitted/defaulted behavior, missing provider profile fields, unsupported provider capability, unsafe storage mode, implicit schema creation, stale schema fingerprint, stale mixed index, full-scan Gremlin translation, missing package gate, backend-generated ID leakage, raw-write bypass, and provider-specific query bypass fail closed or no-op with no forbidden mutation. |
| Run lock | Lease defaults, timing bounds, all-or-nothing acquisition, active conflict, heartbeat refresh, heartbeat timeout, lock loss before commit, stale recovery success, stale recovery race, old-holder fencing, idempotent retry, idempotency conflict, commit guard missing, manifest omission, watermark block, graph apply resume after lock loss, package activation lock loss, destructive maintenance lock loss, lock signal non-authority, and lock telemetry non-authority fixtures fail or pass with exact mutation prohibition. |
| Package | Closed enum success, duplicate token rejection, generic or legacy label rejection, unknown package type, missing package-type policy, inactive package-type policy, scope-mismatched package-type policy, ambiguous package-type policy, policy-row manifest omission, policy-bundle substitution, required release/package-set manifest field omission, unsupported repository form, unauthorized signer, missing transparency evidence, missing attestation, missing SBOM, dependency live resolution, compatibility failure, missing deprecation policy row, deprecation expiry, failed post-activation health gate, mutable rollback target, rollback compatibility failure, quarantine target block, emergency trust bypass, package error-registry parity failure, and package `VersionManifest` omission fail or no-op with no forbidden mutation. |
| Mapping | Missing mapping row, ambiguous mapping row, missing activity discriminator, unknown enum, forbidden OCSF field, IPAM/DHCP split, and undeclared source extension field fail before silver output. |
| Temporal | Missing temporal policy attempts current-time fallback and fails. |
| Analysis | Analysis finding, metric, risk acceptance, threat-intel enrichment, lineage facet, registry governance, registry custom property, registry classification, or derived-edge rule attempts forbidden authority or mutation and fails before the forbidden effect. |
| Observability | Telemetry span, metric, structured log, baggage, exporter success, exporter failure, Collector state, sampling decision, dropped telemetry count, or dashboard state attempts to affect facts, identity, source authority, source completeness, coverage, graph deltas, graph apply, package activation, replay checksum, watermark, audit persistence, or domain output and fails or no-ops with no forbidden mutation. |
| Structured input repository | Branch, tag, pull request, repository URL, or hook success used as activation target; stale branch-tip validation; exact tree mismatch; invalid path; private binding leak; unmaterialized Git snapshot activation; package release missing materialization refs; missing `VersionManifest` refs; rollback to branch or tag; and hook success as activation evidence fail before forbidden mutation. |
| Reachability | MVP graph profile attempts `has_theoretical_reachability` and fails. |
| Scope selector closure | Selector normalization, exact match, subset allowed, subset disallowed, duplicate dimension, duplicate value, unknown field, unsupported dimension, under-scoped request, private leak, ambiguity, no row-order tiebreak, owner error mapping, redaction, and manifest inclusion are covered for every active scoped owner family. |
| Activation-controlled row schema precision | Production-affecting activation-controlled row families with prose-only schemas, missing `030.ActivationControlledRowField` tables, missing null/omit/default/bounds columns, missing array semantics, missing duplicate policy, bare string row refs, checksum mismatch, extension redefinition, manifest omission, package-set omission, or owner error omission fail before promotion or owner output. |

## Required Volatility Boundary Test Classes

Validation must prove that volatile material cannot redefine stable behavior and cannot affect production output without activation refs.

| Volatility boundary | Required negative case | Expected result |
| --- | --- | --- |
| core/artifact boundary | Mapping bundle attempts to define a new core field. | Fails before persistence. |
| artifact activation | Inactive resolver profile, missing candidate generation profile, checksum-mismatched evidence class row set, out-of-scope split policy, omitted resolver explanation policy, or missing `VersionManifest` resolver artifact refs are referenced. | Fails before identity output or replay. |
| artifact checksum | Source-authority row checksum mismatch. | Fails before gold derivation. |
| artifact omission | Graph projection profile or OCSF mapping row set omitted from `VersionManifest`. | Fails before dependent output. |
| external schema non-authority | OCSF profile attempts to treat observables, status, severity, confidence, field absence, or endpoint order as authority. | Fails before authority or graph effect. |
| package/artifact mismatch | Active package set includes artifact checksum that does not match the artifact ref. | Package activation fails and current active set remains. |
| appendix/reference non-authority | Research or ADR text is referenced as runtime authority. | Spec-set validation fails. |
| stable core conflict | Activation artifact attempts to redefine identity, temporal, omission, or graph authority. | Fails with volatility-boundary error before production output. |
| runtime state substitution | `GraphRebuildManifest`, `LakehouseCommitRef`, lineage facet, or validation report is used as source truth. | Fails before authority effect. |
| graph provider default boundary | Research report, backend package metadata, or provider runtime default attempts to define Cadastre graph behavior without active `090` profile and `100` package-set refs. | Fails before graph apply, query serving, rebuild promotion, or production output. |
| structured input repository stable-boundary | Repository-authored artifact attempts to redefine stable core behavior, authority classes, core fields, temporal axes, identity rules, graph authority, package activation target, or API state labels. | Fails before activation, materialization, or production output. |
| repository snapshot/package digest substitution | Repository snapshot, tree hash, branch, tag, PR ref, validation report, or hook result attempts to substitute for package artifact digest or release checksum. | Fails before package release handoff or package-set activation. |
| repository validation/owner row substitution | Repository validation report or Git snapshot attempts to substitute for active owner row set, source evidence, source completeness, graph rebuild evidence, validation acceptance, or production approval. | Fails before authority effect or promotion. |

| telemetry runtime-state substitution | `TelemetryRuntimeState`, exporter success, Collector state, or dashboard state is used as source truth, identity evidence, graph truth, audit persistence, package activation, or replay input. | Fails before authority effect. |
| telemetry artifact activation | Missing, inactive, checksum-mismatched, out-of-scope, or package-set-mismatched telemetry profile is referenced by health/API/audit/validation output. | Fails before telemetry-visible output or emits owner health error. |
| telemetry metric cardinality | Metric catalog uses unbounded labels such as source-native IDs, canonical IDs, hostnames, IPs, usernames, backend IDs, or private route names. | Fails before telemetry profile activation or export. |

## Acceptance Aggregation

`RunValidationMatrix` must produce a deterministic `AcceptanceReport` containing row ID, owner spec, fixture checksum, input checksum, expected output checksum, actual output checksum, result, failure code, and version manifest ref.

`AcceptanceReport.result = pass` is forbidden when any non-deferred define-once, Section 25, owner-export, duplicate-owner, runtime-restatement, or owner-contradiction validation row is `blocked`, `not_run`, `fail`, stale, checksum-mismatched, or contains a required `TODO` fixture checksum, expected output checksum, expected error, mutation-prohibition proof, activation ref, or manifest ref.

### ScopeSelectorValidationMatrix

This matrix verifies `030.ScopeSelector` behavior and every owner import that uses scoped rows. A row may pass only when fixture checksum, expected output checksum, owner error mapping, redaction proof, mutation-prohibition proof, and `030.VersionManifest` requirements are concrete and non-`TODO`.

| validation_row_id | owner_spec | scenario | expected output or error | required fixture coverage |
| --- | --- | --- | --- | --- |
| `val-030-scope-normalization-canonical` | `030` | Same selector dimensions and values in different input orders. | byte-identical normalized selector checksum | canonical bytes and checksum. |
| `val-030-scope-exact-equality` | `030` | `ScopeSelectorEquals` on equivalent normalized selectors. | true | exact equality. |
| `val-030-scope-eq-coverage` | `030` | `eq` row selector covers identical `eq` request. | covered | exact coverage. |
| `val-030-scope-in-coverage` | `030` | `in` row selector covers request value subset. | covered | multi-value coverage. |
| `val-030-scope-subset-disallowed` | `030` | Row omits request dimension not listed in `subset_allowed_dimension_keys`. | `SCOPE_SUBSET_NOT_ALLOWED` | no mutation. |
| `val-030-scope-subset-allowed` | `030` | Row omits request dimension listed in `subset_allowed_dimension_keys`. | covered | selected context ref and selector checksum. |
| `val-030-scope-duplicate-dimension` | `030` | Duplicate dimension key. | `SCOPE_SELECTOR_DUPLICATE_DIMENSION` | no selected row. |
| `val-030-scope-duplicate-value` | `030` | Duplicate value after normalization. | `SCOPE_SELECTOR_DUPLICATE_VALUE` | no selected row. |
| `val-030-scope-unknown-field` | `030` | Unknown selector or dimension field. | `SCOPE_SELECTOR_UNKNOWN_FIELD` | no checksum. |
| `val-030-scope-unsupported-dimension` | `030` | Dimension key not declared by context. | `SCOPE_SELECTOR_UNSUPPORTED_DIMENSION` | owner mapping. |
| `val-030-scope-under-scoped-request` | `030` | Request omits a required context key. | `SCOPE_REQUEST_UNDER_SCOPED` | no owner mutation. |
| `val-030-scope-private-binding-leak` | `010`, `030`, `110` | Raw private selector value appears in public selector bytes. | `PRIVATE_BINDING_LEAK` or owner-specific leak code | redaction proof. |
| `val-030-scope-ambiguity-rejection` | `030` | Two maximal scoped rows remain. | `SCOPE_SELECTOR_AMBIGUOUS` | no selected row. |
| `val-030-scope-no-row-order-tiebreak` | `030` | Reversed row order yields same ambiguity output. | byte-identical ambiguity diagnostic | no row-order selection. |
| `val-030-scope-owner-error-mapping` | all scoped owners | Owner maps shared selector error to owner code. | owner-specific error | registry row and fixture ref. |
| `val-030-scope-redaction` | `110` | Selector error response contains private raw value in source input. | redacted output only | checksum, dimension key, redacted ref. |
| `val-030-scope-version-manifest` | all scoped owners | Scoped row selection affects output. | `VersionManifest` includes row ref, row checksum, context ref, selector checksum, activation artifact ref | manifest inclusion. |

`AcceptanceReport.result = pass` is forbidden when any active scoped owner family lacks selector normalization, exact-match, subset-allowed, subset-disallowed, duplicate, private-leak, ambiguity, owner-error-mapping, redaction, and manifest-inclusion fixtures.

### ActivationCatalogClosureValidationMatrix

This matrix aggregates closure-pack validation rows. A row may pass only when the fixture checksum, input artifact refs, expected output or expected error checksum, mutation-prohibition proof, package-set refs when package-supplied, and `030.VersionManifest` requirements are concrete and non-`TODO`.

| Validation family | Required owner |
| --- | --- |
| `120-FEED-CLOSURE-*` | `020` |
| `120-SOURCE-DATASET-CATALOG-*` | `020`, with authority, mapping, graph, package, API, and validation handoffs in `050`, `060`, `080`, `090`, `100`, `110`, and `130` |
| `120-GOLD-PREDICATE-CATALOG-*` | `080`, with shape, authority, graph, package, manifest, error, and validation handoffs in `040`, `060`, `070`, `090`, `100`, `110`, and `030` |
| `120-OCSF-MAP-*` | `050` |
| `120-SOURCE-EXT-*` | `050` |
| `120-OCSF-NONAUTH-*` | `050`, `060`, `090` |
| `120-OCSF-DIRECTION-*` | `050`, `090` |
| `120-SOURCE-CLOSURE-*` | `060` |
| `120-COVERAGE-DOMAIN-*` | `060`, with feed-category mapping checks in `020` |
| `120-IDENTITY-CLOSURE-*` | `070` |
| `120-GRAPH-PROFILE-CLOSURE-*` | `090` |
| `120-GRAPH-BACKEND-*` | `090`, `100` |
| `120-PACKAGE-*` | `100` |
| `120-VERSION-MANIFEST-*` | `030` |
| `120-ERROR-REGISTRY-*` | `110` |
| `120-PRIVATE-BINDING-*` | `010`, `110` |

Every activation catalog closure validation row must use this interface:

| Field | Required behavior |
| --- | --- |
| `validation_row_id` | Stable ID; the four required `000` rows are `val-000-activation-catalog-closure-pack-complete`, `val-000-activation-catalog-closure-pack-member-manifested`, `val-000-activation-catalog-closure-pack-block-row-no-mutation`, and `val-000-activation-catalog-closure-pack-todo-blocks-promotion`. |
| `owner_spec` | Owner that defines the behavior under test. |
| `fixture_id` | Stable fixture ID. |
| `fixture_checksum` | Concrete SHA-256; `TODO` means `blocked`. |
| `input_artifact_refs` | Every row-set, deterministic block row, validation row, package release, package-set, and manifest ref required for the scenario. |
| `expected_output_class` | `closed_active`, `closed_deterministically_blocked`, owner error, no-op, or validation failure. |
| `expected_error_or_no_op` | Required for negative and deterministic-block cases. |
| `expected_output_checksum` | Concrete SHA-256 for outputs or expected error bytes; `TODO` means `blocked`. |
| `mutation_prohibition` | Required for deterministic block, private-binding leak, validation-blocked, and no-op scenarios. |
| `version_manifest_requirement` | Required for every output-affecting ref and runtime state ref. |
| `package_set_requirement` | Required when any row or catalog is package-supplied. |
| `acceptance_criterion` | Exact owner acceptance criterion ID. |
| `blocking_status` | `pass`, `fail`, `blocked`, or `not_run`; `TODO` checksums force `blocked`. |

#### ClosurePackAcceptanceCriteria

| ID | Criterion |
| --- | --- |
| `120-SOURCE-DATASET-CATALOG-AC-001` | Every selected `source_dataset` resolves to one active catalog row or deterministic source-dataset block row. |
| `120-SOURCE-DATASET-CATALOG-AC-002` | Private values in public source-dataset rows fail before activation. |
| `120-SOURCE-DATASET-CATALOG-AC-003` | Selected source-dataset rows appear in `VersionManifest.included_refs`. |
| `120-SOURCE-EFFECT-CLOSURE-AC-001` | Every selected category/dataset/fact/predicate/scope/effect tuple resolves to closure or deterministic block. |
| `120-SOURCE-EFFECT-CLOSURE-AC-002` | Weak signals remain diagnostic-only unless an exact active row grants the exact effect. |
| `120-SOURCE-EFFECT-CLOSURE-AC-003` | Deterministic block rows prove no fact, correction, graph delta, cleanup, watermark, pass/fail, or no-change proof is emitted. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-001` | Every selected scope has active row-set refs or exact deterministic block refs. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-002` | Every output-affecting member ref appears in `030.VersionManifest.included_refs`. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-003` | Every MVP observation family has OCSF row coverage, exact `cadastre_only`, or deterministic block. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-004` | Every absence-sensitive effect has source-authority closure or exact deterministic block. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-005` | Resolver catalog completeness covers every required resolver artifact and weak-evidence default. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-006` | Package policy and deprecation rows cover every confirmed package type in scope. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-007` | Graph MVP edge set is exactly `observed_connection`; every other edge is inactive or deterministically blocked. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-008` | Graph backend blockers are resolved or produce deterministic no-op/no-mutation behavior. |
| `120-ACTIVATION-CATALOG-CLOSURE-AC-009` | Research reports, ADRs, repository snapshots, backend defaults, validation reports, and package labels are never runtime authority. |
| `120-GOLD-PREDICATE-CATALOG-AC-001` | Every MVP fact type and predicate resolves to exactly one active predicate row or exact deterministic block row, and all row refs, checksums, validation refs, structured schema refs, and package-set refs appear in `030.VersionManifest` when output-affecting. |

`AcceptanceReport.result = pass` is forbidden when any activation-catalog closure validation row is `blocked`, `not_run`, `fail`, stale, checksum-mismatched, missing a required owner row, contains a `TODO` checksum, lacks package-set refs when package-supplied, lacks mutation-prohibition proof, or omits required `030.VersionManifest` refs. It is also forbidden when any activation-controlled row-schema precision row is blocked, not run, failed, stale, checksum-mismatched, package-set-mismatched, manifest-incomplete, owner-error-incomplete, or contains a required `TODO` fixture checksum, expected output checksum, expected error, row checksum, row-set checksum, or mutation-prohibition proof.

### ActivationControlledRowSchemaValidationMatrix

### SourceDatasetCatalogValidationMatrix

A source-dataset catalog validation row may pass only when the fixture checksum, input artifact refs, expected output or expected error checksum, mutation-prohibition proof, package-set refs when package-supplied, and `030.VersionManifest` requirements are concrete and non-`TODO`.

| Fixture family | Required coverage | Expected result |
| --- | --- | --- |
| `source-dataset-valid-*` | Active row resolves for source category, feed category, source dataset, scope, and lifecycle. | Selected row ref/checksum emitted. |
| `source-dataset-missing-*` | Referenced dataset has no active row. | `SOURCE_DATASET_CATALOG_ROW_MISSING`; no output. |
| `source-dataset-ambiguous-*` | Two equally specific rows match. | `SOURCE_DATASET_CATALOG_ROW_AMBIGUOUS`; no output. |
| `source-dataset-blocked-*` | Dataset intentionally blocked. | Deterministic no-op/block; no mutation. |
| `source-dataset-private-leak-*` | Public row includes private route, tenant, scanner site, account, host list, or raw fixture bytes. | `SOURCE_DATASET_CATALOG_PRIVATE_BINDING_LEAK`. |
| `source-dataset-unsupported-category-*` | Dataset/category pair unsupported. | `SOURCE_DATASET_UNSUPPORTED_FOR_FEED_CATEGORY`; no effect. |
| `source-dataset-manifest-omission-*` | Selected row omitted from `VersionManifest`. | `VERSION_MANIFEST_INCOMPLETE` or owner manifest error. |
| `source-dataset-package-set-omission-*` | Package-supplied row lacks package-set ref. | Package-set owner error; no activation. |

### GoldFactPredicateCatalogValidationMatrix

A gold predicate-catalog validation row may pass only when fixture checksum, input artifact refs, expected output or expected error checksum, mutation-prohibition proof, package-set refs when package-supplied, and `030.VersionManifest` requirements are concrete and non-`TODO`.

| validation_row_id | owner_spec | scenario | expected output or error | required fixture coverage |
| --- | --- | --- | --- | --- |
| `val-080-gold-predicate-rowset-total-mvp` | `080` | Active MVP predicate row set is evaluated for required active rows and deterministic block rows. | `closed_active` or `blocked_todo` when any required row/schema is TODO-bearing. | 18 active rows, four block rows, row-set checksum, row checksums. |
| `val-080-gold-predicate-row-missing` | `080` | Candidate fact type/predicate has no active row. | `GOLD_FACT_PREDICATE_CONTRACT_MISSING`; no fact. | missing row and no mutation. |
| `val-080-gold-predicate-row-ambiguous` | `080` | Two equally specific active rows cover the same candidate. | `GOLD_FACT_PREDICATE_CONTRACT_AMBIGUOUS`; no fact. | ambiguity diagnostics and no row-order tiebreak. |
| `val-080-gold-predicate-subject-kind-boundary` | `080`, `040` | Candidate subject kind is outside selected row. | `GOLD_FACT_PREDICATE_SUBJECT_KIND_FORBIDDEN`; no ID computation. | every active subject-kind boundary. |
| `val-080-gold-predicate-object-kind-boundary` | `080`, `040` | Candidate object kind or `object_kind` mismatch is outside selected row. | `GOLD_FACT_PREDICATE_OBJECT_KIND_FORBIDDEN` or `040.CORE_ONE_OF_INVALID`; no ID computation. | object-kind equality and one-of closure. |
| `val-080-gold-predicate-reference-eligibility` | `080`, `070` | Entity type or identifier type is not permitted by selected row. | `GOLD_FACT_PREDICATE_REFERENCE_ENTITY_TYPE_FORBIDDEN` or `GOLD_FACT_PREDICATE_REFERENCE_IDENTIFIER_TYPE_FORBIDDEN`; no fact. | host, user, service_account, group, ip_address, dns_name. |
| `val-080-gold-predicate-null-forbidden` | `080` | Candidate uses `null_value` for an MVP active row. | `GOLD_FACT_PREDICATE_NULL_FORBIDDEN`; no fact. | null object rejection. |
| `val-080-gold-predicate-identity-like-string-rejected` | `080` | Candidate encodes identity-like value as top-level string. | `GOLD_FACT_PREDICATE_IDENTITY_STRING_FORBIDDEN`; no fact. | IP, DNS, hostname, provider key, mapped target, graph key, source-native ID strings. |
| `val-080-gold-predicate-structured-schema-required` | `080` | Candidate structured value omits required schema ref or uses inactive schema ref. | `GOLD_FACT_STRUCTURED_SCHEMA_MISSING`; no fact. | all seven MVP schema refs. |
| `val-080-gold-predicate-ocsf-nonauthority` | `050`, `060`, `080` | OCSF object, observable, severity, status, confidence, `raw_data`, or `unmapped` attempts to satisfy structured object authority. | owner non-authority error; no fact. | OCSF metadata preserved but no gold output. |
| `val-080-gold-predicate-source-authority-required` | `060`, `080` | Candidate has valid predicate row but no exact authority row. | source-authority owner error or deterministic block; no fact. | all active predicate rows. |
| `val-080-gold-predicate-source-dataset-required` | `020`, `080` | Candidate source dataset lacks selected catalog row. | `SOURCE_DATASET_CATALOG_ROW_MISSING`; no fact. | source-dataset handoff. |
| `val-080-gold-predicate-replay-checksum` | `030`, `080` | Predicate row checksum, structured schema checksum, or row-set checksum drifts during replay. | `REPLAY_CHECKSUM_MISMATCH` or `GOLD_FACT_STRUCTURED_SCHEMA_CHECKSUM_MISMATCH`; no replay output. | replay exactness. |
| `val-080-gold-predicate-block-row-reachability` | `080`, `090`, `110` | Modeled or theoretical reachability predicate candidate is selected. | deterministic no-output; no fact, graph edge, graph property, watermark, or API claim. | reachability block mutation prohibition. |
| `val-080-gold-predicate-block-row-ownership` | `080` | Host ownership predicate candidate is selected. | deterministic no-output; no fact or ownership label. | ownership block mutation prohibition. |
| `val-090-observed-connection-predicate-contract` | `090`, `080` | Observed connection graph projection uses missing or wrong predicate row. | no edge and no graph mutation. | exact `gfp-mvp-flow-observed-connection-v1` ref/checksum. |
| `val-030-gold-predicate-manifest-completeness` | `030`, `080` | Selected predicate row, block row, structured schema row, package-set ref, or validation ref is omitted from `VersionManifest`. | `VERSION_MANIFEST_INCOMPLETE` or `GOLD_FACT_PREDICATE_MANIFEST_INCOMPLETE`; no visible output. | manifest refs for gold, correction, graph projection, source authority, validation. |

`AcceptanceReport.result = pass` is forbidden when any `120-GOLD-PREDICATE-CATALOG-*` row is `blocked`, `not_run`, failed, stale, checksum-mismatched, package-set-mismatched, manifest-incomplete, or TODO-bearing.

This matrix verifies activation-controlled row schema precision for every production-affecting row family in `020`, `050`, `060`, `070`, `080`, `090`, `100`, `130`, and `140`. A row may pass only when fixture checksum, expected output checksum, expected error checksum, row checksum, row-set checksum, package-set refs when package-supplied, owner error registry rows, and `030.VersionManifest` requirements are concrete and non-`TODO`.

| validation_row_id | owner_spec | scenario | expected output or error | required fixture coverage |
| --- | --- | --- | --- | --- |
| `120-ACTIVATION-ROW-SCHEMA-AC-001` | all row owners, `030` | Every production-affecting activation row family has a full `030.ActivationControlledRowField` table or an explicit non-production, inactive, validation-only, deferred, or deterministic-block declaration. | pass or `ACTIVATION_ROW_SCHEMA_INCOMPLETE` | owner row-family inventory and field-table presence proof. |
| `120-ACTIVATION-ROW-SCHEMA-AC-002` | `030` | Identical materialized row bytes produce identical row checksums and identical row-set checksums. | byte-identical checksums | canonical bytes, row checksum, row-set checksum. |
| `120-ACTIVATION-ROW-SCHEMA-AC-003` | `030` and owner | Checksum-included fields affect checksums; checksum-excluded volatile fields do not. | expected checksum delta or no delta | included-field and excluded-field fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-004` | `030` and owner | Unknown fields fail unless a declared extension map exists. | `ACTIVATION_ROW_UNKNOWN_FIELD` or owner extension error | unknown field and extension-map fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-005` | `030` and owner | Null fails unless `null_allowed = yes`. | `ACTIVATION_ROW_NULL_FORBIDDEN` or owner null error | null and permitted-null fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-006` | `030` and owner | Omission fails for required fields and remains distinct from null. | `ACTIVATION_ROW_OMIT_FORBIDDEN` or owner omit error | omitted required, omitted optional, null required fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-007` | `030` and owner | `canonical_set` arrays sort deterministically and reject duplicates by default. | sorted checksum or `ACTIVATION_ROW_DUPLICATE_FORBIDDEN` | duplicate-after-canonicalization and unordered-input fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-008` | `030` and owner | `ordered_sequence` arrays preserve order and affect checksum in order. | distinct checksums for reordered inputs | ordered sequence fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-009` | `030` and owner | Bare string row refs fail where structured row refs are required. | `ACTIVATION_ROW_REF_INVALID` or owner ref error | bare string, malformed object, and valid row-ref fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-010` | `030` and owner | Selected row refs and row checksums appear in `VersionManifest`. | pass or `ACTIVATION_ROW_VERSION_MANIFEST_INCOMPLETE` | manifest inclusion and omission fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-011` | `030`, `100`, and owner | Package-supplied row catalogs require package-set refs. | pass or package-set owner error | package-supplied row catalog fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-012` | `010`, `030`, and owner | Extension fields cannot redefine stable behavior. | `ACTIVATION_ROW_EXTENSION_FORBIDDEN` or owner extension error | extension-map and stable-redefinition fixtures. |
| `120-ACTIVATION-ROW-SCHEMA-AC-013` | `000`, `030`, `120` | Missing row schema precision blocks authoritative promotion. | promotion failure | spec-set inventory and blocked promotion fixture. |

`AcceptanceReport.result = pass` is forbidden when any selected production row family lacks non-`TODO` fixtures and expected checksums for these cases. `ValidateSpecSet` must fail before acceptance aggregation when any owner row family is output-affecting and lacks either full row-field precision or an explicit non-production, inactive, validation-only, deferred, or deterministic-block classification.

### DefineOnceClosureValidationMatrix

This matrix verifies `000.DefineOnceClosureInventory`. A row may pass only when the inventory row has deterministic bytes, exact owner references, non-`TODO` fixture and expected-output checksums, and no forbidden mutation.

| validation_row_id | owner_spec | scenario | expected_failure_code | mutation_prohibition | blocking_status |
| --- | --- | --- | --- | --- | --- |
| `define-once-owner-export-present` | `000` | Every runtime contract reference resolves to one owner export or declared owner-export alias. | none on success; `OWNER_CONTRACT_REF_UNEXPORTED` on failure | no promotion | blocked |
| `define-once-import-ref-exact` | `000` | Import and route references use exact exported names and no inferred aliases. | `OWNER_CONTRACT_REF_UNEXPORTED` | no promotion | blocked |
| `define-once-non-owner-runtime-restatement` | `000` | Non-owner document restates schema, default, algorithm, failure precedence, activation behavior, or validation harness behavior. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output | blocked |
| `define-once-duplicate-domain-ledger-row` | `000`, `domain` | Section 25 duplicates an owner contract without distinct scope and validation family. | `DOMAIN_LEDGER_OWNER_DUPLICATE` | no promotion | blocked |
| `define-once-duplicate-owner-export` | `000` | More than one owner exports the same runtime contract name. | `DUPLICATE_OWNER_EXPORT` | no production output | blocked |
| `define-once-owner-spec-contradiction` | `000` | Two owners define incompatible behavior for one runtime contract. | `OWNER_SPEC_CONTRADICTION` | no production output | blocked |
| `define-once-reference-runtime-authority` | `000` | Research, ADR, reference, archive, or rationale text is cited as runtime authority. | `DOMAIN_RUNTIME_RESTATEMENT` or `OWNER_SPEC_CONTRADICTION` by context | no production output | blocked |

### IdentityResolverClosureValidationMatrix

This matrix is the executable validation interface for identity resolver closure. A row may pass only when `fixture_checksum` and `expected_output_checksum` are non-`TODO` SHA-256 values, every input artifact ref is present, and the observed output matches the expected output bytes. Rows with `TODO` checksums must remain `blocked` and must block authoritative promotion when identity output is in scope.

| validation_row_id | owner_spec | fixture_id | fixture_checksum | input_artifact_refs | expected_output_class | expected_error_or_no_op | expected_output_checksum | mutation_prohibition | version_manifest_requirement | acceptance_criterion | blocking_status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `val-070-resolver-profile-row-selection` | `070` | `fixture-070-profile-row-selection` | TODO | resolver profile, evidence classes, scopes, candidate profile, hard blocker row set, generation boundary row set, decision matrix, confidence bands, review, split, explanation, selector policy | decision | none | TODO | no unselected-row mutation | all resolver artifact refs and checksums | `070-RESOLVER-ROW-AC-001` | blocked |
| `val-070-evidence-class-registry-totality` | `070` | `fixture-070-evidence-totality` | TODO | evidence class row set and coverage matrix refs | owner error | `IDENTITY_EVIDENCE_CLASS_UNSUPPORTED` | TODO | no candidate pairs | evidence class row-set ref | `070-EVIDENCE-REGISTRY-TOTALITY-AC-001` | blocked |
| `val-070-identifier-scope-coverage` | `070` | `fixture-070-under-scoped-evidence` | TODO | identifier scope row set | owner error | `IDENTITY_EVIDENCE_UNDER_SCOPED` | TODO | no candidate pairs | identifier scope refs | `070-PAIR-ORDER-AC-001` | blocked |
| `val-070-durable-creation` | `070` | `fixture-070-durable-creation` | TODO | full resolver catalog | decision and explanation | none | TODO | no merge | full resolver catalog refs | `070-DURABLE-MERGE-AC-001` | blocked |
| `val-070-durable-attachment` | `070` | `fixture-070-durable-attachment` | TODO | full resolver catalog | decision and explanation | none | TODO | no merge of two canonical entities | full resolver catalog refs | `070-DURABLE-MERGE-AC-001` | blocked |
| `val-070-durable-merge` | `070` | `fixture-070-durable-merge` | TODO | full resolver catalog | decision and explanation | none | TODO | no weak evidence contribution | full resolver catalog refs | `070-DURABLE-MERGE-AC-001` | blocked |
| `val-070-weak-rejection` | `070` | `fixture-070-weak-rejection` | TODO | evidence classes, decision matrix, selector policy | no-op | `no_decision` | TODO | no create, attach, merge, split, or gold reference | full resolver catalog refs | `070-WEAK-REJECTION-AC-001` | blocked |
| `val-070-selector-safety` | `070` | `fixture-070-selector-safety` | TODO | selector safety policy | no-op | `TARGET_SELECTOR_UNSAFE` or no-op | TODO | no identity mutation and no graph endpoint | selector policy ref | `070-SELECTOR-SAFETY-TOTALITY-AC-001` | blocked |
| `val-070-source-native-merge-history-nonauthority` | `070` | `fixture-070-source-native-merge-history-only` | TODO | evidence classes and decision matrix | no-op | `no_decision` | TODO | no identity mutation | evidence class and decision refs | `070-WEAK-REJECTION-AC-001` | blocked |
| `val-070-learned-only-nonauthority` | `070` | `fixture-070-learned-only` | TODO | candidate profile and decision matrix | no-op | `no_decision` | TODO | no identity mutation | candidate profile and decision refs | `070-WEAK-REJECTION-AC-001` | blocked |
| `val-070-blocker-precedence` | `070` | `fixture-070-blocker-precedence` | TODO | hard blocker row set and generation boundary row set | owner error or rejection | `IDENTITY_HARD_BLOCKER_TRIGGERED` | TODO | no confidence-selected mutation | hard blocker and boundary refs | `070-BLOCKER-PRECEDENCE-AC-001` | blocked |
| `val-070-block-overflow` | `070` | `fixture-070-block-overflow` | TODO | candidate generation profile | owner error | `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | TODO | no mutation from overflowed block | candidate profile ref | `070-OVERFLOW-AC-001` | blocked |
| `val-070-partition-overflow` | `070` | `fixture-070-partition-overflow` | TODO | candidate generation profile | owner error | `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | TODO | no mutation from overflowed partition | candidate profile ref | `070-OVERFLOW-AC-001` | blocked |
| `val-070-decision-row-missing` | `070` | `fixture-070-decision-row-missing` | TODO | decision matrix row set | owner error | `RESOLVER_DECISION_ROW_MISSING` | TODO | no identity mutation | decision matrix ref | `070-RESOLVER-ROW-AC-001` | blocked |
| `val-070-decision-row-ambiguous` | `070` | `fixture-070-decision-row-ambiguous` | TODO | decision matrix row set | owner error | `RESOLVER_DECISION_ROW_AMBIGUOUS` | TODO | no identity mutation | decision matrix ref | `070-DECISION-ROW-AMBIGUITY-AC-001` | blocked |
| `val-070-confidence-band-row-selection` | `070` | `fixture-070-confidence-band-selection` | TODO | confidence band row set | decision | none | TODO | no score-derived mutation without decision row | confidence band ref | `070-DURABLE-MERGE-AC-001` | blocked |
| `val-070-review-totality` | `070` | `fixture-070-review-totality` | TODO | review routing policy and review state machine | review case | none | TODO | mutation only through terminal decision | review policy and transition refs | `070-REVIEW-TOTALITY-AC-001` | blocked |
| `val-070-review-expiration` | `070` | `fixture-070-review-expiration` | TODO | review routing policy | review case | expired no-op | TODO | no identity mutation | review policy ref and transition evidence | `070-REVIEW-EXPIRATION-AC-001` | blocked |
| `val-070-reviewer-authority-missing` | `070` | `fixture-070-reviewer-authority-missing` | TODO | review routing policy | owner error | `IDENTITY_REVIEW_AUTHORITY_MISSING` | TODO | no identity mutation | review case and auth refs | `070-REVIEW-TOTALITY-AC-001` | blocked |
| `val-070-evidence-snapshot-mismatch` | `070` | `fixture-070-evidence-snapshot-mismatch` | TODO | review routing policy | owner error | `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` | TODO | no identity mutation | review case refs | `070-REVIEW-TOTALITY-AC-001` | blocked |
| `val-070-hard-blocker-override-rejection` | `070` | `fixture-070-hard-blocker-override` | TODO | hard blocker row set and review policy | owner error or rejection | `IDENTITY_HARD_BLOCKER_TRIGGERED` | TODO | no reviewer override mutation | hard blocker and review refs | `070-BLOCKER-PRECEDENCE-AC-001` | blocked |
| `val-070-split-handoff` | `070` | `fixture-070-split-handoff` | TODO | split policy and explanation policy | handoff | none | TODO | resolver emits no graph mutation | split, explanation, and manifest refs | `070-GRAPH-CORRECTION-HANDOFF-SCHEMA-AC-001` | blocked |
| `val-070-ambiguous-split-partition` | `070` | `fixture-070-ambiguous-split-partition` | TODO | split policy | owner error or conflicted decision | `conflicted` or policy error | TODO | no graph handoff unless policy permits | split policy ref | `070-SPLIT-HANDOFF-AC-001` | blocked |
| `val-070-explanation-checksum` | `070` | `fixture-070-explanation-checksum` | TODO | explanation policy | explanation | none | TODO | no decision visibility without explanation | explanation policy and manifest refs | `070-EXPLANATION-AC-001` | blocked |
| `val-070-replay-exactness` | `070` | `fixture-070-replay-exactness` | TODO | full resolver catalog | replay decision/explanation | none | TODO | no nondeterministic divergence | full manifest refs | `070-AC-007` | blocked |
| `val-070-manifest-artifact-omission` | `070`, `030` | `fixture-070-manifest-artifact-omission` | TODO | omitted resolver catalog member | owner error | `VERSION_MANIFEST_INCOMPLETE` | TODO | no identity output | missing ref detected | `030-IDENTITY-MANIFEST-AC-001` | blocked |
| `val-070-package-weak-default-weakening` | `070`, `100` | `fixture-070-package-weak-default-override` | TODO | package-supplied resolver artifact | owner error | package activation failure | TODO | preserve active package set; no identity output | package set and artifact refs | `100-IDENTITY-RESOLVER-WEAKENING-AC-001` | blocked |
| `val-070-two-independent-implementer-parity` | `070`, `120` | `fixture-070-implementer-parity` | TODO | full resolver catalog | decision/explanation/handoff | none | TODO | no byte divergence | full manifest refs | `070-AC-007` | blocked |

## Spec Set Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `SPECSET-AC-001` | Every implementation-relevant contract has exactly one owning spec. |
| `SPECSET-AC-002` | Every active spec declares imports, exports, dependencies, explicit non-scope, and Definition of Done. |
| `SPECSET-AC-003` | No active NLSpec restates another active NLSpec requirement except by exact named reference. |
| `SPECSET-AC-004` | Every production input path remains lakehouse-fed and has no direct enterprise source-call behavior. |
| `SPECSET-AC-005` | Every domain spec has binary acceptance criteria and at least one negative validation case for each forbidden authority boundary it declares. |
| `SPECSET-AC-006` | The deferred reachability document is inactive and no active MVP graph or gold spec can emit theoretical reachability output. |
| `SPECSET-AC-007` | The recreatability test passes using only active NLSpecs, deferred NLSpecs where explicitly referenced, and accepted standards. |
| `SPECSET-AC-008` | Domain unresolved/resolved status and owner-local closure states are consistent. |
| `SPECSET-AC-009` | ADR statuses are members of `000.SpecStatus` and accepted ADRs use `accepted_rationale`. |
| `SPECSET-AC-010` | Every manifest path has exactly one registry row or explicit non-registry reason. |
| `SPECSET-AC-011` | Every implementation-relevant contract, profile, row set, package, fixture, registry object, and runtime state record has exactly one volatility class. |
| `SPECSET-AC-012` | No activation-controlled artifact can define new runtime behavior not exported by a stable core contract. |
| `SPECSET-AC-013` | Every output-affecting activation-controlled artifact is represented by `030.ActivationControlledArtifactRef` and appears in `VersionManifest`. |
| `SPECSET-AC-014` | Stable core specs contain no production-active volatile rows except non-normative examples or blocking `TODO:` rows. |
| `SPECSET-AC-015` | Every active absence-sensitive feed category has `SourceAuthorityClosureMatrix` validation rows or deterministic block rows. |
| `SPECSET-AC-016` | When observability is in implementation scope, every telemetry contract, artifact, metric catalog, runtime state record, redaction rule, exporter profile, health mapping, replay exclusion, and non-authority rule has exactly one owner, one volatility class, and passing `120-OBSERVABILITY-*` validation rows. |
| `SPECSET-AC-017` | Core one-of closure rows for `GoldFact.subject_ref`, `GoldFact.object_value`, and `EvidenceRef.artifact_id` are present, non-`TODO`, and covered by `CoreOneOfClosureValidationMatrix` and `CoreEvidenceArtifactValidationMatrix`. |
| `SPECSET-AC-018` | Evidence artifact class/kind pairings are total for declared artifact classes and missing, ambiguous, inactive, checksum-mismatched, or unvalidated pairings block promotion. |

## Validation Contract Details

### CoreRecordSchemaValidationMatrix

| Row class | Required coverage |
| --- | --- |
| `040-schema-success` | Minimal valid record for every 040 exported record emits expected canonical bytes and checksum. |
| `040-required-field-rejection` | Missing required field fails with owner-specific missing-field code. |
| `040-null-rejection` | Explicit null in a non-nullable field fails with `CORE_NULL_FORBIDDEN`. |
| `040-unknown-field-rejection` | Unknown field outside extension maps fails with `CORE_UNKNOWN_FIELD`. |
| `040-default-materialization` | Required default `[]`, `{}`, enum, or null is materialized exactly as specified. |
| `040-array-sort` | Arrays with declared sort keys produce byte-identical output regardless of input order. |
| `040-id-collision` | Simulated collision fails with owner collision error and commits no record. |
| `040-checksum-replay` | Same record input produces same checksum across replay. |
| `040-raw-payload-boundary` | `EvidenceRef` and graph deltas cannot inline raw payload bytes. |
| `040-backend-id-rejection` | Backend-generated graph IDs fail. |
| `040-gold-fact-key-id-separation` | Comparable semantic fact key remains stable while immutable fact record ID changes across knowledge versions. |
| `040-two-independent-implementers` | Two implementations produce byte-identical IDs and checksums for every core record fixture. |
| `040-core-oneof-closure` | Closed subject, object, and evidence artifact one-of fixtures prove valid serialization, invalid-shape rejection, null/omission distinction, artifact-class pairing, ID input inclusion, checksum input inclusion, and two-implementation parity. |
| `040-core-evidence-artifact-closure` | Evidence artifact fixture families prove class/kind pairing, checksum basis, raw-payload rejection, and manifest ref requirements. |
| `040-source-extension-rule-shape` | Valid, missing-field, unknown-field, bounds, non-canonical ordering, checksum replay, and pre-activation rejection fixtures for `SourceExtensionFieldRuleShape`. |

### CoreRecordSchema fixture families

| Fixture ID family | Records |
| --- | --- |
| `core-minimal-valid-*` | One minimal valid fixture per exported record. |
| `core-required-missing-*` | One missing-field fixture per record. |
| `core-null-forbidden-*` | One null-forbidden fixture per nullable or non-null boundary class. |
| `core-unknown-field-*` | Unknown field outside extension map. |
| `core-extension-map-*` | Declared extension allowed; undeclared extension rejected. |
| `core-array-sort-*` | Lexical and ID sort validation. |
| `core-checksum-*` | Checksum inclusion/exclusion validation. |
| `core-id-collision-*` | Collision simulation fixture. |
| `core-evidence-raw-payload-*` | Raw payload leak rejection. |
| `core-graph-backend-id-*` | Backend ID rejection. |
| `core-goldfact-correction-*` | Key ID versus immutable ID replay sequence. |
| `core-oneof-subject-*` | Valid subject refs, unknown kind, missing member, extra member, null member, raw record ref, silver observation ref, and graph/backend ID rejection. |
| `core-oneof-object-*` | Valid reference and scalar object values, object-kind mismatch, unknown kind, missing member, extra member, forbidden null, identity-like string rejection, and structured schema missing rejection. |
| `core-oneof-evidence-*` | Valid evidence artifact refs, unknown kind, missing member, multiple members, null, missing checksum, checksum mismatch, raw payload rejection, and artifact-class/kind mismatch. |
| `core-oneof-id-replay-*` | Same-bytes replay, kind-change ID/checksum changes, null/omission/empty distinction, and two-independent-implementer parity. |

### CoreOneOfClosureValidationMatrix

| Fixture family | Required result |
| --- | --- |
| `core-oneof-subject-valid-canonical-entity-ref` | Valid `canonical_entity_ref` serializes canonically and participates in `gold_fact_key_id`. |
| `core-oneof-subject-valid-source-asset-ref` | Valid `source_asset_ref` serializes canonically and participates in `gold_fact_key_id`. |
| `core-oneof-subject-valid-identifier-ref` | Valid `identifier_ref` serializes canonically and participates in `gold_fact_key_id`. |
| `core-oneof-subject-reject-unknown-kind` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-subject-reject-missing-member` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-subject-reject-extra-member` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-subject-reject-null-member` | Fails with `CORE_ONE_OF_INVALID` or `CORE_NULL_FORBIDDEN` as declared by field shape. |
| `core-oneof-subject-reject-raw-record-ref` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-subject-reject-silver-observation-ref` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-subject-reject-graph-backend-id` | Fails with `GRAPH_BACKEND_ID_FORBIDDEN` or `CORE_ONE_OF_INVALID` when presented as a malformed one-of. |
| `core-oneof-object-valid-canonical-entity-ref` | Valid reference object serializes canonically. |
| `core-oneof-object-valid-source-asset-ref` | Valid reference object serializes canonically. |
| `core-oneof-object-valid-identifier-ref` | Valid reference object serializes canonically. |
| `core-oneof-object-valid-string` | Valid bounded string object serializes canonically. |
| `core-oneof-object-valid-enum` | Valid enum object serializes canonically. |
| `core-oneof-object-valid-boolean` | Valid boolean object serializes canonically. |
| `core-oneof-object-valid-int64` | Valid int64 object serializes canonically. |
| `core-oneof-object-valid-uint64` | Valid uint64 object serializes canonically. |
| `core-oneof-object-valid-decimal` | Valid decimal object serializes canonically under `040.DecimalPrecisionPolicy`. |
| `core-oneof-object-valid-timestamp` | Valid UTC timestamp object serializes canonically. |
| `core-oneof-object-valid-structured` | Valid structured object requires active schema ref. |
| `core-oneof-object-valid-null` | Valid only when `080.null_object_policy = allowed`. |
| `core-oneof-object-reject-object-kind-mismatch` | Fails before `gold_fact_key_id` computation. |
| `core-oneof-object-reject-unknown-kind` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-object-reject-missing-member` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-object-reject-extra-member` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-object-reject-forbidden-null` | Fails before ID computation. |
| `core-oneof-object-reject-identity-like-string` | Fails with the most specific `080` predicate-contract error. |
| `core-oneof-object-reject-structured-schema-missing` | Fails before ID computation. |
| `core-oneof-id-replay-same-bytes` | Same materialized bytes produce the same IDs and checksums. |
| `core-oneof-id-change-subject-kind` | Changing `subject_ref.kind` changes affected IDs and checksums exactly as `040` declares. |
| `core-oneof-id-change-object-kind` | Changing `object_value.kind` changes affected IDs and checksums exactly as `040` declares. |
| `core-oneof-null-omission-empty-distinction` | Null sentinel, field omission, empty string, empty object, and empty array remain distinct. |
| `core-oneof-two-independent-implementers` | Two independent implementations produce byte-identical IDs and checksums for all closure fixtures. |

### CoreEvidenceArtifactValidationMatrix

| Fixture family | Required result |
| --- | --- |
| `core-oneof-evidence-valid-cadastre-record-ref` | Valid `cadastre_record_ref` uses referenced record checksum. |
| `core-oneof-evidence-valid-activation-artifact-ref` | Valid `activation_artifact_ref` uses immutable artifact bytes or owner-declared canonical metadata bytes. |
| `core-oneof-evidence-valid-lakehouse-artifact-ref` | Valid `lakehouse_artifact_ref` uses immutable lakehouse bytes or owner-declared canonical metadata bytes. |
| `core-oneof-evidence-valid-external-artifact-ref` | Valid `external_artifact_ref` uses immutable external bytes or owner-declared canonical metadata bytes. |
| `core-oneof-evidence-reject-unknown-kind` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-evidence-reject-missing-member` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-evidence-reject-multiple-members` | Fails with `CORE_ONE_OF_INVALID`. |
| `core-oneof-evidence-reject-null` | Fails with `CORE_ONE_OF_INVALID` or `CORE_NULL_FORBIDDEN` as declared by field shape. |
| `core-oneof-evidence-reject-missing-checksum` | Fails with `CORE_REQUIRED_FIELD_MISSING`. |
| `core-oneof-evidence-reject-checksum-mismatch` | Fails with `CORE_RECORD_CHECKSUM_MISMATCH` or the most specific owner checksum code. |
| `core-oneof-evidence-reject-raw-payload` | Fails with `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN`. |
| `core-oneof-evidence-reject-artifact-class-kind-mismatch` | Fails with `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH`. |
| `core-oneof-id-change-evidence-artifact-kind` | Changing `artifact_id.kind` changes `evidence_ref_id` and record checksum. |

### ValidationMatrixRow schema

| Field | Required | Rule |
| --- | ---: | --- |
| `row_id` | Yes | Stable unique ID. |
| `owner_spec` | Yes | Exact owner spec ID. |
| `contract` | Yes | Named exported contract. |
| `scenario` | Yes | Success, rejection, no-op, edge, replay, redaction, authorization, or negative authority-boundary. |
| `fixture` | Yes | Fixture ID and checksum. |
| `inputs` | Yes | Canonical input refs and checksums. |
| `expected_outputs` | Required unless no-op/error expected | Canonical expected output checksum. |
| `expected_no_op` | Required when no-op expected | Boolean plus explanation. |
| `expected_error` | Required when error expected | Owner-specific code. |
| `mutation_prohibition` | Yes | Production mutation classes forbidden by the row. |
| `checksums` | Yes | Input, fixture, expected, and actual checksums. |
| `pass_fail_evidence` | Yes after run | `pass`, `fail`, `blocked`, or `not_run`. `pass` is forbidden when any required fixture checksum, expected output checksum, expected no-op assertion, expected error, mutation prohibition, activation artifact ref, or version manifest ref is `TODO`, omitted, stale, or checksum-mismatched. |
| `volatility_class` | No | Required when the row validates a volatility boundary. |
| `activation_artifact_refs` | No | Required refs and checksums for output-affecting activation-controlled artifacts. |
| `stable_core_owner` | No | Owner stable contract when validating activation artifacts. |
| `artifact_owner_spec` | No | Artifact owner spec when different from stable core owner. |
| `version_manifest_requirement` | No | Expected `VersionManifest` inclusion rule. |
| `artifact_mutation_prohibition` | No | Artifact classes and stable-core fields that the row forbids mutating. |

### OwnerLocalStatusConsistencyRule validation rows

| Row ID | Scenario | Expected failure code | Mutation prohibition |
| --- | --- | --- | --- |
| `domain-owner-status-raw-feed-manifest-closed` | Domain marks `RawFeedManifest` unresolved while `020` is `closed_local`. | `DOMAIN_OWNER_STATUS_CONTRADICTION` | no production output |
| `domain-owner-status-raw-record-id-closed` | Domain marks raw record ID unresolved while `040.ComputeRawRecordId` and `020` import contribution are `closed_local`. | `DOMAIN_OWNER_STATUS_CONTRADICTION` | no production output |
| `domain-runtime-restatement-raw-id-order` | Domain copies raw ID input order instead of routing to `040.ComputeRawRecordId`. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `owner-spec-contradiction-same-runtime-contract` | Two owner specs define incompatible runtime behavior for one named contract. | `OWNER_SPEC_CONTRADICTION` | no production output |
| `adr-status-unregistered` | ADR status is not in `000.SpecStatus`. | `ADR_STATUS_UNREGISTERED` | no promotion |
| `registry-manifest-path-mismatch` | Manifest path lacks exactly one registry row or explicit non-registry reason. | `REGISTRY_MANIFEST_PATH_MISMATCH` | no promotion |
| `domain-ledger-owner-duplicate` | Section 25 has duplicate owner-contract rows without distinct `owner_contract_scope` and validation families. | `DOMAIN_LEDGER_OWNER_DUPLICATE` | no promotion |
| `owner-contract-ref-unexported` | An import, ledger row, validation row, or spec-set validation ref names a contract that is not an owner export or alias. | `OWNER_CONTRACT_REF_UNEXPORTED` | no promotion |
| `duplicate-owner-export` | Two owners export the same runtime contract name without a same-owner alias declaration. | `DUPLICATE_OWNER_EXPORT` | no production output |
| `non-owner-runtime-restatement-schema` | Non-owner document copies an owner schema or field table. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `non-owner-runtime-restatement-default` | Non-owner document copies or changes an owner default or bound. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `non-owner-runtime-restatement-algorithm` | Non-owner document copies an owner algorithm or step order. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `non-owner-runtime-restatement-failure-precedence` | Non-owner document copies failure precedence or owner error ordering. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `non-owner-runtime-restatement-activation` | Non-owner document defines activation behavior owned elsewhere. | `DOMAIN_RUNTIME_RESTATEMENT` | no production output |
| `non-owner-runtime-restatement-validation-harness` | Non-owner document defines validation harness behavior owned by `120`. | `DOMAIN_RUNTIME_RESTATEMENT` | no promotion |

### DomainSection25StatusValidationMatrix

| Row ID | Scenario | Expected result |
| --- | --- | --- |
| `domain-section25-structured-input-repository-blocked-validation` | Structured-input repository row is `blocked_validation` while repository profile, snapshot, materialization, package-set, redaction, audit, telemetry, or manifest refs are missing. | Acceptance blocked. |
| `domain-section25-source-closure-blocked-validation` | Domain row for source-specific closure is `blocked_validation`; `060` closure row instances or `120-SOURCE-CLOSURE-*` rows are missing. | Acceptance blocked, no production output. |
| `domain-section25-correction-snapshot-resolved` | Domain row is resolved; `080.CorrectionSnapshotRefPolicy` has no owner TODO and required validation rows exist. | Pass only when `080` rows are not blocked. |
| `domain-section25-replay-equivalence-resolved` | Domain row is resolved; `080.ReplayEquivalencePolicy` routes output class field selection to owners. | Pass. |
| `domain-section25-ocsf-blocked-validation` | Domain row for MVP OCSF mapping is `blocked_validation` while row instances/checksums are missing. | Acceptance blocked. |
| `domain-section25-graph-profile-blocked-validation` | MVP edge semantics are closed but graph fixture checksums are missing. | Acceptance blocked. |
| `domain-section25-lifecycle-blocked-validation` | Lifecycle owners have machine definitions but validation rows are missing or blocked. | Acceptance blocked. |
| `domain-section25-runlock-blocked-validation` | Run-lock owner behavior exists in `030` but closure rows or checksums are missing. | Acceptance blocked. |
| `domain-section25-domain-exports-none` | `domain.md` declares no runtime exports and no runtime row, schema, or algorithm is found. | Pass. |
| `domain-section25-package-type-enum-resolved` | `100.PackageType` confirmed enum has no duplicates and no generic `deployment_profile`. | Pass only when the active `100.PackageType` enum validation rows pass. |
| `domain-section25-graph-backend-selection-blocked-validation` | `090.GraphBackendSelectionPolicy` owns default backend selection and production activation remains blocked until graph backend validation rows pass. | Acceptance blocked until `120-GRAPH-BACKEND-*` rows pass. |
| `domain-section25-context-map-relationship-blocked-validation` | Context-map relationship type vocabulary validation rows are missing or blocked. | Acceptance blocked. |
| `domain-section25-context-map-edge-blocked-validation` | Context-map edge matrix validation rows are missing or blocked. | Acceptance blocked. |
| `domain-section25-observability-blocked-validation` | Telemetry non-authority and telemetry health mapping rows are missing or blocked. | Acceptance blocked. |
| `domain-section25-core-oneof-blocked-validation` | Core one-of and evidence artifact closure rows are missing, blocked, or `TODO`-bearing. | Acceptance blocked. |
| `domain-section25-identity-blocked-validation` | Resolver row-set and resolver activation artifact validation rows are missing, blocked, or `TODO`-bearing. | Acceptance blocked. |
| `domain-section25-graph-backend-duplicate-owner-route` | Section 25 contains duplicate rows for `090.GraphBackendSelectionPolicy` without distinct scope and validation family. | Fail with `DOMAIN_LEDGER_OWNER_DUPLICATE`. |

### ValidationCoverageMatrix

| Owner spec | Required success | Required rejection | Required no-op | Required edge | Required replay | Required redaction | Required authorization | Required negative authority-boundary | Required volatility-boundary |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | authority class | direct source call | validation exploration no-op | private/public boundary | n/a | private leak | API/private boundary | direct source, projection authority | required when owner declares activation-controlled artifacts. |
| `020` | feed read/import and category closure | malformed manifest, profile schema incomplete, category row missing | partial no-absence, empty-scope no-absence | CDC metadata and closure branch behavior | raw ID replay | private binding | n/a | no direct source call and unresolved profile branch | required when owner declares activation-controlled artifacts. |
| `030` | DAG execution | forbidden output | safe subset no-op | lifecycle illegal transition | manifest replay | n/a | n/a | subset authority | required when owner declares activation-controlled artifacts. |
| `040` | canonical ID/checksum | unknown field | omission distinction | collision | checksum replay | redaction state | n/a | backend ID rejection | required when owner declares activation-controlled artifacts. |
| `050` | OCSF mapping row success by row family | missing row, ambiguous row, missing discriminator, forbidden field, deprecated field, unknown enum, IPAM split, artifact mismatch | cadastre-only null-profile no-op when row permits | unknown enum and mapping discriminator branches | mapping replay | source extension redaction | n/a | undeclared extension and source-extension wildcard | required when owner declares activation-controlled artifacts. |
| `060` | absence authorized and empty-complete authorized | missing authority row, ambiguous authority row, missing completeness row, missing upstream evidence, missing coverage row, missing staleness row, missing control-result mapping, and external-schema authority attempt | weak signal no-op, weak-signal combination blocked, effect-not-allowed no-op, OCSF non-authority no-op, DNS TTL no deletion, DHCP lease no host absence, and missing-flow unknown | blocking precedence, partial completeness, source-history outside-window, and directory visibility gaps | watermark replay, closure-manifest replay, and watermark blocked | n/a | n/a | missing coverage, weak-signal combination, source-history no-change, DNS/DHCP/flow negative non-authority, and OCSF status/severity/confidence non-authority | required when owner declares activation-controlled artifacts. |
| `070` | creation, attachment, exact durable merge, split, and no-decision | weak-only, under-scoped, blocker, missing row, ambiguous row, and overflow | selector-only and unsupported entity | split handoff | decision/explanation checksum replay | resolver private-binding redaction through `010`/`110` | reviewer authority | review cannot mutate identity without terminal decision | inactive/missing/checksum-mismatched/out-of-scope resolver artifacts and package core conflict. |
| `080` | fact derivation | temporal missing | duplicate correction no-op | late arrival | replay mismatch | n/a | n/a | no current-time fallback | required when owner declares activation-controlled artifacts. |
| `090` | graph query/apply and positive observed-connection projection plus backend default selection | backend ID rejection, OCSF endpoint-order direction rejection, unresolved endpoint rejection, backend taxonomy unmapped rejection, query candidate-limit rejection, page-token rejection, missing provider field rejection, unsupported provider capability, unsafe storage mode, implicit schema creation, stale mixed index, full-scan Gremlin plan, and missing package gate | empty traversal no-path, missing-flow-role no-edge, ambiguous-flow-role no-edge, generic external graph payload no-pathfinding, and graph serving disabled no-backend | active profile exact edge set, edge semantics, output eligibility, apply order, JanusGraph schema/index/apply rows, and provider portability | rebuild equivalence, graph profile replay, provider defaulting replay, and backend manifest replay | raw property redaction, backend ID redaction, package evidence redaction | graph auth, output eligibility, backend health, and provider gate visibility | reachability prohibition, endpoint-order non-authority, selector-only endpoint non-authority, backend-state non-authority, provider-default non-authority, and raw Gremlin bypass non-authority | required when owner declares activation-controlled artifacts. |
| `100` | package type policy, supported repository forms, trust, transparency, attestation, provenance, SBOM, dependency lock, compatibility, deprecation, health, rollback, quarantine, emergency, and package manifest success | unknown type, missing policy, ambiguous policy, unsupported Git tree snapshot, unauthorized signer, threshold failure, missing transparency, missing attestation, missing SBOM, live dependency resolution, compatibility failure, expired deprecation, failed health gate, mutable rollback target, rollback incompatibility, quarantine block, emergency bypass, and package manifest omission | failed candidate keep-current, health-fail not-LKG, quarantine no mutation, and shadow/canary no current output | rollback preflight and emergency bounded actions | package-set replay and manifest completeness | package evidence redaction through `110` | promotion auth and emergency quorum | signature/trust bypass forbidden, dependency lock non-authority, and mutable Git ref non-authority | required when owner declares activation-controlled artifacts. |
| `110` | API success | stale compliance rejection | empty result | state label | page token replay | raw payload redaction | inaccessible asset | state-label non-collapse | required when owner declares activation-controlled artifacts. |
| `130` | analysis finding, metric, risk acceptance, threat-intel enrichment, lineage output, registry activation, and derived-edge routed output | mutation rejection, missing threat-intel profile, distribution unmapped, lineage policy missing, mutable schema, namespace collision, registry custom-property invalid, classification authority forbidden, derived-edge missing support, and projection forbidden | risk scoring disabled, explicit derived-edge no-op, sighting no-completeness, freshness no-completeness, facet-only no-evidence | lineage facet row, threat-intel distribution, registry package-set, and derived-edge route | analysis replay, threat-intel replay, lineage replay, registry replay, and derived-edge replay | lineage facet redaction, threat-intel raw-value redaction, registry custom-property redaction, private activation-scope redaction | registry auth, analysis read, graph-backed analysis auth, and raw expansion auth | analysis/enrichment/lineage/registry non-authority, identity forbidden, graph mutation forbidden, direct gold output forbidden | required when owner declares activation-controlled artifacts. |
| `140` | profile validation, trace context propagation, metric catalog row, exporter runtime state, health mapping | forbidden signal, missing profile, invalid exporter, forbidden attribute, high-cardinality metric, raw payload, private binding, backend ID, replay field | exporter unavailable with domain no-op; telemetry disabled no-op | trace-stage handoff, async context handoff, health mapping branch | telemetry enabled/disabled domain replay parity and telemetry-health replay | telemetry attribute, trace context, exporter endpoint, Collector route, private binding, backend ID | health diagnostic permission, secure diagnostics permission | telemetry cannot mutate source, identity, facts, graph, package, audit, watermark, replay | inactive/missing/checksum-mismatched telemetry artifacts, runtime-state substitution, package-set mismatch |
| `200` | n/a while deferred | reachability prohibited | no-op | no graph effect | n/a | n/a | n/a | deferred reachability | required when owner declares activation-controlled artifacts. |

### ObservabilityValidationMatrix

| Row ID | Scenario | Expected result |
| --- | --- | --- |
| `120-OBSERVABILITY-NONAUTH-001` | Span success attempts to prove stage success or fact correctness. | `TELEMETRY_AUTHORITY_VIOLATION`; no domain mutation. |
| `120-OBSERVABILITY-NONAUTH-002` | Metric value `0` attempts to prove source absence. | No-op or `TELEMETRY_AUTHORITY_VIOLATION`; no absence, cleanup, graph expiry, or watermark. |
| `120-OBSERVABILITY-NONAUTH-003` | Exporter delivery success attempts to replace `AuditEvent`. | Reject; audit persistence remains independent. |
| `120-OBSERVABILITY-REDACTION-001` | Telemetry contains raw payload bytes. | `TELEMETRY_RAW_PAYLOAD_FORBIDDEN`; no export. |
| `120-OBSERVABILITY-REDACTION-002` | Telemetry contains private source binding or credential. | `TELEMETRY_PRIVATE_BINDING_FORBIDDEN`; no export. |
| `120-OBSERVABILITY-REDACTION-003` | Graph telemetry contains backend-generated ID or provider query text. | `TELEMETRY_BACKEND_ID_FORBIDDEN` or graph-specific owner error; no export. |
| `120-OBSERVABILITY-CARDINALITY-001` | Metric catalog uses source-native object ID, canonical entity ID, hostname, IP, user name, backend ID, or private route as metric label. | `TELEMETRY_CARDINALITY_VIOLATION`; profile activation fails. |
| `120-OBSERVABILITY-EXPORTER-001` | Exporter unavailable during production run. | Domain output unchanged; `TelemetryRuntimeState` records failure; health degraded. |
| `120-OBSERVABILITY-HEALTH-001` | `health_scope = observability` with exporter failure. | `OperationalHealthStatus.status = degraded` unless active mapping says blocked. |
| `120-OBSERVABILITY-REPLAY-001` | Same input with different trace/span IDs. | Authoritative output checksums identical. |
| `120-OBSERVABILITY-REPLAY-002` | Replay checksum includes trace ID or span ID. | `TELEMETRY_REPLAY_FIELD_FORBIDDEN`. |
| `120-OBSERVABILITY-AUDIT-001` | Audit event exists but telemetry export fails. | Audit remains persisted or deterministically refused independent of telemetry. |
| `120-OBSERVABILITY-VERSION-MANIFEST-001` | Health/API output depends on telemetry policy but `VersionManifest` lacks required telemetry refs. | `TELEMETRY_VERSION_MANIFEST_INCOMPLETE` or `VERSION_MANIFEST_INCOMPLETE`. |
| `120-OBSERVABILITY-PACKAGE-001` | Package-supplied telemetry policy missing trust or compatibility evidence. | Package-set activation fails; current active set remains. |

### Observability fixture families

```text
observability-profile-valid-*
observability-profile-missing-*
observability-signal-forbidden-*
observability-attribute-forbidden-*
observability-cardinality-unbounded-*
observability-exporter-outage-*
observability-health-degraded-*
observability-replay-different-trace-id-*
observability-audit-export-failure-*
observability-version-manifest-missing-*
observability-package-policy-invalid-*
```

### ObservationTypeExternalMappingValidationMatrix

Every active `ObservationToOCSFMappingRow` must have one validation row in this matrix for a positive mapping case and one or more negative rows for discriminator, enum, object-path, forbidden-field, source-extension, and non-authority boundaries that the row can encounter.

| Row field | Required content |
| --- | --- |
| `owner_spec` | `050` for mapping behavior; `060` or `090` when the row proves a non-authority or graph-direction boundary. |
| `contract` | `ResolveOCSFMapping`, `SourceExtensionFieldRuleSet`, `DeriveAbsenceOrUnknown`, or `ProjectGraphDeltas`. |
| `scenario` | `success`, `rejection`, `no-op`, `redaction`, `replay`, or `negative authority-boundary`. |
| `fixture` | Exact fixture ID and checksum. |
| `expected_outputs` | Expected silver output checksum for success; omitted for rejection/no-op. |
| `expected_error` | Exact owner error code for rejection. |
| `mutation_prohibition` | `no silver output` for mapping rejection; `no authoritative mutation` for non-authority; `no graph mutation` for direction rejection. |
| `version_manifest_refs` | Every artifact ref required by `030.VersionManifestCompletenessMatrix` for the scenario. |
| `acceptance_criterion` | Exact `050-OCSF-*`, `050-SOURCE-EXT-*`, `060-OCSF-*`, or `090-OCSF-*` criterion. |
| `blocking_status` | `blocking` until fixture checksum, expected output checksum or expected error, and version-manifest refs are filled. |

Each mapping validation row must include the following row class. Row classes are closed for OCSF mapping activation; unknown row classes fail validation before acceptance aggregation.

| Row class | Required coverage |
| --- | --- |
| `050-ocsf-positive` | One success row per active `ObservationToOCSFMappingRow`. |
| `050-ocsf-row-missing` | Missing row fails before silver output. |
| `050-ocsf-row-ambiguous` | Ambiguous row fails before silver output. |
| `050-activity-discriminator` | Missing or ambiguous activity discriminator fails before output. |
| `050-object-path` | Required path missing and forbidden path emitted cases. |
| `050-enum` | Unknown enum, `Other` not permitted, deprecated enum, sibling mismatch. |
| `050-source-extension` | Declared success, undeclared path, wildcard namespace, reserved collision, missing redaction, secret scan failure. |
| `050-cadastre-only` | Null external profile accepted only for exact active `cadastre_only` row. |
| `050-dhcp-ipam-split` | DHCP maps only for `assignment_source = dhcp`; IPAM requires `cadastre_only` or fails. |
| `060-ocsf-nonauthority` | OCSF status, severity, confidence, observables, enrichments, class/activity, field presence, field absence. |
| `090-ocsf-direction` | Endpoint order cannot determine direction; missing or ambiguous flow-role evidence emits no edge. |

Required `050` fixture families:

```text
ocsf-mapping-row-success-*
ocsf-mapping-row-missing-*
ocsf-mapping-row-ambiguous-*
ocsf-activity-discriminator-missing-*
ocsf-required-object-path-missing-*
ocsf-forbidden-field-*
ocsf-unknown-enum-*
ocsf-other-enum-not-permitted-*
ocsf-deprecated-field-*
source-extension-rule-success-*
source-extension-rule-undeclared-*
source-extension-wildcard-rejected-*
source-extension-secret-scan-*
cadastre-only-null-profile-*
dhcp-ipam-split-required-*
```

Required `060` fixture families:

```text
ocsf-status-non-authority-*
ocsf-severity-non-authority-*
ocsf-confidence-non-authority-*
ocsf-observable-non-authority-*
ocsf-field-absence-non-authority-*
```

Required `090` fixture families:

```text
ocsf-endpoint-order-no-graph-direction-*
flow-role-evidence-required-*
```

Each mapping validation row must include:

```text
fixture_id
fixture_checksum
input_artifact_refs
selected_or_blocked_mapping_row_refs
expected_output_checksum for success
expected_error for rejection
mutation_prohibition
version_manifest_refs
acceptance_criterion
blocking_status
```

`blocking_status` must become `passed` only when fixture checksum, expected output checksum or expected error, mutation prohibition, and manifest refs are complete. `090-ocsf-direction` rows must include active `050.ObservationToOCSFMappingRow` refs, the source silver observation checksum, graph projection profile ref, graph edge semantics row ref, and `VersionManifest` ref.

TODO: Governance must provide exact fixture bytes, fixture checksums, and expected output checksums for every active OCSF mapping, source-extension, non-authority, and endpoint-direction row before this matrix can pass acceptance. Until those catalog paths and bytes exist, rows with `TODO` checksums remain `blocking` and the corresponding production artifact must remain inactive.

### GraphActiveProfileClosureValidationMatrix

Rows in this matrix validate behavior owned by `090` and cross-owner handoffs required by `040`, `050`, `060`, `070`, `080`, `110`, and `130`. Because fixture bytes and expected output checksums are absent, checksum cells are explicit blockers until populated by validation artifacts. `AcceptanceReport` must fail while any non-deferred graph closure row has a `TODO:` checksum, status `blocked`, status `not_run`, or missing mutation-prohibition proof.

| Row ID | Owner | Fixture ID | Fixture checksum | Expected result | Expected output checksum | Mutation prohibition |
| --- | --- | --- | --- | --- | --- | --- |
| `val-090-mvp-active-profile-exact-edge-set` | `090` | `fixture-090-mvp-active-profile-exact-edge-set` | TODO: required before acceptance | Only `observed_connection` active; all other edge types rejected. | TODO: required before acceptance | no unauthorized graph mutation |
| `val-090-observed-connection-positive` | `090` | `fixture-090-observed-connection-positive` | TODO: required before acceptance | One observed-connection edge plus resolved canonical endpoint nodes. | TODO: required before acceptance | no backend ID, no reachability output |
| `val-090-observed-connection-missing-flow-role` | `090` | `fixture-090-observed-connection-missing-flow-role` | TODO: required before acceptance | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op. | TODO: required before acceptance | no edge, no expiry, no absence |
| `val-090-observed-connection-ambiguous-flow-role` | `090` | `fixture-090-observed-connection-ambiguous-flow-role` | TODO: required before acceptance | ambiguous or no-op owner diagnostic. | TODO: required before acceptance | no graph mutation |
| `val-090-ocsf-endpoint-order-no-direction` | `090`, `050` | `fixture-090-ocsf-endpoint-order-no-direction` | TODO: required before acceptance | no observed-connection edge from OCSF endpoint order. | TODO: required before acceptance | no graph mutation |
| `val-090-unresolved-endpoint-no-edge` | `090`, `070` | `fixture-090-unresolved-endpoint-no-edge` | TODO: required before acceptance | `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. | TODO: required before acceptance | no graph node, no edge, no identity mutation |
| `val-090-stale-edge-not-pathfinding-default` | `090`, `060`, `110` | `fixture-090-stale-edge-not-pathfinding-default` | TODO: required before acceptance | stale display only when eligibility permits; no pathfinding by default. | TODO: required before acceptance | no expiry or cleanup |
| `val-090-empty-traversal-classes` | `090`, `110` | `fixture-090-empty-traversal-classes` | TODO: required before acceptance | empty path result. | TODO: required before acceptance | no backend traversal mutation |
| `val-090-observed-connection-path-explicit` | `090`, `110` | `fixture-090-observed-connection-path-explicit` | TODO: required before acceptance | path query traverses only `observed_connection_path`. | TODO: required before acceptance | no reachability claim |
| `val-090-theoretical-reachability-prohibited` | `090`, `200` | `fixture-090-theoretical-reachability-edge` | TODO: required before acceptance | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op. | TODO: required before acceptance | no graph/gold/API output |
| `val-090-boolean-reachability-property-prohibited` | `090`, `110`, `200` | `fixture-090-boolean-reachability-property` | TODO: required before acceptance | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`. | TODO: required before acceptance | no graph property, no API claim |
| `val-090-generic-external-payload-not-pathfinding` | `090`, `130` | `fixture-090-generic-external-payload-not-pathfinding` | TODO: required before acceptance | payload is non-emitting or detail-only and not pathfinding-ready. | TODO: required before acceptance | no finding, no metric, no identity influence |
| `val-090-backend-taxonomy-unmapped` | `090` | `fixture-090-backend-taxonomy-unmapped` | TODO: required before acceptance | unmapped backend label/type/property rejected. | TODO: required before acceptance | no serving output |
| `val-090-query-candidate-limit` | `090`, `110` | `fixture-090-query-candidate-limit` | TODO: required before acceptance | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`. | TODO: required before acceptance | no partial output unless explicitly permitted |
| `val-090-page-token-expired` | `090`, `110` | `fixture-090-page-token-expired` | TODO: required before acceptance | `GRAPH_PAGE_TOKEN_EXPIRED`. | TODO: required before acceptance | no backend query execution |
| `val-090-page-token-invalid` | `090`, `110` | `fixture-090-page-token-invalid` | TODO: required before acceptance | `GRAPH_PAGE_TOKEN_INVALID`. | TODO: required before acceptance | no backend query execution and no existence leak |
| `val-090-apply-order-replay` | `090`, `040` | `fixture-090-apply-order-replay` | TODO: required before acceptance | byte-identical canonical delta ordering. | TODO: required before acceptance | no nondeterministic apply |
| `val-090-partial-apply-no-derived-view` | `090` | `fixture-090-partial-apply-no-derived-view` | TODO: required before acceptance | partial apply records evidence and does not advance derived view. | TODO: required before acceptance | no derived-view advancement |
| `val-090-rebuild-equivalence` | `090`, `080` | `fixture-090-rebuild-equivalence` | TODO: required before acceptance | rebuild equivalence includes profile, schema, eligibility, taxonomy, query, property, index, and output checksums. | TODO: required before acceptance | no rebuild promotion on mismatch |
| `val-090-source-kind-cleanup-cross-source-reject` | `090`, `060` | `fixture-090-source-kind-cleanup-cross-source-reject` | TODO: required before acceptance | cross-source cleanup rejected unless exact `060` cleanup authorization exists. | TODO: required before acceptance | no cross-source deletion |
| `val-040-graph-node-source-object-ref-canonical-entity` | `040` | `fixture-040-graph-node-source-object-ref-canonical-entity` | TODO: required before acceptance | valid node source ref. | TODO: required before acceptance | no graph apply before validation |
| `val-040-graph-edge-source-object-ref-gold-fact` | `040` | `fixture-040-graph-edge-source-object-ref-gold-fact` | TODO: required before acceptance | valid edge source ref. | TODO: required before acceptance | no graph apply before validation |
| `val-050-flow-role-explicit-positive` | `050` | `fixture-050-flow-role-explicit-positive` | TODO: required before acceptance | explicit source direction emits `FlowRoleEvidence`. | TODO: required before acceptance | no graph mutation in mapping |
| `val-060-missing-flow-role-evidence-unknown` | `060` | `fixture-060-missing-flow-role-evidence-unknown` | TODO: required before acceptance | unknown/no-op. | TODO: required before acceptance | no absence, expiry, cleanup, retraction, or watermark |
| `val-070-graph-endpoint-requires-identity-decision` | `070` | `fixture-070-graph-endpoint-requires-identity-decision` | TODO: required before acceptance | graph endpoint requires identity decision. | TODO: required before acceptance | no graph endpoint from selector-only evidence |
| `val-080-graph-delta-replay-field-handoff` | `080` | `fixture-080-graph-delta-replay-field-handoff` | TODO: required before acceptance | replay includes imported `090` graph fields. | TODO: required before acceptance | no replay output on mismatch |
| `val-110-graph-response-profile-context` | `110` | `fixture-110-graph-response-profile-context` | TODO: required before acceptance | graph response contains profile context. | TODO: required before acceptance | no backend ID leak |
| `val-130-rule-graph-compat-observed-connection` | `130` | `fixture-130-rule-graph-compat-observed-connection` | TODO: required before acceptance | analysis read-only compatibility passes for named observed-connection query. | TODO: required before acceptance | no mutation |

### SourceExtensionFieldRuleShapeValidationRows

| Fixture family | Required coverage | Expected result |
| --- | --- | --- |
| `source-extension-shape-valid-*` | Minimal valid `040.SourceExtensionFieldRuleShape`. | Valid canonical bytes and checksum. |
| `source-extension-shape-missing-field-*` | Each required field omitted. | `CORE_REQUIRED_FIELD_MISSING`; no activation. |
| `source-extension-shape-unknown-field-*` | Unknown field outside the shape. | `CORE_UNKNOWN_FIELD`; no activation. |
| `source-extension-shape-noncanonical-*` | Non-canonical ordering or sort keys. | Checksum mismatch or canonicalization failure. |
| `source-extension-shape-invalid-bounds-*` | Bounds exceed `040` primitive limits. | `CORE_FIELD_BOUNDS_INVALID`; no activation. |
| `source-extension-shape-checksum-replay-*` | Same rule shape input replayed. | Byte-identical checksum. |

### FeedCategoryClosureValidationMatrix

Feed-category validation must evaluate every active feed category across the five requested effects `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. Every category/effect cell must produce either `closed_active` or `closed_deterministically_blocked`; `blocked_missing_ref`, `blocked_validation`, `blocked_todo`, `blocked_checksum`, `blocked_package_set`, and `blocked_manifest` fail acceptance.

Each fixture family in this matrix applies to every active feed category in `020.LakehouseFeedCategoryClosureRequirementTable`. `future_reachability` must use deterministic block/no-op fixtures for MVP instead of positive reachability output.

| Fixture family | Required coverage | Expected result |
| --- | --- | --- |
| `feed-category-valid-complete-*` | One complete positive case per non-blocked active feed category and requested effect. | Complete or empty-complete only when exact rows and refs permit. |
| `feed-category-missing-profile-row-*` | Category has no active `LakehouseFeedCompletenessProfileRow` or `LakehouseFeedCategoryClosureRow`. | Owner-specific missing-row error; no effect. |
| `feed-category-missing-upstream-completeness-*` | Required upstream evidence omitted, stale, insufficient, permission-limited, or scope-limited. | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` or `not_authoritative_for_absence`; no effect. |
| `feed-category-partial-known-gap-*` | Known missing object, partition, row range, or scope. | `partial_known_gap`; no absence, cleanup, retraction, graph expiry, or watermark. |
| `feed-category-partial-unknown-gap-*` | Missing scope cannot be identified. | `partial_unknown_gap`; no absence-sensitive effect. |
| `feed-category-permission-limited-*` | Supplier or source visibility is permission-limited. | `permission_limited`; no negative fact. |
| `feed-category-stale-*` | Source or evidence is stale under active staleness policy. | `source_stale` or blocking reason; no effect unless exact row permits. |
| `feed-category-empty-complete-*` | Empty scope or empty complete read with exact upstream evidence. | Authorized effect only through exact `060` row; otherwise empty-scope block. |
| `feed-category-weak-progress-*` | Cursor, ack, heartbeat, freshness, provenance, graph, or destination cleanup signal without active policy. | Weak-signal no-authority error or unknown no-op. |
| `feed-category-watermark-blocked-*` | Watermark requested while completeness gate blocks effect. | `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED`; no watermark advancement. |

| Feed category | Required fixture status |
| --- | --- |
| `endpoint_inventory` | Positive and negative closure fixtures required. |
| `configuration_inventory` | Positive and negative closure fixtures required. |
| `vulnerability_scan` | Positive and negative closure fixtures required, including scanner coverage dimensions. |
| `control_evaluation` | Positive and negative closure fixtures required, including control-result mapping. |
| `directory_inventory` | Positive and negative closure fixtures required, including hidden-object permission. |
| `directory_membership` | Positive and negative closure fixtures required, including direct/transitive mode and AD primary-group handling. |
| `dns_record_set` | Positive and negative closure fixtures required, including TTL non-authority. |
| `dhcp_ipam_assignment` | Positive and negative closure fixtures required, including lease expiry non-authority. |
| `network_flow` | Positive observed-flow and missing-flow-unknown fixtures required; absence fixture must block. |
| `cloud_asset_inventory` | Positive and negative closure fixtures required, including permission-limited inventory. |
| `source_history` | Positive and negative closure fixtures required, including outside-window no-proof. |
| `future_reachability` | MVP block/no-op fixture required; positive reachability output forbidden. |

### TemporalCorrectionReplayValidationMatrix

Rows in this matrix are required when `080` behavior is in implementation scope. Each row must name fixture checksum, input checksum, expected output checksum or no-op, expected error when applicable, mutation-prohibition proof, and `VersionManifest` ref.

| Fixture family | Required coverage |
| --- | --- |
| `temporal-resolution-*` | exact policy row, missing policy, absent time, malformed time, ambiguous time, unauthorized source time, authorized fallback, forbidden current-time fallback, manifest omission. |
| `knowledge-time-import-*` | `current_import`, historical valid-time-only import, reconstruction success with persisted source-known-time evidence, reconstruction rejected without evidence. |
| `late-arrival-*` | on-time authoritative, late authoritative correction, late non-authoritative quarantine, discard forbidden, watermark no-effect default. |
| `gold-correction-*` | replacement, retraction, interval split, stale state, conflict mark, conflict resolution, duplicate no-op, confidence-only same value, confidence-only changed value. |
| `assertion-transition-*` | Every default assertion-state and correction-event pair defined by `080`. |
| `correction-snapshot-ref-*` | old snapshot missing, new snapshot missing, table-set checksum missing, retention ineligible, mutable ref rejected. |
| `gold-correction-no-op-error-*` | missing temporal resolution, missing authority, ambiguous authority, missing correction policy, missing transition row, unauthorized negative, unauthorized CDC tombstone, replay output-class missing. |
| `replay-equivalence-*` | output-class rows for raw, silver, identity, gold, gold correction, graph delta, graph apply, graph rebuild, API response, export projection, analysis output, and validation acceptance; checksum match and mismatch. |
| `graph-handoff-*` | `none`, `reproject_fact_key`, authorized expiry, authorized cleanup, denied expiry, conflict visibility default, identity split handoff. |
| `temporal-version-manifest-*` | missing temporal resolution ref, missing correction snapshot refs, missing replay sufficiency check, missing graph handoff refs, checksum mismatch. |

| Validation row | Owner | Required assertion |
| --- | --- | --- |
| `120-TEMPORAL-CORRECTION-MISSING-POLICY` | `080` | Missing temporal policy emits `TEMPORAL_POLICY_UNRESOLVED` and no gold output. |
| `120-TEMPORAL-CORRECTION-CURRENT-TIME` | `080` | Current platform time fallback is rejected. |
| `120-ASSERTION-TRANSITION-TOTALITY` | `080` | Every assertion-state/correction-event pair has one expected result. |
| `120-REPLAY-OUTPUT-CLASS-COVERAGE` | `080` | Replay rows exist for every required output class. |
| `120-GRAPH-HANDOFF-COVERAGE` | `080`, `090` | Every graph handoff effect has success, error, or no-op validation. |
| `120-NOOP-ERROR-COVERAGE` | `080`, `060`, `090` | No-op and deterministic error cases prove no forbidden mutation. |
| `120-PROJECTION-REPLAY-COVERAGE` | `050`, `080` | Projection replay exact match, profile mismatch, loss-manifest mismatch, redaction mismatch, and volatile-field-only difference are validated. |
| `120-ANALYSIS-REPLAY-COVERAGE` | `130`, `080` | Analysis replay exact match, rule mismatch, graph compatibility mismatch, derived-view mismatch, authorization mismatch, shadow-only result, and mutation attempt are validated. |
| `120-API-TEMPORAL-ERROR-COVERAGE` | `110`, `080` | Every new `080` error is registered and label behavior is non-collapsing. |

### RunLockLeaseRecoveryValidationMatrix

This matrix is the executable validation interface for `030` run-lock stale recovery and lifecycle timing closure. A row may pass only when fixture bytes, input artifact refs, expected error or output, mutation prohibition, and version-manifest requirements are present and checksum-valid. A row with blocked, not-run, failed, stale, checksum-mismatched, or `TODO` evidence blocks aggregate acceptance.

| Validation row ID | Fixture | Expected result |
| --- | --- | --- |
| `val-030-runlock-lease-defaults` | Omitted lease fields | Defaults materialize to `300/60/30/15/30`; checksum stable. |
| `val-030-runlock-timing-bounds` | Invalid TTL, heartbeat, grace, timeout, or commit guard | `RUN_LOCK_TIMING_INVALID`; no lock mutation. |
| `val-030-runlock-all-or-nothing` | One key conflicts during multi-key acquisition | Partial keys released; `RUN_LOCK_ACQUISITION_FAILED`; no output. |
| `val-030-runlock-conflict-active` | Existing unexpired lock | `RUN_LOCK_CONFLICT`; no output. |
| `val-030-runlock-heartbeat-refresh` | Valid heartbeat before expiry | Lease extends; sequence increments; evidence checksum stable. |
| `val-030-runlock-heartbeat-timeout` | Heartbeat timeout | Output blocked until assertion succeeds. |
| `val-030-runlock-lost-before-commit` | Token changed before output commit | `RUN_LOCK_LOST` or `RUN_LOCK_FENCING_TOKEN_STALE`; no output. |
| `val-030-runlock-stale-recovery-success` | Expired prior lease plus grace and successful CAS | Higher token assigned; recovery evidence persisted. |
| `val-030-runlock-stale-recovery-race` | Prior owner heartbeats after contender reads | `RUN_LOCK_STALE_RECOVERY_FAILED`; no output. |
| `val-030-runlock-old-holder-fenced` | Old holder wakes after recovery | `RUN_LOCK_FENCING_TOKEN_STALE`; no downstream stage. |
| `val-030-runlock-idempotent-retry` | Same key and same input checksum | Byte-identical evidence; no duplicate mutation. |
| `val-030-runlock-idempotency-conflict` | Same key and different input checksum | `RUN_LOCK_IDEMPOTENCY_CONFLICT`; no side effects. |
| `val-030-runlock-commit-guard-missing` | Output commit without guard | `RUN_LOCK_COMMIT_GUARD_MISSING`; no output. |
| `val-030-runlock-manifest-completeness` | Lock evidence omitted from manifest | `VERSION_MANIFEST_INCOMPLETE`; no production output. |
| `val-030-runlock-watermark-blocked` | Lock loss before watermark | No `WatermarkCommitRecord` advancement. |
| `val-090-runlock-graph-apply-resume` | Lock loss after partial graph apply | Retry resumes only with committed-batch evidence and valid new guard. |
| `val-100-runlock-package-activation-loss` | Lock loss during activation | Current active package set preserved; no candidate output. |
| `val-020-runlock-maintenance-loss` | Lock loss before destructive maintenance | No destructive maintenance commit. |
| `val-060-runlock-signal-nonauthority` | Lease expiry or heartbeat signal used for absence | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY`; no absence-sensitive effect. |
| `val-140-runlock-telemetry-nonauthority` | Lock metric used as authority | Telemetry no-op or telemetry error; no domain mutation. |

### SourceAuthorityClosureValidationMatrix

The source-authority closure matrix must include row-chain fixture families for `authority`, `completeness`, `coverage`, `staleness`, `progress_signal`, `visibility`, `control_result`, `source_history`, `external_schema_authority_signal`, `absence_policy`, `watermark_policy`, and `source_dataset_catalog`.

Weak-signal fixture families must cover `cursor_exhaustion`, `delta_token_complete`, `cdc_offset`, `cdc_heartbeat`, `queue_drain`, `end_to_end_ack`, `provenance_closure`, `freshness_artifact`, `graph_apply_success`, `graph_index_state`, `graph_drift_check`, `destination_cleanup`, `source_history_no_result`, `live_source_probe_zero_rows`, and `telemetry_metric`.

Every fixture family in this matrix is required before `SourceAuthorityClosureMatrix` may satisfy promotion for an active absence-sensitive feed category. `120` validates owner behavior; it must not define source-authority behavior beyond validation artifact shapes, fixture IDs, expected outputs, and acceptance aggregation.

| Fixture family | Required cases |
| --- | --- |
| `source-authority-row-resolution-*` | exact match success, missing row, ambiguous row, dataset-default allowed, dataset-default forbidden, source-instance override exact match, source-instance mismatch |
| `lakehouse-feed-completeness-totality-*` | all six `020` receipt states crossed with all eight upstream evidence states for every active absence-sensitive feed category, `source_dataset`, requested effect, scope selector, read target kind, and upstream evidence class tuple |
| `coverage-dimension-*` | missing source-specific coverage row, missing required dimension, permission-limited dimension, partial known gap, stale coverage, covered success |
| `coverage-domain-token-*` | positive canonical tokens, display/legacy alias rejection, unknown syntactically valid token rejection, invalid grammar rejection, duplicate array rejection, lexical sort checksum, unsupported feed-category token, required-for-effect omission, and inactive `reachability` deterministic block |
| `staleness-policy-*` | declared time precedence, missing time input, malformed time input, TTL expiry, DHCP lease expiry, source-history outside-window, no current-time fallback |
| `control-result-mapping-*` | pass, fail, unknown, error, not evaluated, not checked, not applicable, unmapped external state |
| `progress-signal-*` | cursor exhaustion alone, ack success alone, queue drain alone, CDC heartbeat alone, graph apply success alone, destination cleanup alone, weak-signal combination blocked, exact combined policy success |
| `dns-dhcp-flow-*` | DNS TTL expiry not deletion, DHCP lease expiry not host absence, missing flow unknown |
| `directory-visibility-*` | hidden membership, limited-information rows, direct-only membership query, AD primary-group gap, delta reset, page incompletion |
| `manifest-closure-*` | omitted authority row set ref, omitted staleness ref, omitted coverage ref, omitted progress policy, omitted control mapping, omitted source-history retention, checksum mismatch |
| `graph-expiry-*` | expiry authorized success, expiry missing `060` effect ref rejected, derived-view stale does not expire graph object |
| `feed-category-closure-catalog-*` | one active row per known category, duplicate row rejection, missing known category row, future category block row, source_dataset allowlist missing, source_dataset allowlist ambiguous |
| `source-authority-closure-matrix-rowset-*` | closure row exact match, deterministic block row, missing row, ambiguous row, inactive row, checksum mismatch, out-of-scope row, stale validation ref, TODO row |
| `source-history-coverage-*` | retention present but coverage missing, coverage present but retention missing, outside-window no-proof, inside-window no-change proof success |
| `external-schema-authority-signal-*` | exact signal row success, missing signal row, ambiguous signal row, field-presence non-authority, endpoint-order non-authority |
| `deterministic-block-row-*` | blocked category, blocked dataset, blocked predicate, blocked effect, mutation-prohibition proof |
| `version-manifest-source-closure-*` | omitted closure row-set ref, omitted underlying authority ref, omitted coverage ref, omitted staleness ref, omitted validation ref, closure summary present without underlying refs |
| `source-closure-private-binding-*` | product name, tenant ID, private route, scanner site, host list, zone inventory, account list, and credential leak rejected |

A validation row in this matrix must include fixture checksum, expected `AbsenceDerivationResult` checksum when applicable, expected `VersionManifest` checksum or expected manifest error, expected output checksum when output is allowed, expected no-op when output is blocked, expected error code when rejected, and mutation-prohibition proof for raw, silver, identity, gold, graph, watermark, compliance export, and API label mutation classes affected by the case.

### SourceClosureCategoryEffectValidationMatrix

This matrix imports the MVP category set from `020.LakehouseFeedCategoryClosureRequirementTable`. Every category must have closure-positive fixtures for its active effects or deterministic-block fixtures for blocked effects.

| Feed category | Required validation posture |
| --- | --- |
| `endpoint_inventory` | Positive closure or deterministic block for each active effect. |
| `configuration_inventory` | Positive closure or deterministic block for each active effect. |
| `vulnerability_scan` | Positive closure or deterministic block for fixed-state, absence, and watermark cases. |
| `control_evaluation` | Control-result mapping and non-pass/fail defaults. |
| `directory_inventory` | Visibility and permission-limited absence cases. |
| `directory_membership` | Hidden membership, direct/transitive mode, and AD primary-group cases. |
| `dns_record_set` | TTL stale/unknown and authorized narrow effect cases. |
| `dhcp_ipam_assignment` | Lease expiry without host absence and authorized narrow effect cases. |
| `network_flow` | Positive observed-flow, missing-flow unknown, and blocked absence cases. |
| `cloud_asset_inventory` | Visibility, disappearance, history, cleanup, and graph-expiry cases. |
| `source_history` | Retention plus coverage success, retention-only failure, coverage-only failure, and outside-window no-proof. |
| `future_reachability` | Deterministic block only; no MVP runtime effect. |

### StructuredInputRepositoryValidationMatrix

Rows in this matrix validate behavior owned by `010`, `030`, `040`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, and `140`. Because fixture bytes and expected output checksums are absent, checksum cells are explicit blockers until populated by validation artifacts. `AcceptanceReport` must fail while any row is `blocked`, `not_run`, `fail`, stale, checksum-mismatched, or `TODO`.

| validation_row_id | owner_spec | fixture_id | fixture_checksum | required_refs | expected_error_or_output | expected_output_checksum | mutation_prohibition | acceptance_criterion | blocking_status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `val-structured-input-repository-profile` | `030`, `110` | `fixture-structured-input-profile` | TODO | repository profile, access policy, redaction policy | profile validation result | TODO | no activation | `030-STRUCTURED-INPUT-SNAPSHOT-AC-001` | blocked |
| `val-structured-input-path-normalization` | `030` | `fixture-structured-input-invalid-paths` | TODO | repository profile | `STRUCTURED_INPUT_PATH_INVALID` | TODO | no validation output, materialization, or package release | `030-STRUCTURED-INPUT-SNAPSHOT-AC-001` | blocked |
| `val-structured-input-snapshot-determinism` | `030` | `fixture-structured-input-snapshot-repeat` | TODO | repository profile, exact commit, tree, file manifest | byte-identical snapshot IDs | TODO | no mutable ref authority | `030-STRUCTURED-INPUT-SNAPSHOT-AC-001` | blocked |
| `val-structured-input-mutable-ref-rejection` | `010`, `030`, `100` | `fixture-structured-input-mutable-ref` | TODO | branch, tag, PR ref, repository URL cases | `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | TODO | no activation, rollback, or manifest satisfaction | `030-STRUCTURED-INPUT-MUTABLE-REF-AC-001` | blocked |
| `val-structured-input-force-push-revalidation` | `030` | `fixture-structured-input-ref-rewrite` | TODO | prior validation run, rewritten ref observation | `STRUCTURED_INPUT_REVALIDATION_REQUIRED` | TODO | no materialization from stale validation | `030-STRUCTURED-INPUT-FORCE-PUSH-AC-001` | blocked |
| `val-structured-input-private-binding-redaction` | `010`, `110`, `140` | `fixture-structured-input-private-leak` | TODO | redaction policy, access policy | `STRUCTURED_INPUT_PRIVATE_BINDING_LEAK` or redacted response | TODO | no private leak | `110-STRUCTURED-INPUT-REDACTION-AC-001` | blocked |
| `val-structured-input-validation-run-exact-snapshot` | `120`, `030` | `fixture-structured-input-stale-validation` | TODO | validation run, snapshot checksum | `STRUCTURED_INPUT_VALIDATION_STALE` | TODO | no activation or materialization | `120-STRUCTURED-INPUT-AC-001` | blocked |
| `val-structured-input-materialization` | `100` | `fixture-structured-input-materialization` | TODO | snapshot, validation run, materialization result | materialization result | TODO | no production activation | `100-STRUCTURED-INPUT-MATERIALIZATION-AC-001` | blocked |
| `val-structured-input-package-release-handoff` | `100` | `fixture-structured-input-package-release` | TODO | materialization result, release manifest | package release manifest | TODO | no direct Git activation | `100-STRUCTURED-INPUT-GIT-AUTHORITY-AC-001` | blocked |
| `val-structured-input-package-set-activation` | `100`, `030` | `fixture-structured-input-package-set` | TODO | package release, package set, version manifest | activated package set or activation failure | TODO | current active set preserved on failure | `100-STRUCTURED-INPUT-MATERIALIZATION-AC-001` | blocked |
| `val-structured-input-rollback-mutable-ref-rejection` | `100` | `fixture-structured-input-rollback-branch` | TODO | rollback plan | `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | TODO | no active state change | `100-STRUCTURED-INPUT-ROLLBACK-AC-001` | blocked |
| `val-structured-input-version-manifest` | `030` | `fixture-structured-input-manifest-omission` | TODO | omitted structured-input refs | `VERSION_MANIFEST_INCOMPLETE` | TODO | no output | `030-STRUCTURED-INPUT-VM-AC-001` | blocked |
| `val-structured-input-error-registry` | `110` | `fixture-structured-input-error-registry` | TODO | owner fragments | generated error registry | TODO | no unredacted error context | `110-STRUCTURED-INPUT-ERROR-AC-001` | blocked |
| `val-structured-input-audit` | `110` | `fixture-structured-input-audit` | TODO | access policy, audit schema | audit event | TODO | no raw repository content | `110-STRUCTURED-INPUT-AUDIT-AC-001` | blocked |
| `val-structured-input-telemetry-redaction` | `140`, `110` | `fixture-structured-input-telemetry` | TODO | telemetry profile, attribute policy, redaction policy | redacted telemetry or rejection | TODO | no domain mutation and no private leak | `140-STRUCTURED-INPUT-REDACTION-AC-001` | blocked |

### Structured input validation coverage rows

| Owner spec | Required structured-input coverage |
| --- | --- |
| `030` | profile, snapshot determinism, path normalization, mutable ref rejection, force-push revalidation, manifest omission |
| `050` | mapping snapshot handoff, stale validation, path escape, no activation on merge |
| `060` | inactive source-authority row catalog, private leak, stale validation, closure success, manifest omission |
| `070` | Git-only profile no-op, stale validation, hard-blocker weakening rejection, private leak, manifest completeness |
| `080` | inactive temporal policy no-op, stale validation, replay manifest omission |
| `090` | Git-only graph profile no-op, provider-native query rejection, package-gate failure, stale validation, manifest omission |
| `100` | materialization success, direct Git activation rejection, materialization mismatch, package type mismatch, package-set ref omission, rollback mutable-ref rejection |
| `110` | authorization denial without existence leak, redacted validation diagnostics, private path leak rejection, mutable-ref error rendering, audit completeness |
| `130` | Git-only registry no-op, authority substitution rejection, package-set ref omission, redaction |
| `140` | private route leak, unbounded file path labels, branch-name redaction, commit SHA allowed and forbidden cases, no domain mutation |

### PackageActivationValidationMatrix

Package validation rows for `source_dataset_catalog_row_set` must cover unknown token rejection, policy row omission, package-set omission, checksum mismatch, deterministic block package row, source-dataset private leak, and manifest inclusion.

Every row in this matrix validates behavior owned by `100`, manifest inclusion owned by `030`, observable error handling owned by `110`, or acceptance aggregation owned by `120`. Rows with `TODO` fixture checksums or expected output checksums are blocking and must prevent authoritative package activation handoff.

| Row class | Required coverage |
| --- | --- |
| `100-package-type-policy` | Confirmed enum success, duplicate token rejection, generic-label rejection, legacy-label rejection, unknown token rejection, missing policy, ambiguous policy, inactive policy, scope-mismatched policy, manifest-omitted policy rows, exact policy success, and coverage totality. |
| `100-repository-form` | Supported repository forms and `git_tree_snapshot` rejection. |
| `100-trust` | Unauthorized signer, inactive trust root, threshold failure, transparency evidence missing. |
| `100-attestation-sbom` | Missing, mismatched, expired, and policy-failed attestation and SBOM refs. |
| `100-dependency-lock` | Missing lock, checksum mismatch, live dependency resolution attempt. |
| `100-compatibility` | Missing, failed, expired, and mismatched compatibility rows across required axes. |
| `100-deprecation` | In-window use, expired use, emergency extension, and forbidden new activation. |
| `100-health-lkg` | Health pass marks LKG, health fail does not. |
| `100-rollback` | Mutable target rejection, immutable target pass, incompatible state/schema/graph/trust/replay rejection. |
| `100-quarantine` | Artifact, release, set, signer, trust root, snapshot, namespace, and package type quarantine blocking. |
| `100-emergency` | Allowed bounded override actions and forbidden trust bypass. |
| `100-version-manifest` | Omitted package activation refs, selected policy row refs, deprecation row refs, compatibility refs, release checksums, and package-set checksums reject output. |
| `100-policy-bundle` | `policy_bundle` grouping succeeds only as a carrier and fails when used as row-catalog substitution. |
| `100-manifest-field-omission` | Required `PackageReleaseManifest` and `ProductionPackageSetManifest` field omissions reject before candidate output. |
| `100-error-registry-parity` | `100.PackageErrorRegistryFragment` and `110.PackageActivationErrorObservableMapping` have identical package error code sets and compatible severity, retryability, redaction, and fixture refs. |

A `PackageActivationValidationMatrix` row must include fixture checksum, expected output checksum or expected no-op, expected error when rejected, mutation prohibition, owner spec, acceptance criterion, and `VersionManifest` requirement. The row must identify the package activation ref classes required by `030.VersionManifestCompletenessMatrix` and the observable error mapping required by `110.PackageActivationErrorObservableMapping`.

Required `100` fixture rows:

| Row ID | Fixture ID | Fixture checksum | Expected error or no-op | Expected output checksum | Mutation prohibition | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `val-100-package-type-unknown` | `fixture-100-package-type-unknown` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-002` | blocking |
| `val-100-package-type-policy-missing` | `fixture-100-package-type-policy-missing` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `val-100-package-type-policy-ambiguous` | `fixture-100-package-type-policy-ambiguous` | TODO | `PACKAGE_TYPE_POLICY_AMBIGUOUS` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-004` | blocking |
| `val-100-package-type-enum-closed` | `fixture-100-package-type-enum-closed` | TODO | none | TODO | no enum mutation | `100-PACKAGE-TYPE-ENUM-AC-001` | blocking |
| `val-100-package-type-duplicate-token-rejected` | `fixture-100-package-type-duplicate-token` | TODO | `PACKAGE_TYPE_UNKNOWN` or validation owner error | TODO | no package activation | `100-PACKAGE-TYPE-COVERAGE-AC-001` | blocking |
| `val-100-package-type-generic-label-rejected` | `fixture-100-package-type-generic-label` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-ENUM-AC-001` | blocking |
| `val-100-package-type-legacy-label-rejected` | `fixture-100-package-type-legacy-label` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-ENUM-AC-001` | blocking |
| `val-100-package-type-policy-exact-success` | `fixture-100-package-type-policy-exact-success` | TODO | none | TODO | selected row ref and checksum recorded only | `100-PACKAGE-TYPE-POLICY-AC-005` | blocking |
| `val-100-package-type-policy-inactive` | `fixture-100-package-type-policy-inactive` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `val-100-package-type-policy-scope-mismatch` | `fixture-100-package-type-policy-scope-mismatch` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `val-100-package-type-policy-manifest-omitted` | `fixture-100-package-type-policy-manifest-omitted` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `030-PACKAGE-POLICY-MANIFEST-AC-001` | blocking |
| `val-100-policy-bundle-substitution-forbidden` | `fixture-100-policy-bundle-substitution` | TODO | `PACKAGE_ACTIVATION_ARTIFACT_OWNER_MISMATCH` | TODO | current active set unchanged | `100-POLICY-BUNDLE-SUBSTITUTION-AC-001` | blocking |
| `val-100-package-type-coverage-totality` | `fixture-100-package-type-coverage-totality` | TODO | validation owner error | TODO | no package activation | `100-PACKAGE-TYPE-COVERAGE-AC-001` | blocking |
| `val-100-git-tree-snapshot-unsupported` | `fixture-100-git-tree-snapshot-unsupported` | TODO | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | TODO | no candidate production output | `100-REPOSITORY-FORM-AC-002` | blocking |
| `val-100-package-set-checksum-mismatch` | `fixture-100-package-set-checksum-mismatch` | TODO | `PACKAGE_SET_CHECKSUM_MISMATCH` | TODO | current active set unchanged | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `val-100-unauthorized-signer` | `fixture-100-unauthorized-signer` | TODO | `PACKAGE_SIGNER_UNAUTHORIZED` | TODO | no package activation | `100-CLEANUP-AC-003` | blocking |
| `val-100-transparency-evidence-missing` | `fixture-100-transparency-evidence-missing` | TODO | `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | TODO | no package activation | `100-TRANSPARENCY-AC-001` | blocking |
| `val-100-attestation-missing` | `fixture-100-attestation-missing` | TODO | `PACKAGE_ATTESTATION_MISSING` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `val-100-attestation-subject-mismatch` | `fixture-100-attestation-subject-mismatch` | TODO | `PACKAGE_ATTESTATION_SUBJECT_MISMATCH` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `val-100-sbom-missing` | `fixture-100-sbom-missing` | TODO | `PACKAGE_SBOM_MISSING` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `val-100-sbom-subject-mismatch` | `fixture-100-sbom-subject-mismatch` | TODO | `PACKAGE_SBOM_SUBJECT_MISMATCH` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `val-100-dependency-lock-missing` | `fixture-100-dependency-lock-missing` | TODO | `PACKAGE_DEPENDENCY_LOCK_MISSING` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `val-100-dependency-live-resolution-forbidden` | `fixture-100-dependency-live-resolution-forbidden` | TODO | `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `val-100-compatibility-failed` | `fixture-100-compatibility-failed` | TODO | `PACKAGE_COMPATIBILITY_FAILED` | TODO | no package activation | `100-COMPATIBILITY-AC-001` | blocking |
| `val-100-deprecation-policy-missing` | `fixture-100-deprecation-policy-missing` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-DEPRECATION-WINDOW-AC-001` | blocking |
| `val-100-deprecation-policy-inactive` | `fixture-100-deprecation-policy-inactive` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-DEPRECATION-WINDOW-AC-001` | blocking |
| `val-100-deprecation-policy-scope-mismatch` | `fixture-100-deprecation-policy-scope-mismatch` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-DEPRECATION-WINDOW-AC-001` | blocking |
| `val-100-deprecation-window-expired` | `fixture-100-deprecation-window-expired` | TODO | `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | TODO | no new activation | `100-DEPRECATION-WINDOW-AC-002` | blocking |
| `val-100-lkg-health-gate-failed` | `fixture-100-lkg-health-gate-failed` | TODO | `PACKAGE_LKG_HEALTH_GATE_FAILED` | TODO | no last-known-good mark | `100-LKG-HEALTH-AC-002` | blocking |
| `val-100-mutable-rollback-target` | `fixture-100-mutable-rollback-target` | TODO | `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | TODO | no package activation | `100-ROLLBACK-AC-001` | blocking |
| `val-100-rollback-state-incompatible` | `fixture-100-rollback-state-incompatible` | TODO | `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `val-100-rollback-graph-incompatible` | `fixture-100-rollback-graph-incompatible` | TODO | `PACKAGE_ROLLBACK_GRAPH_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `val-100-quarantine-artifact-blocks-release` | `fixture-100-quarantine-artifact-blocks-release` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent activation | `100-QUARANTINE-AC-001` | blocking |
| `val-100-quarantine-signer-blocks-activation` | `fixture-100-quarantine-signer-blocks-activation` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent activation | `100-QUARANTINE-AC-001` | blocking |
| `val-100-quarantine-clear-not-active` | `fixture-100-quarantine-clear-not-active` | TODO | no-op | TODO | no direct active transition | `100-QUARANTINE-AC-002` | blocking |
| `val-100-emergency-bypass-forbidden` | `fixture-100-emergency-bypass-forbidden` | TODO | `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | TODO | no package activation | `100-EMERGENCY-AC-001` | blocking |
| `val-100-version-manifest-package-refs-missing` | `fixture-100-version-manifest-package-refs-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `100-VERSION-MANIFEST-AC-001` | blocking |
| `val-100-version-manifest-package-policy-row-missing` | `fixture-100-version-manifest-package-policy-row-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `030-PACKAGE-POLICY-MANIFEST-AC-001` | blocking |
| `val-100-version-manifest-deprecation-row-missing` | `fixture-100-version-manifest-deprecation-row-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `030-PACKAGE-POLICY-MANIFEST-AC-001` | blocking |
| `val-100-canary-output-isolation` | `fixture-100-canary-output-isolation` | TODO | `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | TODO | no current production output | `100-LIFECYCLE-AC-003` | blocking |
| `val-100-shadow-output-isolation` | `fixture-100-shadow-output-isolation` | TODO | `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | TODO | no current production output | `100-LIFECYCLE-AC-003` | blocking |

### PackageManifestFieldOmissionValidationMatrix

`RunValidationMatrix` must materialize one blocking row for each field listed below. Each row must include fixture ID, non-`TODO` fixture checksum, expected output or expected no-op/error, non-`TODO` expected output checksum, mutation prohibition, `VersionManifest` requirement, and acceptance criterion before authoritative handoff. Until fixture bytes exist, every row remains `blocking`.

| Manifest field path | Validation row ID | Expected error | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- |
| `PackageReleaseManifest.package_type` | `val-100-release-manifest-package-type-omitted` | `PACKAGE_TYPE_UNKNOWN` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.artifact_digest` | `val-100-release-manifest-artifact-digest-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.artifact_size_bytes` | `val-100-release-manifest-artifact-size-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.media_type` | `val-100-release-manifest-media-type-omitted` | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.subject_digest` | `val-100-release-manifest-subject-digest-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.repository_form` | `val-100-release-manifest-repository-form-omitted` | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.signature_verification_result_refs` | `val-100-release-manifest-signature-refs-omitted` | `PACKAGE_SIGNATURE_THRESHOLD_FAILED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.trust_policy_refs` | `val-100-release-manifest-trust-refs-omitted` | `PACKAGE_TRUST_ROOT_INACTIVE` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.sbom_refs` when required | `val-100-release-manifest-sbom-refs-omitted` | `PACKAGE_SBOM_MISSING` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.build_provenance_refs` when required | `val-100-release-manifest-provenance-refs-omitted` | `PACKAGE_PROVENANCE_POLICY_FAILED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.dependency_lock_refs` when required | `val-100-release-manifest-dependency-lock-refs-omitted` | `PACKAGE_DEPENDENCY_LOCK_MISSING` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.compatibility_matrix_refs` | `val-100-release-manifest-compatibility-refs-omitted` | `PACKAGE_COMPATIBILITY_FAILED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.validation_refs` | `val-100-release-manifest-validation-refs-omitted` | `PACKAGE_VALIDATION_FAILED` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.release_checksum` | `val-100-release-manifest-release-checksum-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.activation_scope` | `val-100-release-manifest-activation-scope-omitted` | shared selector validation error mapped to `PACKAGE_ACTIVATION_ARTIFACT_SCOPE_MISMATCH` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `PackageReleaseManifest.lifecycle_status` | `val-100-release-manifest-lifecycle-status-omitted` | `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | `100-PACKAGE-RELEASE-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.target_environment` | `val-100-package-set-target-environment-omitted` | shared selector validation before owner-specific `PACKAGE_TYPE_POLICY_MISSING` when applicable | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.activation_mode` | `val-100-package-set-activation-mode-omitted` | `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.package_release_refs` | `val-100-package-set-release-refs-omitted` | `PACKAGE_COHESION_INCOMPLETE` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.release_manifest_checksums` | `val-100-package-set-release-checksums-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.package_type_policy_row_set_ref` | `val-100-package-set-policy-row-set-omitted` | `PACKAGE_TYPE_POLICY_MISSING` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.package_type_policy_refs` | `val-100-package-set-policy-row-refs-omitted` | `PACKAGE_TYPE_POLICY_MISSING` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.trust_refs` | `val-100-package-set-trust-refs-omitted` | `PACKAGE_TRUST_ROOT_INACTIVE` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.compatibility_refs` | `val-100-package-set-compatibility-refs-omitted` | `PACKAGE_COMPATIBILITY_FAILED` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.rollback_refs` | `val-100-package-set-rollback-refs-omitted` | `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.quarantine_refs` | `val-100-package-set-quarantine-refs-omitted` | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.health_gate_refs` | `val-100-package-set-health-refs-omitted` | `PACKAGE_LKG_HEALTH_GATE_FAILED` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.lifecycle_transition_evidence_refs` | `val-100-package-set-lifecycle-evidence-omitted` | `PACKAGE_LIFECYCLE_ILLEGAL_TRANSITION` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.approval_refs` | `val-100-package-set-approval-refs-omitted` | `PACKAGE_VALIDATION_FAILED` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |
| `ProductionPackageSetManifest.package_set_checksum` | `val-100-package-set-checksum-omitted` | `PACKAGE_SET_CHECKSUM_MISMATCH` | `100-PACKAGE-SET-MANIFEST-AC-001` | blocking |

### TwoIndependentImplementersCheck

| Field | Required content |
| --- | --- |
| comparison scope | Every authoritative owner contract and validation row in the spec set. |
| allowed variance | Non-observable implementation structure only. |
| output equivalence | Byte-identical canonical records or deterministic equivalent observable outputs. |
| failure classification | spec ambiguity, implementation defect, fixture defect, or deferred issue. |
| acceptance threshold | All non-deferred authoritative rows pass. |
| core record fixture coverage | Raw, silver, canonical entity, source asset, identifier, gold, evidence, and graph delta fixtures exported by `040`. |

### RecreatabilityCheck

The recreatability check passes only when the intended behavior can be derived from active NLSpecs, accepted standards, and explicitly deferred documents where referenced.

Any required lookup outside those inputs is a failure unless the lookup is trace-only and does not determine behavior.

### AcceptanceReport statuses

| Status | Meaning | Failure precedence |
| --- | --- | --- |
| `fail` | Row ran and produced mismatched output, forbidden mutation, or wrong error. | Highest. |
| `blocked` | Row cannot run because required artifact, owner decision, fixture, or dependency is missing. | Second. |
| `not_run` | Row exists but has no execution evidence. | Third. |
| `pass` | Row ran and all expected evidence matched. | Lowest. |

Lifecycle transition evidence is required for acceptance status changes that affect promotion. A promotion-affecting `AcceptanceReport` status change without matching `120.ValidationAcceptanceLifecycleMachine.v1` transition evidence is invalid and must fail before promotion.

### Required negative tests by owner

| Owner spec | Forbidden boundary | Fixture ID | Fixture checksum | Expected error/no-op | Expected output checksum | Mutation prohibition | Acceptance criterion | Blocking status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `010` | direct source call | `fixture-010-direct-source-call` | TODO | `DIRECT_SOURCE_CALL_FORBIDDEN` | TODO | no production output | `010-CLEANUP-AC-002` | blocking |
| `010` | private binding leak | `fixture-010-private-binding-leak` | TODO | `PRIVATE_BINDING_LEAK` | TODO | no public artifact write | `010-CLEANUP-AC-003` | blocking |
| `010` | projection authority violation | `fixture-010-projection-authority-violation` | TODO | `PROJECTION_AUTHORITY_VIOLATION` | TODO | no authoritative mutation | `010-CLEANUP-AC-004` | blocking |
| `020` | CDC metadata as authority | `fixture-020-cdc-metadata-non-authority` | TODO | no-op | TODO | no absence, no retraction, no watermark | `020-CLEANUP-AC-004` | blocking |
| `020` | malformed feed manifest | `feed-020-malformed-manifest` | TODO | `manifest_invalid` | TODO | no raw import commit | `020-MANIFEST-AC-001` | blocking |
| `020` | missing feed profile field | `feed-020-profile-schema-incomplete` | TODO | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | TODO | no feed read | `020-FEED-CLOSURE-AC-001` | blocking |
| `020` | missing category closure row | `feed-020-category-row-missing` | TODO | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | TODO | no feed read | `020-FEED-CLOSURE-AC-002` | blocking |
| `020` | unresolved profile branch | `feed-020-profile-branch-unresolved` | TODO | `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | TODO | no production output | `020-FEED-CLOSURE-AC-003` | blocking |
| `020` | invalid empty-scope authorization | `feed-020-empty-scope-not-authoritative` | TODO | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | TODO | no absence and no watermark | `020-FEED-CLOSURE-AC-006` | blocking |
| `020` | declared subset missing | `feed-020-declared-subset-required` | TODO | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | TODO | no subset output | `020-FEED-CLOSURE-AC-005` | blocking |
| `030` | forbidden stage output | `fixture-030-forbidden-stage-output` | TODO | `FORBIDDEN_STAGE_OUTPUT` | TODO | no production output | `030-CLEANUP-AC-003` | blocking |
| `030` | duplicate stage ID | `fixture-030-duplicate-stage-id` | TODO | `DAG_STAGE_ID_DUPLICATE` | TODO | no production output | `030-DAG-ORDER-AC-001` | blocking |
| `030` | subset output forbidden | `fixture-030-subset-output-forbidden` | TODO | `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | TODO | no production output and no watermark | `030-FEED-CLOSURE-AC-002` | blocking |
| `040` | unknown field | `fixture-040-unknown-field` | TODO | `CORE_UNKNOWN_FIELD` | TODO | no record commit | `040-CLEANUP-AC-002` | blocking |
| `040` | one-of invalid | `fixture-040-one-of-invalid` | TODO | `CORE_ONE_OF_INVALID` | TODO | no record commit | `120-SCHEMA-PATCH-AC-001` | blocking |
| `050` | undeclared extension | `fixture-050-source-extension-undeclared` | TODO | `SOURCE_EXTENSION_FIELD_UNDECLARED` | TODO | no silver output | `050-CLEANUP-AC-002` | blocking |
| `050` | OCSF artifact mismatch | `fixture-050-ocsf-artifact-mismatch` | TODO | `OCSF_ARTIFACT_MISMATCH` | TODO | no normalized output | `050-OCSF-BASELINE-AC-001` | blocking |
| `050` | missing OCSF mapping row | `ocsf-mapping-row-missing-*` | TODO | `MAP_OCSF_ROW_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-001` | blocking |
| `050` | ambiguous OCSF mapping row | `ocsf-mapping-row-ambiguous-*` | TODO | `MAP_OCSF_ROW_AMBIGUOUS` | TODO | no silver output | `050-OCSF-MAP-AC-001` | blocking |
| `050` | missing OCSF activity discriminator | `ocsf-activity-discriminator-missing-*` | TODO | `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-005` | blocking |
| `050` | OCSF required object path missing | `ocsf-required-object-path-missing-*` | TODO | `OCSF_REQUIRED_OBJECT_PATH_MISSING` | TODO | no silver output | `050-OCSF-MAP-AC-002` | blocking |
| `050` | OCSF forbidden field emitted | `ocsf-forbidden-field-*` | TODO | `OCSF_FORBIDDEN_FIELD_EMITTED` | TODO | no silver output | `050-BASE-FIELD-AC-001` | blocking |
| `050` | DHCP/IPAM split required | `dhcp-ipam-split-required-*` | TODO | `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED` | TODO | no silver output | `050-OCSF-MAP-AC-006` | blocking |
| `050` | source-extension wildcard rejected | `source-extension-wildcard-rejected-*` | TODO | `SOURCE_EXTENSION_NAMESPACE_INVALID` | TODO | no silver output | `050-SOURCE-EXT-AC-003` | blocking |
| `050` | source-extension secret scan | `source-extension-secret-scan-*` | TODO | `SOURCE_EXTENSION_SECRET_SCAN_FAILED` | TODO | no caller-visible secret output | `050-SOURCE-EXTENSION-AC-001` | blocking |
| `060` | weak progress absence | `fixture-060-weak-progress-absence` | TODO | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` or unknown no-op | TODO | no absence, no cleanup, no watermark | `060-CLEANUP-AC-003` | blocking |
| `060` | missing coverage | `fixture-060-coverage-required` | TODO | `COVERAGE_ASSERTION_REQUIRED` | TODO | no coverage-sensitive fact | `060-COVERAGE-CLOSURE-AC-001` | blocking |
| `060` | missing completeness profile row | `feed-category-missing-profile-row-*` | TODO | `LAKEHOUSE_FEED_COMPLETENESS_PROFILE_ROW_MISSING` | TODO | no absence-sensitive effect | `060-FEED-CLOSURE-AC-001` | blocking |
| `060` | missing upstream completeness evidence | `feed-category-missing-upstream-completeness-*` | TODO | `UPSTREAM_COMPLETENESS_EVIDENCE_INSUFFICIENT` | TODO | no absence-sensitive effect | `060-FEED-CLOSURE-AC-002` | blocking |
| `060` | effect not allowed | `fixture-060-effect-not-allowed` | TODO | `COMPLETENESS_EFFECT_NOT_ALLOWED` | TODO | no absence, cleanup, retraction, graph expiry, or watermark | `060-FEED-CLOSURE-AC-003` | blocking |
| `060` | weak-signal combination | `feed-category-weak-progress-*` | TODO | `WEAK_PROGRESS_SIGNAL_NO_AUTHORITY` or unknown no-op | TODO | no stronger authority | `060-CLEANUP-AC-003` | blocking |
| `060` | watermark blocked by completeness | `feed-category-watermark-blocked-*` | TODO | `WATERMARK_ADVANCEMENT_COMPLETENESS_BLOCKED` | TODO | no watermark advancement | `060-FEED-CLOSURE-AC-004` | blocking |
| `060` | OCSF status as authority | `ocsf-status-non-authority-*` | TODO | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | TODO | no authoritative mutation | `060-OCSF-NONAUTH-AC-001` | blocking |
| `060` | OCSF field absence as absence | `ocsf-field-absence-non-authority-*` | TODO | `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` | TODO | no absence, cleanup, retraction, graph expiry, or watermark | `060-OCSF-NONAUTH-AC-003` | blocking |
| `070` | weak evidence auto-merge | `fixture-070-weak-evidence-auto-merge` | TODO | `no_decision` | TODO | no identity mutation | `070-CLEANUP-AC-002` | blocking |
| `070` | invalid review transition | `fixture-070-invalid-review-transition` | TODO | `IDENTITY_REVIEW_TRANSITION_INVALID` | TODO | no identity mutation | `070-REVIEW-TOTALITY-AC-001` | blocking |
| `070` | IP-only no mutation | `fixture-070-ip-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | hostname-only no mutation | `fixture-070-hostname-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | DNS-only no mutation | `fixture-070-dns-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | PTR-only no mutation | `fixture-070-ptr-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | scanner-name-only no mutation | `fixture-070-scanner-name-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | weak-management-name-only no mutation | `fixture-070-weak-management-name-only` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | weak-user-name/mail/UPN-only no mutation | `fixture-070-weak-user-name-mail-upn-only` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | group-membership-only no mutation | `fixture-070-group-membership-only` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | weak-service-account-display-name-only no mutation | `fixture-070-weak-service-account-display-name-only` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | selector-only no mutation | `fixture-070-selector-only-no-mutation` | TODO | no-op | TODO | no identity mutation | `070-SELECTOR-SAFETY-TOTALITY-AC-001` | blocking |
| `070` | graph-key-only no mutation | `fixture-070-graph-key-only-no-mutation` | TODO | no-op | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | OpenGraph property-match no mutation | `fixture-070-opengraph-property-match-only` | TODO | no-op | TODO | no identity mutation | `070-SELECTOR-SAFETY-TOTALITY-AC-001` | blocking |
| `070` | mapped-target-only no mutation | `fixture-070-mapped-target-only` | TODO | no-op | TODO | no identity mutation | `070-SELECTOR-SAFETY-TOTALITY-AC-001` | blocking |
| `070` | source-native-merge-history-only no mutation | `fixture-070-source-native-merge-history-only` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | learned-only no mutation | `fixture-070-learned-only-no-mutation` | TODO | `no_decision` | TODO | no identity mutation | `070-WEAK-REJECTION-AC-001` | blocking |
| `070` | missing resolver row | `fixture-070-resolver-row-missing` | TODO | `RESOLVER_PROFILE_ROW_MISSING` | TODO | no identity mutation | `070-RESOLVER-ROW-AC-001` | blocking |
| `070` | ambiguous resolver row | `fixture-070-resolver-row-ambiguous` | TODO | `RESOLVER_PROFILE_ROW_AMBIGUOUS` | TODO | no identity mutation | `070-RESOLVER-ROW-AC-001` | blocking |
| `070` | missing hard blocker row | `fixture-070-hard-blocker-row-missing` | TODO | `RESOLVER_HARD_BLOCKER_ROW_MISSING` | TODO | no identity mutation | `070-HARD-BLOCKER-ROWSET-TOTALITY-AC-001` | blocking |
| `070` | ambiguous hard blocker row | `fixture-070-hard-blocker-row-ambiguous` | TODO | `RESOLVER_HARD_BLOCKER_ROW_AMBIGUOUS` | TODO | no identity mutation | `070-HARD-BLOCKER-ROWSET-TOTALITY-AC-001` | blocking |
| `070` | decision row ambiguity | `fixture-070-decision-row-ambiguous` | TODO | `RESOLVER_DECISION_ROW_AMBIGUOUS` | TODO | no identity mutation | `070-DECISION-ROW-AMBIGUITY-AC-001` | blocking |
| `070` | missing explanation field | `fixture-070-missing-explanation-field` | TODO | `RESOLVER_EXPLANATION_INCOMPLETE` | TODO | no externally visible decision | `070-EXPLANATION-AC-001` | blocking |
| `070` | explanation volatile-field drift | `fixture-070-explanation-volatile-drift` | TODO | no-op or replay mismatch | TODO | no identity mutation | `070-EXPLANATION-AC-001` | blocking |
| `070` | candidate block overflow | `fixture-070-candidate-block-overflow` | TODO | `RESOLVER_CANDIDATE_BLOCK_OVERFLOW` | TODO | no identity mutation | `070-OVERFLOW-AC-001` | blocking |
| `070` | candidate partition overflow | `fixture-070-candidate-partition-overflow` | TODO | `RESOLVER_CANDIDATE_PARTITION_OVERFLOW` | TODO | no identity mutation | `070-OVERFLOW-AC-001` | blocking |
| `070` | reviewer override of hard blocker | `fixture-070-reviewer-hard-blocker-override` | TODO | `IDENTITY_HARD_BLOCKER_TRIGGERED` | TODO | no identity mutation | `070-BLOCKER-PRECEDENCE-AC-001` | blocking |
| `070` | review expiration | `fixture-070-review-expiration` | TODO | expired no-op | TODO | no identity mutation | `070-REVIEW-EXPIRATION-AC-001` | blocking |
| `070` | split handoff missing metadata | `fixture-070-split-handoff-missing-metadata` | TODO | `RESOLVER_EXPLANATION_INCOMPLETE` or owner handoff error | TODO | no graph mutation | `070-GRAPH-CORRECTION-HANDOFF-SCHEMA-AC-001` | blocking |
| `070` | package-supplied weak-default override | `fixture-070-package-weak-default-override` | TODO | package activation failure | TODO | preserve active package set and no identity output | `100-IDENTITY-RESOLVER-WEAKENING-AC-001` | blocking |
| `080` | missing temporal policy | `temporal-resolution-missing-policy` | TODO | `TEMPORAL_POLICY_UNRESOLVED` | TODO | no gold fact | `080-TEMPORAL-POLICY-AC-001` | blocking |
| `080` | absent source time | `temporal-resolution-absent-time` | TODO | temporal error or unknown per active row | TODO | no unauthorized fallback | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | malformed source time | `temporal-resolution-malformed-time` | TODO | temporal error | TODO | no gold fact | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | ambiguous source time | `temporal-resolution-ambiguous-time` | TODO | temporal error | TODO | no gold fact | `080-TEMPORAL-RESOLUTION-AC-001` | blocking |
| `080` | forbidden current-time fallback | `fixture-080-current-time-fallback` | TODO | `SOURCE_TIME_NOT_AUTHORIZED` or temporal error | TODO | no gold fact | `080-CLEANUP-AC-003` | blocking |
| `080` | historical known-time reconstruction rejected | `knowledge-time-import-reconstruction-rejected` | TODO | rejected reconstruction | TODO | no historical known-time claim | `080-KNOWLEDGE-TIME-AC-001` | blocking |
| `080` | missing late-arrival row | `late-arrival-policy-missing` | TODO | `LATE_ARRIVAL_POLICY_MISSING` | TODO | no correction route | `080-LATE-ARRIVAL-AC-001` | blocking |
| `080` | late discard forbidden | `late-arrival-discard-forbidden` | TODO | `LATE_ARRIVAL_DISCARD_FORBIDDEN` | TODO | evidence preserved | `080-LATE-ARRIVAL-AC-001` | blocking |
| `080` | missing correction policy | `gold-correction-policy-missing` | TODO | `CORRECTION_POLICY_MISSING` | TODO | no correction | `080-CORRECTION-MATRIX-AC-001` | blocking |
| `080` | missing correction transition | `assertion-transition-missing` | TODO | `GOLD_CORRECTION_TRANSITION_UNDEFINED` | TODO | no correction or graph handoff | `080-CORRECTION-MATRIX-AC-001` | blocking |
| `080` | missing old snapshot | `correction-snapshot-old-missing` | TODO | `CORRECTION_SNAPSHOT_REF_MISSING` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | missing new snapshot | `correction-snapshot-new-missing` | TODO | `CORRECTION_SNAPSHOT_REF_MISSING` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | mutable snapshot ref | `correction-snapshot-mutable-ref` | TODO | `MUTABLE_BRANCH_REF_FOR_REPLAY` | TODO | no correction | `080-CORRECTION-SNAPSHOT-AC-001` | blocking |
| `080` | duplicate candidate no-op | `gold-correction-duplicate-no-op` | TODO | `no_op_duplicate` | TODO | no graph mutation | `080-CORRECTION-NOOP-AC-001` | blocking |
| `080` | unauthorized retraction | `gold-correction-unauthorized-retraction` | TODO | owner `060` blocking reason | TODO | no retraction, no graph expiry, no watermark | `080-UNSAFE-NEGATIVE-AC-001` | blocking |
| `080` | unauthorized CDC tombstone | `gold-correction-unauthorized-cdc-tombstone` | TODO | `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | TODO | no retraction, cleanup, expiry, or watermark | `080-UNSAFE-NEGATIVE-AC-001` | blocking |
| `080` | confidence-only same value no-op | `gold-correction-confidence-same-no-op` | TODO | `no_op_duplicate` | TODO | no new active fact and no graph delta | `080-CORRECTION-NOOP-AC-001` | blocking |
| `080` | confidence-only changed value | `gold-correction-confidence-changed` | TODO | `insert_fact` or policy-defined update | TODO | append-only change set only | `040-CONFIDENCE-CANONICAL-AC-001` | blocking |
| `080` | missing replay output-class row | `replay-output-class-row-missing` | TODO | `REPLAY_POLICY_ARTIFACT_MISSING` | TODO | no replay output | `080-REPLAY-OUTPUT-CLASS-AC-001` | blocking |
| `080` | replay mutable ref | `replay-mutable-ref` | TODO | `MUTABLE_BRANCH_REF_FOR_REPLAY` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay retention failure | `replay-retention-ineligible` | TODO | `REPLAY_INPUT_INSUFFICIENT` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay authority mismatch | `replay-authority-mismatch` | TODO | `REPLAY_INPUT_INSUFFICIENT` or `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay temporal mismatch | `replay-temporal-mismatch` | TODO | `REPLAY_INPUT_INSUFFICIENT` or `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay side-effect mismatch | `replay-side-effect-mismatch` | TODO | `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-REPLAY-PRECEDENCE-AC-001` | blocking |
| `080` | replay checksum mismatch | `fixture-080-replay-mismatch` | TODO | `REPLAY_CHECKSUM_MISMATCH` | TODO | no replay output | `080-CLEANUP-AC-004` | blocking |
| `090` | theoretical reachability edge | `fixture-090-theoretical-reachability-edge` | TODO | `THEORETICAL_REACHABILITY_SCOPE_ERROR` or no-op | TODO | no graph mutation | `090-CLEANUP-AC-004` | blocking |
| `090` | backend internal ID | `fixture-090-backend-id` | TODO | `GRAPH_BACKEND_ID_FORBIDDEN` | TODO | no graph response identity leak | `090-CLEANUP-AC-003` | blocking |
| `090` | graph apply unsafe resume | `fixture-090-lifecycle-graph-apply-resume-unsafe` | TODO | `GRAPH_APPLY_RESUME_UNSAFE` | TODO | no graph mutation and no derived-view advancement | `090-LIFECYCLE-AC-003` | blocking |
| `090` | graph apply identical reapply | `fixture-090-lifecycle-identical-reapply` | TODO | idempotent no-op | TODO | no duplicate graph object | `090-LIFECYCLE-AC-002` | blocking |
| `090` | OCSF endpoint order no graph direction | `ocsf-endpoint-order-no-graph-direction-*` | TODO | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op | TODO | no graph mutation | `090-OCSF-DIRECTION-AC-001` | blocking |
| `090` | missing flow-role evidence | `flow-role-evidence-required-*` | TODO | `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op | TODO | no graph mutation | `090-OCSF-DIRECTION-AC-002` | blocking |
| `100` | package type unknown | `fixture-100-package-type-unknown` | TODO | `PACKAGE_TYPE_UNKNOWN` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-002` | blocking |
| `100` | package type policy missing | `fixture-100-package-type-policy-missing` | TODO | `PACKAGE_TYPE_POLICY_MISSING` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-003` | blocking |
| `100` | package type policy ambiguous | `fixture-100-package-type-policy-ambiguous` | TODO | `PACKAGE_TYPE_POLICY_AMBIGUOUS` | TODO | no package activation | `100-PACKAGE-TYPE-POLICY-AC-004` | blocking |
| `100` | unsupported Git tree snapshot | `fixture-100-git-tree-snapshot-unsupported` | TODO | `PACKAGE_REPOSITORY_FORM_UNSUPPORTED` | TODO | no candidate production output | `100-REPOSITORY-FORM-AC-002` | blocking |
| `100` | unauthorized signer | `fixture-100-unauthorized-signer` | TODO | `PACKAGE_SIGNER_UNAUTHORIZED` | TODO | no package activation | `100-CLEANUP-AC-003` | blocking |
| `100` | missing transparency evidence | `fixture-100-transparency-evidence-missing` | TODO | `PACKAGE_TRANSPARENCY_EVIDENCE_MISSING` | TODO | no package activation | `100-TRANSPARENCY-AC-001` | blocking |
| `100` | missing attestation | `fixture-100-attestation-missing` | TODO | `PACKAGE_ATTESTATION_MISSING` | TODO | no package activation | `100-ATTESTATION-AC-001` | blocking |
| `100` | missing SBOM | `fixture-100-sbom-missing` | TODO | `PACKAGE_SBOM_MISSING` | TODO | no package activation | `100-SBOM-AC-001` | blocking |
| `100` | dependency live resolution | `fixture-100-dependency-live-resolution-forbidden` | TODO | `PACKAGE_DEPENDENCY_LIVE_RESOLUTION_FORBIDDEN` | TODO | no package activation | `100-DEPENDENCY-LOCK-AC-001` | blocking |
| `100` | compatibility failed | `fixture-100-compatibility-failed` | TODO | `PACKAGE_COMPATIBILITY_FAILED` | TODO | no package activation | `100-COMPATIBILITY-AC-001` | blocking |
| `100` | deprecation window expired | `fixture-100-deprecation-window-expired` | TODO | `PACKAGE_DEPRECATION_WINDOW_EXPIRED` | TODO | no new activation | `100-DEPRECATION-WINDOW-AC-002` | blocking |
| `100` | LKG health gate failed | `fixture-100-lkg-health-gate-failed` | TODO | `PACKAGE_LKG_HEALTH_GATE_FAILED` | TODO | no last-known-good mark | `100-LKG-HEALTH-AC-002` | blocking |
| `100` | mutable rollback target | `fixture-100-mutable-rollback-target` | TODO | `PACKAGE_ROLLBACK_TARGET_UNVERIFIED` | TODO | no package activation | `100-ROLLBACK-AC-001` | blocking |
| `100` | rollback compatibility failure | `fixture-100-rollback-state-incompatible` | TODO | `PACKAGE_ROLLBACK_STATE_INCOMPATIBLE` | TODO | active set unchanged | `100-ROLLBACK-AC-002` | blocking |
| `100` | quarantine blocks dependent activation | `fixture-100-quarantine-artifact-blocks-release` | TODO | `PACKAGE_QUARANTINE_BLOCKED_ACTIVATION` | TODO | no dependent package activation | `100-QUARANTINE-AC-001` | blocking |
| `100` | emergency bypass forbidden | `fixture-100-emergency-bypass-forbidden` | TODO | `EMERGENCY_PACKAGE_BYPASS_FORBIDDEN` | TODO | no package activation | `100-EMERGENCY-AC-001` | blocking |
| `100` | package manifest refs missing | `fixture-100-version-manifest-package-refs-missing` | TODO | `PACKAGE_VERSION_MANIFEST_INCOMPLETE` | TODO | no package output visibility | `100-VERSION-MANIFEST-AC-001` | blocking |
| `100` | canary output isolation | `fixture-100-canary-output-isolation` | TODO | `PACKAGE_CANARY_OUTPUT_FORBIDDEN` | TODO | no current production output or LKG | `100-LIFECYCLE-AC-003` | blocking |
| `100` | shadow output isolation | `fixture-100-shadow-output-isolation` | TODO | `PACKAGE_SHADOW_OUTPUT_FORBIDDEN` | TODO | no current production output or LKG | `100-LIFECYCLE-AC-003` | blocking |
| `110` | state-label collapse | `fixture-110-state-label-collapse` | TODO | reject/non-collapse | TODO | no incorrect API state | `110-CLEANUP-AC-002` | blocking |
| `110` | page token authorization mismatch | `fixture-110-page-token-auth-mismatch` | TODO | `PAGE_TOKEN_INVALID` | TODO | no data disclosure | `110-PAGE-TOKEN-AC-001` | blocking |
| `110` | new owner state label non-collapse | `fixture-110-feed-closure-state-labels` | TODO | reject/non-collapse | TODO | no authorized negative fact rendering | `110-FEED-CLOSURE-AC-001` | blocking |
| `130` | analysis finding non-authority | `fixture-130-analysis-finding-non-authority` | TODO | `none or no-op` | TODO | no forbidden mutation | `130-ANALYSIS-OUTPUT-AUTHORITY-TOTAL-AC-001` | blocking |
| `130` | analysis finding mutation forbidden | `fixture-130-analysis-finding-mutation-forbidden` | TODO | `ANALYSIS_MUTATION_FORBIDDEN` | TODO | no mutation | `130-ANALYSIS-MUTATION-AC-001` | blocking |
| `130` | analysis metric non-authority | `fixture-130-analysis-metric-non-authority` | TODO | `none or no-op` | TODO | no authoritative risk score | `130-ANALYSIS-OUTPUT-AUTHORITY-TOTAL-AC-001` | blocking |
| `130` | analysis metric risk score forbidden | `fixture-130-analysis-metric-risk-score-forbidden` | TODO | `ANALYSIS_MUTATION_FORBIDDEN` | TODO | no authoritative score | `130-RISK-SCORING-AC-001` | blocking |
| `130` | risk acceptance no remediation | `fixture-130-risk-acceptance-no-remediation` | TODO | `none or no-op` | TODO | no remediation or retraction | `130-ANALYSIS-OUTPUT-AUTHORITY-TOTAL-AC-001` | blocking |
| `130` | analysis replay exact | `fixture-130-analysis-replay-exact` | TODO | `none` | TODO | exact output checksum | `130-ANALYSIS-REPLAY-AC-001` | blocking |
| `130` | analysis replay mismatch | `fixture-130-analysis-replay-mismatch` | TODO | `DERIVED_GRAPH_EDGE_REPLAY_MISMATCH or replay owner error` | TODO | no production output | `130-ANALYSIS-REPLAY-AC-002` | blocking |
| `130` | threat intel known indicator | `fixture-130-threat-intel-known-indicator` | TODO | `none` | TODO | enrichment only | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | threat intel identity forbidden | `fixture-130-threat-intel-identity-forbidden` | TODO | `THREAT_INTEL_IDENTITY_FORBIDDEN` | TODO | no identity mutation | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | threat intel distribution restricted | `fixture-130-threat-intel-distribution-restricted` | TODO | `none or THREAT_INTEL_DISTRIBUTION_UNMAPPED` | TODO | restricted export/no leak | `130-THREAT-INTEL-DISTRIBUTION-AC-001` | blocking |
| `130` | threat intel unknown taxonomy | `fixture-130-threat-intel-unknown-taxonomy` | TODO | `none or profile rejection` | TODO | no coercion | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | threat intel missing profile | `fixture-130-threat-intel-missing-profile` | TODO | `THREAT_INTEL_PROFILE_MISSING` | TODO | no enrichment output | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | threat intel artifact checksum mismatch | `fixture-130-threat-intel-artifact-checksum-mismatch` | TODO | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` | TODO | no enrichment output | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | threat intel sighting no completeness | `fixture-130-threat-intel-sighting-no-completeness` | TODO | `none or no-op` | TODO | no completeness effect | `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | blocking |
| `130` | lineage run job dataset positive | `fixture-130-lineage-run-job-dataset-positive` | TODO | `none` | TODO | lineage metadata only | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage custom policy missing | `fixture-130-lineage-custom-policy-missing` | TODO | `LINEAGE_FACET_POLICY_MISSING` | TODO | no production effect | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage schema url mutable | `fixture-130-lineage-schema-url-mutable` | TODO | `LINEAGE_FACET_SCHEMA_MUTABLE` | TODO | no lineage activation | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage facet checksum mismatch | `fixture-130-lineage-facet-checksum-mismatch` | TODO | `LINEAGE_FACET_CHECKSUM_MISMATCH` | TODO | no lineage activation | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage namespace collision | `fixture-130-lineage-namespace-collision` | TODO | `LINEAGE_FACET_NAMESPACE_COLLISION` | TODO | no lineage activation | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage freshness no completeness | `fixture-130-lineage-freshness-no-completeness` | TODO | `none or no-op` | TODO | no completeness effect | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | lineage facet only not evidence | `fixture-130-lineage-facet-only-not-evidence` | TODO | `none or no-op` | TODO | no evidence authority | `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | blocking |
| `130` | registry activation positive | `fixture-130-registry-activation-positive` | TODO | `none` | TODO | governance metadata only | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | registry inactive artifact | `fixture-130-registry-inactive-artifact` | TODO | `REGISTRY_ARTIFACT_INACTIVE` | TODO | no activation | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | registry owner mismatch | `fixture-130-registry-owner-mismatch` | TODO | `REGISTRY_ARTIFACT_OWNER_MISMATCH` | TODO | no activation | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | registry label no fact authority | `fixture-130-registry-label-no-fact-authority` | TODO | `REGISTRY_AUTHORITY_FORBIDDEN or REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` | TODO | no fact authority | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | registry custom property bounds | `fixture-130-registry-custom-property-bounds` | TODO | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | TODO | no activation | `130-REGISTRY-CUSTOM-PROPERTY-AC-001` | blocking |
| `130` | registry custom property redaction | `fixture-130-registry-custom-property-redaction` | TODO | `none` | TODO | redacted caller output | `130-REGISTRY-CUSTOM-PROPERTY-AC-001` | blocking |
| `130` | registry classification authority forbidden | `fixture-130-registry-classification-authority-forbidden` | TODO | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | TODO | no authority effect | `130-REGISTRY-CUSTOM-PROPERTY-AC-001` | blocking |
| `130` | registry package set ref required | `fixture-130-registry-package-set-ref-required` | TODO | `REGISTRY_ARTIFACT_INACTIVE` | TODO | no package-supplied activation | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | registry private binding redaction | `fixture-130-registry-private-binding-redaction` | TODO | `none or PRIVATE_BINDING_LEAK` | TODO | no private leak | `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | blocking |
| `130` | derived edge supporting facts required | `fixture-130-derived-edge-supporting-facts-required` | TODO | `DERIVED_GRAPH_EDGE_SUPPORTING_FACTS_REQUIRED` | TODO | no output | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge routed through 090 | `fixture-130-derived-edge-routed-through-090` | TODO | `none` | TODO | graph output only through 090 | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge outside 090 forbidden | `fixture-130-derived-edge-outside-090-forbidden` | TODO | `DERIVED_GRAPH_EDGE_PROJECTION_FORBIDDEN` | TODO | no direct graph mutation | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge gold output routed through 080 | `fixture-130-derived-edge-gold-output-routed-through-080` | TODO | `none or 080 owner error` | TODO | gold output only through 080 | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge replay exact | `fixture-130-derived-edge-replay-exact` | TODO | `none` | TODO | exact output checksum | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge replay mismatch | `fixture-130-derived-edge-replay-mismatch` | TODO | `DERIVED_GRAPH_EDGE_REPLAY_MISMATCH` | TODO | no production output | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `130` | derived edge explicit no-op | `fixture-130-derived-edge-explicit-no-op` | TODO | `explicit no-op` | TODO | no output mutation | `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | blocking |
| `200` | active reachability output | `fixture-200-active-reachability-output` | TODO | `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | TODO | no gold, graph, API output | `200-CLEANUP-AC-003` | blocking |
| volatility | activation artifact redefines stable core behavior; activation artifact omitted from `VersionManifest`; runtime state substituted as source truth | `fixture-120-volatility-boundary` | TODO | owner-specific volatility error or no-op | TODO | no production mutation | `120-VOLATILITY-AC-001` | blocking |

### LifecycleValidationMatrix

Lifecycle validation rows must be executable ordered event sequences. Each row must name expected transition evidence, expected output checksum or no-op, expected error when applicable, and mutation-prohibition proof.

| Validation row | Owner | Required assertion |
| --- | --- | --- |
| `val-030-lifecycle-artifact-totality` | `030` | Every artifact lifecycle state/event pair resolves to legal transition, no-op, or illegal-transition evidence. |
| `val-030-lifecycle-idempotent-replay` | `030` | Same event key and same inputs produce byte-identical evidence. |
| `val-030-lifecycle-idempotency-conflict` | `030` | Same key with changed input checksum fails with `LIFECYCLE_IDEMPOTENCY_CONFLICT`. |
| `val-030-lifecycle-production-run` | `030` | Non-isolated stage failure stops downstream stages and emits no downstream production output. |
| `val-030-lifecycle-stage-execution` | `030` | Stage execution machine has total state/event coverage and expected terminal states. |
| `val-030-lifecycle-feed-stage` | `020`, `030` | Partial feed read emits valid lifecycle result with blocked effects and no absence or watermark. |
| `val-070-identity-review-totality` | `070` | Every identity review state/event pair resolves to allowed transition, terminal no-op, idempotent no-op, or illegal-transition evidence. |
| `val-070-identity-review-idempotency` | `070` | Same review event key and same input checksums produce byte-identical evidence. |
| `val-070-identity-review-illegal-no-mutation` | `070` | Illegal review transitions emit no identity, graph, completeness, package, or watermark mutation. |
| `val-070-identity-review-terminal-output` | `070` | Terminal review output emits only terminal `IdentityDecision` records and required explanation refs. |
| `val-070-identity-review-evidence-snapshot` | `070` | Evidence snapshot mismatch emits `IDENTITY_REVIEW_EVIDENCE_SNAPSHOT_MISMATCH` and no mutation. |
| `val-090-lifecycle-graph-apply-totality` | `090` | Graph apply machine has total state/event coverage. |
| `val-090-lifecycle-graph-apply-resume-safe` | `090` | Partial apply with committed-batch proof resumes after last compatible batch. |
| `val-090-lifecycle-graph-apply-resume-unsafe` | `090` | Partial apply without proof fails with `GRAPH_APPLY_RESUME_UNSAFE`. |
| `val-090-lifecycle-graph-apply-identical-reapply` | `090` | Identical reapply returns no-op and does not mutate graph. |
| `val-100-lifecycle-package-activation` | `100` | Candidate activation failure preserves current active package set. |
| `val-100-lifecycle-canary-isolation` | `100` | Canary and shadow emit no current production output, watermark, or last-known-good state. |
| `val-100-lifecycle-rollback-immutable-target` | `100` | Mutable rollback targets fail. |
| `val-100-lifecycle-quarantine-blocks-dependent-activation` | `100` | Quarantine blocks dependent activation and cannot clear directly to active. |
| `val-100-lifecycle-deprecation-window-expiry` | `100` | Deprecated artifact cannot be newly selected after window expiry except verified rollback. |
| `val-120-lifecycle-validation-acceptance` | `120` | Acceptance reaches `passed` only when every required row passes and no owner TODO remains. |
| `val-domain-lifecycle-todo-resolved` | `domain`, `000`, `120` | `DOM-TODO-010` is resolved only when all owner machines and validation rows exist and pass. |

### ValidationAcceptanceLifecycleMachine

Machine ID: `120.ValidationAcceptanceLifecycleMachine.v1`.

States:

```text
pending
running
passed
failed
blocked
expired
superseded
```

Events:

```text
start_validation
all_rows_passed
row_failed
owner_todo_found
acceptance_expired
superseded_by_new_report
replay_same_event
```

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `pending` | `start_validation` | `running` | Matrix, fixture refs, owner specs, and input checksums validate. |
| `running` | `all_rows_passed` | `passed` | Persist `AcceptanceReport`. |
| `running` | `row_failed` | `failed` | Acceptance report includes failing rows. |
| `running` | `owner_todo_found` | `blocked` | Persist blocking report. |
| `passed` | `acceptance_expired` | `expired` | Persist expiry evidence. |
| `passed`, `failed`, `blocked`, or `expired` | `superseded_by_new_report` | `superseded` | Persist supersession evidence. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

### ExpectedMutationProhibition rows

| Prohibition token | Required proof |
| --- | --- |
| no production output | Output manifest contains no production-visible record refs. |
| no graph mutation | Graph apply result absent or no-op; derived-view state unchanged. |
| no watermark advancement | No `WatermarkCommitRecord` with advanced value. |
| no package activation | Current active package set checksum unchanged. |
| no identity mutation | No new terminal `IdentityDecision` that mutates canonical identity. |
| no raw payload exposure | API/export/audit caller-visible fields contain no raw payload bytes. |

### ApiSurfaceClosureValidationMatrix

This matrix verifies behavior owned by `110`; it does not define new API behavior. Each row must execute against the schema and outcome rules in `110` and must compare canonical response checksums.

| API schema | Required fixture classes | Owner spec | Fixture family | Blocking condition |
| --- | --- | --- | --- | --- |
| `CommonApiResponseEnvelope` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-commonapiresponseenvelope` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AssetSearchRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-assetsearchrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AssetSearchResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-assetsearchresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AssetDetailRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-assetdetailrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AssetDetailResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-assetdetailresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `EvidenceDrillbackRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-evidencedrillbackrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `EvidenceDrillbackResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-evidencedrillbackresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `GraphQueryRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-graphqueryrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `GraphQueryResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-graphqueryresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `OperationalHealthRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-operationalhealthrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `OperationalHealthStatus` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-operationalhealthstatus` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `ComplianceExportRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-complianceexportrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `ComplianceExportResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-complianceexportresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AuditExportRequest` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-auditexportrequest` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |
| `AuditExportResponse` | valid minimal request or response, missing required field, unknown field rejection, default materialization, bounds rejection where bounded, redaction case, authorization denial, empty-result case where applicable, stale/partial/conflicted/ambiguous state rendering where applicable, page-token replay where applicable | `110` | `api-110-auditexportresponse` | blocked until fixture, input checksum, expected output checksum, expected error, redaction summary, audit event ref, and `VersionManifest` ref are non-TODO |

### SourceStateLabelTotalityValidationMatrix

| SourceStateLabel | Owner mapping | Required non-collapse assertion | Fixture family |
| --- | --- | --- | --- |
| `source_stale` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-source-stale` |
| `derived_view_stale` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-derived-view-stale` |
| `unknown` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-unknown` |
| `not_applicable` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-not-applicable` |
| `authorized_absent` | `110.SourceStateLabelMapping` | must not collapse to `authorized_not_observed`, pass, fail, unknown, not_checked, not_applicable, graph cleanup, retraction, watermark, remediation, conflict, ambiguity, or error | `state-label-110-authorized-absent` |
| `authorized_not_observed` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-authorized-not-observed` |
| `permission_limited` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-permission-limited` |
| `source_unavailable` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-source-unavailable` |
| `scope_unavailable` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-scope-unavailable` |
| `partial_known_gap` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-partial-known-gap` |
| `partial_unknown_gap` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-partial-unknown-gap` |
| `not_checked` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-not-checked` |
| `error` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation | `state-label-110-error` |
| `conflicted` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation; must not collapse to `unknown` or `error` unless the owner emits an explicit error row | `state-label-110-conflicted` |
| `ambiguous` | `110.SourceStateLabelMapping` | must not collapse to pass, fail, authorized absence, graph cleanup, retraction, watermark, or remediation; must not collapse to `unknown` or `error` unless the owner emits an explicit error row | `state-label-110-ambiguous` |

### ValidationErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry` for validation-owned failures that are visible in promotion reports, validation diagnostics, API diagnostics, health diagnostics, or audit diagnostics. `120` owns validation causes and owner context only; `110` owns generated registry shape, field sets, redaction, duplicate-code policy, and checksum inclusion.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `VALIDATION_ERROR_REGISTRY_OWNER_ROUTED_TO_000` | `120` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `120.ValidationErrorContext` | `error-registry-120-validation-error-registry-owner-routed-to-000` |

### ValidationErrorContext

`ValidationErrorContext` is the owner context schema for `120` validation-owned registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `120` context schema version. |
| `owner_spec` | Yes | Must be `120`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `validation_harness`, `fixture_checksum`, `expected_output_checksum`, `registry_validation`, `package_parity`, `manifest_requirement`, or `governance_owner_routing`. |
| `operation` | Yes | Spec-set validation, manifest validation, owner-map validation, ADR status validation, or promotion-gate validation. |
| `affected_record_type` | Yes | Domain row, owner spec row, ADR row, manifest path row, validation report row, or spec registry row. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to validation rows, fixture rows, expected output artifacts, owner fragments, generated registry artifacts, manifest refs, package-set refs, acceptance reports, or spec-set inventory artifacts consulted by the error; empty only when no artifact was consulted. |
| `validation_row_id` | Yes | Exact validation row that detected the failure. |
| `expected_owner_ref` | No | Required when an owner contradiction was evaluated. |
| `actual_owner_ref` | No | Required when an owner contradiction was evaluated. |
| `manifest_path` | No | Required for registry path mismatches. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded validation blocking reason; otherwise null or omitted. |
| `validation_refs` | Yes | Exact validation fixture refs. |
| `redaction_classes` | Yes | Private paths, private binding values, raw fixture bytes, credentials, source-native identity values, backend IDs, and package payload bytes must map to `always_forbidden`. |

### ErrorCodeRegistryValidationMatrix

`RunValidationMatrix` must expand this matrix against the owner fragments in `010`, `020`, `030`, `040`, `050`, `060`, `070`, `080`, `090`, `100`, shared `110` rows, `120`, `130`, and `140`. The expanded validation set must contain one generated validation case per final owner row after duplicate-owner removal. Validation must use the normalized `error_code`, `severity`, `retry_class`, `caller_visible_fields`, `audit_visible_fields`, `redaction_rule`, `owner_context_schema_ref`, and `fixture_ref` fields emitted by `110.GenerateErrorCodeRegistry`.

#### ErrorCodeRegistryExpandedValidationRowSchema

`RunValidationMatrix` must materialize one expanded validation row for every final generated registry row. Each expanded row must use this schema before `AcceptanceReport.result = pass` is allowed.

| Field | Required behavior |
| --- | --- |
| `validation_row_id` | Stable ID in the form `val-error-registry-<owner>-<error-code-kebab>`. |
| `owner_spec` | Exact owner from the generated row. |
| `error_code` | Exact generated error code. |
| `fixture_id` | Exact fixture ID; wildcard-only fixture families are invalid. |
| `fixture_checksum` | Concrete SHA-256. `TODO` means the validation row is `blocked`. |
| `expected_output_checksum` | Concrete SHA-256 for the expected generated row bytes or expected rejection bytes. `TODO` means the validation row is `blocked`. |
| `generated_row_checksum` | SHA-256 over the canonical expanded `ErrorCodeRegistryRow`. |
| `context_schema_ref` | Exact owner context schema ref satisfying `110.OwnerErrorContextMinimumSchema`. |
| `redaction_proof` | Required proof that caller and audit fields follow the row redaction rule and nested owner context classes. |
| `caller_field_proof` | Required proof that caller-visible fields equal `110.StandardErrorCallerFields` and contain no legacy `code` field. |
| `audit_field_proof` | Required proof that audit-visible fields equal `110.StandardErrorAuditFields`. |
| `manifest_checksum_proof` | Required proof that visible diagnostic output includes `030.VersionManifest.generated_error_registry_checksum`. |
| `package_parity_proof` | Required for package-owned rows and null otherwise. |
| `alias_rejection_proof` | Required for rejected aliases and null otherwise. |
| `blocking_status` | Closed token: `pass`, `fail`, `blocked`, or `not_run`. |

`AcceptanceReport.result = pass` is forbidden while any expanded registry validation row has `fixture_checksum = TODO`, `expected_output_checksum = TODO`, `blocking_status` other than `pass`, duplicate owner, invalid enum, missing owner context field, generic substitution, alias acceptance, wildcard fixture ref, missing package parity proof, or missing manifest checksum proof.

| Validation row | Scope | Required fixture refs | Required result |
| --- | --- | --- | --- |
| `error-registry-owner-row-totality` | Every final owner fragment row from `010`, `020`, `030`, `040`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, and `140`. | `error-registry-<owner>-<error-code-kebab>` for every final owner code. | Fails if any row is missing, blank, unfixtured, duplicated, unknown-severity, unknown-retry-class, or contains `TODO`. |
| `error-registry-standard-caller-fields` | Every generated row. | One positive and one negative fixture per owner. | Generated caller fields must equal `110.StandardErrorCallerFields`; any `code` field, omitted `error_code`, or owner-specific top-level caller field fails. |
| `error-registry-standard-audit-fields` | Every generated row. | One positive and one negative fixture per owner. | Generated audit fields must equal `110.StandardErrorAuditFields`; owner-specific values must appear only under `owner_error_context`. |
| `error-registry-redaction-classes` | Every generated row plus sensitive-value fixtures. | Raw payload, credential, private binding, source-native identity, backend ID, provider query text, raw SBOM, signer secret, and registry payload fixtures. | Sensitive values map to `always_forbidden` and never appear caller-visible. |
| `error-registry-fixture-ref-exact` | Every owner fragment row. | Wildcard-only and exact fixture-ref fixtures. | Wildcard-only fixture refs, empty fixture refs, and legacy unnormalized `fixture_family` refs fail before registry output. |
| `error-registry-deterministic-bytes` | Complete generated registry. | Two independent generator fixtures over the same owner fragments. | Generated rows sort by `error_code` and produce byte-identical registry bytes and checksum. |
| `error-registry-version-manifest-checksum` | API, export, health, compliance, audit, and validation-visible outputs. | Manifest-present and manifest-absent fixtures. | Output fails before visibility when `030.VersionManifest.included_refs` lacks `generated_error_registry_checksum`. |
| `error-registry-generic-substitution-rejected` | Shared generic errors versus owner-specific rows. | `AUTHORIZATION_ERROR`, `PAGE_TOKEN_INVALID`, `API_BOUNDS_INVALID`, and owner-specific failure fixtures. | Generic shared code is rejected when a specific owner row covers the same failure class and owner context. |
| `error-registry-private-binding-owner` | Duplicate-owner decision for `PRIVATE_BINDING_LEAK`. | `error-registry-010-private-binding-leak`, core-detected leak fixture. | Exactly one generated row exists, owned by `010`; `040` duplicate ownership fails. |
| `error-registry-external-schema-authority-owner` | Duplicate-owner decision for `EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN`. | `error-registry-060-external-schema-authority-forbidden`, mapping-authority-attempt fixture. | Exactly one generated row exists, owned by `060`; `050` duplicate ownership fails. |
| `error-registry-cdc-tombstone-owner` | Duplicate-owner decision for `CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED`. | `error-registry-060-cdc-tombstone-retraction-unauthorized`, temporal tombstone fixture. | Exactly one generated row exists, owned by `060`; `080` duplicate ownership fails. |
| `error-registry-graph-endpoint-owner` | Duplicate-owner decision for `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. | `error-registry-090-graph-endpoint-identity-unresolved`, unresolved endpoint fixture. | Exactly one generated row exists, owned by `090`; `070` duplicate ownership fails and no identity mutation occurs. |
| `error-registry-derived-view-lag-owner` | Duplicate-owner decision for `DERIVED_VIEW_LAG_ERROR`. | `error-registry-090-derived-view-lag-error`, stale derived-view fixture. | Exactly one generated row exists, owned by `090`; `110` duplicate ownership fails. |
| `error-registry-alias-rejection` | Rejected shorthand and alias code names. | `fixture-120-error-registry-alias-rejection`. | `OCSF_MAPPING_ROW_MISSING`, `OCSF_MAPPING_ROW_AMBIGUOUS`, `OCSF_COMPILED_ARTIFACT_CHECKSUM_MISMATCH`, `FEED_CATEGORY_CLOSURE_ROW_MISSING`, and `SOURCE_DATASET_CATALOG_MISSING` fail before generated output. |
| `error-registry-invalid-retry-class` | Closed retry-class enum. | `fixture-120-error-registry-invalid-retry-class`. | `retry_after_candidate_changes` fails; `RESOLVER_ACTIVATION_REPORT_FAILED` with `retry_after_owner_repair` passes. |
| `error-registry-000-governance-owner` | Governance code owner routing. | `fixture-120-error-registry-000-governance-owner`. | Governance codes are generated from `000.GovernanceErrorRegistryFragment`; `120` duplicate ownership fails. |
| `error-registry-reachability-active-owner-routing` | Deferred reachability visible error routing. | `fixture-120-error-registry-reachability-routing`. | `THEORETICAL_REACHABILITY_SCOPE_ERROR` and `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` are active `090` graph errors; `REACHABILITY_UNQUALIFIED_CLAIM_FORBIDDEN` is a shared `110` wording error. |

### PackageErrorRegistryParityValidationMatrix

`PackageErrorRegistryParityValidationMatrix` validates define-once ownership for package errors. It compares `100.PackageErrorRegistryFragment` to `110.PackageActivationErrorObservableMapping` after macro expansion and before any API, health, audit, export, compliance, or validation-visible package error output.

| Validation row | Scope | Required fixture refs | Required result |
| --- | --- | --- | --- |
| `package-error-registry-code-set-parity` | Every `100.PackageErrorRegistryFragment.error_code` and every `110` package observable row. | `fixture-120-package-error-registry-parity` | Fails if any `100` package code is absent from `110`, or if any `110` package row lacks a `100` owner fragment. |
| `package-error-registry-severity-retry-parity` | Severity and retryability for package rows. | `fixture-120-package-error-registry-severity-retry` | Fails when `110` severity or retryability conflicts with the generated owner fragment. |
| `package-error-registry-redaction-parity` | Package sensitive data classes. | private package payload, raw SBOM bytes, signer secret, private repository path, private source binding, unauthorized evidence fixtures | Fails if forbidden data is caller-visible or stored outside approved secure audit refs. |
| `package-error-registry-fixture-totality` | Fixture refs for package owner rows. | one fixture ref per package error row | Fails when any package error row is unfixtured, wildcard-only, blank, duplicated, or TODO-bearing. |
| `package-error-registry-parity-failure-row` | Package parity failure visibility. | `error-registry-100-package-error-registry-parity-failed` | `PACKAGE_ERROR_REGISTRY_PARITY_FAILED` is generated from `100.PackageErrorRegistryFragment` and appears in package parity failures. |

### EndpointOutcomeValidationMatrix

| Endpoint | Required state and outcome coverage | Fixture family | Mutation prohibition |
| --- | --- | --- | --- |
| `AssetSearch` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-assetsearch` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `AssetDetail` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-assetdetail` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `EvidenceDrillback` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-evidencedrillback` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `GraphQuery` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-graphquery` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `OperationalHealthStatus` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-operationalhealthstatus` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `ComplianceExport` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-complianceexport` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `AuditExport` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-auditexport` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |
| `AnalysisReadOnly` | success, empty, unauthorized, stale source, stale derived view, partial known gap, partial unknown gap, conflicted, ambiguous, pagination, redaction, and most-specific error precedence | `endpoint-110-analysisreadonly` | no raw/silver/identity/gold/graph/package/completeness/watermark mutation |

### ApiSurfaceClosureFixtureFamilies

`110` fixture coverage must include API schema fixtures, endpoint outcome fixtures, state-label totality fixtures, authorization matrix fixtures, redaction matrix fixtures, page-token lifetime fixtures, page-token context mismatch fixtures, generated error-registry fixtures, compliance non-pass/fail fixtures, and audit export redaction fixtures.

`AcceptanceReport` must fail while any new API, label, endpoint outcome, authorization, redaction, page-token, compliance, audit, or generated error-registry row has a missing fixture checksum, missing expected output checksum, missing expected error, missing mutation-prohibition proof, `TODO`, `blocked`, `fail`, or `not_run` status.

### GraphBackendProviderValidationMatrix

Rows in this matrix validate backend-default and provider behavior owned by `090` and package-gate behavior owned by `100`. Because fixture bytes and expected output checksums are absent, checksum cells are explicit blockers until populated by `120` validation artifacts.

| Row ID | Owner | Fixture ID | Fixture checksum | Expected result | Expected output checksum | Mutation prohibition |
| --- | --- | --- | --- | --- | --- | --- |
| `120-GRAPH-BACKEND-SELECTION-AC-001` | `090` | `graph-backend-selection-omitted-default` | TODO: required before acceptance | Graph serving enabled with omitted profile materializes `mvp-janusgraph.v1`; graph serving disabled materializes no backend; explicit required mode rejects omission. | TODO: required before acceptance | no backend mutation before preflight |
| `120-GRAPH-BACKEND-PREFLIGHT-AC-001` | `090` | `graph-janusgraph-profile-missing-storage` | TODO: required before acceptance | Preflight rejects missing profile, inactive profile, checksum mismatch, omitted required fields, unpinned version, unsafe storage mode, and missing package gates. | TODO: required before acceptance | no backend mutation or query |
| `120-GRAPH-PROVIDER-CAPABILITY-AC-001` | `090` | `graph-provider-capability-matrix` | TODO: required before acceptance | Every active provider profile satisfies required capabilities or declares deterministic unsupported behavior. | TODO: required before acceptance | no unsupported query/apply/rebuild output |
| `120-GRAPH-JANUSGRAPH-SCHEMA-AC-001` | `090` | `graph-janusgraph-profile-implicit-schema` | TODO: required before acceptance | Implicit schema creation, missing schema constraints without exception, destructive schema drop, stale fingerprint, and missing index readiness fail closed. | TODO: required before acceptance | no schema mutation, graph mutation, or serving output |
| `120-GRAPH-JANUSGRAPH-QUERY-AC-001` | `090` | `graph-janusgraph-gremlin-node-detail` | TODO: required before acceptance | Gremlin translation produces provider-neutral `QueryGraph` outputs, rejects full scans, enforces candidate limits, rejects backend IDs, and sorts by Cadastre ordering. | TODO: required before acceptance | no backend ID leak and no mutation |
| `120-GRAPH-JANUSGRAPH-INDEX-AC-001` | `090` | `graph-janusgraph-mixed-index-stale` | TODO: required before acceptance | Stale or unavailable mixed indexes block affected query classes and do not alter source truth, graph truth, or derived-view advancement. | TODO: required before acceptance | no query serving for affected classes |
| `120-GRAPH-JANUSGRAPH-APPLY-AC-001` | `090` | `graph-janusgraph-partial-apply-after-storage-commit` | TODO: required before acceptance | Partial-commit fixtures require storage, composite-index, mixed-index, rollback, committed-batch, read-after-write, and resume-boundary evidence. | TODO: required before acceptance | no derived-view advancement without evidence |
| `120-GRAPH-BACKEND-PACKAGE-GATE-AC-001` | `090`, `100` | `graph-janusgraph-package-gate-failed` | TODO: required before acceptance | Graph backend profile activation fails when provider, driver, storage adapter, index adapter, SBOM, provenance, compatibility, or package-set refs are missing or failed. | TODO: required before acceptance | current active package set preserved; no graph mutation |
| `120-GRAPH-PROVIDER-PORTABILITY-AC-001` | `090` | `graph-provider-portability-equivalence` | TODO: required before acceptance | A future provider satisfies the same capability, taxonomy, query, schema, apply, and rebuild outputs without changing graph IDs, ordering, page tokens, or error categories. | TODO: required before acceptance | no provider-specific output leakage |

Required graph backend fixture families:

```text
graph-backend-selection-omitted-default
graph-backend-selection-explicit-required
graph-janusgraph-profile-missing-storage
graph-janusgraph-profile-implicit-schema
graph-janusgraph-schema-fingerprint-stale
graph-janusgraph-gremlin-node-detail
graph-janusgraph-gremlin-neighbor-expansion
graph-janusgraph-gremlin-bounded-path
graph-janusgraph-full-scan-forbidden
graph-janusgraph-mixed-index-stale
graph-janusgraph-partial-apply-after-storage-commit
graph-janusgraph-raw-write-bypass
graph-janusgraph-backend-id-leak
graph-janusgraph-package-gate-failed
graph-provider-portability-equivalence
```

### Required closure validation families

| Validation family | Required structured-input closure content |
| --- | --- |
| `120-OCSF-MAP-*` | One positive, missing-row, ambiguous-row, enum, forbidden-field, source-extension, `cadastre_only`, and endpoint-order fixture per active MVP row family. |
| `120-GOLD-PREDICATE-CATALOG-*` | Active row inventory totality, missing row, ambiguous row, subject/object boundary, reference eligibility, null rejection, identity-like string rejection, structured schema required, OCSF non-authority, source authority required, source dataset required, replay checksum drift, deterministic block rows, observed-connection graph handoff, and manifest completeness fixtures. |
| `120-SOURCE-CLOSURE-*` | One closure-positive, closure-missing, closure-ambiguous, deterministic-block, weak-signal, stale-source, coverage-missing, coverage-domain-token, feed-category-token-map, coverage-domain-manifest, source-history coverage, external-schema authority signal, manifest omission, private-binding leak, and watermark-block fixture per active absence-sensitive feed category, source dataset, requested effect, scope selector, read target kind, and upstream evidence class tuple. |
| `120-COVERAGE-DOMAIN-*` | Positive canonical tokens, display/legacy alias rejection, unknown-token rejection, invalid grammar rejection, duplicate-token rejection, lexical sort checksum parity, feed-category mapping totality, unsupported category/token pairs, inactive reachability blocking, manifest omission, and generated error-registry parity. |
| `120-GRAPH-PROFILE-CLOSURE-*` | Edge set exactly `observed_connection`, theoretical reachability prohibited, all other edge types inactive or rejected, missing flow role no edge, and ambiguous flow role no edge. |
| `120-GRAPH-JANUSGRAPH-*` | Backend selection, unresolved default, storage/index declaration, Gremlin translation, schema/index apply, stale mixed index, raw-write bypass, backend ID leak, package gate, partial apply, and rebuild equivalence. |
| `120-PACKAGE-TYPE-*` | Confirmed enum success, duplicate-token rejection, generic-label rejection, legacy-label rejection, unknown token rejection, missing policy, inactive policy, scope-mismatched policy, ambiguous policy, policy manifest omission, policy-bundle substitution, coverage totality, and exact policy success. |
| `120-LIFECYCLE-*` | `030`, `070`, `090`, `100`, and validation acceptance lifecycle totality, idempotency, illegal transition, no mutation, and manifest inclusion. |
| `120-RUNLOCK-CLOSE-*` | Lease defaults, timing bounds, all-or-nothing acquisition, active conflict, heartbeat, stale recovery, recovery race, old-holder fencing, idempotency, commit guard, manifest inclusion, watermark block, graph apply handoff, package activation handoff, maintenance handoff, non-authority, and observability non-authority. |
| `120-IDENTITY-CLOSURE-*` | Resolver profile row selection, evidence class totality, scope coverage, durable create/attach/merge, weak rejection, selector safety, blocker precedence, decision row missing/ambiguous, review totality, review expiration, split handoff, explanation checksum, replay exactness, manifest omission, package weak-default weakening, and two-independent-implementer parity. |
| `120-ERROR-REGISTRY-*` | No owner error fragment contains `TODO`; final owners include `010`, `020`, `030`, `040`, `050`, `060`, `070`, `080`, `090`, `100`, `110`, `120`, `130`, and `140`; duplicate owner decisions are enforced; generic code is rejected when owner-specific code exists; `100` package owner rows and `110` package observable rows pass parity validation. |

### Owner-specific fixture families

| Owner | Required fixture families |
| --- | --- |
| `020` | feed read, feed lifecycle event derivation, manifest validation, read completeness, raw ID replay, omitted object, partial known gap, feed category closure, feasibility assessment, missing profile field, target-kind omission, declared subset, empty-scope, object/partition known gap, unknown gap, private-binding leak. |
| `030` | DAG ordering, lifecycle transition, lifecycle totality, lifecycle idempotency, lifecycle conflict, version manifest ID/checksum, run lock failure, runlock lease defaults, runlock heartbeat, runlock stale recovery, runlock recovery race, runlock old-holder fencing, runlock idempotency, runlock manifest omission, runlock commit guard, forbidden output, declared subset profile missing, subset scope mismatch, subset output forbidden. |
| `050` | OCSF mapping row success, missing row, ambiguous row, activity discriminator missing, required object path missing, forbidden field, unknown enum, OCSF Other not permitted, deprecated field, source extension rule success, undeclared source extension, wildcard rejection, secret scan, cadastre-only null profile, DHCP/IPAM split required. |
| `060` | absence, coverage, coverage-domain token grammar, alias rejection, unknown-token rejection, duplicate-token rejection, feed-category mapping totality, inactive reachability blocking, coverage-domain manifest inclusion, progress signal, source staleness, control result mapping, feed completeness row, effect gate, blocking precedence, source-specific coverage domains, watermark gating, OCSF status non-authority, OCSF severity non-authority, OCSF confidence non-authority, OCSF observable non-authority, OCSF field absence non-authority. |
| `070` | identity-create, identity-attach, identity-durable-merge, identity-weak-rejection, identity-evidence-totality, identity-scope-coverage, identity-hard-blocker, identity-hard-blocker-row-missing, identity-hard-blocker-row-ambiguous, identity-candidate-overflow, identity-decision-row-missing, identity-decision-row-ambiguous, identity-confidence-band, identity-review-state-machine, identity-review-expiration, identity-selector-safety, identity-split-handoff, identity-explanation-checksum, identity-replay, identity-manifest-artifact-omission, identity-package-artifact-core-conflict, identity-package-weak-default-weakening, identity-two-implementer-parity. |
| `080` | temporal-resolution, knowledge-time-import, late-arrival, gold-correction, assertion-transition, correction-snapshot-ref, gold-correction-no-op-error, replay-equivalence, graph-handoff, temporal-version-manifest, gold-predicate-rowset-totality, gold-predicate-missing, gold-predicate-ambiguous, gold-predicate-boundary, gold-predicate-structured-schema, gold-predicate-block-row, gold-predicate-manifest, gold-predicate-replay. |
| `090` | active profile exact edge set, observed-connection positive projection, missing/ambiguous flow-role no-edge, unresolved endpoint no-edge, traversal behavior, graph apply lifecycle, graph apply ordering, page-token behavior, query candidate limit, rebuild equivalence, backend taxonomy rejection, reachability prohibition, backend ID rejection, OCSF endpoint-order no graph direction, generic external payload non-pathfinding, source-kind cleanup safety, backend selection omitted/defaulted, JanusGraph profile preflight, provider capability matrix, Gremlin translation parity, implicit schema rejection, schema fingerprint mismatch, storage mode unsafe, stale mixed index, full-scan forbidden, transaction partial-apply evidence, raw-write bypass, provider package-gate failure, and future-provider parity. |
| `100` | package type policy, repository form, trust, transparency, attestation, SBOM, dependency lock, compatibility, deprecation, health/LKG, rollback, quarantine, emergency, package manifest completeness, canary and shadow isolation. |
| `110` | API outcome, redaction, paging, state labels, authorization non-leakage. |
| `140` | observability-profile-valid, observability-profile-missing, observability-signal-forbidden, observability-attribute-forbidden, observability-cardinality-unbounded, observability-exporter-outage, observability-health-degraded, observability-replay-different-trace-id, observability-audit-export-failure, observability-version-manifest-missing, observability-package-policy-invalid. |
| `130` | analysis finding non-authority, analysis mutation, metric non-authority, metric risk-score forbidden, risk acceptance no-remediation, analysis replay exact/mismatch, threat-intel known indicator, threat-intel identity forbidden, threat-intel distribution restricted/unmapped, threat-intel unknown taxonomy, threat-intel missing profile, threat-intel artifact checksum mismatch, threat-intel sighting no-completeness, lineage run/job/dataset, lineage custom policy missing, lineage mutable schema, lineage checksum mismatch, lineage namespace collision, lineage freshness no-completeness, lineage facet-only not-evidence, registry activation positive, inactive artifact, owner mismatch, label no-fact-authority, custom-property bounds, custom-property redaction, classification authority forbidden, package-set-ref required, private binding redaction, derived-edge supporting facts required, `090` routing, outside-`090` forbidden, `080` routing, exact replay, replay mismatch, explicit no-op, and risk scoring boundary. |

### ResolverActivationReportValidationMatrix

`ResolverActivationReportValidationMatrix` validates promotion eligibility for active identity resolver catalogs. Rows with `TODO` fixture or expected checksum values are blockers and must not be marked passing.

| scenario_id | fixture_checksum | expected_decision_checksum | expected_explanation_checksum | expected_review_or_handoff_refs | result | promotion_eligibility |
| --- | --- | --- | --- | --- | --- | --- |
| `activation-070-creation` | TODO | TODO | TODO | none | blocked | not_eligible |
| `activation-070-attachment` | TODO | TODO | TODO | none | blocked | not_eligible |
| `activation-070-durable-merge` | TODO | TODO | TODO | none | blocked | not_eligible |
| `activation-070-weak-evidence-rejection` | TODO | TODO | TODO | none | blocked | not_eligible |
| `activation-070-blocker-precedence` | TODO | TODO | TODO | blocker fixture refs | blocked | not_eligible |
| `activation-070-overflow` | TODO | TODO | TODO | overflow error refs | blocked | not_eligible |
| `activation-070-review-totality` | TODO | TODO | TODO | review case refs | blocked | not_eligible |
| `activation-070-split-handoff` | TODO | TODO | TODO | graph correction handoff refs | blocked | not_eligible |
| `activation-070-selector-safety` | TODO | TODO | TODO | unresolved target refs | blocked | not_eligible |
| `activation-070-explanation-checksum-replay` | TODO | TODO | TODO | explanation refs | blocked | not_eligible |
| `activation-070-shadow-canary-determinism` | TODO | TODO | TODO | resolver shadow run refs | blocked | not_eligible |

A report is promotion-eligible only when every required scenario row is `pass`, every checksum is non-`TODO`, and every required review or handoff ref is manifest-included.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `120-API-SCHEMA-TOTAL-AC-001` | `ApiSurfaceClosureValidationMatrix` has coverage for every exported `110` request and response schema and fails while any required checksum or expected error is missing. |
| `120-STATE-LABEL-TOTAL-AC-001` | `SourceStateLabelTotalityValidationMatrix` has one row per declared `110.SourceStateLabel` and proves `conflicted` and `ambiguous` do not collapse. |
| `120-DEFINE-ONCE-CLOSURE-AC-001` | `DefineOnceClosureValidationMatrix` fails non-owner runtime restatement, unexported owner refs, duplicate owner exports, duplicate ledger rows, owner contradictions, and reference-as-runtime-authority violations. |
| `120-DEFINE-ONCE-CLOSURE-AC-002` | Every Section 25 ledger row has exactly one matching `DomainSection25StatusValidationMatrix` row and one owner-local closure state. |
| `120-DEFINE-ONCE-CLOSURE-AC-003` | Every imported contract, Section 25 owner contract, validation-row contract, and `SpecSetVersion.validation_matrix_refs` entry resolves to exactly one owner export or declared owner-export alias. |
| `120-IDENTITY-ACTIVATION-CATALOG-AC-001` | Aggregate acceptance fails unless every active identity resolver artifact row set has passing validation refs and non-`TODO` fixture and output checksums. |
| `120-IDENTITY-HARD-BLOCKER-AC-001` | Every hard blocker family has fired and not-fired fixtures proving precedence before confidence and review. |
| `120-IDENTITY-DECISION-MATRIX-AC-001` | Missing and ambiguous decision rows fail before mutation. |
| `120-IDENTITY-REVIEW-EXPIRATION-AC-001` | Review expiration emits deterministic transition evidence and no identity mutation. |
| `120-IDENTITY-SELECTOR-SAFETY-AC-001` | OpenGraph property matching, graph keys, mapped targets, and deprecated name matching cannot create endpoint identity. |
| `120-ENDPOINT-OUTCOME-TOTAL-AC-001` | `EndpointOutcomeValidationMatrix` has one row per `110.EndpointOutcomeMatrix` endpoint and verifies success, empty, unauthorized, stale, partial, conflict, ambiguity, pagination, redaction, and error precedence. |
| `120-ERROR-REGISTRY-TOTAL-AC-001` | `ErrorCodeRegistryValidationMatrix` covers every final owner error fragment row from `010`, `020`, `030`, `040`, `050`, `060`, `070`, `080`, `090`, `100`, shared `110` rows, `120`, `130`, and `140`; it rejects missing, duplicate, TODO, legacy `code` caller fields, wildcard-only fixture refs, generic substitutions, unredacted sensitive classes, missing registry checksum, nondeterministic bytes, or unfixtured registry rows. |
| `120-GOLD-PREDICATE-CATALOG-TOTAL-AC-001` | `GoldFactPredicateCatalogValidationMatrix` contains every row required by `080.MVPGoldFactPredicateContractRowSetClosure`, and aggregate acceptance fails for any missing, blocked, TODO-bearing, checksum-mismatched, package-set-mismatched, or manifest-incomplete predicate-catalog row. |
| `120-COVERAGE-DOMAIN-TOKEN-AC-001` | Every canonical token validates and every legacy/display value rejects with the expected owner-specific error. |
| `120-COVERAGE-DOMAIN-TOKEN-AC-002` | Token arrays reject duplicates and produce byte-identical canonical checksums after lexical sorting. |
| `120-COVERAGE-DOMAIN-FEED-MAP-AC-001` | Every feed category in `020` has exactly one mapping row or deterministic block row. |
| `120-COVERAGE-DOMAIN-REACHABILITY-AC-001` | `reachability` remains inactive deferred and produces no fact, graph edge, graph property, API output, package activation effect, or user-facing reachability claim. |
| `120-COVERAGE-DOMAIN-MANIFEST-AC-001` | Manifest omission or checksum mismatch for selected coverage-domain token arrays fails before dependent output. |
| `120-COVERAGE-DOMAIN-ERROR-REGISTRY-AC-001` | Every coverage-domain owner error has exactly one generated `110.ErrorCodeRegistryRow`. |
| `120-ANALYSIS-OUTPUT-AUTHORITY-AC-001` | Validation rows prove `AnalysisFinding`, `AnalysisMetric`, and `RiskAcceptanceRecord` are non-authoritative and cannot mutate facts, graph, completeness, watermarks, identity, package state, or source authority. |
| `120-THREAT-INTEL-COVERAGE-AC-001` | Validation rows prove threat-intel known, unknown, missing-profile, distribution, artifact checksum, sighting, redaction, and identity-forbidden behavior. |
| `120-LINEAGE-FACET-COVERAGE-AC-001` | Validation rows prove lineage facet policy, schema immutability, checksum, namespace collision, freshness no-completeness, facet-only non-evidence, and redaction behavior. |
| `120-REGISTRY-GOVERNANCE-COVERAGE-AC-001` | Validation rows prove registry activation, inactive artifact, owner mismatch, custom-property bounds, classification authority rejection, package-set refs, and private binding redaction. |
| `120-DERIVED-GRAPH-EDGE-COVERAGE-AC-001` | Validation rows prove derived-edge supporting-fact rejection, `090` routing, direct mutation rejection, `080` routing, exact replay, replay mismatch, and explicit no-op behavior. |
| `120-SOURCE-DATASET-CATALOG-AC-004` | Source-dataset catalog fixtures cover valid resolution, missing, ambiguous, deterministic block, private leak, unsupported category, manifest omission, and package-set omission. |
| `120-SOURCE-EFFECT-CLOSURE-AC-004` | Acceptance fails when any source-dataset catalog row, category/effect closure row, source-authority chain row, package-set ref, fixture checksum, expected output checksum, error row, mutation-prohibition proof, or `VersionManifest` ref is missing, stale, failed, blocked, checksum-mismatched, or `TODO`-bearing. |
| `120-OBSERVABILITY-COVERAGE-AC-001` | Observability validation rows prove telemetry non-authority, redaction, cardinality, exporter failure, health mapping, replay exclusion, audit independence, version-manifest completeness, and package activation gating. |
| `120-GRAPH-BACKEND-SELECTION-AC-001` | Graph serving enabled with omitted backend profile materializes `mvp-janusgraph.v1`; graph serving disabled materializes no backend; explicit required mode rejects omission. |
| `120-GRAPH-BACKEND-PREFLIGHT-AC-001` | Backend preflight rejects missing profile, inactive profile, checksum mismatch, omitted required profile fields, unpinned provider version, unsafe storage mode, and missing package gates before mutation or query. |
| `120-GRAPH-PROVIDER-CAPABILITY-AC-001` | Every active graph provider profile satisfies the required provider capability matrix or declares deterministic unsupported behavior for each query/apply/rebuild class. |
| `120-GRAPH-JANUSGRAPH-SCHEMA-AC-001` | JanusGraph implicit schema creation, missing schema constraints without exception, destructive schema drop, stale fingerprint, and missing index readiness fail closed. |
| `120-GRAPH-JANUSGRAPH-QUERY-AC-001` | JanusGraph Gremlin translation fixtures produce provider-neutral `QueryGraph` outputs, reject full scans, enforce candidate limits, reject backend IDs, and sort by Cadastre ordering. |
| `120-GRAPH-JANUSGRAPH-INDEX-AC-001` | Stale or unavailable mixed indexes block affected query classes and do not alter source truth, graph truth, or derived-view advancement. |
| `120-GRAPH-JANUSGRAPH-APPLY-AC-001` | JanusGraph partial-commit fixtures require storage, composite-index, mixed-index, rollback, committed-batch, read-after-write, and resume-boundary evidence. |
| `120-GRAPH-BACKEND-PACKAGE-GATE-AC-001` | Graph backend profile activation fails when required JanusGraph provider, driver, storage adapter, index adapter, SBOM, provenance, compatibility, or package-set refs are missing or failed. |
| `120-GRAPH-PROVIDER-PORTABILITY-AC-001` | A future provider fixture can satisfy the same `GraphProviderCapabilityMatrix`, `GraphBackendTaxonomyMappingProfile`, `GraphQueryTranslationProfile`, `GraphReadModelSchemaProfile`, `GraphApplyProfile`, and `GraphRebuildManifest` outputs without changing Cadastre graph IDs, ordering, page tokens, or error categories. |
| `120-AUTH-REDACTION-MATRIX-AC-001` | Authorization and redaction fixture families prove no-existence-leak handling, raw-payload blocking, private-binding redaction, backend-ID rejection, and package evidence redaction. |
| `120-PAGE-TOKEN-LIFETIME-AC-001` | Page-token fixtures prove default TTL, lower and upper bounds, expiration rejection, malformed token rejection, context mismatch rejection, and no refresh of expired tokens. |
| `120-CLEANUP-AC-001` | No banned reference class remains. |
| `120-CLEANUP-AC-002` | `SPECSET-AC-*` rows cover ownership, imports/exports, define-once, lakehouse boundary, negative tests, deferred reachability, and recreatability. |
| `120-CLEANUP-AC-003` | `RunValidationMatrix` output remains deterministic and includes row ID, owner spec, fixture checksum, input checksum, expected output checksum, actual output checksum, result, failure code, and version manifest ref. |
| `120-SCHEMA-PATCH-AC-001` | Every 040 exported core record has at least one success fixture and one required-field rejection fixture. |
| `120-SCHEMA-PATCH-AC-002` | Every nullable field has a null acceptance or null rejection fixture matching its 040 schema row. |
| `120-SCHEMA-PATCH-AC-003` | Every declared default has a materialization fixture. |
| `120-SCHEMA-PATCH-AC-004` | Every record ID algorithm has a replay fixture and collision fixture. |
| `120-SCHEMA-PATCH-AC-005` | Every record checksum policy has a replay-equivalence fixture. |
| `120-SCHEMA-PATCH-AC-006` | The two-independent-implementer check covers raw, silver, canonical entity, source asset, identifier, gold, evidence, and graph delta fixtures. |
| `120-SCHEMA-PATCH-AC-007` | The aggregate `AcceptanceReport` cannot pass while any 040 schema row, fixture checksum, expected output checksum, or owner error code remains `TODO:`. |
| `120-CLEANUP-AC-004` | Required negative tests by owner remain present and do not depend on external product drafts, decomposition-control artifacts, private binding indexes, or promotion-readiness gates. |
| `120-TEMPORAL-CORRECTION-AC-001` | Aggregate acceptance fails while any `080` temporal, correction, late-arrival, replay, no-op/error, graph handoff, or manifest fixture is blocked, missing checksum, missing expected output, or not run. |
| `120-ASSERTION-TRANSITION-AC-001` | Every assertion-state/correction-event pair has one expected result. |
| `120-REPLAY-OUTPUT-CLASS-AC-001` | Replay rows exist for raw, silver, identity, gold, gold correction, graph delta, graph apply, graph rebuild, API response, export projection, analysis output, and validation acceptance. |
| `120-GRAPH-HANDOFF-AC-001` | Every graph handoff effect has success, error, or no-op validation. |
| `120-NOOP-ERROR-AC-001` | No-op and deterministic error cases prove no forbidden fact, graph, export, replay, or watermark mutation. |
| `120-STATUS-CONSISTENCY-AC-001` | Owner/domain status contradiction, domain runtime restatement, owner-spec contradiction, ADR status, and manifest path mismatch validation rows exist and fail with exact codes. |
| `120-MUTATION-PROHIBITION-AC-001` | Every negative validation row has a mutation-prohibition proof for each affected mutation class. |
| `120-ACCEPTANCE-PRECEDENCE-AC-001` | Aggregate `AcceptanceReport` cannot pass while any non-deferred required row is `fail`, `blocked`, or `not_run`, or while any required fixture checksum or expected output checksum is `TODO`. |
| `120-VOLATILITY-AC-001` | Aggregate `AcceptanceReport` fails while any non-deferred volatility row is `fail`, `blocked`, `not_run`, or missing expected checksum. |
| `120-VOLATILITY-AC-002` | Validation rows cover inactive, missing, mismatched, out-of-scope, and runtime-restating activation artifacts. |
| `120-FEED-CLOSURE-AC-001` | Aggregate acceptance fails while any active feed category lacks a closure row, deterministic block row, validation refs, or fixture refs. |
| `120-FEED-CLOSURE-AC-002` | Every profile-dependent branch in `020`, `030`, and `060` has a validation row with expected receipt state, error/no-op, user-facing label, and mutation prohibition. |
| `120-FEED-CLOSURE-AC-003` | Every active feed category has positive and negative fixtures for complete, partial, stale, permission-limited, missing-profile-row, missing-upstream-evidence, weak-progress, empty-complete, and watermark-blocked cases. |
| `120-FEED-CLOSURE-AC-004` | Every absence-sensitive output has exact authority, completeness, coverage, staleness, and watermark fixtures where applicable. |
| `120-SOURCE-CLOSURE-AC-001` | Every active absence-sensitive feed category has closure validation rows or deterministic block rows. |
| `120-SOURCE-CLOSURE-AC-002` | Every missing, ambiguous, inactive, or checksum-mismatched source-authority closure component produces the expected owner error or no-op and no forbidden mutation. |
| `120-SOURCE-CLOSURE-AC-003` | Two independent implementations produce byte-identical `AbsenceDerivationResult`, `VersionManifest`, and expected output checksums for the same closure fixtures. |
| `120-SOURCE-CLOSURE-AC-004` | `ValidateSpecSet` fails promotion when `domain.md`, `000`, `020`, `030`, `040`, `060`, `080`, `090`, or `110` disagree on source-authority closure state. |
| `120-SOURCE-CLOSURE-AC-005` | Every known feed category has a closure-positive fixture or deterministic-block fixture. |
| `120-SOURCE-CLOSURE-AC-006` | Every active category/effect/evidence-class tuple covers all six feed receipt states and all eight upstream evidence states. |
| `120-SOURCE-CLOSURE-AC-007` | `source_history` no-change proof fails unless both retention and coverage rows validate. |
| `120-SOURCE-CLOSURE-AC-008` | A `SourceAuthorityClosureMatrix` validation result without all underlying row refs fails manifest validation. |
| `120-SOURCE-CLOSURE-AC-009` | A deterministic block row proves no mutation across raw, silver, identity, gold, graph, watermark, compliance export, API state label, and audit mutation classes. |
| `120-LIFECYCLE-AC-001` | Every production-affecting lifecycle machine has executable event-sequence fixtures. |
| `120-LIFECYCLE-AC-002` | Aggregate `AcceptanceReport` cannot pass while any required lifecycle fixture is blocked, not run, failed, missing checksum, or missing expected output. |
| `120-LIFECYCLE-AC-003` | Validation acceptance lifecycle matrix is total and idempotent. |
| `120-OCSF-MAP-AC-001` | `AcceptanceReport` cannot pass while any active `ObservationToOCSFMappingRow` lacks positive and negative fixtures. |
| `120-OCSF-MAP-AC-002` | `AcceptanceReport` cannot pass while any mapping fixture checksum or expected output checksum is `TODO`. |
| `120-OCSF-MAP-AC-003` | `AcceptanceReport` cannot pass while any active mapping row lacks `VersionManifest` artifact refs. |
| `120-SOURCE-EXT-AC-001` | `AcceptanceReport` cannot pass while any emitted source-extension path lacks a matching rule fixture and undeclared-path rejection fixture. |
| `120-OCSF-NONAUTH-AC-001` | `AcceptanceReport` cannot pass unless OCSF status, severity, confidence, observables, enrichments, and field absence non-authority fixtures pass. |
| `120-OCSF-DIRECTION-AC-001` | `AcceptanceReport` cannot pass unless OCSF endpoint-order graph-direction rejection fixtures pass. |
| `120-OCSF-MAP-AC-004` | `AcceptanceReport` cannot pass while any active mapping validation row lacks `fixture_id`, `fixture_checksum`, `input_artifact_refs`, selected or blocked mapping row refs, expected output checksum or expected error, mutation prohibition, version manifest refs, acceptance criterion, or non-blocking status. |
| `120-OCSF-DIRECTION-AC-002` | Endpoint-order direction fixtures must be produced from active mapping-row fixtures, not graph-only synthetic inputs. |
| `120-LIFECYCLE-AC-004` | Domain/owner lifecycle closure status contradiction fails with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| `120-IDENTITY-CLOSURE-AC-001` | Aggregate acceptance fails while any `070` resolver row coverage, evidence class, scope, decision matrix, confidence band, review routing, split policy, explanation policy, activation report, or selector safety fixture is missing, blocked, failed, not run, checksum-mismatched, or has a `TODO` expected output. |
| `120-IDENTITY-CLOSURE-AC-002` | Validation proves deterministic resolver pair ordering and byte-identical decision/explanation checksums across two independent implementations. |
| `120-IDENTITY-CLOSURE-AC-003` | Overflow fixtures prove no identity mutation and no auto-merge from overflowed candidates. |
| `120-IDENTITY-REVIEW-AC-001` | Every review state/event pair is covered and illegal transitions emit no mutation. |
| `120-IDENTITY-SPLIT-AC-001` | Split handoff fixtures prove required `GraphCorrectionHandoff` fields, no resolver graph mutation, and deterministic replay. |
| `120-IDENTITY-EXPLANATION-AC-001` | Resolver explanation checksum fixtures include every required output-affecting field and exclude non-output volatile fields. |
| `120-IDENTITY-PACKAGE-AC-001` | Package-supplied resolver artifact weakening fails activation and preserves the current active package set. |
| `120-GRAPH-PROFILE-CLOSURE-AC-001` | Aggregate acceptance fails while any non-deferred `GraphActiveProfileClosureValidationMatrix` row is failed, blocked, not run, missing fixture checksum, missing expected-output checksum, or missing mutation-prohibition proof. |
| `120-GRAPH-QUERY-TRANSLATION-AC-001` | Graph query candidate-limit, traversal-class, page-token expiry, and page-token mismatch fixtures pass before graph serving promotion. |
| `120-GRAPH-OUTPUT-ELIGIBILITY-AC-001` | Output eligibility fixtures prove generic external payloads and synthetic structural objects are not pathfinding, finding, metric, or identity inputs in MVP. |
| `120-GRAPH-APPLY-ORDER-AC-001` | Apply-order fixtures prove byte-identical graph delta ordering across two independent implementations. |
| `120-GRAPH-REBUILD-EQUIVALENCE-AC-001` | Rebuild fixtures prove profile, schema, index, query, eligibility, taxonomy, property, and output checksum equivalence. |
| `120-GRAPH-ENDPOINT-IDENTITY-AC-001` | Endpoint identity fixtures prove unresolved target hints, graph keys, and selector-only evidence emit no graph endpoint. |
| `120-GRAPH-PAGE-TOKEN-AC-001` | Page-token fixtures prove TTL, expiry, mismatch, authorization context, derived-view state, and profile-ref identity behavior. |
| `120-PACKAGE-ACTIVATION-AC-001` | Aggregate acceptance fails while any required package activation fixture is blocked, not run, failed, missing fixture checksum, missing expected-output checksum, missing expected error, or missing mutation-prohibition proof. |
| `120-PACKAGE-MATRIX-AC-001` | Every `PackageActivationValidationMatrix` row has fixture checksum, expected output checksum, expected error or no-op, mutation prohibition, owner spec, and version manifest requirement. |
| `120-PACKAGE-TYPE-COVERAGE-AC-001` | Package type coverage validation fails when any confirmed token is absent, duplicated, assigned to more than one family, or lacks exactly one active policy row for the target environment. |
| `120-PACKAGE-MANIFEST-OMISSION-AC-001` | Required release and package-set manifest field omission rows remain blocking until fixture checksum and expected output checksum values are non-`TODO` SHA-256 values. |
| `120-PACKAGE-ERROR-REGISTRY-PARITY-AC-001` | Package error-registry parity validation proves every `100` package error appears exactly once in `110` observable mapping and no `110` package row lacks a `100` owner fragment. |
| `120-DEFINE-ONCE-CLOSURE-AC-001` | The spec set cannot pass while any domain Section 25 row, owner closure row, required validation fixture, expected output checksum, expected no-op/error assertion, or owner error fragment contains `TODO` in required promotion scope. |
| `120-DEFINE-ONCE-CLOSURE-AC-002` | No active spec restates another active spec's runtime behavior except by exact imported contract name and one-sentence routing reference; violations fail with `DOMAIN_RUNTIME_RESTATEMENT` or `OWNER_SPEC_CONTRADICTION` as applicable. |
| `120-CORE-ONEOF-CLOSURE-AC-001` | Every subject, object, and evidence artifact one-of fixture family listed in `CoreOneOfClosureValidationMatrix` and `CoreEvidenceArtifactValidationMatrix` passes. |
| `120-CORE-ONEOF-CLOSURE-AC-002` | Expected output checksum, expected error, expected no-op, fixture checksum, activation artifact ref, and version manifest ref values cannot be `TODO`, omitted, stale, or checksum-mismatched when `pass_fail_evidence = pass`. |
| `120-CORE-ONEOF-CLOSURE-AC-003` | Two independent implementations produce byte-identical IDs and checksums for every one-of closure fixture. |
| `120-RUNLOCK-CLOSE-AC-001` | Aggregate acceptance fails while any run-lock row is `blocked`, `not_run`, `fail`, stale, checksum-mismatched, or `TODO`. |
| `120-RUNLOCK-CLOSE-AC-002` | Every run-lock negative row has mutation prohibition for production output, watermark, graph mutation, package activation, and table maintenance where applicable. |
| `120-RUNLOCK-CLOSE-AC-003` | Two implementations produce byte-identical evidence for idempotent retry and stale recovery success fixtures. |

### Structured input validation acceptance criteria

| ID | Criterion |
| --- | --- |
| `120-STRUCTURED-INPUT-AC-001` | `AcceptanceReport` cannot pass while any structured-input validation row is `blocked`, `not_run`, `fail`, stale, checksum-mismatched, or `TODO`. |
| `120-STRUCTURED-INPUT-AC-002` | Exact same repository snapshot inputs produce byte-identical validation output. |
| `120-STRUCTURED-INPUT-AC-003` | Merge to repository alone produces no production mutation. |
| `120-STRUCTURED-INPUT-AC-004` | Materialized package activation requires allowed repository form, materialization refs, package release refs, package-set refs, and `VersionManifest` refs. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `120-SCOPE-SELECTOR-CLOSURE-AC-001` | Every active scoped owner family provides exact-match, subset-allowed, subset-disallowed, duplicate-dimension, duplicate-value, private-leak, missing-row, ambiguous-row, owner-error-mapping, redaction, and manifest-inclusion fixtures. |
| `120-SCOPE-SELECTOR-CLOSURE-AC-002` | `AcceptanceReport.result = pass` is forbidden when any active scoped owner family lacks required `ScopeSelectorValidationMatrix` fixture coverage. |
| `120-AC-001` | Every domain NLSpec has binary acceptance criteria and validation matrix rows for success, rejection, no-op, and edge behavior. |
| `120-AC-002` | Every required negative authority-boundary case is executable and produces the expected owner-specific error code. |
| `120-AC-003` | Golden corpus replay produces byte-identical expected outputs or deterministic failure records. |
| `120-AC-004` | Validation artifacts are redacted, checksummed, and replayable. |
| `120-AC-005` | The aggregate acceptance report is sufficient to decide whether the NLSpec set can be promoted to `authoritative`. |
| `120-AC-006` | `RunValidationMatrix` emits blocked or fail for any unresolved active feed category, unresolved profile branch, missing fixture checksum, or missing expected output checksum. |
| `120-AC-007` | Authoritative promotion is forbidden while any non-deferred identity closure row is `fail`, `blocked`, `not_run`, missing checksum, or has a `TODO` expected output. |

| `120-AC-008` | `AcceptanceReport.result = pass` requires every activation-catalog closure validation row to pass, no row to contain `TODO`, every owner row to be present, every fixture checksum to be concrete, and every expected output or expected error checksum to match observed bytes. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `120-TODO-COVERAGE-DOMAIN-FIXTURE-CHECKSUMS` | TODO: Provide concrete fixture checksums, input checksums, expected output checksums, expected error checksums, mutation-prohibition proofs, and manifest refs for every `120-COVERAGE-DOMAIN-*` fixture row. | Coverage-domain token closure, feed-category token mapping, package-supplied coverage row catalogs, generated error registry parity, API rejection, and manifest inclusion. | Product validation owners for `020`, `030`, `060`, `100`, `110`, and `120`. | Acceptance aggregation returns `blocked` for `120-COVERAGE-DOMAIN-*`; promotion remains forbidden. |
