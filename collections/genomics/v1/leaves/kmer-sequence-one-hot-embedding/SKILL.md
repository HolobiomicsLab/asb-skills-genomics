---
name: kmer-sequence-one-hot-embedding
description: Use when preparing segmented k-mer sequence inputs for a transformer-based
  signal simulator, e.g., feeding uncalled4 eventalign 9-mer chunks into the seq2squiggle
  FFT encoder-decoder pipeline.
when_to_use_negative:
- Inputs are already numeric feature vectors or pretrained model embeddings rather
  than raw nucleotide k-mer strings
- The downstream model relies on static k-mer pore models instead of learned sequential
  context, where one-hot encoding adds no value
- Only raw unsegmented reads are available with no event-level alignment to define
  k-mer boundaries
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: seq2squiggle
  role: Transformer-based signal simulator whose pipeline consumes one-hot embedded
    k-mer sequences at the FFT encoder input
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: uncalled4
  role: Generates the eventalign training data (k-mer segments with signal indices
    and sample counts) that are one-hot embedded
  repo: https://github.com/skovaka/uncalled4
- name: Python
  role: Implementation language (PyTorch) for the one-hot embedding and the seq2squiggle
    training loop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:3d690b8d1c14ab20c8e7c09284349a7bd52a37e9c7dce404c718c7dbd7c8ecd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - Python
  - seq2squiggle
  - uncalled4
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# k-mer sequence one-hot embedding

## Summary

Encodes nucleotide k-mer sequences (e.g., 9-mer events from uncalled4 eventalign chunks) as one-hot vectors so a feed-forward transformer can map DNA input to nanopore signal output in seq2squiggle.

## When to use

Use when preparing segmented k-mer sequence inputs for a transformer-based signal simulator, e.g., feeding uncalled4 eventalign 9-mer chunks into the seq2squiggle FFT encoder-decoder pipeline.

## When NOT to use

- Inputs are already numeric feature vectors or pretrained model embeddings rather than raw nucleotide k-mer strings
- The downstream model relies on static k-mer pore models instead of learned sequential context, where one-hot encoding adds no value
- Only raw unsegmented reads are available with no event-level alignment to define k-mer boundaries

## Inputs

- uncalled4 eventalign table (events.tsv) with 9-mer reference segments and signal indices/samples
- Nucleotide k-mer sequences extracted from segmented signal-to-reference alignments
- Raw nanopore signal chunks (max 250 signal points per segmented k-mer window)

## Outputs

- One-hot encoded k-mer tensors ready for the FFT encoder
- Training batches pairing embedded k-mers with corresponding signal points for MSE/duration-NLL/noise-MSE loss computation

## How to apply

Load the uncalled4 eventalign training chunks, where each event covers a 9-mer (16 k-mer windows segmented to at most 250 signal points), and one-hot embed the nucleotide k-mer sequences before the FFT encoder. The embedding feeds the non-autoregressive architecture: a feed-forward transformer (FFT) encoder with 2 blocks, a length regulator, and an FFT decoder with 2 blocks that predicts raw signal values. Because seq2squiggle is non-autoregressive and learns sequential context from segmented signal data (rather than static k-mer models), the one-hot k-mer representation is the interface that carries nucleotide identity into the encoder. Keep the embedding dimension consistent with the model checkpoint (e.g., seq2squiggle_model.pt) and align each embedded k-mer with its per-k-mer signal points so the duration predictor (gamma-parameterized) and noise module operate on correctly paired sequence-signal inputs.

## Related tools

- **seq2squiggle** (Transformer-based signal simulator whose pipeline consumes one-hot embedded k-mer sequences at the FFT encoder input) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **uncalled4** (Generates the eventalign training data (k-mer segments with signal indices and sample counts) that are one-hot embedded) — https://github.com/skovaka/uncalled4
- **Python** (Implementation language (PyTorch) for the one-hot embedding and the seq2squiggle training loop)

## Examples

```
uncalled4 align input.ref input.slow5 --bam-in input.bam --eventalign-out events.tsv --eventalign-flags print-read-names,signal-index,samples --pore-model dna_r10.4.1_400bps_9mer --flowcell FLO-MIN114 --kit SQK-LSK114
```

## Evaluation signals

- Every k-mer in a batch one-hot encodes to exactly one active position per 4-nucleotide slot (A/C/G/T), with shape matching the FFT encoder input expectation
- Each embedded k-mer aligns 1:1 with its eventalign signal segment of at most 250 points
- After training, the combined loss (signal MSE + duration NLL with tau = 0.0005 + noise MSE) converges in the recorded loss_curve.png
- The saved checkpoint seq2squiggle_model.pt loads without shape errors at the embedding layer

## Limitations

- Embedding depends on event-level alignments; without uncalled4 or f5c eventalign tables there are no k-mer boundaries to encode
- uncalled4 >= 4.1.0 outputs normalized pore model values (mean 0, std 1) rather than scaled pA, so signal values paired with embeddings may need rescaling
- Model weights are chemistry-specific; embeddings must match the k-mer size of the pretrained model (e.g., dna_r10.4.1_400bps_9mer uses 9-mers)

## Evidence

- [other] one-hot embed the nucleotide k-mer sequences: [evidence span withheld — anchor status: not_found]
- [abstract] non-autoregressive model generating nanopore signals: "we introduce seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences"
- [abstract] learns sequential contextual information: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"
- [readme] eventalign training data generation: [evidence span withheld — anchor status: external_doc]
