# Cadastre NLSpec Documentation Set

Generated on 2026-05-17 from `PRD-Cadastre.md` and the decomposition plan supplied in the current prompt.

## Status

This is a generated migration-active documentation set. It is intended to replace the monolithic PRD only after the migration ledger, decision ledger, and validation acceptance reports pass.

## Navigation

| File | Purpose |
| --- | --- |
| `docs/nlspec/000-cadastre-spec-index.md` | Cadastre Spec Index and Source-of-Truth Map |
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
| `docs/migration/PRD-to-NLSpec-ledger.md` | Tracks PRD section disposition. |
| `docs/migration/gap-and-ambiguity-ledger.md` | Tracks blockers and unresolved ambiguities. |
| `docs/migration/open-decision-ledger.md` | Tracks PRD Section 24 product decisions. |
| `docs/adr/` | Rationale documents. Runtime behavior lives in NLSpecs, not ADRs. |
| `docs/reference/` | Glossary, research index, source inventory, and standards references. |
| `docs/archive/PRD-Cadastre.revised-draft.md` | Superseded PRD copy retained for traceability. |

## Source-of-Truth Rule

Use `docs/nlspec/000-cadastre-spec-index.md` to determine ownership. Domain specs must import named contracts and must not restate another domain’s owned behavior.

## Current Blocker

The set contains explicit `TODO:` blockers for unresolved PRD Section 24 decisions. A document with unresolved `TODO:` rows must not be marked `authoritative`.
