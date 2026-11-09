---
name: structural-variant-detection-from-spanning-alignments
description: Use when long, error-prone nanopore reads are aligned to a reference
  carrying known structural variants (e.g. Mutatrix-mutated E. coli K-12 with 20 induced
  SVs, 100 bp–4 kbp) and SV calls must be extracted from spanning alignments and scored
  for recall/precision.
when_to_use_negative:
- 'Reference is a large genome (e.g. human): GraphMap''s hash index is ~14x reference
  size with ~42 GB peak index construction, and the paper notes ~100 GB memory for
  the human genome — unsuitable for memory-constrained machines'
- 'Goal is calling point SNVs rather than 100 bp–4 kbp indels: the paper''s SV findings
  concern spanning alignments of indels in that size range, and SNV calling was assessed
  separately via marginAlign'
- 'Reads are short Illumina reads: GraphMap is targeted at long, error-prone third-generation
  reads (MinION 1d/2d, PacBio)'
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Primary mapper producing gapped spanning alignments from which SV calls (100
    bp–4 kbp indels) are directly extracted
  repo: https://github.com/isovic/graphmap
- name: BLASR
  role: Comparison mapper re-run on the same reads; next best mapper on the SV detection
    benchmark
- name: BWA-MEM
  role: Comparison mapper; its alignments require LUMPY post-processing before event
    extraction
- name: Mutatrix
  role: Generates the mutated reference (VCF of variants + mutated FASTA copies) used
    as ground truth
- name: graphmap-reproduce
  role: 'Benchmark scripts/data: run-realdata.py obtains the Mutatrix-mutated E. coli
    reference and matching real MinION read set and runs the tests'
- name: MinION
  role: Instrument providing the real 2D nanopore reads aligned against the mutated
    reference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:3d161d36d50de166ecdfc21561667f10f3ff00d8228376c1e5a1499f46d65417
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
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# structural-variant-detection-from-spanning-alignments

## Summary

Detect structural variants (insertions/deletions of 100 bp–4 kbp) by extracting spanning or split long-read alignments from GraphMap mappings of nanopore (2D MinION) reads, and benchmark mapper recall/precision against a known set of induced SVs.

## When to use

Use when long, error-prone nanopore reads are aligned to a reference carrying known structural variants (e.g. Mutatrix-mutated E. coli K-12 with 20 induced SVs, 100 bp–4 kbp) and SV calls must be extracted from spanning alignments and scored for recall/precision.

## When NOT to use

- Reference is a large genome (e.g. human): GraphMap's hash index is ~14x reference size with ~42 GB peak index construction, and the paper notes ~100 GB memory for the human genome — unsuitable for memory-constrained machines
- Goal is calling point SNVs rather than 100 bp–4 kbp indels: the paper's SV findings concern spanning alignments of indels in that size range, and SNV calling was assessed separately via marginAlign
- Reads are short Illumina reads: GraphMap is targeted at long, error-prone third-generation reads (MinION 1d/2d, PacBio)

## Inputs

- Mutatrix-mutated reference FASTA (e.g. E. coli K-12 with 20 induced SVs, 100 bp–4 kbp)
- Real Oxford Nanopore 2D MinION reads (FASTA/FASTQ) matching the mutated reference
- VCF of induced variants from Mutatrix as the ground-truth SV set
- GraphMap, BLASR, and BWA-MEM alignments (SAM) of the reads to the mutated reference

## Outputs

- Per-mapper SV calls extracted from spanning/split alignments across the 20 induced events
- Per-mapper recall and precision metrics
- sv_detection_results.tsv (per-mapper calls and metrics)

## How to apply

Generate a mutated reference with Mutatrix (e.g. E. coli K-12 with 20 induced insertions/deletions spanning 100 bp–4 kbp) and obtain the matching real 2D MinION read set from the graphmap-reproduce benchmark. Align reads with GraphMap using gapped alignments that span SV events directly — GraphMap's design lets spanning alignments cover the full event so detection needs neither assembly nor split-read analysis. For comparison, re-align with BLASR and with BWA-MEM (applying LUMPY post-processing to the BWA-MEM alignments before event extraction). Score each mapper's calls against the known 20 induced events to compute recall and precision, and write per-mapper calls plus metrics to sv_detection_results.tsv. Expect GraphMap to achieve perfect precision on this benchmark with roughly 35% better recall than the next best mapper (BLASR).

## Related tools

- **GraphMap** (Primary mapper producing gapped spanning alignments from which SV calls (100 bp–4 kbp indels) are directly extracted) — https://github.com/isovic/graphmap
- **BLASR** (Comparison mapper re-run on the same reads; next best mapper on the SV detection benchmark)
- **BWA-MEM** (Comparison mapper; its alignments require LUMPY post-processing before event extraction)
- **Mutatrix** (Generates the mutated reference (VCF of variants + mutated FASTA copies) used as ground truth)
- **graphmap-reproduce** (Benchmark scripts/data: run-realdata.py obtains the Mutatrix-mutated E. coli reference and matching real MinION read set and runs the tests)
- **MinION** (Instrument providing the real 2D nanopore reads aligned against the mutated reference)

## Examples

```
./bin/Linux-x64/graphmap align -r mutated_reference.fa -d ecoli_2d_reads.fasta -o graphmap_sv.sam
```

## Evaluation signals

- Number of extracted SV calls matches the 20 induced events (100 bp–4 kbp) in the Mutatrix VCF
- GraphMap spanning alignments should yield perfect precision on the benchmark SV set
- GraphMap recall should exceed the next best mapper (BLASR) by ~35% on this benchmark
- Output file sv_detection_results.tsv contains per-mapper calls plus recall/precision metrics
- BWA-MEM-based calls should only be produced after LUMPY post-processing of its alignments

## Limitations

- GraphMap requires large-memory machines (~100 GB for the human genome); FM-index-based memory reduction was only 'currently under implementation'
- GraphMap's hash-table index is ~14x reference size (~42 GB peak for human); larger minimizer windows (--minimizer-window) shrink it but probably reduce sensitivity
- Seed percentile filtering (--freq-percentile 0.99) slightly reduces sensitivity; use -x sensitive / --double-index to recover pre-0.5.1 sensitivity
- No custom nanopore variant callers were available in the study — SV/SNV calling relied on inspecting mappings directly or existing downstream tools

## Evidence

- [abstract] GraphMap enables precise detection of structural variants from 100 bp to 4 kbp: "precise detection of structural variants from length 100 bp to 4 kbp"
- [discussion] Spanning alignments allow direct SV detection without assembly or split-read analysis: "readily spanned insertions and deletions over a wide range of sizes (100 bp–4 kbp) allowing for the direct detection of such events"
- [discussion] No custom nanopore variant callers existed; mappings themselves were the calling basis: "Despite the lack of custom variant callers, read mappings from GraphMap were shown to provide sensitive and precise SNV calls"
- [readme] Index size constraint relevant to scaling this SV detection beyond small genomes: [evidence span withheld — anchor status: not_found]
- [readme] Seed percentile filter slightly reduces sensitivity: [evidence span withheld — anchor status: external_doc]
- [readme] Basic GraphMap alignment invocation used to generate the SV-spanning alignments: [evidence span withheld — anchor status: external_doc]
