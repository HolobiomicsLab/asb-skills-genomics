---
name: splicing-bias-visualization
description: Use when comparing AISO/splicing-order measurements across sequencing
  technologies (e.g., ONT long reads vs Illumina paired-end reads from the same sample)
  to detect and visualize technology-driven splicing-order biases.
when_to_use_negative:
- When only one sequencing technology is available — a single Fupfi histogram cannot
  distinguish biological AISO patterns from technology-specific bias
- When the goal is differential splicing (PSI/DSU) between conditions rather than
  splicing-order (AISO) quantification around exons of interest
- When exons exceed the maximum length quantifiable by short reads (read/fragment-size
  limited), as short-read Fupfi counts will be biased toward short exons
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: Insplico
  role: Standalone software that quantifies AISO (upfi, dofi, bus, bos counts and
    Fupfi) from both short and long read RNA-seq BAMs against a user-specified exon
    table
  repo: gitlab.com/aghr/insplico
- name: Minimap2
  role: Splice-aware aligner for ONT long reads to hg38
- name: STAR
  role: 2-pass splice-aware aligner for Illumina paired-end reads with chimSegmentMin
    20 and outFilterMultimapNmax 100
- name: ggplot2
  role: R plotting package used to render the Fupfi histograms per technology
- name: polyester
  role: R package used in the simulation workflow to generate simulated reads for
    validating Insplico's bias behavior
  repo: https://github.com/liniguez/Insplico_simulations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:b67c11d9bca3d4fa31714465ddc1135b0c9e914a424094dbc4f0c76da3f69f75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - Minimap2
  - R
  - STAR
  - ggplot2
  - polyester
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# splicing-bias-visualization

## Summary

Quantifies and visualizes adjacent intron splicing order (AISO) biases between sequencing technologies by computing per-exon upfi/dofi count ratios and plotting genome-wide Fupfi histograms from Insplico output, revealing that long-read (ONT) and short-read (Illumina) protocols yield discrepant AISO patterns (upfi/dofi ≈ 0.41 vs ≈ 1.41).

## When to use

Use when comparing AISO/splicing-order measurements across sequencing technologies (e.g., ONT long reads vs Illumina paired-end reads from the same sample) to detect and visualize technology-driven splicing-order biases.

## When NOT to use

- When only one sequencing technology is available — a single Fupfi histogram cannot distinguish biological AISO patterns from technology-specific bias
- When the goal is differential splicing (PSI/DSU) between conditions rather than splicing-order (AISO) quantification around exons of interest
- When exons exceed the maximum length quantifiable by short reads (read/fragment-size limited), as short-read Fupfi counts will be biased toward short exons

## Inputs

- ONT long-read RNA-seq FASTQ/BAM (e.g., GSE207459 K562 chromatin-fraction reads)
- Illumina paired-end RNA-seq FASTQ/BAM matched to the same sample (GSE207459)
- Genome reference FASTA (hg38, chr1-chrY)
- Genome annotation GTF (e.g., Ensembl v88 with VastDB merge; Hsa38 GTF filtered to transcript_support_level 1)
- Exon-of-interest table (~233,306 exons) for Insplico

## Outputs

- Per-exon upfi, dofi, bus, bos count tables from Insplico for each technology
- Fupfi fraction per exon and genome-wide Fupfi histograms (ggplot2) per technology
- upfi/dofi ratio summary (expected ≈ 0.41 ONT, ≈ 1.41 Illumina) and binomial P-value (e.g., P = 7e-26)

## How to apply

Download matched ONT long-read and Illumina paired-end RNA-seq data (e.g., GSE207459 K562 chromatin/4sU fractions), align ONT reads to hg38 with Minimap2 in splice-aware mode and Illumina reads with STAR in 2-pass mode (chimSegmentMin 20, alignSJoverhangMin 8, alignIntronMin 20, alignIntronMax 200000, outFilterMultimapNmax 100). Build a genome-wide exon-of-interest table by merging Ensembl v88 annotation with VastDB (~233,306 exons), then run Insplico on both BAMs to obtain per-exon upfi, dofi, bus, and bos counts and compute the Fupfi fraction. Plot the Fupfi histograms for both technologies with ggplot2 and compute upfi/dofi ratios; the direction of the ratio imbalance (excess dofi ≈ 0.41 for ONT, excess upfi ≈ 1.41 for Illumina) quantifies the technology-specific bias. Apply Insplico's optional 5'/3' bias-correction module for long reads, since uncorrected long-read library biases can drive the discrepancy.

## Related tools

- **Insplico** (Standalone software that quantifies AISO (upfi, dofi, bus, bos counts and Fupfi) from both short and long read RNA-seq BAMs against a user-specified exon table) — gitlab.com/aghr/insplico
- **Minimap2** (Splice-aware aligner for ONT long reads to hg38)
- **STAR** (2-pass splice-aware aligner for Illumina paired-end reads with chimSegmentMin 20 and outFilterMultimapNmax 100)
- **ggplot2** (R plotting package used to render the Fupfi histograms per technology)
- **polyester** (R package used in the simulation workflow to generate simulated reads for validating Insplico's bias behavior) — https://github.com/liniguez/Insplico_simulations

## Examples

```
insplico -b ont_k562.bam -g Hsa38_only_transcript_support_level_1.gtf -e exon_of_interest.tsv --mode longread ; insplico -b illumina_k562.bam -g Hsa38_only_transcript_support_level_1.gtf -e exon_of_interest.tsv --mode shortread
```

## Evaluation signals

- ONT Fupfi histogram shows excess dofi (upfi/dofi ≈ 0.41) while matched Illumina shows excess upfi (≈ 1.41), reproducing the reported long-read bias
- Binomial test on upfi/dofi counts yields a significant P-value (e.g., P = 7e-26 for the Illumina upfi excess)
- After applying Insplico's 5'/3' bias-correction module to long reads, the ONT and short-read Fupfi patterns harmonize
- Per-exon count table contains valid non-negative upfi, dofi, bus, bos entries for all ~233,306 exons of interest and Fupfi values lie in [0,1]

## Limitations

- Short reads provide almost no splicing-order information beyond the introns directly flanking the exon of interest, unlike long reads
- Read length and paired-end fragment size limit the maximum exon length quantifiable by Insplico with short reads
- Long-read sequencing and library preparation carry 5' and 3' biases that must be corrected to harmonize with short-read results
- Sequencing technology itself impacts AISO results, so cross-technology comparisons of raw AISO are confounded

## Evidence

- [abstract] the first standalone software for quantifying AISO that works with both short and long read sequencing technologies: "we present Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [discussion] It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts): "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [discussion] The optional module to perform these corrections is a unique feature of Insplico: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [discussion] the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
- [supplementary] RNA seq data have been deposited to Gene Expression Omnibus (GEO) and are accessible under the ID GSE207459.: "RNA seq data have been deposited to Gene Expression Omnibus (GEO) and are accessible under the ID GSE207459."
- [readme] Simulation workflow R session setup loading dplyr, Biostrings, polyester, ggplot2 for generating UPFI/DOFI transcripts and read simulation.: "p_load(dplyr)
p_load(Biostrings)
p_load(polyester)
p_load(ggplot2)"
