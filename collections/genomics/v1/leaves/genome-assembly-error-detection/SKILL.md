---
name: genome-assembly-error-detection
description: Use when you have raw NGS and SMS reads plus a draft assembly (FASTA)
  and need to pinpoint misassemblies, regional indel errors, or misjoined contigs
  without a close reference genome.
when_to_use_negative:
- Evaluating completeness (e.g., gene completeness) rather than error detection —
  use BUSCO instead
- Assemblies of species with higher ploidy levels than diploid, where CRAQ accuracy
  'may not be as accurate as the benchmarked cases presented here'
- When only one read type (short or long) is available, since CRAQ integrates both
  NGS and SMS mapping signals
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: 'Core reference-free error detector: extracts clipped alignments from filtered
    BAMs and calls CREs/CSEs and heterozygous CRHs/CSHs at single-nucleotide resolution'
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps short reads ('-ax sr') and long reads ('-ax map-hifi/map-pb/map-ont')
    back to the draft assembly; version 2.17+
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' before CRAQ input; version
    1.3.1+
- name: IGV
  role: Visually inspect filtered alignments and flagged error loci
- name: pycircos
  role: Circos visualization of assembly features and comparative results
  repo: https://github.com/ponnhide/pyCircos
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:c07a45861493584e93dd431f8f780bbf6a42f748abe5f9010300e7db65675777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - IGV
  - Inspector
  - Merqury
  - Minimap2
  - PBSIM
  - QUAST
  - SAMtools
  - SyRI
  - Wgsim
  - pycircos
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# genome-assembly-error-detection

## Summary

Detect regional (CREs) and structural (CSEs) errors in draft genome assemblies at single-nucleotide resolution using CRAQ, which maps NGS short reads and SMS long reads back to the assembly and exploits clipped-alignment signals, read coverage, and filtering of heterozygous loci.

## When to use

Use when you have raw NGS and SMS reads plus a draft assembly (FASTA) and need to pinpoint misassemblies, regional indel errors, or misjoined contigs without a close reference genome.

## When NOT to use

- Evaluating completeness (e.g., gene completeness) rather than error detection — use BUSCO instead
- Assemblies of species with higher ploidy levels than diploid, where CRAQ accuracy 'may not be as accurate as the benchmarked cases presented here'
- When only one read type (short or long) is available, since CRAQ integrates both NGS and SMS mapping signals

## Inputs

- Draft genome assembly in FASTA format (contigs/scaffolds)
- Illumina NGS short-read sequencing data (FASTQ)
- SMS long-read data (PacBio HiFi/CLR or ONT, FASTQ)
- Optional: reference genome (e.g., hg38 GCF_000001405.26) or paired assembly (e.g., S. pennellii CaSM) for benchmarking or SyRI comparison

## Outputs

- CRE and CSE breakpoint coordinate lists (error calls)
- CRH and CSH heterozygous-variant coordinates
- R-AQI and S-AQI assembly quality index scores
- CRAQ-corrected contigs after breaking misjoins at CSE positions (optional)

## How to apply

Map the Illumina short reads to the draft assembly with Minimap2 using '-ax sr' and the long reads with '-ax map-hifi', '-ax map-pb', or '-ax map-ont' depending on chemistry, then filter alignments with 'samtools view -F 1796 -q 20' to drop MAPQ <20, unmapped, secondary, QC-failed, and PCR-duplicated records. Run CRAQ on the filtered BAMs; it extracts clipped reads ('S'/'H' in the CIGAR string) and classifies a locus as heterozygous when the clipping ratio falls in h (default 0.4–0.6) and as a mapping breakpoint when it exceeds f (default 0.75), excluding candidates with coverage below m (2) or above 5x average. CRAQ outputs CRE/CSE breakpoint coordinates plus CRH/CSH heterozygous calls, and converts normalized error counts into R-AQI and S-AQI scores (AQI > 90 = reference quality, 80–90 high, 60–80 draft, < 60 low). For structural misjoins, optionally break conflicting contigs at CSE positions for later correction with optical maps (e.g., Bionano DLS) or Hi-C, and inspect flagged loci visually in IGV.

## Related tools

- **CRAQ** (Core reference-free error detector: extracts clipped alignments from filtered BAMs and calls CREs/CSEs and heterozygous CRHs/CSHs at single-nucleotide resolution) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps short reads ('-ax sr') and long reads ('-ax map-hifi/map-pb/map-ont') back to the draft assembly; version 2.17+) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' before CRAQ input; version 1.3.1+)
- **IGV** (Visually inspect filtered alignments and flagged error loci)
- **pycircos** (Circos visualization of assembly features and comparative results) — https://github.com/ponnhide/pyCircos

## Examples

```
minimap2 -ax map-ont assembly.fa longreads.fq.gz | samtools view -F 1796 -q 20 -b -o ont.bam; minimap2 -ax sr assembly.fa reads_1.fq.gz reads_2.fq.gz | samtools view -F 1796 -q 20 -b -o sr.bam; perl CRAQ.pl -g assembly.fa -l ont.bam -s sr.bam -o out
```

## Evaluation signals

- On the simulated hg38 benchmark, CRAQ F1 scores should reach ~97.463% (CRE) and ~97.067% (CSE), i.e., >97% within ±2 percentage points tolerance
- On the S. pennellii Canu assembly, CRAQ should call 8029 breakpoints (7910 CREs, 119 CSEs), with ~71.4% (5736/8029) overlapping SyRI SVs
- Output BAMs must contain no MAPQ <20, unmapped, secondary, QC-failed, or duplicate records ('-F 1796 -q 20' applied)
- Computed AQI should map to the published quality bands: >90 reference, 80–90 high, 60–80 draft, <60 low
- CRAQ-unique vs SyRI-unique errors should partition into the expected categories (errors in the CaSM reference, heterozygous sites, noisy base-error clusters, low-confidence regions, shared errors, others)

## Limitations

- Evaluation of polyploid genomes may be less accurate than the diploid benchmarked cases
- Regions with clipping ratio between h (0.6) and f (0.75) are reported as 'ambiguous heterozygous or error region' without definitive classification
- CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter '-d', with no universal value stated
- Read mapping is the most resource-intensive step; users must manually split query sequences to parallelize
- Optical-mapping validation only infers approximate misjoin locations

## Evidence

- [abstract] CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants: "CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants"
- [methods] filtered out using the '-F 1796 -q 20' parameters in 'samtools view': "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’"
- [methods] a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6): "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [abstract] AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [discussion] Read mapping is currently the most resource-intensive step of CRAQ. Users could split query sequences into multiple fragments: "Read mapping is currently the most resource-intensive step of CRAQ. Users could split query sequences into multiple fragments"
- [readme] CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator: "CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator that can assess the accuracy of assembled genomic sequences"
