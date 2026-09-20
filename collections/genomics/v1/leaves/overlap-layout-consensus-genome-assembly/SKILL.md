---
name: overlap-layout-consensus-genome-assembly
description: Use when you have raw fast5-derived basecalled reads (FASTQ/FASTA) for
  a small genome and need a de novo draft assembly whose error rate will be compared
  across basecallers.
when_to_use_negative:
- The goal is read-level basecalling accuracy comparison rather than assembly-level
  error comparison (align reads to the reference with graphmap instead)
- Input is already an assembled genome with no underlying basecalled reads available
  for Racon polishing
- The genome is large/human-scale — the paper's workflow was applied to bacterial
  and viral genomes (Lambda, E. coli, M. tuberculosis), not large eukaryotic assemblies
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Minimap2
  role: Aligns overlapping basecalled reads to generate overlaps for the layout step
- name: Miniasm
  role: Lays out the draft genome from Minimap2 read overlaps
- name: Racon
  role: Polishes the Miniasm draft genome for 10 rounds using the corresponding basecalled
    reads
- name: graphmap
  role: Aligns the polished assembly to the reference genome with default parameters
    to compute the assembly error rate
- name: Chiron
  role: Basecalls the raw fast5 files to produce the reads fed into the assembly pipeline
  repo: https://github.com/haotianteng/Chiron
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:d2c1ab8afef67a3d7b396361aba967c0b8e4432f315e143dfbb2215f406638d6
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# overlap-layout-consensus-genome-assembly

## Summary

De novo assembly of nanopore basecalled reads via an overlap-layout-consensus pipeline: build a draft genome by aligning overlapping reads with Minimap2 and laying out the assembly with Miniasm, then polish with 10 rounds of Racon before evaluating assembly error against a reference.

## When to use

Use when you have raw fast5-derived basecalled reads (FASTQ/FASTA) for a small genome and need a de novo draft assembly whose error rate will be compared across basecallers.

## When NOT to use

- The goal is read-level basecalling accuracy comparison rather than assembly-level error comparison (align reads to the reference with graphmap instead)
- Input is already an assembled genome with no underlying basecalled reads available for Racon polishing
- The genome is large/human-scale — the paper's workflow was applied to bacterial and viral genomes (Lambda, E. coli, M. tuberculosis), not large eukaryotic assemblies

## Inputs

- Basecalled nanopore reads (FASTQ/FASTA) from raw fast5 files, per basecaller/species
- Reference genome sequences (e.g. Phage Lambda NCBI reference NC_001416.1)
- Raw fast5 signal files if re-basecalling is required before assembly

## Outputs

- Draft genome assembly per basecaller/species from Minimap2 + Miniasm
- Polished consensus assembly after 10 Racon rounds
- assembly_error_rates.tsv — per-basecaller, per-species assembly error rates

## How to apply

Basecall the raw fast5 files first (e.g. Chiron with the DNA_default model and dna-pre preset: batch_size 400, segment_len 400, jump 390, beam 30). For each basecaller/species combination, align overlapping reads with Minimap2 and lay out the draft genome with Miniasm. Polish each draft with Racon for 10 rounds using the same basecalled reads that produced it. To evaluate, align each polished assembly to its reference (e.g. Phage Lambda NC_001416.1) with graphmap using default parameters and compute the assembly error rate per basecaller and species.

## Related tools

- **Minimap2** (Aligns overlapping basecalled reads to generate overlaps for the layout step)
- **Miniasm** (Lays out the draft genome from Minimap2 read overlaps)
- **Racon** (Polishes the Miniasm draft genome for 10 rounds using the corresponding basecalled reads)
- **graphmap** (Aligns the polished assembly to the reference genome with default parameters to compute the assembly error rate)
- **Chiron** (Basecalls the raw fast5 files to produce the reads fed into the assembly pipeline) — https://github.com/haotianteng/Chiron

## Examples

```
minimap2 -x ava-ont reads.fastq reads.fastq > overlaps.paf; miniasm -f reads.fastq overlaps.paf > draft.gfa
```

## Evaluation signals

- Polished Chiron assemblies reach error rates below 0.5% while Albacore assemblies reach up to 0.8% on Lambda and E. coli
- Exactly 10 rounds of Racon polishing are applied to every draft genome
- Each polished assembly aligns cleanly to its reference (e.g. NC_001416.1) with graphmap default parameters
- Error rates are tabulated per basecaller and species in assembly_error_rates.tsv, and Chiron assemblies show roughly half the errors of Albacore v1/v2 on Phage lambda and E. coli

## Limitations

- Further accuracy gains depend on better alignment of overlapping reads and consensus calling
- Assembly quality reflects the basecaller's read accuracy; Chiron remains within 2% of Albacore accuracy on Human DNA
- The comparison was established only for small genomes (Phage lambda, E. coli, M. tuberculosis)

## Evidence

- [discussion] OLC pipeline tools: "We used Minimap2 [34] and Miniasm [35] to generate a draft genome, then Racon [36] was used to polish on the draft genome for 10 rounds."
- [discussion] Reference alignment for error rate: "aligned to the reference genome using graphmap[33] with the default parameters"
- [discussion] OLC concept context: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
- [readme] Chiron basecalling parameters feeding assembly: "python chiron/entry.py call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default -p dna-pre"
