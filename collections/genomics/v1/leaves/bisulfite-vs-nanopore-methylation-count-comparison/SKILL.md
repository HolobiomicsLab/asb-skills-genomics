---
name: bisulfite-vs-nanopore-methylation-count-comparison
description: Use when comparing the number of methylated cytosine sites detected from
  Nanopore reads (DeepSignal-plant calls) against bisulfite-sequencing site counts,
  per CG/CHG/CHH context, to quantify detection coverage gains.
when_to_use_negative:
- Input data is bisulfite-only (no Nanopore FAST5 signals) — there is nothing to re-squiggle
  or call modifications from
- Evaluating per-read or per-site methylation accuracy against a bisulfite gold standard
  (correlations, precision) rather than counting detected-site totals — that is a
  different comparison task
- Applying to non-plant or non-R9.4 data without a suitable model — the pre-trained
  checkpoint is specific to A. thaliana/O. sativa R9.4 1D reads
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: DeepSignal-plant
  role: Calls per-read 5mC probabilities in all three contexts (call_mods) and per-site
    methylation frequencies (call_freq) that are counted for the comparison
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Annotates raw FAST5 with FASTQ basecalls and re-squiggles signals against
    the genome reference, producing the corrected groups required by call_mods
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls raw Nanopore reads (version 3.6.1, dna_r9.4.1_450bps_hac_prom.cfg)
    to provide the FASTQ used for tombo annotation
- name: split_freq_file_by_5mC_motif.py
  role: Splits the methylation frequency file into CG/CHG/CHH context files so site
    counts can be computed per context
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:7d927a197650c56b91eb1ae4f753ab588c6a1a492e3e885ba3e6defa70b1d670
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - Tombo
  - split_freq_file_by_5mC_motif.py
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# bisulfite-vs-nanopore-methylation-count-comparison

## Summary

Quantifies how many 5mC sites DeepSignal-plant detects from Nanopore reads relative to bisulfite sequencing, by counting per-context methylation calls and computing excess percentages. It tests whether Nanopore-based methylation calling (here, DeepSignal-plant at ~100× coverage) profiles more 5mC sites than bisulfite in A. thaliana and O. sativa.

## When to use

Use when comparing the number of methylated cytosine sites detected from Nanopore reads (DeepSignal-plant calls) against bisulfite-sequencing site counts, per CG/CHG/CHH context, to quantify detection coverage gains.

## When NOT to use

- Input data is bisulfite-only (no Nanopore FAST5 signals) — there is nothing to re-squiggle or call modifications from
- Evaluating per-read or per-site methylation accuracy against a bisulfite gold standard (correlations, precision) rather than counting detected-site totals — that is a different comparison task
- Applying to non-plant or non-R9.4 data without a suitable model — the pre-trained checkpoint is specific to A. thaliana/O. sativa R9.4 1D reads

## Inputs

- Raw single-read Nanopore FAST5 files (R9.4.1) at ~100× coverage for A. thaliana and O. sativa
- FASTQ basecalls from Guppy (e.g., dna_r9.4.1_450bps_hac_prom.cfg) for tombo raw-read annotation
- Genome reference FASTA: GCF_000001735.4_TAIR10.1_genomic.fna (A. thaliana) or IRGSP-1.0 (GCA_001433935.1) for O. sativa
- Bisulfite-sequencing detected-site counts (Supplementary Table 1) as the comparison baseline

## Outputs

- Per-read 5mC probability file from deepsignal_plant call_mods
- Per-site methylation frequency file from call_freq (optionally bedMethyl with --bed --sort)
- Per-context (CG/CHG/CHH) frequency files from split_freq_file_by_5mC_motif.py
- Summary CSV table of detected 5mC site counts per context and species, with excess-over-bisulfite percentages

## How to apply

Basecall raw single-read FAST5 files with Guppy (dna_r9.4.1_450bps_hac_prom.cfg), annotate raw reads with the FASTQ basecalls via tombo preprocess annotate_raw_with_fastqs, and re-squiggle signals against the species reference (TAIR10 for A. thaliana, IRGSP-1.0 for O. sativa) using tombo resquiggle with --corrected-group RawGenomeCorrected_000 --basecall-group Basecall_1D_000. Run deepsignal_plant call_mods with the pre-trained checkpoint model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt and --motifs C to get per-read 5mC probabilities in all three contexts at once, then aggregate to per-site frequencies with deepsignal_plant call_freq (optionally --bed --sort for bedMethyl output). Split the frequency file into CG/CHG/CHH context files with split_freq_file_by_5mC_motif.py, count detected 5mC sites per context and per species, compare against bisulfite-sequencing counts (Supplementary Table 1), and record the excess percentages in a summary CSV table. The rationale is that long Nanopore reads avoid bisulfite's short-read limitation in repetitive regions, so a positive excess is expected and its magnitude is the reporting metric.

## Related tools

- **DeepSignal-plant** (Calls per-read 5mC probabilities in all three contexts (call_mods) and per-site methylation frequencies (call_freq) that are counted for the comparison) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Annotates raw FAST5 with FASTQ basecalls and re-squiggles signals against the genome reference, producing the corrected groups required by call_mods) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls raw Nanopore reads (version 3.6.1, dna_r9.4.1_450bps_hac_prom.cfg) to provide the FASTQ used for tombo annotation)
- **split_freq_file_by_5mC_motif.py** (Splits the methylation frequency file into CG/CHG/CHH context files so site counts can be computed per context)

## Examples

```
deepsignal_plant call_freq --input_path call_mods.tsv --motifs C --bed --sort
```

## Evaluation signals

- Computed excess in detected 5mC sites over bisulfite reproduces 1.1% for A. thaliana and 5.3% for O. sativa at ~100× coverage, matching Supplementary Table 1 and Supplementary Figs. 19-20
- call_freq runs to completion with a valid call_mods input and, with --bed --sort, yields a sorted bedMethyl file
- Splitting with split_freq_file_by_5mC_motif.py produces three non-empty context files covering CG, CHG, and CHH cytosines
- tombo resquiggle succeeds with --corrected-group RawGenomeCorrected_000 --basecall-group Basecall_1D_000 (all FAST5s not failing, e.g., due to missing VBZ HDF5 plugin)

## Limitations

- All fast5s can fail in tombo resquiggle and/or deepsignal_plant call_mods if the ONT VBZ compression HDF5 plugin is missing; setting HDF5_PLUGIN_PATH to the extracted ont-vbz-hdf-plugin is required
- bisulfite sequencing cannot profile 5mCs in repetitive genome regions, so the baseline it provides undercounts sites — the comparison quantifies this excess rather than validating DeepSignal-plant against a complete gold standard
- Count comparison depends on read coverage depth; results are reported at ~100× Nanopore coverage and may not transfer to other depths or flowtypes

## Evidence

- [results] Excess percentages at 100x: [evidence span withheld — anchor status: not_found]
- [methods] call_freq description: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [discussion] Guppy basecalling version: "We use Guppy (version 3.6.1 + 249406c) for the basecalling of all the Nanopore reads"
- [discussion] Tombo resquiggle purpose: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [readme] VBZ plugin known issue: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`."
