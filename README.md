# ICl-Reasoning-Engine

> A research-oriented In-Context Learning cheatsheet that determines **equational implication over magmas** using formal logic rules, counterexample-driven reasoning, and multi-phase verification (finite models + structural proofs).

Built for the **Distillation Challenge** — testing whether LLMs can internalise a formal reasoning procedure from context alone and apply it correctly to unseen equation pairs, with no fine-tuning.

---

## What Is This?

This is a structured **system prompt / reasoning cheatsheet** tested across multiple LLMs (GPT-4, Gemini, etc.) in playground settings. Given two equations over a **magma** (a set with a single binary operation `◇`, no assumed laws), it instructs the model to answer:

> Does Equation 1 imply Equation 2 over **all** magmas?

The cheatsheet encodes a complete formal procedure — the model receives it as context and must follow it step-by-step with no external tools or precomputed theory.

---

## The ICL Connection

**ICL (In-Context Learning)** is the core mechanism here. The model receives the full reasoning procedure as context at inference time and must derive implications on-the-fly — no memorised equation databases, no fine-tuning on algebra.

This directly targets the distillation challenge goal: can a model distil formal algebraic reasoning from a compact ruleset and generalise to arbitrary equation pairs? The cheatsheet is that distilled ruleset.

A key design principle baked into the prompt: **the model is equally likely to output TRUE or FALSE**. It is not biased toward finding proofs. A failed disproof is never treated as a proof.

---

## How It Works

The procedure runs two phases in strict order. **Phase 1 always completes before Phase 2 is entered.**

### Phase 1 — Exhaustive Disproof

| Stage | What it does |
|-------|-------------|
| **1A — Balance Check** | If premise `A` is balanced but conclusion `B` is not → **FALSE** immediately (Theorem 3.8) |
| **1A-EXT — Singleton Check** | If `B` is singleton-forcing and `A` is too → **TRUE**. Otherwise continue. |
| **1B — Canonical Models** | Tests left projection (`a◇b=a`), right projection (`a◇b=b`), constant collapse. `A` holds + `B` fails → **FALSE** |
| **1C — Size-2 Enumeration** | All 16 binary operation tables over `{0,1}`. Any table where `A` holds and `B` fails → **FALSE** |
| **1D — Size-3 Modular Magmas** | Tests Z3-Add, Z3-Neg, Z3-Sub. Catches ~1% of cases that slip past size-2. |
| **1E — Structural Term Check** | Detects **Austin implications** — equations TRUE for all finite models but FALSE over infinite magmas. Free magma term-tree comparison under `A`'s rewrite rule. |

### Phase 2 — Proof Attempt

Only entered after Phase 1 is fully exhausted.

- **Step A**: Trivial checks — singleton forcing (PC1), tautology (PC2), variable rename (PC3)
- **Step B**: Structure analysis — phantom variables, nesting depth, shape type
- **Step C**: Classifies `A` into a rule pattern (F1 Constant Collapse, F2 Left Projection, F3 Right Projection, F4 Absorption, V1 Left Identity, V2 Right Identity, V3 Twisted Commutativity)
- **Step D**: Tier comparison between `A` and `B` to guide proof direction
- **Step E**: Full substitution proof — rewrites `B`'s LHS → RHS using `A` as rewrite rule, with strict no-assumption rules (no commutativity, no associativity, exact matches only)

> **V1/V2 critical note**: these identity rules do NOT imply global projection. Only expressions exactly matching `E` are cancellable.

---

## Verdict Rules

```
TRUE  → Phase 1 fully exhausted (all 16 size-2 + all 3 size-3 + Stage 1E pass)
         AND complete valid Step E proof chain.

FALSE → Explicit counterexample from Phase 1, OR Stage 1A balance check fires,
         OR Stage 1E finds structurally non-identical term trees.
```

---

## Output Format

```
VERDICT:        TRUE or FALSE
REASONING:      Which steps were decisive
PROOF:          Full substitution chain (if TRUE), else empty
COUNTEREXAMPLE: Explicit magma + falsifying values (if FALSE),
                or infinite CE described as a term-tree distinction
```

---

## Background

A **magma** is the most primitive algebraic structure: a set `M` with a binary operation `◇ : M × M → M`, subject to no further axioms. Equational implication over magmas is in general **undecidable**, making the disproof-first strategy and Austin implication detection essential — not optional.

---

## Distillation Challenge

This cheatsheet was submitted to the **Distillation Challenge**, which tests:

1. **In-context generalisation** — can an LLM follow a novel formal procedure from context alone?
2. **Correct implication detection** — without precomputed results or memorised algebraic theory?
3. **Verifiable reasoning** — producing explicit, human-checkable proof traces or counterexamples?

---

## License

MIT
