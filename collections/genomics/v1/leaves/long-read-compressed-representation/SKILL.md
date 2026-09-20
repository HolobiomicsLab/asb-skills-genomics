---
name: long-read-compressed-representation
description: Use when assembling large genomes (e.g., 3 Gbp H. sapiens) from long
  erroneous 3GS reads (PacBio/MinION) plus accurate short-read contigs, and base-level
  all-vs-all long-read overlap is computationally prohibitive.
when_to_use_negative:
- When only long reads are available with no accurate short-read contigs — the compression
  step depends on NGS contig anchors
- When short-read assemblies have been gap-closed or scaffolded — README warns this
  yields poor final results due to errors from short-read assembly heuristics
- When a polished, base-level-accurate assembly is the immediate deliverable — this
  skill covers overlap/layout only; consensus/polishing (BLASR + Sparc/pbdagcon) is
  a separate step consuming ~90% of runtime
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Compresses long reads into contig-anchor representations and performs overlap
    and layout to produce backbone_raw.fasta
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles ~50x Illumina short reads into the accurate DBG contigs (Contigs.txt)
    used as anchors
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads extracts the longest 30x PacBio subset; AssemblyStatistics
    evaluates the draft backbone N50
  repo: https://github.com/yechengxi/AssemblyUtility
- name: BLASR
  role: Aligns all raw reads to the assembly backbone in the downstream consensus
    step that validates the compressed-layout draft
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:bbbfa18f73e13680162b12405f08c6649c9cea380b8dd6c326268a911c7e2e5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# long-read-compressed-representation

## Summary

Compact representation of erroneous long 3GS reads by encoding each PacBio read as an ordered list of matched short-read contig anchors (k-mer matches), enabling fast overlap/layout without base-level alignment of raw long reads.

## When to use

Use when assembling large genomes (e.g., 3 Gbp H. sapiens) from long erroneous 3GS reads (PacBio/MinION) plus accurate short-read contigs, and base-level all-vs-all long-read overlap is computationally prohibitive.

## When NOT to use

- When only long reads are available with no accurate short-read contigs — the compression step depends on NGS contig anchors
- When short-read assemblies have been gap-closed or scaffolded — README warns this yields poor final results due to errors from short-read assembly heuristics
- When a polished, base-level-accurate assembly is the immediate deliverable — this skill covers overlap/layout only; consensus/polishing (BLASR + Sparc/pbdagcon) is a separate step consuming ~90% of runtime

## Inputs

- Accurate short-read DBG contigs in FASTA (Contigs.txt from SparseAssembler, raw contigs without gap closing or scaffolding)
- Long erroneous 3GS reads in FASTA (e.g., PacBio 30x subset selected via SelectLongestReads)

## Outputs

- backbone_raw.fasta — draft assembly backbone of ordered contig anchors
- DBG2OLC_Consensus_info.txt — read-to-backbone layout information for the consensus step
- Assembly statistics (N50/NG50) from AssemblyStatistics on the draft backbone

## How to apply

First preassemble ~50x Illumina reads into raw DBG contigs with SparseAssembler (k 51, g 15, NodeCovTh 1, EdgeCovTh 0; retune with LD 1 if needed), avoiding repeat-resolved/scaffolded contigs. Then compress each long read by matching short contigs against it: a contig anchors to a long read only if the matched k-mer count M passes both thresholds AdaptiveTh*Contig_Length and KmerCovTh. Overlaps between long reads are scored by aligning the compressed (contig-anchor) representations with MinOverlap as the minimum overlap score, with k 17 as the k-mer size and RemoveChimera 1 for >10x coverage. Coverage-dependent tuning ranges apply: for 10x/20x PacBio use KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01; for 50x-100x use KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02. The rationale is the paper's design principles: compact representation of long reads leads to efficient alignments, and base-level errors can be skipped while structural errors are corrected; the draft backbone (backbone_raw.fasta) N50 should be checked before spending compute on the consensus step (~90% of runtime).

## Related tools

- **DBG2OLC** (Compresses long reads into contig-anchor representations and performs overlap and layout to produce backbone_raw.fasta) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles ~50x Illumina short reads into the accurate DBG contigs (Contigs.txt) used as anchors) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads extracts the longest 30x PacBio subset; AssemblyStatistics evaluates the draft backbone N50) — https://github.com/yechengxi/AssemblyUtility
- **BLASR** (Aligns all raw reads to the assembly backbone in the downstream consensus step that validates the compressed-layout draft)

## Examples

```
./DBG2OLC k 17 KmerCovTh 2 MinOverlap 20 AdaptiveTh 0.01 RemoveChimera 1 Contigs Contigs.txt f 30x.fasta >DBG2OLC_LOG.txt
```

## Evaluation signals

- backbone_raw.fasta exists and its N50 evaluated with AssemblyStatistics matches expectations before proceeding to consensus (e.g., ~6 Mbp draft N50 for the human dataset)
- Check the N50 of backbone_raw.fasta and keep tuning parameters (KmerCovTh, MinOverlap, AdaptiveTh) rather than proceeding if unsatisfied, per the README
- Resource envelope matches the paper: ~70 GB memory for the 17-mer index and ~37 CPU hours to compress and align the 30x longest human PacBio reads
- Anchor filtering invariants: every used contig-long-read pair must satisfy M >= AdaptiveTh*Contig_Length AND M >= KmerCovTh
- Nearly diagonal alignment dot plots against a reference indicate structurally correct assembly from as low as 10x long read data

## Limitations

- Combining DBG2OLC with other assemblers/error correctors may worsen results because many tools introduce errors; follow the prescribed pipeline steps
- SparseAssembler parameters may need retuning (LD 1, adjusted NodeCovTh/EdgeCovTh) for complex genomes; initial runs may not give the best contigs
- ChimeraTh/ContigTh must be raised to 2 at ~100x long-read coverage; RemoveChimera 1 is advised only above ~10x coverage
- Scaling: compression/alignment of the 30x human PacBio reads required 70 GB memory and 37 CPU hours, and full consensus took ~2000 CPU hours

## Evidence

- [abstract] Compact representation of the long reads leads to efficient alignments.: "(i) Compact representation of the long reads leads to efficient alignments. (ii) Base-level errors can be skipped; structural errors need to be detected and corrected."
- [results] we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [results] DBG2OLC occupies 70 GB memory to store the 17-mer index, and takes 37 CPU hours to compress and align the 30x longest PacBio reads: "DBG2OLC occupies 70 GB memory to store the 17-mer index, and takes 37 CPU hours to compress and align the 30x longest PacBio reads"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
