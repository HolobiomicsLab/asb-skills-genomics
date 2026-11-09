---
name: synthetic-read-simulation-error-profiles
description: Use when benchmarking read mappers (e.g. GraphMap vs BWA-MEM, LAST, BLASR,
  DALIGNER, BLAST) against ground truth, and you need synthetic long reads spanning
  diverse error profiles (Illumina, PacBio, ONT 1D/2D) with known true origins.
when_to_use_negative:
- When the evaluation question is mapping runtime or sensitivity on real sequencing
  runs where true read origins are unknown (real-data tests use src/run-realdata.py
  instead of simulations)
- When the goal is production mapping of a real sample — simulated error profiles
  from ART/PBSIM may not reproduce the error spectrum of a specific MinION flow cell,
  so conclusions should not be transferred directly
- When only short, low-error Illumina reads with an established mapper benchmark are
  involved — the mappers compared here (GraphMap, BLASR, DALIGNER) target long, error-prone
  third-generation reads
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GraphMap
  role: Mapper under test; its mapping-location precision/recall on simulated reads
    is the primary metric (expected >94%, typically >98%)
  repo: https://github.com/isovic/graphmap
- name: ART
  role: Generates synthetic Illumina read sets with the platform error profile
- name: PBSIM
  role: Generates synthetic PacBio and ONT 1D/2D read sets with platform-specific
    error profiles
- name: aligneval
  role: Reproduction repository holding simulator setup (setup-simdata) and per-mapper
    wrapper scripts for evaluating alignments on simulated data
- name: BWA-MEM
  role: Comparison mapper scored on the same simulated read sets
- name: LAST
  role: Comparison mapper scored on the same simulated read sets
- name: BLASR
  role: Comparison mapper scored on the same simulated read sets
- name: DALIGNER
  role: Comparison mapper scored on the same simulated read sets
- name: BLAST
  role: Sensitivity reference mapper ('BLAST-like sensitivity') scored on the same
    simulated read sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:7d34d22217029d62243de6fa364c94840672d43f7ca56709fbb593adb1cf3c96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - ART
  - BLASR
  - BLAST
  - BWA-MEM
  - DALIGNER
  - GCC
  - GraphMap
  - LAST
  - PBSIM
  - aligneval
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.782608695652174
n_claims: 23
---

# Synthetic read simulation with platform-specific error profiles

## Summary

Generate simulated sequencing reads with known ground-truth origins using ART (Illumina) and PBSIM (PacBio, ONT 1D/2D) so that mapper precision, recall, and per-base alignment accuracy can be measured exactly against simulated truth positions rather than uncertain real-data labels.

## When to use

Use when benchmarking read mappers (e.g. GraphMap vs BWA-MEM, LAST, BLASR, DALIGNER, BLAST) against ground truth, and you need synthetic long reads spanning diverse error profiles (Illumina, PacBio, ONT 1D/2D) with known true origins.

## When NOT to use

- When the evaluation question is mapping runtime or sensitivity on real sequencing runs where true read origins are unknown (real-data tests use src/run-realdata.py instead of simulations)
- When the goal is production mapping of a real sample — simulated error profiles from ART/PBSIM may not reproduce the error spectrum of a specific MinION flow cell, so conclusions should not be transferred directly
- When only short, low-error Illumina reads with an established mapper benchmark are involved — the mappers compared here (GraphMap, BLASR, DALIGNER) target long, error-prone third-generation reads

## Inputs

- Reference genome FASTA file
- Platform-specific error profiles for ART (Illumina) and PBSIM (PacBio, ONT 1D, ONT 2D) as specified in the paper's Methods
- Simulator executables: ART and PBSIM (installed automatically by aligneval/setup-simdata)
- Mapper binaries: GraphMap, BLAST, BWA-MEM, LAST, BLASR, DALIGNER

## Outputs

- Simulated read sets (FASTA/FASTQ) across Illumina, PacBio, ONT 1D and ONT 2D error profiles with known true read origins
- Per-mapper mapping-location precision and recall scores
- Per-base alignment accuracy values (~70% expected for high-error-rate synthetic reads)
- Fig. 2-style comparison plot and Supplementary Data 1-style results table

## How to apply

Simulate synthetic Illumina read sets with ART and PacBio plus ONT 1D/2D read sets with PBSIM using the platform-specific error profiles from the paper's Methods, so every read's true origin is known. Align every simulated read set to the reference genome with all mappers under test (GraphMap, BLAST, BWA-MEM, LAST, BLASR, DALIGNER) and score mapping-location precision and recall by comparing mapped positions to the true simulated origins. Additionally compute per-base alignment accuracy for each mapper (expected ~70% correct per-base alignments for high-error-rate synthetic reads). The paper's reproduction infrastructure wraps this in the 'aligneval' repository, installed automatically via 'setup-simdata', which also auto-downloads and installs all aligners through wrapper scripts. Aggregate precision/recall and per-base accuracy into a Fig. 2-style comparison plot and a Supplementary Data 1-style table; the validation target is GraphMap mapping-location precision/recall above 94%.

## Related tools

- **GraphMap** (Mapper under test; its mapping-location precision/recall on simulated reads is the primary metric (expected >94%, typically >98%)) — https://github.com/isovic/graphmap
- **ART** (Generates synthetic Illumina read sets with the platform error profile)
- **PBSIM** (Generates synthetic PacBio and ONT 1D/2D read sets with platform-specific error profiles)
- **aligneval** (Reproduction repository holding simulator setup (setup-simdata) and per-mapper wrapper scripts for evaluating alignments on simulated data)
- **BWA-MEM** (Comparison mapper scored on the same simulated read sets)
- **LAST** (Comparison mapper scored on the same simulated read sets)
- **BLASR** (Comparison mapper scored on the same simulated read sets)
- **DALIGNER** (Comparison mapper scored on the same simulated read sets)
- **BLAST** (Sensitivity reference mapper ('BLAST-like sensitivity') scored on the same simulated read sets)

## Examples

```
python src/run-realdata.py setup-simdata
```

## Evaluation signals

- GraphMap's mapping-location precision and recall on the synthetic read sets are consistently above 94% across technologies and error rates
- Per-base alignment accuracy for each mapper is around 70% correct per-base alignments on high-error-rate simulated reads
- Every scored alignment can be traced to a known simulated true origin, so precision/recall are computed exactly rather than estimated
- Results reproduce the paper's Fig. 2-style comparison and Supplementary Data 1-style table across all six mappers (GraphMap, BLAST, BWA-MEM, LAST, BLASR, DALIGNER)
- GraphMap's location precision typically exceeds 98%, distinguishing candidate locations >94% identical on the human genome

## Limitations

- Simulation fidelity depends on the ART/PBSIM platform-specific error profiles from the paper's Methods; real MinION runs can vary in error rate and profile across or within runs
- Per-base alignment accuracy on error-prone synthetic reads is only ~70%, so downstream variant calling on simulated mappings is constrained
- Index construction for large references is memory-heavy (~14x reference size for the final hash index, ~42 GB peak for human genome), so simulated benchmarks on large genomes need large-memory machines

## Evidence

- [full_text] Simulate synthetic Illumina read sets with ART and PacBio plus ONT 1D/2D read sets with PBSIM using the platform-specific error profiles from the paper's Methods.: [evidence span withheld — anchor status: not_found]
- [readme] Simulations are implemented in our other repository aligneval which will be downloaded and setup automatically.: [evidence span withheld — anchor status: external_doc]
- [readme] Aligneval includes the wrapper scripts for each mapper/aligner required to run the tests. It will automatically download and install all aligners.: [evidence span withheld — anchor status: external_doc]
- [discussion] GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome.: "GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome"
- [full_text] GraphMap reproduces the reported benchmark, with mapping-location precision/recall above 94% and per-base alignment accuracy around 70%.: [evidence span withheld — anchor status: not_found]
