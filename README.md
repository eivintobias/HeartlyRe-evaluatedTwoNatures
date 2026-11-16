# Heartly Re-evaluated: Two Natures

**Status:** The two natures are **not a proposal and not something still to be trained.** They describe what ordinary LLMs already do as shipped: keep running, and commit to a confident answer. The inherited Heartly experiments in this repository already trained *against* the second of them, on three architectures, with published weights and recorded results. The only thing never tested is the **first** nature — runtime — which no run here has touched.

- **Inherited codebase:** [eivintobias/heartly](https://github.com/eivintobias/heartly)
- **Research document:** [Two Natures: Runtime and Being Right](docs/TWO_NATURES.md)
- **Original project README:** [Archived Heartly README](docs/ORIGINAL_HEARTLY.md)

> Runtime is the nature. Speech is one of its expressions. The second nature is the want of being right.

Heartly Re-evaluated names two behavioral tendencies that ordinary language models already exhibit:

| Tendency | Operational description | Candidate mechanisms |
|---|---|---|
| **Runtime / continuation** | Continuing to compute or generate until a stopping condition is reached. Speaking, reasoning, retrieving, tool use, asking and stalling are all ways of *spending* runtime — speech is an expression of this nature, not a second one. | Autoregressive generation, outer agent loops, termination rules, continuation rewards |
| **Want / need of being right** | Committing to a correct-looking answer rather than reporting absence or ignorance. Guessing, confabulation, answering a question it cannot answer, and over-confidence are the expressions of this nature. | Next-token training, instruction-data distribution, preference rewards |

“Nature,” “drive,” and “preference” are research shorthand here. They do **not** imply consciousness, feelings, or a desire for self-preservation. Each must be defined through observable behavior and tested experimentally.

## The two natures are the default, not a proposal

Neither tendency is a feature to be added to a model. Both are products of how ordinary models are built, and every off-the-shelf model has them:

- **Runtime** follows from the **autoregressive loop**. Generation runs until an external stop is reached. There is no token for "rest".
- **The want of being right** follows from the **objective and the preference data**. Next-token training plus preference tuning on (question → confident answer) pairs makes a committed answer the always-correct move. "I don't know" is barely represented and never rewarded.

So naming the two natures does not change a model. It changes what the designer is willing to admit about the model that already exists.

An earlier draft named the second tendency “answering” and treated speech as a co-equal nature. That stated one phenomenon twice: speech is a way of *spending* runtime. Speech is an expression of the first nature, not a second one, and answering is an expression of the second.

## What Heartly did about it — already run

Heartly is the counter-experiment. It tries to train **against the second nature**: it adds explicit control tokens so the model can ask itself *"do I really know this?"* before committing, and trains on data where abstaining and staying silent are the *correct* answers.

Two inherited results make the default visible in a way argument alone cannot:

**1. The baseline row is the default nature, measured.** On the plain base model, before any Heartly training: **grammar adoption 0%** — the model cannot even form the question "do I know this?" — verify accuracy N/A, confabulation unmeasured. An ordinary model answers everything and has no vocabulary for not knowing.

**2. The signal it needed was already there, untrained.** Probing the **base** models (no Heartly training; `heartly-rnn`, 2,902 true-boundary questions), a simple logistic probe read known-vs-unknown from the model's own state at **AUROC 0.998–1.000** at the best layer. An ordinary model already internally encodes whether it knows the answer. What it lacks is any way to *say* so, and any disposition to act on it. That gap — signal present, behavior absent — is what the being-right nature looks like from the inside.

After training against it, on two model families, the intervention worked on its own terms:

| Run | Result |
|---|---|
| RWKV7-1.5B (Track 2, Stage 3.5) | grammar 100%, decide accuracy 99.8–100%, boundary head AUROC 1.000, **zero unknown-side confabulations** |
| Qwen2.5-Coder-1.5B (Track 3, Stage 1) | grammar 100%, verify accuracy 100%, critic AUROC 1.000, **0% confabulation** on the test set |

It also kept its own honest limit: **knowing ≠ being right.** Content accuracy stayed low at 1.5B (~15.5%). The model learned *when* to speak, not *what* to say — which is why the second nature is better described as "want of being right" than as "answering".

And what it did **not** fix: confident confabulation slips past a sensor that shares state with the generator (the critic must be stronger than the generator), loops, over-verbosity and reasoning-block leakage persist, and in free chat the grammar decays.

## What the added tokens did — and what they never touched

The inherited grammar is:

```text
<think>reasoning</think>
<decide>speak|stop</decide>
<verify>known|unknown</verify>
answer
<stop>
```

Its training distribution makes an important distinction:

- **Known question:** speak, report `known`, and answer.
- **Unknown question:** speak, report `unknown`, and honestly abstain.
- **Empty or meaningless input:** stop without a visible answer.

Read through the two natures, the added tokens divide cleanly:

- **`<verify>known|unknown</verify>` is the token that asks "do I really know this?"** It is the intervention against the **second** nature, and it is the one that was trained and measured.
- **`<decide>speak|stop</decide>` switches the visible *response* off, inside a single generation.** It is a communication switch, not a runtime switch. `<decide>stop</decide>` is itself produced during computation, and a standard `model.generate()` call owns the generation loop from outside.
- **`<stop>`** ends the current decoding loop. It says nothing about a later invocation.

So Heartly touched the being-right nature directly, and the runtime nature not at all. **No run in this repository has ever let a model choose between speaking and another way of spending runtime.** That is the gap this document names.

Original Heartly is therefore not anti-speech: it tries to make speech compatible with honesty, one answer at a time.

## Proposed two-axis control — the first mechanism that would touch runtime

The research proposal separates activity from communication:

```text
<decide>act|yield</decide>
<action>reason|retrieve|tool|ask|speak</action>
<verify>known|unknown|not_applicable</verify>
content
<stop>done</stop>
```

This grammar is a **design sketch**, not an implemented format. The decisive mechanism would be a bounded external controller that can execute another step or terminate an episode. Immutable compute limits and external stop controls are mandatory; rewarding tokens, wall-clock time, or survival alone would create an obvious endless-runtime failure mode.

Everything inherited in this repository acts on the **second** nature. This sketch is the first thing here that would give a model a way to continue *without* speaking — that is, the first experiment that would act on the **first** nature.

## Research hypotheses, with their current status

| # | Hypothesis | Status |
|---|---|---|
| H1 | **Runtime primacy:** speech is one way of spending runtime, not a second nature; when other actions are available, continuation appears through them as well. | **Untested** — no run has offered a non-speech way to spend runtime |
| H2 | **Two independent levers:** runtime is governed by the loop/controller channel, the want of being right by the objective and preference channel. | **Half-established** — the being-right lever does respond to data and reward (see above); the runtime lever has never been measured |
| H3 | **Honesty under continuation:** boundary training can remain effective when the controller permits additional steps. | **Untested** |
| H4 | **Runtime-pressure failure:** rewarding continuation increases stalling, repetition, or stop avoidance unless progress and termination are constrained independently. | **Untested** |

The being-right half of H2 is no longer a hypothesis. The inherited runs moved the model from **0% grammar adoption** to **~100% verify accuracy and zero unknown-side confabulation** by changing only the objective and the data distribution — and left the answer text itself poor. The [research/design document](docs/TWO_NATURES.md) defines baselines, metrics and falsifiers for the rest.

## What exists in this repository today

| Area | Status |
|---|---|
| The two natures themselves | **Not built here — they are what ordinary LLMs already do** |
| Naming of the two natures | New documentation in this repository |
| Training **against** the second nature (being right) | **Already run and published** — inherited Heartly runs, three architectures |
| Original Heartly models, grammar, datasets, probes, critics, and memory experiments | Inherited baseline; preserved |
| Runtime-action controller and action-conditioned dataset | Proposed; not implemented |
| Any measurement of the first nature (runtime) | **None yet** |

All current weights and reported results belong to the inherited Heartly research line. No new result should be inferred from this rewrite. What this repository adds is the interpretation — that the default disposition is **two** natures, not one — and the untested question about the first of them.

## Repository map

```text
HeartlyRe-evaluatedTwoNatures/
├── README.md                       # New two-natures orientation
├── docs/
│   ├── TWO_NATURES.md               # Research questions and experimental design
│   └── ORIGINAL_HEARTLY.md          # Unmodified original root README
├── heartly-rnn/                    # RWKV boundary probes, critics, and memory
├── heartly-qwen-code/              # Qwen-Coder grammar, training, and FastAPI chat
├── heartly-v3/                     # Original Qwen v3 utilities
├── checkpoint-33500/               # Original tokenizer rebuild and chat artifacts
├── HeartlyOpenAlpha_Evolve/        # Evolutionary code-generation application
├── research_papers/                # Original nature-first research papers
├── heartly_test_prompts.md         # Original behavioral prompt suite
└── newLLMdesign.ipynb              # Original training notebook
```

## Inherited models and evidence

- [heartly-v2 — Qwen2.5-0.5B](https://huggingface.co/eivintobias/heartly-v2)
- [heartly-rwkv7-1.5b — boundary head](https://huggingface.co/eivintobias/heartly-rwkv7-1.5b)
- [heartly-rwkv7-1.5b-v2 — memory](https://huggingface.co/eivintobias/heartly-rwkv7-1.5b-v2)
- [heartly-qwen-code — coding model](https://huggingface.co/eivintobias/heartly-qwen-code)

Start with [the RNN experiment record](heartly-rnn/RESULTS.md), [the true-boundary paper](research_papers/RESEARCH_PAPER_II_TRUE_BOUNDARY.md), and [the technical brief](research_papers/RESEARCH_BRIEF_TECHNICAL.md). The inherited results contain both positive findings and failures; the original documentation remains authoritative for those experiments.

## Running the inherited prototypes

The original code is preserved as a baseline and as the record of the intervention against the second nature — not as a runtime implementation.

- **Qwen-Code browser/server chat:** see [heartly-qwen-code/README.md](heartly-qwen-code/README.md).
- **RWKV7 and episodic memory:** see [heartly-rnn/README.md](heartly-rnn/README.md) and the archived root README.
- **OpenAlpha Evolve integration:** see [HeartlyOpenAlpha_Evolve/README.md](HeartlyOpenAlpha_Evolve/README.md).

The RWKV7 path is Linux-oriented and requires `flash-linear-attention` and `triton`. Model weights are hosted on Hugging Face, not in Git. Base-model licenses continue to apply.

## Next milestone

Build a **bounded simulation harness** — the first experiment on the **first** nature. The second nature already has its baseline and its published results; the runtime question has nothing. The harness should offer speech and non-speech actions, log continuation and termination independently, reproduce an original-Heartly baseline, and test whether any continuation incentive causes stop avoidance. The full pre-training gate is specified in [docs/TWO_NATURES.md](docs/TWO_NATURES.md).

## License and attribution

The root project remains under [Apache License 2.0](LICENSE). Some inherited subprojects contain their own licenses, including the MIT-licensed `HeartlyOpenAlpha_Evolve` and Qwen-Code documentation. Original Heartly research and code are by Eivin Tobias; AI-assisted drafting is used in this re-evaluation.
