---
name: bisulfite-sequence-correlation-benchmarking
description: Use when comparing nanopore methylation callers for a plant genome against
  a bisulfite-sequencing (BS-seq) ground truth, when you have R10.4 POD5 plus sorted
  BAM alignments and want genome-wide CHH/CpG/CHG methylation-frequency Pearson r
  as the accuracy metric.
when_to_use_negative:
- Evaluating single-molecule accuracy or F1 score of a caller — this skill measures
  aggregate whole-genome methylation-frequency correlation with BS-seq, not per-read
  agreement
- Data from the R9.4 platform — tools like Tombo, Megalodon, and DeepSignal-Plant
  were built for R9.4 and cannot be directly applied to R10.4, and the benchmark correlations
  reported here are R10.4-specific
- Benchmarking CpG/CHG-only questions where CHH scarcity is not the concern — the
  paper's key contribution and hardest benchmark case is CHH, where high-methylation
  positive samples are scarce
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DeepPlant
  role: Primary methylation caller under benchmark; runs extract_and_call_mods with
    the Bi-LSTM model to emit per-cytosine methylation_rate TSVs, and supplies the
    threshold method used to build the Dorado* variant
  repo: github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Comparator ONT 5mC calling tool on the R10.4 platform; run on identical downsampled
    datasets to produce baseline correlations, and re-scored as Dorado* with DeepPlant's
    threshold
  repo: https://github.com/nanoporetech/dorado
- name: htslib
  role: Bundled library providing samtools view -s for sequence-based random downsampling
    of the sorted BAM read sets to 5x-30x target depths
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:5dfc7b948d99473e968ca0fb36d61daee2eccb76bccd1a7ee34a801d054e1121
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - Dorado
  - htslib
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bisulfite-sequence-correlation-benchmarking

## Summary

Benchmark plant 5mC methylation callers (DeepPlant vs Dorado/Dorado*) on ONT R10.4 data by downsampling reads to 5x-30x depth, aggregating per-cytosine methylation rates into genome-wide methylation frequencies, and computing Pearson correlations against whole-genome BS-seq methylation frequencies across CpG, CHG, and especially CHH contexts.

## When to use

Use when comparing nanopore methylation callers for a plant genome against a bisulfite-sequencing (BS-seq) ground truth, when you have R10.4 POD5 plus sorted BAM alignments and want genome-wide CHH/CpG/CHG methylation-frequency Pearson r as the accuracy metric.

## When NOT to use

- Evaluating single-molecule accuracy or F1 score of a caller — this skill measures aggregate whole-genome methylation-frequency correlation with BS-seq, not per-read agreement
- Data from the R9.4 platform — tools like Tombo, Megalodon, and DeepSignal-Plant were built for R9.4 and cannot be directly applied to R10.4, and the benchmark correlations reported here are R10.4-specific
- Benchmarking CpG/CHG-only questions where CHH scarcity is not the concern — the paper's key contribution and hardest benchmark case is CHH, where high-methylation positive samples are scarce

## Inputs

- ONT R10.4 POD5 read files per species
- Sorted BAM alignments of the nanopore reads to the species reference genome
- Reference genome FASTA and ref_type flag
- DeepPlant pre-trained Bi-LSTM model directory (/DeepPlant/model/bilstm)
- Whole-genome BS-seq methylation frequency table per species (gold standard)

## Outputs

- Per-cytosine methylation_rate TSVs per caller and depth
- Genome-wide CHH (and CpG/CHG) methylation frequencies per species and depth
- Pearson correlation coefficients (r) vs BS-seq for DeepPlant, Dorado, and Dorado*
- chh_correlation_table.csv with rows for all nine species and the three calling methods (Table 1 layout)

## How to apply

Downsample each species' nanopore R10.4 read set (POD5 + sorted BAM) to target depths of 5x-30x using sequence-based random sampling (e.g., samtools view -s from the bundled htslib). Call methylation with DeepPlant's extract_and_call_mods using the pre-trained Bi-LSTM model directory (/DeepPlant/model/bilstm), default context-specific k-mer sizes (CpG 51, CHG 51, CHH 13), and batch size 1024, emitting per-cytosine methylation_rate TSVs. Run Dorado on the same downsampled data, plus a Dorado* variant re-scored with DeepPlant's threshold method for comparability. Aggregate per-site methylation rates into genome-wide CHH methylation frequencies per species and compute Pearson correlations against whole-genome BS-seq methylation frequencies. Compile values for DeepPlant, Dorado, and Dorado* across all nine species into chh_correlation_table.csv, replicating the layout of Table 1.

## Related tools

- **DeepPlant** (Primary methylation caller under benchmark; runs extract_and_call_mods with the Bi-LSTM model to emit per-cytosine methylation_rate TSVs, and supplies the threshold method used to build the Dorado* variant) — github.com/xiaochuanle/DeepPlant
- **Dorado** (Comparator ONT 5mC calling tool on the R10.4 platform; run on identical downsampled datasets to produce baseline correlations, and re-scored as Dorado* with DeepPlant's threshold) — https://github.com/nanoporetech/dorado
- **htslib** (Bundled library providing samtools view -s for sequence-based random downsampling of the sorted BAM read sets to 5x-30x target depths) — https://github.com/samtools/htslib

## Examples

```
docker pull chenhx26/deepplant:1.1.0 && extract_and_call_mods pod5_dir bam_path reference_path ref_type write_dir /DeepPlant/model/bilstm
```

## Evaluation signals

- DeepPlant CHH Pearson correlations fall within the reported 0.705-0.838 range (validation bound 0.705-0.881) with BS-seq
- DeepPlant's improvement over Dorado lies within the reported 14.0%-117.6% margin (abstract reports 23.4%-117.6%)
- chh_correlation_table.csv contains rows for all nine species (three training, six testing) and all three calling methods (DeepPlant, Dorado, Dorado*)
- Correlation trends are consistent across the 5x-30x depth series, with the six testing species evaluated at 30x
- Per-cytosine methylation_rate TSVs are produced for every species/depth/caller combination with the expected context-specific k-mer settings (CpG 51, CHG 51, CHH 13)

## Limitations

- DeepPlant is less computationally efficient than Dorado, taking much longer to call methylation on the same dataset (Supplementary Data 10)
- Reliance on species with naturally abundant CHH methylation for training may not capture the full diversity of methylation patterns across all plant species
- Scarcity of high-methylation CHH samples continues to pose challenges for model training and generalization, and potential overfitting to specific sequence patterns requires careful validation

## Evidence

- [abstract] Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [methods] Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [intro] achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification: "achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification"
- [discussion] Our reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species: "Our reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species"
- [methods] docker pull chenhx26/deepplant:1.1.0: "docker pull chenhx26/deepplant:1.1.0"
