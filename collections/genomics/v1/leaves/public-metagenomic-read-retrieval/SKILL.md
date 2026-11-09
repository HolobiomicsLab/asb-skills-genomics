---
name: public-metagenomic-read-retrieval
description: Use when an analysis requires the raw reads behind a published metagenome
  sample — e.g., an HMP accession like SRS022071 or Horizon oil-spill Illumina data
  — that must be downloaded from SRA/HMP before trimming and assembly.
when_to_use_negative:
- The reads are already available locally and only require trimming or assembly steps
  downstream of retrieval
- The target sample is synthetic (e.g., MetaSim-generated reads for the 1,000-genome
  simulation) and must be generated rather than downloaded
- Only reference genomes or assembled contigs are needed and no read-level reanalysis
  is planned
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: Velvet
  role: 'Downstream consumer: retrieved HMP and oil-spill reads are trimmed and then
    assembled by Velvet among the individual Illumina assemblers'
- name: SOAPdenovo
  role: 'Downstream consumer: assembles the retrieved and trimmed Illumina reads as
    one of the individual assemblies pooled into MeGAMerge'
- name: CLC Bio de novo assembler
  role: 'Downstream consumer: assembles the retrieved and trimmed Illumina reads as
    one of the individual assemblies pooled into MeGAMerge'
- name: MeGAMerge
  role: Terminal consumer of the retrieved read-derived contig sets; the skill supplies
    the raw reads that ultimately feed the merged MergedContigs.fasta output
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:4b91669fc7fd77215bfefac2bce9202643a94120d1c6f3776938df7edf167ba2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - AMOS toolbox
  - CLC Bio de novo assembler
  - IDBA
  - MeGAMerge
  - Minimus2
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# public-metagenomic-read-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw Illumina metagenomic sequencing reads for published samples (e.g., the HMP shotgun sample SRS022071 and Horizon oil-spill data) from public SRA/HMP repositories so they can be trimmed and used as input to downstream assembly pipelines such as MeGAMerge.

## When to use

Use when an analysis requires the raw reads behind a published metagenome sample — e.g., an HMP accession like SRS022071 or Horizon oil-spill Illumina data — that must be downloaded from SRA/HMP before trimming and assembly.

## When NOT to use

- The reads are already available locally and only require trimming or assembly steps downstream of retrieval
- The target sample is synthetic (e.g., MetaSim-generated reads for the 1,000-genome simulation) and must be generated rather than downloaded
- Only reference genomes or assembled contigs are needed and no read-level reanalysis is planned

## Inputs

- Public repository accession for a published metagenome sample (e.g., HMP sample SRS022071)
- Horizon oil-spill Illumina dataset identifiers from SRA
- Supplementary Table S1 metadata (read length, number of reads, total bases per sample) for validation

## Outputs

- Raw Illumina FASTQ read files per sample
- Quality-trimmed FASTQ reads (sliding-window protocol, minimum quality score 2) ready for de novo assembly

## How to apply

Identify the sample accessions (e.g., HMP SRS022071, Horizon oil-spill Illumina runs) cited in the study; read lengths, read counts, and total bases per sample are tabulated in the paper's Supplementary Table S1 (21 metagenome datasets, including 17 HMP shotgun samples — 3 with both Illumina and 454 data — and 3 MetaHit samples). Download the FASTQ read sets from the public SRA/HMP repositories. As the first pipeline step, trim the downloaded reads using a sliding-window based trimming protocol with a minimum quality score of 2 to remove poor-quality data. Verify retrieval success by checking that read counts and total bases match the values recorded in Supplementary Table S1 before handing the reads to individual assemblers (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray) and MeGAMerge.

## Related tools

- **Velvet** (Downstream consumer: retrieved HMP and oil-spill reads are trimmed and then assembled by Velvet among the individual Illumina assemblers)
- **SOAPdenovo** (Downstream consumer: assembles the retrieved and trimmed Illumina reads as one of the individual assemblies pooled into MeGAMerge)
- **CLC Bio de novo assembler** (Downstream consumer: assembles the retrieved and trimmed Illumina reads as one of the individual assemblies pooled into MeGAMerge)
- **MeGAMerge** (Terminal consumer of the retrieved read-derived contig sets; the skill supplies the raw reads that ultimately feed the merged MergedContigs.fasta output) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Evaluation signals

- Downloaded read counts and total bases per sample match the values recorded in Supplementary Table S1 for that dataset
- Retrieved FASTQ files pass the sliding-window trimming protocol at minimum quality score 2 without empty output
- Subsequent individual assemblies (e.g., Velvet at kmer 21-31 and 85-105) can be generated from the retrieved reads, indicating intact, assembly-ready data
- The HMP sample SRS022071 and Horizon oil-spill accessions resolve to the correct study/sample in the SRA/HMP repositories

## Limitations

- Per-sample read length, number of reads, and total bases are documented only in Supplementary Table S1, which must be consulted separately for retrieval validation
- Some HMP samples include both Illumina and 454 data (3 of the 17 shotgun samples); retrieval must account for platform-specific read files
- Retrieval depends on the public SRA/HMP repositories remaining accessible and accessions remaining stable

## Evidence
