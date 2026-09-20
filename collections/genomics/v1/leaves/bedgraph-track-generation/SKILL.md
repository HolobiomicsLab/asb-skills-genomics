---
name: bedgraph-track-generation
description: Use when you have a mapped modBAM of nanopore gDNA with m6A modifications
  and need open-chromatin signal regions plus per-genome BedGraph tracks from the
  Dogme open-chromatin entry point.
when_to_use_negative:
- Input is RNA or cDNA rather than genomic DNA modBAMs — the modkit open-chromatin
  workflow is intended for mapped BAMs with m6A modifications on gDNA.
- modkit version is below 0.5 — the open-chromatin support requires modkit 0.5 and
  higher.
- The goal is transcript isoform quantification rather than chromatin accessibility
  — that is handled by the kallisto/LR-Kallisto branch of Dogme.
tools:
- name: Dogme
  role: Nextflow pipeline whose open-chromatin workflow/entry point chains modkit
    pileup, filtering (filterbed.py), per-chromosome consolidation, and BED/BedGraph
    export.
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: v0.5+ tool run on the m6A modBAM to compute modification pileups that yield
    open-chromatin regions and per-genome BedGraph signal.
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Upstream basecaller that produces the merged BAMs minimap2 maps into the modBAMs
    consumed by the open-chromatin module.
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Maps the Dorado basecalled gDNA reads to the reference genome to create the
    aligned modBAM input.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:6f97999edd410d49e25862ed23d822102a036c3f4c300c084700c6463bef8ab5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - minimap2
  - modkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bedgraph-track-generation

## Summary

Within the Dogme Nextflow pipeline, this skill converts modkit (v0.5+) pileup calls of exogenous-m6A-labeled genomic DNA into a consolidated per-genome open-chromatin BED file and BedGraph tracks, enabling identification of accessible regulatory regions and genome-browser visualization.

## When to use

Use when you have a mapped modBAM of nanopore gDNA with m6A modifications and need open-chromatin signal regions plus per-genome BedGraph tracks from the Dogme open-chromatin entry point.

## When NOT to use

- Input is RNA or cDNA rather than genomic DNA modBAMs — the modkit open-chromatin workflow is intended for mapped BAMs with m6A modifications on gDNA.
- modkit version is below 0.5 — the open-chromatin support requires modkit 0.5 and higher.
- The goal is transcript isoform quantification rather than chromatin accessibility — that is handled by the kallisto/LR-Kallisto branch of Dogme.

## Inputs

- m6A-modified genomic DNA modBAM from Dorado basecalling and minimap2 mapping
- Genome reference used by the Dogme config (for per-genome consolidation)

## Outputs

- open_chromatin.bed.gz — compressed BED of consolidated per-genome open-chromatin intervals
- Per-genome BedGraph track files for genome-browser visualization
- Filtered modkit bedMethyl/BED outputs produced by scripts/filterbed.py

## How to apply

Load the m6A-labeled gDNA modBAM produced by the upstream Dogme stage (Dorado basecalling merged and mapped with minimap2), then run modkit 0.5 or higher to compute per-reference-position modification pileups. Filter the modkit BED output with Dogme's scripts/filterbed.py using the params.minCov (minimum coverage) and params.perMod (per-modification fraction) thresholds to retain high-confidence m6A-modified positions marking open chromatin. Consolidate the filtered per-chromosome intervals into a single per-genome interval set, processing one chromosome at a time to limit memory use, and export the result as a compressed BED (open_chromatin.bed.gz) plus per-genome BedGraph tracks. Judge success by validating that the output BED exists, is valid compressed BED format, and contains the expected consolidated per-genome m6A open-chromatin intervals.

## Related tools

- **Dogme** (Nextflow pipeline whose open-chromatin workflow/entry point chains modkit pileup, filtering (filterbed.py), per-chromosome consolidation, and BED/BedGraph export.) — https://github.com/mortazavilab/dogme
- **modkit** (v0.5+ tool run on the m6A modBAM to compute modification pileups that yield open-chromatin regions and per-genome BedGraph signal.) — https://github.com/nanoporetech/modkit
- **Dorado** (Upstream basecaller that produces the merged BAMs minimap2 maps into the modBAMs consumed by the open-chromatin module.) — https://github.com/nanoporetech/dorado
- **minimap2** (Maps the Dorado basecalled gDNA reads to the reference genome to create the aligned modBAM input.)

## Examples

```
python scripts/filterbed.py --input modkit_pileup.bed.gz --minCov ${params.minCov} --perMod ${params.perMod}
```

## Evaluation signals

- Output BED file exists at the expected path (open_chromatin.bed.gz) and decompresses as valid compressed BED format.
- The BED contains per-genome consolidated m6A open-chromatin intervals derived from the modkit pileup, consistent with the filtered high-confidence positions.
- BedGraph track files are produced per genome and are loadable as genome-browser signal tracks.
- Filtered BED respects the configured params.minCov and params.perMod thresholds applied by scripts/filterbed.py.
- Memory footprint stays bounded because consolidation processes one chromosome at a time.

## Limitations

- Open-chromatin support requires modkit 0.5 or higher; earlier versions cannot run this workflow.
- Requires exogenous m6A labeling of gDNA — accessibility calls are read out through m6A modification signal.
- Per-chromosome processing is used to limit annotation memory use, so fully assembled per-genome outputs depend on completing all chromosomes.

## Evidence

- [readme] modkit open chromatin support: [evidence span withheld — anchor status: external_doc]
- [readme] output formats: [evidence span withheld — anchor status: external_doc]
- [readme] filterbed thresholds: [evidence span withheld — anchor status: external_doc]
- [intro] gDNA modBAM provenance: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [discussion] memory rationale: "Reduced annotation memory use by processing one chromosome at a time."
