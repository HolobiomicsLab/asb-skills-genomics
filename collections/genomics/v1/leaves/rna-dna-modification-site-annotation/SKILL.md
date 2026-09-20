---
name: rna-dna-modification-site-annotation
description: Use when you have an aligned nanopore modBAM (from Dorado basecalling
  + minimap2 mapping) with MM/ML base-modification tags and need per-site modification
  calls filtered to reliable positions.
when_to_use_negative:
- Input reads have not been basecalled with modifications or mapped — the stage requires
  an aligned modBAM from Dorado/minimap2 upstream
- The goal is transcript isoform quantification (RNA/cDNA); Dogme uses kallisto for
  that, not the modkit/filterbed modification path
- Data are single-cell cDNA workflows where modification calling is not the target
  output
edam_operation: http://edamontology.org/operation_0306
tools:
- name: modkit
  role: Runs pileup on the aligned modBAM to create bedMethyl summary counts of modified
    and unmodified bases from MM/ML tags
  repo: https://github.com/nanoporetech/modkit
- name: filterbed.py (Dogme script)
  role: Filters modkit bed outputs by minimum coverage (params.minCov) and per-modification
    thresholds (params.perMod), reading/writing compressed .bed.gz files
  repo: https://github.com/mortazavilab/dogme
- name: Dogme
  role: Nextflow pipeline that orchestrates modkit extraction and custom modification
    filtering for RNA and DNA read types
  repo: https://github.com/mortazavilab/dogme
- name: minimap2
  role: Upstream mapper producing the aligned BAM on which modification extraction
    is run
  repo: https://github.com/mortazavilab/dogme
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:11ed7b0c1dbf5a0bfe7dba5eb2e9fc6eae4996324f5770cfb7615cadb5d68770
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - filterbed.py (Dogme script)
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# rna-dna-modification-site-annotation

## Summary

Extract RNA and DNA base-modification sites from aligned nanopore modBAMs using modkit, then apply the Dogme custom coverage/frequency filter to retain high-confidence sites. This is the modification-detection stage of the Dogme Nextflow pipeline for direct RNA, cDNA, and genomic DNA ONT data.

## When to use

Use when you have an aligned nanopore modBAM (from Dorado basecalling + minimap2 mapping) with MM/ML base-modification tags and need per-site modification calls filtered to reliable positions.

## When NOT to use

- Input reads have not been basecalled with modifications or mapped — the stage requires an aligned modBAM from Dorado/minimap2 upstream
- The goal is transcript isoform quantification (RNA/cDNA); Dogme uses kallisto for that, not the modkit/filterbed modification path
- Data are single-cell cDNA workflows where modification calling is not the target output

## Inputs

- Aligned modBAM from minimap2 mapping with MM/ML base-modification tags (produced by Dogme's Dorado basecalling and merge/map steps)
- Dogme Nextflow config defining params.minCov (minimum read coverage) and params.perMod (minimum modification frequency)

## Outputs

- Filtered modification sites in bed-like .bed.gz (bedMethyl-style) tabular format with per-site read count and modification frequency columns
- Per-reference/per-modification comparison tables across replicates (e.g., Compare_GRCh38_m6A.csv via report_bed_rep.py)

## How to apply

Load the aligned modBAM produced by the upstream Dorado/minimap2 stage of Dogme and run modkit pileup to convert the MM/ML base-modification tags into per-site counts of modified and unmodified bases in the extended bedMethyl format. Then apply the Dogme custom filtering step, implemented in scripts/filterbed.py, which filters the modkit BED outputs by a minimum coverage threshold and a per-modification threshold set via params.minCov and params.perMod (in the pipeline these encode at least 3 supporting reads and at least 5% modification frequency). Retain only sites passing both thresholds, writing filtered modification sites as compressed .bed.gz with read-depth and frequency fields. This dual threshold reduces false-positive modification calls while retaining sensitivity, and was applied to call m6A, m5C, inosine, pseudouridine, and Nm sites across three mouse C2C12 direct RNA replicates.

## Related tools

- **modkit** (Runs pileup on the aligned modBAM to create bedMethyl summary counts of modified and unmodified bases from MM/ML tags) — https://github.com/nanoporetech/modkit
- **filterbed.py (Dogme script)** (Filters modkit bed outputs by minimum coverage (params.minCov) and per-modification thresholds (params.perMod), reading/writing compressed .bed.gz files) — https://github.com/mortazavilab/dogme
- **Dogme** (Nextflow pipeline that orchestrates modkit extraction and custom modification filtering for RNA and DNA read types) — https://github.com/mortazavilab/dogme
- **minimap2** (Upstream mapper producing the aligned BAM on which modification extraction is run) — https://github.com/mortazavilab/dogme

## Examples

```
modkit pileup aligned.modbam.sorted.bam modifications.bedMethyl.gz && python scripts/filterbed.py  # filter by params.minCov / params.perMod
```

## Evaluation signals

- Output file exists and is a valid compressed bedMethyl-like .bed.gz with read-count (coverage) and modification-frequency fields
- Every retained row has read coverage >= 3 (params.minCov) and modification frequency >= 0.05 (params.perMod)
- Sites in the filtered output are a subset of the unfiltered modkit pileup sites; no modification category gains sites after filtering
- Detected modification profiles are reproducible across the three biological replicates (compare via report_bed_rep.py CSVs)
- Software versions recorded via scripts/software_versions.py include modkit version (e.g., mod_kit 0.5)

## Limitations

- Filter thresholds (params.minCov, params.perMod) are adjustable; overly strict settings may remove true low-frequency modification sites
- The open-chromatin modkit feature requires modkit 0.5 or higher
- Multi-genome/multi-strand runs require correct channel grouping and file naming; per-chromosome processing was adopted to reduce annotation memory use

## Evidence

- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] filterbed.py thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] modkit bedMethyl counts: [evidence span withheld — anchor status: external_doc]
- [readme] MM/ML tags source: [evidence span withheld — anchor status: external_doc]
- [abstract] Detection scale in mouse direct RNA replicates: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
