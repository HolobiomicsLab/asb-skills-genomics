---
name: nanopore-read-taxonomic-classification
description: Use when you have untargeted metagenomic nanopore FASTQ reads (e.g. Lassa,
  Ebola, Dengue clinical samples) and need to remove non-viral reads based on Centrifuge
  taxonomic classification confidence before viral genome reconstruction.
when_to_use_negative:
- When you expect only viral reads and minimal contamination — Centrifuge has limited
  accuracy and may remove some false-positive viral reads
- When processing contigs — the centrifuge filter applies to reads only, not contigs;
  contig classification is a separate, coarse step
- When you have deactivated classification with --centrifuge_do_classify false — no
  classification.tsv or Krona plot will be produced
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Centrifuge
  role: Classifies nanopore reads taxonomically against the ViMOP_DB index; classifications
    drive removal of non-viral reads above the confidence-score threshold
- name: ViMOP
  role: Nextflow pipeline wrapping the classification and centrifuge-based read filtering
    step with parameters --centrifuge_do_filter and --centrifuge_filter_min_score
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine executing the ViMOP classification/cleaning module (single
    command or via EPI2ME Desktop)
  repo: https://www.nextflow.io/
- name: minimap2
  role: Maps Centrifuge-surviving reads against host/reagent reference sets from contamination.yaml
    to produce the final cleaned read set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:2367e66d87983b09d5cfc14ed6b618db2f2d9cf24dfc8f3f7650465396074e88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Centrifuge
  - Nextflow
  - ViMOP
  - minimap2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# nanopore-read-taxonomic-classification

## Summary

Classify untargeted Oxford Nanopore reads with Centrifuge against the ViMOP_DB index and remove reads classified as non-viral with a score of at least 150, producing a classification.tsv and a Krona plot. This step reduces bacterial/host background before downstream host/reagent mapping and viral genome assembly.

## When to use

Use when you have untargeted metagenomic nanopore FASTQ reads (e.g. Lassa, Ebola, Dengue clinical samples) and need to remove non-viral reads based on Centrifuge taxonomic classification confidence before viral genome reconstruction.

## When NOT to use

- When you expect only viral reads and minimal contamination — Centrifuge has limited accuracy and may remove some false-positive viral reads
- When processing contigs — the centrifuge filter applies to reads only, not contigs; contig classification is a separate, coarse step
- When you have deactivated classification with --centrifuge_do_classify false — no classification.tsv or Krona plot will be produced

## Inputs

- FASTQ files or directory of FASTQ files (--fastq; barcode01/barcode02 subdirectories treated as samples)
- ViMOP_DB Centrifuge index (all.1.cf-all.4.cf with prefix 'all')
- virus_taxids.txt (viral taxid list retained by the filter)

## Outputs

- classification.tsv (per-read Centrifuge classification)
- Krona plot HTML in the report and, optionally, a separate HTML Krona plot (--output_krona_plot true)
- filtered read set with non-viral reads (score >= min score) removed; per-sample read summary in the tables directory (reads.tsv)

## How to apply

Load the FASTQ directory passed via --fastq (with optional barcode01/barcode02 subdirectories treated as separate samples) and run Centrifuge using the ViMOP_DB centrifuge index (all.1.cf-all.4.cf with prefix 'all' and virus_taxids.txt), producing classification.tsv and optionally a separate HTML Krona plot via --output_krona_plot true. Then apply the centrifuge-based read filter: remove all reads classified to any non-viral taxon with a score of at least --centrifuge_filter_min_score (e.g. 150), keeping unclassified reads and reads matching virus taxids. This filtering can be deactivated with --centrifuge_do_filter false (or all Centrifuge classification skipped with --centrifuge_do_classify false to save time). Rationale: Centrifuge gives an overview of sample composition and a confidence-scored means to remove abundant non-viral contamination such as bacterial reads from fecal samples. Reads surviving this filter are subsequently mapped with minimap2 against host/reagent reference sets from contamination.yaml to produce the cleaned read set.

## Related tools

- **Centrifuge** (Classifies nanopore reads taxonomically against the ViMOP_DB index; classifications drive removal of non-viral reads above the confidence-score threshold)
- **ViMOP** (Nextflow pipeline wrapping the classification and centrifuge-based read filtering step with parameters --centrifuge_do_filter and --centrifuge_filter_min_score) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine executing the ViMOP classification/cleaning module (single command or via EPI2ME Desktop)) — https://www.nextflow.io/
- **minimap2** (Maps Centrifuge-surviving reads against host/reagent reference sets from contamination.yaml to produce the final cleaned read set)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --centrifuge_do_filter true --centrifuge_filter_min_score 150 --contamination_filters "reagent,mouse" --output_krona_plot true
```

## Evaluation signals

- reads.tsv exists in each sample's tables directory after the read-cleaning step
- The cleaned read set contains no reads classified to a non-viral taxon with Centrifuge score >= the configured --centrifuge_filter_min_score (e.g. 150)
- The cleaned read set contains no reads mapping to the selected host/reagent reference sets (e.g. human GCF_000001405.39_GRCh38.p13_genomic.fna.gz, or mouse/reagent when --contamination_filters "reagent,mouse" is set)
- classification.tsv and the Krona plot are present in the report/output when classification is active
- Unclassified reads and reads matching virus taxids are retained, not discarded, by the filter

## Limitations

- Centrifuge is limited in its accuracy — filtering at the confidence threshold may remove some false positives (viral reads misclassified as non-viral)
- The centrifuge filter removes reads but not contigs; contig classification is separate and only used for coarse inspection
- Filtering is deactivated-safe: --centrifuge_do_filter false skips non-viral read removal, and --centrifuge_do_classify false skips all classification (saving time but losing composition overview)

## Evidence

- [readme] Centrifuge is used to classify the reads. This helps to get an overview of how your sample is composed.: "Centrifuge is used to classify the reads. This helps to get an overview of how your sample is composed. A Kronaplot of the read classification will appear in the report"
- [readme] Non-viral read removal by score threshold: "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
- [readme] Filter accuracy caveat: "But beware that centrifuge is limited in its accuracy and you may also remove some false positives."
- [methods] Article methods on classification and filtering: "the centrifuge classifications is used to remove reads (not contigs) that are non-viral to a given degree of confidence."
- [methods] Host/contaminant removal after Centrifuge: "Host and contaminant reads are removed by mapping them against reference sequences and extracting those that map."
