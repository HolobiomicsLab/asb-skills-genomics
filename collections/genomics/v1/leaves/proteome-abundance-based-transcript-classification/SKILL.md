---
name: proteome-abundance-based-transcript-classification
description: Use when testing whether per-gene poly(A) tail lengths (e.g., PAIso-seq
  GV oocyte measurements) correlate with protein abundance by splitting genes into
  detected vs non-detected proteome classes.
when_to_use_negative:
- Input is single-mRNA data (e.g., individualReporter-based translational control
  experiments) rather than transcriptome-wide per-gene tail lengths
- Proteome list lacks gene identifiers matching the tail-length table's gene keys,
  so the join by gene identifier would fail silently
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: R
  role: Joins the tail-length table with the protein-abundance classification by gene
    identifier, computes group means, and writes the summary TSV
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: minimap2
  role: Produces the mm10/GRCm38-aligned CCS reads from which per-gene geometric-mean
    poly(A) tail lengths are derived upstream of the classification
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:3d41d68976a19978d49319064aaf45f1bd8aa867e578a7d8882216f007b1f514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - Perl
  - R
  - minimap2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# proteome-abundance-based-transcript-classification

## Summary

Classify maternal transcripts into high vs low protein abundance groups by joining per-gene geometric-mean poly(A) tail lengths from PAIso-seq GV oocyte data with a GV oocyte mass-spectrometry proteome gene list, then test whether the positive tail-length–translation association holds transcriptome-wide.

## When to use

Use when testing whether per-gene poly(A) tail lengths (e.g., PAIso-seq GV oocyte measurements) correlate with protein abundance by splitting genes into detected vs non-detected proteome classes.

## When NOT to use

- Input is single-mRNA data (e.g., individualReporter-based translational control experiments) rather than transcriptome-wide per-gene tail lengths
- Proteome list lacks gene identifiers matching the tail-length table's gene keys, so the join by gene identifier would fail silently

## Inputs

- Per-gene geometric-mean poly(A) tail length table derived from PAIso-seq GV oocyte CCS reads (minimap2 mm10-aligned, TAPIS/pipeline-filtered tails)
- GV oocyte mass-spectrometry proteome gene list with detected/non-detected status

## Outputs

- Per-gene two-group summary table with protein-abundance group assignments and tail lengths (tail_length_protein_abundance.tsv)
- Group mean tail lengths (62 nt high-abundance vs 56 nt low-abundance) for statistical comparison

## How to apply

Load the per-gene geometric-mean poly(A) tail lengths computed from measured PAIso-seq tails (CCS reads requiring pass number ≥10, aligned to mm10/GRCm38 with minimap2 v2.15-r905). Load the GV oocyte mass-spectrometry proteome gene list and label each gene as high protein abundance if detected (2669 genes) or low protein abundance if not detected (1184 genes). Join the tail-length table with the protein-abundance classification by gene identifier in R, compute the mean tail length per group, and verify the reported values (62 nt for high, 56 nt for low abundance). Check per-gene assignments against the paper: Dnmt1/Tle6/Npm2/Zp2 have tails >60 nt while Btg4/Cnot7/Cnot6l/Plat have tails <60 nt. Write the two-group summary table with group assignments and tail lengths to tail_length_protein_abundance.tsv.

## Related tools

- **R** (Joins the tail-length table with the protein-abundance classification by gene identifier, computes group means, and writes the summary TSV) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **minimap2** (Produces the mm10/GRCm38-aligned CCS reads from which per-gene geometric-mean poly(A) tail lengths are derived upstream of the classification) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- High protein abundance group mean tail length ≈ 62 nt; low protein abundance group mean ≈ 56 nt
- Dnmt1, Tle6, Npm2, Zp2 assigned to >60 nt tail lengths; Btg4, Cnot7, Cnot6l, Plat assigned to <60 nt tail lengths
- Group sizes match expected counts: 2669 detected (high abundance) vs 1184 not-detected (low abundance) genes
- Output tail_length_protein_abundance.tsv contains one row per gene with both a group label and a tail length and no unjoined genes

## Limitations

- Protein abundance classes come from mass-spectrometry detection (detected vs not detected), so 'low abundance' conflates low expression with non-detection limits
- The correlation between PAIso-seq replicates is expected to improve with deeper sequencing, so per-gene tail lengths carry coverage-dependent uncertainty
- Association shown is for mouse GV oocytes; whether it holds transcriptome-wide in other cell types or species is not established by this analysis

## Evidence

- [intro] high-abundance group mean is 62 nt: "In combination with previous mass-spectrometry data of mouse GV oocytes, we find that the high protein abundance genes were with significantly longer poly(A) tails"
- [intro] tail-length association tested transcriptome-wide: "Translational control of several mRNAs by poly(A) tail length has been demonstrated in mouse oocytes5,14,16. However, it remains unknown wchether it is true transcriptome wide."
- [discussion] upstream tail measurement requires CCS pass filtering: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [abstract] per-gene tail lengths derived from measured PAIso-seq tails: "quantification of poly(A) tails along with the full-length cDNA while reading non-adenosine residues within poly(A) tails precisely"
- [readme] R/Perl/minimap2 pipeline produces per-gene tail lengths: "This analysis pipeline is implemented in Perl and R. Packages and software Versions used are listed below:"
