# ADR-0004: Package-Set Activation

## Status

`accepted_rationale`

## Decision

Production activation targets immutable package sets, not individual artifacts or scalar signature summaries.

## Rationale

Cadastre packages are interdependent across adapters, parsers, mappings, resolver profiles, policies, validation, graph projection, and runtime compatibility.

## Consequences

- Implementers must use the owning NLSpec for runtime behavior.
- This ADR must not be used to resolve field shapes, algorithms, defaults, errors, or acceptance criteria.
- Contradiction between this ADR and an authoritative NLSpec must be resolved by updating this ADR or superseding it.

## Authority Boundary

Runtime behavior remains owned by `100`; this ADR is rationale only and must not define field shapes, algorithms, defaults, errors, or acceptance criteria.
