---
name: amplicon-read-orientation-correction
description: Use when INC-Seq circular consensus reads from Oxford Nanopore 16S amplicon
  runs are inconsistently oriented or span the full insert and must be fragmented
  and orientation-corrected before nanoClust clustering or blastn-based accuracy assessment.
when_to_use_negative:
- Input is raw, noisy MinION read data before INC-Seq circular consensus generation
  — chopSeq operates on INC-Seq consensus reads, not raw reads
- Input is already cluster-level OTU consensus sequences from nanoClust — orientation
  correction applies at the read level
- Data are short reads from Illumina MiSeq or Ion Torrent PGM rather than full-length
  nanopore amplicon reads
edam_operation: http://edamontology.org/operation_2003
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads and corrects read orientation; the chopSeq_SS
    variant additionally performs 1,300–1,450 bp size selection
  repo: github.com/umerijaz/nanopore
- name: blastn
  role: Computes the aligned proportion and sequence accuracy of chopped reads against
    reference 16S sequences to validate the correction step
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:5bbc74a3c9b5873629a09f4e08a507623c85d5f5c94bd575a3175f8ea67b0197
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - blastn
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_2003
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# amplicon-read-orientation-correction

## Summary

Fragmentation and read orientation correction of INC-Seq consensus reads using chopSeq, the NanoAmpli-Seq component that converts circular-consensus nanopore reads into consistently oriented, optionally size-selected fragments prior to de novo clustering.

## When to use

Use when INC-Seq circular consensus reads from Oxford Nanopore 16S amplicon runs are inconsistently oriented or span the full insert and must be fragmented and orientation-corrected before nanoClust clustering or blastn-based accuracy assessment.

## When NOT to use

- Input is raw, noisy MinION read data before INC-Seq circular consensus generation — chopSeq operates on INC-Seq consensus reads, not raw reads
- Input is already cluster-level OTU consensus sequences from nanoClust — orientation correction applies at the read level
- Data are short reads from Illumina MiSeq or Ion Torrent PGM rather than full-length nanopore amplicon reads

## Inputs

- INC-Seq circular consensus reads (from rolling-circle alignment of raw MinION nanopore amplicon reads, retaining reads supporting at least 3 concatemers)
- Reference 16S rRNA sequences (e.g., from the mock-community deposit at github.com/umerijaz/nanopore) for blastn validation

## Outputs

- Fragmented, orientation-corrected reads (chopped reads) in FASTA/FASTQ form, ready for nanoClust
- Size-selected reads of 1,300–1,450 bp when using the chopSeq_SS variant
- blastn-derived aligned-proportion and sequence-accuracy metrics (e.g., metrics.csv)

## How to apply

Run chopSeq on the INC-Seq consensus reads to fragment them and correct read orientation, with no size selection in the first pass; then optionally re-run the size-selection variant (chopSeq_SS) retaining reads of 1,300–1,450 bp. Validate each pass by blastn against the reference 16S sequences: aligned proportion should rise from ~73±14.6% (INC-Seq output) to ~96±2.3% after chopSeq, and to ~98.4±0.7% after chopSeq_SS, with overall sequence accuracy reaching ~98±0.23%. The rationale is that correcting orientation and selecting near-full-length fragments lets downstream nanoClust partition reads into clusters and call within-cluster consensus for accurate full-length 16S rRNA gene sequences.

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads and corrects read orientation; the chopSeq_SS variant additionally performs 1,300–1,450 bp size selection) — github.com/umerijaz/nanopore
- **blastn** (Computes the aligned proportion and sequence accuracy of chopped reads against reference 16S sequences to validate the correction step)

## Evaluation signals

- blastn aligned proportion after chopSeq (no size selection) falls within ~96±2.3%
- blastn aligned proportion after chopSeq with 1,300–1,450 bp size selection (chopSeq_SS) falls within ~98.4±0.7%
- Overall sequence accuracy after chopSeq_SS, measured by blastn against the reference, reaches ~98±0.23%
- All corrected reads are consistently oriented relative to the reference 16S sequences
- Aligned proportions and accuracy values are collected into a metrics table (metrics.csv) matching the reported error ranges

## Limitations

- Without size selection, the aligned proportion after chopSeq is ~96±2.3% rather than the ~98.4±0.7% achieved with 1,300–1,450 bp size selection
- Residual errors remaining after the NanoAmpli-Seq workflow are dominated by deletions in homopolymer regions
- Validation ranges are wide at the INC-Seq stage (~73±14.6%), so intermediate metrics should be interpreted with their reported variability

## Evidence

- [abstract] chopSeq fragmentation and orientation correction: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] nanoClust downstream clustering: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [readme] nanopore amplicon processing scripts: "Scripts for processing short amplicon reads from Oxford Nanopore."
