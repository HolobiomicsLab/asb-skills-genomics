---
name: ena-read-data-retrieval
description: Use when a workflow requires raw read data (Illumina MiSeq paired-end
  or Oxford Nanopore MinION FASTQ) and the article supplies ENA study/run accessions
  (ERP/ERR) to obtain it.
when_to_use_negative:
- The read data are already available locally as FASTQ (e.g. the npScarf tutorial's
  Kp2146_paired_1.fastq.gz / Kp2146_ONT.fastq.gz) — re-downloading from ENA is unnecessary.
- The dataset is hosted outside ENA, such as the S. cerevisiae W303 data at schatzlab.cshl.edu/data/nanocorr/
  — use the cited website instead.
- Only a draft assembly or reference genome is needed (e.g. E. coli K12 MG1655 reference
  for QUAST evaluation), not raw reads.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0080
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:07b41337fa200048d42dface4d72d5825f3e3c25920157d458ac2a5d9a7b29bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA-MEM
  - npReader
  - npScarf
  - trimmomatic
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ena-read-data-retrieval

## Summary

Retrieves raw sequencing reads deposited at the European Nucleotide Archive (ENA) by study (ERP) or run (ERR) accession, distinguishing Illumina paired-end FASTQ data from MinION nanopore run data, so downstream pipelines (e.g. npScarf scaffolding) have the correct inputs and coverage.

## When to use

Use when a workflow requires raw read data (Illumina MiSeq paired-end or Oxford Nanopore MinION FASTQ) and the article supplies ENA study/run accessions (ERP/ERR) to obtain it.

## When NOT to use

- The read data are already available locally as FASTQ (e.g. the npScarf tutorial's Kp2146_paired_1.fastq.gz / Kp2146_ONT.fastq.gz) — re-downloading from ENA is unnecessary.
- The dataset is hosted outside ENA, such as the S. cerevisiae W303 data at schatzlab.cshl.edu/data/nanocorr/ — use the cited website instead.
- Only a draft assembly or reference genome is needed (e.g. E. coli K12 MG1655 reference for QUAST evaluation), not raw reads.

## Inputs

- ENA study accession (ERP*, e.g. ERP007108, ERP008615) or run accession (ERR*, e.g. ERR654977, ERR637419, ERR868296)
- Article-stated expected sample and coverage for validation (e.g. E. coli K12 MG1655, R7.3 chemistry, 67-fold coverage)

## Outputs

- Illumina MiSeq paired-end FASTQ read files (e.g. ERR654977)
- MinION nanopore run FASTQ/basecalled read files (e.g. ERR637419) or raw pre-basecalled data (e.g. ERR868296)
- Coverage estimate of the downloaded run for use in deciding termination thresholds

## How to apply

Locate the ENA accession cited for each sequencing modality in the article, noting the distinction between study accessions (ERP, e.g. ERP007108 for the E. coli K12 MG1655 MinION data) and run accessions (ERR, e.g. ERR654977 for Illumina MiSeq and ERR637419 for the R7.3 MinION run at 67-fold genome coverage). Download the run-level FASTQ files from ENA for each accession, preserving whether the data are paired-end short reads or single-end long reads. Check whether the deposited data are basecalled reads or raw signal data: for the Kpn2146 sample the ENA run ERR868296 holds nanopore data before base-calling, whereas the npScarf tutorial also provides pre-trimmed Illumina FASTQs (trimmomatic-trimmed) on the project data site. Verify the retrieved data match the reported sample and coverage (e.g. ~67-fold for ERR637419) before handing the FASTQs to trimming (trimmomatic V0.32), assembly (SPAdes), or real-time scaffolding steps. Some datasets in this paper are not on ENA — e.g. S. cerevisiae W303 data hosted at schatzlab.cshl.edu/data/nanocorr/ — so confirm the correct host per dataset.

## Evaluation signals

- The accession resolves at ENA to the expected sample (e.g. E. coli K12 MG1655 for ERP007108/ERR637419) and sequencing platform (Illumina MiSeq vs MinION R7.3).
- Downloaded run coverage matches the article's figure (67-fold genome coverage for ERR637419).
- Illumina downloads are two paired FASTQ files (forward/reverse, e.g. Kp2146_paired_1/2 style), while nanopore downloads are single-end long reads.
- For runs noted as raw data (e.g. ERR868296), the files are pre-base-calling signal/FAST5 rather than FASTQ.

## Limitations

- Some datasets used in the paper are not deposited in ENA and must be obtained from external sites (S. cerevisiae W303 data from schatzlab.cshl.edu/data/nanocorr/; npScarf project data from data.genomicsresearch.org/Projects/npScarf/data).
- Whether ENA holds basecalled FASTQ or raw pre-base-calling data varies by run (e.g. ERR868296 is raw data before base-calling), affecting whether downstream steps can consume the download directly.

## Evidence

- [discussion] MinION data for E. coli K12 MG1655 were downloaded from ENA (ERP007108): "MinION data for the E. coli K12 MG1655 sample20 were downloaded from the European Nucleotide Archive (ENA) with accession number ERP007108."
- [discussion] Illumina MiSeq data obtained from ENA accession ERR654977: "Illumina MiSeq sequencing data for the sample were also obtained from ENA (assession number ERR654977)."
- [discussion] R7.3 MinION run with 67-fold coverage from accession ERR637419: "the chemistry R7.3 run (67-fold coverage of the genome from run accession ERR637419)"
- [readme] Kpn2146 nanopore raw data available from ENA run ERR868296: "The raw data (before base-calling) of the Kpn2146 can obtained from ENA with run accession ERR868296."
