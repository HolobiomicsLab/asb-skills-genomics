---
name: nanopore-read-mapping-benchmarking
description: Use when comparing long-read mapper sensitivity (mapped reads/bases)
  or consensus accuracy on Oxford Nanopore MinION data against published GraphMap
  results, or when validating a mapper build on real MinION data sets.
when_to_use_negative:
- The task is de novo assembly or read overlapping rather than mapping-to-reference
  benchmarking (use GraphMap's 'owler' overlapping mode instead)
- Data are simulated reads only — the sensitivity/consensus validation thresholds
  here are defined against real MinION data sets from the reproduction bundle
- Mapping to very large genomes on memory-limited machines — GraphMap's hash index
  requires ~14x reference size (~100 GB peak for human genome)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: Mapper under test; build with 'make modules' then 'make' (Linux release)
  repo: https://github.com/isovic/graphmap
- name: LAST
  role: Comparison mapper using recommended nanopore settings
- name: BWA-MEM
  role: Comparison mapper for mapped-bases sensitivity and E. coli consensus error
    baseline
- name: BLASR
  role: Comparison long-read mapper
- name: DALIGNER
  role: Comparison long-read mapper
- name: Nanopolish
  role: Downstream consensus/variant caller that benefits from GraphMap mappings
- name: MinION
  role: Oxford Nanopore instrument producing the benchmark read data sets
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:280f69a0f62f18949257be4ac38162a2a4f003888b346fb6fcd172b0a714d8bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLASR
  - BWA-MEM
  - DALIGNER
  - GCC
  - GraphMap
  - LAST
  - MinION
  - Nanopolish
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# nanopore-read-mapping-benchmarking

## Summary

Benchmark GraphMap against LAST, BWA-MEM, BLASR and DALIGNER on real MinION nanopore read sets by measuring mapped-read/base fractions and whole-genome consensus accuracy, reproducing the paper's Fig. 3 sensitivity and consensus-error claims.

## When to use

Use when comparing long-read mapper sensitivity (mapped reads/bases) or consensus accuracy on Oxford Nanopore MinION data against published GraphMap results, or when validating a mapper build on real MinION data sets.

## When NOT to use

- The task is de novo assembly or read overlapping rather than mapping-to-reference benchmarking (use GraphMap's 'owler' overlapping mode instead)
- Data are simulated reads only — the sensitivity/consensus validation thresholds here are defined against real MinION data sets from the reproduction bundle
- Mapping to very large genomes on memory-limited machines — GraphMap's hash index requires ~14x reference size (~100 GB peak for human genome)

## Inputs

- Eight published Oxford Nanopore MinION read data sets (FASTA/FASTQ) from isovic/graphmap-reproduce
- Reference genomes (FASTA) for each MinION data set
- Built GraphMap binary (bin/Linux-x64/graphmap) plus LAST, BWA-MEM, BLASR and DALIGNER executables
- SAM alignments from each mapper for each data set

## Outputs

- minion_mapping_consensus_benchmark.csv with per-mapper mapped-read fractions, mapped-bases percentages and E. coli consensus error counts
- Per-read and aggregate mapped-bases sensitivity comparisons against Fig. 3b-d published values

## How to apply

Retrieve the eight published MinION read data sets and their reference genomes from the isovic/graphmap-reproduce bundle and build GraphMap from source with 'make modules' followed by 'make' for the Linux release. Map every MinION data set against its reference with GraphMap, LAST, BWA-MEM, BLASR and DALIGNER using each tool's recommended nanopore settings, then compute per-mapper fractions of mapped reads and mapped bases per data set. For consensus accuracy, derive a whole-genome consensus from each mapper's E. coli alignments (~80x coverage) at a Q50 quality threshold and count consensus errors per mapper. Accept the reproduction when GraphMap maps >95% of bases with a 10-80% mapped-bases increase over the other mappers, and the E. coli Q50 consensus yields <40 errors for GraphMap versus >1,000 errors for LAST and BWA-MEM. Tabulate mapped-bases percentages and consensus error counts into minion_mapping_consensus_benchmark.csv and compare against the published Fig. 3b-d values.

## Related tools

- **GraphMap** (Mapper under test; build with 'make modules' then 'make' (Linux release)) — https://github.com/isovic/graphmap
- **LAST** (Comparison mapper using recommended nanopore settings)
- **BWA-MEM** (Comparison mapper for mapped-bases sensitivity and E. coli consensus error baseline)
- **BLASR** (Comparison long-read mapper)
- **DALIGNER** (Comparison long-read mapper)
- **Nanopolish** (Downstream consensus/variant caller that benefits from GraphMap mappings)
- **MinION** (Oxford Nanopore instrument producing the benchmark read data sets)

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fastq -o output.sam
```

## Evaluation signals

- GraphMap maps >95% of bases on average across the MinION data sets
- GraphMap's mapped-bases increase over LAST and the other mappers falls in the 10-80% range
- E. coli Q50 consensus at ~80x coverage yields <40 errors for GraphMap vs >1,000 errors for both LAST and BWA-MEM
- Benchmark table minion_mapping_consensus_benchmark.csv matches published Fig. 3b-d values
- GraphMap maps >90% of reads per data set (paper average)

## Limitations

- GraphMap's hash-based index is ~14x reference size (~42 GB peak for human genome); requires large-memory machines, and an FM-index memory reduction is only 'currently under implementation'
- The --freq-percentile 0.99 seed filtering slightly reduces sensitivity (it 'slightly reduces sensitivity, but plays a huge role when mapping to large references'); use -x sensitive / --double-index to disable
- Benefit for downstream consensus callers such as Nanopolish and PoreSeq is asserted ('should do so') but not demonstrated in the paper

## Evidence

- [readme] Build GraphMap from source: "git clone https://github.com/isovic/graphmap.git
cd graphmap
make modules
make"
- [abstract] 10-80% sensitivity gain and >95% bases mapped: "GraphMap increases mapping sensitivity by 10–80% and maps >95% of bases"
- [discussion] MinION mapped reads/bases averages: "GraphMap was unmatched in terms of sensitivity, mapping >90% of reads and 95% of bases on average. Compared with other mappers, this lead to a 10–80% increase in mapped bases"
- [discussion] Comparison mapper set: "is comparable to BLAST and other mappers (BWA-MEM, LAST, BLASR and DALIGNER)"
- [readme] Seed frequency filtering parameter: "the number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped"
- [readme] Index memory footprint: [evidence span withheld — anchor status: not_found]
