---
name: assembly-parameter-tuning
description: Use when running DBG2OLC overlap-layout on long erroneous reads (PacBio/MinION)
  and the first pass yields poor backbone contiguity, so k-mer anchor and overlap
  thresholds need coverage-aware tuning.
when_to_use_negative:
- When contigs are scaffolded or gap-closed rather than raw DBG contigs — the README
  warns this causes poor final results from short-read pipeline heuristics
- When the goal is base-level consensus polishing rather than overlap/layout tuning
  — that is the Sparc/BLASR consensus step, which consumes ~90% of compute time
- When combining DBG2OLC output with other assemblers or error-correction tools, since
  the README states such combinations may produce worse results
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: DBG2OLC
  role: Runs the k-mer-based read compression, overlap and layout whose AdaptiveTh,
    KmerCovTh, MinOverlap and RemoveChimera parameters are tuned here
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the raw DBG contigs (Contigs.txt) that serve as anchors; its NodeCovTh/EdgeCovTh
    may also be re-tuned on a loaded k-mer graph (LD 1)
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: Provides AssemblyStatistics to compute the backbone N50 used as the acceptance
    criterion after each tuning run, and SelectLongestReads to subset read coverage
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:8abe04a2db289596316a961ebc22dd1ba0a40621c943f47957e247c9d273ef07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# assembly-parameter-tuning

## Summary

Fine-tunes DBG2OLC hybrid assembly parameters (AdaptiveTh, KmerCovTh, MinOverlap, k, RemoveChimera) by coverage regime, checking the backbone N50 before proceeding to consensus.

## When to use

Use when running DBG2OLC overlap-layout on long erroneous reads (PacBio/MinION) and the first pass yields poor backbone contiguity, so k-mer anchor and overlap thresholds need coverage-aware tuning.

## When NOT to use

- When contigs are scaffolded or gap-closed rather than raw DBG contigs — the README warns this causes poor final results from short-read pipeline heuristics
- When the goal is base-level consensus polishing rather than overlap/layout tuning — that is the Sparc/BLASR consensus step, which consumes ~90% of compute time
- When combining DBG2OLC output with other assemblers or error-correction tools, since the README states such combinations may produce worse results

## Inputs

- Contigs.txt: raw de Bruijn graph contigs from SparseAssembler (e.g. k 51, g 15, NodeCovTh 1, EdgeCovTh 0), in FASTA format
- Long erroneous reads in FASTA format (PacBio or Oxford Nanopore MinION)
- Coverage estimate of the long-read dataset (10x-100x) to select parameter ranges

## Outputs

- backbone_raw.fasta: the assembly backbone whose N50 is checked against tuning goals
- DBG2OLC_Consensus_info.txt: read-to-backbone mapping info consumed by the Sparc/pbdagcon consensus step
- AssemblyStatistics .txt files with contig/backbone statistics (N50, NG50)

## How to apply

Run DBG2OLC with contigs from SparseAssembler (raw DBG contigs, no scaffolding) and long-read FASTA using k 17, then tune the three major parameters: M = matched k-mers between a contig and a long read; AdaptiveTh rejects a contig anchor when M < AdaptiveTh*Contig_Length, KmerCovTh rejects when M < KmerCovTh, and MinOverlap is the minimum compressed-read overlap score. Scale thresholds to coverage per the README ranges: for 10x/20x PacBio use KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01; for 50x-100x use KmerCovTh 2-10, MinOverlap 50-150, AdaptiveTh 0.01-0.02; set RemoveChimera 1 when coverage >10x (ChimeraTh/ContigTh 2 at ~100x). Use LD 1 to reload compressed/anchored reads rather than recomputing. Evaluate the backbone with AssemblyStatistics after each run (e.g. N50 583 kbp in the reference S.cer w303 20x PacBio test) and iterate before calling consensus.

## Related tools

- **DBG2OLC** (Runs the k-mer-based read compression, overlap and layout whose AdaptiveTh, KmerCovTh, MinOverlap and RemoveChimera parameters are tuned here) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the raw DBG contigs (Contigs.txt) that serve as anchors; its NodeCovTh/EdgeCovTh may also be re-tuned on a loaded k-mer graph (LD 1)) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (Provides AssemblyStatistics to compute the backbone N50 used as the acceptance criterion after each tuning run, and SelectLongestReads to subset read coverage) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- Backbone N50 computed with AssemblyStatistics meets the dataset-appropriate expectation (e.g. 583 kbp for the S.cer w303 20x PacBio reference run)
- Each long read yields an ordered contig-identifier list roughly threefold shorter than the original read length
- Compressed reads file exists in the ReadsInfoFrom_* format before proceeding to consensus
- Anchor thresholds are consistent: no contig with M < AdaptiveTh*Contig_Length or M < KmerCovTh is used as an anchor
- RemoveChimera is set to 1 only when long-read coverage exceeds 10x

## Limitations

- Parameter ranges are coverage-dependent: thresholds suited to 10x/20x PacBio (AdaptiveTh 0.001~0.01) fail at 50x-100x, where MinOverlap 50-150 and AdaptiveTh 0.01-0.02 are suggested
- k is not very flexible — 17 works well per the README
- At ~100x coverage, ChimeraTh and ContigTh must be raised from the default 1 to 2 to remove problematic reads and false contig anchors
- The first SparseAssembler run may not give the best contigs for complex genomes, requiring LD 1 fine-tuning of NodeCovTh/EdgeCovTh before DBG2OLC tuning

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01.: "For 10x/20x PacBio data: KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01."
- [methods] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed.: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed."
- [readme] Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding.: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
