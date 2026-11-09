---
name: nanopore-amplicon-read-subsampling
description: Decision support for choosing how many ONT MinION reads (2D or 1D2) to
  allocate per OTU when constructing de novo 16S rRNA consensus sequences, or when
  validating that a consensus depth yields sufficient accuracy.
when_to_use_negative:
- Analysis already has >=50 reads per OTU and only needs a point estimate of consensus
  accuracy rather than a depth-accuracy curve
- Input is short-read amplicon data (e.g., Illumina MiSeq or Ion Torrent PGM) rather
  than ONT nanopore 2D/1D2 reads
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    on each read-depth subsample to produce full-length 16S rRNA gene consensus sequences
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Sequencing platform producing the 2D and 1D2 amplicon read sets being subsampled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:54c7c1d96dcccabb9284b8b1dba5e9cf817dbadcb4587da53d48a4a88af925ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - nanoClust
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nanopore-amplicon-read-subsampling

## Summary

A benchmarking skill that subsamples nanopore amplicon reads at varying per-OTU depths (5-100) and quantifies how read count affects nanoClust consensus sequence accuracy, establishing that >99% accuracy is reached above 5 reads, plateaus at 10-15 reads, and is only reliably high at >=50 reads.

## When to use

Decision support for choosing how many ONT MinION reads (2D or 1D2) to allocate per OTU when constructing de novo 16S rRNA consensus sequences, or when validating that a consensus depth yields sufficient accuracy.

## When NOT to use

- Analysis already has >=50 reads per OTU and only needs a point estimate of consensus accuracy rather than a depth-accuracy curve
- Input is short-read amplicon data (e.g., Illumina MiSeq or Ion Torrent PGM) rather than ONT nanopore 2D/1D2 reads

## Inputs

- 2D nanopore amplicon reads (one-organism experiment) from the umerijaz/nanopore repository
- 1D2 nanopore amplicon reads (one-organism experiment) from the umerijaz/nanopore repository
- SILVA 16S reference sequence for the organism under test

## Outputs

- Full-length 16S rRNA gene consensus FASTA per subsampled depth
- accuracy_curve.png plotting consensus accuracy versus subsampled read count

## How to apply

Load the one-organism 2D and 1D2 nanopore amplicon read sets from the nanopore repository data, then for each depth level n in 5-100 subsample n reads per OTU. Run nanoClust read partitioning-based de novo clustering followed by within-cluster consensus calling on each subsample to obtain full-length 16S rRNA gene consensus sequences. Align each consensus against the SILVA 16S reference sequence of the organism and compute sequence identity as the accuracy metric. Repeat across depth levels and plot accuracy versus subsampled read count, marking the >5-read, ~10-15-read plateau, and >=50-read stability regions, writing the consensus FASTA and accuracy_curve.png as outputs.

## Related tools

- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling on each read-depth subsample to produce full-length 16S rRNA gene consensus sequences) — github.com/umerijaz/nanopore
- **ONT MinION** (Sequencing platform producing the 2D and 1D2 amplicon read sets being subsampled)

## Examples

```
python nanoClust.py --input subsample_n.fastq --depth n  # n in 5..100 reads per OTU
```

## Evaluation signals

- Consensus sequence accuracy exceeds 99% for subsamples with more than 5 reads per OTU
- Accuracy curve plateaus at approximately 10-15 reads per OTU
- Accuracy is reliably high only when at least 50 reads per OTU are available
- Computed sequence identity is measured by alignment against the SILVA 16S reference sequence of the organism
- accuracy_curve.png renders the full 5-100 depth range with the three annotated regions

## Limitations

- Residual consensus errors nearly all originate from deletions in homopolymer regions, capping achievable accuracy without homopolymer-aware base calling or error correction
- Accuracy is only reliably high at >=50 reads per OTU; low-depth consensus may be unstable despite >99% nominal accuracy above 5 reads
- Calibrated on the one-organism experiment; mixed microbial communities may shift the plateau position

## Evidence

- [results] Consensus accuracy threshold: [evidence span withheld — anchor status: not_found]
- [abstract] nanoClust role: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [intro] Platform context: "single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform, has opened the possibility of obtaining ultra-long reads"
- [intro] Reference alignment target: "mapping raw, noisy reads to custom or publicly available reference databases (i.e., SILVA)"
- [abstract] Residual error source: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
