---
name: methylation-frequency-aggregation-from-per-site-calls
description: Use when you have per-site methylation_rate TSVs from DeepPlant or Dorado
  calls on ONT R10.4 plant data and need genome-wide CHH methylation frequencies and
  Pearson correlation (r) vs BS-seq.
when_to_use_negative:
- Evaluating single-molecule accuracy or per-site F1 scores — this skill aggregates
  to genome-wide frequencies and only yields Pearson correlations with BS-seq
- CpG or CHG context benchmarking tables — the workflow described here targets the
  CHH context, where DeepPlant's improvement over Dorado is concentrated
- Data from R9.4 flow cells — Tombo, Megalodon, and DeepSignal-Plant cannot be directly
  applied to R10.4 and this aggregation workflow is grounded in R10.4 CHH calling
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Produces per-cytosine methylation_rate TSVs via extract_and_call_mods with
    the Bi-LSTM model; its threshold method also defines the Dorado* re-scoring variant
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT modification caller run on the same downsampled datasets for
    correlation comparison
  repo: https://github.com/nanoporetech/dorado
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform whose read sets (POD5 + sorted BAM) are downsampled to
    5x-30x depths as the aggregation input
- name: htslib
  role: Provides samtools/htslib utilities for sequence-based random downsampling
    of read sets to target depths
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:02196a4079eb95b908ac5b02a7854b741ede5c02d4766d5ea52ae7037538670e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  - htslib
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# methylation-frequency-aggregation-from-per-site-calls

## Summary

Aggregates per-cytosine methylation_rate TSVs emitted by nanopore modification callers (DeepPlant's extract_and_call_mods, Dorado, Dorado*) into genome-wide CHH methylation frequencies per species, then computes Pearson correlations against whole-genome BS-seq methylation frequencies to benchmark callers.

## When to use

Use when you have per-site methylation_rate TSVs from DeepPlant or Dorado calls on ONT R10.4 plant data and need genome-wide CHH methylation frequencies and Pearson correlation (r) vs BS-seq.

## When NOT to use

- Evaluating single-molecule accuracy or per-site F1 scores — this skill aggregates to genome-wide frequencies and only yields Pearson correlations with BS-seq
- CpG or CHG context benchmarking tables — the workflow described here targets the CHH context, where DeepPlant's improvement over Dorado is concentrated
- Data from R9.4 flow cells — Tombo, Megalodon, and DeepSignal-Plant cannot be directly applied to R10.4 and this aggregation workflow is grounded in R10.4 CHH calling

## Inputs

- Per-cytosine methylation_rate TSVs from DeepPlant extract_and_call_mods
- Dorado and Dorado* (DeepPlant-threshold-rescored) modification calls on the same downsampled datasets
- Whole-genome BS-seq methylation frequencies per species (reference ground truth)
- Downsampled ONT R10.4 POD5 and sorted BAM alignment files at 5x-30x depths for nine species

## Outputs

- Genome-wide CHH methylation frequencies per species per caller and depth
- Pearson correlation values (r) of each caller vs BS-seq
- chh_correlation_table.csv replicating Table 1 layout (nine species x three methods)

## How to apply

Run each caller on the same downsampled POD5+BAM read sets (5x-30x, sequence-based random sampling with samtools view -s from bundled htslib); DeepPlant is invoked with the pre-trained Bi-LSTM model (/DeepPlant/model/bilstm) using context-specific k-mer sizes (CpG 51, CHG 51, CHH 13) and batch size 1024 to emit per-cytosine methylation_rate TSVs. For the Dorado* arm, re-score Dorado calls using DeepPlant's threshold method for comparability. Aggregate per-site methylation rates into genome-wide CHH methylation frequencies per species and compute Pearson correlations against whole-genome BS-seq methylation frequencies. Compile DeepPlant, Dorado, and Dorado* correlations across all nine species into a single table (chh_correlation_table.csv) replicating Table 1's layout.

## Related tools

- **DeepPlant** (Produces per-cytosine methylation_rate TSVs via extract_and_call_mods with the Bi-LSTM model; its threshold method also defines the Dorado* re-scoring variant) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT modification caller run on the same downsampled datasets for correlation comparison) — https://github.com/nanoporetech/dorado
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform whose read sets (POD5 + sorted BAM) are downsampled to 5x-30x depths as the aggregation input)
- **htslib** (Provides samtools/htslib utilities for sequence-based random downsampling of read sets to target depths) — https://github.com/samtools/htslib

## Examples

```
extract_and_call_mods pod5_dir bam_path reference_path ref_type write_dir /DeepPlant/model/bilstm
```

## Evaluation signals

- DeepPlant CHH Pearson correlations at 30x on the six testing datasets fall in the 0.705-0.838 range
- Validation range check: DeepPlant CHH correlations fall within 0.705-0.881 and the improvement over Dorado lies within the reported 14.0%-117.6% margin
- chh_correlation_table.csv contains rows for all nine species and columns/values for the three calling methods (DeepPlant, Dorado, Dorado*)
- Dorado and Dorado* are run on identical downsampled datasets so correlation differences are attributable to calling/thresholding, not input differences

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10)
- Reported improvement ranges differ between source cards (14.0%-117.6%) and the abstract (23.4%-117.6%); verify against Table 1 when validating
- The benchmark relies on species screened for abundant high-methylation CHH sites, which may not capture the full diversity of methylation patterns across all plant species

## Evidence

- [abstract] DeepPlant CHH correlations 0.705-0.838 across nine species, improved 23.4-117.6% vs Dorado: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [methods] Usage of extract_and_call_mods with pod5/bam/reference/model arguments: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [intro] DeepPlant outperforms Dorado in CHH whole-genome frequency correlation: "In parallel, we develop DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
- [intro] R9.4-era tools incompatible with R10.4 data used in this benchmark: "methylation detection tools developed for the R9.4 platform, such as Tombo17, Megalodon18, and DeepSignal-Plant16, cannot be directly applied to the R10.4 FlowCell"
- [discussion] DeepPlant runtime disadvantage documented in Supplementary Data 10: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset (Supplementary Data 10)"
