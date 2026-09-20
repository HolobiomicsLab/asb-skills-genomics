---
name: hifi-read-alignment-to-reference
description: Use when raw PacBio HiFi reads (e.g., Revio .hifi_reads.bam) must be
  aligned to a GRCh38 reference as the prerequisite step for SV/CNV discovery, genotyping,
  or benchmarking workflows.
when_to_use_negative:
- Input reads are short-read (Illumina) or ONT data — pbmm2 --preset CCS is tuned
  for HiFi reads and the sawfish caller expects mapped HiFi sequencing reads
- A different downstream caller requires a non-GRCh38 reference (e.g., T2T CHM13);
  the sawfish benchmarks and expected-CN tracks (expected_cn.hg38.XY.bed) are hg38-based
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: pbmm2
  role: Align HiFi reads to the reference with --preset CCS, --sort (paper used pbmm2
    1.13.1)
- name: sawfish
  role: 'Downstream consumer: runs discover and joint-call on the pbmm2-mapped BAM
    for SV/CNV calling'
  repo: https://github.com/PacificBiosciences/sawfish
- name: reference_genomes
  role: Source of the curated human_GRCh38_no_alt_analysis_set reference used for
    all analyses
  repo: https://github.com/PacificBiosciences/reference_genomes
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:5dd830acc41697107964ece3a79114da6b9c02511746d28ae291cb67d1093030
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - MAFFT
  - Sniffles2
  - bcftools
  - pbmm2
  - pbsv
  - reference_genomes
  - sawfish
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hifi-read-alignment-to-reference

## Summary

Map PacBio HiFi sequencing reads to a curated reference genome (e.g., human_GRCh38_no_alt_analysis_set) using pbmm2 with the CCS preset, producing sorted BAMs that serve as the standardized input for downstream SV/CNV calling with sawfish, Sniffles2, or pbsv.

## When to use

Use when raw PacBio HiFi reads (e.g., Revio .hifi_reads.bam) must be aligned to a GRCh38 reference as the prerequisite step for SV/CNV discovery, genotyping, or benchmarking workflows.

## When NOT to use

- Input reads are short-read (Illumina) or ONT data — pbmm2 --preset CCS is tuned for HiFi reads and the sawfish caller expects mapped HiFi sequencing reads
- A different downstream caller requires a non-GRCh38 reference (e.g., T2T CHM13); the sawfish benchmarks and expected-CN tracks (expected_cn.hg38.XY.bed) are hg38-based

## Inputs

- PacBio HiFi reads BAM (e.g., m84011_220902_175841_s1.hifi_reads.bam from HG002 Revio WGS)
- Reference genome FASTA: human_GRCh38_no_alt_analysis_set.fasta

## Outputs

- Coordinate-sorted alignment BAM (e.g., m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam) suitable for sawfish discover, pbsv, and Sniffles2

## How to apply

Download the curated human_GRCh38_no_alt_analysis_set reference FASTA (see PacificBiosciences/reference_genomes) and the HiFi read set (e.g., HG002 Revio reads m84011_220902_175841_s1.hifi_reads.bam). Run pbmm2 align with --preset CCS (HiFi read mode), --sort for coordinate-sorted output, and optionally --sample HG002 and --unmapped to retain unmapped reads; the paper used pbmm2 1.13.1 with 16 jobs/threads. The resulting alignment BAM (e.g., m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam) is the required input for sawfish discover/joint-call and for comparator callers pbsv and Sniffles2, and is reused identically across all callers so that accuracy differences reflect calling rather than mapping. Verify alignment quality (e.g., coverage depth at the expected 32-fold level) before proceeding to SV discovery.

## Related tools

- **pbmm2** (Align HiFi reads to the reference with --preset CCS, --sort (paper used pbmm2 1.13.1))
- **sawfish** (Downstream consumer: runs discover and joint-call on the pbmm2-mapped BAM for SV/CNV calling) — https://github.com/PacificBiosciences/sawfish
- **reference_genomes** (Source of the curated human_GRCh38_no_alt_analysis_set reference used for all analyses) — https://github.com/PacificBiosciences/reference_genomes

## Examples

```
pbmm2 align -j 16 --preset CCS --sample HG002 --sort --unmapped m84011_220902_175841_s1.hifi_reads.bam human_GRCh38_no_alt_analysis_set.fasta m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam
```

## Evaluation signals

- The output BAM is coordinate-sorted and named to reflect the mapper/reference (e.g., *.pbmm2-1.13.1.GRCh38.bam)
- Mapped coverage on HG002 Revio data matches the expected 32-fold depth used in the sawfish accuracy benchmarks
- The same mapped BAM feeds all three callers (sawfish, Sniffles2, pbsv) so downstream F1 differences are attributable to calling, not mapping
- sawfish discover and joint-call run without reference-related errors on the aligned BAM, and the T2T benchmark F1 of ~0.976 is reproducible from this alignment

## Limitations

- Reads with gap-compressed identity below 0.97 or mapping quality below 5 are filtered out by the downstream sawfish scan, so poor alignments silently reduce SV evidence
- The paper's benchmarks use GRCh38 specifically; results on other references (e.g., T2T) are not established in this context

## Evidence

- [methods] pbmm2 read alignment to GRCh38 reference: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
- [methods] paper-consistent analysis versioning: "The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling."
- [methods] curated reference choice: "All analyses use the reference `human_GRCh38_no_alt_analysis_set.fasta`"
- [readme] downstream consumer scope: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
