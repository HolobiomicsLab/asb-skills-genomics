---
name: transcription-start-site-validation
description: Use when a long-read-derived isoform catalog (e.g. lrCaptureSeq GFF)
  needs independent 5'-end validation and matched-tissue CAGE-seq data (e.g. adult
  mouse retina, DDBJ DRA002410) is available.
when_to_use_negative:
- No matched-tissue CAGE-seq data exists for the tissue from which the isoform catalog
  was derived — coverage evidence would not be an independent corroboration.
- The isoform catalog has not been filtered (no exon-count, truncation, or abundance
  filtering), so unvalidated low-quality/truncated transcripts would confound the
  validation fraction.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HISAT2
  role: Splice-aligns short CAGE-seq reads to the mm10 (GRCm38) mouse reference genome
- name: BedTools
  role: coverage/intersect quantifies CAGE 5'-end coverage over each isoform's transcription
    start site
- name: R
  role: Runs the IsoPops filtering steps that produce the fullfilter GFF used for
    TSS coverage testing and writes the coverage CSV
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:2bf6ef1d89e5b1c69255278f6ed26744b5b5d45579acc241085b47be47869a7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BedTools
  - HISAT2
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# transcription-start-site-validation

## Summary

Independently corroborates transcription start sites (TSS) of long-read lrCaptureSeq isoforms by splice-aligning short-read CAGE-seq reads from the same tissue and quantifying CAGE 5'-end coverage over each isoform's start site. Provides an external-evidence check that PacBio-defined isoforms are genuine rather than alignment artifacts.

## When to use

Use when a long-read-derived isoform catalog (e.g. lrCaptureSeq GFF) needs independent 5'-end validation and matched-tissue CAGE-seq data (e.g. adult mouse retina, DDBJ DRA002410) is available.

## When NOT to use

- No matched-tissue CAGE-seq data exists for the tissue from which the isoform catalog was derived — coverage evidence would not be an independent corroboration.
- The isoform catalog has not been filtered (no exon-count, truncation, or abundance filtering), so unvalidated low-quality/truncated transcripts would confound the validation fraction.

## Inputs

- CAGE sequencing FASTQ reads from adult mouse retina (DDBJ DRA002410, samples DRX019832-DRX019834)
- mm10 (GRCm38) mouse reference genome index for HISAT2
- Filtered lrCaptureSeq isoform catalog GFF (fullfilter.gff; 4+ exons, truncation-removed, 95% abundance cutoff) generated via IsoPops filter_top_gff

## Outputs

- cage_tss_coverage.csv with per-isoform 5'-end CAGE coverage values
- TSS-validation fraction (proportion of adult-retina isoforms with CAGE-supported start sites; 1051/1076 = 97.7%)
- Splice-aligned CAGE BAM file against mm10

## How to apply

Retrieve adult mouse retina CAGE reads from DDBJ accession DRA002410 (samples DRX019832-DRX019834) and splice-align the short reads to the mm10 (GRCm38) reference with HISAT2. Prepare the isoform catalog by compiling the IsoPops raw database from transcript, abundance, GFF, and ORF files, then filtering by exon count (4+ exons), removing truncations, and applying the 95% abundance cutoff to yield the fullfilter.gff used for testing. Use BedTools coverage/intersect to quantify CAGE 5'-end coverage over the start site of each adult-retina isoform in the filtered GFF. Compute the TSS-validation fraction as the proportion of cataloged isoforms whose start sites show CAGE 5'-end coverage (1051/1076 = 97.7% in this dataset), and write per-isoform coverage values plus the fraction to cage_tss_coverage.csv.

## Related tools

- **HISAT2** (Splice-aligns short CAGE-seq reads to the mm10 (GRCm38) mouse reference genome)
- **BedTools** (coverage/intersect quantifies CAGE 5'-end coverage over each isoform's transcription start site)
- **R** (Runs the IsoPops filtering steps that produce the fullfilter GFF used for TSS coverage testing and writes the coverage CSV) — github.com/kellycochran/IsoPops

## Examples

```
write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")
```

## Evaluation signals

- Per-isoform coverage table exists at cage_tss_coverage.csv and contains a coverage value for every isoform in fullfilter.gff.
- Validation fraction equals the ratio of isoforms with nonzero CAGE 5'-end coverage to total cataloged adult-retina isoforms (1051/1076 = 97.7% expected for this dataset).
- CAGE alignments were produced with splice-aware alignment (HISAT2) against mm10/GRCm38 — not a non-spliced aligner.
- Only 5'-end (CAGE tag) positions, not full-read coverage, are intersected with the TSS coordinates.
- The tested GFF reflects the full filter chain (4+ exons, truncations removed, 95% abundance cutoff), matching the IsoPops filter_top_gff output.

## Limitations

- Isoforms shorter than 4.5 kb may have evaded detection due to the library-preparation size-selection step, so the catalog being validated may itself be incomplete.
- Catalogs may lack certain short and/or rare transcripts; absence of an isoform cannot be excluded, and the validation fraction applies only to detected isoforms.
- CAGE coverage validates the 5' end only — full-length splice structure of the isoform is not independently corroborated by this step.

## Evidence

- [full_text] CAGE reads from adult mouse retina corroborated 97.7% of lrCaptureSeq transcription start sites, with 1051 of 1076 adult retina isoforms showing CAGE-seq coverage at their 5′ ends.: [evidence span withheld — anchor status: not_found]
- [full_text] Splice-align CAGE reads with HISAT2 to mm10: [evidence span withheld — anchor status: not_found]
- [full_text] Validation fraction computation: [evidence span withheld — anchor status: not_found]
- [methods] Filtered GFF generation in IsoPops: "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [discussion] Catalog completeness caveat: "Thus, while the catalogs may lack certain short and/or rare transcripts, we conclude that we have detected most of the isoforms expressed in our targeted tissues."
