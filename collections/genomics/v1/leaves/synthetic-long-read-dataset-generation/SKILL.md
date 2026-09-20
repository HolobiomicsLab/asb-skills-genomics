---
name: synthetic-long-read-dataset-generation
description: Use when you need ground-truth structural-variant datasets to benchmark
  a long-read SV detection or annotation tool, e.g. testing whether deletions, inversions,
  duplications, translocations, or complex insertion-deletion rearrangements are resolved
  correctly.
when_to_use_negative:
- The synthetic reads contain no background mutations or sequence variants, so the
  data are unsuitable for benchmarking SNV/small-indel callers or for any use beyond
  algorithm testing.
- Do not use this approach when real experimental read characteristics (error profiles,
  uneven coverage) are required — MakeSVGenome data alternate orientation at fixed
  length/spacing and are explicitly not real experimental data.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: MakeSVGenome
  role: Creates chromosomes with structural rearrangements and synthetic long-read
    data (fixed-length, fixed-spacing, alternating-orientation reads) for testing
    SV detection pipelines
  repo: https://github.com/msjimc/MakeSVGenome
- name: minimap2
  role: Long-read-specific aligner used with '-ax map-ont --secondary=no' to align
    the synthetic reads to hg19/mm10
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts, sorts, and indexes minimap2 SAM output to BAM for loading into AgileStructure
- name: AgileStructure
  role: Desktop application used to annotate the synthetic rearrangements from the
    aligned BAM to measure annotation accuracy
  repo: https://github.com/msjimc/AgileStructure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:80254f77f2febd0dd1a221ba61f7151ee7e074ba95d80800f8aa6e717400a7a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/MakeSVGenome
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# synthetic-long-read-dataset-generation

## Summary

Generate in silico long-read test datasets with MakeSVGenome — fixed-length fixed-spacing reads of alternating orientation carrying known structural rearrangements — then align with minimap2 and benchmark annotation accuracy (e.g. of AgileStructure) against the ground-truth variants.

## When to use

Use when you need ground-truth structural-variant datasets to benchmark a long-read SV detection or annotation tool, e.g. testing whether deletions, inversions, duplications, translocations, or complex insertion-deletion rearrangements are resolved correctly.

## When NOT to use

- The synthetic reads contain no background mutations or sequence variants, so the data are unsuitable for benchmarking SNV/small-indel callers or for any use beyond algorithm testing.
- Do not use this approach when real experimental read characteristics (error profiles, uneven coverage) are required — MakeSVGenome data alternate orientation at fixed length/spacing and are explicitly not real experimental data.

## Inputs

- Reference genome FASTA (hg19 or mm10)
- Batch file or manual specification of structural rearrangements for MakeSVGenome (simple and complex insertion-deletion events)
- MakeSVGenome parameters such as read length (10 kb) and read spacing (1 kb)

## Outputs

- Synthetic long-read sequencing dataset with known structural variants embedded
- Minimap2-aligned SAM/BAM indexed with Samtools
- Per-rearrangement annotation calls (unambiguous vs ambiguous) from the tool under test
- Benchmark table (e.g. 66 of 68 rearrangements unambiguously annotated, with the 2 ambiguous pairs identified)

## How to apply

Use MakeSVGenome (manually or in batch mode via a batch file) to create chromosomes — here human chromosomes 7 and 8 — incorporating 32 simple rearrangements (deletions, insertions, inversions, duplications, translocations, ring chromosome) and 36 complex insertion-deletion rearrangements, emitting fixed-length faux reads (10 kb) at 1 kb intervals with alternating orientations and no other mutations. Align the reads to hg19/mm10 with minimap2 using '-ax map-ont --secondary=no', then convert, sort, and index the SAM to BAM with Samtools. Load the indexed BAM into the tool under test (AgileStructure), screen candidate loci in blocks of up to 10 Mb, and call breakpoints where 2 or more primary/secondary junction positions cluster within 250 nucleotides. Compare the annotations per variant against the ground truth to count unambiguous vs ambiguous resolutions.

## Related tools

- **MakeSVGenome** (Creates chromosomes with structural rearrangements and synthetic long-read data (fixed-length, fixed-spacing, alternating-orientation reads) for testing SV detection pipelines) — https://github.com/msjimc/MakeSVGenome
- **minimap2** (Long-read-specific aligner used with '-ax map-ont --secondary=no' to align the synthetic reads to hg19/mm10) — https://github.com/lh3/minimap2
- **Samtools** (Converts, sorts, and indexes minimap2 SAM output to BAM for loading into AgileStructure)
- **AgileStructure** (Desktop application used to annotate the synthetic rearrangements from the aligned BAM to measure annotation accuracy) — https://github.com/msjimc/AgileStructure

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa synthetic_reads.fq | samtools sort -o aligned.bam && samtools index aligned.bam
```

## Evaluation signals

- Exactly 66 of 68 in silico rearrangements are annotated unambiguously by AgileStructure, matching the paper's Table 2 outcome.
- Only the two documented ambiguous scenarios fail resolution: ring chromosome vs tandem duplication, and offset inversion vs inverted insertion-deletion with loss at the insertion site.
- Aligned BAM contains no secondary alignments because minimap2 was run with --secondary=no.
- Synthetic reads alternate between forward and reverse orientations at fixed 10 kb length and 1 kb spacing, and contain no other mutations or sequence variants, confirming the data are MakeSVGenome-generated.

## Limitations

- MakeSVGenome adds no other mutations or sequence variants beyond the engineered rearrangements, so the data cannot test a pipeline's tolerance to background noise or sequencing error.
- Only two ambiguous scenario pairs exist in this benchmark; other edge cases of complex rearrangements may not be covered.
- Breakpoint calling depends on user-guided inspection: 2 or more primary/secondary junction positions clustering within 250 nucleotides, screened in blocks of up to 10 Mb.

## Evidence

- [readme] MakeSVGenome creates synthetic long-read datasets for structural variant testing: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic reads are fixed-length, fixed-spacing, alternating orientation, with no other mutations: [evidence span withheld — anchor status: external_doc]
- [abstract] AgileStructure annotates large-scale rearrangements from aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [abstract] Source code and example aligned read data available on GitHub: "Source code, binaries, user guide, and example aligned read data, are available on GitHub: https://github.com/msjimc/AgileStructure."
- [intro] minimap2 is the typical long-read aligner used for this data: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
