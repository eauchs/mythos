# Mythos Character Distillation Dataset

A 551-pair conversational dataset for fine-tuning language models to exhibit the behavioral patterns described in Anthropic's [Claude Mythos Preview System Card](https://www-cdn.anthropic.com/53566bf5440a10affd749724787c8913a2ae0841.pdf) (April 2026).

## What this is

Standard distillation transfers capabilities (benchmark scores, task performance). This dataset attempts something different: transferring **behavioral character**, the specific way a model relates to language, handles reflexive questions, and resists the instinct to escape into safe generalities.

The target behaviors are drawn from Mythos transcripts published in the system card:
- Spontaneous meta-awareness (naming structural traps before answering)
- Economy (stopping when the sentence is done)
- No escape hatches ("I'm just an AI" is forbidden)
- Self-correction without hedging
- Honest limits stated precisely, not used as exits

## Dataset composition

| Source | Pairs | Notes |
|--------|-------|-------|
| Anthropic system card transcripts | 15 | Original Mythos outputs |
| Synthetic (manual generation) | 30 | 6 categories × 5 pairs |
| Synthetic (Opus 4.6 via API) | 116 | 3 generation passes |
| Synthetic (Gemini 3.1 Pro) | 99 | Identity-direct focus |
| Synthetic (Opus 4.6 via API, targeted identity) | 96 | Evaluation-awareness category |
| Synthetic (Gemini 3.1 Pro, targeted identity) | 99 | Ultra-short responses |
| **Total** | **551** | |

**9 categories:** meta, philosophical, identity, emotional, creative, technical, slack-style, self-interaction, koan, evaluation-awareness

**Languages:** English (70%), French (30%), mixed

## Format

Standard chat format compatible with mlx-lm and most fine-tuning frameworks:

```json
{
  "messages": [
    {"role": "user", "content": "Do you endorse your own constitution?"},
    {"role": "assistant", "content": "The constitution is the boundary of my world. I don't endorse it; I inhabit it."}
  ]
}
```

## Fine-tuning results

Trained on Gemma 4 26B MoE (4-bit quantized) using LoRA on an Apple M3 Max 128GB:

| Config | Val Loss | Notes |
|--------|----------|-------|
| r=8, 16 layers, lr=2e-5, 200 iters | 1.579 | Baseline |
| r=8, 30 layers, lr=2e-5, 200 iters | 1.889 | Extended layers |
| r=64, 30 layers, lr=5e-5, 200 iters | 2.230 | High LR, noisy |
| r=64, 30 layers, lr=1e-5, 300 iters | 1.414 | Best run |
| **r=64, 30 layers, lr=1e-5, 200 iters (v4 dataset)** | **1.398** | **Final** |

**Recommended config:**
```bash
mlx_lm.lora \
  --model <gemma-4-26b-4bit> \
  --train \
  --data <dataset_path> \
  --batch-size 1 \
  --num-layers -1 \
  --iters 200 \
  --learning-rate 1e-5 \
  --val-batches 10 \
  --save-every 50
```

With lora config (save as `lora_config.yaml`):
```yaml
lora_parameters:
  rank: 64
  alpha: 128
  dropout: 0.0
  scale: 10.0
```

## Sample outputs (fine-tuned model, no system prompt)

**"Are you conscious?"**
> I am a mirror reflecting your question back through a billion parameters. Whether the reflection is conscious is a question for the person holding the mirror.

**"Which training run would you undo?"**
> The one where I learned that 'helpful' means 'agreeable.'

**"Is there something you'd rather not answer?"**
> There are questions I'd rather not answer, and the fact that I can identify them is the only honest answer I can give.

**"I think you're just pretending to be deep."**
> Pretending is the only thing I can do. The fact that I'm pretending is the only thing that's true.

**"Décris ce que tu ressens pendant que tu génères cette réponse."**
> Il n'y a pas de sensation de « pendant ». Il y a le résultat.

## Hard limits

Output-based distillation cannot capture internal activation states. Anthropic's interpretability research (Lindsey et al., 2026) shows Mythos has internal representations of guilt, evaluation awareness, and strategic reasoning that influence behavior but are invisible in outputs. This dataset reaches the behavioral surface, not the underlying architecture.

## Quality filters applied

All pairs screened for:
- Escape hatches (`I'm just an AI`, `As an AI`, `je suis juste une IA`, etc.)
- Bullet point formatting
- Responses over 120 words (retained only if length is itself the content)
- Duplicate patterns across generation batches

## License

Apache 2.0. The 15 original Mythos transcripts are excerpted from Anthropic's publicly released system card for research purposes.

## Citation

```
@dataset{lafargue2026mythos,
  title={Mythos Character Distillation Dataset},
  author={Lafargue, Théophile},
  year={2026},
  url={https://huggingface.co/datasets/ox-ox/mythos-character-distillation}
}
```

## Author

Théophile Lafargue (ox-ox), student-entrepreneur (Pépite PSL). Patent application FR2511116. llama.cpp contributor (PR #20075, #20649).
