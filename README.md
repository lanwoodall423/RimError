# RimError

RimError turns RimWorld/mod logs into deterministic, bounded diagnostics. It has no RimWorld or LLM dependency.
When RimTest is present, agents should start with RimTest and use RimError directly only for the
diagnostic drill-down it requests.

## Normal workflow

```text
make change
→ rimtest affected --run --json
→ if a diagnostic id is returned, rimerror show <diagnostic-id>
→ act on the returned root cause and edit again
```

Typical clean comparison output:

```json
{"status":"clean","newErrors":0,"newWarnings":0}
```

Typical failure output contains only actionable root causes:

```json
{"status":"fail","errors":1,"warnings":0,"rootCauses":[{"id":"d-81f72","type":"NullReferenceException","method":"Mod.Type.Tick","count":8127,"confidence":"high"}]}
```

Useful commands:

```text
rimerror ingest Player.log
rimerror ingest --stdin
rimerror latest --json
rimerror latest --json --run <run-id>
rimerror latest --json --all
rimerror show <id>
rimerror baseline create [name]
rimerror compare --json [--baseline <name>]
rimerror export --json
```

The default store is `.rimerror/latest.json`; use `--store <path>` or `RIMERROR_STATE_PATH` to override it. `show` and `latest --all` expose bounded evidence; default JSON omits stacks, raw logs, nulls, and warning-only detail. Baselines retain schema, fingerprint, RimWorld, and mod-profile metadata and fail safely when incompatible.

RimTest normally supplies a bounded, DevBridge-owned, generation-scoped semantic source and
calls `latest --run <run-id>` to prevent nearby-run cross-contamination. Agents should not read
Player.log or configure RimError paths in the normal RimTest loop; explicit paths remain a
fallback for unusual environments.

Exit codes: `0` success/clean, `1` detected actionable diagnostics, `2` RimError or usage failure.

DevBridge2 and RimBridgeServer remain optional metadata sources. The versioned interchange contract is [docs/integration-contract.md](docs/integration-contract.md).

Verification:

```text
dotnet format RimError.sln --verify-no-changes
dotnet build RimError.sln --configuration Release
dotnet test RimError.sln --configuration Release
```

CI guarantee: the Windows offline workflow builds the Release solution and runs the complete
deterministic xUnit suite. RimError remains the parser, root-cause, and correlation authority; it
does not manage RimWorld. The pinned no-RimWorld cross-stack contract gate is owned by RimTest and
checks the `rimerror-integration/v1` handoff and correlated diagnostic projection.
