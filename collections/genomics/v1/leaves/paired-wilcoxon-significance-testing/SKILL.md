---
name: paired-wilcoxon-significance-testing
description: Use when quantifying real-time depletion effects (e.g., RISER globin
  mRNA depletion in whole blood) and a null-comparison is needed against per-transcript
  percent changes arising from random splitting of an independent control run.
when_to_use_negative:
- Input data are unpaired or come from different samples/conditions with no natural
  pairing (e.g., comparing two independent flow-cell runs without matched transcripts)
  — use an unpaired test instead
- Percent-change distributions are already known to differ trivially (e.g., comparing
  against a zero-null when the effect size, >90% globin depletion, is the reported
  claim) without a matched control split
- The transcript set has fewer than the required minimum counts (≥30 reads in both
  conditions), making per-transcript percent changes unstable and the paired test
  underpowered
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3308
tools:
- name: RISER
  role: Provides decisions.csv files that split accepted/rejected reads for the globin-deplete
    and control conditions whose per-transcript percent changes feed the paired test
  repo: https://github.com/comprna/riser
- name: samtools (v1.10)
  role: Filters mapped reads with option -F 2324 before per-transcript counting used
    in the significance comparison
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE reference transcriptome (GRCh38.p13)
    to produce the alignment input for per-transcript counts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:8a59e61d610d09c71f6232791ffc7bb85b36d86f94445b9a0384202d94b967f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# paired-wilcoxon-significance-testing

## Summary

Non-parametric paired significance testing used as a control analysis in the RISER globin-depletion study: per-transcript percent changes computed between two randomly split halves of an independent non-RISER run are compared against RISER-mediated changes to confirm the observed depletion/increase effects exceed random run-to-run variation.

## When to use

Use when quantifying real-time depletion effects (e.g., RISER globin mRNA depletion in whole blood) and a null-comparison is needed against per-transcript percent changes arising from random splitting of an independent control run.

## When NOT to use

- Input data are unpaired or come from different samples/conditions with no natural pairing (e.g., comparing two independent flow-cell runs without matched transcripts) — use an unpaired test instead
- Percent-change distributions are already known to differ trivially (e.g., comparing against a zero-null when the effect size, >90% globin depletion, is the reported claim) without a matched control split
- The transcript set has fewer than the required minimum counts (≥30 reads in both conditions), making per-transcript percent changes unstable and the paired test underpowered

## Inputs

- Per-transcript read counts and nucleotide counts for RISER globin-deplete and control split-flow-cell conditions (from BAM alignments filtered with samtools -F 2324 and RISER decisions.csv files)
- Per-transcript read/nt counts from two randomly split halves of an independent non-RISER whole-blood run
- Globin transcript identifiers (HBA1, HBA2, HBB) and non-globin transcript list passing the ≥95% control-read-share and ≥30-count filters

## Outputs

- Paired Wilcoxon signed rank test statistic and p-value comparing RISER per-transcript percent changes against randomly-split control percent changes
- percent_change_results.csv containing per-transcript and averaged percent changes for globin and non-globin transcripts in both read and nucleotide counts

## How to apply

First compute per-transcript read and nucleotide count percent changes (RISER vs control) by splitting reads by channel (even channels = RISER deplete, remaining = control), restricting to transcripts taking up at least 95% of control-condition reads with at least 30 counts in both conditions and excluding lncRNAs overlapping protein-coding exons. Then, as a significance control, recompute the same per-transcript percent changes between two randomly split halves of an independent non-RISER run, producing a paired distribution under the null hypothesis of no real depletion. Apply a paired Wilcoxon signed rank test to compare the RISER-derived per-transcript percent changes with the randomly-split control changes. Interpret significant differences as evidence that the 93% average globin read decrease (n = 4 transcripts: HBA1, HBA2, HBB) and 16% non-globin read increase (n = 100) exceed run-to-run noise.

## Related tools

- **RISER** (Provides decisions.csv files that split accepted/rejected reads for the globin-deplete and control conditions whose per-transcript percent changes feed the paired test) — https://github.com/comprna/riser
- **samtools (v1.10)** (Filters mapped reads with option -F 2324 before per-transcript counting used in the significance comparison)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE reference transcriptome (GRCh38.p13) to produce the alignment input for per-transcript counts)

## Examples

```
wilcox.test(riser_pct_change, split_pct_change, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- RISER-depleted globin transcripts (HBA1, HBA2, HBB, n = 4) show ~93% average read-count decrease, exceeding the reported >90% depletion threshold
- Non-globin transcripts (n = 100) show ~16% average read-count increase relative to control
- Paired Wilcoxon signed rank test on the randomly split non-RISER control halves yields percent changes consistent with zero (no significant effect), confirming run-to-run noise is smaller than the RISER effect
- Only transcripts with ≥30 reads in both conditions and ≥95% of control-condition reads are included; lncRNAs overlapping protein-coding exons are excluded
- All per-transcript and averaged percent changes are written to percent_change_results.csv for reproducibility

## Limitations

- The lag between RISER's reject command and the hardware voltage reversal, and the efficiency of the reversal, vary across platforms and are beyond RISER's control, adding noise to measured per-transcript changes
- RNA secondary structures on the trans side of the pore can impede or completely inhibit reverse translocation by voltage reversal, weakening the depletion effect being tested
- Very lowly expressed transcripts (below the 30-count threshold) cannot be reliably assessed, limiting the paired test to moderately/highly abundant transcripts

## Evidence

- [abstract] achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average: "achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average"
- [methods] unmapped reads were removed using samtools34 (v1.10) with option -F 2324: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [results] RISER reduced the read count of each globin mRNA transcript by 93% on average (n = 4) while increasing reads for non-globin mRNA transcripts by 16% on average (n = 100): [evidence span withheld — anchor status: not_found]
- [other] recompute percent changes between two randomly split halves of an independent non-RISER run and compare with a paired Wilcoxon signed rank test: [evidence span withheld — anchor status: not_found]
- [discussion] the lag between the read-until API receiving a reject command from RISER and requesting a voltage reversal in the hardware, as well as the efficiency of the voltage reversal itself: "the lag between the read-until API receiving a reject command from RISER and requesting a voltage reversal in the hardware, as well as the efficiency of the voltage reversal itself"
