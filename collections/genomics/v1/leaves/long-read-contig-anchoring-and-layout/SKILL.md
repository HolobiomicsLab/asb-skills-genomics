---
name: long-read-contig-anchoring-and-layout
description: Use when you need to scaffold/assemble a genome from high-error long
  reads (e.g. ONT MinION up to ~40% error, PacBio) by combining them with accurate
  Illumina DBG contigs, producing a backbone for later consensus polishing.
when_to_use_negative:
- The input contigs are already scaffolded or gap-closed — the README warns this introduces
  errors from short-read assembly heuristics and yields poor final results
- You want to interleave DBG2OLC with other assemblers or error-correction tools —
  the README states combining DBG2OLC with other software may produce worse results
  because many such tools introduce errors
- Base-level accuracy of the long reads is the target and no accurate short-read contigs
  are available — this skill skips base-level errors and requires DBG contigs as anchors;
  polishing is a separate Sparc/BLASR consensus step
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Anchors DBG contigs to long erroneous reads via k-mer matching, performs overlap-layout
    of compressed reads, and emits backbone_raw.fasta plus DBG2OLC_Consensus_info.txt
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles ~50x Illumina short reads into raw de Bruijn graph contigs (Contigs.txt)
    that serve as the accurate anchors
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Provides SelectLongestReads for subsetting long-read coverage and AssemblyStatistics
    for checking backbone N50 before consensus
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:936e077f0b90ba5e7ebe9466b62a718a40d3ef1759c8bc015576f9aa101e89bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# long-read-contig-anchoring-and-layout

## Summary

Compress noisy third-generation long reads (PacBio/ONT MinION) by anchoring accurate de Bruijn graph contigs to them via k-mer matching, then overlap-layout the compressed reads to build an assembly backbone, as implemented in DBG2OLC.

## When to use

Use when you need to scaffold/assemble a genome from high-error long reads (e.g. ONT MinION up to ~40% error, PacBio) by combining them with accurate Illumina DBG contigs, producing a backbone for later consensus polishing.

## When NOT to use

- The input contigs are already scaffolded or gap-closed — the README warns this introduces errors from short-read assembly heuristics and yields poor final results
- You want to interleave DBG2OLC with other assemblers or error-correction tools — the README states combining DBG2OLC with other software may produce worse results because many such tools introduce errors
- Base-level accuracy of the long reads is the target and no accurate short-read contigs are available — this skill skips base-level errors and requires DBG contigs as anchors; polishing is a separate Sparc/BLASR consensus step

## Inputs

- Raw DBG contigs file in FASTA format (Contigs.txt) from SparseAssembler, without scaffolding/gap closing
- Third-generation long reads in FASTA format (PacBio or Oxford Nanopore MinION)
- Optional: read-selection settings via SelectLongestReads (AssemblyUtility) to choose a coverage subset, e.g. sum 260000000 longest 0

## Outputs

- backbone_raw.fasta — the compressed long-read overlap-layout backbone
- DBG2OLC_Consensus_info.txt — contig-to-backread placement info used by the Sparc consensus step

## How to apply

First preassemble ~50x Illumina reads into raw (non-scaffolded) DBG contigs with SparseAssembler (e.g. k 51, g 15, NodeCovTh 1, EdgeCovTh 0), outputting Contigs.txt — raw contigs only, since scaffolding heuristics introduce errors. Then run DBG2OLC with the contig file and the long-read FASTA (k 17, AdaptiveTh, KmerCovTh, MinOverlap, RemoveChimera 1): a contig anchors to a long read only if matched k-mers M pass both the adaptive threshold (M >= AdaptiveTh * Contig_Length) and the fixed threshold (M >= KmerCovTh); long-read overlaps are scored by matching k-mers of the compressed reads and pairs below MinOverlap are rejected. Tune per coverage: for 10x/20x data use KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01; for 50x-100x use KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02. Before proceeding to consensus, check the N50 of the produced backbone_raw.fasta with AssemblyStatistics; if unsatisfactory, retune parameters (optionally reloading with LD 1) rather than continuing.

## Related tools

- **DBG2OLC** (Anchors DBG contigs to long erroneous reads via k-mer matching, performs overlap-layout of compressed reads, and emits backbone_raw.fasta plus DBG2OLC_Consensus_info.txt) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles ~50x Illumina short reads into raw de Bruijn graph contigs (Contigs.txt) that serve as the accurate anchors) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Provides SelectLongestReads for subsetting long-read coverage and AssemblyStatistics for checking backbone N50 before consensus) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.005 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f pb_reads.fasta
```

## Evaluation signals

- backbone_raw.fasta exists with a non-trivial N50 — check with './AssemblyStatistics contigs backbone_raw.fasta' and only proceed to consensus if you are satisfied, otherwise keep tuning
- On the E. coli Oxford Nanopore MinION test, the pipeline (with consensus) should resolve to a single contig with a polished error rate near 0.23%
- The S.cer w303 20x PacBio reference run reports a backbone N50 of ~583 kbp with k 17, KmerCovTh 2, MinOverlap 20, AdaptiveTh 0.0001
- Chimeric long reads are flagged: RemoveChimera 1 is suggested whenever long-read coverage exceeds 10x
- Every contig used as an anchor satisfies both M >= AdaptiveTh * Contig_Length and M >= KmerCovTh matched k-mers to its long read

## Limitations

- High-error ONT data is tolerated only because base-level errors are skipped during anchoring; the backbone itself remains unpolished until the BLASR/Sparc consensus step (~90% of total runtime is consensus)
- Parameter tuning may be needed: the first run may not give the best result, and AdaptiveTh/KmerCovTh/MinOverlap suggested ranges differ by long-read coverage (10-20x vs 50-100x)
- At ~100x coverage, extra parameters ChimeraTh and ContigTh should be raised to 2 to remove problematic reads and false contig anchors
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many existing tools introduce errors

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding. Otherwise you may have poor final results"
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [abstract] Compact representation of the long reads leads to efficient alignments. (ii) Base-level errors can be skipped; structural errors need to be detected and corrected.: "(i) Compact representation of the long reads leads to efficient alignments. (ii) Base-level errors can be skipped; structural errors need to be detected and corrected."
- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
