---
name: rna-modification-stoichiometry-quantification
description: Use when nanopore dRNA-Seq fast5 reads must be converted into per-site
  modification stoichiometry values (modRatio) for m6A sites, e.g., to compare HEK293
  WT vs METTL3-KO profiles or to regress measured stoichiometry against known titration
  fractions.
when_to_use_negative:
- Sites with dRNA-Seq coverage below the minimum threshold of 50 — the article requires
  this cutoff before comparison against GLORI
- Ensemble-level or short-read m6A profiling questions where single-molecule read-level
  resolution is not needed (e.g., consensus-only peak calling)
- Modification types outside the trained m6A DRACH motifs — for other modifications
  use the expanded Ψ-co-mAFiA models instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3524
tools:
- name: mAFiA
  role: Detects single m6A nucleotides and aggregates read-level P(m6A) >= 0.5 calls
    into site-level modRatio stoichiometry via test_mAFiA / pileup
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Fully convolutional basecaller whose backbone.torch model is used (batchsize
    4096) for basecalling fast5 files and extracting features for mAFiA
  repo: https://github.com/biodlab/RODAN
- name: minimap2
  role: Aligns basecalled reads to GRCh38 with --secondary=no -ax splice -uf -k14
    before BAM filtering
- name: samtools
  role: Filters (samtools view -bST -q50), sorts, and indexes the aligned BAM; relabels
    MM/ML tags for visualization
- name: sklearn
  role: Fits the linear regression S(mAFiA) = m*S(GLORI) + b used to validate stoichiometric
    sensitivity across the WT/IVT titration
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:cb5efa5ea7944da421c564d38e32d3d21a56608122e9a014e313193c961e6f82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - RODAN
  - mAFiA
  - minimap2
  - samtools
  - sklearn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rna-modification-stoichiometry-quantification

## Summary

Quantifies per-site RNA modification (m6A) stoichiometry from Oxford Nanopore dRNA-Seq data using mAFiA: read-level P(m6A) probabilities are thresholded at 0.5 and aggregated into a site-level modRatio after coverage filtering, enabling stoichiometric comparisons against orthogonal references such as GLORI.

## When to use

Use when nanopore dRNA-Seq fast5 reads must be converted into per-site modification stoichiometry values (modRatio) for m6A sites, e.g., to compare HEK293 WT vs METTL3-KO profiles or to regress measured stoichiometry against known titration fractions.

## When NOT to use

- Sites with dRNA-Seq coverage below the minimum threshold of 50 — the article requires this cutoff before comparison against GLORI
- Ensemble-level or short-read m6A profiling questions where single-molecule read-level resolution is not needed (e.g., consensus-only peak calling)
- Modification types outside the trained m6A DRACH motifs — for other modifications use the expanded Ψ-co-mAFiA models instead

## Inputs

- Raw Oxford Nanopore dRNA-Seq fast5 files (e.g., HEK293 WT fast5_chrX, METTL3 KO fast5 from ENA PRJEB40872)
- Backbone model backbone.torch and pickled logistic regression classifiers directory from zenodo.org/record/8321727
- Reference genome (GRCh38_96.X / GRCh38)
- Query modification sites BED file (e.g., GLORI_chrX.bed)

## Outputs

- mAFiA.sites.bed with per-site coverage, modRatio, and confidence columns
- mAFiA.reads.bam (modBAM) carrying per-read MM/ML modification tags
- Joined site-level stoichiometry comparison table (e.g., mettl3_ko_vs_wt_m6A_stoichiometry.tsv)
- Density scatter plot (density_scatter_SGLORI.png) and slope report (slope_report.txt) for titration validation

## How to apply

Basecall raw fast5 files with the RODAN-based basecalling script (basecall.py, --batchsize 4096, backbone.torch model), align to GRCh38 with minimap2 (--secondary=no -ax splice -uf -k14), and filter with samtools view -bST -q50 before sorting and indexing. Run test_mAFiA on the aligned BAM and fast5 directory against the query BED (e.g., GLORI_chrX.bed) with --min_coverage 50, --max_num_reads 1000, and --mod_prob_thresh 0.5, so that only sites with dRNA-Seq coverage >= 50 are reported and per-read nucleotides with P(m6A) >= 0.5 count as modified. The pileup step aggregates single-read probabilities into mAFiA.sites.bed with coverage, modRatio (fraction of nucleotides with modification probability >= 0.5), and confidence columns; restrict both samples to covered sites and join on genomic coordinates for cross-sample comparisons. For titration validation, fit a linear regression of aggregated S(mAFiA) versus S(GLORI) values with sklearn (L-BFGS solver) and check that the slope tracks the expected WT fraction fWT across 0.00-1.00.

## Related tools

- **mAFiA** (Detects single m6A nucleotides and aggregates read-level P(m6A) >= 0.5 calls into site-level modRatio stoichiometry via test_mAFiA / pileup) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Fully convolutional basecaller whose backbone.torch model is used (batchsize 4096) for basecalling fast5 files and extracting features for mAFiA) — https://github.com/biodlab/RODAN
- **minimap2** (Aligns basecalled reads to GRCh38 with --secondary=no -ax splice -uf -k14 before BAM filtering)
- **samtools** (Filters (samtools view -bST -q50), sorts, and indexes the aligned BAM; relabels MM/ML tags for visualization)
- **sklearn** (Fits the linear regression S(mAFiA) = m*S(GLORI) + b used to validate stoichiometric sensitivity across the WT/IVT titration)

## Examples

```
minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}; samtools index ${bam}
```

## Evaluation signals

- Every reported site has dRNA-Seq coverage above the minimum threshold of 50 and carries a non-null modRatio value
- Read-level modification calls use the P(m6A) threshold of 0.5 (mod_prob_thresh 0.5)
- In the HEK293 WT vs METTL3-KO comparison, chromosomal methylation site locations are conserved while KO shows systematic down-regulation concentrated at sites highly methylated in WT
- In the WT/IVT titration, the fitted regression slope m(S(mAFiA)/S(GLORI)) agrees with (tracks) the expected WT fraction fWT across 0.00-1.00
- BAM is correctly filtered/sorted/indexed: all retained alignments pass samtools view -q50 (mapq >= 50)

## Limitations

- A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements; low-coverage sites are excluded
- The prototype mAFiA covers only the top m6A DRACH motifs and is superseded by Ψ-co-mAFiA, which expands coverage to all 18 m6A DRACH and 16 Ψ motifs
- For high-precision measurements the Ψ-co-mAFiA README recommends a minimum coverage of 50 and confidence of 80%
- In case of discontinuous alignment to the reference, the section of the read with the longest continuous mapping is chosen, which may drop information elsewhere on the read

## Evidence

- [results] A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [results] Read-level P(m6A) threshold: [evidence span withheld — anchor status: not_found]
- [methods] samtools q50 filtering command: "samtools view -bST ${ref} -q50"
- [readme] README: min_coverage definition: [evidence span withheld — anchor status: not_found]
- [readme] README: modRatio column definition: [evidence span withheld — anchor status: not_found]
