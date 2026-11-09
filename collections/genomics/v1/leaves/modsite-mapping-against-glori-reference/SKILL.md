---
name: modsite-mapping-against-glori-reference
description: Use when validating direct-RNA m6A site stoichiometry calls (mAFiA modRatio)
  against the orthogonal GLORI chemical assay, or calibrating measured stoichiometry
  against a known mixing fraction such as HEK293 WT/IVT titration points fWT = 0.00-1.00.
when_to_use_negative:
- Sites fall below the minimum dRNA-Seq coverage of 50 required for comparison against
  GLORI measurements.
- The orthogonal reference is not GLORI-based m6A stoichiometry (e.g. PRAISE pseudouridine
  sites or read-level probabilities rather than site-level ratios).
- The goal is single-molecule modification pattern detection rather than ensemble
  site-level stoichiometry calibration.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: mAFiA
  role: Produces per-site stoichiometry (modRatio) and coverage columns in mAFiA.sites.bed
    via the pileup step of the modBAM
  repo: https://github.com/dieterich-lab/mAFiA
- name: sklearn
  role: Fits the linear regression SmAFiA = m*SGLORI + b to obtain the slope m(SmAFiA/SGLORI)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:62a9cf7a6f11096276d6c5ae35d747cb744ee904ef5744963107fe33f6b76c3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - mAFiA
  - sklearn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# modsite-mapping-against-glori-reference

## Summary

Maps mAFiA site-level m6A stoichiometries (modRatio from mAFiA.sites.bed) onto the GLORI reference site list (GLORI_chrX.bed) and fits SmAFiA = m*SGLORI + b via sklearn linear regression to validate stoichiometric sensitivity, e.g. across HEK293 WT/IVT titration fractions where the slope should track fWT.

## When to use

Use when validating direct-RNA m6A site stoichiometry calls (mAFiA modRatio) against the orthogonal GLORI chemical assay, or calibrating measured stoichiometry against a known mixing fraction such as HEK293 WT/IVT titration points fWT = 0.00-1.00.

## When NOT to use

- Sites fall below the minimum dRNA-Seq coverage of 50 required for comparison against GLORI measurements.
- The orthogonal reference is not GLORI-based m6A stoichiometry (e.g. PRAISE pseudouridine sites or read-level probabilities rather than site-level ratios).
- The goal is single-molecule modification pattern detection rather than ensemble site-level stoichiometry calibration.

## Inputs

- mAFiA.sites.bed per titration sample (columns: coverage, modRatio, confidence)
- GLORI_chrX.bed: query modification sites in bed format from GLORI (Liu et al., Nat Biotechnol 2023)
- HEK293 WT/IVT titration series at fractions fWT = 0.00-1.00

## Outputs

- density_scatter_SGLORI.png: 2D density scatter plot of SmAFiA vs SGLORI
- slope_report.txt: fitted regression slope m(SmAFiA/SGLORI) from sklearn
- Per-fraction paired site table (SmAFiA, SGLORI) for sites with coverage >= 50

## How to apply

Load per-site stoichiometry from the modRatio column of each sample's mAFiA.sites.bed, keeping only sites with coverage >= 50 (min_coverage 50), since mAFiA requires a minimum dRNA-Seq coverage of 50 for a site to be compared against GLORI. Intersect these sites with the query sites in GLORI_chrX.bed to pair each mAFiA site-level stoichiometry with its GLORI reference value. Aggregate paired values into SmAFiA and SGLORI distributions per titration fraction, render a 2D density scatter of SmAFiA vs SGLORI, and fit SmAFiA = m*SGLORI + b with the python module sklearn. Validation: the fitted slope m(SmAFiA/SGLORI) should largely agree with the underlying WT fraction fWT across 0.00-1.00; save density_scatter_SGLORI.png and slope_report.txt.

## Related tools

- **mAFiA** (Produces per-site stoichiometry (modRatio) and coverage columns in mAFiA.sites.bed via the pileup step of the modBAM) — https://github.com/dieterich-lab/mAFiA
- **sklearn** (Fits the linear regression SmAFiA = m*SGLORI + b to obtain the slope m(SmAFiA/SGLORI))

## Examples

```
pileup --bam_file ${out_dir}/mAFiA.reads.bam --sites ${data}/GLORI_chrX.bed --min_coverage 50 --out_dir ${out_dir} --num_jobs 36
```

## Evaluation signals

- The regression slope m(SmAFiA/SGLORI) agrees with the underlying control variable fWT across the titration series 0.00-1.00.
- Every retained site passes the min_coverage 50 filter before pairing with GLORI sites.
- Only sites listed in GLORI_chrX.bed are retained after intersection with the mAFiA site list.
- The pileup outputs the expected mAFiA.sites.bed schema with coverage, modRatio, and confidence columns.
- The fitted slope on HEK293 WT vs GLORI reproduces the reported high correlation between overlapping m6A sites (15928 sites, correlation 0.915 for the Ψ-co-mAFiA benchmark).

## Limitations

- A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI; sites below this cut-off must be dropped, reducing the usable site set.
- For high-precision measurements the README recommends a minimum coverage of 50 and confidence of 80%; low-confidence sites degrade the regression.
- GLORI_chrX.bed covers only the chromosome X subset in the example data, so the comparison is limited to sites on that contig.

## Evidence

- [results] A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [readme] GLORI_chrX.bed is the query modification sites file: "GLORI_chrX.bed: query modification sites in bed format. This file specifically corresponds to those listed in [GLORI](https://www.nature.com/articles/s41587-022-01487-9)."
- [readme] modRatio column definition: [evidence span withheld — anchor status: external_doc]
- [readme] Coverage and confidence recommendations for high-precision measurements: [evidence span withheld — anchor status: external_doc]
