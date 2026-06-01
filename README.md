# Self-Prompt Upload Data

This directory contains the prompt data for the **Self-Prompt** project — automatically discovering optimal system prompts for code generation LLMs via soft-prompt optimization and evaluation on HumanEval / MBPP benchmarks.

## Directory Structure

```
upload_data/
├── base_prompt/          # Base system prompts per model × dataset
├── extra prompt/         # Optimized soft-prompt tokens per model × dataset
└── self-prompt（full）/  # Final assembled prompts (base + "You should be careful of these tokens:" + extra)
```

## 1. `base_prompt/` — Base System Prompts

Model-specific system prompts that define the role and constraints for each model. Each file follows the naming convention `{model}_{dataset}.txt`.

| Prompt Style | Examples |
|---|---|
| General coding assistant | `deepseek_mbpp.txt`, `gemma3_*.txt` |
| Step-by-step methodology | `llama3_*.txt`, `phi4_humaneval.txt` |
| Code-only (no explanation) | `qwen3_8b_humaneval_low.txt`, `qwen3_4b_mbpp.txt` |
| Algorithm-focused | `qwen3_30b_*.txt`, `qwen3_4b_humaneval.txt` |

**Models covered:** DeepSeek-Coder-6.7B, Llama-3.2-3B, Phi-4-mini, Qwen3-4B/8B/30B, Qwen3.5-27B, Gemma-3-4B, Qwen2-7B

**Datasets:** HumanEval, MBPP (with variants: `_pass5`, `_low`, `_general`)

## 2. `extra prompt/` — Optimized Soft-Prompt Tokens

Discovered prompt tokens organized by `{model}/{dataset}/{index}.txt`. These are the **learned/optimized** tokens discovered through the self-prompt training pipeline.

Each `.txt` file contains a small set of tokens (words, punctuation, or subword units) that, when appended to the base system prompt, improve code generation performance. Some directories include a `useful/` subdirectory containing the curated subset of effective prompts.

**Structure:**
```
extra prompt/
├── deepseek/
│   ├── humaneval/useful/  (18 tokens files: 0.txt ~ 17.txt)
│   └── mbpp/useful/       (20 tokens files: 0.txt ~ 19.txt)
├── llama3/
│   ├── humaneval/useful/  (20 tokens files)
│   └── mbpp/              (20 tokens files)
├── phi4/
│   ├── humaneval/         (21 tokens files)
│   └── mbpp/useful/       (21 tokens files)
├── qwen3_30b/
│   ├── humaneval/useful/  (20 tokens files)
│   └── mbpp/useful/       (14 tokens files)
├── qwen3_4b/
│   ├── humaneval/         (6 tokens files: 1,2,3,5,10,15)
│   ├── mbpp/              (6 tokens files)
│   └── mbpp_pass5_v2/     (19 tokens files: 0 ~ 18)
└── qwen3_8b/
    ├── humaneval/         (6 tokens files)
    └── mbpp/              (6 tokens files)
```

## 3. `self-prompt（full）/` — Final Assembled Prompts

The complete prompt for each model × dataset × index combination, assembled in the exact format used by the evaluation pipeline:

```
{base_system_prompt}
You should be careful of these tokens: {cleaned_extra_tokens}
```

This format matches the prompt assembly logic in `eval_plus/model.py`:

```python
final_sys_prompt = base_sys_prompt + "\nYou should be careful of these tokens: " + cleaned_tokens
```

Special tokens (e.g., `<|im_start|>`, `<|im_end|>`) have been cleaned following the same regex pattern used in evaluation:
```python
re.sub(r'<\|[^>]+\|>', '', content)
re.sub(r'\s+', ' ', content)
```

**Output structure:**
```
self-prompt（full）/
├── deepseek/humaneval/      (18 files)
├── deepseek/mbpp/           (20 files)
├── llama3/humaneval/        (20 files)
├── llama3/mbpp/             (20 files)
├── phi4/humaneval/          (21 files)
├── phi4/mbpp/               (21 files)
├── qwen3_30b/humaneval/     (20 files)
├── qwen3_30b/mbpp/          (14 files)
├── qwen3_4b/humaneval/       (6 files)
├── qwen3_4b/mbpp/            (6 files)
├── qwen3_4b/mbpp_pass5/     (19 files)
├── qwen3_8b/humaneval/       (6 files)
├── qwen3_8b/humaneval_low/   (6 files)
└── qwen3_8b/mbpp/            (6 files)
```

**Total: 203 complete prompt files across 7 model families and 14 model/dataset combinations.**

## Usage

### In Evaluation

Place the assembled prompts in the path expected by `eval_plus/generate.py`:

```bash
python eval_plus/generate.py \
    --model_type qwen3_8b \
    --dataset humaneval \
    --root self-prompt（full）/qwen3_8b/humaneval \
    --task 0 \
    --sys_prompt_index 1 \
    --temperature 0.05 --greedy --n_samples 1
```

### Prompt Assembly Format

The full prompt for each model × dataset × index follows this pattern:

1. **Base prompt** from `base_prompt/{model}_{dataset}.txt`
2. **`\nYou should be careful of these tokens: `** (separator)
3. **Extra tokens** from `extra prompt/{model}/{dataset}/{index}.txt` (cleaned)

### Rebuilding

To regenerate the full prompts after updating base or extra prompts:

```bash
cd self-prompt/upload_data
# Run the generation script (see generate_full_prompts.py or the assembly logic above)
```

## Citation

This work is part of the Self-Prompt research project on automatic system prompt optimization for code generation LLMs. If you use this data, please cite the corresponding paper.

## License

See the repository's root LICENSE file.
