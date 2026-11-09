---
name: nanopore-metagenomic-classification-verification
description: Use when a real-time Oxford Nanopore (MinION/Flongle) metagenomic run
  has been classified with Kraken2 and hits to user-defined species of interest or
  pathogens need secondary confirmation before alerting.
when_to_use_negative:
- The goal is initial taxonomic profiling of all reads, not confirmation of specific
  flagged species — Kraken2 classification alone is the correct step
- No species-of-interest watchlist is defined and no taxids have been flagged, so
  there is nothing to validate with BLAST
- Reads are short-read (Illumina) metagenomic data; the skill's pipeline and QC thresholds
  are calibrated for Oxford Nanopore long reads
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0632
tools:
- name: Kraken2
  role: Produces the taxonomic classification reports and per-read assignments from
    which reads of flagged species-of-interest taxids are extracted
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: BLAST
  role: blastn validation of extracted species-of-interest sequences against user-selected
    reference genomes
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: Nanometa Live
  role: Dash web app that surfaces the watchlist detections and renders BLAST identity
    scores and validation views per sample
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: nanometanf
  role: Nextflow pipeline backend that executes the Kraken2 and BLAST/minimap2 validation
    steps and exposes the canonical output layer
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: KrakenTools
  role: Parses Kraken2 output to extract reads assigned to the flagged species-of-interest
    taxids
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:7cb59ee85adb86df0661898197e43da52a61bf0f2a019884e4ed7fe4acb68e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - BLAST
  - Kraken2
  - KrakenTools
  - Nanometa Live
  - nanometanf
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nanopore-metagenomic-classification-verification

## Summary

Verification of Kraken2 taxonomic assignments of Oxford Nanopore metagenomic reads against a user-defined species-of-interest watchlist by extracting flagged reads and validating them with BLAST (blastn) searches against user-selected reference genomes, as implemented in the Nanometa Live / nanometanf workflow.

## When to use

Use when a real-time Oxford Nanopore (MinION/Flongle) metagenomic run has been classified with Kraken2 and hits to user-defined species of interest or pathogens need secondary confirmation before alerting.

## When NOT to use

- The goal is initial taxonomic profiling of all reads, not confirmation of specific flagged species — Kraken2 classification alone is the correct step
- No species-of-interest watchlist is defined and no taxids have been flagged, so there is nothing to validate with BLAST
- Reads are short-read (Illumina) metagenomic data; the skill's pipeline and QC thresholds are calibrated for Oxford Nanopore long reads

## Inputs

- Per-sample Kraken2 reports from results/kraken2 (e.g. sample1.kreport2.txt)
- Kraken2 per-read classification output for the sample
- Species-of-interest watchlist taxids (e.g. 1392 B. anthracis, 632 Y. pestis, 1491 C. botulinum)
- User-selected reference genomes for each flagged species
- Oxford Nanopore FASTQ reads processed by the nanometanf pipeline

## Outputs

- Per-sample BLAST result files under results/blast/ named <sample>_<species>.txt (e.g. sample1_species1.txt)
- validation_results object with per-method statistics and identity/coverage/hit-rate metrics for the dashboard validation tab
- Alerts/surface detections of pathogens of interest in the Nanometa Live dashboard

## How to apply

Load the per-sample Kraken2 reports (e.g. results/kraken2/sample1.kreport2.txt) together with the watchlist taxids (e.g. Bacillus anthracis taxid 1392, Yersinia pestis 632, Clostridium botulinum 1491) to flag designated organisms. Parse the Kraken2 classification output with KrakenTools and extract only the reads assigned to the flagged species-of-interest taxids. Run blastn on those extracted sequences against the reference genomes the user selected for each flagged species, and write per-sample results to results/blast/ as <sample>_<species>.txt (e.g. sample1_species1.txt). Aggregate identity, coverage and hit-rate metrics per method into a validation_results object that the dashboard validation tab renders; in Nanometa Live this BLAST identity scoring is complemented by minimap2 coverage validation, and unexpected detections can be re-validated on demand via Nextflow -resume.

## Related tools

- **Kraken2** (Produces the taxonomic classification reports and per-read assignments from which reads of flagged species-of-interest taxids are extracted) — https://github.com/FOI-Bioinformatics/nanometa_live
- **BLAST** (blastn validation of extracted species-of-interest sequences against user-selected reference genomes) — https://github.com/FOI-Bioinformatics/nanometanf
- **Nanometa Live** (Dash web app that surfaces the watchlist detections and renders BLAST identity scores and validation views per sample) — https://github.com/FOI-Bioinformatics/nanometa_live
- **nanometanf** (Nextflow pipeline backend that executes the Kraken2 and BLAST/minimap2 validation steps and exposes the canonical output layer) — https://github.com/FOI-Bioinformatics/nanometanf
- **KrakenTools** (Parses Kraken2 output to extract reads assigned to the flagged species-of-interest taxids)

## Examples

```
nextflow run foi-bioinformatics/nanometanf --input samplesheet.csv --kraken2_db /path/to/db --outdir results -profile conda  # then extract watchlist-taxid reads with KrakenTools and blastn against selected reference genomes
```

## Evaluation signals

- validation_results exists and parses successfully with ValidationParser.parse_all()
- Per-sample BLAST files are present under results/blast/ for each flagged sample-species pair (e.g. sample1_species1.txt)
- Extracted read set contains only reads whose Kraken2 taxid assignments match the watchlist taxids
- BLAST hits report identity, coverage and hit-rate metrics that are aggregated into validation_results without missing per-method statistics
- Flagged detections appear in the dashboard validation tab with consistent sample/species naming between kraken2 and blast outputs

## Limitations

- No tool versions specified for Kraken2 or BLAST in the dependency documentation, so exact version behaviour is unverified
- Validation depends on the user having selected appropriate reference genomes for each flagged species; poor references yield uninformative blastn results
- Coverage validation relies on minimap2 in addition to BLAST identity scoring, so BLAST alone does not constitute full confirmation

## Evidence

- [methods] KrakenTools extraction of flagged reads: [evidence span withheld — anchor status: not_found]
- [methods] BLAST output layout: [evidence span withheld — anchor status: not_found]
- [methods] Validation tab rendering: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract statement of feature: "Key features include automated BLAST validation"
- [readme] README on validation modes: [evidence span withheld — anchor status: external_doc]
