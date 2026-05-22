# Artifact Manifest

The manifest defines the current documents for the Cadastre project.

If a file is not currently uploaded, it can be added to the context on request.

Every path must be repository-relative, normalized, unique, and must not contain absolute path prefixes, backslashes, empty segments, `.`, or `..` segments.

| Path |
| --- |
| `docs/nlspec/domain.md` |
| `docs/nlspec/000-cadastre-spec-index.md` |
| `docs/nlspec/010-system-boundary-and-authority.md` |
| `docs/nlspec/020-lakehouse-feeds-and-table-state.md` |
| `docs/nlspec/030-processing-dag-lifecycle-and-versioning.md` |
| `docs/nlspec/040-canonical-data-observation-and-fact-model.md` |
| `docs/nlspec/050-normalization-external-schema-and-mapping.md` |
| `docs/nlspec/060-source-authority-completeness-coverage-and-absence.md` |
| `docs/nlspec/070-identity-resolution-and-target-selectors.md` |
| `docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md` |
| `docs/nlspec/090-graph-projection-serving-and-backends.md` |
| `docs/nlspec/100-packages-supply-chain-and-activation.md` |
| `docs/nlspec/110-api-ux-health-errors-and-security.md` |
| `docs/nlspec/120-validation-fixtures-and-acceptance.md` |
| `docs/nlspec/130-analysis-enrichment-and-registry-governance.md` |
| `docs/nlspec/140-observability-instrumentation.md` |
| `docs/deferred/200-future-reachability-analysis-domain.md` |
| `docs/doc_clean-up/MarkdownLint-Rules.md` |
| `docs/doc_clean-up/markdownlint-cleanup-reference.md` |
| `docs/adr/ADR-0001-lakehouse-fed-boundary.md` |
| `docs/adr/ADR-0002-graph-as-derived-projection.md` |
| `docs/adr/ADR-0003-ocsf-as-silver-profile.md` |
| `docs/adr/ADR-0004-package-set-activation.md` |
| `docs/reference/standards/nlspec-spec.md` |
| `docs/reference/research/RES-001-cartography.md` |
| `docs/reference/research/RES-002-jupiterone.md` |
| `docs/reference/research/RES-003-taxi-lang.md` |
| `docs/reference/research/RES-004-ocsf-schema.md` |
| `docs/reference/research/RES-005-Lakehouse-Derived-Graph-Architecture.md` |
| `docs/reference/research/RES-006-source-adapter-ingestion-patterns.md` |
| `docs/reference/research/RES-007-normalization-and-schema-standards.md` |
| `docs/reference/research/RES-008-identity-graph-attack-paths.md` |
| `docs/reference/research/RES-009-identity-resolution-governance.md` |
| `docs/reference/research/RES-010-theoretical-reachability-network-context.md` |
| `docs/reference/research/RES-011-temporal-semantics.md` |
| `docs/reference/research/RES-013-extension-package-supply-chain.md` |
| `docs/reference/research/RES-014-source-authority-staleness-coverage-absence.md` |
| `docs/reference/research/RES-016-batfish.md` |
| `docs/reference/research/RES-017-postgresql-apache-age.md` |
| `docs/reference/research/RES-018-postgresql-apache-age-implementation.md` |
| `docs/archive/PRD-Cadastre.revised-draft.md` |
| `docs/archive/RES-012-graph-backend-comparison.md` |
| `docs/archive/RES-015-janusgraph.md` |

## Unresolved supporting activation-controlled artifacts

The uploaded file set does not include concrete bytes for the production source-dataset, feed-category, source-authority, validation, expected-output, mutation-prohibition, or package-set artifacts required by `000.MVPActivationCatalogClosurePack`.

Do not add synthetic paths to the path table. Product governance must supply exact repository-relative paths before these artifacts can be manifest-listed as production inputs. Until those paths and checksums exist, validation rows that depend on them must remain `blocked`, and promotion must fail.

| Required artifact family | Required future manifest action | Default until supplied |
| --- | --- | --- |
| `020.SourceDatasetCatalogRowSet` | Add the exact public source-dataset row-set path and checksum. | Referenced `source_dataset` values without selected row refs fail closed. |
| selected `020.SourceDatasetCatalogRow` refs and deterministic block rows | Add exact row artifact paths or row-set member refs supplied by governance. | No source-dataset row or block row may affect output. |
| `020.LakehouseFeedCategoryClosureRowSet` | Add the exact feed-category closure row-set path and checksum. | Feed category closure remains blocked for production. |
| `060.SourceAuthorityClosureMatrixRowSet` and underlying `060` row sets | Add exact closure-matrix, authority, completeness, coverage, staleness, progress-signal, visibility, control-result, source-history, absence, external-schema-authority, and watermark row-set paths. | Absence-sensitive effects remain blocked or no-op. |
| validation fixtures and expected outputs | Add exact fixture, expected-output, expected-error, and mutation-prohibition proof paths with checksums. | Validation rows with `TODO:` checksums remain blocked. |
| package release and production package-set manifests | Add exact paths when any closure artifact is package-supplied. | Package-supplied rows cannot activate. |

