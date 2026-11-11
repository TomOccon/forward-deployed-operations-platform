# ADR 0004: Reconcile source volume against activity before building on it

Status: accepted

The raw engagement source listed 580 "In Progress" engagements. Cross-referencing against time entries in the last 90 days reduced that to 123 truly active. Every downstream capacity and utilization number depends on this, so the active set is derived from observed activity, not from the status field.

Related failure modes (resource identity mismatch across systems, join keys beyond 10,000 distinct values) are documented in [customer-data-engineering](https://github.com/TomOccon/customer-data-engineering), which ships runnable checks for them.
