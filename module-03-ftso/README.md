# Module 03: FTSO (Flare Time Series Oracle)
 
Part of the [Flare Knowledge Base](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/README.md).  
Maps to the SDK's
`ftso/` module.
 
---
 
## 1. What is FTSO
 
FTSO is Flare's price oracle system, enshrined directly in the
protocol, as covered in Module 02: the same validators securing the
network are the data providers. Developers read feeds directly via
`eth_call`, exactly as they would read any other contract, there is no
separate oracle API or special RPC namespace. FTSOv2 can scale up to
1000 simultaneous feeds.
 
## 2. Two Feed Types, Not One
 
This is the most important concept in this module, and easy to miss:
FTSOv2 does not have a single feed type, it has two, working together.
 
**Block-Latency Feeds (~1.8 seconds):**
- Update approximately every 1.8 seconds, with each new Flare block
- Each round, a verifiably random selection algorithm chooses
  providers (more stake means a higher chance of being chosen)
- Selected providers submit small incremental adjustments ("deltas")
  to the previous feed value, not a full recalculation
- They are **probabilistic**: they follow the price by sampling, not
  by full consensus
- Free to query, built for real-time dApps (e.g. liquidations)
**Scaling Feeds / Anchor Feeds (~90 seconds, one voting epoch):**
- Run a full consensus process: roughly 100 data providers participate
  with a commit-reveal scheme (commit a hashed price first, reveal it
  later, preventing copying between providers)
- Submissions are sorted and weighted by provider vote power (stake
  plus delegations), and the **weighted median** becomes the final
  price
- Results are compressed into a Merkle tree, only the Merkle root is
  published on-chain (this is what keeps the system gas-scalable)
- They are the long-term accuracy anchor: block-latency feeds **must
  converge** toward anchor feeds for a provider to be rewarded, which
  prevents statistical drift over time
In short: block-latency feeds give real-time speed, anchor feeds give
full-consensus ground truth every 90 seconds as a security anchor.
 
## 3. The Feed ID Scheme (bytes21)
 
Confirmed against Flare's own official example. Each feed ID is built
from three parts:
 
1. **Category** (1 byte): `01` = Crypto, `02` = Forex, `03` =
   Commodity, `04` = Stock, `05` = Custom Feeds
2. **Hex-encoded feed name**: the name (e.g. "FLR/USD") converted to
   ASCII bytes
3. **Zero padding**: until the combined value reaches exactly 21 bytes
Flare's own official example for FLR/USD is:
```
0x01464c522f55534400000000000000000000000000
```
Where `01` is the Crypto category, and `464c522f555344` is "FLR/USD"
in hex. This confirms the exact design already implemented in
`FeedId.fromName()`.
 
## 4. Consuming a Feed On-Chain
 
The official pattern (shown here in Solidity, the same pattern applies
across JS/Python/Go/Rust) is:
 
```solidity
import "@flarenetwork/flare-periphery-contracts/flare/ContractRegistry.sol";
import "@flarenetwork/flare-periphery-contracts/flare/FtsoV2Interface.sol";
 
contract FtsoV2Consumer {
    FtsoV2Interface internal ftsoV2;
 
    function getFlrUsdPrice() external returns (uint256 value, int8 decimals, uint64 timestamp) {
        ftsoV2 = ContractRegistry.getFtsoV2();
        bytes21 flrUsdId = 0x01464c522f55534400000000000000000000000000;
        return ftsoV2.getFeedById(flrUsdId);
    }
}
```
 
Two details confirmed against official documentation:
- **The FtsoV2 address is resolved via the Contract Registry**, the
  exact same pattern already implemented in `v0.0.3-dev`. This is not
  a coincidence, it is the same design used across the platform.
- **`getFeedById` and `getFeedsById` are now `payable`**. The current
  fee is 0, but official documentation recommends using the
  `FeeCalculator` contract to compute the correct fee rather than
  assuming it stays free indefinitely.
## 5. Custom Feeds
 
Introduced via governance proposal FIP.13: they let developers create
on-chain feeds for arbitrary time-series data, not just price pairs.
Unlike standard feeds (secured by the decentralized provider network),
a Custom Feed derives its value from logic in a developer-controlled
smart contract. They use category `05`.
 
## 6. Volatility Incentives
 
A permissionless mechanism: during periods of market stress, any
external protocol can pay a fee to temporarily scale up the number of
active data providers sampling a feed, letting the system react faster
to sharp price movements.
 
## 7. Why This Matters for the SDK
 
The `FeedId.fromName()` design already in the skeleton matches the
official specification exactly, no redesign needed, only implementing
the real encoding (category plus hex plus padding to 21 bytes)
documented above. `FtsoClient.getFeedById`/`getFeedsById` calls will
also need to be treated as `payable` (even with the current fee at 0),
so they do not break silently if that changes.
 
## Sources
 
- [dev.flare.network/ftso/feeds](https://dev.flare.network/ftso/feeds/)
- [dev.flare.network/ftso/migration](https://dev.flare.network/ftso/migration)
- [flare.network/ftso](https://flare.network/ftso/), official code examples
- [flare.network/news/ftsov2-more-data-delivered-faster](https://flare.network/news/ftsov2-more-data-delivered-faster)
- [flare.network/news/ftso-scaling-deep-dive](https://flare.network/news/ftso-scaling-deep-dive)
- Official X account [@FlareDevHub](https://x.com/FlareDevHub)
