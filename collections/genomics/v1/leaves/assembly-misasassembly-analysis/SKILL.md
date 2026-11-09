---
name: assembly-misasassembly-analysis
description: Use when comparing genome assemblies built from raw vs. corrected long
  reads (PacBio/ONT) and you need haplotype coverage, misassembly counts, or error
  rates tabulated per assembler/read-type.
when_to_use_negative:
- You only need read-level error rates (QV, switch error) — the paper evaluates those
  on reads directly (e.g. HG002, CONSENT comparison), not via assembly/QUAST.
- Input is short-read data or an assembly already polished and you only want variant
  calling — this skill targets long-read (PacBio/ONT) de novo assembly comparison.
- The goal is optimizing preprocessing/correction/assembly pipelines per setting —
  the article states this exploration 'is beyond the scope of this study and requires
  further efforts'.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Produces haplotype-aware corrected long reads (reads.corrected.fa) that are
    then assembled; correction is the upstream step whose benefit the assembly/QUAST
    analysis quantifies
  repo: https://github.com/HaploKit/vechat
- name: HiCanu
  role: Haplotype-aware assembler used to assemble both corrected and raw read sets
    with platform-appropriate setting (pb or ont)
- name: Canu
  role: Baseline assembler for raw reads, contrasted against VeChat+HiCanu haplotype
    coverage
- name: Flye
  role: Second, non-haplotype-aware assembler used under equivalent settings for comparison
- name: QUAST
  role: Evaluates every assembly against the reference genome to compute haplotype
    coverage, misassembly counts, and assembly error rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:11303fbe0368966313d8c266bc3f808399b7ad517d8e4f463a6efee37a02a8de
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
n_claims: 8
---

# assembly-misasassembly-analysis

## Summary

Assemble raw and error-corrected long reads with haplotype-aware ((Hi)Canu) and non-haplotype-aware (Flye) assemblers, then evaluate assemblies with QUAST against a reference genome to quantify haplotype coverage, misassembly counts, and assembly error rates, as done to benchmark VeChat correction.

## When to use

Use when comparing genome assemblies built from raw vs. corrected long reads (PacBio/ONT) and you need haplotype coverage, misassembly counts, or error rates tabulated per assembler/read-type.

## When NOT to use

- You only need read-level error rates (QV, switch error) — the paper evaluates those on reads directly (e.g. HG002, CONSENT comparison), not via assembly/QUAST.
- Input is short-read data or an assembly already polished and you only want variant calling — this skill targets long-read (PacBio/ONT) de novo assembly comparison.
- The goal is optimizing preprocessing/correction/assembly pipelines per setting — the article states this exploration 'is beyond the scope of this study and requires further efforts'.

## Inputs

- VeChat-corrected reads (reads.corrected.fa, FASTA)
- Matching raw long-read sets (FASTA/FASTQ, PacBio or ONT) for diploid and tetraploid datasets
- Reference genome for each dataset (FASTA)

## Outputs

- Genome assemblies per (assembler x read-type) combination (e.g. HiCanu, Canu, Flye contigs)
- QUAST reports with haplotype coverage, misassembly counts, and error rates
- Summary table quast_report.tsv of QUAST metrics per assembler and read type

## How to apply

Collect the corrected reads (e.g. VeChat's reads.corrected.fa, FASTA) together with the matching raw FASTA/FASTQ sets for each ploidy level. Assemble both read sets with HiCanu/Canu using the platform-appropriate preset (pb or ont) matching the sequencing technology, and with Flye under equivalent settings as a second, non-haplotype-aware comparison. Run QUAST on every assembly against the corresponding reference genome to compute haplotype coverage (HC), misassembly counts, and assembly error rates. Tabulate QUAST metrics per assembler and read type into quast_report.tsv and verify expected gains — for VeChat+HiCanu, haplotype coverage +7.6% on diploid (92% to 99.6%) and +14% on tetraploid over the uncorrected assembler, fewer misassemblies, and >90x lower error rate.

## Related tools

- **VeChat** (Produces haplotype-aware corrected long reads (reads.corrected.fa) that are then assembled; correction is the upstream step whose benefit the assembly/QUAST analysis quantifies) — https://github.com/HaploKit/vechat
- **HiCanu** (Haplotype-aware assembler used to assemble both corrected and raw read sets with platform-appropriate setting (pb or ont))
- **Canu** (Baseline assembler for raw reads, contrasted against VeChat+HiCanu haplotype coverage)
- **Flye** (Second, non-haplotype-aware assembler used under equivalent settings for comparison)
- **QUAST** (Evaluates every assembly against the reference genome to compute haplotype coverage, misassembly counts, and assembly error rates)

## Examples

```
canu -p diploid -d vechat_hicanu genomeSize=4.8m maxThreads=32 useGrid=false correctedErrorRate=0.05 pacbio reads.corrected.fa && quast.py vechat_hicanu/diploid.contigs.fasta -r ref.fa -o quast_out
```

## Evaluation signals

- Diploid: VeChat+HiCanu haplotype coverage reaches 99.6% vs. 92% for Canu (+7.6%), i.e. both haplotypes reconstructed at nearly full length
- Tetraploid: haplotype coverage gain of +14% over the uncorrected assembler
- VeChat+HiCanu assemblies show fewer misassemblies than assemblies from raw or conventionally corrected reads
- VeChat+HiCanu assembly error rate is >90x lower than the comparison assemblies
- Every assembly has a QUAST row in quast_report.tsv for all (assembler x corrected/raw x dataset) combinations

## Limitations

- VeChat does not win on computational resources: on large datasets it has longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads
- Optimal combinations of preprocessing, error correction and assembly tools for polyploid, cancer, or metagenomic settings are unexplored and require further efforts

## Evidence

- [abstract] Using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies.: "using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies"
- [results] Haplotype coverage gain on diploid datasets: [evidence span withheld — anchor status: not_found]
- [results] Error rate reduction on simulated diploid/triploid/tetraploid PacBio reads: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [discussion] Computational cost caveat: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [readme] README confirms graph-based correction rationale and iterative pruning method: "nodes and edges that are due to errors are pruned from that graph by way of an iterative procedure that is based on principles from frequent itemset mining"
