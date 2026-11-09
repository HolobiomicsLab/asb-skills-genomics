---
name: chh-methylation-coverage-quantification
description: Use when computing per-context methylation-site coverage fractions in
  repetitive regions (centromeres, TEs) to compare nanopore methylation calls against
  BS-seq, e.g. reproducing Fig. 3b-style 88.0% vs 37.7% centromeric CHH coverage benchmarks.
when_to_use_negative:
- Data from ONT R9.4 FlowCells — Tombo, Megalodon, and DeepSignal-Plant were built
  for R9.4 and cannot be directly applied to R10.4, and this skill assumes R10.4 chemistry
  and the DeepPlant Bi-LSTM model.
- Non-plant or low-CHH-methylation organisms — DeepPlant was trained on plant species
  screened for abundant high-methylation CHH sites, so CHH generalization outside
  that training distribution is not supported by this paper.
- When the goal is per-molecule or per-site methylation-level accuracy (F1, correlation
  with BS-seq methylation frequency) rather than the fraction of sites meeting a coverage
  threshold.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3295
tools:
- name: DeepPlant
  role: Bi-LSTM/Transformer model called via 'extract_and_call_mods' to produce per-cytosine
    CHH methylation_rate values from POD5 + BAM on the T2T-NIP reference
  repo: github.com/xiaochuanle/DeepPlant
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform providing the long reads that profile centromeric CHH
    sites inaccessible to BS-seq
- name: Dorado
  role: ONT's 5mC detection model for R10.4; baseline comparator whose CHH coverage/correlation
    DeepPlant improves upon
  repo: nanoporetech/dorado
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:7b9f2cc3aa9225e0df9cbad402ccd20e016a565b72f4305255d4726ec842c8ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chh-methylation-coverage-quantification

## Summary

Quantifies the fraction of CHH cytosines profiled (i.e., meeting a minimum read-coverage threshold) in centromeric vs non-centromeric regions, comparing DeepPlant 5mC calls on ONT R10.4 data against whole-genome BS-seq on the O. sativa T2T-NIP40 reference. It demonstrates that nanopore-based CHH detection rescues coverage in repetitive centromeres where short-read BS-seq fails.

## When to use

Use when computing per-context methylation-site coverage fractions in repetitive regions (centromeres, TEs) to compare nanopore methylation calls against BS-seq, e.g. reproducing Fig. 3b-style 88.0% vs 37.7% centromeric CHH coverage benchmarks.

## When NOT to use

- Data from ONT R9.4 FlowCells — Tombo, Megalodon, and DeepSignal-Plant were built for R9.4 and cannot be directly applied to R10.4, and this skill assumes R10.4 chemistry and the DeepPlant Bi-LSTM model.
- Non-plant or low-CHH-methylation organisms — DeepPlant was trained on plant species screened for abundant high-methylation CHH sites, so CHH generalization outside that training distribution is not supported by this paper.
- When the goal is per-molecule or per-site methylation-level accuracy (F1, correlation with BS-seq methylation frequency) rather than the fraction of sites meeting a coverage threshold.

## Inputs

- ONT R10.4 nanopore reads in POD5 format (~43x O. sativa Nipponbare)
- Sorted BAM file aligning reads to the T2T-NIP40 reference
- T2T-NIP40 reference genome FASTA with centromeric coordinate annotations for chromosomes 1-12
- Published whole-genome BS-seq dataset for the same sample
- Pre-trained DeepPlant Bi-LSTM model directory (/DeepPlant/model/bilstm)

## Outputs

- Per-cytosine TSV from DeepPlant with methylation_rate for each CHH site
- CHH-site coverage fraction (percent of sites with coverage >= 10) for centromeric regions, DeepPlant vs BS-seq
- CHH-site coverage fraction for non-centromeric regions, DeepPlant vs BS-seq
- Comparison table reproducing Fig. 3b values (88.0% vs 37.7% centromeric; ~98% non-centromeric)

## How to apply

Run 'DeepPlant extract_and_call_mods' inside the chenhx26/deepplant:1.1.0 Docker container (launched with NVIDIA Container Toolkit and the data directory mounted read-write) on the ~43x O. sativa POD5 directory, sorted BAM, and T2T-NIP reference, using the pre-trained Bi-LSTM model at /DeepPlant/model/bilstm with chh_kmer_size 13 and ref_type DNA; this yields a per-cytosine TSV with methylation_rate. Restrict CHH calls to centromeric coordinates of chromosomes 1-12 of the T2T-NIP assembly and apply a minimum per-site read-coverage threshold of 10. Separately compute the same fraction for the published whole-genome BS-seq dataset. The rationale is that BS-seq's short reads under-sample centromeres, so coverage fraction (not methylation level) is the fair comparison metric. Validate against the deposited source data (10.5281/zenodo.15062213, Fig. 3b): 88.0% DeepPlant vs 37.7% BS-seq centromeric CHH coverage, and ~98% non-centromeric coverage for both.

## Related tools

- **DeepPlant** (Bi-LSTM/Transformer model called via 'extract_and_call_mods' to produce per-cytosine CHH methylation_rate values from POD5 + BAM on the T2T-NIP reference) — github.com/xiaochuanle/DeepPlant
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform providing the long reads that profile centromeric CHH sites inaccessible to BS-seq)
- **Dorado** (ONT's 5mC detection model for R10.4; baseline comparator whose CHH coverage/correlation DeepPlant improves upon) — nanoporetech/dorado

## Examples

```
DeepPlant extract_and_call_mods <pod5_dir> <sorted.bam> <T2T-NIP.fa> DNA <write_dir> /DeepPlant/model/bilstm --chh_kmer_size 13
```

## Evaluation signals

- Reproduced centromeric CHH coverage ratio for DeepPlant matches 88.0% (Fig. 3b, source data deposit 10.5281/zenodo.15062213).
- Reproduced BS-seq centromeric CHH coverage ratio matches 37.7% (Fig. 3b).
- Non-centromeric CHH coverage for both DeepPlant and BS-seq is around 98%.
- Per-site read coverage threshold of 10 is applied consistently to both DeepPlant calls and BS-seq before computing coverage fractions.
- CHH calls are restricted to centromeric coordinates of chromosomes 1-12 of the T2T-NIP assembly, and the per-cytosine TSV contains a methylation_rate column.

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (benchmark in Supplementary Data 10).
- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization.
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species; potential overfitting to specific sequence patterns requires careful validation.

## Evidence

- [intro] BS-seq's reliance on short-read sequencing technologies limits its ability to accurately profile complex and repetitive genomic regions, such as centromeres and transposable elements (TEs)9.: [evidence span withheld — anchor status: not_found]
- [methods] Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [methods] docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash: "docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash"
- [readme] Nanopore sequencing enables comprehensive detection of 5-methylcytosine (5mC), particularly in transposable elements and centromeric regions.: "Nanopore sequencing enables comprehensive detection of 5-methylcytosine (5mC), particularly in transposable elements and centromeric regions."
- [discussion] The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset (Supplementary Data 10)"
