# PRD to NLSpec Migration Ledger

## Status

This ledger proves migration disposition evidence only. It does not define runtime behavior and does not prove final migration completion until every row has owner-spec coverage, duplicate requirements are removed or replaced by exact named references, and required validation evidence exists in `docs/nlspec/120-validation-fixtures-and-acceptance.md`.

## Row status enum

| Status | Meaning |
| --- | --- |
| `open` | Migration disposition is not complete. |
| `covered` | Target artifact owns the behavior and required validation is present or not required. |
| `covered_with_deferred_items` | Target artifact owns the active behavior and records deferred blockers. |
| `intentionally_excluded` | Source material is not part of Cadastre scope. |
| `deferred` | Source material is preserved inactive. |
| `blocked` | Disposition cannot complete until a gap, decision, or validation row is resolved. |
| `archived` | Source material is retained as trace only. |
| `superseded` | Source material is replaced by a newer artifact. |

## Migration disposition rows

| Row ID | Source artifact or section | Target artifact | Owner contract | Coverage status | Disposition status | Blocking issue ID | Decision ID | Validation row ID | Final cutover state |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `PRD-001` | PRD front matter | `docs/nlspec/000-cadastre-spec-index.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-002` | Section 1 Product Summary | `docs/nlspec/010-system-boundary-and-authority.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-003` | Section 2 Document Status | `000, ADRs, reference/research-index.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-004` | Section 2.1 Lakehouse-Fed Boundary Amendment | `010, 020, 060` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-005` | Section 3 Assumptions | `migration/open-decision-ledger.md, ADRs` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-006` | Section 4 Scope and Non-Scope | `010 plus domain specs` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-007` | Section 4.3 Design Boundary | `010, 000` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-008` | Sections 5-6 System Context and Component Architecture | `010, README` | TODO: owner contract row required | `covered_with_deferred_items` | `covered_with_deferred_items` | none recorded | TODO | TODO | `migration_active` |
| `PRD-009` | Section 7 Data Flow | `010, 020, 030, 080, 090` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-010` | Section 8 User-Facing Behavior | `110 and 090` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-011` | Section 9 Canonical Architecture Requirements | `010, 030, 040` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-012` | Sections 10.1-10.12 | `040` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-013` | Section 10.13 and subsections | `030, 020, 080, 090, 130` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-014` | Sections 10.14-10.17 | `050` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-015` | Sections 10.18-10.20.5 | `060` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-016` | Sections 10.21 and 10.37 | `070` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-017` | Sections 10.22-10.24 | `090` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-018` | Section 10.25 and subsections | `100` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-019` | Sections 10.26-10.27 | `090, 120` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-020` | Sections 10.28-10.33 | `050` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-021` | Sections 10.34-10.35 | `020, 060, 120` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-022` | Sections 10.36 and 10.46 | `100, 030` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-023` | Sections 10.38-10.40 | `090` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-024` | Sections 10.41-10.42 | `110, 050` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-025` | Sections 10.43-10.52 | `020, 030, 060, 120` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-026` | Sections 10.52.1-10.56 | `090, 130` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-027` | Sections 10.57-10.62 | `050, 120, 130, 100` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-028` | Section 10.63 | `docs/deferred/200-future-reachability-analysis-domain.md` | TODO: owner contract row required | `deferred` | `deferred` | none recorded | TODO | TODO | `migration_active` |
| `PRD-029` | Section 11 Interfaces | `Domain specs; optional generated cross-reference` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-030` | Section 12 Defaults and Bounds | `Domain specs and 110` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-031` | Section 13 Source Categories and Normalization | `050, 060, 010` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-032` | Section 14 Identity Resolution | `070` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-033` | Section 15 Incremental Projection | `090` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-034` | Section 16 Extension Package Lifecycle | `100, 030, ADRs` | TODO: owner contract row required | `covered_with_deferred_items` | `covered_with_deferred_items` | none recorded | TODO | TODO | `migration_active` |
| `PRD-035` | Section 17 Error Model | `110 and domain specs` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-036` | Section 18 Edge Cases | `Domain specs plus 120` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-037` | Section 19 Maintenance | `120, 080, 020, 100` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-038` | Section 20 Bounds | `Domain specs and 110` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-039` | Section 21 Security | `110, 100, domain specs` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-040` | Section 22 Ambiguity Audit | `migration/gap-and-ambiguity-ledger.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-041` | Section 23 Acceptance Criteria | `120 and per-domain DoD` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-042` | Section 24 Required Product Decisions | `migration/open-decision-ledger.md and owner specs` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-043` | nlspec-spec.md | `docs/reference/standards/nlspec-spec.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-044` | RES research reports | `docs/reference/research-index.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-045` | Original PRD after migration | `docs/archive/PRD-Cadastre.revised-draft.md` | TODO: owner contract row required | `open` | `open` | none recorded | TODO | TODO | `migration_active` |
| `PRD-DOMAIN-001` | Root domain vocabulary and owner routing | docs/nlspec/domain.md | DomainTermRegistry | `covered_with_deferred_items` | `covered_with_deferred_items` | GAP-DOMAIN-001 | DEC-ROOT-DOMAIN-ROLE | TODO | `migration_active` |
| `PRD-README-001` | Navigation and document use rules | docs/README.md | README navigation contract | `covered` | `covered` | none | none | README-AC-* | `migration_active` |
| `PRD-GLOSSARY-001` | Glossary navigation separation | docs/reference/glossary.md | Glossary owner routing | `covered` | `covered` | none | none | GLOSSARY-PATCH-AC-* | `migration_active` |
| `PRD-RESEARCH-001` | Research evidence containment | docs/reference/research-index.md | Research adoption dispositions | `covered_with_deferred_items` | `covered_with_deferred_items` | none | none | RESEARCH-INDEX-PATCH-AC-* | `migration_active` |
| `PRD-SOURCEINV-001` | Uploaded source inventory and aliases | docs/reference/source-inventory.md | Source inventory reconciliation | `blocked` | `blocked` | GAP-SOURCEINV-001 | DEC-SOURCEINV-RECONCILE | SOURCEINV-PATCH-AC-* | `migration_active` |
| `PRD-MANIFEST-001` | Final documentation artifact set | MANIFEST.md | Manifest path set | `covered` | `covered` | none | none | MANIFEST-AC-* | `migration_active` |
| `PRD-ADR-001` | ADR/reference separation | docs/adr/*.md | Rationale-only boundary | `covered` | `covered` | none | none | ADR-*-PATCH-AC-* | `migration_active` |

## Acceptance criteria

| ID | Criterion |
| --- | --- |
| `PRD-LEDGER-PATCH-AC-001` | Every PRD top-level section and major Section 10 through Section 24 concept has a final disposition row. |
| `PRD-LEDGER-PATCH-AC-002` | No row claims completion while its owner spec remains blocked. |
| `PRD-LEDGER-PATCH-AC-003` | The ledger distinguishes migration evidence from implementation authority. |
