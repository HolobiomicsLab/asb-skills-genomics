---
name: correction-pipeline-config-execution
description: Use when raw long noisy reads (Nanopore R9/R10, PacBio CLR) from a diploid
  sample must be haplotype-aware error corrected, or when benchmarking PECAT correction
  against Canu, FALCON, MECAT2, or NECAT correctors.
when_to_use_negative:
- Input is Nanopore duplex or PacBio HiFi data — PECAT achieves mediocre performance
  on these and is designed for long noisy reads (Nanopore R9/R10, PacBio CLR)
- Coverage is far below 80X and the sample is a large diploid genome — PECAT needs
  higher data coverage for high-quality phased assembly
- The region has very low heterozygosity rate — the correction method may not distinguish
  small errors from heterozygote alleles there
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: Runs the haplotype-aware correction step (pecat.pl correct cfgfile) producing
    1-correct/corrected_reads.fasta
  repo: github.com/lemene/PECAT
- name: minimap2
  role: Finds candidate overlaps between raw reads for correction (-x ava-pb for PacBio
    CLR reads)
  repo: https://github.com/lh3/minimap2
- name: racon
  role: Consensus module used within PECAT's POA-graph-based error correction (v1.4.21+
    dependency)
  repo: https://github.com/lbcb-sci/racon
- name: plgd
  role: Pipeline helper script library for cluster execution (PBS/SGE/LSF/Slurm/local)
    of the PECAT correction workflow
  repo: github.com/lemene/plgd
- name: Clair3
  role: Optional raw-read SNP caller for Nanopore reads feeding haplotype-aware correction
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:b628b280a95670b43c1939823d61853edb000c4d33514d5d57da00147eaefb44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - FALCON-Unzip
  - NECAT
  - PECAT
  - minimap2
  - plgd
  - racon
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# correction-pipeline-config-execution

## Summary

Configure and execute the haplotype-aware error-correction stage of the PECAT pipeline (pecat.pl correct cfgfile), which extracts the longest ~80X raw reads and produces corrected reads that retain heterozygote alleles for downstream diploid assembly.

## When to use

Use when raw long noisy reads (Nanopore R9/R10, PacBio CLR) from a diploid sample must be haplotype-aware error corrected, or when benchmarking PECAT correction against Canu, FALCON, MECAT2, or NECAT correctors.

## When NOT to use

- Input is Nanopore duplex or PacBio HiFi data — PECAT achieves mediocre performance on these and is designed for long noisy reads (Nanopore R9/R10, PacBio CLR)
- Coverage is far below 80X and the sample is a large diploid genome — PECAT needs higher data coverage for high-quality phased assembly
- The region has very low heterozygosity rate — the correction method may not distinguish small errors from heterozygote alleles there

## Inputs

- Raw long reads (FASTA/FASTQ) from Nanopore R9, Nanopore R10, or PacBio CLR sequencing of a diploid sample at ≥80X coverage preferred
- PECAT config file (cfgfile) with genome_size set for the target dataset, templated from demo/configs
- Optional raw-read SNP calls from Clair3 for Nanopore data

## Outputs

- Corrected reads FASTA at 1-correct/corrected_reads.fasta
- Longest-80X raw-read subset used as the correction input
- Corrected read sets from comparator correctors (Canu, FALCON, MECAT2, NECAT) for benchmarking tables

## How to apply

Clone the PECAT repository recursively (git clone --recursive https://github.com/lemene/PECAT.git) or use the pre-built Docker image hub.docker.com/r/lemene/pecat, and satisfy dependencies (python3 3.6+, minimap2 2.17+, racon v1.4.21+, perl 5.22.1+, samtools 1.7+; Clair3 v0.1-r12+ and medaka 1.7.2+ optional). Prepare a per-dataset config file from the demo/configs templates, setting the correct genome_size for the dataset (e.g., the seven trio-classified diploid datasets S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0 / Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman / Bison×Simmental, and HG002). Execute the correction step with pecat.pl correct cfgfile; PECAT and NECAT extract the longest 80X raw reads (or all reads if coverage is below 80X) for error correction, and corrected reads are written to 1-correct/corrected_reads.fasta. Correction follows a POA-graph-based haplotype-aware method that keeps most heterozygote alleles while fixing sequencing errors — this retention is the rationale, since high error rates (> haplotype divergence) otherwise cause heterozygotes to be corrected away. Cluster-level orchestration is provided by the plgd helper library (PBS, SGE, LSF, Slurm, or local execution). For benchmarking, run the correction modules of Canu, FALCON, MECAT2, and NECAT on the same raw inputs so that k-mer consistency/completeness comparisons are apples-to-apples.

## Related tools

- **PECAT** (Runs the haplotype-aware correction step (pecat.pl correct cfgfile) producing 1-correct/corrected_reads.fasta) — github.com/lemene/PECAT
- **minimap2** (Finds candidate overlaps between raw reads for correction (-x ava-pb for PacBio CLR reads)) — https://github.com/lh3/minimap2
- **racon** (Consensus module used within PECAT's POA-graph-based error correction (v1.4.21+ dependency)) — https://github.com/lbcb-sci/racon
- **plgd** (Pipeline helper script library for cluster execution (PBS/SGE/LSF/Slurm/local) of the PECAT correction workflow) — github.com/lemene/plgd
- **Clair3** (Optional raw-read SNP caller for Nanopore reads feeding haplotype-aware correction) — https://github.com/HKU-BAL/Clair3

## Examples

```
pecat.pl correct cfgfile
```

## Evaluation signals

- Corrected reads are present at 1-correct/corrected_reads.fasta after pecat.pl correct completes
- The longest 80X raw reads (or all reads if dataset <80X) were used as the correction input
- On the seven trio-classified diploid datasets, PECAT-corrected reads reach haplotype-specific k-mer consistency ≥99.4% (parent-specific k-mers occurring ≥4 times in the 40X longest reads)
- Corrected reads retain heterozygote (parent-specific) k-mers — completeness measured as the percentage of parent-specific k-mers retained in the 40X longest corrected reads
- Dependency version constraints satisfied: python3 3.6+, minimap2 2.17+, racon v1.4.21+, perl 5.22.1+, samtools 1.7+

## Limitations

- SNP calling accuracy is low in low heterozygosity rate regions
- Error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions
- PECAT needs higher data coverage to obtain high-quality phased assembly
- Parameter details are documented only in Supplementary Notes 4 and 5, not the main text

## Evidence

- [readme] PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [discussion] PECAT and NECAT extract the longest 80X raw reads: "For each of the above data, PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X, for error correction and assembly."
- [methods] git clone --recursive https://github.com/lemene/PECAT.git: "git clone --recursive https://github.com/lemene/PECAT.git"
- [methods] There is a pre-built docker image: "There is a pre-built [docker image](https://hub.docker.com/r/lemene/pecat)."
- [discussion] we develop a haplotype-aware error-correct method to keep most of the heterozygote alleles: "we first develop a haplotype-aware error-correct method to keep most of the heterozygote alleles while correcting sequencing errors"
- [discussion] we find candidate overlaps between raw reads using minimap2: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [discussion] The PECAT error correction method is based on the partial-order alignment (POA) graph method: "The PECAT error correction method is based on the partial-order alignment (POA) graph method4,47."
