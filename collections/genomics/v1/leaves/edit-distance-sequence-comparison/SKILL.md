---
name: edit-distance-sequence-comparison
description: Use when comparing assembled long-read consensus sequences (e.g. MinION
  rDNA amplicons) against a high-accuracy reference consensus and you need a single
  per-sequence error metric for accuracy/coverage sweeps.
when_to_use_negative:
- Input is a bulk-sample metabarcoding feature table or abundance matrix — the article
  notes long rDNA amplicon sequencing is not yet reliable for bulk sample analysis,
  and edit distance to a single reference consensus is meaningless there
- You need alignment with quality-score weighting or homopolymer-aware error modeling
  — Edlib computes unweighted Levenshtein distance only
- Reads still carry barcodes and primers — trim with MiniBar -T first, otherwise the
  untrimmed ends inflate the measured edit distance
edam_operation: http://edamontology.org/operation_0258
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: Edlib
  role: Computes the edit distance (Levenshtein distance) between Nanopore consensus
    sequences and the Illumina reference consensus via its Python module
  repo: https://github.com/Martinsos/edlib
- name: MiniBar
  role: Prepares the input by demultiplexing dual-indexed MinION reads into per-sample
    files and trimming barcodes/primers before consensus assembly and comparison
  repo: https://github.com/calacademy-research/minibar
- name: Python 2.7
  role: Language in which the subsampling, consensus assembly, and Edlib-based distance
    calculation are scripted (also runs in Python 3)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:20b2f8165c5aafd4a7150bf38f5e7f59756e97f4859813c83c278f5331930a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Martinsos/edlib
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0258
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# edit-distance-sequence-comparison

## Summary

Quantifies sequence similarity between DNA consensus sequences (e.g. Nanopore rDNA consensus vs. Illumina-derived reference consensus) using Levenshtein edit distance computed with the Edlib library, expressing distances as percent error relative to consensus length.

## When to use

Use when comparing assembled long-read consensus sequences (e.g. MinION rDNA amplicons) against a high-accuracy reference consensus and you need a single per-sequence error metric for accuracy/coverage sweeps.

## When NOT to use

- Input is a bulk-sample metabarcoding feature table or abundance matrix — the article notes long rDNA amplicon sequencing is not yet reliable for bulk sample analysis, and edit distance to a single reference consensus is meaningless there
- You need alignment with quality-score weighting or homopolymer-aware error modeling — Edlib computes unweighted Levenshtein distance only
- Reads still carry barcodes and primers — trim with MiniBar -T first, otherwise the untrimmed ends inflate the measured edit distance

## Inputs

- Barcode- and primer-trimmed FASTQ of a single specimen (MiniBar -F -T output)
- Illumina-derived reference consensus sequence (nucleotide string/FASTA)
- Assembled Nanopore majority-vote consensus sequences (nucleotide strings)

## Outputs

- Per-consensus percent error values (edit distance / consensus length)
- Aggregated table of mean consensus error vs. number of reads (consensus_accuracy_vs_coverage.csv)

## How to apply

After MiniBar demultiplexing (-F binning with a prefix like Peperomia_ and -T trimming of barcodes/primers from read ends), assemble a majority-vote consensus per base position from the multiple read alignment. For each consensus, compute the edit distance to the Illumina-derived reference consensus using the Edlib Python bindings (edlib.align with global/NW method), which implements Myers's bit-vector algorithm so whole-amplicon alignments are cheap. Convert the raw edit distance to percent error by dividing by the consensus length, then aggregate replicate error rates (e.g. 10 subsamples per coverage level from 10 to 800 reads) into means and spread. The expected behavior is a monotone decrease in percent error with depth: roughly 0.257% at 10 reads, 0.128% at 20 reads, and an optimum near 0.031% around 300 reads before plateauing.

## Related tools

- **Edlib** (Computes the edit distance (Levenshtein distance) between Nanopore consensus sequences and the Illumina reference consensus via its Python module) — https://github.com/Martinsos/edlib
- **MiniBar** (Prepares the input by demultiplexing dual-indexed MinION reads into per-sample files and trimming barcodes/primers before consensus assembly and comparison) — https://github.com/calacademy-research/minibar
- **Python 2.7** (Language in which the subsampling, consensus assembly, and Edlib-based distance calculation are scripted (also runs in Python 3))

## Examples

```
import edlib; dist = edlib.align(consensus, illumina_reference, mode="NW", task="distance").editDistance; pct_err = dist / len(consensus) * 100
```

## Evaluation signals

- Error-rate trend must decrease with depth: ~0.257% at 10 reads, ~0.128% at 20 reads, optimal ~0.031% near 300 reads, then plateau toward 800 reads
- Each consensus's edit distance must be a non-negative integer no greater than the longer sequence length; percent error = distance / consensus length
- MiniBar run summary should report identified samples (IDs count) with few Mult_IDs, e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0'
- Output CSV must contain one mean (and spread) per coverage level across the 18 levels spanning 10 to 800 sequences, with 10 independent subsamples each

## Limitations

- Edit distance is unweighted (unit cost for substitution, insertion, deletion), so it treats all Nanopore error types equally
- Accuracy saturates near 300-fold coverage; adding reads beyond the optimum yields diminishing returns
- Edlib assumes the Python module is installed separately (pip install edlib); minibar.py is written for Python 2.7 but runs in Python 3

## Evidence

- [readme] Calculating edit distance of two strings is as simple as: [evidence span withheld — anchor status: external_doc]
- [discussion] Other requirements: Edlib library module: "Other requirements: Edlib library module (https://github.com/Martinsos/edlib)"
- [readme] Edlib handles very large sequences with low memory: [evidence span withheld — anchor status: not_found]
- [readme] MiniBar -T trims barcodes and primers: "-T trims barcode and primer from each end of the sequence, then outputs record"
