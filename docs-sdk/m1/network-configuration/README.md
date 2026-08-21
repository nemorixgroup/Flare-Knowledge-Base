# Network Configuration
 
Part of [SDK Technical Decisions](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md), M1, Foundation &
Wallet.  
Implemented in `flare_flutter_sdk` `v0.0.2-dev`.
 
---
 
## Decision
 
`NetworkConfig.forNetwork()` resolves a fixed set of values, RPC
endpoint, chain ID, and native currency symbol, for each of Flare's
four networks, using a `switch` statement over the `FlareNetwork` enum
rather than a runtime lookup or configuration file.
 
## Why a Static Switch, Not Dynamic Configuration
 
Flare's network endpoints and chain IDs do not change at runtime, they
are fixed per network by Flare itself. A `switch` with no `default`
case means the Dart analyzer flags a compile time error if a new
`FlareNetwork` value is ever added without updating this method,
instead of silently returning wrong or missing data. This trades a
small amount of flexibility for a correctness guarantee that matters
more for a library other developers depend on.
 
## Verified Network Data
 
| Network | Chain ID | RPC Endpoint | Currency |
|---|---|---|---|
| Flare | 14 | `https://flare-api.flare.network/ext/C/rpc` | FLR |
| Songbird | 19 | `https://songbird-api.flare.network/ext/C/rpc` | SGB |
| Coston2 | 114 | `https://coston2-api.flare.network/ext/C/rpc` | C2FLR |
| Coston | 16 | `https://coston-api.flare.network/ext/C/rpc` | CFLR |
 
All four values were cross-checked against dev.flare.network's Network
Configuration page before implementation, no value in this table came
from a third party aggregator or block explorer.
 
## The `currencySymbol` Field
 
Not part of the original skeleton signature. Added during
implementation because the demo app planned for the M1 milestone
deliverable (wallet balance display) needs to label a balance
correctly, FLR on mainnet is not the same asset as C2FLR on Coston2,
and the SDK should not force every consumer to hardcode that mapping
themselves.
 
## Code
 
```dart
factory NetworkConfig.forNetwork(FlareNetwork network) {
  switch (network) {
    case FlareNetwork.flare:
      return const NetworkConfig(
        network: FlareNetwork.flare,
        rpcUrl: 'https://flare-api.flare.network/ext/C/rpc',
        chainId: 14,
        currencySymbol: 'FLR',
      );
    case FlareNetwork.songbird:
      return const NetworkConfig(
        network: FlareNetwork.songbird,
        rpcUrl: 'https://songbird-api.flare.network/ext/C/rpc',
        chainId: 19,
        currencySymbol: 'SGB',
      );
    case FlareNetwork.coston2:
      return const NetworkConfig(
        network: FlareNetwork.coston2,
        rpcUrl: 'https://coston2-api.flare.network/ext/C/rpc',
        chainId: 114,
        currencySymbol: 'C2FLR',
      );
    case FlareNetwork.coston:
      return const NetworkConfig(
        network: FlareNetwork.coston,
        rpcUrl: 'https://coston-api.flare.network/ext/C/rpc',
        chainId: 16,
        currencySymbol: 'CFLR',
      );
  }
}
```
 
## Test Coverage
 
5 unit tests: one asserting the resolved `chainId`, `rpcUrl`, and
`currencySymbol` for each of the four networks individually, plus one
asserting that all four resolved chain IDs are unique across the set.
 
## Sources
 
- [dev.flare.network/network/overview](https://dev.flare.network/network/overview)
## Related
 
- [Module 02, Consensus & Contract Registry](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-02-consensus/README.md), background on why each network needs its own configuration
