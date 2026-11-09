---
name: maternal-mrna-translational-regulation-analysis
description: Use when comparing single-cell poly(A) tail length measurements (e.g.
  PAIso-seq per-gene means from GV oocytes) against protein abundance groups to test
  whether tail length is associated with translational output transcriptome-wide.
when_to_use_negative:
- Input data is bulk-cell or non-oocyte poly(A) tail data where the GV oocyte protein-abundance
  gene list does not apply
- The goal is isoform-specific (per-transcript) tail–protein comparison rather than
  per-gene geometric-mean grouping
- Poly(A) tails have not passed the PAIso-seq QC filters (pass ≥10, ≥15 nt clip with
  ≥5 adenosines, non-A <50% and <20), so tail lengths are unreliable
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: minimap2
  role: Aligns clean CCS reads to the mouse mm10/GRCm38 reference genome (-x splice)
    as the basis for gene assignment and per-gene tail-length aggregation
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: R
  role: Joins tail-length table with the protein-abundance classification by gene
    identifier and computes group means and per-gene summaries
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: Perl
  role: Implements the PAIso-Seq tail-calling pipeline (run.pl, GetCCSpass.pl) that
    produces the per-gene tail lengths used in the association analysis
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: ccs
  role: Generates circular consensus sequences from PacBio subreads (e.g. --noPolish
    --minPasses 1 or --richQVs) feeding tail-length measurement
  repo: https://github.com/PacificBiosciences/IsoSeq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:341351419e55fe713181c993051fd2b31c2302299b3987f76b96066cfe61716c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - Perl
  - R
  - ccs
  - minimap2
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# maternal-mrna-translational-regulation-analysis

## Summary

Tests transcriptome-wide whether poly(A) tail length correlates with translational output in mouse GV oocytes by joining PAIso-seq per-gene geometric-mean tail lengths to mass-spectrometry protein-abundance labels (high = detected, 2669 genes; low = not detected, 1184 genes).

## When to use

Use when comparing single-cell poly(A) tail length measurements (e.g. PAIso-seq per-gene means from GV oocytes) against protein abundance groups to test whether tail length is associated with translational output transcriptome-wide.

## When NOT to use

- Input data is bulk-cell or non-oocyte poly(A) tail data where the GV oocyte protein-abundance gene list does not apply
- The goal is isoform-specific (per-transcript) tail–protein comparison rather than per-gene geometric-mean grouping
- Poly(A) tails have not passed the PAIso-seq QC filters (pass ≥10, ≥15 nt clip with ≥5 adenosines, non-A <50% and <20), so tail lengths are unreliable

## Inputs

- Per-gene geometric-mean poly(A) tail length table derived from PAIso-seq GV oocyte CCS reads (mouse mm10/GRCm38)
- GV oocyte mass-spectrometry proteome gene list with detection status (detected vs not detected)

## Outputs

- tail_length_protein_abundance.tsv — per-gene two-group summary with protein-abundance group assignment and poly(A) tail length
- Group mean tail lengths (expected: ~62 nt for high-abundance, ~56 nt for low-abundance genes)

## How to apply

Load the per-gene geometric-mean poly(A) tail lengths computed from measured PAIso-seq tails of GV oocytes, where tails passed filters (pass number ≥10, ≥15 nt clipped sequence with ≥5 continuous adenosines, non-A residues <50% and <20 in number). Load the GV oocyte mass-spectrometry proteome gene list and binarize each gene as high protein abundance (detected, 2669 genes) or low (not detected, 1184). Join the two tables by gene identifier in R (Perl scripts from niehu2018/GV_oocyte_PAIsoSeqAnalysis handle upstream tail calling; minimap2 v2.15-r905 aligned CCS reads to mm10/GRCm38). Compute group means and verify the reported values (62 nt high vs 56 nt low) and spot-check individual genes: Dnmt1/Tle6/Npm2/Zp2 should exceed 60 nt while Btg4/Cnot7/Cnot6l/Plat fall below 60 nt. Write the per-gene two-group table (gene, group assignment, tail length) to tail_length_protein_abundance.tsv.

## Related tools

- **minimap2** (Aligns clean CCS reads to the mouse mm10/GRCm38 reference genome (-x splice) as the basis for gene assignment and per-gene tail-length aggregation) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **R** (Joins tail-length table with the protein-abundance classification by gene identifier and computes group means and per-gene summaries) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **Perl** (Implements the PAIso-Seq tail-calling pipeline (run.pl, GetCCSpass.pl) that produces the per-gene tail lengths used in the association analysis) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **ccs** (Generates circular consensus sequences from PacBio subreads (e.g. --noPolish --minPasses 1 or --richQVs) feeding tail-length measurement) — https://github.com/PacificBiosciences/IsoSeq

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Group mean poly(A) tail lengths reproduce the reported values: high protein abundance ~62 nt, low protein abundance ~56 nt
- Spot-check gene assignments: Dnmt1, Tle6, Npm2, Zp2 have mean tails >60 nt; Btg4, Cnot7, Cnot6l, Plat have mean tails <60 nt
- Group sizes match the proteome annotation: 2669 high-abundance (detected) and 1184 low-abundance (not detected) genes after the join
- The output TSV has exactly two group labels and no genes lost to unresolvable identifier mismatches
- Upstream tails satisfy PAIso-seq QC: pass number ≥10, clipped sequence ≥15 nt with ≥5 continuous adenosines, non-A residues <50% and <20 in count

## Limitations

- Correlation of per-gene mean tail length with protein abundance does not establish isoform-level regulation; the transcriptome-wide association was previously demonstrated only for individual mRNAs
- PacBio sequencing cost (~$5 per 1-K reads, ~1000× Illumina per-read cost) limits coverage, and replicate correlation is expected to improve only with deeper sequencing
- PAIso-seq cannot accurately quantify non-A modifications at the very last 3′-end base, and extension primer pairing may introduce a minimal number of additional A residues, slightly biasing measured tails

## Evidence

- [intro] High-abundance vs low-abundance protein genes show longer tails: "In combination with previous mass-spectrometry data of mouse GV oocytes, we find that the high protein abundance genes were with significantly longer poly(A) tails"
- [intro] Transcriptome-wide question motivation: "Translational control of several mRNAs by poly(A) tail length has been demonstrated in mouse oocytes5,14,16. However, it remains unknown wchether it is true transcriptome wide."
- [readme] README workflow dependencies and alignment: "This analysis pipeline is implemented in Perl and R. Packages and software Versions used are listed below: Perl v5.26.2 R 3.5.1 minimap2 2.15-r905"
- [readme] minimap2 splice alignment to GRCm38: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa &>index.log"
- [discussion] Tail-calling QC thresholds: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
