---
name: rnaseq-read-quality-trimming
description: Use when Illumina short-read FASTQ files (e.g. ENA PRJEB36584) must be
  adapter- and quality-trimmed prior to genome-guided spliced alignment with HISAT2
  in a hybrid long-read/short-read transcriptome assembly workflow.
when_to_use_negative:
- Long PacBio reads (ENA PRJEB36569) — these are aligned directly with minimap2, not
  trimmed with fastp
- Reads already adapter-trimmed and quality-filtered by upstream processing
- Assemblies or BAM files rather than raw reads — trimming only applies to FASTQ input
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3572
- http://edamontology.org/topic_3308
tools:
- name: fastp
  role: Automatic adapter and quality trimming of Illumina reads prior to HISAT2 alignment
- name: HISAT2
  role: Downstream spliced aligner that consumes the fastp-trimmed Illumina reads
    against galGal6
- name: Nextflow
  role: Workflow engine wrapping the trimming and alignment steps in the annotate_my_genomes
    pipeline
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:0389a1a66e7c8abd39bd4b242a0b236bf2f727e070bdea59ffa1bd2dcffccd82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - HISAT2
  - Nextflow
  - fastp
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3572
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rnaseq-read-quality-trimming

## Summary

Quality-trim Illumina short reads with fastp before genome alignment, as done in the chicken SCO hybrid RNA-seq workflow where fastp-trimmed reads were aligned to galGal6 with HISAT2 to build the merged PacBio+Illumina BAM assembly input.

## When to use

Use when Illumina short-read FASTQ files (e.g. ENA PRJEB36584) must be adapter- and quality-trimmed prior to genome-guided spliced alignment with HISAT2 in a hybrid long-read/short-read transcriptome assembly workflow.

## When NOT to use

- Long PacBio reads (ENA PRJEB36569) — these are aligned directly with minimap2, not trimmed with fastp
- Reads already adapter-trimmed and quality-filtered by upstream processing
- Assemblies or BAM files rather than raw reads — trimming only applies to FASTQ input

## Inputs

- Raw Illumina paired-end FASTQ reads from ENA accession PRJEB36584

## Outputs

- Trimmed Illumina FASTQ files ready for HISAT2 alignment
- fastp trimming QC report (automatic quality/adapter filtering summary)

## How to apply

Run fastp in automatic mode on the Illumina reads to remove adapters and low-quality bases without manual inspection, as the article states reads 'were automatically trimmed using fastp'. Feed the trimmed FASTQs to HISAT2 to align against the unmasked galGal6 (GCA_000002315.5) genome, producing a coordinate-sorted BAM suitable for merging with the minimap2 PacBio BAM. The merged BAM is then assembled with StringTie (-p 1 -j 2 -c 2 -v -a 4), so the trimming step directly conditions junction and coverage statistics of the resulting stringtie.gtf.

## Related tools

- **fastp** (Automatic adapter and quality trimming of Illumina reads prior to HISAT2 alignment)
- **HISAT2** (Downstream spliced aligner that consumes the fastp-trimmed Illumina reads against galGal6)
- **Nextflow** (Workflow engine wrapping the trimming and alignment steps in the annotate_my_genomes pipeline) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
fastp -i sample_R1.fastq.gz -I sample_R2.fastq.gz -o trimmed_R1.fastq.gz -O trimmed_R2.fastq.gz
```

## Evaluation signals

- fastp completes without error and emits trimmed FASTQs plus a QC report from automatic adapter/quality detection
- Trimmed reads align to galGal6 with HISAT2 producing a valid, coordinate-sortable BAM
- The trimmed-read BAM merges cleanly with the minimap2 PacBio BAM into a single sorted alignment file
- StringTie assembly of the merged BAM yields stringtie.gtf that parses as valid GTF with transcript records (min junction coverage -j 2, min anchor length -a 4 satisfied)

## Limitations

- The article reports fastp default/automatic settings without enumerating specific sliding-window or length thresholds, so parameter tuning must be justified independently
- Trimming aggressiveness affects downstream StringTie junction coverage (-j 2) and anchor length (-a 4); over-trimming can drop transcripts below these thresholds

## Evidence

- [discussion] Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [abstract] Our pipeline, implemented on Anaconda/Nextflow and Docker is an easy-to-use package: "Our pipeline, implemented on Anaconda/Nextflow and Docker is an easy-to-use package"
- [discussion] We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
