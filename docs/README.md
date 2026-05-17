# Cadastre NLSpec Documentation Set

Generated on 2026-05-17 from the Cadastre PRD, research reports, migration ledgers, and NLSpec decomposition material supplied for the migration-finalization stream.

## Status

This is a migration-active documentation set. It is intended to replace the monolithic PRD only after `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence and `docs/nlspec/000-cadastre-spec-index.md` marks the applicable NLSpec documents `authoritative`.

The current cutover state is not defined by this README. The current cutover state must be read from `docs/nlspec/000-cadastre-spec-index.md`.

## Document classes

| Document class | Paths | May define runtime behavior | Required use |
| --- | --- | ---: | --- |
| `authoritative_spec` | `docs/nlspec/*.md` with `status = authoritative` in `000` | Yes | Implementation authority for owned contracts only. |
| `candidate_spec` | `docs/nlspec/*.md` with `status = migration_active` or `candidate_authoritative` in `000` | No | Migration evidence, review target, or controlled spike input only. |
| `deferred_spec` | `docs/deferred/*.md` | No | Preserved future-domain material with no MVP effect. |
| `rationale` | `docs/adr/*.md` | No | Decision rationale only. |
| `reference` | `docs/reference/*.md` and `docs/reference/research/*.md` | No | Navigation, inventory, standards, and research evidence only. |
| `migration_support` | `docs/migration/*.md` | No | Disposition, blocker, and decision evidence only. |
| `archive` | `docs/archive/*.md` | No after cutover | Upstream intent before cutover, historical trace after cutover. |
| `navigation` | `docs/README.md` | No | Repository navigation and status pointer only. |

## How to use this documentation

Implementers must start with `docs/nlspec/000-cadastre-spec-index.md` to determine status, ownership, imports, exports, promotion gates, and cutover state. Implementers must then read the owner spec for the contract being implemented and finally consult `docs/nlspec/120-validation-fixtures-and-acceptance.md` for required validation rows and acceptance reports.

An implementation must not use a README, ADR, research report, glossary row, source-inventory row, manifest row, migration-ledger row, or archived PRD passage as runtime authority when an owning NLSpec is missing, blocked, or not authoritative. Missing authority is a blocker, not permission to infer behavior.

## Navigation

| File | Purpose |
| --- | --- |
| `docs/nlspec/000-cadastre-spec-index.md` | Cadastre Spec Index and Source-of-Truth Map |
| `docs/nlspec/domain.md` | Cadastre root domain vocabulary and owner routing |
| `docs/nlspec/010-system-boundary-and-authority.md` | System Boundary and Authority |
| `docs/nlspec/020-lakehouse-feeds-and-table-state.md` | Lakehouse Feeds and Table State |
| `docs/nlspec/030-processing-dag-lifecycle-and-versioning.md` | Processing DAG, Lifecycle, and Versioning |
| `docs/nlspec/040-canonical-data-observation-and-fact-model.md` | Canonical Data, Observation, and Fact Model |
| `docs/nlspec/050-normalization-external-schema-and-mapping.md` | Normalization, External Schema, and Mapping |
| `docs/nlspec/060-source-authority-completeness-coverage-and-absence.md` | Source Authority, Completeness, Coverage, and Absence |
| `docs/nlspec/070-identity-resolution-and-target-selectors.md` | Identity Resolution and Target Selectors |
| `docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md` | Temporal Corrections, Replay, and Gold Derivation |
| `docs/nlspec/090-graph-projection-serving-and-backends.md` | Graph Projection, Serving, and Backends |
| `docs/nlspec/100-packages-supply-chain-and-activation.md` | Packages, Supply Chain, and Activation |
| `docs/nlspec/110-api-ux-health-errors-and-security.md` | API, UX, Health, Errors, and Security |
| `docs/nlspec/120-validation-fixtures-and-acceptance.md` | Validation, Fixtures, and Acceptance |
| `docs/nlspec/130-analysis-enrichment-and-registry-governance.md` | Analysis, Enrichment, and Registry Governance |
| `docs/deferred/200-future-reachability-analysis-domain.md` | Future Reachability Analysis Domain |
| `docs/migration/PRD-to-NLSpec-ledger.md` | PRD disposition ledger |
| `docs/migration/gap-and-ambiguity-ledger.md` | Gap and ambiguity ledger |
| `docs/migration/open-decision-ledger.md` | Open product decision ledger |
| `docs/adr/` | Rationale documents |
| `docs/reference/` | Glossary, research index, source inventory, and standards references |
| `docs/archive/PRD-Cadastre.revised-draft.md` | Upstream PRD intent before cutover and historical trace after cutover |

## Do not use as authority

| Artifact class | Not authoritative for | Required handling |
| --- | --- | --- |
| README | Runtime behavior, defaults, algorithms, mappings, errors, source-of-truth status | Follow `000` and owner NLSpecs. |
| ADRs | Field shapes, algorithms, defaults, errors, validation outcomes | Use as rationale only. |
| Research reports | Runtime behavior, product decisions, backend selection, external-version claims | Treat as evidence until an owner spec adopts a named contract. |
| Glossary | Runtime definitions | Use as navigation to `domain.md` and owner specs. |
| Source inventory and hashes | Runtime behavior or source authority | Use only for artifact traceability; reconcile mismatches through `000` and the gap ledger. |
| Migration ledgers | Product behavior | Use only for migration disposition, blockers, and cutover evidence. |
| Archived PRD | Implementation authority after cutover | Use only as upstream intent before cutover and trace after cutover. |

## Acceptance criteria

| ID | Criterion |
| --- | --- |
| `README-AC-001` | Every listed path appears in `MANIFEST.md`. |
| `README-AC-002` | Source-of-truth status and cutover state are delegated to `docs/nlspec/000-cadastre-spec-index.md`. |
| `README-AC-003` | This README defines no runtime requirement not owned by an NLSpec. |
| `README-AC-004` | Every non-NLSPEC document class is marked non-authoritative for runtime behavior. |
