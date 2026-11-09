---
name: edit-distance-flank-localization
description: Use when reads carry short barcodes embedded between known flanking sequences
  (e.g. ONT kit B3 flanks 'GGTGCTG'/'TTAACCT' around a 24-bp barcode) and the corrupted
  barcode s' must be located in each FASTQ read before demultiplexing by probability
  divergence.
when_to_use_negative:
- The reads are already-extracted, corrupted barcodes that only need classification
  — that is PRO's demultiplexing step (PRO-dex, probability divergence), not flank
  localization.
- The library preparation does not ligate flanking sequences around the barcode (no
  flank motifs to anchor the extraction), so there is nothing to align.
- Data are PacBio reads — the article states no tests on PacBio data have been performed,
  so extraction behavior is unvalidated there.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: edlib 1.3.9
  role: Python library used to align the two flanking sequences to each read and return
    minimum edit distances and start/end alignment positions for barcode extraction
- name: PRO
  role: Software package whose demultiplexing pipeline uses this flanking-sequence-based
    extraction of corrupted barcodes before probability-divergence identification
  repo: https://github.com/rztongr/PRO
- name: deepsimulator 1.5
  role: Generator of the simulated nanopore sequencing data (FASTQ) on which the extraction
    accuracy (35.25% for kit B3) is validated
  repo: https://github.com/liyu95/DeepSimulator
- name: tqdm
  role: Progress-bar companion package installed alongside edlib for running the extraction
    over read collections
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:87269347023361836007a4a2bab5735f14648643795b4468221d36c817ab9a1d
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
  - edlib 1.3.9
  - tqdm
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# edit-distance-flank-localization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts corrupted barcodes from nanopore reads by aligning the flanking sequences with the edlib Python library (semiglobal edit distance, Dm = min{Df, Db}) and excising the interval between the two flank alignments, as used in PRO's demultiplexing procedure.

## When to use

Use when reads carry short barcodes embedded between known flanking sequences (e.g. ONT kit B3 flanks 'GGTGCTG'/'TTAACCT' around a 24-bp barcode) and the corrupted barcode s' must be located in each FASTQ read before demultiplexing by probability divergence.

## When NOT to use

- The reads are already-extracted, corrupted barcodes that only need classification — that is PRO's demultiplexing step (PRO-dex, probability divergence), not flank localization.
- The library preparation does not ligate flanking sequences around the barcode (no flank motifs to anchor the extraction), so there is nothing to align.
- Data are PacBio reads — the article states no tests on PacBio data have been performed, so extraction behavior is unvalidated there.

## Inputs

- Nanopore read FASTQ files of ligated-barcode libraries (e.g. simulated with deepsimulator 1.5)
- Known flanking sequences (FASTA/text), e.g. 'GGTGCTG' (rear top) and 'TTAACCT' (rear bottom) for official kit B3
- True ligated 24-bp barcode sequences for ground-truth evaluation

## Outputs

- Extracted corrupted barcode s' per read (the interval between the two flank alignments)
- Per-read flank alignment coordinates and edit distances (startf, endf, Df, startb, endb, Db) under the Dm = min{Df, Db} rule
- Accurate-vs-inaccurate extraction fractions (e.g. 35.25% accurate for kit B3), plottable as a matplotlib pie chart

## How to apply

Load the simulated nanopore reads (e.g. B3 reads generated with deepsimulator 1.5) together with the true ligated 24-bp barcodes and the two known flanking sequences. For each read, align the front and rear flanking sequences to the full read using edlib 1.3.9 in semiglobal mode without range narrowing, obtaining minimum edit distances Df and Db and positions startf, endf, startb, endb. Apply the Dm = min{Df, Db} rule (Fig. 3) to decide which flank alignment anchors the extraction, then cut out the corrupted barcode s' lying between the two flank alignments. Optionally narrow the search range to aligns of sequence′[35:77] and sequence′[35+startf:120] to reduce spurious flank matches on long reads. Judge success per read by comparing s' to the true ligated barcode: a barcode is 'accurately extracted' when extraction adds no extra edit errors; for kit B3 the expected accurate fraction is 35.25% (Fig. 3b), quantified with a matplotlib pie chart of accurate vs inaccurate extractions.

## Related tools

- **edlib 1.3.9** (Python library used to align the two flanking sequences to each read and return minimum edit distances and start/end alignment positions for barcode extraction)
- **PRO** (Software package whose demultiplexing pipeline uses this flanking-sequence-based extraction of corrupted barcodes before probability-divergence identification) — https://github.com/rztongr/PRO
- **deepsimulator 1.5** (Generator of the simulated nanopore sequencing data (FASTQ) on which the extraction accuracy (35.25% for kit B3) is validated) — https://github.com/liyu95/DeepSimulator
- **tqdm** (Progress-bar companion package installed alongside edlib for running the extraction over read collections)

## Examples

```
import edlib; r = edlib.align("GGTGCTG", read, mode="HW", task="locations"); startf, endf = r["locations"][0]
```

## Evaluation signals

- For official kit B3, the fraction of barcodes extracted with no extra edit errors should reproduce the reported 35.25% from Fig. 3b.
- Each extracted s' must equal the true ligated 24-bp barcode up to sequencing errors introduced by the simulator, with zero edits added by the extraction step itself.
- The Dm = min{Df, Db} rule must consistently select the flank alignment with the smaller edit distance before cutting the barcode interval.
- With range narrowing to sequence′[35:77] and sequence′[35+startf:120], extraction accuracy should improve relative to the no-narrowing run on the same reads.
- Flank alignment calls must be present (startf/endf, startb/endb found) for a read to yield any extracted barcode at all.

## Limitations

- When flanking sequences resemble the barcode sequences (as in official kit B3), extraction is unreliable: only 35.25% of barcodes are accurately extracted, which is why PRO underperforms Guppy on B1, B2, and B3.
- This similarity also reflects ONT's design strategy for B1–B3, which does not ensure large probability divergence between barcodes, compounding demultiplexing failures.
- No tests on PacBio data have been performed, so the extraction module's applicability outside ONT data is unverified.

## Evidence
