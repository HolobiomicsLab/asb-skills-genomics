---
name: cross-basecaller-accuracy-comparison
description: Use when two or more basecallers must be compared on the same raw FAST5
  datasets and the goal is a per-error-type (insertion/deletion/mismatch) read-accuracy
  profile across species or datasets.
when_to_use_negative:
- When the comparison target is assembly or consensus accuracy rather than per-read
  error profiles — that analysis used Minimap2/Miniasm plus 10 rounds of Racon polishing,
  a different workflow
- When only overall read-level accuracy identity scores are needed and per-error-type
  decomposition (insertion vs deletion vs mismatch) is not required
- When reads cannot be aligned to a common reference (e.g. no reference genome available),
  since error decomposition depends on CIGAR/MD strings
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: Deep-learning basecaller producing FASTQ reads with beam-search decoding (--beam
    30 in dna-pre preset) for one comparison arm
  repo: https://github.com/haotianteng/chiron
- name: Albacore
  role: Proprietary ONT basecaller (v2.0.1) run with default parameters as the reference
    comparison arm
- name: graphmap
  role: Aligner used with default parameters to map each basecaller's reads to the
    species reference genome, producing CIGAR/MD strings for error counting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:09c617e5e93f2a5740878f6a286ef8a64efe2a162b793e4eca91222ed35934ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Albacore
  - Chiron
  - graphmap
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-basecaller-accuracy-comparison

## Summary

Compares read-level accuracy of nanopore basecallers (e.g. Chiron with beam-search decoding versus ONT Albacore v2.0.1) by aligning each basecaller's FASTQ output to a common reference with graphmap and decomposing errors into per-read insertion, deletion, and mismatch rates.

## When to use

Use when two or more basecallers must be compared on the same raw FAST5 datasets and the goal is a per-error-type (insertion/deletion/mismatch) read-accuracy profile across species or datasets.

## When NOT to use

- When the comparison target is assembly or consensus accuracy rather than per-read error profiles — that analysis used Minimap2/Miniasm plus 10 rounds of Racon polishing, a different workflow
- When only overall read-level accuracy identity scores are needed and per-error-type decomposition (insertion vs deletion vs mismatch) is not required
- When reads cannot be aligned to a common reference (e.g. no reference genome available), since error decomposition depends on CIGAR/MD strings

## Inputs

- Raw FAST5 nanopore signal files from each species (Lambda, E. coli, M. tuberculosis, Human NA12878)
- Basecalled FASTQ reads from each basecaller (e.g. Chiron beam-search output, Albacore v2.0.1 output)
- Reference genome sequences (e.g. NCBI reference NC_001416.1 for Phage Lambda)
- Alignment CIGAR/MD strings from graphmap mapping of each read set

## Outputs

- Per-read normalized insertion, deletion, and mismatch rates for each basecaller-species pair
- Aggregated error_rate_profiles.tsv table of mean insertion/deletion/mismatch rates per basecaller per species
- Directional comparison summary of error-profile trade-offs between basecallers

## How to apply

Basecall the same FAST5 inputs with each basecaller under comparable settings: for Chiron use the dna-pre preset (default beam width 30) for beam-search decoding, and default parameters for Albacore v2.0.1. Align each basecaller's FASTQ reads to the species reference genome (e.g. Lambda NC_001416.1, E. coli, M. tuberculosis, Human NA12878) using graphmap with default parameters. Parse the alignment CIGAR/MD strings to count insertions, deletions, and mismatches per read, normalizing each count by read length to get per-read error rates. Aggregate per-read rates into mean insertion, deletion, and mismatch rates per basecaller per species, and compare directional patterns (e.g. Chiron-BS lower insertion rate but slightly higher mismatch rate than Albacore on all species except E. coli) to characterize accuracy trade-offs rather than assuming uniform superiority.

## Related tools

- **Chiron** (Deep-learning basecaller producing FASTQ reads with beam-search decoding (--beam 30 in dna-pre preset) for one comparison arm) — https://github.com/haotianteng/chiron
- **Albacore** (Proprietary ONT basecaller (v2.0.1) run with default parameters as the reference comparison arm)
- **graphmap** (Aligner used with default parameters to map each basecaller's reads to the species reference genome, producing CIGAR/MD strings for error counting)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- Output table contains all four species (Lambda, E. coli, M. tuberculosis, Human) for both basecallers
- Directional pattern reproduced: Chiron-BS has a consistently lower insertion rate across all species than Albacore v2.0.1
- Chiron-BS mismatch rate is slightly higher than Albacore on all species except E. coli
- Every per-read rate is normalized by read length before aggregation into the mean per basecaller/species
- Alignments were performed with graphmap default parameters identically for both basecallers' read sets

## Limitations

- Chiron was within 1% accuracy on bacterial DNA but only within 2% of Albacore on Human DNA, reflecting an accuracy gap on species absent from training
- Albacore is proprietary, limiting open reuse of the comparison
- Chiron beam-search decoding is substantially more computationally expensive than Albacore; beam search decoding rate is ~1204 bp/s on GPU versus greedy's 1652 bp/s
- Training labels derived from Metrichor basecalling introduce errors that must be corrected via alignment to avoid learning Metrichor's bias
- Earlier versions of MinKNOW were used for non-E. coli samples, a software version inconsistency across samples

## Evidence

- [discussion] aligned to the reference genome using graphmap[33] with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [intro] Chiron is as accurate as Albacore v2.0.1 on bacterial and viral basecalling: "Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods"
- [discussion] Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA.: "Chiron is within 1% accuracy on bacterial DNA but only within 2% accuracy on Human DNA."
- [readme] Beam search decoder command from Chiron README: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [readme] Beam search gives higher accuracy; larger beam width can further improve accuracy: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
