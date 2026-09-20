---
name: long-read-circular-rna-pipeline-execution
description: Use when processing isoCirc long-read FASTA/FASTQ data (RCA + nanopore)
  against a reference genome and GTF annotation to obtain per-isoform BSJ counts and
  full-length circRNA isoforms.
when_to_use_negative:
- Input is standard short-read Illumina RNA-seq without RCA/long-read library preparation
  — isoCirc requires long reads spanning full circRNA isoforms
- Goal is quantifying internal alternative splicing or full-length isoform sequences
  from short-read data, which the article states cannot experimentally determine full-length
  circRNA sequences
- The task is circRNA detection from direct total RNA-seq without rolling circle amplification
  template — use a short-read BSJ caller instead
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: isoCirc
  role: 'Main pipeline: consensus calling, filtering, mapping, and high-confidence
    BSJ/full-length circRNA identification'
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Filtering and mapping of consensus sequences (>= 2.11 required)
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Open-source dependency for annotation-overlap operations (>= v2.27.0 required)
- name: LoRDEC
  role: Hybrid error-correction of long reads with short-read data
- name: Tandem Repeats Finder (TRF)
  role: Consensus calling from concatenated RCA repeats
- name: Python
  role: Implementation language; installed via pip
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:353a6b974d51e0f852f45a2feb9a38b29c777c24d6872ad8c7baa58d7e5b960d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - LoRDEC
  - Python
  - Tandem Repeats Finder (TRF)
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-circular-rna-pipeline-execution

## Summary

Run the isoCirc computational pipeline to identify high-confidence back-splice junctions (BSJs) and full-length circRNA isoforms from rolling-circle-amplified nanopore long-read data, using bedtools (>= v2.27.0) and minimap2 (>= 2.11) as dependencies.

## When to use

Use when processing isoCirc long-read FASTA/FASTQ data (RCA + nanopore) against a reference genome and GTF annotation to obtain per-isoform BSJ counts and full-length circRNA isoforms.

## When NOT to use

- Input is standard short-read Illumina RNA-seq without RCA/long-read library preparation — isoCirc requires long reads spanning full circRNA isoforms
- Goal is quantifying internal alternative splicing or full-length isoform sequences from short-read data, which the article states cannot experimentally determine full-length circRNA sequences
- The task is circRNA detection from direct total RNA-seq without rolling circle amplification template — use a short-read BSJ caller instead

## Inputs

- Long-read sequencing FASTA/FASTQ generated with the isoCirc strategy (RCA + nanopore)
- Reference genome sequence FASTA (e.g. chr16_toy.fa)
- Gene annotation GTF (e.g. chr16_toy.gtf)
- Known circRNA BED or GTF (e.g. chr16_circRNA_toy.bed)
- Optional short-read FASTA/FASTQ for hybrid error-correction with LoRDEC

## Outputs

- isocirc.out — per-isoform BSJ read counts
- isocirc.bed — circRNA coordinates
- isocirc_stats.out — summary counts of BSJs and full-length circRNA isoforms

## How to apply

Install isoCirc via `pip install isocirc` (or from source with `git clone https://github.com/Xinglab/isoCirc.git`), ensuring bedtools (>= v2.27.0) and minimap2 (>= 2.11) are on the PATH since isoCirc depends on both. Invoke the CLI with five positional arguments — the long-read FASTA/FASTQ, reference genome FASTA, gene annotation GTF, circRNA BED/GTF, and an output directory — plus `-t` for threads; a toy dataset in `test_data` (read_toy.fa, chr16_toy.fa, chr16_toy.gtf, chr16_circRNA_toy.bed) validates the install. The pipeline internally performs hybrid error-correction with short reads using LoRDEC, consensus calling with Tandem Repeats Finder (TRF), filtering and mapping of consensus sequences with minimap2, then identifies high-confidence BSJs and full-length circRNAs using stringent criteria (stricter for novel-splice-site BSJs, which require near-perfect alignment to genomic sequence around the BSJ). Results are written to `isocirc.out` (per-isoform BSJ read counts), `isocirc.bed`, and `isocirc_stats.out` (summary counts); the `--rescue-low` flag can rescue low-mapping-quality reads with high-mapping-quality reads. The resulting per-isoform BSJ counts can then be used in downstream comparisons (e.g. against Illumina short-read RNase R or poly(A) libraries) via the reported 3.5-4.0% BSJ-read proportion and ~0.5 Spearman correlation at read count >= 2.

## Related tools

- **isoCirc** (Main pipeline: consensus calling, filtering, mapping, and high-confidence BSJ/full-length circRNA identification) — https://github.com/Xinglab/isoCirc
- **minimap2** (Filtering and mapping of consensus sequences (>= 2.11 required)) — https://github.com/lh3/minimap2
- **bedtools** (Open-source dependency for annotation-overlap operations (>= v2.27.0 required))
- **LoRDEC** (Hybrid error-correction of long reads with short-read data)
- **Tandem Repeats Finder (TRF)** (Consensus calling from concatenated RCA repeats)
- **Python** (Implementation language; installed via pip)

## Examples

```
pip install isocirc; isocirc -t 1 reads.fq ref.fa anno.gtf circRNAs.bed output_dir
```

## Evaluation signals

- Pipeline completes without errors on the toy example: `isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output` produces isocirc.out, isocirc.bed, and isocirc_stats.out
- isocirc_stats.out reports summary counts consistent with expectations (e.g. number of BSJs and full-length isoforms)
- Novel-splice-site BSJs show near-perfect alignment between the isoCirc consensus sequence and genomic sequence around the BSJ (stricter filter than known-splice-site BSJs)
- Downstream HEK293 comparison: isoCirc BSJ read proportion falls in the 3.5-4.0% range, over 70-fold enrichment vs RNase R-treated Illumina (~0.045-0.048%)
- BSJ quantitation vs short reads correlates with Spearman ~0.5 when restricted to BSJs at read count >= 2 in both platforms

## Limitations

- At current sequencing depth isoCirc is far from saturation; more circRNA isoforms will be identified with deeper sequencing
- Relatively low throughput and high per-sequence cost of long-read RNA-seq make deep transcriptome sampling prohibitively expensive
- Filtering criteria are much stricter for BSJs involving novel splice sites, so such events may be under-recovered relative to known splice-site BSJs
- Short circRNAs and low-copy consensus reads may fail internal thresholds (e.g. min-copy, min-frac parameters governing consensus calling)

## Evidence

- [readme] isoCirc is dependent on two open-source software packages: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [readme] isocirc CLI invocation: "isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output"
- [readme] output files: [evidence span withheld — anchor status: not_found]
- [discussion] stringent filtering: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [discussion] novel splice site strictness: "requiring near-perfect alignment between the isoCirc consensus sequence and the corresponding genomic sequence around the BSJ"
- [discussion] 70-fold enrichment: "an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries"
- [methods] pipeline steps: "Hybrid error-correction with short-read data (LoRDEC):"
- [readme] rescue flag: "--rescue-low          Use high-mapping quality reads to rescue low-mapping quality reads"
