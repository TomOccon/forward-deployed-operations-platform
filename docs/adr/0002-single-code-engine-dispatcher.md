# ADR 0002: Single Code Engine dispatcher

Status: accepted

All server-side functions route through one dispatcher keyed by `function_name`. This avoids declaring many functions in the app manifest, which produced blank apps at publish time when package permissions were not granted before first load.

Tradeoff: one package is one deployment unit, so adding a function means redeploying the package.
