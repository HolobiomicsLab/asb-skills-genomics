---
name: encode-dataset-acquisition-and-reprocessing
description: Use when raw ENCODE nanopore signal files (POD5/FAST5) must be downloaded
  and re-basecalled from scratch to reproduce published RNA modification site counts
  per modification type.
when_to_use_negative:
- Input is already basecalled/mapped (BAM) — use Dogme's remap, kallisto, or reports
  entry points instead of re-running basecalling.
- The data are cDNA or genomic DNA rather than direct RNA — Dogme treats these as
  different read types ('CDNA'/'DNA') with different processing branches.
- The goal is transcript quantification only, not modification site counting.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline that orchestrates model download, basecalling, mapping,
    modification extraction and filtering for the ENCODE reprocessing run
  repo: https://github.com/mortazavilab/dogme
- name: Dorado
  role: Basecaller applied to the retrieved POD5 signal files; Dogme downloads the
    latest Dorado models automatically
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Maps the merged basecalled BAMs to GRCm39 using the direct-RNA spliced preset
- name: modkit
  role: Extracts m6A, m5C, inosine, pseudouridine and Nm modifications from the aligned
    BAMs into bedMethyl/bed outputs that Dogme then filters
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:d4d96f854ea57170e4b3b4cf5b3b714a9f29d95beb69c85ba87001cf5d3f7a97
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# encode-dataset-acquisition-and-reprocessing

## Summary

Reproduces published RNA modification site counts by retrieving raw ENCODE nanopore signal data (POD5 from ENCSR160HKZ, three mouse C2C12 myoblast direct-RNA MinION replicates) and reprocessing it end-to-end with the Dogme Nextflow pipeline (Dorado basecalling, minimap2 mapping, modkit modification extraction).

## When to use

Use when raw ENCODE nanopore signal files (POD5/FAST5) must be downloaded and re-basecalled from scratch to reproduce published RNA modification site counts per modification type.

## When NOT to use

- Input is already basecalled/mapped (BAM) — use Dogme's remap, kallisto, or reports entry points instead of re-running basecalling.
- The data are cDNA or genomic DNA rather than direct RNA — Dogme treats these as different read types ('CDNA'/'DNA') with different processing branches.
- The goal is transcript quantification only, not modification site counting.

## Inputs

- ENCODE ENCSR160HKZ POD5 signal files (three C2C12 dRNA MinION replicates)
- Mouse GRCm39 reference genome FASTA
- GENCODE v36 GTF annotation
- Dogme Nextflow config file (pod5 directory, work dir, genome_annot_refs, readType='RNA')

## Outputs

- Dorado basecalled BAMs (merged per replicate)
- minimap2-aligned direct-RNA BAMs against GRCm39/GENCODE v36
- modkit-derived filtered modification BED files (m6A, m5C, inosine, pseudouridine, Nm)
- modification_site_counts.csv with per-sample average counts per modification type

## How to apply

Retrieve the three C2C12 dRNA replicate POD5 signal files for ENCODE accession ENCSR160HKZ, then run Dogme (Nextflow, readType='RNA') letting it download the latest Dorado models and basecall the POD5s; Dogme then merges the per-replicate BAMs and maps them with minimap2 using the direct-RNA spliced preset against the GRCm39 reference with GENCODE v36 annotation (with junction BED auto-converted from the GTF and --splice-max 500000). Extract modifications (m6A, m5C, inosine, pseudouridine, Nm) with modkit and apply Dogme's filtering step, implemented by scripts/filterbed.py, which filters the modkit bed outputs by minimum coverage and per-mod thresholds (params.minCov and params.perMod). Aggregate detected sites per replicate and compute per-sample averages; validate against the target averages of 147 915 m6A, 86 723 m5C, 21 256 inosine, 24 545 pseudouridine, and 83 207 Nm, writing modification_site_counts.csv.

## Related tools

- **Dogme** (Nextflow pipeline that orchestrates model download, basecalling, mapping, modification extraction and filtering for the ENCODE reprocessing run) — https://github.com/mortazavilab/dogme
- **Dorado** (Basecaller applied to the retrieved POD5 signal files; Dogme downloads the latest Dorado models automatically) — https://github.com/nanoporetech/dorado
- **minimap2** (Maps the merged basecalled BAMs to GRCm39 using the direct-RNA spliced preset)
- **modkit** (Extracts m6A, m5C, inosine, pseudouridine and Nm modifications from the aligned BAMs into bedMethyl/bed outputs that Dogme then filters) — https://github.com/nanoporetech/modkit

## Examples

```
nextflow run mortazavilab/dogme -c dogme.config --readType RNA --pod5_dir ENCSR160HKZ_pod5 --genome GRCm39.fa --gtf gencode.v36.annotation.gtf
```

## Evaluation signals

- modification_site_counts.csv contains all five modification types with the expected per-sample averages: ~147 915 m6A, ~86 723 m5C, ~21 256 inosine, ~24 545 pseudouridine, ~83 207 Nm
- Three separate replicate outputs are produced and aggregated (one per ENCSR160HKZ replicate), with counts computed as per-sample averages rather than a single pooled total
- Filtered modkit BED outputs exist only after scripts/filterbed.py has applied params.minCov and params.perMod thresholds
- Aligned BAMs map to GRCm39 with GENCODE v36 transcript annotation (junction BED generated from the GTF before minimap2)
- Reproducible modification profiles across replicates, consistent with the pipeline's reported reproducibility claim

## Limitations

- Published counts are version-dependent: the paper reported 96 603 m6A, 43 476 m5C, 8 829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates, whereas the task target cites per-sample averages (147 915 m6A, etc.), so agreement depends on Dogme/Dorado/modkit versions and filter thresholds (README-tested versions: dorado 1.2, minimap2 2.28, mod_kit 0.5).
- Dorado model download is only run if the model directory does not already exist, so stale cached models can silently change basecalling results.
- No specific Dorado model versions or checksums are recorded in the article, so exact reproducibility of basecalling is not guaranteed.

## Evidence

- [intro] Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2.: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [abstract] three separate mouse C2C12 myoblast replicates using direct RNA sequencing on MinION flow cells: "three separate mouse C2C12 myoblast replicates using direct RNA sequencing on MinION flow cells"
- [abstract] We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [readme] Filters modkit bed outputs by minimum coverage and per-mod thresholds (params.minCov and params.perMod): [evidence span withheld — anchor status: external_doc]
- [readme] You must first install dorado, minimap2, samtools, modkit, kallisto, and bustools, and the latest version of nextflow: [evidence span withheld — anchor status: external_doc]
