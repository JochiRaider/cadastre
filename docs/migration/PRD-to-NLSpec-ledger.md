# PRD to NLSpec Migration Ledger

## Status

This ledger is generated from the decomposition plan. It proves current disposition intent; it does not prove final migration completion.

| Source artifact or section | Current role | Target artifact | Migration action | Status |
| --- | --- | --- | --- | --- |
| PRD front matter | Draft identity and related-doc index | `docs/nlspec/000-cadastre-spec-index.md` | Rewrite into document registry | `migration_active` |
| Section 1 Product Summary | Core intent and boundary | `docs/nlspec/010-system-boundary-and-authority.md` | Rewrite into normative boundary rules | `migration_active` |
| Section 2 Document Status | Mixed status and rationale | `000, ADRs, reference/research-index.md` | Split status from rationale | `migration_active` |
| Section 2.1 Lakehouse-Fed Boundary Amendment | Production input rule | `010, 020, 060` | Move normative mappings into owner specs | `migration_active` |
| Section 3 Assumptions | Preconditions and unresolved constraints | `migration/open-decision-ledger.md, ADRs` | Promote resolved items or keep as decisions | `migration_active` |
| Section 4 Scope and Non-Scope | Broad requirement inventory | `010 plus domain specs` | Split each row to owner spec | `migration_active` |
| Section 4.3 Design Boundary | Boundary inventory | `010, 000` | Rewrite into imports/exports table | `migration_active` |
| Sections 5-6 System Context and Component Architecture | Architecture diagrams | `010, README` | Keep as explanatory overview; behavior in specs | `partially_retained` |
| Section 7 Data Flow | Runtime flow | `010, 020, 030, 080, 090` | Split by stage ownership | `migration_active` |
| Section 8 User-Facing Behavior | API and UX contracts | `110 and 090` | Move API contracts to 110; graph execution to 090 | `migration_active` |
| Section 9 Canonical Architecture Requirements | Layer contract and lifecycle | `010, 030, 040` | Split by boundary, execution, model | `migration_active` |
| Sections 10.1-10.12 | Core model definitions | `040` | Move and normalize core model | `migration_active` |
| Section 10.13 and subsections | Version/lakehouse/replay/lineage models | `030, 020, 080, 090, 130` | Split by ownership | `migration_active` |
| Sections 10.14-10.17 | External schema, overlays, import, CIM | `050` | Move to normalization and mapping spec | `migration_active` |
| Sections 10.18-10.20.5 | Completeness, authority, absence | `060` | Move as authoritative core | `migration_active` |
| Sections 10.21 and 10.37 | Unresolved targets and selector safety | `070` | Move together | `migration_active` |
| Sections 10.22-10.24 | Graph projection primitives and structural nodes | `090` | Move to graph spec | `migration_active` |
| Section 10.25 and subsections | Package/supply-chain data model | `100` | Move and tighten around package-set activation | `migration_active` |
| Sections 10.26-10.27 | Graph property evidence and validation matrix | `090, 120` | Split semantics and validation | `migration_active` |
| Sections 10.28-10.33 | OCSF artifacts and policies | `050` | Move together | `migration_active` |
| Sections 10.34-10.35 | Feed corpus and completeness | `020, 060, 120` | Split feed mechanics, interpretation, fixtures | `migration_active` |
| Sections 10.36 and 10.46 | Package stage binding and developer contract | `100, 030` | Split package fields and stage execution | `migration_active` |
| Sections 10.38-10.40 | Graph edge semantics and taxonomy | `090` | Move as authoritative graph semantics | `migration_active` |
| Sections 10.41-10.42 | Diagnostics and mapping rules | `110, 050` | Split errors/health and mapping validation | `migration_active` |
| Sections 10.43-10.52 | Feed profile schema, state, lifecycle, stage, fixtures, coverage, read policy, locks, subset | `020, 030, 060, 120` | Split by behavior owner | `migration_active` |
| Sections 10.52.1-10.56 | Graph backend, schema, apply, analysis, drift | `090, 130` | Split graph mechanics and analysis outputs | `migration_active` |
| Sections 10.57-10.62 | Mapping manifest, tooling, validation scenario, validation output, registry | `050, 120, 130, 100` | Split by owner | `migration_active` |
| Section 10.63 | Future reachability | `docs/deferred/200-future-reachability-analysis-domain.md` | Move out of active set | `deferred` |
| Section 11 Interfaces | Interface catalog | `Domain specs; optional generated cross-reference` | Move each interface to owning spec | `migration_active` |
| Section 12 Defaults and Bounds | Shared defaults | `Domain specs and 110` | Move defaults to owning behavior | `migration_active` |
| Section 13 Source Categories and Normalization | Source taxonomy and external mapping | `050, 060, 010` | Split mapping, authority, boundary | `migration_active` |
| Section 14 Identity Resolution | Identity requirements | `070` | Move to identity spec | `migration_active` |
| Section 15 Incremental Projection | Projection requirements | `090` | Move to graph spec | `migration_active` |
| Section 16 Extension Package Lifecycle | Lifecycle diagram and package lifecycle prose | `100, 030, ADRs` | Move behavior; keep diagram as explanatory | `partially_retained` |
| Section 17 Error Model | Error schema and code registry | `110 and domain specs` | Move shared shape and assign codes to owners | `migration_active` |
| Section 18 Edge Cases | Cross-domain edge cases | `Domain specs plus 120` | Split edge cases by owner | `migration_active` |
| Section 19 Maintenance | Golden corpus, shadow, replay, maintenance | `120, 080, 020, 100` | Split by owner | `migration_active` |
| Section 20 Bounds | Global limits | `Domain specs and 110` | Move to owning behavior | `migration_active` |
| Section 21 Security | Security requirements | `110, 100, domain specs` | Split shared security and package trust | `migration_active` |
| Section 22 Ambiguity Audit | Resolved and unresolved ambiguity | `migration/gap-and-ambiguity-ledger.md` | Move unresolved entries to ledger | `migration_active` |
| Section 23 Acceptance Criteria | Aggregate DoD | `120 and per-domain DoD` | Split to owner specs and aggregate matrix | `migration_active` |
| Section 24 Required Product Decisions | Open decision list | `migration/open-decision-ledger.md and owner specs` | Preserve and assign owners | `blocking` |
| nlspec-spec.md | Specification-quality standard | `docs/reference/standards/nlspec-spec.md` | Retain unchanged as standard reference | `kept` |
| RES research reports | Evidence and rationale | `docs/reference/research-index.md` | Non-authoritative references | `referenced` |
| Original PRD after migration | Intent and historical trace | `docs/archive/PRD-Cadastre.revised-draft.md` | Archive after ledger reaches complete | `archived_copy_created` |

## Completion Rule

A row is complete only when the target artifact owns the behavior, duplicate definitions are removed or replaced by exact named references, and any residual blocker appears in the gap or decision ledger.
