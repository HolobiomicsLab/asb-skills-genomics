---
name: long-read-alignment-benchmarking
description: Use when you need to quantitatively compare long-read mappers on synthetic
  Illumina/PacBio/ONT reads whose true genomic origins are known, using precision/recall
  of mapping location and per-base accuracy.
when_to_use_negative:
- Input reads are real (non-simulated) data with no ground-truth mapping locations
  — location precision/recall cannot be scored without true origins
- The goal is short-read variant calling or downstream consensus/variant analysis
  rather than mapper comparison
- Reference genomes far smaller than memory constraints are not the issue here, but
  if the machine lacks the ~100 GB RAM GraphMap needs for the human genome index,
  GraphMap-based benchmarks cannot be run as specified
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Mapper under test; compiled from source and used to align all simulated read
    sets
  repo: https://github.com/isovic/graphmap
- name: BLAST
  role: Comparator mapper providing BLAST-like sensitivity baseline
- name: BWA-MEM
  role: Comparator mapper for long and short reads
- name: LAST
  role: Comparator mapper
- name: BLASR
  role: Comparator mapper (PacBio-oriented)
- name: DALIGNER
  role: Comparator mapper
- name: ART
  role: Simulates synthetic Illumina read sets with platform error profiles
- name: PBSIM
  role: Simulates PacBio and ONT 1D/2D read sets with platform-specific error profiles
- name: graphmap-reproduce
  role: Scripts to reproduce the paper's benchmark results; installs evaluation tools
    and aligneval wrappers
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:72066972d172062ae99abf1d2017ef91dfe503f2b9d407e336163b05911b345a
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
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# long-read-alignment-benchmarking

## Summary

Benchmark read mappers (GraphMap, BLAST, BWA-MEM, LAST, BLASR, DALIGNER) on simulated long reads from diverse sequencing technologies by scoring mapping-location precision and recall against true simulated origins, plus per-base alignment accuracy.

## When to use

Use when you need to quantitatively compare long-read mappers on synthetic Illumina/PacBio/ONT reads whose true genomic origins are known, using precision/recall of mapping location and per-base accuracy.

## When NOT to use

- Input reads are real (non-simulated) data with no ground-truth mapping locations — location precision/recall cannot be scored without true origins
- The goal is short-read variant calling or downstream consensus/variant analysis rather than mapper comparison
- Reference genomes far smaller than memory constraints are not the issue here, but if the machine lacks the ~100 GB RAM GraphMap needs for the human genome index, GraphMap-based benchmarks cannot be run as specified

## Inputs

- Reference genome FASTA (e.g. human genome)
- Simulated Illumina read sets generated with ART
- Simulated PacBio and ONT 1D/2D read sets generated with PBSIM
- Ground-truth simulated read origins (positions) for precision/recall scoring
- GraphMap source repository for compilation

## Outputs

- Per-mapper mapping-location precision and recall values
- Per-base alignment accuracy per mapper (~70% expected)
- Fig. 2-style precision/recall comparison plot across technologies
- Supplementary Data 1-style summary table

## How to apply

Compile GraphMap from source (GCC/G++ >= 4.7: 'make modules' then 'make'). Generate synthetic read sets that capture platform-specific error profiles — Illumina reads with ART, PacBio and ONT 1D/2D reads with PBSIM — using the error models described in the paper's Methods. Align every simulated set to the reference genome with GraphMap, BLAST, BWA-MEM, LAST, BLASR and DALIGNER under comparable settings. Score each mapper by comparing mapped positions to the true simulated read origins to compute mapping-location precision and recall, then compute per-base alignment accuracy (the paper expects ~70% correct per-base alignments even while location precision/recall stay above 94%). Aggregate results into a Fig. 2-style comparison plot and a Supplementary Data 1-style table. Evaluation helpers are available in the isovic/graphmap-reproduce and aligneval repositories, which wrap each aligner for evaluation.

## Related tools

- **GraphMap** (Mapper under test; compiled from source and used to align all simulated read sets) — https://github.com/isovic/graphmap
- **BLAST** (Comparator mapper providing BLAST-like sensitivity baseline)
- **BWA-MEM** (Comparator mapper for long and short reads)
- **LAST** (Comparator mapper)
- **BLASR** (Comparator mapper (PacBio-oriented))
- **DALIGNER** (Comparator mapper)
- **ART** (Simulates synthetic Illumina read sets with platform error profiles)
- **PBSIM** (Simulates PacBio and ONT 1D/2D read sets with platform-specific error profiles)
- **graphmap-reproduce** (Scripts to reproduce the paper's benchmark results; installs evaluation tools and aligneval wrappers) — https://github.com/isovic/graphmap-reproduce

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- GraphMap mapping-location precision and recall above 94% on all simulated technology-specific read sets, including high-error-rate ONT reads
- Per-base alignment accuracy around 70% correct per-base alignments
- GraphMap location precision typically >98% and ability to distinguish candidate locations >94% identical on the human genome
- GraphMap mapping sensitivity exceeds comparators, with >95% of bases mapped and a 10–80% increase in mapped bases versus other mappers
- Results reproduce the Fig. 2-style plot and Supplementary Data 1-style table when run via graphmap-reproduce scripts

## Limitations

- GraphMap's hash-table index requires large-memory machines (~100 GB for the human genome; ~14x reference size, ~42 GB peak during construction); FM-index memory reduction was 'currently under implementation'
- Percentile seed filtering (--freq-percentile 0.99) skips 1% of most repetitive seeds and slightly reduces sensitivity, though it greatly helps on large references; use -x sensitive / --double-index to disable
- Benchmark results depend on the simulated error profiles used; results may not transfer to runs with different error characteristics
- Transcriptome mapping had a recall drop fixed only in later versions (v0.5.2); the spliced-alignment mode for RNA-seq was still experimental

## Evidence

- [results] GraphMap reproduces the reported benchmark, with mapping-location precision/recall above 94% and per-base alignment accuracy around 70%: [evidence span withheld — anchor status: not_found]
- [methods] Compile GraphMap from the source repository with GCC/G++ >= 4.7: [evidence span withheld — anchor status: not_found]
- [discussion] GraphMap was the only mapper that exhibited BLAST-like sensitivity, while being orders of magnitude faster than BLAST: "GraphMap was the only mapper that exhibited BLAST-like sensitivity, while being orders of magnitude faster than BLAST"
- [readme] After running `git pull`, please `make modules` before `make`.: "**Note:** After running `git pull`, please `make modules` before `make`."
- [readme] The number of hits for each seed lookup is now thresholded by a percentile value: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped."
