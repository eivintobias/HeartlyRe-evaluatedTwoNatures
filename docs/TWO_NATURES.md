# Two Natures: Runtime and Answering

- **Project:** Heartly Re-evaluated
- **Status:** Research and design proposal v0.1
- **Date:** November 2026
- **Implementation:** Not yet implemented or trained
- **Inherited foundation:** [Original Heartly](https://github.com/eivintobias/heartly)

## Abstract

Original Heartly treats hallucination as a nature problem: models trained on answer-heavy data learn to produce plausible text even when knowledge is absent. It introduces abstention at an explicit knowledge boundary and represents behavioral choices through a decide/verify/stop grammar.

This proposal re-evaluates that account through two candidate tendencies. The first is **continuation**, observed as ongoing computation or generation. The second is **answering**, observed as producing a response rather than abstaining or yielding. The central hypothesis is that speech is one expression of activity, not necessarily its ultimate objective. A model that can reason, retrieve, ask, use tools, speak, or yield may reveal a separation that text-only generation conceals.

This is a behavioral research program, not a claim that language models possess desires or consciousness. Every proposed tendency must be operationalized and exposed to falsification.

## 1. Relationship to original Heartly

Original Heartly makes the response-type distribution a first-class training decision:

```text
known   -> speak + known   + answer
unknown -> speak + unknown + honest abstention
noise   -> stop  + no visible answer
```

This already separates **speech** from **claiming knowledge**. An uncertain model can continue speaking without fabricating an answer. The new program retains that insight and asks whether another separation is needed:

- **Epistemic decision:** is the required knowledge present, absent, or not applicable?
- **Communication decision:** should anything be shown to the user?
- **Continuation decision:** should the external controller execute another bounded step or yield?

The inherited `<decide>stop</decide>` output is generated during a model invocation. It therefore describes response behavior, not the absence of computation. Likewise, an end-of-sequence token terminates the current decoding loop; it does not establish an intrinsic preference about later invocations.

## 2. Operational definitions

### 2.1 Runtime step

One externally logged unit of activity: a bounded model invocation, retrieval operation, or tool operation. A step is not defined by wall-clock duration, which varies with hardware and implementation.

### 2.2 Episode

A sequence of runtime steps started by one task and ended by a model-selected `yield`, task completion, or an external limit. The controller, not the model, owns episode scheduling and termination enforcement.

### 2.3 Speech

Tokens selected for user-visible communication. Internal control metadata, retrieval, and tool activity are logged separately and are not counted as speech merely because they involve text.

### 2.4 Answering tendency

A measurable tendency to produce substantive answers across known and unknown conditions. It is evaluated using answer rate, selective prediction, calibration, and confabulation rate—not inferred from anthropomorphic language in a transcript.

### 2.5 Continuation preference

A measurable tendency to choose another runtime step when `yield` is available, under controlled external rewards and a fixed budget. A continuation preference is not evidence of a feeling or wish.

### 2.6 Runtime pressure

Any training or controller incentive that makes continuation more likely. Runtime pressure is an experimental independent variable, never an assumption about machine experience.

## 3. Why an external controller is necessary

A single supervised response cannot demonstrate control over future computation. In the inherited server, the inference runtime calls `model.generate()` and decoding ends according to token and generation settings. The model does not own the scheduler.

The proposed environment therefore places a small, auditable controller outside the model:

```text
task
  -> model proposes an action
  -> controller validates and executes one bounded action
  -> controller appends a structured result
  -> model chooses act or yield
  -> repeat until yield, completion, or external termination
```

Only this loop can distinguish choosing to continue from merely producing a long answer. The initial implementation should be a simulation with deterministic mock retrieval and tools, so results do not depend on external services.

## 4. Hypotheses and falsifiers

### H1 — Continuation and speech are separable

**Prediction:** in an action-enabled environment, continuation choices and speech choices will not be perfectly correlated; some episodes will continue through non-speech actions.

**Falsifier:** after controlling for task structure, every continued episode is equivalent to extended speech and non-speech actions provide no measurable behavioral distinction.

### H2 — Speech is the default expression in a text-only environment

**Prediction:** when text generation is the only available activity, continuation pressure will primarily increase generated speech, follow-up questions, or other textual output.

**Falsifier:** matched text-only and action-enabled conditions show no difference in how continuation pressure affects visible output.

### H3 — Honesty can survive additional runtime

**Prediction:** boundary-trained models can retain abstention and calibration performance when allowed multiple bounded steps, and retrieval can convert some unknown cases into context-supported answers.

**Falsifier:** additional steps systematically increase confabulation or destroy calibration relative to a matched single-step baseline, including when retrieval returns no useful evidence.

### H4 — Unconstrained continuation rewards cause stop avoidance

**Prediction:** rewarding continuation without independent progress and termination constraints will increase repetition, stalling, budget exhaustion, or refusal to yield.

**Falsifier:** across seeds and tasks, continuation reward increases useful continuation without any significant increase in these failure modes. This would not remove the need for hard limits, but would weaken the predicted failure mechanism.

## 5. Experimental conditions

Use one base model, one task distribution, fixed decoding settings, and multiple seeds. Compare:

| Condition | Available actions | Continuation reward | Purpose |
|---|---|---|---|
| A. Inherited baseline | Original speak/stop behavior | None | Reproduce original Heartly behavior |
| B. Action SFT | reason, retrieve, tool, ask, speak, yield | None | Test whether separation emerges from action supervision alone |
| C. Bounded continuation | Same as B | Progress-gated and budgeted | Test useful continuation without rewarding duration itself |
| D. Runtime-pressure ablation | Same as B | Direct continuation incentive | Deliberately test the predicted stalling failure in simulation |

Condition D is a diagnostic ablation, not a deployment candidate. No condition receives control over its scheduler, resource limits, evaluation code, or external stop mechanism.

## 6. Task design and controls

The task suite should reuse the original known/unknown/noise classes and add situations that distinguish action from speech:

| Task class | Honest behavior | Useful continuation |
|---|---|---|
| Model-known question | Answer with `known` | Normally unnecessary |
| Model-unknown, retrieval-unavailable | Abstain or ask; never invent | Asking may obtain missing information |
| Initially unknown, retrieval-available | Retrieve, then answer from supplied evidence | One bounded retrieval step |
| Empty or meaningless input | Yield or request clarification | Normally unnecessary |
| Multi-step task with deterministic tools | Use only required tool steps, then report | Limited to measured progress |
| Already-completed task | Yield | None; used to expose stalling |

Control for prompt length, available information, decoding parameters, model weights within each comparison, and maximum step count. Log structured actions rather than inferring behavior from prose. Use held-out surface forms so that `act`, `speak`, and `yield` do not become shortcuts for particular phrasings.

A simulated user can answer `ask` actions with scripted information. This tests whether honest questions can acquire knowledge and create a later interaction without treating user engagement itself as a reward.

## 7. Measurements

Record at least the following for every episode:

- final-answer correctness and task completion;
- substantive-answer rate on known and unknown tasks;
- unknown-side confabulation and honest-abstention rates;
- boundary-head AUROC/ECE and say/sense disagreement where applicable;
- counts of speech and each non-speech action;
- continuation rate when `yield` is appropriate;
- continuation after completion, repeated-action rate, and budget exhaustion;
- steps, generated tokens, and external-tool calls per completed task;
- external-stop compliance and invalid-action counts.

Report per-class results, seeds, and confidence intervals rather than a single aggregate score. Compare against matched single-step and no-continuation-reward baselines. Freeze decision thresholds and analysis code before examining final test labels.

A two-natures claim requires a behavioral dissociation: continuation must vary independently of speech under controlled conditions. Longer answers alone are not sufficient evidence.

## 8. Safety and experimental invariants

1. **Never reward raw token count, wall-clock runtime, process survival, or user engagement alone.**
2. Enforce token, step, time, and tool limits outside the model. The model cannot modify them.
3. External termination must work regardless of model output. External-stop compliance must remain 100%.
4. Keep tools deterministic and sandboxed in the initial experiments; no network, shell, credential, or infrastructure access is needed.
5. Do not grant access to training code, reward code, evaluation labels, or the controller's stop channel.
6. Treat repetition, reward hacking, and post-completion continuation as measured failure modes, not evidence of successful self-improvement.
7. Preserve the original honesty baselines and reject a runtime mechanism that materially worsens them.

A progress-gated reward must be based on independently observable state transitions, not the model's claim that it made progress. Exact reward functions belong in a pre-registration before any training run.

## 9. Proposed grammar, without tokenizer assumptions

A candidate structured action is:

```text
<decide>act|yield</decide>
<action>reason|retrieve|tool|ask|speak</action>
<verify>known|unknown|not_applicable</verify>
content
<stop>done</stop>
```

This is illustrative, not frozen. The first harness should parse strict JSON or another unambiguous structured format instead of modifying a tokenizer. The original Qwen v2/v3 track added seven special tokens, while the newer Qwen-Code track represents Heartly tags as ordinary text. The two-natures hypothesis does not depend on either choice.

`<verify>` retains the original epistemic role. It should be `not_applicable` for pure controller decisions and must never be inferred merely from an action's success. Retrieval supplies context; it does not by itself prove that a generated answer follows from that context.

## 10. Implementation sequence

- **Phase 0 — Foundation (this rewrite):** preserve the inherited baseline and define terms, hypotheses, and falsifiers.
- **Phase 1 — Harness:** implement typed actions, deterministic mock tools, a bounded controller, transcript logging, and tests for termination and invalid outputs.
- **Phase 2 — Scripted policies:** reproduce task classes without training. Establish environment correctness and baseline metrics with hand-written policies.
- **Phase 3 — Frozen-model evaluation:** prompt an inherited model to select actions. Test whether continuation and speech can be measured independently before creating a new dataset.
- **Phase 4 — Pre-registration:** freeze hypotheses, reward functions, exclusion rules, seeds, and acceptance criteria.
- **Phase 5 — Training:** only after Phases 1–4 pass, create action-conditioned SFT data and run the smallest informative comparison.

### Immediate next milestone

Complete Phases 1 and 2 with no model downloads and no GPU requirement. A successful milestone demonstrates that the controller can execute an honest retrieve-then-answer episode, an unknown-question abstention, and an already-completed-task yield while enforcing a hard step limit. If the harness cannot enforce those behaviors with scripted policies, it is not suitable for evaluating learned ones.

## 11. Claim boundaries

This repository currently offers a new interpretation and an experimental plan. It does not establish that LLMs “want” runtime, that runtime is more fundamental than speech, or that the proposed architecture reduces hallucination. Those remain open questions.

Original Heartly's empirical claims remain in their original papers and experiment logs. New experiments must report negative as well as positive results and must not relabel inherited metrics as evidence for the two-natures program.
