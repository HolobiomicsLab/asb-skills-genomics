---
name: consensus-error-quantification
description: Use when basecalled FASTQ read sets from nanopore simulators must be
  compared against real experimental reads by de novo bacterial assembly, polishing,
  and QUAST consensus metrics.
when_to_use_negative:
- The goal is short-read or Illumina assembly benchmarking rather than ONT nanopore
  consensus accuracy
- Input reads are not basecalled FASTQ from a nanopore signal simulator (e.g., raw
  POD5 signal without basecalling)
- The reference genome is a complex eukaryote rather than a bacterial genome; the
  paper's validation criterion (sub-1-mismatch-per-100-kbp, 99.98%–100.00% genome
  fraction) was established only on the five bacterial species
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Generates the simulated nanopore signals whose Dorado-basecalled FASTQ reads
    are benchmarked against real experimental reads
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated reads into FASTQ via the NanoSimFormer --basecall
    option before assembly
  repo: github.com/nanoporetech/dorado
- name: nanoMDBG
  role: Performs de novo assembly of each simulator's basecalled read set against
    the bacterial genome target
- name: Medaka
  role: Polishes each nanoMDBG assembly to correct residual consensus errors
  repo: https://github.com/nanoporetech/medaka
- name: QUAST
  role: Evaluates polished assemblies against the ground-truth reference, reporting
    mismatches per 100 kbp and genome fraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:1a6f282f58e013b08a54450e35e0220d17bb94b6aa68efb399d97cd8224e3a98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Dorado
  - Medaka
  - NanoSimFormer
  - QUAST
  - nanoMDBG
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# consensus-error-quantification

## Summary

Benchmarks de novo bacterial assembly quality from simulated versus real nanopore reads by running nanoMDBG assembly, Medaka polishing, and QUAST evaluation, extracting mismatches per 100 kbp and genome fraction to test whether a simulator's basecalled reads reproduce experimental consensus accuracy.

## When to use

Use when basecalled FASTQ read sets from nanopore simulators must be compared against real experimental reads by de novo bacterial assembly, polishing, and QUAST consensus metrics.

## When NOT to use

- The goal is short-read or Illumina assembly benchmarking rather than ONT nanopore consensus accuracy
- Input reads are not basecalled FASTQ from a nanopore signal simulator (e.g., raw POD5 signal without basecalling)
- The reference genome is a complex eukaryote rather than a bacterial genome; the paper's validation criterion (sub-1-mismatch-per-100-kbp, 99.98%–100.00% genome fraction) was established only on the five bacterial species

## Inputs

- Basecalled FASTQ read sets from NanoSimFormer and competitor simulators (reference-based simulation of E. coli, K. pneumoniae, M. morganii, P. mirabilis, P. mirabilis excluded—five bacterial genomes)
- Ground-truth bacterial reference genome FASTA files for each of the five species

## Outputs

- Polished nanoMDBG+Medaka assemblies per simulator per genome (FASTA)
- quast_report.tsv with per-genome, per-simulator mismatches per 100 kbp and genome fraction

## How to apply

Load the Dorado-basecalled FASTQ read sets produced by NanoSimFormer (and each competitor simulator) from reference-based simulation of the five bacterial genomes (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis). Run nanoMDBG de novo assembly on each read set using the corresponding bacterial genome as the assembly target, then polish each assembly with Medaka to correct residual consensus errors. Evaluate every polished assembly with QUAST against its ground-truth reference genome, extracting mismatches per 100 kbp and genome fraction, and compile the per-genome, per-simulator metrics into quast_report.tsv. The success criterion mirrors the paper: NanoSimFormer consensus error below one mismatch per 100 kbp with genome fraction 99.98%–100.00%, whereas competitors fall short (e.g., Squigulator's 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis).

## Related tools

- **NanoSimFormer** (Generates the simulated nanopore signals whose Dorado-basecalled FASTQ reads are benchmarked against real experimental reads) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated reads into FASTQ via the NanoSimFormer --basecall option before assembly) — github.com/nanoporetech/dorado
- **nanoMDBG** (Performs de novo assembly of each simulator's basecalled read set against the bacterial genome target)
- **Medaka** (Polishes each nanoMDBG assembly to correct residual consensus errors) — https://github.com/nanoporetech/medaka
- **QUAST** (Evaluates polished assemblies against the ground-truth reference, reporting mismatches per 100 kbp and genome fraction)

## Examples

```
medaka_consensus -i basecalls.fastq -d draft_assm/assm_final.fa -o medaka_consensus -t $(nproc)
```

## Evaluation signals

- NanoSimFormer-derived assemblies show fewer than one mismatch or indel per 100 kbp across all five evaluated bacterial genomes
- NanoSimFormer genome fraction falls in the 99.98%–100.00% range on each genome
- Competitor simulators show quantified shortfalls (e.g., Squigulator: 1,067 mismatches per 100 kbp on E. coli; 94.8% genome fraction on P. mirabilis), confirming the report discriminates between simulators
- quast_report.tsv contains a complete per-genome, per-simulator grid with no missing QUAST entries

## Limitations

- The benchmark is validated on five bacterial genomes; performance on eukaryotic or metagenomic assemblies is not covered by the paper's criterion
- Consensus quality depends on Dorado basecalling of simulated signals; simulator bias in homopolymer and short tandem repeat regions can inflate assembly errors
- Comparisons require identically configured simulation and polishing pipelines across all simulators to be a fair benchmark

## Evidence

- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp"
- [methods] workflow assembly and QUAST: [evidence span withheld — anchor status: not_found]
- [methods] Polish with Medaka: [evidence span withheld — anchor status: not_found]
- [methods] QUAST criteria: [evidence span withheld — anchor status: not_found]
- [readme] medaka usage: [evidence span withheld — anchor status: external_doc]
