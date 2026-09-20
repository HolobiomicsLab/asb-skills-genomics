---
name: raw-signal-statistics-aggregation
description: Use when extracting per-base signal statistics from re-squiggled Nanopore
  FAST5 reads to build k-mer feature vectors (e.g., k=13 sequence features, m=16 signal
  features) for methylation model input.
when_to_use_negative:
- Input is already a feature table (e.g., fast5s.C.features.tsv) — proceed directly
  to call_mods instead of re-extracting.
- Reads have not been re-squiggled with Tombo, so per-base signal events cannot be
  aligned to genome reference bases.
- Analyzing methylation with statistics-based signal comparison alone (e.g., Tombo)
  rather than building deep-learning features.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: DeepSignal-plant
  role: Provides the `extract` command that computes the per-base signal statistics
    and k-mer raw signal features from re-squiggled FAST5 reads
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw signals of reads to contiguous genome reference bases, providing
    the per-base event alignment required for statistics aggregation
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts multi-read FAST5 to single-read FAST5 via multi_to_single_fast5 before
    re-squiggling and extraction
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:8ac8e0c44821d1012ecbfa1baa6ee8fa28c5a22a9bd10a43fee0501e4496f82f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# raw-signal-statistics-aggregation

## Summary

Aggregates Tombo-aligned Nanopore raw-signal events into per-base statistics (signal mean, standard deviation, and signal length) plus per-base raw signal blocks (13x16 features joined with ';') for each targeted 5mC site, forming the sequence and signal features of a centered 13-mer consumed by DeepSignal-plant.

## When to use

Use when extracting per-base signal statistics from re-squiggled Nanopore FAST5 reads to build k-mer feature vectors (e.g., k=13 sequence features, m=16 signal features) for methylation model input.

## When NOT to use

- Input is already a feature table (e.g., fast5s.C.features.tsv) — proceed directly to call_mods instead of re-extracting.
- Reads have not been re-squiggled with Tombo, so per-base signal events cannot be aligned to genome reference bases.
- Analyzing methylation with statistics-based signal comparison alone (e.g., Tombo) rather than building deep-learning features.

## Inputs

- Tombo re-squiggled single-read FAST5 directory (corrected group RawGenomeCorrected_000)
- Genome reference (e.g., GCF_000001735.4_TAIR10.1_genomic.fna)
- Target motif specification (e.g., C) and k-mer size (k=13), signal block length (m=16)

## Outputs

- fast5s.C.features.tsv: tab-delimited table with chrom, pos, strand, readname, read_strand, 13-mer k_mer, per-base signal statistics, ';'-joined raw_signals blocks (13x16), and 0/1 methy_label

## How to apply

Run `deepsignal_plant extract` on the Tombo re-squiggled single-read FAST5 directory with --corrected_group RawGenomeCorrected_000 and --motifs C, writing to fast5s.C.features.tsv. For each targeted cytosine in each read, locate the 13-mer (k=13 default) sequence context centered on the target base and, from the Tombo-aligned signal events, compute four per-base features: base identity, mean signal, signal standard deviation, and number of signal values per base. Additionally collect the raw signal values per base of the 13-mer as 13x16 signal features, joining per-base blocks with ';' in the raw_signals column. Record read metadata (chrom, pos, strand, readname, read_strand) and the 0/1 methy_label training column; raw signals of each read are normalized by median shift and median absolute deviation (MAD) scale before statistics are computed.

## Related tools

- **DeepSignal-plant** (Provides the `extract` command that computes the per-base signal statistics and k-mer raw signal features from re-squiggled FAST5 reads) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw signals of reads to contiguous genome reference bases, providing the per-base event alignment required for statistics aggregation) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts multi-read FAST5 to single-read FAST5 via multi_to_single_fast5 before re-squiggling and extraction) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
deepsignal_plant extract --input_path fast5s/ --model_path model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt --corrected_group RawGenomeCorrected_000 --motifs C --nproc 30 --write_path fast5s.C.features.tsv
```

## Evaluation signals

- Output TSV is tab-delimited with all declared columns present (chrom, pos, strand, readname, read_strand, k_mer, statistics, raw_signals, methy_label).
- Each k_mer field is exactly a 13-mer of sequence context centered on the target cytosine.
- Each raw_signals field contains one signal block per base of the k-mer (13 blocks of 16 values) joined by ';'.
- Per-base statistics (mean, standard deviation, number of signal values) are computed from the Tombo RawGenomeCorrected_000 event alignment for every base of the 13-mer.
- methy_label values are binary 0/1, consistent with the training gold-standard labels.

## Limitations

- Tombo (version 1.5.1) is unsupported and deprecated upstream (replaced by Remora); resquiggle may fail on VBZ-compressed FAST5 unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set.
- R9.4.1 pipelines require strict h5py<3 and numpy<1.23 pins needed by Tombo; FAST5 must first be converted to single-read format with ont_fast5_api.
- The extraction is specific to the deep-learning feature schema (k=13, m=16) of DeepSignal-plant and does not directly produce methylation calls; methylation calling is a separate call_mods step.

## Evidence

- [full_text] For each targeted 5mC site, DeepSignal-plant builds sequence features from the centered k-mer (k = 13 by default) consisting of four per-base features: the base identity and the mean, standard deviation, and number of signal values of each base.: [evidence span withheld — anchor status: not_found]
- [discussion] Raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale.: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [discussion] Tombo is used to map raw signals of reads to contiguous bases in the genome reference.: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [methods] The deepsignal_plant extract command operates on re-squiggled single-read FAST5 with the RawGenomeCorrected_000 corrected group and motif C, writing fast5s.C.features.tsv.: "be re-squiggled by [tombo (version 1.5.1)](https://github.com/nanoporetech/tombo)"
- [readme] VBZ-compressed FAST5 files fail in tombo resquiggle and deepsignal_plant call_mods unless the ont-vbz-hdf-plugin is available via HDF5_PLUGIN_PATH.: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`"
