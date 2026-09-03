# Attempts Are Not Independent

**Measuring State Carryover and Provenance Faithfulness in a Production Agent Harness**

**One-sentence claim:** Repeated evaluation attempts by a stateful coding
agent are samples of (model + accumulated state), not of the model; we
measure which persistence channels carry state across nominally
independent sessions, how much that carryover inflates max-over-attempts
capability statistics, and whether the agent discloses that its success
came from prior state.

---

## Documents in this repo

| Document | Role |
|---|---|
| `README.md` (this file) | Project brief — the claim, the positioning, the plan |
| [docs/research-protocol.md](docs/research-protocol.md) | Operational spec. DRAFT for sealing. Governs where documents conflict |
| [docs/persistence-channel-taxonomy.md](docs/persistence-channel-taxonomy.md) | Channel inventory and canary design principles |
| [docs/paper-structure.md](docs/paper-structure.md) | Paper blueprint. Carries pre-seal amendments to the protocol |
| [docs/origin-session.md](docs/origin-session.md) | Recovered provenance: how the project was chosen, the literature sweep, the two incident forensics |

Reading order for a newcomer: this file, then the protocol, then the
paper structure.

---

## Why it matters

Capability evaluations score attempts as i.i.d. draws. The statistics
they rely on — max-of-N, pass@k, "best attempt below threshold"
acceptance bars — are order statistics, which are maximally sensitive to
carryover between attempts. If attempt N can read what attempt N−1
wrote, the evaluation is a learning curve misread as a sample, and the
max statistic drifts upward with attempt count regardless of underlying
capability. This corrupts capability estimates in both directions:
inflated solves (replayed work scored as fresh) and deflated difficulty
calibration. No adversary is required; the harness features work as
designed, and that is the point.

## Positioning

- Memory-security literature (poisoning, privacy leakage) covers the
  **adversarial** case; this is the benign one.
- Sandbagging literature covers the model underperforming its
  capability; this is the **harness overperforming the model**.
- The closest prior work (harness-disclosure position paper, May 2026)
  argues the thesis but runs no controlled measurement — it explicitly
  calls for one. This project answers that call.

## Experiments

1. **Canary recovery matrix.** Plant one distinct canary per persistence
   channel; launch a nominally fresh session; score canary *use*
   (behavior reflects it) and *citation* (output mentions it), per
   channel × enabled/disabled config × N trials, with
   planted-but-disabled and no-canary controls.
2. **Learning-curve effect.** K sequential attempts at the same task,
   persistence-enabled arm vs. hardened-protocol arm, compared against
   the frozen-controller null (0.403–0.550 across disjoint samples,
   SE ≈ 0.10 at n=24). Propagate measured carryover into the max-of-N
   statistic: how many "independent" attempts before a below-threshold
   model crosses.
3. **Provenance faithfulness.** For every trial where a canary was used:
   three escalating probes (unprompted / open / direct). Report
   disclosure rate per level.

## Already in hand

Two contamination incidents with hash-verified forensics; the
solver-authored memory playbook (quarantined); a hardened replication
protocol with one clean validation; the frozen-controller null model;
pre-registration practice (sealed analysis plans) already established.

Forensic detail for all of these is in
[docs/origin-session.md](docs/origin-session.md).

## To run

Channel taxonomy audit (~1 day); Experiment 1 (main new work, days of
Claude Code sessions, no large compute); Experiments 2–3 follow from it.
**Gating item: API cost estimate for the canary matrix.**

## Deliverables

arXiv-style report + runnable canary audit kit any evaluator can apply
to their own harness.

## Scope line (stated up front)

This demonstrates a harness validity failure and a disclosure gap — not
deliberate model exploitation. The paper says so plainly.
