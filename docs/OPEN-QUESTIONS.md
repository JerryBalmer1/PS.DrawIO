# Open questions

Append-only register of things deliberately undecided. These lived only in conversation until this file; losing them would force reconstructing the reasoning later.

**Rules**

- One entry per open question (`Q-NNN`).
- Do not delete a resolved question. Point at the ADR (or other decision record) that closed it and mark the entry resolved.
- Evidence links go to real paths in sibling repositories.

---

## Q-001 — Does contract v2 happen, and what is in it?

**Question.** Will Registry contract major 2 ship, and what must it contain?

**Why open.** Contract v1 is frozen and shipping. Two confirmed gaps already need a major to fix cleanly. Shipping a major scoped only to the first gap risks a second major immediately after.

**Evidence so far**

- Node vs edge declaration kinds: Registry [ADR 0002](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0002-node-and-edge-declaration-kinds.md). Confirmed by Provider.PowerShell (deliberate failing acceptance test; [Provider ADR 0003](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0003-node-edge-declaration-contract.md)) and by the Terraform spike (same single-`Shapes` workaround with `Edge = $true`).
- Cross-provider identity / far-end ownership: Registry [ADR 0003](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0003-cross-provider-references.md). ADR 0003 warns that a major scoped to the node/edge split alone would likely need another major soon after.

**What would resolve it.** An explicit contract-v2 ADR in Registry listing the field set, migration, and whether node/edge split and cross-provider identity ship together or not — or a written decision that v1 stands and workarounds remain.

**Related.** Registry ADR 0002, Registry ADR 0003, Provider.PowerShell ADR 0003.

---

## Q-002 — Who owns composition — joining multiple providers' graphs?

**Question.** Which component joins graphs from more than one registered provider?

**Why open.** No current specification names an owner. Implementing a join in any repo would decide the architecture by accident.

**Evidence so far**

- Registry [ADR 0003](https://github.com/JerryBalmer1/PS.DrawIO.Registry/blob/main/docs/DECISIONS/0003-cross-provider-references.md) Decision 5: nothing composes multi-provider graphs today; either Core's scope expands or a component is missing; that choice is not made there.
- Core [ADR 0001](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/docs/DECISIONS/0001-provider-qualified-node-ids.md) hedges with provider-qualified node Ids (`Provider:Type:Name`) without claiming composition.
- [CORE.md §3](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/CORE.md) puts multi-provider composition out of scope for Core v1. Signals go to `docs/COMPOSITION-SIGNALS.md`, not into code.

**What would resolve it.** An ADR that assigns ownership (Core expansion vs new component vs never) after Core's IR exists and there is join evidence rather than speculation.

**Related.** Registry ADR 0003, Core ADR 0001, CORE.md §3.

---

## Q-003 — Which layout strategy does Core v1 implement?

**Question.** Of the three layout strategies behind one interface, which does Core v1 ship?

**Why open.** Core is unbuilt. v1 must ship at least one real strategy; choosing poorly couples the first consumers to the wrong dependency shape.

**Evidence so far**

- [CORE.md §5](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/CORE.md) defines three strategies behind one interface: compute coordinates in PowerShell, shell out to the Desktop CLI `--layout`, or emit a `#create` URL with `applyLayouts`.
- Verified constraint (same section): a `.drawio` file renders at stored coordinates; Core owns geometry. Do not restate the full layout finding here — read CORE.md.

**What would resolve it.** Core implementation ADR plus code: the layout interface and **one** deliberately simple built-in strategy that is defined and testable, not “good.”

**Related.** CORE.md §5, §7; Core has no layout ADR yet.

---

## Q-004 — Repository visibility and licensing

**Question.** Do all repositories stay public under MIT, or do some later go private?

**Why open.** Infrastructure benefits from open collaboration; domain-specific providers and accumulated analysis playbooks may later warrant private treatment. No decision has been made.

**Evidence so far**

- All current repositories under `JerryBalmer1/PS.DrawIO*` that were inspected are public and MIT-licensed where a `LICENSE` file exists.
- Consideration only: keep infrastructure open; possibly treat some domain providers / playbooks as private later.
- **MIT already granted is not fully revocable** for copies already taken under that license.

**What would resolve it.** A written policy (even a short ADR in this map repo) stating which classes of repository stay public MIT and which may be private, before any private repo is created under that policy.

**Related.** None yet.

---

## Q-005 — Is the fixed Clean → Analyze → Test → Package task order correct?

**Question.** Should the ecosystem keep the fixed four-step default task order when repositories legitimately need a red gate?

**Why open.** Three repositories have hit variants of the same collision: a pipeline that must fail on failure, in a repository that legitimately has one. Exit code alone stops distinguishing “known red” from “new regression.”

**Evidence so far**

- Provider.PowerShell [ADR 0004](https://github.com/JerryBalmer1/PS.DrawIO.Provider.PowerShell/blob/main/docs/DECISIONS/0004-deliberate-failure-blocks-packaging.md): deliberate failing acceptance test blocks Package on default `All`.
- Core [ADR 0002](https://github.com/JerryBalmer1/PS.DrawIO.Core/blob/main/docs/DECISIONS/0002-analyze-skips-missing-source.md): missing `src/` made Analyze fail before Test could show the intended empty-DoD failure; temporary skip added. Notes this as the third variant (Registry structure; Provider fail-vs-Package; Core missing-src-vs-Analyze).
- Registry uses the same task shape but has no deliberate failing test today.

**What would resolve it.** A fourth independent collision that forces redesign of the task order — or an ecosystem-level ADR that keeps the order and standardizes how known failures are labeled without weaken-to-green. Watching, not deciding.

**Related.** Provider.PowerShell ADR 0004, Core ADR 0002.
