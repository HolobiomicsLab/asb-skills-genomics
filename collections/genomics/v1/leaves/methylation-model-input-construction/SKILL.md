---
name: methylation-model-input-construction
description: Use when Nanopolish eventalign output must be converted into CHEUI Model
  1 input files — per-read 9-mer signal vectors (100 values) plus distance vectors
  — for m6A (centered on A) or m5C (centered on C) read-level prediction.
when_to_use_negative:
- The input is already a preprocessed 9-mer signal/distance vector file (e.g., cheui_model1_input_signals.tsv)
  — proceed directly to CHEUI_predict_model1 instead of re-preprocessing
- Data are from the RNA004 chemistry or require pseudouridine calling — the CHEUI
  README directs those cases to SWARM (https://github.com/comprna/SWARM), which supersedes
  CHEUI for RNA002 accuracy and supports RNA004
- The goal is site-level stoichiometry or differential methylation between conditions
  — those are handled downstream by CHEUI Model 2 or CHEUI-diff, not by this preprocessing
  skill
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CHEUI
  role: Provides the preprocessing scripts (CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py),
    the 5-mer k-mer model (model_kmer.csv), and the Model 1 .h5 networks that consume
    the 100-value signal + distance vectors
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish eventalign
  role: Resquiggles and aligns read signals to the matched transcript references,
    emitting per-5-mer sample lists with --scale-events --signal-index --samples --print-read-names
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: Minimap2
  role: Aligns Guppy-basecalled reads to the reference transcriptome, producing the
    sorted BAM (primary, positive-strand alignments) that eventalign consumes
- name: Guppy
  role: Basecalls FAST5 raw signals to FASTQ (template_rna_r9.4.1_70bps models, 4.0.11+)
    as the first step before signal-to-transcript alignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:702f89b369deac2a6439b30f850f30168a6a3bebba9a1a6ecabf8008a4da2514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Guppy
  - Minimap2
  - Nanopolish eventalign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# methylation-model-input-construction

## Summary

Reconstructs CHEUI's Nanopore Signal Preprocessing step: converting Nanopolish eventalign ionic-current signals into per-read 9-mer observed-signal and distance vectors (5 × 20 = 100 values each) that feed CHEUI-solo Model 1 for m6A and m5C prediction.

## When to use

Use when Nanopolish eventalign output must be converted into CHEUI Model 1 input files — per-read 9-mer signal vectors (100 values) plus distance vectors — for m6A (centered on A) or m5C (centered on C) read-level prediction.

## When NOT to use

- The input is already a preprocessed 9-mer signal/distance vector file (e.g., cheui_model1_input_signals.tsv) — proceed directly to CHEUI_predict_model1 instead of re-preprocessing
- Data are from the RNA004 chemistry or require pseudouridine calling — the CHEUI README directs those cases to SWARM (https://github.com/comprna/SWARM), which supersedes CHEUI for RNA002 accuracy and supports RNA004
- The goal is site-level stoichiometry or differential methylation between conditions — those are handled downstream by CHEUI Model 2 or CHEUI-diff, not by this preprocessing skill

## Inputs

- Nanopolish eventalign output (nanopolish_out.txt) produced with --scale-events --signal-index --samples --print-read-names, containing per-5-mer signal lists and read IDs
- Nanopolish 5-mer k-mer model file of expected signal means (CHEUI/kmer_models/model_kmer.csv)
- Minimap2-aligned, samtools-filtered transcriptome BAM (primary, positive-strand alignments) and indexed transcriptome FASTA used to run eventalign

## Outputs

- cheui_model1_input_signals.tsv: per read/9-mer record with read ID, sequence context, observed length-100 signal vector (5 × 20 segment medians), and length-100 distance vector (element-wise |observed − expected|)
- Per-read 9-mer feature matrix ready for CHEUI-solo Model 1 (CHEUI_m6A_model1.h5 or CHEUI_m5C_model1.h5)

## How to apply

Start from the Nanopolish eventalign output generated with --scale-events --signal-index --samples --print-read-names, and flip each 5-mer's signal list from 3'–5' to 5'–3' ordering before concatenation. For each position of interest (adenosine for m6A, cytosine for m5C), gather the five consecutive overlapping 5-mers covering the centered 9-mer (NNNN(A|C)NNNN) and convert each 5-mer's signal list into 20 order-preserving segment medians, padding with the overall median if fewer than 20 values exist; concatenate the five vectors into a length-100 observed vector. Build the expected unmodified vector from the Nanopolish 5-mer k-mer model (model_kmer.csv), repeating each expected 5-mer mean 20 times, then compute the element-wise absolute distance between observed and expected vectors. Write each read/9-mer's read ID, sequence context, observed vector, and distance vector to cheui_model1_input_signals.tsv. Use CHEUI_preprocess_m6A.py or CHEUI_preprocess_m5C.py — the README warns that m6A and m5C require different preprocessing scripts.

## Related tools

- **CHEUI** (Provides the preprocessing scripts (CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py), the 5-mer k-mer model (model_kmer.csv), and the Model 1 .h5 networks that consume the 100-value signal + distance vectors) — https://github.com/comprna/CHEUI
- **Nanopolish eventalign** (Resquiggles and aligns read signals to the matched transcript references, emitting per-5-mer sample lists with --scale-events --signal-index --samples --print-read-names) — https://nanopolish.readthedocs.io/en/latest/
- **Minimap2** (Aligns Guppy-basecalled reads to the reference transcriptome, producing the sorted BAM (primary, positive-strand alignments) that eventalign consumes)
- **Guppy** (Basecalls FAST5 raw signals to FASTQ (template_rna_r9.4.1_70bps models, 4.0.11+) as the first step before signal-to-transcript alignment)

## Examples

```
python ../scripts/CHEUI_preprocess_m6A.py -i nanopolish_out.txt -m CHEUI/kmer_models/model_kmer.csv -o out_dir -s sample1 -n 8
```

## Evaluation signals

- Every output record contains exactly two vectors of length 100 (5 overlapping 5-mers × 20 segment medians each), matching the CHEUI-solo Model 1 input specification
- The expected vector equals each Nanopolish 5-mer model mean repeated 20 times over its 5-mer's span, and the distance vector equals the element-wise absolute difference between observed and expected vectors
- Per-5-mer signal lists are in 5'–3' order after flipping (they arrive 3'–5' from eventalign), and 5-mer blocks of 20 values align to the corresponding five consecutive overlapping 5-mers of the centered 9-mer
- 9-mers missing sufficient samples show the overall-median padding (not zeros) in under-filled segments
- m6A records are centered on adenosine and m5C records on cytosine, consistent with using the correct CHEUI_preprocess script per modification

## Limitations

- Nanopore signal variability can make modified-vs-unmodified read differences comparable to differences among non-modified reads, limiting single-read separability
- Signal interference at distances <5 nt is exacerbated in densely modified molecules such as ribosomal RNA
- The CHEUI README notes it targets RNA002 (r9.4.1) direct RNA data; SWARM improves accuracy for RNA002 and adds RNA004 and pseudouridine support

## Evidence

- [abstract] CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [methods] eventalign aligns signals: "eventalign was then used to align the read signals to the matched transcript references"
- [methods] flipping per 5-mer: "we thus flipped the signals per 5 mer before concatenating the signals from overlapping 5 mers."
- [readme] preprocessing script description: [evidence span withheld — anchor status: external_doc]
- [readme] eventalign flags: [evidence span withheld — anchor status: external_doc]
