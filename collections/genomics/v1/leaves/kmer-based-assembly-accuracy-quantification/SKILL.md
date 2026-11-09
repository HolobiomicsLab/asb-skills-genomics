---
name: kmer-based-assembly-accuracy-quantification
description: Use when you need to assess whether a polishing tool improved or degraded
  a HiFi/T2T assembly, by measuring QV and counting changed K-mers and potential overcorrection
  K-mers against short-read k-mer datasets.
when_to_use_negative:
- You only need variant-level error counts against a reference benchmark (e.g., GIAB
  high-confidence variants for HG005 vs GRCh37) — use DeepVariant/paftools comparison
  instead of k-mer ratios
- The assembly is not from HiFi long reads and lacks matching high-coverage short
  reads, since NextPolish2's k-mer checking 'relies heavily on short reads' and yak
  datasets require >=60X short-read coverage
- You are evaluating structural/haplotype switch errors rather than base-level consensus
  accuracy — the article notes switch errors are assessed separately via the built-in
  phasing evaluation, not via k-mer overcorrection counts
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: yak
  role: Builds the short-read k-mer ground-truth dataset files (k21.yak, k31.yak)
    from >=60X short reads used to check whether k-mers contain errors
  repo: https://github.com/lh3/yak
- name: meryl
  role: Counts k=15 k-mers in the unpolished and polished assemblies to establish
    the pre/post-polishing k-mer baseline and detect changed k-mers
- name: NextPolish2
  role: Produces the polished assembly being evaluated (asm.np2.fa) that is compared
    against the k-mer ground truth
  repo: https://github.com/Nextomics/NextPolish2
- name: Merqury
  role: One of the three evaluation approaches, used to assess the assembly quality
    value (QV) before and after polishing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:9daf582a861355b2122f19c7d2e607351fc247831c0589f2927bb3e8b7ca4516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/yak
  tools:
  - Merqury
  - NextPolish2
  - meryl
  - yak
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# kmer-based-assembly-accuracy-quantification

## Summary

Quantifies HiFi assembly consensus accuracy (QV) and polishing-induced overcorrection by comparing k-mers of unpolished and polished assemblies against a short-read k-mer ground truth built with yak and counted with meryl. Used in the NextPolish2 benchmark to show RM introduces ~5.31–547.64x more changed K-mers than NextPolish2 on human assemblies without QV gains.

## When to use

Use when you need to assess whether a polishing tool improved or degraded a HiFi/T2T assembly, by measuring QV and counting changed K-mers and potential overcorrection K-mers against short-read k-mer datasets.

## When NOT to use

- You only need variant-level error counts against a reference benchmark (e.g., GIAB high-confidence variants for HG005 vs GRCh37) — use DeepVariant/paftools comparison instead of k-mer ratios
- The assembly is not from HiFi long reads and lacks matching high-coverage short reads, since NextPolish2's k-mer checking 'relies heavily on short reads' and yak datasets require >=60X short-read coverage
- You are evaluating structural/haplotype switch errors rather than base-level consensus accuracy — the article notes switch errors are assessed separately via the built-in phasing evaluation, not via k-mer overcorrection counts

## Inputs

- Unpolished HiFi (or T2T) genome assembly in FASTA/FASTA.gz format
- Polished assemblies (e.g., NextPolish2-polished asm.np2.fa and RM-polished assembly)
- High-coverage short reads (>=60X, e.g., paired-end Illumina FASTQ, QC-filtered with fastp)
- yak k-mer dataset files k21.yak and k31.yak built from short reads

## Outputs

- meryl k=15 k-mer count databases (merylDB directories) for unpolished and each polished assembly
- Per-assembly QV estimates (e.g., Merqury output; NextPolish2 improved HG005 QV from 53.8544 to 55.6257)
- Counts of changed K-mers and potential overcorrection K-mers per polished assembly
- Fold-change ratios of RM over NextPolish2 for both metrics, written to overcorrection_kmer_counts.csv

## How to apply

First build short-read k-mer ground truth with yak from high-coverage (>=60X recommended) short reads, producing 21-mer and 31-mer dataset files (k21.yak, k31.yak). Then count k=15 k-mers in the unpolished HiFi assembly and in each polished assembly separately using 'meryl count k=15 output merylDB asm.fa.gz'. Classify k-mers relative to the short-read truth into 'changed K-mers' (k-mers altered by polishing) and 'potential overcorrection K-mers' (k-mers incorrectly altered) per polished assembly. Compute per-assembly QV with Merqury as one of three evaluation approaches (alongside DeepVariant and Paftools.js variant counting). Finally compute fold-change ratios of RM over NextPolish2 for both k-mer metrics; on human datasets the validation expectation is changed K-mers ~5.31–547.64x and potential overcorrection K-mers ~4.98–63.33x, with no significant QV improvement from RM, indicating overcorrection of authentic base sequences.

## Related tools

- **yak** (Builds the short-read k-mer ground-truth dataset files (k21.yak, k31.yak) from >=60X short reads used to check whether k-mers contain errors) — https://github.com/lh3/yak
- **meryl** (Counts k=15 k-mers in the unpolished and polished assemblies to establish the pre/post-polishing k-mer baseline and detect changed k-mers)
- **NextPolish2** (Produces the polished assembly being evaluated (asm.np2.fa) that is compared against the k-mer ground truth) — https://github.com/Nextomics/NextPolish2
- **Merqury** (One of the three evaluation approaches, used to assess the assembly quality value (QV) before and after polishing)

## Examples

```
meryl count k=15 output merylDB asm.fa.gz
```

## Evaluation signals

- RM/NextPolish2 fold-change of changed K-mers falls within ~5.31–547.64x on human datasets
- RM/NextPolish2 fold-change of potential overcorrection K-mers falls within ~4.98–63.33x
- Polishing should improve QV (e.g., NextPolish2 raised HG005 QV from 53.8544 to 55.6257) without large increases in changed/overcorrection K-mers
- Both 21-mer and 31-mer yak datasets were built from short reads at >=60X coverage before polishing evaluation
- All per-assembly counts and ratios are present in overcorrection_kmer_counts.csv

## Limitations

- NextPolish2's performance 'relies heavily on short reads'; uncorrected errors can arise from biases inherent in short-read sequencing technologies, and no specific coverage threshold beyond the >=60X recommendation is given
- RM produced large excesses of changed/overcorrection K-mers without any significant QV improvement, so QV alone is insufficient to detect overcorrection — k-mer classification is needed
- K-mer metrics were benchmarked on human assemblies (HG005, HG002, CHM13-derived); ratio ranges may not transfer to other genomes

## Evidence

- [methods] Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads).: "Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads)."
- [methods] meryl count k=15 output merylDB asm.fa.gz: "meryl count k=15 output merylDB asm.fa.gz"
- [methods] 1. **Merqury:** To assess the quality value (QV).: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
- [methods] NextPolish2 improved the QV of the assembled genome from 53.8544 to 55.6257: "`NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257"
- [discussion] The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors.: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [abstract] NextPolish2 can fix base errors in HiFi long-read assemblies without introducing excessive overcorrections and haplotype switch errors: "which can fix base errors remaining in those “highly accurate” genomes assembled from HiFi long reads without introducing excessive overcorrections and haplotype switch errors"
