---
name: per-read-methylation-aggregation
description: Use when you have CHEUI Model 1 read-level m6A/m5C predictions from pooled
  nanopore direct RNA replicates and need to define significantly modified sites,
  filter transcript/site coverage, and compute single-molecule modification statistics
  such as co-occurrence.
when_to_use_negative:
- Sites or molecules in ribosomal RNA, which are highly and densely modified and where
  signal interference at distances <5 nt exacerbates per-read prediction errors
- Differential methylation between two conditions — use CHEUI-diff instead of aggregating
  per-read states
- When the input is raw FAST5 signal with no basecalling/alignment/eventalign preprocessing;
  CHEUI preprocessing requires Guppy-basecalled, minimap2-aligned, nanopolish-eventaligned
  data first
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: CHEUI
  role: Two-stage neural network that predicts m6A and m5C in individual reads (Model
    1) and site stoichiometry/modification probability (Model 2) from nanopolish-eventaligned
    nanopore signals
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish
  role: Resquiggles raw signals to aligned transcript sequences via eventalign before
    CHEUI preprocessing
- name: Minimap2
  role: Aligns Guppy-basecalled reads to the reference transcriptome with primary
    positive-strand alignments selected via samtools -F 232457
- name: R2Dtool
  role: Converts CHEUI methylation calls to bed-like format (cheui_to_bed.sh), annotates
    sites against a GTF (R2_annotate), and lifts transcriptome coordinates to the
    genome (R2_lift)
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:67a09b3c2960472ff16b639b3d6420a2977253c4095b7308a0bfef065d0ce40a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Minimap2
  - Nanopolish
  - R2Dtool
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# per-read-methylation-aggregation

## Summary

Aggregates CHEUI-solo Model 1 per-read m6A/m5C probabilities with Model 2 site-level modification probabilities (threshold >0.9999) to identify significantly modified sites and quantify read-level co-occurrence of two modification types on the same mRNA molecules in nanopore direct RNA data.

## When to use

Use when you have CHEUI Model 1 read-level m6A/m5C predictions from pooled nanopore direct RNA replicates and need to define significantly modified sites, filter transcript/site coverage, and compute single-molecule modification statistics such as co-occurrence.

## When NOT to use

- Sites or molecules in ribosomal RNA, which are highly and densely modified and where signal interference at distances <5 nt exacerbates per-read prediction errors
- Differential methylation between two conditions — use CHEUI-diff instead of aggregating per-read states
- When the input is raw FAST5 signal with no basecalling/alignment/eventalign preprocessing; CHEUI preprocessing requires Guppy-basecalled, minimap2-aligned, nanopolish-eventaligned data first

## Inputs

- CHEUI-solo Model 1 per-read m6A and m5C prediction files (read ID, transcript, site, probability) from pooled nanopore direct RNA replicates
- CHEUI-solo Model 2 site-level stoichiometry/probability output for the same samples
- Reference transcriptome annotation (Gencode v38 GTF for human GRCh38; Ensembl v104 GTF for mouse GRCm39)
- Aligned BAM files produced by minimap2 + samtools -F 232457 filtering
- Nanopolish eventalign output with --scale-events --signal-index --samples --print-read-names

## Outputs

- Set of significantly modified transcriptomic sites (Model 2 probability >0.9999) with coverage >=20 reads
- Observed vs expected distance-resolved read-level co-occurrence fractions per site pair
- Transcript-level contingency table (both/only-m6A/only-m5C/neither) with two-tailed Fisher's exact p-value (paper reports p = 1.76E-06)
- Per-transcript co-occurrence statistics (e.g. CCDC102A pair at ~78%) written to co_occurrence_stats.tsv

## How to apply

Load per-read CHEUI-solo Model 1 m6A and m5C predictions for pooled replicates (e.g. HEK293 WT, mouse cortex E12/E15/E18), retaining only protein-coding transcripts with at least two tested sites, each covered by >=20 reads, and excluding rRNA reads/sites. Define significantly modified sites using CHEUI-solo Model 2 site-level probabilities with a strict cutoff of >0.9999, chosen because the large number of tested transcriptomic sites demands a very low expected false discovery rate. For read-level co-occurrence, select site pairs at a relative distance of 1-15 nt within the same transcript and compute the observed co-occurrence as reads sharing the same modification state at both sites divided by total reads covering both sites; estimate expectation by sampling 1000 random pairs of modified sites from different transcripts. Assess transcript-level enrichment by partitioning mRNAs into four contingency classes (both m6A and m5C, only m6A, only m5C, neither) and running a two-tailed Fisher's exact test. R2Dtool scripts (cheui_to_bed.sh, R2_annotate, R2_lift) can convert and annotate the CHEUI methylation calls against Gencode/Ensembl GTFs.

## Related tools

- **CHEUI** (Two-stage neural network that predicts m6A and m5C in individual reads (Model 1) and site stoichiometry/modification probability (Model 2) from nanopolish-eventaligned nanopore signals) — https://github.com/comprna/CHEUI
- **Nanopolish** (Resquiggles raw signals to aligned transcript sequences via eventalign before CHEUI preprocessing)
- **Minimap2** (Aligns Guppy-basecalled reads to the reference transcriptome with primary positive-strand alignments selected via samtools -F 232457)
- **R2Dtool** (Converts CHEUI methylation calls to bed-like format (cheui_to_bed.sh), annotates sites against a GTF (R2_annotate), and lifts transcriptome coordinates to the genome (R2_lift)) — https://github.com/comprna/R2Dtool

## Examples

```
nanopolish eventalign -t 48 --reads <read.fastq> --bam <sorted.bam> --genome <transcriptome.fa> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- Observed read-level co-occurrence must exceed the 1000-random-pair permuted expectation at the 5-15 nt distances
- Fisher's exact test on the four-class transcript partition must be significant; paper reports p = 1.76E-06 for transcript-level enrichment
- The CCDC102A transcript pair co-occurrence fraction should be near the reported ~78%
- All retained sites must have CHEUI-solo Model 2 probability >0.9999 and >=20 covering reads; transcripts must be protein-coding with at least two tested sites and rRNA excluded

## Limitations

- Very strict per-read/site cut-offs are needed to maintain a low expected FDR given the large number of sites tested, which reduces CHEUI sensitivity; many sites found by orthogonal methods have high CHEUI probabilities just below the thresholds
- Nanopore signal variability limits per-read discrimination: differences between modified and non-modified reads are often comparable to variation among non-modified reads
- Modified sites detected only from nanopore reads require independent validation experiments
- The mechanisms underlying the identified co-occurrence of modifications in reads and across transcripts remain to be elucidated

## Evidence

- [abstract] CHEUI predicts N6-methyladenosine (m6A) and 5-methylcytosine (m5C) in individual molecules from the same sample, the stoichiometry at transcript reference sites: "CHEUI predicts N6-methyladenosine (m6A) and 5-methylcytosine (m5C) in individual molecules from the same sample, the stoichiometry at transcript reference sites, and differential methylation between"
- [methods] Select transcripts with significantly modified site pairs at 1-15 nt distance: [evidence span withheld — anchor status: not_found]
- [discussion] Strict cutoffs to maintain low FDR reduce sensitivity: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [readme] Nanopolish eventalign command from CHEUI README: [evidence span withheld — anchor status: not_found]
- [methods] Read filtering with samtools flags: "Reads were then filtered to select the best match for each read using samtools -F 232457."
