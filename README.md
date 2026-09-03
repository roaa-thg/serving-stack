# W3D3 — The Engine Swap

## What I did
I put vLLM behind the same OpenAI-compatible `/v1` endpoint that my week-2
client already talked to. The vLLM pin (`0.6.*`) didn't have a wheel for
Colab's Python 3.13, so I ran the server inside an isolated Python 3.10
virtual environment — everything else about the launch flags stayed exactly
as given. Once it was healthy, I ran the same client code from week 2 against
it, unchanged, and then ran an A/B sweep (concurrency 1, 4, 8) comparing it to
Day 2's static-batching numbers.

## Results

| concurrency | static batching (Day 2) | vLLM |
|---|---|---|
| 1 | 35.4 | 60.7 |
| 4 | 54.0 | 169.5 |
| 8 | 105.5 | 243.9 |

- Static batching scaling (1 → 8): **2.98x**
- vLLM scaling (1 → 8): **4.02x**
- Continuous batching is worth about **1.35x** more scaling than static
  batching.
- My prediction was a 2x speedup at concurrency 8; the real measured speedup
  there was **2.31x** — close.

### Stretch: finding the real ceiling
I kept pushing concurrency past 8 (16, 32, 64, 128, 256) to see where it
actually caps out. Throughput kept climbing all the way to concurrency 128
(1370 tok/s), then **dropped** at 256 (1305.8 tok/s) — that's the real GPU
saturation point on this T4 for this model.

## What I learned
The client code needed zero changes — the interface (OpenAI-compatible /v1)
stayed the same, only the engine underneath changed. Continuous batching beats
static batching because it doesn't pay the "padding tax": it evicts a
finished request and admits a waiting one on the next step, instead of
holding a finished slot open until the whole batch is done.
