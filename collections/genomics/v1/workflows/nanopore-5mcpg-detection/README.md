# nanopore-5mcpg-detection-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **basecall_with_move_tables** — basecall raw nanopore signal while retaining signal-to-base moves  →  `basecalling-with-move-tables`, `dorado-move-table-basecalling`, `parallel-basecalling-of-pod5-batches`
2. **align_move_table_bam** — align basecalls without dropping signal-to-base move tags  →  `ont-read-alignment-to-reference-genome`, `bam-file-processing-and-indexing`, `sam-to-bam-conversion-and-indexing`, `sam-bam-alignment-file-handling`
3. **detect_5mcpg** — infer per-read 5mCpG probabilities from signal-aligned basecalls  →  `signal-based-cpg-methylation-detection`, `nanopore-modified-basecalling`, `nanopore-methylation-calling--ab6e4dfe90fd7ce1`, `reference-anchored-cpg-methylation-aggregation`
4. **aggregate_5mcpg_sites** — aggregate per-read 5mCpG calls into filtered CpG-site frequencies  →  `modkit-modification-pileup`, `per-site-methylation-frequency-aggregation`, `modkit-modification-site-counting`, `bedmethyl-frequency-aggregation`, `low-coverage-site-filtering`
5. **validate_methylation_profile** — compare nanopore 5mCpG frequencies with orthogonal CpG measurements  →  `bisulfite-methylation-ground-truth-matching`, `cross-platform-methylation-correlation-analysis--50dd8f2e672836d6`, `bedtools-genomic-interval-intersection`, `pearson-correlation-of-per-site-methylation-levels`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
