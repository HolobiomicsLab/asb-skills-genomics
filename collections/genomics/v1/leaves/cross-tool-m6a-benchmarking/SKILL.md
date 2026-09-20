---
name: cross-tool-m6a-benchmarking
description: Use when evaluating which nanopore-based differential RNA modification
  caller detects m6A stoichiometry changes between two multi-replicate conditions,
  e.g. WT vs methyltransferase knockout.
when_to_use_negative:
- Single-condition m6A/m5C site and stoichiometry calling without a comparison condition
  — use CHEUI-solo models instead of CHEUI-diff
- Data are not nanopore direct RNA reads with per-site modification probabilities
  (e.g. short-read m6A-seq or antibody-based maps) — the per-read probability and
  signal-level comparisons do not apply
- Conditions lack replicates or have fewer than 20 reads per adenosine site in both
  conditions — sites fall outside the tested set by design
tools:
- name: CHEUI
  role: 'Primary differential caller: CHEUI-diff tests CHEUI-solo Model 1 per-read
    modification probabilities with a Mann-Whitney U-test after restricting to sites
    with >20 reads in both conditions'
  repo: https://github.com/comprna/CHEUI
- name: Xpore
  role: Competing differential modification tool run on the same 3 WT vs 3 KO replicates;
    output filtered with the 'xpore postprocessing' module
- name: Nanocompore
  role: Competing differential modification tool; significance thresholds applied
    directly to its pre-adjusted p-values at alpha = 0.05, 0.01, 0.001
- name: R2Dtool
  role: Positional annotation of transcriptome-mapped modification calls via cheui_to_bed.sh,
    R2_annotate, and R2_lift
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:260fef9887b1d9aaad35828f29682f5926e4a016f9ffb8fb415ec74a69093fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Nanocompore
  - R2Dtool
  - Xpore
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cross-tool-m6a-benchmarking

## Summary

Head-to-head benchmarking of differential m6A detection from Oxford Nanopore direct RNA data, comparing CHEUI-diff, Xpore, and Nanocompore on matched WT vs METTL3-KO HEK293 replicates at shared significance thresholds, with validation against m6ACE-seq/miCLIP sites and KO-only false-positive estimation.

## When to use

Use when evaluating which nanopore-based differential RNA modification caller best detects m6A stoichiometry changes between two multi-replicate conditions, e.g. WT vs methyltransferase knockout.

## When NOT to use

- Single-condition m6A/m5C site and stoichiometry calling without a comparison condition — use CHEUI-solo models instead of CHEUI-diff
- Data are not nanopore direct RNA reads with per-site modification probabilities (e.g. short-read m6A-seq or antibody-based maps) — the per-read probability and signal-level comparisons do not apply
- Conditions lack replicates or have fewer than 20 reads per adenosine site in both conditions — sites fall outside the tested set by design

## Inputs

- Oxford Nanopore direct RNA FAST5/FASTQ data for 3 HEK293 WT and 3 METTL3-KO replicates
- CHEUI-solo Model 1 per-read m6A modification probability outputs for each condition
- Reference transcriptome (GRCh38, Gencode v38) used for Minimap2 alignment and Nanopolish eventalign
- Union of m6ACE-seq and miCLIP m6A site calls for orthogonal validation
- Xpore and Nanocompore differential outputs on the same WT vs KO replicates

## Outputs

- differential_m6A_HEK293_WT_vs_METTL3KO.csv with per-tool differential site counts at alpha = 0.05, 0.01, and 0.001
- Per-tool lists of sites with higher m6A stoichiometry in WT (BH-FDR-corrected for CHEUI-diff)
- Per-tool false-positive estimates from METTL3-KO replicates 2 and 3 only
- Agreement statistics of predicted WT m6A sites versus the m6ACE-seq/miCLIP union

## How to apply

Restrict the analysis to adenosine sites with >20 aligned reads in both the WT and METTL3-KO HEK293 conditions, and run CHEUI-diff on the CHEUI-solo Model 1 per-read modification probabilities using a Mann-Whitney U-test, excluding sites with an absolute stoichiometry difference <0.1. Apply Benjamini-Hochberg FDR correction to CHEUI-diff p-values and count differentially modified sites (higher stoichiometry in WT) at alpha = 0.05, 0.01, and 0.001. Run Xpore and Nanocompore on the same 3 WT vs 3 KO replicates — filtering Xpore with the 'xpore postprocessing' module and applying thresholds directly to Nanocompore's pre-adjusted p-values — counting significant sites at the same three alpha levels. Validate predicted WT m6A sites against the union of m6ACE-seq and miCLIP sites, and estimate per-tool false positives by re-running each method on METTL3-KO replicates 2 and 3 only. Assemble per-tool differential counts into differential_m6A_HEK293_WT_vs_METTL3KO.csv and confirm the KO sample shows reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254).

## Related tools

- **CHEUI** (Primary differential caller: CHEUI-diff tests CHEUI-solo Model 1 per-read modification probabilities with a Mann-Whitney U-test after restricting to sites with >20 reads in both conditions) — https://github.com/comprna/CHEUI
- **Xpore** (Competing differential modification tool run on the same 3 WT vs 3 KO replicates; output filtered with the 'xpore postprocessing' module)
- **Nanocompore** (Competing differential modification tool; significance thresholds applied directly to its pre-adjusted p-values at alpha = 0.05, 0.01, 0.001)
- **R2Dtool** (Positional annotation of transcriptome-mapped modification calls via cheui_to_bed.sh, R2_annotate, and R2_lift) — https://github.com/comprna/R2Dtool

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- CHEUI-diff calls more WT-enriched differential sites than Xpore and Nanocompore at all three thresholds (alpha = 0.05, 0.01, 0.001), as in Fig. 3d
- METTL3-KO sample shows reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254)
- No site is tested unless it has >20 aligned reads in both conditions and an absolute stoichiometry difference >= 0.1
- KO-only runs (METTL3-KO replicates 2 and 3) provide a per-tool false-positive baseline that can be compared across tools
- CHEUI-diff p-values are Benjamini-Hochberg FDR corrected before counting, while Nanocompore counts use its pre-adjusted p-values directly

## Limitations

- Sites with an absolute stoichiometry difference <0.1 are not tested by CHEUI-diff, excluding small-effect differential sites
- Nanocompore p-values are pre-adjusted, so thresholding them is not strictly equivalent to BH-FDR-correcting raw p-values as done for CHEUI-diff
- Nanopore signal variability can be comparable between modified and non-modified reads, limiting per-tool sensitivity; strict cutoffs reduce CHEUI sensitivity to maintain low FDR

## Evidence

- [methods] run CHEUI-diff on the CHEUI-solo Model 1 per-read modification probabilities using a Mann-Whitney U-test: [evidence span withheld — anchor status: not_found]
- [methods] Sites with absolute stoichiometry difference <0.1 are excluded: [evidence span withheld — anchor status: not_found]
- [methods] Xpore output filtering step: "For Xpore, we used the module xpore postprocessing to filter the output."
- [methods] Orthogonal validation reference set: "we employed the union of m6ACE-seq and miCLIP sites39,40."
- [results] Benchmark finding favoring CHEUI-diff: [evidence span withheld — anchor status: not_found]
- [readme] CHEUI differential mode availability: "CHEUI also detects differential m6A or m5C between any two conditions."
