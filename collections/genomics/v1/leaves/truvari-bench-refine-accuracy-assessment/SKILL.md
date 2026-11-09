---
name: truvari-bench-refine-accuracy-assessment
description: Use when you need to reproduce published SV caller accuracy (e.g., sawfish
  F1 of 0.976 on 32x HG002 HiFi) by benchmarking query VCFs against the GIAB draft
  T2T SV benchmark with truvari.
when_to_use_negative:
- Assessing the CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which uses truvari
  bench alone without the bench/refine/ga4gh consolidation workflow
- Input callsets include non-PASS records, when the goal is to match the published
  sawfish evaluation, since --passonly filters these
- Comparing callers on datasets other than mapped HiFi germline SV VCFs against GIAB-style
  truth sets
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: truvari 4.2.2
  role: Bench, refine, and ga4gh consolidation of SV calls against the T2T benchmark;
    run via singularity image library://ctsa/truvari/truvari:4.2.2
  repo: https://github.com/PacificBiosciences/sawfish
- name: MAFFT
  role: Alignment engine used by truvari refine for sequence refinement of candidate
    regions
- name: bcftools
  role: Filter T2T benchmark VCF to SVTYPE records with bcftools view -i 'INFO/SVTYPE!="."'
- name: pbmm2 1.13.1
  role: Maps HG002 Revio HiFi reads to human_GRCh38_no_alt_analysis_set with --preset
    CCS and --sort to generate query callsets
  repo: https://github.com/PacificBiosciences/sawfish
- name: sawfish 0.12.9
  role: Produces the primary query SV callset via sawfish discover and sawfish joint-call,
    validated to reach F1 ~0.976
  repo: https://github.com/PacificBiosciences/sawfish
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:ce6cbc602539701da224ad5110890f06ef1c7e8bf1acbf01c002674d3143e167
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - MAFFT
  - bcftools
  - pbmm2
  - pbmm2 1.13.1
  - sawfish
  - sawfish 0.12.9
  - truvari
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# truvari-bench-refine-accuracy-assessment

## Summary

Benchmark long-read SV caller output against a GIAB truth set using truvari bench followed by MAFFT-based truvari refine, consolidated via truvari ga4gh, to compute F1-scores comparable to those reported for sawfish, Sniffles2, and pbsv on the GIAB T2T SV benchmark.

## When to use

Use when you need to reproduce published SV caller accuracy (e.g., sawfish F1 of 0.976 on 32x HG002 HiFi) by benchmarking query VCFs against the GIAB draft T2T SV benchmark with truvari.

## When NOT to use

- Assessing the CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which uses truvari bench alone without the bench/refine/ga4gh consolidation workflow
- Input callsets include non-PASS records, when the goal is to match the published sawfish evaluation, since --passonly filters these
- Comparing callers on datasets other than mapped HiFi germline SV VCFs against GIAB-style truth sets

## Inputs

- Query SV VCF from sawfish 0.12.9 (and optionally Sniffles2, pbsv) on 32x HG002 HiFi data
- GIAB draft T2T SV benchmark VCF V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar)
- T2T benchmark confident regions BED, restricted to autosomes
- human_GRCh38_no_alt_analysis_set.fasta reference
- Aligned BAM/BAI from pbmm2 CCS-preset mapping of HG002 Revio HiFi reads

## Outputs

- truvari bench output directories per caller with candidate.refine.bed regions
- truvari refine MAFFT-refined results and truvari ga4gh consolidation VCFs
- Per-caller F1-score summary (process_truvari_ga4gh_vcfs.py), including size-stratified accuracy statistics

## How to apply

Map HG002 Revio HiFi reads to human_GRCh38_no_alt_analysis_set with pbmm2 (--preset CCS, --sort), call SVs with sawfish 0.12.9 (discover then joint-call) plus Sniffles2 and pbsv to obtain query callsets. Preprocess the T2T benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar) with `bcftools view -i 'INFO/SVTYPE!="."'` and restrict the confident regions BED to autosomes via `grep -v chr[XY]`. Run truvari 4.2.2 (singularity image library://ctsa/truvari/truvari:4.2.2) bench for each caller with --reference, --includebed (autosomal BED), --base, --comp, --passonly, --pick ac, and --dup-to-ins, then run truvari refine with --recount --use-region-coords --use-original-vcfs --align mafft --threads 16 on candidate.refine.bed regions (keep tmp on the regular filesystem for MAFFT). Consolidate results with truvari ga4gh --with-refine and summarize per caller with process_truvari_ga4gh_vcfs.py; the sawfish size-stratified accuracy stats should yield an F1 of approximately 0.976 against the T2T benchmark, exceeding Sniffles2 by ~1.2% and pbsv by ~1.4% (Supplementary Table S1).

## Related tools

- **truvari 4.2.2** (Bench, refine, and ga4gh consolidation of SV calls against the T2T benchmark; run via singularity image library://ctsa/truvari/truvari:4.2.2) — https://github.com/PacificBiosciences/sawfish
- **MAFFT** (Alignment engine used by truvari refine for sequence refinement of candidate regions)
- **bcftools** (Filter T2T benchmark VCF to SVTYPE records with bcftools view -i 'INFO/SVTYPE!="."')
- **pbmm2 1.13.1** (Maps HG002 Revio HiFi reads to human_GRCh38_no_alt_analysis_set with --preset CCS and --sort to generate query callsets) — https://github.com/PacificBiosciences/sawfish
- **sawfish 0.12.9** (Produces the primary query SV callset via sawfish discover and sawfish joint-call, validated to reach F1 ~0.976) — https://github.com/PacificBiosciences/sawfish

## Examples

```
truvari bench --reference human_GRCh38_no_alt_analysis_set.fasta --includebed T2T_autosomal.bed --base GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz --comp sawfish.vcf.gz --passonly --pick ac --dup-to-ins -o bench_sawfish
```

## Evaluation signals

- Sawfish size-stratified accuracy stats reproduce an F1-score of approximately 0.976 against the T2T benchmark
- Sawfish F1 exceeds Sniffles2 by ~1.2% and pbsv by ~1.4%, matching Supplementary Table S1
- Benchmark VCF is filtered so only SVTYPE records remain (no '.' SVTYPE entries) and confident regions BED contains no chr[XY] lines
- truvari bench was run with --passonly --pick ac --dup-to-ins and refine completed on candidate.refine.bed with MAFFT alignment
- Bench/refine results were consolidated via truvari ga4gh --with-refine before computing summary statistics

## Limitations

- VACmap is still unsupported; truvari alignment tooling is still working towards clear error messages for problematic alignments from any source
- The refine step requires tmp on the regular filesystem for MAFFT; using non-local tmp can break refinement
- The published comparison targets the draft T2T benchmark (V0.019-20241113) on autosomes; results may not transfer to other benchmarks or sex chromosomes without revalidation

## Evidence

- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] it is important to use the bench and refine command, with the bench/refine results consolidated as described further below.: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [methods] The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."': "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
- [methods] the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly: "the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly"
- [methods] The confident regions are restricted to autosomes as follows: grep -v chr[XY]: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
