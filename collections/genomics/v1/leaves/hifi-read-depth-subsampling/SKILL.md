---
name: hifi-read-depth-subsampling
description: 'Use when evaluating how SV caller accuracy degrades at low HiFi coverage:
  subsample a deep HiFi BAM to fixed mean depths (e.g. 10x, 15x, 30x) before mapping
  and calling variants.'
when_to_use_negative:
- Input data are short reads, ONT reads, or already low-coverage HiFi where subsampling
  cannot yield a 10-30x depth range
- The goal is a single-coverage production SV call rather than a coverage-dependence
  accuracy benchmark
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: sawfish
  role: SV/CNV caller run at each subsampled coverage level (discover then joint-call)
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps each subsampled BAM to human_GRCh38_no_alt_analysis_set.fasta with --preset
    CCS --sort
- name: truvari
  role: bench/refine/ga4gh assessment of caller VCFs against the T2T benchmark at
    each coverage level
  repo: library://ctsa/truvari/truvari:4.2.2
- name: bcftools
  role: Filters the benchmark VCF to SVTYPE records before benchmarking
- name: hifiasm
  role: Global assembly of each subsampled read set for the PAV-based comparison arm
- name: Sniffles2
  role: Competing SV caller run on each subsampled coverage level
- name: pbsv
  role: Competing SV caller run on each subsampled coverage level
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:681479ba3d27ec18dcd89e3661c92cc795c1c60aea846846146f0adf4aa863a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - MAFFT
  - PAV
  - Sniffles2
  - bcftools
  - hifiasm
  - pbmm2
  - pbsv
  - sawfish
  - singularity
  - truvari
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# hifi-read-depth-subsampling

## Summary

Deterministically subsample a HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam) to controlled mean depths (10-, 15-, 30-fold) so that SV caller accuracy can be benchmarked as a function of coverage against the GIAB T2T SV truth set.

## When to use

Use when evaluating how SV caller accuracy degrades at low HiFi coverage: subsample a deep HiFi BAM to fixed mean depths (e.g. 10x, 15x, 30x) before mapping and calling variants.

## When NOT to use

- Input data are short reads, ONT reads, or already low-coverage HiFi where subsampling cannot yield a 10-30x depth range
- The goal is a single-coverage production SV call rather than a coverage-dependence accuracy benchmark

## Inputs

- Deep-coverage HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam)
- human_GRCh38_no_alt_analysis_set.fasta reference genome
- GIAB draft T2T SV benchmark VCF (GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz)
- GIAB T2T benchmark confident-regions BED (GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed)

## Outputs

- Subsampled HiFi BAMs at 10-, 15-, and 30-fold mean coverage
- Per-coverage mapped BAMs (pbmm2 --preset CCS --sort)
- Per-caller, per-coverage SV VCFs (sawfish, Sniffles2, pbsv, hifiasm+PAV)
- Per-coverage F1-score table comparable to Supplementary Table S3 / Supplementary Fig. S2

## How to apply

Start from the deep HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam) and subsample it to 10-, 15-, and 30-fold mean coverage, keeping the subsampled sets separate so each coverage level is processed independently. Map each subsampled BAM to human_GRCh38_no_alt_analysis_set.fasta with pbmm2 1.13.1 using --preset CCS --sort, then run sawfish discover (with --threads 16) plus sawfish joint-call, Sniffles2, and pbsv on each depth level; optionally also globally assemble each subsampled set with hifiasm and call SVs from the assembly with PAV. The rationale is that lower depth tests each caller's sensitivity under reduced evidence, allowing direct F1 comparison at 10x/15x/30x against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar), filtered with bcftools to SVTYPE records and autosomal confident regions (grep -v chr[XY]). Accuracy per coverage level is then assessed with truvari 4.2.2 bench (--passonly --pick ac --dup-to-ins) refined with MAFFT, consolidated via truvari ga4gh --with-refine, and summarized with process_truvari_ga4gh_vcfs.py.

## Related tools

- **sawfish** (SV/CNV caller run at each subsampled coverage level (discover then joint-call)) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps each subsampled BAM to human_GRCh38_no_alt_analysis_set.fasta with --preset CCS --sort)
- **truvari** (bench/refine/ga4gh assessment of caller VCFs against the T2T benchmark at each coverage level) — library://ctsa/truvari/truvari:4.2.2
- **bcftools** (Filters the benchmark VCF to SVTYPE records before benchmarking)
- **hifiasm** (Global assembly of each subsampled read set for the PAV-based comparison arm)
- **Sniffles2** (Competing SV caller run on each subsampled coverage level)
- **pbsv** (Competing SV caller run on each subsampled coverage level) — https://github.com/PacificBiosciences/pbsv

## Examples

```
pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped subsampled_10x.bam human_GRCh38_no_alt_analysis_set.fasta HG002_10x.hifi.bam
```

## Evaluation signals

- F1 at 10x is highest for sawfish — over 2.1% above Sniffles2 and 7.9% above pbsv on the GIAB T2T SV benchmark
- Other callers require ~30-fold coverage to match sawfish's 15-fold F1; the generated table should reproduce this qualitative ordering
- Each subsampled BAM has the intended mean coverage (10x/15x/30x) before mapping
- Benchmark VCF retains only records with INFO/SVTYPE!="." and the benchmark BED excludes chrX and chrY
- Per-coverage truvari ga4gh consolidated results summarize without errors via process_truvari_ga4gh_vcfs.py

## Limitations

- Conclusions are tied to HG002 Revio HiFi data and the GIAB T2T-Q100 benchmark; transfer to other samples/genomes is not established by this analysis
- Very low depths below 10-fold were not tested; behavior at extreme subsampling is unknown
- Accuracy metrics depend on the specific truvari 4.2.2 bench/refine/ga4gh pipeline (MAFFT refinement, --passonly --pick ac --dup-to-ins); other evaluation schemes may yield different F1 values

## Evidence

- [abstract] sawfish maintains the highest accuracy at every tested depth level from 10- to 32-fold coverage; other callers required at least 30-fold coverage to match sawfish accuracy at 15-fold coverage: "sawfish maintains the highest accuracy at every tested depth level from 10- to 32-fold coverage, such that other callers required at least 30-fold coverage to match sawfish accuracy at 15-fold"
- [methods] The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling.: "The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling."
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] Download the example HiFi Revio sequencing reads for HG002 here: "Download the example HiFi Revio sequencing reads for HG002 here: https://downloads.pacbcloud.com/public/revio/2022Q4/HG002-rep1/m84011_220902_175841_s1.hifi_reads.bam"
