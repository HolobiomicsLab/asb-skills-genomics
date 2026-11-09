---
name: insertion-deletion-mismatch-quantification
description: Use when you need per-error-type (insertion/deletion/mismatch) read-level
  profiles of a basecaller or sequencing protocol, rather than a single aggregate
  accuracy number.
when_to_use_negative:
- You only need an overall read-level accuracy or identity score — a single alignment-based
  accuracy metric suffices and this error-type decomposition is unnecessary.
- Your reads cannot be aligned to a reference genome (no reference available), since
  CIGAR/MD-based error decomposition requires reference alignment.
- Comparing raw signal-level quality rather than basecalled sequence errors — the
  segmentation/decoder behavior is not captured by post-hoc CIGAR parsing.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: Beam-search-decoding basecaller whose output FASTQ reads are profiled per
    error type
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: Comparison basecaller (v2.0.1, default parameters) whose reads are profiled
    for error types
- name: graphmap
  role: Aligner used to map basecalled reads to each reference genome with default
    parameters, producing the CIGAR/MD strings parsed for error counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:200d93a4cff9606e9d32402b78d69f6594d73b6b5b0ea48d8f96f10280d99880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - Chiron
  - graphmap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# insertion-deletion-mismatch-quantification

## Summary

Quantifies per-read insertion, deletion, and mismatch rates by parsing CIGAR/MD strings from read-to-reference alignments, as done to profile Chiron-BS versus Albacore v2.0.1 error profiles across Lambda, E. coli, M. tuberculosis, and Human nanopore reads.

## When to use

Use when you need per-error-type (insertion/deletion/mismatch) read-level profiles of a basecaller or sequencing protocol, rather than a single aggregate accuracy number.

## When NOT to use

- You only need an overall read-level accuracy or identity score — a single alignment-based accuracy metric suffices and this error-type decomposition is unnecessary.
- Your reads cannot be aligned to a reference genome (no reference available), since CIGAR/MD-based error decomposition requires reference alignment.
- Comparing raw signal-level quality rather than basecalled sequence errors — the segmentation/decoder behavior is not captured by post-hoc CIGAR parsing.

## Inputs

- Basecalled FASTQ reads from each basecaller (e.g., Chiron-BS beam-search output and Albacore v2.0.1 output)
- Reference genome sequences for each tested species (e.g., Phage Lambda NC_001416.1)
- graphmap alignments of reads against each reference (CIGAR/MD strings)

## Outputs

- Per-read insertion, deletion, and mismatch rates (counts normalized by read length)
- Aggregated error_rate_profiles.tsv: mean insertion/deletion/mismatch rate per basecaller per species

## How to apply

Basecall the raw FAST5 signal with the basecaller under comparison (e.g., Chiron with beam search decoding, default beam width 30 from the dna-pre preset, vs. Albacore v2.0.1 with default parameters). Align the basecalled FASTQ reads to each species' reference genome (e.g., Phage Lambda NC_001416.1, E. coli, M. tuberculosis, Human NA12878) using graphmap with default parameters. Parse each alignment's CIGAR/MD strings to count insertions, deletions, and mismatches per read, normalizing each count by read length to obtain per-read error rates. Aggregate per-read rates into mean insertion, deletion, and mismatch rates per basecaller per species (written to a table such as error_rate_profiles.tsv) and compare directional patterns between basecallers — in the Chiron study this revealed Chiron-BS had consistently lower insertion rates but slightly higher mismatch rates on all species except E. coli.

## Related tools

- **Chiron** (Beam-search-decoding basecaller whose output FASTQ reads are profiled per error type) — https://github.com/haotianteng/Chiron
- **Albacore** (Comparison basecaller (v2.0.1, default parameters) whose reads are profiled for error types)
- **graphmap** (Aligner used to map basecalled reads to each reference genome with default parameters, producing the CIGAR/MD strings parsed for error counts)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- Output table contains all four species (Lambda, E. coli, M. tuberculosis, Human) for both basecallers
- Directional pattern reproduced: Chiron-BS insertion rate lower than Albacore v2.0.1 across all species
- Chiron-BS mismatch rate is slightly higher than Albacore v2.0.1 on all species except E. coli
- Per-read error rates are normalized by read length (counts/read_length), not raw counts
- Each alignment CIGAR/MD string yields three mutually consistent count categories (I, D, mismatch) per read

## Limitations

- Alignment-based error quantification inherits aligner bias — graphmap with default parameters is used, and read alignment errors can inflate apparent mismatch/indel rates.
- Comparisons are read-level only; consensus/assembly accuracy can differ, and the paper notes accuracy gains from better alignment of overlapping reads and consensus calling.
- Error-profile trade-offs mean no basecaller is uniformly superior — Chiron-BS's lower insertion rate comes with a slightly higher mismatch rate on most species.

## Evidence

- [discussion] aligned to the reference genome using graphmap[33] with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [readme] beam search decoding with default beam width 30 from dna-pre preset: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [results] Chiron-BS lower insertion rate but higher mismatch rate trade-off pattern: "Chiron-BS had a consistently lower insertion rate across all species tested, as well as a lower deletion rate on Lambda and E. coli"
- [readme] Beam search vs greedy decoder choice in Chiron: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
