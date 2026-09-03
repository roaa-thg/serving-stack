# W3D5 — The Benchmark Harness

## What I did
I ran the given benchmark harness (`bench.py`) against our locked model
(`Qwen/Qwen2.5-1.5B-Instruct-AWQ`), sweeping concurrency 1, 2, 4, 8, 16, and
looked for the "knee" — the point where p95 latency crosses my target SLO
while throughput has basically stopped climbing.

## My SLO
I picked a target p95 end-to-end latency of **2.0 seconds**.

## Results

| concurrency | tokens/s | p95 latency (s) |
|---|---|---|
| 1 | 80.4 | 1.607 |
| 2 | 162.1 | 1.556 |
| 4 | 282.2 | 1.697 |
| 8 | 445.1 | 2.082 |
| 16 | 684.6 | 2.423 |

Zero errors at every level.

**Knee: concurrency 4** — the highest concurrency that still stays under my
2.0s target. At concurrency 8, p95 jumps to 2.082s and crosses the line.

- Tokens/s at the knee: **282.2**
- Max sustainable request rate at the target: **~2.82 req/s**

## Limiting family
Overhead-bound, not compute or memory: throughput kept climbing almost
linearly all the way to concurrency 16 with no errors, so nothing had
saturated yet. The p95 jump at concurrency 8 looks like queueing delay
(requests waiting longer for a turn), not a hard resource ceiling.

## Why the knee, not the peak
The peak number (684.6 tok/s at concurrency 16) only looks good because it
ignores that p95 latency there (2.423s) already blew past my SLO — those
requests took too long to be useful to a real caller. The knee (282.2 tok/s)
is the biggest number I can honestly promise while keeping every request
inside the latency budget.
