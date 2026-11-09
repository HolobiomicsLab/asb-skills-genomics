---
name: cross-mapper-alignment-comparison
description: Use when a new mapper must be validated against established tools on
  error-prone long reads (e.g. MinION 1D/2D, PacBio) and quantitative sensitivity/precision/consensus-error
  comparisons are required.
when_to_use_negative:
- The goal is de novo assembly or read overlapping rather than mapping accuracy benchmarking
  (GraphMap's 'owler' overlap mode serves that purpose separately).
- 'The reference is very large and memory-constrained: GraphMap''s hash index is ~14x
  reference size (~100 GB for human genome), making a cross-mapper benchmark on large
  genomes resource-prohibitive.'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: Mapper under test; built from source with 'make modules' and 'make', run with
    default parameters across Illumina, PacBio and MinION read sets
  repo: https://github.com/isovic/graphmap
- name: LAST
  role: Comparison mapper run with recommended nanopore settings; reference point
    for mapped-bases sensitivity and consensus-error counts
- name: BWA-MEM
  role: Comparison mapper for synthetic and real MinION benchmarks; consensus-error
    baseline (>1,000 E. coli errors)
- name: BLASR
  role: Comparison mapper for PacBio-style long reads in the precision/recall and
    mapped-bases benchmarks
- name: DALIGNER
  role: Comparison mapper included in the synthetic and MinION mapping benchmark panels
- name: BLAST
  role: Sensitivity gold standard on synthetic long reads; GraphMap matched BLAST
    sensitivity while being orders of magnitude faster
- name: ART
  role: Simulator generating synthetic Illumina read sets with defined error profiles
    for ground-truth benchmarking
- name: PBSIM
  role: Simulator generating PacBio and ONT 1D/2D read sets with platform-specific
    error profiles
- name: Nanopolish
  role: Downstream consensus caller whose benefit from GraphMap mappings is asserted
    but not demonstrated in the benchmark
- name: graphmap-reproduce
  role: 'Reproduction bundle: run-realdata.py retrieves real MinION data sets and
    installs benchmark tools (PyVCF, Tabix, VCFtools, LoFreq, samscripts, Mutatrix,
    Bamsurgeon)'
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:5432a25ae464366e85512872342fdcd98f21f8cacdc7edb90bb48a8b1fd9a526
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
  - MinION
  - Nanopolish
  - PBSIM
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cross-mapper-alignment-comparison

## Summary

Benchmark multiple sequence mappers (GraphMap, LAST, BWA-MEM, BLASR, DALIGNER, BLAST) head-to-head on the same simulated and real MinION read sets, scoring mapping-location precision/recall, fraction of mapped reads/bases, and downstream consensus accuracy.

## When to use

Use when a new mapper must be validated against established tools on error-prone long reads (e.g. MinION 1D/2D, PacBio) and quantitative sensitivity/precision/consensus-error comparisons are required.

## When NOT to use

- The goal is de novo assembly or read overlapping rather than mapping accuracy benchmarking (GraphMap's 'owler' overlap mode serves that purpose separately).
- The reference is very large and memory-constrained: GraphMap's hash index is ~14x reference size (~100 GB for human genome), making a cross-mapper benchmark on large genomes resource-prohibitive.

## Inputs

- Simulated long reads (PacBio, ONT 1D/2D) with known true origins, generated with PBSIM from platform error profiles
- Simulated short reads (Illumina) generated with ART
- Published real MinION read sets with reference genomes packaged in the isovic/graphmap-reproduce bundle (eight data sets incl. E. coli ~80x coverage)
- Reference genome FASTA files for each read set

## Outputs

- Per-mapper mapping-location precision and recall table (Supplementary Data 1-style)
- Per-mapper fraction of mapped reads and mapped bases (e.g. minion_mapping_consensus_benchmark.csv)
- Per-mapper whole-genome consensus error counts at a Q50 threshold
- Fig. 2-style precision/recall comparison plot

## How to apply

Compile each mapper from source (GraphMap: 'make modules' then 'make' with GCC >= 4.7) and simulate reads per technology with ART (Illumina) and PBSIM (PacBio, ONT 1D/2D) using platform-specific error profiles, or retrieve real published MinION sets from the isovic/graphmap-reproduce bundle. Align every read set to its reference with each mapper using each tool's recommended nanopore settings, then score mapping-location precision and recall against the true simulated origins, compute fractions of mapped reads and mapped bases per data set, and derive whole-genome consensus from alignments (e.g. E. coli ~80x coverage at Q50 quality threshold) counting consensus errors per mapper. Aggregate the numbers into a Fig. 2-style comparison plot, Supplementary Data 1-style table, and a per-mapper CSV. Accept the reproduction when GraphMap shows >94% precision/recall, ~70% per-base alignment accuracy, >95% mapped bases with a 10-80% gain over other mappers, and <40 E. coli Q50 consensus errors versus >1,000 for LAST and BWA-MEM.

## Related tools

- **GraphMap** (Mapper under test; built from source with 'make modules' and 'make', run with default parameters across Illumina, PacBio and MinION read sets) — https://github.com/isovic/graphmap
- **LAST** (Comparison mapper run with recommended nanopore settings; reference point for mapped-bases sensitivity and consensus-error counts)
- **BWA-MEM** (Comparison mapper for synthetic and real MinION benchmarks; consensus-error baseline (>1,000 E. coli errors))
- **BLASR** (Comparison mapper for PacBio-style long reads in the precision/recall and mapped-bases benchmarks)
- **DALIGNER** (Comparison mapper included in the synthetic and MinION mapping benchmark panels)
- **BLAST** (Sensitivity gold standard on synthetic long reads; GraphMap matched BLAST sensitivity while being orders of magnitude faster)
- **ART** (Simulator generating synthetic Illumina read sets with defined error profiles for ground-truth benchmarking)
- **PBSIM** (Simulator generating PacBio and ONT 1D/2D read sets with platform-specific error profiles)
- **Nanopolish** (Downstream consensus caller whose benefit from GraphMap mappings is asserted but not demonstrated in the benchmark)
- **graphmap-reproduce** (Reproduction bundle: run-realdata.py retrieves real MinION data sets and installs benchmark tools (PyVCF, Tabix, VCFtools, LoFreq, samscripts, Mutatrix, Bamsurgeon)) — https://github.com/isovic/graphmap-reproduce

## Examples

```
git clone https://github.com/isovic/graphmap.git && cd graphmap && make modules && make && ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fastq -o out.sam
```

## Evaluation signals

- Synthetic-data mapping-location precision and recall exceed 94% for GraphMap on every simulated technology, including high-error-rate ONT 1D reads
- Per-base alignment accuracy is ~70% correct for GraphMap alignments
- On real MinION data GraphMap maps >90% of reads and >95% of bases, with a 10-80% increase in mapped bases over other mappers (18% on a recent MinION MkI set)
- E. coli Q50 consensus at ~80x coverage yields <40 errors for GraphMap versus >1,000 errors for LAST and BWA-MEM
- Fig. 2-style precision/recall values and Supplementary Data 1-style table match the published Fig. 2/3b-d benchmark values

## Limitations

- GraphMap requires large-memory machines (~100 GB for the human genome) because the hash index is ~14x reference size; FM-index memory reduction was only 'currently under implementation'
- Benefit for consensus callers (Nanopolish, PoreSeq) was asserted ('should do so') but not demonstrated in the benchmark
- Minimizer/frequency-percentile filtering (--freq-percentile 0.99) slightly reduces sensitivity, though it plays a huge role when mapping to large references; use '-x sensitive' or '--double-index' to disable it
- Percentile-based seed filtering and large minimizer windows trade sensitivity for speed and memory on large references

## Evidence

- [abstract] GraphMap increases mapping sensitivity by 10-80% and maps >95% of bases compared with short- and long-read mappers: "GraphMap increases mapping sensitivity by 10–80% and maps >95% of bases"
- [discussion] BLAST-like sensitivity while orders of magnitude faster: "GraphMap was the only mapper that exhibited BLAST-like sensitivity, while being orders of magnitude faster than BLAST"
- [discussion] MinION sensitivity lead quantification: "GraphMap was unmatched in terms of sensitivity, mapping >90% of reads and 95% of bases on average. Compared with other mappers, this lead to a 10–80% increase in mapped bases"
- [readme] Build-from-source requirement for the benchmark: "git clone https://github.com/isovic/graphmap.git
cd graphmap
make modules
make"
- [readme] Frequency-percentile seed filtering trade-off on large references: [evidence span withheld — anchor status: external_doc]
