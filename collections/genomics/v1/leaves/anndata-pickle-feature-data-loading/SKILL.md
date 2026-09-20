---
name: anndata-pickle-feature-data-loading
description: Use when a UniFORM feature-level normalization or benchmarking task needs
  the PRAD-CyCIF single-cell feature data, delivered as Pickle *.pkl or AnnData *.h5ad
  files from Zenodo record 16383766.
when_to_use_negative:
- Input is pixel-level MTI data (OME-TIFF) — the Pixel-Level (OME-TIFF) example dataset
  from this paper is described as 'Being assembled and will be available soon', so
  it cannot be fetched from this record
- Target dataset is TMA-Lunaphore — those data are 'available from the authors upon
  request' and are not part of the public Zenodo deposit
- Data are already normalized or come from a different platform (e.g. CRC-ORION, which
  is distributed via HTAN rather than this Zenodo record)
edam_operation: http://edamontology.org/operation_2422
tools:
- name: UniFORM
  role: Provides the Python 3.10 environment, requirements.txt, and the feature-level
    input loaders that consume the .pkl/.h5ad PRAD-CyCIF files
  repo: https://github.com/kunlunW/UniFORM
- name: Python
  role: Runtime for loading Pickle *.pkl and AnnData *.h5ad feature tables inside
    the UniFORM conda environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:845d115f3730097cab9b0fe1cce5e275ad26cbcf352b7b872f92617ec59321b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# anndata-pickle-feature-data-loading

## Summary

Downloads and loads the PRAD-CyCIF single-cell feature-level MTI dataset from Zenodo record 16383766 into the UniFORM Python 3.10 environment, accepting either Pickle *.pkl or AnnData *.h5ad feature tables as normalization input.

## When to use

Use when a UniFORM feature-level normalization or benchmarking task needs the PRAD-CyCIF single-cell feature data, delivered as Pickle *.pkl or AnnData *.h5ad files from Zenodo record 16383766.

## When NOT to use

- Input is pixel-level MTI data (OME-TIFF) — the Pixel-Level (OME-TIFF) example dataset from this paper is described as 'Being assembled and will be available soon', so it cannot be fetched from this record
- Target dataset is TMA-Lunaphore — those data are 'available from the authors upon request' and are not part of the public Zenodo deposit
- Data are already normalized or come from a different platform (e.g. CRC-ORION, which is distributed via HTAN rather than this Zenodo record)

## Inputs

- Zenodo record 16383766 feature-level dataset: Pickle *.pkl files (PRAD-CyCIF single-cell features)
- Zenodo record 16383766 feature-level dataset: AnnData *.h5ad files (PRAD-CyCIF single-cell features)

## Outputs

- In-memory PRAD-CyCIF feature table (cells x markers) in AnnData or pandas/Pickle form, ready for UniFORM feature-level normalization
- Loaded feature data used downstream for runtime benchmarking or normalization comparisons

## How to apply

Clone the UniFORM repository (git clone https://github.com/kunlunW/UniFORM.git), create a Python 3.10 conda environment, and install dependencies with pip install -r requirements.txt. Download the PRAD-CyCIF feature-level example dataset (Pickle *.pkl and AnnData *.h5ad) from Zenodo record 16383766. Choose between the two accepted feature-level input formats — UniFORM's feature-level pipeline accepts both AnnData .h5ad and Pickle .pkl, so pick the one that fits the downstream workflow. Load the file into an in-memory feature table (cells x markers) suitable for the UniFORM pipeline of histogram calculation, functional-data registration, and normalization. Validate that the loaded table matches the PRAD-CyCIF design (20 prostate cancer whole-slide samples across 7 batches on the CyCIF platform) before proceeding to normalization or runtime benchmarking.

## Related tools

- **UniFORM** (Provides the Python 3.10 environment, requirements.txt, and the feature-level input loaders that consume the .pkl/.h5ad PRAD-CyCIF files) — https://github.com/kunlunW/UniFORM
- **Python** (Runtime for loading Pickle *.pkl and AnnData *.h5ad feature tables inside the UniFORM conda environment)

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git; conda create -n uniform python=3.10; pip install -r requirements.txt
```

## Evaluation signals

- The downloaded archive from Zenodo record 16383766 contains both Pickle *.pkl and AnnData *.h5ad feature-level files for PRAD-CyCIF
- The loaded feature table reflects the paper's PRAD-CyCIF design: 20 prostate cancer whole-slide samples across 7 batches on the CyCIF platform
- The conda environment is Python 3.10 and 'pip install -r requirements.txt' completed without errors before loading
- Downstream UniFORM steps (histogram calculation, functional-data registration, normalization) run without input-format errors on the loaded table

## Limitations

- The Pixel-Level (OME-TIFF) example dataset and its tutorial are not yet available ('Being assembled and will be available soon'), so this loader covers feature-level data only
- No software version numbers are stated for any of the tools used, so environment reproducibility beyond Python 3.10 is not pinned in the paper
- UniFORM's effectiveness depends on the quality of the MTI dataset and the cell/nuclei segmentation masks used to extract the signals loaded here

## Evidence

- [other] Download the PRAD-CyCIF feature-level example dataset (Pickle *.pkl and AnnData *.h5ad) from Zenodo record 16383766.: [evidence span withheld — anchor status: not_found]
- [readme] README: feature-level input formats: "UniFORM’s feature‐level pipeline accepts two formats. Choose the one that fits your workflow:"
- [discussion] Zenodo deposit of PRAD-CyCIF: "The PRAD-CyCIF single-cell feature data have been deposited on Zenodo and are publicly available as of the date of publication."
