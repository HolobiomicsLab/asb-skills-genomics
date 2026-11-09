---
name: read-level-methylation-confidence-filtering
description: Use when you have per-read per-CpG methylation probabilities (e.g. ccsmeth
  5mCpG scores on PacBio HiFi reads) and need higher read-level accuracy at the cost
  of dropping 8.9-12.8% of ambiguous calls.
when_to_use_negative:
- You need to keep every call or detect hemimethylated CpGs — filtering removes 8.9-12.8%
  of calls and these methods cannot detect hemimethylated CpGs
- The target modification is non-CpG 5mC or another modification such as 6mA — ccsmeth's
  design covers CpG 5mC detection only
- You are working at the genome-wide site level rather than the read level — site-level
  aggregation of calls is a different model/task
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ccsmeth
  role: Produces the per-read 5mCpG probability calls (Pr) on PacBio CCS reads whose
    confidence margins are filtered by delta-p
  repo: https://github.com/PengNi/ccsmeth
- name: nanomethphase
  role: Reference implementation of analogous delta-probability call thresholding
    in its methyl_call_processor module, where calls between methylated/unmethylated
    thresholds are classed as ambiguous
  repo: https://github.com/vahidAK/NanoMethPhase
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:5c52a3a5d3177a3a4f6398337bcae8ab93ab33a7e7e5d028ac376a99c69351c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccsmeth
  - nanomethphase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# read-level-methylation-confidence-filtering

## Summary

A confidence-margin filter for per-read 5mCpG methylation calls: discard ambiguous calls whose delta-p = |Pr − (1 − Pr)| falls below a 0.33 threshold to raise ccsmeth read-level accuracy, trading away a small fraction of calls.

## When to use

Use when you have per-read per-CpG methylation probabilities (e.g. ccsmeth 5mCpG scores on PacBio HiFi reads) and need higher read-level accuracy at the cost of dropping 8.9-12.8% of ambiguous calls.

## When NOT to use

- You need to keep every call or detect hemimethylated CpGs — filtering removes 8.9-12.8% of calls and these methods cannot detect hemimethylated CpGs
- The target modification is non-CpG 5mC or another modification such as 6mA — ccsmeth's design covers CpG 5mC detection only
- You are working at the genome-wide site level rather than the read level — site-level aggregation of calls is a different model/task

## Inputs

- ccsmeth read-level 5mCpG probability calls on PacBio HiFi/CCS reads (per-read, per-CpG Pr values)
- Ground-truth methylation states from PCR-treated (unmethylated) and M.SssI-methylated control DNA (e.g. NA12898)
- Delta-p threshold (default 0.33)

## Outputs

- Filtered high-confidence per-read methylation calls (ambiguous calls removed)
- Read-level accuracy before and after filtering
- Percentage of calls discarded by the filter
- accuracy_metrics.tsv containing all computed metrics

## How to apply

Start from ccsmeth read-level 5mCpG probability calls (Pr per read per CpG) produced on PacBio HiFi reads, ideally with ground truth from PCR-treated (unmethylated) and M.SssI-methylated control DNA. For every call compute the confidence margin delta-p = |Pr − P'r| where P'r = 1 − Pr, i.e. the distance of the probability from the 0.5 ambiguity boundary. Discard calls with delta-p below the 0.33 threshold, keeping only high-confidence methylated (Pr near 1) or unmethylated (Pr near 0) calls. Recompute read-level accuracy against ground truth both before and after filtering, and record the percentage of calls discarded. Validate success by confirming accuracy improves by 3.5-4.2 percentage points over unfiltered calls while discarding 8.9-12.8% of calls; write all metrics to accuracy_metrics.tsv. The same delta-probability filtering concept is used downstream by tools such as nanomethphase's methyl_call_processor, which treats calls between thresholds as ambiguous.

## Related tools

- **ccsmeth** (Produces the per-read 5mCpG probability calls (Pr) on PacBio CCS reads whose confidence margins are filtered by delta-p) — https://github.com/PengNi/ccsmeth
- **nanomethphase** (Reference implementation of analogous delta-probability call thresholding in its methyl_call_processor module, where calls between methylated/unmethylated thresholds are classed as ambiguous) — https://github.com/vahidAK/NanoMethPhase

## Examples

```
python -c "import csv,sys; rows=[r for r in csv.DictReader(open('ccsmeth_per_read.tsv'),delimiter='\t') if abs(float(r['Pr'])-(1-float(r['Pr'])))>=0.33]; csv.writer(open('filtered_calls.tsv','w'),delimiter='\t',linewriter=sys.stdout).writerows(rows)"
```

## Evaluation signals

- Filtered read-level accuracy exceeds the unfiltered ccsmeth accuracy by 3.5-4.2 percentage points
- Fraction of discarded calls falls in the expected 8.9-12.8% range
- Every retained call satisfies delta-p = |Pr - (1 - Pr)| >= 0.33 (i.e. Pr >= ~0.665 or <= ~0.335)
- Accuracy is computed against known methylation states from PCR-treated and M.SssI-methylated control DNA rather than an inferred label
- All metrics (before/after accuracy, discard rate) are present and well-formed in accuracy_metrics.tsv

## Limitations

- Filtering trades coverage for precision: 8.9-12.8% of per-read calls are discarded
- ccsmeth methods only consider symmetric methylation, combining features from both strands, and are incapable of detecting hemimethylated CpGs
- The approach cannot be applied to non-CpG 5mC detection or other modifications such as 6mA
- The strand-specific-methylation model has significantly reduced performance due to limited subread depth

## Evidence

- [results] ccsmeth read-level accuracy with delta-p threshold: [evidence span withheld — anchor status: not_found]
- [abstract] ccsmeth read-level baseline accuracy: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [abstract] Ground-truth training/validation material: "We sequence polymerase-chain-reaction treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth"
- [readme] Delta-probability ambiguity thresholding precedent: [evidence span withheld — anchor status: external_doc]
