---
name: long-read-structural-variant-calling
description: Use when calling and genotyping germline structural variants from mapped
  PacBio HiFi reads (e.g. HG002 Revio data), or when benchmarking SV caller accuracy
  (F1, precision, recall) against GIAB T2T or CMRG benchmarks.
when_to_use_negative:
- Input reads are short-read or noisy long-read data rather than high-quality mapped
  HiFi reads, since sawfish is a joint SV/CNV caller designed for mapped HiFi sequencing
  reads
- The analysis requires somatic or de-novo SV detection — sawfish discovers germline
  structural variants and jointly genotypes across samples
- Only a quick rough callset is needed without benchmarking; if no truth set exists,
  the truvari bench/refine/ga4gh assessment portion does not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: 'Primary SV/CNV caller: discover step scans alignments for breakpoint evidence
    via local assembly, joint-call step merges and genotypes SVs and calls CNVs'
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HiFi reads to the GRCh38 reference with --preset CCS --sort prior to
    SV calling
- name: truvari
  role: 'Benchmarking engine: bench (with --passonly --pick ac --dup-to-ins), refine
    with MAFFT, and ga4gh consolidation of T2T/CMRG assessments'
  repo: https://doi.org/10.5281/zenodo.14898462
- name: MAFFT
  role: Alignment engine used by truvari refine for refinement of candidate regions
- name: bcftools
  role: Filters the T2T benchmark VCF to structural-variant records with 'bcftools
    view -i INFO/SVTYPE!="."'
- name: pbsv
  role: Comparison SV caller run on the same mapped BAM to produce benchmark query
    callsets
  repo: https://github.com/PacificBiosciences/pbsv
- name: Sniffles2
  role: Comparison SV caller run on the same mapped BAM to produce benchmark query
    callsets
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:5d329d5363e3a41e8885924946b136d1d5ac0dfaf5950e7424429e613673c30b
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# long-read-structural-variant-calling

## Summary

A benchmarked workflow for germline SV discovery and genotyping from mapped HiFi reads using sawfish's local haplotype modeling, with accuracy assessed against GIAB T2T and CMRG truth sets using truvari bench/refine/ga4gh.

## When to use

Use when calling and genotyping germline structural variants from mapped PacBio HiFi reads (e.g. HG002 Revio data), or when benchmarking SV caller accuracy (F1, precision, recall) against GIAB T2T or CMRG benchmarks.

## When NOT to use

- Input reads are short-read or noisy long-read data rather than high-quality mapped HiFi reads, since sawfish is a joint SV/CNV caller designed for mapped HiFi sequencing reads
- The analysis requires somatic or de-novo SV detection — sawfish discovers germline structural variants and jointly genotypes across samples
- Only a quick rough callset is needed without benchmarking; if no truth set exists, the truvari bench/refine/ga4gh assessment portion does not apply

## Inputs

- Mapped HiFi BAM (e.g. pbmm2-aligned HG002 Revio reads, m84011_220902_175841_s1.hifi_reads.bam)
- Reference FASTA human_GRCh38_no_alt_analysis_set.fasta
- GIAB T2T SV benchmark VCF GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz with confident-region BED (filtered to SVTYPE records and autosomes)
- GIAB CMRG SV benchmark HG002_GRCh38_CMRG_SV_v1.00.vcf.gz/.tbi/.bed
- expected_cn.hg38.XY.bed copy-number track and annotation_and_common_cnv.hg38.bed.gz CNV excluded-regions track (for sawfish joint-call)

## Outputs

- Per-sample discover output and joint-genotyped SV/CNV VCF (sawfish_joint-call_output/genotyped.sv.vcf.gz)
- truvari bench/refine result directories with candidate.refine.bed and per-caller summary files
- Consolidated ga4gh VCFs and per-caller F1/precision/recall tables (e.g. cmrg_sv_f1_comparison.csv, Supplementary Table S1/S3/S4-style summaries)

## How to apply

Map HiFi reads (e.g. m84011_220902_175841_s1.hifi_reads.bam) to human_GRCh38_no_alt_analysis_set.fasta with pbmm2 1.13.1 using --preset CCS --sample HG002 --sort. Run 'sawfish discover --threads 16' once per sample, then 'sawfish joint-call --threads 16' to merge, genotype SVs and call CNVs, producing genotyped.sv.vcf.gz; optionally run Sniffles2 and pbsv for comparison. Prepare truth sets by filtering the GIAB T2T benchmark with 'bcftools view -i INFO/SVTYPE!="."' and restricting the confident-region BED to autosomes with 'grep -v chr[XY]'. Assess with truvari 4.2.2 bench (--passonly --pick ac --dup-to-ins --includebed --reference), then truvari refine with MAFFT (--recount --use-region-coords --use-original-vcfs --align mafft), consolidate with 'truvari ga4gh --with-refine', and summarize with process_truvari_ga4gh_vcfs.py; the CMRG assessment uses truvari bench alone. Expected results: sawfish F1 ≈ 0.976 on the T2T benchmark at 32x (1.2% above Sniffles2, 1.4% above pbsv) and F1 = 0.993 with 2 FN and 1 FP on the 217-SV CMRG benchmark.

## Related tools

- **sawfish** (Primary SV/CNV caller: discover step scans alignments for breakpoint evidence via local assembly, joint-call step merges and genotypes SVs and calls CNVs) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HiFi reads to the GRCh38 reference with --preset CCS --sort prior to SV calling)
- **truvari** (Benchmarking engine: bench (with --passonly --pick ac --dup-to-ins), refine with MAFFT, and ga4gh consolidation of T2T/CMRG assessments) — https://doi.org/10.5281/zenodo.14898462
- **MAFFT** (Alignment engine used by truvari refine for refinement of candidate regions)
- **bcftools** (Filters the T2T benchmark VCF to structural-variant records with 'bcftools view -i INFO/SVTYPE!="."')
- **pbsv** (Comparison SV caller run on the same mapped BAM to produce benchmark query callsets) — https://github.com/PacificBiosciences/pbsv
- **Sniffles2** (Comparison SV caller run on the same mapped BAM to produce benchmark query callsets)

## Examples

```
pbmm2 align -j 16 --preset CCS --sample HG002 --sort m84011_220902_175841_s1.hifi_reads.bam human_GRCh38_no_alt_analysis_set.fasta m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam && sawfish discover --threads 16 -b m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam -o sawfish_discover_output
```

## Evaluation signals

- Sawfish T2T benchmark F1 at 32x HG002 HiFi reproduces approximately 0.976, ~1.2% above Sniffles2 and ~1.4% above pbsv (Supplementary Table S1)
- CMRG assessment reproduces sawfish F1 = 0.993 with two false negatives and one false positive against the 217-SV benchmark, versus Sniffles2 0.964 and pbsv 0.971 (Supplementary Table S4)
- Downscaling check: at 10x coverage sawfish F1 exceeds Sniffles2 by >2.1% and pbsv by >7.9%, and other callers require ≥30x to match sawfish at 15x
- Benchmark VCF preprocessing invariant: only records with INFO/SVTYPE != '.' remain, and the confident-region BED contains no chrX/chrY entries
- All truvari comparisons use PASS-only calls (--passonly --pick ac --dup-to-ins) so per-caller summaries are directly comparable

## Limitations

- Accuracy figures are tied to specific benchmark versions (GIAB draft T2T SV V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar; CMRG v1.00) and HG002 Revio 32x data; other inputs may not reproduce them
- MAFFT in truvari refine requires tmp placed on a regular filesystem, and the workflow depends on the specific truvari 4.2.2 singularity image (library://ctsa/truvari/truvari:4.2.2)
- Sawfish read filtering drops reads with gap-compressed identity below 0.97 or mapping quality below 5, and single-read breakpoint clusters are discarded as noise, so very low-support or low-identity events may be missed

## Evidence

- [other] Assessed with Truvari against the GIAB T2T SV benchmark, sawfish achieves an F1-score of 0.976 on 32-fold HG002 HiFi data, which is 1.2% higher than Sniffles2 and 1.4% higher than pbsv: [evidence span withheld — anchor status: not_found]
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [readme] Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads.: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
- [intro] All breakpoint-based structural variants are modeled and genotyped as local haplotypes, yielding substantial accuracy gains on modern SV truth sets: "All breakpoint-based structural variants are modeled and genotyped as local haplotypes, yielding substantial accuracy gains on modern SV truth sets"
- [abstract] sawfish maintains the highest accuracy at every tested depth level from 10- to 32-fold coverage, such that other callers required at least 30-fold coverage to match sawfish accuracy at 15-fold: "sawfish maintains the highest accuracy at every tested depth level from 10- to 32-fold coverage"
