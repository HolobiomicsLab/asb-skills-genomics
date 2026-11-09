---
name: long-read-transcriptome-annotation
description: Use when you have a merged StringTie GTF assembly from hybrid long-read
  (PacBio) plus Illumina RNA-seq and need coding/noncoding classification, GTF reconciliation
  against a reference annotation, and identification of novel genes absent from that
  reference.
when_to_use_negative:
- The organism has no assembled genome or no NCBI-available annotation in GTF format,
  since the pipeline requires both as reference inputs
- Input is raw reads rather than a genome-guided GTF assembly; upstream alignment
  (minimap2 for PacBio, fastp/HISAT2 for Illumina) and StringTie assembly must be
  done first
- The goal is de novo gene prediction without a transcript assembly; annotation pipelines
  like BRAKER1/2 or MAKER are the appropriate alternatives
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: annotate_my_genomes
  role: Core Nextflow/Anaconda/Docker pipeline performing GTF reconciliation, coding/noncoding
    classification, and novel gene identification
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: GAWN
  role: Evidence-based genome annotation of assembled transcripts via Blastx against
    Swissprot
  repo: https://github.com/enormandeau/gawn
- name: TransDecoder
  role: Predicts coding regions and deduced proteins in transcripts left unclassified
    by other steps
  repo: https://github.com/TransDecoder/TransDecoder
- name: AGAT
  role: GTF/GFF toolkit that must be installed as a pipeline dependency
  repo: https://github.com/NBISweden/AGAT
- name: pyGenomeTracks
  role: Standalone visualization of genome annotations/tracks
  repo: https://github.com/deeptools/pyGenomeTracks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:a71e889bdb03041968f8939b22a7965b97bf7966a1ce6ecffa893b6a84abefc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - Docker
  - GAWN
  - SAMtools
  - TransDecoder
  - annotate_my_genomes
  - pyGenomeTracks
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# long-read-transcriptome-annotation

## Summary

Annotate a genome-guided StringTie transcriptome assembly built from hybrid PacBio/Illumina RNA-seq using the annotate_my_genomes pipeline, which reconciles the assembly against an NCBI GTF reference, classifies coding vs long non-coding RNAs (TransDecoder, FEELnc, GAWN/Blastx), and reports novel candidate genes.

## When to use

Use when you have a merged StringTie GTF assembly from hybrid long-read (PacBio) plus Illumina RNA-seq and need coding/noncoding classification, GTF reconciliation against a reference annotation, and identification of novel genes absent from that reference.

## When NOT to use

- The organism has no assembled genome or no NCBI-available annotation in GTF format, since the pipeline requires both as reference inputs
- Input is raw reads rather than a genome-guided GTF assembly; upstream alignment (minimap2 for PacBio, fastp/HISAT2 for Illumina) and StringTie assembly must be done first
- The goal is de novo gene prediction without a transcript assembly; annotation pipelines like BRAKER1/2 or MAKER are the appropriate alternatives

## Inputs

- Merged StringTie assembly GTF from hybrid PacBio + Illumina BAM alignments
- Reference genome assembly in FASTA (non-masked), e.g. galGal6 (GCA_000002315.5)
- Reference coding gene GTF (e.g. galGal6 coding GTF)
- NCBI RefSeq GTF annotation for the same genome
- gawn_config.sh configuration file (NCPUS setting for blast processing)

## Outputs

- final_annotated.gtf with reconciled gene identities (gene_id field) and novel gene/lncRNA classification (second field)
- Coding/noncoding transcript classification including FEELnc lncRNA classes
- Reconciled gene and transcript counts (e.g. 19690 genes / 61940 transcripts)
- List of novel candidate genes absent from the reference annotation (e.g. 4292 for chicken GRCg6a)

## How to apply

Download the reference genome (e.g. galGal6 non-masked FASTA), coding GTF, and NCBI RefSeq GTF with the genome-download utility, then run annotate-my-genomes (or annotate-my-genomes.nf under the 22.04_environment.yml conda environment) passing the merged StringTie GTF (-a), the coding GTF (-r), the genome FASTA (-g), and gawn_config.sh (NCPUS=10 for blast processing) with threads (-t 20) into a dedicated output folder. The pipeline reconciles input transcripts with the previous GTF annotation, annotates proteins with GAWN/Blastx against Swissprot (-evalue 1e-5 -qcov_hsp_perc 10), trains the FEELnc classifier with known coding RNAs ('NM_' prefix transcripts) to classify lncRNAs, and predicts coding regions in remaining unclassified transcripts with TransDecoder. Inspect final_annotated.gtf: the gene_id field carries reconciled gene identities and the second field carries novel gene and lncRNA classification. Optionally run add-ncbi-annotation.nf and isoform-identification.nf afterward. In the chick SCO case this yielded 19690 reconciled genes (4292 novel candidates vs chicken GRCg6a) and 61940 reconciled transcripts, with FEELnc classifying ~27% of lncRNAs as divergent.

## Related tools

- **annotate_my_genomes** (Core Nextflow/Anaconda/Docker pipeline performing GTF reconciliation, coding/noncoding classification, and novel gene identification) — https://github.com/cfarkas/annotate_my_genomes
- **GAWN** (Evidence-based genome annotation of assembled transcripts via Blastx against Swissprot) — https://github.com/enormandeau/gawn
- **TransDecoder** (Predicts coding regions and deduced proteins in transcripts left unclassified by other steps) — https://github.com/TransDecoder/TransDecoder
- **AGAT** (GTF/GFF toolkit that must be installed as a pipeline dependency) — https://github.com/NBISweden/AGAT
- **pyGenomeTracks** (Standalone visualization of genome annotations/tracks) — https://github.com/deeptools/pyGenomeTracks

## Examples

```
nextflow run annotate-my-genomes.nf -r 22.04_environment.yml --gtf merged_stringtie.gtf -a merged_stringtie.gtf --ref galGal6_ncbiRefSeq.gtf -g galGal6.fa --gawn gawn_config.sh -t 20
```

## Evaluation signals

- final_annotated.gtf exists and parses as a valid GTF
- Gene count in final_annotated.gtf matches the reported reconciled total (19690 for the chick SCO merged assembly)
- Novel candidate gene count not present in chicken GRCg6a equals 4292
- Transcript count matches the reported 61940 reconciled transcripts
- FEELnc lncRNA classification summary reproduces the reported ~27% divergent lncRNA proportion (Figure 1D-1G, Supplementary Table 1)

## Limitations

- Pipeline applicability is restricted to organisms that have an assembled genome and an NCBI-available annotation in GTF format; no results are reported for organisms lacking these
- Protein products of assembled SSPO isoforms (89-exon and 25-exon) remain to be validated; smaller SSPO isoform-to-protein correspondence is inferred only from 200-50 kD lower bands
- When using StringTie de-novo assemblies the method performed equal or better than existing pipelines by F1-score, but benchmark precision/recall values are deferred to Supplementary Table 2 rather than the main text

## Evidence

- [abstract] an easy-to-use genome-guided transcriptome annotation pipeline that uses assembled transcripts from hybrid sequencing data as input: "an easy-to-use genome-guided transcriptome annotation pipeline that uses assembled transcripts from hybrid sequencing data as input"
- [abstract] distinguishes coding and long non-coding RNAs with gene reconciliation: "distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format"
- [discussion] Blastx against Swissprot parameters: "setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10 [61]"
- [discussion] FEELnc training with NM_ transcripts: "we assessed long noncoding RNAs training FEELnc classifier [34] with known coding RNAs from chicken (“NM_” prefix transcripts)"
- [methods] run annotate-my-genomes.nf: "2.2) Run annotate-my-genomes.nf ."
- [methods] AGAT must be installed: "the AGAT toolkit (https://github.com/NBISweden/AGAT) must be installed"
- [abstract] pipeline implemented on Anaconda/Nextflow and Docker: "Our pipeline, implemented on Anaconda/Nextflow and Docker is an easy-to-use package"
- [discussion] organism requirement caveat: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
