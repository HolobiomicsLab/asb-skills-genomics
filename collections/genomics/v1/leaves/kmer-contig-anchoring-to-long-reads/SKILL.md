---
name: kmer-contig-anchoring-to-long-reads
description: 'Use when assembling a genome from hybrid data: accurate short-read contigs
  (FASTA from an DBG-assembler) plus low-coverage (10x-20x) erroneous long reads,
  and a high-contiguity backbone (e.g. N50 583 kbp on yeast) is needed.'
when_to_use_negative:
- You already have high-quality long reads and plan a non-hybrid pipeline (e.g. HGAP/Falcon)
  — the article compares against these, and hybrid preassembly heuristics may not
  help
- Your short-read contigs were produced with gap-closing or scaffolding heuristics
  — the README warns these errors propagate and give poor final results
- Coverage is very high (~100x PacBio) without adjusting ChimeraTh/ContigTh and the
  50x-100x parameter ranges (KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02)
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: 'Core tool: compresses long reads into k-mer space and anchors DBG contigs
    onto them for overlap and layout, producing backbone_raw.fasta'
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles the 50x Illumina reads into accurate raw DBG contigs (Contigs.txt)
    that serve as anchors
  repo: https://github.com/yechengxi/SparseAssembler
- name: BLASR
  role: Aligns all raw PacBio reads to the assembly backbone (e.g. -minMatch 19, -m
    5) as input to the consensus step; consumes ~90% of runtime
- name: Sparc
  role: Sparsity-based consensus/polishing of the backbone using the BLASR m5 alignments
  repo: https://github.com/yechengxi/Sparc
- name: AssemblyUtility
  role: Provides SelectLongestReads to subset PacBio coverage (e.g. sum 260000000
    longest 0) and AssemblyStatistics to compute assembly N50
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:84a0e832b78d919753c9d3d6a9127ff525d690bf04e36ba720942c5f6857f01f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# k-mer contig anchoring to long reads

## Summary

Compresses long erroneous 3GS reads (e.g. PacBio) into k-mer space and anchors accurate preassembled DBG contigs onto them to build a genome backbone, skipping base-level errors while correcting structural errors. Achieves N50s >10x better than Illumina alone from as little as 10x-20x PacBio coverage.

## When to use

Use when assembling a genome from hybrid data: accurate short-read contigs (FASTA from an DBG-assembler) plus low-coverage (10x-20x) erroneous long reads, and a high-contiguity backbone (e.g. N50 583 kbp on yeast) is needed.

## When NOT to use

- You already have high-quality long reads and plan a non-hybrid pipeline (e.g. HGAP/Falcon) — the article compares against these, and hybrid preassembly heuristics may not help
- Your short-read contigs were produced with gap-closing or scaffolding heuristics — the README warns these errors propagate and give poor final results
- Coverage is very high (~100x PacBio) without adjusting ChimeraTh/ContigTh and the 50x-100x parameter ranges (KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02)

## Inputs

- Raw DBG contigs in FASTA format (Contigs.txt) preassembled from ~50x Illumina short reads (e.g. 2x300 bp FASTQ)
- 10x-20x coverage long erroneous reads in FASTA format (e.g. Pacbio_20x.fasta from the w303 Pacbio.fasta)
- Estimated genome size (e.g. GS 12000000 bp for the 12 Mbp yeast genome)

## Outputs

- backbone_raw.fasta — structurally correct assembly backbone from DBG2OLC overlap/layout
- DBG2OLC_Consensus_info.txt — read-to-backbone layout info for the consensus step
- Polished consensus assembly after Sparc, evaluated for N50/NG50 with AssemblyStatistics or QUAST 3.0

## How to apply

First preassemble ~50x Illumina reads with SparseAssembler (e.g. k 51, g 15, NodeCovTh 1, EdgeCovTh 0, GS 12000000 for the S.cer w303 dataset) into raw DBG contigs (Contigs.txt), explicitly avoiding gap-closing/scaffolding heuristics that introduce errors. Run DBG2OLC with k 17, AdaptiveTh 0.0001, KmerCovTh 2, MinOverlap 20, RemoveChimera 1 on the 20x PacBio reads; a contig is only anchored to a long read if matched k-mers M pass both the adaptive threshold (M >= AdaptiveTh * Contig_Length) and the fixed KmerCovTh, and long-read pairs need overlap scores >= MinOverlap. Check backbone_raw.fasta N50 (583 kbp in the w303 test); if unsatisfactory, retune within suggested ranges before proceeding — for 10x/20x PacBio: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01. Finally, concatenate contigs and PacBio reads, run BLASR-based Sparc consensus polishing (split_and_run_sparc.sh with ulimit -n unlimited) and evaluate the final N50 with AssemblyStatistics or QUAST 3.0.

## Related tools

- **DBG2OLC** (Core tool: compresses long reads into k-mer space and anchors DBG contigs onto them for overlap and layout, producing backbone_raw.fasta) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles the 50x Illumina reads into accurate raw DBG contigs (Contigs.txt) that serve as anchors) — https://github.com/yechengxi/SparseAssembler
- **BLASR** (Aligns all raw PacBio reads to the assembly backbone (e.g. -minMatch 19, -m 5) as input to the consensus step; consumes ~90% of runtime)
- **Sparc** (Sparsity-based consensus/polishing of the backbone using the BLASR m5 alignments) — https://github.com/yechengxi/Sparc
- **AssemblyUtility** (Provides SelectLongestReads to subset PacBio coverage (e.g. sum 260000000 longest 0) and AssemblyStatistics to compute assembly N50) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f Pacbio_20x.fasta
```

## Evaluation signals

- Backbone N50 of backbone_raw.fasta reaches the reference scale (583 kbp on the 20x PacBio S.cer w303 test); if not, retune KmerCovTh/MinOverlap/AdaptiveTh before proceeding to consensus
- Final assembly N50 evaluated with AssemblyStatistics or QUAST 3.0 in default settings is at least 10x better than the Illumina-only assembly
- Nearly perfect diagonal dot plots against a reference indicate a structurally correct assembly from as low as 10x long-read coverage
- Illumina contigs are raw DBG contigs (no gap-closing/scaffolding) — mixing in error-correcting/scaffolding tools should be avoided per the README

## Limitations

- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many existing tools introduce errors
- Parameter tuning is required per genome/coverage; the first run may not yield the best backbone, and the README advises checking N50 of backbone_raw.fasta and not proceeding until satisfied
- RemoveChimera 1 is only suggested at >10x coverage; at ~100x coverage ChimeraTh/ContigTh must be raised to 2 or problematic reads and false contig anchors leak into the layout
- ~90% of computation time is spent on the BLASR consensus step, which dominates wall-clock cost for large genomes

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] It is important to follow the steps before you invent your own pipeline. As combining DBG2OLC with other software tools may produce worse results.: "It is important to follow the steps before you invent your own pipeline. As combining DBG2OLC with other software tools may produce worse results."
- [results] Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone: "Using 10x–20x coverage of PacBio sequence data, we obtained assembly N50s that are significantly (>10x) better than Illumina data alone"
- [readme] In our test run, the N50 is 583kbp.: "In our test run, the N50 is 583kbp."
