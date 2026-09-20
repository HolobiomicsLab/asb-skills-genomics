---
name: cmrg-benchmark-accuracy-comparison-across-callers
description: Use when you need to quantify and compare SV calling accuracy of multiple
  long-read callers against the GIAB HG002 CMRG SV truth set on HiFi data.
when_to_use_negative:
- Benchmarking against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar),
  which requires truvari bench plus refine with consolidation rather than bench alone
- Evaluating somatic or CNV-only calling rather than germline SV calls on HiFi data
- The caller VCFs are not all generated from the same mapped BAM/alignment, making
  the comparison invalid
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: sawfish 0.12.9
  role: query SV caller under benchmark (discover + joint-call)
  repo: https://github.com/PacificBiosciences/sawfish
- name: Sniffles2
  role: comparison SV caller producing a query VCF
- name: pbsv
  role: comparison SV caller producing a query VCF
  repo: https://github.com/PacificBiosciences/pbsv
- name: pbmm2 1.13.1
  role: aligns HiFi reads to GRCh38 before calling
- name: truvari 4.2.2
  role: bench-based comparison of query VCFs against the CMRG truth set
  repo: library://ctsa/truvari/truvari:4.2.2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:43d4a5a42bb782bb18b29b4b008821abdafeb27b03bd1a6f2833033cafd84f14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - Sniffles2
  - pbmm2
  - pbmm2 1.13.1
  - pbsv
  - sawfish 0.12.9
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cmrg-benchmark-accuracy-comparison-across-callers

## Summary

Benchmarks single-sample SV caller accuracy (sawfish vs Sniffles2 vs pbsv) against the GIAB challenging medically relevant genes (CMRG) benchmark of 217 SVs using truvari 4.2.2, yielding precision/recall/F1 per caller (sawfish F1 0.993, Sniffles2 0.964, pbsv 0.971).

## When to use

Use when you need to quantify and compare SV calling accuracy of multiple long-read callers against the GIAB HG002 CMRG SV truth set on HiFi data.

## When NOT to use

- Benchmarking against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar), which requires truvari bench plus refine with consolidation rather than bench alone
- Evaluating somatic or CNV-only calling rather than germline SV calls on HiFi data
- The caller VCFs are not all generated from the same mapped BAM/alignment, making the comparison invalid

## Inputs

- HG002 Revio HiFi reads BAM (m84011_220902_175841_s1.hifi_reads.bam)
- human_GRCh38_no_alt_analysis_set.fasta reference
- GIAB CMRG SV benchmark HG002_GRCh38_CMRG_SV_v1.00.vcf.gz with .tbi and .bed
- Query VCFs from sawfish 0.12.9, Sniffles2, and pbsv on the same mapped BAM

## Outputs

- truvari_cmrg_result/summary per caller (truvari bench output with TP, FP, FN, precision, recall, F1)
- cmrg_sv_f1_comparison.csv compiling per-caller benchmark metrics

## How to apply

Align HG002 Revio HiFi reads (m84011_220902_175841_s1.hifi_reads.bam) to human_GRCh38_no_alt_analysis_set with pbmm2 1.13.1 using --preset CCS --sample HG002 --sort. Call SVs on the same mapped BAM with sawfish 0.12.9 (sawfish discover then sawfish joint-call), Sniffles2, and pbsv to produce comparable query VCFs. Download HG002_GRCh38_CMRG_SV_v1.00.vcf.gz (.tbi, .bed) from the GIAB FTP CMRG_v1.00 release directory. Run truvari 4.2.2 bench per caller VCF against the CMRG benchmark with --reference, --includebed, --passonly, --pick ac, and --dup-to-ins (e.g. via the library://ctsa/truvari/truvari:4.2.2 singularity image), writing results to truvari_cmrg_result/summary per caller. Parse each truvari summary for true/false positives and negatives, precision, recall, and F1, and compile per-caller metrics into cmrg_sv_f1_comparison.csv. Unlike the T2T SV assessment, the CMRG assessment uses the truvari bench command alone, without refine/consolidation.

## Related tools

- **sawfish 0.12.9** (query SV caller under benchmark (discover + joint-call)) — https://github.com/PacificBiosciences/sawfish
- **Sniffles2** (comparison SV caller producing a query VCF)
- **pbsv** (comparison SV caller producing a query VCF) — https://github.com/PacificBiosciences/pbsv
- **pbmm2 1.13.1** (aligns HiFi reads to GRCh38 before calling)
- **truvari 4.2.2** (bench-based comparison of query VCFs against the CMRG truth set) — library://ctsa/truvari/truvari:4.2.2

## Examples

```
singularity run library://ctsa/truvari/truvari:4.2.2 bench -b HG002_GRCh38_CMRG_SV_v1.00.vcf.gz -c sawfish_joint-call_output/genotyped.sv.vcf.gz -f human_GRCh38_no_alt_analysis_set.fasta --includebed CMRG.bed --passonly --pick ac --dup-to-ins -o truvari_cmrg_result/sawfish/summary
```

## Evaluation signals

- Compiled F1 scores match Supplementary Table S4: sawfish 0.993 (2 FN, 1 FP against the 217-SV CMRG benchmark), Sniffles2 0.964, pbsv 0.971
- Each truvari bench run uses --reference, --includebed, --passonly, --pick ac, and --dup-to-ins consistently across all callers
- All three caller VCFs derive from the identical pbmm2-aligned BAM (m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam)
- Truth set files checksum/version is HG002_GRCh38_CMRG_SV_v1.00 from the GIAB CMRG_v1.00 release

## Limitations

- CMRG benchmark contains only 217 SVs, so single-call differences (e.g. 2 FN + 1 FP) strongly shift F1
- Only PASS calls (--passonly) are assessed, which may penalize callers with different FILTER conventions
- Assessment depends on truvari 4.2.2 with --pick ac and --dup-to-ins matching choices; other settings can change results

## Evidence

- [results] On the GIAB challenging medically relevant genes benchmark, sawfish achieved the highest F1-score of 0.993 with only two false negatives and one false positive, versus 0.964 for Sniffles and 0.971 for pbsv.: [evidence span withheld — anchor status: not_found]
- [methods] The CMRG SV benchmark was obtained from the GIAB release directory for HG002 CMRG_v1.00.: "The CMRG SV benchmark was obtained from the following URL: https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/CMRG_v1.00"
- [methods] The CMRG assessment uses the truvari bench command alone.: "The CMRG assessment uses the truvari bench command alone"
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling.: "The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling."
