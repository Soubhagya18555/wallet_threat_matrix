# Mobile Wallet Threat Landscape

**Author:** Soubhagya  
**Platforms:** iOS 16+, Android 12+  
**Scope:** Native mobile wallet applications and mobile browser wallet interfaces

## Threat Model Assumptions

Mobile wallets operate in hostile environments: untrusted networks, shared devices, aggressive app ecosystems with broad permission models, and OS level surveillance tools. Unlike browser extensions with defined origin policies, mobile apps must defend against OS level adversaries, overlay attacks, and cross app data leakage.

## iOS Specific Threats

### M1: Jailbreak Detection Bypass
**Description:** Jailbroken devices allow filesystem introspection, Frida hooking of keychain APIs, and runtime method swizzling on `SecItemAdd` calls.  
**Severity:** Critical  
**Mitigation:** Multi signal jailbreak detection (file paths, syscall behavior, code signing integrity), degrade to watch only mode without signing capability.  
**Detection:** Anomalous signing latency indicating hook overhead.

### M2: iCloud Keychain Sync Leakage
**Description:** If wallet stores encrypted vault in iCloud Keychain without device only flag, mnemonic backup may sync to compromised Apple ID.  
**Severity:** High  
**Mitigation:** `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`, never store seed in iCloud.  
**Detection:** Keychain attribute audit during security review.

### M3: Universal Link Hijacking
**Description:** Attacker registers associated domain with typo, intercepts `https://wallet.app/connect` universal links.  
**Severity:** High  
**Mitigation:** Verify associated domains entitlement, display full URL before processing link.  
**Detection:** Apple App Site Association file monitoring.

### M4: Screen Recording / Screenshot on Seed Display
**Description:** iOS screen recording API captures mnemonic during onboarding.  
**Severity:** Critical  
**Mitigation:** `UITextField.isSecureTextEntry`, detect `UIScreen.isCaptured` and blur sensitive views.  
**Detection:** Screen capture notification to user.

## Android Specific Threats

### M5: Accessibility Service Overlay Attack
**Description:** Malicious app with accessibility permission draws invisible overlay capturing taps intended for wallet confirm button.  
**Severity:** Critical  
**Mitigation:** Detect active accessibility services during signing, warn user, implement tapjacking protection (`filterTouchesWhenObscured`).  
**Detection:** `Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES` enumeration.

### M6: Task Hijacking / StrandHogg
**Description:** Attacker activity appears in wallet task stack, mimics wallet UI for credential entry.  
**Severity:** High  
**Mitigation:** `android:taskAffinity=""`, validate activity back stack, custom URL scheme validation.  
**Detection:** Activity manager anomaly logging.

### M7: APK Repackaging
**Description:** Attacker decompiles wallet APK, injects keylogger, distributes via third party stores.  
**Severity:** Critical  
**Mitigation:** Play Integrity API, certificate pinning, in app signature verification.  
**Detection:** Google Play Protect flags, user reports from sideload channels.

### M8: Intent Redirection
**Description:** Exported activity processes malicious intent extras, triggering unintended signing flow.  
**Severity:** High  
**Mitigation:** Set `android:exported="false"` on sensitive activities, validate intent caller UID.  
**Detection:** Static analysis for exported components.

## Cross Platform Mobile Threats

### M9: Deep Link Parameter Injection
**Description:** `mywallet://sign?tx=BASE64_DRAIN_TX` opened from SMS triggers signing UI.  
**Severity:** Critical  
**Mitigation:** Require in app navigation to signing screen, never auto sign from deep link, display full decoded transaction.  
**Detection:** Telemetry on deep link initiated signatures.

### M10: Push Notification Phishing
**Description:** Attacker sends fake push via compromised backend API key urging immediate seed entry.  
**Severity:** High  
**Mitigation:** Sign push payloads, never include action links requesting secrets, educate users on legitimate notification content.  
**Detection:** Push content policy violations in logs.

### M11: Biometric Bypass
**Description:** Attacker uses high resolution photo or 3D print to pass face unlock on wallet containing decrypted keys.  
**Severity:** High  
**Mitigation:** Require strong biometric class (`BIOMETRIC_STRONG`), re prompt for high value transactions, optional PIN overlay.  
**Detection:** Biometric failure rate spikes.

### M12: Local Network RPC Sniffing
**Description:** Wallet on public WiFi sends unencrypted JSON RPC revealing account addresses and transaction history.  
**Severity:** Medium  
**Mitigation:** TLS mandatory for all RPC, certificate pinning, VPN recommendation for public networks.  
**Detection:** Network traffic audit.

### M13: Clipboard Address Swap
**Description:** Malware monitors clipboard, replaces copied withdrawal address with attacker address.  
**Severity:** Critical  
**Mitigation:** Address confirmation screen with first/last character highlight, clipboard change detection warning.  
**Detection:** Mismatch between clipboard at copy time and paste time.

### M14: Secure Enclave / StrongBox Key Extraction
**Description:** Side channel attack on TEE extracts hardware bound keys.  
**Severity:** Critical (theoretical)  
**Mitigation:** Use platform keystore for key wrapping only, keep seed in encrypted vault with TEE derived KEK, monitor for known TEE CVEs.  
**Detection:** Device firmware version below patched threshold.

### M15: App Clone / Parallel Space
**Description:** User runs wallet in dual app container; container app intercepts IPC.  
**Severity:** High  
**Mitigation:** Detect virtual environment, warn or block signing in cloned instances.  
**Detection:** Package manager signature checks, known virtual app package list.

## Mobile Threat Severity Matrix

| ID | Threat | iOS | Android | Severity |
|----|--------|-----|---------|----------|
| M1 | Jailbreak / root exploitation | ✓ | ✓ | Critical |
| M5 | Accessibility overlay | | ✓ | Critical |
| M7 | APK repackaging | | ✓ | Critical |
| M9 | Deep link injection | ✓ | ✓ | Critical |
| M13 | Clipboard swap | ✓ | ✓ | Critical |
| M4 | Screen capture on seed | ✓ | ✓ | Critical |
| M11 | Biometric bypass | ✓ | ✓ | High |
| M12 | RPC sniffing | ✓ | ✓ | Medium |

## Recommended Mobile Security Controls

1. **Runtime application self protection (RASP):** Detect Frida, Xposed, Magisk, debugger attachment.
2. **Certificate transparency:** Monitor for rogue apps using similar bundle IDs.
3. **Transaction simulation on device:** Never trust server side preview alone.
4. **Hardware backed key storage:** Android Keystore / iOS Secure Enclave for KEK.
5. **Per transaction risk scoring:** Block or delay high risk signing on mobile networks.
6. **Offline signing support:** Airgapped QR flow for high value transfers.

## Testing Methodology

| Test | Tool | Pass Criteria |
|------|------|---------------|
| Overlay attack | Custom accessibility app | Signing blocked with warning |
| Deep link fuzzing | ADB / Xcode URL scheme tester | No auto sign without preview |
| SSL stripping | mitmproxy | Connection fails closed |
| Root detection | Magisk hide enabled device | Signing disabled or warned |
| Clipboard attack | Clipboard monitor script | User warned on address change |
