# Wallet Threat Matrix

**Author:** Soubhagya  
**Classification:** Public security research  
**Scope:** Non custodial cryptocurrency wallet threat modeling

A structured threat intelligence framework for evaluating security posture across browser extensions, mobile applications, hardware wallets, and signing infrastructure. This repository provides machine readable catalogs, STRIDE decomposition, attack trees, and signing flow analysis suitable for red team exercises, product security reviews, and SOC threat hunting.

## Repository Structure

```
wallet_threat_matrix/
├── README.md
├── matrix/
│   ├── STRIDE_analysis.md
│   ├── attack_trees.md
│   ├── mobile_threats.md
│   └── signing_flow_threats.md
├── schema/
│   └── threat_entry.schema.json
└── data/
    └── threat_catalog.json
```

## Threat Catalog

The canonical threat inventory lives in `data/threat_catalog.json`. Each entry conforms to `schema/threat_entry.schema.json` and includes:

| Field | Description |
|-------|-------------|
| `id` | Stable identifier (e.g. `WTM_001`) |
| `title` | Human readable threat name |
| `category` | Primary attack surface |
| `stride` | STRIDE classification |
| `severity` | critical / high / medium / low |
| `likelihood` | estimated probability |
| `impact` | financial, reputational, availability |
| `attack_vector` | How the threat manifests |
| `prerequisites` | Conditions required for exploitation |
| `mitigations` | Defensive controls |
| `detection_signals` | Observable indicators |
| `references` | CVEs, advisories, research |

**Current catalog size:** 35 documented threats across extension, mobile, signing, key management, and supply chain surfaces.

## Matrix Documents

| Document | Focus |
|----------|-------|
| [STRIDE_analysis.md](matrix/STRIDE_analysis.md) | Full STRIDE decomposition per wallet component |
| [attack_trees.md](matrix/attack_trees.md) | Goal oriented attack trees with AND/OR nodes |
| [mobile_threats.md](matrix/mobile_threats.md) | iOS/Android specific threat landscape |
| [signing_flow_threats.md](matrix/signing_flow_threats.md) | Transaction preview, blind signing, dApp injection |

## Usage

### Security Review Workflow

1. Import `threat_catalog.json` into your GRC or issue tracker.
2. Map product features to threat categories.
3. Walk attack trees for high value user journeys (onboarding, swap, send, connect).
4. Validate mitigations against each applicable `WTM_*` entry.
5. File gaps as tracked findings with severity inherited from catalog.

### Threat Modeling Integration

Cross reference with [web3_threat_model_framework](../web3_threat_model_framework) for PASTA aligned product reviews. Solana specific on chain threats are documented in [solana_common_vulnerabilities](../solana_common_vulnerabilities).

## Severity Scale

| Level | Definition |
|-------|------------|
| **Critical** | Full key compromise or unauthorized fund transfer at scale |
| **High** | Partial key exposure, persistent session hijack, or signing of unintended transactions |
| **Medium** | Information disclosure enabling chained attacks |
| **Low** | Denial of service, UI deception without fund loss |

## Contributing

Submit pull requests with new `threat_catalog.json` entries. All additions must validate against the JSON schema and include at least one concrete mitigation and detection signal.

## License

MIT License. Attribution appreciated.
