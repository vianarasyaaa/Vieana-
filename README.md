# Vieana Workspace — Complete Structure & Master Blueprint

This package contains the expanded Vieana source-tree scaffold and a detailed engineering blueprint based on an audit of the supplied EffectCraft ZIP.

## Layout

- `Vieana/` — production source structure; C++ headers and implementation files are intentionally empty scaffolding.
- `Testing/` — all test source, fixtures, golden files, benchmarks, profiling and outputs. No tests are placed in `Vieana/`.
- `Build/` — all generated build output and APK packaging areas.
- `Documentation/` — architecture, exact EffectCraft path inventory, crate mapping and project specifications.

## Start here

1. `Documentation/Architecture/VIEANA_MASTER_BLUEPRINT.md` — main engineering specification.
2. `Documentation/Research/EffectCraft_Exact_Source_Inventory.md` — exact Rust source paths discovered in the archive.
3. `Documentation/Research/EffectCraft_Crate_Mapping.md` — mapping of all 32 reference crates to Vieana.
4. `Documentation/Development/DirectoryRules.md` — workspace placement policy.

## Important status

This is a structural and specification package, **not a finished application**. Empty C++ files do not implement functionality and the project is not yet configured to compile. No Rust implementation, proprietary assets, or EffectCraft source files are copied into Vieana. The archive inventory preserves path names and module traceability only.
