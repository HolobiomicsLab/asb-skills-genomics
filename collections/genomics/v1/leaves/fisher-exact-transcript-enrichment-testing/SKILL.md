---
name: fisher-exact-transcript-enrichment-testing
description: Use when per-read or per-site m6A and m5C calls (e.g. CHEUI Model 2 output)
  are available and the question is whether transcripts co-carried by both modification
  types are enriched beyond chance, as opposed to within-read co-occurrence.
when_to_use_negative:
- When the question is read-level co-occurrence of two sites within single mRNA molecules
  — use the read-level overlap statistic instead; this test only assesses transcript-level
  co-enrichment.
- When site calls come from orthogonal methods (miCLIP/m6ACE-seq) without a filtered
  CHEUI Model 2 probability table — the contingency classes are defined by the strict
  CHEUI >0.9999 threshold.
- When testing enrichment of co-occurring sites at a specific genomic distance (1-15
  nt) — that requires the distance-resolved observed-vs-expected permutation analysis,
  not a transcript-level contingency table.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: CHEUI
  role: supplies the per-transcript m6A and m5C site predictions (Model 2 probabilities)
    that populate the four contingency classes
  repo: https://github.com/comprna/CHEUI
- name: R2Dtool
  role: annotates and lifts the CHEUI methylation calls (cheui_to_bed.sh, R2_annotate,
    R2_lift) so modified sites can be assigned to protein-coding transcripts before
    tabulation
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:35f5cc8f600d5ceb4bfe5eb7c87ed874a509daa2117506380a836f531b9cb38a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - R2Dtool
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fisher-exact-transcript-enrichment-testing

## Summary

Tests whether mRNAs carrying both m6A and m5C modification sites occur more often than expected under independence, by tabulating CHEUI-derived site calls into a 2x2 transcript-level contingency table (both / only m6A / only m5C / neither) and running a two-tailed Fisher's exact test.

## When to use

Use when per-read or per-site m6A and m5C calls (e.g. CHEUI Model 2 output) are available and the question is whether transcripts co-carried by both modification types are enriched beyond chance, as opposed to within-read co-occurrence.

## When NOT to use

- When the question is read-level co-occurrence of two sites within single mRNA molecules — use the read-level overlap statistic instead; this test only assesses transcript-level co-enrichment.
- When site calls come from orthogonal methods (miCLIP/m6ACE-seq) without a filtered CHEUI Model 2 probability table — the contingency classes are defined by the strict CHEUI >0.9999 threshold.
- When testing enrichment of co-occurring sites at a specific genomic distance (1-15 nt) — that requires the distance-resolved observed-vs-expected permutation analysis, not a transcript-level contingency table.

## Inputs

- CHEUI-solo Model 2 site-level predictions (probability >0.9999) for m6A from pooled HEK293 WT replicates
- CHEUI-solo Model 2 site-level predictions (probability >0.9999) for m5C from the same samples
- Protein-coding transcript annotation (GRCh38 / Gencode v38; GRCm39 / Ensembl v104 for mouse cortex E12/E15/E18)
- Per-site read coverage table (sites with >=20 reads; rRNA reads/sites excluded)

## Outputs

- 2x2 contingency counts over transcripts (both m6A+m5C, only m6A, only m5C, neither)
- Two-tailed Fisher's exact test p-value for transcript-level co-enrichment (expected ~1.76E-06 in HEK293 WT)
- Row appended to co_occurrence_stats.tsv alongside distance-resolved co-occurrence and CCDC102A values

## How to apply

Start from the filtered set of protein-coding transcripts with at least two tested sites, each covered by >=20 reads, with sites called by CHEUI-solo Model 2 at probability >0.9999 (strict cut-offs used to keep the expected FDR low). Partition all tested mRNAs into four contingency classes: transcripts with both an m6A and an m5C site, only m6A sites, only m5C sites, and neither. Build a 2x2 table (m5C site present/absent vs m6A site present/absent over transcripts) and run a two-tailed Fisher's exact test to assess transcript-level co-enrichment. The paper reports p = 1.76E-06 for transcript-level enrichment in the HEK293 WT data, so the result should be reported alongside the read-level co-occurrence statistic rather than replacing it. Write the Fisher's exact test p-value to co_occurrence_stats.tsv together with the observed-vs-expected distance-resolved co-occurrence values and the CCDC102A pair statistic.

## Related tools

- **CHEUI** (supplies the per-transcript m6A and m5C site predictions (Model 2 probabilities) that populate the four contingency classes) — https://github.com/comprna/CHEUI
- **R2Dtool** (annotates and lifts the CHEUI methylation calls (cheui_to_bed.sh, R2_annotate, R2_lift) so modified sites can be assigned to protein-coding transcripts before tabulation) — https://github.com/comprna/R2Dtool

## Examples

```
fisher.test(matrix(c(n_both, n_m6a_only, n_m5c_only, n_neither), nrow=2), alternative="two.sided")
```

## Evaluation signals

- Contingency table row/column sums must equal the number of tested protein-coding transcripts after all filters (>=2 tested sites, >=20 reads/site, rRNA excluded).
- Two-tailed Fisher's exact test must be used (not one-tailed); paper reports p = 1.76E-06 for HEK293 WT transcript-level enrichment.
- Test result must be significant (p < 0.05) to pass validation alongside observed > expected read-level co-occurrence at 5-15 nt distances.
- Output p-value must be written to co_occurrence_stats.tsv with the distance-resolved co-occurrence values and the CCDC102A co-occurrence (~78%) statistic.
- Transcripts entering the 'both' class must have at least one m6A and one m5C site with Model 2 probability >0.9999.

## Limitations

- Very strict site cut-offs (Model 2 probability >0.9999) reduce CHEUI sensitivity; relaxing thresholds would recover more orthogonal sites but increases false positives, shifting contingency counts.
- Sites detected only from nanopore reads require independent validation; the Fisher enrichment reflects CHEUI calls, not validated biology.
- The mechanism underlying transcript-level m6A/m5C co-occurrence remains unexplained in the paper; a significant test does not establish causality.

## Evidence

- [methods] run a two-tailed Fisher's exact test on this partition to assess transcript-level enrichment: [evidence span withheld — anchor status: not_found]
- [results] paper reports p = 1.76E-06 for transcript-level enrichment: [evidence span withheld — anchor status: not_found]
- [abstract] co-occurrence of m6A and m5C in individual mRNAs in cell line and tissue transcriptomes: "CHEUI's capability to identify two modification types in the same sample reveals a co-occurrence of m6A and m5C in individual mRNAs in cell line and tissue transcriptomes."
- [discussion] strict cut-offs to maintain a low expected FDR: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)."
- [readme] CHEUI detects m6A and m5C at single read and site resolution: "CHEUI can be used to detect m6A and m5C modifications at single read resolution and to identified modified sites and their stoichiometry in a transcriptome annotation"
