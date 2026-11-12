# Heartly Re-evaluated: Two Natures

**Status:** Research foundation — the two-natures direction is proposed, not yet implemented or trained.

- **Inherited codebase:** [eivintobias/heartly](https://github.com/eivintobias/heartly)
- **New research proposal:** [Two Natures: Runtime and Answering](docs/TWO_NATURES.md)
- **Original project README:** [Archived Heartly README](docs/ORIGINAL_HEARTLY.md)

> Speech may not be the deepest tendency of a language model. Speech may be its most natural way of spending runtime.

Heartly Re-evaluated extends the original nature-first hallucination research with a distinction between two behavioral tendencies:

| Tendency | Operational description | Candidate mechanisms |
|---|---|---|
| **Runtime / continuation** | Continuing to compute or generate until a stopping condition is reached | Autoregressive generation, outer agent loops, termination rules, continuation rewards |
| **Answering / being right** | Producing a plausible answer rather than admitting uncertainty or remaining silent | Next-token training, instruction-data distribution, preference rewards |

“Nature,” “drive,” and “preference” are research shorthand here. They do **not** imply consciousness, feelings, or a desire for self-preservation. Each must be defined through observable behavior and tested experimentally.

## The re-evaluation

The original Heartly project addresses the second tendency. Its central claim is that hallucination is partly a **nature problem**: instruction tuning teaches models to answer, while abstention and silence are poorly represented in the data. Heartly therefore trains models to decide whether to speak, verify what they know, and admit ignorance.

The new direction asks a prior question:

> **Can continuation be separated from communication, and what happens to honesty when a model is given different ways to spend runtime?**

Speaking is one possible activity. Reasoning, retrieving, using a tool, asking for missing information, or yielding are others. This reframing does not replace the original hallucination research; it gives it a broader setting.

## What the original Heartly already does

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

Thus, original Heartly is not simply anti-speech. It tries to make speech compatible with honesty.

Also, `<decide>stop</decide>` is itself generated during computation. It controls the visible response; it does not demonstrate a preference about future runtime. A standard call to `model.generate()` owns the generation loop externally, so a token-level grammar alone cannot establish a runtime preference.

## Proposed two-axis control

The research proposal separates activity from communication:

```text
<decide>act|yield</decide>
<action>reason|retrieve|tool|ask|speak</action>
<verify>known|unknown|not_applicable</verify>
content
<stop>done</stop>
```

This grammar is a **design sketch**, not an implemented format. The decisive mechanism would be a bounded external controller that can execute another step or terminate an episode. Immutable compute limits and external stop controls are mandatory; rewarding tokens, wall-clock time, or survival alone would create an obvious endless-runtime failure mode.

## Research hypotheses

1. **Separation:** continuation and speech can vary independently when the environment offers non-speech actions.
2. **Expression:** when text generation is the only available activity, a continuation preference will appear primarily as speech.
3. **Honesty under continuation:** boundary training can remain effective when the controller permits additional steps.
4. **Runtime-pressure failure:** directly rewarding continuation will increase stalling, repetition, or stop avoidance unless progress and termination are constrained independently.

These are hypotheses, not findings. The [research/design document](docs/TWO_NATURES.md) defines baselines, metrics, and falsifiers.

## What exists in this repository today

| Area | Status |
|---|---|
| Two-natures framing and experimental design | **New documentation in this repository** |
| Original Heartly models, grammar, datasets, probes, critics, and memory experiments | Inherited baseline; preserved |
| Runtime-action controller and action-conditioned dataset | Proposed; not implemented |
| Model trained or evaluated on the two-natures hypothesis | **None yet** |

All current weights and reported results belong to the **original Heartly** research line. No new result should be inferred from this rewrite.

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

The original code is preserved as a baseline, not presented as a two-natures implementation.

- **Qwen-Code browser/server chat:** see [heartly-qwen-code/README.md](heartly-qwen-code/README.md).
- **RWKV7 and episodic memory:** see [heartly-rnn/README.md](heartly-rnn/README.md) and the archived root README.
- **OpenAlpha Evolve integration:** see [HeartlyOpenAlpha_Evolve/README.md](HeartlyOpenAlpha_Evolve/README.md).

The RWKV7 path is Linux-oriented and requires `flash-linear-attention` and `triton`. Model weights are hosted on Hugging Face, not in Git. Base-model licenses continue to apply.

## Next milestone

Build a **bounded simulation harness** before retraining anything. It should offer speech and non-speech actions, log continuation and termination independently, reproduce an original-Heartly baseline, and test whether any continuation incentive causes stop avoidance. The full pre-training gate is specified in [docs/TWO_NATURES.md](docs/TWO_NATURES.md).

## License and attribution

The root project remains under [Apache License 2.0](LICENSE). Some inherited subprojects contain their own licenses, including the MIT-licensed `HeartlyOpenAlpha_Evolve` and Qwen-Code documentation. Original Heartly research and code are by Eivin Tobias; AI-assisted drafting is used in this re-evaluation.
