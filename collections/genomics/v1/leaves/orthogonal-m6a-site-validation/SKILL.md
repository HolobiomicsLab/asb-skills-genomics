---
name: orthogonal-m6a-site-validation
description: Use when a list of predicted m6A sites from nanopore differential analysis
  needs independent support from antibody-based orthogonal methods, or when comparing
  tools' agreement rates and false-positive behavior.
when_to_use_negative:
- Validating modification types other than m6A (e.g. m5C or pseudouridine) — the m6ACE-seq/miCLIP
  union covers m6A only
- Sites with fewer than 20 aligned reads in either condition, or adenosine sites whose
  absolute stoichiometry difference is <0.1 — CHEUI-diff does not test them
- Datasets lacking orthogonal antibody-based evidence (m6ACE-seq/miCLIP) for the same
  cell type; only the KO-only false-positive estimation would be possible
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CHEUI
  role: Source of differential m6A calls (CHEUI-diff) and read-level probabilities
    (CHEUI-solo Model 1) to be validated against orthogonal sites
  repo: github.com/comprna/CHEUI-public
- name: R2Dtool
  role: Converts CHEUI methylation calls to a bed-like format (cheui_to_bed.sh) and
    annotates/lifts them (R2_annotate, R2_lift) for positional comparison with m6ACE-seq/miCLIP
    sites
  repo: https://github.com/comprna/R2Dtool
- name: Xpore
  role: Competing differential m6A caller, output filtered via the 'xpore postprocessing'
    module before orthogonal-site comparison
- name: Nanocompore
  role: Competing differential m6A caller, thresholds applied directly to pre-adjusted
    p-values before orthogonal-site comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:cce505a9da01b03d5d6c94b20848d23bcb34b06c68914a39f042004e555ae690
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHEUI
  - Nanocompore
  - R2Dtool
  - Xpore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# orthogonal-m6a-site-validation

## Summary

Validate nanopore-derived differential m6A calls (e.g. CHEUI-diff on HEK293 WT vs METTL3-KO) by intersecting them with a union of m6ACE-seq and miCLIP sites and by estimating per-tool false positives from knock-out-only replicates.

## When to use

Use when a list of predicted m6A sites from nanopore differential analysis needs independent support from antibody-based orthogonal methods, or when comparing tools' agreement rates and false-positive behavior.

## When NOT to use

- Validating modification types other than m6A (e.g. m5C or pseudouridine) — the m6ACE-seq/miCLIP union covers m6A only
- Sites with fewer than 20 aligned reads in either condition, or adenosine sites whose absolute stoichiometry difference is <0.1 — CHEUI-diff does not test them
- Datasets lacking orthogonal antibody-based evidence (m6ACE-seq/miCLIP) for the same cell type; only the KO-only false-positive estimation would be possible

## Inputs

- Differentially modified m6A site lists from CHEUI-diff (FDR-corrected, alpha = 0.05/0.01/0.001)
- Differential m6A site lists from Xpore (post 'xpore postprocessing' filtering) and Nanocompore (pre-adjusted p-values)
- Union of m6ACE-seq and miCLIP m6A site calls
- GRCh38 genome and Gencode v38 annotation for coordinate mapping (R2Dtool R2_annotate/R2_lift)
- CHEUI methylation calls (converted via cheui_to_bed.sh to bed-like format)

## Outputs

- Per-tool counts of differential m6A sites at alpha = 0.05, 0.01, 0.001 (differential_m6A_HEK293_WT_vs_METTL3KO.csv)
- Overlap statistics of predicted WT m6A sites against the union of m6ACE-seq and miCLIP sites
- Per-tool false-positive estimates from METTL3-KO replicates 2 vs 3 runs

## How to apply

After running CHEUI-diff (Mann-Whitney U-test on CHEUI-solo Model 1 per-read probabilities for adenosine sites with >20 aligned reads in both conditions, sites with absolute stoichiometry difference <0.1 excluded, Benjamini-Hochberg FDR correction), intersect the predicted WT-enriched m6A sites with the union of m6ACE-seq and miCLIP sites to quantify agreement with orthogonal evidence. Apply the same evaluation uniformly to Xpore (filtered with the 'xpore postprocessing' module) and Nanocompore (thresholds on pre-adjusted p-values) so per-tool counts at alpha = 0.05, 0.01, 0.001 are comparable, as visualized in Fig. 3d. Use R2Dtool (cheui_to_bed.sh, R2_annotate, R2_lift) to convert CHEUI methylation calls to a bed-like format against GRCh38/Gencode v38 annotation before genomic overlap. Estimate per-tool false positives by running each method on METTL3-KO replicates 2 and 3 only, where true differential m6A should be absent. Confirm the underlying biology with the reported METTL3-KO m6A reduction (Mann-Whitney U-test, p = 1.3E-254).

## Related tools

- **CHEUI** (Source of differential m6A calls (CHEUI-diff) and read-level probabilities (CHEUI-solo Model 1) to be validated against orthogonal sites) — github.com/comprna/CHEUI-public
- **R2Dtool** (Converts CHEUI methylation calls to a bed-like format (cheui_to_bed.sh) and annotates/lifts them (R2_annotate, R2_lift) for positional comparison with m6ACE-seq/miCLIP sites) — https://github.com/comprna/R2Dtool
- **Xpore** (Competing differential m6A caller, output filtered via the 'xpore postprocessing' module before orthogonal-site comparison)
- **Nanocompore** (Competing differential m6A caller, thresholds applied directly to pre-adjusted p-values before orthogonal-site comparison)

## Examples

```
bash cheui_to_bed.sh
```

## Evaluation signals

- CHEUI-diff detects more WT-enriched differential m6A sites than Xpore and Nanocompore at alpha = 0.05, 0.01, and 0.001 (Fig. 3d pattern)
- Predicted WT m6A sites show measurable overlap with the union of m6ACE-seq and miCLIP sites
- KO-only runs (METTL3-KO replicates 2 vs 3) yield near-zero differential sites, bounding per-tool false positives
- METTL3-KO sample shows reduced m6A burden relative to WT (Mann-Whitney U-test, p = 1.3E-254 as reported in the paper)

## Limitations

- Many sites identified by orthogonal methods had CHEUI probabilities below the set thresholds; relaxing thresholds recovers more sites but at the cost of potential false positives
- Strict cut-offs used on cell transcriptome sites to maintain a low expected FDR reduce CHEUI sensitivity
- Modified sites detected only from nanopore reads require independent validation experiments to confirm predictions or rule out other signal sources
- Biases and differences in detection rates exist between technologies, so low overlap with m6ACE-seq/miCLIP may reflect technology-specific detection bias rather than false positives

## Evidence

- [methods] union of m6ACE-seq and miCLIP sites as orthogonal evidence: "we employed the union of m6ACE-seq and miCLIP sites39,40."
- [methods] Xpore output filtering before comparison: "For Xpore, we used the module xpore postprocessing to filter the output."
- [methods] CHEUI calls converted to bed format for annotation: "the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format"
- [discussion] orthogonal sites can fall below CHEUI thresholds: "many of sites identified by orthogonal methods had high CHEUI probabilities that were below the set thresholds."
- [discussion] nanopore-only sites need independent validation: "Independent validation experiments of the modified sites detected only from nanopore reads will be necessary to confirm these predictions"
