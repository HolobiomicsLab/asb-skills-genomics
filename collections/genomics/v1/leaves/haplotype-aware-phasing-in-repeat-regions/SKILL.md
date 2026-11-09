---
name: haplotype-aware-phasing-in-repeat-regions
description: Use when polishing a HiFi-assembled T2T genome that contains repetitive
  elements (e.
when_to_use_negative:
- The assembly was built from noisy long reads without high-coverage short-read support,
  since NextPolish2's error checking relies heavily on short reads to judge whether
  a K-mer contains errors
- The goal is de novo haplotype phasing of a diploid genome rather than polishing
  an existing assembly while preserving the reference haplotype
- Short-read coverage is far below the recommended >=60X, risking uncorrected errors
  caused by short-read sequencing biases
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0102
tools:
- name: NextPolish2
  role: Runs the repeat-aware correction engine; the -r flag enables the built-in
    phasing module that groups reads by haplotype/repeat copy via Louvain community
    detection
  repo: https://github.com/Nextomics/NextPolish2
- name: Winnowmap
  role: Maps HiFi reads to the draft assembly using weighted minimizers and a repetitive
    k-mer list (-W) to preserve mapping accuracy in complex repeats feeding the LQR/phasing
    analysis
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi-read mapper (map-pb/map-hifi presets) for preparing the BAM
    input to NextPolish2
  repo: https://github.com/lh3/minimap2/
- name: yak
  role: Builds k21/k31 short-read k-mer datasets (-b 37) used to check whether K-mers
    contain errors and to arbitrate conflicting read communities
  repo: https://github.com/lh3/yak
- name: meryl
  role: Counts assembly k-mers (k=15) and prints k-mers with distinct>0.9998 as the
    repetitive k-mer list passed to winnowmap -W
  repo: https://github.com/marbl/meryl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:c125037cc03a1057858e37aabaf41275f528dd441a7f6eb25d32b5a2ede2965a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - NextPolish2
  - Winnowmap
  - meryl
  - minimap2
  - yak
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# haplotype-aware-phasing-in-repeat-regions

## Summary

A repeat-aware polishing/phasing approach from NextPolish2 that detects low-confidence (LQR) regions, groups reads spanning them by haplotype or repeat copy via Louvain community detection, and resolves conflicting communities by retaining the one sharing the most K-mers with the reference so the original haplotype is preserved.

## When to use

Use when polishing a HiFi-assembled T2T genome that contains repetitive elements (e.g., segmental duplications, centromeric satellite arrays, homopolymers) where reads spanning the same low-quality region originate from different haplotypes or repeat copies and must be grouped without introducing switch errors.

## When NOT to use

- The assembly was built from noisy long reads without high-coverage short-read support, since NextPolish2's error checking relies heavily on short reads to judge whether a K-mer contains errors
- The goal is de novo haplotype phasing of a diploid genome rather than polishing an existing assembly while preserving the reference haplotype
- Short-read coverage is far below the recommended >=60X, risking uncorrected errors caused by short-read sequencing biases

## Inputs

- Sorted, indexed HiFi-read alignment BAM against the draft assembly (hifi.map.sort.bam, produced by winnowmap/minimap2 + samtools)
- Draft assembly FASTA (gzipped, e.g. asm.fa.gz) from HiFi-based assembly (e.g. HiFiasm primary assembly)
- yak k-mer datasets from >=60X short reads (k21.yak and k31.yak, built with yak count -k 21/-k 31 -b 37)
- Repetitive k-mer list for winnowmap (repetitive_k15.txt from meryl count k=15 with distinct>0.9998)
- Quality-filtered short reads (fastp: -5 -3 -f 5 -F 5 -t 5 -T 5 -q 20)

## Outputs

- Polished, haplotype-consistent assembly FASTA (asm.np2.fa) with corrected SNV/indels and original haplotype maintained in repeats
- Repeat-copy/haplotype read groupings per LQR produced by the Louvain community-detection phasing module

## How to apply

First map HiFi reads to the draft assembly with winnowmap (passing a precomputed repetitive k-mer list, e.g. meryl k=15 with distinct>0.9998, via -W) or minimap2, and sort/index with samtools to get hifi.map.sort.bam. Run NextPolish2 with the -r flag (./target/release/nextPolish2 -t 5 -r hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa) to enable the repeat-aware phasing module, supplying yak k-mer datasets (k=21 and k=31 from >=60X quality-filtered short reads). The module detects low-quality regions (LQRs), applies the Louvain community detection algorithm to cluster reads spanning each LQR by haplotype or repeat copy, and for conflicting communities keeps by default the community sharing the most K-mers with the reference while discarding reads from other communities, which maintains the reference haplotype and avoids overcorrection. Optionally, for trio-binned assemblies, discard reads belonging to the other haplotype before mapping. Validate that polishing improves consensus quality (HG005 benchmark: QV 53.8544 -> 55.6257, homozygous high-quality variants 14,155 -> 6,955) without inflating switch errors; the built-in phasing module reportedly even slightly reduces switch errors in heterozygous regions.

## Related tools

- **NextPolish2** (Runs the repeat-aware correction engine; the -r flag enables the built-in phasing module that groups reads by haplotype/repeat copy via Louvain community detection) — https://github.com/Nextomics/NextPolish2
- **Winnowmap** (Maps HiFi reads to the draft assembly using weighted minimizers and a repetitive k-mer list (-W) to preserve mapping accuracy in complex repeats feeding the LQR/phasing analysis) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi-read mapper (map-pb/map-hifi presets) for preparing the BAM input to NextPolish2) — https://github.com/lh3/minimap2/
- **yak** (Builds k21/k31 short-read k-mer datasets (-b 37) used to check whether K-mers contain errors and to arbitrate conflicting read communities) — https://github.com/lh3/yak
- **meryl** (Counts assembly k-mers (k=15) and prints k-mers with distinct>0.9998 as the repetitive k-mer list passed to winnowmap -W) — https://github.com/marbl/meryl

## Examples

```
winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz | samtools sort -o hifi.map.sort.bam -; ./target/release/nextPolish2 -t 5 -r hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- QV improvement over the draft assembly measured with Merqury (HG005 benchmark rose from 53.8544 to 55.6257)
- Reduction in homozygous high-quality variants (potential errors) versus a truth-adjacent reference (HG005: 14,155 -> 6,955 via GRCh37/GIAB comparison with DeepVariant)
- No increase — ideally a slight decrease — in switch errors in heterozygous regions after phasing-aware polishing
- Absence of overcorrection: no excess changes in highly repetitive regions compared to the draft
- Consensus accuracy evaluated before/after polishing with the three approaches Merqury, DeepVariant, and Paftools.js

## Limitations

- Performance relies heavily on short reads to check whether a K-mer contains errors; without high-coverage short reads, uncorrected errors caused by short-read sequencing biases remain
- For conflicting read communities, the default keeps the community sharing the most K-mers with the reference, so the reference haplotype is maintained rather than an alternative haplotype recovered
- No software version numbers are stated for NextPolish2 or its dependencies, complicating reproducibility
- Genomes still contain residual errors where HiFi reads also stumble (homopolymers, low-complexity microsatellites), and gap-filling via error-prone ONT ultra-long reads requires further polishing

## Evidence

- [results] NextPolish2 applies the Louvain community detection algorithm to group reads spanning the same LQR by haplotype or repeat copy: [evidence span withheld — anchor status: not_found]
- [results] Default conflict resolution keeps the K-mer-sharing community: [evidence span withheld — anchor status: not_found]
- [intro] Built-in phasing module maintains haplotype consistency and reduces switch errors: "Through the built-in phasing module, it can not only correct the error bases, but also maintain the original haplotype consistency. In fact, our evaluation shows that it even slightly reduces switch"
- [methods] Run NextPolish2 command with inputs: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [methods] HG005 QV and homozygous variant improvement benchmark: "NextPolish2 improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955."
- [discussion] Reliance on short reads for K-mer error checking: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [abstract] Abstract claim of no overcorrection or switch errors: "which can fix base errors remaining in those “highly accurate” genomes assembled from HiFi long reads without introducing excessive overcorrections and haplotype switch errors"
- [methods] Discard reads of different haplotype before mapping: "You can discard reads that have different haplotype with the reference before the mapping procedure"
