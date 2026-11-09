---
name: rna-modification-call-filtering
description: Use when reprocessing ONT POD5 files from direct RNA (or cDNA/DNA) runs
  requires basecalling with Dorado, mapping with minimap2, and modkit extraction/filtering
  of modified-base calls into filtered BED/bedMethyl outputs.
when_to_use_negative:
- Input starts from already-mapped BAMs and only transcript quantification is needed
  — use the standalone kallisto entry point instead of re-running basecalling and
  modification extraction
- Data are single-cell cDNA runs requiring UMI/barcode processing with splitcode and
  kallisto/bustools rather than bulk modification site filtering
- Reads are not Oxford Nanopore (POD5/modBAM) data — the Dorado/modkit chain is specific
  to ONT modified-base MM/ML tags
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Dogme
  role: Nextflow pipeline orchestrating basecalling, mapping, modkit extraction, and
    filtering of modification calls
  repo: https://github.com/mortazavilab/dogme
- name: Dorado
  role: Performs modified basecalling of POD5 files, producing modBAMs with MM/ML
    modification tags
  repo: nanoporetech/dorado
- name: minimap2
  role: Maps Dorado-merged BAMs to the genome with spliced alignment (GTF-derived
    junction BED, --splice-max 500000)
- name: modkit
  role: Extracts and converts modifications from modBAM to bedMethyl; outputs are
    then threshold-filtered
  repo: nanoporetech/modkit
- name: filterbed.py
  role: Dogme helper script that filters modkit BED outputs by params.minCov and params.perMod
    thresholds, reading/writing compressed .bed.gz files
  repo: https://github.com/mortazavilab/dogme
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:ecfa07f4f3630aec29806d771bd30c0411bef86a1f63d5c5cc6cd62404393f04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - filterbed.py
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# rna-modification-call-filtering

## Summary

Extract and filter nanopore direct RNA modification calls (m6A, m5C, inosine, pseudouridine, Nm) using modkit on Dorado-basecalled, minimap2-mapped reads within the Dogme Nextflow pipeline, producing filtered bedMethyl/BED site sets that support cross-replicate reproducibility analyses.

## When to use

Use when reprocessing ONT POD5 files from direct RNA (or cDNA/DNA) runs requires basecalling with Dorado, mapping with minimap2, and modkit extraction/filtering of modified-base calls into filtered BED/bedMethyl outputs.

## When NOT to use

- Input starts from already-mapped BAMs and only transcript quantification is needed — use the standalone kallisto entry point instead of re-running basecalling and modification extraction
- Data are single-cell cDNA runs requiring UMI/barcode processing with splitcode and kallisto/bustools rather than bulk modification site filtering
- Reads are not Oxford Nanopore (POD5/modBAM) data — the Dorado/modkit chain is specific to ONT modified-base MM/ML tags

## Inputs

- ONT POD5 files (direct RNA, cDNA, or genomic DNA read folders)
- Genome FASTA and GTF annotation references (genome_annot_refs)
- Dorado basecalled, merged BAMs (modBAM with MM/ML tags)
- Filtering parameters params.minCov and params.perMod (Dogme config)

## Outputs

- Filtered modification BED files (compressed .bed.gz) per sample and modification type (m6A, m5C, inosine, pseudouridine, Nm)
- bedMethyl tables produced by modkit from modBAM
- Per-replicate cross-sample comparison CSVs (e.g. Compare_GRCh38_m6A.csv via scripts/report_bed_rep.py)
- ${sample}.softwareVersion.txt recording software and model versions used

## How to apply

Given a folder of POD5 files, run the Dogme Nextflow pipeline: it downloads the latest Dorado models, runs Dorado basecalling, merges BAMs, and maps reads with minimap2 (with automatic GTF-to-junction-BED conversion and --splice-max 500000 for RNA spliced alignment). For RNA and DNA read types, extract modifications using modkit (version 0.5 or higher is supported, e.g. for open-chromatin workflows) and filter the resulting bedMethyl/BED outputs using scripts/filterbed.py, which applies params.minCov (minimum coverage) and params.perMod (per-modification threshold) thresholds and reads/writes compressed .bed.gz files. Filtered per-replicate BED sets can then be intersected across replicates (e.g. three C2C12 dRNA biological replicates) to recover conserved modification sites per modification type. Validate outputs against expected reproducibility figures such as 96,603 shared m6A sites (63.2% of the average per-sample total) and 43,476 shared m5C sites across three replicates.

## Related tools

- **Dogme** (Nextflow pipeline orchestrating basecalling, mapping, modkit extraction, and filtering of modification calls) — https://github.com/mortazavilab/dogme
- **Dorado** (Performs modified basecalling of POD5 files, producing modBAMs with MM/ML modification tags) — nanoporetech/dorado
- **minimap2** (Maps Dorado-merged BAMs to the genome with spliced alignment (GTF-derived junction BED, --splice-max 500000))
- **modkit** (Extracts and converts modifications from modBAM to bedMethyl; outputs are then threshold-filtered) — nanoporetech/modkit
- **filterbed.py** (Dogme helper script that filters modkit BED outputs by params.minCov and params.perMod thresholds, reading/writing compressed .bed.gz files) — https://github.com/mortazavilab/dogme

## Examples

```
nextflow run main.nf --pod5 /data/pod5 --readType 'RNA' --minCov 20 --perMod 0.75   # Dogme: Dorado basecall → minimap2 map → modkit extract → filterbed.py filter
```

## Evaluation signals

- Filtered outputs are valid compressed .bed.gz files reflecting the applied params.minCov and params.perMod thresholds
- Cross-replicate three-way intersections of filtered m6A sites recover ~96,603 shared sites (63.2% of average per-sample total) and ~43,476 shared m5C sites for the three C2C12 dRNA replicates
- Validation reference counts match: 96,603 m6A, 43,476 m5C, 8,829 inosine, 10,055 pseudouridine, 30,320 Nm sites
- Gene-coverage checks recover reported m6A coverage of 12,179 genes (88.2%) and m5C coverage of 8,547 genes (74.1%)
- ${sample}.softwareVersion.txt records the Dorado model and tool versions (dorado, minimap2 2.28, modkit 0.5, samtools 1.15.1) actually used in the run

## Limitations

- modkit open-chromatin features require modkit 0.5 or higher; older modkit versions are not supported for that workflow
- Dorado model downloads occur only if the model directory does not already exist, so stale cached models could be silently reused
- Reproducibility counts depend on strict three-way replicate intersections; per-replicate totals vary, so shared-site fractions are computed against the average per-replicate total
- Dorado model selection is automatic but specific model versions/checksums are not pinned in the article context

## Evidence

- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] readme filterbed thresholds: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract detected sites: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [readme] readme basecalling-mapping chain: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [abstract] abstract pipeline chain: "Dogme automates the reprocessing of ONT POD5 files by integrating basecalling using Dorado, read mapping using minimap2 and subsequent analysis steps such as running modkit"
