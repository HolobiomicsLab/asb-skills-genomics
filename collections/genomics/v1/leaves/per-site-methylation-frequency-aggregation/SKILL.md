---
name: per-site-methylation-frequency-aggregation
description: Use when per-read 5mC calls from DeepSignal-plant call_mods need collapsing
  into per-cytosine methylation frequencies (optionally bedMethyl) for CG/CHG/CHH
  comparison against bisulfite sequencing.
when_to_use_negative:
- Per-read modification states are the deliverable — use the call_mods per-read output
  directly instead of aggregating.
- Input is already a per-site methylation table (e.g., bisulfite-derived bedMethyl)
  — there are no Nanopore per-read calls left to aggregate.
- Raw FAST5s have not yet been basecalled and re-squiggled (Guppy + tombo resquiggle
  into RawGenomeCorrected_000) — call_freq has no call_mods file to consume.
edam_operation: http://edamontology.org/operation_0366
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3168
tools:
- name: DeepSignal-plant
  role: Its call_freq function aggregates the call_mods per-read 5mC file into per-site
    methylation frequencies (optionally bedMethyl via --bed --sort)
  repo: https://github.com/PengNi/deepsignal-plant
- name: split_freq_file_by_5mC_motif.py
  role: Splits the aggregated modification-frequency file into CG/CHG/CHH context
    files for per-context evaluation
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:01ae10e204eb65c4ddabd51d6166b09688d2d361bff176e2924323dbfc0272f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - PyTorch
  - Tombo
  - split_freq_file_by_5mC_motif.py
  edam_operation: http://edamontology.org/operation_0366
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-site-methylation-frequency-aggregation

## Summary

Collapse DeepSignal-plant's per-read 5mC probability calls (the call_mods output) into per-cytosine methylation frequencies with call_freq, optionally emitting sorted bedMethyl, then split by CG/CHG/CHH context so Nanopore methylation levels can be compared directly against bisulfite sequencing.

## When to use

Use when per-read 5mC calls from DeepSignal-plant call_mods need collapsing into per-cytosine methylation frequencies (optionally bedMethyl) for CG/CHG/CHH comparison against bisulfite sequencing.

## When NOT to use

- Per-read modification states are the deliverable — use the call_mods per-read output directly instead of aggregating.
- Input is already a per-site methylation table (e.g., bisulfite-derived bedMethyl) — there are no Nanopore per-read calls left to aggregate.
- Raw FAST5s have not yet been basecalled and re-squiggled (Guppy + tombo resquiggle into RawGenomeCorrected_000) — call_freq has no call_mods file to consume.

## Inputs

- call_mods output file (TSV of per-read 5mC probabilities from deepsignal_plant call_mods, e.g. from R9.4 FAST5s re-squiggled by tombo into RawGenomeCorrected_000)
- Reference genome context (e.g. TAIR10 GCF_000001735.4 / IRGSP-1.0) implicitly defining cytosine positions and CG/CHG/CHH motifs

## Outputs

- Modification-frequency file of per-site methylation frequencies (call_freq output)
- Sorted bedMethyl file (when called with --bed --sort)
- Per-context frequency files (CG, CHG, CHH) produced by split_freq_file_by_5mC_motif.py

## How to apply

Run `deepsignal_plant call_freq` on the call_mods output (per-read 5mC probabilities from the BiLSTM model, called with --motifs C so all three contexts are scored at once); each cytosine's frequency is computed as the number of reads called methylated divided by the total number of reads mapped to that site. Pass --bed --sort when a sorted bedMethyl track is the desired deliverable. Split the resulting modification-frequency file by sequence context using split_freq_file_by_5mC_motif.py into CG/CHG/CHH files so each context is evaluated independently. Benchmark by computing per-site Pearson correlation of Nanopore frequencies against each bisulfite replicate, restricted to cytosines with ≥5 mapped Nanopore reads (and ≥10 mapped bisulfite reads for comparisons); the denoised A. thaliana model reaches r=0.9122 (CHG) and r=0.7840 (CHH). Key threshold logic mirrors label construction: high-confidence methylated sites need ≥5 reads and ≥0.9 methylation frequency, unmethylated sites need ≥5 reads and zero frequency, so sites below the 5-read coverage floor are excluded from validation.

## Related tools

- **DeepSignal-plant** (Its call_freq function aggregates the call_mods per-read 5mC file into per-site methylation frequencies (optionally bedMethyl via --bed --sort)) — https://github.com/PengNi/deepsignal-plant
- **split_freq_file_by_5mC_motif.py** (Splits the aggregated modification-frequency file into CG/CHG/CHH context files for per-context evaluation) — https://github.com/PengNi/deepsignal-plant

## Examples

```
deepsignal_plant call_freq --input_path call_mods.tsv --bed --sort
```

## Evaluation signals

- Spot-check: for sampled cytosines, the call_freq frequency equals (number of reads called methylated) / (total number of reads mapped to the site) recomputed from the call_mods file.
- Per-site Pearson correlation against each of the three bisulfite replicates reproduces the reference values (e.g., 0.9122 CHG and 0.7840 CHH for the denoised A. thaliana model, Fig. 1d).
- Coverage invariant: every evaluated site has ≥5 mapped Nanopore reads (and ≥10 mapped bisulfite reads for benchmarking); no site below this floor enters the comparison.
- Context split sanity: the CG/CHG/CHH files produced by split_freq_file_by_5mC_motif.py partition cytosines by motif with no overlaps or missing contexts.
- With --bed --sort, the output is a chromosome-sorted bedMethyl whose scores equal the per-site methylation frequencies.

## Limitations

- Frequencies are noisy at low coverage: the workflow excludes cytosines with fewer than 5 mapped reads, and CHG/CHH gold standards came from the union of three bisulfite replicates (potential label noise).
- Agreement metrics are bounded by bisulfite quality, since bisulfite sequencing is the gold standard and cannot profile repetitive regions or avoid incomplete conversion.
- The aggregation is only as reliable as upstream call_mods: VBZ-compressed FAST5s require the ont-vbz-hdf-plugin (HDF5_PLUGIN_PATH set) or resquiggle/call_mods fail, leaving call_freq without input.

## Evidence

- [discussion] by counting the number of reads where the site is called methylated and the total number of reads mapped to the site, DeepSignal-plant calculates a methylation frequency: "by counting the number of reads where the site is called methylated and the total number of reads mapped to the site, DeepSignal-plant calculates a methylation frequency"
- [methods] A modification-frequency file can be generated by `call_freq` function with the call_mods file as input: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [discussion] cytosines covered with at least five reads and had at least 0.9 methylation frequency as high-confidence methylated sites: "cytosines covered with at least five reads and had at least 0.9 methylation frequency as high-confidence methylated sites"
- [readme] required to basecall and re-squiggle the raw signals from nanopore reads before running deepsignal-plant: "required to basecall and re-squiggle the raw signals from nanopore reads before running deepsignal-plant"
