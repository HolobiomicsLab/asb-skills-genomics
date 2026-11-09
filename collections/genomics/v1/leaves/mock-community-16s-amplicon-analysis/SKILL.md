---
name: mock-community-16s-amplicon-analysis
description: Use when raw Oxford Nanopore MinION full-length 16S rRNA amplicon reads
  from mixed microbial communities need de novo consensus OTU calling without reference-based
  mapping, and benchmark accuracy against mock-community references is required.
when_to_use_negative:
- Input reads are short-read amplicon data (e.g., Illumina MiSeq, Ion Torrent PGM)
  where long-read consensus generation is unnecessary
- The analysis goal is reference-based mapping of raw noisy nanopore reads — high
  raw-read error rates make such mapping unreliable
- Only taxonomy classification is needed; classification-only analysis cannot differentiate
  closely related sequences and risks false detections, so full consensus calling
  adds unnecessary cost
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads and corrects read orientation before clustering
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Partitions chopped reads into 3 clusters, performs de novo clustering and
    within-cluster consensus calling to obtain full-length 16S rRNA gene sequences
  repo: github.com/umerijaz/nanopore
- name: INC-Seq
  role: Rolling-circle alignment builds circular consensus sequences from raw reads,
    retaining only reads supporting at least 3 concatemers
- name: MAFFT-G-INS-i
  role: Aligns 50 reads per OTU for within-cluster consensus calling
- name: ONT MinION
  role: Sequencing platform producing the raw 2D and 1D2 nanopore amplicon reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:573e2d165de340c0e861e6cd5c71c740cd587fa3f4f9b14e4dd1e3931bfa88e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INC-Seq
  - MAFFT-G-INS-i
  - ONT MinION
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mock-community-16s-amplicon-analysis

## Summary

Reproduces the NanoAmpli-Seq de novo amplicon workflow on MinION 16S rRNA gene mock-community reads, combining INC-Seq circular consensus (≥3 concatemers), chopSeq fragmentation/orientation correction, and nanoClust partitioned clustering to call full-length OTU consensus sequences.

## When to use

Use when raw Oxford Nanopore MinION full-length 16S rRNA amplicon reads from mixed microbial communities need de novo consensus OTU calling without reference-based mapping, and benchmark accuracy against mock-community references is required.

## When NOT to use

- Input reads are short-read amplicon data (e.g., Illumina MiSeq, Ion Torrent PGM) where long-read consensus generation is unnecessary
- The analysis goal is reference-based mapping of raw noisy nanopore reads — high raw-read error rates make such mapping unreliable
- Only taxonomy classification is needed; classification-only analysis cannot differentiate closely related sequences and risks false detections, so full consensus calling adds unnecessary cost

## Inputs

- Raw MinION nanopore full-length 16S rRNA gene amplicon reads (2D and 1D2) for single-organism and 10-organism mock communities (github.com/umerijaz/nanopore deposit)
- Reference 16S rRNA gene sequences of the mock community organisms for accuracy benchmarking

## Outputs

- nanoampliseq_consensus.fasta — full-length 16S rRNA gene per-OTU consensus sequences
- Per-OTU consensus composition of both mock communities
- Per-OTU consensus accuracy values against reference 16S sequences (expected range 99.2%–100%, average ~99.5%)

## How to apply

Download the raw MinION nanopore amplicon reads for the single-organism and 10-organism mock communities from github.com/umerijaz/nanopore. Run INC-Seq with rolling-circle alignment to build circular consensus sequences, retaining only reads supporting at least 3 concatemers, then fragment and orientation-correct those consensus reads with chopSeq. Run nanoClust on the chopped reads, partitioning reads into 3 clusters with de novo clustering followed by within-cluster consensus calling (MAFFT-G-INS-i alignment of 50 reads per OTU) to obtain full-length 16S rRNA gene OTU sequences. Write the per-OTU consensus sequences to nanoampliseq_consensus.fasta, compare each against the mock-community reference 16S sequences, and verify that average accuracy is ~99.5% with per-OTU accuracy of 99.2%–100% and residual errors dominated by deletions in homopolymer regions. Rationale: the completely de novo approach avoids incorrect mapping of noisy raw nanopore reads to reference databases and outperforms classification-only analysis for closely related sequences.

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads and corrects read orientation before clustering) — github.com/umerijaz/nanopore
- **nanoClust** (Partitions chopped reads into 3 clusters, performs de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene sequences) — github.com/umerijaz/nanopore
- **INC-Seq** (Rolling-circle alignment builds circular consensus sequences from raw reads, retaining only reads supporting at least 3 concatemers)
- **MAFFT-G-INS-i** (Aligns 50 reads per OTU for within-cluster consensus calling)
- **ONT MinION** (Sequencing platform producing the raw 2D and 1D2 nanopore amplicon reads)

## Examples

```
INC-Seq -i mock_reads.fastq --min_len 1400 --min_concordance 0.95 -t 3 | chopSeq | nanoClust --clusters 3 > nanoampliseq_consensus.fasta
```

## Evaluation signals

- Average consensus sequence accuracy across OTUs reaches ~99.5% (expected for 2D and 1D2 sequencing on tested mock communities)
- Per-OTU accuracy falls within the 99.2%–100% range
- Residual errors are dominated by deletions in homopolymer regions, not substitutions or insertions
- INC-Seq retains only reads supporting at least 3 concatemers
- Each called consensus aligns over its entire length to the reference 16S rRNA sequence after MAFFT-G-INS-i alignment of 50 reads per OTU

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions
- Accuracy comparable to short-read platforms would require homopolymer-aware base calling or error correction
- INC-Seq protocol with the 3-concatemer threshold and reduced processing trades read retention for processing time and accuracy

## Evidence

- [abstract] NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [intro] The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference: "The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore."
