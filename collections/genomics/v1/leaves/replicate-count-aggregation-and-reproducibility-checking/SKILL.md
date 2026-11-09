---
name: replicate-count-aggregation-and-reproducibility-checking
description: Use when several nanopore dRNA replicates have been processed through
  Dogme and you need per-replicate modkit BED counts merged across plus/minus strands,
  averaged per modification type, and checked against reported values.
when_to_use_negative:
- When you only have a single sample/run and no replicates to compare — the aggregation
  and cross-replicate reproducibility check has no basis
- When modification sites have not yet been extracted and filtered with modkit (raw
  modBAMs, unfiltered BEDs, or basecalled-only FASTQs are the wrong inputs)
- When the goal is transcript quantification (kallisto/LR-Kallisto outputs) rather
  than modification site counts
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline whose report_bed_rep.py script compares per-sample modkit
    BED outputs across replicate folders to produce comparison CSVs
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Produces the per-replicate bedMethyl/BED modification site outputs that feed
    the aggregation and comparison step
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Basecaller generating the modBAMs upstream; model version changes affect count
    reproducibility across replicates
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:f17ae2a6cea22fb63eff4b01148ae41cf19caf62a69f8a3ba423e8be4af6ac9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - Nextflow
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# replicate-count-aggregation-and-reproducibility-checking

## Summary

Aggregates per-replicate RNA modification site counts (m6A, m5C, inosine, pseudouridine, Nm) from Dogme/modkit BED outputs across ONT direct RNA replicates, computes per-sample averages, and compares them against reported targets to verify reproducibility of the pipeline's modification profiles.

## When to use

Use when several nanopore dRNA replicates have been processed through Dogme and you need per-replicate modkit BED counts merged across plus/minus strands, averaged per modification type, and checked against reported values.

## When NOT to use

- When you only have a single sample/run and no replicates to compare — the aggregation and cross-replicate reproducibility check has no basis
- When modification sites have not yet been extracted and filtered with modkit (raw modBAMs, unfiltered BEDs, or basecalled-only FASTQs are the wrong inputs)
- When the goal is transcript quantification (kallisto/LR-Kallisto outputs) rather than modification site counts

## Inputs

- Per-replicate filtered modkit BED outputs (.bed.gz) from Dogme's modification filtering step
- Sample folder structure grouping replicates (e.g., three C2C12 dRNA replicates from ENCODE ENCSR160HKZ)
- Reference genome/annotation identifiers used for comparison (e.g., GRCm39 with GENCODE v36)

## Outputs

- Per-reference/modification comparison CSVs (e.g., Compare_GRCh38_m6A.csv)
- Aggregated per-replicate modification site counts per modification type (m6A, m5C, inosine, pseudouridine, Nm)
- Per-sample average counts table written to modification_site_counts.csv

## How to apply

After Dogme (Dorado basecalling, minimap2 mapping, modkit extraction and filtering) has produced per-replicate filtered BED outputs, run the replicate comparison script over the per-sample BED folders; it reads compressed .bed.gz files, merges plus/minus strand sites per sample, and produces per-reference/modification comparison CSVs (e.g., Compare_GRCh38_m6A.csv). Aggregate detected sites per replicate and compute per-sample average counts for each of the five modification types. Judge correctness by comparing computed averages to the reported targets — for the C2C12 ENCODE ENCSR160HKZ replicates (GRCm39, GENCODE v36): 147915 m6A, 86723 m5C, 21256 inosine, 24545 pseudouridine, and 83207 Nm — and write the counts table to modification_site_counts.csv. Discrepancies indicate changes in basecalling models, filtering thresholds (params.minCov, params.perMod), or mapping that break reproducibility.

## Related tools

- **Dogme** (Nextflow pipeline whose report_bed_rep.py script compares per-sample modkit BED outputs across replicate folders to produce comparison CSVs) — https://github.com/mortazavilab/dogme
- **modkit** (Produces the per-replicate bedMethyl/BED modification site outputs that feed the aggregation and comparison step) — https://github.com/nanoporetech/modkit
- **Dorado** (Basecaller generating the modBAMs upstream; model version changes affect count reproducibility across replicates) — https://github.com/nanoporetech/dorado

## Examples

```
python scripts/report_bed_rep.py --folders rep1/ rep2/ rep3/ --out Compare_GRCh38_m6A.csv && nextflow run mortazavilab/dogme -entry reports
```

## Evaluation signals

- Per-sample average counts match the reported targets: 147915 m6A, 86723 m5C, 21256 inosine, 24545 pseudouridine, 83207 Nm for the three C2C12 ENCSR160HKZ replicates
- Counts table modification_site_counts.csv is produced with one averaged value per modification type across all replicates
- Per-reference/modification comparison CSVs (e.g., Compare_GRCh38_m6A.csv) are generated with entries for every replicate and modification
- Counts are computed from compressed .bed.gz files with plus/minus strand sites merged per sample
- The pipeline shows reproducible modification profiles across biological replicates (counts of similar magnitude across replicates, no replicate-specific dropout)

## Limitations

- Counts are sensitive to the Dorado model version used; the README notes dorado model download is skipped if the model directory already exists, so stale models can shift counts
- Filtering thresholds params.minCov and params.perMod (applied by scripts/filterbed.py) directly change the number of retained sites and must be identical across replicates
- The comparison script assumes per-sample BED folders and compressed .bed.gz inputs; uncompressed or differently organized outputs are not handled by the documented workflow

## Evidence

- [readme] Compares per-sample modkit BED outputs across folders (replicates), including compressed `.bed.gz` files, merges plus/minus strands per sample and produces per-reference/modification comparison CSVs: [evidence span withheld — anchor status: external_doc]
- [other] Reported C2C12 replicate average targets: [evidence span withheld — anchor status: not_found]
- [abstract] Reproducible profiles finding: "The pipeline produced reproducible modification profiles and transcript expression levels across replicates"
- [intro] Filtering step controlling counts: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] filterbed.py threshold parameters: [evidence span withheld — anchor status: external_doc]
