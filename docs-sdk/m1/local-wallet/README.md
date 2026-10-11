# Local Wallet, Key Derivation and Balance

Part of [SDK Technical Decisions](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md), M1, Foundation &
Wallet.  
**Status:** ✅ Implemented and verified  
Implemented in `flare_flutter_sdk` `v0.0.4-dev`.

---

## Decision

`LocalWallet.fromMnemonic()` derives a C-Chain wallet from a BIP-39
mnemonic along the BIP-44 path `m/44'/60'/account'/0/address`.
`getAddress()` returns the address with an EIP-55 checksum, and
`getBalance()` reads the native balance in wei through the shared
`JsonRpcClient`. The wallet is bound to one `FlareNetwork` when it is
created.

## Scope of v0.0.4-dev

Included:

- Key derivation from a mnemonic, with optional `accountIndex`,
  `addressIndex`, and BIP-39 `passphrase`
- C-Chain address derivation with EIP-55 checksum
- Native balance read (`eth_getBalance`)

Not included yet (still `UnimplementedError` or not started):

- Transaction signing (`signTransaction`)
- P-Chain address derivation (Bech32), planned for `v0.0.5-dev`
- `WalletConnectController`

## Verified Network Facts

| Fact | Source |
| ---- | ------ |
| Default derivation path is `m/44'/60'/0'/0/0` (BIP-44 / Ledger Live standard, Ethereum coin type 60) | Official `flare-stake-tool`, `--derivation-path` option |
| C-Chain addresses use the Ethereum hex format, P-Chain addresses use Bech32 | Official `flare-stake-tool`, `addresses` command output |
| Address space matches Ethereum: 20-byte addresses using ECDSA | dev.flare.network, Network page, Transaction format |
| Transactions follow EIP-2718, encoded with RLP, with Type 0 (legacy) and Type 2 (EIP-1559) | dev.flare.network, Network page, Transaction format |

Flare's documentation does not publish a dedicated derivation
specification for C-Chain wallets. The path used here comes from the
official staking tool, and the pipeline follows from the documented
Ethereum-compatible address space. The tool's default path is a tool
default, not a protocol rule; the SDK adopts it as its default for the
same reason Ethereum tooling does, because the address space is the
same.

## Derivation Pipeline

1. The mnemonic is trimmed and its whitespace is collapsed, then
   validated as a BIP-39 phrase
2. The mnemonic (plus optional passphrase) becomes a 64-byte seed
   through BIP-39's PBKDF2 step
3. BIP-32 derives the child key along
   `m/44'/60'/account'/0/address`
4. The public key is computed as the private key times the secp256k1
   generator point and encoded uncompressed (`0x04 || X || Y`)
5. The address is the last 20 bytes of the Keccak-256 hash of `X || Y`
   (the `0x04` prefix is dropped)
6. The address is formatted with EIP-55 checksum casing

## Design Decisions

### EIP-55 Checksum Casing

`getAddress()` returns a mixed-case address. EIP-55 is an Ethereum
convention, not a Flare requirement, and Flare's documentation does not
mention it. It is adopted for interoperability: wallets and explorers
in the Ethereum ecosystem display addresses this way, and the casing
catches most typing errors. Because it only changes letter casing, the
address is equivalent to its lowercase form on-chain.

### The Network Is Required, With No Default

`fromMnemonic` takes a required `FlareNetwork`. The address is
identical on all four networks, but balances are not, and the signing
chain ID will not be either once signing lands. A silent default (for
example Coston2) would make it easy to read or, later, sign on the
wrong network without noticing. An explicit parameter makes that
impossible by construction. The wallet exposes the `NetworkConfig` it
is bound to through `config`.

### The Private Key Has No Getter

The private key stays inside `LocalWallet`. The `Wallet` interface
never assumes key custody, and no API returns the raw key. The signing
step in a later version will use the key internally.

### Two Kinds of Errors

- An invalid mnemonic throws `FlareException`, because it is user
  input that can be wrong at runtime
- An index outside `0` to `0x7FFFFFFF` throws `RangeError`, because it
  is a programmer error; the upper bound is where BIP-32 reserves the
  hardened range

### Injecting a JsonRpcClient, for Tests Only

`fromMnemonic` accepts an optional `JsonRpcClient`, the same pattern
as `ContractRegistryClient`. Production code leaves it unset. If the
injected client targets a different network than the wallet, the
constructor throws `ArgumentError`, so a mismatched client cannot
silently read the wrong balances.

## getBalance

`getBalance()` calls `eth_getBalance` with the wallet's own address
and the `latest` block tag. The node returns a hex quantity in wei
(18 decimals), which is parsed into a `BigInt`. A `BigInt` is required
because a balance of 1000 FLR is `10^21` wei, far beyond 64 bits.

A result that is not a `0x`-prefixed hex quantity, including a bare
`0x` with no digits, throws `FlareException` instead of returning a
wrong number. Node errors and non-200 responses are already surfaced
as `FlareException` by `JsonRpcClient`.

## Test Vectors

The address derivation is validated against known values, so the full
chain (BIP-39, BIP-32, secp256k1, Keccak-256, EIP-55) is checked and
not only "does not throw":

- Hardhat default development mnemonic
  (`test test test test test test test test test test test junk`):
  index 0 gives `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`, index 1
  gives `0x70997970C51812dc3A010C7d01b50e0d17dc79C8`
- Standard BIP-39 test mnemonic (`abandon` repeated 11 times plus
  `about`): index 0 gives `0x9858EfFD232B4033E47d90003D41EC34EcaEda94`

These are public test phrases. Their addresses are known to everyone,
so they must never hold real funds.

## Test Coverage

23 unit tests, no real network call during the test suite
(`package:http/testing.dart`'s `MockClient`):

- Known vectors: the three addresses above
- Format and determinism: `0x` plus 40 hex characters, same inputs give
  the same address
- Derivation parameters: `accountIndex` and passphrase change the
  address, extra whitespace is tolerated, the address is the same on
  every network
- Network binding: `config` reflects the bound network, an `rpcClient`
  for another network is rejected
- Input validation: invalid mnemonic, negative indexes, and an index in
  the hardened range
- `getBalance`: zero, 1 FLR (`10^18` wei), a value beyond 64 bits, the
  exact request sent (method, address, `latest`), RPC error object,
  HTTP error status, a non-hex result, and an empty `0x` result

## Known Limitations

- `JsonRpcClient.call` casts `result` to `String`. A node answering
  `result: null` without an `error` would raise a `TypeError` instead
  of a `FlareException`. It is outside this version's scope and will be
  hardened in a separate change
- BIP-39's PBKDF2 runs in pure Dart on the calling isolate and can take
  a noticeable moment; it has not been measured on devices yet
- The private key lives in a Dart `Uint8List`. Dart gives no guarantee
  that memory is wiped when the object is released

## Sources

- [github.com/flare-foundation/flare-stake-tool](https://github.com/flare-foundation/flare-stake-tool)
- [dev.flare.network/network/overview](https://dev.flare.network/network/overview)
- [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
- [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
- [EIP-55](https://eips.ethereum.org/EIPS/eip-55)

## Related

- [Network Configuration](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/network-configuration/README.md), the `NetworkConfig` each wallet is bound to
- [Contract Registry Client](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/contract_registry_client/README.md), which introduced the `JsonRpcClient` transport that `getBalance` reuses
- [Module 01, Foundations](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-01-foundations/README.md), the background on Flare's networks and the C-Chain
