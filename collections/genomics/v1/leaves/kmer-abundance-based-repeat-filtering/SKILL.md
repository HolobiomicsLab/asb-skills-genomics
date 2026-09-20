---
name: kmer-abundance-based-repeat-filtering
description: Use when you need to derive sparse, low-repetitiveness seed sequences
  (e.g. 270 bp or 285 bp) from merged short reads for a long-read seed-based assembler
  like TULIP.
when_to_use_negative:
- When seeds with repetitive sequence are acceptable and runtime is not a constraint
  — the paper notes repetitive seeds 'should be fine' but take much longer to untangle
  the graph
- When the goal is genome-size estimation rather than seed selection — k-mer analysis
  there (~860 Mbp) is used to estimate coverage, not to filter fragments
- When input fragments already have uniform length and unique identifiers, i.e. are
  already validated TULIP seeds
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Jellyfish
  role: Counts 25-mer occurrences in merged Illumina fragments to identify over-represented
    (repetitive) k-mers
- name: FLASh
  role: Merges paired-end Illumina reads with a minimum overlap of 15 nt to produce
    the fragments that are k-mer filtered
- name: Sickle
  role: Sliding-window adaptive quality trimming of FASTQ reads prior to merging
  repo: https://github.com/najoshi/sickle
- name: TULIP
  role: Downstream assembler whose strict seed requirements (unique numeric IDs, uniform
    seed length, low repetitiveness) the filtered seed sets must satisfy
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:0359215fac3abaeb08c8ed88cd04c241d88b09eff520af25a74e3a9dbd670653
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# k-mer abundance-based repeat filtering

## Summary

Removes repetitive sequences from Illumina read fragments by counting k-mers (25-mers) with Jellyfish and discarding any fragment containing a k-mer observed more than 25 times, yielding sparse 'not too repetitive' seed fragments for the TULIP assembly of the ~860 Mbp European eel genome.

## When to use

Use when you need to derive sparse, low-repetitiveness seed sequences (e.g. 270 bp or 285 bp) from merged short reads for a long-read seed-based assembler like TULIP.

## When NOT to use

- When seeds with repetitive sequence are acceptable and runtime is not a constraint — the paper notes repetitive seeds 'should be fine' but take much longer to untangle the graph
- When the goal is genome-size estimation rather than seed selection — k-mer analysis there (~860 Mbp) is used to estimate coverage, not to filter fragments
- When input fragments already have uniform length and unique identifiers, i.e. are already validated TULIP seeds

## Inputs

- Paired-end Illumina FASTQ read files (quality-trimmed, e.g. with Sickle)
- Merged paired-end fragments in FASTA/FASTQ produced by FLASh (min overlap 15 nt)
- Jellyfish 25-mer count table computed on the merged fragments

## Outputs

- FASTA of 5,019,778 seed fragments of exactly 270 bp (1.58x coverage)
- FASTA of 873,058 seed fragments of exactly 285 bp (0.29x coverage)
- Seed FASTA files with unique numeric sequence identifiers (excluding 0) and uniform sequence length

## How to apply

First merge paired-end Illumina reads with FLASh, requiring a minimum mate-pair overlap of 15 nt, and quality-trim with Sickle. Count 25-mers across all merged fragments using Jellyfish. Then apply the abundance filter: discard any fragment that contains a 25-mer occurring more than 25 times, which operationalizes the TULIP 'not too repetitive' seed selection criterion — repetitive seeds are tolerable in principle but make untangling the TULIP graph much slower. Finally size-select the surviving fragments to a uniform length (here 5,019,778 fragments of exactly 270 bp, corresponding to 1.58x coverage of the ~860 Mbp genome, or 873,058 fragments of exactly 285 bp, 0.29x coverage) and write each set to FASTA with unique numeric identifiers (excluding 0).

## Related tools

- **Jellyfish** (Counts 25-mer occurrences in merged Illumina fragments to identify over-represented (repetitive) k-mers)
- **FLASh** (Merges paired-end Illumina reads with a minimum overlap of 15 nt to produce the fragments that are k-mer filtered)
- **Sickle** (Sliding-window adaptive quality trimming of FASTQ reads prior to merging) — https://github.com/najoshi/sickle
- **TULIP** (Downstream assembler whose strict seed requirements (unique numeric IDs, uniform seed length, low repetitiveness) the filtered seed sets must satisfy) — https://github.com/Generade-nl/TULIP

## Evaluation signals

- Seed FASTA contains exactly 5,019,778 sequences of 270 bp and/or 873,058 sequences of 285 bp
- Reported coverage matches expectations: 1.58x (270 bp set) or 0.29x (285 bp set) against the ~860 Mbp eel genome estimated from k-mer analysis
- All sequence identifiers in each seed file are unique numeric values excluding 0
- Every seed in a file is of exactly the same length, as required by tulipseed.perl
- No retained fragment contains a 25-mer with abundance greater than 25 in the Jellyfish count table

## Limitations

- Large repetitive regions and (sub)telomeric repeats may lack frequent islands of unique sequence and can therefore be absent from the resulting assembly
- Scaffolds that could be merged based on long reads remain separate if these reads do not share a fortuitous seed alignment in the correct position
- Seeds containing repetitive sequences are allowed in principle but make untangling the TULIP graph and optimizing seed numbers much slower

## Evidence

- [readme] Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph, and to optimize seed numbers.: "Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph, and to optimize seed numbers."
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [methods] These are pre-selected seed sequenced, with selection criteria 'not too repetitive'.: "These are pre-selected seed sequenced, with selection criteria 'not too repetitive'."
- [discussion] the genome size predicted from k-mer analysis (~860 Mbp): "the genome size predicted from k-mer analysis (~860 Mbp)"
