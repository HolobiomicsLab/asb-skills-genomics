---
name: nanopore-modification-calling-with-deepbam
description: Use when nanopore R10.4 FASTQ/BAM/POD5 data need CpG methylation calling
  and per-site modification rates, especially when Dorado's methylation model shows
  inconsistent performance across heterogeneous human or plant datasets.
when_to_use_negative:
- 'Data are from older nanopore chemistries: DeepBAM targets the R10.4 flowcell, and
  only a small number of tools (DeepMod2, Dorado) support newer R10.4 data.'
- Methylation types other than CpG/5mC are the target, as the model and workflows
  here (motif_type=CG) are built for CpG methylation detection.
- A pre-trained model suffices and no new training is needed — do not run extract_hc_sites/train_lstm.py
  unless retraining on new high-confidence bisulfite data is the goal.
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3673
tools:
- name: DeepBAM
  role: 'Primary tool: deep neural network for CpG methylation calling on nanopore
    R10.4 data via extract_and_call_mods; training via extract_hc_sites and train_lstm.py'
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: Baseline ONT methylation caller used for comparison with DeepBAM's modification
    calls
- name: samtools
  role: Down-sampling nanopore BAM datasets to defined coverage depths (e.g., 10x,
    30x) with samtools view -s
- name: pod5-file-format
  role: C++ ABI for reading nanopore POD5 signal files during feature extraction
  repo: https://github.com/nanoporetech/pod5-file-format
- name: cnpy
  role: Library to read/write .npy and .npz feature files in the C++ calling pipeline
  repo: https://github.com/rogersce/cnpy
- name: ThreadPool
  role: C++11 thread pool used to parallelize asynchronous feature extraction during
    modification calling
  repo: https://github.com/progschj/ThreadPool
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:4e70e9ce68400db9ab6e50556f87b2ed9b7e044c5d6ad7b16726630b3004b88a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  - Oxford Nanopore Technology (ONT)
  - ThreadPool
  - cnpy
  - pod5-file-format
  - samtools
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nanopore-modification-calling-with-deepbam

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for calling CpG methylation states at single-molecule resolution from Oxford Nanopore R10.4 data using DeepBAM, a deep neural network tool that outperforms Dorado in accuracy and cross-dataset robustness on human and plant datasets.

## When to use

Use when nanopore R10.4 FASTQ/BAM/POD5 data need CpG methylation calling and per-site modification rates, especially when Dorado's methylation model shows inconsistent performance across heterogeneous human or plant datasets.

## When NOT to use

- Data are from older nanopore chemistries: DeepBAM targets the R10.4 flowcell, and only a small number of tools (DeepMod2, Dorado) support newer R10.4 data.
- Methylation types other than CpG/5mC are the target, as the model and workflows here (motif_type=CG) are built for CpG methylation detection.
- A pre-trained model suffices and no new training is needed — do not run extract_hc_sites/train_lstm.py unless retraining on new high-confidence bisulfite data is the goal.

## Inputs

- Oxford Nanopore R10.4 sequencing data (POD5/BAM) from human cell lines or plant species
- Reference genome (FASTA) for feature extraction and modification calling
- High-confidence bisulfite (BS-seq) sites, if training a new DeepBAM model
- Matched BS-seq methylation frequency tracks for validation

## Outputs

- Per-site CpG methylation calls / per-site methylation rates
- Genome-wide CpG methylation frequency profiles
- Trained DeepBAM model (when using extract_hc_sites + train_lstm.py)
- Correlation table of DeepBAM vs BS-seq methylation frequencies (e.g., bs_seq_correlation_table.csv)

## How to apply

Clone the DeepBAM repository and run its extract_and_call_mods mode against a reference genome to asynchronously extract features from nanopore signals (using POD5 inputs and the pod5 C++ ABI, cnpy for .npy/.npz exchange, and a C++11 ThreadPool for parallelism) and pass them to the trained model for CpG modification calling; for the CG motif the paper uses motif_type=CG with kmer_size=51, batch_size=1024, and num_workers=10. For model training from scratch, first run the extract_hc_sites process to extract features from high-confidence bisulfite data, then run py/train_lstm.py to train the LSTM model. Compare against Dorado's methylation calling on the same reads, since DeepBAM achieved higher AUROC (98.47% on average, up to 7.36% improvement) and F1 (94.97% on average, up to 16.24% improvement) across heterogeneous datasets. Aggregate per-site methylation calls into genome-wide methylation frequency profiles and validate against BS-seq tracks via Pearson correlation, expecting >0.95 on most datasets.

## Related tools

- **DeepBAM** (Primary tool: deep neural network for CpG methylation calling on nanopore R10.4 data via extract_and_call_mods; training via extract_hc_sites and train_lstm.py) — https://github.com/huicongyao/DeepBAM
- **Dorado** (Baseline ONT methylation caller used for comparison with DeepBAM's modification calls)
- **samtools** (Down-sampling nanopore BAM datasets to defined coverage depths (e.g., 10x, 30x) with samtools view -s)
- **pod5-file-format** (C++ ABI for reading nanopore POD5 signal files during feature extraction) — https://github.com/nanoporetech/pod5-file-format
- **cnpy** (Library to read/write .npy and .npz feature files in the C++ calling pipeline) — https://github.com/rogersce/cnpy
- **ThreadPool** (C++11 thread pool used to parallelize asynchronous feature extraction during modification calling) — https://github.com/progschj/ThreadPool

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git && ./DeepBAM extract_and_call_mods --motif_type CG --kmer_size 51 --batch_size 1024 --num_workers 10
```

## Evaluation signals

- Single-molecule AUROC of DeepBAM calls should average ~98.47% and reach ≥0.956 across datasets, exceeding Dorado (up to 7.36% improvement).
- F1 scores should average ~94.97% across datasets, exceeding Dorado (up to 16.24% improvement).
- Genome-wide CpG methylation frequency profiles should show Pearson correlation >0.95 with BS-seq on four of five datasets (e.g., 0.9032–0.9847 at 10x; 0.9212–0.9872 at 30x).
- DeepBAM-based methylation frequencies should outperform Dorado-based frequencies in correlation with BS-seq on all datasets tested.
- The output CSV (e.g., bs_seq_correlation_table.csv) should contain rows for both coverages (10x, 30x) and both callers (DeepBAM, Dorado).

## Limitations

- DeepBAM's performance currently requires ongoing optimization; the authors acknowledge 'the necessity for ongoing optimization of DeepBAM'.
- Only 9-kmer model modules have been released in the updated version; other kmer modules were pending release ('We have now updated 9-kmer modules and we will soon release all the other kmer modules').
- Dorado's methylation calling model 'remains poorly tested', so cross-dataset variation in baseline calls can affect comparative conclusions.
- No software version or release identifier for DeepBAM is stated, complicating reproducibility.

## Evidence
