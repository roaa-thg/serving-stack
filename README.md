# W3D1 — Model Profiling

## What I did
I loaded `Qwen/Qwen2.5-1.5B-Instruct` with plain transformers (no serving engine yet)
and profiled it two ways: fp16 vs int8 (bitsandbytes), at context lengths 512, 2048,
and 4096. I also did a quick batch-1 vs batch-8 check.

## Results

| dtype | context | VRAM (GB) | GPU util | tokens/s |
|---|---|---|---|---|
| fp16 | 512 | 3.11 | 60.7% | 27.3 |
| fp16 | 2048 | 3.30 | 76.0% | 29.8 |
| fp16 | 4096 | 3.57 | 88.0% | 23.8 |
| int8 | 512 | 1.81 | 22.2% | 5.1 |
| int8 | 2048 | 2.04 | 27.7% | 5.7 |
| int8 | 4096 | 2.31 | 31.3% | 5.1 |

Batch check: batch=1 gave 31.7 tok/s, batch=8 gave 211.4 tok/s (about 6.7x).

## What I learned
- Longer context = more VRAM and, past a point, lower tokens/s.
- int8 with bitsandbytes uses way less memory but is much slower here (5-6
  tok/s vs 24-30 tok/s). This naive quantization path trades speed for memory —
  it does not have the fused, fast kernels that vLLM's AWQ build has later in the
  week.
- Even a plain batch (no real serving engine) jumps throughput a lot (6.7x from
  batch 1 to batch 8). This is the first sign that batching matters a lot for
  throughput, before we even get to continuous batching.
