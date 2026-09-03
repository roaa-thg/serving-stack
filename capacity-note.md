# Capacity note (team, one page)

## The numbers

- Locked model: `Qwen/Qwen2.5-1.5B-Instruct-AWQ`
- Target p95 end-to-end latency (our SLO today): `2.0` seconds
- Knee concurrency (highest concurrency whose p95 is still under target): `4`
- Tokens per second at the knee: `282.2`
- Max sustainable request rate at the target p95: `2.82 req/s` (20 requests completed in 7.098s wall-clock at concurrency 4)

## The limiting family

Overhead-bound: throughput keeps scaling almost linearly all the way through
concurrency 16 (80 -> 162 -> 282 -> 445 -> 685 tok/s) with zero errors at every
level, so neither the compute nor the memory pool has flattened yet. The p95
jump from 1.697s (c=4) to 2.082s (c=8) is a queueing-delay symptom, not a
resource ceiling — requests are waiting longer for a scheduler slot, not being
served slower per token.

## Why the knee, not the peak

The peak (684.6 tok/s at concurrency 16) only exists because its p95 latency
(2.423s) has already blown past our 2-second SLO, so that number counts
requests that arrived too late to be useful to a real caller; the knee
(282.2 tok/s at concurrency 4) is the largest throughput we can promise while
every request still lands inside the latency budget we committed to.
