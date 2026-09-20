---
name: visium-hd-bin-to-cell-reconstruction
description: Use when working with Visium HD 2 μm bin-resolution data (with a reference
  H&E or IF morphology image) and you need cell-level objects better than the default
  8 μm bins for downstream annotation or spatial analysis.
when_to_use_negative:
- Data are already at cell resolution (e.g. conventional single-cell RNA-seq or aggregated
  8 μm bins) — bin grouping is unnecessary
- No reference morphology image is available and only the 8 μm output of Space Ranger
  is needed
- The task is custom probe design for Visium HD panels — use gene2probe instead
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: bin2cell
  role: 'Core package: destripe correction, StarDist segmentation wrapper (scaled_he_image,
    grid_image, stardist), label insertion/expansion, salvage_secondary_labels, and
    bin_to_cell grouping of 2 μm bins into cells'
  repo: https://github.com/Teichlab/bin2cell
- name: StarDist
  role: Nucleus segmentation on the scaled H&E/IF image; optional bin2cell dependency
    group installed via pip install bin2cell[stardist]
  repo: https://github.com/Teichlab/bin2cell
- name: CellTypist
  role: Used downstream to annotate both reconstructed cells and 8 μm bins and compare
    prediction confidence (e.g. demo notebook notebooks/destripe_celltypist.ipynb)
- name: Visium HD
  role: Source platform providing 2 μm bin transcriptomic data paired with a reference
    morphology image
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae546
    title: Bin2cell reconstructs cells from high resolution Visium HD data
  source_content_hash: sha256:964ac0e780eb2377c7a881fc0232522f8a699bcbfe482a110f75a3bfe3325868
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bin2cell
  tools:
  - CellTypist
  - Python
  - StarDist
  - Visium HD
  - bin2cell
  - numpy
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# visium-hd-bin-to-cell-reconstruction

## Summary

Reconstructs cells from 2 μm Visium HD bins by grouping bins via morphology-image segmentation (StarDist on H&E or IF images) and/or gene-expression visualisation, after correcting the bin-dimension destriping artifact, to produce a cell-level AnnData that outperforms default 8 μm bins.

## When to use

Use when working with Visium HD 2 μm bin-resolution data (with a reference H&E or IF morphology image) and you need cell-level objects better than the default 8 μm bins for downstream annotation or spatial analysis.

## When NOT to use

- Data are already at cell resolution (e.g. conventional single-cell RNA-seq or aggregated 8 μm bins) — bin grouping is unnecessary
- No reference morphology image is available and only the 8 μm output of Space Ranger is needed
- The task is custom probe design for Visium HD panels — use gene2probe instead

## Inputs

- Visium HD 2 μm bin-resolution count matrix from Space Ranger
- Reference H&E or immunofluorescence (IF) morphology image
- Optional: CellTypist model (e.g. trained on Yao et al. 2023 mouse brain atlas) for downstream confidence comparison

## Outputs

- Reconstructed cell-by-gene AnnData object (.h5ad) with cell-level observations, spatial coordinates, and sharper morphology images
- Comparison table of CellTypist confidence and per-cell gene counts (b2c cells vs 8 μm bins), e.g. celltypist_b2c_vs_8um_comparison.csv

## How to apply

Start from 2 μm bin counts plus the reference morphology image and generate a scaled image with b2c.scaled_he_image (or b2c.scaled_if_image) at an mpp in the recommended 0.2–0.5 range, since StarDist models were trained at mpp ~0.3. Correct the novel technical artifact stemming from variable bin dimensions with b2c.destripe before segmentation. Segment nuclei with b2c.stardist (e.g. prob_thresh=0.05 as the reported sweet spot between the overly stringent default and overly lenient 0.01), expand labels with b2c.expand_labels, and map them onto the 2 μm bin grid with b2c.insert_labels. Recover transcripts from bins filtered out during segmentation with b2c.salvage_secondary_labels, then group bins into cells with b2c.bin_to_cell, saving the cell-by-gene AnnData as .h5ad. The end result carries spatial information and sharper morphology images, and the reconstructed cells should be compared against the retained default 8 μm bin object.

## Related tools

- **bin2cell** (Core package: destripe correction, StarDist segmentation wrapper (scaled_he_image, grid_image, stardist), label insertion/expansion, salvage_secondary_labels, and bin_to_cell grouping of 2 μm bins into cells) — https://github.com/Teichlab/bin2cell
- **StarDist** (Nucleus segmentation on the scaled H&E/IF image; optional bin2cell dependency group installed via pip install bin2cell[stardist]) — https://github.com/Teichlab/bin2cell
- **CellTypist** (Used downstream to annotate both reconstructed cells and 8 μm bins and compare prediction confidence (e.g. demo notebook notebooks/destripe_celltypist.ipynb))
- **Visium HD** (Source platform providing 2 μm bin transcriptomic data paired with a reference morphology image)

## Examples

```
import bin2cell as b2c; b2c.scaled_he_image(path, output_path, mpp=0.3); b2c.destripe(bins); b2c.stardist(mpp=0.3, prob_thresh=0.05); b2c.expand_labels(labels); b2c.insert_labels(...); b2c.salvage_secondary_labels(bdata); b2c.bin_to_cell(bdata); bdata.write('crc_bin2cell_cells.h5ad')
```

## Evaluation signals

- The emitted .h5ad loads as an AnnData object with cell-level observations and spatial coordinates
- Per-cell gene detection is higher in b2c cells than in default 8 μm bins (reported: all 113 mouse brain cell types have higher average genes detected in b2c)
- CellTypist median confidence is higher in b2c cells than 8 μm bins for most cell types (reported: 90 of 113 cell types on mouse brain)
- Reconstructed cells recapitulate fine morphological structures (e.g. CRC grouping correctly predicted as CMS2 where 8 μm bins err)
- Segmentation quality depends on image scale: mpp within 0.2–0.5 (e.g. 0.35 drastically improved segmentation of a lung IF image versus 0.5)

## Limitations

- StarDist is an optional dependency group; a default pip install of bin2cell lacks the segmentation dependency, and TensorFlow is additionally required for segmentation
- numpy is capped at 1.x with no exact pin, leaving a loose reproducibility gap
- StarDist model performance is highly dependent on input image resolution; wrong mpp (e.g. 0.5 with default prob_thresh) can drastically degrade segmentation
- No versioned release artifacts, DOIs, or dataset accessions are stated; only the source repository is referenced
- Alternative expansion behavior depends on user-controlled parameters (k, max_call_distance) without default performance characterization

## Evidence

- [abstract] Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information: "Bin2cell reconstructs cells from the highest resolution data (2 μm bins) by leveraging morphology image segmentation and gene expression information"
- [methods] StarDist models trained at mpp ~0.3, recommended range 0.2-0.5: [evidence span withheld — anchor status: not_found]
- [methods] mpp=0.35 drastically improved StarDist segmentation of a lung IF image: "A new image with ``mpp=0.35`` was generated, and ``prob_thresh`` was set back to 0.1. The segmentation drastically improved."
- [readme] README: install bin2cell; StarDist optional dependency and TensorFlow required: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion: numpy capped at 1.x and StarDist moved to optional dependency: "cap `numpy` at 1.x for the time being"
