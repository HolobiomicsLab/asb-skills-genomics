---
name: alignment-configuration-file-preparation
description: Use when running TULIP with multiple nanopore alignment files (e.g. R7.3,
  R9 1D/2D, R9.4 1D against eel_seeds_285.fasta) and a config file mapping each short
  SAM to its original FASTA is required.
when_to_use_negative:
- Input alignments are not in SAM format (earlier TULIP versions supported e.g. DALIGNER
  output, but current TULIP 0.4 accepts SAM only)
- Read identifiers are not unique per FASTA file because data were naively concatenated
  — the config-file approach exists precisely to avoid concatenation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: TULIP
  role: Consumes the configuration file via --config in tulipseed.perl and tulipbulb.perl
    to link each alignment set to its original read FASTA
  repo: github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Generates the SAM alignment files referenced by the config, using chemistry-specific
    -k/-W seed-length settings with common -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 scoring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:1daa1eadd1bf6ef81cb9b62fde63bdd7f9b1389a1436be94b663bc11ae1e1310
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  - tulipbulb.perl
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# alignment-configuration-file-preparation

## Summary

Prepare a TULIP configuration file (alignments.txt) that tab-delimits each SAM alignment file with its format token ('sam') and the original long-read FASTA, after generating chemistry-specific BWA MEM alignments of nanopore reads against 285 bp seed sequences. TULIP reads this file to associate each seed-alignment set with the corresponding raw reads during graph construction.

## When to use

Use when running TULIP with multiple nanopore alignment files (e.g. R7.3, R9 1D/2D, R9.4 1D against eel_seeds_285.fasta) and a config file mapping each short SAM to its original FASTA is required.

## When NOT to use

- Input alignments are not in SAM format (earlier TULIP versions supported e.g. DALIGNER output, but current TULIP 0.4 accepts SAM only)
- Read identifiers are not unique per FASTA file because data were naively concatenated — the config-file approach exists precisely to avoid concatenation

## Inputs

- eel_seeds_285.fasta (873058 pre-selected 285 bp seed sequences with unique numeric identifiers)
- Nanopore read FASTA files: R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta
- Short SAM alignment files (first six SAM fields only), e.g. R73_vs_285.shortsam

## Outputs

- alignments.txt: tab-delimited configuration file with lines of the form 'sam [tab] alignment [tab] original fasta'

## How to apply

First index the seed FASTA with 'bwa index -p 285_seeds eel_seeds_285.fasta' and align each nanopore read file with chemistry-specific bwa mem settings (R7.3: -k 14 -W 45; R9 1D: -k 16 -W 50; R9 2D: -k 19 -W 60; R9.4 1D: -k 16 -W 60; all with -t 4 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), piping through 'cut -f 1,2,3,4,5,6' because TULIP uses only the first six SAM fields up to the CIGAR string. Then write each line of alignments.txt as 'sam<TAB>alignment_file<TAB>original_fasta', listing every alignment file; TULIP consumes this config in both tulipseed.perl (--config alignments.txt) and tulipbulb.perl. The rationale is that the config lets a single TULIP run integrate reads from three generations of nanopore chemistry in one seed graph without concatenating FASTA files, which would break unique read identifiers.

## Related tools

- **TULIP** (Consumes the configuration file via --config in tulipseed.perl and tulipbulb.perl to link each alignment set to its original read FASTA) — github.com/Generade-nl/TULIP
- **BWA MEM** (Generates the SAM alignment files referenced by the config, using chemistry-specific -k/-W seed-length settings with common -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 scoring)

## Examples

```
bwa mem -t 4 -k 16 -W 60 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9.4_pass_1d.fasta | cut -f 1,2,3,4,5,6 > R941D_vs_285.shortsam
```

## Evaluation signals

- Every line of alignments.txt contains exactly three tab-separated fields: the literal format token 'sam', an existing .shortsam path, and an existing FASTA path
- Each SAM file contains only the first six fields (flag through CIGAR) since TULIP ignores the remainder
- The number of config lines equals the number of nanopore read files (four in the eel workflow) and each referenced FASTA identifier set is unique
- tulipseed.perl runs to completion with --config alignments.txt and produces tulip/eel.graph, tulip/eel.scaffolds, and tulip/eel.scaffolds_stats without 'file not found' log errors

## Limitations

- The FASTA paths in the config must be the original read files; concatenating read files breaks unique identifiers, so careful bookkeeping is required across chemistries
- Only SAM format is supported in this TULIP prototype; DALIGNER-style support existed in earlier versions and might reappear
- TULIP 0.4 is a prototype: tulipbulb.perl has a known bug that can emit gaps (NNNs) when sequence cannot be added

## Evidence

- [readme] When using multiple alignment files, you should specify their locations in a short configuration file: "When using multiple alignment files, you should specify their locations in a short configuration file"
- [readme] Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string): "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [readme] We used BWA MEM to align reads to seeds, but other aligners should work.: "We used BWA MEM to align reads to seeds, but other aligners should work. Currently, TULIP accepts SAM format"
- [methods] bwa mem chemistry-specific parameters for R9.4 1D reads: "bwa mem -t 4 -k 16 -W 60 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9.4_pass_1d.fasta"
