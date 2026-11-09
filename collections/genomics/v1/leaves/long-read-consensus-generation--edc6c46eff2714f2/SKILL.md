---
name: long-read-consensus-generation
description: Use when raw nanopore amplicon reads (2D or 1D2) from mock or mixed microbial
  communities are too error-prone (~15% raw error) and a de novo, reference-free full-length
  16S consensus with ~99.5% accuracy is the analysis goal.
when_to_use_negative:
- Input reads are short-read amplicon data (e.g., Illumina MiSeq, Ion Torrent PGM),
  whose short lengths make rolling-circle INC-Seq concatemer building inapplicable
- The goal is reference-based classification of raw long reads only — the article
  notes such approaches are limited in differentiating closely related sequences and
  risk false detections
- Reads are already high-quality consensus sequences or a processed OTU/feature table,
  so consensus generation would be redundant
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: chopSeq
  role: Fragments INC-Seq consensus reads and corrects read orientation; with 1,300–1,450
    bp size selection (chopSeq_SS) raises the contiguously aligned proportion to ~98.4±0.7%
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Partitions chopped reads into 3 clusters, performs de novo clustering and
    within-cluster consensus calling (MAFFT G-INS-i of ~50 reads per OTU) to obtain
    full-length 16S rRNA gene sequences
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:a91dc6c153ca4a5adb0e79032b0afb70da88ab7626760bd7778aae9311ece7fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# long-read-consensus-generation

## Summary

Generates high-accuracy full-length 16S rRNA gene consensus sequences from noisy ONT MinION amplicon reads using the NanoAmpli-Seq chain: INC-Seq rolling-circle consensus (>=3 concatemers), chopSeq fragmentation/orientation correction, and nanoClust partitioned de novo clustering with within-cluster consensus calling.

## When to use

Use when raw nanopore amplicon reads (2D or 1D2) from mock or mixed microbial communities are too error-prone (~15% raw error) and a de novo, reference-free full-length 16S consensus with ~99.5% accuracy is the analysis goal.

## When NOT to use

- Input reads are short-read amplicon data (e.g., Illumina MiSeq, Ion Torrent PGM), whose short lengths make rolling-circle INC-Seq concatemer building inapplicable
- The goal is reference-based classification of raw long reads only — the article notes such approaches are limited in differentiating closely related sequences and risk false detections
- Reads are already high-quality consensus sequences or a processed OTU/feature table, so consensus generation would be redundant

## Inputs

- Raw ONT MinION 2D/1D2 nanopore amplicon FASTQ reads from mixed microbial communities or mock communities
- Reference 16S rRNA gene sequences (e.g., mock-community organism references or SILVA) for blastn-based accuracy evaluation

## Outputs

- INC-Seq circular consensus reads (FASTA/FASTQ)
- chopSeq-corrected, fragmented and orientation-corrected reads (optionally size-selected at 1,300–1,450 bp)
- Full-length 16S rRNA gene OTU consensus sequences (e.g., nanoampliseq_consensus.fasta)
- Alignment-proportion and accuracy metrics table (e.g., metrics.csv with ~73±14.6% INC-Seq, ~96±2.3% chopSeq, ~98.4±0.7% chopSeq_SS, 98±0.23% accuracy)

## How to apply

Start by running INC-Seq on the raw MinION reads, using rolling-circle alignment to build circular consensus sequences and retaining only reads supported by at least 3 concatemers. Next, fragment the INC-Seq consensus reads and correct read orientation with chopSeq; optionally apply a 1,300–1,450 bp size-selection step (chopSeq_SS) to raise the contiguously aligned read proportion from ~96±2.3% to ~98.4±0.7%. Then run nanoClust on the chopped reads, partitioning them into 3 clusters for de novo clustering, and perform within-cluster consensus calling (MAFFT G-INS-i alignment of ~50 reads per OTU followed by consensus generation) to obtain full-length 16S rRNA OTU sequences. Judge success by blastn comparison against reference 16S sequences: the full workflow should yield ~99.5% average consensus accuracy (per-OTU range 99.2%–100%), with residual errors dominated by deletions in homopolymer regions.

## Related tools

- **chopSeq** (Fragments INC-Seq consensus reads and corrects read orientation; with 1,300–1,450 bp size selection (chopSeq_SS) raises the contiguously aligned proportion to ~98.4±0.7%) — github.com/umerijaz/nanopore
- **nanoClust** (Partitions chopped reads into 3 clusters, performs de novo clustering and within-cluster consensus calling (MAFFT G-INS-i of ~50 reads per OTU) to obtain full-length 16S rRNA gene sequences) — github.com/umerijaz/nanopore

## Evaluation signals

- blastn aligned proportion of INC-Seq consensus reads falls within ~73±14.6%
- After chopSeq without size selection, aligned proportion is ~96±2.3%; after chopSeq_SS (1,300–1,450 bp) it is ~98.4±0.7%
- Consensus sequence accuracy vs. reference 16S reaches ~98±0.23% after chopSeq_SS and ~99.5% average (99.2%–100% per OTU) after the full workflow
- The entire consensus read aligns contiguously to the reference 16S sequence
- Residual consensus errors are dominated by deletions in homopolymer regions rather than substitutions

## Limitations

- Residual errors concentrate as deletions in homopolymer regions; homopolymer-aware base calling or error correction would be needed to approach short-read accuracy
- INC-Seq discards reads that do not support at least 3 concatemers, so very short or low-quality raw reads are lost
- Incorrect mapping of noisy long reads to a reference database can discard reads from organisms absent from the reference, motivating the de novo (reference-free) clustering route

## Evidence

- [abstract] The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [abstract] NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5%: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5%"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
