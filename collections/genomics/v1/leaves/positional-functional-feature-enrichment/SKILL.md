---
name: positional-functional-feature-enrichment
description: Use when a coDIU gene set from acorde must be tested for enrichment of
  transcript-level functional features against a DIU gene background, prior to biological
  interpretation of isoform co-usage networks.
when_to_use_negative:
- Your gene sets are disjoint or you lack the gene-to-condition table needed for the
  partially-overlapping samples z-test
- You want enrichment of whole-gene reference annotation rather than functional elements
  transferred onto long read-defined isoforms via IsoAnnotLite
- The transcriptome is de novo (not from a compatible pre-annotated reference), so
  positional functional labels cannot be transferred
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: tappAS
  role: Runs the functional enrichment analysis (partially-overlapping samples z-test)
    comparing coDIU vs DIU genes over positional features and GO terms
  repo: https://github.com/ConesaLab/tappAS
- name: acorde
  role: Produces the DIU and coDIU gene sets and the IsoAnnotLite-annotated transcriptome
    that serve as enrichment inputs
  repo: https://github.com/ConesaLab/acorde
- name: Revigo
  role: Removes redundant GO terms from the enriched GO term list, leaving the non-redundant
    term set
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:61e4c146f56ad440d1d08ed701f9de0c2ee398eec07166df881bf6db66031407
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# positional-functional-feature-enrichment

## Summary

Detect positional functional features (domains, motifs, sites) and GO terms annotated in a significantly higher proportion in coDIU versus DIU genes, using tappAS' partially-overlapping samples z-test with Benjamini-Hochberg FDR correction on a long-read-defined, IsoAnnotLite-annotated mouse transcriptome.

## When to use

Use when a coDIU gene set from acorde must be tested for enrichment of transcript-level functional features against a DIU gene background, prior to biological interpretation of isoform co-usage networks.

## When NOT to use

- Your gene sets are disjoint or you lack the gene-to-condition table needed for the partially-overlapping samples z-test
- You want enrichment of whole-gene reference annotation rather than functional elements transferred onto long read-defined isoforms via IsoAnnotLite
- The transcriptome is de novo (not from a compatible pre-annotated reference), so positional functional labels cannot be transferred

## Inputs

- Long read-defined, IsoAnnotLite-annotated transcriptome annotation (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3) from the acorde pipeline
- tappAS Mus musculus RefSeq functional annotation file (positional features: motifs, domains, sites, GO terms)
- Gene-to-condition table assigning genes to the coDIU and DIU sets from acorde's DIU/coDIU analysis

## Outputs

- Table of positional functional features and GO terms enriched in coDIU vs DIU genes with p-values and BH-corrected FDR values (FDR < 0.05), e.g. codiu_functional_enrichment.tsv
- Redundancy-filtered list of unique enriched GO terms (26 unique GO terms after Revigo)

## How to apply

Load the long read-defined, IsoAnnotLite (v2.6)-annotated Mus musculus transcriptome (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3) produced by the acorde pipeline, together with the tappAS mouse RefSeq functional annotation and the gene-to-condition table assigning genes to coDIU and DIU sets. In tappAS, run the functional enrichment analysis comparing coDIU genes against DIU genes with the partially-overlapping samples z-test, which is appropriate because the two gene sets are not disjoint. Correct enrichment p-values across all tested features with the Benjamini-Hochberg procedure and retain features at FDR < 0.05. Submit the enriched GO term list to Revigo to remove redundant terms and record the positional-feature and GO-term results with FDR values. The rationale is that functional elements were predicted/transferred onto long read-defined isoforms, so enrichment tests operate on isoform-level annotation rather than gene-level reference annotation.

## Related tools

- **tappAS** (Runs the functional enrichment analysis (partially-overlapping samples z-test) comparing coDIU vs DIU genes over positional features and GO terms) — https://github.com/ConesaLab/tappAS
- **acorde** (Produces the DIU and coDIU gene sets and the IsoAnnotLite-annotated transcriptome that serve as enrichment inputs) — https://github.com/ConesaLab/acorde
- **Revigo** (Removes redundant GO terms from the enriched GO term list, leaving the non-redundant term set)

## Evaluation signals

- Enrichment output reports 91 positional functional features and 59 GO terms annotated in a significantly higher proportion in coDIU vs DIU genes at FDR < 0.05
- All reported p-values are Benjamini-Hochberg corrected across every tested functional feature, not raw
- Revigo filtering of the 59 enriched GO terms yields exactly 26 unique (non-redundant) GO terms
- The statistical test recorded for the enrichment is the partially-overlapping samples z-test, not a Fisher/chi-square test
- Results file (e.g. codiu_functional_enrichment.tsv) contains feature identifiers paired with FDR values

## Limitations

- The acorde R package does not include the functional analyses; they rely on external tools (IsoAnnotLite, tappAS), so functional annotation transfer requires a compatible pre-annotated reference (mouse RefSeq) and cannot be done de novo yet
- Reported functional insights require further experimental validation and serve mainly as hypothesis-generating output
- Using a reference that is not an updated/compatible version of the mouse RefSeq transcriptome will break IsoAnnotLite label transfer from tappAS pre-annotated references

## Evidence

- [abstract] we predict functional elements from long read-defined isoforms and provide insight into biological processes, motifs, and domains potentially controlled by the coordination of post-transcriptional: "we predict functional elements from long read-defined isoforms and provide insight into biological processes, motifs, and domains potentially controlled by the coordination of post-transcriptional"
- [intro] we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information: "we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information"
- [readme] we used IsoAnnotLite (v2.6) to transfer functional labels from tappAS’ pre-annotated mouse RefSeq 78 reference as well as from the mouse: [evidence span withheld — anchor status: external_doc]
- [readme] To use the mouse brain transcriptome in our manuscript, download the `Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3` file.: [evidence span withheld — anchor status: external_doc]
- [discussion] While these insights need to be subject to further experimental validation, they serve to illustrate the hypothesis-generating power of our pipeline.: "While these insights need to be subject to further experimental validation, they serve to illustrate the hypothesis-generating power of our pipeline."
