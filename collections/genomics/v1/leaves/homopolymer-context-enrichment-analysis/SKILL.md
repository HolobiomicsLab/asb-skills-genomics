---
name: homopolymer-context-enrichment-analysis
description: Use when auditing long-read (PacBio/ONT) SNV call sets against a truth
  set (e.g., GIAB) to determine whether false positives or false negatives are enriched
  in specific sequence contexts such as homopolymers or indel-proximate sites.
when_to_use_negative:
- The goal is calling short indels themselves with long reads — the paper notes accurate
  calling of short indels using SMS reads is challenging due to the high insertion/deletion
  error rate, and Longshot calls only SNVs from pileup.
- Input alignments are short-read (Illumina) calls — the FP-near-indel and FN-in-homopolymer
  patterns characterized here are specific to SMS read error profiles and Longshot's
  indel-blind candidate selection.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Calls phased SNVs from PacBio BAMs whose FP/FN error contexts are then annotated;
    installed via Bioconda and run per chromosome with --region
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:dbc7dcb3bd928a412ded8ea13e8be0f0fa4457f3806c3c9818ffd1f50223127f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# homopolymer-context-enrichment-analysis

## Summary

Characterize sequence-context biases in long-read SNV call errors by annotating Longshot false-negative SNVs for homopolymer context and false-positive SNVs for proximity to known indels on GIAB PacBio genomes, then quantifying enrichment over random expectation. Grounded in Edge & Bansal 2019, where FNs concentrated in homopolymers of length ≥5 bp and FPs clustered within 5 bp of true indels because Longshot does not consider indels as candidate variants.

## When to use

Use when auditing long-read (PacBio/ONT) SNV call sets against a truth set (e.g., GIAB) to determine whether false positives or false negatives are enriched in specific sequence contexts such as homopolymers or indel-proximate sites.

## When NOT to use

- The goal is calling short indels themselves with long reads — the paper notes accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate, and Longshot calls only SNVs from pileup.
- Input alignments are short-read (Illumina) calls — the FP-near-indel and FN-in-homopolymer patterns characterized here are specific to SMS read error profiles and Longshot's indel-blind candidate selection.

## Inputs

- Longshot SNV call VCFs from PacBio BAM alignments (per-genome, per-chromosome)
- GIAB high-confidence truth variant call sets
- Mills + 1000 Genomes Gold Standard Indels VCF
- Indexed reference FASTA (e.g., hs37d5)

## Outputs

- error_context_table.tsv with per-genome FN-in-homopolymer and FP-near-indel fractions
- FP-within-5bp-of-indel fraction and FN-in-homopolymer(≥5bp) fraction per genome
- FP reduction and recall change after known-common-indel proximity filtering

## How to apply

First run Longshot on the PacBio BAM alignments (e.g., the 45× NA12878 hs37d5 BLASR alignments), processing chromosomes separately with the --region option and a variable GQ cutoff matched to the median read coverage. Classify calls as TP/FP/FN by set comparison against the GIAB high-confidence call sets. For each false-negative SNV, annotate whether it falls in a homopolymer of length ≥5 bp in the reference and compare the observed fraction to the random expectation (paper reports 19.5% vs ~3.4× enrichment for NA12878). For each false-positive SNV, annotate the distance to the nearest true indel (Mills + 1000 Genomes Gold Standard Indels) and compute the fraction within 5 bp (target 71.4% for NA12878). As a decision point, apply a proximity filter against known common indels and recompute FP counts and recall to quantify FP reduction (34–45% across four GIAB genomes; FDR reducible to 0.3%) versus recall cost. Compile per-genome context metrics into a table such as error_context_table.tsv.

## Related tools

- **Longshot** (Calls phased SNVs from PacBio BAMs whose FP/FN error contexts are then annotated; installed via Bioconda and run per chromosome with --region) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf --region chr20
```

## Evaluation signals

- Fraction of FP SNVs within 5 bp of a true indel should approximate 71.4% for NA12878
- Fraction of FN SNVs in homopolymers of length ≥5 bp should approximate 19.5%, about 3.4× the random expectation
- Filtering calls near Mills + 1000 Genomes Gold Standard indels should reduce FPs by 34–45% across the four GIAB genomes with only a slight recall decrease
- FDR after known-common-indel filtering should be consistent with the reported 0.3%
- TP/FP/FN set-comparison counts must reconcile exactly with the input VCF and GIAB truth set membership

## Limitations

- Longshot does not consider indels as candidate variants, which is the mechanism producing the indel-proximate FPs measured here
- Some Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets, since those truth sets are biased in favor of short-read technologies
- Accurate calling of short indels with SMS reads is challenging due to the high insertion/deletion error rate
- Allele quality filtering (phred-scaled threshold 7.0 by default) reduces the effective read depth for SMS reads, which can affect error-context counts

## Evidence

- [discussion] we find that the FDR can be reduced further to 0.3% by filtering out known common indels.: [evidence span withheld — anchor status: not_found]
- [discussion] accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate.: "accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate."
- [discussion] For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants.: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [readme] Longshot takes as input an aligned BAM/CRAM file and outputs a phased VCF file with variants and haplotype information.: "It takes as input an aligned BAM/CRAM file and outputs a phased VCF file with variants and haplotype information."
- [discussion] The 45× coverage PacBio SMRT reads for NA12878, aligned to the hs37d5 reference genome using BLASR, were obtained from the GIAB consortium: [evidence span withheld — anchor status: not_found]
