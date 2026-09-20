---
name: low-coverage-read-downsampling
description: 'Use when reproducing the dysgu Table 7 analysis: you need low-coverage
  PE and long-read callsets from GIAB HG002 to test whether merging PE calls into
  long-read callsets recovers SVs otherwise missed at low coverage.'
when_to_use_negative:
- Input reads are already at target low coverage — downsampling would discard data
  without changing the callset
- The goal is caller-vs-caller comparison at a single coverage level (e.g. manta/delly/lumpy
  benchmarking) rather than cross-technology merging
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: dysgu
  role: Calls SVs from downsampled paired-end and long-read BAMs and merges callsets
    across technologies with the 'merge' command
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Benchmarks merged and single-technology callsets against the GIAB HG002 truth
    set, computing precision and recall per SV type
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:08793ba3c80fa2051242b1a8904282551a8ff7fb1c1a3e98ba7671fec13b53f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - svbench
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# low-coverage-read-downsampling

## Summary

Downsamples full-coverage Illumina paired-end (GIAB NIST_HiSeq_HG002_Homogeneity-10953946) and PacBio long-read (SRR10188368, SRR10188369) HG002 data to low coverage (e.g. 8x long reads, 40x PE) so that dysgu callsets from different read technologies can be merged and compared against higher-coverage benchmarks in svbench.

## When to use

Use when reproducing the dysgu Table 7 analysis: you need low-coverage PE and long-read callsets from GIAB HG002 to test whether merging PE calls into long-read callsets recovers SVs otherwise missed at low coverage.

## When NOT to use

- Input reads are already at target low coverage — downsampling would discard data without changing the callset
- The goal is caller-vs-caller comparison at a single coverage level (e.g. manta/delly/lumpy benchmarking) rather than cross-technology merging

## Inputs

- GIAB HG002 Illumina paired-end sequencing data (NIST_HiSeq_HG002_Homogeneity-10953946, downloaded from the AshkenazimTrio GIAB FTP)
- PacBio long reads from SRA accessions SRR10188368 and SRR10188369
- Indexed reference genome FASTA
- Sorted and indexed BAM/CRAM alignment files for each read technology

## Outputs

- Downsampled low-coverage BAM/CRAM files per read technology
- Per-technology dysgu SV callsets in VCF format
- Merged PacBio+PE and ONT+PE VCFs filtered at model probability >=0.5 (PacBio) or >=0.35 (ONT)
- table7_merged_lr_pe_benchmark.tsv: svbench precision/recall per SV type comparing merged vs single-technology vs higher-coverage callsets

## How to apply

Download GIAB HG002 Illumina data (NIST_HiSeq_HG002_Homogeneity-10953946) and the two PacBio lanes (SRR10188368, SRR10188369), then downsample each dataset to low coverage. Run dysgu independently on each downsampled dataset (dysgu run for paired-end BAMs, dysgu call --mode pacbio-revio or --mode nanopore-r10 for long reads) to produce per-technology VCF callsets. Merge the Illumina paired-end calls into the long-read callsets with dysgu's 'merge' command, then filter merged calls at a maximum model probability >=0.5 for PacBio combinations or >=0.35 for ONT combinations. The rationale is to show that combining low coverage paired-end and long-reads is competitive with long-reads at higher coverage values, benchmarked per SV type against the GIAB HG002 truth set using svbench.

## Related tools

- **dysgu** (Calls SVs from downsampled paired-end and long-read BAMs and merges callsets across technologies with the 'merge' command) — https://github.com/kcleal/dysgu
- **svbench** (Benchmarks merged and single-technology callsets against the GIAB HG002 truth set, computing precision and recall per SV type) — https://github.com/kcleal/svbench

## Examples

```
dysgu run -p4 reference.fa temp_dir downsampled_pe.bam > pe_svs.vcf; dysgu call --mode pacbio-revio reference.fa temp_dir downsampled_pb.bam > pb_svs.vcf; dysgu merge --pro 0.5 pe_svs.vcf pb_svs.vcf > merged.vcf
```

## Evaluation signals

- The merged PacBio+PE and ONT+PE VCFs both exist and contain events beyond the long-read-only callset
- Merging 40x Illumina calls into 8x PacBio calls yields an additional 1286 deletions and 894 insertions for the All-regions benchmark, and 150 deletions and 33 insertions for Tier 1
- Merged calls are filtered at max model probability >=0.5 (PacBio) / >=0.35 (ONT) before benchmarking
- svbench output (table7_merged_lr_pe_benchmark.tsv) shows combined low-coverage PE+LR recall approaching that of long reads at higher coverage

## Limitations

- Subsampling is per-dataset, so resulting coverage levels and sampling variance affect the exact number of recovered SVs
- Filtering thresholds (>=0.5 PacBio, >=0.35 ONT) are specific to these combinations and may need retuning for other technologies or coverage depths
- No software or dysgu version is stated in the results section, hindering exact reproducibility

## Evidence

- [results] Adding dysgu calls from 40× Illumina data to dysgu calls from 8× PacBio data detected an additional 1286 deletions and 894 insertions for the All-regions benchmark, and 150 additional deletions and 33 insertions for Tier 1 regions.: [evidence span withheld — anchor status: not_found]
- [results] Dysgu supports merging of SVs from different runs using a 'merge' command: "Dysgu supports merging of SVs from different runs using a ‘merge’ command"
- [supplementary] Illumina sequencing data for Ashkenazim HG002 (34) sample was downloaded from GIAB: "Illumina sequencing data for Ashkenazim HG002 (34) sample was downloaded from GIAB (ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/"
- [supplementary] Two lanes of PacBio data were downloaded from SRA under accessions SRR10188368 and SRR10188369.: "Two lanes of PacBio data were downloaded from SRA (https://www.ncbi.nlm.nih.gov/sra) under accessions SRR10188368 and SRR10188369."
- [abstract] We find that combining low coverage paired-end and long-reads is competitive in terms of performance with long-reads at higher coverage values.: "We find that combining low coverage paired-end and long-reads is competitive in terms of performance with long-reads at higher coverage values."
