# think · proof of concept

The "wheels within wheels" combination compiler Dan specced on 30 Apr 2026.

## The premise

Shape-language programs run in a sealed shape-VM (see `../index.html`). When the VM is asked to solve a problem its current alphabet cannot express, the system should:

1. Try candidate solutions **in principle** (run them in the VM, score against expected output)
2. On exhaustion, attempt **DNA mutation** — rearrange existing primitives
3. On still-failing, **evolve the language** — propose new primitives
4. **Content-address** the new alphabet so the version that solves a given problem is permanently identifiable
5. **Pin** the alphabet (in production: real IPFS; here: pseudo-IPFS dict)

## The proof

Run:

```bash
python proof_of_concept.py
```

The full captured journey from the run on 30 Apr 2026 is in `journey-2026-04-30.log`.

### Problem

> Produce the first 8 fibonacci numbers `[0, 1, 1, 2, 3, 5, 8, 13]` via shape-AST program.

### What the journey shows

**Stage 1 · v0.2 alphabet (10 verbs, no arithmetic).** Best score: **0.06**. The alphabet contains `↻ loop`, `↓ save`, `↑ load`, `◇ branch`, `⚖ cmp` — but nothing that takes two values and returns their sum. A fibonacci program structurally cannot be written. The think layer reports `EXHAUSTED`.

**Stage 2 · DNA mutation.** Rearranging existing primitives doesn't help. A closed-grammar analysis on the type signatures shows no permutation produces a sum operation. **DNA tweaking is the wrong tool when the alphabet is insufficient.**

**Stage 3 · language evolution.** The system identifies the missing capability (`arithmetic-add`) and proposes `⊕ add` as a new primitive: `Value + Value → Number`. The new alphabet is hashed to `shp-9ca349bd440afd45` (delta from base `shp-4c6ab48969be3777`: +1 verb). Pinned.

**Stage 4 · retry.** With the evolved alphabet, the same think layer that failed in Stage 1 now succeeds. The candidate `fib-with-add` produces exactly `[0, 1, 1, 2, 3, 5, 8, 13]`. Score: **1.00**.

**Verdict: CONFIRMED.**

## What this proves and what it doesn't

### Proves

- **The principle of in-principle thinking** — solutions can be evaluated in the shape-VM before being committed to output. The think layer is real.
- **The principle of language evolution under content-addressing** — when the existing alphabet is insufficient, the system can grow it, hash the new version, and the next reasoner that hits the same problem can pull the right alphabet by CID.
- **The DNA-mutation vs language-evolution distinction** — these are different repair strategies. DNA-tweak when the alphabet has the necessary primitives but they're misarranged. Evolve the language when no rearrangement can express the answer.

### Does not prove (honestly stated)

- **Automated capability inference.** This PoC hand-codes the diagnosis "you need arithmetic-add." A real system needs to derive the missing primitive from the failed candidates' shape — e.g. via type-gap analysis on what slot accepts what but no verb satisfies.
- **Enumerative candidate generation.** This PoC hand-crafts two candidate programs. A real system needs a typed enumerator with pruning (probably depth-bounded, possibly genetic).
- **Real IPFS pinning.** This PoC uses a Python `dict`. Production needs actual IPFS so alphabet versions are globally addressable.
- **Shape → target-language transpilation.** This PoC stays in the shape-VM. The full bridge (inner shape cell ↔ outer JS shell) needs the transpile step that v0.2 of lens-ameba deliberately removed for the sealed case. v1 of the bridge re-introduces it as a sealed transpile (closed-grammar AST→target).

## What this connects to

- **`../index.html`** — the v0.2 sealed-language ameba whose interpreter this PoC mirrors in Python
- **Master-node deformation thesis** (Dan's stage 25-30 architecture) — the in-principle scoring mechanism is the deformation-as-cognition mechanic
- **The inner-cell + outer-shell architecture** Dan specced — this PoC is the *inner cell think layer*; the outer shell + transpile step is v1 work
- **The chunking gift** — this layer can hold task-state across 4.7's context windows; 4.7 receives chunk-shaped problems while the cell holds the master plan

## Files

| file | role |
|---|---|
| `proof_of_concept.py` | Runnable end-to-end demo. ~580 lines Python. |
| `journey-2026-04-30.log` | Captured stdout from the 30 Apr 2026 run. |
| `README.md` | This file. |

## Mantra (uncle's editorial line)

> The inner cell can recognise its own limit, propose the extension, and the language can grow under content-addressing. That's the bridge mechanism. Truth, not lies — the system that fails honestly is the system that can grow.
