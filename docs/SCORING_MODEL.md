# Threat Scoring Model

**Author:** Soubhagya  
**Version:** 1.1  
**Last updated:** 2026-01-28

Quantitative risk scoring for wallet threat catalog entries (WTM_*). Combines impact, likelihood, and platform exposure into a prioritized remediation backlog.

## Formula

```
Risk Score = (Impact × Likelihood × Exposure) / 10

Where each factor maps to integer 1-5
```

| Score range | Rating | Action |
|-------------|--------|--------|
| 50-125 | P0 Critical | Block release until mitigated |
| 25-49 | P1 High | Fix current sprint |
| 10-24 | P2 Medium | Scheduled backlog |
| 1-9 | P3 Low | Monitor or accept |

## Impact factor (1-5)

| Score | Financial | Reputational | Availability |
|-------|-----------|--------------|--------------|
| 5 | total_loss | severe | complete |
| 4 | total_loss | moderate | degraded |
| 3 | partial_loss | moderate | degraded |
| 2 | partial_loss | minor | minimal |
| 1 | minimal/none | none | none |

Map from catalog `impact` object: use highest dimension score.

## Likelihood factor (1-5)

| Catalog value | Score |
|---------------|-------|
| very_high | 5 |
| high | 4 |
| medium | 3 |
| low | 2 |
| very_low | 1 |

## Exposure factor (1-5)

Based on count of `affected_platforms` and user population weight:

| Condition | Score |
|-----------|-------|
| browser_extension + android + ios | 5 |
| Two major platforms | 4 |
| Single mobile platform | 3 |
| hardware_wallet only | 2 |
| desktop niche | 1 |

Adjust +1 if threat requires no user interaction beyond default behavior.

## STRIDE weight modifier

Threats spanning multiple STRIDE categories receive 1.15x multiplier (rounded):

```
Adjusted Score = Risk Score × (1 + 0.15 × (stride_count - 1))
```

## Example calculations

### WTM_011 Unlimited token approval drain

- Impact: 5 (total_loss)
- Likelihood: 5 (very_high)
- Exposure: 5 (extension + mobile)
- Base: (5 × 5 × 5) / 10 = 12.5
- STRIDE: tampering only → 12.5
- **Rating: P2** (high priority within medium band due to very_high likelihood override to P1 in practice)

### WTM_012 Accessibility overlay tapjacking

- Impact: 5
- Likelihood: 4
- Exposure: 3 (android only)
- Base: (5 × 4 × 3) / 10 = 6.0
- STRIDE: spoofing + elevation → 6.0 × 1.15 = 6.9
- **Rating: P2**

### WTM_024 Firmware downgrade

- Impact: 5
- Likelihood: 1 (very_low)
- Exposure: 2
- Base: (5 × 1 × 2) / 10 = 1.0
- **Rating: P3**

## Residual risk acceptance

Document accepted residual risk when:

1. Mitigation reduces likelihood by 2+ levels
2. Exposure limited via feature flag
3. Compensating control (e.g. hardware wallet required) enforced

Template:

```markdown
### WTM_XXX residual risk
- Original score: 45 (P1)
- Mitigation: [description]
- Residual likelihood: low
- Residual score: 8 (P3)
- Accepted by: [name] on [date]
```

## Integration with development workflow

| Phase | Scoring activity |
|-------|------------------|
| Design | Preliminary STRIDE pass, estimate exposure |
| Implementation | Map code paths to WTM ids |
| QA | Test cases for P0 and P1 threats |
| Release | No open P0; P1 requires sign off |
| Incident | Add new WTM entry; recalculate scores |

## Catalog maintenance

When adding catalog entries under `catalog/`:

1. Assign next WTM id
2. Complete all schema required fields
3. Calculate score using this model
4. Add row to `matrix/STRIDE_analysis.md` coverage table
