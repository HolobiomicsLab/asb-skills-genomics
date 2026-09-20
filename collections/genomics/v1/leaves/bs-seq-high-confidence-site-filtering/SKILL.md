---
name: bs-seq-high-confidence-site-filtering
description: Use when building labeled positive/negative 5mC site sets from plant
  nanopore + BS-seq pairs to train or benchmark methylation callers (especially CHH
  context on R10.4 FlowCell data).
when_to_use_negative:
- When the goal is methylation calling on new nanopore reads rather than building
  training labels — use DeepPlant inference (extract_and_call_mods) instead
- When trying to generate CHH-positive samples by in vitro enzyme-catalyzed methylation
  — the paper states this is ineffective because CHH methylation depends on RNA-directed
  DNA methylation (RdDM) requiring non-coding RNA guidance and DRM1/DRM2
- When working with non-plant species or R9.4 platform data, where the tooling (Tombo,
  Megalodon, DeepSignal-Plant lineage) and sample-abundance screening assumptions
  do not transfer
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0654
tools:
- name: DeepPlant
  role: Provides the extract_hc_sites routine that labels high-confidence positive/negative
    methylation sites from POD5 + BAM + BS-seq evidence for k-mer-context training
    data
  repo: github.com/xiaochuanle/DeepPlant
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform whose improved basecalling accuracy and stability supplies
    the nanopore signal data aligned in the input BAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:2d26cc697b2dd3ac4496eb6afaa049bb9837eb6c200f2845d454d9bbe4791861
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bs-seq-high-confidence-site-filtering

## Summary

Extracts high-confidence methylation-positive and methylation-negative cytosine sites from reanalyzed bisulfite sequencing (BS-seq) data on Oxford Nanopore R10.4 read alignments, producing labeled per-site training samples with defined k-mer contexts. This is the DeepPlant approach for overcoming the scarcity of high-methylation CHH training samples in plants.

## When to use

Use when building labeled positive/negative 5mC site sets from plant nanopore + BS-seq pairs to train or benchmark methylation callers (especially CHH context on R10.4 FlowCell data).

## When NOT to use

- When the goal is methylation calling on new nanopore reads rather than building training labels — use DeepPlant inference (extract_and_call_mods) instead
- When trying to generate CHH-positive samples by in vitro enzyme-catalyzed methylation — the paper states this is ineffective because CHH methylation depends on RNA-directed DNA methylation (RdDM) requiring non-coding RNA guidance and DRM1/DRM2
- When working with non-plant species or R9.4 platform data, where the tooling (Tombo, Megalodon, DeepSignal-Plant lineage) and sample-abundance screening assumptions do not transfer

## Inputs

- POD5 nanopore read directory (R10.4 FlowCell data)
- BAM file aligning nanopore reads to the reference genome
- Reference genome FASTA and reference type specifier
- Reanalyzed BS-seq data defining methylation-positive and methylation-negative cytosines
- k-mer size parameter (9 for CHH-centered 9-mer contexts) and num_workers thread count

## Outputs

- High-confidence positive and negative methylation site lists (per-cytosine labels with sequence context)
- 9-mer CHH context coverage report (e.g., chh_9mer_coverage_report.csv with per-9-mer positive/negative support counts and coverage fraction)
- Training/test sample datasets covering diverse 9-mer motifs for DeepPlant

## How to apply

Reanalyze publicly available BS-seq datasets to first screen species with abundant high-methylation CHH sites (e.g., S. miltiorrhiza, S. tuberosum, R. communis), then generate matched nanopore data for those species. Run the DeepPlant extract_hc_sites routine, which takes the POD5 read directory, the read-to-reference BAM, the reference sequence and its type, a write directory, positive and negative site lists, a k-mer size (the DeepPlant training set uses 9-mers centered on CHH motifs), and a number of worker threads. The routine labels cytosines as high-confidence positive or negative sites from the bisulfite evidence, yielding per-site k-mer context assignments. Assess coverage by enumerating all 36,864 possible 9-mer contexts (9 CHH central trinucleotides x 4^6 flanking bases) and counting contexts supported by >=50 positive and >=50 negative samples; the DeepPlant combined three-species training set reaches 97.2% coverage averaging 9225 samples per context.

## Related tools

- **DeepPlant** (Provides the extract_hc_sites routine that labels high-confidence positive/negative methylation sites from POD5 + BAM + BS-seq evidence for k-mer-context training data) — github.com/xiaochuanle/DeepPlant
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform whose improved basecalling accuracy and stability supplies the nanopore signal data aligned in the input BAM)

## Examples

```
extract_hc_sites ./pod5 ./reads.bam reference.fa genome ./out pos_sites.csv neg_sites.csv 9 8
```

## Evaluation signals

- Enumerate all 36,864 candidate 9-mer contexts and verify that a 9-mer counts as covered only when it has >=50 occurrences in both the positive and negative site sets
- Compare the computed covered-context fraction against the reported 97.2% for the combined S. miltiorrhiza / S. tuberosum / R. communis dataset
- Check the mean per-context sample count against the reported average of over 9225 samples per 9-mer CHH context
- Verify the central trinucleotide of each extracted 9-mer matches a CHH motif (9 possible central trinucleotides) and flanks span all 4^6 A/C/G/T combinations
- Confirm the output schema includes per-9-mer positive and negative support counts (e.g., rows in chh_9mer_coverage_report.csv)

## Limitations

- The scarcity of high-methylation CHH samples, although partially mitigated in this study, continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- In vitro enzyme-catalyzed DNA methylation cannot substitute for BS-seq-screened natural samples because CHH methylation depends on RdDM requiring non-coding RNA guidance and DRM1/DRM2 methyltransferases

## Evidence

- [methods] extract_hc_sites: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [discussion] training dataset coverage: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [abstract] rationale for BS-seq screening: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing"
- [discussion] ongoing scarcity caveat: "The scarcity of high-methylation CHH samples, although partially mitigated in this study, continues to pose challenges for model training and generalization"
- [readme] DeepPlant README framing: "CHH methylation detection in plants is limited by the scarcity of high-methylation positive samples, reducing generalization across species."
