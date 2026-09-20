---
name: modkit-modification-site-counting
description: Use when Dorado-basecalled ONT modBAM files (direct RNA, cDNA, or DNA)
  need conversion to bedMethyl and per-modification site counts, with replicate-level
  aggregation like the C2C12 dRNA ENCSR160HKZ benchmark.
when_to_use_negative:
- Reads are standard nucleotide FASTQ/BAM without MM/ML modified-base tags — modkit's
  bedMethyl extraction requires modBAM tags produced by the basecaller
- The goal is transcript isoform quantification rather than modification site counting
  — use the Dogme kallisto/LR-Kallisto branch instead
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: modkit
  role: Extracts modifications from modBAM into bedMethyl files and provides summary
    statistics
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Orchestrates the modkit extraction and filtering steps and provides filterbed.py
    and report_bed_rep.py for thresholding and replicate comparison
  repo: https://github.com/mortazavilab/dogme
- name: Dorado
  role: Produces the modBAM input files by basecalling POD5 signal data with modified-base
    models
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Maps the basecalled reads (direct-RNA preset against GRCm39 with GENCODE v36)
    to create the aligned modBAMs modkit consumes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:27cfc51afbc9724268beeb89c12b219f33495b09218b2b2e317a565c5f3e5458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - Dorado
  - Nextflow
  - minimap2
  - modkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# modkit-modification-site-counting

## Summary

Extract and count RNA modification sites (m6A, m5C, inosine, pseudouridine, Nm) from Oxford Nanopore modBAMs by running modkit to produce bedMethyl files and filtering them with Dogme's filterbed.py using minimum coverage and per-mod thresholds, then aggregating per-modification site counts across replicates.

## When to use

Use when Dorado-basecalled ONT modBAM files (direct RNA, cDNA, or DNA) need conversion to bedMethyl and per-modification site counts, with replicate-level aggregation like the C2C12 dRNA ENCSR160HKZ benchmark.

## When NOT to use

- Reads are standard nucleotide FASTQ/BAM without MM/ML modified-base tags — modkit's bedMethyl extraction requires modBAM tags produced by the basecaller
- The goal is transcript isoform quantification rather than modification site counting — use the Dogme kallisto/LR-Kallisto branch instead

## Inputs

- Aligned modBAM files from Dorado basecalling and minimap2 mapping (merged across replicates)
- ONT POD5/FAST5 signal files (via the upstream Dogme basecalling step)
- Dogme config parameters params.minCov and params.perMod for the filtering step

## Outputs

- bedMethyl (.bed.gz) files of modification sites per modification type
- Filtered bedMethyl files produced by scripts/filterbed.py
- Per-reference/per-modification replicate comparison CSVs (e.g., Compare_GRCh38_m6A.csv) and a modification_site_counts.csv table of per-sample average counts

## How to apply

After basecalling POD5 files with Dorado and mapping with minimap2, run modkit on the merged BAMs to extract modifications into bedMethyl format. Apply Dogme's filtering step via scripts/filterbed.py, which filters the modkit BED outputs by minimum coverage (params.minCov) and per-modification probability thresholds (params.perMod), reading and writing compressed .bed.gz files. For replicate comparisons, use scripts/report_bed_rep.py to merge plus/minus strands per sample and produce per-reference/modification comparison CSVs (e.g., Compare_GRCh38_m6A.csv). Aggregate detected sites per replicate and compute per-sample averages per modification type; for the three C2C12 dRNA replicates the target averages are 147,915 m6A, 86,723 m5C, 21,256 inosine, 24,545 pseudouridine, and 83,207 Nm sites.

## Related tools

- **modkit** (Extracts modifications from modBAM into bedMethyl files and provides summary statistics) — https://github.com/nanoporetech/modkit
- **Dogme** (Orchestrates the modkit extraction and filtering steps and provides filterbed.py and report_bed_rep.py for thresholding and replicate comparison) — https://github.com/mortazavilab/dogme
- **Dorado** (Produces the modBAM input files by basecalling POD5 signal data with modified-base models) — https://github.com/nanoporetech/dorado
- **minimap2** (Maps the basecalled reads (direct-RNA preset against GRCm39 with GENCODE v36) to create the aligned modBAMs modkit consumes)

## Examples

```
python scripts/filterbed.py --input mods.bed.gz --output mods.filtered.bed.gz  # filters modkit bed output at params.minCov/params.perMod thresholds
```

## Evaluation signals

- Filtered .bed.gz outputs are valid compressed bedMethyl files readable by downstream Dogme reporting scripts
- Per-sample average site counts are reproducible across the three replicates and fall near the reported targets (147,915 m6A, 86,723 m5C, 21,256 inosine, 24,545 pseudouridine, 83,207 Nm for the C2C12 data)
- filterbed.py applied both thresholds (params.minCov and params.perMod), so counts differ from raw unfiltered modkit output
- report_bed_rep.py successfully merges plus/minus strand entries per sample before the per-modification comparison CSVs are produced
- The pipeline reports reproducible modification profiles across biological replicates

## Limitations

- Filtering by minimum coverage and per-mod probability thresholds reduces site counts relative to raw modkit output, so counts are threshold-dependent
- No specific Dorado model versions or checksums are recorded, and automatic Dorado model downloads may vary over time, affecting reproducibility
- Annotation memory can be high; Dogme mitigates this by processing one chromosome at a time

## Evidence

- [intro] modkit extraction and filtering in Dogme: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] filterbed.py behavior: [evidence span withheld — anchor status: external_doc]
- [readme] modkit purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] reported site counts in three replicates: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [abstract] reproducibility across replicates: "The pipeline produced reproducible modification profiles and transcript expression levels across replicates"
