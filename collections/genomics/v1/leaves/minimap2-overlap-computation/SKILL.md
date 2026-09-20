---
name: minimap2-overlap-computation
description: Use when long reads (FASTA/FASTQ, PacBio or ONT) must be overlapped to
  build alignment piles before MSA-based self-correction or assembly polishing with
  CONSENT.
when_to_use_negative:
- 'When short reads are the input: minimap2 overlaps here serve CONSENT''s long-read
  self-correction (PacBio/ONT), not short-read alignment'
- 'When disk space is constrained and no external storage is available: the overlaps
  file alone can require ~5 TB for a full human dataset'
- When the goal is standalone read mapping/alignment rather than feeding CONSENT's
  correction or polishing pipeline
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Wraps minimap2 to compute overlaps that define per-read alignment piles for
    MSA-based correction and polishing
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Computes the long-read overlaps; its index can be split (--minimapIndex, default
    500M) and must be on the PATH
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:78dd202c8956bd29dc50fa2ee295e957730c1648767aa6a0e41482d9f2c8ab06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Miniasm
  - ONT
  - RACON
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# minimap2-overlap-computation

## Summary

Compute pairwise overlaps between long reads (PacBio or ONT) with minimap2, as the first step of CONSENT's self-correction/polishing pipeline; the resulting PAF alignment file defines the per-read alignment piles used for MSA-based consensus and correction.

## When to use

Use when long reads (FASTA/FASTQ, PacBio or ONT) must be overlapped to build alignment piles before MSA-based self-correction or assembly polishing with CONSENT.

## When NOT to use

- When short reads are the input: minimap2 overlaps here serve CONSENT's long-read self-correction (PacBio/ONT), not short-read alignment
- When disk space is constrained and no external storage is available: the overlaps file alone can require ~5 TB for a full human dataset
- When the goal is standalone read mapping/alignment rather than feeding CONSENT's correction or polishing pipeline

## Inputs

- Long reads in FASTA or FASTQ format (PacBio or ONT), passed as --in/--reads
- Contigs to polish in FASTA or FASTQ format (polishing mode only), passed as --contigs
- Technology flag --type PB or --type ONT (self-correction mode)

## Outputs

- Temporary overlaps file in PAF format (Alignments_dateTimeStamp.paf) in the tmpdir
- Corrected reads FASTA (--out of CONSENT-correct) or polished assembly FASTA (--out of CONSENT-polish) once downstream MSA/consensus steps complete

## How to apply

Run CONSENT-correct or CONSENT-polish, which internally invokes minimap2 (it must be on the PATH, e.g. via export PATH=$PWD/minimap2:$PATH) to compute overlaps between the input long reads or between long reads and contigs. Control the overlap computation with --minimapIndex (split the minimap2 index every INT input bases, default 500M) to bound memory on large datasets, and route the temporary overlaps file with --tmpdir (written as Alignments_dateTimeStamp.paf; on a full human dataset this file can reach ~5 TB, so an external drive may be needed). Once overlaps exist, select the N highest-identity overlaps per pile (capped by --maxSupport 150), divide overlapping regions into windows (--windowSize 500, --windowOverlap 50), and proceed to MSA consensus and local de Bruijn graph polishing with defaults --merSize 9, --solid 4, --anchorSupport 8, --minAnchors 2. Choose --type PB or --type ONT to match the reads' sequencing technology.

## Related tools

- **CONSENT** (Wraps minimap2 to compute overlaps that define per-read alignment piles for MSA-based correction and polishing) — https://github.com/morispi/CONSENT
- **minimap2** (Computes the long-read overlaps; its index can be split (--minimapIndex, default 500M) and must be on the PATH)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Alignments_dateTimeStamp.paf overlaps file is produced in the tmpdir without minimap2 'command not found' errors (PATH check: export PATH=$PWD/minimap2:$PATH)
- Correction of the example 10x simulated PacBio dataset completes in ~2 min using up to 750 MB RAM on 4 cores; polishing in ~15 sec using at most 150 MB RAM on 4 cores
- Each read's overlap pile does not exceed --maxSupport (default 150) overlaps, with the N highest-identity overlaps retained
- Windowing parameters hold: windows of --windowSize 500 with --windowOverlap 50, requiring --minAnchors 2 anchors and --anchorSupport 8 supporting sequences per chain
- Output FASTA (corrected reads or polished assembly) is non-empty and downstream QUAST-LG evaluation shows error reduction versus the raw input

## Limitations

- The overlaps file drives a ~5 TB disk space requirement on a full human dataset, though it can be stored on an external hard drive
- Selecting only the N highest-identity overlaps per pile may prove inefficient when the length of repeated regions grows longer
- CONSENT's correction-step runtime tends to be higher than other state-of-the-art methods
- CONSENT has been developed and tested only on x86-64 GNU/Linux; other platforms are untested

## Evidence

- [discussion] CONSENT starts by computing overlaps between the long reads to correct.: "CONSENT starts by computing overlaps between the long reads to correct."
- [readme] tmpdir option: "--tmpdir STRING, -t STRING:    Path where to store the temporary overlaps file (default: working directory, as Alignments_dateTimeStamp.paf)."
- [readme] minimap2 requirement: "Minimap2 available through you path."
- [discussion] 5 TB disk: "The 5 TB disk space requirement comes from the overlaps file, which can easily be stored on a external hard drive."
- [methods] minSupport default: "--minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4)"
