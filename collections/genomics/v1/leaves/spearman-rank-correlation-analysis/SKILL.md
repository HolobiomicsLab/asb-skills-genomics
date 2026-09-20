---
name: spearman-rank-correlation-analysis
description: Use when validating Nanodynamo inference on simulated RNA life-cycle
  data by correlating modelled vs expected RNA species abundances and inferred vs
  expected synthesis, processing, degradation, export, and translation rates per gene.
when_to_use_negative:
- Input is experimental (not simulated) Nanodynamo data with no ground-truth expected
  rates — correlating against unknown truth is not meaningful; use experimentalDataAnalysis.R
  / mainModelingResults.R instead
- Data require linear-agreement or scale-sensitive comparison where Pearson correlation
  or direct rate error metrics are more appropriate than rank-based correlation
- The simulation was run with a different design (e.g. different number of replicates
  or labelling pulse duration), so the 0.68–0.96 rate and >0.98 abundance acceptance
  ranges may not apply
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language in which the Spearman correlation goodness-of-fit computations are
    implemented (R script to reproduce analyses on simulated data)
  repo: github.com/mfurla/Nanodynamo
- name: dataSimulationAnalysis.R
  role: R script from the Nanodynamo repository used to simulate genes and analyze
    the resulting dataset, producing the modelled abundances and inferred rates that
    are Spearman-correlated against expected values
  repo: github.com/mfurla/Nanodynamo
- name: Nanodynamo
  role: Experimental and computational workflow whose inference pipeline (fitting
    synthesis, processing, degradation, export, and translation rates) is being benchmarked
    with Spearman correlations
  repo: github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:40916723a13118304cc95defd52f36f946bd6a0ecb8fd22098cb0d224f4e43da
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  - dataSimulationAnalysis.R
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spearman-rank-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify rank-based agreement between Nanodynamo-inferred RNA species abundances/kinetic rates and expected values on simulated data (1000 genes, 2 replicates, 20 min 4SU-style labelling pulse), using median Spearman correlations >0.98 for abundances and Spearman coefficients of 0.68–0.96 for rates as acceptance criteria.

## When to use

Use when validating Nanodynamo inference on simulated RNA life-cycle data by correlating modelled vs expected RNA species abundances and inferred vs expected synthesis, processing, degradation, export, and translation rates per gene.

## When NOT to use

- Input is experimental (not simulated) Nanodynamo data with no ground-truth expected rates — correlating against unknown truth is not meaningful; use experimentalDataAnalysis.R / mainModelingResults.R instead
- Data require linear-agreement or scale-sensitive comparison where Pearson correlation or direct rate error metrics are more appropriate than rank-based correlation
- The simulation was run with a different design (e.g. different number of replicates or labelling pulse duration), so the 0.68–0.96 rate and >0.98 abundance acceptance ranges may not apply

## Inputs

- Simulated RNA species abundances and expected kinetic rates from the Nanodynamo simulation (task_003 output, 1000 genes, 2 replicates, 20 min labelling pulse)
- dataSimulationAnalysis.R workflow script from github.com/mfurla/Nanodynamo

## Outputs

- simulated_inference_gof.csv containing inferred kinetic rates and goodness-of-fit statistics (median Spearman correlation of modelled vs expected abundances; per-rate-type Spearman coefficients)
- Per-gene Spearman correlations between modelled and expected RNA species abundances

## How to apply

Load the simulated RNA species abundances and expected kinetic rates produced by the task_003 simulation into R, then run the dataSimulationAnalysis.R workflow from github.com/mfurla/Nanodynamo to fit the model and estimate synthesis, processing, degradation, export, and translation rates. For each gene, compute Spearman rank correlations between modelled and expected RNA species abundances and take their median as the first goodness-of-fit metric. For each rate type, compute the Spearman coefficient between inferred and expected values as the second metric. Acceptance requires the median abundance Spearman correlation to exceed 0.98 and the rate correlations to fall within the 0.68–0.96 range reported in the paper. Write the inferred rates and goodness-of-fit statistics to simulated_inference_gof.csv.

## Related tools

- **R** (Language in which the Spearman correlation goodness-of-fit computations are implemented (R script to reproduce analyses on simulated data)) — github.com/mfurla/Nanodynamo
- **dataSimulationAnalysis.R** (R script from the Nanodynamo repository used to simulate genes and analyze the resulting dataset, producing the modelled abundances and inferred rates that are Spearman-correlated against expected values) — github.com/mfurla/Nanodynamo
- **Nanodynamo** (Experimental and computational workflow whose inference pipeline (fitting synthesis, processing, degradation, export, and translation rates) is being benchmarked with Spearman correlations) — github.com/mfurla/Nanodynamo

## Examples

```
Rscript dataSimulationAnalysis.R  # compute per-gene Spearman correlations of modelled vs expected abundances and per-rate-type Spearman coefficients, writing simulated_inference_gof.csv
```

## Evaluation signals

- Median per-gene Spearman correlation between modelled and expected RNA species abundances exceeds 0.98
- Spearman coefficients between inferred and expected kinetic rates fall within the 0.68–0.96 range reported in the paper
- simulated_inference_gof.csv is produced and contains both inferred rates and the goodness-of-fit statistics
- Rate-specific correlations are computed separately for each rate type (synthesis, processing, degradation, export, translation) rather than pooled
- Inference was run with the correct simulation design: 1000 genes, 2 replicates, 20 min labelling pulse

## Limitations

- The 0.68–0.96 rate-recovery range indicates inferred kinetic rates are less accurately recovered than abundances (which reach >0.98), so rate-level conclusions drawn from simulations carry larger uncertainty
- Acceptance thresholds were established for the specific simulation design (1000 genes, 2 replicates, 20 min labelling pulse); other designs may yield different correlation ranges
- Rank-based Spearman correlation is insensitive to absolute-scale errors, so agreement does not guarantee numerically accurate rate estimates

## Evidence

- [results] modelled RNA species abundances matched expected levels with median Spearman correlations >0.98, and inferred kinetic rates correlated with expected values with Spearman coefficients in the 0.68-0.96 range: [evidence span withheld — anchor status: not_found]
