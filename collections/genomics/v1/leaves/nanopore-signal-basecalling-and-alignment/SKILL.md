---
name: nanopore-signal-basecalling-and-alignment
description: Use when raw nanopore direct RNA sequencing fast5 files must be assigned
  transcript identities (e.g., to label RNA classes for model training or quantify
  transcript abundance) via basecalling, transcriptome mapping, and high-confidence
  alignment filtering.
when_to_use_negative:
- You need real-time reject decisions during live sequencing — RISER classifies raw
  signal directly and is faster than GPU-accelerated basecalling and mapping, so basecall-plus-map
  is the wrong runtime path
- Reads are intended for a reference-free or de novo analysis where transcriptome
  mapping would bias the result
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files with options --flowcell FLO-MIN106 --kit SQK-RNA002
    to convert raw DRS signals into sequences
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE release 34 transcriptome with -ax map-ont
    --secondary=no
- name: samtools (v1.10)
  role: Filters alignments to high-confidence primary forward-strand mappings using
    option -F 2324
- name: BoostNano
  role: Optionally trims the sequencing adapter and poly(A) tail from raw signals
    before/at preprocessing of fast5 files
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:5fd33cc901f8e9235bd45ff5b00c52dc6084cafc380a5c0e14bec28a0f26d554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - PyTorch (v1.9.0)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nanopore-signal-basecalling-and-alignment

## Summary

Converts raw nanopore fast5 signals into mapped, high-confidence transcript alignments by basecalling with Guppy, aligning with minimap2 to a GENCODE transcriptome, and filtering with samtools — the labeling backbone used to curate RISER's mRNA/non-mRNA training data.

## When to use

Use when raw nanopore direct RNA sequencing fast5 files must be assigned transcript identities (e.g., to label RNA classes for model training or quantify transcript abundance) via basecalling, transcriptome mapping, and high-confidence alignment filtering.

## When NOT to use

- You need real-time reject decisions during live sequencing — RISER classifies raw signal directly and is faster than GPU-accelerated basecalling and mapping, so basecall-plus-map is the wrong runtime path
- Reads are intended for a reference-free or de novo analysis where transcriptome mapping would bias the result

## Inputs

- Raw nanopore direct RNA sequencing fast5 files (e.g., MinION DRS runs on FLO-MIN106 with SQK-RNA002)
- GENCODE reference transcriptome (release 34, GRCh38.p13) index for minimap2

## Outputs

- Basecalled read sequences from Guppy
- Transcriptome-aligned BAM with primary forward-strand high-confidence mappings after samtools -F 2324 filtering
- Class-labeled reads (e.g., mRNA and non-mRNA assignments from GENCODE biotypes) for downstream training-set curation

## How to apply

Basecall the model-development fast5 files with Guppy v4.0.14 using options --flowcell FLO-MIN106 --kit SQK-RNA002. Map the resulting reads to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap2 v2.17 using options -ax map-ont --secondary=no. Retain only high-confidence mappings by removing reverse-strand, secondary, supplementary, and unmapped alignments with samtools v1.10 using option -F 2324. The mapped and filtered reads then serve as ground-truth labels (e.g., mRNA vs non-mRNA via GENCODE biotypes) for downstream signal classification and other analyses.

## Related tools

- **Guppy (v4.0.14)** (Basecalls fast5 files with options --flowcell FLO-MIN106 --kit SQK-RNA002 to convert raw DRS signals into sequences)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE release 34 transcriptome with -ax map-ont --secondary=no)
- **samtools (v1.10)** (Filters alignments to high-confidence primary forward-strand mappings using option -F 2324)
- **BoostNano** (Optionally trims the sequencing adapter and poly(A) tail from raw signals before/at preprocessing of fast5 files) — https://github.com/haotianteng/BoostNano

## Examples

```
guppy_basecaller --flowcell FLO-MIN106 --kit SQK-RNA002 -i fast5/ -s basecalled/ && minimap2 -ax map-ont --secondary=no gencode_r34.fa reads.fa | samtools view -F 2324 -b -o filtered.bam
```

## Evaluation signals

- All reverse-strand, secondary, supplementary, and unmapped records are absent from the final BAM (samtools -F 2324 semantics satisfied)
- Every basecalled read maps to GENCODE release 34 (GRCh38.p13) or is explicitly removed as unmapped
- Resulting class labels (mRNA vs non-mRNA from GENCODE biotypes) yield a balanced, mappable set consistent with the paper's curation (e.g., after 50/50 undersampling, 1,073,720 training signals)
- Basecalling options match the run chemistry: flowcell FLO-MIN106 and kit SQK-RNA002

## Limitations

- Mapping-based labeling depends on transcriptome completeness; reads from poorly annotated classes may be mislabeled or dropped
- Post-sequencing bioinformatic discarding of unwanted reads (e.g., globin) is insufficient to counteract limited coverage of non-target RNAs — enrichment requires real-time rejection instead
- Biochemical/targeted labeling approaches can induce RNA degradation and compromise read quality; signal-level classification (RISER) avoids this but is a separate skill

## Evidence

- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap2 (v2.17) using options -ax map-ont --secondary=no: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] To retain only high-confidence mappings, alignments to the reverse strand, secondary or supplementary mappings and unmapped reads were removed using samtools: "To retain only high-confidence mappings, alignments to the reverse strand, secondary or supplementary mappings and unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [intro] unlike real-time basecalling, RISER can be run with just a CPU: "unlike real-time basecalling, RISER can be run with just a CPU"
- [discussion] bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs: "bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs25"
