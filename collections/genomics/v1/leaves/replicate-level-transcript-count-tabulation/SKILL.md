---
name: replicate-level-transcript-count-tabulation
description: Use when you have LR-Kallisto transcript quantification output for multiple
  dRNA replicates and need replicate-wise counts of transcripts classified as known,
  ISM, NIC, or NNC.
when_to_use_negative:
- Input is single-cell or bulk cDNA quantification without replicate structure rather
  than per-replicate dRNA transcript quantification
- The goal is modification-site counting (m6A, m5C, inosine, pseudouridine, Nm) rather
  than transcript isoform classification counts
- Only one sample is available, so replicate-level concordance cannot be assessed
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: LR-Kallisto
  role: Long-read transcript quantification of full-length isoforms producing the
    per-replicate count input
- name: Dogme
  role: Nextflow pipeline whose annotation entry point (annotateRNA.py, TALON-compatible
    outputs) generates the mapped BAMs and transcript classifications being tabulated
  repo: https://github.com/mortazavilab/dogme
- name: minimap2
  role: Spliced mapping of Dorado-basecalled reads (with --splice-max 500000) that
    underlies transcript detection
- name: MinION
  role: Instrument used for the three C2C12 dRNA replicates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:6081abbc10092f500d7c8bdfc0b6ccc89b430b603ca24656eaf537277b9f5723
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - LR-Kallisto
  - MinION
  - minimap2
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Replicate-level transcript count tabulation

## Summary

Aggregate per-replicate transcript detection and classification counts (known, ISM, NIC, NNC) from Dogme/LR-Kallisto outputs on dRNA MinION replicates into a validation-checked table (transcript_class_counts.csv).

## When to use

Use when you have LR-Kallisto transcript quantification output for multiple dRNA replicates and need replicate-wise counts of transcripts classified as known, ISM, NIC, or NNC.

## When NOT to use

- Input is single-cell or bulk cDNA quantification without replicate structure rather than per-replicate dRNA transcript quantification
- The goal is modification-site counting (m6A, m5C, inosine, pseudouridine, Nm) rather than transcript isoform classification counts
- Only one sample is available, so replicate-level concordance cannot be assessed

## Inputs

- LR-Kallisto transcript quantification output per dRNA replicate
- Reference transcript annotation (GTF) for splice-site matching
- Mapped/annotated BAM files from the Dogme pipeline

## Outputs

- transcript_class_counts.csv (transcript counts per class [known, ISM, NIC, NNC] per replicate)

## How to apply

For each of the three mouse C2C12 myoblast dRNA (direct RNA, MinION) replicates, load the LR-Kallisto transcript quantification output produced by the Dogme Nextflow pipeline. Match each detected full-length transcript against the reference transcript annotation to label known isoforms; for unmatched transcripts, classify splice junction chains as Incomplete Splice Match (ISM), Novel in Catalog (NIC), or Novel Not in Catalog (NNC) using splice-site matching rules. Aggregate counts into a per-class, per-replicate table (transcript_class_counts.csv). Validate that the CSV contains the four class labels and three replicates and that per-replicate class counts sum to the total number of detected transcripts in that replicate.

## Related tools

- **LR-Kallisto** (Long-read transcript quantification of full-length isoforms producing the per-replicate count input)
- **Dogme** (Nextflow pipeline whose annotation entry point (annotateRNA.py, TALON-compatible outputs) generates the mapped BAMs and transcript classifications being tabulated) — https://github.com/mortazavilab/dogme
- **minimap2** (Spliced mapping of Dorado-basecalled reads (with --splice-max 500000) that underlies transcript detection)
- **MinION** (Instrument used for the three C2C12 dRNA replicates)

## Examples

```
nextflow run mortazavilab/dogme -entry annotateRNA --inputDir mapped_bams/ --genome_annot_refs 'mm10.gtf'
```

## Evaluation signals

- transcript_class_counts.csv exists and contains all four class labels (known, ISM, NIC, NNC)
- The table contains exactly the three C2C12 dRNA replicates
- Per-replicate class counts sum to the total number of detected transcripts in that replicate (no unclassified or double-counted transcripts)
- Class proportions (majority known plus ISM; smaller NIC/NNC fractions) are concordant across the three replicates

## Limitations

- ISM/NIC/NNC assignment depends on the reference annotation and splice-site matching rules; transcripts unannotated in the reference may be mislabeled
- Requires full-length transcript detection quality from Dorado basecalling and minimap2 spliced alignment; errors in basecalling propagate to classification
- The README notes no version-tagged release or specific Dorado model checksums were recorded for reproducibility, so exact classifications may vary across pipeline versions

## Evidence

- [abstract] LR-Kallisto quantification of full-length transcripts: "while concurrently quantifying full-length transcript isoforms LR-Kallisto for transcript quantification for dRNA and cDNA"
- [abstract] Reproducible transcript expression across replicates: "The pipeline produced reproducible modification profiles and transcript expression levels across replicates"
- [abstract] Three C2C12 dRNA MinION replicates: "three separate mouse C2C12 myoblast replicates using direct RNA sequencing on MinION flow cells"
- [readme] annotateRNA.py produces TALON outputs supporting isoform classification: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 spliced alignment parameters affect transcript detection: [evidence span withheld — anchor status: external_doc]
