# Glossary

This glossary is a navigation aid. Runtime definitions live in the owning NLSpecs.

| Term | Meaning |
| --- | --- |
| Authoritative lakehouse record | A raw, silver, identity, gold, graph-delta, graph-apply, registry, or health table record governed by Cadastre table-state contracts. |
| Derived projection | A rebuildable output from authoritative records, such as graph serving, CIM output, OCSF export, or consumer projection. |
| Lakehouse-fed boundary | The rule that Cadastre production reads declared lakehouse feeds and must not collect directly from enterprise source systems. |
| Source absence | A negative or absence outcome authorized only by source authority, completeness, coverage, and staleness contracts. |
| Completeness receipt | Evidence that Cadastre read a declared lakehouse target; not source absence authority by itself. |
| Coverage assertion | A source-backed assertion that required coverage dimensions are satisfied for a coverage-sensitive domain. |
| Resolver profile | The sole production authority for identity resolution behavior. |
| Gold fact | A canonical bitemporal assertion with valid time, known time, evidence, authority, assertion state, and confidence. |
| Graph delta | A deterministic graph read-model mutation derived from authoritative records. |
| Package set | The immutable coherent set of package releases that can affect validation or production output. |
| Validation matrix | Rows that prove success, rejection, no-op, edge, and negative authority-boundary behavior. |
| Deferred reachability | Future theoretical reachability material preserved inactive, with no MVP graph or gold effect. |
