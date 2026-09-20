---
name: genome-coverage-estimation-from-seed-sets
description: Use when selecting or validating sparse short seed fragments (e.g. 270
  bp or 285 bp FASTA seeds) for a long-read scaffolding pipeline and you need to report
  or target a specific genome coverage.
when_to_use_negative:
- Seed sequences within a file vary in length or use non-numeric/duplicated identifiers,
  violating TULIP's strict requirements
- No independent genome size estimate is available, so coverage cannot be computed
  (or only by inventing a genome size)
- The goal is read coverage of raw sequencing data rather than coverage represented
  by a sparse, deliberately non-repetitive seed set
edam_operation: http://edamontology.org/operation_3782
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Jellyfish
  role: Counts 25-mers in merged fragments to identify and remove repetitive seeds
    before coverage computation
- name: FLASh
  role: Merges paired-end Illumina reads with a minimum overlap of 15 nt to produce
    candidate seed fragments
- name: Sickle
  role: Quality-trims Illumina reads prior to merging and seed selection
  repo: https://github.com/najoshi/sickle
- name: TULIP
  role: Consumes the seed FASTA files; imposes the requirements that identifiers be
    unique numbers (excluding 0) and seeds be exactly the same length
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:0a6b145d5c27dfb159d7fb67a75cda82fa7234dbd79ef9ff7b5e0bc1cb0a1265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - FLASh
  - Jellyfish
  - Sickle
  - TULIP
  edam_operation: http://edamontology.org/operation_3782
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# genome-coverage-estimation-from-seed-sets

## Summary

Estimates the effective genome coverage of a sparse seed set for TULIP assembly by dividing the number of seed fragments of a fixed length by the k-mer-estimated genome size, as done for the European eel where 5,019,778 × 270 bp seeds gave 1.58x and 873,058 × 285 bp seeds gave 0.29x coverage of the ~860 Mbp genome.

## When to use

Use when selecting or validating sparse short seed fragments (e.g. 270 bp or 285 bp FASTA seeds) for a long-read scaffolding pipeline and you need to report or target a specific genome coverage.

## When NOT to use

- Seed sequences within a file vary in length or use non-numeric/duplicated identifiers, violating TULIP's strict requirements
- No independent genome size estimate is available, so coverage cannot be computed (or only by inventing a genome size)
- The goal is read coverage of raw sequencing data rather than coverage represented by a sparse, deliberately non-repetitive seed set

## Inputs

- Seed FASTA file with unique numeric identifiers (excluding 0) and uniform sequence length, e.g. eel_seeds_285.fasta
- k-mer based genome size estimate (e.g. ~860 Mbp for the European eel from k-mer analysis)
- Merged paired-end Illumina fragments for seed derivation (FLASh output, min overlap 15 nt)
- Jellyfish 25-mer counts used to filter overly repetitive fragments

## Outputs

- Coverage value per seed set (e.g. 1.58x for 270 bp seeds, 0.29x for 285 bp seeds)
- Validated seed FASTA counts (5,019,778 sequences of 270 bp; 873,058 sequences of 285 bp)

## How to apply

First estimate genome size independently, e.g. by k-mer analysis (the eel genome was estimated at ~860 Mbp). Then count how many seed fragments of a single fixed length (all seeds in a file must be exactly the same length, e.g. 270 bp or 285 bp) are present in the seed FASTA. Compute coverage as (number of seeds × seed length) / genome size; for the eel, 5,019,778 seeds of 270 bp correspond to 1.58-fold coverage and 873,058 seeds of 285 bp to 0.29-fold coverage. Seeds themselves are derived by merging paired-end Illumina reads with FLASh (minimum overlap 15 nt), quality trimming with Sickle, counting 25-mers with Jellyfish, and discarding fragments containing any 25-mer occurring more than 25 times to satisfy the 'not too repetitive' criterion. Validate that sequence identifiers are unique numbers excluding 0 and that all sequences within a seed file share identical length, since the TULIP seed step (tulipseed.perl --seedlength) depends on these invariants.

## Related tools

- **Jellyfish** (Counts 25-mers in merged fragments to identify and remove repetitive seeds before coverage computation)
- **FLASh** (Merges paired-end Illumina reads with a minimum overlap of 15 nt to produce candidate seed fragments)
- **Sickle** (Quality-trims Illumina reads prior to merging and seed selection) — https://github.com/najoshi/sickle
- **TULIP** (Consumes the seed FASTA files; imposes the requirements that identifiers be unique numbers (excluding 0) and seeds be exactly the same length) — https://github.com/Generade-nl/TULIP

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Seed file contains exactly the expected number of sequences (e.g. 5,019,778 of 270 bp; 873,058 of 285 bp)
- All sequence identifiers are unique numbers excluding 0
- All sequences within one seed file have exactly the same length
- Computed coverage matches the expected ratio of (seed count × seed length) / k-mer-estimated genome size (1.58x and 0.29x for the ~860 Mbp eel genome)
- Seeds pass the 'not too repetitive' filter: no fragment contains a 25-mer occurring more than 25 times

## Limitations

- Large repetitive regions and (sub)telomeric repeats may not contain frequent islands of unique seed-length sequence and could therefore be absent from the assembly
- Scaffolds that could be merged based on long reads remain separate if the reads do not share a fortuitous seed alignment in the correct position
- Coverage depends on the accuracy of the k-mer-based genome size estimate; the eel estimate was ~860 Mbp

## Evidence

- [results] Seed counts vs coverage: [evidence span withheld — anchor status: not_found]
- [readme] Strict TULIP seed requirements: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [methods] Not too repetitive criterion: "These are pre-selected seed sequenced, with selection criteria 'not too repetitive'."
- [discussion] Genome size from k-mer analysis: "the genome size predicted from k-mer analysis (~860 Mbp)"
- [discussion] Repetitive regions absent: "large repetitive regions in the genome, as well as (sub) telomeric repeats will not always contain frequent 285 bp islands of unique sequence, and hence could be absent from the assembly"
