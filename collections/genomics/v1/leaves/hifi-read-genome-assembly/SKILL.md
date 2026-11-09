---
name: hifi-read-genome-assembly
description: Use when the input is PacBio HiFi long reads (optionally with ONT gap-filling
  and >=60X short reads) and the goal is a primary assembly whose consensus base errors
  (SNV/indel) must be corrected without introducing haplotype switch errors.
when_to_use_negative:
- The draft assembly was built from noisy long reads without any HiFi data or >=60X
  short reads — NextPolish2's error checking depends on short-read k-mer spectra
- The goal is de novo contig construction from ONT ultra-long reads alone; this skill
  covers HiFi-based assembly plus consensus polishing, not ONT-only assembly
- Only interested in variant calling against a reference rather than improving assembly
  consensus accuracy — DeepVariant-based evaluation may be more appropriate than polishing
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: hifiasm
  role: Assembles HiFi reads into primary contigs that serve as the draft input for
    polishing
- name: NextPolish2
  role: Repeat-aware correction engine producing asm.np2.fa; -r enables the Louvain-based
    haplotype/repeat-copy phasing module
  repo: https://github.com/Nextomics/NextPolish2
- name: winnowmap
  role: Maps HiFi reads to the assembly using the repetitive_k15.txt weighted-minimizer
    list (-W, map-pb preset)
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi mapper using the map-hifi preset when repetitive k-mer lists
    are not needed
  repo: https://github.com/lh3/minimap2/
- name: meryl
  role: Counts k=15 k-mers of the assembly and prints those with distinct>0.9998 as
    the repetitive k-mer list for winnowmap
- name: yak
  role: Builds k=21 and k=31 short-read k-mer count datasets (-b 37) used by NextPolish2
    and yak qv for QV evaluation
  repo: https://github.com/lh3/yak
- name: fastp
  role: Quality filters short reads (-5 -3 -f 5 -F 5 -t 5 -T 5 -q 20) with adapter
    removal before yak counting
- name: samtools
  role: Sorts and indexes the HiFi alignment BAM (hifi.map.sort.bam)
- name: Merqury
  role: Evaluates k-mer consensus QV and switch error rate of unpolished vs polished
    assemblies
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:8332d3988dd8a409b69ef8239a8a5645fd91bf701576e600b5ea4e4999796457
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - Merqury
  - NextPolish2
  - PacBio HiFi
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# hifi-read-genome-assembly

## Summary

Assemble high-quality draft genomes from PacBio HiFi long reads (e.g., with hifiasm) and bring them to T2T-grade consensus accuracy by repeat-aware polishing with NextPolish2, using HiFi alignments (winnowmap/minimap2) plus short-read yak k-mer datasets as evidence.

## When to use

Use when the input is PacBio HiFi long reads (optionally with ONT gap-filling and >=60X short reads) and the goal is a primary assembly whose consensus base errors (SNV/indel) must be corrected without introducing haplotype switch errors.

## When NOT to use

- The draft assembly was built from noisy long reads without any HiFi data or >=60X short reads — NextPolish2's error checking depends on short-read k-mer spectra
- The goal is de novo contig construction from ONT ultra-long reads alone; this skill covers HiFi-based assembly plus consensus polishing, not ONT-only assembly
- Only interested in variant calling against a reference rather than improving assembly consensus accuracy — DeepVariant-based evaluation may be more appropriate than polishing

## Inputs

- PacBio HiFi long reads (FASTQ/FASTA, e.g., ~30x HG005 or Col-XJTU HiFi reads)
- Draft HiFi-assembled contigs (hifiasm primary assembly, FASTA/FASTA.gz)
- High-coverage short reads (>=60X recommended) for yak k-mer datasets
- ONP/ONT ultra-long reads for gap filling (optional)

## Outputs

- Polished assembly FASTA (asm.np2.fa)
- Sorted HiFi alignment BAM (hifi.map.sort.bam) with samtools index
- yak k-mer count datasets at k=21 and k=31 (k21.yak, k31.yak)
- Merqury/yak QV and switch-error benchmark table for pre- vs post-polish assemblies

## How to apply

Assemble HiFi reads into primary contigs with hifiasm, then prepare the polishing inputs exactly as the NextPolish2 workflow prescribes: count repetitive k-mers with meryl (k=15, print k-mers with distinct>0.9998) and map HiFi reads with winnowmap (-W repetitive_k15.txt, map-pb preset) or minimap2 (map-hifi preset), sorting and indexing the BAM with samtools. Quality-filter short reads with fastp (-q 20 with adapter removal and 5'/3' trimming) and build two yak k-mer datasets at k=21 and k=31 using -b 37 to discard singletons, since NextPolish2 relies heavily on short reads to decide whether a K-mer contains errors. Run the correction engine (nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa); optionally rerun with -r for the repeat-aware phasing module that groups reads by haplotype or repeat copy via Louvain community detection, keeping the community sharing the most K-mers with the reference. Evaluate unpolished and polished assemblies with Merqury/yak k-mer consensus QV and switch error rate, comparing against a Racon-based comparator.

## Related tools

- **hifiasm** (Assembles HiFi reads into primary contigs that serve as the draft input for polishing)
- **NextPolish2** (Repeat-aware correction engine producing asm.np2.fa; -r enables the Louvain-based haplotype/repeat-copy phasing module) — https://github.com/Nextomics/NextPolish2
- **winnowmap** (Maps HiFi reads to the assembly using the repetitive_k15.txt weighted-minimizer list (-W, map-pb preset)) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi mapper using the map-hifi preset when repetitive k-mer lists are not needed) — https://github.com/lh3/minimap2/
- **meryl** (Counts k=15 k-mers of the assembly and prints those with distinct>0.9998 as the repetitive k-mer list for winnowmap)
- **yak** (Builds k=21 and k=31 short-read k-mer count datasets (-b 37) used by NextPolish2 and yak qv for QV evaluation) — https://github.com/lh3/yak
- **fastp** (Quality filters short reads (-5 -3 -f 5 -F 5 -t 5 -T 5 -q 20) with adapter removal before yak counting)
- **samtools** (Sorts and indexes the HiFi alignment BAM (hifi.map.sort.bam))
- **Merqury** (Evaluates k-mer consensus QV and switch error rate of unpolished vs polished assemblies)

## Examples

```
./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- Merqury/yak QV of the polished assembly exceeds the hifiasm primary draft (e.g., HG005: 53.8544 -> 55.6257; Col-XJTU: 58.03 -> 64.26)
- Homozygous high-quality (potential-error) variant count against GRCh37 outside GIAB high-confidence regions decreases (HG005: 14,155 -> 6,955)
- On the simulated highly heterozygous diploid A. thaliana benchmark, switch error rate drops from 1.99 to 0.35 per 10,000 rather than rising (RM pipeline: 737.45)
- Sorted BAM is indexed and produced with winnowmap -W repetitive_k15.txt (or minimap2 map-hifi), and yak datasets exist for both k=21 and k=31 built with -b 37
- Polished FASTA is non-empty, contains the same contig set as the draft, and outperforms a Racon-based comparator pipeline on QV

## Limitations

- NextPolish2's performance relies heavily on short reads; residual errors concentrate where short-read coverage is too low to validate k-mers (Col-XJTU: 34.52% of Merqury-defined erroneous k-mers in regions of depth <= 3; HG002: 76.29%)
- For conflict communities during phasing, the default keeps the community sharing the most K-mers with the reference, so a wrong reference haplotype is preserved rather than switched
- No software version numbers are stated for NextPolish2 or its dependencies in the article or README

## Evidence

- [readme] NextPolish2 can be used to fix these errors (SNV/Indel) in a high quality assembly.: "NextPolish2 can be used to fix these errors (SNV/Indel) in a high quality assembly."
- [methods] winnowmap mapping command with repetitive k-mer list: "winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz|samtools sort -o hifi.map.sort.bam -"
- [methods] NextPolish2 correction engine invocation: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [methods] fastp short-read quality filtering parameters: "# Quality control and filtering.
# fastp -5 -3 -n 0 -f 5 -F 5 -t 5 -T 5 -q 20"
- [methods] HG005 QV improvement after polishing: "NextPolish2 improved the QV of the assembled genome from 53.8544 to 55.6257"
- [discussion] dependence on short reads: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
