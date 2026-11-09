---
name: blastn-reference-genome-validation
description: Use when a Kraken2-based taxonomic classification flags reads assigned
  to a user-defined species-of-interest watchlist taxid and those assignments need
  confirmation by blastn against user-selected reference genomes.
when_to_use_negative:
- No species-of-interest watchlist taxids were flagged by Kraken2, so there are no
  designated sequences to validate
- The goal is general taxonomic profiling of all reads rather than confirmation of
  specific watchlist detections — use Kraken2 classification directly
- Input is raw FASTQ that has not yet been taxonomically classified by Kraken2 in
  the nanometanf pipeline
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: BLAST
  role: blastn search of Kraken2-flagged species-of-interest sequences against user-selected
    reference genomes
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Kraken2
  role: taxonomic classification whose per-read output supplies the reads assigned
    to flagged taxids
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: Nanometa Live
  role: orchestrates watchlist flagging and presents BLAST validation metrics in the
    dashboard validation tab
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:45ad3838a2856fced82ab378f8a11956417f54c2d85a454c615bb968ff730f0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - BLAST
  - Kraken2
  - Nanometa Live
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# blastn-reference-genome-validation

## Summary

Automated BLAST validation of metagenomic sequences that Kraken2 assigns to user-defined species of interest, as implemented in the Nanometa Live / nanometanf workflow for Oxford Nanopore data.

## When to use

Use when a Kraken2-based taxonomic classification flags reads assigned to a user-defined species-of-interest watchlist taxid and those assignments need confirmation by blastn against user-selected reference genomes.

## When NOT to use

- No species-of-interest watchlist taxids were flagged by Kraken2, so there are no designated sequences to validate
- The goal is general taxonomic profiling of all reads rather than confirmation of specific watchlist detections — use Kraken2 classification directly
- Input is raw FASTQ that has not yet been taxonomically classified by Kraken2 in the nanometanf pipeline

## Inputs

- Per-sample Kraken2 reports (results/kraken2/, e.g. sample1.kreport2.txt)
- Kraken2 per-read classification output for reads assigned to species-of-interest taxids
- User-defined species-of-interest watchlist taxids (e.g. Bacillus anthracis taxid 1392, Yersinia pestis taxid 632, Clostridium botulinum taxid 1491)
- User-selected reference genomes per flagged species

## Outputs

- Per-sample BLAST result files at results/blast/<sample>_<species>.txt (e.g. sample1_species1.txt)
- validation_results with per-method statistics and identity/coverage/hit-rate metrics surfaced in the dashboard validation tab

## How to apply

Load per-sample Kraken2 reports from results/kraken2 (e.g. sample1.kreport2.txt) together with the species-of-interest watchlist taxids to flag designated organisms. Extract the reads assigned to the flagged taxids from the Kraken2 classification output using KrakenTools. Search these designated sequences with BLAST blastn against the reference genomes the user selected for each flagged species, since Kraken2 k-mer classification alone is treated as insufficient for pathogen-level calls. Write per-sample BLAST results to results/blast/ as <sample>_<species>.txt files (e.g. sample1_species1.txt) and aggregate per-method statistics plus identity/coverage/hit-rate metrics into validation_results for the dashboard validation tab.

## Related tools

- **BLAST** (blastn search of Kraken2-flagged species-of-interest sequences against user-selected reference genomes) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Kraken2** (taxonomic classification whose per-read output supplies the reads assigned to flagged taxids) — https://github.com/FOI-Bioinformatics/nanometanf
- **Nanometa Live** (orchestrates watchlist flagging and presents BLAST validation metrics in the dashboard validation tab) — https://github.com/FOI-Bioinformatics/nanometa_live

## Examples

```
blastn -query sample1_species1.fa -ref <selected_reference_genome> -out results/blast/sample1_species1.txt
```

## Evaluation signals

- validation_results file exists and parses successfully with ValidationParser.parse_all()
- A per-sample BLAST file results/blast/<sample>_<species>.txt is present for every flagged sample-species pair
- BLAST outputs report identity/coverage/hit-rate metrics that can be aggregated into the dashboard validation tab
- Every validated sequence traceably originates from reads assigned to a species-of-interest taxid in the Kraken2 classification output
- Reference genomes used per species match those selected by the user in the configuration

## Limitations

- Validation is restricted to species on the user-defined watchlist; unexpected off-watchlist detections require on-demand validation via Nextflow -resume
- No tool versions specified for Kraken2 or BLAST in the dependency specifications, so exact version behaviour is not pinned in the article context
- BLAST validation complements but does not replace minimap2 coverage validation; both are part of the nanometanf validation layer

## Evidence

- [methods] BLAST validation step: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract BLAST feature: "Key features include automated BLAST validation"
- [readme] README validation feature: [evidence span withheld — anchor status: external_doc]
- [readme] nanometanf validation scope: "taxonomic classification (Kraken2),
and validation (BLAST, minimap2)"
