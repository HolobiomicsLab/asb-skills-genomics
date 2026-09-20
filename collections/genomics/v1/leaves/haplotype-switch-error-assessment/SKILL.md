---
name: haplotype-switch-error-assessment
description: Use when comparing polishing pipelines (e.g. NextPolish2 vs Racon+Merfin
  'RM') on HiFi assemblies where switch-error rates and QV in heterozygous regions
  must be benchmarked against reported values.
when_to_use_negative:
- The genome is haploid or homozygous (e.g. CHM13), so there are no heterozygous regions
  in which switch errors can be measured
- The assembly was not produced from HiFi long reads or lacks phase information/trio
  binning, making switch-error computation meaningless
- No independent high-coverage short reads are available to build yak k-mer datasets
  for k-mer consensus evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3678
tools:
- name: yak
  role: Builds k=21/k=31 k-mer datasets (-b 37) from short reads and computes k-mer
    consensus QV for assemblies before/after polishing
  repo: https://github.com/lh3/yak
- name: Merqury
  role: K-mer consensus evaluation yielding QV and switch error rates in heterozygous
    regions
- name: hifiasm
  role: Assembles HiFi reads into primary contigs plus trio-binned paternal/maternal
    contigs
- name: meryl
  role: Counts k=15 k-mers of the assembly and extracts repetitive k-mers (distinct>0.9998)
    for the winnowmap filter list
  repo: https://github.com/marbl/meryl
- name: winnowmap
  role: Maps HiFi reads to the assembly with the repetitive k-mer filter (-W repetitive_k15.txt,
    map-pb preset)
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi read mapper using the map-hifi preset
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts and indexes HiFi alignments into hifi.map.sort.bam
- name: fastp
  role: Quality-filters and trims short reads (-q 20, adapter removal, 5'/3' trimming)
    before yak k-mer counting
- name: NextPolish2
  role: Polishing method under evaluation; its built-in phasing module should maintain
    haplotype consistency while correcting bases
  repo: https://github.com/Nextomics/NextPolish2
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:2f125d8043e64719978fb15d0b70e7afaafee659075fc8a0855eb889468fcf0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/yak
  tools:
  - Merqury
  - NextPolish2
  - Racon
  - fastp
  - hifiasm
  - meryl
  - minimap2
  - samtools
  - winnowmap
  - yak
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# haplotype-switch-error-assessment

## Summary

Quantifies haplotype switch errors and consensus quality values (QV) of HiFi-assembled genomes before and after polishing, using k-mer-based evaluation (yak/Merqury) on heterozygous diploid and trio-binned assemblies. It exists to verify that polishing tools correct SNV/indel errors without breaking local haplotype consistency.

## When to use

Use when comparing polishing pipelines (e.g. NextPolish2 vs Racon+Merfin 'RM') on HiFi assemblies where switch-error rates and QV in heterozygous regions must be benchmarked against reported values.

## When NOT to use

- The genome is haploid or homozygous (e.g. CHM13), so there are no heterozygous regions in which switch errors can be measured
- The assembly was not produced from HiFi long reads or lacks phase information/trio binning, making switch-error computation meaningless
- No independent high-coverage short reads are available to build yak k-mer datasets for k-mer consensus evaluation

## Inputs

- HiFi long reads (FASTA/FASTQ, e.g. hifi.fasta.gz)
- Short Illumina reads (>=60X) for yak k-mer datasets
- Assembly FASTA/GZ (unpolished hifiasm primary contigs, trio-binned paternal/maternal contigs, and polished asm.np2.fa)
- Repetitive k-mer filter list (repetitive_k15.txt from meryl)
- Sorted HiFi alignment BAM (hifi.map.sort.bam)
- yak k-mer datasets (k21.yak, k31.yak)

## Outputs

- QV per assembly (unpolished vs polished) from yak/Merqury, e.g. asm-sr.qv.txt
- Switch error rates (‰) in heterozygous regions per assembly/contig set
- Benchmark table comparing NextPolish2 vs Racon-based (RM) comparator

## How to apply

Assemble HiFi reads with hifiasm to obtain primary and, for trios, paternal/maternal contigs (discarding reads of differing haplotype before mapping). Count repetitive 15-mers with meryl (k=15, distinct>0.9998) and map HiFi reads with winnowmap (-W repetitive_k15.txt, map-pb) or minimap2 (map-hifi), sorting with samtools into hifi.map.sort.bam. Build yak k-mer datasets at k=21 and k=31 with -b 37 from >=60X short reads quality-filtered by fastp (-q 20, -f 5 -F 5 -t 5 -T 5). After polishing (e.g. nextPolish2 -t 5 BAM asm.fa.gz k21.yak k31.yak), evaluate QV and switch error rates with yak/Merqury k-mer consensus for both unpolished and polished assemblies. Judge success by reproducing reported values such as simulated A. thaliana QV 47.67→65.42 and switch error 1.99→0.35 ‱ (vs 737.45 ‱ for RM), and HG002 hifiasm primary QV 60.25 with switch error 0.15 vs 0.14 after NextPolish2.

## Related tools

- **yak** (Builds k=21/k=31 k-mer datasets (-b 37) from short reads and computes k-mer consensus QV for assemblies before/after polishing) — https://github.com/lh3/yak
- **Merqury** (K-mer consensus evaluation yielding QV and switch error rates in heterozygous regions)
- **hifiasm** (Assembles HiFi reads into primary contigs plus trio-binned paternal/maternal contigs)
- **meryl** (Counts k=15 k-mers of the assembly and extracts repetitive k-mers (distinct>0.9998) for the winnowmap filter list) — https://github.com/marbl/meryl
- **winnowmap** (Maps HiFi reads to the assembly with the repetitive k-mer filter (-W repetitive_k15.txt, map-pb preset)) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi read mapper using the map-hifi preset) — https://github.com/lh3/minimap2/
- **samtools** (Sorts and indexes HiFi alignments into hifi.map.sort.bam)
- **fastp** (Quality-filters and trims short reads (-q 20, adapter removal, 5'/3' trimming) before yak k-mer counting)
- **NextPolish2** (Polishing method under evaluation; its built-in phasing module should maintain haplotype consistency while correcting bases) — https://github.com/Nextomics/NextPolish2

## Examples

```
./yak qv -t32 -p -K3.2g -l100k sr.yak asm.np2.fa > asm.np2-sr.qv.txt
```

## Evaluation signals

- Simulated heterozygous A. thaliana: QV improves from 47.67 (hifiasm primary) to 65.42 with NextPolish2 and switch error rate drops from 1.99 to 0.35 ‱, while RM raises switch errors to 737.45 ‱
- Col-XJTU: QV improves from 58.03 (hifiasm primary) to 64.26 with NextPolish2
- HG002: switch error rates after NextPolish2 are 0.14‰ (child), 0.20‰ (paternal), 0.30‰ (maternal) — each ≤ the original assembly — vs RM at 5.40‰, 0.69‰, 1.60‰
- HG002 benchmark values: hifiasm primary QV 60.25 vs NextPolish2 62.87; trio paternal 59.77 vs 63.49; trio maternal 59.78 vs 63.29
- Polished assembly must show no increase in switch error rate relative to the unpolished assembly (NextPolish2 should slightly reduce it)

## Limitations

- Performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors; uncorrected errors may be caused by biases inherent in short-read sequencing technologies, and no specific coverage threshold is given
- Trio-binned evaluations require discarding reads with different haplotype before mapping, otherwise haplotype assessment is confounded
- No software version numbers stated for NextPolish2 or dependencies, complicating exact reproduction

## Evidence

- [abstract] abstract claim that phasing preserves haplotypes while removing base errors: "which can fix base errors remaining in those “highly accurate” genomes assembled from HiFi long reads without introducing excessive overcorrections and haplotype switch errors"
- [intro] built-in phasing module maintains haplotype consistency and slightly reduces switch errors: "Through the built-in phasing module, it can not only correct the error bases, but also maintain the original haplotype consistency."
- [methods] three-pronged evaluation before and after polishing including Merqury QV: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
- [methods] haplotype read filtering prerequisite for trio switch-error assessment: "You can discard reads that have different haplotype with the reference before the mapping procedure"
- [methods] reported HG002 benchmark QV and switch error values to reproduce: "| Hifiasm  (primary) | 60.25   | 0.15                    |"
- [readme] yak QV computation command used for k-mer consensus evaluation: [evidence span withheld — anchor status: external_doc]
