---
name: hybrid-assembly-overlap-layout
description: Use when assembling a large genome from 10x-100x long erroneous 3GS reads
  (PacBio/ONT) plus accurate NGS contigs, and you need overlap/layout without base-level
  error correction before consensus.
when_to_use_negative:
- You only have NGS short reads and no 3GS long reads — use the DBG2OLC non-hybrid
  NGS assembly mode instead.
- You already have a structurally correct backbone and only need base-level polishing
  — proceed to the consensus step (BLASR + Sparc/pbdagcon) rather than re-running
  overlap/layout.
- The NGS input contains repeat-resolved, gap-closed, or scaffolded contigs — the
  README warns this 'may have poor final results due to the errors introduced by the
  heuristics used in short read assembly pipelines'.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: DBG2OLC
  role: Performs k-mer-anchored compression of long reads, overlap scoring, chimera
    removal, and layout to produce backbone_raw.fasta
  repo: github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles ~50x Illumina reads into the short accurate contigs (Contigs.txt)
    that anchor the long reads
  repo: github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Selects read subsets (SelectLongestReads) and evaluates assembly statistics
    (AssemblyStatistics)
  repo: https://github.com/yechengxi/AssemblyUtility
- name: Sparc
  role: Downstream consensus module applied to the backbone produced by this skill
  repo: https://github.com/yechengxi/Sparc
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:61df8439b4691018ee2ea5a407a6f729c72738e924c76b6565711bc6466e563e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AssemblyUtility
  - DBG2OLC
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hybrid-assembly-overlap-layout

## Summary

Performs the DBG2OLC overlap-and-layout stage of hybrid assembly: accurate short-read DBG contigs (e.g. SparseAssembler Contigs.txt) are used as k-mer anchors (k 17) to compress 3GS long reads into ordered contig-identifier lists and emit the assembly backbone (backbone_raw.fasta). This skips base-level errors and corrects only structural errors, cutting compute and sequencing cost.

## When to use

Use when assembling a large genome from 10x-100x long erroneous 3GS reads (PacBio/ONT) plus accurate NGS contigs, and you need overlap/layout without base-level error correction before consensus.

## When NOT to use

- You only have NGS short reads and no 3GS long reads — use the DBG2OLC non-hybrid NGS assembly mode instead.
- You already have a structurally correct backbone and only need base-level polishing — proceed to the consensus step (BLASR + Sparc/pbdagcon) rather than re-running overlap/layout.
- The NGS input contains repeat-resolved, gap-closed, or scaffolded contigs — the README warns this 'may have poor final results due to the errors introduced by the heuristics used in short read assembly pipelines'.

## Inputs

- NGS contig FASTA from an accurate DBG assembler (Contigs.txt, e.g. SparseAssembler output; raw contigs, no scaffolding/gap closing)
- 3GS long-read FASTA (e.g. Pacbio_20x.fasta, 30x human PacBio reads)
- Optional: read subsets selected with AssemblyUtility SelectLongestReads (sum/longest modes)

## Outputs

- backbone_raw.fasta (assembly backbone/layout)
- DBG2OLC_Consensus_info.txt (per-backbone read mapping used by the consensus step)
- Compressed long reads: ordered contig-identifier lists (~3x shorter than original reads)

## How to apply

First preassemble ~50x Illumina reads into short accurate contigs with SparseAssembler (e.g. k 51, g 15, NodeCovTh 1, EdgeCovTh 0; retune with LD 1 if needed), using raw DBG contigs without gap closing or scaffolding. Then run DBG2OLC with k 17, KmerCovTh 2, MinOverlap 20, AdaptiveTh tuned to coverage (0.0001 for 10x/20x PacBio, 0.01 for 50x-100x), and RemoveChimera 1 when coverage exceeds 10x; AdaptiveTh scales with contig length (a contig anchoring a read needs M >= AdaptiveTh*Contig_Length matched k-mers) while KmerCovTh is a fixed minimum, and overlap scores between compressed reads are computed from matching k-mers with the MinOverlap cutoff. For ~100x coverage set ChimeraTh 2 and ContigTh 2, which are used in the multiple alignment to remove problematic reads and false contig anchors. Check the N50 of backbone_raw.fasta (583 kbp in the reference S.cer w303 20x run; ~6 Mbp draft N50 on the 30x human dataset) and keep tuning parameters (KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x/20x data) rather than proceeding to consensus if unsatisfied. LD 1 can reload previously computed compressed/anchored reads when retuning.

## Related tools

- **DBG2OLC** (Performs k-mer-anchored compression of long reads, overlap scoring, chimera removal, and layout to produce backbone_raw.fasta) — github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles ~50x Illumina reads into the short accurate contigs (Contigs.txt) that anchor the long reads) — github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Selects read subsets (SelectLongestReads) and evaluates assembly statistics (AssemblyStatistics)) — https://github.com/yechengxi/AssemblyUtility
- **Sparc** (Downstream consensus module applied to the backbone produced by this skill) — https://github.com/yechengxi/Sparc

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- backbone_raw.fasta exists and its N50 is acceptable — 583 kbp in the reference S.cer w303 20x PacBio run; ~6 Mbp draft N50 on the 30x human dataset (NG50 433 kbp from 10x reads)
- Each long read is compressed into an ordered contig-identifier list roughly 3x shorter than the original read (threshold range 0.001-0.02)*Contig_Length
- Multi-contig k-mers are excluded from the 17-mer index, and contigs with fewer than AdaptiveTh*Contig_Length or KmerCovTh matched k-mers do not anchor any read
- With RemoveChimera 1, chimeric reads and spurious contig identifiers detected by the multiple sequence alignment are absent from the cleaned compressed-read set
- Alignment of raw long reads against the backbone yields near-diagonal dot plots, indicating structural correctness even from 10x long-read data

## Limitations

- Assembling with tools other than the prescribed pipeline 'may produce worse results' because existing assembly/error-correction tools introduce errors; follow the steps before inventing your own pipeline.
- Combining DBG2OLC with repeat-resolving techniques (gap closing, scaffolding) in the contig input introduces errors and degrades final results.
- Scaling caveats: the 54x human run used 70 GB memory for the 17-mer index and 37 CPU hours for compression/alignment of the 30x longest reads, with consensus taking roughly 2000 CPU hours.
- At ~100x coverage the default ChimeraTh/ContigTh (1) are too lenient and should be set to 2.

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read.: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [results] an identifier is reported only if its uniquely matching k-mer count exceeds an adaptive threshold based on contig length: [evidence span withheld — anchor status: not_found]
- [readme] It is important to follow the steps before you invent your own pipeline.: "It is important to follow the steps before you invent your own pipeline. As combining DBG2OLC with other software tools may produce worse results."
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
