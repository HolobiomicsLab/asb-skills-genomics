---
name: cross-species-cell-type-mapping
description: Use when the reference scRNA-seq dataset and the query dataset come from
  different species (e.g. mouse reference, human query or cross-species pairs), so
  gene identifiers must first be mapped to orthologs before reference-based classification.
when_to_use_negative:
- Reference and query datasets are from the same species and share gene identifiers
  — direct RBC without ortholog mapping applies
- No labeled reference or atlas is available at all — reference-free clustering, not
  cross-species RBC, is appropriate (though scMAGIC_atlas with MCA/HCL remains an
  option within the same species)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Two-round reference-based classifier run on ortholog-mapped cross-species
    reference/query matrices
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: homologene
  role: Maps genes between species to one-to-one orthologs so cross-species matrices
    share a gene space
- name: SingleCellNet
  role: Competing machine-learning-based RBC method with explicit cross-species training/query
    modes used at optimal settings in benchmarks
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) to give all
    methods comparable input matrices
- name: SciBet
  role: Competing statistical-metric-based method; installed from GitHub for benchmarking,
    SciBet_R variant used on Windows
  repo: https://github.com/PaulingLiu/scibet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:4423c879a33cfb67aa418b1740187a78157be20a02a2d6febf9eb167d2ae102e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - AUCell
  - CHETAH
  - COSG
  - CaSTLe
  - SVMrejection
  - SciBet
  - Seurat
  - SingleCellNet
  - SingleR
  - homologene
  - mclust
  - parallel
  - pcaPP
  - scClassify
  - scID
  - scMAGIC
  - scPred
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# cross-species-cell-type-mapping

## Summary

Map query single-cell RNA-seq annotations from one species onto another by restricting reference and query matrices to one-to-one orthologous genes (e.g. via homologene) and running reference-based classification (scMAGIC) on the harmonized matrices, as done in the cross-species situation of the 86-benchmark scMAGIC evaluation.

## When to use

Use when the reference scRNA-seq dataset and the query dataset come from different species (e.g. mouse reference, human query or cross-species pairs), so gene identifiers must first be mapped to orthologs before reference-based classification.

## When NOT to use

- Reference and query datasets are from the same species and share gene identifiers — direct RBC without ortholog mapping applies
- No labeled reference or atlas is available at all — reference-free clustering, not cross-species RBC, is appropriate (though scMAGIC_atlas with MCA/HCL remains an option within the same species)

## Inputs

- Reference scRNA-seq expression matrix with cell-type labels from one species (e.g. mouse MCA or HCL atlas data)
- Query scRNA-seq expression matrix from another species (raw count matrix)
- Ortholog/gene homology mapping (homologene) for the two species
- Atlas expression matrix (e.g. MCA_ref) used by scMAGIC for marker-gene identification

## Outputs

- Per-cell predicted cell-type labels for the query dataset (scMAGIC.tag)
- Accuracy, balanced accuracy and cluster accuracy per benchmark test for each RBC method
- Median performance ranks per method across the cross-species benchmark tests

## How to apply

First convert the reference and query expression matrices to a shared gene space by mapping genes to one-to-one orthologs with homologene (version 1.4.68.19.3.27 in the scMAGIC environment). Preprocess each reference-query pair with Seurat (NormalizeData, FindVariableFeatures, ScaleData) so all competing methods receive comparable input. Then run scMAGIC on the ortholog-mapped matrices — first identifying candidate cell types by expression-profile correlation, verifying identities via reference cell-type marker genes, and reclassifying with confidently validated query cells as a new local reference to absorb batch effects. Benchmark against the same 13 competing RBC methods (statistical-metric-, tree- and machine-learning-based) at their optimal settings and score accuracy, balanced accuracy and cluster accuracy per benchmark test, as in the cross-species situation among the 86 benchmark tests.

## Related tools

- **scMAGIC** (Two-round reference-based classifier run on ortholog-mapped cross-species reference/query matrices) — https://github.com/TianLab-Bioinfo/scMAGIC
- **homologene** (Maps genes between species to one-to-one orthologs so cross-species matrices share a gene space)
- **SingleCellNet** (Competing machine-learning-based RBC method with explicit cross-species training/query modes used at optimal settings in benchmarks)
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) to give all methods comparable input matrices)
- **SciBet** (Competing statistical-metric-based method; installed from GitHub for benchmarking, SciBet_R variant used on Windows) — https://github.com/PaulingLiu/scibet

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- Predicted labels can be cross-tabulated against held-out true query labels (table(label_sc, pred.scMAGIC)) to compute accuracy, balanced accuracy and cluster accuracy
- Across the 86 benchmark tests scMAGIC should attain a median accuracy rank of 1, ahead of Seurat v4 (median rank 4) and scSemiCluster (median rank 5), matching Figure 5A
- The ortholog-mapped reference and query matrices must have identical gene identifiers (same number of rows) before classification
- scMAGIC should remain competitive or superior even when significant batch effects separate reference and query, which is the regime the second-round RBC targets
- The output object exposes scMAGIC.tag and is compatible with Seurat objects as input and output

## Limitations

- Dataset accession codes and tool version numbers are deferred to Materials and Methods and not stated in the discussion; no archived code release or Zenodo DOI is given for scMAGIC beyond the GitHub repository
- Tools like scMCA/scHCL that use MCA and HCL atlases directly 'do not work well in the four query datasets', so atlas-based annotation quality varies by query dataset
- Windows users must switch the high-variable-gene method to 'SciBet_R' and pre-configure the reticulate/numpy environment before running scMAGIC

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [intro] Benchmark design covering user situations including cross-species pairs: "We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets for these situations."
- [readme] ortholog mapping dependency installed in the scMAGIC environment: "install.packages('homologene')      # 1.4.68.19.3.27"
- [readme] two-round RBC overcomes interspecies/batch effects between reference and query: "scMAGIC conducts a second-round of RBC by using target single cells with verified cell identity as the new reference to overcome the significant batch effects between the reference and the query data."
- [methods] concrete scMAGIC call on reference and query matrices: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] competing method with explicit cross-species training/query modes: [evidence span withheld — anchor status: external_doc]
