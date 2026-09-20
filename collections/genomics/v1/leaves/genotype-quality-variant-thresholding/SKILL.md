---
name: genotype-quality-variant-thresholding
description: Use when filtering or benchmarking SNV calls from long-read (ONT/PacBio)
  data, where a fixed GQ cutoff is inappropriate and the threshold must be matched
  to median read coverage before computing precision/recall/F1.
when_to_use_negative:
- 'Calling short indels: Longshot only calls SNVs and can merely genotype indels supplied
  in an input VCF; accurate de novo indel calling with SMS reads is hampered by the
  high insertion/deletion error rate'
- Structural variant discovery — use a dedicated SV caller such as Delly instead
- Short-read (Illumina) data, where a fixed GQ cutoff of 50 applied to short-read
  callers is the established practice
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Long-read SNV caller whose VCF GQ field is thresholded; also emits genotype
    qualities used for the variable cutoff sweep
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:bb44999145d936fbced03d569ae88ebd531f4af6af46096ec25de4719163b6af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genotype-quality-variant-thresholding

## Summary

Calibration and application of genotype-quality (GQ) thresholds to filter long-read SNV calls: a variable GQ cutoff matched to the median read coverage is used for Longshot calls, versus a fixed GQ 50 for short-read callers, when benchmarking against GIAB high-confidence truth sets.

## When to use

Use when filtering or benchmarking SNV calls from long-read (ONT/PacBio) data, where a fixed GQ cutoff is inappropriate and the threshold must be matched to median read coverage before computing precision/recall/F1.

## When NOT to use

- Calling short indels: Longshot only calls SNVs and can merely genotype indels supplied in an input VCF; accurate de novo indel calling with SMS reads is hampered by the high insertion/deletion error rate
- Structural variant discovery — use a dedicated SV caller such as Delly instead
- Short-read (Illumina) data, where a fixed GQ cutoff of 50 applied to short-read callers is the established practice

## Inputs

- Aligned long-read BAM/CRAM (e.g., minimap2-aligned ONT reads for NA12878) plus indexed reference FASTA (GRCh38)
- Unfiltered Longshot VCF with genotype quality scores (GQ field)
- Truth set VCF, e.g., GIAB high-confidence call set for GRCh38

## Outputs

- Filtered phased VCF of high-GQ SNV calls
- Contingency counts (TP/FP/FN/TN) versus the GIAB truth set
- Accuracy table (e.g., na12878_ont_accuracy.tsv) with precision, recall, and F1 per GQ cutoff

## How to apply

Run the variant caller (e.g., Longshot with defaults: min_mapq 20, min_allele_qual 7.0, strand-bias Fisher exact p < 0.01) and write an unfiltered phased VCF. Compute the median read depth d across the whole dataset, then sweep a variable GQ cutoff chosen to match that median coverage (GQ 65 reproduced the paper's ONT NA12878 whole-genome result at ~37x, whereas short-read variant calls use a fixed GQ cutoff of 50). At each cutoff, compare calls against the GIAB high-confidence set, counting true/false positives and negatives to compute precision, recall, and F1. Also apply the depth-based ceiling filter — variants with read depth above d + 5*sqrt(d) are removed as copy-number-amplification artifacts — before scoring, since depth outliers otherwise inflate false positives.

## Related tools

- **Longshot** (Long-read SNV caller whose VCF GQ field is thresholded; also emits genotype qualities used for the variable cutoff sweep) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam ont.bam --ref grch38.fa --out na12878_ont_longshot.vcf
```

## Evaluation signals

- At GQ 65 on NA12878 ONT whole-genome data, precision reaches 0.991 and recall 0.933, giving a best F1 of 0.967 against GIAB high-confidence calls on GRCh38
- The chosen GQ cutoff equals the value matched to the dataset's median read coverage (e.g., ~65 at 37x ONT coverage)
- No retained variant exceeds the depth ceiling d + 5*sqrt(d), where d is the median read depth
- The strand-bias filter (Fisher's exact test p < 0.01) remains active so stranded false SNV sites are excluded before genotyping
- Precision increases monotonically with the GQ cutoff while recall decreases; the operating point is selected at maximal F1

## Limitations

- GIAB and Platinum Genomes truth sets were generated with short-read data and are therefore biased in favor of short-read technologies, so some Longshot false positives may actually be false negatives in the GIAB call sets
- Allele-quality filtering (default phred threshold 7.0) discards low-quality allele observations, reducing the effective read depth for SMS reads
- Correctly mapping long reads in highly similar segmental duplications remains a challenge and can inflate apparent error

## Evidence

- [discussion] Variable GQ cutoff matched to median read coverage for Longshot: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] Fixed GQ 50 for short-read callers: "The GQ cutoff was fixed at 50 for short reads."
- [discussion] Depth-based filter for copy-number amplifications: [evidence span withheld — anchor status: not_found]
- [discussion] Strand-bias pre-filter before haplotype-informed genotyping: [evidence span withheld — anchor status: not_found]
- [discussion] Truth-set bias caveat: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets"
- [readme] Example run from README: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
