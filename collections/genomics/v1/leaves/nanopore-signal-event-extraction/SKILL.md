---
name: nanopore-signal-event-extraction
description: Use when raw ONT fast5 reads must be resquiggled, converted to multi-fast5,
  and parsed into position-anchored signal events to support k-mer-level comparisons
  of native vs WGA samples.
when_to_use_negative:
- Input fast5 files have not been resquiggled with Tombo — raw unanchored signal cannot
  be keyed by genome position
- Genome coverage of the native sample or WGA control is below the minimal coverage
  depth (default 40), making per-k-mer signal samples unreliable
- The goal is single-read modification calling per site rather than motif-level comparison
  of native vs WGA signal distributions
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Tombo
  role: Resquiggles raw nanopore signal to the reference so events are position-anchored;
    provides the ont-tombo event data read via Python
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts Tombo-resquiggled single-read fast5 files into the multi-fast5 format
    that Snapper reads
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: h5py
  role: Python library used to read the HDF5/multi-fast5 files and extract stored
    event data
- name: snapper-ont
  role: Pip-distributed Snapper implementation that performs the signal-event extraction
    and per-k-mer aggregation from sample and control multi-fast5 directories
  repo: https://github.com/DNKonanov/Snapper
- name: scipy
  role: Provides scipy.stats.ks_2samp for the downstream two-sample KS-test on the
    extracted per-k-mer signal samples
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:1f1862771e02482aa530b34d61fb5df6b0bcf4e71de869a2cbfcd7be3a35d545
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - Python
  - Snapper
  - Tombo
  - h5py
  - ont_fast5_api
  - scipy
  - snapper-ont
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nanopore-signal-event-extraction

## Summary

Extract normalized per-read signal events from Tombo-resquiggled multi-fast5 files using h5py/ont-tombo event data, and key each read's signal by genome position and strand so per-k-mer signal distributions can be compared between native and WGA control samples.

## When to use

Use when raw ONT fast5 reads must be resquiggled, converted to multi-fast5, and parsed into position-anchored signal events to support k-mer-level comparisons of native vs WGA samples.

## When NOT to use

- Input fast5 files have not been resquiggled with Tombo — raw unanchored signal cannot be keyed by genome position
- Genome coverage of the native sample or WGA control is below the minimal coverage depth (default 40), making per-k-mer signal samples unreliable
- The goal is single-read modification calling per site rather than motif-level comparison of native vs WGA signal distributions

## Inputs

- Raw ONT fast5 read files (sample and WGA control)
- Tombo-resquiggled fast5 files
- Multi-fast5 directories (converted via ont_fast5_api)
- Reference genome in fasta format

## Outputs

- Per-read normalized signal events keyed by genome position and strand
- Hash table of per-k-mer aggregated signal values for both strands
- Balanced (N=200) native and WGA signal samples per k-mer for KS-testing

## How to apply

First resquiggle the fast5 files with Tombo so raw signal is aligned to the reference, then convert them to the multi-fast5 format with ont_fast5_api. Read the resquiggled multi-fast5 files from the sample and control directories using h5py/ont-tombo event data, keying each read's normalized signal events by genome position and strand. Enumerate all odd-sized k-mers (k_size default 15, minimum 11) of the reference genome in fasta format for both strands and aggregate per-k-mer signal values into a hash table. For each k-mer, draw equal-size balanced signal samples (N=200) from the native sample and the WGA control to avoid coverage bias before downstream KS-test comparison. Coverage should meet the minimal genome coverage depth (default 40) for the signal aggregation to be reliable.

## Related tools

- **Tombo** (Resquiggles raw nanopore signal to the reference so events are position-anchored; provides the ont-tombo event data read via Python) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts Tombo-resquiggled single-read fast5 files into the multi-fast5 format that Snapper reads) — https://github.com/nanoporetech/ont_fast5_api
- **h5py** (Python library used to read the HDF5/multi-fast5 files and extract stored event data)
- **snapper-ont** (Pip-distributed Snapper implementation that performs the signal-event extraction and per-k-mer aggregation from sample and control multi-fast5 directories) — https://github.com/DNKonanov/Snapper
- **scipy** (Provides scipy.stats.ks_2samp for the downstream two-sample KS-test on the extracted per-k-mer signal samples)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- Resquiggling completed without errors for both sample and control fast5 directories before multi-fast5 conversion
- All odd-sized k-mers (k_size default 15, minimum 11) of the reference genome are enumerated for both strands in the hash table
- Per-k-mer native and WGA signal samples are equal size (N=200), confirming balanced sampling with no coverage bias
- Genome coverage depth meets or exceeds the minimal coverage threshold (default 40)
- Retained candidates have -log KS-test p-value above the ks_t threshold (default 3) with confidence above min_conf (default 100)

## Limitations

- Python 3.7 is required; later versions might be incompatible because of inner biopython dependencies
- k-mer size must be odd and should not be less than 11 (default 15)
- Tombo repository is now unsupported and deprecated in favor of Remora, which may affect long-term reproducibility

## Evidence

- [methods] Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software.: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] After resquiggling, fast5 files should be converted to the multi-fast5 format using ont_fast5_api.: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [readme] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [readme] k_size K_SIZE            k-mer size, must be odd, should not be less than 11 (default 15): "  -k_size K_SIZE        k-mer size, must be odd, 
                        should not be less than 11 (default 15)"
- [readme] Tombo is a suite of tools primarily for the identification of modified nucleotides from nanopore sequencing data.: [evidence span withheld — anchor status: external_doc]
