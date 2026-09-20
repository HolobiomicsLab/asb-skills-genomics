---
name: nanopore-read-alignment
description: Use when aligning Oxford Nanopore MinION 1d/2d reads (or PacBio reads)
  to a reference genome with high sensitivity despite high error rates, or when downstream
  SV/SNV calling requires gapped alignments that span 100 bp–4 kbp insertion/deletion
  events directly.
when_to_use_negative:
- Input reads are short Illumina reads on a small genome — the paper's advantage targets
  long, error-prone reads; short-read mappers may suffice.
- Alignment of RNA-seq spliced reads is required and the experimental spliced-alignment
  mode is not yet stable — the paper notes this mode is 'under way'.
- Reference is a very large genome (e.g. human) on a memory-limited machine — GraphMap's
  hash index needs ~14x reference size (~42 GB peak for human) and the FM-index reduction
  was 'currently under implementation'.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: 'Primary mapper: gapped spaced seed lookup, graph-based seed processing, LCSk
    chaining, and semiglobal/Gotoh alignment of MinION reads to a reference'
  repo: https://github.com/isovic/graphmap
- name: BWA-MEM
  role: Comparison mapper for benchmarking mapped bases and SV detection recall against
    GraphMap
- name: BLASR
  role: Comparison mapper (PacBio-oriented) used as the next-best baseline in SV detection
    benchmarks
- name: marginAlign
  role: Downstream realigner/variant caller whose SNV calling improves when seeded
    with GraphMap mappings
  repo: https://github.com/isovic/marginAlign
- name: Mutatrix
  role: Simulator that generates the mutated reference (induced SVs) used to build
    SV-detection benchmarks
  repo: https://github.com/ekg/mutatrix
- name: samscripts
  role: Companion scripts for working with GraphMap SAM output
  repo: https://github.com/isovic/samscripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:ae6d3e014bc9cc21116ef2b578d9ee18d08a54d78d8067933e399bcafad3fffc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLASR
  - BWA-MEM
  - GraphMap
  - MinION
  - Mutatrix
  - marginAlign
  - minimap2
  - samscripts
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- nanopore-read-alignment-with-minimap2
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# nanopore-read-alignment

## Summary

Align long, error-prone Oxford Nanopore MinION reads to a reference using GraphMap, a mapper built on gapped spaced seeds, graph-based seed processing, and LCSk anchor chaining that achieves >95% precision and 10–80% higher mapping sensitivity than BWA-MEM, LAST, BLASR, and DALIGNER on nanopore data.

## When to use

Use when aligning Oxford Nanopore MinION 1d/2d reads (or PacBio reads) to a reference genome with high sensitivity despite high error rates, or when downstream SV/SNV calling requires gapped alignments that span 100 bp–4 kbp insertion/deletion events directly.

## When NOT to use

- Input reads are short Illumina reads on a small genome — the paper's advantage targets long, error-prone reads; short-read mappers may suffice.
- Alignment of RNA-seq spliced reads is required and the experimental spliced-alignment mode is not yet stable — the paper notes this mode is 'under way'.
- Reference is a very large genome (e.g. human) on a memory-limited machine — GraphMap's hash index needs ~14x reference size (~42 GB peak for human) and the FM-index reduction was 'currently under implementation'.

## Inputs

- Oxford Nanopore MinION 1d or 2D read set in FASTA/FASTQ format
- Reference genome in FASTA format (e.g. Mutatrix-mutated E. coli K-12 with induced SVs)
- Optional GTF annotation file for transcriptome mapping mode

## Outputs

- SAM/BAM alignment file with per-read alignments, CIGAR strings, E-value and mapping-quality tags (ZE:f: as a valid float)
- Mapped-base fraction and alignment precision metrics per mapper for benchmark comparisons
- Per-mapper SV calls spanning 100 bp–4 kbp events for variant-detection evaluation

## How to apply

Clone and build GraphMap from github.com/isovic/graphmap, running `make modules` before `make` (after any `git pull`, `make modules` must be re-run). Index the reference FASTA and align the FASTQ reads with `graphmap align -r reference.fa -d reads.fastq -o out.sam`; default parameters handle MinION, PacBio, and Illumina error profiles without retuning. On references where memory is a concern, rely on the minimizer index (window size 5 by default, tunable via `--minimizer-window`) and the seed-hit percentile filter (`--freq-percentile 0.99` skips the 1% most repetitive seeds), or use `-x sensitive` to disable both and reproduce pre-0.5.1 sensitivity. For a fair baseline comparison, re-run the same read set with BWA-MEM, BLASR, LAST, or DALIGNER and compare mapped-base fractions and alignment precision. For SV benchmarking, inspect gapped alignments that span or split across induced events (e.g. Mutatrix-mutated E. coli K-12 with 20 SVs of 100 bp–4 kbp) and score calls against the known variant set for recall and precision.

## Related tools

- **GraphMap** (Primary mapper: gapped spaced seed lookup, graph-based seed processing, LCSk chaining, and semiglobal/Gotoh alignment of MinION reads to a reference) — https://github.com/isovic/graphmap
- **BWA-MEM** (Comparison mapper for benchmarking mapped bases and SV detection recall against GraphMap)
- **BLASR** (Comparison mapper (PacBio-oriented) used as the next-best baseline in SV detection benchmarks)
- **marginAlign** (Downstream realigner/variant caller whose SNV calling improves when seeded with GraphMap mappings) — https://github.com/isovic/marginAlign
- **Mutatrix** (Simulator that generates the mutated reference (induced SVs) used to build SV-detection benchmarks) — https://github.com/ekg/mutatrix
- **samscripts** (Companion scripts for working with GraphMap SAM output) — https://github.com/isovic/samscripts

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Mapped-base fraction should exceed ~95% of bases and >90% of reads on MinION data, with a 10–80% gain over other mappers
- Alignment precision should be >95% per the abstract; location precision typically >98% on the human genome
- In version ≥0.5.1, index construction memory should peak near the final index size rather than far above it (README regression fix)
- For SV benchmarks, recall/precision should be scored against the known set of 20 induced Mutatrix SVs (100 bp–4 kbp); GraphMap spanning alignments should capture these without split-read post-processing
- Unmapped reads must carry a valid float ZE:f: E-value tag (std::numeric_limits<float>::max()), not 'Inf' (v0.5.2 fix)

## Limitations

- The hash-table index is ~14x reference size; mapping the human genome peaks at ~42 GB during construction and ~100 GB was cited for large genomes in the paper
- Increasing the minimizer window (--minimizer-window) or enabling --freq-percentile filtering reduces index size but can drop sensitivity; use -x sensitive to recover previous behavior
- FM-index-based memory reduction was only 'currently under implementation' at publication time
- v0.5.1's percentile seed filtering slightly reduces sensitivity, though it plays a large role when mapping to large references
- SV detection here is direct inspection of spanning/split gapped alignments; custom nanopore variant callers were not available and downstream SNV calling still relies on external tools (marginAlign, Nanopolish)

## Evidence

- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%): "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] readily spanned insertions and deletions over a wide range of sizes (100 bp–4 kbp) allowing for the direct detection of such events: "readily spanned insertions and deletions over a wide range of sizes (100 bp–4 kbp) allowing for the direct detection of such events"
- [methods] make modules: [evidence span withheld — anchor status: not_found]
- [readme] After running `git pull`, please `make modules` before `make`.: "After running `git pull`, please `make modules` before `make`."
- [readme] graphmap align -r reference.fa -d reads.fastq -o output.sam: [evidence span withheld — anchor status: external_doc]
- [readme] the minimizer window size can be specified via command line and by default is equal to 5: "The index supports minimizers - the minimizer window size can be specified via command line and by default is equal to 5"
- [discussion] A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome): "A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
