---
name: pearson-correlation-of-per-site-methylation-levels
description: Use when comparing methylation levels between a targeted/enrichment Nanopore
  run (e.g., RRMS adaptive sampling) and a whole-genome Nanopore run to quantify concordance
  of per-site CpG methylation frequencies.
when_to_use_negative:
- The goal is accuracy benchmarking of a methylation caller against short-read bisulfite
  or microarray ground truth rather than comparing two Nanopore assays on the same
  sample
- The two datasets come from different cell lines or samples — per-site correlation
  is only meaningful for the same biological sample (e.g., HG002)
- Only one sequencing run exists and there is no second assay to correlate against
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepMod2
  role: Reference-anchored 5mC methylation calling per read and per site from POD5
    signal files
  repo: github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalling of POD5 with --emit-moves and GRCh38 reference to produce aligned
    BAMs with move tables
  repo: https://github.com/nanoporetech/dorado
- name: bedtools
  role: Intersect RRMS and whole-genome per-site outputs to retain overlapping CpG
    sites
- name: modkit
  role: Conversion of modBAM to bedMethyl per-site methylation tables
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Align basecalled reads to reference genome while retaining move tables
- name: samtools
  role: BAM manipulation during realignment of basecalled reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:b7e0b67424822a3be1af9bebaab2e08ff08a5c7578b4b85e3e0ecdb0f48aa937
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - Dorado
  - bedtools
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pearson-correlation-of-per-site-methylation-levels

## Summary

Reproduce the reported r = 0.96 Pearson correlation between per-site CpG methylation frequencies from reduced representation methylation sequencing (RRMS via adaptive sampling) and whole-genome Nanopore sequencing of HG002, validating RRMS as a cost-effective methylation profiling strategy.

## When to use

Use when comparing methylation levels between a targeted/enrichment Nanopore run (e.g., RRMS adaptive sampling) and a whole-genome Nanopore run to quantify concordance of per-site CpG methylation frequencies.

## When NOT to use

- The goal is accuracy benchmarking of a methylation caller against short-read bisulfite or microarray ground truth rather than comparing two Nanopore assays on the same sample
- The two datasets come from different cell lines or samples — per-site correlation is only meaningful for the same biological sample (e.g., HG002)
- Only one sequencing run exists and there is no second assay to correlate against

## Inputs

- POD5 signal directories for HG002 ONT RRMS (adaptive sampling) runs
- POD5 signal directories for HG002 ONT whole-genome sequencing runs (R9.4.1 and R10.4.1)
- GRCh38 reference FASTA (GRCh38.fa)
- DeepMod2 per-site methylation BED files (output.per_site / output.per_site.aggregated) from each run

## Outputs

- methylation_correlation_scatter.tsv containing paired RRMS vs whole-genome per-site methylation percentages and the computed Pearson r
- Bedtools-intersected set of overlapping CpG sites passing the 10X Nanopore coverage filter

## How to apply

Basecall each POD5 dataset (RRMS and whole-genome) with Dorado using --emit-moves and the GRCh38 reference (model dna_r10.4.1_e8.2_400bps_hac@v4.3.0) to obtain aligned BAMs with move tables. Run DeepMod2's detect module (e.g., --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5) separately on each dataset with the reference FASTA for reference-anchored per-site methylation calling. Collect per-site methylation frequencies from each run's output.per_site / output.per_site.aggregated BED files, then use bedtools intersect to keep only CpG sites present in both outputs, dropping sites with Nanopore coverage below 10X. Compute the Pearson correlation coefficient of the paired per-site methylation percentages over the overlapping CpG sites and write the paired values plus r to methylation_correlation_scatter.tsv.

## Related tools

- **DeepMod2** (Reference-anchored 5mC methylation calling per read and per site from POD5 signal files) — github.com/WGLab/DeepMod2
- **Dorado** (Basecalling of POD5 with --emit-moves and GRCh38 reference to produce aligned BAMs with move tables) — https://github.com/nanoporetech/dorado
- **bedtools** (Intersect RRMS and whole-genome per-site outputs to retain overlapping CpG sites)
- **modkit** (Conversion of modBAM to bedMethyl per-site methylation tables) — https://github.com/nanoporetech/modkit
- **minimap2** (Align basecalled reads to reference genome while retaining move tables)
- **samtools** (BAM manipulation during realignment of basecalled reads)

## Examples

```
python deepmod2 detect --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --bam aligned.bam --input signal_dir --ref GRCh38.fa --threads 8
```

## Evaluation signals

- Computed Pearson r over overlapping CpG sites should closely match the reported r = 0.96 (article reports 95.78–96.45% correlation)
- No CpG site retained in the comparison has Nanopore coverage below 10X in either dataset
- methylation_correlation_scatter.tsv contains both the paired per-site methylation values and the correlation value
- Per-site methylation frequencies derive from DeepMod2 reference-anchored calls (BAM aligned to GRCh38 with move tables retained)
- Overlapping CpG set is the exact intersection of the two per-site BED files (bedtools intersect), not the union

## Limitations

- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Move tables provide only approximate signal-to-basecall alignment, which can produce uninformative signal summary statistics at some sites
- Correlation is restricted to CpG sites covered ≥10X in both RRMS and whole-genome runs; low-coverage sites are excluded, so correlation may not reflect performance genome-wide
- DeepMod2 currently only provides models for 5mCpG methylation; other modifications (e.g., 5hmC) are not detected

## Evidence

- [abstract] we show a high correlation (r = 0.96) between reduced representation and whole-genome Nanopore sequencing: "we show a high correlation (r = 0.96) between reduced representation and whole-genome Nanopore sequencing"
- [intro] high correlation of RRMS vs WGS HG002: "high correlation (r = 0.96) between reduced representation and whole-genome Nanopore sequencing of HG002, suggesting that it can be a viable strategy for large-scale cost-effective methylation"
- [results] per-site aggregation description: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [results] correlation analysis coverage filter: "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [readme] Dorado basecalling with move tables: [evidence span withheld — anchor status: external_doc]
