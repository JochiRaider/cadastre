# ADR-0001: Lakehouse-Fed Boundary

## Status

`accepted-draft`

## Decision

Cadastre consumes declared lakehouse feeds and table/object refs. External suppliers collect from enterprise source systems.

## Rationale

This separates source transport from Cadastre interpretation and preserves replay, evidence, and supplier-boundary clarity.

## Consequences

- Implementers must use the owning NLSpec for runtime behavior.
- This ADR must not be used to resolve field shapes, algorithms, defaults, errors, or acceptance criteria.
- Contradiction between this ADR and an authoritative NLSpec must be resolved by updating this ADR or superseding it.

## Authority Boundary

Runtime behavior is owned by `010` and `020`; this ADR is rationale only.
