# Caveman Model Research

Last checked: 2026-07-05

This note tracks open-weight models and adapters that bake caveman-style terse output into model weights instead of relying on the `caveman` skill prompt. Focus: thinking or reasoning-family models that reduce visible chain-of-thought or verbose answers.

## Short Answer

There are caveman fine-tunes. There was no direct VibeThinker caveman fine-tune found in this pass.

Best current leads:

| Model | Base | Fit | Evidence | Caveat |
|---|---|---|---|---|
| [`Mintzs/caveman-qwen3`](https://huggingface.co/Mintzs/caveman-qwen3) | Qwen3-8B | Best match for "thinking model made terse" | Project README says Qwen3 default thinking output is suppressed and essential answer is emitted instead. | GGUF only. Benchmark is token count first; quality eval is limited. |
| [`njmason/caveman-qwen3.6`](https://huggingface.co/njmason/caveman-qwen3.6) | Qwen3.6-35B-A3B MoE | Strong reasoning-family LoRA lead | Adapter trained for native brevity. Card says `enable_thinking=False` gives terse-only behavior; thinking can stay enabled for harder tasks. | Adapter only. Smoke-tested, no full benchmark suite. |
| [`durgasai299792458/qwen3-4b-caveman-THINKING`](https://huggingface.co/durgasai299792458/qwen3-4b-caveman-THINKING) | Qwen3-4B | Explicit "thinking" caveman artifact | Actual Qwen3 safetensor model exists. | No README or training/eval detail. Treat as experimental. |
| [`JBrussee/gemma-4-31B-caveman`](https://huggingface.co/JBrussee/gemma-4-31B-caveman) and [`JBrussee/gemma-4-31B-caveman-lora`](https://huggingface.co/JBrussee/gemma-4-31B-caveman-lora) | Gemma 4 31B IT | Best documented native caveman model | Full card includes training data, recipe, holdout eval, merged model, and LoRA adapter. | Not specifically a "thinking model"; compression weaker than gold prompt pairs. |

## VibeThinker Check

[`WeiboAI/VibeThinker-3B`](https://huggingface.co/WeiboAI/VibeThinker-3B) is a compact reasoning model for verifiable tasks such as math, code, and STEM. Its model card says it is not trained for tool calling or autonomous coding-agent workflows. Search found no VibeThinker derivative fine-tuned specifically for caveman-style terse output.

Useful VibeThinker references:

- [`WeiboAI/VibeThinker-3B`](https://huggingface.co/WeiboAI/VibeThinker-3B)
- [`WeiboAI/VibeThinker`](https://github.com/WeiboAI/VibeThinker)
- [VibeThinker-3B paper](https://huggingface.co/papers/2606.16140)

## Detailed Notes

### Mintzs/caveman-qwen3

Project: [`Mintzs/oogaboogalm`](https://github.com/Mintzs/oogaboogalm)

The repo describes OogaBoogaLM as QLoRA fine-tunes of Qwen2.5-7B-Instruct and Qwen3-8B trained to produce minimized direct responses by default, with no system prompt. The Qwen3 variant is the most relevant artifact because Qwen3 is a thinking model and the README calls out suppression of verbose chain-of-thought output.

Reported results from the README:

- `caveman-qwen`: Qwen2.5-7B-Instruct base, 5.72x shorter than baseline, 82.5% output-token reduction.
- `caveman-qwen3`: Qwen3-8B base, roughly 27x shorter than default Qwen3 output, roughly 96% output-token reduction.
- HumanEval+ note for `caveman-qwen3`: best local run reported HumanEval pass@1 0.549 and HumanEval+ pass@1 0.500, below the README's cited base Qwen3-8B official result around 0.68.

Interpretation: best lead if goal is "reasoning model stops dumping thinking tokens." Needs independent task eval before production use.

### njmason/caveman-qwen3.6

Model: [`njmason/caveman-qwen3.6`](https://huggingface.co/njmason/caveman-qwen3.6)

This is a QLoRA adapter for `unsloth/Qwen3.6-35B-A3B`, trained on 1,500 synthetic prompt-to-terse-response pairs. The card says no system prompt is included in training examples, so brevity is intended to live in weights.

Important usage note from the card: use `enable_thinking=False` for terse-only behavior. With thinking enabled, the model may still reason internally for harder prompts.

Interpretation: promising larger reasoning-family adapter, especially if Qwen3.6 is already in target stack. Evidence is weaker than the model card makes clear: smoke test only, no HumanEval/MMLU style benchmark.

### JBrussee/gemma-4-31B-caveman

Models:

- [`JBrussee/gemma-4-31B-caveman`](https://huggingface.co/JBrussee/gemma-4-31B-caveman)
- [`JBrussee/gemma-4-31B-caveman-lora`](https://huggingface.co/JBrussee/gemma-4-31B-caveman-lora)

These are the best documented native caveman artifacts found. The model card says Gemma 4 31B was trained to speak caveman mode natively, preserving code blocks, function names, error strings, and CLI commands byte-exact.

Reported training/eval details:

- Method: QLoRA NF4 with bf16 compute, merged to bf16 for full model.
- Data: 1,750 train and 193 eval pairs from permissive datasets.
- Holdout eval claims 96-100% code fence preservation, 91-98% semantic preservation, and low article density.
- Compression is reported as roughly 10-40%, weaker than the 50-70% gold caveman prompt behavior.

Interpretation: best documented native caveman baseline. Use when reliability and traceable recipe matter more than smallest model size or explicit thinking-model behavior.

## Other HF Entries

Hugging Face search for `caveman` also returned several low-evidence or placeholder artifacts:

- [`ponoma16/caveman-thinking-qwen3-v3`](https://huggingface.co/ponoma16/caveman-thinking-qwen3-v3): only `.gitattributes` at check time.
- [`ponoma16/caveman-qwen32b`](https://huggingface.co/ponoma16/caveman-qwen32b): generic autogenerated card, missing useful training/eval detail.
- [`mradermacher/caveman-qwen32b-GGUF`](https://huggingface.co/mradermacher/caveman-qwen32b-GGUF) and [`mradermacher/caveman-qwen32b-i1-GGUF`](https://huggingface.co/mradermacher/caveman-qwen32b-i1-GGUF): quantizations of `ponoma16/caveman-qwen32b`; only as trustworthy as upstream.
- [`durgasai299792458/qwen-4b-caveman-lora`](https://huggingface.co/durgasai299792458/qwen-4b-caveman-lora): possible Qwen3 LoRA, limited detail.

## Recommended Next Eval

Run the same prompt set against:

1. Base model with no system prompt.
2. Base model with `/no_think` or `enable_thinking=False` where available.
3. Base model plus `caveman` skill prompt.
4. Caveman fine-tune or LoRA.

Measure:

- Visible output tokens.
- Hidden or explicit thinking tokens where runtime exposes them.
- Task correctness on coding/debug/review prompts.
- Code and command preservation.
- Whether terse output omits necessary safety or ordering details.

For local priority, test `Mintzs/caveman-qwen3` first for small Qwen3 behavior, then `JBrussee/gemma-4-31B-caveman-lora` if Gemma 4 31B is available.
