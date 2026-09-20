---
name: megalodon-modified-base-calling
description: Use when an ONT R9.4.1 nanopore dataset (FAST5 + basecalled reads) needs
  a reference or comparison caller for 6mA in all contexts and 5mC in CpG context,
  e.g. benchmarking against nanoHiMe on E. coli K12 MG1655 data.
when_to_use_negative:
- When the goal is joint, phased calling of 6mA and endogenous mCpG on single molecules
  — the article shows Megalodon cannot accurately identify methylated CpG sites and
  has significantly lower 6mA accuracy than nanoHiMe on dual-modification DNA
- When input reads are not R9.4.1 ONT signal data with an eventalign-able reference,
  since the benchmark workflow requires nanopolish eventalign tables shared across
  callers
edam_operation: http://edamontology.org/operation_3900
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: Megalodon
  role: ONT-provided modified-base caller run as the baseline for 6mA/5mC benchmarking
    (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg)
- name: nanoHiMe
  role: HMM-based comparator caller whose 6mA site-classification AUCs are contrasted
    against Megalodon's across 12–100 bp site lengths
  repo: github.com/YinLabTJ/nanoHiMe
- name: Guppy
  role: Basecaller (v4.4.2, high accuracy model) producing the FASTQ and signal index
    shared by both callers
- name: Minimap2
  role: Aligns basecalled reads (v2.17, -ax map-ont, 8 threads) before filtering and
    calling
- name: nanopolish
  role: Indexes reads to FAST5 (v0.13.2) and produces the eventalign table used in
    the shared preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:3194e0a4373e25494f71bd2a92b035ca10439271100a394c6d64bfa53af2450e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3900
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# megalodon-modified-base-calling

## Summary

Runs ONT's Megalodon (v2.5.0, model res_dna_r941_min_modbases-all-context_v001.cfg) as the vendor baseline for calling 6mA and 5mC from nanopore reads, and uses its per-site likelihood scores to benchmark against nanoHiMe's HMM-based calling on E. coli 6mA site-classification AUCs.

## When to use

Use when an ONT R9.4.1 nanopore dataset (FAST5 + basecalled reads) needs a reference or comparison caller for 6mA in all contexts and 5mC in CpG context, e.g. benchmarking against nanoHiMe on E. coli K12 MG1655 data.

## When NOT to use

- When the goal is joint, phased calling of 6mA and endogenous mCpG on single molecules — the article shows Megalodon cannot accurately identify methylated CpG sites and has significantly lower 6mA accuracy than nanoHiMe on dual-modification DNA
- When input reads are not R9.4.1 ONT signal data with an eventalign-able reference, since the benchmark workflow requires nanopolish eventalign tables shared across callers

## Inputs

- Guppy v4.4.2 (high accuracy) basecalled FASTQ from E. coli K12 MG1655 FAST5 signal data, nanopolish-indexed to the FAST5 files
- minimap2 -ax map-ont alignment BAM sorted/indexed and filtered to mapped, MAPQ>=20 reads (samtools view -b -q 20 -F 4)
- E. coli K12 MG1655 reference genome (ATCC 47076) with uppercase bases and samtools faidx index

## Outputs

- Megalodon modified-base calls (6mA all contexts and 5mC CpG) with per-site scores
- Per-length, per-condition ROC AUC table (6ma_auc_vs_length.csv) and AUC-vs-site-length plot comparing Megalodon to nanoHiMe

## How to apply

Run Megalodon v2.5.0 with the res_dna_r941_min_modbases-all-context_v001.cfg model on the same Guppy-basecalled (v4.4.2, high accuracy), minimap2-aligned (map-ont), MAPQ>=20-filtered reads used for nanoHiMe, so both callers see identical eventalign-backed inputs. Sample genomic sites of 12–100 bp from 6mA-positive and 6mA-negative E. coli K12 MG1655 datasets, with and without methylated CpG context, and classify 6mA-containing windows from both callers' per-site likelihood ratios. Compute ROC AUC per site length and condition (6ma_auc_vs_length.csv) and plot AUC versus site length. Expect Megalodon to be the weaker caller on dual-modification DNA: on 40 bp sites the reported values are AUC 0.979 (no mCpG) and 0.967 (with mCpG), versus nanoHiMe's 0.98 and 0.992.

## Related tools

- **Megalodon** (ONT-provided modified-base caller run as the baseline for 6mA/5mC benchmarking (v2.5.0, res_dna_r941_min_modbases-all-context_v001.cfg))
- **nanoHiMe** (HMM-based comparator caller whose 6mA site-classification AUCs are contrasted against Megalodon's across 12–100 bp site lengths) — github.com/YinLabTJ/nanoHiMe
- **Guppy** (Basecaller (v4.4.2, high accuracy model) producing the FASTQ and signal index shared by both callers)
- **Minimap2** (Aligns basecalled reads (v2.17, -ax map-ont, 8 threads) before filtering and calling)
- **nanopolish** (Indexes reads to FAST5 (v0.13.2) and produces the eventalign table used in the shared preprocessing)

## Examples

```
megalodon test.fastq --outputs basecalls mod_basecalls --mod-model res_dna_r941_min_modbases-all-context_v001.cfg --guppy-config dna_r9.4.1_450bps_hac.cfg --reference E.coli_K12_MG1655.fasta
```

## Evaluation signals

- 40 bp site AUCs reproduce reported values: Megalodon 0.979 (no-mCpG) and 0.967 (with-mCpG), versus nanoHiMe 0.98 and 0.992
- nanoHiMe exceeds Megalodon at every site length, with Megalodon spanning 0.967–0.982 on 40–100 bp with-mCpG sites versus nanoHiMe's 0.992–0.999
- 6ma_auc_vs_length.csv contains one ROC AUC per (caller, site length 12–100 bp, mCpG-context condition)
- Megalodon run used model res_dna_r941_min_modbases-all-context_v001.cfg on the identical MAPQ>=20-filtered BAM as nanoHiMe

## Limitations

- Megalodon cannot accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites is significantly lower than nanoHiMe's on DNA containing both modifications
- Performance was not systematically evaluated for jointly calling 6mA and 5mC from a single DNA molecule prior to this benchmark
- Results are grounded in E. coli K12 MG1655 spike-in benchmarks; expect different behavior on complex eukaryotic methylomes

## Evidence

- [discussion] Megalodon cannot accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe: "Megalodon couldn’t accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe"
- [discussion] Megalodon (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg: "Megalodon (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg"
- [intro] the performance of these tools at jointly calling 6mA and 5mC from a single DNA molecule hasn't been systematically evaluated: [evidence span withheld — anchor status: not_found]
- [readme] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [intro] only the ONT-provided computational tools Megalodon and Guppy are currently available to identify 6mAs in all possible contexts and 5mCs in CpG context: "only the ONT-provided computational tools Megalodon and Guppy are currently available to identify 6mAs in all possible contexts and 5mCs in CpG context"
