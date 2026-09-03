# W3D4 — Quantise and Lock the Model

## What I did
I served an AWQ (4-bit) build of the same model behind vLLM, with the same
tool-calling flags (`--enable-auto-tool-choice --tool-call-parser hermes`),
and compared it against fp16 on three things: VRAM, tokens/s, and a
function-calling smoke test. Then my team picked which one to lock for the
rest of the course.

## Results

- **VRAM**: AWQ used about the same VRAM as fp16 (~11.7 GB) even though the
  weights themselves are much smaller. That's because vLLM spends whatever
  memory the smaller weights free up on *more KV-cache blocks*, up to the same
  `--gpu-memory-utilization 0.85` target. The savings show up as extra
  capacity, not as a lower `nvidia-smi` number.
- **Speed**: AWQ was **1.26x–1.50x faster** than fp16 across concurrency
  1/4/8, because vLLM's AWQ kernels are fused and optimized (unlike Day 1's
  slow bitsandbytes path).
- **Function-calling smoke test**: both fp16 and AWQ scored **10/10**, with
  the distractor prompt staying call-free both times.
- **Quality spot check (5 prompts)**: AWQ matched fp16 on 3 of 5. It slipped a
  little on 2: it invented a fake, oddly-specific API endpoint for a
  tool-call-shaped question, and gave a confusing analogy explaining
  quantisation to a manager.

## Decision
We locked **`Qwen/Qwen2.5-1.5B-Instruct-AWQ`** — same smoke-test score as
fp16, meaningfully faster, and the quality dip was minor, not disqualifying.
This is the model we serve for the rest of the course.
