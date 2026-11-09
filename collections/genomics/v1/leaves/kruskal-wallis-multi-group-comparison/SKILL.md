---
name: kruskal-wallis-multi-group-comparison
description: Use when comparing a per-sample numeric quantity (e.g., Argo total ARG
  cpg) across ≥3 independent groups — such as human fecal samples from CN, KR, SG,
  DE, US versus NH non-human primates — where normality cannot be assumed.
when_to_use_negative:
- Only two groups are being compared — a plain Mann–Whitney U test on the cpg values
  suffices without the multi-group Kruskal–Wallis step.
- Samples are paired or otherwise non-independent (e.g., longitudinal sampling of
  the same hosts), violating the independence assumption of the test.
- The goal is de novo discovery of ARG genomic contexts and hosts in new, understudied
  environments rather than group comparison of already-profiled cpg values — assembly-based
  (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are better suited
  there.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3941
tools:
- name: R
  role: Executes Kruskal–Wallis and Bonferroni-corrected pairwise Mann–Whitney tests
    on per-sample cpg values
- name: ggplot2
  role: Draws boxplots with Mann–Whitney/Kruskal–Wallis significance annotations
- name: Argo
  role: Upstream profiler producing the per-sample ARG cpg abundance table that the
    comparison tests
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:a4aecbd31f715cfa6614104847314f298212ae1b6bdd7340d8c75b0f6057aa0a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Kraken2
  - Melon
  - Porechop
  - R
  - ggplot2
  - nanoq
  - vegan
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# kruskal-wallis-multi-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-parametric testing of a per-sample continuous metric — here total ARG abundance expressed as copies per genome (cpg) from Argo — across three or more independent groups, using Kruskal–Wallis with Bonferroni-corrected pairwise Mann–Whitney follow-ups in R. It establishes group ordering (e.g., a decreasing CN→NH trend) without assuming normality.

## When to use

Use when comparing a per-sample numeric quantity (e.g., Argo total ARG cpg) across ≥3 independent groups — such as human fecal samples from CN, KR, SG, DE, US versus NH non-human primates — where normality cannot be assumed.

## When NOT to use

- Only two groups are being compared — a plain Mann–Whitney U test on the cpg values suffices without the multi-group Kruskal–Wallis step.
- Samples are paired or otherwise non-independent (e.g., longitudinal sampling of the same hosts), violating the independence assumption of the test.
- The goal is de novo discovery of ARG genomic contexts and hosts in new, understudied environments rather than group comparison of already-profiled cpg values — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are better suited there.

## Inputs

- Per-sample Argo abundance table of total ARG copies per genome (cpg), one row per fecal long-read metagenomic sample
- Group-label mapping for the 329-sample global fecal dataset (CN, KR, SG, DE, US, NH)
- Optionally genus/type-level Br.–Curtis dissimilarity matrices for downstream PCoA/PERMANOVA

## Outputs

- Kruskal–Wallis χ² statistic and p-value across the six groups
- Pairwise Mann–Whitney p-values with Bonferroni correction
- ggplot2 boxplots annotated with Kruskal–Wallis/Mann–Whitney significance
- Group ordering summary (decreasing CN→NH cpg trend, NH lowest)

## How to apply

Aggregate per-sample Argo outputs into a total ARG abundance (cpg) per sample, retaining group labels CN, KR, SG, DE, US, and NH (317 human plus 12 chimpanzee/bonobo samples). In R, run kruskal.test on cpg across the six groups, then follow with pairwise Mann–Whitney tests corrected with Bonferroni (pairwise.wilcox.test, p.adjust.method='bonferroni') to localize which groups differ. The validated outcome is Kruskal–Wallis χ2 = 98.4 with adjusted p < 2e−16 and a decreasing CN→NH trend, with NH primates distinctly lower in cpg than all human groups. Annotate ggplot2 boxplots with the Kruskal–Wallis and pairwise Mann–Whitney statistics for reporting. Decision point: restrict the input to species-resolved Argo calls above the 1× genome-copy threshold — 'unclassified' calls from sub-1× species should not enter the cpg aggregation.

## Related tools

- **R** (Executes Kruskal–Wallis and Bonferroni-corrected pairwise Mann–Whitney tests on per-sample cpg values)
- **ggplot2** (Draws boxplots with Mann–Whitney/Kruskal–Wallis significance annotations)
- **Argo** (Upstream profiler producing the per-sample ARG cpg abundance table that the comparison tests) — https://github.com/xinehc/argo

## Examples

```
kruskal.test(cpg ~ group, data = argo_res); pairwise.wilcox.test(argo_res$cpg, argo_res$group, p.adjust.method = "bonferroni")
```

## Evaluation signals

- Reproduced total-ARG cpg comparison yields Kruskal–Wallis χ2 = 98.4 with adjusted p < 2e−16
- Resulting group ordering shows a decreasing CN→NH trend, with NH primates distinctly lower in cpg than all human groups (CN, KR, SG, DE, US)
- All pairwise p-values are Bonferroni-adjusted before any significance claim is made
- Input cpg table covers all 329 samples (317 human + 12 chimpanzee/bonobo) and excludes Argo 'unclassified' sub-1× calls
- Boxplots display both the Kruskal–Wallis statistic and pairwise Mann–Whitney annotations

## Limitations

- Group sizes are highly unbalanced in the validated use case (317 human vs 12 NH primate samples), which limits power for NH-specific pairwise comparisons.
- Kruskal–Wallis detects distributional differences rather than median shifts alone, so the decreasing CN→NH ordering must be confirmed by the Bonferroni-adjusted pairwise tests.
- Insufficient coverage can make per-sample cpg unstable — Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified', and low-coverage samples can distort the group comparison.

## Evidence

- [methods] Aggregate per-sample outputs into total ARG abundance (cpg) per sample and test group differences across CN, KR, SG, DE, US, and NH with Kruskal–Wallis: [evidence span withheld — anchor status: not_found]
- [results] confirm the reproduced total-ARG cpg comparison yields Kruskal–Wallis χ2 = 98.4 with adjusted p < 2e−16 and a decreasing CN→NH trend: [evidence span withheld — anchor status: not_found]
- [intro] By leveraging a dataset comprising 329 human and non-human primate fecal samples, we illustrate Argo's effectiveness in surveilling ARGs within complex environments: [evidence span withheld — anchor status: not_found]
