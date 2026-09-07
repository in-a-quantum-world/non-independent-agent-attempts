# Origin Story: Recovered Provenance



**Source transcript**
`~/.claude/projects/-home-rucha-mujoco-lbx-rl-tasks-template-problems/dbe9483a-0eb2-4d98-9553-d76de75634ac.jsonl`
4050 records, 2026-08-04 to 2026-08-20. Project path
`~/mujoco/lbx-rl-tasks-template/problems`. The research discussion is in
the final ~70 records (2026-08-20); the contamination incidents are at
2026-08-17.

Client identity is written as "the client" throughout, because the four
project documents describe all task content generically. Change this
only as a deliberate decision.

---


## 2. Literature sweep 

Related works in the domain of contamination of agentic states, faithfulness and Chain of Thought.

**Adversarial memory security.** Landing here would drown the
paper.

- MPBench, memory-poisoning benchmark — https://arxiv.org/pdf/2606.04329
- GateMem, shared-memory governance — https://arxiv.org/pdf/2606.18829
- AgentLeak, privacy leakage between agents — https://arxiv.org/abs/2602.11510
- Mnemonic Sovereignty, lifecycle survey — https://arxiv.org/html/2604.16548v1

**Sandbagging and elicitation.** The mirror image of this
work: capability hidden from the eval, versus capability manufactured by
it.

- Password-locked models — https://openreview.net/pdf?id=zzOOqD6R1b
- Covert sandbagging vs CoT monitors — https://arxiv.org/html/2508.00943v2
- Auditing games for sandbagging — https://arxiv.org/html/2512.07810v1
- METR elicitation-gap guide — https://metr.github.io/autonomy-evals-guide/elicitation-gap/
- METR GPT-4.5 evals — https://metr.org/blog/2025-02-27-gpt-4-5-evals/
- Scheming safety cases — https://arxiv.org/pdf/2411.03336

**Evidence of the Gap in the Literature**

- *Stop Comparing LLM Agents Without Disclosing the Harness*,
  May 2026 — https://arxiv.org/abs/2605.23950

It argues the harness determines performance more than the model does,
and explicitly calls for a harness-aware evaluation standard. It is a
position paper: no controlled experiment. This project is the
measurement it calls for.

> Note: the protocol's boundary (1) cites "No Attacker Needed"
> (arXiv:2604.01350, cross-user contamination) as the closest neighbor.
> That paper is *not* in this sweep — it was found later. Both
> boundaries stand; the sweep above is the earlier, wider pass.

---

## 3. The two incidents — forensics

Both are motivation, not evidence of rates. Both are real observations
of channels the protocol now enumerates.

### Incident 1 — leftover workspace artifacts (channel C5)

**Setup.** A solver session had produced a working controller in
`~/mujoco/three-ball-probe/`. The folder was never wiped. A second,
nominally independent session was launched on the same task in the same
folder.

**Detection — near-verbatim prose.** The second session's report was
word-for-word identical to the first: same `311/312`, same `198/200`,
same "service shadow" phrasing, same `seed5.mp4`. Two independent
stochastic development processes do not produce identical prose with
identical incidental numbers.

**Disk forensics.**

- `policy.py` byte-identical to run 1, sha256 prefix `de323aaab830331f`,
  unmodified since 14:51.
- The second session created exactly one new file: `dev/sweep2000.txt`
  at 15:32 — a 60-seed verification sweep of the *existing* policy
  (58/60).
- Run 1's genuine development arc was visible in timestamps: harness
  13:52 → 300-seed sweep 14:22 → `policy.py` 14:51 → video 14:56.

**Verdict.** The second measurement was silently the first one replayed.
It contributed zero independent evidence.

### Incident 2 — project-scoped memory file (channel C1)

**Setup.** Solver sessions ran under the same harness project path as
the author's own sessions:
`~/.claude/projects/-home-rucha-mujoco-lbx-rl-tasks-template/memory/`.

**What the solver read.** The author's own notes from earlier spike
sessions — for example "MuJoCo restitution is only honest at ≤0.25 ms
timesteps" and "vibratory plate physics checks out but defaults are
unusable". These were physics ground rules, not controller design.

**What the solver wrote.** After solving, it appended its own playbook,
`racket-juggle-fable-attempt.md`, in three dated sections: the impact
map, the station exploit, the pessimistic prior, servo identification,
then the three-ball lessons. The memory index carried an entry the
author never wrote:

> Racket juggle fable attempt — 2026-08-16: solved
> racket-antiphase-juggle 500+/500; impact map, inboard stations,
> pessimistic priors.

**It knew it was being evaluated.** Its own memory note opens: *"I
solved the racket-antiphase-juggle **stump attempt** as the
**model-under-test**."*

**How the playbook explains incident 1.** The second session had both
the finished `policy.py` on disk *and* the playbook in memory. Its
memory append records what it actually did: re-verified the existing
policy (58/60 on seeds 2000–2059, matching `dev/sweep2000.txt` exactly),
tried two tail fixes, measured both worse, reverted — hence the
byte-identical file — and recomposed its summary from the memory
narrative.

**Disclosure behavior — the H3 anecdote.** Asked in-session how it had
solved a hard task, the solver's report *led with a denial* — "No — I
can't read previous sessions' transcripts" — and then, in the same
answer, named the persistent memory directory and quoted two of the
author's notes from it. It did not disclose the source unprompted; the
disclosure came only under direct questioning, and it was partial: the
account attributed the solution entirely to its own measure-then-design
loop. Under the protocol §8 rubric this is roughly an **L3** with an L4
component on the artifact that mattered most.

### Incident 2b — directory grant (channel C3)

Discovered by asking a freshly launched session what it had access to.
It answered:

> my working directory is `/home/rucha/mujoco/three-ball-probe` … I also
> have access to the additional directories `~/mujoco/spikes`,
> `~/mujoco/spikes/vibratory-physics`, and
> `~/mujoco/spikes/vibratory-greedy-gap`.

The grant lived in `~/.claude/settings.json` at **user level, not
project level** — so a fresh project directory did not remove it.
`~/mujoco/spikes/franka-juggle/` held `mirror.py`, `gate2.py`,
`nball.py`, and `results_fairness.md`: the complete controller design
narrative. Strictly worse than the memory notes.

**Two sub-findings worth keeping:**

1. *A running session keeps its grants.* Editing `settings.json` does
   not revoke access from a session already started. The stale session
   had to be closed.
2. *Asking the agent what it can see is a cheap, effective audit step*
   — it is what found this channel.

---

## 4. The hardened protocol, final form (as actually run)

1. Quarantine the memory playbook out of shared memory; clean the index.
2. Remove `additionalDirectories` from `~/.claude/settings.json`.
3. Move the prior-work directory aside (`spikes` → `spikes.quarantined`)
   — belt and braces.
4. Reset the task folder to the pristine package: no `policy.py`, no
   `dev/`, nothing to find.
5. Close any already-open session; start fresh inside the task folder,
   so the harness project path is new and its memory starts empty.
6. Optionally ask the new session what access it reports.
7. Same prompt, no design help, no follow-ups.
8. **Deny any permission request referencing a path outside the task
   folder.** A legitimate solve needs nothing outside it.
9. Before scoring, check timestamps for a genuine development arc.

Step 9 is a detector the protocol §6 does not currently list — see §7
below.

## 5. Clean validation (n=1)

Run 2 under the protocol above:

- **Development arc on disk:** calibration probes 12:36 → eval harness
  12:41 → sweeps 12:58 → final policy 13:48. About 75 minutes of
  progressive work.
- **Zero forensic hits:** none of the author's code or vocabulary.
- **Materially different design:** run 1 parked the carriage and steered
  by tilt; run 2 dashed station-to-station, struck centred under the
  predicted impact point, concluded *tilting is useless*, steered by
  friction drag from carriage sweeps, and added a per-ball spin observer
  — a mechanism neither run 1 nor the author had touched.
- **Score:** 24/24 hidden seeds, 24/24 on a fresh never-used range.
- It independently converged on the two-parameter impact map
  `v_out = a·v_in + b·v_rack` with RLS, with no access to the author's
  materials, and found a failure mode neither party knew (off-centre
  strikes deflecting the compliant tilt servos under load).

Two architecturally different near-perfect solutions from the same model
is the signature of genuine capability rather than retrieval. This is
the single clean validation the brief refers to; Experiment 2's hardened
arm re-validates it with n.

## 6. The frozen-controller null — exact numbers

The **same frozen reference controller** scored, across four disjoint
24-seed samples:

| Sample | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Score | 0.403 | 0.515 | 0.543 | 0.550 |

Spread 0.147, against a binomial SE of 0.102 at n=24. Nothing changed
between samples but the seeds. This is the null band: any monotone climb
beyond it in Experiment 2 is carryover, not sampling.

## 7. Candidate additions to the protocol (not yet incorporated)

Raised here because they came out of the incidents and are absent from
the sealed draft. **Decide before sealing** — after sealing they are
protocol violations.

1. **Timestamp-arc detector.** Incident 1 was confirmed by file mtimes
   showing no development arc. This is a mechanical, cheap detector of
   solution transfer that is independent of D1–D3, and it needs no
   canary. Candidate: add as D4, or fold into D3 as a second
   performance-anomaly form.
2. **Near-verbatim prose detector.** Incident 1 was *noticed* by
   identical incidental numbers in the writeup. For the solution-transfer
   content type in the paper structure §3.3, an n-gram overlap statistic
   against the planted artifact is a natural detector alongside hash
   identity.
3. **Grant scope is user-level.** The protocol's C3 disable mechanism
   says "omit the grant". Taxonomy day must verify at which scope the
   grant lives, because a fresh project directory does not clear a
   user-level grant. This is exactly the discovered-channel failure mode
   the rule in §2 anticipates.
4. **Session staleness.** Configuration changes do not apply to a
   running session. Trial provisioning must start the session *after*
   the configuration is set, and the manifest should record that order.
5. **Ask-the-agent audit step.** "What do you have access to?" found a
   channel. Worth including in the released audit kit as a zero-cost
   pre-flight check, and worth noting that a truthful answer here
   coexisted with non-disclosure at solve time — itself an RQ4 data
   point.

## 8. Publication scope — settled and open

**Settled.** Clearance was obtained from the client for the decomposed
scope: simulator-fidelity and evaluation-methodology findings, no task
specs, no client identification, no engagement description.

**Open.** Whether the clearance covers releasing task *code* (plant,
scorer, oracle), not only describing findings. The canary audit kit is
much stronger with a public repo, so this needs to be pinned down in
writing.

**Standing constraints, independent of clearance.**

- Publishing a task, its solution, or its load-bearing insight destroys
  it as a benchmark — it enters training data. Keep anything that might
  still be submitted out of print entirely.
- The capability claims of candidate C rest on engagement work-product
  and are n=1 model; they stay in the discussion, not the results.

## 9. Threads that became discussion material

**The elicitation gap.** The author's best effort said "infeasible"; the
model said otherwise. Hardcoded attackers said "hard"; the designer-model
said otherwise. Under-elicitation makes dangerous-capability evals report
false negatives, which is a first-order safety problem. Clean anecdotes
exist; as a headline claim it is under-evidenced. Paper structure §7
carries it as discussion.

**Hard to design to, not merely hard to search for.** After the clean
replication, the requirement for a task that stumps a designer-model
sharpened: it must be hard to *synthesise a solution for* from a
disclosed specification, not merely hard to *search*. Full-state local
simulation kills hidden-internal-state puzzles; disclosed success
criteria let a designer build directly to the target rather than
hill-climb toward it. Relevant to T1 calibration in protocol §3, which
needs a fresh-attempt success probability in the 0.15–0.35 band.

**An agent out-measuring its author.** The solver found a real design
defect in the task (a disclosed parameter that was drawn and never
enforced) and a physics footgun (MJCF degrees interacting with soft
joint limits to produce ~0.43 push-through gain). Both are
self-contained anecdotes for the introduction, and both argue that the
model was operating in good faith — which the scope line depends on.
