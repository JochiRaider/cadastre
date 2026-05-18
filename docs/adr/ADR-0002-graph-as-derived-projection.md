# ADR-0002: Graph as Derived Projection

## Status

`accepted_rationale`

## Decision

Graph state is a replaceable read model derived from authoritative lakehouse records and persisted graph deltas.

## Rationale

This prevents graph backend state, graph cleanup, and graph drift checks from becoming de facto truth.

## Consequences

- Implementers must use the owning NLSpec for runtime behavior.
- This ADR must not be used to resolve field shapes, algorithms, defaults, errors, or acceptance criteria.
- Contradiction between this ADR and an authoritative NLSpec must be resolved by updating this ADR or superseding it.

## Authority Boundary

Runtime behavior remains owned by `090`; this ADR is rationale only and must not define field shapes, algorithms, defaults, errors, or acceptance criteria.
