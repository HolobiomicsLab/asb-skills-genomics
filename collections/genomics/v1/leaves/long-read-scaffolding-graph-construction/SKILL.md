---
name: long-read-scaffolding-graph-construction
description: Use when you have long nanopore reads (mixed ONT chemistries) plus many
  short unique seed sequences (all exactly the same length, numeric unique IDs excluding
  0), and need a de novo genome scaffold graph without expensive whole-read assembly.
when_to_use_negative:
- Input reads have no suitably sparse, non-repetitive short seeds — repetitive seeds
  will make it take much longer to untangle the graph
- You need a polished reference-grade assembly; TULIP output is a candidate assembly
  that still needs Racon/Pilon correction and planned optical-mapping validation
- You require merging of scaffolds that lack a fortuitous seed alignment in the correct
  position — such scaffolds remain separate
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BWA MEM
  role: Aligns each nanopore read set against the 285 bp seed index with chemistry-specific
    -k/-W parameters, outputting 6-field SAM ('shortsam') files
- name: TULIP (tulipseed.perl)
  role: Consumes the seed-alignment config to build the seed graph and ordered scaffold
    structures (--seedlength 285 --diploid)
  repo: github.com/Generade-nl/TULIP
- name: tulipbulb.perl
  role: Adds seed and long-read sequence to the tulipseed scaffolds, emitting per-scaffold
    FASTA and read bundles
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:375197ee0e7f4a9d09eddf2bfecb6b1b96b421a490f08abca5b7c77190963e32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  - TULIP (tulipseed.perl)
  - tulipbulb.perl
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# long-read-scaffolding-graph-construction

## Summary

Builds ordered seed-linkage scaffolds by aligning uncorrected nanopore long reads (R7.3, R9 1D/2D, R9.4 1D) against sparse 285 bp 'not too repetitive' seed sequences with BWA MEM, then running TULIP's tulipseed.perl to infer a seed graph and tulipbulb.perl to add seed/read sequence. This substitutes quadratic whole-genome long-read assembly with nearly linear scaffolding of manageable subsets.

## When to use

Use when you have long nanopore reads (mixed ONT chemistries) plus many short unique seed sequences (all exactly the same length, numeric unique IDs excluding 0), and need a de novo genome scaffold graph without expensive whole-read assembly.

## When NOT to use

- Input reads have no suitably sparse, non-repetitive short seeds — repetitive seeds will make it take much longer to untangle the graph
- You need a polished reference-grade assembly; TULIP output is a candidate assembly that still needs Racon/Pilon correction and planned optical-mapping validation
- You require merging of scaffolds that lack a fortuitous seed alignment in the correct position — such scaffolds remain separate

## Inputs

- eel_seeds_285.fasta — 873058 pre-selected 285 bp seed sequences, all exactly the same length, unique numeric identifiers excluding 0
- R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta — uncorrected nanopore long reads, one line per read in FASTA, unique identifiers per file
- alignments.txt — configuration file with 'format [tab] alignment [tab] original fasta' rows (e.g. 'sam R92D_vs_285.shortsam R9_pass_2d.fasta')

## Outputs

- tulip/eel.graph — simplified seed graph linking seeds with orientation (ii|io|oi|oo), evidence (number of long read alignments), min/mean/max gap, StDev, and scaffold ID
- tulip/eel.scaffolds and tulip/eel.scaffolds_stats — ordered seed scaffolds and per-scaffold length statistics
- tulip/eel.*_tmp binary files, seeds_tmp usage info, and layout_log consumed by tulipbulb.perl
- bulb/eel_scaffold_999.fasta — scaffold sequence (seeds uppercase, long-read derived sequence lowercase) and eel_readbundle_999.fasta per scaffold

## How to apply

Index the 285 bp seed FASTA with bwa index, then align each nanopore read file (R7.3, R9_pass_1d, R9_pass_2d, R9.4_pass_1d) with chemistry-specific bwa mem settings (-k 14 -W 45 for R7.3; -k 16 -W 50 for R9 1D; -k 19 -W 60 for R9 2D; -k 16 -W 60 for R9.4 1D, plus -t 4 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), piping through cut -f 1-6 since TULIP uses only the first six SAM fields. Write an alignments.txt config with lines of 'sam [tab] alignment file [tab] original fasta'. Run tulipseed.perl with --seedlength 285 --diploid to build tulip/eel.graph (seed links with orientation ii/io/oi/oo, evidence counts, min/mean/max gap and StDev) and ordered scaffolds, then tulipbulb.perl to attach seed and long-read sequence, producing bulb/eel_scaffold_999.fasta and eel_readbundle_999.fasta files.

## Related tools

- **BWA MEM** (Aligns each nanopore read set against the 285 bp seed index with chemistry-specific -k/-W parameters, outputting 6-field SAM ('shortsam') files)
- **TULIP (tulipseed.perl)** (Consumes the seed-alignment config to build the seed graph and ordered scaffold structures (--seedlength 285 --diploid)) — github.com/Generade-nl/TULIP
- **tulipbulb.perl** (Adds seed and long-read sequence to the tulipseed scaffolds, emitting per-scaffold FASTA and read bundles) — github.com/Generade-nl/TULIP

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Candidate assembly should contain ~2366 scaffolds spanning ~431 bp to 8.7 Mbp for the full 873058-seed eel run
- 232 scaffolds larger than 1 Mbp should make up ~56% of assembly length; N50 contiguity ~1.2 Mbp
- Mean evidence per seed link in tulip/eel.graph should be ~7.4 long-read alignments
- Graph 'Hypothetical' column must be 1 for actual links vs 0 for inferred links, with gap min/mean/max/StDev populated per link
- Whole pipeline (seed graph + bundling for the 860 Mbp genome) should complete in about two days on a single CPU

## Limitations

- tulipbulb.perl contains a bug that in rare cases prevents sequence addition and inserts NNN gaps instead
- Large repetitive regions and (sub)telomeric repeats may be absent if they lack frequent 285 bp islands of unique sequence
- Scaffolds that could be merged based on long reads remain separate without a fortuitous seed alignment; the result is a candidate assembly of uncertain correctness without a high-quality reference
- Being a diploid (--diploid) hybrid of two animals and three nanopore chemistries, the consensus could in theory mix up to four different haplotypes

## Evidence

- [readme] Tulipseed takes as input alignments files of long reads to sparse short seeds, and outputs a graph and scaffold structures.: "Tulipseed takes as input alignments files of long reads to sparse short seeds, and outputs a graph and scaffold structures."
- [readme] BWA MEM chemistry-specific alignment: "bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta         | cut -f 1,2,3,4,5,6 > R73_vs_285.shortsam"
- [readme] Only first six SAM fields are used: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string)"
- [intro] TULIP splits large assembly problems into subsets: "its primary purpose is to split up such large assembly problems into manageable subsets of long reads"
- [readme] tulipbulb adds sequence to scaffolds: "This will add sequence from the original seeds and reads to the graph output by `tulipseed.perl`."
