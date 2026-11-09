---
name: long-read-circrna-consensus-sequence-calling
description: Use when nanopore long-read data generated from rolling-circle-amplified
  circRNAs must be converted into corrected consensus sequences, high-confidence BSJs,
  and full-length circRNA isoforms (isocirc.out/bed/stats outputs).
when_to_use_negative:
- Input is short-read RNA-seq, which does not experimentally determine the full-length
  sequences and exact exonic compositions of circRNAs
- The long reads are not RCA tandem-repeat products (e.g. plain ONT/PacBio cDNA reads)
  — the TRF consensus-calling model of tandem copies does not apply
- Consensus sequences have already been called and the task is only downstream isoform
  catalog merging
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0097
tools:
- name: isoCirc
  role: Wraps the full consensus-calling pipeline (LoRDEC correction, TRF consensus,
    minimap2 mapping, BSJ/isoform identification) behind a single command
  repo: https://github.com/Xinglab/isoCirc
- name: LoRDEC
  role: Hybrid error-correction of long reads with matching short-read data (--short-read,
    --kmer 21, --solid 3)
- name: Tandem Repeats Finder (TRF)
  role: Calls consensus sequences from RCA tandem-repeat reads (match 2, mismatch
    7, indel 7, match-frac 80, indel-frac 10, min-score 100, max-period 2000)
- name: minimap2
  role: Spliced alignment of filtered consensus sequences to the genome (high-quality
    ratios 1.1/0.9, identity 0.75)
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Required dependency (>= v2.27.0) used internally by the isocirc pipeline
  repo: https://bedtools.readthedocs.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:c7c65df368d11251afca41edef4040083e23b7aa5688a5d9591ea0b3d8db606a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - LoRDEC
  - Python
  - Tandem Repeats Finder (TRF)
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-circrna-consensus-sequence-calling

## Summary

Consensus-sequence calling step of the isoCirc pipeline: RCA tandem-repeat nanopore reads are hybrid error-corrected with LoRDEC, consensus sequences are called with Tandem Repeats Finder, filtered, and mapped to the genome with minimap2 to yield full-length circRNA isoform sequences.

## When to use

Use when nanopore long-read data generated from rolling-circle-amplified circRNAs must be converted into corrected consensus sequences, high-confidence BSJs, and full-length circRNA isoforms (isocirc.out/bed/stats outputs).

## When NOT to use

- Input is short-read RNA-seq, which does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs
- The long reads are not RCA tandem-repeat products (e.g. plain ONT/PacBio cDNA reads) — the TRF consensus-calling model of tandem copies does not apply
- Consensus sequences have already been called and the task is only downstream isoform catalog merging

## Inputs

- Long-read sequencing FASTA/FASTQ generated with isoCirc (RCA nanopore reads)
- Reference genome sequence FASTA (e.g. hg19)
- Gene annotation GTF (e.g. Ensembl GRCh37.87)
- circRNA annotation in BED/GTF format (multiple files joined with ',')
- Matching short-read FASTA/FASTQ for LoRDEC hybrid error correction

## Outputs

- Filtered consensus sequences per circRNA (min-len 30, min-copy 2.0)
- Genomic alignments of consensus sequences (minimap2)
- High-confidence back-splice junction list (isocirc.bed)
- Full-length circRNA isoform table (isocirc.out)
- Per-library summary statistics (isocirc_stats.out)

## How to apply

Run the isocirc command with the RCA long-read FASTA/FASTQ, reference genome FASTA (e.g. hg19), Ensembl GRCh37.87 gene annotation GTF, and circRNA annotation in BED/GTF format (multiple files joined with ','), supplying matching short reads for LoRDEC hybrid error correction with k-mer size 21 and solid k-mer abundance threshold 3. Call consensus sequences from RCA tandem-repeat reads with Tandem Repeats Finder using match 2, mismatch 7, indel 7, match-frac 80, indel-frac 10, min-score 100, max-period 2000; the rationale is that each RCA read contains tandem copies of the original circRNA, so TRF collapses repeats into a higher-accuracy consensus. Filter consensus sequences with min-len 30 and min-copy 2.0, then map them to the genome with minimap2 using high-quality alignment ratios 1.1/0.9 and identity 0.75. Identify high-confidence BSJs and full-length circRNAs using the canonical GT/AG motif, allowing at most 1 mis/ins/del in the 20-bp exonic flanks of a BSJ (0 in the 4-bp key flanks) and a minimum BSJ genomic distance of 150, with much stricter near-perfect alignment criteria applied to novel splice sites. Collect the three per-library result files (isocirc.out, isocirc.bed, isocirc_stats.out) for downstream merging of isoforms supported by at least 2 reads.

## Related tools

- **isoCirc** (Wraps the full consensus-calling pipeline (LoRDEC correction, TRF consensus, minimap2 mapping, BSJ/isoform identification) behind a single command) — https://github.com/Xinglab/isoCirc
- **LoRDEC** (Hybrid error-correction of long reads with matching short-read data (--short-read, --kmer 21, --solid 3))
- **Tandem Repeats Finder (TRF)** (Calls consensus sequences from RCA tandem-repeat reads (match 2, mismatch 7, indel 7, match-frac 80, indel-frac 10, min-score 100, max-period 2000))
- **minimap2** (Spliced alignment of filtered consensus sequences to the genome (high-quality ratios 1.1/0.9, identity 0.75)) — https://github.com/lh3/minimap2
- **bedtools** (Required dependency (>= v2.27.0) used internally by the isocirc pipeline) — https://bedtools.readthedocs.io/

## Examples

```
isocirc -t 4 --short-read short.fq --kmer 21 --solid 3 read.fq hg19.fa GRCh37.87.gtf circRNAs.bed output
```

## Evaluation signals

- isocirc_stats.out exists for every library and reports the consensus-calling/mapping stages completed
- All retained consensus sequences have length >= 30 and copy number >= 2.0 (matches min-copy filter)
- Reported full-length isoforms satisfy the canonical GT/AG motif with <= 1 mis/ins/del in the 20-bp exonic flanks and 0 in the 4-bp key flanks, at BSJ genomic distance >= 150
- Per-library high-confidence BSJ counts fall in the reported 47,434–66,194 range and isoform counts in the 41,095–57,763 range for HEK293 libraries
- Toy example in test_data runs end-to-end, producing isocirc.out, isocirc.bed, and isocirc_stats.out

## Limitations

- Filtering criteria are much stricter for BSJs involving novel splice sites than for known splice sites, so novel-junction sensitivity is lower
- At the current sequencing depth, isoCirc is far from saturation, and more circRNA isoforms will be identified with deeper sequencing
- The relatively low throughput and high per-sequence cost of long-read RNA-seq make deeply sampling the transcriptome prohibitively expensive

## Evidence

- [methods] Consensus calling with TRF under stringent settings: "Consensus calling with Tandem Repeats Finder (TRF)):"
- [methods] Filtering and mapping of consensus sequences: "Filtering and mapping of consensus sequences (minimap2):"
- [discussion] Stringent post-consensus false-positive removal: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [discussion] Stricter criteria for novel splice sites: "requiring near-perfect alignment between the isoCirc consensus sequence and the corresponding genomic sequence around the BSJ"
- [readme] Pipeline dependencies: [evidence span withheld — anchor status: not_found]
