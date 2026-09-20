---
name: cytosine-motif-targeting
description: Use when raw Nanopore FAST5/POD5 reads have been basecalled and re-squiggled
  to a plant genome reference and you need per-cytosine k-mer and raw-signal features
  for 5mC detection in CpG/CHG/CHH contexts.
when_to_use_negative:
- The input is already the call_mods output or a per-site methylation-frequency file
  produced by `call_freq` — this skill operates upstream, on re-squiggled FAST5s
- The reads have not been basecalled and re-squiggled (no RawGenomeCorrected_000 group)
  — run Guppy basecalling and `tombo resquiggle` first
- The modification of interest is not a cytosine methylation in a CG/CHG/CHH-style
  motif (e.g. non-cytosine modifications) — the targeting and 13-mer context logic
  is cytosine-motif specific
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: DeepSignal-plant
  role: Implements the `extract` subcommand that scans the motif, centers the k=13
    13-mer on each target cytosine, and writes sequence/signal features to the TSV
  repo: github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw signals to contiguous reference bases (version 1.5.1), producing
    the per-base signal events the extractor reads from RawGenomeCorrected_000
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts multi-read FAST5 files to single-read format (`multi_to_single_fast5`)
    required before resquiggling and extraction
  repo: github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:85439245a88737a654ac1f3bc973f12ef149eac13b883343cb28d86c0e8b1835
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSignal-plant
  - Tombo
  - ont_fast5_api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cytosine-motif-targeting

## Summary

Locate target cytosines in a specified sequence motif (e.g. C, CpG, CHG, CHH) within re-squiggled Nanopore reads and extract the centered 13-mer (k=13) sequence features plus 13x16 signal features for each site, forming the per-site feature table that DeepSignal-plant consumes.

## When to use

Use when raw Nanopore FAST5/POD5 reads have been basecalled and re-squiggled to a plant genome reference and you need per-cytosine k-mer and raw-signal features for 5mC detection in CpG/CHG/CHH contexts.

## When NOT to use

- The input is already the call_mods output or a per-site methylation-frequency file produced by `call_freq` — this skill operates upstream, on re-squiggled FAST5s
- The reads have not been basecalled and re-squiggled (no RawGenomeCorrected_000 group) — run Guppy basecalling and `tombo resquiggle` first
- The modification of interest is not a cytosine methylation in a CG/CHG/CHH-style motif (e.g. non-cytosine modifications) — the targeting and 13-mer context logic is cytosine-motif specific

## Inputs

- Single-read FAST5 directory re-squiggled by Tombo with corrected group RawGenomeCorrected_000
- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna)
- Motif string(s) for target cytosines (C, CpG, CHG, CHH)
- Guppy basecalled reads (Guppy >=3.6.1) as prerequisite

## Outputs

- Tab-delimited feature TSV (e.g. fast5s.C.features.tsv) with columns chrom, pos, strand, readname, read_strand, k_mer (13-mer), raw_signals (13x16 per-base signal blocks joined by ';'), and methy_label (0/1)
- Feature table serving as input to `deepsignal_plant call_mods`

## How to apply

Run `deepsignal_plant extract` on the re-squiggled single-read FAST5 directory (group RawGenomeCorrected_000 produced by Tombo resquiggle) with --motifs C (or CpG/CHG/CHH), --nproc 30, writing to fast5s.C.features.tsv. For each targeted cytosine in each read, extract the 13-mer sequence context centered on the target base and, from the Tombo-aligned signal events, compute per-base signal means, standard deviations, and signal lengths after median-shift/MAD-scale normalization; collect 13x16 raw signal values with per-base blocks joined by ';' in the raw_signals column. Record read metadata (chrom, pos, strand, readname, read_strand) plus the 0/1 methy_label training column. Rationale: the BRNN model needs both sequence and signal representations of the site's local context, and supporting all three plant methylcytosine contexts (CpG, CHG, CHH) is the paper's central goal.

## Related tools

- **DeepSignal-plant** (Implements the `extract` subcommand that scans the motif, centers the k=13 13-mer on each target cytosine, and writes sequence/signal features to the TSV) — github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw signals to contiguous reference bases (version 1.5.1), producing the per-base signal events the extractor reads from RawGenomeCorrected_000) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts multi-read FAST5 files to single-read format (`multi_to_single_fast5`) required before resquiggling and extraction) — github.com/nanoporetech/ont_fast5_api

## Examples

```
deepsignal_plant extract --input_path fast5s/ --corrected_group RawGenomeCorrected_000 --motifs C --nproc 30 --out_fn fast5s.C.features.tsv
```

## Evaluation signals

- Output TSV is tab-delimited and contains all declared columns (chrom, pos, strand, readname, read_strand, k_mer, raw_signals, methy_label)
- Every k_mer field is exactly a 13-mer and its 7th base is the cytosine at the recorded pos
- Each raw_signals field contains a signal block for every base of the k-mer, with per-base blocks joined by ';'
- Per-base features cover base identity, signal mean, standard deviation, and number of signal values (13x16 signal features total)
- Targeted positions match cytosines in the reference genome for the requested motif (C/CpG/CHG/CHH)

## Limitations

- Requires prior Guppy basecalling and Tombo 1.5.1 resquiggling; Tombo is now deprecated/unsupported by Oxford Nanopore
- VBZ-compressed FAST5s can fail resquiggle/extraction unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set
- The R9 pipeline pins h5py<3 and numpy<1.23 (required by Tombo); newer FAST5/POD5 + Dorado R10 workflows use a separate environment
- Detailed hyperparameters and some training/evaluation details are deferred to supplementary materials not included in the main text

## Evidence

- [full_text] DeepSignal-plant builds sequence features from the centered k-mer (k = 13 by default) consisting of four per-base features: [evidence span withheld — anchor status: not_found]
- [discussion] raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [methods] deepsignal_plant call_mods --input_path fast5s.C.features.tsv: "deepsignal_plant call_mods --input_path fast5s.C.features.tsv"
- [readme] Guppy and tombo are required to basecall and re-squiggle the raw signals from nanopore reads before running deepsignal-plant.: "Guppy and tombo are required to basecall and re-squiggle the raw signals from nanopore reads before running deepsignal-plant."
