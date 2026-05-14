# Paper improvement plan

## Typos

- [ ] Line 403: "sequene" → "sequence"
- [ ] Line 444: "becuase" → "because"
- [ ] Line 687: "sturcture" → "structure"
- [ ] Line 957: "how how different" → "how different"
- [ ] Line 1076: `$\iota_1, \ldots, \sigma_n$` → `$\iota_1, \ldots, \iota_n$` (σ/ι mix-up in the second construction)
- [ ] Line 1075: "temporarilly" → "temporarily"
- [ ] Line 1105: "dicusssed" → "discussed"
- [ ] Line 1144: "soundess" → "soundness"
- [ ] Line 1179: "does not, in general, does not" — duplicate phrase, delete second "does not"

## Formal corrections

- [ ] **Uniqueness definition only compares same-length paths** (lines 1160–1170). Both
  paths are universally quantified over the same index `n`, so two paths of different
  lengths leading to the same expression fall outside the definition entirely. Fix: use
  separate indices `n` and `m` for the two paths; the conclusion should be `n = m` and
  `∀i ∈ 1..n. ιᵢ = ι'ᵢ ∧ σᵢ = σ'ᵢ`.

- [ ] **Correctness and completeness definitions leave σ₀ free** (lines 989, 1101).
  Both definitions reference `choices(σᵢ₋₁)` for `i = 1`, i.e. `choices(σ₀)`, but
  `σ₀` is never bound. The definitions should open with `∀σ₀, σ₁, ..., σₙ`.

- [ ] **`select` for AI assistants is defined as `argmax` but described as probabilistic**
  (line 684 vs. lines 714–715). `argmax` is deterministic; the text then says
  "rerunning `select` with the same constraints may result in a different cleaning
  script if the machine learning algorithm is probabilistic." These are inconsistent.
  Either acknowledge that the `argmax` model is an idealisation of a deterministic
  version, or reformulate `select` to allow non-determinism and note what that does
  to the properties.

## Content TODOs

- [ ] **Sketch the `decode` operation for type providers** (line 624). The paper says
  "it is easy to define the decode operation… because the choices are items of the
  member chain" but never actually defines it. A one-line definition — something like
  `decode(e.ι₁.…..ιₙ) = (ι₁,…,ιₙ)` — would make the reconstructibility claim precise
  rather than asserted.

- [ ] **Formalise the Coq tactic-sequence case** (Section 4.4). The section discusses
  both Coq (state = sequence of tactics) and Idris (state = term) but only provides
  formal definitions for the Idris case. Adding even a brief formal treatment for Coq
  would be consistent with every other section and would make the reconstructible vs.
  non-reconstructible distinction concrete for theorem provers.

- [ ] **Clarify the two Gamma models at the start of Section 4** (lines 483–486). The
  section opens by announcing a model "inspired by The Gamma, but different in one
  notable way" without saying what the difference is until Section 4.2. State the
  difference upfront: Section 4.1 generates expressions that directly call operations;
  Section 4.2 models the actual Gamma, which encodes choices as member accesses and
  never exposes the underlying operations in the source code.

- [ ] **Direct manipulation section** (line 1500): the current text says "It remains
  to be seen if the choose-your-own-adventure calculus can provide a new way of
  looking at [ARK / Morphic]", which is too optimistic. Per Brian's email, these
  systems don't fit because they lack `select(σ)` — interactive state is never
  reified into code. Revise to acknowledge this as a genuine boundary of the
  calculus: ARK/Morphic are the "opposite" of the theorem-prover case (where state
  *is* the expression) — here there are only states and no separate expressions,
  which makes the key properties vacuous. A state-diff model is conceivable but
  uninteresting for the same reason.

## From ECOOP reviews

All three reviewers praised the writing but rated soundness and significance as subpar.
The shared diagnosis: the paper offers definitions but no results. The actionable items below
address the specific gaps they named.

**Substance**

- [ ] **Add at least one non-trivial theorem.** Every reviewer noted the lack of
  meta-theoretic results. A natural candidate: show that the two constructions in
  Section 4 (lines 1075–1087) that convert an eventually-correct system into a
  correct one actually work — i.e., prove that the output satisfies the correctness
  definition. Another candidate: prove that completeness is preserved (or not) by
  those constructions.

- [ ] **Motivate each property in terms of user experience**, not just PL analogy.
  Review B asks explicitly: why should a reader care about uniqueness? For each
  property (correctness, completeness, uniqueness, eventual correctness) add a
  sentence or two on what *goes wrong* in practice if a system lacks it. The LLM
  auto-complete example (Review A) — where incorrect options are offered — is a
  natural hook for motivating correctness.

- [ ] **Add a "what does formalism buy you" argument.** Reviews B and C both ask
  why informal reasoning isn't sufficient. Add a short passage (intro or
  Section 3) that gives a concrete example of a subtle distinction the paper
  makes precise that would be easy to miss or get wrong informally — the
  reconstructibility / uniqueness gap (lines 1188–1195) is a good candidate,
  since it is genuinely non-obvious.

**Scope**

- [ ] **Acknowledge the LLM distinction explicitly** (Review B). The reviewer
  correctly points out that LLM-based completion differs fundamentally from
  type providers: choices are probabilistic, the state spans conversation history,
  and options may be incorrect. The paper currently puts them in the same model
  without comment. Add a paragraph noting how the model accommodates (or does not
  accommodate) these differences — probabilistic `choices` is fine formally; the
  correctness property is what captures the unsoundness.

- [ ] **Add one more example domain** to counter the "anecdotal" criticism (Reviews
  A and C). Brian suggested parametric/feature-based CAD (e.g. OnShape) as a
  domain where you construct a model by choosing operations from a history tree.
  Even a brief paragraph showing it fits the calculus would broaden the claim.

- [ ] **Discuss Programming by Navigation** [lubin-2025-pbn] as a closely related
  formal framework that independently confirms the CYOA insight from the program
  synthesis direction. The correspondence is precise:

  | PbN | CYOA |
  |-----|------|
  | Working sketch `e` (expression with holes) | State σ — same as the theorem-prover case where `select(e) = e` |
  | Step set `S(e)` returned by step provider | `choices(σ)` |
  | Step σ (e.g. `?h ↝ f(…)`) | Identifier ι |
  | **Strong Soundness** | **Correctness** |
  | **Strong Completeness** | **Completeness** |

  PbN's step provider maps exactly to the CYOA `choices` operation; its step
  decider (human or automated) is the user making a choice. The state is the
  expression itself, making this another instance of the theorem-prover case
  (Section 4.4) where `select(σ) = σ`.

  Two things worth highlighting in the discussion:

  1. PbN proves three meta-theorems about its calculus (Fail Fast, Progress,
     Constructability — Section 3.3 of the paper) that are the exact kind of
     non-trivial results the ECOOP reviewers said CYOA lacked. These theorems
     hold for any system satisfying the navigation relation properties
     (Determinism, No Loops, Reachability, Finite Between). The CYOA paper
     could note that analogous results hold for CYOA systems satisfying similar
     conditions, citing PbN as a proof-of-concept.

  2. CYOA captures something PbN does not: the **reconstructibility** distinction.
     PbN does not discuss whether the sequence of steps can be decoded from the
     final expression — this is a genuine contribution of CYOA not present in PbN.

  The best placement is probably Section 4.4 (extending the type-directed
  synthesis example) and/or as a related work discussion.

## LLM experiments (Section 5.2)

The experiments have been rerun with new models. The paper's experiment section needs
to be rewritten to reflect this. Key differences between what the paper currently says
and the new results in `cyoa-experiments/`:

- **Models**: paper says "GPT, Gemini, DeepSeek" → now Claude Haiku 4.5 and Opus 4.7.
- **Dataset**: paper says "Eurostat database, 10 queries" → now The Gamma gallery,
  61 snippets across 5 providers (olympics, worldbank, shared, expenditure, drWho),
  665 steps total.
- **Conditions**: paper describes "five different prompting strategies" including
  lookahead (inline and nested). The new experiments test two conditions: no system
  prompt (baseline) and a system prompt containing per-provider navigation rules.
  Lookahead is no longer one of the conditions.
- **New figure**: replace the current Figure 8 with `results/accuracy.png` (grouped
  bar chart, Haiku vs Opus × no-prompt vs with-prompt, broken down by provider).
- **Key finding has changed**: the paper currently says lookahead helps GPT but not
  other LLMs. The new finding is that the navigation-rules prompt adds ~11 pp for
  Haiku (54% → 65%) and ~19 pp for Opus (60% → 79%) — both models benefit, but
  Opus benefits substantially more. Discuss why: Opus may be better at following
  structural navigation rules, or it is simply more capable of exploiting the
  additional context.
- **The example prompt in Figure 7** shows a bare per-step query; the new system
  prompt (`prompts/default-prompt.txt`) is a provider-level navigation guide passed
  as a system message, not part of the per-step query. Clarify this architectural
  distinction in the text.
- **The "LLM assistant is broadly applicable" argument** (lines 1396–1426) does not
  depend on the specific models tested and can stay, but the claim about lookahead
  improving quality should be removed or replaced with a note about domain-knowledge
  prompts being the more effective lever in the new experiments.
