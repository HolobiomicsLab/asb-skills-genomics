---
name: genomic-interval-consolidation
description: Use when you have an exogenous-m6A-labeled nanopore genomic DNA modBAM
  and need to convert modkit pileup signals into consolidated, high-confidence open-chromatin
  intervals and per-genome BedGraph tracks.
when_to_use_negative:
- Input is direct RNA or cDNA data — Dogme routes those to kallisto/LR-Kallisto transcript
  quantification, not open-chromatin calling.
- The modBAM was not basecalled with m6A-modification models — modkit pileup will
  find no m6A signal to consolidate.
- modkit is older than v0.5 — the open-chromatin workflow explicitly requires modkit
  0.5 and higher.
tools:
- name: modkit
  role: computes pileups of m6A modification signals from the gDNA modBAM (v0.5+ required
    for open chromatin support)
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Nextflow pipeline orchestrating the open-chromatin workflow, including filterbed.py
    threshold filtering and per-chromosome consolidation into BED/BedGraph outputs
  repo: https://github.com/mortazavilab/dogme
- name: dorado
  role: upstream basecaller producing the modBAM with m6A calls that feeds the open-chromatin
    module
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: maps Dorado basecalled reads to the reference genome to create the mapped
    BAM used for pileup
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:302bc9b7ef2c4c2b9968010d094f71f88c5f055f638080fcf5d2ce5ccb2820a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - dorado
  - minimap2
  - modkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-interval-consolidation

## Summary

Consolidates per-chromosome m6A open-chromatin modification calls from modkit pileups of nanopore gDNA modBAMs into a single per-genome interval set, exported as a compressed BED file (open_chromatin.bed.gz) and BedGraph tracks. This is the open-chromatin module of the Dogme pipeline for m6A-labeled genomic DNA.

## When to use

Use when you have an exogenous-m6A-labeled nanopore genomic DNA modBAM and need to convert modkit pileup signals into consolidated, high-confidence open-chromatin intervals and per-genome BedGraph tracks.

## When NOT to use

- Input is direct RNA or cDNA data — Dogme routes those to kallisto/LR-Kallisto transcript quantification, not open-chromatin calling.
- The modBAM was not basecalled with m6A-modification models — modkit pileup will find no m6A signal to consolidate.
- modkit is older than v0.5 — the open-chromatin workflow explicitly requires modkit 0.5 and higher.

## Inputs

- exogenous-m6A-labeled nanopore gDNA modBAM (Dorado basecalled, minimap2-mapped)
- modkit pileup output (bedMethyl / per-position m6A calls, compressed .bed.gz)
- params.minCov and params.perMod thresholds for the BED filtering step

## Outputs

- per-genome consolidated open-chromatin interval set as compressed BED (open_chromatin.bed.gz)
- per-genome BedGraph tracks for genome-browser visualization of open-chromatin signal

## How to apply

Load the m6A-labeled gDNA modBAM produced by the upstream Dogme stage (Dorado basecalling merged and mapped with minimap2). Run modkit (v0.5 or higher) pileup on the modBAM to extract per-reference-position m6A modification calls, then filter the pileup to retain high-confidence modified positions using Dogme's scripts/filterbed.py with the params.minCov and params.perMod thresholds (reading and writing compressed .bed.gz). Consolidate the filtered per-chromosome calls into one per-genome open-chromatin interval set, processing one chromosome at a time to keep the memory footprint low, mirroring Dogme's 2-pass per-chromosome annotation strategy. Finally, publish the consolidated intervals as open_chromatin.bed.gz plus a BedGraph track per genome for browser visualization.

## Related tools

- **modkit** (computes pileups of m6A modification signals from the gDNA modBAM (v0.5+ required for open chromatin support)) — https://github.com/nanoporetech/modkit
- **Dogme** (Nextflow pipeline orchestrating the open-chromatin workflow, including filterbed.py threshold filtering and per-chromosome consolidation into BED/BedGraph outputs) — https://github.com/mortazavilab/dogme
- **dorado** (upstream basecaller producing the modBAM with m6A calls that feeds the open-chromatin module) — https://github.com/nanoporetech/dorado
- **minimap2** (maps Dorado basecalled reads to the reference genome to create the mapped BAM used for pileup)

## Evaluation signals

- open_chromatin.bed.gz exists and is a valid compressed BED file.
- The BED contains per-genome consolidated m6A open-chromatin intervals derived from the modkit pileup (intervals span multiple chromosomes, not a single contig).
- Intervals pass the filterbed.py thresholds (params.minCov minimum coverage and params.perMod per-modification fraction).
- A corresponding per-genome BedGraph track is produced alongside the BED file.
- Per-chromosome processing did not collapse positions: modkit-pileup positions retained in the BED should correspond to filtered high-confidence m6A-modified reference positions in the modBAM.

## Limitations

- Requires modkit v0.5 or higher; older modkit versions lack the open-chromatin workflow.
- Consolidation quality depends on upstream basecalling with m6A-capable Dorado models; Dogme only downloads models if absent, so stale model directories could degrade calls.
- Open chromatin support in Dogme 1.4.0 applies to mapped BAMs with m6A modifications (gDNA read type), not RNA/cDNA workflows.

## Evidence

- [readme] open chromatin workflow entry: [evidence span withheld — anchor status: external_doc]
- [readme] filterbed thresholds: [evidence span withheld — anchor status: external_doc]
- [intro] modkit extraction and filtering: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [discussion] per-chromosome memory strategy: "Reduced annotation memory use by processing one chromosome at a time."
