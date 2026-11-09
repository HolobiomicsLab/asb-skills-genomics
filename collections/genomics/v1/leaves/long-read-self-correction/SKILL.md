---
name: long-read-self-correction
description: Use when you have FASTA/FASTQ long reads (PacBio or ONT) with raw per-base
  error rates (e.g. ~12%) and need corrected reads as input for assembly or downstream
  variant/coverage analyses.
when_to_use_negative:
- Your goal is to fix consensus-level assembly errors rather than per-read errors
  — use CONSENT-polish on contigs instead of CONSENT-correct on reads.
- The input is already-corrected reads or short reads; CONSENT is a self-correction
  method designed for raw PacBio/ONT long reads, not short-read polishing.
- You need very fast correction of short/easy datasets at the expense of output volume
  — other tools may outpace CONSENT whose correction-step runtime tends to be higher
  than some state-of-the-art methods.
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: CONSENT
  role: 'Primary self-correction engine: computes overlaps (CONSENT-correct), builds
    windowed MSAs with k-mer-chaining segmentation, polishes with local de Bruijn
    graphs, and locally realigns consensus back to reads; also offers CONSENT-polish
    for raw assemblies.'
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Used internally by CONSENT to compute long-read overlaps and to realign corrected
    reads back to the reference for downstream evaluation (N50, identity, aligned
    fraction).
- name: ELECTOR
  role: Evaluation harness built on minimap2 that scores corrected reads against the
    reference (post-correction error rate, recall, precision, N50, genome coverage)
    to judge correction quality.
- name: Flye
  role: Downstream assembler used to build assemblies from CONSENT-corrected reads;
    CONSENT correction was shown to improve Flye assembly quality.
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:5a8883e48024ece693a76dd0ae1874e9b952fcabca9736f26b213940c3a6f831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - Daccord
  - ELECTOR
  - FLAS
  - Flye
  - MECAT
  - ONT
  - minimap2
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# long-read-self-correction

## Summary

Self-correct long reads (PacBio or ONT) with CONSENT, which computes overlapping-read alignment piles, splits them into windows, derives per-window MSA consensuses, polishes them with local de Bruijn graphs, and locally realigns the polished consensus back to each read.

## When to use

Use when you have FASTA/FASTQ long reads (PacBio or ONT) with raw per-base error rates (e.g. ~12%) and need corrected reads as input for assembly or downstream variant/coverage analyses.

## When NOT to use

- Your goal is to fix consensus-level assembly errors rather than per-read errors — use CONSENT-polish on contigs instead of CONSENT-correct on reads.
- The input is already-corrected reads or short reads; CONSENT is a self-correction method designed for raw PacBio/ONT long reads, not short-read polishing.
- You need very fast correction of short/easy datasets at the expense of output volume — other tools may outpace CONSENT whose correction-step runtime tends to be higher than some state-of-the-art methods.

## Inputs

- Long reads in FASTA or FASTQ format (PacBio or Oxford Nanopore), e.g. example/reads.fasta
- Optionally, a reference genome (E. coli, S. cerevisiae, D. melanogaster, H. sapiens) for ELECTOR/minimap2-based evaluation of corrected reads
- Optional raw assembly FASTA when using the polishing mode (example/rawAssembly.fasta)

## Outputs

- correctedReads.fasta — long reads after windowed MSA consensus + local de Bruijn graph polishing + local realignment
- Alignments_dateTimeStamp.paf — intermediate overlaps file (needs ~5 TB disk for a full human dataset)
- Optionally, ELECTOR/minimap2 alignment statistics (aligned fraction, identity, N50) used to validate correction quality

## How to apply

Run ./CONSENT-correct --in longReads --out correctedReads.fasta, setting --type PB for PacBio data or --type ONT for Nanopore data; CONSENT first computes overlaps between long reads with minimap2 (index split every 500M bases by default), then divides each read's alignment pile into windows (default windowSize 500, windowOverlap 50) and computes a multiple sequence alignment and consensus per window (minSupport 4, maxSupport 150, merSize 9, anchorSupport 8, minAnchors 2). Each window consensus is polished with a local de Bruijn graph using a solid k-mer threshold of 4 to remove remaining errors, and is finally locally realigned to the read to emit correctedReads.fasta. Decide at runtime between correction-only (CONSENT-correct) and assembly polishing (CONSENT-polish) depending on whether the target is raw reads or contigs; on a full human ONT dataset, point --tmpdir to external storage since the overlaps file requires ~5 TB of disk, and expect ~10 days runtime and <100 GB RAM with --nproc parallelism. Rationale: the k-mer-chaining segmentation strategy reduces MSA time footprint by 47x (5 h 31 min to 7 min on a 50x 12%-error simulated E. coli set) while decreasing post-correction error rate 1.77-fold versus unsegmented MSA.

## Related tools

- **CONSENT** (Primary self-correction engine: computes overlaps (CONSENT-correct), builds windowed MSAs with k-mer-chaining segmentation, polishes with local de Bruijn graphs, and locally realigns consensus back to reads; also offers CONSENT-polish for raw assemblies.) — https://github.com/morispi/CONSENT
- **minimap2** (Used internally by CONSENT to compute long-read overlaps and to realign corrected reads back to the reference for downstream evaluation (N50, identity, aligned fraction).)
- **ELECTOR** (Evaluation harness built on minimap2 that scores corrected reads against the reference (post-correction error rate, recall, precision, N50, genome coverage) to judge correction quality.)
- **Flye** (Downstream assembler used to build assemblies from CONSENT-corrected reads; CONSENT correction was shown to improve Flye assembly quality.)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- correctedReads.fasta exists and is non-empty after CONSENT-correct completes.
- On the bundled example (10x simulated PacBio), the run takes about 2 min and uses up to ~750 MB RAM on 4 cores.
- ELECTOR-reported post-correction error rate drops to roughly the published values: ~0.31% (E. coli 30x), ~0.18% (E. coli 60x), ~0.39% (C. elegans 60x) with recall/precision in reported ranges.
- With the k-mer-chaining segmentation strategy enabled, the MSA runtime on a simulated 50x 12%-error E. coli set shrinks ~47x (5 h 31 min -> 7 min) with a 1.77x reduction in post-correction error rate and ~0.15% precision gain.
- On the full human ONT set, N50 ~12,880 bp, aligned fraction ~98.4%, identity ~93.86%, runtime ~10 days, peak RAM <100 GB, matching Table 5.

## Limitations

- Runtime of the correction step tends to be higher than some other state-of-the-art methods; CONSENT is best justified on ONT/ultra-long data where it is the only method that scales to a 113 Gbp human dataset in 10 days.
- Selecting only the N highest-identity overlaps per alignment pile may be inefficient when repeated regions grow long.
- Window size and minimum anchors significantly affect runtime, and no optimal runtime/quality parameter compromise is established.
- Correct-then-assemble on a full human dataset required >17 days and >1 TB RAM, whereas assemble-then-polish (CONSENT-polish) ran in 7.5 days with <400 GB RAM and yielded better NGA50 and fewer errors per 100 kbp.
- Only tested on x86-64 GNU/Linux; other platforms are unsupported.

## Evidence

- [intro] CONSENT indeed starts by computing multiple sequence alignments (MSA) between: "CONSENT indeed starts by computing multiple sequence alignments (MSA) between overlapping regions of the long reads, in order to compute consensus sequences."
- [intro] We also introduce an efficient segmentation strategy based on k-mer chaining: "We also introduce an efficient segmentation strategy based on k-mer chaining, which allows to reduce the time footprint of the MSA."
- [methods] --minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4): "--minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4)"
- [readme] This should take about 2 min and use up to 750 MB of RAM, using 4 cores.: "This should take about 2 min and use up to 750 MB of RAM, using 4 cores."
- [results] CONSENT managed to process the full 113 Gbp ONT human dataset (reads up to 1.5 Mbp) in 10 days: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
