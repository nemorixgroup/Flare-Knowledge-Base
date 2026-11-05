# Local Wallet, P-Chain Address

Part of [SDK Technical Decisions](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md), M1, Foundation &
Wallet.  
Status: ✅ Implemented and verified.  
Implemented in `flare_flutter_sdk` `v0.0.5-dev`.

---

## Decision

`LocalWallet.getPChainAddress()` returns the wallet's P-Chain address
(Bech32, with the `P-` prefix) for the network the wallet is bound to.
The P-Chain and C-Chain addresses come from the same secp256k1 public
key, so the wallet reuses the key it already derives at
`m/44'/60'/account'/0/address`; no second derivation path exists.

## Scope of v0.0.5-dev

Included:

- P-Chain address derivation for the four Flare networks
- An internal Bech32 encoder (BIP-173, encoder only) and an internal
  `PChainAddress` helper, neither exported from the public barrel

Not included yet:

- P-Chain balance, which uses a different endpoint (`/ext/bc/P`) than
  the C-Chain `JsonRpcClient`
- Transaction signing (`signTransaction`), planned for `v0.0.6-dev`
- `WalletConnectController`

## Verified Network Facts

| Fact | Source |
| ---- | ------ |
| P-Chain addresses are Bech32; C-Chain addresses use the Ethereum hex format | Official `flare-stake-tool`, `addresses` command output |
| The P-Chain and C-Chain addresses are derived from the same public key | Official `flare-stake-tool` documentation |
| The public key is in the standard compressed format (33 bytes) | Official `flare-stake-tool`, `addresses` command output |
| Bech32 prefixes `flare`, `songbird` and `coston` | `go-flare` repository, constants |
| The tool accepts `costwo` as the network name for Coston2 | Official `flare-stake-tool`, `--network <flare\|costwo>` |

The official sample response of `flare-stake-tool addresses` pairs a
compressed public key with its P-Chain address:

```
P-chain address: P-flare1pynhfl09rfrf20s83lf6ra5egqylmx757ahxn6
secp256k1 public key: 0x02efe41c5d213089cb7a9e808505e9084bb9eb2bf3aa8050ea92a5ae9e20e5a692
```

## Derivation Pipeline

1. The wallet derives its key as for the C-Chain (BIP-39, BIP-32, path
   `m/44'/60'/account'/0/address`) and keeps the public key in
   compressed form: `0x02` or `0x03` followed by the 32-byte X
   coordinate
2. SHA-256 of the 33 compressed bytes
3. RIPEMD-160 of that digest, giving a 20-byte hash
4. The 20 bytes are regrouped into 5-bit values
5. Bech32 (BIP-173) encodes them with the network prefix and a
   6-character checksum
6. The string `P-` is prepended

Example: the public Hardhat mnemonic
(`test test test test test test test test test test test junk`,
index 0) has the compressed public key
`038318535b54105d4a7aae60c08fc45f9687181b4fdfc625bd1a753fa7397fed75`,
and the same hash gives one address per network:

| Network | Prefix | Address |
| ------- | ------ | ------- |
| Flare | `flare` | `P-flare15428vq2uzwhm3taey9sr9x5vm6tk78ewmae7dk` |
| Songbird | `songbird` | `P-songbird15428vq2uzwhm3taey9sr9x5vm6tk78eww63d63` |
| Coston2 | `costwo` | `P-costwo15428vq2uzwhm3taey9sr9x5vm6tk78ewrnq4xh` |
| Coston | `coston` | `P-coston15428vq2uzwhm3taey9sr9x5vm6tk78ew46xnz9` |

The part after the prefix and the `1` separator is the same on every
network except for the 6-character checksum, because the hash is the
same and only the prefix feeds the checksum.

## Design Decisions

### The Algorithm Is Not Spelled Out in Flare's Documentation

Flare's documentation shows the result (Bech32 P-Chain addresses
derived from the same public key) but does not list the steps. SHA-256,
RIPEMD-160 and Bech32 follow Avalanche, from which Flare's platform
chain is forked. Rather than trust that by assumption, the pipeline was
checked against the official sample: the compressed public key above
produces exactly
`P-flare1pynhfl09rfrf20s83lf6ra5egqylmx757ahxn6`. That test is part of
the suite.

### Prefixes and the Coston2 Caveat

`flare`, `songbird` and `coston` come from the constants in Flare's
`go-flare` repository. `costwo` for Coston2 is not listed on that
constants page. It is supported by the network name accepted by the
official `flare-stake-tool` (`--network <flare|costwo>`) and by a third
party integration (BitGoJS). It is the best available evidence, but it
is not a primary protocol specification; it should be rechecked if
Flare publishes the prefix explicitly.

### One Key, Two Chains

The compressed public key is stored next to the private key when the
wallet is created, so `getPChainAddress()` needs no extra derivation.
The same account therefore has a C-Chain address and a P-Chain address
that always belong together, which is what the official tool documents.

### The Method Lives on LocalWallet, Not on Wallet

`getPChainAddress()` is not part of the `Wallet` interface. A
WalletConnect wallet exposes EIP-1193 accounts, which are C-Chain
addresses, and gives no access to a P-Chain public key. Adding the
method to the interface would force an implementation that cannot be
honored.

### Hand-Rolled Bech32, Encoder Only

The encoder is about 60 lines, the same dependency-surface decision made
for the ABI encoding in `v0.0.3-dev`. It validates its input: the
prefix must be non-empty, lowercase and printable ASCII, and every data
value must fit in 5 bits; otherwise it throws `ArgumentError`. There is
no decoder because nothing in the SDK reads Bech32 yet.

### Input Validation

`PChainAddress` accepts only a 33-byte compressed key starting with
`0x02` or `0x03`. An uncompressed key or any other length throws
`ArgumentError`, because hashing the wrong form would silently produce
a valid-looking but wrong address.

## Test Vectors

The suite does not check the code against itself:

- The official `flare-stake-tool` sample above
- The BIP-173 vectors for the Bech32 encoder
- The per-network Hardhat addresses in the table above, generated
  outside Dart with an independent implementation (pure-Python
  secp256k1 and a separate Keccak library), so a shared mistake in the
  Dart code cannot confirm itself

## Test Coverage

18 new unit tests (52 in total), with no real network call:

- `Bech32`: BIP-173 vectors, prefix validation, 5-bit range validation
- `PChainAddress`: the official sample, the four network vectors,
  invalid public key length and prefix
- `LocalWallet.getPChainAddress`: result per network, the same address
  across repeated calls, consistency with the C-Chain address of the
  same wallet

## Known Limitations

- The derivation steps come from Avalanche and the official sample, not
  from a written Flare specification
- `costwo` has third party confirmation only (see above)
- Only the address is derived; the P-Chain balance and any P-Chain
  transaction need the platform API, which is not part of this version
- The P-Chain address uses the account's first key only; the SDK does
  not manage multiple P-Chain keys per wallet

## Sources

- [github.com/flare-foundation/flare-stake-tool](https://github.com/flare-foundation/flare-stake-tool)
- [dev.flare.network, Flare Stake Tool](https://dev.flare.network/network/guides/flare-stake-tool)
- [github.com/flare-foundation/go-flare](https://github.com/flare-foundation/go-flare)
- [BIP-173, Bech32](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki)

## Related

- [Local Wallet, Key Derivation and Balance](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/local-wallet/README.md), the key this address is derived from
- [Network Configuration](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/network-configuration/README.md), the `FlareNetwork` that selects the prefix
- [Module 01, Foundations](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-01-foundations/README.md), the background on Flare's chains
