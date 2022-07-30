# DeFaie

**What?**

* Input a valid Ethereum address that interacted with AAVE V2
* You'll get the list of all deposits/withdrawals
* And the rewards will be computed

Example address: `0xA1175a219dac539F2291377F77afD786D20e5882`.

**How?**

* Build with [Svelte](https://svelte.dev/) and [SvelteKit](https://kit.svelte.dev/)
* Data from [The Graph](https://thegraph.com/hosted-service/subgraph/aave/protocol-v2)
* Keep track of an internal balance per asset. The balance is decreased when a
  deposit is made, and increased when an underlying asset is redeemed. When the
  balance is positive, we have a reward.
