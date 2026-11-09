---
name: read-level-basecalling-accuracy-evaluation
description: Use when basecalled reads from a nanopore basecaller (e.g. Chiron FASTQ/FASTA
  output) need a per-read accuracy score against a reference genome, especially to
  compare model variants or basecallers.
when_to_use_negative:
- The goal is consensus/assembly quality rather than per-read basecalling accuracy
  — the paper evaluates read-level accuracy separately from Minimap2/Miniasm assembly
  with Racon polishing.
- The basecaller output has no reference genome to align against, so accuracy cannot
  be computed (e.g. truly novel de novo samples).
- You are benchmarking raw inference throughput (bases/second) rather than accuracy
  — use the decoder speed comparison path instead.
edam_operation: http://edamontology.org/operation_0292
tools:
- name: graphmap
  role: Aligns basecalled reads to the reference genome with default parameters to
    enable read-level accuracy computation
- name: Chiron
  role: Basecalls raw fast5 signal into FASTQ/FASTA reads that are the input to the
    accuracy evaluation
  repo: https://github.com/haotianteng/Chiron
- name: h5py
  role: Reads the .fast5 raw signal files (installed automatically with Chiron via
    pip)
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:b498b5b54571af44386ffecd70440315e45397939d7f11115b95efacfd382779
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Chiron
  - graphmap
  - h5py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# read-level-basecalling-accuracy-evaluation

## Summary

Quantifies basecaller quality by aligning basecalled nanopore reads to a reference genome with graphmap (default parameters) and computing read-level accuracy, enabling side-by-side comparison of basecalling models such as full CNN+RNN+CTC Chiron versus ablated CNN-only or RNN-only variants.

## When to use

Use when basecalled reads from a nanopore basecaller (e.g. Chiron FASTQ/FASTA output) need a per-read accuracy score against a reference genome, especially to compare model variants or basecallers.

## When NOT to use

- The goal is consensus/assembly quality rather than per-read basecalling accuracy — the paper evaluates read-level accuracy separately from Minimap2/Miniasm assembly with Racon polishing.
- The basecaller output has no reference genome to align against, so accuracy cannot be computed (e.g. truly novel de novo samples).
- You are benchmarking raw inference throughput (bases/second) rather than accuracy — use the decoder speed comparison path instead.

## Inputs

- Basecalled reads in FASTQ/FASTA produced by 'chiron call' from raw nanopore fast5 files
- Reference genome sequence (e.g. Phage Lambda NC_001416.1)
- Trained Chiron model(s) (e.g. DNA_default, or custom CNN+RNN+CTC and ablated variants)

## Outputs

- Per-read alignment of basecalled reads to the reference (graphmap output)
- Read-level accuracy value per evaluated model
- Side-by-side accuracy comparison table (e.g. ablation_accuracy_comparison.csv)

## How to apply

Basecall the held-out evaluation reads with each model under test using 'chiron call -i <input_fast5_folder> -o <output_folder> -m <model_folder>', producing per-fast5 FASTQ/FASTA files in the output 'result' folder. Align each basecalled read set to the appropriate reference genome (e.g. Phage Lambda NC_001416.1) using graphmap with default parameters, mirroring the paper's evaluation protocol. Compute read-level accuracy for each model from the alignments. Tabulate the accuracies side by side (e.g. full CNN+RNN+CTC model vs CNN-removed and RNN-removed variants) in a comparison CSV; the comparison is valid if the full model shows substantially higher read-level accuracy than both ablated variants, since the convolutional layers extract local signal patterns and the recurrent layers integrate them into basecall probabilities.

## Related tools

- **graphmap** (Aligns basecalled reads to the reference genome with default parameters to enable read-level accuracy computation)
- **Chiron** (Basecalls raw fast5 signal into FASTQ/FASTA reads that are the input to the accuracy evaluation) — https://github.com/haotianteng/Chiron
- **h5py** (Reads the .fast5 raw signal files (installed automatically with Chiron via pip)) — https://github.com/h5py/h5py

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default
```

## Evaluation signals

- Graphmap alignment completes with default parameters for every evaluated model's read set against the reference genome
- A single read-level accuracy value is computed per model (full CNN+RNN+CTC, CNN-ablated, RNN-ablated)
- The full model's read-level accuracy is substantially higher than both ablated variants, qualitatively reproducing the paper's reported drop
- Comparison values are written to a single side-by-side table such as ablation_accuracy_comparison.csv

## Limitations

- Chiron is within 1% accuracy on bacterial DNA but only within 2% of Albacore on Human DNA, so variant differences may be small and need enough evaluation reads.
- The most accurate comparator (Albacore) is proprietary, limiting open side-by-side benchmarking.
- Increasing the sliding window size improves accuracy but at the cost of increased memory and running time; Chiron is substantially more computationally expensive than Albacore and somewhat more than BasecRAWller.

## Evidence

- [discussion] graphmap alignment to reference: "aligned to the reference genome using graphmap[33] with the default parameters"
- [discussion] read-level accuracy reporting: "Our model was either the best or second-best in terms of accuracy on all of the datasets we tested in terms of read-level accuracy."
- [readme] Chiron call invocation: "chiron call -i <input_fast5_folder> -o <output_folder> -m <model_folder>"
- [intro] two-component architecture rationale: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [readme] training data extraction with h5py/fast5: "This will install Chiron, and [`h5py`](https://github.com/h5py/h5py) (required for reading in `.fast5` files)."
