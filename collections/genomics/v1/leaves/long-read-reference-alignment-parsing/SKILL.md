---
name: long-read-reference-alignment-parsing
description: Use when low-coverage HiFi reads (e.g., ~5x human/plant datasets) must
  be assigned to reference chromosomes or scanned for split mappings spanning different
  chromosomes before de novo chromosome-by-chromosome assembly.
when_to_use_negative:
- Input reads are noisy long reads (ONT or PacBio CLR) rather than HiFi/CCS — the
  map-hifi preset and downstream HiFiCCL logic assume high-fidelity reads
- The goal is splice-aware RNA-seq or short-read alignment — other minimap2 presets
  (splice, sr) apply instead
- Reads are already classified into chromosome bins and only de novo assembly (e.g.,
  hifiasm) is required
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns HiFi reads to the linear reference genome with the '-ax map-hifi' preset
    to produce per-read split alignment records
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Parses the SAM alignment output and extracts each read's split-alignment tuples
    and chromosome classes (used within HiFiCCL, version 0.21.0)
  repo: https://github.com/pysam-developers/pysam
- name: HiFiCCL
  role: Implements the alignment-parsing logic that converts split alignments into
    chromosome classes and SigBND translocation signals
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: CuteSV
  role: Source of the breakpoint (BND) signal formulation used when extracting translocation
    signals from split alignments
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:3aa0c285e00cac7956671e1d86e3a9b361e84e520e30a33a39e5db0d7c8d4b14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CuteSV
  - HiFiCCL
  - minigraph
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# long-read-reference-alignment-parsing

## Summary

Align PacBio HiFi reads to a linear reference genome with minimap2 using the '-ax map-hifi' preset and parse the resulting split-alignment records (via pysam within HiFiCCL) into per-read mapping tuples that feed chromosome classification and translocation-signal extraction in the HiFiCCL pipeline.

## When to use

Use when low-coverage HiFi reads (e.g., ~5x human/plant datasets) must be assigned to reference chromosomes or scanned for split mappings spanning different chromosomes before de novo chromosome-by-chromosome assembly.

## When NOT to use

- Input reads are noisy long reads (ONT or PacBio CLR) rather than HiFi/CCS — the map-hifi preset and downstream HiFiCCL logic assume high-fidelity reads
- The goal is splice-aware RNA-seq or short-read alignment — other minimap2 presets (splice, sr) apply instead
- Reads are already classified into chromosome bins and only de novo assembly (e.g., hifiasm) is required

## Inputs

- PacBio HiFi/CCS reads in FASTQ or FASTA format (e.g., ~5x coverage human or plant datasets)
- Linear reference genome FASTA (e.g., CHM13 v2.0, CHM13 v1.0, or GRCh38)

## Outputs

- SAM-format per-read reference alignment file (*prefix*.map_to_reference.sam)
- Per-read split-alignment 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand)
- Initial chromosome class per read ('chr'-prefixed labels or 'None' for unaligned reads)
- Translocation signals (SigBND) for reads with split alignments on different chromosomes separated by <100 bp

## How to apply

First align all HiFi reads to the linear reference (e.g., CHM13 v2.0 or GRCh38) with minimap2 using the '-ax map-hifi' preset, writing the per-read reference alignment to *prefix*.map_to_reference.sam. Parse each read's split alignments with pysam/HiFiCCL logic as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand), aggregating every chromosome class when a read maps to multiple chromosomes and labeling reads with no alignment as 'None'. When two segments of a read map to different chromosomes and are separated by less than 100 bp on the read, extract a translocation signal (SigBND) following the CuteSV-style breakpoint formulation; these signals are later sorted by genomic coordinates and greedily clustered within a TH_BND distance threshold. The rationale is that reference-based clustering alone can misassign reads from translocated genomic regions, so accurate split-alignment parsing is the foundation for both initial chromosome classes and the translocation correction module.

## Related tools

- **minimap2** (Aligns HiFi reads to the linear reference genome with the '-ax map-hifi' preset to produce per-read split alignment records) — https://github.com/lh3/minimap2
- **pysam** (Parses the SAM alignment output and extracts each read's split-alignment tuples and chromosome classes (used within HiFiCCL, version 0.21.0)) — https://github.com/pysam-developers/pysam
- **HiFiCCL** (Implements the alignment-parsing logic that converts split alignments into chromosome classes and SigBND translocation signals) — https://github.com/zjjbuqi/HiFiCCL
- **CuteSV** (Source of the breakpoint (BND) signal formulation used when extracting translocation signals from split alignments)

## Examples

```
./minimap2 -ax map-hifi CHM13v2.fa hifi_reads.fq.gz > prefix.map_to_reference.sam
```

## Evaluation signals

- Each read exits the classification stage with a non-'None' 'chr'-prefixed chromosome label wherever reference alignment information supports it, matching read-level ground-truth chromosome classes
- The reference alignment file *prefix*.map_to_reference.sam exists and contains one alignment record per input read
- Reads with segments mapping to different chromosomes and <100 bp separation on the read yield SigBND translocation signals
- Reads spanning detected translocation breakpoints carry 'chr'-prefixed labels consistent with the detected insertion breakpoint chromosomes in translocation_class_assignments.tsv
- Reads mapping to multiple chromosomes have every chromosome class aggregated rather than a single arbitrary class retained

## Limitations

- Reads with no alignment to the linear reference are labeled 'None' and require the optional pangenome-graph path (minigraph '-cx lr' plus bidirectional DFS in the Pangenome Label Graph) to be resolved
- Reference-based classification alone may incorrectly assign reads from translocated genomic regions to the wrong chromosomes without the translocation correction step
- Split-alignment translocation signals separated by >=100 bp on the read are not captured by the <100 bp rule

## Evidence

- [methods] Align the HiFi reads to the linear reference genome with minimap2 using '-ax map-hifi': [evidence span withheld — anchor status: not_found]
- [methods] split alignment signal extraction: "their split alignment information is extracted as a translocation variant signal"
- [readme] minimap2 map-hifi preset: [evidence span withheld — anchor status: external_doc]
- [discussion] translocation correction rationale: "By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints"
