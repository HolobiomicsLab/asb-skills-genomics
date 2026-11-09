---
name: single-cell-cdna-library-pipeline-execution
description: Use when input FASTQ/BAM reads come from 10x Genomics whole-transcriptome
  cDNA libraries or targeted amplicons sequenced on ONT or Pacific Biosciences devices
  and need nanoranger processing, including multimer deconcatenation.
when_to_use_negative:
- Input is short-read (e.g., Illumina) single-cell data — the workflow is designed
  for long-read ONT/PacBio reads, as short reads cannot resolve structural transcriptomic
  variants like fusions and isoforms
- Input is bulk (non-single-cell) sequencing data without 10x cell barcodes/UMIs —
  nanoranger starts from intermediate single-cell cDNA libraries
- Target loci are far from the 5' end of poorly expressed genes or transcripts longer
  than ~4 kB — the paper defines optimal nanoranger targets as loci close to the 5'
  end of highly expressed genes with transcripts shorter than 4 kB
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: Primary pipeline that starts from intermediate single-cell cDNA libraries
    and performs multimer-read deconcatenation and long-read single-cell processing
    (e.g., 5p10XGEX mode)
  repo: github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R package for downstream processing of nanoranger output — extracts SNVs,
    indels, fusion genes/CAR sequences and isoforms; collapses UMIs with starcode
    within Levenshtein distance 3
  repo: liviuspenter/nanoranger.R
- name: minimap2
  role: Long-read aligner used for mapping long noisy reads (e.g., with PacBio presets
    such as map-pb/splice:hq for MAS-seq/Iso-seq data)
  repo: lh3/minimap2
- name: Pacific Biosciences
  role: Sequencing platform producing the long reads and MAS-seq multimer arrays consumed
    by the pipeline
- name: 10x Genomics
  role: Source chemistry providing 5'-anchored whole-transcriptome scRNA-seq cDNA
    libraries that are the pipeline's input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:900b2349b604edf6d048312befd47edb086d2a0d1490ea2ea9eb649fb6cefda1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - minimap2
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-cdna-library-pipeline-execution

## Summary

Execute the nanoranger long-read single-cell pipeline on 5'-anchored 10x Genomics scRNA-seq cDNA libraries, including deconcatenation of multimer reads (e.g., PacBio 15-mer MAS-seq arrays) into subreads before alignment and barcode/UMI-aware processing.

## When to use

Use when input FASTQ/BAM reads come from 10x Genomics whole-transcriptome cDNA libraries or targeted amplicons sequenced on ONT or Pacific Biosciences devices and need nanoranger processing, including multimer deconcatenation.

## When NOT to use

- Input is short-read (e.g., Illumina) single-cell data — the workflow is designed for long-read ONT/PacBio reads, as short reads cannot resolve structural transcriptomic variants like fusions and isoforms
- Input is bulk (non-single-cell) sequencing data without 10x cell barcodes/UMIs — nanoranger starts from intermediate single-cell cDNA libraries
- Target loci are far from the 5' end of poorly expressed genes or transcripts longer than ~4 kB — the paper defines optimal nanoranger targets as loci close to the 5' end of highly expressed genes with transcripts shorter than 4 kB

## Inputs

- FASTQ/BAM reads from 10x Genomics whole-transcriptome cDNA libraries (5'-anchored) sequenced on ONT or Pacific Biosciences devices
- PacBio MAS-seq multimer arrays (e.g., 15-mer MAS-seq / MAS-ISO-seq reads such as 1019_mtDNA.fastq.gz)
- Targeted amplicon sequencing reads from single-cell cDNA libraries
- Reference sequences (e.g., mitochondrial transcript and chromosome FASTA)

## Outputs

- Deconcatenated subreads per multimer input read (with deconcatenation log confirming multiple subreads per read)
- Aligned reads against the chosen reference (e.g., mitochondrial references)
- subread_deconcatenation_stats.tsv summarizing deconcatenated subreads, mapped reads, and max transcripts per read
- nanoranger output suitable for downstream nanoranger.R feature extraction (SNVs, indels, fusions, CAR/TCR, isoforms)

## How to apply

Obtain input data as 10x Genomics whole-transcriptome cDNA libraries or targeted amplicon sequencing output from ONT or PacBio devices (e.g., the repository sample 1019_mtDNA.fastq.gz of PacBio 15-mer MAS-seq arrays). Run nanoranger in 5p10XGEX mode against the relevant references (e.g., mitochondrial transcript/chromosome sequences only), which enables multimer-read deconcatenation into subreads before mapping. Check the deconcatenation log to confirm multiple subreads are extracted per multimer read, using the paper's 15-mer artificial MAS-ISO-seq benchmark (10.7M deconcatenated vs 8.5M mapped reads, compared against longbow) as the reference ratio. Collect deconcatenated subread counts, mapped reads, and maximum transcripts recovered per read (the paper reports up to 478 transcripts from a single read across 13 amplicon libraries) into a stats table such as subread_deconcatenation_stats.tsv, then continue with downstream nanoranger.R for variant, fusion, CAR/TCR, and isoform extraction.

## Related tools

- **nanoranger** (Primary pipeline that starts from intermediate single-cell cDNA libraries and performs multimer-read deconcatenation and long-read single-cell processing (e.g., 5p10XGEX mode)) — github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing of nanoranger output — extracts SNVs, indels, fusion genes/CAR sequences and isoforms; collapses UMIs with starcode within Levenshtein distance 3) — liviuspenter/nanoranger.R
- **minimap2** (Long-read aligner used for mapping long noisy reads (e.g., with PacBio presets such as map-pb/splice:hq for MAS-seq/Iso-seq data)) — lh3/minimap2
- **Pacific Biosciences** (Sequencing platform producing the long reads and MAS-seq multimer arrays consumed by the pipeline)
- **10x Genomics** (Source chemistry providing 5'-anchored whole-transcriptome scRNA-seq cDNA libraries that are the pipeline's input)

## Examples

```
nanoranger --mode 5p10XGEX --fastq 1019_mtDNA.fastq.gz --reference mt_references.fa
```

## Evaluation signals

- Deconcatenation recovers substantially more reads than the mapped-read count, consistent with the 15-mer benchmark ratio of 10.7M deconcatenated vs 8.5M mapped reads
- Deconcatenation log confirms multiple subreads extracted per multimer read
- Coverage distributions of nanoranger-deconcatenated reads are similar to the longbow comparison on artificially generated 15-mer MAS-ISO-seq arrays
- subread_deconcatenation_stats.tsv is populated with deconcatenated subread counts, mapped reads, and max transcripts per read (paper reference: up to 478 transcripts from a single read across 13 amplicon libraries)
- Aligned output carries 10x cell barcode and UMI information suitable for downstream nanoranger.R feature extraction

## Limitations

- Genotyping ceiling is determined by low, incomplete representation of many transcripts and cannot merely be overcome through amplicon optimization or deeper sequencing
- Optimal targets are restricted to loci close to the 5' end of highly expressed genes with transcripts shorter than 4 kB
- Current long-read flow cells generating 5–50 million reads constrain the size of detectable primer panels
- Co-sequencing of DNA and RNA needed to further overcome genotyping limitations is still under development

## Evidence

- [abstract] nanoranger starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [methods] Input data source for the pipeline: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] Devices and chemistry the workflow accepts: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [abstract] Optimal target definition for nanoranger: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [discussion] Long-read flow cell read budget for panel sizing: "Current long-read flow cells generating 5–50 million reads enable primer panels designed to detect multiple informative isoforms"
