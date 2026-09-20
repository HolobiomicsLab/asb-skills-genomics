---
name: semiglobal-flank-alignment-with-edlib
description: Use when demultiplexing ONT FASTQ/FASTA reads whose structure is [flank]-barcode-[flank]
  and you must locate and extract the ligated 24-bp barcode between known flanking
  sequences before probability-divergence matching.
when_to_use_negative:
- The barcode kit was designed by PRO so that candidates similar to the flanking sequences
  were already removed — extraction is reliable there and no diagnostic extraction-accuracy
  experiment is needed
- Barcodes are not flanked by known constant sequences (e.g. embedded adapters without
  defined flanks), so flank-anchored substring extraction cannot localize the barcode
- You need demultiplexing itself rather than the extraction step — use PRO-dex (probability
  divergence) or Guppy 6.5.7 on the extracted barcodes instead
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: edlib 1.3.9
  role: Python library performing the semiglobal alignment of flanking sequences to
    each read, returning edit distances and start/end positions used to delimit the
    corrupted barcode
  repo: https://github.com/rztongr/PRO
- name: PRO
  role: Package whose demultiplexing pipeline consumes the extracted corrupted barcode
    via the Dm = min{Df, Db} rule and identifies it by probability divergence (PRO-dex)
  repo: https://github.com/rztongr/PRO
- name: deepsimulator 1.5
  role: Simulator that generated the B3 ONT reads on which the extraction-accuracy
    benchmark (35.25%, Fig. 3b) is evaluated
  repo: https://github.com/rztongr/PRO
- name: tqdm
  role: Progress bar for iterating the per-read alignment/extraction loop over large
    read sets
  repo: https://github.com/rztongr/PRO
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:e88a41801cab8950e0b5def7c965a293821dd79ebbe93c4ce12430a34f3ba02c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/rztongr/PRO
  tools:
  - PRO
  - Python
  - deepsimulator 1.5
  - edlib
  - edlib 1.3.9
  - tqdm
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# semiglobal-flank-alignment-with-edlib

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A barcode-extraction method from the PRO package: align the known flanking sequences (e.g. 'GGTGCTG' and 'TTAACCT' from ONT kit B3) to each nanopore read with edlib's semiglobal alignment to locate the interval between them, from which the corrupted barcode s' is extracted for downstream demultiplexing by probability divergence.

## When to use

Use when demultiplexing ONT FASTQ/FASTA reads whose structure is [flank]-barcode-[flank] and you must locate and extract the ligated 24-bp barcode between known flanking sequences before probability-divergence matching.

## When NOT to use

- The barcode kit was designed by PRO so that candidates similar to the flanking sequences were already removed — extraction is reliable there and no diagnostic extraction-accuracy experiment is needed
- Barcodes are not flanked by known constant sequences (e.g. embedded adapters without defined flanks), so flank-anchored substring extraction cannot localize the barcode
- You need demultiplexing itself rather than the extraction step — use PRO-dex (probability divergence) or Guppy 6.5.7 on the extracted barcodes instead

## Inputs

- Nanopore sequencing reads in FASTQ/FASTA (e.g. simulated with deepsimulator 1.5)
- FASTQ/FASTA file of known flanking sequences used in library preparation (e.g. 'GGTGCTG' rear top and 'TTAACCT' rear bottom for kit B3)
- True ligated 24-bp barcodes (for accuracy evaluation in simulation)

## Outputs

- Alignment spans and edit distances (Df, Db, startf, endf, startb, endb) per read
- Extracted corrupted barcode s' per read
- Accurate vs inaccurate extraction fractions (e.g. the 35.25% accurate-extraction pie chart of Fig. 3b)

## How to apply

For each read, align both flanking sequences to the full read using edlib (semiglobal mode) to obtain minimum edit distances Df and Db together with the alignment spans startf, endf, startb, endb. Apply the rule Dm = min{Df, Db} of Fig. 3 to choose which flanking alignment anchors the extraction, and take the read substring between the two alignments as the corrupted barcode s'. Optionally narrow the search window (PRO narrows to sequence′[35:77] and sequence′[35+startf:120]) to speed up alignment on long ONT reads. Pass the extracted s' to PRO-dex, which identifies the barcode by probability divergence; compare against Guppy 6.5.7 as the official baseline. Rationale: extraction quality critically depends on flanking sequences being dissimilar to barcodes — with kit B3, whose flanks resemble the barcodes, only 35.25% of barcodes were extracted without added errors (Fig. 3b), which is why PRO removes candidates semiglobal-alignment-close to the flanks (edit distances both <= 1) during barcode generation.

## Related tools

- **edlib 1.3.9** (Python library performing the semiglobal alignment of flanking sequences to each read, returning edit distances and start/end positions used to delimit the corrupted barcode) — https://github.com/rztongr/PRO
- **PRO** (Package whose demultiplexing pipeline consumes the extracted corrupted barcode via the Dm = min{Df, Db} rule and identifies it by probability divergence (PRO-dex)) — https://github.com/rztongr/PRO
- **deepsimulator 1.5** (Simulator that generated the B3 ONT reads on which the extraction-accuracy benchmark (35.25%, Fig. 3b) is evaluated) — https://github.com/rztongr/PRO
- **tqdm** (Progress bar for iterating the per-read alignment/extraction loop over large read sets) — https://github.com/rztongr/PRO

## Examples

```
import edlib; aln = edlib.align("GGTGCTG", read, mode="HW"); startf, endf = aln["locations"][0]
```

## Evaluation signals

- On B3 simulated reads, the fraction of barcodes extracted without added edit errors reproduces the reported 35.25% from Fig. 3b
- Each extracted barcode s' must lie strictly between the two flanking alignment spans (startf/endf and startb/endb) of the same read
- The chosen anchor flank follows the Dm = min{Df, Db} rule; per-read extraction should be deterministic given the edlib results
- Extraction should add no extra edit errors beyond those already present in s': edit distance between s' and the true ligated 24-bp barcode equals the read's barcode-region error count
- When PRO-designed kits are used (flank-similar candidates removed with both semiglobal edit distances <= 1), the accurate-extraction fraction should be substantially higher than 35.25%

## Limitations

- Extraction is unreliable when flanking sequences resemble the barcodes: for the official kit B3 only 35.25% of barcodes were extracted accurately (Fig. 3b)
- ONT's design strategy for kits B1, B2, and B3 does not ensure large probability divergence between barcodes, so PRO underperforms Guppy even beyond extraction errors
- Range narrowing of the flanking-alignment search (sequence′[35:77] and sequence′[35+startf:120]) is an efficiency heuristic; skipping it is slower on long reads, while wrong narrowing could miss alignments

## Evidence
