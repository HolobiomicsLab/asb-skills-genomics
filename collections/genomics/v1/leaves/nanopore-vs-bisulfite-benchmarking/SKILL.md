---
name: nanopore-vs-bisulfite-benchmarking
description: Use when comparing nanopore methylation calls to whole-genome BS-seq
  in repetitive or centromeric regions, to quantify the fraction of CHH sites profiled
  by each method at a minimum read-coverage threshold.
when_to_use_negative:
- When the goal is single-molecule methylation accuracy or F1-score benchmarking rather
  than region-level coverage of profiled sites
- When BS-seq data are long-read or region-specific rather than short-read whole-genome,
  since the coverage advantage being tested depends on BS-seq's short-read limitation
  in repetitive regions
- When the species/platform is not plant R10.4 nanopore data, since DeepPlant's Bi-LSTM
  model and training set target plant CHH methylation on R10.4
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Calls per-cytosine 5mC methylation (extract_and_call_mods with bilstm model,
    chh_kmer_size 13) from POD5/BAM/reference to produce the nanopore CHH calls being
    benchmarked
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform providing the raw nanopore signal data (~43x O. sativa)
    used for methylation calling
- name: Dorado
  role: Baseline plant 5mC detection tool for R10.4 that DeepPlant outperforms on
    CHH whole-genome methylation frequency correlation with BS-seq (0.135-0.381 improvement);
    used as comparison in benchmarking
  repo: nanoporetech/dorado
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:4120e6956f22f5683b3bb4170711ba4501a2e4a700de7864327246aea1da031e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-vs-bisulfite-benchmarking

## Summary

Benchmarks nanopore-based 5mC methylation calls (e.g., DeepPlant on ONT R10.4 data) against whole-genome BS-seq by computing per-site CHH coverage ratios in hard-to-map regions like centromeres, where short-read BS-seq coverage collapses.

## When to use

Use when comparing nanopore methylation calls to whole-genome BS-seq in repetitive or centromeric regions, to quantify the fraction of CHH sites profiled by each method at a minimum read-coverage threshold.

## When NOT to use

- When the goal is single-molecule methylation accuracy or F1-score benchmarking rather than region-level coverage of profiled sites
- When BS-seq data are long-read or region-specific rather than short-read whole-genome, since the coverage advantage being tested depends on BS-seq's short-read limitation in repetitive regions
- When the species/platform is not plant R10.4 nanopore data, since DeepPlant's Bi-LSTM model and training set target plant CHH methylation on R10.4

## Inputs

- ONT nanopore POD5 directory (~43x coverage O. sativa data)
- Sorted BAM alignment of nanopore reads to the reference genome
- Reference genome FASTA (e.g., T2T-NIP40 O. sativa assembly)
- Centromeric coordinate definitions (BED/interval list for chromosomes 1-12)
- Whole-genome BS-seq methylation calls for the same sample/species

## Outputs

- Per-cytosine TSV from DeepPlant with methylation_rate values
- Per-region CHH site coverage ratios (percent of CHH sites with coverage >= 10) for nanopore and BS-seq, split by centromeric vs non-centromeric
- Benchmark comparison table (e.g., 88.0% vs 37.7% centromeric coverage)

## How to apply

Run DeepPlant's 'extract_and_call_mods' inside the chenhx26/deepplant:1.1.0 Docker container (with NVIDIA Container Toolkit, data mounted read-write) on the POD5 directory, sorted BAM, and reference genome, using the pre-trained Bi-LSTM model at /DeepPlant/model/bilstm with chh_kmer_size 13 and ref_type DNA, yielding a per-cytosine TSV with methylation_rate. Restrict CHH calls to the genomic coordinates of interest (e.g., centromeres of chromosomes 1-12 of the T2T-NIP assembly) and apply a minimum per-site read-coverage threshold of 10. Compute the fraction of CHH sites meeting the coverage threshold separately for nanopore calls and for the whole-genome BS-seq dataset, in both centromeric and non-centromeric regions. Compare the resulting coverage ratios to deposited reference values (e.g., Fig. 3b of the source data: 88.0% DeepPlant vs 37.7% BS-seq centromeric, ~98% non-centromeric) to judge whether the benchmark reproduced correctly.

## Related tools

- **DeepPlant** (Calls per-cytosine 5mC methylation (extract_and_call_mods with bilstm model, chh_kmer_size 13) from POD5/BAM/reference to produce the nanopore CHH calls being benchmarked) — https://github.com/xiaochuanle/DeepPlant
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform providing the raw nanopore signal data (~43x O. sativa) used for methylation calling)
- **Dorado** (Baseline plant 5mC detection tool for R10.4 that DeepPlant outperforms on CHH whole-genome methylation frequency correlation with BS-seq (0.135-0.381 improvement); used as comparison in benchmarking) — nanoporetech/dorado

## Examples

```
docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash -c 'DeepPlant extract_and_call_mods pod5_dir sorted.bam ref.fa DNA out_dir /DeepPlant/model/bilstm'
```

## Evaluation signals

- Reproduced centromeric CHH coverage ratio for DeepPlant matches the deposited Fig. 3b value of 88.0%
- Reproduced BS-seq centromeric CHH coverage ratio matches 37.7%
- Non-centromeric CHH coverage is around 98% for both methods, within tolerance of the deposited values
- Every reported CHH site meets the minimum per-site read coverage threshold of 10 before ratios are computed
- Whole-genome CHH methylation frequency correlation between nanopore calls and BS-seq falls in the reported 0.705-0.838 range across species

## Limitations

- DeepPlant is less computationally efficient than Dorado and took much longer to call methylation on the same dataset (Supplementary Data 10)
- Short-read BS-seq inherently under-covers centromeres and transposable elements, so coverage-ratio comparisons are biased toward regions BS-seq cannot map well
- DeepPlant's reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species

## Evidence

- [results] DeepPlant profiled 88.0% of CHH sites in O. sativa centromeric regions: [evidence span withheld — anchor status: not_found]
- [intro] BS-seq's short reads limit centromere profiling: [evidence span withheld — anchor status: not_found]
- [methods] extract_and_call_mods usage: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [methods] Docker container invocation: "docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash"
- [intro] DeepPlant outperforms Dorado vs BS-seq correlation: "DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
