---
name: cross-technology-callset-merging
description: Use when you have independently generated dysgu SV callsets (VCF) from
  paired-end and long-read data and want to quantify how many additional deletions/insertions
  the combined callset recovers against a GIAB benchmark.
when_to_use_negative:
- The callsets were not produced by dysgu (its 'merge' command and model-probability
  field are required for the >=0.5 / >=0.35 filtering step)
- You need to resolve multi-allelic sites or full-length insertion sequences — the
  article notes dysgu lacks a de novo assembly stage for these
- The analysis targets somatic samples with copy-number changes, where a non-diploid
  dysgu model rather than merged germline callsets is appropriate
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: dysgu
  role: SV caller applied independently to each downsampled read set; its 'merge'
    command combines callsets across runs and its model probability field supports
    thresholded filtering
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: benchmarks merged and single-technology callsets against the GIAB HG002 truth
    set to compute per-SV-type precision and recall
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:47f041acf6d0872d6bf61b502ce3332ea1e790c52ecf25b4befbafae4a19c84f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - svbench
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-technology-callset-merging

## Summary

Merge structural variant calls from Illumina paired-end data into dysgu callsets from low-coverage PacBio/ONT long reads using dysgu's 'merge' command, then filter with technology-specific model-probability thresholds (>=0.5 PacBio, >=0.35 ONT) to boost recall toward GIAB HG002 truth sets.

## When to use

Use when you have independently generated dysgu SV callsets (VCF) from paired-end and long-read data and want to quantify how many additional deletions/insertions the combined callset recovers against a GIAB benchmark.

## When NOT to use

- The callsets were not produced by dysgu (its 'merge' command and model-probability field are required for the >=0.5 / >=0.35 filtering step)
- You need to resolve multi-allelic sites or full-length insertion sequences — the article notes dysgu lacks a de novo assembly stage for these
- The analysis targets somatic samples with copy-number changes, where a non-diploid dysgu model rather than merged germline callsets is appropriate

## Inputs

- dysgu SV callsets in VCF format from downsampled Illumina paired-end data (e.g. NIST_HiSeq_HG002_Homogeneity-10953946)
- dysgu SV callsets in VCF format from downsampled PacBio long reads (SRR10188368, SRR10188369) or ONT data
- GIAB HG002 truth set VCF with Tier 1 and All-regions confidence stratifications (BED)
- indexed reference genome FASTA

## Outputs

- merged PacBio+PE and ONT+PE SV callsets (VCF) filtered by maximum model probability
- benchmark table of precision/recall per SV type (e.g. table7_merged_lr_pe_benchmark.tsv) showing recall gains: +1286 deletions/+894 insertions (All regions), +150 deletions/+33 insertions (Tier 1)

## How to apply

Run dysgu independently on downsampled paired-end (e.g. NIST_HiSeq_HG002_Homogeneity-10953946) and long-read (e.g. SRR10188368/SRR10188369 PacBio) datasets to produce per-technology VCF callsets. Merge the paired-end calls into the long-read callsets with dysgu's 'merge' command, which supports merging of SVs from different runs. Filter merged calls with maximum model probability >=0.5 for PacBio combinations or >=0.35 for ONT combinations. Benchmark merged versus single-technology callsets against the GIAB HG002 truth set with svbench, computing precision and recall per SV type (deletions, insertions) for All-regions and Tier 1 stratifications. The rationale is that dysgu's gradient-boosting probability score lets low-coverage PE calls complement LR calls, making combined low-coverage PE+LR competitive with long reads at higher coverage.

## Related tools

- **dysgu** (SV caller applied independently to each downsampled read set; its 'merge' command combines callsets across runs and its model probability field supports thresholded filtering) — https://github.com/kcleal/dysgu
- **svbench** (benchmarks merged and single-technology callsets against the GIAB HG002 truth set to compute per-SV-type precision and recall) — https://github.com/kcleal/svbench

## Examples

```
dysgu merge --names pacbio8x illumina40x --max_probs 0.5 pacbio8x.vcf illumina40x.vcf > merged.vcf && svbench GIAB_HG002.vcf merged.vcf --pass-only
```

## Evaluation signals

- Merged PacBio+PE and ONT+PE VCFs exist and contain only calls passing the model-probability thresholds (>=0.5 PacBio, >=0.35 ONT)
- Benchmark against GIAB HG002 reproduces the reported recall-gain pattern: additional deletions and insertions for the merged callset versus LR alone (e.g. +1286 DEL, +894 INS for All regions)
- Tier 1 stratified recall gains (e.g. +150 DEL, +33 INS) are smaller than All-region gains, matching the published Table 7 pattern
- Combined low-coverage PE+LR performance is competitive with higher-coverage long-read-only callsets in precision/recall
- svbench runs use consistent matching parameters (e.g. --slop 250 default) across all compared callsets

## Limitations

- Dysgu relies on several heuristics such as span position distance and minimizer support that are unoptimized and could be tuned with a larger test set
- Multi-allelic sites and full-length insertion sequences in paired-end data remain unresolved because dysgu does not incorporate a de novo assembly stage
- Model-probability thresholds are technology-specific (0.5 PacBio vs 0.35 ONT); applying the wrong threshold shifts the precision-recall balance
- Temporary 'fetch' files are large (>5 Gb) and a 30X genome takes ~1 h on a single core, so low-coverage downsampling helps but storage must be planned

## Evidence

- [results] Merging SVs from different runs using dysgu's 'merge' command: "Dysgu supports merging of SVs from different runs using a ‘merge’ command"
- [results] Filter thresholds per technology: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [abstract] Core finding motivating the skill: "We find that combining low coverage paired-end and long-reads is competitive in terms of performance with long-reads at higher coverage values."
- [readme] Merged call command availability: [evidence span withheld — anchor status: external_doc]
- [readme] Post-merge filtering recommendation: [evidence span withheld — anchor status: external_doc]
