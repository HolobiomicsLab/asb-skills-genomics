---
name: k-mer-chaining-segmentation
description: Use when performing CONSENT long-read self-correction on long or ultra-long
  reads and MSA computation is the runtime bottleneck; input is PacBio (--type PB)
  or ONT long reads in FASTA/FASTQ requiring fast, scalable consensus computation.
when_to_use_negative:
- Input is short reads or already-polished assemblies where self-correction of long
  reads is not applicable (CONSENT's correction step runtime can exceed other methods
  on smaller datasets where the MSA is not the bottleneck).
- Datasets with very long repeated regions where selecting only the N highest identity
  overlaps may be inefficient and segmentation may not help.
- Use CONSENT-polish instead when the goal is correcting a raw assembly rather than
  self-correcting individual long reads.
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: CONSENT
  role: Implements the k-mer-chaining segmentation of windowed MSAs within CONSENT-correct
    for long-read self-correction
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Computes the long-read overlaps that are divided into windows and segmented
    prior to MSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:92d4834b34b44b4af49de16769b05056215fa9422fcda30c8dd2bd49d9b0d15f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - minimap2
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# k-mer-chaining-segmentation

## Summary

An efficient segmentation strategy based on k-mer chaining that splits each windowed multiple sequence alignment (MSA) into smaller instances, drastically reducing the time footprint of MSA-based long-read self-correction in CONSENT while preserving or improving correction quality.

## When to use

Use when performing CONSENT long-read self-correction on long or ultra-long reads and MSA computation is the runtime bottleneck; input is PacBio (--type PB) or ONT long reads in FASTA/FASTQ requiring fast, scalable consensus computation.

## When NOT to use

- Input is short reads or already-polished assemblies where self-correction of long reads is not applicable (CONSENT's correction step runtime can exceed other methods on smaller datasets where the MSA is not the bottleneck).
- Datasets with very long repeated regions where selecting only the N highest identity overlaps may be inefficient and segmentation may not help.
- Use CONSENT-polish instead when the goal is correcting a raw assembly rather than self-correcting individual long reads.

## Inputs

- Long-read dataset in FASTA or FASTQ format (PacBio or Oxford Nanopore)
- Reference genome for downstream quality evaluation (e.g. via ELECTOR)

## Outputs

- correctedReads.fasta containing self-corrected long reads
- Runtime and quality metrics: post-correction error rate and precision versus an unsegmented MSA baseline

## How to apply

Run CONSENT-correct on the long-read FASTA/FASTQ (e.g. --type PB for PacBio data): overlaps computed with minimap2 are split into windows (windowSize 500, minSupport 4, maxSupport 150, merSize 9), and the k-mer chaining segmentation further divides each window's MSA into smaller instances before consensus computation. Tune chaining parameters anchorSupport (default 8, minimum number of sequences supporting consecutive anchors Ai–Ai+1) and minAnchors (default 2, minimum anchors per window to allow consensus computation). The consensus is then polished with a local de Bruijn graph (solid k-mer threshold 4) and locally realigned to the original read to emit correctedReads.fasta. Rationale: the k-mer chaining strategy 'allows to further divide the MSA into smaller instances', yielding a 47x runtime reduction (5 h 31 min to 7 min on simulated 50x, 12%-error E. coli PacBio reads) while decreasing post-correction error rate 1.77x and increasing precision ~0.15% (evaluated with ELECTOR).

## Related tools

- **CONSENT** (Implements the k-mer-chaining segmentation of windowed MSAs within CONSENT-correct for long-read self-correction) — https://github.com/morispi/CONSENT
- **minimap2** (Computes the long-read overlaps that are divided into windows and segmented prior to MSA)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Runtime of the segmentation-enabled run should be roughly ~7 min versus ~5 h 31 min for regular MSA on a simulated 50x coverage, 12%-error E. coli PacBio read set (47x speedup).
- Post-correction error rate measured by ELECTOR should decrease by ~1.77x relative to the unsegmented baseline.
- Post-correction precision should increase by almost 0.15% relative to the unsegmented baseline.
- correctedReads.fasta must exist, be valid FASTA, and contain corrected versions of the input reads.
- Chaining parameters remain at defaults (anchorSupport 8, minAnchors 2, merSize 9) unless explicitly tuned; windows failing the minAnchors threshold are excluded from consensus computation.

## Limitations

- Runtime of CONSENT's correction step tends to be higher than that of other state-of-the-art methods, even with segmentation.
- The strategy of selecting only the N highest identity overlaps might prove inefficient when the length of repeated regions grows longer.
- Window size and minimum number of anchors parameters significantly impact runtime; an optimal parameter compromise between runtime and quality is not established.
- K-mer chaining requires a minimum of minAnchors (default 2) anchors per window; windows below this threshold do not get a consensus computed.

## Evidence

- [intro] Introduce an efficient segmentation strategy based on k-mer chaining to reduce the time footprint of the MSA and enable scalability to ultra-long reads: "We also introduce an efficient segmentation strategy based on k-mer chaining, which allows to reduce the time footprint of the MSA."
- [discussion] k-mer chaining strategy to split MSA into smaller instances: "This method is combined to an efficient k-mer chaining strategy, which allows to further divide the MSA into smaller instances"
- [methods] anchorSupport parameter definition: "--anchorSupport INT, -c INT:   Minimum number of sequences supporting (Ai) - (Ai+1) to keep the two anchors in the chaining. (default: 8)"
- [methods] minAnchors parameter definition: "--minAnchors INT, -a INT:      Minimum number of anchors in a window to allow consensus computation. (default: 2)"
- [readme] CONSENT-correct invocation from README: "./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB"
