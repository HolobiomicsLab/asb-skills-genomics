---
name: fusion-spanning-read-filtering
description: Use when you have ONT nanopore FASTQ reads (optionally with an existing
  BAM/VCF) from fusion-gene-enriched samples and must call fusions from structural
  variants with reproducible reliability cutoffs.
when_to_use_negative:
- Input is FFPE or otherwise fragmented DNA — the FUDGE crRNA protocol requires non-fragmented
  DNA and will most likely fail to comprehensively identify fusion genes
- Sequencing data is short-read (Illumina) rather than long-read nanopore — NanoFG
  is built around long-read SV calling with NanoSV/Sniffles
- Material is WGA-amplified from very low DNA input with only one known fusion partner
  — WGA breakpoint identification only works with prior knowledge of both fusion gene
  partners
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NanoFG
  role: Pipeline integrating mapping, SV calling, gene annotation, fusion flagging,
    filtering and primer design for nanopore fusion detection
  repo: https://github.com/SdeBlank/NanoFG
- name: Minimap2
  role: Maps long ONT reads to the GRCh37 reference genome (v2.6) before SV calling
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: Calls structural variants from long-read alignments (v1.2.4) whose gene-spanning
    ends become candidate fusions
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller (v1.0.9) whose output VCFs are accepted by NanoFG
- name: Primer3
  role: Designs breakpoint-spanning validation primers for reported fusions
  repo: https://github.com/primer3-org/primer3
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:7f42de8351289b502d89cbba2682413a0cd8fe8b26645cc9d9d8aae7371ea9f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - LAST
  - Minimap2
  - NanoFG
  - NanoSV
  - Primer3
  - Samtools
  - Sniffles
  - Wtdbg2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# fusion-spanning-read-filtering

## Summary

Filters candidate fusion genes from long-read nanopore sequencing data using the NanoFG logic: keep only same-strand SVs whose two ends fall in different ENSEMBL genes and that are supported by at least two fusion-spanning reads, yielding reliable fusion calls with exact breakpoints and Primer3 validation primers.

## When to use

Use when you have ONT nanopore FASTQ reads (optionally with an existing BAM/VCF) from fusion-gene-enriched samples and must call fusions from structural variants with reproducible reliability cutoffs.

## When NOT to use

- Input is FFPE or otherwise fragmented DNA — the FUDGE crRNA protocol requires non-fragmented DNA and will most likely fail to comprehensively identify fusion genes
- Sequencing data is short-read (Illumina) rather than long-read nanopore — NanoFG is built around long-read SV calling with NanoSV/Sniffles
- Material is WGA-amplified from very low DNA input with only one known fusion partner — WGA breakpoint identification only works with prior knowledge of both fusion gene partners

## Inputs

- Nanopore FASTQ files (or aligned BAM plus SV VCF) from targeted fusion-gene-enriched samples
- GRCh37 human reference genome
- ENSEMBL gene annotation database

## Outputs

- Filtered fusion gene calls (fusion partners, exact breakpoint locations, breakpoint sequences) passing the >=2 fusion-spanning-read cutoff
- Fusion-spanning read counts per fusion (e.g., tabulated in fusion_calls.tsv)
- Breakpoint-spanning validation primer sequences designed with Primer3

## How to apply

Map enriched nanopore reads to the GRCh37 human reference genome with minimap2 (v2.6), then call structural variants with NanoSV (v1.2.4; min_mapq=12, depth_support=False, mapq_flag=48) or Sniffles (v1.0.9; -s 2 -n -1 --genotype). Remove all SVs failing the built-in NanoSV/Sniffles filters and all insertions from the VCF, annotate both ends of each SV with ENSEMBL genes, and flag SVs whose two ends lie in different genes as possible fusions. Retain only fusions able to produce a continuous transcript on the same strand, and apply the paper's cut-off of at least two fusion-spanning reads per fusion to call a fusion reliably without further validation; fusions supported by a single read require manual breakpoint PCR validation. The rationale is that two independent fusion-spanning reads plus a continuous same-strand transcript model minimize false positives from mapping artifacts in noisy ONT data.

## Related tools

- **NanoFG** (Pipeline integrating mapping, SV calling, gene annotation, fusion flagging, filtering and primer design for nanopore fusion detection) — https://github.com/SdeBlank/NanoFG
- **Minimap2** (Maps long ONT reads to the GRCh37 reference genome (v2.6) before SV calling) — https://github.com/lh3/minimap2
- **NanoSV** (Calls structural variants from long-read alignments (v1.2.4) whose gene-spanning ends become candidate fusions) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller (v1.0.9) whose output VCFs are accepted by NanoFG)
- **Primer3** (Designs breakpoint-spanning validation primers for reported fusions) — https://github.com/primer3-org/primer3

## Examples

```
bash NanoFG.sh -f /path/to/fastqdir -n SAMPLE_NAME
```

## Evaluation signals

- Every reported fusion is supported by at least two fusion-spanning reads (single-read fusions flagged for manual breakpoint PCR validation)
- Both SV breakpoints map to two different ENSEMBL genes on the same strand, with a continuous transcript possible
- VCF contains no insertions and no SVs failing built-in NanoSV/Sniffles filters
- Final fusion report lists fusion partners, exact breakpoint locations, breakpoint sequences and fusion-spanning primers (verified on the bundled NanoFG test set)
- Run time sanity check: NanoFG processes ~25000 nanopore reads in approximately 20 min on a single thread

## Limitations

- Two fusions detected with only one fusion-spanning read each require manual validation by breakpoint PCR
- Decreased on-target coverage observed for low-throughput sequencing runs and/or more distal breakpoint events
- WGA for low-DNA input only identifies breakpoint locations when both fusion gene partners are already known
- Intratumoral heterogeneity and tumor purity likely influence the lower detection limits

## Evidence

- [discussion] Candidate fusion flagging: "If both ends of the SV are positioned in different genes it was flagged as a possible fusion."
- [discussion] Same-strand transcript filter: "Only fusions that have the ability to produce a continuous transcript on the same strand were retained"
- [discussion] Reliability cutoff: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] VCF pre-filtering: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [readme] Pipeline scope: "NanoFG integrates existing genomic tools and additonal code to provide an easy-to-use pipeline for the detection of fusion genes from Nanopore sequencing data"
