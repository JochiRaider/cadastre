# Gap and Ambiguity Ledger

| ID | Gap or ambiguity | Why it matters | Required owner | Required resolution |
| --- | --- | --- | --- | --- |
| `GAP-001` | Filename mismatch: prompt says PRD-cadaster.md; available file is PRD-Cadastre.md. | Traceability and scripts may fail. | `000` | Normalize canonical source filename and aliases. |
| `GAP-002` | Feed feasibility catalog unresolved. | Feed activation cannot be deterministic. | `020,120` | Define feasibility rows for every MVP feed. |
| `GAP-003` | Raw supplier profile set unresolved. | Public/private source binding cannot be validated. | `010,020` | Define supplier classes and private binding rules. |
| `GAP-004` | Lakehouse read policy catalog unresolved. | Feed reads may diverge. | `020` | Define supported read target kinds and policy rows. |
| `GAP-005` | Raw feed manifest schema unresolved. | Raw import identity and replay may diverge. | `020` | Finalize counts, hashes, refs, time bounds, lineage, completeness evidence, and checksum serialization. |
| `GAP-006` | Feed completeness evidence policy unresolved. | Missing rows could become unsafe absence or unknown. | `060` | Define sufficiency by source category, dataset, fact type, predicate, and scope. |
| `GAP-007` | CDC-shaped feed metadata mapping unresolved. | CDC offsets and tombstones may be overinterpreted. | `020,030,080` | Define CDC metadata mapping, state, replay, and non-authority rules. |
| `GAP-008` | Temporal policy catalog unresolved. | Implementers may choose different valid-time and known-time sources. | `080` | Define source-dataset-specific temporal policy rows. |
| `GAP-009` | Knowledge-time import mode unresolved. | Historical import may falsely claim earlier Cadastre knowledge. | `080` | Decide reconstructed known-time policy. |
| `GAP-010` | Gold correction policy catalog unresolved. | Comparable fact matching and interval splitting may diverge. | `080` | Define per-fact correction rows and sort keys. |
| `GAP-011` | Replay equivalence policy unresolved. | Replay may pass despite mismatched output-affecting inputs. | `030,080,120` | Define included/excluded fields and failure precedence. |
| `GAP-012` | Progress signal authority matrix not finalized. | Weak operational signals could combine into unauthorized completeness. | `060` | Complete total progress-signal matrix. |
| `GAP-013` | Package repository model unresolved. | Package identity and anti-rollback behavior cannot be implemented. | `100` | Choose supported repository models and resolver/evidence rules. |
| `GAP-014` | Package trust-root governance unresolved. | Signature verification cannot determine authorized signer status. | `100` | Define trust roots, signer rotation, thresholds, issuers, scopes, approval authority. |
| `GAP-015` | Package SBOM and provenance policy unresolved. | Activation gates cannot decide package eligibility. | `100` | Define SBOM/provenance formats, subject matching, material policy, thresholds. |
| `GAP-016` | Package cohesion groups unresolved. | Partial activation could occur accidentally. | `100` | Define cohesion groups. |
| `GAP-017` | OCSF compiled artifact details unresolved. | Silver validation cannot be reproduced. | `050` | Pin compiled artifact, compiler, validator, checksum, profiles, extensions, allowlist. |
| `GAP-018` | OCSF observation mapping rows incomplete. | Mapping implementers could choose different rows. | `050,120` | Define exhaustive MVP observation-to-OCSF mapping and validation matrix. |
| `GAP-019` | Risk and exposure scoring defaults unresolved. | Numeric risk outputs could diverge. | `130,110` | Keep scoring non-authoritative or add explicit scoring contract. |
| `GAP-020` | Graph backend not selected. | Backend-specific apply/query behavior cannot be activated. | `090` | Keep backend-neutral until preflight and validation pass. |
| `GAP-021` | Full historical graph serving not MVP. | API expectations could exceed graph retention. | `090,110` | Specify current/recent graph behavior and lakehouse historical fallback. |
| `GAP-022` | Future reachability is detailed but inactive. | Implementers may expose theoretical reachability accidentally. | `200,090` | Keep deferred and require explicit activation. |
| `GAP-023` | Error-code ownership can drift. | Shared registry may hide domain failures. | `110,domain specs` | Centralize error shape and assign codes to owners. |
| `GAP-024` | Acceptance criteria are aggregate and overlapping. | Broad acceptance may not prove domain behavior. | `120` | Split to per-spec DoD and aggregate matrix. |
| `GAP-025` | Diagrams are representational. | Implementers may implement diagrams instead of normative tables. | `030,100` | Keep diagrams non-authoritative unless generated from lifecycle definitions. |
| `GAP-026` | Public/private source inventory boundary needs enforcement. | Public specs could leak sensitive source details. | `010,020,110` | Add validation rule for private binding leakage. |
| `GAP-027` | Canonical serialization and checksum rules may not be centralized enough. | Specs may hash similar artifacts differently. | `040,030` | Centralize scalar, canonical JSON, ID, checksum rules. |
| `GAP-028` | Source extension fields and OCSF unmapped handling can be confused. | Unknown fields may bypass declaration and redaction. | `050,110` | Define exact extension field policy and reject undeclared paths. |

Rows in this ledger block authoritative status for the owning spec until resolved or explicitly deferred.
