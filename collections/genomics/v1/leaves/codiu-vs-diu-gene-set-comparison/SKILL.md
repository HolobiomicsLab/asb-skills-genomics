---
name: codiu-vs-diu-gene-set-comparison
description: Use when two gene sets (coDIU and DIU) derived from isoform usage analysis
  must be functionally contrasted to ask which positional features or GO terms are
  annotated in a significantly higher proportion in one set.
when_to_use_negative:
- The input transcriptome is de novo and lacks a compatible pre-annotated tappAS reference,
  since IsoAnnotLite can only transfer functional labels from a compatible annotated
  reference until a de novo annotation tool is available
- The gene sets to compare are independent, in which case the partially-overlapping-samples
  z-test used for the non-independent coDIU/DIU sets is not the correct enrichment
  statistic
- The goal is isoform co-expression network construction rather than functional enrichment
  of already-defined DIU/coDIU gene sets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tappAS
  role: Runs the coDIU-vs-DIU functional enrichment analysis with the partially-overlapping-samples
    z-test over positional features and GO terms
  repo: https://github.com/ConesaLab/tappAS
- name: acorde
  role: R package that produces the DIU/coDIU gene sets and the IsoAnnotLite-annotated
    long read transcriptome feeding the enrichment analysis
  repo: https://github.com/ConesaLab/acorde
- name: Revigo
  role: Redundancy reduction of the enriched GO term list (59 terms reduced to 26
    unique terms)
- name: IsoAnnotLite
  role: Transfers functional labels from the tappAS mouse RefSeq 78 pre-annotated
    reference onto the long read-defined isoforms used for enrichment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:abb231db5aec64aeaa221c8a48fec90b875720184f4f72fb88a04c789217e05a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - IsoAnnotLite
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# codiu-vs-diu-gene-set-comparison

## Summary

This skill runs a tappAS functional enrichment analysis contrasting coDIU genes against DIU genes with the partially-overlapping-samples z-test, testing whether positional functional features and Gene Ontology terms are over-represented in coDIU genes. It recovers the paper's key result that 91 positional functional features and 59 GO terms are significantly enriched in coDIU vs DIU genes (FDR < 0.05, Benjamini-Hochberg corrected).

## When to use

Use when two gene sets (coDIU and DIU) derived from isoform usage analysis must be functionally contrasted to ask which positional features or GO terms are annotated in a significantly higher proportion in one set.

## When NOT to use

- The input transcriptome is de novo and lacks a compatible pre-annotated tappAS reference, since IsoAnnotLite can only transfer functional labels from a compatible annotated reference until a de novo annotation tool is available
- The gene sets to compare are independent, in which case the partially-overlapping-samples z-test used for the non-independent coDIU/DIU sets is not the correct enrichment statistic
- The goal is isoform co-expression network construction rather than functional enrichment of already-defined DIU/coDIU gene sets

## Inputs

- Long read-defined, IsoAnnotLite-annotated Mus musculus transcriptome annotation from the acorde pipeline (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3)
- tappAS pre-annotated Mus musculus RefSeq (RefSeq 78) functional annotation
- Gene-to-condition table assigning genes to the coDIU and DIU sets
- DIU/coDIU gene lists from the acorde Differential Isoform Usage analysis

## Outputs

- codiu_functional_enrichment.tsv with positional feature and GO term enrichment results and FDR values
- List of 91 positional functional features and 59 GO terms enriched in coDIU vs DIU genes at FDR < 0.05
- 26 unique GO terms after Revigo redundancy filtering

## How to apply

Load the long read-defined, IsoAnnotLite (v2.6)-annotated Mus musculus transcriptome produced by the acorde pipeline, together with the tappAS mouse RefSeq functional annotation and a gene-to-condition table assigning genes to the coDIU and DIU sets. In tappAS, run the functional enrichment analysis of coDIU vs DIU genes using the partially-overlapping samples z-test, the method appropriate because the two gene sets are not independent (coDIU genes are a subset/overlap of the DIU universe). Correct p-values across all tested features with the Benjamini-Hochberg procedure and retain features at FDR < 0.05, which in the paper recovers 91 positional functional features and 59 GO terms. Submit the enriched GO term list to Revigo to remove redundant terms, leaving 26 unique GO terms, and write feature and GO results with FDR values to codiu_functional_enrichment.tsv.

## Related tools

- **tappAS** (Runs the coDIU-vs-DIU functional enrichment analysis with the partially-overlapping-samples z-test over positional features and GO terms) — https://github.com/ConesaLab/tappAS
- **acorde** (R package that produces the DIU/coDIU gene sets and the IsoAnnotLite-annotated long read transcriptome feeding the enrichment analysis) — https://github.com/ConesaLab/acorde
- **Revigo** (Redundancy reduction of the enriched GO term list (59 terms reduced to 26 unique terms))
- **IsoAnnotLite** (Transfers functional labels from the tappAS mouse RefSeq 78 pre-annotated reference onto the long read-defined isoforms used for enrichment)

## Examples

```
Rscript -e "devtools::install_github('ConesaLab/acorde', build_vignettes = TRUE)"
```

## Evaluation signals

- The output reproduces 91 positional functional features enriched in coDIU vs DIU genes at FDR < 0.05 (Benjamini-Hochberg corrected)
- The output reproduces 59 GO terms enriched in coDIU vs DIU genes at FDR < 0.05
- Revigo filtering of the 59 enriched GO terms leaves exactly 26 unique GO terms
- The enrichment test statistic used is the partially-overlapping samples z-test, not a standard hypergeometric or rank-based enrichment test
- codiu_functional_enrichment.tsv contains FDR values for both positional features and GO terms

## Limitations

- The acorde R package does not include the functional analyses; enrichment must be run through external tools (tappAS GUI, Revigo)
- Functional annotation of long read-defined isoforms depends on a compatible pre-annotated reference (mouse RefSeq via IsoAnnotLite v2.6); de novo annotation was not available at the time
- The reported functional insights are hypothesis-generating and require further experimental validation

## Evidence

- [results] Using the partially-overlapping samples z test, 91 positional functional features and 59 GO terms were found annotated in a significantly higher proportion in coDIU vs DIU genes (FDR < 0.05).: [evidence span withheld — anchor status: not_found]
- [readme] we used IsoAnnotLite (v2.6) to transfer functional labels from tappAS’ pre-annotated mouse RefSeq 78 reference: "we used IsoAnnotLite (v2.6) to transfer functional labels 
from tappAS’ pre-annotated mouse RefSeq 78 reference"
- [intro] we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information: "we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information"
- [abstract] By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [readme] Regarding functional analyses, we have included a section in the vignette providing instructions to perform them: "Regarding **functional analyses**, we have included a section in the vignette 
providing instructions to perform them"
