---
name: smith-waterman-compressed-read-alignment
description: Use when long reads have already been compressed into contig-identifier
  sequences and you must identify overlapping read pairs efficiently, avoiding all-against-all
  base-level alignment.
when_to_use_negative:
- Input long reads are raw, uncompressed sequences rather than compressed contig-identifier
  representations — the Smith-Waterman step operates on identifier sequences, not
  bases
- A base-level all-against-all read overlap computation is explicitly required (e.g.
  for non-hybrid long-read-only overlap graphs) rather than anchor-guided candidate
  selection
- Reads contain chimeric segments and RemoveChimera cannot be enabled (e.g. <10x coverage),
  so scored links cannot be trusted
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: DBG2OLC
  role: Implements the inverted index over compressed reads, anchor selection with
    AdaptiveTh/KmerCovTh, Smith-Waterman scoring of compressed-read pairs against
    MinOverlap, and RemoveChimera filtering
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate DBG contigs (Contigs.txt) that are compressed into the
    long reads used as the identifier alphabet for alignment
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:519aa368695f909489f21020a002821e4ee67dcd26d2f17212c98ed9c096b4a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# smith-waterman-compressed-read-alignment

## Summary

Reconstructs DBG2OLC's fast pair-wise overlap-scoring step, in which candidate read pairs sharing contig anchors are aligned via Smith-Waterman over their compressed contig-identifier sequences to compute overlap scores without base-level read alignment.

## When to use

Use when long reads have already been compressed into contig-identifier sequences and you must identify overlapping read pairs efficiently, avoiding all-against-all base-level alignment.

## When NOT to use

- Input long reads are raw, uncompressed sequences rather than compressed contig-identifier representations — the Smith-Waterman step operates on identifier sequences, not bases
- A base-level all-against-all read overlap computation is explicitly required (e.g. for non-hybrid long-read-only overlap graphs) rather than anchor-guided candidate selection
- Reads contain chimeric segments and RemoveChimera cannot be enabled (e.g. <10x coverage), so scored links cannot be trusted

## Inputs

- Compressed-reads file (ReadsInfoFrom_*) mapping long reads to contig identifier sequences
- Contigs file in FASTA format (e.g. Contigs.txt from SparseAssembler)
- Long reads in FASTA format (e.g. 20x PacBio reads)

## Outputs

- candidate_overlaps.tsv with read-pair and overlap-score fields
- Set of retained read pairs whose overlap score >= MinOverlap, with chimeric links removed

## How to apply

Load the compressed-reads file (ReadsInfoFrom_*) and build an inverted index mapping each contig identifier to the long reads containing it, so candidate pairs can be selected from shared contig identifiers before any alignment. Filter anchors with the DBG2OLC thresholds: matched k-mers M must satisfy M >= AdaptiveTh * Contig_Length and M >= KmerCovTh (e.g. AdaptiveTh 0.0001-0.01, KmerCovTh 2 for 20x PacBio data). For each candidate pair, run Smith-Waterman dynamic programming over the compressed-read identifier sequences, scoring identifier matches/mismatches weighted by contig length or matching k-mer counts to yield an overlap score. Retain pairs whose overlap score meets MinOverlap (10-30 for 10x/20x PacBio data, 50-150 for 50x-100x data) and apply RemoveChimera 1 to drop chimeric read links before reporting the pair/score table.

## Related tools

- **DBG2OLC** (Implements the inverted index over compressed reads, anchor selection with AdaptiveTh/KmerCovTh, Smith-Waterman scoring of compressed-read pairs against MinOverlap, and RemoveChimera filtering) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate DBG contigs (Contigs.txt) that are compressed into the long reads used as the identifier alphabet for alignment) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- candidate_overlaps.tsv exists and contains read-pair and score fields
- Every retained read pair has an overlap score >= MinOverlap (e.g. 20 for the w303/A. thaliana 20x runs)
- No retained pair uses a contig anchor with matched k-mers M below both AdaptiveTh*Contig_Length and KmerCovTh
- Chimeric read links have been dropped when RemoveChimera 1 was set (coverage >10x)
- Downstream check: N50 of backbone_raw.fasta (e.g. 583 kbp in the README's w303 20x run) is acceptable before proceeding to consensus

## Limitations

- Results are sensitive to the three quality-affecting parameters; README suggests tuning ranges (KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x/20x; KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02 for 50x-100x)
- Contigs from short-read assemblers that applied repeat-resolving heuristics (gap closing, scaffolding) introduce errors and can degrade the overlap/layout results
- Chimeric reads cannot be reliably removed below ~10x coverage (RemoveChimera suggested 1 only at >10x); at ~100x coverage ChimeraTh/ContigTh may need to be raised to 2 to remove false contig anchors

## Evidence

- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [methods] MinOverlap: minimum overlap score between a pair of long reads.: "MinOverlap: minimum overlap score between a pair of long reads."
- [methods] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [abstract] Compact representation of the long reads leads to efficient alignments.: "Compact representation of the long reads leads to efficient alignments."
