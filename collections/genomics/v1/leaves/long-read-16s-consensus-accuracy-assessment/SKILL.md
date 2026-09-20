---
name: long-read-16s-consensus-accuracy-assessment
description: Use when you have full-length 16S rRNA consensus sequences from a nanopore
  (ONT MinION) mock-community amplicon run with known expected OTU references, and
  you need per-OTU and average consensus accuracy numbers to validate a de novo consensus
  workflow.
when_to_use_negative:
- Input is raw, noisy nanopore reads rather than consensus sequences — reference-based
  mapping of such reads is limited by incorrect mapping due to high error rates; cluster
  first (nanoClust) before assessing accuracy
- The mock community composition is unknown (no expected OTU reference sequences)
  — accuracy cannot be computed against references and only diversity estimation applies
- Data comes from short-read platforms (Illumina MiSeq, Ion Torrent PGM) where this
  long-read consensus accuracy workflow does not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads and corrects read orientation to produce
    the fragmented reads that are clustered and consensus-called prior to accuracy
    assessment
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    that yields the full-length 16S rRNA consensus sequences whose accuracy is assessed
  repo: github.com/umerijaz/nanopore
- name: SILVA
  role: Publicly available reference database context for mapping/filtering; expected
    OTU references for mock communities are used for accuracy computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:bc9a667751d9a6ec26cb55609390df8858f036946663d0c19ce51bde6165f709
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SILVA
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# long-read-16s-consensus-accuracy-assessment

## Summary

A benchmarking skill for quantifying consensus sequence accuracy of de novo long-read 16S rRNA amplicon workflows (e.g., NanoAmpli-Seq: chopSeq fragmentation/orientation correction + nanoClust clustering/consensus calling) by aligning called consensus sequences against expected OTU reference sequences on mock communities.

## When to use

Use when you have full-length 16S rRNA consensus sequences from a nanopore (ONT MinION) mock-community amplicon run with known expected OTU references, and you need per-OTU and average consensus accuracy numbers to validate a de novo consensus workflow.

## When NOT to use

- Input is raw, noisy nanopore reads rather than consensus sequences — reference-based mapping of such reads is limited by incorrect mapping due to high error rates; cluster first (nanoClust) before assessing accuracy
- The mock community composition is unknown (no expected OTU reference sequences) — accuracy cannot be computed against references and only diversity estimation applies
- Data comes from short-read platforms (Illumina MiSeq, Ion Torrent PGM) where this long-read consensus accuracy workflow does not apply

## Inputs

- Full-length 16S rRNA consensus sequences from de novo clustering and consensus calling (nanoClust output) of ONT MinION amplicon reads (2D or 1D2 chemistry)
- Expected OTU reference sequences for the mock community (e.g., 10-organism community with 8 theoretical OTUs)
- Optionally, pre-correction INC-Seq consensus reads (e.g., Li et al. Ladder replicate data from github.com/umerijaz/nanopore) as the input to chopSeq/nanoClust

## Outputs

- Per-OTU consensus sequence accuracy values (percent identity of each called consensus against its expected OTU reference)
- Average consensus sequence accuracy across OTUs (target ~99.5% for 2D/1D2)
- Error profile showing residual errors, expected to be predominantly homopolymer deletions

## How to apply

First run the workflow whose accuracy you are measuring: chopSeq to fragment and correct read orientation of INC-Seq consensus reads, then nanoClust for read partitioning-based de novo clustering and within-cluster consensus calling, partitioning the mock community (e.g., a 10-organism community) into its theoretical number of OTUs (e.g., 8). Align each called consensus sequence against the expected OTU reference sequences to compute per-OTU consensus accuracy, including hard cases such as the Fusobacterium OTU. Average the per-OTU accuracies and compare against the INC-Seq baseline of 97–98% median accuracy; accept the assessment when average accuracy reaches approximately 99.5% (as NanoAmpli-Seq achieves for 2D and 1D2 ONT chemistry) with the hardest OTU near 98.75%. Finally, inspect residual mismatches: nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, so classify errors by type to determine whether homopolymer-aware base calling or error correction would be needed.

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads and corrects read orientation to produce the fragmented reads that are clustered and consensus-called prior to accuracy assessment) — github.com/umerijaz/nanopore
- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling that yields the full-length 16S rRNA consensus sequences whose accuracy is assessed) — github.com/umerijaz/nanopore
- **SILVA** (Publicly available reference database context for mapping/filtering; expected OTU references for mock communities are used for accuracy computation)

## Evaluation signals

- Average consensus accuracy across all OTUs reaches approximately 99.5%, matching NanoAmpli-Seq's reported accuracy for 2D and 1D2 ONT sequencing on tested mock communities
- The hardest OTU (Fusobacterium) achieves approximately 98.75% consensus accuracy
- Reproduced accuracy exceeds the INC-Seq baseline of 97–98% median accuracy on the same Li et al. Ladder replicate (R7 chemistry) data
- Residual errors, when inspected, are almost entirely deletions in homopolymer regions rather than substitutions or random insertions
- Number of consensus OTUs recovered equals the theoretical OTU count of the mock community (e.g., 8 for a 10-organism community)

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, capping accuracy below short-read platform levels without homopolymer-aware base calling or error correction
- Accuracy depends on sequencing chemistry: the 99.5% figure was demonstrated for 2D and 1D2 ONT chemistry on mock communities; older R7-chemistry data reproduced via this workflow reaches the same average but with per-OTU variation (e.g., Fusobacterium at ~98.75%)
- Reference-based accuracy assessment requires known expected OTU sequences, so it is only applicable to mock/controlled communities

## Evidence

- [abstract] NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [intro] The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference: "The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference"
- [intro] incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see: ST Calus, UZ Ijaz, and A Pinto.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see: ST Calus, UZ Ijaz, and A Pinto."
