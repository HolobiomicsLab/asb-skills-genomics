---
name: nanopore-read-probability-statistical-testing
description: Use when comparing nanopore direct RNA m6A between two conditions (e.g.
  WT vs knock-out replicates) using per-read modification probabilities to call differentially
  modified transcriptomic sites.
when_to_use_negative:
- Sites with an absolute stoichiometry difference <0.1 between conditions — CHEUI-diff
  does not test these
- Sites lacking >20 aligned reads in both conditions, or sites that are not adenosines
- Single-condition m6A/m5C discovery without a control condition — use CHEUI-solo
  site-level prediction instead of the differential test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHEUI
  role: Predicts per-read m6A modification probabilities (CHEUI-solo Model 1) and
    tests differential modification between conditions (CHEUI-diff, Mann-Whitney U-test
    with BH FDR)
  repo: https://github.com/comprna/CHEUI-public
- name: Xpore
  role: Benchmark comparison tool for differential m6A; output filtered with the xpore
    postprocessing module
- name: Nanocompore
  role: Benchmark comparison tool for differential m6A; significance thresholds applied
    directly to its pre-adjusted p-values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:3e38d933b9595ee66a9bdbc64bf0009917a8e07dd4b88a8f6c1f85fbb9596b24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - Nanocompore
  - Xpore
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# nanopore-read-probability-statistical-testing

## Summary

Detects differential m6A between two conditions by applying a Mann-Whitney U-test to CHEUI-solo Model 1 per-read modification probabilities, with Benjamini-Hochberg FDR correction, as implemented in CHEUI-diff and benchmarked against Xpore and Nanocompore on HEK293 WT vs METTL3-KO data.

## When to use

Use when comparing nanopore direct RNA m6A between two conditions (e.g. WT vs knock-out replicates) using per-read modification probabilities to call differentially modified transcriptomic sites.

## When NOT to use

- Sites with an absolute stoichiometry difference <0.1 between conditions — CHEUI-diff does not test these
- Sites lacking >20 aligned reads in both conditions, or sites that are not adenosines
- Single-condition m6A/m5C discovery without a control condition — use CHEUI-solo site-level prediction instead of the differential test

## Inputs

- CHEUI-solo Model 1 per-read m6A modification probabilities for each condition (WT and KO replicates)
- Nanopore direct RNA FAST5/FASTQ aligned to a GRCh38 / Gencode v38 transcriptome reference (samtools -F 232457-filtered BAM)
- Union of m6ACE-seq and miCLIP m6A sites for orthogonal validation
- Comparison tool outputs (Xpore with xpore postprocessing filtering; Nanocompore pre-adjusted p-values)

## Outputs

- BH-FDR-corrected differential m6A site calls at alpha = 0.05, 0.01, and 0.001
- differential_m6A_HEK293_WT_vs_METTL3KO.csv with per-tool differential site counts at the three significance levels
- Per-tool false-positive estimates from KO replicate 2 vs replicate 3 runs

## How to apply

Restrict testing to adenosine sites with >20 aligned reads in both conditions, and run CHEUI-diff on CHEUI-solo Model 1 per-read modification probabilities using a Mann-Whitney U-test; sites with an absolute stoichiometry difference <0.1 are not tested. Apply Benjamini-Hochberg FDR correction to the resulting p-values and count differentially modified sites (increased stoichiometry in WT) at alpha = 0.05, 0.01, and 0.001. For benchmarking, run Xpore (filtered with the 'xpore postprocessing' module) and Nanocompore (thresholds applied directly to pre-adjusted p-values) on the same 3 WT vs 3 KO replicates at the same three alpha levels. Assess agreement of predicted WT m6A sites against the union of m6ACE-seq and miCLIP sites, and estimate per-tool false positives by running each method on METTL3-KO replicates 2 and 3 only. Assemble per-tool differential counts into differential_m6A_HEK293_WT_vs_METTL3KO.csv and verify that the METTL3-KO sample shows reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254).

## Related tools

- **CHEUI** (Predicts per-read m6A modification probabilities (CHEUI-solo Model 1) and tests differential modification between conditions (CHEUI-diff, Mann-Whitney U-test with BH FDR)) — https://github.com/comprna/CHEUI-public
- **Xpore** (Benchmark comparison tool for differential m6A; output filtered with the xpore postprocessing module)
- **Nanocompore** (Benchmark comparison tool for differential m6A; significance thresholds applied directly to its pre-adjusted p-values)

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- CHEUI-diff detects more sites with higher m6A stoichiometry in WT than Xpore and Nanocompore at alpha = 0.05, 0.01, and 0.001 (Fig. 3d expectation)
- Mann-Whitney U-test confirms METTL3-KO has reduced m6A burden relative to WT with p = 1.3E-254
- All counted sites are adenosines with >20 aligned reads in both WT and METTL3-KO and |stoichiometry difference| >= 0.1
- p-values show Benjamini-Hochberg FDR correction; counts are reported at three alpha thresholds
- KO-vs-KO replicate runs (replicates 2 vs 3) yield low false-positive counts for each tool

## Limitations

- Very strict cut-offs are used on cell transcriptome sites to maintain a low expected FDR, reducing CHEUI sensitivity; relaxing thresholds recovers more orthogonal-method sites but at the cost of potential false positives
- Differences between modified and non-modified read signals are often comparable to variation among non-modified reads, limiting single-read discrimination
- Sites detected only from nanopore reads require independent validation experiments
- For detecting m6A vs m5C, different CHEUI preprocessing scripts and trained models must be used (CHEUI_preprocess_m6A.py vs CHEUI_preprocess_m5C.py; model1/model2 .h5 files per modification)

## Evidence

- [abstract] abstract support for CHEUI-diff: "CHEUI predicts N6-methyladenosine (m6A) and 5-methylcytosine (m5C) in individual molecules from the same sample, the stoichiometry at transcript reference sites, and differential methylation between"
- [full_text] workflow: site and read filters plus U-test: [evidence span withheld — anchor status: not_found]
- [methods] benchmark comparison setup: "For Xpore, we used the module xpore postprocessing to filter the output."
- [methods] orthogonal validation reference set: "we employed the union of m6ACE-seq and miCLIP sites39,40."
- [readme] readme: CHEUI differential capability: "CHEUI also detects differential m6A or m5C between any two conditions."
- [readme] readme: signal-to-transcript alignment prerequisite: [evidence span withheld — anchor status: external_doc]
