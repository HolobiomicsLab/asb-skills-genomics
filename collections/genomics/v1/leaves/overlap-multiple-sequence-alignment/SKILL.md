---
name: overlap-multiple-sequence-alignment
description: Use when self-correcting PacBio or ONT long reads from a FASTA/FASTQ
  input without external short-read or reference data, or when benchmarking MSA segmentation
  strategies on high-error-rate (e.g. 12%) simulated read sets.
when_to_use_negative:
- The goal is polishing an existing assembly rather than correcting reads — use CONSENT-polish
  instead (CONSENT-polish --contigs ... --reads ... --out ...).
- Only N-highest-identity overlap selection may be inefficient when repeated regions
  are very long, so accuracy on long repeats should not be assumed from this skill
  alone.
- Correction of organisms far larger/ more complex than those benchmarked (e.g. full
  human datasets require ~10 days and ~5 TB disk for the overlaps file) — resource-constrained
  settings should not attempt this blindly.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CONSENT
  role: 'Performs the full self-correction pipeline: overlap computation, windowed
    MSA consensus with k-mer chaining, local de Bruijn graph polishing, and realignment
    to emit corrected reads'
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Computes long-read overlaps used to build alignment piles; required on the
    PATH (minimapIndex split default 500M)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:a1266b4178372ae00b62b7623853838f82077596e54f1b38fac2315a691d15d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - minimap2
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# overlap-multiple-sequence-alignment

## Summary

Compute windowed multiple sequence alignments (MSA) over overlapping regions of long reads to derive consensus sequences for self-correction, using CONSENT's k-mer-chaining segmentation strategy to shrink each MSA instance. This is the core correction mechanism of CONSENT, combining overlap piles, windowed MSA consensus, and local de Bruijn graph polishing.

## When to use

Use when self-correcting PacBio or ONT long reads from a FASTA/FASTQ input without external short-read or reference data, or when benchmarking MSA segmentation strategies on high-error-rate (e.g. 12%) simulated read sets.

## When NOT to use

- The goal is polishing an existing assembly rather than correcting reads — use CONSENT-polish instead (CONSENT-polish --contigs ... --reads ... --out ...).
- Only N-highest-identity overlap selection may be inefficient when repeated regions are very long, so accuracy on long repeats should not be assumed from this skill alone.
- Correction of organisms far larger/ more complex than those benchmarked (e.g. full human datasets require ~10 days and ~5 TB disk for the overlaps file) — resource-constrained settings should not attempt this blindly.

## Inputs

- FASTA or FASTQ file of long reads to self-correct
- Read technology flag: PB (PacBio) or ONT (Oxford Nanopore)
- Optional: reference genome for downstream error-rate evaluation (e.g. with ELECTOR)

## Outputs

- correctedReads.fasta — corrected long reads emitted after per-window consensus, de Bruijn graph polishing, and local realignment
- Alignments_dateTimeStamp.paf — temporary overlaps file produced by minimap2 (path settable via --tmpdir)

## How to apply

Run CONSENT-correct with --type PB (PacBio) or --type ONT on the long-read input; CONSENT first computes overlaps with minimap2 to define an alignment pile per read (bounded by --maxSupport 150). Each pile is divided into windows (--windowSize 500, overlap 50 bp, --minSupport 4) and an MSA is computed per window to derive a consensus; the k-mer-chaining segmentation (k-mer size 9 via --merSize, --anchorSupport 8, --minAnchors 2) further splits the MSA into smaller instances, which the paper reports cuts runtime 47x (5 h 31 min to 7 min) on 50x simulated E. coli reads with 12% error while improving post-correction error rate 1.77x. Each window consensus is then polished with a local de Bruijn graph (solid k-mer threshold --solid 4) and locally realigned to the original read to emit the corrected read. Decision point: toggle the segmentation strategy on/off to quantify its runtime/quality contribution, and evaluate outputs against a reference with ELECTOR when available.

## Related tools

- **CONSENT** (Performs the full self-correction pipeline: overlap computation, windowed MSA consensus with k-mer chaining, local de Bruijn graph polishing, and realignment to emit corrected reads) — https://github.com/morispi/CONSENT
- **minimap2** (Computes long-read overlaps used to build alignment piles; required on the PATH (minimapIndex split default 500M))

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- correctedReads.fasta exists and contains corrected reads for the input set
- Runtime comparison of segmented vs. unsegmented MSA reproduces the ~47x speedup reported for simulated 50x, 12%-error E. coli reads (5 h 31 min vs. 7 min)
- Post-correction error rate against the reference (e.g. via ELECTOR) improves ~1.77x over the baseline, with precision gain of roughly 0.15%
- Default parameters in the run log match windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2
- Alignment piles respect --maxSupport 150 and windows are only corrected when support >= minSupport (4) and anchors >= minAnchors (2)

## Limitations

- Selecting only the N highest identity overlaps per pile can be inefficient when repeated regions are long.
- Window size and minimum anchor count strongly affect runtime; an optimal runtime/quality compromise has not been established.
- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods; a full human dataset requires ~10 days and ~5 TB of disk space driven by the overlaps file.
- Corrected FASTA output has not been benchmarked on larger and more complex organisms beyond those tested.

## Evidence

- [intro] CONSENT indeed starts by computing multiple sequence alignments (MSA) between overlapping regions of the long reads, in order to compute consensus sequences.: "CONSENT indeed starts by computing multiple sequence alignments (MSA) between overlapping regions of the long reads, in order to compute consensus sequences."
- [discussion] This method is combined to an efficient k-mer chaining strategy, which allows to further divide the MSA into smaller instances: "This method is combined to an efficient k-mer chaining strategy, which allows to further divide the MSA into smaller instances"
- [readme] --windowSize INT, -l INT:      Size of the windows to process. (default: 500): "--windowSize INT, -l INT:      Size of the windows to process. (default: 500)"
- [intro] We also introduce an efficient segmentation strategy based on k-mer chaining, which allows to reduce the time footprint of the MSA.: "We also introduce an efficient segmentation strategy based on k-mer chaining, which allows to reduce the time footprint of the MSA."
