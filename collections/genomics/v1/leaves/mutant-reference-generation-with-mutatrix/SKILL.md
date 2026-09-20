---
name: mutant-reference-generation-with-mutatrix
description: 'Use when building a benchmark with known ground-truth variants: a reference
  FASTA must be mutated (e.g. insertions/deletions of 100 bp–4 kbp) so real reads
  can be aligned and SV calls scored against the induced event list.'
when_to_use_negative:
- You need realistic biological variation or allele frequencies drawn from real population
  data — Mutatrix draws alleles from a simulated zeta-distribution power-law spectrum,
  not observed variants.
- Your mappers are being benchmarked on SNVs only in the original reference — no mutated
  reference is needed since no truth set must be constructed.
- A published truth set (e.g. GIAB benchmarks) already covers your reference; simulating
  variants would duplicate it and add no validation value.
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: Mutatrix
  role: Population genome simulator that mutates the reference FASTA and emits the
    VCF truth set used as ground truth for SV detection benchmarking
  repo: https://github.com/ekg/mutatrix
- name: graphmap-reproduce
  role: Benchmark harness that supplies the Mutatrix-mutated E. coli K-12 reference
    (20 induced SVs, 100 bp–4 kbp) and the matching 2D MinION read set
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:cb50d192d167b9f0959da99ce7e1cea4b2a54da79ab5d05ef7594756ac74894d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ekg/mutatrix
  tools:
  - BLASR
  - BWA-MEM
  - GraphMap
  - MinION
  - Mutatrix
  - graphmap-reproduce
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mutant-reference-generation-with-mutatrix

## Summary

Generate a mutated reference genome (FASTA + VCF of induced variants) with the Mutatrix population genome simulator so that known structural variants can be used as ground truth when benchmarking mapper-based SV detection, as done for the E. coli K-12 nanopore benchmark with 20 induced SVs of 100 bp–4 kbp.

## When to use

Use when building a benchmark with known ground-truth variants: a reference FASTA must be mutated (e.g. insertions/deletions of 100 bp–4 kbp) so real reads can be aligned and SV calls scored against the induced event list.

## When NOT to use

- You need realistic biological variation or allele frequencies drawn from real population data — Mutatrix draws alleles from a simulated zeta-distribution power-law spectrum, not observed variants.
- Your mappers are being benchmarked on SNVs only in the original reference — no mutated reference is needed since no truth set must be constructed.
- A published truth set (e.g. GIAB benchmarks) already covers your reference; simulating variants would duplicate it and add no validation value.

## Inputs

- Reference genome FASTA (e.g. E. coli K-12)
- Mutatrix command-line parameters (-S sample id, -P output path, -p copy number, -n sample count)

## Outputs

- VCF file on stdout describing the induced variants (truth set)
- Mutated reference FASTA copies written as <prefix>/<sample id>:<fasta sequence name>:<copy number>.fa

## How to apply

Run Mutatrix on the reference FASTA (e.g. E. coli K-12): it reads the reference, emits a VCF describing the induced variants to stdout, and writes mutated reference copies to a user-defined path or the current directory. Control sample naming with -S, the output directory with -P, and copy count/parameters with -p and -n; each output file follows the naming format <prefix>/<sample id>:<fasta sequence name>:<copy number>.fa. Choose mutation events sized in the 100 bp–4 kbp range so spanning alignments of nanopore reads can detect them directly, and keep the VCF as the truth set. Then feed the mutated reference plus the matching real read set (e.g. 2D MinION reads) into the mapper benchmark, scoring each mapper's calls against the known induced SVs to compute recall and precision.

## Related tools

- **Mutatrix** (Population genome simulator that mutates the reference FASTA and emits the VCF truth set used as ground truth for SV detection benchmarking) — https://github.com/ekg/mutatrix
- **graphmap-reproduce** (Benchmark harness that supplies the Mutatrix-mutated E. coli K-12 reference (20 induced SVs, 100 bp–4 kbp) and the matching 2D MinION read set) — https://github.com/isovic/graphmap-reproduce

## Examples

```
./mutatrix -S sample -P test/ -p 2 -n 10 reference.fasta
```

## Evaluation signals

- The VCF on stdout lists exactly the expected number of induced events (e.g. 20 SVs for the E. coli benchmark) with sizes falling in the 100 bp–4 kbp range.
- Each mutated copy file exists at the expected path and follows the naming format <prefix>/<sample id>:<fasta sequence name>:<copy number>.fa.
- Diffing the mutated FASTA against the original reference reveals only the intended insertion/deletion events, consistent with the VCF records.
- Downstream mapper calls (e.g. GraphMap spanning alignments) recover the induced events, giving non-zero recall and precision when scored against the VCF truth set.

## Limitations

- Mutatrix is a simulator: it distributes alleles according to a zeta distribution intended to mimic the power-law allele frequency spectrum of large population projects, so induced variants are pseudorandom, not biologically observed ones.
- Mutated references are synthetic copies identified by sample id and copy number; pooled-variant detector testing requires managing multiple copies per sample.

## Evidence

- [readme] Mutatrix is a population genome simulator that mutates a reference FASTA and emits a VCF truth set.: [evidence span withheld — anchor status: external_doc]
- [readme] Mutatrix reads a reference FASTA and outputs a VCF of variants to stdout plus mutated copies.: [evidence span withheld — anchor status: external_doc]
- [readme] Mutator's allele-frequency model follows a power-law spectrum consistent with population sequencing projects.: [evidence span withheld — anchor status: external_doc]
- [abstract] GraphMap benchmark uses SVs in the 100 bp–4 kbp range detectable from spanning alignments.: "precise detection of structural variants from length 100 bp to 4 kbp"
