---
name: kmer-signal-hash-table-indexing
description: Use when ONT multi-fast5 reads have been Tombo-resquiggled and you must
  aggregate raw signal events per reference k-mer (default odd k=15) to find modified
  motifs against a WGA control.
when_to_use_negative:
- Input fast5 files are still single-read and/or not resquiggled with Tombo — Snapper's
  pipeline expects resquiggled multi-fast5 event data
- No WGA control sample is available, since the KS-test requires comparing native
  vs control signal distributions
- You want probabilistic per-read modified-base calls rather than de novo methylation
  motif detection — Tombo or Remora are the appropriate tools
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: Snapper
  role: 'Main tool: builds the per-k-mer signal hash table and runs the KS-test to
    emit the candidate seq_set'
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw nanopore signal to the reference so signal events can be indexed
    by genome position
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper
    consumes
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: pip distribution of Snapper used for installation in a python 3.7 conda environment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:477cbabf591ce5b17845ff6abe6ec26f57ed5c489154424c11118c5cf448d93f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - snapper-ont
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# k-mer signal hash-table indexing for nanopore methylation motif detection

## Summary

Indexes Tombo-resquiggled, ont_fast5_api-converted multi-fast5 nanopore signals by genomic k-mer so that per-k-mer native vs WGA signal distributions can be compared with a KS-test (Snapper's signal-comparison stage).

## When to use

Use when ONT multi-fast5 reads have been Tombo-resquiggled and you must aggregate raw signal events per reference k-mer (default odd k=15) to find modified motifs against a WGA control.

## When NOT to use

- Input fast5 files are still single-read and/or not resquiggled with Tombo — Snapper's pipeline expects resquiggled multi-fast5 event data
- No WGA control sample is available, since the KS-test requires comparing native vs control signal distributions
- You want probabilistic per-read modified-base calls rather than de novo methylation motif detection — Tombo or Remora are the appropriate tools

## Inputs

- Tombo-resquiggled multi-read fast5 files for the native sample (directory)
- Tombo-resquiggled multi-read fast5 files for the WGA control (directory)
- Reference genome in FASTA format

## Outputs

- seq_set of candidate modified k-mers written to passed_motifs_[strand]_[contig_name].fasta

## How to apply

First resquiggle the fast5 files with Tombo and convert them to multi-fast5 format with ont_fast5_api, since Snapper reads the ont-tombo event data with h5py. Run snapper with -sample_fast5dir and -control_fast5dir pointing at native and WGA multi-fast5 directories plus a fasta reference; read normalized signal events keyed by genome position and strand. Enumerate all odd-sized k-mers (k_size default 15, minimum 11, odd mandatory) of the reference for both strands and aggregate per-k-mer signal values into a hash table. For each k-mer draw balanced equal-size (N=200) native and WGA signal samples to avoid coverage bias, run scipy.stats.ks_2samp per k-mer, and retain k-mers whose -log KS-test p-value exceeds ks_t (default 3) with confidence above min_conf (default 100). The rationale is Snapper's balanced per-k-mer statistic plus the high-sensitive greedy motif-selection algorithm for position-specific motif enrichment.

## Related tools

- **Snapper** (Main tool: builds the per-k-mer signal hash table and runs the KS-test to emit the candidate seq_set) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw nanopore signal to the reference so signal events can be indexed by genome position) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper consumes) — https://github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (pip distribution of Snapper used for installation in a python 3.7 conda environment)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- passed_motifs_[strand]_[contig_name].fasta files are produced per strand/contig and contain only odd-length k-mers of length >= k_size (default 15)
- Retained k-mers have -log KS-test p-value above the ks_t threshold (default 3) and confidence above min_conf (default 100)
- Native and WGA per-k-mer signal samples are equal size (N=200) — coverage bias should not skew the KS-test
- Genome coverage meets the -coverage minimum (default 40) for k-mers to be tested
- Tooling ran in the documented python 3.7 environment (later versions may break due to inner biopython dependencies)

## Limitations

- Python 3.7 required; later versions might be incompatible because of inner biopython dependencies
- Tombo repository is deprecated/unsupported; ONT recommends Remora instead, so resquiggled-input workflows are fragile
- Minimal genome coverage depth (default 40) must be met or k-mers cannot be assessed
- k_size must be odd and no smaller than 11

## Evidence

- [abstract] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [methods] resquiggle and multi-fast5 conversion prerequisites: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [readme] k-mer size constraints from CLI help: "-k_size K_SIZE            k-mer size, must be odd, should not be less than 11 (default 15)"
- [readme] KS-test threshold from CLI help: "-ks_t KS_T            -log ks_test p-value (default 3)."
