---
name: clinical-hla-type-comparison
description: Use when you have HLAminer prediction CSVs (HLAminer_HPRA.csv/HLAminer_HPTASR.csv)
  for individuals with clinically derived HLA types (e.g. NA12878, NA19240 from Tables
  4/5) and need to quantify 2-digit group-level concordance.
when_to_use_negative:
- The individual/sample has no clinically derived HLA types available, so there is
  no ground truth to compare against.
- You need allele-level (4-digit) concordance assessment — the article's benchmark
  is group-level (2-digit) resolution only.
- The goal is to generate the predictions themselves (alignment and streaming into
  HLAminer.pl) rather than to score already-produced HLAminer_HPRA.csv files.
tools:
- name: HLAminer
  role: Produces the HLAminer_HPRA.csv / HLAminer_HPRA.log prediction files and the
    concordance-relevant top-two-per-gene summary that this skill benchmarks against
    clinical types
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Upstream aligner whose streamed SAM output (-ax map-ont --MD against the chr6-masked
    GRCh38 + HLA-I/II reference) generated the predictions being scored
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:f4c4bc53034d0def840795364bb93e935909a532fbc7bf60e991f5be9719e790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - minimap2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clinical-hla-type-comparison

## Summary

Benchmarks HLAminer group-level (2-digit) HLA-I and HLA-II predictions from long-read WGS datasets against clinically confirmed HLA types, computing per-gene and per-class concordance (reported as 94% HLA-I and 88% HLA-II accuracy). This is the validation step that turns HLAminer_HPRA.csv prediction files into a quantified accuracy report.

## When to use

Use when you have HLAminer prediction CSVs (HLAminer_HPRA.csv/HLAminer_HPTASR.csv) for individuals with clinically derived HLA types (e.g. NA12878, NA19240 from Tables 4/5) and need to quantify 2-digit group-level concordance.

## When NOT to use

- The individual/sample has no clinically derived HLA types available, so there is no ground truth to compare against.
- You need allele-level (4-digit) concordance assessment — the article's benchmark is group-level (2-digit) resolution only.
- The goal is to generate the predictions themselves (alignment and streaming into HLAminer.pl) rather than to score already-produced HLAminer_HPRA.csv files.

## Inputs

- HLAminer_HPRA.csv summary (top two highest-scoring predictions per HLA gene, per dataset)
- HLAminer_HPRA.log (full mining process, all predictions by highest score, for resolving ambiguities)
- Clinically derived HLA types for the individual (article Tables 4 and 5; NA19240 from Supp Tab S6)
- hla_nom_p.txt P-designation file from http://hla.alleles.org/wmda/hla_nom_p.txt

## Outputs

- Per-gene concordance table (match/mismatch per HLA gene at 2-digit group resolution)
- Per-class concordance percentages for HLA-I (A, B, C) and HLA-II (DP, DQ, DR)
- hla_concordance_report.csv

## How to apply

Load each HLAminer_HPRA.csv (or HLAminer_HPTASR.csv) summary, which keeps the top two highest-scoring predictions per HLA gene, and reduce each prediction to 2-digit group resolution, accounting for P-designation alleles (only the 'P' designated allele is kept when sequences share the same antigen binding domain) and 'ex.' exclusion annotations. Load the clinically derived HLA types for the individual (e.g. from Tables 4 and 5 of the article, or per-individual tabulations such as the NA19240 reference Supp Tab S6) and truncate them to 2-digit group resolution. For each of the HLA genes (A, B, C for HLA-I; DQA1, DQB1, DRB1, DPA1, DPB1, DRA for HLA-II), match the two predicted alleles against the two reported alleles at group level. Aggregate matched genes per class and compute concordance percentages for HLA-I (A, B, C) and HLA-II (DP, DQ, DR); the reported benchmark to reproduce is all-but-one (94%) for HLA-I and all-but-three (88%) for HLA-II group-level accuracy. Write per-gene and per-class results to hla_concordance_report.csv so the comparison is auditable.

## Related tools

- **HLAminer** (Produces the HLAminer_HPRA.csv / HLAminer_HPRA.log prediction files and the concordance-relevant top-two-per-gene summary that this skill benchmarks against clinical types) — https://github.com/bcgsc/HLAminer
- **minimap2** (Upstream aligner whose streamed SAM output (-ax map-ont --MD against the chr6-masked GRCh38 + HLA-I/II reference) generated the predictions being scored) — https://github.com/lh3/minimap2

## Evaluation signals

- Computed group-level concordance reproduces the reported 94% HLA-I accuracy (all but one clinical type matched) and 88% HLA-II accuracy (all but three clinical types matched).
- hla_concordance_report.csv contains entries for all nine HLA genes (A, B, C, DQA1, DQB1, DRB1, DPA1, DPB1, DRA) per dataset.
- P-designation alleles are collapsed to a single entry and 'ex.' exclusions are handled before matching, so no gene is double-counted.
- Each gene comparison uses exactly the top two highest-scoring HLAminer predictions and exactly the two reported clinical alleles at 2-digit resolution.
- Predictions from low-coverage or older-chemistry datasets (e.g. 2018 NA19240 PromethION at ~10x) should still show robust group-level concordance per the article's findings.

## Limitations

- Accuracy is assessed at group-level (2-digit) resolution; allele-level (4-digit) resolution is not benchmarked, and short reads can make 4-digit predictions ambiguous per the HLAminer README.
- Concordance depends on clinically confirmed HLA types being available for the individual; datasets without ground truth cannot be scored this way.
- The reported benchmark used one PacBio HiFi and four ONT PromethION WGS datasets (older 2018 chemistry in some cases, down to ~10x coverage); performance on other chemistries or coverages may differ.

## Evidence

- [results] Group-level HLAminer predictions matched all but one (94% accuracy) and three (88% accuracy) of the clinically derived HLA-I and HLA-II types, respectively.: [evidence span withheld — anchor status: not_found]
- [readme] A more concise HLA allele summary in HLAminer_HPTASR.csv and HLAminer_HPRA.csv (associated .log is unchanged and lists all predictions): "A more concise HLA allele summary in HLAminer_HPTASR.csv and HLAminer_HPRA.csv (associated .log is unchanged and lists all predictions)"
- [readme] Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain.: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences"
- [methods] Results below demonstrate HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments: "HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments"
- [abstract] demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
