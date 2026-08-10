---
layout: page
title: "Building an Earthquake Catastrophe Risk Model"
permalink: /projects/seismic-catastrophe-risk-model/
---

**From USGS seismic hazard to portfolio damage, insurance loss, reinsurance recovery, and catastrophe-risk metrics**

<div class="case-study-actions">
  <a class="case-study-button" href="https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss">
    View Source Code on GitHub
  </a>
</div>

---

## Project overview

I built an end-to-end earthquake catastrophe-risk model that connects seismic hazard with the financial loss metrics used in insurance and reinsurance.

Engineering analyses often stop at hazard or physical damage, while insurance analyses often begin with financial loss. I wanted to build the chain between the two.

The model starts with the **USGS 2018 National Seismic Hazard Model** and progresses through earthquake occurrence, stochastic event simulation, ground motion, building damage, ground-up loss, insurance, reinsurance, and portfolio risk metrics.

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

## Catastrophe-risk workflow

<div class="cat-workflow">

  <div class="workflow-step">USGS NSHM</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Event Catalog</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Ground Motion</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Damage</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Ground-Up Loss</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Insurance</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">Reinsurance</div>
  <div class="workflow-arrow">→</div>

  <div class="workflow-step">AAL · AEP · OEP · PML</div>

</div>

The model is implemented as seven sequential and restartable Jupyter notebooks, with validation at every major stage.

---

## How the model works

### Hazard and event simulation

The model uses the **USGS 2018 CONUS National Seismic Hazard Model (5.2.4)** to represent Cascadia interface and Oregon intraslab earthquakes.

Rupture-level annual occurrence rates are used to generate a **2-million-year stochastic catalog containing 10,630 earthquake occurrences**.

The source-model workflow preserves important distinctions between physical earthquake occurrence, epistemic alternatives, source scaling, and magnitude-frequency distributions.

### Ground motion, damage, and loss

PGA and SA(0.4) are simulated across all 470 portfolio locations.

Phase 1 includes:

- shared between-event variability across the portfolio
- correlated PGA and SA(0.4) residuals at the same site
- conditionally independent within-event residuals between different sites

Spatial correlation between sites is deliberately excluded from Phase 1 so that it can later be introduced as a controlled model extension.

Ground motion is translated into structural and nonstructural damage and then into building-level repair loss.

### Insurance and reinsurance

A synthetic insurance program applies a **10% building-level deductible**.

The resulting financial views include:

- uninsured loss
- gross insured loss
- ceded reinsurance loss
- net retained loss

A synthetic occurrence excess-of-loss layer attaches at approximately **$18.81 million** and has a **$61.84 million occurrence limit**.

This allows the same earthquake portfolio to be viewed from engineering, insurer, and reinsurer perspectives.

---

## Key findings

### 1. Insurance and reinsurance materially reshape loss

The baseline ground-up AAL is:

**$195,922**

After insurance terms, gross insured AAL is:

**$122,980**

Of that insured loss:

- **$63,677** is ceded to reinsurance
- **$59,303** remains as net retained AAL

![Baseline average annual loss flow](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_aal_loss_flow.png)

Approximately **37% of ground-up AAL remains uninsured**, despite full insurance take-up in the synthetic portfolio, largely because of the building-level deductible.

---

### 2. Nonstructural damage dominates expected loss

| Damage component | Share of ground-up AAL |
|---|---:|
| Structural | 8.74% |
| Nonstructural drift-sensitive | 15.01% |
| Nonstructural acceleration-sensitive | **76.25%** |

More than three quarters of expected repair loss comes from **acceleration-sensitive nonstructural damage**.

This highlights an important difference between structural safety and portfolio financial risk: the components most important for life safety are not necessarily the components driving expected economic loss.

---

### 3. Reinsurance increases concentration in Cascadia risk

Cascadia interface earthquakes contribute approximately:

- **90.60% of ground-up AAL**
- **95.49% of ceded AAL**

![Average annual loss contribution by earthquake source](https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/main/data/processed/notebook_7_baseline_results_validation/plots/baseline_source_aal_contributions.png)

The ceded portfolio is therefore even more concentrated in Cascadia risk than the underlying physical-loss portfolio.

Because excess-of-loss reinsurance responds disproportionately to severe events, the reinsurance structure changes not only the amount of loss retained but also the composition of the risk transferred.

---

### 4. Occurrence and annual aggregate risk are different

The largest modeled occurrence is a **magnitude 9.34 Cascadia interface earthquake** producing approximately:

- **$282.75 million** ground-up loss
- **$244.33 million** gross insured loss
- **$61.84 million** ceded loss

The ceded loss reaches the full occurrence-layer limit.

However, maximum **annual aggregate ceded loss** reaches approximately **$90.41 million**.

This can exceed the $61.84 million occurrence limit because multiple earthquakes can generate separate reinsurance recoveries within the same year.

It illustrates why catastrophe models distinguish between:

- **OEP**, which focuses on the largest event loss in a year
- **AEP**, which captures total annual loss accumulation

---

## Validation

The workflow was designed to be auditable and restartable, with validation spanning source rates, event accounting, ground motion, damage, financial transformations, exceedance curves, and output integrity.

**87 critical reporting checks:** 0 failures, 0 unresolved warnings

**54 repository-hardening checks:** 0 critical failures

The complete validation records are available in the GitHub repository.

---

## What's next: spatial correlation

Phase 2 will reuse the same portfolio, rupture set, event catalog, damage model, insurance terms, and reinsurance structure while introducing spatial correlation among within-event ground-motion residuals.

Keeping the baseline inputs fixed is important because it allows differences in portfolio loss to be attributed to the dependence assumption rather than to different earthquake realizations.

<p>The next question is:</p>

<div class="key-question">
How does spatial dependence change portfolio concentration, tail loss, PML, and reinsurance risk?
</div>

---

## Scope

This project is a transparent research and portfolio demonstration, not a production catastrophe model or insurance quotation.

The baseline uses synthetic insurance and reinsurance terms, a W2 demonstration portfolio, building repair-cost losses only, a direct-SA(0.4) fragility approximation, and no spatial correlation among Phase 1 within-event site residuals.

The complete assumptions, methodology, limitations, validation outputs, and implementation details are documented in the repository.

---

## Explore the project

The repository contains the seven modeling notebooks, methodology, validation records, figures, setup instructions, and reproducibility documentation.

**[View the complete project on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)**

[Back to Projects](/projects/)
