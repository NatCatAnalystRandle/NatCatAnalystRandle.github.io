---
layout: page
title: "Building an Earthquake Catastrophe Risk Model"
permalink: /projects/seismic-catastrophe-risk-model/
---

**From USGS seismic sources to portfolio damage, insurance loss, reinsurance recovery, and catastrophe-risk metrics**

[View the source code on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)

---

## Project overview

Earthquake catastrophe-risk modeling requires a continuous connection between physical hazard, building damage, financial loss, insurance recovery, and reinsurance risk transfer.

I built this project to create that connection from end to end.

The model begins with official U.S. Geological Survey seismic-source information and progresses through rupture occurrence, a long stochastic event catalog, site-level ground motion, structural and nonstructural damage, ground-up repair loss, insurance recovery, occurrence excess-of-loss reinsurance, and portfolio risk metrics.

The current release is **Phase 1**, a fully validated baseline without spatial correlation among within-event site residuals.

### Project at a glance

| Item | Baseline value |
|---|---:|
| Portfolio location | Seaside, Oregon |
| Buildings | 470 |
| Portfolio replacement value | $384,236,605 |
| Monetary basis | 2022 USD |
| Stochastic catalog duration | 2,000,000 years |
| Simulated earthquake occurrences | 10,630 |
| Ground-up AAL | $195,922 |
| Gross insured AAL | $122,980 |
| Ceded AAL | $63,677 |
| Net retained AAL | $59,303 |

---

## Why I built it

Many engineering analyses end at hazard or physical damage.

Many insurance analyses begin with financial loss tables.

I wanted to build the chain between the two.

A catastrophe-risk model has to answer a connected set of questions:

1. Which earthquakes can affect the portfolio?
2. How often does each rupture occur?
3. Which earthquakes occur in each simulated year?
4. How much shaking reaches each insured location?
5. How do buildings transition into damage states?
6. How does physical damage translate into repair cost?
7. How do deductibles and limits change insured loss?
8. How much loss is transferred to reinsurance?
9. What do the resulting AAL, AEP, OEP, and PML distributions look like?

That full chain is important because the largest physical loss is not necessarily the largest retained insurer loss, and the sources dominating ground-up risk do not necessarily contribute to ceded risk in the same proportions.

---

## End-to-end catastrophe-risk workflow

**USGS NSHM 2018**

↓  

**Rupture-level annual occurrence rates**

↓  

**2,000,000-year stochastic event catalog**

↓  

**Rupture-to-site distances**

↓  

**PGA and SA(0.4) ground-motion fields**

↓  

**Structural and nonstructural damage states**

↓  

**Ground-up repair loss**

↓  

**Gross insured and uninsured loss**

↓  

**Occurrence XoL ceded and retained loss**

↓  

**AAL · AEP · OEP · PML**

The implementation is organized into seven sequential notebooks. Each notebook validates the accepted output from the preceding stage before performing the next transformation.

---

## 1. Hazard model and rupture occurrence

The project uses the **2018 USGS National Seismic Hazard Model for the conterminous United States**, with the source model pinned to:

`nshm-conus` release `5.2.4`

The portfolio demonstration focuses on two source families relevant to coastal Oregon:

- Cascadia subduction interface earthquakes
- Oregon intraslab earthquakes

Rupture-level annual occurrence rates are expanded using the official USGS software stack, including `nshmp-haz 2.6.5`, JDK 11, and the pinned Gradle wrapper.

### A technically important modeling decision

One of the most important parts of this stage was distinguishing:

- epistemic logic-tree weights
- physical source occurrence
- source-scale factors
- magnitude-frequency distributions
- rupture-level annual rates

These quantities cannot simply be multiplied or added without respecting the structure of the USGS source model.

For example, mutually exclusive epistemic alternatives should not be interpreted as independent physical earthquake sources. Similarly, source-scale factors are retained separately from epistemic branch probabilities.

This distinction was explicitly validated before any stochastic events were generated.

---

## 2. Building the stochastic event catalog

Rupture occurrence rates are converted into a **2,000,000-year annual stochastic catalog**.

The resulting catalog contains:

| Catalog measure | Value |
|---|---:|
| Total occurrences | 10,630 |
| Cascadia interface occurrences | 6,680 |
| Oregon intraslab occurrences | 3,950 |
| Years containing at least one earthquake | 10,593 |
| Zero-event years | 1,989,407 |
| Multiple-event years | 36 |
| Maximum events in one year | 3 |
| Mean event rate | 0.005315 per year |

The implied mean time between modeled events is approximately 188 years.

### Why zero-event years matter

The annual structure of the catalog is preserved explicitly.

AAL is calculated over **all two million years**, including the 1,989,407 years with zero loss.

Similarly, AEP and OEP are annual exceedance probabilities, so removing zero-event years would condition the results on an earthquake occurring and would substantially overstate annual risk.

Multiple-event years are also retained because aggregate annual loss can differ from the largest individual occurrence loss in the same year.

---

## 3. Ground-motion simulation

For every simulated earthquake occurrence, ground motion is calculated at all 470 building sites.

The production model uses the **Parker NGA-Subduction ground-motion implementation** through the official USGS software stack.

The two primary intensity measures are:

- PGA
- SA(0.4 s)

For occurrence \(e\) and site \(s\), the aleatory model is represented conceptually as:

\[
\ln(IM_{e,s})
=
\mu_{e,s}
+
\tau_{e,s}\eta_e
+
\phi_{e,s}\epsilon_{e,s}
\]

where:

- \(\mu\) is the median logarithmic ground-motion prediction
- \(\eta_e\) is the between-event residual
- \(\epsilon_{e,s}\) is the within-event residual
- \(\tau\) is between-event variability
- \(\phi\) is within-event variability

The completed production table contains:

**10,630 occurrences × 470 sites = 4,996,100 occurrence-site rows.**

---

## 4. The Phase 1 dependence model

Phase 1 deliberately does **not** include spatial correlation among within-event site residuals.

It is not intended to represent spatial independence as a realistic physical assumption.

It is an experimental baseline.

The dependence structure is:

| Dependence feature | Phase 1 treatment |
|---|---|
| Between-event variability | One occurrence-level residual shared across the portfolio |
| Within-event residuals across different sites | Conditionally independent |
| PGA and SA(0.4) residuals at the same site | Cross-correlated |
| PGA-to-SA(0.4) correlation | Approximately 0.732 |
| Spatial correlation | Not applied |
| Event and site random streams | Frozen for Phase 2 |

The shared between-event residual gives each earthquake event portfolio-wide coherence. An event that is systematically stronger than its median prediction tends to be stronger across the entire portfolio.

The conditionally independent within-event residuals then provide the controlled reference case for the spatial-correlation extension.

This distinction is important: **the baseline is not fully independent ground motion.** It contains shared event-level variability but excludes spatial dependence in the within-event site residuals.

---

## 5. Damage modeling

Ground motion is translated into three separate building repair-loss components:

1. structural damage
2. nonstructural drift-sensitive damage
3. nonstructural acceleration-sensitive damage

The portfolio consists of W2 buildings, with HAZUS-style lognormal fragility relationships used to calculate the probability of reaching or exceeding each damage-state threshold.

The damage states are:

- None
- Slight
- Moderate
- Extensive
- Complete

For a damage threshold \(k\), the fragility relationship has the form:

\[
P(DS \geq k \mid Sa)
=
\Phi
\left[
\frac{\ln(Sa)-\ln(\theta_k)}
{\beta_k}
\right]
\]

where \(\theta_k\) is the median intensity and \(\beta_k\) is the lognormal dispersion.

A deterministic random value is assigned to each occurrence-site-component combination so that structural, drift-sensitive, and acceleration-sensitive damage can be sampled reproducibly.

### Direct SA(0.4) approximation

The structural and nonstructural models use a documented direct-SA(0.4) approximation.

For structural fragility, source spectral-displacement medians are converted to equivalent SA(0.4) values at a fixed 0.40-second period.

This is a deliberate modeling simplification and **not the full HAZUS capacity-spectrum procedure**.

The approximation is retained explicitly in the project assumptions rather than being hidden inside the calculation.

---

## 6. From damage to ground-up loss

Each sampled damage state is mapped to a component-specific repair-cost ratio.

The total ground-up building repair loss is:

\[
L_{\text{ground-up}}
=
L_{\text{structural}}
+
L_{\text{NS drift}}
+
L_{\text{NS acceleration}}
\]

The portfolio model therefore preserves the engineering breakdown of loss before applying financial terms.

This makes it possible to ask not only **how much** the portfolio loses, but **which physical damage mechanisms are driving the loss**.

---

## 7. Insurance transformation

The baseline insurance policy is synthetic and designed to demonstrate the financial transformation transparently.

| Policy term | Baseline assumption |
|---|---:|
| Insurance take-up | 100% |
| Covered repair-cost share | 100% |
| Deductible | 10% of building replacement value |
| Policy limit | 100% of replacement value |
| Coinsurance | 100% |
| Application | Per building, per occurrence |

Even with 100% take-up and full coverage above the deductible, insured loss is materially lower than ground-up loss because the 10% building-level deductible absorbs many small and moderate losses.

---

## 8. Occurrence excess-of-loss reinsurance

The project then applies a synthetic **occurrence excess-of-loss reinsurance layer** to gross insured occurrence loss.

The layer is calibrated from the modeled gross-insured OEP curve.

| Reinsurance term | Baseline value |
|---|---:|
| Attachment basis | 500-year gross insured OEP |
| Attachment | $18,811,084 |
| Exhaustion basis | 2,500-year gross insured OEP |
| Exhaustion | $80,649,067 |
| Occurrence limit | $61,837,983 |
| Reinsurer participation | 100% |
| Annual aggregate cap | None |
| Reinstatement restriction | None |

For gross insured occurrence loss \(L\):

\[
L_{\text{ceded}}
=
\min
\left[
\max(L-A,0),
U
\right]
\]

where:

- \(A\) is the attachment
- \(U\) is the occurrence layer limit

Net retained loss is:

\[
L_{\text{retained}}
=
L-L_{\text{ceded}}
\]

This makes the effect of risk transfer directly observable across the loss distribution.

---

## 9. Average annual loss transformation

The completed baseline produces the following average annual loss flow:

| Loss view | AAL | Interpretation |
|---|---:|---|
| Ground-up | $195,922.45 | Physical repair loss before insurance |
| Gross insured | $122,979.56 | Insurer liability after policy terms |
| Uninsured | $72,942.89 | Loss absorbed below or outside insurance recovery |
| Ceded | $63,676.60 | Average annual loss transferred to reinsurance |
| Net retained | $59,302.96 | Insurer loss remaining after reinsurance |

![Baseline average annual loss flow](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_aal_loss_flow.png)

The accounting identities reconcile within numerical tolerance:

\[
\text{Ground-up AAL}
=
\text{Gross insured AAL}
+
\text{Uninsured AAL}
\]

and

\[
\text{Gross insured AAL}
=
\text{Ceded AAL}
+
\text{Net retained AAL}
\]

### What the financial transformation tells us

The policy recovers approximately **62.77% of ground-up AAL**.

Approximately **37.23% remains uninsured**, despite 100% take-up, because the building-level deductible absorbs a substantial amount of lower-level repair loss.

The occurrence XoL layer then transfers approximately **51.78% of gross insured AAL** to the reinsurer.

The remaining **48.22% of gross insured AAL** stays with the insurer as net retained loss.

---

## 10. What actually drives expected loss?

One of the strongest results of the analysis comes from decomposing ground-up AAL by damage component.

| Damage component | Share of ground-up AAL |
|---|---:|
| Structural | 8.74% |
| Nonstructural drift-sensitive | 15.01% |
| Nonstructural acceleration-sensitive | **76.25%** |

The expected portfolio loss is therefore dominated by **acceleration-sensitive nonstructural damage**, not structural damage.

This is important from an engineering-risk perspective.

Structural performance receives substantial attention because of its relationship with safety and collapse. But an insurance portfolio is financially exposed to repair costs across all damage components.

For this portfolio and consequence model, nonstructural acceleration-sensitive damage is the primary driver of expected repair loss.

---

## 11. Earthquake-source concentration

The portfolio is also strongly concentrated by earthquake source.

Cascadia interface earthquakes contribute approximately:

- **90.60% of ground-up AAL**
- **95.49% of ceded AAL**

![Average annual loss contribution by earthquake source](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_source_aal_contributions.png)

This produces an important insurance result:

**The ceded portfolio is more concentrated in Cascadia interface risk than the underlying ground-up portfolio.**

In other words, the reinsurance structure does not simply scale the physical loss distribution.

It changes its composition.

The largest events are disproportionately important for the excess-of-loss layer, causing Cascadia interface earthquakes to account for an even greater share of transferred loss than of physical portfolio loss.

---

## 12. AEP, OEP, and PML

The project calculates both aggregate and occurrence annual exceedance distributions.

### Aggregate Exceedance Probability

AEP answers:

> What is the annual probability that the **total loss from all earthquakes during the year** exceeds a selected amount?

This includes accumulation from multiple events.

![Full AEP exceedance curves](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_full_aep_exceedance_curves.png)

### Occurrence Exceedance Probability

OEP answers:

> What is the annual probability that the **largest individual earthquake loss during the year** exceeds a selected amount?

This is particularly relevant for occurrence-based reinsurance.

![Full OEP exceedance curves](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_full_oep_exceedance_curves.png)

Headline PML results are reported at:

- 100 years
- 250 years
- 500 years
- 1,000 years
- 2,500 years
- 5,000 years
- 10,000 years

The project deliberately avoids presenting extremely thin-tail points as primary results. Return periods of 200,000 years and longer are retained only as diagnostics because fewer than 20 annual order statistics support those estimates.

---

## 13. The largest modeled earthquake loss

The largest occurrence in the two-million-year catalog is a **magnitude 9.34 Cascadia interface earthquake**.

It produces approximately:

| Loss measure | Largest occurrence |
|---|---:|
| Ground-up loss | $282.75 million |
| Gross insured loss | $244.33 million |
| Ceded loss | $61.84 million |

The ground-up loss is approximately **73.6% of total portfolio replacement value**.

The ceded occurrence loss reaches the full $61.84 million layer limit.

This illustrates the basic shape of an occurrence excess-of-loss contract.

The reinsurer absorbs a defined middle layer of loss above attachment, but once the layer is exhausted, additional extreme loss returns to the insurer's retained position.

So reinsurance significantly reduces tail loss without eliminating the retained extreme-event tail.

---

## 14. Why annual ceded loss can exceed the occurrence limit

The largest **annual aggregate ceded loss** is approximately:

**$90.41 million**

That is larger than the $61.84 million occurrence-layer limit.

This is not a modeling error.

The $61.84 million limit applies **per occurrence**.

If more than one earthquake produces a recoverable loss in the same year, each event can generate a separate recovery.

Because the synthetic baseline contract has:

- no annual aggregate cap
- no reinstatement restriction

the total ceded recovery across a year can exceed the maximum recovery from any single occurrence.

This is a useful example of why AEP and OEP answer different financial questions.

---

## 15. Validation as part of the model

Validation was treated as part of the catastrophe-risk workflow rather than as a final cosmetic check.

The model includes checks for:

- downloaded archive integrity and SHA-256 hashes
- rupture and event identifiers
- logic-tree and annual-rate reconciliation
- stochastic catalog accounting
- zero-event and multiple-event years
- rupture-to-site joins
- distance validity
- occurrence-site row counts
- ground-motion reconstruction
- residual distributions and cross-IMT dependence
- fragility probability bounds
- damage-state probability sums
- repair-cost bounds
- sampled versus analytical loss behavior
- insurance equations
- reinsurance equations
- AAL reconciliation
- AEP and OEP construction
- PML monotonicity
- source-level reconciliation
- output manifests
- deterministic random streams
- restartable chunk processing
- repository portability

The final reporting notebook completed:

**87 critical validation checks**

with:

**0 critical failures**

and:

**0 unresolved warnings**

A separate repository-hardening validator completed:

**54 checks with 0 critical failures.**

The goal was not merely to obtain plausible final numbers.

The goal was to build an auditable chain in which each major transformation could be traced and checked.

---

## 16. Reproducibility and engineering design

The project was built as a restartable workflow rather than as a single large script.

The seven notebooks are:

| Notebook | Purpose |
|---|---|
| `01_download_and_inspect_usgs_nshm2018.ipynb` | Download, verify, inventory, and inspect the USGS model |
| `02_extract_usgs_rupture_rates.ipynb` | Expand source definitions into rupture-level annual rates |
| `03_generate_annual_event_catalog.ipynb` | Generate the two-million-year annual event catalog |
| `04_generate_ground_motion_fields.ipynb` | Calculate distances and simulate ground-motion fields |
| `05_calculate_ground_up_losses.ipynb` | Simulate damage and calculate ground-up loss |
| `06_apply_insurance_terms.ipynb` | Apply insurance and reinsurance terms |
| `07_baseline_results_and_validation.ipynb` | Produce final metrics, figures, and validation |

The workflow uses:

- Python
- pandas
- NumPy
- SciPy
- Matplotlib
- Jupyter
- Java
- Gradle
- USGS NSHM
- HAZUS-style fragility modeling
- Monte Carlo simulation
- deterministic random-number namespaces
- SHA-256 artifact verification
- chunk manifests and restart markers

Large raw datasets and generated production tables are intentionally excluded from Git, while model versions, validation records, metadata, selected outputs, setup instructions, and reporting figures are retained.

---

## 17. What I learned from the baseline

Several findings stand out.

### 1. Physical and financial risk are not the same thing

Ground-up loss, insured loss, retained loss, and ceded loss represent different views of the same catastrophe.

Deductibles and reinsurance meaningfully reshape the loss distribution.

### 2. Nonstructural damage can dominate expected portfolio loss

Acceleration-sensitive nonstructural loss contributes more than three quarters of ground-up AAL in the baseline.

Engineering importance and insurance-cost importance are therefore not necessarily concentrated in the same components.

### 3. Reinsurance can increase apparent source concentration

Cascadia interface events account for 90.60% of ground-up AAL but 95.49% of ceded AAL.

The excess-of-loss structure preferentially transfers losses from the most severe events.

### 4. Annual and occurrence risk are different

AEP and OEP may be similar when one event dominates a year, but multiple-event years can create materially different aggregate outcomes.

The $90.41 million maximum annual ceded loss, compared with the $61.84 million occurrence limit, illustrates this distinction directly.

### 5. Dependence deserves explicit treatment

The marginal loss probability of one building does not fully describe portfolio catastrophe risk.

What matters for tail loss is also **how many locations experience unusually high ground motion and damage together**.

That is the motivation for Phase 2.

---

## 18. Phase 2: spatial correlation

Phase 1 establishes the controlled reference model.

Phase 2 will replace the conditionally independent within-event site residuals with source-appropriate spatially correlated residual fields.

The comparison will preserve:

- the same 470 buildings
- the same building order
- the same rupture set
- the same two-million-year annual event catalog
- the same marginal ground-motion model
- the same insurance terms
- the same reinsurance terms
- the same damage and repair-cost framework
- the same deterministic damage uniforms
- as much of the same random-number structure as practical

This paired design is intentional.

If independent and correlated cases used different earthquake catalogs, differences in loss could come from different earthquake realizations rather than from correlation.

By holding the event catalog and other major inputs fixed, the analysis can isolate the effect of **spatial dependence itself**.

The Phase 2 comparison will examine changes in:

- ground-up AAL
- insured AAL
- AEP and OEP curves
- PML
- largest occurrence loss
- portfolio loss concentration
- ceded loss
- retained loss
- layer attachment behavior
- layer exhaustion behavior
- sensitivity to the selected correlation model

---

## Scope and limitations

This project is a transparent catastrophe-risk modeling demonstration.

It is **not** a production catastrophe model, an insurance quotation, or a regulatory capital model.

Important Phase 1 limitations include:

- one demonstration portfolio in Seaside, Oregon
- W2 building focus
- building repair-cost loss only
- synthetic insurance terms
- synthetic reinsurance terms
- no contents loss
- no business-interruption loss
- no additional-living-expense loss
- no casualty loss
- no demand surge
- no post-event inflation
- no claims-adjustment expense
- no annual aggregate reinsurance cap
- no reinstatement restrictions
- direct-SA(0.4) fragility approximation rather than the full HAZUS capacity-spectrum procedure
- no spatial correlation among within-event site residuals in Phase 1

These boundaries are documented so that the results are interpreted as transparent baseline estimates for model development and comparison rather than as market loss estimates.

---

## Why this project matters to catastrophe risk

The main value of the project is not any one AAL or PML number.

It is the complete connection between:

**earthquake science → engineering response → physical damage → financial loss → insurance → reinsurance → portfolio risk**

The workflow demonstrates how assumptions introduced at the hazard or damage stage can propagate into decisions relevant to:

- catastrophe modelers
- insurers
- reinsurers
- reinsurance brokers
- risk engineers
- portfolio risk managers
- insurance-linked securities analysts

It also creates a controlled platform for the project's central next question:

> **How does spatial dependence change the concentration, tail behavior, and financial transfer of earthquake portfolio loss?**

---

## Code

The complete Phase 1 repository, including the seven notebooks, setup documentation, validation records, selected figures, and repository-validation tools, is available here:

**[View the project on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)**

[Back to Projects](/projects/)
