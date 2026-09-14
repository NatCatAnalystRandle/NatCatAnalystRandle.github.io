---
layout: page
title: Projects
permalink: /projects/
---

<p class="page-intro">Selected technical work connecting natural-hazard engineering, probabilistic catastrophe-risk modeling, and financial risk. Each project emphasizes transparent assumptions, reproducible analysis, and decision-relevant results.</p>

<div class="project-card-large">
  <div>
    <span class="eyebrow">Featured · Completed and validated</span>
    <h2>Seismic Correlation and Insurance Loss</h2>

    <div class="tag-list">
      <span class="tag">Earthquake catastrophe risk</span>
      <span class="tag">USGS NSHM</span>
      <span class="tag">Spatial dependence</span>
      <span class="tag">Reinsurance</span>
      <span class="tag">Parametric basis risk</span>
    </div>

    <p>A reproducible 13-notebook workflow connecting the USGS 2018 National Seismic Hazard Model to earthquake occurrence, paired ground-motion fields, structural and nonstructural damage, portfolio loss, insurance, reinsurance, and parametric catastrophe-bond analysis.</p>

    <div class="metric-grid">
      <div class="metric-card">
        <strong>2M</strong>
        <span>Catalog years</span>
      </div>
      <div class="metric-card">
        <strong>470</strong>
        <span>Buildings</span>
      </div>
      <div class="metric-card">
        <strong>$384.24M</strong>
        <span>Portfolio value</span>
      </div>
      <div class="metric-card">
        <strong>3</strong>
        <span>Dependence cases</span>
      </div>
    </div>

    <p>The comparison holds the catalog, portfolio, policy terms, and paired random streams fixed across the independent and correlated cases. This isolates the modeled effect of spatial dependence from ordinary simulation differences.</p>

    <div class="button-row">
      <a class="button button-primary" href="/projects/seismic-catastrophe-risk-model/">Read the case study</a>
      <a class="button" href="https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/tree/v2.0.0">View validated source</a>
    </div>
  </div>

  <figure class="project-visual">
    <img src="https://raw.githubusercontent.com/NatCatAnalystRandle/seismic-correlation-insurance-loss/v2.0.0/data/processed/phase_2/notebook_13_phase_2_results/plots/gross_insured_tail_curves.png" alt="Modeled gross insured aggregate and occurrence exceedance curves for three ground-motion dependence cases">
    <figcaption class="figure-label">Gross insured AEP and OEP curves across the independent and two spatial-correlation cases.</figcaption>
  </figure>
</div>

## What the workflow covers

<div class="capability-grid">
  <article class="capability-card">
    <h3>Hazard and event simulation</h3>
    <p>Rupture-level annual-rate extraction, a 2,000,000-year stochastic catalog, source-appropriate ground motion, and paired dependence cases.</p>
  </article>

  <article class="capability-card">
    <h3>Damage and policy loss</h3>
    <p>Structural and nonstructural damage, ground-up loss, per-building deductibles and limits, gross insured loss, and uninsured loss.</p>
  </article>

  <article class="capability-card">
    <h3>Risk transfer and tails</h3>
    <p>Occurrence excess-of-loss, aggregate protection, retained and ceded loss, AAL, AEP, OEP, PML, TVaR, and parametric basis risk.</p>
  </article>
</div>

## Scope and interpretation

This is a transparent research and portfolio demonstration, not a production catastrophe model or an insurance quotation. It uses a synthetic 470-building portfolio, synthetic policy and risk-transfer terms, HAZUS-style fragility and repair-cost approximations, and building repair loss only. The case study reports the material limitations alongside the results.

<p><a class="text-link" href="/projects/seismic-catastrophe-risk-model/">Read the methodology, findings, validation, and limitations</a></p>
