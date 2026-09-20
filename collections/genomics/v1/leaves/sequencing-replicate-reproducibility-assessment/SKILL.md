---
name: sequencing-replicate-reproducibility-assessment
description: Use when the same single-cell cDNA libraries (e.g., 10x Genomics corneal
  limbus samples) have been measured on two sequencing platforms or as independent
  replicates, and you need numeric concordance statistics to validate reproducibility.
when_to_use_negative:
- The datasets being compared are from different biological samples or different single-cell
  capture runs, so barcode-level or gene-level correlations would conflate biological
  with technical variation
- You need isoform-level concordance rather than gene/UMI count-level agreement; this
  skill operates on cellBC and gene UMI totals only
- Count matrices have not been demultiplexed and corrected (e.g., with cellBC_UMI_corrector
  using the whitelist posterior probability threshold >0.975), so per-cell counts
  are unreliable
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scISA-Tools
  role: Generates the single-cell gene expression matrices (scGene_matrix) and corrected
    cellBC/UMI counts that are the inputs to the correlation analysis
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: R
  role: Computes Pearson correlations between summed UMI counts (by cellBC, by gene)
    and between replicate matrices, using Matrix/Seurat
- name: PacBio Sequel II
  role: Platform generating the HIT-scISOseq long-read UMI counts compared against
    Illumina NGS short-read counts
- name: Illumina NextSeq 550
  role: Short-read platform whose gene-level UMI counts serve as the NGS reference
    in the concordance comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:8920a14b97c4b76d80e8309d97f8a354fa7bf3471d5ccaed30a6d62ff6dfa9f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - Illumina NextSeq 550
  - PacBio Sequel II
  - R
  - Seurat
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sequencing-replicate-reproducibility-assessment

## Summary

Quantify concordance between HIT-scISOseq (PacBio Sequel II) and Illumina short-read NGS measurements, and between biological replicates, by computing Pearson correlations of UMI counts per cell barcode, per gene, and across replicate count matrices.

## When to use

Use when the same single-cell cDNA libraries (e.g., 10x Genomics corneal limbus samples) have been measured on two sequencing platforms or as independent replicates, and you need numeric concordance statistics to validate reproducibility.

## When NOT to use

- The datasets being compared are from different biological samples or different single-cell capture runs, so barcode-level or gene-level correlations would conflate biological with technical variation
- You need isoform-level concordance rather than gene/UMI count-level agreement; this skill operates on cellBC and gene UMI totals only
- Count matrices have not been demultiplexed and corrected (e.g., with cellBC_UMI_corrector using the whitelist posterior probability threshold >0.975), so per-cell counts are unreliable

## Inputs

- Cell-by-barcode UMI count matrix from HIT-scISOseq (Source Data for Fig. 3)
- Cell-by-gene UMI count matrices from HIT-scISOseq and Illumina NGS short-read data on the same 10x Genomics limbal epithelium cDNA samples
- UMI count matrices from two independent HIT-scISOseq biological replicates
- Single-cell gene/UMI count matrices produced by the scGene_matrix utility of scISA-Tools

## Outputs

- fig3_pearson_correlations.csv containing three Pearson r values with comparison labels (cellBC r=0.992, gene r=0.956, replicate r=0.998)
- Recomputed Pearson correlation coefficients for platform concordance and replicate concordance

## How to apply

Load the deposited single-cell gene/UMI count matrices from the Source Data for Fig. 3 (cell-by-BC, cell-by-gene, and replicate matrices) into R (3.4.1 or later, with Seurat/Matrix available per the scISA-Tools dependencies). Sum UMI counts per cell barcode and per gene, then compute the Pearson correlation between HIT-scISOseq and Illumina NGS UMI totals by cellBC (reported r=0.992) and by gene (reported r=0.956). Separately compute the Pearson correlation between the UMI count matrices of the two HIT-scISOseq biological replicates (reported r=0.998). Write all three correlation coefficients, with their comparison labels, to fig3_pearson_correlations.csv, validating each recomputed value against the reported values within a tolerance of ±0.01.

## Related tools

- **scISA-Tools** (Generates the single-cell gene expression matrices (scGene_matrix) and corrected cellBC/UMI counts that are the inputs to the correlation analysis) — https://github.com/shizhuoxing/scISA-Tools
- **R** (Computes Pearson correlations between summed UMI counts (by cellBC, by gene) and between replicate matrices, using Matrix/Seurat)
- **PacBio Sequel II** (Platform generating the HIT-scISOseq long-read UMI counts compared against Illumina NGS short-read counts)
- **Illumina NextSeq 550** (Short-read platform whose gene-level UMI counts serve as the NGS reference in the concordance comparison)

## Examples

```
Rscript -e 'cor.test(hit_umi_by_cellBC, ngs_umi_by_cellBC)$estimate' # expect r ≈ 0.992; write c(cellBC=0.992, gene=0.956, replicate=0.998) to fig3_pearson_correlations.csv
```

## Evaluation signals

- Recomputed Pearson r for UMI counts by cellBC (HIT-scISOseq vs Illumina NGS) falls within ±0.01 of 0.992
- Recomputed Pearson r for UMI counts by gene (HIT-scISOseq vs Illumina NGS) falls within ±0.01 of 0.956
- Recomputed Pearson r between the two HIT-scISOseq biological replicates falls within ±0.01 of 0.998
- fig3_pearson_correlations.csv contains exactly three labeled correlation coefficients matching the reported values
- Row/column identifiers (cell barcodes, genes) align across the two platform matrices before correlation is computed

## Limitations

- Only two biological replicates (one sample each from two animals) were used, with no randomization or blinding, limiting generalizability of the reproducibility estimate
- Concordance was demonstrated only on 10x Genomics corneal limbus cDNA; compatibility with other single-cell platforms is claimed but not demonstrated
- Gene-level correlation (r=0.956) is lower than cellBC-level correlation (r=0.992), so gene-level agreement should not be assumed to reach barcode-level strength

## Evidence

- [results] HIT-scISOseq and Illumina NGS show strong agreement on the same limbal epithelium cDNA samples, with Pearson r = 0.992 for UMI counts by cellBC and r = 0.956 for UMI counts by gene.: [evidence span withheld — anchor status: not_found]
- [methods] Load the deposited single-cell gene/UMI count matrices from the Source Data for Fig. 3: [evidence span withheld — anchor status: not_found]
- [methods] we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
- [readme] R-3.4.1 or later with `ggplot2 | gridExtra | grid | Seurat | DropletUtils`: "R-3.4.1 or later with `ggplot2 | gridExtra | grid | Seurat | DropletUtils`"
- [methods] we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
