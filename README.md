# investment-calculator

Investment projection tool with income-driven contributions, salary overrides, and fee drag modeling. Deployed as a PWA at **https://josiah-d.github.io/investment-calculator/**.

## What it does

Monthly contributions are derived from income — never entered directly. Income compounds annually and can be anchored at specific ages via salary overrides. Two optional fee layers model the real cost of managed accounts: an AUM advisory fee (tiered, marginal) and fund expense ratios (blended across allocations).

## Usage

Open the URL in any browser. On iOS Safari, use Share → Add to Home Screen to install as a PWA.

To share a specific scenario: configure your inputs, hit **Share** — a URL encoding the full state is copied to your clipboard. Anyone who opens that link sees your exact scenario.

Settings auto-save to localStorage and persist across sessions in the same browser.

## Inputs

| Field | Description |
|---|---|
| Current Age | Your age today |
| Retirement Age | Target retirement age |
| Current Annual Income | Starting salary |
| Annual Income Growth | % rate income compounds each year |
| Contribution Rate | % of income invested; drives monthly contribution |
| Initial Balance | Existing savings at start |
| Expected Return | Gross annual return rate, compounded monthly |

**Monthly contribution is a calculated field** — `(income × rate%) ÷ 12`. Never entered directly.

## Salary Overrides

Pin a salary at a specific age. Growth resumes from that anchor the following year. Useful for non-linear jumps — job change, promotion, career shift. Multiple overrides are supported and processed in age order.

## AUM Advisory Fee (optional)

Tiered, marginal fee applied to the balance — like income tax, each dollar sits in its bracket. Pre-populated with a 6-tier schedule; fully editable.

**Billing frequency:** Quarterly (default), Monthly, or Annually. Quarterly and monthly billing deducts the fee *before* that period compounds, matching standard AUM billing practice (in advance, on prior period-end value).

Default tier schedule:

| Up To | Rate |
|---|---|
| $100,000 | 1.95% |
| $250,000 | 1.25% |
| $1,000,000 | 1.00% |
| $2,500,000 | 0.85% |
| $10,000,000 | 0.65% |
| Above | 0.50% |

## Fund Expense Ratios (optional)

Enter funds by name, allocation %, and expense ratio %. The blended weighted average reduces the effective return rate on the adjusted balance simulation. Independent of the AUM fee — both layers can be enabled simultaneously.

## Math

**Income each year:**
```
income[age] = override.salary              // if override exists
income[age] = income[age-1] × (1 + g/100) // otherwise
```

**Monthly contribution:**
```
monthly = (income[age] × contribRate / 100) / 12
```

**Gross balance (monthly compounding):**
```
monthlyReturn = annualReturn / 100 / 12
balance = balance × (1 + monthlyReturn) + monthly
```

**AUM fee (marginal, per tier):**
```
fee = Σ (min(balance, tier.ceiling) - tier.floor) × tier.rate / 100
```

Applied quarterly (default): `quarterlyFee = annualFee(balance) / 4`, deducted at the start of each quarter before compounding.

**Adjusted balance return rate:**
```
monthlyNetReturn = (annualReturn - blendedExpenseRatio) / 100 / 12
```

**Total drag:**
```
drag = grossBalance - adjustedBalance
```

## Files

| File | Purpose |
|---|---|
| `index.html` | Full application — single file, no build step |
| `manifest.json` | PWA manifest (name, icons, display mode) |
| `sw.js` | Service worker — caches assets for offline use |
| `icon.svg` | App icon |

## Known Limitations

- Two salary overrides at the same age — last write wins; no validation
- SVG icons lack full maskable support; Android adaptive icons may crop
- Share link uses `btoa`/`atob` — state is encoded, not encrypted
