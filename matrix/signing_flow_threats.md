# Signing Flow Threat Analysis

**Author:** Soubhagya  
**Scope:** End to end transaction signing pipeline from dApp request to broadcast

## Signing Flow Architecture

```
┌─────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────┐    ┌─────────┐
│  dApp   │───▶│   Provider   │───▶│  Simulator  │───▶│ Preview  │───▶│ Signer  │
│ (web)   │    │ (injection)  │    │  (RPC call) │    │   UI     │    │ module  │
└─────────┘    └──────────────┘    └─────────────┘    └──────────┘    └─────────┘
                     │                    │                 │               │
                     ▼                    ▼                 ▼               ▼
               Origin trust         State accuracy    Human readable   Key access
```

Each stage represents a distinct trust boundary. Failures at any stage can result in unauthorized asset movement while the user believes they approved a benign action.

## Stage 1: Provider Injection

### Threat SF_001: Provider Spoofing
Malicious script injects `window.solana` before legitimate wallet provider loads (race condition on page load).  
**Attack:** dApp calls attacker's provider which returns fake public key and intercepts sign requests.  
**Mitigation:** Provider injection at `document_start`, integrity check that provider matches extension ID, warn on multiple providers detected.

### Threat SF_002: Origin Spoofing via iframe
Attacker embeds victim dApp in invisible iframe, wallet shows parent origin instead of iframe origin on connect prompt.  
**Mitigation:** Display effective origin from `event.source`, block connect from cross origin iframes without explicit user gesture.

### Threat SF_003: Message Channel Confusion
WalletConnect session bound to wrong chain ID, user signs Solana transaction while believing they are on Ethereum.  
**Mitigation:** Chain ID and cluster prominently displayed, reject cross chain session proposals.

## Stage 2: Transaction Simulation

### Threat SF_004: Simulation RPC Manipulation
Attacker controls RPC endpoint, returns simulation showing token receive when actual execution drains wallet.  
**Mitigation:** Simulate against multiple independent RPCs, compare instruction hashes, flag divergence.

### Threat SF_005: Stale Simulation
User approves simulation result, mempool frontrunner modifies transaction before broadcast (less relevant on Solana but applicable to nonce based chains).  
**Mitigation:** Bind simulation to recent blockhash, expire preview after timeout, re simulate on sign click.

### Threat SF_006: Incomplete Simulation Coverage
Novel program instruction not supported by simulator, returns empty diff. User signs blind.  
**Mitigation:** Block signing when simulation unavailable for unknown programs, require explicit "advanced mode" acknowledgment.

### Threat SF_007: Token Account Confusion
Simulation shows transfer of legitimate USDC but instruction creates new token account with attacker mint.  
**Mitigation:** Resolve mint addresses against curated registry, highlight unknown mints in red.

## Stage 3: Preview UI

### Threat SF_008: UI Rendering Mismatch
Wallet UI renders "Send 1 SOL to Alice" but serialized transaction sends 100 SOL to Bob (UI decodes different bytes than signer).  
**Mitigation:** Single source of truth: UI renders from exact bytes passed to signer, display transaction hash of bytes.

### Threat SF_009: Decimal Precision Attack
Token with 6 decimals displays as 1.000000 but amount field encodes 1000000 units (1 full token vs dust).  
**Mitigation:** Show raw amount alongside human readable, use mint metadata for decimal places.

### Threat SF_010: Address Truncation Deception
UI shows `7xKXt...9fG2` matching user's saved contact but full address is different lookalike.  
**Mitigation:** Require full address display on hover, copy full address on confirm, vanity address warnings.

### Threat SF_011: Approval Scope Obfuscation
"Approve swap" button approves unlimited token delegate to drainer program.  
**Mitigation:** Parse and display approval amount, default to exact spend limit, warn on unlimited approvals.

## Stage 4: Signing Module

### Threat SF_012: Signer API Confusion
`signTransaction` vs `signAllTransactions` batch: user approves one tx, wallet signs batch of ten drain transactions.  
**Mitigation:** Enumerate every transaction in batch UI, require per transaction confirmation for high risk.

### Threat SF_013: Message Signing Abuse
User asked to "sign message to login", message is actually SPL token transfer authorization or off chain permit.  
**Mitigation:** Classify message type, warn on structured data resembling transaction, display decoded SIWE/SIWS clearly.

### Threat SF_014: Partial Signature Leak
Multi sig flow: user provides partial signature, attacker combines with their key to complete malicious transaction.  
**Mitigation:** Display full multi sig context, show remaining required signers, warn if user signature alone enables execution.

### Threat SF_015: Nonce Reuse / Replay
Signed transaction captured and rebroadcast (Ethereum). On Solana: durable nonce account misuse.  
**Mitigation:** Display nonce account state, warn on durable nonce transactions from unknown initiators.

## Stage 5: Post Sign Broadcast

### Threat SF_016: Broadcast Interception
Malware replaces signed transaction bytes before RPC submission.  
**Mitigation:** Wallet retains signed bytes, verify broadcast matches signed hash, display confirmation with explorer link.

### Threat SF_017: Private Mempool Front Running
Signed swap transaction visible to validator, sandwich attacked.  
**Mitigation:** Jito bundle option, slippage warnings, MEV risk disclosure.

## Signing Flow Control Requirements

| Control ID | Requirement | Priority |
|------------|-------------|----------|
| SC_01 | No auto sign for any transaction | Mandatory |
| SC_02 | Simulation required before preview | Mandatory |
| SC_03 | Unknown program requires advanced acknowledgment | Mandatory |
| SC_04 | Batch sign shows count and individual previews | Mandatory |
| SC_05 | Origin displayed on every signing request | Mandatory |
| SC_06 | Signed bytes hash shown before confirm | Recommended |
| SC_07 | Multi RPC simulation consensus | Recommended |
| SC_08 | Transaction expiry on blockhash timeout | Recommended |

## Solana Specific Signing Considerations

### Versioned Transactions (v0)
Address lookup tables compress account lists. Attacker includes lookup table resolving benign addresses in preview but swaps table on chain before execution.  
**Mitigation:** Resolve and display all accounts from lookup tables at preview time, verify table contents on chain at sign moment.

### CPI Heavy Programs
Outer instruction appears benign (memo), inner CPI transfers all tokens.  
**Mitigation:** Recursive simulation exposing inner instructions, tree view of CPI calls in preview.

### Program Derived Address Signers
Transaction includes PDA as signer via `invoke_signed`. User may not recognize PDA authority implications.  
**Mitigation:** Flag transactions where wallet is not sole signer, explain PDA authority chain.

## Forensic Artifacts

Preserve for incident response:

1. Raw transaction bytes presented to signer
2. Simulation RPC response JSON
3. dApp origin and referrer
4. User agent and wallet version
5. Timestamp of preview vs sign action (hesitation indicator)
6. RPC endpoint used
7. Blockhash at time of signing

These artifacts enable post incident reconstruction of whether the wallet displayed accurate information.
