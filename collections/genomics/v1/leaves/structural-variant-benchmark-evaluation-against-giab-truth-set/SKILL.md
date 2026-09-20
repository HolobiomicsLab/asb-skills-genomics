---
name: structural-variant-benchmark-evaluation-against-giab-truth-set
description: Use when SV calls in VCF format (single-technology or merged callsets
  from dysgu 'merge') must be evaluated against the GIAB HG002 benchmark, reporting
  precision/recall per SV type for All-regions vs Tier 1 regions.
when_to_use_negative:
- Truth set is not GIAB HG002-compatible (e.g. syndip/CHM1-CHM13 assemblies call for
  a different benchmarking setup)
- Calls have not been merged/filtered yet and the question is single-caller performance,
  not recall gains from merging
- Small variant (<30 bp indel/SNV) calling accuracy is the target; svbench defaults
  apply a --min-size-query of 30 suited to SVs ≥50 bp
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Calls SVs from PE or LR data per technology and merges callsets from different
    runs via the 'merge' command; supports probability-based filtering of merged calls
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Benchmarks query VCFs against the reference GIAB truth set VCF, with --include
    region BEDs, --pass-only, --slop, and per-type precision/recall output
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:68a820b896ecfc58dcbaafa53a53586f1b676b9eb5909ab0d437683caa5699c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - svbench
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# structural-variant-benchmark-evaluation-against-giab-truth-set

## Summary

Benchmark structural variant callsets (e.g. dysgu VCFs from paired-end Illumina, PacBio, or ONT reads, or merged PE+LR callsets) against the Genome in a Bottle HG002 Ashkenazi son truth set using svbench, computing precision and recall per SV type to quantify recall gains from call merging.

## When to use

Use when SV calls in VCF format (single-technology or merged callsets from dysgu 'merge') must be evaluated against the GIAB HG002 benchmark, reporting precision/recall per SV type for All-regions vs Tier 1 regions.

## When NOT to use

- Truth set is not GIAB HG002-compatible (e.g. syndip/CHM1-CHM13 assemblies call for a different benchmarking setup)
- Calls have not been merged/filtered yet and the question is single-caller performance, not recall gains from merging
- Small variant (<30 bp indel/SNV) calling accuracy is the target; svbench defaults apply a --min-size-query of 30 suited to SVs ≥50 bp

## Inputs

- dysgu SV callsets in VCF format from downsampled (low-coverage) Illumina PE, PacBio, or ONT data
- Merged PE+LR VCFs produced by dysgu merge (filtered at max model probability ≥0.5 PacBio / ≥0.35 ONT)
- GIAB HG002 structural variant truth set (Tier 1 and All-regions benchmarks)
- Region BED file(s) specifying benchmark regions for svbench --include

## Outputs

- Per-SV-type precision and recall values for each callset (single-technology vs merged) from svbench
- Table 7 comparison (table7_merged_lr_pe_benchmark.tsv) showing recall gains from adding PE calls to LR callsets

## How to apply

Run dysgu independently on each downsampled dataset (e.g. GIAB HG002 Illumina NIST_HiSeq_Homogeneity-10953946 PE data and PacBio SRR10188368/SRR10188369 long reads), then merge the Illumina paired-end calls into the long-read callsets with dysgu's 'merge' command. Filter merged calls with a maximum model probability ≥0.5 for PacBio combinations or ≥0.35 for ONT combinations before benchmarking. Benchmark merged and single-technology callsets against the GIAB HG002 truth set with svbench, using --include for the region BED (All regions vs Tier 1) and stratifying precision and recall by SV type (deletions, insertions). Correctness is judged by reproducing the reported pattern: adding dysgu PE calls to 8× PacBio calls detected an additional 1286 deletions and 894 insertions genome-wide and 150 additional deletions and 33 insertions for Tier 1, with combined low-coverage PE+LR competitive with higher-coverage long reads.

## Related tools

- **dysgu** (Calls SVs from PE or LR data per technology and merges callsets from different runs via the 'merge' command; supports probability-based filtering of merged calls) — https://github.com/kcleal/dysgu
- **svbench** (Benchmarks query VCFs against the reference GIAB truth set VCF, with --include region BEDs, --pass-only, --slop, and per-type precision/recall output) — https://github.com/kcleal/svbench

## Examples

```
svbench --include tier1.bed --pass-only --slop 250 giab_hg002_truth.vcf merged_pacbio_pe.filtered.vcf
```

## Evaluation signals

- Merged PacBio+PE and ONT+PE VCFs exist and contain calls surviving the probability filters (≥0.5 PacBio, ≥0.35 ONT)
- Merged 40×-PE + 8×-PacBio All-regions callset shows ~+1286 deletions and +894 insertions over the LR-only callset; Tier 1 shows ~+150 deletions and +33 insertions
- svbench reports both precision and recall per SV type (deletions, insertions) for All-regions and Tier 1 strata
- Combined low-coverage PE+LR performance approaches long-read performance at higher coverage, matching the abstract's claimed pattern

## Limitations

- Filter thresholds differ by read technology (≥0.5 PacBio vs ≥0.35 ONT), so PacBio thresholds cannot be applied to ONT-merged callsets
- PE-based dysgu does not resolve full-length insertion sequences: 'no de novo assembly stage incorporated; multi-allelic sites and full-length insertions in PE data remain unresolved'
- Dysgu's insertion sizes in PE mode may be less accurate than assembly-based callers (noted as 'data not shown')
- No software versions for dysgu or svbench are stated in the article, complicating exact reproduction

## Evidence

- [results] Merging SVs from different runs using dysgu's 'merge' command: "Dysgu supports merging of SVs from different runs using a ‘merge’ command"
- [results] Probability filtering thresholds for merged callsets: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [abstract] Recall gain finding from merging PE calls into LR callsets: "We find that combining low coverage paired-end and long-reads is competitive in terms of performance with long-reads at higher coverage values."
- [discussion] GIAB HG002 truth set and region strata: "Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions."
- [readme] svbench CLI usage for reference-vs-query benchmarking with region inclusion: [evidence span withheld — anchor status: not_found]
- [supplementary] PacBio and Illumina input data accessions: "Two lanes of PacBio data were downloaded from SRA (https://www.ncbi.nlm.nih.gov/sra) under accessions SRR10188368 and SRR10188369."
