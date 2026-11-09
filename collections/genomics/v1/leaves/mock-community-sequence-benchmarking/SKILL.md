---
name: mock-community-sequence-benchmarking
description: Use when raw nanopore 16S amplicon reads from a mock community of known
  composition are available and you need to quantify sequence accuracy, aligned proportion,
  or OTU-count behavior of a de novo consensus workflow.
when_to_use_negative:
- Input is an already-collapsed OTU/ASV feature table rather than raw nanopore reads
  — no INC-Seq consensus or partitioning can be applied.
- The true community composition is unknown, so theoretical OTU counts and reference-based
  accuracy cannot be computed.
- The data are short Illumina MiSeq or Ion Torrent PGM amplicon reads rather than
  long ONT MinION reads; the workflow is designed for full-length 16S nanopore data.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads, corrects read orientation, and optionally
    applies 1,300–1,450 bp size selection before accuracy benchmarking
  repo: https://github.com/umerijaz/nanopore
- name: nanoClust
  role: Performs read partitioning-based de novo clustering and within-cluster consensus
    calling to produce full-length 16S rRNA gene sequences for OTU-count and accuracy
    sweeps
  repo: https://github.com/umerijaz/nanopore
- name: ONT MinION
  role: Sequencing platform producing the raw 2D and 1D2 mock-community nanopore reads
    used as benchmarking input
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:25198b89f23c0283023c4a15585313698b191d86a01881224e1f86edbf15bd22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - ONT MinION
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mock-community-sequence-benchmarking

## Summary

Benchmark a nanopore amplicon workflow (e.g., NanoAmpli-Seq with chopSeq and nanoClust) by processing raw MinION reads from defined mock communities and measuring consensus sequence accuracy, aligned proportion, and OTU counts against theoretical expectations.

## When to use

Use when raw nanopore 16S amplicon reads from a mock community of known composition are available and you need to quantify sequence accuracy, aligned proportion, or OTU-count behavior of a de novo consensus workflow.

## When NOT to use

- Input is an already-collapsed OTU/ASV feature table rather than raw nanopore reads — no INC-Seq consensus or partitioning can be applied.
- The true community composition is unknown, so theoretical OTU counts and reference-based accuracy cannot be computed.
- The data are short Illumina MiSeq or Ion Torrent PGM amplicon reads rather than long ONT MinION reads; the workflow is designed for full-length 16S nanopore data.

## Inputs

- raw nanopore 16S amplicon reads from defined mock communities (1-organism and 10-organism mocks, 2D and 1D2 sequencing modes)
- mock community reference 16S rRNA gene sequences for blastn accuracy/alignment computation

## Outputs

- blastn aligned-proportion and consensus accuracy table (e.g., metrics.csv)
- OTU counts and accuracy per partition count 1–7 (e.g., partition_sweep_results.csv)
- chopSeq-processed INC-Seq consensus reads and nanoClust consensus full-length 16S rRNA gene sequences

## How to apply

Download the deposited mock-community raw nanopore reads (1-organism and 10-organism mocks, 2D and 1D2 modes) from github.com/umerijaz/nanopore. Run INC-Seq to build consensus reads, then chopSeq for fragmentation and read orientation correction, optionally with 1,300–1,450 bp size selection (chopSeq_SS), and quantify contiguous alignment to reference 16S sequences with blastn — expecting roughly 73±14.6% (INC-Seq), 96±2.3% (chopSeq), and 98.4±0.7% (chopSeq_SS) aligned proportion. Next, run the nanoClust de novo read partitioning-based clustering with partition counts swept from 1 to 7, calling within-cluster consensus full-length 16S rRNA gene sequences at each setting. Count OTUs per partition setting and align each consensus to mock reference 16S sequences to compute accuracy; expect accuracy to fall and OTU counts to inflate above the theoretical number as partition count rises, with the low-partition optimum reaching ~99.5% average accuracy for 2D and 1D2 sequencing. Tabulate metrics (e.g., metrics.csv, partition_sweep_results.csv) and validate against the reported error ranges.

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads, corrects read orientation, and optionally applies 1,300–1,450 bp size selection before accuracy benchmarking) — https://github.com/umerijaz/nanopore
- **nanoClust** (Performs read partitioning-based de novo clustering and within-cluster consensus calling to produce full-length 16S rRNA gene sequences for OTU-count and accuracy sweeps) — https://github.com/umerijaz/nanopore
- **ONT MinION** (Sequencing platform producing the raw 2D and 1D2 mock-community nanopore reads used as benchmarking input)

## Examples

```
python partition_sweep.py --reads mock_1org_2D.fastq --partitions 1 2 3 4 5 6 7 --out partition_sweep_results.csv
```

## Evaluation signals

- Aligned proportion after INC-Seq falls within ~73±14.6%; after chopSeq within ~96±2.3%; after chopSeq_SS within ~98.4±0.7%.
- Consensus sequence accuracy after chopSeq_SS is ~98±0.23%, and nanoClust consensus accuracy at the optimal low partition setting reaches ~99.5% for 2D and 1D2 sequencing.
- OTU counts inflate above the theoretical number of organisms as partition count increases from 1 toward 7, while accuracy simultaneously decreases.
- metrics.csv and partition_sweep_results.csv reproduce the reported trend when compared against the paper's Table 2 and Fig. 7 values.

## Limitations

- Nearly all residual errors in NanoAmpli-Seq consensus sequences originate from deletions in homopolymer regions, so accuracy gains likely require homopolymer-aware base calling or error correction.
- Theoretical OTU thresholds are only known for defined mock communities; accuracy and OTU-inflation checks are not applicable to environmental samples.
- Reference-based benchmarking via blastn presupposes the community members are present in the reference set.

## Evidence

- [abstract] NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads, while nanoClust algorithm was designed for read partitioning-based de novo clustering: "chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see: ST Calus, UZ Ijaz, and A Pinto. NanoAmpli-Seq: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
- [intro] single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform, has opened the possibility of obtaining ultra-long reads: "single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform, has opened the possibility of obtaining ultra-long reads"
