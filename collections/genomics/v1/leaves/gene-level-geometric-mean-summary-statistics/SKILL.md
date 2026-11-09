---
name: gene-level-geometric-mean-summary-statistics
description: Use when a gene/isoform has multiple measured per-read tail lengths (e.g.
  from PAIso-seq CCS reads) and a single per-gene length is needed for group-level
  comparison against an external binary classification such as detected vs not-detected
  proteins.
when_to_use_negative:
- When per-isoform or per-tail-site resolution is required (e.g. APA analysis with
  the TAPIS pipeline), since this skill collapses tails to one value per gene
- When the proteome classification is graded (e.g. quantitative MS intensity) rather
  than binary detected/not-detected, as the group-contrast design assumes a two-class
  label
- When tail measurements come from a technology with a bounded length range (e.g.
  TAIL-seq's ~250-nt upper limit), where the full-tail distribution and geometric
  mean are not comparable
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Aligns clean CCS reads to the mouse mm10/GRCm38 reference so per-read tail
    calls can be attributed to genes before aggregation
- name: R
  role: Joins the per-gene geometric-mean tail-length table with the proteome classification
    by gene identifier and computes group means
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: Perl
  role: Runs the GV_oocyte_PAIsoSeqAnalysis scripts (e.g. GetCCSpass.pl, run.pl) that
    produce the per-read tail measurements feeding the summary statistics
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:27ee25547e69f5f40448f551e2d978ab365fefe11f6b2abafc599e418e489026
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - Perl
  - R
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gene-level geometric-mean summary statistics

## Summary

Aggregates the multiple measured PAIso-seq poly(A) tail lengths per gene into a single per-gene geometric-mean tail length (mouse GV oocyte data), then groups genes by protein-abundance status to test the transcriptome-wide association between tail length and translational output.

## When to use

Use when a gene/isoform has multiple measured per-read tail lengths (e.g. from PAIso-seq CCS reads) and a single per-gene length is needed for group-level comparison against an external binary classification such as detected vs not-detected proteins.

## When NOT to use

- When per-isoform or per-tail-site resolution is required (e.g. APA analysis with the TAPIS pipeline), since this skill collapses tails to one value per gene
- When the proteome classification is graded (e.g. quantitative MS intensity) rather than binary detected/not-detected, as the group-contrast design assumes a two-class label
- When tail measurements come from a technology with a bounded length range (e.g. TAIL-seq's ~250-nt upper limit), where the full-tail distribution and geometric mean are not comparable

## Inputs

- Per-read poly(A) tail measurements from PAIso-seq GV oocyte data (CCS-derived, tail calls per isoform/gene)
- Mouse GV oocyte mass-spectrometry proteome gene list (detected/not-detected labels)
- Gene identifier mapping between tail-length table and proteome table

## Outputs

- Per-gene geometric-mean poly(A) tail length table
- Two-group summary table with protein-abundance group assignments and tail lengths (tail_length_protein_abundance.tsv)
- Group-level mean tail lengths (62 nt high abundance, 56 nt low abundance)

## How to apply

From the PAIso-seq GV oocyte dataset, collect all measured tail lengths per gene (after CCS generation with --minPasses filtering, pass number ≥10, and tail definition of ≥15 nt clipped sequence with ≥5 continuous adenosines), and compute the per-gene geometric mean rather than the arithmetic mean to reduce skew from the heavy-tailed length distribution. In R, join this per-gene tail-length table with the mouse GV oocyte mass-spectrometry proteome list, labeling genes as high protein abundance (detected, 2669 genes) or low protein abundance (not detected, 1184 genes). Compute the mean tail length per group and verify the reported values (62 nt for high vs 56 nt for low abundance). Spot-check per-gene assignments: Dnmt1/Tle6/Npm2/Zp2 should exceed 60 nt while Btg4/Cnot7/Cnot6l/Plat should be below 60 nt. Write the annotated two-group table to tail_length_protein_abundance.tsv.

## Related tools

- **minimap2** (Aligns clean CCS reads to the mouse mm10/GRCm38 reference so per-read tail calls can be attributed to genes before aggregation)
- **R** (Joins the per-gene geometric-mean tail-length table with the proteome classification by gene identifier and computes group means) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **Perl** (Runs the GV_oocyte_PAIsoSeqAnalysis scripts (e.g. GetCCSpass.pl, run.pl) that produce the per-read tail measurements feeding the summary statistics) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt
```

## Evaluation signals

- Group mean tail lengths reproduce the reported values: 62 nt for high protein abundance genes and 56 nt for low protein abundance genes
- Group sizes match the proteome classification: 2669 detected genes and 1184 not-detected genes
- Per-gene spot checks: Dnmt1, Tle6, Npm2, Zp2 have mean tails >60 nt; Btg4, Cnot7, Cnot6l, Plat have mean tails <60 nt
- Every gene in the summary table has a defined geometric mean (no genes with zero measured tails silently dropped)
- Output TSV contains both the group assignment column and the per-gene tail length for all joined genes

## Limitations

- PAIso-seq per-read tail calls can include non-A residues within tail bodies; aggregation inherits any noise from the tail-definition filters (≥15 nt clipped, ≥5 continuous adenosines, non-A <50% and <20 residues)
- Current PacBio sequencing coverage limits replicate correlation; deeper sequencing would improve the per-gene length estimates feeding these statistics
- The geometric-mean aggregation discards within-gene isoform-specific tail-length heterogeneity and APA information

## Evidence

- [intro] High-abundance group mean verified from article finding: "In combination with previous mass-spectrometry data of mouse GV oocytes, we find that the high protein abundance genes were with significantly longer poly(A) tails"
- [methods] Tail-definition filters that must be applied before per-gene aggregation: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [readme] Implementation language of the upstream pipeline producing tail measurements: "This analysis pipeline is implemented in Perl and R. Packages and software Versions used are listed below:"
- [discussion] Alignment step attributing tail reads to genes on mm10: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905)"
