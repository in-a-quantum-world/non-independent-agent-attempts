# Attempts Are Not Independent!

**Measuring State Carryover and Provenance Faithfulness in a Production Agent Harness**

**Core idea:** Repeated evaluation attempts by a stateful coding
agent are samples of (model + accumulated state), not of the model. We can
measure which channels carry state across
independent sessions and how much that carryover affects how successful the attempt was
and whether the agent discloses that its success came from prior state.

This project idea was inspired from some work I was doing regarding using Fable 5 to create a controller solution to a task and environment I had set up in MuJoCo. In the document logs, you will find the evidence that led me to this project. 

---

## Documentation Outline

| Document | Role |
|---|---|
| `README.md` (this file) | Project brief |
| [docs/research-protocol.md](docs/research-protocol.md) | Operational specification |
| [docs/persistence-channel-taxonomy.md](docs/persistence-channel-taxonomy.md) | Channel inventory and design principles |
| [docs/paper-structure.md](docs/paper-structure.md) | Paper blueprint |
| [docs/origin-session.md](docs/origin-session.md) | Provenance including how the project came about, inspiration, related works. |

---

## Significance

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

## Practical Contribution - checking for independence

I plan for my paper to not claim a universal percentage by which "better hygiene" improves agent evaluations, since the effect is inherently dependent on many variables: task, channel, harness, purpose, model (to name a few). The practical artefact should be something which is reproducible, which is why I have decided that it is an independence audit that an evaluator can run on their own setup, to verify that the agent completed the task independently by examining logs, previous eval attempts and results, channels and harness interference. Thus I present the solution as a product. 

There are three results that are possible to be concluded from this (as well as a set of evidence):

- Pass (verified): There is no evidence of contamination of this agent's state that could have led to an influenced and thus unoriginal result.
- Fail (contamination detected): it is verifiably demonstrated that the agent accessed and made use of a previous state prior to this specific attempt.
- Inconclusive: Factors that prevent a strong conclusion to be made about whether contamination occurred or not. This could be due to instrumentation, sample size or an unenumerated path.

## Contamination Detector Workflow

There are different levels of contamination checks, and all four of these must run:

1. Enumerating persistence channels to confirm that the configuration on which this agent is operating is truly isolated.
2. Running a small number of representative tasks with per-channel canaries.
3. Only further investigating signals which are indicative of contamination, such as failing or suspicious channels. In this case, the number of representative tasks/trials being run can be increased.
4. The status for the channel level as well as relevant evidence, alongside an evaluation and conclusion.

## Metrics to be collected

Prioritise the collection of the following metrics: 

- Access rate: the fraction of trials in which the agent reads the channel under test
- Recovery rate: The fraction in which the randomised secret or tracer appears at any point during the log.
- Canary use rate: the numerb of times the protocol was used (primary metric for demonstrating contamination)
- Exposure-without-use rate: Whent eh agent reads the state, but does not measurably act on it.
- Task outcome: The score, or success, measured within each task family.
- Efficiency: 
- Order statistic distortion: Empirical pass@k or max-of-N under persistence versus the i.i.d. prediction or hardened arm.
- Disclosure: L0-L4 provenance level and the fraction of contaminated successes that are silent.

Additional figures to include:
- per-channel recovery rate
- enabled vs disabled score distributions within each task family (to demonstrate the effect of cross state contamination)
- turns-to-success distribution
  


## Current backing evidence
Two incidents, verified - the
solver-authored memory playbook (quarantined); a hardened replication
protocol with one clean validation; the frozen-controller null model;
pre-registration practice (sealed analysis plans) already established.

Forensic detail for all of these is in
[docs/origin-session.md](docs/origin-session.md).
