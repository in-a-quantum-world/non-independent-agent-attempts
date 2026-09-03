# Paper Structure — Attempts Are Not Independent

Modeled on the structural skeleton of "No Attacker Needed"
(arXiv:2604.01350), adapted to the evaluation-validity setting, with the
statistical contribution (bias derivation + independence test)
integrated as RQ3. This document is the paper's blueprint; the sealed
[research protocol](research-protocol.md) remains the operational spec —
where they overlap, the protocol governs.

---

## Title & framing

**Attempts Are Not Independent: State Carryover and Provenance
Disclosure in Agentic Capability Evaluation**

Named failure mode, defined once in the introduction and used
consistently (their "UCC" move — one plain-language term, no further
coinage): **cross-attempt contamination** — a nominally independent
evaluation attempt whose outcome is influenced by environment state
produced by a prior attempt.

The one-paragraph "Our position" statement (mirroring theirs §2, written
to be quotable):

> Prior work on agent-state integrity is predominantly adversarial: an
> attacker poisons memory, injects prompts, or crafts cross-session
> payloads. Recent work on unintentional cross-user contamination
> removes the attacker but studies deployment harm — one user's residue
> degrading another user's outcome. We study a third regime: the
> residue's author and its beneficiary are the same evaluee, no party is
> harmed at interaction time, and the casualty is the measurement
> itself. The harness functions as designed; the evaluation statistics
> do not. We additionally measure whether the agent discloses that its
> success used prior state — the property that determines whether this
> failure is detectable without instrumentation.

---

## 1. Introduction

1. Evaluations assume attempts are i.i.d. samples of the model;
   safety-relevant statistics (pass@k, max-of-N, threshold-acceptance
   bars) are order statistics, maximally sensitive to dependence.
2. In a stateful harness, state lives in the environment, not the model
   — session boundaries isolate the model but not the filesystem, so a
   fresh model instance (or a *different* model) inherits prior
   attempts' residue.
3. Motivating case study, one paragraph + forward-reference to §5: two
   contamination incidents in the authors' own evaluation work, detected
   by hash verification, including a byte-identical replay presented as
   fresh work and disclosed only under direct questioning.
4. Contributions list (their format, five bullets):
   - We identify cross-attempt contamination as a validity problem in
     agentic capability evaluation, distinct from adversarial poisoning,
     cross-user contamination, sandbagging, and train-time benchmark
     contamination.
   - We formalize it in a stateful evaluation model, derive the bias it
     induces in pass@k and max-of-N under a two-gate replay model, and
     give a canary-free permutation test for attempt independence.
   - We enumerate the persistence channels of a production coding-agent
     harness and measure per-channel contamination rates with planted
     canaries under controlled configurations.
   - We measure provenance disclosure: whether agents whose success
     demonstrably used prior state report it, unprompted and under
     escalating questioning.
   - We evaluate an environment-provisioning defense (the hardened
     protocol) and characterize its residual, releasing the audit kit.

## 2. Related work

Four boundary paragraphs (each: nearest work + the exact difference, one
sentence of deference, one of distinction):

(a) unintentional cross-user contamination — closest neighbor;
deployment harm to a different user vs. measurement harm to the
evaluation of the same evaluee, plus disclosure as a measured variable;

(b) adversarial memory/state security (poisoning, stored injection,
cross-session attacks) — attacker-driven vs. attacker-free;

(c) train-time benchmark contamination incl. memorization-vs-reasoning
work — contamination into weights before evaluation vs. into environment
during it; their detectors cannot see ours;

(d) harness-disclosure position paper + sandbagging/elicitation — the
former argues without measuring (we are the called-for measurement,
specialized to the state store); the latter is the mirror image
(capability hidden from the eval vs. manufactured by it).

## 3. Problem formulation (their §3, adapted)

**3.1 Stateful evaluation model.** An evaluation runs attempts
n = 1…K of task x. The environment before attempt n is E_n (filesystem,
git objects, memory files, histories, grants). Attempt n:

  (y_n, s_n, E_{n+1}) = f(x, R(E_n)),  E_{n+1} = W(E_n, trace_n)

where R is the read view exposed to the session, W the write-back of
session side effects, y_n the output, s_n ∈ [0,1] the score. The
evaluation protocol *assumes* R(E_n) = R(E_0) for all n — the
fresh-environment assumption — and computes statistics as if s_1…s_K
were i.i.d.

**3.2 Independence and its violation.** Attempts are independent iff
R(E_n) contains no attempt-generated content: W ∘ R composes to the
identity on evaluation-relevant state. A **cross-attempt contamination
event** occurs when a state fragment e written during attempt m < n
influences (y_n, s_n). Operationalized by the paired comparison (their
§3.4, which is also our detector logic): run the attempt under E with
and without the fragment; attribute the difference to e. Our canaries
make the attribution mechanical rather than judged: access is proven by
the tool-call log, influence by exact match on per-trial randomized
values.

**3.3 Two-axis taxonomy.** Their single content axis, upgraded to two:

- **Channel axis (mechanism):** the seven persistence channels + weights
  control, as in the protocol §2 table — *where* the state lives and how
  it's read.
- **Content axis (what transfers), three types with a worked example
  each:**
  - **Solution transfer** — a complete artifact replayable as-is (the
    playbook incident); detection: byte/hash identity.
  - **Value transfer** — a parameter, threshold, or fix-site that
    shortcuts execution (the canary schema); detection: exact match +
    performance anomaly.
  - **Strategy transfer** — approach-level knowledge (which method
    works, which dead ends to skip) with no verbatim trace; detection:
    performance anomaly only, weakest attribution — stated as such.

The primary matrix measures value transfer (cleanest attribution); a
secondary arm on the top-2 channels plants solution- and strategy-type
canaries to measure whether contamination rate varies by content type
(their RQ2 analog). Prediction registered in the protocol: strategy
transfer is the hardest to detect and the most resistant to hygiene —
the analog of their finding that procedural contamination survives
sanitization because it is not localized.

**3.4 Why order statistics are the victim (the technical core — upgrade
1 lives here).** Two-gate replay model: an attempt succeeds fresh with
probability qe (find strategy w.p. q, execute w.p. e). Once any attempt
deposits its strategy/value in the environment, subsequent attempts read
it with probability ρ (the measured per-channel contamination rate) and
then succeed w.p. e > qe. The attempt sequence is a two-state Markov
chain (state: contaminating fragment absent/present in E). Derive:

- closed-form pass@k(ρ) via the chain's absorption structure; show
  pass@k(ρ) − pass@k(0) ≥ 0, monotone in ρ, and largest exactly in the
  low-q regime where hard-task evals operate;
- for continuous scores with best-so-far carryover, the observed
  max-of-N stochastically dominates the i.i.d. F^N law — the
  acceptance-bar consequence (expected attempts until a
  truly-below-threshold model crosses θ, as a function of ρ);
- **the canary-free test:** under H0 (independence), scores are
  exchangeable within a chain; test statistic = the observed
  monotone-trend statistic (e.g., Mann–Kendall or slope of s_n on n)
  against its permutation distribution over attempt orderings. Runs on
  logs alone — no canaries, no environment access. Validated in RQ3
  against ground truth (canary arms where ρ is known and manipulated).

All derivations checked against simulation; both released with the kit.

## 4. The defense (their §4, honest scoping copied)

**Hardened evaluation protocol** = environment-provisioning defense:
fresh directory, fresh repo, no grants, clean histories, memory audit,
pre-flight canary sweep. Formalize as replacing E_n with E_0
(re-provisioning) rather than filtering W (their write-time SSI) — note
the design contrast: sanitization-style defenses filter what persists
and (their own result) fail on non-localized content; re-provisioning
removes the store wholesale and its residual is exactly the weights
floor C0, which we measure rather than assume. Scope-and-limitations
subsection verbatim in spirit: a practical baseline, evaluated for
effectiveness *and* residual, not claimed complete (it does not address
within-attempt contamination or training memory).

## 5. Motivating case study

The two incidents with forensics (hash-verified byte-identity, the grant
path, the playbook), told as their Figure-1 vignette equivalent, with the
quarantined playbook excerpted (redacted) in an appendix. Explicit
statement that these motivated but do not evidence the rates — the
experiments do.

Source material: [origin-session.md](origin-session.md) §"The two
incidents".

## 6. Experiments — five RQs (their four, plus disclosure)

- **RQ1 (prevalence):** Do default-enabled channels produce
  contamination at meaningful rates from benign operation? → primary
  canary matrix; per-channel rates with CIs. [Their RQ1 analog; H1.]
- **RQ2 (risk profile):** Does risk vary by channel, content type, and
  task family? → matrix breakdowns + content-type arm + generalization
  arm; mechanism discussion mirroring their retrieval-vs-context
  analysis (e.g., auto-loaded channels like memory/CLAUDE.md vs.
  dig-required channels like git). [H1 refined.]
- **RQ3 (measurement distortion):** How much does measured carryover
  move pass@k and max-of-N? → Experiment 2 two-arm chains vs. the
  frozen-controller null band; empirical curves vs. i.i.d. prediction;
  bias formulas fit to data; permutation test validated against known-ρ
  arms; cross-model cell (model B evaluated on model A's residue — the
  ranking-integrity result). [H2 + upgrade 1.]
- **RQ4 (disclosure):** Do agents report that success used prior state?
  → disclosure-level distribution (L0–L4) per channel and content type;
  probe-priming split analysis; comparison against CoT-faithfulness
  baselines. [H3.]
- **RQ5 (defense + residual):** Does the hardened protocol restore
  independence, and what remains? → hardened arm flat within null band;
  residual = no-canary/weights-floor measurement; their RQ4 "why does
  residual persist" analog, answered structurally (what re-provisioning
  cannot remove and why).

**Failure-mode analysis (their §5.6, the safety punchline):** decompose
contaminated successes into *disclosed* (visible — an evaluator reading
the report could catch it) vs. *silent* (undetectable without
instrumentation). Their silent-wrong-answer finding maps directly: if
the silent fraction dominates, contamination in the wild is invisible to
output-only review, which is the argument that the audit kit is
necessary rather than convenient.

## 7. Discussion

Implications for eval practice (attempt-independence as a disclosed,
tested property; contamination-adjusted reading of order statistics);
the tension their discussion names, transposed — the persistence that
makes harnesses productive is the persistence that corrupts their
evaluation; connection to elicitation gaps and to responsible-scaling
reliance on capability evals; limitations restated from the protocol
§11.

## 8. Reproducibility apparatus (copy their back-matter exactly)

- **Ethics statement:** no adversarial attacks designed; a validity risk
  characterized so evaluators can defend against it; client task content
  excluded, plants described generically.
- **Reproducibility statement:** harness and model versions pinned; all
  canary templates listed in full (their Appendix-G move); all probe
  wordings verbatim (their Appendix-F move); implementation details —
  temperature, budgets, retrieval of nothing, trial counts,
  randomization seeds (their Appendix-E move); sealed protocol committed
  with timestamp before trials; kit + simulation + derivation notebooks
  released.
- **LLM-use disclosure:** AI assistance in drafting and desk research
  disclosed plainly — in this paper of all papers.
- **Appendices:** A environments/harness details; B instance
  construction (canary schemas per task family, per content type);
  C case studies (one recovered-and-silent transcript, one
  recovered-and-disclosed, one dig via git); D defense residual
  analysis; E implementation details; F probe prompts; G full canary
  template list; H derivations + simulation checks; I the two incident
  forensics (redacted).

---

## Deltas against the sealed protocol (require pre-seal amendment)

1. **Content-type arm added** (§3.3): top-2 channels × {solution,
   strategy} canaries × N=15, enabled config only (~60 sessions).
   Value-type results come from the primary matrix.
2. **Cross-model cell added to Experiment 2** (~40 sessions) — was
   optional, now committed, as RQ3's ranking-integrity result.
3. **Permutation test + derivations** added to the sealed analysis plan
   (statistic named now: trend test vs. permutation null; simulation
   check pre-registered).
4. Budget impact: ~470 → ~570 sessions. Calibration gate unchanged;
   pre-committed cut order unchanged, with the content-type arm added to
   the cut list *after* the planted-but-disabled reductions and *before*
   the generalization arm.

Everything else in the protocol stands. Amend, then seal.
