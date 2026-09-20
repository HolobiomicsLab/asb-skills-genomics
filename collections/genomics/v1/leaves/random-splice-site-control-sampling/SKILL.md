---
name: random-splice-site-control-sampling
description: Use when testing whether a set of circular RNA BSJs (or any splice-site
  pairs) is enriched for a genomic feature (e.g. inverted Alu repeats) and you need
  a matched non-BSJ control set sampled from the same annotation.
when_to_use_negative:
- The goal is reconstructing full-length circRNA isoform sequences rather than testing
  genomic-feature enrichment of BSJs
- No gene annotation or RepeatMasker Alu BED is available for the reference genome,
  so matched control sampling or the window scan cannot be performed
- The hypothesis concerns Alu orientation effects in a specific locus rather than
  genome-wide enrichment, where a global sampling control is underpowered or mis-specified
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces the isocirc.out BSJ table whose known/novel splice-site classifications
    seed the case sets; its --Alu and --flank-len options define the Alu annotation
    and flanking-window scheme mirrored in the control scan
  repo: https://github.com/Xinglab/isoCirc
- name: bedtools
  role: Interval operations to extract 1000-nt and 2000-nt flanking intronic windows
    around each BSJ and control splice-site pair and intersect them with the Alu BED
    annotation
  repo: https://bedtools.readthedocs.io/
- name: Python
  role: Random sampling of 10,000 non-BSJ splice-site pairs from the GTF and running
    scipy.stats.fisher_exact two-sided tests; isoCirc itself is pip-installable as
    a Python package
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:7d6365392db0c2a14f26d935b8b10c0668b2509d0e60d7ae2d6fdb7629d123bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# random-splice-site-control-sampling

## Summary

Builds a matched negative-control set of splice-site pairs by randomly sampling 10,000 non-BSJ splice-site pairs from the same gene annotation, then applies the identical flanking-intron Alu window scan used for isoCirc-derived known and novel back-splice junctions (BSJs), enabling Fisher's exact tests of inverted-Alu enrichment with a two-sided design.

## When to use

Use when testing whether a set of circular RNA BSJs (or any splice-site pairs) is enriched for a genomic feature (e.g. inverted Alu repeats) and you need a matched non-BSJ control set sampled from the same annotation.

## When NOT to use

- The goal is reconstructing full-length circRNA isoform sequences rather than testing genomic-feature enrichment of BSJs
- No gene annotation or RepeatMasker Alu BED is available for the reference genome, so matched control sampling or the window scan cannot be performed
- The hypothesis concerns Alu orientation effects in a specific locus rather than genome-wide enrichment, where a global sampling control is underpowered or mis-specified

## Inputs

- isocirc.out table with BSJ genomic coordinates and known/novel splice-site classifications (HEK293 sample)
- Gene annotation file (GTF) for the same reference genome (GRCh37) used to sample random splice-site pairs
- GRCh37 RepeatMasker Alu annotation supplied as a BED file (isocirc --Alu input)
- Reference genome FASTA for extracting flanking intronic sequence

## Outputs

- Set of 10,000 sampled non-BSJ splice-site pairs (negative-control BSJ set)
- Contingency tables of inverted-Alu-positive vs Alu-negative counts for known BSJs, novel BSJs, and controls (1000-nt and 2000-nt windows)
- alu_enrichment_fisher_tests.tsv with Fisher's exact test p-values and odds ratios for each known/novel versus control comparison

## How to apply

Parse the isocirc.out table (produced by isoCirc from nanopore RCA reads against GRCh37) to obtain HEK293 BSJ coordinates tagged as known or novel splice-site pairs. From the same gene annotation (GTF), sample 10,000 random splice-site pairs that are NOT observed BSJs — this controls for genomic and annotation bias. For both the BSJ sets and the control set, apply the identical window-scan: extract 1000-nt and 2000-nt flanking intronic windows with bedtools and intersect with the GRCh37 RepeatMasker Alu BED supplied via the isocirc --Alu option (the --flank-len parameter controls flanking search length, default 500 in isoCirc, here overridden to 1000/2000). Classify Alu pairs by orientation relative to the gene (convergent, divergent, or both) and tabulate inverted-Alu-positive versus Alu-negative counts. Run two-sided Fisher's exact tests (scipy.stats.fisher_exact) for each known-vs-control and novel-vs-control comparison, recording p-values and odds ratios in alu_enrichment_fisher_tests.tsv.

## Related tools

- **isoCirc** (Produces the isocirc.out BSJ table whose known/novel splice-site classifications seed the case sets; its --Alu and --flank-len options define the Alu annotation and flanking-window scheme mirrored in the control scan) — https://github.com/Xinglab/isoCirc
- **bedtools** (Interval operations to extract 1000-nt and 2000-nt flanking intronic windows around each BSJ and control splice-site pair and intersect them with the Alu BED annotation) — https://bedtools.readthedocs.io/
- **Python** (Random sampling of 10,000 non-BSJ splice-site pairs from the GTF and running scipy.stats.fisher_exact two-sided tests; isoCirc itself is pip-installable as a Python package) — https://github.com/Xinglab/isoCirc

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[k_inv, k_neg], [c_inv, c_neg]], alternative='two-sided')
```

## Evaluation signals

- All known/novel versus control Fisher's exact p-values fall below the reported enrichment threshold of 3.6e-50
- The control set contains exactly 10,000 sampled splice-site pairs drawn from the same gene annotation, with no overlap with observed BSJs
- Control splice-site pairs underwent the identical 1000-nt and 2000-nt flanking-window scan and Alu-pair orientation classification as the BSJ sets
- Known BSJs show a higher proportion of inverted-Alu flanks than novel BSJs, consistent with the article's reported ordering
- alu_enrichment_fisher_tests.tsv contains both p-values and odds ratios for every known/novel versus control comparison

## Limitations

- Random splice-site pairs control only for genomic/annotation composition, not for features correlated with actual circularization such as host-gene expression or intron length
- isoCirc applies much stricter filtering to BSJs with novel splice sites, so the known-vs-novel proportion difference partly reflects detection-stringency asymmetry
- Results are genome- and annotation-specific (GRCh37, RepeatMasker Alu annotation); flanking-window length choice (1000 nt vs 2000 nt) can alter counts
- isoCirc default --flank-len is 500 nt, so window length must be explicitly set to match the 1000/2000-nt enrichment design

## Evidence

- [other] Control sampling design: [evidence span withheld — anchor status: not_found]
- [other] Enrichment finding and threshold: [evidence span withheld — anchor status: not_found]
- [readme] isoCirc Alu option grounding: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [abstract] Pipeline provenance of BSJ calls: "We describe an integrated computational pipeline to reliably characterize full-length circRNA isoforms using isoCirc data"
- [discussion] Novel-BSJ filtering asymmetry: "the filtering criteria are much stricter (as compared to BSJs involving known splice sites)"
