---
name: structural-variant-breakpoint-clustering
description: Use when assembling low-coverage HiFi reads with reference-guided chromosome
  binning and reads spanning translocation breakpoints risk being assigned to the
  wrong chromosome class.
when_to_use_negative:
- Input reads are high-coverage and will be assembled with a purely de novo assembler
  such as hifiasm alone — the translocation correction module is specific to reference-guided
  chromosome-by-chromosome binning
- The analysis targets structural variants smaller than the workflow's scope (large
  germline SVs larger than 10,000 bp detected downstream) rather than chromosome-class
  correction of reads
- No reference genome is available, since split-alignment SigBND extraction against
  a linear reference is the basis of the signal
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns HiFi reads to the linear reference genome with '-ax map-hifi' to produce
    the split alignment records used for SigBND translocation signal extraction
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Parses each read's split alignments as 6-tuples (Read_s, Read_e, Ref_s, Ref_e,
    Chr, Strand) to detect inter-chromosome split mappings
  repo: https://github.com/pysam-developers/pysam
- name: HiFiCCL
  role: 'Implements the translocation correction module: SigBND clustering, breakpoint
    read chromosome assignment, and label propagation through the Alignment Label
    Graph'
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: CuteSV
  role: Source of the breakpoint (BND) signal formulation adopted for translocation
    signal extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:2c4f41ad456a2d65e1ece4b9b457f28c7a36eedbac4f2d86f33cdf646ea1bba3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CuteSV
  - HiFiCCL
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# structural-variant-breakpoint-clustering

## Summary

Reconstructs HiFiCCL's translocation class correction: extracting split-alignment translocation signals (SigBND), clustering breakpoints by genomic-coordinate distance, and reassigning breakpoint-spanning reads to the correct chromosome so they are not misclassified during reference-guided clustering.

## When to use

Use when assembling low-coverage HiFi reads with reference-guided chromosome binning and reads spanning translocation breakpoints risk being assigned to the wrong chromosome class.

## When NOT to use

- Input reads are high-coverage and will be assembled with a purely de novo assembler such as hifiasm alone — the translocation correction module is specific to reference-guided chromosome-by-chromosome binning
- The analysis targets structural variants smaller than the workflow's scope (large germline SVs larger than 10,000 bp detected downstream) rather than chromosome-class correction of reads
- No reference genome is available, since split-alignment SigBND extraction against a linear reference is the basis of the signal

## Inputs

- PacBio HiFi/CCS FASTQ/FASTA reads at low coverage (~5x)
- Linear reference genome (e.g., GRCh38, CHM13 v2.0) in FASTA
- minimap2 '-ax map-hifi' SAM/BAM split alignment records
- Predefined breakpoint support threshold and TH_BND distance threshold

## Outputs

- SigBND translocation signal records per read
- Clustered translocation breakpoints (two deletion breakpoints, one insertion breakpoint per event)
- translocation_class_assignments.tsv with one row per read containing read-ID and 'chr'-prefixed chromosome-class fields
- Enhanced nodes in the Alignment Label Graph carrying corrected chromosome labels

## How to apply

Align HiFi reads to a linear reference (e.g., GRCh38 or CHM13) with minimap2 using '-ax map-hifi' to obtain per-read split alignment records, then parse each read's segments as (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand) 6-tuples with pysam. When two segments of a read map to different chromosomes and are separated by less than 100 bp on the read, extract a translocation signal (SigBND) following the CuteSV-style breakpoint formulation. Sort SigBND signals by genomic coordinates and cluster them greedily, retaining a signal in a cluster only when both prior and subsequent coordinate distances fall within the TH_BND threshold; discard clusters below the support threshold and refine the remaining ones to recover three breakpoints (two deletion, one insertion). Assign reads carrying translocation breakpoints the chromosome label of the insertion breakpoint site and mark them as enhanced nodes in the Alignment Label Graph; during de novo class correction traversal (the heuristic Z-drop-inspired ALG search), propagate the enhanced-node chromosome label to all reads in the path so breakpoint-spanning reads and their overlapping reads receive the corrected class.

## Related tools

- **minimap2** (Aligns HiFi reads to the linear reference genome with '-ax map-hifi' to produce the split alignment records used for SigBND translocation signal extraction) — https://github.com/lh3/minimap2
- **pysam** (Parses each read's split alignments as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand) to detect inter-chromosome split mappings) — https://github.com/pysam-developers/pysam
- **HiFiCCL** (Implements the translocation correction module: SigBND clustering, breakpoint read chromosome assignment, and label propagation through the Alignment Label Graph) — https://github.com/zjjbuqi/HiFiCCL
- **CuteSV** (Source of the breakpoint (BND) signal formulation adopted for translocation signal extraction)

## Examples

```
minimap2 -ax map-hifi GRCh38.fa hifi_reads.fq.gz > aln.sam
```

## Evaluation signals

- translocation_class_assignments.tsv exists and contains one row per read with read-ID and chromosome-class fields
- Reads spanning translocation breakpoints carry 'chr'-prefixed labels consistent with the detected insertion breakpoint chromosomes
- Retained breakpoint clusters satisfy the TH_BND distance threshold on both prior and subsequent coordinate distances and meet the predefined support threshold
- Each refined cluster resolves to exactly three breakpoints: two deletion and one insertion
- During ALG traversal, enhanced (breakpoint) nodes propagate their chromosome class to all reads in the traversed path

## Limitations

- Reference-based clustering alone can incorrectly assign reads from translocated genomic regions to the wrong chromosomes, which is exactly why the translocation correction step is required
- Version numbers for HiFiCCL are not stated in the article, and no changelog exists in the source repository
- At ~5x coverage many genomic regions remain missing, so translocation-driven corrections cannot recover reads absent from the input data

## Evidence

- [methods] their split alignment information is extracted as a translocation variant signal: "their split alignment information is extracted as a translocation variant signal"
- [methods] the Sig BND information is clustered by first sorting it based on genomic coordinates and creating a cluster: "the Sig BND information is clustered by first sorting it based on genomic coordinates and creating a cluster"
- [methods] we propose a heuristic algorithm to search the ALG and correct sequence classes: "we propose a heuristic algorithm to search the ALG and correct sequence classes, thereby completing the classes correction of reads"
- [discussion] By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints: "By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints"
- [readme] ./minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam: [evidence span withheld — anchor status: external_doc]
