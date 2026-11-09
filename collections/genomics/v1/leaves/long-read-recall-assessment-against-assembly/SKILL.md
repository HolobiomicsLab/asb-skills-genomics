---
name: long-read-recall-assessment-against-assembly
description: Use when evaluating how many k-mers and synthetic mate-pair read pairs
  from raw or Lordec-corrected long reads are recovered against an Illumina assembly,
  e.g. benchmarking alignment-free library extraction.
when_to_use_negative:
- The goal is de novo hybrid assembly of large genomes (e.g. NA12878, Arabidopsis)
  rather than benchmarking library-extraction recall against an existing assembly.
- The long reads already come from true long-range mate-pair technologies (10X Genomics,
  DOVETAIL genomics) where insert sizes and pair orientation vary, making recall against
  a synthetic-library ground truth ill-defined.
- The reference assembly is not from the same organism/sample as the long reads, so
  recovered k-mers and pairs are not meaningful recall.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: Fast-SG
  role: Alignment-free scaffolding-graph builder used with -k 15 to extract synthetic
    mate-pair libraries from raw and corrected long reads for recall assessment
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: K-mer counter whose binaries are placed in the Fast-SG directory and used
    for k-mer counting underlying k-mer recall computation
  repo: https://github.com/refresh-bio/KMC
- name: PERL
  role: Language in which the FAST-SG.pl driver script is written and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:857179ccfd8691b610be993e7c6dd6193ca9e5b13af803e3a5c1cdf13430dbb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - KMC
  - PERL
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Long-read recall assessment against assembly

## Summary

Quantifies how faithfully Fast-SG extracts synthetic mate-pair libraries from raw versus hybrid error-corrected ONT (R9.2) and PacBio Sequel long reads, by computing k-mer recall, synthetic mate-pair read recall, and outlier-pair percentage against an Illumina reference assembly (E. coli K12).

## When to use

Use when evaluating how many k-mers and synthetic mate-pair read pairs from raw or Lordec-corrected long reads are recovered against an Illumina assembly, e.g. benchmarking alignment-free library extraction.

## When NOT to use

- The goal is de novo hybrid assembly of large genomes (e.g. NA12878, Arabidopsis) rather than benchmarking library-extraction recall against an existing assembly.
- The long reads already come from true long-range mate-pair technologies (10X Genomics, DOVETAIL genomics) where insert sizes and pair orientation vary, making recall against a synthetic-library ground truth ill-defined.
- The reference assembly is not from the same organism/sample as the long reads, so recovered k-mers and pairs are not meaningful recall.

## Inputs

- Read configuration file in space-separated long-read format (#type libID Path(long read) Insert-sizes SAM), e.g. examples/ecoli-reads.txt
- Raw ONT R9.2 and PacBio Sequel E. coli K12 long-read FASTQ libraries (insert sizes 0.5–8 kb)
- Lordec hybrid error-corrected versions of the same long-read libraries
- Illumina E. coli K12 assembly contigs in FASTA (possibly gzipped), e.g. example/ecoli-illumina.fa.gz

## Outputs

- ecoli_k12_recall_metrics.tsv with k-mer recall, synthetic mate-pair read recall, and outlier percentage per technology/correction-state combination
- Synthetic mate-pair libraries extracted by Fast-SG from each long-read library

## How to apply

Author a space-separated read configuration file (#type libID Path(long read) Insert-sizes SAM) listing the raw and Lordec-corrected ONT R9.2 and PacBio Sequel E. coli K12 libraries with insert sizes spanning 0.5–8 kb. Run FAST-SG.pl with k-mer size k=15, the read configuration file (-l), the Illumina E. coli K12 assembly contigs in FASTA (-r), and an output prefix (-p) to build the alignment-free scaffolding graph and extract synthetic mate-pair libraries. Then compute k-mer recall and synthetic mate-pair read recall by counting how many k-mers and synthetic pairs from the long reads are recovered against the Illumina assembly, and calculate the percentage of outlier synthetic pairs per library. Tabulate the three metrics per technology (ONT R9.2 vs PacBio Sequel) and correction state (raw vs corrected) and write them to ecoli_k12_recall_metrics.tsv. The rationale is that hybrid error correction (Lordec) should raise recall, which the reference recall values (raw ONT 49.42%, raw PacBio 31.65%; corrected ONT 75.12%, corrected PacBio 65.02%) demonstrate.

## Related tools

- **Fast-SG** (Alignment-free scaffolding-graph builder used with -k 15 to extract synthetic mate-pair libraries from raw and corrected long reads for recall assessment) — https://github.com/adigenova/fast-sg
- **KMC** (K-mer counter whose binaries are placed in the Fast-SG directory and used for k-mer counting underlying k-mer recall computation) — https://github.com/refresh-bio/KMC
- **PERL** (Language in which the FAST-SG.pl driver script is written and executed)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- ecoli_k12_recall_metrics.tsv contains a metrics row for each of the four technology/correction-state combinations (ONT R9.2 vs PacBio Sequel; raw vs Lordec-corrected)
- All recall and outlier values parse as percentages within the range [0, 100]
- Corrected libraries show higher mate-pair recall than raw ones (reference: ONT 49.42%→75.12%, PacBio 31.65%→65.02%)
- The synthetic library insert sizes respect the constraint that insert size is restricted to the actual long-read size
- The FAST-SG.pl run completes with the mandatory arguments (-k, -l, -r, -p) and produces output under the given prefix

## Limitations

- Synthetic library insert size is restricted to the actual long-read size, capping the achievable insert span.
- Recall depends on hybrid error correction: raw ONT (49.42%) and raw PacBio (31.65%) recall is substantially lower than Lordec-corrected values (75.12% and 65.02%).
- Recall is measured against an Illumina assembly, so errors in the reference assembly propagate into the recall metric.

## Evidence

- [methods] Run Fast-SG via FAST-SG.pl with k-mer size, library file, contigs, and output prefix: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] Synthetic libraries from corrected or uncorrected long reads: "synthetic mate-pair libraries extracted from corrected or uncorrected long reads using the E. coli K12 dataset"
- [discussion] Insert size bounded by long-read length: "The insert size is restricted to the actual long-read size."
- [abstract] Fast-SG extracts linking information from raw or corrected reads: "can be used to extract linking information from either raw or error-corrected long reads"
- [readme] Fast-SG command-line interface: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
