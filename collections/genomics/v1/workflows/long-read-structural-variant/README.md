# long-read-structural-variant-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **align_long_reads** — align long reads to the reference genome for structural-variant calling  →  `long-read-genome-alignment--8cec567f57b0cb03`, `long-read-alignment-processing`, `long-read-genome-alignment-minimap2`, `invocation-and-execution-of-long-read-aligners`
2. **call_structural_variants** — generate separate structural-variant call sets from the alignments  →  `nanopore-structural-variant-calling`, `structural-variant-calling-long-reads`, `structural-variant-calling-from-bam`, `structural-variant-calling-from-long-read-sequencing`
3. **filter_per_caller_calls** — verify read-depth support of the copy-number calls in each per-caller call set  →  `coverage-based-sv-filtering`, `cnv-call-filtering-by-read-depth`, `read-depth-coverage-filtering`, `deletion-duplication-coverage-filtering`
4. **merge_consensus_calls** — consolidate the filtered per-caller structural variants  →  `multi-caller-sv-callset-merging`, `sv-call-filtering-by-caller-support`, `sv-variant-overlap-collapse`, `structural-variant-vcf-merging`
5. **benchmark_callset** — compare consensus calls with truth under truth-set-specific metric limits  →  `truvari-parameter-tuning-evaluation`, `truvari-callset-comparison-against-giab`, `vcf-callset-precision-recall-computation`, `structural-variant-truth-set-benchmarking`, `truvari-benchmark-vcf-comparison`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
