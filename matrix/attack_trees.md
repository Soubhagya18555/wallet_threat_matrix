# Wallet Attack Trees

**Author:** Soubhagya  
**Notation:** `AND` = all children required, `OR` = any child sufficient

## Tree 1: Steal User Funds (Primary Goal)

```
GOAL: Transfer assets from victim wallet to attacker
│
├─ OR ─ Obtain signing key material
│   ├─ AND ─ Extract mnemonic
│   │   ├─ OR ─ Phishing seed phrase entry (fake support, airdrop claim)
│   │   ├─ OR ─ Malware keylogger on backup flow
│   │   ├─ OR ─ Clipboard hijack during recovery
│   │   └─ OR ─ Supply chain compromise of wallet binary
│   ├─ OR ─ Extract derived private key
│   │   ├─ OR ─ Memory scrape while key decrypted
│   │   ├─ OR ─ Side channel on signing operation
│   │   └─ OR ─ Exploit in secure enclave API
│   └─ OR ─ Bypass key requirement entirely
│       ├─ OR ─ Exploit smart contract approval (unlimited delegate)
│       └─ OR ─ Session key with excessive permissions
│
├─ OR ─ Coerce valid signature without key theft
│   ├─ AND ─ Transaction substitution
│   │   ├─ Malicious dApp presents benign UI
│   │   └─ Actual instruction set transfers to attacker
│   ├─ AND ─ Blind signing exploitation
│   │   ├─ User approves opaque hex blob
│   │   └─ Blob contains drain instruction
│   ├─ OR ─ Address poisoning
│   │   └─ User copies lookalike address from history
│   └─ OR ─ Social engineering direct transfer
│       └─ Attacker impersonates support requesting send
│
└─ OR ─ Exploit wallet infrastructure
    ├─ OR ─ Compromise update server (deliver backdoored build)
    ├─ OR ─ RPC manipulation (hide outgoing drain tx)
    └─ OR ─ WalletConnect relay interception
```

### Kill Chain Mapping

| Stage | Tree Branch | Typical TTP |
|-------|-------------|-------------|
| Recon | Identify high value target | OSINT on public wallet labels |
| Weaponize | Build drainer site / malware | Pink drainer kit, custom Anchor program |
| Deliver | Phishing link, malvertising | Discord DM, Google ad |
| Exploit | Trigger signing or seed capture | Fake mint, support scam |
| Exfiltrate | Transfer to mixer | Cross chain bridge, CEX deposit |

## Tree 2: Persistent Wallet Surveillance

```
GOAL: Monitor victim transactions without immediate theft
│
├─ OR ─ Network level observation
│   ├─ Compromise RPC provider logs
│   ├─ MITM on unencrypted RPC (legacy endpoints)
│   └─ Blockchain indexer API key abuse
│
├─ OR ─ Application level observation
│   ├─ Malicious extension with read permissions
│   ├─ Compromised analytics SDK
│   └─ dApp with excessive `getAccounts` polling
│
└─ OR ─ Device level observation
    ├─ Screen recording malware
    ├─ Accessibility service abuse (Android)
    └─ iOS MDM profile with network inspection
```

## Tree 3: Deny User Access to Funds

```
GOAL: Lock victim out of wallet (ransom / harassment)
│
├─ OR ─ Destroy key material
│   ├─ Overwrite encrypted vault
│   ├─ Trigger factory reset via malware
│   └─ Social engineer user to delete wallet
│
├─ OR ─ Exhaust authentication
│   ├─ Brute force PIN to lockout threshold
│   └─ Flood biometric sensor to disable
│
└─ OR ─ Network level blocking
    ├─ RPC endpoint blacklist of victim addresses
    └─ DNS sinkhole of wallet API domains
```

## Tree 4: Compromise Wallet Development Pipeline

```
GOAL: Ship malicious code to all wallet users
│
├─ AND ─ Gain CI/CD access
│   ├─ OR ─ Stolen developer credentials
│   ├─ OR ─ Compromised GitHub OAuth app
│   └─ OR ─ Insider threat
│
├─ AND ─ Inject payload
│   ├─ OR ─ Malicious npm package (typosquat)
│   ├─ OR ─ Modified build script
│   └─ OR ─ Backdoored compiler (advanced)
│
└─ AND ─ Evade detection
    ├─ Delayed activation (time bomb)
    ├─ Targeted delivery (geo, balance threshold)
    └─ Obfuscated exfiltration channel
```

## Defensive Countermeasures by Tree Node

| Attack Node | Primary Defense | Detection Signal |
|-------------|----------------|------------------|
| Phishing seed entry | Never ask for seed in app UI | User report, domain takedown |
| Transaction substitution | Hardware wallet screen verification | Mismatch between UI hash and signed bytes |
| Blind signing | Mandatory simulation + human readable preview | High approval rate on unknown programs |
| Supply chain | Reproducible builds, Sigstore signing | Dependency hash drift alert |
| RPC manipulation | Multi endpoint state consensus | Balance discrepancy across RPCs |
| CI/CD compromise | Branch protection, hardware 2FA for releases | Unexpected release artifact hash |

## Attack Tree Usage in Red Team Exercises

1. Select goal tree aligned with engagement scope.
2. Identify lowest cost OR branch (attacker economics).
3. Test each AND gate as separate attack scenario.
4. Document which gates existing controls block.
5. Report unblocked paths as findings with inherited severity from `threat_catalog.json`.
