---
name: long-read-amplicon-sequence-analysis
description: Use when raw noisy ONT MinION 2D/1D2 amplicon reads from a mock or mixed
  microbial community must be converted into high-accuracy full-length 16S rRNA gene
  consensus sequences without reference-based mapping, and when residual errors need
  error-mode attribution (e.g., homopolymer deletions).
when_to_use_negative:
- The analysis goal is only taxonomic classification of long reads — the article notes
  classification-only approaches are limited in differentiating closely related sequences
  and risk false detections
- Reads are short amplicon reads from short-read platforms (e.g., Illumina MiSeq,
  Ion Torrent PGM), which the workflow was not designed for
- A reference-based consensus or reference-guided mapping approach is acceptable —
  this workflow is explicitly de novo and does not map raw reads to SILVA or custom
  databases
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Fragmentation and read orientation correction of INC-Seq consensus reads
  repo: https://github.com/umerijaz/nanopore
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    to obtain full-length 16S rRNA gene sequences
  repo: https://github.com/umerijaz/nanopore
- name: blastn
  role: Aligns reads at each processing stage against reference 16S sequences to compute
    alignment length ratios and extract mismatch/insertion/deletion patterns
- name: Biopython and pandas
  role: Parse blastn tabular output and compute per-read alignment length ratios and
    ratio distributions per stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:caec492fa5879cb5fea3df2c07ae1f626119913111861a2a3b9ed82912abd35a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - Biopython and pandas
  - blastn
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# long-read-amplicon-sequence-analysis

## Summary

Method for processing Oxford Nanopore 16S amplicon reads from mixed microbial communities through the NanoAmpli-Seq workflow: INC-Seq consensus generation, chopSeq fragmentation/orientation correction, and nanoClust de novo clustering to obtain accurate full-length 16S rRNA gene sequences (~99.5% consensus accuracy).

## When to use

Use when raw noisy ONT MinION 2D/1D2 amplicon reads from a mock or mixed microbial community must be converted into high-accuracy full-length 16S rRNA gene consensus sequences without reference-based mapping, and when residual errors need error-mode attribution (e.g., homopolymer deletions).

## When NOT to use

- The analysis goal is only taxonomic classification of long reads — the article notes classification-only approaches are limited in differentiating closely related sequences and risk false detections
- Reads are short amplicon reads from short-read platforms (e.g., Illumina MiSeq, Ion Torrent PGM), which the workflow was not designed for
- A reference-based consensus or reference-guided mapping approach is acceptable — this workflow is explicitly de novo and does not map raw reads to SILVA or custom databases

## Inputs

- Raw Oxford Nanopore (ONT MinION) 2D or 1D2 amplicon sequencing reads from mixed microbial communities (FASTQ/FASTA)
- INC-Seq consensus reads (FASTA)
- chopSeq-fragmented reads (FASTA)
- nanoClust OTU-consensus sequences (FASTA)
- Reference 16S rRNA sequences (mock-community references / SILVA-derived references)

## Outputs

- Full-length 16S rRNA gene OTU consensus sequences with ~99.5% average accuracy (FASTA)
- Per-stage alignment length ratio distributions (INC-Seq, chopSeq, OTU consensus) computed from blastn tabular output
- Tabulated deletion counts localizing single base-pair deletions to homopolymer tracts >4 bp

## How to apply

Retrieve the deposited FASTA files (INC-Seq consensus reads, chopSeq-fragmented reads, and nanoClust OTU-consensus sequences) from github.com/umerijaz/nanopore. First generate INC-Seq consensus reads, then apply chopSeq for fragmentation and read orientation correction of those consensus reads, and finally run nanoClust for read partitioning-based de novo clustering with within-cluster consensus calling to obtain full-length 16S rRNA gene sequences. To attribute residual error, run blastn of reads at each stage against reference 16S sequences (mock-community / SILVA-derived references), compute per-read alignment length ratios (alignment length / reference length) with Biopython and pandas, and verify ratios are predominantly <1, indicating deletions dominate over insertions. Then extract mismatch/insertion/deletion patterns from the blastn alignments and count single base-pair deletions within homopolymer tracts longer than 4 bp; correct application yields the reported ~99.5% average consensus accuracy for 2D and 1D2 sequencing.

## Related tools

- **chopSeq** (Fragmentation and read orientation correction of INC-Seq consensus reads) — https://github.com/umerijaz/nanopore
- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene sequences) — https://github.com/umerijaz/nanopore
- **blastn** (Aligns reads at each processing stage against reference 16S sequences to compute alignment length ratios and extract mismatch/insertion/deletion patterns)
- **Biopython and pandas** (Parse blastn tabular output and compute per-read alignment length ratios and ratio distributions per stage)

## Examples

```
python analyze_alignment_ratios.py --input incseq_reads.fasta chopseq_reads.fasta otu_consensus.fasta --ref silva_16S_refs.fasta --tool blastn
```

## Evaluation signals

- Alignment length ratios (alignment length / reference length) are predominantly <1 at each stage, confirming deletions dominate over insertions
- Residual consensus errors are concentrated as single base-pair deletions within homopolymer tracts longer than 4 bp, with no detectable insertions
- Average consensus sequence accuracy of the final OTU consensus reads reaches ~99.5% for 2D and 1D2 sequencing on tested mock communities
- Reproduced OTU consensus sequences are full-length 16S rRNA gene sequences obtained via de novo clustering, not reference-mapped
- Estimated diversity of mock communities matches expected composition

## Limitations

- Nearly all residual errors originate from deletions in homopolymer regions, so accuracy is bounded by the base caller's homopolymer handling
- Reference-based clustering of noisy long reads is limited by incorrect mapping and discarding of reads from organisms absent from the reference database — hence the de novo design, but de novo clustering requires sufficient within-cluster read depth
- Homopolymer-aware base calling or error correction would be needed to reach accuracy comparable to short-read platforms; the current workflow does not achieve this

## Evidence

- [abstract] The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] nanoClust was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] NanoAmpli-Seq achieves 99.5% average consensus accuracy for 2D and 1D2 sequencing: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] Nearly all residual errors originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [intro] incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] Repository README points to scripts for processing short amplicon reads from Oxford Nanopore: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
