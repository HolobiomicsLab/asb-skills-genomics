---
name: wgbs-comparison-agreement-analysis
description: Use when nanopore per-read 5mC predictions (nCATS, Cas9-targeted, or
  WGS) must be quantitatively compared to whole-genome bisulfite sequencing at CpG
  sites to rank tools by agreement.
when_to_use_negative:
- Input data lacks an independent bisulfite (WGBS) methylation ground truth, so no
  reference-anchored agreement metrics can be computed
- The goal is per-read accuracy on controlled methylation mixtures rather than per-site
  agreement against WGBS
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Provides Snakemake pipelines for the six tools plus random-forest (combination_model_prediction.py)
    and regression (meteore_reg.py) consensus models and prediction_to_mod_frequency.py
    for per-site conversion
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Methylation calling on nCATS fast5 reads via the METEORE Snakemake pipeline
    to generate per-read scores
- name: DeepSignal
  role: Methylation calling on nCATS fast5 reads; its scores are combined with Megalodon's
    in the best-performing METEORE consensus
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified base prediction via Snakemake pipeline (annotate_raw_with_fastqs
    preprocessing) for per-read scores
- name: Guppy
  role: Modified basecalling with the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg
    config; gcf52ref converts guppy calls to reference-anchored output
  repo: https://github.com/kpalin/gcf52ref
- name: Megalodon
  role: Modified base detection (Guppy/Remora backend) providing the best individual-tool
    performance; scores used in the consensus
  repo: https://github.com/nanoporetech/megalodon
- name: DeepMod
  role: Deep-learning methylation calling on nCATS fast5 reads for the comparison
  repo: https://github.com/WGLab/DeepMod
- name: Snakemake
  role: Workflow manager for reproducible execution of the six tool pipelines and
    per-read-output targets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:89f0ef6f20aaf6377fab9bdf329ab390aae734c62cf0a7959cc68ed6f9e081f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepMod
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Snakemake
  - Tombo
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# wgbs-comparison-agreement-analysis

## Summary

Benchmark nanopore CpG methylation callers by matching their combined-strand per-site methylation frequencies against ENCODE WGBS CpG frequencies and computing N, Pearson r, r^2, Spearman rho, and RMSE per method, as done in Table 1 of the METEORE benchmarking study.

## When to use

Use when nanopore per-read 5mC predictions (nCATS, Cas9-targeted, or WGS) must be quantitatively compared to whole-genome bisulfite sequencing at CpG sites to rank tools by agreement.

## When NOT to use

- Input data lacks an independent bisulfite (WGBS) methylation ground truth, so no reference-anchored agreement metrics can be computed
- The goal is per-read accuracy on controlled methylation mixtures rather than per-site agreement against WGBS

## Inputs

- nCATS (Cas9-targeted) nanopore fast5 reads from NA12878
- Per-read methylation score files (ID, Chr, Pos, Strand, Score) for each tool
- ENCODE WGBS CpG methylation frequency datasets ENCFF279HCL and ENCFF835NTC
- Reference genome FASTA for alignment and CpG anchoring

## Outputs

- Combined-strand per-site methylation frequency tables (augmented BED format with chrom, start, end, coverage, methylation frequency) per tool
- Consensus per-site frequencies from METEORE random-forest (combination_model_prediction.py) and regression (meteore_reg.py) models
- Agreement metrics table (e.g. table1_agreement_metrics.csv) with N, Pearson r, r^2, Spearman rho, RMSE per method

## How to apply

Run each methylation-calling tool (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod) on the nanopore fast5 reads via the METEORE Snakemake pipelines, then produce per-read score files (ID, Chr, Pos, Strand, Score). Convert per-read predictions to per-site frequencies (e.g. prediction_to_mod_frequency.py at the 0.5 default threshold), combining both strands per CpG by averaging methylation frequencies and summing coverage. For the METEORE consensus, run combination_model_prediction.py for the random-forest model and meteore_reg.py for the regression model using per-tool trunc_min/trunc_max score ranges (e.g. -5/5 and -50/50), the default --filter, and MinMax scaling. Match each method's combined-strand per-site frequencies to the ENCODE WGBS CpG frequencies (ENCFF279HCL, ENCFF835NTC) and compute per-method N, Pearson r, r^2, Spearman rho, and RMSE, assembling the statistics into a table such as table1_agreement_metrics.csv.

## Related tools

- **METEORE** (Provides Snakemake pipelines for the six tools plus random-forest (combination_model_prediction.py) and regression (meteore_reg.py) consensus models and prediction_to_mod_frequency.py for per-site conversion) — https://github.com/comprna/METEORE
- **Nanopolish** (Methylation calling on nCATS fast5 reads via the METEORE Snakemake pipeline to generate per-read scores)
- **DeepSignal** (Methylation calling on nCATS fast5 reads; its scores are combined with Megalodon's in the best-performing METEORE consensus) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified base prediction via Snakemake pipeline (annotate_raw_with_fastqs preprocessing) for per-read scores)
- **Guppy** (Modified basecalling with the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config; gcf52ref converts guppy calls to reference-anchored output) — https://github.com/kpalin/gcf52ref
- **Megalodon** (Modified base detection (Guppy/Remora backend) providing the best individual-tool performance; scores used in the consensus) — https://github.com/nanoporetech/megalodon
- **DeepMod** (Deep-learning methylation calling on nCATS fast5 reads for the comparison) — https://github.com/WGLab/DeepMod
- **Snakemake** (Workflow manager for reproducible execution of the six tool pipelines and per-read-output targets)

## Examples

```
python combination_model_prediction.py -m default -i score1.txt score2.txt -r ref.fa --threads 8 && python prediction_to_mod_frequency.py -i combined_scores.txt -t 0.5 -o per_site.bed
```

## Evaluation signals

- METEORE REG achieves the highest Pearson correlation (r = 0.8885) and lowest RMSE (0.1607) among all methods against WGBS, matching the published Table 1 ranking
- Per-site frequency tables use combined strands: averaged methylation frequencies and summed coverage per CpG, in the augmented BED format (chrom, start, end, coverage, methylation frequency)
- All six tools plus METEORE show positive Pearson correlation with WGBS signals when both strands are combined
- Per-read score files follow the ID, Chr, Pos, Strand, Score column schema before per-site aggregation

## Limitations

- Discrepancies with WGBS could stem from detection differences between nanopore and WGBS or from disparate model types and training regimes used by the tools
- Running the full tool combination may be impractical for routine analyses because some methods are time-inefficient without GPU support; Megalodon relies on GPUs to be time efficient
- No single method predicts correctly across all ranges of methylation frequency; e.g. Guppy failed at fully methylated sites while Nanopolish and Tombo had high false positives at unmethylated sites
- Megalodon is unsupported/deprecated upstream, with ONT recommending Dorado/Remora/modkit instead

## Evidence

- [readme] In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage.: "In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
- [methods] The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [discussion] The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [abstract] based on the combination of the predictions from two or more tools that shows improved accuracy over individual tools: "based on the combination of the predictions from two or more tools that shows improved accuracy over individual tools"
- [discussion] Megalodon provided overall the best performance but relies on GPUs to be time efficient.: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
