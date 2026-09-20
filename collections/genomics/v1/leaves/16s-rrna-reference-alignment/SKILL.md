---
name: 16s-rrna-reference-alignment
description: Use when you have full-length 16S rRNA consensus sequences built de novo
  from nanopore amplicon reads (2D or 1D2) and need a quantitative accuracy metric
  by aligning them to a SILVA reference of the known organism, e.g., to characterize
  a reads-per-consensus accuracy curve.
when_to_use_negative:
- The true organism reference sequence is unknown or the community is uncharacterized
  — the article emphasizes de novo consensus generation precisely to avoid reliance
  on reference databases
- Sequences are raw noisy nanopore reads rather than post-nanoClust consensus sequences;
  raw read error rates make direct reference-identity uninformative
- Data are short Illumina MiSeq or Ion Torrent PGM amplicon reads rather than full-length
  ONT MinION amplicon consensus sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: nanoClust
  role: Produces the read-partitioning de novo clusters and within-cluster consensus
    16S sequences that are then aligned to the SILVA reference for identity scoring
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Nanopore platform supplying the 2D and 1D2 amplicon reads subsampled per OTU
    for consensus construction and accuracy evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:577ee074c0df4e6f76fdbe1a1ddc26d316ae20b193fbdc2eb2ea983d978c0760
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - nanoClust
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# 16s-rrna-reference-alignment

## Summary

Aligns de novo nanoClust-derived full-length 16S rRNA gene consensus sequences against a SILVA 16S reference sequence of the target organism and computes sequence identity as the consensus accuracy metric, used to characterize how accuracy changes with reads-per-OTU depth (e.g., >99% beyond 5 reads, plateau at ~10-15, stable at >=50).

## When to use

Use when you have full-length 16S rRNA consensus sequences built de novo from nanopore amplicon reads (2D or 1D2) and need a quantitative accuracy metric by aligning them to a SILVA reference of the known organism, e.g., to characterize a reads-per-consensus accuracy curve.

## When NOT to use

- The true organism reference sequence is unknown or the community is uncharacterized — the article emphasizes de novo consensus generation precisely to avoid reliance on reference databases
- Sequences are raw noisy nanopore reads rather than post-nanoClust consensus sequences; raw read error rates make direct reference-identity uninformative
- Data are short Illumina MiSeq or Ion Torrent PGM amplicon reads rather than full-length ONT MinION amplicon consensus sequences

## Inputs

- Full-length 16S rRNA gene consensus sequences (FASTA) from nanoClust within-cluster consensus calling
- Subsampled nanopore amplicon read sets (2D and 1D2) per OTU at varying depths (n = 5-100 reads)
- SILVA 16S rRNA reference sequence for the target organism

## Outputs

- Consensus sequence FASTA per subsampling depth
- Sequence identity (accuracy) values versus read count
- Accuracy-vs-depth plot (e.g., accuracy_curve.png) with marked plateau and stability regions

## How to apply

After running nanoClust read-partitioning-based de novo clustering and within-cluster consensus calling on subsampled read sets (e.g., n = 5 to 100 reads per OTU from the one-organism 2D and 1D2 ONT MinION amplicon datasets), retrieve the SILVA 16S reference sequence of the organism. Align each full-length consensus sequence against this reference and compute sequence identity as the accuracy metric. Vary the subsampled read depth and repeat subsampling plus consensus construction to characterize the accuracy-versus-depth relationship; report thresholds such as accuracy surpassing 99% above 5 reads, a plateau at approximately 10-15 reads, and reliable accuracy only with at least 50 reads. Because NanoAmpli-Seq residual errors are almost entirely deletions in homopolymer regions, inspect homopolymer loci if identity is unexpectedly low. Write the consensus FASTA and plot the accuracy curve (e.g., accuracy_curve.png) marking the depth regions.

## Related tools

- **nanoClust** (Produces the read-partitioning de novo clusters and within-cluster consensus 16S sequences that are then aligned to the SILVA reference for identity scoring) — github.com/umerijaz/nanopore
- **ONT MinION** (Nanopore platform supplying the 2D and 1D2 amplicon reads subsampled per OTU for consensus construction and accuracy evaluation)

## Examples

```
Rscript consensus_accuracy.R --reads one_organism_2D.fastq --depth 5:100 --ref silva_16S.fasta --out accuracy_curve.png
```

## Evaluation signals

- Consensus sequence identity against the SILVA 16S reference exceeds 99% when more than five reads are used per OTU consensus
- Accuracy plateaus at approximately 10-15 reads and is reliably high only when at least 50 reads are available
- Mock-community average consensus accuracy of 99.5% for both 2D and 1D2 sequencing is reproduced
- Residual mismatches in alignments are concentrated as deletions in homopolymer regions, not substitutions
- Alignment outputs are full-length 16S rRNA gene sequences, not truncated or orientation-corrected fragments (chopSeq applied beforehand)

## Limitations

- Nearly all residual errors in NanoAmpli-Seq consensus sequences originate from deletions in homopolymer regions, so identity is bounded by homopolymer base-calling quality
- Reliable consensus accuracy requires at least 50 reads per OTU; below that accuracy may not be stably high despite the 10-15 read plateau
- Accuracy assessment requires a correct SILVA reference sequence for the organism; reference-based evaluation fails for organisms absent from the database

## Evidence

- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [intro] mapping raw, noisy reads to custom or publicly available reference databases (i.e., SILVA): "mapping raw, noisy reads to custom or publicly available reference databases (i.e., SILVA)"
- [abstract] average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore."
