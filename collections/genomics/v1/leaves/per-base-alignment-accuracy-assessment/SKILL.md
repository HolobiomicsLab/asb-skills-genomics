---
name: per-base-alignment-accuracy-assessment
description: Use when benchmarking long-read mappers (GraphMap, BWA-MEM, LAST, BLASR,
  DALIGNER, BLAST) on simulated Illumina/PacBio/ONT 1D/2D reads where the true genomic
  origin of every read is known and per-base alignment quality must be quantified.
when_to_use_negative:
- 'Real (non-simulated) reads: per-base accuracy cannot be computed because the true
  origin of each base is unknown — the metric requires simulated reads with recorded
  origins.'
- 'When only mapping-location precision/recall is needed: the article reports location
  precision/recall >94% while per-base accuracy is ~70%, so a high-precision mapper
  is not automatically a high per-base-accuracy one, but the metric adds cost only
  when alignment-string fidelity matters.'
- 'Reference genomes too large for available memory: GraphMap''s hash index peaks
  at ~42 GB for a human genome (README), so per-base benchmarking on large genomes
  requires ~100 GB machines per the article''s discussion.'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Mapper under evaluation; its alignments are scored for per-base accuracy against
    simulated read origins
  repo: https://github.com/isovic/graphmap
- name: graphmap-reproduce
  role: Reproduction scripts; installs aligneval, which wraps each mapper/aligner
    for consistent evaluation of simulated-data accuracy
- name: BWA-MEM
  role: Comparison mapper scored alongside GraphMap for per-base alignment accuracy
- name: PBSIM
  role: Simulates PacBio and ONT 1D/2D reads with known origins so per-base accuracy
    can be computed
- name: ART
  role: Simulates Illumina read sets with known origins for the accuracy benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:39b44964d73e29c9e222723124990befdae0447000e9e086b9e85d0971319eb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - ART
  - BLASR
  - BWA-MEM
  - DALIGNER
  - GCC
  - GraphMap
  - LAST
  - PBSIM
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# per-base-alignment-accuracy-assessment

## Summary

Scores how faithfully a mapper's alignments reproduce the true simulated origin of each read at the base level, complementing mapping-location precision/recall. In the GraphMap benchmark this distinguishes mappers that place reads correctly (precision/recall >94%) from those whose internal alignment strings are only ~70% correct per base.

## When to use

Use when benchmarking long-read mappers (GraphMap, BWA-MEM, LAST, BLASR, DALIGNER, BLAST) on simulated Illumina/PacBio/ONT 1D/2D reads where the true genomic origin of every read is known and per-base alignment quality must be quantified.

## When NOT to use

- Real (non-simulated) reads: per-base accuracy cannot be computed because the true origin of each base is unknown — the metric requires simulated reads with recorded origins.
- When only mapping-location precision/recall is needed: the article reports location precision/recall >94% while per-base accuracy is ~70%, so a high-precision mapper is not automatically a high per-base-accuracy one, but the metric adds cost only when alignment-string fidelity matters.
- Reference genomes too large for available memory: GraphMap's hash index peaks at ~42 GB for a human genome (README), so per-base benchmarking on large genomes requires ~100 GB machines per the article's discussion.

## Inputs

- Simulated read sets with known true origins (ART-simulated Illumina; PBSIM-simulated PacBio and ONT 1D/2D FASTA/FASTQ)
- Reference genome FASTA used for both alignment and simulation
- SAM/BAM alignment outputs from each mapper (GraphMap, BLAST, BWA-MEM, LAST, BLASR, DALIGNER)

## Outputs

- Per-mapper per-base alignment accuracy (fraction of correctly aligned bases; expected ~70%)
- Combined benchmark table (Supplementary Data 1 style) and Fig. 2-style comparison plot of precision, recall and per-base accuracy

## How to apply

Simulate reads whose origins are recorded (ART for Illumina; PBSIM for PacBio and ONT 1D/2D using the paper's platform-specific error profiles), then align every read set to the reference with each mapper under test. Because simulated reads carry their true coordinates, compare each mapper's mapped positions and CIGAR alignments against the true origins rather than against a second mapper. Compute the fraction of correctly aligned bases per mapper — in the GraphMap study this per-base accuracy sits around 70% even for mappers achieving >94% mapping-location precision and recall, so the two metrics must be reported together. Use the aligneval evaluation package (auto-installed by the graphmap-reproduce setup) which provides wrapper scripts for each mapper/aligner so that scoring is applied consistently across tools. Aggregate the per-base accuracy numbers alongside location precision/recall into a Fig. 2-style comparison plot and Supplementary Data 1-style table.

## Related tools

- **GraphMap** (Mapper under evaluation; its alignments are scored for per-base accuracy against simulated read origins) — https://github.com/isovic/graphmap
- **graphmap-reproduce** (Reproduction scripts; installs aligneval, which wraps each mapper/aligner for consistent evaluation of simulated-data accuracy)
- **BWA-MEM** (Comparison mapper scored alongside GraphMap for per-base alignment accuracy)
- **PBSIM** (Simulates PacBio and ONT 1D/2D reads with known origins so per-base accuracy can be computed)
- **ART** (Simulates Illumina read sets with known origins for the accuracy benchmark)

## Examples

```
src/run-realdata.py setup-tools && src/run-realdata.py setup-simdata
```

## Evaluation signals

- Per-base alignment accuracy lands near 70% for the assessed mappers, matching the paper's reported benchmark.
- Per-base accuracy is reported jointly with mapping-location precision/recall (both >94% for GraphMap), not as a standalone number.
- All mappers are scored with the same wrapper scripts from the aligneval package, so differences reflect mapper behavior not scoring logic.
- Simulated reads' true origins are available for every read, allowing direct position and CIGAR comparison rather than cross-mapper agreement.
- Results aggregate cleanly into the Fig. 2-style plot and Supplementary Data 1-style table without unmapped or unparseable alignments dominating the score.

## Limitations

- Per-base accuracy is only computable on simulated data; on real nanopore data the paper instead reports sensitivity proxies such as >90% of reads and 95% of bases mapped.
- GraphMap's hash-table index requires large-memory machines (~100 GB for the human genome per the article; ~42 GB peak during construction per the README v0.5.1 notes).
- The default minimizer window (5) and --freq-percentile 0.99 seed filtering slightly reduce sensitivity on large references; use -x sensitive to reproduce earlier behavior when benchmarking sensitivity-critical scenarios.

## Evidence

- [methods] Compute per-base alignment accuracy for each mapper (expected ~70% correct per-base alignments).: [evidence span withheld — anchor status: not_found]
- [results] GraphMap reproduces the reported benchmark, with mapping-location precision/recall above 94% and per-base alignment accuracy around 70%.: [evidence span withheld — anchor status: not_found]
- [other] Align every simulated read set to the reference genome with GraphMap, BLAST, BWA-MEM, LAST, BLASR and DALIGNER.: [evidence span withheld — anchor status: not_found]
- [readme] Aligneval includes the wrapper scripts for each mapper/aligner required to run the tests.: [evidence span withheld — anchor status: not_found]
