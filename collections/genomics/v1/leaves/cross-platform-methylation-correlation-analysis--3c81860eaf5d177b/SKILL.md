---
name: cross-platform-methylation-correlation-analysis
description: Use when genome-wide DNA methylation levels estimated from single-molecule
  sequencing (e.g., ONT mCaller motif-site calls) must be benchmarked against an orthogonal
  quantitative assay such as MethylFlash ELISA across multiple species or samples.
when_to_use_negative:
- Comparing per-site methylation calls across platforms rather than genome-wide aggregate
  levels — the article treats non-motif site discrepancies as method-specific false
  positives reflecting different noise sources
- Species or motifs for which the nanopore model was not trained — mCaller notably
  failed at one S. aureus motif and cannot confidently assign motifs in the two Dikarya
  fungi (C. neoformans, S. cerevisiae)
- Estimating m5C or other modifications — PacBio-based validation here addresses m6A
  and m4C; m5C is better distinguished in nanopore data but this ELISA correlation
  targets m6A only
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: mCaller
  role: Produces per-read m6A predictions from nanopolish eventalign data, whose per-position
    summary bed file (via make_bed.py) supplies the nanopore-derived percent m6A axis
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes reads, aligns them (with bwa mem or graphmap), and runs eventalign
    to produce the scaled event TSV that mCaller consumes
  repo: https://github.com/jts/nanopolish
- name: scipy
  role: scipy.stats.spearmanr computes the Spearman rank correlation between nanopore
    and ELISA percent m6A values
- name: matplotlib
  role: Renders the ELISA-vs-nanopore scatter plot annotated with the Spearman statistics
    (Fig 3b reproduction)
- name: seaborn
  role: Used with matplotlib for styling the correlation scatter plot
- name: pandas
  role: Aggregates the bed-file percent-methylated columns over motif positions per
    species
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:5e61d5681915b2aa9c750754ccc9a0f5ffd14ce82d7338613e54e08d59b94abb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - mCaller
  - matplotlib
  - nanopolish
  - pandas
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-platform-methylation-correlation-analysis

## Summary

Validates nanopore-derived genome-wide percent m6A estimates by correlating them with orthogonal experimental methylation measurements (MethylFlash ELISA) across microbial reference species, reproducing the Spearman correlation of the McIntyre et al. 2019 Nature Communications study.

## When to use

Use when genome-wide DNA methylation levels estimated from single-molecule sequencing (e.g., ONT mCaller motif-site calls) must be benchmarked against an orthogonal quantitative assay such as MethylFlash ELISA across multiple species or samples.

## When NOT to use

- Comparing per-site methylation calls across platforms rather than genome-wide aggregate levels — the article treats non-motif site discrepancies as method-specific false positives reflecting different noise sources
- Species or motifs for which the nanopore model was not trained — mCaller notably failed at one S. aureus motif and cannot confidently assign motifs in the two Dikarya fungi (C. neoformans, S. cerevisiae)
- Estimating m5C or other modifications — PacBio-based validation here addresses m6A and m4C; m5C is better distinguished in nanopore data but this ELISA correlation targets m6A only

## Inputs

- mCaller per-read methylation prediction file for ONT (nanopore) data from the eight microbial reference species
- Per-position methylation summary bed file (make_bed.py output with -d 15, -t 0.5)
- Published MethylFlash ELISA global percent m6A values for the same reference species

## Outputs

- Genome-wide percent m6A value per reference species (aggregated over motif positions)
- Spearman rank correlation (rho and p-value) between nanopore-derived and ELISA percent m6A
- Scatter plot of ELISA vs nanopore percent m6A annotated with rho and p-value (e.g., m6a_methylflash_spearman.png)

## How to apply

First generate per-position methylation summaries by running make_bed.py on the mCaller per-read prediction file with a minimum read depth of -d 15 and a methylation threshold of -t 0.5 (fraction of per-read observations labelled methylated at >= 50% predicted probability). Aggregate the bed-file percent-methylated columns over all motif positions per species to yield one genome-wide percent m6A value for each of the eight reference species. Retrieve the published MethylFlash ELISA global m6A values for the same species from the article's reported results. Compute the Spearman rank correlation between the two value sets using scipy.stats.spearmanr, expecting rho = 0.97 with p = 6.54E-6. Finally, plot the ELISA-vs-nanopore percent m6A scatter with matplotlib/seaborn, annotating rho and p-value, to reproduce Fig 3b.

## Related tools

- **mCaller** (Produces per-read m6A predictions from nanopolish eventalign data, whose per-position summary bed file (via make_bed.py) supplies the nanopore-derived percent m6A axis) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes reads, aligns them (with bwa mem or graphmap), and runs eventalign to produce the scaled event TSV that mCaller consumes) — https://github.com/jts/nanopolish
- **scipy** (scipy.stats.spearmanr computes the Spearman rank correlation between nanopore and ELISA percent m6A values)
- **matplotlib** (Renders the ELISA-vs-nanopore scatter plot annotated with the Spearman statistics (Fig 3b reproduction))
- **seaborn** (Used with matplotlib for styling the correlation scatter plot)
- **pandas** (Aggregates the bed-file percent-methylated columns over motif positions per species)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(nanopore_pct_m6a, elisa_pct_m6a)
```

## Evaluation signals

- Spearman rho between the eight genome-wide nanopore percent m6A values and MethylFlash ELISA values equals 0.97 (p = 6.54E-6)
- Each species yields exactly one aggregated genome-wide percent m6A value computed only over motif positions detected as methylated
- make_bed.py was run with the specified thresholds: minimum read depth -d 15 and mod_threshold -t 0.5
- The scatter plot is saved (e.g., as m6a_methylflash_spearman.png) and annotated with both rho and p-value, matching Fig 3b
- mCaller per-position accuracy at trained motifs falls within the reported 60–98% range, confirming the upstream calls feeding the correlation are sound

## Limitations

- mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif, which can skew the aggregate percent m6A for that species
- The method has limitations for sequence contexts on which the model has not been trained; training biases could decrease accuracy at contexts dissimilar to training examples
- Both single-molecule methods can fail to distinguish between types of methylation, as seen for the GATCGVNY motif in S. aureus
- Non-motif site calls are method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios, so only motif-site aggregates should be correlated

## Evidence

- [results] Calculations of percent m6A in the reference species based on motif sites detected as methylated by PacBio correlated highly (Spearman ρ = 0.97, p = 6.54E−6) with experimental values measured using an ELISA kit.: [evidence span withheld — anchor status: not_found]
- [methods] run summary script to generate a bed file of per-position methylation predictions: "run summary script to generate a bed file of per-position methylation predictions"
- [methods] `-d` indicates the minimum read depth for inclusion: "`-d` indicates the minimum read depth for inclusion"
- [methods] while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [discussion] mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
