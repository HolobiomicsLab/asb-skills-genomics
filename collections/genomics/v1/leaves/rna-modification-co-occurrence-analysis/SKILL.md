---
name: rna-modification-co-occurrence-analysis
description: Use when you have per-read modification predictions for two modification
  types (e.g. CHEUI-solo m6A and m5C calls) on the same direct RNA reads and want
  to test single-molecule co-occurrence at pairs of modified sites 1–15 nt apart.
when_to_use_negative:
- The input is only site-level (stoichiometry) data with no per-read modification
  calls — single-molecule co-occurrence cannot be computed without read-level state
  at both sites.
- Transcripts carry only one tested modification site — pairs at 1–15 nt distance
  cannot be formed, so co-occurrence is undefined.
- The question is differential methylation between two conditions (use CHEUI-diff),
  not co-occurrence of two modification types within the same reads.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CHEUI
  role: Produces the per-read (Model 1) and per-site (Model 2) m6A and m5C modification
    predictions from nanopore direct RNA signals that feed the co-occurrence statistics
  repo: https://github.com/comprna/CHEUI
- name: R2Dtool
  role: Used (cheui_to_bed.sh, R2_annotate, R2_lift) to convert CHEUI calls to bed-like
    format and annotate/lift site positions so relative distances between modified
    sites in the same transcript can be computed
  repo: https://github.com/comprna/R2Dtool
- name: nanopore direct RNA sequencing
  role: 'Data source: single-molecule RNA reads whose ionic-current signals CHEUI
    classifies as m6A- or m5C-modified per molecule'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:bac44e4864c07ea4db96bb8fd82f9b6b9c9e6efefc77973053662aef0c86ad1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - R2Dtool
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# rna-modification-co-occurrence-analysis

## Summary

Read-level analysis of whether m6A and m5C modifications co-occur within individual mRNA molecules more often than expected by chance, using CHEUI-solo Model 1 and Model 2 per-read predictions from nanopore direct RNA sequencing (HEK293 WT and mouse cortex E12/E15/E18).

## When to use

Use when you have per-read modification predictions for two modification types (e.g. CHEUI-solo m6A and m5C calls) on the same direct RNA reads and want to test single-molecule co-occurrence at pairs of modified sites 1–15 nt apart.

## When NOT to use

- The input is only site-level (stoichiometry) data with no per-read modification calls — single-molecule co-occurrence cannot be computed without read-level state at both sites.
- Transcripts carry only one tested modification site — pairs at 1–15 nt distance cannot be formed, so co-occurrence is undefined.
- The question is differential methylation between two conditions (use CHEUI-diff), not co-occurrence of two modification types within the same reads.

## Inputs

- CHEUI-solo Model 1 per-read m6A prediction tables for pooled HEK293 WT replicates
- CHEUI-solo Model 1 per-read m5C prediction tables for the same samples
- CHEUI-solo Model 2 site-level probabilities for m6A and m5C (site significance filtering)
- Transcript annotations for protein-coding transcripts (Gencode v38 / GRCh38 for human; GRCm39 / Ensembl v104 for mouse) used to compute relative site distances and exclude rRNA

## Outputs

- co_occurrence_stats.tsv with observed-vs-expected distance-resolved read-level co-occurrence values (1–15 nt), the Fisher's exact test p-value for transcript-level enrichment, and the CCDC102A pair co-occurrence fraction

## How to apply

Load the CHEUI-solo Model 1 per-read m6A and m5C predictions for the pooled HEK293 WT replicates (and mouse cortex E12/E15/E18 samples), retaining only protein-coding transcripts with at least two tested sites covered by >=20 reads and excluding rRNA reads/sites. Select transcripts containing a pair of significantly modified sites (Model 2 probability >0.9999) at a relative distance of 1–15 nt. For each selected pair compute the observed read-level co-occurrence as the number of reads carrying the same modification state at both sites divided by the total reads covering both sites; estimate the expected background by sampling 1000 random pairs of modified sites from different transcripts and recomputing the same per-read overlap statistic. Additionally partition all tested mRNAs into four contingency classes (both m6A and m5C sites, only m6A, only m5C, neither) and run a two-tailed Fisher's exact test on this partition to assess transcript-level enrichment, then extract the CCDC102A transcript pair co-occurrence fraction as a single-transcript example.

## Related tools

- **CHEUI** (Produces the per-read (Model 1) and per-site (Model 2) m6A and m5C modification predictions from nanopore direct RNA signals that feed the co-occurrence statistics) — https://github.com/comprna/CHEUI
- **R2Dtool** (Used (cheui_to_bed.sh, R2_annotate, R2_lift) to convert CHEUI calls to bed-like format and annotate/lift site positions so relative distances between modified sites in the same transcript can be computed) — https://github.com/comprna/R2Dtool
- **nanopore direct RNA sequencing** (Data source: single-molecule RNA reads whose ionic-current signals CHEUI classifies as m6A- or m5C-modified per molecule)

## Examples

```
python co_occurrence_analysis.py --cheui_m6A preds_m6A.tsv --cheui_m5C preds_m5C.tsv --model2_prob_threshold 0.9999 --min_reads 20 --min_distance 1 --max_distance 15 --n_random_pairs 1000 --out co_occurrence_stats.tsv
```

## Evaluation signals

- Observed read-level co-occurrence must exceed the permuted expectation (1000 random inter-transcript site pairs) at the 5–15 nt distances.
- Two-tailed Fisher's exact test on the four contingency classes (both / only m6A / only m5C / neither) must be significant; the paper reports p = 1.76E-06 for transcript-level enrichment.
- The CCDC102A transcript pair co-occurrence fraction should be near the reported ~78%.
- All analyzed pairs must derive from significantly modified sites with Model 2 probability >0.9999 and per-site coverage >=20 reads; rRNA reads/sites must be excluded.
- Output co_occurrence_stats.tsv must contain the distance-resolved observed/expected values, the Fisher p-value, and the CCDC102A statistic.

## Limitations

- Very strict cut-offs on cell transcriptome sites (needed to keep the expected FDR low across the large number of sites tested) reduce CHEUI sensitivity; relaxing thresholds would recover more sites at the cost of false positives.
- Modified sites detected only from nanopore reads require independent validation experiments to confirm the predictions.
- The mechanisms underlying the co-occurrence of modifications in reads and across transcripts remain to be elucidated.
- Signal interference at distances <5 nt would be exacerbated in highly and densely modified molecules such as rRNA, which motivates the 1–15 nt pair selection and rRNA exclusion.

## Evidence

- [results] Read-level modification co-occurrence ... higher than expected by chance for both m6A and m5C modifications in HEK293 WT reads.: [evidence span withheld — anchor status: not_found]
- [discussion] Select transcripts containing a pair of significantly modified sites at 1-15 nt distance (workflow card).: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)."
- [abstract] The paper reports p = 1.76E-06 for transcript-level enrichment of co-occurrence.: "co-occurrence of m6A and m5C in individual mRNAs in cell line and tissue transcriptomes"
- [discussion] rRNA exclusion rationale: signal interference at short distances is exacerbated in densely modified rRNA.: "This effect on the individual read prediction would be exacerbated in ribosomal RNA molecules since they are highly and densely modified."
- [readme] CHEUI can predict m6A and m5C from the same sample in individual molecules, enabling same-molecule co-occurrence tests.: [evidence span withheld — anchor status: external_doc]
