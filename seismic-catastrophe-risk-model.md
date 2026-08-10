---
layout: page
title: "Building an Earthquake Catastrophe Risk Model"
permalink: /projects/seismic-catastrophe-risk-model/
---

**From USGS seismic hazard to portfolio damage, insurance loss, reinsurance recovery, and catastrophe-risk metrics**

[View the source code on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)

---

## Project overview

I built an end-to-end earthquake catastrophe-risk model that connects seismic hazard with the financial loss metrics used in insurance and reinsurance.

The workflow starts with the **USGS 2018 National Seismic Hazard Model** and progresses through earthquake occurrence, stochastic event simulation, ground motion, building damage, ground-up loss, insurance recovery, reinsurance, and portfolio risk metrics.

### At a glance

| Metric | Baseline result |
|---|---:|
| Portfolio | 470 buildings in Seaside, Oregon |
| Replacement value | $384.24 million |
| Stochastic catalog | 2,000,000 years |
| Simulated earthquake occurrences | 10,630 |
| Ground-up AAL | $195,922 |
| Gross insured AAL | $122,980 |
| Ceded AAL | $63,677 |
| Net retained AAL | $59,303 |

The current release is **Phase 1**, a validated baseline model without spatial correlation among within-event site residuals.

---

## Why I built it

Engineering analyses often stop at hazard or physical damage.

Insurance analyses often begin with financial loss.

I wanted to build the chain between the two.

The project answers a single end-to-end question:

> **How does earthquake occurrence ultimately become portfolio loss, insured loss, and reinsurance loss?**

---

## Catastrophe-risk workflow

**USGS NSHM**

↓  

**Rupture occurrence rates**

↓  

**2-million-year stochastic event catalog**

↓  

**Ground-motion simulation**

↓  

**Structural and nonstructural damage**

↓  

**Ground-up repair loss**

↓  

**Insurance terms**

↓  

**Reinsurance**

↓  

**AAL · AEP · OEP · PML**

The model is implemented as seven sequential and restartable Jupyter notebooks, with validation at every major stage.

---

## How the model works

### 1. Earthquake occurrence

The model uses the **USGS 2018 CONUS National Seismic Hazard Model**, pinned to `nshm-conus 5.2.4`.

The demonstration portfolio focuses on two earthquake-source families relevant to coastal Oregon:

- Cascadia subduction interface earthquakes
- Oregon intraslab earthquakes

USGS source definitions and magnitude-frequency distributions are expanded into rupture-level annual occurrence rates while preserving the distinction between physical source occurrence, epistemic alternatives, and source scaling.

Those rates are then used to generate a **2,000,000-year stochastic earthquake catalog** containing 10,630 simulated occurrences.

---

### 2. Ground motion and damage

Ground motion is simulated at all 470 portfolio locations using PGA and SA(0.4).

The Phase 1 baseline includes:

- shared between-event variability across the portfolio
- correlated PGA and SA(0.4) residuals at the same site
- conditionally independent within-event residuals between different sites

Spatial correlation between sites is deliberately excluded from Phase 1 so that it can be introduced later as a controlled model extension.

Ground motions are translated into:

- structural damage
- nonstructural drift-sensitive damage
- nonstructural acceleration-sensitive damage

and then into building-level repair costs.

---

### 3. Insurance and reinsurance

Ground-up losses are transformed using a synthetic insurance structure with a **10% building-level deductible**.

The resulting losses are separated into:

- uninsured loss
- gross insured loss
- ceded reinsurance loss
- net retained loss

A synthetic occurrence excess-of-loss reinsurance layer attaches at approximately **$18.81 million** and exhausts at approximately **$80.65 million**, giving a **$61.84 million occurrence limit**.

This allows the same earthquake portfolio to be viewed from engineering, insurer, and reinsurer perspectives.

---

## What the model found

### 1. Insurance materially reshapes the physical loss

The baseline ground-up AAL is:

**$195,922**

After policy terms:

**$122,980** becomes gross insured AAL.

Of that insured loss:

- **$63,677** is ceded to reinsurance
- **$59,303** remains with the insurer

![Baseline average annual loss flow](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_aal_loss_flow.png)

Approximately **37% of ground-up AAL remains uninsured**, despite full insurance take-up in the synthetic portfolio, largely because of the building-level deductible.

---

### 2. Nonstructural damage dominates expected loss

The contribution to ground-up AAL is:

| Damage component | Share |
|---|---:|
| Structural | 8.74% |
| Nonstructural drift-sensitive | 15.01% |
| Nonstructural acceleration-sensitive | **76.25%** |

More than three quarters of expected repair loss comes from **acceleration-sensitive nonstructural damage**.

This highlights an important difference between structural safety and financial portfolio risk: the components most important for life safety are not necessarily the components driving expected insured loss.

---

### 3. Reinsurance increases concentration in Cascadia risk

Cascadia interface earthquakes contribute approximately:

- **90.60% of ground-up AAL**
- **95.49% of ceded AAL**

![Average annual loss contribution by earthquake source](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_source_aal_contributions.png)

The reinsurance portfolio is therefore even more concentrated in Cascadia risk than the underlying physical-loss portfolio.

The reason is intuitive: an excess-of-loss layer responds disproportionately to the largest events.

---

### 4. Occurrence risk and annual aggregate risk are different

The largest modeled occurrence is a **magnitude 9.34 Cascadia interface earthquake** producing approximately:

- **$282.75 million** ground-up loss
- **$244.33 million** gross insured loss
- **$61.84 million** ceded loss

The ceded loss reaches the full occurrence-layer limit.

However, the largest **annual aggregate ceded loss** is approximately **$90.41 million**.

That can exceed the $61.84 million occurrence limit because multiple earthquakes can trigger separate reinsurance recoveries within the same year.

This is one reason catastrophe models distinguish between:

- **OEP**, which focuses on the largest event in a year
- **AEP**, which captures total annual accumulation

---

## Validation

The project was designed to be auditable, not simply to produce plausible numbers.

Validation covers:

- USGS source interpretation
- rupture occurrence rates
- stochastic event accounting
- rupture-to-site distances
- ground-motion calculations
- damage probabilities
- repair-cost bounds
- insurance equations
- reinsurance equations
- AAL reconciliation
- AEP and OEP construction
- PML behavior
- deterministic random streams
- output integrity and reproducibility

The final reporting workflow completed:

**87 critical checks with 0 critical failures and 0 unresolved warnings.**

A separate repository-hardening validator completed:

**54 checks with 0 critical failures.**

---

## Why Phase 1 matters

The baseline is intentionally constructed without spatial correlation among within-event site residuals.

That gives Phase 2 a controlled reference case.

The spatial-correlation extension will reuse:

- the same 470 buildings
- the same rupture set
- the same 2-million-year event catalog
- the same damage model
- the same insurance terms
- the same reinsurance terms
- as much of the same random-number structure as possible

The goal is to isolate one question:

> **How does spatial dependence change earthquake portfolio concentration, tail loss, and reinsurance risk?**

---

## Scope

This is a transparent research and portfolio project, not a production catastrophe model or insurance quotation.

Important simplifications include synthetic insurance and reinsurance terms, a W2 demonstration portfolio, repair-cost loss only, a direct-SA(0.4) fragility approximation, and no spatial correlation in the Phase 1 within-event site residuals.

These assumptions are documented explicitly in the repository.

---

## Explore the project

The full repository contains the seven modeling notebooks, methodology, validation outputs, figures, setup instructions, and reproducibility documentation.

**[View the complete project on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)**

[Back to Projects](/projects/)
