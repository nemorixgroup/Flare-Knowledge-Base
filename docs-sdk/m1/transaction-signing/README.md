# Local Wallet, Transaction Signing

Part of [SDK Technical Decisions](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md), M1, Foundation &
Wallet.  
Implemented in `flare_flutter_sdk` `v0.0.6-dev`.

---

## Decision

`LocalWallet.signTransaction()` signs an `Eip1559Transaction` (type 2)
and returns the raw transaction as a `0x`-prefixed hex string, ready for
`eth_sendRawTransaction`. The wallet supplies the chain ID of the network
it is bound to, so the transaction object has no chain ID of its own. The
SDK signs only: it does not send, look up the nonce or estimate fees.

## Scope of v0.0.6-dev

Included:

- `Eip1559Transaction`, a public unsigned type 2 transaction
- `LocalWallet.signTransaction()`
- An internal RLP encoder and an internal deterministic secp256k1 signer

Not included yet:

- Type 0 (legacy) transactions
- Sending a transaction (`eth_sendRawTransaction`), nonce lookup and fee
  estimation, planned for M2 (`TransactionBuilder`)
- Contract creation (`to` is required) and non-empty access lists
- `WalletConnectController`, planned for `v0.0.7-dev`

## Verified Network Facts

| Fact | Source |
| ---- | ------ |
| Transactions follow EIP-2718 and are encoded with RLP | dev.flare.network, Network page, Transaction format |
| Supported types: Type 0 (fee is `gasUsed * gasPrice`) and Type 2 (fee is `(baseFee + priorityFee) * gas`); Type 1 is not mentioned | dev.flare.network, Network page, Transaction format |
| All transaction fees are burned, for both types | dev.flare.network, Network page, Transaction format |
| Addresses are 20 bytes and use ECDSA | dev.flare.network, Network page, Transaction format |
| The minimum C-chain base fee (floor) was raised from 25 to 500 GWei in go-flare v1.14.0 (Granite upgrade) | go-flare, release notes |
| Granite activation, 12 PM UTC: Coston on 11 June 2026, Coston2 on 16 June, Songbird on 7 July, Flare on 14 July | go-flare, release notes |

Flare's documentation does not cover the transaction gas limit, the
block gas limit or default priority fee values, and does not describe
the signature fields. Those parts follow Ethereum's specifications, which
Flare's page references through EIP-2718.

The fee floor needs a caveat. FIP.16, the governance proposal behind the
change, states 500 gwei in its text but shows 1200 gwei in a comparison
table, and does not explain the difference. The release notes are the
source used here. The SDK does not set fees, so this value does not
enter the code; it matters for M2, which must read fees from the network.

## Transaction Format

The signed type 2 transaction is `0x02 || rlp([chain_id, nonce,
max_priority_fee_per_gas, max_fee_per_gas, gas_limit, destination, amount,
data, access_list, signature_y_parity, signature_r, signature_s])`. The
signature is made over the Keccak-256 hash of the same payload without the
last three fields.

As a worked example, this transfer of 1 FLR on Coston2 (chain ID 114)
signed by account 0 of the public Hardhat mnemonic:

| Field | Value |
| ----- | ----- |
| nonce | 5 |
| maxPriorityFeePerGas | 2000000000 (2 gwei) |
| maxFeePerGas | 1000000000000 (1000 gwei) |
| gasLimit | 21000 |
| to | `0x70997970C51812dc3A010C7d01b50e0d17dc79C8` |
| value | 1000000000000000000 (1 FLR) |
| data | empty |

has this signing payload, which breaks down field by field:

```
02 f0 72 05 84 77359400 85 e8d4a51000 82 5208
94 70997970c51812dc3a010c7d01b50e0d17dc79c8
88 0de0b6b3a7640000 80 c0
```

| Bytes | Meaning |
| ----- | ------- |
| `02` | Transaction type |
| `f0` | List prefix, 48 payload bytes |
| `72` | Chain ID 114 |
| `05` | Nonce 5 |
| `84 77359400` | Priority fee, 4 bytes |
| `85 e8d4a51000` | Max fee, 5 bytes |
| `82 5208` | Gas limit 21000 |
| `94 ...` | Recipient, 20 bytes |
| `88 0de0b6b3a7640000` | Value, 8 bytes |
| `80` | Empty data |
| `c0` | Empty access list |

and this signed result:

```
0x02f8737205847735940085e8d4a510008252089470997970c51812dc3a010c7d01b50e0d17dc79c8880de0b6b3a764000080c001a0a75d09e40b503f0cc03ba9d29ac9890efe22506914ae22352981fa05a35cc6b5a07a01e16129d14720365577d223232b335ecedb51f0ef119886630c3179eee48f
```

with `yParity` 1.

## Signing Pipeline

1. `Eip1559Transaction.encodeForSigning(chainId)` builds
   `0x02 || rlp([...])` with an empty access list
2. The payload is hashed with Keccak-256
3. The signer derives the nonce `k` with RFC 6979 (HMAC-SHA256) from the
   private key and the hash, so the same inputs always give the same
   signature
4. `r` is the x coordinate of `k * G`, and
   `s = k^-1 * (hash + r * key) mod n`
5. If `s` is above half the curve order, it is replaced by `n - s` and
   the y parity is flipped (EIP-2)
6. `encodeSigned()` appends `yParity`, `r` and `s` to the RLP list and
   prefixes the type byte

## Design Decisions

### Only Type 2

Flare supports type 0 and type 2. Type 2 covers the normal case and is the
one whose fee model Flare documents for new use, so this version builds
only type 2. Type 0 can be added later without changing the current API.

### The Chain ID Belongs to the Wallet

`Eip1559Transaction` has no chain ID. `signTransaction` takes it from the
`NetworkConfig` the wallet is bound to. This follows the decision made
for `fromMnemonic` in `v0.0.4-dev`: a transaction cannot be signed for
the wrong network by mistake, and the same transaction object signed by a
Flare wallet and a Coston2 wallet gives two different, valid results.
A WalletConnect wallet will not need a chain ID in the transaction
either.

### A Signer Written for the SDK

Signing uses `pointycastle` primitives with the RFC 6979 nonce written in
the SDK. The package that provides key derivation (`bip32`) was not used to
sign, because its exact signing behavior (output format, low `s`) could
not be confirmed. Owning the code keeps those guarantees inside the SDK,
where its tests check them.

### The Y Parity Comes From the Nonce Point

The recovery id is the parity of the y coordinate of `k * G`, which is
known at signing time, so no trial recovery of the public key is needed.
When `s` is mirrored, the point is mirrored with it and the parity flips.
The rare case where the x coordinate is at or above the curve order is
skipped and the next RFC 6979 nonce is used.

### Validation in the Transaction

`Eip1559Transaction` throws `ArgumentError` for: negative numbers,
fees or value above 256 bits, a recipient that is not a `0x`-prefixed
20-byte hex string, and a `maxFeePerGas` below `maxPriorityFeePerGas`.
The last rule is not stated in Flare's documentation. It is the common
client rule in the Ethereum ecosystem (viem and reth reject the same
case), and catching it before signing avoids a transaction that a node
would refuse. The EIP-55 checksum of the recipient is not verified.

### Signing Only

The method returns the signed bytes and sends nothing, so nothing leaves
the device. Broadcasting, reading the nonce and estimating fees belong to
M2, where the transaction builder will need the network.

### Private Key Hygiene

The private key is never included in an error message, and `LocalWallet`
still has no getter for it.

## Test Vectors

The tests do not check the code against itself:

- Four transactions, one per network (Coston2, Flare with an ERC-20
  `transfer`, Songbird with values above 64 bits, Coston with zero
  fees), were signed outside Dart with `eth-account`. The signing payload
  was built independently with the Python `rlp` package, and `eth_keys`
  recovered the signer from each signature. The three agree byte for byte
  and the recovered key is the one of
  `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`. The SDK reproduces every
  signed transaction exactly
- Six standalone signatures over fixed hashes, three of them with a raw
  `s` above half the order, which checks the EIP-2 mirroring and the
  parity flip
- RLP encodings from ethereum.org and the Python `rlp` package, including
  the boundaries at 55 and 56 bytes and a payload above 255 bytes

## Test Coverage

64 new unit tests (116 in total), with no network call:

- `Rlp`, 24 tests: integers, byte strings, lists, boundaries and errors
- `Secp256k1Signer`, 12 tests: the six vectors, determinism, `s` in the
  lower half over 30 hashes, input validation and no key in error
  messages
- `Eip1559Transaction`, 19 tests: signing payload and signed form for the
  four vectors, and every validation rule
- `LocalWallet.signTransaction`, 9 tests: the four vectors through the
  wallet, the chain ID of each network, determinism, a different result on
  another network, the `0x02` prefix and rejection of other types

## Known Limitations

- Dart big integers do not run in constant time, so signing is meant for
  client side use, not for a shared service exposed to timing
  measurements
- Only type 2 is supported, with an empty access list and a required
  recipient, so contract creation is not possible yet
- `nonce` and `gasLimit` are Dart `int`, which has 53 bits of precision on
  Flutter web; realistic values are far below that
- `Eip1559Transaction.data` returns the stored list, so a caller can
  modify it after construction; the constructor copies its input but the
  getter does not copy again
- The EIP texts were cross-checked through other specifications that
  quote them and through the byte for byte agreement of three independent
  implementations, not by reading each EIP page in full
- The minimum base fee changed in July 2026; a fee value hardcoded from
  older documentation would be rejected

## Sources

- [dev.flare.network, Network overview](https://dev.flare.network/network/overview)
- [go-flare, release notes](https://github.com/flare-foundation/go-flare/blob/main/RELEASES-flare.md)
- [FIP.16](https://proposals.flare.network/FIP/FIP_16.html)
- [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559)
- [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718)
- [EIP-2](https://eips.ethereum.org/EIPS/eip-2)
- [RFC 6979](https://www.rfc-editor.org/rfc/rfc6979)
- [ethereum.org, RLP](https://ethereum.org/developers/docs/data-structures-and-encoding/rlp/)

## Related

- [Local Wallet, Key Derivation and Balance](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/local-wallet/README.md), the key and the network binding that signing relies on
- [Local Wallet, P-Chain Address](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/p-chain-address/README.md), the other use of the same key
- [Network Configuration](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/m1/network-configuration/README.md), the chain IDs the wallet signs with
- [Module 01, Foundations](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-01-foundations/README.md), the background on Flare's networks
