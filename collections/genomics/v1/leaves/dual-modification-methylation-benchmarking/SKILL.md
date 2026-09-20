---
name: dual-modification-methylation-benchmarking
description: Use when comparing methylation-calling tools on nanopore reads containing
  co-occurring base modifications, or when reproducing the nanoHiMe/nanopolish/Megalodon
  CpG AUC benchmark on enzymatically treated E. coli data.
when_to_use_negative:
- Input DNA carries only a single modification type (no co-occurring 6mA) and the
  goal is absolute accuracy, not tool comparison
- Data are from organisms where ground-truth methylation status cannot be established
  enzymatically (no M.SssI/Hia5-defined truth set)
- The task is joint histone-modification profiling (nanoHiMe-seq wet-lab assay), not
  computational methylation-caller benchmarking
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: nanoHiMe
  role: HMM-based CpG methylation calling producing per-read-region log-likelihood
    ratios from eventalign files
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Baseline CpG methylation caller (v0.13.2); also produces the eventalign files
    consumed by nanoHiMe
- name: Megalodon
  role: ONT baseline methylation caller (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg
- name: Minimap2
  role: Aligns basecalled ONT reads to the reference genome (v2.17, default settings)
    prior to event alignment
- name: samtools
  role: Sorts BAM, indexes references, and filters mapped reads at mapping quality
    >= 20 (samtools view -b -q 20 -F 4)
- name: Guppy
  role: Basecalling of FAST5 with the high-accuracy model (v4.4.2, dna_r9.4.1_450bps_hac.cfg)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:f6f5858b1f4beb3892173b17089f3de043afd980426d4d44e4011871963b54ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dual-modification-methylation-benchmarking

## Summary

Benchmark CpG methylation-calling performance of nanoHiMe, nanopolish v0.13.2, and Megalodon v2.5.0 on E. coli K12 MG1655 PCR-amplicon nanopore datasets, comparing sites without 6mA versus sites that also carry 6mA, using AUC from threshold-swept ROC curves.

## When to use

Use when comparing methylation-calling tools on nanopore reads containing co-occurring base modifications, or when reproducing the nanoHiMe/nanopolish/Megalodon CpG AUC benchmark on enzymatically treated E. coli data.

## When NOT to use

- Input DNA carries only a single modification type (no co-occurring 6mA) and the goal is absolute accuracy, not tool comparison
- Data are from organisms where ground-truth methylation status cannot be established enzymatically (no M.SssI/Hia5-defined truth set)
- The task is joint histone-modification profiling (nanoHiMe-seq wet-lab assay), not computational methylation-caller benchmarking

## Inputs

- nanopolish v0.13.2 eventalign files (test.eventalign.txt or .gz) from E. coli K12 MG1655 PCR-amplicon nanopore training datasets
- Uppercase reference genome FASTA (ref_upper.fa, e.g. Ecoli_K12_MG1655 with samtools faidx index)
- Ground-truth CpG methylation status derived from M.SssI treatment and 6mA presence/absence from Hia5 treatment
- Nanopore signal-level FAST5 files and basecalled FASTQ for Megalodon/nanopolish runs

## Outputs

- output.mCG.eventalign.txt with per-read-region Log(LIKELIHOOD_RATIO)_mCG values from nanoHiMe
- Per-tool CpG methylation calls from nanopolish v0.13.2 and Megalodon v2.5.0
- ROC AUC values per tool per condition (sites without 6mA vs with 6mA)
- auc_comparison_table.csv with rows for all three tools under both conditions

## How to apply

Run nanoHiMe in mCG-calling mode (./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG) on nanopolish eventalign files from the four E. coli training conditions to obtain per-read-region CpG log-likelihood ratios, where positive values support mCG methylation. Independently call CpG methylation with nanopolish v0.13.2 and Megalodon v2.5.0 (model res_dna_r941_min_modbases-all-context_v001.cfg) on the same datasets. Define ground truth from the enzymatic treatments: M.SssI-methylated CpGs, with sites partitioned into those without 6mA and those with 6mA based on Hia5 treatment. Sweep methylation-calling thresholds to compute ROC curves and AUCs under both conditions, then assemble the per-tool/per-condition values into a CSV table. Judge correctness by whether reproduced AUCs fall within a reasonable tolerance of the reference values (0.953/0.951/0.934 without 6mA; 0.603/0.919/0.922 with 6mA, where nanoHiMe achieves 0.922 vs nanopolish 0.919 and Megalodon 0.603).

## Related tools

- **nanoHiMe** (HMM-based CpG methylation calling producing per-read-region log-likelihood ratios from eventalign files) — https://github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Baseline CpG methylation caller (v0.13.2); also produces the eventalign files consumed by nanoHiMe)
- **Megalodon** (ONT baseline methylation caller (v2.5.0) using model res_dna_r941_min_modbases-all-context_v001.cfg)
- **Minimap2** (Aligns basecalled ONT reads to the reference genome (v2.17, default settings) prior to event alignment)
- **samtools** (Sorts BAM, indexes references, and filters mapped reads at mapping quality >= 20 (samtools view -b -q 20 -F 4))
- **Guppy** (Basecalling of FAST5 with the high-accuracy model (v4.4.2, dna_r9.4.1_450bps_hac.cfg))

## Examples

```
perl nanoHiMe/perl_script/upper.pl Ecoli_K12_MG1655.fasta > ref_upper.fa; samtools faidx ref_upper.fa; ./nanoHiMe mCG ref_upper.fa test.eventalign.txt.gz output.mCG
```

## Evaluation signals

- auc_comparison_table.csv exists in CSV format and contains rows for all three tools (nanoHiMe, nanopolish, Megalodon) under both conditions (without and with 6mA)
- Reproduced AUCs fall within a reasonable tolerance of the reference values: 0.953/0.951/0.934 (without 6mA) and 0.603/0.919/0.922 (with 6mA)
- nanoHiMe mCG output contains positive Log(LIKELIHOOD_RATIO)_mCG for methylated CpGs and negative values for unmethylated CpGs
- All tools were run on the identical E. coli K12 MG1655 eventalign/BAM inputs, with reads filtered at mapping quality >= 20
- Reference FASTA was uppercased (upper.pl) and samtools-faidx-indexed before nanoHiMe mCG-calling, per README requirement

## Limitations

- Megalodon could not accurately identify methylated CpG sites, and its 6mA prediction accuracy was significantly lower than nanoHiMe's on dual-modification DNA (AUC 0.603 vs 0.922 on 6mA-containing sites)
- nanoHiMe's trained model does not cover all combinations of introduced modifications (5-hmC, 5-fC, 4-mC, GpC methylation); a unified calling model is deferred to future work
- Haplotype-level phasing of 6mA regions and CpG methylation depends on heterozygous SNV/INDEL density, degrading at low-density loci
- 6mA calling across the whole genome (when no input.peak.bed is given) can be quite slow

## Evidence

- [results] On sites that also contained 6mA, nanoHiMe and nanopolish markedly outperformed Megalodon at CpG methylation calling, with nanoHiMe achieving the best AUC (0.922 vs 0.919 and 0.603).: [evidence span withheld — anchor status: not_found]
- [discussion] Megalodon couldn't accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe: [evidence span withheld — anchor status: not_found]
- [readme] ./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG: "./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG"
- [readme] A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation.: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation"
- [discussion] we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
