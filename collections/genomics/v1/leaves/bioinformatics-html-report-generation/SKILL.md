---
name: bioinformatics-html-report-generation
description: 'Use when a completed ViMOP Nextflow run (e.g. a barcode01 sample directory)
  must be turned into its per-sample deliverables: consensus/contig FASTAs, BAM alignments,
  consensus.tsv/contigs.tsv/reads.tsv tables, and the structured HTML report.'
when_to_use_negative:
- The ViMOP Nextflow run has not completed or its results directory lacks the consensus,
  assembly or classification artifacts needed to populate the report
- The user needs raw read QC or basecalling summaries rather than a summary of assembled/consensus
  virus-genome outputs
- Samples are not organized as barcode01, barcode02, ... subdirectories under the
  --fastq directory, so ViMOP will not treat them as separate per-sample outputs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ViMOP
  role: Pipeline that produces the per-sample consensus, contig, table and HTML-report
    outputs this skill assembles and renders
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine running ViMOP in a single command whose results directory
    feeds the report generation
  repo: https://www.nextflow.io/
- name: EPI2ME Desktop
  role: Optional ONT interface for launching ViMOP and viewing the summarized HTML
    report
  repo: https://nanoporetech.com/software/other/epi2me-desktop-application
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:51ff2d1d87c605f8784cc09aef4df98ab813ec8b70ac343290f5c5b8997fd99a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - EPI2ME Desktop
  - Nextflow
  - ViMOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bioinformatics-html-report-generation

## Summary

Reconstructs ViMOP's per-sample output generation: collecting reference-accession-named consensus FASTA/BAM/VCF artifacts, de novo contig and curated segment FASTAs, tab-delimited result tables, and rendering a three-section per-sample HTML report summarizing a Nextflow nanopore virus-reconstruction run.

## When to use

Use when a completed ViMOP Nextflow run (e.g. a barcode01 sample directory) must be turned into its per-sample deliverables: consensus/contig FASTAs, BAM alignments, consensus.tsv/contigs.tsv/reads.tsv tables, and the structured HTML report.

## When NOT to use

- The ViMOP Nextflow run has not completed or its results directory lacks the consensus, assembly or classification artifacts needed to populate the report
- The user needs raw read QC or basecalling summaries rather than a summary of assembled/consensus virus-genome outputs
- Samples are not organized as barcode01, barcode02, ... subdirectories under the --fastq directory, so ViMOP will not treat them as separate per-sample outputs

## Inputs

- ViMOP Nextflow results directory for a sample (e.g. barcode01) containing classification, consensus, assembly and selected_consensus artifacts
- Reference-accession-named consensus outputs (.consensus.fasta, .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta, .variants.vcf.gz)
- De novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta)
- Curated selected-consensus segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta)

## Outputs

- Per-sample HTML report (e.g. report_barcode01.html) with three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification
- tables/consensus.tsv, tables/contigs.tsv, tables/reads.tsv
- consensus/ directory with per-reference FASTA, BAM/BAI, depth.txt, reference FASTA and variants.vcf.gz files
- assembly/ and selected_consensus/ FASTA directories
- Optional separate Krona plot HTML (--output_krona_plot true)

## How to apply

Load the results directory of the finished ViMOP Nextflow run for the sample (e.g. barcode01), which holds classification, consensus, assembly and selected_consensus artifacts. Collect the reference-accession-named consensus outputs (e.g. AB627954.consensus.fasta with its .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta and .variants.vcf.gz) into the consensus folder, and place the de novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta) plus curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta) into assembly/ and selected_consensus/. Write the tables directory with consensus.tsv, contigs.tsv and reads.tsv so the numbers match what the report displays. Render the per-sample HTML report in three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification; optionally include the Centrifuge Krona plot when --output_krona_plot true was set. The rationale is that results from a single Nextflow command (or EPI2ME Desktop run) must be summarized in a self-contained HTML report plus machine-readable per-sample files for downstream review, e.g. checking the alignment .bam files.

## Related tools

- **ViMOP** (Pipeline that produces the per-sample consensus, contig, table and HTML-report outputs this skill assembles and renders) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine running ViMOP in a single command whose results directory feeds the report generation) — https://www.nextflow.io/
- **EPI2ME Desktop** (Optional ONT interface for launching ViMOP and viewing the summarized HTML report) — https://nanoporetech.com/software/other/epi2me-desktop-application

## Examples

```
nextflow run opr-group-bnitm/vimop -profile docker --fastq <fastq_dir_with_barcode_subdirs>
```

## Evaluation signals

- report_barcode01.html exists and renders three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification
- tables/ contains populated consensus.tsv, contigs.tsv and reads.tsv whose numbers match those displayed in the HTML report
- Every consensus FASTA in consensus/ named by reference accession (e.g. AB627954.consensus.fasta) has a corresponding .reads.bam, .reads.bam.bai, .depth.txt, .reference.fasta and .variants.vcf.gz
- no-target.contigs.fasta and re-assembly.contigs.fasta are present in the assembly folder and curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta) in selected_consensus/
- If --output_krona_plot true was used, the Centrifuge Krona plot appears in the report or as a separate HTML

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Output should be carefully reviewed manually, e.g. the alignment .bam files, as recommended by the ViMOP authors
- Per-sample outputs are only produced when the --fastq input directory contains barcode01, barcode02, ... subdirectories treated as separate samples

## Evidence

- [abstract] Results are delivered as an HTML report via a single Nextflow command: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [intro] ViMOP detects known virus species and reconstructs their genomes: "ViMOP is a pipeline for detecting known virus species in untargeted Oxford Nanopore sequencing data and reconstructing their genomes."
- [readme] Krona plot appears in the report, optionally as separate HTML: "A Kronaplot of the read classification will appear in the report and optionally you can choose to generate a separate HTML with the Krona plot with --output_krona_plot true."
- [readme] Recommendation to review output BAM alignments: "In any case, we recommend carefully reviewing your output (e.g. the alignment .bam files)."
- [readme] Per-sample treatment of barcode subdirectories: "In this case, the different barcodes are treated as different samples with separate output produced for each."
