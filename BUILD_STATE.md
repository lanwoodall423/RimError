# Build state

## Current completed stage

Stage 9 PASS — production-readiness audit completed.

## Important architecture decisions

- Core is dependency-free C#/.NET: streaming framing, bounded normalization/fingerprinting, deduplication, classification, persistence, baselines, source indexing, causal grouping, and compact JSON reporting.
- Latest default JSON returns actionable root causes; warning-only detail and downstream evidence require `latest --all` or `show <id>`. Compare default JSON returns only new actionable diagnostics.
- Store schema `v:1`, fingerprint schema `1`, causal schema `1`, and integration contract `rimerror-integration/v1` remain stable. JSON omits nulls and stacks by default.
- DevBridge2 owns lifecycle/test leases; RimBridgeServer owns live operations/logs; RimError consumes optional bounded JSON projections and has no external runtime dependency or LLM.
- Compiled deterministic regexes are the only Stage 9 hot-path optimization; one million repeated lines remains one retained record.

## Verification commands

- `dotnet format RimError.sln --verify-no-changes` — PASS
- `dotnet build RimError.sln --configuration Release` — 0 warnings, 0 errors
- `dotnet test RimError.sln --configuration Release --no-build` — 124 passed
- Stage 9 matrix and one-million-line stress fixtures — PASS
- CLI ingest/latest/compare/show and baseline paths manually exercised — PASS

## Known constraints

- Text framing is conservative for undocumented formats; exact source lines require index evidence.
- Live log rotation and multiple concurrent writers are not a shared-state protocol; optional bridge metadata can be stale or ambiguous and is not used by fingerprinting.
- Baseline filtering is separate from `latest`; use `compare --json` for known-noise suppression. Corrupt stores fail as RimError operational errors.

## Next-stage notes

- Final architecture is complete for this scope; preserve bounded evidence, deterministic ordering, schema versions, and external ownership boundaries.
- Reproducible audit metrics and limitations are in `docs/stage9-audit.md`; do not add broad features in follow-up work.
