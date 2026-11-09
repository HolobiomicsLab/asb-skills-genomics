---
name: read-chromosome-classification
description: Use when low-coverage (~5x) PacBio HiFi reads must be partitioned into
  chromosomal bins prior to per-chromosome de novo assembly, especially when translocations
  cause reads to span two chromosomes and naive reference alignment would misclassify
  them.
when_to_use_negative:
- Input reads are ONT/noisy reads — HiFiCCL is designed specifically for low-coverage
  high-fidelity (HiFi) reads
- The goal is reference-based scaffolding of existing assemblies rather than partitioning
  raw reads for de novo assembly — RagTag scaffolding is a separate, lossier approach
  that may overlook true SVs
- Coverage is high (e.g., 41x HiFi as for HG00438 standard pipelines) — the framework
  targets low-coverage cost-sensitive scenarios
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns HiFi reads to the reference with the 'map-hifi' preset to obtain split
    alignment records for translocation signal extraction
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Parses SAM/BAM split alignments into 6-tuple read segment records used to
    detect translocation signals
  repo: https://github.com/pysam-developers/pysam
- name: HiFiCCL
  role: Implements SigBND extraction, breakpoint clustering, chromosome label assignment,
    and ALG-based de novo class correction
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: CuteSV
  role: Source of the breakpoint (SigBND) signal formulation reused for translocation
    variant calling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:8b8ef7eb110bfe9363bf405bb4fe7d208ebfe3b7324d3bbe9852ce7eeafa475e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CuteSV
  - HiFiCCL
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# read-chromosome-classification

## Summary

Assigns individual HiFi reads to chromosome classes in HiFiCCL, using split-alignment translocation signals (SigBND) to correctly label breakpoint-spanning reads that reference-based clustering would otherwise assign to the wrong chromosome.

## When to use

Use when low-coverage (~5x) PacBio HiFi reads must be partitioned into chromosomal bins prior to per-chromosome de novo assembly, especially when translocations cause reads to span two chromosomes and naive reference alignment would misclassify them.

## When NOT to use

- Input reads are ONT/noisy reads — HiFiCCL is designed specifically for low-coverage high-fidelity (HiFi) reads
- The goal is reference-based scaffolding of existing assemblies rather than partitioning raw reads for de novo assembly — RagTag scaffolding is a separate, lossier approach that may overlook true SVs
- Coverage is high (e.g., 41x HiFi as for HG00438 standard pipelines) — the framework targets low-coverage cost-sensitive scenarios

## Inputs

- PacBio HiFi/CCS reads (FASTQ/FASTA, ~5x coverage)
- Linear reference genome (e.g., GRCh38 or CHM13 v2.0)
- minimap2 '-ax map-hifi' alignment records (SAM/BAM, split alignments)
- Detected translocation breakpoint signals (SigBND)
- Alignment Label Graph (ALG) with per-read chromosome classes

## Outputs

- translocation_class_assignments.tsv with one row per read (read-ID, chromosome-class)
- Per-read chromosome class labels ('chr'-prefixed) including enhanced breakpoint-spanning reads
- Chromosomal read bins for downstream chromosome-by-chromosome de novo assembly

## How to apply

Align HiFi reads to the linear reference with minimap2 using '-ax map-hifi' and parse each read's split alignments as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand) via pysam. When two segments of a read map to different chromosomes within 100 bp on the read, extract a translocation signal (SigBND) following the CuteSV-style breakpoint formulation; sort signals by genomic coordinates and greedily cluster them retaining those with prior/subsequent coordinate distances within the TH_BND threshold, discarding clusters below the support threshold and refining survivors to recover three breakpoints (two deletion, one insertion). Assign breakpoint reads the chromosome label of the insertion breakpoint site and mark them as enhanced nodes in the Alignment Label Graph (ALG). During de novo class correction — a heuristic Z-drop-inspired ALG search — propagate the enhanced-node label so all reads on the path receive the corrected chromosome class, then bin reads into chromosomal bins based on their final assigned classes.

## Related tools

- **minimap2** (Aligns HiFi reads to the reference with the 'map-hifi' preset to obtain split alignment records for translocation signal extraction) — https://github.com/lh3/minimap2
- **pysam** (Parses SAM/BAM split alignments into 6-tuple read segment records used to detect translocation signals) — https://github.com/pysam-developers/pysam
- **HiFiCCL** (Implements SigBND extraction, breakpoint clustering, chromosome label assignment, and ALG-based de novo class correction) — https://github.com/zjjbuqi/HiFiCCL
- **CuteSV** (Source of the breakpoint (SigBND) signal formulation reused for translocation variant calling)

## Examples

```
minimap2 -ax map-hifi GRCh38.fa hifi_reads.fq.gz > aln.sam
```

## Evaluation signals

- translocation_class_assignments.tsv exists and contains exactly one row per read with read-ID and chromosome-class fields
- Reads spanning translocation breakpoints carry 'chr'-prefixed labels consistent with the detected insertion breakpoint chromosomes
- Enhanced (breakpoint) nodes propagate their class to all reads encountered along the ALG traversal path
- Discarded SigBND clusters must fall below the predefined support threshold; retained clusters satisfy the TH_BND coordinate distance criterion on both sides
- Downstream check: misassembled contig length after per-chromosome assembly with hifiasm decreases relative to hifiasm alone (article reports average 21.19% reduction, up to 38.58%)

## Limitations

- Split-alignment translocation signals only trigger when the two read segments map to different chromosomes separated by less than 100 bp on the read
- SigBND clusters below the predefined support threshold are discarded, so weakly supported translocations yield no label correction
- Reference-based chromosome binning alone can misclassify translocated reads; the translocation correction module and de novo correction module are required to fix these classes
- Many genomic regions remain missing at ~5x coverage, so not all reads receive a reliable chromosome class

## Evidence

- [methods] Split alignment translocation signal extraction (SigBND): "their split alignment information is extracted as a translocation variant signal"
- [methods] BND signal clustering: "the Sig BND information is clustered by first sorting it based on genomic coordinates and creating a cluster"
- [methods] De novo class correction via ALG search: "Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes"
- [discussion] Translocation correction rationale: "By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints"
- [readme] minimap2 HiFi preset: [evidence span withheld — anchor status: external_doc]
