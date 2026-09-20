---
name: nanopore-signal-preprocessing
description: Use when raw DRS FAST5 data has been basecalled (Guppy 4.0.11+), aligned
  (minimap2, samtools -F 232457), and eventaligned (Nanopolish --scale-events), and
  per-read 9-mer signal inputs are needed for CHEUI Model 1.
when_to_use_negative:
- Input is raw FAST5 or an unfiltered BAM rather than a Nanopolish eventalign output
  — run eventalign with --scale-events --signal-index --samples --print-read-names
  first
- The goal is site-level stoichiometry or differential testing rather than per-read
  prediction — CHEUI-solo Model 2 and CHEUI-diff consume Model 1/2 outputs, not these
  raw vectors directly
- The data uses the RNA004 chemistry or a modification other than m6A/m5C — the CHEUI
  README directs such cases to SWARM, which adds pseudouridine and RNA004 support
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: CHEUI
  role: Provides CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py scripts and the
    kmer model that convert eventalign output into Model 1 signal inputs
  repo: https://github.com/comprna/R2Dtool
- name: Nanopolish
  role: eventalign with --scale-events --signal-index --samples --print-read-names
    rescales read signals to transcript references and emits the per-5-mer signal
    lists this preprocessing consumes
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: Guppy
  role: Basecalls FAST5 files (template_rna_r9.4.1_70bps model, 4.0.11 or later) to
    produce the FASTQ required upstream of eventalign
- name: minimap2
  role: Aligns basecalled reads to the reference transcriptome (e.g. Gencode v38)
    before samtools -F 232457 best-match filtering and eventalign
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:fda5c6dcd0a65ffbbc3c2bea5a5233cef0fbcd0b2b251f1176445916e9cd4acb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/R2Dtool
  tools:
  - CHEUI
  - Guppy
  - Nanopolish
  - minimap2
  - nanopore direct RNA sequencing
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# nanopore-signal-preprocessing

## Summary

Converts Nanopolish eventalign per-5-mer signal lists from nanopore direct RNA sequencing (DRS) reads into the two length-100 vectors (observed signals and observed-vs-expected distance) that feed CHEUI-solo Model 1 for m6A/m5C per-read prediction.

## When to use

Use when raw DRS FAST5 data has been basecalled (Guppy 4.0.11+), aligned (minimap2, samtools -F 232457), and eventaligned (Nanopolish --scale-events), and per-read 9-mer signal inputs are needed for CHEUI Model 1.

## When NOT to use

- Input is raw FAST5 or an unfiltered BAM rather than a Nanopolish eventalign output — run eventalign with --scale-events --signal-index --samples --print-read-names first
- The goal is site-level stoichiometry or differential testing rather than per-read prediction — CHEUI-solo Model 2 and CHEUI-diff consume Model 1/2 outputs, not these raw vectors directly
- The data uses the RNA004 chemistry or a modification other than m6A/m5C — the CHEUI README directs such cases to SWARM, which adds pseudouridine and RNA004 support

## Inputs

- Nanopolish eventalign output (eventalign run with --scale-events --signal-index --samples --print-read-names)
- Nanopolish 5-mer k-mer model file (e.g. CHEUI/kmer_models/model_kmer.csv)
- Target modification type (m6A uses CHEUI_preprocess_m6A.py; m5C uses CHEUI_preprocess_m5C.py)

## Outputs

- cheui_model1_input_signals.tsv containing, per read/9-mer, the read ID, sequence context, observed 100-value signal vector, and distance 100-value vector

## How to apply

Read the Nanopolish eventalign output (5-mers along the transcript reference with per-5-mer signal lists) and flip each 5-mer's signal list from 3'–5' to 5'–3' ordering. For each target base (A for m6A, C for m5C), gather the signals of the five consecutive overlapping 5-mers covering the centered 9-mer (NNNN(A|C)NNNN). For each 5-mer, split its signal list into 20 order-preserving segments and take each segment median (padding with the overall median if fewer than 20 values exist), then concatenate the five 20-value vectors into a length-100 observed vector. Build the expected unmodified vector from the Nanopolish 5-mer k-mer model (CHEUI/kmer_models/model_kmer.csv) by repeating each expected 5-mer value 20 times, and compute the element-wise absolute distance between observed and expected vectors. Write per read/9-mer the read ID, sequence context, observed vector, and distance vector; every record must carry two length-100 (5 × 20) vectors per the CHEUI Model 1 input specification.

## Related tools

- **CHEUI** (Provides CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py scripts and the kmer model that convert eventalign output into Model 1 signal inputs) — https://github.com/comprna/R2Dtool
- **Nanopolish** (eventalign with --scale-events --signal-index --samples --print-read-names rescales read signals to transcript references and emits the per-5-mer signal lists this preprocessing consumes) — https://nanopolish.readthedocs.io/en/latest/
- **Guppy** (Basecalls FAST5 files (template_rna_r9.4.1_70bps model, 4.0.11 or later) to produce the FASTQ required upstream of eventalign)
- **minimap2** (Aligns basecalled reads to the reference transcriptome (e.g. Gencode v38) before samtools -F 232457 best-match filtering and eventalign)

## Examples

```
python CHEUI_preprocess_m5C.py -i nanopolish_out.txt -m ../CHEUI/kmer_models/model_kmer.csv -o out_dir -s m5C_signals -n 8
```

## Evaluation signals

- Every output record contains two vectors of exactly length 100 (5 × 20), matching the CHEUI-solo Model 1 input specification
- Each 5-mer contributes 20 order-preserving segment medians, padded with the overall median when fewer than 20 signal samples exist
- Signal lists are flipped from 3'–5' to 5'–3' orientation per 5-mer before concatenation
- Distance vector equals the element-wise absolute difference between the observed length-100 vector and the expected vector built by repeating each Nanopolish 5-mer model value 20 times
- The sequence contexts are 9-mers (NNNN(A|C)NNNN) centered on adenosines for m6A and cytosines for m5C

## Limitations

- Downstream prediction is limited by nanopore signal variability, where differences between modified and non-modified reads are often comparable to differences among non-modified reads themselves
- CHEUI requires very strict site-level cutoffs on transcriptome-wide data to keep the expected FDR low, which reduces sensitivity; many sites validated by orthogonal methods fall just below the thresholds
- Basecalling must use Guppy 4.0.11+ (template_rna_r9.4.1_70bps model); the CHEUI models apply to the RNA002 chemistry, with SWARM recommended for RNA004 and pseudouridine

## Evidence

- [methods] Signal flipping per 5 mer to 5′–3′ orientation: "we thus flipped the signals per 5 mer before concatenating the signals from overlapping 5 mers."
- [methods] Signal-to-transcript alignment with Nanopolish eventalign: "eventalign was then used to align the read signals to the matched transcript references"
- [methods] Read filtering with samtools -F 232457: "Reads were then filtered to select the best match for each read using samtools -F 232457."
- [readme] CHEUI preprocessing step description: [evidence span withheld — anchor status: external_doc]
- [readme] Distinct preprocessing per modification: [evidence span withheld — anchor status: external_doc]
