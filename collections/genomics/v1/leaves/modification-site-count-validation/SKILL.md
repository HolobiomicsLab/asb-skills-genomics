---
name: modification-site-count-validation
description: Use when comparing per-replicate modkit BED modification call sets to
  quantify and validate three-way (all-replicate) shared site counts and reproducibility
  percentages, e.g. against Dogme's C2C12 dRNA reference values.
when_to_use_negative:
- Input is a single-replicate modification call set with no replicate folders to intersect
  — reproducibility counts require at least two, and here exactly three, per-sample
  modkit BED outputs
- Data are cDNA or genomic DNA reads rather than direct RNA (RNA) modification beds
  from the Dogme RNA workflow
- You need de novo modification detection from raw POD5 files — this skill starts
  from already basecalled, mapped, and modkit-extracted/filtered BED outputs
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline that produced the per-replicate modification BEDs being
    compared; provides the filterbed.py and report_bed_rep.py helper scripts used
    for filtering and replicate comparison
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Extracts and filters the modification calls from modBAM into the BED outputs
    that feed the replicate intersection and counting
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Mapped the Dorado-basecalled direct RNA reads so that modification sites share
    a consistent genomic coordinate system for intersection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:d887dd7f697905f55eaa93638e44c9b80c13b4f425028d9987acd88066af89a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# modification-site-count-validation

## Summary

Counts and validates nanopore direct RNA modification sites shared across the three mouse C2C12 myoblast (ENCODE ENCSR160HKZ) dRNA replicates processed by Dogme (Dorado basecalling, minimap2 mapping, modkit extraction/filtering). It checks the reported reproducibility figures per modification type (m6A, m5C, inosine, pseudouridine, Nm).

## When to use

Use when comparing per-replicate modkit BED modification call sets to quantify and validate three-way (all-replicate) shared site counts and reproducibility percentages, e.g. against Dogme's C2C12 dRNA reference values.

## When NOT to use

- Input is a single-replicate modification call set with no replicate folders to intersect — reproducibility counts require at least two, and here exactly three, per-sample modkit BED outputs
- Data are cDNA or genomic DNA reads rather than direct RNA (RNA) modification beds from the Dogme RNA workflow
- You need de novo modification detection from raw POD5 files — this skill starts from already basecalled, mapped, and modkit-extracted/filtered BED outputs

## Inputs

- Per-replicate filtered modkit BED / bedMethyl (.bed.gz) modification call sets from the three ENCSR160HKZ C2C12 dRNA replicates
- Genome annotation (GTF) for mapping shared sites to genes
- Reference site counts per modification type for validation

## Outputs

- Per-modification counts of sites shared across all three replicates (expected: 96603 m6A, 43476 m5C, 8829 inosine, 10055 pseudouridine, 30320 Nm)
- Reproducibility fraction for m6A (shared/average per-replicate total = 63.2%)
- Per-reference/modification comparison CSVs (e.g. Compare_GRCh38_m6A.csv)
- Per-modification gene coverage statistics (m6A: 12179 genes, 88.2%; m5C: 8547 genes, 74.1%)

## How to apply

Load the per-replicate filtered modkit BED outputs produced by Dogme (Dorado basecalling, minimap2 mapping, modkit extraction) for the three ENCSR160HKZ biological replicates. For each modification type (m6A, m5C, inosine, pseudouridine, Nm), intersect the genomic site coordinates across all three replicates to retain only strictly shared sites, using scripts/report_bed_rep.py to merge plus/minus strands per sample and produce per-reference/modification comparison CSVs (e.g. Compare_GRCh38_m6A.csv). Count the shared sites per modification and, for m6A, also compute the shared count as a fraction of the average per-replicate total to recover the reported 63.2% reproducibility figure. Optionally map shared sites to gene annotations to check per-modification gene coverage (reported as 12179 genes, 88.2% for m6A and 8547 genes, 74.1% for m5C). Validate by comparing computed counts against the reference integers 96603 m6A, 43476 m5C, 8829 inosine, 10055 pseudouridine, and 30320 Nm; output must match within exact integer counts to pass.

## Related tools

- **Dogme** (Nextflow pipeline that produced the per-replicate modification BEDs being compared; provides the filterbed.py and report_bed_rep.py helper scripts used for filtering and replicate comparison) — https://github.com/mortazavilab/dogme
- **modkit** (Extracts and filters the modification calls from modBAM into the BED outputs that feed the replicate intersection and counting) — https://github.com/nanoporetech/modkit
- **minimap2** (Mapped the Dorado-basecalled direct RNA reads so that modification sites share a consistent genomic coordinate system for intersection)

## Examples

```
python scripts/report_bed_rep.py  # compare per-sample modkit BED (.bed.gz) outputs across replicate folders, merging strands and emitting per-reference/modification comparison CSVs
```

## Evaluation signals

- Strict three-way intersection counts match the reference integers exactly: 96603 m6A, 43476 m5C, 8829 inosine, 10055 pseudouridine, 30320 Nm
- Computed m6A reproducibility fraction equals 63.2% of the average per-replicate total
- Gene coverage of shared sites recovers the reported values: 12179 genes / 88.2% for m6A and 8547 genes / 74.1% for m5C
- Comparison CSVs are emitted per reference and modification (e.g. Compare_GRCh38_m6A.csv) with plus/minus strands merged per sample
- Filtering thresholds params.minCov and params.perMod are consistently applied before intersection so replicate call sets are comparable

## Limitations

- Counts depend on the exact Dorado model, modkit filtering thresholds (params.minCov, params.perMod), and minimap2 settings; changing any of these will shift site counts away from the reference values
- Strict three-way intersection penalizes single-replicate dropouts, so shared counts underestimate true biological conservation
- The article's discussion lacks recorded dataset accessions and version-tagged repository releases, so exact software versions used for the reference counts are not fully traceable

## Evidence

- [abstract] Detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [abstract] Reproducible modification profiles across replicates: "The pipeline produced reproducible modification profiles and transcript expression levels across replicates"
- [intro] modkit extraction and filtering of modifications: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] report_bed_rep.py replicate comparison script: [evidence span withheld — anchor status: external_doc]
- [readme] filterbed.py threshold parameters: [evidence span withheld — anchor status: external_doc]
