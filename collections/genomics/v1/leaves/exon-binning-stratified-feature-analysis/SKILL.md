---
name: exon-binning-stratified-feature-analysis
description: Use when you have per-exon AISO measurements (upfi/dofi counts, Fupfi)
  plus extracted intron-exon features and want to reveal monotonic or non-monotonic
  feature–Fupfi relationships, e.g. reproducing Figure 6 of the Insplico study.
when_to_use_negative:
- You only have long-read RNA-seq without the 5'/3' bias correction module applied
  — uncorrected long-read biases will distort Fupfi bins relative to short-read patterns
- You need AISO information beyond the introns directly flanking the exon of interest
  — short reads barely provide splicing-order information beyond flanking introns
- The goal is per-exon (not population-level) AISO inference or quantification of
  AISO itself — use Insplico's AISO quantification, not this binning/plotting skill
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: Insplico
  role: Quantifies adjacent-intron splicing order around user-specified exons, yielding
    per-exon upfi/dofi counts and Fupfi that feed the binned analysis
  repo: gitlab.com/aghr/insplico
- name: Matt
  role: cmpr_features extracts the 55 intron-exon features (splice-site strengths,
    lengths, branch point-to-AG distances) used as binning variables
- name: dplyr
  role: Groups exons into five equal bins per feature and computes mean Fupfi per
    bin
- name: tidyr
  role: Reshapes the binned feature–Fupfi table for plotting and cross-species aggregation
- name: ggplot2
  role: Plots mean Fupfi per bin for each of the 55 features to visualize consensus
    associations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:bc04f6f3f9dbe0ea4c679e51645d29b5fb6401994f80b9624bb53eb6cc280316
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - Matt
  - R
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# exon-binning-stratified-feature-analysis

## Summary

Stratifies exons into five equal subsets per intron-exon feature (splice-site strengths, intron/exon lengths, branch point-to-AG distance), computes mean Fupfi per bin in R, and plots binned relationships with ggplot2 to expose universal genomic features associated with adjacent-intron splicing order (AISO) across species.

## When to use

Use when you have per-exon AISO measurements (upfi/dofi counts, Fupfi) plus extracted intron-exon features and want to reveal monotonic or non-monotonic feature–Fupfi relationships, e.g. reproducing Figure 6 of the Insplico study.

## When NOT to use

- You only have long-read RNA-seq without the 5'/3' bias correction module applied — uncorrected long-read biases will distort Fupfi bins relative to short-read patterns
- You need AISO information beyond the introns directly flanking the exon of interest — short reads barely provide splicing-order information beyond flanking introns
- The goal is per-exon (not population-level) AISO inference or quantification of AISO itself — use Insplico's AISO quantification, not this binning/plotting skill

## Inputs

- Per-exon AISO counts from Insplico (upfi, dofi, bus, bos counts; Fupfi)
- Intron-exon feature table from Matt's cmpr_features (55 features: splice-site strengths, intron/exon lengths, branch point-to-AG distances)
- Species-specific genome FASTA files and GTF annotations (human, mouse, fruitfly, round worm, Arabidopsis, rice)
- Short-read RNA-seq datasets (GEO GSE207459)

## Outputs

- Per-feature binned Fupfi table (five equal bins per feature with mean Fupfi per bin)
- ggplot2 panels of mean Fupfi per bin for each of the 55 features, per species and aggregated
- Consensus set of universal AISO-associated features across animal and plant species

## How to apply

Run Insplico on short-read RNA-seq data (e.g. the 13 nuclear/chromatin datasets from GEO GSE207459) to obtain per-exon upfi/dofi counts and the fraction of transcripts with the upstream intron spliced first (Fupfi). Extract 55 intron-exon features around each exon with Matt's cmpr_features, covering splice-site strengths, intron/exon lengths, and branch point-to-AG distances. In R (dplyr, tidyr), bin exons into five equal subsets per feature and compute mean Fupfi per bin, then plot mean Fupfi per bin for each of the 55 features with ggplot2. Aggregate the binned plots across human, mouse, fruitfly, round worm, Arabidopsis, and rice to derive consensus feature associations. The rationale is that binning smooths exon-level noise and exposes trend direction, enabling cross-species comparison of which features are universally associated with upfi (upstream intron removed first) versus dofi patterns.

## Related tools

- **Insplico** (Quantifies adjacent-intron splicing order around user-specified exons, yielding per-exon upfi/dofi counts and Fupfi that feed the binned analysis) — gitlab.com/aghr/insplico
- **Matt** (cmpr_features extracts the 55 intron-exon features (splice-site strengths, lengths, branch point-to-AG distances) used as binning variables)
- **dplyr** (Groups exons into five equal bins per feature and computes mean Fupfi per bin)
- **tidyr** (Reshapes the binned feature–Fupfi table for plotting and cross-species aggregation)
- **ggplot2** (Plots mean Fupfi per bin for each of the 55 features to visualize consensus associations)

## Examples

```
Rscript -e 'p_load(dplyr); p_load(tidyr); p_load(ggplot2); binned <- exons %>% mutate(bin = ntile(feature, 5)) %>% group_by(bin) %>% summarize(mean_Fupfi = mean(Fupfi)); ggplot(binned, aes(bin, mean_Fupfi)) + geom_line()'
```

## Evaluation signals

- Mean Fupfi rises monotonically with upstream 5' and 3' splice-site strength bins and with branch point proximity to the 3' splice site (AG)
- Mean Fupfi falls with increasing upstream intron length bins and rises with increasing downstream intron length bins
- Binned patterns are directionally consistent across all six species (human, mouse, fruitfly, round worm, Arabidopsis, rice), matching the reported universal associations
- Per-exon Fupfi is bounded in [0, 1] and each feature has exactly five equal-size bins before aggregation

## Limitations

- Short reads limit the maximum length of exons quantifiable by Insplico to the read length and paired-end fragment size, so binned analyses omit very long exons
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon
- A full mechanistic account of the universal AISO code would require integrating mathematical modelling of co-transcriptional splicing, RBP binding, chromatin features, and spliceosomal kinetics, which has not yet been done

## Evidence

- [abstract] A set of universal features associated with AISO patterns was established across various animal and plant species: "We also establish a set of universal features associated with AISO patterns across various animal and plant species"
- [discussion] Fupfi and surrounding AISO measurements around user-specified exons: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [readme] R environment and packages used for the binned analysis: "p_load(dplyr)
p_load(Biostrings)
p_load(polyester)
p_load(ggplot2)
p_load(tibble)"
- [discussion] Short-read length limitation on quantifiable exons: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
