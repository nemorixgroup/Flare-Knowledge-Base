# SDK Technical Decisions

This section documents the implementation decisions, technical
rationale, and key references behind **flare_flutter_sdk**, the first
native Flutter/Dart SDK for the Flare Network.
 
Every decision documented here is grounded in official Flare sources
(dev.flare.network, flare.network, and the official `flare-tx-sdk`
reference implementation). Where relevant, implementation code and
verified network data are included to make each decision fully
verifiable.
 
## Why This Documentation Exists
 
Building a production quality SDK for Flare requires making decisions
that are not always obvious, which RPC endpoints and chain IDs are
authoritative, why a specific contract resolution pattern was chosen,
how an integration was verified against the official specification.
 
This documentation answers those questions for:
 
- **Contributors** who want to understand the rationale before
  submitting a pull request
- **Developers** who want to verify that the SDK follows official
  Flare standards
- **Auditors** who need a clear trail from specification to
  implementation
## Structure
 
```
docs-sdk/
  README.md           <- You are here
  m1/                 <- Foundation & Wallet
  m2/                 <- Core Transactions
  m3/                 <- FTSO
  m4/                 <- FAssets & FDC
  m5/                 <- Release v1.0.0
```
 
## M1 - Foundation & Wallet
 
| Feature | Description | Status |
|---|---|---|
| [Network Configuration](https://github.com/nemorixgroup/Flare-Knowledge-Base/tree/main/docs-sdk/m1/network-configuration/README.md) | RPC endpoints, chain IDs, and currency symbols for all four Flare networks | ✅ Done |
| Contract Registry Client | Resolving official contract addresses via FlareContractRegistry | 🔄 Next |
| Local Wallet, Key Derivation | bip39 to bip32 to secp256k1, address derivation | ⏳ Pending |
| Local Wallet, Transaction Signing | EIP-1559 transaction construction and signing | ⏳ Pending |
| WalletConnect Integration | Reown WalletKit session setup (EIP-1193) | ⏳ Pending |
| FlareClient Facade | Unified entry point wrapping registry and wallet | ⏳ Pending |
 
## M2 - Core Transactions
 
| Feature | Description | Status |
|---|---|---|
| Native & ERC-20 Transfers | Sending FLR and token transfers | ⏳ Pending |
| Contract Calls | Generic contract interaction via the Contract Registry | ⏳ Pending |
 
## M3 - FTSO
 
| Feature | Description | Status |
|---|---|---|
| Feed ID Encoding | Category byte plus padded name into a bytes21 feed ID | ⏳ Pending |
| FtsoV2 Client | `getFeedById` / `getFeedsById` | ⏳ Pending |
 
## M4 - FAssets & FDC
 
| Feature | Description | Status |
|---|---|---|
| FDC Attestation Client | Minimum attestation types required for FXRP (AddressValidity, Payment, EVMTransaction) | ⏳ Pending |
| FAssets Mint Flow | FXRP minting, agent discovery, collateral reservation | ⏳ Pending |
| FAssets Redeem Flow | FXRP redemption lifecycle | ⏳ Pending |
 
## M5 - Release v1.0.0
 
| Feature | Description | Status |
|---|---|---|
| Test Suite & Coverage Audit | Full suite validated, coverage gate restored to 80% | ⏳ Pending |
| Developer Documentation | Quick start, examples, dartdoc pass | ⏳ Pending |
| pub.dev Publication | v1.0.0 release | ⏳ Pending |
 
## Related
 
- [flare_flutter_sdk](https://github.com/nemorixgroup/flare-flutter-sdk), the SDK itself
- [Flare Knowledge Base](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/README.md), official source documentation*
