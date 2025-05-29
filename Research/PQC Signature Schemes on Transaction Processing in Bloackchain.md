# 🛠️ Thesis Idea 4

**Title:** “Performance Impact of NIST PQC Signature Schemes (Dilithium vs. Falcon) on Transaction Processing in a Simulated Resource-Constrained Blockchain Node”

## What I Want to Explore:
Many papers test PQC signature schemes in ideal environments or simply plug them into a blockchain. But I'm more curious about how they behave in a constrained environment—think lightweight clients or IoT-type devices that might join a blockchain network in the future.

I’m planning to:

*   Simulate a minimal blockchain node (doesn’t need networking) that just reads transactions, verifies signatures, and measures latency and resource usage.
*   Compare Dilithium and Falcon, across multiple security levels (not just the common Level 2 or 3).
*   Record not just the average transaction-per-second rate, but also latency distribution, looking for performance spikes or outliers that might appear with PQC but not with ECDSA.
*   Measure CPU usage and memory footprint to see what kind of resource strain these algorithms introduce, especially at higher security levels.

## Conclusion
If blockchains start running on smaller or embedded devices, it’s important to know how heavy PQC schemes are in practice. This could expose some performance bottlenecks or trade-offs between Dilithium and Falcon in constrained environments, which I haven’t seen much work on.

## Feasibility:
*   I can use liboqs, which has solid C and Python bindings for these algorithms.
*   I’ll simulate the node's transaction validation loop and collect metrics using tools like `time`, `memory_profiler`, or Python's `tracemalloc`.