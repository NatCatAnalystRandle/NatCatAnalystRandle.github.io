---
layout: page
title: Projects
kicker: Independent technical work
description: Engineering models applied to catastrophe risk, insurance, and reinsurance.
permalink: /projects/
---

<section class="text-entry">
  <p class="entry-date">Completed / v2.0.0</p>
  <h2>Seismic Correlation and Insurance Loss</h2>
  <p>A reproducible earthquake catastrophe-risk workflow that connects seismic sources to structural and nonstructural damage, insured portfolio loss, reinsurance, and parametric basis risk.</p>
  <p class="entry-meta">470 buildings in Seaside, Oregon<br>$384.24 million replacement value<br>2,000,000 simulated years / 13 notebooks</p>
  <div class="inline-links">
    <a href="{{ '/projects/seismic-catastrophe-risk-model/' | relative_url }}">Read the full case study</a>
    <a href="https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/tree/v2.0.0">View source &amp; results</a>
  </div>
  <figure class="entry-photo">
    <img src="{{ '/assets/images/gross-insured-tail-curves.png' | relative_url }}" alt="Modeled gross insured aggregate and occurrence exceedance curves for independent and spatially correlated cases">
    <figcaption>Comparing portfolio tails while keeping the event catalog and financial assumptions fixed.</figcaption>
  </figure>
</section>

## The question

<div class="key-question">How does spatial dependence in ground motion change the risk retained by an insurer, the reinsurance needed to transfer that risk, and the shortfalls left by parametric protection?</div>

The model uses the same event catalog, portfolio, policy terms, and paired random streams across three dependence cases. This allows the comparison to focus on the spatial-dependence assumption.

## What the project covers

- USGS rupture-level annual rates and stochastic event simulation.
- Paired ground-motion fields and structural and nonstructural damage.
- Ground-up, gross insured, uninsured, ceded, and retained losses.
- AAL, AEP, OEP, PML, and TVaR-based tail comparisons.
- Occurrence and annual aggregate reinsurance sensitivity.
- Parametric trigger calibration, held-out evaluation, and basis risk.
- Reproducibility checks and paired bootstrap uncertainty.

<p class="note">This is a portfolio demonstration using synthetic exposure and financial terms. Results are conditional on the selected hazard, damage, and contract assumptions. The case study documents the limitations.</p>
