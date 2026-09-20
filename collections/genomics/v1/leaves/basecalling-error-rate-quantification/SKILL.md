---
name: basecalling-error-rate-quantification
description: Use when comparing basecalling accuracy of one or more ONT basecallers
  on raw fast5 test sets — e.g. compiling a basecaller-by-dataset table of identity/indel/mismatch
  rates like Table 1 of the Chiron paper.
when_to_use_negative:
- Input is already FASTQ/basecalled reads and no basecaller comparison is needed —
  use a variant-calling or assembly-error-rate method instead
- You need consensus-level or assembly-level accuracy (e.g. after Minimap2/Miniasm
  + 10 rounds of Racon polishing), not raw read-level basecalling accuracy
- The fast5 raw signal has not been basecalled by any method yet — run a basecaller
  (this skill quantifies basecaller output, it does not produce it)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Chiron
  role: Deep-learning basecaller whose beam-search (--beam 30) and greedy (--beam
    0) decoder outputs are quantified and compared
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: ONT basecaller (v1.1.1 and v2.0.1) run with default parameters as an accuracy
    benchmark
- name: BasecRAWller
  role: RNN-based basecaller (v0.1, default parameters) included in the read-level
    accuracy comparison
- name: Metrichor
  role: ONT cloud basecaller used as an additional comparison baseline
- name: graphmap
  role: Aligner used to map basecalled reads to the reference genome with default
    parameters, from which error rates are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:4ec75929caa5cbdeb89bae65a161273df50f1b135100e7aa1401c59f1e7860f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - BasecRAWller
  - Chiron
  - Metrichor
  - graphmap
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# basecalling-error-rate-quantification

## Summary

Quantifies nanopore basecaller accuracy by aligning basecalled reads (FASTQ) to a reference genome with graphmap and computing per-read identity, deletion, insertion, and mismatch rates for every basecaller–dataset combination, as done in the Chiron (Teng et al., 2018) benchmark of Chiron vs Metrichor, Albacore v1.1.1/v2.0.1, and BasecRAWller v0.1.

## When to use

Use when comparing basecalling accuracy of one or more ONT basecallers on raw fast5 test sets — e.g. compiling a basecaller-by-dataset table of identity/indel/mismatch rates like Table 1 of the Chiron paper.

## When NOT to use

- Input is already FASTQ/basecalled reads and no basecaller comparison is needed — use a variant-calling or assembly-error-rate method instead
- You need consensus-level or assembly-level accuracy (e.g. after Minimap2/Miniasm + 10 rounds of Racon polishing), not raw read-level basecalling accuracy
- The fast5 raw signal has not been basecalled by any method yet — run a basecaller (this skill quantifies basecaller output, it does not produce it)

## Inputs

- Raw nanopore fast5 read folders (test sets, e.g. Lambda, E. coli, M. tuberculosis PRJNA386696, Human NA12878)
- Reference genome sequences (e.g. NC_001416.1 for Phage Lambda)
- Basecalled FASTQ output folders from each basecaller (Chiron beam/greedy, Albacore, BasecRAWller, Metrichor)

## Outputs

- Per-read identity, deletion, insertion, and mismatch/error rates for each basecaller-dataset combination
- Compiled CSV table (basecaller rows x dataset columns) reproducing the paper's Table 1 layout

## How to apply

Basecall each fast5 read set with every basecaller under comparison (Chiron with the DNA_default model and dna-pre preset, batch_size 400, segment_len 400, jump 390, run twice: beam-search decoder --beam 30 and greedy decoder --beam 0; Albacore v1.1.1/v2.0.1, BasecRAWller v0.1, and Metrichor with default parameters), writing FASTQ output. Align each basecaller's reads to the matching reference (e.g. Phage Lambda NC_001416.1, E. coli, M. tuberculosis PRJNA386696, Human NA12878) using graphmap with default parameters. From each alignment compute per-read identity, deletion, insertion, and mismatch/error rates, then aggregate into a CSV table of basecaller rows x dataset columns. Rationale: Chiron's beam search decoder gives higher accuracy than greedy decoding, so both decoders should be evaluated separately; validation checks that the expected pattern reproduces (Chiron matching Albacore v2.0.1 on bacterial/viral data, and within ~2% of Albacore on Human DNA).

## Related tools

- **Chiron** (Deep-learning basecaller whose beam-search (--beam 30) and greedy (--beam 0) decoder outputs are quantified and compared) — https://github.com/haotianteng/Chiron
- **Albacore** (ONT basecaller (v1.1.1 and v2.0.1) run with default parameters as an accuracy benchmark)
- **BasecRAWller** (RNN-based basecaller (v0.1, default parameters) included in the read-level accuracy comparison)
- **Metrichor** (ONT cloud basecaller used as an additional comparison baseline)
- **graphmap** (Aligner used to map basecalled reads to the reference genome with default parameters, from which error rates are computed)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default -p dna-pre --beam 30
```

## Evaluation signals

- The alignment-based table reproduces the reported pattern: Chiron best or second-best in read-level accuracy on every tested dataset
- Chiron identity rate matches Albacore v2.0.1 on bacterial and viral data (within ~1%) but shows the known ~2% gap on Human DNA
- Beam-search decoding (--beam 30) yields equal or higher identity than greedy decoding (--beam 0) for the same reads
- All error components (identity, insertion, deletion, mismatch) are present for every basecaller-dataset cell in the CSV, summing consistently to 100% identity accounting
- Per-read rates are computed for every read in each FASTQ (no silent read loss between basecalling and alignment)

## Limitations

- Albacore is proprietary, which limits reproducibility and reuse of the comparison
- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller
- Earlier versions of MinKNOW were used for non-E. coli samples (E. coli was run on MinKNOW v1.4.3), introducing a software-version inconsistency across test datasets

## Evidence

- [discussion] aligned to the reference genome using graphmap[33] with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [methods] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [readme] Beam Search decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
- [discussion] Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA.: "Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA."
