# Flare Knowledge Base
 
**A bilingual (Spanish/English) technical knowledge base for the Flare Network**,
built alongside [flare_flutter_sdk](https://github.com/nemorixgroup/flare-flutter-sdk),
the first native Flutter/Dart SDK for Flare.
 
> **Status: getting started.** This repository currently holds the roadmap
> below. Modules are published one at a time, in step with the SDK's own
> milestones.
 
 
## Why this exists
 
Flare's official documentation is thorough but spread across several
sources (the developer hub, protocol-specific pages, the official SDK
docs). This knowledge base consolidates it into one structured, bilingual
reference, written in the process of building `flare_flutter_sdk`, so the
explanations reflect what an SDK implementer actually needs to know, not
just a restatement of the docs.
 
Every module is sourced exclusively from official Flare references
(dev.flare.network and flare.network).
 
## Structure
 
Six modules, each maps to a protocol boundary in the SDK itself:
 
| # | Module | Maps to SDK | Status |
|---|---|---|---|
| 01 | [Flare Foundations](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-01-foundations/README.md) (Flare, Songbird, Coston2, Coston) | `network/` | ✅ Done |
| 02 | [Consensus & Contract Registry](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/module-02-consensus/README.md) | `registry/` | ✅ Done |
| 03 | FTSO, the Flare Time Series Oracle, feeds, feed IDs, block-latency updates | `ftso/` | ⏳ Planned |
| 04 | FDC, the Flare Data Connector, attestation types, the verifier and DA Layer flow | `fdc/` | ⏳ Planned |
| 05 | FAssets, trustless bridging, the FXRP mint/redeem lifecycle, agents and collateral | `fassets/` | ⏳ Planned |
| 06 | Wallets & Account Abstraction, wallet connection patterns and Flare Smart Accounts (FSA) | `wallet/` | ⏳ Planned |
| 07 | Use cases, institutional adoption, development, roadmap | `use_case/` | ⏳ Planned |
 
Modules are published in this order because it mirrors the SDK's own
milestone sequence (M1 through M4), so each module lands once the
corresponding SDK code is far enough along to validate the explanations
against real, working behavior.
 
## SDK Technical Decisions
 
This repository also documents the implementation decisions behind
`flare_flutter_sdk`, the first native Flutter/Dart SDK for the Flare
Network. Every implementation decision is grounded in the official
sources documented in this repository, no third-party references, no
unverified code.
 
Current status: Module 01 (Foundations) complete; SDK implementation
not yet started (v0.0.1-dev on pub.dev is a reserved scaffold, with all
public members throwing `UnimplementedError` by design).
 
| Milestone | Status |
|---|---|
| M1 - Foundation & Wallet | 🔄 In progress |
| M2 - Core Transactions | ⏳ Pending |
| M3 - FTSO | ⏳ Pending |
| M4 - FAssets + FDC | ⏳ Pending |
| M5 - Release v1.0.0 | ⏳ Pending |
 
See [docs-sdk/](https://github.com/nemorixgroup/Flare-Knowledge-Base/blob/main/docs-sdk/README.md) for the full documentation by milestone.

## Related
 
- SDK: [flare_flutter_sdk](https://github.com/nemorixgroup/flare-flutter-sdk)
- Official Flare developer hub: [dev.flare.network](https://dev.flare.network)
- Nemorix Group: [nemorixpay.com](https://nemorixpay.com)
- Contact: sdks@nemorixpay.com

## License
 
MIT License - see [LICENSE](./LICENSE) for details.

## Support This Project

If this project is useful to you or your team, consider supporting its development. Every contribution helps cover infrastructure, documentation, and the time invested in building and maintaining this
open source resource for the Flare blockchain community. Thank you!

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-FFDD00?logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/nemorixgroupllc)
[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-EA4AAA?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/nemorixgroup)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/nemorixgroupllc)

---

*Maintained by [Nemorix Group](https://github.com/nemorixgroup) - built with official sources only.*
