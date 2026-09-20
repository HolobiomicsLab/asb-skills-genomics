---
name: long-read-overlap-scoring
description: Use when assembling large genomes from long erroneous 3GS reads (PacBio/ONT)
  hybridized with accurate DBG contigs, and you need to find overlapping read pairs
  without all-vs-all base-level alignment.
when_to_use_negative:
- Input reads are already error-corrected base-level overlaps (e.g. from PacBioToCA/MHAP/HGAP)
  — this skill scores compressed contig identifiers, not base-level alignments
- Short-read contigs come from repeat-resolved assemblies (gap closing/scaffolding),
  which the README warns introduces errors and yields poor final results
- Coverage is below 10x, where RemoveChimera is not suggested and anchor thresholds
  may not be met
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DBG2OLC
  role: Builds the contig-to-read index, applies AdaptiveTh/KmerCovTh anchor filters,
    scores pair-wise overlaps of compressed reads against MinOverlap, and writes backbone_raw.fasta
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles ~50x Illumina reads into the raw DBG contigs whose identifiers
    compress the long reads for overlap scoring
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads prepares the long-read FASTA subset before overlap scoring
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:e49e6f4275b0a61fae835d8b91047aa87d907b3cc8149873e82658cff47c6e3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# long-read-overlap-scoring

## Summary

Efficiently identifies overlapping long reads in DBG2OLC by anchoring compressed reads to short-read contigs via k-mer matching thresholds (AdaptiveTh, KmerCovTh) and scoring pair-wise overlaps by aligning compressed-read identifier sequences with a MinOverlap cutoff, avoiding all-vs-all base-level alignment.

## When to use

Use when assembling large genomes from long erroneous 3GS reads (PacBio/ONT) hybridized with accurate DBG contigs, and you need to find overlapping read pairs without all-vs-all base-level alignment.

## When NOT to use

- Input reads are already error-corrected base-level overlaps (e.g. from PacBioToCA/MHAP/HGAP) — this skill scores compressed contig identifiers, not base-level alignments
- Short-read contigs come from repeat-resolved assemblies (gap closing/scaffolding), which the README warns introduces errors and yields poor final results
- Coverage is below 10x, where RemoveChimera is not suggested and anchor thresholds may not be met

## Inputs

- Compressed-reads file (ReadsInfoFrom_*) containing contig-identifier sequences per long read
- NGS contig file in FASTA format from SparseAssembler (Contigs.txt)
- Long reads in FASTA format (e.g. Pacbio_20x.fasta)

## Outputs

- Anchored/cleaned compressed reads (loadable with LD 1)
- Overlap-scored read pairs retained above MinOverlap (e.g. candidate_overlaps.tsv with pair and score fields)
- backbone_raw.fasta layout backbone with DBG2OLC_Consensus_info.txt

## How to apply

First compress each long read into a sequence of short-read contig identifiers (from SparseAssembler Contigs.txt) and build an inverted index mapping each contig ID to the reads containing it. A contig anchors a long read only if the number of matched k-mers M passes both thresholds: M >= AdaptiveTh * Contig_Length (e.g. 0.0001-0.01 for 10x-20x PacBio) and M >= KmerCovTh (e.g. 2). For each candidate read pair, align their compressed identifier sequences and score the overlap with the matching k-mers, keeping pairs with overlap score >= MinOverlap (10-30 for 10x/20x PacBio; 50-150 for 50x-100x). Apply RemoveChimera 1 to drop chimeric links when coverage exceeds 10x, then chain the surviving best overlaps into the layout backbone. Retune KmerCovTh/MinOverlap/AdaptiveTh if the N50 of backbone_raw.fasta is unsatisfactory before calling consensus with BLASR/Sparc.

## Related tools

- **DBG2OLC** (Builds the contig-to-read index, applies AdaptiveTh/KmerCovTh anchor filters, scores pair-wise overlaps of compressed reads against MinOverlap, and writes backbone_raw.fasta) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles ~50x Illumina reads into the raw DBG contigs whose identifiers compress the long reads for overlap scoring) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads prepares the long-read FASTA subset before overlap scoring) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- Every retained read pair has an overlap score >= MinOverlap (e.g. 20 for 20x PacBio w303 data)
- Each anchored contig satisfies both M >= AdaptiveTh * Contig_Length and M >= KmerCovTh
- backbone_raw.fasta exists with positive node/edge counts consistent with the cleaned-read set
- N50 of backbone_raw.fasta meets expectations (e.g. 583 kbp in the README w303 test run); otherwise retune KmerCovTh/MinOverlap/AdaptiveTh before proceeding to consensus
- Chimeric read links are dropped when RemoveChimera 1 is set (>10x coverage)

## Limitations

- Merging DBG2OLC with other assembly/error-correction tools can produce worse results because such tools introduce errors; follow the prescribed hybrid steps first
- Parameter tuning is coverage-dependent: 10x/20x PacBio uses KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01, while 50x-100x uses KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02
- At ~100x coverage, ChimeraTh and ContigTh must be raised to 2 to remove problematic reads and false contig anchors in the multiple alignment
- k is fixed at 17 as the value that works well for the compressed-read overlap scoring

## Evidence

- [methods] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read.: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [abstract] Compact representation of the long reads leads to efficient alignments.: "Compact representation of the long reads leads to efficient alignments."
