---
name: contig-blast-centrifuge-annotation-reporting
description: Use when generating or interpreting ViMOP per-sample deliverables (consensus/contig
  FASTA, .reads.bam, consensus.tsv/contigs.tsv/reads.tsv, report_barcode01.html) or
  annotating untyped metagenomic contigs against the ViMOP virus reference database.
when_to_use_negative:
- The input is targeted sequencing against an already-known reference genome, so contig-to-reference
  BLAST matching adds no value
- You need finished genomes of large repetitive DNA viruses (e.g. mpox) — ViMOP output
  there is explicitly draft and requires manual curation beyond this reporting module
- You only need read-level taxonomic classification without de novo assembly; --centrifuge_do_classify
  false / separate classification workflows would be more appropriate
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0091
tools:
- name: ViMOP
  role: Pipeline that assembles de novo contigs, BLASTs them to the virus reference
    database, classifies contigs with Centrifuge and renders the per-sample HTML report
    and TSV tables
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine; a single Nextflow command (or EPI2ME Desktop) runs the pipeline
    and triggers report generation
  repo: https://www.nextflow.io/
- name: Docker
  role: Default container profile under which the ViMOP output/reporting processes
    run
  repo: https://github.com/OPR-group-BNITM/vimop/blob/main/tutorials/01b_installation_tutorial_command_line.md
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:6d095604a0fcc59d171eed145142ed4ed7cd22d390d42513fb808065daecbfa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# contig-blast-centrifuge-annotation-reporting

## Summary

Reconstructs ViMOP's per-sample output module: de novo contigs are matched to known viral references by BLAST and cross-checked with Centrifuge classification, then results are packaged into per-sample consensus/contig FASTA, BAM alignments, tab-delimited tables and a three-section HTML report.

## When to use

Use when generating or interpreting ViMOP per-sample deliverables (consensus/contig FASTA, .reads.bam, consensus.tsv/contigs.tsv/reads.tsv, report_barcode01.html) or annotating untyped metagenomic contigs against the ViMOP virus reference database.

## When NOT to use

- The input is targeted sequencing against an already-known reference genome, so contig-to-reference BLAST matching adds no value
- You need finished genomes of large repetitive DNA viruses (e.g. mpox) — ViMOP output there is explicitly draft and requires manual curation beyond this reporting module
- You only need read-level taxonomic classification without de novo assembly; --centrifuge_do_classify false / separate classification workflows would be more appropriate

## Inputs

- barcode-partitioned FASTQ directory (barcode01, barcode02, ...) passed via --fastq
- ViMOP virus reference database (ViMOP_DB) with Centrifuge index and BLAST reference sequences
- contamination.yaml database config defining host/contaminant reference sets

## Outputs

- Per-sample HTML report (e.g. report_barcode01.html) with three sections: read filtering statistics, identified viruses with genome recovery statistics, contigs with BLAST hits and Centrifuge classification
- Consensus folder: per-reference FASTA, .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta, .variants.vcf.gz (e.g. AB627954.*)
- Assembly outputs: no-target.contigs.fasta and re-assembly.contigs.fasta, plus curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta)
- tables/ directory with consensus.tsv, contigs.tsv and reads.tsv
- Optional standalone Krona HTML plot (--output_krona_plot true)

## How to apply

Run the ViMOP Nextflow pipeline on barcode-split FASTQ input so each barcode is treated as a separate sample with separate output. For each filtered read set, the assembly produces contigs; every contig is searched by BLAST against the virus reference database and the highest scoring hit is adopted as the reference genome for reference-guided consensus assembly, while Centrifuge independently classifies contigs to flag those that match no database entry or only partially. Collect the reference-accession-named consensus artifacts (e.g. AB627954.consensus.fasta with .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta and .variants.vcf.gz), the de novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta) and curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta). Write the tables directory with consensus.tsv, contigs.tsv and reads.tsv holding the same numbers shown in the report, then render the per-sample HTML report in three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification. Optionally emit a standalone Krona HTML with --output_krona_plot true for the read-classification overview.

## Related tools

- **ViMOP** (Pipeline that assembles de novo contigs, BLASTs them to the virus reference database, classifies contigs with Centrifuge and renders the per-sample HTML report and TSV tables) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine; a single Nextflow command (or EPI2ME Desktop) runs the pipeline and triggers report generation) — https://www.nextflow.io/
- **Docker** (Default container profile under which the ViMOP output/reporting processes run) — https://github.com/OPR-group-BNITM/vimop/blob/main/tutorials/01b_installation_tutorial_command_line.md

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq /data/fastq --output_krona_plot true -profile docker
```

## Evaluation signals

- report_barcode01.html exists and contains the three expected sections (read filtering statistics, identified viruses with genome recovery statistics, contigs with BLAST hits and Centrifuge classification)
- tables/consensus.tsv, tables/contigs.tsv and tables/reads.tsv are present, populated, and their numbers match those shown in the HTML report
- Every consensus FASTA in the consensus folder has matching .reads.bam, .reads.bam.bai, .reference.fasta, .variants.vcf.gz and .depth.txt sidecars named by reference accession
- Each reported contig carries a BLAST hit against the ViMOP virus reference database plus a Centrifuge classification (or is flagged as non-matching/partial)
- Optionally, the standalone Krona HTML is generated when --output_krona_plot true was set

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge is limited in accuracy; its read filter may remove some false positives, so classification-based annotation of contigs is only a rough indication for contigs that do not match any reference database entry or only partially
- An unresolved error occurs when <INV>-denoted inversions are present in the VCF file before consensus calling with bcftools, which can interrupt output generation
- A bug causing a crash on non-standard (non-ACGT, non-N) nucleotides was only fixed in ViMOP 1.0.1; output should be reviewed carefully (e.g. the alignment .bam files)

## Evidence

- [readme] Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome.: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [readme] Centrifuge also classifies the contigs to get a rough idea about contigs that do not match any entry in the reference database or only partially.: "Centrifuge also classifies the contigs (see later) to get a rough idea about contigs that do not match any entry in the reference database or only partially."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [abstract] Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report.: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [readme] The directory can contain subdirectories called barcode01, barcode02, ... In this case, the different barcodes are treated as different samples with separate output produced for each.: "In this case, the different barcodes are treated as different samples with separate output produced for each."
- [readme] In any case, we recommend carefully reviewing your output (e.g. the alignment .bam files).: "In any case, we recommend carefully reviewing your output (e.g. the alignment .bam files)."
