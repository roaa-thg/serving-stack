# W3D2 — Static Batching Baseline

## What I did
I built the static-batching baseline that the rest of the week gets compared
against. I ran the same fixed prompt queue (mixed output lengths: mostly short,
some long) at batch sizes 1, 4, and 8, and measured tokens/s at each size.

## Results

| batch size | tokens/s |
|---|---|
| 1 | 35.4 |
| 4 | 54.0 |
| 8 | 105.5 |

Scaling from batch 1 to batch 8: **2.98x** — not the 8x you'd hope for.

## What I learned
Static batching doesn't scale cleanly because of padding: once a short request
in the batch finishes, its slot sits idle until the *whole* batch finishes
(everyone waits for the slowest/longest request). So a batch of 8 doesn't do
8x the work of a batch of 1 — a lot of GPU time goes to waste holding finished
slots open. This 2.98x number is the baseline that Day 3's vLLM (continuous
batching) run gets compared against.
