---
name: spearman-rank-correlation-accuracy-assessment
description: Use when benchmarking DRS or long-read transcript quantification tools
  against spike-in controls with known concentrations (e.g. SIRV E2), to rank tools
  by rank-preserving accuracy via Spearman's rho.
when_to_use_negative:
- Data have no ground-truth known abundances (e.g. human transcriptome samples without
  spike-ins) — rho cannot be computed against a reference truth
- The benchmark mix has equal concentrations across all transcripts (like SIRV E0),
  where rank correlation is undefined/meaningless; use coefficient of variance instead
- The goal is gene-level differential expression testing rather than isoform quantification
  accuracy — use DESeq2-based analysis as in the paper
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: NanoCount
  role: Candidate quantifier benchmarked; uses EM and DRS-specific filtering, expected
    to achieve the highest Spearman rho for E2
  repo: https://github.com/a-slide/NanoCount
- name: Salmon
  role: Competing quantifier run with --noErrorModel on the same alignments for comparison
- name: StringTie2
  role: Competing quantifier run in default mode
- name: FLAIR
  role: Competing quantifier run in default mode
- name: LIQA
  role: Competing quantifier run in default mode on the same alignments
- name: minimap2
  role: Aligns SIRV E0/E2 direct-RNA reads to the transcriptome with -ax map-ont and
    -N 10 to retain >=10 secondary mappings
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:aa25c85830472264451851796fd877debdb5421864ca6b5ffba4020ad4a4170d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - FLAIR
  - LIQA
  - NanoCount
  - Salmon
  - StringTie2
  - minimap2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spearman-rank-correlation-accuracy-assessment

## Summary

Assess transcript quantification accuracy by computing Spearman's rank correlation between known and measured isoform abundances, as done in the NanoCount paper's SIRV spike-in benchmark comparing NanoCount against Salmon, StringTie2, FLAIR and LIQA. Complemented by the coefficient of variance for the equal-concentration mix where ranks are uninformative.

## When to use

Use when benchmarking DRS or long-read transcript quantification tools against spike-in controls with known concentrations (e.g. SIRV E2), to rank tools by rank-preserving accuracy via Spearman's rho.

## When NOT to use

- Data have no ground-truth known abundances (e.g. human transcriptome samples without spike-ins) — rho cannot be computed against a reference truth
- The benchmark mix has equal concentrations across all transcripts (like SIRV E0), where rank correlation is undefined/meaningless; use coefficient of variance instead
- The goal is gene-level differential expression testing rather than isoform quantification accuracy — use DESeq2-based analysis as in the paper

## Inputs

- BAM/alignment file of direct-RNA reads mapped to the SIRV transcriptome reference (minimap2 -ax map-ont -N 10)
- Known SIRV transcript concentration table for the spike-in mix (equal-concentration E0 and varied-concentration E2)
- Per-tool transcript abundance estimates (NanoCount, Salmon, StringTie2, FLAIR, LIQA) under complete, insufficient and over annotations

## Outputs

- Spearman's rank correlation rho between known and measured abundance for mix E2, one value per quantification tool per annotation variant
- Coefficient of variance across transcripts for mix E0, one value per tool per annotation variant
- TSV benchmarking table comparable to Supplementary Table S1

## How to apply

Align the direct-RNA reads (e.g. SIRV mix SRR6058583) to the SIRV transcriptome reference with minimap2 using -ax map-ont and -N 10 so at least 10 secondary mappings are retained, then convert to BAM with samtools. Run each candidate quantifier (NanoCount v1.0.0 with its default DRS filtering — 3' end <=50nt and secondary AS >=95% of best alignment — plus Salmon with --noErrorModel, StringTie2, FLAIR and LIQA in default modes) on the same alignments. For the varied-concentration SIRV mix (E2), compute Spearman's rank correlation rho between known and measured transcript abundance per tool; for the equal-concentration mix (E0) Spearman's rho is uninformative because true abundances have no rank order, so compute the coefficient of variance across transcripts instead. Assemble per-tool rho and CV values across annotation variants (C, I, O) into a TSV benchmarking table mirroring Supplementary Table S1. Decision point: a tool is judged best when it shows the highest rho for E2 and lowest CV for E0 — in the source study NanoCount met this criterion, outperforming Salmon, StringTie2, FLAIR and LIQA.

## Related tools

- **NanoCount** (Candidate quantifier benchmarked; uses EM and DRS-specific filtering, expected to achieve the highest Spearman rho for E2) — https://github.com/a-slide/NanoCount
- **Salmon** (Competing quantifier run with --noErrorModel on the same alignments for comparison)
- **StringTie2** (Competing quantifier run in default mode)
- **FLAIR** (Competing quantifier run in default mode)
- **LIQA** (Competing quantifier run in default mode on the same alignments)
- **minimap2** (Aligns SIRV E0/E2 direct-RNA reads to the transcriptome with -ax map-ont and -N 10 to retain >=10 secondary mappings) — https://github.com/lh3/minimap2

## Examples

```
from scipy.stats import spearmanr; rho, pval = spearmanr(known_concentrations, measured_abundances)  # mix E2, per tool
```

## Evaluation signals

- Output table's per-tool rho and CV values match the corresponding entries of Supplementary Table S1 (output_matches_reference / expert_review)
- NanoCount exhibits the highest Spearman rho for E2 among Salmon, StringTie2, FLAIR and LIQA (per the paper's finding)
- NanoCount shows the least variability in measured abundances (lowest CV) for mix E0
- All rho values fall within [-1, 1] and rho for E0 is not computed (CV is used instead since concentrations are equal)
- Every tool is evaluated on the identical minimap2-produced BAM across all three annotation variants (C, I, O)

## Limitations

- SIRV/sequin spike-in transcriptomes are simpler than a human transcriptome, so caution is needed extrapolating benchmark conclusions to complex samples
- Rank correlation is only informative for mixes with varied concentrations (E2); it cannot discriminate tools on equal-concentration mixes (E0)
- Results depend on annotation variant (complete/insufficient/over annotation), so conclusions may not transfer across annotation regimes

## Evidence

- [results] We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [results] SIRV spike-in benchmark data: "using SIRV RNA spike-in DRS data (Materials and Methods)."
- [results] Concentration cutoff determination: "a segmental linear regression was utilised to help identify the concentration where quantitative measurement began."
- [readme] Multi-mapping uncertainty handled by EM in quantifiers: "using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads"
- [discussion] Caution extrapolating spike-in results: "some caution should be taken in extrapolating the sequin results due to the lower complexity of the sequin transcriptome"
