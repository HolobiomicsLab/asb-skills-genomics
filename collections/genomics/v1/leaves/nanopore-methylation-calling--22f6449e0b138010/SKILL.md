---
name: nanopore-methylation-calling
description: Use when calling 5mC methylation from ONT R10.4 POD5/BAM plant nanopore
  data in CpG, CHG, or CHH contexts, or when needing per-site methylation rates comparable
  to BS-seq across species.
when_to_use_negative:
- Data are from the older R9.4 FlowCell — tools like Tombo, Megalodon, and DeepSignal-Plant
  developed for R9.4 cannot be directly applied to R10.4, and this skill's models
  are R10.4-specific
- The goal is CpG/CHG-only methylation profiling where Dorado's default performance
  is already sufficient and computational runtime is the binding constraint
- Input is already aggregated methylation frequency data rather than raw nanopore
  signal/read alignments
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Primary deep learning (Bi-LSTM + Transformer) 5mC detection tool for plant
    R10.4 nanopore data; run via extract_and_call_mods with the pre-trained Bi-LSTM
    model
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT modification-calling engine used to produce comparison calls;
    its output is optionally re-thresholded (Dorado*) with DeepPlant's adaptive P_th
    method
  repo: https://github.com/nanoporetech/dorado
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform whose raw signal data (POD5) is the input for methylation
    calling
- name: htslib
  role: Bundled library (e.g., samtools view -s) used to downsample nanopore read
    sets to target sequencing depths before calling
  repo: https://github.com/samtools/htslib
- name: DeepSignal-Plant
  role: Prior R9.4-era deep learning tool from the same group; incompatible with R10.4
    but contextualizes the method
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:03d9be4bbb6390ad49e3f0d484cd0eb4e516c2c15ed7bcb65989b6a076ca15de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - DeepSignal-Plant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  - htslib
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nanopore-methylation-calling

## Summary

Detect 5-methylcytosine (5mC) modifications in plant genomes from Oxford Nanopore R10.4 sequencing data using DeepPlant (Bi-LSTM + Transformer), benchmarking against Dorado, and refine methylation calls with an adaptive score-threshold selection method before computing genome-wide methylation frequency correlations with BS-seq.

## When to use

Use when calling 5mC methylation from ONT R10.4 POD5/BAM plant nanopore data in CpG, CHG, or CHH contexts, or when needing per-site methylation rates comparable to BS-seq across species.

## When NOT to use

- Data are from the older R9.4 FlowCell — tools like Tombo, Megalodon, and DeepSignal-Plant developed for R9.4 cannot be directly applied to R10.4, and this skill's models are R10.4-specific
- The goal is CpG/CHG-only methylation profiling where Dorado's default performance is already sufficient and computational runtime is the binding constraint
- Input is already aggregated methylation frequency data rather than raw nanopore signal/read alignments

## Inputs

- ONT R10.4 POD5 read files
- Sorted BAM alignment files
- Reference genome (with ref_type flag for extract_and_call_mods)
- Pre-trained DeepPlant Bi-LSTM model directory (/DeepPlant/model/bilstm)
- Whole-genome BS-seq methylation frequency data (for correlation validation)

## Outputs

- Per-cytosine methylation_rate TSV files from DeepPlant and Dorado
- Thresholded (P_th-filtered) methylation calls (Dorado* condition)
- Genome-wide methylation frequency correlations (Pearson r) with BS-seq per species and context (e.g., chh_correlation_table.csv)

## How to apply

Run DeepPlant's extract_and_call_mods (via docker image chenhx26/deepplant:1.1.0 with --gpus all and mounted data paths) on POD5 read sets, sorted BAM alignments, and a reference genome, pointing to the pre-trained Bi-LSTM model directory (/DeepPlant/model/bilstm) with context-specific k-mer sizes (CpG 51, CHG 51, CHH 13), batch size 1024, and multiple feature-extraction workers to emit per-cytosine methylation_rate TSVs. In parallel, run Dorado modification calling on the same downsampled datasets (e.g., depths of 5x-30x produced by sequence-based random sampling with samtools view -s from bundled htslib). To improve quantification, apply the adaptive threshold method: bin called cytosine sites by methylation_rate across 0.2-0.9 into 70 equal bins of step 0.01, set the confidence threshold P_th at the left endpoint of the minimum-population bin, and retain calls with methylation_rate >= P_th. Aggregate per-site methylation rates into genome-wide methylation frequencies per species and sequence context, then compute Pearson correlations against whole-genome BS-seq methylation frequencies to assess accuracy. Rationale: Dorado is the only existing plant 5mC tool on R10.4 but lacks extensive species testing, while DeepPlant was trained on datasets covering 97.2% of all possible CHH 9-mer motifs screened from species with abundant high-methylation CHH sites.

## Related tools

- **DeepPlant** (Primary deep learning (Bi-LSTM + Transformer) 5mC detection tool for plant R10.4 nanopore data; run via extract_and_call_mods with the pre-trained Bi-LSTM model) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT modification-calling engine used to produce comparison calls; its output is optionally re-thresholded (Dorado*) with DeepPlant's adaptive P_th method) — https://github.com/nanoporetech/dorado
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform whose raw signal data (POD5) is the input for methylation calling)
- **htslib** (Bundled library (e.g., samtools view -s) used to downsample nanopore read sets to target sequencing depths before calling) — https://github.com/samtools/htslib
- **DeepSignal-Plant** (Prior R9.4-era deep learning tool from the same group; incompatible with R10.4 but contextualizes the method)

## Examples

```
docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 extract_and_call_mods pod5_dir bam_path reference_path ref_type write_dir /DeepPlant/model/bilstm
```

## Evaluation signals

- Whole-genome CHH methylation frequency Pearson correlations with BS-seq fall in the reported 0.705-0.838 range across species
- DeepPlant's correlation improvement over Dorado falls within the reported margin (e.g., 14.0%-117.6% at varying depths; 23.4-117.6% across nine species)
- Applying the adaptive threshold (P_th at minimum-population bin endpoint, 70 bins over 0.2-0.9) to Dorado calls (Dorado*) improves site-level correlation with BS-seq relative to Dorado defaults
- DeepPlant achieves superior single-molecule accuracy and F1 score versus Dorado on benchmarked sites
- Output table contains rows for all nine species and all three calling methods (DeepPlant, Dorado, Dorado*)

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (benchmark in Supplementary Data 10)
- Scarcity of high-methylation CHH samples continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species; in vitro enzyme-catalyzed methylation is ineffective for CHH because CHH methylation depends on RdDM
- Potential for overfitting to specific sequence patterns; in vitro approaches and PCR with base substitutions have significant difficulties producing high-purity 5mC and alter base contexts

## Evidence

- [abstract] DeepPlant achieves high whole-genome CHH correlations: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [methods] extract_and_call_mods usage: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [methods] Docker image pull: "docker pull chenhx26/deepplant:1.1.0"
- [intro] Correlation improvement range vs Dorado: "In parallel, we develop DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
- [discussion] Training dataset 9-mer context coverage: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [intro] R9.4 tools incompatible with R10.4: "methylation detection tools developed for the R9.4 platform, such as Tombo17, Megalodon18, and DeepSignal-Plant16, cannot be directly applied to the R10.4 FlowCell"
- [readme] Dorado lacks extensive species testing: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing."
