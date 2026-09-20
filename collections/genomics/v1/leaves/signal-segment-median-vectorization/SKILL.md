---
name: signal-segment-median-vectorization
description: 'Use when converting Nanopolish eventalign output into CHEUI-solo Model
  1 inputs for nanopore direct RNA m6A/m5C prediction: length-100 observed and distance
  signal vectors for 9-mers centered on candidate A (m6A) or C (m5C) sites.'
when_to_use_negative:
- For RNA004 chemistry reads or pseudouridine detection — the CHEUI README directs
  users to SWARM (https://github.com/comprna/SWARM), which supersedes CHEUI for those
  cases
- Input is only raw FAST5 or basecalled FASTQ without Nanopolish eventalign --samples
  output — per-5-mer signal lists cannot be built
- Goal is site-level stoichiometry or differential methylation from already-preprocessed
  read-level probabilities (CHEUI Model 2 / CHEUI-diff stage) — no signal vectorization
  is needed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3168
tools:
- name: CHEUI
  role: Provides CHEUI_preprocess_m6A.py and CHEUI_preprocess_m5C.py, which segment
    each 5-mer's signal list into 20 medians, concatenate five overlapping 5-mers
    per 9-mer, and emit the length-100 signal and distance vectors consumed by CHEUI
    Model 1
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish
  role: eventalign resquiggles read signals to transcript references (--scale-events
    --signal-index --samples --print-read-names), producing the per-5-mer signal lists
    that are vectorized
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: Guppy
  role: Basecalls FAST5 to FASTQ (template_rna_r9.4.1_70bps model, 4.0.11 or later)
    upstream of eventalign resquiggling
  repo: https://community.nanoporetech.com/downloads/guppy/
- name: minimap2
  role: Aligns basecalled reads to the reference transcriptome so eventalign can match
    signals to transcript 5-mers
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:9f7a9a630a3b22d140af451ac1a5e1cdaae64acd7dfeb2ee655ecef8c1bb566a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Guppy
  - Nanopolish
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# signal-segment-median-vectorization

## Summary

CHEUI signal preprocessing that converts Nanopolish eventalign per-5-mer signal lists into 20-segment median vectors, concatenates five consecutive overlapping 5-mers covering a 9-mer into a length-100 observed-signal vector, and pairs it with an equal-length distance vector against Nanopolish k-mer model expectations.

## When to use

Use when converting Nanopolish eventalign output into CHEUI-solo Model 1 inputs for nanopore direct RNA m6A/m5C prediction: length-100 observed and distance signal vectors for 9-mers centered on candidate A (m6A) or C (m5C) sites.

## When NOT to use

- For RNA004 chemistry reads or pseudouridine detection — the CHEUI README directs users to SWARM (https://github.com/comprna/SWARM), which supersedes CHEUI for those cases
- Input is only raw FAST5 or basecalled FASTQ without Nanopolish eventalign --samples output — per-5-mer signal lists cannot be built
- Goal is site-level stoichiometry or differential methylation from already-preprocessed read-level probabilities (CHEUI Model 2 / CHEUI-diff stage) — no signal vectorization is needed

## Inputs

- Nanopolish eventalign output (nanopolish_out.txt) produced with --scale-events --signal-index --samples --print-read-names
- Nanopolish 5-mer k-mer model file of expected signal means (CHEUI/kmer_models/model_kmer.csv)
- Guppy (>=4.0.11)-basecalled FASTQ/FAST5 and minimap2-aligned, samtools-filtered sorted BAM feeding eventalign

## Outputs

- cheui_model1_input_signals.tsv with, per read/9-mer: read ID, sequence context, observed length-100 signal vector, and length-100 distance vector (5 x 20 = 100 values each)

## How to apply

Start from Nanopolish eventalign output generated with --scale-events --signal-index --samples --print-read-names on Guppy (>=4.0.11)-basecalled reads aligned with minimap2 and filtered to primary positive-strand alignments; flip each 5-mer's signal list from 3'–5' to 5'–3' orientation before concatenation. For each position of interest (adenosine for m6A, cytosine for m5C), gather the signals of the five consecutive overlapping 5-mers covering the centered 9-mer (NNNN(A|C)NNNN). Convert each 5-mer's signal list into a 20-value vector by dividing it into 20 order-preserving segments and taking each segment's median, padding with the overall median if fewer than 20 values exist, then concatenate the five vectors into a length-100 observed-signal vector. Build the expected unmodified vector from the Nanopolish 5-mer k-mer model (CHEUI/kmer_models/model_kmer.csv), repeating each expected 5-mer mean 20 times to length 100, and compute the element-wise absolute distance between observed and expected vectors. Write per read/9-mer the read ID, sequence context, observed 100-value vector, and distance 100-value vector using the modification-specific script CHEUI_preprocess_m6A.py or CHEUI_preprocess_m5C.py.

## Related tools

- **CHEUI** (Provides CHEUI_preprocess_m6A.py and CHEUI_preprocess_m5C.py, which segment each 5-mer's signal list into 20 medians, concatenate five overlapping 5-mers per 9-mer, and emit the length-100 signal and distance vectors consumed by CHEUI Model 1) — https://github.com/comprna/CHEUI
- **Nanopolish** (eventalign resquiggles read signals to transcript references (--scale-events --signal-index --samples --print-read-names), producing the per-5-mer signal lists that are vectorized) — https://nanopolish.readthedocs.io/en/latest/
- **Guppy** (Basecalls FAST5 to FASTQ (template_rna_r9.4.1_70bps model, 4.0.11 or later) upstream of eventalign resquiggling) — https://community.nanoporetech.com/downloads/guppy/
- **minimap2** (Aligns basecalled reads to the reference transcriptome so eventalign can match signals to transcript 5-mers)

## Examples

```
python ../scripts/CHEUI_preprocess_m6A.py -i nanopolish_out.txt -m CHEUI/kmer_models/model_kmer.csv -o cheui_preprocessed/
```

## Evaluation signals

- Every output record contains exactly two vectors of length 100 (5 x 20), matching the CHEUI-solo Model 1 input specification of 100 signal values plus a same-length distance vector
- Each 5-mer's signal list is ordered 5'–3' (flipped from eventalign's 3'–5' order) before segment medians are computed
- 5-mer vectors with fewer than 20 samples are padded with the overall median; no emitted 5-mer vector is shorter than 20 values
- The distance vector equals the element-wise absolute difference between the observed length-100 vector and the expected vector built by repeating each Nanopolish k-mer model 5-mer mean 20 times
- Each record carries a read ID and a 9-mer sequence context centered on the target adenosine (m6A) or cytosine (m5C)

## Limitations

- Differences between modified and non-modified read signals are often comparable to read-to-read (population) variability, capping single-molecule prediction sensitivity
- Signal interference at distances <5 nt would be exacerbated in densely modified molecules such as ribosomal RNA, hurting individual read prediction
- Very strict cut-offs are used on cell transcriptome sites to maintain low expected FDR, reducing CHEUI sensitivity; many orthogonally supported sites fall below thresholds
- m6A and m5C require different preprocessing scripts (CHEUI_preprocess_m6A.py vs CHEUI_preprocess_m5C.py) and different trained model files (--DL_model)

## Evidence

- [methods] Signal flipping per 5 mer to 5′–3′ orientation: "we thus flipped the signals per 5 mer before concatenating the signals from overlapping 5 mers."
- [readme] CHEUI preprocessing step: [evidence span withheld — anchor status: external_doc]
- [readme] Nanopolish eventalign flags required for CHEUI preprocessing: [evidence span withheld — anchor status: external_doc]
- [abstract] CHEUI processes observed and expected signals: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [discussion] Signal variability limitation: "This limitation may stem from the variability of the nanopore signals, where the differences between modified and non-modified reads are often comparable"
- [discussion] Densely modified rRNA caveat: "This effect on the individual read prediction would be exacerbated in ribosomal RNA molecules since they are highly and densely modified."
