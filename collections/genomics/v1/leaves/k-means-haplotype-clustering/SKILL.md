---
name: k-means-haplotype-clustering
description: Use when building per-chromosome-end TVS signatures from telomap per-read
  output (sample.telomap.tsv) and reads at one chromosome end need to be partitioned
  into two haplotype groups to avoid mixing maternal/paternal telomere profiles.
when_to_use_negative:
- Reads cannot be unambiguously assigned to a specific chromosome end (e.g., subtelomeric
  anchor too short against T2T-CHM13 after RsaI/HinfI trimming), since haplotype balancing
  requires a single-end read set
- The goal is total telomere length measurement (TELOMERE_LEN) rather than TVS signature
  characterization, where haplotype separation is unnecessary
- Telomeres are from ALT cells, whose very long telomeres exceed PacBio HiFi read
  length and cannot be captured as full-length per-read profiles
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: telomap
  role: Produces the per-read table (sample.telomap.tsv) with telomeric region coordinates,
    detected MOTIF, and chromosome-end assignment used to build the per-read TVS profiles
    that k-means clusters
  repo: https://github.com/cytham/telomap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:37ebc8f9d7a9957a98609fee1928c4f7f2d73655e358d610b805f73f5dac151c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - telomap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# k-means-haplotype-clustering

## Summary

Applies k-means clustering (k=2) to per-read TVS profiles at a single mappable chromosome end so the two telomere haplotypes are balanced before generating the per-nucleotide TVS signature used to fingerprint patient samples.

## When to use

Use when building per-chromosome-end TVS signatures from telomap per-read output (sample.telomap.tsv) and reads at one chromosome end need to be partitioned into two haplotype groups to avoid mixing maternal/paternal telomere profiles.

## When NOT to use

- Reads cannot be unambiguously assigned to a specific chromosome end (e.g., subtelomeric anchor too short against T2T-CHM13 after RsaI/HinfI trimming), since haplotype balancing requires a single-end read set
- The goal is total telomere length measurement (TELOMERE_LEN) rather than TVS signature characterization, where haplotype separation is unnecessary
- Telomeres are from ALT cells, whose very long telomeres exceed PacBio HiFi read length and cannot be captured as full-length per-read profiles

## Inputs

- Chromosome-end-assigned per-read telomap output table (sample.telomap.tsv) with columns including CHROM, MOTIF, TELOMERE_START, TELOMERE_END, RAW_TELOMERE_LEN, TELOMERE_LEN
- Read sequences (FASTQ/BAM) underlying the telomap table for extracting telomeric repeat tracts
- Chromosome-end target specification (e.g., Chr 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, 17p)

## Outputs

- TVS signature plot per haplotype group (e.g., chr7q_tvs_signature.png) showing per-nucleotide TVS percentage over the first 3 kb of the telomeric region
- Pairwise Euclidean-distance TVS correlation scores between samples (tvs_correlation_scores.csv)
- Per-read haplotype cluster assignments (k=2 labels) for the target chromosome end

## How to apply

From the chromosome-end-assigned telomap table (e.g., reads with CHROM = Chr 7q), extract each read's telomeric region bounded by TELOMERE_START and TELOMERE_END, tabulate non-canonical telomeric variant sequences against the detected canonical 5'-TTAGGG-3' motif in the MOTIF column, and compute the per-nucleotide TVS percentage over the first 3 kb of the telomeric region. Treat each read's per-position TVS percentage vector as a feature row and run k-means with k=2, which balances the two haplotypes present at a single chromosome end so each cluster approximates one parental telomere. Generate the TVS signature plot (per-nucleotide TVS percentage profile per haplotype group, e.g., chr7q_tvs_signature.png) and compute pairwise Euclidean-distance-based TVS correlation scores across replicate samples to confirm the signature is patient-specific. The rationale is that TVSs are extremely heterogeneous and dispersed throughout the repeat region, so a haplotype-mixed profile would blur the individual-specific dispersed non-canonical motif pattern.

## Related tools

- **telomap** (Produces the per-read table (sample.telomap.tsv) with telomeric region coordinates, detected MOTIF, and chromosome-end assignment used to build the per-read TVS profiles that k-means clusters) — https://github.com/cytham/telomap

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=2, random_state=0).fit_predict(tvs_profiles_chr7q)
```

## Evaluation signals

- Replicate pairs must achieve a TVS correlation score greater than 0.90 in tvs_correlation_scores.csv
- TVS signature plot file (e.g., chr7q_tvs_signature.png) must exist and depict a dispersed distribution of non-canonical motifs across the telomeric region
- k-means converges to exactly k=2 clusters whose per-nucleotide TVS percentage profiles differ, consistent with two distinct haplotypes
- Per-nucleotide TVS percentages are computed over a fixed 3 kb window so profiles from different samples are directly comparable under Euclidean distance
- TVS signatures from unrelated patients show low correlation while within-patient replicates remain high, indicating the signature is patient-specific

## Limitations

- Subtelomeric regions can be too short after RsaI/HinfI trimming for unambiguous chromosome-end identification against T2T-CHM13, blocking haplotype clustering at that end
- The signature uses only the first 3 kb of the telomeric region, so variant distributions deeper in long telomeres are not represented
- Very long telomeres (e.g., ALT cells) exceed PacBio HiFi read length, so complete per-read TVS profiles cannot be obtained for clustering

## Evidence

- [readme] Analyze the distribution of non-canonical telomeric motifs (Telomere variant sequences (TVS)): "Analyze the distribution of non-canonical telomeric motifs (Telomere variant sequences (TVS))"
- [methods] Telomap identifies and analyzes telomeric reads: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [abstract] TVS heterogeneity dispersed throughout the repeat region: "Our results also reveal the extreme heterogeneity of telomeric variant sequences (TVSs) that are dispersed throughout the telomere repeat region"
- [readme] TELOMERE_LEN excludes non-telomeric sequences: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
