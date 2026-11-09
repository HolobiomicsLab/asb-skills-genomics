---
name: hap-eval-sv-callset-scoring
description: Use when validating SV caller accuracy rankings (e.g. sawfish vs Sniffles2
  vs pbsv) on HiFi WGS callsets with a second, independent benchmarking method instead
  of only Truvari.
when_to_use_negative:
- The callsets are short-read or non-HiFi SV calls outside the 20 bp–100 kb SV size
  range where the compared callers operate
- A Truvari-based assessment is already sufficient and no independent cross-method
  validation is required
- The benchmark VCF contains no SVTYPE-annotated records and cannot be filtered to
  structural variants
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Sentieon/hap-eval
  role: Independent VCF comparison engine that scores each SV callset against the
    GIAB T2T benchmark to produce precision, recall, and F1
  repo: https://github.com/Sentieon/hap-eval
- name: bcftools
  role: 'Preprocesses the benchmark: selects SV records via ''bcftools view -i INFO/SVTYPE!="."''
    and indexes with ''bcftools index -t'''
- name: sawfish
  role: Caller under evaluation; supplies the genotyped.sv.vcf.gz callset being scored
  repo: https://github.com/PacificBiosciences/sawfish
- name: PacificBiosciences/pbsv
  role: Comparison caller whose SV callset F1 is contrasted against sawfish
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:201bc3e49986fd2fec8dfd762e5a66feb1184902d0e9c036e49fb9d7775203fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Sentieon/hap-eval
  tools:
  - PacificBiosciences/pbsv
  - Sentieon/hap-eval
  - bcftools
  - sawfish
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# hap-eval-sv-callset-scoring

## Summary

Scores long-read SV caller outputs against a GIAB structural variant benchmark using Sentieon hap-eval, an independent VCF comparison engine, to compute precision, recall, and F1 as a cross-check on Truvari-based assessments.

## When to use

Use when validating SV caller accuracy rankings (e.g. sawfish vs Sniffles2 vs pbsv) on HiFi WGS callsets with a second, independent benchmarking method instead of only Truvari.

## When NOT to use

- The callsets are short-read or non-HiFi SV calls outside the 20 bp–100 kb SV size range where the compared callers operate
- A Truvari-based assessment is already sufficient and no independent cross-method validation is required
- The benchmark VCF contains no SVTYPE-annotated records and cannot be filtered to structural variants

## Inputs

- sawfish genotyped.sv.vcf.gz callset (32x HG002)
- Sniffles2 SV callset VCF (32x HG002)
- pbsv SV callset VCF (32x HG002)
- GIAB draft T2T SV benchmark VCF V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar)
- T2T benchmark confident-region BED file
- human_GRCh38_no_alt_analysis_set.fasta reference

## Outputs

- hapeval_t2t_32x_f1_scores.csv with caller, precision, recall, F1, and margin vs sawfish
- per-caller precision, recall, and F1 scores from hap-eval
- sawfish-vs-Sniffles2 and sawfish-vs-pbsv F1 margins

## How to apply

Collect the three 32x HG002 SV callset VCFs (sawfish genotyped.sv.vcf.gz, Sniffles2, pbsv), the GIAB draft T2T SV benchmark (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar), its confident-region BED, and the human_GRCh38_no_alt_analysis_set reference. Preprocess the benchmark with bcftools: restrict to structural-variant records via 'bcftools view -i INFO/SVTYPE!="."' and index with 'bcftools index -t'; restrict the confident-region BED to autosomes with 'grep -v chr[XY]'. Run hap-eval genome-wide on each callset against the filtered benchmark VCF and autosome BED using the GRCh38 reference FASTA. Parse hap-eval output for precision, recall and F1 per caller, then compute the sawfish-vs-Sniffles2 and sawfish-vs-pbsv F1 margins and write the comparison table to hapeval_t2t_32x_f1_scores.csv.

## Related tools

- **Sentieon/hap-eval** (Independent VCF comparison engine that scores each SV callset against the GIAB T2T benchmark to produce precision, recall, and F1) — https://github.com/Sentieon/hap-eval
- **bcftools** (Preprocesses the benchmark: selects SV records via 'bcftools view -i INFO/SVTYPE!="."' and indexes with 'bcftools index -t')
- **sawfish** (Caller under evaluation; supplies the genotyped.sv.vcf.gz callset being scored) — https://github.com/PacificBiosciences/sawfish
- **PacificBiosciences/pbsv** (Comparison caller whose SV callset F1 is contrasted against sawfish) — https://github.com/PacificBiosciences/pbsv

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz; bcftools index -t GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz; hap_eval -r human_GRCh38_no_alt_analysis_set.fasta -b GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz -c sawfish_genotyped.sv.vcf.gz -i benchmark_regions.autosome.bed
```

## Evaluation signals

- sawfish F1 must match the reported 0.977 on the 32x HG002 assessment within tolerance
- Sniffles2 F1 deficit relative to sawfish must reproduce the reported 3.7% margin within tolerance
- pbsv F1 deficit relative to sawfish must reproduce the reported 4.5% margin within tolerance
- The filtered benchmark VCF must contain only records passing INFO/SVTYPE!="." and be bgzip-indexed
- Cross-method consistency: hap-eval must confirm the sawfish superiority seen with Truvari

## Limitations

- hap-eval is pre-release software under active development and may not work with all VCFs
- Conclusions are grounded in the GIAB draft T2T SV benchmark (V0.019-20241113), which is a draft release
- Confident regions are restricted to autosomes (chrX/chrY excluded), so sex-chromosome SVs are unassessed

## Evidence

- [methods] The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."': "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
- [readme] An open-source VCF comparison engine for structural variant benchmarking: [evidence span withheld — anchor status: external_doc]
- [other] Using hap-eval, sawfish achieves an F1-score of 0.977 on the 32x HG002 assessment, exceeding Sniffles2 by 3.7% and pbsv by 4.5%: [evidence span withheld — anchor status: not_found]
- [methods] the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly: "the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly"
