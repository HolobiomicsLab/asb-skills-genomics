---
name: telomeric-variant-sequence-distribution-analysis
description: Use when you need to characterize and compare the per-nucleotide distribution
  of non-canonical telomeric variant sequences (TVS) within telomere repeat regions
  from telomap-processed long reads, e.g. to test whether a TVS signature at a specific
  chromosome end is patient-unique.
when_to_use_negative:
- Telomere length estimation only (use RAW_TELOMERE_LEN/TELOMERE_LEN summaries instead
  of per-nucleotide TVS profiling)
- ALT cells with very long telomeres — the article notes the PacBio HiFi method is
  incompatible with measuring telomeres of ALT cells, so TVS signatures cannot be
  derived there
- Reads lacking unambiguous chromosome-end assignment because the subtelomeric region
  is too short (a known limitation of RsaI/HinfI trimming against T2T-CHM13)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Produces sample.telomap.tsv with per-read telomeric region coordinates (TELOMERE_START/TELOMERE_END),
    detected MOTIF, and TVS distribution analyses feeding the per-nucleotide TVS profiles
  repo: https://github.com/cytham/telomap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:c418c884216cf6d0c1fc2ee265db2b20c4cc61e913f189da374a7efe2788da8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - telomap
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# telomeric-variant-sequence-distribution-analysis

## Summary

Computes per-nucleotide TVS (telomeric variant sequence) percentage profiles over the first 3 kb of telomeric regions from chromosome-end-assigned PacBio HiFi telomeric reads using telomap, generates TVS signature plots per haplotype, and quantifies signature uniqueness across patient samples via pairwise Euclidean-distance TVS correlation scores.

## When to use

Use when you need to characterize and compare the per-nucleotide distribution of non-canonical telomeric variant sequences (TVS) within telomere repeat regions from telomap-processed long reads, e.g. to test whether a TVS signature at a specific chromosome end is patient-unique.

## When NOT to use

- Telomere length estimation only (use RAW_TELOMERE_LEN/TELOMERE_LEN summaries instead of per-nucleotide TVS profiling)
- ALT cells with very long telomeres — the article notes the PacBio HiFi method is incompatible with measuring telomeres of ALT cells, so TVS signatures cannot be derived there
- Reads lacking unambiguous chromosome-end assignment because the subtelomeric region is too short (a known limitation of RsaI/HinfI trimming against T2T-CHM13)

## Inputs

- Chromosome-end-assigned per-read telomap output table (sample.telomap.tsv) with columns including RNAME, MOTIF, TELOMERE_START, TELOMERE_END, TELOMERE_LEN
- Target chromosome-end identifier (e.g. Chr 7q, 2p, 3p, 5p, 6q, 11q, 12q, 14q, or 17p)

## Outputs

- TVS signature plot showing per-nucleotide TVS percentage profiles per haplotype group (e.g. chr7q_tvs_signature.png)
- Pairwise Euclidean-distance TVS correlation scores table (tvs_correlation_scores.csv)

## How to apply

Load the chromosome-end-assigned per-read table (sample.telomap.tsv) from telomap and filter reads assigned to the target mappable chromosome end (e.g. CHROM = Chr 7q). For each read, extract the telomeric region between TELOMERE_START and TELOMERE_END (TELOMERE_START is the 1-based position demarcated by at least two consecutive telomeric repeats 5'-TTAGGGTTAGGG-3') and tabulate non-canonical variant sequences against the canonical motif detected in the MOTIF column. Compute the per-nucleotide TVS percentage over the first 3 kb of the telomeric region by aggregating variant positions across reads. Run k-means clustering (k=2) on per-read TVS profiles to balance the two haplotypes at the chromosome end, then plot the per-nucleotide TVS percentage profile per haplotype group. Finally, compute pairwise Euclidean-distance-based TVS correlation scores between replicate-sample TVS profiles; scores >0.90 indicate the same underlying signature.

## Related tools

- **telomap** (Produces sample.telomap.tsv with per-read telomeric region coordinates (TELOMERE_START/TELOMERE_END), detected MOTIF, and TVS distribution analyses feeding the per-nucleotide TVS profiles) — https://github.com/cytham/telomap

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa -m TTAGGG
```

## Evaluation signals

- Replicate-sample pairs must achieve a pairwise TVS correlation score greater than 0.90 in tvs_correlation_scores.csv
- The TVS signature plot file (e.g. chr7q_tvs_signature.png) must exist and depict dispersed distribution of non-canonical motifs across the telomeric region
- Per-read filtering must retain only reads assigned to the target chromosome end (e.g. CHROM = Chr 7q)
- Per-nucleotide TVS percentage must be computed over exactly the first 3 kb of the telomeric region starting at TELOMERE_START
- k-means clustering (k=2) must yield two haplotype groups with balanced per-read TVS profiles

## Limitations

- Restriction enzymes cutting close to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length — distorting TVS distribution profiles
- Telomere-containing genomic DNA trimming is limited by PacBio HiFi DNA polymerase processivity, constraining observable telomeric region length
- The method is incompatible with measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi read lengths

## Evidence

- [readme] Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data.: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [methods] TELOMERE_START defined by consecutive telomeric repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [abstract] TVS heterogeneity finding: "Our results also reveal the extreme heterogeneity of telomeric variant sequences (TVSs) that are dispersed throughout the telomere repeat region"
- [methods] TVS distribution analysis workflow step: "Analyze the distribution of non-canonical telomeric motifs (Telomere variant sequences (TVS))"
- [discussion] MnlI cutting within TVSs: "we found that HphI and MnlI restriction endonucleases, which are known to cut closer to the telomere-containing region, frequently cut in the TVSs within the telomere repeat regions"
