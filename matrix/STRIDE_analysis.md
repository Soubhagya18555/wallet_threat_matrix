# STRIDE Analysis: Wallet Security Components

**Author:** Soubhagya  
**Version:** 1.0  
**Methodology:** Microsoft STRIDE adapted for non custodial wallet architecture

## Component Decomposition

| Component | Trust Boundary | Assets Protected |
|-----------|---------------|------------------|
| Seed phrase / mnemonic | User device secure enclave | Root key material |
| Key derivation (BIP39/BIP44/SLIP-0010) | Application memory | Derived signing keys |
| Transaction builder | Wallet UI + RPC layer | Unsigned transaction bytes |
| Signing module | Isolated signing context | Signature nonces |
| dApp provider / WalletConnect | Browser ↔ extension IPC | Session keys, origin trust |
| RPC endpoint | Network boundary | Account state, simulation data |
| Update channel | CDN / app store | Binary integrity |
| Backup / recovery flow | Cloud or paper | Mnemonic plaintext |

## Spoofing

### S1: Malicious dApp Origin Spoofing
**Threat:** Attacker registers domain visually similar to legitimate DeFi protocol (`uniswаp.org` using Cyrillic `а`).  
**Impact:** User connects wallet to drainer contract believing they interact with trusted protocol.  
**Mitigation:** Display punycode warnings, enforce allowlist for high value operations, origin pinning for known partners.

### S2: WalletConnect Session Impersonation
**Threat:** Attacker initiates WalletConnect URI relay, victim scans attacker's QR from phishing page.  
**Impact:** Attacker gains signing session without browser extension visibility.  
**Mitigation:** Display connecting dApp metadata, require explicit user confirmation of pairing domain, session timeout.

### S3: RPC Endpoint Spoofing
**Threat:** Compromised DNS or malicious WiFi redirects RPC calls to attacker controlled node returning falsified account balances.  
**Impact:** User believes they hold tokens that do not exist, or misses pending drain transactions.  
**Mitigation:** Multi RPC consensus checks, certificate pinning for hosted RPC, display block hash alongside state.

## Tampering

### T1: Transaction Instruction Substitution
**Threat:** Malware hooks `signTransaction` API, replaces transfer recipient between UI render and signing.  
**Impact:** Funds sent to attacker address while UI shows intended recipient.  
**Mitigation:** Hash displayed fields into signing payload, hardware wallet screen shows canonical recipient, immutable preview snapshot.

### T2: Extension Supply Chain Tampering
**Threat:** Compromised npm dependency injects key exfiltration into wallet build pipeline.  
**Impact:** Silent mnemonic exfiltration on wallet creation.  
**Mitigation:** Reproducible builds, dependency pinning with hash verification, subresource integrity for web assets.

### T3: Local Storage Tampering
**Threat:** Another extension with `storage` permission overwrites encrypted vault blob.  
**Impact:** Wallet corruption or substitution of attacker controlled ciphertext.  
**Mitigation:** Authenticated encryption with associated data binding vault to extension ID, integrity checks on load.

## Repudiation

### R1: Unsigned Action Logging Gap
**Threat:** Wallet does not log rejected signing attempts; user later disputes unauthorized transfer.  
**Impact:** Forensic investigation lacks evidence of user consent or phishing exposure.  
**Mitigation:** Append only local audit log of all signing requests with origin, timestamp, transaction hash preview.

### R2: Missing Transaction Simulation Audit Trail
**Threat:** No record of simulation results shown to user before signing.  
**Impact:** Cannot prove whether wallet displayed accurate token transfer preview.  
**Mitigation:** Store simulation RPC response hash alongside signed transaction.

## Information Disclosure

### I1: Mnemonic Clipboard Exposure
**Threat:** Seed phrase copied to system clipboard remains accessible to other applications.  
**Impact:** Clipboard monitoring malware harvests recovery phrase.  
**Mitigation:** Auto clear clipboard after timeout, warn on copy, prefer QR based backup.

### I2: Private Key in Crash Reports
**Threat:** Error telemetry includes memory dumps containing decrypted key material.  
**Impact:** Third party analytics provider receives key material.  
**Mitigation:** Scrub sensitive fields from crash reports, local only error logging for crypto operations.

### I3: Account Enumeration via Public RPC
**Threat:** Wallet queries all derived addresses on connect, leaking user portfolio to RPC operator.  
**Impact:** Privacy degradation, targeted phishing based on holdings.  
**Mitigation:** Lazy address derivation, optional Tor/proxy RPC, minimize address fanout queries.

## Denial of Service

### D1: RPC Rate Limit Exhaustion
**Threat:** Attacker floods wallet with pending transaction notifications via spam airdrops.  
**Impact:** Wallet UI becomes unusable, user cannot access legitimate funds.  
**Mitigation:** Spam token filtering, notification throttling, hide unknown token transfers by default.

### D2: Vault Lockout via Failed PIN
**Threat:** Attacker triggers maximum PIN attempts remotely if device left unlocked.  
**Impact:** Legitimate user locked out, forced recovery via mnemonic.  
**Mitigation:** Exponential backoff, biometric fallback, remote wipe capability.

## Elevation of Privilege

### E1: Content Script Privilege Escalation
**Threat:** dApp content script exploits extension messaging to invoke privileged APIs (export key).  
**Impact:** Full wallet compromise from malicious webpage.  
**Mitigation:** Strict message origin validation, capability based API surface, no key export via content script path.

### E2: Deep Link Handler Abuse
**Threat:** Mobile wallet processes `solana://` deep link with crafted parameters triggering unintended signing.  
**Impact:** One click fund drain from SMS or email link.  
**Mitigation:** Deep link allowlist, always show full transaction preview, disable auto sign.

### E3: Firmware Downgrade Attack (Hardware)
**Threat:** Attacker flashes older firmware with known vulnerability to hardware wallet.  
**Impact:** Bypass of patched secure element protections.  
**Mitigation:** Secure boot with monotonic version counters, signed firmware only.

## Cross Component Threat Matrix

| Threat ID | Spoofing | Tampering | Repudiation | Info Disclosure | DoS | Elevation |
|-----------|----------|-----------|-------------|-----------------|-----|-----------|
| WTM_001 | ✓ | | | | | |
| WTM_002 | | ✓ | | | | ✓ |
| WTM_003 | ✓ | | | ✓ | | |
| WTM_004 | | ✓ | ✓ | | | |
| WTM_005 | | | | ✓ | | ✓ |
| WTM_006 | | | | | ✓ | |
| WTM_007 | ✓ | ✓ | | | | ✓ |

## Review Cadence

Reassess STRIDE mappings quarterly or upon: new wallet feature release, critical CVE in dependency tree, observed in the wild exploit campaign targeting wallet users.
