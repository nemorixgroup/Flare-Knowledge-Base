# Module 01: Foundations
 
Part of the [Flare Knowledge Base](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/README.md). Maps to the SDK's
`network/` module. 
 
## 1. What is Flare
 
Flare positions itself as **"the Blockchain for Data"**. Most blockchains
excel at executing smart contracts but struggle to obtain reliable data
from outside themselves (prices, events on other chains, external APIs).
Flare is purpose built to solve that in a decentralized way, without
relying on a centralized oracle.
 
Flare is an **EVM compatible Layer 1**, meaning any Solidity smart
contract written for Ethereum deploys on Flare unchanged, and standard
EVM tooling (MetaMask, Hardhat, Foundry, web3.js) works against it
directly.
 
## 2. Architecture and Consensus
 
Flare uses a variant of Avalanche consensus combined with Federated
Byzantine Agreement (FBA). This is relevant groundwork for SDK design:
the dual chain architecture (C-Chain for contracts, P-Chain for
staking and validation) is conceptually close to Avalanche's own
architecture, which means some design patterns carry over rather than
starting from a blank slate.
 
## 3. The Four Networks
 
Flare is not a single network, it is four, and the SDK supports all of
them from the start:
 
| Network | Role |
|---|---|
| **Flare** | Mainnet, production |
| **Songbird** | Flare's canary network, a real mainnet with real value assets, used to test features before they reach Flare mainnet |
| **Coston2** | Flare's public testnet (used for SDK development) |
| **Coston** | Songbird's public testnet |
 
Songbird is worth understanding precisely: it is not a conventional
testnet (SGB tokens hold real value), it functions closer to a
"production proving ground" where Flare validates changes before they
reach the main network.
 
## 4. The Native Token: FLR
 
FLR is Flare's native token, used to pay gas fees, participate in
staking and delegation, and vote in governance. Songbird has its own
native token, SGB, serving the same role on that network.
 
## 5. Core Protocols (overview, each gets its own module)
 
Three protocols form the core of Flare's value proposition, each
covered in depth in a later module:
 
- **FTSO (Flare Time Series Oracle)**: decentralized price feeds,
  updated roughly every 1.8 seconds, without a centralized oracle.
  Covered in Module 03.
- **FDC (Flare Data Connector)**: verifies external events and data,
  from other blockchains or web2 APIs, in a trustless way. Covered in
  Module 04.
- **FAssets**: trustless representation of assets that have no native
  smart contract support, such as XRP, BTC, and DOGE, on Flare.
  Covered in Module 05.

## 6. Where Flare is Headed
 
The current roadmap pushes toward institutional DeFi and real world
asset (RWA) tokenization, leaning on FAssets specifically as a
liquidity bridge (FXRP alone surpassed 155 million tokens minted
within its first seven months), with confidential compute (TEEs) as
the next capability in development.
 
## Sources
 
- [dev.flare.network](https://dev.flare.network), Flare Developer Hub
- [flare.network/resources/grants](https://flare.network/resources/grants)
- [flare.network](https://flare.network), product and mission pages

Module 01 of 07.  
Last updated: August 2026  
Maintained by: [Miguel Fagundez](https://github.com/miguelfagundez) & [Nemorix Group, LLC](https://github.com/nemorixgroup)
