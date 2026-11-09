---
name: precision-recall-benchmark-tabulation
description: Use when comparing insertion-caller outputs (VCFs of NRS events >300
  bp) against a truth callset and you need per-tool precision/recall values, e.g.
  for a sensitivity-vs-coverage curve or a head-to-head benchmark table.
when_to_use_negative:
- Input calls are already matched to truth — no need to re-run the matching step;
  only recompute metrics if tolerances changed
- Insertions shorter than 300 bp are the target — the paper explicitly restricts the
  analysis and output filtering to insertions longer than 300 bp with anchors summing
  over 300 bp, so metrics from this scheme do not generalize to short indels
- Evaluating tools on data types they cannot process (e.g. running PopIns2/Pamir on
  platforms without equal-length paired ends) — the paper notes short-read baselines
  fail to correctly assemble and locate most insertions >300 bp, making the benchmark
  uninformative
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Novel-X
  role: Primary NRS insertion caller being benchmarked; Snakemake pipeline run with
    --10x --high-coverage defaults to emit VCFs of insertions >300 bp
  repo: https://github.com/1dayac/Novel-X
- name: PopIns2
  role: Short-read baseline caller whose precision/recall is computed across the 52x/39x/26x/13x
    downsampled simulated datasets
- name: NUI
  role: Synthetic long-read baseline caller compared to Novel-X at each coverage level
- name: SuperNova
  role: Whole-genome diploid assembler baseline; its assembly is aligned to the reference
    and novel insertions extracted via Paftools for the NA19240-10x comparison
- name: Paftools
  role: Extracts novel insertion calls from the SuperNova assembly-to-reference (Minimap2
    PAF) alignment before matching against the PacBio validation callset
- name: LongRanger
  role: Produces the 10x-aligned BAM that serves as the common input to all benchmarked
    callers
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:564aa67a4714ac21167c7c651d39e771223cd0082df44e52209a1f8ab431e638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - LongRanger
  - NUI
  - Novel-X
  - Paftools
  - PopIns2
  - SuperNova
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# precision-recall-benchmark-tabulation

## Summary

Tabulate precision and recall of non-reference sequence (NRS) insertion callers — e.g. Novel-X versus PopIns2, NUI, or SuperNova/Paftools — by matching each tool's VCF calls against a ground-truth or PacBio validation callset under fixed matching tolerances (breakpoints within 100 bp, lengths within 5%). This is how the Novel-X paper quantified accuracy on both downsampled simulated and real 10x/stLFR/TELL-Seq datasets.

## When to use

Use when comparing insertion-caller outputs (VCFs of NRS events >300 bp) against a truth callset and you need per-tool precision/recall values, e.g. for a sensitivity-vs-coverage curve or a head-to-head benchmark table.

## When NOT to use

- Input calls are already matched to truth — no need to re-run the matching step; only recompute metrics if tolerances changed
- Insertions shorter than 300 bp are the target — the paper explicitly restricts the analysis and output filtering to insertions longer than 300 bp with anchors summing over 300 bp, so metrics from this scheme do not generalize to short indels
- Evaluating tools on data types they cannot process (e.g. running PopIns2/Pamir on platforms without equal-length paired ends) — the paper notes short-read baselines fail to correctly assemble and locate most insertions >300 bp, making the benchmark uninformative

## Inputs

- Tool-generated VCF callsets of NRS insertions >300 bp (e.g. from Novel-X, PopIns2, NUI, SuperNova+Paftools)
- Ground-truth insertion callset from simulated data (e.g. 2000 LRSIM-embedded insertions in GRCh38) or published PacBio long-read validation callsets (e.g. for NA19240)
- Synthetic long-read BAM files (10x Genomics Chromium LongRanger-aligned, TELL-Seq, or stLFR)
- Downsampled read sets at 52x/39x/26x/13x coverage (from 0.8/0.6/0.4/0.2 downsampling)

## Outputs

- benchmark_table.csv with precision and recall per tool per coverage level or per dataset
- Per-coverage-level recall/precision table mirroring Supplementary Table S1
- VCFs used for benchmarking Figures 4–5 and Tables S1–S3 (as distributed in novel_insertions_supplementary)

## How to apply

Run each tool (Novel-X, PopIns2, NUI, or SuperNova/Paftools) on the same linked-read BAM with default recommended parameters, emitting VCFs of insertions longer than 300 bp with anchor-length sums exceeding 300 bp. For simulated data, downsample the 65x LRSIM-simulated read set (2000 random insertions of 50–6000 bp in GRCh38) at 0.8/0.6/0.4/0.2 to obtain 52x/39x/26x/13x coverage levels. Match calls to the truth set counting a true positive only when breakpoint positions differ by ≤100 bp AND insertion lengths differ by ≤5% (the paper's tolerance scheme); optionally apply a QUAST-like best-alignment-subset filtering that maximizes continuously covered base pairs when scoring contig alignments. Compute recall and precision per tool per coverage level (mirroring Supplementary Table S1) or per tool per real dataset (mirroring Figures 4–5), writing results to a benchmark table such as benchmark_table.csv.

## Related tools

- **Novel-X** (Primary NRS insertion caller being benchmarked; Snakemake pipeline run with --10x --high-coverage defaults to emit VCFs of insertions >300 bp) — https://github.com/1dayac/Novel-X
- **PopIns2** (Short-read baseline caller whose precision/recall is computed across the 52x/39x/26x/13x downsampled simulated datasets)
- **NUI** (Synthetic long-read baseline caller compared to Novel-X at each coverage level)
- **SuperNova** (Whole-genome diploid assembler baseline; its assembly is aligned to the reference and novel insertions extracted via Paftools for the NA19240-10x comparison)
- **Paftools** (Extracts novel insertion calls from the SuperNova assembly-to-reference (Minimap2 PAF) alignment before matching against the PacBio validation callset)
- **LongRanger** (Produces the 10x-aligned BAM that serves as the common input to all benchmarked callers)

## Examples

```
python novel-x.py run --10x --high-coverage  # then match output VCF to truth, counting TPs when breakpoints differ ≤100 bp and lengths differ ≤5%
```

## Evaluation signals

- Novel-X and NUI should show stable precision/recall at 26x coverage and above with a drop only at 13x; PopIns2's performance should worsen gradually with each downsample step — deviations indicate a benchmark-setup error
- On NA19240-10x, Novel-X should achieve precision ≈59% and recall ≈37.0%, versus SuperNova/Paftools precision ≈56% and recall ≈37.4%; the benchmark table is accepted only if these reproduce
- Every counted true positive satisfies both tolerances: breakpoint positions differ by ≤100 bp AND insertion lengths differ by ≤5%
- All reported calls respect the output filter: insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp
- Benchmark VCFs should be cross-checkable against the novel_insertions_supplementary repository, which contains all VCFs used for Figures 4–5 and Tables S1–S3

## Limitations

- The 100 bp / 5% matching scheme is only meaningful for insertions >300 bp; small insertions often lack sufficient barcode coverage for anchor assembly, so Novel-X underperforms there and the truth-matching scheme does not extend below 300 bp
- SuperNova offers no user control over assembler parameters (defaults tuned for whole human genome diploid assembly), so baseline differences may partly reflect parameter mismatch rather than method quality
- The Novel-X repository notes an unresolved issue on the filter_target_contig stage, which can affect callsets feeding into the benchmark
- On simulated data, results depend on LRSIM simulation parameters and downsampling ratios (0.8/0.6/0.4/0.2); different simulation pipelines will not reproduce Supplementary Table S1 exactly

## Evidence

- [readme] Novel-X precision/recall vs SuperNova/Paftools on NA19240: [evidence span withheld — anchor status: external_doc]
- [readme] Downsampling scheme for simulated datasets: [evidence span withheld — anchor status: external_doc]
- [methods] Output call filter applied before benchmarking: [evidence span withheld — anchor status: not_found]
- [methods] Scoring of contig alignments against truth: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [intro] Rationale for head-to-head benchmark against existing methods: "In this study, we introduce an integrated mapping-based and assembly-based algorithm using synthetic long reads, which is substantially more accurate than existing methods for NRS insertion discovery"
- [discussion] Small-insertion coverage caveat affecting recall at low sizes: "Novel-X needs a substantial number of distinct barcodes to be associated with it. For small insertions, the number of associated barcodes is usually small and the coverage is not sufficient"
