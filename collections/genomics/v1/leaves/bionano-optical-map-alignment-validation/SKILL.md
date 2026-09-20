---
name: bionano-optical-map-alignment-validation
description: Use when a draft assembly has been split at CRAQ CSE breakpoints (e.g.,
  A. oxysepala ctg8) and an independent structural check of each split fragment is
  needed before Hi-C rescaffolding; requires Bionano optical maps.
when_to_use_negative:
- No Bionano optical map data exist for the organism — CRAQ CSE detection still works,
  but optical-map validation cannot be performed
- Validation of nucleotide-level assembly accuracy (single-nucleotide CREs) is the
  goal — optical maps only infer approximate misjoin locations, not base-level errors
- The draft is already scaffolded at chromosome scale — reference-based assessment
  with R-AQI rather than split validation is more appropriate
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: RefAligner (Bionano Solve)
  role: Aligns CRAQ-corrected contigs back to the Bionano DLE-1 optical consensus
    maps (default parameters) to confirm that split fragments map to distinct CMAPs
- name: CRAQ
  role: Produces the corrected split contigs (out_correct.fa) and CSE breakpoint coordinates
    that the optical-map alignment validates
  repo: github.com/JiaoLaboratory/CRAQ
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:1df6739783974b5f9ff58d77fef6bc4ba28e680d56549e1d312bd0ed75695729
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - RefAligner (Bionano Solve)
  - SAMtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bionano-optical-map-alignment-validation

## Summary

Validates CRAQ-detected CSE misjoin splits by aligning the corrected contigs back to Bionano DLE-1 optical consensus maps with RefAligner (Bionano Solve), confirming that split fragments map to independent CMAPs and that optical-map conflicts drop after correction.

## When to use

Use when a draft assembly has been split at CRAQ CSE breakpoints (e.g., A. oxysepala ctg8) and an independent structural check of each split fragment is needed before Hi-C rescaffolding; requires Bionano optical maps.

## When NOT to use

- No Bionano optical map data exist for the organism — CRAQ CSE detection still works, but optical-map validation cannot be performed
- Validation of nucleotide-level assembly accuracy (single-nucleotide CREs) is the goal — optical maps only infer approximate misjoin locations, not base-level errors
- The draft is already scaffolded at chromosome scale — reference-based assessment with R-AQI rather than split validation is more appropriate

## Inputs

- CRAQ-corrected split contigs (e.g., A.oxysepala_contig_CRAQcorrect.fa, FASTA) with out_correct.fa and strER_out/out_final.CSE.bed breakpoints
- Original Falcon draft contigs (A.oxysepala_contig_draft.fa, FASTA) for before-correction comparison
- Bionano Genomics Direct Label and Stain (DLS) optical consensus maps (CMAP files, e.g., CMAP-1, CMAP-10)

## Outputs

- RefAligner/Bionano Solve alignment records mapping split fragments to CMAPs (ctg8_1 -> CMAP-1, ctg8_2 -> CMAP-10)
- Pre- and post-correction optical-map conflict counts
- Confirmation record row in craq_correction_metrics.tsv accompanying the N50/R-AQI/S-AQI comparison table

## How to apply

Run CRAQ with --break|-b T on the Falcon draft contigs using SMS and NGS BAMs (clipping ratio cutoff f = 0.75, heterozygous band h = 0.4-0.6, minimum clipped-read count 2) to split contigs at CSE breakpoints, writing out_correct.fa and strER_out/out_final.CSE.bed. Align both original and CRAQ-corrected contigs to the Bionano DLE-1 optical consensus maps using RefAligner within the Bionano Solve pipeline under default parameters. For the A. oxysepala misjoin, verify that the split fragment ctg8_1 maps to CMAP-1 and ctg8_2 maps to CMAP-10, demonstrating that one draft contig spanned two distinct chromosomes. Count optical-map conflicts before and after correction so the confirmation record can be tabulated (craq_correction_metrics.tsv) alongside the S-AQI/N50/R-AQI gains (S-AQI 58.1 vs 39.0; N50 28 Mb vs 20 Mb; R-AQI 78.7 vs 79.1). Note that optical maps localize misjoins approximately, so they serve as structural confirmation of splits, not nucleotide-level error calls.

## Related tools

- **RefAligner (Bionano Solve)** (Aligns CRAQ-corrected contigs back to the Bionano DLE-1 optical consensus maps (default parameters) to confirm that split fragments map to distinct CMAPs)
- **CRAQ** (Produces the corrected split contigs (out_correct.fa) and CSE breakpoint coordinates that the optical-map alignment validates) — github.com/JiaoLaboratory/CRAQ

## Evaluation signals

- ctg8_1 maps to CMAP-1 and ctg8_2 maps to CMAP-10 in the RefAligner alignment, confirming the single misjoin split
- Number of optical-map conflicts is reduced after correction relative to the original draft contigs
- Detected CSE count is in the vicinity of the reported 117, including the breakpoint at contig8:1,874,290
- Corrected scaffolds reproduce S-AQI ~58.1 and N50 ~28 Mb vs original S-AQI 39.0 and N50 20 Mb, with R-AQI nearly unchanged (78.7 vs 79.1)

## Limitations

- The optical mapping-based approach could only infer the approximate genomic locations of misjoins, so split validation is structural rather than nucleotide-resolved
- Optical-map confirmation depends on DLE-1 label density and map quality; regions without adequate optical coverage cannot be validated
- CRAQ classification of ambiguous regions (coverage between h and f) and polyploid genomes may be less reliable, affecting which splits merit optical validation

## Evidence

- [discussion] Bionano Genomics Direct Label and Stain (DLS) optical consensus maps of A. oxysepala were used to identify potential chimeric errors: "Bionano Genomics Direct Label and Stain (DLS) optical consensus maps of A. oxysepala were used to identify potential chimeric errors"
- [discussion] The optical mapping-based approach could only infer the approximate genomic locations of misjoins.: "The optical mapping-based approach could only infer the approximate genomic locations of misjoins."
- [readme] A.oxysepala_contig_CRAQcorrect.fa: The new generated A.oxysepala contigs after splitting misjoins at CSE positions.: [evidence span withheld — anchor status: external_doc]
- [abstract] CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction: "CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction"
