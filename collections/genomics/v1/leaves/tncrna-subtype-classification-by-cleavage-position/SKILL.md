---
name: tncrna-subtype-classification-by-cleavage-position
description: Use when small RNA-seq reads have been mapped to an artificial tRNA genome
  (mature tRNA ± 50 nt flanks) and each unique read must be assigned a tncRNA subtype
  based on where it cleaves from the pre-tRNA or mature tRNA.
when_to_use_negative:
- The input reads do not map exclusively to the artificial tRNA chromosomes (tRNA
  plus 50 nt leader and trailer) — ambiguous reads from non-tRNA regions must not
  be classified as tncRNAs.
- The goal is profiling other small RNA classes (miRNAs, piRNAs, rRNA-derived sRNAs)
  rather than tRNA-derived fragments — use a general annotation pipeline such as SPORTS1.0
  instead.
- The analysis targets human/animal datasets where tools like tDRmapper, MINTmap,
  or tRF2Cancer — trained and tested on human data — may be more appropriate.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: tncRNA Toolkit
  role: In-house workflow implementing the subtype classification logic, RPM quantification,
    read-count cut-off, and pairwise2 visualization
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Bowtie1
  role: Re-aligns unique mapped reads to mature tRNA + 50 bp leader/trailer features
    with '--norc -v 0 -m 50 --best --strata' (plus-strand, zero-mismatch, suppressing
    >50 alignments)
- name: samtools
  role: Fetches mapped reads from the SAM/BAM (flags 0 & 16) to build the unique-read
    FASTA with counts
- name: biopython
  role: pairwise2 module builds local alignments of each tncRNA to its sequence of
    origin (match 1, mismatch -1, gap-open -1, gap-extend -0.5)
- name: HAMR
  role: Predicts modified nucleoside sites that are attached to each classified tncRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:946b94cb4eeba0891629ed9ad0d0b0cf48abaaaae749f2d8601498f0fe327440
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie1
  - HAMR
  - biopython
  - samtools
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tncrna-subtype-classification-by-cleavage-position

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies mapped tRNA-derived small RNA reads into tncRNA subtypes (tRF-5, tRF-3 with/without CCA, tRF-1, tsRNA, leader tRF, and 5'/3' tRNA halves) based on cleavage position and origin relative to the mature tRNA and its 50 nt leader/trailer flanks, as implemented in the plant tncRNA Toolkit.

## When to use

Use when small RNA-seq reads have been mapped to an artificial tRNA genome (mature tRNA ± 50 nt flanks) and each unique read must be assigned a tncRNA subtype based on where it cleaves from the pre-tRNA or mature tRNA.

## When NOT to use

- The input reads do not map exclusively to the artificial tRNA chromosomes (tRNA plus 50 nt leader and trailer) — ambiguous reads from non-tRNA regions must not be classified as tncRNAs.
- The goal is profiling other small RNA classes (miRNAs, piRNAs, rRNA-derived sRNAs) rather than tRNA-derived fragments — use a general annotation pipeline such as SPORTS1.0 instead.
- The analysis targets human/animal datasets where tools like tDRmapper, MINTmap, or tRF2Cancer — trained and tested on human data — may be more appropriate.

## Inputs

- SAM/BAM file of sRNA-seq reads mapped to the artificial tRNA genome (mature tRNA + 50 nt leader/trailer artificial chromosomes)
- Quality-filtered, adapter-trimmed single-end small RNA FASTQ file
- Bowtie1 index of the artificial genome (mature tRNA ± 50 nt flanks as artificial chromosomes, headers >chr[Num], >chrMt, >chrPt)
- HAMR-predicted tRNA modification-site annotations

## Outputs

- Table of unique mapped reads with tncRNA subtype assignment (tRF-5, tRF-3, tRF-3-CCA, tRF-1, tsRNA, leader tRF, 5'/3' tRH, tRH-CCA), locus, location, and length relative to mature tRNA and flanks
- Per-tncRNA RPM quantification (reads mapped to progenitor tRNA / Per Million Factor)
- tncRNA sequences annotated with modification sites predicted by HAMR
- Local (pairwise2) alignments of each tncRNA to its sequence of origin for visualization

## How to apply

First, fetch mapped reads from the SAM/BAM file with samtools using flags 0 and 16, and build a FASTA of unique reads with per-sequence counts. Re-align these unique reads to mature tRNA plus 50 bp leader/trailer features using bowtie1 with '--norc -v 0 -m 50 --best --strata', keeping only plus-strand zero-mismatch alignments and discarding reads aligning more than 50 times (rationale: even a single mismatch can cause base misidentification and inflate false negatives). Determine each read's locus, location, and length relative to the mature tRNA, 5' flank, and 3' flank, then assign the subtype: pre-tRNA leader/trailer cleavage yields leader tRF and tRF-1/tsRNA; mature-tRNA cleavage in the D region yields tRF-5, in the T region yields tRF-3 (with/without CCA); anticodon-loop cleavage yields 5'/3' tRHs, with tRNA halves classified by cleavage at the anticodon loop (2 nt + 3 nt anticodon + 2 nt = 7 nt). Attach HAMR-predicted modification-site information to each classified tncRNA, quantify it as RPM (Per Million Factor = total mapped reads / 10^6; RPM = reads mapped to progenitor tRNA / PMF), and build a biopython pairwise2 local alignment (identical = 1, non-identical = -1, gap-open = -1, gap-extend = -0.5) for visualization. Reads are only called tncRNAs if they map exclusively to the artificial tRNA chromosomes and pass the default read-count cut-off of 10.

## Related tools

- **tncRNA Toolkit** (In-house workflow implementing the subtype classification logic, RPM quantification, read-count cut-off, and pairwise2 visualization) — https://github.com/skbinfo/tncRNA-Toolkit
- **Bowtie1** (Re-aligns unique mapped reads to mature tRNA + 50 bp leader/trailer features with '--norc -v 0 -m 50 --best --strata' (plus-strand, zero-mismatch, suppressing >50 alignments))
- **samtools** (Fetches mapped reads from the SAM/BAM (flags 0 & 16) to build the unique-read FASTA with counts)
- **biopython** (pairwise2 module builds local alignments of each tncRNA to its sequence of origin (match 1, mismatch -1, gap-open -1, gap-extend -0.5))
- **HAMR** (Predicts modified nucleoside sites that are attached to each classified tncRNA)

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads>.fastq -o <output dir>
```

## Evaluation signals

- Every reported tncRNA maps exclusively to the artificial tRNA chromosomes (tRNA ± 50 nt flank).
- Each tncRNA carries a single, unambiguous subtype assignment derived from its cleavage position and origin.
- All classified tncRNAs pass the default read-count cut-off of 10 (adjustable via the -c option).
- No read classified as a tncRNA aligned more than 50 times (bowtie1 -m 50 suppression enforced).
- RPM values are reproducible from Per Million Factor = total mapped reads / 10^6 and reads mapped to the progenitor tRNA.

## Limitations

- Allowing even a single mismatch during mapping may lead to base misidentification and inflation of false negatives; the pipeline therefore uses zero-mismatch (-v 0) re-alignment for classification.
- Conventional RNA-seq suffers from tRNA modification biases, which can affect detection; tRNA-seq would be advantageous.
- There is no single standardized nomenclature for tncRNA subtypes, so subtype labels follow the classical terms (tRF-5s, tRF-3s, tRF-1, tRNA halves).
- In-silico predictions still require experimental validation to distinguish bona fide tncRNAs.

## Evidence
