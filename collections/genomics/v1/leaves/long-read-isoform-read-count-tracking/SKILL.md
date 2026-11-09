---
name: long-read-isoform-read-count-tracking
description: Use when you have SQANTI3 classification.txt output for multiple Iso-Seq
  samples whose PB.X.Y isoform IDs cannot be matched across samples, and you need
  a cross-sample isoform/read-count database that can absorb new samples on later
  runs.
when_to_use_negative:
- You have only a single sample and no need to match isoform IDs across runs — per-sample
  SQANTI3 output is sufficient
- Input transcripts have not yet been annotated/filtered through SQANTI3 (e.g. raw
  cluster BAMs or cDNA_Cupcake collapse output) — run sqanti3_RulesFilter.py first
- Your goal is raw Iso-Seq read processing (ccs, lima, isoseq refine/cluster2) rather
  than cross-sample isoform comparison
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Builds and updates the SQLite database, assigns intron-chain isoform IDs,
    and stores per-sample counts
  repo: https://github.com/christine-liu/isoSeQL
- name: SQANTI3
  role: Produces the RulesFilter-filtered classification.txt files that isoSeQL ingests
    as input
  repo: https://github.com/ConesaLab/SQANTI3
- name: Python 3.9.6
  role: Implementation language; isoSeQL loads classification files with pandas to
    build the database
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:427b8e27e76d36e0671e21bc178f6985437612af829807e7a3c271c3707149d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - Python 3.9.6
  - SQANTI3
  - isoSeQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-isoform-read-count-tracking

## Summary

Populates an updatable SQLite database (isoseql.db) from per-sample SQANTI3 RulesFilter-filtered classification.txt files, defining isoforms by intron chain so cross-sample PB.X.Y IDs are consolidated into stable common-junction isoform IDs with a separate start/end coordinate (isoform_ends/ends_counts) table.

## When to use

Use when you have SQANTI3 classification.txt output for multiple Iso-Seq samples whose PB.X.Y isoform IDs cannot be matched across samples, and you need a cross-sample isoform/read-count database that can absorb new samples on later runs.

## When NOT to use

- You have only a single sample and no need to match isoform IDs across runs — per-sample SQANTI3 output is sufficient
- Input transcripts have not yet been annotated/filtered through SQANTI3 (e.g. raw cluster BAMs or cDNA_Cupcake collapse output) — run sqanti3_RulesFilter.py first
- Your goal is raw Iso-Seq read processing (ccs, lima, isoseq refine/cluster2) rather than cross-sample isoform comparison

## Inputs

- Per-sample SQANTI3 sqanti3_RulesFilter.py-filtered classification.txt files
- Cell line/tissue source-material metadata per sample
- Software version, reference version, and experiment date metadata (for the exp table)
- For single-cell runs: cellular barcode and cell-type information

## Outputs

- isoseql.db SQLite database with isoform, counts, isoform_ends, ends_counts, sampleData, and exp tables
- Common-junction isoform IDs (stable intron-chain keys) linking entries across tables
- Per-sample read counts consistent with input classification files
- Optional cellular barcode and cell-type tables for single-cell inputs

## How to apply

Load each sample's SQANTI3 RulesFilter-filtered classification.txt file with pandas, then reduce every transcript model to its intron chain and assign a stable common-junction isoform ID so that isoforms with identical junctions across samples consolidate to one ID regardless of start/end coordinates. Insert isoform structure records and per-sample read counts into the isoform and counts tables, and record each distinct start/end coordinate combination in the isoform_ends table with its counts in the ends_counts table. Populate the sampleData table with cell line/tissue source-material information and the exp table with software versions, reference versions, and experiment date; for single-cell inputs, additionally add the cellular barcode and cell-type tables. Write everything into isoseql.db so subsequent runs append additional samples rather than rebuilding. The rationale is that PB.X.Y IDs from SQANTI2/SQANTI3 differ between samples, so intron-chain identity is the only stable cross-sample key.

## Related tools

- **isoSeQL** (Builds and updates the SQLite database, assigns intron-chain isoform IDs, and stores per-sample counts) — https://github.com/christine-liu/isoSeQL
- **SQANTI3** (Produces the RulesFilter-filtered classification.txt files that isoSeQL ingests as input) — https://github.com/ConesaLab/SQANTI3
- **Python 3.9.6** (Implementation language; isoSeQL loads classification files with pandas to build the database)

## Evaluation signals

- Isoforms sharing the same intron chain across different samples resolve to a single stable isoform ID (fixing the unmatched PB.X.Y IDs problem)
- Per-sample read counts in the counts table are consistent with the input classification.txt files
- Each distinct start/end coordinate combination appears once in isoform_ends with matching counts in ends_counts
- sampleData table contains the correct cell line/tissue source-material info and exp table records software/reference versions and experiment date
- Re-running the tool with an additional sample appends new entries to the existing isoseql.db instead of overwriting it

## Limitations

- Requires input already processed through SQANTI3 (or SQANTI2) with the RulesFilter step; it does not cluster or align reads itself
- Isoform identity is defined only by intron chain, so transcripts with identical junctions but different TSS/polyA sites share an ID and are distinguished only in the isoform_ends table
- The older cDNA_Cupcake-based alternative does not scale well with increased sample numbers or SMRTcell throughput, which is the motivating limitation this database approach replaces

## Evidence

- [intro] isoSeQL was created to work with the output of SQANTI2 (now SQANTI3): "isoSeQL was created to work with the output of SQANTI2 (now SQANTI3)"
- [intro] PB.X.Y IDs cannot be matched across samples: "I was working with multiple samples that I wanted to compare to each other, but quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched."
- [methods] isoSeQL implemented in Python 3.9.6: "isoSeQL was developed using python 3.9.6"
- [abstract] isoSeQL identifies common and unique isoforms across samples: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples"
- [methods] RulesFilter step required upstream of isoSeQL: "python /path/to/SQANTI3/sqanti3_RulesFilter.py /outdir/sample_isoSeqccs_demux_flnc_cluster_mapped_collapsed_classification.txt"
