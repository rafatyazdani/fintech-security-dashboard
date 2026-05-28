# FinTech Security ROI Dashboard

A self-contained, single-file HTML dashboard for quantifying security investment value in financial services. No build step, no backend, no dependencies beyond a CDN-loaded Chart.js.

Open `index.html` in any modern browser.

---

## Purpose

Security budgets in FinTech are routinely challenged by finance and the board. This tool translates security posture inputs into the financial language that secures and defends those budgets:

- How much fraud are we exposed to, and how much are we preventing?
- What is our regulatory tail risk under GDPR, PSD2, DORA, and PSR?
- What would a breach cost us?
- What is the ROI of our current security spend, and what does toggling a specific control do to that number?

It is designed for use in CISO presentations, budget cycles, risk committee reporting, and security programme business cases.

---

## Features

### Inputs

| Input | Description |
|---|---|
| Business Segment | Payments/PSP · Lending/BNPL · Neobank · Crypto — drives fraud rates, regulatory multipliers, breach severity, and control lists |
| Total Payment Volume (TPV) | Annual volume in € billions |
| Annual Security Spend | Total security programme budget in € millions |
| Headcount | Used to estimate customer record exposure for breach cost |
| EU Transaction Share | Scales GDPR and PSD2/DORA regulatory exposure |
| UK Transaction Share | Scales PSR APP reimbursement liability |
| Control Coverage % | Quality of control implementation; degrades effective fraud reduction |
| Model Currency (days) | Days since last ML model retrain; introduces detection decay above 14 days |
| False Decline Rate | % of legitimate transactions blocked; added as a cost against ROI |

### Output Cards

| Card | What it shows |
|---|---|
| Gross Fraud Exposure | Segment baseline fraud rate × TPV; net figure after controls shown as sub-line |
| Regulatory Tail Risk | Probability-weighted expected annual loss across GDPR, PSD2, DORA, and PSR — broken out by component |
| Breach Cost Estimate | IBM X-Force 2024 formula: €4.88M base + €195/record × estimated records, with segment severity multiplier |
| Security ROI | (Risk protected − total cost) / total cost; total cost = security spend + false decline opportunity cost |

### ROI Breakdown

Horizontal bar chart and labelled bar rows showing each cost and benefit component side by side:

- Security Spend (cost)
- False Decline Cost (cost)
- Fraud Prevention Value
- Regulatory Risk Reduction
- Breach Cost Avoidance
- Total Risk Value Protected

### Fraud Control Deep-Dive

Segment-specific control lists (12–13 controls per segment) with:

- **Interactive checkboxes** — toggling a control immediately recalculates fraud prevention value and ROI
- **Multiplicative reduction model** — each control reduces residual fraud independently; combined reduction is capped at 78% to reflect real-world overlap and adversarial adaptation
- **Horizontal bar chart** — active controls shown at full opacity by tier colour; inactive controls greyed out
- **Prioritisation table** — ranked by marginal reduction %, with Must-Have / High / Medium tier badges

Control lists by segment:

| Segment | Segment-specific controls |
|---|---|
| Payments/PSP | 3DS2/SCA, APP Fraud Detection, Chargeback Prevention (Ethoca), AVS |
| Lending/BNPL | Synthetic Identity Detection, Open Banking/Income Verify, Credit Bureau Fraud Score, Employment Verification |
| Neobank | APP Fraud Detection, Liveness/Deepfake Detection, Mule Account Detection, DORA ICT Resilience Controls |
| Crypto | On-Chain Analytics (AML/CFT), FATF Travel Rule, Cold Wallet/HSM Key Management, Wash Trading Detection, Smart Contract Audit |

### Fraud KPI Reference Framework

24 KPIs across four measurement layers. Status dots (green / amber / red) are **live** — they update as you adjust inputs, not hardcoded benchmarks:

| Layer | KPIs | Driven by |
|---|---|---|
| Detection | FDR, FPR, Precision, Recall, Alert-to-Case Ratio, TTD | Coverage %, model age |
| Friction | Step-Up Auth Rate, False Decline Rate, Abandonment, Completion, Latency P95, NPS Delta | False decline slider, coverage % |
| Financial | Fraud Loss Rate, Chargeback Rate, Cost/Case, Recovery Rate, OpEx Ratio, Regulatory Penalties | Net fraud bps, regTail magnitude |
| Control Health | Coverage, Model Staleness, Rule Effectiveness, SLA, Data Quality, Audit Pass Rate | Coverage %, model age |

---

## Methodology

### Fraud Exposure

```
Base fraud rate (bps of TPV) — segment-specific, sourced from Nilson Report 2024:
  Payments/PSP:   6 bps
  Lending/BNPL:  15 bps
  Neobank:       10 bps
  Crypto:        20 bps

Fraud reduction (multiplicative model):
  residual = Π (1 − pct_i) for each active control i
  ctrlReduction = min(0.78, 1 − residual)

Effective coverage:
  modelDecay = min(0.25, max(0, (modelAgeDays − 14) × 0.003))
  effCov = coverage × (1 − modelDecay)

Net fraud saved = grossFraud × ctrlReduction × effCov
```

### False Decline Cost

```
Segment transaction margin:
  Payments: 0.8%  Lending: 2.5%  Neobank: 1.5%  Crypto: 0.6%

falseDeclineCost = TPV × falseDeclineRate × segmentMargin
```

This is added to the ROI denominator: total cost = security spend + false decline cost.

### Regulatory Tail Risk

```
GDPR:
  max fine = max(€20M, 4% × TPV)
  P(enforcement) = 5% × (1 − coverage × 0.6) × euShare
  expected = maxFine × probability

PSD2 / sector-specific:
  potential fine = TPV × euShare × 2% (PSD2 entities) or 1% (non-PSD2)
  P(enforcement) = 10% × (1 − coverage × 0.5) × euShare

DORA (all segments, Jan 2025):
  daily fine = min(€10M, TPV/365 × 1%)
  P(incident) = 12% × (1 − coverage × 0.7) × euPresenceFactor
  expected = dailyFine × 5 days × probability

PSR APP Reimbursement (UK, Oct 2024 — Payments/PSP and Neobank only):
  APP fraud rate: 3 bps (Payments), 4 bps (Neobank)
  PSP liability = TPV × ukShare × appFraudRate × 50%

regTail = (GDPR + PSD2 + DORA + PSR) × segmentRegulatoryMultiplier
```

### Breach Cost

```
Source: IBM X-Force Cost of a Data Breach 2024 (financial sector)
  base cost:       €4.88M
  per-record cost: €195
  record estimate: headcount × 300

rawBreachCost = (4.88M + records × 195) × segmentBreachMultiplier
breachProbability = 15% × (1 − coverage × 0.5)
expectedAnnualBreach = rawBreachCost × breachProbability
```

Segment breach multipliers reflect sector-specific incident severity: Neobank 2.0×, Lending 1.8×, Crypto 1.6×, Payments 1.5×.

### Security ROI

```
Total risk protected = fraudSaved + (regTail × coverage × 0.7) + (expBreach × coverage × 0.6)
Total cost           = securitySpend + falseDeclineCost
ROI                  = (totalProtected − totalCost) / totalCost × 100
Return per €1 spent  = totalProtected / totalCost
```

---

## Data Sources

| Source | Used for |
|---|---|
| Nilson Report 2024 | Segment baseline fraud rates (bps of TPV) |
| IBM X-Force Cost of a Data Breach 2024 | Breach base cost (€4.88M), per-record cost (€195 financial sector) |
| ECB Payment Statistics / PSSR | Payment volume benchmarks |
| EBA Guidelines on ICT and Security Risk | Control framework reference |
| ENISA Threat Landscape 2024 | Segment threat taxonomy |
| GDPR (Regulation EU 2016/679) | Fine structure: max(€20M, 4% global turnover) |
| PSD2 (Directive EU 2015/2366) | SCA requirements, enforcement probability |
| DORA (Regulation EU 2022/2554) | ICT risk fine structure; mandatory from 17 Jan 2025 |
| PSR APP Fraud Reimbursement Policy | Mandatory 50/50 liability split; UK only, effective Oct 2024 |
| AML6 / FATF Recommendations | Crypto segment regulatory multiplier |

---

## Limitations

All figures are **indicative estimates** for planning and business case purposes. They are not actuarial valuations.

- Regulatory fine probabilities are modelled; actual enforcement depends on supervisory authority, jurisdiction, incident severity, and cooperation.
- The breach cost formula uses sector averages. Specific incidents vary by data type, geography, and response maturity.
- Control reduction percentages are marginal contributions under a multiplicative model. Real-world control effectiveness depends on implementation quality, attacker sophistication, and data availability.
- The 78% cap on combined fraud reduction reflects realistic programme limits; a well-run fraud operation targeting 85%+ FDR with residual leakage is achievable but not guaranteed by control selection alone.
- DORA applies to entities in scope under Article 2; verify applicability with legal counsel.
- PSR reimbursement applies to UK-authorised PSPs only. Maximum reimbursement per claim is £85,000.

---

## Usage Tips

**For a board / audit committee pack** — set inputs to reflect actual TPV and security spend, then screenshot the four metric cards and the ROI breakdown bar. The regulatory card now shows GDPR, PSD2, DORA, and PSR as separate line items, which maps directly to risk register entries.

**For a budget justification** — use the controls section. Check only the controls currently implemented. Observe the ROI and fraud prevention figures. Then check additional controls in the proposed roadmap to show the incremental improvement. The delta in the ROI card is the financial case for the next tranche of investment.

**For a segment comparison** — run the same TPV and spend figures across all four segment tabs. Differences in output reflect structural risk differences in the business model, not just control choices.

**For a false decline conversation** — use the false decline slider to show finance that tightening controls beyond a threshold has a measurable revenue cost. At 2% false decline on €5B TPV in a lending business, the opportunity cost (~€2.5M) is visible in both the breakdown bar and the ROI denominator.

---

## File Structure

```
fintech-security-dashboard/
└── index.html      # Complete dashboard — HTML, CSS, and JS in one file
```

No build tooling, no npm, no framework. Chart.js is loaded from jsDelivr CDN (`chart.js@4.4.0`). The dashboard requires an internet connection only for that CDN request; all calculation logic runs locally in the browser.

---

## Related artifacts in the portfolio

- This is the **FinTech-segment-specific** version (Payments / Lending / Neobank / Crypto), with DORA, PSR APP, PSD2, and AML6 regulatory modelling built in. For the **industry-agnostic** board view see [`cyber-risk-dashboard`](https://github.com/rafatyazdani/cyber-risk-dashboard).
- For the **regulatory remediation companion** (AI-powered analysis of audit findings against the same regs), see [`compliance-copilot`](https://github.com/rafatyazdani/compliance-copilot).
- For the **multi-client consulting platform** with FAIR Monte Carlo, PDF export, and consultant views, see [`CRQ-Dashboard`](https://github.com/rafatyazdani/CRQ-Dashboard).
- Methodology source: **CRQ-F Framework** Phase 5 — Board Risk P&L (segment-specialized).

---

**Built by Rafat Yazdani** — CPA · CISSP · CISA · CCSK · AWS Certified
Security Strategy & Cyber Risk Quantification — Accenture Security

> Methodology backbone: **CRQ-F Framework** (Cyber Risk Quantification — Financial).
> Portfolio overview: [github.com/rafatyazdani](https://github.com/rafatyazdani) — Methodology · Quantification math · Consulting products · Governance & maturity · Operations · Board output.
