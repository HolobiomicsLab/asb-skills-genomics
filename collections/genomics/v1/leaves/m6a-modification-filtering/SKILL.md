---
name: m6a-modification-filtering
description: Use when nanopore POD5 data has been basecalled (Dorado) and mapped (minimap2)
  into a modBAM carrying m6A MM/ML tags, and per-position modification calls must
  be extracted, filtered, and consolidated into genome-wide accessibility intervals.
when_to_use_negative:
- Input reads are RNA or cDNA rather than genomic DNA — Dogme routes those to LR-Kallisto
  transcript quantification instead of the open-chromatin workflow
- The modBAM lacks m6A MM/ML modification tags (e.g., unmodified basecalls), so no
  m6A positions exist to filter
- modkit version is below 0.5, which Dogme requires for the open-chromatin entry point
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3168
tools:
- name: Dogme
  role: Nextflow pipeline orchestrating the gDNA modification-extraction, filtering
    (filterbed.py), and per-genome BED/BedGraph consolidation steps
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Piles up m6A modification signals from the modBAM into bed outputs; v0.5+
    powers Dogme's open-chromatin workflow
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Upstream basecaller producing the modified-base (MM/ML) modBAM that this skill
    filters
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Upstream mapper that aligns Dorado basecalls to the reference genome before
    modification filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:513010af9d324c2dd68422fd675a34b7e6e1e64263ae23246ab5a6b784f26041
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# m6a-modification-filtering

## Summary

Extracts and filters m6A modification calls from nanopore modBAM files using modkit (v0.5+), then consolidates high-confidence modified positions — for Dogme gDNA runs these m6A marks serve as open-chromatin accessibility calls exported as BED and BedGraph tracks.

## When to use

Use when nanopore POD5 data has been basecalled (Dorado) and mapped (minimap2) into a modBAM carrying m6A MM/ML tags, and per-position modification calls must be extracted, filtered, and consolidated into genome-wide accessibility intervals.

## When NOT to use

- Input reads are RNA or cDNA rather than genomic DNA — Dogme routes those to LR-Kallisto transcript quantification instead of the open-chromatin workflow
- The modBAM lacks m6A MM/ML modification tags (e.g., unmodified basecalls), so no m6A positions exist to filter
- modkit version is below 0.5, which Dogme requires for the open-chromatin entry point

## Inputs

- m6A-modified gDNA modBAM from Dogme Dorado basecalling and minimap2 mapping
- Dogme config params.minCov and params.perMod filter thresholds
- Per-chromosome reference intervals (processed one chromosome at a time)

## Outputs

- Filtered, consolidated per-genome open-chromatin interval set as compressed BED (open_chromatin.bed.gz)
- Per-genome BedGraph tracks for genome-browser visualization
- Intermediary modkit pileup bedMethyl/bed outputs (compressed .bed.gz)

## How to apply

Start from the exogenous-m6A-labeled gDNA modBAM produced by the Dogme basecalling/alignment stage (Dorado basecalling, minimap2 mapping). Run modkit pileup (v0.5 or higher) on the modBAM to extract m6A modification calls per reference position. Filter the resulting bed with Dogme's filterbed.py, which applies the params.minCov and params.perMod thresholds to retain only high-confidence modified positions, reading and writing compressed .bed.gz files. Consolidate per-chromosome intervals into a single per-genome interval set, processing one chromosome at a time to limit memory use — the same memory-saving pattern Dogme uses for annotation. Export the result as open_chromatin.bed.gz plus per-genome BedGraph tracks for genome-browser visualization.

## Related tools

- **Dogme** (Nextflow pipeline orchestrating the gDNA modification-extraction, filtering (filterbed.py), and per-genome BED/BedGraph consolidation steps) — https://github.com/mortazavilab/dogme
- **modkit** (Piles up m6A modification signals from the modBAM into bed outputs; v0.5+ powers Dogme's open-chromatin workflow) — https://github.com/nanoporetech/modkit
- **Dorado** (Upstream basecaller producing the modified-base (MM/ML) modBAM that this skill filters) — https://github.com/nanoporetech/dorado
- **minimap2** (Upstream mapper that aligns Dorado basecalls to the reference genome before modification filtering)

## Examples

```
modkit pileup sample.modbam.bam open_chromatin.bed.gz --mod m6A && python scripts/filterbed.py --input open_chromatin.bed.gz --output open_chromatin.filtered.bed.gz
```

## Evaluation signals

- open_chromatin.bed.gz exists and is a valid compressed BED file
- Output BED contains the expected per-genome consolidated m6A open-chromatin intervals derived from the modkit pileup
- All retained positions pass the configured params.minCov and params.perMod thresholds applied by filterbed.py
- BedGraph tracks cover the same per-genome intervals as the BED file and are viewable in a genome browser
- Memory footprint stays bounded because chromosomes are processed one at a time

## Limitations

- Requires modkit 0.5 or higher; older modkit versions cannot run the open-chromatin workflow
- Open-chromatin calling depends on exogenous m6A labeling of genomic DNA — unlabeled gDNA will not yield accessibility signal
- Dorado model versions and download behavior affect the modBAM input; Dogme only downloads models when the model directory does not already exist

## Evidence

- [intro] modkit extracts and filters modifications: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] filterbed.py thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] open chromatin workflow: [evidence span withheld — anchor status: external_doc]
- [discussion] per-chromosome memory reduction: "Reduced annotation memory use by processing one chromosome at a time."
