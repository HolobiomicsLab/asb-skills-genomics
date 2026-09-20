---
name: viral-consensus-genome-file-organization
description: Reference for assembling the results directory of a completed ViMOP Nextflow
  run for one sample (e.g. barcode01), so consensus FASTAs, BAMs, contigs, tables
  and the HTML report follow the pipeline's layout.
when_to_use_negative:
- The task is running or re-running the ViMOP pipeline itself rather than organizing/rendering
  its completed per-sample outputs
- Input is raw unclassified FASTQ data — output generation only applies after trimming,
  Centrifuge classification, filtering and assembly have completed
- You need untargeted novel virus discovery rather than packaging results of detecting
  known virus species
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: pipeline whose per-sample results (consensus FASTAs, BAMs, VCFs, contigs,
    tables, HTML report) are organized by this skill
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: workflow engine that executes ViMOP and produces the results directory being
    organized
  repo: https://github.com/opr-group-bnitm/vimop
- name: Docker
  role: default execution profile under which the ViMOP Nextflow run generating these
    outputs is performed
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:fd37ea837ab27ccdc2dc0c211e5a15ec072c10fe9fb2a8dbccd51c5701fc602b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# viral-consensus-genome-file-organization

## Summary

Organizes ViMOP per-sample outputs into consensus, assembly, selected_consensus and tables folders (consensus/contig FASTAs, BAM+BAI, VCF, depth tables, consensus.tsv/contigs.tsv/reads.tsv) and renders a three-section HTML report per barcode.

## When to use

Reference for assembling the results directory of a completed ViMOP Nextflow run for one sample (e.g. barcode01), so consensus FASTAs, BAMs, contigs, tables and the HTML report follow the pipeline's layout.

## When NOT to use

- The task is running or re-running the ViMOP pipeline itself rather than organizing/rendering its completed per-sample outputs
- Input is raw unclassified FASTQ data — output generation only applies after trimming, Centrifuge classification, filtering and assembly have completed
- You need untargeted novel virus discovery rather than packaging results of detecting known virus species

## Inputs

- ViMOP Nextflow results directory for one sample/barcode (barcode01, ...)
- classification, consensus, assembly and selected_consensus artifacts
- reference-accession-named consensus files (.consensus.fasta, .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta, .variants.vcf.gz)
- de novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta)

## Outputs

- consensus folder with per-reference FASTA, BAM, BAM index, VCF and depth files
- assembly folder with no-target.contigs.fasta and re-assembly.contigs.fasta
- selected_consensus folder with curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta)
- tables directory with consensus.tsv, contigs.tsv and reads.tsv
- per-sample HTML report (e.g. report_barcode01.html) in three sections

## How to apply

Load the results directory of the finished ViMOP run (classification, consensus, assembly and selected_consensus artifacts). Collect the reference-accession-named consensus outputs (e.g. AB627954.consensus.fasta plus matching .depth.txt, .reads.bam, .reads.bam.bai, .reference.fasta and .variants.vcf.gz) into the consensus folder. Place the de novo contig FASTAs (no-target.contigs.fasta, re-assembly.contigs.fasta) and curated segment FASTAs (e.g. LCMV_L.fasta, LCMV_S.fasta) into assembly and selected_consensus. Write consensus.tsv, contigs.tsv and reads.tsv in the tables directory holding the same numbers shown in the report. Render the per-sample HTML report with three sections: read filtering statistics, identified viruses with genome recovery statistics, and contigs with BLAST hits and Centrifuge classification.

## Related tools

- **ViMOP** (pipeline whose per-sample results (consensus FASTAs, BAMs, VCFs, contigs, tables, HTML report) are organized by this skill) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (workflow engine that executes ViMOP and produces the results directory being organized) — https://github.com/opr-group-bnitm/vimop
- **Docker** (default execution profile under which the ViMOP Nextflow run generating these outputs is performed) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq reads/ --outdir results/ --targets "LCMV" --output_krona_plot true
```

## Evaluation signals

- report_barcode01.html exists in the results directory
- tables directory contains populated consensus.tsv, contigs.tsv and reads.tsv whose numbers match the HTML report
- every consensus FASTA (e.g. AB627954.consensus.fasta) has a matching .reads.bam, .reads.bam.bai and .variants.vcf.gz in the consensus directory
- assembly folder contains both no-target.contigs.fasta and re-assembly.contigs.fasta
- selected_consensus folder holds curated segment FASTAs matching the identified virus segments

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), consensus assemblies may contain inaccuracies and should be regarded as draft genomes
- A bcftools consensus-call error can occur when <INV>-denoted inversions are present in the VCF file
- HTML report summarization may fail with Nextflow 25.10.2 ('Cannot invoke method optional() on null object'), fixed only in ViMOP 1.0.4

## Evidence

- [abstract] Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report.: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [readme] A directory with fastq files is passed to --fastq.: "The directory can contain subdirectories called barcode01, barcode02, ... In this case, the different barcodes are treated as different samples with separate output produced for each."
- [readme] we recommend carefully reviewing your output: "In any case, we recommend carefully reviewing your output (e.g. the alignment .bam files)."
- [readme] A Kronaplot of the read classification will appear in the report: "A Kronaplot of the read classification will appear in the report and optionally you can choose to generate a separate HTML with the Krona plot with --output_krona_plot true."
