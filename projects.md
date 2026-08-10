---
layout: page
title: Projects
permalink: /projects/
---

# Projects

## Seismic Portfolio Catastrophe Risk Model

A reproducible earthquake catastrophe-risk modeling workflow connecting the USGS National Seismic Hazard Model to portfolio-level financial, insurance, and reinsurance risk.

The project develops a complete probabilistic workflow from earthquake occurrence through portfolio loss:

**USGS NSHM → Rupture Rates → Stochastic Event Catalog → Ground Motion → Damage → Ground-Up Loss → Insurance → Reinsurance → AAL / OEP / AEP / PML**

### What the project includes

- rupture-level earthquake occurrence rates derived from the USGS NSHM
- a stochastic annual earthquake event catalog
- portfolio ground-motion simulation
- building damage and ground-up economic loss
- insurance deductibles and limits
- retained and ceded reinsurance loss
- Average Annual Loss (AAL)
- Occurrence Exceedance Probability (OEP)
- Aggregate Exceedance Probability (AEP)
- Probable Maximum Loss (PML)
- validation checks throughout the modeling workflow

### Project status

**Phase 1: Baseline catastrophe-risk model - Complete**

The current baseline implements the full catastrophe-risk workflow without spatial ground-motion correlation.

**Phase 2: Spatial correlation extension - Planned**

The next phase will use the same event catalog to compare conditionally independent and spatially correlated ground-motion residuals and quantify their effect on portfolio and reinsurance risk.

[View the source code on GitHub](https://github.com/NatCatAnalystRandle/seismic-correlation-insurance-loss)
