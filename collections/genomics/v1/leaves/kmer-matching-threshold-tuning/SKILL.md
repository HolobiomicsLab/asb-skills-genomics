---
name: kmer-matching-threshold-tuning
description: Use when running DBG2OLC overlap/layout on PacBio (3GS) long reads with
  preassembled Illumina contigs and the initial parameter set yields a low backbone
  N50 or poor anchoring.
when_to_use_negative:
- The input long reads have already been error-corrected or processed by other assembly/error-correction
  tools — the README warns these introduce errors and give worse DBG2OLC results
- Coverage is below ~10x, where RemoveChimera 1 and standard thresholds are not suggested
- The goal is polishing/consensus rather than draft contiguity — ~90% of computation
  time is spent on the BLASR consensus step, so tune thresholds first instead of paying
  that cost
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Performs the overlap and layout whose contig-anchoring (AdaptiveTh, KmerCovTh)
    and long-read overlap scoring (MinOverlap) parameters are being tuned
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate raw DBG contigs (e.g. LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh
    0; retuned with LD 1) that serve as anchors during threshold tuning
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Provides AssemblyStatistics to evaluate the N50 of backbone_raw.fasta after
    each tuning iteration
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:da8e95d92a5d4b056716639288659cd35a8c04b9db0da72bdcec52888f188932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
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

# kmer-matching-threshold-tuning

## Summary

Fine-tune DBG2OLC's k-mer matching thresholds (AdaptiveTh, KmerCovTh, MinOverlap) that decide whether a short-read contig anchors to a long read and how long-read overlaps are scored, to maximize draft assembly N50 before proceeding to consensus.

## When to use

Use when running DBG2OLC overlap/layout on PacBio (3GS) long reads with preassembled Illumina contigs and the initial parameter set yields a low backbone N50 or poor anchoring.

## When NOT to use

- The input long reads have already been error-corrected or processed by other assembly/error-correction tools — the README warns these introduce errors and give worse DBG2OLC results
- Coverage is below ~10x, where RemoveChimera 1 and standard thresholds are not suggested
- The goal is polishing/consensus rather than draft contiguity — ~90% of computation time is spent on the BLASR consensus step, so tune thresholds first instead of paying that cost

## Inputs

- SparseAssembler DBG contigs in FASTA format (Contigs.txt, raw contigs without gap closing or scaffolding)
- PacBio/3GS long reads in FASTA format (e.g. 20x, 30x, or 50x-100x subsets)
- Previously computed DBG2OLC compressed/anchored reads (loadable via LD 1) for iterative tuning

## Outputs

- backbone_raw.fasta (DBG2OLC draft assembly backbone)
- DBG2OLC_Consensus_info.txt (contig-to-backbone layout for the consensus step)
- Assembly statistics (N50/NG50) from AssemblyStatistics used to judge the tuning

## How to apply

Run DBG2OLC on the SparseAssembler contigs (Contigs.txt) with a starting parameter set such as k 17, KmerCovTh 2, MinOverlap 20, AdaptiveTh 0.01, RemoveChimera 1, then evaluate backbone_raw.fasta N50 with AssemblyStatistics before calling consensus. The three governing quantities are: M = matched k-mers between a contig and a long read; AdaptiveTh rejects a contig as anchor if M < AdaptiveTh * Contig_Length (adaptive), KmerCovTh applies a fixed k-mer matching threshold, and MinOverlap sets the minimum compressed-read overlap score between a pair of long reads. Tune within the README's coverage-dependent ranges: for 10x/20x PacBio data use KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01; for 50x-100x PacBio data use KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02. Use LD 1 to reload the compressed/anchored reads between tuning iterations rather than recomputing from scratch. Keep k at 17 (it 'works well'), keep RemoveChimera 1 when coverage exceeds 10x, and do not proceed to the BLASR/Sparc consensus step until the N50 is satisfactory.

## Related tools

- **DBG2OLC** (Performs the overlap and layout whose contig-anchoring (AdaptiveTh, KmerCovTh) and long-read overlap scoring (MinOverlap) parameters are being tuned) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate raw DBG contigs (e.g. LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0; retuned with LD 1) that serve as anchors during threshold tuning) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Provides AssemblyStatistics to evaluate the N50 of backbone_raw.fasta after each tuning iteration) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 KmerCovTh 2 MinOverlap 20 AdaptiveTh 0.01 RemoveChimera 1 Contigs Contigs.txt f 30x.fasta >DBG2OLC_LOG.txt
```

## Evaluation signals

- backbone_raw.fasta exists and its N50, measured with AssemblyStatistics, improves across tuning iterations (e.g. 583 kbp on the S.cer w303 20x PacBio test; 6 Mbp draft N50 on the human 30x subset)
- The N50 is judged satisfactory before proceeding to consensus — the README explicitly says to check the N50 of backbone_raw.fasta and otherwise 'keep tuning and don't proceed'
- Anchor selection behaves as specified: a contig is rejected as an anchor when M < AdaptiveTh * Contig_Length or M < KmerCovTh, and long-read pairs fail when the compressed-read overlap score is below MinOverlap
- Chimeric reads are removed (RemoveChimera 1) whenever coverage exceeds 10x, and LD 1 successfully reloads compressed/anchored reads between runs

## Limitations

- At high coverage (~100x) additional parameters ChimeraTh and ContigTh should be set to 2; standard thresholds alone may leave problematic reads and false contig anchors in the multiple alignment
- Combining DBG2OLC with other assembly/error-correction software may produce worse results because such tools introduce errors; use raw DBG contigs without repeat resolving (gap closing, scaffolding)
- The DBG2OLC overlap/layout stage on large genomes is resource-heavy: on the human 30x longest PacBio reads it occupies 70 GB for the 17-mer index and takes 37 CPU hours

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01.: "For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01."
- [readme] You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed.: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don¡¯t proceed."
- [results] The final consensus takes roughly 2000 CPU hours. In an initial report by PacBio scientists, the overlapping process took 405,000 CPU hours: "The final consensus takes roughly 2000 CPU hours. In an initial report by PacBio scientists, the overlapping process took 405,000 CPU hours"
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
