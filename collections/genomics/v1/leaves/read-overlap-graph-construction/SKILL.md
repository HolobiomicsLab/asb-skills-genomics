---
name: read-overlap-graph-construction
description: Use when running DBG2OLC overlap-layout on hybrid data — an accurate
  DBG contig set (e.g., SparseAssembler Contigs.txt, N50 ~29-32 kbp) plus 10x-100x
  error-prone PacBio/ONT FASTA reads — and a backbone_raw.fasta must be produced before
  consensus.
when_to_use_negative:
- You are already past layout and hold backbone_raw.fasta plus DBG2OLC_Consensus_info.txt
  — the next skill is BLASR-based consensus (Sparc/pbdagcon), not overlap graph construction.
- Your DBG contigs come from a short-read pipeline that applied repeat resolving such
  as gap closing or scaffolding — the README warns these heuristics introduce errors
  and yield poor final results.
- You are assembling short reads alone without long reads — this is the hybrid overlap-layout
  skill, not the non-hybrid NGS assembly commands.
edam_operation: http://edamontology.org/operation_0524
edam_topics: []
tools:
- name: DBG2OLC
  role: Compresses long reads with DBG-contig anchors, builds the two-round best overlap
    graph (containment filtering + suffix-prefix overlap scoring), and emits backbone_raw.fasta
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate raw DBG contigs (Contigs.txt) that serve as anchors
    for compressing long reads in the overlap graph
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads subsets long reads to a target coverage (e.g., sum 260000000
    for 20x) before overlap graph construction
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:2433ea48188899c7bea97b658e27a216e3dafb0e7a4fd8df65cede87edb6c9d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# read-overlap-graph-construction

## Summary

Construct DBG2OLC's best overlap graph from long erroneous reads compressed into DBG-contig anchors: round 1 removes contained nodes, round 2 finds suffix-prefix overlaps by aligning compressed reads via matching 17-mers, then tips are pruned and bubbles merged to yield backbone_raw.fasta.

## When to use

Use when running DBG2OLC overlap-layout on hybrid data — an accurate DBG contig set (e.g., SparseAssembler Contigs.txt, N50 ~29-32 kbp) plus 10x-100x error-prone PacBio/ONT FASTA reads — and a backbone_raw.fasta must be produced before consensus.

## When NOT to use

- You are already past layout and hold backbone_raw.fasta plus DBG2OLC_Consensus_info.txt — the next skill is BLASR-based consensus (Sparc/pbdagcon), not overlap graph construction.
- Your DBG contigs come from a short-read pipeline that applied repeat resolving such as gap closing or scaffolding — the README warns these heuristics introduce errors and yield poor final results.
- You are assembling short reads alone without long reads — this is the hybrid overlap-layout skill, not the non-hybrid NGS assembly commands.

## Inputs

- DBG contig file in FASTA format (e.g., Contigs.txt from SparseAssembler, raw DBG contigs without repeat-resolving heuristics)
- Long erroneous reads in FASTA format (e.g., Pacbio_20x.fasta, 10x-100x PacBio or Oxford Nanopore)
- Parameter values: k (17), KmerCovTh, AdaptiveTh, MinOverlap, RemoveChimera, optional MinLen/ChimeraTh/ContigTh

## Outputs

- backbone_raw.fasta — the overlap-graph layout backbone to be checked for N50 before consensus
- DBG2OLC_Consensus_info.txt — read-to-backbone mapping consumed by the consensus step
- Compressed (anchored) long reads — reloadable with LD 1 for parameter retuning
- Console log (e.g., DBG2OLC_LOG.txt) with overlap graph node/edge counts

## How to apply

Feed DBG2OLC the raw DBG contigs (no gap closing/scaffolding) plus the long-read FASTA with k 17, scoring each pair of long reads by aligning their compressed reads via matching k-mers. Enforce anchoring thresholds M >= KmerCovTh (fixed, e.g., 2) and M >= AdaptiveTh*Contig_Length (adaptive, 0.0001-0.01 for 10x-20x PacBio, 0.01-0.02 for 50x-100x), and require the pair overlap score to exceed MinOverlap (10-30 for 10x-20x PacBio, 50-150 for 50x-100x); set RemoveChimera 1 above 10x coverage to drop chimeric reads. Round 1 of graph construction filters off contained nodes (a read fully contained in another is removed), and round 2 detects suffix-prefix overlaps among remaining nodes, recording the best overlapped nodes before and after each node as links. Chain each read along its directed best-overlap edges and simplify: remove tiny tips, merge bubbles, and record truly unresolvable repeats as assembly breakpoints rather than misjoining. Use LD 1 to reload previously computed compressed/anchored reads when retuning, and check the N50 of backbone_raw.fasta before proceeding to consensus (example: 583 kbp N50 on S.cer w303 20x PacBio); if unsatisfactory, retune KmerCovTh/MinOverlap/AdaptiveTh and rerun rather than advancing to the Sparc/pbdagcon consensus step.

## Related tools

- **DBG2OLC** (Compresses long reads with DBG-contig anchors, builds the two-round best overlap graph (containment filtering + suffix-prefix overlap scoring), and emits backbone_raw.fasta) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate raw DBG contigs (Contigs.txt) that serve as anchors for compressing long reads in the overlap graph) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads subsets long reads to a target coverage (e.g., sum 260000000 for 20x) before overlap graph construction) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- backbone_raw.fasta exists with positive node/edge counts consistent with the cleaned, chimera-filtered read set
- N50 of backbone_raw.fasta meets expectation before proceeding (README example: 583 kbp on S.cer w303 with 20x PacBio); otherwise retune KmerCovTh/MinOverlap/AdaptiveTh and do not advance
- Every long-read pair overlap score derives from aligning compressed reads via matching k-mers, with anchors rejected when M < KmerCovTh or M < AdaptiveTh*Contig_Length
- Contained nodes are removed in round 1 (e.g., {Contig_a, Contig_b} dropped when {Contig_a, Contig_b, Contig_c} is present), and unresolvable repeats persist as explicit breakpoints rather than misjoins
- Nearly perfect diagonal dot plots against the reference indicate the layout is structurally correct, as reported for assemblies from as low as 10x long-read data

## Limitations

- Parameters are coverage- and genome-dependent: suggested ranges are KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x/20x PacBio, but KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02 for 50x-100x; at ~100x also set ChimeraTh 2 and ContigTh 2
- Truly unresolvable repeats remain as assembly breakpoints — the graph does not resolve them, so the backbone may break there
- k 17 is the only well-validated k-mer size ('17 works well'); MinLen is a less flexible secondary parameter
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many existing tools introduce errors, so the prescribed pipeline order matters

## Evidence

- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] AdaptiveTh and KmerCovTh anchoring rules: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] MinOverlap definition: "MinOverlap: minimum overlap score between a pair of long reads."
- [readme] RemoveChimera guidance: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] N50 gate before consensus: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
- [methods] Input contig file requirement: "Feed DBG2OLC with the contig file in fasta format from the previous step (Contigs.txt in this example)."
