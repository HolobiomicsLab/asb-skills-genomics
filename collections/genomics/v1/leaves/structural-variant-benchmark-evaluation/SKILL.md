---
name: structural-variant-benchmark-evaluation
description: Use when comparing SV caller accuracy on HiFi WGS callsets against a
  GIAB SV truth set and you need an independent second evaluator (hap-eval) to confirm
  rankings previously established with Truvari.
when_to_use_negative:
- The truth set is the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which
  per the article is assessed with 'truvari bench' alone rather than the hap-eval
  T2T workflow
- The assessment must include sex chromosomes chrX/chrY, since the confident-region
  BED is deliberately restricted to autosomes with 'grep -v chr[XY]'
- The callset is not a structural-variant VCF from a long-read HiFi caller (e.g.,
  short-read variant VCFs or read-level data)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sentieon/hap-eval
  role: Independent open-source VCF comparison engine run against the T2T benchmark
    to produce precision, recall, and F1 per caller as a second method beyond Truvari
  repo: https://github.com/Sentieon/hap-eval
- name: bcftools
  role: 'Preprocessing of the T2T benchmark VCF: restrict to structural-variant records
    (INFO/SVTYPE!=".") and tabix index'
- name: sawfish
  role: Caller under evaluation; provides the genotyped.sv.vcf.gz 32x HG002 callset
    whose F1 (0.977) is the reference for margins
  repo: https://github.com/PacificBiosciences/sawfish
- name: PacificBiosciences/pbsv
  role: Competing SV caller whose 32x HG002 callset F1 deficit (4.5% vs sawfish) is
    reproduced under hap-eval
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:d5d9f3ff65f348d9c9b196a3b48627770d0a8dea886229ace229d6f0cdad84ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Sentieon/hap-eval
  tools:
  - PacificBiosciences/pbsv
  - Sentieon/hap-eval
  - bcftools
  - sawfish
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# structural-variant-benchmark-evaluation

## Summary

Independently benchmark long-read SV caller outputs (sawfish, Sniffles2, pbsv) against the GIAB draft T2T-HG002-Q100 SV benchmark using Sentieon hap-eval as a second assessment method beyond Truvari, extracting per-caller precision, recall, and F1 to confirm accuracy rankings cross-method.

## When to use

Use when comparing SV caller accuracy on HiFi WGS callsets against a GIAB SV truth set and you need an independent second evaluator (hap-eval) to confirm rankings previously established with Truvari.

## When NOT to use

- The truth set is the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which per the article is assessed with 'truvari bench' alone rather than the hap-eval T2T workflow
- The assessment must include sex chromosomes chrX/chrY, since the confident-region BED is deliberately restricted to autosomes with 'grep -v chr[XY]'
- The callset is not a structural-variant VCF from a long-read HiFi caller (e.g., short-read variant VCFs or read-level data)

## Inputs

- 32x HG002 SV callset VCFs: sawfish genotyped.sv.vcf.gz, Sniffles2 output, pbsv output
- GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar VCF)
- T2T SV benchmark confident-region BED
- human_GRCh38_no_alt_analysis_set.fasta reference genome

## Outputs

- hapeval_t2t_32x_f1_scores.csv with columns caller, precision, recall, F1, margin vs sawfish
- hap-eval per-caller precision/recall/F1 metrics confirming the sawfish F1 of 0.977 and deficits of 3.7% (Sniffles2) and 4.5% (pbsv)

## How to apply

Collect the three 32x HG002 SV callset VCFs (sawfish genotyped.sv.vcf.gz, Sniffles2, pbsv), the GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar), its confident-region BED, and the human_GRCh38_no_alt_analysis_set reference. Preprocess the benchmark with bcftools: select SV records only via 'bcftools view -i INFO/SVTYPE!="."', index with 'bcftools index -t', and restrict the confident-region BED to autosomes with 'grep -v chr[XY]'. Run Sentieon hap-eval genome-wide per callset against the filtered T2T benchmark VCF and autosome-restricted BED using the GRCh38 reference. Parse hap-eval results to extract precision, recall, and F1 per caller, compute sawfish-vs-Sniffles2 and sawfish-vs-pbsv F1 margins, and write the comparison table to hapeval_t2t_32x_f1_scores.csv.

## Related tools

- **Sentieon/hap-eval** (Independent open-source VCF comparison engine run against the T2T benchmark to produce precision, recall, and F1 per caller as a second method beyond Truvari) — https://github.com/Sentieon/hap-eval
- **bcftools** (Preprocessing of the T2T benchmark VCF: restrict to structural-variant records (INFO/SVTYPE!=".") and tabix index)
- **sawfish** (Caller under evaluation; provides the genotyped.sv.vcf.gz 32x HG002 callset whose F1 (0.977) is the reference for margins) — https://github.com/PacificBiosciences/sawfish
- **PacificBiosciences/pbsv** (Competing SV caller whose 32x HG002 callset F1 deficit (4.5% vs sawfish) is reproduced under hap-eval) — https://github.com/PacificBiosciences/pbsv

## Examples

```
hap_eval -r human_GRCh38_no_alt_analysis_set.fasta -b GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz -c sawfish_genotyped.sv.vcf.gz -i confident_regions.autosome.bed
```

## Evaluation signals

- The sawfish F1 from hap-eval must match the reported 0.977 within tolerance
- The Sniffles2 and pbsv F1 deficits must reproduce the reported 3.7% and 4.5% margins within tolerance, confirming cross-method consistency with Truvari
- The benchmark VCF must contain only SVTYPE records: verify no entry has INFO/SVTYPE=="." after bcftools filtering, and that the output is tabix-indexed
- The confident-region BED must exclude chrX and chrY before hap-eval is run
- The output CSV must contain rows for all three callers with non-null precision, recall, and F1 values

## Limitations

- hap-eval is pre-release software under active development; if it fails on a VCF, the README directs users to file an issue
- The T2T assessment excludes sex chromosomes because the confident regions are restricted to autosomes
- The benchmark is a draft GIAB release (V0.019-20241113), so absolute metrics are against a provisional truth set

## Evidence

- [methods] bcftools filtering of T2T benchmark VCF to SVTYPE records: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [methods] Autosome restriction of confident regions: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [methods] T2T benchmark version and source: "the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly"
- [readme] hap-eval tool purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] Accuracy ranking finding being cross-validated: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
