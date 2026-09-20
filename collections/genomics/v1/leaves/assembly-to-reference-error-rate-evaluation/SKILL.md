---
name: assembly-to-reference-error-rate-evaluation
description: Use when you have de novo assemblies built from basecalled nanopore reads
  (Chiron or Albacore FASTQ) for organisms with a known reference genome, and you
  need a per-basecaller error-rate comparison rather than a read-level accuracy metric.
when_to_use_negative:
- The target organism has no suitable reference genome — this skill is an assembly-to-reference
  comparison and cannot compute error rates de novo.
- You only need read-level basecalling accuracy — read-level accuracy against a reference
  with graphmap is a separate evaluation, not assembly error rate.
- Assemblies are expected to have large structural differences from the reference
  (e.g., unknown plasmids, rearrangements) where whole-assembly-to-reference alignment
  misrepresents error rate.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Chiron
  role: Basecaller whose raw-signal-to-sequence output reads feed the de novo assemblies
    being evaluated
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: Baseline ONT basecaller (v1.1.1 and v2.0.1, default parameters) whose assemblies
    provide the comparison error rates
- name: Minimap2
  role: Aligns overlapping basecalled reads to build the draft assembly
- name: Miniasm
  role: Lays out the draft genome from Minimap2 read overlaps
- name: Racon
  role: Polishes each draft genome for 10 rounds using the corresponding basecalled
    reads
- name: graphmap
  role: Aligns each polished assembly to its reference genome with default parameters
    for error-rate computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:c8f768f8dfbfd8c31f21bdc118c2e05ed7eccacf0fa8552cd1f68f04e922f449
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# assembly-to-reference-error-rate-evaluation

## Summary

Quantifies de novo genome assembly error rate by aligning polished assemblies back to a reference genome with graphmap and tabulating errors per basecaller/species combination. Used to compare assembly quality across basecallers (e.g., Chiron vs Albacore v1.1.1/v2.0.1) on nanopore data.

## When to use

Use when you have de novo assemblies built from basecalled nanopore reads (Chiron or Albacore FASTQ) for organisms with a known reference genome, and you need a per-basecaller error-rate comparison rather than a read-level accuracy metric.

## When NOT to use

- The target organism has no suitable reference genome — this skill is an assembly-to-reference comparison and cannot compute error rates de novo.
- You only need read-level basecalling accuracy — read-level accuracy against a reference with graphmap is a separate evaluation, not assembly error rate.
- Assemblies are expected to have large structural differences from the reference (e.g., unknown plasmids, rearrangements) where whole-assembly-to-reference alignment misrepresents error rate.

## Inputs

- Basecalled reads (FASTQ) from Chiron (DNA_default model, dna-pre preset) and Albacore v1.1.1/v2.0.1 for each species
- Raw fast5 nanopore files for Lambda, E. coli, and M. tuberculosis (for basecalling)
- Reference genome sequences (e.g., NCBI reference NC_001416.1 for Phage Lambda and bacterial reference sequences)

## Outputs

- Polished de novo assemblies (one per basecaller/species combination, after 10 Racon rounds)
- assembly_error_rates.tsv with error rates per basecaller and species

## How to apply

For each basecaller/species combination, generate a draft genome by aligning overlapping basecalled reads with Minimap2 and laying out the assembly with Miniasm. Polish each draft with Racon for 10 rounds using the corresponding basecalled reads. Align each polished assembly to its reference (e.g., Phage Lambda NC_001416.1, E. coli, M. tuberculosis references) using graphmap with default parameters, then compute the assembly error rate from the alignment. Tabulate error rates per basecaller and species into a TSV; the expected pattern is Chiron-derived assemblies below 0.5% error versus Albacore assemblies reaching up to 0.8% on Lambda and E. coli.

## Related tools

- **Chiron** (Basecaller whose raw-signal-to-sequence output reads feed the de novo assemblies being evaluated) — https://github.com/haotianteng/Chiron
- **Albacore** (Baseline ONT basecaller (v1.1.1 and v2.0.1, default parameters) whose assemblies provide the comparison error rates)
- **Minimap2** (Aligns overlapping basecalled reads to build the draft assembly)
- **Miniasm** (Lays out the draft genome from Minimap2 read overlaps)
- **Racon** (Polishes each draft genome for 10 rounds using the corresponding basecalled reads)
- **graphmap** (Aligns each polished assembly to its reference genome with default parameters for error-rate computation)

## Examples

```
python chiron/entry.py call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- Chiron-derived polished assemblies achieve error rates below 0.5% on Phage Lambda and E. coli
- Albacore-derived assemblies reach up to 0.8% error rate on Lambda and E. coli, so the comparison must reproduce the Chiron-advantage pattern
- Every draft assembly has exactly 10 rounds of Racon polishing before reference alignment
- Output is a per-basecaller/per-species error-rate table (assembly_error_rates.tsv) covering Lambda, E. coli, and M. tuberculosis combinations
- graphmap is run with default parameters for all assembly-to-reference alignments (no per-sample tuning)

## Limitations

- Polishing quality depends on the basecalled reads themselves; better alignment of overlapping reads and consensus calling was noted as a source of further accuracy gains.
- Computational cost: the upstream Chiron basecalling step is substantially more computationally expensive than Albacore.
- Reference-based evaluation only works where a high-quality reference exists (e.g., NC_001416.1 for Phage Lambda); Chiron's advantage was species-dependent (smaller gain for M. tuberculosis vs Albacore v1).

## Evidence

- [discussion] Draft assembly with Miniasm and Racon polishing: "We used Minimap2 [34] and Miniasm [35] to generate a draft genome, then Racon [36] was used to polish on the draft genome for 10 rounds."
- [discussion] graphmap reference alignment: "aligned to the reference genome using graphmap[33] with the default parameters"
- [readme] dna-pre preset parameters used for basecalling reads feeding the assemblies: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [discussion] Assembly built from pairwise read alignment: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
