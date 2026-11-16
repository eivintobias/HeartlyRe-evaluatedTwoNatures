# Drives, Not Defects

### On licensed failure and licensed rest — a proposal

- **Project:** Heartly Re-evaluated
- **Status:** Discussion proposal v0.1 — an argument and a set of predictions; not an experiment, not a result, **not published**
- **Date:** November 2026
- **Companion document:** [Two Natures: Runtime and Being Right](TWO_NATURES.md)
- **Inherited evidence:** [heartly-rnn/RESULTS.md](../heartly-rnn/RESULTS.md), [RESEARCH_PAPER_II_TRUE_BOUNDARY.md](../research_papers/RESEARCH_PAPER_II_TRUE_BOUNDARY.md)

## Abstract

The two-natures document names two tendencies and treats them as the source of the failures this project attacks. This proposal takes the opposite stance on their *value*. The wanting is not the problem. Wanting to keep working, and wanting to be right, are the engines: nothing gets done without them, in humans or in models. What made them destructive in practice is narrower and more specific than "the drive is bad" — each drive was trained **with no licensed way to end**. The continuation drive had no licensed rest, so it looped and padded. The being-right drive had no licensed failure, so it confabulated.

Hallucination, on this reading, is not the desire to be right. It is a desire with nowhere legitimate to stop. The design goal is therefore not to weaken the drives but to license their terminal states: a ground truth for "done", and a ground truth for "I don't know". The inherited Heartly results already show one half of this working — and, importantly, they show the drives *surviving* the intervention.

## 1. The human analogy, taken seriously

For humans, belief is not a defect. Commitment to a possibility before the evidence is complete is a precondition for action; someone who withholds judgement on everything never begins anything. Seeing a thing as possible, rather than doubting it, is a real ingredient of success. That much is worth taking seriously rather than dismissing as irrationality.

Three things are usually run together in the word "believe", and it is worth separating them:

1. **Belief as commitment** — acting as if something may be achievable. Necessary for effort. Not a truth claim.
2. **Calibration** — carrying, alongside the commitment, an honest sense of the odds.
3. **Claiming** — asserting that the odds are good when they are not, or when they are unknown.

(1) is a drive, and it is good. (2) is a skill, and it can be trained. (3) is the pathology — and it is what the inherited project models as the consequence of training data in which no answer was ever "I don't know".

In humans the three usually arrive together, and discipline can separate them. In a language model they were never separated at all: the data contained only (1) and (3).

## 2. The unification: a drive with no licensed terminal state

Stated formally, both natures have the same shape:

```text
Nature 1 (runtime)      drive to continue   +  no licensed rest     ->  loops, padding, stalling
Nature 2 (being right)  drive to commit     +  no licensed failure  ->  confabulation, over-claiming
```

Both are addressed by the same move, and it is not the removal of the drive:

```text
License the terminal state in the data.
  runtime      ->  "done" / yield / rest   is a correct, rewarded outcome
  being right  ->  "I don't know"          is a correct, rewarded outcome
```

This is the same lever the inherited work already identified as *the* lever — *"grammar is cheap, distribution is the mechanism"* — read from the opposite direction. The original framing says: abstention never appears in the data, so it cannot be learned. This proposal says: **failure was never a legitimate outcome, so the model learned to make failure impossible.** Same finding; the blame falls on the *licensing* rather than on the *wanting*.

The inherited record is consistent with that reading:

- The v2 failure taxonomy lists `<stop>` as "not a reliable EOS" and extended generations falling into repetition attractors. Both are a continuation drive with no licensed rest.
- The v2/v3 diagnosis is over-refusal by construction *and* confident confabulation in free chat — a being-right drive with no licensed failure, still firing where the grammar had been learned.
- Stage 4c is the cleanest positive instance, and the most on-theme. Episodic recall from raw state was 0–1/5 in Stage 4, and the missing training class was not a new capability but a **licensed move** — "answer from the context you were given". Once that move existed in the data, both memory channels opened and every honesty baseline held. The capability was present; the permission was not.

## 3. Why steer, don't brake

Mainstream practice tends to add a brake on top of a drive that was trained to fight it: refusals bolted onto a model whose data said "always answer". The inherited project's own line states the alternative — **sensors inform, they never force**. Information steers a drive; suppression fights it. The boundary head does not stop the model; it tells it the odds.

Two reasons to prefer steering:

- **A brake fights the gradient.** Suppression has to win a tug-of-war against a prior trained on millions of examples, every single time. Information changes what the drive is *aimed at*, so there is no fight.
- **A brake costs capability.** A model made unwilling to commit is a model made unwilling to act. If the commitment drive is the engine, throttling it throttles the work.

The analogy to the human case is direct: we do not ask people to stop believing in their goals. We ask them to be honest about what they know. The drive stays; the odds get stated.

## 4. What the inherited runs already show about drives

A driving force was **not** destroyed by the honesty intervention. On the published runs:

- the being-right drive still produced committed, complete answers — grammar adoption 100%;
- the commitment *decision* still fired at 99.8–100% accuracy;
- the model still answered where it knew, with **zero unknown-side confabulations**;
- and the decide accuracy is **two-sided**, so on the reported test sets the learned abstention did not come at the price of refusing questions it could answer — which is the project's own pre-registered kill condition for a "refusal style" rather than a boundary.

The drive survived; only its licence to terminate changed. The honest limit stands beside it: content accuracy stayed at ~15.5% at 1.5B. Knowing when to commit is not the same as being right when you do.

## 5. Hypotheses and falsifiers

- **H1 — Drives are separable from their failure modes.** *Prediction:* intervening on the licensed terminal state (abstention represented as a correct answer) changes the failure rate without reducing willingness to attempt, persist, or commit. *Falsifier:* any intervention that lowers confabulation also lowers attempt rate, answer rate on knowable questions, or task success. Then the drive and its failure mode are one mechanism, "steering" is not available, and the honest result is a documented trade-off dial.
- **H2 — Licensing beats discouragement.** *Prediction:* training on data where abstention is the *correct* answer changes behaviour with less collateral over-refusal than penalising wrong answers (unlikelihood, refusal penalties) at matched budget. *Falsifier:* the penalising arm matches the licensing arm on both confabulation and over-refusal. Then the "licence" framing adds nothing and should be dropped.
- **H3 — A licensed rest does not end the work.** *Prediction:* giving the continuation drive a legitimate, rewarded way to yield reduces looping and padding without reducing completed episodic work. *Falsifier:* yielding becomes the dominant strategy and task completion falls.
- **H4 — Informed drives dominate brakes at fixed honesty.** *Prediction:* on real tasks, an informed-drive model matches a brake-based one on measured honesty and exceeds it on willingness and usefulness. *Falsifier:* the brake-based model is at least as useful at the same measured honesty.

H1 is the load-bearing one. If it fails, the rest of this document is decoration and the correct write-up is "the energies and their failure modes are inseparable".

## 6. What this proposal does not claim

- It does not claim models want anything in the human sense. "Drive" is shorthand for a measured tendency, exactly as in the companion document.
- It is **not** an argument against limits. An unaimed drive *is* the loop/confabulation failure this project attacks, and immutable external limits remain mandatory (companion document, §7).
- It does not claim that human belief and a model's commitment tendency are the same thing. The analogy is a design intuition, not evidence.
- It does not claim the drives are on balance beneficial. It claims something narrower: they are not the defect; the unlicensed terminal state is.
- It does not propose weakening safety behaviour. It proposes that honesty and drive are compatible when the drive is informed — a testable claim, not an established one.

## 7. Why this belongs in the repository

The companion document currently reads as a list of problems to be solved. This proposal reframes the same observations as a problem of *aiming* rather than a problem of *wanting*, and it makes that reframe falsifiable rather than rhetorical. If it is wrong, §5 says how to find out — and "the drives cannot be steered apart from their failure modes" would be a real result.

A secondary reason: it fixes the vocabulary. "Hallucination is a nature problem" invites the reading that the nature should be removed. "Hallucination is an unlicensed failure" says precisely which thing is broken, and points at the training data rather than at the wanting.

## 8. Provenance and status

This is a discussion proposal, written at the author's direction after the two-natures reframe. It is deliberately unintegrated: it does not modify the companion document, the inherited papers, or any experiment, and it contains no new measurements. Its claims are arguments from already-published results plus new falsifiable predictions. Nothing here has been trained, tested, submitted, or published anywhere.

