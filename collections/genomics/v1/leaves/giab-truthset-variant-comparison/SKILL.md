---
name: giab-truthset-variant-comparison
description: Use when a long-read SNV VCF must be scored against GIAB high-confidence
  truth sets, or when accuracy (precision/recall) must be tabulated across sub-sampled
  coverages for GIAB samples (NA12878, NA24385, NA24149, NA24143).
when_to_use_negative:
- The caller output is a structural variant callset (Longshot only calls SNVs; SVs
  >50 bp are unsupported)
- Truth sets are unavailable or the sample has no GIAB high-confidence regions — accuracy
  cannot be computed by set comparison
- The goal is de novo discovery rather than benchmarking — comparison to a short-read-biased
  truth set would discard genuinely novel calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: Longshot
  role: Produces the SNV VCFs (phased, with GQ scores) that are classified against
    GIAB truth sets; run per chromosome with --region and the -A auto max-coverage
    flag
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Internal phasing engine used by Longshot during haplotype-informed genotyping
    of candidate SNVs prior to truth-set comparison
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:bf392ce1b20fa712c6cfd4398ccf22f5012db5beebfb4f501153c279661000c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - HapCUT2
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# giab-truthset-variant-comparison

## Summary

Benchmark SNV caller output (e.g., Longshot VCFs on GIAB PacBio/BLASR alignments) against Genome in a Bottle high-confidence truth sets to compute per-sample precision and recall, including coverage-dependent accuracy. Also supports TP/FP/FN classification for downstream error-context analyses such as indel-proximate FPs and homopolymer FNs.

## When to use

Use when a long-read SNV VCF must be scored against GIAB high-confidence truth sets, or when accuracy (precision/recall) must be tabulated across sub-sampled coverages for GIAB samples (NA12878, NA24385, NA24149, NA24143).

## When NOT to use

- The caller output is a structural variant callset (Longshot only calls SNVs; SVs >50 bp are unsupported)
- Truth sets are unavailable or the sample has no GIAB high-confidence regions — accuracy cannot be computed by set comparison
- The goal is de novo discovery rather than benchmarking — comparison to a short-read-biased truth set would discard genuinely novel calls

## Inputs

- Longshot VCF of called SNVs (filtered at the coverage-matched GQ cutoff)
- GIAB high-confidence truth variant call sets (VCF) for NA12878, NA24385, NA24149, NA24143
- Sub-sampled GIAB PacBio BAM alignments (BLASR-aligned; e.g., NA24385 28x/37x/46x/64x, NA12878 30x/45x)
- Indexed reference FASTA (hs37d5 or hg38)

## Outputs

- Per-sample precision and recall values for SNVs (longshot_giab_accuracy_metrics.csv)
- TP/FP/FN classified variant lists per genome for error-context analysis
- Coverage-dependent accuracy table (e.g., NA24385 recall 0.9608 at 28x rising to 0.9798 at 64x)

## How to apply

Run the caller (Longshot with min_mapq 20, min_cov 6, min_alt_count 3, min_alt_frac 0.125, and -A auto max-coverage of mean + 5*sqrt(mean)) on GIAB PacBio BAMs, then classify each call as TP, FP, or FN by set comparison against the GIAB high-confidence call sets. Apply the paper's variable GQ cutoff matched to the median read coverage for Longshot variants (GQ fixed at 50 for short-read comparator sets like GIAB/Platinum Genomes). Compare per chromosome using the same reference (hs37d5 for NA12878, hg38 for the AJ trio) as used for BLASR alignment. Tabulate precision and recall per sample and coverage; note that GIAB truth sets are short-read-derived and therefore biased in favor of short-read technologies, so some Longshot FPs may be GIAB false negatives.

## Related tools

- **Longshot** (Produces the SNV VCFs (phased, with GQ scores) that are classified against GIAB truth sets; run per chromosome with --region and the -A auto max-coverage flag) — https://github.com/pjedge/longshot
- **HapCUT2** (Internal phasing engine used by Longshot during haplotype-informed genotyping of candidate SNVs prior to truth-set comparison) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf -A --region chr20
```

## Evaluation signals

- NA12878 30x should reproduce P=0.9942 / R=0.9592; 45x recall should reach R=0.9734
- NA24385 recall should rise monotonically from 0.9608 at 28x to 0.9798 at 64x with only moderate precision change
- Every candidate SNV counted as a call must pass the Fisher's exact strand-bias filter (two-tailed p < 0.01) before genotyping
- The GQ cutoff used for Longshot variants must equal the median read coverage of the dataset; short-read truth/comparator sets use GQ 50
- FP lists should be re-checkable: ~71.4% of NA12878 FPs lie within 5 bp of a true indel, and filtering known common indels reduces FDR to ~0.3% (34–45% FP reduction)

## Limitations

- GIAB and Platinum Genomes truth sets were generated with short-read data and are biased in favor of short-read technologies, so Longshot FPs may be truth-set false negatives
- Allele quality filtering (phred threshold 7.0 by default) reduces the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge
- Accuracy estimates are SNV-only; Longshot does not call indels de novo because accurate short indel calling with SMS reads is challenging

## Evidence

- [discussion] High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling: "High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling"
- [discussion] For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants.: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] GIAB truth set bias caveat: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets, are therefore biased in favor of short-read technologies"
- [readme] README usage line: "$ longshot [FLAGS] [OPTIONS] --bam <BAM/CRAM> --ref <FASTA> --out <VCF>"
- [discussion] Strand bias filter: [evidence span withheld — anchor status: not_found]
