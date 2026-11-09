---
name: best-overlap-layout-chaining
description: Use when you have long erroneous reads (e.g. PacBio) compressed by DBG
  contigs and need an overlap-layout backbone before consensus; e.g. running DBG2OLC
  Step2 with k 17, KmerCovTh, AdaptiveTh, MinOverlap.
when_to_use_negative:
- The contigs came from a pipeline that already applied repeat resolving such as gap
  closing or scaffolding — the README warns this yields poor final results due to
  heuristics-introduced errors
- You are at the consensus/polishing stage (Step 3 with BLASR and Sparc/pbdagcon)
  — this skill ends at backbone_raw.fasta
- You have only short Illumina reads with no long reads to compress — use the DBG
  assembler directly instead
edam_operation: http://edamontology.org/operation_0525
edam_topics: []
tools:
- name: DBG2OLC
  role: Performs the two-round containment filtering, suffix-prefix overlap detection
    via compressed-read k-mer matching, best-overlap chaining, and tip/bubble simplification
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate raw DBG contigs (Contigs.txt) used to anchor and compress
    the long reads that feed the overlap graph
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Selects the read subset (SelectLongestReads) and evaluates backbone statistics
    (AssemblyStatistics) around the layout step
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:ca0be9d8e5ab6643cc69fe90844ff901643ddea2fd3bbb30d8cef7e06605a00e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# best-overlap-layout-chaining

## Summary

Constructs the DBG2OLC best overlap graph from compressed long reads: a two-round process that first removes contained nodes, then detects suffix-prefix overlaps by aligning k-mer-compressed reads, chains best-overlap links, and simplifies tips/bubbles to produce backbone_raw.fasta.

## When to use

Use when you have long erroneous reads (e.g. PacBio) compressed by DBG contigs and need an overlap-layout backbone before consensus; e.g. running DBG2OLC Step2 with k 17, KmerCovTh, AdaptiveTh, MinOverlap.

## When NOT to use

- The contigs came from a pipeline that already applied repeat resolving such as gap closing or scaffolding — the README warns this yields poor final results due to heuristics-introduced errors
- You are at the consensus/polishing stage (Step 3 with BLASR and Sparc/pbdagcon) — this skill ends at backbone_raw.fasta
- You have only short Illumina reads with no long reads to compress — use the DBG assembler directly instead

## Inputs

- Cleaned compressed reads from DBG2OLC (long reads compressed/anchored by DBG contigs, loadable with LD 1)
- DBG contigs in fasta format from the short-read DBG assembler (Contigs.txt from SparseAssembler)
- Long erroneous reads in fasta format (e.g. 10x-20x PacBio coverage)

## Outputs

- backbone_raw.fasta — the chained overlap-layout backbone
- DBG2OLC_Consensus_info.txt — read-to-backbone mapping consumed by the consensus step
- Recorded assembly breakpoints at unresolvable repeats

## How to apply

Load the cleaned compressed reads (long reads anchored by SparseAssembler DBG contigs) and run round 1 of overlap filtering to remove contained nodes — {Contig_a, Contig_b} is dropped if {Contig_a, Contig_b, Contig_c} is present. In round 2, detect suffix-prefix overlaps among remaining nodes using k 17: for each pair of long reads, compute M = matched k-mers and enforce AdaptiveTh (M >= AdaptiveTh * Contig_Length), KmerCovTh (M >= KmerCovTh), and MinOverlap thresholds, with RemoveChimera 1 when coverage >10x. For 10x–20x PacBio data use KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01; for 50x–100x use KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02 (ChimeraTh/ContigTh 2 at ~100x). Record each node's best overlapped nodes before/after as directed links and chain reads along these edges into the layout. Simplify the graph by removing tiny tips and merging bubbles, keeping unresolvable repeats as explicit assembly breakpoints rather than misjoining; if the N50 of backbone_raw.fasta is unsatisfactory, retune KmerCovTh/MinOverlap/AdaptiveTh before consensus.

## Related tools

- **DBG2OLC** (Performs the two-round containment filtering, suffix-prefix overlap detection via compressed-read k-mer matching, best-overlap chaining, and tip/bubble simplification) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate raw DBG contigs (Contigs.txt) used to anchor and compress the long reads that feed the overlap graph) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Selects the read subset (SelectLongestReads) and evaluates backbone statistics (AssemblyStatistics) around the layout step) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- backbone_raw.fasta exists with positive node/edge counts consistent with the cleaned compressed-read set
- N50 of backbone_raw.fasta (measured with AssemblyStatistics contigs) is satisfactory; README: if not satisfied, 'keep tuning and don't proceed' to consensus
- Every recorded best-overlap link satisfies MinOverlap (overlap score from aligning compressed reads by matching k-mers)
- Contained nodes were removed in round 1 (e.g. {Contig_a, Contig_b} absent when {Contig_a, Contig_b, Contig_c} is present)
- Chimeric reads dropped when RemoveChimera 1 is set at >10x coverage; unresolvable repeats left as breakpoints, not misjoined

## Limitations

- Truly unresolvable repeats remain as assembly breakpoints in the backbone rather than being resolved
- Parameter ranges are coverage-dependent: 10x/20x PacBio needs KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01, while 50x-100x needs MinOverlap 50-150; at ~100x set ChimeraTh 2 and ContigTh 2
- k 17 is described as the value that 'works well'; other k sizes are less supported
- Combining DBG2OLC with other assembly/error-correction tools may worsen results because those tools introduce errors

## Evidence

- [methods] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [methods] KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read.: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers.: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [readme] For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01.: "For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01."
- [readme] You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed.: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
