---
name: fast5-signal-data-acquisition-from-sra
description: Use when a task requires raw nanopore signal data (FAST5) for direct
  RNA or DNA datasets that are only available as tar.gz archives under SRA's s3://sra-pub-src
  buckets, e.g. to reproduce basecalling, event alignment, or signal visualization
  results.
when_to_use_negative:
- The analysis only needs basecalled sequences (FASTQ/BAM), which can be pulled directly
  from SRA without downloading the source FAST5 archives
- The dataset's raw signal is stored in POD5 format rather than FAST5; use a converter
  such as blue-crab to reach SLOW5/BLOW5 instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Squigualiser
  role: Downstream consumer of the acquired FAST5 signal data for signal-base alignment
    visualization; its documentation demonstrates the FAST5 acquisition workflow from
    SRA source buckets
  repo: https://github.com/hiruna72/squigualiser
- name: guppy (guppy_basecaller)
  role: Basecalls the extracted FAST5 files (e.g. v6.3.7 with rna_r9.4.1_70bps_hac_prom.cfg)
    to produce BAM output used to validate the acquisition succeeded
- name: Python 3.8
  role: Runtime environment in which Squigualiser (installed via pip or bioconda)
    processes the acquired signal data
  repo: https://github.com/hiruna72/squigualiser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:6b794f7b770ba168cf996a704cefa84aed1511b76c50f2301887e0d156aacd4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - Squigualiser
  - guppy (guppy_basecaller)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fast5-signal-data-acquisition-from-sra

## Summary

Acquire raw nanopore signal data (FAST5 archives) from SRA via direct S3 links to sra-pub-src, then extract them for downstream signal-level analysis such as basecalling with guppy and visualization with Squigualiser. This skill bridges public archive deposition and signal-level workflows that require FAST5 input rather than FASTQ alone.

## When to use

Use when a task requires raw nanopore signal data (FAST5) for direct RNA or DNA datasets that are only available as tar.gz archives under SRA's s3://sra-pub-src buckets, e.g. to reproduce basecalling, event alignment, or signal visualization results.

## When NOT to use

- The analysis only needs basecalled sequences (FASTQ/BAM), which can be pulled directly from SRA without downloading the source FAST5 archives
- The dataset's raw signal is stored in POD5 format rather than FAST5; use a converter such as blue-crab to reach SLOW5/BLOW5 instead

## Inputs

- SRA accession (e.g. SRR22888949, SRR22888950) with a deposited source data archive in the sra-pub-src S3 bucket
- S3 path to a tar.gz FAST5 archive (e.g. s3://sra-pub-src-6/SRR22888949/dRNA_UnMod_37C_NEBT7_BaseGfpmRNA_Fast5.tar.gz.1)
- Local disk space sufficient for the extracted FAST5 files

## Outputs

- Extracted directory of FAST5 files containing raw nanopore signal per read
- FAST5 dataset ready for basecalling (guppy_basecaller) or signal visualization (squigualiser)

## How to apply

Locate the S3 paths for the deposited FAST5 archives associated with the SRA accession of interest (e.g. SRR22888949 at s3://sra-pub-src-6/SRR22888949/dRNA_UnMod_37C_NEBT7_BaseGfpmRNA_Fast5.tar.gz.1 and SRR22888950 at s3://sra-pub-src-6/SRR22888950/dRNA_Mod_37C_NEBT7_BaseGfpmRNA_Fast5.tar.gz.1). Download each archive and extract the FAST5 files. Then feed the extracted FAST5 files to the intended downstream tool — for example, basecall with guppy_basecaller v6.3.7 using the rna_r9.4.1_70bps_hac_prom.cfg config with --bam_out and --save_path, or align signals with squigualiser. Rationale: signal-level methods (realign, eventalign, pore model training) require the raw signal stored in FAST5/SLOW5 formats, which standard SRA FASTQ retrieval does not provide.

## Related tools

- **Squigualiser** (Downstream consumer of the acquired FAST5 signal data for signal-base alignment visualization; its documentation demonstrates the FAST5 acquisition workflow from SRA source buckets) — https://github.com/hiruna72/squigualiser
- **guppy (guppy_basecaller)** (Basecalls the extracted FAST5 files (e.g. v6.3.7 with rna_r9.4.1_70bps_hac_prom.cfg) to produce BAM output used to validate the acquisition succeeded)
- **Python 3.8** (Runtime environment in which Squigualiser (installed via pip or bioconda) processes the acquired signal data) — https://github.com/hiruna72/squigualiser

## Examples

```
guppy_basecaller --config rna_r9.4.1_70bps_hac_prom.cfg --bam_out --save_path basecall_out -i fast5/ ; samtools merge basecall_out/pass/*.bam -o pass_bam.bam
```

## Evaluation signals

- The tar.gz archive downloads completely from the sra-pub-src S3 path and extracts without errors into FAST5 files
- guppy_basecaller run on the extracted FAST5 files completes and produces pass/*.bam output under the --save_path directory
- For SRR22888949 and SRR22888950 basecalled with rna_r9.4.1_70bps_hac_prom.cfg, computed pass-read percentages match the reported ~92% and ~62% respectively
- Merging the passed BAMs with 'samtools merge pass/*.bam -o pass_bam.bam' yields a valid BAM whose read count matches the sum of per-file pass reads

## Limitations

- SRA source archives (sra-pub-src S3 paths) may have non-standard suffixes such as '.tar.gz.1' and may require direct S3 access rather than the standard SRA web interface
- The published pass-read percentages (92% unmethylated, 62% methylatedU) are specific to guppy v6.3.7 with rna_r9.4.1_70bps_hac_prom.cfg; other basecaller versions or models will give different results
- Guppy basecaller downloads require ONT community access; wrappers like buttery-eel note that 'An ONT login is required to access that page'

## Evidence

- [methods] Download the FAST5 archives for SRR22888949 and SRR22888950 and extract them: [evidence span withheld — anchor status: not_found]
- [methods] Basecalling with guppy v6.3.7 config: "1. Basecall using `guppy_v6.3.7` with the model `rna_r9.4.1_70bps_hac_prom.cfg`."
- [methods] Merging passed BAM files from guppy output: "samtools merge pass/*.bam -o pass_bam.bam # merge passed BAM files to create a single BAM file"
- [readme] Squigualiser installability: "The software was developed using Python 3.8 and can be installed with pip or bioconda"
