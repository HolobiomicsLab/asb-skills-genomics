---
name: sequence-motif-logo-visualization
description: Use when you have a set of genomic or transcriptomic modification sites
  (e.g. CHEUI-diff m5C calls) and need to verify biological plausibility by checking
  enrichment of a known sequence motif around the modified positions.
when_to_use_negative:
- The modification type has no known sequence-context motif (logo would show no interpretable
  pattern and provide no validation signal)
- Input is raw nanopore signal or FAST5 files rather than a curated list of called
  sites — motif analysis must follow modification calling and annotation, not precede
  it
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: WebLogo
  role: Compute sequence logos of nucleotides around NSUN2-dependent m5C positions
    to confirm enrichment of the 5'-m5CNGGG-3' motif
- name: R2Dtool
  role: Annotate and lift CHEUI differential m5C calls to genomic coordinates (R2_annotate
    with Gencode v38 GTF, R2_lift) so flanking sequences can be extracted for logo
    computation
  repo: https://github.com/comprna/R2Dtool
- name: CHEUI
  role: Upstream tool whose CHEUI-diff output (186 NSUN2-dependent m5C sites) provides
    the site list used as input for motif logo visualization
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:19c5b1752d26a9c4ac51651777a78e808c041816e6baefbe14ecd60f59f9c5bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/R2Dtool
  tools:
  - CHEUI
  - R2Dtool
  - WebLogo
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sequence-motif-logo-visualization

## Summary

Generate and inspect sequence logos (e.g. with WebLogo) around predicted RNA modification sites to confirm expected sequence-context motifs, such as the 5'-m5CNGGG-3' motif enriched at NSUN2-dependent m5C sites in HeLa DRS data.

## When to use

Use when you have a set of genomic or transcriptomic modification sites (e.g. CHEUI-diff m5C calls) and need to verify biological plausibility by checking enrichment of a known sequence motif around the modified positions.

## When NOT to use

- The modification type has no known sequence-context motif (logo would show no interpretable pattern and provide no validation signal)
- Input is raw nanopore signal or FAST5 files rather than a curated list of called sites — motif analysis must follow modification calling and annotation, not precede it

## Inputs

- List of genomic coordinates of differential modification sites (e.g. R2Dtool-annotated NSUN2-dependent m5C sites)
- Reference genome/transcriptome sequences for extracting flanking sequence windows (e.g. GRCh38 / Gencode v38)

## Outputs

- Sequence logo image (e.g. WebLogo PNG/PDF) showing nucleotide frequency enrichment around modified positions
- Confirmation of motif enrichment (5'-m5CNGGG-3' for NSUN2-dependent m5C sites) written into the results table

## How to apply

After identifying differential modification sites (e.g. the 186 NSUN2-dependent m5C sites found by CHEUI-diff at Benjamini-Hochberg FDR 0.05), extract the nucleotide sequences flanking each modified position and feed them to WebLogo to compute sequence logos centered on the m5C positions. A correct result shows visual enrichment of the NSUN2 target motif 5'-m5CNGGG-3' in the logo, confirming that the called sites match the known NSUN2 sequence specificity. This step serves as a sequence-level validation that complements the statistical differential calls and the bsRNA-seq overlap (18 of 186 sites with prior support).

## Related tools

- **WebLogo** (Compute sequence logos of nucleotides around NSUN2-dependent m5C positions to confirm enrichment of the 5'-m5CNGGG-3' motif)
- **R2Dtool** (Annotate and lift CHEUI differential m5C calls to genomic coordinates (R2_annotate with Gencode v38 GTF, R2_lift) so flanking sequences can be extracted for logo computation) — https://github.com/comprna/R2Dtool
- **CHEUI** (Upstream tool whose CHEUI-diff output (186 NSUN2-dependent m5C sites) provides the site list used as input for motif logo visualization) — https://github.com/comprna/R2Dtool

## Examples

```
weblogo --format png --size large --title "NSUN2-dependent m5C sites" --input m5c_site_flanks.txt -c classic -o nsun2_m5c_logo.png
```

## Evaluation signals

- The WebLogo centered on the differential m5C positions shows a clear 5'-m5CNGGG-3' motif consistent with NSUN2 specificity
- The number of sites contributing to the logo matches the CHEUI-diff output (186 unique genomic m5C sites)
- Site coordinates used for logo extraction are genomic (post R2_lift), annotated with the correct GTF (Gencode v38)
- Flanking sequence windows are extracted in a consistent orientation and centered exactly on the modified cytosine

## Limitations

- Motif enrichment is a consistency check, not independent validation — the article notes that sites detected only from nanopore reads still require independent validation experiments
- Biases and differences in detection rates between technologies mean the logo reflects only sites recoverable by CHEUI at the applied thresholds (Model 2 probability cutoffs, coverage >20 reads)

## Evidence

- [other] Compute sequence logos of the sites around NSUN2-dependent m5C positions using WebLogo to confirm enrichment of the 5'-m5CNGGG-3' motif.: [evidence span withheld — anchor status: not_found]
- [other] CHEUI-diff identified 186 potential NSUN2-dependent unique genomic m5C sites in the HeLa WT vs NSUN2-KO comparison, 18 of which had prior bsRNA-seq support.: [evidence span withheld — anchor status: not_found]
- [methods] the R2Dtool R2_annotate command was used with default parameters and the relevant GTF annotation: "the R2Dtool R2_annotate command was used with default parameters and the relevant GTF annotation"
- [discussion] Independent validation experiments of the modified sites detected only from nanopore reads will be necessary to confirm these predictions: "Independent validation experiments of the modified sites detected only from nanopore reads will be necessary to confirm these predictions"
