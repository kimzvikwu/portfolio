# A Formula to Calculate Agent Trustworthiness in Cortex AI

Another patent I participated in at IBM was the topic of agent trustworthiness. While one of the participants brought in the Cortex AI expertise, the other brought in his calculations on agent trustworthiness at any given enrichment level. So I won't reproduce any of their side that I may remember here. The one thing I contributed to is the formula. I decided the best formula is a 3D vector graph multi-factor model.

This is to allow each enrichment stage in Cortex AI to follow an algebraic multi-factor weighted model and then see how this particular stage relates — like the positive attraction of a magnet — to the next stage or not. So if one level is highly correlated to another, it is considered more trustworthy than uncorrelated level scores.

## The Multi-Factor Model in 3D

To map a multi-factor model onto a 3D coordinate system $(X, Y, Z)$, we restrict the formula to two risk factors ($f_1$ and $f_2$). The formula represents a **2D plane cutting through a 3D vector space**:

$$
R_i = R_f + \beta_{i,1} f_1 + \beta_{i,2} f_2 + e_i
$$

Where the 3D vector coordinates correspond to:

- **X-axis (Factor 1 Sensitivity):** $\beta_{i,1}$ — beta or loading on the first factor (e.g., the broader market)
- **Y-axis (Factor 2 Sensitivity):** $\beta_{i,2}$ — beta or loading on the second factor (e.g., size or value)
- **Z-axis (Expected Return):** $R_i$ — the total expected return of asset $i$

### Component Breakdown

- **$R_f$ (The Z-Intercept):** The risk-free rate. This is where the asset plane intersects the vertical Z-axis when an asset has zero exposure to both risk factors ($\beta_1 = 0$, $\beta_2 = 0$).
- **$\beta_{i,1}, \beta_{i,2}$ (The Directional Vectors):** The factor loadings. They act as vectors that dictate the tilt or slope of the asset return plane along the X and Y axes.
- **$f_1, f_2$ (Factor Risk Premia):** The expected premium or reward for taking on a unit of risk for each respective factor.
- **$e_i$ (Residual Error):** The idiosyncratic risk unique to that specific asset, represented visually as the vertical distance a real-world data point sits above or below the theoretical 3D factor plane.

### Visualizing the 3D Vector Space

In a 3D vector plot, individual portfolios or assets are plotted as points or vectors extending from the origin.

- An **aggressive market stock with no size tilt** would sit far out on the X-axis and high up on the Z-axis.
- A **small-cap stock neutral to the broader market** would sit far out on the Y-axis.
- The multi-factor formula establishes a **"Security Market Plane"** (instead of a Security Market Line), where all perfectly priced diversified assets must land.

### General Matrix/Vector Formula (for N Factors)

When moving beyond 3D visualization into hyper-dimensional space (more than two factors, like the Fama-French 3-factor or 5-factor models), the formula is written cleanly using linear algebra vector notation:

$$
\mathbf{R} = R_f \mathbf{1} + \mathbf{B}\mathbf{F} + \mathbf{E}
$$

Where:

- $\mathbf{R}$ is the vector of asset returns,
- $\mathbf{B}$ is the matrix of factor sensitivities (betas),
- $\mathbf{F}$ is the vector of factor risk premia, and
- $\mathbf{E}$ is the vector of residual errors.

---

## Applying the Model to Cortex AI Enrichment Layers

The above formula shows how assets contribute to overall risk. Now, instead of assets, apply this to the enrichment layers in the Cortex AI model.

Cortex (XSIAM / XDL) enrichment is grouped into distinct mechanisms:

1. **Enrichment at ingestion.** As data enters Cortex XDL, it's automatically enriched with threat intelligence, asset context, user behavior baselines, and environmental information. These are really four enrichment types in one sentence (2–5 below expand on them). [[paloaltonetworks]](https://www.paloaltonetworks.com/blog/security-operations/stopping-cross-domain-attacks-with-cortex-xdl-cortex-xsiam/)
2. **Threat intel / Unit 42.** Unit 42 Intel is a continuous feed integrated into the XSIAM data lake, running in the background for automated threat matching and incident scoring. WildFire artifacts and third-party feeds are added on top. [[paloaltonetworks]](https://live.paloaltonetworks.com/t5/cortex-xsiam-discussions/xsiam-threat-intelligence-management-module/td-p/1254103)
3. **Asset and vulnerability context.** The admin guide describes an "enrich asset" function that gets vulnerability information for an asset or group of assets in the organization. Xpanse/ASM adds outside-in attack surface data. [[scribd]](https://www.scribd.com/document/735696099/Cortex-Xsiam-Admin-Guide)
4. **Identity analytics.** Cortex XSIAM uses Identity Analytics to aggregate information on a user and displays insights about the user. [[scribd]](https://www.scribd.com/document/735696099/Cortex-Xsiam-Admin-Guide)
5. **Behavioral baselines.** User and entity behavior baselines feed the ML detection models.
6. **Normalization to a common schema.** Cortex XDL automatically translates data from hundreds of security tools into a common schema, enabling seamless correlation across previously incompatible systems. [[paloaltonetworks]](https://www.paloaltonetworks.com/blog/security-operations/stopping-cross-domain-attacks-with-cortex-xdl-cortex-xsiam/)
7. **Stitching across data sources.** XSIAM automatically stitches together endpoint, network, cloud, and identity data into unified event context. [[paloaltonetworks]](https://www.paloaltonetworks.com/cyberpedia/revolutionizing-soc-operations-with-ai-soc-solutions)
8. **Alert grouping and risk scoring.** With this enriched data, SmartGrouping links related alerts into comprehensive incidents, along with dynamic risk scoring. [[bakotech]](https://bakotech.com/wp-content/uploads/2026/01/cortex-xsiam.pdf)
9. **Playbook-driven indicator enrichment.** Here, enrichment means adding context (reputation, tags, related incidents) to indicators like IPs, domains, hashes, and URLs, automated through TIM jobs and playbooks. [[paloaltonetworks]](https://live.paloaltonetworks.com/t5/cortex-xsiam-discussions/xsiam-threat-intelligence-management-module/td-p/1254103)

The key design choice is to treat trust as something that propagates downstream. Each stage is scored on three axes, and a stage's effective trust can never exceed the weakest stage feeding it. Its permitted agent autonomy then depends on how much damage a wrong action would do.

### The Three Dimensions

**P: Provenance and integrity.** How trustworthy is the input?

- Is the source authenticated?
- Does it meet its freshness SLA?
- What share of the environment does it cover?
- Is it tamper-evident?
- Can an adversary influence it? This is the factor that matters most for agents: attacker-controlled fields like URLs, email bodies, and user-agent strings can poison any LLM step that reads them.

**F: Inference fidelity.** How reliable is the transformation?

- Is it deterministic or probabilistic?
- What are its precision and recall against labeled data?
- Is it explainable?
- How fast does it drift?

**C: Consequence.** What happens if an agent acts on a wrong enrichment?

- Is the action reversible?
- How large is the blast radius?
- How privileged is the target?
- What are the regulatory and contractual implications?

### Scoring Rules

P and F are scored 1–5, where higher is better. C is scored 1–5, where higher is more severe.

Effective trust:

$$
T_{eff} = \min(P,\ F,\ T_{eff} \text{ of upstream inputs})
$$

**Autonomy tiers:**

| Tier | Meaning |
|------|---------|
| A0 | Observe only |
| A1 | Recommend |
| A2 | Act with human approval |
| A3 | Act autonomously, reversible actions only |
| A4 | Act fully autonomously |

**How T_eff and C map to a tier:**

- If $T_{eff} \geq C$, the stage gets A3.
- If $T_{eff} = C - 1$, it gets A2.
- If $T_{eff} \leq C - 2$, it gets A1.
- If $T_{eff} = 1$, it gets A0.
- A4 is reserved for stages where $T_{eff} \geq 4$ and $C \leq 2$.

### Overlay on the 9 Stages

The stages are listed in pipeline order. The scores are illustrative defaults to calibrate against your own telemetry.

| # | Stage | P | F | C | T_eff | Max autonomy | Dominant failure mode |
|---|-------|---|---|---|-------|--------------|-----------------------|
| 1 | Normalization | 4 | 5 | 3 | 4 | A3 | Parser drift when vendor log formats change, silent field drops, timezone skew |
| 2 | Stitching | 4 | 4 | 3 | 4 | A3 | Wrong-entity attribution (DHCP/NAT, shared hosts) |
| 3 | Threat intel (Unit 42, WildFire, third-party) | 3 | 3 | 4 | 3 | A2 (A3 for first-party hash verdicts) | Aged indicators, shared infrastructure such as CDN IPs, low-quality feeds |
| 4 | Asset and vulnerability context | 3 | 4 | 3 | 3 | A3 | Stale CMDB ownership, ephemeral cloud assets, scanner coverage gaps |
| 5 | Identity analytics | 4 | 3 | 5 | 3 | A1 for privileged accounts, A3 for low-privilege session revocation | Peer-group and role misinference |
| 6 | Behavioral baselines | 3 | 2 | 4 | 2 | A1 | Baseline poisoning during the learning period, drift, low explainability |
| 7 | SmartGrouping and risk scoring | 2 (inherited) | 3 | 4 | 2 | A1 | Over- or under-merging incidents; false negatives hidden by de-prioritization |
| 8 | Indicator enrichment playbooks (TIM) | 3 | 3 | 5 | 2–3 | A1–A2 | External API integrity, prompt injection through attacker-controlled content |

### What the Model Exposes

1. **Trust degrades downstream.** Stage 7 sits at $T_{eff} = 2$ because it consumes the behavioral baselines. Yet that risk score is what decides what analysts see. The least trustworthy layer drives prioritization, which is the main finding to act on.
2. **Fix it by splitting the lineage.** Tag baseline-derived signals separately so grouping can carry two scores: a high-trust score from deterministic evidence and a low-trust score from anomaly evidence. Agents may then act autonomously only on the deterministic path.
3. **Auto-close is the real risk, not auto-block.** Wrongly blocking something is visible and reversible. Wrongly suppressing an incident is invisible. Any agent auto-closing at stage 7 needs random-sample audits and a measured false-negative rate.
4. **Use action granularity to move within the consequence axis.** For identity, revoking a session is C3 while disabling an account is C5. Breaking actions into smaller, reversible steps earns more autonomy without raising trust.
5. **Trust should decay over time.** Score P with a freshness factor so threat intel and asset context lose trust as they age, and recompute $T_{eff}$ continuously rather than once at design time.

> **Bottom line:** Let agents run autonomously at stages 1–4 with reversible actions only. Keep stages 5–8 at recommend or approve until you've split the deterministic and anomaly lineages and have measured precision and false-negative rates to justify promoting them.

---

## Star Trek 3D Chess Analogy

Like most "techies," we admire Star Trek for its hopeful view of the universe and a relatively peaceful Federation of interplanetary nations. And we can credit Spock for introducing us to 3D chess! For this example, 3D chess is an excellent way to illustrate how enrichment stages in Cortex work in concert to elevate risk during a cyber attack — and in some cases completely pivot the board and jump back up to a prior stage for more enrichment with another separate agent, perhaps on the target Internet Protocol (IP) address.

In the figure below, you have the knight at the top. Its single job is to enrich geolocation data and evaluate issues like impossible travel. When done, it passes its enriched data to the next dimension, to the bishop. The bishop enriches the source IP address to see if it knows its attacker. When it completes, it may pass its enriched data to the queen on its level, who may enrich more IP data such as the target — your customer, for example.

By applying the above formula to each separate agent doing its own independent enriching, we can create a sum of how trustworthy each agent is, as well as the sum of all the levels for many agents and the overall fidelity of the attack score. This experience is stored, and over time the agent becomes more or less trustworthy too.

![3D chess board illustrating agents enriching data across levels](image6.png)

*Figure: 3D chess analogy — image not included in this export; insert the original figure from the Google Doc.*
