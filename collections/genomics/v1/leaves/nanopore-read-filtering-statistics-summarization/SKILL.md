---
name: nanopore-read-filtering-statistics-summarization
description: Reference for generating or verifying per-sample output tables (reads.tsv,
  consensus.tsv, contigs.tsv) and HTML report sections summarizing read filtering
  statistics from a ViMOP Nextflow run.
when_to_use_negative:
- The input is raw FASTQ before any ViMOP run — this skill summarizes completed run
  outputs, it does not perform filtering itself.
- The pipeline was run with --centrifuge_do_classify false, so no classification/Krona
  content is available for the report's contig section.
- The goal is genome reconstruction quality assessment rather than per-sample reporting;
  consult the alignment .bam files directly instead.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ViMOP
  role: Pipeline whose per-sample output module produces the consensus/contig FASTAs,
    BAM files, TSV tables and the three-section HTML report
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Executes ViMOP via a single command, producing the results directory summarized
    by this skill
  repo: https://www.nextflow.io/
- name: Docker
  role: Default execution profile for ViMOP that isolates the toolchain generating
    the report and tables
  repo: https://www.docker.com/
- name: EPI2ME Desktop
  role: Optional ONT interface to launch the same Nextflow workflow and receive the
    HTML report
  repo: https://nanoporetech.com/software/other/epi2me-desktop-application
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:87df6802dff8a4cf157e6b95ae9f6b397946522b29dfd09c2f05815471899947
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - EPI2ME Desktop
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# nanopore-read-filtering-statistics-summarization

## Summary

Reconstruct ViMOP's per-sample summarization of read filtering statistics into tab-delimited tables and a three-section HTML report, so that read trimming, Centrifuge classification, and host/contaminant removal results are reproducibly reported per barcode.

## When to use

Reference for generating or verifying per-sample output tables (reads.tsv, consensus.tsv, contigs.tsv) and HTML report sections summarizing read filtering statistics from a ViMOP Nextflow run.

## When NOT to use

- The input is raw FASTQ before any ViMOP run — this skill summarizes completed run outputs, it does not perform filtering itself.
- The pipeline was run with --centrifuge_do_classify false, so no classification/Krona content is available for the report's contig section.
- The goal is genome reconstruction quality assessment rather than per-sample reporting; consult the alignment .bam files directly instead.

## Inputs

- ViMOP Nextflow results directory for a sample (e.g. barcode01) with classification, consensus, assembly and selected_consensus artifacts
- Centrifuge read classification output for the sample
- Reference-accession-named consensus outputs (consensus.fasta, .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta, .variants.vcf.gz)
- De novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta)

## Outputs

- tables/reads.tsv with per-sample read filtering statistics
- tables/consensus.tsv and tables/contigs.tsv
- Per-sample HTML report (e.g. report_barcode01.html) with three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification
- Optionally a separate Krona plot HTML (--output_krona_plot true)

## How to apply

After a ViMOP Nextflow run completes for a barcode (e.g. barcode01), load the results directory containing classification, consensus, assembly and selected_consensus artifacts. Collect reference-accession-named consensus outputs (e.g. AB627954.consensus.fasta with .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta and .variants.vcf.gz) into the consensus folder, and the de novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta) plus curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta) into assembly/selected_consensus folders. Write a tables directory with consensus.tsv, contigs.tsv and reads.tsv holding exactly the numbers shown in the HTML report. Render the per-sample HTML report with three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification. The filtering statistics derive from read trimming (--trim_length), Centrifuge classification/removal of non-viral reads (--centrifuge_do_filter, --centrifuge_filter_min_score, default example 150), and mapping-based host/contaminant removal (--contamination_filters, default human).

## Related tools

- **ViMOP** (Pipeline whose per-sample output module produces the consensus/contig FASTAs, BAM files, TSV tables and the three-section HTML report) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Executes ViMOP via a single command, producing the results directory summarized by this skill) — https://www.nextflow.io/
- **Docker** (Default execution profile for ViMOP that isolates the toolchain generating the report and tables) — https://www.docker.com/
- **EPI2ME Desktop** (Optional ONT interface to launch the same Nextflow workflow and receive the HTML report) — https://nanoporetech.com/software/other/epi2me-desktop-application

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq path/to/fastq --outdir results --output_krona_plot true
```

## Evaluation signals

- report_barcode01.html exists and contains exactly three sections (read filtering statistics, identified viruses with genome recovery statistics, contigs with BLAST hits and Centrifuge classification)
- consensus.tsv, contigs.tsv and reads.tsv are present in the tables directory and populated with the same numbers shown in the HTML report
- Each reference-accession-named consensus FASTA (e.g. AB627954.consensus.fasta) has a matching .reads.bam, .reads.bam.bai and .variants.vcf.gz in the consensus directory
- no-target.contigs.fasta and re-assembly.contigs.fasta are present in the assembly outputs
- Optionally, a separate Krona plot HTML is generated when --output_krona_plot true was set

## Limitations

- Large DNA viruses with extensive repetitive regions (e.g. mpox) yield draft assemblies with possible inaccuracies; outputs should be reviewed (e.g. the alignment .bam files).
- Centrifuge classification is limited in accuracy and may remove false positives when --centrifuge_do_filter is active (minimum score, e.g. 150).
- Crash when the sequence contains a non-standard nucleotide (non-ACGT, non-N) in versions before 1.0.1; Nextflow 25.10.2 bug fixed only in 1.0.4.

## Evidence

- [abstract] Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report.: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [readme] A directory with fastq files is passed to --fastq. The directory can contain subdirectories called barcode01, barcode02, ... In this case, the different barcodes are treated as different samples with separate output produced for each.: "the different barcodes are treated as different samples with separate output produced for each"
- [readme] Centrifuge is used to classify the reads. This helps to get an overview of how your sample is composed. A Kronaplot of the read classification will appear in the report and optionally you can choose to generate a separate HTML with the Krona plot with --output_krona_plot true.: "A Kronaplot of the read classification will appear in the report and optionally you can choose to generate a separate HTML with the Krona plot with --output_krona_plot true."
- [readme] The pipeline will remove all reads that are classified to anything that is not viral with at least this score.: "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
