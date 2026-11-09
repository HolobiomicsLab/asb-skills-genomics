---
name: long-read-consensus-polishing
description: Use when you have basecalled nanopore FASTQ/FASTA reads for a small genome
  (e.g. Phage Lambda, E. coli, M. tuberculosis) and need a de novo assembly whose
  error rate quantifies basecaller quality against a known reference.
when_to_use_negative:
- Input is Illumina short-read data — the draft layout step relies on pairwise alignment
  of overlapping long reads
- No reference genome exists for the organism, since the error-rate quantification
  step requires aligning the polished assembly to a reference with graphmap
- The goal is read-level basecalling accuracy (identity to reference reads) rather
  than assembly-level error rate — Chiron reports read-level accuracy separately using
  graphmap alignment of reads
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Aligns overlapping basecalled nanopore reads to build the assembly graph for
    the draft genome
- name: Miniasm
  role: Lays out the Minimap2 alignment graph into a draft genome sequence
- name: Racon
  role: Polishes the Miniasm draft genome for 10 rounds with the corresponding basecalled
    reads
- name: graphmap
  role: Aligns the polished assembly to the reference genome with default parameters
    to compute the assembly error rate
- name: Chiron
  role: 'Produces the basecalled reads (DNA_default model, dna-pre preset: batch_size
    400, segment_len 400, jump 390, beam 30) that serve as assembly input'
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: Reference basecaller (v1.1.1 and v2.0.1, default parameters) whose assemblies
    are compared against Chiron-derived assemblies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:00665949e051838a36462bf597e86a3fd8ea1b95d37101983012516db0b7183e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - Chiron
  - Miniasm
  - Minimap2
  - Racon
  - graphmap
  - minimap2
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-consensus-polishing

## Summary

Builds de novo draft genomes from long nanopore basecalled reads with Minimap2/Miniasm, then iteratively polishes the consensus with 10 rounds of Racon using the same basecalled reads, and quantifies assembly error rate by aligning the polished assembly to a reference with graphmap. This is the workflow the Chiron paper used to convert basecaller accuracy differences into assembly error-rate comparisons.

## When to use

Use when you have basecalled nanopore FASTQ/FASTA reads for a small genome (e.g. Phage Lambda, E. coli, M. tuberculosis) and need a de novo assembly whose error rate quantifies basecaller quality against a known reference.

## When NOT to use

- Input is Illumina short-read data — the draft layout step relies on pairwise alignment of overlapping long reads
- No reference genome exists for the organism, since the error-rate quantification step requires aligning the polished assembly to a reference with graphmap
- The goal is read-level basecalling accuracy (identity to reference reads) rather than assembly-level error rate — Chiron reports read-level accuracy separately using graphmap alignment of reads

## Inputs

- Basecalled nanopore reads (FASTQ/FASTA) from the basecaller under test (e.g. Chiron DNA_default/dna-pre output)
- Raw fast5 files (if basecalling must be re-run first, e.g. with chiron call -p dna-pre)
- Reference genome sequence (e.g. NCBI reference NC_001416.1 for Phage Lambda, bacterial reference sequences)

## Outputs

- Draft genome assemblies per basecaller/species (Minimap2 + Miniasm)
- Polished consensus assemblies (10 rounds of Racon)
- Assembly error-rate table per basecaller and species (assembly_error_rates.tsv)

## How to apply

For each basecaller/species combination, align overlapping basecalled reads with Minimap2 and lay out the assembly with Miniasm to produce a draft genome. Polish each draft with Racon for exactly 10 rounds using the corresponding basecalled reads, since the Chiron paper shows this iteration drives Chiron-derived assemblies to error rates below 0.5% while Albacore-derived assemblies remain up to 0.8% on Lambda and E. coli. Align each polished assembly to its reference (e.g. Phage Lambda NC_001416.1) with graphmap using default parameters and compute the assembly error rate. Tabulate per-basecaller/per-species error rates (e.g. assembly_error_rates.tsv) to compare basecallers at the assembly level rather than the read level.

## Related tools

- **Minimap2** (Aligns overlapping basecalled nanopore reads to build the assembly graph for the draft genome)
- **Miniasm** (Lays out the Minimap2 alignment graph into a draft genome sequence)
- **Racon** (Polishes the Miniasm draft genome for 10 rounds with the corresponding basecalled reads)
- **graphmap** (Aligns the polished assembly to the reference genome with default parameters to compute the assembly error rate)
- **Chiron** (Produces the basecalled reads (DNA_default model, dna-pre preset: batch_size 400, segment_len 400, jump 390, beam 30) that serve as assembly input) — https://github.com/haotianteng/Chiron
- **Albacore** (Reference basecaller (v1.1.1 and v2.0.1, default parameters) whose assemblies are compared against Chiron-derived assemblies)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default -p dna-pre  # then Minimap2+Miniasm draft, 10x Racon, graphmap to NC_001416.1
```

## Evaluation signals

- Chiron-derived polished assemblies reach error rates below 0.5%, while Albacore v1/v2 assemblies reach up to 0.8% on Phage Lambda and E. coli
- Chiron assemblies contain approximately half as many errors as Albacore v1 or v2 assemblies for Phage lambda and E. coli, and fewer errors than Albacore v1 for M. tuberculosis
- Each draft genome has been polished for exactly 10 Racon rounds before error-rate computation
- Error-rate table (assembly_error_rates.tsv) contains one row per basecaller/species combination, with error rates computed after graphmap alignment to the correct reference (e.g. NC_001416.1)

## Limitations

- Further accuracy gains depend on better alignment of overlapping reads and consensus calling, per the paper's discussion
- The comparison is anchored to specific references (Phage Lambda NC_001416.1 and bacterial references); error-rate estimates are only as good as the reference and graphmap alignment
- Earlier versions of MinKNOW were used for non-E. coli samples, a software version inconsistency that could confound cross-dataset comparisons

## Evidence

- [discussion] de novo assembly with Minimap2/Miniasm and 10 rounds of Racon polishing: "We used Minimap2 [34] and Miniasm [35] to generate a draft genome, then Racon [36] was used to polish on the draft genome for 10 rounds."
- [discussion] graphmap alignment to reference with default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [discussion] assembly from pairwise alignment of overlapping reads: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
- [readme] basecalled reads feed downstream assembly: "Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer."
