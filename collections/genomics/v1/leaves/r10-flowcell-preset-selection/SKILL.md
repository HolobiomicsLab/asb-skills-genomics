---
name: r10-flowcell-preset-selection
description: Use when input raw signals come from an R10.4/R10.4.1 Oxford Nanopore
  flow cell (FAST5/POD5/SLOW5) and must be mapped with RawHash2, or when comparing
  R10.4 versus R9.4 mapping accuracy.
when_to_use_negative:
- Input signals are legacy r9.4.1 data — use the standard R9.4 Sigmap segmentation
  parameters/preset instead; UNCALLED itself only supports legacy r9.4.1 data
- The goal is reference-free all-vs-all overlap detection between raw signals — use
  the Rawsamble mechanism rather than the reference-guided R10.4 mapping preset
- A base-level read mapper on basecalled FASTQ/POD5 basecalls is sufficient — minimap2
  on sequences, not RawHash2 on raw signals, is the right tool
edam_operation: http://edamontology.org/operation_0292
tools:
- name: RawHash2
  role: Raw signal mapper run with the R10.4 preset (--r10) and R10.4.1 UNCALLED4
    k-mer model for indexing; provides the R10.4 flow cell support and mapping output
  repo: https://github.com/CMU-SAFARI/RawHash
- name: minimap2
  role: Produces ground-truth basecall/sequence alignments of the same reads used
    to compute RawHash2 mapping accuracy/F1
  repo: https://github.com/lh3/minimap2
- name: Sigmap
  role: Source of the R9.4 segmentation parameters that RawHash2 uses by default,
    against which R10.4-specific parameterization is contrasted
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:4341432b212f227a81d1f48cfde1b560925aa364be9d9bce19f28ac9898df3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# r10-flowcell-preset-selection

## Summary

Select the correct RawHash2 k-mer model and mapping preset when mapping raw nanopore signals from the newer R10.4 flow cell, using the UNCALLED4 R10.4.1 k-mer model for indexing and the --r10 flag plus a genome-size preset for mapping, because R10.4 signals require different segmentation/scoring parameters than legacy R9.4 data.

## When to use

Use when input raw signals come from an R10.4/R10.4.1 Oxford Nanopore flow cell (FAST5/POD5/SLOW5) and must be mapped with RawHash2, or when comparing R10.4 versus R9.4 mapping accuracy.

## When NOT to use

- Input signals are legacy r9.4.1 data — use the standard R9.4 Sigmap segmentation parameters/preset instead; UNCALLED itself only supports legacy r9.4.1 data
- The goal is reference-free all-vs-all overlap detection between raw signals — use the Rawsamble mechanism rather than the reference-guided R10.4 mapping preset
- A base-level read mapper on basecalled FASTQ/POD5 basecalls is sufficient — minimap2 on sequences, not RawHash2 on raw signals, is the right tool

## Inputs

- Raw nanopore signal files from an R10.4.1 flow cell (FAST5 files, or POD5/SLOW5 as supported by RawHash2)
- Reference genome in FASTA format
- UNCALLED4 R10.4.1 local k-mer model (means-only text file under extern/local_kmer_models/)

## Outputs

- RawHash2 index file (ref.ind) built with the R10.4.1 k-mer model
- Mapping output in UNCALLED-style modified PAF format (mapping.paf)
- Mapping accuracy/F1 statistics computed against minimap2 ground-truth PAF alignments

## How to apply

First build the RawHash2 index from the reference genome FASTA using the R10.4.1 UNCALLED4 local k-mer model (e.g. extern/local_kmer_models/uncalled_r1041_model_only_means.txt) with rawhash2 -d ref.ind -p <model> -t 32 ref.fasta, since index generation uses expected signal representations from a pore model. Then map the R10.4.1 raw signals with the R10.4 preset together with a genome-size preset (e.g. rawhash2 -t 32 -x sensitive --r10 ref.ind fast5_files > mapping.paf); the --r10 flag activates R10.4-optimized segmentation and scoring, whereas RawHash2 otherwise uses the R9.4 segmentation parameters from Sigmap. Output mappings are reported in the UNCALLED-style modified PAF format, which you compare against minimap2 ground-truth alignments of the same reads to compute mapping accuracy/F1. Validation: the R10.4 accuracy should reproduce the Section 3.5 result — reasonably good (useful for contamination analysis) but lower than the R9.4 accuracy reported in Supplementary Table S4, reflecting the flow-cell-specific parameter sensitivity.

## Related tools

- **RawHash2** (Raw signal mapper run with the R10.4 preset (--r10) and R10.4.1 UNCALLED4 k-mer model for indexing; provides the R10.4 flow cell support and mapping output) — https://github.com/CMU-SAFARI/RawHash
- **minimap2** (Produces ground-truth basecall/sequence alignments of the same reads used to compute RawHash2 mapping accuracy/F1) — https://github.com/lh3/minimap2
- **Sigmap** (Source of the R9.4 segmentation parameters that RawHash2 uses by default, against which R10.4-specific parameterization is contrasted) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -d ref.ind -p extern/local_kmer_models/uncalled_r1041_model_only_means.txt -t 32 ref.fasta; rawhash2 -t 32 -x sensitive --r10 ref.ind test/data/d6_ecoli_r104/fast5_files > mapping.paf
```

## Evaluation signals

- Mapping output PAF entries follow the UNCALLED-style modified PAF format used for accuracy/F1 comparison
- Computed R10.4 mapping accuracy/F1 reproduces the Section 3.5 result — reasonably good but lower than R9.4 accuracy in Supplementary Table S4
- Index build succeeded only when the R10.4.1 UNCALLED4 k-mer model (-p extern/local_kmer_models/uncalled_r1041_model_only_means.txt) was supplied with the -d ref.ind index output
- The --r10 flag was present in the mapping command; omitting it silently applies R9.4 Sigmap segmentation parameters and degrades R10.4 accuracy
- F1 comparison against minimap2 ground-truth alignments of the same reads shows R10.4 accuracy remains sufficient for contamination analysis

## Limitations

- RawHash2 accuracy on R10.4 raw signals is lower than its accuracy on R9.4 flow cells despite optimized segmentation and scoring parameters
- Accuracy is described as only reasonably accurate — useful for contamination analysis rather than high-confidence base-level mapping
- UNCALLED itself cannot be applied to r10.4.1 data (legacy r9.4.1 only); only RawHash2 with the --r10 preset handles the newer flow cell
- RawHash2's minimizer seeding mode is slightly less accurate than non-minimizer mode (though ~3X faster), which can affect accuracy-sensitive R10.4 runs

## Evidence

- [abstract] R10.4 flow cell support: "support for the R10.4 flow cell version and POD5 and SLOW5 file formats"
- [methods] indexing with k-mer model: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] R9.4 segmentation defaults: "RawHash2 uses the R9.4 segmentation parameters from [Sigmap](https://github.com/haowenz/sigmap)."
- [readme] UNCALLED legacy-only caveat: [evidence span withheld — anchor status: external_doc]
- [methods] minimizer accuracy tradeoff: "which is slightly less accurate than the non-minimizer mode but much faster (around 3X)."
