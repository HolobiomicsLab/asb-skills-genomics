---
name: modbam-modification-extraction
description: Use when you have an aligned modBAM from nanopore reads (direct RNA,
  cDNA, or genomic DNA basecalled with Dorado) and need per-site modification calls
  with coverage/frequency filtering to reduce false positives.
when_to_use_negative:
- Input is a plain BAM without MM/ML modification tags — modkit pileup cannot extract
  modification information from unmodified basecalls.
- The goal is transcript isoform quantification rather than modification calling —
  for RNA/cDNA reads Dogme instead extracts a FASTQ and runs LR-Kallisto.
- Input is raw POD5 signal — basecalling with Dorado must happen first; this skill
  starts from already-aligned modBAM.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: modkit
  role: Runs pileup on the modBAM to convert MM/ML base-modification tags into per-site
    bedMethyl counts and frequencies
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Nextflow pipeline that orchestrates modkit extraction and applies the custom
    modification filter (filterbed.py) with minCov/perMod thresholds
  repo: https://github.com/mortazavilab/dogme
- name: minimap2
  role: Upstream mapper that produces the aligned BAM/modBAM used as input to modkit
    pileup
- name: Dorado
  role: Basecaller that produces the modBAM with modified-base MM/ML tags from POD5
    files
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:4cc30ce2a80b3c504def08e2ac7b1198e16daaabbef7f2d7fd8874e550b080fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - Dorado
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# modBAM modification extraction and filtering

## Summary

Extracts per-site RNA and DNA modification counts and frequencies from aligned nanopore modBAMs using modkit pileup (MM/ML tags), then applies Dogme's custom filter retaining only sites with at least 3 supporting reads and at least 5% modification frequency.

## When to use

Use when you have an aligned modBAM from nanopore reads (direct RNA, cDNA, or genomic DNA basecalled with Dorado) and need per-site modification calls with coverage/frequency filtering to reduce false positives.

## When NOT to use

- Input is a plain BAM without MM/ML modification tags — modkit pileup cannot extract modification information from unmodified basecalls.
- The goal is transcript isoform quantification rather than modification calling — for RNA/cDNA reads Dogme instead extracts a FASTQ and runs LR-Kallisto.
- Input is raw POD5 signal — basecalling with Dorado must happen first; this skill starts from already-aligned modBAM.

## Inputs

- Aligned modBAM with MM/ML base-modification tags (from Dorado basecalling and minimap2 mapping)
- Filtering parameters params.minCov (minimum read coverage, e.g., 3) and params.perMod (minimum modification frequency, e.g., 0.05)

## Outputs

- modkit pileup bedMethyl table of per-site modification counts and frequencies
- filtered_modifications.bed (.bed.gz) with read-count and modification-frequency columns
- Per-replicate comparison CSVs (e.g., Compare_GRCh38_m6A.csv) via report_bed_rep.py

## How to apply

Load the aligned modBAM produced by the upstream minimap2 mapping step in the Dogme Nextflow pipeline. Run modkit pileup on the modBAM to summarize the MM/ML base-modification tags into per-site modified/unmodified base counts in an extended bedMethyl table. Then apply Dogme's custom filtering step (scripts/filterbed.py, driven by params.minCov and params.perMod), which by default retains sites with >=3 supporting reads and >=5% modification frequency (0.05) to suppress false positives while keeping sensitivity. Write the retained sites to filtered_modifications.bed in a bed-like tabular format that includes read-depth and modification-frequency columns; filterbed.py reads and writes compressed .bed.gz files in the BED workflow. For reproducible replicate comparisons, per-sample modkit BED outputs can be compared with report_bed_rep.py, merging plus/minus strands per sample.

## Related tools

- **modkit** (Runs pileup on the modBAM to convert MM/ML base-modification tags into per-site bedMethyl counts and frequencies) — https://github.com/nanoporetech/modkit
- **Dogme** (Nextflow pipeline that orchestrates modkit extraction and applies the custom modification filter (filterbed.py) with minCov/perMod thresholds) — https://github.com/mortazavilab/dogme
- **minimap2** (Upstream mapper that produces the aligned BAM/modBAM used as input to modkit pileup)
- **Dorado** (Basecaller that produces the modBAM with modified-base MM/ML tags from POD5 files) — https://github.com/nanoporetech/dorado

## Examples

```
modkit pileup aligned.mod.bam modifications.bed && python scripts/filterbed.py --input modifications.bed.gz --output filtered_modifications.bed.gz --minCov 3 --perMod 0.05
```

## Evaluation signals

- filtered_modifications.bed exists and contains read-count and modification-frequency fields.
- Every retained row has read count >=3 and modification frequency >=0.05.
- The filtered BED is a valid (possibly compressed) bedMethyl-style table readable by downstream scripts such as report_bed_rep.py.
- Modification profiles are reproducible across biological replicates (compare per-sample BED outputs, e.g., m6A/m5C/inosine/pseudouridine/Nm site counts).
- Software versions used in the run are recorded (e.g., mod_kit 0.5) in ${sample}.softwareVersion.txt.

## Limitations

- Filtering thresholds (>=3 reads, >=5% frequency) trade specificity for sensitivity; rare or low-coverage modification sites will be discarded.
- Open-chromatin calling from m6A-modified BAMs requires modkit 0.5 or higher.
- Per-sample replicate comparison and filtering scripts expect compressed .bed.gz inputs in the BED workflow.

## Evidence

- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [methods] Filter thresholds rationale from source card workflow.: [evidence span withheld — anchor status: not_found]
- [readme] filterbed.py description: [evidence span withheld — anchor status: external_doc]
- [readme] modkit purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] Detected modification site counts: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
