---
name: pairwise-sequence-alignment-of-derived-fragments
description: Use when small RNA-seq reads have been mapped to tRNA loci and you need
  to define the exact read-end/tRNA cleavage interface, classify tRNA halves vs tRF-5/tRF-3,
  or verify a conserved cleavage motif.
when_to_use_negative:
- When the question is tncRNA discovery/annotation from raw sRNA-seq rather than characterizing
  cleavage interfaces of already-mapped tncRNAs (use the full tncRNA Toolkit pipeline
  instead)
- When reads are not uniquely mappable to a single tRNA locus (multi-mapped reads
  aligned > 50 times are discarded by the upstream bowtie '-m 50' step), so cleavage
  position is ambiguous
- When studying non-tRNA-derived small RNAs such as miRNAs or piRNAs — the AG::TGG
  interface motif and anticodon-loop window are specific to tRNA cleavage
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: biopython
  role: pairwise2 module builds local alignments of tncRNAs to their tRNAs of origin
    using scoring 1 / -1 / gap-open -1 / gap-extend -0.5; installed as biopython v1.77
    in the toolkit
- name: tncRNA Toolkit
  role: upstream pipeline that produces the mapped tncRNA reads, tRNA annotations,
    and cleavage-position classifications (tRF-5s, tRF-3s, tRF-1, tRNA halves) that
    the alignments interrogate
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bowtie1
  role: maps tncRNA reads to the artificial tRNA genome ('--norc -v 0 -m 50 --best
    --strata') to determine each read's cleavage position relative to its progenitor
    tRNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:c7a15a17ef1deff9177902cfd65b772127ba6b67ed8370fe914f4eeaee0525c2
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
  - biopython
  - bowtie1
  - pandas
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pairwise-sequence-alignment-of-derived-fragments

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locally aligns tRNA-derived small RNA reads (tncRNAs) to their progenitor mature tRNA sequences to pinpoint the cleavage interface and detect the conserved tRF-5::tRNA motif (AG::TGG), demonstrating that plant tRNA cleavage is codon-dependent and specific rather than random degradation.

## When to use

Use when small RNA-seq reads have been mapped to tRNA loci and you need to define the exact read-end/tRNA cleavage interface, classify tRNA halves vs tRF-5/tRF-3, or verify a conserved cleavage motif.

## When NOT to use

- When the question is tncRNA discovery/annotation from raw sRNA-seq rather than characterizing cleavage interfaces of already-mapped tncRNAs (use the full tncRNA Toolkit pipeline instead)
- When reads are not uniquely mappable to a single tRNA locus (multi-mapped reads aligned > 50 times are discarded by the upstream bowtie '-m 50' step), so cleavage position is ambiguous
- When studying non-tRNA-derived small RNAs such as miRNAs or piRNAs — the AG::TGG interface motif and anticodon-loop window are specific to tRNA cleavage

## Inputs

- FASTA of unique mapped tncRNA reads with per-read counts (extracted from bowtie SAM flags 0 and 16)
- Mature tRNA annotations with 50 bp 5' and 3' flanks (from tRNAscan-SE predictions, pseudogenes and score < 50 removed)
- bowtie1 mapping results (locus, location, read length) for each tncRNA
- Progenitor tRNA sequences for alignment

## Outputs

- TSV table of cleavage-position counts and nucleotide compositions per tRNA isoacceptor
- Local alignments of each tncRNA to its tRNA of origin with read-end/interface bases annotated
- Detected tRF-5::tRNA interface motif (AG::TGG) calls and RPM-normalized abundance per progenitor tRNA

## How to apply

After bowtie1 mapping ('--norc -v 0 -m 50 --best --strata') assigns each uniquely mapped tncRNA read (with per-read counts from SAM flags 0 & 16) to a progenitor mature tRNA annotated with 50 bp 5'/3' flanks, compute the cleavage position from the locus, location, and read length — classifying tRNA halves by cleavage within the 7 nt anticodon-loop window (2 nt + 3 nt anticodon + 2 nt). Then build local alignments of each tncRNA against its tRNA of origin with the Biopython pairwise2 module using the scoring scheme identical = 1, non-identical = -1, gap-open = -1, gap-extend = -0.5, and inspect the read-end/tRNA-interface bases. The goal is to detect the conserved AG::TGG interface motif at tRF-5 5'-ends for tRF-5s of 15–19 nt across isoacceptors (e.g. GluCTC, GlyTCC, LeuTAA, ProCGG). Concurrently, tabulate nucleotide frequencies at cleavage positions per isoacceptor and normalize abundance as RPM = reads mapped to progenitor tRNA / (total mapped reads / 1e6); a non-uniform, codon-dependent nucleotide distribution (rather than uniform random cleavage) plus motif recovery validates the analysis against the paper's reported precise endoribonucleolytic cleavage pattern.

## Related tools

- **biopython** (pairwise2 module builds local alignments of tncRNAs to their tRNAs of origin using scoring 1 / -1 / gap-open -1 / gap-extend -0.5; installed as biopython v1.77 in the toolkit)
- **tncRNA Toolkit** (upstream pipeline that produces the mapped tncRNA reads, tRNA annotations, and cleavage-position classifications (tRF-5s, tRF-3s, tRF-1, tRNA halves) that the alignments interrogate) — https://github.com/skbinfo/tncRNA-Toolkit
- **bowtie1** (maps tncRNA reads to the artificial tRNA genome ('--norc -v 0 -m 50 --best --strata') to determine each read's cleavage position relative to its progenitor tRNA)

## Examples

```
from Bio import pairwise2; aln = pairwise2.align.localms(tncRNA_seq, tRNA_seq, 1, -1, -1, -0.5)
```

## Evaluation signals

- The output TSV shows a non-uniform, codon-dependent nucleotide distribution at cleavage positions across isoacceptors (GluCTC, GlyTCC, LeuTAA, ProCGG, etc.), not a uniform random pattern
- The conserved AG::TGG interface motif is detected at the 5'-ends of tRF-5s ranging between 15 and 19 nt across all plants examined
- tRNA halves are called only when cleavage falls within the 7 nt anticodon-loop window (2 nt + 3 nt anticodon + 2 nt)
- Local alignments recover the tncRNA as a contiguous subsequence of the progenitor tRNA (allowing the specified gap penalties) with cleavage position consistent between alignment and bowtie-derived locus/location
- RPM normalization yields values consistent with the paper's reported precise endoribonucleolytic cleavage pattern per isoacceptor

## Limitations

- Conventional RNA-seq suffers from tRNA modification biases; modification-induced mismatches can distort cleavage-position nucleotide calls, and tRNA-seq would be needed to overcome this
- Allowing even a single mismatch to accommodate sequencing errors risks base misidentification and inflation of false negatives, so the analysis relies on zero-mismatch mapping (-v 0)
- The existence of tRNAs with the same anticodon sequence (isodecoders) makes assignment of a tncRNA to a unique progenitor tRNA ambiguous
- The AG::TGG motif and codon-dependence were established for tRF-5s of 15–19 nt in six angiosperms; extrapolation to other length classes or organisms is not supported by this evidence

## Evidence
