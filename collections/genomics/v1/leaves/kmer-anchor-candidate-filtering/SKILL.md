---
name: kmer-anchor-candidate-filtering
description: Use when assembling long erroneous 3GS reads (PacBio/ONT) against short-read
  contigs and you need to find overlapping long-read pairs without all-vs-all base-level
  alignment, e.g. tuning DBG2OLC's AdaptiveTh, KmerCovTh, MinOverlap.
when_to_use_negative:
- You need base-level alignment accuracy for polishing — use the BLASR/Sparc/pbdagcon
  consensus step instead (~90% of DBG2OLC runtime)
- The contigs come from a repeat-resolved/scaffolded short-read assembly — the manual
  warns this introduces heuristics errors and gives poor final results
- You are doing pure short-read de novo assembly without long reads — the k-mer anchor
  scoring is defined only between compressed long reads
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: DBG2OLC
  role: Implements the compression, inverted-index anchor selection, Smith-Waterman
    scoring of compressed reads, MinOverlap filtering and RemoveChimera step
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate short-read DBG contigs (Contigs.txt) that serve as the
    k-mer anchors compressed into the long reads
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads subsets the long-read FASTA input (e.g. first 20x PacBio
    reads) before compression and scoring
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:06c9bfb5bd29062e590adc062ec15d67c1c78bd0376509556efbbe81a0db24f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# kmer-anchor-candidate-filtering

## Summary

Reconstructs DBG2OLC's fast pair-wise overlap-scoring stage: long reads are compressed into contig-identifier sequences, candidate overlapping read pairs are selected via shared k-mer anchors (AdaptiveTh/KmerCovTh), then scored by Smith-Waterman alignment of the compressed identifier sequences and filtered by MinOverlap.

## When to use

Use when assembling long erroneous 3GS reads (PacBio/ONT) against short-read contigs and you need to find overlapping long-read pairs without all-vs-all base-level alignment, e.g. tuning DBG2OLC's AdaptiveTh, KmerCovTh, MinOverlap.

## When NOT to use

- You need base-level alignment accuracy for polishing — use the BLASR/Sparc/pbdagcon consensus step instead (~90% of DBG2OLC runtime)
- The contigs come from a repeat-resolved/scaffolded short-read assembly — the manual warns this introduces heuristics errors and gives poor final results
- You are doing pure short-read de novo assembly without long reads — the k-mer anchor scoring is defined only between compressed long reads

## Inputs

- Compressed long-read file produced by DBG2OLC (ReadsInfoFrom_* files), where each long read is a sequence of contig identifiers
- Short-read DBG contigs in FASTA format (Contigs.txt from SparseAssembler, raw contigs without scaffolding)
- Long reads in FASTA format (e.g. Pacbio_20x.fasta), optionally subsetted with SelectLongestReads

## Outputs

- Table of candidate overlapping read pairs with overlap scores, restricted to pairs with score >= MinOverlap
- Backbone layout (backbone_raw.fasta) of contigs threading through long reads after chimera removal

## How to apply

Load the compressed-reads file produced by the DBG2OLC compression step and build an inverted index mapping each contig identifier to the long reads containing it. For each contig, retrieve the reads that share it and keep it as an anchor only if the matched k-mer count M satisfies both M >= AdaptiveTh*Contig_Length and M >= KmerCovTh (k 17 works well). For each candidate pair, run Smith-Waterman dynamic programming over the compressed-read identifier sequences, scoring matches weighted by matching k-mers, to compute an overlap score. Retain pairs whose score meets MinOverlap (10-30 for 10x/20x PacBio data, 50-150 for 50x-100x data) and apply RemoveChimera 1 to drop chimeric read links. The rationale is the paper's design principle that compact long-read representations make alignments efficient, avoiding expensive base-level overlap computation.

## Related tools

- **DBG2OLC** (Implements the compression, inverted-index anchor selection, Smith-Waterman scoring of compressed reads, MinOverlap filtering and RemoveChimera step) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate short-read DBG contigs (Contigs.txt) that serve as the k-mer anchors compressed into the long reads) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads subsets the long-read FASTA input (e.g. first 20x PacBio reads) before compression and scoring) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- Every retained read pair in the candidate overlap table has an overlap score >= MinOverlap (e.g. 20 in the w303 20x PacBio test)
- Anchor contigs satisfy both thresholds: matched k-mers M >= AdaptiveTh*Contig_Length and M >= KmerCovTh
- Chimeric read links are dropped when RemoveChimera 1 is set (suggest 1 for >10x coverage; ChimeraTh 2 at ~100x)
- Resulting backbone_raw.fasta N50 should be checked before consensus — e.g. N50 583 kbp on the S.cer w303 20x PacBio run — and if unsatisfactory, retune KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01

## Limitations

- Parameters are coverage-dependent: 10x/20x PacBio uses KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01, while 50x-100x needs MinOverlap 50-150, AdaptiveTh 0.01-0.02; wrong settings degrade assembly quality
- At ~100x coverage, ChimeraTh and ContigTh must be raised to 2 to remove problematic reads and false contig anchors in the multiple alignment
- Anchors from a scaffolded/gap-closed contig set introduce heuristics errors and give poor final results per the manual's warning

## Evidence

- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] AdaptiveTh threshold definition: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh threshold definition: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [methods] MinOverlap definition: "MinOverlap: minimum overlap score between a pair of long reads."
- [readme] RemoveChimera suggestion: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [abstract] Compact representation design principle: "(i) Compact representation of the long reads leads to efficient alignments."
