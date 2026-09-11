# Contract Registry Client
 
Part of [SDK Technical Decisions](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md), M1, Foundation &
Wallet.  
Implemented in `flare_flutter_sdk` `v0.0.3-dev`.
 
---
 
## Decision
 
`ContractRegistryClient.getContractAddress()` resolves official Flare
contract addresses by name through the FlareContractRegistry, using a
hand-rolled JSON-RPC `eth_call` rather than a generic EVM library.
 
## Verified Contract Interface
 
Confirmed against four independent official sources (Flare's JS, Go,
and FAssets integration guides, plus the Contract Registry guide
itself):
 
- Registry address: `0xaD67FE66660Fb8dFE9d6b1b4240d8650e30F6019`,
  identical on all four Flare networks
- Function: `getContractAddressByName(string calldata) external view
  returns (address)`
## Why Hand-Rolled ABI Encoding, Not an EVM Library
 
A single function call with one `string` parameter and one `address`
return value does not justify a full EVM library dependency. The
encoding needed is small and stable: a 4-byte function selector
(Keccak-256 of the function signature) followed by the ABI encoding of
one dynamic string parameter (offset, length, then padded UTF-8
bytes). Keeping this hand-rolled keeps the SDK's dependency surface
intentional rather than importing a large library for one call
pattern; if a future milestone needs broader ABI coverage, that
decision can be revisited then.
 
## The Zero Address Case
 
The registry does not revert when asked for an unregistered contract
name, it returns the zero address
(`0x0000000000000000000000000000000000000000`). `getContractAddress`
checks for this explicitly and throws `FlareException` instead of
returning an address that looks valid but is not usable.
 
## JsonRpcClient: a Shared Transport, Not a One-Off
 
The JSON-RPC transport used here (`network/json_rpc_client.dart`) was
built as a shared class rather than embedded directly inside
`ContractRegistryClient`, since wallets (balance queries),
transactions, FTSO, and FAssets will all need the same `eth_call` /
`eth_sendRawTransaction` style transport in later milestones.
 
## Test Coverage
 
3 unit tests using `package:http/testing.dart`'s `MockClient`, no real
network call is made during the test suite:
 
- A successful resolution, decoding a known mocked address correctly
- An unregistered contract name, asserting `FlareException` is thrown
- A non-200 HTTP response, asserting `FlareException` is thrown
## Sources
 
- [dev.flare.network/network/guides/flare-contracts-registry](https://dev.flare.network/network/guides/flare-contracts-registry)
- [dev.flare.network/network/guides/flare-for-javascript-developers](https://dev.flare.network/network/guides/flare-for-javascript-developers/)
- [dev.flare.network/network/guides/flare-for-go-developers](https://dev.flare.network/network/guides/flare-for-go-developers)
## Related
 
- [Network Configuration](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/network-configuration/README.md), the `NetworkConfig` this client depends on
- [Module 02, Consensus & Contract Registry](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-02-consensus/README.md), the conceptual background for this pattern
