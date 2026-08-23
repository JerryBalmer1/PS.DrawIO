# PS.DrawIO

## What this repository is

This repository is a **map** of the PS.DrawIO ecosystem. It ships no code, no module, and no tests.

Its scope bar: material that spans more than one repository **and** has no single owning repository. Anything that fails either test belongs elsewhere.

## The ecosystem

```
  PS.DrawIO.Provider.*                PS.DrawIO.Registry              PS.DrawIO.Core
  (domain extraction)                 (contract broker)               (unbuilt)
  +------------------+                +------------------+            +------------------+
  | declare shapes,  |  register /    | validate store   |  resolve   | apply geometry   |
  | hints, links,    | -------------> | declarations     | ---------> | emit .drawio XML |
  | theme defaults   |                | Resolve-Shape    |            |                  |
  +------------------+                +------------------+            +------------------+
         |                                      ^
         |                                      |
         +---- vocabulary only -----------------+
              (no XML, no layout pixels)
```

This diagram is hand-maintained until the ecosystem can draw itself; replacing it with generated output is a real milestone.

## Current state

Gathered 2026-08-22 from local clones. Versions and tags are facts; status is blunt on purpose.

| Repository | Role | Version / tag | Status |
|---|---|---|---|
| [PS.DrawIO](https://github.com/JerryBalmer1/PS.DrawIO) | Ecosystem map | untagged (`500905c`) | Map only. No module, no `src/`, no CI runs. |
| [PS.DrawIO.Registry](https://github.com/JerryBalmer1/PS.DrawIO.Registry) | Contract broker: register, resolve, conformance | **v1.1.0** (`ModuleVersion` 1.1.0; tags `v1.0.0`, `v1.1.0`) | **Ships.** `src/` has 13 `.ps1` files. Latest CI: success. |
| [PS.DrawIO.Provider.PowerShell](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell) | Static AST analysis → `PSModuleGraph` | **v1.0.0** (`ModuleVersion` 1.0.0) | **Ships** with one deliberate failing acceptance test (node/edge declaration split). `src/` has 18 `.ps1` files. Latest CI: failure (that known test). Default `All` build does not package; use `-Task Package`. |
| [PS.DrawIO.Provider.Terraform](https://github.com/JerryBalmer1/PS.DrawIO.Provider.Terraform) | Intended Terraform provider | untagged; spike manifest `ModuleVersion` 0.0.0 | **Not a product.** Two throwaway contract spikes and a non-shipping manifest; `src/` has **0** `.ps1` files. No CI runs. |
| [PS.DrawIO.Provider._Template](https://github.com/JerryBalmer1/PS.DrawIO.Provider._Template) | Provider scaffold placeholder | untagged | **Empty.** README only; no manifest, no `src/`, no CI. |
| [PS.DrawIO.Core](https://github.com/JerryBalmer1/PS.DrawIO.Core) | Serialization, layout, `.drawio` emission | untagged | **Unbuilt.** `CORE.md` + acceptance harness; no manifest, no `src/`. Latest CI: failure (empty Definition of Done meta-test). |

Five sibling product names are not five shipping products. Registry and Provider.PowerShell are the only modules that ship today.

## How the pieces relate

Providers declare vocabulary: semantic types, variants, link templates, layout **hints**, and theme defaults. They extract domain graphs. They do not place shapes or write draw.io XML.

The registry validates declarations against contract v1, stores them, and resolves `(Provider, Type)` to opaque shape data. It never renders and never runs provider code during resolve.

Core (when built) consumes a provider graph, resolves types through the registry, owns geometry, and emits uncompressed `.drawio` XML. See [PS.DrawIO.Core/CORE.md](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/CORE.md) for layout ownership and the swappable layout interface — do not restate those constraints here.

**Module version and contract version are different things.** A provider may ship `ModuleVersion` 1.0.0 while declaring `ContractVersion = 1`. Never conflate them. Contract majors are ecosystem events; module versions are per-repository release labels.

## Decision records

All nine ADRs across the three repositories that have them. Subjects are taken from the ADR bodies, not the filenames.

| Repository | # | Title | Subject | Link |
|---|---|---|---|---|
| Registry | 0001 | Class identity at module boundaries | Keep PS classes internal; exchange `PSCustomObject` + `PSTypeName` across module boundaries. | [0001](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0001-class-identity-at-module-boundaries.md) |
| Registry | 0002 | Node and edge declaration kinds | Contract v1's single `Shapes` map cannot separate node and edge kinds; defect recorded, contract unchanged. | [0002](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0002-node-and-edge-declaration-kinds.md) |
| Registry | 0003 | Cross-provider references | No far-end owner, join, or composition API in v1; composition ownership left open. | [0003](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0003-cross-provider-references.md) |
| Provider.PowerShell | 0001 | Closed graph | Emit placeholder nodes so every edge endpoint resolves to a node `Id`. | [0001](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0001-closed-graph.md) |
| Provider.PowerShell | 0002 | External classification taxonomy | Tag external edges `BuiltIn` / `Module` / `Unknown`. | [0002](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0002-external-classification-taxonomy.md) |
| Provider.PowerShell | 0003 | Node/edge declaration contract | Stay on Registry v1 `Shapes`; keep the failing separate-declaration acceptance test as evidence. | [0003](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0003-node-edge-declaration-contract.md) |
| Provider.PowerShell | 0004 | Deliberate failure blocks packaging | Known failing test + throw-on-failure means default `All` never reaches Package; use `-Task Package`. | [0004](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0004-deliberate-failure-blocks-packaging.md) |
| Core | 0001 | Provider-qualified node Ids | IR node Ids are `Provider:Type:Name` — a cheap hedge; composition ownership still undecided. | [0001](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/docs/DECISIONS/0001-provider-qualified-node-ids.md) |
| Core | 0002 | Analyze skips missing source | Temporary: ScriptAnalyzer warns and continues when `src/` is absent so Test can surface the real signal. | [0002](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/docs/DECISIONS/0002-analyze-skips-missing-source.md) |

**Ecosystem-level despite living in Registry:** 0002 (node and edge declaration kinds) and 0003 (cross-provider references). Both affect every provider and Core, not only the registry module.

## Open questions

Deliberately undecided items live in [docs/OPEN-QUESTIONS.md](docs/OPEN-QUESTIONS.md).

## Agent execution protocol

Agent work starts from [`.agent/TRAPS.md`](.agent/TRAPS.md) (read once) and logs the run in `.agent/EXECUTION.md` (gitignored). See [`.agent/README.md`](.agent/README.md).