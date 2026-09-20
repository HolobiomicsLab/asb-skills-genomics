---
name: split-alignment-translocation-signal-extraction
description: Use when HiFi reads aligned with minimap2 '-ax map-hifi' produce split
  alignment records whose segments map to different chromosomes, and you must extract
  SigBND translocation signals to recover correct chromosome labels for breakpoint-spanning
  reads.
when_to_use_negative:
- Reads produce single-segment (non-split) alignments mapping entirely to one chromosome
  — no translocation signal can be extracted and de novo class correction alone suffices
- Split segments map to different chromosomes but are separated by 100 bp or more
  on the read — the signal does not meet the <100 bp read-gap criterion for a SigBND
  event
- Signal clusters fall below the predefined support threshold and must be discarded
  rather than propagated as breakpoint labels
edam_operation: http://edamontology.org/operation_0454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns HiFi reads to the linear reference with the '-ax map-hifi' preset to
    produce the split alignment records parsed for translocation signals
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Python library (v0.21.0 in the HiFiCCL pipeline) used to parse each read's
    split alignments as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand)
  repo: https://github.com/pysam-developers/pysam
- name: HiFiCCL
  role: Host framework whose translocation class correction module consumes the SigBND
    signals, clusters them, and propagates chromosome labels via the alignment label
    graph
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: CuteSV
  role: Source of the breakpoint formulation that HiFiCCL follows when extracting
    SigBND signals from split alignments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:a6c3661e51fff7f8eda7d4d708f9e18471f47a24da703b7f8c0e4d9c16af1f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CuteSV
  - HiFiCCL
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# split-alignment-translocation-signal-extraction

## Summary

Extracts translocation (BND) variant signals from split alignments of PacBio HiFi reads against a linear reference, following a CuteSV-style breakpoint formulation, as the first stage of HiFiCCL's translocation class correction. It enables breakpoint-spanning reads to be labeled with their correct chromosome despite reference-based misclassification.

## When to use

Use when HiFi reads aligned with minimap2 '-ax map-hifi' produce split alignment records whose segments map to different chromosomes, and you must extract SigBND translocation signals to recover correct chromosome labels for breakpoint-spanning reads.

## When NOT to use

- Reads produce single-segment (non-split) alignments mapping entirely to one chromosome — no translocation signal can be extracted and de novo class correction alone suffices
- Split segments map to different chromosomes but are separated by 100 bp or more on the read — the signal does not meet the <100 bp read-gap criterion for a SigBND event
- Signal clusters fall below the predefined support threshold and must be discarded rather than propagated as breakpoint labels

## Inputs

- PacBio HiFi reads (FASTQ/FASTA)
- Linear reference genome FASTA (e.g., GRCh38 or CHM13)
- Split alignment records (SAM/BAM) from minimap2 '-ax map-hifi'
- TH_BND distance threshold and predefined cluster support threshold (CuteSV-style)

## Outputs

- SigBND translocation signal records (split-alignment-derived variant signals)
- Clustered BND signals after coordinate sorting and greedy TH_BND clustering
- Refined breakpoint set: two deletion breakpoints and one insertion breakpoint per event
- Breakpoint-spanning reads labeled with insertion-site chromosome ('chr'-prefixed) and marked as enhanced nodes in the alignment label graph

## How to apply

Align the HiFi reads to the linear reference (e.g., GRCh38 or CHM13) with minimap2 using the '-ax map-hifi' preset to obtain per-read split alignment records, parsed with pysam as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand). Emit a translocation signal (SigBND) only when two segments of the same read map to different chromosomes AND are separated by less than 100 bp on the read, using the CuteSV-style breakpoint formulation. Sort the SigBND signals by genomic coordinates and cluster them greedily, retaining a signal in a cluster when both prior and subsequent coordinate distances fall within the TH_BND threshold; discard clusters below the predefined support threshold. Refine surviving clusters to recover three breakpoints (two deletion, one insertion), then assign reads carrying the breakpoint signals the chromosome label of the insertion breakpoint site and mark them as enhanced nodes for downstream alignment-label-graph traversal.

## Related tools

- **minimap2** (Aligns HiFi reads to the linear reference with the '-ax map-hifi' preset to produce the split alignment records parsed for translocation signals) — https://github.com/lh3/minimap2
- **pysam** (Python library (v0.21.0 in the HiFiCCL pipeline) used to parse each read's split alignments as 6-tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand)) — https://github.com/pysam-developers/pysam
- **HiFiCCL** (Host framework whose translocation class correction module consumes the SigBND signals, clusters them, and propagates chromosome labels via the alignment label graph) — https://github.com/zjjbuqi/HiFiCCL
- **CuteSV** (Source of the breakpoint formulation that HiFiCCL follows when extracting SigBND signals from split alignments)

## Examples

```
./minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam
```

## Evaluation signals

- Every SigBND signal traces to a read whose two split segments map to different chromosomes with a read-coordinate gap under 100 bp
- All SigBND signals are sorted by genomic coordinates before greedy clustering, and every retained cluster satisfies the TH_BND distance threshold on both prior and subsequent coordinate distances
- Clusters below the predefined support threshold are discarded; surviving clusters refine to exactly three breakpoints (two deletion, one insertion)
- Reads spanning translocation breakpoints receive 'chr'-prefixed labels matching the detected insertion breakpoint chromosomes (verifiable via translocation_class_assignments.tsv with one row per read containing read-ID and chromosome-class fields)
- Downstream ALG traversal propagates the enhanced-node chromosome label to all reads in the path containing a breakpoint read

## Limitations

- Translocation detection depends on split alignment records from minimap2; reads whose breakpoints are not captured as inter-chromosomal split alignments within 100 bp on the read yield no SigBND signal
- Low-support signal clusters are discarded, so rare translocation events may be missed at ~5x coverage
- At low coverage, many genomic regions remain missing, which the article notes can limit completeness of downstream correction

## Evidence

- [methods] Split alignment translocation signal extraction (SigBND): "their split alignment information is extracted as a translocation variant signal"
- [methods] BND signal clustering with distance threshold: "the Sig BND information is clustered by first sorting it based on genomic coordinates and creating a cluster"
- [discussion] Translocation class correction in HiFiCCL: "By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints"
- [readme] minimap2 map-hifi preset used for HiFi read alignment: [evidence span withheld — anchor status: external_doc]
