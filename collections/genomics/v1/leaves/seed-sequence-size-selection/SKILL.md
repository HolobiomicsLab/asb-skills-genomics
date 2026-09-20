---
name: seed-sequence-size-selection
description: 'Use when preparing input seed FASTA files for TULIP: paired Illumina
  reads must be converted into short, uniformly sized fragments that are unique enough
  not to tangle the seed-layout graph.'
when_to_use_negative:
- Input is nanopore long reads rather than merged short Illumina fragments — seeds
  must be short genomic islands, not long reads
- You need seeds containing repetitive sequence — this is allowed by TULIP but makes
  the graph much slower to untangle and optimize, so this skill's strict filtering
  is counterproductive only if you accept the runtime cost
- The downstream assembler does not require uniform-length, numerically identified
  seeds (these constraints are TULIP-specific)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: FLASh
  role: Merges paired-end Illumina reads into single fragments with a minimum 15 nt
    mate overlap, generating the fragment pool from which seeds are selected
- name: Jellyfish
  role: Counts 25-mers in merged fragments so fragments carrying 25-mers occurring
    more than 25 times can be discarded as repetitive
- name: Sickle
  role: Windowed adaptive quality trimming of FASTQ reads prior to merging and k-mer
    filtering
  repo: https://github.com/najoshi/sickle
- name: TULIP
  role: Downstream consumer of the seed FASTA; enforces the strict seed format (unique
    numeric identifiers excluding 0, exact uniform length)
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:2d876ca97d26f3eae3f9c60e219b6dca2f0396721096c6a969ebc25a5b518a6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - FLASh
  - Jellyfish
  - Sickle
  - TULIP
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# seed-sequence-size-selection

## Summary

Derives sparse, 'not too repetitive' short seed sequences (270 bp or 285 bp) from merged Illumina read pairs by k-mer occurrence filtering, for use as anchors in the TULIP long-read assembly of the ~860 Mbp European eel genome.

## When to use

Use when preparing input seed FASTA files for TULIP: paired Illumina reads must be converted into short, uniformly sized fragments that are unique enough not to tangle the seed-layout graph.

## When NOT to use

- Input is nanopore long reads rather than merged short Illumina fragments — seeds must be short genomic islands, not long reads
- You need seeds containing repetitive sequence — this is allowed by TULIP but makes the graph much slower to untangle and optimize, so this skill's strict filtering is counterproductive only if you accept the runtime cost
- The downstream assembler does not require uniform-length, numerically identified seeds (these constraints are TULIP-specific)

## Inputs

- Paired-end Illumina FASTQ reads (e.g. R1/R2 FASTQ pairs from an eel specimen)
- Estimated genome size for coverage calculation (~860 Mbp from k-mer analysis)

## Outputs

- eel_seeds_270.fasta — 5,019,778 fragments of exactly 270 bp (~1.58x coverage) with unique numeric IDs (no 0)
- eel_seeds_285.fasta — 873,058 fragments of exactly 285 bp (~0.29x coverage) with unique numeric IDs (no 0)

## How to apply

First merge paired-end Illumina reads with FLASh, requiring a minimum mate-pair overlap of 15 nt, optionally after quality trimming (e.g. Sickle). Count 25-mers across all merged fragments with Jellyfish, then discard any fragment containing a 25-mer that occurs more than 25 times — this operationalizes the 'not too repetitive' seed criterion and removes repeats that would otherwise make the TULIP graph slow to untangle. Size-select the surviving fragments to exact lengths (5,019,778 fragments of 270 bp ≈ 1.58x coverage; 873,058 fragments of 285 bp ≈ 0.29x coverage of the ~860 Mbp genome). Write each seed set to FASTA with unique numeric identifiers (excluding 0) and a single uniform sequence length per file, since these are strict TULIP parser requirements.

## Related tools

- **FLASh** (Merges paired-end Illumina reads into single fragments with a minimum 15 nt mate overlap, generating the fragment pool from which seeds are selected)
- **Jellyfish** (Counts 25-mers in merged fragments so fragments carrying 25-mers occurring more than 25 times can be discarded as repetitive)
- **Sickle** (Windowed adaptive quality trimming of FASTQ reads prior to merging and k-mer filtering) — https://github.com/najoshi/sickle
- **TULIP** (Downstream consumer of the seed FASTA; enforces the strict seed format (unique numeric identifiers excluding 0, exact uniform length)) — https://github.com/Generade-nl/TULIP

## Examples

```
flash reads_1.fastq reads_2.fastq -m 15; jellyfish count -m 25 -s 100M merged.fa; ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Seed file contains exactly 5,019,778 sequences of 270 bp (or 873,058 of 285 bp), matching the reported 1.58x / 0.29x coverage of the ~860 Mbp eel genome
- Every sequence in a seed file has exactly the same length and every identifier is a unique number excluding 0
- No fragment in the output contains a 25-mer occurring more than 25 times in the merged read set
- Merged fragments derive from read pairs with at least 15 nt of overlap between mates

## Limitations

- Large repetitive regions and (sub)telomeric repeats may lack frequent unique 285 bp islands and can therefore be absent from the resulting assembly
- Seeds containing repetitive sequences are permissible but make the TULIP graph much slower to untangle and to optimize seed numbers
- Even with good seeds, scaffolds that could be merged based on long reads may remain separate if the reads do not share a fortuitous seed alignment in the correct position

## Evidence

- [methods] Discard any fragment that contains a 25-mer occurring more than 25 times, removing repetitive sequence as per the 'not too repetitive' seed selection criterion.: [evidence span withheld — anchor status: not_found]
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [readme] Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph, and to optimize seed numbers.: "Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph, and to optimize seed numbers."
- [results] the seed-selection step yielded 5019778 fragments of 270 bp or 873058 fragments of 285 bp from merged Illumina read pairs, corresponding to 1.58-fold or 0.29-fold genome coverage: [evidence span withheld — anchor status: not_found]
- [discussion] large repetitive regions in the genome, as well as (sub) telomeric repeats will not always contain frequent 285 bp islands of unique sequence: "large repetitive regions in the genome, as well as (sub) telomeric repeats will not always contain frequent 285 bp islands of unique sequence"
