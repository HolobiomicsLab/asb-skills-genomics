---
name: multi-caller-variant-call-comparison
description: Use when comparing SNV caller performance on trio nanopore data, e.g.
  replicating Mendelian Consistency Rate benchmarks across miniSNV, Clair3, and Longshot
  VCFs on GIAB sup-datasets.
when_to_use_negative:
- Input is raw ONT signal/FASTQ rather than called variant VCFs — run the callers
  (miniSNV, Clair3, Longshot) first via their own pipelines
- The datasets are not a parent–child trio (e.g. single-sample HG002 k12-data with
  Guppy basecalls) — Mendelian Consistency Rate cannot be computed without parental
  genotypes
- Evaluating indels or structural variants — this skill is restricted to biallelic
  SNV genotype comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3572
tools:
- name: miniSNV
  role: Primary ONT SNV caller under evaluation; provides trio VCFs for MCR comparison
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparison caller combining pileup and full-alignment; run with ONT-trained
    rerio clair3_models
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Diploid SNV comparison caller using a pair hidden Markov model
- name: WhatsHap
  role: Used within miniSNV to phase heterozygotes and haplotag variants, enabling
    haplotype-aware genotyping feeding the trio calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:a427c13ab8cfccb4114e5a06246e1da0ad8c39884f18873d54948e0425ca2121
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Longshot
  - WhatsHap
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-caller-variant-call-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple SNV callers (miniSNV, Clair3, Longshot) on the same ONT long-read datasets by regenerating per-caller variant calls and computing Mendelian Consistency Rate on the GIAB Ashkenazi Trio (HG002/HG003/HG004) to assess caller accuracy beyond F1-score.

## When to use

Use when comparing SNV caller performance on trio nanopore data, e.g. replicating Mendelian Consistency Rate benchmarks across miniSNV, Clair3, and Longshot VCFs on GIAB sup-datasets.

## When NOT to use

- Input is raw ONT signal/FASTQ rather than called variant VCFs — run the callers (miniSNV, Clair3, Longshot) first via their own pipelines
- The datasets are not a parent–child trio (e.g. single-sample HG002 k12-data with Guppy basecalls) — Mendelian Consistency Rate cannot be computed without parental genotypes
- Evaluating indels or structural variants — this skill is restricted to biallelic SNV genotype comparison

## Inputs

- Per-caller trio VCF files (miniSNV, Clair3, Longshot) for HG002, HG003, HG004 from GIAB Ashkenazi Trio ONT sup-datasets
- GIAB Ashkenazi Trio ONT data (kit v14 chemistry, R10.4.1 flowcells, ~200 Gbases per sample, Dorado basecalled)
- Clair3 model from nanoporetech/rerio clair3_models (e.g. r1041_e82_400bps_sup for Dorado SUP basecalls)

## Outputs

- Per-caller Mendelian Consistency Rate values in mcr_table.csv
- Per-child-SNV Mendelian-consistency classifications (biallelic SNV genotypes vs. parental genotypes)

## How to apply

Start from the per-caller trio VCF files for HG002, HG003, and HG004 produced by miniSNV, Clair3, and Longshot on the GIAB Ashkenazi Trio sup-datasets (PromethION, ligation kit v14, R10.4.1 flowcells, Dorado basecalled). Parse each VCF to extract biallelic SNV genotypes (0/0, 0/1, 1/1) for the child and both parents per caller, then apply Mendelian inheritance rules to classify each child SNV as Mendelian-consistent or not against the parental genotypes. Compute the Mendelian Consistency Rate (MCR) per caller as the fraction of child SNVs consistent with the parental genotypes, and write the per-caller values to mcr_table.csv. Validate by checking that miniSNV and Longshot MCRs exceed Clair3's by roughly 3 percentage points, consistent with the paper's reported trio benchmark. Note that Clair3 was run with the latest ONT-trained model from nanoporetech/rerio clair3_models, and that rerio Clair3 models are compatible with Clair3 v1.x but not v2.x.

## Related tools

- **miniSNV** (Primary ONT SNV caller under evaluation; provides trio VCFs for MCR comparison) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparison caller combining pileup and full-alignment; run with ONT-trained rerio clair3_models) — https://github.com/nanoporetech/rerio
- **Longshot** (Diploid SNV comparison caller using a pair hidden Markov model)
- **WhatsHap** (Used within miniSNV to phase heterozygotes and haplotag variants, enabling haplotype-aware genotyping feeding the trio calls)

## Examples

```
python compute_mcr.py --trio HG002 HG003 HG004 --callers miniSNV Clair3 Longshot --vcf-dir sup_dataset_vcfs/ --out mcr_table.csv
```

## Evaluation signals

- Regenerated mcr_table.csv shows miniSNV and Longshot MCRs exceeding Clair3's by approximately 3 percentage points on the HG002/HG003/HG004 sup-dataset trio
- All child SNVs are classified into exactly one of Mendelian-consistent or Mendelian-inconsistent per caller, with MCR = consistent fraction per caller
- Only biallelic SNV genotypes from {0/0, 0/1, 1/1} enter the comparison; multiallelic and indel records are excluded
- miniSNV trio calls should also reproduce F1-score and recall > 99.7% on HG002, HG003, and HG004 sup-datasets as reported in the paper

## Limitations

- MCR depends on trio pedigree assumptions and only covers biallelic SNVs; complex regions excluded by GIAB v3.3 stratifications (difficult-to-map and low-complexity regions) may affect comparability
- rerio Clair3 models are compatible with Clair3 v1.x but not v2.x, so model/basecaller version mismatches can change comparison results
- Research-release Dorado/rerio models 'have not been scrutinized nor validated' by Oxford Nanopore, adding uncertainty to caller comparisons built on them

## Evidence

- [results] On the sup-dataset trio, both miniSNV and Longshot achieve higher Mendelian Consistency Rates than Clair3 by approximately 3%.: [evidence span withheld — anchor status: not_found]
- [readme] rerio Clair3 model version compatibility caveat: [evidence span withheld — anchor status: external_doc]
