---
name: bedtools-genomic-interval-intersection
description: Use when two methylation callers/runs (e.g., DeepMod2 output.per_site
  BED files from RRMS and whole-genome Nanopore sequencing) must be reduced to a common
  set of CpG sites before computing agreement statistics like Pearson r.
when_to_use_negative:
- The methylation data is still in modBAM form with MM/ML tags and has not yet been
  summarized to per-site bedMethyl tables — convert with modkit first.
- The two per-site files reference different genome builds or coordinate systems,
  where naive coordinate intersection would falsely match CpGs.
- The goal is region-level (e.g., differentially methylated region) overlap rather
  than single-CpG site concordance.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: bedtools
  role: Performs the genomic interval intersection that keeps only CpG sites present
    in both the RRMS and whole-genome per-site outputs
- name: modkit
  role: Converts DeepMod2 modBAM output (MM/ML tags) to bedMethyl per-site tables
    that are then intersected
  repo: https://github.com/nanoporetech/modkit
- name: DeepMod2
  role: Produces the reference-anchored per-site methylation frequency BED files for
    each sequencing run prior to intersection
  repo: https://github.com/WGLab/DeepMod2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:b7c11291c29651b9617550ee65cebb3d36212ba4fcf57c84eef2246aac0e28b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - DeepMod2
  - bedtools
  - minimap2
  - modkit
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bedtools-genomic-interval-intersection

## Summary

Restricts per-site DNA methylation comparisons to CpG loci reported in two separate Nanopore per-site BED/bedMethyl outputs (e.g., RRMS via adaptive sampling vs. whole-genome ONT sequencing of HG002) by genomic interval intersection, so correlation statistics are computed only on shared, adequately covered sites.

## When to use

Use when two methylation callers/runs (e.g., DeepMod2 output.per_site BED files from RRMS and whole-genome Nanopore sequencing) must be reduced to a common set of CpG sites before computing agreement statistics like Pearson r.

## When NOT to use

- The methylation data is still in modBAM form with MM/ML tags and has not yet been summarized to per-site bedMethyl tables — convert with modkit first.
- The two per-site files reference different genome builds or coordinate systems, where naive coordinate intersection would falsely match CpGs.
- The goal is region-level (e.g., differentially methylated region) overlap rather than single-CpG site concordance.

## Inputs

- DeepMod2 output.per_site / output.per_site.aggregated BED (bedMethyl) files for the RRMS run
- DeepMod2 output.per_site / output.per_site.aggregated BED (bedMethyl) files for the whole-genome run
- Optionally the GRCh38 reference FASTA used to anchor methylation calls

## Outputs

- BED/TSV of overlapping CpG sites shared between RRMS and whole-genome per-site outputs
- Paired per-site methylation percentages plus Pearson correlation value (methylation_correlation_scatter.tsv)

## How to apply

Run DeepMod2's detect module separately on each POD5 dataset (e.g., HG002 RRMS and ONT whole-genome, R9.4.1/R10.4.1) to produce output.per_site / output.per_site.aggregated BED files of per-site methylation frequencies. Use `bedtools intersect` with both per-site BED files to retain only CpG sites present in both runs, discarding sites covered below 10X Nanopore coverage (10X/5X for Ashkenazim trio/NIH3T3 ground truth). The intersection guarantees that the Pearson correlation between RRMS and whole-genome methylation percentages is computed on identically keyed genomic positions rather than on mismatched site sets. This mirrors the article's validation where intersected CpG sites yielded correlations of 95.78–96.45%, matching the reported r = 0.96. If the methylation calls are still in modBAM form, first convert them to bedMethyl tables with modkit before intersecting.

## Related tools

- **bedtools** (Performs the genomic interval intersection that keeps only CpG sites present in both the RRMS and whole-genome per-site outputs)
- **modkit** (Converts DeepMod2 modBAM output (MM/ML tags) to bedMethyl per-site tables that are then intersected) — https://github.com/nanoporetech/modkit
- **DeepMod2** (Produces the reference-anchored per-site methylation frequency BED files for each sequencing run prior to intersection) — https://github.com/WGLab/DeepMod2

## Examples

```
bedtools intersect -a rrms_output.per_site.bed -b wgs_output.per_site.bed -wa -wb > shared_cpg_sites.tsv
```

## Evaluation signals

- Every row in the intersection output is a CpG site present in both per-site BED files, keyed by identical chromosome/coordinate.
- Sites with Nanopore coverage below 10X are excluded before the correlation is computed.
- The Pearson r over the intersected CpG sites should closely match the reported correlation of r = 0.96 (95.78–96.45% in the article's validation).
- The paired per-site values and correlation value are written to methylation_correlation_scatter.tsv.

## Limitations

- Low-coverage filtering reduces the number of intersected sites and can lower statistical power: 'the resulting decrease in coverage for per-site predictions can lead to lower statistical power in downstream analyses, especially for low coverage samples'.
- Move-table-based signal alignment is approximate, so per-site calls feeding the BED files may be noisy: 'the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics'.
- Sites with intermediate methylation (20–80%) show poor concordance between platforms and were excluded from ground-truth evaluation, so correlations over all intersected sites may differ from benchmark-restricted correlations.

## Evidence

- [results] after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively): "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [results] In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [readme] Specifically for quickly converting modBAM to bedMethyl files using best practices, but also manipulating modBAM files and generating summary statistics.: [evidence span withheld — anchor status: external_doc]
