---
name: assembly-graph-tip-and-bubble-simplification
description: Use when building a long-read layout backbone from DBG contig-compressed
  reads (e.g., PacBio reads compressed by SparseAssembler contigs) and the overlap
  graph must be pruned of contained nodes, tips, and bubbles before consensus.
when_to_use_negative:
- The short-read contigs are not raw DBG contigs — gap closing, scaffolding, or repeat-resolving
  heuristics introduce errors that degrade the final assembly
- Coverage is below ~10x, where RemoveChimera-based chimera removal and MinOverlap
  tuning become unreliable
- The task is base-level polishing rather than structural layout — this skill skips
  base-level errors; base-level consensus is handled later by Sparc/pbdagcon with
  BLASR
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: DBG2OLC
  role: Builds the two-round best overlap graph (containment filtering, suffix-prefix
    overlap scoring via k-mer matching of compressed reads), chains best-overlap edges,
    and removes tips / merges bubbles; emits backbone_raw.fasta
  repo: github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the raw DBG contigs (Contigs.txt) used to compress long reads before
    overlap-graph construction
  repo: github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: AssemblyStatistics evaluates the backbone/assembly statistics (N50, NG50)
    when validating the simplified graph output
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:beaedafd7de6df5783d4505d4f77af68e11cd32c6178dd9ae6f1247f7f7503cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# assembly-graph-tip-and-bubble-simplification

## Summary

Constructs DBG2OLC's best overlap graph from compressed long reads via two rounds of filtering (containment removal, then suffix-prefix overlap detection) and simplifies it by removing tiny tips and merging bubbles, leaving unresolvable repeats as breakpoints rather than misjoins.

## When to use

Use when building a long-read layout backbone from DBG contig-compressed reads (e.g., PacBio reads compressed by SparseAssembler contigs) and the overlap graph must be pruned of contained nodes, tips, and bubbles before consensus.

## When NOT to use

- The short-read contigs are not raw DBG contigs — gap closing, scaffolding, or repeat-resolving heuristics introduce errors that degrade the final assembly
- Coverage is below ~10x, where RemoveChimera-based chimera removal and MinOverlap tuning become unreliable
- The task is base-level polishing rather than structural layout — this skill skips base-level errors; base-level consensus is handled later by Sparc/pbdagcon with BLASR

## Inputs

- Cleaned compressed long reads (PacBio/3GS reads anchored by DBG contigs from the compression step)
- DBG contigs in FASTA format (SparseAssembler Contigs.txt)
- Long reads in FASTA format (e.g., Pacbio_20x.fasta)

## Outputs

- backbone_raw.fasta (chained layout backbone of best-overlap edges)
- DBG2OLC_Consensus_info.txt (read-to-backbone layout used by the consensus step)
- Best overlap graph with recorded best-overlap links before/after each node; unresolvable repeats marked as breakpoints

## How to apply

Load the cleaned compressed reads (long reads anchored by DBG contigs, k 17) produced in the compression step. In round 1, remove contained nodes from the overlap graph (e.g., {Contig_a, Contig_b} is dropped if {Contig_a, Contig_b, Contig_c} is present). In round 2, detect suffix-prefix overlaps among the remaining nodes: for each pair of long reads, compute an overlap score by aligning their compressed reads with matching k-mers, anchoring contigs only if M >= KmerCovTh (fixed threshold, e.g., 2) and M >= AdaptiveTh*Contig_Length (adaptive threshold, 0.0001-0.01), with MinOverlap 10-30 for 10x-20x PacBio data and RemoveChimera 1 to drop chimeric reads. Record, for each node, the best overlapped nodes before and after it as directed links, then chain each read along these best-overlap edges to form the layout backbone (backbone_raw.fasta). Finally, simplify the graph: remove tiny tips, merge bubbles, and record truly unresolvable repeats as assembly breakpoints instead of misjoining sequences.

## Related tools

- **DBG2OLC** (Builds the two-round best overlap graph (containment filtering, suffix-prefix overlap scoring via k-mer matching of compressed reads), chains best-overlap edges, and removes tips / merges bubbles; emits backbone_raw.fasta) — github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the raw DBG contigs (Contigs.txt) used to compress long reads before overlap-graph construction) — github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (AssemblyStatistics evaluates the backbone/assembly statistics (N50, NG50) when validating the simplified graph output) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- backbone_raw.fasta exists with positive node/edge counts consistent with the cleaned compressed-read set
- Check the N50 of backbone_raw.fasta; if unsatisfactory, retune KmerCovTh/MinOverlap/AdaptiveTh and do not proceed to consensus (per README)
- Contained-node check: no node set (e.g., {Contig_a, Contig_b}) remains in the graph when a super-set set ({Contig_a, Contig_b, Contig_c}) exists
- AssemblyStatistics output on the final assembly shows N50/NG50 within expectations for the dataset (e.g., N50 583 kbp on S.cer w303 20x PacBio; NG50 433 kbp on 10x PacBio human)
- Unresolvable repeats appear as recorded breakpoints rather than as misjoined contigs (near-perfect diagonal dot plots indicate structurally correct assembly)

## Limitations

- KmerCovTh, MinOverlap, and AdaptiveTh are dataset-dependent: 10x/20x PacBio suggests KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01, while 50x-100x needs KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02; wrong ranges degrade the graph
- Truly unresolvable repeats cannot be removed by tip/bubble simplification and remain as assembly breakpoints
- High coverage (~100x) requires more stringent ChimeraTh 2 and ContigTh 2 to remove problematic reads and false contig anchors in the multiple alignment
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because such tools introduce errors

## Evidence

- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [results] thorough evaluations of our algorithm show that DBG2OLC can provide high quality results with fewer structural errors: "thorough evaluations of our algorithm show that DBG2OLC can provide high quality results with fewer structural errors"
- [results] The nearly perfect diagonal dot plots indicate that DBG2OLC can produce structurally correct assemblies from as low as 10x long read data: "The nearly perfect diagonal dot plots indicate that DBG2OLC can produce structurally correct assemblies from as low as 10x long read data"
