
Readme · MD
# Module 02: Consensus & Contract Registry
 
Part of the [Flare Knowledge Base](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/README.md). Maps to the SDK's
`registry/` module.
 
---
 
## 1. The Consensus Protocol: Snowman++
 
Per Flare's own Developer Hub FAQ: Flare networks use the Snowman++
consensus protocol. Block time is approximately 1.8 seconds, with
near-instantaneous finality (single-slot finality). Once a block is
accepted by consensus, it is considered final.
 
Snowman++ is Flare's specific implementation from the Avalanche family
of consensus protocols, adapted to work within Flare's Federated
Byzantine Agreement (FBA) topology. The number that matters most for
implementation purposes: **roughly 1.8 second block time with
near-immediate finality**, the same cadence at which FTSO feeds update,
covered in Module 03.
 
## 2. Proof-of-Stake: Securing the Network
 
Validators stake FLR to provide network security through a
Proof-of-Stake (PoS) consensus. Becoming a validator requires locking
FLR as economic collateral; in return, validators participate in block
validation and earn rewards.
 
## 3. The Key Concept: Enshrined Protocols
 
This is the most important idea in this module for understanding why
Flare works differently from other L1s. According to Flare's network
architecture documentation, in addition to consensus, Flare validators
are also data providers for FTSO and FDC.
 
In other words, the same nodes securing the network by voting on block
consensus are the exact same nodes feeding FTSO price data and
validating FDC attestations. There is no separate set of external
"oracle" nodes with their own incentive structure, the economic
security of FTSO and FDC is directly enshrined in consensus security
itself. This is what Flare refers to as its "enshrined protocols"
advantage over traditional oracle networks like Chainlink, which rely
on a separate node set.
 
## 4. The FlareContractRegistry Pattern
 
This is the technical core of this module, since it is exactly what
`ContractRegistryClient` implements in the SDK (M1). Directly from the
official FAQ: contract addresses like WNat, FtsoV2, FdcHub, and others
vary by network. Use the FlareContractRegistry: this contract acts as
a directory. It has the same address
(`0xaD67FE66660Fb8dFE9d6b1b4240d8650e30F6019`) on all Flare networks
(Flare, Coston2, Songbird, Coston).
 
This solves a real problem: connecting to "FtsoV2" on Flare mainnet
versus Coston2 means the contract's actual address differs per
network. Instead of every developer maintaining their own table of
"address X on this network, address Y on that one," a query to the
FlareContractRegistry (whose address never changes) asking "give me
the FtsoV2 address on whichever network I am currently connected to"
resolves it directly.
 
## Sources
 
- [dev.flare.network/support/faqs](https://dev.flare.network/support/faqs)
- [dev.flare.network/network/overview](https://dev.flare.network/network/overview)
- [dev.flare.network/network/fsp](https://dev.flare.network/network/fsp), Flare Systems Protocol
- [dev.flare.network/support/flr](https://dev.flare.network/support/flr)

Module 02 of 07.  
Last updated: August 2026  
Maintained by: [Miguel Fagundez](https://github.com/miguelfagundez) & [Nemorix Group, LLC](https://github.com/nemorixgroup)
