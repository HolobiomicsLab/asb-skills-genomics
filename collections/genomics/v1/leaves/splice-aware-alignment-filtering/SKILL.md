---
name: splice-aware-alignment-filtering
description: Use when full-length PacBio CCS reads (e.g. tail-trimmed sample.out.fasta)
  must be spliced-aligned to mm10/GRCm38 and cleaned so downstream per-gene poly(A)
  site or non-A residue calls use only uniquely assigned reads.
when_to_use_negative:
- Input reads are short Illumina reads rather than full-length PacBio CCS reads from
  PAIso−seq libraries
- The analysis goal requires keeping multi-gene or chimeric/supplementary alignments
  rather than unique per-gene assignment
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Splice-aware alignment of clean CCS reads to the mouse mm10 reference using
    a pre-built .mmi index (v2.15-r905)
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: GMAP
  role: Alternative splice-aware aligner used to map tail-trimmed CCS reads to mm10
    before TAPIS APA analysis
- name: ccs
  role: Generates circular consensus reads (version 3.4.1) whose pass numbers are
    counted to enforce the ≥10-pass accuracy filter
  repo: https://github.com/PacificBiosciences/IsoSeq3
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:1ea21260ab7bbac31305f883cd6cb1bc5ab1a1fb9bacce8b80bbec0606b400d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - GMAP
  - ccs
  - ccs (version 3.4.1)
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# splice-aware-alignment-filtering

## Summary

Splice-aware alignment of full-length PAIso−seq CCS reads to the mouse mm10/GRCm38 genome followed by filtering of multi-mapping and supplementary alignments so that each remaining read can be confidently assigned to a single gene for poly(A) tail and APA analyses.

## When to use

Use when full-length PacBio CCS reads (e.g. tail-trimmed sample.out.fasta) must be spliced-aligned to mm10/GRCm38 and cleaned so downstream per-gene poly(A) site or non-A residue calls use only uniquely assigned reads.

## When NOT to use

- Input reads are short Illumina reads rather than full-length PacBio CCS reads from PAIso−seq libraries
- The analysis goal requires keeping multi-gene or chimeric/supplementary alignments rather than unique per-gene assignment

## Inputs

- Tail-trimmed clean CCS reads in FASTA (sample.out.fasta)
- Mus_musculus.GRCm38.dna.toplevel.chromosome.fa reference genome (GRCm38/mm10)
- PacBio movie CCS BAM (for pass counting via GetCCSpass.pl)
- Ensembl Mus musculus GRCm38 (release 92) genome and annotation

## Outputs

- Splice-aware alignments of clean CCS reads to mm10 (SAM/BAM via minimap2 or GMAP)
- Set of uniquely gene-assigned reads free of supplementary alignments
- Filtered alignments feeding poly(A) tail extraction and TAPIS APA detection

## How to apply

Build a splice-aware minimap2 index of the Mus_musculus.GRCm38.dna.toplevel.chromosome.fa reference (`minimap2 -x splice -t 20 -d Mus_musculus.mmi ...`, minimap2 v2.15-r905) or align with GMAP for the TAPIS APA branch. Restrict analysis to CCS reads with a pass number of 10 or more to ensure base-calling accuracy. After alignment, ignore alignments carrying a supplementary alignment tag and discard CCSs that map to multiple genes, so each retained read is uniquely attributable to one transcript. The resulting filtered, uniquely assigned alignments are the input for poly(A) tail clipping (clipped sequence ≥15 nt with ≥5 continuous adenosines) and TAPIS-based poly(A) site detection against Ensembl GRCm38 release 92 annotation.

## Related tools

- **minimap2** (Splice-aware alignment of clean CCS reads to the mouse mm10 reference using a pre-built .mmi index (v2.15-r905)) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **GMAP** (Alternative splice-aware aligner used to map tail-trimmed CCS reads to mm10 before TAPIS APA analysis)
- **ccs** (Generates circular consensus reads (version 3.4.1) whose pass numbers are counted to enforce the ≥10-pass accuracy filter) — https://github.com/PacificBiosciences/IsoSeq3

## Examples

```
minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa
```

## Evaluation signals

- Every retained alignment lacks a supplementary alignment tag
- No CCS read is assigned to more than one gene after filtering
- All analyzed CCS reads have pass count ≥10 in ccs.pass.txt
- Downstream TAPIS run reproduces 3511 genes with two poly(A) sites, 762 with three, and 220 with >3 sites on GV oocyte data
- Poly(A) tails called from clipped sequences meet the ≥15 nt and ≥5 continuous adenosine criteria

## Limitations

- Concatemer reads with internal primers must be detected and trimmed before alignment
- Discarding multi-gene CCSs reduces effective coverage; replicate correlations are expected to improve with deeper sequencing
- Alignment alone does not resolve the terminal 3′-end base of poly(A) tails, where non-A quantification is inaccurate

## Evidence

- [discussion] Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905): "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905)"
- [discussion] We require a pass number of 10 or more to ensure the accuracy of base calling.: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [discussion] the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [readme] minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
- [discussion] CCSs assigned to multiple genes were discarded.: "CCSs assigned to multiple genes were discarded."
