---
name: pcoa-permanova-beta-diversity-analysis
description: Use when per-sample species/genera-resolved ARG abundance tables (cpg
  values from Argo) must be compared across sample groups (e.g., CN, KR, SG, DE, US
  human vs NH non-human primate) by composition rather than total abundance.
when_to_use_negative:
- Input is raw long reads or unprocessed Argo output files rather than aggregated
  per-sample abundance matrices
- The question concerns differences in total ARG abundance (cpg) per group — use Kruskal–Wallis
  with pairwise Mann–Whitney (Bonferroni-corrected) tests instead of ordination-based
  testing
- Reads are short reads or abundance data without species/genera resolution, since
  Argo's species-resolved cpg outputs are the expected basis for the dissimilarity
  matrix
tools:
- name: R
  role: Environment for running vegan ordination and PERMANOVA tests and for post-hoc
    statistical testing
- name: vegan
  role: Provides Bray–Curtis dissimilarities (vegdist), PCoA ('wcmdscale'), and PERMANOVA
    ('adonis2', 9999 permutations)
- name: ggplot2
  role: Renders PCoA biplots and annotated boxplots of the abundance comparisons
- name: Argo
  role: Produces the upstream species-resolved ARG tables (types/subtypes, hosts,
    cpg abundances) that feed the ordination
  repo: github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:2a244b21511802a1bd4cb9eda324f69cbb4e02d1f2e9d47eea90945b514dc06a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Kraken2
  - Melon
  - Porechop
  - R
  - ggplot2
  - nanoq
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pcoa-permanova-beta-diversity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Decomposes ARG abundance profiles (e.g., genera-level or ARG-type compositions in copies-per-genome from Argo outputs) into beta-diversity structure using principal coordinate analysis and tests group differences with PERMANOVA, as applied to compare human versus non-human primate fecal ARG profiles.

## When to use

Use when per-sample species/genera-resolved ARG abundance tables (cpg values from Argo) must be compared across sample groups (e.g., CN, KR, SG, DE, US human vs NH non-human primate) by composition rather than total abundance.

## When NOT to use

- Input is raw long reads or unprocessed Argo output files rather than aggregated per-sample abundance matrices
- The question concerns differences in total ARG abundance (cpg) per group — use Kruskal–Wallis with pairwise Mann–Whitney (Bonferroni-corrected) tests instead of ordination-based testing
- Reads are short reads or abundance data without species/genera resolution, since Argo's species-resolved cpg outputs are the expected basis for the dissimilarity matrix

## Inputs

- Per-sample matrix of ARG abundances by genera (cpg) from Argo/SARG+ profiling
- Per-sample matrix of ARG-type abundances (cpg) from Argo/SARG+ profiling
- Sample metadata table mapping sample IDs to groups (CN, KR, SG, DE, US, NH)

## Outputs

- PCoA ordination of samples with coordinates for biplot visualization
- PERMANOVA test statistics and p-values for group differences (9999 permutations)
- Bray–Curtis dissimilarity matrix
- ggplot2 PCoA biplots colored by group (human countries vs non-human primates)

## How to apply

Aggregate Argo per-sample outputs into matrices of ARG abundances (cpg) by genera or by ARG type. Compute Bray–Curtis dissimilarities between samples with vegan's vegdist, then ordinate with PCoA via vegan's 'wcmdscale' to visualize groupings (e.g., human countries vs NH primates). Test whether group centroids/dispersion differ using PERMANOVA with vegan's 'adonis2' and 9999 permutations. Plot PCoA biplots annotated with group centroids using ggplot2, and interpret group separation alongside univariate Kruskal–Wallis/Mann–Whitney tests on total cpg.

## Related tools

- **R** (Environment for running vegan ordination and PERMANOVA tests and for post-hoc statistical testing)
- **vegan** (Provides Bray–Curtis dissimilarities (vegdist), PCoA ('wcmdscale'), and PERMANOVA ('adonis2', 9999 permutations))
- **ggplot2** (Renders PCoA biplots and annotated boxplots of the abundance comparisons)
- **Argo** (Produces the upstream species-resolved ARG tables (types/subtypes, hosts, cpg abundances) that feed the ordination) — github.com/xinehc/argo

## Examples

```
ord <- vegan::wcmdscale(vegdist(arg_matrix, method="bray"), k=2); adonis2(arg_matrix ~ group, data=meta, permutations=9999)
```

## Evaluation signals

- PERMANOVA run with exactly 9999 permutations on Bray–Curtis dissimilarities of cpg matrices returns significant group separation (p < 0.05) between human groups and NH primates
- PCoA biplot shows NH primate samples separating from all human country groups (CN, KR, SG, DE, US) in genera and ARG-type space
- The dissimilarity matrix is symmetric with zero diagonal and derived from the same sample set (317 human + 12 chimp/bonobo fecal samples)
- Ordination results are consistent with the univariate validation: Kruskal–Wallis χ2 = 98.4 with adjusted p < 2e−16 and a decreasing CN→NH cpg trend
- Sul and mcr genes absent from NH primates should contribute to their distinct position in ARG-type space

## Limitations

- Argo marks ARGs of species below 1× genome copy as 'unclassified', so rare organisms may be excluded from the abundance matrices feeding the ordination
- Argo is reference-based and may misclassify unseen species or novel plasmid syntenies, propagating host-assignment uncertainty into the beta-diversity comparisons
- PERMANOVA differences can reflect dispersion as well as centroid shifts; interpret alongside univariate Kruskal–Wallis/Mann–Whitney tests

## Evidence

- [other] Decompose total ARG abundances by genera and ARG types using PCoA ('wcmdscale') and PERMANOVA ('adonis2') from vegan with Bray–Curtis dissimilarities and 9999 permutations.: [evidence span withheld — anchor status: not_found]
- [other] Validation signal for total-ARG comparison: [evidence span withheld — anchor status: not_found]
