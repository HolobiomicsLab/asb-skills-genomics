---
name: sv-precision-recall-benchmarking
description: Use when comparing SV-caller outputs (VCFs from dysgu, manta, delly,
  lumpy, strelka, gatk) against a reference call set (e.g. NIST_SVs_Integration_v0.6)
  to quantify per-class precision/recall/F1.
when_to_use_negative:
- Input is a single production sample call set with no truth/benchmark VCF — svbench
  requires a reference_vcf to compare against
- Evaluating small-variant (SNP/indel <30 bp) genotyping accuracy with tools like
  rtg/CHM-eval rather than SV presence/absence calling
- Benchmarking long-read SV callers where truth sets other than GIAB SV sets (e.g.
  PacBio HiFi deletion/insertion benchmark) are required
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Paired-end and long-read SV/indel caller under test ('--mode pe' preset);
    supplies query VCF to the benchmark
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Computes TP/FP/FN, precision, recall and F1 by interval-overlap matching (1000
    bp slop, 15% size similarity, PASS-only) of query VCFs vs the GIAB reference VCF
  repo: https://github.com/kcleal/svbench
- name: manta
  role: Paired-end SV caller baseline used for F1 comparison
- name: delly
  role: Paired-end SV caller baseline used for F1 comparison
- name: lumpy
  role: Paired-end SV caller baseline used for F1 comparison
- name: strelka
  role: Indel-calling baseline for variants down to 30 bp
- name: gatk
  role: Indel-calling baseline for variants down to 30 bp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:283add5d59ad0b588a94b6072788cb7bd7a232472d20f4fd7bb0587d9e0e04ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - delly
  - dysgu
  - gatk
  - lumpy
  - manta
  - strelka
  - svbench
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sv-precision-recall-benchmarking

## Summary

Benchmark structural-variant callers by computing precision, recall and F1 per SV type (deletions, insertions) against a GIAB truth set, using svbench for interval-overlap matching (1000 bp overlap, 15% size similarity). Used to rank callers such as dysgu, manta, delly and lumpy on HG002 Tier 1 regions.

## When to use

Use when comparing SV-caller outputs (VCFs from dysgu, manta, delly, lumpy, strelka, gatk) against a reference call set (e.g. NIST_SVs_Integration_v0.6) to quantify per-class precision/recall/F1.

## When NOT to use

- Input is a single production sample call set with no truth/benchmark VCF — svbench requires a reference_vcf to compare against
- Evaluating small-variant (SNP/indel <30 bp) genotyping accuracy with tools like rtg/CHM-eval rather than SV presence/absence calling
- Benchmarking long-read SV callers where truth sets other than GIAB SV sets (e.g. PacBio HiFi deletion/insertion benchmark) are required

## Inputs

- Sorted and indexed BAM/CRAM of paired-end Illumina alignments (e.g. GIAB HG002 NA24385_son NIST_HiSeq_HG002_Homogeneity-10953946)
- Indexed reference genome in FASTA format
- GIAB benchmark VCF (NIST_SVs_Integration_v0.6) as reference/truth call set
- Caller output VCFs from dysgu, manta, delly, lumpy, strelka, gatk
- BED file of high-confidence Tier 1 regions (passed to svbench --include)

## Outputs

- Per-caller, per-SV-type precision, recall and F1 table (e.g. benchmark_table.tsv)
- Optional TP/FP/FN variant lists from svbench (--tp/--fp/--fn flags)
- SV VCF per caller (e.g. svs.vcf from dysgu run)

## How to apply

Run each caller (dysgu in paired-end mode via 'dysgu --mode pe' or 'dysgu run reference.fa temp_dir input.bam', plus manta, delly, lumpy; strelka and gatk for indels down to 30 bp) on the same sorted/indexed BAM, e.g. GIAB AshkenazimTrio HG002_NA24385_son Illumina HiSeq data. Evaluate every call set with svbench against the NIST_SVs_Integration_v0.6 benchmark, restricting to PASS variants, applying minimum event sizes (30/50 bp), a 1000 bp interval-overlap criterion and 15% size similarity. Stratify results per caller for high-confidence Tier 1 regions as well as all genomic regions, and tabulate precision, recall and F1 per SV type into a table (e.g. benchmark_table.tsv). Validate by checking that the top-ranked caller's numbers are consistent with reported values such as dysgu's 95.7% deletion precision and 62.2%/23.7% recall for deletions/insertions at 20x on HG002 Tier 1.

## Related tools

- **dysgu** (Paired-end and long-read SV/indel caller under test ('--mode pe' preset); supplies query VCF to the benchmark) — https://github.com/kcleal/dysgu
- **svbench** (Computes TP/FP/FN, precision, recall and F1 by interval-overlap matching (1000 bp slop, 15% size similarity, PASS-only) of query VCFs vs the GIAB reference VCF) — https://github.com/kcleal/svbench
- **manta** (Paired-end SV caller baseline used for F1 comparison)
- **delly** (Paired-end SV caller baseline used for F1 comparison)
- **lumpy** (Paired-end SV caller baseline used for F1 comparison)
- **strelka** (Indel-calling baseline for variants down to 30 bp)
- **gatk** (Indel-calling baseline for variants down to 30 bp)

## Examples

```
dysgu run reference.fa temp_dir input.bam > svs.vcf; svbench --pass-only --include tier1.bed --min-size-query 30 NIST_SVs_Integration_v0.6.vcf dysgu.vcf manta.vcf delly.vcf lumpy.vcf
```

## Evaluation signals

- dysgu attains the highest Tier 1 F1 among paired-end callers, with F1 4.7% higher than manta for deletions and 13.1% higher for insertions at 20x
- Dysgu deletion precision = 95.7%; recall = 62.2% (deletions) and 23.7% (insertions) at 20x on HG002 Tier 1
- Dysgu calls the largest number of true deletions+insertions (n = 3913), 727 more than manta (n = 3186)
- svbench outputs should only count PASS variants and respect min-size (30 bp query) and duplicate-TP handling (--no-duplicates)
- Per-SV-type stratification: deletions and insertions scored separately; check table stratifies Tier 1 vs all genomic regions

## Limitations

- Insertion size accuracy of dysgu in PE mode is expected to be less accurate than de novo assembly-based callers such as manta and gatk (data not shown)
- No software versions for any caller are stated in the paper, complicating exact reproduction
- Analysis scripts must be taken from the svbench repository; the paper section itself omits repository URLs and accession details

## Evidence

- [results] On HG002 Tier 1 SVs at 20× coverage, dysgu showed higher F1 scores than the next best caller manta, with an F1 score 4.7% higher for deletions and 13.1% higher for insertions.: [evidence span withheld — anchor status: not_found]
- [results] Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%).: "Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%)"
- [supplementary] Svbench is also released under the MIT license and can be found at https://github.com/kcleal/svbench.: "Svbench is also released under the MIT license and can be found at https://github.com/kcleal/svbench"
- [results] We also compared indel calling performance with strelka (44) and gatk down to a size of 30 bp.: "We also compared indel calling performance with strelka (44) and gatk down to a size of 30 bp"
- [readme] Usage: svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS... --pass-only Assess only PASS variants --include PATH Include regions .bed file: [evidence span withheld — anchor status: external_doc]
