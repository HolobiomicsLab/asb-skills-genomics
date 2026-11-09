---
name: indel-proximity-variant-filtering
description: Use when filtering SNV calls from a long-read variant caller (e.g., Longshot
  on PacBio BAMs) to reduce indel-proximate false positives, with the goal of lowering
  FDR (target ~0.3%) while minimally hurting recall.
when_to_use_negative:
- The variant caller already models indels as candidate variants (the filter targets
  errors arising because Longshot does not consider indels).
- You need to genotype/phase indels themselves — Longshot currently only calls SNVs
  and can genotype indels only if supplied in an input VCF.
- Accurate short indel calls from SMS reads are the goal — the article notes this
  remains challenging due to the high insertion/deletion error rate.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Generates the SNV call VCFs (per-chromosome via --region with variable GQ
    cutoff) that are subsequently filtered by proximity to known common indels; installed
    via Bioconda.
  repo: github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:f802c9de96c386bf98ce3c2f38f88308da1b7104e63ad0e1e4d29026abaeeba3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# indel-proximity-variant-filtering

## Summary

A post-calling filter for Longshot SNV calls on long-read (PacBio) data that removes SNVs located near known common indels (Mills + 1000 Genomes Gold Standard Indels), exploiting the fact that many indel-driven false positives arise because Longshot does not consider indels as candidate variants.

## When to use

Use when filtering SNV calls from a long-read variant caller (e.g., Longshot on PacBio BAMs) to reduce indel-proximate false positives, with the goal of lowering FDR (target ~0.3%) while minimally hurting recall.

## When NOT to use

- The variant caller already models indels as candidate variants (the filter targets errors arising because Longshot does not consider indels).
- You need to genotype/phase indels themselves — Longshot currently only calls SNVs and can genotype indels only if supplied in an input VCF.
- Accurate short indel calls from SMS reads are the goal — the article notes this remains challenging due to the high insertion/deletion error rate.

## Inputs

- Longshot SNV call VCFs from PacBio BAM alignments (hs37d5/hg38)
- Mills + 1000 Genomes Gold Standard Indels set (known common indels)
- GIAB high-confidence truth variant call sets for TP/FP/FN classification

## Outputs

- Filtered SNV VCF with indel-proximate calls removed
- error_context_table.tsv with per-genome FP-within-5bp-of-indel fractions, FN homopolymer fractions, FP reduction (%), and recall change

## How to apply

First call SNVs with Longshot per chromosome using the --region option and a variable GQ cutoff matched to the median read coverage, then classify calls as TP/FP/FN against GIAB high-confidence truth sets. Annotate each false-positive SNV with its distance to the nearest true indel (e.g., Mills + 1000 Genomes Gold Standard Indels set) and compute the fraction within 5 bp (71.4% for NA12878). Apply a proximity filter for known common indels to the VCF and recompute FP counts, FDR, and recall; in the paper this reduced FDR to 0.3% and cut FPs by 34–45% across the four GIAB genomes at only a slight recall cost. The rationale is that Longshot only calls SNVs and does not consider indels as candidate variants, so indel-related errors masquerade as nearby SNV FPs.

## Related tools

- **Longshot** (Generates the SNV call VCFs (per-chromosome via --region with variable GQ cutoff) that are subsequently filtered by proximity to known common indels; installed via Bioconda.) — github.com/pjedge/longshot

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out longshot_chr20.vcf -r chr20
```

## Evaluation signals

- Fraction of false-positive SNVs within 5 bp of a known true indel should approach the reported 71.4% (NA12878).
- FP reduction after filtering known common indels should fall in the reported 34–45% range across the four GIAB genomes.
- Filtered-call FDR should drop to approximately 0.3%, consistent with the paper's reported value.
- Recall loss from the filter should be small (only slightly decreased recall).
- FP reduction must be reproducible when recomputed from the filtered VCF via set comparison against GIAB truth sets.

## Limitations

- Some Longshot false positives may actually correspond to false negatives in the GIAB high-confidence call sets, which are biased in favor of short-read technologies.
- Accurate calling of short indels using SMS reads remains challenging due to the high insertion/deletion error rate.
- Filtering slightly decreases recall, so truly indel-proximate real SNVs can be lost.
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, which can contribute to residual errors.

## Evidence

- [discussion] FDR reduced by filtering known common indels: [evidence span withheld — anchor status: not_found]
- [discussion] Motivation: indel calling challenges: "accurate calling of short indels using SMS reads is challenging due to the high insertion/deletion error rate."
- [readme] Longshot SNV-only scope: "Currently, it only calls single nucleotide variants (SNVs), but it can genotype indels if they are given in an input VCF."
- [discussion] Variable GQ cutoff for Longshot calls: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
