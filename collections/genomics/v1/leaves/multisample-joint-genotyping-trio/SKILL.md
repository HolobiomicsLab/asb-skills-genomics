---
name: multisample-joint-genotyping-trio
description: Use when per-sample VCFs and sample_info.txt files exist for all members
  of a trio (e.g. NA12878/NA12891/NA12892) and a joint multisample VCF plus pedigree
  file is needed for downstream family-based annotation, scoring, and prioritization.
when_to_use_negative:
- Only a single sample needs processing — eDiVA-Predict alone (eDiVA-Predict.nf) suffices;
  joint calling requires all family members' calls
- Variants are already annotated and ranked (e.g. combined.variants.supplement.ranked.csv
  exists) — proceed to eDiVA-Prioritize instead of regenotyping
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: Provides the eDiVA_prepare_for_annotation.nf NextFlow workflow that performs
    the joint multisample calling of variant positions and generates pedigree.tree
  repo: https://github.com/mbosio85/ediva
- name: GATK
  role: Per-sample HaplotypeCaller variant calls (all_variants.vcf) produced upstream
    feed the joint multisample regenotyping step
- name: Nextflow
  role: Workflow engine used to launch eDiVA_prepare_for_annotation.nf with OUTF,
    FAMILY_INFO, and CPU parameters
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:7a6db1bfe00d40bbf8e0a549b474a11af55453776edeb36d9f6889c6930654d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - GATK
  - Nextflow
  - bwa-mem
  - eDiVA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multisample-joint-genotyping-trio

## Summary

Regenotypes every variant position harboring an SNV or small indel in at least one trio member to build a joint multisample VCF matrix, using eDiVA_prepare_for_annotation.nf after individual sample variant calling. This consolidated multisample matrix is the prerequisite input for eDiVA-Annotate, eDiVA-Score, and eDiVA-Prioritize.

## When to use

Use when per-sample VCFs and sample_info.txt files exist for all members of a trio (e.g. NA12878/NA12891/NA12892) and a joint multisample VCF plus pedigree file is needed for downstream family-based annotation, scoring, and prioritization.

## When NOT to use

- Only a single sample needs processing — eDiVA-Predict alone (eDiVA-Predict.nf) suffices; joint calling requires all family members' calls
- Variants are already annotated and ranked (e.g. combined.variants.supplement.ranked.csv exists) — proceed to eDiVA-Prioritize instead of regenotyping

## Inputs

- Per-sample all_variants.vcf files from eDiVA-Predict (GATK HaplotypeCaller calls)
- Concatenated sample_info.txt files (one line per sample, with correct AFFECTED column: 0 for non-affected, 1 for affected)
- Family configuration text file (FAMILY_INFO)

## Outputs

- combined.variants.supplement.vcf — multisample VCF matrix of variants for the trio
- pedigree.tree — pedigree file for downstream eDiVA-Prioritize steps

## How to apply

After eDiVA-Predict has produced all_variants.vcf and sample_info.txt for each sample (with the AFFECTED column set to 0 for parents and 1 for affected offspring), concatenate the per-sample sample_info.txt files into a single FAMILY_INFO file, one sample per line, and verify the AFFECTED column. Run the eDiVA_prepare_for_annotation.nf NextFlow workflow with --OUTF, --FAMILY_INFO, and --CPU 2; it regenotypes every position with an SNV or small indel in at least one family member to gather genotype information across all samples. The workflow emits combined.variants.supplement.vcf (multisample VCF) and pedigree.tree, which feed eDiVA-Annotate/eDiVA-Score and eDiVA-Prioritize respectively.

## Related tools

- **eDiVA** (Provides the eDiVA_prepare_for_annotation.nf NextFlow workflow that performs the joint multisample calling of variant positions and generates pedigree.tree) — https://github.com/mbosio85/ediva
- **GATK** (Per-sample HaplotypeCaller variant calls (all_variants.vcf) produced upstream feed the joint multisample regenotyping step)
- **Nextflow** (Workflow engine used to launch eDiVA_prepare_for_annotation.nf with OUTF, FAMILY_INFO, and CPU parameters) — https://www.nextflow.io/

## Examples

```
nextflow run eDiVA_prepare_for_annotation.nf --OUTF annotation/ --FAMILY_INFO family_config --CPU 2
```

## Evaluation signals

- combined.variants.supplement.vcf exists and parses as valid VCF with genotype fields for all three samples
- Multisample VCF contains genotype quality and coverage fields for NA12878, NA12891, and NA12892
- Variant count is consistent with the ~138,705 variants reported for the CEPH trio in the paper
- FAMILY_INFO text file has one line per sample with the AFFECTED column verified (0 for parents, 1 if applicable)
- pedigree.tree is emitted alongside the VCF and is consumable by eDiVA-Prioritize

## Limitations

- Joint calling is only performed after all family samples are processed by eDiVA-Predict; a single missing sample blocks Step 1.5
- Errors in the AFFECTED column of the concatenated FAMILY_INFO file propagate to the pedigree and downstream inheritance filtering
- CNV analysis is not yet available in eDiVA, so the multisample VCF covers SNVs and small indels only

## Evidence

- [readme] Once all samples from a family are processed and all variants have been called, this step takes care to perform the joint multisample calling: "Once all samples from a family are processed and all variants have been called, this step takes care to perform the joint multisample calling of vairant positions"
- [supplementary] run the eDiVA joint multisample calling step (eDiVA_prepare_for_annotation.nf) to regenotype every position: [evidence span withheld — anchor status: not_found]
- [methods] The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK best practices: "The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK (McKenna et al., 2010) best practices"
