---
name: cross-validation-read-level-accuracy-evaluation
description: Use when a newly trained Chiron checkpoint (e.g., 3 epochs on ~4,000
  E. coli/Lambda reads on a K80) must be judged fit for use by held-out read-level
  accuracy rather than training loss alone.
when_to_use_negative:
- All reads were included in training (no cross-validation remainder exists), so held-out
  read-level accuracy cannot be computed without re-partitioning.
- Evaluating a pre-shipped DNA_default/RNA_default model on the README sample fast5
  files rather than validating a user-trained checkpoint.
- Human DNA data, where the paper reports a larger ~2% accuracy gap to Albacore —
  read-level parity conclusions from 4,000 bacterial/viral reads do not transfer.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Chiron
  role: Basecalls the held-out fast5 reads with the trained checkpoint via `chiron
    call` for accuracy evaluation
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: GPU training framework (tensorflow-gpu 1.15) that produces the checkpoint
    being validated
- name: h5py
  role: Reads .fast5 raw signal files into the basecaller during held-out evaluation
  repo: https://github.com/h5py/h5py
- name: graphmap
  role: Aligns basecalled reads to the reference genome with default parameters to
    derive read-level accuracy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:21a0603f8ca4a09f1c375980d13a99f508a816f841c476ebe2548eca7be114fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Chiron
  - TensorFlow
  - graphmap
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-validation-read-level-accuracy-evaluation

## Summary

Validates a freshly trained Chiron CNN+RNN+CTC basecalling checkpoint by measuring read-level accuracy on the cross-validation remainder (reads excluded from the small ~4,000-read training set), benchmarked against ONT Albacore accuracy expectations.

## When to use

Use when a newly trained Chiron checkpoint (e.g., 3 epochs on ~4,000 E. coli/Lambda reads on a K80) must be judged fit for use by held-out read-level accuracy rather than training loss alone.

## When NOT to use

- All reads were included in training (no cross-validation remainder exists), so held-out read-level accuracy cannot be computed without re-partitioning.
- Evaluating a pre-shipped DNA_default/RNA_default model on the README sample fast5 files rather than validating a user-trained checkpoint.
- Human DNA data, where the paper reports a larger ~2% accuracy gap to Albacore — read-level parity conclusions from 4,000 bacterial/viral reads do not transfer.

## Inputs

- Trained Chiron model checkpoint directory (log_dir/model_name/) from a 3-epoch run on ~4,000 reads
- Held-out cross-validation remainder of re-squiggled .fast5 nanopore reads (excluded from the 4,000-read training set)
- Reference genome (e.g., Phage Lambda NC_001416.1 or E. coli) for alignment-based accuracy scoring

## Outputs

- Per-read accuracy of basecalled held-out reads vs reference
- Aggregate read-level accuracy of the checkpoint on the cross-validation remainder
- Comparison table vs Albacore v1.1.1/v2.0.1 and BasecRAWller 0.1 baselines

## How to apply

Reserve a cross-validation remainder of fast5 reads before training; train Chiron for 3 epochs on only the ~4,000-read (E. coli + Lambda) subset with tensorflow-gpu 1.15 so the remainder stays untouched. After the ~10 h K80 run, confirm a checkpoint exists in log_dir/model_name/, then basecall the held-out fast5 files with the checkpoint (`chiron call`, beam width 30 per the dna-pre preset, or --beam 0 for greedy). Align the basecalled reads to the reference (e.g., graphmap with default parameters) and compute read-level accuracy. Judge success if the small-data-trained model reaches accuracy consistent with the paper's claim of state-of-the-art basecalling from 4,000 reads — roughly on par with Albacore v2.0.1 on bacterial/viral DNA (within ~1% on bacterial DNA).

## Related tools

- **Chiron** (Basecalls the held-out fast5 reads with the trained checkpoint via `chiron call` for accuracy evaluation) — https://github.com/haotianteng/Chiron
- **TensorFlow** (GPU training framework (tensorflow-gpu 1.15) that produces the checkpoint being validated)
- **h5py** (Reads .fast5 raw signal files into the basecaller during held-out evaluation) — https://github.com/h5py/h5py
- **graphmap** (Aligns basecalled reads to the reference genome with default parameters to derive read-level accuracy)

## Examples

```
chiron call -i <held_out_fast5_folder> -o <output_folder> -m <log_dir>/<model_name> --beam 30
```

## Evaluation signals

- A model checkpoint file exists in log_dir/model_name/ after the 3-epoch, ~10 h K80 training run.
- Read-level accuracy on the cross-validation remainder is consistent with state-of-the-art quality reported for a model trained on only 4,000 reads.
- Accuracy on bacterial DNA is within ~1% of ONT Albacore v2.0.1 (within ~2% on Human DNA, where parity is not expected).
- Beam-search decoding (--beam 30, dna-pre preset) yields higher held-out accuracy than the greedy decoder (--beam 0).
- Held-out reads are verified disjoint from the 4,000-read training set (2,000 E. coli + 2,000 Lambda).

## Limitations

- The Chiron model is substantially more computationally expensive than Albacore and somewhat more than BasecRAWller, so held-out inference over many reads is slow.
- Training labels derive from Metrichor basecalling plus Nanoraw error correction, so Metrichor errors must be corrected via alignment to avoid bias being learned into the model — residual label errors cap achievable held-out accuracy.
- The checkpoint's accuracy is validated only on R9.4 viral/bacterial data; it is within 2% of Albacore on Human DNA and more extensive training on broader species spectra would be needed to close that gap.

## Evidence

- [abstract] Trained with only a small set of 4,000 reads, we show that our model provides state-of-the-art basecalling accuracy, even on previously unseen species.: "Trained with only a small set of 4,000 reads, we show that our model provides state-of-the-art basecalling accuracy, even on previously unseen species."
- [discussion] Our model was either the best or second-best in terms of accuracy on all of the datasets we tested in terms of read-level accuracy.: "Our model was either the best or second-best in terms of accuracy on all of the datasets we tested in terms of read-level accuracy."
- [intro] Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods: "Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods"
- [discussion] Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA.: "Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA."
- [readme] Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer.: "Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer."
