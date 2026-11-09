---
name: inverted-index-contig-to-read-mapping
description: Use when long erroneous 3GS reads must be compressed against DBG contigs
  and overlapping read pairs identified without all-vs-all base-level alignment, e.g.
  10x-100x PacBio data with SparseAssembler contigs.
when_to_use_negative:
- The long reads have not been compressed against DBG contigs, so no contig identifiers
  exist to index
- Coverage is below ~10x, where RemoveChimera is not suggested and read anchors are
  unreliable
- Short-read contigs come from repeat-resolved assemblies (gap closing/scaffolding),
  which the README warns introduce heuristics errors into the final assembly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DBG2OLC
  role: Builds the contig-to-read inverted index, applies AdaptiveTh/KmerCovTh anchor
    thresholds, computes Smith-Waterman overlap scores on compressed reads, and filters
    chimeras (RemoveChimera)
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate DBG contigs (Contigs.txt) whose identifiers populate
    the inverted index
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads prepares the read subset (e.g. sum 260000000 longest 0)
    fed into the compression/overlap step
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:a317235ca9e04b0312b9f063e7183c0ad71bdafd078be2d05f3eec11d85dc846
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
yes_rate: 1.0
n_claims: 10
---

# Inverted index contig-to-read mapping for fast pair-wise overlap scoring

## Summary

DBG2OLC's overlap-and-layout accelerator: an inverted index maps each short-read contig identifier to the long reads whose compressed representation contains it, so candidate overlapping read pairs are selected by shared contig identifiers before Smith-Waterman alignment of compressed identifier sequences.

## When to use

Use when long erroneous 3GS reads must be compressed against DBG contigs and overlapping read pairs identified without all-vs-all base-level alignment, e.g. 10x-100x PacBio data with SparseAssembler contigs.

## When NOT to use

- The long reads have not been compressed against DBG contigs, so no contig identifiers exist to index
- Coverage is below ~10x, where RemoveChimera is not suggested and read anchors are unreliable
- Short-read contigs come from repeat-resolved assemblies (gap closing/scaffolding), which the README warns introduce heuristics errors into the final assembly

## Inputs

- Compressed long reads (ReadsInfoFrom_* files) produced by the DBG2OLC compression step
- DBG contig file in FASTA format (e.g. SparseAssembler Contigs.txt)
- Long reads in FASTA format (e.g. Pacbio_20x.fasta)
- Parameter values: k (17), AdaptiveTh, KmerCovTh, MinOverlap, RemoveChimera

## Outputs

- Inverted index from contig identifier to list of long reads containing it
- Anchored contig-to-read pairs passing AdaptiveTh and KmerCovTh thresholds
- Scored candidate overlap table of read pairs with overlap scores >= MinOverlap
- Layout/backbone file (backbone_raw.fasta) after chimera filtering

## How to apply

Load the compressed-reads file produced by the compression step and build an inverted index mapping each contig identifier to the long reads containing it. For each contig, retrieve reads sharing it and keep contig-to-read anchors only if matched k-mers M satisfy M >= AdaptiveTh*Contig_Length and M >= KmerCovTh (e.g. AdaptiveTh 0.0001-0.01, KmerCovTh 2 for 20x PacBio data). For each candidate pair, run Smith-Waterman over the compressed identifier sequences, scoring identifier matches weighted by matching k-mer counts, and retain pairs whose overlap score >= MinOverlap (10-30 for 10x/20x, 50-150 for 50x-100x PacBio data). Run RemoveChimera 1 (coverage >10x) to drop chimeric read links before writing the scored overlap table. The rationale is that compact contig-identifier representations make alignments efficient, avoiding expensive base-level all-vs-all comparisons.

## Related tools

- **DBG2OLC** (Builds the contig-to-read inverted index, applies AdaptiveTh/KmerCovTh anchor thresholds, computes Smith-Waterman overlap scores on compressed reads, and filters chimeras (RemoveChimera)) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate DBG contigs (Contigs.txt) whose identifiers populate the inverted index) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads prepares the read subset (e.g. sum 260000000 longest 0) fed into the compression/overlap step) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- Every retained read pair in the overlap table has an overlap score >= MinOverlap (e.g. 20 for 20x PacBio data)
- Each anchored contig-read link satisfies both M >= AdaptiveTh*Contig_Length and M >= KmerCovTh matched k-mers
- Chimeric read links are absent after RemoveChimera 1, verified on datasets with >10x coverage
- Downstream layout produces a backbone_raw.fasta whose N50 (e.g. 583 kbp on S.cer w303 20x PacBio) is checked before proceeding to consensus
- Re-running with LD 1 reloads the same compressed/anchored reads, so the index and overlap scores should be reproducible

## Limitations

- Thresholds are coverage-dependent: 10x/20x PacBio uses KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01, while 50x-100x needs KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02; wrong ranges degrade assembly quality
- At ~100x coverage, false contig anchors and problematic reads require ChimeraTh/ContigTh set to 2
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because those tools introduce errors

## Evidence

- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] AdaptiveTh anchor rule: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [methods] MinOverlap definition: "MinOverlap: minimum overlap score between a pair of long reads."
- [methods] RemoveChimera suggestion: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [abstract] Compact representation rationale: "(i) Compact representation of the long reads leads to efficient alignments."
