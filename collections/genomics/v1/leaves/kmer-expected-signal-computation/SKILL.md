---
name: kmer-expected-signal-computation
description: 'Use when reconstructing CHEUI preprocessing that needs expected unmodified
  ionic-current vectors for 9-mers: converting Nanopolish 5-mer k-mer model means
  (model_kmer.csv) into length-100 expected signals and distance vectors from eventalign
  output.'
when_to_use_negative:
- Site-level stoichiometry estimation or differential methylation (CHEUI-solo Model
  2 or CHEUI-diff) — this expected-signal construction is only the Model 1 read-level
  input; downstream site models consume Model 1 probabilities instead
- Pseudouridine detection or RNA004-chemistry data — the README states SWARM (not
  CHEUI) handles pseudouridine and RNA004 predictions
- Input is already a CHEUI Model 1 probability table sorted per site — no k-mer signal
  computation is needed at that stage
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: CHEUI
  role: CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py convert eventalign signal
    lists into observed and expected length-100 vectors plus distance vectors for
    Model 1
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish
  role: eventalign with --scale-events --signal-index --samples --print-read-names
    resquiggles signals to transcript references and supplies the per-5-mer sample
    lists; its 5-mer k-mer model provides the expected signal means
  repo: https://nanopolish.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:e32f87fdb7d31c11238877d7709ceae12126e3b8ffc14edd5b36a3c473a1e26e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# kmer-expected-signal-computation

## Summary

Computes the expected unmodified Nanopore ionic-current vector for each 9-mer context by repeating each 5-mer's k-mer model mean 20 times to length 100, then derives the element-wise absolute distance between observed and expected signals — the paired vector input required by CHEUI-solo Model 1 for m6A and m5C read-level prediction.

## When to use

Use when reconstructing CHEUI preprocessing that needs expected unmodified ionic-current vectors for 9-mers: converting Nanopolish 5-mer k-mer model means (model_kmer.csv) into length-100 expected signals and distance vectors from eventalign output.

## When NOT to use

- Site-level stoichiometry estimation or differential methylation (CHEUI-solo Model 2 or CHEUI-diff) — this expected-signal construction is only the Model 1 read-level input; downstream site models consume Model 1 probabilities instead
- Pseudouridine detection or RNA004-chemistry data — the README states SWARM (not CHEUI) handles pseudouridine and RNA004 predictions
- Input is already a CHEUI Model 1 probability table sorted per site — no k-mer signal computation is needed at that stage

## Inputs

- Nanopolish eventalign output (e.g. nanopolish_out.txt) run with --scale-events --signal-index --samples --print-read-names, listing per-5-mer samples along transcript references with read IDs
- 5-mer k-mer model file with expected signal means (CHEUI/kmer_models/model_kmer.csv)
- Guppy-basecalled (>= 4.0.11, template_rna_r9.4.1_70bps model) and minimap2-aligned direct RNA reads selected with samtools -F 2324

## Outputs

- Per-read 9-mer records in cheui_model1_input_signals.tsv containing read ID, sequence context, a length-100 observed signal vector (5 x 20 segment medians), and a length-100 distance vector of element-wise absolute observed-minus-expected differences
- Distance vector feeding CHEUI-solo Model 1 (CHEUI_m6A_model1.h5 / CHEUI_m5C_model1.h5) read-level modification predictions

## How to apply

Start from Nanopolish eventalign output generated with --scale-events --signal-index --samples --print-read-names, which provides per-5-mer signal lists along the transcript reference with read IDs, and flip each 5-mer's signal list from 3'–5' to 5'–3' ordering before concatenation. For each position of interest (adenosine for m6A, cytosine for m5C), gather the five consecutive overlapping 5-mers covering the centered 9-mer (NNNN(A|C)NNNN). Convert each 5-mer's signals into a 20-value vector by dividing into 20 order-preserving segments and taking segment medians (padding with the overall median if fewer than 20 values), concatenating five such vectors into the length-100 observed vector. Build the expected unmodified vector from the Nanopolish 5-mer k-mer model (CHEUI/kmer_models/model_kmer.csv), repeating each expected 5-mer mean 20 times to length 100. Compute the element-wise absolute distance between observed and expected length-100 vectors, then write each read/9-mer record (read ID, sequence context, observed vector, distance vector) via CHEUI_preprocess_m6A.py or CHEUI_preprocess_m5C.py, matching the 5 × 20 = 100 CHEUI-solo Model 1 input specification.

## Related tools

- **CHEUI** (CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py convert eventalign signal lists into observed and expected length-100 vectors plus distance vectors for Model 1) — https://github.com/comprna/CHEUI
- **Nanopolish** (eventalign with --scale-events --signal-index --samples --print-read-names resquiggles signals to transcript references and supplies the per-5-mer sample lists; its 5-mer k-mer model provides the expected signal means) — https://nanopolish.readthedocs.io/en/latest/

## Examples

```
python ../scripts/CHEUI_preprocess_m6A.py -i nanopolish_out.txt -m ../kmer_models/model_kmer.csv -o out_dir
```

## Evaluation signals

- Every output record contains two length-100 vectors (5 x 20), matching the CHEUI-solo Model 1 input specification of 100 signal values plus a same-length distance vector
- Expected vector values occur in runs of 20 identical values, confirming each 5-mer k-mer model mean was repeated 20 times in transcript order
- Distance vector equals the element-wise absolute difference between the observed and expected length-100 vectors for each read/9-mer
- Each 9-mer's five 5-mer expected values match the k-mer model lookups for the overlapping 5-mers of the centered NNNN(A|C)NNNN context
- Nanopolish was invoked with --scale-events --signal-index --samples --print-read-names; absence of --samples/--signal-index yields no per-5-mer signal lists and empty vectors

## Limitations

- Fewer than 20 samples for a 5-mer requires padding with the overall median, degrading signal information for low-coverage reads
- Signal lists from eventalign arrive in 3'–5' order and must be flipped per 5-mer before concatenating overlapping 5-mers; skipping the flip corrupts the 9-mer vector
- Nanopore signal variability can make modified vs non-modified differences comparable to variation among non-modified reads, and signal interference at distances <5 nt is exacerbated in densely modified molecules such as rRNA

## Evidence

- [abstract] CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [methods] Signal flipping per 5 mer to 5′–3′ orientation: "we thus flipped the signals per 5 mer before concatenating the signals from overlapping 5 mers."
- [readme] CHEUI preprocessing input and k-mer model file: [evidence span withheld — anchor status: external_doc]
- [readme] K-mer model location in CHEUI repository: [evidence span withheld — anchor status: external_doc]
