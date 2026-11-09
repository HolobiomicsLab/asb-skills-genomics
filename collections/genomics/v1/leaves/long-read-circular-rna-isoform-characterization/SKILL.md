---
name: long-read-circular-rna-isoform-characterization
description: Use when nanopore long-read sequencing of RCA-amplified circRNA products
  (FASTA/FASTQ) must be converted into a catalog of high-confidence full-length circRNA
  isoforms and back-splice junctions.
when_to_use_negative:
- Input is standard short-read RNA-seq without RCA — short-read RNA-seq does not experimentally
  determine the full-length sequences and exact exonic compositions of circRNAs
- Existing short-read circRNA reconstruction is sufficient for short circRNAs (200–500
  nt) — computational reconstruction methods on short reads are only applicable to
  short circRNAs and cannot interrogate longer full-length circRNAs
- Input long reads were not generated from rolling circle amplification of circular
  molecules — the consensus-calling (TRF tandem-repeat) logic assumes tandem RCA copies
  per read
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: isoCirc
  role: Integrated computational pipeline that performs TRF consensus calling, minimap2
    filtering/mapping, and high-confidence BSJ/full-length circRNA identification
    from RCA long-read data
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Splice-aware mapping and filtering of TRF consensus sequences with minimum
    consensus length 30 and minimum copy number 2.0
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Required open-source dependency of the isoCirc pipeline for genomic interval
    operations (>= v2.27.0)
  repo: https://bedtools.readthedocs.io/
- name: pip
  role: Package manager used to install the Python-based isocirc pipeline (pip install
    isocirc)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:d7069755d341c34c205eee08f342d5907882616be8f0064a56d3b3fcf83624c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  - pip
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# long-read-circular-rna-isoform-characterization

## Summary

Characterizes full-length circRNA isoforms by combining rolling circle amplification (RCA) with nanopore long-read sequencing and the isoCirc computational pipeline, which calls consensus sequences and applies stringent BSJ filters to produce a catalog of full-length circRNA isoforms with exact exonic compositions.

## When to use

Use when nanopore long-read sequencing of RCA-amplified circRNA products (FASTA/FASTQ) must be converted into a catalog of high-confidence full-length circRNA isoforms and back-splice junctions.

## When NOT to use

- Input is standard short-read RNA-seq without RCA — short-read RNA-seq does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs
- Existing short-read circRNA reconstruction is sufficient for short circRNAs (200–500 nt) — computational reconstruction methods on short reads are only applicable to short circRNAs and cannot interrogate longer full-length circRNAs
- Input long reads were not generated from rolling circle amplification of circular molecules — the consensus-calling (TRF tandem-repeat) logic assumes tandem RCA copies per read

## Inputs

- Long-read nanopore FASTA/FASTQ files of RCA-amplified circRNA products
- Reference genome FASTA
- Gene annotation GTF
- circRNA annotation BED/GTF
- Optional short-read FASTA/FASTQ for hybrid error correction with LoRDEC

## Outputs

- isocirc.out: 35-column table of BSJs and full-length circRNA isoforms with readCount column
- isocirc.bed: genomic coordinates of identified circRNAs/BSJs
- isocirc_stats.out: per-run statistics
- Per-sample catalog of full-length circRNA isoform counts (e.g., readCount >= 1)

## How to apply

Install isoCirc via pip (it is written in Python) and ensure bedtools (>= v2.27.0) and minimap2 (>= 2.11) are present. For each sample, run the isocirc pipeline supplying the long-read FASTA/FASTQ, reference genome FASTA, gene annotation GTF, and circRNA annotation BED/GTF. The pipeline performs consensus calling on tandem-repeat RCA reads with Tandem Repeats Finder (defaults: match 2, mismatch 7, indel 7, max period 2000, min score 100), then filters and maps consensus sequences with minimap2 (minimum consensus length 30, minimum copy number 2.0). Identify high-confidence BSJs and full-length circRNAs using canonical GT/AG motif filtering, allowing at most 1 mismatch/indel/deletion in the 20-bp exonic sequence flanking the BSJ (0 for the 4-bp core) and a minimum genomic distance of 150 between back-splice sites; apply stricter criteria for BSJs involving novel splice sites. Aggregate the 35-column isocirc.out tables (using the readCount column at read >= 1) across samples, and optionally normalize BSJ numbers by sequencing depth to correlate against an independent short-read BSJ dataset.

## Related tools

- **isoCirc** (Integrated computational pipeline that performs TRF consensus calling, minimap2 filtering/mapping, and high-confidence BSJ/full-length circRNA identification from RCA long-read data) — https://github.com/Xinglab/isoCirc
- **minimap2** (Splice-aware mapping and filtering of TRF consensus sequences with minimum consensus length 30 and minimum copy number 2.0) — https://github.com/lh3/minimap2
- **bedtools** (Required open-source dependency of the isoCirc pipeline for genomic interval operations (>= v2.27.0)) — https://bedtools.readthedocs.io/
- **pip** (Package manager used to install the Python-based isocirc pipeline (pip install isocirc))

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Run the toy example in test_data (e.g., isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output) and confirm isocirc.out, isocirc.bed, and isocirc_stats.out are produced
- Per-tissue full-length isoform counts at readCount >= 1 reproduce published values (testis 115,348; prostate 18,312; skeletal muscle 19,261)
- Read-depth-normalized BSJ numbers correlate with independent short-read RNA-seq BSJ numbers across shared tissues with Pearson r ≈ 0.86
- All reported BSJs pass canonical GT/AG motif filtering with ≤1 mismatch/indel in the 20-bp flanking exonic sequence (0 in the 4-bp core) and genomic back-splice distance ≥ 150
- isoCirc libraries show enrichment of circRNA BSJ reads relative to RNase R-treated short-read libraries (reported as over 70-fold)

## Limitations

- At current sequencing depth isoCirc is far from saturation; more circRNA isoforms are expected with deeper sequencing
- Relatively low throughput and high per-sequence cost of long-read RNA-seq make deep transcriptome sampling prohibitively expensive
- Filtering is much stricter for BSJs involving novel splice sites than for known splice sites, so novel-splice-site BSJs may be under-called
- In this work isoCirc was applied only to whole cells and tissues; subcellular-fraction RNAs (e.g., chromatin-associated) were not analyzed

## Evidence

- [abstract] we report isoCirc, a strategy for sequencing full-length circRNA isoforms, using rolling circle amplification followed by nanopore long-read sequencing: "we report isoCirc, a strategy for sequencing full-length circRNA isoforms, using rolling circle amplification followed by nanopore long-read sequencing"
- [abstract] catalog of 107,147 full-length circRNA isoforms: "we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293), including 40,628 isoforms ≥500 nt in length"
- [methods] isoCirc dependencies: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [discussion] stringent filtering rationale: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [readme] RCA + long-read sequencing workflow: "computational pipeline to characterize full-length circRNA isoforms using rolling circle amplification (RCA) followed by long-read sequencing"
- [discussion] BSJ read enrichment: "an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries"
