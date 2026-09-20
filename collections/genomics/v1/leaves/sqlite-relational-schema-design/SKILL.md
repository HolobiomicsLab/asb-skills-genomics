---
name: sqlite-relational-schema-design
description: Use when you must consolidate per-sample SQANTI3 classification outputs
  whose PB.X.Y isoform IDs cannot be matched across samples, and need an updatable
  relational store that tracks isoforms by intron chain with per-sample and per-end
  counts.
when_to_use_negative:
- You only have a single sample and no need to match isoform IDs across runs — a plain
  SQANTI3 classification file suffices
- Isoforms must be distinguished by full-length coordinates rather than intron chain
  — this schema deliberately ignores start/end coordinates when defining isoform identity
- Input data was not processed through SQANTI3 (e.g., short-read counts or de novo
  transcriptomes without junction annotations), since ID assignment depends on intron
  chains from SQANTI3 output
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: 'Implements the schema design: builds the updatable isoseql.db from SQANTI3
    outputs, assigning intron-chain isoform IDs and populating isoform/counts/ends
    tables'
  repo: https://github.com/christine-liu/isoSeQL
- name: SQANTI3
  role: Produces the RulesFilter-filtered classification.txt files that serve as the
    per-sample input records for the database
  repo: https://github.com/ConesaLab/SQANTI3
- name: Python 3.9.6
  role: Language in which isoSeQL was developed; classification.txt files are loaded
    with pandas during database population
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:3ee3a67a75dc65d1bc8fb47294d1e4a5cae3bf11587a5684c21579da94db5b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - Python 3.9.6
  - SQANTI3
  - isoSeQL
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# sqlite-relational-schema-design

## Summary

Design a cross-sample relational SQLite schema (isoseql.db) for long-read Iso-Seq data in which isoforms are identified by intron chain with a stable common-junction ID, and start/end coordinate variation, per-sample counts, sample metadata, and experiment provenance are stored in linked tables so the database can be updated with new samples over time.

## When to use

Use when you must consolidate per-sample SQANTI3 classification outputs whose PB.X.Y isoform IDs cannot be matched across samples, and need an updatable relational store that tracks isoforms by intron chain with per-sample and per-end counts.

## When NOT to use

- You only have a single sample and no need to match isoform IDs across runs — a plain SQANTI3 classification file suffices
- Isoforms must be distinguished by full-length coordinates rather than intron chain — this schema deliberately ignores start/end coordinates when defining isoform identity
- Input data was not processed through SQANTI3 (e.g., short-read counts or de novo transcriptomes without junction annotations), since ID assignment depends on intron chains from SQANTI3 output

## Inputs

- SQANTI3 RulesFilter-filtered classification.txt files, one per sample
- Per-sample read counts from the SQANTI3 output
- Sample metadata (cell line/tissue source material)
- Software versions, reference versions, and experiment date
- Optional: cellular barcode and cell-type information for single-cell inputs

## Outputs

- Updatable SQLite database (isoseql.db)
- isoform table keyed by common-junction isoform ID with intron-chain structure records
- counts table with per-sample read counts
- isoform_ends table of distinct start/end coordinate combinations
- ends_counts table with counts per coordinate combination
- sampleData and exp provenance tables

## How to apply

Load each sample's SQANTI3 RulesFilter-filtered classification.txt file with pandas, then reduce each transcript model to its intron chain and assign a stable common-junction isoform ID so that isoforms with identical junctions across samples share one ID, regardless of start/end coordinates (this resolves the unmatched PB.X.Y ID problem). Insert isoform structure records and per-sample read counts into isoform and counts tables; record each distinct start/end coordinate combination in an isoform_ends table with its counts in an ends_counts table, decoupling end-site variation from the junction-defined isoform identity. Populate a sampleData table with cell line/tissue source-material information and an exp table with software versions, reference versions, and experiment date; for single-cell inputs, add cellular barcode and cell-type tables. Write all tables into a single SQLite database (isoseql.db) so additional samples can be appended on subsequent runs rather than rebuilt from scratch.

## Related tools

- **isoSeQL** (Implements the schema design: builds the updatable isoseql.db from SQANTI3 outputs, assigning intron-chain isoform IDs and populating isoform/counts/ends tables) — https://github.com/christine-liu/isoSeQL
- **SQANTI3** (Produces the RulesFilter-filtered classification.txt files that serve as the per-sample input records for the database) — https://github.com/ConesaLab/SQANTI3
- **Python 3.9.6** (Language in which isoSeQL was developed; classification.txt files are loaded with pandas during database population)

## Examples

```
python /path/to/isoSeQL/populate_db.py --classification outdir/sample_isoSeqccs_demux_flnc_cluster_mapped_collapsed_classification.txt --db isoseql.db
```

## Evaluation signals

- Isoforms sharing the same intron chain across samples resolve to a single stable common-junction ID (no unmatched PB.X.Y IDs across samples)
- Per-sample counts in the counts table are consistent with the input classification.txt files
- Counts in the ends_counts table match the per-end counts in the input classification files
- Re-running the tool with an additional sample appends records to the existing isoseql.db rather than overwriting prior samples
- Every distinct start/end coordinate combination observed appears as a row in isoform_ends linked to the correct intron-chain isoform ID

## Limitations

- Isoform identity is defined solely by intron chain, so isoforms with identical junctions but different transcription start/end sites share one ID; end-site detail is only recoverable via the isoform_ends table
- The workflow assumes input has passed SQANTI3 RulesFilter filtering; upstream PB.X.Y IDs from unfiltered or differently processed runs will not match
- Cross-sample comparability depends on consistent annotation upstream (isoseq > minimap2/pbmm2 > collapse > SQANTI3 pipeline)

## Evidence

- [other] isoSeQL defines isoforms by their intron chains and assigns them isoform ID numbers that are used to link entries across different SQLite tables.: [evidence span withheld — anchor status: not_found]
- [other] consolidating isoforms with identical junctions across samples regardless of start and end coordinate: [evidence span withheld — anchor status: not_found]
- [other] Write all tables into an updatable SQLite database (isoseql.db) so additional samples can be appended on subsequent runs.: [evidence span withheld — anchor status: not_found]
- [intro] the isoform ID #s (PB.X.Y) in each sample couldn't be matched: "I was working with multiple samples that I wanted to compare to each other, but quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched."
- [intro] isoSeQL was created to work with the output of SQANTI2 (now SQANTI3): "isoSeQL was created to work with the output of SQANTI2 (now SQANTI3)"
- [abstract] a tool for compiling long-read transcriptomic data: "a tool for compiling long-read transcriptomic data"
