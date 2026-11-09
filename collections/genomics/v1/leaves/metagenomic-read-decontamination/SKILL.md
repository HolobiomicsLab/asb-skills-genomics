---
name: metagenomic-read-decontamination
description: Use when untargeted Oxford Nanopore FASTQ reads from clinical or animal
  samples contain host (e.g. human, mouse, mastomys), reagent, or bacterial contamination
  that must be removed before viral de novo assembly.
when_to_use_negative:
- Deactivate the centrifuge filter with --centrifuge_do_filter false when Centrifuge's
  limited accuracy would risk removing true viral reads as false positives
- 'Large DNA viruses with extensive repetitive regions (e.g. mpox): decontamination
  alone does not fix assembly inaccuracies; outputs should be treated as draft genomes'
- Do not apply the non-viral read filter to contigs — the centrifuge classification
  of contigs is only for overview, filtering applies to reads (not contigs)
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Centrifuge
  role: Classifies reads against the ViMOP_DB index; classifications drive removal
    of non-viral reads at score >= 150 and generate a Krona plot
- name: minimap2
  role: Maps surviving reads against host/reagent reference sequence sets from contamination.yaml
    so mapped contaminant reads can be extracted and discarded
- name: ViMOP
  role: Nextflow-based pipeline orchestrating the classification and host/contaminant
    removal steps within the read-cleaning stage
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine running the ViMOP read-cleaning module as a single command
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:1e6488b680de4c93d9ac8bc0e1a65534d769f7dcd04e3b181533c28f69f2a799
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Centrifuge
  - Nextflow
  - ViMOP
  - minimap2
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metagenomic-read-decontamination

## Summary

Two-stage removal of non-viral reads from untargeted nanopore FASTQ data: Centrifuge taxonomic classification filters reads classified as non-viral with score >= 150, then minimap2 mapping against host/reagent reference sets (contamination.yaml) extracts and discards remaining contaminant reads, yielding a cleaned read set for downstream assembly.

## When to use

Use when untargeted Oxford Nanopore FASTQ reads from clinical or animal samples contain host (e.g. human, mouse, mastomys), reagent, or bacterial contamination that must be removed before viral de novo assembly.

## When NOT to use

- Deactivate the centrifuge filter with --centrifuge_do_filter false when Centrifuge's limited accuracy would risk removing true viral reads as false positives
- Large DNA viruses with extensive repetitive regions (e.g. mpox): decontamination alone does not fix assembly inaccuracies; outputs should be treated as draft genomes
- Do not apply the non-viral read filter to contigs — the centrifuge classification of contigs is only for overview, filtering applies to reads (not contigs)

## Inputs

- Untargeted nanopore FASTQ directory (--fastq), optionally with barcode01/barcode02 subdirectories
- ViMOP_DB Centrifuge index (all.1.cf-all.4.cf) with virus_taxids.txt
- contamination.yaml defining contaminant reference sets (e.g. GCF_000001405.39_GRCh38.p13_genomic.fna.gz human, GCA_000001635.8_GRCm38.p6_genomic.fna.gz mouse, GCF_008632895.1_UCSF_Mcou_1_genomic.fna.gz mastomys)
- --contamination_filters selection (default human reads)

## Outputs

- classification.tsv (Centrifuge per-read classifications)
- Krona plot HTML (in report and optionally separate via --output_krona_plot true)
- Cleaned read set (FASTQ) free of non-viral reads (score >= 150) and host/reagent-mapping reads
- Per-sample reads.tsv summary in the sample's tables directory

## How to apply

Load the input fastq directory passed via --fastq (with optional barcode01/barcode02 subdirectories) and run Centrifuge classification against the ViMOP_DB index (all.1.cf-all.4.cf, prefix 'all', virus_taxids.txt), producing classification.tsv and an optional Krona HTML (--output_krona_plot true). Apply the centrifuge-based read filter (--centrifuge_do_filter, deactivate-safe) to remove reads classified to any non-viral taxon with a score of at least --centrifuge_filter_min_score 150, keeping unclassified reads and virus-taxid matches. Then map surviving reads with minimap2 against the host/reagent reference sets defined in contamination.yaml, controlled by --contamination_filters (default human; e.g. 'reagent,mouse' for GCA_000001635.8_GRCm38.p6_genomic.fna.gz). Extract reads that map to the contaminant references, discard them, and retain the cleaned read set plus a per-sample reads.tsv summary in the tables directory.

## Related tools

- **Centrifuge** (Classifies reads against the ViMOP_DB index; classifications drive removal of non-viral reads at score >= 150 and generate a Krona plot)
- **minimap2** (Maps surviving reads against host/reagent reference sequence sets from contamination.yaml so mapped contaminant reads can be extracted and discarded)
- **ViMOP** (Nextflow-based pipeline orchestrating the classification and host/contaminant removal steps within the read-cleaning stage) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine running the ViMOP read-cleaning module as a single command) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --contamination_filters "reagent,mouse" --centrifuge_filter_min_score 150 --output_krona_plot true
```

## Evaluation signals

- reads.tsv exists in each sample's tables directory
- Cleaned read set contains no reads classified as non-viral with Centrifuge score >= 150
- Cleaned read set contains no reads mapping to the selected host/reagent references (e.g. GRCh38 human, GRCm38 mouse)
- classification.tsv is produced for each sample, and a Krona HTML appears in the report when --output_krona_plot true
- Unclassified reads and reads matching virus taxids are retained (not removed) by the centrifuge filter

## Limitations

- Centrifuge is limited in its accuracy and some true viral reads may be removed as false positives at the chosen confidence threshold
- The filter removes reads only, not contigs; contaminant-derived contigs must be handled downstream
- For large DNA viruses with repetitive regions (e.g. mpox), cleaned reads may still yield inaccurate draft assemblies

## Evidence

- [methods] Centrifuge classification filters non-viral reads: "Centrifuge is used to classify the reads."
- [methods] Host removal via mapping and extraction: "Host and contaminant reads are removed by mapping them against reference sequences and extracting those that map."
- [methods] Contamination filter option: "Use the option `--contamination_filters "reagent,mouse"` for example to remove mouse and reagent reads (the default is human reads)."
- [readme] Centrifuge read filter parameters (README): "Deactivate this with `--centrifuge_do_filter false` and use `--centrifuge_filter_min_score 150` to set the minimum level of confidence"
- [readme] Non-viral removal criterion (README): "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
- [readme] Contamination.yaml reference sets (README): "The database config file (contamination.yaml) defines the reference sets and the key values assigned to them."
