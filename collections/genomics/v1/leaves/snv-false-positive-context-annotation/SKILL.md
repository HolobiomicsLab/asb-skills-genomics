---
name: snv-false-positive-context-annotation
description: Use when benchmarking long-read SNV calls against GIAB truth sets and
  you need to characterize error sequence contexts (indel-proximate FPs, homopolymer
  FNs) or evaluate an indel-proximity filter for FP reduction.
when_to_use_negative:
- Longshot cannot call indels de novo — it only calls SNVs and can genotype indels
  only if supplied via a --potential_variants VCF, so this skill will not recover
  indel variants themselves
- Truth-set comparison is meaningless where the reference truth panel itself is incomplete;
  GIAB/Platinum Genomes sets are biased in favor of short-read technologies, so some
  Longshot FPs may actually be truth-set FNs
- Structural variants >50 bp are unsupported by Longshot (--potential_variants explicitly
  excludes them), so this annotation approach cannot be applied to SV calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Longshot
  role: Diploid SNV caller for long error-prone reads; run per chromosome with --region
    and a variable GQ cutoff matched to median read coverage to produce the SNV VCFs
    that are then error-annotated and indel-filtered
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:ba576eaf0e4cce88d7d3661281d5dfe3eb283a6e3dfcb629d7ca3524fef60a25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# snv-false-positive-context-annotation

## Summary

Classify long-read SNV calls (Longshot) against GIAB high-confidence truth sets, annotate false positives by proximity to known indels and false negatives by homopolymer context, and quantify FP reduction from filtering calls near known common indels (Mills + 1000 Genomes Gold Standard Indels).

## When to use

Use when benchmarking long-read SNV calls against GIAB truth sets and you need to characterize error sequence contexts (indel-proximate FPs, homopolymer FNs) or evaluate an indel-proximity filter for FP reduction.

## When NOT to use

- Longshot cannot call indels de novo — it only calls SNVs and can genotype indels only if supplied via a --potential_variants VCF, so this skill will not recover indel variants themselves
- Truth-set comparison is meaningless where the reference truth panel itself is incomplete; GIAB/Platinum Genomes sets are biased in favor of short-read technologies, so some Longshot FPs may actually be truth-set FNs
- Structural variants >50 bp are unsupported by Longshot (--potential_variants explicitly excludes them), so this annotation approach cannot be applied to SV calls

## Inputs

- Sorted, indexed BAM/CRAM alignments of PacBio (or ONT) long reads (e.g., 45× NA12878 SMRT reads aligned to hs37d5 with BLASR)
- Indexed reference FASTA (hs37d5 or hg38)
- GIAB high-confidence truth variant call sets (VCF)
- Mills + 1000 Genomes Gold Standard Indels VCF (gzipped, tabix indexed)

## Outputs

- Per-genome Longshot SNV VCF files (callable per chromosome via --region)
- TP/FP/FN classification of each Longshot call relative to GIAB truth sets
- Per-FP annotations of distance to nearest true indel and fraction within 5 bp
- Per-FN homopolymer-context annotations and enrichment statistics (≥5 bp homopolymers)
- FP counts and recall before and after known-common-indel proximity filtering
- Compiled per-genome metrics table (error_context_table.tsv)

## How to apply

Run Longshot per chromosome on GIAB PacBio BAM alignments using the --region option with a variable GQ cutoff matched to the median read coverage, then classify calls as TP/FP/FN by set comparison against the GIAB high-confidence truth variant VCFs. Annotate each false-positive SNV with its distance to the nearest true indel and compute the fraction within 5 bp (target: 71.4% for NA12878). Annotate false negatives for homopolymer context and compute the fraction in homopolymers of length ≥5 bp versus the random expectation (~3.4× enrichment, target: 19.5%). Rationale: Longshot does not consider indels as candidate variants, so true SNVs adjacent to indels are missed and indel-associated error contexts produce FPs. Finally, filter SNV calls near known common indels (Mills + 1000 Genomes Gold Standard Indels set) and recompute FP counts and recall; expect 34–45% FP reduction with only a slight recall decrease, consistent with the paper's FDR reduction to 0.3%.

## Related tools

- **Longshot** (Diploid SNV caller for long error-prone reads; run per chromosome with --region and a variable GQ cutoff matched to median read coverage to produce the SNV VCFs that are then error-annotated and indel-filtered) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam NA12878_pacbio.bam --ref hs37d5.fa --region chr20 --out NA12878_chr20.vcf
```

## Evaluation signals

- Fraction of false positives within 5 bp of the nearest true indel should reach ~71.4% for NA12878
- Fraction of false negatives in homopolymers of length ≥5 bp should be ~19.5%, ~3.4× above the random expectation
- Filtering calls near Mills + 1000G Gold Standard indels should reduce FPs by 34–45% across the four GIAB PacBio genomes with only slight recall loss
- Post-filter FDR should drop to ~0.3% with known common indel filtering, matching the paper's reported value
- All per-genome metrics should be compiled consistently in error_context_table.tsv

## Limitations

- The GIAB and Platinum Genomes truth sets were generated using short reads and are biased in favor of short-read technologies; some Longshot false positives may correspond to false negatives in the GIAB high-confidence call sets
- FPs largely arise because Longshot does not consider indels as candidate variants, so error correction via context annotation cannot recover SNVs hidden by indel errors
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge and can inflate apparent FP counts

## Evidence

- [discussion] we find that the FDR can be reduced further to 0.3% by filtering out known common indels.: [evidence span withheld — anchor status: not_found]
- [discussion] For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants.: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] some of the false-positive calls by Longshot may actually correspond to false negatives in the GIAB high-confidence call sets.: "some of the false-positive calls by Longshot may actually correspond to false negatives in the GIAB high-confidence call sets."
- [discussion] The 45× coverage PacBio SMRT reads for NA12878, aligned to the hs37d5 reference genome using BLASR, were obtained from the GIAB consortium: [evidence span withheld — anchor status: not_found]
- [readme] Currently, it only calls single nucleotide variants (SNVs), but it can genotype indels if they are given in an input VCF.: "Currently, it only calls single nucleotide variants (SNVs), but it can genotype indels if they are given in an input VCF."
- [readme] Region in format <chrom> or <chrom:start-stop> in which to call variants (1-based, inclusive).: "Region in format <chrom> or <chrom:start-stop> in which to call variants (1-based, inclusive)."
