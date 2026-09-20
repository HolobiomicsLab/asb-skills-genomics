---
name: transcript-to-genomic-coordinate-lifting
description: Use when modification predictions are in transcript coordinates (e.g.,
  CHEUI output on a transcriptome reference) and genomic positions, gene annotation,
  or comparison to genome-based datasets (e.g., bsRNA-seq sites) are required.
when_to_use_negative:
- The modification calls are already in genomic coordinates or were produced by a
  genome-based mapper — R2_lift expects transcriptome-mapped features and would double-lift
  them
- No matching GTF annotation is available for the transcriptome reference used during
  alignment, since liftover requires the same transcript definitions used by minimap2
- The goal is single-read (per-molecule) resolution analysis rather than per-site
  coordinate conversion — R2Dtool handles isoform-resolved site-level features
edam_operation: http://edamontology.org/operation_3806
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0099
tools:
- name: R2Dtool
  role: Performs the conversion (cheui_to_bed.sh), isoform-specific annotation (R2_annotate
    with Gencode v38 GTF), and transcriptome-to-genome liftover (R2_lift) of the CHEUI
    methylation calls
  repo: https://github.com/comprna/R2Dtool
- name: CHEUI
  role: Upstream producer of transcript-coordinate m5C/m6A site calls that are lifted
    to genomic coordinates
  repo: https://github.com/comprna/CHEUI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:6af6c0c5e06d8f36e71590d29e40764c1dbdd3cddd814c135cc1ebc12ecb3237
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/R2Dtool
  tools:
  - CHEUI
  - R2Dtool
  edam_operation: http://edamontology.org/operation_3806
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Transcript-to-genomic coordinate lifting

## Summary

This skill converts transcriptome-mapped RNA modification calls (e.g., CHEUI m5C/m6A predictions in transcript coordinates) into genomic coordinates and annotates them isoform-specifically using R2Dtool, so differential sites can be reported and interpreted on the genome.

## When to use

Use when modification predictions are in transcript coordinates (e.g., CHEUI output on a transcriptome reference) and genomic positions, gene annotation, or comparison to genome-based datasets (e.g., bsRNA-seq sites) are required.

## When NOT to use

- The modification calls are already in genomic coordinates or were produced by a genome-based mapper — R2_lift expects transcriptome-mapped features and would double-lift them
- No matching GTF annotation is available for the transcriptome reference used during alignment, since liftover requires the same transcript definitions used by minimap2
- The goal is single-read (per-molecule) resolution analysis rather than per-site coordinate conversion — R2Dtool handles isoform-resolved site-level features

## Inputs

- CHEUI methylation call files in transcript coordinates (per-site predictions, e.g. CHEUI-diff output at Model 2 probability ≥0.9999 or CHEUI-diff BH-adjusted p<0.05 sites)
- Transcriptome annotation GTF matching the alignment reference (e.g. Gencode v38 for GRCh38)
- Reference genome/annotation version information (GRCh38 + Gencode v38)

## Outputs

- Bed-like file of methylation calls produced by cheui_to_bed.sh
- Annotated site table with isoform-specific metatranscript positions and distances to landmarks (R2_annotate output)
- Genomic-coordinate BED/TSV of lifted modification sites (R2_lift output), e.g. nsun2_m5c_analysis_results.tsv
- WebLogo sequence motif plots around lifted sites

## How to apply

First convert CHEUI methylation calls to a bed-like format with the R2Dtool script cheui_to_bed.sh using default parameters. Then annotate the transcript-mapped sites with R2_annotate (default parameters) using the relevant GTF annotation, e.g. Gencode v38 for the HeLa data, which adds isoform-specific metatranscript and landmark-relative positions. Finally run R2_lift with default parameters to transpose the annotated transcriptome-mapped calls to genomic coordinates (GRCh38 for the human HeLa data). Validate the lifted differential sites, e.g. the 186 NSUN2-dependent m5C sites, by computing sequence logos around the lifted positions with WebLogo and confirming enrichment of the 5'-m5CNGGG-3' NSUN2 motif.

## Related tools

- **R2Dtool** (Performs the conversion (cheui_to_bed.sh), isoform-specific annotation (R2_annotate with Gencode v38 GTF), and transcriptome-to-genome liftover (R2_lift) of the CHEUI methylation calls) — https://github.com/comprna/R2Dtool
- **CHEUI** (Upstream producer of transcript-coordinate m5C/m6A site calls that are lifted to genomic coordinates) — https://github.com/comprna/CHEUI

## Examples

```
R2_lift --input cheui_m5c_annotated.bed --output nsun2_m5c_genomic.bed
```

## Evaluation signals

- cheui_to_bed.sh, R2_annotate, and R2_lift are all run with default parameters as specified in the methods
- The GTF supplied to R2_annotate matches the reference used for alignment (Gencode v38 / GRCh38 for HeLa)
- Lifted genomic coordinates correspond to transcript positions on primary, positive-strand alignments (samtools -F 232457 filtered reads)
- For the HeLa NSUN2 analysis, the lifted output recovers 186 NSUN2-dependent genomic m5C sites, 18 with prior bsRNA-seq support
- WebLogo around the lifted sites shows enrichment of the 5'-m5CNGGG-3' NSUN2 motif

## Limitations

- Liftover is isoform-dependent: transcript positions map to different genomic loci depending on the annotated isoform, so annotation quality depends on the GTF
- Only sites detected with strict cut-offs (e.g. CHEUI Model 2 probability ≥0.9999 or CHEUI-diff FDR 0.05 with Benjamini-Hochberg correction) are carried forward; many orthogonally supported sites fall below these thresholds
- Transcript-only detections still require independent validation once mapped to the genome

## Evidence

- [methods] the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format: "the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format"
- [methods] R2_lift transposes annotated methylation calls to genomic coordinates: "the R2Dtool R2_lift command was run with default parameters to transpose the annotated methylation calls"
- [readme] R2Dtool performs liftover of transcriptome-mapped RNA features to their corresponding genomic coordinates: [evidence span withheld — anchor status: external_doc]
- [results] CHEUI-diff identified 186 potential NSUN2-dependent unique genomic m5C sites: [evidence span withheld — anchor status: not_found]
- [methods] R2_annotate used with default parameters and the relevant GTF annotation: "the R2Dtool R2_annotate command was used with default parameters and the relevant GTF annotation"
