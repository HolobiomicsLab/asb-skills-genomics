---
name: m6a-method-benchmarking-against-glori
description: Use when you have a bed file of predicted m6A site stoichiometries (e.g.
  mAFiA.sites.bed with modRatio) and an orthogonal reference such as GLORI_chrX.bed,
  and need to compute and compare site-level correlation benchmarks.
when_to_use_negative:
- Sites with dRNA-Seq coverage below the minimum threshold of 50 — the paper explicitly
  requires a minimum coverage of 50 for comparison against GLORI measurements
- Benchmarking read-level modification probabilities rather than site-level stoichiometry
  — this skill operates on aggregated mAFiA.sites.bed output, not single-read P(m6A)
  values
- Comparing Ψ or other RNA modifications against GLORI — GLORI measures m6A only;
  Ψ benchmarks use PRAISE
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: mAFiA
  role: Produces mAFiA.sites.bed site-level stoichiometry (modRatio) whose correlation
    against GLORI is the benchmarked quantity; run via RODAN basecalling, minimap2
    alignment, process_reads and pileup
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Fully convolutional basecaller used upstream to generate the basecalls and
    hidden features from which mAFiA predicts read-level m6A probabilities
  repo: https://github.com/biodlab/RODAN
- name: psi-co-mAFiA
  role: Superseding tool whose pileup command generates the same mAFiA.sites.bed stoichiometry
    format used in benchmarking against GLORI (15928 overlapping sites, correlation
    0.915 in the updated README)
  repo: https://github.com/dieterich-lab/psi-co-mAFiA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:53bd49e4e027d7ce790fe1ada3c91eccfdccddf20e95af24c1a0a32c820e74f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - RODAN
  - mAFiA
  - psi-co-mAFiA
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# m6a-method-benchmarking-against-glori

## Summary

Benchmarks site-level m6A stoichiometry predictions from dRNA-Seq methods (mAFiA, CHEUI, m6Anet) against GLORI chemical-measurement stoichiometry by matching sites on genomic coordinates and computing Pearson/Spearman correlations. Reproduces the paper's headline result of mAFiA vs GLORI correlation 0.86 on 5925 HEK293 WT transcriptome sites.

## When to use

Use when you have a bed file of predicted m6A site stoichiometries (e.g. mAFiA.sites.bed with modRatio) and an orthogonal reference such as GLORI_chrX.bed, and need to compute and compare site-level correlation benchmarks.

## When NOT to use

- Sites with dRNA-Seq coverage below the minimum threshold of 50 — the paper explicitly requires a minimum coverage of 50 for comparison against GLORI measurements
- Benchmarking read-level modification probabilities rather than site-level stoichiometry — this skill operates on aggregated mAFiA.sites.bed output, not single-read P(m6A) values
- Comparing Ψ or other RNA modifications against GLORI — GLORI measures m6A only; Ψ benchmarks use PRAISE

## Inputs

- mAFiA.sites.bed (bed file with coverage, modRatio, confidence columns from the pileup step)
- GLORI_chrX.bed (query modification sites in bed format with stoichiometry values, from zenodo.org/record/8321727)
- HEK293 WT dRNA-Seq chrX fast5 data and GRCh38_96.X genome reference (for upstream mAFiA run)

## Outputs

- Matched benchmark table of 5925 genomic sites with mAFiA modRatio and GLORI stoichiometry
- Pearson (and Spearman) correlation coefficients between predicted and GLORI stoichiometry (expected ~0.86 for mAFiA)
- Comparison table including published m6Anet (0.80) and CHEUI (0.64) correlations

## How to apply

Load mAFiA.sites.bed produced by the pileup step, retaining only sites with coverage at or above the minimum threshold of 50, and extract the 'modRatio' column (the ratio of single nucleotides at the site with modification probability >= 0.5). Load GLORI_chrX.bed, the bed-format query modification sites corresponding to those listed in the GLORI publication. Match sites between the two files by genomic coordinate (chromosome, position, strand) to obtain the 5925-site HEK293 WT benchmark set. Compute Pearson (and Spearman) correlation between mAFiA modRatio and GLORI stoichiometry over the matched sites, then tabulate alongside the published comparators m6Anet = 0.80 and CHEUI = 0.64 on the same site set.

## Related tools

- **mAFiA** (Produces mAFiA.sites.bed site-level stoichiometry (modRatio) whose correlation against GLORI is the benchmarked quantity; run via RODAN basecalling, minimap2 alignment, process_reads and pileup) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Fully convolutional basecaller used upstream to generate the basecalls and hidden features from which mAFiA predicts read-level m6A probabilities) — https://github.com/biodlab/RODAN
- **psi-co-mAFiA** (Superseding tool whose pileup command generates the same mAFiA.sites.bed stoichiometry format used in benchmarking against GLORI (15928 overlapping sites, correlation 0.915 in the updated README)) — https://github.com/dieterich-lab/psi-co-mAFiA

## Examples

```
pileup --bam_file ${out_dir}/mAFiA.reads.bam --sites ${sites} --min_coverage 50 --out_dir ${out_dir} --num_jobs 36
```

## Evaluation signals

- The matched site table must contain exactly 5925 HEK293 WT transcriptome sites
- Recomputed Pearson correlation must reproduce S_mAFiA/S_GLRORI = 0.86 within numerical tolerance
- Recomputed correlations for CHEUI and m6Anet on the same site set must match 0.64 and 0.80 respectively
- Every matched site must have mAFiA coverage >= 50 (the minimum dRNA-Seq coverage required for comparison against GLORI measurements)
- Coordinate matching must respect chromosome, position and strand; mismatches between strand fields must be excluded

## Limitations

- The comparison is restricted to chrX data shipped in the zenodo subset (zenodo.org/record/8321727); the full HEK293 WT dataset for all chromosomes is in a separate deposit (zenodo.org/record/8319583), so genome-wide reproduction needs the larger dataset
- For high-precision stoichiometry measurements the README recommends minimum coverage of 50 and confidence of 80%; relaxed thresholds will shift correlations away from the published 0.86
- The later psi-co-mAFiA README reports a different benchmark configuration (15928 sites, coverage >= 10, confidence >= 80%, correlation 0.915), so results are not directly comparable across the two tool versions

## Evidence

- [results] Minimum coverage of 50 required for a site to be compared against GLORI: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [methods] GLORI chrX reference file description: "GLORI_chrX.bed: query modification sites in bed format. This file specifically corresponds to those listed in [GLORI]"
- [readme] modRatio definition from README: [evidence span withheld — anchor status: external_doc]
- [readme] README benchmark figures against GLORI: [evidence span withheld — anchor status: external_doc]
- [methods] Full HEK293 WT dataset availability: "The complete HEK293 WT dataset for all chromosomes can be downloaded from [zenodo](https://zenodo.org/record/8319583)."
