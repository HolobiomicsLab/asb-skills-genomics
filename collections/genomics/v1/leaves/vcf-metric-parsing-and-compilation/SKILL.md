---
name: vcf-metric-parsing-and-compilation
description: Use when you have truvari bench summary outputs from multiple SV caller
  VCFs benchmarked against a truth set (e.g. GIAB CMRG SV v1.00) and need to extract
  and tabulate TP/FP/FN/precision/recall/F1 per caller for comparison.
when_to_use_negative:
- When the goal is SV calling or genotyping itself rather than summarizing already-computed
  benchmark statistics
- When the benchmarking used truvari bench+refine with consolidation (T2T assessment)
  instead of the bench-only CMRG workflow, since summary consolidation semantics differ
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: truvari
  role: produces the bench summary files whose metrics (TP/FP/FN/precision/recall/F1)
    are parsed; used at version 4.2.2
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish
  role: one of the caller VCFs whose benchmarking summary is compiled (sawfish 0.12.9)
  repo: https://github.com/PacificBiosciences/sawfish
- name: Sniffles2
  role: comparison caller VCF whose benchmarking summary is compiled
- name: pbsv
  role: comparison caller VCF whose benchmarking summary is compiled
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:6b50f2a0a981723112ea083b409c49aef4b1d7419451b3ff4d278ca2e09bc9ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Sniffles2
  - pbmm2
  - pbsv
  - sawfish
  - truvari
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# vcf-metric-parsing-and-compilation

## Summary

Parse per-caller truvari bench summary files (true positives, false positives, false negatives, precision, recall, F1) from SV benchmarking runs and compile them into a single comparative CSV table for accuracy comparison across SV callers.

## When to use

Use when you have truvari bench summary outputs from multiple SV caller VCFs benchmarked against a truth set (e.g. GIAB CMRG SV v1.00) and need to extract and tabulate TP/FP/FN/precision/recall/F1 per caller for comparison.

## When NOT to use

- When the goal is SV calling or genotyping itself rather than summarizing already-computed benchmark statistics
- When the benchmarking used truvari bench+refine with consolidation (T2T assessment) instead of the bench-only CMRG workflow, since summary consolidation semantics differ

## Inputs

- truvari bench result directories with summary. per caller (truvari_cmrg_result/summary.*)
- Query VCFs from sawfish (sawfish_joint-call_output/genotyped.sv.vcf.gz), Sniffles2, and pbsv (implicit, upstream)
- GIAB CMRG SV benchmark HG002_GRCh38_CMRG_SV_v1.00.vcf.gz (truth set context)

## Outputs

- cmrg_sv_f1_comparison.csv with per-caller TP, FP, FN, precision, recall, and F1

## How to apply

After running truvari 4.2.2 bench once per caller VCF against the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00) with --reference, --includebed, --passonly, --pick ac, and --dup-to-ins, locate each run's truvari_cmrg_result/summary.. Extract the counts of true positives, false positives, and false negatives plus the reported precision, recall, and F1 values from each summary. Compile these per-caller metrics into a single cmrg_sv_f1_comparison.csv with one row (or column set) per caller (sawfish, Sniffles2, pbsv). Validate the compilation by checking the F1 scores against expected benchmark values — sawfish 0.993 (two false negatives and one false positive against the 217-SV CMRG benchmark), Sniffles2 0.964, and pbsv 0.971.

## Related tools

- **truvari** (produces the bench summary files whose metrics (TP/FP/FN/precision/recall/F1) are parsed; used at version 4.2.2) — library://ctsa/truvari/truvari:4.2.2
- **sawfish** (one of the caller VCFs whose benchmarking summary is compiled (sawfish 0.12.9)) — https://github.com/PacificBiosciences/sawfish
- **Sniffles2** (comparison caller VCF whose benchmarking summary is compiled)
- **pbsv** (comparison caller VCF whose benchmarking summary is compiled) — https://github.com/PacificBiosciences/pbsv

## Examples

```
python parse_truvari_summaries.py --summaries truvari_cmrg_sawfish/summary. truvari_cmrg_sniffles2/summary. truvari_cmrg_pbsv/summary. --output cmrg_sv_f1_comparison.csv
```

## Evaluation signals

- Compiled sawfish F1 equals 0.993 with exactly 2 false negatives and 1 false positive against the 217-SV CMRG benchmark
- Sniffles2 F1 in the CSV equals 0.964 and pbsv F1 equals 0.971
- CSV contains one complete metric set (TP, FP, FN, precision, recall, F1) for each of the three callers
- Precision and recall values in the CSV are consistent with the TP/FP/FN counts (F1 = 2*P*R/(P+R))

## Limitations

- Validation values are specific to the sawfish 0.12.9 / Sniffles2 / pbsv comparison on the HG002 Revio HiFi reads (m84011_220902_175841_s1) against the 217-SV CMRG v1.00 benchmark; other inputs will produce different numbers
- Metrics reflect truvari 4.2.2 bench semantics with --passonly and --pick ac; summary files from other benchmark configurations are not directly comparable

## Evidence

- [methods] truvari bench CMRG assessment: "The CMRG assessment uses the truvari bench command alone"
- [methods] truvari 4.2.2 with MAFFT: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [abstract] CMRG benchmark accuracy finding: "Sawfish also shows the highest accuracy in the GIAB challenging medically relevant genes benchmark"
- [supplementary] task workflow validation: [evidence span withheld — anchor status: not_found]
- [methods] CMRG benchmark source: "The CMRG SV benchmark was obtained from the following URL: https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/CMRG_v1.00"
