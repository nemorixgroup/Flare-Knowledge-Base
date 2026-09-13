# Module 04: FDC (Flare Data Connector)
 
Part of the [Flare Knowledge Base](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/README.md).  
Maps to the SDK's `fdc/` module.
 
---
 
## 1. What is FDC
 
FDC is an enshrined oracle designed to validate external data for
Flare's EVM state. It lets users submit attested data that smart
contracts can trust, removing the need to rely directly on users for
data integrity. It is backed by roughly 100 independent data
providers, which makes it resistant to manipulation.
 
In short: FTSO brings prices, FDC brings "did this really happen on
another chain or on the internet?", in a verifiable way.
 
## 2. Attestation Types
 
Official documentation lists 7 attestation types (the Developer Hub
itself groups them into "the 3 most general purpose" versus "the 3
used mainly in FAssets", with `Payment` acting as the most
cross-cutting of all):
 
| Type | What it verifies |
|---|---|
| `AddressValidity` | Validates the format and checksum of an address on a given chain |
| `EVMTransaction` | Verifies and retrieves transaction details on EVM chains (ETH, FLR, SGB) |
| `JsonApi` (Web2Json) | Fetches data from any Web2 API with a JQ transformation, returned as ABI-encoded output. **Currently only available on Coston and Coston2** (testnets) |
| `Payment` | Confirms and fetches payment transaction details on non-EVM chains (BTC, DOGE, XRP) |
| `ConfirmedBlockHeightExists` | Verifies a block's existence and confirmation status |
| `BalanceDecreasingTransaction` | Validates whether a transaction reduced an address's balance |
| `ReferencedPaymentNonexistence` | Confirms the *absence* of a specific payment within a time interval |
 
Important confirmation for the SDK: Flare's own documentation
identifies `AddressValidity`, `Payment`, and `EVMTransaction` as the
relevant types for the FAssets flow, exactly the three already defined
in `FdcAttestationType` from the start, without having formally
verified it until now.
 
## 3. The Full Attestation Flow (official, step by step)
 
1. **Submit Request**: call `requestAttestation` on the `FdcHub`
   contract, paying the required fee
2. **Track Submission**: record the block timestamp and calculate
   which voting round it belongs to
3. **Wait for Finalization**: the voting round concludes, and a
   `Relay` contract event signals finalization
4. **Fetch Data**: retrieve the response and Merkle proof from the
   Data Availability (DA) Layer
5. **Submit to Smart Contract**: provide the response and proof to
   the destination contract
6. **Verify Proofs**: the `FdcVerification` contract validates the
   proof against the on-chain Merkle root
7. **Act on Data**: the contract uses the now-verified data for its
   logic

## 4. How Providers Reach Consensus (internal detail, useful for debugging)
 
On the provider side, roughly 100 providers retrieve responses from
verifier servers, validate them using MIC (Message Integrity Code) and
LUT (Lowest Usable Timestamp) checks, and submit BitVotes indicating
which requests they consider valid, during the "choose phase" (90 to
135 seconds into the round). At least 50% of provider signature weight
must agree for an attestation to be confirmed.
 
## 5. Security and Storage
 
- Consensus: requires 50%+ signature weight from data providers
- Efficient storage: verified data is stored in a Merkle tree, only
  the root is stored on-chain (the same pattern seen in FTSO Scaling)
- Offchain accessibility: providers serve responses and Merkle proofs
  offchain through the DA Layer

## 6. Why This Matters for the SDK
 
This confirms, rather than changes, the existing `FdcClient` /
`FdcAttestationType` design in the skeleton, but gives the full 7-step
flow to implement at M4: `requestAttestation` on `FdcHub`, voting
round calculation, waiting for finalization, querying the DA Layer,
and final verification via `FdcVerification`. It also confirms the
`requestAttestation` fee needs explicit handling, the same note made
about `FeeCalculator` in the FTSO module.
 
## Sources
 
- [dev.flare.network/fdc/overview](https://dev.flare.network/fdc/overview)
- [dev.flare.network/fdc/attestation-types](https://dev.flare.network/fdc/attestation-types/)
- [flare.network/products/flare-data-connector](https://flare.network/products/flare-data-connector)
- [github.com/flare-foundation/fdc-client](https://github.com/flare-foundation/fdc-client)
