---
name: open-chromatin-bed-generation
description: Use when starting from an m6A-modification-bearing mapped gDNA modBAM
  and the goal is to identify accessible (open) chromatin regions as a per-genome
  BED and BedGraph for browser visualization.
when_to_use_negative:
- Input is direct RNA or cDNA data — the open-chromatin workflow targets mapped BAMs
  with m6A modifications from genomic DNA; RNA/cDNA reads route to the kallisto/LR-Kallisto
  transcript quantification branch instead.
- The modBAM lacks m6A modification calls (e.g., unlabeled gDNA or a model without
  modification tagging) — modkit pileup would yield no accessibility signal.
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: Dogme
  role: Nextflow pipeline orchestrating the open-chromatin workflow entry point, including
    filtering (filterbed.py) and per-genome consolidation
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Computes modification pileups from the m6A modBAM; version 0.5 and higher
    provides the open-chromatin calling used here
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Upstream basecaller producing the modBAM with modification tags consumed by
    this skill
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Maps the basecalled gDNA reads to the reference genome before modification
    pileup
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:dc7543f0e22f700c4ae329ecd07a3e7665416854a9fb21ac789b5d994d8d246e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# open-chromatin-bed-generation

## Summary

Generates consolidated open-chromatin calls from m6A-labeled nanopore genomic DNA by running modkit (v0.5+) pileup on the mapped modBAM, filtering high-confidence m6A positions, and exporting a per-genome BED plus BedGraph tracks.

## When to use

Use when starting from an m6A-modification-bearing mapped gDNA modBAM and the goal is to identify accessible (open) chromatin regions as a per-genome BED and BedGraph for browser visualization.

## When NOT to use

- Input is direct RNA or cDNA data — the open-chromatin workflow targets mapped BAMs with m6A modifications from genomic DNA; RNA/cDNA reads route to the kallisto/LR-Kallisto transcript quantification branch instead.
- The modBAM lacks m6A modification calls (e.g., unlabeled gDNA or a model without modification tagging) — modkit pileup would yield no accessibility signal.

## Inputs

- m6A-labeled genomic DNA modBAM from Dorado basecalling, BAM merging, and minimap2 mapping
- Compressed modkit bedMethyl/pileup .bed.gz with m6A calls per reference position

## Outputs

- open_chromatin.bed.gz — consolidated per-genome open-chromatin intervals (compressed BED)
- Per-genome BedGraph tracks of open-chromatin signal for genome-browser visualization

## How to apply

Take the exogenous-m6A-labeled gDNA modBAM produced by the upstream Dogme stage (Dorado basecalling, BAM merge, minimap2 mapping) and run modkit 0.5 or higher to compute a pileup of modification signals per reference position. Filter the pileup to retain high-confidence m6A-modified positions — Dogme's filterbed.py applies minimum-coverage and per-mod thresholds (params.minCov and params.perMod) on the compressed .bed.gz modkit output. Consolidate the filtered per-chromosome calls into one per-genome open-chromatin interval set, processing one chromosome at a time to limit memory use, as Dogme does for its annotation steps. Export the consolidated intervals as a compressed BED (open_chromatin.bed.gz) and as per-genome BedGraph tracks; the README notes the open-chromatin workflow 'produces both a bed files of regions and a bedgraph per genome'. The rationale is that m6A marks deposited on accessible gDNA serve as a proxy for open chromatin, so modified-base frequency at each reference position reports accessibility.

## Related tools

- **Dogme** (Nextflow pipeline orchestrating the open-chromatin workflow entry point, including filtering (filterbed.py) and per-genome consolidation) — https://github.com/mortazavilab/dogme
- **modkit** (Computes modification pileups from the m6A modBAM; version 0.5 and higher provides the open-chromatin calling used here) — https://github.com/nanoporetech/modkit
- **Dorado** (Upstream basecaller producing the modBAM with modification tags consumed by this skill) — https://github.com/nanoporetech/dorado
- **minimap2** (Maps the basecalled gDNA reads to the reference genome before modification pileup)

## Examples

```
modkit open-chromatin predict aligned.mod.bam
```

## Evaluation signals

- Output BED file exists, is a valid compressed BED format, and is non-empty.
- BED intervals represent the expected consolidated per-genome m6A open-chromatin set derived from the modkit pileup (single per-genome file, not per-chromosome fragments).
- A BedGraph track is produced for each genome alongside the BED regions.
- Filtered positions meet the configured params.minCov and params.perMod thresholds applied by filterbed.py.
- Downstream consistency check: open-chromatin calls should align to reference positions with high m6A fraction in the input modBAM.

## Limitations

- Requires modkit 0.5 or higher; the open-chromatin entry point was introduced in Dogme 1.4.0 and is absent from earlier releases.
- Dogme reduces annotation memory use by processing one chromosome at a time, so memory behavior depends on per-chromosome processing being preserved.
- Only meaningful for gDNA with exogenous m6A labeling; it does not replace transcript-level modification or quantification workflows.

## Evidence

- [readme] Modkit Open Chromatin support: [evidence span withheld — anchor status: external_doc]
- [readme] Output artifacts: [evidence span withheld — anchor status: external_doc]
- [readme] Filtering thresholds: [evidence span withheld — anchor status: external_doc]
- [intro] Modification extraction step: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [discussion] Memory strategy: "Reduced annotation memory use by processing one chromosome at a time."
