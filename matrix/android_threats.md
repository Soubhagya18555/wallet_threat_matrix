# Android Wallet Threat Reference

**Author:** Soubhagya  
**Version:** 1.1  
**Last updated:** 2026-01-28  
**Min SDK:** API 28 (Android 9)  
**Target SDK:** API 34+

Deep dive on Android specific threats for mobile Solana wallets. Maps to catalog WTM ids and matrix mobile_threats.md.

## Threat model assumptions

- User may sideload APKs from third party stores
- Accessibility services may be enabled by unrelated apps
- Device may be rooted with Magisk hide
- Local network may be hostile (captive portal, ARP spoof)
- Other apps share the same user profile and storage namespace

## Platform attack surface

```
┌─────────────────────────────────────────────┐
│           Other installed apps              │
│  (malware, accessibility, clipboard)        │
└──────────────────┬──────────────────────────┘
                   │ Intents, IPC, overlays
┌──────────────────▼──────────────────────────┐
│           Wallet Android app                │
│  Activities / Services / WebView / Keystore │
└──────────────────┬──────────────────────────┘
                   │ TLS, RPC, WalletConnect
┌──────────────────▼──────────────────────────┐
│         Network / dApps / RPC               │
└─────────────────────────────────────────────┘
```

## Critical threats

### A1: Accessibility overlay tapjacking (WTM_012)

**Mechanism:** `AccessibilityService` draws full screen overlay with transparent confirm button aligned over wallet approve control.

**Detection code pattern:**

```kotlin
fun accessibilityServicesActive(): Boolean {
    val enabled = Settings.Secure.getString(
        contentResolver,
        Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES
    ) ?: return false
    return enabled.isNotEmpty()
}
```

**Mitigations:**
- Block signing when accessibility enabled unless user acknowledges risk
- `filterTouchesWhenObscured = true` on approve button
- `FLAG_SECURE` on signing activity

### A2: Intent redirection (WTM_036)

**Mechanism:** Exported `SignTransactionActivity` accepts `ACTION_VIEW` with transaction extra from malicious app.

**Mitigations:**
- `android:exported="false"` on all signing activities
- Use explicit internal intents only
- Validate `callingPackage` against allowlist for deep links

### A3: APK repackaging (WTM_016)

**Mechanism:** Attacker decompiles APK, injects keylogger, resigns with debug cert, distributes via Telegram.

**Mitigations:**
- Play Integrity API `MEETS_DEVICE_INTEGRITY`
- Certificate pinning on update channel
- In app display of signing certificate SHA256 fingerprint

### A4: Clipboard address swap (WTM_007)

**Mechanism:** `ClipboardManager` listener in malware replaces Solana address on copy.

**Mitigations:**
- Detect clipboard change between copy and paste in send flow
- Require user to confirm first and last 8 characters verbally highlighted
- Offer QR scan instead of paste default

### A5: Task hijacking (mobile_threats M6)

**Mechanism:** Attacker activity with `taskAffinity` matching wallet appears in recents stack mimicking login screen.

**Mitigations:**
- Empty task affinity on sensitive activities
- Custom back stack validation before PIN entry
- No password fields in WebView

## High threats

### A6: Cross app URI hijack (WTM_048)

Custom scheme `solana://` or `walletname://` registered by malicious app if wallet does not use App Links with verified domain.

**Mitigation:** Android App Links with `autoVerify=true` and hosted assetlinks.json.

### A7: WebView bridge abuse (WTM_037)

**Mitigation:** Never call `addJavascriptInterface` on pages not loaded from `file:///android_asset/trusted/`.

### A8: Root and Magisk hide (WTM_031)

Multi signal detection:
- `Build.TAGS` contains test-keys
- `/system/bin/su` exists
- `which su` via native check
- Play Integrity device recognition verdict

Degrade to watch-only mode on positive signal.

### A9: Local RPC proxy MITM (WTM_053)

Malware sets device VPN or localhost proxy forwarding RPC to attacker node.

**Mitigation:** Certificate pinning on known RPC providers; warn on user configured HTTP RPC.

### A10: Keystore weak PIN brute force (WTM_052)

**Mitigation:** Argon2id or scrypt KDF with minimum 100ms; exponential backoff; wipe after 10 failures.

## IPC and component security checklist

- [ ] No exported components except verified App Link activity
- [ ] `android:allowBackup="false"` in manifest
- [ ] Encrypted SharedPreferences or SQLCipher for non key metadata
- [ ] Android Keystore with `setUserAuthenticationRequired(true)`
- [ ] No mnemonic in Logcat, crash reports, or Analytics
- [ ] ProGuard/R8 obfuscation on release builds
- [ ] PendingIntent flags `FLAG_IMMUTABLE`
- [ ] FileProvider paths restricted, no world readable files

## Testing matrix

| Test | Tool | Expected result |
|------|------|-----------------|
| Overlay tap | Manual accessibility app | Sign blocked or warning |
| Intent injection | adb shell am start | Rejected without caller verify |
| Repackaged APK | apktool resign | Integrity check fails |
| Frida hook | frida-server on rooted device | Signing disabled |
| Clipboard swap | Mock clipboard app | Mismatch alert |

## Incident response

1. Collect APK hash, package name, install source
2. Preserve logcat during signing attempt (redact keys)
3. Map behavior to WTM id for catalog update
4. Issue IoC package name list to blocklist partners

## Related documents

- `matrix/mobile_threats.md` — cross platform mobile overview
- `docs/SCORING_MODEL.md` — risk prioritization
- `catalog/WTM_036.yaml` through `catalog/WTM_055.yaml` — structured entries
