# Attempts Are Not Independent!

**My theory is that there is state carryover when using agents for evals, and this can occur a lot more often than suspected**

The idea at the heart of this project is that repeated evaluation attempts by a stateful coding
agent ('stateful' meaning that, it has a state at any given time, eg what eval attempt it is on, whether it was successful - something that we can actively keep track of and measure) are samples of the model AND an 'accumulated state', not of the model. 

We can measure which channels carry state across independent sessions and how much that carryover affects how successful the attempt was and whether the agent discloses that its success came from prior state.

This project idea was inspired from some work I was doing regarding using Fable 5 to create a controller solution to a task and environment I had set up in MuJoCo. The document logs include the work I did which led me to this project idea.

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
carryover between attempts. 

If attempt N can read what attempt N−1
wrote, the evaluation is essentially a learning curve which we are misreading to be a sample, and the
max statistic drifts upward with increasing attempts regardless of underlying
capability of the model. 

This corrupts capability estimates in both directions:
inflated eval results, but also, decreased difficulty
calibration. 

## Positioning

These are related works that I often refer to:

- Memory-security literature (poisoning, privacy leakage) covers the
  **adversarial** case, whereas I am not focusing on whether the model is becoming adversarial.
- Sandbagging literature covers the model underperforming its
  capability 
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

