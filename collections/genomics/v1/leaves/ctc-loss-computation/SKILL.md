---
name: ctc-loss-computation
description: Use when training a nanopore signal simulator (e.g. NanoSimFormer) and
  you need the generated signals to be decode-accurate under a fixed production basecaller,
  via a differentiable CTC objective.
when_to_use_negative:
- You only need to basecall existing reads into FASTQ/BAM — use Dorado directly via
  the simulate --basecall option instead of a training loss.
- 'You are preparing the training data rather than training: dataset creation uses
  remora signal-to-sequence alignment, not the CTC objective.'
- The simulator output is not intended to be decoded by an ONT basecaller (e.g. purely
  statistical k-mer-level simulation), where basecaller guidance is unnecessary.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Provides the training loop, dataset loader, and BaseCaller interface that
    wraps the frozen basecaller for CTC loss computation
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: bonito
  role: Loads official ONT basecalling models (frozen, eval mode) used as the differentiable
    guidance network for the CTC loss
  repo: github.com/nanoporetech/bonito
- name: remora
  role: Signal-to-sequence alignment during training dataset preparation, producing
    the dwell-time targets used in the signal-level MSE losses
  repo: github.com/nanoporetech/remora
- name: Dorado
  role: Pre-trained super-accuracy basecaller whose frozen weights provide the CTC
    guidance target
  repo: github.com/nanoporetech/dorado
- name: PyTorch
  role: torch.nn.functional.ctc_loss and DDP multi-GPU training of the simulator
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:9a6444a9484b487fde9288a2c9e862eafcbbf232156b9e352e24de8fbc5aa43c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Dorado
  - NanoSimFormer
  - PyTorch
  - Python
  - bonito
  - remora
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ctc-loss-computation

## Summary

Computes a basecaller-guided CTC negative log-likelihood loss by running simulated nanopore signals through a frozen, pre-trained ONT basecaller (Dorado v5 super-accuracy model loaded via bonito), so the Transformer simulator is trained to produce signals the basecaller decodes correctly.

## When to use

Use when training a nanopore signal simulator (e.g. NanoSimFormer) and you need the generated signals to be decode-accurate under a fixed production basecaller, via a differentiable CTC objective.

## When NOT to use

- You only need to basecall existing reads into FASTQ/BAM — use Dorado directly via the simulate --basecall option instead of a training loss.
- You are preparing the training data rather than training: dataset creation uses remora signal-to-sequence alignment, not the CTC objective.
- The simulator output is not intended to be decoded by an ONT basecaller (e.g. purely statistical k-mer-level simulation), where basecaller guidance is unnecessary.

## Inputs

- Training chunk arrays: signal.npy (simulated signal chunks, 5000 samples for dna_r1041, 4000 for rna004)
- sequence.npy (target base sequences) and sequence_length.npy (unpadded target lengths)
- dwells.npy (resquiggle-derived per-base dwell assignments from remora signal-to-sequence alignment)
- Frozen pre-trained ONT basecalling model checkpoint (Dorado v5 super-accuracy, loaded via bonito) for preset dna_r1041 or rna004

## Outputs

- Scalar CTC negative log-likelihood loss per batch, combined with per-base mean and noise-std MSE terms into the total training objective
- Trained Transformer simulator weights written per-epoch to weights/ with training curves logged to log/ for TensorBoard

## How to apply

Load the training dataset (signal.npy, sequence.npy, sequence_length.npy, dwells.npy) with the NanoSimFormer loader, verifying per-base dwell times sum to signal_chunk_len (5000 samples for dna_r1041, 4000 for rna004). Implement a BaseCaller subclass (default caller_name ONTBasecaller) whose load_model loads the frozen pretrained ONT basecalling model via bonito for the chosen preset and sets it to eval mode so its weights are never updated. Compute torch.nn.functional.ctc_loss on log-softmax logits permuted to (time, batch, classes), encoding targets as A=1, C=2, G=3, T/U=4, pad=0, with unpadded target lengths from sequence_length.npy. Combine this CTC term with MSE losses on per-base signal mean and noise standard deviation derived from resquiggle dwell assignments in dwells.npy, then train with DDP (e.g. --ngpus 2, batch-size 64 or 128, lr 0.0002, epochs 5, seed 42).

## Related tools

- **NanoSimFormer** (Provides the training loop, dataset loader, and BaseCaller interface that wraps the frozen basecaller for CTC loss computation) — https://github.com/BioinfoSZU/NanoSimFormer
- **bonito** (Loads official ONT basecalling models (frozen, eval mode) used as the differentiable guidance network for the CTC loss) — github.com/nanoporetech/bonito
- **remora** (Signal-to-sequence alignment during training dataset preparation, producing the dwell-time targets used in the signal-level MSE losses) — github.com/nanoporetech/remora
- **Dorado** (Pre-trained super-accuracy basecaller whose frozen weights provide the CTC guidance target) — github.com/nanoporetech/dorado
- **PyTorch** (torch.nn.functional.ctc_loss and DDP multi-GPU training of the simulator)

## Examples

```
python -m nano_signal_simulator train --preset ont_r1041_dna_5khz --ngpus 2 --batch-size 64 --lr 0.0002 --epochs 5 --seed 42
```

## Evaluation signals

- Per-base dwell times from dwells.npy sum exactly to signal_chunk_len (5000 for dna_r1041, 4000 for rna004).
- Frozen basecaller weights are bitwise unchanged after training (model kept in eval mode, requires_grad disabled).
- Target encoding round-trips: A=1, C=2, G=3, T/U=4, pad=0, with ctc_loss target lengths equal to sequence_length.npy values.
- Training loss decreases monotonically enough across 5 epochs in the TensorBoard log/ directory; per-epoch checkpoints appear in weights/.
- Downstream check: simulated reads basecalled with Dorado reach median basecalling accuracy > 99% and Q-scores above 22.8 (per abstract).

## Limitations

- Requires an NVIDIA GPU with CUDA compute capability >= 8.x and driver >= 525; Linux (tested on Ubuntu 22.04).
- Guidance is tied to specific ONT chemistries (dna_r1041 and rna004 presets); new chemistries require the documented adaptation workflow.
- The paper reports a finding of >99% median accuracy for R10.4.1 DNA and direct RNA; performance on other chemistries or non-human samples is not established by this evidence.

## Evidence

- [methods] Training minimizes MSE losses on per-base mean signal level and noise standard deviation plus a CTC negative log-likelihood loss computed through a pre-trained, frozen Dorado v5 super-accuracy basecaller: [evidence span withheld — anchor status: not_found]
- [methods] Compute the basecaller-guided CTC loss with torch.nn.functional.ctc_loss on log-softmax logits permuted to (time, batch, classes), using target sequences encoded as A=1, C=2, G=3, T/U=4, pad=0: [evidence span withheld — anchor status: not_found]
- [methods] We use bonito (https://github.com/nanoporetech/bonito) for loading official ONT basecalling models: "We use bonito (https://github.com/nanoporetech/bonito) for loading official ONT basecalling models"
- [methods] remora (https://github.com/nanoporetech/remora) for signal-to-sequence alignment during training dataset preparation: "remora (https://github.com/nanoporetech/remora) for signal-to-sequence alignment during training dataset preparation"
- [abstract] an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals: "an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals"
- [readme] pip install nanosimformer -i https://pypi.org/simple --extra-index-url https://download.pytorch.org/whl/cu126: "pip install nanosimformer -i https://pypi.org/simple --extra-index-url https://download.pytorch.org/whl/cu126"
