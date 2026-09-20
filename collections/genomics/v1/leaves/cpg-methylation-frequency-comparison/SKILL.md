---
name: cpg-methylation-frequency-comparison
description: Use when per-site methylation frequencies from nanopore methylation callers
  (Nanopolish, Megalodon, Guppy, Tombo, DeepSignal, DeepMod, METEORE) must be compared
  against WGBS CpG frequencies to quantify agreement or sequence-context bias.
when_to_use_negative:
- When you only have raw fast5 files and no methylation calls yet — methylation calling
  pipelines must be run first via METEORE Snakemake workflows
- When no matched WGBS or otherwise trusted ground-truth methylation data exists for
  the same sample — correlation/RMSE comparison and discrepancy ranking cannot be
  computed
- When analyzing non-CpG methylation (e.g., dam/dcm contexts) — this skill is specific
  to 5mC at CG dinucleotides
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3674
tools:
- name: METEORE
  role: Provides Snakemake pipelines for the six tools and the consensus (random forest
    and regression) models whose per-site outputs are compared against WGBS
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Methylation caller producing augmented BED per-site methylation frequency
    files used as one comparison arm
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Best-performing individual methylation detector whose calls are compared against
    WGBS and combined in METEORE consensus
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Methylation caller whose per-read scores feed the METEORE Megalodon+DeepSignal
    consensus used in the WGBS comparison
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: ont_fast5_api
  role: Converts multi-read fast5 to single-read fast5 as input preparation for the
    methylation callers
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Snakemake
  role: Workflow engine running the per-tool methylation-calling pipelines reproducibly
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:0b637fe49dd95701b462ad655866c67c2955ebb3ee2c3cd72328b0567ca412e0
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
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# cpg-methylation-frequency-comparison

## Summary

Benchmark nanopore CpG methylation predictions against WGBS by matching per-site methylation frequencies from augmented BED outputs, computing agreement metrics (N, Pearson r, r^2, Spearman rho, RMSE) per tool, and ranking 8-mer sequence-context discrepancies to reveal AT-rich bias.

## When to use

Use when per-site methylation frequencies from nanopore methylation callers (Nanopolish, Megalodon, Guppy, Tombo, DeepSignal, DeepMod, METEORE) must be compared against WGBS CpG frequencies to quantify agreement or sequence-context bias.

## When NOT to use

- When you only have raw fast5 files and no methylation calls yet — methylation calling pipelines must be run first via METEORE Snakemake workflows
- When no matched WGBS or otherwise trusted ground-truth methylation data exists for the same sample — correlation/RMSE comparison and discrepancy ranking cannot be computed
- When analyzing non-CpG methylation (e.g., dam/dcm contexts) — this skill is specific to 5mC at CG dinucleotides

## Inputs

- nCATS NA12878 fast5 single-read files (converted with multi_to_single_fast5 from ont_fast5_api)
- Reference genome FASTA (e.g., human or data/ecoli_k12_mg1655.fasta for the example)
- Per-read methylation score files per tool (ID, Chr, Pos, Strand, Score columns)
- Nanopolish/Megalodon/METEORE per-site methylation frequency files in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand)
- ENCODE WGBS CpG methylation frequency files (ENCFF279HCL, ENCFF835NTC)

## Outputs

- Per-site methylation frequency tables with both strands combined (averaged frequency, summed coverage)
- Agreement metrics table (e.g., table1_agreement_metrics.csv) with N, Pearson r, r^2, Spearman rho, RMSE per method
- Ranked 8-mer (NNNCGNNN) mean-discrepancy table per tool
- pLogo motif comparisons of low- vs high-discrepancy CpG flanking sequences per GC-content group

## How to apply

Run the six methylation-calling pipelines on nCATS NA12878 fast5 reads via the METEORE Snakemake pipelines, generate per-read score files (ID, Chr, Pos, Strand, Score), and derive per-site methylation frequency tables by merging both strands — averaging methylation frequencies and summing coverage per CpG. For METEORE, run the random-forest consensus (combination_model_prediction.py with -m default and -m optimized) and the regression consensus (meteore_reg.py on WGBS-anchored training data with per-tool trunc_min/trunc_max score ranges such as -5/5 and -50/50, default --filter and MinMax scaling), then convert per-read predictions with prediction_to_mod_frequency.py at the 0.5 default threshold. Match each method's combined-strand per-site frequencies to the ENCODE WGBS (ENCFF279HCL, ENCFF835NTC) CpG frequencies and compute N, Pearson r, r^2, Spearman rho, and RMSE per method into a table such as table1_agreement_metrics.csv. For context bias, extract the NNNCGNNN 8-mer centered on each CpG from the reference genome, group per-site absolute discrepancies by k-mer, rank k-mers from highest to lowest mean discrepancy per tool, and partition CpG sites into low- and high-discrepancy sets within GC-content groups for pLogo motif comparison. The rationale is that no single method predicts correctly across all methylation frequency ranges, so consensus (METEORE REG) improves agreement, and ranked k-mer tables plus pLogos localize residual nanopore–WGBS disagreement.

## Related tools

- **METEORE** (Provides Snakemake pipelines for the six tools and the consensus (random forest and regression) models whose per-site outputs are compared against WGBS) — https://github.com/comprna/METEORE
- **Nanopolish** (Methylation caller producing augmented BED per-site methylation frequency files used as one comparison arm) — https://github.com/comprna/METEORE
- **Megalodon** (Best-performing individual methylation detector whose calls are compared against WGBS and combined in METEORE consensus) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Methylation caller whose per-read scores feed the METEORE Megalodon+DeepSignal consensus used in the WGBS comparison) — https://github.com/bioinfomaticsCSU/deepsignal
- **ont_fast5_api** (Converts multi-read fast5 to single-read fast5 as input preparation for the methylation callers) — https://github.com/nanoporetech/ont_fast5_api
- **Snakemake** (Workflow engine running the per-tool methylation-calling pipelines reproducibly) — https://github.com/comprna/METEORE

## Examples

```
python meteore_reg.py -i per_read_scores.tsv -t wgbs_training.tsv -m regression -trunc_min -5 -trunc_max 50; python prediction_to_mod_frequency.py --threshold 0.5
```

## Evaluation signals

- METEORE REG (Megalodon + DeepSignal) attains the highest Pearson correlation r = 0.8885 and lowest RMSE = 0.1607 against WGBS among all methods
- All six tools plus METEORE show positive Pearson correlation with WGBS signals when both strands are combined
- The combined-strand per-site file must contain exactly the 5 fields: chromosome, start, end, coverage (summed), methylation frequency (averaged)
- Ranked 8-mer tables and pLogos show largest nanopore–WGBS disagreement in AT-rich contexts; Nanopolish, Megalodon, and METEORE show no significant sequence-context motif enrichment
- Per-site frequencies match between strands only after averaging frequencies and summing coverage per CpG

## Limitations

- Discrepancies with WGBS could stem from detection differences between nanopore and WGBS or from disparate model types and training regimes of the tools
- Combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support
- Megalodon provided overall the best performance but relies on GPUs to be time efficient; DeepMod output is not yet in the augmented BED per-site format
- Extreme tool behaviors (e.g., Guppy failing at fully methylated sites; Nanopolish and Tombo high false positives at unmethylated sites) mean no single method predicts correctly across all ranges of methylation frequency

## Evidence

- [results] all tools showed positive correlation with WGBS signals, and METEORE (REG) combining Megalodon and DeepSignal achieved the highest Pearson correlation and lowest RMSE values: [evidence span withheld — anchor status: not_found]
- [readme] we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage"
- [supplementary] Ranking 8-mers by average discrepancy across methods shows that nanopore methylation calls and WGBS disagree the most in AT-rich sequence contexts: [evidence span withheld — anchor status: not_found]
- [methods] The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [discussion] Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested"
