---
name: de-bruijn-contig-kmer-indexing
description: Use when compressing long erroneous PacBio/ONT reads against accurate
  DBG contigs with DBG2OLC, or when a contig-to-k-mer lookup must exclude ambiguous
  multi-contig k-mers before read anchoring.
when_to_use_negative:
- The short-read contigs are not raw DBG contigs (e.g. gap-closed or scaffolded),
  since the repeat-resolving heuristics introduce errors that propagate to the backbone
- You have no NGS contigs and only long reads available for a non-hybrid assembly
  task
- The long-read dataset is already error-corrected or compressed; this skill targets
  raw erroneous 3GS reads
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Builds the 17-mer index over contigs, excludes multi-contig k-mers, compresses
    long reads into ordered contig-identifier lists, and produces the overlap/layout
    backbone
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Preassembles the ~50x Illumina short reads into the accurate DBG contigs that
    are indexed (Contigs.txt)
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:58f4984baca63eda1fab068ebf6d94666d5455239e5dde3b06cb0aa9fee15d0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# de-bruijn-contig-kmer-indexing

## Summary

Build a k-mer index over preassembled NGS DBG contigs (k=17) so long 3GS reads can be compressed into ordered contig-identifier lists, excluding k-mers shared by multiple contigs to avoid ambiguous anchors.

## When to use

Use when compressing long erroneous PacBio/ONT reads against accurate DBG contigs with DBG2OLC, or when a contig-to-k-mer lookup must exclude ambiguous multi-contig k-mers before read anchoring.

## When NOT to use

- The short-read contigs are not raw DBG contigs (e.g. gap-closed or scaffolded), since the repeat-resolving heuristics introduce errors that propagate to the backbone
- You have no NGS contigs and only long reads available for a non-hybrid assembly task
- The long-read dataset is already error-corrected or compressed; this skill targets raw erroneous 3GS reads

## Inputs

- SparseAssembler DBG contigs in fasta format (Contigs.txt), raw contigs without gap closing or scaffolding
- 3GS long reads in fasta format (e.g. Pacbio_20x.fasta, 10x-20x PacBio coverage)
- k-mer size parameter (k 17 recommended)

## Outputs

- 17-mer index over contigs held in memory (multi-contig k-mers excluded)
- Per-read compressed representations as ordered contig-identifier lists (~3x shorter than the original reads)
- Overlap/layout backbone (backbone_raw.fasta) from scoring compressed-read overlaps with MinOverlap

## How to apply

First preassemble the 50x Illumina reads with SparseAssembler (e.g. LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0) to produce raw DBG contigs (Contigs.txt), avoiding repeat-resolved/scaffolded contigs whose heuristics introduce errors. Feed Contigs.txt to DBG2OLC with k 17 so it stores a 17-mer index over the contigs — for a 3 Gbp human dataset this index occupies about 70 GB of memory. During indexing, k-mers that appear in multiple contigs are excluded so that a matched k-mer uniquely identifies one contig. Anchoring then applies AdaptiveTh 0.0001 (M < AdaptiveTh*Contig_Length disqualifies the contig) and KmerCovTh 2 (M < KmerCovTh disqualifies) to decide whether a contig can anchor a long read, and RemoveChimera 1 prunes chimeric reads given >10x coverage.

## Related tools

- **DBG2OLC** (Builds the 17-mer index over contigs, excludes multi-contig k-mers, compresses long reads into ordered contig-identifier lists, and produces the overlap/layout backbone) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Preassembles the ~50x Illumina short reads into the accurate DBG contigs that are indexed (Contigs.txt)) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- The compressed-reads output exists in the ReadsInfoFrom_* format and each read is an ordered contig-identifier list roughly threefold shorter than the original read
- N50 of backbone_raw.fasta is acceptable (583 kbp in the reference S.cer w303 run with k 17, AdaptiveTh 0.0001, KmerCovTh 2, MinOverlap 20, RemoveChimera 1); if unsatisfied, keep tuning parameters and do not proceed to consensus
- No ambiguous anchors: k-mers shared by multiple contigs are excluded from the index, so each matched k-mer maps to a single contig
- Tuned parameters stay in the suggested ranges: for 10x/20x PacBio data KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001~0.01; k 17 works well

## Limitations

- For a 3 Gbp human dataset the 17-mer index occupies ~70 GB of memory and compressing/aligning the 30x longest PacBio reads takes 37 CPU hours
- Contigs must be raw DBG contigs; gap closing or scaffolding introduces errors that can yield poor final results
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because such tools introduce errors
- The first SparseAssembler run may not generate the best result; NodeCovTh/EdgeCovTh may need fine-tuning with LD 1

## Evidence

- [results] k-mer index over contigs: "DBG2OLC occupies 70 GB memory to store the 17-mer index, and takes 37 CPU hours to compress and align the 30x longest PacBio reads"
- [methods] AdaptiveTh definition: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh definition: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] raw contig requirement: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
- [readme] reference invocation: "./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta"
