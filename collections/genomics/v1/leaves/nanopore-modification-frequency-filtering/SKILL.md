---
name: nanopore-modification-frequency-filtering
description: Use when you have an aligned nanopore modBAM with MM/ML base-modification
  tags from direct RNA, cDNA, or genomic DNA reads and need high-confidence per-site
  modification calls filtered by coverage and modification frequency.
when_to_use_negative:
- The input is not a modBAM with MM/ML base-modification tags — modkit pileup cannot
  extract modification frequencies without these tags
- You need transcript isoform quantification rather than modification site detection
  — for RNA and cDNA, Dogme runs kallisto on an extracted FASTQ instead
- Coverage is uniformly below the minimum threshold (fewer than 3 reads per site)
  — the filter will discard nearly all sites and the output will be uninformative
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: modkit
  role: Runs modkit pileup on the aligned modBAM to extract per-site modification
    counts and frequencies from MM/ML tags
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Pipeline that applies the custom modification filter (scripts/filterbed.py
    with params.minCov and params.perMod) to the modkit BED output
  repo: https://github.com/mortazavilab/dogme
- name: minimap2
  role: Upstream mapper producing the aligned BAM on which modkit modification extraction
    operates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:aa7b212237c735b5bb332b4c98f950fb2f8acf4de8a2d89c41f5866c6149115d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# nanopore-modification-frequency-filtering

## Summary

Extracts per-site RNA and DNA modification counts from nanopore modBAMs using modkit, then applies the Dogme custom filter — retaining only sites with at least 3 supporting reads and a modification frequency of at least 5% — to reduce false positives while preserving sensitivity.

## When to use

Use when you have an aligned nanopore modBAM with MM/ML base-modification tags from direct RNA, cDNA, or genomic DNA reads and need high-confidence per-site modification calls filtered by coverage and modification frequency.

## When NOT to use

- The input is not a modBAM with MM/ML base-modification tags — modkit pileup cannot extract modification frequencies without these tags
- You need transcript isoform quantification rather than modification site detection — for RNA and cDNA, Dogme runs kallisto on an extracted FASTQ instead
- Coverage is uniformly below the minimum threshold (fewer than 3 reads per site) — the filter will discard nearly all sites and the output will be uninformative

## Inputs

- Aligned modBAM with MM/ML base-modification tags (produced by Dorado basecalling and minimap2 mapping)
- Filtering thresholds: minimum coverage (params.minCov, e.g., 3 reads) and minimum per-modification frequency (params.perMod, e.g., 0.05)

## Outputs

- Filtered modification sites table (filtered_modifications.bed, bed-like format with read-depth and modification-frequency columns)
- Per-site modification counts and frequencies from modkit pileup (intermediate bedMethyl)

## How to apply

Load the aligned modBAM produced by the upstream minimap2 mapping step and run modkit pileup to extract per-site modification counts and frequencies from the MM/ML base-modification tags into a bedMethyl-like table. Then apply the Dogme custom modification filter (e.g., via scripts/filterbed.py, which uses params.minCov and params.perMod), retaining only sites with >=3 supporting reads and >=5% modification frequency. Write the retained sites to filtered_modifications.bed in tabular format with read-depth and frequency columns. The thresholds (min coverage 3, min modification fraction 0.05) are adjustable and are the key decision points balancing false-positive reduction against sensitivity.

## Related tools

- **modkit** (Runs modkit pileup on the aligned modBAM to extract per-site modification counts and frequencies from MM/ML tags) — https://github.com/nanoporetech/modkit
- **Dogme** (Pipeline that applies the custom modification filter (scripts/filterbed.py with params.minCov and params.perMod) to the modkit BED output) — https://github.com/mortazavilab/dogme
- **minimap2** (Upstream mapper producing the aligned BAM on which modkit modification extraction operates)

## Examples

```
modkit pileup aligned_modBAM.bam modifications.bed && python scripts/filterbed.py --minCov 3 --perMod 0.05 -o filtered_modifications.bed
```

## Evaluation signals

- Verify filtered_modifications.bed exists and contains read-count (coverage) and modification-frequency fields
- Confirm every retained row has read count >= 3
- Confirm every retained row has modification frequency >= 0.05
- Check that rows below threshold present in the raw modkit pileup output were correctly dropped
- Verify output is in bed-like tabular format with per-reference/modification columns (e.g., as consumed by report_bed_rep.py across replicates)

## Limitations

- Filtering thresholds are adjustable; overly strict coverage/frequency cutoffs reduce sensitivity, overly permissive ones inflate false positives
- Requires modkit 0.5 or higher for some extended workflows (e.g., open-chromatin signal from m6A-modified BAMs)
- The pipeline processes compressed .bed.gz files in the BED workflow, so uncompressed intermediates may need compression before filtering

## Evidence

- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] filterbed.py filters modkit bed outputs by coverage and per-mod thresholds: [evidence span withheld — anchor status: external_doc]
- [full_text] After Modkit extraction, Dogme applies an adjustable custom filter (>=3 reads, >=5% frequency).: [evidence span withheld — anchor status: not_found]
- [readme] Modkit converts modBAM to bedMethyl using MM and ML tags.: [evidence span withheld — anchor status: external_doc]
