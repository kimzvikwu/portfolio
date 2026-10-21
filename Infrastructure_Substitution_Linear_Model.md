# Infrastructure Substitution in a Business Continuity Scenario

This is a patent I filed at IBM. I am unsure if their architecture committee reviewed it to file with the patent office or not. I was able to reproduce this from memory.

When you have Terraform or other Infrastructure as Code (IaC) scripts to handle a failover event such as an earthquake you won't be sure if your code will work on that day due to many factors not limited to availability zones offline for maintenance or exhaustion of infrastructure resources such as XLarge EC2 instances on AWS in us-east-1 for example. Generally companies test business continuity at best once a year. So the system drift can occur through the year such as a cloud provider retiring an instance type. When you need your script to gracefully fail over from one region to another you are not guaranteed it will do so.

Hashicorp's Drift Detection will only alert you if your failover is running on standby as a Hot, Medium, or Cold instance. But many companies will not have a standby region and may run stale scripts in a disaster scenario. Also, when companies do exercise their fail over procedures they don't plan on having to compete for a limited amount of resources as the other companies in their region. It is very plausible that all us-west-1 companies may fail over to us-east-1 at the same time. So it is easy to conclude that several of these companies may exhaust the limited pool of infrastructure resources available in us-east-1.

Also, companies don't really care which region they fail over to so long as it resides in a country of their choosing. So if half really failed over to us-east-2 it would still be considered a successful failover procedure. While an XXLarge EC2 instance is more expensive to run, a company would be happy to trade that for a successful failover scenario which called for XLarge EC2 instances as well. So the ability to negotiate real time with the cloud provider for substitute resources would also be considered a success.

To represent infrastructure substitution you can employ the use of a linear algebraic formula which employs the use of factors and weights to enable a company to state their preferences. This is like turning the dials of a radio to fine tune out the fuzz to get a clear signal. You can represent the formula like this which mirrors the multi-factor models used to separate systematic risk from residual risk in equity portfolios. This is a multi-factor model which can be used with infrastructure attributes to calculate the level of risk for failing to gracefully fail over to another region.

## Linear Multi-Factor Failover Risk Model

### 1. The formula

For each candidate substitute *j* (a region and instance-type pair):

```
R_j = β₀ + Σ_k w_k · x_jk + ε_j
```

| Term | Meaning |
|---|---|
| **R_j** | Risk that failover to candidate *j* fails to complete gracefully (0–1) |
| **β₀** | Baseline risk present in any failover, regardless of target |
| **x_jk** | Candidate *j*'s exposure to risk factor *k*, normalized 0–1 (higher means riskier) |
| **w_k** | Weight on factor *k*. These are the "dials" a company turns to state its preferences, and they sum to 1. |
| **ε_j** | Residual: risk the factors don't explain, measured as the gap between predicted and actual outcomes in DR tests |

**Decision rule:** pick the candidate with the lowest R_j that passes your hard constraints, and recompute in real time as factor values change.

### 2. Factors

| # | Factor | Exposure x (0 = low risk, 1 = high) |
|---|---|---|
| x₁ | **Capacity contention** | 1 − (available capacity ÷ required), including demand from other companies failing over to the same region. AWS Spot placement scores are one real-time proxy. |
| x₂ | **Substitution distance** | 0 = exact instance type; 0.3 = same family, larger size; 0.6 = different family; 1 = unsupported |
| x₃ | **IaC drift** | Days since last successful failover test ÷ 365, or drifted resources ÷ total resources |
| x₄ | **Correlation with primary** | Probability the target is affected by the same event (shared control plane, IdP, seismic zone) |
| x₅ | **Cost premium** | (Substitute cost − planned cost) ÷ planned cost, capped at 1 |
| x₆ | **Latency penalty** | Added latency ÷ tolerated latency, capped at 1 |

**Hard constraints, kept outside the formula:** country or data residency, and compliance boundary. A linear model can't express a veto, so these act as filters, not weights. A non-compliant candidate is excluded rather than scored.

### 3. Worked example

**Scenario:** An earthquake takes out us-west-1. The failover script calls for XLarge instances in us-east-1, and the script was last tested 6 months ago. The company's preferences are below (β₀ = 0.02):

| Weight | w₁ Capacity | w₂ Substitution | w₃ Drift | w₄ Correlation | w₅ Cost | w₆ Latency |
|---|---|---|---|---|---|---|
| Value | 0.30 | 0.15 | 0.20 | 0.20 | 0.05 | 0.10 |

**Candidates:**

| Candidate | x₁ | x₂ | x₃ | x₄ | x₅ | x₆ | **R_j** |
|---|---|---|---|---|---|---|---|
| A: us-east-1, XLarge (the plan) | 0.70 | 0 | 0.50 | 0.15 | 0 | 0.60 | **0.420** |
| B: us-east-1, XXLarge | 0.30 | 0.30 | 0.50 | 0.15 | 1.00 | 0.60 | **0.395** |
| C: us-east-2, XLarge | 0.20 | 0 | 0.50 | 0.10 | 0.05 | 0.50 | **0.253** |
| D: us-west-2, XLarge | 0.50 | 0 | 0.50 | 0.25 | 0 | 0.10 | **0.330** |

Sample calculation for C:

```
0.02 + (0.30·0.20) + (0.15·0) + (0.20·0.50) + (0.20·0.10) + (0.05·0.05) + (0.10·0.50) = 0.253
```

**Result:** the scripted plan (A) is the *riskiest* option, because every company in us-west-1 is converging on us-east-1 XLarge capacity. Moving to us-east-2 cuts failover risk by about 40%. Paying for XXLarge in us-east-1 (B) barely helps, since the cost premium offsets the contention gain.

### 4. Reading the model

- **Factor contribution shows which dial matters.** In A, capacity contention contributes 0.21 of the 0.42 total. That is half the risk, and it's the case for real-time capacity negotiation.
- **Drift acts like a market factor.** It adds 0.10 to *every* candidate, so switching regions can't diversify it away. Only more frequent testing reduces it, which is the argument against annual DR tests.
- **The weights encode business preferences.** A cost-sensitive company raises w₅; a latency-sensitive trading platform raises w₆. The same factor data yields different rankings for different companies.

### 5. Calibrating weights

- **With no history,** set weights by expert judgment, like the preferences above.
- **With DR test or game-day history,** estimate the weights by regressing test outcomes on factor exposures. Logistic regression, P(fail) = σ(β₀ + Σβ_k·x_k), is better than linear when the output must be a true probability.
- **The residual ε** comes from comparing predicted risk to actual test failures. A large, persistent ε means you're missing a factor.
- **Check factor independence.** Ensure that you have low correlation for each factor in your model. Otherwise, if they cross correlate it will reduce the signal from the formula. Contention and latency can both rise with region popularity. Run a correlation matrix or VIF test, and merge or drop factors that are highly correlated (VIF > 5), or they'll double-count risk.

### 6. Mapping to the equity factor model

| Equity model: r = α + Σβ·f + ε | Failover model |
|---|---|
| Factor exposures (β) | Factor exposures x_jk |
| Factor returns (f) | Preference weights w_k |
| α (intercept) | β₀, baseline failover risk |
| ε (specific risk) | ε, residual risk the factors don't explain |
| Portfolio optimization | Selecting the lowest-risk substitute subject to constraints |

## Risk of Highly Correlated Factors

These are resources that look redundant on paper but fail together because they share a hidden dependency. Each would get a high correlation exposure (x₄) in the model.

**Shared control plane**

- **Multi-AZ in one region:** AZs have separate power and networking, but share the regional control plane. The us-east-1 outages (Dec 2021, Oct 2025) took down workloads across all AZs because the regional APIs, IAM, and DynamoDB/STS endpoints failed together.
- **Multi-region on one cloud:** Better than multi-AZ, but global services (IAM, Route 53 control plane, CloudFront config) are single points. Correlation is lower but not near zero.

**Shared identity**

- **Standby in a second cloud that still authenticates through the same Okta or Entra tenant:** If the IdP is down or compromised, you can't log in to activate the standby. This is one of the most common gaps in DR plans.
- **Break-glass accounts federated through the same SSO they're meant to bypass.**

**Shared keys and secrets**

- **Replicated databases encrypted with the same KMS key or HSM cluster:** Lose or revoke the key and both copies are unreadable.
- **Secrets manager in one region feeding both primary and standby.**

**Shared DNS and certificates**

- **Primary and failover endpoints behind the same DNS provider:** The Dyn DDoS (2016) made healthy infrastructure unreachable.
- **Certificates from the same CA or automation pipeline:** An expiry or revocation hits every site at once.

**Shared software and configuration**

- **Identical stacks deployed by the same pipeline:** A bad push replicates everywhere. CrowdStrike (July 2024) bricked Windows hosts across every region and provider at the same moment. Bad config in an active-active pair is the classic case of redundancy amplifying the failure.
- **Same OS image or base container:** A zero-day or a bad kernel patch hits all copies.

**Shared physical layer**

- **Two "diverse" ISP circuits in the same conduit:** One backhoe cuts both.
- **Two data centers on the same power substation or in the same flood or fire zone.**
- **Colo DR site within 30 miles of HQ:** The same earthquake or wildfire affects both. This matters for the Bay Area specifically.

**Shared people and process**

- **The same small on-call team runs both environments:** If they're unavailable, or the runbook lives in the system that's down (Confluence on the failed infra), activation fails.
- **Ransomware via shared admin credentials:** Attackers routinely target backups and DR because they're reachable from the same domain admin accounts. Your "substitute" gets encrypted alongside the primary.

**Shared vendor or supply chain**

- **Two SaaS vendors that both run on the same cloud region or CDN.**
- **SolarWinds-style compromise:** A trusted management tool with access to every environment.

**Rough calibration of correlation exposure (x₄)**

| Arrangement | Typical x₄ |
|---|---|
| Same AZ, different hosts | 0.6–0.9 |
| Multi-AZ, same region | 0.3–0.5 |
| Multi-region, same cloud, shared IdP/pipeline | 0.15–0.3 |
| Different cloud, shared IdP | 0.1–0.2 |
| Different cloud, separate IdP, keys, and pipeline | 0.01–0.05 |

**Bottom line:** most DR plans control for geography but miss logical correlation in identity, keys, deployment, and admin access. Those are where correlation risk actually lives. A useful exercise is to trace activation of the substitute step by step and list every system you touch. Anything that also serves the primary belongs in x₄.
