# ADR 0001: AppDB for mutable state, datasets for analytical reads

Status: accepted

Mutable, user-generated content (notes, engagement changes, audit events, configuration) lives in AppDB collections. Analytical data (hours, pipeline, health) lives in datasets. This keeps the write path fast and the read path scalable.

Tradeoff: AppDB has no row-level security. Client-portal isolation is enforced in the application layer, through the persona system. Truly sensitive multi-tenant use would need a proxy layer.
