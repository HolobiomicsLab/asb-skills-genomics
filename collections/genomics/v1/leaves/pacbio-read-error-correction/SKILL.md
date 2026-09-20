---
name: pacbio-read-error-correction
description: Use when raw PacBio or ONT long reads (FASTA/FASTQ) contain sequencing
  errors that must be reduced via self-correction before assembly, mapping, or downstream
  benchmarking, without short-read or reference-based correction.
when_to_use_negative:
- The input is already-corrected reads or a polished assembly — CONSENT-polish, not
  self-correction, is the appropriate mode for raw assemblies.
- The goal is to correct reads using a reference genome or short reads (hybrid/reference-based
  correction, e.g. CoLoRMAP/HECiL style) rather than self-correction from the long
  reads alone.
- The dataset exceeds the scale that alignment-pile-based correctors like Canu or
  Daccord can handle (e.g. C. elegans / full human) and you need an alternative memory-lean
  strategy — although this is exactly where CONSENT is preferred, correction of larger
  and more complex organisms than human has not been performed.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: CONSENT
  role: 'Self-correction engine: computes overlaps, per-window MSAs, local de Bruijn
    graph polishing, and local realignment to output corrected reads via CONSENT-correct'
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Computes overlaps between long reads inside CONSENT (index split every 500M
    bases by default); also used to align corrected reads back to a reference for
    quality assessment
  repo: https://github.com/morispi/CONSENT
- name: ELECTOR
  role: Reference-alignment module used to evaluate corrected reads against a reference
    genome (error rate, recall, precision, identity, genome coverage)
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:87e7af0f19548b2902de0fd9adc4d383682110b5d4564111c6d2ea95a55457be
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
  - MECAT
  - ONT
  - PacBio
  - minimap2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# pacbio-read-error-correction

## Summary

Self-correction of PacBio (and ONT) long reads using CONSENT, which computes multiple sequence alignments between overlapping read regions, derives per-window consensus sequences, and polishes them with local de Bruijn graphs to lower the final error rate.

## When to use

Use when raw PacBio or ONT long reads (FASTA/FASTQ) contain sequencing errors that must be reduced via self-correction before assembly, mapping, or downstream benchmarking, without short-read or reference-based correction.

## When NOT to use

- The input is already-corrected reads or a polished assembly — CONSENT-polish, not self-correction, is the appropriate mode for raw assemblies.
- The goal is to correct reads using a reference genome or short reads (hybrid/reference-based correction, e.g. CoLoRMAP/HECiL style) rather than self-correction from the long reads alone.
- The dataset exceeds the scale that alignment-pile-based correctors like Canu or Daccord can handle (e.g. C. elegans / full human) and you need an alternative memory-lean strategy — although this is exactly where CONSENT is preferred, correction of larger and more complex organisms than human has not been performed.

## Inputs

- PacBio long reads in FASTA or FASTQ format (e.g. simulated 30x/60x E. coli, S. cerevisiae, C. elegans datasets or real S. cerevisiae PacBio reads SRR9617898)
- Sequencing technology flag (--type PB for PacBio, --type ONT for Oxford Nanopore)
- Optional output path for corrected reads and --tmpdir path for the overlaps file

## Outputs

- Corrected long reads as a FASTA file (result.fasta)
- Temporary overlaps file (Alignments_dateTimeStamp.paf)
- Alignment statistics when corrected reads are evaluated (e.g. with ELECTOR/minimap2): aligned-read proportion, alignment identity, N50, genome coverage

## How to apply

Clone the CONSENT repository with submodules and run install.sh, ensuring minimap2 is on the PATH. Run CONSENT-correct with --type PB for PacBio reads (or --type ONT for Nanopore reads), relying on defaults windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50. CONSENT first computes overlaps between long reads, divides overlapping regions into windows, computes an MSA-derived consensus per window via k-mer chaining, polishes the consensus with a local de Bruijn graph, then locally realigns it to the read to correct it. On the example simulated PacBio dataset (10x), correction takes ~2 min and up to ~750 MB RAM on 4 cores. For large datasets, set --tmpdir to external storage since the temporary overlaps file (Alignments_dateTimeStamp.paf) can reach ~5 TB for a full human dataset, and tune --nproc for parallel execution.

## Related tools

- **CONSENT** (Self-correction engine: computes overlaps, per-window MSAs, local de Bruijn graph polishing, and local realignment to output corrected reads via CONSENT-correct) — https://github.com/morispi/CONSENT
- **minimap2** (Computes overlaps between long reads inside CONSENT (index split every 500M bases by default); also used to align corrected reads back to a reference for quality assessment) — https://github.com/morispi/CONSENT
- **ELECTOR** (Reference-alignment module used to evaluate corrected reads against a reference genome (error rate, recall, precision, identity, genome coverage))

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- On the example simulated PacBio dataset (10x), the run completes in ~2 min using up to ~750 MB RAM on 4 cores.
- Reproduced post-correction error rates match Table 1 values: ~0.3111% (E. coli 30x), ~0.1833% (E. coli 60x), ~0.3858% (C. elegans 60x).
- Corrected reads align back to the reference with measurable alignment identity and genome coverage (on real S. cerevisiae/D. melanogaster/H. sapiens chr 1, CONSENT reached the largest coverage among compared tools).
- On a full human ONT dataset (~113 Gbp, reads up to 1.5 Mbp), expected outcomes are ~10 days runtime, <100 GB RAM, corrected-read N50 ~12,880 bp, ~98.4% aligned fraction, and ~93.5–93.86% identity.
- Output file is valid FASTA and the temporary overlaps file (Alignments_dateTimeStamp.paf) was produced at the --tmpdir location.

## Limitations

- The runtime of CONSENT's correction step tends to be higher than that of other state-of-the-art methods.
- Selecting only the N highest identity overlaps per pile may prove inefficient when the length of repeated regions grows longer.
- Window size and minimum number of anchors parameters significantly impact runtime; an optimal parameter compromise is not established.
- A full human dataset requires ~5 TB of disk for the overlaps file, needing external storage.
- CONSENT has been developed and tested on x86-64 GNU/Linux; support for any other platform has not been tested.
- Correction of larger and more complex organisms than those benchmarked has not yet been performed.

## Evidence

- [abstract] CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data.: "CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data."
- [discussion] CONSENT starts by computing overlaps between the long reads to correct.: "CONSENT starts by computing overlaps between the long reads to correct."
- [methods] --minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4): "--minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4)"
- [readme] ./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB: "./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB"
- [abstract] CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days.: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
