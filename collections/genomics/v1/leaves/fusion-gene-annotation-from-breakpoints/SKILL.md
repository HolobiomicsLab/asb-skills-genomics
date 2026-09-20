---
name: fusion-gene-annotation-from-breakpoints
description: Use when nanopore FASTQ (or BAM) reads from fusion-enriched samples must
  be converted into a table of gene-pair fusion calls with per-call fusion-spanning
  read counts.
when_to_use_negative:
- Input DNA is fragmented FFPE material — the authors state the assay requires non-fragmented
  DNA and FUDGE will most likely fail to comprehensively identify fusion genes on
  FFPE
- Low-DNA WGA samples where neither fusion gene partner is known — WGA use is only
  useful for breakpoint identification with prior knowledge of both partners
- Fusions supported by only one fusion-spanning read — these fall below the two-read
  cut-off and required manual breakpoint PCR validation in the paper
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: NanoFG
  role: End-to-end pipeline orchestrating SV calling and fusion gene annotation from
    nanopore data
  repo: https://github.com/SdeBlank/NanoFG
- name: minimap2
  role: Maps enriched nanopore FASTQ reads to the GRCh37 human reference genome (v2.6)
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: Structural variant caller for long reads (v1.2.4) whose output is annotated
    for fusion detection
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller (v1.0.9, parameters -s 2 -n -1 --genotype)
- name: samtools
  role: Integrated by NanoFG for BAM handling during alignment processing
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:0a1d95d300ebedd219041ac389c9a535d1e9a0380860e9841538525e22b74ff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - Minimap2
  - NanoFG
  - NanoSV
  - Samtools
  - Sniffles
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# fusion-gene-annotation-from-breakpoints

## Summary

Detect and annotate fusion genes from enriched nanopore reads using the NanoFG pipeline: map reads with minimap2 to GRCh37, call structural variants with NanoSV or Sniffles, flag SVs whose two ends fall in different ENSEMBL genes as candidate fusions, and retain calls supported by at least two fusion-spanning reads.

## When to use

Use when nanopore FASTQ (or BAM) reads from fusion-enriched samples must be converted into a table of gene-pair fusion calls with per-call fusion-spanning read counts.

## When NOT to use

- Input DNA is fragmented FFPE material — the authors state the assay requires non-fragmented DNA and FUDGE will most likely fail to comprehensively identify fusion genes on FFPE
- Low-DNA WGA samples where neither fusion gene partner is known — WGA use is only useful for breakpoint identification with prior knowledge of both partners
- Fusions supported by only one fusion-spanning read — these fall below the two-read cut-off and required manual breakpoint PCR validation in the paper

## Inputs

- nanopore FASTQ reads from FUDGE-enriched samples (cell lines, tumor material, blinded samples, WGA material, multiplexed libraries)
- GRCh37 human reference genome
- pre-computed BAM and VCF files (alternative NanoFG entry mode)

## Outputs

- fusion_calls.tsv with sample name, fusion gene pair, and fusion-spanning read count per call
- filtered VCF of structural variants passing NanoSV/Sniffles built-in filters (insertions removed)

## How to apply

Clone NanoFG from github.com/SdeBlank/NanoFG, create a python3 virtualenv, and adjust tool paths in paths.ini. Map the enriched nanopore FASTQ reads to the GRCh37 human reference genome with minimap2 (v2.6), then call SVs with NanoSV (v1.2.4; min_mapq=12, depth_support=False, mapq_flag=48) or Sniffles (v1.0.9; -s 2 -n -1 --genotype). Remove all SVs failing the built-in filters plus all insertions from the VCF. Annotate both ends of each SV with ENSEMBL genes and flag SVs whose ends lie in different genes as candidate fusions, retaining only fusions able to produce a continuous transcript on the same strand. Finally apply the reliability cut-off of at least two fusion-spanning reads and tabulate sample name, fusion gene pair, and fusion-spanning read count (e.g. KMT2A-MLLT6 with 29 reads in sample B1 and BCR-ABL1 with 27 reads in B2).

## Related tools

- **NanoFG** (End-to-end pipeline orchestrating SV calling and fusion gene annotation from nanopore data) — https://github.com/SdeBlank/NanoFG
- **minimap2** (Maps enriched nanopore FASTQ reads to the GRCh37 human reference genome (v2.6)) — https://github.com/lh3/minimap2
- **NanoSV** (Structural variant caller for long reads (v1.2.4) whose output is annotated for fusion detection) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller (v1.0.9, parameters -s 2 -n -1 --genotype))
- **samtools** (Integrated by NanoFG for BAM handling during alignment processing) — http://samtools.sourceforge.net/

## Examples

```
git clone https://github.com/SdeBlank/NanoFG.git NanoFG && cd NanoFG && virtualenv venv -p python3 && . venv/bin/activate && pip install -r requirements.txt && bash NanoFG.sh -f /path/to/fastqdir -n SAMPLE_NAME
```

## Evaluation signals

- Every fusion call in fusion_calls.tsv is supported by at least two fusion-spanning reads, matching the paper's cut-off for reliable detection without further validation
- All SVs failing NanoSV/Sniffles built-in filters and all insertions have been removed from the VCF before annotation
- Retained fusions have both SV ends in different genes and can produce a continuous transcript on the same strand
- Fusions between known gene pairs recapitulate expected calls, e.g. KMT2A-MLLT6 in blinded sample B1 (29 reads) and BCR-ABL1 in B2 (27 reads)
- NanoFG completes on ~25000 nanopore reads in roughly 20 min on a single thread, as a runtime sanity check

## Limitations

- Requires non-fragmented DNA; FFPE material will most likely fail to yield comprehensive fusion identification
- Two fusions detected with only one fusion-spanning read each required manual validation by breakpoint PCR
- WGA of low-input DNA only identifies breakpoint location when both fusion gene partners are known in advance
- On-target coverage decreases for low-throughput sequencing runs and more distal breakpoint events

## Evidence

- [discussion] By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF.: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [discussion] We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation.: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] If both ends of the SV are positioned in different genes it was flagged as a possible fusion.: "If both ends of the SV are positioned in different genes it was flagged as a possible fusion."
- [discussion] Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6): "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
- [readme] NanoFG integrates existing genomic tools and additonal code to provide an easy-to-use pipeline for the detection of fusion genes from Nanopore sequencing data: "NanoFG integrates existing genomic tools and additonal code to provide an easy-to-use pipeline for the detection of fusion genes from Nanopore sequencing data"
- [discussion] Only fusions that have the ability to produce a continuous transcript on the same strand were retained: "Only fusions that have the ability to produce a continuous transcript on the same strand were retained"
