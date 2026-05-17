# ADR-0003: OCSF as Silver Profile

## Status

`accepted-draft`

## Decision

OCSF governs `CadastreSilverObservation.normalized_fields` only when the active external schema profile and mapping rows permit it.

## Rationale

This gives silver observations an external semantic profile while keeping omission, lineage, identity, temporal, confidence, gold, and graph semantics Cadastre-owned.

## Consequences

- Implementers must use the owning NLSpec for runtime behavior.
- This ADR must not be used to resolve field shapes, algorithms, defaults, errors, or acceptance criteria.
- Contradiction between this ADR and an authoritative NLSpec must be resolved by updating this ADR or superseding it.

## Authority Boundary

Runtime behavior is owned by `050`; this ADR is rationale only.
