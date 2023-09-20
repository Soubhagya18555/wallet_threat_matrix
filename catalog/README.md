# Catalog Directory

**Author:** Soubhagya  
**Format:** YAML per threat entry  
**Schema:** `../schema/threat_entry.schema.json`

Individual threat entries for granular review and CI validation. Entries WTM_001 through WTM_035 remain in `data/threat_catalog.json`. New and expanded entries live here as WTM_036+.

## Usage

```bash
# Validate a single entry against schema (requires jsonschema and pyyaml)
python -c "
import json, yaml, jsonschema
from jsonschema import Draft202012Validator
schema = json.load(open('schema/threat_entry.schema.json'))
entry = yaml.safe_load(open('catalog/WTM_036.yaml'))
Draft202012Validator(schema).validate(entry)
print('ok')
"
```

## Index WTM_036 to WTM_055

| ID | Title | Category | Severity |
|----|-------|----------|----------|
| WTM_036 | Android intent redirection | mobile | critical |
| WTM_037 | WebView JavaScript bridge | extension | critical |
| WTM_038 | Secure enclave side channel | key_management | high |
| WTM_039 | Transaction history UI poisoning | dapp_interaction | medium |
| WTM_040 | Coinstalled extension race | extension | critical |
| WTM_041 | SIM swap recovery bypass | recovery | critical |
| WTM_042 | Malicious QR transaction | mobile | critical |
| WTM_043 | Plaintext mnemonic SQLite | key_management | critical |
| WTM_044 | Hardware wallet ghost touch | hardware | high |
| WTM_045 | TLS strip on RPC websocket | network | high |
| WTM_046 | Malicious OTA sideload update | supply_chain | critical |
| WTM_047 | Keyboard autocomplete seed leak | key_management | high |
| WTM_048 | URI scheme hijack Android | mobile | high |
| WTM_049 | Cached simulation trust | signing | critical |
| WTM_050 | Forced token account close | signing | high |
| WTM_051 | NFT metadata XSS | dapp_interaction | high |
| WTM_052 | Weak PIN keystore brute force | key_management | high |
| WTM_053 | Localhost RPC proxy MITM | network | high |
| WTM_054 | Social recovery collusion | recovery | critical |
| WTM_055 | Remote config security override | supply_chain | critical |

See `docs/SCORING_MODEL.md` for prioritization and `matrix/android_threats.md` for Android deep dive.
