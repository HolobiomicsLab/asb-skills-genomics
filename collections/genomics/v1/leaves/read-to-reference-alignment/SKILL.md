---
name: read-to-reference-alignment
description: 'Use when comparing basecaller accuracy: you have basecalled FASTQ reads
  from one or more basecallers and the corresponding reference genome, and you need
  per-read identity/error-rate metrics.'
when_to_use_negative:
- No reference genome exists for the sample — use the de novo route (Minimap2/Miniasm
  draft + 10 rounds of Racon polishing) instead of read-to-reference alignment
- Input is raw fast5 signal rather than basecalled reads — basecall first (e.g. Chiron
  dna-pre preset) before aligning
- Reads have already been aligned and error rates extracted — re-running graphmap
  alignment would duplicate the analysis
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: graphmap
  role: Pairwise aligner used to map basecalled reads to the reference genome with
    default parameters; also used to align polished assemblies to references to compute
    assembly error rates
- name: Chiron
  role: Source of basecalled FASTQ reads (beam-search, --beam 30, or greedy, --beam
    0 decoders) that are aligned for benchmarking
  repo: https://github.com/haotianteng/Chiron
- name: Albacore
  role: ONT baseline basecaller (v1.1.1 and v2.0.1, default parameters) whose basecalled
    reads are aligned for comparison
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:e47be07e075cb3bb28b4fd04c085964575a264098fa231fa735766db8412dabf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - BasecRAWller
  - Chiron
  - Metrichor
  - Miniasm
  - Minimap2
  - Racon
  - graphmap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# read-to-reference-alignment

## Summary

Aligns basecalled nanopore reads (FASTQ/FASTA from Chiron, Albacore, Metrichor, or BasecRAWller) to a known reference genome with graphmap using default parameters, then parses alignment CIGAR/MD strings to derive per-read identity, insertion, deletion, and mismatch rates for basecaller benchmarking.

## When to use

Use when comparing basecaller accuracy: you have basecalled FASTQ reads from one or more basecallers and the corresponding reference genome, and you need per-read identity/error-rate metrics.

## When NOT to use

- No reference genome exists for the sample — use the de novo route (Minimap2/Miniasm draft + 10 rounds of Racon polishing) instead of read-to-reference alignment
- Input is raw fast5 signal rather than basecalled reads — basecall first (e.g. Chiron dna-pre preset) before aligning
- Reads have already been aligned and error rates extracted — re-running graphmap alignment would duplicate the analysis

## Inputs

- Basecalled FASTQ/FASTA read files per basecaller (e.g. Chiron beam-search with beam 30, Chiron greedy --beam 0, Albacore v1.1.1, Albacore v2.0.1, BasecRAWller v0.1)
- Reference genome sequences (e.g. Phage Lambda NC_001416.1, E. coli, M. tuberculosis, Human NA12878)

## Outputs

- Per-read identity, deletion, insertion, and mismatch/error rates per basecaller-dataset combination (CSV/TSV, e.g. error_rate_profiles.tsv or a Table 1-style basecaller rows x dataset columns table)
- Alignment files (one set per basecaller-species pair) usable for downstream assembly error-rate computation

## How to apply

For each basecaller-dataset combination (e.g. Chiron beam-search vs greedy vs Albacore v1.1.1/v2.0.1 on Phage Lambda NC_001416.1, E. coli, M. tuberculosis PRJNA386696, and Human NA12878), align the basecalled FASTQ reads to the matching reference genome using graphmap with default parameters. Parse each alignment's CIGAR/MD strings to count insertions, deletions, and mismatches per read, normalizing counts by read length to obtain per-read identity, deletion, insertion, and mismatch/error rates. Aggregate per-read rates into mean rates per basecaller per species and write them to a table (e.g. error_rate_profiles.tsv) laid out as basecaller rows x dataset columns. The same alignment step also feeds downstream assembly error-rate computation, where polished assemblies are aligned back to the reference with graphmap to compute assembly error rates.

## Related tools

- **graphmap** (Pairwise aligner used to map basecalled reads to the reference genome with default parameters; also used to align polished assemblies to references to compute assembly error rates)
- **Chiron** (Source of basecalled FASTQ reads (beam-search, --beam 30, or greedy, --beam 0 decoders) that are aligned for benchmarking) — https://github.com/haotianteng/Chiron
- **Albacore** (ONT baseline basecaller (v1.1.1 and v2.0.1, default parameters) whose basecalled reads are aligned for comparison)

## Evaluation signals

- The output table contains all four species (Lambda, E. coli, M. tuberculosis, Human) for every basecaller, with per-read rates normalized by read length
- Read-level identity rates reproduce the reported pattern: Chiron best or within 0.01 of the best method, matching Albacore v2.0.1 on bacterial and viral DNA
- Error-profile directionality holds: Chiron-BS shows a consistently lower insertion rate than Albacore v2.0.1 and a slightly higher mismatch rate on all species except E. coli
- Aggregate error rates reconcile with de novo results: Chiron assembly error rates below 0.5% vs Albacore up to 0.8% on Lambda and E. coli
- Human DNA shows the known accuracy gap (Chiron within ~2% of Albacore), confirming reads were aligned to the correct NA12878 reference data

## Limitations

- The Human comparison carries a known ~2% accuracy gap vs Albacore because Chiron's training set contained no Human DNA
- Earlier versions of MinKNOW were used for non-E. coli samples (E. coli ran on MinKNOW v1.4.3), a software version inconsistency across datasets
- graphmap is run with default parameters; the article gives no tuning guidance for the alignment step itself

## Evidence

- [discussion] graphmap alignment of basecalled reads: "aligned to the reference genome using graphmap[33] with the default parameters"
- [discussion] graphmap also used for assembly error-rate computation: "We used Minimap2 [34] and Miniasm [35] to generate a draft genome, then Racon [36] was used to polish on the draft genome for 10 rounds."
- [intro] accuracy comparison across basecallers is the purpose of the alignment: "Chiron is as accurate as the ONT-designed and -trained Albacore v2.0.1 on bacterial and viral basecalling and outperforms all other existing methods"
- [readme] Chiron reads to be aligned are produced with specific decoder settings: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [discussion] read-level accuracy scope across datasets: "Our model was either the best or second-best in terms of accuracy on all of the datasets we tested in terms of read-level accuracy."
