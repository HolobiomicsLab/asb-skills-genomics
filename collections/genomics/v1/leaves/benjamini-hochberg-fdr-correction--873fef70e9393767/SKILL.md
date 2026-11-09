---
name: benjamini-hochberg-fdr-correction
description: Use when p-values from CHEUI-diff's per-site Mann-Whitney U-tests (or
  a comparable differential nanopore m6A test) need Benjamini-Hochberg FDR correction
  before counting significant sites at alpha = 0.05, 0.01, and 0.001.
when_to_use_negative:
- Nanocompore already provides pre-adjusted p-values, and Xpore output is meant to
  be filtered with the 'xpore postprocessing' module — applying BH a second time to
  these outputs would double-correct and undercount significant sites.
- Sites with an absolute stoichiometry difference <0.1 or fewer than 20 aligned reads
  in either condition are not part of the CHEUI-diff test set and must be excluded
  before correction.
- Single-condition m6A/m5C site calling with CHEUI-solo, where FDR is instead estimated
  by shuffling read signals across transcriptomic sites — this skill addresses two-condition
  differential testing only.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CHEUI
  role: CHEUI-diff performs the per-site Mann-Whitney U-tests on CHEUI-solo Model
    1 per-read modification probabilities whose p-values are Benjamini-Hochberg corrected
    by this skill
  repo: https://github.com/comprna/CHEUI
- name: Xpore
  role: Benchmark comparator; its output is filtered via the 'xpore postprocessing'
    module and significant sites counted at the same three alpha levels
- name: Nanocompore
  role: Benchmark comparator; significance thresholds are applied directly to its
    pre-adjusted p-values at the same three alpha levels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:a26a8cba05700068de4f331265002d0d13fb2360e38ac047e9283e9d47a59775
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Nanocompore
  - Xpore
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# benjamini-hochberg-fdr-correction

## Summary

Applies Benjamini-Hochberg FDR correction to CHEUI-diff p-values (per-site Mann-Whitney U-tests on CHEUI-solo Model 1 per-read m6A probabilities) so differentially modified adenosine sites between HEK293 WT and METTL3-KO can be called at alpha = 0.05, 0.01, and 0.001 and compared head-to-head with Xpore and Nanocompore.

## When to use

Use when p-values from CHEUI-diff's per-site Mann-Whitney U-tests (or a comparable differential nanopore m6A test) need Benjamini-Hochberg FDR correction before counting significant sites at alpha = 0.05, 0.01, and 0.001.

## When NOT to use

- Nanocompore already provides pre-adjusted p-values, and Xpore output is meant to be filtered with the 'xpore postprocessing' module — applying BH a second time to these outputs would double-correct and undercount significant sites.
- Sites with an absolute stoichiometry difference <0.1 or fewer than 20 aligned reads in either condition are not part of the CHEUI-diff test set and must be excluded before correction.
- Single-condition m6A/m5C site calling with CHEUI-solo, where FDR is instead estimated by shuffling read signals across transcriptomic sites — this skill addresses two-condition differential testing only.

## Inputs

- CHEUI-diff output table of per-site Mann-Whitney U-test p-values for adenosine sites in both conditions
- CHEUI-solo Model 1 per-read modification probability files for the 3 WT and 3 METTL3-KO replicates
- Per-site stoichiometry estimates for WT and METTL3-KO (to filter sites with |Δstoichiometry| < 0.1)
- Read-coverage filter: sites with >20 aligned reads in both conditions
- Significance thresholds alpha = 0.05, 0.01, 0.001

## Outputs

- Benjamini-Hochberg-adjusted q-values for each tested adenosine site
- Counts of differentially modified sites (increased stoichiometry in WT) at alpha = 0.05, 0.01, and 0.001
- differential_m6A_HEK293_WT_vs_METTL3KO.csv with per-tool differential site counts at the three significance levels

## How to apply

First restrict the test set to adenosine sites with >20 aligned reads in both the WT and METTL3-KO HEK293 conditions, and exclude sites with an absolute stoichiometry difference <0.1, since CHEUI-diff does not test them. Run CHEUI-diff on the CHEUI-solo Model 1 per-read modification probabilities using a Mann-Whitney U-test, then apply Benjamini-Hochberg FDR correction across the tested sites and count differentially modified sites with increased stoichiometry in WT at alpha = 0.05, 0.01, and 0.001. For the benchmark, count Xpore significant sites after filtering with the 'xpore postprocessing' module, and apply thresholds directly to Nanocompore's pre-adjusted p-values — do NOT re-apply BH to already-adjusted p-values. Validate the biology by confirming that the METTL3-KO sample shows reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254 as reported in the paper) and estimate per-tool false positives by running the same correction on METTL3-KO replicates 2 and 3 only. Assemble the per-tool site counts at the three alpha levels into differential_m6A_HEK293_WT_vs_METTL3KO.csv.

## Related tools

- **CHEUI** (CHEUI-diff performs the per-site Mann-Whitney U-tests on CHEUI-solo Model 1 per-read modification probabilities whose p-values are Benjamini-Hochberg corrected by this skill) — https://github.com/comprna/CHEUI
- **Xpore** (Benchmark comparator; its output is filtered via the 'xpore postprocessing' module and significant sites counted at the same three alpha levels)
- **Nanocompore** (Benchmark comparator; significance thresholds are applied directly to its pre-adjusted p-values at the same three alpha levels)

## Examples

```
from statsmodels.stats.multitest import multipletests; q = multipletests(cheui_diff_pvalues, method="fdr_bh")[1]
```

## Evaluation signals

- Tested-site set must exclude sites with |Δstoichiometry| < 0.1 and require >20 aligned reads in both WT and METTL3-KO before correction is applied.
- Significant-site counts must be monotonic in alpha: more sites at 0.05 than at 0.01 than at 0.001 for every tool.
- After BH correction, CHEUI-diff should detect more WT-increased m6A sites than Xpore and Nanocompore at all three alpha levels (Fig. 3d head-to-head benchmark).
- The METTL3-KO sample should show reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254 as reported in the paper).
- The METTL3-KO replicates 2 vs 3 false-positive control should yield near-zero significant sites after correction for each tool.

## Limitations

- Given the large number of transcriptomic sites tested, very strict cut-offs are needed to maintain a low expected FDR, which reduces CHEUI sensitivity; relaxing thresholds recovers more sites supported by m6ACE-seq/miCLIP but at the cost of potential false positives.
- Many sites identified by orthogonal methods had high CHEUI probabilities below the set thresholds, so corrected calls may miss orthogonally supported sites.
- Nanopore signal variability can make differences between modified and non-modified reads comparable to variation among non-modified reads, limiting differential-test power; sites detected only from nanopore reads require independent validation.

## Evidence

- [methods] Apply Benjamini-Hochberg FDR correction to the CHEUI-diff p-values and count differentially modified sites (increased stoichiometry in WT) at alpha = 0.05, 0.01, and 0.001.: [evidence span withheld — anchor status: not_found]
- [discussion] we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity: "we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [results] CHEUI-diff detected more sites with higher m6A modification stoichiometry in WT than Xpore and Nanocompore at three significance thresholds (alpha = 0.05, 0.01, 0.001): [evidence span withheld — anchor status: not_found]
- [readme] CHEUI also detects differential m6A or m5C between any two conditions.: "CHEUI also detects differential m6A or m5C between any two conditions."
