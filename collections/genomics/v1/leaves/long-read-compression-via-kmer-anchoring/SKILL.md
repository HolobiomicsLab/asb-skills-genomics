---
name: long-read-compression-via-kmer-anchoring
description: 'Use when hybrid-assembling genomes with DBG2OLC: raw long 3GS reads
  must be converted to compact contig-identifier lists using accurate short-read DBG
  contigs before overlap, layout, and consensus.'
when_to_use_negative:
- The contigs are polished, gap-closed, or scaffolded rather than raw DBG contigs
  — the README warns this yields poor final results due to heuristics introduced by
  short-read assembly pipelines
- Only NGS short reads are available and no long reads exist to compress — the non-hybrid
  NGS assembly mode applies instead
- Long reads are already base-corrected or replaced by error-corrected overlaps (e.g.,
  PacBioToCA/ECTools/MHAP/HGAP/Falcon correction pipelines) — DBG2OLC's design explicitly
  skips base-level errors, so pre-correction is unnecessary and the pipeline should
  not be mixed with other correction tools
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Builds the 17-mer index over contigs, anchors long reads via AdaptiveTh/KmerCovTh
    thresholds, removes chimeras, scores compressed-read overlaps with MinOverlap,
    and emits compressed reads plus the layout backbone
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles 50x Illumina short reads into raw DBG contigs (Contigs.txt) that
    serve as the k-mer anchors for compressing 3GS reads
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads extracts the read subset (e.g., 20x PacBio) used as the
    long-read input; AssemblyStatistics evaluates assembly contigs
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:ad5d57a114a33e33a9157514c921a35b2f3acfc24f2767656902fd3281ab89da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# long-read-compression-via-kmer-anchoring

## Summary

Compress erroneous 3GS long reads (e.g., 20x PacBio FASTA) into ordered lists of contig identifiers by k-mer anchoring against preassembled NGS contigs, giving roughly a threefold read-length reduction that enables fast overlap/layout in DBG2OLC.

## When to use

Use when hybrid-assembling genomes with DBG2OLC: raw long 3GS reads must be converted to compact contig-identifier lists using accurate short-read DBG contigs before overlap, layout, and consensus.

## When NOT to use

- The contigs are polished, gap-closed, or scaffolded rather than raw DBG contigs — the README warns this yields poor final results due to heuristics introduced by short-read assembly pipelines
- Only NGS short reads are available and no long reads exist to compress — the non-hybrid NGS assembly mode applies instead
- Long reads are already base-corrected or replaced by error-corrected overlaps (e.g., PacBioToCA/ECTools/MHAP/HGAP/Falcon correction pipelines) — DBG2OLC's design explicitly skips base-level errors, so pre-correction is unnecessary and the pipeline should not be mixed with other correction tools

## Inputs

- NGS contig file in FASTA format from an accurate DBG assembler (Contigs.txt from SparseAssembler)
- 3GS long reads in FASTA format (e.g., Pacbio_20x.fasta selected from Pacbio.fasta)

## Outputs

- Per-read ordered contig-identifier lists (compressed reads, ReadsInfoFrom_* format), roughly threefold shorter than the original reads
- Assembly backbone (backbone_raw.fasta; N50 ~583 kbp in the reference S.cer w303 run)
- DBG2OLC_Consensus_info.txt mapping file for downstream consensus

## How to apply

First preassemble 50x Illumina reads into raw de Bruijn contigs with SparseAssembler (e.g., k 51, g 15, NodeCovTh 1, EdgeCovTh 0), avoiding gap closing or scaffolding, and keep the output Contigs.txt. Build a 17-mer index over the contigs with DBG2OLC (k 17), excluding k-mers shared by multiple contigs to avoid ambiguous anchors, then anchor each long read using AdaptiveTh 0.0001 (or 0.001–0.02 depending on coverage) and KmerCovTh 2: a contig with M matched k-mers where M < AdaptiveTh*Contig_Length or M < KmerCovTh cannot anchor the read. Score overlaps between compressed reads with MinOverlap 20 (10–30 for 10x/20x PacBio) and set RemoveChimera 1 when coverage exceeds 10x. The output compressed reads (ReadsInfoFrom_* files) and backbone_raw.fasta layout are checked before proceeding to BLASR/Sparc consensus.

## Related tools

- **DBG2OLC** (Builds the 17-mer index over contigs, anchors long reads via AdaptiveTh/KmerCovTh thresholds, removes chimeras, scores compressed-read overlaps with MinOverlap, and emits compressed reads plus the layout backbone) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles 50x Illumina short reads into raw DBG contigs (Contigs.txt) that serve as the k-mer anchors for compressing 3GS reads) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads extracts the read subset (e.g., 20x PacBio) used as the long-read input; AssemblyStatistics evaluates assembly contigs) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- The compressed-reads file exists in the ReadsInfoFrom_* format and each read is an ordered identifier list roughly threefold shorter than the original long read
- N50 of backbone_raw.fasta is acceptable before proceeding to consensus (~583 kbp in the reference S.cer w303 test run); keep tuning otherwise
- Anchoring respects both thresholds: no contig anchors a read when M < AdaptiveTh*Contig_Length or M < KmerCovTh matched 17-mers
- Parameter settings fall within the suggested ranges for the coverage (10x/20x PacBio: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01)
- Contigs used are raw DBG contigs — gap closing/scaffolding steps were not applied before compression

## Limitations

- Multi-contig k-mers are excluded, so ambiguous anchors cannot be used; at ~100x coverage additional ChimeraTh/ContigTh (set to 2) are needed to remove problematic reads and false contig anchors
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many such tools introduce errors; the prescribed pipeline order should be followed
- The first SparseAssembler/DBG2OLC run may not give the best result — LD 1 can reload computations and AdaptiveTh/KmerCovTh/MinOverlap may need fine-tuning

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read.: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] In the following example, the first 20x PacBio reads are extracted from the abovementioned file and we can assemble with:: "In the following example, the first 20x PacBio reads are extracted from the abovementioned file and we can assemble with:"
- [abstract] Compact representation of the long reads leads to efficient alignments.: "(i) Compact representation of the long reads leads to efficient alignments."
- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
