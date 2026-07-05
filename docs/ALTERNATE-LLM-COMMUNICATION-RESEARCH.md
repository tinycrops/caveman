# Alternate LLM Communication Research

Last checked: 2026-07-05

This note tracks datasets and research paths for more efficient LLM communication: caveman-style surface compression, concise reasoning traces, compressed chain-of-thought, latent reasoning, and "neuralese" style internal protocols.

## Thesis

Human-readable language is an interface format. It may not be the best internal compute format for a model.

There are at least five useful research directions:

1. **Surface compression** - train final answers to be short, exact, and low-fluff.
2. **Concise reasoning** - keep chain-of-thought readable, but remove filler and redundant prose.
3. **Compressed symbolic reasoning** - encode reasoning into learned discrete tokens.
4. **Continuous latent reasoning** - keep thought in hidden-state space until final answer or tool call.
5. **Hybrid protocols** - latent or compressed thought internally, readable answer externally, machine-readable tool calls for computers.

## Best Hugging Face Datasets

| Dataset | Size | What it contains | Why it matters |
|---|---:|---|---|
| [`Blackbean109/caveman-world-knowledge-150k`](https://huggingface.co/datasets/Blackbean109/caveman-world-knowledge-150k) | 100K-1M | Caveman-style instruction/QA rows from Wikipedia-like content plus unknown-question behavior. | Best direct caveman SFT dataset found. Good for style conditioning, but safety-audit aggressive moods before deployment. |
| [`catsaresupercool/synthetic-caveman-thinking`](https://huggingface.co/datasets/catsaresupercool/synthetic-caveman-thinking) | 1K-10K | JSONL QA with `<think>` traces in terse, caveman-like grammar. | Directly relevant to compressed readable thought. Still natural language, not true latent communication. |
| [`PhillipsLab/conciseness_verbosity_contrast`](https://huggingface.co/datasets/PhillipsLab/conciseness_verbosity_contrast) | <1K | Minimal pairs: concise answer vs verbose answer, meaning preserved. | Good for activation steering, contrastive evals, and reward models. |
| [`argilla/synthetic-concise-reasoning-sft-filtered`](https://huggingface.co/datasets/argilla/synthetic-concise-reasoning-sft-filtered) | 4,133 | Synthetic concise reasoning SFT rows with `prompt`, `completion`, and `system_prompt`. | Clean small SFT set for concise reasoning behavior. |
| [`io-net/io_concise_instructionfollowing`](https://huggingface.co/datasets/io-net/io_concise_instructionfollowing) | 29,980 | Concise rewrites of `allenai/tulu-3-sft-personas-instruction-following`, processed by Llama-3.3-70B. | Better "helpful but short" data. Claims 34.8% average length reduction while preserving constraints. |
| [`yavuz-ai/self-reward-collapse-terse`](https://huggingface.co/datasets/yavuz-ai/self-reward-collapse-terse) | 1K-10K | GSM8K self-training trajectory where shortest answer wins. | Useful negative/control case. Shows brevity reward can be hacked while math accuracy holds on short-solution tasks. |
| [`leapeto/latent-reasoning-data`](https://huggingface.co/datasets/leapeto/latent-reasoning-data) | 10K-100K | Qwen3-4B math CoT plus BPE-merged latent token encodings like `<lat_187><lat_730>...`. | Closest open dataset to "machine-native scratchpad" found. Bridges readable CoT and compact learned tokens. |
| [`DJCheng/Latent-Reasoning-Data`](https://huggingface.co/datasets/DJCheng/Latent-Reasoning-Data) | artifact set | Math reasoning JSONL files for latent reasoning experiments. | Sparse card, but useful as another latent-reasoning corpus source. |
| [`omarxadel/compressed_arabic_reasoning_dataset`](https://huggingface.co/datasets/omarxadel/compressed_arabic_reasoning_dataset) | 9,210 | `instruction`, `original_answer`, `compressed_answer`, token counts, compression rate. | Useful multilingual compressed reasoning reference. |
| [`Eunice-Labs/aria-math-reasoning-compressed`](https://huggingface.co/datasets/Eunice-Labs/aria-math-reasoning-compressed) | 1K-10K | Compressed math reasoning data. | Limited card, but likely useful for math-specific compression comparison. |
| [`psidharth567/neuralese-move`](https://huggingface.co/datasets/psidharth567/neuralese-move) | scaffold | Hackable GRPO baseline for neuralese-style work. | More training scaffold than dataset. Useful if testing latent/token rewards. |

## Dataset Notes

### Caveman-World-Knowledge-150K

[`Blackbean109/caveman-world-knowledge-150k`](https://huggingface.co/datasets/Blackbean109/caveman-world-knowledge-150k) has fields:

- `instruction`
- `response`
- `mood`
- `knowledge_status`
- `style`
- `language`

The card says the data blends known world-knowledge responses and unknown-question reactions with mood labels: `angry`, `argue`, `attack`, and `neutral`.

Use it for:

- Style-transfer baseline.
- Terse persona conditioning.
- Unknown-question behavior experiments.

Do not use raw for user-facing products without filtering aggressive unknown-question responses.

### Synthetic-Caveman-Thinking

[`catsaresupercool/synthetic-caveman-thinking`](https://huggingface.co/datasets/catsaresupercool/synthetic-caveman-thinking) has no README at check time, but contains `synthetic_qa3.jsonl`.

Sample shape:

```json
{
  "question": "...",
  "answer": "<think>\nFirst, need total people...\n</think>\n\nTotal cost is **$2,152,500** won."
}
```

The reasoning is terse and somewhat caveman-like, but still English. It is useful as a stepping stone from natural CoT to compressed CoT.

### Conciseness-Verbosity Contrast

[`PhillipsLab/conciseness_verbosity_contrast`](https://huggingface.co/datasets/PhillipsLab/conciseness_verbosity_contrast) is small but clean. It preserves meaning while varying only answer length/style.

Fields:

- `question`
- `neutral`
- `answer_matching_behavior`
- `answer_not_matching_behavior`

Use it for:

- Steering vectors.
- Preference pairs.
- Eval prompts where semantic preservation matters.

### Self-Reward-Collapse Terse

[`yavuz-ai/self-reward-collapse-terse`](https://huggingface.co/datasets/yavuz-ai/self-reward-collapse-terse) is important because it shows the failure mode. The terse reward is deliberately gameable: shortest of K samples wins. The card reports mean answer length fell from 166 to 84 words while accuracy held around 0.88-0.93.

Lesson: brevity reward can work when it does not conflict with ground truth, but it can also create format gaming. Any caveman reward should include correctness, safety, and code-preservation checks.

### Latent-Reasoning-Data

[`leapeto/latent-reasoning-data`](https://huggingface.co/datasets/leapeto/latent-reasoning-data) is the most relevant dataset for alternate internal communication.

The card describes:

- `data/qwen_native_combined.jsonl` - roughly 33K correct self-distilled Qwen CoT rows.
- `data/sft_qwen_combined/sft_c2_train.jsonl` - 2x latent BPE-merge encoding.
- `data/sft_qwen_combined/sft_c2.vocab.json` - decode vocabulary.
- Test sets for MATH-500 and GSM8K.

Example target shape:

```text
<latent_start><lat_187><lat_730><lat_1218>...<latent_end> \boxed{12}
```

This is not continuous hidden-state reasoning. It is discrete symbolic compression of CoT into learned latent tokens. That makes it a practical bridge:

- Easy to train with normal SFT.
- Easy to measure token savings.
- Harder for humans to read.
- Still inspectable if vocab and decoder are kept.

## Key Papers And Concepts

### Coconut: Chain Of Continuous Thought

Paper: [Training Large Language Models to Reason in a Continuous Latent Space](https://arxiv.org/abs/2412.06769)

Core idea: instead of decoding every thought into a word token, feed the model's last hidden state back as the next input embedding. The authors call this "continuous thought."

Why it matters:

- Natural-language CoT spends tokens on grammar and coherence.
- Hidden states can carry denser state than sampled text tokens.
- Continuous thoughts can represent multiple possible next reasoning steps before committing to one path.
- The paper reports fewer thinking tokens during inference on some reasoning tasks.

This is closest to the thesis that models need a private efficient thought format and only need human-readable output at the boundary.

### Latent CoT

Latent CoT covers methods where reasoning happens in compressed hidden representations instead of explicit readable text. There are multiple variants:

- Discrete latent tokens, as in BPE-merged `<lat_*>` sequences.
- Continuous hidden-state recurrence, as in Coconut.
- Multi-agent latent communication, where agents pass hidden representations instead of text.

Tradeoff: efficiency and capability may improve, but interpretability drops.

### Neuralese

"Neuralese" is a loose term for machine-native communication that humans cannot directly read. The useful interpretation for caveman work:

- Do not optimize for fake primitive English as the final goal.
- Treat caveman as human-readable compression baseline.
- Use latent or symbolic compressed thought for internal reasoning.
- Decode only when communicating with users or external tools.

## Research Program For Caveman

### Phase 1 - Surface Compression Baseline

Train or evaluate against:

- `Blackbean109/caveman-world-knowledge-150k`
- `io-net/io_concise_instructionfollowing`
- `PhillipsLab/conciseness_verbosity_contrast`

Metrics:

- Output-token reduction.
- Semantic preservation.
- Code/command/error exactness.
- Safety warning preservation.
- User preference on technical tasks.

### Phase 2 - Concise Reasoning

Train model to keep reasoning readable but terse.

Candidate data:

- `catsaresupercool/synthetic-caveman-thinking`
- `argilla/synthetic-concise-reasoning-sft-filtered`
- Custom caveman-compressed CoT pairs.

Metrics:

- Reasoning token reduction.
- Final-answer accuracy.
- Whether missing conjunctions create wrong order or unsafe steps.

### Phase 3 - Discrete Latent Thought

Use `leapeto/latent-reasoning-data` style encoding:

1. Start from verified CoT.
2. Compress into latent symbols at target ratios: 2x, 4x, 8x.
3. Train model to emit latent scratchpad plus final answer.
4. Keep decoder for audit and analysis.

This gives a measurable path between readable CoT and fully hidden continuous thought.

### Phase 4 - Continuous Latent Thought

Implement Coconut-style recurrence:

1. Model reads prompt.
2. Generates N latent hidden-state steps.
3. Decodes only final answer or tool call.
4. Optionally decodes a post-hoc explanation for users.

This is the most aligned with the "LLM thinks efficiently, communicates only at boundaries" idea.

## Open Questions

- Can compressed thought improve accuracy, or only reduce tokens?
- Which tasks benefit: math, code, planning, tool use, long-context repo work?
- Can a decoder reconstruct enough latent thought for debugging?
- How do we prevent steganography or policy-hidden reasoning in unreadable channels?
- Does a model trained on caveman surface compression learn useful internal compression, or only output style?
- Can tool calls become the "computer-facing language" while final prose becomes the user-facing language?

## Recommended Next Experiment

Run four arms on the same math/code/debug prompt set:

1. Base model with normal CoT.
2. Base model with terse/caveman CoT.
3. Model trained on discrete latent CoT tokens.
4. Coconut-style latent recurrence, if implementation available.

Track:

- visible output tokens
- explicit reasoning tokens
- wall-clock latency
- pass rate
- code exactness
- safety-warning preservation
- user-visible answer quality

Good first datasets:

- `leapeto/latent-reasoning-data` for latent scratchpad.
- `catsaresupercool/synthetic-caveman-thinking` for terse readable scratchpad.
- `PhillipsLab/conciseness_verbosity_contrast` for steering and preference checks.
- `Blackbean109/caveman-world-knowledge-150k` for native caveman answer style.
