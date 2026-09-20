---
name: enzymatic-methyltransferase-truth-set-design
description: Use when benchmarking CpG/6mA methylation callers on nanopore reads and
  you need experimentally defined methylated/unmethylated truth sites from enzymatic
  treatments (M.SssI, Hia5) rather than bisulfite or model calls.
when_to_use_negative:
- The data are native human/other genomes without enzymatic spike-in treatments, so
  no modification-status ground truth can be defined from treatments
- The goal is detecting arbitrary base modifications (e.g. 5-hmC, 5-fC, 4-mC, GpC
  methylation) — the authors note a unified calling model for these combinations does
  not yet exist
- Reads cannot be aligned to a known reference (eventalign calls require a reference
  FASTA and nanopolish event alignment)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: Hia5 methyltransferase
  role: Non-specific adenine methyltransferase used enzymatically to introduce 6mA,
    defining which sites contain 6mA (truth-set condition)
- name: M.SssI methyltransferase
  role: CpG-specific methyltransferase treatment used to define methylated CpG ground-truth
    status in E. coli datasets
- name: Guppy
  role: Basecalling of the training FAST5 files (v4.4.2, high accuracy model, dna_r9.4.1_450bps_hac.cfg)
    prior to event alignment
- name: nanopolish
  role: v0.13.2 event alignment producing test.eventalign.txt for truth-set reads;
    also benchmarked as an mCpG caller
- name: Megalodon
  role: v2.5.0 (model res_dna_r941_min_modbases-all-context_v001.cfg) benchmarked
    against the enzymatic truth sets
- name: nanoHiMe
  role: HMM-based mCG/6mA caller benchmarked against enzymatic truth sets (./nanoHiMe
    mCG ref_upper.fa test.eventalign.txt output.mCG)
  repo: github.com/YinLabTJ/nanoHiMe
- name: Minimap2
  role: v2.17 alignment of basecalled reads to the reference genome before event alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:20130d8540586351eea0b6ac0b81bff85f6e544c07d41930e5ccb334b9a9f7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Hia5 methyltransferase
  - M.SssI methyltransferase
  - Megalodon
  - Minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# enzymatic-methyltransferase-truth-set-design

## Summary

Design enzymatic-methylation ground-truth datasets (M.SssI-methylated CpGs, Hia5-derived 6mA) on E. coli K12 MG1655 PCR-amplicon nanopore reads so that methylation callers such as nanoHiMe, nanopolish v0.13.2 and Megalodon v2.5.0 can be benchmarked by threshold-swept AUC.

## When to use

Use when benchmarking CpG/6mA methylation callers on nanopore reads and you need experimentally defined methylated/unmethylated truth sites from enzymatic treatments (M.SssI, Hia5) rather than bisulfite or model calls.

## When NOT to use

- The data are native human/other genomes without enzymatic spike-in treatments, so no modification-status ground truth can be defined from treatments
- The goal is detecting arbitrary base modifications (e.g. 5-hmC, 5-fC, 4-mC, GpC methylation) — the authors note a unified calling model for these combinations does not yet exist
- Reads cannot be aligned to a known reference (eventalign calls require a reference FASTA and nanopolish event alignment)

## Inputs

- E. coli K12 MG1655 (ATCC 47076) genomic DNA or PCR-amplicon nanopore FAST5 datasets from M.SssI and Hia5 enzymatic treatments
- Reference genome FASTA (E.coli_K12_MG1655.fasta, upper-cased and faidx-indexed)
- Guppy v4.4.2 basecalled FASTQ and Minimap2 v2.17 alignments (BAM)
- nanopolish v0.13.2 eventalign files (test.eventalign.txt(.gz))

## Outputs

- Per-read, per-region CpG methylation log-likelihood ratios from each caller (e.g. output.mCG.eventalign.txt)
- Ground-truth site annotations: M.SssI-methylated CpGs; sites without 6mA vs sites with 6mA (Hia5 condition)
- ROC curves and per-tool/per-condition AUC table (auc_comparison_table.csv)

## How to apply

Prepare PCR-amplicon or genomic DNA from E. coli K12 MG1655 (ATCC 47076) under controlled enzymatic treatments: M.SssI to methylate all CpGs, and Hia5 treatment variants to separate sites without 6mA from sites that also contain 6mA. Sequence on MinION R9.4.1 flowcells, basecall with Guppy v4.4.2 (high-accuracy model), align with Minimap2 v2.17, and produce nanopolish v0.13.2 eventalign files (test.eventalign.txt). Define per-site ground-truth methylation status directly from the enzymatic treatments — CpGs are methylated where M.SssI was applied, and 6mA presence/absence is set by the Hia5 condition. Sweep each caller's log-likelihood-ratio/threshold to build ROC curves per condition and integrate to obtain AUCs (reference: sites without 6mA ~0.953 nanoHiMe / 0.951 nanopolish / 0.934 Megalodon; sites with 6mA ~0.603 / 0.919 / 0.922). The rationale is that enzymatic treatments give unambiguous modification labels that neither ONT-supplied tools nor bisulfite-based references provide.

## Related tools

- **Hia5 methyltransferase** (Non-specific adenine methyltransferase used enzymatically to introduce 6mA, defining which sites contain 6mA (truth-set condition))
- **M.SssI methyltransferase** (CpG-specific methyltransferase treatment used to define methylated CpG ground-truth status in E. coli datasets)
- **Guppy** (Basecalling of the training FAST5 files (v4.4.2, high accuracy model, dna_r9.4.1_450bps_hac.cfg) prior to event alignment)
- **nanopolish** (v0.13.2 event alignment producing test.eventalign.txt for truth-set reads; also benchmarked as an mCpG caller)
- **Megalodon** (v2.5.0 (model res_dna_r941_min_modbases-all-context_v001.cfg) benchmarked against the enzymatic truth sets)
- **nanoHiMe** (HMM-based mCG/6mA caller benchmarked against enzymatic truth sets (./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG)) — github.com/YinLabTJ/nanoHiMe
- **Minimap2** (v2.17 alignment of basecalled reads to the reference genome before event alignment)

## Examples

```
perl nanoHiMe/perl_script/upper.pl E.coli_K12_MG1655.fasta > ref_upper.fa; samtools faidx ref_upper.fa; ./nanoHiMe mCG ref_upper.fa test.eventalign.txt.gz output.mCG
```

## Evaluation signals

- auc_comparison_table.csv exists in CSV format and contains rows for all three tools (nanoHiMe, nanopolish, Megalodon) under both conditions (sites without 6mA and sites with 6mA)
- Reproduced AUCs fall within a reasonable tolerance of references: ~0.953/0.951/0.934 (no-6mA) and ~0.603/0.919/0.922 (with 6mA)
- Ground-truth labels match the enzymatic treatments: all CpGs methylated in M.SssI-treated samples; 6mA presence confined to Hia5-treated conditions
- Event alignment was produced with mapping-quality filtering (samtools view -q 20 -F 4) and reads indexed to FAST5 signal data via nanopolish index

## Limitations

- On sites that also contain 6mA, ONT-supplied Megalodon confounds dual-modification calling: Megalodon couldn't accurately identify methylated CpG sites and its 6mA accuracy was significantly lower than nanoHiMe's, so truth sets containing both marks are harsh benchmarks for joint callers
- Assignment of trained parameters to k-mers with methyladenine(s) in different positions is imprecise; such k-mers are collapsed to an averaged k-mer (reproducibility gap)
- A unified methylation-calling model covering 5-hmC, 5-fC, 4-mC and GpC methylation combinations does not yet exist and is deferred to future work

## Evidence

- [discussion] Hia5 truth-set design (discussion): "nanoHiMe-seq used Hia5, a non-specific methyltransferase with higher enzymatic efficiency, to label adenine bases proximal to H3K27me3 or H3K4me3 nucleosomes"
- [discussion] Dual-modification caller weakness (discussion): "Megalodon couldn’t accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe"
- [readme] Reference genome for truth-set analysis (readme): "#ref.fa is the reference genome fasta file, such as hg19.fa and E.coli_K12_MG1655.fasta"
- [readme] Basecalling configuration for training data (readme): "guppy_basecaller --input_path fast5 --save_path output_fast5 -c dna_r9.4.1_450bps_hac.cfg --qscore_filtering -x cuda:0,1"
- [discussion] Source organism for truth-set DNA (discussion): "Genomic DNA from HepG2 cells and from E. coli K12 MG1655 (ATCC, 47076)"
