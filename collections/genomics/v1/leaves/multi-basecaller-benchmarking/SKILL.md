---
name: multi-basecaller-benchmarking
description: Use when comparing the read-level basecalling accuracy of several nanopore
  basecallers across species datasets (e.g. Lambda, E. coli, M. tuberculosis, Human)
  using identity/error-rate metrics.
when_to_use_negative:
- Input data are already basecalled (e.g. fastq/fast5 only with reads) — this skill
  benchmarks raw-signal basecallers, it does not evaluate read alignment or assembly
- You need a consensus-level or assembly-level accuracy comparison — the reported
  validation metric is read-level accuracy, not consensus accuracy
- The basecaller under test does not output fastq/fasta suitable for alignment to
  a reference genome
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Chiron
  role: Candidate basecaller under test; run with DNA_default model and dna-pre preset,
    both beam search (--beam 30) and greedy (--beam 0) decoders
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: Reference ONT basecaller benchmark (v1.1.1 and v2.0.1, default parameters)
- name: Metrichor
  role: Reference cloud-service ONT basecaller benchmark
- name: BasecRAWller
  role: Reference RNN-based basecaller benchmark (version 0.1, default parameters)
- name: graphmap
  role: Aligns basecalled reads from each basecaller to the reference genome with
    default parameters to compute per-read accuracy statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:55750fd61e867f8b542021a1ddaa25ed131e0c1bc1ae1438ab293ebb2a9a6989
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-basecaller-benchmarking

## Summary

Benchmarks multiple nanopore basecallers (Chiron, Albacore, Metrichor, BasecRAWller) on the same fast5 read sets by aligning their fastq output to reference genomes and computing per-read identity, insertion, deletion, and mismatch rates.

## When to use

Use when comparing the read-level basecalling accuracy of several nanopore basecallers across species datasets (e.g. Lambda, E. coli, M. tuberculosis, Human) using identity/error-rate metrics.

## When NOT to use

- Input data are already basecalled (e.g. fastq/fast5 only with reads) — this skill benchmarks raw-signal basecallers, it does not evaluate read alignment or assembly
- You need a consensus-level or assembly-level accuracy comparison — the reported validation metric is read-level accuracy, not consensus accuracy
- The basecaller under test does not output fastq/fasta suitable for alignment to a reference genome

## Inputs

- Test fast5 read sets for Phage Lambda (NC_001416.1), E. coli, M. tuberculosis (PRJNA386696), and Human NA12878 from the GigaDB Chiron supporting data deposit
- Reference genome sequences for each test dataset
- Reference basecaller outputs from Metrichor, Albacore v1.1.1/v2.0.1, and BasecRAWller v0.1 (default parameters)

## Outputs

- Per-read identity, deletion, insertion, and mismatch rates for every basecaller-dataset combination
- A single CSV table with basecaller rows x dataset columns reproducing the paper's Table 1 layout

## How to apply

Download the test fast5 read sets from the GigaDB Chiron supporting data deposit (Lambda NC_001416.1, E. coli, M. tuberculosis PRJNA386696, Human NA12878). Basecall each read set with Chiron using the DNA_default model and dna-pre preset (batch_size 400, segment_len 400, jump 390, beam 30) once with the beam search decoder (--beam 30) and once with the greedy decoder (--beam 0), writing fastq. Basecall the same read sets with Metrichor, Albacore v1.1.1 and v2.0.1, and BasecRAWller v0.1, all with default parameters. Align the basecalled reads from each basecaller to the corresponding reference genome with graphmap default parameters, then compute per-read identity, deletion, insertion, and mismatch rates from the alignments. Compile per-basecaller, per-dataset statistics into a single CSV table matching the Table 1 layout. Decision point: greedy decoding is faster (1652 bp/s on GPU) while beam search decoding (beam width 30) gives higher accuracy.

## Related tools

- **Chiron** (Candidate basecaller under test; run with DNA_default model and dna-pre preset, both beam search (--beam 30) and greedy (--beam 0) decoders) — https://github.com/haotianteng/Chiron
- **Albacore** (Reference ONT basecaller benchmark (v1.1.1 and v2.0.1, default parameters))
- **Metrichor** (Reference cloud-service ONT basecaller benchmark)
- **BasecRAWller** (Reference RNN-based basecaller benchmark (version 0.1, default parameters))
- **graphmap** (Aligns basecalled reads from each basecaller to the reference genome with default parameters to compute per-read accuracy statistics)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default -p dna-pre --beam 30
```

## Evaluation signals

- Chiron achieves the highest identity rate on the Lambda, E. coli, and M. tuberculosis samples
- On the Human dataset, Chiron is within 0.01 of the best method's identity rate, with a known accuracy gap of up to 2% vs Albacore on Human DNA
- Chiron matches Albacore v2.0.1 accuracy on bacterial and viral basecalling and outperforms all other existing methods
- The compiled table reproduces the Table 1 pattern: Chiron best or second-best in read-level accuracy on all tested datasets
- Beam-search decoding (beam 30) yields equal or higher identity than greedy decoding (--beam 0) for the same input reads

## Limitations

- Chiron's basecalling is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller
- Chiron has a known accuracy gap of up to 2% vs Albacore on Human DNA, since it was trained only on viral and bacterial reads
- Earlier MinKNOW versions were used for non-E. coli samples (E. coli was run on MinKNOW version 1.4.3), introducing a software version inconsistency across datasets

## Evidence

- [discussion] All basecalling with Albacore (v1.1.1 and v2.0.1) and BasecRAWller (version 0.1) was done with default parameters.: "All basecalling with Albacore (v1.1.1 and v2.0.1) and BasecRAWller [13] (version 0.1) was done with default parameters."
- [discussion] aligned to the reference genome using graphmap with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [methods] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [discussion] The E. coli sample was run on the MinKNOW, version 1.4.3, and the other samples in earlier versions of the software.: "The E. coli sample was run on the MinKNOW, version 1.4.3, and the other samples in earlier versions of the software."
- [readme] Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.
Greedy decoder give a faster decoding speed than the beam search decoder"
