---
name: f1-score-cross-method-comparison
description: Use when accuracy rankings of SV callers (e.g. sawfish vs Sniffles2 vs
  pbsv on 32x HG002 HiFi WGS) must be confirmed with an independent evaluator instead
  of only Truvari 4.2.2, via precision/recall/F1 against a GIAB T2T SV benchmark.
when_to_use_negative:
- When the goal is per-call, genotype- or breakpoint-level refinement of individual
  SV records (use the truvari bench/refine workflow with MAFFT instead of a genome-wide
  F1 comparison)
- When the callsets or benchmark are not structural variants — this skill presumes
  SVTYPE-bearing records selected via 'bcftools view -i INFO/SVTYPE!="."' against
  an SV benchmark
- When the benchmark truth set is not GIAB SV with confident-region BED support; hap-eval
  is pre-release software and may not work with arbitrary VCF inputs
edam_operation: http://edamontology.org/operation_3199
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sentieon hap-eval
  role: Independent VCF comparison engine producing genome-wide precision/recall/F1
    for each SV callset against the T2T benchmark
  repo: https://github.com/Sentieon/hap-eval
- name: bcftools
  role: 'Preprocessing the T2T benchmark: SVTYPE record selection (''bcftools view
    -i INFO/SVTYPE!="."'') and tabix indexing (''bcftools index -t'')'
- name: sawfish
  role: Caller under evaluation; its genotyped.sv.vcf.gz 32x HG002 callset is compared
    against the benchmark
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Comparator SV caller whose 32x HG002 output provides the pbsv F1 baseline
    (deficit ~4.5% vs sawfish)
  repo: https://github.com/PacificBiosciences/pbsv
- name: Sniffles2
  role: Comparator SV caller whose 32x HG002 output provides the Sniffles2 F1 baseline
    (deficit ~3.7% vs sawfish)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:7e2af93b8f2d53b6722ae0239d6d579b27641803d37cf243a60605619fde3e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Sentieon/hap-eval
  tools:
  - PacificBiosciences/pbsv
  - Sentieon hap-eval
  - Sniffles2
  - bcftools
  - pbsv
  - sawfish
  edam_operation: http://edamontology.org/operation_3199
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# f1-score-cross-method-comparison

## Summary

Independent second-method assessment of long-read SV caller accuracy: run Sentieon hap-eval genome-wide on sawfish, Sniffles2, and pbsv 32x HG002 callsets against the GIAB draft T2T SV benchmark (V0.019-20241113), comparing precision/recall/F1 to confirm Truvari-based rankings.

## When to use

Use when accuracy rankings of SV callers (e.g. sawfish vs Sniffles2 vs pbsv on 32x HG002 HiFi WGS) must be confirmed with an independent evaluator instead of only Truvari 4.2.2, via precision/recall/F1 against a GIAB T2T SV benchmark.

## When NOT to use

- When the goal is per-call, genotype- or breakpoint-level refinement of individual SV records (use the truvari bench/refine workflow with MAFFT instead of a genome-wide F1 comparison)
- When the callsets or benchmark are not structural variants — this skill presumes SVTYPE-bearing records selected via 'bcftools view -i INFO/SVTYPE!="."' against an SV benchmark
- When the benchmark truth set is not GIAB SV with confident-region BED support; hap-eval is pre-release software and may not work with arbitrary VCF inputs

## Inputs

- 32x HG002 SV callset VCFs: sawfish genotyped.sv.vcf.gz, Sniffles2 output, pbsv output
- GIAB draft T2T SV benchmark VCF V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar)
- GIAB T2T SV benchmark confident-region BED file
- human_GRCh38_no_alt_analysis_set reference FASTA

## Outputs

- Per-caller hap-eval precision, recall, and F1 scores for the 32x HG002 T2T assessment
- Comparison table hapeval_t2t_32x_f1_scores.csv with columns: caller, precision, recall, F1, margin vs sawfish

## How to apply

Collect the three 32x HG002 SV callset VCFs (sawfish genotyped.sv.vcf.gz, Sniffles2, pbsv), the GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar), its confident-region BED, and the human_GRCh38_no_alt_analysis_set reference. Preprocess the benchmark by selecting structural-variant records only with 'bcftools view -i INFO/SVTYPE!="."' followed by 'bcftools index -t', and restrict the confident-region BED to autosomes with 'grep -v chr[XY]'. Run hap-eval genome-wide for each callset using -r for the GRCh38 reference, -b for the filtered benchmark VCF, -c for the caller VCF, and -i for the autosome-restricted BED. Parse hap-eval's precision, recall and F1 per caller, then compute the sawfish-vs-Sniffles2 and sawfish-vs-pbsv F1 margins and write them to hapeval_t2t_32x_f1_scores.csv. The rationale is to test whether Truvari-based superiority (sawfish highest accuracy across SV size groups) holds under a different comparison engine; validation requires sawfish F1 ≈ 0.977 with Sniffles2 and pbsv deficits reproducing ~3.7% and ~4.5%.

## Related tools

- **Sentieon hap-eval** (Independent VCF comparison engine producing genome-wide precision/recall/F1 for each SV callset against the T2T benchmark) — https://github.com/Sentieon/hap-eval
- **bcftools** (Preprocessing the T2T benchmark: SVTYPE record selection ('bcftools view -i INFO/SVTYPE!="."') and tabix indexing ('bcftools index -t'))
- **sawfish** (Caller under evaluation; its genotyped.sv.vcf.gz 32x HG002 callset is compared against the benchmark) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Comparator SV caller whose 32x HG002 output provides the pbsv F1 baseline (deficit ~4.5% vs sawfish)) — https://github.com/PacificBiosciences/pbsv
- **Sniffles2** (Comparator SV caller whose 32x HG002 output provides the Sniffles2 F1 baseline (deficit ~3.7% vs sawfish))

## Examples

```
hap_eval -r human_GRCh38_no_alt_analysis_set.fasta -b GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz -c sawfish.genotyped.sv.vcf.gz -i T2T_confident_regions.autosomes.bed
```

## Evaluation signals

- sawfish F1 on the 32x HG002 hap-eval assessment matches the reported 0.977 within tolerance
- Sniffles2 F1 deficit vs sawfish reproduces ~3.7% and pbsv deficit ~4.5% within tolerance
- Caller ranking (sawfish > Sniffles2 > pbsv) is consistent with the Truvari 4.2.2 + MAFFT assessment, confirming cross-method consistency
- Benchmark VCF contains only SV records — every retained record has SVTYPE != "." — and the BED covers autosomes only (chr[XY] lines removed)
- hapeval_t2t_32x_f1_scores.csv is populated with precision, recall, F1 and margin columns for all three callers

## Limitations

- hap-eval is pre-release software under active development and may fail on some VCFs; failures should be reported as issues
- The T2T SV benchmark is a draft release (V0.019-20241113) from the T2T-HG002-Q100v1.1 diploid assembly, so results are provisional
- The confident-region BED is restricted to autosomes, excluding sex chromosomes from the assessment; findings apply to the 32x HG002 WGS setting tested

## Evidence

- [other] Using hap-eval, sawfish achieves an F1-score of 0.977 on the 32x HG002 assessment, exceeding Sniffles2 by 3.7% and pbsv by 4.5%: [evidence span withheld — anchor status: not_found]
- [methods] The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."': "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [methods] The confident regions are restricted to autosomes as follows: grep -v chr[XY]: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [readme] An open-source VCF comparison engine for structural variant benchmarking: [evidence span withheld — anchor status: external_doc]
- [abstract] sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group.: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
