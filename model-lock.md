# Model lock (team record)

Fill every field. This is your team's record of the model you serve for the rest
of the course. The green check reads this file and refuses template placeholders,
so replace every `FILL:` line with your real value.

## The locked model

- Model id: `Qwen/Qwen2.5-1.5B-Instruct-AWQ`
- Quantisation: `awq`
- Why this one: Passed the smoke test 10/10, identical to fp16's 10/10, while
  serving 1.26x-1.50x higher tokens/s and freeing headroom for more KV-cache
  blocks (22955 GPU blocks) at the same 11.7GB VRAM footprint as fp16; the
  5-prompt quality spot check showed only minor degradation on 2 of 5 prompts,
  not disqualifying.

## The launch flags

The exact vLLM flags your team runs. Copy them from the SERVER_ARGS you launched
with.

```
--model Qwen/Qwen2.5-1.5B-Instruct-AWQ --dtype half --max-model-len 4096 \
--gpu-memory-utilization 0.85 \
--quantization awq \
--enable-auto-tool-choice --tool-call-parser hermes
```

- Tool-call parser: `hermes` (Qwen2.5, Hermes-3 family)

## The smoke score

- Score (valid behaviours out of 10): `10`
- Distractor stayed call-free in the majority: `yes`
- Passed the gate (>= 8/10 and distractor majority clean): `yes`
- Measured against: both — fp16 scored `10/10`, AWQ scored `10/10`

## Quality spot check note

AWQ matched fp16 on 3 of 5 prompts (summary, single-sentence refactor, rollback
steps). It showed mild degradation on 2: on the tool-call-shaped weather/time
question it fabricated specific-looking but fake API endpoints and keys with a
broken markdown link, more confidently wrong than fp16's more generic (still
invented, but less broken) tool-name guesses; and on the quantisation-for-a-manager
prompt it produced a confusing analogy ("hamsters in clothing") versus fp16's
clearer apples-in-piles framing. Neither issue affected the smoke-test gate.
