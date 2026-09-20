---
name: genomic-interval-intersection
description: Use when you need to corroborate 5′ ends (TSSs) of long-read-derived
  isoforms against independent 5′-end-enriched short-read data such as CAGE-seq, computing
  a per-isoform coverage-supported validation fraction.
when_to_use_negative:
- When the goal is full-length isoform discovery rather than independent validation
  of already-cataloged 5′ ends — short CAGE reads cannot resolve multi-kilobase transcript
  structure.
- When the isoform catalog has not been filtered (e.g., by exon count, truncations,
  abundance cutoff), since unfiltered TSS lists include low-confidence artifacts.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: HISAT2
  role: Splice-aligns short CAGE-seq reads to the mm10 mouse reference genome
- name: BedTools
  role: coverage/intersect to quantify CAGE 5'-end coverage over isoform TSSs from
    the filtered GFF
- name: R
  role: Computes the per-isoform validation statistics and writes cage_tss_coverage.csv
  repo: github.com/kellycochran/IsoPops
- name: IsoPops
  role: Produces the filtered catalog GFF (fullfilter.gff) whose TSSs are validated;
    filter_db, filter_truncations, filter_top_gff
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:6f4bd4fa16b8f7d752a0087e532f2d9e09a68c392be12b7b2f85ea6dcc1f847b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BedTools
  - HISAT2
  - IsoPops
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genomic-interval-intersection

## Summary

Independently validates transcription start sites of lrCaptureSeq isoforms by intersecting CAGE-seq 5'-end coverage against cataloged isoform start coordinates in a filtered GFF, quantifying the fraction of TSSs corroborated by an orthogonal short-read dataset.

## When to use

Use when you need to corroborate 5′ ends (TSSs) of long-read-derived isoforms against independent 5′-end-enriched short-read data such as CAGE-seq, computing a per-isoform coverage-supported validation fraction.

## When NOT to use

- When the goal is full-length isoform discovery rather than independent validation of already-cataloged 5′ ends — short CAGE reads cannot resolve multi-kilobase transcript structure.
- When the isoform catalog has not been filtered (e.g., by exon count, truncations, abundance cutoff), since unfiltered TSS lists include low-confidence artifacts.

## Inputs

- CAGE-seq short reads from adult mouse retina (DDBJ DRA002410, samples DRX019832-DRX019834)
- mm10 (GRCm38) mouse reference genome
- Filtered lrCaptureSeq isoform catalog in GFF format (fullfilter.gff; 4+ exons, truncation-removed, 95% abundance cutoff)

## Outputs

- Per-isoform CAGE 5'-end coverage values and TSS-validation fraction written to cage_tss_coverage.csv
- TSS-validation fraction (e.g., 1051/1076 = 97.7% of adult retina isoforms corroborated)

## How to apply

Retrieve adult mouse retina CAGE sequencing reads from DDBJ accession DRA002410 (samples DRX019832-DRX019834) and splice-align the short CAGE reads to the mm10 (GRCm38) mouse reference genome using HISAT2. Extract the transcription start sites of each adult-retina isoform from the filtered catalog GFF (fullfilter.gff), which was produced by the IsoPops pipeline with 4+ exons, truncation removal, and a 95% abundance cutoff. Use BedTools coverage/intersect to quantify CAGE 5'-end coverage over each isoform's start site. Compute the TSS-validation fraction as the proportion of cataloged adult-retina isoforms whose start sites show CAGE 5'-end coverage (in this study, 1051/1076 = 97.7%). Write per-isoform 5' coverage values and the validation fraction to a CSV (cage_tss_coverage.csv); the skill succeeds when the vast majority of long-read TSSs are independently supported.

## Related tools

- **HISAT2** (Splice-aligns short CAGE-seq reads to the mm10 mouse reference genome)
- **BedTools** (coverage/intersect to quantify CAGE 5'-end coverage over isoform TSSs from the filtered GFF)
- **R** (Computes the per-isoform validation statistics and writes cage_tss_coverage.csv) — github.com/kellycochran/IsoPops
- **IsoPops** (Produces the filtered catalog GFF (fullfilter.gff) whose TSSs are validated; filter_db, filter_truncations, filter_top_gff) — github.com/kellycochran/IsoPops

## Examples

```
bedtools intersect -a fullfilter.gff -b cage_5ends.bed -c > cage_tss_coverage.bed; Rscript compute_validation_fraction.R --input cage_tss_coverage.bed --output cage_tss_coverage.csv
```

## Evaluation signals

- The computed validation fraction matches expectations for a high-confidence catalog (≈97.7%, 1051/1076 adult retina isoforms with CAGE coverage at their 5′ ends).
- cage_tss_coverage.csv contains one row per cataloged adult-retina isoform with a numeric 5′ coverage value.
- CAGE alignments map correctly to mm10 with splice-aware settings (HISAT2) and produce sensible coverage tracks over known retinal gene loci.
- The GFF used for TSS extraction is the filtered catalog (fullfilter.gff) — exon count ≥ 4, truncations removed, 95% abundance cutoff applied.

## Limitations

- CAGE-seq coverage at a TSS validates the 5′ end only, not the full-length exon structure of the isoform.
- Catalogs may lack certain short and/or rare transcripts, and some isoforms smaller than 4.5 kb may have evaded detection due to size selection in library preparation, biasing the denominator of the validation fraction.

## Evidence

- [other] Splice-align the short CAGE reads to the mm10 (GRCm38) mouse reference genome using HISAT2.: [evidence span withheld — anchor status: not_found]
- [other] Use BedTools coverage/intersect to quantify CAGE 5'-end coverage over the transcription start sites: [evidence span withheld — anchor status: not_found]
- [discussion] some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
