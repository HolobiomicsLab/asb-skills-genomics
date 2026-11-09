---
name: corrected-vs-raw-assembly-benchmarking
description: Use when evaluating whether haplotype-aware error correction (e.g., VeChat)
  prior to de novo assembly improves polyploid or diploid assembly quality, requiring
  a corrected-read vs. raw-read assembler comparison.
when_to_use_negative:
- The goal is read-level error-rate comparison rather than assembly-quality benchmarking;
  the article evaluates correction separately via QV/switch error rather than through
  assemblies
- 'Compute-constrained settings on large datasets: the article notes VeChat ''does
  not win the competition in terms of computational resources'' with longer runtimes
  and higher peak memory usage'
- Short-read or Illumina-only data; this benchmark targets long TGS reads (PacBio/ONT)
  assembled with haplotype-aware tools
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Produces the haplotype-aware corrected reads (reads.corrected.fa) used as
    the corrected-read arm of the benchmark
  repo: https://github.com/HaploKit/vechat
- name: HiCanu
  role: Haplotype-aware assembler used on both corrected and raw read sets with platform-appropriate
    pb/ont settings
- name: Canu
  role: Baseline assembler arm; VeChat+HiCanu is compared against raw-read Canu (92%
    to 99.6% haplotype coverage on diploid)
- name: Flye
  role: Second, non-haplotype-aware assembler used under equivalent settings on both
    read types
- name: QUAST
  role: Evaluates every assembly against the reference genome to compute haplotype
    coverage, misassemblies, and error rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:a9e5821d73138c016b8880cdf0cefe17a6cb39d29cf705a44d25b1afc56712b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Canu
  - Flye
  - HiCanu
  - QUAST
  - VeChat
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Corrected vs. raw read assembly benchmarking

## Summary

Benchmarks haplotype-aware genome assemblies built from VeChat-corrected long reads against assemblies of the matching raw reads, using (meta)Flye and HiCanu/Canu, then quantifies gains with QUAST (haplotype coverage, misassemblies, error rate).

## When to use

Use when evaluating whether haplotype-aware error correction (e.g., VeChat) prior to de novo assembly improves polyploid or diploid assembly quality, requiring a corrected-read vs. raw-read assembler comparison.

## When NOT to use

- The goal is read-level error-rate comparison rather than assembly-quality benchmarking; the article evaluates correction separately via QV/switch error rather than through assemblies
- Compute-constrained settings on large datasets: the article notes VeChat 'does not win the competition in terms of computational resources' with longer runtimes and higher peak memory usage
- Short-read or Illumina-only data; this benchmark targets long TGS reads (PacBio/ONT) assembled with haplotype-aware tools

## Inputs

- VeChat-corrected reads (reads.corrected.fa, FASTA)
- Matching raw long-read sets (FASTA/FASTQ) for diploid and tetraploid datasets
- Reference genome per dataset

## Outputs

- Assemblies from corrected and raw reads per assembler (HiCanu/Canu, Flye)
- QUAST report metrics per assembler and read type
- Summary table quast_report.tsv with haplotype coverage, misassembly counts, and error rates

## How to apply

Collect the VeChat-corrected reads (reads.corrected.fa, FASTA) together with the matching raw FASTA/FASTQ sets for the diploid and tetraploid datasets. Assemble both read types with HiCanu/Canu using the platform-appropriate setting (pb or ont) matching each dataset's sequencing technology, and with Flye under equivalent settings as a non-haplotype-aware comparison. Run QUAST on every assembly against the corresponding reference genome to compute haplotype coverage (HC), misassembly counts, and assembly error rates. Tabulate the QUAST metrics per assembler and read type and verify the reported gains: +7.6% haplotype coverage on diploid (92% to 99.6% for VeChat+HiCanu over Canu), +14% on tetraploid, fewer misassemblies, and >90x lower error rate for VeChat+HiCanu; write the summary to quast_report.tsv.

## Related tools

- **VeChat** (Produces the haplotype-aware corrected reads (reads.corrected.fa) used as the corrected-read arm of the benchmark) — https://github.com/HaploKit/vechat
- **HiCanu** (Haplotype-aware assembler used on both corrected and raw read sets with platform-appropriate pb/ont settings)
- **Canu** (Baseline assembler arm; VeChat+HiCanu is compared against raw-read Canu (92% to 99.6% haplotype coverage on diploid))
- **Flye** (Second, non-haplotype-aware assembler used under equivalent settings on both read types)
- **QUAST** (Evaluates every assembly against the reference genome to compute haplotype coverage, misassemblies, and error rates)

## Evaluation signals

- Diploid VeChat+HiCanu haplotype coverage reaches 99.6% vs. 92% for raw-read Canu (+7.6%)
- Tetraploid haplotype coverage improves by +14% over the raw-read baseline
- VeChat+HiCanu assemblies show fewer misassemblies than raw-read assemblies in the QUAST report
- VeChat+HiCanu assembly error rate is >90x lower than the raw-read comparison
- quast_report.tsv contains one row of QUAST metrics per assembler x read-type combination with no missing assemblies

## Limitations

- VeChat has higher runtimes and peak memory usage on large datasets and does not win in terms of computational resources
- Exploration of optimal preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer, or metagenomic settings is beyond the scope of the study and requires further efforts
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads; support for Supplementary Table 5 is referenced but the provided supplementary section is empty

## Evidence

- [other] Diploid HC gain finding: [evidence span withheld — anchor status: not_found]
- [other] Workflow correction output: [evidence span withheld — anchor status: not_found]
- [abstract] Assembly claim: "using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies"
- [discussion] Compute limitation: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [readme] README method summary: "VeChat distinguishes errors from haplotype-specific true variants based on variation graphs, which reflect a popular type of data structure for pangenome reference systems."
