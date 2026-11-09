---
name: m6a-motif-enrichment-quantification
description: Use when you have CHEUI-solo site-level methylation probabilities from
  direct RNA sequencing (DRS) and need to compute DRACH motif enrichment and significant
  m6A site counts at a fixed probability cutoff.
when_to_use_negative:
- Differential methylation between two conditions — use CHEUI-diff instead of single-condition
  site calling with motif enrichment
- The input is not transcriptome-aligned nanopore signal (e.g. no eventaligned 5mer
  signals), since Model 1/2 inference requires signal-to-transcript aligned features
- You need higher sensitivity than strict cutoffs allow — the paper notes these very
  strict cut-offs reduce CHEUI sensitivity, and sites detected only by nanopore still
  require independent validation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: CHEUI
  role: Two-stage deep learning (Model 1 per-read prediction, Model 2 CNN site-level
    stoichiometry/probability) producing the m6A site calls at probability >=0.9999
  repo: https://github.com/comprna/CHEUI-public
- name: R2Dtool
  role: r2d liftover and r2d annotate to transpose transcriptome-mapped m6A sites
    to GRCh38 genomic coordinates and annotate them for motif/context analysis
  repo: https://github.com/comprna/R2Dtool
- name: nanopore direct RNA sequencing
  role: 'Data source: ionic-current signals from HEK293 WT and IVT-control DRS replicates
    processed by CHEUI'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:e027a669cfc3e7bc6de7178fa91f1634a0900a43219e85fa6edcfcda91171103
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - R2Dtool
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# m6a-motif-enrichment-quantification

## Summary

Quantifies DRACH motif enrichment and site counts among CHEUI-solo m6A calls from nanopore direct RNA sequencing, using a probability >0.9999 site-level cutoff, and benchmarks the resulting call set (10,036 sites, 85.12% DRACH) against the published HEK293 WT reference.

## When to use

Use when you have CHEUI-solo site-level methylation probabilities from direct RNA sequencing (DRS) and need to compute DRACH motif enrichment and significant m6A site counts at a fixed probability cutoff.

## When NOT to use

- Differential methylation between two conditions — use CHEUI-diff instead of single-condition site calling with motif enrichment
- The input is not transcriptome-aligned nanopore signal (e.g. no eventaligned 5mer signals), since Model 1/2 inference requires signal-to-transcript aligned features
- You need higher sensitivity than strict cutoffs allow — the paper notes these very strict cut-offs reduce CHEUI sensitivity, and sites detected only by nanopore still require independent validation

## Inputs

- CHEUI-solo Model 1 per-read probability distributions for transcriptomic A sites (coverage >20 reads)
- Preprocessed 5 x 20 median signal vectors and per-5mer signal distance vectors from nanopore DRS (GRCh38/Gencode v38)
- Gencode v38 GTF annotation for R2Dtool liftover of transcriptomic sites to genomic coordinates
- IVT-control DRS replicates (in-vitro transcribed WT HeLa) for FPR estimation

## Outputs

- m6A_sites.tsv: significant m6A transcriptomic site calls at probability >=0.9999
- m6A_site_metrics.tsv: site count, DRACH motif proportion, and IVT-derived m6A FPR
- Genomic-coordinate BED of lifted-over m6A sites via R2Dtool

## How to apply

Load preprocessed HEK293 WT DRS signal features (5 x 20 median signal vectors and per-5mer signal distance vectors from CHEUI-solo Model 1 inference on replicates aligned to GRCh38/Gencode v38), then run CHEUI-solo Model 2 (CNN, Keras/TensorFlow) on Model 1 per-read probability distributions for transcriptomic A sites with coverage >20 reads. Apply the significance cutoff probability >=0.9999, which was established by shuffling read signals across sites to keep the estimated m6A FDR at 0, and retain the significant sites. Compute the DRACH motif proportion among the significant sites and optionally lift transcriptomic sites to genomic coordinates with R2Dtool (r2d liftover using the Gencode v38 GTF) to deduplicate 10,036 transcriptomic sites into ~8776 genomic sites. As a negative control, run the same Models 1 and 2 pipeline with the same 0.9999 cutoff on IVT transcribed HeLa replicates to estimate the m6A FPR (expected ~0.0003).

## Related tools

- **CHEUI** (Two-stage deep learning (Model 1 per-read prediction, Model 2 CNN site-level stoichiometry/probability) producing the m6A site calls at probability >=0.9999) — https://github.com/comprna/CHEUI-public
- **R2Dtool** (r2d liftover and r2d annotate to transpose transcriptome-mapped m6A sites to GRCh38 genomic coordinates and annotate them for motif/context analysis) — https://github.com/comprna/R2Dtool
- **nanopore direct RNA sequencing** (Data source: ionic-current signals from HEK293 WT and IVT-control DRS replicates processed by CHEUI)

## Examples

```
r2d liftover -H -g GRCh38.110.gtf -i m6A_isoform_sites.bed > m6A_sites_genomic.bed
```

## Evaluation signals

- Reproduced call set contains ~10,036 significant m6A transcriptomic sites (probability >=0.9999)
- DRACH motif proportion among significant sites is ~85.12%
- Lifted-to-genomic site count is ~8776 genomic sites on 3905 transcripts
- IVT-control-derived m6A FPR matches the reported value of 0.0003
- FDR at the 0.9999 cutoff is 0 as established by the read-signal shuffling procedure

## Limitations

- Very strict cutoffs used to maintain low expected FDR reduce CHEUI sensitivity; relaxing thresholds would recover more sites but at the cost of potential false positives
- Many sites identified by orthogonal methods had high CHEUI probabilities below the set thresholds
- Nanopore signal variability limits read-level discrimination, as modified vs non-modified read differences can be comparable to variation among non-modified reads
- Sites detected only from nanopore reads require independent validation experiments

## Evidence

- [results] 10,036 significant sites with 85.12% DRACH: [evidence span withheld — anchor status: not_found]
- [methods] cutoff rationale via shuffling: [evidence span withheld — anchor status: not_found]
- [discussion] strict cutoff trade-off: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [readme] R2Dtool liftover usage: [evidence span withheld — anchor status: not_found]
