---
name: reference-free-single-cell-annotation-with-atlas
description: Use when a query scRNA-seq dataset has no matched reference dataset and
  cell-type labels are needed; mouse queries can be mapped to the MCA atlas and human
  queries to the HCL atlas.
when_to_use_negative:
- A study-specific, well-annotated single-cell reference dataset matching the query
  tissue exists — use standard scMAGIC RBC (scMAGIC() or scMAGIC_Seurat) instead of
  the atlas mode
- The query tissue's cell types are largely absent from the MCA/HCL atlas — labeled
  accuracy cannot be meaningfully computed and atlas annotation will be unreliable
- The query is from an organism other than mouse or human, for which no MCA/HCL atlas
  reference is shipped with scMAGIC
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Primary annotation method; scMAGIC_atlas uses the MCA or HCL atlas expression
    matrix plus marker-gene identification and two-round RBC
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: scMCA
  role: Atlas-native baseline classifier for mouse queries using MCA, run for accuracy
    comparison
  repo: https://github.com/ggjlab/scMCA
- name: scHCL
  role: Atlas-native baseline classifier for human queries using HCL, run for accuracy
    comparison
  repo: https://github.com/ggjlab/scHCL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:609cad0f544c4fd308626c327f8c7621057fd4911d1d38d59189e7394ee1a265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - scHCL
  - scMAGIC
  - scMCA
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# reference-free-single-cell-annotation-with-atlas

## Summary

Annotate query scRNA-seq cells when no study-specific reference dataset exists, by running scMAGIC_atlas with a large-scale atlas (MCA for mouse, HCL for human) as the reference and comparing against the atlas-native tools scMCA and scHCL.

## When to use

Use when a query scRNA-seq dataset has no matched reference dataset and cell-type labels are needed; mouse queries can be mapped to the MCA atlas and human queries to the HCL atlas.

## When NOT to use

- A study-specific, well-annotated single-cell reference dataset matching the query tissue exists — use standard scMAGIC RBC (scMAGIC() or scMAGIC_Seurat) instead of the atlas mode
- The query tissue's cell types are largely absent from the MCA/HCL atlas — labeled accuracy cannot be meaningfully computed and atlas annotation will be unreliable
- The query is from an organism other than mouse or human, for which no MCA/HCL atlas reference is shipped with scMAGIC

## Inputs

- Query scRNA-seq expression matrix (e.g. mouse neocortex GSE115746, mouse duodenum GSE92332, human pancreas GSE84133, human PBMC GSE132044)
- Atlas reference expression matrix and marker-gene tables from scMAGIC: data('MCA_ref') for mouse, data('HCL_ref') for human
- True cell-type labels for the query dataset (for benchmark evaluation)

## Outputs

- Per-cell predicted cell-type labels from scMAGIC_atlas
- Accuracy and labeled accuracy (plus balanced variants) per method-dataset pair via table(true_label, predicted_label)
- Comparison table / Figure 7A-style bar plot of scMAGIC versus scMCA/scHCL

## How to apply

Load the atlas expression matrices and marker-gene tables shipped with scMAGIC (data('MCA_ref') for mouse, data('HCL_ref') for human). Run scMAGIC_atlas on the query expression matrix with atlas='MCA' or 'HCL', using type_ref='sum-counts', use_RUVseq=FALSE, min_cell=5, and multiple threads (e.g. num_threads=10). scMAGIC first identifies a candidate reference cell type by expression-profile correlation, validates identity via the reference cell type's marker genes, then performs a second-round RBC using confidently validated query cells as a new reference, which eliminates batch effects between atlas and query. For benchmarking, run the atlas-native classifiers scMCA (mouse) and scHCL (human) on the same queries, tabulate predicted versus true labels with table(true_label, predicted_label), and treat cell types absent from the atlas as unassignable when computing labeled accuracy.

## Related tools

- **scMAGIC** (Primary annotation method; scMAGIC_atlas uses the MCA or HCL atlas expression matrix plus marker-gene identification and two-round RBC) — https://github.com/TianLab-Bioinfo/scMAGIC
- **scMCA** (Atlas-native baseline classifier for mouse queries using MCA, run for accuracy comparison) — https://github.com/ggjlab/scMCA
- **scHCL** (Atlas-native baseline classifier for human queries using HCL, run for accuracy comparison) — https://github.com/ggjlab/scHCL

## Examples

```
output.scMAGIC <- scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA', type_ref = 'sum-counts', use_RUVseq = FALSE, min_cell = 5, num_threads = 10)
```

## Evaluation signals

- scMAGIC accuracy on the four atlas-annotated query datasets should fall in the reported 0.88–0.98 range (balanced accuracy 0.77–0.88)
- Labeled accuracy should be 0.92–0.98 with labeled balanced accuracy 0.85–0.96, clearly exceeding scMCA/scHCL labeled balanced accuracy of 0.25–0.46
- Confusion-matrix dimensions from table(true_label, predicted_label) should cover query cell types, with cell types absent from the atlas excluded from labeled-accuracy computation
- Reproduced Figure 7A bar plot should show scMAGIC bars above scMCA/scHCL bars for all four query datasets

## Limitations

- Query datasets may include cell types not covered by the atlas (e.g. the mouse duodenum dataset), which must be treated as unassignable for labeled accuracy
- Atlas-based annotation achieves only 'reasonably high' accuracy (0.88–0.98), lower than what matched reference datasets deliver
- No version numbers for scMAGIC, R, or dependent packages are stated; no archived code release or data deposition DOI is provided beyond GitHub links

## Evidence

- [abstract] atlas annotation finding: "when no reference dataset is available, scMAGIC can annotate query cells with reasonably high accuracy by using an atlas dataset as the reference"
- [methods] atlas classification call: "output.scMAGIC <- scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA',"
- [discussion] atlas-native baselines underperform: "scMCA and scHCL, the tools provided by MCA and HCL, do not work well in the four query datasets"
- [discussion] uncovered cell types: "a query dataset may include some cell types not covered by the atlas, such as the mouse duodenum dataset (32)"
- [methods] installation: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
