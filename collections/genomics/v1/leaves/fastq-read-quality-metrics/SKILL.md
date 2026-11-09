---
name: fastq-read-quality-metrics
description: Use when basecalled FASTQ reads from ONT basecallers (e.g., Chiron beam-search
  vs Albacore) must be compared by error type—insertions, deletions, mismatches—per
  species, rather than by overall read identity alone.
when_to_use_negative:
- Reads cannot be aligned to a reference genome (no suitable reference available),
  since the metrics are derived from alignment CIGAR/MD strings
- Only overall read-level accuracy/identity is needed and the insertion/deletion/mismatch
  decomposition is irrelevant
- Input is raw fast5 signal that has not been basecalled to FASTQ by a basecaller
  such as Chiron or Albacore
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: End-to-end CNN+RNN+CTC basecaller producing the FASTQ reads; beam search decoding
    (--beam, default 30 in dna-pre preset) yields higher accuracy reads whose error
    profile is measured
  repo: https://github.com/haotianteng/chiron
- name: graphmap
  role: Aligns basecalled reads to the species reference genome with default parameters;
    the alignments' CIGAR/MD strings are parsed to count error types
- name: Albacore v2.0.1
  role: Proprietary ONT comparator basecaller run with default parameters to produce
    baseline FASTQ reads for error-profile comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:91e44ce2ef8de0fe99080509d288cf25cbd8cc2c5fb9289fb9f11434d6e9ea98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Albacore
  - Albacore v2.0.1
  - Chiron
  - graphmap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fastq-read-quality-metrics

## Summary

Computes per-read error profiles (insertion, deletion, and mismatch rates) for ONT basecalled FASTQ reads by aligning them to a reference genome with graphmap and parsing the CIGAR/MD strings, enabling error-type-level comparison between basecallers such as Chiron-BS and Albacore v2.0.1.

## When to use

Use when basecalled FASTQ reads from ONT basecallers (e.g., Chiron beam-search vs Albacore) must be compared by error type—insertions, deletions, mismatches—per species, rather than by overall read identity alone.

## When NOT to use

- Reads cannot be aligned to a reference genome (no suitable reference available), since the metrics are derived from alignment CIGAR/MD strings
- Only overall read-level accuracy/identity is needed and the insertion/deletion/mismatch decomposition is irrelevant
- Input is raw fast5 signal that has not been basecalled to FASTQ by a basecaller such as Chiron or Albacore

## Inputs

- Basecalled FASTQ reads per species per basecaller (e.g., Chiron beam-search output with beam width 30; Albacore v2.0.1 default output)
- Reference genome sequences per species (e.g., Phage Lambda NCBI reference sequence NC_001416.1)
- graphmap alignment output (BAM/SAM with CIGAR and MD strings)

## Outputs

- Per-read insertion, deletion, and mismatch rates normalized by read length
- Aggregated mean error-rate table per basecaller per species (error_rate_profiles.tsv)

## How to apply

Basecall the raw fast5 signal for each species (Lambda, E. coli, M. tuberculosis, Human) with Chiron using beam search decoding (dna-pre preset defaults to beam width 30), and produce the comparator reads with Albacore v2.0.1 at default parameters. Align each basecaller's FASTQ output to its species reference genome (e.g., Phage Lambda NC_001416.1) using graphmap with default parameters, as in the Chiron paper's evaluation pipeline. Parse each alignment's CIGAR/MD strings to count insertions, deletions, and mismatches per read, normalizing each count by read length to obtain per-read error rates. Aggregate per-read rates into mean insertion, deletion, and mismatch rates per basecaller per species and write the table (e.g., error_rate_profiles.tsv). Rationale: read-level accuracy alone hides error-profile trade-offs—Chiron-BS showed consistently lower insertion rates but slightly higher mismatch rates than Albacore v2.0.1 on all species except E. coli, which per-error-type decomposition reveals.

## Related tools

- **Chiron** (End-to-end CNN+RNN+CTC basecaller producing the FASTQ reads; beam search decoding (--beam, default 30 in dna-pre preset) yields higher accuracy reads whose error profile is measured) — https://github.com/haotianteng/chiron
- **graphmap** (Aligns basecalled reads to the species reference genome with default parameters; the alignments' CIGAR/MD strings are parsed to count error types)
- **Albacore v2.0.1** (Proprietary ONT comparator basecaller run with default parameters to produce baseline FASTQ reads for error-profile comparison)

## Examples

```
chiron call -i <input> -o <output> --beam 30
```

## Evaluation signals

- Output table contains all four species (Lambda, E. coli, M. tuberculosis, Human) for both basecallers
- Directional pattern reproduced: Chiron-BS has lower insertion rate than Albacore v2.0.1 on all species tested
- Chiron-BS mismatch rate is higher than Albacore v2.0.1 on all species except E. coli
- Deletion rate is lower for Chiron-BS on Lambda and E. coli
- Per-read rates are normalized by read length (each rate in [0,1]) before aggregation into means

## Limitations

- The result is an error-profile trade-off rather than uniform superiority: Chiron-BS trades lower insertions for a slightly higher mismatch rate on all species except E. coli
- Beam search decoding is slower than greedy decoding (README: 21 vs 17 bp/s on CPU; 1652 vs 1204 bp/s on GPU at beam_width=50)
- Albacore is proprietary, limiting open comparison/reuse; Chiron remains within 1% accuracy on bacterial DNA but only within 2% on Human DNA
- High computational cost of Chiron relative to Albacore and BasecRAWller can limit throughput of repeated basecalling comparisons

## Evidence

- [discussion] aligned to the reference genome using graphmap[33] with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [methods] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [readme] Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
- [intro] Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods: "Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods"
