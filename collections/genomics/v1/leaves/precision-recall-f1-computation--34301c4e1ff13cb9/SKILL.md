---
name: precision-recall-f1-computation
description: Use when benchmarking SV callers whose outputs are VCFs against a simulated
  or GIAB truth set, and you need per-SV-type (insertion vs deletion) recall, precision,
  and F1 to rank methods.
when_to_use_negative:
- The caller outputs have not been filtered to the common size range (>50 bp insertions/deletions),
  so metrics would not be comparable across callers.
- There is no truth set available; recall/precision/F1 cannot be computed without
  a benchmark reference.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: Blackbird
  role: SV caller under evaluation; its out_50.vcf (insertions and deletions longer
    than 50 bp) supplies the calls scored for recall/precision/F1
  repo: https://github.com/1dayac/Blackbird
- name: BWA
  role: Provides the BWA-indexed reference genome required for the Blackbird runs
    whose VCFs are scored
- name: SPAdes
  role: spades_for_blackbird codebase used to compile/run Blackbird's local assembly,
    generating the VCF calls being benchmarked
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:3c4c16662768f08f4eb876eea38bca380a626bd02fba0e4d2277b5c0446fc2ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# precision-recall-f1-computation

## Summary

Computes recall, precision, and F1 score for structural variant callers (e.g. Blackbird, LongRanger, Manta, SvABA, LEVIATHAN) by matching each caller's VCF insertions/deletions longer than 50 bp against a truth set, enabling head-to-head benchmarking on the same BAM.

## When to use

Use when benchmarking SV callers whose outputs are VCFs against a simulated or GIAB truth set, and you need per-SV-type (insertion vs deletion) recall, precision, and F1 to rank methods.

## When NOT to use

- The caller outputs have not been filtered to the common size range (>50 bp insertions/deletions), so metrics would not be comparable across callers.
- There is no truth set available; recall/precision/F1 cannot be computed without a benchmark reference.

## Inputs

- VCF files of insertion and deletion calls from each SV caller (e.g. Blackbird out_50.vcf plus LongRanger, Manta, SvABA, LEVIATHAN outputs)
- Simulated truth-set VCF of structural variants (e.g. 37x SLR chromosome-1 simulation) or GIAB benchmark VCF

## Outputs

- Per-caller table of recall, precision, and F1 computed separately for insertions and deletions (e.g. blackbird_slr_benchmark_metrics.csv)

## How to apply

Parse each caller's VCF output and filter to insertion and deletion calls longer than 50 bp, matching Blackbird's reporting range in out_50.vcf. Compare filtered calls to the simulated truth set using a match-tolerant evaluation such as SURVIVOR-style reciprocal overlap, so calls are counted as true positives when they agree in type and reciprocal overlap at the same locus. Compute recall = TP/(TP+FN), precision = TP/(TP+FP), and F1 = 2*precision*recall/(precision+recall) separately for insertions and deletions for each caller. Aggregate the per-caller metrics into one table (e.g. blackbird_slr_benchmark_metrics.csv) so relative performance is directly inspectable. Validate against known results: on the simulated 37x SLR chromosome-1 dataset, Blackbird SLR mode should reproduce insertions recall 58%/F1 0.71/precision 94% and deletions recall 72%/F1 0.81/precision 94%, exceeding LongRanger, Manta, SvABA, and LEVIATHAN.

## Related tools

- **Blackbird** (SV caller under evaluation; its out_50.vcf (insertions and deletions longer than 50 bp) supplies the calls scored for recall/precision/F1) — https://github.com/1dayac/Blackbird
- **BWA** (Provides the BWA-indexed reference genome required for the Blackbird runs whose VCFs are scored)
- **SPAdes** (spades_for_blackbird codebase used to compile/run Blackbird's local assembly, generating the VCF calls being benchmarked) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
survivor merge blackbird_out_50.filtered.vcf truth.vcf 500 0.7 0 0 0 0 > merged.vcf  # then compute recall/precision/F1 per SV type from TP/FP/FN counts
```

## Evaluation signals

- Blackbird's SLR-mode row reproduces insertions recall 58%, F1 0.71, precision 94% and deletions recall 72%, F1 0.81, precision 94% on the simulated 37x SLR chromosome-1 dataset.
- All metrics are computed on calls filtered to >50 bp for both the caller outputs and the truth set, matching Blackbird's out_50.vcf range.
- F1 values are consistent with the reported precision and recall (F1 = 2PR/(P+R)); e.g. insertion precision 94% with recall 58% gives F1 ~0.71.
- Blackbird's recall and F1 exceed LongRanger, Manta, SvABA, and LEVIATHAN, especially for insertions, in the aggregated metrics table.

## Limitations

- Match-tolerant evaluation (e.g. SURVIVOR-style reciprocal overlap) means near-boundary calls may be scored as true positives, so exact genotype/size agreement is not required.
- Calls in Blackbird's out.vcf ('all other calls') are described as not reliable; only the >50 bp out_50.vcf range is appropriate for this benchmark.
- Reported values are specific to the simulated 37x SLR chromosome-1 dataset and GRCh37/hg19 reference; other datasets or references will yield different metrics.

## Evidence

- [methods] In a SLR read mode Blackbird outperforms existing short-read and SLR read methods, especially for insertions.: "In a SLR read mode Blackbird outperforms existing short-read and SLR read methods, especially for insertions."
- [readme] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [results] Outperformed competitors with best overall result: insertions recall 58% F1 0.71 precision 94%; deletions recall 72% F1 0.81 precision 94%.: [evidence span withheld — anchor status: not_found]
- [readme] `out.vcf` contains all other calls, though they are not reliable.: "`out.vcf` contains all other calls, though they are not reliable."
