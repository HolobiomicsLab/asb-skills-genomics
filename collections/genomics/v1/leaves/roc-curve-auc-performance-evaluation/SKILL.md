---
name: roc-curve-auc-performance-evaluation
description: Use when comparing methylation- or modification-calling tools on datasets
  with enzymatic-treatment ground truth (e.g. M.SssI-methylated CpGs, Hia5-defined
  6mA), where per-site scores can be threshold-swept into ROC curves and AUCs.
when_to_use_negative:
- Input data lack enzymatic-treatment ground truth for CpG methylation status, so
  ROC/AUC cannot be computed against known labels
- The goal is calling 6mA-containing regions rather than benchmarking CpG methylation-call
  discrimination between tools
tools:
- name: nanoHiMe
  role: HMM-based mCG-calling mode produces per-read-region Log(LIKELIHOOD_RATIO)_mCG
    scores swept for ROC curves
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: v0.13.2 eventalign and independent CpG methylation caller benchmarked in the
    AUC comparison
- name: Megalodon
  role: v2.5.0 ONT methylation caller (res_dna_r941_min_modbases-all-context_v001.cfg
    model) benchmarked as the ONT-provided baseline
- name: Minimap2
  role: v2.17 alignment of basecalled reads used to build the BAMs feeding methylation
    calls
- name: samtools
  role: v1.9 filtering of mapped, mapping-quality >= 20 reads (samtools view -b -q
    20 -F 4) for downstream methylation calling
- name: Guppy
  role: v4.4.2 high-accuracy basecaller producing the FASTQ/event data upstream of
    the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:8b1c61c912251c3851267c4c7e93f0796804f31ed3aa8d251c086df575d81efd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# roc-curve-auc-performance-evaluation

## Summary

Benchmark CpG methylation-calling tools (nanoHiMe, nanopolish, Megalodon) by sweeping their per-read log-likelihood-ratio score thresholds to build ROC curves and compute AUCs against enzymatically defined ground truth on E. coli K12 MG1655 PCR-amplicon data.

## When to use

Use when comparing methylation- or modification-calling tools on datasets with enzymatic-treatment ground truth (e.g. M.SssI-methylated CpGs, Hia5-defined 6mA), where per-site scores can be threshold-swept into ROC curves and AUCs.

## When NOT to use

- Input data lack enzymatic-treatment ground truth for CpG methylation status, so ROC/AUC cannot be computed against known labels
- The goal is calling 6mA-containing regions rather than benchmarking CpG methylation-call discrimination between tools

## Inputs

- nanopolish eventalign files (test.eventalign.txt or .gz) from E. coli K12 MG1655 PCR-amplicon training datasets
- Megalodon v2.5.0 methylation calls (model res_dna_r941_min_modbases-all-context_v001.cfg)
- nanopolish v0.13.2 methylation calls
- Ground-truth CpG methylation status labels derived from enzymatic treatments (M.SssI for mCpG, Hia5 for 6mA presence/absence)

## Outputs

- Per-tool/per-condition AUC comparison table (auc_comparison_table.csv) with rows for nanoHiMe, nanopolish, and Megalodon under both the without-6mA and with-6mA conditions
- ROC curves per tool per condition obtained by threshold sweeping of methylation scores

## How to apply

Run each caller on the same eventalign-aligned nanopore reads: nanoHiMe in mCG mode (./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG), nanopolish v0.13.2, and Megalodon v2.5.0 with the res_dna_r941_min_modbases-all-context_v001.cfg model, on the four E. coli training conditions. Define ground truth from the enzymatic treatments — CpGs methylated by M.SssI, stratifying sites into those without 6mA versus those that also carry 6mA (from Hia5 treatment). For each tool, sweep the methylation score threshold (e.g. nanoHiMe's Log(LIKELIHOOD_RATIO)_mCG, where positive values support methylation and negative support unmethylation), compute ROC curves, and integrate to obtain AUCs per tool per condition. Assemble results into a per-tool/per-condition table written as auc_comparison_table.csv; reference AUCs are ~0.953/0.951/0.934 for sites without 6mA and ~0.603/0.919/0.922 for sites that also contain 6mA.

## Related tools

- **nanoHiMe** (HMM-based mCG-calling mode produces per-read-region Log(LIKELIHOOD_RATIO)_mCG scores swept for ROC curves) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (v0.13.2 eventalign and independent CpG methylation caller benchmarked in the AUC comparison)
- **Megalodon** (v2.5.0 ONT methylation caller (res_dna_r941_min_modbases-all-context_v001.cfg model) benchmarked as the ONT-provided baseline)
- **Minimap2** (v2.17 alignment of basecalled reads used to build the BAMs feeding methylation calls)
- **samtools** (v1.9 filtering of mapped, mapping-quality >= 20 reads (samtools view -b -q 20 -F 4) for downstream methylation calling)
- **Guppy** (v4.4.2 high-accuracy basecaller producing the FASTQ/event data upstream of the comparison)

## Examples

```
python -c "from sklearn.metrics import roc_auc_score; print(roc_auc_score(labels, scores))"  # labels: M.SssI CpG truth; scores: nanoHiMe Log(LIKELIHOOD_RATIO)_mCG
```

## Evaluation signals

- auc_comparison_table.csv exists, is valid CSV, and contains rows for all three tools (nanoHiMe, nanopolish, Megalodon) under both conditions (without 6mA and with 6mA)
- Reproduced AUCs fall within reasonable tolerance of reference values 0.953/0.951/0.934 (sites without 6mA) and 0.603/0.919/0.922 (sites with 6mA)
- ROC curves are monotonically non-decreasing as thresholds sweep from most-positive to most-negative Log(LIKELIHOOD_RATIO)_mCG scores
- Ground-truth labels correctly segregate M.SssI-methylated CpGs into sites without 6mA versus sites also containing 6mA per the Hia5 treatment design
- All three callers were run on the same E. coli eventalign datasets so scores are directly comparable

## Limitations

- Megalodon's CpG methylation AUC drops sharply (~0.603) on sites that also carry 6mA, showing ONT tools confound co-occurring 6mA and 5mC in dual-modification DNA
- nanoHiMe's trained parameters for k-mers with methyladenine(s) in different positions are collapsed to an averaged k-mer because partial adenine methylation prevents precise assignment
- Ground truth depends entirely on enzymatic treatment design (M.SssI, Hia5); a unified model covering other modifications (5-hmC, 5-fC, 4-mC, GpC) does not yet exist

## Evidence

- [other] workflow thresholds: [evidence span withheld — anchor status: not_found]
- [readme] mCG-calling output format: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
- [discussion] Megalodon performance gap: "Megalodon couldn’t accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower"
- [methods] read filtering: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] k-mer parameter ambiguity: "the partial adenine methylation made it impossible to precisely assign the learned parameters to k-mers"
