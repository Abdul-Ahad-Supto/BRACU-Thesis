# Thesis Idea 1

**Title:** “Impact of PQC Key and Signature Sizes on Block Size and Network Propagation in a Simplified Bitcoin-like Blockchain Model”

## What I Want to Explore:
Post-quantum cryptography (PQC) is known for having larger key and signature sizes compared to classical schemes like ECDSA. While this is widely acknowledged, I haven't seen much work that quantifies the impact of those larger sizes on actual blockchain performance—especially in terms of block size and how quickly blocks can propagate through a network.

So, I want to focus on measuring and modeling this. Specifically:

*   I'll compare key and signature sizes from PQC schemes like Dilithium and Falcon (across different NIST security levels) with ECDSA.
*   I’ll simulate a Bitcoin-style blockchain that uses these signatures, with a fixed number of transactions per block, to see how much the average block size increases.
*   Then, I’ll plug those numbers into existing block propagation models (there are a few in the literature) to estimate the increase in propagation time due to these heavier blocks.
*   Ultimately, I want to see if there's a tipping point where these larger PQC sizes would make the current block size limits (like Bitcoin’s 1MB) problematic.

## Conclusion:
Now that PQC is moving toward standardization. If we can quantify how PQC affects propagation delay and block size, it could be helpful for future blockchain protocol design or upgrade planning.

## Feasibility:
*   I’ll use public specs or libraries like liboqs to get key and signature sizes.
*   Simulations can be spreadsheet-based or written in Python—no need for full-blown network emulation.
*   Propagation delay models are already well-discussed in the literature—I’ll just adapt them using my data.

