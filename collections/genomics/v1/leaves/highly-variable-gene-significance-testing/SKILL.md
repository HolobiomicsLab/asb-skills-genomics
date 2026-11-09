---
name: highly-variable-gene-significance-testing
description: Use when a scRNA-seq expression matrix needs a statistically tested set
  of significantly highly variable genes, and dropout-based methods (M3Drop/DANB)
  should be benchmarked against a mean-variance-based alternative.
when_to_use_negative:
- The dataset is UMI-quantified and the goal is dropout-based or Pearson-residual
  feature selection — M3Drop's DANB model (NBumi functions) is tailored for UMIs instead
- The gene set is already selected and only downstream clustering or pseudotime analysis
  is required — feature selection would be redundant
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: M3Drop
  role: R package that includes the Brennecke et al. (2015) significantly-HVG detection
    algorithm for comparison against its dropout-based (modified Michaelis-Menten)
    feature selection
  repo: github.com/tallulandrews/M3Drop
- name: M3DExampleData
  role: Provides the example scRNA-seq expression matrix used as input when running
    the Brennecke HVG test
  repo: github.com/tallulandrews/M3DExampleData
- name: remotes
  role: Used to install M3Drop and M3DExampleData from GitHub (install_github)
- name: R
  role: Runtime environment in which the M3Drop functions and HVG comparison are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/1873-3468.12684
    title: Computational approaches for interpreting sc<scp>RNA</scp>‐seq data
  source_content_hash: sha256:313c422270cc067c0bd66c50f9a42d7fae2d322e2aa605559ab8342b55eb52e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - M3DExampleData
  - M3Drop
  - R
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# highly-variable-gene-significance-testing

## Summary

Detects significantly highly variable genes (HVGs) in scRNA-seq data using the Brennecke et al. (2015) mean-variance significance algorithm, which the M3Drop R package ships alongside its dropout-based (Michaelis-Menten) feature selection specifically for comparison purposes.

## When to use

Use when a scRNA-seq expression matrix needs a statistically tested set of significantly highly variable genes, and dropout-based methods (M3Drop/DANB) should be benchmarked against a mean-variance-based alternative.

## When NOT to use

- The dataset is UMI-quantified and the goal is dropout-based or Pearson-residual feature selection — M3Drop's DANB model (NBumi functions) is tailored for UMIs instead
- The gene set is already selected and only downstream clustering or pseudotime analysis is required — feature selection would be redundant

## Inputs

- scRNA-seq expression matrix (gene-by-cell count matrix), e.g. example data from the M3DExampleData R package
- Optionally, spike-in control gene measurements for estimating technical mean-variance behaviour

## Outputs

- List/data frame of genes flagged as significantly highly variable (e.g. exported to hvg_brennecke.csv)
- A comparable HVG set to contrast with M3Drop's dropout-based feature selection output

## How to apply

Load an example scRNA-seq expression matrix from the M3DExampleData package, which is installed alongside M3Drop from GitHub or Bioconductor. Run the Brennecke et al. (2015) routine included in M3Drop: it models technical variability (mean-variance behaviour, compatible with spike-in-based fitting) and tests each gene for significantly elevated squared expression variation rather than fitting the modified Michaelis-Menten dropout curve. Extract the genes flagged as significantly highly variable (e.g. writing them to hvg_brennecke.csv) and compare this gene set against the dropout-based M3Drop feature selection to see which genes each strategy deems low-technical-noise for downstream clustering or pseudotime analysis. The comparison is the point: the Brennecke implementation is bundled explicitly as a reference method, so treat its output as the significance-testing baseline.

## Related tools

- **M3Drop** (R package that includes the Brennecke et al. (2015) significantly-HVG detection algorithm for comparison against its dropout-based (modified Michaelis-Menten) feature selection) — github.com/tallulandrews/M3Drop
- **M3DExampleData** (Provides the example scRNA-seq expression matrix used as input when running the Brennecke HVG test) — github.com/tallulandrews/M3DExampleData
- **remotes** (Used to install M3Drop and M3DExampleData from GitHub (install_github))
- **R** (Runtime environment in which the M3Drop functions and HVG comparison are executed)

## Examples

```
require("remotes"); install_github('tallulandrews/M3Drop'); install_github('tallulandrews/M3DExampleData')
```

## Evaluation signals

- The output gene list contains only genes tested for significantly elevated squared expression variation, not genes ranked by dropout rate (a Michaelis-Menten curve output indicates the wrong method was used)
- The returned HVG set is directly comparable (same gene identifiers, same input matrix) to the dropout-based M3Drop selection for a fair benchmark
- M3Drop and M3DExampleData install successfully via install_github or BiocManager::install before the test is run
- The significance calls reflect the Brennecke et al. (2015) mean-variance/technical-variability model, consistent with the bundled 'for comparison' implementation

## Limitations

- The Brennecke implementation is included in M3Drop primarily as a comparison baseline against dropout-based feature selection, not as the package's primary method
- M3Drop's alternative model, the Depth-Adjusted Negative Binomial (DANB), is specifically tailored for UMI-quantified datasets; the mean-variance significance test's assumptions differ and may not suit UMI data the same way
- The feature-selection methods are intended to reduce technical noise for downstream analyses such as clustering or pseudotime analysis — they are not validated as final gene-level biological results

## Evidence

- [intro] Brennecke algorithm included in M3Drop: "the algorithm presented in Brennecke et al (2015) for detection of significantly highly variable genes is included."
- [readme] Bundled for comparison against dropout-based selection: "For comparison, the algorithm presented in Brennecke et al (2015) for detection of significantly highly variable genes is included."
- [readme] Dropout-based feature selection purpose: "Functions are provided for fitting each model as well as performing dropout-based feature selection. These can be used to reduce technical noise in downstream analyses such as clustering or"
- [methods] Installation of the package containing the algorithm: "require("remotes")
install_github('tallulandrews/M3Drop')"
- [methods] Example data source for the input matrix: "install_github('tallulandrews/M3DExampleData')"
