---
name: per-nucleotide-motif-frequency-profiling
description: Use when telomeric long reads have already been chromosome-end-assigned
  (e.g., Chr 7q in sample.telomap.tsv) and the goal is to profile, cluster, and compare
  sample-specific TVS distribution signatures across the telomeric repeat region.
when_to_use_negative:
- The goal is telomere length distribution (TELOMERE_LEN histograms) rather than positional
  TVS variant-sequence signature comparison
- Input reads lack unambiguous subtelomeric anchor assignment to a specific chromosome
  end (e.g., subtelomeric regions too short against T2T-CHM13 after RsaI/HinfI trimming)
- Data are from ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase
  processivity, so full telomere tracts are not captured in reads
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: telomap
  role: Produces the chromosome-end-assigned per-read table (sample.telomap.tsv) and
    performs the TVS distribution analysis and analytic plotting used by this skill
  repo: https://github.com/cytham/telomap
- name: pbmm2
  role: Aligns PacBio HiFi reads upstream so telomeric reads can be identified and
    chromosome ends mapped
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:a38a94c9a297bb37b6995bca87b9f2e3b7f72266c8e6517d262ca1b5223e77bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pbmm2
  - telomap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# per-nucleotide-motif-frequency-profiling

## Summary

Quantifies the per-nucleotide frequency of non-canonical telomeric variant sequences (TVS) across the first 3 kb of individual chromosome-end telomeres from PacBio HiFi reads processed by telomap, yielding a dispersed TVS signature that is patient/sample-specific and evaluated by pairwise Euclidean-distance correlation.

## When to use

Use when telomeric long reads have already been chromosome-end-assigned (e.g., Chr 7q in sample.telomap.tsv) and the goal is to profile, cluster, and compare sample-specific TVS distribution signatures across the telomeric repeat region.

## When NOT to use

- The goal is telomere length distribution (TELOMERE_LEN histograms) rather than positional TVS variant-sequence signature comparison
- Input reads lack unambiguous subtelomeric anchor assignment to a specific chromosome end (e.g., subtelomeric regions too short against T2T-CHM13 after RsaI/HinfI trimming)
- Data are from ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase processivity, so full telomere tracts are not captured in reads

## Inputs

- Chromosome-end-assigned per-read telomap output table (sample.telomap.tsv) with RNAME, STRAND, MOTIF, TELOMERE_START, TELOMERE_END, RAW_TELOMERE_LEN, TELOMERE_LEN columns
- Telomeric motif reference (default 5'-TTAGGG-3')
- Target chromosome-end identifier (e.g., Chr 7q)

## Outputs

- TVS signature plot of per-nucleotide TVS percentage profile per haplotype group (chr7q_tvs_signature.png)
- Pairwise Euclidean-distance-based TVS correlation scores table (tvs_correlation_scores.csv)

## How to apply

Load the chromosome-end-assigned per-read table (sample.telomap.tsv) produced by telomap and filter reads to the target chromosome end (e.g., CHROM = Chr 7q). For each read, extract the telomeric interval defined by TELOMERE_START and TELOMERE_END — TELOMERE_START being the 1-based position at the 5' end of a region containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3') — and tabulate non-canonical variant sequences against the canonical motif detected in the MOTIF column (default TTAGGG). Aggregate variant positions across reads to compute the per-nucleotide TVS percentage over the first 3 kb of the telomeric region. Balance haplotypes at the chromosome end by k-means clustering (k=2) on per-read TVS profiles, then plot the per-nucleotide TVS percentage profile per haplotype group (chr7q_tvs_signature.png) and compute pairwise Euclidean-distance-based TVS correlation scores between replicate samples (tvs_correlation_scores.csv). The rationale is that TVSs show extreme heterogeneity dispersed throughout the telomere repeat region, so their positional distribution acts as a patient-specific fingerprint at mappable ends (e.g., Chr 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, 17p).

## Related tools

- **telomap** (Produces the chromosome-end-assigned per-read table (sample.telomap.tsv) and performs the TVS distribution analysis and analytic plotting used by this skill) — https://github.com/cytham/telomap
- **pbmm2** (Aligns PacBio HiFi reads upstream so telomeric reads can be identified and chromosome ends mapped) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=2, random_state=0).fit(per_read_tvs_profiles)
```

## Evaluation signals

- Replicate sample pairs achieve a pairwise TVS correlation score greater than 0.90 in tvs_correlation_scores.csv
- chr7q_tvs_signature.png exists and depicts a dispersed distribution of non-canonical motifs across the telomeric region rather than clustering at one position
- TELOMERE_START positions in the input table satisfy the telomap definition of at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')
- k-means clustering (k=2) yields two haplotype groups each with sufficient read coverage for stable per-nucleotide TVS percentages over the first 3 kb
- Per-nucleotide TVS percentages differ reproducibly between unrelated patient samples while remaining consistent within replicates, confirming signature uniqueness

## Limitations

- The signature requires unambiguous chromosome-end identification; RsaI/HinfI trimming can leave subtelomeric regions too short for unambiguous mapping against T2T-CHM13
- PacBio HiFi read length is limited by DNA polymerase processivity, making the method incompatible with ALT cells whose very long telomeres exceed read length
- Careful selection of restriction enzymes is necessary because HphI and MnlI cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length

## Evidence

- [methods] TVS distribution analysis of telomere variant sequences: "Analyze the distribution of non-canonical telomeric motifs (Telomere variant sequences (TVS))"
- [abstract] Dispersed TVS heterogeneity motivating signature uniqueness: "Our results also reveal the extreme heterogeneity of telomeric variant sequences (TVSs) that are dispersed throughout the telomere repeat region"
- [readme] TELOMERE_START column definition anchoring the profiling window: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] Anchor mapping dependency on T2T-CHM13 for chromosome-end assignment: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
