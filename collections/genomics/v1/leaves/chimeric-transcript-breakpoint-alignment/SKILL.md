---
name: chimeric-transcript-breakpoint-alignment
description: Use when detecting fusion transcripts in single cells from long-read
  (ONT or PacBio) sequencing of 10x Genomics 5' whole-transcriptome cDNA libraries,
  e.g., counting BCR::ABL1+ K562 or RUNX1::RUNX1T1+ Kasumi-1 cells in a mixing library.
when_to_use_negative:
- Input is Illumina short-read data — short-read sequencing cannot easily resolve
  structural transcriptomic variants such as gene fusions or isoforms
- The target fusion junction lies far from the 5' end of a lowly expressed or long
  (>4 kB) transcript, where nanoranger sensitivity is inherently limited
- Input is bulk RNA-seq or genomic DNA rather than single-cell cDNA carrying 10x cell
  barcode and UMI information
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: nanoranger
  role: Long-read single-cell pipeline run in 5p10XGEX fusion mode to align reads
    to fusion junctions and extract cell barcode/UMI from chimeric reads
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R downstream package that reads out fusion genes/CAR sequences from nanoranger
    amplicon output and collapses UMIs with starcode (Levenshtein distance 3)
  repo: https://github.com/liviuspenter/nanoranger.R
- name: minimap2
  role: Splice-aware long-read aligner for mapping noisy ONT cDNA reads against the
    reference
  repo: https://github.com/lh3/minimap2
- name: STAR
  role: Spliced aligner option for chimeric/spliced transcript alignment in the workflow
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:5deb2c4c6d0cd613cfff8e3ecdf6a7e426be8a0e1a2923e8b417689365425665
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - STAR
  - minimap2
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chimeric-transcript-breakpoint-alignment

## Summary

Detects fusion genes at single-cell resolution by aligning ONT long reads from 5'-anchored 10x Genomics scRNA-seq cDNA libraries to reference transcripts spanning fusion junctions (e.g., BCR::ABL1, RUNX1::RUNX1T1) with nanoranger, then deduplicating barcode-UMI pairs per cell to call fusion-positive cells.

## When to use

Use when detecting fusion transcripts in single cells from long-read (ONT or PacBio) sequencing of 10x Genomics 5' whole-transcriptome cDNA libraries, e.g., counting BCR::ABL1+ K562 or RUNX1::RUNX1T1+ Kasumi-1 cells in a mixing library.

## When NOT to use

- Input is Illumina short-read data — short-read sequencing cannot easily resolve structural transcriptomic variants such as gene fusions or isoforms
- The target fusion junction lies far from the 5' end of a lowly expressed or long (>4 kB) transcript, where nanoranger sensitivity is inherently limited
- Input is bulk RNA-seq or genomic DNA rather than single-cell cDNA carrying 10x cell barcode and UMI information

## Inputs

- Gzipped FASTQ of ONT long reads from a 10x Genomics 5' whole-transcriptome cDNA library (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz) with cell barcode and UMI sequences in read IDs
- Reference transcripts spanning the target fusion junctions (e.g., BCR-ABL1 and RUNX1-RUNX1T1)

## Outputs

- fusion_calls.tsv table with fusion gene entries (BCR::ABL1, RUNX1::RUNX1T1) and cell barcode and UMI fields
- Per-cell fusion-positive counts and fractions (e.g., 124 (2.7%) BCR::ABL1+ K562 cells; 88 (3.9%) RUNX1::RUNX1T1+ Kasumi-1 cells)

## How to apply

Start from a gzipped FASTQ of ONT long reads derived from a 10x Genomics 5' GEX whole-transcriptome cDNA library (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz) and run nanoranger in 5p10XGEX fusion mode, aligning reads to reference transcripts spanning the fusion junctions of interest. Retain only chimeric reads that span the fusion breakpoint, together with their 10x cell barcode (CB) and UMI sequences. Run the downstream fusion_gene.py script to deduplicate barcode-UMI pairs per cell — in the R downstream package (nanoranger.R), UMIs are collapsed with starcode within a Levenshtein distance of 3 because 10x UMIs have no whitelist — and tabulate fusion-supporting reads per fusion gene. Compare the resulting fusion-positive cell counts and fractions against benchmarks (124 (2.7%) BCR::ABL1+ K562 cells; 88 (3.9%) RUNX1::RUNX1T1+ Kasumi-1 cells). Low detection fractions are expected when fusion breakpoints lie distant from the 5' end, since nanoranger targets are optimally loci close to the 5' end of highly expressed genes with transcripts shorter than 4 kB.

## Related tools

- **nanoranger** (Long-read single-cell pipeline run in 5p10XGEX fusion mode to align reads to fusion junctions and extract cell barcode/UMI from chimeric reads) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream package that reads out fusion genes/CAR sequences from nanoranger amplicon output and collapses UMIs with starcode (Levenshtein distance 3)) — https://github.com/liviuspenter/nanoranger.R
- **minimap2** (Splice-aware long-read aligner for mapping noisy ONT cDNA reads against the reference) — https://github.com/lh3/minimap2
- **STAR** (Spliced aligner option for chimeric/spliced transcript alignment in the workflow) — https://github.com/alexdobin/STAR

## Examples

```
nanoranger --mode 5p10XGEX --fusion K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz; python fusion_gene.py --input fusion_calls.tsv
```

## Evaluation signals

- fusion_calls.tsv exists and contains BCR::ABL1 and RUNX1::RUNX1T1 entries with cell barcode and UMI fields
- Number of fusion-positive cells matches the reported benchmark: 124 (2.7%) K562 cells for BCR::ABL1 and 88 (3.9%) Kasumi-1 cells for RUNX1::RUNX1T1
- Only chimeric reads spanning either fusion breakpoint, with associated 10x cell barcode and UMI, are retained before deduplication
- UMI deduplication collapses sequencing-error UMIs within a Levenshtein distance of 3 (starcode), consistent with the 10x UMI having no whitelist
- Detected fraction of fusion-positive cells falls in the same range as the reported 2.7% (K562) and 3.9% (Kasumi-1) values

## Limitations

- Fusion breakpoints distant from the 5' end are detected inefficiently — only 2.7% of K562 (BCR::ABL1) and 3.9% of Kasumi-1 (RUNX1::RUNX1T1) cells were called positive
- The genotyping ceiling is determined by the low, incomplete representation of many transcripts and cannot be merely overcome through amplicon optimization or deeper sequencing
- ONT data analysis must handle sequencing-error-derived UMIs, which lack a whitelist for correction and require distance-based collapsing

## Evidence

- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [abstract] starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms, sequences of chimeric antigen and TCRs"
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [intro] short-read sequencing is unable to easily resolve structural transcriptomic variants such as gene fusions, transgenes or even isoforms: "short-read sequencing is unable to easily resolve structural transcriptomic variants such as gene fusions, transgenes or even isoforms that characterize malignant and immune cell subpopulations."
- [readme] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [readme] UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3: [evidence span withheld — anchor status: external_doc]
