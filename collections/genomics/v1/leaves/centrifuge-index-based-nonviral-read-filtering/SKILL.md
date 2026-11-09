---
name: centrifuge-index-based-nonviral-read-filtering
description: Use when cleaning untargeted nanopore FASTQ reads (e.g. Lassa/Ebola/Dengue
  clinical samples) by discarding reads Centrifuge classifies as non-viral with high
  confidence before host/reagent filtering and assembly.
when_to_use_negative:
- When the goal is host-DNA removal rather than broad taxonomic decontamination —
  ViMOP removes host reads separately by minimap2 mapping against contamination.yaml
  reference sets (default human), not via Centrifuge
- When samples are highly enriched/on-target viral data with little non-viral contamination
  — deactivating with --centrifuge_do_filter false (or --centrifuge_do_classify false)
  saves time
- When filtering assembled contigs — this filter applies to reads only, not contigs,
  although contigs are separately classified for overview
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3174
tools:
- name: ViMOP
  role: Pipeline implementing the classification-based read-cleaning stage and its
    parameters (centrifuge_do_filter, centrifuge_filter_min_score, output_krona_plot)
  repo: https://github.com/opr-group-bnitm/vimop
- name: Centrifuge
  role: Classifies nanopore reads against the ViMOP_DB index; classifications drive
    removal of non-viral reads at score >= 150
- name: Nextflow
  role: Workflow engine that executes the ViMOP read-cleaning stage from a single
    command or via EPI2ME Desktop
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:6ffe02cbc50934ca48b32f981c2499835ad47bd3998bd6ec9ad2bbfb58fab97f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Centrifuge
  - Nextflow
  - ViMOP
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# centrifuge-index-based-nonviral-read-filtering

## Summary

ViMOP's read-cleaning step that classifies untargeted nanopore FASTQ reads with Centrifuge (ViMOP_DB index, virus_taxids.txt) and removes reads classified to any non-viral taxon at or above a confidence score of 150, retaining unclassified and virus-matching reads.

## When to use

Use when cleaning untargeted nanopore FASTQ reads (e.g. Lassa/Ebola/Dengue clinical samples) by discarding reads Centrifuge classifies as non-viral with high confidence before host/reagent filtering and assembly.

## When NOT to use

- When the goal is host-DNA removal rather than broad taxonomic decontamination — ViMOP removes host reads separately by minimap2 mapping against contamination.yaml reference sets (default human), not via Centrifuge
- When samples are highly enriched/on-target viral data with little non-viral contamination — deactivating with --centrifuge_do_filter false (or --centrifuge_do_classify false) saves time
- When filtering assembled contigs — this filter applies to reads only, not contigs, although contigs are separately classified for overview

## Inputs

- Directory of nanopore FASTQ files passed via --fastq (optionally with barcode01, barcode02, ... subdirectories)
- ViMOP_DB Centrifuge index (all.1.cf-all.4.cf, prefix 'all')
- virus_taxids.txt from the ViMOP database

## Outputs

- Centrifuge classification.tsv per sample
- Krona plot (embedded in HTML report; optional separate HTML via --output_krona_plot true)
- Read set with non-viral reads (score >= 150) removed, passed onward to host/reagent filtering
- Per-sample read summary table in the tables directory (reads.tsv)

## How to apply

Load the input FASTQ directory passed via --fastq (barcode01/barcode02 subdirectories are treated as separate samples) and run Centrifuge classification against the ViMOP_DB centrifuge index (all.1.cf-all.4.cf with prefix 'all' and virus_taxids.txt), producing classification.tsv plus a Krona plot (--output_krona_plot true writes a separate HTML). Then apply the centrifuge read filter, which removes reads (not contigs) classified to anything that is not viral with a score of at least --centrifuge_filter_min_score (default 150); unclassified reads and reads matching virus taxids are kept. The step can be deactivated with --centrifuge_do_filter false, or all Centrifuge classifications skipped with --centrifuge_do_classify false to save time. Rationale: this removes broad non-viral contamination such as bacterial reads from fecal samples before minimap2-based host/reagent removal against references in contamination.yaml. Note the classifier's limited accuracy: some viral reads may be false-positive removed, so review the Krona classification output when tuning the score threshold.

## Related tools

- **ViMOP** (Pipeline implementing the classification-based read-cleaning stage and its parameters (centrifuge_do_filter, centrifuge_filter_min_score, output_krona_plot)) — https://github.com/opr-group-bnitm/vimop
- **Centrifuge** (Classifies nanopore reads against the ViMOP_DB index; classifications drive removal of non-viral reads at score >= 150)
- **Nextflow** (Workflow engine that executes the ViMOP read-cleaning stage from a single command or via EPI2ME Desktop) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq data/fastq --centrifuge_do_filter true --centrifuge_filter_min_score 150 --output_krona_plot true -profile docker
```

## Evaluation signals

- reads.tsv exists in each sample's tables directory
- Cleaned read set contains no reads classified to a non-viral taxon with Centrifuge score >= 150
- Unclassified reads and reads matching taxids in virus_taxids.txt are retained in the cleaned read set
- classification.tsv and a Krona plot (embedded in report or separate HTML) are produced per sample
- Cleaned reads should still be present for downstream minimap2 host/reagent filtering and de novo assembly steps

## Limitations

- Centrifuge has limited accuracy and some viral reads may be falsely removed as non-viral at the chosen score threshold (e.g. 150)
- Filter applies only to reads, not contigs; contig-level classification is for overview only
- Large DNA viruses with repetitive regions (e.g. mpox) yield draft genomes regardless, so this cleaning step does not guarantee assembly accuracy downstream

## Evidence

- [methods] Centrifuge is used to classify the reads.: "Taxonomic classification and removal of non-viral reads — Centrifuge is used to classify the reads."
- [methods] the centrifuge classifications is used to remove reads (not contigs) that are non-viral to a given degree of confidence.: "Additionally, the centrifuge classifications is used to remove reads (not contigs) that are non-viral to a given degree of confidence."
- [readme] The pipeline will remove all reads that are classified to anything that is not viral with at least this score.: "use `--centrifuge_filter_min_score 150` to set the minimum level of confidence (e.g. 150 in our example). The pipeline will remove all reads that are classified to anything that is not viral with at"
- [results] After Centrifuge classification and removal of non-viral reads, the remaining reads are mapped with minimap2 against the host/reagent reference sequence sets: [evidence span withheld — anchor status: not_found]
- [readme] But beware that centrifuge is limited in its accuracy and you may also remove some false positives.: "But beware that centrifuge is limited in its accuracy and you may also remove some false positives."
