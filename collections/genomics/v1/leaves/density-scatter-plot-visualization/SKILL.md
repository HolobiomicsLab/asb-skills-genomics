---
name: density-scatter-plot-visualization
description: Use when you need to assess quantitative agreement between site-level
  modification stoichiometry measurements and a reference (e.g., mAFiA modRatio vs
  GLORI) across a titration or dilution series, by visualizing the joint distribution
  and regressing one against the other.
when_to_use_negative:
- Sites fall below the minimum dRNA-Seq coverage of 50 required for comparison against
  GLORI measurements — low-coverage sites will have unstable modRatio estimates and
  should be excluded rather than plotted.
- The goal is single-molecule (read-level) modification detection rather than site-level
  ensemble stoichiometry comparison against an orthogonal method like GLORI.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: mAFiA
  role: Produces the per-site mAFiA.sites.bed with coverage and modRatio stoichiometry
    values that populate the density scatter plot
  repo: https://github.com/dieterich-lab/mAFiA
- name: sklearn
  role: Fits the linear regression SmAFiA = m*SGLORI + b to obtain the slope m across
    the titration series
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:8d7b966605d26090a71012058769e2252ab3e83b59ff3442136958487bc9261b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - mAFiA
  - sklearn
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# density-scatter-plot-visualization

## Summary

Renders a 2D density scatter plot comparing per-site m6A stoichiometry measured by mAFiA (SmAFiA) against GLORI reference values (SGLORI) across the HEK293 WT/IVT titration series, then fits a linear regression SmAFiA = m*SGLORI + b with sklearn to quantify how the slope m tracks the WT fraction fWT (0.00–1.00).

## When to use

Use when you need to assess quantitative agreement between site-level modification stoichiometry measurements and a reference (e.g., mAFiA modRatio vs GLORI) across a titration or dilution series, by visualizing the joint distribution and regressing one against the other.

## When NOT to use

- Sites fall below the minimum dRNA-Seq coverage of 50 required for comparison against GLORI measurements — low-coverage sites will have unstable modRatio estimates and should be excluded rather than plotted.
- The goal is single-molecule (read-level) modification detection rather than site-level ensemble stoichiometry comparison against an orthogonal method like GLORI.

## Inputs

- mAFiA.sites.bed files (columns include coverage, modRatio, confidence) for each WT/IVT titration sample (fWT = 0.00–1.00)
- GLORI_chrX.bed: query modification sites in bed format corresponding to GLORI-listed sites
- Minimum coverage threshold parameter (min_coverage 50)

## Outputs

- density_scatter_SGLORI.png: 2D density scatter plot of SmAFiA vs SGLORI across the titration series
- slope_report.txt: fitted regression slope m(SmAFiA/SGLORI) with sklearn

## How to apply

Load per-site stoichiometry from the 'modRatio' column of each titration sample's mAFiA.sites.bed, keeping only sites with coverage >= 50 (min_coverage 50), and intersect them with query sites in GLORI_chrX.bed so each mAFiA stoichiometry is paired with its GLORI reference value. Aggregate per-fraction measurements into SmAFiA and SGLORI values spanning the fWT = 0.00–1.00 titration points, then render a 2D density scatter plot of SmAFiA vs SGLORI to reveal where measurement mass concentrates across the site distribution. Fit a linear regression SmAFiA = m*SGLORI + b with the python module sklearn (which the article also uses with its default L-BFGS solver) to obtain the slope m(SmAFiA/SGLORI). The rationale is that in mixtures of wildtype and IVT mRNA, the expected site stoichiometry scales with the WT fraction fWT, so the fitted slope should largely agree with fWT, confirming mAFiA's stoichiometric sensitivity across the titration series.

## Related tools

- **mAFiA** (Produces the per-site mAFiA.sites.bed with coverage and modRatio stoichiometry values that populate the density scatter plot) — https://github.com/dieterich-lab/mAFiA
- **sklearn** (Fits the linear regression SmAFiA = m*SGLORI + b to obtain the slope m across the titration series)

## Examples

```
pileup --bam_file ${out_dir}/mAFiA.reads.bam --sites ${sites} --min_coverage 50 --out_dir ${out_dir} --num_jobs 36
```

## Evaluation signals

- The fitted regression slope m(SmAFiA/SGLORI) agrees with the underlying control variable fWT and tracks it across the 0.00–1.00 titration points.
- Every plotted site has coverage >= 50 in its mAFiA.sites.bed entry (min_coverage 50 filter applied).
- Each SmAFiA–SGLORI pair is confirmed to originate from intersecting mAFiA sites with the query sites listed in GLORI_chrX.bed.
- The density scatter plot renders a system-wide distribution of individual site stoichiometries spanning the fWT = 0.00–1.00 titration points.
- Outputs density_scatter_SGLORI.png and slope_report.txt are produced as specified.

## Limitations

- The comparison requires a minimum dRNA-Seq coverage of 50 per site; low-coverage sites are excluded, which reduces the number of comparable sites.
- Agreement is evaluated at the ensemble (site-level) level; the density scatter cannot capture single-molecule modification patterns that would appear identical at the ensemble level.
- The slope agreement validates stoichiometric sensitivity only within the WT/IVT titration design; other biological perturbations (e.g., METTL3 KO) require separate validation.

## Evidence

- [results] A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [methods] GLORI_chrX.bed description: "GLORI_chrX.bed: query modification sites in bed format. This file specifically corresponds to those listed in [GLORI]"
- [readme] modRatio and coverage column semantics from psi-co-mAFiA README: [evidence span withheld — anchor status: external_doc]
- [results] sklearn usage in article: "Optimization is performed with the python module sklearn, using the default L-BFGS solver19"
- [readme] GLORI comparison correlation from psi-co-mAFiA README: [evidence span withheld — anchor status: external_doc]
